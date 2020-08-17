---
title: 'Inicio rápido: Configuración de la plataforma para usar el SDK de voz para JavaScript (Browser): servicio de voz'
titleSuffix: Azure Cognitive Services
description: Use esta guía para configurar la plataforma para usar JavaScript (Browser) con el SDK del servicio de voz.
services: cognitive-services
author: markamos
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: include
ms.date: 10/11/2019
ms.author: erhopf
ms.custom: devx-track-javascript
ms.openlocfilehash: 54fe96ecee99f3de37667d255b57b89c34aecc14
ms.sourcegitcommit: 42107c62f721da8550621a4651b3ef6c68704cd3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87406437"
---
En esta guía se muestra cómo instalar el [SDK de voz](~/articles/cognitive-services/speech-service/speech-sdk.md) para JavaScript para su uso con una página web.

[!INCLUDE [License Notice](~/includes/cognitive-services-speech-service-license-notice.md)]

## <a name="create-a-new-website-folder"></a>Creación de una nueva carpeta Website

Cree una nueva carpeta vacía. En caso de que desee hospedar el ejemplo en un servidor web, asegúrese de que este puede acceder a la carpeta.

## <a name="unpack-the-speech-sdk-for-javascript-into-that-folder"></a>Desempaquete el SDK de Voz para JavaScript en esa carpeta

Descargue el SDK de Voz en forma de [paquete .zip](https://aka.ms/csspeech/jsbrowserpackage) y desempaquételo en la carpeta recién creada. Como resultado se desempaquetan cinco archivos:
* `microsoft.cognitiveservices.speech.sdk.bundle.js` Una versión en lenguaje natural del SDK de voz.
* `microsoft.cognitiveservices.speech.sdk.bundle.js.map` Un archivo de asignación que se usa para depurar código del SDK.
* `microsoft.cognitiveservices.speech.sdk.bundle.d.ts` Definiciones de objetos para su uso con TypeScript
* `microsoft.cognitiveservices.speech.sdk.bundle-min.js` Una versión reducida del SDK de voz.
* `speech-processor.js` Código para mejorar el rendimiento en algunos exploradores.

## <a name="create-an-indexhtml-page"></a>Creación de una página index.html

Cree un nuevo archivo llamado `index.html` en la carpeta y ábralo con un editor de texto.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [windows](../quickstart-list.md)]