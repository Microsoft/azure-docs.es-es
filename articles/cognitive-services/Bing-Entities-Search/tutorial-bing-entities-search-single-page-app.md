---
title: 'Tutorial: Aplicación web de página única de Bing Entity Search'
titleSuffix: Azure Cognitive Services
description: En este tutorial se indica cómo utilizar Bing Entity Search API en una aplicación web de una sola página.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-entity-search
ms.topic: tutorial
ms.date: 03/05/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: d14681e07518188deabd9f48a04b80a489d858b8
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87407938"
---
# <a name="tutorial-single-page-web-app"></a>Tutorial: Aplicación web de una sola página

Bing Entity Search API le permite buscar en Internet información acerca de *entidades* y *lugares.* Puede solicitar cualquier tipo de resultado, o ambos, en una consulta determinada. Las definiciones de lugares y entidades se proporcionan a continuación.

| Resultado | Descripción |
|-|-|
|Entidades|Gente conocida, lugares y cosas que puede encontrar en función del nombre.|
|Lugares|Restaurantes, hoteles y otras empresas locales que encuentra en función del nombre *o* el tipo (restaurantes italianos).|

En este tutorial, creamos una aplicación web de una sola página que usa Bing Entity Search API para mostrar los resultados de la búsqueda directamente en la página. La aplicación incluye componentes HTML, CSS y JavaScript.

Esta API le permite priorizar los resultados según la ubicación. En una aplicación móvil, puede solicitar al dispositivo su propia ubicación. En cambio, en una aplicación web, puede usar la función `getPosition()`. Pero esta llamada solo funciona en contextos seguros y es posible que no proporcione una ubicación precisa. Además, es posible que el usuario quiera buscar entidades cerca de una ubicación que no sea la suya.

Por lo tanto, nuestra aplicación recurre al servicio Mapas de Bing para obtener la latitud y la longitud desde una ubicación que haya especificado el usuario. El usuario solo tiene que escribir el nombre de un punto de referencia ("Space Needle") o una dirección completa o parcial ("New York City"), y la API de Mapas de Bing le proporciona las coordenadas.

<!-- Remove until we can replace with a sanitized version.
![[Single-page Bing Entity Search app]](media/entity-search-spa-demo.png)
-->

> [!NOTE]
> Los encabezados HTTP y JSON de la parte inferior de la página revelan la respuesta JSON y la información de la solicitud HTTP al hacer clic en ellos. Estos detalles resultan útiles al explorar el servicio.

En la aplicación del tutorial se muestra cómo:

> [!div class="checklist"]
> * Realizar una llamada a Bing Entity Search API en JavaScript
> * Realizar una llamada a la API `locationQuery` de Mapas de Bing en JavaScript
> * Pasar opciones de búsqueda a las llamadas a API
> * Mostrar los resultados de la búsqueda
> * Administrar el id. de cliente de Bing y las claves de suscripción de la API
> * Controlar los errores que se puedan producir

La página del tutorial es completamente independiente: no utiliza marcos, hojas de estilo ni archivos de imagen externos. Usa solo características del lenguaje JavaScript ampliamente compatibles y funciona con las versiones actuales de los principales exploradores web.

En este tutorial, solo se describen determinadas partes del código fuente. El código fuente completo está disponible [en otra página](tutorial-bing-entities-search-single-page-app-source.md). Copie y pegue este código en un editor de texto y guárdelo como `bing.html`.

> [!NOTE]
> Este tutorial es bastante similar al [tutorial de la aplicación Bing Web Search de una sola página](../Bing-Web-Search/tutorial-bing-web-search-single-page-app.md), pero solo se centra en los resultados de búsqueda de imágenes.

## <a name="prerequisites"></a>Requisitos previos

Para seguir con el tutorial, necesita claves de suscripción para Bing Search API y Bing Maps API. 

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* Una vez que tenga la suscripción de Azure:
  * <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesBingSearch-v7"  title="Cree un recurso de Bing Search"  target="_blank">Crear un recurso de Bing Search <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
  * <a href="https://www.microsoft.com/maps/create-a-bing-maps-key.aspx"  title="Cree un recurso de Computer Vision"  target="_blank">Crear un recurso de Bing Maps <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.

