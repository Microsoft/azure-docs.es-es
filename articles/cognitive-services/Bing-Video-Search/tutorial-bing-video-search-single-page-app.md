---
title: 'Tutorial: Compilación de una aplicación de Bing Video Search de página única'
titleSuffix: Azure Cognitive Services
description: En este tutorial se indica cómo utilizar Bing Video Search API en una aplicación web de una sola página.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-video-search
ms.topic: tutorial
ms.date: 02/03/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: c3515d6dc7556995c990180330212bffb1445ad2
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406238"
---
# <a name="tutorial-single-page-video-search-app"></a>Tutorial: Aplicación de Video Search de página única
Bing Video Search API permite buscar en la Web y obtener resultados de vídeo correspondientes a la consulta de búsqueda. En este tutorial, crearemos una aplicación web de una sola página que usa Bing Video Search API para mostrar los resultados de la búsqueda en la página. La aplicación incluye componentes HTML, CSS y JavaScript.

<!-- Remove until it can be replaced with a sanitized version.
![Single-page Bing Video Search app](./media/video-search-singlepage.png)
-->

> [!NOTE]
> Los encabezados HTTP y JSON de la parte inferior de la página revelan la respuesta JSON y la información de la solicitud HTTP al hacer clic en ellos. Estos detalles resultan útiles al explorar el servicio.

![Resultados JSON, HTTP sin formato](./media/json-http-raw-results.png)

En esta aplicación de tutorial se muestra cómo:
> [!div class="checklist"]
> * Realizar una llamada a Bing Video Search API en JavaScript.
> * Pasar opciones de búsqueda a Bing Search API
> * Mostrar resultados de búsqueda de vídeo u, opcionalmente, incluir páginas web, noticias o imágenes
> * Buscar en intervalos de tiempo de búsqueda de 24 horas, la semana o el mes pasados o todo el tiempo disponible
> * Navegar por las páginas de resultados de la búsqueda.
> * Administrar el identificador de cliente de Bing y la clave de suscripción de la API
> * Controlar los errores que se puedan producir.

La página del tutorial es completamente independiente: no utiliza marcos, hojas de estilo ni archivos de imagen externos. Usa solo características del lenguaje JavaScript ampliamente compatibles y funciona con las versiones actuales de los principales exploradores web.

En este tutorial, solo se describen determinadas partes del código fuente. Tiene disponible todo el [código fuente](tutorial-bing-video-search-single-page-app-source.md). Para ejecutar el ejemplo, copie y pegue el código fuente en un editor de texto y guárdelo como `bing.html`.

## <a name="app-components"></a>Componentes de la aplicación
Igual que sucede con cualquier aplicación web de una sola página, la aplicación del tutorial incluye tres partes:

> [!div class="checklist"]
> * HTML: define la estructura y el contenido de la página
> * CSS: define el aspecto de la página
> * JavaScript: define el comportamiento de la página

La mayor parte del código HTML y CSS es convencional, por lo que el tutorial no lo abarca. El código HTML contiene el formulario de búsqueda en que el usuario escribe una consulta y elige las opciones de búsqueda. El formulario está conectado al código JavaScript que realiza la búsqueda mediante el atributo `onsubmit` de la etiqueta `<form>`:

```html
<form name="bing" onsubmit="return bingWebSearch(this)">
```
El controlador `onsubmit` devuelve `false`, lo que impide que el formulario que se envíe a un servidor. El código de JavaScript se encarga de recopilar la información necesaria del formulario y realizar la búsqueda.

El HTML también contiene las divisiones (etiquetas `<div>` HTML) en que aparecen los resultados de la búsqueda.

## <a name="managing-subscription-key"></a>Administrar la clave de suscripción

Para evitar tener que incluir la clave de suscripción de Bing Search API en el código, usamos el almacenamiento persistente del explorador para almacenar la clave. Antes de almacenar la clave, se solicitará la clave de usuario. Si, más tarde, la API rechaza la clave, se invalidará la clave almacenada y se le volverá a pedir al usuario.

Definimos las funciones `storeValue` y `retrieveValue` que usan el objeto `localStorage` (no todos los exploradores lo admite) o una cookie. La función `getSubscriptionKey()` usa estas funciones para almacenar y recuperar la clave del usuario.

