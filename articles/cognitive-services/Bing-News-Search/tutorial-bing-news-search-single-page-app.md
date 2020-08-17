---
title: 'Tutorial: Crear una aplicación web de una sola página con Bing News Search API'
titleSuffix: Azure Cognitive Services
description: Use este tutorial para crear una aplicación web de una sola página que pueda enviar consultas a Bing News API y mostrar los resultados en la página web.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-news-search
ms.topic: tutorial
ms.date: 06/23/2020
ms.author: aahi
ms.custom: seodec2018, devx-track-javascript
ms.openlocfilehash: 2391b63cbc0539b9c2e505e8cf531aeb78eb9bfd
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406085"
---
# <a name="tutorial-create-a-single-page-web-app"></a>Tutorial: Creación de una aplicación web de una sola página

Bing News Search API permite buscar en la Web y obtener resultados relacionados con las noticias y que sean relevantes para una consulta de búsqueda. En este tutorial, crearemos una aplicación web de una sola página que usa Bing News Search API para mostrar los resultados de la búsqueda directamente en la página. La aplicación incluye componentes HTML, CSS y JavaScript. El código fuente del ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/Tutorials/BingNewsSearchApp.html).

<!-- Remove until we can replace it with sanitized copy
![Single-page Bing News Search app](media/news-search-singlepage.png)
-->

> [!NOTE]
> Los encabezados HTTP y JSON de la parte inferior de la página revelan la respuesta JSON y la información de la solicitud HTTP al hacer clic en ellos. Estos detalles resultan útiles al explorar el servicio.

En la aplicación del tutorial se muestra cómo:
> [!div class="checklist"]
> * Realizar una llamada a Bing News Search API en JavaScript.
> * Pasar opciones de búsqueda a Bing News Search API.
> * Mostar los resultados de búsqueda de noticias de cuatro categorías: de cualquier tipo, negocios, salud o política, a partir períodos 24 horas, la semana pasada, el mes pasado o todo el tiempo disponible.
> * Navegar por las páginas de resultados de la búsqueda.
> * Administrar el identificador de cliente de Bing y la clave de suscripción de la API
> * Controlar los errores que se puedan producir.

La página del tutorial es completamente independiente: no utiliza marcos, hojas de estilo ni archivos de imagen externos. Usa solo características del lenguaje JavaScript ampliamente compatibles y funciona con las versiones actuales de los principales exploradores web.


## <a name="prerequisites"></a>Requisitos previos

Para seguir con el tutorial, necesita claves de suscripción para Bing Search API. Si no las tiene, debe crearlas:

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="cree un recurso de Bing Search"  target="_blank">crear un recurso de Bing Search<span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.

## <a name="app-components"></a>Componentes de la aplicación
Igual que sucede con cualquier aplicación web de una sola página, la aplicación del tutorial incluye tres partes:

> [!div class="checklist"]
> * HTML: define la estructura y el contenido de la página
> * CSS: define el aspecto de la página
> * JavaScript: define el comportamiento de la página

La mayor parte del código HTML y CSS es convencional, por lo que el tutorial no lo abarca. El código HTML contiene el formulario de búsqueda en que el usuario escribe una consulta y elige las opciones de búsqueda. El formulario está conectado al código de JavaScript que realmente realiza la búsqueda mediante el atributo `onsubmit` de la etiqueta `<form>`:

```html
<form name="bing" onsubmit="return newBingNewsSearch(this)">
```
El controlador `onsubmit` devuelve `false`, lo que impide que el formulario que se envíe a un servidor. El código de JavaScript se encarga de recopilar la información necesaria del formulario y realizar la búsqueda.

El HTML también contiene las divisiones (etiquetas `<div>` HTML) en que aparecen los resultados de la búsqueda.

## <a name="managing-subscription-key"></a>Administrar la clave de suscripción

Para evitar tener que incluir la clave de suscripción de Bing Search API en el código, usamos el almacenamiento persistente del explorador para almacenar la clave. Antes de almacenar la clave, se solicitará la clave de usuario. Si, más tarde, la API rechaza la clave, se invalidará la clave almacenada y se le volverá a pedir al usuario.

