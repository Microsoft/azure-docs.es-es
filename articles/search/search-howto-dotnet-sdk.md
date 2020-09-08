---
title: Uso de Microsoft.Azure.Search (v10) en .NET
titleSuffix: Azure Cognitive Search
description: Aprenda a crear y administrar objetos de búsqueda de una aplicación .NET con C# y la versión 10 del SDK de .NET. Los fragmentos de código muestran la conexión al servicio, la creación de índices y las consultas.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.devlang: dotnet
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 08/05/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 394c87bcd3e4580289fbccc6a31b164f914dc8a3
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020803"
---
# <a name="how-to-use-microsoftazuresearch-v10-in-a-net-application"></a>Uso de Microsoft.Azure.Search (v10) en una aplicación .NET

En este artículo se explica cómo crear y administrar objetos de búsqueda con C# y el [SDK de .NET de Azure Cognitive Search (v10)](/dotnet/api/overview/azure/search). La versión 10 es la última versión del paquete Microsoft.Azure.Search. En el futuro, se implementarán características nuevas en [Azure.Search.Documents](/dotnet/api/overview/azure/search.documents-readme) desde el equipo de SDK de Azure.

Si tiene proyectos de desarrollo existentes o en proceso, siga usando la versión 10. En el caso de los proyectos nuevos, o para usar las características nuevas, debe realizar la transición de una solución de búsqueda existente a la biblioteca nueva.

## <a name="whats-in-version-10"></a>Novedades de la versión 10

El SDK se compone de unas cuantas bibliotecas de cliente que le permiten administrar los índices, orígenes de datos, indexadores y asignaciones de sinónimos, así como cargar y administrar documentos y ejecutar consultas; todo ello sin tener que lidiar con los detalles de HTTP y JSON. Esas bibliotecas de cliente se distribuyen como paquetes de NuGet.

El paquete de NuGet principal es `Microsoft.Azure.Search`, que es un metapaquete que incluye el resto de paquetes a modo de dependencias. Use este paquete si está comenzando a trabajar con él o si sabe que la aplicación necesitará todas las características de Búsqueda cognitiva de Azure.

Estos son otros paquetes de NuGet que tiene el SDK:
 
  - `Microsoft.Azure.Search.Data`: use este paquete si va a desarrollar una aplicación .NET mediante Búsqueda cognitiva de Azure y solo necesita consultar o actualizar los documentos en los índices. Si también tiene que crear o actualizar los índices, las asignaciones de sinónimos u otros recursos de nivel de servicio, use el paquete `Microsoft.Azure.Search`.
  - `Microsoft.Azure.Search.Service`: use este paquete si está desarrollando la automatización en .NET para administrar los índices, las asignaciones de sinónimos, los indexadores, los orígenes de datos u otros recursos de nivel de servicio de Búsqueda cognitiva de Azure. Si solo necesita consultar o actualizar los documentos de los índices, use el paquete `Microsoft.Azure.Search.Data` en su lugar. Si necesita toda la funcionalidad de Búsqueda cognitiva de Azure, use el paquete `Microsoft.Azure.Search`.
  - `Microsoft.Azure.Search.Common`: tipos comunes que necesitan las bibliotecas de .NET de Búsqueda cognitiva de Azure. No es necesario usar este paquete directamente en la aplicación, sino que está concebido únicamente para usarse como una dependencia.

Las diferentes bibliotecas de cliente definen clases como `Index`, `Field` y `Document`, además de operaciones como `Indexes.Create` y `Documents.Search` en las clases `SearchServiceClient` y `SearchIndexClient`. Estas clases están organizadas en los espacios de nombres siguientes:

* [Microsoft.Azure.Search](/dotnet/api/microsoft.azure.search)
* [Microsoft.Azure.Search.Models](/dotnet/api/microsoft.azure.search.models)

