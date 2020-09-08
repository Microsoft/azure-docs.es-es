---
title: Términos parciales, patrones y caracteres especiales
titleSuffix: Azure Cognitive Search
description: Use consultas de caracteres comodín, expresiones regulares y prefijos para que coincidan con términos completos o parciales en una solicitud de consulta de Azure Cognitive Search. Los patrones de coincidencia compleja que incluyen caracteres especiales pueden resolverse mediante la sintaxis de consulta completa y los analizadores personalizados.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 357f44149cb17976556c1e4609f6f2af531b80ee
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935779"
---
# <a name="partial-term-search-and-patterns-with-special-characters-wildcard-regex-patterns"></a>Búsqueda de términos parciales y patrones con caracteres especiales (carácter comodín, expresión regular y patrones)

Una *búsqueda de términos parciales* hace referencia a las consultas que se componen de fragmentos de términos, en las que en lugar de un término completo, es posible que tenga solo el inicio, el centro o el final del término (a veces se denominan consultas de prefijo, infijo o sufijo). Una búsqueda de términos parciales podría ser una combinación de fragmentos, a menudo con caracteres especiales, como guiones o barras diagonales, que forman parte de la cadena de consulta. Entre los casos de uso comunes se incluyen partes de un número de teléfono, una dirección URL, códigos o palabras compuestas con guiones.

