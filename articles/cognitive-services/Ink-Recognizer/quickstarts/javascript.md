---
title: 'Inicio rápido: Reconocimiento de la entrada de lápiz digital con la API REST Ink Recognizer y Node.js'
titleSuffix: Azure Cognitive Services
description: En este inicio rápido, usará API Ink Recognizer y JavaScript para comenzar a reconocer los trazos de entrada de lápiz digital.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: ink-recognizer
ms.topic: quickstart
ms.date: 08/24/2020
ms.author: aahi
ms.custom: devx-track-javascript
ms.openlocfilehash: b8d8464fb9defb64c039ef9026812de89436091c
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89050918"
---
# <a name="quickstart-recognize-digital-ink-with-the-ink-recognizer-rest-api-and-javascript"></a>Inicio rápido: Reconocimiento de la entrada de lápiz digital con la API REST Ink Recognizer y JavaScript

[!INCLUDE [ink-recognizer-deprecation](../includes/deprecation-note.md)]

Use este inicio rápido para empezar a usar la API Ink Recognizer en trazos de entrada de lápiz digital. Esta aplicación de JavaScript envía una solicitud de API que contiene los datos del trazo de tinta con formato JSON y muestra la respuesta.

Si bien esta aplicación está escrita en JavaScript y se ejecuta en el explorador web, la API es un servicio web RESTful compatible con la mayoría de los lenguajes de programación.

Normalmente se llamaría a la API desde una aplicación de entrada de lápiz digital. En este inicio rápido se envían los datos del trazo de entrada de lápiz digital para la siguiente muestra escrita a mano desde un archivo JSON.

![imagen de un texto manuscrito](../media/handwriting-sample.jpg)

El código fuente de este inicio rápido está disponible en [GitHub](https://go.microsoft.com/fwlink/?linkid=2089905).

## <a name="prerequisites"></a>Requisitos previos

- Un explorador web
- Los datos de trazo de entrada de lápiz para este inicio rápido se pueden encontrar en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/javascript/InkRecognition/quickstart/example-ink-strokes.json).

### <a name="create-an-ink-recognizer-resource"></a>Creación de un recurso de Ink Recognizer

[!INCLUDE [creating an ink recognizer resource](../includes/setup-instructions.md)]

## <a name="create-a-new-application"></a>Creación de una aplicación

1. En su IDE o editor favoritos, cree un archivo `.html`. A continuación, agréguele HTML básico para el código que se agregará más adelante.
    
    ```html
    <!DOCTYPE html>
    <html>
    
        <head>
            <script type="text/javascript">
            </script>
        </head>
        
        <body>
        </body>
    
    </html>
    ```

2. Dentro de la etiqueta `<body>`, agregue el siguiente HTML:
    1. Dos áreas de texto para mostrar la solicitud JSON y la respuesta.
    2. Un botón para llamar a la función `recognizeInk()` que se creará más adelante.
    
    ```HTML
    <!-- <body>-->
        <h2>Send a request to the Ink Recognition API</h2>
        <p>Request:</p>
        <textarea id="request" style="width:800px;height:300px"></textarea>
        <p>Response:</p>
        <textarea id="response" style="width:800px;height:300px"></textarea>
        <br>
        <button type="button" onclick="recognizeInk()">Recognize Ink</button>
    <!--</body>-->
    ```

## <a name="load-the-example-json-data"></a>Carga de los datos JSON de ejemplo

1. Dentro de la etiqueta `<script>`, cree una variable para el archivo JSON de muestra. A continuación, cree una función de JavaScript denominada `openFile()` que abre un explorador de archivos para que pueda seleccionar el archivo JSON. Cuando se hace clic en el botón `Recognize ink`, se llama a esta función y se empieza a leer el archivo.
2. Use una función `onload()` del objeto `FileReader` para procesar el archivo de forma asincrónica. 
    1. Reemplace cualquier carácter `\n` o `\r` en el archivo con una cadena vacía. 
    2. Use `JSON.parse()` para convertir el texto en un archivo JSON válido.
    3. Actualice el cuadro de texto `request` de la aplicación. Use `JSON.stringify()` para dar formato a la cadena JSON. 
    
    ```javascript
    var sampleJson = "";
    function openFile(event) {
        var input = event.target;
    
        var reader = new FileReader();
        reader.onload = function(){
            sampleJson = reader.result.replace(/(\\r\\n|\\n|\\r)/gm, "");
            sampleJson = JSON.parse(sampleJson);
            document.getElementById('request').innerHTML = JSON.stringify(sampleJson, null, 2);
        };
        reader.readAsText(input.files[0]);
    };
    ```

