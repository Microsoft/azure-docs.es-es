---
title: 'Tutorial: Crear una aplicación web de una sola página: Bing Image Search API'
titleSuffix: Azure cognitive services
description: Bing Image Search API permite buscar en la Web imágenes relevantes de alta calidad. Use este tutorial para crear una aplicación web de una sola página que pueda enviar consultas a la API y mostrar los resultados en la página web.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: tutorial
ms.date: 03/05/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: cc73dcf6376dc5a222b1b68d67e4f02090258807
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407870"
---
# <a name="tutorial-create-a-single-page-app-using-the-bing-image-search-api"></a>Tutorial: Crear una aplicación de una sola página con Bing Image Search API

Bing Image Search API permite buscar en la Web imágenes relevantes de alta calidad. Use este tutorial para crear una aplicación web de una sola página que pueda enviar consultas a la API y mostrar los resultados en la página web. Este tutorial es similar al [tutorial correspondiente](../Bing-Web-Search/tutorial-bing-web-search-single-page-app.md) para Bing Web Search.

En la aplicación del tutorial se muestra cómo:

> [!div class="checklist"]
> * Realizar una llamada a Bing Image Search API en JavaScript
> * Mejorar los resultados de la búsqueda mediante las opciones de búsqueda
> * Mostrar y paginar los resultados de la búsqueda
> * Solicite y controle una clave de suscripción de API y un identificador de cliente de Bing.

El código fuente completo de este tutorial está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/Tutorials/Bing-Image-Search).

## <a name="prerequisites"></a>Requisitos previos

* La última versión de [Node.js](https://nodejs.org/).
* El marco de [Express.js](https://expressjs.com/) para Node.js. Las instrucciones de instalación del código fuente están disponibles en el archivo Léame del ejemplo de GitHub.

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-image-search-signup-requirements.md)]

## <a name="manage-and-store-user-subscription-keys"></a>Administración y almacenamiento de las claves de suscripción de usuarios