La búsqueda de términos parciales y las cadenas de consulta que incluyen caracteres especiales pueden resultar problemáticas si el índice no tiene tokens con el formato esperado. Durante la [fase de análisis léxico](search-lucene-query-architecture.md#stage-2-lexical-analysis) de la indexación, con el analizador estándar predeterminado, los caracteres especiales se descartan, las palabras compuestas se dividen y los espacios en blanco se eliminan, lo que puede provocar un error en las consultas cuando no se encuentra ninguna coincidencia. Por ejemplo, un número de teléfono como `+1 (425) 703-6214` (tokenizado como `"1"`, `"425"`, `"703"`, `"6214"`) no se mostrará en una consulta `"3-62"` porque ese contenido no existe realmente en el índice. 

La solución consiste en invocar un analizador durante la indexación que conserve una cadena completa, incluidos espacios y caracteres especiales, si es necesario, para que pueda incluir los espacios y caracteres en la cadena de consulta. Del mismo modo, tener una cadena completa que no se convierte en un token en partes más pequeñas permite la coincidencia de patrones para las consultas de tipo "empieza con" o "finaliza con", donde el patrón proporcionado se puede evaluar con respecto a un término que no se transforma por el análisis léxico. La creación de un campo adicional para una cadena intacta, además de usar un analizador de conservación de contenido que emite tokens a largo plazo, es la solución tanto para la coincidencia de patrones como para buscar coincidencias en cadenas de consulta que incluyen caracteres especiales.

> [!TIP]
> Si está familiarizado con las API de Postman y REST, [descargue la colección de ejemplos de consultas](https://github.com/Azure-Samples/azure-search-postman-samples/) para consultar los términos parciales y caracteres especiales que se describen en este artículo.

## <a name="what-is-partial-term-search-in-azure-cognitive-search"></a>Definición de la búsqueda de términos parciales en Azure Cognitive Search

Azure Cognitive Search examina si hay términos con tokens completos en el índice y no encontrará una coincidencia en un término parcial a menos que incluya operadores de marcadores de posición de caracteres comodín (`*` y `?`), o aplique un formato de expresión regular a la consulta. Los términos parciales se especifican mediante estas técnicas:

+ Las [consultas de expresiones regulares](query-lucene-syntax.md#bkmk_regex) pueden ser cualquier expresión regular que sea válida en Apache Lucene. 

+ Los [operadores de caracteres comodín con coincidencia de prefijos](query-simple-syntax.md#prefix-search) hacen referencia a un patrón reconocido generalmente que incluye el principio de un término, seguido de los operadores de sufijo `*` o `?`, como la coincidencia de `search=cap*` en "Cap'n Jack's Waterfront Inn" o "Gacc Capital". La coincidencia de prefijos es compatible con la sintaxis de consulta de Lucene simple y completa.

+ El [carácter comodín con coincidencia de infijos y sufijos](query-lucene-syntax.md#bkmk_wildcard) coloca los operadores `*` y `?` dentro o al principio de un término y requiere la sintaxis de expresiones regulares (donde la expresión se incluye entre barras diagonales). Por ejemplo, la cadena de consulta (`search=/.*numeric*./`) devuelve resultados en "alphanumeric" y "alphanumerical" como coincidencias de sufijo e infijo.

En el caso de la búsqueda de términos parciales o patrones, y en algunos otros formularios de consulta como la búsqueda aproximada, los analizadores no se usan en el momento de la consulta. En el caso de estos formularios de consulta, en los que el analizador detecta por la presencia de operadores y delimitadores, la cadena de consulta se pasa al motor sin análisis léxico. En estos formularios de consulta, el analizador especificado en el campo se omite.

> [!NOTE]
> Cuando una cadena de consulta parcial incluye caracteres, como barras en un fragmento de dirección URL, puede que tenga que agregar caracteres de escape. En JSON, una barra diagonal `/` se escapa con una barra diagonal inversa `\`. Así, `search=/.*microsoft.com\/azure\/.*/` es la sintaxis del fragmento de dirección URL "microsoft.com/azure/".

## <a name="solving-partialpattern-search-problems"></a>Solución de problemas de búsqueda de términos parciales o patrones

Si necesita buscar en fragmentos, patrones o caracteres especiales, puede invalidar el analizador predeterminado con un analizador personalizado que funcione con reglas de tokenización más sencillas y conserve toda la cadena en el índice. Si realiza un paso atrás, el enfoque tiene el siguiente aspecto:

+ Defina un campo para almacenar una versión intacta de la cadena (suponiendo que desee texto analizado y no analizado en el momento de la consulta).
+ Evalúe y elija entre los distintos analizadores que emiten tokens en el nivel de granularidad adecuado.
+ Asigne el analizador al campo.
+ Compile y pruebe el índice.

> [!TIP]
> La evaluación de analizadores es un proceso iterativo que requiere recompilaciones frecuentes de índices. Para simplificar este paso, puede usar Postman y las API REST para [crear un índice](/rest/api/searchservice/create-index), [eliminar un índice](/rest/api/searchservice/delete-index),[cargar documentos](/rest/api/searchservice/addupdate-or-delete-documents)y [buscar documentos](/rest/api/searchservice/search-documents). Para la carga de documentos, el cuerpo de la solicitud debe contener un pequeño conjunto de datos representativo que quiera probar (por ejemplo, un campo con números de teléfono o códigos de producto). Con estas API en la misma colección de Postman, puede seguir estos pasos rápidamente.

## <a name="duplicate-fields-for-different-scenarios"></a>Campos duplicados para diferentes escenarios

Los analizadores determinan cómo se acortan los términos en un índice. Dado que los analizadores se asignan por campo, puede crear campos en el índice para optimizar los distintos escenarios. Por ejemplo, puede definir "featureCode" y "featureCodeRegex" para admitir la búsqueda de texto completo normal en el primero y la coincidencia de patrones avanzada en el segundo. Los analizadores asignados a cada campo determinan cómo se acorta el contenido de cada campo en el índice.  

```json
{
  "name": "featureCode",
  "type": "Edm.String",
  "retrievable": true,
  "searchable": true,
  "analyzer": null
},
{
  "name": "featureCodeRegex",
  "type": "Edm.String",
  "retrievable": true,
  "searchable": true,
  "analyzer": "my_custom_analyzer"
},
```

## <a name="choose-an-analyzer"></a>Selección de un analizador

Al elegir un analizador que produce tokens de términos completos, los siguientes analizadores son opciones comunes:

| Analizador | Comportamientos |
|----------|-----------|
| [Analizadores de idiomas](index-add-language-analyzers.md) | Conserva los guiones en palabras compuestas o cadenas, mutaciones vocálicas y formas verbales. Si los patrones de consulta incluyen guiones, el uso de un analizador de idioma puede ser suficiente. |
| [keyword](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html) | El contenido de todo el campo se acorta para pasar a ser un solo término. |
| [whitespace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/WhitespaceAnalyzer.html) | Solo se separa en espacios en blanco. Los términos que incluyen guiones u otros caracteres se tratan como un solo token. |
| [analizador personalizado](index-add-custom-analyzers.md) | (Recomendado) La creación de un analizador personalizado le permite especificar el filtro de tokens y el tokenizador. Los analizadores anteriores deben usarse tal cual. Un analizador personalizado le permite elegir los filtros de token y los tokenizadores que quiere usar. <br><br>Una combinación recomendada es el [tokenizador de palabras claves](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordTokenizer.html) con un [filtro de tokens en minúsculas](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/LowerCaseFilter.html). Por sí solo, el [analizador de palabras clave](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html) predefinido no pasa a minúsculas ningún texto en mayúsculas, lo que puede provocar errores en las consultas. Un analizador personalizado le ofrece un mecanismo para agregar el filtro de tokens en minúsculas. |

Si usa una herramienta de prueba de API web como Postman, puede agregar la [llamada de REST del analizador de pruebas](/rest/api/searchservice/test-analyzer) para inspeccionar la salida con tokens.

Debe tener un índice relleno con el que trabajar. Si se da una situación en que tiene un índice y un campo que contiene guiones o términos parciales, puede probar varios analizadores en términos específicos para ver qué tokens se emiten.  

1. En primer lugar, compruebe el analizador estándar para ver cómo se acortan los términos de forma predeterminada.

   ```json
   {
   "text": "SVP10-NOR-00",
   "analyzer": "standard"
   }
    ```

1. Evalúe la respuesta para ver cómo se acorta el texto en el índice. Observe que cada término está en minúsculas y dividido. Solo las consultas que coincidan con estos tokens devolverán este documento en los resultados. Se producirá un error en una consulta que incluya "10-NOR".

    ```json
    {
        "tokens": [
            {
                "token": "svp10",
                "startOffset": 0,
                "endOffset": 5,
                "position": 0
            },
            {
                "token": "nor",
                "startOffset": 6,
                "endOffset": 9,
                "position": 1
            },
            {
                "token": "00",
                "startOffset": 10,
                "endOffset": 12,
                "position": 2
            }
        ]
    }
    ```
1. Ahora, modifique la solicitud para usar el analizador `whitespace` o `keyword`:

    ```json
    {
    "text": "SVP10-NOR-00",
    "analyzer": "keyword"
    }
    ```

1. Ahora la respuesta se compone de un solo token escrito en mayúsculas con guiones que se mantienen como parte de la cadena. Si necesita realizar una búsqueda en un patrón o un término parcial, como "10-NOR", el motor de consultas ahora cuenta con la base para encontrar una coincidencia.


    ```json
    {

        "tokens": [
            {
                "token": "SVP10-NOR-00",
                "startOffset": 0,
                "endOffset": 12,
                "position": 0
            }
        ]
    }
    ```
> [!Important]
> Tenga en cuenta que los analizadores de consultas a menudo usan términos en minúsculas en una expresión de búsqueda al compilar el árbol de consulta. Si usa un analizador que no introduce las entradas de texto en minúsculas durante la indexación y no obtiene los resultados esperados, este podría ser el motivo. La solución consiste en agregar un filtro de tokens en minúsculas, tal como se describe en la sección "Uso de analizadores personalizados" más adelante.

## <a name="configure-an-analyzer"></a>Configuración de un analizador
 
Tanto si está evaluando analizadores como si avanza con una configuración concreta, tendrá que especificar el analizador en la definición del campo y, posiblemente, deberá configurar el analizador en sí si no usa ningún analizador integrado. Al intercambiar analizadores, normalmente debe volver a generar el índice (quitarlo, volverlo a crear y cargarlo de nuevo). 

### <a name="use-built-in-analyzers"></a>Uso de analizadores integrados

Los analizadores predefinidos o integrados se pueden especificar por su nombre en una propiedad `analyzer` de una definición de campo, sin necesidad de establecer configuración adicional en el índice. En el ejemplo siguiente se muestra cómo debería establecer el analizador `whitespace` en un campo. 

Para ver otros escenarios y obtener más información sobre otros analizadores integrados, consulte [Lista de analizadores predefinidos](/azure/search/index-add-custom-analyzers#predefined-analyzers-reference). 

```json
    {
      "name": "phoneNumber",
      "type": "Edm.String",
      "key": false,
      "retrievable": true,
      "searchable": true,
      "analyzer": "whitespace"
    }
```

### <a name="use-custom-analyzers"></a>Uso de analizadores personalizados

Si usa un [analizador personalizado](index-add-custom-analyzers.md), debe definirlo en el índice con una combinación definida por el usuario de tokenizador, filtro de tokens y posibles valores de configuración. A continuación, haga referencia a él en una definición de campo, tal como lo haría con un analizador integrado.

Cuando el objetivo sea la tokenización de términos completos, le recomendamos que use un analizador personalizado que conste de un **tokenizador de palabras clave** y un **filtro de tokens en minúsculas**.

+ El tokenizador de palabras clave crea un solo token para todo el contenido de un campo.
+ El filtro de tokens en minúsculas transforma las letras mayúsculas en texto en minúsculas. Los analizadores de consultas suelen convertir en minúsculas cualquier entrada de texto en mayúsculas. El uso de minúsculas homogeneiza las entradas con los términos acortados.

En el ejemplo siguiente se muestra un analizador personalizado que proporciona el tokenizador de palabras clave y un filtro de tokens en minúsculas.

```json
{
"fields": [
  {
  "name": "accountNumber",
  "analyzer":"myCustomAnalyzer",
  "type": "Edm.String",
  "searchable": true,
  "filterable": true,
  "retrievable": true,
  "sortable": false,
  "facetable": false
  }
],

"analyzers": [
  {
  "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
  "name":"myCustomAnalyzer",
  "charFilters":[],
  "tokenizer":"keyword_v2",
  "tokenFilters":["lowercase"]
  }
],
"tokenizers":[],
"charFilters": [],
"tokenFilters": []
```

> [!NOTE]
> El sistema conoce el tokenizador `keyword_v2` y el filtro de tokens `lowercase` y estos utilizan sus configuraciones predeterminadas, por lo que puede hacer referencia a ellos por su nombre sin tener que definirlos primero.

## <a name="build-and-test"></a>Compilación y prueba

Una vez que haya definido un índice con analizadores y definiciones de campo que admitan su escenario, cargue documentos que tengan cadenas representativas para poder probar consultas de cadenas parciales. 

La lógica se explicó en las secciones anteriores. En esta sección se explican las API que debe llamar al probar la solución. Como se indicó anteriormente, si usa una herramienta de prueba web interactiva, como Postman, puede realizar estas tareas rápidamente.

+ [Eliminación de índice](/rest/api/searchservice/delete-index) quita un índice existente con el mismo nombre para que pueda volver a crearlo.

+ [Creación de índice](/rest/api/searchservice/create-index) crea la estructura de índice en el servicio de búsqueda, incluidas las definiciones y los campos, así como una especificación, del analizador.

+ [Carga de documentos](/rest/api/searchservice/addupdate-or-delete-documents) importa documentos que tienen la misma estructura que el índice, así como contenido que se puede buscar. Después de este paso, el índice está listo para realizar consultas o pruebas.

+ El [analizador de pruebas](/rest/api/searchservice/test-analyzer) se presentó en [Selección de un analizador](#choose-an-analyzer). Pruebe algunas de las cadenas del índice con diversos analizadores para entender cómo se acortan los términos.

+ [Búsqueda de documentos](/rest/api/searchservice/search-documents) explica cómo construir una solicitud de consulta mediante una [sintaxis simple](query-simple-syntax.md) o la [sintaxis de Lucene completa](query-lucene-syntax.md) para caracteres comodín y expresiones regulares.

  En el caso de las consultas de términos parciales, como la consulta de "3-6214" para encontrar una coincidencia en "+ 1 (425) 703-6214", puede usar la sintaxis simple: `search=3-6214&queryType=simple`.

  En el caso de las consultas de infijos y de sufijos, como consultar "num" o "numéric" para encontrar una coincidencia en "alfanumérico", use la sintaxis completa de Lucene y una expresión regular: `search=/.*num.*/&queryType=full`

## <a name="tune-query-performance"></a>Ajustar rendimiento de consulta

Si implementa la configuración recomendada que incluye el tokenizador keyword_v2 y el filtro de tokens en minúsculas, es posible que observe una disminución en el rendimiento de las consultas debido al procesamiento adicional de los filtros de tokens en los tokens existentes en el índice. 

En el ejemplo siguiente se agrega el elemento [EdgeNGramTokenFilter](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/EdgeNGramTokenizer.html) para crear coincidencias de prefijo de manera más rápida. Se generan tokens adicionales para combinaciones de 2 a 25 caracteres que incluyen los caracteres siguientes: (no solo MS, MSF, MSFT, MSFT/, MSFT/S, MSFT/SQ, MSFT/SQL). 

Como puede imaginar, la tokenización adicional da como resultado un índice mayor. Si tiene capacidad suficiente para alojar el índice más grande, este enfoque con su tiempo de respuesta más rápido podría ser una solución mejor.

```json
{
"fields": [
  {
  "name": "accountNumber",
  "analyzer":"myCustomAnalyzer",
  "type": "Edm.String",
  "searchable": true,
  "filterable": true,
  "retrievable": true,
  "sortable": false,
  "facetable": false
  }
],

"analyzers": [
  {
  "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
  "name":"myCustomAnalyzer",
  "charFilters":[],
  "tokenizer":"keyword_v2",
  "tokenFilters":["lowercase", "my_edgeNGram"]
  }
],
"tokenizers":[],
"charFilters": [],
"tokenFilters": [
  {
  "@odata.type":"#Microsoft.Azure.Search.EdgeNGramTokenFilterV2",
  "name":"my_edgeNGram",
  "minGram": 2,
  "maxGram": 25,
  "side": "front"
  }
]
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo se explica cómo contribuyen ambos analizadores a los problemas de consultas y a cómo solucionarlos. A continuación, examine más detenidamente el impacto del analizador en la indexación y el procesamiento de consultas. En concreto, considere la posibilidad de usar la API de análisis de texto para devolver la salida con tokens, de modo que pueda ver exactamente lo que crea un analizador para el índice.

+ [Analizadores de idiomas](search-language-support.md)
+ [Analizadores para el procesamiento de texto en Azure Cognitive Search](search-analyzers.md)
+ [API de análisis de texto (REST)](/rest/api/searchservice/test-analyzer)
+ [Cómo funciona la búsqueda de texto completo (arquitectura de consultas)](search-lucene-query-architecture.md)