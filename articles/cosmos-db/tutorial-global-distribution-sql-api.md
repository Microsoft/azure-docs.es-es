---
title: 'Tutorial: Tutorial de distribución global de Azure Cosmos DB para API de SQL'
description: 'Tutorial: Información sobre cómo configurar la distribución global de Azure Cosmos DB mediante SQL API con .NET, Java, Python y otros SDK'
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.topic: tutorial
ms.date: 11/05/2019
ms.reviewer: sngun
ms.custom: devx-track-python, devx-track-javascript
ms.openlocfilehash: 9498c4a0e0dc5b528066ffadd6a9638fe99b1135
ms.sourcegitcommit: dea88d5e28bd4bbd55f5303d7d58785fad5a341d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87874799"
---
# <a name="tutorial-set-up-azure-cosmos-db-global-distribution-using-the-sql-api"></a>Tutorial: Configuración de la distribución global de Azure Cosmos DB con SQL API

En este artículo se muestra cómo usar Azure Portal para configurar la distribución global de Azure Cosmos DB y luego conectarse con la API de SQL.

En este artículo se tratan las tareas siguientes: 

> [!div class="checklist"]
> * Configuración de la distribución global con Azure Portal
> * Configuración de la distribución global con las [API de SQL](sql-api-introduction.md)

<a id="portal"></a>
[!INCLUDE [cosmos-db-tutorial-global-distribution-portal](../../includes/cosmos-db-tutorial-global-distribution-portal.md)]

## <a name="connecting-to-a-preferred-region-using-the-sql-api"></a><a id="preferred-locations"></a> Conexión a una región de preferencia con la API de SQL

Para aprovechar las ventajas de la [distribución global](distribute-data-globally.md), las aplicaciones cliente pueden especificar la lista del orden de preferencia de regiones que se usará para llevar a cabo operaciones de documentos. Según la configuración de la cuenta de Azure Cosmos DB, la disponibilidad regional actual y la lista de preferencias especificada, el SDK de SQL elegirá el punto de conexión óptimo para realizar las operaciones de lectura y escritura.

Esta lista de preferencias se especifica cuando se inicializa una conexión mediante los SDK de SQL. Los SDK aceptan un parámetro opcional `PreferredLocations`, que es una lista ordenada de regiones de Azure.

El SDK enviará automáticamente todas las escrituras a la región de escritura actual. Todas las lecturas se enviarán a la primera región disponible de la lista de ubicaciones preferidas. Si se produce un error en la solicitud, el cliente pasará a la siguiente región de la lista.

Los SDK solo intentarán leer en las regiones especificadas en las ubicaciones preferidas. Así que, por ejemplo, si la cuenta de Azure Cosmos DB está disponible en cuatro regiones, pero el cliente solo especifica dos de las regiones de lectura (no escritura) en `PreferredLocations`, no se atenderá ninguna lectura desde la región de lectura que no se especifique en `PreferredLocations`. Si las regiones de lectura especificadas en `PreferredLocations` no están disponibles, las lecturas se atenderán fuera de la región de escritura.

La aplicación puede comprobar los puntos de conexión de escritura y lectura actuales elegidos por el SDK. Para ello, comprueba dos propiedades, `WriteEndpoint` y `ReadEndpoint`, disponibles en el SDK 1.8 y versiones posteriores. Si la propiedad `PreferredLocations` no está establecida, atenderá todas las solicitudes procedentes de la región de escritura actual.

Si no se especifican las ubicaciones preferidas pero se usa el método `setCurrentLocation`, el SDK rellena automáticamente las ubicaciones preferidas en función de la región actual en la que se ejecute el cliente. El SDK ordena las regiones en función de la proximidad de una región a la región actual.

## <a name="net-sdk"></a>.NET SDK

Se puede usar el SDK sin cambiar el código. En este caso, el SDK dirige automáticamente tanto las lecturas como las escrituras a la región de escritura actual.

