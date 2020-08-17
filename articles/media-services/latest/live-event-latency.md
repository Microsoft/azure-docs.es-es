---
title: Configuración de latencia baja de LiveEvent en Azure Media Services | Microsoft Docs
description: En este tema se proporciona información general sobre la configuración de latencia baja de LiveEvent y se muestra cómo establecer una latencia baja.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 04/22/2019
ms.author: juliako
ms.openlocfilehash: 24eb5f8691c94c1f319820f3133b66065504ace3
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87000898"
---
# <a name="live-event-low-latency-settings"></a>Configuración de latencia baja en eventos en directo

En este artículo se muestra cómo establecer una latencia baja en un objeto [LiveEvent](/rest/api/media/liveevents). También se describen los resultados típicos que se ven cuando se usa la configuración de latencia baja en varios reproductores. Los resultados varían en función de la latencia de red y CDN.

Para usar la nueva característica **LowLatency**, puede establecer **StreamOptionsFlag** en **LowLatency** en **LiveEvent**. Al crear [LiveOutput](/rest/api/media/liveoutputs) para la reproducción de HLS, establezca [LiveOutput.Hls.fragmentsPerTsSegment](/rest/api/media/liveoutputs/create#hls) en 1. Una vez que la transmisión esté en funcionamiento, puede usar la página de demostración de [Azure Media Player](https://ampdemo.azureedge.net/) (AMP) y establecer las opciones de reproducción para usar "Low Latency Heuristics Profile" (Perfil de heurística de baja latencia).

> [!NOTE]
> Actualmente, LowLatency HeuristicProfile en Azure Media Player está diseñado para reproducir secuencias en protocolo MPEG-DASH con formato CSF o CMAF (por ejemplo, `format=mdp-time-csf` o `format=mdp-time-cmaf`). 

El siguiente ejemplo de .NET muestra cómo establecer **LowLatency** en **LiveEvent**:

```csharp
LiveEvent liveEvent = new LiveEvent(
            location: mediaService.Location, 
            description: "Sample LiveEvent for testing",
            vanityUrl: false,
            encoding: new LiveEventEncoding(
                        // Set this to Standard to enable a transcoding LiveEvent, and None to enable a pass-through LiveEvent
                        encodingType:LiveEventEncodingType.None, 
                        presetName:null
                    ),
            input: new LiveEventInput(LiveEventInputProtocol.RTMP,liveEventInputAccess), 
            preview: liveEventPreview,
            streamOptions: new List<StreamOptionsFlag?>()
            {
                // Set this to Default or Low Latency
                // To use low latency optimally, you should tune your encoder settings down to 1 second "Group Of Pictures" (GOP) length instead of 2 seconds.
                StreamOptionsFlag.LowLatency
            }
        );
```                

Observe el ejemplo completo: [MediaV3LiveApp](https://github.com/Azure-Samples/media-services-v3-dotnet-core-tutorials/blob/master/NETCore/Live/MediaV3LiveApp/Program.cs#L126).

## <a name="live-events-latency"></a>Latencia de objetos LiveEvent

En la tabla siguiente se muestran los resultados típicos de latencia (cuando está habilitada la marca LowLatency) en Media Services, medida desde el momento en que la fuente de contribución alcanza el servicio hasta cuando un usuario visualiza la reproducción en el reproductor. Para usar una latencia baja de forma óptima, debe ajustar la configuración del codificador a la longitud de grupo de imágenes (GOP) de 1 segundo. Cuando se usa una longitud GOP mayor, se minimiza el consumo de ancho de banda y se reduce la velocidad de bits en la misma velocidad de fotogramas. Esto resulta especialmente útil en vídeos con menos movimiento.

### <a name="pass-through"></a>Paso a través 

||Baja latencia de GOP de 2 s habilitada|Baja latencia de GOP de 1 s habilitada|
|---|---|---|
|**DASH en AMP**|10 s|8 s|
|**HLS en el reproductor de iOS nativo**|14 s|10 s|

### <a name="live-encoding"></a>Live Encoding

||Baja latencia de GOP de 2 s habilitada|Baja latencia de GOP de 1 s habilitada|
|---|---|---|
|**DASH en AMP**|14 s|10 s|
|**HLS en el reproductor de iOS nativo**|18 s|13 s|

> [!NOTE]
> Latencia de extremo a extremo puede variar según las condiciones de la red local o al introducir una capa de almacenamiento en caché de CDN. Debe probar las configuraciones exactas.

## <a name="next-steps"></a>Pasos siguientes

- [Introducción al streaming en vivo](live-streaming-overview.md)
- [Tutorial de Live Streaming](stream-live-tutorial-with-api.md)
