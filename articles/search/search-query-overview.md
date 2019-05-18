---
title: Tipos de consultas y composición en Azure Search
description: Conceptos básicos para la creación de una consulta de búsqueda en Azure Search, utilizando parámetros para filtrar, seleccionar y ordenar resultados.
author: HeidiSteen
manager: cgronlun
ms.author: heidist
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/13/2019
ms.custom: seodec2018
ms.openlocfilehash: 95f5dde12ad9e34a0a04c988a816538ac30e01e6
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65595984"
---
# <a name="how-to-compose-a-query-in-azure-search"></a>Cómo crear una consulta en Azure Search

En Azure Search, una consulta es una especificación completa de una operación de ida y vuelta. Los parámetros en la consulta proporcionan los criterios de coincidencia para buscar documentos en un índice, instrucciones de ejecución para el motor y directivas de estructuración de la respuesta. 

Una solicitud de consulta es un constructor enriquecido, que especifica los campos que pertenecen al ámbito, la búsqueda, los campos que se van a devolver, el filtrado y la ordenación, etc. Una consulta sin especificar se ejecuta en todos los campos de búsqueda como una operación de búsqueda de texto completo, devolviendo un conjunto de resultados sin puntuar en orden aleatorio.

## <a name="apis-and-tools-for-testing"></a>API y herramientas de pruebas

En la tabla siguiente se enumeran las API y los métodos basados en herramientas para el envío de consultas.

