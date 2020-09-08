---
title: Incorporación de analizadores personalizados a campos de cadena
titleSuffix: Azure Cognitive Search
description: Configure los tokenizadores de texto y los filtros de caracteres utilizados en consultas de búsqueda de texto completo de Azure Cognitive Search.
author: HeidiSteen
manager: nitinme
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/05/2020
ms.openlocfilehash: 8a9f8b02d8821c76b2a7766f35244834934db1d0
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935915"
---
# <a name="add-custom-analyzers-to-string-fields-in-an-azure-cognitive-search-index"></a>Incorporación de analizadores personalizados a campos de cadena en un índice de Azure Cognitive Search

Un *analizador personalizado* es un tipo específico de [analizador de texto](search-analyzers.md) que se compone de una combinación definida por el usuario de un tokenizador existente y unos filtros opcionales. Si combina los tokenizadores y los filtros de nuevas formas, puede personalizar el procesamiento de texto del motor de búsqueda para lograr resultados específicos. Por ejemplo, podría crear un analizador personalizado con un *filtro de caracteres* para quitar el marcado HTML antes de que se tokenicen las entradas de texto.

 Puede definir varios analizadores personalizados para variar la combinación de filtros, pero cada campo solo puede usar un analizador para el análisis de indexación y otro para el análisis de la búsqueda. Para ver una ilustración del aspecto que tendría un analizador de clientes, consulte un [ejemplo de un analizador personalizado](search-analyzers.md#Custom-analyzer-example).

## <a name="overview"></a>Información general

 El rol de un [motor de búsqueda de texto completo](search-lucene-query-architecture.md), en términos simples, es procesar y almacenar documentos de manera que sea posible realizar de forma eficiente consultas y recuperación. A nivel general, se trata en definitiva de extraer palabras importantes de los documentos, colocarlas en un índice y luego utilizar el índice para buscar documentos que coincidan con las palabras de una consulta determinada. El proceso que extrae palabras de documentos y consultas de búsqueda se denomina *análisis léxico*. Los componentes que realizan el análisis léxico se denominan *analizadores*.

 En Azure Cognitive Search, puede elegir entre el conjunto de analizadores predefinidos independientes del idioma que aparecen en la tabla [Analizadores](#AnalyzerTable) o los analizadores específicos de idioma que se indican en [Analizadores de idiomas &#40;API REST de Azure Cognitive Search Service&#41;](index-add-language-analyzers.md). También tiene una opción para definir sus propios analizadores personalizados.  

 Un analizador personalizado le permite hacerse con el control del proceso de conversión de texto en tokens que se pueden indexar o buscar. Se trata de una configuración definida por el usuario que consta de un único tokenizador predefinido, uno o más filtros de token y uno o varios filtros de caracteres. El tokenizer es el responsable de dividir texto en tokens y los filtros de token son los que modifican los tokens emitidos por el tokenizer. Los filtros de caracteres se aplican para preparar el texto de entrada antes de que el tokenizador lo procese. Por ejemplo, el filtro de caracteres puede reemplazar determinados caracteres o símbolos.

 Escenarios populares habilitados por analizadores personalizados incluyen:  

- Búsqueda fonética. Agregue un filtro fonético para habilitar la búsqueda basada en cómo suena una palabra en lugar de en cómo se escribe.  

- Deshabilitar el análisis léxico. Use el analizador de palabras clave para crear campos de búsqueda que no se analizan.  

- Búsqueda rápida de prefijo o sufijo. Agregue el filtro de token de n-gramas perimetrales a prefijos de índice de palabras para habilitar la coincidencia de prefijo rápida. Combínelo con el filtro de token de tipo Reverse para habilitar la coincidencia de sufijo.  

- Tokenización personalizada. Por ejemplo, use el tokenizador Whitespace para dividir las oraciones en tokens usando el espacio en blanco como delimitador.  

- Plegado ASCII. Agregue el filtro de plegado ASCII estándar para normalizar los signos diacríticos como ö o ê en los términos de búsqueda.  

  En esta página se proporciona una lista de analizadores, tokenizadores, filtros de token y filtros de caracteres admitidos. También puede encontrar una descripción de los cambios en la definición del índice con un ejemplo de uso. Para más información sobre la tecnología subyacente aprovechada en la implementación de Azure Cognitive Search, consulte [el resumen del paquete de análisis Lucene](https://lucene.apache.org/core/6_0_0/core/org/apache/lucene/codecs/lucene60/package-summary.html). Para ver ejemplos de las configuraciones del analizador, consulte [Incorporación de analizadores en Azure Cognitive Search](search-analyzers.md#examples).

## <a name="validation-rules"></a>Reglas de validación  
 Los nombres de los analizadores, tokenizadores, filtros de token y filtros de caracteres tienen que ser únicos y no pueden ser los mismos que cualquiera de los analizadores, tokenizadores, filtros de token o filtros de caracteres predefinidos. Consulte la sección [Referencia de propiedades](#PropertyReference) para los nombres que ya están en uso.

## <a name="create-custom-analyzers"></a>Creación de analizadores personalizados
 Puede definir analizadores personalizados en el momento de la creación de índices. En esta sección se describe la sintaxis para especificar un analizador personalizado. Para familiarizarse con la sintaxis, también puede revisar las definiciones de muestra de [Incorporación de analizadores en Azure Cognitive Search](search-analyzers.md#examples).  

 Una definición de analizador incluye un nombre, un tipo, uno o varios filtros de caracteres, un máximo de un tokenizador y uno o varios filtros de token para el procesamiento posterior a la tokenización. Los filtros de caracteres se aplican antes de la tokenización. Los filtros de token y los filtros de caracteres se aplican de izquierda a derecha.

 `tokenizer_name` es el nombre de un tokenizador, `token_filter_name_1` y `token_filter_name_2` son los nombres de los filtros de token, y `char_filter_name_1` y `char_filter_name_2` son los nombres de los filtros de caracteres (consulte las tablas [Tokenizador](#Tokenizers), [ Filtros de token](#TokenFilters) y Filtros de caracteres para ver los valores válidos).

La definición del analizador es una parte del índice más grande. Consulte [Create Index API](/rest/api/searchservice/create-index) (Creación de API de índice) para información sobre el resto del índice.

```
"analyzers":(optional)[
   {
      "name":"name of analyzer",
      "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
      "charFilters":[
         "char_filter_name_1",
         "char_filter_name_2"
      ],
      "tokenizer":"tokenizer_name",
      "tokenFilters":[
         "token_filter_name_1",
         "token_filter_name_2"
      ]
   },
   {
      "name":"name of analyzer",
      "@odata.type":"#analyzer_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"charFilters":(optional)[
   {
      "name":"char_filter_name",
      "@odata.type":"#char_filter_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"tokenizers":(optional)[
   {
      "name":"tokenizer_name",
      "@odata.type":"#tokenizer_type",
      "option1":value1,
      "option2":value2,
      ...
   }
],
"tokenFilters":(optional)[
   {
      "name":"token_filter_name",
      "@odata.type":"#token_filter_type",
      "option1":value1,
      "option2":value2,
      ...
   }
]
```

> [!NOTE]  
>  Los analizadores personalizados que cree, no se exponen en el portal de Azure. La única manera de agregar un analizador personalizado es a través del código que realiza llamadas a la API cuando se define un índice.  

 Dentro de una definición de índice, puede colocar esta sección en cualquier lugar del cuerpo de una solicitud de creación de índice, pero normalmente va al final:  

```
{
  "name": "name_of_index",
  "fields": [ ],
  "suggesters": [ ],
  "scoringProfiles": [ ],
  "defaultScoringProfile": (optional) "...",
  "corsOptions": (optional) { },
  "analyzers":(optional)[ ],
  "charFilters":(optional)[ ],
  "tokenizers":(optional)[ ],
  "tokenFilters":(optional)[ ]
}
```

Las definiciones para los filtros de caracteres, tokenizadores y filtros de token se agregan al índice solo si está configurando las opciones personalizadas. Para utilizar un filtro existente o tokenizador como está, especifíquelo por nombre en la definición del analizador.

<a name="Testing custom analyzers"></a>

## <a name="test-custom-analyzers"></a>Prueba de analizadores personalizados

Puede usar la **operación Probar analizador** de la [API REST](/rest/api/searchservice/test-analyzer) para ver cómo un analizador descompone el texto dado en tokens.

**Solicitud**
```
  POST https://[search service name].search.windows.net/indexes/[index name]/analyze?api-version=[api-version]
  Content-Type: application/json
    api-key: [admin key]

  {
     "analyzer":"my_analyzer",
     "text": "Vis-à-vis means Opposite"
  }
```
**Respuesta**
```
  {
    "tokens": [
      {
        "token": "vis_a_vis",
        "startOffset": 0,
        "endOffset": 9,
        "position": 0
      },
      {
        "token": "vis_à_vis",
        "startOffset": 0,
        "endOffset": 9,
        "position": 0
      },
      {
        "token": "means",
        "startOffset": 10,
        "endOffset": 15,
        "position": 1
      },
      {
        "token": "opposite",
        "startOffset": 16,
        "endOffset": 24,
        "position": 2
      }
    ]
  }
```

## <a name="update-custom-analyzers"></a>Actualización de analizadores personalizados

Una vez definido un analizador, un tokenizador, un filtro de token o un filtro de caracteres, no se puede modificar. Se pueden agregar unos nuevos a un índice existente solo si la marca de `allowIndexDowntime` está establecida en true en la solicitud de actualización del índice:

```
PUT https://[search service name].search.windows.net/indexes/[index name]?api-version=[api-version]&allowIndexDowntime=true
```

Esta operación toma el índice sin conexión durante al menos unos segundos, provocando un error en la indexación y las solicitudes de consulta. El rendimiento y la disponibilidad de escritura del índice pueden ser desiguales durante varios minutos después de que se actualice el índice, o durante más tiempo en el caso de índices muy grandes, pero estos efectos son temporales y, al final, se resuelven por sí solos.

 <a name="ReferenceIndexAttributes"></a>

## <a name="analyzer-reference"></a>Referencia de los analizadores

En las tablas siguientes se enumeran las propiedades de configuración para los analizadores, tokenizadores, filtros de token y filtros de caracteres de una definición de índice. La estructura de un analizador, tokenizador o filtro en el índice se compone de estos atributos. Para información de asignación de valor, vea la sección [Referencia de propiedades](#PropertyReference).

### <a name="analyzers"></a>Analizadores

Para los analizadores, los atributos de índice varían dependiendo de si usa analizadores predefinidos o personalizados.

#### <a name="predefined-analyzers"></a>Analizadores predefinidos

| Tipo | Descripción |
| ---- | ----------- |  
|Nombre|Solo puede contener letras, dígitos, espacios, guiones o guiones bajos, debe empezar y acabar con caracteres alfanuméricos y no puede superar los 128 caracteres.|  
|Tipo|Tipo de analizador de la lista de analizadores admitidos. Consulte la columna **analyzer_type** de la tabla [Analizadores](#AnalyzerTable) siguiente.|  
|Opciones|Deben ser opciones válidas de un analizador predefinido que se muestran en la tabla [Analizadores](#AnalyzerTable) siguiente.|  

#### <a name="custom-analyzers"></a>Analizadores personalizados

| Tipo | Descripción |
| ---- | ----------- |  
|Nombre|Solo puede contener letras, dígitos, espacios, guiones o guiones bajos, debe empezar y acabar con caracteres alfanuméricos y no puede superar los 128 caracteres.|  
|Tipo|Debe ser "#Microsoft.Azure.Search.CustomAnalyzer".|  
|CharFilters|Establézcalo en uno de los filtros de caracteres predefinidos enumerados en la tabla [Filtros de caracteres](#char-filters-reference) o en un filtro de caracteres personalizado especificado en la definición del índice.|  
|Tokenizador|Necesario. Establézcalo en uno de los tokenizadores predefinidos enumerados en la tabla [Tokenizadores](#Tokenizers) siguiente o en un tokenizador personalizado especificado en la definición del índice.|  
|TokenFilters|Establézcalo en uno de los filtros de token predefinidos enumerados en la tabla [Filtros de token](#TokenFilters) o en un filtro de token personalizado especificado en la definición del índice.|  

> [!NOTE]
> Es necesario que configure su analizador personalizado para no generar tokens de más de 300 caracteres. La indexación no se puede realizar para documentos con esos tokens. Para recortarlos u omitirlos, utilice **TruncateTokenFilter** y **LengthTokenFilter** respectivamente.  Compruebe [**Filtros de Token**](#TokenFilters) como referencia.

<a name="CharFilter"></a>

### <a name="char-filters"></a>Filtros de caracteres

 Un filtro de caracteres se usa para preparar el texto de entrada antes de que el tokenizador lo procese. Por ejemplo, pueden reemplazar determinados caracteres o símbolos. Puede tener varios filtros de caracteres en un analizador personalizado. Los filtros de caracteres se ejecutan en el orden en el que aparecen.  

| Tipo | Descripción |
| ---- | ----------- | 
|Nombre|Solo puede contener letras, dígitos, espacios, guiones o guiones bajos, debe empezar y acabar con caracteres alfanuméricos y no puede superar los 128 caracteres.|  
|Tipo|Tipo de filtro de caracteres de la lista de filtros de caracteres admitidos. Consulte la columna **char_filter_type** de la tabla [Filtros de caracteres](#char-filters-reference) siguiente.|  
|Opciones|Deben ser opciones válidas de un tipo [Filtros de caracteres](#char-filters-reference) dado.|  

### <a name="tokenizers"></a>Tokenizer

 Un tokenizer divide un texto continuo en una secuencia de tokens, como una frase se divide en palabras.  

 Puede especificar exactamente un tokenizer por analizador personalizado. Si necesita más de un tokenizer, puede crear varios analizadores personalizados y asignarlos campo por campo en el esquema de índice.  
Un analizador personalizado puede usar un tokenizer predefinido con opciones predeterminadas o personalizadas.  

| Tipo | Descripción |
| ---- | ----------- | 
|Nombre|Solo puede contener letras, dígitos, espacios, guiones o guiones bajos, debe empezar y acabar con caracteres alfanuméricos y no puede superar los 128 caracteres.|  
|Tipo|Nombre de tokenizador de la lista de tokenizadores admitidos. Consulte la columna **tokenizer_type** de la tabla [Tokenizadores](#Tokenizers) siguiente.|  
|Opciones|Deben ser opciones válidas de un tipo de tokenizador determinado enumerado en la tabla [Tokenizadores](#Tokenizers) siguiente.|  

### <a name="token-filters"></a>Filtros de token

 Un filtro de token se utiliza para filtrar o modificar los tokens generados por un tokenizer. Por ejemplo, puede especificar un filtro en minúsculas que convierte todos los caracteres a minúsculas.   
Puede tener varios filtros de token en un analizador personalizado. Los filtros de token se ejecutan en el orden en que aparecen.  

| Tipo | Descripción |
| ---- | ----------- |  
|Nombre|Solo puede contener letras, dígitos, espacios, guiones o guiones bajos, debe empezar y acabar con caracteres alfanuméricos y no puede superar los 128 caracteres.|  
|Tipo|Nombre de filtro de token de la lista de filtros de token admitidos. Consulte la columna **token_filter_type** de la tabla [Filtros de token](#TokenFilters) siguiente.|  
|Opciones|Deben ser [filtros de token](#TokenFilters) de un tipo de filtro de token dado.|  

<a name="PropertyReference"></a>  

## <a name="property-reference"></a>Referencia de propiedades

En esta sección se proporcionan los valores válidos para los atributos especificados en la definición de un analizador, tokenizador, filtro de caracteres o filtro de token personalizado en el índice. Los analizadores, tokenizadores y filtros que se implementan mediante Apache Lucene tienen vínculos a documentación de la API Lucene.

<a name="AnalyzerTable"></a>

###  <a name="predefined-analyzers-reference"></a>Referencia de analizadores predefinidos

|**analyzer_name**|**analyzer_type**  <sup>1</sup>|**Descripción y opciones**|  
|-|-|-|  
|[keyword](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html)| (el tipo solo se aplica cuando las opciones están disponibles) |Trata todo el contenido de un campo como un solo token. Esto es útil para los datos como códigos postales, identificadores y algunos nombres de producto.|  
|[pattern](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/miscellaneous/PatternAnalyzer.html)|PatternAnalyzer|Separa el texto de manera flexible en términos a través de un patrón de expresión regular.<br /><br /> **Opciones**<br /><br /> lowercase (tipo: booleano): determina si los términos se ponen en minúsculas. El valor predeterminado es true.<br /><br /> [pattern](https://docs.oracle.com/javase/7/docs/api/java/util/regex/Pattern.html?is-external=true) (tipo: cadena): un patrón de expresión regular para que coincida con los separadores de token. El valor predeterminado es `\W+`, que coincide con los caracteres que no son una palabra.<br /><br /> [flags](https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html#field_summary) (tipo: cadena): marcas de expresión regular. El valor predeterminado es una cadena vacía. Valores permitidos: CANON_EQ, CASE_INSENSITIVE, COMMENTS, DOTALL, LITERAL, MULTILINE, UNICODE_CASE, UNIX_LINES<br /><br /> stopwords (tipo: matriz de cadenas): una lista de palabras irrelevantes. El valor predeterminado es una lista vacía.|  
|[simple](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/SimpleAnalyzer.html)|(el tipo solo se aplica cuando las opciones están disponibles) |Divide el texto por donde no hay letras y lo convierte en minúsculas. |  
|[standard](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/analysis/standard/StandardAnalyzer.html) <br />(También se denomina standard.lucene)|StandardAnalyzer|Analizador de Lucene estándar, formado por el tokenizador estándar, filtro de minúsculas y filtro de palabras irrelevantes.<br /><br /> **Opciones**<br /><br /> maxTokenLength (tipo: entero): la longitud máxima del token. El valor predeterminado es 255. Los tokens que sobrepasen la longitud máxima se dividen. La longitud máxima del token que se puede usar es 300 caracteres.<br /><br /> stopwords (tipo: matriz de cadenas): una lista de palabras irrelevantes. El valor predeterminado es una lista vacía.|  
|standardasciifolding.lucene|(el tipo solo se aplica cuando las opciones están disponibles) |Analizador estándar con el filtro de plegado ASCII. |  
|[stop](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/StopAnalyzer.html)|StopAnalyzer|Divide el texto por donde no hay letras y aplica los filtros de token de minúsculas y palabra irrelevante.<br /><br /> **Opciones**<br /><br /> stopwords (tipo: matriz de cadenas): una lista de palabras irrelevantes. El valor predeterminado es una lista predefinida para inglés. |  
|[whitespace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/WhitespaceAnalyzer.html)|(el tipo solo se aplica cuando las opciones están disponibles) |Un analizador que usa el tokenizador whitespace. Los tokens cuya longitud no es superior a 255 caracteres se dividen.|  

 <sup>1</sup> Los tipos de analizadores siempre tienen el prefijo "#Microsoft.Azure.Search" en el código, de forma que "PatternAnalyzer" realmente se especificaría como "#Microsoft.Azure.Search.PatternAnalyzer". Hemos quitado el prefijo por brevedad, pero es obligatorio en el código. 
 
analyzer_type solo se proporciona para analizadores que se pueden personalizar. Si no hay opciones, como sucede con el analizador de palabras clave, no hay ningún tipo #Microsoft.Azure.Search asociado.


<a name="CharFilter"></a>

###  <a name="char-filters-reference"></a>Referencia de filtros de caracteres

En la tabla siguiente, los filtros de caracteres que se implementan mediante Apache Lucene se vinculan a la documentación de la API Lucene.

|**char_filter_name**|**char_filter_type** <sup>1</sup>|**Descripción y opciones**|  
|-|-|-|
|[html_strip](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/charfilter/HTMLStripCharFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Filtro de caracteres que intenta eliminar construcciones HTML.|  
|[mapping](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/charfilter/MappingCharFilter.html)|MappingCharFilter|Un filtro de caracteres que aplica asignaciones definidas con la opción de asignaciones. La coincidencia es ambiciosa (la mayor coincidencia de patrones en un momento dado gana). Se permite que la sustitución sea una cadena vacía.<br /><br /> **Opciones**<br /><br /> mappings (tipo: matriz de cadenas): una lista de asignaciones con el siguiente formato: "a = > b" (todas las repeticiones del carácter "a" se reemplazan por el caracter "b"). Necesario.|  
|[pattern_replace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternReplaceCharFilter.html)|PatternReplaceCharFilter|Un filtro de caracteres que reemplaza los caracteres en la cadena de entrada. Usa una expresión regular para identificar las secuencias de caracteres que se desean conservar y un patrón de reemplazo para identificar los caracteres que se desean reemplazar. Por ejemplo, texto de entrada = "aa  bb aa bb", pattern="(aa)\\\s+(bb)" replacement="$1#$2", resultado = "aa#bb aa#bb".<br /><br /> **Opciones**<br /><br /> pattern (tipo: cadena): necesario.<br /><br /> replacement (tipo: cadena): necesario.|  

 <sup>1</sup> Los tipos de filtro de caracteres siempre tienen el prefijo "#Microsoft.Azure.Search" en el código, de forma que "MappingCharFilter" realmente se especificaría como "#Microsoft.Azure.Search.MappingCharFilter". Hemos quitado el prefijo para reducir el ancho de la tabla, pero recuerde que debe incluirlo en el código. Tenga en cuenta que char_filter_type solo se proporciona para filtros que se pueden personalizar. Si no hay opciones, como sucede con html_strip, no hay ningún tipo #Microsoft.Azure.Search asociado.

<a name="Tokenizers"></a>

###  <a name="tokenizers-reference"></a>Referencia de tokenizadores

En la tabla siguiente, los tokenizadores que se implementan mediante Apache Lucene se vinculan a la documentación de la API Lucene.

|**tokenizer_name**|**tokenizer_type** <sup>1</sup>|**Descripción y opciones**|  
|-|-|-|  
|[clásico](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/standard/ClassicTokenizer.html)|ClassicTokenizer|Tokenizador basado en la gramática que es adecuado para procesar la mayoría de los documentos de idiomas Europaos.<br /><br /> **Opciones**<br /><br /> maxTokenLength (tipo: entero): la longitud máxima del token. Valor predeterminado: 255, valor máximo: 300. Los tokens que sobrepasen la longitud máxima se dividen.|  
|[edgeNGram](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/EdgeNGramTokenizer.html)|EdgeNGramTokenizer|Tokeniza la entrada de un perímetro en n-gramas de tamaños dados.<br /><br /> **Opciones**<br /><br /> minGram (tipo: entero) - Valor predeterminado: 1, valor máximo: 300.<br /><br /> maxGram (tipo: entero) - Valor predeterminado: 2, valor máximo: 300. El valor debe ser mayor que minGram.<br /><br /> tokenChars (tipo: matriz de cadenas): clases de caracteres para mantener en los tokens. Valores permitidos: <br />"letter", "digit", "whitespace", "punctuation" y "symbol". Los valores predeterminados son una matriz vacía: mantiene todos los caracteres. |  
|[keyword_v2](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordTokenizer.html)|KeywordTokenizerV2|Emite la entrada completa como un solo token.<br /><br /> **Opciones**<br /><br /> maxTokenLength (tipo: entero): la longitud máxima del token. Valor predeterminado: 256, valor máximo: 300. Los tokens que sobrepasen la longitud máxima se dividen.|  
|[letter](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/LetterTokenizer.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Divide el texto por donde no hay letras. Los tokens cuya longitud no es superior a 255 caracteres se dividen.|  
|[lowercase](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/LowerCaseTokenizer.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Divide el texto por donde no hay letras y lo convierte en minúsculas. Los tokens cuya longitud no es superior a 255 caracteres se dividen.|  
| microsoft_language_tokenizer| MicrosoftLanguageTokenizer| Divide el texto mediante reglas específicas del idioma.<br /><br /> **Opciones**<br /><br /> maxTokenLength (tipo: entero): la longitud máxima del token. Valor predeterminado: 255, valor máximo: 300. Los tokens que sobrepasen la longitud máxima se dividen. Los tokens de más de 300 caracteres primero se dividen en tokens de 300 de longitud y, luego, cada uno de esos tokens se divide según el valor de maxTokenLength establecido.<br /><br />isSearchTokenizer (tipo: booleano): se establece en true si se usa como el tokenizador de búsqueda; se establece en false si se usa como el tokenizador de indexación. <br /><br /> language (tipo: cadena); idioma que se va a usar; el valor predeterminado es "english". Los valores permitidos son:<br />"bangla", "bulgarian", "catalan", "chineseSimplified",  "chineseTraditional", "croatian", "czech", "danish", "dutch", "english",  "french", "german", "greek", "gujarati", "hindi", "icelandic", "indonesian", "italian", "japanese", "kannada", "korean", "malay", "malayalam", "marathi", "norwegianBokmaal", "polish", "portuguese", "portugueseBrazilian", "punjabi", "romanian", "russian", "serbianCyrillic", "serbianLatin", "slovenian", "spanish", "swedish", "tamil", "telugu", "thai", "ukrainian", "urdu", "vietnamese" |
| microsoft_language_stemming_tokenizer | MicrosoftLanguageStemmingTokenizer| Divide el texto mediante reglas específicas del idioma y reduce las palabras a sus formas base<br /><br /> **Opciones**<br /><br />maxTokenLength (tipo: entero): la longitud máxima del token. Valor predeterminado: 255, valor máximo: 300. Los tokens que sobrepasen la longitud máxima se dividen. Los tokens de más de 300 caracteres primero se dividen en tokens de 300 de longitud y, luego, cada uno de esos tokens se divide según el valor de maxTokenLength establecido.<br /><br /> isSearchTokenizer (tipo: booleano): se establece en true si se usa como el tokenizador de búsqueda; se establece en false si se usa como el tokenizador de indexación.<br /><br /> language (tipo: cadena); idioma que se va a usar; el valor predeterminado es "english". Los valores permitidos son:<br />"arabic", "bangla", "bulgarian", "catalan", "croatian", "czech", "danish", "dutch", "english", "estonian", "finnish", "french", "german", "greek", "gujarati", "hebrew", "hindi", "hungarian", "icelandic", "indonesian", "italian", "kannada", "latvian", "lithuanian", "malay", "malayalam", "marathi", "norwegianBokmaal", "polish", "portuguese", "portugueseBrazilian", "punjabi", "romanian", "russian", "serbianCyrillic", "serbianLatin", "slovak", "slovenian", "spanish", "swedish", "tamil", "telugu", "turkish", "ukrainian", "urdu" |
|[nGram](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/NGramTokenizer.html)|NGramTokenizer|Tokeniza la entrada en n-gramas de tamaños dados.<br /><br /> **Opciones**<br /><br /> minGram (tipo: entero) - Valor predeterminado: 1, valor máximo: 300.<br /><br /> maxGram (tipo: entero) - Valor predeterminado: 2, valor máximo: 300. El valor debe ser mayor que minGram. <br /><br /> tokenChars (tipo: matriz de cadenas): clases de caracteres para mantener en los tokens. Valores permitidos: "letter", "digit", "whitespace", "punctuation" y "symbol". Los valores predeterminados son una matriz vacía: mantiene todos los caracteres. |  
|[path_hierarchy_v2](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/path/PathHierarchyTokenizer.html)|PathHierarchyTokenizerV2|Tokenizador para las jerarquías parecidas a rutas de acceso.<br /><br /> **Opciones**<br /><br /> delimiter (tipo: cadena) - Default: '/.<br /><br /> replacement (tipo: cadena): si está establecido, reemplaza el carácter delimitador. El valor predeterminado es el mismo que el valor del delimitador.<br /><br /> maxTokenLength (tipo: entero): la longitud máxima del token. Valor predeterminado: 300, valor máximo: 300. Las rutas de acceso más largas de maxTokenLength se omiten.<br /><br /> reverse (tipo: booleano): si es true, genera el token en orden inverso. Valor predeterminado: false.<br /><br /> skip (tipo: booleano): tokens iniciales que se omitirán. El valor predeterminado es 0.|  
|[pattern](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternTokenizer.html)|PatternTokenizer|Este tokenizador usa la coincidencia de patrones de expresión regular para construir tokens distintos.<br /><br /> **Opciones**<br /><br /> [pattern](https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html) (tipo: cadena): patrón de expresión regular que coincidirá con los separadores de token. El valor predeterminado es `\W+`, que coincide con los caracteres que no son una palabra. <br /><br /> [flags](https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html#field_summary) (tipo: cadena): marcas de expresión regular. El valor predeterminado es una cadena vacía. Valores permitidos: CANON_EQ, CASE_INSENSITIVE, COMMENTS, DOTALL, LITERAL, MULTILINE, UNICODE_CASE, UNIX_LINES<br /><br /> group (tipo: entero): el grupo para extraer en tokens. El valor predeterminado es 1 (dividir).|
|[standard_v2](https://lucene.apache.org/core/6_6_1/core/org/apache/lucene/analysis/standard/StandardTokenizer.html)|StandardTokenizerV2|Divide el texto siguiendo las [reglas de segmentación de texto Unicode](https://unicode.org/reports/tr29/).<br /><br /> **Opciones**<br /><br /> maxTokenLength (tipo: entero): la longitud máxima del token. Valor predeterminado: 255, valor máximo: 300. Los tokens que sobrepasen la longitud máxima se dividen.|  
|[uax_url_email](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/standard/UAX29URLEmailTokenizer.html)|UaxUrlEmailTokenizer|Tokeniza las direcciones URL y los correos electrónicos como un token.<br /><br /> **Opciones**<br /><br /> maxTokenLength (tipo: entero): la longitud máxima del token. Valor predeterminado: 255, valor máximo: 300. Los tokens que sobrepasen la longitud máxima se dividen.|  
|[whitespace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/WhitespaceTokenizer.html)|(el tipo solo se aplica cuando las opciones están disponibles) |Divide el texto por los espacios en blanco. Los tokens cuya longitud no es superior a 255 caracteres se dividen.|  

 <sup>1</sup> Los tipos de tokenizador siempre tienen el prefijo "#Microsoft.Azure.Search" en el código, de forma que "ClassicTokenizer" realmente se especificaría como "#Microsoft.Azure.Search.ClassicTokenizer". Hemos quitado el prefijo para reducir el ancho de la tabla, pero recuerde que debe incluirlo en el código. Tenga en cuenta que tokenizer_type solo se proporciona para tokenizadores que se puedan personalizar. Si no hay opciones, como sucede con el tokenizador de letras, no hay ningún tipo #Microsoft.Azure.Search asociado.

<a name="TokenFilters"></a>

###  <a name="token-filters-reference"></a>Referencia de filtros de token

En la tabla siguiente, los filtros de token que se implementan mediante Apache Lucene se vinculan a la documentación de la API Lucene.

|**token_filter_name**|**token_filter_type** <sup>1</sup>|**Descripción y opciones**|  
|-|-|-|  
|[arabic_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ar/ArabicNormalizationFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Un filtro de token que aplica el normalizador de árabe para normalizar la ortografía.|  
|[apostrophe](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/tr/ApostropheFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Elimina todos los caracteres después de un apóstrofo (incluido el propio apóstrofo). |  
|[asciifolding](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/ASCIIFoldingFilter.html)|AsciiFoldingTokenFilter|Convierte los caracteres Unicode alfabéticos, numéricos y simbólicos que no están en los primeros 127 caracteres ASCII (el bloque Unicode "Latín básico") en sus valores equivalentes ASCII, si existe uno.<br /><br /> **Opciones**<br /><br /> preserveOriginal (tipo: booleano): si es true, se mantiene el token original. El valor predeterminado es false.|  
|[cjk_bigram](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/cjk/CJKBigramFilter.html)|CjkBigramTokenFilter|Forma bigramas de términos CJK que se generan a partir de StandardTokenizer.<br /><br /> **Opciones**<br /><br /> ignoreScripts (tipo: matriz de cadenas): scripts que se desea pasar por alto. Los valores permitidos son: "han", "hiragana", "katakana" y "hangul". El valor predeterminado es una lista vacía.<br /><br /> outputUnigrams (tipo: bool): establézcalo en true si siempre desea unigramas y bigramas a la salida. El valor predeterminado es false.|  
|[cjk_width](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/cjk/CJKWidthFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Normaliza las diferencias de ancho de CJK. Pliega la variantes de ASCII de ancho completo en el latín básico equivalente y las variantes de Katakana de ancho medio en el kana equivalente. |  
|[clásico](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/standard/ClassicFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Quita los posesivos de inglés y los puntos de los acrónimos. |  
|[common_grams](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/commongrams/CommonGramsFilter.html)|CommonGramTokenFilter|Construye bigramas para términos que se repiten con frecuencia durante la indexación. Los términos individuales también se indexan, con los bigramas superpuestos.<br /><br /> **Opciones**<br /><br /> commonWords (tipo: matriz de cadenas): el conjunto de palabras comunes. El valor predeterminado es una lista vacía. Necesario.<br /><br /> ignoreCase (tipo: booleano): si es true, la coincidencia distingue entre mayúsculas y minúsculas. El valor predeterminado es false.<br /><br /> queryMode (tipo: booleano): genera bigramas y luego quita las palabras comunes y los términos individuales seguidos de una palabra común. El valor predeterminado es false.|  
|[dictionary_decompounder](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/compound/DictionaryCompoundWordTokenFilter.html)|DictionaryDecompounderTokenFilter|Descompone las palabras compuestas que se encuentran en muchas lenguas germánicas.<br /><br /> **Opciones**<br /><br /> wordList (tipo: matriz de cadenas): la lista de palabras que se van a comparar. El valor predeterminado es una lista vacía. Necesario.<br /><br /> minWordSize (tipo: entero): solo se procesan las palabras más largas que este valor. El valor predeterminado es 5.<br /><br /> minSubwordSize (tipo: entero): solo se obtienen las subpalabras más largas que este valor. El valor predeterminado es 2.<br /><br /> maxSubwordSize (tipo: entero): solo se obtienen las subpalabras más cortas que este valor. El valor predeterminado es 15.<br /><br /> onlyLongestMatch (tipo: booleano): agregue solamente la subpalabra coincidente más larga a la salida. El valor predeterminado es false.|  
|[edgeNGram_v2](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/EdgeNGramTokenFilter.html)|EdgeNGramTokenFilterV2|Genera n-gramas de los tamaños dados a partir de la parte delantera o trasera de un token de entrada.<br /><br /> **Opciones**<br /><br /> minGram (tipo: entero) - Valor predeterminado: 1, valor máximo: 300.<br /><br /> maxGram (tipo: entero) - Valor predeterminado: 2, valor máximo: 300. El valor debe ser mayor que minGram.<br /><br /> side (tipo: cadena): especifica de qué lado de la entrada se debe generar el n-grama. Valores permitidos: "front" y "back" |  
|[elision](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/util/ElisionFilter.html)|ElisionTokenFilter|Quita las elisiones. Por ejemplo, "l'avion" (el avión) se convierte en "avion" (avión).<br /><br /> **Opciones**<br /><br /> articles (tipo: matriz de cadenas): un conjunto de artículos para quitar. El valor predeterminado es una lista vacía. Si no hay ninguna lista de artículos establecida, se quitan todos los artículos en francés de forma predeterminada.|  
|[german_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/de/GermanNormalizationFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Normaliza los caracteres de alemán según la heurística del [algoritmo de bola de nieve German2](https://snowballstem.org/algorithms/german2/stemmer.html) .|  
|[hindi_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/hi/HindiNormalizationFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Normaliza el texto en hindi para quitar algunas diferencias en las variaciones ortográficas. |  
|[indic_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/in/IndicNormalizationFilter.html)|IndicNormalizationTokenFilter|Normaliza la representación Unicode de texto en las lenguas hindúes.
|[keep](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeepWordFilter.html)|KeepTokenFilter|Filtro de token que solo mantiene los tokens con el texto contenido en la lista especificada de palabras.<br /><br /> **Opciones**<br /><br /> keepWords (tipo: matriz de cadenas): una lista de palabras para mantener. El valor predeterminado es una lista vacía. Necesario.<br /><br /> keepWordsCase (tipo: bool): si es true, primero se ponen en minúsculas todas las palabras. El valor predeterminado es false.|  
|[keyword_marker](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeywordMarkerFilter.html)|KeywordMarkerTokenFilter|Marca los términos como palabras clave.<br /><br /> **Opciones**<br /><br /> keywords (tipo: matriz de cadenas): una lista de palabras para marcar como palabras clave. El valor predeterminado es una lista vacía. Necesario.<br /><br /> ignoreCase (tipo: booleano): si es true, primero se ponen en minúsculas todas las palabras. El valor predeterminado es false.|  
|[keyword_repeat](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/KeywordRepeatFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Emite cada token de entrada dos veces, una vez como palabra clave y otra como no palabra clave. |  
|[kstem](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/en/KStemFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Un filtro kstem de alto rendimiento para inglés. |  
|[length](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/LengthFilter.html)|LengthTokenFilter|Quita las palabras que son demasiado largas o demasiado cortas.<br /><br /> **Opciones**<br /><br /> min (tipo: entero): el número mínimo. Valor predeterminado: 0, valor máximo: 300.<br /><br /> max (tipo: entero): el número máximo. Valor predeterminado: 300, valor máximo: 300.|  
|[limit](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/LimitTokenCountFilter.html)|Microsoft.Azure.Search.LimitTokenFilter|Limita el número de tokens durante la indexación.<br /><br /> **Opciones**<br /><br /> maxTokenCount (tipo: int): número máximo de tokens para producir. El valor predeterminado es 1.<br /><br /> consumeAllTokens (tipo: booleano): indica si se deben consumir todos los tokens de la entrada incluso si se alcanza maxTokenCount. El valor predeterminado es false.|  
|[lowercase](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/LowerCaseFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Normaliza el texto de token a minúsculas. |  
|[nGram_v2](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/NGramTokenFilter.html)|NGramTokenFilterV2|Genera n-gramas de los tamaños dados.<br /><br /> **Opciones**<br /><br /> minGram (tipo: entero) - Valor predeterminado: 1, valor máximo: 300.<br /><br /> maxGram (tipo: entero) - Valor predeterminado: 2, valor máximo: 300. El valor debe ser mayor que minGram.|  
|[pattern_capture](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternCaptureGroupTokenFilter.html)|PatternCaptureTokenFilter|Usa expresiones regulares de Java para emitir varios tokens, uno para cada grupo de capturas en uno o varios patrones.<br /><br /> **Opciones**<br /><br /> patterns (tipo: matriz de cadenas): una lista de patrones para comparar con cada token. Necesario.<br /><br /> preserveOriginal (tipo: booleano): establézcalo en true para devolver el token original incluso si uno de los patrones coincide; valor predeterminado: true. |  
|[pattern_replace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/pattern/PatternReplaceFilter.html)|PatternReplaceTokenFilter|Un filtro de token que aplica un patrón a cada token de la secuencia, reemplazando las repeticiones de la coincidencia por la cadena de reemplazo especificada.<br /><br /> **Opciones**<br /><br /> pattern (tipo: cadena): necesario.<br /><br /> replacement (tipo: cadena): necesario.|  
|[persian_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/fa/PersianNormalizationFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles) |Aplica la normalización para persa. |  
|[phonetic](https://lucene.apache.org/core/6_6_1/analyzers-phonetic/org/apache/lucene/analysis/phonetic/package-tree.html)|PhoneticTokenFilter|Crea tokens para coincidencias fonéticas.<br /><br /> **Opciones**<br /><br /> encoder (tipo: cadena): codificador fonético que se va a usar. Los valores permitidos son: "metaphone", "doubleMetaphone", "soundex", "refinedSoundex", "caverphone1", "caverphone2", "cologne", "nysiis", "koelnerPhonetik", "haasePhonetik" y "beiderMorse". Valor predeterminado: "metaphone". El valor predeterminado es metaphone.<br /><br /> Vea el [codificador](https://commons.apache.org/proper/commons-codec/archives/1.10/apidocs/org/apache/commons/codec/language/package-frame.html) para más información.<br /><br /> replace (tipo: bool): true si los tokens codificados deben reemplazar a los tokens originales; false si se deben agregar como sinónimos. El valor predeterminado es true.|  
|[porter_stem](https://lucene.apache.org/core/6_6_1/analyzers-common/org/tartarus/snowball/ext/PorterStemmer.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Transforma la secuencia de tokens según el [algoritmo de lematización de Porter](https://tartarus.org/~martin/PorterStemmer/). |  
|[reverse](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/reverse/ReverseStringFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Invierte la cadena de token. |  
|[scandinavian_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianNormalizationFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Normaliza el uso de los caracteres de escandinavo intercambiables. |  
|[scandinavian_folding](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/ScandinavianFoldingFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Pliega los caracteres de escandinavo åÅäæÄÆ->a y öÖøØ->o. También discrimina el uso de las vocales dobles aa, ae, ao, oe y oo, dejando solo la primera de ellas. |  
|[shingle](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/shingle/ShingleFilter.html)|ShingleTokenFilter|Crea combinaciones de tokens como un solo token.<br /><br /> **Opciones**<br /><br /> maxShingleSize (tipo: entero): el valor predeterminado es 2.<br /><br /> minShingleSize (tipo: entero): el valor predeterminado es 2.<br /><br /> outputUnigrams (tipo: booleano): si es true, la secuencia de salida contiene los tokens de entrada (unigramas), así como n-gramas de palabras. El valor predeterminado es true.<br /><br /> outputUnigramsIfNoShingles (tipo: booleano): si es true, se invalida el comportamiento de outputUnigrams==false para aquellas ocasiones en las que no hay n-gramas de palabras. El valor predeterminado es false.<br /><br /> tokenSeparator (tipo: cadena): la cadena que se usará al unir los tokens adyacentes para formar un n-grama de palabras. El valor predeterminados es " ".<br /><br /> filterToken (tipo: cadena): la cadena que se va a insertar para cada posición en la que no hay ningún token. El valor predeterminado es "_".|  
|[snowball](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/snowball/SnowballFilter.html)|SnowballTokenFilter|Filtro de token de bola de nieve.<br /><br /> **Opciones**<br /><br /> language (tipo: cadena): los valores permitidos son: "armenian", "basque", "catalan", "danish", "dutch", "english", "finnish", "french", "german", "german2", "hungarian", "italian", "kp", "lovins", "norwegian", "porter", "portuguese", "romanian", "russian", "spanish", "swedish" y "turkish"|  
|[sorani_normalization](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ckb/SoraniNormalizationFilter.html)|SoraniNormalizationTokenFilter|Normaliza la representación de Unicode de texto del idioma sorani.<br /><br /> **Opciones**<br /><br /> Ninguno.|  
|stemmer|StemmerTokenFilter|Filtro de lematización específico del idioma.<br /><br /> **Opciones**<br /><br /> language (tipo: cadena): los valores permitidos son: <br /> -   ["arabic"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ar/ArabicStemmer.html)<br />-   ["armenian"](https://snowballstem.org/algorithms/armenian/stemmer.html)<br />-   ["basque"](https://snowballstem.org/algorithms/basque/stemmer.html)<br />-   ["brazilian"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/br/BrazilianStemmer.html)<br />-   "bulgarian"<br />-   ["catalan"](https://snowballstem.org/algorithms/catalan/stemmer.html)<br />-   ["czech"](https://portal.acm.org/citation.cfm?id=1598600)<br />-   ["danish"](https://snowballstem.org/algorithms/danish/stemmer.html)<br />-   ["dutch"](https://snowballstem.org/algorithms/dutch/stemmer.html)<br />-   ["dutchKp"](https://snowballstem.org/algorithms/kraaij_pohlmann/stemmer.html)<br />-   ["english"](https://snowballstem.org/algorithms/porter/stemmer.html)<br />-   ["lightEnglish"](https://ciir.cs.umass.edu/pubfiles/ir-35.pdf)<br />-   ["minimalEnglish"](https://www.researchgate.net/publication/220433848_How_effective_is_suffixing)<br />-   ["possessiveEnglish"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/en/EnglishPossessiveFilter.html)<br />-   ["porter2"](https://snowballstem.org/algorithms/english/stemmer.html)<br />-   ["lovins"](https://snowballstem.org/algorithms/lovins/stemmer.html)<br />-   ["finnish"](https://snowballstem.org/algorithms/finnish/stemmer.html)<br />-   "lightFinnish"<br />-   ["french"](https://snowballstem.org/algorithms/french/stemmer.html)<br />-   ["lightFrench"](https://dl.acm.org/citation.cfm?id=1141523)<br />-   ["minimalFrench"](https://dl.acm.org/citation.cfm?id=318984)<br />-   "galician"<br />-   "minimalGalician"<br />-   ["german"](https://snowballstem.org/algorithms/german/stemmer.html)<br />-   ["german2"](https://snowballstem.org/algorithms/german2/stemmer.html)<br />-   ["lightGerman"](https://dl.acm.org/citation.cfm?id=1141523)<br />-   "minimalGerman"<br />-   ["greek"](https://sais.se/mthprize/2007/ntais2007.pdf)<br />-   "hindi"<br />-   ["hungarian"](https://snowballstem.org/algorithms/hungarian/stemmer.html)<br />-   ["lightHungarian"](https://dl.acm.org/citation.cfm?id=1141523&dl=ACM&coll=DL&CFID=179095584&CFTOKEN=80067181)<br />-   ["indonesian"](https://www.illc.uva.nl/Publications/ResearchReports/MoL-2003-02.text.pdf)<br />-   ["irish"](https://snowballstem.org/otherapps/oregan/)<br />-   ["italian"](https://snowballstem.org/algorithms/italian/stemmer.html)<br />-   ["lightItalian"](https://www.ercim.eu/publication/ws-proceedings/CLEF2/savoy.pdf)<br />-   ["sorani"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ckb/SoraniStemmer.html)<br />-   ["latvian"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/lv/LatvianStemmer.html)<br />-   ["norwegian"](https://snowballstem.org/algorithms/norwegian/stemmer.html)<br />-   ["lightNorwegian"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianLightStemmer.html)<br />-   ["minimalNorwegian"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianMinimalStemmer.html)<br />-   ["lightNynorsk"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianLightStemmer.html)<br />-   ["minimalNynorsk"](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/no/NorwegianMinimalStemmer.html)<br />-   ["portuguese"](https://snowballstem.org/algorithms/portuguese/stemmer.html)<br />-   ["lightPortuguese"](https://dl.acm.org/citation.cfm?id=1141523&dl=ACM&coll=DL&CFID=179095584&CFTOKEN=80067181)<br />-   ["minimalPortuguese"](https://www.inf.ufrgs.br/~buriol/papers/Orengo_CLEF07.pdf)<br />-   ["portugueseRslp"](https://www.inf.ufrgs.br//~viviane/rslp/index.htm)<br />-   ["romanian"](https://snowballstem.org/otherapps/romanian/)<br />-   ["russian"](https://snowballstem.org/algorithms/russian/stemmer.html)<br />-   ["lightRussian"](https://doc.rero.ch/lm.php?url=1000%2C43%2C4%2C20091209094227-CA%2FDolamic_Ljiljana_-_Indexing_and_Searching_Strategies_for_the_Russian_20091209.pdf)<br />-   ["spanish"](https://snowballstem.org/algorithms/spanish/stemmer.html)<br />-   ["lightSpanish"](https://www.ercim.eu/publication/ws-proceedings/CLEF2/savoy.pdf)<br />-   ["swedish"](https://snowballstem.org/algorithms/swedish/stemmer.html)<br />-   "lightSwedish"<br />-   ["turkish"](https://snowballstem.org/algorithms/turkish/stemmer.html)|  
|[stemmer_override](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/StemmerOverrideFilter.html)|StemmerOverrideTokenFilter|Los términos lematizados según el diccionario se marcan como palabras clave, lo que evita la lematización de la cadena. Se debe colocar antes de los filtros de lematización.<br /><br /> **Opciones**<br /><br /> rules (tipo: matriz de cadenas): reglas de lematización con el formato "palabra => lema" por ejemplo "ejecutado = > ejecutar". El valor predeterminado es una lista vacía.  Necesario.|  
|[stopwords](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/StopFilter.html)|StopwordsTokenFilter|Quita las palabras irrelevantes de una secuencia de tokens. De forma predeterminada, el filtro utiliza una lista de palabras irrelevantes predefinidas para inglés.<br /><br /> **Opciones**<br /><br /> stopwords (tipo: matriz de cadenas): una lista de palabras irrelevantes. No se puede especificar si se especifica stopwordsList.<br /><br /> stopwordsList (tipo: cadena): una lista predefinida de palabras irrelevantes. No se puede especificar si se especifica stopwords. Los valores permitidos son: "arabic", "armenian", "basque", "brazilian", "bulgarian", "catalan", "czech", "danish", "dutch", "english", "finnish", "french", "galician", "german", "greek", "hindi", "hungarian", "indonesian", "irish", "italian", "latvian", "norwegian", "persian", "portuguese", "romanian", "russian", "sorani", "spanish", "swedish", "thai" y "turkish"; valor predeterminado: "english". No se puede especificar si se especifica stopwords. <br /><br /> ignoreCase (tipo: booleano): si es true, primero se ponen en minúsculas todas las palabras. El valor predeterminado es false.<br /><br /> removeTrailing (tipo: booleano): si es true, se omite el último término de búsqueda si es una palabra irrelevante. El valor predeterminado es true.
|[synonym](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/synonym/SynonymFilter.html)|SynonymTokenFilter|Hace coincidir sinónimos de una o varias palabras en una secuencia de tokens.<br /><br /> **Opciones**<br /><br /> synonyms (tipo: matriz de cadenas): requerido. Lista de sinónimos en uno de los dos formatos siguientes:<br /><br /> -increíble, asombroso, fabuloso => impresionante: todos los términos que se encuentran en el lado izquierdo del símbolo => se reemplazan por todos los términos que se encentran en su lado derecho.<br /><br /> -increíble, asombroso, fabuloso, impresionante: una lista de palabras equivalentes separadas por comas. Establezca la opción de expansión para cambiar cómo se interpreta esta lista.<br /><br /> ignoreCase (tipo: booleano): entradas de pliegues de mayúsculas y minúsculas para coincidencia. El valor predeterminado es false.<br /><br /> expand (tipo: bool): si es true, todas las palabras de la lista de sinónimos (si no se usa la notación =>) se asignan entre sí. <br />La lista “increíble, asombroso, fabuloso, impresionante” equivale a “increíble, asombroso, fabuloso, impresionante => increíble, asombroso, fabuloso, impresionante”.<br /><br />- Si es false, la lista “increíble, asombroso, fabuloso, impresionante” equivale a “increíble, asombroso, fabuloso, impresionante => increíble”.|  
|[trim](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/TrimFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Recorta el espacio en blanco inicial y final de los tokens. |  
|[truncate](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/TruncateTokenFilter.html)|TruncateTokenFilter|Trunca los términos en una longitud específica.<br /><br /> **Opciones**<br /><br /> length (tipo: int): valor predeterminado: 300, valor máximo: 300. Necesario.|  
|[unique](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/RemoveDuplicatesTokenFilter.html)|UniqueTokenFilter|Filtra los tokens con el mismo texto que el token anterior.<br /><br /> **Opciones**<br /><br /> onlyOnSamePosition (tipo: booleano): si se establece, quita duplicados solo en la misma posición. El valor predeterminado es true.|  
|[uppercase](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/UpperCaseFilter.html)|(el tipo solo se aplica cuando las opciones están disponibles)  |Normaliza el texto de token a mayúsculas. |  
|[word_delimiter](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/miscellaneous/WordDelimiterFilter.html)|WordDelimiterTokenFilter|Divide palabras en subpalabras y realiza transformaciones opcionales en los grupos de subpalabras.<br /><br /> **Opciones**<br /><br /> generateWordParts (tipo: booleano): crea las partes de las palabras que se van a generar, por ejemplo, "AzureSearch" se convierte en "Azure" "Search". El valor predeterminado es true.<br /><br /> generateNumberParts (tipo: bool): crea el número de subpalabras que se van a generar. El valor predeterminado es true.<br /><br /> catenateWords (tipo: booleano): crea las ejecuciones máximas de partes de palabras que se van a concatenar, por ejemplo, "Azure-Search" se convierte en "AzureSearch". El valor predeterminado es false.<br /><br /> catenateNumbers (tipo: booleano): crea las ejecuciones máximas de partes numéricas que se van a concatenar, por ejemplo, "1-2" se convierte en "12". El valor predeterminado es false.<br /><br /> catenateAll (tipo: bool): crea todas las partes de subpalabras que se van a concatenar, por ejemplo, "Azure-Search-1" se convierte en "AzureSearch1". El valor predeterminado es false.<br /><br /> splitOnCaseChange (tipo: bool): si es true, divide las palabras en caseChange, por ejemplo, "AzureSearch" se convierte en "Azure" "Search". El valor predeterminado es true.<br /><br /> preserveOriginal: crea palabras originales que se van a conservar y a agregar a la lista de subpalabras. El valor predeterminado es false.<br /><br /> splitOnNumerics (tipo: booleano): si es true, divide por los números, por ejemplo, "Azure1Search" se convierte en "Azure" "1" "Search". El valor predeterminado es true.<br /><br /> stemEnglishPossessive (tipo: booleano): hace que las "'s" finales se quiten de cada subpalabra. El valor predeterminado es true.<br /><br /> protectedWords (tipo: matriz de cadenas): tokens para impedir la delimitación. El valor predeterminado es una lista vacía.|  

 <sup>1</sup> Los tipos de filtro de token siempre tienen el prefijo "#Microsoft.Azure.Search" en el código, de forma que "ArabicNormalizationTokenFilter" realmente se especificaría como "#Microsoft.Azure.Search.ArabicNormalizationTokenFilter".  Hemos quitado el prefijo para reducir el ancho de la tabla, pero recuerde que debe incluirlo en el código.  


## <a name="see-also"></a>Consulte también  
 [API REST de Azure Cognitive Search](/rest/api/searchservice/)   
 [Analizadores de Azure Cognitive Search > ejemplos](search-analyzers.md#examples)    
 [Creación de un índice &#40;API REST de Azure Cognitive Search&#41;](/rest/api/searchservice/create-index)