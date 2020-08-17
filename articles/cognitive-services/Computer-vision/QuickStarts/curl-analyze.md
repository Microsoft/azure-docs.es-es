---
title: 'Inicio rápido: Análisis de imágenes remotas (REST, cURL)'
titleSuffix: Azure Cognitive Services
description: En esta guía de inicio rápido, analizará una imagen remota mediante Computer Vision API con cURL.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: quickstart
ms.date: 08/05/2020
ms.author: pafarley
ms.custom: seodec18
ms.openlocfilehash: 82b2ae42c500702fa95d65f86d1b3934c2a1d361
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835381"
---
# <a name="quickstart-analyze-a-remote-image-using-the-computer-vision-rest-api-and-curl"></a>Inicio rápido: Análisis de imágenes remotas mediante la API REST Computer Vision y cURL

En este inicio rápido, analizará una imagen almacenada de forma remota para extraer características visuales con la API REST de Computer Vision. Con el [método de análisis de imagen](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa), puede extraer características visuales basadas en el contenido de una imagen.

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
    1. Si lo desea, cambie la dirección URL de la imagen del cuerpo de la solicitud (`http://upload.wikimedia.org/wikipedia/commons/3/3c/Shaki_waterfall.jpg\`) por la dirección URL de una imagen diferente que desee analizar.
1. Abra una ventana de símbolo del sistema.
1. Pegue el comando del editor de texto en la ventana del símbolo del sistema y después ejecute el comando.

```bash
curl -H "Ocp-Apim-Subscription-Key: <subscriptionKey>" -H "Content-Type: application/json" "https://westcentralus.api.cognitive.microsoft.com/vision/v3.0/analyze?visualFeatures=Categories,Description&details=Landmarks" -d "{\"url\":\"http://upload.wikimedia.org/wikipedia/commons/3/3c/Shaki_waterfall.jpg\"}"
```

## <a name="examine-the-response"></a>Examen de la respuesta

Se devuelve una respuesta correcta en JSON. La aplicación de ejemplo analiza y muestra una respuesta correcta en la ventana del símbolo del sistema, parecida a la del ejemplo siguiente:

```json
{
  "categories": [
    {
      "name": "outdoor_water",
      "score": 0.9921875,
      "detail": {
        "landmarks": []
      }
    }
  ],
  "description": {
    "tags": [
      "nature",
      "water",
      "waterfall",
      "outdoor",
      "rock",
      "mountain",
      "rocky",
      "grass",
      "hill",
      "covered",
      "hillside",
      "standing",
      "side",
      "group",
      "walking",
      "white",
      "man",
      "large",
      "snow",
      "grazing",
      "forest",
      "slope",
      "herd",
      "river",
      "giraffe",
      "field"
    ],
    "captions": [
      {
        "text": "a large waterfall over a rocky cliff",
        "confidence": 0.916458423253597
      }
    ]
  },
  "requestId": "b6e33879-abb2-43a0-a96e-02cb5ae0b795",
  "metadata": {
    "height": 959,
    "width": 1280,
    "format": "Jpeg"
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

Explore las versiones de Computer Vision API que se usan para analizar una imagen, detectar celebridades y sitios emblemáticos, crear una miniatura y extraer texto impreso y manuscrito. Para experimentar rápidamente con la versión de Computer Vision API, pruebe la [consola de pruebas de Open API](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa/console).

> [!div class="nextstepaction"]
> [Explore Computer Vision API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44)
