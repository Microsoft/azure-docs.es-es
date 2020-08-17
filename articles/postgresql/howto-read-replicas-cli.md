---
title: 'Administración de réplicas de lectura mediante la CLI de Azure y API REST en Azure Database for PostgreSQL: servidor único'
description: 'Aprenda a administrar réplicas de lectura desde la CLI de Azure con API REST en Azure Database for PostgreSQL: servidor único'
author: rachel-msft
ms.author: raagyema
ms.service: postgresql
ms.topic: how-to
ms.date: 07/10/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 491b3ecfc950fa5f76bfe78eec52e81433294c23
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500085"
---
# <a name="create-and-manage-read-replicas-from-the-azure-cli-rest-api"></a>Creación y administración de réplicas de lectura desde la CLI de Azure con API REST

En este artículo aprenderá a crear y administrar réplicas de lectura en Azure Database for PostgreSQL desde la CLI de Azure con API REST. Para más información acerca de las réplicas de lectura, consulte la [introducción](concepts-read-replicas.md).

## <a name="azure-replication-support"></a>Soporte de replicación de Azure
Tanto[las réplicas de lectura](concepts-read-replicas.md) como [la descodificación lógica](concepts-logical.md) dependen del registro de escritura previa (WAL) Postgres para obtener información. Estas dos características necesitan diferentes niveles de registro de Postgres. La descodificación lógica requiere un mayor nivel de registro que las réplicas de lectura.

Para configurar el nivel de registro adecuado, use el parámetro de soporte de replicación de Azure. El soporte de la replicación de Azure tiene tres opciones de valor:

* **Desactivado**: coloca la más mínima información en el WAL. Este valor no está disponible en la mayoría de los servidores Azure Database for PostgreSQL.  
* **Réplica**: más detallado que **Off**. Este es el nivel mínimo de registro necesario para que [las réplicas de lectura](concepts-read-replicas.md) funcionen. Esta es la configuración predeterminada en la mayoría de los servidores.
* **Lógico**: más detallado que **Réplica**. Este es el nivel mínimo de registro para que funcione la descodificación lógica. Las réplicas de lectura también funcionan con este valor.

El servidor debe reiniciarse después de un cambio de este parámetro. Internamente, este parámetro establece los parámetros Postgres `wal_level`, `max_replication_slots` y `max_wal_senders`.

## <a name="azure-cli"></a>Azure CLI
Puede crear y administrar réplicas de lectura mediante la CLI de Azure.

### <a name="prerequisites"></a>Requisitos previos

- [Instalación de la CLI de Azure 2.0](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest)
- Un [servidor de Azure Database for PostgreSQL](quickstart-create-server-up-azure-cli.md) que se usará como servidor maestro.


### <a name="prepare-the-master-server"></a>Preparación del servidor maestro

1. Compruebe el valor de `azure.replication_support` del servidor maestro. Debe ser al menos una réplica para que las réplicas de lectura funcionen.

   ```azurecli-interactive
   az postgres server configuration show --resource-group myresourcegroup --server-name mydemoserver --name azure.replication_support
   ```

2. Si `azure.replication_support` no es al menos una réplica, establézcala. 

   ```azurecli-interactive
   az postgres server configuration set --resource-group myresourcegroup --server-name mydemoserver --name azure.replication_support --value REPLICA
   ```

3. Reinicie el servidor para aplicar el cambio.

   ```azurecli-interactive
   az postgres server restart --name mydemoserver --resource-group myresourcegroup
   ```

### <a name="create-a-read-replica"></a>Creación de una réplica de lectura

