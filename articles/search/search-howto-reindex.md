---
title: Recompilar un índice de búsqueda
titleSuffix: Azure Cognitive Search
description: Agregue elementos nuevos, actualice elementos o documentos existentes o elimine documentos obsoletos en una recompilación competa o una indexación parcial para actualizar un índice de Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/18/2020
ms.openlocfilehash: 10c7d4146d61f5b589b29bc8faad5fa8e60a293a
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88924034"
---
# <a name="how-to-rebuild-an-index-in-azure-cognitive-search"></a>Cómo recompilar un índice en Azure Cognitive Search

En este artículo se explica cómo recompilar un índice de Azure Cognitive Search, las circunstancias en las que las recompilaciones son necesarias y recomendaciones para mitigar el impacto de las recompilaciones en las solicitudes de consulta en curso.

Una *recompilación* se refiere a quitar y volver a crear las estructuras de datos físicas asociadas con un índice, incluidos todos los índices invertidos basados en campos. En Azure Cognitive Search, no se pueden quitar y volver a crear campos individuales. Para recompilar un índice, se debe eliminar todo el almacenamiento de campos, se debe volver a crear según un esquema de índice existente o revisado y, a continuación, se debe volver a completar con los datos insertados en el índice o extraídos de orígenes externos. 

Es común recompilar los índices durante el desarrollo mientras se itera por el diseño del índice, pero también es posible que deba recompilar un índice de nivel de producción para dar cabida a cambios estructurales, como la adición de tipos complejos o campos a los proveedores de sugerencias.

## <a name="rebuild-versus-refresh"></a>"Recompilar" frente a "actualizar"

La recompilación no se debe confundir con la actualización del contenido de un índice con documentos nuevos, modificados o eliminados. La actualización de una corpus de búsqueda es casi obligatorio en cada aplicación de búsqueda, con algunos escenarios que requieren actualizaciones al minuto (por ejemplo, cuando un corpus de búsqueda necesita reflejar cambios de inventario en una aplicación de ventas en línea).

Siempre y cuando no cambie la estructura del índice, puede actualizarlo con las mismas técnicas que usó para cargarlo inicialmente:

* Para la indexación de modo de inserción, realice una llamada para [agregar, actualizar o eliminar documentos](/rest/api/searchservice/addupdate-or-delete-documents) con el fin de enviar los cambios a un índice.

* En el caso de los indexadores, puede [programar la ejecución del indexador](search-howto-schedule-indexers.md) y usar las marcas de tiempo o el seguimiento de cambios para identificar la diferencia. Si las actualizaciones se deben reflejar más pronto de lo que puede hacerlo el programador, en su lugar puede usar la indexación del modo de inserción.

## <a name="rebuild-conditions"></a>Condiciones de recompilación

Elimine y vuelva a crear un índice si se cumple cualquiera de las condiciones siguientes. 

