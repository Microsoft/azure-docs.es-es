---
title: 'Inicio rápido: Extracción de texto impreso (REST, cURL)'
titleSuffix: Azure Cognitive Services
description: En esta guía de inicio rápido, extraerá texto impreso de una imagen mediante Computer Vision API con cURL.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: quickstart
ms.date: 08/05/2020
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 66aa123628001ea88d047419bba9d3bc8976c4b4
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835364"
---
# <a name="quickstart-extract-printed-text-ocr-using-the-computer-vision-rest-api-and-curl"></a>Inicio rápido: Extracción de texto impreso (OCR) mediante la API REST Computer Vision y cURL

En este inicio rápido, extraerá texto impreso con el reconocimiento óptico de caracteres (OCR) de una imagen con la API REST de Computer Vision. Con el método [OCR](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fc), puede detectar texto impreso en cualquier imagen y extraer los caracteres reconocidos en una secuencia de caracteres que pueda usar una máquina.

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/). 
* [cURL](https://curl.haxx.se/)
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Creación de un recurso de Computer Vision"  target="_blank">cree un recurso de Computer Vision <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
  * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación al servicio Computer Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
  * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="create-and-run-the-sample-command"></a>Creación y ejecución del comando de ejemplo

Para crear y ejecutar el ejemplo, siga estos pasos:

1. Copie el comando siguiente en un editor de texto.
1. Realice los siguientes cambios en el comando donde sea necesario:
    1. Reemplace el valor de `<subscriptionKey>` por la clave de suscripción.
    1. Reemplace la primera parte de la dirección URL de la solicitud (`westcentralus`) por el texto de la dirección URL de su punto de conexión.
        [!INCLUDE [Custom subdomains notice](../../../../includes/cognitive-services-custom-subdomains-note.md)]
    1. Si lo desea, cambie la dirección URL de la imagen del cuerpo de la solicitud (`https://upload.wikimedia.org/wikipedia/commons/thumb/a/af/Atomist_quote_from_Democritus.png/338px-Atomist_quote_from_Democritus.png\`) por la dirección URL de una imagen diferente que desee analizar.
1. Abra una ventana de símbolo del sistema.
1. Pegue el comando del editor de texto en la ventana del símbolo del sistema y después ejecute el comando.

```bash
curl -H "Ocp-Apim-Subscription-Key: <subscriptionKey>" -H "Content-Type: application/json" "https://westcentralus.api.cognitive.microsoft.com/vision/v3.0/ocr?language=unk&detectOrientation=true" -d "{\"url\":\"https://upload.wikimedia.org/wikipedia/commons/thumb/a/af/Atomist_quote_from_Democritus.png/338px-Atomist_quote_from_Democritus.png\"}"
```

## <a name="examine-the-response"></a>Examen de la respuesta

Se devuelve una respuesta correcta en JSON. La aplicación de ejemplo analiza y muestra una respuesta correcta en la ventana del símbolo del sistema, parecida a la del ejemplo siguiente:

```json
{
  "language": "en",
  "orientation": "Up",
  "textAngle": 0,
  "regions": [
    {
      "boundingBox": "21,16,304,451",
      "lines": [
        {
          "boundingBox": "28,16,288,41",
          "words": [
            {
              "boundingBox": "28,16,288,41",
              "text": "NOTHING"
            }
          ]
        },
        {
          "boundingBox": "27,66,283,52",
          "words": [
            {
              "boundingBox": "27,66,283,52",
              "text": "EXISTS"
            }
          ]
        },
        {
          "boundingBox": "27,128,292,49",
          "words": [
            {
              "boundingBox": "27,128,292,49",
              "text": "EXCEPT"
            }
          ]
        },
        {
          "boundingBox": "24,188,292,54",
          "words": [
            {
              "boundingBox": "24,188,292,54",
              "text": "ATOMS"
            }
          ]
        },
        {
          "boundingBox": "22,253,297,32",
          "words": [
            {
              "boundingBox": "22,253,105,32",
              "text": "AND"
            },
            {
              "boundingBox": "144,253,175,32",
              "text": "EMPTY"
            }
          ]
        },
        {
          "boundingBox": "21,298,304,60",
          "words": [
            {
              "boundingBox": "21,298,304,60",
              "text": "SPACE."
            }
          ]
        },
        {
          "boundingBox": "26,387,294,37",
          "words": [
            {
              "boundingBox": "26,387,210,37",
              "text": "Everything"
            },
            {
              "boundingBox": "249,389,71,27",
              "text": "else"
            }
          ]
        },
        {
          "boundingBox": "127,431,198,36",
          "words": [
            {
              "boundingBox": "127,431,31,29",
              "text": "is"
            },
            {
              "boundingBox": "172,431,153,36",
              "text": "opinion."
            }
          ]
        }
      ]
    }
  ]
}
```

## <a name="next-steps"></a>Pasos siguientes

Explore las versiones de Computer Vision API que se usan para analizar una imagen, detectar celebridades y sitios emblemáticos, crear una miniatura y extraer texto impreso y manuscrito. Para experimentar rápidamente con la versión de Computer Vision API, pruebe la [consola de pruebas de Open API](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa/console).

> [!div class="nextstepaction"]
> [Explore Computer Vision API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44)
