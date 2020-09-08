---
title: Configuración de los codificadores locales al usar Azure Media Services para crear transmisiones con velocidad de bits múltiple | Microsoft Docs
description: En este tema se enumeran los codificadores en directo locales que puede utilizar para capturar los eventos en directo y enviar una secuencia en directo con velocidad de bits única a canales AMS (que permiten la codificación en directo) para su posterior procesamiento. El tema establece vínculos a tutoriales que muestran cómo configurar los codificadores que se presentan.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.assetid: 0ec6f046-0841-4673-9057-883bdbc30d5c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 77f2d433b32d180c7b3a819af96a50b721c2087f
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89263443"
---
# <a name="how-to-configure-on-premises-encoders-when-using-azure-media-services-to-create-multi-bitrate-streams"></a>Configuración de los codificadores locales al usar Azure Media Services para crear transmisiones con velocidad de bits múltiple

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

En este tema se enumeran los codificadores en directo locales que puede utilizar para capturar los eventos en directo y enviar una secuencia en directo con velocidad de bits única a canales AMS (que permiten la codificación en directo) para su posterior procesamiento. El tema también incluye vínculos a tutoriales que muestran cómo configurar los codificadores presentados.

> [!NOTE]
> Al hacer el streaming mediante RTMP, compruebe la configuración del firewall o del proxy para confirmar que los puertos TCP salientes 1935 y 1936 están abiertos.

## <a name="haivision-kb-encoder"></a>Codificador Haivision KB
Para información sobre cómo configurar el [codificador Haivision KB](https://www.haivision.com/products/kb-series/) para enviar una secuencia en directo con velocidad de bits única a un canal AMS, consulte [Configuring Haivision KB Encoder](media-services-configure-kb-live-encoder.md) (Configuración del codificador Haivision KB).

## <a name="telestream-wirecast"></a>Telestream Wirecast
Para obtener información sobre cómo configurar el codificador [Telestream Wirecast](https://www.telestream.net/wirecast/overview.htm) para enviar una secuencia en directo con velocidad de bits única a un canal AMS, consulte [Configuración de Wirecast](media-services-configure-wirecast-live-encoder.md).

## <a name="elemental-live"></a>Elemental Live
Para obtener más información, consulte [Elemental Live](https://www.elemental.com/products/aws-elemental-appliances-software/#elemental-live).

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Pasos siguientes

[Streaming en vivo con Azure Media Services para crear transmisiones con velocidad de bits múltiple](media-services-manage-live-encoder-enabled-channels.md)

