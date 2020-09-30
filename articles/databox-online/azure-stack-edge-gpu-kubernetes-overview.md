---
title: Información general del clúster de Kubernetes en el dispositivo Microsoft Azure Stack Edge Pro | Microsoft Docs
description: Se describe cómo se implementa Kubernetes en el dispositivo Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: alkohli
ms.openlocfilehash: 089476980ff3e8b2d3b72cb2d229934cf7850c03
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90899255"
---
# <a name="kubernetes-on-your-azure-stack-edge-pro-gpu-device"></a>Kubernetes en el dispositivo Azure Stack Edge Pro con GPU

Kubernetes es una conocida plataforma de código abierto para orquestar aplicaciones en contenedor. En este artículo se proporciona información general sobre Kubernetes y, después, se describe cómo funciona Kubernetes en el dispositivo Azure Stack Edge Pro. 

## <a name="about-kubernetes"></a>Acerca de Kubernetes 

Kubernetes proporciona una plataforma sencilla y confiable para administrar aplicaciones basadas en contenedores y sus componentes de red y almacenamiento asociados. Con Kubernetes puede compilar, distribuir y escalar rápidamente aplicaciones en contenedores.

Como plataforma abierta, Kubernetes le permite compilar aplicaciones con el lenguaje de programación, las bibliotecas de sistema operativo o el bus de mensajería que prefiera. Para programar e implementar versiones, Kubernetes puede integrarse con herramientas de integración continua y de entrega continua ya existentes.

