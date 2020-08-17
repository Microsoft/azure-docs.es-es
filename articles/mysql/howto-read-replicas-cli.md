---
title: 'Administración de réplicas de lectura mediante la CLI de Azure y API REST: Azure Database for MySQL'
description: Aprenda a crear y administrar réplicas de lectura en Azure Database for MySQL mediante la CLI de Azure o API REST.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 6/10/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 41c12dcb2e4c5e3b280bb349e81bcb58a0bafd01
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495768"
---
# <a name="how-to-create-and-manage-read-replicas-in-azure-database-for-mysql-using-the-azure-cli-and-rest-api"></a>Creación y administración de réplicas de lectura en Azure Database for MySQL mediante la CLI de Azure y API REST

En este artículo aprenderá a crear y administrar réplicas de lectura en el servicio Azure Database for MySQL mediante la CLI de Azure y API REST. Para más información acerca de las réplicas de lectura, consulte la [introducción](concepts-read-replicas.md).

## <a name="azure-cli"></a>Azure CLI
Puede crear y administrar réplicas de lectura mediante la CLI de Azure.

### <a name="prerequisites"></a>Prerrequisitos

- [Instalación de la CLI de Azure 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)
- Un [servidor de Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-portal.md) que se usará como servidor maestro. 

> [!IMPORTANT]
> La característica de réplica de lectura solo está disponible para servidores de Azure Database for MySQL en los planes de tarifa De uso general u Optimizada para memoria. Asegúrese de que el servidor maestro está en uno de estos planes de tarifa.

### <a name="create-a-read-replica"></a>Creación de una réplica de lectura

> [!IMPORTANT]
> Cuando se crea una réplica para un servidor maestro que no tiene réplicas existentes, el maestro se reiniciará en primer lugar para prepararse para la replicación. Téngalo en cuenta y realice estas operaciones durante un período de poca actividad.

Un servidor de réplica de lectura se puede crear mediante el comando siguiente:

```azurecli-interactive
az mysql server replica create --name mydemoreplicaserver --source-server mydemoserver --resource-group myresourcegroup
```

El comando `az mysql server replica create` requiere los siguientes parámetros:

| Configuración | Valor de ejemplo | Descripción  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  Grupo de recursos donde se creará el servidor de réplica.  |
| name | mydemoreplicaserver | Nombre del nuevo servidor de réplica que se crea. |
| source-server | mydemoserver | Nombre o identificador del servidor principal existente desde el que replicar. |

Para crear una réplica de lectura entre regiones, use el parámetro `--location`. El siguiente ejemplo de la CLI crea la réplica en Oeste de EE. UU.

```azurecli-interactive
az mysql server replica create --name mydemoreplicaserver --source-server mydemoserver --resource-group myresourcegroup --location westus
```

> [!NOTE]
> Para más información sobre las regiones en las que puede crear una réplica, consulte el [artículo sobre los conceptos de la réplica de lectura](concepts-read-replicas.md). 

> [!NOTE]
> Las réplicas de lectura se crean con la misma configuración de servidor que el servidor maestro. Una vez creado, se puede cambiar la configuración del servidor de réplica. Se recomienda mantener la configuración del servidor de réplica con valores iguales o mayores que el maestro para asegurarse de que la réplica trabajar al mismo nivel que el servidor maestro.


### <a name="list-replicas-for-a-master-server"></a>Lista de réplicas de un servidor maestro

Para ver todas las réplicas de un determinado servidor maestro, ejecute el siguiente comando: 

```azurecli-interactive
az mysql server replica list --server-name mydemoserver --resource-group myresourcegroup
```

El comando `az mysql server replica list` requiere los siguientes parámetros:

| Configuración | Valor de ejemplo | Descripción  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  Grupo de recursos donde se creará el servidor de réplica.  |
| server-name | mydemoserver | Nombre o identificador del servidor maestro. |

### <a name="stop-replication-to-a-replica-server"></a>Detención de la replicación en un servidor de réplica

> [!IMPORTANT]
> La detención la replicación en un servidor es irreversible. Una vez que detenida, la replicación entre un servidor maestro y una réplica no se puede deshacer. Después, el servidor de réplica se convierte en un servidor independiente que admite operaciones de lectura y escritura. Este servidor no puede volver a convertirse en una réplica.

La replicación de un servidor de réplica de lectura se puede detener mediante el comando siguiente:

```azurecli-interactive
az mysql server replica stop --name mydemoreplicaserver --resource-group myresourcegroup
```

