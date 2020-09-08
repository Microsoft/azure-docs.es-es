---
title: Creación de una consulta simple
titleSuffix: Azure Cognitive Search
description: Aprenda por ejemplo mediante la ejecución de consultas basadas en la sintaxis simple para la búsqueda de texto completo, la búsqueda filtrada, la búsqueda geográfica o la búsqueda por facetas en un índice de Azure Cognitive Search.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 11/04/2019
ms.openlocfilehash: afc9f8e29cf27734787da9cab3e3456e5414d9ac
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88918033"
---
# <a name="create-a-simple-query-in-azure-cognitive-search"></a>Creación de una consulta simple en Azure Cognitive Search

En Azure Cognitive Search, la [sintaxis de consulta simplificada](query-simple-syntax.md) invoca el analizador de consultas predeterminado para ejecutar consultas de búsqueda de texto completo en un índice. Este analizador es rápido y gestiona escenarios comunes, incluida la búsqueda de texto completo, la búsqueda por facetas, la búsqueda filtrada y la búsqueda geográfica. 

En este artículo, usamos ejemplos para ilustrar la sintaxis simple, rellenando el parámetro `search=` de una operación de [búsqueda de documentos](/rest/api/searchservice/search-documents).

Una sintaxis de consulta alternativa es [Lucene completa](query-lucene-syntax.md), compatible con estructuras de consulta más complejas, como la búsqueda aproximada y la búsqueda con caracteres comodín, por lo que puede tardar más tiempo en procesarse. Para más información y ejemplos que muestran la sintaxis completa, consulte [Uso de la sintaxis comleta de Lucene](search-query-lucene-examples.md).

## <a name="formulate-requests-in-postman"></a>Formulación de solicitudes en Postman

