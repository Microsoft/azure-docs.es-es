---
title: Sintaxis de consulta de Lucene
titleSuffix: Azure Cognitive Search
description: Referencia para la sintaxis de consulta completa de Lucene, tal como se usa en Azure Cognitive Search para caracteres comodín, búsqueda aproximada, RegEx y otras construcciones de consulta avanzadas.
manager: nitinme
author: brjohnstmsft
ms.author: brjohnst
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/23/2020
translation.priority.mt:
- de-de
- es-es
- fr-fr
- it-it
- ja-jp
- ko-kr
- pt-br
- ru-ru
- zh-cn
- zh-tw
ms.openlocfilehash: 6ea8bc2551df4f85e4b856dc9cf1c06a9bd571fd
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88923456"
---
# <a name="lucene-query-syntax-in-azure-cognitive-search"></a>Sintaxis de consulta de Lucene en Azure Cognitive Search

Puede escribir consultas en Azure Cognitive Search basadas en la sintaxis enriquecida del [analizador de consultas de Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html) para formas de consulta especializadas, por ejemplo, comodines, búsqueda aproximada, búsqueda por proximidad o expresiones regulares. Gran parte de la sintaxis del analizador de consultas de Lucene [se implementa tal cual en Azure Cognitive Search](search-lucene-query-architecture.md), a excepción de las *búsquedas de intervalo* que se construyen mediante expresiones `$filter`. 

> [!NOTE]
> La sintaxis completa de Lucene se usa para las expresiones de consulta que se pasan en el parámetro **search** de la API [Buscar documentos](/rest/api/searchservice/search-documents), no se debe confundir con la [sintaxis de OData](query-odata-filter-orderby-syntax.md) que se usa para el parámetro [$filter](search-filters.md) de esa API. Estas distintas sintaxis tienen sus propias reglas para construir consultas, cadenas de escape, etc.

## <a name="invoke-full-parsing"></a>Invocación del análisis completo

Establezca el parámetro de búsqueda `queryType` para especificar qué analizador usar. Los valores válidos incluyen `simple|full`, siendo `simple` el predeterminado, y `full` en Lucene. 

<a name="bkmk_example"></a> 

### <a name="example-showing-full-syntax"></a>Ejemplo que muestra la sintaxis completa

En el ejemplo siguiente se buscan documentos en el índice mediante la sintaxis de consulta de Lucene, evidente en el parámetro `queryType=full`. Esta consulta devuelve los hoteles en los que el campo de categoría contiene el término "budget" y todos los campos de búsqueda que incluyen la expresión "recently renovated". Los documentos que contienen la expresión "recently renovated" tienen una clasificación más alta como resultado del valor de impulso del término (3).  

El parámetro `searchMode=all` es significativo en este ejemplo. Cada vez que los operadores están en la consulta, normalmente debe establecer `searchMode=all` para asegurarse de que se busca la coincidencia con *todos* los criterios.

```
GET /indexes/hotels/docs?search=category:budget AND \"recently renovated\"^3&searchMode=all&api-version=2020-06-30&querytype=full
```

 Como alternativa, use POST:  

```
POST /indexes/hotels/docs/search?api-version=2020-06-30
{
  "search": "category:budget AND \"recently renovated\"^3",
  "queryType": "full",
  "searchMode": "all"
}
```

