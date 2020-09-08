---
title: Búsqueda a través de blobs JSON
titleSuffix: Azure Cognitive Search
description: Rastree el contenido de texto de los blobs JSON de Azure mediante el indexador de blobs de Azure Cognitive Search. Los indexadores automatizan la ingesta de datos para orígenes de datos seleccionados, como Azure Blob Storage.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/11/2020
ms.openlocfilehash: 6a5a3634f1d5ddc7af2af2e27c2b1d7c8ce9a2af
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88917999"
---
# <a name="how-to-index-json-blobs-using-a-blob-indexer-in-azure-cognitive-search"></a>Indexación de blobs JSON con el indexador de blobs de Azure Cognitive Search

En este artículo se muestra cómo configurar un [indexador](search-indexer-overview.md) de blobs de Azure Cognitive Search para extraer contenido estructurado de los documentos JSON en Azure Blob Storage y permitir que se hagan búsquedas en él con Azure Cognitive Search. Este flujo de trabajo crea un índice de Azure Cognitive Search y lo carga con texto extraído de los blobs JSON. 

Puede usar el [portal](#json-indexer-portal), las [API REST](#json-indexer-rest) o el [SDK de .NET](#json-indexer-dotnet) para indexar el contenido JSON. En todas las opciones anteriores, los documentos JSON se encuentran en un contenedor de blobs en una cuenta de Azure Storage. Para instrucciones sobre la inserción de documentos JSON desde otras plataformas que no sean de Azure, consulte [Importación de datos en Azure Cognitive Search](search-what-is-data-import.md).

Los blobs JSON de Azure Blob Storage suelen ser un documento JSON único (el modo de análisis es `json`) o una colección de entidades JSON. Para las colecciones, el blob podría tener una **matriz** de elementos JSON bien formados (el modo de análisis es `jsonArray`). Los blobs también podrían estar compuestos de varias entidades JSON individuales, separadas por una línea nueva (el modo de análisis es `jsonLines`). El parámetro **parsingMode** de la solicitud determina las estructuras de salida.

> [!NOTE]
> Para más información sobre la indexación de varios documentos de búsqueda desde un solo blob, consulte [Indexación de uno a varios](search-howto-index-one-to-many-blobs.md).

<a name="json-indexer-portal"></a>

## <a name="use-the-portal"></a>Uso del portal

El método más sencillo para la indexación de documentos JSON es usar un asistente en [Azure Portal](https://portal.azure.com/). Mediante el análisis de metadatos en el contenedor de blobs de Azure, el asistente [**Importar datos**](search-import-data-portal.md) puede crear un índice predeterminado, asignar campos de origen a campos de índice de destino y cargar el índice en una sola operación. Según el tamaño y la complejidad del origen de datos, puede tener un índice de búsqueda de texto completo y operativo en cuestión de minutos.

Se recomienda usar la misma región o ubicación para Azure Cognitive Search y Azure Storage a fin de conseguir una menor latencia y evitar cargos por ancho de banda.

### <a name="1---prepare-source-data"></a>1: Preparación de los datos de origen

[Inicie sesión en Azure Portal](https://portal.azure.com/) y [cree un contenedor de blobs](../storage/blobs/storage-quickstart-blobs-portal.md) que contenga los datos. El nivel de acceso público se puede establecer en cualquiera de sus valores válidos.

Necesitará el nombre de la cuenta de almacenamiento, el nombre del contenedor y una clave de acceso para recuperar los datos en el asistente para la **importación de datos**.

### <a name="2---start-import-data-wizard"></a>2: Inicio del asistente para la importación de datos

En la página de información general del servicio de búsqueda, puede [iniciar el asistente](search-import-data-portal.md) desde la barra de comandos.

   ![Comando de importación de datos en el portal](./media/search-import-data-portal/import-data-cmd2.png "Inicio del Asistente para la importación de datos")

### <a name="3---set-the-data-source"></a>3: Configuración del origen de datos

En la página **origen de datos**, el origen debe ser **Azure Blob Storage** con las especificaciones siguientes:

+ **Datos que se extraerán** debe seleccionar el valor *Contenido y metadatos*. Si elige esta opción, permitirá que el asistente deduzca un esquema de índice y asigne los campos para la importación.
   
+ El **Modo de análisis** debe establecerse en *JSON*, *Matriz JSON* o *Líneas JSON*. 

  La opción *JSON* articula cada blob como un documento único de búsqueda, por lo que aparecen como un elemento independiente en los resultados de búsqueda. 

  *Matriz JSON* es para los blobs que contienen datos JSON bien formados; el JSON bien formado corresponde a una matriz de objetos o tiene una propiedad que es una matriz de objetos y le interesa que cada elemento se articule como un documento de búsqueda independiente. Si los blobs son complejos y no elige la opción *matriz JSON*, todo el blob se ingiere como un único documento.

  *Líneas JSON* es para blobs compuestos por varias entidades JSON separadas por una nueva línea, donde le interesa que cada elemento se articule como un documento de búsqueda independiente. Si los blobs son complejos y no elige el modo de análisis *Líneas JSON*, todo el blob se ingiere como un único documento.
   
+ El **Contenedor de almacenamiento** debe especificar su cuenta de almacenamiento y contenedor o una cadena de conexión que se resuelva en el contenedor. Puede obtener las cadenas de conexión en la página del portal de Blob service.

   ![Definición del origen de datos del blob](media/search-howto-index-json/import-wizard-json-data-source.png)

### <a name="4---skip-the-enrich-content-page-in-the-wizard"></a>4: omitir la página"Enriquecer contenido" del asistente

Agregar habilidades cognitivas (o enriquecimiento) no es un requisito de importación. A menos que tenga una necesidad específica de [agregar enriquecimiento de inteligencia artificial](cognitive-search-concept-intro.md) a su canalización de indexación, debe omitir este paso.

Para omitir el paso, haga clic en los botones azules en la parte inferior de la página para "Siguiente" y "Omitir".

### <a name="5---set-index-attributes"></a>5: Configuración de los atributos de índice

En la página de **Índice**, debería mostrarse una lista de campos con un tipo de datos y una serie de casillas de verificación para configurar los atributos del índice. El asistente puede generar una lista de campos basada en metadatos y mediante el muestreo de los datos de origen. 

Puede seleccionar atributos de forma masiva si activa la casilla situada en la parte superior de una columna de atributos. Elija **Se puede recuperar** y **Se puede buscar** para cada campo que se debe devolver a una aplicación cliente y está sujeto a un proceso de búsqueda de texto completo. Observará que los enteros no permiten búsquedas de texto completo o de texto parcial (los números se evalúan literalmente y suelen ser útiles en filtros).

Revise la descripción de [atributos de índice](/rest/api/searchservice/create-index#bkmk_indexAttrib) y [analizadores de lenguaje](/rest/api/searchservice/language-support) para más información. 

Dedique un momento a la revisión de las selecciones. Una vez que se ejecuta al asistente, se crean las estructuras de datos físicas y no podrá modificar estos campos sin quitar y volver a crear todos los objetos.

   ![Definición de índice de blob](media/search-howto-index-json/import-wizard-json-index.png)

### <a name="6---create-indexer"></a>6: Creación del indizador

Con todas las especificaciones agregadas, el asistente crea tres objetos distintos en el servicio de búsqueda. Un objeto de origen de datos y un objeto de índice se guardan como recursos con nombre en el servicio Azure Cognitive Search. El último paso crea un objeto de indizador. Al asignarle un nombre al indizador, este puede existir como un recurso independiente que se puede programar y administrar independientemente de los objetos de origen de datos y del de índice que se crearon en la misma secuencia del asistente.

Si no está familiarizado con los indexadores, un *indexador* es un recurso en Azure Cognitive Search que rastrea un origen de datos externo en busca de contenido utilizable en búsquedas. La salida del asistente **Importar datos**  es un indexador que rastrea el origen de datos JSON, extrae el contenido utilizable en búsquedas y lo importa a un índice en Azure Cognitive Search.

   ![Definición del indizador del blob](media/search-howto-index-json/import-wizard-json-indexer.png)

Haga clic en **Aceptar** para ejecutar el asistente y crear todos los objetos. La indexación comienza inmediatamente.

Puede supervisar la importación de datos en las páginas del portal. Las notificaciones de progreso indican el estado de la indexación y cuántos documentos se cargan. 

Cuando se complete la indización, puede usar el [Explorador de búsqueda](search-explorer.md) para consultar el índice.

> [!NOTE]
> Si no se muestran los datos esperados, tiene que establecer más atributos en más campos. Elimine el índice y el indexador que acaba de crear y vuelva a seguir los pasos del asistente, pero esta vez cambie las opciones elegidas para los atributos de índice del paso 5. 

<a name="json-indexer-rest"></a>

## <a name="use-rest-apis"></a>Uso de API REST

Puede usar la API REST para indexar blobs JSON, siguiendo un flujo de trabajo de tres partes común a todos los indexadores en Azure Cognitive Search: se crea un origen de datos, se crea un índice y se crea un indexador. La extracción de datos de Blob Storage se produce cuando se envía la solicitud Crear indizador. Una vez finalizada esta solicitud, tendrá un índice que permite consultas. 

Eche un vistazo al [código de ejemplo de REST](#rest-example) al final de esta sección para ver cómo crear los tres objetos. Esta sección también contiene detalles sobre [modos de análisis de JSON](#parsing-modes), [blobs únicos](#parsing-single-blobs), [matrices JSON](#parsing-arrays) y [matrices anidadas](#nested-json-arrays).

Para la indexación de JSON basado en código, use [Postman](search-get-started-postman.md) y la API REST para crear estos objetos:

+ [índice](/rest/api/searchservice/create-index)
+ [origen de datos](/rest/api/searchservice/create-data-source)
+ [indexador](/rest/api/searchservice/create-indexer)

El orden de las operaciones requiere que los objetos se creen y se llamen en este orden. A diferencia del flujo de trabajo del portal, un enfoque de código necesita un índice disponible para aceptar los documentos JSON que se envían a través de la solicitud **Crear indizador**.

Los blobs JSON de Azure Blob Storage suelen ser un documento JSON único o una "matriz" JSON. El indexador de blobs de Azure Cognitive Search puede analizar cualquier construcción, según cómo configure el parámetro **parsingMode** en la solicitud.

| Documento JSON | parsingMode | Descripción | Disponibilidad |
|--------------|-------------|--------------|--------------|
| Uno por blob | `json` | Analiza los blobs JSON como un único fragmento de texto. Cada blob JSON se convierte en un único documento de Azure Cognitive Search. | Por lo general, está disponible en la API [REST](/rest/api/searchservice/indexer-operations) y el SDK de [.NET](/dotnet/api/microsoft.azure.search.models.indexer). |
| Varios por blob | `jsonArray` | Analiza una matriz JSON en el blob, donde cada elemento de la matriz se convierte en un documento de Azure Cognitive Search independiente.  | Por lo general, está disponible en la API [REST](/rest/api/searchservice/indexer-operations) y el SDK de [.NET](/dotnet/api/microsoft.azure.search.models.indexer). |
| Varios por blob | `jsonLines` | Analiza un blob que contiene varias entidades JSON (una "matriz"), separadas por una nueva línea, donde cada entidad se convierte en un documento de Azure Cognitive Search independiente. | Por lo general, está disponible en la API [REST](/rest/api/searchservice/indexer-operations) y el SDK de [.NET](/dotnet/api/microsoft.azure.search.models.indexer). |

### <a name="1---assemble-inputs-for-the-request"></a>1\. Ensamblado de las entradas para la solicitud

Para cada solicitud, debe proporcionar el nombre del servicio y la clave de administrador de Azure Cognitive Search (en el encabezado POST) y el nombre de cuenta de almacenamiento y la clave para Blob Storage. Puede usar [Postman](search-get-started-postman.md) para enviar solicitudes HTTP a Azure Cognitive Search.

Copie los cuatro valores siguientes en el Bloc de notas para poder pegarlos en una solicitud:

+ El nombre del servicio Azure Cognitive Search
+ La clave de administrador de Azure Cognitive Search
+ Nombre de la cuenta de almacenamiento de Azure
+ Clave de la cuenta de Azure Storage

Encontrará estos valores en el portal:

1. En las páginas del portal de Azure Cognitive Search, copie la URL del servicio de búsqueda desde la página de información general.

2. En el panel de navegación izquierdo, haga clic en **Claves** y, luego, copie la clave primaria o la secundaria (son equivalentes).

3. Cambie a las páginas del portal para la cuenta de almacenamiento. En el panel de navegación de la izquierda, en **Configuración**, haga clic en **Claves de acceso**. Esta página proporciona el nombre de cuenta y la clave. Copie el nombre de cuenta de almacenamiento y una de las claves en el Bloc de notas.

### <a name="2---create-a-data-source"></a>2\. Crear un origen de datos

El primer paso es proporcionar la información de conexión de origen de datos usada por el indexador. El origen de datos es un objeto con nombre en Azure Cognitive Search que conserva la información de conexión. El tipo de origen de datos, `azureblob`, determina qué comportamientos de extracción de datos se invocan mediante el indexador. 

Sustituya los valores válidos para los marcadores de posición de nombre del servicio, clave de administración, cuenta de almacenamiento y clave de la cuenta.

```http
    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key for Azure Cognitive Search]

    {
        "name" : "my-blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-container", "query" : "optional, my-folder" }
    }   
```

### <a name="3---create-a-target-search-index"></a>3\. Crear un índice de búsqueda de destino 

Los indexadores se emparejan con un esquema de índice. Si utiliza la API (en lugar del portal), prepare un índice de antemano para poder especificarlo en la operación del indexador.

El índice almacena el contenido utilizable en búsquedas en Azure Cognitive Search. Para crear un índice, proporcione un esquema que especifique los campos de un documento, los atributos y otras construcciones que conforman la experiencia de búsqueda. Si crea un índice que tenga los mismos nombres de campo y tipos de datos que el origen, el indizador hará coincidir los campos de origen y de destino, por lo que se ahorrará el trabajo asignar explícitamente los campos.

El ejemplo siguiente muestra una solicitud de [Creación de índice](/rest/api/searchservice/create-index). El índice tendrá un campo `content` utilizable en búsquedas para almacenar el texto extraído de los blobs:   

```http
    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key for Azure Cognitive Search]

    {
          "name" : "my-target-index",
          "fields": [
            { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
            { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
          ]
    }
```


### <a name="4---configure-and-run-the-indexer"></a>4\. Configurar y ejecutar el indexador

Al igual que ocurre con los índices y los orígenes de datos, los indexadores son objetos con nombre que se pueden crear y volver a usar en un servicio Azure Cognitive Search. Una solicitud totalmente especificada para crear un indexador se vería similar a la siguiente:

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key for Azure Cognitive Search]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "json" } }
    }
```

La configuración del indexador está en el cuerpo de la solicitud. Necesita un origen de datos y un índice de destino vacío que ya existe en Azure Cognitive Search. 

La programación y los parámetros son opcionales. Así que si los omite, el indexador se ejecuta de inmediato y usa `json` como modo de análisis.

Este indexador en concreto no incluye asignaciones de campos. En la definición del indexador, puede no incluir las **asignaciones de campo** si las propiedades del documento JSON de origen coinciden con los campos del índice de búsqueda de destino. 


### <a name="rest-example"></a>Ejemplo de REST

Esta sección es un resumen de todas las solicitudes que se usan para crear objetos. Para obtener una explicación de las partes componentes, vea las secciones anteriores de este artículo.

### <a name="data-source-request"></a>Solicitud de origen de datos

Todos los indexadores necesitan un objeto de origen de datos que proporcione información de conexión a los datos existentes. 

```http
    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key for Azure Cognitive Search]

    {
        "name" : "my-blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-container", "query" : "optional, my-folder" }
    }  