El comando `az mysql server replica stop` requiere los siguientes parámetros:

| Configuración | Valor de ejemplo | Descripción  |
| --- | --- | --- |
| resource-group |  myresourcegroup |  Grupo de recursos donde se encuentra el servidor de réplica.  |
| name | mydemoreplicaserver | Nombre del servidor de réplica para el que desea detener la replicación. |

### <a name="delete-a-replica-server"></a>Eliminación de un servidor de réplica

La eliminación de un servidor de réplica de lectura se puede realizar mediante la ejecución del comando **[az mysql server delete](/cli/azure/mysql/server)** .

```azurecli-interactive
az mysql server delete --resource-group myresourcegroup --name mydemoreplicaserver
```

### <a name="delete-a-master-server"></a>Eliminación de un servidor maestro

> [!IMPORTANT]
> Al eliminar un servidor maestro, se detiene la replicación en todos los servidores de réplica y se elimina el propio servidor maestro. Los servidores de réplica se convierten en servidores independientes que ahora admiten tanto lectura como escritura.

Para eliminar un servidor maestro, puede ejecutar el comando **[az mysql server delete](/cli/azure/mysql/server)** .

```azurecli-interactive
az mysql server delete --resource-group myresourcegroup --name mydemoserver
```


## <a name="rest-api"></a>API DE REST
Puede crear y administrar réplicas de lectura mediante [API REST de Azure](/rest/api/azure/).

### <a name="create-a-read-replica"></a>Creación de una réplica de lectura
Puede crear una réplica de lectura mediante la [API de creación](/rest/api/mysql/servers/create):

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMySQL/servers/{replicaName}?api-version=2017-12-01
```

```json
{
  "location": "southeastasia",
  "properties": {
    "createMode": "Replica",
    "sourceServerId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMySQL/servers/{masterServerName}"
  }
}
```

> [!NOTE]
> Para más información sobre las regiones en las que puede crear una réplica, consulte el [artículo sobre los conceptos de la réplica de lectura](concepts-read-replicas.md). 

Si no ha establecido el parámetro `azure.replication_support` en **REPLICA** en un servidor maestro de uso general u optimizado para memoria y no ha reiniciado el servidor, recibirá un error. Complete estos dos pasos antes de crear una réplica.

Una réplica se crea con la misma configuración de proceso y almacenamiento que el servidor maestro. Después de crear una réplica, se pueden cambiar varias configuraciones independientemente del servidor maestro: generación de proceso, núcleos virtuales, almacenamiento y período de retención de copia de seguridad. El plan de tarifa también se puede cambiar de forma independiente, excepto si es con origen o destino en el nivel Básico.


> [!IMPORTANT]
> Antes de actualizar la configuración de un servidor maestro a un nuevo valor, actualice la configuración de réplica a un valor igual o superior. Esta acción ayuda a que la réplica haga frente a los cambios realizados en el servidor maestro.

### <a name="list-replicas"></a>Lista de réplicas
Puede ver la lista de réplicas de un servidor maestro mediante la [API de lista de réplicas](/rest/api/mysql/replicas/listbyserver):

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMySQL/servers/{masterServerName}/Replicas?api-version=2017-12-01
```

### <a name="stop-replication-to-a-replica-server"></a>Detención de la replicación en un servidor de réplica
Puede detener la replicación entre un servidor maestro y una réplica de lectura mediante la [API de actualización](/rest/api/mysql/servers/update).

Después de detener la replicación en un servidor maestro y en una réplica de lectura, este proceso no se puede deshacer. La réplica de lectura se convierte en un servidor independiente que admite operaciones de lectura y escritura. Este servidor independiente no puede volver a convertirse en una réplica.

```http
PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMySQL/servers/{masterServerName}?api-version=2017-12-01
```

```json
{
  "properties": {
    "replicationRole":"None"  
   }
}
```

### <a name="delete-a-master-or-replica-server"></a>Eliminación de un servidor maestro o de réplica
Para eliminar un servidor maestro o una réplica, use la [API de eliminación](/rest/api/mysql/servers/delete):

Cuando se elimina un servidor maestro, la replicación se detiene en todas las réplicas de lectura. Las réplicas de lectura se convierten en servidores independientes que ahora admiten tanto lectura como escritura.

```http
DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMySQL/servers/{serverName}?api-version=2017-12-01
```


## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre [las réplicas de lectura](concepts-read-replicas.md)
