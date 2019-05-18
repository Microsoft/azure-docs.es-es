---
title: 'Analizadores para procesamientos lingüísticos y textuales: Azure Search'
description: Asigne analizadores a los campos de texto que permiten búsquedas de un índice para reemplazar la API Lucene estándar personalizada por otras alternativas personalizadas, predefinidas o específicas del lenguaje.
services: search
ms.service: search
ms.topic: conceptual
ms.date: 03/27/2019
ms.author: heidist
manager: cgronlun
author: HeidiSteen
ms.custom: seodec2018
ms.openlocfilehash: f76d944f614f07a4428d4e4100f6a08a375d96dc
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65795800"
---
# <a name="analyzers-for-text-processing-in-azure-search"></a>Analizadores para procesamientos textuales en Azure Search

Un *analizador* es un componente del [motor de búsqueda de texto completo](search-lucene-query-architecture.md) responsable del procesamiento de texto en cadenas de consulta y documentos indexados. Diferentes analizadores manipulan el texto de maneras diferentes según el escenario. Los analizadores de idioma procesan el texto mediante reglas lingüísticas para mejorar la calidad de la búsqueda, mientras que otros analizadores realizan tareas más básicas como la conversión de caracteres a minúsculas, por ejemplo. 

Estos analizadores son los que se usan con más frecuencia, y hay un analizador de idioma predeterminado asignado a cada campo de búsqueda en un índice de Azure Search. Las siguientes transformaciones de lenguaje son habituales durante el análisis de texto:

+ Se quitan las palabras no esenciales (palabras irrelevantes) y los signos de puntuación.
+ Las frases y las palabras con guiones se dividen en las partes que los componen.
+ Las palabras en mayúsculas se ponen en minúsculas.
+ Las palabras se reducen a sus formas raíz, con el fin de que se puedan encontrar coincidencias independientemente del tiempo verbal.

Los analizadores de idioma convierten una entrada de texto en formularios primitivos o raíz que son eficaces para la recuperación y el almacenamiento de información. La conversión se produce durante la indexación, cuando se crea el índice y, de nuevo, durante la búsqueda cuando se lee el índice. Es más probable que obtenga los resultados de búsqueda que espera si usa el mismo analizador en ambas operaciones.

## <a name="default-analyzer"></a>Analizador predeterminado  

