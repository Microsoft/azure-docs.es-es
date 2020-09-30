---
title: Indexadores para rastrear datos durante la importación
titleSuffix: Azure Cognitive Search
description: Rastree Azure SQL Database, SQL Managed Instance, Azure Cosmos DB o Azure Storage para extraer los datos utilizables en búsquedas y rellenar un índice de Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/22/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: 3f3c53d412105489a814f082bbc0fd209476ecf9
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90971398"
---
# <a name="indexers-in-azure-cognitive-search"></a>Indexadores de Azure Cognitive Search

Un *indexador* de Azure Cognitive Search es un rastreador (crawler) que extrae datos y metadatos utilizables en búsquedas de un origen de datos de Azure externo y rellena un índice basado en las asignaciones de un campo a otro entre el índice y su origen de datos. Este enfoque se denomina a veces "modelo de extracción", porque el servicio extrae datos sin que sea preciso escribir código que agregue datos en un índice.

Los indexadores se basan en tipos de orígenes de datos o plataformas, y hay un indexador individual para SQL Server en Azure, Cosmos DB, Azure Table Storage y Blob Storage. Los indexadores de Blob Storage tienen propiedades adicionales específicas de tipos de contenido del blob.

Puede utilizar un indexador como único medio para la ingesta de datos. o bien utilizar una combinación de técnicas que incluyan el uso de un indexador para cargar solo algunos de los campos en el índice.

Puede ejecutar los indexadores a petición o con una programación de actualización periódica de datos que se ejecuta con una frecuencia de incluso cada cinco minutos. Las actualizaciones más frecuentes requieren un modelo de inserción que actualice simultáneamente los datos en Azure Cognitive Search y su origen de datos externo.

## <a name="approaches-for-creating-and-managing-indexers"></a>Enfoques para crear y administrar indexadores

Puede crear y administrar indexadores mediante estos enfoques:

* [Portal > Asistente para la importación de datos](search-import-data-portal.md)
* [API de REST de servicio](/rest/api/searchservice/Indexer-operations)
* [SDK de .NET](/dotnet/api/microsoft.azure.search.iindexersoperations)

Inicialmente, un nuevo indexador se anuncia como una característica en versión preliminar. Las características en versión preliminar se introdujeron en las API (REST y. NET) y luego se integraron en el portal después de aprobarse para disponibilidad general. Si va a evaluar un nuevo indexador, debe pensar en escribir código.

## <a name="permissions"></a>Permisos

Todas las operaciones relacionadas con los indexadores, incluidas las solicitudes GET para estado o definiciones, requieren una [clave de API de administración](search-security-api-keys.md).

<a name="supported-data-sources"></a>

## <a name="supported-data-sources"></a>Orígenes de datos admitidos

Los indexadores rastrean los almacenes de datos en Azure.

* [Azure Blob Storage](search-howto-indexing-azure-blob-storage.md)
* [Azure Data Lake Storage Gen2](search-howto-index-azure-data-lake-storage.md) (en versión preliminar)
* [Azure Table Storage](search-howto-indexing-azure-tables.md)
* [Azure Cosmos DB](search-howto-index-cosmosdb.md)
* [Azure SQL Database](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)
* [Instancia administrada de SQL](search-howto-connecting-azure-sql-mi-to-azure-search-using-indexers.md)
* [SQL Server en Azure Virtual Machines](search-howto-connecting-azure-sql-iaas-to-azure-search-using-indexers.md)

## <a name="indexer-stages"></a>Fases del indexador

En una ejecución inicial, cuando el índice está vacío, un indexador lee todos los datos proporcionados en la tabla o el contenedor. En las ejecuciones posteriores, el indexador normalmente puede detectar y recuperar solo los datos que han cambiado. En el caso de los datos de blob, la detección de cambios es automática. En otros orígenes de datos, como Azure SQL o Cosmos DB, la detección de cambios se debe habilitar.

En cada uno de los documentos que ingiere, un indexador implementa o coordina varios pasos, desde la recuperación del documento hasta una "entrega" final del motor de búsqueda para la indexación. Opcionalmente, un indexador también es fundamental para impulsar la ejecución y los resultados del conjunto de aptitudes, siempre que se haya definido uno.

![Fases del indexador](./media/search-indexer-overview/indexer-stages.png "fases del indexador")

### <a name="stage-1-document-cracking"></a>Fase 1 Descifrado de documentos

El descifrado de documentos es el proceso de abrir archivos y extraer contenido. Según el tipo de origen de datos, el indexador intenta realizar diferentes operaciones para extraer contenido que posiblemente se pueda indexar.  

Ejemplos:  

