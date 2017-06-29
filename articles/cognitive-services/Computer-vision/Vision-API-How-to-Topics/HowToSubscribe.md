---
title: Get subscription keys for the Computer Vision API | Microsoft Docs
description: Learn how to get subscription keys for calls to the Computer Vision API in Cognitive Services.
services: cognitive-services
author: JuliaNik
manager: ytkuo
ms.service: cognitive-services
ms.technology: computer-vision
ms.topic: article
ms.date: 05/19/2017
ms.author: juliakuz
ms.translationtype: Human Translation
ms.sourcegitcommit: a30a90682948b657fb31dd14101172282988cbf0
ms.openlocfilehash: 64f0a9116947baf52013889f98e0471fd9d9ef76
ms.contentlocale: es-es
ms.lasthandoff: 05/25/2017

---

## <a name="obtaining-subscription-keys"></a>Obteniendo las llaves de suscripción

Los servicios de Computer Vision requieren una llave de suscripción especial. Cada llamada a la API de Computer Vision requiere una llave de suscripción. Esta llave necesita ser pasada a través de un parámetro en el query string o especificado en el header de la petición.

Para suscribirse a las llaves de suscripción, revisa [Suscripciones](https://azure.microsoft.com/try/cognitive-services/). Es gratis registrarse. El precio por los servicios está sujeto a cambios.

>[!NOTE]
Tus llaves de suscripción son válidas únicamente para estas [Regiones de Microsoft Azure](https://azure.microsoft.com/regions/).

| Región | Dirección |
|---|---|
| Oeste de EE. UU. | westus.api.cognitive.microsoft.com |
| Este de EE. UU. 2 | eastus2.api.cognitive.microsoft.com |
| Centro occidental de EE.UU. | westcentralus.api.cognitive.microsoft.com |
| Europa Occidental | westeurope.api.cognitive.microsoft.com |
| Sudeste de Asia | southeastasia.api.cognitive.microsoft.com |

Si se registra usando la versión de prueba de Computer Vision, las llaves de suscripción serán válidas para la región **westcentral** (`https://westcentralus.api.cognitive.microsoft.com/vision/v1.0/`). Ese ese el caso más común. Aún así, si se registra a Computer Vision con su cuenta de Microsoft Azure a través del sitio [https://azure.microsoft.com/](https://azure.microsoft.com/), puede especificar la región para su versión de prueba de la lista de regiones antes mencionada.

Por ejemplo, si se registró a Computer Vision con su cuenta de Microsoft Azure y especificó `westus` para su región, deberá usar la región `westus` para sus llamadas a la API REST (`https://westus.api.cognitive.microsoft.com/vision/v1.0/`).

Si llega a olvidar la región de su suscripción después de obtener la llave de prueba, puede encontrar su región en [https://azure.microsoft.com/try/cognitive-services/my-apis/](https://azure.microsoft.com/try/cognitive-services/my-apis/).

Suscripciones

### <a name="related-links"></a>Links relacionados:
* [Opciones de pago para las APIs de Microsoft Cognitive](https://azure.microsoft.com/pricing/details/cognitive-services/)