Los ejemplos siguientes aprovechan un índice de búsqueda de trabajos de Nueva York que consta de los disponibles según un conjunto de datos proporcionado por la iniciativa [City of New York OpenData](https://nycopendata.socrata.com/). Estos datos no deben considerarse actuales o completos. El índice está en un servicio de espacio aislado que proporciona Microsoft, lo que significa que no necesita una suscripción a Azure o a Azure Cognitive Search para probar estas consultas.

Lo que necesita es Postman o una herramienta equivalente para emitir la solicitud HTTP en GET. Para más información, consulte [Inicio rápido: Exploración de la API REST de Azure Cognitive Search mediante Postman](search-get-started-postman.md).

### <a name="set-the-request-header"></a>Establecimiento del encabezado de solicitud

1. En el encabezado de solicitud, establezca **Content-Type** en `application/json`.

2. Agregue un valor de **clave-api** y establézcala en esta cadena: `252044BE3886FE4A8E3BAA4F595114BB`. Se trata de una clave de consulta para el servicio de búsqueda de espacio aislado que hospeda el índice de trabajos de Nueva York.

Después de especificar el encabezado de solicitud, puede volver a usarlo para todas las consultas de este artículo, cambiando solo la cadena **search=** . 

  ![Encabezado de solicitud de Postman](media/search-query-lucene-examples/postman-header.png)

### <a name="set-the-request-url"></a>Establecimiento de la dirección URL de solicitud

La solicitud es un comando GET emparejado con una dirección URL que contiene la cadena de búsqueda y el punto de conexión de Azure Cognitive Search.

  ![Encabezado de solicitud de Postman](media/search-query-lucene-examples/postman-basic-url-request-elements.png)

La composición de dirección URL tiene los siguientes elementos:

+ **`https://azs-playground.search.windows.net/`** es un servicio de búsqueda de espacio aislado que mantiene el equipo de desarrollo de Azure Cognitive Search. 
+ **`indexes/nycjobs/`** es el índice de trabajos de Nueva York en la colección de índices de ese servicio. Tanto el nombre del servicio como el índice son necesarios en la solicitud.
+ **`docs`** es la colección de documentos que incluye todo el contenido que permite búsquedas. El valor de clave-api de la consulta proporcionada en el encabezado de solicitud solo funciona en las operaciones de lectura destinadas a la colección de documentos.
+ **`api-version=2020-06-30`** establece la versión de api, que es un parámetro necesario en cada solicitud.
+ **`search=*`** es la cadena de consulta, que en la consulta inicial es NULL, con lo que se devuelven los 50 primeros resultados (de forma predeterminada).

## <a name="send-your-first-query"></a>Envío de la primera consulta

Como paso de comprobación, pegue la siguiente solicitud en GET y haga clic en **Enviar**. Los resultados se devuelven como documentos JSON detallados. Se devuelven documentos completos, lo que permite ver todos los campos y todos los valores.

Pegue esta URL en un cliente REST como paso de validación y para ver la estructura del documento.

  ```http
  https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&search=*
  ```

La cadena de consulta, **`search=*`** , es una búsqueda sin especificar equivalente a la búsqueda vacía o NULL. No resulta especialmente útil, pero es la búsqueda más sencilla que se puede hacer.

Si lo desea, puede agregar **`$count=true`** a la dirección URL para devolver un recuento de los documentos que coinciden con los criterios de búsqueda. En una cadena de búsqueda vacía, se trata de todos los documentos del índice (unos 2800 en el caso de los trabajos de Nueva York).

## <a name="how-to-invoke-simple-query-parsing"></a>Cómo invocar el análisis de consulta simplificada

Para las consultas interactivas, no tiene que especificar nada: la consulta simplificada es la predeterminada. En el código, si se ha invocado anteriormente **queryType=full** para la sintaxis de consulta completa, podría restablecer el valor predeterminado con **queryType=simple**.

## <a name="example-1-field-scoped-query"></a>Ejemplo 1: Consulta de ámbito de campo

El primer ejemplo no es específico del analizador, pero sirve para introducir el primer concepto fundamental de consulta: la independencia. En este ejemplo se establece el ámbito de ejecución de consultas y la respuesta a unos pocos campos específicos. Es importante saber cómo estructurar una respuesta JSON legible cuando la herramienta es Postman o el Explorador de búsqueda. 

Por brevedad, la consulta tiene como destino únicamente el campo *business_title* y especifica que se devuelvan solo los puestos de empresa. La sintaxis es **searchFields** para restringir la ejecución de consultas a solo el campo de business_title y **select** para especificar qué campos se incluyen en la respuesta.

### <a name="partial-query-string"></a>Cadena de consulta parcial

```http
searchFields=business_title&$select=business_title&search=*
```

Esta es la misma consulta con varios campos en una lista delimitada por comas.

```http
search=*&searchFields=business_title, posting_type&$select=business_title, posting_type
```

### <a name="full-url"></a>URL completa

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&searchFields=business_title&$select=business_title&search=*
```

La respuesta de esta consulta debe tener un aspecto similar a la siguiente captura de pantalla.

  ![Respuesta de ejemplo de Postman](media/search-query-lucene-examples/postman-sample-results.png)

Tal vez haya notado la puntuación de búsqueda en la respuesta. Las puntuaciones uniformes de 1 se producen cuando no hay ninguna clasificación, ya sea debido a que la búsqueda no era de texto completo o porque no se ha aplicado ningún criterio. Para la búsqueda de valores null sin ningún criterio, las filas vuelven en orden aleatorio. Al incluir criterios reales, verá que las puntuaciones de búsqueda evolucionan en valores significativos.

## <a name="example-2-look-up-by-id"></a>Ejemplo 2: Búsqueda por identificador

Este ejemplo es un poco inusual, pero al evaluar comportamientos de búsqueda, es posible que desee inspeccionar todo el contenido de un documento concreto para entender por qué se incluye o se excluye de los resultados. Para devolver un solo documento al completo, utilice una [operación de búsqueda](/rest/api/searchservice/lookup-document) para pasar el identificador del documento.

Todos los documentos tienen un identificador único. Para probar la sintaxis de una consulta de búsqueda, en primer lugar devuelva una lista de identificadores de documento para elegir el que desea utilizar. Para NYC Jobs, los identificadores están almacenados en el campo `id`.

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&searchFields=id&$select=id&search=*
```

El ejemplo siguiente es una consulta de búsqueda que devuelve un documento específico basado en el `id` "9E1E3AF9-0660-4E00-AF51-9B654925A2D5", que apareció por primera vez en la respuesta anterior. La consulta siguiente devuelve el documento completo, no solo campos seleccionados. 

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs/9E1E3AF9-0660-4E00-AF51-9B654925A2D5?api-version=2020-06-30&$count=true&search=*
```

## <a name="example-3-filter-queries"></a>Ejemplo 3: Consultas de filtro

La [sintaxis de filtro](./search-query-odata-filter.md) es una expresión de OData que se puede usar con la **búsqueda** o por sí sola. Un filtro independiente, sin parámetros de consulta, resulta útil cuando la expresión de filtro puede definir completamente los documentos de interés. Sin una cadena de consulta, no hay ningún análisis lingüístico ni léxico, ni puntuación (todas son 1), ni clasificación. Tenga en cuenta que la cadena de búsqueda está vacía.

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "search": "",
      "filter": "salary_frequency eq 'Annual' and salary_range_from gt 90000",
      "select": "job_id, business_title, agency, salary_range_from",
      "count": "true"
    }
```

Si los usa conjuntamente, el filtro se aplica primero a todo el índice y, después, se realiza la búsqueda en los resultados del filtro. Por lo tanto, los filtros pueden ser una técnica útil para mejorar el rendimiento porque reducen el conjunto de documentos en los que se debe procesar la consulta de búsqueda.

  ![Respuesta de la consulta de filtro](media/search-query-simple-examples/filtered-query.png)

Si desea probar esto en Postman mediante GET, puede pegar en esta cadena:

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&$select=job_id,business_title,agency,salary_range_from&search=&$filter=salary_frequency eq 'Annual' and salary_range_from gt 90000
```

Otra manera eficaz de combinar el filtrado y la búsqueda es mediante **`search.ismatch*()`** en una expresión de filtro, donde puede usar una consulta de búsqueda dentro del filtro. Esta expresión de filtro utiliza un carácter comodín en *plan* seleccionar business_title, incluido el periodo, el programador, el planeamiento, etc.

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&$select=job_id,business_title,agency&search=&$filter=search.ismatch('plan*', 'business_title', 'full', 'any')
```

Para más información acerca de la función, consulte [search.ismatch en los ejemplos de filtro](./search-query-odata-full-text-search-functions.md#examples).

## <a name="example-4-range-filters"></a>Ejemplo 4: Filtros de rango

Se admite el filtrado de rango mediante expresiones **`$filter`** para cualquier tipo de datos. En los siguientes ejemplos se busca en campos numéricos y de cadena. 

Los tipos de datos son importantes en los filtros de rango y funcionan mejor cuando los campos numéricos contienen datos numéricos y los de cadena, datos de cadena. Los datos numéricos en los campos de cadena no son adecuados para los rangos, ya que los números y las cadenas no se pueden comparar en Azure Cognitive Search. 

Los ejemplos siguientes están en formato POST para mejorar la legibilidad (rango numérico, seguido por el rango de texto):

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "search": "",
      "filter": "num_of_positions ge 5 and num_of_positions lt 10",
      "select": "job_id, business_title, num_of_positions, agency",
      "orderby": "agency",
      "count": "true"
    }
```
  ![Filtro de rango para rangos numéricos](media/search-query-simple-examples/rangefilternumeric.png)


```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "search": "",
      "filter": "business_title ge 'A*' and business_title lt 'C*'",
      "select": "job_id, business_title, agency",
      "orderby": "business_title",
      "count": "true"
    }
