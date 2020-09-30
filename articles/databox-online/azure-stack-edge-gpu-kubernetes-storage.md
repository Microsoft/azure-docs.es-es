---
title: Descripción de la administración de almacenamiento de Kubernetes en un dispositivo Azure Stack Edge Pro | Microsoft Docs
description: Describe cómo se produce la administración del almacenamiento de Kubernetes en un dispositivo Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 08/27/2020
ms.author: alkohli
ms.openlocfilehash: ff2a473ca008e9b283d03ebb05f35122473d778a
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90899266"
---
# <a name="kubernetes-storage-management-on-your-azure-stack-edge-pro-gpu-device"></a>Administración del almacenamiento de Kubernetes en un dispositivo Azure Stack Edge Pro con GPU

En un dispositivo Azure Stack Edge Pro, se crea un clúster de Kubernetes al configurar el rol de proceso. Una vez creado el clúster de Kubernetes, las aplicaciones en contenedor se pueden implementar en el clúster de Kubernetes en pods. Hay distintas formas de proporcionar almacenamiento para los pods en el clúster de Kubernetes. 

En este artículo se describen los métodos para aprovisionar el almacenamiento en un clúster de Kubernetes en general y específicamente en el contexto de un dispositivo Azure Stack Edge Pro. 

## <a name="storage-requirements-for-kubernetes-pods"></a>Requisitos de almacenamiento para pods de Kubernetes

Los pods de Kubernetes no tienen estado, pero las aplicaciones que ejecutan suelen tener estado. Dado que los pods pueden ser de corta duración y se reinician, conmutan por error o se mueven entre nodos de Kubernetes, se deben cumplir los siguientes requisitos para el almacenamiento asociado al pod. 

El almacenamiento debe:

- Residir fuera del pod.
- Ser independiente del ciclo de vida del pod.
- Ser accesible desde todos los nodos de Kubernetes.

Para entender cómo se administra el almacenamiento de Kubernetes, es preciso comprender dos recursos de API: 

- **PersistentVolume (PV)** : Se trata de un fragmento de almacenamiento en el clúster de Kubernetes. El almacenamiento Kubernetes se puede aprovisionar estáticamente como `PersistentVolume`. También se puede aprovisionar dinámicamente como `StorageClass`.

- **PersistentVolumeClaim (PVC)** : Se trata de una solicitud de almacenamiento por parte de un usuario. Las PVC consumen recursos de PV. Las PVC pueden solicitar tamaños y modos de acceso específicos. 

    Dado que los usuarios necesitan `PersistentVolumes` con distintas propiedades para diferentes problemas, los administradores de clústeres deben ser capaces de ofrecer una variedad de `PersistentVolumes` que difieran no solo en el tamaño y los modos de acceso. Para estas necesidades, se necesita el recurso `StorageClass`.

El aprovisionamiento de almacenamiento puede ser estático o dinámico. En las siguientes secciones se analizan cada uno de estos tipos de aprovisionamiento.

## <a name="staticprovisioning"></a>Aprovisionamiento estático

Los administradores de clústeres de Kubernetes pueden aprovisionar el almacenamiento de forma estática. Para ello, pueden usar el back-end de almacenamiento basado en los sistemas de archivos SMB/NFS, o bien usar discos iSCSI que se conectan localmente a través de la red en un entorno local, o incluso usar Azure Files o discos de Azure en la nube. Este tipo de almacenamiento no se aprovisiona de manera predeterminada y los administradores de clústeres tienen que planear y administrar este aprovisionamiento. 
 
Este es un diagrama que muestra cómo se consume el almacenamiento aprovisionado estáticamente en Kubernetes: 

![Aprovisionamiento estático a través de PersistentVolumes](./media/azure-stack-edge-gpu-kubernetes-storage/static-provisioning-persistent-volumes-1.png)

Tienen lugar los pasos siguientes: 

1. **Almacenamiento del almacenamiento**: El administrador del clúster aprovisiona el almacenamiento. En este ejemplo, el administrador de clústeres crea uno o varios recursos compartidos de SMB que crean automáticamente objetos de volumen persistentes en el clúster de Kubernetes correspondiente a estos recursos compartidos. 

1. **Notificación del almacenamiento**: Usted envía una implementación de PVC que solicita el almacenamiento. Esta notificación de almacenamiento es PersistentVolumeClaim (PVC). Si el tamaño y el modo de acceso de PV coinciden con los del PVC, la PVC se enlaza al PV. La PVC y el PV se asignan uno a uno.

1. **Montaje de la PVC en el contenedor**: Una vez que la PVC esté enlazada al PV, puede montar esta PVC en una ruta de acceso del contenedor. Cuando la lógica de la aplicación en el contenedor lee o escribe en esta ruta de acceso, los datos se escriben en el almacenamiento SMB.
 

## <a name="dynamicprovisioning"></a>Aprovisionamiento dinámico

Este es un diagrama que muestra cómo se consume el almacenamiento aprovisionado estáticamente en Kubernetes: 

![Aprovisionamiento dinámico a través de StorageClasses](./media/azure-stack-edge-gpu-kubernetes-storage/dynamic-provisioning-storage-classes-1.png)

Tienen lugar los pasos siguientes: 