En la versión 1.8 y posteriores del SDK de .NET, el parámetro ConnectionPolicy para el constructor DocumentClient tiene una propiedad denominada Microsoft.Azure.Documents.ConnectionPolicy.PreferredLocations. Esta propiedad es del tipo Collection `<string>` y debería contener una lista de nombres de región. Se da formato a los valores de cadena según la columna de nombre de región en la página [Regiones de Azure][regions], sin espacios antes ni después del primer o el último carácter, respectivamente.

Los puntos de conexión de lectura y escritura actuales están disponibles en DocumentClient.ReadEndpoint y DocumentClient.WriteEndpoint, respectivamente.

> [!NOTE]
> Las direcciones URL de los puntos de conexión no deben considerarse constantes de larga duración. El servicio puede actualizarlas en cualquier momento. El SDK administra este cambio automáticamente.
>

# <a name="net-sdk-v2"></a>[SDK para .NET V2](#tab/dotnetv2)

Si emplea el SDK de .NET v2, use la propiedad `PreferredLocations` para establecer la región preferida.

```csharp
// Getting endpoints from application settings or other configuration location
Uri accountEndPoint = new Uri(Properties.Settings.Default.GlobalDatabaseUri);
string accountKey = Properties.Settings.Default.GlobalDatabaseKey;
  
ConnectionPolicy connectionPolicy = new ConnectionPolicy();

//Setting read region selection preference
connectionPolicy.PreferredLocations.Add(LocationNames.WestUS); // first preference
connectionPolicy.PreferredLocations.Add(LocationNames.EastUS); // second preference
connectionPolicy.PreferredLocations.Add(LocationNames.NorthEurope); // third preference

// initialize connection
DocumentClient docClient = new DocumentClient(
    accountEndPoint,
    accountKey,
    connectionPolicy);

// connect to DocDB
await docClient.OpenAsync().ConfigureAwait(false);
```

Como alternativa, puede usar la propiedad `SetCurrentLocation` y dejar que el SDK elija la ubicación preferida en función de la proximidad.

```csharp
// Getting endpoints from application settings or other configuration location
Uri accountEndPoint = new Uri(Properties.Settings.Default.GlobalDatabaseUri);
string accountKey = Properties.Settings.Default.GlobalDatabaseKey;
  
ConnectionPolicy connectionPolicy = new ConnectionPolicy();

connectionPolicy.SetCurrentLocation("West US 2"); /

// initialize connection
DocumentClient docClient = new DocumentClient(
    accountEndPoint,
    accountKey,
    connectionPolicy);

// connect to DocDB
await docClient.OpenAsync().ConfigureAwait(false);
```

# <a name="net-sdk-v3"></a>[SDK para .NET V3](#tab/dotnetv3)

Si emplea el SDK de .NET v3, use la propiedad `ApplicationPreferredRegions` para establecer la región preferida.

```csharp

CosmosClientOptions options = new CosmosClientOptions();
options.ApplicationName = "MyApp";
options.ApplicationPreferredRegions = new List<string> {Regions.WestUS, Regions.WestUS2};

CosmosClient client = new CosmosClient(connectionString, options);

```

Como alternativa, puede usar la propiedad `ApplicationRegion` y dejar que el SDK elija la ubicación preferida en función de la proximidad.

```csharp
CosmosClientOptions options = new CosmosClientOptions();
options.ApplicationName = "MyApp";
// If the application is running in West US
options.ApplicationRegion = Regions.WestUS;

CosmosClient client = new CosmosClient(connectionString, options);
```

---

## <a name="nodejsjavascript"></a>Node.js o JavaScript

> [!NOTE]
> Las direcciones URL de los puntos de conexión no deben considerarse constantes de larga duración. El servicio puede actualizarlas en cualquier momento. El SDK administrará este cambio automáticamente.
>
>

A continuación, se muestra un ejemplo de código para Node.js y JavaScript.

```JavaScript
// Setting read region selection preference, in the following order -
// 1 - West US
// 2 - East US
// 3 - North Europe
const preferredLocations = ['West US', 'East US', 'North Europe'];

// initialize the connection
const client = new CosmosClient{ endpoint, key, connectionPolicy: { preferredLocations } });
```

## <a name="python-sdk"></a>SDK de Python

El siguiente código muestra cómo establecer ubicaciones preferidas con el SDK de Python:

