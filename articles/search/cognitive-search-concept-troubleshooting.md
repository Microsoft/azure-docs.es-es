---
title: Sugerencias para el diseño del enriquecimiento con IA
titleSuffix: Azure Cognitive Search
description: Sugerencias y solución de problemas para configurar las canalizaciones de enriquecimiento con inteligencia artificial en Azure Cognitive Search.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/08/2020
ms.openlocfilehash: 8347ca5a33790d0b35176be47a0fa4811a19e3f1
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935473"
---
# <a name="tips-for-ai-enrichment-in-azure-cognitive-search"></a>Sugerencias para el enriquecimiento con IA en Azure Cognitive Search

En este artículo encontrará una lista de sugerencias y trucos para avanzar sin pausa a medida que empieza a usar las funcionalidades de enriquecimiento con IA en Azure Cognitive Search. 

Si aún no lo ha hecho, siga las instrucciones de [Tutorial: Obtenga información sobre cómo llamar a las API de enriquecimiento con IA](cognitive-search-quickstart-blob.md) para practicar la aplicación de los enriquecimientos con IA a un origen de datos de blobs.

## <a name="tip-1-start-with-a-small-dataset"></a>Sugerencia 1: Comience con un pequeño conjunto de datos.
La mejor forma de detectar problemas rápidamente es aumentar la velocidad de solución de esos problemas. La mejor forma de reducir el tiempo de indexación es reducir el número de documentos que se indexarán. 

Comience creando un origen de datos con un pequeño conjunto de documentos y registros. La muestra de documentos debe ser una representación adecuada de la variedad de documentos que se indexarán. 

Ejecute la muestra de documentos mediante la canalización de un extremo a otro, y compruebe que los resultados satisfagan sus necesidades. Una vez que esté satisfecho con los resultados, puede agregar más archivos a la fuente de datos.