Esta aplicación utiliza el almacenamiento persistente de los exploradores web para almacenar las claves de suscripción de API. Si no se almacena ninguna clave, la página web solicitará al usuario la clave y la almacenará para futuros usos. Si la API rechaza la clave más adelante, la aplicación la eliminará del almacenamiento. En este ejemplo se usa el punto de conexión global. También puede usar el punto de conexión del [subdominio personalizado](../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.


Defina funciones `storeValue` y `retrieveValue` que usan el objeto `localStorage` (si el explorador lo admite) o una cookie.

```javascript
// Cookie names for data being stored
API_KEY_COOKIE   = "bing-search-api-key";
CLIENT_ID_COOKIE = "bing-search-client-id";
// The Bing Image Search API endpoint
BING_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/images/search";

try { //Try to use localStorage first
    localStorage.getItem;   

    window.retrieveValue = function (name) {
        return localStorage.getItem(name) || "";
    }
    window.storeValue = function(name, value) {
        localStorage.setItem(name, value);
    }
} catch (e) {
    //If the browser doesn't support localStorage, try a cookie
    window.retrieveValue = function (name) {
        var cookies = document.cookie.split(";");
        for (var i = 0; i < cookies.length; i++) {
            var keyvalue = cookies[i].split("=");
            if (keyvalue[0].trim() === name) return keyvalue[1];
        }
        return "";
    }
    window.storeValue = function (name, value) {
        var expiry = new Date();
        expiry.setFullYear(expiry.getFullYear() + 1);
        document.cookie = name + "=" + value.trim() + "; expires=" + expiry.toUTCString();
    }
}
```

La función `getSubscriptionKey()` intenta recuperar una clave almacenada previamente con el uso de `retrieveValue`. Si no se encuentra ninguna clave, pedirá al usuario que la proporcione y la almacenará con `storeValue`.

```javascript

// Get the stored API subscription key, or prompt if it's not found
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

La etiqueta `<form>` HTML `onsubmit` llama a la función `bingWebSearch` para devolver los resultados de la búsqueda. `bingWebSearch` utiliza `getSubscriptionKey` para autenticar cada consulta. Tal como se muestra en la definición anterior, si no se ha especificado la clave `getSubscriptionKey` se la pide al usuario. A continuación, la clave se almacena para que la siga usando la aplicación.

```html
<form name="bing" onsubmit="this.offset.value = 0; return bingWebSearch(this.query.value,
bingSearchOptions(this), getSubscriptionKey())">
```

## <a name="send-search-requests"></a>Envío de solicitudes de búsqueda

Esta aplicación usa un valor HTML `<form>` para enviar solicitudes de búsqueda de usuario, mediante el atributo `onsubmit` para llamar a `newBingImageSearch()`.

```html
<form name="bing" onsubmit="return newBingImageSearch(this)">
```

De forma predeterminada, el controlador `onsubmit` devuelve `false`, lo que impide que el formulario se envíe.

## <a name="select-search-options"></a>Selección de opciones de búsqueda

![[Formulario de Bing Image Search]](media/cognitive-services-bing-images-api/image-search-spa-form.png)

Bing Image Search API ofrece varios [parámetros de consulta de filtro](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#filter-query-parameters) para limitar y filtrar los resultados de la búsqueda. El formulario HTML de esta aplicación usa y muestra las siguientes opciones de parámetro:

| Opción | Descripción |
|--------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `where`      | Un menú desplegable para seleccionar el mercado (ubicación e idioma) que se usa en la búsqueda.                                                                                             |
| `query`      | El campo de texto en el que se especifican los términos de búsqueda.                                                                                                                                 |
| `aspect`     | Botones de radio para elegir las proporciones de las imágenes que se encuentren: cuadradas, horizontales o verticales.                                                                                     |
| `color`      |                                                                                                                                                                                    |
| `when`       | Menú desplegable para, opcionalmente, limitar la búsqueda al día, la semana o el mes más reciente.                                                                                          |
| `safe`       | Una casilla que indica si se usa la característica Búsqueda segura de Bing para filtrar los resultados "para adultos".                                                                                      |
| `count`      | Campo oculto. Número de resultados de búsqueda que se devolverán en cada solicitud. Cambie este valor para mostrar más o menos resultados por página.                                                            |
| `offset`     | Campo oculto. El desplazamiento del primer resultado de búsqueda de la solicitud; se usa para la paginación. Se restablece a `0` en cada nueva solicitud.                                                           |
| `nextoffset` | Campo oculto. Al recibir un resultado de búsqueda, este campo se define con el valor de `nextOffset` en la respuesta. El uso de este campo evita resultados superpuestos en páginas sucesivas. |
| `stack`      | Campo oculto. Una lista codificada en JSON de los desplazamientos de las páginas anteriores de los resultados de búsqueda para volver a páginas anteriores.                                                      |

La función `bingSearchOptions()` da formato a estas opciones en una cadena de consulta parcial, que se puede usar en las solicitudes de API de la aplicación.  

```javascript
// Build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "off"));
    if (form.when.value.length) options.push("freshness=" + form.when.value);
    var aspect = "all";
    for (var i = 0; i < form.aspect.length; i++) {
        if (form.aspect[i].checked) {
            aspect = form.aspect[i].value;
            break;
        }
    }
    options.push("aspect=" + aspect);
    if (form.color.value) options.push("color=" + form.color.value);
    options.push("count=" + form.count.value);
    options.push("offset=" + form.offset.value);
    return options.join("&");
}
```

## <a name="performing-the-request"></a>Realizar la solicitud

Con el uso de la consulta de búsqueda, la cadena de opciones y la clave de API, la función `BingImageSearch()` utiliza un objeto XMLHttpRequest para enviar la solicitud al punto de conexión de Bing Image Search.


```javascript
// perform a search given query, options string, and API key
function bingImageSearch(query, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("results", "related", "_json", "_http", "paging1", "paging2", "error");

    var request = new XMLHttpRequest();
    var queryurl = BING_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;

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

Tras la correcta finalización de la solicitud HTTP, JavaScript llama a controlador de eventos `handleBingResponse()` de "carga" para controlar una solicitud HTTP GET correcta.

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
            if (jsobj._type === "Images") {
                if (jsobj.nextOffset) document.forms.bing.nextoffset.value = jsobj.nextOffset;
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
> Las solicitudes HTTP correctas pueden contener información de búsqueda errónea. Si se produce un error durante la operación de búsqueda, Bing Image Search API devuelve un código de estado HTTP distinto de 200 e información de error en la respuesta JSON. Además, si la solicitud tenía limitación de frecuencia, la API devuelve una respuesta vacía.

## <a name="display-the-search-results"></a>Visualización de los resultados de la búsqueda

Los resultados de la búsqueda se muestran mediante la función `renderSearchResults()`, que toma el valor JSON devuelto por el servicio Bing Image Search y llama a una función de representador apropiada en cualquier imagen devuelta y en las búsquedas relacionadas.

```javascript
function renderSearchResults(results) {

    // add Prev / Next links with result count
    var pagingLinks = renderPagingLinks(results);
    showDiv("paging1", pagingLinks);
    showDiv("paging2", pagingLinks);

    showDiv("results", renderImageResults(results.value));
    if (results.relatedSearches)
        showDiv("sidebar", renderRelatedItems(results.relatedSearches));
}
```

Los resultados de la búsqueda de imágenes están incluidos en el objeto `value` de alto nivel dentro de la respuesta JSON. Se pasan a `renderImageResults()`, que realizan la iteración por los resultados y convierte cada elemento a HTML.

```javascript
function renderImageResults(items) {
    var len = items.length;
    var html = [];
    if (!len) {
        showDiv("noresults", "No results.");
        hideDivs("paging1", "paging2");
        return "";
    }
    for (var i = 0; i < len; i++) {
        html.push(searchItemRenderers.images(items[i], i, len));
    }
    return html.join("\n\n");
}
```

Bing Image Search API puede devolver los cuatro tipos de sugerencias de búsqueda para ayudar a guiar las experiencias de búsqueda de los usuarios, cada una en su propio objeto de nivel superior:

| Sugerencia         | Descripción                                                                                                                                                                                                         |
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `pivotSuggestions` | Consultas que reemplazan una palabra dinámica de la búsqueda original por otra. Por ejemplo, si busca "flores rojas", una palabra dinámica podría ser "rojas", y una sugerencia dinámica podría ser "flores amarillas". |
| `queryExpansions`  | Consultas que limitan la búsqueda original al agregar más términos. Por ejemplo, si busca "Microsoft Surface", una expansión de consulta podría ser "Microsoft Surface Pro".                                   |
| `relatedSearches`  | Consultas que también han realizado otros usuarios que escribieron la búsqueda original. Por ejemplo, su buscó "Monte Rainier", una búsqueda relacionada podría ser "Monte Santa Elena".                       |
| `similarTerms`     | Consultas que son similares en significado a la búsqueda original. Por ejemplo, si busca "gatos", un término similar podría ser "monos".                                                                   |

Esta aplicación solo representa las sugerencias de `relatedItems` y coloca los vínculos resultantes en la barra lateral de la página.

## <a name="rendering-search-results"></a>Representación de los resultados de la búsqueda

En esta aplicación, el objeto `searchItemRenderers` contiene funciones de representador que generan HTML para cada tipo de resultado de la búsqueda.

```javascript
searchItemRenderers = {
    images: function(item, index, count) { ... },
    relatedSearches: function(item) { ... }
}
```

Estas funciones de representador aceptan los parámetros siguientes:

| Parámetro         | Descripción                                                                                              |
|---------|----------------------------------------------------------------------------------------------|
| `item`  | El objeto de JavaScript que contiene las propiedades del elemento, como su dirección URL y la descripción. |
| `index` | El índice del elemento de resultado dentro de su colección.                                          |
| `count` | El número de elementos de la colección del elemento de resultado de la búsqueda.                                  |

Los parámetros `index` y `count` se usan para enumerar los resultados, generar HTML para las colecciones y organizar el contenido. Concretamente:

* Calcula el tamaño de miniatura de imagen (el ancho varía, con un mínimo de 120 píxeles, mientras que la altura se fija en 90 píxeles).
* Crea la etiqueta HTML `<img>` para mostrar la miniatura de la imagen.
* Compila etiquetas `<a>` HTML vinculadas con la imagen y la página que la contiene.
* Compila la descripción que muestra información sobre la imagen y el sitio en que se encuentra.

```javascript
    images: function (item, index, count) {
        var height = 120;
        var width = Math.max(Math.round(height * item.thumbnail.width / item.thumbnail.height), 120);
        var html = [];
        if (index === 0) html.push("<p class='images'>");
        var title = escape(item.name) + "\n" + getHost(item.hostPageDisplayUrl);
        html.push("<p class='images' style='max-width: " + width + "px'>");
        html.push("<img src='"+ item.thumbnailUrl + "&h=" + height + "&w=" + width +
            "' height=" + height + " width=" + width + "'>");
        html.push("<br>");
        html.push("<nobr><a href='" + item.contentUrl + "'>Image</a> - ");
        html.push("<a href='" + item.hostPageUrl + "'>Page</a></nobr><br>");
        html.push(title.replace("\n", " (").replace(/([a-z0-9])\.([a-z0-9])/g, "$1.<wbr>$2") + ")</p>");
        return html.join("");
    }, // relatedSearches renderer omitted
```

`height` y `width` de la imagen en miniatura se usan tanto en la etiqueta `<img>` como en los campos `h` y `w` de la dirección URL de la miniatura. Esto permite que Bing devuelva [una miniatura](../bing-web-search/resize-and-crop-thumbnails.md) de ese tamaño exactamente.

## <a name="persisting-client-id"></a>Identificador de cliente persistente

Las respuestas de Bing Search API pueden incluir un encabezado `X-MSEdge-ClientID` que debe devolverse a la API con las solicitudes sucesivas. Si se usan varias instancias de Bing Search API, se debe usar el mismo identificador de cliente con todas ellas, si es posible.

Especificar el encabezado `X-MSEdge-ClientID` permite a las API de Bing asociar todas las búsquedas de un usuario, algo que resulta útil.

En primer lugar, permite al motor de búsqueda de Bing aplicar un contexto pasado a las búsquedas, con el fin de encontrar los resultados que más satisfagan al usuario. Si un usuario ha buscado previamente términos relacionados con la navegación, por ejemplo, una búsqueda posterior de "nudos" podría devolver información acerca de los nudos que se usan en la navegación de forma preferente.

En segundo lugar, Bing puede seleccionar aleatoriamente usuarios para disfrutar de nuevas características antes de que estén disponibles públicamente. Proporcionar el mismo identificador de cliente con cada solicitud garantiza que los usuarios elegidos para ver una característica la vean siempre. Sin el identificador de cliente, el usuario puede ver una característica aparecer y desaparecer, de forma aparentemente aleatoria, en los resultados de búsqueda.

Las directivas de seguridad del explorador (CORS) pueden impedir que el encabezado `X-MSEdge-ClientID` esté disponible para JavaScript. Esta limitación tiene lugar cuando la respuesta a la búsqueda tiene un origen distinto al de la página que la solicitó. En un entorno de producción, debería abordar esta directiva mediante el hospedaje de un script de lado servidor que realice la llamada API en el mismo dominio que la página web. Como el script tiene el mismo origen que la página web, el encabezado `X-MSEdge-ClientID` está disponible para JavaScript.

> [!NOTE]
> En una aplicación web de producción, debe realizar la solicitud del lado servidor de todos modos. En caso contrario, es preciso incluir la clave de Bing Search API en la página web, donde está disponible para cualquiera que vea el origen. Se le facturará todo el uso bajo su clave de suscripción a API, incluso las solicitudes que realicen partes no autorizadas, por lo que es importante no exponer su clave.

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
> [Extracción de detalles de imágenes con Bing Image Search API](tutorial-image-post.md)

## <a name="see-also"></a>Consulte también

* [Referencia de Bing Image Search API](//docs.microsoft.com/rest/api/cognitiveservices/bing-images-api-v7-reference)