Azure Search usa el [analizador estándar de Apache Lucene (Lucene estándar)](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/standard/StandardAnalyzer.html) que divide el texto en elementos siguiendo las reglas de ["Segmentación de texto Unicode"](https://unicode.org/reports/tr29/). Además, el analizador estándar convierte todos los caracteres en minúsculas. Los documentos indexados y lo términos de búsqueda son sometidos a análisis durante la indexación y el procesamiento de consultas.  

Se usa automáticamente en cada campo que permite búsquedas. Puede invalidar el valor predeterminado campo por campo. Los analizadores alternativos pueden ser un [analizador de idioma](index-add-language-analyzers.md), un [analizador personalizado](index-add-custom-analyzers.md) o un analizador predefinido de la [lista de analizadores disponibles](index-add-custom-analyzers.md#AnalyzerTable).


## <a name="types-of-analyzers"></a>Tipos de analizadores

En la lista siguiente se describen los analizadores que están disponibles en Azure Search.

| Categoría | DESCRIPCIÓN |
|----------|-------------|
| [Analizador Lucene estándar](https://lucene.apache.org/core/4_0_0/analyzers-common/org/apache/lucene/analysis/standard/StandardAnalyzer.html) | Predeterminada. No se requieren ninguna especificación ni configuración. Este analizador de uso general funciona bien en la mayoría de lenguajes y escenarios.|
| Analizadores predefinidos | Se ofrecen como un producto acabado concebido para usarse tal cual. <br/>Hay dos tipos: especializados y de lenguaje. Lo que hace que sean "predefinidos" es que se hace referencia a ellos por su nombre, sin configuración ni personalización alguna. <br/><br/>Los [analizadores especializados (independientes del idioma)](index-add-custom-analyzers.md#AnalyzerTable) se usan cuando las entradas de texto requieren procesamiento especializado o un procesamiento mínimo. Los analizadores predefinidos sin lenguaje incluyen **Asciifolding**, **Keyword**, **Pattern**, **Simple**, **Stop** y **Whitespace**.<br/><br/>Los [analizadores de lenguaje](index-add-language-analyzers.md) se utilizan cuando se necesita compatibilidad lingüística enriquecida con lenguajes individuales. Azure Search admite 35 analizadores de lenguaje de Lucene y 50 analizadores de procesamiento de lenguaje natural de Microsoft. |
|[Analizadores personalizados](https://docs.microsoft.com/rest/api/searchservice/Custom-analyzers-in-Azure-Search) | Hacen referencia a una configuración definida por el usuario de una combinación de los elementos existentes, que consta de un tokenizador (obligatorio) y filtros opcionales (char o token).|

Algunos analizadores predefinidos, como **Pattern** o **Stop**, admiten un conjunto limitado de opciones de configuración. Para establecer estas opciones, debe crear eficazmente un analizador personalizado, que conste del analizador predefinido y una de las opciones alternativas documentadas en [Referencia de analizadores predefinidos](index-add-custom-analyzers.md#AnalyzerTable). Como en cualquier configuración personalizada, proporcione a la nueva configuración un nombre, como *myPatternAnalyzer* para distinguirla del analizador de Lucene Pattern.

## <a name="how-to-specify-analyzers"></a>Especificación de analizadores

1. Solo para analizadores personalizados: cree una sección de **analyzer** con nombre en la definición del índice. Para más información, consulte [Create Index](https://docs.microsoft.com/rest/api/searchservice/create-index) y también [Incorporación de analizadores personalizados](index-add-custom-analyzers.md).

2. En una [definición de campo](https://docs.microsoft.com/rest/api/searchservice/create-index) del índice, establezca la propiedad **analyzer** del campo en el nombre de un analizador de destino (por ejemplo, `"analyzer" = "keyword"`. Los valores válidos incluyen el nombre de un analizador predefinido, un analizador del lenguaje o un analizador personalizado también definido en el esquema de índice. Planee la asignación del analizador en la fase de definición del índice antes de que este se cree en el servicio.

3. Opcionalmente, en lugar de una propiedad **analyzer**, puede establecer diferentes analizadores para indexación y consulta mediante los parámetros de campo **indexAnalyzer** y **searchAnalyzer**. Si una de esas actividades requiere una transformación específica que otras no necesitan, usará diferentes analizadores para la recuperación y la preparación de los datos.

No se permite asignar **analyzer** o **indexAnalyzer** a un campo que ya se ha creado físicamente. Si alguno de estos puntos no está claro, revise la siguiente tabla para ver un desglose de las acciones necesarias en una recompilación y porqué.
 
 | Escenario | Impacto | Pasos |
 |----------|--------|-------|
 | Adición de un nuevo campo | mínimo | Si el campo no existe todavía en el esquema, no hay ninguna revisión del campo que realizar porque el campo no tiene todavía una presencia física en el índice. Puede usar [Update Index](https://docs.microsoft.com/rest/api/searchservice/update-index) para agregar un nuevo campo a un índice existente, y [mergeOrUpload](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents) para rellenarlo.|
 | Agregue un elemento **analyzer** o **indexAnalyzer** a un campo indexado existente. | [rebuild](search-howto-reindex.md) | El índice invertido para ese campo debe volver a crearse desde el principio y se debe volver a indexar el contenido de esos campos. <br/> <br/>En el caso de índices en desarrollo activo, [elimine](https://docs.microsoft.com/rest/api/searchservice/delete-index) y [cree](https://docs.microsoft.com/rest/api/searchservice/create-index) el índice para que recoja la nueva definición del campo. <br/> <br/>Para los índices de producción, puede aplazar una recompilación mediante la creación de un nuevo campo para proporcionar la definición revisada y empezar a usarla en lugar de la antigua. Use [Update Index](https://docs.microsoft.com/rest/api/searchservice/update-index) para incorporar el nuevo campo y [mergeOrUpload](https://docs.microsoft.com/rest/api/searchservice/addupdate-or-delete-documents) para rellenarlo. Más adelante, como parte de un mantenimiento planeado del índice, puede limpiarlo para quitar campos obsoletos. |

## <a name="when-to-add-analyzers"></a>Cuándo agregar analizadores

El mejor momento para agregar y asignar analizadores es durante el desarrollo activo, cuando quitar y volver a crear índices se convierte en una rutina.

A medida que una nueva definición de índice se solidifica, puede anexar nuevas construcciones de análisis a un índice, pero deberá pasar la marca **allowIndexDowntime** a [Update Index](https://docs.microsoft.com/rest/api/searchservice/update-index) si quiere evitar este error:

*"No se permite la actualización del índice porque provocaría tiempo de inactividad. Para agregar nuevos analizadores, tokenizadores, filtros de token o filtros de caracteres a un índice existente, establezca el parámetro de consulta "allowIndexDowntime" en "true" en la solicitud de actualización del índice. Tenga en cuenta que esta operación hará que el índice pase a estar sin conexión durante al menos unos segundos, de modo que las solicitudes de indexación y consulta darán error. El rendimiento y la disponibilidad de escritura del índice pueden ser desiguales durante varios minutos después de que se actualice el índice, o durante más tiempo en el caso de índices muy grandes."*

Lo mismo sucede al asignar un analizador a un campo. Un analizador es una parte integral de la definición del campo, por lo que solo puede agregarlo cuando se crea el campo. Si quiere agregar analizadores a campos existentes, tendrá que [quitar y recompilar](search-howto-reindex.md) el índice, o bien agregar un nuevo campo con el analizador deseado.

Como se ha mencionado, una excepción es la variante **searchAnalyzer**. De las tres formas de especificar analizadores (**analyzer**, **indexAnalyzer** y **searchAnalyzer**), solo el atributo **searchAnalyzer** se puede cambiar en un campo existente.

## <a name="recommendations-for-working-with-analyzers"></a>Recomendaciones para trabajar con analizadores

En esta sección se proporcionan consejos para trabajar con los analizadores.

### <a name="one-analyzer-for-read-write-unless-you-have-specific-requirements"></a>Un analizador para lectura y escritura, salvo que tenga requisitos específicos

Azure Search le permite especificar diferentes analizadores para indexación y búsqueda mediante los parámetros de campo adicionales **indexAnalyzer** y **searchAnalyzer**. Si no se especifica, el analizador establecido con la propiedad **analyzer** se usa para la indexación y la búsqueda. Si `analyzer` no se especifica, se utiliza el analizador Lucene estándar predeterminado.

Una regla general es usar el mismo analizador de para los índices y las consultas, a menos que los requisitos específicos indiquen lo contrario. Asegúrese de realizar pruebas exhaustivas. Si el procesamiento de texto es diferente en el momento de la búsqueda y la indexación, corre el riesgo de que se produzca una falta de coincidencia entre los términos de la consulta y los términos indexados si las configuraciones del analizador de búsqueda y el de indexación no están alineadas.

### <a name="test-during-active-development"></a>Prueba durante el desarrollo activo

El reemplazo del analizador estándar requiere que se reconstruya el índice. Si es posible, decida qué analizadores se van a usar durante el desarrollo activo antes de poner un índice en producción.

### <a name="inspect-tokenized-terms"></a>Inspección de términos con tokens

Si se produce un error en una búsqueda para devolver los resultados esperados, el escenario más probable es que se creen discrepancias de tokens entre las entradas de término en la consulta y términos con tokens en el índice. Si los tokens no son los mismos, las coincidencias no pueden materializarse. Para inspeccionar la salida del tokenizador recomendamos usar la [API de análisis](https://docs.microsoft.com/rest/api/searchservice/test-analyzer) como herramienta de investigación. La respuesta consta de los tokens que genera un analizador concreto.

<a name="examples"></a>

## <a name="rest-examples"></a>Ejemplos de REST

Los ejemplos siguientes muestran las definiciones del analizador en algunos escenarios claves.

+ [Ejemplo de analizador personalizado](#Custom-analyzer-example)
+ [Ejemplo de asignación de analizadores a un campo](#Per-field-analyzer-assignment-example)
+ [Mezcla de analizadores para indexación y búsqueda](#Mixing-analyzers-for-indexing-and-search-operations)
+ [Ejemplo de analizador de idioma](#Language-analyzer-example)

<a name="Custom-analyzer-example"></a>

### <a name="custom-analyzer-example"></a>Ejemplo de analizador personalizado

Este ejemplo ilustra una definición del analizador con opciones personalizadas. Las opciones personalizadas de los filtros de char, tokenizadores y filtros de token se especifican por separado como construcciones con nombre y, después, se hace referencia a ellas en la definición del analizador. Los elementos predefinidos se usan tal cual y se hace referencia a ellos por su nombre.

Descripción de este ejemplo:

* Los analizadores son una propiedad de la clase de campo de un campo que permite la búsqueda.
* Un analizador personalizado forma parte de una definición de índice. Su personalización puede ser escasa (por ejemplo, la personalización de una sola opción de un filtro) o puede estar personalizado en varios lugares.
* En este caso, el analizador personalizado es "my_analyzer", que a su vez utiliza un tokenizador estándar personalizado, "my_standard_tokenizer", y dos filtros de token: lowercase y el filtro de asciifolding personalizado "my_asciifolding".
* También define 2 filtros de char personalizados "map_dash" y "remove_whitespace". La primera de ellas reemplaza todos los guiones por caracteres de subrayado, mientras que la segunda quita todos los espacios. Los espacios tienen que tener codificación UTF-8 en las reglas de asignación. Los filtros de char se aplican antes de la tokenización y afectarán a los tokens resultantes (el tokenizador estándar se interrumpe en guiones y espacios, pero no en un carácter de subrayado).

~~~~
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text",
           "type":"Edm.String",
           "searchable":true,
           "analyzer":"my_analyzer"
        }
     ],
     "analyzers":[
        {
           "name":"my_analyzer",
           "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
           "charFilters":[
              "map_dash",
              "remove_whitespace"
           ],
           "tokenizer":"my_standard_tokenizer",
           "tokenFilters":[
              "my_asciifolding",
              "lowercase"
           ]
        }
     ],
     "charFilters":[
        {
           "name":"map_dash",
           "@odata.type":"#Microsoft.Azure.Search.MappingCharFilter",
           "mappings":["-=>_"]
        },
        {
           "name":"remove_whitespace",
           "@odata.type":"#Microsoft.Azure.Search.MappingCharFilter",
           "mappings":["\\u0020=>"]
        }
     ],
     "tokenizers":[
        {
           "name":"my_standard_tokenizer",
           "@odata.type":"#Microsoft.Azure.Search.StandardTokenizerV2",
           "maxTokenLength":20
        }
     ],
     "tokenFilters":[
        {
           "name":"my_asciifolding",
           "@odata.type":"#Microsoft.Azure.Search.AsciiFoldingTokenFilter",
           "preserveOriginal":true
        }
     ]
  }
~~~~

<a name="Per-field-analyzer-assignment-example"></a>

### <a name="per-field-analyzer-assignment-example"></a>Ejemplo de asignación de analizador por campo

El analizador estándar es el predeterminado. Imagine que desea reemplazar el analizador predeterminado por otro analizador predefinido, como el analizador de patrón. Si no va a establecer opciones personalizadas, basta con que lo especifique por nombre en la definición del campo.

El elemento "analizador" reemplaza el analizador estándar campo a campo. No se produce un reemplazo global. En este ejemplo, `text1` utiliza el analizador de patrón y `text2`, que no especifica un analizador, usa el predeterminado.

~~~~
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text1",
           "type":"Edm.String",
           "searchable":true,
           "analyzer":"pattern"
        },
        {
           "name":"text2",
           "type":"Edm.String",
           "searchable":true
        }
     ]
  }
~~~~

<a name="Mixing-analyzers-for-indexing-and-search-operations"></a>

### <a name="mixing-analyzers-for-indexing-and-search-operations"></a>Mezcla de analizadores para las operaciones de indexación y búsqueda

Las API incluyen atributos de índice adicionales para especificar diferentes analizadores para las operaciones de indexación y búsqueda. Los atributos **searchAnalyzer** y **indexAnalyzer** se deben especificar como un par, de forma que se reemplaza el atributo único **analyzer**.


~~~~
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text",
           "type":"Edm.String",
           "searchable":true,
           "indexAnalyzer":"whitespace",
           "searchAnalyzer":"simple"
        },
     ],
  }
~~~~

<a name="Language-analyzer-example"></a>

### <a name="language-analyzer-example"></a>Ejemplo de analizador de idioma

Los campos que contienen cadenas en diferentes idiomas pueden utilizar un analizador del lenguaje, mientras que otros campos conservan el predeterminado (o usan otro analizador predefinido o personalizado). Si utiliza un analizador del lenguaje, debe hacerlo tanto para las operaciones de indexación como para las de búsqueda. Los campos que usan un analizador del lenguaje no pueden tener un analizador para la indexación y otro para la búsqueda.

~~~~
  {
     "name":"myindex",
     "fields":[
        {
           "name":"id",
           "type":"Edm.String",
           "key":true,
           "searchable":false
        },
        {
           "name":"text",
           "type":"Edm.String",
           "searchable":true,
           "indexAnalyzer":"whitespace",
           "searchAnalyzer":"simple"
        },
        {
           "name":"text_fr",
           "type":"Edm.String",
           "searchable":true,
           "analyzer":"fr.lucene"
        }
     ],
  }
~~~~

## <a name="c-examples"></a>C#ejemplos

Si utiliza los ejemplos de código del SDK de. NET, puede anexar estos ejemplos para usar o configurar los analizadores.

+ [Asignar un analizador integrado](#Assign-a-language-analyzer)
+ [Configurar un analizador](#Define-a-custom-analyzer)

<a name="Assign-a-language-analyzer"></a>

### <a name="assign-a-language-analyzer"></a>Asignar un analizador de lenguaje

Cualquier analizador que se usa como-es, sin ninguna configuración, se especifica en una definición de campo. No hay ningún requisito para la creación de una construcción de analizador. 

Este ejemplo asigna a los campos de descripción analizadores de Microsoft English y francés. Es un fragmento de código que se toman de una definición del índice de hoteles, crear mediante la clase del Hotel en el archivo hotels.cs de mayor tamaño el [DotNetHowTo](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowTo) ejemplo.

Llame a [analizador](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.analyzer?view=azure-dotnet), especificando el [AnalyzerName](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.analyzername?view=azure-dotnet) tipo que proporciona un analizador de texto que se admiten en Azure Search.

```csharp
    public partial class Hotel
    {
       . . . 

        [IsSearchable]
        [Analyzer(AnalyzerName.AsString.EnMicrosoft)]
        [JsonProperty("description")]
        public string Description { get; set; }

        [IsSearchable]
        [Analyzer(AnalyzerName.AsString.FrLucene)]
        [JsonProperty("description_fr")]
        public string DescriptionFr { get; set; }

      . . .
    }
```
<a name="Define-a-custom-analyzer"></a>

### <a name="define-a-custom-analyzer"></a>Definir un analizador personalizado

Cuando se requiere la personalización o configuración, deberá agregar una construcción del analizador a un índice. Una vez que se defina, puede agregarlo la definición de campo, como se muestra en el ejemplo anterior.

Crear un [CustomAnalyzer](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.models.customanalyzer?view=azure-dotnet) objeto. Para obtener más ejemplos, vea [CustomAnalyzerTests.cs](https://github.com/Azure/azure-sdk-for-net/blob/master/src/SDKs/Search/DataPlane/Search.Tests/Tests/CustomAnalyzerTests.cs).

```csharp
{
   var definition = new Index()
   {
         Name = "hotels",
         Fields = FieldBuilder.BuildForType<Hotel>(),
         Analyzers = new[]
            {
               new CustomAnalyzer()
               {
                     Name = "url-analyze",
                     Tokenizer = TokenizerName.UaxUrlEmail,
                     TokenFilters = new[] { TokenFilterName.Lowercase }
               }
            },
   };

   serviceClient.Indexes.Create(definition);
```

## <a name="next-steps"></a>Pasos siguientes

+ Revise la explicación detallada de [Cómo funciona la búsqueda de texto completo en Azure Search](search-lucene-query-architecture.md). En este artículo se usan ejemplos que explican comportamientos que, a simple vista, podrían parecer contradictorios.

+ Probar la sintaxis de consulta adicional de la sección de ejemplo [Buscar documentos](https://docs.microsoft.com/rest/api/searchservice/search-documents#bkmk_examples) o desde [Sintaxis de consulta simple](query-simple-syntax.md) en el explorador de búsqueda en el portal.

+ Obtener información sobre cómo aplicar [analizadores léxicos específicos del idioma](index-add-language-analyzers.md).

+ [Configure analizadores personalizados](index-add-custom-analyzers.md) para un procesamiento mínimo o un procesamiento especializado en los campos individuales.

## <a name="see-also"></a>Vea también

 [API de REST de documentos de búsqueda](https://docs.microsoft.com/rest/api/searchservice/search-documents) 

 [Sintaxis de consulta simplificada](query-simple-syntax.md) 

 [Sintaxis de consulta completa de Lucene ](query-lucene-syntax.md) 
 
 [Control de los resultados de la búsqueda](search-pagination-page-layout.md)

<!--Image references-->
[1]: ./media/search-lucene-query-architecture/architecture-diagram2.png
