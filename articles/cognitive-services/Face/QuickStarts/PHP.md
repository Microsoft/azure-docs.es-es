---
title: 'Inicio rápido: Detección de caras en una imagen mediante la API REST y PHP'
titleSuffix: Azure Cognitive Services
description: En este inicio rápido se muestra cómo detectar caras en una imagen mediante la API REST de Face con PHP.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: quickstart
ms.date: 08/05/2020
ms.author: pafarley
ms.openlocfilehash: 7ae54d1d1c649da510c9653acbd7f118069d366c
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87833919"
---
# <a name="quickstart-detect-faces-in-an-image-using-the-rest-api-and-php"></a>Inicio rápido: Detección de caras en una imagen mediante la API REST y PHP

En este inicio rápido, usará la API REST de Azure Face con PHP para detectar caras humanas en una imagen.

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesFace"  title="Creación de un recurso de Face"  target="_blank">cree un recurso de Face <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a Face API. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* Un editor de código como [Visual Studio Code](https://code.visualstudio.com/download).
* Paquete PHP [HTTP_Request2](https://pear.php.net/package/HTTP_Request2).
* Un explorador web habilitado para PHP. Si no se ha configurado esto, puede hacerlo mediante la instalación y la configuración de [XAMPP](https://www.apachefriends.org/) en su máquina.

## <a name="initialize-the-html-file"></a>Inicialización del archivo HTML

Cree un nuevo archivo HTML, *detectFaces.html*, y agregue el código siguiente.

```html
<html>
    <head>
        <title>Face Detect Sample</title>
    </head>
    <body></body>
</html>
```

## <a name="write-the-php-script"></a>Escritura del script PHP

Agregue el siguiente código dentro del elemento `body` del documento. Este código configura una interfaz de usuario básica con un campo de dirección URL, un botón **Analizar cara**, un panel de respuesta y un panel de información de la imagen.

```php
<?php
// Replace <Subscription Key> with a valid subscription key.
$ocpApimSubscriptionKey = '<Subscription Key>';

// Replace <My Endpoint String> with the string in your endpoint URL.
$uriBase = 'https:/<My Endpoint String>.com/face/v1.0/';

$imageUrl =
    'https://upload.wikimedia.org/wikipedia/commons/3/37/Dagestani_man_and_woman.jpg';

// This sample uses the PHP5 HTTP_Request2 package
// (https://pear.php.net/package/HTTP_Request2).
require_once 'HTTP/Request2.php';

$request = new Http_Request2($uriBase . '/detect');
$url = $request->getUrl();

$headers = array(
    // Request headers
    'Content-Type' => 'application/json',
    'Ocp-Apim-Subscription-Key' => $ocpApimSubscriptionKey
);
$request->setHeader($headers);

$parameters = array(
    // Request parameters
    'returnFaceId' => 'true',
    'returnFaceLandmarks' => 'false',
    'returnFaceAttributes' => 'age,gender,headPose,smile,facialHair,glasses,' .
        'emotion,hair,makeup,occlusion,accessories,blur,exposure,noise');
$url->setQueryVariables($parameters);

$request->setMethod(HTTP_Request2::METHOD_POST);

// Request body parameters
$body = json_encode(array('url' => $imageUrl));

// Request body
$request->setBody($body);

try
{
    $response = $request->send();
    echo "<pre>" .
        json_encode(json_decode($response->getBody()), JSON_PRETTY_PRINT) . "</pre>";
}
catch (HttpException $ex)
{
    echo "<pre>" . $ex . "</pre>";
}
?>
```

Deberá actualizar el campo `subscriptionKey` con el valor de la clave de suscripción y debe cambiar la cadena `uriBase` para que contenga la cadena del punto de conexión correcta. El campo `returnFaceAttributes` especifica qué atributos de cara recuperar; puede ser conveniente cambiar esta cadena según su uso previsto.

[!INCLUDE [subdomains-note](../../../../includes/cognitive-services-custom-subdomains-note.md)]

## <a name="run-the-script"></a>Ejecute el script.

Abra el archivo en un explorador web compatible con PHP. Debe obtener una cadena JSON de datos de cara, similar a la siguiente.

```json
[
    {
        "faceId": "e93e0db1-036e-4819-b5b6-4f39e0f73509",
        "faceRectangle": {
            "top": 621,
            "left": 616,
            "width": 195,
            "height": 195
        },
        "faceAttributes": {
            "smile": 0,
            "headPose": {
                "pitch": 0,
                "roll": 6.8,
                "yaw": 3.7
            },
            "gender": "male",
            "age": 37,
            "facialHair": {
                "moustache": 0.4,
                "beard": 0.4,
                "sideburns": 0.1
            },
            "glasses": "NoGlasses",
            "emotion": {
                "anger": 0,
                "contempt": 0,
                "disgust": 0,
                "fear": 0,
                "happiness": 0,
                "neutral": 0.999,
                "sadness": 0.001,
                "surprise": 0
            },
            "blur": {
                "blurLevel": "high",
                "value": 0.89
            },
            "exposure": {
                "exposureLevel": "goodExposure",
                "value": 0.51
            },
            "noise": {
                "noiseLevel": "medium",
                "value": 0.59
            },
            "makeup": {
                "eyeMakeup": true,
                "lipMakeup": false
            },
            "accessories": [],
            "occlusion": {
                "foreheadOccluded": false,
                "eyeOccluded": false,
                "mouthOccluded": false
            },
            "hair": {
                "bald": 0.04,
                "invisible": false,
                "hairColor": [
                    {
                        "color": "black",
                        "confidence": 0.98
                    },
                    {
                        "color": "brown",
                        "confidence": 0.87
                    },
                    {
                        "color": "gray",
                        "confidence": 0.85
                    },
                    {
                        "color": "other",
                        "confidence": 0.25
                    },
                    {
                        "color": "blond",
                        "confidence": 0.07
                    },
                    {
                        "color": "red",
                        "confidence": 0.02
                    }
                ]
            }
        }
    },
    {
        "faceId": "37c7c4bc-fda3-4d8d-94e8-b85b8deaf878",
        "faceRectangle": {
            "top": 693,
            "left": 1503,
            "width": 180,
            "height": 180
        },
        "faceAttributes": {
            "smile": 0.003,
            "headPose": {
                "pitch": 0,
                "roll": 2,
                "yaw": -2.2
            },
            "gender": "female",
            "age": 56,
            "facialHair": {
                "moustache": 0,
                "beard": 0,
                "sideburns": 0
            },
            "glasses": "NoGlasses",
            "emotion": {
                "anger": 0,
                "contempt": 0.001,
                "disgust": 0,
                "fear": 0,
                "happiness": 0.003,
                "neutral": 0.984,
                "sadness": 0.011,
                "surprise": 0
            },
            "blur": {
                "blurLevel": "high",
                "value": 0.83
            },
            "exposure": {
                "exposureLevel": "goodExposure",
                "value": 0.41
            },
            "noise": {
                "noiseLevel": "high",
                "value": 0.76
            },
            "makeup": {
                "eyeMakeup": false,
                "lipMakeup": false
            },
            "accessories": [],
            "occlusion": {
                "foreheadOccluded": false,
                "eyeOccluded": false,
                "mouthOccluded": false
            },
            "hair": {
                "bald": 0.06,
                "invisible": false,
                "hairColor": [
                    {
                        "color": "black",
                        "confidence": 0.99
                    },
                    {
                        "color": "gray",
                        "confidence": 0.89
                    },
                    {
                        "color": "other",
                        "confidence": 0.64
                    },
                    {
                        "color": "brown",
                        "confidence": 0.34
                    },
                    {
                        "color": "blond",
                        "confidence": 0.07
                    },
                    {
                        "color": "red",
                        "confidence": 0.03
                    }
                ]
            }
        }
    }
]
```

## <a name="next-steps"></a>Pasos siguientes

Explore API Face, que se utiliza para detectar rostros humanos en una imagen, delimitarlos con rectángulos y devolver atributos como la edad y el sexo.

> [!div class="nextstepaction"]
> [Versiones de Face API](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236)
