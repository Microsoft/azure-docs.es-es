---
title: 'Tutorial: Carga de imágenes mediante la API Bing Visual Search'
titleSuffix: Azure Cognitive Services
description: Aprenda a cargar imágenes en Bing, obtener información sobre ellas y mostrar la respuesta.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-visual-search
ms.topic: tutorial
ms.date: 03/31/2020
ms.author: scottwhi
ms.custom: devx-track-javascript
ms.openlocfilehash: c2e0e907734f30010690817ddb7bb15527c93bbf
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87404043"
---
# <a name="tutorial-upload-images-to-the-bing-visual-search-api"></a>Tutorial: Carga de imágenes en Bing Visual Search API

Bing Visual Search API le permite buscar en Web imágenes similares a las que carga. Use este tutorial para crear una aplicación web que pueda enviar una imagen a la API y mostrar las conclusiones que devuelve dentro de la página web. Tenga en cuenta que esta aplicación no cumple todos los [requisitos de visualización y uso de Bing](../bing-web-search/use-display-requirements.md) para la utilización de la API.

El código fuente completo de este ejemplo está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/Tutorials/Bing-Visual-Search/BingVisualSearchUploadImage.html) con anotaciones y control de errores adicionales.

En la aplicación del tutorial se muestra cómo:

> [!div class="checklist"]
> * Cargar una imagen en Bing Visual Search API
> * Mostrar los resultados de búsqueda de imágenes en una aplicación web
> * Explorar las distintas conclusiones proporcionadas por la API

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [cognitive-services-bing-image-search-signup-requirements](../../../includes/cognitive-services-bing-visual-search-signup-requirements.md)]

## <a name="create-and-structure-the-webpage"></a>Creación y estructuración de la página web

Crear una página HTML que envíe una imagen a Bing Visual Search API, recibe la información y la muestre. En el editor o entorno de desarrollo integrado que prefiera, cree un archivo llamado"uploaddemo.html". Agregue la siguiente estructura HTML básica al archivo:

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <title>Visual Search Upload Demo</title>
    </head>

    <body>
    </body>
</html>
```

Divida la página en una sección de solicitud, donde el usuario especifica toda la información necesaria para la solicitud, y una sección de respuesta donde se muestra la información. Agregue las siguientes etiquetas `<div>` a `<body>`. La etiqueta `<hr>` separa visualmente ambas secciones:

```html
<div id="requestSection"></div>
<hr />
<div id="responseSection"></div>
```

Agregue una etiqueta `<script>` a la etiqueta `<head>` para que contenga el código JavaScript de la aplicación:

```html
<script>
<\script>
```

## <a name="get-the-upload-file"></a>Obtención del archivo de carga

Para permitir que el usuario seleccione una imagen para cargar, la aplicación usa la etiqueta `<input>` con el atributo de tipo establecido en `file`. La interfaz de usuario necesita dejar claro que la aplicación usa Bing para obtener los resultados de búsqueda.

Agregue el siguiente elemento `<div>` a `requestSection` `<div>`. La entrada del archivo acepta un archivo único de cualquier tipo de imagen (por ejemplo, .jpg, .gif, .png). El evento `onchange` especifica el controlador que se llama cuando el usuario selecciona un archivo.

La etiqueta `<output>` se usa para mostrar una miniatura de la imagen seleccionada:

```html
<div>
    <p>Select image to get insights from Bing:
        <input type="file" accept="image/*" id="uploadImage" name="files[]" size=40 onchange="handleFileSelect('uploadImage')" />
    </p>

    <output id="thumbnail"></output>
