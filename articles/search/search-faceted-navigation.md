---
title: 'Procedimiento para implementar la navegación por facetas en una jerarquía de categorías: Azure Search'
description: Agregue navegación con facetas a aplicaciones que se integran con Búsqueda de Azure, un servicio de búsqueda hospedado en la nube en Microsoft Azure.
author: HeidiSteen
manager: cgronlun
services: search
ms.service: search
ms.topic: conceptual
ms.date: 05/13/2019
ms.author: heidist
ms.custom: seodec2018
ms.openlocfilehash: c032dbc528ed5034280d0ecb4c95700b51869991
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65793618"
---
# <a name="how-to-implement-faceted-navigation-in-azure-search"></a>Procedimiento para implementar la navegación por facetas en Azure Search
La navegación por facetas es un mecanismo de filtrado que proporciona una navegación en profundidad autodirigida en aplicaciones de búsqueda. El término "navegación por facetas" puede resultarle desconocido, pero probablemente es algo que ya ha usado con anterioridad. Como se muestra en el siguiente ejemplo, la navegación por facetas no es más que las categorías usadas para filtrar los resultados.

 ![Demo del Portal de Azure Search trabajo](media/search-faceted-navigation/azure-search-faceting-example.png "trabajo Demo del Portal de Azure Search")

La navegación por facetas es un punto de entrada alternativo para buscar. Ofrece una alternativa cómoda a escribir expresiones de búsqueda complejas a mano. Las facetas pueden ayudarle a encontrar lo que busca y le garantizan que obtendrá al menos un resultado. Como desarrollador, las facetas permiten exponer los criterios de búsqueda más útiles para navegar por el índice de búsqueda. En las aplicaciones de tienda en línea, la navegación por facetas suele basarse en marcas, departamentos (zapatos para niños), tamaño, precio, popularidad y clasificación. 

La implementación de la navegación por facetas difiere en las distintas tecnologías de búsqueda. En Azure Search, la navegación por facetas se compila en tiempo de consulta usando campos que haya atribuido anteriormente en el esquema.

-   En las consultas que la aplicación crea, una consulta tiene que enviar *parámetros de consulta de faceta* para obtener los valores de filtro de faceta disponibles para ese conjunto de resultados de documento.

-   Para recortar realmente el conjunto de resultados de documento, la aplicación tiene que aplicar también una expresión `$filter`.

En el desarrollo de aplicaciones, la escritura del código que construye las consultas supone la mayor parte del trabajo. El servicio proporciona muchos de los comportamientos de aplicación que podrían interesarle de la navegación por facetas, incluida la compatibilidad integrada para establecer intervalos y obtener recuentos de los resultados de la faceta. El servicio también incluye valores predeterminados razonables que le ayudarán a evitar estructuras de navegación difíciles de manejar. 

## <a name="sample-code-and-demo"></a>Demostración y código de ejemplo
En este artículo se utiliza como ejemplo un portal de búsqueda de trabajo. El ejemplo se implementa como una aplicación ASP.NET MVC.

