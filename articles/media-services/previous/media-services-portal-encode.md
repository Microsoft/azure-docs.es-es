---
title: Codificación de un recurso con Media Encoder Standard en Azure Portal | Microsoft Docs
description: Este tutorial describe los pasos de codificación de un recurso con Media Encoder Standard en Azure Portal.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 107d9e9a-71e9-43e5-b17c-6e00983aceab
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 606a4d753675475cf0138b688523e9029e6d3b0a
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89260788"
---
# <a name="encode-an-asset-by-using-media-encoder-standard-in-the-azure-portal"></a>Codificación de un recurso con Media Encoder Standard en Azure Portal

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> Para completar este tutorial, deberá tener una cuenta de Azure. Para más información, consulte [Evaluación gratuita de Azure](https://azure.microsoft.com/pricing/free-trial/). 
> 
> 

Cuando se trabaja con Azure Media Services, uno de los escenarios más comunes es entregar streaming de velocidad de bits adaptable a los clientes. Media Services admite las siguientes tecnologías de streaming con velocidad de bits adaptable: Apple HTTP Live Streaming (HLS), Microsoft Smooth Streaming y Dynamic Adaptive Streaming sobre HTTP (DASH, también llamado MPEG-DASH). Para preparar vídeos para streaming con velocidad de bits adaptable, primero debe codificar el vídeo de origen como archivos de varias velocidades de bits. Puede usar Media Encoder Standard para codificar los vídeos.  

Media Services proporciona empaquetado dinámico. Con empaquetado dinámico, puede entregar los MP4 de varias velocidades de bits en HLS, Smooth Streaming y MPEG-DASH, sin volver a empaquetar en estos formatos de streaming. Gracias al empaquetado dinámico, puede almacenar y pagar por los archivos en un solo formato de almacenamiento. Media Services crea y publica la respuesta adecuada según la solicitud del cliente.

Para aprovechar las ventajas del empaquetado dinámico, debe codificar el archivo de origen en un conjunto de archivos MP4 con velocidad de bits múltiple. Los pasos de codificación se muestran más adelante en este artículo.

Para obtener información sobre cómo escalar el procesamiento multimedia, consulte [Escalado del procesamiento multimedia mediante Azure Portal](media-services-portal-scale-media-processing.md).

## <a name="encode-in-the-azure-portal"></a>Codificación en Azure Portal

Para codificar el contenido mediante Media Encoder Standard:

1. En [Azure Portal](https://portal.azure.com/), seleccione la cuenta de Azure Media Services.
2. Seleccione **Configuración** > **Activos**. Seleccione el recurso que desea codificar.
3. Seleccione el botón **Codificar**.
4. En el panel **Codificar un recurso**, seleccione el procesador **Media Encoder Standard** y un valor predeterminado. Para más información sobre los valores predeterminados, consulte los artículos sobre la [generación automática de una escalera de velocidad de bits](media-services-autogen-bitrate-ladder-with-mes.md) y [Valores preestablecidos de las tareas de MES (Media Encoder Standard)](media-services-mes-presets-overview.md). Es importante elegir el valor predeterminado que funcionará mejor con la entrada de vídeo. Por ejemplo, si sabe que la entrada de vídeo tiene una resolución de 1920x1080 píxeles, se podría usar el valor predeterminado **H264 Multiple Bitrate 1080p**. Si tiene un vídeo de baja resolución (640x360), no debería usar el valor predeterminado **H264 Multiple Bitrate 1080p**.
   
   Para ayudarlo a administrar los recursos, puede editar el nombre del recurso de salida y el nombre del trabajo.
   
   ![Codificación de recursos](./media/media-services-portal-vod-get-started/media-services-encode1.png)
5. Seleccione **Crear**.

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Pasos siguientes
* [Supervisión del progreso del trabajo de codificación](media-services-portal-check-job-progress.md) en Azure Portal.  