Si desea proporcionar comentarios para una actualización futura del SDK, vaya a nuestra [página de comentarios](https://feedback.azure.com/forums/263029-azure-search/) o cree un problema en [GitHub](https://github.com/azure/azure-sdk-for-net/issues) y mencione "Búsqueda cognitiva de Azure" en el título del problema.

El SDK de .NET tiene como destino la versión `2019-05-06` de la [API REST de Azure Cognitive Search](/rest/api/searchservice/). Esta versión incluye compatibilidad con [tipos complejos](search-howto-complex-data-types.md), con el [enriquecimiento con inteligencia artificial](cognitive-search-concept-intro.md), con la característica [Autocompletar](/rest/api/searchservice/autocomplete) y con el [modo de análisis JsonLines](search-howto-index-json-blobs.md) al indexar Azure Blobs. 

Este SDK no admite [operaciones de administración](/rest/api/searchmanagement/) como la creación y el escalado de servicios de Search y las claves de API de administración. Si necesita administrar los recursos de Search desde una aplicación .NET, puede usar el [SDK de administración para .NET de Búsqueda cognitiva de Azure](https://aka.ms/search-mgmt-sdk).

## <a name="upgrading-to-the-latest-version-of-the-sdk"></a>Actualización a la versión más reciente del SDK
Si ya utiliza una versión anterior del SDK para .NET de Búsqueda cognitiva de Azure y desea actualizar a la versión más reciente disponible con carácter general, en [este artículo](search-dotnet-sdk-migration-version-9.md) se explica el proceso.

## <a name="requirements-for-the-sdk"></a>Requisitos para el SDK
1. Visual Studio 2017 o cualquier versión posterior.
2. Su propio servicio Búsqueda cognitiva de Azure. Para usar el SDK, será necesario el nombre del servicio y una o varias claves de API. [Crear un servicio en el portal](search-create-service-portal.md) le ayudará con estos pasos.
3. Descargue el [paquete NuGet](https://www.nuget.org/packages/Microsoft.Azure.Search) del SDK de Búsqueda cognitiva de Azure para .NET mediante "Administrar paquetes NuGet" en Visual Studio. Simplemente busque el nombre del paquete `Microsoft.Azure.Search` en NuGet.org (o uno de los otros nombres de paquetes mencionados anteriormente si solo necesita un subconjunto de la funcionalidad).

El SDK para .NET de Búsqueda cognitiva de Azure es compatible con las aplicaciones destinadas a .NET Framework 4.5.2 y otras versiones superiores, así como a .NET Core 2.0 y otras versiones superiores.

## <a name="core-scenarios"></a>Escenarios principales
Hay varias tareas que debe realizar en su aplicación de búsqueda. En este tutorial, hablaremos sobre estos escenarios básicos:

* Creación de un índice
* Llenado del índice con documentos
* Búsqueda de documentos mediante filtros y búsqueda de texto completo

El código de ejemplo siguiente ilustra cada uno de estos escenarios. No dude en usar los fragmentos de código en su propia aplicación.

### <a name="overview"></a>Información general
La aplicación de ejemplo que vamos a explorar crea un nuevo índice denominado "hotels", lo rellena con varios documentos y, a continuación, ejecuta varias consultas de búsqueda. Este es el programa principal, que muestra el flujo general:

```csharp
// This sample shows how to delete, create, upload documents and query an index
static void Main(string[] args)
{
    IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
    IConfigurationRoot configuration = builder.Build();

    SearchServiceClient serviceClient = CreateSearchServiceClient(configuration);

    string indexName = configuration["SearchIndexName"];

    Console.WriteLine("{0}", "Deleting index...\n");
    DeleteIndexIfExists(indexName, serviceClient);

    Console.WriteLine("{0}", "Creating index...\n");
    CreateIndex(indexName, serviceClient);

    ISearchIndexClient indexClient = serviceClient.Indexes.GetClient(indexName);

    Console.WriteLine("{0}", "Uploading documents...\n");
    UploadDocuments(indexClient);

    ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(configuration);

    RunQueries(indexClientForQueries);

    Console.WriteLine("{0}", "Complete.  Press any key to end application...\n");
    Console.ReadKey();
}
```

> [!NOTE]
> Puede encontrar el código fuente completo de la aplicación de ejemplo usada en este tutorial en [GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo).
> 
>

Lo recorreremos paso a paso. Primero, debemos crear un nuevo `SearchServiceClient`. Este objeto le permite administrar índices. Para crear uno, deberá proporcionar el nombre de su servicio Búsqueda cognitiva de Azure y una clave de API de administración. Puede escribir esta información en el archivo `appsettings.json` de la [aplicación de ejemplo](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo).

```csharp
private static SearchServiceClient CreateSearchServiceClient(IConfigurationRoot configuration)
{
    string searchServiceName = configuration["SearchServiceName"];
    string adminApiKey = configuration["SearchServiceAdminApiKey"];

    SearchServiceClient serviceClient = new SearchServiceClient(searchServiceName, new SearchCredentials(adminApiKey));
    return serviceClient;
}
```

> [!NOTE]
> Si proporciona una clave incorrecta (por ejemplo, una clave de consulta cuando se requiere una clave de administración), el `SearchServiceClient` producirá un `CloudException` con el mensaje de error "Forbidden" (Prohibido) cuando llame a un método de operación con él, como `Indexes.Create`. Si esto sucede, compruebe la clave de API.
> 
> 

Las siguientes líneas llaman a métodos para crear un índice llamado "hotels", eliminándolo antes si ya existe. Describiremos estos métodos más adelante.

```csharp
Console.WriteLine("{0}", "Deleting index...\n");
DeleteIndexIfExists(indexName, serviceClient);

Console.WriteLine("{0}", "Creating index...\n");
CreateIndex(indexName, serviceClient);
```

A continuación, el índice debe rellenarse. Para rellenar el índice, necesitamos un objeto `SearchIndexClient`. Hay dos maneras de obtener uno: se puede crear o se puede llamar a `Indexes.GetClient` en el `SearchServiceClient`. Usamos esta última por motivos de comodidad.

```csharp
ISearchIndexClient indexClient = serviceClient.Indexes.GetClient(indexName);
```

> [!NOTE]
> En una aplicación de búsqueda típica, el llenado y la administración de índices se pueden controlar mediante un componente independiente de las consultas de búsqueda. `Indexes.GetClient` resulta cómodo para rellenar un índice, porque evita la molestia de dar otro `SearchCredentials` más. Con este fin, pasa la clave de administrador que se usó para crear el `SearchServiceClient` al nuevo `SearchIndexClient`. Sin embargo, en la parte de la aplicación que ejecuta consultas, es mejor crear directamente el objeto `SearchIndexClient` para poder pasar una clave de consulta (que solo permite leer datos) en lugar de una clave de administración. Esto está en consonancia con el principio de privilegios mínimos y le ayudará a proteger su aplicación. Puede encontrar más información acerca de las claves de administración y consulta [aquí](/rest/api/searchservice/#authentication-and-authorization).
> 
> 

Ahora que tenemos un `SearchIndexClient`, podemos rellenar el índice. Para ello, utilizaremos otro método que trataremos más adelante.

```csharp
Console.WriteLine("{0}", "Uploading documents...\n");
UploadDocuments(indexClient);
```

Finalmente, ejecutamos algunas consultas de búsqueda y mostramos los resultados. Esta vez usamos otro `SearchIndexClient`:

```csharp
ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(indexName, configuration);

RunQueries(indexClientForQueries);
```

Echaremos un vistazo al método `RunQueries` más adelante. Este es el código para crear el nuevo elemento `SearchIndexClient`:

```csharp
private static SearchIndexClient CreateSearchIndexClient(string indexName, IConfigurationRoot configuration)
{
    string searchServiceName = configuration["SearchServiceName"];
    string queryApiKey = configuration["SearchServiceQueryApiKey"];

    SearchIndexClient indexClient = new SearchIndexClient(searchServiceName, indexName, new SearchCredentials(queryApiKey));
    return indexClient;
}
```

Esta vez usamos una clave de consulta dado que no necesitamos acceso de escritura al índice. Puede escribir esta información en el archivo `appsettings.json` de la [aplicación de ejemplo](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo).

Si ejecuta esta aplicación con un nombre de servicio válido y claves de API, la salida será parecida a la de este ejemplo (parte de la salida de la consola se ha reemplazado por "..." con fines meramente ilustrativos):

```output

Deleting index...

Creating index...

Uploading documents...

Waiting for documents to be indexed...

Search the entire index for the term 'motel' and return only the HotelName field:

Name: Secret Point Motel

Name: Twin Dome Motel


Apply a filter to the index to find hotels with a room cheaper than $100 per night, and return the hotelId and description:

HotelId: 1
Description: The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Times Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.

HotelId: 2
Description: The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.


Search the entire index, order by a specific field (lastRenovationDate) in descending order, take the top two results, and show only hotelName and lastRenovationDate:

Name: Triple Landscape Hotel
Last renovated on: 9/20/2015 12:00:00 AM +00:00

Name: Twin Dome Motel
Last renovated on: 2/18/1979 12:00:00 AM +00:00


Search the hotel names for the term 'hotel':

HotelId: 3
Name: Triple Landscape Hotel
...

Complete.  Press any key to end application... 
```

El código fuente completo de la aplicación se proporciona al final de este artículo.

A continuación, veremos más de cerca cada uno de los métodos llamados por `Main`.

### <a name="creating-an-index"></a>Creación de un índice
Después de crear un objeto `SearchServiceClient`, `Main` es eliminar el índice "hotels", si ya existe. Esta eliminación se lleva a cabo mediante el método siguiente:

```csharp
private static void DeleteIndexIfExists(string indexName, SearchServiceClient serviceClient)
{
    if (serviceClient.Indexes.Exists(indexName))
    {
        serviceClient.Indexes.Delete(indexName);
    }
}
```

Este método utiliza el `SearchServiceClient` proporcionado para comprobar si existe el índice y, en ese caso, eliminarlo.

> [!NOTE]
> El código de ejemplo de este artículo usa los métodos sincrónicos del SDK de Búsqueda cognitiva de Azure para .NET por motivos de simplicidad. En sus propias aplicaciones, recomendamos que use métodos asincrónicos para mantener su escalabilidad y capacidad de respuesta. Por ejemplo, en el método anterior podría utilizar `ExistsAsync` y `DeleteAsync` en lugar de `Exists` y `Delete`.
> 
> 

A continuación, `Main` crea un nuevo índice "hotels" mediante una llamada a este método:

```csharp
private static void CreateIndex(string indexName, SearchServiceClient serviceClient)
{
    var definition = new Index()
    {
        Name = indexName,
        Fields = FieldBuilder.BuildForType<Hotel>()
    };
    
    serviceClient.Indexes.Create(definition);
}
```

Este método crea un nuevo objeto `Index` con una lista de objetos `Field` que define el esquema del nuevo índice. Cada campo tiene un nombre, un tipo de datos y varios atributos que definen su comportamiento de búsqueda. La clase `FieldBuilder` usa reflexión para crear una lista de objetos `Field` para el índice examinando las propiedades públicas y los atributos de la clase de modelo `Hotel` dada. Examinaremos más de cerca la clase `Hotel` más adelante.

> [!NOTE]
> En caso necesario, siempre puede crear la lista de objetos `Field` directamente en lugar de usar `FieldBuilder`. Por ejemplo, puede que no quiera usar una clase model o que necesite usar una clase model existente que no quiere modificar agregando atributos.
>
> 

Además de campos, puede agregar al índice perfiles de puntuación, proveedores de sugerencias u opciones de CORS (estos parámetros se omiten en el ejemplo para mayor brevedad). Puede encontrar más información sobre el objeto Index y sus partes constituyentes en la [referencia del SDK](/dotnet/api/microsoft.azure.search.models.index), así como en la [referencia de la API de REST de Búsqueda cognitiva de Azure](/rest/api/searchservice/).

### <a name="populating-the-index"></a>Llenado del índice
El siguiente paso en `Main` rellena el índice recién creado. Esto se lleva a cabo mediante el método siguiente (parte del código se ha reemplazado por "..." con fines meramente ilustrativos,  pero puede consultar la solución de ejemplo completa para ver el código de llenado de datos completo):

```csharp
private static void UploadDocuments(ISearchIndexClient indexClient)
{
    var hotels = new Hotel[]
    {
        new Hotel()
        {
            HotelId = "1",
            HotelName = "Secret Point Motel",
            ...
            Address = new Address()
            {
                StreetAddress = "677 5th Ave",
                ...
            },
            Rooms = new Room[]
            {
                new Room()
                {
                    Description = "Budget Room, 1 Queen Bed (Cityside)",
                    ...
                },
                new Room()
                {
                    Description = "Budget Room, 1 King Bed (Mountain View)",
                    ...
                },
                new Room()
                {
                    Description = "Deluxe Room, 2 Double Beds (City View)",
                    ...
                }
            }
        },
        new Hotel()
        {
            HotelId = "2",
            HotelName = "Twin Dome Motel",
            ...
            {
                StreetAddress = "140 University Town Center Dr",
                ...
            },
            Rooms = new Room[]
            {
                new Room()
                {
                    Description = "Suite, 2 Double Beds (Mountain View)",
                    ...
                },
                new Room()
                {
                    Description = "Standard Room, 1 Queen Bed (City View)",
                    ...
                },
                new Room()
                {
                    Description = "Budget Room, 1 King Bed (Waterfront View)",
                    ...
                }
            }
        },
        new Hotel()
        {
            HotelId = "3",
            HotelName = "Triple Landscape Hotel",
            ...
            Address = new Address()
            {
                StreetAddress = "3393 Peachtree Rd",
                ...
            },
            Rooms = new Room[]
            {
                new Room()
                {
                    Description = "Standard Room, 2 Queen Beds (Amenities)",
                    ...
                },
                new Room ()
                {
                    Description = "Standard Room, 2 Double Beds (Waterfront View)",
                    ...
                },
                new Room()
                {
                    Description = "Deluxe Room, 2 Double Beds (Cityside)",
                    ...
                }
            }
        }
    };

    var batch = IndexBatch.Upload(hotels);

    try
    {
        indexClient.Documents.Index(batch);
    }
    catch (IndexBatchException e)
    {
        // Sometimes when your Search service is under load, indexing will fail for some of the documents in
        // the batch. Depending on your application, you can take compensating actions like delaying and
        // retrying. For this simple demo, we just log the failed document keys and continue.
        Console.WriteLine(
            "Failed to index some of the documents: {0}",
            String.Join(", ", e.IndexingResults.Where(r => !r.Succeeded).Select(r => r.Key)));
    }

    Console.WriteLine("Waiting for documents to be indexed...\n");
    Thread.Sleep(2000);
}
```

Este método tiene cuatro partes. La primera crea una matriz de tres objetos `Hotel` (cada uno con tres objetos `Room`), que serán los datos de entrada que cargaremos en el índice. Estos datos están incluidos en el código por motivos de simplicidad. En su propia aplicación, los datos provendrán normalmente de un origen de datos externo, como una base de datos SQL.

En la segunda parte se crea un `IndexBatch` que contiene los documentos. Especifique la operación que desee aplicar al lote en el momento de su creación, en este caso llamando a `IndexBatch.Upload`. A continuación, el lote se carga en el índice de Búsqueda cognitiva de Azure mediante el método `Documents.Index` .

> [!NOTE]
> En este ejemplo nos limitamos a cargar documentos. Si desea combinar los cambios en los documentos existentes o eliminar documentos, puede crear lotes llamando a `IndexBatch.Merge`, `IndexBatch.MergeOrUpload` o `IndexBatch.Delete`. También puede combinar diferentes operaciones en un único lote llamando a `IndexBatch.New`, que selecciona una colección de objetos de `IndexAction`, que indican a Búsqueda cognitiva de Azure que realice una operación determinada en un documento. Puede crear cada `IndexAction` con su propia operación llamando al método correspondiente, como `IndexAction.Merge`, `IndexAction.Upload`, etc.
> 
> 

La tercera parte de este método es un bloque catch que controla un caso de error importante para la indización. Si su servicio Búsqueda cognitiva de Azure no logra indexar algunos de los documentos del lote, aparece una `IndexBatchException` producida por `Documents.Index`. Esta excepción puede suceder si se indexan documentos mientras el servicio está sobrecargado. **Recomendamos encarecidamente controlar este caso de forma explícita en el código.** Puede retrasar la indización de los documentos que dieron error y volver a intentarlo; puede crear un registro y continuar, como hace el ejemplo, o puede adoptar otro enfoque según los requisitos de coherencia de datos de la aplicación.

> [!NOTE]
> Puede usar el método [`FindFailedActionsToRetry`](/dotnet/api/microsoft.azure.search.indexbatchexception.findfailedactionstoretry) para construir un nuevo lote que contenga solo las acciones que dieron error en una llamada anterior a `Index`. Hay una explicación sobre cómo se usa correctamente [en StackOverflow](https://stackoverflow.com/questions/40012885/azure-search-net-sdk-how-to-use-findfailedactionstoretry).
>
>

Por último, el método `UploadDocuments` se retrasa durante dos segundos. La indexación ocurre de manera asincrónica en el servicio Búsqueda cognitiva de Azure, por lo que la aplicación de ejemplo debe esperar unos momentos para asegurarse de que los documentos estén disponibles para la búsqueda. Retrasos así solo suelen ser necesarios en las pruebas, demostraciones y aplicaciones de ejemplo.

<a name="how-dotnet-handles-documents"></a>

#### <a name="how-the-net-sdk-handles-documents"></a>Gestión de documentos del SDK de .NET
Quizás se pregunte cómo consigue el SDK de Búsqueda cognitiva de Azure para .NET cargar en el índice las instancias de una clase definida por el usuario como `Hotel` . Para responder mejor a esa pregunta, echemos un vistazo a la clase `Hotel` :

```csharp
using System;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Microsoft.Spatial;
using Newtonsoft.Json;

public partial class Hotel
{
    [System.ComponentModel.DataAnnotations.Key]
    [IsFilterable]
    public string HotelId { get; set; }

    [IsSearchable, IsSortable]
    public string HotelName { get; set; }

    [IsSearchable]
    [Analyzer(AnalyzerName.AsString.EnLucene)]
    public string Description { get; set; }

    [IsSearchable]
    [Analyzer(AnalyzerName.AsString.FrLucene)]
    [JsonProperty("Description_fr")]
    public string DescriptionFr { get; set; }

    [IsSearchable, IsFilterable, IsSortable, IsFacetable]
    public string Category { get; set; }

    [IsSearchable, IsFilterable, IsFacetable]
    public string[] Tags { get; set; }

    [IsFilterable, IsSortable, IsFacetable]
    public bool? ParkingIncluded { get; set; }

    // SmokingAllowed reflects whether any room in the hotel allows smoking.
    // The JsonIgnore attribute indicates that a field should not be created 
    // in the index for this property and it will only be used by code in the client.
    [JsonIgnore]
    public bool? SmokingAllowed => (Rooms != null) ? Array.Exists(Rooms, element => element.SmokingAllowed == true) : (bool?)null;

    [IsFilterable, IsSortable, IsFacetable]
    public DateTimeOffset? LastRenovationDate { get; set; }

    [IsFilterable, IsSortable, IsFacetable]
    public double? Rating { get; set; }

    public Address Address { get; set; }

    [IsFilterable, IsSortable]
    public GeographyPoint Location { get; set; }

    public Room[] Rooms { get; set; }
}
```

Lo primero que se debe tener en cuenta es que el nombre de cada propiedad pública de la clase `Hotel` se asignará a un campo con el mismo nombre en la definición del índice. Si quiere que cada campo comience por una letra minúscula, puede indicar al SDK que asigne los nombres de propiedad en mayúscula y minúscula automáticamente con el atributo `[SerializePropertyNamesAsCamelCase]` de la clase. Este escenario es habitual en aplicaciones .NET que realizan enlaces de datos cuando el esquema de destino está fuera del control del desarrollador de la aplicación, sin infringir ninguna directriz de nomenclatura de "Pascal Case" en .NET.

> [!NOTE]
> El SDK para .NET de Búsqueda cognitiva de Azure usa la biblioteca [NewtonSoft JSON.NET](https://www.newtonsoft.com/json/help/html/Introduction.htm) para serializar y deserializar los objetos de modelo personalizados en JSON y de este. Puede personalizar esta serialización si es necesario. Para más información, vea [Serialización personalizada con JSON.NET](#JsonDotNet).
> 
> 

La segunda cuestión que debe considerarse es que cada propiedad se representa con atributos como `IsFilterable`, `IsSearchable`, `Key` y `Analyzer`. Estos atributos se asignan directamente a los [atributos de campo correspondientes de un índice de Búsqueda cognitiva de Azure](/rest/api/searchservice/create-index). La clase `FieldBuilder` usa estas propiedades para construir definiciones de campo para el índice.

La tercera cuestión importante sobre la clase `Hotel` son los tipos de datos de las propiedades públicas. Los tipos .NET de esas propiedades se asignan a los tipos de campo equivalentes de la definición del índice. Por ejemplo, la propiedad de cadena `Category` se asigna al campo `category`, que es de tipo `Edm.String`. Se dan asignaciones de tipos semejantes entre `bool?`, `Edm.Boolean`, `DateTimeOffset?` y `Edm.DateTimeOffset`, etc. Las reglas específicas para la asignación de tipos se documentan con el método `Documents.Get` en la [referencia del SDK de Búsqueda cognitiva de Azure para .NET](/dotnet/api/microsoft.azure.search.documentsoperationsextensions.get). Aunque la clase `FieldBuilder` se ocupa de esta asignación, todavía puede ser útil comprenderlo por si necesitara solucionar los problemas de serialización.

¿Se ha dado cuenta de la propiedad `SmokingAllowed`?

```csharp
[JsonIgnore]
public bool? SmokingAllowed => (Rooms != null) ? Array.Exists(Rooms, element => element.SmokingAllowed == true) : (bool?)null;
```

El atributo `JsonIgnore` de esta propiedad indica a `FieldBuilder` que no la serialice en el índice como un campo.  Se trata de una excelente manera de crear propiedades calculadas del lado cliente que puede usar como aplicaciones auxiliares en su aplicación.  En este caso, la propiedad `SmokingAllowed` refleja si está permitido fumar en algún objeto `Room` de la colección de objetos `Rooms`.  Si todos están establecidos en false, quiere decir que no se puede fumar en todo el hotel.

Algunas propiedades, como `Address` y `Rooms`, son instancias de clases. NET.  Estas propiedades representan estructuras de datos más complejas y, como resultado, se requieren campos con un [tipo de datos complejo](./search-howto-complex-data-types.md) en el índice.

La propiedad `Address` representa un conjunto de varios valores de la clase `Address`, definida aquí:

```csharp
using System;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Newtonsoft.Json;

namespace AzureSearch.SDKHowTo
{
    public partial class Address
    {
        [IsSearchable]
        public string StreetAddress { get; set; }

        [IsSearchable, IsFilterable, IsSortable, IsFacetable]
        public string City { get; set; }

        [IsSearchable, IsFilterable, IsSortable, IsFacetable]
        public string StateProvince { get; set; }

        [IsSearchable, IsFilterable, IsSortable, IsFacetable]
        public string PostalCode { get; set; }

        [IsSearchable, IsFilterable, IsSortable, IsFacetable]
        public string Country { get; set; }
    }
}
```

Esta clase contiene los valores estándar que se usan para describir direcciones de Estados Unidos o Canadá. Puede usar tipos como este para agrupar campos lógicos en el índice.

La propiedad `Rooms` representa una matriz de objetos `Room`:

```csharp
using System;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Newtonsoft.Json;

namespace AzureSearch.SDKHowTo
{
    public partial class Room
    {
        [IsSearchable]
        [Analyzer(AnalyzerName.AsString.EnMicrosoft)]
        public string Description { get; set; }

        [IsSearchable]
        [Analyzer(AnalyzerName.AsString.FrMicrosoft)]
        [JsonProperty("Description_fr")]
        public string DescriptionFr { get; set; }

        [IsSearchable, IsFilterable, IsFacetable]
        public string Type { get; set; }

        [IsFilterable, IsFacetable]
        public double? BaseRate { get; set; }

        [IsSearchable, IsFilterable, IsFacetable]
        public string BedOptions { get; set; }

        [IsFilterable, IsFacetable]
        public int SleepsCount { get; set; }

        [IsFilterable, IsFacetable]
        public bool? SmokingAllowed { get; set; }

        [IsSearchable, IsFilterable, IsFacetable]
        public string[] Tags { get; set; }
    }
}
```

El modelo de datos de .NET y su esquema de índice correspondiente se deben diseñar para dar cabida a la experiencia de búsqueda que quiera proporcionar al usuario final. Cada objeto de nivel superior en .NET (es decir, cada documento en el índice) corresponde a un resultado de búsqueda que aparecería en la interfaz de usuario. Por ejemplo, en una aplicación de búsqueda de alojamiento, puede que los usuarios finales quieran realizar búsquedas por nombre de hotel, por las características del hotel o por las características de una habitación determinada. Analizaremos algunos ejemplos de consultas un poco más adelante.

Esta posibilidad de usar sus propias clases para interactuar con los documentos del índice funciona en ambas direcciones: también puede recuperar los resultados de la búsqueda y hacer que el SDK los deserialice automáticamente a un tipo de su elección, como veremos en la siguiente sección.

> [!NOTE]
> El SDK de Búsqueda cognitiva de Azure para .NET también admite documentos de tipo dinámico mediante la clase `Document`, que es una asignación clave/valor de nombres de campo a valores de campo. Esto es útil en escenarios en los que no se conoce el esquema del índice en el momento del diseño o en los que resulte inconveniente enlazar a clases de modelo específicas. Todos los métodos del SDK que se ocupan de los documentos tienen sobrecargas que funcionan con la clase `Document` , así como sobrecargas de asignación rigurosa que aceptan un parámetro de tipo genérico. En el código de ejemplo de este tutorial solo se utilizan las últimas. La [clase `Document`](/dotnet/api/microsoft.azure.search.models.document) se hereda de `Dictionary<string, object>`.
> 
>

**¿Por qué debería usar tipos de datos que aceptan valores null?**

Al diseñar sus propias clases de modelo para asignar a un índice de Búsqueda cognitiva de Azure, es recomendable declarar las propiedades de tipos de valor como `bool` y `int` que aceptan valores NULL (por ejemplo: `bool?` en lugar de `bool`). Si usa un tipo de modelo con una propiedad que no acepta valores NULL, tendrá que **garantizar** que ningún documento del índice contiene un valor NULL para el campo correspondiente. Ni el SDK ni el servicio Búsqueda cognitiva de Azure le permitirá aplicar esto.

No solo se trata de una cuestión hipotética: imagine un escenario donde se agrega un nuevo campo a un índice existente que es de tipo `Edm.Int32`. Después de actualizar la definición del índice, todos los documentos tendrán un valor null para ese campo nuevo (ya que todos los tipos aceptan valores NULL en Búsqueda cognitiva de Azure). Si después usa una clase de modelo con una propiedad `int` que no acepta valores NULL para ese campo, obtendrá `JsonSerializationException` así al intentar recuperar documentos:

```output
Error converting value {null} to type 'System.Int32'. Path 'IntValue'.
```

Por este motivo, recomendamos utilizar tipos que aceptan valores NULL en las clases de modelo como procedimiento recomendado.

<a name="JsonDotNet"></a>

#### <a name="custom-serialization-with-jsonnet"></a>Serialización personalizada con JSON.NET
El SDK usa JSON.NET para serializar y deserializar documentos. Puede personalizar la serialización y deserialización si lo necesita definiendo su propio `JsonConverter` o `IContractResolver`. Para más información, vea la [documentación de JSON.NET](https://www.newtonsoft.com/json/help/html/Introduction.htm). Esto puede ser útil cuando desea adaptar una clase de modelo existente de la aplicación para usarla con Búsqueda cognitiva de Azure y otros escenarios más avanzados. Por ejemplo, con la serialización personalizada, puede:

* Incluir o excluir determinadas propiedades de la clase de modelo para que se almacenen como campos del documento.
* Asignar entre los nombres de propiedad del código y los nombres de campo del índice.
* Cree atributos personalizados que se puedan usar para asignar propiedades a campos de documento.

Puede encontrar ejemplos de implementación de serialización personalizada en las pruebas unitarias del SDK para .NET de Búsqueda cognitiva de Azure en GitHub. [Esta carpeta](https://github.com/Azure/azure-sdk-for-net/tree/4f6f4e4c90200c1b0621c4cead302a91e89f2aba/sdk/search/Microsoft.Azure.Search/tests/Tests/Models) es un buen punto de partida. Contiene clases que las pruebas de serialización personalizada utilizan.

### <a name="searching-for-documents-in-the-index"></a>Búsqueda de documentos en el índice
El último paso de la aplicación de ejemplo es buscar algunos documentos en el índice:

```csharp
private static void RunQueries(ISearchIndexClient indexClient)
{
    SearchParameters parameters;
    DocumentSearchResult<Hotel> results;

    Console.WriteLine("Search the entire index for the term 'motel' and return only the HotelName field:\n");

    parameters =
        new SearchParameters()
        {
            Select = new[] { "HotelName" }
        };

    results = indexClient.Documents.Search<Hotel>("motel", parameters);

    WriteDocuments(results);

    Console.Write("Apply a filter to the index to find hotels with a room cheaper than $100 per night, ");
    Console.WriteLine("and return the hotelId and description:\n");

    parameters =
        new SearchParameters()
        {
            Filter = "Rooms/any(r: r/BaseRate lt 100)",
            Select = new[] { "HotelId", "Description" }
        };

    results = indexClient.Documents.Search<Hotel>("*", parameters);

    WriteDocuments(results);

    Console.Write("Search the entire index, order by a specific field (lastRenovationDate) ");
    Console.Write("in descending order, take the top two results, and show only hotelName and ");
    Console.WriteLine("lastRenovationDate:\n");

    parameters =
        new SearchParameters()
        {
            OrderBy = new[] { "LastRenovationDate desc" },
            Select = new[] { "HotelName", "LastRenovationDate" },
            Top = 2
        };

    results = indexClient.Documents.Search<Hotel>("*", parameters);

    WriteDocuments(results);

    Console.WriteLine("Search the entire index for the term 'hotel':\n");

    parameters = new SearchParameters();
    results = indexClient.Documents.Search<Hotel>("hotel", parameters);

    WriteDocuments(results);
}
```

Cada vez que ejecuta una consulta, este método crea primero un nuevo objeto `SearchParameters`. Este objeto se usa para especificar más opciones en la consulta, como el orden, los filtros, la paginación y las facetas. En este método, vamos a establecer la propiedad `Filter`, `Select`, `OrderBy` y `Top` para diferentes consultas. Todas las `SearchParameters`propiedades se documentan [aquí](/dotnet/api/microsoft.azure.search.models.searchparameters).

El siguiente paso consiste en ejecutar la consulta de búsqueda. La búsqueda se ejecuta mediante el método `Documents.Search`. Para cada consulta, pasamos el texto de búsqueda para usarlo como cadena (o `"*"` si no hay ningún texto de búsqueda), además de los parámetros de búsqueda creados anteriormente. También especificamos `Hotel` como el parámetro de tipo para `Documents.Search`, lo que indica al SDK que deserialice los documentos de los resultados de búsqueda en objetos de tipo `Hotel`.

> [!NOTE]
> Puede encontrar más información acerca de la sintaxis de expresiones de consulta de búsqueda [aquí](/rest/api/searchservice/Simple-query-syntax-in-Azure-Search).
> 
> 

Por último, después de cada consulta este método recorre en iteración todas las coincidencias de los resultados de búsqueda e imprime cada documento en la consola:

```csharp
private static void WriteDocuments(DocumentSearchResult<Hotel> searchResults)
{
    foreach (SearchResult<Hotel> result in searchResults.Results)
    {
        Console.WriteLine(result.Document);
    }

    Console.WriteLine();
}
```

Examinemos más de cerca cada una de las consultas de una en una. Este es el código para ejecutar la primera consulta:

```csharp
parameters =
    new SearchParameters()
    {
        Select = new[] { "HotelName" }
    };

results = indexClient.Documents.Search<Hotel>("motel", parameters);

WriteDocuments(results);
```

En este caso, buscamos en todo el índice la palabra "motel" en cualquier campo de búsqueda, y solo queremos recuperar los nombres de hotel, según lo especificado por el parámetro `Select`. He aquí los resultados:

```output
Name: Secret Point Motel

Name: Twin Dome Motel
```

La consulta siguiente es un poco más interesante.  Queremos encontrar hoteles que tengan habitaciones con una tarifa por noche inferior a 100 $, y que solo se devuelvan el identificador del hotel y la descripción:

```csharp
parameters =
    new SearchParameters()
    {
        Filter = "Rooms/any(r: r/BaseRate lt 100)",
        Select = new[] { "HotelId", "Description" }
    };

results = indexClient.Documents.Search<Hotel>("*", parameters);

WriteDocuments(results);
```

Esta consulta usa una expresión `$filter` de OData, `Rooms/any(r: r/BaseRate lt 100)`, para filtrar los documentos del índice. Aquí se usa [cualquier operador](./search-query-odata-collection-operators.md) para aplicar "BaseRate lt 100" a todos los elementos de la colección de objetos Rooms. Puede encontrar más información acerca de la sintaxis de OData admitida por Búsqueda cognitiva de Azure [aquí](./query-odata-filter-orderby-syntax.md).

Estos son los resultados de la consulta:

```output
HotelId: 1
Description: The hotel is ideally located on the main commercial artery of the city in the heart of New York...

HotelId: 2
Description: The hotel is situated in a nineteenth century plaza, which has been expanded and renovated to...
```

Seguidamente, quiere encontrar los dos mejores hoteles que se han renovado más recientemente y mostrar el nombre del hotel y la última fecha de renovación. Este es el código: 

```csharp
parameters =
    new SearchParameters()
    {
        OrderBy = new[] { "LastRenovationDate desc" },
        Select = new[] { "HotelName", "LastRenovationDate" },
        Top = 2
    };

results = indexClient.Documents.Search<Hotel>("*", parameters);

WriteDocuments(results);
```

En este caso, usamos de nuevo la sintaxis de OData para especificar el parámetro `OrderBy` como `lastRenovationDate desc`. También establecemos `Top` en 2 para tener la seguridad de que solo obtenemos los dos documentos principales. Como antes, establecemos `Select` para especificar los campos que se deben devolver.

He aquí los resultados:

```output
Name: Fancy Stay        Last renovated on: 6/27/2010 12:00:00 AM +00:00
Name: Roach Motel       Last renovated on: 4/28/1982 12:00:00 AM +00:00
```

Por último, queremos encontrar todos los nombres de hoteles que coincidan con la palabra "hotel":

```csharp
parameters = new SearchParameters()
{
    SearchFields = new[] { "HotelName" }
};
results = indexClient.Documents.Search<Hotel>("hotel", parameters);

WriteDocuments(results);
```

Y estos son los resultados, que incluyen todos los campos, ya que no se especificó la propiedad `Select`:

```output
    HotelId: 3
    Name: Triple Landscape Hotel
    ...
```

Este paso finaliza el tutorial, pero no se detenga aquí. **En los pasos siguientes se proporcionan recursos adicionales para obtener más información sobre Búsqueda cognitiva de Azure.

## <a name="next-steps"></a>Pasos siguientes
* Examine las referencias del [SDK para .NET](/dotnet/api/microsoft.azure.search) y la [API de REST](/rest/api/searchservice/).
* Consulte las [convenciones de nomenclatura](/rest/api/searchservice/Naming-rules) para conocer las reglas que deben seguir los nombres de diversos objetos.
* Revise los [tipos de datos admitidos](/rest/api/searchservice/Supported-data-types) en Búsqueda cognitiva de Azure.