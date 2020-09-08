---
title: Errores y advertencias del indexador
titleSuffix: Azure Cognitive Search
description: En este artículo se proporciona información y soluciones a errores y advertencias comunes que pueden surgir durante el enriquecimiento de IA en Azure Cognitive Search.
manager: nitinme
author: amotley
ms.author: abmotley
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: fdae02ca9d3c434a77eb972bfd4b955161bd72c4
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935558"
---
# <a name="troubleshooting-common-indexer-errors-and-warnings-in-azure-cognitive-search"></a>Solución de errores y advertencias comunes con el indexador en Azure Cognitive Search

En este artículo se proporciona información y soluciones a errores y advertencias comunes que pueden surgir durante la indexación y el enriquecimiento de IA en Azure Cognitive Search.

La indexación se detiene cuando el recuento de errores supera ["maxfaileditems"](cognitive-search-concept-troubleshooting.md#tip-3-see-what-works-even-if-there-are-some-failures). 

Si desea que los indexadores omitan estos errores (y omitan los "documentos con errores"), considere la posibilidad de actualizar `maxFailedItems` y `maxFailedItemsPerBatch` como se describe [aquí](/rest/api/searchservice/create-indexer#general-parameters-for-all-indexers).

> [!NOTE]
> Cada documento con error junto con su clave de documento (si está disponible) se mostrará como un error en el estado de ejecución del indexador. Puede utilizar la [API de índice](/rest/api/searchservice/addupdate-or-delete-documents) para cargar manualmente los documentos en un momento posterior si ha configurado el indexador para tolerar errores.

La información de error de este artículo pueden ayudarle a resolver errores, de forma que la indexación continúe.

Las advertencias no detienen la indexación, sino que indican las condiciones que podrían dar lugar a resultados inesperados. Que realice o no una acción depende de los datos y del escenario.

A partir de la versión de API `2019-05-06`, los errores y advertencias del indexador de nivel de elemento están estructurados para proporcionar una mayor claridad en torno a las causas y los pasos siguientes. Contienen las siguientes propiedades:

| Propiedad | Descripción | Ejemplo |
| --- | --- | --- |
| key | Identificador de documento del documento afectado por el error o la advertencia. | https:\//coromsearch.blob.core.windows.net/jfk-1k/docid-32112954.pdf |
| name | Nombre de la operación que describe dónde se produjo el error o la advertencia. Lo genera la siguiente estructura: [category].[subcategory].[resourceType].[resourceName] | DocumentExtraction.azureblob.myBlobContainerName Enrichment.WebApiSkill.mySkillName Projection.SearchIndex.OutputFieldMapping.myOutputFieldName Projection.SearchIndex.MergeOrUpload.myIndexName Projection.KnowledgeStore.Table.myTableName |
| message | Descripción de alto nivel del error o la advertencia. | No se pudo ejecutar la aptitud debido a un error en la solicitud a la API web. |
| detalles | Detalles adicionales que pueden ser útiles para diagnosticar el problema, como la respuesta de WebAPI, si se produce un error en la ejecución de una aptitud personalizada. | `link-cryptonyms-list - Error processing the request record : System.ArgumentNullException: Value cannot be null. Parameter name: source at System.Linq.Enumerable.All[TSource](IEnumerable`Un origen, Func`2 predicate) at Microsoft.CognitiveSearch.WebApiSkills.JfkWebApiSkills.` ...resto del seguimiento de la pila... |
| documentationLink | Vínculo a la documentación pertinente con información detallada para depurar y resolver el problema. Este vínculo suele apuntar a una de las secciones siguientes de esta página. | https://go.microsoft.com/fwlink/?linkid=2106475 |

<a name="could-not-read-document"></a>

## <a name="error-could-not-read-document"></a>Error: No se pudo leer el documento

El indexador no pudo leer el documento del origen de datos. Estos pueden ser los motivos:

| Motivo | Detalles/ejemplo | Resolución |
| --- | --- | --- |
| Tipos de campo incoherentes en distintos documentos | "El tipo de valor no coincide con el tipo de columna. No se pudo almacenar `'{47.6,-122.1}'` en la columna de autores.  El tipo esperado es JArray".  "Error al convertir el tipo de datos nvarchar a float".  "Error de conversión al convertir el valor nvarchar "12 meses" al tipo de datos int".  “Error de desbordamiento aritmético al convertir expresión al tipo de datos int”. | Asegúrese de que el tipo de cada campo sea el mismo en los distintos documentos. Por ejemplo, si el campo `'startTime'` del primer documento es DateTime y el del segundo documento es una cadena, se mostrará este error. |
| errores del servicio subyacente del origen de datos | (de Cosmos DB) `{"Errors":["Request rate is large"]}` | Compruebe la instancia de almacenamiento para asegurarse de que es correcta. Es posible que tenga que ajustar el escalado o la creación de particiones. |
| problemas transitorios | Error en el nivel del transporte al recibir los resultados del servidor. (proveedor: Proveedor TCP, error: 0: El host remoto forzó el cierre de la conexión existente. | En ocasiones, hay problemas de conectividad inesperados. Intente volver a ejecutar el documento mediante el indexador más adelante. |

<a name="could-not-extract-document-content"></a>

## <a name="error-could-not-extract-content-or-metadata-from-your-document"></a>Error: No se pudo extraer el contenido o los metadatos del documento
El indexador con un origen de datos de blob no pudo extraer el contenido o los metadatos del documento (por ejemplo, un archivo PDF). Estos pueden ser los motivos:

| Motivo | Detalles/ejemplo | Resolución |
| --- | --- | --- |
| el blob está por encima del límite de tamaño | El documento tiene `'150441598'` bytes, lo que supera el tamaño máximo de `'134217728'` bytes para la extracción de documentos con el nivel de servicio actual. | [errores de indexación de blobs](search-howto-indexing-azure-blob-storage.md#dealing-with-errors) |
| el blob tiene un tipo de contenido no admitido | El documento tiene un tipo de contenido no admitido `'image/png'` | [errores de indexación de blobs](search-howto-indexing-azure-blob-storage.md#dealing-with-errors) |
| el blob está cifrado | No se pudo procesar el documento; puede que esté cifrado o protegido con contraseña. | Puede omitir el blob con la [configuración de blob](search-howto-indexing-azure-blob-storage.md#controlling-which-parts-of-the-blob-are-indexed). |
| problemas transitorios | "Error al procesar el blob: Se anuló la solicitud: se canceló la solicitud." "Se agotó el tiempo de espera del documento durante el procesamiento". | En ocasiones, hay problemas de conectividad inesperados. Intente volver a ejecutar el documento mediante el indexador más adelante. |

<a name="could-not-parse-document"></a>

## <a name="error-could-not-parse-document"></a>Error: No se pudo analizar el documento
El indexador leyó el documento desde el origen de datos, pero hubo un problema al convertir el contenido del documento en el esquema de asignación de campos especificado. Estos pueden ser los motivos:

| Motivo | Detalles/ejemplo | Resolución |
| --- | --- | --- |
| Falta la clave del documento. | No puede faltar la clave del documento ni estar vacía | Asegúrese de que todos los documentos tengan claves de documento válidas. La clave del documento se determina estableciendo la propiedad "key" como parte de la [definición del índice](/rest/api/searchservice/create-index#request-body). Los indizadores emitirán este error cuando la propiedad marcada como "key" no se encuentre en un documento concreto. |
| La clave del documento no es válida | La clave del documento no puede tener más de 1024 caracteres | Modifique la clave del documento para que cumpla los requisitos de validación. |
| No se pudo aplicar la asignación de campos a un campo | No se pudo aplicar la función de asignación `'functionName'` al campo `'fieldName'`. La matriz no puede ser NULL. Nombre de parámetro: bytes | Compruebe las [asignaciones de campos](search-indexer-field-mappings.md) definidas en el indexador y compárelas con los datos del campo especificado del documento con errores. Puede que sea necesario modificar las asignaciones de campos o los datos del documento. |
| No se pudo leer el valor del campo | No se pudo leer el valor de la columna `'fieldName'` en el índice `'fieldIndex'`. Error en el nivel del transporte al recibir los resultados del servidor. (proveedor: Proveedor TCP, error: 0: El host remoto forzó el cierre de la conexión existente). | Normalmente, estos errores se deben a problemas de conectividad inesperados con el servicio subyacente del origen de datos. Intente volver a ejecutar el documento mediante el indexador más adelante. |

<a name="Could not map output field '`xyz`' to search index due to deserialization problem while applying mapping function '`abc`'"></a>

## <a name="error-could-not-map-output-field-xyz-to-search-index-due-to-deserialization-problem-while-applying-mapping-function-abc"></a>Error: No se pudo asignar el campo de salida "`xyz`" al índice de búsqueda debido a un problema de deserialización al aplicar la función de asignación "`abc`".
Es posible que se haya producido un error en la asignación de salida porque los datos de salida tienen un formato incorrecto en la función de asignación que está usando. Por ejemplo, si se aplica la función de asignación Base64Encode en datos binarios, se generaría este error. Para resolver el problema, vuelva a ejecutar el indexador sin especificar la función de asignación o asegúrese de que la función de asignación es compatible con el tipo de datos del campo de salida. Consulte [Asignación de campos de salida](cognitive-search-output-field-mapping.md) para obtener más información.

<a name="could-not-execute-skill"></a>

## <a name="error-could-not-execute-skill"></a>Error: No se pudo ejecutar la aptitud
El indexador no pudo ejecutar una aptitud del conjunto de aptitudes.

| Motivo | Detalles/ejemplo | Resolución |
| --- | --- | --- |
| Problemas de conectividad transitorios | Se produjo un error transitorio. Inténtelo de nuevo más tarde. | En ocasiones, hay problemas de conectividad inesperados. Intente volver a ejecutar el documento mediante el indexador más adelante. |
| Posible error del producto | Se ha producido un error inesperado. | Esto indica una clase desconocida de error y puede significar que hay un error del producto. Registre una [incidencia de soporte técnico](https://ms.portal.azure.com/#create/Microsoft.Support) para obtener ayuda. |
| Una aptitud detectó un error durante la ejecución | (De la Aptitud Combinación) Uno o varios valores de desplazamiento no eran válidos y no se pudieron analizar. Los elementos se insertaron al final del texto | Use la información del mensaje de error para solucionar el problema. Este tipo de error requerirá la acción de resolución. |

<a name="could-not-execute-skill-because-the-web-api-request-failed"></a>

## <a name="error-could-not-execute-skill-because-the-web-api-request-failed"></a>Error: No se pudo ejecutar la aptitud debido a un error en la solicitud a la API web
Error al ejecutar la aptitud porque no se pudo realizar la llamada a la API web. Por lo general, esta clase de error se produce cuando se usan aptitudes personalizadas, en cuyo caso necesitará depurar el código personalizado para resolver el problema. Por el contrario, si el error proviene de una aptitud integrada, consulte el mensaje de error para saber cómo corregir el problema.

Durante la depuración de este problema, asegúrese de prestar atención a las [advertencias de entrada de aptitudes](#warning-skill-input-was-invalid) de esta aptitud. Puede que el punto de conexión de la API web no funcione porque el indexador está pasándole una entrada inesperada.

<a name="could-not-execute-skill-because-web-api-skill-response-is-invalid"></a>

## <a name="error-could-not-execute-skill-because-web-api-skill-response-is-invalid"></a>Error: No se pudo ejecutar la aptitud porque la respuesta de la aptitud de API web no es válida
No se pudo ejecutar la aptitud porque la llamada a la API web devolvió una respuesta no válida. Por lo general, esta clase de error se produce cuando se usan aptitudes personalizadas, en cuyo caso necesitará depurar el código personalizado para resolver el problema. Por el contrario, si el error proviene de una aptitud integrada, registre una [incidencia de soporte técnico](https://ms.portal.azure.com/#create/Microsoft.Support) para obtener ayuda.

<a name="skill-did-not-execute-within-the-time-limit"></a>

## <a name="error-skill-did-not-execute-within-the-time-limit"></a>Error: La aptitud no se ejecutó dentro del límite de tiempo
Hay dos casos en los que puede encontrarse con este mensaje de error, cada uno de los cuales debe tratarse de forma diferente. Siga las instrucciones que se indican a continuación según la aptitud que devuelva este error en su caso.

### <a name="built-in-cognitive-service-skills"></a>Aptitudes integradas del servicio cognitivo
Muchas de las aptitudes cognitivas integradas, como la detección del idioma, el reconocimiento de entidades o el método OCR, están respaldadas por un punto de conexión de API de servicio cognitivo. En ocasiones, hay problemas transitorios con estos puntos de conexión y se agota el tiempo de espera de la solicitud. En estos casos, la única solución es esperar y volver a intentarlo. Como medida de mitigación, podría configurar un indexador para que [se ejecute según una programación](search-howto-schedule-indexers.md). La indexación programada reanuda el proceso donde se detuvo. Suponiendo que se resuelvan los problemas transitorios, la indexación y el procesamiento de aptitudes cognitivas podrán continuar en la siguiente ejecución programada.

Si sigue viendo este error en el mismo documento para una aptitud cognitiva integrada, registre una [incidencia de soporte técnico](https://ms.portal.azure.com/#create/Microsoft.Support) para obtener ayuda, ya que no se espera.

### <a name="custom-skills"></a>Aptitudes personalizadas
Si se produce un error de tiempo de espera con una aptitud personalizada que haya creado, hay un par de cosas que puede intentar. En primer lugar, revise la aptitud personalizada y asegúrese de que no se quede atascada en un bucle y que devuelva un resultado coherente. Cuando haya confirmado que ese es el caso, determine cuál es el tiempo de ejecución de su aptitud. Si no estableció explícitamente un valor `timeout` en la definición de aptitud personalizada, el valor `timeout` predeterminado es 30 segundos. Si 30 segundos no es un tiempo lo bastante largo para que se ejecute su aptitud, puede especificar un valor de `timeout` superior en la definición de aptitud personalizada. Este es un ejemplo de una definición de aptitud personalizada en la que el tiempo de espera se establece en 90 segundos:

```json
  {
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "uri": "<your custom skill uri>",
        "batchSize": 1,
        "timeout": "PT90S",
        "context": "/document",
        "inputs": [
          {
            "name": "input",
            "source": "/document/content"
          }
        ],
        "outputs": [
          {
            "name": "output",
            "targetName": "output"
          }
        ]
      }
```

El valor máximo que puede establecer para el parámetro `timeout` es de 230 segundos.  Si su aptitud personalizada no se puede ejecutar de forma coherente en 230 segundos, considere la posibilidad de reducir el valor de `batchSize` de su aptitud personalizada para que tenga menos documentos que procesar en una sola ejecución.  Si ya ha establecido `batchSize` en 1, tendrá que volver a escribir la aptitud para que se pueda ejecutar en menos de 230 segundos; o también puede dividirla en varias aptitudes personalizadas para que el tiempo de ejecución de una de ellas sea como máximo 230 segundos. Para más información, revise la [documentación de las aptitudes personalizadas](cognitive-search-custom-skill-web-api.md).

<a name="could-not-mergeorupload--delete-document-to-the-search-index"></a>

## <a name="error-could-not-mergeorupload--delete-document-to-the-search-index"></a>Error: No se pudo "`MergeOrUpload`" | "`Delete`" al índice de búsqueda

El documento se leyó y se procesó, pero el indexador no pudo agregarlo al índice de búsqueda. Estos pueden ser los motivos:

| Motivo | Detalles/ejemplo | Resolución |
| --- | --- | --- |
| Un campo contiene un término demasiado grande | Un término del documento es mayor que el [límite de 32 KB](search-limits-quotas-capacity.md#api-request-limits) | Para evitar esta restricción, asegúrese de que el campo no está configurado como filtrable, con facetas o que se puede ordenar.
| El documento es demasiado grande para indexarlo | Un documento es mayor que el [tamaño de solicitud de API máximo](search-limits-quotas-capacity.md#api-request-limits) | [Indexación de grandes conjuntos de datos](search-howto-large-index.md)
| El documento contiene demasiados objetos en la colección | Una colección del documento supera el [número máximo de elementos en el límite de todas las colecciones complejas](search-limits-quotas-capacity.md#index-limits) "El documento con la clave `'1000052'` tiene `'4303'` objetos en colecciones (matrices de JSON). Se permiten como máximo `'3000'` objetos en las colecciones de todo el documento. Quite los objetos de las colecciones e intente indexar el documento de nuevo". | Se recomienda reducir el tamaño de la colección compleja en el documento para bajar el límite y evitar un uso elevado del almacenamiento.
| Problemas de conexión con el índice de destino (que persiste después de varios reintentos) porque el servicio está bajo otra carga, como consulta o indexación. | No se puede establecer la conexión para actualizar el índice. El servicio de búsqueda está sometido a mucha carga. | [Escalar verticalmente el servicio de búsqueda](search-capacity-planning.md)
| El servicio de búsqueda se está revisando para la actualización del servicio o está en medio de una reconfiguración de la topología. | No se puede establecer la conexión para actualizar el índice. El servicio de búsqueda está inactivo o está experimentando una transición. | Configure el servicio con al menos 3 réplicas para una disponibilidad del 99,9 % según se indica en la [documentación del Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/search/v1_0/).
| Error en el recurso de proceso o de red subyacente (poco frecuente) | No se puede establecer la conexión para actualizar el índice. Se produjo un error desconocido. | Establezca los indexadores en [Ejecutar según una programación](search-howto-schedule-indexers.md) para que se recuperen de un estado de error.
| Una solicitud de indexación realizada al índice de destino no se confirmó en un período de tiempo de espera debido a problemas de red. | No se pudo establecer la conexión con el índice de búsqueda de manera oportuna. | Establezca los indexadores en [Ejecutar según una programación](search-howto-schedule-indexers.md) para que se recuperen de un estado de error. Además, intente reducir el [tamaño del lote](/rest/api/searchservice/create-indexer#parameters) del índice si esta condición de error persiste.

<a name="could-not-index-document-because-the-indexer-data-to-index-was-invalid"></a>

## <a name="error-could-not-index-document-because-some-of-the-documents-data-was-not-valid"></a>Error: Could not index document because some of the document's data was not valid (No se pudo indexar el documento porque algunos datos del documento no eran válidos)

El indexador leyó y procesó el documento, pero debido a un error de coincidencia en la configuración de los campos del índice y los datos extraídos y procesados por el indexador, no se pudo agregar al índice de búsqueda. Estos pueden ser los motivos:

| Motivo | Detalles/ejemplo
| --- | ---
| El tipo de datos de los campos extraídos por el indexador no es compatible con el modelo de datos del campo de índice de destino correspondiente. | El campo de datos "_data_" del documento con la clave "888" tiene un valor no válido "of type 'Edm.String'". El tipo esperado era "Collection(Edm.String)". |
| Error al extraer una entidad JSON de un valor de cadena. | No se pudo analizar el valor "of type 'Edm.String'" del campo "_data_" como objeto JSON. Error: "After parsing a value an unexpected character was encountered: ''. Path '_path_', line 1, position 3162". |
| Error al extraer una colección de entidades JSON de un valor de cadena.  | No se pudo analizar el valor "of type 'Edm.String'" del campo "_data_" como matriz JSON. Error: "After parsing a value an unexpected character was encountered: ''. Path '[0]', line 1, position 27". |
| Se detectó un tipo desconocido en el documento de origen. | No se puede indexar el tipo desconocido "_unknown_" |
| Se usó una notación no compatible para los puntos geográficos en el documento de origen. | No se admiten los literales de cadena WKT POINT. En su lugar, use los literales de punto GeoJSON. |

En todos estos casos, consulte [Tipos de datos admitidos](/rest/api/searchservice/supported-data-types) y [Asignación de tipos de datos para indexadores](/rest/api/searchservice/data-type-map-for-indexers-in-azure-search) para asegurarse de compilar correctamente el esquema de índice y de haber configurado las [asignaciones de campos de indexador](search-indexer-field-mappings.md) adecuadas. El mensaje de error incluirá detalles que pueden ayudar a realizar un seguimiento del origen del error de coincidencia.

## <a name="error-integrated-change-tracking-policy-cannot-be-used-because-table-has-a-composite-primary-key"></a>Error: No se puede usar la directiva de seguimiento de cambios integrada porque la tabla tiene una clave principal compuesta.

Esto se aplica a tablas SQL y suele ocurrir cuando la clave se define como una clave compuesta o cuando la tabla ha definido un índice agrupado único (como en un índice de SQL, no un índice de Azure Search). La razón principal es que el atributo clave se ha modificado para que sea una clave principal compuesta en el caso de un [índice agrupado único](/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described?view=sql-server-ver15). En ese caso, asegúrese de que la tabla SQL no tenga un índice agrupado único o de que asigna el campo de clave a un campo del que se garantiza que no tiene valores duplicados.

<a name="could-not-process-document-within-indexer-max-run-time"></a>

## <a name="error-could-not-process-document-within-indexer-max-run-time"></a>Error: No se pudo procesar el documento en el tiempo de ejecución máximo del indexador

Este error se produce cuando el indexador no puede finalizar el procesamiento de un único documento desde el origen de datos en el tiempo de ejecución permitido. El [tiempo de ejecución máximo](search-limits-quotas-capacity.md#indexer-limits) es más corto cuando se usan conjuntos de aptitudes. Cuando se produce este error, si tiene maxFailedItems establecido en un valor distinto de 0, el indexador omite el documento en futuras ejecuciones para que la indexación pueda continuar. Si no puede permitirse omitir ningún documento, o si ve este error sistemáticamente, podría dividir los documentos en otros más pequeños para que se pueda avanzar parcialmente dentro de una única ejecución del indexador.

<a name="could-not-project-document></a>

## <a name="error-could-not-project-document"></a>Error: No se pudo proyectar el documento

Este error se produce cuando el indexador está intentando [proyectar datos en un almacén de conocimiento](knowledge-store-projection-overview.md) y se genera un error al hacerlo.  Este error puede ser constante y necesitar corrección, o puede ser un error transitorio del receptor de salida de la proyección. En este último caso, puede que tenga que esperar y volver a intentarlo para que se resuelva.  A continuación se muestra un conjunto de estados de error conocidos y posibles soluciones.

| Motivo | Detalles/ejemplo | Resolución |
| --- | --- | --- |
| No se pudo actualizar el BLOB de proyección `'blobUri'` en el contenedor `'containerName'` |El contenedor especificado no existe. | El indexador comprobará si el contenedor especificado se ha creado previamente y lo creará si es necesario, pero solo realizará esta comprobación una vez por cada ejecución de indexador. Este error significa que algo eliminó el contenedor después de este paso.  Para resolver este error, pruebe esto: deje solo la información de la cuenta de almacenamiento, espere a que finalice el indexador y vuelva a ejecutarlo. |
| No se pudo actualizar el BLOB de proyección `'blobUri'` en el contenedor `'containerName'` |No se pueden escribir los datos en la conexión de transporte: El host remoto forzó el cierre de la conexión existente. | Se considera que este es un error transitorio con Azure Storage y que, por tanto, para resolverlo debe volver a ejecutar el indexador. Si se produce este error de forma constante, registre una [incidencia de soporte técnico](https://ms.portal.azure.com/#create/Microsoft.Support) para que se pueda investigar en profundidad.  |
| No se pudo actualizar la fila `'projectionRow'` de la tabla `'tableName'` | El servidor está ocupado. | Se considera que este es un error transitorio con Azure Storage y que, por tanto, para resolverlo debe volver a ejecutar el indexador. Si se produce este error de forma constante, registre una [incidencia de soporte técnico](https://ms.portal.azure.com/#create/Microsoft.Support) para que se pueda investigar en profundidad.  |

<a name="could-not-execute-skill-because-a-skill-input-was-invalid"></a>

## <a name="warning-skill-input-was-invalid"></a>Advertencia: La entrada de aptitud no era válida
Faltaba una entrada para la aptitud, era de un tipo incorrecto o bien no era válida. El mensaje de advertencia indicará el impacto:
1) No se pudo ejecutar la aptitud
2) La aptitud se ejecutó, pero puede tener resultados inesperados

Las aptitudes cognitivas tienen entradas obligatorias y entradas opcionales. Por ejemplo, la [aptitud Extracción de frases clave](cognitive-search-skill-keyphrases.md) tiene dos entradas obligatorias `text`, `languageCode`, y no tienen ninguna entrada opcional. Las entradas de aptitudes personalizadas se consideran entradas opcionales.

Si faltan entradas necesarias o si alguna de estas no es el tipo correcto, se omite la aptitud y se genera una advertencia. Las aptitudes omitidas no generan ninguna salida, por lo que si otras aptitudes usan salidas de la aptitud omitida, podrían generar más advertencias.

Si falta una entrada opcional, la aptitud seguirá ejecutándose, pero puede producir una salida inesperada debido a que falta la entrada.

En ambos casos, se puede esperar esta advertencia debido a la forma de los datos. Por ejemplo, si tiene un documento que contiene información sobre personas con los campos `firstName`, `middleName`y `lastName`, es posible que tenga algunos documentos que no incluyan una entrada para `middleName`. Si va a pasar `middleName` como una entrada a una aptitud de la canalización, se espera que falte algo de tiempo en esta entrada de aptitud. Tendrá que evaluar los datos y el escenario para determinar si se requiere alguna acción como resultado de esta advertencia.

Si quiere proporcionar un valor predeterminado en caso de una entrada omitida, puede usar la [aptitud condicional](cognitive-search-skill-conditional.md) para generar un valor predeterminado y luego usar la salida de la [aptitud condicional](cognitive-search-skill-conditional.md) como la entrada de la aptitud.


```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document",
    "inputs": [
        { "name": "condition", "source": "= $(/document/language) == null" },
        { "name": "whenTrue", "source": "= 'en'" },
        { "name": "whenFalse", "source": "= $(/document/language)" }
    ],
    "outputs": [ { "name": "output", "targetName": "languageWithDefault" } ]
}
```

| Motivo | Detalles/ejemplo | Resolución |
| --- | --- | --- |
| La entrada de aptitud tiene un tipo incorrecto | "La entrada de aptitud obligatoria no era del tipo esperado `String`. Nombre: `text`; origen: `/document/merged_content`".  "La entrada de aptitud obligatoria no tenía el formato esperado. Nombre: `text`; origen: `/document/merged_content`".  "No se puede iterar en el elemento `/document/normalized_images/0/imageCelebrities/0/detail/celebrities`, que no es una matriz".  "No se puede seleccionar `0` en un elemento `/document/normalized_images/0/imageCelebrities/0/detail/celebrities`, que no es una matriz". | Ciertas aptitudes esperan entradas de tipos determinados, por ejemplo, la [aptitud Opinión](cognitive-search-skill-sentiment.md) espera que `text` sea una cadena. Si la entrada especifica un valor que no es de cadena, la aptitud no se ejecuta y no genera ninguna salida. Asegúrese de que el conjunto de datos tiene valores de entrada con un tipo uniforme, o bien use una [aptitud API web personalizada](cognitive-search-custom-skill-web-api.md) para procesar previamente la entrada. Si va a iterar la aptitud en una matriz, revise que la entrada y el contexto de la aptitud tengan `*` en las posiciones correctas. Por lo general, tanto el contexto como el origen de entrada deben finalizar con `*` para las matrices. |
| Falta la entrada de aptitud | "Falta la entrada de aptitud obligatoria. Nombre: `text`; origen: `/document/merged_content`". "Falta el valor `/document/normalized_images/0/imageTags`".  "No se puede seleccionar `0` en una matriz `/document/pages` de longitud `0`". | Si todos los documentos reciben esta advertencia, lo más probable es que haya un error ortográfico en las rutas de acceso de las entradas, por lo que debe revisar nuevamente el uso de mayúsculas y minúsculas en el nombre de la propiedad, ver si faltan o sobran `*` en la ruta de acceso y asegurarse de que los documentos del origen de datos proporcionan las entradas obligatorias. |
| La entrada de código de idioma de aptitud no es válida | La entrada de aptitud `languageCode` tiene los códigos de idioma siguientes `X,Y,Z`, uno de los cuales es al menos no válido. | Consulte más detalles [a continuación](cognitive-search-common-errors-warnings.md#skill-input-languagecode-has-the-following-language-codes-xyz-at-least-one-of-which-is-invalid). |

<a name="skill-input-languagecode-has-the-following-language-codes-xyz-at-least-one-of-which-is-invalid"></a>

## <a name="warning--skill-input-languagecode-has-the-following-language-codes-xyz-at-least-one-of-which-is-invalid"></a>Advertencia:  La entrada de aptitud "languageCode" tiene los siguientes códigos de idioma "X,Y,Z", uno de los cuales es al menos válido.
Uno o varios de los valores que pasan a la entrada `languageCode` opcional de una aptitud de nivel inferior no se admiten. Esta situación puede darse si se pasa la salida de [LanguageDetectionSkil](cognitive-search-skill-language-detection.md) a aptitudes posteriores y la salida consta de más idiomas de los que se admiten en esas aptitudes de nivel inferior.

Si sabe que el conjunto de datos solo tiene un idioma, debe quitar el elemento [LanguageDetectionSkill](cognitive-search-skill-language-detection.md) y la entrada de aptitud `languageCode` y usar en cambio el parámetro de aptitud `defaultLanguageCode` para esa aptitud, suponiendo que el idioma sea compatible con ella.

Si sabe que el conjunto de datos contiene varios idiomas y, por tanto, necesita el elemento [LanguageDetectionSkill](cognitive-search-skill-language-detection.md) y la entrada `languageCode`, considere agregar un elemento [ConditionalSkill](cognitive-search-skill-conditional.md) para filtrar el texto con los idiomas que no se admiten antes de pasar el texto a la aptitud de nivel inferior.  Este es un ejemplo de cuál podría ser el aspecto del elemento EntityRecognitionSkill:

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document",
    "inputs": [
        { "name": "condition", "source": "= $(/document/language) == 'de' || $(/document/language) == 'en' || $(/document/language) == 'es' || $(/document/language) == 'fr' || $(/document/language) == 'it'" },
        { "name": "whenTrue", "source": "/document/content" },
        { "name": "whenFalse", "source": "= null" }
    ],
    "outputs": [ { "name": "output", "targetName": "supportedByEntityRecognitionSkill" } ]
}
```

Estas son algunas referencias de los idiomas admitidos actualmente en cada una de las aptitudes que pueden producir este mensaje de error:
* [Idiomas compatibles con Text Analytics](../cognitive-services/text-analytics/language-support.md) (para [KeyPhraseExtractionSkill](cognitive-search-skill-keyphrases.md), [EntityRecognitionSkill](cognitive-search-skill-entity-recognition.md), [SentimentSkill](cognitive-search-skill-sentiment.md) y [PIIDetectionSkill](cognitive-search-skill-pii-detection.md))
* [Idiomas compatibles con Translator](../cognitive-services/translator/language-support.md) (para [Text TranslationSkill](cognitive-search-skill-text-translation.md))
* Idiomas compatibles con [Text SplitSkill](cognitive-search-skill-textsplit.md): `da, de, en, es, fi, fr, it, ko, pt`

<a name="skill-input-was-truncated"></a>

## <a name="warning-skill-input-was-truncated"></a>Advertencia: La entrada de aptitud se truncó
Las aptitudes cognitivas tienen límites en cuanto a la longitud del texto que se puede analizar a la vez. Si la entrada de texto de estas aptitudes está por encima de ese límite, se truncará el texto para cumplir el límite y, luego, se realizará el enriquecimiento sobre ese texto truncado. Esto significa que la aptitud se ejecuta, pero no sobre todos los datos.

En el ejemplo de LanguageDetectionSkill que se indica a continuación, el campo de entrada `'text'` puede desencadenar esta advertencia si está por encima del límite de caracteres. Puede encontrar los límites de entrada de aptitudes en la [documentación de las aptitudes](cognitive-search-predefined-skills.md).

```json
 {
    "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
    "inputs": [
      {
        "name": "text",
        "source": "/document/text"
      }
    ],
    "outputs": [...]
  }
```

Si quiere asegurarse de que se analice todo el texto, considere la posibilidad de usar la [aptitud dividida](cognitive-search-skill-textsplit.md).

<a name="web-api-skill-response-contains-warnings"></a>

## <a name="warning-web-api-skill-response-contains-warnings"></a>Advertencia: La respuesta de la aptitud API web contiene advertencias
Los indexadores podían ejecutar una aptitud del conjunto de aptitudes, pero la respuesta de la solicitud de API web indicaban que había advertencias durante la ejecución. Revise las advertencias para entender cómo se ven afectados los datos y si es necesario realizar alguna acción o no.

<a name="the-current-indexer-configuration-does-not-support-incremental-progress"></a>

## <a name="warning-the-current-indexer-configuration-does-not-support-incremental-progress"></a>Advertencia: La configuración del indexador actual no admite el progreso incremental

Esta advertencia solo se produce para los orígenes de datos de Cosmos DB.

El progreso incremental durante la indexación garantiza que si se interrumpe la ejecución del indexador por errores transitorios o por el límite de tiempo de ejecución, dicho indexador puede retomar la ejecución por donde la dejó la próxima vez que se ejecute, en lugar de tener que volver a indexar toda la colección desde el principio. Esto es especialmente importante cuando se indexan colecciones grandes.

La capacidad de reanudar un trabajo de indexación sin terminar se basa en tener documentos ordenados por la columna de `_ts`. El indexador usa la marca de tiempo para determinar qué documento se debe seleccionar a continuación. Si falta la columna de `_ts` o si el indexador no puede determinar si se ha ordenado una consulta personalizada, el indexador comienza al principio y usted verá esta advertencia.

Es posible invalidar este comportamiento, habilitar el progreso incremental y suprimir esta advertencia si usa la propiedad de configuración `assumeOrderByHighWatermarkColumn`.

Para más información, consulte [Progreso incremental y consultas personalizadas](search-howto-index-cosmosdb.md#IncrementalProgress).

<a name="some-data-was-lost-during projection-row-x-in-table-y-has-string-property-z-which-was-too-long"></a>

## <a name="warning-some-data-was-lost-during-projection-row-x-in-table-y-has-string-property-z-which-was-too-long"></a>Advertencia: Algunos datos se perdieron durante la proyección. La fila 'X' de la tabla 'Y' tiene la propiedad de cadena 'Z', que era demasiado larga.

El [servicio de Table Storage](https://azure.microsoft.com/services/storage/tables) tiene límites en cuanto a qué tan grandes pueden ser las [propiedades de las entidades](/rest/api/storageservices/understanding-the-table-service-data-model#property-types). Las cadenas pueden tener 32.000 caracteres como máximo. Si una fila con una propiedad de cadena de más de 32.000 caracteres se está proyectando, solo se conservan los primeros 32.000 caracteres. Para evitar este problema, evite proyectar filas con propiedades de cadena de más de 32.000 caracteres.

<a name="truncated-extracted-text-to-x-characters"></a>

## <a name="warning-truncated-extracted-text-to-x-characters"></a>Advertencia: El texto extraído se ha truncado a X caracteres
Los indexadores limitan la cantidad de texto que se puede extraer de cualquier documento. Este límite depende del plan de tarifa: 32 000 caracteres para el nivel Gratis, 64 000 para Básico, 4 millones para Estándar, 8 millones para Estándar S2 y 16 millones para Estándar S3. El texto que se ha truncado no se indexará. Para evitar esta advertencia, intente dividir los documentos con grandes cantidades de texto en varios documentos más pequeños. 

Para más información, consulte [Límites de índice](search-limits-quotas-capacity.md#indexer-limits).

<a name="could-not-map-output-field-x-to-search-index"></a>

## <a name="warning-could-not-map-output-field-x-to-search-index"></a>Advertencia: No se pudo asignar el campo de salida "X" al índice de búsqueda
Las asignaciones de campos de salida que hagan referencia a datos inexistentes o nulos generarán advertencias con cada documento y producirán un campo de índice vacío. Para solucionar este problema, compruebe las rutas de acceso de origen de la asignación de campos de salida en busca de posibles errores tipográficos o establezca un valor predeterminado mediante la [aptitud condicional](cognitive-search-skill-conditional.md#sample-skill-definition-2-set-a-default-value-for-a-value-that-doesnt-exist). Consulte [Asignación de campos de salida](cognitive-search-output-field-mapping.md) para obtener más información.

| Motivo | Detalles/ejemplo | Resolución |
| --- | --- | --- |
| No se puede iterar en un elemento que no sea una matriz | "No se puede iterar en el elemento `/document/normalized_images/0/imageCelebrities/0/detail/celebrities`, que no es una matriz". | Este error se produce cuando la salida no es una matriz. Si cree que la salida es una matriz, compruebe si hay errores en la ruta de acceso del campo de origen de salida indicada. Por ejemplo, es posible que tenga un elemento `*` adicional o ausente en el nombre del campo de origen. También es posible que la entrada a esta aptitud sea NULL, lo que da lugar a una matriz vacía. Busque detalles similares en la sección [La entrada de aptitudes no es válida](cognitive-search-common-errors-warnings.md#warning-skill-input-was-invalid).    |
| No se puede seleccionar `0` en un elemento que no sea una matriz | "No se puede seleccionar `0` en un elemento que no sea una matriz `/document/pages`". | Esto puede ocurrir si la salida de las aptitudes no produce una matriz y el nombre del campo de origen de salida tiene un índice de matriz o `*` en su ruta de acceso. Compruebe las rutas de acceso proporcionadas en los nombres del campo de origen de salida y en el valor de campo del nombre de campo indicado. Busque detalles similares en la sección [La entrada de aptitudes no es válida](cognitive-search-common-errors-warnings.md#warning-skill-input-was-invalid).  |

<a name="the-data-change-detection-policy-is-configured-to-use-key-column-x"></a>

## <a name="warning-the-data-change-detection-policy-is-configured-to-use-key-column-x"></a>Advertencia: La directiva de detección de cambios de datos está configurada para usar la columna de clave 'X'
[Las directivas de detección de cambios de datos](/rest/api/searchservice/create-data-source#data-change-detection-policies) tienen requisitos específicos para las columnas que usan para detectar cambios. Uno de estos requisitos es que esta columna se actualice cada vez que se cambia el elemento de origen. Otro requisito es que el nuevo valor de esta columna sea mayor que el valor anterior. Las columnas de clave no cumplen este requisito porque no cambian en cada actualización. Para solucionar este problema, seleccione una columna diferente para la directiva de detección de cambios.

<a name="document-text-appears-to-be-utf-16-encoded-but-is-missing-a-byte-order-mark"></a>

## <a name="warning-document-text-appears-to-be-utf-16-encoded-but-is-missing-a-byte-order-mark"></a>Advertencia: El texto del documento parece estar codificado en UTF-16, pero le falta una marca BOM.

Los [modos de análisis del indexador](/rest/api/searchservice/create-indexer#blob-configuration-parameters) necesitan saber cómo se codifica el texto antes de analizarlo. Las dos formas más comunes de codificar texto son UTF-16 y UTF-8. UTF-8 es una codificación de longitud variable en la que cada carácter tiene entre 1 byte y 4 bytes de longitud. UTF-16 es una codificación de longitud fija en la que cada carácter tiene una longitud de 2 bytes. UTF-16 tiene dos variantes diferentes, "big endian" y "little endian". La codificación de texto viene determinada por una "marca BOM", una serie de bytes antes del texto.

| Encoding | Marca BOM |
| --- | --- |
| UTF-16 Big Endian | 0xFE 0xFF |
| UTF-16 Little Endian | 0xFF 0xFE |
| UTF-8 | 0xEF 0xBB 0xBF |

Si no está presente ninguna marca BOM, se asume que el texto está codificado como UTF-8.

Para solucionar esta advertencia, determine cuál es la codificación de texto para este blob y agregue la marca BOM adecuada.

<a name="cosmos-db-collection-has-a-lazy-indexing-policy"></a>

## <a name="warning-cosmos-db-collection-x-has-a-lazy-indexing-policy-some-data-may-be-lost"></a>Advertencia: La colección "X" de Cosmos DB tiene una directiva de indexación diferida. Se podrían perder algunos datos

Las colecciones con directivas de indexación [diferida](/azure/cosmos-db/index-policy#indexing-mode) no se pueden consultar de forma coherente, lo que hace que su indizador pierda datos. Para evitar esta advertencia, cambie su directiva de indexación a Coherente.