</div>
```

## <a name="create-a-file-handler"></a>Creación de un identificador de archivos

Cree una función de controlador que pueda leer en la imagen que quiere cargar. Al recorrer en iteración los archivos en el objeto `FileList`, el controlador debe asegurarse de que el archivo seleccionado es un archivo de imagen y que su tamaño es de 1 MB o menos. Si la imagen es mayor, debe reducir su tamaño para cargarla. Por último, el controlador muestra una miniatura de la imagen:

```javascript
function handleFileSelect(selector) {

    var files = document.getElementById(selector).files; // A FileList object

    for (var i = 0, f; f = files[i]; i++) {

        // Ensure the file is an image file.
        if (!f.type.match('image.*')) {
            alert("Selected file must be an image file.");
            document.getElementById("uploadImage").value = null;
            continue;
        }

        // Image must be <= 1 MB and should be about 1500px.
        if (f.size > 1000000) {
            alert("Image must be less than 1 MB.");
            document.getElementById("uploadImage").value = null;
            continue;
        }

        var reader = new FileReader();

        // Capture the file information.
        reader.onload = (function(theFile) {
            return function(e) {
                var fileOutput = document.getElementById('thumbnail');

                if (fileOutput.childElementCount > 0) {
                    fileOutput.removeChild(fileOutput.lastChild);  // Remove the current pic, if it exists
                }

                // Render thumbnail.
                var span = document.createElement('span');
                span.innerHTML = ['<img class="thumb" src="', e.target.result,
                                    '" title="', escape(theFile.name), '"/>'].join('');
                fileOutput.insertBefore(span, null);
            };
        })(f);

        // Read in the image file as a data URL.
        reader.readAsDataURL(f);
    }
}
```

## <a name="add-and-store-a-subscription-key"></a>Incorporación y almacenamiento de una clave de suscripción

La aplicación requiere una clave de suscripción para realizar llamadas a Bing Visual Search API. En este tutorial, la proporcionará en la interfaz de usuario. Agregue la siguiente etiqueta `<input>` (con el atributo de tipo establecido en texto) al elemento `<body>` inmediatamente debajo de la etiqueta `<output>` del archivo:

```html
    <div>
        <p>Subscription key: 
            <input type="text" id="key" name="subscription" size=40 maxlength="32" />
        </p>
    </div>
```

Con la imagen y la clave de suscripción, puede realizar la llamada a Bing Visual Search para obtener conclusiones sobre la imagen. En este tutorial, la llamada usa el mercado predeterminado (`en-us`) y el valor de búsqueda segura (`moderate`).

Esta aplicación tiene una opción para cambiar estos valores. Agregue el siguiente `<div>` debajo del elemento `<div>` de la clave de suscripción. La aplicación usa una etiqueta `<select>` para proporcionar una lista desplegable de valores de búsqueda segura y mercado. Ambas listas muestran el valor predeterminado.

```html
<div>
    <p><a href="#" onclick="expandCollapse(options.id)">Query options</a></p>

    <div id="options" style="display:none">
        <p style="margin-left: 20px">Market: 
            <select id="mkt">
                <option value="es-AR">Argentina (Spanish)</option>
                <option value="en-AU">Australia (English)</option>
                <option value="de-AT">Austria (German)</option>
                <option value="nl-BE">Belgium (Dutch)</option>
                <option value="fr-BE">Belgium (French)</option>
                <option value="pt-BR">Brazil (Portuguese)</option>
                <option value="en-CA">Canada (English)</option>
                <option value="fr-CA">Canada (French)</option>
                <option value="es-CL">Chile (Spanish)</option>
                <option value="da-DK">Denmark (Danish)</option>
                <option value="fi-FI">Finland (Finnish)</option>
                <option value="fr-FR">France (French)</option>
                <option value="de-DE">Germany (German)</option>
                <option value="zh-HK">Hong Kong SAR(Traditional Chinese)</option>
                <option value="en-IN">India (English)</option>
                <option value="en-ID">Indonesia (English)</option>
                <option value="it-IT">Italy (Italian)</option>
                <option value="ja-JP">Japan (Japanese)</option>
                <option value="ko-KR">Korea (Korean)</option>
                <option value="en-MY">Malaysia (English)</option>
                <option value="es-MX">Mexico (Spanish)</option>
                <option value="nl-NL">Netherlands (Dutch)</option>
                <option value="en-NZ">New Zealand (English)</option>
                <option value="no-NO">Norway (Norwegian)</option>
                <option value="zh-CN">People's Republic of China (Chinese)</option>
                <option value="pl-PL">Poland (Polish)</option>
                <option value="pt-PT">Portugal (Portuguese)</option>
                <option value="en-PH">Philippines (English)</option>
                <option value="ru-RU">Russia (Russian)</option>
                <option value="ar-SA">Saudi Arabia (Arabic)</option>
                <option value="en-ZA">South Africa (English)</option>
                <option value="es-ES">Spain (Spanish)</option>
                <option value="sv-SE">Sweden (Swedish)</option>
                <option value="fr-CH">Switzerland (French)</option>
                <option value="de-CH">Switzerland (German)</option>
                <option value="zh-TW">Taiwan (Traditional Chinese)</option>
                <option value="tr-TR">Turkey (Turkish)</option>
                <option value="en-GB">United Kingdom (English)</option>
                <option value="en-US" selected>United States (English)</option>
                <option value="es-US">United States (Spanish)</option>
            </select>
        </p>
        <p style="margin-left: 20px">Safe search: 
            <select id="safesearch">
                <option value="moderate" selected>Moderate</option>
                <option value="strict">Strict</option>
                <option value="off">off</option>
            </select>
        </p>
    </div>
