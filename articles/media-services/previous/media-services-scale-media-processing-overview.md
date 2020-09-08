---
title: Información general del escalado de procesamiento de medios | Microsoft Docs
description: Este tema ofrece una introducción al escalado de procesamiento de medios con on Azure Media Services.
services: media-services
documentationcenter: ''
author: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/26/2019
ms.author: juliako
ms.openlocfilehash: 1d2ef02ea77ad2bca37f1e397b784d06481538fa
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89264018"
---
# <a name="scaling-media-processing-overview"></a>Información general del escalado de procesamiento de medios 

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Esta página proporciona una introducción a cómo y por qué se debe escalar el procesamiento de medios. 

## <a name="overview"></a>Información general
Una cuenta de Media Services está asociada con un tipo de unidad reservada que determina la rapidez con la que se procesan las tareas de procesamiento multimedia. Puede elegir uno de los siguientes tipos de unidad reservada: **S1**, **S2** o **S3**. Por ejemplo, el mismo trabajo de codificación se ejecuta más rápido cuando se usa el tipo de unidad reservada **S2** en comparación con el tipo**S1**. Para obtener más información, consulte el blog [Encoding Reserved Unit Types](https://azure.microsoft.com/blog/high-speed-encoding-with-azure-media-services/)(Codificación de tipos de unidad reservada).

Además de especificar el tipo de unidad reservada, puede especificar el aprovisionamiento de su cuenta con unidades reservadas de codificación. El número de unidades reservadas de codificación aprovisionadas determina el número de tareas de medios que se pueden procesar de forma simultánea en una cuenta determinada. Por ejemplo, si la cuenta tiene cinco unidades reservadas, se ejecutarán simultáneamente cinco tareas multimedia siempre que haya tareas para procesar. Las tareas restantes esperarán en la cola y se elegirán para el procesamiento secuencialmente cuando finalice la tarea en ejecución. Si una cuenta no tiene ninguna unidad reservada aprovisionada, las tareas se elegirán de manera secuencial. En este caso, el tiempo de espera entre la finalización de una tarea y el inicio de la siguiente dependerá de la disponibilidad de los recursos del sistema.

## <a name="choosing-between-different-reserved-unit-types"></a>Selección de los distintos tipos de unidad reservada
Con esta tabla será más fácil tomar la decisión de elegir entre distintas velocidades de codificación. En ella se proporcionan también algunos casos de referencias comparativas en [un vídeo que puede descargar](https://nimbuspmteam.blob.core.windows.net/asset-46f1f723-5d76-477e-a153-3fd0f9f90f73/SeattlePikePlaceMarket_7min.ts?sv=2015-07-08&sr=c&si=013ab6a6-5ebf-431e-8243-9983a6b5b01c&sig=YCgEB8DxYKK%2B8W9LnBykzm1ZRUTwQAAH9QFUGw%2BIWuc%3D&se=2118-09-21T19%3A28%3A57Z) para realizar sus propias pruebas:

|Tipo de RU|Escenario|Resultados de ejemplo para el [vídeo de 7 min y 1080 p](https://nimbuspmteam.blob.core.windows.net/asset-46f1f723-5d76-477e-a153-3fd0f9f90f73/SeattlePikePlaceMarket_7min.ts?sv=2015-07-08&sr=c&si=013ab6a6-5ebf-431e-8243-9983a6b5b01c&sig=YCgEB8DxYKK%2B8W9LnBykzm1ZRUTwQAAH9QFUGw%2BIWuc%3D&se=2118-09-21T19%3A28%3A57Z)|
|---|---|---|
| **S1**|Codificación con velocidad de bits sencilla. <br/>Archivos con resolución SD o menor, no sujetos a limitación temporal y de bajo costo.|La codificación en el archivo MP4 de resolución SD de velocidad de bits única con "H264 Single Bitrate SD 16x9" tarda en torno a 7 minutos.|
| **S2**|Codificación con velocidad de bits sencilla y múltiple.<br/>Uso normal para codificación SD y HD.|La codificación con el valor predeterminado "H264 Single Bitrate 720p" tarda en torno a 6 minutos.<br/><br/>La codificación con el valor predeterminado "H264 Multiple Bitrate 720p" tarda en torno a 12 minutos.|
| **S3**|Codificación con velocidad de bits sencilla y múltiple.<br/>Vídeos con resolución Full HD y 4K. Codificación con respuesta más rápida, sujeta a limitación temporal.|La codificación con el valor predeterminado "H264 Single Bitrate 1080p" tardará aproximadamente 3 minutos.<br/><br/>La codificación con el valor predeterminado "H264 Multiple Bitrate 1080p" tarda aproximadamente 8 minutos.|

## <a name="considerations"></a>Consideraciones
> [!IMPORTANT]
> Revise las consideraciones descritas en esta sección.  
> 
> 

* Para los trabajos de análisis de audio y vídeo desencadenados por Media Services v3 o Video Indexer, se recomienda encarecidamente el tipo de unidad S3.
* Si utiliza el grupo compartido, es decir, sin ninguna unidad reservada, las tareas de codificación tendrán el mismo rendimiento que con las RU S1. Sin embargo, no existe ningún límite superior para el tiempo que las tareas pueden estar en el estado en cola, y en un momento determinado, se estará ejecutando a lo sumo una tarea.

## <a name="billing"></a>Facturación

Se le cobra en función del número de minutos que se aprovisionan las unidades reservadas de multimedia en su cuenta. Esto ocurre independientemente de si se ejecutan trabajos en la cuenta. Para obtener una explicación detallada, vea la sección de preguntas más frecuentes de la página de [precios de Media Services](https://azure.microsoft.com/pricing/details/media-services/).   

## <a name="quotas-and-limitations"></a>Cuotas y limitaciones
Para obtener información sobre las cuotas y limitaciones y sobre cómo abrir una incidencia de soporte técnico, consulte [Cuotas y limitaciones](media-services-quotas-and-limitations.md).

## <a name="next-step"></a>Paso siguiente
Llevar a cabo la tarea de escalado de procesamiento de medios con alguna de estas tecnologías: 

> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-encoding-units.md)
> * [Portal](media-services-portal-scale-media-processing.md)
> * [REST](/rest/api/media/operations/encodingreservedunittype)
> * [Java](https://github.com/southworkscom/azure-sdk-for-media-services-java-samples)
> * [PHP](https://github.com/Azure/azure-sdk-for-php/tree/master/examples/MediaServices)
> 

> [!NOTE]
> Para obtener la versión más reciente del SDK de Java y empezar a realizar desarrollos con Java, consulte [Introducción al SDK de cliente de Java para Azure Media Services](./media-services-java-how-to-use.md). <br/>
> Para descargar el SDK de PHP más reciente para Media Services, busque la versión 0.5.7 del paquete de Microsoft/WindowAzure en el [repositorio Packagist](https://packagist.org/packages/microsoft/windowsazure#v0.5.7).  

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