* Si el documento es un registro de un [origen de datos de Azure SQL](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md), el indexador extrae cada uno de los campos del registro.
* Si el documento es un archivo PDF de un [origen de datos de Azure Blob Storage](search-howto-indexing-azure-blob-storage.md), el indexador extrae el texto, las imágenes y los metadatos del archivo.
* Si el documento es un registro de un [origen de datos de Cosmos DB](search-howto-index-cosmosdb.md), el indexador extrae los campos y subcampos del documento de Cosmos DB.

### <a name="stage-2-field-mappings"></a>Fase 2: Asignaciones de campos 

Un indexador extrae texto de un campo de origen y lo envía a un campo de destino de un índice o un almacén de conocimiento. Si los nombres de los campos y los tipos coinciden, la ruta de acceso está clara. Pero es posible que quiera nombres o tipos diferentes en el resultado, en cuyo caso debe indicar al indexador cómo asignar el campo. Este paso se produce después del descifrado de documentos, pero antes de las transformaciones, cuando el indexador lee los documentos de origen. Al definir una [asignación de campos](search-indexer-field-mappings.md), el valor del campo de origen se envía tal cual al campo de destino, sin modificaciones. Las asignaciones de campos son opcionales.

### <a name="stage-3-skillset-execution"></a>Fase 3: Ejecución del conjunto de aptitudes

La ejecución del conjunto de aptitudes es un paso opcional que invoca al procesamiento de IA integrado o personalizado. Es posible que lo necesite para el reconocimiento óptico de caracteres (OCR) en forma de análisis de imágenes, o puede que necesite traducción de idiomas. Sea cual sea la transformación, la ejecución del conjunto de aptitudes es donde se produce el enriquecimiento. Si un indexador es una canalización, puede imaginarse un [conjunto de aptitudes](cognitive-search-defining-skillset.md) como una "canalización dentro de la canalización". Un conjunto de aptitudes tiene su propia secuencia de pasos, denominados aptitudes.

### <a name="stage-4-output-field-mappings"></a>Fase 4: Asignaciones de campos de salida

El resultado de un conjunto de aptitudes es en realidad un árbol de información denominado documento enriquecido. Las asignaciones de campos de salida permiten seleccionar qué partes de este árbol se asignan a campos del índice. Aprenda a [definir asignaciones de campos de salida](cognitive-search-output-field-mapping.md).

Al igual que las asignaciones de campos que asocian valores textuales de campos de origen a campos de destino, las asignaciones de campos de salida indican al indexador cómo asociar los valores transformados del documento enriquecido a los campos de destino del índice. A diferencia de las asignaciones de campos, que se consideran opcionales, siempre debe definir una asignación de campos de salida para cualquier contenido transformado que deba residir en un índice.

En la siguiente imagen se muestra una representación de [sesión de depuración](cognitive-search-debug-session.md) de un indexador de ejemplo de las fases del indexador: descifrado de documentos, asignaciones de campos, ejecución del conjunto de aptitudes y asignaciones de campos de salida.

:::image type="content" source="media/search-indexer-overview/sample-debug-session.png" alt-text="Sesión de depuración de ejemplo" lightbox="media/search-indexer-overview/sample-debug-session.png":::

## <a name="basic-configuration-steps"></a>Pasos básicos de configuración

Los indexadores pueden ofrecer características que son exclusivas del origen de datos. En este sentido, algunos aspectos de la configuración de orígenes de datos o indexadores varían según el tipo de indexador. No obstante, todos los indexadores comparten composición básica y requisitos. Más adelante, se explican los pasos que son comunes a todos los indexadores.

### <a name="step-1-create-a-data-source"></a>Paso 1: Creación de un origen de datos
Un indexador obtiene la conexión de origen de datos desde un objeto de *origen de datos*. La definición del origen de datos proporciona una cadena de conexión y, posiblemente, las credenciales. Para crear el recurso, llame a la API de REST [Create Datasource](/rest/api/searchservice/create-data-source) o [clase DataSource](/dotnet/api/microsoft.azure.search.models.datasource).

Los orígenes de datos se configuran y administran independientemente de los indexadores que los usan, lo que significa que varios indexadores pueden usar un origen de datos para cargar más de un índice a la vez.

