---
title: Introducción a Azure Media Player
description: Más información acerca de Azure Media Player.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 04/20/2020
ms.openlocfilehash: 6f3c173c61e9f6d82a51fcaf7e11cda47eb47c1c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87011721"
---
# <a name="azure-media-player-overview"></a>Información general sobre Azure Media Player #

Azure Media Player es un reproductor de vídeo web integrado que reproduce contenido multimedia de [Microsoft Azure Media Services](https://azure.microsoft.com/services/media-services/) en una gran variedad de exploradores y dispositivos. Explorador multimedia de Azure utiliza los estándares del sector, como HTML5, Media Source Extensions (MSE, extensiones de origen multimedia) y Encrypted Media Extensions (EME, extensiones multimedia cifradas) para proporcionar una experiencia de streaming adaptativa enriquecida.  Cuando estos estándares no están disponibles en un dispositivo o en un explorador, Azure Media Player usa Flash y Silverlight como tecnología de reserva. Independientemente de la tecnología de reproducción que se usa, los desarrolladores tendrán una interfaz unificada de JavaScript para tener acceso a las API.  Esto permite que el contenido proporcionado por Azure Media Services se reproduzca a través de una amplia gama de dispositivos y exploradores sin ningún esfuerzo adicional.

Microsoft Azure Media Services permite ofrecer contenido con formatos de streaming DASH, Smooth Streaming y HLS para reproducir contenido. Azure Media Player tiene en cuenta estos diversos formatos y reproduce automáticamente el vínculo mejor según las capacidades del explorador y plataforma. Microsoft Azure Media Services también permite el cifrado dinámico de recursos con el cifrado común (PlayReady o Widevine) o el cifrado de sobre de AES de 128 bits. Azure Media Player permite el descifrado del contenido cifrado de AES de 128 bits y PlayReady cuando se configura correctamente.  Consulte la sección [Contenido protegido](azure-media-player-protected-content.md) para ver cómo se configura.

Para solicitar características nuevas, proporcionar ideas o comentarios, envíelos a [UserVoice para Azure Media Player](https://aka.ms/ampuservoice). Si tiene problemas concretos, preguntas o encontró algún error, escríbanos a ampinfo@microsoft.com.

[Regístrese](https://aka.ms/ampsignup) para no perder nunca una versión y mantenerse al día con las últimas novedades que Azure Media Player ofrece.

> [!NOTE]
> Tenga en cuenta que Azure Media Player solo admite transmisiones multimedia de Azure Media Services.

## <a name="license"></a>Licencia ##

Azure Media Player se usa bajo licencia y está sujeto a los términos descritos en los términos de licencia del software de Microsoft para Azure Media Player. Consulte el [archivo de licencia](/legal/azure-media-player/azure-media-player-license) para ver todos los términos. Para obtener más información, consulte la [Declaración de privacidad](https://www.microsoft.com/en-us/privacystatement/default.aspx).

Copyright 2015 Microsoft Corporation.

## <a name="next-steps"></a>Pasos siguientes ##

- [Inicio rápido de Azure Media Player](azure-media-player-quickstart.md)
