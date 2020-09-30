---
title: Descripción de las redes de Kubernetes en un dispositivo de Azure Stack Edge Pro | Microsoft Docs
description: Se describe cómo funcionan las redes de Kubernetes en un dispositivo de Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 08/21/2020
ms.author: alkohli
ms.openlocfilehash: 001304ad6eda27db2285aaa9ad8b28929e2a04f8
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90899313"
---
# <a name="kubernetes-networking-in-your-azure-stack-edge-pro-gpu-device"></a>Redes de Kubernetes en un dispositivo Azure Stack Edge Pro con GPU

En el dispositivo de Azure Stack Edge Pro, se crea un clúster de Kubernetes al configurar el rol de proceso. Una vez creado el clúster de Kubernetes, las aplicaciones en contenedor se pueden implementar en el clúster de Kubernetes en pods. Hay distintas maneras de usar las funciones de red para los pods en el clúster de Kubernetes. 

En este artículo se describen las redes en un clúster de Kubernetes en general y específicamente en el contexto de un dispositivo de Azure Stack Edge Pro. 

## <a name="networking-requirements"></a>Requisitos de red

Este es un ejemplo de una aplicación típica de 2 niveles que se implementa en el clúster de Kubernetes.

- La aplicación tiene un front-end de servidor web y una aplicación de bases de datos en el back-end. 
- A cada pod se le asigna una dirección IP, pero estas direcciones IP pueden cambiar tras el reinicio y la conmutación por error del pod. 
- Cada aplicación se compone de varios pods, y debe haber un equilibrio de carga del tráfico entre todas las réplicas de los pods. 

![Requisitos de red de Kubernetes](./media/azure-stack-edge-gpu-kubernetes-networking/kubernetes-networking-1.png)

El escenario anterior da como resultado los siguientes requisitos de red:

 - Existe la necesidad de que el usuario de una aplicación orientada al exterior acceda a la aplicación fuera del clúster de Kubernetes mediante un nombre o una dirección IP. 
 - Las aplicaciones en el clúster de Kubernetes, por ejemplo, los pods de front-end y back-end, deben ser capaces de comunicarse entre sí.

Para resolver ambas necesidades anteriores, se introduce un servicio de Kubernetes. 


## <a name="networking-services"></a>Servicios de red

Existen dos tipos de servicios de Kubernetes: 

- **Servicio IP del clúster**: Piense en esto como si proporcionara un punto de conexión constante para los pods de aplicación. No se puede tener acceso a ningún pod asociado a estos servicios desde fuera del clúster de Kubernetes. La dirección IP usada con estos servicios procede del espacio de direcciones de la red privada. 
- **IP del equilibrador de carga**: Como el servicio IP del clúster, pero la dirección IP asociada procede de la red externa y se puede acceder a ella desde fuera del clúster de Kubernetes.


<!--## Networking example for an app


![Kubernetes networking example](./media/azure-stack-edge-gpu-kubernetes-networking/kubernetes-networking-2.png)

Each of these applications pods has a label associated with it. For example, the web server application pods have a label `app = WS` and the service has a label selector which the same as `app = WS`. Whenever a service of type load balancer or cluster IP is created, there is a control loop that runs in the master and publishes an endpoint corresponding to this service. This service uses a combination of labels and label selectors to discover the pods associated with this service. As a pod gets created, the new endpoint for the pod is added to the endpoint mapping. Whenever a pod is deleted, it gets deleted from the endpoint mapping. Using this endpoint controller, the service has a most up-to-date view of the pods that make up this application.

For discovery of applications within the cluster, Kubernetes cluster has an inbuilt DNS server pod. This is a cluster DNS that resolves service names to cluster IP. Anytime a cluster IP service is created, a DNS record is added to the DNS server that maps the name of the service to the cluster internal IP. That is how the applications within the cluster can discover each other. For load balancing, there is also the `kube-proxy`. This runs on every node and captures the traffic that comes in through the cluster IP and then distributes the traffic across the pods. 

When an application or the end user would first use the IP address associated with the service of type load balancer to discover the service. Then it would use the label select `app = WS` to discover the pods associated with the application. The `kube-proxy` component would then distribute the traffic and ensure that it hits one of the web server application pods. If the web server app wanted to talk to the database app, then it would simply use the name of the service and using the name and the DNS server pod, resolve the name to an IP address. Again using labels and label selector, it would discover the pods associated with the database application. The `kube-proxy` would then distribute the traffic across each of the database app nodes.-->


## <a name="kubernetes-networking-on-azure-stack-edge-pro"></a>Redes de Kubernetes en Azure Stack Edge Pro

Calico, Metallb y Core DNS son todos los componentes que se instalan para las redes en Azure Stack Edge Pro. 

- **Calico** asigna una dirección IP de un intervalo de direcciones IP privadas a cada pod, y configura las redes para estos pods de modo que pod en un nodo pueda comunicarse con el pod en otro nodo. 
- **Metallb** se ejecuta en un pod en el clúster y asigna la dirección IP a los servicios de tipo equilibrador de carga. Las direcciones IP del equilibrador de carga se eligen del intervalo IP del servicio, proporcionado a través de la interfaz de usuario local. 
- **Core DNS**: Este complemento configura el nombre del servicio de asignación de registros DNS a la dirección IP del clúster.

Las direcciones IP que se usan para los nodos de Kubernetes y los servicios externos se proporcionan a través de la página **Proceso** en la interfaz de usuario local del dispositivo.

![Asignación de la dirección IP de Kubernetes en la interfaz de usuario local](./media/azure-stack-edge-gpu-kubernetes-networking/kubernetes-ip-assignment-local-ui-1.png)

La asignación de IP es para:

- **Kubernetes node IPs** (Direcciones IP del nodo de Kubernetes): Este intervalo IP se usa para los nodos maestro y de trabajo de Kubernetes. Estas direcciones IP se usan cuando los nodos de Kubernetes se comunican entre sí.

- **Kubernetes external service IPs** (Direcciones IP del servicio externo de Kubernetes): Este intervalo IP se usa para los servicios externos (también conocidos como servicios del equilibrador de carga) que se exponen fuera del clúster de Kubernetes.  


## <a name="next-steps"></a>Pasos siguientes

Para configurar las redes de Kubernetes en un dispositivo Azure Stack Edge Pro, vea lo siguiente:

- [Exposición de una aplicación sin estado de forma externa en el dispositivo Azure Stack Edge Pro mediante IoT Edge](azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module.md)

- [Exposición de una aplicación sin estado de forma externa en el dispositivo Azure Stack Edge Pro mediante kuebctl](azure-stack-edge-j-series-deploy-stateless-application-kubernetes.md)