### <a name="step-2-create-an-index"></a>Paso 2: Creación de un índice
Un indexador automatizará algunas tareas relacionadas con la ingesta de datos, pero la creación de un índice no suele ser una de ellas. Como requisito previo, debe tener un índice predefinido con campos que coincidan con los del origen de datos externo. Los campos deben coincidir por nombre y tipo de datos. Para más información sobre la estructura de un índice, consulte [Create an Index (Azure Cognitive Search REST API)](/rest/api/searchservice/Create-Index) [Creación de un índice (API de REST de Azure Cognitive Search)] o [Index class](/dotnet/api/microsoft.azure.search.models.index) (Clase Index). Para obtener ayuda con las asociaciones de campo, consulte [Field mappings in Azure Cognitive Search indexers](search-indexer-field-mappings.md) (Asignaciones de campos en los indexadores de Azure Cognitive Search).

> [!Tip]
> Aunque los indexadores no pueden generar un índice, el Asistente para **importar datos** del portal puede ayudarle. En la mayoría de los casos, el asistente puede inferir un esquema de índice de los metadatos existentes en el origen y presentar un esquema de índice preliminar que se pueda modificar en línea mientras el asistente esté activo. Una vez que el índice se crea en el servicio, la mayoría de las posteriores modificaciones se limitan a agregar nuevos campos. Tenga en cuenta al asistente para crear índices, pero no para revisarlos. Para obtener conocimientos prácticos, recorra el [tutorial del portal](search-get-started-portal.md).

### <a name="step-3-create-and-schedule-the-indexer"></a>Paso 3: Creación y programación del indizador
La definición del indexador es una construcción que reúne todos los elementos relacionados con la ingesta de datos. Los elementos necesarios incluyen un origen de datos y un índice. Los elementos opcionales incluyen una programación y asignaciones de campos. La asignación de campos solo es opcional si los campos de origen y los campos de índice se corresponden claramente. Para más información sobre la estructura de un indexador, consulte [Create Indexer (Azure Search Service REST API)](/rest/api/searchservice/Create-Indexer) [Crear el indexador (API de REST de Azure Cognitive Search)].

<a id="RunIndexer"></a>

## <a name="run-indexers-on-demand"></a>Ejecutar indexadores a petición

Aunque es común para programar la indexación, un indexador también puede invocarse a petición mediante el [comando Ejecutar](/rest/api/searchservice/run-indexer):

```http
POST https://[service name].search.windows.net/indexers/[indexer name]/run?api-version=2020-06-30
api-key: [Search service admin key]
```

> [!NOTE]
> Cuando la API de Run se devuelve correctamente, se ha programado la invocación del indexador, pero el procesamiento real se produce de forma asincrónica. 

El estado del indexador se puede supervisar en el portal o mediante la API de Obtener el estado del indexador. 

<a name="GetIndexerStatus"></a>

## <a name="get-indexer-status"></a>Obtener el estado del indexador

Puede recuperar el estado y el historial de ejecución de un indexador a través del [comando Obtener estado del indexador](/rest/api/searchservice/get-indexer-status):

```http
GET https://[service name].search.windows.net/indexers/[indexer name]/status?api-version=2020-06-30
api-key: [Search service admin key]
```

La respuesta contiene el estado general del indexador, la última vez que se invocó al indexador (o en curso) y el historial de las recientes invocaciones al indexador.

```output
{
    "status":"running",
    "lastResult": {
        "status":"success",
        "errorMessage":null,
        "startTime":"2018-11-26T03:37:18.853Z",
        "endTime":"2018-11-26T03:37:19.012Z",
        "errors":[],
        "itemsProcessed":11,
        "itemsFailed":0,
        "initialTrackingState":null,
        "finalTrackingState":null
     },
    "executionHistory":[ {
        "status":"success",
         "errorMessage":null,
        "startTime":"2018-11-26T03:37:18.853Z",
        "endTime":"2018-11-26T03:37:19.012Z",
        "errors":[],
        "itemsProcessed":11,
        "itemsFailed":0,
        "initialTrackingState":null,
        "finalTrackingState":null
    }]
}
```

El historial de ejecución contiene como máximo las 50 ejecuciones completadas más recientemente en orden cronológico inverso (la ejecución más reciente aparece en primer lugar).

## <a name="next-steps"></a>Pasos siguientes
Ahora que tiene el concepto básico, el paso siguiente consiste en revisar los requisitos y las tareas específicos de cada tipo de origen de datos.

* [Azure SQL Database, SQL Managed Instance o SQL Server en una máquina virtual de Azure](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)
* [Azure Cosmos DB](search-howto-index-cosmosdb.md)
* [Azure Blob Storage](search-howto-indexing-azure-blob-storage.md)
* [Azure Table Storage](search-howto-indexing-azure-tables.md)
* [Indexación de blobs CSV con el indexador de blobs de Azure Cognitive Search](search-howto-index-csv-blobs.md)
* [Indexación de blobs JSON con el indexador de blobs de Azure Cognitive Search](search-howto-index-json-blobs.md)