```

### <a name="index-request"></a>Solicitud de índice

Todos los indizadores necesitan un índice de destino que reciba los datos. El cuerpo de la solicitud define el esquema de índice, que consta de campos con atributos para admitir el comportamiento que quiera en un índice de búsqueda. Este índice debe estar vacío cuando ejecute el indexador. 

```http
    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key for Azure Cognitive Search]

    {
          "name" : "my-target-index",
          "fields": [
            { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
            { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
          ]
    }
```

### <a name="indexer-request"></a>Solicitud del indexador

Esta solicitud muestra un indexador completamente especificado. Incluye asignaciones de campos que se omitieron en ejemplos anteriores. Recuerde que los valores "schedule", "parameters" y "fieldMappings" son opcionales, siempre haya un valor predeterminado disponible. Si se omite "schedule", el indexador se ejecuta inmediatamente. Si se omite "parsingMode", el índice usará el valor predeterminado "json".

Si se crea el indexador en Azure Cognitive Search, se desencadena la importación de datos. Se ejecuta inmediatamente y, después, se ejecuta siguiendo una programación si se ha proporcionado una.

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key for Azure Cognitive Search]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "json" } },
      "fieldMappings" : [
        { "sourceFieldName" : "/article/text", "targetFieldName" : "text" },
        { "sourceFieldName" : "/article/datePublished", "targetFieldName" : "date" },
        { "sourceFieldName" : "/article/tags", "targetFieldName" : "tags" }
        ]
    }
```

<a name="json-indexer-dotnet"></a>

## <a name="use-net-sdk"></a>Uso del SDK de .NET

El SDK de .NET tiene paridad completa con la API REST. Se recomienda que revise la sección anterior de la API REST para obtener información sobre los conceptos, el flujo de trabajo y los requisitos. A continuación, puede consultar la siguiente documentación de referencia de la API de .NET para implementar un indizador JSON en código administrado.

+ [Microsoft.Azure.Search.Models.DataSource](/dotnet/api/microsoft.azure.search.models.datasource?view=azure-dotnet)
+ [Microsoft.azure.search.models.datasourcetype](/dotnet/api/microsoft.azure.search.models.datasourcetype?view=azure-dotnet) 
+ [Microsoft.azure.search.models.index](/dotnet/api/microsoft.azure.search.models.index?view=azure-dotnet) 
+ [Microsoft.azure.search.models.indexer](/dotnet/api/microsoft.azure.search.models.indexer?view=azure-dotnet)

<a name="parsing-modes"></a>

## <a name="parsing-modes"></a>Modos de análisis

Los blobs JSON pueden asumir varias formas. El parámetro **parsingMode** en el indexador JSON determina cómo se analiza el contenido del blob JSON y se estructura en un índice de Azure Cognitive Search:

| parsingMode | Descripción |
|-------------|-------------|
| `json`  | Indexe cada blob como un solo documento. Este es el valor predeterminado. |
| `jsonArray` | Elija este modo si los blobs están compuestos por matrices JSON y necesita que cada elemento de la matriz se convierta en un documento independiente en Azure Cognitive Search. |
|`jsonLines` | Elija este modo si los blobs están compuestos por varias entidades JSON separadas por una nueva línea y necesita que cada entidad se convierta en un documento independiente en Azure Cognitive Search. |

Un documento se puede pensar como un solo elemento en los resultados de búsqueda. Si quiere que cada elemento de la matriz aparezca en los resultados de búsqueda como un elemento independiente, use la opción `jsonArray` o `jsonLines`, según le convenga.

En la definición del indizador, utilice las [asignaciones de campo](search-indexer-field-mappings.md) para elegir las propiedades del documento JSON de origen que se utilizarán para completar el índice de búsqueda de destino. En el caso del modo de análisis `jsonArray`, si la matriz existe como propiedad de nivel inferior, puede establecer una raíz del documento que indique dónde se coloca la matriz en el blob.

> [!IMPORTANT]
> Cuando se usa el modo de análisis `json`, `jsonArray` o `jsonLines`, Azure Cognitive Search da por hecho que todos los blobs del origen de datos contienen JSON. Si necesita admitir una mezcla de blobs JSON y que no son JSON en el mismo origen de datos, háganoslo saber en [nuestro sitio UserVoice](https://feedback.azure.com/forums/263029-azure-search).


<a name="parsing-single-blobs"></a>

## <a name="parse-single-json-blobs"></a>Analizar blobs JSON únicos

De manera predeterminada, el [indexador de blobs de Azure Cognitive Search](search-howto-indexing-azure-blob-storage.md) analiza los blobs JSON como un único fragmento de texto. A menudo se desea conservar la estructura de los documentos de JSON. Por ejemplo, suponga que tiene el siguiente documento JSON en Azure Blob Storage:

```http
    {
        "article" : {
            "text" : "A hopefully useful article explaining how to parse JSON blobs",
            "datePublished" : "2016-04-13",
            "tags" : [ "search", "storage", "howto" ]    
        }
    }
```

El indexador de blobs analiza y convierte el documento JSON en un único documento de Azure Cognitive Search. El indizador carga un índice al hacer que coincidan los campos "text", "datePublished" y "tags" del origen con los campos de destino con nombre y tipo idénticos.

Como se indicó, las asignaciones de campos no son necesarias. Dado un índice con los campos "text", "datePublished" y "tags", el indexador de blobs puede inferir la asignación correcta sin que haya presente una asignación de campos en la solicitud.

<a name="parsing-arrays"></a>

## <a name="parse-json-arrays"></a>Analizar matrices JSON

Si lo prefiere, puede usar la opción de matriz JSON. Esta opción es útil cuando los blobs contienen una *matriz de objetos JSON bien formados* y quiere que cada elemento se convierta en un documento de Azure Cognitive Search independiente. Por ejemplo, dado el blob JSON siguiente, puede rellenar el índice de Azure Cognitive Search con tres documentos independientes, cada uno de ellos con los campos "id" y "text".  

```text
    [
        { "id" : "1", "text" : "example 1" },
        { "id" : "2", "text" : "example 2" },
        { "id" : "3", "text" : "example 3" }
    ]
```

Para una matriz JSON, la definición del indizador debe ser similar a la del ejemplo siguiente. Tenga en cuenta que el parámetro parsingMode especifica al analizador `jsonArray`. Para indexar blobs JSON, los dos únicos requisitos específicos para las matrices son especificar el analizador correcto y tener los datos de entrada correctos.

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "jsonArray" } }
    }