## <a name="send-a-request-to-the-ink-recognizer-api"></a>Envíe una solicitud a la API Ink Recognizer

1. Dentro de la etiqueta `<script>`, cree una función denominada `recognizeInk()`. Esta función más adelante llamará a la API y actualizará la página con la respuesta. Agregue el código con los pasos siguientes dentro de esta función. 
        
    ```javascript
    function recognizeInk() {
    // add the code from the below steps here 
    }
    ```

    1. Cree variables para la dirección URL del punto de conexión, la clave de suscripción y el archivo JSON de muestra. A continuación, cree un objeto `XMLHttpRequest` para enviar la solicitud de API. 
        
        ```javascript
        // Replace the below URL with the correct one for your subscription. 
        // Your endpoint can be found in the Azure portal. For example: "https://<your-custom-subdomain>.cognitiveservices.azure.com";
        var SERVER_ADDRESS = process.env["INK_RECOGNITION_ENDPOINT"];
        var ENDPOINT_URL = SERVER_ADDRESS + "/inkrecognizer/v1.0-preview/recognize";
        var SUBSCRIPTION_KEY = process.env["INK_RECOGNITION_SUBSCRIPTION_KEY"];
        var xhttp = new XMLHttpRequest();
        ```
    2. Cree la función de devolución para el objeto `XMLHttpRequest`. Esta función analizará la respuesta de API desde una solicitud correcta y la mostrará en la aplicación. 
            
        ```javascript
        function returnFunction(xhttp) {
            var response = JSON.parse(xhttp.responseText);
            console.log("Response: %s ", response);
            document.getElementById('response').innerHTML = JSON.stringify(response, null, 2);
        }
        ```
    3. Cree la función de error para el objeto de solicitud. Esta función registra el error en la consola. 
            
        ```javascript
        function errorFunction() {
            console.log("Error: %s, Detail: %s", xhttp.status, xhttp.responseText);
        }
        ```

    4. Cree una función para la propiedad `onreadystatechange` del objeto de solicitud. Cuando el estado de preparación del objeto de solicitud cambie, se aplicarán las funciones de valor devuelto y de error anteriores.
            
        ```javascript
        xhttp.onreadystatechange = function () {
            if (this.readyState === 4) {
                if (this.status === 200) {
                    returnFunction(xhttp);
                } else {
                    errorFunction(xhttp);
                }
            }
        };
        ```
    
    5. Envíe la solicitud API. Agregue la clave de suscripción al encabezado `Ocp-Apim-Subscription-Key` y establezca `content-type` en `application/json`.
    
        ```javascript
        xhttp.open("PUT", ENDPOINT_URL, true);
        xhttp.setRequestHeader("Ocp-Apim-Subscription-Key", SUBSCRIPTION_KEY);
        xhttp.setRequestHeader("content-type", "application/json");
        xhttp.send(JSON.stringify(sampleJson));
        };
        ```

## <a name="run-the-application-and-view-the-response"></a>Ejecución de la aplicación y visualización de la respuesta

Esta aplicación se puede ejecutar dentro del explorador web. Se devuelve una respuesta correcta en formato JSON. También puede encontrar la respuesta JSON en [GitHub](https://github.com/Azure-Samples/cognitive-services-REST-api-samples/blob/master/javascript/InkRecognition/quickstart/example-response.json):

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Referencia de API de REST](https://go.microsoft.com/fwlink/?linkid=2089907)

Para ver el funcionamiento de la API Ink Recognition en una aplicación de entrada de lápiz digital, eche un vistazo a las siguientes aplicaciones de ejemplo en GitHub:
* [C# y Plataforma universal de Windows (UWP)](https://go.microsoft.com/fwlink/?linkid=2089803)  
* [C# y Windows Presentation Foundation (WPF)](https://go.microsoft.com/fwlink/?linkid=2089804)
* [Aplicación para explorador web de JavaScript](https://go.microsoft.com/fwlink/?linkid=2089908)       
* [Aplicación móvil Java y Android](https://go.microsoft.com/fwlink/?linkid=2089906)
* [Aplicación móvil iOS y SWIFT](https://go.microsoft.com/fwlink/?linkid=2089805)
