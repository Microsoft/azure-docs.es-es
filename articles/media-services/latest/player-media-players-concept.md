---
title: Información general sobre reproductores multimedia para Media Services
description: ¿Qué reproductores multimedia puedo usar con Azure Media Services? Por el momento, Azure Media Player, Shaka y Video.js.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 3/08/2021
ms.openlocfilehash: 43d0a8ee82a84a57be7c8ded9e7f5afc172bd9d6
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106281632"
---
# <a name="media-players-for-media-services"></a>Reproductores multimedia para Media Services

Puede usar varios reproductores multimedia con Media Services.

## <a name="azure-media-player"></a>Azure Media Player

Azure Media Player es un reproductor de vídeo para una amplia variedad de exploradores y dispositivos. Explorador multimedia de Azure utiliza los estándares del sector, como HTML5, Media Source Extensions (MSE, extensiones de origen multimedia) y Encrypted Media Extensions (EME, extensiones multimedia cifradas) para proporcionar una experiencia de streaming adaptativa enriquecida. Cuando estos estándares no están disponibles en un dispositivo o en un explorador, Azure Media Player usa Flash y Silverlight como tecnología de reserva. Independientemente de la tecnología de reproducción que se use, los desarrolladores tendrán una interfaz de JavaScript unificada para acceder a las API. El contenido proporcionado por Azure Media Services se puede reproducir en una amplia gama de dispositivos y exploradores sin ningún esfuerzo adicional.

Consulte [Información general sobre Azure Media Player](../azure-media-player/azure-media-player-overview.md).

## <a name="shaka"></a>Shaka

Shaka Player es una biblioteca JavaScript de código abierto para medios adaptables. Reproduce formatos de elementos multimedia adaptables (como DASH y HLS) en un explorador, sin usar complementos ni Flash. En su lugar, Shaka Player usa los estándares de web abierta MediaSource Extensions y Encrypted Media Extensions.

Consulte [Uso del reproductor Shaka con Azure Media Services](player-shaka-player-how-to.md).

## <a name="videojs"></a>Video.js

Video.js es un reproductor de vídeo que reproduce formatos multimedia adaptables (como DASH y HLS) en un explorador. Video.js usa los estándares de web abierta MediaSource Extensions y Encrypted Media Extensions. Admite la reproducción de vídeo en equipos de escritorio y dispositivos móviles. La documentación oficial se puede encontrar en https://docs.videojs.com/.

Consulte [Uso del reproductor Video.js con Azure Media Services](player-how-to-video-js-player.md).


## <a name="next-steps"></a>Pasos siguientes ##

- [Inicio rápido de Azure Media Player](../azure-media-player/azure-media-player-quickstart.md)