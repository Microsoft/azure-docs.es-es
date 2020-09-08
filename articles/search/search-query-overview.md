---
title: Tipos de consulta y composición
titleSuffix: Azure Cognitive Search
description: Conceptos básicos para la creación de una consulta de búsqueda en Azure Cognitive Search, mediante parámetros para filtrar, seleccionar y ordenar resultados.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 06/22/2020
ms.openlocfilehash: 018c3fb08c7fa0ad35fa567bffbeae48b6fbbce9
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88928843"
---
# <a name="query-types-and-composition-in-azure-cognitive-search"></a>Tipos y composición de consultas en Azure Cognitive Search

En Azure Cognitive Search, una consulta es una especificación completa de una operación de ida y vuelta. En la solicitud, hay parámetros que proporcionan instrucciones de ejecución para el motor, así como parámetros que conforman la devolución de la respuesta. Se ejecuta una consulta sin especificar (`search=*`), sin criterios de coincidencia y mediante el uso de parámetros nulos o predeterminados, en todos los campos que permiten búsqueda como una operación de búsqueda de texto completo, que devuelve un conjunto de resultados sin puntuar en orden aleatorio.

El ejemplo siguiente es una consulta representativa creada en la [API REST](/rest/api/searchservice/search-documents). Este ejemplo tiene como destino el [índice de demostración de hoteles](search-get-started-portal.md) e incluye parámetros comunes para que pueda hacerse una idea del aspecto de una consulta.

```
{
    "queryType": "simple" 
    "search": "+New York +restaurant",
    "searchFields": "Description, Address/City, Tags",
    "select": "HotelId, HotelName, Description, Rating, Address/City, Tags",
    "top": "10",
    "count": "true",
    "orderby": "Rating desc"
}
```

+ **`queryType`** establece el analizador, que es el [analizador de consultas sencillas predeterminado](search-query-simple-examples.md) (óptimo para la búsqueda de texto completo) o el [analizador de consultas completas de Lucene](search-query-lucene-examples.md) usado en construcciones de consultas avanzadas como las expresiones regulares, la búsqueda de proximidad, la búsqueda aproximada y por caracteres comodín, por nombrar algunas.

+ **`search`** proporciona los criterios de coincidencia, normalmente términos completos o frases, pero a menudo acompañado por operadores booleanos. Los términos individuales y sencillos constituyen las consultas por *término*. Las consultas entrecomilladas de varias partes son las consultas de *frases*. La búsqueda puede ser indefinida, como en **`search=*`** , pero sin ningún criterio para coincidir, el conjunto de resultados se compone de documentos seleccionados de forma arbitraria.

+ **`searchFields`** restringe la ejecución de consultas a campos específicos. Cualquier campo que tenga el atributo *que permite búsquedas* en el esquema de índice es un candidato para este parámetro.

Las respuestas también están formadas por los parámetros que se incluyan en la consulta.

+ **`select`** especifica los campos que se van a devolver en la respuesta. Solo los campos marcados como *recuperable* en el índice se pueden usar en una instrucción select.

+ **`top`** devuelve el número especificado de documentos con la mejor coincidencia. En este ejemplo, solo se devuelven 10 resultados. Puede usar top y skip (no se muestra) para paginar los resultados.

+ **`count`** indica cuántos documentos en el índice completo coinciden globalmente, lo que puede ser mayor de lo que se devuelve. 

+ **`orderby`** se utiliza si desea ordenar los resultados por un valor, como una clasificación o una ubicación. De lo contrario, el valor predeterminado es usar la puntuación de relevancia para clasificar los resultados.

En Azure Cognitive Search, la ejecución de la consulta se realiza siempre en un índice, que se autentica mediante una clave de API proporcionada en la solicitud. En REST, ambos se proporcionan en los encabezados de solicitud.

### <a name="how-to-run-this-query"></a>Ejecución de esta consulta

Antes de escribir código, puede usar las herramientas de consulta para conocer la sintaxis y experimentar con distintos parámetros. El enfoque más rápido es la herramienta de portal integrada, el [explorador de búsqueda](search-explorer.md).

