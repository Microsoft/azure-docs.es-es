---
title: 'Inicio rápido: Generación de una miniatura (REST, Node.js)'
titleSuffix: Azure Cognitive Services
description: En esta guía de inicio rápido, generará una miniatura de una imagen mediante Computer Vision API con Node.js.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: quickstart
ms.date: 08/05/2020
ms.author: pafarley
ms.custom: seodec18, devx-track-javascript
ms.openlocfilehash: 84713cfec4d57167b912c472911c60eeda213550
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87834939"
---
# <a name="quickstart-generate-a-thumbnail-using-the-computer-vision-rest-api-and-nodejs"></a>Inicio rápido: Generación de una miniatura mediante la API REST Computer Vision y Node.js

En esta guía de inicio rápido, generará una miniatura a partir de una imagen mediante la API REST de Computer Vision. Con el método [Obtener miniatura](https://westcentralus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44/operations/56f91f2e778daf14a499e1fb), puede generar una miniatura de una imagen. Debe especificar el alto y el ancho, que pueden ser diferentes a la relación de aspecto de la imagen de entrada. Computer Vision usa el recorte inteligente para identificar el área de interés de forma inteligente y generar coordenadas de recorte en función de esa región.

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
1. Guarde el código como un archivo con una extensión `.js`. Por ejemplo, `get-thumbnail.js`.
1. Abra una ventana de símbolo del sistema.
1. En el símbolo del sistema, utilice el comando `node` para ejecutar el archivo. Por ejemplo, `node get-thumbnail.js`.

```javascript
'use strict';

const fs = require('fs');
const request = require('request').defaults({ encoding: null });

let subscriptionKey = process.env['COMPUTER_VISION_SUBSCRIPTION_KEY'];
let endpoint = process.env['COMPUTER_VISION_ENDPOINT']

var uriBase = endpoint + 'vision/v3.0/generateThumbnail';

const imageUrl = 'https://upload.wikimedia.org/wikipedia/commons/9/94/Bloodhound_Puppy.jpg';

// Request parameters.
const params = {
    'width': '100',
    'height': '100',
    'smartCropping': 'true'
};

// Construct the request
const options = {
    uri: uriBase,
    qs: params,
    body: '{"url": ' + '"' + imageUrl + '"}',
    headers: {
        'Content-Type': 'application/json',
        'Ocp-Apim-Subscription-Key' : subscriptionKey
    }
}

// Post the request and get the response (an image stream)
request.post(options, (error, response, body) => {
    // Write the stream to file
    var buf = Buffer.from(body, 'base64');
    fs.writeFile('thumbnail.png', buf, function (err) {
        if (err) throw err;
    });

    console.log('Image saved')
});
```

## <a name="examine-the-response"></a>Examen de la respuesta

Se mostrará un elemento emergente de la imagen en miniatura.
Se devuelve una respuesta correcta como datos binarios, que representan los datos de imagen para la miniatura. Si se produce un error en la solicitud, la respuesta se muestra en la ventana de consola. La respuesta de la solicitud con error contiene un código de error y un mensaje para ayudar a determinar qué salió mal.

## <a name="next-steps"></a>Pasos siguientes

Después, explore las versiones de Computer Vision API que se usan para analizar una imagen, detectar celebridades y sitios emblemáticos, crear una miniatura y extraer texto impreso y manuscrito.

> [!div class="nextstepaction"]
> [Explore Computer Vision API](https://westus.dev.cognitive.microsoft.com/docs/services/5adf991815e1060e6355ad44)
