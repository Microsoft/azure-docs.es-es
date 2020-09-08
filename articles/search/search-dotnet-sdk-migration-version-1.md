---
title: Actualización a la versión 1.1 del SDK de .NET para Azure Search
titleSuffix: Azure Cognitive Search
description: Migre el código a la versión 1.1 del SDK de Azure Search para .NET a partir de versiones de API anteriores. Obtenga información sobre las novedades y los cambios de código necesarios.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.devlang: dotnet
ms.topic: conceptual
ms.date: 11/04/2019
ms.custom: devx-track-csharp
ms.openlocfilehash: 4860918fc4f995ad267fc35b142d3fcb460ce177
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002834"
---
# <a name="upgrade-to-azure-search-net-sdk-version-11"></a>Actualización a la versión 1.1 del SDK de .NET para Azure Search

Si usa la versión preliminar 1.0.2 o anterior del [SDK de .NET para Azure Search](/dotnet/api/overview/azure/search), este artículo le ayudará a actualizar la aplicación para que use la versión 1.1.

Para obtener un tutorial más general del SDK que incluya ejemplos, vea [Cómo usar Azure Search desde una aplicación .NET](search-howto-dotnet-sdk.md).

> [!NOTE]
> Tras la actualización a la versión 1.1, o si ya está usando una versión entre la 1.1 y la versión preliminar 2.0 (ambas incluidas), debe actualizar a la versión 3. Consulte [Actualización a la versión 3 del SDK de .NET para Azure Search](search-dotnet-sdk-migration.md) para instrucciones.
>

En primer lugar, actualice la referencia de NuGet para `Microsoft.Azure.Search` mediante la Consola del Administrador de paquetes NuGet, o bien haga clic con el botón derecho en las referencias del proyecto y seleccione "Administrar paquetes NuGet" en Visual Studio.

Una vez que NuGet ha descargado los nuevos paquetes y sus dependencias, recompile el proyecto.

Si antes usaba las versiones preliminares 1.0.0, 1.0.1 o 1.0.2, la compilación debería ejecutarse correctamente y estará lista para comenzar a usarse.

De lo contrario, si usaba la versión preliminar 0.13.0 u otra anterior, verá errores de compilación como los siguientes:

```output
Program.cs(137,56,137,62): error CS0117: 'Microsoft.Azure.Search.Models.IndexBatch' does not contain a definition for 'Create'
Program.cs(137,99,137,105): error CS0117: 'Microsoft.Azure.Search.Models.IndexAction' does not contain a definition for 'Create'
Program.cs(146,41,146,54): error CS1061: 'Microsoft.Azure.Search.IndexBatchException' does not contain a definition for 'IndexResponse' and no extension method 'IndexResponse' accepting a first argument of type 'Microsoft.Azure.Search.IndexBatchException' could be found (are you missing a using directive or an assembly reference?)
Program.cs(163,13,163,42): error CS0246: The type or namespace name 'DocumentSearchResponse' could not be found (are you missing a using directive or an assembly reference?)
```

