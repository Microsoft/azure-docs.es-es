---
title: Información general sobre similitud y puntuación
titleSuffix: Azure Cognitive Search
description: En este tema se explican los conceptos de similitud y puntuación, y lo que un desarrollador puede hacer para personalizar el resultado de la puntuación.
manager: nitinme
author: luiscabrer
ms.author: luisca
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 300da87ecff13fc160ec08684cf1d032f9a19f71
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88924493"
---
# <a name="similarity-and-scoring-in-azure-cognitive-search"></a>Similitud y puntuación en Azure Cognitive Search

La puntuación hace referencia al cálculo de una puntuación de búsqueda de todos los artículos devueltos en los resultados de la búsqueda de las consultas de búsqueda de texto completo. La puntuación es un indicador de la importancia de un elemento en el contexto de la operación de búsqueda actual. Cuanto mayor sea la puntuación, mayor importancia tendrá el elemento. En los resultados de búsqueda, los artículos están ordenados de mayor a menor, según las puntuaciones de búsqueda calculadas para cada uno. 

De forma predeterminada, se devuelven los 50 mejores en la respuesta, pero puede usar el parámetro **$top** para devolver un número mayor o menor de elementos (hasta 1000 en una sola respuesta) y **$skip** para obtener el siguiente conjunto de resultados.

