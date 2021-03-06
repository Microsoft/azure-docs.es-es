---
title: Procedimiento para unir dos o más archivos de vídeo con .NET | Microsoft Docs
description: En este artículo se muestra cómo unir dos o más archivos de vídeo.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
ms.service: media-services
ms.topic: how-to
ms.date: 03/24/2021
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: 562398b17ec5adaaa590b36c1e30e9818592a485
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109684818"
---
# <a name="how-to-stitch-two-or-more-video-files-with-net"></a>Procedimiento para unir dos o más archivos de vídeo con .NET

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

## <a name="stitch-two-or-more-video-files"></a>Unión de dos o más archivos de vídeo

En el ejemplo siguiente se ilustra cómo generar un valor preestablecido para unir dos o más archivos de vídeo. El escenario más frecuente es cuando desea agregar un encabezado o un finalizador al vídeo principal.

> [!NOTE]
> Los archivos de vídeo que se editan juntos deben compartir propiedades (resolución de vídeo, velocidad de fotogramas, número de pistas de audio, etc.). Procure no para combinar vídeos de distintas velocidades de fotogramas o con un número diferente de pistas de audio.

## <a name="prerequisites"></a>Requisitos previos

Clone o descargue los [ejemplos de .NET de Media Services](https://github.com/Azure-Samples/media-services-v3-dotnet/).  El código al que se hace referencia a continuación se encuentra en la [carpeta EncodingWithMESCustomStitchTwoAssets](https://github.com/Azure-Samples/media-services-v3-dotnet/blob/main/VideoEncoding/Encoding_StitchTwoAssets/Program.cs).