Si ha seguido este [inicio rápido para crear el índice de demostración de hoteles](search-get-started-portal.md), puede pegar esta cadena de consulta en la barra de búsqueda del explorador para ejecutar la primera consulta: `search=+"New York" +restaurant&searchFields=Description, Address/City, Tags&$select=HotelId, HotelName, Description, Rating, Address/City, Tags&$top=10&$orderby=Rating desc&$count=true`.

## <a name="how-query-operations-are-enabled-by-the-index"></a>Habilitación de las operaciones de consulta según el índice

El diseño del índice y de la consulta están estrechamente unidos en Azure Cognitive Search. Es crucial saber de antemano que el *esquema de índice*, con los atributos en cada campo, determina el tipo de consulta que se puede compilar. 

Los atributos de índice de un campo establecen las operaciones permitidas: si es un campo *que permite búsquedas* en el índice, *recuperable* en los resultados, *ordenable*, *filtrable*,etc. En la cadena de consulta de ejemplo, `"$orderby": "Rating"` solo funciona porque el campo Clasificación está marcado como *ordenable* en el esquema de índice. 

![Definición del índice para el ejemplo del hotel](./media/search-query-overview/hotel-sample-index-definition.png "Definición del índice para el ejemplo del hotel")

La captura de pantalla anterior es una lista parcial de atributos de índice para el ejemplo de hoteles. En el portal puede ver el esquema de índice completo. Para más información acerca de los atributos de índice, consulte el artículo sobre la [creación de índices con la API REST](/rest/api/searchservice/create-index).

> [!Note]
> Algunas funcionalidades de consulta están habilitadas para todo el índice en lugar de por campo. Estas son algunas de las que se incluyen: [asignaciones de sinónimos](search-synonyms.md), [analizadores personalizados](index-add-custom-analyzers.md), [construcciones del proveedor de sugerencias (para Autocompletar y consultas sugeridas)](index-add-suggesters.md), [puntuación lógica para la clasificación de resultados](index-add-scoring-profiles.md).

## <a name="elements-of-a-query-request"></a>Elementos de una solicitud de consulta

Las consultas siempre se dirigen a un solo índice. No es posible combinar índices ni crear estructuras de datos personalizadas o temporales como destino de la consulta. 

Entre los elementos obligatorios en una solicitud de consulta se incluyen los siguientes componentes:

+ Punto de conexión de servicio y colección de los documentos del índice, expresado como dirección URL con los componentes fijos y los definidos por el usuario: **`https://<your-service-name>.search.windows.net/indexes/<your-index-name>/docs`**
+ Se necesita **`api-version`** (solo REST), ya que siempre hay más de una versión de la API disponible. 
+ **`api-key`** , clave de API de consulta o administrador que autentique la solicitud en el servicio.
+ **`queryType`** , simplificada o completa, que se puede omitir si usa la sintaxis simplificada predeterminada integrada.
+ **`search`** o **`filter`** proporcionan los criterios de coincidencia, pueden dejarse sin especificar si se desea realizar una búsqueda vacía. Ambos tipos de consulta se describen en términos del analizador sencillo, pero incluso las consultas avanzadas requieren el parámetro de búsqueda para pasar las expresiones de consulta complejas.

Todos los demás parámetros de búsqueda son opcionales. Para la lista completa de atributos, consulte el artículo de [creación de índices (REST)](/rest/api/searchservice/create-index). Para una visión más detallada de cómo se usan los parámetros durante el procesamiento, consulte [Cómo funciona la búsqueda de texto completo en Azure Cognitive Search](search-lucene-query-architecture.md).

## <a name="choose-apis-and-tools"></a>Selección de API y herramientas

En la tabla siguiente se enumeran las API y los métodos basados en herramientas para el envío de consultas.

