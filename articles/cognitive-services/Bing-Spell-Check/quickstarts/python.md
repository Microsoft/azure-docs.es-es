---
title: 'Inicio rápido: Revisión ortográfica con la API REST y Python: Bing Spell Check'
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
ms.custom: devx-track-python
ms.openlocfilehash: e121aaf6725c179189b25dff6534b019c5de730c
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852742"
---
# <a name="quickstart-check-spelling-with-the-bing-spell-check-rest-api-and-python"></a>Inicio rápido: Revisión ortográfica con la API REST de Bing Spell Check y Python.

Use este inicio rápido para realizar la primera llamada a la API REST de Bing Spell Check. Esta sencilla aplicación de Python envía una solicitud a la API y devuelve una lista de correcciones sugeridas. 

Aunque esta aplicación está escrita en Python, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación. El código fuente de esta aplicación está disponible en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/python/Search/BingEntitySearchv7.py).

## <a name="prerequisites"></a>Prerrequisitos

* Python [3.x](https://www.python.org)

[!INCLUDE [cognitive-services-bing-spell-check-signup-requirements](../../../../includes/cognitive-services-bing-spell-check-signup-requirements.md)]

## <a name="initialize-the-application"></a>Inicialización de la aplicación

1. Cree un archivo de Python en el IDE o editor que prefiera y agregue las siguientes instrucciones de importación:

   ```python
   import requests
   import json
   ```

2. Cree variables para el texto cuya ortografía quiere revisar, para la clave de suscripción y para el punto de conexión de Bing Spell Check. Puede usar el punto de conexión global en el código siguiente, o el punto de conexión del [subdominio personalizado](../../../cognitive-services/cognitive-services-custom-subdomains.md) que se muestra en Azure Portal para el recurso.

    ```python
    api_key = "<ENTER-KEY-HERE>"
    example_text = "Hollo, wrld" # the text to be spell-checked
    endpoint = "https://api.cognitive.microsoft.com/bing/v7.0/SpellCheck"
    ```

## <a name="create-the-parameters-for-the-request"></a>Creación de los parámetros para la solicitud

1. Cree un diccionario con `text` como clave y el texto como valor.

    ```python
    data = {'text': example_text}
    ```

2. Agregue los parámetros de la solicitud: 

   1. Asigne el código de mercado al parámetro `mkt` con el operador `=`. El código de mercado es el código del país desde el que se realiza la solicitud. 

   1. Agregue el parámetro `mode` con el operador `&` y, a continuación, asigne el modo de revisión ortográfica. El modo puede ser `proof` (detecta la mayoría de los errores ortográficos y gramaticales) o `spell` (detecta la mayoría de los errores ortográficos, pero no todos los gramaticales). 
 
    ```python
    params = {
        'mkt':'en-us',
        'mode':'proof'
        }
    ```

3. Agregue un encabezado `Content-Type` y la clave de suscripción al encabezado `Ocp-Apim-Subscription-Key`.

    ```python
    headers = {
        'Content-Type': 'application/x-www-form-urlencoded',
        'Ocp-Apim-Subscription-Key': api_key,
        }
    ```

## <a name="send-the-request-and-read-the-response"></a>Envío de la solicitud y lectura de la respuesta

1. Envíe la solicitud POST, mediante la biblioteca de solicitudes.

    ```python
    response = requests.post(endpoint, headers=headers, params=params, data=data)
    ```

2. Obtenga la respuesta JSON e imprímala.

    ```python
    json_response = response.json()
    print(json.dumps(json_response, indent=4))
    ```


## <a name="run-the-application"></a>Ejecución de la aplicación

Si usa la línea de comandos, utilice los siguientes comandos para ejecutar la aplicación:

```bash
python <FILE_NAME>.py
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
