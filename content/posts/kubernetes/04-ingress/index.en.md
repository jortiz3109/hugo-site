---
title: "How to access HTTP web services using Ingress"
slug: "ingress"
date: 2025-07-01
tags: ["containers", "cloud", "PAAS", "OCI"]
categories: ["Kubernetes"]
series: ["Kubernetes course"]
showTableOfContents: true
summary: In this article, we will explore how to use Ingress to access HTTP web services, even when those services are deployed across multiple pods.
featureimagecaption: Photo by [Fredick F.](https://unsplash.com/@blackspeaker93) on [Unsplash](https://unsplash.com)
enableEmoji: true
---

Once services are deployed in a Kubernetes cluster, it becomes necessary to access them remotely. There are several alternatives for this, such as the Gateway API[^gatewayAPIDefinition] (which we will cover in a future article). However, the most popular option today is Ingress[^ingressDefinition]: a controller that manages access to HTTP services within a Kubernetes cluster.

## What is Ingress
Ingress is a Kubernetes resource that manages external access to HTTP services within the cluster. It acts as an entry point, using routing rules to direct incoming traffic to the cluster’s internal services.

### Features
* Multiple implementations (NGINX, APISIX, KONG)
* Virtual hosts support
* Load balancing
* TLS encryption

### How It Works
Ingress operates based on a set of rules (domains, hosts, and paths). These rules are evaluated by an Ingress Controller, which redirects incoming traffic to the appropriate services within the cluster. Ingress can also be configured to encrypt traffic using TLS.

<div style="background-color: #fff; border-radius: 10px; padding: 5px;">
{{< mermaid >}}
flowchart LR;
C(["Client"])
I("Ingress")
FE["Frontend"]
BE["Backend"]
P1[Pod]
P2[Pod]
P3[Pod]
P4[Pod]

C -- Load balancer --> I
I -- routing rule (host: host.example.com)--> FE
I -- routing rule (path: host.example.com/api)--> BE
FE --> P1
FE --> P2

BE --> P3
BE --> P4
{{< /mermaid >}}
</div>

## Exposing a Service Using Ingress
### Prerequisites
To expose a service using Ingress, it is necessary to install an Ingress Controller in your cluster. There are many Ingress Controllers available, which you can explore at the following link: [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/).

### Installing ingress-nginx
The Ingress Controller I will use in this article is [ingress-nginx](https://kubernetes.github.io/ingress-nginx/). While there are several ways to install it, I consider the easiest one to be using the [helm](https://helm.sh/) package manager:

{{< highlight bash >}}
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
{{< /highlight >}}

{{< alert "circle-info">}}
This command is idempotent:

* If the Ingress Controller has not been installed, it will be installed.
* If the Ingress Controller is already installed, it will be updated.
{{< /alert >}}

#### Ports
Ingress-nginx uses ports 80, 443, and 8443.

* Port **8443**: It is used for the ingress-nginx admission controller[^aCDefinition].
* Port **80**: It is used to receive HTTP traffic from outside the cluster.
* Port **443**: It is used to receive HTTPS traffic from outside the cluster.

### Defining an Ingress Resource
Before creating our Ingress resource, we will deploy a service and a deployment:

First, we will create a file `hello-app-deployment.yaml`:

{{< highlight yaml >}}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app-deployment
  labels:
    app: hello
spec:
  replicas: 2
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

Apply the changes
{{< highlight bash >}}
# kubectl apply -f hello-app-deployment.yaml
deployment.apps/hello-app-deployment created
{{< /highlight >}}

Next, we create the file `hello-app-service.yaml`:

{{< highlight yaml >}}
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
    - port: 8080
      targetPort: 8080
  type: ClusterIP
{{< /highlight >}}

Apply the changes
{{< highlight bash >}}
# kubectl apply -f Kubernetes/hello/hello-app-service.yaml
service/hello-app-service created
{{< /highlight >}}

We can verify that both resources (deployment and service) were created successfully:
{{< highlight bash >}}
# kubectl get svc,deploy,pod -l app=hello 
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/hello-app-service   ClusterIP   10.100.83.156   <none>        8080/TCP   2m22s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-app-deployment   2/2     2            2           2m22s

NAME                                        READY   STATUS    RESTARTS   AGE
pod/hello-app-deployment-6d9b84546c-47cbr   1/1     Running   0          4s
pod/hello-app-deployment-6d9b84546c-n8sws   1/1     Running   0          2m22s
{{< /highlight >}}

We can see that the requested resources have been successfully created.

Once our service has been deployed, we will create the file `hello-app-ingress.yaml`:

{{< highlight yaml >}}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-app-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: hello-app.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: hello-app-service
            port:
              number: 8080
{{< /highlight >}}

Apply the changes
{{< highlight bash "hl_lines=2">}}
# kubectl apply -f hello-app-ingress.yaml
ingress.networking.k8s.io/hello-app-ingress created
{{< /highlight >}}

We verify that the Ingress resource was created successfully:

{{< highlight bash >}}
# kubectl get ingress --field-selector metadata.name=hello-app-ingress
NAME                CLASS   HOSTS                     ADDRESS          PORTS   AGE
hello-app-ingress   nginx   hello-app.ingress.local   192.168.68.210   80      4m17s
{{< /highlight >}}

We can obtain detailed information about the Ingress resource using the `describe` command.

{{< highlight bash >}}
# kubectl describe ingress hello-app-ingress 
Name:             hello-app-ingress
Labels:           <none>
Namespace:        default
Address:          192.168.68.210
Ingress Class:    nginx
Default backend:  <default>
Rules:
  Host                     Path  Backends
  ----                     ----  --------
  hello-app.ingress.local  
                           /   hello-app-service:8080 (10.244.1.113:8080,10.244.1.114:8080)
Annotations:               <none>
Events:
  Type    Reason  Age                    From                      Message
  ----    ------  ----                   ----                      -------
  Normal  Sync    4m54s (x2 over 5m25s)  nginx-ingress-controller  Scheduled for sync
{{< /highlight >}}

Finally, if we access the URL `http://hello-app.ingress.local/`, we should be able to reach the `hello-app-service`.

{{< figure src="hello-app-ingress.png" alt="hello-app-service service accessible through Ingress">}}

## Conclusions

In conclusion, Ingress is a fundamental tool for managing external traffic to our applications in Kubernetes. Its ability to route HTTP/S requests, apply rules based on paths or domains, and leverage controllers like NGINX makes it a powerful and flexible solution. Understanding its functionality and basic configuration enables us to build more organized, secure, and scalable architectures.

## Additional resources

If you want to explore this tool further, you can visit the following links:

* [Ingress official documentation](https://kubernetes.io/docs/concepts/services-networking/ingress/)
* [Ingress-NGINX official documentation](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/)
* [Source code repository with the examples used in this article](https://github.com/jortiz3109/kubernetes-course/tree/main/04-ingress)

[^gatewayAPIDefinition]: Gateway API is a family of APIs that enable dynamic infrastructure provisioning and advanced traffic routing.
https://kubernetes.io/docs/concepts/services-networking/gateway/

[^ingressDefinition]: Ingress allows mapping traffic to different backends based on rules that can be defined through the Kubernetes API.
https://kubernetes.io/docs/concepts/services-networking/ingress/

[^aCDefinition]: An admission controller is a piece of code that intercepts requests to the Kubernetes API before they reach the targeted resource, but after the request has been authenticated and authorized.