```

  ![Filtro de rango para rangos de texto](media/search-query-simple-examples/rangefiltertext.png)

También puede probarlos en Postman mediante GET:

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&search=&$filter=num_of_positions ge 5 and num_of_positions lt 10&$select=job_id, business_title, num_of_positions, agency&$orderby=agency&$count=true
```

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&search=&$filter=business_title ge 'A*' and business_title lt 'C*'&$select=job_id, business_title, agency&$orderby=business_title&$count=true
```

> [!NOTE]
> El uso de facetas en intervalos de valores es un requisito habitual de las aplicaciones de búsqueda. Para más información y ejemplos sobre la compilación de filtros para estructuras de navegación de facetas, consulte ["Filtro basado en un intervalo" en *Procedimiento para implementar la navegación por facetas*](search-faceted-navigation.md#filter-based-on-a-range).

## <a name="example-5-geo-search"></a>Ejemplo 5: Búsqueda georreferenciada

El índice de ejemplo incluye un campo geo_location con las coordenadas de latitud y longitud. En este ejemplo se usa la [función geo.distance](./search-query-odata-geo-spatial-functions.md#examples), que filtra en documentos dentro de la circunferencia de un punto de partida, hasta la distancia arbitraria (en kilómetros) que proporcione. Puede ajustar el último valor en la consulta (4) para reducir o ampliar el área expuesta de la consulta.

El ejemplo siguiente está en formato POST para mejorar la legibilidad:

```http
POST /indexes/nycjobs/docs/search?api-version=2020-06-30
    {
      "search": "",
      "filter": "geo.distance(geo_location, geography'POINT(-74.11734 40.634384)') le 4",
      "select": "job_id, business_title, work_location",
      "count": "true"
    }
```
Para obtener resultados más legibles, los resultados de búsqueda se recortan para incluir el identificador, el puesto y la ubicación de trabajo. Las coordenadas iniciales se obtuvieron de un documento aleatorio del índice (en este caso, para una ubicación de trabajo en la isla Staten).

También puede probarlo en Postman mediante GET:

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&search=&$select=job_id, business_title, work_location&$filter=geo.distance(geo_location, geography'POINT(-74.11734 40.634384)') le 4
```

## <a name="example-6-search-precision"></a>Ejemplo 6: Precisión de la búsqueda

Las consultas de términos son términos individuales (puede que muchos de ellos) que se evalúan de forma independiente. Las consultas de frases se incluyen entre comillas y se evalúan como una cadena textual. La precisión de la coincidencia se controla mediante operadores y el parámetro searchMode.

