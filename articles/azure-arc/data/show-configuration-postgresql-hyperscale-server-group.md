---
title: Visualización de la configuración de un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc
titleSuffix: Azure Arc enabled data services
description: Visualización de la configuración de un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: ceab9af7e6556b2d957fafce8cd89d4a0daf9508
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90932600"
---
# <a name="show-the-configuration-of-an-arc-enabled-postgresql-hyperscale-server-group"></a>Visualización de la configuración de un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc

En este artículo se explica cómo mostrar la configuración de los grupos de servidores. Lo explica anticipando algunas preguntas que puede que se haga y que encuentre respuestas. En ocasiones, puede haber varias respuestas válidas. En este artículo se detallan las más comunes y otras de utilidad. Esas preguntas se agrupan por tema:

- desde el punto de vista de Kubernetes
- desde el punto de vista de los servicios de datos habilitados para Azure Arc

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="from-a-kubernetes-point-of-view"></a>Desde el punto de vista de Kubernetes

### <a name="how-many-pods-are-used-by-azure-arc-enabled-postgresql-hyperscale"></a>¿Cuántos pods se usan en Hiperescala de PostgreSQL habilitada para Azure Arc?

Enumere los recursos de Kubernetes del tipo Postgres. Ejecute el comando:

```console
kubectl get postgresqls [-n <namespace name>]
```

La salida de este comando muestra la lista de grupos de servidores creados. Para cada uno, se indica el número de pods. Por ejemplo:

```output
NAME                                             STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
postgresql-12.arcdata.microsoft.com/postgres01   Ready   3/3          10.0.0.4:30499      6h34m
postgresql-12.arcdata.microsoft.com/postgres02   Ready   3/3          10.0.0.4:31066      6d7h
```

En este ejemplo se muestra la creación de dos grupos de servidores y cada uno de ellos se ejecuta en 3 pods (1 coordinador + 2 nodos de trabajo). Esto significa que los grupos de servidores creados en este controlador de datos de Azure Arc usan 6 pods.

### <a name="what-pods-are-used-by-azure-arc-enabled-postgresql-hyperscale-server-groups"></a>¿Qué pods se usan en los grupos de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc?

Ejecute:

```console
kubectl get pods [-n <namespace name>]
```

Esto devuelve la lista de pods. Verá los pods usados por los grupos de servidores en función de los nombres asignados a esos grupos de servidores. Por ejemplo:

```console 
NAME                 READY   STATUS    RESTARTS   AGE
bootstrapper-vdltm   1/1     Running   0          6d8h
control-h6kc9        2/2     Running   0          6d8h
controldb-0          2/2     Running   0          6d8h
controlwd-96sbn      1/1     Running   0          6d8h
logsdb-0             1/1     Running   0          6d8h
logsui-7wkg2         1/1     Running   0          6d8h
metricsdb-0          1/1     Running   0          6d8h
metricsdc-28ffl      1/1     Running   0          6d8h
metricsui-k7qsh      1/1     Running   0          6d8h
mgmtproxy-gd84z      2/2     Running   0          6d8h
postgres01-0         3/3     Running   0          6h50m
postgres01-1         3/3     Running   0          6h50m
postgres01-2         3/3     Running   0          6h50m
postgres02-0         3/3     Running   0          22h
postgres02-1         3/3     Running   0          22h
postgres02-2         3/3     Running   0          22h
```

En este ejemplo, los seis pods que hospedan los dos grupos de servidores que se han creado son:
- `postgres01-0`
- `postgres01-1`
- `postgres01-2`
- `postgres02-0`
- `postgres02-1`
- `postgres02-2`  

### <a name="what-server-group-pod-is-used-for-what-role-the-server-group"></a>¿Qué pod del grupo de servidores se usa para cada rol en el grupo de servidores?

Cualquier nombre de pod con el sufijo `-0` representa un nodo de coordinación. Cualquier nombre de nodo con el sufijo `-x`, donde x es 1 o superior, es un nodo de trabajo. En el ejemplo anterior:
- Los coordinadores son: `postgres01-0` y `postgres02-0`
- Los nodos de trabajo son: `postgres01-2`, `postgres01-2`, `postgres02-1` y `postgres02-2`

### <a name="what-is-the-status-of-the-pods"></a>¿Cuál es el estado de los pods?

Ejecute `kubectl get pods` y examine la columna `STATUS`.

### <a name="what-persistent-volume-claims-pvcs-are-being-used"></a>¿Qué notificaciones de volumen persistente (PVC) se usan? 

Para comprender qué PVC se usan, así como las que se usan para datos, registros y copias de seguridad, ejecute: 

```console
kubectl get pvc [-n <namespace name>]
```

De forma predeterminada, el prefijo del nombre de una PVC indica su uso:

