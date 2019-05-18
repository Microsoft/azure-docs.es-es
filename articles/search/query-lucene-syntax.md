---
title: 'Sintaxis de consulta de Lucene: Azure Search'
description: Referencia de la sintaxis completa de Lucene, como se usa con Azure Search.
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/13/2019
author: brjohnstmsft
ms.author: brjohnst
ms.manager: cgronlun
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
ms.openlocfilehash: b051f844b8c221e2e53c5fcf204878f80447cfe8
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65596550"
---
# <a name="lucene-query-syntax-in-azure-search"></a>Sintaxis de consulta de Lucene en Azure Search
Puede escribir consultas en Azure Search basadas en la sintaxis enriquecida del [analizador de consultas de Lucene](https://lucene.apache.org/core/4_10_2/queryparser/org/apache/lucene/queryparser/classic/package-summary.html) para formularios de consulta especializados, por ejemplo, carácter comodín, búsqueda aproximada, búsqueda por proximidad o expresiones regulares. Gran parte de la sintaxis del analizador de consultas de Lucene [se implementa tal cual en Azure Search](search-lucene-query-architecture.md), a excepción de las *búsquedas de intervalo* que se construyen en Azure Search mediante expresiones `$filter`. 

## <a name="how-to-invoke-full-parsing"></a>Invocación del análisis completo

Establezca el parámetro de búsqueda `queryType` para especificar qué analizador usar. Los valores válidos incluyen `simple|full`, siendo `simple` el predeterminado, y `full` en Lucene. 

<a name="bkmk_example"></a> 

### <a name="example-showing-full-syntax"></a>Ejemplo que muestra la sintaxis completa

En el ejemplo siguiente se buscan documentos en el índice mediante la sintaxis de consulta de Lucene, evidente en el parámetro `queryType=full`. Esta consulta devuelve los hoteles en los que el campo de categoría contiene el término "budget" y todos los campos de búsqueda que incluyen la expresión "recently renovated". Los documentos que contienen la expresión "recently renovated" tienen una clasificación más alta como resultado del valor de impulso del término (3).  

El parámetro `searchMode=all` es significativo en este ejemplo. Cada vez que los operadores están en la consulta, normalmente debe establecer `searchMode=all` para asegurarse de que se busca la coincidencia con *todos* los criterios.

```
GET /indexes/hotels/docs?search=category:budget AND \"recently renovated\"^3&searchMode=all&api-version=2019-05-06&querytype=full
```

 Como alternativa, use POST:  

```
POST /indexes/hotels/docs/search?api-version=2019-05-06
{
  "search": "category:budget AND \"recently renovated\"^3",
  "queryType": "full",
  "searchMode": "all"
}
```

Para ver ejemplos adicionales, consulte los [ejemplos de sintaxis de consulta de Lucene para la creación de consultas en Azure Search](search-query-lucene-examples.md). Para más información sobre cómo especificar el contingente completo de parámetros de consulta, consulte [Search Documents &#40;Azure Search Service REST API&#41;](https://docs.microsoft.com/rest/api/searchservice/Search-Documents) (Buscar documentos [API REST del servicio Azure Search]).

> [!NOTE]  
>  Azure Search también admite [sintaxis de consulta simple](query-simple-syntax.md), un lenguaje de consulta sencillo y robusto que se puede usar para la búsqueda directa de palabras clave.  

##  <a name="bkmk_syntax"></a> Fundamentos de sintaxis  
 Los siguientes fundamentos de sintaxis se aplican a todas las consultas que usan la sintaxis de Lucene.  

### <a name="operator-evaluation-in-context"></a>Evaluación de operadores en contexto

La selección de ubicación determina si un símbolo se interpreta como un operador o como otro carácter de una cadena.

Por ejemplo, en la sintaxis completa de Lucene, se usa la tilde (~) para la búsqueda aproximada y la búsqueda por proximidad. Cuando se coloca detrás de una frase entre comillas, ~ invoca la búsqueda por proximidad. Cuando se coloca al final de un término, ~ invoca la búsqueda aproximada.

Dentro de un término, como "business~analyst", el carácter no se evalúa como un operador. En este caso, suponiendo que la consulta es de un término o una frase, la [búsqueda de texto completo](search-lucene-query-architecture.md) con [análisis léxico](search-lucene-query-architecture.md#stage-2-lexical-analysis) elimina la tilde ~ y divide el término "business~analyst" en dos: business OR analyst.

El ejemplo anterior es la tilde (~), pero el mismo principio se aplica a todos los operadores.

### <a name="escaping-special-characters"></a>Escape de caracteres especiales

 Los caracteres especiales se deben escapar para poder usarse como parte del texto de búsqueda. Para ello, deben ir prefijados con una barra diagonal inversa (\\). Los caracteres especiales que se deben escapar son los siguientes:  
`+ - && || ! ( ) { } [ ] ^ " ~ * ? : \ /`  

 Por ejemplo, para escapar un carácter comodín, use \\*.

### <a name="encoding-unsafe-and-reserved-characters-in-urls"></a>Codificación de caracteres reservados y no seguros en las direcciones URL

 Asegúrese de que todos los caracteres reservados y no seguros están codificados en una dirección URL. Por ejemplo, "#" es un carácter no seguro porque es un identificador de delimitador o fragmento en una dirección URL. El carácter debe codificarse con `%23` si se usa en una dirección URL. "&" y "=" son ejemplos de caracteres reservados, ya que delimitan parámetros y especifican valores en Azure Search. Consulte [RFC1738: Localizadores uniformes de recursos (URL)](https://www.ietf.org/rfc/rfc1738.txt) para más información.

 Los caracteres no seguros son ``" ` < > # % { } | \ ^ ~ [ ]``. Los caracteres reservados son `; / ? : @ = + &`.

### <a name="precedence-operators-grouping-and-field-grouping"></a>Operadores de precedencia: agrupación y agrupación de campos  
 Puede usar paréntesis para crear subconsultas, incluidos los operadores dentro de la instrucción entre paréntesis. Por ejemplo, `motel+(wifi||luxury)` buscará los documentos que contengan el término "motel" y "wifi" o "luxury" (o ambos).

La agrupación de campos es parecida pero establece el ámbito de la agrupación en un único campo. Por ejemplo, `hotelAmenities:(gym+(wifi||pool))` busca "gym" y "wifi", o "gym" y "pool" en el campo "hotelAmenities".  

### <a name="searchmode-parameter-considerations"></a>Consideraciones sobre los parámetros SearchMode  
 El efecto de `searchMode` sobre las consultas, como se describe en [Simple query syntax in Azure Search](query-simple-syntax.md) (Sintaxis de consulta simple en Azure Search), se aplica igualmente a la sintaxis de consulta de Lucene. Es decir, `searchMode` junto con los operadores NOT pueden dar lugar a resultados de consulta que pueden parecer inusuales si no están claros en las implicaciones de cómo establecer el parámetro. Si mantiene el valor predeterminado, `searchMode=any`, y usa un operador NOT, la operación se calcula como una acción OR, tal que "New York" NOT "Seattle" devuelve todas las ciudades que no son Seattle.  

##  <a name="bkmk_boolean"></a> Operadores booleanos (AND, OR, NOT) 
 Especifique siempre operadores booleanos de texto (AND, OR, NOT) todo en mayúsculas.  

### <a name="or-operator-or-or-"></a>Operador OR `OR` o `||`

El operador OR es una barra vertical o el carácter de barra vertical. Por ejemplo: `wifi || luxury` buscará documentos que contengan "wifi" o "luxury" o ambos. Como OR es el operador de conjunción predeterminado, podría también dejarlo fuera, de forma que `wifi luxury` es el equivalente de `wifi || luxuery`.

### <a name="and-operator-and--or-"></a>Operador AND `AND`, `&&` o `+`

El operador AND es una Y comercial o un signo más. Por ejemplo: `wifi && luxury` buscará documentos que contengan "wifi" y "luxury". El carácter de signo más (+) se usa para los términos requeridos. Por ejemplo, `+wifi +luxury` estipula que ambos términos deben aparecer en algún lugar en el campo de un documento.


### <a name="not-operator-not--or--"></a>Operador NOT `NOT` `!` o `-`

El operador NOT es un signo de exclamación o el signo menos. Por ejemplo, `wifi !luxury` buscará documentos que contengan el término "wifi" y/o no tengan "luxury". La opción `searchMode` controla si se aplica AND u OR a un término con el operador NOT con los otros términos de la consulta en ausencia de un operador + o ||. Recuerde que `searchMode` puede establecerse en `any` (valor predeterminado) o en `all`.

El uso de `searchMode=any` aumenta la recuperación de consultas al incluirse más resultados y, de forma predeterminada, - se interpretará como "OR NOT". Por ejemplo, `wifi -luxury` encontrará documentos que contengan el término *wifi* o aquellos que no contengan el término *luxury*.

El uso de `searchMode=all` aumenta la precisión de las consultas al incluirse menos resultados y, de forma predeterminada, - se interpretará como "AND NOT". Por ejemplo, `wifi -luxury` encontrará documentos que contengan el término `wifi` y no contengan el término `luxury`. Se puede decir que este comportamiento del operador - es más intuitivo. Por lo tanto, debería considerar elegir `searchMode=all` sobre `searchMode=any` si quiere optimizar la precisión de las búsquedas en lugar de la recuperación, *y* los usuarios usan con frecuencia el operador `-` en las búsquedas.

##  <a name="bkmk_querysizelimits"></a> Límites de tamaño de consulta  
 Existe un límite en el tamaño de las consultas que se pueden enviar a Azure Search. En concreto, puede tener como máximo 1024 cláusulas (expresiones separadas por AND, OR, etc.). También hay un límite de aproximadamente 32 KB en el tamaño de cualquier término individual en una consulta. Si la aplicación genera consultas de búsqueda mediante programación, se recomienda que la diseña de manera que no genere consultas de tamaño ilimitado.  

##  <a name="bkmk_searchscoreforwildcardandregexqueries"></a> Puntuación de consultas de caracteres comodín y expresiones regulares
 Azure Search usa la puntuación basada en la frecuencia ([TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf)) para las consultas de texto. Sin embargo, para consultas con caracteres comodín y expresiones regulares donde el ámbito de los términos puede ser posiblemente amplio, se omite el factor de frecuencia para evitar que la clasificación se desvíe hacia las coincidencias de términos menos frecuentes. Todas las coincidencias se tratan por igual en las búsquedas con caracteres comodín y expresiones regulares.

##  <a name="bkmk_fields"></a> Clasificada por campos de búsqueda  
Puede definir una operación de búsqueda clasificada por campos con el `fieldName:searchExpression` sintaxis, donde la expresión de búsqueda puede ser una sola palabra o una frase o una expresión más compleja entre paréntesis, opcionalmente con operadores booleanos. Estos son algunos ejemplos:  

- genre:jazz NOT history  

- artists:("Miles Davis" "John Coltrane")

Asegúrese de colocar varias cadenas entre comillas si quiere que las dos cadenas se evalúen como una sola entidad, como en este caso donde se buscan dos ciudades distintas en el campo `artists`.  

El campo especificado en `fieldName:searchExpression` debe ser un campo `searchable`.  Consulte [Create Index](https://docs.microsoft.com/rest/api/searchservice/create-index) (Crear índice) para más información sobre cómo se usan los atributos de índice en las definiciones de campo.  

> [!NOTE]
> Al usar responderán a las expresiones de búsqueda, no es necesario usar el `searchFields` parámetro porque cada responderán expresión de búsqueda tiene un nombre de campo especificado explícitamente. Sin embargo, todavía puede usar el `searchFields` parámetro si desea ejecutar una consulta donde algunas partes se limitan a un campo específico y el resto se podría aplicar a varios campos. Por ejemplo, la consulta `search=genre:jazz NOT history&searchFields=description` coincidiría con `jazz` únicamente a la `genre` campo mientras coincidiría con `NOT history` con el `description` campo. El nombre de campo proporcionado en `fieldName:searchExpression` siempre tiene prioridad sobre la `searchFields` parámetro, que es por eso que en este ejemplo, no es necesario incluir `genre` en el `searchFields` parámetro.

##  <a name="bkmk_fuzzy"></a> Búsqueda aproximada  
 Una búsqueda aproximada busca coincidencias en términos que tienen una construcción similar. Según la [documentación de Lucene](https://lucene.apache.org/core/4_10_2/queryparser/org/apache/lucene/queryparser/classic/package-summary.html), las búsquedas aproximadas se basan en la [distancia Levenshtein-Damerau](https://en.wikipedia.org/wiki/Damerau%E2%80%93Levenshtein_distance). Las búsquedas aproximadas pueden expandir un término hasta un máximo de 50 términos que satisfagan los criterios de distancia. 

 Para realizar una búsqueda aproximada, use el símbolo "~" de tilde de la Ñ al final de una sola palabra con un parámetro opcional, un número entre 0 y 2, que especifica la distancia de edición. Por ejemplo, "blue~" o "blue~1" devolvería "blue", "blues" y "glue".

 Búsqueda aproximada sólo puede aplicarse a los términos, no frases, pero no se puede anexar la tilde para cada término individualmente en un nombre de varias partes o frase. Por ejemplo, "Unviersty ~ de ~" Wshington ~ "coincidiría con"Universidad de Zaragoza".
 

##  <a name="bkmk_proximity"></a> Búsqueda por proximidad  
 Las búsquedas de proximidad se utilizan para buscar términos que están cerca entre sí en un documento. Inserte un símbolo "~" de la tilde de la Ñ al final de una frase seguido del número de palabras que crea el límite de proximidad. Por ejemplo, `"hotel airport"~5` encontrará los términos "hotel" y "airport" a menos de 5 palabras de distancia unos de otros en un documento.  


##  <a name="bkmk_termboost"></a> Priorización de términos  
 "Priorización de términos" hace referencia a la valoración de un documento superior si contiene el término prioritario con respecto a los documentos que no contienen el término. Esto difiere de los perfiles de puntuación en los que aumentan ciertos campos, en lugar de términos específicos.  

En el siguiente ejemplo se muestran las diferencias. Suponga que hay un perfil de puntuación que impulsa las coincidencias en un campo determinado, como *genre* en el [ejemplo musicstoreindex](index-add-scoring-profiles.md#bkmk_ex). La priorización de términos podría utilizarse para dar mayor prioridad a determinados términos de búsqueda frente a otros. Por ejemplo, `rock^2 electronic` impulsará los documentos que contengan los términos de búsqueda en el campo genre frente a otros campos que permitan búsquedas en el índice. Además, los documentos que contienen el término de búsqueda *rock* tendrán una clasificación mayor que los del término de búsqueda *electronic* como resultado del valor de impulso de términos (2).  

 Para impulsar un término, use el símbolo de intercalación, "^", un símbolo con un factor de impulso (un número) al final del término que quiera buscar. También puede impulsar frases. Cuanto mayor sea el factor de prioridad, más relevante será el término en relación con otros términos de búsqueda. De forma predeterminada, el factor de prioridad es 1. Aunque el factor de impulso debe ser positivo, puede ser inferior a 1 (por ejemplo, 0,20).  

##  <a name="bkmk_regex"></a> Búsqueda de expresiones regulares  
 Una búsqueda de expresión regular encuentra una coincidencia en función del contenido entre barras diagonales "/", como se documentó en la [clase RegExp](https://lucene.apache.org/core/4_10_2/core/org/apache/lucene/util/automaton/RegExp.html).  

 Por ejemplo, para encontrar documentos que contengan "motel" o "hotel", especifique `/[mh]otel/`.  Las búsquedas mediante expresiones regulares se comparan con las palabras individuales.   

##  <a name="bkmk_wildcard"></a> Búsqueda con caracteres comodín  
 Puede utilizar la sintaxis generalmente reconocida para búsquedas con caracteres comodín únicas (?) o múltiples (*). Tenga en cuenta que el Analizador de consultas de Lucene admite el uso de estos símbolos con un único término y no una frase.  

 Por ejemplo, para encontrar documentos que contengan las palabras con el prefijo "note", como "notebook" o "notepad", especifique "note*".  

> [!NOTE]  
>  ¿No puede utilizar un símbolo * o ? como primer carácter de la búsqueda.  
>  No se realiza ningún análisis de texto en consultas de búsqueda con caracteres comodín. En tiempo de consulta, los términos de consulta con caracteres comodín se comparan con los términos analizados en el índice de búsqueda y expandidos.

## <a name="see-also"></a>Vea también  

+ [Buscar en documentos](https://docs.microsoft.com/rest/api/searchservice/Search-Documents)
+ [OData expression syntax for filters and sorting](query-odata-filter-orderby-syntax.md) (Sintaxis de expresiones de OData para filtros y ordenación)   
+ [Simple query syntax in Azure Search](query-simple-syntax.md) (Sintaxis de consulta simple en Azure Search)   
