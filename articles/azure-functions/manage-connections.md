---
title: Administrar conexiones en Azure Functions
description: Aprenda a evitar problemas de rendimiento en Azure Functions mediante el uso de los clientes de conexión estáticos.
services: functions
author: ggailey777
manager: jeconnoc
ms.service: azure-functions
ms.topic: conceptual
ms.date: 02/25/2018
ms.author: glenga
ms.openlocfilehash: e15d6ad445c3fdde0632c3ad468eee7da836a394
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65785957"
---
# <a name="manage-connections-in-azure-functions"></a>Administrar conexiones en Azure Functions

Las funciones en una aplicación de función comparten recursos. Entre esos recursos compartidos se encuentran las conexiones: Las conexiones HTTP, las conexiones de base de datos y las conexiones a servicios como Azure Storage. Cuando se ejecutan simultáneamente muchas funciones es posible que se agoten las conexiones disponibles. En este artículo se explica cómo programar las funciones para evitar el uso más conexiones que necesitan.

## <a name="connection-limit"></a>Límite de conexiones

El número de conexiones disponibles está limitado en parte porque una aplicación de función se ejecuta en un [entorno de recinto](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox). Una de las restricciones que impone el recinto de seguridad en el código es un límite en el número de conexiones (actualmente en 600 conexiones activas y 1.200 total de conexiones) por instancia. Cuando se alcanza este límite, el runtime de las funciones crea un registro con el siguiente mensaje: `Host thresholds exceeded: Connections`.