- `backups-`...: es una PVC que se usa para copias de seguridad.
- `data-`...: es una PVC que se usa para archivos de datos.
- `logs-`...: es una PVC que se usa para los registros de transacciones y los archivos WAL.

Por ejemplo:

```output
NAME                   STATUS   VOLUME              CAPACITY   ACCESS MODES   STORAGECLASS    AGE
backups-postgres01-0   Bound    local-pv-485e37db   1938Gi     RWO            local-storage   6d6h
backups-postgres01-1   Bound    local-pv-9d3d4a15   1938Gi     RWO            local-storage   6d6h
backups-postgres01-2   Bound    local-pv-7b8dd819   1938Gi     RWO            local-storage   6d6h
...
data-postgres01-0      Bound    local-pv-3c1a8cc5   1938Gi     RWO            local-storage   6d6h
data-postgres01-1      Bound    local-pv-8303ab19   1938Gi     RWO            local-storage   6d6h
data-postgres01-2      Bound    local-pv-55572fe6   1938Gi     RWO            local-storage   6d6h
...
logs-postgres01-0      Bound    local-pv-5e852b76   1938Gi     RWO            local-storage   6d6h
logs-postgres01-1      Bound    local-pv-55d309a7   1938Gi     RWO            local-storage   6d6h
logs-postgres01-2      Bound    local-pv-5ccd02e6   1938Gi     RWO            local-storage   6d6h
...
```


## <a name="from-an-azure-arc-enabled-data-services-point-of-view"></a>Desde el punto de vista de los servicios de datos habilitados para Azure Arc:

* ¿Cuántos grupos de servidores se han creado en un controlador de datos de Arc?
* ¿Cuáles son sus nombres?
* ¿Cuántos nodos de trabajo usan?
* ¿Qué versión de Postgres ejecutan?

Use uno de los siguientes comandos.
- **Con kubectl:**

   ```console
   kubectl get postgresqls [-n <namespace name>]
   ``` 

   Por ejemplo, genera la siguiente salida, donde cada línea es un `servergroup`. La estructura del nombre en la pantalla siguiente se compone de:

   `<Name-Of-Custom-Resource-Definition>`/`<Server-Group-Name>`

   ```output
   NAME                                             STATE   READY-PODS   EXTERNAL-ENDPOINT   AGE
   postgresql-12.arcdata.microsoft.com/postgres01   Ready   3/3          10.0.0.4:30499      7h15m
   postgresql-12.arcdata.microsoft.com/postgres02   Ready   3/3          10.0.0.4:31066      6d7h
   ```

   La salida anterior muestra dos grupos de servidores que ejecutan la versión 12 de Postgres. Si la versión fuera Postgres 11, el nombre del CRD sería postgresql-11.arcdata.microsoft.com.

   Cada uno de ellos se ejecuta en 3 nodos o pods: 1 nodo de coordinación y 2 nodos de trabajo.

- **Con azdata:**

Ejecute el siguiente comando. La salida muestra información similar a la que muestra kubectl:

   ```console
   azdata arc postgres server list

   `output
   Name        State    Workers
   ----------  -------  ---------
   postgres01  Ready    2
   postgres02  Ready    2
   ```


### <a name="how-much-memory-and-vcores-are-being-used-and-what-extensions-were-created-for-a-group"></a>¿Cuánta memoria y núcleos virtuales se usan y qué extensiones se han creado para un grupo?

Ejecute uno de los siguientes comandos:

**Con kubectl:**

Use kubectl para describir los recursos de Postgres. Para ello, necesita su tipo (nombre del recurso de Kubernetes (CRD) para la versión de Postgres correspondiente, tal y como se mostró anteriormente) y el nombre del grupo de servidores.
El formato general de este comando es:

```console
kubectl describe <CRD name>/<server group name> [-n <namespace name>]
```

Por ejemplo:

```console
kubectl describe postgresql-12/postgres02
```

Este comando muestra la configuración del grupo de servidores:

```output
Name:         postgres02
Namespace:    arc
Labels:       <none>
Annotations:  <none>
API Version:  arcdata.microsoft.com/v1alpha1
Kind:         postgresql-12
Metadata:
  Creation Timestamp:  2020-08-31T21:01:07Z
  Generation:          10
  Resource Version:    569516
  Self Link:           /apis/arcdata.microsoft.com/v1alpha1/namespaces/arc/postgresql-12s/postgres02
  UID:                 8a9cd118-361b-4a2e-8a9d-5f9257bf6abb
