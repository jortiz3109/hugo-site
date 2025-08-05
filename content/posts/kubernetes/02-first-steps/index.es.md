---
title: "Primeros pasos con Kubernetes"
slug: "first-steps"
date: 2024-06-03
tags: ["containers", "cloud", "PAAS", "OCI"]
categories: ["Kubernetes"]
series: ["Curso de Kubernetes"]
showTableOfContents: true
summary: Ya que tenemos una idea clara sobre qué es Kubernetes, vamos a ejecutar nuestro primer POD!.
featureimagecaption: Foto de [orbtal media](https://unsplash.com/es/@orbtalmedia) en [Unsplash](https://unsplash.com)
---

## Instalación
Instalar y ejecutar [kubernetes](https://kubernetes.io/) en nuestro entorno local puede ser todo un reto. Sin embargo existen algunas alternativas para simplificar ésta tarea:
### Docker desktop
Docker Desktop es una aplicación todo en uno para gestionar contenedores lista para usar que ofrece a desarrolladores y equipos desarrollo un conjunto de herramientas para crear, compartir y ejecutar contenedores en cualquier entorno.

Para instalar kubernetes usando [Docker Desktop](https://www.docker.com/products/docker-desktop/) debes seguir los siguientes pasos:

{{< figure src="docker-desktop-installation.png" title="Instalación de Kubernetes usando Docker Desktop" caption="Pantalla de configuración de Kubernetes en Docker Desktop" >}}

1. En el panel de control de Docker, seleccione Configuración.
2. Seleccione Kubernetes en la barra lateral izquierda.
3. Seleccione la casilla de verificación **Habilitar Kubernetes**.
4. Seleccione Aplicar y reiniciar para guardar la configuración y, a continuación, seleccione Instalar para confirmar.
    > Esto instanciará las imágenes necesarias para ejecutar el servidor Kubernetes como contenedores e instalará el comando /usr/local/bin/kubectl en su máquina.

### Minikube
{{< alert "circle-info">}}
Para instalar Minikube debes contar con una solución de virtualización previamente instalada cómo [Docker](https://www.docker.com/) o [Virtualbox](https://www.virtualbox.org/).
{{< /alert >}}

[Minikube](https://minikube.sigs.k8s.io/docs/) es una herramienta de código abierto que mediante el uso de virtualización; permite disponer de un entorno local kubernetes.

#### Instalar Minikube en macOS
Puedes instalar Minikube en macOS usando [Homebrew](https://brew.sh/)
1. Ejecutamos el comando de instalación
{{< highlight bash >}}
brew install minikube
{{< /highlight >}}

2. Inicia el cluster con el comando:
{{< highlight bash >}}
minikube start
{{< /highlight >}}

#### Instalar Minikube en Linux

1. Ejecutamos las instrucciones de instalación
{{< highlight bash >}}
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64
{{< /highlight >}}

2. Inicia el cluster con el comando:
{{< highlight bash >}}
minikube start
{{< /highlight >}}

{{< alert "circle-info">}}
Para más opciones de instalación consulta la [documentación oficial de minikube](https://minikube.sigs.k8s.io/docs/start/)
{{< /alert >}}

## Primeros pasos con Kubernetes
Ahora que hemos instalado una instancia local de kubernetes. Es momento de crear y ejecutar nuestro primer POD.
### Kubectl
Kubectl es una herramienta de linea de comandos que permite ejecutar opraciones contra un cluster de Kubernetes.
Con Kubectl podemos desplegar aplicaciones, listar, administrar, escalar recursos, ver logs y más.

{{< alert "circle-info">}}
Kubectl se comunica con el cluster de Kubernetes a través de [Kube-API-Server]({{< ref "/posts/kubernetes/01-introduction/#kube-api-server" >}}) (componente de [Control plane]({{< ref "/posts/kubernetes/01-introduction/#control-plane" >}}))
{{< /alert >}}

#### Instalación
##### Instalar Kubectl en macOS
Puedes instalar Kubectl en macOS usando [Homebrew](https://brew.sh/)

1. Ejecutamos el comando de instalación
{{< highlight bash >}}
brew install kubectl
{{< /highlight >}}
o
{{< highlight bash >}}
brew install kubernetes-cli
{{< /highlight >}}

2. Verificamos la versión de Kubectl instalada
{{< highlight bash >}}
kubectl version --client
{{< /highlight >}}

##### Instalar Kubectl en Linux

1. Descargar la última versión estable de Kubectl
{{< highlight bash >}}
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
{{< /highlight >}}

2. Verificar el binario descargado
    1. Descargar el archivo con la suma de verificación
    {{< highlight bash >}}
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
    {{< /highlight >}}

    2. Validar el binario de kubectl contra la el archivo con la suma de verificación
    {{< highlight bash >}}
    echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
    --
    kubectl: OK
    {{< /highlight >}}

3. Instalar kubectl
{{< highlight bash >}}
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
{{< /highlight >}}

4. Verificamos la versión de Kubectl instalada
{{< highlight bash >}}
kubectl version --client
{{< /highlight >}}

{{< alert "circle-info">}}
Para más opciones de instalación consulta la [documentación de kubectl](https://kubernetes.io/docs/tasks/tools/)
{{< /alert >}}

#### Cofiguración
Toda la configuración referente a [Kubernetes](https://kubernetes.io/) y kubectl se almacena en el archivo de configuración ***.kube/config***. Sin embargo kubectl te permite gestionar ciertos parámetros de configuración atraves del comando ***config***.
{{< highlight bash >}}
kubectl config --help

Available Commands:
  current-context   Display the current-context
  delete-cluster    Delete the specified cluster from the kubeconfig
  delete-context    Delete the specified context from the kubeconfig
  delete-user       Delete the specified user from the kubeconfig
  get-clusters      Display clusters defined in the kubeconfig
  get-contexts      Describe one or many contexts
  get-users         Display users defined in the kubeconfig
  rename-context    Rename a context from the kubeconfig file
  set               Set an individual value in a kubeconfig file
  set-cluster       Set a cluster entry in kubeconfig
  set-context       Set a context entry in kubeconfig
  set-credentials   Set a user entry in kubeconfig
  unset             Unset an individual value in a kubeconfig file
  use-context       Set the current-context in a kubeconfig file
  view              Display merged kubeconfig settings or a specified kubeconfig file
{{< /highlight >}}

Para efectos de éste primer acercamiento, configuraremos el contexto[^contextDefinition] sobre el que vamos a trabajar:

1. Listar los contextos disponibles:
{{< highlight bash >}}
kubectl config get-contexts
CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
*         docker-desktop   docker-desktop   docker-desktop
          minikube         minikube         minikube         default
{{< /highlight >}}

2. Usar el contexto deseado (para efectos de esta serie se usará minikube)
{{< highlight bash >}}
kubectl config use-context minikube
Switched to context "minikube".
{{< /highlight >}}

3. Verificamos que ahora se esta usando el contexto deseado
{{< highlight bash >}}
kubectl config current-context
minikube
{{< /highlight >}}

> En la medida en que avancemos en esta serie, iremos ahondando en otras opciones de configuración como: Creacion de clusters, contextos y usuarios.

#### Uso
Una vez hemos seleccionado exitosamente el contexto sobre el que vamos a trabajar es hora de correr un primer pod.
##### Listar objetos
Para listar objetos en kubernetes tenemos a dispocisión el comando *get*

**pods**
{{< highlight bash >}}
kubectl get pods
No resources found in default namespace.
{{< /highlight >}}

**services**
{{< highlight bash >}}
kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   10d
{{< /highlight >}}

**deployments**
{{< highlight bash >}}
kubectl get deployments
No resources found in default namespace.
{{< /highlight >}}

> Existen otros tipos de recursos en kubernetes (*ConfigMap*, *CronJob*, *Job*, *Secret*, etc) sobre los cuales ahondaremos en futuras entregas de esta serie.

##### Crear objetos
Existen dos formas de crear objetos[^objectDefinition] en un cluster kubernetes:

**Mediante los comandos de kubectl**

1. Creamos el pod con el comando *run*
{{< highlight bash "hl_lines=2">}}
kubectl run nginx-k8s --image=nginx
pod/nginx-k8s created
{{< /highlight >}}

2. Verificamos el estado del pod
{{< highlight bash "hl_lines=3">}}
kubectl get pod nginx-k8s
NAME        READY   STATUS    RESTARTS   AGE
nginx-k8s   1/1     Running   0          4s
{{< /highlight >}}

**Mediante un fichero de manifiesto**

Gestionar los recursos en un clúster de kubernetes mediante los comandos de kubectl puede ser bastante complejo. Para ello existe la posibilidad de definir los diferentes tipos de recursos de kubernetes mediante un archivo de manifiesto escrito en YAML[^YamlSite], dichos cambios pueden ser aplicados mediante el comando ***apply*** de kubectl.

1. Creamos un archivo ***nginx-pod.yaml*** con el siguiente contenido
{{< highlight yaml >}}
apiVersion: v1
kind: Pod
metadata:
  name: nginx-k8s
  labels:
    name: nginx-k8s
spec:
  containers:
  - name: nginx-k8s
    image: nginx:stable
    resources:
      limits:
        memory: "128Mi"
        cpu: "500m"
{{< /highlight >}}

2. Aplicamos los cambios
{{< highlight bash "hl_lines=2">}}
kubectl apply -f nginx-pod.yaml
pod/nginx-k8s created
{{< /highlight >}}

3. Verificamos el estado del pod
{{< highlight bash "hl_lines=3">}}
kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
nginx-k8s        1/1     Running   0          16m
{{< /highlight >}}

> Podemos ver cómo se ha creado un nuevo pod con el nombre *nginx-k8s*
##### Describir objetos
Para ver los detalles de cualquier recurso de kubernetes, usamos el comando ***describe*** de kubectl.

El comando ***describe*** permite mostrar los detalles de un recurso específico o de un grupo de recursos. Imprime una descripción detallada de los recursos seleccionados, incluidos los recursos relacionados, como eventos o controladores.

{{< highlight bash "hl_lines=8">}}
kubectl describe pod nginx-k8s
Name:             nginx-k8s
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Mon, 03 Jun 2024 12:39:49 -0500
Labels:           name=nginx-k8s
Annotations:      <none>
Status:           Running
IP:               10.244.0.5
IPs:
  IP:  10.244.0.5
Containers:
  nginx-k8s:
    Container ID:   docker://7582c54b05625add5401fe0178526a628d2eacad843a6be3fe8816b1c435d499
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:0f04e4f646a3f14bf31d8bc8d885b6c951fdcf42589d06845f64d18aec6a3c4d
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 03 Jun 2024 12:39:51 -0500
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-mvjl2 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-mvjl2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  49m   default-scheduler  Successfully assigned default/nginx-k8s to minikube
  Normal  Pulling    49m   kubelet            Pulling image "nginx"
  Normal  Pulled     49m   kubelet            Successfully pulled image "nginx" in 1.168s (1.168s including waiting). Image size: 192993703 bytes.
  Normal  Created    49m   kubelet            Created container nginx-k8s
  Normal  Started    49m   kubelet            Started container nginx-k8s
{{< /highlight >}}

> Existen otros tipos de recursos en kubernetes (*Service*, *Deployment*, *ConfigMap*, *CronJob*, *Job*, *Secret*, etc) sobre los cuales ahondaremos en futuras entregas de esta serie.
##### Editar objetos
Kubectl nos permite realizar algunas modificaciones a los recursos de kubernetes mediante un archivo de manifiesto.
 1. Creamos un archivo ***nginx-pod-edit.yaml*** con el siguiente contenido
{{< highlight yaml "hl_lines=7-8 14-16">}}
apiVersion: v1
kind: Pod
metadata:
  name: nginx-k8s
  labels:
    name: nginx-k8s
  annotations:
    desc: "This is a nginx pod created as example"
spec:
  containers:
  - name: nginx-k8s
    image: nginx:stable
    resources:
      limits:
        memory: "254Mi"
        cpu: "1000m"

{{< /highlight >}}

2. Aplicamos los cambios
{{< highlight bash "hl_lines=2">}}
kubectl apply -f nginx-pod-edit.yaml
pod/nginx-k8s configured
{{< /highlight >}}

3. Verificamos los cambios en el pod
{{< highlight bash "hl_lines=9 26-27">}}
kubectl describe pod nginx-k8s
Name:             nginx-k8s
Namespace:        default
Priority:         0
Service Account:  default
Node:             minikube/192.168.49.2
Start Time:       Mon, 03 Jun 2024 14:17:04 -0500
Labels:           name=nginx-k8s
Annotations:      desc: This is a nginx pod created as example
Status:           Running
IP:               10.244.0.9
IPs:
  IP:  10.244.0.9
Containers:
  nginx-k8s:
    Container ID:   docker://bafb513c9231b50570d4f601799661f48da8d214167ade9f92fc70d44bd693a8
    Image:          nginx:stable
    Image ID:       docker-pullable://nginx@sha256:ec881e8e2b068e388e992e0e12360ff9f93737f90eb3e346d0ceac83710383d8
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 03 Jun 2024 14:17:04 -0500
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     1
      memory:  254Mi
    Requests:
      cpu:        1
      memory:     254Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-tfk4m (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True
  Initialized                 True
  Ready                       True
  ContainersReady             True
  PodScheduled                True
Volumes:
  kube-api-access-tfk4m:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Guaranteed
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  12m   default-scheduler  Successfully assigned default/nginx-k8s to minikube
  Normal  Pulled     12m   kubelet            Container image "nginx:stable" already present on machine
  Normal  Created    12m   kubelet            Created container nginx-k8s
  Normal  Started    12m   kubelet            Started container nginx-k8s
{{< /highlight >}}
##### Eliminar objetos
Para eliminar un recurso de un clúster kubernetes, usamos el comando ***delete*** de kubectl.

**Eliminar un pod por nombre**
{{< highlight bash "hl_lines=2">}}
kubectl delete pod nginx-k8s
pod "nginx-k8s" deleted
{{< /highlight >}}

**Eliminar recursos por label**
{{< highlight bash "hl_lines=2">}}
kubectl delete pod -l name=nginx-k8s
pod "nginx-k8s" deleted
{{< /highlight >}}

> Existen otros tipos de recursos en kubernetes (*Service*, *Deployment*, *ConfigMap*, *CronJob*, *Job*, *Secret*, etc) sobre los cuales ahondaremos en futuras entregas de esta serie.
##### Obtener logs de un pod (contenedor)
En ciertas ocasiones necesitaremos acceder a los registros de log. Para esto existe el comando ***log*** de kubectl.

{{< highlight bash>}}
kubectl logs nginx-k8s
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Sourcing /docker-entrypoint.d/15-local-resolvers.envsh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2024/06/03 19:17:04 [notice] 1#1: using the "epoll" event method
2024/06/03 19:17:04 [notice] 1#1: nginx/1.26.1
2024/06/03 19:17:04 [notice] 1#1: built by gcc 12.2.0 (Debian 12.2.0-14)
2024/06/03 19:17:04 [notice] 1#1: OS: Linux 6.6.26-linuxkit
2024/06/03 19:17:04 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2024/06/03 19:17:04 [notice] 1#1: start worker processes
2024/06/03 19:17:04 [notice] 1#1: start worker process 29
2024/06/03 19:17:04 [notice] 1#1: start worker process 30
2024/06/03 19:17:04 [notice] 1#1: start worker process 31
2024/06/03 19:17:04 [notice] 1#1: start worker process 32
{{< /highlight >}}

##### Ejecutar comandos dentro de un pod
Para ejecutar comandos en los contenedores de un pod usamos el comando ***exec*** de kubernetes.
{{< highlight bash "hl_lines=2">}}
kubectl exec nginx-k8s -- date
Mon Jun  3 20:57:23 UTC 2024
{{< /highlight >}}

**Ejecutar un comando de manera interactiva**
El comando ***exec*** de kubernetes por defecto ejecutará la operación que le solicitemos, devolverá la respuesta y cerrará la ejecución del mismo. Sin embargo en ocasiones podemos requerir que la ejecución se realice de manera interactiva (como en el caso de querer acceder a las shell del contenedor) para esos casos usaremos la opción **it**.
{{< highlight bash "hl_lines=2-4">}}
kubectl exec nginx-k8s -it -- /bin/sh
# whoami
root
# exit
{{< /highlight >}}

[^contextDefinition]: Un contexto es una agrupación de parámetros de acceso bajo un nombre común. Adicional, un contexto se compone de tres parámetros: cluster, espacio de nombres y usuario.
[^objectDefinition]: Los objetos Kubernetes son entidades persistentes. Kubernetes utiliza estas entidades para representar el estado de su clúster.
[^YamlSite]: https://yaml.org/