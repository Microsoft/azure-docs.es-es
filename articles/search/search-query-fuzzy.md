---
title: Búsqueda aproximada
titleSuffix: Azure Cognitive Search
description: Implemente una experiencia de búsqueda del tipo "quiso decir" para corregir automáticamente un término mal escrito o un error de escritura.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/08/2020
ms.openlocfilehash: faa98f1c52cfe2dd0e19f085f4d33dedb6f01851
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88934895"
---
# <a name="fuzzy-search-to-correct-misspellings-and-typos"></a>Búsqueda aproximada para corregir errores ortográficos y tipográficos

Azure Cognitive Search admite la búsqueda aproximada, un tipo de consulta que compensa los errores tipográficos y los términos mal escritos en la cadena de entrada. Para ello, examina los términos que tienen una composición similar. La expansión de la búsqueda para que incluya coincidencias aproximadas tiene el efecto de corregir automáticamente un error tipográfico si la discrepancia se limita solo a algunos caracteres mal colocados. 

## <a name="what-is-fuzzy-search"></a>¿Qué es la búsqueda aproximada?

Se trata de un ejercicio de expansión que genera una coincidencia en términos que tienen una composición similar. Cuando se especifica una búsqueda aproximada, el motor genera un grafo (basado en la [teoría del autómata finito determinista](https://en.wikipedia.org/wiki/Deterministic_finite_automaton)) de términos compuestos de forma similar, para todos los términos completos de la consulta. Por ejemplo, si la consulta incluye tres términos, como en "university of washington" (universidad de washington), se crea un grafo para cada término de la consulta `search=university~ of~ washington~` (no hay eliminación de las palabras irrelevantes en la búsqueda aproximada, por lo que "of" también obtiene un grafo).

El grafo consta de hasta 50 expansiones, o permutaciones, de cada término, que capturan las variantes correctas e incorrectas en el proceso. A continuación, el motor devuelve las coincidencias más significativas en la respuesta. 

Para un término como "universidad", el grafo podría incluir "unversdad, universdad, universidad, universo, inverso". Los documentos que coinciden con los del grafo se incluirán en los resultados. A diferencia de otras consultas que analizan el texto para controlar diferentes formas de la misma palabra ("mice" y "mouse"), las comparaciones de una consulta aproximada se toman al pie de la letra sin ningún análisis lingüístico del texto. "Universo" e "inverso", que son semánticamente diferentes, coincidirán porque las discrepancias sintácticas son pequeñas.

Una coincidencia se realiza correctamente si las discrepancias se limitan a dos o menos ediciones. Una edición es un carácter insertado, eliminado, sustituido o transpuesto. El algoritmo de corrección de cadenas que especifica la diferencia es la métrica denominada [distancia Damerau-Levenshtein](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance), que se describe como el "número mínimo de operaciones (inserciones, eliminaciones, sustituciones o transposiciones de dos caracteres adyacentes) necesarias para cambiar una palabra en otra". 

En Azure Cognitive Search:

+ La consulta aproximada se aplica a términos completos, pero puede admitir también expresiones y construcciones. Por ejemplo, "Unviersdad~ de~ "Zragoza~" coincidiría con "Universidad de Zaragoza".

+ La distancia predeterminada de una edición es 2. Un valor de `~0` significa que no hay ninguna expansión (solo el término exacto se considera una coincidencia), pero puede especificar `~1` para un grado de diferencia o una edición. 

+ Una consulta aproximada puede expandir un término a un máximo de 50 permutaciones adicionales. Este límite no se puede configurar, pero puede reducir de forma eficaz el número de expansiones reduciendo la distancia de edición a 1.

+ Las respuestas se componen de documentos que contienen una coincidencia pertinente (con un máximo de 50).

Colectivamente, los grafos se envían como criterios de coincidencia en los tokens del índice. Como puede imaginar, la búsqueda aproximada es intrínsecamente más lenta que otros formularios de consulta. El tamaño y la complejidad del índice pueden determinar si las ventajas son suficientes para compensar la latencia de la respuesta.

> [!NOTE]
> Dado que la búsqueda aproximada tiende a ser lenta, puede que merezca la pena investigar alternativas como la indexación de n-gramas, con su progresión de secuencias cortas de caracteres (dos y tres secuencias de caracteres para los tokens bigrama y trigrama). En función del lenguaje y la superficie de la consulta, la opción con n-gramas podría proporcionarle un mejor rendimiento. La desventaja es que la indexación de n-gramas tiene un uso intensivo del almacenamiento y genera índices mucho mayores.
>
> Otra alternativa, que podría tener en cuenta si desea administrar solo los casos más notorios, sería un [mapa de sinónimos](search-synonyms.md). Por ejemplo, asigne "búsqueda" a "búsquerda, búsquda, bsqueda" o "recuperar" a "reucperar".

## <a name="indexing-for-fuzzy-search"></a>Indexación de búsquedas aproximadas

Los analizadores no se usan durante el procesamiento de consultas para crear un grafo de expansión, pero eso no significa que los analizadores se deban omitir en escenarios de búsqueda aproximada. Después de todo, los analizadores se usan durante la indexación para crear tokens con los que se realiza la búsqueda de coincidencias, si la consulta tiene una forma libre, una búsqueda filtrada o una búsqueda aproximada con un grafo como entrada. 

Por lo general, al asignar analizadores a cada campo, la decisión de ajustar la cadena de análisis se basa en el caso de uso principal (un filtro o una búsqueda de texto completo) en lugar de formularios de consulta especializados como la búsqueda aproximada. Por este motivo, no hay ninguna recomendación específica del analizador para las búsquedas aproximadas. 

Sin embargo, si las consultas de prueba no producen las coincidencias esperadas, puede intentar variar el analizador de indexación, estableciéndolo en un [analizador de idiomas](index-add-language-analyzers.md), para ver si obtiene mejores resultados. Algunos idiomas, especialmente los que tienen mutaciones vocálicas, pueden beneficiarse de las formas declinadas y de palabras irregulares generadas por los procesadores de lenguaje natural de Microsoft. En algunos casos, el uso del analizador de idioma adecuado puede suponer una diferencia entre si un término se tokeniza de forma que sea compatible con el valor proporcionado por el usuario.

## <a name="how-to-use-fuzzy-search"></a>Uso de la búsqueda aproximada

Las consultas aproximadas se construyen mediante la sintaxis de consulta completa de Lucene, invocando el [analizador de consultas de Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html).

1. Establezca el analizador de Lucene completo en la consulta (`queryType=full`).

1. Opcionalmente, se puede limitar el ámbito de la solicitud a campos específicos mediante este parámetro (`searchFields=<field1,field2>`). 

1. Anexe el operador de tilde (`~`) al final del término completo (`search=<string>~`).

   Incluya un parámetro opcional, un número entre 0 y 2 (valor predeterminado), si desea especificar la distancia de edición (`~1`). Por ejemplo, "blue~" o "blue~1" devolvería "blue", "blues" y "glue".

En Azure Cognitive Search, además del término y la distancia (2 como máximo), no hay parámetros adicionales que establecer en la consulta.

> [!NOTE]
> Durante el procesamiento de consultas, las consultas aproximadas no se someten a un [análisis léxico](search-lucene-query-architecture.md#stage-2-lexical-analysis). La entrada de la consulta se agrega directamente al árbol de consultas y se expande para crear un grafo de términos. La única transformación realizada es la conversión a minúsculas.

## <a name="testing-fuzzy-search"></a>Prueba de la búsqueda aproximada

Para realizar pruebas sencillas, se recomienda el [Explorador de búsqueda](search-explorer.md) o [Postman](search-get-started-postman.md) para recorrer en iteración una expresión de consulta. Ambas herramientas son interactivas, lo que significa que puede recorrer rápidamente varias variantes de un término y evaluar las respuestas que se devuelven.

Cuando los resultados son ambiguos, el [resaltado de referencias](search-pagination-page-layout.md#hit-highlighting) puede ayudarle a identificar la coincidencia en la respuesta. 

> [!Note]
> El uso del resaltado de referencias para identificar coincidencias aproximadas tiene limitaciones y solo funciona para la búsqueda aproximada básica. Si el índice tiene perfiles de puntuación, o si formula la consulta por niveles con una sintaxis adicional, es posible que el resaltado de referencias no identifique la coincidencia. 

### <a name="example-1-fuzzy-search-with-the-exact-term"></a>Ejemplo 1: búsqueda aproximada con el término exacto

Suponga que existe la siguiente cadena en un campo `"Description"` de un documento de búsqueda: `"Test queries with special characters, plus strings for MSFT, SQL and Java."`

Comience con una búsqueda aproximada de "especial" y agregue el resaltado de referencias al campo de descripción:

```console
search=special~&highlight=Description
```

En la respuesta, como ha agregado el resaltado de referencias, el formato se aplica a "especial" como término coincidente.

```output
"@search.highlights": {
    "Description": [
        "Test queries with <em>special</em> characters, plus strings for MSFT, SQL and Java."
    ]
```

Vuelva a intentar la solicitud, escribiendo incorrectamente "especial" quitando varias letras (en este ejemplo, "pe"):

```console
search=scial~&highlight=Description
```

Hasta el momento, no hay ningún cambio en la respuesta. Si se usa el valor predeterminado de 2 grados de distancia, la eliminación de dos caracteres como "pe" en el caso de "especial" sigue permitiendo una coincidencia correcta en ese término.

```output
"@search.highlights": {
    "Description": [
        "Test queries with <em>special</em> characters, plus strings for MSFT, SQL and Java."
    ]
```

Vuelta a intentar otra solicitud, modificando aún más el término de búsqueda quitando un último carácter hasta un total de tres eliminaciones (por ejemplo, pasar de "especial" a "escal"):

```console
search=scal~&highlight=Description
```

Observe que se devuelve la misma respuesta, pero ahora, en lugar de buscar coincidencias de "especial", la coincidencia aproximada es para "SQL".

```output
        "@search.score": 0.4232868,
        "@search.highlights": {
            "Description": [
                "Mix of special characters, plus strings for MSFT, <em>SQL</em>, 2019, Linux, Java."
            ]
```

El objetivo de este ejemplo expandido es ilustrar la claridad que el resaltado de referencias puede aportar a resultados ambiguos. En todos los casos, se devuelve el mismo documento. Si hubiera confiado en los identificadores de documento para comprobar una coincidencia, podría haberse perdido el cambio de "especial" a "SQL".

## <a name="see-also"></a>Consulte también

+ [Cómo funciona la búsqueda de texto completo en Azure Cognitive Search (arquitectura de análisis de consultas)](search-lucene-query-architecture.md)
+ [Explorador de búsqueda](search-explorer.md)
+ [Consultas con .NET](./search-get-started-dotnet.md)
+ [Consultas con REST](./search-get-started-powershell.md)