1. **Definición de la clase de almacenamiento**: El administrador de clústeres define una clase de almacenamiento en función del entorno operativo para el clúster de Kubernetes. Además, el administrador de clústeres implementa un aprovisionador, que es otro pod o aplicación que se implementa en el clúster de Kubernetes. El aprovisionador tiene todos los detalles para aprovisionar los recursos compartidos de manera dinámica.  

1. **Notificación del almacenamiento**: Usted envía una aplicación que notificaría el almacenamiento. Una vez que se crea una PVC con esta referencia de clase de almacenamiento, se invoca el aprovisionamiento. 

1. **Aprovisionamiento del almacenamiento de forma dinámica**: El aprovisionador crea dinámicamente el recurso compartido asociado al almacenamiento en disco local. Una vez creado el recurso compartido, también crea un objeto PV automáticamente correspondiente a este recurso compartido.

1. **Montaje de la PVC en el contenedor**: Una vez que la PVC está enlazada al PV, puede montar la PVC en el contenedor en una ruta de acceso de la misma manera que el aprovisionamiento estático, y leer o escribir en el recurso compartido.


## <a name="storage-provisioning-on-azure-stack-edge-pro"></a>Aprovisionamiento de almacenamiento en Azure Stack Edge Pro

En el dispositivo Azure Stack Edge Pro, los `PersistentVolumes` aprovisionados de forma estática se crean mediante las funcionalidades de almacenamiento del dispositivo. Cuando se aprovisiona un recurso compartido y la opción **Usar el recurso compartido con el proceso perimetral** está habilitada, esta acción crea un recurso PV automáticamente en el clúster de Kubernetes.

![Creación de recursos compartidos locales en Azure Portal para el aprovisionamiento estático](./media/azure-stack-edge-gpu-kubernetes-storage/static-provisioning-azure-portal-2.png)

Para usar la nube por niveles, puede crear un recurso compartido en la nube de Edge con la opción Usar el recurso compartido con el proceso perimetral habilitada. Se vuelve a crear automáticamente un PV para este recurso compartido. Los datos de aplicación que se escriben en el recurso compartido de Edge están organizados en capas en la nube. 

![Creación de recursos compartidos de nube en Azure Portal para el aprovisionamiento estático](./media/azure-stack-edge-gpu-kubernetes-storage/static-provisioning-azure-portal-1.png)

Se pueden crear recursos compartidos de archivos SMB y NFS para aprovisionar PV de forma estática en un dispositivo Azure Stack Edge Pro. Una vez aprovisionado el PV, usted enviará una PVC para notificar este almacenamiento. Este es un ejemplo de una implementación de PVC `yaml` que notifica el almacenamiento y usa los recursos compartidos aprovisionados.


```yml
kind: PersistentVolumeClaim 
apiVersion: v1 
metadata: 
  name: pvc-smb-flexvol 
spec: 
  accessModes: 
  - ReadWriteMany 
  resources: 
    requests: 
      storage: 10Gi 
  volumeName: <nfs-or-smb-share-name-here> 
  storageClassName: ""
```

Para más información, consulte [Implementación de una aplicación con estado a través del aprovisionamiento estático en Azure Stack Edge Pro mediante kubectl](azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes.md).

Azure Stack Edge Pro también tiene una `StorageClass` integrada denominada `ase-node-local` que usa un almacenamiento en disco de datos conectado al nodo de Kubernetes. Esta `StorageClass` admite el aprovisionamiento dinámico. Puede crear una referencia a `StorageClass` en las aplicaciones del pod, y un PV se crea automáticamente. Para obtener más información, consulte [Panel de Kubernetes](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md) para consultar por `ase-node-local StorageClass`.

![Clase de almacenamiento integrada en el panel de Kubernetes](./media/azure-stack-edge-gpu-kubernetes-storage/dynamic-provisioning-builtin-storage-class-1.png)

Para más información, consulte [Implementación de una aplicación con estado a través del aprovisionamiento dinámico en Azure Stack Edge Pro mediante kubectl](azure-stack-edge-gpu-deploy-stateful-application-dynamic-provision-kubernetes.md).

## <a name="choose-storage-type"></a>Elección del tipo de almacenamiento

Es posible que tenga que elegir el tipo de almacenamiento en función de la carga de trabajo que va a implementar. 

- Si quiere el modo de acceso `ReadWriteMany` para su `PersistentVolumes`, donde los volúmenes se montan como de lectura y escritura en muchos nodos que se implementan, use el aprovisionamiento estático para los recursos compartidos de SMB o NFS.

- Si las aplicaciones que está implementando tienen un requisito de cumplimiento de POSIX, por ejemplo, aplicaciones como MongoDB, PostgreSQL, MySQL o Prometheus, use la StorageClass integrada. Los modos de acceso son `ReadWriteOnce` o el volumen se monta como de lectura y escritura en un único nodo. 


Para obtener más información sobre los modos de acceso, consulte los [modos de acceso de volúmenes de Kubernetes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes).


## <a name="next-steps"></a>Pasos siguientes

Para comprender cómo puede aprovisionar un `PersistentVolume` de forma estática, consulte:

- [Implementación de una aplicación con estado a través del aprovisionamiento estático en Azure Stack Edge Pro mediante kubectl](azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes.md).

Para aprender a aprovisionar un `StorageClass` de forma dinámica, consulte:

- [Implementación de una aplicación con estado a través del aprovisionamiento dinámico en Azure Stack Edge Pro mediante kuebctl](azure-stack-edge-gpu-deploy-stateful-application-dynamic-provision-kubernetes.md).