</div>
```

## <a name="add-search-options-to-the-webpage"></a>Incorporación de opciones de búsqueda a la página web

La aplicación oculta las listas en un elemento `<div>` contraíble que está controlado por el vínculo de opciones de consulta. Al hacer clic en dicho vínculo, el elemento `<div>` se expande para que se puedan ver y modificar las opciones de consulta. Si hace clic en el vínculo por segunda vez, el elemento `<div>` se contrae y queda oculto. El siguiente fragmento de código muestra el controlador `onclick` del vínculo de opciones de consulta. Este controla si el elemento `<div>` está expandido o contraído. Agregue este controlador a la sección `<script>`. El controlador lo usan todas las secciones `<div>` de la demostración.

```javascript
// Contains the toggle state of divs.
var divToggleMap = {};  // divToggleMap['foo'] = 0;  // 1 = show, 0 = hide


// Toggles between showing and hiding the specified div.
function expandCollapse(divToToggle) {
    var div = document.getElementById(divToToggle);

    if (divToggleMap[divToToggle] == 1) {   // if div is expanded
        div.style.display = "none";
        divToggleMap[divToToggle] = 0;
    }
    else {                                  // if div is collapsed
        div.style.display = "inline-block";
        divToggleMap[divToToggle] = 1;
    }
}
```

## <a name="call-the-onclick-handler"></a>Llamada al controlador `onclick`

Agregue el siguiente botón `"Get insights"` debajo de la sección `<div>` de opciones del cuerpo. El botón le permite iniciar la llamada. La hacer clic en el botón, el cursor pasa a ser a un cursor de espera giratorio y se llama al controlador `onclick`.

```html
<p><input type="button" id="query" value="Get insights" onclick="document.body.style.cursor='wait'; handleQuery()" /></p>
```

Agregue el controlador `onclick` del botón, `handleQuery()` a la etiqueta `<script>`.

## <a name="handle-the-query"></a>Gestión de la consulta

El controlador `handleQuery()` garantiza que la clave de suscripción está presente y que tiene 32 caracteres, y que se ha seleccionado una imagen. También borra cualquier información de una consulta anterior. Después, llama a la función `sendRequest()` para realizar la llamada.

```javascript
function handleQuery() {
    var subscriptionKey = document.getElementById('key').value;

    // Make sure user provided a subscription key and image.
    // For this demo, the user provides the key but typically you'd
    // get it from secured storage.
    if (subscriptionKey.length !== 32) {
        alert("Subscription key length is not valid. Enter a valid key.");
        document.getElementById('key').focus();
        return;
    }

    var imagePath = document.getElementById('uploadImage').value;

    if (imagePath.length === 0)
    {
        alert("Please select an image to upload.");
        document.getElementById('uploadImage').focus();
        return;
    }

    var responseDiv = document.getElementById('responseSection');

    // Clear out the response from the last query.
    while (responseDiv.childElementCount > 0) {
        responseDiv.removeChild(responseDiv.lastChild);
    }

    // Send the request to Bing to get insights about the image.
    var f = document.getElementById('uploadImage').files[0];
    sendRequest(f, subscriptionKey);
}
```

## <a name="send-the-search-request"></a>Envío de la solicitud de búsqueda

La función `sendRequest()` da formato a la dirección URL del punto de conexión, establece el encabezado `Ocp-Apim-Subscription-Key` en la clave de suscripción, anexa el binario de la imagen que se va a cargar, especifica el controlador de respuesta y realiza la llamada:

```javascript
function sendRequest(file, key) {
    var market = document.getElementById('mkt').value;
    var safeSearch = document.getElementById('safesearch').value;
    var baseUri = `https://api.cognitive.microsoft.com/bing/v7.0/images/visualsearch?mkt=${market}&safesearch=${safeSearch}`;

    var form = new FormData();
    form.append("image", file);

    var request = new XMLHttpRequest();

    request.open("POST", baseUri);
    request.setRequestHeader('Ocp-Apim-Subscription-Key', key);
    request.addEventListener('load', handleResponse);
    request.send(form);
}
```

## <a name="get-and-handle-the-api-response"></a>Obtención y gestión de la respuesta de API

La función `handleResponse()` gestiona la respuesta de la llamada a Bing Visual Search. Si la llamada se realiza correctamente, analiza la respuesta JSON en las etiquetas individuales, que contienen la información. A continuación, agrega los resultados de búsqueda a la página. Luego, la aplicación crea una sección `<div>` contraíble para cada etiqueta, con el fin de administrar la cantidad de datos que se muestran. Agregue el controlador a la sección `<script>`.

```javascript
function handleResponse() {
    if(this.status !== 200){
        alert("Error calling Bing Visual Search. See console log for details.");
        console.log(this.responseText);
        return;
    }

    var tags = parseResponse(JSON.parse(this.responseText));
    var h4 = document.createElement('h4');
    h4.textContent = 'Bing internet search results';
    document.getElementById('responseSection').appendChild(h4);
    buildTagSections(tags);

    document.body.style.cursor = 'default'; // reset the wait cursor set by query insights button
}
```

### <a name="parse-the-response"></a>Análisis de la respuesta

La función `parseResponse` convierte la respuesta JSON en un objeto de diccionario mediante el recorrido en iteración de `json.tags`.

```javascript
function parseResponse(json) {
    var dict = {};

    for (var i =0; i < json.tags.length; i++) {
        var tag = json.tags[i];

        if (tag.displayName === '') {
            dict['Default'] = JSON.stringify(tag);
        }
        else {
            dict[tag.displayName] = JSON.stringify(tag);
        }
    }

    return(dict);
}
```

### <a name="build-a-tag-section"></a>Compilación de una sección de etiqueta

La función `buildTagSections()` recorre en iteración las etiquetas JSON analizadas y llama a la función `buildDiv()` para crear un elemento `<div>` para cada etiqueta. Cada etiqueta se muestra como un vínculo. Cuando se hace clic en el vínculo, la etiqueta se expande y muestra las conclusiones asociadas con ella. Al hacer clic de nuevo en el vínculo, la sección se contrae.

```javascript
function buildTagSections(tags) {
    for (var tag in tags) {
        if (tags.hasOwnProperty(tag)) {
            var tagSection = buildDiv(tags, tag);
            document.getElementById('responseSection').appendChild(tagSection);
        }
    }  
}

