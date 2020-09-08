---
title: Procedimientos para trabajar con los resultados de búsqueda
titleSuffix: Azure Cognitive Search
description: Estructure y ordene los resultados de búsqueda, obtenga el número de documentos e incorpore la navegación por el contenido a los resultados de búsqueda de Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/01/2020
ms.openlocfilehash: 08641814e2a4fdf6f174f94b1e38e4124cf531d0
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934929"
---
# <a name="how-to-work-with-search-results-in-azure-cognitive-search"></a>Procedimientos para trabajar con los resultados de búsqueda en Azure Cognitive Search

En este artículo se explica cómo obtener la respuesta a una consulta, donde se devuelve el recuento total de documentos coincidentes, resultados paginados, resultados ordenados y términos con resaltado de referencias.

La estructura de una respuesta viene determinada por los parámetros de la consulta: [Documento de búsqueda](/rest/api/searchservice/Search-Documents) en la API de REST o [clase DocumentSearchResult](/dotnet/api/microsoft.azure.search.models.documentsearchresult-1) en el SDK de .NET.

## <a name="result-composition"></a>Redacción de los resultados

Aunque un documento de búsqueda puede constar de un gran número de campos, normalmente solo se necesitan unos pocos para representar cada documento en el conjunto de resultados. En una solicitud de consulta, anexe `$select=<field list>` para especificar los campos que aparecen en la respuesta. Un campo se debe atribuir como **Recuperable** en el índice que se va a incluir en un resultado. 

Entre los campos que funcionan mejor se incluyen aquellos que contrastan y diferencian entre documentos, y proporcionan información suficiente para invitar a una respuesta de navegación a través de clics de parte del usuario. En un sitio de comercio electrónico, puede tratarse del nombre de un producto, la descripción, la marca, el color, el tamaño, el precio y la clasificación. En el ejemplo integrado hotels-sample-index, podría tratarse de campos en el ejemplo siguiente:

```http
POST /indexes/hotels-sample-index/docs/search?api-version=2020-06-30 
    {  
      "search": "sandy beaches",
      "select": "HotelId, HotelName, Description, Rating, Address/City"
      "count": true
    }
```

