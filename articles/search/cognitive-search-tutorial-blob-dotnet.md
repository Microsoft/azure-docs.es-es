---
title: Tutorial de C# sobre el uso de inteligencia artificial en blobs de Azure
titleSuffix: Azure Cognitive Search
description: Recorra un ejemplo de extracción de texto y de procesamiento de lenguaje natural por el contenido del almacenamiento de blobs con el SDK .NET de Azure Cognitive Search y C#.
manager: nitinme
author: MarkHeff
ms.author: maheff
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 08/20/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 09273cf901830e850acca05c57c0b110ffd7e28a
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89002869"
---
# <a name="tutorial-ai-generated-searchable-content-from-azure-blobs-using-the-net-sdk"></a>Tutorial: Contenido de blobs de Azure con capacidad de búsqueda y generado mediante inteligencia artificial con el SDK de .NET

Si tiene texto no estructurado o imágenes en Azure Blob Storage, una [canalización de enriquecimiento de inteligencia artificial](cognitive-search-concept-intro.md) puede extraer la información y crear contenido útil en escenarios de búsqueda de texto completo o minería de conocimiento. En este tutorial de C#, aplique el reconocimiento óptico de caracteres (OCR) en las imágenes y realice el procesamiento de lenguaje natural para crear campos nuevos que pueda aprovechar en las consultas, las facetas y los filtros.

En este tutorial, usará C# y el [SDK de .NET](/dotnet/api/overview/azure/search) para realizar las tareas siguientes:

> [!div class="checklist"]
> * Comience con imágenes y archivos de aplicación en Azure Blob Storage.
> * Defina una canalización para agregar OCR, extracción de texto, detección de idioma y reconocimiento de frases clave y entidades.
> * Defina un índice para almacenar la salida (contenido sin procesar, además de pares nombre-valor generados por canalización).
> * Ejecute la canalización para iniciar transformaciones y análisis, así como para crear y cargar el índice.
> * Explore los resultados mediante la búsqueda de texto completo y una sintaxis de consulta enriquecida.

Si no tiene una suscripción a Azure, abra una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