function buildDiv(tags, tag) {
    var tagSection = document.createElement('div');
    tagSection.setAttribute('class', 'subSection');

    var link = document.createElement('a');
    link.setAttribute('href', '#');
    link.setAttribute('style', 'float: left;')
    link.text = tag;
    tagSection.appendChild(link);

    var contentDiv = document.createElement('div');
    contentDiv.setAttribute('id', tag);
    contentDiv.setAttribute('style', 'clear: left;')
    contentDiv.setAttribute('class', 'section');
    tagSection.appendChild(contentDiv);

    link.setAttribute('onclick', `expandCollapse("${tag}")`);
    divToggleMap[tag] = 0;  // 1 = show, 0 = hide

    addDivContent(contentDiv, tag, tags[tag]);

    return tagSection;
}
```

## <a name="display-the-search-results-in-the-webpage"></a>Visualización de los resultados de búsqueda en la página web

La función `buildDiv()` llama a la función `addDivContent` para compilar el contenido del elemento `<div>` contraíble de cada etiqueta.

El contenido de la etiqueta incluye el JSON de la respuesta para la etiqueta. Inicialmente, solo se muestran los 100 primeros caracteres del JSON, pero puede hacer clic en la cadena JSON para mostrar todo el JSON. Si vuelve a hacer clic en ella, la cadena JSON se contrae hasta 100 caracteres.

Después, agregue los tipos de acciones que se encuentran en la etiqueta. Para cada tipo de acción, llame a las funciones adecuadas para agregar su información:

```javascript
function addDivContent(div, tag, json) {

    // Adds the first 100 characters of the json that contains
    // the tag's data. The user can click the text to show the
    // full json. They can click it again to collapse the json.
    var para = document.createElement('p');
    para.textContent = String(json).substr(0, 100) + '...';
    para.setAttribute('title', 'click to expand');
    para.setAttribute('style', 'cursor: pointer;')
    para.setAttribute('data-json', json);
    para.addEventListener('click', function(e) {
        var json = e.target.getAttribute('data-json');

        if (e.target.textContent.length <= 103) {  // 100 + '...'
            e.target.textContent = json;
            para.setAttribute('title', 'click to collapse');
        }
        else {
            para.textContent = String(json).substr(0, 100) + '...';
            para.setAttribute('title', 'click to expand');
        }
    });
    div.appendChild(para); 

    var parsedJson = JSON.parse(json);

    // Loop through all the actions in the tag and display them.
    for (var j = 0; j < parsedJson.actions.length; j++) {
        var action = parsedJson.actions[j];

        var subSectionDiv = document.createElement('div');
        subSectionDiv.setAttribute('class', 'subSection');
        div.appendChild(subSectionDiv);

        var h4 = document.createElement('h4');
        h4.innerHTML = action.actionType;
        subSectionDiv.appendChild(h4);

        if (action.actionType === 'ImageResults') {
            addImageWithWebSearchUrl(subSectionDiv, parsedJson.image, action);
        }
        else if (action.actionType === 'DocumentLevelSuggestions') {
            addRelatedSearches(subSectionDiv, action.data.value);
        }
        else if (action.actionType === 'RelatedSearches') {
            addRelatedSearches(subSectionDiv, action.data.value);
        }
        else if (action.actionType === 'PagesIncluding') {
            addPagesIncluding(subSectionDiv, action.data.value);
        }
        else if (action.actionType === 'VisualSearch') {
            addRelatedImages(subSectionDiv, action.data.value);
        }
        else if (action.actionType === 'Recipes') {
            addRecipes(subSectionDiv, action.data.value);
        }
        else if (action.actionType === 'ShoppingSources') {
            addShopping(subSectionDiv, action.data.offers);
        }
        else if (action.actionType === 'ProductVisualSearch') {
            addProducts(subSectionDiv, action.data.value);
        }
        else if (action.actionType === 'TextResults') {
            addTextResult(subSectionDiv, action);
        }
        else if (action.actionType === 'Entity') {
            addEntity(subSectionDiv, action);
        }
    }
}
```

## <a name="display-insights-for-different-actions"></a>Visualización de conclusiones para distintas acciones

Las siguientes funciones muestran conclusiones para distintas acciones. Las funciones proporcionan una imagen en la que se puede hacer clic o un vínculo en el que se puede hacer clic que le envía a una página web con más información sobre la imagen. Esta página está hospedada en Bing.com o en el sitio web original de la imagen. No todos los datos de las conclusiones se muestran en esta aplicación. Para ver todos los campos disponibles para una conclusión, consulte la referencia [Images - Visual Search](https://aka.ms/bingvisualsearchreferencedoc) (Imágenes: Visual Search).

> [!NOTE]
> Hay una cantidad mínima de información de conclusiones que debe mostrar en la página. Para más información, consulte [Requisitos de uso y visualización de Bing Search API](../bing-web-search/use-display-requirements.md).

### <a name="relatedimages-insights"></a>Conclusiones de RelatedImages

La función `addRelatedImages()` crea un título para cada uno de los sitios web que hospedan la imagen relacionada, para lo que recorre en iteración la lista de acciones `RelatedImages` y anexa una etiqueta `<img>` al elemento `<div>` exterior de cada una de ellas:

```javascript
    function addRelatedImages(div, images) {
        var length = (images.length > 10) ? 10 : images.length;

        // Set the title to the website that hosts the image. The title displays
        // when the user hovers over the image.

        // Make the image clickable. If the user clicks the image, they're taken
        // to the image in Bing.com.

        for (var j = 0; j < length; j++) {
            var img = document.createElement('img');
            img.setAttribute('src', images[j].thumbnailUrl + '&w=120&h=120');
            img.setAttribute('style', 'margin: 20px 20px 0 0; cursor: pointer;');
            img.setAttribute('title', images[j].hostPageDisplayUrl);
            img.setAttribute('data-webSearchUrl', images[j].webSearchUrl)

            img.addEventListener('click', function(e) {
                var url = e.target.getAttribute('data-webSearchUrl');
                window.open(url, 'foo');
            })

            div.appendChild(img);
        }
    }
