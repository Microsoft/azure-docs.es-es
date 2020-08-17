---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 08/06/2019
ms.author: erhopf
ms.custom: devx-track-javascript
ms.openlocfilehash: 1f2159f3f4c88c7ddafab63a7780f4dbf41ca130
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87405437"
---
[!INCLUDE [Prerequisites](prerequisites-nodejs.md)]

[!INCLUDE [Set up and use environment variables](setup-env-variables.md)]

## <a name="create-a-project-and-import-required-modules"></a>Creación de un proyecto e importación de los módulos necesarios

Cree un proyecto con su IDE o editor favorito. A continuación, copie este fragmento de código en un archivo llamado `get-languages.js`.

```javascript
const request = require('request');
const uuidv4 = require('uuid/v4');
```

> [!NOTE]
> Si no ha usado estos módulos deberá instalarlos antes de ejecutar el programa. Para instalar estos paquetes, ejecute: `npm install request uuidv4`.

Estos módulos son necesarios para construir la solicitud HTTP y crear un identificador único para el encabezado `'X-ClientTraceId'`.

## <a name="set-the-endpoint"></a>Establecimiento del punto de conexión

En este ejemplo se intenta leer el punto de conexión de Traductor desde una variable de entorno: `TRANSLATOR_TEXT_ENDPOINT`. Si no está familiarizado con las variables de entorno, puede establecer `endpoint` como una cadena y convertir en comentario la instrucción condicional.

```javascript
lorum ipsum
```

## <a name="configure-the-request"></a>Configuración de la solicitud

El método `request()`, disponible mediante el módulo de solicitud, nos permite pasar el método HTTP, la dirección URL, los parámetros de la solicitud, los encabezados y el cuerpo de JSON como un objeto `options`. En este fragmento de código, vamos a configurar la solicitud:

>[!NOTE]
> Para más información sobre los puntos de conexión, las rutas y los parámetros de la solicitud, consulte [Traductor 3.0: Idiomas](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-languages).

```javascript
let options = {
    method: 'GET',
    baseUrl: endpoint,
    url: 'languages',
    qs: {
      'api-version': '3.0',
    },
    headers: {
      'Content-type': 'application/json',
      'X-ClientTraceId': uuidv4().toString()
    },
    json: true,
};
```

Si usa una suscripción a varios servicios de Cognitive Services, también debe incluir `Ocp-Apim-Subscription-Region` en los parámetros de la solicitud. [Más información sobre la autenticación con la suscripción a varios servicios](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#authentication).

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
node get-languages.js
```

Si desea comparar su código con el nuestro, el ejemplo completo está disponible en [GitHub](https://github.com/MicrosoftTranslator/Text-Translation-API-V3-NodeJS).

## <a name="sample-response"></a>Respuesta de muestra

Busque la abreviatura del país o región en esta [lista de idiomas](https://docs.microsoft.com/azure/cognitive-services/translator/language-support).

Este ejemplo se ha truncado para mostrar un fragmento del resultado:

```json
{
    "translation": {
        "af": {
            "name": "Afrikaans",
            "nativeName": "Afrikaans",
            "dir": "ltr"
        },
        "ar": {
            "name": "Arabic",
            "nativeName": "العربية",
            "dir": "rtl"
        },
        ...
    },
    "transliteration": {
        "ar": {
            "name": "Arabic",
            "nativeName": "العربية",
            "scripts": [
                {
                    "code": "Arab",
                    "name": "Arabic",
                    "nativeName": "العربية",
                    "dir": "rtl",
                    "toScripts": [
                        {
                            "code": "Latn",
                            "name": "Latin",
                            "nativeName": "اللاتينية",
                            "dir": "ltr"
                        }
                    ]
                },
                {
                    "code": "Latn",
                    "name": "Latin",
                    "nativeName": "اللاتينية",
                    "dir": "ltr",
                    "toScripts": [
                        {
                            "code": "Arab",
                            "name": "Arabic",
                            "nativeName": "العربية",
                            "dir": "rtl"
                        }
                    ]
                }
            ]
        },
      ...
    },
    "dictionary": {
        "af": {
            "name": "Afrikaans",
            "nativeName": "Afrikaans",
            "dir": "ltr",
            "translations": [
                {
                    "name": "English",
                    "nativeName": "English",
                    "dir": "ltr",
                    "code": "en"
                }
            ]
        },
        "ar": {
            "name": "Arabic",
            "nativeName": "العربية",
            "dir": "rtl",
            "translations": [
                {
                    "name": "English",
                    "nativeName": "English",
                    "dir": "ltr",
                    "code": "en"
                }
            ]
        },
      ...
    }
}
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha codificado de forma rígida la clave de suscripción en el programa, asegúrese de quitarla cuando haya terminado con esta guía de inicio rápido.

## <a name="next-steps"></a>Pasos siguientes

Eche un vistazo a la referencia de API para comprender todo lo que puede hacer con Traductor.

> [!div class="nextstepaction"]
> [Referencia de API](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference)
