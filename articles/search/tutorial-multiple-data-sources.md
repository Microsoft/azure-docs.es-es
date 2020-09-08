---
title: Tutorial de C# sobre la indexación de varios orígenes de datos de Azure
titleSuffix: Azure Cognitive Search
description: Aprenda a importar datos desde varios orígenes en un único índice de Azure Cognitive Search mediante indexadores. Este tutorial y el código de ejemplo están en C#.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 06/20/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: d3dd75d246c1f74253a9ce910e50b05402065464
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88998465"
---
# <a name="tutorial-index-from-multiple-data-sources-using-the-net-sdk"></a>Tutorial: Indexación de varios orígenes de datos mediante el SDK de .NET

Azure Cognitive Search puede importar, analizar e indexar datos desde varios orígenes de datos en un único índice de búsqueda consolidado. Esto es compatible con situaciones donde se agregan datos estructurados con datos menos estructurados o datos de texto sin formato, o incluso de otros orígenes, como documentos de texto, HTML o JSON.

En este tutorial se describe cómo indexar datos de hoteles desde un origen de datos de Azure Cosmos DB y combinarlos con los detalles de las habitaciones de hotel dibujados de los documentos de Azure Blob Storage. El resultado será un índice de búsqueda combinado de hoteles que contiene tipos de datos complejos.

En este tutorial se usa C# y el [SDK de .NET](/dotnet/api/overview/azure/search). En este tutorial, realizará las siguientes tareas:

> [!div class="checklist"]
> * Cargar datos de ejemplo y crear orígenes de datos
> * Identificar la clave del documento
> * Definir y crear el índice
> * Indexar los datos de los hoteles desde Azure Cosmos DB
> * Combinar los datos de las habitaciones de hotel desde Blob Storage

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