```python
connectionPolicy = documents.ConnectionPolicy()
connectionPolicy.PreferredLocations = ['West US', 'East US', 'North Europe']
client = cosmos_client.CosmosClient(ENDPOINT, {'masterKey': MASTER_KEY}, connectionPolicy)

```

## <a name="java-v4-sdk"></a><a id="java4-preferred-locations"></a> SDK de Java V4

El siguiente código muestra cómo establecer las ubicaciones preferidas con el SDK de Java:

# <a name="async"></a>[Async](#tab/api-async)

   [SDK para Java V4](sql-api-sdk-java-v4.md) (Maven [com.azure::azure-cosmos](https://mvnrepository.com/artifact/com.azure/azure-cosmos)) API asincrónica

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=TutorialGlobalDistributionPreferredLocationAsync)]

# <a name="sync"></a>[Sincronizar](#tab/api-sync)

   [SDK para Java V4](sql-api-sdk-java-v4.md) (Maven [com.azure::azure-cosmos](https://mvnrepository.com/artifact/com.azure/azure-cosmos)) API sincrónica

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=TutorialGlobalDistributionPreferredLocationSync)]

--- 

## <a name="rest"></a>REST

Una vez que una cuenta de base de datos está disponible en varias regiones, los clientes pueden consultar su disponibilidad llevando a cabo una solicitud GET en este URI `https://{databaseaccount}.documents.azure.com/`.

El servicio devolverá una lista de regiones y los URI de punto de conexión de Azure Cosmos DB correspondientes para las réplicas. Se indicará la región de escritura actual en la respuesta. Después, el cliente puede seleccionar el punto de conexión adecuado para las demás solicitudes de API de REST como sigue.

Respuesta de ejemplo

```json
{
    "_dbs": "//dbs/",
    "media": "//media/",
    "writableLocations": [
        {
            "Name": "West US",
            "DatabaseAccountEndpoint": "https://globaldbexample-westus.documents.azure.com:443/"
        }
    ],
    "readableLocations": [
        {
            "Name": "East US",
            "DatabaseAccountEndpoint": "https://globaldbexample-eastus.documents.azure.com:443/"
        }
    ],
    "MaxMediaStorageUsageInMB": 2048,
    "MediaStorageUsageInMB": 0,
    "ConsistencyPolicy": {
        "defaultConsistencyLevel": "Session",
        "maxStalenessPrefix": 100,
        "maxIntervalInSeconds": 5
    },
    "addresses": "//addresses/",
    "id": "globaldbexample",
    "_rid": "globaldbexample.documents.azure.com",
    "_self": "",
    "_ts": 0,
    "_etag": null
}
```

* Todas las solicitudes PUT, POST y DELETE deben ir al URI de escritura indicado.
* Todas las solicitudes GET y otras solicitudes de solo lectura (como las consultas) pueden ir a cualquier punto de conexión elegido por el cliente.

Las solicitudes de escritura a regiones de solo lectura producirán un error con el código de error HTTP 403 ("Prohibido").

Si la región de escritura cambia después de la fase de descubrimiento inicial del cliente, se producirá un error en las escrituras posteriores en la región de escritura anterior con el código de error HTTP 403 ("Prohibido"). A continuación, el cliente debería de nuevo conseguir con GET la lista de regiones para obtener la región de escritura actualizada.

De este modo finaliza este tutorial. Para información sobre cómo administrar la coherencia de la cuenta replicada globalmente, lea [Niveles de coherencia en Azure Cosmos DB](consistency-levels.md). Para más información sobre cómo funciona la replicación global de bases de datos en Azure Cosmos DB, consulte [Distribución global de datos con Azure Cosmos DB](distribute-data-globally.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha hecho lo siguiente:

> [!div class="checklist"]
> * Configuración de la distribución global con Azure Portal
> * Configuración de la distribución global con las API de SQL

Ahora puede continuar en el tutorial siguiente para aprender a desarrollar localmente con el emulador local de Azure Cosmos DB.

> [!div class="nextstepaction"]
> [Desarrollo local con el emulador](local-emulator.md)

[regions]: https://azure.microsoft.com/regions/