+ [Almacenamiento de Azure](https://azure.microsoft.com/services/storage/)
+ [Visual Studio](https://visualstudio.microsoft.com/downloads/)
+ [Creación](search-create-service-portal.md) o [búsqueda de un servicio de búsqueda existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) 

> [!Note]
> Puede usar un servicio gratuito para este tutorial. Un servicio de búsqueda gratuito tiene una limitación de tres índices, tres indexadores y tres orígenes de datos. En este tutorial se crea uno de cada uno. Antes de empezar, asegúrese de que haya espacio en el servicio para aceptar los nuevos recursos.

## <a name="download-files"></a>Descarga de archivos

1. Abra esta [carpeta de OneDrive](https://1drv.ms/f/s!As7Oy81M_gVPa-LCb5lC_3hbS-4) y, en la esquina superior izquierda, haga clic en **Descargar** para copiar los archivos en el equipo. 

1. Haga clic con el botón derecho en el archivo ZIP y seleccione **Extraer todo**. Hay 14 archivos de varios tipos. Para este ejercicio, usará 7.

Si lo desea, puede descargar el código fuente de este tutorial. El código fuente está disponible en la carpeta tutorial-ai-enrichment del repositorio [azure-search-dotnet-samples](https://github.com/Azure-Samples/azure-search-dotnet-samples).

## <a name="1---create-services"></a>1: Creación de servicios

En este tutorial se usa Azure Cognitive Search en la indexación y las consultas, Cognitive Services en el back-end para el enriquecimiento de inteligencia artificial y Azure Blob Storage en la provisión de datos. Este tutorial permanece por debajo de la asignación gratuita de 20 transacciones por indexador al día en Cognitive Services, por lo que los únicos servicios que tiene que crear son la búsqueda y el almacenamiento.

Si es posible, cree los dos en la misma región y grupo de recursos para la proximidad y la capacidad de administración. En la práctica, la cuenta de Azure Storage puede estar en cualquier región.

### <a name="start-with-azure-storage"></a>Comienzo con Azure Storage

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y haga clic en **+ Crear un recurso**.

1. Busque *cuenta de almacenamiento* y seleccione la oferta de Cuenta de almacenamiento de Microsoft.

   ![Creación de una cuenta de almacenamiento](media/cognitive-search-tutorial-blob/storage-account.png "Crear cuenta de almacenamiento")

1. En la pestaña Datos básicos, se necesitan los siguientes elementos. Acepte los valores predeterminados para todo lo demás.

   + **Grupo de recursos**. Seleccione un grupo existente o cree uno nuevo, pero use el mismo grupo para todos los servicios, con el fin de que pueda administrarlos colectivamente.

   + **Nombre de cuenta de almacenamiento**. Si cree que puede tener varios recursos del mismo tipo, use el nombre para diferenciarlos por tipo y región, por ejemplo *blobstoragewestus*. 

   + **Ubicación**. Si es posible, elija la misma ubicación que se usa para Azure Cognitive Search y Cognitive Services. Una ubicación única anula los cargos de ancho de banda.

   + **Tipo de cuenta**. Elija el valor predeterminado, *StorageV2 (uso general v2)* .

1. Haga clic en **Revisar y crear** para crear el servicio.

1. Una vez creado, haga clic en **Go to the resource** (Ir al recurso) para abrir la página de información general.

1. Haga clic en el servicio **Blobs**.

1. Haga clic en **+ Contenedor** para crear un contenedor y asígnele el nombre *cog-search-demo*.

1. Seleccione *cog-search-demo* y haga clic en **Cargar** para abrir la carpeta en la que guardó los archivos de descarga. Seleccione los catorce archivos y haga clic en **Aceptar** para cargarlos.

   ![Carga de archivos de ejemplo](media/cognitive-search-quickstart-blob/sample-data.png "Carga de archivos de ejemplo")

1. Antes de salir de Azure Storage, obtenga una cadena de conexión para poder formular una conexión en Azure Cognitive Search. 

   1. Vuelva a la página de información general de la cuenta de almacenamiento (se ha usado *blobstragewestus* como ejemplo). 
   
   1. En el panel de navegación izquierdo, seleccione **Claves de acceso** y copie una de las cadenas de conexión. 

   La cadena de conexión es una dirección URL parecida a la del ejemplo siguiente:

      ```http
      DefaultEndpointsProtocol=https;AccountName=blobstoragewestus;AccountKey=<your account key>;EndpointSuffix=core.windows.net
      ```

1. Guarde la cadena de conexión en el Bloc de notas. La necesitará más adelante, al configurar la conexión del origen de datos.

### <a name="cognitive-services"></a>Cognitive Services

El enriquecimiento de inteligencia artificial está respaldado por Cognitive Services, lo que incluye Text Analytics y Computer Vision para el procesamiento de imágenes y del lenguaje natural. Si su objetivo fuera trabajar con un prototipo o un proyecto reales, en este punto aprovisionaría Cognitive Services (en la misma región que Azure Cognitive Search) para poder conectarlo con las operaciones de indexación.

Sin embargo, en este ejercicio puede omitir el aprovisionamiento de recursos, porque Azure Cognitive Search puede conectarse a Cognitive Services en segundo plano y proporcionarle 20 transacciones gratuitas por cada ejecución del indexador. Dado que en este tutorial se usan 14 transacciones, la asignación gratuita es suficiente. En el caso de proyectos mayores, planee aprovisionamiento de Cognitive Services en el nivel S0 de pago por uso. Para más información, consulte [Asociación de Cognitive Services](cognitive-search-attach-cognitive-services.md).

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

Para comenzar, abra Visual Studio y cree un nuevo proyecto de Aplicación de consola que pueda ejecutarse en .NET Core.

### <a name="install-nuget-packages"></a>Instalación de paquetes NuGet

El [SDK de .NET de Azure Cognitive Search](/dotnet/api/overview/azure/search) consta de algunas bibliotecas de cliente que le permiten administrar los índices, los orígenes de datos, los indexadores y los conjuntos de aptitudes, así como cargar y administrar documentos y ejecutar consultas. Todo ello sin tener que encargarse de los detalles de HTTP y JSON. Esas bibliotecas de cliente se distribuyen como paquetes de NuGet.

Para este proyecto, instale la versión 9 o posterior del paquete NuGet de `Microsoft.Azure.Search`.

1. En un explorador, vaya a la [página de paquetes NuGet de Microsoft.Azure.Search](https://www.nuget.org/packages/Microsoft.Azure.Search).

1. Seleccione la versión más reciente (9 o posterior).

1. Copie el comando del Administrador de paquetes.

1. Abra la Consola del administrador de paquetes. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Consola del Administrador de paquetes**. 

1. Pegue y ejecute el comando que copió en el paso anterior.

Después, instale el último paquete NuGet `Microsoft.Extensions.Configuration.Json`.

1. Seleccione **Herramientas** > **Administrador de paquetes NuGet** > **Administrar paquetes NuGet para la solución...** . 

1. Haga clic en **Examinar** y busque el paquete NuGet `Microsoft.Extensions.Configuration.Json`. 

1. Seleccione el paquete, seleccione el proyecto, confirme que la versión sea la versión estable más reciente y, luego, haga clic en **Instalar**.

### <a name="add-service-connection-information"></a>Incorporación de la información de conexión del servicio

1. Haga clic con el botón derecho en el proyecto en el Explorador de soluciones y seleccione **Agregar** > **Nuevo elemento...** 

1. Asigne el nombre `appsettings.json` al archivo y seleccione **Agregar**. 

1. Incluya este archivo en el directorio de salida.
    1. Haga clic con el botón derecho en `appsettings.json` y seleccione **Propiedades**. 
    1. Cambie el valor de **Copiar en el directorio de salida** a **Copiar si es posterior**.

1. Copie el siguiente código JSON en el nuevo archivo JSON.

    ```json
    {
      "SearchServiceName": "Put your search service name here",
      "SearchServiceAdminApiKey": "Put your primary or secondary API key here",
      "SearchServiceQueryApiKey": "Put your query API key here",
      "AzureBlobConnectionString": "Put your Azure Blob connection string here",
    }
    ```
    
Agregue la información del servicio de búsqueda y de la cuenta de almacenamiento de blobs. Recuerde que puede obtener esta información en los pasos de aprovisionamiento del servicio indicados en la sección anterior.

En **SearchServiceName**, escriba el nombre corto del servicio y no la dirección URL completa.

### <a name="add-namespaces"></a>Agregar espacios de nombres

En `Program.cs`, agregue los siguientes espacios de nombres.

```csharp
using System;
using System.Collections.Generic;
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;
using Microsoft.Extensions.Configuration;

namespace EnrichwithAI
```

### <a name="create-a-client"></a>Creación de un cliente

Cree una instancia de la clase `SearchServiceClient` debajo de `Main`.

```csharp
public static void Main(string[] args)
{
    // Create service client
    IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
    IConfigurationRoot configuration = builder.Build();
    SearchServiceClient serviceClient = CreateSearchServiceClient(configuration);
```

`CreateSearchServiceClient` crea un nuevo `SearchServiceClient` mediante valores que se almacenan en el archivo de configuración de la aplicación (appsettings.json).

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
> La clase `SearchServiceClient` administra las conexiones con el servicio de búsqueda. Para evitar la apertura de un número excesivo de conexiones, debe intentar, si es posible, compartir una única instancia de `SearchServiceClient` en la aplicación. Sus métodos son seguros para subprocesos lo que permite habilitar este tipo de uso compartido.
> 

### <a name="add-function-to-exit-the-program-during-failure"></a>Adición de una función para salir del programa durante un error

Este tutorial está pensado para ayudarle a conocer los distintos pasos de la canalización de indexación. Si hay algún problema crítico que impida que el programa cree el origen de datos, conjunto de aptitudes, índice o indexador, el programa generará el mensaje de error y se cerrará, con el fin de que el problema se pueda entender y resolver.

Agregue `ExitProgram` a `Main` para controlar los escenarios que requieran que el programa se cierre.

```csharp
private static void ExitProgram(string message)
{
    Console.WriteLine("{0}", message);
    Console.WriteLine("Press any key to exit the program...");
    Console.ReadKey();
    Environment.Exit(0);
}
```

## <a name="3---create-the-pipeline"></a>3: Creación de la canalización

En Azure Cognitive Search, el procesamiento de la inteligencia artificial se produce durante la indexación (o la ingesta de datos). En esta parte del tutorial se crean cuatro objetos: origen de datos, definición de índice, conjunto de aptitudes, indexador. 

### <a name="step-1-create-a-data-source"></a>Paso 1: Creación de un origen de datos

`SearchServiceClient` tiene una propiedad `DataSources`. Esta propiedad proporciona todos los métodos que necesita para crear, enumerar, actualizar o eliminar orígenes de datos de Azure Cognitive Search.

Cree una nueva instancia de `DataSource` mediante una llamada a `serviceClient.DataSources.CreateOrUpdate(dataSource)`. `DataSource.AzureBlobStorage` requiere que especifique el nombre del origen de datos, la cadena de conexión y el nombre del contenedor de blobs.

```csharp
private static DataSource CreateOrUpdateDataSource(SearchServiceClient serviceClient, IConfigurationRoot configuration)
{
    DataSource dataSource = DataSource.AzureBlobStorage(
        name: "demodata",
        storageConnectionString: configuration["AzureBlobConnectionString"],
        containerName: "cog-search-demo",
        description: "Demo files to demonstrate cognitive search capabilities.");

    // The data source does not need to be deleted if it was already created
    // since we are using the CreateOrUpdate method
    try
    {
        serviceClient.DataSources.CreateOrUpdate(dataSource);
    }
    catch (Exception e)
    {
        Console.WriteLine("Failed to create or update the data source\n Exception message: {0}\n", e.Message);
        ExitProgram("Cannot continue without a data source");
    }

    return dataSource;
}
```

Para una solicitud correcta, el método devolverá el origen de datos que se creó. Si hay un problema con la solicitud, como un parámetro no válido, el método producirá una excepción.

Ahora, agregue una línea a `Main` para llamar a la función `CreateOrUpdateDataSource` que acaba de agregar.

```csharp
public static void Main(string[] args)
{
    // Create service client
    IConfigurationBuilder builder = new ConfigurationBuilder().AddJsonFile("appsettings.json");
    IConfigurationRoot configuration = builder.Build();
    SearchServiceClient serviceClient = CreateSearchServiceClient(configuration);

    // Create or Update the data source
    Console.WriteLine("Creating or updating the data source...");
    DataSource dataSource = CreateOrUpdateDataSource(serviceClient, configuration);
```

Compile y ejecute la solución. Puesto que se trata de su primera solicitud, consulte Azure Portal para confirmar el origen de datos se creó en Azure Cognitive Search. En la página del panel del servicio de búsqueda, compruebe que el icono de Data Sources tiene un nuevo elemento. Debe esperar unos minutos a que la página del portal se actualice.

  ![Icono de orígenes de datos en el portal](./media/cognitive-search-tutorial-blob/data-source-tile.png "Icono de orígenes de datos en el portal")

### <a name="step-2-create-a-skillset"></a>Paso 2: Creación de un conjunto de aptitudes

En esta sección, definirá un conjunto de pasos de enriquecimiento que quiere aplicar en los datos. Cada paso de enriquecimiento se denomina una *aptitud*, y el conjunto de pasos de enriquecimiento, un *conjunto de aptitudes*. Este tutorial usa un conjunto de [aptitudes cognitivas integradas](cognitive-search-predefined-skills.md):

+ [Reconocimiento óptico de caracteres](cognitive-search-skill-ocr.md) para reconocer texto escrito a mano e impreso en los archivos de imagen.

+ [Combinación de texto](cognitive-search-skill-textmerger.md) para consolidar el texto de una colección de campos en un solo campo.

+ [Detección de idioma](cognitive-search-skill-language-detection.md) para identificar el idioma del contenido.

+ [División de texto](cognitive-search-skill-textsplit.md) para dividir contenido grande en fragmentos más pequeños antes de llamar a la aptitud de extracción de frases clave y a la aptitud de reconocimiento de entidades. La extracción de frases clave y el reconocimiento de entidades aceptan entradas de 50 000 caracteres, o menos. Algunos de los archivos de ejemplo deben dividirse para no superar este límite.

+ [Reconocimiento de entidades](cognitive-search-skill-entity-recognition.md) para extraer los nombres de las organizaciones del contenido del contenedor de blobs.

+ [Extracción de frases clave](cognitive-search-skill-keyphrases.md) para extraer las frases clave principales.

Durante el procesamiento inicial, Azure Cognitive Search descifra cada documento para leer el contenido de distintos formatos de archivo. El texto encontrado procedente del archivo de origen se coloca en un campo ```content``` generado, uno para cada documento. Por lo tanto, establezca la entrada como ```"/document/content"``` para usar este texto. 

Las salidas se pueden asignar a un índice, usar como entrada para una aptitud descendente, o ambas cosas como sucede con el código de idioma. En el índice, un código de idioma es útil para el filtrado. Como entrada, el código de idioma se usa en las aptitudes de análisis de texto para informar a las reglas lingüísticas de la separación de palabras.

Para obtener más información sobre los conceptos básicos del conjunto de aptitudes, consulte el tema sobre la [definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md).

### <a name="ocr-skill"></a>Habilidad de OCR

La aptitud **OCR** extrae el texto de las imágenes. Esta aptitud supone que existe un campo normalized_images. Para generar este campo, más adelante en el tutorial también estableceremos la configuración de ```"imageAction"``` en la definición del indexador en ```"generateNormalizedImages"```.

```csharp
private static OcrSkill CreateOcrSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>
    {
        new InputFieldMappingEntry(
        name: "image",
        source: "/document/normalized_images/*")
    };

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>
    {
        new OutputFieldMappingEntry(
        name: "text",
        targetName: "text")
    };

    OcrSkill ocrSkill = new OcrSkill(
        description: "Extract text (plain and structured) from image",
        context: "/document/normalized_images/*",
        inputs: inputMappings,
        outputs: outputMappings,
        defaultLanguageCode: OcrSkillLanguage.En,
        shouldDetectOrientation: true);

    return ocrSkill;
}
```

### <a name="merge-skill"></a>Aptitud Combinar

En esta sección, creará una aptitud **Combinar** que combina el campo de contenido del documento con el texto que se ha producido con la aptitud de OCR.

```csharp
private static MergeSkill CreateMergeSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>
    {
        new InputFieldMappingEntry(
        name: "text",
        source: "/document/content"),
        new InputFieldMappingEntry(
        name: "itemsToInsert",
        source: "/document/normalized_images/*/text"),
        new InputFieldMappingEntry(
        name: "offsets",
        source: "/document/normalized_images/*/contentOffset")
    };

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>
    {
        new OutputFieldMappingEntry(
        name: "mergedText",
        targetName: "merged_text")
    };

    MergeSkill mergeSkill = new MergeSkill(
        description: "Create merged_text which includes all the textual representation of each image inserted at the right location in the content field.",
        context: "/document",
        inputs: inputMappings,
        outputs: outputMappings,
        insertPreTag: " ",
        insertPostTag: " ");

    return mergeSkill;
}
```

### <a name="language-detection-skill"></a>Aptitud Detección de idioma

La aptitud **Detección de idioma** detecta el idioma del texto de entrada e informa de un único código de idioma para cada documento enviado en la solicitud. Vamos a usar la salida de la aptitud **Detección de idioma** como parte de la entrada de la aptitud **División de texto**.

```csharp
private static LanguageDetectionSkill CreateLanguageDetectionSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>
    {
        new InputFieldMappingEntry(
        name: "text",
        source: "/document/merged_text")
    };

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>
    {
        new OutputFieldMappingEntry(
        name: "languageCode",
        targetName: "languageCode")
    };

    LanguageDetectionSkill languageDetectionSkill = new LanguageDetectionSkill(
        description: "Detect the language used in the document",
        context: "/document",
        inputs: inputMappings,
        outputs: outputMappings);

    return languageDetectionSkill;
}
```

### <a name="text-split-skill"></a>Aptitud División de texto

La aptitud **División** siguiente divide el texto por páginas y limita la longitud de la página a 4000 caracteres, según lo medido por `String.Length`. El algoritmo intentará dividir el texto en fragmentos que tengan un tamaño de `maximumPageLength` como máximo. En este caso, el algoritmo hará todo lo posible por dividir la frase en un límite de oración, por lo que el tamaño del fragmento puede ser ligeramente inferior a `maximumPageLength`.

```csharp
private static SplitSkill CreateSplitSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>
    {
        new InputFieldMappingEntry(
        name: "text",
        source: "/document/merged_text"),
        new InputFieldMappingEntry(
        name: "languageCode",
        source: "/document/languageCode")
    };

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>
    {
        new OutputFieldMappingEntry(
        name: "textItems",
        targetName: "pages")
    };

    SplitSkill splitSkill = new SplitSkill(
        description: "Split content into pages",
        context: "/document",
        inputs: inputMappings,
        outputs: outputMappings,
        textSplitMode: TextSplitMode.Pages,
        maximumPageLength: 4000);

    return splitSkill;
}
```

### <a name="entity-recognition-skill"></a>Aptitud Reconocimiento de entidades

Esta instancia `EntityRecognitionSkill` está configurada para reconocer el tipo de categoría `organization`. La aptitud **Reconocimiento de entidades** también puede reconocer los tipos de categoría `person` y `location`.

Tenga en cuenta que el campo "context" se establece en ```"/document/pages/*"``` con un asterisco, lo que significa que se llama al paso de enriquecimiento para cada página en ```"/document/pages"```.

```csharp
private static EntityRecognitionSkill CreateEntityRecognitionSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>
    {
        new InputFieldMappingEntry(
        name: "text",
        source: "/document/pages/*")
    };

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>
    {
        new OutputFieldMappingEntry(
        name: "organizations",
        targetName: "organizations")
    };

    List<EntityCategory> entityCategory = new List<EntityCategory>
    {
        EntityCategory.Organization
    };

    EntityRecognitionSkill entityRecognitionSkill = new EntityRecognitionSkill(
        description: "Recognize organizations",
        context: "/document/pages/*",
        inputs: inputMappings,
        outputs: outputMappings,
        categories: entityCategory,
        defaultLanguageCode: EntityRecognitionSkillLanguage.En);

    return entityRecognitionSkill;
}
```

### <a name="key-phrase-extraction-skill"></a>Aptitud Extracción de frases clave

Al igual que la instancia `EntityRecognitionSkill` recién creada, se llama a la aptitud **Extracción de frases clave** para cada página del documento.

```csharp
private static KeyPhraseExtractionSkill CreateKeyPhraseExtractionSkill()
{
    List<InputFieldMappingEntry> inputMappings = new List<InputFieldMappingEntry>();
    inputMappings.Add(new InputFieldMappingEntry(
        name: "text",
        source: "/document/pages/*"));
    inputMappings.Add(new InputFieldMappingEntry(
        name: "languageCode",
        source: "/document/languageCode"));

    List<OutputFieldMappingEntry> outputMappings = new List<OutputFieldMappingEntry>();
    outputMappings.Add(new OutputFieldMappingEntry(
        name: "keyPhrases",
        targetName: "keyPhrases"));

    KeyPhraseExtractionSkill keyPhraseExtractionSkill = new KeyPhraseExtractionSkill(
        description: "Extract the key phrases",
        context: "/document/pages/*",
        inputs: inputMappings,
        outputs: outputMappings);

    return keyPhraseExtractionSkill;
}
```

### <a name="build-and-create-the-skillset"></a>Compilar y crear el conjunto de aptitudes

Compile `Skillset` con las aptitudes que ha creado.

```csharp
private static Skillset CreateOrUpdateDemoSkillSet(SearchServiceClient serviceClient, IList<Skill> skills)
{
    Skillset skillset = new Skillset(
        name: "demoskillset",
        description: "Demo skillset",
        skills: skills);

    // Create the skillset in your search service.
    // The skillset does not need to be deleted if it was already created
    // since we are using the CreateOrUpdate method
    try
    {
        serviceClient.Skillsets.CreateOrUpdate(skillset);
    }
    catch (Exception e)
    {
        Console.WriteLine("Failed to create the skillset\n Exception message: {0}\n", e.Message);
        ExitProgram("Cannot continue without a skillset");
    }

    return skillset;
}
```

Agregue las siguientes líneas a `Main`.

```csharp
    // Create the skills
    Console.WriteLine("Creating the skills...");
    OcrSkill ocrSkill = CreateOcrSkill();
    MergeSkill mergeSkill = CreateMergeSkill();
    EntityRecognitionSkill entityRecognitionSkill = CreateEntityRecognitionSkill();
    LanguageDetectionSkill languageDetectionSkill = CreateLanguageDetectionSkill();
    SplitSkill splitSkill = CreateSplitSkill();
    KeyPhraseExtractionSkill keyPhraseExtractionSkill = CreateKeyPhraseExtractionSkill();

    // Create the skillset
    Console.WriteLine("Creating or updating the skillset...");
    List<Skill> skills = new List<Skill>
    {
        ocrSkill,
        mergeSkill,
        languageDetectionSkill,
        splitSkill,
        entityRecognitionSkill,
        keyPhraseExtractionSkill
    };

    Skillset skillset = CreateOrUpdateDemoSkillSet(serviceClient, skills);
```

### <a name="step-3-create-an-index"></a>Paso 3: Creación de un índice

En esta sección, se define el esquema de índice. Para ello, se especifican los campos que se incluirán en el índice que permite búsquedas y los atributos de búsqueda de cada campo. Los campos tienen un tipo y pueden tener atributos que determinen cómo se utiliza el campo (si permite búsquedas, se puede ordenar, etc.). Los nombres de campos de un índice no tienen que coincidir exactamente con los nombres de campo del origen. En un paso posterior, agregará asignaciones de campos en un indexador para conectar los campos de origen y destino. Para este paso, defina el índice con convenciones de nomenclatura de campos adecuadas para la aplicación de búsqueda.

En este ejercicio se utilizan los siguientes campos y tipos de campos:

| Nombres de campo | Tipos de campo |
| --- | --- |
| id | Edm.String |
| content | Edm.String |
| languageCode | Edm.String |
| keyPhrases | List<Edm.String> |
| organizations | List<Edm.String> |

#### <a name="create-demoindex-class"></a>Crear la clase DemoIndex

Los campos de este índice se definen mediante una clase de modelo. Cada propiedad de la clase de modelo tiene atributos que determinan los comportamientos relacionados con la búsqueda del campo de índice correspondiente. 

Vamos a agregar la clase del modelo a un nuevo archivo de C#. Haga clic con el botón derecho en el proyecto y seleccione **Agregar** > **Nuevo elemento...** , seleccione "Class" y el nombre del archivo `DemoIndex.cs`. Luego, seleccione **Agregar**.

Asegúrese de indicar que quiere usar tipos de los espacios de nombres `Microsoft.Azure.Search` y `Microsoft.Azure.Search.Models`.

Agregue la siguiente definición de la clase de modelo en `DemoIndex.cs` e inclúyala en el mismo espacio de nombres donde creará el índice.

```csharp
using Microsoft.Azure.Search;
using Microsoft.Azure.Search.Models;

namespace EnrichwithAI
{
    // The SerializePropertyNamesAsCamelCase attribute is defined in the Azure Search .NET SDK.
    // It ensures that Pascal-case property names in the model class are mapped to camel-case
    // field names in the index.
    [SerializePropertyNamesAsCamelCase]
    public class DemoIndex
    {
        [System.ComponentModel.DataAnnotations.Key]
        [IsSearchable, IsSortable]
        public string Id { get; set; }

        [IsSearchable]
        public string Content { get; set; }

        [IsSearchable]
        public string LanguageCode { get; set; }

        [IsSearchable]
        public string[] KeyPhrases { get; set; }

        [IsSearchable]
        public string[] Organizations { get; set; }
    }
}
```

Ahora que ha definido una clase de modelo, de nuevo en `Program.cs`, puede crear una definición de índice con bastante facilidad. El nombre de este índice será `demoindex`. Si ya existe un índice con ese nombre, se eliminará.

```csharp
private static Index CreateDemoIndex(SearchServiceClient serviceClient)
{
    var index = new Index()
    {
        Name = "demoindex",
        Fields = FieldBuilder.BuildForType<DemoIndex>()
    };

    try
    {
        bool exists = serviceClient.Indexes.Exists(index.Name);

        if (exists)
        {
            serviceClient.Indexes.Delete(index.Name);
        }

        serviceClient.Indexes.Create(index);
    }
    catch (Exception e)
    {
        Console.WriteLine("Failed to create the index\n Exception message: {0}\n", e.Message);
        ExitProgram("Cannot continue without an index");
    }

    return index;
}
```

Durante las pruebas, es posible que intente crear el índice de más de una vez. Por este motivo, compruebe si el índice que está por crear ya existe antes de intentar crearlo.

Agregue las siguientes líneas a `Main`.

```csharp
    // Create the index
    Console.WriteLine("Creating the index...");
    Microsoft.Azure.Search.Models.Index demoIndex = CreateDemoIndex(serviceClient);
```

Agregue la instrucción using para resolver la referencia relativa a la eliminación de ambigüedad.

```csharp
using Index = Microsoft.Azure.Search.Models.Index;
```

Para más información acerca de cómo definir un índice, consulte [Creación de un índice (API REST de Azure Cognitive Search)](/rest/api/searchservice/create-index).

### <a name="step-4-create-and-run-an-indexer"></a>Paso 4: Creación y ejecución de un indexador

Hasta ahora, ha creado un origen de datos, un conjunto de aptitudes y un índice. Estos tres componentes pasan a formar parte de un [indexador](search-indexer-overview.md) que extrae todas las piezas juntas en una sola operación de varias fases. Para unirlas en un indexador, debe definir las asignaciones de campos.

+ Los campos fieldMappings se procesan antes que el conjunto de aptitudes, lo que asigna los campos de origen del origen de datos a los campos de destino de un índice. Si los tipos y nombres de campo son los mismos en ambos extremos, no se requiere ninguna asignación.

+ Los campos outputFieldMappings se procesan después que el conjunto de aptitudes y hacen referencia a las asignaciones sourceFieldNames que no existen hasta que el descifrado o enriquecimiento de documentos las crean. targetFieldName es un campo de un índice.

Además de enlazar las entradas con las salidas, también puede usar las asignaciones de campos para aplanar las estructuras de datos. Para más información, consulte [Asignación de campos enriquecidos a un índice de búsqueda](cognitive-search-output-field-mapping.md).

```csharp
private static Indexer CreateDemoIndexer(SearchServiceClient serviceClient, DataSource dataSource, Skillset skillSet, Index index)
{
    IDictionary<string, object> config = new Dictionary<string, object>();
    config.Add(
        key: "dataToExtract",
        value: "contentAndMetadata");
    config.Add(
        key: "imageAction",
        value: "generateNormalizedImages");

    List<FieldMapping> fieldMappings = new List<FieldMapping>
    {
        new FieldMapping(
        sourceFieldName: "metadata_storage_path",
        targetFieldName: "id",
        mappingFunction: new FieldMappingFunction(
            name: "base64Encode")),
        new FieldMapping(
        sourceFieldName: "content",
        targetFieldName: "content")
    };

    List<FieldMapping> outputMappings = new List<FieldMapping>
    {
        new FieldMapping(
        sourceFieldName: "/document/pages/*/organizations/*",
        targetFieldName: "organizations"),
        new FieldMapping(
        sourceFieldName: "/document/pages/*/keyPhrases/*",
        targetFieldName: "keyPhrases"),
        new FieldMapping(
        sourceFieldName: "/document/languageCode",
        targetFieldName: "languageCode")
    };

    Indexer indexer = new Indexer(
        name: "demoindexer",
        dataSourceName: dataSource.Name,
        targetIndexName: index.Name,
        description: "Demo Indexer",
        skillsetName: skillSet.Name,
        parameters: new IndexingParameters(
            maxFailedItems: -1,
            maxFailedItemsPerBatch: -1,
            configuration: config),
        fieldMappings: fieldMappings,
        outputFieldMappings: outputMappings);

    try
    {
        bool exists = serviceClient.Indexers.Exists(indexer.Name);

        if (exists)
        {
            serviceClient.Indexers.Delete(indexer.Name);
        }

        serviceClient.Indexers.Create(indexer);
    }
    catch (Exception e)
    {
        Console.WriteLine("Failed to create the indexer\n Exception message: {0}\n", e.Message);
        ExitProgram("Cannot continue without creating an indexer");
    }

    return indexer;
}
```
Agregue las siguientes líneas a `Main`.

```csharp
    // Create the indexer, map fields, and execute transformations
    Console.WriteLine("Creating the indexer and executing the pipeline...");
    Indexer demoIndexer = CreateDemoIndexer(serviceClient, dataSource, skillset, demoIndex);
```

Es de esperar que la creación del indexador tarde algo de tiempo en completarse. Aunque el conjunto de datos es pequeño, las aptitudes analíticas realiza un uso intensivo de los recursos. Algunas aptitudes, como el análisis de imágenes, son de larga ejecución.

> [!TIP]
> La creación de un indexador invoca la canalización. Si hay problemas para conectar con los datos, las entradas y salidas de asignación o el orden de las operaciones, se muestran en esta fase.

### <a name="explore-creating-the-indexer"></a>Explorar la creación del indexador

El código establece ```"maxFailedItems"``` en -1, que indica al motor de indexación que omita los errores durante la importación de datos. Esto es útil porque hay muy pocos documentos en el origen de datos de demostración. Para un origen de datos mayor, debería establecer un valor mayor que 0.

Observe también que ```"dataToExtract"``` se estable en ```"contentAndMetadata"```. Esta instrucción indica al indexador que extraiga automáticamente el contenido de diferentes formatos de archivo, así como los metadatos relacionados con cada archivo.

Una vez extraído el contenido, puede establecer `imageAction` para que se extraiga el texto de las imágenes que se encuentran en el origen de datos. La configuración de ```"imageAction"``` establecido en ```"generateNormalizedImages"```, combinada con la aptitud de OCR y la aptitud de Combinación de texto, indica al indexador que extraiga el texto de las imágenes (por ejemplo, la palabra "stop" de la señal de tráfico Stop) y lo inserte como parte del campo de contenido. Este comportamiento se aplica a las imágenes incrustadas en los documentos (piense en una imagen de un archivo PDF), así como a las imágenes que se encuentran en el origen de datos, como un archivo JPG.

<a name="check-indexer-status"></a>

## <a name="4---monitor-indexing"></a>4: Supervisión de la indexación

Una vez definido el indexador, se ejecuta automáticamente cuando se envía la solicitud. Dependiendo de las aptitudes cognitivas definidas, la indexación puede tardar más de lo esperado. Para averiguar si el indexador todavía se está ejecutando, use el método `GetStatus`.

```csharp
private static void CheckIndexerOverallStatus(SearchServiceClient serviceClient, Indexer indexer)
{
    try
    {
        IndexerExecutionInfo demoIndexerExecutionInfo = serviceClient.Indexers.GetStatus(indexer.Name);

        switch (demoIndexerExecutionInfo.Status)
        {
            case IndexerStatus.Error:
                ExitProgram("Indexer has error status. Check the Azure Portal to further understand the error.");
                break;
            case IndexerStatus.Running:
                Console.WriteLine("Indexer is running");
                break;
            case IndexerStatus.Unknown:
                Console.WriteLine("Indexer status is unknown");
                break;
            default:
                Console.WriteLine("No indexer information");
                break;
        }
    }
    catch (Exception e)
    {
        Console.WriteLine("Failed to get indexer overall status\n Exception message: {0}\n", e.Message);
    }
}
```

`IndexerExecutionInfo` representa el estado actual y el historial de ejecución de un indexador.

Las advertencias son comunes con algunas combinaciones de aptitudes y archivos de origen y no siempre indican un problema. En este tutorial, las advertencias son benignas (por ejemplo, no hay entradas de texto de archivos JPEG).

Agregue las siguientes líneas a `Main`.

```csharp
    // Check indexer overall status
    Console.WriteLine("Check the indexer overall status...");
    CheckIndexerOverallStatus(serviceClient, demoIndexer);
```
 
## <a name="5---search"></a>5: Búsqueda

Una vez finalizada la indexación, puede ejecutar consultas que devuelvan el contenido de los campos individuales. De forma predeterminada, Azure Cognitive Search devuelve los primeros 50 resultados. Los datos de ejemplo tiene un tamaño reducido, por lo que la configuración predeterminada funciona bien. Sin embargo, si trabaja con conjuntos de datos más grandes, deberá incluir parámetros en la cadena de consulta para que se devuelvan más resultados. Para ver las instrucciones, consulte [Cómo paginar los resultados de la búsqueda en Azure Cognitive Search](search-pagination-page-layout.md).

Como paso de verificación, consulte el índice para todos los campos.

Agregue las siguientes líneas a `Main`.

```csharp
DocumentSearchResult<DemoIndex> results;

ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(configuration);

SearchParameters parameters =
    new SearchParameters
    {
        Select = new[] { "organizations" }
    };

try
{
    results = indexClientForQueries.Documents.Search<DemoIndex>("*", parameters);
}
catch (Exception e)
{
    // Handle exception
}
```

`CreateSearchIndexClient` crea un nuevo `SearchIndexClient` mediante valores que se almacenan en el archivo de configuración de la aplicación (appsettings.json). Tenga en cuenta que se usa la clave de API de consulta del servicio de búsqueda, no la clave de administrador.

```csharp
private static SearchIndexClient CreateSearchIndexClient(IConfigurationRoot configuration)
{
   string searchServiceName = configuration["SearchServiceName"];
   string queryApiKey = configuration["SearchServiceQueryApiKey"];

   SearchIndexClient indexClient = new SearchIndexClient(searchServiceName, "demoindex", new SearchCredentials(queryApiKey));
   return indexClient;
}
```

Agregue el siguiente código a `Main`. La primera instrucción try-catch devuelve la definición del índice, con el nombre, el tipo y los atributos de cada campo. La segunda es una consulta con parámetros, donde `Select` especifica qué campos se van a incluir en los resultados, por ejemplo `organizations`. Una cadena de búsqueda de `"*"` devuelve todo el contenido de un solo campo.

```csharp
//Verify content is returned after indexing is finished
ISearchIndexClient indexClientForQueries = CreateSearchIndexClient(configuration);

try
{
    results = indexClientForQueries.Documents.Search<DemoIndex>("*");
    Console.WriteLine("First query succeeded with a result count of {0}", results.Results.Count);
}
catch (Exception e)
{
    Console.WriteLine("First query failed\n Exception message: {0}\n", e.Message);
}

SearchParameters parameters =
    new SearchParameters
    {
        Select = new[] { "organizations" }
    };

try
{
    results = indexClientForQueries.Documents.Search<DemoIndex>("*", parameters);
    Console.WriteLine("Second query succeeded with a result count of {0}", results.Results.Count);
}
catch (Exception e)
{
    Console.WriteLine("Second query failed\n Exception message: {0}\n", e.Message);
}
```

Repita el proceso para otros campos: contenido, languageCode, keyPhrases y organizaciones del ejercicio. Puede devolver varios campos a través de la propiedad [Select](/dotnet/api/microsoft.azure.search.models.searchparameters.select?view=azure-dotnet) con una lista delimitada por comas.

<a name="reset"></a>

## <a name="reset-and-rerun"></a>Restablecer y volver a ejecutar

En las primeras etapas experimentales de desarrollo, el enfoque más práctico para las iteraciones del diseño es eliminar los objetos de Azure Cognitive Search y permitir que el código vuelva a generarlos. Los nombres de los recursos son únicos. La eliminación de un objeto permite volver a crearlo con el mismo nombre.

En el código de ejemplo de este tutorial se comprueban los objetos existentes y se eliminan para que pueda volver a ejecutar el código.

También puede usar el portal para eliminar los índices, los indexadores y los conjuntos de aptitudes.

## <a name="takeaways"></a>Puntos clave

En este tutorial se han mostrado los pasos básicos para la creación de una canalización de indexación enriquecida a través de la creación de componentes: un origen de datos, un conjunto de aptitudes, un índice y un indexador.

Se presentaron las [aptitudes integradas](cognitive-search-predefined-skills.md), junto con la definición del conjunto de aptitudes y los mecanismos de encadenamiento de aptitudes, mediante entradas y salidas. También aprendió que `outputFieldMappings` en la definición del indexador es necesario para enrutar los valores enriquecidos de la canalización a un índice que permita búsquedas en un servicio Azure Cognitive Search.

Por último, ha aprendido cómo probar los resultados y restablecer el sistema para otras iteraciones. Ha aprendido que emitir consultas en el índice devuelve la salida creada por la canalización de indexación enriquecida. También ha aprendido cómo comprobar el estado del indexador y qué objetos se deben eliminar antes de volver a ejecutar una canalización.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, al final de un proyecto, es recomendable eliminar los recursos que ya no necesite. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede buscar y administrar los recursos en el portal, mediante el vínculo Todos los recursos o Grupos de recursos en el panel de navegación izquierdo.

## <a name="next-steps"></a>Pasos siguientes

Ahora que está familiarizado con todos los objetos de una canalización de enriquecimiento de AI, echemos un vistazo más de cerca a las definiciones de actitudes y a los conocimientos individuales.

> [!div class="nextstepaction"]
> [Creación de un conjunto de aptitudes](cognitive-search-defining-skillset.md)