## <a name="app-components"></a>Componentes de la aplicación

Al igual que cualquier aplicación web de una sola página, la aplicación del tutorial incluye tres partes:

> [!div class="checklist"]
> * HTML: define la estructura y el contenido de la página
> * CSS: define el aspecto de la página
> * JavaScript: define el comportamiento de la página

Este tutorial no profundiza en la mayor parte de HTML o CSS, ya que es sencillo.

El código HTML contiene el formulario de búsqueda en que el usuario escribe una consulta y elige las opciones de búsqueda. El formulario está conectado al código JavaScript que realmente realiza la búsqueda mediante el atributo `onsubmit` de la etiqueta `<form>`:

```html
<form name="bing" onsubmit="return newBingEntitySearch(this)">
```

El controlador `onsubmit` devuelve `false`, lo que impide que el formulario que se envíe a un servidor. El código JavaScript hace el trabajo de recopilar la información necesaria del formulario y realizar la búsqueda.

La búsqueda se realiza en dos fases. En primer lugar, si el usuario ha escrito una restricción de ubicación, se realiza una consulta de Mapas de Bing para convertirla en coordenadas. A continuación, la devolución de llamada de esta consulta inicia la consulta de Bing Entity Search.

El HTML también contiene las divisiones (etiquetas `<div>` HTML) en que aparecen los resultados de la búsqueda.

## <a name="managing-subscription-keys"></a>Administrar la clave de suscripción

> [!NOTE]
> Esta aplicación requiere claves de suscripción para Bing Search API y la API de Mapas de Bing.

Para evitar tener que incluir las claves de suscripción de Bing Search API y la API de Mapas de Bing en el código, usamos el almacenamiento persistente del explorador para almacenar las claves. Si no se ha almacenado ninguna de las dos claves, las solicitamos y las almacenamos para su uso posterior. Si, más tarde, la API rechaza la clave, se invalidará la clave almacenada y se le volverá a preguntar al usuario en la siguiente búsqueda.

