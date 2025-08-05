---
title: "Deployments and Services"
slug: "deployments-and-services"
date: 2025-06-02
tags: ["containers", "cloud", "PAAS", "OCI"]
categories: ["Kubernetes"]
series: ["Kubernetes course"]
showTableOfContents: true
summary: In this article we will explore what they are, how they work, and why they are fundamental components in any Kubernetes architecture :nerd_face:.
featureimagecaption: Photo by [orbtal media](https://unsplash.com/es/@orbtalmedia) on [Unsplash](https://unsplash.com)
draft: false
enableEmoji: true
---

## Introduction

Kubernetes has become the de facto standard for container orchestration. However, for many developers, the concepts of deployments and services can be confusing at first.

## Basic Concepts
### Deployment

A **deployment** in Kubernetes is a set of instructions that defines how a group of pods should be created, run, and maintained within the cluster.

Essentially, it is the way you can tell the cluster:

> I want my application to always be available, with a specific number of replicas.

#### Features
* Automatic failure handling (restart of failed pods)
* Horizontal scalability (more replicas as demand increases)
* Stabilty (rolling updates[^rollingUpdateDefinition] and rollbacks[^rollbackDefinition])

### Service

{{< alert "circle-info">}}
One of the characteristics of pods is that they are ephemeral. This means they can disappear or be replaced at any moment
{{< /alert >}}

When a **deployment** is used to dynamically create, delete, or replace pods, there is no straightforward way to know how many pods are active or how many are in a healthy state. This is why **services** become a powerful tool for exposing an application running across one or more pods in a cluster, even when those pods are running on different nodes.

## Creating a Deployment

Next, we will create a deployment named `hello-app-deployment`, which will be configured to start a single pod consisting of a container running Google’s hello-app image. We will also define CPU and RAM limits, as well as expose port 8080 on each pod.

We create a file named `hello-app-deployment.yaml` with the following content:

{{< codeimporter url="https://gist.githubusercontent.com/jortiz3109/0eaaa734fbcc062fcbf7f7d9cfc3a3ca/raw/fbd8713ea1137b11e9dde6d4c735a812df1da52b/hello-app-deployment.yaml" type="yaml" >}}


{{< highlight yaml "linenos=inline, hl_lines=5-6">}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app-deployment
  labels:
    app: hello
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
        - name: hello-app
          image: gcr.io/google-samples/hello-app:1.0
          ports:
            - containerPort: 8080
          resources:
            requests:
              cpu: "250m"
              memory: "64Mi"
            limits:
              cpu: "250m"
              memory: "128Mi"
{{< /highlight >}}

{{< alert "circle-info">}}
Take note of lines 5 and 6; we will use this data later to associate the service with this deployment.
{{< /alert >}}

Apply the changes
{{< highlight bash "hl_lines=2">}}
# kubectl apply -f hello-app-deployment.yaml
deployment.apps/hello-app-deployment created
{{< /highlight >}}

Check the status of the deployment
{{< highlight bash "hl_lines=3">}}
# kubectl get deployments
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
hello-app-deployment   1/1     1            1           30m
{{< /highlight >}}

{{< alert "circle-info">}}
n the output of the previous command, in the `READY` column, we can see that the number of replicas is 1.
{{< /alert >}}

We can also get detailed information about the deployment using the `describe` command
{{< highlight bash "hl_lines=8">}}
# kubectl describe deployment hello-app-deployment
Name:                   hello-app-deployment
Namespace:              default
CreationTimestamp:      Wed, 04 Jun 2025 18:07:05 -0500
Labels:                 app=hello
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=hello
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=hello
  Containers:
   hello-app:
    Image:      us-docker.pkg.dev/google-samples/containers/gke/hello-app:1.0
    Port:       8080/TCP
    Host Port:  0/TCP
    Limits:
      CPU:     250m
      memory:  128Mi
    Requests:
      CPU:         250m
      memory:      64Mi
    Environment:   <none>
    Mounts:        <none>
  Volumes:         <none>
  Node-Selectors:  <none>
  Tolerations:     <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hello-app-6d4c4c8966 (1/1 replicas created)
Events:          <none>
{{< /highlight >}}

Let's slightly modify the file `hello-app-deployment.yaml` to increase the number of replicas to 3.

{{< highlight yaml "linenos=inline, hl_lines=16">}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app-deployment
  labels:
    app: hello
spec:
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      replicas: 3
      containers:
      - name: hello-app
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
        resources:
            requests:
              CPU: "250m"
              memory: "64Mi"
            limits:
              CPU: "250m"
              memory: "128Mi"
{{< /highlight >}}

Apply the changes
{{< highlight bash "hl_lines=2">}}
# kubectl apply -f hello-app-deployment.yaml
deployment.apps/hello-app-deployment configured
{{< /highlight >}}

Check the status of the deployment.
As we can see, the value in the `READY` column changed from `1/1` to `3/3`.
{{< highlight bash "hl_lines=3">}}
# kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
hello-app   3/3     3            3           44m
{{< /highlight >}}

We can also see the created pods
{{< highlight bash "hl_lines=3-5">}}
# kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-app-deployment-6d4c4c8966-7rx67   1/1     Running   0          71m
hello-app-deployment-6d4c4c8966-sjclg   1/1     Running   0          71m
hello-app-deployment-6d4c4c8966-wdzsb   1/1     Running   0          115m
{{< /highlight >}}

## Defining a Service

We have successfully created a deployment, but how do we access it? We could access it using port forwarding; however, we will expose our application using a service configured as follows:

> Our service will group the pods that have the `app` label set to `hello`, and it will also expose port 8000 externally to access the `hello-app-deployment`.

We create a file named `hello-app-service.yaml` with the following content:

{{< highlight yaml "linenos=inline, hl_lines=10-13">}}
apiVersion: v1
kind: Service
metadata:
  name: hello-app-service
  labels:
    app: hello
spec:
  selector:
    app: hello
  ports:
    - port: 8000
      targetPort: 8080
  type: LoadBalancer
{{< /highlight >}}

{{< alert>}}
Note: depending on the tool you used to install your Kubernetes cluster, you may need to choose a different service type than `LoadBalancer`
{{< /alert >}}

Apply the changes
{{< highlight bash "hl_lines=2">}}
# kubectl apply -f Kubernetes/hello/hello-app-service.yaml
service/hello-app-service created
{{< /highlight >}}

Check the status of the deployment
{{< highlight bash "hl_lines=3">}}
# kubectl get services
NAME                TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-app-service   LoadBalancer   10.96.175.173   10.89.2.5     8000:30513/TCP   40s
{{< /highlight >}}

Finally, we access the service using the external IP that has been assigned to it.

{{< figure src="hello-app-service-exposed.png" alt="Service for the hello application exposed on port 8000">}}

[^rollingUpdateDefinition]: Allows deployments to be updated with zero downtime by incrementally replacing old Pods with new ones.
https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/

[^rollbackDefinition]: Allows reverting a Pod or group of Pods to a previous state.
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment