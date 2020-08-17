---
title: Administración de la coherencia en Azure Cosmos DB
description: Aprenda a configurar y administrar los niveles de coherencia en Azure Cosmos DB mediante Azure Portal, el SDK de .NET, el SDK de Java y otros SDK
author: anfeldma-ms
ms.service: cosmos-db
ms.topic: how-to
ms.date: 06/10/2020
ms.author: anfeldma
ms.custom: devx-track-javascript
ms.openlocfilehash: 48f728e697e3ca92eee0469c8368e966479fe7fb
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87422239"
---
# <a name="manage-consistency-levels-in-azure-cosmos-db"></a>Administración de los niveles de coherencia en Azure Cosmos DB

Este artículo explica cómo administrar los niveles de coherencia en Azure Cosmos DB. Aprenderá a configurar el nivel de coherencia predeterminado, invalidar la coherencia predeterminada, administrar manualmente tokens de sesión y comprender la métrica de obsolescencia limitada por probabilidades (PBS).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="configure-the-default-consistency-level"></a>Configuración del nivel de coherencia predeterminado

El [nivel de coherencia predeterminado](consistency-levels.md) es el que los clientes usan de forma predeterminada.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para ver o modificar el nivel de coherencia predeterminado, inicie sesión en Azure Portal. Busque la cuenta de Azure Cosmos y abra el panel **Coherencia predeterminada**. Seleccione el nivel de coherencia que desee como el nuevo valor predeterminado y, a continuación, seleccione **Guardar**. Azure Portal también proporciona una visualización de los diferentes niveles de coherencia con notas musicales. 

:::image type="content" source="./media/how-to-manage-consistency/consistency-settings.png" alt-text="Menú de coherencia en Azure Portal":::

# <a name="cli"></a>[CLI](#tab/cli)

Cree una cuenta de Cosmos con coherencia de sesión y, después, actualice la coherencia predeterminada.

```azurecli
# Create a new account with Session consistency
az cosmosdb create --name $accountName --resource-group $resourceGroupName --default-consistency-level Session

# update an existing account's default consistency
az cosmosdb update --name $accountName --resource-group $resourceGroupName --default-consistency-level Strong
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Cree una cuenta de Cosmos con coherencia de sesión y, después, actualice la coherencia predeterminada.

```azurepowershell-interactive
# Create a new account with Session consistency
New-AzCosmosDBAccount -ResourceGroupName $resourceGroupName `
  -Location $locations -Name $accountName -DefaultConsistencyLevel "Session"

# Update an existing account's default consistency
Update-AzCosmosDBAccount -ResourceGroupName $resourceGroupName `
  -Name $accountName -DefaultConsistencyLevel "Strong"
```

---

## <a name="override-the-default-consistency-level"></a>Invalidación del nivel de coherencia predeterminado

Los clientes pueden invalidar el nivel de coherencia predeterminado establecido por el servicio. El nivel de coherencia se puede establecer por cada solicitud, lo que invalida el nivel de coherencia predeterminado establecido en el nivel de cuenta.

> [!TIP]
> La coherencia solo se puede **relajar** en el nivel de solicitud. Para pasar de una coherencia más débil a una más fuerte, actualice la coherencia predeterminada para la cuenta de Cosmos.

### <a name="net-sdk"></a><a id="override-default-consistency-dotnet"></a>SDK para .NET

# <a name="net-sdk-v2"></a>[SDK de .NET V2](#tab/dotnetv2)

```csharp
// Override consistency at the client level
documentClient = new DocumentClient(new Uri(endpoint), authKey, connectionPolicy, ConsistencyLevel.Eventual);

// Override consistency at the request level via request options
RequestOptions requestOptions = new RequestOptions { ConsistencyLevel = ConsistencyLevel.Eventual };

var response = await client.CreateDocumentAsync(collectionUri, document, requestOptions);
```

# <a name="net-sdk-v3"></a>[SDK para .NET V3](#tab/dotnetv3)

```csharp
// Override consistency at the request level via request options
ItemRequestOptions requestOptions = new ItemRequestOptions { ConsistencyLevel = ConsistencyLevel.Strong };

var response = await client.GetContainer(databaseName, containerName)
    .CreateItemAsync(
        item,
        new PartitionKey(itemPartitionKey),
        requestOptions);
