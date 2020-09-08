---
title: Uso de reproductores existentes para reproducir el contenido - Azure | Microsoft Docs
description: En este artículo se enumeran los reproductores existentes que puede usar para reproducir el contenido.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 7e9fcf89-0fb6-4fa4-96cb-666320684d69
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: bd09b734f1ac5ac3c98b6c0c717a48de19b0106f
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89261085"
---
# <a name="playing-your-content-with-existing-players"></a>Reproducción de contenido con existentes

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Azure Media Services admite muchos formatos de streaming populares como Smooth Streaming, HTTP Live Streaming y MPEG-Dash. Este tema remite a reproductores existentes que puede usar para probar sus transmisiones.

### <a name="the-azure-portal-media-services-content-player"></a>Reproductor del contenido de Media Services de Azure Portal
**Azure Portal** proporciona un reproductor de contenido que puede usar para probar el vídeo.

Haga clic en el vídeo deseado (asegúrese de que se ha [publicado](media-services-portal-publish.md)) y haga clic en el botón **Reproducir** situado en la parte inferior del portal.

Se aplican algunas consideraciones:

* El **REPRODUCTOR DE CONTENIDO DE SERVICIOS MULTIMEDIA** reproduce desde el extremo de streaming predeterminado. Si desea reproducir desde un extremo de streaming que no esté predeterminado, use otro reproductor. Por ejemplo, [Azure Media Player](https://aka.ms/azuremediaplayer).

![AMSPlayer][AMSPlayer]

### <a name="azure-media-player"></a>Azure Media Player

Use [Azure Media Player](https://aka.ms/azuremediaplayer) para reproducir el contenido (libre o protegido) en cualquiera de los siguientes formatos:

* Smooth Streaming
* MPEG DASH
* HLS
* MP4 progresivo

### <a name="flash-player"></a>Flash Player

#### <a name="playready-with-token"></a>PlayReady con token

[https://sltoken.azurewebsites.net](https://sltoken.azurewebsites.net)

### <a name="dash-players"></a>Reproductores DASH

[https://dashplayer.azurewebsites.net](https://dashplayer.azurewebsites.net)

[https://dashif.org](https://dashif.org)

### <a name="other"></a>Otros
Para probar las direcciones URL de HLS también puede utilizar:

* **Safari** en un dispositivo iOS o
* **3ivx HLS Player** en Windows.

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

[AMSPlayer]: ./media/media-services-playback-content-with-existing-players/media-services-portal-player.png