Definimos funciones `storeValue` y `retrieveValue` que usan el objeto `localStorage` (si el explorador lo admite) o una cookie. Nuestra función `getSubscriptionKey()` usa estas funciones para almacenar y recuperar la clave del usuario. Puede usar el punto de conexión global siguiente o el punto de conexión del [subdominio personalizado](../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

```javascript
// cookie names for data we store
SEARCH_API_KEY_COOKIE = "bing-search-api-key";
MAPS_API_KEY_COOKIE   = "bing-maps-api-key";
CLIENT_ID_COOKIE      = "bing-search-client-id";

// API endpoints
SEARCH_ENDPOINT = "https://api.cognitive.microsoft.com/bing/v7.0/entities";
MAPS_ENDPOINT   = "https://dev.virtualearth.net/REST/v1/Locations";

// ... omitted definitions of storeValue() and retrieveValue()

// get stored API subscription key, or prompt if it's not found
function getSubscriptionKey(cookie_name, key_length, api_name) {
    var key = retrieveValue(cookie_name);
    while (key.length !== key_length) {
        key = prompt("Enter " + api_name + " API subscription key:", "").trim();
    }
    // always set the cookie in order to update the expiration date
    storeValue(cookie_name, key);
    return key;
}

function getMapsSubscriptionKey() {
    return getSubscriptionKey(MAPS_API_KEY_COOKIE, 64, "Bing Maps");
}

function getSearchSubscriptionKey() {
    return getSubscriptionKey(SEARCH_API_KEY_COOKIE, 32, "Bing Search");
}
```

La etiqueta HTML `<body>` incluye un atributo `onload` que llama a `getSearchSubscriptionKey()` y `getMapsSubscriptionKey()` cuando la página ha terminado de cargarse. Estas llamadas sirven para solicitar inmediatamente al usuario sus claves, si es que aún no las ha introducido.

```html
<body onload="document.forms.bing.query.focus(); getSearchSubscriptionKey(); getMapsSubscriptionKey();">
```

## <a name="selecting-search-options"></a>Selección de las opciones de búsqueda

![[impreso de Bing Entity Search]](media/entity-search-spa-form.png)

El formulario HTML incluye los siguientes controles:

| Control | Descripción |
|-|-|
|`where`|Un menú desplegable para seleccionar el mercado (ubicación e idioma) que se usa en la búsqueda.|
|`query`|El campo de texto en el que se especifican los términos de búsqueda.|
|`safe`|Una casilla que indica si SafeSearch está activado (restringe los resultados para "adultos").|
|`what`|Un menú para elegir buscar entidades, lugares o ambos.|
|`mapquery`|El campo de texto en el que el usuario puede escribir una dirección completa o parcial, un punto de referencia, etc. para que Bing Entity Search pueda devolver resultados más relevantes.|

> [!NOTE]
> Los resultados de la opción Lugares solo están actualmente disponibles en los Estados Unidos. Los menús `where` y `what` tienen código para aplicar esta restricción. Si elige un mercado fuera de los EE. UU. al seleccionar Lugares en el menú `what`, `what` cambia a Cualquier cosa. Si elige Lugares y selecciona un mercado fuera de los EE. UU. En el menú `where`, `where` cambia a los EE. UU.

Nuestra función de JavaScript `bingSearchOptions()` convierte estos campos en una cadena de consulta parcial en para Bing Search API.

```javascript
// build query options from the HTML form
function bingSearchOptions(form) {

    var options = [];
    options.push("mkt=" + form.where.value);
    options.push("SafeSearch=" + (form.safe.checked ? "strict" : "off"));
    if (form.what.selectedIndex) options.push("responseFilter=" + form.what.value);
    return options.join("&");
}
```

Por ejemplo, la característica Búsqueda segura puede tener el valor `strict`, `moderate` u `off`. El valor predeterminado es `moderate`. Sin embargo, nuestro formulario usa una casilla que solo tiene dos estados. El código JavaScript convierte este valor en `strict` u `off` (no usamos `moderate`).

El campo `mapquery` no se maneja en `bingSearchOptions()` porque se usa en la consulta de ubicación de Mapas de Bing, no para Bing Entity Search.

## <a name="obtaining-a-location"></a>Obtener una ubicación

La API de Mapas de Bing ofrece un [`locationQuery`método](//msdn.microsoft.com/library/ff701711.aspx), que usamos para encontrar la latitud y la longitud de la ubicación que especifica el usuario. Estas coordenadas se pasan a Bing Entity Search API junto con la solicitud del usuario. Los resultados de búsqueda clasifican las entidades y los lugares que están cerca de la ubicación especificada.

No podemos acceder a la API de Mapas de Bing si usamos una consulta ordinaria `XMLHttpRequest` en una aplicación web, porque el servicio no admite consultas de origen cruzado. Afortunadamente, es compatible con JSONP (la "P" se refiere a "padded" o acolchado). Una respuesta JSONP es una respuesta JSON ordinaria encapsulada en una llamada a la función. La solicitud se realiza insertándola con una etiqueta `<script>` en el documento. (La carga de scripts no está sujeta a las directivas de seguridad del explorador).

La función `bingMapsLocate()` crea e inserta la etiqueta `<script>` para la consulta. El segmento `jsonp=bingMapsCallback` de la cadena de consulta especifica el nombre de la función que se llamará con la respuesta.

```javascript
function bingMapsLocate(where) {

    where = where.trim();
    var url = MAPS_ENDPOINT + "?q=" + encodeURIComponent(where) + 
                "&jsonp=bingMapsCallback&maxResults=1&key=" + getMapsSubscriptionKey();

    var script = document.getElementById("bingMapsResult")
    if (script) script.parentElement.removeChild(script);

    // global variable holds reference to timer that will complete the search if the maps query fails
    timer = setTimeout(function() {
        timer = null;
        var form = document.forms.bing;
        bingEntitySearch(form.query.value, "", bingSearchOptions(form), getSearchSubscriptionKey());
    }, 5000);

    script = document.createElement("script");
    script.setAttribute("type", "text/javascript");
    script.setAttribute("id", "bingMapsResult");
    script.setAttribute("src", url);
    script.setAttribute("onerror", "BingMapsCallback(null)");
    document.body.appendChild(script);

    return false;
}
```

> [!NOTE]
> Si la API de Mapas de Bing no responde, nunca se llama a la función `bingMapsCallBack()`. Esto normalmente significa que no se llama a `bingEntitySearch()`, y que no aparecen los resultados de la búsqueda de la entidad. Para evitar esta situación, `bingMapsLocate()` también establece un temporizador para llamar a `bingEntitySearch()` después de cinco segundos. Hay lógica en la función de devolución de llamada para evitar realizar la búsqueda de entidades dos veces.

Cuando la consulta finaliza, se llama a la función `bingMapsCallback()` según lo solicitado.

```javascript
function bingMapsCallback(response) {

    if (timer) {    // we beat the timer; stop it from firing
        clearTimeout(timer);
        timer = null;
    } else {        // the timer beat us; don't do anything
        return; 
    }

    var location = "";
    var name = "";
    var radius = 1000;

    if (response) {
        try {
            if (response.statusCode === 401) {
                invalidateMapsKey();
            } else if (response.statusCode === 200) {
                var resource = response.resourceSets[0].resources[0];
                var coords   = resource.point.coordinates;
                name         = resource.name;

                // the radius is the largest of the distances between the location and the corners
                // of its bounding box (in case it's not in the center) with a minimum of 1 km
                try {
                    var bbox    = resource.bbox;
                    radius  = Math.max(haversineDistance(bbox[0], bbox[1], coords[0], coords[1]),
                                       haversineDistance(coords[0], coords[1], bbox[2], bbox[1]),
                                       haversineDistance(bbox[0], bbox[3], coords[0], coords[1]),
                                       haversineDistance(coords[0], coords[1], bbox[2], bbox[3]), 1000);
                } catch(e) {  }
                var location = "lat:" + coords[0] + ";long:" + coords[1] + ";re:" + Math.round(radius);
            }
        }
        catch (e) { }   // response is unexpected. this isn't fatal, so just don't provide location
    }

    var form = document.forms.bing;
    if (name) form.mapquery.value = name;
    bingEntitySearch(form.query.value, location, bingSearchOptions(form), getSearchSubscriptionKey());

}
```

Además de la latitud y la longitud, la consulta de Bing Entity Search requiere un *radio* que indique la precisión de la información de la ubicación. Calculamos el radio utilizando el *rectángulo delimitador* provisto en la respuesta de Mapas de Bing. El rectángulo delimitador es un rectángulo que rodea toda la ubicación. Por ejemplo, si el usuario escribe `NYC`, el resultado contiene aproximadamente las coordenadas centrales de la ciudad de Nueva York y un rectángulo delimitador que abarca la ciudad. 

Primero calculamos las distancias desde las coordenadas principales hacia cada una de las cuatro esquinas del rectángulo delimitador, mediante la función `haversineDistance()` (no se muestra). Usamos la mayor de estas cuatro distancias a modo de radio. El radio mínimo es de un kilómetro. Este valor también se usa como valor predeterminado si no se proporciona un rectángulo delimitador en la respuesta.

Una vez obtenidos los valores de las coordenadas y el radio, llamamos a `bingEntitySearch()` para realizar la búsqueda real.

## <a name="performing-the-search"></a>Realizar la búsqueda

Dada la consulta, una ubicación, una cadena de opciones y la clave de API, la función `BingEntitySearch()` realiza la solicitud de búsqueda de Bing Entity Search.

```javascript
// perform a search given query, location, options string, and API keys
function bingEntitySearch(query, latlong, options, key) {

    // scroll to top of window
    window.scrollTo(0, 0);
    if (!query.trim().length) return false;     // empty query, do nothing

    showDiv("noresults", "Working. Please wait.");
    hideDivs("pole", "mainline", "sidebar", "_json", "_http", "error");

    var request = new XMLHttpRequest();
    var queryurl = SEARCH_ENDPOINT + "?q=" + encodeURIComponent(query) + "&" + options;

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

    if (latlong) request.setRequestHeader("X-Search-Location", latlong);

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

Tras la correcta finalización de la solicitud HTTP, JavaScript llama nuestro controlador de eventos `load`, la función `handleBingResponse()`, para controlar una solicitud HTTP GET a la API correcta. 

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
            if (jsobj._type === "SearchResponse") {
                renderSearchResults(jsobj);
            } else {
                renderErrorMessage("No search results in JSON response");
            }
        } else {
            renderErrorMessage("Empty response (are you sending too many requests too quickly?)");
        }
    if (divHidden("pole") && divHidden("mainline") && divHidden("sidebar")) 
        showDiv("noresults", "No results.<p><small>Looking for restaurants or other local businesses? Those currently areen't supported outside the US.</small>");
    }

    // Any other HTTP status is an error
    else {
        // 401 is unauthorized; force re-prompt for API key for next request
        if (this.status === 401) invalidateSearchKey();

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
> Una solicitud HTTP correcta *no* indica necesariamente que la búsqueda se haya realizado correctamente. Si se produce un error en la operación de búsqueda, Bing Entity Search API devuelve un código de estado HTTP distinto de 200, e incluye información de error en la respuesta JSON. Además, si la solicitud tenía limitación de frecuencia, la API devuelve una respuesta vacía.

Gran parte del código de las dos funciones anteriores está dedicado al control de errores. Pueden producirse errores en las siguientes fases:

|Fase|Errores posibles|Controlado por|
|-|-|-|
|Crear un objeto de solicitud de JavaScript|Dirección URL no válida|Bloqueo `try`/`catch`|
|Hacer la solicitud|Errores de red, conexiones anuladas|Controladores de eventos `error` y `abort`|
|Realizar la búsqueda|Solicitud no válida, JSON no válido, límites de frecuencia|Pruebas en el controlador de eventos `load`|

Los errores se controlan mediante una llamada a `renderErrorMessage()` con los detalles que se conocen sobre el error. Si la respuesta pasa todas las pruebas de error, llamamos a `renderSearchResults()` para mostrar los resultados de la búsqueda en la página.

## <a name="displaying-search-results"></a>Presentación de los resultados de la búsqueda

Bing Entity Search API [le solicitará mostrar los resultados en un orden específico](use-display-requirements.md). Como la API puede devolver dos tipos diferentes de respuestas, no es suficiente iterar a través de las colecciones `Entities` o `Places` de nivel superior en la respuesta JSON y mostrar esos resultados. (Si solo quiere un tipo de resultado, use el parámetro de consulta `responseFilter`).

En su lugar, usamos la colección `rankingResponse` en los resultados de la búsqueda para ordenar los resultados que se mostrarán. Este objeto se refiere a los artículos en las colecciones `Entitiess` o `Places`.

`rankingResponse` puede contener hasta tres colecciones designadas de resultados de búsqueda; esto es, `pole`, `mainline` y `sidebar`. 

`pole`, si está presente, es el resultado de búsqueda más apropiado y debe mostrarse de manera destacada. `mainline` se refiere a la mayor parte de los resultados de la búsqueda. Los resultados principales se deben mostrar inmediatamente después de `pole` (o antes, si `pole` no está presente). 

Por último. `sidebar` hace referencia a los resultados de búsqueda auxiliares. Se pueden mostrar en una barra lateral o simplemente después de los resultados principales. Hemos elegido este último para nuestra aplicación tutorial.

Todos y cada uno de los elementos de una colección `rankingResponse` hacen referencia a los resultados reales de la búsqueda, y en dos formas diferentes pero equivalentes.

| Elemento | Descripción |
|-|-|
|`id`|`id` parece una dirección URL, pero no debe usarse para los vínculos. El tipo `id` de un resultado de clasificación coincide con el `id` de un elemento del resultado de búsqueda en una colección de respuestas, *o*  una colección de respuestas completa (como, por ejemplo, `Entities`).
|`answerType`<br>`resultIndex`|`answerType` se refiere a la colección de respuestas de nivel superior que contiene el resultado (por ejemplo, `Entities`). `resultIndex` se refiere al índice del resultado dentro de la colección. Si se omite `resultIndex`, el resultado de la clasificación se refiere a toda la colección.

> [!NOTE]
> Para más información acerca de esta parte de la respuesta de búsqueda, consulte [Clasificación de resultados](rank-results.md).

Puede usar el método que sea más conveniente para su aplicación para ubicar el elemento resultado de la búsqueda al que se hace referencia. En el código del tutorial, usamos `answerType` y `resultIndex` para ubicar cada resultado de búsqueda.

Por último, ha llegado el momento de echar un vistazo a nuestra función `renderSearchResults()`. Esta función se itera en las tres colecciones `rankingResponse` que representan las tres secciones de los resultados de la búsqueda. En cada sección, llamamos a `renderResultsItems()` para que represente los resultados de dicha sección.

```javascript
// render the search results given the parsed JSON response
function renderSearchResults(results) {

    // if spelling was corrected, update search field
    if (results.queryContext.alteredQuery) 
        document.forms.bing.query.value = results.queryContext.alteredQuery;

    // for each possible section, render the results from that section
    for (section in {pole: 0, mainline: 0, sidebar: 0}) {
        if (results.rankingResponse[section])
            showDiv(section, renderResultsItems(section, results));
    }
}
```

## <a name="rendering-result-items"></a>Representación de los elementos del resultado

En nuestro código JavaScript es un objeto, `searchItemRenderers`, que contiene *representadores:* funciones que generan código HTML para cada tipo de resultado de búsqueda.

```javascript
searchItemRenderers = { 
    entities: function(item) { ... },
    places: function(item) { ... }
}
```

Una función de representador puede aceptar los parámetros siguientes:

| Parámetro | Descripción |
|-|-|
|`item`|El objeto de JavaScript que contiene las propiedades del elemento, como su dirección URL y la descripción.|
|`index`|El índice del elemento de resultado dentro de su colección.|
|`count`|El número de elementos de la colección del elemento de resultado de la búsqueda.|

Los parámetros `index` y `count` pueden usarse para numerar los resultados, para generar un código HTML especial para el principio o el final de una colección, para insertar saltos de línea después de cierto número de elementos, etc. Si un representador no necesita esta funcionalidad, no es necesario aceptar estos dos parámetros. De hecho, no los usamos en los representadores de nuestra aplicación del tutorial.

Observemos más de cerca el representador `entities`:

```javascript
    entities: function(item) {
        var html = [];
        html.push("<p class='entity'>");
        if (item.image) {
            var img = item.image;
            if (img.hostPageUrl) html.push("<a href='" + img.hostPageUrl + "'>");
            html.push("<img src='" + img.thumbnailUrl +  "' title='" + img.name + "' height=" + img.height + " width= " + img.width + ">");
            if (img.hostPageUrl) html.push("</a>");
            if (img.provider) {
                var provider = img.provider[0];
                html.push("<small>Image from ");
                if (provider.url) html.push("<a href='" + provider.url + "'>");
                html.push(provider.name ? provider.name : getHost(provider.url));
                if (provider.url) html.push("</a>");
                html.push("</small>");
            }
        }
        html.push("<p>");
        if (item.entityPresentationInfo) {
            var pi = item.entityPresentationInfo;
            if (pi.entityTypeHints || pi.entityTypeDisplayHint) {
                html.push("<i>");
                if (pi.entityTypeDisplayHint) html.push(pi.entityTypeDisplayHint);
                else if (pi.entityTypeHints) html.push(pi.entityTypeHints.join("/"));
                html.push("</i> - ");
            }
        }
        html.push(item.description);
        if (item.webSearchUrl) html.push("&nbsp;<a href='" + item.webSearchUrl + "'>More</a>")
        if (item.contractualRules) {
            html.push("<p><small>");
            var rules = [];
            for (var i = 0; i < item.contractualRules.length; i++) {
                var rule = item.contractualRules[i];
                var link = [];
                if (rule.license) rule = rule.license;
                if (rule.url) link.push("<a href='" + rule.url + "'>");
                link.push(rule.name || rule.text || rule.targetPropertyName + " source");
                if (rule.url) link.push("</a>");
                rules.push(link.join(""));
            }
            html.push("License: " + rules.join(" - "));
            html.push("</small>");
        }
        return html.join("");
    }, // places renderer omitted
```

La función del representador de imágenes:

> [!div class="checklist"]
> * Compila la etiqueta HTML `<img>` para mostrar la miniatura de la imagen, si la hubiera. 
> * Compila la etiqueta `<a>` HTML vinculada a la página que contiene la imagen.
> * Compila la descripción que muestra información sobre la imagen y el sitio en que se encuentra.
> * Incorpora una clasificación de la entidad mediante las sugerencias de presentación, si las hubiera.
> * Incluye un vínculo a una búsqueda de Bing para obtener más información sobre la entidad.
> * Muestra cualquier información de licencia o atribución que requieran las fuentes de datos.

## <a name="persisting-client-id"></a>Identificador de cliente persistente

Las respuestas de Bing Search API pueden incluir un encabezado `X-MSEdge-ClientID` que debe devolverse a la API con las solicitudes sucesivas. Si se usan varias instancias de Bing Search API, se debe usar el mismo identificador de cliente con todas ellas, si es posible.

Si se especifica el encabezado `X-MSEdge-ClientID`, las API de Bing podrán asociar todas las búsquedas de un usuario, lo que tiene dos ventajas importantes.

En primer lugar, permite al motor de búsqueda de Bing aplicar un contexto pasado a las búsquedas, con el fin de encontrar los resultados que más satisfagan al usuario. Si un usuario ha buscado previamente términos relacionados con la navegación, por ejemplo, una búsqueda posterior de "muelles" podría devolver información acerca de los muelles que se usan para dejar un velero.

En segundo lugar, Bing puede seleccionar aleatoriamente usuarios para disfrutar de nuevas características antes de que estén disponibles públicamente. Proporcionar el mismo identificador de cliente con cada solicitud garantiza que los usuarios elegidos para ver una característica la vean siempre. Sin el identificador de cliente, el usuario puede ver una característica aparecer y desaparecer, de forma aparentemente aleatoria, en los resultados de búsqueda.

Las directivas de seguridad del explorador (CORS) pueden impedir que el encabezado `X-MSEdge-ClientID` esté disponible para JavaScript. Esta limitación tiene lugar cuando la respuesta a la búsqueda tiene un origen distinto al de la página que la solicitó. En un entorno de producción, debería abordar esta directiva mediante el hospedaje de un script de lado servidor que realice la llamada API en el mismo dominio que la página web. Como el script tiene el mismo origen que la página web, el encabezado `X-MSEdge-ClientID` está disponible para JavaScript.

> [!NOTE]
> En una aplicación web de producción, debe realizar la solicitud del lado servidor de todos modos. En caso contrario, es preciso incluir la clave de Bing Search API en la página web, donde está disponible para cualquiera que vea el origen. Se le facturará todo el uso bajo su clave de suscripción a API, incluso las solicitudes que realicen partes no autorizadas, por lo que es importante no exponer su clave.

Para fines de desarrollo, puede realizar la solicitud de Bing Web Search API a través de un proxy CORS. La respuesta de un proxy de este tipo tiene un encabezado `Access-Control-Expose-Headers` que permite los encabezados de respuesta de las listas y hace que estén disponibles para JavaScript.

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
> [Referencia a Bing Entity Search API](//docs.microsoft.com/rest/api/cognitiveservices/bing-entities-api-v7-reference)

> [!div class="nextstepaction"]
> [Documentación de la API de Mapas de Bing](//msdn.microsoft.com/library/dd877180.aspx)
