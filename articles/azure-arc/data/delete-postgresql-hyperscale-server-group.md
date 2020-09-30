---
title: Eliminación de un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc
description: Eliminación de un grupo de servidores de Hiperescala de Postgres habilitada para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: dcabe4b1520c66b8d5bfa398dc1248972587cd32
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932609"
---
# <a name="delete-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Eliminación de un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc

En este documento se describen los pasos para eliminar un grupo de servidores de la configuración de Azure Arc.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="delete-the-server-group"></a>Eliminación del grupo de servidores

Por ejemplo, supongamos que queremos eliminar la instancia _postgres01_ de la configuración siguiente:

```console
azdata arc postgres server list
Name        State    Workers
----------  -------  ---------
postgres01  Ready    3
```

El formato general del comando de eliminación es el siguiente:
```console
azdata arc postgres server delete -n <server group name>
```
Para más detalles sobre el comando de eliminación, ejecute:
```console
azdata arc postgres server delete --help
```

### <a name="lets-delete-the-server-group-used-in-this-example"></a>Vamos a eliminar el grupo de servidores que se usa en este ejemplo:
```console
azdata arc postgres server delete -n postgres01
```

## <a name="reclaim-the-kubernetes-persistent-volume-claims-pvcs"></a>Reclamación de las notificaciones de volumen persistente (PVC) de Kubernetes

La eliminación de un grupo de servidores no elimina sus [PVC](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) asociadas. es así por diseño. La intención es ayudar al usuario a tener acceso a los archivos de base de datos en caso de que la eliminación de la instancia fuera accidental. La eliminación de las PVC no es obligatoria. Sin embargo, se recomienda. Si no reclama estas PVC, finalmente terminará con errores, ya que el clúster de Kubernetes pensará que se está quedando sin espacio en disco. Para reclamar las PVC, realice los pasos siguientes:

### <a name="1-list-the-pvcs-for-the-server-group-you-deleted"></a>1. Enumerar las PVC del grupo de servidores que ha eliminado
Para enumerar las PVC, ejecute este comando:
```console
kubectl get pvc [-n <namespace name>]
```

Devuelve la lista de PVS y, en particular, las PVC del grupo de servidores que ha eliminado. Por ejemplo:
```console
kubectl get pvc
NAME                STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-postgres01-0   Bound    pvc-72ccc225-dad0-4dee-8eae-ed352be847aa   5Gi        RWO            default        2d18h
data-postgres01-1   Bound    pvc-ce6f0c51-faed-45ae-9472-8cdf390deb0d   5Gi        RWO            default        2d18h
data-postgres01-2   Bound    pvc-5a863ab9-522a-45f3-889b-8084c48c32f8   5Gi        RWO            default        2d18h
data-postgres01-3   Bound    pvc-00e1ace3-1452-434f-8445-767ec39c23f2   5Gi        RWO            default        2d15h
logs-postgres01-0   Bound    pvc-8b810f4c-d72a-474a-a5d7-64ec26fa32de   5Gi        RWO            default        2d18h
logs-postgres01-1   Bound    pvc-51d1e91b-08a9-4b6b-858d-38e8e06e60f9   5Gi        RWO            default        2d18h
logs-postgres01-2   Bound    pvc-8e5ad55e-300d-4353-92d8-2e383b3fe96e   5Gi        RWO            default        2d18h
logs-postgres01-3   Bound    pvc-f9e4cb98-c943-45b0-aa07-dd5cff7ea585   5Gi        RWO            default        2d15h
```
Hay 8 PVC para este grupo de servidores.

### <a name="2-delete-each-of-the-pvcs"></a>2. Eliminar cada PVC
Elimine las PVC de datos y de registro de cada uno de los nodos de Hiperescala de PostgresSQL (nodo de coordinación y nodos de trabajo) del grupo de servidores que ha eliminado.
El formato general de este comando es: 
```console
kubectl delete pvc <name of pvc>  [-n <namespace name>]
```

Por ejemplo:
```console
kubectl delete pvc data-postgres01-0
kubectl delete pvc data-postgres01-1 
kubectl delete pvc data-postgres01-2
kubectl delete pvc data-postgres01-3
kubectl delete pvc logs-postgres01-0
kubectl delete pvc logs-postgres01-1
kubectl delete pvc logs-postgres01-2
kubectl delete pvc logs-postgres01-3
```

Cada uno de estos comandos kubectl confirmará la eliminación correcta del PVC. Por ejemplo:
```console
persistentvolumeclaim "data-postgres01-0" deleted
```
  

>**NOTA** Tal y como se indicó, si no elimina las PVC podría llevar al clúster de Kubernetes a una situación en la que se produzcan errores. Algunos de estos errores pueden incluir que no se puede iniciar sesión en el clúster de Kubernetes con azdata, ya que es posible que se expulsen los pods debido a este problema de almacenamiento (comportamiento normal de Kubernetes).
>
> Por ejemplo, puede ver mensajes en los registros similares a:  
    > Anotaciones: microsoft.com/ignore-pod-health: true  
    > Estado:         Con error  
    > Motivo:         Expulsado  
    > Mensaje:        El nodo tenía poco recursos: almacenamiento efímero. El controlador del contenedor utilizaba 16 372 Ki, que supera la solicitud de 0.
    
## <a name="next-step"></a>Paso siguiente
Creación de [Hiperescala de PostgreSQL habilitada para Azure Arc](create-postgresql-hyperscale-server-group.md)
