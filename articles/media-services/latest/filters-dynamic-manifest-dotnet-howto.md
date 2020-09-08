---
title: Creación de filtros con el SDK de .NET de Azure Media Services v3
description: En este tema se describe cómo crear filtros para que su cliente pueda usarlos para el streaming de secciones específicas de una secuencia. Media Services crea manifiestos dinámicos para lograr este streaming selectivo.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 08/31/2020
ms.author: inhenkel
ms.custom: devx-track-csharp
ms.openlocfilehash: b2a392ab5301a51edff1df88596f2fe68d85ea63
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266721"
---
# <a name="create-filters-with-media-services-net-sdk"></a>Creación de filtros con el SDK de .NET de Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Al entregar su contenido a los clientes (streaming de eventos en directo o vídeo bajo demanda), es posible que el cliente necesite más flexibilidad que la descrita en el archivo de manifiesto del recurso predeterminado. Azure Media Services le permite definir filtros de cuenta y filtros de recurso para su contenido. 

Para obtener una descripción detallada de esta característica y los escenarios donde se utiliza, vea [Manifiestos dinámicos](filters-dynamic-manifest-overview.md) y [Filtros](filters-concept.md).

En este tema se muestra cómo usar el SDK de .NET de Media Services para definir un filtro para un recurso Vídeo bajo demanda y crear [Filtros de cuenta](/dotnet/api/microsoft.azure.management.media.models.accountfilter?view=azure-dotnet) y [Filtros de recurso](/dotnet/api/microsoft.azure.management.media.models.assetfilter?view=azure-dotnet). 

> [!NOTE]
> No olvide revisar [presentationTimeRange](filters-concept.md#presentationtimerange).

## <a name="prerequisites"></a>Prerrequisitos 

- Consulte [Filtros y manifiestos dinámicos](filters-dynamic-manifest-overview.md).
- [Cree una cuenta de Media Services](./create-account-howto.md). Asegúrese de recordar el nombre del grupo de recursos y el nombre de la cuenta de Media Services. 
- Obtenga la información que necesita para [acceder a las API](./access-api-howto.md)
- Revise [Carga, codificación y transmisión con Azure Media Services](stream-files-tutorial-with-api.md) para ver cómo [empezar a usar el SDK de .NET](stream-files-tutorial-with-api.md#start-using-media-services-apis-with-net-sdk)

## <a name="define-a-filter"></a>Definición de un filtro  

En .NET, configura selecciones de pista con las clases [FilterTrackSelection](/dotnet/api/microsoft.azure.management.media.models.filtertrackselection?view=azure-dotnet) y [FilterTrackPropertyCondition](/dotnet/api/microsoft.azure.management.media.models.filtertrackpropertycondition?view=azure-dotnet). 

El siguiente código define un filtro que incluye las pistas de audio con EC-3 y las pistas de vídeo con velocidad de bits en el intervalo 0 a 1000000.

```csharp
var audioConditions = new List<FilterTrackPropertyCondition>()
{
    new FilterTrackPropertyCondition(FilterTrackPropertyType.Type, "Audio", FilterTrackPropertyCompareOperation.Equal),
    new FilterTrackPropertyCondition(FilterTrackPropertyType.FourCC, "EC-3", FilterTrackPropertyCompareOperation.Equal)
};

var videoConditions = new List<FilterTrackPropertyCondition>()
{
    new FilterTrackPropertyCondition(FilterTrackPropertyType.Type, "Video", FilterTrackPropertyCompareOperation.Equal),
    new FilterTrackPropertyCondition(FilterTrackPropertyType.Bitrate, "0-1000000", FilterTrackPropertyCompareOperation.Equal)
};

List<FilterTrackSelection> includedTracks = new List<FilterTrackSelection>()
{
    new FilterTrackSelection(audioConditions),
    new FilterTrackSelection(videoConditions)
};
```

## <a name="create-account-filters"></a>Creación de filtros de cuenta

El siguiente código muestra cómo usar .NET para crear un filtro de cuenta que incluya todas las selecciones de pista [definidas anteriormente](#define-a-filter). 

```csharp
AccountFilter accountFilterParams = new AccountFilter(tracks: includedTracks);
client.AccountFilters.CreateOrUpdate(config.ResourceGroup, config.AccountName, "accountFilterName1", accountFilter);
```

## <a name="create-asset-filters"></a>Creación de filtros de recurso

El siguiente código muestra cómo usar .NET para crear un filtro de recurso que incluya todas las selecciones de pista [definidas anteriormente](#define-a-filter). 

```csharp
AssetFilter assetFilterParams = new AssetFilter(tracks: includedTracks);
client.AssetFilters.CreateOrUpdate(config.ResourceGroup, config.AccountName, encodedOutputAsset.Name, "assetFilterName1", assetFilterParams);
```

## <a name="associate-filters-with-streaming-locator"></a>Asociación de filtros con localizadores de streaming

Ahora puede especificar una lista de filtros de cuentas o recursos, que se aplicarían a su localizador de streaming. El [empaquetado dinámico (punto de conexión de streaming)](dynamic-packaging-overview.md) se aplica a esta lista de filtros junto con los que el cliente especifica en la dirección URL. Esta combinación genera un [manifiesto dinámico](filters-dynamic-manifest-overview.md), que se basa en los filtros de la dirección URL y en los filtros que especifique en el localizador de streaming. Se recomienda usar esta característica si quiere aplicar filtros, pero no quiere exponer los nombres de filtro en la dirección URL.

El siguiente código C# muestra cómo crear un localizador de streaming y especificar `StreamingLocator.Filters`. Esta es una propiedad opcional que toma una `IList<string>` de nombres de filtro.

```csharp
IList<string> filters = new List<string>();
filters.Add("filterName");

StreamingLocator locator = await client.StreamingLocators.CreateAsync(
    resourceGroup,
    accountName,
    locatorName,
    new StreamingLocator
    {
        AssetName = assetName,
        StreamingPolicyName = PredefinedStreamingPolicy.ClearStreamingOnly,
        Filters = filters
    });
```
      
## <a name="stream-using-filters"></a>Streaming con filtros

Una vez que defina los filtros, los clientes podrán utilizarlos en la dirección URL de streaming. Se podrían aplicar filtros a protocolos de streaming con velocidad de bits adaptable: formatos Apple HTTP Live Streaming (HLS), MPEG-DASH y Smooth Streaming

En la tabla siguiente se muestran algunos ejemplos de direcciones URL con filtros:

|Protocolo|Ejemplo|
|---|---|
|HLS|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=m3u8-aapl,filter=myAccountFilter)`|
|MPEG DASH|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(format=mpd-time-csf,filter=myAssetFilter)`|
|Smooth Streaming|`https://amsv3account-usw22.streaming.media.azure.net/fecebb23-46f6-490d-8b70-203e86b0df58/bigbuckbunny.ism/manifest(filter=myAssetFilter)`|

## <a name="next-steps"></a>Pasos siguientes

[Streaming de vídeos](stream-files-tutorial-with-api.md) 
