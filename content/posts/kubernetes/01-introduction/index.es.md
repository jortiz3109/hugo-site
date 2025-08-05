---
title: "Introducción a Kubernetes"
slug: "introduction"
date: 2024-05-06
tags: ["containers", "cloud", "PAAS", "OCI"]
categories: ["Kubernetes"]
series: ["Curso de Kubernetes"]
draft: false
showTableOfContents: true
summary: Kubernetes, también conocida como ***k8s*** o ***kube***, es una plataforma de orquestación de contenedores diseñada para automatizar la implementación, la gestión y el escalado de aplicaciones en contenedores.
---

Kubernetes fue desarrollado por primera vez por ingenieros de Google antes de ser de código abierto en 2014. Kubernetes significa timonel o piloto en griego, de ahí el timón en el logotipo de Kubernetes.
## Conceptos básicos
### Contenedor
Podría definirse contenedor como una unidad de computación que empaqueta el código fuente y los requerimientos de software necesarios para distribuir y ejecutar aplicaciones o servicios.
Los contenedores aprovechan una forma de virtualización del sistema operativo anfritrión que permite compartir recursos, aisla procesos y controla la cantidad de cpu, memoria y disco a la que pueden acceder los mismos.
La principal ventaja de los contenedores es que no necesitan incluir un sistema operativo en cada instancia; en su lugar aprovechan una forma de virtualización del sistema operativo anfitrión para acceder a recursos como CPU, memoria, disco y red para ejecutar aplicaciones o servicios. Lo que los hace extremadamente pequeños, rápidos y portables.
 
#### Beneficios

##### Eficiencia
Los contenedores requieren menos recursos del sistema anfitrión, por lo que suelen ser mas eficientes que otros tipos de virtualización gracias a que no requieren la instalación de un nuevo sistema operativo para funcionar.

##### Portabilidad
Gracias a que empaquetan tanto código fuente como requerimientos de software (lenguajes de programación, librerías, etc) los contenedores pueden ponerse en marcha fácilmente. Tanto en equipos de desarrolladores, como en centros de datos y en múltiples sistemas operativos.

##### Interoperabilidad
Los contenedores son especialmente útiles para los equipos de desarrollo, pues liberan al desarrollador de la instalación de dependencias y entornos de desarrollo, pues empaquetan todo lo necesario para instalar su entorno local.

### POD
un POD es un conjunto de uno o más contenedores que comparten los mismos recursos informáticos. Es considerado el componente más pequeño de un cluster kubernetes.
Generalmente un POD se compone de un único contenedor, pero en algunos casos avanzados puede albergar más de un contenedor.

## Componentes principales de un cluster Kubernetes
### Control plane
{{< figure
    src="control-plane.png"
    alt="Diagrama de flujo de control plane"
    >}}
Un cluster de kubernetes consiste en una o más máquinas llamados nodos, en los cuales se ejecutan las aplicaciones empaquetadas. Dichos nodos son gestionados mediante una capa de orquestación conocida como **control plane**.

Sus componentes son los responsables de asegurar el buen funcioamiento del cluster respondiendo a los eventos del mismo (por ejemplo: iniciar un nuevo pod cuando la cantidad de réplicas no es la deseada).

{{< alert "circle-info" >}}
Los componentes de control plane tipicamente se ejecutan en el mismo nodo separados del espacio de nombres de los contenedores de usuario.
{{< /alert >}}

#### kube-api-server
Proporciona una API que permite controlar el cluster de Kubernetes. Es responsable de gestionar las solicitudes externas e internas, además de determinar si una solicitud es válida y luego procesarla. Se puede acceder a la API a través de la interfaz de línea de comandos de [kubectl](https://kubernetes.io/docs/reference/kubectl/) y mediante peticiones REST.

#### etcd
Una base de datos de clave => valor que contiene información sobre el estado y la configuración del clúster.

#### scheduler
Responsable de monitorear en búsqueda de pods recién creados o no asignados y de asignarles a los nodos que mejor se ajusten a sus requerimientos.

Algunos factores que se tienen en cuenta para seleccionar el mejor nodo incluyen:
* Política de recursos.
* Restricciones de hardware y software.
* Localización de los datos.

#### kube-controller-manager
Ejecuta los procesos de los controladores.
{{< alert "circle-info">}}
En Kubernetes, los controladores son estructuras de control que monitorean el estado del clúster y luego realizan o solicitan cambios cuando sea necesario buscando mantener el estado actual del cluster lo más cercano posible al estado deseado.
{{< /alert>}}

Cada controlador es un binario compilado que se encarga de ejecutar un proceso específico. Hay muchos tipos diferentes de controladores, algunos ejemplos de ellos son:

**Node Controller**: Responsable de monitorizar el estado de los nodos y responder cuando estos se caen.

**Job Controller**: Busca objetos de trabajo que representan tareas únicas y luego crea pods para ejecutar esas tareas hasta su finalización.

**EndpointSlice Controller**: Se encarga de proporcionar un enlace entre los servicios y los pods.

#### cloud-controller-manager
Un componente de control-plane que incorpora una lógica de control específica de la nube. Por ejemplo: cloud-controller-manager permite a proveedores de servicios en la nube integrar sus plataformas con nuestro cluster de kubernetes.

### Kubelet
Kubelet es el agente encargado de gestionar cada nodo de acuerdo a un conjunto de especificaciones llamadas **PodSpecs**, garantizando que los contenedores descritos allí se encuentren funcionando y en buen estado. Entre sus responsabilidades encontramos:
* Garantizar que los contenedores se ejecuten en el pod correcto.
* Monitorizar que los contenedores se ejecuten correctamente.
* Detener los contenedores cuando le sea requerido.

Hasta aquí hemos visto una introducción a kubernetes, algunos conceptos básicos y sus componentes principales; en posteriores entregas de esta serie hablaremos sobre cómo instalar kubernetes en nuestro entorno local, crearemos nuestro primer pod y verificaremos su correcto funcionamiento.