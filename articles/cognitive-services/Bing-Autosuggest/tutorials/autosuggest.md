---
title: 'Tutorial: Obtención de resultados de sugerencias automáticas mediante Bing Autosuggest API'
titleSuffix: Azure Cognitive Services
description: En este tutorial, creará una página web que permite a los usuarios consultar el Bing Autosuggest API y mostrar los resultados de la consulta.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-autosuggest
ms.topic: tutorial
ms.date: 03/05/2019
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: a45000c59f975a03f51aa5ee86846c772cbdfaaa
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406476"
---
# <a name="tutorial-get-search-suggestions-on-a-web-page"></a>Tutorial: Obtención de sugerencias de búsqueda en una página web

En este tutorial, crearemos una página web que permita a los usuarios consultar Bing Autosuggest API.

En este tutorial se muestra cómo realizar las siguientes acciones:

> [!div class="checklist"]
> - Realizar una consulta simple en Bing Autosuggest API
> - Visualización de resultados de la consulta

## <a name="prerequisites"></a>Requisitos previos

Para seguir con el tutorial, necesita una clave de suscripción para Bing Autosuggest API. Si no tiene, [cree un recurso de Bing Autosuggest](https://portal.azure.com/#create/Microsoft.CognitiveServicesBingAutosuggest-v7) en Azure Portal.

## <a name="create-a-new-web-page"></a>Creación de una página web

Abra un editor de texto. Cree un nuevo archivo denominado, por ejemplo, autosuggest.html.

## <a name="html-header"></a>Encabezado HTML

Agregue la información del encabezado HTML y comience la sección de script como se indica a continuación.

```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Bing Autosuggest</title>

<style type="text/css">
    html, body, div, p, h1, h2 {font-family: Verdana, "Lucida Sans", sans-serif;}

    html, body, div, p  {font-weight: normal;}
    h1, h2 {font-weight: bold;}
    sup {font-weight: normal;}

    html, body, div, p  {font-size: 12px;}
    h1 {font-size: 20px;}
    h2 {font-size: 16px;}
    h1, h2 {clear: left;}

    img#logo {float: right;
</style>

<script type="text/javascript">
```

## <a name="getsubscriptionkey-function"></a>Función getSubscriptionKey

La función getSubscriptionKey devuelve la clave de Bing Autosuggest API. La recupera del almacenamiento local (es decir, una cookie) o, de ser necesario, la solicita al usuario.

Inicie la función getSubscriptionKey y declare el nombre de la cookie como se indica a continuación.

```html
getSubscriptionKey = function() {

    var COOKIE = "bing-autosuggest-api-key";   // name used to store API key in key/value storage
```

La función auxiliar findCookie devuelve el valor de la cookie especificada; si no se encuentra la cookie, devuelve una cadena vacía.

```html
    function findCookie(name) {
        var cookies = document.cookie.split(";");
        for (var i = 0; i < cookies.length; i++) {
            var keyvalue = cookies[i].split("=");
            if (keyvalue[0].trim() === name) {
                return keyvalue[1];
            }
        }
        return "";
        }
```

La función auxiliar getSubscriptionKeyCookie pide al usuario el valor de la clave de Bing Autosuggest API y devuelve el valor de la clave.

```html
    function getSubscriptionKeyCookie() {
        var key = findCookie(COOKIE);
        while (key.length !== 32) {
            key = prompt("Enter Bing Autosuggest API subscription key:", "").trim();
            var expiry = new Date();
            expiry.setFullYear(expiry.getFullYear() + 2);
            document.cookie = COOKIE + "=" + key.trim() + "; expires=" + expiry.toUTCString();
        }
        return key;
    }
```

En primer lugar, la función auxiliar getSubscriptionKeyLocalStorage intenta recuperar la clave de Bing Autosuggest API buscando la cookie adecuada. Si no se encuentra la cookie, pide al usuario el valor de la clave. A continuación, devuelve el valor de la clave.

```html
    function getSubscriptionKeyLocalStorage() {
        var key = localStorage.getItem(COOKIE) || "";
        while (key.length !== 32)
            key = prompt("Enter Bing Autosuggest API subscription key:", "").trim();
        localStorage.setItem(COOKIE, key)
        return key;
    }
```

La función auxiliar getSubscriptionKey toma un parámetro, **invalidate**. Si **invalidate** es **true**, getSubscriptionKey elimina la cookie que contiene la clave de Bing Autosuggest API. Si **invalidate** es **false**, getSubscriptionKey devuelve el valor de la clave de Bing Autosuggest API.

```html
    function getSubscriptionKey(invalidate) {
        if (invalidate) {
            try {
                localStorage.removeItem(COOKIE);
            } catch (e) {
                document.cookie = COOKIE + "=";
            }
        } else {
            try {
                return getSubscriptionKeyLocalStorage();
            } catch (e) {
                return getSubscriptionKeyCookie();
            }
        }
    }
```

Devuelve la función auxiliar getSubscriptionKey como resultado de la función getSubscriptionKey externa. Cierre la definición de la función getSubscriptionKey externa.

```html
    return getSubscriptionKey;

}();
```

## <a name="helper-functions"></a>Funciones auxiliares

La función auxiliar pre devuelve el texto especificado con un formato previo de etiqueta HTML [pre](https://www.w3schools.com/tags/tag_pre.asp).

```html
function pre(text) {
    return "<pre>" + text.replace(/&/g, "&amp;").replace(/</g, "&lt;") + "</pre>"
}
```

La función renderSearchResults muestra los resultados especificados de Bing Autosuggest API, mediante la impresión con sangría JSON.

```html
function renderSearchResults(results) {
    document.getElementById("results").innerHTML = pre(JSON.stringify(results, null, 2));
}
```

La función renderErrorMessage muestra el mensaje de error especificado y el código de error.

```html
function renderErrorMessage(message, code) {
    if (code)
        document.getElementById("results").innerHTML = "<pre>Status " + code + ": " + message + "</pre>";
    else
        document.getElementById("results").innerHTML = "<pre>" + message + "</pre>";
}
```

## <a name="bingautosuggest-function"></a>Función bingAutosuggest

Se llama a la función bingAutosuggest cada vez que el usuario escribe texto en el campo del formulario HTML.
Toma dos parámetros: el contenido del campo de formulario HTML y la clave de Bing Autosuggest API.

```html
function bingAutosuggest(query, key) {
```

Especifique el punto de conexión de Bing Autosuggest API y declare un objeto XMLHttpRequest, que se usará para enviar solicitudes. Puede usar el punto de conexión global siguiente o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

```html
    var endpoint = "https://api.cognitive.microsoft.com/bing/v7.0/Suggestions";

    var request = new XMLHttpRequest();

    try {
        request.open("GET", endpoint + "?q=" + encodeURIComponent(query));
    }
    catch (e) {
        renderErrorMessage("Bad request");
        return false;
    }
```

Establezca el encabezado **Ocp-Apim-Subscription-Key** en el valor de la clave de Bing Autosuggest API.

```html
    request.setRequestHeader("Ocp-Apim-Subscription-Key", key);
```

Controle la respuesta del punto de conexión. Si el estado es 200 (correcto), muestra los resultados. En caso contrario, muestra la información de error.

```html
    request.addEventListener("load", function() {
        if (this.status === 200) {
            renderSearchResults(JSON.parse(this.responseText));
        }
        else {
            if (this.status === 401) getSubscriptionKey(true);
            renderErrorMessage(this.statusText, this.status);
        }
    });
```

Además, controle los posibles eventos de error del objeto XMLHttpRequest.

```html
    request.addEventListener("error", function() {
        renderErrorMessage("Network error");
    });

    request.addEventListener("abort", function() {
        renderErrorMessage("Request aborted");
    });
```

Envíe la solicitud. Cierre la función bingAutosuggest, la etiqueta **script** y la etiqueta **head**.

```html
    request.send();
    return false;
}
// --></script>

</head>
```

## <a name="html-body"></a>Cuerpo HTML

Cuando se cargue la página web, asegúrese de que se tenga la clave de Bing Autosuggest API, y pídala al usuario, si es necesario.

```html
<body onload="document.forms.bing.query.focus(); getSubscriptionKey();">
```

Muestre el logotipo de Bing.

```html
<img id="logo" align=base src="data:image/png;base64,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">
```

Cree un formulario HTML con un campo de texto. Controle el evento `oninput` y llame a la función `bingAutosuggest()`, pasando el contenido del campo de texto y la clave de Bing Autosuggest API.

```html
<form name="bing" oninput="return bingAutosuggest(this.query.value, getSubscriptionKey())">
    <h2>Autosuggest</h2>
    <input type="text" name="query" size="80" placeholder="Autosuggest" autocomplete=off>
</form>
```

Agregue la etiqueta HTML **div** que se usa para mostrar los resultados. El código de JavaScript definido anteriormente hace referencia a esta etiqueta **div**.

```html
<h2>Results</h2>
<div id="results">
<p>None yet.</p>

</div>

</body>
</html>
```

Guarde el archivo.

## <a name="display-results"></a>Visualización de resultados

Abra la página web en el explorador. En el símbolo del sistema, escriba la clave de suscripción de Bing Autosuggest API. A continuación, escriba una consulta (por ejemplo, "sail") en el cuadro de texto **Autosuggest**. A medida que escribe, la página web se actualiza automáticamente para mostrar los resultados de Autosuggest.

```json
{
  "_type": "Suggestions",
  "queryContext": {
    "originalQuery": "sail"
  },
  "suggestionGroups": [
    {
      "name": "Web",
      "searchSuggestions": [
        {
          "url": "https://www.bing.com/cr?IG=30C49D910FAE478288D54A8DBC5D66F1&CID=122B759B00966D02199C7E9001906C30&rd=1&h=vheQSvKZylM3dlX_B9bQ8-hQEsEJo8zDD2y7H1nsBjE&v=1&r=https%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailor%2bbrinkley%2bcook%26FORM%3dUSBAPI&p=DevEx,5003.1",
          "displayText": "sailor brinkley cook",
          "query": "sailor brinkley cook",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG=30C49D910FAE478288D54A8DBC5D66F1&CID=122B759B00966D02199C7E9001906C30&rd=1&h=EStLqAfxGCa44Ur3jEMXBv-Qp-lXUSFJbkBfnUdKKDg&v=1&r=https%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailor%2bbrinkley%26FORM%3dUSBAPI&p=DevEx,5004.1",
          "displayText": "sailor brinkley",
          "query": "sailor brinkley",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG=30C49D910FAE478288D54A8DBC5D66F1&CID=122B759B00966D02199C7E9001906C30&rd=1&h=gvtP9TS9NwhajSapY2Se6y1eCbP2fq_GiP2n-cxi6OY&v=1&r=https%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailrite%26FORM%3dUSBAPI&p=DevEx,5005.1",
          "displayText": "sailrite",
          "query": "sailrite",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG=30C49D910FAE478288D54A8DBC5D66F1&CID=122B759B00966D02199C7E9001906C30&rd=1&h=c0QOA_j6swCZJy9FxqOwke2KslJE7ZRmMooGClAuCpY&v=1&r=https%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailboats%2bfor%2bsale%26FORM%3dUSBAPI&p=DevEx,5006.1",
          "displayText": "sailboats for sale",
          "query": "sailboats for sale",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG=30C49D910FAE478288D54A8DBC5D66F1&CID=122B759B00966D02199C7E9001906C30&rd=1&h=mnMdREUH20SepmHQH1zlh9Hy_w7jpOlZFm3KG2R_BoA&v=1&r=https%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailing%2banarchy%26FORM%3dUSBAPI&p=DevEx,5007.1",
          "displayText": "sailing anarchy",
          "query": "sailing anarchy",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG=30C49D910FAE478288D54A8DBC5D66F1&CID=122B759B00966D02199C7E9001906C30&rd=1&h=0udadFl0gCTKCp0QmzQTXS3_y08iO8FpwsoKPHPS6kw&v=1&r=https%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailboatdata%26FORM%3dUSBAPI&p=DevEx,5008.1",
          "displayText": "sailboatdata",
          "query": "sailboatdata",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG=30C49D910FAE478288D54A8DBC5D66F1&CID=122B759B00966D02199C7E9001906C30&rd=1&h=BTS0G6AakxntIl9rmbDXtk1n6rQpsZZ99aQ7ClE7dTY&v=1&r=https%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsail%2bsand%2bpoint%26FORM%3dUSBAPI&p=DevEx,5009.1",
          "displayText": "sail sand point",
          "query": "sail sand point",
          "searchKind": "WebSearch"
        },
        {
          "url": "https://www.bing.com/cr?IG=30C49D910FAE478288D54A8DBC5D66F1&CID=122B759B00966D02199C7E9001906C30&rd=1&h=quBMwmKlGwqC5wAU0K7n416plhWcR8zQCi7r-Fw9Y0w&v=1&r=https%3a%2f%2fwww.bing.com%2fsearch%3fq%3dsailflow%26FORM%3dUSBAPI&p=DevEx,5010.1",
          "displayText": "sailflow",
          "query": "sailflow",
          "searchKind": "WebSearch"
        }
      ]
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Requisitos de uso y visualización](../UseAndDisplayRequirements.md)