> [!NOTE]
> Si quiere incluir archivos de imagen en un resultado, como una foto o logotipo del producto, almacénelos fuera de Azure Cognitive Search, pero incluya un campo en el índice para hacer referencia a la dirección URL de la imagen en el documento de búsqueda. Los índices de ejemplo que admiten imágenes en los resultados son la demostración **realestate-sample-us**, que se incluyen en esta guía de [inicio rápido](search-create-app-portal.md)y la [aplicación de demostración de trabajos de la ciudad de Nueva York](https://aka.ms/azjobsdemo).

## <a name="paging-results"></a>Paginación de resultados

De forma predeterminada, el motor de búsqueda devuelve hasta las primeras 50 coincidencias, según lo determine la puntuación de búsqueda si la consulta es una búsqueda de texto completo, o en un orden arbitrario para las consultas de coincidencia exacta.

Para devolver un número diferente de documentos coincidentes, agregue los parámetros `$top` y `$skip` a la solicitud de consulta. En la lista siguiente se explica la lógica.

+ Agregue `$count=true` para obtener un recuento del número total de documentos coincidentes dentro de un índice.

+ Devuelve el primer conjunto de 15 documentos coincidentes más un recuento total de coincidencias: `GET /indexes/<INDEX-NAME>/docs?search=<QUERY STRING>&$top=15&$skip=0&$count=true`.

+ Devuelve el segundo conjunto, omitiendo los 15 primeros para obtener los 15 siguientes: `$top=15&$skip=15`. Haga lo mismo para el tercer conjunto de 15: `$top=15&$skip=30`.

No se garantiza que los resultados de las consultas paginadas sean estables si el índice subyacente cambia. La paginación cambia el valor de `$skip` para cada página, pero cada consulta es independiente y funciona en la vista actual de los datos tal como existen en el índice en el momento de la consulta (es decir, no se almacenan en caché ni se hacen instantáneas de los resultados, como los que se encuentran en una base de datos de uso general).
 
A continuación, encontrará un ejemplo de cómo podría obtener duplicados. Supongamos que tiene un índice con cuatro documentos:

```text
{ "id": "1", "rating": 5 }
{ "id": "2", "rating": 3 }
{ "id": "3", "rating": 2 }
{ "id": "4", "rating": 1 }
```
 
Ahora supongamos que desea que los resultados se devuelvan dos a la vez, ordenados por calificación. Ejecutaría esta consulta para obtener la primera página de resultados: `$top=2&$skip=0&$orderby=rating desc`, lo que produce los siguientes resultados:

```text
{ "id": "1", "rating": 5 }
{ "id": "2", "rating": 3 }
```
 
En el servicio, suponga que se agrega un quinto documento al índice entre las llamadas de consulta: `{ "id": "5", "rating": 4 }`.  Poco después, ejecuta una consulta para capturar la segunda página: `$top=2&$skip=2&$orderby=rating desc`, y obtiene estos resultados:

```text
{ "id": "2", "rating": 3 }
{ "id": "3", "rating": 2 }
```
 
Observe que el documento 2 se captura dos veces. Esto se debe a que el nuevo documento 5 tiene un valor mayor de calificación, por lo que se ordena antes del documento 2 y se sitúa en la primera página. Si bien este comportamiento podría ser inesperado, es típico de cómo se comporta un motor de búsqueda.

## <a name="ordering-results"></a>Ordenación de los resultados

En el caso de las consultas de búsqueda de texto completo, los resultados se clasifican automáticamente mediante una puntuación de búsqueda, calculada en función de la frecuencia de los términos y la proximidad en un documento, donde los documentos con más coincidencias o más exactas de un término de búsqueda reciben las puntuaciones más altas. 

Las puntuaciones de búsqueda transmiten el sentido general de pertinencia, lo que refleja la exactitud de la coincidencia en comparación con otros documentos en el mismo conjunto de resultados. Las puntuaciones no siempre son coherentes de una consulta a la siguiente, por lo que, al trabajar con consultas, es posible que observe pequeñas discrepancias en cómo se ordenan los documentos de búsqueda. Hay varias explicaciones de por qué puede ocurrir esto.

| Causa | Descripción |
|-----------|-------------|
| Volatilidad de los datos | El contenido del índice varía a medida que se agregan, modifican o eliminan documentos. Las frecuencias de los términos cambiarán a medida que se procesen las actualizaciones del índice a lo largo del tiempo, lo que afecta a las puntuaciones de búsqueda de los documentos coincidentes |
| Varias réplicas | En el caso de los servicios que usan varias réplicas, las consultas se emiten para cada réplica en paralelo. Las estadísticas del índice usadas para calcular una puntuación de búsqueda se calculan por réplica, donde los resultados se combinan y ordenan en la respuesta de la consulta. Las réplicas son principalmente reflejos unas de otras, pero las estadísticas pueden diferir debido a pequeñas diferencias en el estado. Por ejemplo, una réplica podría tener documentos eliminados que contribuyan a sus estadísticas, que se combinaron a partir de otras réplicas. Normalmente, las diferencias en las estadísticas por réplica son más evidentes en los índices más pequeños. |
| Puntuaciones idénticas | Si varios documentos tienen la misma puntuación, cualquiera de ellos podría aparecer en primer lugar.  |

### <a name="consistent-ordering"></a>Ordenación coherente

Dada la flexibilidad en el orden de los resultados, es posible que desee explorar otras opciones si la coherencia es un requisito de la aplicación. El enfoque más sencillo es ordenar por un valor de campo, como una calificación o fecha. Para los escenarios en los que desea ordenar por un campo específico, como una calificación o fecha, puede definir de manera explícita una [expresión `$orderby`](query-odata-filter-orderby-syntax.md), que se puede aplicar a cualquier campo que se indexe como **Ordenable**.

Otra opción es usar un [perfil de puntuación personalizado](index-add-scoring-profiles.md). Los perfiles de puntuación ofrecen mayor control sobre la clasificación de los elementos en los resultados de la búsqueda, con la capacidad de aumentar las coincidencias que se encuentran en campos específicos. La lógica de puntuación adicional puede ayudar a invalidar las diferencias menores entre las réplicas, ya que las puntuaciones de búsqueda de cada documento están más alejadas. Para este enfoque, se recomienda usar el [algoritmo de clasificación](index-ranking-similarity.md).

## <a name="hit-highlighting"></a>Resaltado de referencias

El resaltado de referencias hace referencia al formato del texto (por ejemplo, negrita o resaltados en amarillo) que se aplica a los términos coincidentes en un resultado, lo que facilita la ubicación de las coincidencias. Se proporcionan instrucciones de resaltado de referencias en la [solicitud de consulta](/rest/api/searchservice/search-documents). 

Para habilitar el resaltado de referencias, agregue `highlight=[comma-delimited list of string fields]` para especificar los campos que usarán el resaltado. Este resulta útil para los campos de contenido más largos, como un campo de descripción, en el que la coincidencia no es evidente de inmediato. Solo las definiciones de campo atribuidas como **searchable** (que permiten búsquedas) califican para el resaltado de referencias.

De forma predeterminada, Azure Cognitive Search devuelve hasta cinco elementos resaltados por campo. Puede ajustar este número al anexar al campo un guión seguido de un entero. Por ejemplo, `highlight=Description-10` devuelve hasta 10 elementos resaltados sobre el contenido coincidente en el campo Description.

El formato se aplica a las consultas de términos completos. El tipo de formato lo determinan las etiquetas (`highlightPreTag` y `highlightPostTag`), y el código controla la respuesta (por ejemplo, la aplicación de una fuente en negrita o un fondo amarillo).

En el ejemplo siguiente, los términos "sandy", "sand", "beaches", "beach" que se encuentran en el campo de descripción están etiquetados para el resaltado. Las consultas que desencadenan la expansión de consultas en el motor, como las búsquedas aproximadas y las de caracteres comodín, tienen una compatibilidad limitada con el resaltado de referencias.

```http
GET /indexes/hotels-sample-index/docs/search=sandy beaches&highlight=Description?api-version=2020-06-30 
```

```http
POST /indexes/hotels-sample-index/docs/search?api-version=2020-06-30 
    {  
      "search": "sandy beaches",  
      "highlight": "Description"
    }
```

### <a name="new-behavior-starting-july-15"></a>Nuevo comportamiento (a partir del 15 de julio)

Los servicios creados después del 15 de julio de 2020 proporcionarán una experiencia de resaltado diferente. Los servicios creados antes de esa fecha no cambiarán en lo que respecta al comportamiento de resaltado. 

Con el nuevo comportamiento:

* Solo se devolverán las frases que coincidan con la consulta de frases completas. La consulta "super bowl" devolverá resultados como este:

    ```html
    '<em>super bowl</em> is super awesome with a bowl of chips'
    ```
  Tenga en cuenta que el término *bowl of chips* no aparece como resultado porque no coincide con la frase completa.

Al escribir código de cliente que implementa el resaltado de aciertos, tenga en cuenta este cambio. Debe saber que esto no le afectará, a menos que cree un servicio de búsqueda completamente nuevo.

## <a name="next-steps"></a>Pasos siguientes

Para generar rápidamente una página de búsqueda para el cliente, tenga en cuenta estas opciones:

+ [Generador de aplicaciones](search-create-app-portal.md), en el portal, crea una página HTML con una barra de búsqueda, navegación por facetas y área de resultados que incluye imágenes.
+ [Creación de la primera aplicación en C#](tutorial-csharp-create-first-app.md) es un tutorial para crear un cliente funcional. En el código de ejemplo se muestran las consultas paginadas, el resaltado de referencias y la ordenación.

Varios ejemplos de código que puede encontrar aquí incluyen una interfaz de front-end web: [Aplicación de demostración de trabajos de la ciudad de Nueva York](https://aka.ms/azjobsdemo), [código de ejemplo de JavaScript con un sitio de demostración activo](https://github.com/liamca/azure-search-javascript-samples) y [CognitiveSearchFrontEnd](https://github.com/LuisCabrer/CognitiveSearchFrontEnd).