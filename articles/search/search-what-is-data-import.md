---
title: Importación e ingesta de datos en índices de búsqueda
titleSuffix: Azure Cognitive Search
description: Rellene y cargue datos en un índice de Azure Cognitive Search desde orígenes de datos externos.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/30/2020
ms.openlocfilehash: 50c95dc9d045711cb6968b98957d255b4ca73d2c
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88932770"
---
# <a name="data-import-overview---azure-cognitive-search"></a>Introducción a la importación de datos: Azure Cognitive Search

En Azure Cognitive Search, las consultas se ejecutan sobre el contenido cargado y guardado en un [índice de búsqueda](search-what-is-an-index.md). Este artículo examina los dos enfoques básicos para rellenar un índice: *insertar* los datos en el índice mediante programación o apuntar un [indizador de Azure Cognitive Search](search-indexer-overview.md) en un origen de datos admitido para *extraer* en los datos.

En ambos métodos, el objetivo consiste en cargar datos de un origen de datos externo en un índice de Azure Cognitive Search. Azure Cognitive Search le permitirá crear un índice vacío, pero, hasta que se inserten o extraigan datos en él, no es consultable.

> [!NOTE]
> Si el [enriquecimiento con IA](cognitive-search-concept-intro.md) es un requisito de la solución, debe usar el modelo de extracción (indexadores) para cargar un índice. El procesamiento externo solo es posible a través de conjuntos de aptitudes conectados a un indexador.

## <a name="pushing-data-to-an-index"></a>Inserción de datos en un índice

El modelo de inserción, que se usa para enviar a Azure Cognitive Search los datos mediante programación, es el enfoque más flexible. En primer lugar, no tiene ninguna restricción en el tipo de origen de datos. Cualquier conjunto de datos que se compone de documentos JSON se puede insertar en un índice de Azure Cognitive Search si cada documento en el conjunto de datos tiene campos asignados a los campos definidos en el esquema de índice. En segundo lugar, no tiene ninguna restricción en la frecuencia de ejecución. Puede insertar los cambios a un índice tantas veces como desee. En el caso de las aplicaciones con requisitos de latencia muy baja (por ejemplo, si se necesita que las operaciones de búsqueda estén sincronizadas con las bases de datos dinámicas del inventario), la única opción es un modelo de inserción.

Este enfoque es más flexible que el modelo de extracción, ya que los documentos se pueden cargar individualmente o en lotes (hasta 1000 por lote o 16 MB, lo que ocurra primero). El modelo de inserción también le permite cargar documentos en Azure Cognitive Search independientemente de dónde están los datos.

### <a name="how-to-push-data-to-an-azure-cognitive-search-index"></a>Inserción de los datos en un índice de Azure Cognitive Search

Puede usar las siguientes API para cargar uno o varios documentos en un índice:

+ [Agregar, Actualizar o Eliminar documentos (API de REST)](/rest/api/searchservice/AddUpdate-or-Delete-Documents)
+ [Clase indexAction](/dotnet/api/microsoft.azure.search.models.indexaction?view=azure-dotnet) o [clase indexBatch](/dotnet/api/microsoft.azure.search.models.indexbatch?view=azure-dotnet) 

Actualmente no se admite ninguna herramienta para insertar datos mediante el portal.

