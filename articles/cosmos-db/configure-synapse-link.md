---
title: Configuración y uso de Azure Synapse Link para Azure Cosmos DB (versión preliminar)
description: Obtenga información sobre cómo habilitar Synapse Link para las cuentas de Azure Cosmos, crear un contenedor con el almacén analítico habilitado, conectar la base de datos de Azure Cosmos a un área de trabajo de Synapse y ejecutar consultas.
author: Rodrigossz
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/19/2020
ms.author: rosouz
ms.openlocfilehash: 4c5f812bf1a5a60a6d1344d6a39fbd95898f55fc
ms.sourcegitcommit: d39f2cd3e0b917b351046112ef1b8dc240a47a4f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88815579"
---
# <a name="configure-and-use-azure-synapse-link-for-azure-cosmos-db-preview"></a>Configuración y uso de Azure Synapse Link para Azure Cosmos DB (versión preliminar)

Synapse Link para Azure Cosmos DB es una funcionalidad de procesamiento analítico y transaccional híbrido nativo de nube (HTAP) que permite ejecutar análisis casi en tiempo real de datos operativos en Azure Cosmos DB. Synapse Link crea una integración perfecta y sin contratiempos entre Azure Cosmos DB y Azure Synapse Analytics.


> [!IMPORTANT]
> Para usar Azure Synapse Link, asegúrese de que está aprovisionando su cuenta de Azure Cosmos y el área de trabajo de Azure Synapse Analytics en alguna de las regiones compatibles. Azure Synapse Link actualmente está disponible en las siguientes regiones de Azure: Centro-oeste de EE. UU., Este de EE. UU., Oeste de EE. UU. 2, Norte de Europa, Oeste de Europa, Centro-sur de EE. UU., Sudeste de Asia, Este de Australia, Este de EE. UU. 2, Sur de Reino Unido.

Siga estos pasos para ejecutar consultas analíticas con Synapse Link para Azure Cosmos DB:

* [Habilitación de Synapse Link para las cuentas de Azure Cosmos](#enable-synapse-link)
* [Creación de un contenedor de Azure Cosmos habilitado para el almacén analítico](#create-analytical-ttl)
* [Conexión de la base de datos de Azure Cosmos a un área de trabajo de Synapse](#connect-to-cosmos-database)
* [Consulta del almacén analítico mediante Spark en Synapse](#query-analytical-store)

## <a name="enable-azure-synapse-link-for-azure-cosmos-accounts"></a><a id="enable-synapse-link"></a>Habilitación de Azure Synapse Link para cuentas de Azure Cosmos

### <a name="azure-portal"></a>Azure portal

1. Inicie sesión en el [Portal de Azure](https://portal.azure.com/).

1. [Cree una nueva cuenta de Azure](create-sql-api-dotnet.md#create-account), o bien seleccione una cuenta de Azure Cosmos ya existente.

1. Vaya a la cuenta de Azure Cosmos y abra el panel **Características**.

1. Seleccione **Synapse Link** en la lista de características.

   :::image type="content" source="./media/configure-synapse-link/find-synapse-link-feature.png" alt-text="Búsqueda de la característica en vista previa Synapse Link":::

1. A continuación, se le pedirá que habilite Synapse Link en su cuenta. Seleccione Habilitar.

   :::image type="content" source="./media/configure-synapse-link/enable-synapse-link-feature.png" alt-text="Habilitación de la característica Synapse Link":::

1. Ahora su cuenta está habilitada para usar Synapse Link. A continuación, consulte cómo crear contenedores habilitados para el almacén analítico para iniciar de forma automática la replicación de sus datos operativos desde el almacén transaccional hacia el almacén analítico.

### <a name="azure-resource-manager-template"></a>Plantilla del Administrador de recursos de Azure

La [plantilla de Azure Resource Manager](manage-sql-with-resource-manager.md#azure-cosmos-account-with-analytical-store) crea una cuenta de Azure Cosmos habilitada para Synapse Link para la API de SQL. Esta plantilla crea una cuenta de Core (SQL) API en una región con un contenedor configurado con TTL analítico habilitado y una opción para usar la capacidad de proceso manual o de escalado automático. Para implementar esta plantilla, haga clic en **Implementar en Azure** en la página Léame.

## <a name="create-an-azure-cosmos-container-with-analytical-store"></a><a id="create-analytical-ttl"></a>Creación de un contenedor de Azure Cosmos con un almacén analítico

Puede activar el almacén analítico en un contenedor de Azure Cosmos durante su creación. Puede usar Azure Portal o configurar la propiedad `analyticalTTL` durante la creación del contenedor mediante los SDK de Azure Cosmos DB.

> [!NOTE]
> En este momento, puede habilitar el almacén analítico para los **nuevos** contenedores (tanto en cuentas nuevas como ya existentes).

### <a name="azure-portal"></a>Azure portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) o en el [explorador de Azure Cosmos](https://cosmos.azure.com/).

1. Vaya a la cuenta de Azure Cosmos y abra la pestaña **Explorador de datos**.

1. Seleccione **Nuevo contenedor** y escriba un nombre para la base de datos, el contenedor, la clave de partición y los detalles de capacidad de proceso. Active la opción **Analytical store** (Almacén analítico). Después de habilitar el almacén analítico, se crea un contenedor con la propiedad `AnalyicalTTL` establecida en el valor predeterminado de -1 (retención de datos infinita). Este almacén analítico conserva todas las versiones históricas de los registros.

   :::image type="content" source="./media/configure-synapse-link/create-container-analytical-store.png" alt-text="Activación de un almacén analítico para un contenedor de Azure Cosmos":::

1. Si no ha habilitado Synapse Link anteriormente en esta cuenta, se le pedirá que lo haga, ya que se trata de un requisito previo para crear un contenedor habilitado para almacén analítico. Si se le solicita, seleccione **Enable Synapse Link** (Habilitar Synapse Link).

1. Seleccione **Aceptar** para crear un contenedor de Azure Cosmos habilitado para el almacén analítico.

### <a name="net-sdk"></a>.NET SDK

En el código siguiente se crea un contenedor con un almacén analítico mediante el SDK de .NET. Establezca la propiedad de TTL analítico en el valor requerido. Para ver la lista de valores permitidos, consulte el artículo [Valores de TTL analítico admitidos](analytical-store-introduction.md#analytical-ttl):

```csharp
// Create a container with a partition key, and analytical TTL configured to  -1 (infinite retention)
string containerId = “myContainerName”;
int analyticalTtlInSec = -1;
ContainerProperties cpInput = new ContainerProperties()
            {
Id = containerId,
PartitionKeyPath = "/id",
AnalyticalStorageTimeToLiveInSeconds = analyticalTtlInSec,
};
 await this. cosmosClient.GetDatabase("myDatabase").CreateContainerAsync(cpInput);
```

### <a name="java-v4-sdk"></a>SDK de Java V4

En el código siguiente se crea un contenedor con un almacén analítico mediante el SDK de Java V4. Establezca la propiedad `AnalyticalStoreTimeToLiveInSeconds` en el valor requerido:

```java
// Create a container with a partition key and  analytical TTL configured to  -1 (infinite retention) 
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");

containerProperties.setAnalyticalStoreTimeToLiveInSeconds(-1);

container = database.createContainerIfNotExists(containerProperties, 400).block().getContainer();
```

### <a name="python-v4-sdk"></a>SDK para Python v4

El SDK para Python 2.7 y el SDK de Azure Cosmos DB 4.1.0 son las versiones mínimas necesarias, y el SDK solo es compatible con la API de SQL.

El primer paso consiste en asegurarse de que está usando como mínimo la versión 4.1.0 del SDK de [Azure Cosmos DB para Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/cosmos/azure-cosmos):

```python
import azure.cosmos as cosmos

print (cosmos.__version__)
```
En el siguiente paso se crea un contenedor con un almacén analítico mediante el SDK de Azure Cosmos DB para Python:

```python
# Azure Cosmos DB Python SDK, for SQL API only.
# Creating an analytical store enabled container.

import azure.cosmos.cosmos_client as cosmos_client
import azure.cosmos.exceptions as exceptions
from azure.cosmos.partition_key import PartitionKey

HOST = 'your-cosmos-db-account-URI'
KEY = 'your-cosmos-db-account-key'
DATABASE = 'your-cosmos-db-database-name'
CONTAINER = 'your-cosmos-db-container-name'

client = cosmos_client.CosmosClient(HOST,  KEY )
# setup database for this sample. 
# If doesn't exist, creates a new one with the name informed above.
try:
    db = client.create_database(DATABASE)

except exceptions.CosmosResourceExistsError:
    db = client.get_database_client(DATABASE)

# Creating the container with analytical store enabled, using the name informed above.
# If a container with the same name exists, an error is returned.
#
# The 3 options for the analytical_storage_ttl parameter are:
# 1) 0 or Null or not informed (Not enabled).
# 2) -1 (The data will be stored in analytical store infinitely).
# 3) Any other number is the actual ttl, in seconds.

try:
    container = db.create_container(
        id=CONTAINER,
        partition_key=PartitionKey(path='/id', kind='Hash'),analytical_storage_ttl=-1
    )
    properties = container.read()
    print('Container with id \'{0}\' created'.format(container.id))
    print('Partition Key - \'{0}\''.format(properties['partitionKey']))

except exceptions.CosmosResourceExistsError:
    print('A container with already exists')
```

### <a name="update-the-analytical-store-time-to-live"></a><a id="update-analytical-ttl"></a> Actualización del período de vida del almacén analítico

Después de habilitar el almacén analítico con un valor de TTL determinado, puede actualizarlo a un valor válido diferente más tarde. Puede actualizar el valor mediante Azure Portal o los SDK. Para obtener información sobre las distintas opciones de configuración de TTL analítico, consulte el artículo [valores de TTL analítico admitidos](analytical-store-introduction.md#analytical-ttl).

#### <a name="azure-portal"></a>Azure portal

Si creó un contenedor habilitado para el almacén analítico mediante Azure Portal, dicho contenedor contiene un TTL analítico predeterminado de -1. Siga estos pasos para actualizar dicho valor:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) o en el [explorador de Azure Cosmos](https://cosmos.azure.com/).

1. Vaya a la cuenta de Azure Cosmos y abra la pestaña **Explorador de datos**.

1. Seleccione un contenedor existente que tenga habilitado el almacén analítico. Expanda el contenedor y modifique los valores siguientes:

  * Abra la ventana **Escala y configuración**.
  * En **Configuración**, busque **Período de vida de almacenamiento analítico**.
  * Seleccione **Activado (valor no predeterminado)** o seleccione **Activado** y establezca un valor para el período de vida
  * Haga clic en **Guardar** para guardar los cambios.

#### <a name="net-sdk"></a>.NET SDK

En el código siguiente se muestra cómo actualizar el TTL para el almacén analítico mediante el SDK de .NET:

```csharp
// Get the container, update AnalyticalStorageTimeToLiveInSeconds 
ContainerResponse containerResponse = await client.GetContainer("database", "container").ReadContainerAsync();
// Update analytical store TTL
containerResponse.Resource. AnalyticalStorageTimeToLiveInSeconds = 60 * 60 * 24 * 180  // Expire analytical store data in 6 months;
await client.GetContainer("database", "container").ReplaceContainerAsync(containerResponse.Resource);
```

#### <a name="java-v4-sdk"></a>SDK de Java V4

En el código siguiente se muestra cómo actualizar el TTL para el almacén analítico mediante el SDK de Java V4:

```java
CosmosContainerProperties containerProperties = new CosmosContainerProperties("myContainer", "/myPartitionKey");

// Update analytical store TTL to expire analytical store data in 6 months;
containerProperties.setAnalyticalStoreTimeToLiveInSeconds (60 * 60 * 24 * 180 );  
 
// Update container settings
container.replace(containerProperties).block();
```

## <a name="connect-to-a-synapse-workspace"></a><a id="connect-to-cosmos-database"></a> Conexión a un área de trabajo de Synapse

Siga las instrucciones del artículo [Conexión a Azure Synapse Link](../synapse-analytics/synapse-link/how-to-connect-synapse-link-cosmos-db.md) sobre cómo acceder a una base de datos de Azure Cosmos DB desde Azure Synapse Analytics Studio con Azure Synapse Link.

## <a name="query-using-synapse-spark"></a><a id="query-analytical-store"></a> Consulta mediante Spark en Synapse

Siga las instrucciones que se indican en el artículo [Consulta al almacén analítico de Azure Cosmos DB](../synapse-analytics/synapse-link/how-to-query-analytical-store-spark.md) sobre cómo realizar consultas con Spark en Synapse. En este artículo se ofrecen algunos ejemplos sobre cómo puede interactuar con el almacén analítico desde los gestos de Synapse. Dichos gestos se muestran cuando se hace clic con el botón secundario en un contenedor. Con ellos, puede generar código rápidamente y adaptarlo a sus necesidades. También son ideales para detectar datos con un solo clic.

## <a name="getting-started-with-azure-synpase-link---samples"></a><a id="cosmosdb-synapse-link-samples"></a> Introducción a Azure Synapse Link: ejemplos

Puede encontrar ejemplos para empezar a trabajar con Azure Synapse Link en [GitHub](https://aka.ms/cosmosdb-synapselink-samples). Estos presentan soluciones integrales con escenarios de IoT y de venta directa.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte la siguiente documentación:

* [Azure Synapse Link para Azure Cosmos DB.](synapse-link.md)

* [Introducción al almacén analítico de Azure Cosmos DB.](analytical-store-introduction.md)

* [Preguntas frecuentes sobre Synapse Link para Azure Cosmos DB.](synapse-link-frequently-asked-questions.md)

* [Apache Spark en Azure Synapse Analytics](../synapse-analytics/spark/apache-spark-concepts.md).

* [SQL sin servidor/a petición en Azure Synapse Analytics](../synapse-analytics/sql/on-demand-workspace-overview.md).
