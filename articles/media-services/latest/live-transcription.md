---
title: Transcripción en directo
titleSuffix: Azure Media Services
description: Más información sobre la transcripción en directo de Azure Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: how-to
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: c3465e294af104c4d9c3b34960f5e95cf41e7cb8
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291422"
---
# <a name="live-transcription-preview"></a>Transcripción en directo (versión preliminar)

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Azure Media Services ofrece vídeo, audio y texto en diferentes protocolos. Cuando publica el streaming en vivo mediante MPEG-DASH o HLS/CMAF, el servicio proporcionará junto con el vídeo y el audio, el texto transcrito en formato IMSC1.1 compatible con TTML. La entrega se empaqueta en fragmentos MPEG-4 parte 30 (ISO/IEC 14496-30). Si la entrega se realiza mediante HLS/TS, el texto se proporcionará como VTT fragmentado.

Cuando se activa Transcripción en directo, se aplican cargos adicionales. Revise la información de precios en la sección Vídeo en directo de la [página de precios de Media Services](https://azure.microsoft.com/pricing/details/media-services/).

En este artículo se describe cómo habilitar Transcripción en directo al hacer streaming de un evento en directo con Azure Media Services. Antes de continuar, asegúrese de que está familiarizado con el uso de las API REST de Media Services v3 (consulte [este tutorial](stream-files-tutorial-with-rest.md) para más información). También debe estar familiarizado con el concepto de [streaming en vivo](live-streaming-overview.md). Se recomienda completar el tutorial [Streaming en vivo con Media Services](stream-live-tutorial-with-api.md).

## <a name="live-transcription-preview-regions-and-languages"></a>Regiones y lenguajes de la versión preliminar de Transcripción en directo

Transcripción en directo está disponible en las siguientes regiones:

- Sudeste de Asia
- Oeste de Europa
- Norte de Europa
- Este de EE. UU.
- Centro de EE. UU.
- Centro-sur de EE. UU.
- Oeste de EE. UU. 2
- Sur de Brasil

Esta es la lista de idiomas disponibles que se pueden asignar, use el código de idioma de la API.

| Idioma | Código de lenguaje |
| -------- | ------------- |
| Catalán  | ca-ES |
| Danés (Dinamarca) | da-DK |
| Alemán (Alemania) | de-DE |
| Inglés (Australia) | en-AU |
| Inglés (Canadá) | en-CA |
| Inglés (Reino Unido) | en-GB |
| Inglés (India) | en-IN |
| Inglés (Nueva Zelanda) | en-NZ |
| Spanish (Traditional Sort) - Spain | es-ES |
| Español (España) | es-ES |
| Español (México) | es-MX |
| Finés (Finlandia) | fi-FI |
| Francés (Canadá) | fr-CA |
| Francés (Francia) | fr-FR |
| Italiano (Italia) | it-IT |
| Neerlandés (Países Bajos) | nl-NL |
| Portugués (Brasil) | pt-BR |
| Portugués (Portugal) | pt-PT |
| Sueco (Suecia) | sv-SE |

## <a name="create-the-live-event-with-live-transcription"></a>Creación del evento en directo con Transcripción en directo

Para crear un evento en directo con la transcripción activada, envíe la operación PUT con la versión de API 2019-05-01-preview; por ejemplo:

```
PUT https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/liveEvents/:liveEventName?api-version=2019-05-01-preview&autoStart=true 
```

La operación tiene el cuerpo siguiente (en el que se crea un evento en directo con paso a través mediante RTMP como protocolo de ingesta). Observe que se ha agregado una propiedad de transcripciones.

```
{
  "properties": {
    "description": "Demonstrate how to enable live transcriptions",
    "input": {
      "streamingProtocol": "RTMP",
      "accessControl": {
        "ip": {
          "allow": [
            {
              "name": "Allow All",
              "address": "0.0.0.0",
              "subnetPrefixLength": 0
            }
          ]
        }
      }
    },
    "preview": {
      "accessControl": {
        "ip": {
          "allow": [
            {
              "name": "Allow All",
              "address": "0.0.0.0",
              "subnetPrefixLength": 0
            }
          ]
        }
      }
    },
    "encoding": {
      "encodingType": "None"
    },
    "transcriptions": [
      {
        "language": "en-US"
      }
    ],
    "vanityUrl": false,
    "streamOptions": [
      "Default"
    ]
  },
  "location": "West US 2"
}
```

## <a name="start-or-stop-transcription-after-the-live-event-has-started"></a>Inicio o detención de la transcripción después de que se haya iniciado el evento en directo

Puede iniciar y detener la transcripción en directo mientras el evento en directo está en ejecución. Para obtener más información sobre cómo iniciar y detener eventos en directo, lea la sección de operaciones de ejecución prolongada en el artículo de [desarrollo con las API de Media Services v3](media-services-apis-overview.md#long-running-operations).

Para activar las transcripciones en directo o actualizar el idioma de la transcripción, revise el evento en directo para incluir una propiedad "transcriptions". Para desactivar las transcripciones en directo, quite la propiedad "transcriptions" del objeto de evento activo.  

> [!NOTE]
> No se admite la activación o desactivación de la transcripción **más de una vez** durante el evento en directo.

Esta es la llamada de ejemplo para activar las transcripciones en directo.

PATCH: ```https://management.azure.com/subscriptions/:subscriptionId/resourceGroups/:resourceGroupName/providers/Microsoft.Media/mediaServices/:accountName/liveEvents/:liveEventName?api-version=2019-05-01-preview```

```
{
  "properties": {
    "description": "Demonstrate how to enable live transcriptions", 
    "input": {
      "streamingProtocol": "RTMP",
      "accessControl": {
        "ip": {
          "allow": [
            {
              "name": "Allow All",
              "address": "0.0.0.0",
              "subnetPrefixLength": 0
            }
          ]
        }
      }
    },
    "preview": {
      "accessControl": {
        "ip": {
          "allow": [
            {
              "name": "Allow All",
              "address": "0.0.0.0",
              "subnetPrefixLength": 0
            }
          ]
        }
      }
    },
    "encoding": {
      "encodingType": "None"
    },
    "transcriptions": [
      {
        "language": "en-US"
      }
    ],
    "vanityUrl": false,
    "streamOptions": [
      "Default"
    ]
  },
  "location": "West US 2"
}
```

## <a name="transcription-delivery-and-playback"></a>Entrega y reproducción de la transcripción

Revise el artículo [Introducción al empaquetado dinámico](dynamic-packaging-overview.md#to-prepare-your-source-files-for-delivery) que describe cómo usa el servicio el empaquetado dinámico para ofrecer vídeo, audio y texto en diferentes protocolos. Cuando publica el streaming en vivo mediante MPEG-DASH o HLS/CMAF, el servicio proporcionará junto con el vídeo y el audio, el texto transcrito en formato IMSC1.1 compatible con TTML. La entrega se empaqueta en fragmentos MPEG-4 parte 30 (ISO/IEC 14496-30). Si la entrega se realiza mediante HLS/TS, el texto se proporcionará como VTT fragmentado. Puede usar un reproductor web como [Azure Media Player](use-azure-media-player.md) para reproducir la secuencia.  

> [!NOTE]
> Si usa Azure Media Player, use la versión 2.3.3 o posterior.

## <a name="known-issues"></a>Problemas conocidos

En la versión preliminar, estos son los problemas conocidos de la transcripción en directo:

- Las aplicaciones deben usar las API de la versión preliminar que se describen en la [especificación de Media Services v3 OpenAPI](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/mediaservices/resource-manager/Microsoft.Media/preview/2019-05-01-preview/streamingservice.json).
- La protección de administración de derechos digitales (DRM) no se aplica a la pista de texto, solo es posible el cifrado de sobre AES.

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Media Services](media-services-overview.md)