Para más información, vea el vídeo [¿Cómo funciona Kubernetes?](https://www.youtube.com/watch?v=q1PcAawa4Bg&list=PLLasX02E8BPCrIhFrc_ZiINhbRkYMKdPT&index=2&t=0s).

## <a name="kubernetes-on-azure-stack-edge-pro"></a>Kubernetes en Azure Stack Edge Pro

En el dispositivo Azure Stack Edge Pro, puede crear un clúster de Kubernetes mediante la configuración del proceso. Cuando se configura el rol de proceso, el clúster de Kubernetes, incluidos los nodos maestro y de trabajo, se implementan y configuran automáticamente. Este clúster se usa para la implementación de cargas de trabajo mediante `kubectl`, IoT Edge o Azure Arc.

El dispositivo Azure Stack Edge Pro está disponible como una configuración de un nodo que constituye el clúster de infraestructura. El clúster de Kubernetes es independiente del clúster de infraestructura y se implementa a partir del clúster de infraestructura. El clúster de infraestructura proporciona el almacenamiento persistente para el dispositivo Azure Stack Edge Pro, mientras que el clúster de Kubernetes se encarga únicamente de la orquestación de la aplicación. 

En este caso, el clúster de Kubernetes tiene un nodo maestro y un nodo de trabajo. Los nodos de Kubernetes de un clúster son máquinas virtuales que ejecutan sus aplicaciones y flujos de trabajo en la nube. 

El nodo maestro de Kubernetes es responsable de mantener el estado deseado para el clúster. El nodo maestro controla al nodo de trabajo que a su vez ejecuta las aplicaciones en contenedor. 

En el diagrama siguiente se ilustra la implementación de Kubernetes en un dispositivo Azure Stack Edge Pro de un nodo. El dispositivo de un nodo no tiene alta disponibilidad y, si se produce un error en el único nodo, el dispositivo deja de funcionar. El clúster de Kubernetes también deja de funcionar.

![Arquitectura de Kubernetes para un dispositivo Azure Stack Edge Pro de un nodo](media/azure-stack-edge-gpu-kubernetes-overview/kubernetes-architecture-1-node.png)

Para más información sobre la arquitectura de clúster de Kubernetes, consulte los [Conceptos básicos de Kubernetes](https://kubernetes.io/docs/concepts/architecture/).


<!--The Kubernetes cluster control plane components make global decisions about the cluster. The control plane has:

- *kubeapiserver* that is the front end of the Kubernetes API and exposes the API.
- *etcd* that is a highly available key value store that backs up all the Kubernetes cluster data.
- *kube-scheduler* that makes scheduling decisions.
- *kube-controller-manager* that runs controller processes such as those for node controllers, replications controllers, endpoint controllers, and service account and token controllers. -->

## <a name="storage-volume-provisioning"></a>Aprovisionamiento de volumen de almacenamiento

Con el fin de admitir las cargas de trabajo de la aplicación, puede montar volúmenes de almacenamiento para los datos persistentes en los recursos compartidos del dispositivo Azure Stack Edge Pro. Se pueden usar volúmenes estáticos y dinámicos. 

A fin de obtener más información, vea las opciones de aprovisionamiento de almacenamiento para aplicaciones en [Almacenamiento de Kubernetes para el dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-kubernetes-storage.md).

## <a name="networking"></a>Redes

Las funciones de red de Kubernetes le permiten configurar la comunicación dentro de la red de Kubernetes, como la comunicación de red de contenedor a contenedor, de pod a pod, de pod a servicio y de Internet a servicio. Para obtener más información, vea el modelo de red en [Funciones de red de Kubernetes para el dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-kubernetes-networking.md).

## <a name="updates"></a>Actualizaciones

A medida que las nuevas versiones de Kubernetes estén disponibles, el clúster se puede actualizar con las actualizaciones estándar disponibles para el dispositivo Azure Stack Edge Pro. Para conocer los pasos necesarios a fin de realizar la actualización, vea [Aplicación de actualizaciones para Azure Stack Edge Pro](azure-stack-edge-gpu-install-update.md).

## <a name="access-monitoring"></a>Acceso, supervisión

El clúster de Kubernetes en el dispositivo Azure Stack Edge Pro permite el control de acceso basado en rol (RBAC). A fin de obtener más información, vea [Control de acceso basado en rol para el clúster de Kubernetes en el dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-kubernetes-rbac.md).

También puede supervisar el estado del clúster y los recursos mediante el panel de Kubernetes. También hay disponibles registros de contenedor. A fin de obtener más información, vea [Uso del panel de Kubernetes para supervisar el estado del clúster de Kubernetes en el dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md).

Azure Monitor también está disponible como complemento para recopilar datos de estado de los contenedores, los nodos y los controladores. Para más información, consulte [Introducción a Azure Monitor](../azure-monitor/overview.md).

<!--## Private container registry

Kubernetes on Azure Stack Edge Pro device allows for the private storage of your images by providing a local container registry.-->

## <a name="application-management"></a>Administración de aplicaciones

Después de crear un clúster de Kubernetes en el dispositivo Azure Stack Edge Pro, puede administrar las aplicaciones implementadas en este clúster mediante cualquiera de los métodos siguientes:

- Acceso nativo mediante `kubectl`
- IoT Edge 
- Azure Arc

Los métodos se explican en las siguientes secciones.


### <a name="kubernetes-and-kubectl"></a>Kubernetes y kubectl

Una vez implementado el clúster de Kubernetes, puede administrar las aplicaciones implementadas en el clúster localmente desde una máquina cliente. Para ello use una herramienta nativa como *kubectl* desde la línea de comandos para interactuar con las aplicaciones. 

Para obtener más información sobre la implementación del clúster de Kubernetes, vaya a [Implementación de un clúster de Kubernetes en el dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md). A fin de obtener información sobre la administración, vaya a [Uso de kubectl para administrar el clúster de Kubernetes en el dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md).


### <a name="kubernetes-and-iot-edge"></a>Kubernetes e IoT Edge

Kubernetes también se puede integrar con cargas de trabajo de IoT Edge en el dispositivo Azure Stack Edge Pro, donde Kubernetes proporciona escalado, y el ecosistema e IoT proporciona el ecosistema centrado en IoT. La capa de Kubernetes se usa como una capa de infraestructura para implementar cargas de trabajo de Azure IoT Edge. La vigencia del módulo y el equilibrio de carga de red se administran mediante Kubernetes, mientras que la plataforma de aplicación perimetral se administra mediante IoT Edge.

Para más información sobre la implementación de aplicaciones en el clúster de Kubernetes mediante de IoT Edge, vaya a: 

- [Exposición de aplicaciones sin estado en el dispositivo Azure Stack Edge Pro mediante IoT Edge](azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module.md).


### <a name="kubernetes-and-azure-arc"></a>Kubernetes y Azure Arc

Azure Arc es una herramienta de administración híbrida que le permitirá implementar aplicaciones en los clústeres de Kubernetes. Azure Arc también le permite usar Azure Monitor para contenedores para ver y supervisar los clústeres. Para más información, consulte [¿Qué es Kubernetes habilitado para Azure-Arc?](https://docs.microsoft.com/azure/azure-arc/kubernetes/overview). Para más información sobre los precios de Azure Arc, vaya a [Precios de Azure ARC](https://azure.microsoft.com/services/azure-arc/#pricing).


## <a name="next-steps"></a>Pasos siguientes

- Más información sobre el almacenamiento de Kubernetes en [dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-kubernetes-storage.md).
- Descubra el modelo de redes de Kubernetes en [dispositivo Azure Stack Edge Pro](azure-stack-edge-gpu-kubernetes-networking.md).
- Implemente [Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md) en Azure Portal.