El comando [az postgres server replica create](/cli/azure/postgres/server/replica?view=azure-cli-latest#az-postgres-server-replica-create) requiere los siguientes parámetros:

| Configuración | Valor de ejemplo | Descripción  |
| --- | --- | --- |
| resource-group | myresourcegroup |  Grupo de recursos donde se creará el servidor de réplica.  |
| name | mydemoserver-replica | Nombre del nuevo servidor de réplica que se crea. |
| source-server | mydemoserver | Nombre o identificador de recurso del servidor maestro existente desde el que replicar. Use el identificador de recurso si quiere que los grupos de recursos de la réplica y del maestro sean diferentes. |

En el siguiente ejemplo de la CLI, la réplica se crea en la misma región que el maestro.

```azurecli-interactive
az postgres server replica create --name mydemoserver-replica --source-server mydemoserver --resource-group myresourcegroup
```

Para crear una réplica de lectura entre regiones, use el parámetro `--location`. El siguiente ejemplo de la CLI crea la réplica en Oeste de EE. UU.

```azurecli-interactive
az postgres server replica create --name mydemoserver-replica --source-server mydemoserver --resource-group myresourcegroup --location westus
```

> [!NOTE]
> Para más información sobre las regiones en las que puede crear una réplica, consulte el [artículo sobre los conceptos de la réplica de lectura](concepts-read-replicas.md). 

Si no ha establecido el parámetro `azure.replication_support` en **REPLICA** en un servidor maestro de uso general u optimizado para memoria y no ha reiniciado el servidor, recibirá un error. Complete estos dos pasos antes de crear una réplica.

> [!IMPORTANT]
> Revise la [sección sobre las consideraciones de la información general de Réplicas de lectura](concepts-read-replicas.md#considerations).
>
> Antes de actualizar la configuración de un servidor maestro a un nuevo valor, actualice la configuración de réplica a un valor igual o superior. Esta acción ayuda a que la réplica haga frente a los cambios realizados en el servidor maestro.

### <a name="list-replicas"></a>Lista de réplicas
Puede ver la lista de réplicas de un servidor maestro con el comando [az postgres server replica list](/cli/azure/postgres/server/replica?view=azure-cli-latest#az-postgres-server-replica-list).

```azurecli-interactive
az postgres server replica list --server-name mydemoserver --resource-group myresourcegroup 
```

### <a name="stop-replication-to-a-replica-server"></a>Detención de la replicación en un servidor de réplica
Puede detener la replicación entre un servidor maestro y una réplica de lectura mediante el uso del comando [az postgres server replica stop](/cli/azure/postgres/server/replica?view=azure-cli-latest#az-postgres-server-replica-stop).

Después de detener la replicación en un servidor maestro y en una réplica de lectura, este proceso no se puede deshacer. La réplica de lectura se convierte en un servidor independiente que admite operaciones de lectura y escritura. Este servidor independiente no puede volver a convertirse en una réplica.

```azurecli-interactive
az postgres server replica stop --name mydemoserver-replica --resource-group myresourcegroup 
```

### <a name="delete-a-master-or-replica-server"></a>Eliminación de un servidor maestro o de réplica
Para eliminar un servidor maestro o de réplica, use el comando [az postgres server delete](/cli/azure/postgres/server?view=azure-cli-latest#az-postgres-server-delete).

Cuando se elimina un servidor maestro, la replicación se detiene en todas las réplicas de lectura. Las réplicas de lectura se convierten en servidores independientes que ahora admiten tanto lectura como escritura.

```azurecli-interactive
az postgres server delete --name myserver --resource-group myresourcegroup
```

## <a name="rest-api"></a>API DE REST
Puede crear y administrar réplicas de lectura mediante [API REST de Azure](/rest/api/azure/).

### <a name="prepare-the-master-server"></a>Preparación del servidor maestro

1. Compruebe el valor de `azure.replication_support` del servidor maestro. Debe ser al menos una réplica para que las réplicas de lectura funcionen.

   ```http
   GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{masterServerName}/configurations/azure.replication_support?api-version=2017-12-01
   ```

2. Si `azure.replication_support` no es al menos una réplica, establézcala.

   ```http
   PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{masterServerName}/configurations/azure.replication_support?api-version=2017-12-01
   ```

   ```JSON
   {
    "properties": {
    "value": "replica"
    }
   }
   ```

2. [Reinicie el servidor](/rest/api/postgresql/servers/restart) para aplicar el cambio.

   ```http
   POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{masterServerName}/restart?api-version=2017-12-01
   ```

### <a name="create-a-read-replica"></a>Creación de una réplica de lectura
Puede crear una réplica de lectura mediante la [API de creación](/rest/api/postgresql/servers/create):

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{replicaName}?api-version=2017-12-01
```

```json
{
  "location": "southeastasia",
  "properties": {
    "createMode": "Replica",
    "sourceServerId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{masterServerName}"
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
Puede ver la lista de réplicas de un servidor maestro mediante la [API de lista de réplicas](/rest/api/postgresql/replicas/listbyserver):

```http
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{masterServerName}/Replicas?api-version=2017-12-01
```

### <a name="stop-replication-to-a-replica-server"></a>Detención de la replicación en un servidor de réplica
Puede detener la replicación entre un servidor maestro y una réplica de lectura mediante la [API de actualización](/rest/api/postgresql/servers/update).

Después de detener la replicación en un servidor maestro y en una réplica de lectura, este proceso no se puede deshacer. La réplica de lectura se convierte en un servidor independiente que admite operaciones de lectura y escritura. Este servidor independiente no puede volver a convertirse en una réplica.

```http
PATCH https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{masterServerName}?api-version=2017-12-01
```

```json
{
  "properties": {
    "replicationRole":"None"  
   }
}
```

### <a name="delete-a-master-or-replica-server"></a>Eliminación de un servidor maestro o de réplica
Para eliminar un servidor maestro o una réplica, use la [API de eliminación](/rest/api/postgresql/servers/delete):

Cuando se elimina un servidor maestro, la replicación se detiene en todas las réplicas de lectura. Las réplicas de lectura se convierten en servidores independientes que ahora admiten tanto lectura como escritura.

```http
DELETE https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforPostgreSQL/servers/{serverName}?api-version=2017-12-01
```

## <a name="next-steps"></a>Pasos siguientes
* Más información sobre las [réplicas de lectura en Azure Database for PostgreSQL](concepts-read-replicas.md).
* Aprenda a [crear y administrar réplicas de lectura en Azure Portal](howto-read-replicas-portal.md).
