---
title: 'Guía de inicio rápido: Envío de una consulta a Bing Local Business Search API mediante Node.js | Microsoft Docs'
titleSuffix: Azure Cognitive Services
description: Empiece a usar Bing Local Business Search API en Node.js.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: article
ms.date: 11/01/2018
ms.author: rosh, v-gedod
ms.openlocfilehash: d0760f89eb98955f7ebb503ce59f904192635f7a
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65796892"
---
# <a name="quickstart-send-a-query-to-the-bing-local-business-search-api-using-nodejs"></a>Inicio rápido: Envío de una consulta a Bing Local Business Search API mediante Node.js

Use esta guía de inicio rápido para empezar a enviar solicitudes a Bing Local Business Search API, que es un servicio de Azure Cognitive Services. Si bien esta aplicación sencilla está escrita en Node.js, la API es un servicio web RESTful compatible con cualquier lenguaje de programación que pueda realizar solicitudes HTTP y analizar JSON.

Esta aplicación de ejemplo obtiene los datos de respuesta local de la API para la consulta de búsqueda `hotel in Bellevue`.

## <a name="prerequisites"></a>Requisitos previos

* La última versión de [Node.js](https://nodejs.org/en/download/).

* La [biblioteca de solicitudes de JavaScript](https://github.com/request/request).

Debe tener una [cuenta de Cognitive Services API](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account) con las API de Bing. La [cuenta de evaluación gratuita](https://azure.microsoft.com/try/cognitive-services/?api=bing-web-search-api) es suficiente para esta guía de inicio rápido. Utilice la clave de acceso proporcionada por la evaluación gratuita.  Consulte también [Precios de Cognitive Services - Bing Search API](https://azure.microsoft.com/pricing/details/cognitive-services/search-api/).

## <a name="code-scenario"></a>Escenario de código

El código siguiente obtiene, define y envía la solicitud. Se implementa en los pasos siguientes:

1. Declare las variables para especificar el punto de conexión por host y ruta de acceso.
2. Especifique la consulta y agregue el parámetro de consulta.
3. Cree una función de controlador para la respuesta.
4. Defina la función de búsqueda que crea la solicitud y agrega el encabezado Ocp-Apim-Subscription-Key.
5. Ejecute la función Search.

Este es el código completo de esta demostración:

```
'use strict';

let https = require('https');

// Replace the subscriptionKey string value with your valid subscription key.
let subscriptionKey = 'your-access-key';

let host = 'api.cognitive.microsoft.com/bing';
let path = '/v7.0/localbusinesses/search';

let mkt = 'en-US';
let q = 'hotel in Bellevue';

let params = '?q=' + encodeURI(q) + "&mkt=" + mkt;

let response_handler = function (response) {
    let body = '';
    response.on('data', function (d) {
        body += d;
    });
    response.on('end', function () {
        let body_ = JSON.parse(body);
        let body__ = JSON.stringify(body_, null, '  ');
        console.log(body__);
    });
    response.on('error', function (e) {
        console.log('Error: ' + e.message);
    });
};

let Search = function () {
    let request_params = {
        method: 'GET',
        hostname: host,
        path: path + params,
        headers: {
            'Ocp-Apim-Subscription-Key': subscriptionKey,
        }
    };

    let req = https.request(request_params, response_handler);
    req.end();
}

Search();

```

## <a name="next-steps"></a>Pasos siguientes

* [Guía de inicio rápido de Local Business Search](local-quickstart.md)
* [Guía de inicio rápido de Local Business Search para Java](local-search-java-quickstart.md)
* [Guía de inicio rápido de Local Business Search para Python](local-search-python-quickstart.md)
