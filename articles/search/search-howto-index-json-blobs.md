---
title: 'Indexación de blobs JSON desde el indexador de blobs de Azure para la búsqueda de texto completo: Azure Search'
description: Rastree el contenido de texto de los blobs JSON de Azure mediante el indexador de blobs de Azure Search. Los indexadores automatizan la ingesta de datos para orígenes de datos seleccionados, como Azure Blob Storage.
ms.date: 05/02/2019
author: HeidiSteen
manager: cgronlun
ms.author: heidist
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seodec2018
ms.openlocfilehash: 4f3fb624f5e6137c9edb0be97adc16d8c808ebd9
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65523068"
---
# <a name="how-to-index-json-blobs-using-azure-search-blob-indexer"></a>Cómo indexar blobs JSON con el indexador de blobs de Azure Search
Este artículo muestra cómo configurar un blob de Azure Search [indizador](search-indexer-overview.md) para extraer contenido estructurado de documentos JSON en Azure Blob storage y facilitar su búsqueda en Azure Search. Este flujo de trabajo crea un índice de búsqueda de Azure y lo carga con texto extraído de los blobs JSON. 

Puede usar el [portal](#json-indexer-portal), las [API REST](#json-indexer-rest) o el [SDK de .NET](#json-indexer-dotnet) para indexar el contenido JSON. Comunes a todos los métodos es que se encuentran documentos JSON en un contenedor de blobs en una cuenta de almacenamiento de Azure. Para obtener instrucciones sobre la inserción de documentos JSON desde otras plataformas que no sean de Azure, consulte [Importación de datos en Azure Search](search-what-is-data-import.md).

Los blobs JSON en Azure Blob storage suelen ser un solo documento JSON o una colección de entidades JSON. Para las colecciones de JSON, el blob podría tener un **matriz** de elementos JSON con formato correcto. Los blobs también podrían estar compuestos de varias entidades JSON individuales separadas por una línea nueva. El indexador de blobs de Azure Search puede analizar dicha construcción, según cómo configure el **parsingMode** parámetro en la solicitud.

JSON de todos los modos de análisis (`json`, `jsonArray`, `jsonLines`) ahora están disponibles con carácter general. 

> [!NOTE]
> Siga las recomendaciones de configuración de indexador de [indización de uno a varios](search-howto-index-one-to-many-blobs.md) para generar varios documentos de búsqueda de un blob de Azure.

<a name="json-indexer-portal"></a>

## <a name="use-the-portal"></a>Uso del portal

El método más sencillo para la indexación de documentos JSON es usar un asistente en [Azure Portal](https://portal.azure.com/). Mediante el análisis de metadatos en el contenedor de blobs de Azure, el asistente [**Importar datos**](search-import-data-portal.md) puede crear un índice predeterminado, asignar campos de origen a campos de índice de destino y cargar el índice en una sola operación. Según el tamaño y la complejidad del origen de datos, puede tener un índice de búsqueda de texto completo y operativo en cuestión de minutos.

Se recomienda usar la misma suscripción de Azure para Azure Search y Azure storage, preferiblemente en la misma región.

### <a name="1---prepare-source-data"></a>1: Preparación de los datos de origen

1. [Inicie sesión en Azure Portal](https://portal.azure.com/).

1. [Crear un contenedor de blobs](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal) para contener los datos. El nivel de acceso público se puede establecer en cualquiera de sus valores válidos.

Necesitará el nombre de cuenta de almacenamiento, nombre del contenedor y una clave de acceso para recuperar los datos en el **importar datos** asistente.

### <a name="2---start-import-data-wizard"></a>2: Inicio del asistente para la importación de datos

En la página información general del servicio Azure Search, puede [iniciar el Asistente para](search-import-data-portal.md) desde la barra de comandos, o haciendo clic en **agregar Azure Search** en el **servicio Blob** sección de la cuenta de almacenamiento izquierdo del panel de navegación.

   ![Comando de importación de datos en el portal](./media/search-import-data-portal/import-data-cmd2.png "Iniciar el asistente para la importación de datos")

### <a name="3---set-the-data-source"></a>3: Configuración del origen de datos

En la página **origen de datos**, el origen debe ser **Azure Blob Storage** con las especificaciones siguientes:

+ **Datos que se extraerán** debe seleccionar el valor *Contenido y metadatos*. Si elige esta opción, permitirá que el asistente deduzca un esquema de índice y asigne los campos para la importación.
   
+ **Modo de análisis** debe establecerse en *JSON*, *matriz JSON* o *líneas JSON*. 

  La opción *JSON* articula cada blob como un documento único de búsqueda, por lo que aparecen como un elemento independiente en los resultados de búsqueda. 

  *Matriz JSON* es para los blobs que contienen datos JSON con formato correcto - JSON correcto corresponde a una matriz de objetos, o tiene una propiedad que es una matriz de objetos y quiere que cada elemento se articula como una independiente, el documento de búsqueda independiente. Si los blobs son complejos y no elige la opción *matriz JSON*, todo el blob se ingiere como un único documento.

  *Líneas JSON* es para los blobs se componen de varias entidades JSON separadas por una nueva línea, donde desea que se articula como documento independiente búsqueda independiente cada entidad. Si los blobs son complejos y no elige *líneas JSON* análisis modo, a continuación, en todo el blob se ingieren como un único documento.
   
+ El **Contenedor de almacenamiento** debe especificar su cuenta de almacenamiento y contenedor o una cadena de conexión que se resuelva en el contenedor. Puede obtener las cadenas de conexión en la página del portal de Blob service.

   ![Definición del origen de datos del blob](media/search-howto-index-json/import-wizard-json-data-source.png)

### <a name="4---skip-the-add-cognitive-search-page-in-the-wizard"></a>4: Omisión de la página "Agregar Cognitive Search" en el asistente

No es necesario agregar conocimientos cognitivos para la importación de documentos JSON. A menos que necesite específicamente [incluir Cognitive Services APIs y transformaciones](cognitive-search-concept-intro.md) a la canalización de indización, omita este paso.

Para omitir el paso, vaya a la página siguiente.

   ![Botón Página siguiente de Cognitive Search](media/search-get-started-portal/next-button-add-cog-search.png)

Desde dicha página puede ir directamente a la personalización del índice.

   ![Omitir el paso sobre aptitud cognitiva](media/search-get-started-portal/skip-cog-skill-step.png)

### <a name="5---set-index-attributes"></a>5: Configuración de los atributos de índice

En la página de **Índice**, debería mostrarse una lista de campos con un tipo de datos y una serie de casillas de verificación para configurar los atributos del índice. El asistente puede generar una lista de campos en función de los metadatos y mediante el muestreo de los datos de origen. 

Puede seleccionar atributos de forma masiva haciendo clic en la casilla de verificación en la parte superior de una columna de atributos. Elija **Retrievable** y **Searchable** para cada campo que se debe devolver a una aplicación cliente y está sujeta a proceso de búsqueda de texto completo. Observará que enteros no son de texto completo o parcial que se puede buscar (los números se evalúan literalmente y suelen ser útiles en filtros).

Revise la descripción de [atributos de índice](https://docs.microsoft.com/rest/api/searchservice/create-index#bkmk_indexAttrib) y [analizadores de lenguaje](https://docs.microsoft.com/rest/api/searchservice/language-support) para obtener más información. 

Dedique un momento a la revisión de las selecciones. Una vez que se ejecuta al asistente, se crean las estructuras de datos físicas y no podrá modificar estos campos sin quitar y volver a crear todos los objetos.

   ![Definición de índice de blob](media/search-howto-index-json/import-wizard-json-index.png)

### <a name="6---create-indexer"></a>6: Creación del indizador

Con todas las especificaciones agregadas, el asistente crea tres objetos distintos en el servicio de búsqueda. Un objeto de origen de datos y un objeto de índice se guardan como recursos con nombre en el servicio Azure Search. El último paso crea un objeto de indizador. Al asignarle un nombre al indizador, este puede existir como un recurso independiente que se puede programar y administrar independientemente de los objetos de origen de datos y del de índice que se crearon en la misma secuencia del asistente.

Si no está familiarizado con los indizadores, un *indizador* es un recurso en Azure Search que rastrea un origen de datos externo en busca de contenido utilizable en búsquedas. La salida del asistente **Importar datos**  es un indizador que se rastrea el origen de datos JSON, extrae el contenido utilizable en búsquedas y lo importa a un índice en Azure Search.

   ![Definición del indizador del blob](media/search-howto-index-json/import-wizard-json-indexer.png)

Haga clic en **Aceptar** para ejecutar el asistente y crear todos los objetos. La indexación comienza inmediatamente.

Puede supervisar la importación de datos en las páginas del portal. Las notificaciones de progreso indican el estado de la indexación y cuántos documentos se cargan. 

Cuando se complete la indización, puede usar el [Explorador de búsqueda](search-explorer.md) para consultar el índice.

> [!NOTE]
> Si no ve los datos esperados, necesita establecer más atributos en varios campos. Eliminar el índice e indexador que acaba de crea y recorrer el Asistente para nuevo, modificar las selecciones para los atributos de índice en el paso 5. 

<a name="json-indexer-rest"></a>

## <a name="use-rest-apis"></a>Uso de API REST

Puede usar la API de REST para indexar blobs JSON, después de un flujo de trabajo de tres partes comunes a todos los indexadores de Azure Search: crear un origen de datos, crear un índice, cree un indexador. Extracción de datos de blob storage se produce cuando se envía la solicitud de creación de indizador. Una vez finalizada esta solicitud, tendrá un índice consultable. 

Puede revisar [código de ejemplo REST](#rest-example) al final de esta sección se muestra cómo crear los tres objetos. En esta sección también contiene detalles sobre [modos de análisis de JSON](#parsing-modes), [único blobs](#parsing-single-blobs), [matrices JSON](#parsing-arrays), y [anidados matrices](#nested-json-arrays).

Para la indexación de JSON de basado en código, utilice [Postman](search-fiddler.md) y la API de REST para crear estos objetos:

+ [index](https://docs.microsoft.com/rest/api/searchservice/create-index)
+ [Origen de datos](https://docs.microsoft.com/rest/api/searchservice/create-data-source)
+ [indexer](https://docs.microsoft.com/rest/api/searchservice/create-indexer)

Orden de las operaciones requiere que crear y llamar a los objetos en este orden. A diferencia del flujo de trabajo de portal, un enfoque de código requiere un índice disponible para aceptar los documentos JSON envían a través de la **crear indizador** solicitud.

Los blobs JSON en Azure Blob storage suelen ser un solo documento JSON o una "matriz" de JSON. El indexador de blobs de Azure Search puede analizar cualquier construcción, según cómo configure el parámetro **parsingMode** en la solicitud.

| Documento JSON | parsingMode | DESCRIPCIÓN | Disponibilidad |
|--------------|-------------|--------------|--------------|
| Uno por blob | `json` | Analiza los blobs JSON como un único fragmento de texto. Cada blob JSON se convierte en un único documento de Azure Search. | Disponible con carácter general en ambos [REST](https://docs.microsoft.com/rest/api/searchservice/indexer-operations) API y [.NET](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexer) SDK. |
| Varios por blob | `jsonArray` | Analiza una matriz JSON en el blob, donde cada elemento de la matriz se convierte en un documento de Azure Search independiente.  | Disponible con carácter general en ambos [REST](https://docs.microsoft.com/rest/api/searchservice/indexer-operations) API y [.NET](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexer) SDK. |
| Varios por blob | `jsonLines` | Analiza un blob que contiene varias entidades JSON (una "matriz"), separadas por una nueva línea, donde cada entidad se convierte en un documento de Azure Search independiente. | Disponible con carácter general en ambos [REST](https://docs.microsoft.com/rest/api/searchservice/indexer-operations) API y [.NET](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexer) SDK. |

### <a name="1---assemble-inputs-for-the-request"></a>1 - ensamblar las entradas para la solicitud

Para cada solicitud, debe proporcionar el nombre del servicio y la clave de administrador de Azure Search (en el encabezado POST) y el nombre de cuenta de almacenamiento y la clave para blob storage. Puede usar [Postman](search-fiddler.md) para enviar solicitudes HTTP a Azure Search.

Copie los cuatro valores siguientes en el Bloc de notas para que pueda pegarlos en una solicitud:

+ Nombre del servicio Azure Search
+ Clave de administración de Azure Search
+ Nombre de la cuenta de almacenamiento de Azure
+ Clave de cuenta de almacenamiento de Azure

Puede encontrar estos valores en el portal:

1. En las páginas del portal de Azure Search, copie la dirección URL de servicio de búsqueda desde la página información general.

2. En el panel de navegación izquierdo, haga clic en **claves** y, a continuación, copie la clave principal o secundaria (son equivalentes).

3. Cambiar a las páginas del portal para la cuenta de almacenamiento. En el panel de navegación izquierdo, bajo **configuración**, haga clic en **claves de acceso**. Esta página proporciona el nombre de cuenta y la clave. Copie el nombre de cuenta de almacenamiento y una de las claves en el Bloc de notas.

### <a name="2---create-a-data-source"></a>2 - Creación de un origen de datos

Este paso proporciona información de conexión de origen de datos utilizado por el indizador. El origen de datos es un objeto con nombre en Azure Search que conserva la información de conexión. Tipo de origen de los datos `azureblob`, determina qué comportamientos de extracción de datos se invocan mediante el indizador. 

Sustituya los valores válidos para el nombre del servicio, la clave de administración, la cuenta de almacenamiento y marcadores de posición de claves de cuenta.

    POST https://[service name].search.windows.net/datasources?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key for Azure Search]

    {
        "name" : "my-blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-container", "query" : "optional, my-folder" }
    }   

### <a name="3---create-a-target-search-index"></a>3 - Creación de un índice de búsqueda de destino 

Los indexadores se emparejan con un esquema de índice. Si utiliza la API (en lugar del portal), prepare un índice de antemano para poder especificarlo en la operación del indexador.

El índice almacena el contenido utilizable en búsquedas en Azure Search. Para crear un índice, proporcione un esquema que especifique los campos de un documento, los atributos y otras construcciones que conforman la experiencia de búsqueda. Si crea un índice que tenga los mismos nombres de campo y tipos de datos que el origen, el indizador hará coincidir los campos de origen y de destino, por lo que se ahorrará el trabajo asignar explícitamente los campos.

El ejemplo siguiente muestra una solicitud de [Creación de índice](https://docs.microsoft.com/rest/api/searchservice/create-index). El índice tendrá un campo `content` utilizable en búsquedas para almacenar el texto extraído de los blobs:   

    POST https://[service name].search.windows.net/indexes?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key for Azure Search]

    {
          "name" : "my-target-index",
          "fields": [
            { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
            { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
          ]
    }


### <a name="4---configure-and-run-the-indexer"></a>4: configurar y ejecutar el indexador

Como con un índice y datos de un origen y el indizador también es un nombre de objeto que puede crear y volver a usar en un servicio Azure Search. Una solicitud totalmente especificada para crear un indexador podría tener el aspecto siguiente:

    POST https://[service name].search.windows.net/indexers?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key for Azure Search]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "json" } }
    }

Configuración del indexador está en el cuerpo de la solicitud. Requiere un origen de datos y un índice de destino vacío que ya existe en Azure Search. 

Programación y los parámetros son opcionales. Si se omite, el indizador se ejecuta inmediatamente, con `json` como el modo de análisis.

Este indizador determinado no incluye las asignaciones de campos. Dentro de la definición del indexador, puede dejar al margen **asignaciones de campos** si las propiedades del documento JSON de origen coincide con los campos del índice de búsqueda de destino. 


### <a name="rest-example"></a>Ejemplo de REST

En esta sección es un resumen de todas las solicitudes que se usa para crear objetos. Para obtener una explicación de las partes componentes, consulte las secciones anteriores de este artículo.

### <a name="data-source-request"></a>Solicitud de origen de datos

Todos los indizadores requieren un objeto de origen de datos que proporciona información de conexión a los datos existentes. 

    POST https://[service name].search.windows.net/datasources?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key for Azure Search]

    {
        "name" : "my-blob-datasource",
        "type" : "azureblob",
        "credentials" : { "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>;" },
        "container" : { "name" : "my-container", "query" : "optional, my-folder" }
    }  


### <a name="index-request"></a>Solicitud de índice

Todos los indizadores requieren un índice de destino que recibe los datos. El cuerpo de la solicitud define el esquema de índice, que consta de campos, con atributos para admitir el comportamiento deseado en un índice de búsqueda. Este índice debe estar vacío cuando se ejecuta el indizador. 

    POST https://[service name].search.windows.net/indexes?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key for Azure Search]

    {
          "name" : "my-target-index",
          "fields": [
            { "name": "id", "type": "Edm.String", "key": true, "searchable": false },
            { "name": "content", "type": "Edm.String", "searchable": true, "filterable": false, "sortable": false, "facetable": false }
          ]
    }


### <a name="indexer-request"></a>Solicitud del indexador

Esta solicitud muestra un indexador completamente especificada. Incluye las asignaciones de campos que se hayan omitido en los ejemplos anteriores. Recuerde que "programar", "parameters", y "fieldMappings" son opcionales, siempre hay un valor predeterminado disponible. Si se omite "programar" hace que el indizador se ejecute inmediatamente. Si se omite "parsingMode" hace que el índice que se utilizará el valor predeterminado de "json".

Crear el indexador de Azure Search desencadena la importación de datos. Inmediatamente y que posteriormente se ejecuta según una programación si se ha proporcionado uno.

    POST https://[service name].search.windows.net/indexers?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key for Azure Search]

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


<a name="json-indexer-dotnet"></a>

## <a name="use-net-sdk"></a>Uso del SDK de .NET

El SDK de .NET tiene paridad completa con la API de REST. Se recomienda que revise la sección anterior de la API REST para obtener información sobre los conceptos, el flujo de trabajo y los requisitos. A continuación, puede consultar la siguiente documentación de referencia de la API de .NET para implementar un indizador JSON en código administrado.

+ [Microsoft.Azure.Search.Models.DataSource](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.datasource?view=azure-dotnet)
+ [Microsoft.azure.search.models.datasourcetype](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.datasourcetype?view=azure-dotnet) 
+ [Microsoft.azure.search.models.index](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.index?view=azure-dotnet) 
+ [Microsoft.azure.search.models.indexer](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexer?view=azure-dotnet)

<a name="parsing-modes"></a>

## <a name="parsing-modes"></a>Modos de análisis

Los blobs JSON pueden asumir varias formas. El **parsingMode** parámetro en el indizador JSON determina cómo el contenido de JSON blob se analice y estructure en un índice de Azure Search:

| parsingMode | DESCRIPCIÓN |
|-------------|-------------|
| `json`  | Indexar cada blob como un solo documento. Este es el valor predeterminado. |
| `jsonArray` | Elegir este modo, si los blobs constan de las matrices JSON y necesita que cada elemento de la matriz se convierta en un documento independiente en Azure Search. |
|`jsonLines` | Elegir este modo, si los blobs constan de varias entidades JSON, que están separadas por una nueva línea, y necesita que cada entidad que se va a convertirse en un documento independiente en Azure Search. |

Un documento se puede pensar como un solo elemento en los resultados de búsqueda. Si desea que cada elemento de la matriz en aparecer en los resultados de búsqueda como un elemento independiente, a continuación, utilice el `jsonArray` o `jsonLines` opción según corresponda.

En la definición del indizador, utilice las [asignaciones de campo](search-indexer-field-mappings.md) para elegir las propiedades del documento JSON de origen que se utilizarán para completar el índice de búsqueda de destino. Para `jsonArray` analiza el modo, si la matriz existe como una propiedad de nivel inferior, puede establecer una raíz del documento que indica dónde se coloca la matriz en el blob.

> [!IMPORTANT]
> Cuando usas `json`, `jsonArray` o `jsonLines` modo de análisis, Azure Search da por hecho que todos los blobs en el origen de datos contienen JSON. Si necesita admitir una mezcla de blobs JSON y que no son JSON en el mismo origen de datos, háganoslo saber en [nuestro sitio UserVoice](https://feedback.azure.com/forums/263029-azure-search).


<a name="parsing-single-blobs"></a>

## <a name="parse-single-json-blobs"></a>Analizar blobs JSON únicos

De forma predeterminada, el [indexador de blobs de Azure Search](search-howto-indexing-azure-blob-storage.md) analiza los blobs JSON como un único fragmento de texto. A menudo se desea conservar la estructura de los documentos de JSON. Por ejemplo, suponga que tiene el siguiente documento JSON en Azure Blob Storage:

    {
        "article" : {
            "text" : "A hopefully useful article explaining how to parse JSON blobs",
            "datePublished" : "2016-04-13",
            "tags" : [ "search", "storage", "howto" ]    
        }
    }

El indizador de blobs analiza y convierte el documento JSON en un único documento de Azure Search. El indizador carga un índice al hacer que coincidan los campos "text", "datePublished" y "tags" del origen con los campos de destino con nombre y tipo idénticos.

Como se indicó, las asignaciones de campos no son necesarias. Dado un índice con los campos "text", "datePublished" y "tags", el indexador de blobs puede inferir la asignación correcta sin que haya presente una asignación de campos en la solicitud.

<a name="parsing-arrays"></a>

## <a name="parse-json-arrays"></a>Analizar matrices JSON

Como alternativa, puede usar la opción de la matriz JSON. Esta opción es útil cuando los blobs contienen una *matriz de objetos JSON con formato correcto*, y desea que cada elemento se convierta en un documento de Azure Search independiente. Por ejemplo, dado el blob JSON siguiente, puede rellenar el índice de Azure Search con tres documentos independientes, cada uno de ellos con los campos "id" y "text".  

    [
        { "id" : "1", "text" : "example 1" },
        { "id" : "2", "text" : "example 2" },
        { "id" : "3", "text" : "example 3" }
    ]

Para una matriz JSON, la definición del indizador debe ser similar a la del ejemplo siguiente. Tenga en cuenta que el parámetro parsingMode especifica al analizador `jsonArray`. Especificando el analizador secundario y tener los datos correctos entrada son los requisitos específicos de la matriz solo dos para indexación de blobs JSON.

    POST https://[service name].search.windows.net/indexers?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "jsonArray" } }
    }

Nuevamente, tenga en cuenta que las asignaciones de campos pueden omitirse. Dado un índice con campos "id" y "text" del mismo nombre, el indizador de blobs puede inferir la asignación correcta sin una lista de asignación de campos explícita.

<a name="nested-json-arrays"></a>

## <a name="parse-nested-arrays"></a>Analizar las matrices anidadas
Para las matrices de JSON tiene elementos anidados, puede especificar un `documentRoot` para indicar una estructura de varios nivel. Por ejemplo, si los blobs tienen el siguiente aspecto:

    {
        "level1" : {
            "level2" : [
                { "id" : "1", "text" : "Use the documentRoot property" },
                { "id" : "2", "text" : "to pluck the array you want to index" },
                { "id" : "3", "text" : "even if it's nested inside the document" }  
            ]
        }
    }

Utilice esta configuración para indexar la matriz de la propiedad `level2`:

    {
        "name" : "my-json-array-indexer",
        ... other indexer properties
        "parameters" : { "configuration" : { "parsingMode" : "jsonArray", "documentRoot" : "/level1/level2" } }
    }

## <a name="parse-blobs-separated-by-newlines"></a>Analizar blobs separados por nuevas líneas

Si el blob contiene varias entidades JSON separadas por una nueva línea, y desea que cada elemento se convierta en un documento independiente de Azure Search, puede optar por la opción de líneas JSON. Por ejemplo, dado el blob siguiente (donde hay tres entidades JSON diferentes), puede rellenar el índice de Azure Search con tres documentos independientes, cada uno con campos "id" y "text".

    { "id" : "1", "text" : "example 1" }
    { "id" : "2", "text" : "example 2" }
    { "id" : "3", "text" : "example 3" }

Para las líneas JSON, la definición del indizador debe ser similar al ejemplo siguiente. Tenga en cuenta que el parámetro parsingMode especifica al analizador `jsonLines`. 

    POST https://[service name].search.windows.net/indexers?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "my-json-indexer",
      "dataSourceName" : "my-blob-datasource",
      "targetIndexName" : "my-target-index",
      "schedule" : { "interval" : "PT2H" },
      "parameters" : { "configuration" : { "parsingMode" : "jsonLines" } }
    }

De nuevo, tenga en cuenta que las asignaciones de campos pueden se omite, similar a la `jsonArray` modo de análisis.

## <a name="add-field-mappings"></a>Agregar asignaciones de campos

Cuando los campos de origen y de destino no estén perfectamente alineados, puede definir una sección de asignación de campos en el cuerpo de la solicitud para crear asociaciones explícitas de campo a campo.

Actualmente, Azure Search no puede indexar documentos JSON arbitrarios directamente porque admite solo tipos de datos primitivos, matrices de cadenas y puntos de GeoJSON. Sin embargo, puede usar **asignaciones de campo** para seleccionar partes del documento JSON y los "elevan" a campos de nivel superior del documento de búsqueda. Para obtener información acerca de los conceptos básicos de las asignaciones de campos, consulte [Asignaciones de campos en los indexadores de Azure Search](search-indexer-field-mappings.md).

Volviendo al documento JSON de ejemplo:

    {
        "article" : {
            "text" : "A hopefully useful article explaining how to parse JSON blobs",
            "datePublished" : "2016-04-13"
            "tags" : [ "search", "storage", "howto" ]    
        }
    }

Supongamos que tiene un índice de búsqueda con los siguientes campos: `text` del tipo `Edm.String`, `date` del tipo `Edm.DateTimeOffset` y `tags` del tipo `Collection(Edm.String)`. Tenga en cuenta la diferencia entre "datePublished" en el origen y el campo `date` en el índice. Para asignar JSON en la forma deseada, utilice las siguientes asignaciones de campo:

    "fieldMappings" : [
        { "sourceFieldName" : "/article/text", "targetFieldName" : "text" },
        { "sourceFieldName" : "/article/datePublished", "targetFieldName" : "date" },
        { "sourceFieldName" : "/article/tags", "targetFieldName" : "tags" }
      ]

Los nombres de campo de origen en las asignaciones se especifican con la notación [puntero JSON](https://tools.ietf.org/html/rfc6901) . Comience con una barra diagonal para hacer referencia a la raíz del documento JSON y, después, seleccione en la propiedad que desee (a un nivel arbitrario de anidamiento) mediante una ruta de acceso separada por una barra diagonal.

También puede hacer referencia a elementos individuales de la matriz mediante un índice de base cero. Por ejemplo, para elegir el primer elemento de la matriz "etiquetas" del ejemplo anterior, use una asignación de campo como esta:

    { "sourceFieldName" : "/article/tags/0", "targetFieldName" : "firstTag" }

> [!NOTE]
> Si un nombre de campo de origen en una ruta de acceso de asignación de campo hace referencia a una propiedad que no existe en JSON, la asignación se omite sin errores. Esto se realiza, por lo que podemos admitir documentos con un esquema diferente (que es un caso de uso frecuente). Puesto que no hay ninguna validación, tiene que procurar evitar tipográficos en la especificación de las asignaciones de campo.
>
>

## <a name="see-also"></a>Vea también

+ [Indexadores de Azure Search](search-indexer-overview.md)
+ [Indexación de Azure Blob Storage con Azure Search](search-howto-index-json-blobs.md)
+ [Indexación de blobs CSV con el indexador de blobs de Azure Search](search-howto-index-csv-blobs.md)
+ [Tutorial: Búsqueda de datos semiestructurados en Azure Blob Storage](search-semi-structured-data.md)
