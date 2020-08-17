---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 08/06/2019
ms.author: erhopf
ms.custom: devx-track-javascript
ms.openlocfilehash: c6ca405a6e14f92ab0b09d8f277c99c0c51288bf
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87404637"
---
[!INCLUDE [Prerequisites](prerequisites-nodejs.md)]

[!INCLUDE [Set up and use environment variables](setup-env-variables.md)]

## <a name="create-a-project-and-import-required-modules"></a>Creación de un proyecto e importación de los módulos necesarios

Cree un proyecto con su editor o IDE favoritos, o cree una carpeta en el escritorio. Copie este fragmento de código en un archivo llamado `alt-translations.js` en el proyecto o carpeta.

```javascript
const request = require('request');
const uuidv4 = require('uuid/v4');
```

> [!NOTE]
> Si no ha usado estos módulos deberá instalarlos antes de ejecutar el programa. Para instalar estos paquetes, ejecute: `npm install request uuidv4`.

Estos módulos son necesarios para construir la solicitud HTTP y crear un identificador único para el encabezado `'X-ClientTraceId'`.

## <a name="set-the-subscription-key-and-endpoint"></a>Establecimiento de la clave de suscripción y el punto de conexión

En este ejemplo se intenta leer la clave de suscripción y el punto de conexión de Traductor desde estas variables de entorno: `TRANSLATOR_TEXT_SUBSCRIPTION_KEY` y `TRANSLATOR_TEXT_ENDPOINT`. Si no está familiarizado con las variables de entorno, puede establecer `subscriptionKey` y `endpoint` como cadenas y convertir en comentario las instrucciones condicionales.

Copie este código en el proyecto:

```javascript
var key_var = 'TRANSLATOR_TEXT_SUBSCRIPTION_KEY';
if (!process.env[key_var]) {
    throw new Error('Please set/export the following environment variable: ' + key_var);
}
var subscriptionKey = process.env[key_var];
var endpoint_var = 'TRANSLATOR_TEXT_ENDPOINT';
if (!process.env[endpoint_var]) {
    throw new Error('Please set/export the following environment variable: ' + endpoint_var);
}
var endpoint = process.env[endpoint_var];
```

## <a name="configure-the-request"></a>Configuración de la solicitud

El método `request()`, disponible mediante el módulo de solicitud, nos permite pasar el método HTTP, la dirección URL, los parámetros de la solicitud, los encabezados y el cuerpo de JSON como un objeto `options`. En este fragmento de código, vamos a configurar la solicitud:

>[!NOTE]
> Para más información sobre los puntos de conexión, las rutas y los parámetros de la solicitud, consulte [Traductor 3.0: Búsqueda de diccionario](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-dictionary-lookup).

```javascript
let options = {
    method: 'POST',
    baseUrl: endpoint,
    url: 'dictionary/lookup',
    qs: {
      'api-version': '3.0',
      'from': 'en',
      'to': 'es'
    },
    headers: {
      'Ocp-Apim-Subscription-Key': subscriptionKey,
      'Content-type': 'application/json',
      'X-ClientTraceId': uuidv4().toString()
    },
    body: [{
          'text': 'Elephants'
    }],
    json: true,
};
```

La manera más fácil de autenticar una solicitud es pasar la clave de suscripción como un encabezado `Ocp-Apim-Subscription-Key`, que es el que se usa en este ejemplo. O bien, puede intercambiar la clave de suscripción para un token de acceso y pasar este token como un encabezado `Authorization` para validar la solicitud.

Si usa una suscripción a varios servicios de Cognitive Services, también debe incluir `Ocp-Apim-Subscription-Region` en los encabezados de la solicitud.

Para más información, consulte [Autenticación](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#authentication).

## <a name="make-the-request-and-print-the-response"></a>Realización de solicitud e impresión de la respuesta

A continuación, vamos a crear una solicitud mediante el método `request()`. Toma el objeto `options` que se creó en la sección anterior como el primer argumento y, a continuación, imprime la respuesta JSON embellecida.

```javascript
request(options, function(err, res, body){
    console.log(JSON.stringify(body, null, 4));
});
```

>[!NOTE]
> En este ejemplo, vamos a definir la solicitud HTTP en el objeto `options`. Sin embargo, el módulo de solicitud también admite métodos de conveniencia como `.post` y `.get`. Para más información, consulte [métodos de conveniencia](https://github.com/request/request#convenience-methods).

## <a name="put-it-all-together"></a>Colocación de todo junto

Eso es todo, ha creado un sencillo programa que llama a Traductor y devuelve una respuesta JSON. Ahora es el momento de ejecutar el programa:

```console
node alt-translations.js
```

Si desea comparar su código con el nuestro, el ejemplo completo está disponible en [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-NodeJS).

## <a name="sample-response"></a>Respuesta de muestra

```json
[
    {
        "displaySource": "elephants",
        "normalizedSource": "elephants",
        "translations": [
            {
                "backTranslations": [
                    {
                        "displayText": "elephants",
                        "frequencyCount": 1207,
                        "normalizedText": "elephants",
                        "numExamples": 5
                    }
                ],
                "confidence": 1.0,
                "displayTarget": "elefantes",
                "normalizedTarget": "elefantes",
                "posTag": "NOUN",
                "prefixWord": ""
            }
        ]
    }
]
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha codificado de forma rígida la clave de suscripción en el programa, asegúrese de quitarla cuando haya terminado con esta guía de inicio rápido.

## <a name="next-steps"></a>Pasos siguientes

Eche un vistazo a la referencia de API para comprender todo lo que puede hacer con Traductor.

> [!div class="nextstepaction"]
> [Referencia de API](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference)