Definimos las funciones `storeValue` y `retrieveValue` que usan el objeto `localStorage` (no todos los exploradores lo admite) o una cookie. La función `getSubscriptionKey()` usa estas funciones para almacenar y recuperar la clave del usuario. Puede usar el punto de conexión global siguiente o el punto de conexión del [subdominio personalizado](../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

``` javascript
// Cookie names for data we store
API_KEY_COOKIE   = "bing-search-api-key";
CLIENT_ID_COOKIE = "bing-search-client-id";

// Bing Search API endpoint
BING_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/news";

// ... omitted definitions of storeValue() and retrieveValue()
// Browsers differ in their support for persistent storage by 
// local HTML files. See the source code for browser-specific
// options.

// Get stored API subscription key, or 
// prompt if it's not found.
function getSubscriptionKey() {
    var key = retrieveValue(API_KEY_COOKIE);
    while (key.length !== 32) {
        key = prompt("Enter Bing Search API subscription key:", "").trim();
    }
    // always set the cookie in order to update the expiration date
    storeValue(API_KEY_COOKIE, key);
    return key;
}
```
La etiqueta `<form>` HTML `onsubmit` llama a la función `bingWebSearch` para devolver los resultados de la búsqueda. `bingWebSearch` utiliza `getSubscriptionKey()` para autenticar cada consulta. Tal como se muestra en la definición anterior, si no se ha especificado la clave `getSubscriptionKey` se la pide al usuario. A continuación, la clave se almacena para que la siga usando la aplicación.

```html
<form name="bing" onsubmit="this.offset.value = 0; return bingWebSearch(this.query.value, 
    bingSearchOptions(this), getSubscriptionKey())">
```
## <a name="selecting-search-options"></a>Selección de las opciones de búsqueda
En la siguiente ilustración se muestra el cuadro de texto de la consulta y las opciones que definen una búsqueda de noticias acerca de fondos escolares.

![Opciones de Bing News Search](media/news-search-categories.png)

El formulario HTML incluye elementos con los nombres siguientes:

|Elemento|Descripción|
|-|-|
| `where` | Un menú desplegable para seleccionar el mercado (ubicación e idioma) que se usa en la búsqueda. |
| `query` | El campo de texto en el que se especifican los términos de búsqueda. |
| `category` | Las casillas para la promoción de determinados tipos de resultados. Por ejemplo, al promocionar la salud, aumenta la clasificación de las noticias relacionadas con la salud. |
| `when` | Menú desplegable para, opcionalmente, limitar la búsqueda al día, la semana o el mes más reciente. |
| `safe` | Una casilla que indica si se usa la característica Búsqueda segura de Bing para filtrar los resultados "para adultos". |
| `count` | Campo oculto. Número de resultados de búsqueda que se devolverán en cada solicitud. Cambie este valor para mostrar más o menos resultados por página. |
| `offset`|  Campo oculto. El desplazamiento del primer resultado de búsqueda de la solicitud; se usa para la paginación. Se restablece a `0` en cada nueva solicitud. |

> [!NOTE]
> Bing Web Search ofrece muchos más parámetros de consulta. Aquí solo vamos a usar algunos.

``` javascript
// build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "off"));
    if (form.when.value.length) options.push("freshness=" + form.when.value);

    for (var i = 0; i < form.category.length; i++) {
        if (form.category[i].checked) {
            category = form.category[i].value;
            break;
        }
    }
    if (category.valueOf() != "all".valueOf()) { 
        options.push("category=" + category); 
        }
    options.push("count=" + form.count.value);
    options.push("offset=" + form.offset.value);
    return options.join("&");
}
```

Por ejemplo, el parámetro `SafeSearch` de una llamada API real puede ser `strict`, `moderate` o `off`, pero `moderate` es el valor predeterminado. Sin embargo, nuestro formulario usa una casilla que solo tiene dos estados. El código JavaScript convierte este valor en `strict` u `off` (`moderate` no se usa).

## <a name="performing-the-request"></a>Realizar la solicitud
Dada la consulta, la cadena de opciones y la clave de API, la función `BingNewsSearch` utiliza un objeto `XMLHttpRequest` para enviar la solicitud al punto de conexión de Bing News Search.

```javascript
// perform a search given query, options string, and API key
function bingNewsSearch(query, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("results", "related", "_json", "_http", "paging1", "paging2", "error");

    var request = new XMLHttpRequest();
     if (category.valueOf() != "all".valueOf()) {
        var queryurl = BING_ENDPOINT + "/search?" + "?q=" + encodeURIComponent(query) + "&" + options;
    }
    else
    {
        if (query){
        var queryurl = BING_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;
        }
        else {
            var queryurl = BING_ENDPOINT + "?" + options;
        }
    }

    // open the request
    try {
        request.open("GET", queryurl);
    } 
    catch (e) {
        renderErrorMessage("Bad request (invalid URL)\n" + queryurl);
        return false;
    }

    // add request headers
    request.setRequestHeader("Ocp-Apim-Subscription-Key", key);
    request.setRequestHeader("Accept", "application/json");
    var clientid = retrieveValue(CLIENT_ID_COOKIE);
    if (clientid) request.setRequestHeader("X-MSEdge-ClientID", clientid);

    // event handler for successful response
    request.addEventListener("load", handleBingResponse);

    // event handler for erorrs
    request.addEventListener("error", function() {
        renderErrorMessage("Error completing request");
    });

    // event handler for aborted request
    request.addEventListener("abort", function() {
        renderErrorMessage("Request aborted");
    });

    // send the request
    request.send();
    return false;
}
```
Tras la correcta finalización de la solicitud HTTP, JavaScript llama a nuestro controlador de eventos `load`, la función `handleBingResponse()`, para controlar una solicitud GET HTTP correcta a la API. 

```javascript
// handle Bing search request results
function handleBingResponse() {
    hideDivs("noresults");

    var json = this.responseText.trim();
    var jsobj = {};

    // try to parse JSON results
    try {
        if (json.length) jsobj = JSON.parse(json);
    } catch(e) {
        renderErrorMessage("Invalid JSON response");
    }

    // show raw JSON and HTTP request
    showDiv("json", preFormat(JSON.stringify(jsobj, null, 2)));
    showDiv("http", preFormat("GET " + this.responseURL + "\n\nStatus: " + this.status + " " + 
        this.statusText + "\n" + this.getAllResponseHeaders()));

    // if HTTP response is 200 OK, try to render search results
    if (this.status === 200) {
        var clientid = this.getResponseHeader("X-MSEdge-ClientID");
        if (clientid) retrieveValue(CLIENT_ID_COOKIE, clientid);
        if (json.length) {
            if (jsobj._type === "News") {
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    }

    // Any other HTTP response is an error
    else {
        // 401 is unauthorized; force re-prompt for API key for next request
        if (this.status === 401) invalidateSubscriptionKey();

        // some error responses don't have a top-level errors object, so gin one up
        var errors = jsobj.errors || [jsobj];
        var errmsg = [];

        // display HTTP status code
        errmsg.push("HTTP Status " + this.status + " " + this.statusText + "\n");

        // add all fields from all error responses
        for (var i = 0; i < errors.length; i++) {
            if (i) errmsg.push("\n");
            for (var k in errors[i]) errmsg.push(k + ": " + errors[i][k]);
        }

        // also display Bing Trace ID if it isn't blocked by CORS
        var traceid = this.getResponseHeader("BingAPIs-TraceId");
        if (traceid) errmsg.push("\nTrace ID " + traceid);

        // and display the error message
        renderErrorMessage(errmsg.join("\n"));
    }
}
```

> [!IMPORTANT]
> Una solicitud HTTP correcta *no* indica necesariamente que la búsqueda se haya realizado correctamente. Si se produce un error en la operación de búsqueda, Bing News Search API devuelve un código de estado HTTP distinto de 200 e incluye información del error en la respuesta JSON. Además, si la solicitud tenía limitación de frecuencia, la API devuelve una respuesta vacía.

Gran parte del código de las dos funciones anteriores está dedicado al control de errores. Pueden producirse errores en las siguientes fases:

|Fase|Errores posibles|Controlado por|
|-|-|-|
|Compilar un objeto de solicitud de JavaScript|Dirección URL no válida|Bloqueo `try`/`catch`|
|Hacer la solicitud|Errores de red, conexiones anuladas|Controladores de eventos `error` y `abort`|
|Realizar la búsqueda|Solicitud no válida, JSON no válido, límites de frecuencia|Pruebas en el controlador de eventos `load`|

Los errores se controlan mediante una llamada a `renderErrorMessage()` con los detalles que se conocen sobre el error. Si la respuesta pasa todas las pruebas de error, llamamos a `renderSearchResults()` para mostrar los resultados de la búsqueda en la página.

## <a name="displaying-search-results"></a>Presentación de los resultados de la búsqueda
La función principal para mostrar los resultados de búsqueda es `renderSearchResults()`. Esta función utiliza el valor de JSON que devuelve el servicio Bing News Search y procesa los resultados de noticias y las búsquedas relacionadas, si los hay.

```javascript
// render the search results given the parsed JSON response
    function renderSearchResults(results) {

    // add Prev / Next links with result count
    var pagingLinks = renderPagingLinks(results);
    showDiv("paging1", pagingLinks);
    showDiv("paging2", pagingLinks);

    showDiv("results", renderResults(results.value));
    if (results.relatedSearches)
        showDiv("sidebar", renderRelatedItems(results.relatedSearches));
}
```
Se devuelven los resultados de búsqueda principal como el objeto `value` de nivel superior de la respuesta JSON. Los pasamos a nuestra función `renderResults()`, que los itera y llama a otra función para representar cada elemento en HTML. El HTML resultante se devuelve a `renderSearchResults()`, donde se inserta en la división `results` de la página.

```javascript
function renderResults(items) {
    var len = items.length;
    var html = [];
    if (!len) {
        showDiv("noresults", "No results.");
        hideDivs("paging1", "paging2");
        return "";
    }
    for (var i = 0; i < len; i++) {
        html.push(searchItemRenderers.news(items[i], i, len));
    }
    return html.join("\n\n");
}
```
Bing News Search API devuelve hasta cuatro tipos distintos de resultados relacionados, cada uno en su propio objeto de nivel superior. Son las siguientes:

|Relación|Descripción|
|-|-|
|`pivotSuggestions`|Consultas que reemplazan una palabra dinámica de la búsqueda original por otra. Por ejemplo, si busca "flores rojas", una palabra dinámica podría ser "rojas", y una sugerencia dinámica podría ser "flores amarillas".|
|`queryExpansions`|Consultas que limitan la búsqueda original al agregar más términos. Por ejemplo, si busca "Microsoft Surface", una expansión de consulta podría ser "Microsoft Surface Pro".|
|`relatedSearches`|Consultas que también han realizado otros usuarios que escribieron la búsqueda original. Por ejemplo, su buscó "Monte Rainier", una búsqueda relacionada podría ser "Monte Santa Elena".|
|`similarTerms`|Consultas que son similares en significado a la búsqueda original. Por ejemplo, si busca "escuelas", un término similar podría ser "educación".|

Como ya hemos visto en `renderSearchResults()`, solo representamos sugerencias de `relatedItems` y colocamos los vínculos resultantes en la barra lateral de la página.

## <a name="rendering-result-items"></a>Representación de los elementos del resultado

En el código de JavaScript, `searchItemRenderers` es un objeto que contiene *representadores:* esto es, funciones que generan código HTML para cada tipo de resultado de búsqueda.

```javascript
searchItemRenderers = {
    news: function(item) { ... },
    webPages: function (item) { ... }, 
    images: function(item, index, count) { ... },
    relatedSearches: function(item) { ... }
}
```
Una función de representador puede aceptar los parámetros siguientes:

|Parámetro|Descripción|
|-|-|
|`item`| El objeto de JavaScript que contiene las propiedades del elemento, como su dirección URL y la descripción.|
|`index`| El índice del elemento de resultado dentro de su colección.|
|`count`| El número de elementos de la colección del elemento de resultado de la búsqueda.|

Los parámetros `index` y `count` pueden usarse para numerar los resultados, para generar un código HTML especial para el principio o el final de una colección, para insertar saltos de línea después de cierto número de elementos, etc. Si un representador no necesita esta funcionalidad, no es necesario aceptar estos dos parámetros.

El representador `news` se muestra en el siguiente extracto de JavaScript.
```javascript
    // render news story
    news: function (item) {
        var html = [];
        html.push("<p class='news'>");
        if (item.image) {
            width = 60;
            height = Math.round(width * item.image.thumbnail.height / item.image.thumbnail.width);
            html.push("<img src='" + item.image.thumbnail.contentUrl +
                "&h=" + height + "&w=" + width + "' width=" + width + " height=" + height + ">");
        }
        html.push("<a href='" + item.url + "'>" + item.name + "</a>");
        if (item.category) html.push(" - " + item.category);
        if (item.contractualRules) {    // MUST display source attributions
            html.push(" (");
            var rules = [];
            for (var i = 0; i < item.contractualRules.length; i++)
                rules.push(item.contractualRules[i].text);
                html.push(rules.join(", "));
                html.push(")");
            }
        html.push(" (" + getHost(item.url) + ")");
        html.push("<br>" + item.description);
        return html.join("");
    },
```
Función del representador de noticias:
> [!div class="checklist"]
> * Crea una etiqueta de párrafo, la asigna a la clase `news` y la inserta en la matriz HTML.
> * Calcula el tamaño de la miniatura de la imagen (el ancho se fija en 60 píxeles, y el alto se calcula proporcionalmente).
> * Crea la etiqueta HTML `<img>` para mostrar la miniatura de la imagen. 
> * Compila etiquetas `<a>` HTML vinculadas con la imagen y la página que la contiene.
> * Compila la descripción que muestra información sobre la imagen y el sitio en que se encuentra.

El tamaño de las miniaturas se usa tanto en la etiqueta `<img>` como en los campos `h` y `w` en la dirección URL de la miniatura. El [servicio de miniaturas de Bing](../bing-web-search/resize-and-crop-thumbnails.md), a continuación, proporciona una miniatura de exactamente ese tamaño.

## <a name="persisting-client-id"></a>Identificador de cliente persistente
Las respuestas de Bing Search API pueden incluir un encabezado `X-MSEdge-ClientID` que debe devolverse a la API con las solicitudes sucesivas. Si se usan varias instancias de Bing Search API, se debe usar el mismo identificador de cliente con todas ellas, si es posible.

Especificar el encabezado `X-MSEdge-ClientID` permite a las API de Bing asociar todas las búsquedas de un usuario, lo que tiene dos ventajas importantes.

En primer lugar, permite al motor de búsqueda de Bing aplicar un contexto pasado a las búsquedas, con el fin de encontrar los resultados que más satisfagan al usuario. Si un usuario ha buscado previamente términos relacionados con la navegación, por ejemplo, una búsqueda posterior de "nudos" podría devolver información acerca de los nudos que se usan en la navegación de forma preferente.

En segundo lugar, Bing puede seleccionar aleatoriamente usuarios para disfrutar de nuevas características antes de que estén disponibles públicamente. Especificar el mismo identificador de cliente en todas las solicitudes garantiza que los usuarios que ven una característica la vean siempre. Sin el identificador de cliente, el usuario puede ver una característica aparecer y desaparecer, de forma aparentemente aleatoria, en los resultados de búsqueda.

Las directivas de seguridad del explorador (CORS) pueden impedir que el encabezado `X-MSEdge-ClientID` esté disponible para JavaScript. Esta limitación tiene lugar cuando la respuesta a la búsqueda tiene un origen distinto al de la página que la solicitó. En un entorno de producción, debería abordar esta directiva mediante el hospedaje de un script de lado servidor que realice la llamada API en el mismo dominio que la página web. Como el script tiene el mismo origen que la página web, el encabezado `X-MSEdge-ClientID` está disponible para JavaScript.

> [!NOTE]
> En una aplicación web de producción, debe realizar la solicitud del lado servidor. En caso contrario, es preciso incluir la clave de Bing Search API en la página web, donde está disponible para cualquiera que vea el origen. Se le facturará todo el uso bajo su clave de suscripción a API, incluso las solicitudes que realicen partes no autorizadas, por lo que es importante no exponer su clave.

Para fines de desarrollo, puede realizar la solicitud de Bing Web Search API a través de un proxy CORS. La respuesta de un proxy de este tipo tiene un encabezado `Access-Control-Expose-Headers` que permite los encabezados de respuesta y hace que estén disponibles para JavaScript.

Es fácil instalar un proxy CORS para permitir que nuestra aplicación de tutorial acceda al encabezado de identificador de cliente. En primer lugar, si aún no lo tiene, [instale Node.js](https://nodejs.org/en/download/). Escriba el comando siguiente en una ventana de comandos:

```console
npm install -g cors-proxy-server
```

A continuación, cambie el punto de conexión de Bing Web Search en el archivo HTML a:\
`http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/search`

Por último, inicie el proxy CORS con el siguiente comando:

```console
cors-proxy-server
```

Deje abierta la ventana de comandos mientras usa la aplicación del tutorial, ya que si la cierra, se detendrá el proxy. En la sección de encabezados HTTP expandibles situada bajo los resultados de la búsqueda, puede ver el encabezado `X-MSEdge-ClientID` (entre otras cosas) y comprobar que es el mismo en todas las solicitudes.

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [Referencia de Bing News Search API](//docs.microsoft.com/rest/api/cognitiveservices/bing-news-api-v7-reference)
