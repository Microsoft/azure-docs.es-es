---
title: Conceptos de enriquecimiento incremental (versión preliminar)
titleSuffix: Azure Cognitive Search
description: Almacene en caché contenido intermedio y cambios incrementales de la canalización de enriquecimiento con IA en Azure Storage para conservar las inversiones en los documentos procesados existentes. Esta característica actualmente está en su versión preliminar pública.
manager: nitinme
author: Vkurpad
ms.author: vikurpad
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/18/2020
ms.openlocfilehash: 5596a2db32a0fe5b6b5eddf3ae20501e6edb0b99
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935388"
---
# <a name="incremental-enrichment-and-caching-in-azure-cognitive-search"></a>Enriquecimiento incremental y almacenamiento en caché en Azure Cognitive Search

> [!IMPORTANT] 
> El enriquecimiento incremental se encuentra actualmente en versión preliminar pública. Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Las [versiones de API de REST 2019-05-06-Preview y 2020-06-30-Preview](search-api-preview.md) proporcionan esta característica. Por el momento, no hay compatibilidad con el portal ni con .NET SDK.

El *enriquecimiento incremental* es una característica que tiene como destino los [conjuntos de aptitudes](cognitive-search-working-with-skillsets.md). Aprovecha Azure Storage para guardar la salida de procesamiento que emite una canalización de enriquecimiento para su reutilización en ejecuciones futuras del indizador. Siempre que sea posible, el indizador reutiliza cualquier salida almacenada en caché que todavía sea válida. 

El enriquecimiento incremental no solo permite conservar la inversión monetaria en el procesamiento (en concreto, OCR y procesamiento de imágenes), sino que también ofrece un sistema más eficaz. Cuando las estructuras y el contenido se almacenan en la caché, un indexador puede determinar qué aptitudes han cambiado y ejecutar solo aquellas que se han modificado, así como cualquier aptitud dependiente de bajada. 

Los flujos de trabajo que usan el almacenamiento en caché incremental incluyen estos pasos:

1. [Cree o identifique una cuenta de Azure Storage](../storage/common/storage-account-create.md) para almacenar la caché.
1. [Habilite el enriquecimiento incremental](search-howto-incremental-index.md) en el indizador.
1. [Cree un indizador](/rest/api/searchservice/create-indexer) (además de un [conjunto de aptitudes](/rest/api/searchservice/create-skillset)) para invocar la canalización. Durante el procesamiento, se guardan las fases de enriquecimiento para cada documento en el almacenamiento de blobs para un uso futuro.
1. Pruebe el código y, después de realizar los cambios, use [Actualizar conjunto de aptitudes](/rest/api/searchservice/update-skillset) para modificar una definición.
1. [Ejecute el indizador](/rest/api/searchservice/run-indexer) para invocar la canalización y recuperar la salida almacenada en caché para un procesamiento más rápido y rentable.

Para obtener más información sobre los pasos y las consideraciones al trabajar con un indizador existente, consulte [Configurar el enriquecimiento incremental](search-howto-incremental-index.md).

## <a name="indexer-cache"></a>Caché de indexador

El enriquecimiento incremental agrega una caché a la canalización de enriquecimiento. El indizador almacena en caché los resultados del descifrado de documentos más las salidas de todas las aptitudes de todos los documentos. Cuando se actualiza un conjunto de aptitudes, solo se vuelven a ejecutar las aptitudes modificadas o descendentes. Los resultados actualizados se escriben en la caché y el documento se actualiza en el índice de búsqueda o el almacén de conocimiento.

Físicamente, la caché se almacena en un contenedor de blobs en la cuenta de Azure Storage. La memoria caché también utiliza Table Storage para obtener un registro interno de las actualizaciones de procesamiento. Todos los índices de un servicio de búsqueda pueden compartir la misma cuenta de almacenamiento para la caché del indexador. A cada indizador se le asigna un identificador de caché único e inmutable al contenedor que usa.

## <a name="cache-configuration"></a>Configuración de la caché

Para empezar a beneficiarse del enriquecimiento incremental, tendrá que establecer la propiedad `cache` en el indizador. En el ejemplo siguiente se muestra un indexador con el almacenamiento en caché habilitado. En las secciones siguientes se describen partes específicas de esta configuración. Para más información, vea [Configuración del enriquecimiento incremental](search-howto-incremental-index.md).

```json
{
    "name": "myIndexerName",
    "targetIndexName": "myIndex",
    "dataSourceName": "myDatasource",
    "skillsetName": "mySkillset",
    "cache" : {
        "storageConnectionString" : "Your storage account connection string",
        "enableReprocessing": true
    },
    "fieldMappings" : [],
    "outputFieldMappings": [],
    "parameters": []
}
```

