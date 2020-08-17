---
title: 'Inicio rápido: Revisión ortográfica con la API REST y Node.js: Bing Spell Check'
titleSuffix: Azure Cognitive Services
description: Empiece a utilizar la API REST de Bing Spell Check para la revisión ortográfica y gramatical con este inicio rápido.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-spell-check
ms.topic: quickstart
ms.date: 05/21/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: aaaa571928556a6972d3136ef4cacaa3bd4cb798
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87405014"
---
# <a name="quickstart-check-spelling-with-the-bing-spell-check-rest-api-and-nodejs"></a>Inicio rápido: Revisión ortográfica con la API REST de Bing Spell Check y Node.js

Use este inicio rápido para realizar la primera llamada a la API REST de Bing Spell Check. Esta sencilla aplicación de JavaScript envía una solicitud a la API y devuelve una lista de correcciones sugeridas. 

Aunque esta aplicación está escrita en JavaScript, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación. El código fuente de esta aplicación está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/nodejs/Search/BingSpellCheckv7.js).

## <a name="prerequisites"></a>Prerrequisitos

* [Node.js 6](https://nodejs.org/en/download/) o posterior.

[!INCLUDE [cognitive-services-bing-spell-check-signup-requirements](../../../../includes/cognitive-services-bing-spell-check-signup-requirements.md)]


## <a name="create-and-initialize-a-project"></a>Creación e inicialización de un proyecto

1. Cree un archivo de JavaScript en su IDE o editor favorito. Establezca la rigurosidad y que se requiera `https`. A continuación, cree variables para el host, la ruta de acceso y la clave de suscripción del punto de conexión de API. Puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```javascript
    'use strict';
    let https = require ('https');

    let host = 'api.cognitive.microsoft.com';
    let path = '/bing/v7.0/spellcheck';
    let key = '<ENTER-KEY-HERE>';
    ```

2. Cree las variables de los parámetros de búsqueda y el texto que desea comprobar: 

   1. Asigne el código de mercado al parámetro `mkt` con el operador `=`. El código de mercado es el código del país desde el que se realiza la solicitud. 

   1. Agregue el parámetro `mode` con el operador `&` y, a continuación, asigne el modo de revisión ortográfica. El modo puede ser `proof` (detecta la mayoría de los errores ortográficos y gramaticales) o `spell` (detecta la mayoría de los errores ortográficos, pero no todos los gramaticales).

    ```javascript
    let mkt = "en-US";
    let mode = "proof";
    let text = "Hollo, wrld!";
    let query_string = "?mkt=" + mkt + "&mode=" + mode;
    ```

## <a name="create-the-request-parameters"></a>Creación de los parámetros de solicitud

Para crear los parámetros de solicitud, cree un objeto con un método `POST`. Agregue la ruta de acceso mediante la anexión de la ruta de acceso del punto de conexión y la cadena de consulta. A continuación, agregue la clave de suscripción al encabezado `Ocp-Apim-Subscription-Key`.

```javascript
let request_params = {
   method : 'POST',
   hostname : host,
   path : path + query_string,
   headers : {
   'Content-Type' : 'application/x-www-form-urlencoded',
   'Content-Length' : text.length + 5,
      'Ocp-Apim-Subscription-Key' : key,
   }
};
```

## <a name="create-a-response-handler"></a>Creación de un controlador de respuesta

Cree una función denominada `response_handler` que tome la respuesta JSON de la API e imprímala. Cree una variable para el cuerpo de la respuesta. Anexe la respuesta cuando se reciba una marca `data`, mediante `response.on()`. Después de recibir una marca `end`, imprima el cuerpo JSON en la consola.

```javascript
let response_handler = function (response) {
    let body = '';
    response.on ('data', function (d) {
        body += d;
    });
    response.on ('end', function () {
        let body_ = JSON.parse (body);
        console.log (body_);
    });
    response.on ('error', function (e) {
        console.log ('Error: ' + e.message);
    });
};
```

## <a name="send-the-request"></a>Envío de la solicitud

Llame a la API mediante `https.request()` con los parámetros de solicitud y el controlador de respuesta. Escriba el texto en la API y finalice la solicitud.

```javascript
let req = https.request (request_params, response_handler);
req.write ("text=" + text);
req.end ();
```


## <a name="run-the-application"></a>Ejecución de la aplicación

1. Compile y ejecute el proyecto.

1. Si usa la línea de comandos, utilice los siguientes comandos para compilar y ejecutar la aplicación:

   ```bash
   node <FILE_NAME>.js
   ```


## <a name="example-json-response"></a>Ejemplo de respuesta JSON

Se devuelve una respuesta correcta en JSON, como se muestra en el siguiente ejemplo:

```json
{
   "_type": "SpellCheck",
   "flaggedTokens": [
      {
         "offset": 0,
         "token": "Hollo",
         "type": "UnknownToken",
         "suggestions": [
            {
               "suggestion": "Hello",
               "score": 0.9115257530801
            },
            {
               "suggestion": "Hollow",
               "score": 0.858039839213461
            },
            {
               "suggestion": "Hallo",
               "score": 0.597385084464481
            }
         ]
      },
      {
         "offset": 7,
         "token": "wrld",
         "type": "UnknownToken",
         "suggestions": [
            {
               "suggestion": "world",
               "score": 0.9115257530801
            }
         ]
      }
   ]
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Creación de una aplicación web de una sola página](../tutorials/spellcheck.md)

- [¿Qué es Bing Spell Check API?](../overview.md)
- [Referencia de la API Bing Spell Check v7](https://docs.microsoft.com/rest/api/cognitiveservices-bingsearch/bing-spell-check-api-v7-reference)
