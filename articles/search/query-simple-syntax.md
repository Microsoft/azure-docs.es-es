---
title: Sintaxis de consulta simplificada
titleSuffix: Azure Cognitive Search
description: Referencia de la sintaxis de consulta simple que se usa para las consultas de búsqueda de texto completo en Azure Cognitive Search.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 04/24/2020
ms.openlocfilehash: d07364e20cc11abc52ad9b308eb5daed8a65c146
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88923388"
---
# <a name="simple-query-syntax-in-azure-cognitive-search"></a>Sintaxis de consulta simple en Azure Cognitive Search

Azure Cognitive Search implementa dos lenguajes de consulta basados en Lucene: [Analizador de consultas simple](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/simple/SimpleQueryParser.html) y [Analizador de consultas de Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html).

El analizador simple es más flexible e intentará interpretar una solicitud incluso si no está perfectamente compuesta. Debido a esta flexibilidad, es el valor predeterminado para las consultas en Azure Cognitive Search. 

La sintaxis simple se usa para las expresiones de consulta que se pasan en el parámetro `search` de una [solicitud de búsqueda de documentos](/rest/api/searchservice/search-documents), no se debe confundir con la [sintaxis de OData](query-odata-filter-orderby-syntax.md) que se usa para el parámetro [$filter expressions](search-filters.md) de la misma API de búsqueda de documentos. Los parámetros `search` y `$filter` tienen diferente sintaxis, con sus propias reglas para construir consultas, cadenas de escape, etc.

Aunque el analizador simple se basa en la clase del [analizador de consultas simples de Apache Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/simple/SimpleQueryParser.html), la implementación en Azure Cognitive Search excluye la búsqueda aproximada. Si necesita [búsqueda aproximada](search-query-fuzzy.md) u otros formularios de consulta avanzados, considere la [sintaxis de consulta completa de Lucene](query-lucene-syntax.md) alternativa en su lugar.

## <a name="invoke-simple-parsing"></a>Invocación del análisis simple

La sintaxis simple es la opción predeterminada. La invocación solo es necesaria si se está restableciendo la sintaxis de completa a simple. Para establecer la sintaxis explícitamente, use el parámetro de búsqueda `queryType`. Entre los valores válidos se incluyen `queryType=simple` o `queryType=full`, donde `simple` es el valor predeterminado y `full` invoca el [analizador de consultas completas de Lucene](query-lucene-syntax.md) para consultas más avanzadas. 

## <a name="syntax-fundamentals"></a>Fundamentos de sintaxis

Cualquier texto con uno o varios términos se considera un punto inicial válido para la ejecución de consultas. Azure Cognitive Search encontrará coincidencias en los documentos que contengan cualquiera de los términos o todos ellos, incluidas las variaciones encontradas durante el análisis del texto.