La puntuación de búsqueda se calcula en función de las propiedades estadísticas de los datos y la consulta. Azure Cognitive Search busca documentos que coinciden con los términos de búsqueda (algunos o todos, dependiendo de [searchMode](/rest/api/searchservice/search-documents#searchmodeany--all-optional)), lo cual favorece a los documentos que contienen muchas instancias del término de búsqueda. La puntuación de búsqueda aumenta todavía más si el término es poco frecuente en el índice de datos, pero común dentro del documento. La base de este enfoque de relevancia informática se denomina *TF-IDF* o frecuencia de documento inverso de la frecuencia del término.

Los valores de puntuación de búsqueda pueden repetirse a lo largo de un conjunto de resultados. Cuando varios resultados tienen la misma puntuación de búsqueda, el orden de estos elementos puntuados no se define y no es estable. Vuelva a ejecutar la consulta y podrá ver que los elementos cambian de posición, especialmente si usa el servicio gratuito o un servicio facturable con varias réplicas. Si dos elementos disponen de la misma puntuación, no hay ninguna garantía de cuál aparecerá en primer lugar.

Si desea romper el vínculo entre las puntuaciones repetidas, puede agregar una cláusula **$orderby** para ordenar primero por puntuación y, a continuación, ordenar por otro campo que se pueda ordenar (por ejemplo, `$orderby=search.score() desc,Rating desc`). Para obtener más información, consulte [$orderby](./search-query-odata-orderby.md).

> [!NOTE]
> `@search.score = 1.00` indica un conjunto de resultados sin puntuar o sin clasificar. La puntuación es uniforme en todos los resultados. Los resultados sin puntuar se producen cuando el formulario de consulta es una búsqueda aproximada, una consulta con caracteres comodín o regex, o una expresión **$filter**. 

## <a name="scoring-profiles"></a>Perfiles de puntuación

Puede personalizar la forma en que se clasifican los distintos campos mediante la definición de un *perfil de puntuación* personalizado. Los perfiles de puntuación proporcionan un mayor control sobre la clasificación de los elementos en los resultados de búsqueda. Por ejemplo, desea aumentar los elementos según su potencial de ingresos, promover elementos más recientes o quizás aumentar los elementos que han permanecido en inventario demasiado largo. 

Un perfil de puntuación es parte de la definición del índice que se compone de campos, funciones y parámetros ponderados. Para obtener más información sobre cómo definir uno, consulte [Perfiles de puntuación](index-add-scoring-profiles.md).

<a name="scoring-statistics"></a>

## <a name="scoring-statistics-and-sticky-sessions"></a>Estadísticas de puntuación y sesiones permanentes

Para ofrecer escalabilidad, Azure Cognitive Search distribuye cada índice horizontalmente a través de un proceso de particionamiento, lo que significa que las partes de un índice están físicamente separadas.

De forma predeterminada, la puntuación de un documento se calcula en función de las propiedades estadísticas de los datos *de una partición*. Este enfoque no suele ser un problema para una gran corpus de datos y proporciona un mejor rendimiento que el cálculo de la puntuación basado en la información de todas las particiones. Dicho esto, el uso de esta optimización del rendimiento puede provocar que dos documentos muy similares (o incluso idénticos) terminen con puntuaciones de relevancia diferentes si acaban en diferentes particiones.

Si prefiere calcular la puntuación en función de las propiedades estadísticas de todas las particiones, para hacerlo puede agregar *scoringStatistics=global* como un [parámetro de consulta](/rest/api/searchservice/search-documents) (o agregar *"scoringStatistics": "global"* como un parámetro del cuerpo de la [solicitud de consulta](/rest/api/searchservice/search-documents)).

```http
GET https://[service name].search.windows.net/indexes/[index name]/docs?scoringStatistics=global&api-version=2020-06-30&search=[search term]
  Content-Type: application/json
  api-key: [admin or query key]  
```
El uso de scoringStatistics garantizará que todas las particiones de la misma réplica proporcionen los mismos resultados. Dicho esto, diferentes réplicas pueden ser ligeramente distintas entre sí, ya que siempre se actualizan con los cambios más recientes en el índice. En algunos casos, puede que desee que los usuarios obtengan resultados más coherentes durante una "sesión de consulta". En esos casos, puede proporcionar un `sessionId` como parte de las consultas. El `sessionId` es una cadena única que se crea para hacer referencia a una sesión de usuario única.

```http
GET https://[service name].search.windows.net/indexes/[index name]/docs?sessionId=[string]&api-version=2020-06-30&search=[search term]
  Content-Type: application/json
  api-key: [admin or query key]  
```
Siempre y cuando se use el mismo `sessionId`, se intentará en lo posible dirigirse a la misma réplica, lo que aumentará la coherencia de los resultados que verán los usuarios. 

> [!NOTE]
> Reutilizar los mismos valores de `sessionId` repetidamente puede interferir en el equilibrio de carga de las solicitudes entre réplicas y afectar negativamente al rendimiento del servicio de búsqueda. El valor utilizado como sessionId no puede empezar con el carácter "_".

## <a name="similarity-ranking-algorithms"></a>Algoritmo de clasificación de similitud

Azure Cognitive Search admite dos algoritmos de clasificación de similitud diferentes: un algoritmo de *similitud clásico* y la implementación oficial del algoritmo *Okapi BM25* (actualmente en versión preliminar). El algoritmo de similitud clásico es el predeterminado, pero los nuevos servicios creados a partir del 15 de julio usarán el nuevo algoritmo BM25. Será el único algoritmo disponible en los nuevos servicios.

Por ahora, puede especificar el algoritmo de clasificación de similitud que le gustaría usar. Para obtener más información, consulte [Algoritmo de clasificación](index-ranking-similarity.md).

El siguiente segmento de vídeo avanza rápidamente para mostrar una explicación de cómo funciona el procesamiento de texto en Azure Cognitive Search. Puede ver el vídeo completo para obtener más información.

> [!VIDEO https://www.youtube.com/embed/Y_X6USgvB1g?version=3&start=322&end=643]

<a name="featuresMode-param"></a>

## <a name="featuresmode-parameter-preview"></a>Parámetro featuresMode (versión preliminar)

Las solicitudes para [buscar documentos](/rest/api/searchservice/preview-api/search-documents) tienen un nuevo parámetro [featuresMode](/rest/api/searchservice/preview-api/search-documents#featuresmode) que puede proporcionar detalles adicionales sobre la relevancia en el nivel de campo. Mientras que `@searchScore` se calcula para todo el documento (según corresponda, en el contexto de esta consulta), a través de featuresMode puede obtener información sobre los campos individuales, tal y como se expresa en una estructura de `@search.features`. La estructura contiene todos los campos usados en la consulta (campos específicos mediante **searchFields** en una consulta o todos los campos asignados como que **permite búsquedas** en un índice). Para cada campo, obtiene los valores siguientes:

+ Número de tokens únicos encontrados en el campo
+ Puntuación de similitud, o una medida de la similitud del contenido del campo con respecto al término de consulta
+ Frecuencia del término, o el número de veces que el término de la consulta se encontró en el campo

Para una consulta que tiene como destino los campos "descripción" y "título", una respuesta que incluye `@search.features` podría ser similar a la siguiente:

```json
"value": [
 {
    "@search.score": 5.1958685,
    "@search.features": {
        "description": {
            "uniqueTokenMatches": 1.0,
            "similarityScore": 0.29541412,
            "termFrequency" : 2
        },
        "title": {
            "uniqueTokenMatches": 3.0,
            "similarityScore": 1.75451557,
            "termFrequency" : 6
        }
```

Puede utilizar estos puntos de datos en las [soluciones de puntuación personalizadas](https://github.com/Azure-Samples/search-ranking-tutorial) o utilizar la información para depurar los problemas de relevancia de búsqueda.


## <a name="see-also"></a>Consulte también

 [Perfiles de puntuación](index-add-scoring-profiles.md) [Referencia de la API de REST](/rest/api/searchservice/)   
 [API de búsqueda de documentos](/rest/api/searchservice/search-documents)   
 [SDK de .NET de Azure Cognitive Search](/dotnet/api/overview/azure/search?view=azure-dotnet)