## <a name="tip-2-make-sure-your-data-source-credentials-are-correct"></a>Sugerencia 2: Asegúrese de que las credenciales del origen de datos sean correctas.
La conexión del origen de datos no se validará hasta que se defina un indexador que la use. Si ve algún error que indique que el indexador no puede acceder a los datos, asegúrese de que:
- La cadena de conexión es correcta. Igualmente, cuando cree tokens de SAS, asegúrese de usar el formato que espera Azure Cognitive Search. Consulte [la sección de especificación de credenciales ](
https://docs.microsoft.com/azure/search/search-howto-indexing-azure-blob-storage#how-to-specify-credentials) para conocer los diferentes formatos compatibles.
- El nombre del contenedor en el indexador es correcto.

## <a name="tip-3-see-what-works-even-if-there-are-some-failures"></a>Sugerencia 3: Confirme los elementos que funcionan incluso si hay algunos errores.
A veces, un pequeño error detiene el proceso del indexador. Esto no supone ningún problema si planea solucionar los errores uno por uno. Sin embargo, es posible que quiera ignorar un tipo particular de error, permitiendo así que el indexador continúe con el proceso para poder ver qué flujos están funcionando realmente.

En ese caso, le puede indicar al indexador que ignore los errores. Para ello, establezca los valores de *maxFailedItems* y *maxFailedItemsPerBatch* en -1, como parte de la definición del indexador.

```
{
  "// rest of your indexer definition
   "parameters":
   {
      "maxFailedItems":-1,
      "maxFailedItemsPerBatch":-1
   }
}
```
> [!NOTE]
> Como procedimiento recomendado, establezca maxFailedItems, maxFailedItemsPerBatch en 0 para cargas de trabajo de producción.

## <a name="tip-4-use-debug-sessions-to-identify-and-resolve-issues-with-your-skillset"></a>Sugerencia 4: Usar sesiones de depuración para identificar y resolver problemas con el conjunto de aptitudes 

Sesiones de depuración es un editor visual que funciona con un conjunto de aptitudes existente en Azure Portal. En una sesión de depuración, puede identificar y resolver errores, validar cambios y confirmar cambios en un conjunto de aptitudes en la canalización de enriquecimiento con IA. Se trata de una característica de vista previa [leer la documentación](./cognitive-search-debug-session.md). Para obtener más información sobre los conceptos y la introducción, vea [Sesiones de depuración](./cognitive-search-tutorial-debug-sessions.md).

Las sesiones de depuración que funcionan en un único documento son una excelente manera de crear de forma iterativa canalizaciones de enriquecimiento más complejas.

## <a name="tip-5-looking-at-enriched-documents-under-the-hood"></a>Sugerencia 5: Examine los documentos enriquecidos en segundo plano. 
Los documentos enriquecidos son estructuras temporales que se crean durante el enriquecimiento, y que se eliminan cuando se completa el proceso.

Para capturar una instantánea del documento enriquecido creado durante la indexación, agregue un campo denominado ```enriched``` al índice. El indexador vuelca automáticamente en el campo una representación de cadena del enriquecimiento del documento.

El campo ```enriched``` contendrá una cadena que es una representación lógica del documento enriquecido en memoria en formato JSON.  Sin embargo, el valor del campo es un documento JSON válido. Las comillas son caracteres de escape, por lo que deberá reemplazar `\"` por `"` para ver el documento con el formato JSON. 

El campo enriquecido está destinado para realizar únicamente operaciones de depuración, que le ayudarán a entender la forma lógica del contenido en el que se evalúan las expresiones. No debe depender de este campo para realizar la indexación.

Agregue un campo ```enriched``` como parte de la definición del índice para fines de depuración:

#### <a name="request-body-syntax"></a>Sintaxis del cuerpo de la solicitud
```json
{
  "fields": [
    // other fields go here.
    {
      "name": "enriched",
      "type": "Edm.String",
      "searchable": false,
      "sortable": false,
      "filterable": false,
      "facetable": false
    }
  ]
}
```

## <a name="tip-6-expected-content-fails-to-appear"></a>Sugerencia 6: El contenido esperado no aparece.

Si falta contenido, esto puede deberse a que se descartaron documentos durante la indexación. Los niveles Gratuito y Básico tienen límites bajos en lo que respecta al tamaño de los documentos. Cualquier archivo que exceda el límite se descarta durante la indexación. Puede comprobar si hay documentos descartados en Azure Portal. En el panel del servicio de búsqueda, haga doble clic en el icono Indexadores. Revise la proporción de documentos que se indexaron con éxito. Si no es del 100 %, puede hacer clic en la proporción para obtener más detalles. 

Si el problema está relacionado con el tamaño del archivo, es posible que vea un error como este: "El blob \<file-name> tiene un tamaño de \<file-size> bytes, lo cual excede el tamaño máximo para la extracción de documentos de su nivel de servicio actual". Para obtener más información, consulte los [límites del servicio](search-limits-quotas-capacity.md).

Una segunda razón por la que el contenido no aparece, puede deberse errores de asignación de entradas y salidas relacionadas. Por ejemplo, el nombre de destino de salida es "Personas", pero el nombre del campo de índice es "personas" en minúsculas. El sistema puede devolver 201 mensajes de éxito de toda la canalización y hacerle creer que la indexación tuvo éxito, cuando en realidad uno de los campos está vacío. 

## <a name="tip-7-extend-processing-beyond-maximum-run-time-24-hour-window"></a>Sugerencia 7: Amplíe el procesamiento más allá del tiempo máximo de ejecución (periodo de 24 horas).

El análisis de imágenes es un proceso intensivo a nivel computacional, incluso cuando se trata de casos simples; debido a ello, cuando las imágenes son especialmente grandes o complejas, los tiempos de procesamiento pueden exceder el tiempo máximo permitido. 

El tiempo máximo de ejecución varía según el nivel: varios minutos en el nivel gratuito, y una indexación de 24 horas en niveles de pago. Si el procesamiento no se completa dentro de un período de 24 horas según el procesamiento bajo demanda, use una programación en la que el indexador pueda retomar el procesamiento desde donde lo dejó. 

En cuanto a los indexadores programados, la indexación se reanuda según la programación del último documento válido conocido. Al usar una programación recurrente, el indexador puede abrirse camino a través de las imágenes pendientes durante una serie de horas o días, hasta que se procesen todas aquellas imágenes que no estén procesadas. Para más información acerca de la sintaxis de programación, consulte [Paso 3: Cree un indexador](search-howto-indexing-azure-blob-storage.md#step-3-create-an-indexer) o consulte [Programación de indexadores para Azure Cognitive Search](search-howto-schedule-indexers.md).

> [!NOTE]
> Si un indexador se establece en una programación determinada pero se produce repetidamente un error en el mismo documento una y otra vez cada vez se ejecuta, el indexador comenzará a ejecutarse en un intervalo menos frecuente (hasta un máximo de al menos una vez cada 24 horas) hasta que vuelva a avanzar correctamente.  Si cree que solucionó el problema que hacía que el indexador se bloqueara en un punto determinado, puede realizar una ejecución a petición del indexador y, si avanza correctamente, el indexador volverá a su intervalo de programación establecido.

Si realiza una indexación basada en el portal (tal como se describe en la guía de inicio rápido), la elección de la opción del indexador "ejecutar una vez" limita el procesamiento a 1 hora (`"maxRunTime": "PT1H"`). Es posible que quiera extender el período de procesamiento para que sea algo más largo.

## <a name="tip-8-increase-indexing-throughput"></a>Sugerencia 8: Aumente el rendimiento de la indexación.

Para realizar una [indexación paralela](search-howto-large-index.md), coloque los datos en varios contenedores o carpetas virtuales múltiples dentro del mismo contenedor. A continuación, cree varios pares de orígenes de datos e indexadores. Todos los indexadores pueden usar el mismo conjunto de aptitudes y escribir en el mismo índice de búsqueda de destino, por lo que la aplicación de búsqueda no necesita conocer esta partición.
Para obtener más información, consulte [Indexing Large Datasets](search-howto-indexing-azure-blob-storage.md#indexing-large-datasets) (Indexar grandes conjuntos de datos).

## <a name="see-also"></a>Consulte también
+ [Inicio rápido: Cree una canalización de enriquecimiento con IA en el portal](cognitive-search-quickstart-blob.md)
+ [Tutorial: Obtenga información sobre las API REST de enriquecimiento con IA](cognitive-search-tutorial-blob.md)
+ [Especificar las credenciales de origen de datos.](search-howto-indexing-azure-blob-storage.md#how-to-specify-credentials)
+ [Indexar grandes conjuntos de datos](search-howto-indexing-azure-blob-storage.md#indexing-large-datasets)
+ [Definición de un conjunto de aptitudes](cognitive-search-defining-skillset.md)
+ [Asignar campos enriquecidos a un índice](cognitive-search-output-field-mapping.md)