| Metodología | Descripción |
|-------------|-------------|
| [Explorador de búsqueda (portal)](search-explorer.md) | Proporciona una barra de búsqueda y opciones para la selección del índice y la versión de API. Los resultados se devuelven como documentos JSON. Recomendado para exploración, prueba y validación. <br/>[Más información.](search-get-started-portal.md#query-index) | 
| [Postman u otras herramientas de REST](search-get-started-postman.md) | Las herramientas de pruebas Web son una opción excelente para formular llamadas REST. La API REST es compatible con todas las operaciones posibles de Azure Cognitive Search. En este artículo, se explica cómo configurar el encabezado y el cuerpo de una solicitud HTTP para enviar solicitudes a Azure Cognitive Search.  |
| [SearchIndexClient (.NET)](/dotnet/api/microsoft.azure.search.searchindexclient?view=azure-dotnet) | Cliente que puede usarse para consultar un índice de Azure Cognitive Search.  <br/>[Más información.](search-howto-dotnet-sdk.md#core-scenarios)  |
| [Documentos de búsqueda (API REST)](/rest/api/searchservice/search-documents) | Métodos GET o POST en un índice, con parámetros de consulta para entradas adicionales.  |

## <a name="choose-a-parser-simple--full"></a>Elección del analizador: básico | completo

Azure Cognitive Search se basa en Apache Lucene y le permite elegir entre dos analizadores de consulta para gestionar consultas típicas y especializadas. Las solicitudes se formulan mediante el analizador básico con [sintaxis de consulta simplificada](query-simple-syntax.md), seleccionada como predeterminada por su velocidad y eficacia para las consultas de texto libre. Esta sintaxis admite varios operadores de búsqueda comunes entre los que se incluyen los operadores de frase, sufijo, precedencia, AND, OR y NOT.

La [sintaxis de consulta completa de Lucene](query-Lucene-syntax.md#bkmk_syntax), que se habilita al agregar `queryType=full` a la solicitud, permite usar el expresivo lenguaje de consulta adoptado ampliamente desarrollado en [Apache Lucene](https://lucene.apache.org/core/6_6_1/queryparser/org/apache/lucene/queryparser/classic/package-summary.html). La sintaxis completa amplía la sintaxis simplificada. Cualquier consulta que escriba con sintaxis simplificada se ejecuta en el analizador de Lucene completo. 

Los ejemplos siguientes ilustran el punto: misma consulta, pero con valores de queryType distintos, producen resultados diferentes. En la primera consulta, `^3` después de `historic` se trata como parte del término de búsqueda. El resultado clasificado en primer lugar en esta consulta es "Marquis Plaza & Suites", que tiene *ocean* en su descripción.

```
queryType=simple&search=ocean historic^3&searchFields=Description, Tags&$select=HotelId, HotelName, Tags, Description&$count=true
```

La misma consulta con el analizador de Lucene completo interpreta `^3` como impulsor de términos en el campo. Los analizadores de conmutación cambian la clasificación, de forma que los resultados que contienen el término *historic* se mueven al principio.

```
queryType=full&search=ocean historic^3&searchFields=Description, Tags&$select=HotelId, HotelName, Tags, Description&$count=true
```

<a name="types-of-queries"></a>

## <a name="types-of-queries"></a>Tipos de consultas

Azure Cognitive Search admite una amplia gama de tipos de consulta. 

| Tipo de consulta | Uso | Más información y ejemplos |
|------------|--------|-------------------------------|
| Búsqueda de texto libre | Parámetro de búsqueda y cualquier analizador| La consulta de texto completo busca uno o más términos en todos los campos *que permiten búsquedas* del índice y funciona tal y como cabría esperar de un motor de búsqueda como Google o Bing. El ejemplo de la introducción es una búsqueda de texto completo.<br/><br/>La búsqueda de texto completo se somete a análisis léxico con el analizador de Lucene estándar (predeterminado) para poner todos los términos en minúscula y eliminar palabras vacías como "the". Puede invalidar el valor predeterminado con [analizadores de idiomas distintos al inglés](index-add-language-analyzers.md#language-analyzer-list) o [analizadores independientes del idioma especializados](index-add-custom-analyzers.md#AnalyzerTable) que modifiquen el análisis léxico. Un ejemplo es el analizador por [palabra clave](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html), que trata el contenido de un campo como token único. Esto es útil para los datos como códigos postales, identificadores y algunos nombres de producto. | 
| Búsqueda filtrada | [Expresión de filtro de OData](query-odata-filter-orderby-syntax.md) y cualquier analizador | Las consultas de filtro evalúan una expresión booleana en todos los campos *filtrables* de un índice. A diferencia de la búsqueda, una consulta de filtro busca el contenido exacto de un campo, incluidas las mayúsculas y minúsculas de los campos de cadena. Otra diferencia es que las consultas de filtro se expresan con sintaxis de OData. <br/>[Ejemplo de expresión de filtro](search-query-simple-examples.md#example-3-filter-queries) |
| Búsqueda georreferenciada | [Tipo Edm.GeographyPoint](/rest/api/searchservice/supported-data-types) en el campo, la expresión de filtro y cualquier analizador | Las coordenadas que se almacenan en un campo con Edm.GeographyPoint se utilizan para "buscar cerca de mí" o los controles de búsqueda con mapas. <br/>[Ejemplo de búsqueda georreferenciada](search-query-simple-examples.md#example-5-geo-search)|
| Búsqueda de intervalo | Expresión de filtro y analizador básico | En Azure Cognitive Search, las consultas de intervalo se compilan con el parámetro de filtro. <br/>[Ejemplo de filtro de intervalo](search-query-simple-examples.md#example-4-range-filters) | 
| [Búsqueda clasificada por campos](query-lucene-syntax.md#bkmk_fields) | Parámetro de búsqueda y analizador completo | Compila una expresión de consulta compuesta con un único campo como destino. <br/>[Ejemplo de búsqueda clasificada por campos](search-query-lucene-examples.md#example-2-fielded-search) |
| [Búsqueda aproximada](query-lucene-syntax.md#bkmk_fuzzy) | Parámetro de búsqueda y analizador completo | Coincidencias en términos con construcción u ortografía similares. <br/>[Ejemplo de búsqueda aproximada](search-query-lucene-examples.md#example-3-fuzzy-search) |
| [Búsqueda por proximidad](query-lucene-syntax.md#bkmk_proximity) | Parámetro de búsqueda y analizador completo | Busca términos que están cerca uno del otro en un documento. <br/>[Ejemplo de búsqueda por proximidad](search-query-lucene-examples.md#example-4-proximity-search) |
| [Priorización de términos](query-lucene-syntax.md#bkmk_termboost) | Parámetro de búsqueda y analizador completo | Clasifica un documento superior si contiene el término prioritario con respecto a otros que no lo tienen. <br/>[Ejemplo de priorización de términos](search-query-lucene-examples.md#example-5-term-boosting) |
| [Búsqueda de expresiones regulares](query-lucene-syntax.md#bkmk_regex) | Parámetro de búsqueda y analizador completo | Coincidencias basadas en el contenido de una expresión regular. <br/>[Ejemplo de expresión regular](search-query-lucene-examples.md#example-6-regex) |
|  [búsqueda de prefijo o de carácter comodín](query-lucene-syntax.md#bkmk_wildcard) | Parámetro de búsqueda y analizador completo | Coincidencias según un prefijo y una virgulilla (`~`) o carácter individual (`?`). <br/>[Ejemplo de búsqueda con caracteres comodín](search-query-lucene-examples.md#example-7-wildcard-search) |

## <a name="manage-search-results"></a>Administración de los resultados de búsqueda 

Los resultados de la consulta se transmiten como documentos JSON en la API REST, aunque si usa API. NET, la serialización está integrada. Puede moldear los resultados mediante la definición de parámetros en la consulta, seleccionando campos específicos para la respuesta.

Los parámetros de la consulta se pueden usar para estructurar el conjunto de resultados de las siguientes maneras:

+ Limitar o procesar por lotes el número de documentos de los resultados (50 de forma predeterminada)
+ Seleccionar campos que incluir en los resultados
+ Definir un criterio de ordenación
+ Añadir resaltado de referencias para centrar la atención sobre términos coincidentes en el cuerpo de los resultados de búsqueda

### <a name="tips-for-unexpected-results"></a>Sugerencias para resultados inesperados

En ocasiones, la sustancia, y no la estructura de los resultados, no es la esperada. Cuando los resultados de la consulta no son los esperados, puede probar estas modificaciones de consulta para ver si mejoran:

+ Cambie **`searchMode=any`** (predeterminado) a **`searchMode=all`** para requerir coincidencias en todos los criterios en lugar de en cualquiera de ellos. Esto sucede especialmente cuando se incluyen operadores booleanos en la consulta.

+ Cambie la técnica de la consulta si se necesita análisis léxico o de texto, pero el tipo de consulta impide el procesamiento lingüístico. En la búsqueda de texto completo, el análisis léxico o de texto corrige de forma automática errores de ortografía, formas de palabras con errores de concordancia de número e incluso sustantivos o verbos irregulares. En algunas consultas, como la búsqueda aproximada o la búsqueda con caracteres comodín, el análisis léxico no forma parte de la canalización de análisis de la consulta. En algunos escenarios, se han utilizado expresiones regulares como solución alternativa. 

### <a name="paging-results"></a>Paginación de resultados
Azure Cognitive Search facilita la implementación de la paginación de los resultados de búsqueda. Mediante el uso de los parámetros **`top`** y **`skip`** , puede emitir fácilmente solicitudes de búsqueda que le permitan recibir el conjunto total de resultados de búsqueda dividido en subconjuntos ordenados, sencillos de administrar que permitan habilitar fácilmente prácticas eficaces de interfaz de usuario de búsqueda. Al recibir estos subconjuntos más pequeños de resultados, también puede recibir el número de documentos del conjunto total de resultados de la búsqueda.

Para más información acerca de la paginación de resultados de búsqueda, consulte el artículo [Cómo paginar los resultados de la búsqueda en Azure Cognitive Search](search-pagination-page-layout.md).

### <a name="ordering-results"></a>Ordenación de los resultados
Al recibir los resultados de una consulta de búsqueda, puede solicitar que Azure Cognitive Search muestre los resultados ordenados según los valores de un campo específico. De forma predeterminada, Azure Cognitive Search ordena los resultados de búsqueda basándose en el rango de puntuación de búsqueda de cada documento, que se deriva del valor [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf).

Si desea que Azure Cognitive Search devuelva los resultados ordenados según un valor distinto de la puntuación de búsqueda, puede usar el parámetro **`orderby`** . Puede especificar el valor del parámetro **`orderby`** para que incluya nombres de campo y llamadas a la [ **`geo.distance()` función**](query-odata-filter-orderby-syntax.md) para los valores geoespaciales. Cada expresión puede ir seguida de `asc` para indicar que se solicitan los resultados en orden ascendente y de **`desc`** para indicar que se solicitan los resultados en orden descendente. El valor predeterminado es el orden ascendente.


### <a name="hit-highlighting"></a>Resaltado de referencias
En Azure Cognitive Search, resaltar la parte exacta de los resultados de búsqueda que coincide con la consulta de búsqueda es fácil mediante el uso de los parámetros **`highlight`** , **`highlightPreTag`** y **`highlightPostTag`** . Puede especificar qué campos *habilitados para búsquedas* deben tener su texto coincidente resaltado, así como especificar las etiquetas de cadena exactas que se anexarán al comienzo y al final del texto coincidente que devuelve Azure Cognitive Search.

## <a name="see-also"></a>Consulte también

+ [Cómo funciona la búsqueda de texto completo en Azure Cognitive Search (arquitectura de análisis de consultas)](search-lucene-query-architecture.md)
+ [Explorador de búsqueda](search-explorer.md)
+ [Consultas con .NET](./search-get-started-dotnet.md)
+ [Consultas con REST](./search-get-started-powershell.md)