```

### <a name="pagesincluding-insights"></a>Conclusiones de PagesIncluding

La función `addPagesIncluding()` crea un vínculo para cada uno de los sitios web que hospedan la imagen cargada, para lo cual recorre en iteración la lista de acciones `PagesIncluding` y anexa una etiqueta `<img>` al elemento `<div>` exterior de cada una de ellas:

```javascript

    // Display links to the first 5 webpages that include the image.
    // TODO: Add 'more' link in case the user wants to see all of them.
    function addPagesIncluding(div, pages) {
        var length = (pages.length > 5) ? 5 : pages.length;

        for (var j = 0; j < length; j++) {
            var page = document.createElement('a');
            page.text = pages[j].name;
            page.setAttribute('href', pages[j].hostPageUrl);
            page.setAttribute('style', 'margin: 20px 20px 0 0');
            page.setAttribute('target', '_blank')
            div.appendChild(page);

            div.appendChild(document.createElement('br'));
        }
    }
```

### <a name="relatedsearches-insights"></a>Conclusiones de RelatedSearches

La función `addRelatedSearches()` crea un vínculo para el sitio web que hospeda la imagen, para lo cual recorre en iteración la lista de acciones `RelatedSearches` y anexa una etiqueta `<img>` al elemento `<div>` exterior de cada una de ellas:

```javascript

    // Display the first 10 related searches. Include a link with the image
    // that when clicked, takes the user to Bing.com and displays the 
    // related search results.
    // TODO: Add 'more' link in case the user wants to see all of them.
    function addRelatedSearches(div, relatedSearches) {
        var length = (relatedSearches.length > 10) ? 10 : relatedSearches.length;

        for (var j = 0; j < length; j++) {
            var childDiv = document.createElement('div');
            childDiv.setAttribute('class', 'stackLink');
            div.appendChild(childDiv);

            var img = document.createElement('img');
            img.setAttribute('src', relatedSearches[j].thumbnail.url + '&w=120&h=120');
            img.setAttribute('style', 'margin: 20px 20px 0 0;');
            childDiv.appendChild(img);

            var relatedSearch = document.createElement('a');
            relatedSearch.text = relatedSearches[j].displayText;
            relatedSearch.setAttribute('href', relatedSearches[j].webSearchUrl);
            relatedSearch.setAttribute('target', '_blank');
            childDiv.appendChild(relatedSearch);

        }
    }