Para establecer esta propiedad en un indizador existente, tendrá que restablecer y volver a ejecutar el indizador, lo que hará que todos los documentos del origen de datos se vuelvan a procesar. Este paso es necesario para eliminar los documentos enriquecidos por versiones anteriores del conjunto de aptitudes. 

## <a name="cache-management"></a>Administración de la memoria caché

El indexador administra el ciclo de vida de la caché. Si la propiedad `cache` del indizador está establecida en NULL o la cadena de conexión ha cambiado, se eliminará la caché existente en la siguiente ejecución del indizador. El ciclo de vida de la caché también está asociado al ciclo de vida del indexador. Si se elimina un indexador, también se elimina la caché asociada.

Aunque el enriquecimiento incremental está diseñado para detectar y responder a los cambios sin intervención por su parte, existen parámetros que puede usar para invalidar los comportamientos predeterminados:

+ Priorizar nuevos documentos
+ Omitir comprobaciones de conjuntos de aptitudes
+ Omitir comprobaciones de origen de datos
+ Forzar la evaluación de conjuntos de aptitudes

### <a name="prioritize-new-documents"></a>Priorizar nuevos documentos

Establezca la propiedad `enableReprocessing` para controlar el procesamiento de los documentos entrantes ya representados en la caché. Cuando es `true` (valor predeterminado), los documentos que ya están en la caché se vuelven a procesar cuando se vuelve a ejecutar el indizador, siempre que la actualización de las aptitudes afecta a ese documento. 

Cuando es `false`, los documentos existentes no se vuelven a procesar, con lo que se prioriza de forma eficaz el contenido nuevo y entrante sobre el existente. Solo debe establecer `enableReprocessing` en `false` de forma temporal. Para garantizar la coherencia en el corpus, `enableReprocessing` debe ser `true` la mayor parte del tiempo, para asegurarse de que todos los documentos, tanto nuevos como existentes, sean válidos según la definición del conjunto de aptitudes actual.

### <a name="bypass-skillset-evaluation"></a>Omitir la evaluación del conjunto de aptitudes

La modificación y el reprocesamiento de un conjunto de aptitudes suelen estar relacionados. Pero algunos cambios de un conjunto de aptitudes no deben dar como resultado el reprocesamiento (por ejemplo, la implementación de una aptitud personalizada en una ubicación nueva o con una clave de acceso nueva). Lo más probable es que se trate de modificaciones periféricas, sin un impacto real en la sustancia del propio conjunto de aptitudes. 

Si sabe que un cambio en el conjunto de aptitudes es superficial, debe establecer el parámetro `disableCacheReprocessingChangeDetection` en `true` para invalidar la evaluación de conjuntos de aptitudes:

1. Llame a Update Skillset y modifique la definición del conjunto de aptitudes.
1. Anexe el parámetro `disableCacheReprocessingChangeDetection=true` en la solicitud.
1. Envíe el cambio.

Al establecer este parámetro se garantiza que solo se confirman las actualizaciones de la definición del conjunto de aptitudes y que el cambio no se evalúa para los efectos en el corpus existente.

En el ejemplo siguiente se muestra una solicitud Update Skillset con el parámetro:

```http
PUT https://customerdemos.search.windows.net/skillsets/callcenter-text-skillset?api-version=2020-06-30-Preview&disableCacheReprocessingChangeDetection=true
```

### <a name="bypass-data-source-validation-checks"></a>Omitir comprobaciones de validación de origen de datos

La mayoría de los cambios en una definición de origen de datos invalidarán la caché. Pero para los escenarios en los que sabe que un cambio no debe invalidar la caché, como cambiar una cadena de conexión o rotar la clave en la cuenta de almacenamiento, anexe el parámetro `ignoreResetRequirement` en la actualización del origen de datos. Establecer este parámetro en `true` permite que se realice la confirmación, sin desencadenar una condición de restablecimiento que daría lugar a que todos los objetos se volvieran a generar y se rellenaran desde el principio.

```http
PUT https://customerdemos.search.windows.net/datasources/callcenter-ds?api-version=2020-06-30-Preview&ignoreResetRequirement=true
```

### <a name="force-skillset-evaluation"></a>Forzar la evaluación de conjuntos de aptitudes

El propósito de la caché es evitar procesamientos innecesarios, pero imagine que realiza un cambio en una aptitud que el indizador no detecta (por ejemplo, cambiar un elemento del código externo, como una aptitud personalizada).

En este caso, puede usar [Reset Skills](/rest/api/searchservice/preview-api/reset-skills) para forzar el reprocesamiento de una aptitud determinada, incluidos los conocimientos de nivel inferior que tengan una dependencia en la salida de esa aptitud. Esta API acepta una solicitud POST con una lista de aptitudes que se deben invalidar y marcar para volver a procesarse. Después de Reset Skills, ejecute el indizador para invocar la canalización.