Tan sencillo como suena: hay un aspecto de la ejecución de consultas en Azure Cognitive Search que *podría* producir resultados inesperados, aumentando (en lugar de disminuir) los resultados de búsqueda a medida que se agregan más términos y operadores a la cadena de entrada. Que se produzca o no esta expansión realmente depende de la inclusión de un operador NOT, combinado con una configuración del parámetro **searchMode**, que determina cómo se interpreta NOT en términos de comportamientos de AND u OR. Para más información, vea [NOT operator](#not-operator) (Operador NOT).

### <a name="precedence-operators-grouping"></a>Operadores de precedencia (agrupación)

Puede usar paréntesis para crear subconsultas, incluidos los operadores dentro de la instrucción entre paréntesis. Por ejemplo, `motel+(wifi|luxury)` buscará los documentos que contengan el término "motel" y "wifi" o "luxury" (o ambos).

La agrupación de campos es parecida pero establece el ámbito de la agrupación en un único campo. Por ejemplo, `hotelAmenities:(gym+(wifi|pool))` busca "gym" y "wifi", o "gym" y "pool" en el campo "hotelAmenities".  

### <a name="escaping-search-operators"></a>Escape de los operadores de búsqueda  

En la sintaxis simple, los operadores de búsqueda incluyen estos caracteres: `+ | " ( ) ' \`  

Si alguno de estos caracteres forma parte de un token en el índice, aplique el escape anteponiendo un carácter de barra diagonal inversa (`\`) en la consulta. Por ejemplo, supongamos que ha usado un analizador personalizado para la tokenización de todo el término y el índice contiene la cadena "Luxury+Hotel". Para obtener una coincidencia exacta en este token, inserte un carácter de escape:  `search=luxury\+hotel`. 

Con el fin de simplificar las cosas para los casos más típicos, existen dos excepciones a esta regla donde la secuencia de escape no es necesaria:  

+ El operador NOT `-` necesita escaparse solamente si es el primer carácter después de un espacio en blanco. Si `-` aparece en el centro (por ejemplo, en `3352CDD0-EF30-4A2E-A512-3B30AF40F3FD`), puede omitir el escape.

+ El operador de sufijo `*` solo debe ser un carácter de escape si es el último carácter antes de un espacio en blanco. Si `*` aparece en el centro (por ejemplo, en `4*4=16`), no se necesita ningún escape.

> [!NOTE]  
> De forma predeterminada, el analizador estándar eliminará y dividirá palabras en guiones, espacios en blanco, y comercial y otros caracteres durante el [análisis léxico](search-lucene-query-architecture.md#stage-2-lexical-analysis). Si necesita que caracteres especiales permanezcan en la cadena de consulta, es posible que necesite un analizador que los conserve en el índice. Algunas opciones incluyen [analizadores de lenguaje](index-add-language-analyzers.md) natural de Microsoft, que conserva palabras con guiones o un analizador personalizado para patrones más complejos. Para más información, vea [Términos parciales, patrones y caracteres especiales](search-query-partial-matching.md).

### <a name="encoding-unsafe-and-reserved-characters-in-urls"></a>Codificación de caracteres reservados y no seguros en las direcciones URL

Asegúrese de que todos los caracteres reservados y no seguros están codificados en una dirección URL. Por ejemplo, "#" es un carácter no seguro porque es un identificador de delimitador o fragmento en una dirección URL. El carácter debe codificarse con `%23` si se usa en una dirección URL. "&" y "=" son ejemplos de caracteres reservados, ya que delimitan parámetros y especifican valores en Azure Cognitive Search. Consulte [RFC1738: Localizadores uniformes de recursos (URL)](https://www.ietf.org/rfc/rfc1738.txt) para más información.

Los caracteres no seguros son ``" ` < > # % { } | \ ^ ~ [ ]``. Los caracteres reservados son `; / ? : @ = + &`.

### <a name="querying-for-special-characters"></a>Consulta para caracteres especiales

En algunas circunstancias, es posible que quiera buscar un carácter especial, como el emoji "❤" o el signo "€". En ese caso, asegúrese de que el analizador que usa no filtre esos caracteres.  El analizador estándar omite muchos de los caracteres especiales para que no se conviertan en tokens del índice.

Por lo tanto, el primer paso consiste en asegurarse de que se usa un analizador que tomará en cuenta los tokens de los elementos. Por ejemplo, el analizador "whitespace" incluye todas las secuencias de caracteres separadas por espacios en blanco como tokens, por lo que la cadena "❤" se consideraría un token. Además, un analizador como el del idioma inglés de Microsoft ("en.microsoft") debe incluir la cadena "€" como un token. Puede [probar un analizador](/rest/api/searchservice/test-analyzer) para ver qué tokens genera para una consulta determinada.

Al usar caracteres Unicode, asegúrese de que los símbolos se escapen correctamente en la dirección URL de la consulta (por ejemplo, para "❤" usaría la secuencia de escape `%E2%9D%A4+`). Postman realiza esta traducción automáticamente.

###  <a name="query-size-limits"></a><a name="bkmk_querysizelimits"></a> Límites de tamaño de consulta

 Existe un límite en el tamaño de las consultas que se pueden enviar a Azure Cognitive Search. En concreto, puede tener como máximo 1024 cláusulas (expresiones separadas por AND, OR, etc.). También hay un límite de aproximadamente 32 KB en el tamaño de cualquier término individual en una consulta. Si la aplicación genera consultas de búsqueda mediante programación, se recomienda que la diseña de manera que no genere consultas de tamaño ilimitado.  

## <a name="boolean-search"></a>Búsqueda booleana

Puede insertar operadores booleanos (AND, OR o NOT) en una cadena de consulta para crear un conjunto enriquecido de criterios en los que se encuentran los documentos coincidentes. 

### <a name="and-operator-"></a>Operador AND `+`

El operador AND es un signo más. Por ejemplo, `wifi + luxury` buscará documentos que contengan tanto `wifi` como `luxury`.

### <a name="or-operator-"></a>Operador OR `|`

El operador OR es una barra vertical o el carácter de barra vertical. Por ejemplo, `wifi | luxury` buscará documentos que contengan `wifi`, `luxury` o ambos.

<a name="not-operator"></a>

### <a name="not-operator--"></a>Operador NOT `-`

El operador NOT es un signo menos. Por ejemplo, `wifi –luxury` buscará documentos que contengan el término `wifi` o que no tengan `luxury`.

El parámetro **searchMode** de una solicitud de consulta controla si un término con el operador NOT es ANDed o ORed con otros términos de la consulta (suponiendo que no haya ningún operador `+` o `|` en los demás términos). Los valores válidos son `any` o `all`.

`searchMode=any` aumenta la recuperación de consultas al incluirse más resultados y, de forma predeterminada, `-` se interpretará como "OR NOT". Por ejemplo, `wifi -luxury` encontrará coincidencias en los documentos que contengan el término `wifi` o en aquellos que no contengan el término `luxury`.

`searchMode=all` aumenta la precisión de las consultas al incluirse menos resultados y, de forma predeterminada, - se interpretará como "AND NOT". Por ejemplo, `wifi -luxury` encontrará coincidencias en los documentos que contengan el término `wifi` y no contengan el término "luxury". Este es posiblemente un comportamiento más intuitivo para el operador `-`. Por lo tanto, debe considerar el uso de `searchMode=all` en lugar de `searchMode=any` si desea optimizar las búsquedas para precisión en lugar de para recuperación, *y* si los usuarios utilizan con frecuencia el operador `-` en las búsquedas.

Al decidir sobre una configuración de **searchMode**, tenga en cuenta los patrones de interacción del usuario para las consultas en varias aplicaciones. Es más probable que los usuarios que buscan información incluyan un operador en una consulta, en lugar de sitios de comercio electrónico que tengan más estructuras de navegación integradas.

<a name="prefix-search"></a>

## <a name="wildcard-prefix-matching--"></a>Coincidencia de prefijos de los caracteres comodín (*, ?)

En el caso de las consultas "empieza por", agregue un operador de sufijo como marcador de posición para la última parte de un término. Use un asterisco `*` para varios caracteres o `?` para caracteres individuales. Por ejemplo, `lingui*` coincidirá con "linguistic" o "linguini", sin distinción de mayúsculas y minúsculas. 

De forma similar a los filtros, una consulta de prefijo busca una coincidencia exacta. Como tal, no hay ninguna puntuación de relevancia (todos los resultados reciben una puntuación de búsqueda de 1,0). Tenga en cuenta que las consultas de prefijo pueden ser lentas, en especial si el índice es grande y el prefijo está formado por un número pequeño de caracteres. Podría funcionar más rápido una metodología alternativa, como la tokenización de n-gramas perimetrales.

Para otras variantes de consulta con caracteres comodín, como la coincidencia de sufijos o interfijos en la parte inicial o intermedia de un término, use la [sintaxis de Lucene completa para la búsqueda con caracteres comodín](query-lucene-syntax.md#bkmk_wildcard).

## <a name="phrase-search-"></a>Búsqueda de frases `"`

Una búsqueda de términos es una consulta para uno o varios términos, donde cualquiera de los términos se considera una coincidencia. Una búsqueda de frases es una frase exacta entre comillas `" "`. Por ejemplo, mientras que `Roach Motel` (sin comillas) buscaría documentos que contuvieran `Roach` y/o `Motel` en cualquier lugar y en cualquier orden, `"Roach Motel"` (con comillas) solo encontrará coincidencias en documentos que contengan esa frase completa junta y en ese orden (el análisis léxico sigue aplicándose).

## <a name="see-also"></a>Consulte también  

+ [Funcionamiento de la búsqueda de texto completo en Azure Cognitive Search](search-lucene-query-architecture.md)
+ [Ejemplos de consulta para una búsqueda simple](search-query-simple-examples.md)
+ [Ejemplos de consulta para una búsqueda completa de Lucene](search-query-lucene-examples.md)
+ [API de REST de documentos de búsqueda](/rest/api/searchservice/Search-Documents)
+ [Sintaxis de consulta de Lucene](query-lucene-syntax.md)
+ [Sintaxis de expresión de OData](query-odata-filter-orderby-syntax.md)