---
title: 'Indexación de un origen de datos de Azure Cosmos DB: Azure Search'
description: Rastree un origen de datos de Azure Cosmos DB e introduzca los datos en un índice de búsqueda de texto completo en Azure Search. Los indexadores automatizan la ingesta de datos para orígenes de datos seleccionados, como Azure Cosmos DB.
ms.date: 05/02/2019
author: mgottein
manager: cgronlun
ms.author: magottei
services: search
ms.service: search
ms.devlang: rest-api
ms.topic: conceptual
ms.custom: seodec2018
ms.openlocfilehash: 07989b06b756e1e360ac3c37927a8267c84d9162
ms.sourcegitcommit: bb85a238f7dbe1ef2b1acf1b6d368d2abdc89f10
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2019
ms.locfileid: "65522830"
---
# <a name="how-to-index-cosmos-db-using-an-azure-search-indexer"></a>Cómo indexar Cosmos DB mediante un indexador de Azure Search


> [!Note]
> Compatibilidad con la API de MongoDB está en versión preliminar y no se ha diseñado para su uso en producción. El [API de REST versión 2019-05-06-Preview](search-api-preview.md) proporciona esta característica. No hay ningún portal o la compatibilidad con el SDK de .NET en este momento.
>
> API de SQL está disponible con carácter general.

Este artículo muestra cómo configurar una instancia de Azure Cosmos DB [indizador](search-indexer-overview.md) para extraer contenido y facilitar su búsqueda en Azure Search. Este flujo de trabajo crea un índice de búsqueda de Azure y lo carga con el texto existente que se extraen de Azure Cosmos DB. 

Porque terminología puede resultar confusa, merece la pena mencionar que [indexación de Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/index-overview) y [indexación de Azure Search](search-what-is-an-index.md) son operaciones distintas, únicas para cada servicio. Antes de empezar a Azure Search Index Server, la base de datos de Azure Cosmos DB debe existir previamente y contienen datos.

Puede usar el [portal](#cosmos-indexer-portal), las API de REST o SDK de .NET para indizar el contenido de Cosmos. El indexador de Cosmos DB en Azure Search puede rastrear [elementos de Azure Cosmos](https://docs.microsoft.com/azure/cosmos-db/databases-containers-items#azure-cosmos-items) tiene acceso a través de estos protocolos:

* [SQL API](https://docs.microsoft.com/azure/cosmos-db/sql-api-query-reference) 
* [API de MongoDB (versión preliminar)](https://docs.microsoft.com/azure/cosmos-db/mongodb-introduction)

> [!Note]
> Voz del usuario tiene los elementos existentes para obtener soporte técnico de API. Puede enviar un voto para las API de Cosmos que gustaría ver admitidos en Azure Search: [API de tabla](https://feedback.azure.com/forums/263029-azure-search/suggestions/32759746-azure-search-should-be-able-to-index-cosmos-db-tab), [de Graph API](https://feedback.azure.com/forums/263029-azure-search/suggestions/13285011-add-graph-databases-to-your-data-sources-eg-neo4), [Apache Cassandra API](https://feedback.azure.com/forums/263029-azure-search/suggestions/32857525-indexer-crawler-for-apache-cassandra-api-in-azu).
>

<a name="cosmos-indexer-portal"></a>

## <a name="use-the-portal"></a>Uso del portal

El método más sencillo para la indización de los elementos de Azure Cosmos es usar un asistente en el [portal Azure](https://portal.azure.com/). Muestreo de datos y leer los metadatos del contenedor, el [ **importar datos** ](search-import-data-portal.md) asistente en Azure Search puede crear un índice predeterminado, los campos de origen se asignan a los campos de índice de destino y cargar el índice en una única operación. Según el tamaño y la complejidad del origen de datos, puede tener un índice de búsqueda de texto completo y operativo en cuestión de minutos.

Se recomienda usar la misma suscripción de Azure para Azure Search y Azure Cosmos DB, preferiblemente en la misma región.

### <a name="1---prepare-source-data"></a>1: Preparación de los datos de origen

Debe tener una cuenta de Cosmos, una base de datos de Azure Cosmos asignado a la API de SQL o MongoDB API y un contenedor de documentos JSON. 

Asegúrese de que la base de datos de Cosmos DB contiene datos. El [Asistente para importar datos](search-import-data-portal.md) lee los metadatos y realiza el muestreo de datos para deducir un esquema de índice, pero también carga los datos de Cosmos DB. Si los datos no está presente, el asistente se detiene con este error "esquema de índice a detectar errores de origen de datos: No se pudo crear un índice prototipo porque el origen de datos 'emptycollection' no devolvió datos".

### <a name="2---start-import-data-wizard"></a>2: Inicio del asistente para la importación de datos

También puede [iniciar el Asistente para](search-import-data-portal.md) desde la barra de comandos en la página del servicio Azure Search, o haga clic en **agregar Azure Search** en el **configuración** izquierdo de la sección de la cuenta de almacenamiento panel de navegación.

   ![Comando de importación de datos en el portal](./media/search-import-data-portal/import-data-cmd2.png "Iniciar el asistente para la importación de datos")

### <a name="3---set-the-data-source"></a>3: Configuración del origen de datos

> [!NOTE] 
> Actualmente, no se puede crear o editar **MongoDB** orígenes de datos mediante Azure portal o el SDK. NET. Sin embargo, se **puede** supervisar el historial de ejecución de los indexadores de MongoDB en el portal.

En el **origen de datos** página, el origen debe ser **Cosmos DB**, con las especificaciones siguientes:

+ **Nombre** es el nombre del objeto de origen de datos. Una vez creado, puede elegir para otras cargas de trabajo.

+ **Cuenta de COSMOS DB** debe ser la cadena de conexión principal o secundaria de Cosmos DB, con un `AccountEndpoint` y un `AccountKey`. La cuenta determina si los datos se convierten como SQL API o Mongo DB API

+ **Base de datos** es una base de datos de la cuenta. 

+ **Colección** es un contenedor de documentos. Documentos deben existir para que la importación se realice correctamente. 

+ **Consulta** puede estar en blanco si desea que todos los documentos, en caso contrario, puede escribir una consulta que selecciona un subconjunto del documento. 

   ![Definición del origen de datos de COSMOS DB](media/search-howto-index-cosmosdb/cosmosdb-datasource.png "definición del origen de datos de Cosmos DB")

### <a name="4---skip-the-add-cognitive-search-page-in-the-wizard"></a>4: Omisión de la página "Agregar Cognitive Search" en el asistente

Agregar conocimientos cognitivos no es necesaria para la importación del documento. A menos que necesite específicamente [incluir Cognitive Services APIs y transformaciones](cognitive-search-concept-intro.md) a la canalización de indización, omita este paso.

Para omitir el paso, vaya a la página siguiente.

   ![Botón Página siguiente de Cognitive Search](media/search-get-started-portal/next-button-add-cog-search.png)

Desde dicha página puede ir directamente a la personalización del índice.

   ![Omitir el paso sobre aptitud cognitiva](media/search-get-started-portal/skip-cog-skill-step.png)

### <a name="5---set-index-attributes"></a>5: Configuración de los atributos de índice

En la página de **Índice**, debería mostrarse una lista de campos con un tipo de datos y una serie de casillas de verificación para configurar los atributos del índice. El asistente puede generar una lista de campos en función de los metadatos y mediante el muestreo de los datos de origen. 

Puede seleccionar atributos de forma masiva haciendo clic en la casilla de verificación en la parte superior de una columna de atributos. Elija **Retrievable** y **Searchable** para cada campo que se debe devolver a una aplicación cliente y está sujeta a proceso de búsqueda de texto completo. Observará que enteros no son de texto completo o parcial que se puede buscar (los números se evalúan literalmente y suelen ser útiles en filtros).

Revise la descripción de [atributos de índice](https://docs.microsoft.com/rest/api/searchservice/create-index#bkmk_indexAttrib) y [analizadores de lenguaje](https://docs.microsoft.com/rest/api/searchservice/language-support) para obtener más información. 

Dedique un momento a la revisión de las selecciones. Una vez que se ejecuta al asistente, se crean las estructuras de datos físicas y no podrá modificar estos campos sin quitar y volver a crear todos los objetos.

   ![Definición de índice de COSMOS DB](media/search-howto-index-cosmosdb/cosmosdb-index-schema.png "definición del índice de Cosmos DB")

### <a name="6---create-indexer"></a>6: Creación del indizador

Con todas las especificaciones agregadas, el asistente crea tres objetos distintos en el servicio de búsqueda. Un objeto de origen de datos y un objeto de índice se guardan como recursos con nombre en el servicio Azure Search. El último paso crea un objeto de indizador. Al asignarle un nombre al indizador, este puede existir como un recurso independiente que se puede programar y administrar independientemente de los objetos de origen de datos y del de índice que se crearon en la misma secuencia del asistente.

Si no está familiarizado con los indizadores, un *indizador* es un recurso en Azure Search que rastrea un origen de datos externo en busca de contenido utilizable en búsquedas. La salida de la **importar datos** asistente es un indizador que se rastrea el origen de datos de Cosmos DB, extrae el contenido utilizable en búsquedas y lo importa en un índice en Azure Search.

Captura de pantalla siguiente muestra la configuración predeterminada del indizador. Puede cambiar a **una vez** si desea ejecutar el indexador una vez. Haga clic en **enviar** para ejecutar el asistente y crear todos los objetos. La indexación comienza inmediatamente.

   ![Definición de indexador de COSMOS DB](media/search-howto-index-cosmosdb/cosmosdb-indexer.png "definición del indexador de Cosmos DB")

Puede supervisar la importación de datos en las páginas del portal. Las notificaciones de progreso indican el estado de la indexación y cuántos documentos se cargan. 

Cuando se complete la indización, puede usar el [Explorador de búsqueda](search-explorer.md) para consultar el índice.

> [!NOTE]
> Si no ve los datos esperados, necesita establecer más atributos en varios campos. Eliminar el índice e indexador que acaba de crea y recorrer el Asistente para nuevo, modificar las selecciones para los atributos de índice en el paso 5. 

<a name="cosmosdb-indexer-rest"></a>

## <a name="use-rest-apis"></a>Uso de API REST

Puede usar la API de REST para indexar los datos de Azure Cosmos DB, después de un flujo de trabajo de tres partes comunes a todos los indexadores de Azure Search: crear un origen de datos, crear un índice, cree un indexador. Extracción de datos de almacenamiento de Cosmos tiene lugar cuando se envía la solicitud de creación de indizador. Una vez finalizada esta solicitud, tendrá un índice consultable. 

Si va a evaluar MongoDB, debe usar el resto `api-version=2019-05-06-Preview` para crear el origen de datos.

En la cuenta de Cosmos DB, puede elegir si desea que la recopilación indexe automáticamente todos los documentos. De forma predeterminada, todos los documentos se indexan automáticamente, pero puede desactivar la indexación automática. Cuando se desactiva la indexación, solo se puede acceder a los documentos a través de sus propios vínculos o mediante su identificador. Azure Search necesita que se active la indexación automática de Cosmos DB en la colección que Azure Search va a indexar. 

> [!WARNING]
> Azure Cosmos DB es la siguiente generación de DocumentDB. Anteriormente con la versión de API **2017-11-11** podría utilizar el `documentdb` sintaxis. Esto significaba que puede especificar el tipo de origen de datos como `cosmosdb` o `documentdb`. Comenzando con la versión de API **2019-05-06** la API de Azure Search y el Portal solo admiten el `cosmosdb` sintaxis como se indica en este artículo. Esto significa que el tipo de origen de datos debe `cosmosdb` si desea conectarse a un punto de conexión de Cosmos DB.

### <a name="1---assemble-inputs-for-the-request"></a>1 - ensamblar las entradas para la solicitud

Para cada solicitud, debe proporcionar el nombre del servicio y la clave de administrador de Azure Search (en el encabezado POST) y el nombre de cuenta de almacenamiento y la clave para blob storage. Puede usar [Postman](search-fiddler.md) para enviar solicitudes HTTP a Azure Search.

Copie los cuatro valores siguientes en el Bloc de notas para que pueda pegarlos en una solicitud:

+ Nombre del servicio Azure Search
+ Clave de administración de Azure Search
+ Cadena de conexión de COSMOS DB

Puede encontrar estos valores en el portal:

1. En las páginas del portal de Azure Search, copie la dirección URL de servicio de búsqueda desde la página información general.

2. En el panel de navegación izquierdo, haga clic en **claves** y, a continuación, copie la clave principal o secundaria (son equivalentes).

3. Cambiar a las páginas del portal para la cuenta de almacenamiento de Cosmos. En el panel de navegación izquierdo, bajo **configuración**, haga clic en **claves**. Esta página proporciona un URI, dos conjuntos de cadenas de conexión, y dos conjuntos de claves. En el Bloc de notas, copie una de las cadenas de conexión.

### <a name="2---create-a-data-source"></a>2 - Creación de un origen de datos

A **origen de datos** especifica los datos para indexar, las credenciales y las directivas para identificar cambios en los datos (por ejemplo, los documentos modificados o eliminados dentro de la colección). El origen de datos se define como un recurso independiente para que puedan usarlo múltiples indizadores.

Para crear un origen de datos, formular una solicitud POST:

    POST https://[service name].search.windows.net/datasources?api-version=2019-05-06
    Content-Type: application/json
    api-key: [Search service admin key]

    {
        "name": "mycosmosdbdatasource",
        "type": "cosmosdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myCosmosDbEndpoint.documents.azure.com;AccountKey=myCosmosDbAuthKey;Database=myCosmosDbDatabaseId"
        },
        "container": { "name": "myCollection", "query": null },
        "dataChangeDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "_ts"
        }
    }

El cuerpo de la solicitud contiene la definición del origen de datos, que debe incluir los siguientes campos:

| Campo   | DESCRIPCIÓN |
|---------|-------------|
| **name** | Necesario. Elija un nombre para representar el objeto de origen de datos. |
|**type**| Necesario. Debe ser `cosmosdb`. |
|**credentials** | Necesario. Debe ser una cadena de conexión de Cosmos DB.<br/>Para las colecciones de SQL, las cadenas de conexión están en este formato: `AccountEndpoint=<Cosmos DB endpoint url>;AccountKey=<Cosmos DB auth key>;Database=<Cosmos DB database id>`<br/>Para las colecciones de MongoDB, agregue **determinada = MongoDb** a la cadena de conexión:<br/>`AccountEndpoint=<Cosmos DB endpoint url>;AccountKey=<Cosmos DB auth key>;Database=<Cosmos DB database id>;ApiKind=MongoDb`<br/>Evite los números de puerto en la dirección URL del punto de conexión. Si incluye el número de puerto, Azure Search no podrá indexar la base de datos de Azure Cosmos DB.|
| **container** | contiene los siguientes elementos: <br/>**name**: Necesario. Especifique el identificador de la colección de base de datos que se puede indizar.<br/>**query**: Opcional. Puede especificar una consulta para acoplar un documento JSON arbitrario en un esquema plano que Azure Search pueda indizar.<br/>Para las colecciones de MongoDB, no se admiten las consultas. |
| **dataChangeDetectionPolicy** | Se recomienda su uso. Consulte la sección [Indexación de documentos modificados](#DataChangeDetectionPolicy).|
|**dataDeletionDetectionPolicy** | Opcional. Consulte la sección [Indexación de documentos eliminados](#DataDeletionDetectionPolicy).|

### <a name="using-queries-to-shape-indexed-data"></a>Uso de consultas para dar forma a los datos indizados
Puede especificar una consulta de SQL para eliminar el formato de las propiedades o matrices anidadas y de las propiedades JSON del proyecto, y para filtrar los datos que se van a indexar. 

> [!WARNING]
> No se admiten consultas personalizadas para colecciones de **MongoDB**: el parámetro `container.query` debe establecerse en null u omitirse. Si tiene que usar una consulta personalizada, háganoslo saber en [User Voice](https://feedback.azure.com/forums/263029-azure-search).

Documento de ejemplo:

    {
        "userId": 10001,
        "contact": {
            "firstName": "andy",
            "lastName": "hoh"
        },
        "company": "microsoft",
        "tags": ["azure", "cosmosdb", "search"]
    }

Consulta de filtro:

    SELECT * FROM c WHERE c.company = "microsoft" and c._ts >= @HighWaterMark ORDER BY c._ts

Consulta sin formato:

    SELECT c.id, c.userId, c.contact.firstName, c.contact.lastName, c.company, c._ts FROM c WHERE c._ts >= @HighWaterMark ORDER BY c._ts
    
    
Consulta de proyección:

    SELECT VALUE { "id":c.id, "Name":c.contact.firstName, "Company":c.company, "_ts":c._ts } FROM c WHERE c._ts >= @HighWaterMark ORDER BY c._ts


Consulta sin formato de matriz:

    SELECT c.id, c.userId, tag, c._ts FROM c JOIN tag IN c.tags WHERE c._ts >= @HighWaterMark ORDER BY c._ts


### <a name="3---create-a-target-search-index"></a>3 - Creación de un índice de búsqueda de destino 

[Crear un índice de búsqueda de Azure de destino](/rest/api/searchservice/create-index) si aún no tiene uno. El ejemplo siguiente crea un índice con un campo de identificador y la descripción:

    POST https://[service name].search.windows.net/indexes?api-version=2019-05-06
    Content-Type: application/json
    api-key: [Search service admin key]

    {
       "name": "mysearchindex",
       "fields": [{
         "name": "id",
         "type": "Edm.String",
         "key": true,
         "searchable": false
       }, {
         "name": "description",
         "type": "Edm.String",
         "filterable": false,
         "sortable": false,
         "facetable": false,
         "suggestions": true
       }]
     }

Asegúrese de que el esquema del índice de destino es compatible con el de los documentos JSON de origen o el resultado de la proyección de consultas personalizada.

> [!NOTE]
> Para las colecciones particionadas, la clave de documento predeterminada es la propiedad `_rid` de Azure Cosmos DB, para la que Azure Search cambia el nombre automáticamente a `rid`, porque los nombres de los campos no pueden comenzar con un carácter de guion bajo. Además, los valores `_rid` de Azure Cosmos DB contienen caracteres que no son válidos en las claves de Azure Search. Por este motivo, la `_rid` valores están codificados con Base64.
> 
> Para las colecciones de MongoDB, Azure Search cambia el nombre de la propiedad `_id` automáticamente a `doc_id`.  

### <a name="mapping-between-json-data-types-and-azure-search-data-types"></a>asignación entre tipos de datos de JSON y de Azure Search
| Tipo de datos de JSON | Tipos de campos de índice de destino compatibles |
| --- | --- |
| Bool |Edm.Boolean, Edm.String |
| Números que parecen enteros |Edm.Int32, Edm.Int64, Edm.String |
| Números que parecen puntos flotantes |Edm.Double, Edm.String |
| String |Edm.String |
| Matrices de tipos primitivos, por ejemplo ["a", "b", "c"] |Collection(Edm.String) |
| Cadenas que parecen fechas |Edm.DateTimeOffset, Edm.String |
| Objetos GeoJSON, por ejemplo {"type": "Point", "coordinates": [long, lat] } |Edm.GeographyPoint |
| Otros objetos JSON |N/D |

### <a name="4---configure-and-run-the-indexer"></a>4: configurar y ejecutar el indexador

Una vez creados el origen de datos y los índices, ya podrá crear el indizador:

    POST https://[service name].search.windows.net/indexers?api-version=2019-05-06
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "mycosmosdbindexer",
      "dataSourceName" : "mycosmosdbdatasource",
      "targetIndexName" : "mysearchindex",
      "schedule" : { "interval" : "PT2H" }
    }

Este indexador se ejecuta cada dos horas (el intervalo de programación se establece en "PT2H"). Para ejecutar un indizador cada 30 minutos, establézcalo en PT30M. El intervalo más breve que se admite es de 5 minutos. La programación es opcional: si se omite, el indizador solo se ejecuta una vez cuando se crea. Sin embargo, puede ejecutarlo a petición en cualquier momento.   

Para más información sobre la API Create Indexer, consulte [Crear indexador](https://docs.microsoft.com/rest/api/searchservice/create-indexer).

## <a name="use-net"></a>Uso de .NET

El SDK de .NET disponible con carácter general tiene paridad completa con la API de REST disponible con carácter general. Se recomienda que revise la sección anterior de la API REST para obtener información sobre los conceptos, el flujo de trabajo y los requisitos. A continuación, puede consultar la siguiente documentación de referencia de la API de .NET para implementar un indizador JSON en código administrado.

+ [Microsoft.Azure.Search.Models.DataSource](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.datasource?view=azure-dotnet)
+ [Microsoft.azure.search.models.datasourcetype](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.datasourcetype?view=azure-dotnet) 
+ [Microsoft.azure.search.models.index](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.index?view=azure-dotnet) 
+ [Microsoft.azure.search.models.indexer](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.indexer?view=azure-dotnet)

<a name="DataChangeDetectionPolicy"></a>

## <a name="indexing-changed-documents"></a>Indexación de documentos modificados

El fin de una directiva de detección de cambios de datos es identificar de forma eficaz los elementos de datos que han cambiado. Actualmente, `High Water Mark` es la única directiva compatible que usa la propiedad `_ts` (marca de tiempo) que proporciona Azure Cosmos DB, y se especifica de la siguiente forma:

    {
        "@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
        "highWaterMarkColumnName" : "_ts"
    }

Se recomienda encarecidamente usar esta directiva para garantizar un buen rendimiento del indexador. 

Si usa una consulta personalizada, asegúrese de que la consulta proyecta la propiedad `_ts`.

<a name="IncrementalProgress"></a>

### <a name="incremental-progress-and-custom-queries"></a>Progreso incremental y consultas personalizadas

El progreso incremental durante la indexación garantiza que si se interrumpe la ejecución del indexador por errores transitorios o por el límite de tiempo de ejecución, dicho indexador puede retomar la ejecución por donde la dejó la próxima vez que se ejecute, en lugar de tener que volver a indexar toda la colección desde el principio. Esto es especialmente importante cuando se indexan colecciones grandes. 

Para habilitar el progreso incremental cuando se usa una consulta personalizada, asegúrese de que la consulta ordena los resultados por la columna `_ts`. Esto permite la creación de puntos de comprobación, que Azure Search usa para proporcionar el progreso incremental en presencia de errores.   

En algunos casos, incluso si la consulta contiene una cláusula `ORDER BY [collection alias]._ts`, Azure Search puede no deducir que la consulta se ordene por `_ts`. Puede indicar a Azure Search que los resultados se ordenen mediante el uso de la propiedad de configuración `assumeOrderByHighWaterMarkColumn`. Para especificar esta sugerencia, cree o actualice el indexador como se indica a continuación: 

    {
     ... other indexer definition properties
     "parameters" : {
            "configuration" : { "assumeOrderByHighWaterMarkColumn" : true } }
    } 

<a name="DataDeletionDetectionPolicy"></a>

## <a name="indexing-deleted-documents"></a>Indexación de documentos eliminados

Cuando se eliminan filas de la recopilación, lo habitual es que también se deseen eliminar del índice de búsqueda. El fin de una directiva de detección de eliminación de datos es identificar eficazmente los elementos de datos eliminados. Actualmente, solo es compatible la directiva `Soft Delete` (la eliminación se indica con algún tipo de marca), especificada de la siguiente forma:

    {
        "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
        "softDeleteColumnName" : "the property that specifies whether a document was deleted",
        "softDeleteMarkerValue" : "the value that identifies a document as deleted"
    }

Si usa una consulta personalizada, asegúrese de que la consulta proyecta la propiedad a la que `softDeleteColumnName` hace referencia.

En el ejemplo siguiente se crea un origen de datos con una directiva de eliminación temporal:

    POST https://[service name].search.windows.net/datasources?api-version=2019-05-06
    Content-Type: application/json
    api-key: [Search service admin key]

    {
        "name": "mycosmosdbdatasource",
        "type": "cosmosdb",
        "credentials": {
            "connectionString": "AccountEndpoint=https://myCosmosDbEndpoint.documents.azure.com;AccountKey=myCosmosDbAuthKey;Database=myCosmosDbDatabaseId"
        },
        "container": { "name": "myCosmosDbCollectionId" },
        "dataChangeDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
            "highWaterMarkColumnName": "_ts"
        },
        "dataDeletionDetectionPolicy": {
            "@odata.type": "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
            "softDeleteColumnName": "isDeleted",
            "softDeleteMarkerValue": "true"
        }
    }

## <a name="NextSteps"></a>Pasos siguientes

Felicidades. En este artículo, aprendió a integrar Azure Cosmos DB con Azure Search con un indexador.

* Para más información acerca de Azure Cosmos DB, consulte la [página de servicio de Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).
* Para obtener más información sobre Azure Search, consulte la [página del servicio Search](https://azure.microsoft.com/services/search/).
