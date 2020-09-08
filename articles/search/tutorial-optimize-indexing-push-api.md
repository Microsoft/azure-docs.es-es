---
title: 'Tutorial: Optimización de la indexación mediante la API de inserción'
titleSuffix: Azure Cognitive Search
description: Aprenda a indexar datos de forma eficaz mediante la API de inserción de Azure Cognitive Search. Este tutorial y el código de ejemplo están en C#.
manager: liamca
author: dereklegenzoff
ms.author: delegenz
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 08/21/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: bfb2598fb3a207bbdfaade9086efd07827b077dd
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998431"
---
# <a name="tutorial-optimize-indexing-with-the-push-api"></a>Tutorial: Optimización de la indexación mediante la API de inserción

Azure Cognitive Search admite [dos enfoques básicos](search-what-is-data-import.md) para importar datos a un índice de búsqueda: *insertar* los datos en el índice mediante programación, o bien señalar un [indexador de Azure Cognitive Search](search-indexer-overview.md) en un origen de datos admitido para *extraer* los datos.

En este tutorial, se describe cómo indexar los datos de forma eficaz utilizando el [modelo de inserción](search-what-is-data-import.md#pushing-data-to-an-index) mediante la creación de solicitudes por lotes y el uso de una estrategia de reintento de retroceso exponencial. Es posible [descarga y ejecutar la aplicación](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/optimize-data-indexing). En este artículo, se explican los aspectos fundamentales de la aplicación y los factores que se deben tener en cuenta a la hora de indexar datos.

En este tutorial, usará C# y el [SDK de .NET](/dotnet/api/overview/azure/search) para realizar las tareas siguientes:

> [!div class="checklist"]
> * Creación de un índice
> * Prueba de varios tamaños de lote para determinar cuál es el más eficaz
> * Indexación de datos de forma asincrónica
> * Uso de varios subprocesos para aumentar las velocidades de indexación
> * Uso de una estrategia de reintento de retroceso exponencial para volver a probar los elementos con errores

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Este tutorial requiere los siguientes servicios y herramientas:

+ [Visual Studio](https://visualstudio.microsoft.com/downloads/), cualquier edición. Se han probado código de ejemplo e instrucciones en la edición Community Edition gratuita.

+ [Cree un servicio Azure Cognitive Search](search-create-service-portal.md) o [busque uno existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) en su suscripción actual.

<a name="get-service-info"></a>

## <a name="download-files"></a>Descarga de archivos

El código fuente para este tutorial está disponible en la carpeta [optimize-data-indexing](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/optimize-data-indexing) del repositorio de GitHub [Azure-Samples/azure-search-dotnet-samples](https://github.com/Azure-Samples/azure-search-dotnet-samples).

## <a name="key-considerations"></a>Consideraciones clave

Al insertar datos en un índice, hay varias consideraciones importantes que afectan a las velocidades de indexación. Puede obtener más información sobre estas consideraciones en el artículo [Indexación de grandes conjuntos de datos en Azure Cognitive Search](search-howto-large-index.md).

Los seis factores principales a tener en cuenta son:

+ **El nivel de servicio y el número de particiones o réplicas**: la adición de particiones y el aumento del nivel incrementará las velocidades de indexación.
+ **Esquema de índice**: la incorporación de campos y de propiedades adicionales a los campos (por ejemplo, que permitan *buscar*, *clasificar* o *filtrar*) reducirá las velocidades de indexación.
+ **Tamaño de lote**: el tamaño de lote óptimo varía en función del esquema de índice y del conjunto de datos.
+ **Número de subprocesos o trabajos**: un único subproceso no aprovecha al máximo las velocidades de indexación.
+ **Estrategia de reintento**: se debe usar una estrategia de reintento de retroceso exponencial para optimizar la indexación.
+ **Velocidades de transferencia de datos de red**: las velocidades de transferencia de datos pueden ser un factor limitador. Indexe los datos desde su entorno de Azure para aumentar las velocidades de transferencia de datos.


## <a name="1---create-azure-cognitive-search-service"></a>1\. Creación del servicio Azure Cognitive Search

Para finalizar este tutorial, necesitará un servicio Azure Cognitive Search, que puede [crear en el portal](search-create-service-portal.md). Se recomienda usar el mismo nivel que tiene previsto utilizar en producción para que pueda probar y optimizar con precisión las velocidades de indexación.

### <a name="get-an-admin-api-key-and-url-for-azure-cognitive-search"></a>Obtención de una clave de API de administración y una dirección URL para Azure Cognitive Search

Las llamadas API requieren la dirección URL del servicio y una clave de acceso. Con ambos se crea un servicio de búsqueda, por lo que, si ha agregado Azure Cognitive Search a su suscripción, siga estos pasos para obtener la información necesaria:

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y en la página **Introducción** del servicio de búsqueda, obtenga la dirección URL. Un punto de conexión de ejemplo podría ser similar a `https://mydemo.search.windows.net`.

1. En **Configuración** > **Claves**, obtenga una clave de administrador para tener derechos completos en el servicio. Se proporcionan dos claves de administrador intercambiables para lograr la continuidad empresarial, por si necesitara sustituir una de ellas. Puede usar la clave principal o secundaria en las solicitudes para agregar, modificar y eliminar objetos.

   ![Obtención de una clave de acceso y un punto de conexión HTTP](media/search-get-started-postman/get-url-key.png "Obtención de una clave de acceso y un punto de conexión HTTP")

## <a name="2---set-up-your-environment"></a>2: Configuración del entorno

1. Inicie Visual Studio y abra **OptimizeDataIndexing. sln**.
1. En el Explorador de soluciones, abra **appsettings.json** para proporcionar información de conexión.
1. Para `searchServiceName`, si la dirección URL completa es "https://my-demo-service.search.windows.net", el nombre del servicio que se va a proporcionar es "my-demo-service".

```json
{
  "SearchServiceName": "<YOUR-SEARCH-SERVICE-NAME>",
  "SearchServiceAdminApiKey": "<YOUR-ADMIN-API-KEY>",
  "SearchIndexName": "optimize-indexing"
}
```

## <a name="3---explore-the-code"></a>3\. Exploración del código

Una vez que actualice *appSettings. json*, el programa de ejemplo de **OptimizeDataIndexing. sln** debería estar listo para la compilación y ejecución.

Este código se deriva de [Inicio rápido de C#](search-get-started-dotnet.md). Encontrará información más detallada sobre los conceptos básicos de trabajar con SDK para .NET en ese artículo.

Esta sencilla aplicación de consola de C#/.NET realiza las siguientes tareas:

+ Crea un nuevo índice basado en la estructura de datos de la clase Hotel de C#, que también hace referencia a las clases Address y Room.
+ Pruebas de varios tamaños de lote para determinar cuál es el más eficaz
+ Indexación de datos de forma asincrónica
    + Uso de varios subprocesos para aumentar las velocidades de indexación
    + Uso de una estrategia de reintento de retroceso exponencial para reintentar los elementos con errores

 Antes de ejecutar el programa, dedique unos minutos a examinar el código y las definiciones de índice de este ejemplo. El código pertinente aparece en varios archivos:

  + **Hotel.cs** y **Address.cs** contienen el esquema que define el índice.
  + **DataGenerator.cs** contiene una clase simple para facilitar la creación de grandes cantidades de datos de Hotel.
  + **ExponentialBackoff.cs** contiene código para optimizar el proceso de indexación, tal y como se describe más adelante.
  + **Program.cs** contiene funciones que crean y eliminan el índice de Azure Cognitive Search, indexa lotes de datos y prueba diferentes tamaños de lote.

### <a name="creating-the-index"></a>Creación del índice

Este programa de ejemplo usa el SDK de .NET para definir y crear un índice de Azure Cognitive Search. Aprovecha la clase [FieldBuilder](/dotnet/api/microsoft.azure.search.fieldbuilder) para generar una estructura de índice a partir de una clase de modelo de datos de C#.

El modelo de datos se define mediante la clase Hotel, que también contiene referencias a la clase Address. FieldBuilder explora en profundidad varias definiciones de clase para generar una estructura de datos compleja para el índice. Se usan etiquetas de metadatos para definir los atributos de cada campo (por ejemplo, si permiten hacer búsquedas o clasificaciones).

Los siguientes fragmentos de código del archivo **Hotel.cs** muestran cómo se pueden especificar un campo único y una referencia a otra clase de modelo de datos.

```csharp
. . .
[IsSearchable, IsSortable]
public string HotelName { get; set; }
. . .
public Address Address { get; set; }
. . .
```

En el archivo **Program.cs**, el índice se define con un nombre y una colección de campos generados por el método `FieldBuilder.BuildForType<Hotel>()` y se crea como sigue:

```csharp
private static async Task CreateIndex(string indexName, SearchServiceClient searchService)
{
    // Create a new search index structure that matches the properties of the Hotel class.
    // The Address class is referenced from the Hotel class. The FieldBuilder
    // will enumerate these to create a complex data structure for the index.
    var definition = new Index()
    {
        Name = indexName,
        Fields = FieldBuilder.BuildForType<Hotel>()
    };
    await searchService.Indexes.CreateAsync(definition);
}
```

### <a name="generating-data"></a>Generación de datos

Una clase simple se implementa en el archivo **DataGenerator.cs** para generar datos con fines de prueba. La única finalidad de esta clase es facilitar la generación de un gran número de documentos con un identificador único para la indexación.

Para obtener una lista de los 100 000 hoteles con identificadores únicos, debe ejecutar las dos líneas de código siguientes:

```csharp
DataGenerator dg = new DataGenerator();
List<Hotel> hotels = dg.GetHotels(100000, "large");
```

Hay dos tamaños de hotel disponibles para las pruebas de este ejemplo: **pequeño** y **grande**.

El esquema del índice puede tener un efecto significativo en las velocidades de indexación. Debido a ese efecto, tiene sentido convertir esta clase para generar datos que coincidan con el esquema de índice previsto después de ejecutar este tutorial.

## <a name="4---test-batch-sizes"></a>4\. Tamaños de los lotes de pruebas

Azure Cognitive Search admite las siguientes API para cargar uno o varios documentos en un índice:

+ [Agregar, Actualizar o Eliminar documentos (API de REST)](/rest/api/searchservice/AddUpdate-or-Delete-Documents)
+ [Clase indexAction](/dotnet/api/microsoft.azure.search.models.indexaction?view=azure-dotnet) o [clase indexBatch](/dotnet/api/microsoft.azure.search.models.indexbatch?view=azure-dotnet)

La indexación de documentos en lotes mejora significativamente el rendimiento de la indexación. Estos lotes pueden incluir hasta 1000 documentos o unos 16 MB por lote.

Determinar el tamaño de lote óptimo para los datos es un aspecto fundamental de la optimización de las velocidades de indexación. Los dos principales aspectos que influyen en el tamaño de lote óptimo son:

+ El esquema del índice
+ El tamaño de los datos

Dado que el tamaño de lote óptimo depende del índice y los datos, el mejor enfoque consiste en probar distintos tamaños de lote para determinar cuál ofrece mayores velocidades de indexación en su caso.

La siguiente función muestra un enfoque sencillo para probar los tamaños de lote.

```csharp
public static async Task TestBatchSizes(ISearchIndexClient indexClient, int min = 100, int max = 1000, int step = 100, int numTries = 3)
{
    DataGenerator dg = new DataGenerator();

    Console.WriteLine("Batch Size \t Size in MB \t MB / Doc \t Time (ms) \t MB / Second");
    for (int numDocs = min; numDocs <= max; numDocs += step)
    {
        List<TimeSpan> durations = new List<TimeSpan>();
        double sizeInMb = 0.0;
        for (int x = 0; x < numTries; x++)
        {
            List<Hotel> hotels = dg.GetHotels(numDocs, "large");

            DateTime startTime = DateTime.Now;
            await UploadDocuments(indexClient, hotels);
            DateTime endTime = DateTime.Now;
            durations.Add(endTime - startTime);

            sizeInMb = EstimateObjectSize(hotels);
        }

        var avgDuration = durations.Average(timeSpan => timeSpan.TotalMilliseconds);
        var avgDurationInSeconds = avgDuration / 1000;
        var mbPerSecond = sizeInMb / avgDurationInSeconds;

        Console.WriteLine("{0} \t\t {1} \t\t {2} \t\t {3} \t {4}", numDocs, Math.Round(sizeInMb, 3), Math.Round(sizeInMb / numDocs, 3), Math.Round(avgDuration, 3), Math.Round(mbPerSecond, 3));

        // Pausing 2 seconds to let the search service catch its breath
        Thread.Sleep(2000);
    }
}
```

Dado que no todos los documentos tienen el mismo tamaño (aunque en este ejemplo sí lo tengan), calcularemos el tamaño de los datos que se van a enviar al servicio de búsqueda. Para ello, se usa la función siguiente que primero convierte el objeto al formato JSON y luego determina el tamaño en bytes. Esta técnica nos permite determinar qué tamaños de lote son más eficaces en términos de MB/s y de velocidades de indexación.

```csharp
public static double EstimateObjectSize(object data)
{
    // converting data to json for more accurate sizing
    var json = JsonConvert.SerializeObject(data);

    // converting object to byte[] to determine the size of the data
    BinaryFormatter bf = new BinaryFormatter();
    MemoryStream ms = new MemoryStream();
    byte[] Array;

    bf.Serialize(ms, json);
    Array = ms.ToArray();

    // converting from bytes to megabytes
    double sizeInMb = (double)Array.Length / 1000000;

    return sizeInMb;
}
```

La función requiere un `ISearchIndexClient`, así como el número de intentos que deseamos probar con cada tamaño de lote. Como los tiempos de indexación de cada lote pueden variar, cada lote se prueba tres veces de forma predeterminada para que el valor estadístico de los resultados sea más significativo.

```csharp
await TestBatchSizes(indexClient, numTries: 3);
```

Al ejecutar la función, su consola debería mostrar resultados similares a los siguientes:

   ![Salida de la función de tamaño de lotes de pruebas](media/tutorial-optimize-data-indexing/test-batch-sizes.png "Salida de la función de tamaño de lotes de pruebas")

Identifique el tamaño de lote más eficaz y utilice ese tamaño de lote en el siguiente paso del tutorial. Es posible que vea una valor estable en MB/s para distintos tamaños de lote.

## <a name="5---index-data"></a>5\. Datos de índice

Después de identificar el tamaño de lote que vamos a usar, el siguiente paso es empezar a indexar los datos. Para indexar los datos de forma eficaz, este ejemplo:

* Usa varios subprocesos o trabajos.
* Adopta una estrategia de reintentos de retroceso exponencial.

### <a name="use-multiple-threadsworkers"></a>Uso de varios subprocesos o trabajos

Para aprovechar al máximo las velocidades de indexación de Azure Cognitive Search, es posible que tenga que usar varios subprocesos para enviar solicitudes de indexación por lotes de forma simultánea al servicio.  

Algunas de las principales consideraciones mencionadas anteriormente afectan al número óptimo de subprocesos. Puede modificar este ejemplo y probar diferentes números de subprocesos para determinar cuál es el número óptimo de subprocesos en su caso. Sin embargo, siempre que tenga varios subprocesos que se ejecuten simultáneamente, se deberían poder aprovechar al máximo la mayoría de las ventajas de eficiencia.

A medida que aumenta las solicitudes que alcanzan el servicio de búsqueda, puede encontrarse con [códigos de estado HTTP](/rest/api/searchservice/http-status-codes) que indican que la solicitud no se completó correctamente. Durante la indexación, dos de los códigos de estado HTTP comunes son:

+ **Servicio no disponible 503**: este error significa que el sistema está sobrecargado y la solicitud no se puede procesar en ese momento.
+ **207 Varios estados**: este error significa que algunos documentos se han procesado correctamente, pero al menos uno generó error.

### <a name="implement-an-exponential-backoff-retry-strategy"></a>Implementación de una estrategia de reintento de retroceso exponencial

Si se produce un error, las solicitudes deberán reintentarse utilizando una [estrategia de reintento de retroceso exponencial](/dotnet/architecture/microservices/implement-resilient-applications/implement-retries-exponential-backoff).

El SDK de Azure Cognitive Search para .NET reintenta automáticamente las solicitudes 503 y otras con error, pero es preciso que implemente su propia lógica para reintentar las solicitudes 207. También se pueden usar herramientas de código abierto como [Polly](https://github.com/App-vNext/Polly) para implementar una estrategia de reintento. 

En este ejemplo, implementamos una estrategia propia de reintento de retroceso exponencial. Para implementar esta estrategia, empezamos por definir algunas variables, como `maxRetryAttempts` y `delay` inicial para una solicitud que ha generado error:

```csharp
// Create batch of documents for indexing
IndexBatch<Hotel> batch = IndexBatch.Upload(hotels);

// Define parameters for exponential backoff
int attempts = 0;
TimeSpan delay = delay = TimeSpan.FromSeconds(2);
int maxRetryAttempts = 5;
```

Es importante detectar [IndexBatchException](/dotnet/api/microsoft.azure.search.indexbatchexception?view=azure-dotnet), ya que estas excepciones indican que la operación de indexación no se llegó a completar correctamente (solicitudes 207). Los elementos con errores deben reintentarse mediante el método `FindFailedActionsToRetry` que facilita la creación de un nuevo lote que solo contenga elementos con errores.

También se detectarán las excepciones distintas de `IndexBatchException`, que indican que la solicitud ha generado un error. Estas excepciones son menos comunes, especialmente con el SDK para .NET, ya que reintenta las solicitudes 503 automáticamente.

```csharp
// Implement exponential backoff
do
{
    try
    {
        attempts++;
        var response = await indexClient.Documents.IndexAsync(batch);
        break;
    }
    catch (IndexBatchException ex)
    {
        Console.WriteLine("[Attempt: {0} of {1} Failed] - Error: {2}", attempts, maxRetryAttempts, ex.Message);

        if (attempts == maxRetryAttempts)
            break;

        // Find the failed items and create a new batch to retry
        batch = ex.FindFailedActionsToRetry(batch, x => x.HotelId);
        Console.WriteLine("Retrying failed documents using exponential backoff...\n");

        Task.Delay(delay).Wait();
        delay = delay * 2;
    }
    catch (Exception ex)
    {
        Console.WriteLine("[Attempt: {0} of {1} Failed] - Error: {2} \n", attempts, maxRetryAttempts, ex.Message);

        if (attempts == maxRetryAttempts)
            break;

        Task.Delay(delay).Wait();
        delay = delay * 2;
    }
} while (true);
```

Desde aquí, encapsulamos el código de retroceso exponencial en una función que permita llamar al código fácilmente.

A continuación, se crea otra función para administrar los subprocesos activos. Para simplificar las cosas, esa función no se incluye aquí, pero se puede ver en [ExponentialBackoff.cs](https://github.com/Azure-Samples/azure-search-dotnet-samples/blob/master/optimize-data-indexing/OptimizeDataIndexing/ExponentialBackoff.cs). Es posible llamar a la función mediante el siguiente comando, donde `hotels` son los datos que queremos cargar, `1000` es el tamaño del lote y `8` es el número de subprocesos simultáneos:

```csharp
ExponentialBackoff.IndexData(indexClient, hotels, 1000, 8).Wait();
```

Al ejecutar la función, debería ver resultados similares a los siguientes:

![Salida de la función de datos de índice](media/tutorial-optimize-data-indexing/index-data-start.png "Salida de la función de datos de índice")

Cuando un lote de documentos genera error, se imprime el error indicado y se reintenta el lote:

![Error de la función de datos de índice](media/tutorial-optimize-data-indexing/index-data-error.png "Salida de la función de tamaño de lotes de pruebas")

Una vez la función se ha ejecutado, puede comprobar si todos los documentos se han agregado al índice.

## <a name="6---explore-index"></a>6\. Exploración del índice

Puede explorar el índice de búsqueda rellenado tras la ejecución del programa mediante programación o mediante el [**Explorador de búsqueda**](search-explorer.md) del portal.

### <a name="programatically"></a>Mediante programación

Hay dos opciones principales para comprobar el número de documentos en un índice: la API de [recuento de documentos](/rest/api/searchservice/count-documents) y la API de [obtención de estadísticas de índice](/rest/api/searchservice/get-index-statistics). Ambas rutas pueden requerir tiempo adicional para la actualización, por lo que no debe extrañarse si el número de documentos devuelto es inferior al esperado inicialmente.

#### <a name="count-documents"></a>Documentos de recuento

La operación de recuento de documentos obtiene el número de documentos en un índice de búsqueda:

```csharp
long indexDocCount = indexClient.Documents.Count();
```

#### <a name="get-index-statistics"></a>Obtención de estadísticas de índice

La operación de obtención de estadísticas de índice devuelve un número de documentos para el índice actual, junto con el uso del almacenamiento. Las estadísticas de índice tardan más en actualizarse que el número de documentos.

```csharp
IndexGetStatisticsResult indexStats = serviceClient.Indexes.GetStatistics(configuration["SearchIndexName"]);
```

### <a name="azure-portal"></a>Azure Portal

En Azure Portal, abra la página de **información general** del servicio de búsqueda y localice el índice **optimize-indexing** en la lista de **Índices**.

  ![Lista de índices de Azure Cognitive Search](media/tutorial-optimize-data-indexing/portal-output.png "Lista de índices de Azure Cognitive Search")

El *recuento de documentos* y el *tamaño de almacenamiento* se basan en la [API de obtención de estadísticas](/rest/api/searchservice/get-index-statistics) y pueden tardar varios minutos en actualizarse.

## <a name="reset-and-rerun"></a>Restablecer y volver a ejecutar

En las primeras etapas experimentales de desarrollo, el enfoque más práctico para las iteraciones del diseño es eliminar los objetos de Azure Cognitive Search y permitir que el código vuelva a generarlos. Los nombres de los recursos son únicos. La eliminación de un objeto permite volver a crearlo con el mismo nombre.

El código de ejemplo de este tutorial comprueba los índices existentes y los elimina para que el usuario pueda volver a ejecutar su código.

Puede usar el portal para eliminar los índices.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, al final de un proyecto, es recomendable eliminar los recursos que ya no necesite. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede encontrar y administrar recursos en el portal, mediante el vínculo **Todos los recursos** o **Grupos de recursos** en el panel de navegación izquierdo.

## <a name="next-steps"></a>Pasos siguientes

Como ya se ha familiarizado con el concepto de ingesta de datos de forma eficaz, vamos a ver más de cerca la arquitectura de consulta de Lucene y cómo funciona la búsqueda de texto completo en Azure Cognitive Search.

> [!div class="nextstepaction"]
> [Funcionamiento de la búsqueda de texto completo en Azure Cognitive Search](search-lucene-query-architecture.md)