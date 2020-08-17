---
title: 'Inicio rápido: Análisis de una imagen remota con la API REST y Node.js'
titleSuffix: Azure Cognitive Services
description: En esta guía de inicio rápido, analizará una imagen mediante Computer Vision API con Node.js.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: quickstart
ms.date: 08/05/2020
ms.author: pafarley
ms.custom: seodec18, seo-javascript-september2018, seo-javascript-october2019, devx-track-javascript
ms.openlocfilehash: d6962ae364c8f86d5069c9ec47eed6ffc5d2d419
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835007"
---
# <a name="quickstart-analyze-a-remote-image-using-the-computer-vision-rest-api-with-nodejs"></a>Inicio rápido: Análisis de imágenes remotas mediante la API REST Computer Vision con Node.js

En este inicio rápido, va a analizar una imagen almacenada de forma remota para extraer características visuales mediante el uso de la API REST de Computer Vision con Node.js. Con el [método de análisis de imagen](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fa), puede extraer características visuales basadas en el contenido de una imagen.

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/).
* [Node.js](https://nodejs.org) 4.x o posterior 
* [npm](https://www.npmjs.com/) 
* Una vez que tenga la suscripción de Azure, <a href="https://portal.azure.com/#create/Microsoft.CognitiveServicesComputerVision"  title="Creación de un recurso de Computer Vision"  target="_blank">cree un recurso de Computer Vision <span class="docon docon-navigate-external x-hidden-focus"></span></a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación al servicio Computer Vision. En una sección posterior de este mismo inicio rápido pegará la clave y el punto de conexión en el código siguiente.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.
* [Cree las variables de entorno](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#configure-an-environment-variable-for-authentication) `COMPUTER_VISION_SUBSCRIPTION_KEY` y `COMPUTER_VISION_ENDPOINT` para la clave y la dirección URL del punto de conexión, respectivamente.

## <a name="create-and-run-the-sample"></a>Creación y ejecución del ejemplo

Para crear y ejecutar el ejemplo, siga estos pasos:

1. Instale el paquete [`request`](https://www.npmjs.com/package/request) de npm.
   1. Abra una ventana del símbolo del sistema como administrador.
   1. Ejecute el siguiente comando:

      ```console
      npm install request
      ```

   1. Una vez que el paquete se haya instalado correctamente, cierre la ventana del símbolo del sistema.

1. Copie el código siguiente en un editor de texto.
1. También puede reemplazar el valor de `imageUrl` por la dirección URL de una imagen diferente que desee analizar.
1. Si lo desea, reemplace el valor del parámetro de la solicitud `language` por un idioma diferente.
1. Guarde el código como un archivo con una extensión `.js`. Por ejemplo, `analyze-image.js`.
1. Abra una ventana de símbolo del sistema.
1. En el símbolo del sistema, utilice el comando `node` para ejecutar el archivo. Por ejemplo, `node analyze-image.js`.

```javascript
'use strict';

const request = require('request');

let subscriptionKey = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
let endpoint = process.env['COMPUTER_VISION_ENDPOINT']
if (!subscriptionKey) { throw new Error('Set your environment variables for your subscription key and endpoint.'); }

var uriBase = endpoint + 'vision/v3.0/analyze';

const imageUrl =
    'https://upload.wikimedia.org/wikipedia/commons/3/3c/Shaki_waterfall.jpg';

// Request parameters.
const params = {
    'visualFeatures': 'Categories,Description,Color',
    'details': '',
    'language': 'en'
};

const options = {
    uri: uriBase,
    qs: params,
    body: '{"url": ' + '"' + imageUrl + '"}',
    headers: {
        'Content-Type': 'application/json',
        'Ocp-Apim-Subscription-Key' : subscriptionKey
    }
};

request.post(options, (error, response, body) => {
  if (error) {
    console.log('Error: ', error);
    return;
  }
  let jsonResponse = JSON.stringify(JSON.parse(body), null, '  ');
  console.log('JSON Response\n');
  console.log(jsonResponse);
});
```

## <a name="examine-the-response"></a>Examen de la respuesta

Se devuelve una respuesta correcta en JSON. El ejemplo analiza y muestra una respuesta correcta en la ventana del símbolo del sistema, parecida a la del ejemplo siguiente:

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
  "color": {
    "dominantColorForeground": "Grey",
    "dominantColorBackground": "Green",
    "dominantColors": [
      "Grey",
      "Green"
    ],
    "accentColor": "4D5E2F",
    "isBwImg": false
  },
  "requestId": "81b4e400-e3c1-41f1-9020-e6871ad9f0ed",
  "metadata": {
    "height": 959,
    "width": 1280,
    "format": "Jpeg"
  }
}
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sea necesario, elimine el archivo y después desinstale el paquete `request` de npm. Para desinstalar el paquete, realice los siguientes pasos:

1. Abra una ventana del símbolo del sistema como administrador.
2. Ejecute el siguiente comando:

   ```console
   npm uninstall request
   ```

3. Una vez que el paquete se haya desinstalado correctamente, cierre la ventana del símbolo del sistema.

## <a name="next-steps"></a>Pasos siguientes

Después, explore las versiones de las API de Computer Vision que se usan para analizar una imagen, detectar personajes y sitios emblemáticos, crear una miniatura y extraer texto impreso y escrito a mano.

> [!div class="nextstepaction"]
> [Explore Computer Vision API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44)
