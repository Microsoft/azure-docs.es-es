---
title: Nubes y regiones en las que Azure Media Services v3 está disponible
description: En este artículo se trata sobre las nubes y regiones de Azure donde Azure Media Services v3 está disponible.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 5/28/2020
ms.author: inhenkel
ms.custom: references_regions
ms.openlocfilehash: 6ecc8cad4480528477f6bb470c7bc32ee2a20e11
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87001356"
---
# <a name="clouds-and-regions-in-which-azure-media-services-v3-exists"></a>Nubes y regiones donde existe Azure Media Services v3

Azure Media Services v3 está disponible a través del manifiesto de Azure Resource Manager en Azure global, Azure Government, Azure Alemania, Azure China 21Vianet. Sin embargo, no todas las características de Media Services están disponibles en todas las nubes de Azure. En este documento se describen las disponibilidades de los componentes principales de Media Services v3.

## <a name="feature-availability-in-azure-clouds"></a>Disponibilidad de características en las nubes de Azure

| Característica|con disponibilidad general | Azure Government|Azure Alemania|Azure China 21Vianet|
| --- | --- | --- | --- | --- |
| [Azure EventGrid](reacting-to-media-services-events.md) | Disponible | No disponible | No disponible | No disponible |
| [VideoAnalyzerPreset](analyzing-video-audio-files-concept.md) |  Disponible | No disponible | No disponible | No disponible |
| [AudioAnalyzerPreset](analyzing-video-audio-files-concept.md) |  Disponible | No disponible | No disponible | No disponible |
| [StandardEncoderPreset](encoding-concept.md) | Disponible | Disponible | Disponible | Disponible |
| [LiveEvents](live-streaming-overview.md) | Disponible | Disponible | Disponible | Disponible |
| [StreamingEndpoints](streaming-endpoint-concept.md) | Disponible | Disponible | Disponible | Disponible |

## <a name="feature-availability-in-preview"></a>Disponibilidad de características en versión preliminar

[LiveTranscription](live-transcription.md) está disponible en las siguientes regiones:

- Sudeste de Asia
- Oeste de Europa
- Norte de Europa
- Este de EE. UU.
- Centro de EE. UU.
- Centro-sur de EE. UU.
- Oeste de EE. UU. 2
- Sur de Brasil

## <a name="regionsgeographieslocations"></a>Regiones, zonas geográficas y ubicaciones

[Regiones en las que se implementa el servicio Azure Media Services](https://azure.microsoft.com/global-infrastructure/services/?products=media-services)

### <a name="region-code-name"></a>Nombre de código de región

Cuando tenga que proporcionar el parámetro **location**, deberá proporcionar el nombre de código de región como el valor **location**. Para obtener el nombre de código de la región en la que está su cuenta y a la que se debe enrutar la llamada, puede ejecutar la siguiente línea en la [CLI de Azure](/cli/azure/?view=azure-cli-latest)

```azurecli-interactive
az account list-locations
```

Una vez que ejecuta la línea mostrada anteriormente, obtiene una lista de todas las regiones de Azure. Vaya a la región de Azure que tenga el elemento *displayName* que busca y use su valor *name* para el parámetro **location**.

Por ejemplo, para la región Oeste de EE. UU. 2 de Azure (se muestra a continuación), usará "westus2" para el parámetro **location**.

```json
   {
      "displayName": "West US 2",
      "id": "/subscriptions/00000000-23da-4fce-b59c-f6fb9513eeeb/locations/westus2",
      "latitude": "47.233",
      "longitude": "-119.852",
      "name": "westus2",
      "subscriptionId": null
    }
```

## <a name="endpoints"></a>Puntos de conexión  

Es importante conocer los siguientes puntos de conexión al conectarse a cuentas de Media Services desde diferentes nubes nacionales de Azure.

### <a name="global-azure"></a>Azure global

| Servicio | Punto de conexión |
| ------- | -------- |
| Azure Resource Manager |  `https://management.azure.com/` |
| Authentication | `https://login.microsoftonline.com/` |
| Público del token | `https://management.core.windows.net/` |

### <a name="azure-government"></a>Azure Government

| Servicio | Punto de conexión |
| ------- | -------- |
| Azure Resource Manager |  `https://management.usgovcloudapi.net/` |
| Authentication | `https://login.microsoftonline.us/` |
| Público del token | `https://management.core.usgovcloudapi.net/` |

### <a name="azure-germany"></a>Azure Alemania

| Servicio | Punto de conexión |
| ------- | -------- |
| Azure Resource Manager | `https://management.cloudapi.de/` |
| Authentication | `https://login.microsoftonline.de/` |
| Público del token | `https://management.core.cloudapi.de/`|

### <a name="azure-china-21vianet"></a>Azure China 21Vianet

| Servicio | Punto de conexión |
| ------- | -------- |
| Azure Resource Manager | `https://management.chinacloudapi.cn/` |
| Authentication | `https://login.chinacloudapi.cn/` |
| Público del token |  `https://management.core.chinacloudapi.cn/` |

## <a name="see-also"></a>Consulte también

* [Regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions/)
* [Zonas geográficas de Azure](https://azure.microsoft.com/global-infrastructure/geographies/)
* [Ubicaciones de Azure](https://azure.microsoft.com/global-infrastructure/locations/)

## <a name="next-steps"></a>Pasos siguientes

[Introducción a Media Services v3](media-services-overview.md)