-   Vea y pruebe la demostración de trabajo en línea en [Azure Search Job Portal Demo](https://azjobsdemo.azurewebsites.net/) (Demostración del portal de búsqueda de trabajo de Azure Search).

-   Descargue el código del [repositorio de ejemplos de Azure en GitHub](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs).

## <a name="get-started"></a>Primeros pasos
Si no está familiarizado con el desarrollo de búsquedas, la mejor manera de pensar en la navegación por facetas es que muestra las posibilidades de una búsqueda autodirigida. Es un tipo de experiencia de búsqueda en profundidad basada en filtros predefinidos, que se usa para restringir rápidamente los resultados de búsqueda mediante acciones de apuntar y hacer clic. 

### <a name="interaction-model"></a>Modelo de interacción

La experiencia de búsqueda de la navegación por facetas es iterativa, por lo que comenzaremos por describirla como una secuencia de consultas que se despliega en respuesta a las acciones del usuario.

El punto de partida es una página de aplicación que proporciona navegación por facetas, normalmente situada en la periferia. La navegación por facetas suele tener una estructura de árbol con casillas para cada valor o texto en el que se puede hacer clic. 

1. Una consulta enviada a Búsqueda de Azure especifica la estructura de la navegación por facetas mediante uno o más parámetros de consulta de faceta. Por ejemplo, la consulta podría incluir `facet=Rating`, quizá con una opción `:values` o `:sort` para refinar aún más la presentación.
2. La capa de presentación representa una página de búsqueda que proporciona navegación por facetas, usando las facetas especificadas en la solicitud.
3. En una estructura de navegación por facetas dada que incluye el campo de valoración Rating, haga clic en "4" para indicar que solo se deben mostrar los productos con una valoración de 4 o superior. 
4. En respuesta, la aplicación envía una consulta que incluye `$filter=Rating ge 4` 
5. La capa de presentación actualiza la página para mostrar un conjunto de resultados reducido únicamente con los elementos que cumplen el nuevo criterio (en este caso, los productos con una valoración de 4 y superior).

Una faceta es un parámetro de consulta, pero no lo confunda con una entrada de consulta. Nunca se usa como criterio de selección en una consulta. En su lugar, piense en los parámetros de consulta de faceta como entradas a la estructura de navegación que se devuelven en la respuesta. Para cada parámetro de consulta de faceta que proporcione, Azure Search evaluará cuántos documentos hay en los resultados parciales de cada valor de faceta.

Observe la expresión `$filter` en el paso 4. El filtro es un aspecto importante de la navegación por facetas. Aunque las facetas y los filtros son independientes en la API, ambos son necesarios para proporcionarle la experiencia que desea. 

### <a name="app-design-pattern"></a>Patrón de diseño de la aplicación

En el código de aplicación, el patrón es usar parámetros de consulta de faceta para devolver la estructura de navegación por facetas junto con los resultados de las facetas, además de una expresión $filter.  La expresión de filtro controla el evento de clic en el valor de faceta. Piense en la expresión `$filter` como en el código subyacente al recorte real de los resultados de búsqueda que se devuelven a la capa de presentación. En una faceta Colors dada, la posibilidad de hacer clic en el color Red se implementa mediante una expresión `$filter` que selecciona solo los elementos que tienen un color rojo. 

### <a name="query-basics"></a>Conceptos básicos de las consultas

En Búsqueda de Azure, una solicitud se especifica mediante uno o más parámetros de consulta (consulte [Buscar documentos](https://docs.microsoft.com/rest/api/searchservice/Search-Documents) para obtener una descripción de cada uno). Ninguno de los parámetros de consulta son obligatorios, pero debe tener al menos uno para que una consulta sea válida.

La precisión, entendida como la capacidad de filtrar los resultados irrelevantes, se consigue mediante una o ambas de estas expresiones:

-   **search=**  
     El valor de este parámetro constituye la expresión de búsqueda. Podría ser un fragmento de texto o una expresión de búsqueda compleja que incluye varios términos y operadores. En el servidor, las expresiones de búsqueda se usan para realizar búsquedas de texto completo que consultan en los campos de búsqueda del índice si hay términos coincidentes y devuelven los resultados en orden. Si establece `search` en NULL, la ejecución de la consulta se realiza en todo el índice (es decir, `search=*`). En este caso, los demás elementos de la consulta, como un `$filter` o el perfil de puntuación, son los principales factores que afectan a qué documentos se devuelven (`($filter`) y en qué orden (`scoringProfile` o `$orderby`).

-   **$filter=**  
     Un filtro es un mecanismo eficaz para limitar el tamaño de los resultados de búsqueda según los valores de atributos de documento específicos. Primero se evalúa una expresión `$filter` y después la lógica de uso de facetas que genera los valores disponibles y los correspondientes recuentos para cada valor.

Las expresiones de búsqueda complejas reducen el rendimiento de la consulta. Siempre que sea posible, use expresiones de filtro bien construidas para aumentar la precisión y mejorar el rendimiento de las consultas.

Para entender mejor cómo los filtros agregan más precisión, compare una expresión de búsqueda compleja con una que incluya una expresión de filtro:

-   `GET /indexes/hotel/docs?search=lodging budget +Seattle –motel +parking`
-   `GET /indexes/hotel/docs?search=lodging&$filter=City eq ‘Seattle’ and Parking and Type ne ‘motel’`

Aunque ambas consultas son válidas, la segunda es mejor si está buscando hoteles que no sean moteles con estacionamiento en Seattle.
-   La primera consulta se basa en las palabras específicas que se mencionan o no en campos de cadena tales como nombre, descripción u otros que contengan datos que se pueden buscar.
-   La segunda consulta busca coincidencias precisas en datos estructurados y es probable que sea mucho más exacta.

En las aplicaciones que incluyen navegación por facetas, asegúrese de que cada acción del usuario en una estructura de navegación por facetas vaya acompañada de una restricción de los resultados de búsqueda. Para restringir los resultados, utilice una expresión de filtro.

<a name="howtobuildit"></a>

## <a name="build-a-faceted-navigation-app"></a>Compilación de una aplicación de navegación por facetas
Implemente la navegación por facetas con Azure Search en el código de la aplicación que compila la solicitud de búsqueda. El panel de navegación por facetas se basa en los elementos del esquema que definió anteriormente.

En el índice de búsqueda está predefinido el atributo de índice `Facetable [true|false]` , que se establece en los campos seleccionados para habilitar o deshabilitar su uso en una estructura de navegación por facetas. Sin `"Facetable" = true`, un campo no se puede usar en la navegación por facetas.

La capa de presentación del código proporciona la experiencia del usuario. Debe enumerar las partes que constituyen la navegación por facetas, como la etiqueta, los valores, las casillas y el recuento. La API de REST de Búsqueda de Azure es independiente de la plataforma, así que puede usar cualquier lenguaje y plataforma que desee. Lo importante es incluir elementos de interfaz de usuario que admitan la actualización incremental, actualizando el estado de la interfaz de usuario cuando se seleccione cada faceta adicional. 

En tiempo de consulta, el código de aplicación crea una solicitud que incluye `facet=[string]`, un parámetro de solicitud que proporciona el campo por el que se realizará la consulta por facetas. Una consulta puede tener varias facetas, como `&facet=color&facet=category&facet=rating`, separadas por un carácter de y comercial (&).

El código de aplicación también debe construir una expresión `$filter` para controlar los eventos de clic en la navegación por facetas. Una expresión `$filter` reduce los resultados de búsqueda, usando el valor de la faceta como criterio de filtro.

Azure Search devuelve los resultados de búsqueda, basados en uno o más de los términos que haya escrito, junto con las actualizaciones de la estructura de navegación por facetas. En Búsqueda de Azure, la navegación por facetas es una construcción de un solo nivel, con los valores de las facetas y los recuentos de cuántos resultados se encuentran para cada una.

En las secciones siguientes veremos con más detalle cómo crear cada parte.

<a name="buildindex"></a>

## <a name="build-the-index"></a>Crear el índice
El uso de facetas se habilita para cada campo en el índice mediante este atributo de índice: `"Facetable": true`.  
Todos los tipos de campo que podrían usarse en la navegación por facetas son `Facetable` de forma predeterminada. Entre estos tipos de campo se incluyen `Edm.String`, `Edm.DateTimeOffset`, y todos los tipos de campo numéricos (básicamente, todos los tipos de campo se pueden usar con facetas excepto `Edm.GeographyPoint`, que no se puede usar en la navegación por facetas). 

Al crear un índice, el procedimiento recomendado para la navegación por facetas desactivar explícitamente el uso de facetas para los campos que nunca deben usarse como facetas.  En concreto, los campos de cadena de valores singleton, tales como un identificador o un nombre de producto, deben establecerse en `"Facetable": false` para impedir su uso accidental (e ineficaz) en una exploración por facetas. Desactivar el uso de facetas donde no es necesario ayuda a mantener un tamaño de índice pequeño y, por lo general, mejora el rendimiento.

El siguiente es parte del esquema de la aplicación de ejemplo de la demostración del portal de búsqueda de trabajo, al que se le han recortado algunos atributos para reducir su tamaño:

```json
{
  ...
  "name": "nycjobs",
  "fields": [
    { “name”: "id",                 "type": "Edm.String",              "searchable": false, "filterable": false, ... "facetable": false, ... },
    { “name”: "job_id",             "type": "Edm.String",              "searchable": false, "filterable": false, ... "facetable": false, ... },
    { “name”: "agency",              "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { “name”: "posting_type",        "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { “name”: "num_of_positions",    "type": "Edm.Int32",              "searchable": false, "filterable": true, ...  "facetable": true, ...  },
    { “name”: "business_title",      "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { “name”: "civil_service_title", "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { “name”: "title_code_no",       "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { “name”: "level",               "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { “name”: "salary_range_from",   "type": "Edm.Int32",              "searchable": false, "filterable": true, ...  "facetable": true, ...  },
    { “name”: "salary_range_to",     "type": "Edm.Int32",              "searchable": false, "filterable": true, ...  "facetable": true, ...  },
    { “name”: "salary_frequency",    "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
    { “name”: "work_location",       "type": "Edm.String",             "searchable": true,  "filterable": true, ...  "facetable": true, ...  },
…
    { “name”: "geo_location",        "type": "Edm.GeographyPoint",     "searchable": false, "filterable": true, ...  "facetable": false, ... },
    { “name”: "tags",                "type": "Collection(Edm.String)", "searchable": true,  "filterable": true, ...  "facetable": true, ...  }
  ],
…
}
```

Como puede observar en el esquema del ejemplo, `Facetable` se ha desactivado para los campos de cadena que no deben usarse como facetas, como los valores de identificador. Desactivar el uso de facetas donde no es necesario ayuda a mantener un tamaño de índice pequeño y, por lo general, mejora el rendimiento.

> [!TIP]
> Como procedimiento recomendado, incluya el conjunto completo de atributos de índice para cada campo. Aunque `Facetable` está activado para casi todos los campos de forma predeterminada, establecer deliberadamente cada atributo puede ayudar a considerar detenidamente las implicaciones de cada decisión del esquema. 

<a name="checkdata"></a>

## <a name="check-the-data"></a>Comprobación de los datos
La calidad de los datos tiene un efecto directo en la posibilidad de que la estructura de navegación por facetas se materialice tal y como se espera. También afecta a la facilidad de creación de filtros para reducir el conjunto de resultados.

Si desea usar la marca o el precio como facetas, cada documento debe contener valores de *BrandName* y *ProductPrice* que sean válidos, coherentes y productivos como opción de filtro.

A continuación tiene algunos recordatorios de lo que se debe pulir:

* Para cada campo que desee usar como faceta, pregúntese si contiene valores que son adecuados como filtros en búsquedas autodirigidas. Los valores deben ser breves, descriptivos y suficientemente distintivos para ofrecer una opción clara entre las opciones de la competencia.
* Errores ortográficos o valores casi coincidentes. Si usa Color como faceta y los valores de campo incluyen Naranja y Nraanja (una palabra incorrecta), una faceta basada en el campo Color seleccionará ambas.
* La mezcla de mayúsculas y minúsculas en el texto también puede causar estragos en la navegación por facetas, porque naranja y Naranja aparecen como dos valores diferentes. 
* Las versiones en singular y plural del mismo valor pueden producir una faceta diferente para cada una.

Como puede imaginar, la diligencia en la preparación de los datos es un aspecto esencial de una navegación por facetas eficiente.

<a name="presentationlayer"></a>

## <a name="build-the-ui"></a>Creación de la interfaz de usuario
Empezar a trabajar desde la capa de presentación puede ayudarle a descubrir requisitos que de otra forma podrían perderse, y comprender las características que son esenciales para la experiencia de búsqueda.

En términos de la navegación por facetas, la página o aplicación web muestra la estructura de la navegación por facetas, detecta la entrada del usuario en la página e inserta los elementos modificados. 

En el caso de las aplicaciones web, normalmente se usa AJAX en la capa de presentación porque permite actualizar los cambios incrementales. También puede usar ASP.NET MVC o cualquier otra plataforma de visualización que puede conectarse a un servicio Búsqueda de Azure a través de HTTP. La aplicación de ejemplo a la que se hace referencia a lo largo de este artículo ( **Azure Search Job Portal Demo** [Demostración del portal de búsqueda de trabajo de Azure Search]) es una aplicación ASP.NET MVC.

En el ejemplo, la navegación por facetas se integra en la página de resultados de búsqueda. El ejemplo siguiente, tomado del archivo `index.cshtml` de la aplicación de ejemplo, muestra una estructura HTML estática para mostrar la navegación por facetas en la página de resultados de búsqueda. La lista de facetas se compila o recompila dinámicamente cuando se envía un término de búsqueda, o cuando se selecciona o borra una faceta.

```html
<div class="widget sidebar-widget jobs-filter-widget">
  <h5 class="widget-title">Filter Results</h5>
    <p id="filterReset"></p>
    <div class="widget-content">

      <h6 id="businessTitleFacetTitle">Business Title</h6>
      <ul class="filter-list" id="business_title_facets">
      </ul>

      <h6>Location</h6>
      <ul class="filter-list" id="posting_type_facets">
      </ul>

      <h6>Posting Type</h6>
      <ul class="filter-list" id="posting_type_facets"></ul>

      <h6>Minimum Salary</h6>
      <ul class="filter-list" id="salary_range_facets">
      </ul>

  </div>
</div>
```

El siguiente fragmento de código desde la página `index.cshtml` compila de forma dinámica el HTML para mostrar la primera faceta, Business Title (cargo empresarial). Funciones similares compilan dinámicamente el HTML para las otras facetas. Cada faceta tiene una etiqueta y un recuento, que muestra el número de elementos encontrados para ese resultado de la faceta.

```js
function UpdateBusinessTitleFacets(data) {
  var facetResultsHTML = '';
  for (var i = 0; i < data.length; i++) {
    facetResultsHTML += '<li><a href="javascript:void(0)" onclick="ChooseBusinessTitleFacet(\'' + data[i].Value + '\');">' + data[i].Value + ' (' + data[i].Count + ')</span></a></li>';
  }

  $("#business_title_facets").html(facetResultsHTML);
}
```

> [!TIP]
> Al diseñar la página de resultados de búsqueda, no olvide agregar un mecanismo para borrar las facetas. Si agrega casillas de verificación, puede ver fácilmente cómo borrar los filtros. En otros diseños, puede que necesite un patrón de ruta de navegación u otro enfoque creativo. Por ejemplo, en la aplicación de ejemplo del portal de búsqueda de trabajo, puede hacer clic en `[X]` después de una faceta seleccionada para borrar la faceta.

<a name="buildquery"></a>

## <a name="build-the-query"></a>Crear la consulta
El código que se escribe para crear consultas debe especificar todas las partes de una consulta válida, incluidas las expresiones de búsqueda, las facetas o los filtros de puntuación (todo lo que se usa para formular una solicitud). En esta sección, exploraremos cómo encajan las facetas en una consulta y cómo se usan los filtros con facetas para entregar un conjunto de resultados reducido.

Observe que las facetas son una parte integral de esta aplicación de ejemplo. La experiencia de búsqueda en la demostración del portal de búsqueda de trabajos está diseñada en torno a los filtros y la navegación por facetas. La colocación destacada de navegación por facetas en la página muestra su importancia. 

Un ejemplo suele ser una buena manera de comenzar. En el ejemplo siguiente, tomado del archivo `JobsSearch.cs`, se crea una solicitud que crea una navegación por facetas basada en Business Title (cargo empresarial), Location (ubicación), Posting Type (tipo de puesto) y Minimum Salary (salario mínimo). 

```cs
SearchParameters sp = new SearchParameters()
{
  ...
  // Add facets
  Facets = new List<String>() { "business_title", "posting_type", "level", "salary_range_from,interval:50000" },
};
```

Un parámetro de consulta de faceta se establece en un campo y, según el tipo de datos, se puede parametrizar aún más con una lista delimitada por comas que incluya `count:<integer>`, `sort:<>`, `interval:<integer>` y `values:<list>`. Se admiten listas de valores para datos numéricos cuando se establecen intervalos. Consulte [Buscar documentos (API de Azure Search)](https://docs.microsoft.com/rest/api/searchservice/Search-Documents) para obtener más información sobre su uso.

Además de las facetas, la solicitud que la aplicación formula también debe generar filtros para reducir el conjunto de documentos candidatos en función de una selección de valores de faceta. Para una tienda de bicicletas, la navegación por facetas proporciona pistas a preguntas como *¿qué colores, fabricantes y tipos de bicicletas están disponibles?*. El filtrado responde a preguntas como *¿qué bicicletas exactas son rojas, bicicletas de montaña, dentro de este precio de intervalo de precios?*. Cuando hace clic en "Red" para indicar que solo se deben mostrar productos de color rojo, la consulta siguiente que la aplicación envía incluye`$filter=Color eq ‘Red’`.

El siguiente fragmento de código de la página `JobsSearch.cs` agrega el cargo empresarial seleccionado al filtro si selecciona un valor de la faceta de Business Title.

```cs
if (businessTitleFacet != "")
  filter = "business_title eq '" + businessTitleFacet + "'";
```

<a name="tips"></a> 

## <a name="tips-and-best-practices"></a>Sugerencias y procedimientos recomendados

### <a name="indexing-tips"></a>Sugerencias de indexación
**Mejora de la eficacia del índice si no utiliza un cuadro de búsqueda**

Si la aplicación usa navegación por facetas exclusivamente (es decir, ningún cuadro de búsqueda), puede marcar el campo como `searchable=false`, `facetable=true` para generar un índice más compacto. Además, la indización se produce solo en los valores de faceta completos, sin saltos de palabras ni indización de los componentes de un valor de varias palabras.

**Especificación de qué campos se pueden usar como facetas**

Recuerde que el esquema del índice determina qué campos están disponibles para usar como facetas. Suponiendo que un campo se pueda usar para la búsqueda por facetas, la consulta especifica qué campos se van a usar para la búsqueda por facetas. El campo que se va a usar para la búsqueda por facetas proporciona los valores que aparecerán debajo de la etiqueta. 

Los valores que aparecen debajo de cada etiqueta se recuperan del índice. Por ejemplo, si el campo de faceta es *Color*, los valores disponibles para un filtrado adicional son los valores de ese campo (Red, Black, etc.).

Solo para los valores Numeric y DateTime, puede establecer explícitamente valores en el campo de faceta (por ejemplo, `facet=Rating,values:1|2|3|4|5`). Se permite una lista de valores para estos tipos de campo para simplificar la separación de los resultados de faceta en intervalos contiguos (intervalos basados en valores numéricos o en períodos de tiempo). 

**De forma predeterminada solo puede tener un nivel de navegación por facetas** 

Como ya se mencionó, no hay compatibilidad directa para facetas anidadas en una jerarquía. De forma predeterminada, la navegación por facetas en Azure Search solo admite un nivel de filtros. Sin embargo, existen soluciones alternativas. Puede codificar una estructura jerárquica de facetas en una `Collection(Edm.String)` con un punto de entrada por jerarquía. La implementación de esta solución queda fuera del ámbito de este artículo. 

### <a name="querying-tips"></a>Sugerencias de consulta
**Validar los campos**

Si compila la lista de facetas de manera dinámica basándose en la entrada de usuarios que no son de confianza, valide que los nombres de los campos con facetas son válidos. O bien, anule los nombres al generar direcciones URL utilizando o bien `Uri.EscapeDataString()` en. NET, o su equivalente en la plataforma que elija.

### <a name="filtering-tips"></a>Sugerencias de filtrado
**Aumento de la precisión de la búsqueda con filtros**

 Use filtros. Si confía solamente en expresiones de búsqueda, la lematización podría provocar que se devuelva un documento que no tiene un valor de faceta preciso en ninguno de sus campos.

**Aumento del rendimiento de la búsqueda con filtros**

 Los filtros restringen el conjunto de documentos candidatos para la búsqueda y los excluyen de la clasificación. Si tiene un conjunto grande de documentos, usar una profundización por facetas selectiva suele proporcionar un mayor rendimiento.
  
**Filtrado solo de campos por facetas**

En la profundización por facetas, normalmente solo debe incluir los documentos que tienen el valor de faceta en un campo (con faceta) específico, no en cualquiera de todos los campos en los que se puede buscar. Agregar un filtro refuerza el campo de destino y dirige el servicio para que busque un valor coincidente solo en el campo con faceta.

**Recorte de los resultados de faceta con más filtros**

Los resultados de faceta son documentos que se encuentran en los resultados de búsqueda y que coinciden con un término de faceta. En el ejemplo siguiente, en los resultados de búsqueda de *cloud computing*, 254 elementos también tienen *internal specification* como tipo de contenido. Los elementos no son necesariamente excluyentes mutuamente. Si un elemento cumple los criterios de ambos filtros, se contabiliza en cada uno de ellos. Esta duplicación es posible cuando se usan facetas en campos `Collection(Edm.String)`, que suelen usarse para implementar el etiquetado de documentos.

        Search term: "cloud computing"
        Content type
           Internal specification (254)
           Video (10) 

En general, si encuentra que los resultados de faceta son demasiado grandes de forma persistente, se recomienda agregar más filtros para ofrecer a los usuarios más opciones para delimitar la búsqueda.

### <a name="tips-about-result-count"></a>Sugerencias sobre el recuento de resultados

**Limitación del número de elementos en la navegación por facetas**

Para cada campo de faceta del árbol de navegación, hay un límite predeterminado de 10 valores. Este valor predeterminado tiene sentido en las estructuras de navegación porque mantiene la lista de valores en un tamaño fácil de administrar. Puede invalidar el valor predeterminado y asignar un valor a count.

* `&facet=city,count:5` especifica que solo se devuelven como resultado de la faceta las cinco primeras ciudades encontradas en los principales resultados. Considere una consulta de ejemplo con un término de búsqueda "aeropuerto" y 32 coincidencias. Si la consulta especifica `&facet=city,count:5`, solo se incluyen en los resultados de la faceta las primeras cinco ciudades con más documentos en los resultados de búsqueda.

Observe la diferencia entre resultados de búsqueda y resultados de faceta. Los resultados de búsqueda son todos los documentos que coinciden con la consulta. Los resultados de faceta son las coincidencias para cada valor de la faceta. En el ejemplo, los resultados de búsqueda incluyen los nombres de ciudades que no están en la lista de clasificación de faceta (5 en nuestro ejemplo). Los resultados que se filtran mediante la navegación por facetas son visibles cuando se borran las facetas o se eligen otras facetas además de City. 

> [!NOTE]
> Analizar `count` cuando hay más de un tipo puede ser confuso. En la tabla siguiente se ofrece un breve resumen de cómo se usa el término en la API de Búsqueda de Azure así como código de ejemplo y documentación. 

* `@colorFacet.count`<br/>
   En el código de presentación, verá un parámetro count en la faceta, que se usa para mostrar el número de resultados de faceta. En los resultados de faceta, count indica el número de documentos que coinciden con el término de faceta o con el intervalo de facetas.
* `&facet=City,count:12`<br/>
   En una consulta de faceta, puede establecer count en un valor.  El valor predeterminado es 10, pero puede establecer uno inferior o superior. Al establecer `count:12` se obtienen las 12 principales coincidencias en los resultados de faceta por recuento de documentos.
* "`@odata.count`"<br/>
   En la respuesta de la consulta, este valor indica el número de elementos coincidentes en los resultados de búsqueda. Por término medio, es mayor que la suma de todos los resultados de faceta combinados, debido a la presencia de elementos que coinciden con el término de búsqueda pero con ninguno de los valores de faceta.

**Obtención de recuentos en los resultados de faceta**

Al agregar un filtro a una consulta por facetas, quizás quiera conservar la instrucción facet (por ejemplo, `facet=Rating&$filter=Rating ge 4`). Técnicamente, facet=Rating no es necesaria, pero conservarla devuelve los recuentos de los valores de faceta para las valoraciones de 4 y superiores. Por ejemplo, si hace clic en "4" y la consulta incluye un filtro para valores mayores o iguales que "4", se devuelven los recuentos de valoraciones de 4 y superiores.  

**Garantía de obtención de recuentos de faceta adecuados**

En determinadas circunstancias, puede que vea que los recuentos de faceta no coinciden con los conjuntos de resultados (consulte [Navegación por facetas en Búsqueda de Azure (publicación del foro)](https://social.msdn.microsoft.com/Forums/azure/06461173-ea26-4e6a-9545-fbbd7ee61c8f/faceting-on-azure-search?forum=azuresearch)).

Los recuentos de faceta pueden ser incorrectos debido a la arquitectura de particionamiento. Cada índice de búsqueda tiene varias particiones, y cada una notifica las N primeras facetas por recuento de documentos, que después se combina en un único resultado. Si algunas particiones tienen muchos valores coincidentes, mientras que otros tienen menos, verá que algunos valores de faceta faltan o se contabilizan con un número inferior en los resultados.

Aunque este comportamiento podría cambiar en cualquier momento, si se produce este comportamiento hoy en día, puede trabajar a su alrededor infla artificialmente el recuento:\<número > para un número grande para forzar reporting completa de cada partición. Si el valor de count: es mayor o igual que el número de valores únicos en el campo, se garantizan resultados precisos. Sin embargo, si el recuento de documentos es alto, esto afecta al rendimiento, por lo que debe usar esta opción con prudencia.

### <a name="user-interface-tips"></a>Sugerencias de interfaz de usuario
**Agregar etiquetas para cada campo en la navegación por facetas**

Las etiquetas suelen definirse en el HTML o formulario (`index.cshtml` en la aplicación de ejemplo). No hay ninguna API en Azure Search para las etiquetas de navegación por facetas u otros tipos de metadatos.

<a name="rangefacets"></a>

## <a name="filter-based-on-a-range"></a>Filtro basado en un intervalo
El uso de facetas en intervalos de valores es un requisito habitual de las aplicaciones de búsqueda. Se admiten intervalos para datos numéricos y valores de fecha y hora. Puede leer más acerca de cada enfoque en [Buscar documentos (API de Azure Search)](https://docs.microsoft.com/rest/api/searchservice/Search-Documents).

Búsqueda de Azure simplifica la construcción de intervalos mediante dos enfoques de cálculo de intervalos. En ambos enfoques, Búsqueda de Azure crea los intervalos adecuados con las entradas proporcionadas. Por ejemplo, si especifica valores de intervalo de 10|20|30, creará automáticamente los intervalos 0-10, 10-20, 20-30. La aplicación tiene la opción de quitar los intervalos que estén vacíos. 

**Enfoque 1: Usar el parámetro de intervalo**  
Para establecer las facetas de precio en incrementos de 10 USD, especificaría: `&facet=price,interval:10`

**Enfoque 2: Usar una lista de valores**  
 Para datos numéricos, puede usar una lista de valores.  Tenga en cuenta el intervalo de facetas para un campo `listPrice`, que se representa como sigue:

  ![Lista de valores de ejemplo](media/search-faceted-navigation/Facet-5-Prices.PNG "lista de valores de ejemplo")

Para especificar un intervalo de faceta semejante al de la captura de pantalla anterior, use una lista de valores:

    facet=listPrice,values:10|25|100|500|1000|2500

Cada intervalo se genera usando 0 como punto de partida, un valor de la lista como extremo y, después, se recorta el intervalo anterior para crear diferentes intervalos. Azure Search hace esto como parte de la navegación por facetas. No es necesario escribir código para estructurar cada intervalo.

### <a name="build-a-filter-for-a-range"></a>Creación de un filtro para un intervalo
Para filtrar los documentos según un intervalo que seleccione, puede usar los operadores de filtro `"ge"` y `"lt"` en una expresión de dos partes que define los puntos de conexión del intervalo. Por ejemplo, si elige el intervalo de 10 a 25 para un campo `listPrice`, el filtro sería `$filter=listPrice ge 10 and listPrice lt 25`. En el código de ejemplo, la expresión de filtro usa los parámetros **priceFrom** y **priceTo** para establecer los puntos de conexión. 

  ![Consulta para un intervalo de valores](media/search-faceted-navigation/Facet-6-buildfilter.PNG "consulta para un intervalo de valores")

<a name="geofacets"></a> 

## <a name="filter-based-on-distance"></a>Filtro basado en la distancia
Es habitual ver filtros que ayudan a elegir una tienda, un restaurante o un destino en función de su proximidad a la ubicación actual. Aunque este tipo de filtro puede parecer una navegación por facetas, en realidad es solo un filtro. Lo mencionamos aquí para aquellos que buscan específicamente consejos de implementación para ese problema de diseño concreto.

Hay dos funciones geoespaciales en Azure Search, **geo.distance** y **geo.intersects**.

* La función **geo.distance** devuelve la distancia en kilómetros entre dos puntos. Un punto es un campo y otra es una constante que se pasa como parte del filtro. 
* La función **geo.intersects** devuelve true si un punto determinado se encuentra dentro de un polígono determinado. El punto es un campo y el polígono se especifica como una lista constante de coordenadas que se pasa como parte del filtro.

Puede encontrar ejemplos de filtros en [Sintaxis de expresiones de OData (Búsqueda de Azure)](query-odata-filter-orderby-syntax.md).

<a name="tryitout"></a>

## <a name="try-the-demo"></a>Prueba de la demostración
La demostración de Azure Search Job Portal contiene los ejemplos a los que se hace referencia en este artículo.

-   Vea y pruebe la demostración de trabajo en línea en [Azure Search Job Portal Demo](https://azjobsdemo.azurewebsites.net/) (Demostración del portal de búsqueda de trabajo de Azure Search).

-   Descargue el código del [repositorio de ejemplos de Azure en GitHub](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs).

Cuando trabaje con los resultados de búsqueda, vigile los posibles cambios en las direcciones URL en la construcción de la consulta. Esta aplicación anexa facetas al URI a medida que las selecciona.

1. Para usar la funcionalidad de asignación de la aplicación de demostración, obtenga una clave de Bing Maps en [Bing Maps Dev Center](https://www.bingmapsportal.com/) (Centro para desarrolladores de Bing Maps). Pegue sobre la clave existente en la página `index.cshtml`. El ajuste `BingApiKey` en el archivo `Web.config` no se utiliza. 

2. Ejecute la aplicación. Dese el paseo opcional o descarte el cuadro de diálogo.
   
3. Escriba un término de búsqueda, como "analyst" y haga clic en el icono de búsqueda. La consulta se ejecuta rápidamente.
   
   Con los resultados de búsqueda se devuelve también una estructura de navegación por facetas. En la página de resultados de búsqueda, la estructura de navegación por facetas incluye los recuentos de cada resultado de faceta. No hay facetas seleccionadas, por lo que se devuelven todos los resultados de búsqueda que coinciden.
   
   ![Los resultados de búsqueda antes de seleccionar las facetas](media/search-faceted-navigation/faceted-search-before-facets.png "los resultados de búsqueda antes de seleccionar las facetas")

4. Haga clic en un Business Title (cargo empresarial), Location (ubicación) o Minimum Salary (salario mínimo). Las facetas son NULL en la búsqueda inicial, pero a medida que toman valores, en los resultados de búsqueda se recortan los elementos que ya no coinciden.
   
   ![Los resultados de búsqueda después de seleccionar las facetas](media/search-faceted-navigation/faceted-search-after-facets.png "los resultados de búsqueda después de seleccionar las facetas")

5. Para borrar la consulta por facetas para poder probar comportamientos de consulta diferentes, haga clic en `[X]` después de las facetas seleccionadas para borrar las facetas.
   
<a name="nextstep"></a>

## <a name="learn-more"></a>Más información
Vea el vídeo de [profundización en Azure Search](https://channel9.msdn.com/Events/TechEd/Europe/2014/DBI-B410). En el minuto 45:25, hay una demostración sobre cómo implementar las facetas.

Para obtener más información sobre los principios de diseño de la navegación por facetas, recomendamos los siguientes vínculos:

* [Patrones de diseño: Navegación por facetas](https://alistapart.com/article/design-patterns-faceted-navigation)
* [Problemas de Front-End al implementar la búsqueda por facetas – parte 1 ](https://articles.uie.com/faceted_search2/)