```

### <a name="recipes-insights"></a>Conclusiones de Recipes

La función `addRecipes()` crea un vínculo para cada una de las recetas devueltas, para lo cual recorre en iteración la lista de acciones `Recipes` y anexa una etiqueta `<img>` al elemento `<div>` exterior de cada una de ellas:

```javascript
    // Display links to the first 10 recipes. Include the recipe's rating,
    // if available.
    // TODO: Add 'more' link in case the user wants to see all of them.
    function addRecipes(div, recipes) {
        var length = (recipes.length > 10) ? 10 : recipes.length;

        for (var j = 0; j < length; j++) {
            var para = document.createElement('p');

            var recipe = document.createElement('a');
            recipe.text = recipes[j].name;
            recipe.setAttribute('href', recipes[j].url);
            recipe.setAttribute('style', 'margin: 20px 20px 0 0');
            recipe.setAttribute('target', '_blank')
            para.appendChild(recipe);

            if (recipes[j].hasOwnProperty('aggregateRating')) {
                var span = document.createElement('span');
                span.textContent = 'rating: ' + recipes[j].aggregateRating.text;
                para.appendChild(span);
            }

            div.appendChild(para);
        }
    }
```

### <a name="shopping-insights"></a>Conclusiones de Shopping

La función `addShopping()` crea un vínculo con los resultados de la compra devuelta, para lo que recorre en iteración la lista de acciones `RelatedImages` y anexa una etiqueta `<img>` al elemento `<div>` exterior de cada una:

```javascript
    // Display links for the first 10 shopping offers.
    // TODO: Add 'more' link in case the user wants to see all of them.
    function addShopping(div, offers) {
        var length = (offers.length > 10) ? 10 : offers.length;

        for (var j = 0; j < length; j++) {
            var para = document.createElement('p');

            var offer = document.createElement('a');
            offer.text = offers[j].name;
            offer.setAttribute('href', offers[j].url);
            offer.setAttribute('style', 'margin: 20px 20px 0 0');
            offer.setAttribute('target', '_blank')
            para.appendChild(offer);

            var span = document.createElement('span');
            span.textContent = 'by ' + offers[j].seller.name + ' | ' + offers[j].price + ' ' + offers[j].priceCurrency;
            para.appendChild(span);

            div.appendChild(para);
        }
    }