```
---

### <a name="java-v4-sdk"></a><a id="override-default-consistency-javav4"></a> SDK de Java V4

# <a name="async"></a>[Asincrónico](#tab/api-async)

   API asincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=ManageConsistencyAsync)]

# <a name="sync"></a>[Sincronizar](#tab/api-sync)

   API sincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=ManageConsistencySync)]

--- 

### <a name="java-v2-sdks"></a><a id="override-default-consistency-javav2"></a> SDK de Java V2

# <a name="async"></a>[Asincrónico](#tab/api-async)

SDK de Java v2 asincrónico (Maven com.microsoft.azure::azure-cosmosdb)

```java
// Override consistency at the client level
ConnectionPolicy policy = new ConnectionPolicy();

AsyncDocumentClient client =
        new AsyncDocumentClient.Builder()
                .withMasterKey(this.accountKey)
                .withServiceEndpoint(this.accountEndpoint)
                .withConsistencyLevel(ConsistencyLevel.Eventual)
                .withConnectionPolicy(policy).build();
```

# <a name="sync"></a>[Sincronizar](#tab/api-sync)

SDK de Java V2 sincrónico (Maven com.microsoft.azure::azure-documentdb)

```java
// Override consistency at the client level
ConnectionPolicy connectionPolicy = new ConnectionPolicy();
DocumentClient client = new DocumentClient(accountEndpoint, accountKey, connectionPolicy, ConsistencyLevel.Eventual);
```
---

### <a name="nodejsjavascripttypescript-sdk"></a><a id="override-default-consistency-javascript"></a>SDK para Node.js/JavaScript/TypeScript

```javascript
// Override consistency at the client level
const client = new CosmosClient({
  /* other config... */
  consistencyLevel: ConsistencyLevel.Eventual
});

// Override consistency at the request level via request options
const { body } = await item.read({ consistencyLevel: ConsistencyLevel.Eventual });
```

### <a name="python-sdk"></a><a id="override-default-consistency-python"></a>SDK para Python

```python
# Override consistency at the client level
connection_policy = documents.ConnectionPolicy()
client = cosmos_client.CosmosClient(self.account_endpoint, {
                                    'masterKey': self.account_key}, connection_policy, documents.ConsistencyLevel.Eventual)
```

## <a name="utilize-session-tokens"></a>Uso de tokens de sesión

Uno de los niveles de coherencia de Azure Cosmos DB es *Sesión*. Este es el nivel predeterminado que se aplica a las cuentas de Cosmos de forma predeterminada. Al trabajar con la coherencia de *Sesión*, el cliente usa un token de sesión internamente con cada solicitud de lectura/consulta para garantizar que se mantiene el nivel de coherencia establecido.

Para administrar los tokens de sesión manualmente, obtenga el token de sesión de la respuesta y establézcalos por cada solicitud. Si no tiene la necesidad de administrar manualmente los tokens de sesión, no es necesario que utilice estos ejemplos. El SDK realiza el seguimiento de los tokens de sesión automáticamente. Si no establece el token de sesión manualmente, el SDK usa el token de sesión más reciente de forma predeterminada.

### <a name="net-sdk"></a><a id="utilize-session-tokens-dotnet"></a>SDK para .NET

# <a name="net-sdk-v2"></a>[SDK de .NET V2](#tab/dotnetv2)

```csharp
var response = await client.ReadDocumentAsync(
                UriFactory.CreateDocumentUri(databaseName, collectionName, "SalesOrder1"));
string sessionToken = response.SessionToken;

RequestOptions options = new RequestOptions();
options.SessionToken = sessionToken;
var response = await client.ReadDocumentAsync(
                UriFactory.CreateDocumentUri(databaseName, collectionName, "SalesOrder1"), options);
```

# <a name="net-sdk-v3"></a>[SDK para .NET V3](#tab/dotnetv3)

```csharp
Container container = client.GetContainer(databaseName, collectionName);
ItemResponse<SalesOrder> response = await container.CreateItemAsync<SalesOrder>(salesOrder);
string sessionToken = response.Headers.Session;