| Metodología | DESCRIPCIÓN |
|-------------|-------------|
| [Explorador de búsqueda (portal)](search-explorer.md) | Proporciona una barra de búsqueda y opciones para la selección del índice y la versión de API. Los resultados se devuelven como documentos JSON. <br/>[Más información.](search-get-started-portal.md#query-index) | 
| [Postman o Fiddler](search-fiddler.md) | Las herramientas de pruebas Web son una opción excelente para formular llamadas REST. La API REST es compatible con todas las operaciones posibles de Azure Search. En este artículo, se explica cómo configurar el encabezado y el cuerpo de una solicitud HTTP para enviar solicitudes a Azure Search.  |
| [SearchIndexClient (.NET)](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.searchindexclient?view=azure-dotnet) | Cliente que puede usarse para consultar un índice de Azure Search.  <br/>[Más información.](search-howto-dotnet-sdk.md#core-scenarios)  |
| [Documentos de búsqueda (API REST)](https://docs.microsoft.com/rest/api/searchservice/search-documents) | Métodos GET o POST en un índice, con parámetros de consulta para entradas adicionales.  |

## <a name="a-first-look-at-query-requests"></a>Primer vistazo a las solicitudes de consulta

Los ejemplos son útiles para introducir nuevos conceptos. Esta consulta representativa construida en la [API REST](https://docs.microsoft.com/rest/api/searchservice/search-documents), es un ejemplo cuyo destino es el [índice de la demostración de bienes inmuebles](search-get-started-portal.md) e incluye parámetros comunes.

```
{
    "queryType": "simple" 
    "search": "seattle townhouse* +\"lake\"",
    "searchFields": "description, city",
    "count": "true",
    "select": "listingId, street, status, daysOnMarket, description",
    "top": "10",
    "orderby": "daysOnMarket"
}
```

+ **`queryType`** establece el analizador, que en Azure Search puede ser el [analizador de consultas sencillo predeterminado](search-query-simple-examples.md) (óptimo para la búsqueda de texto completo) o el [analizador completo de consultas de Lucene](search-query-lucene-examples.md) utilizado para construcciones de consultas avanzadas como las expresiones regulares, la búsqueda de proximidad, la búsqueda aproximada y por caracteres comodín, por nombrar algunas.

+ **`search`** proporciona los criterios de coincidencia, normalmente texto, pero a menudo acompañado por operadores booleanos. Los términos individuales y sencillos constituyen las consultas por *término*. Las consultas entrecomilladas de varias partes son las consultas de *frases clave*. Búsqueda puede ser indefinida, como en **`search=*`**, pero es más probable que consta de operadores comunes similares a lo que aparece en el ejemplo, frases y términos.

+ **`searchFields`** es opcional, se usa para restringir la ejecución de consultas a campos específicos.

Las respuestas también están formadas por los parámetros que incluya en la consulta. En el ejemplo, el conjunto de resultados se compone de campos que aparecen en la instrucción **`select`**. En esta consulta se devuelven solo los 10 primeros puestos, pero **`count`** indica cuántos documentos coinciden en general. En esta consulta, las filas se ordenan por daysOnMarket.

En Azure Search, la ejecución de la consulta se realiza siempre en un índice, que se autentica mediante una clave de API proporcionada en la solicitud. En REST, ambos se proporcionan en los encabezados de solicitud.

### <a name="how-to-run-this-query"></a>Ejecución de esta consulta

Para ejecutar esta consulta, utilice el [explorador de búsqueda y el índice de demostración de bienes inmuebles](search-get-started-portal.md). 

Puede pegar esta cadena de consulta en la barra de búsqueda del explorador: `search=seattle townhouse +lake&searchFields=description, city&$count=true&$select=listingId, street, status, daysOnMarket, description&$top=10&$orderby=daysOnMarket`

## <a name="how-query-operations-are-enabled-by-the-index"></a>Habilitación de las operaciones de consulta según el índice

El diseño del índice y de la consulta están estrechamente unidos en Azure Search. Es crucial saber de antemano que el *esquema de índice*, con los atributos en cada campo, determina el tipo de consulta que se puede compilar. 

Los atributos de índice de un campo establecen las operaciones permitidas: si es un campo *que permite búsquedas* en el índice, *recuperable* en los resultados, *ordenable*, *filtrable*,etc. En la cadena de consulta de ejemplo, `"$orderby": "daysOnMarket"` solo funciona porque el campo daysOnMarket está marcado como *ordenable* en el esquema de índice. 

![Definición del índice en el ejemplo de bienes inmuebles](./media/search-query-overview/realestate-sample-index-definition.png "Definición del índice en el ejemplo de bienes inmuebles")

La captura de pantalla anterior es una lista parcial de atributos de índice para el ejemplo de bienes inmuebles. En el portal puede ver el esquema de índice completo. Para más información acerca de los atributos de índice, consulte el artículo sobre la [creación de índices con la API REST](https://docs.microsoft.com/rest/api/searchservice/create-index).

> [!Note]
> Algunas funcionalidades de consulta están habilitadas para todo el índice en lugar de por campo. Estas funcionalidades incluyen: [asignaciones de sinónimos](search-synonyms.md), [analizadores personalizados](index-add-custom-analyzers.md), [construcciones de proveedor de sugerencias (para Autocompletar y las consultas sugeridas)](index-add-suggesters.md), [lógica de puntuación para clasificar los resultados](index-add-scoring-profiles.md).

## <a name="elements-of-a-query-request"></a>Elementos de una solicitud de consulta

Las consultas siempre se dirigen a un solo índice. No es posible combinar índices ni crear estructuras de datos personalizadas o temporales como destino de la consulta. 

Entre los elementos obligatorios en una solicitud de consulta se incluyen los siguientes componentes:

+ Punto de conexión de servicio y colección de los documentos del índice, expresado como dirección URL con los componentes fijos y los definidos por el usuario: **`https://<your-service-name>.search.windows.net/indexes/<your-index-name>/docs`**
+ Se necesita **`api-version`** (solo REST), ya que siempre hay más de una versión de la API disponible. 
+ **`api-key`**, clave de API de consulta o administrador que autentique la solicitud en el servicio.
+ **`queryType`**, simplificada o completa, que se puede omitir si usa la sintaxis simplificada predeterminada integrada.
+ **`search`** o **`filter`** proporcionan los criterios de coincidencia, pueden dejarse sin especificar si se desea realizar una búsqueda vacía. Ambos tipos de consulta se describen en términos del analizador sencillo, pero incluso las consultas avanzadas requieren el parámetro de búsqueda para pasar las expresiones de consulta complejas.

Todos los demás parámetros de búsqueda son opcionales. Para la lista completa de atributos, consulte el artículo de [creación de índices (REST)](https://docs.microsoft.com/rest/api/searchservice/create-index). Para una visión más detallada de cómo se usan los parámetros durante el procesamiento, consulte [Cómo funciona la búsqueda de texto completo en Azure Search](search-lucene-query-architecture.md).

## <a name="choose-a-parser-simple--full"></a>Elección del analizador: básico | completo

Azure Search se basa en Apache Lucene y le permite elegir entre dos analizadores de consulta para gestionar consultas típicas y especializadas. Las solicitudes se formulan mediante el analizador básico con [sintaxis de consulta simplificada](query-simple-syntax.md), seleccionada como predeterminada por su velocidad y eficacia para las consultas de texto libre. Esta sintaxis admite varios operadores de búsqueda comunes entre los que se incluyen los operadores de frase, sufijo, precedencia, AND, OR y NOT.

La [sintaxis de consulta completa de Lucene](query-Lucene-syntax.md#bkmk_syntax), que se habilita al agregar `queryType=full` a la solicitud, permite usar el expresivo lenguaje de consulta adoptado ampliamente desarrollado en [Apache Lucene](https://lucene.apache.org/core/4_10_2/queryparser/org/apache/lucene/queryparser/classic/package-summary.html). La sintaxis completa amplía la sintaxis simplificada. Cualquier consulta que escriba con sintaxis simplificada se ejecuta en el analizador de Lucene completo. 

Los ejemplos siguientes ilustran el punto: misma consulta, pero con valores de queryType distintos, producen resultados diferentes. En la primera consulta, `^3` se trata como parte del término de búsqueda.

```
queryType=simple&search=mountain beach garden ranch^3&searchFields=description&$count=true&$select=listingId, street, status, daysOnMarket, description&$top=10&$orderby=daysOnMarket
```

La misma consulta con el analizador de Lucene completo interpreta el aumento en el campo "ranch", aumentando así la clasificación de búsqueda de los resultados que contengan ese término específico.

```
queryType=full&search=mountain beach garden ranch^3&searchFields=description&$count=true&$select=listingId, street, status, daysOnMarket, description&$top=10&$orderby=daysOnMarket
```

<a name="types-of-queries"></a>

## <a name="types-of-queries"></a>Tipos de consultas

Azure Search admite una amplia gama de tipos de consulta. 

| Tipo de consulta | Uso | Más información y ejemplos |
|------------|--------|-------------------------------|
| Búsqueda de texto libre | Parámetro de búsqueda y cualquier analizador| La consulta de texto completo busca uno o más términos en todos los campos *que permiten búsquedas* del índice y funciona tal y como cabría esperar de un motor de búsqueda como Google o Bing. El ejemplo de la introducción es una búsqueda de texto completo.<br/><br/>La búsqueda de texto completo se somete a análisis de texto con el analizador de Lucene estándar (predeterminado) para poner todos los términos en minúscula y eliminar palabras vacías como "the". Puede invalidar el valor predeterminado con [analizadores de idiomas distintos al inglés](index-add-language-analyzers.md#language-analyzer-list) o [analizadores independientes del idioma especializados](index-add-custom-analyzers.md#AnalyzerTable) que modifiquen el análisis de texto. Un ejemplo es el analizador por [palabra clave](https://lucene.apache.org/core/4_10_3/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html), que trata el contenido de un campo como token único. Esto es útil para los datos como códigos postales, identificadores y algunos nombres de producto. | 
| Búsqueda filtrada | [Expresión de filtro de OData](query-odata-filter-orderby-syntax.md) y cualquier analizador | Las consultas de filtro evalúan una expresión booleana en todos los campos *filtrables* de un índice. A diferencia de la búsqueda, una consulta de filtro busca el contenido exacto de un campo, incluidas las mayúsculas y minúsculas de los campos de cadena. Otra diferencia es que las consultas de filtro se expresan con sintaxis de OData. <br/>[Ejemplo de expresión de filtro](search-query-simple-examples.md#example-3-filter-queries) |
| Búsqueda georreferenciada | [Tipo Edm.GeographyPoint](https://docs.microsoft.com/rest/api/searchservice/supported-data-types) en el campo, la expresión de filtro y cualquier analizador | Las coordenadas que se almacenan en un campo con Edm.GeographyPoint se utilizan para "buscar cerca de mí" o los controles de búsqueda con mapas. <br/>[Ejemplo de búsqueda georreferenciada](search-query-simple-examples.md#example-5-geo-search)|
| Búsqueda de intervalo | Expresión de filtro y analizador básico | En Azure Search, las consultas de intervalo se compilan con el parámetro de filtro. <br/>[Ejemplo de filtro de intervalo](search-query-simple-examples.md#example-4-range-filters) | 
| [Clasificada por campos de búsqueda](query-lucene-syntax.md#bkmk_fields) | Parámetro de búsqueda y analizador completo | Compila una expresión de consulta compuesta con un único campo como destino. <br/>[Ejemplo de búsqueda clasificada por campos](search-query-lucene-examples.md#example-2-fielded-search) |
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

+ Cambie la técnica de la consulta si se necesita análisis léxico o de texto, pero el tipo de consulta impide el procesamiento lingüístico. En la búsqueda de texto completo, texto o análisis léxico autocorrects para errores de ortografía, formas de palabras de singular a plural y verbos incluso irregulares o sustantivos. En algunas consultas, como la búsqueda aproximada o la búsqueda con caracteres comodín, el análisis de texto no forma parte de la canalización de análisis de la consulta. En algunos escenarios, se han utilizado expresiones regulares como solución alternativa. 

### <a name="paging-results"></a>Paginación de resultados
Azure Search facilita la implementación de la paginación de los resultados de búsqueda. Mediante el uso de los parámetros **`top`** y **`skip`**, puede emitir fácilmente solicitudes de búsqueda que le permitan recibir el conjunto total de resultados de búsqueda dividido en subconjuntos ordenados, sencillos de administrar que permitan habilitar fácilmente prácticas eficaces de interfaz de usuario de búsqueda. Al recibir estos subconjuntos más pequeños de resultados, también puede recibir el número de documentos del conjunto total de resultados de la búsqueda.

Para más información acerca de la paginación de resultados de búsqueda, consulte el artículo [Cómo paginar los resultados de la búsqueda en Azure Search](search-pagination-page-layout.md).

### <a name="ordering-results"></a>Ordenación de los resultados
Al recibir los resultados de una consulta de búsqueda, puede solicitar que Azure Search muestre los resultados ordenados según los valores de un campo específico. De forma predeterminada, Azure Search ordena los resultados de búsqueda basándose en el rango de puntuación de búsqueda de cada documento, que se deriva del valor [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf).

Si desea que Azure Search devuelva los resultados ordenados según un valor distinto de la puntuación de búsqueda, puede usar el parámetro **`orderby`**. Puede especificar el valor del parámetro **`orderby`** para que incluya nombres de campo y llamadas a la [**`geo.distance()` función**](query-odata-filter-orderby-syntax.md) para los valores geoespaciales. Cada expresión puede ir seguida de `asc` para indicar que se solicitan los resultados en orden ascendente y de **`desc`** para indicar que se solicitan los resultados en orden descendente. El valor predeterminado es el orden ascendente.


### <a name="hit-highlighting"></a>Resaltado de referencias
En Azure Search, resaltar la parte exacta de los resultados de búsqueda que coincide con la consulta de búsqueda es fácil mediante el uso de los parámetros **`highlight`**, **`highlightPreTag`**, y **`highlightPostTag`**. Puede especificar qué campos *habilitados para búsquedas* deben tener su texto coincidente resaltado, así como especificar las etiquetas de cadena exactas que se anexarán al comienzo y al final del texto coincidente que devuelve Azure Search.

## <a name="see-also"></a>Vea también

+ [Cómo funciona la búsqueda de texto completo en Azure Search (arquitectura de análisis de consultas)](search-lucene-query-architecture.md)
+ [Explorador de búsqueda](search-explorer.md)
+ [Consultas con .NET](search-query-dotnet.md)
+ [Consultas con REST](search-create-index-rest-api.md)