```

### <a name="products-insights"></a>Conclusiones de Products

La función `addProducts()` crea un vínculo con los resultados de los productos devueltos, para lo que recorre en iteración la lista de acciones `Products` y anexa una etiqueta `<img>` al elemento `<div>` exterior de cada una:

```javascript

    // Display the first 10 related products. Display a clickable image of the
    // product that takes the user to Bing.com search results for the product.
    // If there are any offers associated with the product, provide links to the offers.
    // TODO: Add 'more' link in case the user wants to see all of them.
    function addProducts(div, products) {
        var length = (products.length > 10) ? 10 : products.length;

        for (var j = 0; j < length; j++) {
            var childDiv = document.createElement('div');
            childDiv.setAttribute('class', 'stackLink');
            div.appendChild(childDiv);

            var img = document.createElement('img');
            img.setAttribute('src', products[j].thumbnailUrl + '&w=120&h=120');
            img.setAttribute('title', products[j].name);
            img.setAttribute('style', 'margin: 20px 20px 0 0; cursor: pointer;');
            img.setAttribute('data-webSearchUrl', products[j].webSearchUrl)
            img.addEventListener('click', function(e) {
                var url = e.target.getAttribute('data-webSearchUrl');
                window.open(url, 'foo');
            })
            childDiv.appendChild(img);

            if (products[j].insightsMetadata.hasOwnProperty('aggregateOffer')) {
                if (products[j].insightsMetadata.aggregateOffer.offerCount > 0) {
                    var offers = products[j].insightsMetadata.aggregateOffer.offers;

                    // Show all the offers. Not all markets provide links to offers.
                    for (var i = 0; i < offers.length; i++) {  
                        var para = document.createElement('p');

                        var offer = document.createElement('a');
                        offer.text = offers[i].name;
                        offer.setAttribute('href', offers[i].url);
                        offer.setAttribute('style', 'margin: 20px 20px 0 0');
                        offer.setAttribute('target', '_blank')
                        para.appendChild(offer);

                        var span = document.createElement('span');
                        span.textContent = 'by ' + offers[i].seller.name + ' | ' + offers[i].price + ' ' + offers[i].priceCurrency;
                        para.appendChild(span);

                        childDiv.appendChild(para);
                    }
                }
                else {  // Otherwise, just show the lowest price that Bing found.
                    var offer = products[j].insightsMetadata.aggregateOffer;

                    var para = document.createElement('p');
                    para.textContent = `${offer.name} | ${offer.lowPrice} ${offer.priceCurrency}`; 

                    childDiv.appendChild(para);
                }
            }
        }
    }