Spec:
  Backups:
    Delta Minutes:  3
    Full Minutes:   10
    Tiers:
      Retention:
        Maximums:
          6
          512MB
        Minimums:
          3
      Storage:
        Volume Size:  1Gi
  Engine:
    Extensions:
      Name:  citus
      Name:  pg_stat_statements
  Scale:
    Shards:  2
  Scheduling:
    Default:
      Resources:
        Limits:
          Cpu:     4
          Memory:  1024Mi
        Requests:
          Cpu:     1
          Memory:  512Mi
  Service:
    Type:  NodePort
  Storage:
    Data:
      Class Name:  local-storage
      Size:        5Gi
    Logs:
      Class Name:  local-storage
      Size:        5Gi
Status:
  External Endpoint:  10.0.0.4:31066
  Ready Pods:         3/3
  State:              Ready
Events:               <none>
```

Vamos a explicar algunos puntos específicos de interés de la descripción del elemento `servergroup` mostrada anteriormente. ¿Qué nos informa sobre este grupo de servidores?

- Se trata de la versión 12 de Postgres: 
   > Tipo: `postgresql-12`
- Se creó durante el mes de agosto de 2020:
   > Marca de tiempo de creación: `2020-08-31T21:01:07Z`
- Se han creado dos extensiones de Postgres en este grupo de servidores: `citus` y `pg_stat_statements`
   > Motor: Extensiones:   Nombre:  `citus` Nombre: `pg_stat_statements`
- Usa dos nodos de trabajo.
   > Escala: Particiones: `2`
- Se garantiza el uso de 1 CPU/núcleo virtual y 512 MB de memoria RAM por nodo. Utilizará más de 4 CPU/núcleos virtuales y 1024 MB de memoria:
   > Programación: Valor predeterminado:   Recursos:     Límites:       CPU:     4 Memoria:  1024 Mi Solicitudes:       CPU:     1 Memoria:  512 Mi
 - Está disponible para consultas y no tiene ningún problema. Todos los nodos están en funcionamiento:
   > Estado: ... Pods listos:         3/3 Estado:              Ready

**Con azdata:**

El formato general del comando es:

```console
azdata arc postgres server show -n <server group name>
```

Por ejemplo:

```console
azdata arc postgres server show -n postgres02
```

Devuelve la siguiente salida en un formato y contenido muy similar al devuelto por kubectl.

```output
{
  "apiVersion": "arcdata.microsoft.com/v1alpha1",
  "kind": "postgresql-12",
  "metadata": {
    "creationTimestamp": "2020-08-31T21:01:07Z",
    "generation": 10,
    "name": "postgres02",
    "namespace": "arc",
    "resourceVersion": "569516",
    "selfLink": "/apis/arcdata.microsoft.com/v1alpha1/namespaces/arc/postgresql-12s/postgres02",
    "uid": "8a9cd118-361b-4a2e-8a9d-5f9257bf6abb"
  },
  "spec": {
    "backups": {
      "deltaMinutes": 3,
      "fullMinutes": 10,
      "tiers": [
        {
          "retention": {
            "maximums": [
              "6",
              "512MB"
            ],
            "minimums": [
              "3"
            ]
          },
          "storage": {
            "volumeSize": "1Gi"
          }
        }
      ]
    },
    "engine": {
      "extensions": [
        {
          "name": "citus"
        },
        {
          "name": "pg_stat_statements"
        }
      ]
    },
    "scale": {
      "shards": 2
    },
    "scheduling": {
      "default": {
        "resources": {
          "limits": {
            "cpu": "4",
            "memory": "1024Mi"
          },
          "requests": {
            "cpu": "1",
            "memory": "512Mi"
          }
        }
      }
    },
    "service": {
      "type": "NodePort"
    },
    "storage": {
      "data": {
        "className": "local-storage",
        "size": "5Gi"
      },
      "logs": {
        "className": "local-storage",
        "size": "5Gi"
      }
    }
  },
  "status": {
    "externalEndpoint": "10.0.0.4:31066",
    "readyPods": "3/3",
    "state": "Ready"
  }
}
```

## <a name="next-steps"></a>Pasos siguientes
- [Obtenga información sobre los conceptos de Hiperescala de PostgreSQL habilitada para Azure Arc](concepts-distributed-postgres-hyperscale.md).
- [Obtenga información acerca de cómo escalar horizontalmente (agregar nodos de trabajo) un grupo de servidores](scale-out-postgresql-hyperscale-server-group.md).
- [Obtenga información acerca de cómo escalar o reducir verticalmente (aumentar o reducir la memoria y/o los núcleos virtuales) de un grupo de servidores](scale-up-down-postgresql-hyperscale-server-group-using-cli.md).
- [Obtenga información sobre la configuración de almacenamiento](storage-configuration.md).
- [Consulte cómo supervisar una instancia de base de datos](monitor-grafana-kibana.md)
- [Uso de extensiones de PostgreSQL en el grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc](using-extensions-in-postgresql-hyperscale-server-group.md)
- [Configuración de la seguridad del grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc](configure-security-postgres-hyperscale.md)
