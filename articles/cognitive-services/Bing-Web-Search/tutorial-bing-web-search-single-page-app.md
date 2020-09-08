---
title: 'Tutorial: Creación una aplicación web de una sola página (Bing Web Search API)'
titleSuffix: Azure Cognitive Services
description: Esta aplicación de una página muestra cómo se puede usar Bing Web Search API para recuperar, analizar y mostrar los resultados de la búsqueda pertinentes en una aplicación de una página.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: tutorial
ms.date: 03/05/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: 251dbf6897f7efe56d968122c2ccad7093d39e44
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89300058"
---
# <a name="tutorial-create-a-single-page-app-using-the-bing-web-search-api"></a>Tutorial: Creación de una aplicación de una sola página mediante Bing Web Search API

Esta aplicación de una página muestra cómo recuperar, analizar y mostrar los resultados de la búsqueda desde Bing Web Search API. El tutorial usa código HTML y CSS reutilizable y se centra en el código de JavaScript. Los archivos HTML, CSS y JS están disponibles en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/tree/master/Tutorials/Bing-Web-Search) con instrucciones de inicio rápido.

Esta aplicación de ejemplo puede:

> [!div class="checklist"]
> * Llamar a Bing Web Search API con opciones de búsqueda
> * Mostrar resultados de vídeo, imágenes, noticias y web
> * Paginar resultados
> * Administrar claves de suscripción
> * errores

Para usar esta aplicación, se requiere una [cuenta de Azure Cognitive Services](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) con Bing Search API.

## <a name="prerequisites"></a>Requisitos previos

A continuación se detalla lo que necesitará para ejecutar la aplicación:

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="cree un recurso de Bing Search"  target="_blank">crear un recurso de Bing Search<span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.

* Node.js 8 o posterior

El primer paso consiste en clonar el repositorio con el código fuente de la aplicación de ejemplo.

```console
git clone https://github.com/Azure-Samples/cognitive-services-REST-api-samples.git
```

A continuación, ejecute `npm install`. Para este tutorial, Express.js es la única dependencia.

```console
cd <path-to-repo>/cognitive-services-REST-api-samples/Tutorials/Bing-Web-Search
npm install
```

## <a name="app-components"></a>Componentes de la aplicación

La aplicación de ejemplo que estamos compilando se compone de cuatro partes:

* `bing-web-search.js`: nuestra aplicación Express.js. Controla la lógica de solicitud/respuesta y el enrutamiento.
* `public/index.html`: el esqueleto de nuestra aplicación; define cómo se presentan los datos al usuario.
* `public/css/styles.css`: define los estilos de página, como fuentes, colores y tamaño del texto.
* `public/js/scripts.js`: contiene la lógica para realizar solicitudes a Bing Web Search API, administrar claves de suscripción, administrar y analizar respuestas y mostrar resultados.

En este tutorial se centra en `scripts.js` y la lógica necesaria para llamar a Bing Web Search API y controlar la respuesta.

## <a name="html-form"></a>Formulario HTML

El elemento `index.html` incluye un formulario que permite a los usuarios buscar y seleccionar las opciones de búsqueda. El atributo `onsubmit` se desencadena cuando se envía el formulario, con una llamada al método `bingWebSearch()` definido en `scripts.js`. Toma tres argumentos:

* Consulta de búsqueda
* Opciones seleccionadas
* Subscription key

```html
<form name="bing" onsubmit="return bingWebSearch(this.query.value,
    bingSearchOptions(this), getSubscriptionKey())">
```

## <a name="query-options"></a>Opciones de consulta