```

### <a name="textresult-insights"></a>Conclusiones de TextResult

La función `addTextResult()` muestra cualquier texto que se haya reconocido en la imagen:

```javascript

    function addTextResult(div, action) {
        var text = document.createElement('p');
        text.textContent = action.displayName;
        div.appendChild(text);
    }
```

La función `addEntity()` muestra un vínculo que lleva al usuario a Bing.com, donde puede obtener detalles sobre el tipo de entidad de la imagen, si se ha detectado alguno:

```javascript
    // If the image is of a person, the tag might include an entity
    // action type. Display a link that takes the user to Bing.com
    // where they can get details about the entity.
    function addEntity(div, action) {
        var entity = document.createElement('a');
        entity.text = action.displayName;
        entity.setAttribute('href', action.webSearchUrl);
        entity.setAttribute('style', 'margin: 20px 20px 0 0');
        entity.setAttribute('target', '_blank');
        div.appendChild(entity);
    }
```

La función `addImageWithWebSearchUrl()` muestra una imagen en la que se puede hacer clic del elemento `<div>` que lleva al usuario a los resultados de búsqueda en Bing.com:

```javascript
    function addImageWithWebSearchUrl(div, image, action) {
        var img = document.createElement('img');
        img.setAttribute('src', image.thumbnailUrl + '&w=120&h=120');
        img.setAttribute('style', 'margin: 20px 20px 0 0; cursor: pointer;');
        img.setAttribute('data-webSearchUrl', action.webSearchUrl);
        img.addEventListener('click', function(e) {
            var url = e.target.getAttribute('data-webSearchUrl');
            window.open(url, 'foo');
        })
        div.appendChild(img);
    }

```

## <a name="add-a-css-style"></a>Incorporación de un estilo CSS

Agregue la siguiente sección `<style>` a la etiqueta `<head>` para organizar el diseño de la página web:

```html
        <style>

            .thumb {
                height: 75px;
                border: 1px solid #000;
            }

            .stackLink {
                width:180px;
                min-height:210px;
                display:inline-block;
            }
            .stackLink a {
                float:left;
                clear:left;
            }

            .section {
                float:left;
                display:none;
            }

            .subSection {
                clear:left;
                float:left;
            }

        </style>
```

## <a name="next-steps"></a>Pasos siguientes

>[!div class="nextstepaction"]
> [Tutorial: Búsqueda de imágenes similares de búsquedas anteriores mediante ImageInsightsToken](./tutorial-visual-search-insights-token.md)