## <a name="change-detection"></a>Detección de cambios

Una vez que se ha habilitado la caché, el indizador evalúa los cambios en la composición de la canalización para determinar qué contenido se puede reusar y cuál se debe volver a procesar. En esta sección se enumeran los cambios que invalidan la caché directamente, seguidos de los que desencadenan el procesamiento incremental. 

### <a name="changes-that-invalidate-the-cache"></a>Cambios que invalidan la caché

Un cambio invalidador es aquél en el que la caché completa deja de ser válida. Un ejemplo de cambio invalidador es en el que se actualiza el origen de datos. Esta es la lista completa de cambios que invalidarían la memoria caché:

* Cambiar al tipo de origen de datos
* Cambiar al contenedor del origen de datos
* Credenciales del origen de datos
* Directiva de detección de cambios en el origen de datos
* Directiva de detección de eliminaciones en el origen de datos
* Asignaciones de campos del indexador
* Parámetros de un indexador
    * Modo de análisis
    * Extensiones de nombre de archivo excluidas
    * Extensiones de nombre de archivo indexadas
    * Metadatos de almacenamiento de índice solo para documentos de gran tamaño
    * Encabezados de texto delimitado
    * Delimitador de texto delimitado
    * Raíz del documento
    * Acción de imagen (cambios en la forma en que se extraen las imágenes)

### <a name="changes-that-trigger-incremental-processing"></a>Cambios que desencadenan el procesamiento incremental

El procesamiento incremental evalúa la definición del conjunto de aptitudes y determina qué aptitudes se deben volver a ejecutar, con lo que se actualizan selectivamente las partes afectadas del árbol del documento. Esta es la lista completa de cambios que dan como resultado el enriquecimiento incremental:

* La aptitud del conjunto de aptitudes tiene un tipo diferente. El tipo de oData de la aptitud está actualizado
* Los valores predeterminados de los parámetros específicos de la aptitud actualizados, como la dirección URL, u otros parámetros
* Cambios en las salidas de la aptitud, la aptitud devuelve salidas adicionales o diferentes
* Las actualizaciones de aptitudes resultantes tienen otra ascendencia, el encadenamiento de aptitudes ha cambiado, es decir, las entradas de las aptitudes
* Cualquier invalidación de aptitudes ascendente, si se actualiza una aptitud que proporcione una entrada a esta aptitud
* Actualizaciones de la ubicación de la proyección del almacén de información, lo que provoca que se vuelvan a proyectar los documentos
* Cambios en las proyecciones del almacén de información, lo que provoca que se vuelvan a proyectar los documentos
* Las asignaciones de campos de salida cambiadas en un indexador provocan que se vuelvan a proyectos los documentos en el índice

## <a name="api-reference"></a>Referencia de API

La versión de la API de REST `2020-06-30-Preview` proporciona enriquecimiento incremental a través de propiedades adicionales en indizadores. Los conjuntos de aptitudes y los orígenes de datos pueden usar la versión disponible con carácter general. Además de la documentación de referencia, consulte [Configuración del almacenamiento en caché para el enriquecimiento incremental](search-howto-incremental-index.md) para más información sobre cómo llamar a las API.

+ [Crear indizador (api-version=2020-06-30-Preview)](/rest/api/searchservice/create-indexer) 

+ [Actualizar indizador (api-version=2020-06-30-Preview)](/rest/api/searchservice/update-indexer) 

+ [Actualizar conjunto de aptitudes (api-version=2020-06-30)](/rest/api/searchservice/update-skillset) (nuevo parámetro URI en la solicitud)

+ [Restablecer aptitudes (api-version=2020-06-30)](/rest/api/searchservice/preview-api/reset-skills)

+ Indizadores de base de datos (Azure SQL, Cosmos DB). Algunos indizadores recuperan datos a través de consultas. En cuanto a las consultas que recuperan datos, [Update Data Source](/rest/api/searchservice/update-data-source) admite un nuevo parámetro en una solicitud **ignoreResetRequirement**, que se tiene que establecer en `true` si la acción de actualización no debe invalidar la caché. 

  Use **ignoreResetRequirement** con moderación, ya que puede dar lugar a incoherencias no intencionadas en los datos, que no se detectarán fácilmente.

## <a name="next-steps"></a>Pasos siguientes

El enriquecimiento incremental es una característica eficaz que extiende el seguimiento de cambios en conjuntos de aptitudes y el enriquecimiento con IA. El enriquecimiento incremental permite la reutilización del contenido procesado existente mientras se realiza la iteración en el diseño del conjunto de aptitudes.

Para continuar, habilite el almacenamiento en caché en un indizador existente o agregue una caché al definir un indizador nuevo.

> [!div class="nextstepaction"]
> [Configuración del almacenamiento en caché para el enriquecimiento incremental](search-howto-incremental-index.md)