``` javascript
// Cookie names for data we store
API_KEY_COOKIE   = "bing-search-api-key";
CLIENT_ID_COOKIE = "bing-search-client-id";

// ... omitted definitions of store value and retrieve value
// Browsers differ in their support for persistent storage by 
// local HTML files. See the source code for browser-specific
// options.

// Get stored API subscription key, or prompt if it's not found.
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
En la siguiente ilustración se muestra el cuadro de texto de la consulta y las opciones que definen una búsqueda.

![Opciones de Bing News Search](media/video-search-options.png)

El formulario HTML incluye elementos con los nombres siguientes:

|Elemento|Descripción|
|-|-|
| `where` | Un menú desplegable para seleccionar el mercado (ubicación e idioma) que se usa en la búsqueda. |
| `query` | El campo de texto en el que se especifican los términos de búsqueda. |
| `modules` | Casillas de verificación para promocionar módulos concretos de resultados, todos los resultados o vídeos relacionados. |
| `when` | Menú desplegable para, opcionalmente, limitar la búsqueda al día, la semana o el mes más reciente. |
| `safe` | Una casilla que indica si se usa la característica Búsqueda segura de Bing para filtrar los resultados "para adultos". |
| `count` | Campo oculto. Número de resultados de búsqueda que se devolverán en cada solicitud. Cambie este valor para mostrar más o menos resultados por página. |
| `offset`|  Campo oculto. El desplazamiento del primer resultado de búsqueda de la solicitud; se usa para la paginación. Se restablece a `0` en cada nueva solicitud. |

> [!NOTE]
> Bing Web Search ofrece muchos más parámetros de consulta. Aquí solo vamos a usar algunos.

``` javascript
// build query options from the HTML form
// build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "moderate"));

    if (form.when.value.length) options.push("freshness=" + form.when.value);
    var what = [];
    for (var i = 0; i < form.what.length; i++) 
        if (form.what[i].checked) what.push(form.what[i].value);
    if (what.length) {
        options.push("modules=" + what.join(","));
        options.push("answerCount=9");
    }
    options.push("count=" + form.count.value);
    options.push("offset=" + form.offset.value);
    options.push("textDecorations=true");
    options.push("textFormat=HTML");
    return options.join("&");
}
```

Por ejemplo, el parámetro `SafeSearch` de una llamada API real puede ser `strict` o `moderate`, pero `moderate` es el valor predeterminado.

## <a name="performing-the-request"></a>Realizar la solicitud
Dada la consulta, la cadena de opciones y la clave de API, la función `BingWebSearch` utiliza un objeto `XMLHttpRequest` para enviar la solicitud al punto de conexión de Bing Search. Puede usar el punto de conexión global siguiente o el punto de conexión del [subdominio personalizado](../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

```javascript
// Search on the query, using search options, authenticated by the key.
function bingWebSearch(query, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("pole", "mainline", "sidebar", "_json", "_headers", "paging1", "paging2", "error");

    var endpoint = "https://api.cognitive.microsoft.com/bing/v7.0/videos/search";
    var request = new XMLHttpRequest();
    var queryurl = endpoint + "?q=" + encodeURIComponent(query) + "&" + options;

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
    request.addEventListener("load", handleOnLoad);

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
Tras la correcta finalización de la solicitud HTTP, JavaScript llama a nuestro controlador de eventos `load`, `handleOnLoad()`, para controlar una solicitud HTTP GET correcta a la API. 

```javascript
// handle Bing search request results
function handleOnLoad() {
    hideDivs("noresults");

    var json = this.responseText.trim();
    var jsobj = {};

    // try to parse JSON results
    try {
        if (json.length) jsobj = JSON.parse(json);
    } catch(e) {
        renderErrorMessage("Invalid JSON response");
    }

    // show raw JSON and headers
    showDiv("json", preFormat(JSON.stringify(jsobj, null, 2)));
    showDiv("http", preFormat("GET " + this.responseURL + "\n\nStatus: " + this.status + " " + 
        this.statusText + "\n" + this.getAllResponseHeaders()));

    // if HTTP response is 200 OK, try to render search results
    if (this.status === 200) {
        var clientid = this.getResponseHeader("X-MSEdge-ClientID");
        if (clientid) retrieveValue(CLIENT_ID_COOKIE, clientid);
        if (json.length) {
            if (jsobj._type === "Videos") {//"SearchResponse" && "rankingResponse" in jsobj) {
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
> Si se produce un error en la operación de búsqueda, Bing News Search API devuelve un código de estado HTTP distinto de 200 e incluye información del error en la respuesta JSON. Además, si la solicitud tenía limitación de frecuencia, la API devuelve una respuesta vacía.
Una solicitud HTTP correcta *no* indica necesariamente que la búsqueda se haya realizado correctamente. 

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

    // Render the results to the mainline section
    for (section in { mainline: 0 }) {
         showDiv(section, renderResultsItems(section, results));
    }
}
```
Se devuelven los resultados de búsqueda como el objeto `value` de nivel superior de la respuesta JSON. Los pasamos a nuestra función `renderResultsItems()`, que los itera y llama a una función para representar cada elemento en HTML. El HTML resultante se devuelve a `renderSearchResults()`, donde se inserta en la división `results` de la página.

```javascript
// render search results
    function renderResultsItems(section, results) {   

        var items = results.value;
        var html = [];
        for (var i = 0; i < items.length; i++) { 
            var item = items[i];
            // collection name has lowercase first letter
            var type = "videos";
            var render = searchItemRenderers[type];
            html.push(render(item, section));  
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

En el código de JavaScript, el objeto, `searchItemRenderers`, puede contener *representadores:* funciones que generan código HTML para cada tipo de resultado de búsqueda. La página de búsqueda de vídeo solo usa `videos`. Consulte en otros tutoriales los distintos tipos de representadores.

```javascript
searchItemRenderers = {
    news: function(item) { ... },
    webPages: function (item) { ... }, 
    images: function(item, index, count) { ... },
    videos: function (item, section, index, count) { ... },
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

El representador `video` se muestra en el siguiente fragmento de JavaScript. Con el punto de conexión de vídeos, todos los resultados son del tipo `Videos`. Los elementos `searchItemRenderers` se muestran en el siguiente segmento de código.

```javascript
// render functions for various types of search results
    searchItemRenderers = {

    videos: function (item, section, index, count) {
        var height = 60;
        var width = Math.round(height * item.thumbnail.width / item.thumbnail.height);
        var html = [];

        html.push("<p class='images'>");
        html.push("<a href='" + item.hostPageUrl + "'>");
        var title = escapeQuotes(item.name) + "\n" + getHost(item.hostPageDisplayUrl);
        html.push("<img src='" + item.thumbnailUrl + "&h=" + height + "&w=" + width +
            "' height=" + height + " width=" + width + " title='" + title + "' alt='" + title + "'>");
        html.push("</a>");
        html.push("<br>");
        html.push("<nobr><a href='" + item.contentUrl + "'>Video page source</a> - ");
        html.push(title.replace("\n", " (").replace(/([a-z0-9])\.([a-z0-9])/g, "$1.<wbr>$2") + ")</p>");
        return html.join("");
    }
}
```

Función del representador:
> [!div class="checklist"]
> * Crea una etiqueta de párrafo, la asigna a la clase `images` y la inserta en la matriz HTML.
> * Calcula el tamaño de la miniatura de la imagen (el ancho se fija en 60 píxeles, y el alto se calcula proporcionalmente).
> * Crea la etiqueta HTML `<img>` para mostrar la miniatura de la imagen. 
> * Compila etiquetas `<a>` HTML vinculadas con la imagen y la página que la contiene.
> * Compila la descripción que muestra información sobre la imagen y el sitio en que se encuentra.

El tamaño de las miniaturas se usa tanto en la etiqueta `<img>` como en los campos `h` y `w` en la dirección URL de la miniatura. Bing devolverá [una miniatura](../bing-web-search/resize-and-crop-thumbnails.md) de ese tamaño exactamente.

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
> [Referencia de Bing Video Search API](//docs.microsoft.com/rest/api/cognitiveservices/bing-video-api-v7-reference)