ItemRequestOptions options = new ItemRequestOptions();
options.SessionToken = sessionToken;
ItemResponse<SalesOrder> response = await container.ReadItemAsync<SalesOrder>(salesOrder.Id, new PartitionKey(salesOrder.PartitionKey), options);
```
---

### <a name="java-v4-sdk"></a><a id="override-default-consistency-javav4"></a> SDK de Java V4

# <a name="async"></a>[Asincrónico](#tab/api-async)

   API asincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/async/SampleDocumentationSnippetsAsync.java?name=ManageConsistencySessionAsync)]

# <a name="sync"></a>[Sincronizar](#tab/api-sync)

   API sincrónica del SDK para Java V4 (Maven com.azure::azure-cosmos)

   [!code-java[](~/azure-cosmos-java-sql-api-samples/src/main/java/com/azure/cosmos/examples/documentationsnippets/sync/SampleDocumentationSnippets.java?name=ManageConsistencySessionSync)]

--- 

### <a name="java-v2-sdks"></a><a id="utilize-session-tokens-javav2"></a>SDK de Java V2

# <a name="async"></a>[Asincrónico](#tab/api-async)

SDK de Java v2 asincrónico (Maven com.microsoft.azure::azure-cosmosdb)

```java
// Get session token from response
RequestOptions options = new RequestOptions();
options.setPartitionKey(new PartitionKey(document.get("mypk")));
Observable<ResourceResponse<Document>> readObservable = client.readDocument(document.getSelfLink(), options);
readObservable.single()           // we know there will be one response
  .subscribe(
      documentResourceResponse -> {
          System.out.println(documentResourceResponse.getSessionToken());
      },
      error -> {
          System.err.println("an error happened: " + error.getMessage());
      });

// Resume the session by setting the session token on RequestOptions
RequestOptions options = new RequestOptions();
requestOptions.setSessionToken(sessionToken);
Observable<ResourceResponse<Document>> readObservable = client.readDocument(document.getSelfLink(), options);
```

# <a name="sync"></a>[Sincronizar](#tab/api-sync)

SDK de Java V2 sincrónico (Maven com.microsoft.azure::azure-documentdb)

```java
// Get session token from response
ResourceResponse<Document> response = client.readDocument(documentLink, null);
String sessionToken = response.getSessionToken();

// Resume the session by setting the session token on the RequestOptions
RequestOptions options = new RequestOptions();
options.setSessionToken(sessionToken);
ResourceResponse<Document> response = client.readDocument(documentLink, options);
```
---

### <a name="nodejsjavascripttypescript-sdk"></a><a id="utilize-session-tokens-javascript"></a>SDK para Node.js/JavaScript/TypeScript

```javascript
// Get session token from response
const { headers, item } = await container.items.create({ id: "meaningful-id" });
const sessionToken = headers["x-ms-session-token"];

// Immediately or later, you can use that sessionToken from the header to resume that session.
const { body } = await item.read({ sessionToken });
```

### <a name="python-sdk"></a><a id="utilize-session-tokens-python"></a>SDK para Python

```python
// Get the session token from the last response headers
item = client.ReadItem(item_link)
session_token = client.last_response_headers["x-ms-session-token"]

// Resume the session by setting the session token on the options for the request
options = {
    "sessionToken": session_token
}
item = client.ReadItem(doc_link, options)
```

## <a name="monitor-probabilistically-bounded-staleness-pbs-metric"></a>Supervisión de la métrica de obsolescencia limitada de manera probabilística (PBS)

¿Cómo de eventual es la coherencia eventual? Por término medio, se puede ofrecer obsolescencia limitada con respecto al historial de versiones y la hora. La métrica [**Obsolescencia limitada de manera probabilística (PBS)** ](https://pbs.cs.berkeley.edu/) intenta cuantificar la probabilidad de obsolescencia y la muestra como una métrica. Para ver la métrica de PBS, vaya a la cuenta de Azure Cosmos en Azure Portal. Abra el panel **Métricas** y seleccione la pestaña **Coherencia**. Examine el gráfico llamado **Probabilidad de lecturas con coherencia fuerte según la carga de trabajo (consultar PBS)** .

:::image type="content" source="./media/how-to-manage-consistency/pbs-metric.png" alt-text="Gráfico de PBS en Azure Portal":::

## <a name="next-steps"></a>Pasos siguientes

Aprenda cómo administrar los conflictos de datos o pase al siguiente concepto principal de Azure Cosmos DB. Vea los artículos siguientes:

* [Niveles de coherencia en Azure Cosmos DB](consistency-levels.md)
* [Administración de conflictos entre regiones](how-to-manage-conflicts.md)
* [Creación de particiones y distribución de datos](partition-data.md)
* [Consistency Tradeoffs in Modern Distributed Database Systems Design](https://www.computer.org/csdl/magazine/co/2012/02/mco2012020037/13rRUxjyX7k) (Compromisos de coherencia en el diseño de sistemas modernos de bases de datos distribuidas)
* [Alta disponibilidad](high-availability.md)
* [Acuerdo de Nivel de Servicio de Azure Cosmos DB](https://azure.microsoft.com/support/legal/sla/cosmos-db/v1_2/)