Ejemplo 1: **`&search=fire`** devuelve 150 resultados, donde todas las coincidencias contienen la palabra "fire" en algún lugar del documento.

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&search=fire
```

Ejemplo 2: **`&search=fire department`** devuelve 2002 resultados. Se devuelven coincidencias para documentos que contienen las palabras "fire" o "department".

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&search=fire department
```

Ejemplo 3: **`&search="fire department"`** devuelve 82 resultados. Al incluir la cadena entre comillas se convierte en una búsqueda textual de ambos términos y las coincidencias encontradas en términos con tokens en el índice están formadas por los términos combinados. Esto explica por qué una búsqueda como **`search=+fire +department`** no es equivalente. Ambos términos son obligatorios, pero se buscan de forma independiente. 

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&search="fire department"
```

## <a name="example-7-booleans-with-searchmode"></a>Ejemplo 7: Valores booleanos con searchMode

La sintaxis simplificada es compatible con operadores booleanos en forma de caracteres (`+, -, |`). El parámetro searchMode informa de las compensaciones entre precisión y recuperación, donde `searchMode=any` favorece la recuperación (la coincidencia con cualquier criterio permite incluir a un documento en el conjunto de resultados) y `searchMode=all` favorece la precisión (deben cumplirse todos los criterios). El valor predeterminado es `searchMode=any`, lo que puede llevar a confusión si se apila una consulta con varios operadores y se obtienen resultados más amplios, en lugar de más restringidos. Esto sucede especialmente con el operador NOT, donde los resultados incluyen todos los documentos que "no contienen" un término específico.

Con el parámetro searchMode predeterminado (cualquiera), se devuelven 2800 documentos: aquellos que contienen el término de varias partes "fire department", más todos los documentos que no contienen el término "Metrotech Center".

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&searchMode=any&search="fire department"  -"Metrotech Center"
```

  ![modo de búsqueda: any](media/search-query-simple-examples/searchmodeany.png)

Al cambiar searchMode a `all`, se aplica un efecto acumulativo en los criterios y se devuelve un conjunto de resultados más pequeño (21 documentos) que consta de los documentos que contienen la frase completa "fire department", menos los trabajos con la dirección de Metrotech Center.

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&searchMode=all&search="fire department"  -"Metrotech Center"
```
  ![modo de búsqueda: all](media/search-query-simple-examples/searchmodeall.png)

## <a name="example-8-structuring-results"></a>Ejemplo 8: Estructuración de los resultados

Varios parámetros controlan qué campos se incluyen en la búsqueda de resultados, el número de documentos devueltos en cada lote y el criterio de ordenación. En este ejemplo reaparecen algunos de los ejemplos anteriores, al limitar los resultados a campos específicos mediante la instrucción **$select** y criterios de búsqueda textual, devolviendo 82 coincidencias 

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&$select=job_id,agency,business_title,civil_service_title,work_location,job_description&search="fire department"
```
Al anexarlo al ejemplo anterior, podrá ordenar por título. Esta ordenación funciona porque civil_service_title *se puede ordenar* en el índice.

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&$select=job_id,agency,business_title,civil_service_title,work_location,job_description&search="fire department"&$orderby=civil_service_title
```

La paginación de resultados se implementa mediante el parámetro **$top**, devolviendo en este caso los cinco documentos principales:

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&$select=job_id,agency,business_title,civil_service_title,work_location,job_description&search="fire department"&$orderby=civil_service_title&$top=5&$skip=0
```

Para obtener los cinco siguientes, omita el primer lote:

```http
https://azs-playground.search.windows.net/indexes/nycjobs/docs?api-version=2020-06-30&$count=true&$select=job_id,agency,business_title,civil_service_title,work_location,job_description&search="fire department"&$orderby=civil_service_title&$top=5&$skip=5
```

## <a name="next-steps"></a>Pasos siguientes
Pruebe a especificar las consultas en el código. Los vínculos siguientes explican cómo configurar las consultas de búsqueda para la API REST y .NET mediante la sintaxis simplificada predeterminada.

* [Consultas del índice con el SDK para .NET](./search-get-started-dotnet.md)
* [Consultas del índice con la API REST](./search-get-started-powershell.md)

En los vínculos siguientes, se puede encontrar una referencia de la sintaxis, la arquitectura de las consultas y ejemplos:

+ [Ejemplos de consulta con sintaxis de Lucene para la creación de consultas avanzadas](search-query-lucene-examples.md)
+ [Funcionamiento de la búsqueda de texto completo en Azure Cognitive Search](search-lucene-query-architecture.md)
+ [Sintaxis de consulta simplificada](/rest/api/searchservice/simple-query-syntax-in-azure-search)
+ [Consulta completa de Lucene](/rest/api/searchservice/lucene-query-syntax-in-azure-search)
+ [Sintaxis de filtro y de ordenación](/rest/api/searchservice/odata-expression-syntax-for-azure-search)