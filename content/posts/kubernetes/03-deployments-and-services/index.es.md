---
title: "Despliegues y Servicios"
slug: "deployments-and-services"
date: 2025-06-02
tags: ["containers", "cloud", "PAAS", "OCI"]
categories: ["Kubernetes"]
series: ["Curso de Kubernetes"]
showTableOfContents: true
summary: En este artículo exploraremos qué son, cómo funcionan y por qué son piezas fundamentales en cualquier arquitectura sobre Kubernetes :nerd_face:.
featureimagecaption: Foto de [orbtal media](https://unsplash.com/es/@orbtalmedia) en [Unsplash](https://unsplash.com)
draft: false
enableEmoji: true
---

## Introducción

Kubernetes se ha convertido en el estándar de facto para la orquestación de contenedores. Sin embargo, para muchos desarrolladores los conceptos de despliegues (deployments) y servicios (services) pueden resultar confusos al principio.

## Conceptos básicos
### Despliegue

Un **deployment** en Kubernetes es un conjunto de instrucciones que define cómo deben ser creados, ejecutados y mantenidos un grupo de pods dentro del cluster.

Básicamente es la forma en la que puedes indicarle al cluster:

> Quiero que mi aplicación siempre esté disponible, con un número determinado de réplicas.

#### Características
* Manejo automático de fallos (reinicio de pods caídos)
* Escalabilidad horizontal (a mayor demanda, mayor número de réplicas)
* Estabilidad (rolling updates[^rollingUpdateDefinition] y rollbacks[^rollbackDefinition])

### Servicio

{{< alert "circle-info">}}
Una de las características de los pods es que son efímeros. Esto significa que, en cualquier momento, pueden desaparecer o ser reemplazados por otro.
{{< /alert >}}

Cuando se usa un **deployment** para crear/eliminar/reemplazar pods de manera dinámica, no existe una manera sencilla para saber cuantos pods se encuentran activos y cuantos de ellos se encuentran en buen estado de salud. Es por esto que los **services** se convierten en una poderosa herramienta para exponer una aplicación que corre atraves de uno o mas pods en un cluster incluso cuando dichos pods son ejecutados en diferentes nodos.

## Creando un despliegue

A continuación, vamos a crear un deployment con el nombre `hello-app-deployment`, el cual parametrizaremos para iniciar un único pod, conformado por un contenedor corriendo la imagen `hello-app` de Google. También definiremos unos límites de CPU y RAM, así como la apertura del puerto 8080 en cada pod.

Creamos un archivo `hello-app-deployment.yaml` con el siguiente contenido:

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
Tome nota de las líneas 5 y 6, más adelante usaremos este dato para asociar el servicio a este despliegue.
{{< /alert >}}

Aplicamos los cambios
{{< highlight bash "hl_lines=2">}}
# kubectl apply -f hello-app-deployment.yaml
deployment.apps/hello-app-deployment created
{{< /highlight >}}

Verificamos el estado del deployment
{{< highlight bash "hl_lines=3">}}
# kubectl get deployments
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
hello-app-deployment   1/1     1            1           30m
{{< /highlight >}}

{{< alert "circle-info">}}
En la salida del comando anterior, en la columna `READY` podemos ver que el número de réplicas es de 1.
{{< /alert >}}

También podemos obtener información detallada sobre el deployment con el comando `describe`
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

Vamos a modificar un poco el archivo `hello-app-deployment.yaml` para incrementar el número de réplicas a 3.

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

Aplicamos los cambios
{{< highlight bash "hl_lines=2">}}
# kubectl apply -f hello-app-deployment.yaml
deployment.apps/hello-app-deployment configured
{{< /highlight >}}

Verificamos el estado del deployment.
Tal como se puede ver el valor en la columna `READY` cambió de `1/1` a `3/3`
{{< highlight bash "hl_lines=3">}}
# kubectl get deployments
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
hello-app   3/3     3            3           44m
{{< /highlight >}}

También podemos ver los pods creados
{{< highlight bash "hl_lines=3-5">}}
# kubectl get pods
NAME                                    READY   STATUS    RESTARTS   AGE
hello-app-deployment-6d4c4c8966-7rx67   1/1     Running   0          71m
hello-app-deployment-6d4c4c8966-sjclg   1/1     Running   0          71m
hello-app-deployment-6d4c4c8966-wdzsb   1/1     Running   0          115m
{{< /highlight >}}

## Definiendo un servicio

Ya hemos creado exitosamente un despliegue, pero ¿cómo accedo a él?, podriamos acceder a él usando el reenvío de puertos; sin embargo, vamos a publicar nuestra aplicación usando un servicio, el cual configuraremos de la siguiente manera:

> Nuestro servicio agrupará los pods que tengan la etiqueta `app` con el valor de `hello`, también expondrá el puerto 8000 al exterior para acceder al despliegue `hello-app-deployment`.

Creamos un archivo `hello-app-service.yaml` con el siguiente contenido:

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
Tenga en cuenta que: de acuerdo a la herramienta elegida por usted para instalar su cluster de kubernetes. Es muy probable que deba seleccionar un tipo de servicio diferente a `LoadBalancer`
{{< /alert >}}

Aplicamos los cambios
{{< highlight bash "hl_lines=2">}}
# kubectl apply -f Kubernetes/hello/hello-app-service.yaml
service/hello-app-service created
{{< /highlight >}}

Verificamos el estado del deployment
{{< highlight bash "hl_lines=3">}}
# kubectl get services
NAME                TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-app-service   LoadBalancer   10.96.175.173   10.89.2.5     8000:30513/TCP   40s
{{< /highlight >}}

finalmente accedemos al servicio usando la ip externa que ha sido asignada al mismo.

{{< figure src="hello-app-service-exposed.png" alt="Servicio para la aplicación hello expuesta en el puerto 8000">}}

[^rollingUpdateDefinition]: Permite que la actualización de los despliegues se realice sin tiempo de inactividad, actualizando incrementalmente las instancias de Pods con otras nuevas. https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/

[^rollbackDefinition]: Permite devolver un pod o grupo de pods a un estado anterior.
https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment