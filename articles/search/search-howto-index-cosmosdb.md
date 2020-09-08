---
title: Búsqueda en datos de Azure Cosmos DB
titleSuffix: Azure Cognitive Search
description: Importe datos de Azure Cosmos DB a un índice que permite búsquedas en Azure Cognitive Search. Los indexadores automatizan la ingesta de datos para orígenes de datos seleccionados, como Azure Cosmos DB.
author: mgottein
manager: nitinme
ms.author: magottei
ms.devlang: rest-api
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 07/11/2020
ms.openlocfilehash: db6dfb36c579f57f9cef66fa00a07b0d1dc2bc03
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88929676"
---
# <a name="how-to-index-cosmos-db-data-using-an-indexer-in-azure-cognitive-search"></a>Indexación de datos de Cosmos DB mediante un indizador en Azure Cognitive Search 

> [!IMPORTANT] 
> SQL API está disponible con carácter general.
> MongoDB API, Gremlin API y Cassandra API se encuentran actualmente en versión preliminar pública. La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Para solicitar acceso a las versiones preliminares, rellene [este formulario](https://aka.ms/azure-cognitive-search/indexer-preview). En la [API REST, versión 2020-06-30-Preview](search-api-preview.md) se proporcionan características en vista previa (GB). Actualmente hay compatibilidad limitada con el portal y no la hay con el SDK de .NET.

> [!WARNING]
> Solo las colecciones de Cosmos DB con una [directiva de indexación](/azure/cosmos-db/index-policy) establecida en [Coherente](/azure/cosmos-db/index-policy#indexing-mode) son compatibles con Azure Cognitive Search. La indexación de colecciones con una directiva de indexación diferida no se recomienda y puede dar lugar a la pérdida de datos. No se admiten las colecciones con indexación deshabilitada.

En este artículo se muestra cómo configurar un [indizador](search-indexer-overview.md) de Azure Cosmos DB para extraer contenido y que se pueda buscar en Azure Cognitive Search. Este flujo de trabajo crea un índice de Azure Cognitive Search y lo carga con texto existente extraído de Azure Cosmos DB. 

Como la terminología puede resultar confusa, merece la pena mencionar que la [indexación de Azure Cosmos DB](/azure/cosmos-db/index-overview) y la [indexación de Azure Cognitive Search](search-what-is-an-index.md) son operaciones distintas, únicas para cada servicio. Antes de comenzar la indexación de Azure Cognitive Search, la base de datos Azure Cosmos DB ya debe existir y contener datos.

El indexador de Cosmos DB en Azure Cognitive Search puede rastrear [elementos de Azure Cosmos DB](../cosmos-db/databases-containers-items.md#azure-cosmos-items) a los que se ha accedido a través de distintos protocolos. 

+ Para [SQL API](../cosmos-db/sql-query-getting-started.md), que está disponible con carácter general, puede usar el [portal](#cosmos-indexer-portal), la [API REST](/rest/api/searchservice/indexer-operations) o el [SDK de .NET](/dotnet/api/microsoft.azure.search.models.indexer?view=azure-dotnet) para crear el origen de datos y el indexador.

+ Para [MongoDB API (versión preliminar)](../cosmos-db/mongodb-introduction.md) puede usar el [portal](#cosmos-indexer-portal) o la [API REST, versión 2020-06-30-Preview](search-api-preview.md) para crear el origen de datos y el indexador.

+ Para [Cassandra API (versión preliminar)](../cosmos-db/cassandra-introduction.md) y [Gremlin API (versión preliminar)](../cosmos-db/graph-introduction.md), puede usar solo la [API REST, versión 2020-06-30-Preview](search-api-preview.md) para crear el origen de datos y el indexador.


> [!Note]
> En UserVoice puede votar por [Table API](https://feedback.azure.com/forums/263029-azure-search/suggestions/32759746-azure-search-should-be-able-to-index-cosmos-db-tab) si desea que se admita en Azure Cognitive Search.
>

<a name="cosmos-indexer-portal"></a>

## <a name="use-the-portal"></a>Uso del portal

> [!Note]
> El portal admite actualmente SQL API y MongoDB API (versión preliminar).

El método más sencillo para indexar elementos de Azure Cosmos DB es usar un asistente en [Azure Portal](https://portal.azure.com/). Mediante el muestre de datos y la lectura de metadatos en el contenedor, el asistente [**Importar datos**](search-import-data-portal.md) de Azure Cognitive Search puede crear un índice predeterminado, asignar campos de origen a campos de índice de destino y cargar el índice en una sola operación. Según el tamaño y la complejidad del origen de datos, puede tener un índice de búsqueda de texto completo y operativo en cuestión de minutos.

Se recomienda usar la misma región o ubicación para Azure Cognitive Search y Azure Cosmos DB a fin de conseguir una menor latencia y evitar cargos por ancho de banda.

### <a name="1---prepare-source-data"></a>1: Preparación de los datos de origen

Debe tener una cuenta de Cosmos DB, una base de datos de Azure Cosmos DB asignada a SQL API, MongoDB API (versión preliminar) o Gremlin API (versión preliminar) y el contenido de la base de datos.

Asegúrese de que la base de datos de Cosmos DB contiene datos. El [Asistente para importar datos](search-import-data-portal.md) lee los metadatos y realiza el muestreo de datos para inferir un esquema de índice, pero también carga los datos de Cosmos DB. Si los datos no están presentes, el asistente se detiene con este error "Error al detectar el esquema de índice desde el origen de datos: No se pudo crear un índice prototipo porque el origen de datos 'emptycollection' no devolvió datos".

### <a name="2---start-import-data-wizard"></a>2: Inicio del asistente para la importación de datos

También puede [iniciar el asistente](search-import-data-portal.md) desde la barra de comandos de la página del servicio Azure Cognitive Search, o bien, si se va a conectar a SQL API de Cosmos DB, puede hacer clic en **Add Azure Cognitive Search** (Agregar Azure Cognitive Search) en la sección **Configuración** del panel de navegación izquierdo de la cuenta de Cosmos DB.

   ![Comando de importación de datos en el portal](./media/search-import-data-portal/import-data-cmd2.png "Inicio del Asistente para la importación de datos")

### <a name="3---set-the-data-source"></a>3: Configuración del origen de datos

En la página **origen de datos**, el origen debe ser **Cosmos DB** con las especificaciones siguientes:

+ **Nombre** es el nombre del objeto de origen de datos. Una vez creado, puede elegirlo para otras cargas de trabajo.

+ **Cuenta de Cosmos DB** debe ser la cadena de conexión principal o secundaria de Cosmos DB, con el siguiente formato: `AccountEndpoint=https://<Cosmos DB account name>.documents.azure.com;AccountKey=<Cosmos DB auth key>;`.
    + En el caso de las **colecciones de MongoDB** de las versiones 3.2 y 3.6 use el formato siguiente para la cuenta de Cosmos DB en Azure Portal: `AccountEndpoint=https://<Cosmos DB account name>.documents.azure.com;AccountKey=<Cosmos DB auth key>;ApiKind=MongoDb`.
    + En el caso de los **gráficos de Gremlin y las tablas de Cassandra**, regístrese para la [versión preliminar del indexador validada](https://aka.ms/azure-cognitive-search/indexer-preview) para obtener acceso a la versión preliminar e información sobre cómo dar formato a las credenciales.

+ **Base de datos** es una base de datos existente de la cuenta. 

+ **Colección** es un contenedor de documentos. Deben existir los documentos para que la importación se realice correctamente. 

+ **Consulta** puede estar en blanco si desea ver todos los documentos; en caso contrario, escriba una consulta que seleccione un subconjunto de documentos. **Consulta** solo está disponible para SQL API.

   ![Definición del origen de datos de Cosmos DB](media/search-howto-index-cosmosdb/cosmosdb-datasource.png "Definición del origen de datos de Cosmos DB")

### <a name="4---skip-the-enrich-content-page-in-the-wizard"></a>4: omitir la página"Enriquecer contenido" del asistente

Agregar habilidades cognitivas (o enriquecimiento) no es un requisito de importación. A menos que tenga una necesidad específica de [agregar enriquecimiento de inteligencia artificial](cognitive-search-concept-intro.md) a su canalización de indexación, debe omitir este paso.

Para omitir el paso, haga clic en los botones azules en la parte inferior de la página para "Siguiente" y "Omitir".

### <a name="5---set-index-attributes"></a>5: Configuración de los atributos de índice

En la página de **Índice**, debería mostrarse una lista de campos con un tipo de datos y una serie de casillas de verificación para configurar los atributos del índice. El asistente puede generar una lista de campos basada en metadatos y mediante el muestreo de los datos de origen. 

Puede seleccionar atributos de forma masiva si activa la casilla situada en la parte superior de una columna de atributos. Elija **Se puede recuperar** y **Se puede buscar** para cada campo que se debe devolver a una aplicación cliente y está sujeto a un proceso de búsqueda de texto completo. Observará que los enteros no permiten búsquedas de texto completo o de texto parcial (los números se evalúan literalmente y suelen ser útiles en filtros).

Revise la descripción de [atributos de índice](/rest/api/searchservice/create-index#bkmk_indexAttrib) y [analizadores de lenguaje](/rest/api/searchservice/language-support) para más información. 

Dedique un momento a la revisión de las selecciones. Una vez que se ejecuta al asistente, se crean las estructuras de datos físicas y no podrá modificar estos campos sin quitar y volver a crear todos los objetos.

   ![Definición de índice de Cosmos DB](media/search-howto-index-cosmosdb/cosmosdb-index-schema.png "Definición de índice de Cosmos DB")

### <a name="6---create-indexer"></a>6: Creación del indizador

Con todas las especificaciones agregadas, el asistente crea tres objetos distintos en el servicio de búsqueda. Un objeto de origen de datos y un objeto de índice se guardan como recursos con nombre en el servicio Azure Cognitive Search. El último paso crea un objeto de indizador. Al asignarle un nombre al indizador, este puede existir como un recurso independiente que se puede programar y administrar independientemente de los objetos de origen de datos y del de índice que se crearon en la misma secuencia del asistente.

Si no está familiarizado con los indexadores, un *indexador* es un recurso en Azure Cognitive Search que rastrea un origen de datos externo en busca de contenido utilizable en búsquedas. La salida del asistente **Importar datos** es un indizador que se rastrea el origen de datos de Cosmos DB, extrae el contenido utilizable en búsquedas y lo importa a un índice en Azure Cognitive Search.

En la siguiente captura se muestra la configuración del indexador predeterminado. Puede cambiar a **Una vez** si desea ejecutar el indexador una vez. Haga clic en **Enviar** para ejecutar el asistente y crear todos los objetos. La indexación comienza inmediatamente.

   ![Definición del indizador de Cosmos DB](media/search-howto-index-cosmosdb/cosmosdb-indexer.png "Definición del indizador de Cosmos DB")

Puede supervisar la importación de datos en las páginas del portal. Las notificaciones de progreso indican el estado de la indexación y cuántos documentos se cargan. 

Cuando se complete la indización, puede usar el [Explorador de búsqueda](search-explorer.md) para consultar el índice.

> [!NOTE]
> Si no se muestran los datos esperados, tiene que establecer más atributos en más campos. Elimine el índice y el indexador que acaba de crear y vuelva a seguir los pasos del asistente, pero esta vez cambie las opciones elegidas para los atributos de índice del paso 5. 

<a name="cosmosdb-indexer-rest"></a>

## <a name="use-rest-apis"></a>Uso de API REST

Puede usar la API REST para indexar datos de Azure Cosmos DB, siguiendo un flujo de trabajo de tres partes común a todos los indizadores en Azure Cognitive Search: se crea un origen de datos, se crea un índice y se crea un indizador. La extracción de datos de Cosmos DB se produce cuando se envía la solicitud Create Indexer. Una vez finalizada esta solicitud, tendrá un índice que permite consultas. 

> [!NOTE]
> Para indexar datos de Gremlin API de Cosmos DB o Cassandra API de Cosmos DB antes debe solicitar acceso a las versiones preliminares validadas, para lo que debe rellenar [este formulario](https://aka.ms/azure-cognitive-search/indexer-preview). Una vez que se procese la solicitud, recibirá instrucciones para usar la [API REST, version 2020-06-30-Preview,](search-api-preview.md) para crear el origen de datos.

En este artículo ya se ha mencionado que la [indexación de Azure Cosmos DB](/azure/cosmos-db/index-overview) y la [indexación de Azure Cognitive Search](search-what-is-an-index.md) son operaciones distintas. En el caso de la indexación de Cosmos DB, de forma predeterminada, todos los documentos se indexan automáticamente, excepto con Cassandra API. Si se desactiva la indexación automática, solo se puede acceder a los documentos mediante sus propios vínculos o mediante consultas con el identificador del documento. La indexación de Azure Cognitive Search requiere que se active la indexación automática de Cosmos DB en la colección que Azure Cognitive Search va a indexar. Al registrarse para la versión preliminar del indexador de Cassandra API de Cosmos DB, se le proporcionarán las instrucciones necesarias para configurar la indexación de Cosmos DB.

> [!WARNING]
> Azure Cosmos DB es la siguiente generación de DocumentDB. Anteriormente con la versión de API **2017-11-11** se podría utilizar la sintaxis `documentdb`. Esto significaba que puede especificar el tipo de origen de datos como `cosmosdb` o `documentdb`. A partir de la versión **2019-05-06** de la API, tanto las API de Azure Cognitive Search como el portal solo admiten la sintaxis `cosmosdb` tal y como se indica en este artículo. Esto significa que el tipo de origen de datos debe `cosmosdb` si desea conectarse a un punto de conexión de Cosmos DB.

### <a name="1---assemble-inputs-for-the-request"></a>1\. Ensamblado de las entradas para la solicitud

Para cada solicitud, debe proporcionar el nombre del servicio y la clave de administrador de Azure Cognitive Search (en el encabezado POST) y el nombre de cuenta de almacenamiento y la clave para Blob Storage. Puede usar [Postman](search-get-started-postman.md) para enviar solicitudes HTTP a Azure Cognitive Search.

Copie los cuatro valores siguientes en el Bloc de notas para poder pegarlos en una solicitud:

+ El nombre del servicio Azure Cognitive Search
+ La clave de administrador de Azure Cognitive Search
+ Cadena de conexión de Cosmos DB

Encontrará estos valores en el portal:

1. En las páginas del portal de Azure Cognitive Search, copie la URL del servicio de búsqueda desde la página de información general.

2. En el panel de navegación izquierdo, haga clic en **Claves** y, luego, copie la clave primaria o la secundaria (son equivalentes).

3. Cambie a las páginas del portal para la cuenta de almacenamiento de Cosmos. En el panel de navegación de la izquierda, en **Configuración**, haga clic en **Claves**. En esta página se proporciona un URI, dos conjuntos de cadenas de conexión y dos conjuntos de claves. En el Bloc de notas, copie una de las cadenas de conexión.

### <a name="2---create-a-data-source"></a>2\. Crear un origen de datos

A **origen de datos** especifica los datos para indexar, las credenciales y las directivas para identificar cambios en los datos (por ejemplo, los documentos modificados o eliminados dentro de la colección). El origen de datos se define como un recurso independiente para que puedan usarlo múltiples indizadores.

Para crear un origen de datos, formule una solicitud POST:

```http

    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
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
```

El cuerpo de la solicitud contiene la definición del origen de datos, que debe incluir los siguientes campos:

| Campo   | Descripción |
|---------|-------------|
| **name** | Necesario. Elija un nombre para representar el objeto de origen de datos. |
|**type**| Necesario. Debe ser `cosmosdb`. |
|**credentials** | Necesario. Debe ser una cadena de conexión de Cosmos DB.<br/><br/>Para las **colecciones de SQL**, las cadenas de conexión están en este formato: `AccountEndpoint=https://<Cosmos DB account name>.documents.azure.com;AccountKey=<Cosmos DB auth key>;Database=<Cosmos DB database id>`.<br/><br/>En el caso de las **colecciones de MongoDB** de las versiones 3.2 y 3.6 use el formato siguiente para la cadena de conexión: `AccountEndpoint=https://<Cosmos DB account name>.documents.azure.com;AccountKey=<Cosmos DB auth key>;Database=<Cosmos DB database id>;ApiKind=MongoDb`.<br/><br/>En el caso de los **gráficos de Gremlin y las tablas de Cassandra**, regístrese para la [versión preliminar del indexador validada](https://aka.ms/azure-cognitive-search/indexer-preview) para obtener acceso a la versión preliminar e información sobre cómo dar formato a las credenciales.<br/><br/>Evite los números de puerto en la dirección URL del punto de conexión. Si incluye el número de puerto, Azure Cognitive Search no podrá indexar la base de datos de Azure Cosmos DB.|
| **container** | Contiene los siguientes elementos: <br/>**name**: Necesario. Especifique el identificador de la colección de la base de datos que se va a indexar.<br/>**query**: Opcional. Puede especificar una consulta para acoplar un documento JSON arbitrario en un esquema plano que Azure Cognitive Search pueda indizar.<br/>En MongoDB API, Gremlin API y Cassandra API, no se admiten consultas. |
| **dataChangeDetectionPolicy** | Se recomienda su uso. Consulte la sección [Indexación de documentos modificados](#DataChangeDetectionPolicy).|
|**dataDeletionDetectionPolicy** | Opcional. Consulte la sección [Indexación de documentos eliminados](#DataDeletionDetectionPolicy).|

### <a name="using-queries-to-shape-indexed-data"></a>Uso de consultas para dar forma a los datos indizados
Puede especificar una consulta de SQL para eliminar el formato de las propiedades o matrices anidadas y de las propiedades JSON del proyecto, y para filtrar los datos que se van a indexar. 

> [!WARNING]
> No se admiten consultas personalizadas en la **MongoDB API**, **Gremlin API** y **Cassandra API**: el parámetro `container.query` debe establecerse en null u omitirse. Si tiene que usar una consulta personalizada, háganoslo saber en [User Voice](https://feedback.azure.com/forums/263029-azure-search).

Documento de ejemplo:

```http
    {
        "userId": 10001,
        "contact": {
            "firstName": "andy",
            "lastName": "hoh"
        },
        "company": "microsoft",
        "tags": ["azure", "cosmosdb", "search"]
    }
```

Consulta de filtro:

```sql
SELECT * FROM c WHERE c.company = "microsoft" and c._ts >= @HighWaterMark ORDER BY c._ts
```

Consulta sin formato:

```sql
SELECT c.id, c.userId, c.contact.firstName, c.contact.lastName, c.company, c._ts FROM c WHERE c._ts >= @HighWaterMark ORDER BY c._ts
```

Consulta de proyección:

```sql
SELECT VALUE { "id":c.id, "Name":c.contact.firstName, "Company":c.company, "_ts":c._ts } FROM c WHERE c._ts >= @HighWaterMark ORDER BY c._ts
```

Consulta sin formato de matriz:

```sql
SELECT c.id, c.userId, tag, c._ts FROM c JOIN tag IN c.tags WHERE c._ts >= @HighWaterMark ORDER BY c._ts
```

### <a name="3---create-a-target-search-index"></a>3\. Crear un índice de búsqueda de destino 

Si aún no tiene un índice de [Azure Cognitive Search](/rest/api/searchservice/create-index) de destino, créelo. En el ejemplo siguiente se crea un índice con un campo de identificador y de descripción:

```http
    POST https://[service name].search.windows.net/indexes?api-version=2020-06-30
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
```

Asegúrese de que el esquema del índice de destino es compatible con el de los documentos JSON de origen o el resultado de la proyección de consultas personalizada.

> [!NOTE]
> En el caso de las colecciones con particiones, la clave de documento predeterminada es la propiedad `_rid` de Azure Cosmos DB, a la que Azure Cognitive Search cambia el nombre automáticamente a `rid`, porque los nombres de los campos no pueden comenzar por un carácter de guion bajo. Además, los valores `_rid` de Azure Cosmos DB contienen caracteres que no son válidos en las claves de Azure Cognitive Search. Por este motivo, la `_rid` valores están codificados con Base64.
> 
> Para las colecciones de MongoDB, Azure Cognitive Search cambia el nombre de la propiedad `_id` automáticamente a `id`.  

### <a name="mapping-between-json-data-types-and-azure-cognitive-search-data-types"></a>Asignación entre tipos de datos de JSON y de Azure Cognitive Search
| Tipo de datos JSON | Tipos de campos de índice de destino compatibles |
| --- | --- |
| Bool |Edm.Boolean, Edm.String |
| Números que parecen enteros |Edm.Int32, Edm.Int64, Edm.String |
| Números que parecen puntos flotantes |Edm.Double, Edm.String |
| String |Edm.String |
| Matrices de tipos primitivos, por ejemplo ["a", "b", "c"] |Collection(Edm.String) |
| Cadenas que parecen fechas |Edm.DateTimeOffset, Edm.String |
| Objetos GeoJSON, por ejemplo {"type": "Point", "coordinates": [long, lat] } |Edm.GeographyPoint |
| Otros objetos JSON |N/D |

### <a name="4---configure-and-run-the-indexer"></a>4\. Configurar y ejecutar el indexador

Una vez creados el origen de datos y los índices, ya podrá crear el indizador:

```http
    POST https://[service name].search.windows.net/indexers?api-version=2020-06-30
    Content-Type: application/json
    api-key: [admin key]

    {
      "name" : "mycosmosdbindexer",
      "dataSourceName" : "mycosmosdbdatasource",
      "targetIndexName" : "mysearchindex",
      "schedule" : { "interval" : "PT2H" }
    }
```

Este indexador se ejecuta cada dos horas (el intervalo de programación se establece en "PT2H"). Para ejecutar un indizador cada 30 minutos, establézcalo en PT30M. El intervalo más breve que se admite es de 5 minutos. La programación es opcional: si se omite, el indizador solo se ejecuta una vez cuando se crea. Sin embargo, puede ejecutarlo a petición en cualquier momento.   

Para más información sobre la API Create Indexer, consulte [Crear indexador](/rest/api/searchservice/create-indexer).

Para más información sobre cómo definir las programaciones del indexador, consulte [Programación de indexadores para Azure Cognitive Search](search-howto-schedule-indexers.md).

## <a name="use-net"></a>Uso de .NET

El SDK de.NET generalmente disponible tiene plena paridad con la API REST disponible con carácter general. Se recomienda que revise la sección anterior de la API REST para obtener información sobre los conceptos, el flujo de trabajo y los requisitos. A continuación, puede consultar la siguiente documentación de referencia de la API de .NET para implementar un indizador JSON en código administrado.

+ [Microsoft.Azure.Search.Models.DataSource](/dotnet/api/microsoft.azure.search.models.datasource?view=azure-dotnet)
+ [Microsoft.azure.search.models.datasourcetype](/dotnet/api/microsoft.azure.search.models.datasourcetype?view=azure-dotnet) 
+ [Microsoft.azure.search.models.index](/dotnet/api/microsoft.azure.search.models.index?view=azure-dotnet) 
+ [Microsoft.azure.search.models.indexer](/dotnet/api/microsoft.azure.search.models.indexer?view=azure-dotnet)

<a name="DataChangeDetectionPolicy"></a>

## <a name="indexing-changed-documents"></a>Indexación de documentos modificados

El fin de una directiva de detección de cambios de datos es identificar de forma eficaz los elementos de datos que han cambiado. Actualmente, la única directiva compatible es [`HighWaterMarkChangeDetectionPolicy`](/dotnet/api/microsoft.azure.search.models.highwatermarkchangedetectionpolicy) que usa la propiedad `_ts` (marca de tiempo) que proporciona Azure Cosmos DB, y se especifica de la siguiente forma:

```http
    {
        "@odata.type" : "#Microsoft.Azure.Search.HighWaterMarkChangeDetectionPolicy",
        "highWaterMarkColumnName" : "_ts"
    }
```

Se recomienda encarecidamente usar esta directiva para garantizar un buen rendimiento del indexador. 

Si usa una consulta personalizada, asegúrese de que la consulta proyecta la propiedad `_ts`.

<a name="IncrementalProgress"></a>

### <a name="incremental-progress-and-custom-queries"></a>Progreso incremental y consultas personalizadas

El progreso incremental durante la indexación garantiza que si se interrumpe la ejecución del indexador por errores transitorios o por el límite de tiempo de ejecución, dicho indexador puede retomar la ejecución por donde la dejó la próxima vez que se ejecute, en lugar de tener que volver a indexar toda la colección desde el principio. Esto es especialmente importante cuando se indexan colecciones grandes. 

Para habilitar el progreso incremental cuando se usa una consulta personalizada, asegúrese de que la consulta ordena los resultados por la columna `_ts`. Esto permite la creación de puntos de comprobación, que Azure Cognitive Search usa para proporcionar el progreso incremental en presencia de errores.   

En algunos casos, incluso si la consulta contiene una cláusula `ORDER BY [collection alias]._ts`, Azure Cognitive Search puede no deducir que la consulta se ordene por `_ts`. Puede indicar a Azure Cognitive Search que los resultados se ordenen mediante el uso de la propiedad de configuración `assumeOrderByHighWaterMarkColumn`. Para especificar esta sugerencia, cree o actualice el indexador como se indica a continuación: 

```http
    {
     ... other indexer definition properties
     "parameters" : {
            "configuration" : { "assumeOrderByHighWaterMarkColumn" : true } }
    } 
```

<a name="DataDeletionDetectionPolicy"></a>

## <a name="indexing-deleted-documents"></a>Indexación de documentos eliminados

Cuando se eliminan filas de la recopilación, lo habitual es que también se deseen eliminar del índice de búsqueda. El fin de una directiva de detección de eliminación de datos es identificar eficazmente los elementos de datos eliminados. Actualmente, solo es compatible la directiva `Soft Delete` (la eliminación se indica con algún tipo de marca), especificada de la siguiente forma:

```http
    {
        "@odata.type" : "#Microsoft.Azure.Search.SoftDeleteColumnDeletionDetectionPolicy",
        "softDeleteColumnName" : "the property that specifies whether a document was deleted",
        "softDeleteMarkerValue" : "the value that identifies a document as deleted"
    }
```

Si usa una consulta personalizada, asegúrese de que la consulta proyecta la propiedad a la que `softDeleteColumnName` hace referencia.

En el ejemplo siguiente se crea un origen de datos con una directiva de eliminación temporal:

```http
    POST https://[service name].search.windows.net/datasources?api-version=2020-06-30
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
```

## <a name="next-steps"></a><a name="NextSteps"></a>Pasos siguientes

Felicidades. En este artículo, aprendió a integrar Azure Cosmos DB con Azure Cognitive Search con un indizador.

* Para más información acerca de Azure Cosmos DB, consulte la [página de servicio de Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/).
* Para obtener más información sobre Azure Cognitive Search, consulte la [página del servicio Search](https://azure.microsoft.com/services/search/).