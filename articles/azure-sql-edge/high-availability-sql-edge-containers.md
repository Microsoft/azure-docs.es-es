---
title: 'Alta disponibilidad para los contenedores de Azure SQL Edge: Azure SQL Edge'
description: Más información sobre la alta disponibilidad para los contenedores de Azure SQL Edge
keywords: SQL Edge, contenedores, alta disponibilidad
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: 56b9f06547f737bc05d573f98ce1dbac2ba48758
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90931953"
---
# <a name="high-availability-for-azure-sql-edge-containers"></a>Alta disponibilidad para los contenedores de Azure SQL Edge

Cree y administre las instancias de Azure SQL Edge de forma nativa en Kubernetes. Implemente Azure SQL Edge en contenedores de Docker administrados por [Kubernetes](https://kubernetes.io/). En Kubernetes, un contenedor con una instancia de Azure SQL Edge se puede recuperar automáticamente en caso de error en un nodo de clúster. Puede configurar la imagen del contenedor de SQL Edge con una notificación de volumen persistente (PVC) de Kubernetes. Kubernetes supervisa el proceso de Azure SQL Edge en el contenedor. Si se produce un error en el proceso, el pod, el contenedor o el nodo, Kubernetes arranca automáticamente otra instancia y vuelve a conectar al almacenamiento.

## <a name="azure-sql-edge-containers-on-kubernetes"></a>Contenedores de Azure SQL Edge en Kubernetes

Kubernetes 1.6 y las versiones posteriores admiten [*clases de almacenamiento*](https://kubernetes.io/docs/concepts/storage/storage-classes/) y [*notificaciones de volumen persistente*](https://kubernetes.io/docs/concepts/storage/storage-classes/#persistentvolumeclaims).

En esta configuración, Kubernetes desempeña el rol de orquestador de contenedores. 

![Diagrama de Azure SQL Edge en un clúster de Kubernetes](media/deploy-kubernetes/kubernetes-sql-edge.png)

En el diagrama anterior, `azure-sql-edge` es un contenedor en un [pod](https://kubernetes.io/docs/concepts/workloads/pods/pod/). Kubernetes orquesta los recursos del clúster. Un [conjunto de réplicas](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/) garantiza que el pod se recupere automáticamente tras un error de nodo. Las aplicaciones se conectan al servicio. En este caso, el servicio representa un equilibrador de carga que hospeda una dirección IP que permanece igual tras un error de `azure-sql-edge`.

En el diagrama siguiente, se ha producido un error del contenedor `azure-sql-edge`. Como orquestador, Kubernetes garantiza el recuento correcto de instancias correctas en el conjunto de réplicas e inicia un nuevo contenedor de acuerdo con la configuración. El orquestador inicia un nuevo pod en el mismo nodo y `azure-sql-edge` se vuelve a conectar al mismo almacenamiento persistente. El servicio se conecta al nuevo `azure-sql-edge` creado.

![Azure SQL Edge en un clúster de Kubernetes tras un error de pod](media/deploy-kubernetes/kubernetes-sql-edge-after-pod-fail.png)

En el diagrama siguiente, se ha producido un error en el nodo que hospeda el contenedor `azure-sql-edge`. El orquestador inicia el nuevo pod en otro nodo y `azure-sql-edge` se vuelve a conectar al mismo almacenamiento persistente. El servicio se conecta al nuevo `azure-sql-edge` creado.

![Azure SQL Edge en un clúster de Kubernetes tras un error de nodo](media/deploy-kubernetes/kubernetes-sql-edge-after-node-fail.png).

Para crear un contenedor en Kubernetes, consulte [Implementación de un contenedor de Azure SQL Edge en Kubernetes](deploy-Kubernetes.md)

## <a name="next-steps"></a>Pasos siguientes

Para implementar contenedores de Azure SQL Edge en Azure Kubernetes Service (AKS), consulte los siguientes artículos:
- [Implementación de un contenedor de Azure SQL Edge en Kubernetes](deploy-Kubernetes.md)
- [Aprendizaje automático e inteligencia artificial con ONNX en SQL Edge](onnx-overview.md).
- [Creación de una solución de IoT de un extremo a otro con SQL Edge mediante IoT Edge](tutorial-deploy-azure-resources.md).
- [Streaming de datos en Azure SQL Edge](stream-data.md)