+ [Azure Cosmos DB](../cosmos-db/create-cosmosdb-resources-portal.md)
+ [Almacenamiento de Azure](../storage/common/storage-account-create.md)
+ [Visual Studio 2019](https://visualstudio.microsoft.com/)
+ [Creación](search-create-service-portal.md) o [búsqueda de un servicio de búsqueda existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) 

> [!Note]
> Puede usar un servicio gratuito para este tutorial. Un servicio de búsqueda gratuito tiene una limitación de tres índices, tres indexadores y tres orígenes de datos. En este tutorial se crea uno de cada uno. Antes de empezar, asegúrese de que haya espacio en el servicio para aceptar los nuevos recursos.

## <a name="download-files"></a>Descarga de archivos

El código fuente para este tutorial se encuentra en el repositorio de GitHub [azure-search-dotnet-samples](https://github.com/Azure-Samples/azure-search-dotnet-samples), en la carpeta [multiple-data-sources](https://github.com/Azure-Samples/azure-search-dotnet-samples/tree/master/multiple-data-sources).

## <a name="1---create-services"></a>1: Creación de servicios

En este tutorial se usa Azure Cognitive Search para la indexación y las consultas, Azure Cosmos DB para un conjunto de datos y Azure Blob Storage para el segundo conjunto de datos. 

Si es posible, cree todos los servicios en la misma región y grupo de recursos por proximidad y capacidad de administración. En la práctica, los servicios pueden estar en cualquier región.

Este ejemplo utiliza dos conjuntos pequeños de datos que describen las siete hoteles ficticios. Uno de ellos describe los hoteles en sí y se cargará en una base de datos de Azure Cosmos DB. El otro contiene los detalles de las habitaciones y se proporciona como siete archivos JSON independientes que se cargarán en Azure Blob Storage.

### <a name="start-with-cosmos-db"></a>Comenzar con Cosmos DB

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a la página Introducción de la cuenta de Azure Cosmos DB.

1. Seleccione **Explorador de datos** y luego **Nueva base de datos**.

   ![Crear una base de datos nueva](media/tutorial-multiple-data-sources/cosmos-newdb.png "Creación de una base de datos")

1. Escriba el nombre **hotel-rooms-db**. Acepte los valores predeterminados para la configuración restante.

   ![Configuración de la base de datos](media/tutorial-multiple-data-sources/cosmos-dbname.png "Configuración de la base de datos")

1. Cree un contenedor nuevo. Use la base de datos existente que acaba de crear. Escriba **hotels** como nombre del contenedor y use **/HotelId** como clave de partición.

   ![Agregar contenedor](media/tutorial-multiple-data-sources/cosmos-add-container.png "Agregar contenedor")

1. Seleccione **Elementos** en **hotels**y luego haga clic en **Upload Item** (Cargar elemento) en la barra de comandos. Vaya al archivo **cosmosdb/HotelsDataSubset_CosmosDb.json** de la carpeta del proyecto y selecciónelo.

   ![Carga en la colección de Azure Cosmos DB](media/tutorial-multiple-data-sources/cosmos-upload.png "Carga en una colección de Cosmos DB")

1. Utilice el botón Actualizar para actualizar la vista de los elementos de la colección de hoteles. Verá siete nuevos documentos de base de datos enumerados.

### <a name="azure-blob-storage"></a>Azure Blob Storage

1. Inicie sesión en [Azure Portal](https://portal.azure.com), vaya a su cuenta de Azure Storage, haga clic en **Blobs** y, después, en **+Contenedor**.

1. [Cree un contenedor de blobs](../storage/blobs/storage-quickstart-blobs-portal.md) denominado **hotel-rooms** para almacenar los archivos JSON de las habitaciones de hotel de ejemplo. Puede establecer el nivel de acceso público a cualquiera de sus valores válidos.

   ![Creación de un contenedor de blobs](media/tutorial-multiple-data-sources/blob-add-container.png "Creación de un contenedor de blobs")

1. Una vez creado el contenedor, ábralo y seleccione **Cargar** en la barra de comandos. Vaya a la carpeta que contiene los archivos de ejemplo. Selecciónelos todos ellos y haga clic en **Cargar**.

   ![Carga de archivos](media/tutorial-multiple-data-sources/blob-upload.png "Carga de archivos")

Una vez finalizada la carga, los archivos deberían aparecer en la lista del contenedor de datos.

### <a name="azure-cognitive-search"></a>Azure Cognitive Search

El tercer componente es Azure Cognitive Search, que se puede [crear en el portal](search-create-service-portal.md). Puede usar el nivel Gratis para completar este tutorial. 

### <a name="get-an-admin-api-key-and-url-for-azure-cognitive-search"></a>Obtención de una clave de API de administración y una dirección URL para Azure Cognitive Search

Para interactuar con el servicio Azure Cognitive Search, necesitará la dirección URL del servicio y una clave de acceso. Con ambos se crea un servicio de búsqueda, por lo que, si ha agregado Azure Cognitive Search a su suscripción, siga estos pasos para obtener la información necesaria:

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y en la página **Introducción** del servicio de búsqueda, obtenga la dirección URL. Un punto de conexión de ejemplo podría ser similar a `https://mydemo.search.windows.net`.

1. En **Configuración** > **Claves**, obtenga una clave de administrador para tener derechos completos en el servicio. Se proporcionan dos claves de administrador intercambiables para lograr la continuidad empresarial, por si necesitara sustituir una de ellas. Puede usar la clave principal o secundaria en las solicitudes para agregar, modificar y eliminar objetos.

   Obtenga también la clave de consulta. Es una práctica recomendada emitir solicitudes de consulta con acceso de solo lectura.

   ![Obtención del nombre del servicio y las claves de consulta y administrador](media/search-get-started-nodejs/service-name-and-keys.png)

Tener una clave válida genera la confianza, solicitud a solicitud, entre la aplicación que envía la solicitud y el servicio que se encarga de ella.

## <a name="2---set-up-your-environment"></a>2: Configuración del entorno

1. Inicie Visual Studio 2019 y, en el menú **Herramientas**, seleccione **Administrador de paquetes NuGet** y luego **Administrar paquetes NuGet para la solución...** 

1. En la pestaña **Examinar**, busque e instale **Microsoft.Azure.Search** (versión 9.0.1 o posterior). Tendrá que hacer clic en algunos cuadros de diálogo adicionales para completar la instalación.

    ![Uso de NuGet para agregar bibliotecas de Azure](./media/tutorial-csharp-create-first-app/azure-search-nuget-azure.png)

1. Busque el paquete NuGet **Microsoft.Extensions.Configuration.Json** e instálelo también.

1. Abra el archivo de solución **AzureSearchMultipleDataSources.sln**.

1. En el Explorador de soluciones, edite el archivo **appsettings.json** para agregar información de conexión.  

    ```json
    {
      "SearchServiceName": "Put your search service name here",
      "SearchServiceAdminApiKey": "Put your primary or secondary API key here",
      "BlobStorageAccountName": "Put your Azure Storage account name here",
      "BlobStorageConnectionString": "Put your Azure Blob Storage connection string here",
      "CosmosDBConnectionString": "Put your Cosmos DB connection string here",
      "CosmosDBDatabaseName": "hotel-rooms-db"
    }
    ```

Las dos primeras entradas usan las claves de dirección URL y de administrador del servicio Azure Cognitive Search. Por ejemplo, dado un punto de conexión de `https://mydemo.search.windows.net`, el nombre del servicio que debe proporcionar es `mydemo`.

Las siguientes entradas especifican nombres de cuenta e información de la cadena de conexión de los orígenes de datos de Azure Blob Storage y Azure Cosmos DB.

## <a name="3---map-key-fields"></a>3: Asignación de campos de clave

La combinación de contenido requiere que ambos flujos de datos tengan como destino los mismos documentos en el índice de búsqueda. 

En Azure Cognitive Search, el campo de clave identifica de manera exclusiva cada documento. Cada índice de búsqueda debe tener exactamente un campo clave de tipo `Edm.String`. Ese campo de clave debe estar presente para cada documento de un origen de datos que se agregue al índice. (de hecho, es el único campo obligatorio).

Al indexar datos de varios orígenes de datos, asegúrese de que cada documento o fila entrante contiene una clave de documento común para combinar datos de dos documentos de origen físicamente distintos en un nuevo documento de búsqueda en el índice combinado. 

A menudo requiere planeación inicial para identificar una clave de documentación significativa para el índice; asegúrese de que existe en ambos orígenes de datos. En esta demostración, la clave `HotelId` de cada hotel de Cosmos DB también está presente en los blobs JSON de Blob Storage.

Los indexadores de Azure Cognitive Search pueden usar asignaciones de campo para cambiar el nombre e incluso el formato de campos de datos durante la indexación para poder dirigir los datos de origen al campo de índice correcto. Por ejemplo, en Cosmos DB, el identificador de los hoteles se denomina **`HotelId`** . Pero en los archivos de blob JSON de las habitaciones de los hoteles, se denomina **`Id`** . El programa se encarga de ello mediante la asignación del campo **`Id`** desde los blobs hasta el campo de clave **`HotelId`** del índice.

> [!NOTE]
> En la mayoría de los casos, las claves de documento generadas automáticamente, como las que crean algunos indexadores de forma predeterminada, no son buenas claves de documento para los índices combinados. En general, es preferible usar un valor de clave único y significativo que ya exista en los orígenes de datos o que se pueda agregar fácilmente a ellos.

## <a name="4---explore-the-code"></a>4: Exploración del código

Una vez que los valores de configuración y los datos estén en su lugar, el ejemplo de programa de **AzureSearchMultipleDataSources.sln** estará listo para la compilación y la ejecución.

Esta sencilla aplicación de consola de C#/.NET realiza las siguientes tareas:

* Crea un nuevo índice basado en la estructura de datos de la clase Hotel de C# (que también hace referencia a las clases Address y Room).
* Crea un nuevo origen de datos y un indexador que asigna los datos de Azure Cosmos DB a campos de índice. Ambos son objetos de Azure Cognitive Search.
* Ejecuta el indexador para cargar datos de Hotel desde Cosmos DB.
* Crea un segundo origen de datos y un indexador que asigna datos de blob JSON a campos de índice.
* Ejecuta el segundo indexador para cargar datos de Rooms desde Blob Storage.

 Antes de ejecutar el programa, dedíquele un minuto a estudiar el código y las definiciones de índice e indexador de este ejemplo. El código pertinente aparece en dos archivos:

  + **Hotel.cs** contiene el esquema que define el índice.
  + **Program.cs** contiene las funciones que crean el índice de Azure Cognitive Search, los orígenes de datos y los indexadores, y cargan los resultados combinados en el índice.

### <a name="create-an-index"></a>Creación de un índice

Este programa de ejemplo usa el SDK de .NET para definir y crear un índice de Azure Cognitive Search. Aprovecha la clase [FieldBuilder](/dotnet/api/microsoft.azure.search.fieldbuilder) para generar una estructura de índice a partir de una clase de modelo de datos de C#.

El modelo de datos se define mediante la clase Hotel, que también contiene referencias a las clases Address y Room. FieldBuilder explora en profundidad varias definiciones de clase para generar una estructura de datos compleja para el índice. Se usan etiquetas de metadatos para definir los atributos de cada campo, como si se puede buscar u ordenar.

Los siguientes fragmentos de código del archivo **Hotel.cs** muestran cómo se pueden especificar un campo único y una referencia a otra clase de modelo de datos.

```csharp
. . . 
[IsSearchable, IsFilterable, IsSortable]
public string HotelName { get; set; }
. . .
public Room[] Rooms { get; set; }
. . .
```

En el archivo **Program.cs**, el índice se define con un nombre y una colección de campos generados por el método `FieldBuilder.BuildForType<Hotel>()` y se crea como sigue:

```csharp
private static async Task CreateIndex(string indexName, SearchServiceClient searchService)
{
    // Create a new search index structure that matches the properties of the Hotel class.
    // The Address and Room classes are referenced from the Hotel class. The FieldBuilder
    // will enumerate these to create a complex data structure for the index.
    var definition = new Index()
    {
        Name = indexName,
        Fields = FieldBuilder.BuildForType<Hotel>()
    };
    await searchService.Indexes.CreateAsync(definition);
}
```

### <a name="create-azure-cosmos-db-data-source-and-indexer"></a>Creación de un origen de datos de Azure Cosmos DB y de un indexador

A continuación, el programa principal incluirá lógica para crear el origen de datos de Azure Cosmos DB para los datos de los hoteles.

En primer lugar, concatena el nombre de la base de datos de Azure Cosmos DB con la cadena de conexión. A continuación, define el objeto de origen de datos, incluida la configuración específica de los orígenes de Azure Cosmos DB, como la propiedad [useChangeDetection].

  ```csharp
private static async Task CreateAndRunCosmosDbIndexer(string indexName, SearchServiceClient searchService)
{
    // Append the database name to the connection string
    string cosmosConnectString = 
        configuration["CosmosDBConnectionString"]
        + ";Database=" 
        + configuration["CosmosDBDatabaseName"];

    DataSource cosmosDbDataSource = DataSource.CosmosDb(
        name: configuration["CosmosDBDatabaseName"], 
        cosmosDbConnectionString: cosmosConnectString,
        collectionName: "hotels",
        useChangeDetection: true);

    // The Azure Cosmos DB data source does not need to be deleted if it already exists,
    // but the connection string might need to be updated if it has changed.
    await searchService.DataSources.CreateOrUpdateAsync(cosmosDbDataSource);
  ```

Una vez creado el origen de datos, el programa configura un indexador de Azure Cosmos DB denominado **hotel-rooms-cosmos-indexer**.

```csharp
    Indexer cosmosDbIndexer = new Indexer(
        name: "hotel-rooms-cosmos-indexer",
        dataSourceName: cosmosDbDataSource.Name,
        targetIndexName: indexName,
        schedule: new IndexingSchedule(TimeSpan.FromDays(1)));
    
    // Indexers keep metadata about how much they have already indexed.
    // If we already ran this sample, the indexer will remember that it already
    // indexed the sample data and not run again.
    // To avoid this, reset the indexer if it exists.
    bool exists = await searchService.Indexers.ExistsAsync(cosmosDbIndexer.Name);
    if (exists)
    {
        await searchService.Indexers.ResetAsync(cosmosDbIndexer.Name);
    }
    await searchService.Indexers.CreateOrUpdateAsync(cosmosDbIndexer);
```
El programa eliminará los indexadores existentes con el mismo nombre antes de crear el nuevo, por si desea ejecutar este ejemplo más veces.

Este ejemplo define una programación para el indexador para que se ejecute una vez al día. Puede quitar la propiedad de programación de esta llamada si no desea que el indexador se vuelva a ejecutar automáticamente en el futuro.

### <a name="index-azure-cosmos-db-data"></a>Indexación de datos de Azure Cosmos DB

Una vez creados el origen de datos y el indexador, el código que ejecuta este último es breve:

```csharp
    try
    {
        await searchService.Indexers.RunAsync(cosmosDbIndexer.Name);
    }
    catch (CloudException e) when (e.Response.StatusCode == (HttpStatusCode)429)
    {
        Console.WriteLine("Failed to run indexer: {0}", e.Response.Content);
    }
```

Este ejemplo incluye un bloque sencillo try-catch para notificar los errores que puedan producirse durante la ejecución.

Tras la ejecución del indexador de Azure Cosmos DB, el índice de búsqueda contendrá un conjunto integral de documentos de ejemplo de hoteles. Sin embargo, el campo de habitaciones de cada hotel será una matriz vacía, ya que el origen de datos de Azure Cosmos DB no contenía detalles de las habitaciones. A continuación, el programa realizará una extracción de Blob Storage para cargar y combinar los datos de las habitaciones.

### <a name="create-blob-storage-data-source-and-indexer"></a>Creación del indexador y del origen de datos de Blob Storage

Para obtener los detalles de las habitaciones, el programa primero configura el origen de datos de Blob Storage para hacer referencia a un conjunto de archivos de blob JSON individuales.

```csharp
private static async Task CreateAndRunBlobIndexer(string indexName, SearchServiceClient searchService)
{
    DataSource blobDataSource = DataSource.AzureBlobStorage(
        name: configuration["BlobStorageAccountName"],
        storageConnectionString: configuration["BlobStorageConnectionString"],
        containerName: "hotel-rooms");

    // The blob data source does not need to be deleted if it already exists,
    // but the connection string might need to be updated if it has changed.
    await searchService.DataSources.CreateOrUpdateAsync(blobDataSource);
```

Una vez creado el origen de datos, el programa configura un indexador de blobs denominado **hotel-rooms-blob-indexer**.

```csharp
    // Add a field mapping to match the Id field in the documents to 
    // the HotelId key field in the index
    List<FieldMapping> map = new List<FieldMapping> {
        new FieldMapping("Id", "HotelId")
    };

    Indexer blobIndexer = new Indexer(
        name: "hotel-rooms-blob-indexer",
        dataSourceName: blobDataSource.Name,
        targetIndexName: indexName,
        fieldMappings: map,
        parameters: new IndexingParameters().ParseJson(),
        schedule: new IndexingSchedule(TimeSpan.FromDays(1)));

    // Reset the indexer if it already exists
    bool exists = await searchService.Indexers.ExistsAsync(blobIndexer.Name);
    if (exists)
    {
        await searchService.Indexers.ResetAsync(blobIndexer.Name);
    }
    await searchService.Indexers.CreateOrUpdateAsync(blobIndexer);
```

Los blobs JSON contienen un campo de clave denominado **`Id`** en lugar de **`HotelId`** . El código usa la clase `FieldMapping` para indicar al indexador que dirija el valor del campo **`Id`** a la clave de documento **`HotelId`** del índice.

Los indexadores de Blob Storage pueden usar parámetros que identifiquen el modo de análisis que usar. El modo de análisis es diferente para los blobs que representan un único documento o varios en el mismo blob. En este ejemplo, cada blob representa un documento de índice único, por lo que el código usa el parámetro `IndexingParameters.ParseJson()`.

Para más información sobre los parámetros de análisis del indexador, consulte el artículo sobre la [Indexación de blobs JSON](search-howto-index-json-blobs.md). Para más información sobre la especificación de estos parámetros con el SDK de .NET, consulte la clase [IndexerParametersExtension](/dotnet/api/microsoft.azure.search.models.indexingparametersextensions).

El programa eliminará los indexadores existentes con el mismo nombre antes de crear el nuevo, por si desea ejecutar este ejemplo más veces.

Este ejemplo define una programación para el indexador para que se ejecute una vez al día. Puede quitar la propiedad de programación de esta llamada si no desea que el indexador se vuelva a ejecutar automáticamente en el futuro.

### <a name="index-blob-data"></a>Indexación de datos de blob

Una vez creados el origen de datos de Blob Storage y el indexador, el código que ejecuta el indexador es simple:

```csharp
    try
    {
        await searchService.Indexers.RunAsync(cosmosDbIndexer.Name);
    }
    catch (CloudException e) when (e.Response.StatusCode == (HttpStatusCode)429)
    {
        Console.WriteLine("Failed to run indexer: {0}", e.Response.Content);
    }
```

Dado que el índice ya se ha rellenado con datos de los hoteles desde la base de datos de Azure Cosmos DB, el indexador de blobs actualiza los documentos existentes en el índice y agrega los detalles de las habitaciones.

> [!NOTE]
> Si tiene los mismos campos (distintos del clave) en ambos orígenes de datos y los datos de esos campos no coinciden, el índice contendrá los valores del indexador que se ejecutara más recientemente. En nuestro ejemplo, ambos orígenes de datos contienen un campo **HotelName**. Si, por alguna razón, los datos de este campo difieren para documentos con el mismo valor de calve, los datos de **HotelName** del origen de datos que se indexara más recientemente serán los que se almacenen en el índice como valor.

## <a name="5---search"></a>5: Búsqueda

Puede explorar el índice de búsqueda relleno tras la ejecución del programa con el [**Explorador de búsqueda**](search-explorer.md) del portal.

En Azure Portal, abra la página **Introducción** del servicio de búsqueda y busque el índice **hotel-rooms-sample** en la lista **Índices**.

  ![Lista de índices de Azure Cognitive Search](media/tutorial-multiple-data-sources/index-list.png "Lista de índices de Azure Cognitive Search")

Haga clic en el índice hotel-rooms-sample en la lista. Verá una interfaz del Explorador de búsqueda para el índice. Escriba una consulta con un término, como "Luxury". Debería ver al menos un documento en los resultados y este documento debería mostrar una lista de objetos de las habitaciones de la matriz.

## <a name="reset-and-rerun"></a>Restablecer y volver a ejecutar

En las primeras etapas experimentales de desarrollo, el enfoque más práctico para las iteraciones del diseño es eliminar los objetos de Azure Cognitive Search y permitir que el código vuelva a generarlos. Los nombres de los recursos son únicos. La eliminación de un objeto permite volver a crearlo con el mismo nombre.

En el código de ejemplo de este tutorial se comprueban los objetos existentes y se eliminan para que pueda volver a ejecutar el código.

También puede usar el portal para eliminar los índices, indexadores y orígenes de datos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, al final de un proyecto, es recomendable eliminar los recursos que ya no necesite. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede buscar y administrar los recursos en el portal, mediante el vínculo Todos los recursos o Grupos de recursos en el panel de navegación izquierdo.

## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con el concepto de ingesta de datos de varios orígenes, echemos un vistazo más de cerca a la configuración del indexador, comenzando por Cosmos DB.

> [!div class="nextstepaction"]
> [Configuración de un indexador de Azure Cosmos DB](search-howto-index-cosmosdb.md)