Para obtener una introducción a cada metodología, consulte [Quickstart: Creación de un índice de Azure Cognitive Search mediante PowerShell](./search-get-started-powershell.md) o [Inicio rápido de C#: Creación de un índice de Azure Cognitive Search mediante el SDK para .NET](search-get-started-dotnet.md).

<a name="indexing-actions"></a>

### <a name="indexing-actions-upload-merge-mergeorupload-delete"></a>Acciones de indexación: carga, combinación, mergeOrUpload y eliminación

Puede controlar el tipo de acción de indexación por documento, especificando si se debe cargar el documento totalmente, combinarlo con el contenido del documento existente o eliminarlo.

En la API REST, emita solicitudes HTTP POST con cuerpos de solicitud JSON a la dirección URL del punto de conexión del índice de Azure Cognitive Search. Cada objeto JSON de la matriz "value" contiene la clave del documento y especifica si una acción de indexación debe incorporar, actualizar o eliminar contenido del documento. Para obtener un ejemplo de código, consulte [Carga de documentos](search-get-started-dotnet.md#load-documents).

En el SDK de .NET, empaquete los datos en un objeto `IndexBatch`. Un `IndexBatch` encapsula una colección de objetos `IndexAction`, cada uno de los cuales contiene un documento y una propiedad que indica a Azure Cognitive Search qué acción debe realizar en el documento. Para obtener un ejemplo de código, consulte el [Inicio rápido de C#](search-get-started-dotnet.md).


| @search.action | Descripción | Campos necesarios para cada documento | Notas |
| -------------- | ----------- | ---------------------------------- | ----- |
| `upload` |Una acción `upload` es similar a un "upsert" donde se insertará el documento si es nuevo y se actualizará/reemplazará si ya existe. |la clave, además de cualquier otro campo que desee definir |Al actualizar o reemplazar un documento existente, cualquier campo que no esté especificado en la solicitud tendrá su campo establecido en `null`. Esto ocurre incluso cuando el campo se ha establecido previamente en un valor que no sea nulo. |
| `merge` |Permite actualizar un documento existente con los campos especificados. Si el documento no existe en el índice, se producirá un error en la combinación. |la clave, además de cualquier otro campo que desee definir |Cualquier campo que se especifica en una combinación reemplazará al campo existente en el documento. En el SDK de .NET se incluyen los campos de tipo `DataType.Collection(DataType.String)`. En la API REST se incluyen los campos de tipo `Collection(Edm.String)`. Por ejemplo, si el documento contiene un campo `tags` con el valor `["budget"]` y ejecuta una combinación con el valor `["economy", "pool"]` para `tags`, el valor final del campo `tags` será `["economy", "pool"]`. No será `["budget", "economy", "pool"]`. |
| `mergeOrUpload` |Esta acción se comporta como `merge` si ya existe un documento con la clave especificada en el índice. Si el documento no existe, se comporta como `upload` con un nuevo documento. |la clave, además de cualquier otro campo que desee definir |- |
| `delete` |Quita el documento especificado del índice. |solo la clave |Todos los campos que especifique que no sean el campo de clave, se omitirán. Si desea quitar un campo individual de un documento, use `merge` en su lugar y establezca el campo explícitamente con el valor NULL. |

### <a name="formulate-your-query"></a>Formulación de la consulta

Hay dos maneras de [realizar búsquedas en el índice mediante la API de REST](/rest/api/searchservice/Search-Documents). Una de ellas es emitir una solicitud HTTP POST en la que los parámetros de consulta se definen en un objeto JSON del cuerpo de la solicitud. La otra es emitir una solicitud HTTP GET en la que los parámetros de la consulta se definen en la dirección URL de la solicitud. POST tiene unos [límites más flexibles](/rest/api/searchservice/Search-Documents) en relación con el tamaño de los parámetros de la consulta que GET. Por este motivo, se recomienda usar la solicitud POST a menos que haya circunstancias especiales en las que utilizar la solicitud GET sea más adecuado.

Tanto en POST como en GET, debe indicar su *nombre del servicio*, el *nombre del índice*y una *versión de la API* en la dirección URL de la solicitud. 

En el caso de GET, la *cadena de consulta* del final de la dirección URL es donde se proporcionar los parámetros de la consulta. Consulte a continuación el formato de dirección URL:

```http
    https://[service name].search.windows.net/indexes/[index name]/docs?[query string]&api-version=2019-05-06
```

El formato de la solicitud POST es el mismo, pero con `api-version` en los parámetros de la cadena de consulta.

## <a name="pulling-data-into-an-index"></a>Extracción de datos en un índice

El modelo de extracción rastrea un origen de datos compatible y carga automáticamente los datos en el índice. En Azure Cognitive Search, esta funcionalidad se implementa a través de *indizadores*, que en este momento se encuentran disponibles en las siguientes plataformas:

+ [Blob Storage](search-howto-indexing-azure-blob-storage.md)
+ [Table storage](search-howto-indexing-azure-tables.md)
+ [Azure Cosmos DB](search-howto-index-cosmosdb.md)
+ [Azure SQL Database, Instancia administrada de SQL y SQL Server en máquinas virtuales de Azure](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md)

Los indexadores conectan un índice a un origen de datos (normalmente una tabla, vista o estructura equivalente) y asignan campos de origen a los campos equivalentes del índice. Durante la ejecución, el conjunto de filas se transforma automáticamente en JSON y se carga en el índice especificado. Todos los indexadores admiten la programación, de modo que se puede especificar con qué frecuencia se deben actualizar los datos. La mayoría de los indexadores proporcionan seguimiento de cambios, siempre el origen de datos lo admita. Mediante el seguimiento de cambios y eliminaciones en documentos existentes, además de reconocer nuevos documentos, los indexadores eliminan la necesidad de administrar activamente los datos del índice. 


### <a name="how-to-pull-data-into-an-azure-cognitive-search-index"></a>Extracción de datos en un índice de Azure Cognitive Search

La funcionalidad del indexador se expone en [Azure Portal](search-import-data-portal.md) así como en la [API de REST](/rest/api/searchservice/Indexer-operations) y el [SDK de .NET](/dotnet/api/microsoft.azure.search.indexersoperationsextensions). 

Una ventaja del uso el portal es que Azure Cognitive Search normalmente puede generar automáticamente un esquema de índice predeterminado mediante la lectura de los metadatos del conjunto de datos de origen. El índice generado se puede modificar hasta que se procese, tras lo cual las únicas modificaciones de esquema que se permiten son las que no requieren volver a indexar. Si los cambios que desea realizar afectan directamente el esquema, necesitaría volver a generar el índice. 

## <a name="verify-data-import-with-search-explorer"></a>Verificación de la importación de datos con el Explorador de búsqueda

Una forma rápida de realizar una comprobación preliminar en la carga de documentos es usar el **Explorador de búsqueda** en el portal. El explorador permite consultar un índice sin tener que escribir código. La búsqueda se basa en los valores predeterminados, como la [sintaxis simple](/rest/api/searchservice/simple-query-syntax-in-azure-search) y el [parámetro de consulta searchMode](/rest/api/searchservice/search-documents) predeterminado. Los resultados se devuelven en formato JSON, con el fin de que pueda revisar todo el documento.

> [!TIP]
> Muchos [ejemplos de código de Azure Cognitive Search](https://github.com/Azure-Samples/?utf8=%E2%9C%93&query=search) incluyen conjuntos de datos incrustados o rápidamente disponibles, lo que supone una forma sencilla de empezar a trabajar. El portal también proporciona un indexador de ejemplo y un origen de datos que consta del conjunto de datos de una pequeña inmobiliaria (denominado "realestate-us-sample"). Al ejecutar el indexador preconfigurado en el origen de datos de ejemplo, se crea un índice que se carga con documentos que, luego, se pueden consultar en el Explorador de búsqueda o mediante un código creado por usted.

## <a name="see-also"></a>Consulte también

+ [Información general del indexador](search-indexer-overview.md)
+ [Tutorial del portal: crear, cargar, consultar un índice](search-get-started-portal.md)