El formulario HTML incluye opciones que se asignan a parámetros de consulta en [Bing Web Search API v7](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#query-parameters). En esta tabla se proporciona un desglose de cómo los usuarios pueden filtrar los resultados de la búsqueda mediante la aplicación de ejemplo:

| Parámetro | Descripción |
|-----------|-------------|
| `query` | Un campo de texto para escribir una cadena de consulta. |
| `where` | Un menú desplegable para seleccionar el mercado (ubicación e idioma). |
| `what` | Casillas para promover tipos de resultados específicos. Por ejemplo, la promoción de imágenes aumenta el orden de clasificación de las imágenes en los resultados de la búsqueda. |
| `when` | Un menú desplegable que permite al usuario limitar los resultados de la búsqueda para hoy, esta semana o este mes. |
| `safe` | Una casilla para habilitar la búsqueda segura de Bing, que filtra el contenido para adultos. |
| `count` | Campo oculto. Número de resultados de búsqueda que se devolverán en cada solicitud. Cambie este valor para mostrar más o menos resultados por página. |
| `offset` | Campo oculto. El desplazamiento del primer resultado de la búsqueda de la solicitud, que se usa para la paginación. Se restablece a `0` en cada nueva solicitud. |

> [!NOTE]
> Bing Web Search API proporciona parámetros de consulta adicionales para ayudar a refinar los resultados de la búsqueda. Este ejemplo usa solo algunos. Para obtener una lista completa de los parámetros disponibles, vea la [referencia de Bing Web Search API v7](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#query-parameters).

La función `bingSearchOptions()` convierte estas opciones para que coincidan con el formato que requiere Bing Search API.

```javascript
// Build query options from selections in the HTML form.
function bingSearchOptions(form) {

    var options = [];
    // Where option.
    options.push("mkt=" + form.where.value);
    // SafeSearch option.
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "moderate"));
    // Freshness option.
    if (form.when.value.length) options.push("freshness=" + form.when.value);
    var what = [];
    for (var i = 0; i < form.what.length; i++)
        if (form.what[i].checked) what.push(form.what[i].value);
    // Promote option.
    if (what.length) {
        options.push("promote=" + what.join(","));
        options.push("answerCount=9");
    }
    // Count option.
    options.push("count=" + form.count.value);
    // Offset option.
    options.push("offset=" + form.offset.value);
    // Hardcoded text decoration option.
    options.push("textDecorations=true");
    // Hardcoded text format option.
    options.push("textFormat=HTML");
    return options.join("&");
}
```

`SafeSearch` se puede establecer en `strict`, `moderate` o `off`, con `moderate` como la configuración predeterminada para Bing Web Search. Este formulario usa una casilla que tiene dos estados: `strict` o `moderate`.

Si cualquiera de las casillas **Promover** está seleccionada, se agrega el parámetro `answerCount` a la consulta. `answerCount` es obligatorio cuando se usa el parámetro `promote`. En este fragmento de código, el valor se establece en `9` para devolver todos los tipos de resultados disponibles.
> [!NOTE]
> La promoción de un tipo de resultado no *garantiza* que se incluirá en los resultados de la búsqueda. En su lugar, la promoción aumenta la clasificación de esos tipos de resultados con respecto a su clasificación habitual. Para limitar las búsquedas a determinados tipos de resultados, use el parámetro de consulta `responseFilter` o llamar a un punto de conexión más concreto como Bing Image Search o Bing News Search.

Los parámetros de consulta `textDecoration` y `textFormat` están codificados de forma rígida en el script para que el término de búsqueda aparezca en negrita en los resultados de la búsqueda. Estos parámetros no son necesarios.

## <a name="manage-subscription-keys"></a>Administrar claves de suscripción

Para evitar tener que codificar de forma rígida la clave de suscripción de Bing Search API, esta aplicación de ejemplo usa el almacenamiento persistente del explorador para almacenar la clave de suscripción. Si no se almacena ninguna clave de suscripción, se solicita al usuario que escriba una. Si se rechaza la clave de suscripción mediante la API, se pedirá al usuario que vuelva a escribir una clave de suscripción.

La función `getSubscriptionKey()` usa las funciones `storeValue` y `retrieveValue` para almacenar y recuperar la clave de suscripción de un usuario. Estas funciones usan el objeto `localStorage`, si se admite, o cookies.

```javascript
// Cookie names for stored data.
API_KEY_COOKIE   = "bing-search-api-key";
CLIENT_ID_COOKIE = "bing-search-client-id";

BING_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/search";

// See source code for storeValue and retrieveValue definitions.

// Get stored subscription key, or prompt if it isn't found.
function getSubscriptionKey() {
    var key = retrieveValue(API_KEY_COOKIE);
    while (key.length !== 32) {
        key = prompt("Enter Bing Search API subscription key:", "").trim();
    }
    // Always set the cookie in order to update the expiration date.
    storeValue(API_KEY_COOKIE, key);
    return key;
}
```

Como vimos anteriormente, cuando se envía el formulario, se activa `onsubmit`, con una llamada a `bingWebSearch`. Esta función se inicializa y envía la solicitud. Se llama a `getSubscriptionKey` en cada envío para autenticar la solicitud.

## <a name="call-bing-web-search"></a>Llamar a Bing Web Search

Dada la consulta, la cadena de opciones y la clave de suscripción, la función `BingWebSearch` crea un objeto `XMLHttpRequest` para llamar al punto de conexión de Bing Web Search.

```javascript
// Perform a search constructed from the query, options, and subscription key.
function bingWebSearch(query, options, key) {
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;

    showDiv("noresults", "Working. Please wait.");
    hideDivs("pole", "mainline", "sidebar", "_json", "_http", "paging1", "paging2", "error");

    var request = new XMLHttpRequest();
    var queryurl = BING_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;

    // Initialize the request.
    try {
        request.open("GET", queryurl);
    }
    catch (e) {
        renderErrorMessage("Bad request (invalid URL)\n" + queryurl);
        return false;
    }

    // Add request headers.
    request.setRequestHeader("Ocp-Apim-Subscription-Key", key);
    request.setRequestHeader("Accept", "application/json");
    var clientid = retrieveValue(CLIENT_ID_COOKIE);
    if (clientid) request.setRequestHeader("X-MSEdge-ClientID", clientid);

    // Event handler for successful response.
    request.addEventListener("load", handleBingResponse);

    // Event handler for errors.
    request.addEventListener("error", function() {
        renderErrorMessage("Error completing request");
    });

    // Event handler for an aborted request.
    request.addEventListener("abort", function() {
        renderErrorMessage("Request aborted");
    });

    // Send the request.
    request.send();
    return false;
}
```

Después de una solicitud correcta, el controlador de eventos `load` se activa y llama a la función `handleBingResponse`. `handleBingResponse` analiza el objeto de resultado, muestra los resultados y contiene la lógica de error para las solicitudes con error.

```javascript
function handleBingResponse() {
    hideDivs("noresults");

    var json = this.responseText.trim();
    var jsobj = {};

    // Try to parse results object.
    try {
        if (json.length) jsobj = JSON.parse(json);
    } catch(e) {
        renderErrorMessage("Invalid JSON response");
        return;
    }

    // Show raw JSON and the HTTP request.
    showDiv("json", preFormat(JSON.stringify(jsobj, null, 2)));
    showDiv("http", preFormat("GET " + this.responseURL + "\n\nStatus: " + this.status + " " +
        this.statusText + "\n" + this.getAllResponseHeaders()));

    // If the HTTP response is 200 OK, try to render the results.
    if (this.status === 200) {
        var clientid = this.getResponseHeader("X-MSEdge-ClientID");
        if (clientid) retrieveValue(CLIENT_ID_COOKIE, clientid);
        if (json.length) {
            if (jsobj._type === "SearchResponse" && "rankingResponse" in jsobj) {
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    }

    // Any other HTTP response is considered an error.
    else {
        // 401 is unauthorized; force a re-prompt for the user's subscription
        // key on the next request.
        if (this.status === 401) invalidateSubscriptionKey();

        // Some error responses don't have a top-level errors object, if absent
        // create one.
        var errors = jsobj.errors || [jsobj];
        var errmsg = [];

        // Display the HTTP status code.
        errmsg.push("HTTP Status " + this.status + " " + this.statusText + "\n");

        // Add all fields from all error responses.
        for (var i = 0; i < errors.length; i++) {
            if (i) errmsg.push("\n");
            for (var k in errors[i]) errmsg.push(k + ": " + errors[i][k]);
        }

        // Display Bing Trace ID if it isn't blocked by CORS.
        var traceid = this.getResponseHeader("BingAPIs-TraceId");
        if (traceid) errmsg.push("\nTrace ID " + traceid);

        // Display the error message.
        renderErrorMessage(errmsg.join("\n"));
    }
}
```

> [!IMPORTANT]
> Una solicitud HTTP correcta *no* significa que la búsqueda en sí se realizó correctamente. Si se produce un error en la operación de búsqueda, Bing Web Search API devuelve un código de estado HTTP distinto de 200 e incluye información de error en la respuesta JSON. Si la solicitud tenía limitación de frecuencia, la API devuelve una respuesta vacía.

Gran parte del código de las dos funciones anteriores está dedicado al control de errores. Pueden producirse errores en las siguientes fases:

| Fase | Errores posibles | Controlado por |
|-------|--------------------|------------|
| Creación de un objeto de solicitud | Dirección URL no válida | Bloqueo `try` / `catch` |
| Hacer la solicitud | Errores de red, conexiones anuladas | Controladores de eventos `error` y `abort` |
| Realizar la búsqueda | Solicitud no válida, JSON no válido, límites de frecuencia | Pruebas en el controlador de eventos `load` |

Los errores se controlan con una llamada a `renderErrorMessage()`. Si la respuesta pasa todas las pruebas de error, se llama a `renderSearchResults()` para mostrar los resultados de la búsqueda.

## <a name="display-search-results"></a>Mostrar los resultados de la búsqueda

Existen [requisitos de uso y visualización](useanddisplayrequirements.md) para los resultados devueltos por Bing Web Search API. Puesto que una respuesta puede incluir varios tipos de resultados, no es suficiente iterar por la colección `WebPages` de alto nivel. En su lugar, la aplicación de ejemplo usa `RankingResponse` para ordenar los resultados según la especificación.

> [!NOTE]
> Si solo desea un único tipo de resultado, use el parámetro de consulta `responseFilter` o considere el uso de uno de los otros puntos de conexión de Bing Search, como Bing Image Search.

Cada respuesta tiene un objeto `RankingResponse` que puede incluir hasta tres colecciones: `pole`, `mainline` y `sidebar`. `pole`, si está presente, es el resultado de búsqueda más apropiado y debe mostrarse de manera destacada. `mainline` contiene la mayoría de los resultados de la búsqueda y se muestra inmediatamente después de `pole`. `sidebar` incluye los resultados de la búsqueda auxiliares. Si es posible, estos resultados se deben mostrar en la barra lateral. Si los límites de la pantalla hacen que una barra lateral sea poco práctica, estos resultados deben aparecer después de los resultados de `mainline`.

Cada `RankingResponse` incluye una matriz `RankingItem` que especifica cómo se deben ordenar los resultados. Nuestra aplicación de ejemplo usa los parámetros `answerType` y `resultIndex` para identificar el resultado.

> [!NOTE]
> Hay otras formas de identificar y clasificar los resultados. Para más información, vea [Uso de la clasificación para mostrar resultados](rank-results.md).

Veamos el código:

```javascript
// Render the search results from the JSON response.
function renderSearchResults(results) {

    // If spelling was corrected, update the search field.
    if (results.queryContext.alteredQuery)
        document.forms.bing.query.value = results.queryContext.alteredQuery;

    // Add Prev / Next links with result count.
    var pagingLinks = renderPagingLinks(results);
    showDiv("paging1", pagingLinks);
    showDiv("paging2", pagingLinks);

    // Render the results for each section.
    for (section in {pole: 0, mainline: 0, sidebar: 0}) {
        if (results.rankingResponse[section])
            showDiv(section, renderResultsItems(section, results));
    }
}
```

La función `renderResultsItems()` realiza una iteración por los elementos de cada colección `RankingResponse`, asigna cada resultado de la clasificación a un resultado de la búsqueda mediante los valores `answerType` y `resultIndex`, y llama a la función de representación adecuada para generar el HTML. Si `resultIndex` no se especifica para un elemento, `renderResultsItems()` realiza una iteración por todos los resultados de ese tipo y llama a la función de representación en cada elemento. El HTML resultante se inserta en el elemento `<div>` correspondiente de `index.html`.

```javascript
// Render search results from the RankingResponse object per rank response and
// use and display requirements.
function renderResultsItems(section, results) {

    var items = results.rankingResponse[section].items;
    var html = [];
    for (var i = 0; i < items.length; i++) {
        var item = items[i];
        // Collection name has lowercase first letter while answerType has uppercase
        // e.g. `WebPages` RankingResult type is in the `webPages` top-level collection.
        var type = item.answerType[0].toLowerCase() + item.answerType.slice(1);
        if (type in results && type in searchItemRenderers) {
            var render = searchItemRenderers[type];
            // This ranking item refers to ONE result of the specified type.
            if ("resultIndex" in item) {
                html.push(render(results[type].value[item.resultIndex], section));
            // This ranking item refers to ALL results of the specified type.
            } else {
                var len = results[type].value.length;
                for (var j = 0; j < len; j++) {
                    html.push(render(results[type].value[j], section, j, len));
                }
            }
        }
    }
    return html.join("\n\n");
}
```

## <a name="review-renderer-functions"></a>Revisión de las funciones de representador

En nuestra aplicación de ejemplo, el objeto `searchItemRenderers` incluye funciones que generan código HTML para cada tipo de resultado de la búsqueda.

```javascript
// Render functions for each result type.
searchItemRenderers = {
    webPages: function(item) { ... },
    news: function(item) { ... },
    images: function(item, section, index, count) { ... },
    videos: function(item, section, index, count) { ... },
    relatedSearches: function(item, section, index, count) { ... }
}
```

> [!IMPORTANT]
> La aplicación de ejemplo tiene representadores para páginas web, noticias, imágenes, vídeos y búsquedas relacionadas. La aplicación necesitará representadores para cualquier tipo de resultados que puede recibir, lo que podría incluir cálculos, sugerencias de ortografía, entidades, zonas horarias y definiciones.

Algunas de las funciones de representación solo aceptan el parámetro `item`. Otros aceptan parámetros adicionales, que se pueden usar para representar elementos de forma diferente según el contexto. Un representador que no usa esta información no necesita aceptar estos parámetros.

Los argumentos del contexto son:

| Parámetro  | Descripción |
|------------|-------------|
| `section` | La sección de resultados (`pole`, `mainline`, o `sidebar`) en que aparece el elemento. |
| `index`<br>`count` | Disponible cuando el elemento `RankingResponse` especifica que se deben mostrar todos los resultados de una colección determinada; en caso contrario, `undefined`. El índice del elemento en su colección y el número total de elementos de dicha colección. Puede usar esta información para numerar los resultados, generar HTML diferente para el primer o el último resultado y así sucesivamente. |

En la aplicación de ejemplo, los representadores `images` y `relatedSearches` usan los argumentos de contexto para personalizar el HTML generado. Observemos más de cerca el representador `images`:

```javascript
searchItemRenderers = {
    // Render image result with thumbnail.
    images: function(item, section, index, count) {
        var height = 60;
        var width = Math.round(height * item.thumbnail.width / item.thumbnail.height);
        var html = [];
        if (section === "sidebar") {
            if (index) html.push("<br>");
        } else {
            if (!index) html.push("<p class='images'>");
        }
        html.push("<a href='" + item.hostPageUrl + "'>");
        var title = escape(item.name) + "\n" + getHost(item.hostPageDisplayUrl);
        html.push("<img src='"+ item.thumbnailUrl + "&h=" + height + "&w=" + width +
            "' height=" + height + " width=" + width + " title='" + title + "' alt='" + title + "'>");
        html.push("</a>");
        return html.join("");
    },
    // Other renderers are omitted from this sample...
}
```

El representador de imágenes:

* Calcula el tamaño de la miniatura de la imagen (el ancho varía, mientras que la altura se fija en 60 píxeles).
* Inserta el código HTML que precede al resultado de la imagen en función del contexto.
* Compila la etiqueta `<a>` HTML vinculada a la página que contiene la imagen.
* Crea la etiqueta HTML `<img>` para mostrar la miniatura de la imagen.

El representador de imágenes utiliza las variables `section` y `index` para mostrar los resultados de forma diferente en función del lugar en que aparezcan. Se inserta un salto de línea (etiqueta `<br>`) entre los resultados de la imagen de la barra lateral, con el fin de que la barra lateral muestre una columna de imágenes. En otras secciones, el primer resultado de la imagen `(index === 0)` va precedido por la etiqueta `<p>`.

El tamaño de las miniaturas se usa tanto en la etiqueta `<img>` como en los campos `h` y `w` en la dirección URL de la miniatura. Los atributos `title` y `alt` (una descripción textual de la imagen) se construyen a partir de nombre de la imagen y el nombre de host de la dirección URL.

Este es un ejemplo de cómo las imágenes se muestran en la aplicación de ejemplo:

![[Resultados de imágenes de Bing]](media/cognitive-services-bing-web-api/web-search-spa-images.png)

## <a name="persist-the-client-id"></a>Conservación del identificador de cliente

Las respuestas de Bing Search API pueden incluir un encabezado `X-MSEdge-ClientID` que debe volver a enviarse a la API con cada solicitud sucesiva. Si su aplicación usa más de una instancia de Bing Search APIs, asegúrese de que se envía el mismo identificador de cliente con cada solicitud en los servicios.

Proporcionar el encabezado `X-MSEdge-ClientID` permite a las API de Bing asociar las búsquedas de un usuario. En primer lugar, permite al motor de búsqueda de Bing aplicar un contexto pasado a las búsquedas, con el fin de encontrar los resultados que más satisfagan a la solicitud. Si un usuario ha buscado previamente términos relacionados con la navegación, por ejemplo, una búsqueda posterior de "nudos" podría devolver información acerca de los nudos que se usan en la navegación de forma preferente. En segundo lugar, Bing puede seleccionar aleatoriamente usuarios para disfrutar de nuevas características antes de que estén disponibles públicamente. Especificar el mismo identificador de cliente en todas las solicitudes garantiza que los usuarios elegidos para ver una característica siempre la verán. Sin el identificador de cliente, el usuario puede ver una característica aparecer y desaparecer, de forma aparentemente aleatoria, en los resultados de búsqueda.

Las directivas de seguridad del explorador, como el uso compartido de recursos entre orígenes (CORS), pueden evitar que la aplicación de ejemplo acceda al encabezado `X-MSEdge-ClientID`. Esta limitación tiene lugar cuando la respuesta a la búsqueda tiene un origen distinto al de la página que la solicitó. En un entorno de producción, debería abordar esta directiva mediante el hospedaje de un script de lado servidor que realice la llamada API en el mismo dominio que la página web. Como el script tiene el mismo origen que la página web, el encabezado `X-MSEdge-ClientID` está disponible para JavaScript.

> [!NOTE]
> En una aplicación web de producción, debe realizar la solicitud del lado servidor de todos modos. En caso contrario, es preciso incluir la clave de suscripción de Bing Search API en la página web, donde está disponible para cualquiera que vea el origen. Se le facturará todo el uso bajo su clave de suscripción a API, incluso las solicitudes que realicen partes no autorizadas, por lo que es importante no exponer su clave.

Para fines de desarrollo, puede realizar la solicitud a través de un proxy CORS. La respuesta de un proxy de este tipo tiene un encabezado `Access-Control-Expose-Headers` que filtra los encabezados de respuesta y hace que estén disponibles para JavaScript.

Es fácil instalar un proxy CORS para permitir que nuestra aplicación de ejemplo acceda al encabezado de identificador de cliente. Ejecute este comando:

```console
npm install -g cors-proxy-server
```

A continuación, cambie el punto de conexión de Bing Web Search de `script.js` a:

```javascript
http://localhost:9090/https://api.cognitive.microsoft.com/bing/v7.0/search
```

Inicie el proxy CORS con este comando:

```console
cors-proxy-server
```

Deje la ventana de comandos abierta mientras utiliza la aplicación de ejemplo; si cierra la ventana, el proxy se detiene. En la sección de encabezados HTTP ampliable debajo de los resultados de la búsqueda, el encabezado `X-MSEdge-ClientID` debe estar visible. Compruebe que sea el mismo para cada solicitud.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de Bing Web Search API v7](//docs.microsoft.com/rest/api/cognitiveservices/bing-web-api-v7-reference)