| Condición | Descripción |
|-----------|-------------|
| Cambiar la definición de un campo | Revisar el nombre, el tipo de datos o los [atributos de índice](/rest/api/searchservice/create-index) específicos (si se pueden buscar, filtrar, ordenar, clasificar) de un campo requiere una recompilación completa. |
| Asignar un analizador a un campo | Los [analizadores](search-analyzers.md) se definen en un índice y, a continuación, se asignan a los campos. Puede agregar una nueva definición de analizador a un índice en cualquier momento, pero solo puede *asignar* un analizador cuando se crea el campo. Esto es así tanto para la propiedad **analyzer** como para la propiedad **indexAnalyzer**. La propiedad **searchAnalyzer** es una excepción (puede asignar esta propiedad a un campo existente). |
| Actualizar o eliminar una definición de analizador en un índice | No puede eliminar ni cambiar una configuración de analizador existente (analizador, tokenizador, filtro de token o filtro de caracteres) en el índice, a menos que recompile todo el índice. |
| Agregar un campo a un proveedor de sugerencias | Si ya existe un campo y desea agregarlo a una construcción de [proveedor de sugerencias](index-add-suggesters.md), debe recompilar el índice. |
| Eliminar un campo | Para quitar físicamente todos los rastros de un campo, tendrá que recompilar el índice. Cuando una recompilación inmediata no es práctica, puede modificar el código de aplicación para deshabilitar el acceso al campo "eliminado" o usar el [parámetro de consulta $select](search-query-odata-select.md) para elegir los campos que se representan en el conjunto de resultados. Físicamente, el contenido y la definición del campo permanecen en el índice hasta la próxima recompilación, cuando aplica un esquema que omite el campo en cuestión. |
| Cambiar los niveles | Si necesita más capacidad, no hay ninguna actualización local en Azure Portal. Se debe crear otro servicio y los índices se deben compilar de cero en el servicio nuevo. Para ayudar a automatizar este proceso, puede usar el código de ejemplo **index-backup-restore** de este [repositorio de ejemplo .NET de Azure Cognitive Search](https://github.com/Azure-Samples/azure-search-dotnet-samples). Esta aplicación hará una copia de seguridad del índice en una serie de archivos JSON y luego volverá a crear el índice en un servicio de búsqueda que especifique.|

## <a name="update-conditions"></a>Actualización de condiciones

Se pueden realizar muchas otras modificaciones sin que ello afecte a las estructuras físicas existentes. En concreto, los siguientes cambios *no* requieren una recompilación del índice. Para estos cambios, puede [actualizar una definición del índice](/rest/api/searchservice/update-index) con los cambios.

+ Adición de un nuevo campo
+ Definición del atributo **retrievable** en un campo existente
+ Definición de **searchAnalyzer** en un campo existente
+ Adición de una nueva definición de analizador en un índice
+ Adición, actualización o eliminación de los perfiles de puntuación
+ Adición, actualización o eliminación de configuraciones CORS
+ Adición, actualización o eliminación de synonymMaps

Cuando agrega un campo nuevo, a los documentos indexados existentes se les asigna un valor NULL para el campo nuevo. En una futura nueva indexación, valores provenientes de los datos de origen reemplazarán los valores NULL que agregó Azure Cognitive Search. Para obtener más información sobre cómo actualizar el contenido de un índice, consulte [Add, update, or delete documents in Azure Search](/rest/api/searchservice/addupdate-or-delete-documents) (Agregar, actualizar o eliminar documentos en Azure Search).

## <a name="how-to-rebuild-an-index"></a>Cómo recompilar un índice

Durante el desarrollo, el esquema de índice cambia con frecuencia. Puede planearlo mediante la creación de índices que se pueden eliminar, volver a crear y recargar rápidamente con un pequeño conjunto de datos representativo.

Para las aplicaciones ya en producción, se recomienda crear un nuevo índice que se ejecute en paralelo a un índice existente para evitar el tiempo de inactividad de la consulta. El código de aplicación proporciona redireccionamiento al nuevo índice.

La indexación no se ejecuta en segundo plano y el servicio equilibrará la indexación adicional con las consultas en curso. Durante la indexación, puede [supervisar las solicitudes de consulta](search-monitor-queries.md) en el portal para asegurarse de que las consultas se completen puntualmente.

1. Determine si es necesaria una recompilación. Si solo va a agregar campos o a cambiar alguna parte del índice que no esté relacionada con los campos, es posible que pueda [actualizar la definición](/rest/api/searchservice/update-index) sin eliminarla, volver a crearla y a cargarla por completo.

1. [Obtenga una definición de índice](/rest/api/searchservice/get-index), por si la necesita para futuras referencias.

1. [Elimine el índice existente](/rest/api/searchservice/delete-index), siempre que no esté ejecutando índices nuevos y antiguos en paralelo. 

   Se quitan inmediatamente todas las consultas destinadas a ese índice. Recuerde que la eliminación de un índice es una acción irreversible, ya que se destruye el almacenamiento físico para la colección de campos y otras construcciones. Deténgase a pensar en las implicaciones antes de quitarlo. 

1. [Cree un índice revisado](/rest/api/searchservice/create-index), en el que el cuerpo de la solicitud incluya definiciones de campo modificadas o cambiadas.

1. [Cargue documentos en el índice](/rest/api/searchservice/addupdate-or-delete-documents) desde un origen de datos externo.

Cuando se crea el índice, se asigna almacenamiento físico para cada campo en el esquema de índice, con un índice invertido que se crea para cada campo de búsqueda. Los campos que no pueden buscarse se pueden usar en filtros o expresiones, pero no cuentan con índices invertidos y no están habilitados para búsquedas de texto completo o aproximadas. En una recompilación de índices, se eliminan y vuelven a crear estos índices invertidos en función del esquema de índice proporcionado.

Al cargar el índice, el índice invertido de cada campo se rellena con todas las palabras únicas y con tokens de cada documento, con una asignación a los identificadores del documento correspondiente. Por ejemplo, al indizar un conjunto de datos de hoteles, el índice invertido creado para un campo Ciudad podría contener términos para Seattle, Portland y así sucesivamente. El identificador de documento de los documentos que incluyan Seattle o Portland en el campo Ciudad aparecerá junto con el término. En cualquier operación para [agregar, actualizar o eliminar](/rest/api/searchservice/addupdate-or-delete-documents), los términos y la lista de identificadores de documento se actualizan en consecuencia.

> [!NOTE]
> Si tiene requisitos estrictos de Acuerdo de Nivel de Servicio, puede considerar la posibilidad de aprovisionar un nuevo servicio específicamente para esta tarea, para que el desarrollo y la indización se produzcan completamente aislados del índice de producción. Un servicio independiente se ejecuta en su propio hardware, lo que elimina cualquier posibilidad de contención de recursos. Una vez completado el desarrollo, puede dejar el nuevo índice en su lugar, redirigir las consultas hacia el nuevo punto de conexión e índice o puede ejecutar código terminado para publicar un índice revisado en el servicio de Azure Cognitive Search original. Actualmente no hay ningún mecanismo para mover un índice listo para usarse a otro servicio.

## <a name="check-for-updates"></a>Buscar actualizaciones

Puede empezar a consultar un índice en cuanto se carga el primer documento. Si conoce el identificador de un documento, la [API REST para buscar documentos](/rest/api/searchservice/lookup-document) devuelve el documento específico. Para realizar pruebas más amplias, debe esperar hasta que el índice se haya cargado completamente y, a continuación, usar consultas para comprobar el contexto que espera ver.

Puede usar el [Explorador de búsqueda](search-explorer.md) o una herramienta de pruebas web como [Postman](search-get-started-postman.md) para comprobar el contenido actualizado.

Si ha agregado un campo o le ha cambiado el nombre, use [$select](search-query-odata-select.md) para volver a ese campo: `search=*&$select=document-id,my-new-field,some-old-field&$count=true`

## <a name="see-also"></a>Consulte también

+ [Información general del indexador](search-indexer-overview.md)
+ [Indexación de grandes conjuntos de datos a escala](search-howto-large-index.md)
+ [Indexing in the portal](search-import-data-portal.md) (Indexación en el portal)
+ [Azure SQL Database indexer](search-howto-connecting-azure-sql-database-to-azure-search-using-indexers.md) (Indexador de Azure SQL Database)
+ [Indexador de Azure Cosmos DB](search-howto-index-cosmosdb.md)
+ [Indexador de Azure Blob Storage](search-howto-indexing-azure-blob-storage.md)
+ [Indexador de Azure Table Storage](search-howto-indexing-azure-tables.md)
+ [Seguridad en Azure Cognitive Search](search-security-overview.md)