Para ver ejemplos adicionales, consulte los [ejemplos de sintaxis de consulta de Lucene para la creación de consultas en Azure Cognitive Search](search-query-lucene-examples.md). Para más información sobre cómo especificar el contingente completo de parámetros de consulta, consulte [Documentos de búsqueda &#40;API REST de Azure Cognitive Search&#41;](/rest/api/searchservice/Search-Documents).

> [!NOTE]  
>  Azure Cognitive Search también admite [sintaxis de consulta simple](query-simple-syntax.md), un lenguaje de consulta sencillo y robusto que se puede usar para la búsqueda directa de palabras clave.  

##  <a name="syntax-fundamentals"></a><a name="bkmk_syntax"></a> Fundamentos de sintaxis  

Los siguientes fundamentos de sintaxis se aplican a todas las consultas que usan la sintaxis de Lucene.  

### <a name="operator-evaluation-in-context"></a>Evaluación de operadores en contexto

La selección de ubicación determina si un símbolo se interpreta como un operador o como otro carácter de una cadena.

Por ejemplo, en la sintaxis completa de Lucene, se usa la tilde (~) para la búsqueda aproximada y la búsqueda por proximidad. Cuando se coloca detrás de una frase entre comillas, ~ invoca la búsqueda por proximidad. Cuando se coloca al final de un término, ~ invoca la búsqueda aproximada.

Dentro de un término, como "business~analyst", el carácter no se evalúa como un operador. En este caso, suponiendo que la consulta es de un término o una frase, la [búsqueda de texto completo](search-lucene-query-architecture.md) con [análisis léxico](search-lucene-query-architecture.md#stage-2-lexical-analysis) elimina la tilde ~ y divide el término "business~analyst" en dos: business OR analyst.

El ejemplo anterior es la tilde (~), pero el mismo principio se aplica a todos los operadores.

### <a name="escaping-special-characters"></a>Escape de caracteres especiales

Para usar cualquiera de los operadores de búsqueda como parte del texto de búsqueda, debe anteponer el carácter como prefijo con una sola barra diagonal inversa (`\`). Por ejemplo, para realizar una búsqueda con caracteres comodín en `https://`, donde `://` forma parte de la cadena de consulta, debe especificar `search=https\:\/\/*`. Del mismo modo, un patrón de número de teléfono con escape podría ser similar a este `\+1 \(800\) 642\-7676`.

Los caracteres especiales que requieren escape son los siguientes:  
`+ - & | ! ( ) { } [ ] ^ " ~ * ? : \ /`  

> [!NOTE]  
> Aunque el escape mantiene juntos los tokens, el [análisis léxico](search-lucene-query-architecture.md#stage-2-lexical-analysis) durante la indexación puede eliminarlos. Por ejemplo, el analizador de Lucene estándar dividirá palabras con guiones, espacios en blanco y otros caracteres. Si necesita caracteres especiales en la cadena de consulta, es posible que necesite un analizador que los conserve en el índice. Algunas opciones incluyen [analizadores de lenguaje](index-add-language-analyzers.md) natural de Microsoft, que conserva palabras con guiones o un analizador personalizado para patrones más complejos. Para más información, vea [Términos parciales, patrones y caracteres especiales](search-query-partial-matching.md).

### <a name="encoding-unsafe-and-reserved-characters-in-urls"></a>Codificación de caracteres reservados y no seguros en las direcciones URL

Asegúrese de que todos los caracteres reservados y no seguros están codificados en una dirección URL. Por ejemplo, "#" es un carácter no seguro porque es un identificador de delimitador o fragmento en una dirección URL. El carácter debe codificarse con `%23` si se usa en una dirección URL. "&" y "=" son ejemplos de caracteres reservados, ya que delimitan parámetros y especifican valores en Azure Cognitive Search. Consulte [RFC1738: Localizadores uniformes de recursos (URL)](https://www.ietf.org/rfc/rfc1738.txt) para más información.

Los caracteres no seguros son ``" ` < > # % { } | \ ^ ~ [ ]``. Los caracteres reservados son `; / ? : @ = + &`.

###  <a name="query-size-limits"></a><a name="bkmk_querysizelimits"></a> Límites de tamaño de consulta

 Existe un límite en el tamaño de las consultas que se pueden enviar a Azure Cognitive Search. En concreto, puede tener como máximo 1024 cláusulas (expresiones separadas por AND, OR, etc.). También hay un límite de aproximadamente 32 KB en el tamaño de cualquier término individual en una consulta. Si la aplicación genera consultas de búsqueda mediante programación, se recomienda que la diseña de manera que no genere consultas de tamaño ilimitado.  

### <a name="precedence-operators-grouping"></a>Operadores de precedencia (agrupación)

 Puede usar paréntesis para crear subconsultas, incluidos los operadores dentro de la instrucción entre paréntesis. Por ejemplo, `motel+(wifi||luxury)` buscará los documentos que contengan el término "motel" y "wifi" o "luxury" (o ambos).

La agrupación de campos es parecida pero establece el ámbito de la agrupación en un único campo. Por ejemplo, `hotelAmenities:(gym+(wifi||pool))` busca "gym" y "wifi", o "gym" y "pool" en el campo "hotelAmenities".  

##  <a name="boolean-search"></a><a name="bkmk_boolean"></a> Búsqueda booleana

 Especifique siempre operadores booleanos de texto (AND, OR, NOT) todo en mayúsculas.  

### <a name="or-operator-or-or-"></a>Operador OR `OR` o `||`

El operador OR es una barra vertical o el carácter de barra vertical. Por ejemplo: `wifi || luxury` buscará documentos que contengan "wifi" o "luxury" o ambos. Como OR es el operador de conjunción predeterminado, podría también dejarlo fuera, de forma que `wifi luxury` es el equivalente de `wifi || luxury`.

### <a name="and-operator-and--or-"></a>Operador AND `AND`, `&&` o `+`

El operador AND es una Y comercial o un signo más. Por ejemplo: `wifi && luxury` buscará documentos que contengan "wifi" y "luxury". El carácter de signo más (+) se usa para los términos requeridos. Por ejemplo, `+wifi +luxury` estipula que ambos términos deben aparecer en algún lugar en el campo de un documento.

### <a name="not-operator-not--or--"></a>Operador NOT `NOT``!` o `-`

El operador NOT es un signo menos. Por ejemplo, `wifi –luxury` buscará documentos que contengan el término `wifi` o que no tengan `luxury`.

El parámetro **searchMode** de una solicitud de consulta controla si un término con el operador NOT es ANDed o ORed con otros términos de la consulta (suponiendo que no haya ningún operador `+` o `|` en los demás términos). Los valores válidos son `any` o `all`.

`searchMode=any` aumenta la recuperación de consultas al incluirse más resultados y, de forma predeterminada, `-` se interpretará como "OR NOT". Por ejemplo, `wifi -luxury` encontrará coincidencias en los documentos que contengan el término `wifi` o en aquellos que no contengan el término `luxury`.

`searchMode=all` aumenta la precisión de las consultas al incluirse menos resultados y, de forma predeterminada, - se interpretará como "AND NOT". Por ejemplo, `wifi -luxury` encontrará coincidencias en los documentos que contengan el término `wifi` y no contengan el término "luxury". Este es posiblemente un comportamiento más intuitivo para el operador `-`. Por lo tanto, debe considerar el uso de `searchMode=all` en lugar de `searchMode=any` si desea optimizar las búsquedas para precisión en lugar de para recuperación, *y* si los usuarios utilizan con frecuencia el operador `-` en las búsquedas.

Al decidir sobre una configuración de **searchMode**, tenga en cuenta los patrones de interacción del usuario para las consultas en varias aplicaciones. Es más probable que los usuarios que buscan información incluyan un operador en una consulta, en lugar de sitios de comercio electrónico que tengan más estructuras de navegación integradas.

##  <a name="fielded-search"></a><a name="bkmk_fields"></a> Búsqueda clasificada por campos

Puede definir una operación de búsqueda clasificada por campos con la sintaxis `fieldName:searchExpression`, donde la expresión de búsqueda puede ser una sola palabra, una frase o una expresión más compleja entre paréntesis, opcionalmente con operadores booleanos. A continuación se muestran algunos ejemplos:  

- genre:jazz NOT history  

- artists:("Miles Davis" "John Coltrane")

Asegúrese de colocar varias cadenas entre comillas si quiere que las dos cadenas se evalúen como una sola entidad, como en este caso donde se buscan dos ciudades distintas en el campo `artists`.  

El campo especificado en `fieldName:searchExpression` debe ser un campo `searchable`.  Consulte [Create Index](/rest/api/searchservice/create-index) (Crear índice) para más información sobre cómo se usan los atributos de índice en las definiciones de campo.  

> [!NOTE]
> Al usar expresiones de búsqueda clasificada por campos, no es necesario usar el parámetro `searchFields` porque cada expresión de búsqueda clasificada por campos tiene un nombre de campo especificado explícitamente. Pero tenga en cuenta que todavía puede usar el parámetro `searchFields` si quiere ejecutar una consulta donde algunas partes se limitan a un campo específico y el resto se podría aplicar a varios campos. Por ejemplo, la consulta `search=genre:jazz NOT history&searchFields=description` coincidiría con `jazz` únicamente con el campo `genre`, aunque coincidiría con `NOT history` con el campo `description`. El nombre de campo proporcionado en `fieldName:searchExpression` siempre tiene prioridad sobre el parámetro `searchFields`. Por eso en este ejemplo no es necesario incluir `genre` en el parámetro `searchFields`.

##  <a name="fuzzy-search"></a><a name="bkmk_fuzzy"></a> Búsqueda aproximada

Una búsqueda aproximada busca coincidencias en términos que tienen una construcción similar, expandiendo un término hasta el máximo de 50 términos que cumplen los criterios de distancia de dos o menos. Para más información, vea [Búsqueda aproximada](search-query-fuzzy.md).

 Para realizar una búsqueda aproximada, use el símbolo "~" de tilde de la Ñ al final de una sola palabra con un parámetro opcional, un número entre 0 y 2, que especifica la distancia de edición. Por ejemplo, "blue~" o "blue~1" devolvería "blue", "blues" y "glue".

 La búsqueda aproximada solo puede aplicarse a términos, no a frases, pero se puede anexar la tilde a cada término individualmente en un nombre de varias partes o frase. Por ejemplo, "Unviersdad~ de~ "Zragoza~" coincidiría con "Universidad de Zaragoza".
 
##  <a name="proximity-search"></a><a name="bkmk_proximity"></a> Búsqueda por proximidad

Las búsquedas de proximidad se utilizan para buscar términos que están cerca entre sí en un documento. Inserte un símbolo "~" de la tilde de la Ñ al final de una frase seguido del número de palabras que crea el límite de proximidad. Por ejemplo, `"hotel airport"~5` encontrará los términos "hotel" y "airport" a menos de 5 palabras de distancia unos de otros en un documento.  


##  <a name="term-boosting"></a><a name="bkmk_termboost"></a> Priorización de términos

"Priorización de términos" hace referencia a la valoración de un documento superior si contiene el término prioritario con respecto a los documentos que no contienen el término. Esto difiere de los perfiles de puntuación en los que aumentan ciertos campos, en lugar de términos específicos.  

En el siguiente ejemplo se muestran las diferencias. Suponga que hay un perfil de puntuación que impulsa las coincidencias en un campo determinado, como *genre* en el [ejemplo musicstoreindex](index-add-scoring-profiles.md#bkmk_ex). La priorización de términos podría utilizarse para dar mayor prioridad a determinados términos de búsqueda frente a otros. Por ejemplo, `rock^2 electronic` impulsará los documentos que contengan los términos de búsqueda en el campo genre frente a otros campos que permitan búsquedas en el índice. Además, los documentos que contienen el término de búsqueda *rock* tendrán una clasificación mayor que los del término de búsqueda *electronic* como resultado del valor de impulso de términos (2).  

 Para impulsar un término, use el símbolo de intercalación, "^", un símbolo con un factor de impulso (un número) al final del término que quiera buscar. También puede impulsar frases. Cuanto mayor sea el factor de prioridad, más relevante será el término en relación con otros términos de búsqueda. De forma predeterminada, el factor de prioridad es 1. Aunque el factor de impulso debe ser positivo, puede ser inferior a 1 (por ejemplo, 0,20).  

##  <a name="regular-expression-search"></a><a name="bkmk_regex"></a> Búsqueda de expresiones regulares  
 Una búsqueda de expresión regular encuentra una coincidencia en función de los patrones que son válidos en Apache Lucene, como se documenta en la [clase RegExp](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/util/automaton/RegExp.html). En Azure Cognitive Search, se incluye una expresión regular entre barras diagonales `/`.

 Por ejemplo, para encontrar documentos que contengan "motel" o "hotel", especifique `/[mh]otel/`. Las búsquedas mediante expresiones regulares se comparan con las palabras individuales.

Algunas herramientas y algunos idiomas imponen requisitos adicionales de caracteres de escape. En el caso de JSON, las cadenas que incluyen una barra diagonal tienen un carácter de escape con una barra diagonal inversa: "microsoft.com/azure/" se convierte en `search=/.*microsoft.com\/azure\/.*/`, donde `search=/.* <string-placeholder>.*/` configura la expresión regular y `microsoft.com\/azure\/` es la cadena con una barra diagonal de escape.

##  <a name="wildcard-search"></a><a name="bkmk_wildcard"></a> Búsqueda con caracteres comodín

Puede usar la sintaxis generalmente reconocida para búsquedas con caracteres comodín únicas (`*`) o múltiples (`?`). Por ejemplo, una expresión de consulta de `search=alpha*` devuelve "alfanumérico" o "alfabético". Tenga en cuenta que el Analizador de consultas de Lucene admite el uso de estos símbolos con un único término y no una frase.

La sintaxis de Lucene completa admite la coincidencia de prefijos, infijos y sufijos. Sin embargo, si solo necesita la coincidencia de prefijos, puede utilizar la sintaxis simple (la coincidencia de prefijos se admite en ambos).

La coincidencia de sufijos, en la que `*` o `?` precede a la cadena (como en `search=/.*numeric./`) o la coincidencia de infijos requiere una sintaxis completa de Lucene, así como los delimitadores de barra diagonal `/` de la expresión regular. ¿No puede utilizar un símbolo * o ? como primer carácter de un término, o dentro de un término, sin `/`. 

> [!NOTE]  
> Como norma general, la coincidencia de patrones es lenta, por lo que es posible que desee explorar métodos alternativos, como la tokenización de n-gramas perimetrales que crea tokens para las secuencias de caracteres de un término. El índice será mayor, pero las consultas se podrían ejecutar más rápido, según la construcción del patrón y la longitud de las cadenas que se van a indexar.
>

### <a name="impact-of-an-analyzer-on-wildcard-queries"></a>Impacto de un analizador en las consultas con caracteres comodín

Durante el análisis de consultas, las consultas que se formulan como prefijo, sufijo, carácter comodín o expresiones regulares se pasan tal cual al árbol de consultas y se omite el [análisis léxico](search-lucene-query-architecture.md#stage-2-lexical-analysis). Solo se encontrarán coincidencias si el índice contiene las cadenas en el formato que especifica la consulta. En la mayoría de los casos, durante la indexación necesitará un analizador que conserve la integridad de las cadenas para que la coincidencia de patrones y términos parciales sea correcta. Para obtener más información, consulte [Búsqueda de términos parciales en las consultas de Azure Cognitive Search](search-query-partial-matching.md).

Piense en una situación en la que quiere que la consulta de búsqueda "terminat*" devuelva resultados que contengan términos como "terminate", "termination" y "terminates".

Si usara el analizador en.lucene (Inglés Lucene), aplicaría una lematización agresiva de cada término. Por ejemplo, "terminate", "termination", "terminates" sufrirían una reducción hasta el token "termi" en el índice. Por otra parte, los términos de las consultas que usan caracteres comodín o búsqueda aproximada no se analizan, por lo que no habrá resultados que coincidan con la consulta "terminat*".

Además, los analizadores de Microsoft (en este caso, en.microsoft) son un poco más avanzados y usan lemas en lugar de lexemas. Esto significa que todos los tokens generados deben ser palabras en inglés válidas. Por ejemplo, ''terminate", "terminates" y "termination" se mantendrán en su totalidad en el índice, y esta sería una opción preferible para escenarios que dependen mucho de los caracteres comodín y la búsqueda aproximada.

##  <a name="scoring-wildcard-and-regex-queries"></a><a name="bkmk_searchscoreforwildcardandregexqueries"></a> Puntuación de consultas de caracteres comodín y expresiones regulares

Azure Cognitive Search usa la puntuación basada en la frecuencia ([TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf)) para las consultas de texto. Sin embargo, para consultas con caracteres comodín y expresiones regulares donde el ámbito de los términos puede ser posiblemente amplio, se omite el factor de frecuencia para evitar que la clasificación se desvíe hacia las coincidencias de términos menos frecuentes. Todas las coincidencias se tratan por igual en las búsquedas con caracteres comodín y expresiones regulares.

## <a name="see-also"></a>Consulte también

+ [Ejemplos de consulta para una búsqueda simple](search-query-simple-examples.md)
+ [Ejemplos de consulta para una búsqueda completa de Lucene](search-query-lucene-examples.md)
+ [Buscar en documentos](/rest/api/searchservice/Search-Documents)
+ [OData expression syntax for filters and sorting](query-odata-filter-orderby-syntax.md) (Sintaxis de expresiones de OData para filtros y ordenación)   
+ [Sintaxis de consulta simple en Azure Cognitive Search](query-simple-syntax.md)