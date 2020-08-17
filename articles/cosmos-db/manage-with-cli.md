---
title: Administración de recursos de Azure Cosmos DB mediante la CLI de Azure
description: Use la CLI de Azure para administrar la cuenta, la base de datos y los contenedores de Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.topic: how-to
ms.date: 07/29/2020
ms.author: mjbrown
ms.openlocfilehash: 0ae29039702a6f73a33f73afc366532077aa4b71
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432829"
---
# <a name="manage-azure-cosmos-resources-using-azure-cli"></a>Administración de recursos de Azure Cosmos mediante la CLI de Azure

En la siguiente guía, se describen los comandos comunes para automatizar la administración de las cuentas, las bases de datos y los contenedores de Azure Cosmos DB mediante la CLI de Azure. Hay páginas de referencia para todos los comandos de CLI de Azure Cosmos DB disponibles en la [referencia de la CLI de Azure](https://docs.microsoft.com/cli/azure/cosmosdb). Para ver más ejemplos, consulte [Ejemplos de la CLI de Azure para Azure Cosmos DB](cli-samples.md), incluidos ejemplos sobre cómo crear y administrar cuentas, bases de datos y contenedores de Cosmos DB para MongoDB, Gremlin, Cassandra y Table API.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este tema es preciso que ejecute la CLI de Azure versión 2.9.1 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="azure-cosmos-accounts"></a>Cuentas de Azure Cosmos

En las secciones siguientes se muestra cómo administrar la cuenta de Azure Cosmos, entre ellas:

* [Creación de una cuenta de Azure Cosmos](#create-an-azure-cosmos-db-account)
* [Agregar o quitar regiones](#add-or-remove-regions)
* [Habilitación de escrituras en varias regiones](#enable-multiple-write-regions)
* [Establecimiento de la prioridad de conmutación por error regional](#set-failover-priority)
* [Habilitación de la conmutación automática por error](#enable-automatic-failover)
* [Desencadenamiento de una conmutación por error manual](#trigger-manual-failover)
* [Enumeración de claves de cuenta](#list-account-keys)
* [Enumeración de claves de cuenta de solo lectura](#list-read-only-account-keys)
* [Enumeración de cadenas de conexión](#list-connection-strings)
* [Regeneración de la clave de cuenta](#regenerate-account-key)

### <a name="create-an-azure-cosmos-db-account"></a>Creación de una cuenta de Azure Cosmos DB

Cree una cuenta de Azure Cosmos DB con SQL API, coherencia de sesión en las regiones Oeste de EE. UU. 2 y Este de EE. UU. 2:

> [!IMPORTANT]
> El nombre de la cuenta de Azure Cosmos debe estar en minúscula y tener menos de 44 caracteres.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount' #needs to be lower case and less than 44 characters

az cosmosdb create \
    -n $accountName \
    -g $resourceGroupName \
    --default-consistency-level Session \
    --locations regionName='West US 2' failoverPriority=0 isZoneRedundant=False \
    --locations regionName='East US 2' failoverPriority=1 isZoneRedundant=False
```

### <a name="add-or-remove-regions"></a>Agregar o quitar regiones

Cree una cuenta de Azure Cosmos con dos regiones, agregue una región y quite otra.

> [!NOTE]
> En una cuenta de Azure Cosmos no es posible agregar o quitar ubicaciones `locations` y, al mismo tiempo, cambiar otras propiedades. La modificación de regiones debe realizarse como una operación independiente, nunca a la vez que otro cambio en el recurso de la cuenta.
> [!NOTE]
> Este comando permite agregar y quitar regiones, pero no permite modificar las prioridades de conmutación por error ni desencadenar una conmutación por error manual. Consulte [Establecimiento de la prioridad de conmutación por error](#set-failover-priority) y [Desencadenamiento de una conmutación por error manual](#trigger-manual-failover).

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Create an account with 2 regions
az cosmosdb create --name $accountName --resource-group $resourceGroupName \
    --locations regionName="West US 2" failoverPriority=0 isZoneRedundant=False \
    --locations regionName="East US 2" failoverPriority=1 isZoneRedundant=False

# Add a region
az cosmosdb update --name $accountName --resource-group $resourceGroupName \
    --locations regionName="West US 2" failoverPriority=0 isZoneRedundant=False \
    --locations regionName="East US 2" failoverPriority=1 isZoneRedundant=False \
    --locations regionName="South Central US" failoverPriority=2 isZoneRedundant=False

# Remove a region
az cosmosdb update --name $accountName --resource-group $resourceGroupName \
    --locations regionName="West US 2" failoverPriority=0 isZoneRedundant=False \
    --locations regionName="East US 2" failoverPriority=1 isZoneRedundant=False
```

### <a name="enable-multiple-write-regions"></a>Habilitar regiones de varias escrituras

Habilite la arquitectura multimaestro para una cuenta de Cosmos.

```azurecli-interactive
# Update an Azure Cosmos account from single to multi-master
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Get the account resource id for an existing account
accountId=$(az cosmosdb show -g $resourceGroupName -n $accountName --query id -o tsv)

az cosmosdb update --ids $accountId --enable-multiple-write-locations true
```

### <a name="set-failover-priority"></a>Establecimiento de la prioridad de conmutación por error

Establezca la prioridad de conmutación por error para una cuenta de Cosmos configurada para la conmutación automática por error.

```azurecli-interactive
# Assume region order is initially 'West US 2'=0 'East US 2'=1 'South Central US'=2 for account
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Get the account resource id for an existing account
accountId=$(az cosmosdb show -g $resourceGroupName -n $accountName --query id -o tsv)

# Make South Central US the next region to fail over to instead of East US 2
az cosmosdb failover-priority-change --ids $accountId \
    --failover-policies 'West US 2=0' 'South Central US=1' 'East US 2=2'
```

### <a name="enable-automatic-failover"></a>Habilitar la conmutación automática por error

```azurecli-interactive
# Enable automatic failover on an existing account
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Get the account resource id for an existing account
accountId=$(az cosmosdb show -g $resourceGroupName -n $accountName --query id -o tsv)

az cosmosdb update --ids $accountId --enable-automatic-failover true
```

### <a name="trigger-manual-failover"></a>Desencadenamiento de una conmutación por error manual

> [!CAUTION]
> Al cambiar una región con una prioridad = 0, se desencadenará una conmutación por error manual para una cuenta de Azure Cosmos. Los restantes cambios de prioridad no desencadenarán ninguna conmutación por error.

```azurecli-interactive
# Assume region order is initially 'West US 2=0' 'East US 2=1' 'South Central US=2' for account
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'

# Get the account resource id for an existing account
accountId=$(az cosmosdb show -g $resourceGroupName -n $accountName --query id -o tsv)

# Trigger a manual failover to promote East US 2 as new write region
az cosmosdb failover-priority-change --ids $accountId \
    --failover-policies 'East US 2=0' 'South Central US=1' 'West US 2=2'
```

### <a name="list-all-account-keys"></a><a id="list-account-keys"></a> Enumeración de todas las claves de cuenta

Obtenga todas las claves de una cuenta de Cosmos.

```azurecli-interactive
# List all account keys
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'

az cosmosdb keys list \
   -n $accountName \
   -g $resourceGroupName
```

### <a name="list-read-only-account-keys"></a>Enumeración de claves de cuenta de solo lectura

Obtenga claves de solo lectura de una cuenta de Cosmos.

```azurecli-interactive
# List read-only account keys
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'

az cosmosdb keys list \
    -n $accountName \
    -g $resourceGroupName \
    --type read-only-keys
```

### <a name="list-connection-strings"></a>Enumeración de cadenas de conexión

Obtenga las cadenas de conexión para una cuenta de Cosmos.

```azurecli-interactive
# List connection strings
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'

az cosmosdb keys list \
    -n $accountName \
    -g $resourceGroupName \
    --type connection-strings
```

### <a name="regenerate-account-key"></a>Regeneración de la clave de cuenta

Vuelva a generar una nueva clave para una cuenta de Cosmos.

```azurecli-interactive
# Regenerate secondary account keys
# key-kind values: primary, primaryReadonly, secondary, secondaryReadonly
az cosmosdb keys regenerate \
    -n $accountName \
    -g $resourceGroupName \
    --key-kind secondary
```

## <a name="azure-cosmos-db-database"></a>Base de datos de Azure Cosmos DB

En las siguientes secciones se muestra cómo se administra la base de datos de Azure Cosmos DB, lo que incluye:

* [Creación de una base de datos](#create-a-database)
* [Creación de una base de datos con capacidad de proceso compartida](#create-a-database-with-shared-throughput)
* [Cambio del rendimiento de una base de datos](#change-database-throughput)
* [Administración de los bloqueos en una base de datos](#manage-lock-on-a-database)

### <a name="create-a-database"></a>Crear una base de datos

Cree una base de datos de Cosmos.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'

az cosmosdb sql database create \
    -a $accountName \
    -g $resourceGroupName \
    -n $databaseName
```

### <a name="create-a-database-with-shared-throughput"></a>Creación de una base de datos con capacidad de proceso compartida

Cree una base de datos de Cosmos con capacidad de proceso compartida.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
throughput=400

az cosmosdb sql database create \
    -a $accountName \
    -g $resourceGroupName \
    -n $databaseName \
    --throughput $throughput
```

### <a name="change-database-throughput"></a>Cambio del rendimiento de una base de datos

Aumente la capacidad de proceso de una base de datos de Cosmos en 1000 RU/s.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
newRU=1000

# Get minimum throughput to make sure newRU is not lower than minRU
minRU=$(az cosmosdb sql database throughput show \
    -g $resourceGroupName -a $accountName -n $databaseName \
    --query resource.minimumThroughput -o tsv)

if [ $minRU -gt $newRU ]; then
    newRU=$minRU
fi

az cosmosdb sql database throughput update \
    -a $accountName \
    -g $resourceGroupName \
    -n $databaseName \
    --throughput $newRU
```

### <a name="manage-lock-on-a-database"></a>Administración del bloqueo en una base de datos

Coloque un bloqueo de eliminación en una base de datos. Para más información sobre cómo habilitar esto, consulte [Evitar cambios de SDK](role-based-access-control.md#prevent-sdk-changes).

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='my-cosmos-account'
databaseName='myDatabase'

lockType='CanNotDelete' # CanNotDelete or ReadOnly
databaseParent="databaseAccounts/$accountName"
databaseLockName="$databaseName-Lock"

# Create a delete lock on database
az lock create --name $databaseLockName \
    --resource-group $resourceGroupName \
    --resource-type Microsoft.DocumentDB/sqlDatabases \
    --lock-type $lockType \
    --parent $databaseParent \
    --resource $databaseName

# Delete lock on database
lockid=$(az lock show --name $databaseLockName \
        --resource-group $resourceGroupName \
        --resource-type Microsoft.DocumentDB/sqlDatabases \
        --resource $databaseName \
        --parent $databaseParent \
        --output tsv --query id)
az lock delete --ids $lockid
```

## <a name="azure-cosmos-db-container"></a>Contenedor de Azure Cosmos DB

En las siguientes secciones se muestra cómo administrar el contenedor de Azure Cosmos DB, lo que incluye:

* [Creación de un contenedor](#create-a-container)
* [Creación de un contenedor con escalabilidad automática](#create-a-container-with-autoscale)
* [Creación de un contenedor con TTL habilitado](#create-a-container-with-ttl)
* [Creación de un contenedor con una directiva de índice personalizada](#create-a-container-with-a-custom-index-policy)
* [Cambio del rendimiento del contenedor](#change-container-throughput)
* [Administración de bloqueos en un contenedor](#manage-lock-on-a-container)

### <a name="create-a-container"></a>Crear un contenedor

Cree un contenedor de Cosmos con una directiva de índice predeterminada, una clave de partición y 400 RU/s.

```azurecli-interactive
# Create a SQL API container
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'
partitionKey='/myPartitionKey'
throughput=400

az cosmosdb sql container create \
    -a $accountName -g $resourceGroupName \
    -d $databaseName -n $containerName \
    -p $partitionKey --throughput $throughput
```

### <a name="create-a-container-with-autoscale"></a>Creación de un contenedor con escalabilidad automática

Cree un contenedor de Cosmos con una directiva de índice predeterminada, una clave de partición y 4000 RU/s de escalabilidad automática.

```azurecli-interactive
# Create a SQL API container
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'
partitionKey='/myPartitionKey'
maxThroughput=4000

az cosmosdb sql container create \
    -a $accountName -g $resourceGroupName \
    -d $databaseName -n $containerName \
    -p $partitionKey --max-throughput $maxThroughput
```

### <a name="create-a-container-with-ttl"></a>Creación de un contenedor con TTL

Cree un contenedor de Cosmos con TTL habilitado.

```azurecli-interactive
# Create an Azure Cosmos container with TTL of one day
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'

az cosmosdb sql container update \
    -g $resourceGroupName \
    -a $accountName \
    -d $databaseName \
    -n $containerName \
    --ttl=86400
```

### <a name="create-a-container-with-a-custom-index-policy"></a>Creación de un contenedor con directiva de índice personalizada

Cree un contenedor de Cosmos con una directiva de índice personalizada, un índice espacial, un índice compuesto, una clave de partición y 400 RU/s.

```azurecli-interactive
# Create a SQL API container
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'
partitionKey='/myPartitionKey'
throughput=400

# Generate a unique 10 character alphanumeric string to ensure unique resource names
uniqueId=$(env LC_CTYPE=C tr -dc 'a-z0-9' < /dev/urandom | fold -w 10 | head -n 1)

# Define the index policy for the container, include spatial and composite indexes
idxpolicy=$(cat << EOF
{
    "indexingMode": "consistent",
    "includedPaths": [
        {"path": "/*"}
    ],
    "excludedPaths": [
        { "path": "/headquarters/employees/?"}
    ],
    "spatialIndexes": [
        {"path": "/*", "types": ["Point"]}
    ],
    "compositeIndexes":[
        [
            { "path":"/name", "order":"ascending" },
            { "path":"/age", "order":"descending" }
        ]
    ]
}
EOF
)
# Persist index policy to json file
echo "$idxpolicy" > "idxpolicy-$uniqueId.json"


az cosmosdb sql container create \
    -a $accountName -g $resourceGroupName \
    -d $databaseName -n $containerName \
    -p $partitionKey --throughput $throughput \
    --idx @idxpolicy-$uniqueId.json

# Clean up temporary index policy file
rm -f "idxpolicy-$uniqueId.json"
```

### <a name="change-container-throughput"></a>Cambio del rendimiento del contenedor

Aumente la capacidad de procesamiento de un contenedor de Cosmos en 1000 RU/s.

```azurecli-interactive
resourceGroupName='MyResourceGroup'
accountName='mycosmosaccount'
databaseName='database1'
containerName='container1'
newRU=1000

# Get minimum throughput to make sure newRU is not lower than minRU
minRU=$(az cosmosdb sql container throughput show \
    -g $resourceGroupName -a $accountName -d $databaseName \
    -n $containerName --query resource.minimumThroughput -o tsv)

if [ $minRU -gt $newRU ]; then
    newRU=$minRU
fi

az cosmosdb sql container throughput update \
    -a $accountName \
    -g $resourceGroupName \
    -d $databaseName \
    -n $containerName \
    --throughput $newRU
```

### <a name="manage-lock-on-a-container"></a>Administración del bloqueo de un contenedor

Coloque un bloqueo de eliminación en un contenedor. Para más información sobre cómo habilitar esto, consulte [Evitar cambios de SDK](role-based-access-control.md#prevent-sdk-changes).

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='my-cosmos-account'
databaseName='myDatabase'
containerName='myContainer'

lockType='CanNotDelete' # CanNotDelete or ReadOnly
databaseParent="databaseAccounts/$accountName"
containerParent="databaseAccounts/$accountName/sqlDatabases/$databaseName"
containerLockName="$containerName-Lock"

# Create a delete lock on container
az lock create --name $containerLockName \
    --resource-group $resourceGroupName \
    --resource-type Microsoft.DocumentDB/containers \
    --lock-type $lockType \
    --parent $containerParent \
    --resource $containerName

# Delete lock on container
lockid=$(az lock show --name $containerLockName \
        --resource-group $resourceGroupName \
        --resource-type Microsoft.DocumentDB/containers \
        --resource-name $containerName \
        --parent $containerParent \
        --output tsv --query id)
az lock delete --ids $lockid
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte:

- [Instalación de la CLI de Azure](/cli/azure/install-azure-cli)
- [Referencia de CLI de Azure](https://docs.microsoft.com/cli/azure/cosmosdb)
- [Ejemplos adicionales de la CLI de Azure para Azure Cosmos DB](cli-samples.md)