```

Nuevamente, tenga en cuenta que las asignaciones de campos pueden omitirse. Dado un índice con campos "id" y "text" del mismo nombre, el indizador de blobs puede inferir la asignación correcta sin una lista de asignación de campos explícita.

<a name="nested-json-arrays"></a>

## <a name="parse-nested-arrays"></a>Analizar matrices anidadas
Para las matrices JSON que tienen elementos anidados, puede especificar un `documentRoot` para indicar una estructura de varios niveles. Por ejemplo, si los blobs tienen el siguiente aspecto:

```http
    {
        "level1" : {
            "level2" : [
                { "id" : "1", "text" : "Use the documentRoot property" },
                { "id" : "2", "text" : "to pluck the array you want to index" },
                { "id" : "3", "text" : "even if it's nested inside the document" }  
            ]
        }
    }
```

Utilice esta configuración para indexar la matriz de la propiedad `level2`:

```http
    {
        "name" : "my-json-array-indexer",
        ... other indexer properties
        "parameters" : { "configuration" : { "parsingMode" : "jsonArray", "documentRoot" : "/level1/level2" } }
    }
```

## <a name="parse-blobs-separated-by-newlines"></a>Analizar blobs separados por nuevas líneas

Si el blob contiene varias entidades JSON separadas por una nueva línea y quiere que cada elemento se convierta en un documento independiente en Azure Cognitive Search, puede optar por la opción de líneas JSON. Por ejemplo, en el blob JSON siguiente (que tiene tres entidades JSON distintas), puede rellenar el índice de Azure Cognitive Search con tres documentos independientes, cada uno de ellos con los campos "id" y "text".

```text
{ "id" : "1", "text" : "example 1" }
{ "id" : "2", "text" : "example 2" }
{ "id" : "3", "text" : "example 3" }
```

Para las líneas JSON, la definición del indizador debe ser similar a la del ejemplo siguiente. Tenga en cuenta que el parámetro parsingMode especifica al analizador `jsonLines`. 

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "jsonLines" } }
    }
```

Al igual que ocurría con el modo de análisis `jsonArray`, tenga en cuenta que las asignaciones de campos pueden omitirse.

## <a name="add-field-mappings"></a>Agregar asignaciones de campos

Cuando los campos de origen y de destino no estén perfectamente alineados, puede definir una sección de asignación de campos en el cuerpo de la solicitud para crear asociaciones explícitas de campo a campo.

Actualmente, Azure Cognitive Search no puede indexar documentos JSON arbitrarios directamente, ya que admite solo tipos de datos primitivos, matrices de cadenas y puntos de GeoJSON. Sin embargo, puede usar **asignaciones de campo** para seleccionar partes del documento JSON y los "elevan" a campos de nivel superior del documento de búsqueda. Para información sobre los conceptos básicos de las asignaciones de campos, consulte [Asignaciones de campos en los indexadores de Azure Cognitive Search](search-indexer-field-mappings.md).

Volviendo al documento JSON de ejemplo:

```http
    {
        "article" : {
            "text" : "A hopefully useful article explaining how to parse JSON blobs",
            "datePublished" : "2016-04-13"
            "tags" : [ "search", "storage", "howto" ]    
        }
    }
```

Supongamos que tiene un índice de búsqueda con los siguientes campos: `text` del tipo `Edm.String`, `date` del tipo `Edm.DateTimeOffset` y `tags` del tipo `Collection(Edm.String)`. Tenga en cuenta la diferencia entre "datePublished" en el origen y el campo `date` en el índice. Para asignar JSON en la forma deseada, utilice las siguientes asignaciones de campo:

```http
    "fieldMappings" : [
        { "sourceFieldName" : "/article/text", "targetFieldName" : "text" },
        { "sourceFieldName" : "/article/datePublished", "targetFieldName" : "date" },
        { "sourceFieldName" : "/article/tags", "targetFieldName" : "tags" }
      ]
```

Los nombres de campo de origen en las asignaciones se especifican con la notación [puntero JSON](https://tools.ietf.org/html/rfc6901) . Comience con una barra diagonal para hacer referencia a la raíz del documento JSON y, después, seleccione en la propiedad que desee (a un nivel arbitrario de anidamiento) mediante una ruta de acceso separada por una barra diagonal.

También puede hacer referencia a elementos individuales de la matriz mediante un índice de base cero. Por ejemplo, para elegir el primer elemento de la matriz "etiquetas" del ejemplo anterior, use una asignación de campo como esta:

```http
    { "sourceFieldName" : "/article/tags/0", "targetFieldName" : "firstTag" }
```

> [!NOTE]
> Si un nombre de campo de origen en una ruta de acceso de asignación de campo hace referencia a una propiedad que no existe en JSON, la asignación se omite sin errores. Esto se realiza, por lo que podemos admitir documentos con un esquema diferente (que es un caso de uso frecuente). Puesto que no hay ninguna validación, tiene que procurar evitar tipográficos en la especificación de las asignaciones de campo.
>
>

## <a name="see-also"></a>Consulte también

+ [Indexadores de Azure Cognitive Search](search-indexer-overview.md)
+ [Indexación de Azure Blob Storage con Azure Cognitive Search](search-howto-index-json-blobs.md)
+ [Indexación de blobs CSV con el indexador de blobs de Azure Cognitive Search](search-howto-index-csv-blobs.md)
+ [Tutorial: Búsqueda de datos semiestructurados en Azure Blob Storage](search-semi-structured-data.md)