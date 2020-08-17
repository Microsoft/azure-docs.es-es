---
title: Creación de filtros con la API REST de Azure Media Services v3
description: En este tema se describe cómo crear filtros para que su cliente pueda usarlos para el streaming de secciones específicas de una secuencia. Media Services crea manifiestos dinámicos para lograr este streaming selectivo.
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
ms.date: 06/13/2019
ms.author: juliako
ms.openlocfilehash: 93ef342ecee9bb9e2f32400655db0cacc01835c0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87023116"
---
# <a name="creating-filters-with-media-services-rest-api"></a>Creación de filtros con la API REST de Media Services

Al entregar su contenido a los clientes (streaming de eventos en directo o vídeo bajo demanda), es posible que el cliente necesite más flexibilidad que la descrita en el archivo de manifiesto del recurso predeterminado. Azure Media Services le permite definir filtros de cuenta y filtros de recurso para su contenido. 

Para obtener una descripción detallada de esta característica y los escenarios donde se utiliza, vea [Manifiestos dinámicos](filters-dynamic-manifest-overview.md) y [Filtros](filters-concept.md).

En este tema se muestra cómo definir un filtro para un recurso Vídeo bajo demanda y usar las API REST para crear [Filtros de cuenta](/rest/api/media/accountfilters) y [Filtros de recurso](/rest/api/media/assetfilters). 

> [!NOTE]
> No olvide revisar [presentationTimeRange](filters-concept.md#presentationtimerange).

## <a name="prerequisites"></a>Prerrequisitos 

Para completar los pasos descritos en este tema, ha de:

- Consulte [Filtros y manifiestos dinámicos](filters-dynamic-manifest-overview.md).
- [Configuración de Postman para llamadas API REST de Azure Media Services](media-rest-apis-with-postman.md).

    Asegúrese de seguir el último paso en el tema [Obtención del token de Azure AD](media-rest-apis-with-postman.md#get-azure-ad-token). 

## <a name="define-a-filter"></a>Definición de un filtro  

Este es el ejemplo de **Cuerpo de la solicitud** que define las condiciones de selección de seguimiento que se agregan al manifiesto. Este filtro incluye las pistas de audio con EC-3 y las pistas de vídeo con velocidad de bits en el intervalo 0 a 1000000.

```json
{
    "properties": {
        "tracks": [
          {
                "trackSelections": [
                    {
                        "property": "Type",
                        "value": "Audio",
                        "operation": "Equal"
                    },
                    {
                        "property": "FourCC",
                        "value": "EC-3",
                        "operation": "Equal"
                    }
                ]
            },
            {
                "trackSelections": [
                    {
                        "property": "Type",
                        "value": "Video",
                        "operation": "Equal"
                    },
                    {
                        "property": "Bitrate",
                        "value": "0-1000000",
                        "operation": "Equal"
                    }
                ]
            }
        ]
     }
}
```

## <a name="create-account-filters"></a>Creación de filtros de cuenta

En la colección de Postman que ha descargado, seleccione **Filtros de cuenta**->**Create or update an Account Filter** (Crear o actualizar un filtro de cuenta).

El método **PUT** de solicitud HTTP es similar a:

```
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Media/mediaServices/{accountName}/accountFilters/{filterName}?api-version=2018-07-01
```

Seleccione la pestaña **Cuerpo** y pegue el código json [que definió anteriormente](#define-a-filter).

Seleccione **Enviar**. 

Se ha creado el filtro.

Para más información, vea el tema sobre [creación o actualización](/rest/api/media/accountfilters/createorupdate). Vea también la sección de [ejemplos de JSON para los filtros](/rest/api/media/accountfilters/createorupdate#create-an-account-filter).

## <a name="create-asset-filters"></a>Creación de filtros de recurso  

En la colección de Postman "Media Services v3" que ha descargado, seleccione **Recursos**->**Create or update Asset Filter (Crear o actualizar filtro de recurso)** .

El método **PUT** de solicitud HTTP es similar a:

```
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Media/mediaServices/{accountName}/assets/{assetName}/assetFilters/{filterName}?api-version=2018-07-01
```

Seleccione la pestaña **Cuerpo** y pegue el código json [que definió anteriormente](#define-a-filter).

Seleccione **Enviar**. 

Se ha creado el filtro de recurso.

Para más información sobre cómo crear o actualizar los filtros de recurso, vea el tema sobre [creación o actualización](/rest/api/media/assetfilters/createorupdate). Vea también la sección de [ejemplos de JSON para los filtros](/rest/api/media/assetfilters/createorupdate#create-an-asset-filter). 

## <a name="associate-filters-with-streaming-locator"></a>Asociación de filtros con localizadores de streaming

Ahora puede especificar una lista de filtros de cuentas o recursos, que se aplicarían a su localizador de streaming. El [empaquetado dinámico (punto de conexión de streaming)](dynamic-packaging-overview.md) se aplica a esta lista de filtros junto con los que el cliente especifica en la dirección URL. Esta combinación genera un [manifiesto dinámico](filters-dynamic-manifest-overview.md), que se basa en los filtros de la dirección URL y en los filtros que especifique en el localizador de streaming. Se recomienda usar esta característica si quiere aplicar filtros, pero no quiere exponer los nombres de filtro en la dirección URL.

Para crear y asociar filtros a un localizador de Streaming mediante REST, utilice la API [Streaming Locators - Create](/rest/api/media/streaminglocators/create) y especifique `properties.filters` en el [Cuerpo de la solicitud](/rest/api/media/streaminglocators/create#request-body).
                                
## <a name="stream-using-filters"></a>Streaming con filtros

Una vez que defina los filtros, los clientes podrán utilizarlos en la dirección URL de streaming. Se podrían aplicar filtros a protocolos de streaming con velocidad de bits adaptable: formatos Apple HTTP Live Streaming (HLS), MPEG-DASH y Smooth Streaming

En la tabla siguiente se muestran algunos ejemplos de direcciones URL con filtros:

|Protocolo|Ejemplo|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|Smooth Streaming|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

## <a name="next-steps"></a>Pasos siguientes

[Streaming de vídeos](stream-files-tutorial-with-rest.md) 
