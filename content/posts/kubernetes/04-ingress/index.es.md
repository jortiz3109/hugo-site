---
title: "Cómo acceder a servicios web HTTP usando Ingress"
slug: "ingress"
date: 2025-07-01
tags: ["containers", "cloud", "PAAS", "OCI"]
categories: ["Kubernetes"]
series: ["Curso de Kubernetes"]
showTableOfContents: true
summary: En este artículo exploraremos como usar Ingress para acceder a servicios web HTTP. Incluso cuando dichos servicios se encuentran desplegados en múltiples PODs.
featureimagecaption: Foto de [Fredick F.](https://unsplash.com/@blackspeaker93) en [Unsplash](https://unsplash.com)
enableEmoji: true
---

Una vez desplegados los servicios en un cluster de Kubernetes; se hace necesario acceder a ellos de manera remota. Para ello, existen múltiples alternativas como Gateway API[^gatewayAPIDefinition] *del cual hablarémos en un artículo próximo*. Sin embargo la opción mas popular hoy en día es Ingress[^ingressDefinition]: Un controlador que permite gestionar el acceso a servicios HTTP en un cluster de Kubernetes. 

## Qué es Ingress
Ingress es un recurso de Kubernetes que gestiona el acceso externo a los servicios HTTP del clúster. Funciona como un punto de acceso que funciona a partir de reglas de enrutamiento para dirigir el tráfico entrante a los servicios internos del clúster.

### Caraterísticas
* Múltiples implementaciones (NGINX, APISIX, KONG)
* Hosts virtuales.
* Balanceo de cargas.
* Cifrado TLS.

### Cómo funciona
Ingress funciona a partir de un conjunto de reglas (dominios, hosts y paths), dichas reglas son evaluadas por un Controlador de Ingress, el cual redirige el tráfico entrante hacia sus correspondientes servicios al interior del cluster. Finalmente Ingress puede ser configurado para cifrar el tráfico usando TLS.

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

## Exponiendo un servicio usando Ingress
### Prerequisitos
Para exponer un servicio usando Ingress es necesario instalar en nuestro cluster un controlador de Ingress. Existen multitud de controladores de Ingress los cuales puedes consultar en este enlace: [Controladores de Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/).

### Instalando ingress-nginx
El controlador de Ingress que usaré en este artículo es [ingress-nginx](https://kubernetes.github.io/ingress-nginx/). Si bien existen varias maneras de instalarlo, considero que la mas fácil es usando el administrador de paquetes [helm](https://helm.sh/):

{{< highlight bash >}}
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
{{< /highlight >}}

{{< alert "circle-info">}}
Este comando es idempotente:

* Si el controlador de Ingress no ha sido instalado, se instalará,
* Si el controlador de Ingress ya ha sido instalado, entonces se actualizará.
{{< /alert >}}

#### Puertos
Ingress-nginx usa los puertos 80, 443 y 8443

* Puerto **8443**: Se utiliza para el controlador de admisión[^aCDefinition] ingress-nginx.
* Puerto **80**: Se utiliza para recibir el tráfico HTTP desde el exterior.
* Puerto **443**: Se utiliza para recibir el tráfico HTTPS desde el exterior.

### Definiendo un recurso Ingress
Antes de crear nuestro recurso de Ingress, vamos a desplegar un servicio y un despliegue:

Primero, creamos un archivo `hello-app-deployment.yaml`:

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

Aplicamos los cambios
{{< highlight bash >}}
# kubectl apply -f hello-app-deployment.yaml
deployment.apps/hello-app-deployment created
{{< /highlight >}}

Luego, creamos el archivo `hello-app-service.yaml`:

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

Aplicamos los cambios
{{< highlight bash >}}
# kubectl apply -f Kubernetes/hello/hello-app-service.yaml
service/hello-app-service created
{{< /highlight >}}

Podemos verificar que ambos recursos (despliegue y servicio) fueron creados correctamente:
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

Podemos ver que se han creado exitosamente los recursos solicitados.

Una vez desplegado nuestro servicio, vamos a crear el archivo `hello-app-ingress.yaml`:

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

Aplicamos los cambios
{{< highlight bash "hl_lines=2">}}
# kubectl apply -f hello-app-ingress.yaml
ingress.networking.k8s.io/hello-app-ingress created
{{< /highlight >}}

Verificamos que el recurso de Ingress fue creado exitosamente:

{{< highlight bash >}}
# kubectl get ingress --field-selector metadata.name=hello-app-ingress
NAME                CLASS   HOSTS                     ADDRESS          PORTS   AGE
hello-app-ingress   nginx   hello-app.ingress.local   192.168.68.210   80      4m17s
{{< /highlight >}}

Podemos obtener información detallada del recurso ingress con el comando `describe`

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

Finalmente: si accedemos a la url `http://hello-app.ingress.local/` deberiamos tener acceso al servicio `hello-app-service`.

{{< figure src="hello-app-ingress.png" alt="Servicio hello-app-service accessible através de Ingress">}}

## Conclusiones

En conclusión, Ingress es una herramienta fundamental para gestionar el tráfico externo hacia nuestras aplicaciones en Kubernetes. Su capacidad para enrutar peticiones HTTP/S, aplicar reglas basadas en rutas o dominios, y aprovechar controladores como NGINX, lo convierte en una solución potente y flexible. Comprender su funcionamiento y configuración básica nos permite construir arquitecturas más organizadas, seguras y escalables.

## Recursos adicionales

Si deseas profundizar en esta herramienta puedes acceder a los siguientes enlaces:

* [Documentación oficial de Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
* [Documentación oficial de Ingress-NGINX](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/)
* [Repositorio de código con los ejemplos usados en este artículo](https://github.com/jortiz3109/kubernetes-course/tree/main/04-ingress)

[^gatewayAPIDefinition]: Gateway API es una familia APIs que permiten aprovisionamiento dinámico de infraestructura y enrutamiento de tráfico avanzado.
https://kubernetes.io/docs/concepts/services-networking/gateway/

[^ingressDefinition]: Ingress permite mapear tráfico hacia diferentes backends basado en reglas que pueden ser definidas atraves del API de Kubernetes.
https://kubernetes.io/docs/concepts/services-networking/ingress/

[^aCDefinition]: Un controlador de admisión es una pieza de código que intercepta las solicitudes al API de Kubernetes antes de alcanzar el recurso solicitado, pero después de que la solicitud haya sido autenticada y autorizada.