Este límite es por instancia.  Cuando el [controlador de escala agrega instancias de function app](functions-scale.md#how-the-consumption-and-premium-plans-work) para controlar más solicitudes, cada instancia tiene un límite de conexiones independientes. Esto significa que no hay ningún límite de conexión global y puede tener mucho más de 600 conexiones activas a través de todas las instancias activas.

Para solucionar el problema, asegúrese de que ha habilitado Application Insights para su aplicación de función. Application Insights permite ver las métricas para las aplicaciones de función, como las ejecuciones. Para obtener más información, consulte [ver datos de telemetría en Application Insights](functions-monitoring.md#view-telemetry-in-application-insights).  

## <a name="static-clients"></a>Clientes estáticos

Para evitar mantener más conexiones de las necesarias, reutilice las instancias de cliente en lugar de crear nuevas con cada invocación de función. Se recomienda la reutilización de conexiones de cliente para cualquier lenguaje que se haya escrito su función en. Por ejemplo, los clientes de .NET como el [HttpClient](https://msdn.microsoft.com/library/system.net.http.httpclient(v=vs.110).aspx), [DocumentClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.documentclient
), y los clientes de almacenamiento de Azure pueden administrar las conexiones si usa un cliente único y estático.

Estas son algunas directrices que seguir cuando se usa un cliente específico de servicio en una aplicación de Azure Functions:

- *No* crear un nuevo cliente con cada invocación de función.
- *Hacer* crear un cliente único y estático que puede utilizar cada invocación de función.
- *Considere la posibilidad de* creación de un cliente único y estático en una clase auxiliar compartida si diferentes funciones usan el mismo servicio.

## <a name="client-code-examples"></a>Ejemplos de código de cliente

En esta sección se muestran los procedimientos recomendados para crear y utilizar clientes a partir del código de función.

### <a name="httpclient-example-c"></a>Ejemplo de HttpClient (C#)

Este es un ejemplo de C# código que crea un estático de función [HttpClient](https://msdn.microsoft.com/library/system.net.http.httpclient(v=vs.110).aspx) instancia:

```cs
// Create a single, static HttpClient
private static HttpClient httpClient = new HttpClient();

public static async Task Run(string input)
{
    var response = await httpClient.GetAsync("https://example.com");
    // Rest of function
}
```

Una pregunta común sobre [HttpClient](https://msdn.microsoft.com/library/system.net.http.httpclient(v=vs.110).aspx) en. NET, es "¿dispose de mi cliente?" En general, se eliminan los objetos que implementan `IDisposable` cuando haya terminado con ellos. Pero no dispone de un cliente estático porque ya no son usarlo cuando finaliza la función. Desea que el cliente estático viva en toda la duración de la aplicación.

### <a name="http-agent-examples-javascript"></a>Ejemplos de agente HTTP (JavaScript)

Debido a que proporciona mejores opciones de administración de conexiones, debería utilizar la clase nativa [`http.agent`](https://nodejs.org/dist/latest-v6.x/docs/api/http.html#http_class_http_agent) en lugar de los métodos no nativos, como el módulo `node-fetch`. Parámetros de conexión se configuran a través de las opciones en el `http.agent` clase. Para opciones detalladas disponibles con el agente HTTP, consulte [nuevo agente (\[opciones\])](https://nodejs.org/dist/latest-v6.x/docs/api/http.html#http_new_agent_options).

Global `http.globalAgent` clase usada por `http.request()` tiene todos estos valores con sus valores predeterminados correspondientes. La forma recomendada de configurar los límites de conexión en Functions es establecer un número máximo globalmente. El siguiente ejemplo establece el número máximo de sockets para la aplicación de función:

```js
http.globalAgent.maxSockets = 200;
```

 El ejemplo siguiente crea una nueva solicitud HTTP con un agente HTTP personalizado solo para esa solicitud:

```js
var http = require('http');
var httpAgent = new http.Agent();
httpAgent.maxSockets = 200;
options.agent = httpAgent;
http.request(options, onResponseCallback);
```

### <a name="documentclient-code-example-c"></a>Ejemplo de código de DocumentClient (C#)

[DocumentClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.documentclient
) se conecta a una instancia de Azure Cosmos DB. La documentación de Azure Cosmos DB recomienda el [uso de un cliente de Azure Cosmos DB singleton para el ciclo de vida de la aplicación](https://docs.microsoft.com/azure/cosmos-db/performance-tips#sdk-usage). En el ejemplo siguiente se muestra un patrón para realizarlo en una función:

```cs
#r "Microsoft.Azure.Documents.Client"
using Microsoft.Azure.Documents.Client;

private static Lazy<DocumentClient> lazyClient = new Lazy<DocumentClient>(InitializeDocumentClient);
private static DocumentClient documentClient => lazyClient.Value;

private static DocumentClient InitializeDocumentClient()
{
    // Perform any initialization here
    var uri = new Uri("example");
    var authKey = "authKey";
    
    return new DocumentClient(uri, authKey);
}

public static async Task Run(string input)
{
    Uri collectionUri = UriFactory.CreateDocumentCollectionUri("database", "collection");
    object document = new { Data = "example" };
    await documentClient.UpsertDocumentAsync(collectionUri, document);
    
    // Rest of function
}
```

### <a name="cosmosclient-code-example-javascript"></a>Ejemplo de código CosmosClient (JavaScript)
[CosmosClient](/javascript/api/@azure/cosmos/cosmosclient) se conecta a una instancia de Azure Cosmos DB. La documentación de Azure Cosmos DB recomienda el [uso de un cliente de Azure Cosmos DB singleton para el ciclo de vida de la aplicación](../cosmos-db/performance-tips.md#sdk-usage). En el ejemplo siguiente se muestra un patrón para realizarlo en una función:

```javascript
const cosmos = require('@azure/cosmos');
const endpoint = process.env.COSMOS_API_URL;
const masterKey = process.env.COSMOS_API_KEY;
const { CosmosClient } = cosmos;

const client = new CosmosClient({ endpoint, auth: { masterKey } });
// All function invocations also reference the same database and container.
const container = client.database("MyDatabaseName").container("MyContainerName");

module.exports = async function (context) {
    const { result: itemArray } = await container.items.readAll().toArray();
    context.log(itemArray);
}
```

## <a name="sqlclient-connections"></a>Conexiones de SqlClient

El código de función puede usar el proveedor de datos de .NET Framework para SQL Server ([SqlClient](https://msdn.microsoft.com/library/system.data.sqlclient(v=vs.110).aspx)) para realizar conexiones a una base de datos relacional de SQL. Esto también es el proveedor subyacente para entornos de datos que se basan en ADO.NET, como [Entity Framework](https://msdn.microsoft.com/library/aa937723(v=vs.113).aspx). A diferencia de las conexiones de [HttpClient](https://msdn.microsoft.com/library/system.net.http.httpclient(v=vs.110).aspx) y [DocumentClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.client.documentclient
), ADO.NET implementa la agrupación de conexiones de manera predeterminada. Pero, dado que todavía puede quedarse sin conexiones, debe optimizar las conexiones a la base de datos. Para más información, consulte el artículo sobre la [agrupación de conexiones de SQL Server (ADO.NET)](https://docs.microsoft.com/dotnet/framework/data/adonet/sql-server-connection-pooling).

> [!TIP]
> Algunos marcos de datos, como Entity Framework, suelen obtención las cadenas de conexión desde el **ConnectionStrings** sección de un archivo de configuración. En este caso, debe agregar explícitamente las cadenas de conexión de base de datos SQL a la colección **Cadenas de conexión** de la configuración de la aplicación de función y en el [archivo local.settings.json](functions-run-local.md#local-settings-file) del proyecto local. Si va a crear una instancia de [SqlConnection](https://msdn.microsoft.com/library/system.data.sqlclient.sqlconnection(v=vs.110).aspx) en el código de función, debe almacenar el valor de cadena de conexión **configuración de la aplicación** con las otras conexiones.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información acerca de por qué se recomiendan clientes estáticos, vea [antipatrón instanciación](https://docs.microsoft.com/azure/architecture/antipatterns/improper-instantiation/).

Para más sugerencias de rendimiento de Azure Functions, consulte [Optimización del rendimiento y la confiabilidad de Azure Functions](functions-best-practices.md).