El paso siguiente consiste en corregir los errores de compilación uno a uno. La mayoría requiere cambiar algunos nombres de clase y método cuyo nombre ha cambiado en el SDK. [lista de cambios importantes de la versión 1.1](#ListOfChangesV1) contiene una lista de estos cambios de nombre.

Si usa clases personalizadas para modelar los documentos, y esas clases tienen propiedades de tipos primitivos que no aceptan valores nulos (por ejemplo, `int` o `bool` en C#), hay una corrección de errores en la versión 1.1 del SDK que debe tener en cuenta. Para obtener más información, consulte la sección [Correcciones de errores en la versión 1.1](#BugFixesV1) .

Por último, una vez que haya solucionado los errores de compilación, puede realizar cambios a la aplicación para aprovechar las ventajas de la nueva funcionalidad, si lo desea.

<a name="ListOfChangesV1"></a>

## <a name="list-of-breaking-changes-in-version-11"></a>lista de cambios importantes de la versión 1.1
La siguiente lista se ordena por la probabilidad de que el cambio afecte al código de aplicación.

### <a name="indexbatch-and-indexaction-changes"></a>Cambios de IndexBatch e IndexAction
Se ha cambiado el nombre de `IndexBatch.Create` por `IndexBatch.New` y ya no tiene un argumento `params`. Puede usar `IndexBatch.New` para los lotes que mezclan distintos tipos de acciones (combinaciones, eliminaciones, etc.). Además, existen nuevos métodos estáticos para crear lotes donde todas las acciones son las mismas: `Delete`, `Merge`, `MergeOrUpload` y `Upload`.

`IndexAction` ya no tiene constructores públicos y sus propiedades ahora son inmutables. Necesita usar los nuevos métodos estáticos para crear acciones para propósitos diferentes: `Delete`, `Merge`, `MergeOrUpload` y `Upload`. `IndexAction.Create` se ha quitado. Si utiliza la sobrecarga que solo usa un documento, asegúrese de usar `Upload` en su lugar.

#### <a name="example"></a>Ejemplo
Si el código tiene el siguiente aspecto:

```csharp
var batch = IndexBatch.Create(documents.Select(doc => IndexAction.Create(doc)));
indexClient.Documents.Index(batch);
```

Puede cambiarlo por este para corregir cualquier error de compilación:

```csharp
var batch = IndexBatch.New(documents.Select(doc => IndexAction.Upload(doc)));
indexClient.Documents.Index(batch);
```

Si lo desea, puede simplificarlo aún más y optar por este:

```csharp
var batch = IndexBatch.Upload(documents);
indexClient.Documents.Index(batch);
```

### <a name="indexbatchexception-changes"></a>Cambios de IndexBatchException
La propiedad `IndexBatchException.IndexResponse` ha cambiado a `IndexingResults`, y su tipo ahora es `IList<IndexingResult>`.

#### <a name="example"></a>Ejemplo
Si el código tiene el siguiente aspecto:

```csharp
catch (IndexBatchException e)
{
    Console.WriteLine(
        "Failed to index some of the documents: {0}",
        String.Join(", ", e.IndexResponse.Results.Where(r => !r.Succeeded).Select(r => r.Key)));
}
```

Puede cambiarlo por este para corregir cualquier error de compilación:

```csharp
catch (IndexBatchException e)
{
    Console.WriteLine(
        "Failed to index some of the documents: {0}",
        String.Join(", ", e.IndexingResults.Where(r => !r.Succeeded).Select(r => r.Key)));
}
```

<a name="OperationMethodChanges"></a>

### <a name="operation-method-changes"></a>Cambios del método de operación
Cada operación del SDK de .NET para Azure Search se expone como un conjunto de sobrecargas de método para los autores de llamadas sincrónicas y asincrónicas. Las firmas y la factorización de estas sobrecargas de método han cambiado en la versión 1.1.

Por ejemplo, la operación de "Obtener estadísticas de índice" en versiones anteriores del SDK expone estas firmas:

En `IIndexOperations`:

```csharp
// Asynchronous operation with all parameters
Task<IndexGetStatisticsResponse> GetStatisticsAsync(
    string indexName,
    CancellationToken cancellationToken);
```

En `IndexOperationsExtensions`:

```csharp
// Asynchronous operation with only required parameters
public static Task<IndexGetStatisticsResponse> GetStatisticsAsync(
    this IIndexOperations operations,
    string indexName);

// Synchronous operation with only required parameters
public static IndexGetStatisticsResponse GetStatistics(
    this IIndexOperations operations,
    string indexName);
```

Las firmas de método para la misma operación en la versión 1.1 tienen este aspecto:

En `IIndexesOperations`:

```csharp
// Asynchronous operation with lower-level HTTP features exposed
Task<AzureOperationResponse<IndexGetStatisticsResult>> GetStatisticsWithHttpMessagesAsync(
    string indexName,
    SearchRequestOptions searchRequestOptions = default(SearchRequestOptions),
    Dictionary<string, List<string>> customHeaders = null,
    CancellationToken cancellationToken = default(CancellationToken));
```

En `IndexesOperationsExtensions`:

```csharp
// Simplified asynchronous operation
public static Task<IndexGetStatisticsResult> GetStatisticsAsync(
    this IIndexesOperations operations,
    string indexName,
    SearchRequestOptions searchRequestOptions = default(SearchRequestOptions),
    CancellationToken cancellationToken = default(CancellationToken));

// Simplified synchronous operation
public static IndexGetStatisticsResult GetStatistics(
    this IIndexesOperations operations,
    string indexName,
    SearchRequestOptions searchRequestOptions = default(SearchRequestOptions));
```

A partir de la versión 1.1, el SDK de .NET para Azure Search organiza los métodos de operación de forma diferente:

* Los parámetros opcionales ahora se modelan como predeterminados en lugar de como sobrecargas de método adicionales. Esto reduce el número de sobrecargas de método, en ocasiones drásticamente.
* Los métodos de extensión ahora ocultan muchos de los detalles superfluos de HTTP del autor de llamada. Por ejemplo, las versiones anteriores del SDK devuelven un objeto de respuesta con un código de estado HTTP, que a menudo no necesitará comprobar, ya que los métodos de operación lanzan `CloudException` para cualquier código de estado que indica un error. Los nuevos métodos de extensión solo devuelven objetos de modelo, lo que le ahorra la molestia de tener que desencapsularlos en el código.
* Por el contrario, las interfaces principales ahora exponen métodos que permiten ejercer un mayor control a nivel de HTTP en caso de que sea necesario. Ahora puede pasar encabezados HTTP personalizados para incluirlos en solicitudes y el nuevo tipo de valor devuelto `AzureOperationResponse<T>` le ofrece acceso directo a `HttpRequestMessage` y `HttpResponseMessage` para la operación. `AzureOperationResponse` se define en el espacio de nombres `Microsoft.Rest.Azure` y reemplaza a `Hyak.Common.OperationResponse`.

### <a name="scoringparameters-changes"></a>Cambios de ScoringParameters
En el último SDK se agregó una nueva clase llamada `ScoringParameter` para que resulte más fácil proporcionar parámetros a los perfiles de puntuación en una consulta de búsqueda. Antes, la propiedad `ScoringProfiles` de la clase `SearchParameters` tenía el tipo `IList<string>`. Ahora, el tipo es `IList<ScoringParameter>`.

#### <a name="example"></a>Ejemplo
Si el código tiene el siguiente aspecto:

```csharp
var sp = new SearchParameters();
sp.ScoringProfile = "jobsScoringFeatured";      // Use a scoring profile
sp.ScoringParameters = new[] { "featuredParam-featured", "mapCenterParam-" + lon + "," + lat };
```

Puede cambiarlo por este para corregir cualquier error de compilación: 

```csharp
var sp = new SearchParameters();
sp.ScoringProfile = "jobsScoringFeatured";      // Use a scoring profile
sp.ScoringParameters =
    new[]
    {
        new ScoringParameter("featuredParam", new[] { "featured" }),
        new ScoringParameter("mapCenterParam", GeographyPoint.Create(lat, lon))
    };
```

### <a name="model-class-changes"></a>Cambios de la clase de modelo
Debido a los cambios de firma descritos en [Cambios del método de operación](#OperationMethodChanges), muchas clases del espacio de nombres `Microsoft.Azure.Search.Models` han cambiado de nombre o se han eliminado. Por ejemplo:

* `IndexDefinitionResponse` se ha reemplazado por `AzureOperationResponse<Index>`
* El nombre de `DocumentSearchResponse` ha cambiado a `DocumentSearchResult`
* El nombre de `IndexResult` ha cambiado a `IndexingResult`
* `Documents.Count()` ahora devuelve `long` con el número de documentos en lugar de `DocumentCountResponse`
* El nombre de `IndexGetStatisticsResponse` ha cambiado a `IndexGetStatisticsResult`
* El nombre de `IndexListResponse` ha cambiado a `IndexListResult`

Para resumir, las clases derivadas de `OperationResponse`que existían solo para encapsular un objeto de modelo se han quitado. El sufijo de las demás clases ha cambiado de `Response` a `Result`.

#### <a name="example"></a>Ejemplo
Si el código tiene el siguiente aspecto:

```csharp
IndexerGetStatusResponse statusResponse = null;

try
{
    statusResponse = _searchClient.Indexers.GetStatus(indexer.Name);
}
catch (Exception ex)
{
    Console.WriteLine("Error polling for indexer status: {0}", ex.Message);
    return;
}

IndexerExecutionResult lastResult = statusResponse.ExecutionInfo.LastResult;
```

Puede cambiarlo por este para corregir cualquier error de compilación:

```csharp
IndexerExecutionInfo status = null;

try
{
    status = _searchClient.Indexers.GetStatus(indexer.Name);
}
catch (Exception ex)
{
    Console.WriteLine("Error polling for indexer status: {0}", ex.Message);
    return;
}

IndexerExecutionResult lastResult = status.LastResult;
```

#### <a name="response-classes-and-ienumerable"></a>Clases de respuesta e IEnumerable
Un cambio adicional que puede afectar a su código es que las clases de respuesta que contienen colecciones ya no implementan `IEnumerable<T>`. En su lugar, puede acceder a la propiedad de colección directamente. Por ejemplo, si el código tiene el siguiente aspecto:

```csharp
DocumentSearchResponse<Hotel> response = indexClient.Documents.Search<Hotel>(searchText, sp);
foreach (SearchResult<Hotel> result in response)
{
    Console.WriteLine(result.Document);
}
```

Puede cambiarlo por este para corregir cualquier error de compilación:

```csharp
DocumentSearchResult<Hotel> response = indexClient.Documents.Search<Hotel>(searchText, sp);
foreach (SearchResult<Hotel> result in response.Results)
{
    Console.WriteLine(result.Document);
}
```

#### <a name="special-case-for-web-applications"></a>Caso especial para aplicaciones web
Si tiene una aplicación web que serializa `DocumentSearchResponse` directamente para enviar los resultados de la búsqueda al explorador, debe cambiar el código o los resultados no se serializarán correctamente. Por ejemplo, si el código tiene el siguiente aspecto:

```csharp
public ActionResult Search(string q = "")
{
    // If blank search, assume they want to search everything
    if (string.IsNullOrWhiteSpace(q))
        q = "*";

    return new JsonResult
    {
        JsonRequestBehavior = JsonRequestBehavior.AllowGet,
        Data = _featuresSearch.Search(q)
    };
}
```

Para cambiarlo, obtenga la propiedad `.Results` de la respuesta de búsqueda para corregir la representación de los resultados de la búsqueda:

```csharp
public ActionResult Search(string q = "")
{
    // If blank search, assume they want to search everything
    if (string.IsNullOrWhiteSpace(q))
        q = "*";

    return new JsonResult
    {
        JsonRequestBehavior = JsonRequestBehavior.AllowGet,
        Data = _featuresSearch.Search(q).Results
    };
}
```

Tendrá que buscar esos casos en el código por su cuenta. **El compilador no muestra advertencias** porque `JsonResult.Data` es del tipo `object`.

### <a name="cloudexception-changes"></a>Cambios de CloudException
La clase `CloudException` se ha desplazado desde el espacio de nombres `Hyak.Common` al espacio de nombres `Microsoft.Rest.Azure`. Además, el nombre de su propiedad `Error` ha cambiado a `Body`.

### <a name="searchserviceclient-and-searchindexclient-changes"></a>Cambios de SearchServiceClient y SearchIndexClient
El tipo de la propiedad `Credentials` ha cambiado de `SearchCredentials` a su clase base, `ServiceClientCredentials`. Si necesita acceder al valor de `SearchCredentials` de `SearchIndexClient` o `SearchServiceClient`, use la nueva propiedad `SearchCredentials`.

En versiones anteriores del SDK, `SearchServiceClient` y `SearchIndexClient` tenían constructores con un parámetro `HttpClient`. Estos se han reemplazado con constructores que obtienen `HttpClientHandler` y una matriz de objetos `DelegatingHandler`. Esto facilita instalar controladores personalizados para procesar previamente las solicitudes HTTP si es necesario.

Por último, los constructores que requerían `Uri` y `SearchCredentials` han cambiado. Por ejemplo, si tiene código con este aspecto:

```csharp
var client =
    new SearchServiceClient(
        new SearchCredentials("abc123"),
        new Uri("http://myservice.search.windows.net"));
```

Puede cambiarlo por este para corregir cualquier error de compilación:

```csharp
var client =
    new SearchServiceClient(
        new Uri("http://myservice.search.windows.net"),
        new SearchCredentials("abc123"));
```

Observe también que el tipo del parámetro de credenciales ha cambiado a `ServiceClientCredentials`. Es poco probable que esto afecte al código, ya que `SearchCredentials` deriva de `ServiceClientCredentials`.

### <a name="passing-a-request-id"></a>Transferir un identificador de solicitud
En versiones anteriores del SDK, se podía establecer un identificador de solicitud en `SearchServiceClient` o `SearchIndexClient`, que se incluiría en cada solicitud a la API de REST. Esto es útil para solucionar problemas con el servicio de búsqueda si necesita ponerse en contacto con soporte técnico. Sin embargo, es más útil establecer un identificador único de solicitud para cada operación, en lugar de utilizar el mismo identificador para todas las operaciones. Por este motivo, los métodos `SetClientRequestId` de `SearchServiceClient` y `SearchIndexClient` se han quitado. En su lugar, puede pasar un identificador de solicitud a cada método de operación mediante el parámetro opcional `SearchRequestOptions` .

> [!NOTE]
> En una futura versión del SDK, agregaremos un nuevo mecanismo para establecer un identificador de solicitud globalmente en los objetos cliente que sea coherente con el enfoque usado por otros SDK de Azure.
> 
> 

### <a name="example"></a>Ejemplo
Si tiene código con este aspecto:

```csharp
client.SetClientRequestId(Guid.NewGuid());
...
long count = client.Documents.Count();
```

Puede cambiarlo por este para corregir cualquier error de compilación:

```csharp
long count = client.Documents.Count(new SearchRequestOptions(requestId: Guid.NewGuid()));
```

### <a name="interface-name-changes"></a>Cambios de nombre de interfaz
Los nombres de interfaz del grupo de operaciones han cambiado por coherencia con sus nombres de propiedad correspondientes:

* El nombre del tipo de `ISearchServiceClient.Indexes` ha cambiado de `IIndexOperations` a `IIndexesOperations`.
* El nombre del tipo de `ISearchServiceClient.Indexers` ha cambiado de `IIndexerOperations` a `IIndexersOperations`.
* El nombre del tipo de `ISearchServiceClient.DataSources` ha cambiado de `IDataSourceOperations` a `IDataSourcesOperations`.
* El nombre del tipo de `ISearchIndexClient.Documents` ha cambiado de `IDocumentOperations` a `IDocumentsOperations`.

Es poco probable que este cambio afecte al código, a menos que cree simulacros de estas interfaces a efectos de pruebas.

<a name="BugFixesV1"></a>

## <a name="bug-fixes-in-version-11"></a>Correcciones de errores en la versión 1.1
Se produjo un error en las versiones anteriores del SDK de .NET para Azure Search en relación con la serialización de clases de modelos personalizados. El error puede producirse si ha creado una clase de modelo personalizado con una propiedad de un tipo de valor que no acepta valores NULL.

### <a name="steps-to-reproduce"></a>Pasos para reproducir
Cree una clase de modelo personalizado con una propiedad de tipo de valor que no acepta valores NULL. Por ejemplo, agregue una propiedad `UnitCount` pública de tipo `int` en lugar de `int?`.

Si indexa un documento con el valor predeterminado de ese tipo (por ejemplo, 0 para `int`), el campo será null en Azure Search. Si desea buscar posteriormente ese documento, la llamada `Search` producirá `JsonSerializationException` indicando que no se puede convertir `null` a `int`.

Además, los filtros pueden no funcionar según lo esperado, ya que se escribió null en el índice, en lugar del valor previsto.

### <a name="fix-details"></a>Detalles de corrección
Corregimos este problema en la versión 1.1 del SDK. Ahora, si tiene una clase de modelo similar a la siguiente:

```csharp
public class Model
{
    public string Key { get; set; }

    public int IntValue { get; set; }
}
```

y establece `IntValue` en 0, ese valor ahora se serializa correctamente como 0 en la conexión y se almacena como 0 en el índice. El recorrido de ida y vuelta también funciona según lo previsto.

Hay un posible problema que hay que tener en cuenta con este enfoque: si usa un tipo de modelo con una propiedad que no acepta valores NULL, tendrá que **garantizar** que ningún documento del índice contiene un valor NULL para el campo correspondiente. Ni el SDK ni la API de REST para Azure Search le permitirá aplicar esto.

Esto no es solo una inquietud hipotética: imagine un escenario donde se agrega un nuevo campo a un índice existente que es de tipo `Edm.Int32`. Después de actualizar la definición del índice, todos los documentos tendrán un valor null para ese campo nuevo (ya que todos los tipos aceptan valores NULL en Azure Search). Si después usa una clase de modelo con una propiedad `int` que no acepta valores NULL para ese campo, obtendrá `JsonSerializationException` así al intentar recuperar documentos:

```output
Error converting value {null} to type 'System.Int32'. Path 'IntValue'.
```

Por este motivo, recomendamos utilizar tipos que aceptan valores null en las clases de modelo como procedimiento recomendado.

Para más detalles sobre este error y la corrección, vea [este problema en GitHub](https://github.com/Azure/azure-sdk-for-net/issues/1063).