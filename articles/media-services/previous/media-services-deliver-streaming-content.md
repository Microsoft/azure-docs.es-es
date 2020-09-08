---
title: Publicación de contenido de Azure Media Services mediante .NET | Microsoft Docs
description: Aprenda a crear un localizador que se usa para generar una dirección URL de streaming. Los ejemplos de código están escritos en C# y utilizan el SDK de Media Services para .NET.
author: juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: c53b1f83-4cb1-4b09-840f-9c145b7d6f8d
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: a0343c607a342c3438d68627b97c71cdf05199b2
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267792"
---
# <a name="publish-media-services-content-using-net"></a>Publicación de contenido de Media Services con.NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [REST](media-services-rest-deliver-streaming-content.md)
> * [.NET](media-services-deliver-streaming-content.md)
> * [Portal](media-services-portal-publish.md)
> 
> 

## <a name="overview"></a>Información general
Puede transmitir un conjunto de archivos MP4 de velocidad de bits adaptable creando un localizador de streaming a petición y compilando una dirección URL de streaming. El tema [Codificación de un recurso](media-services-encode-asset.md) muestra cómo codificar en un conjunto de MP4 de velocidad de bits adaptable. 

> [!NOTE]
> Si el contenido está cifrado, configure la directiva de entrega de recursos (como se describe en [este](media-services-dotnet-configure-asset-delivery-policy.md) tema) antes de crear un localizador. 
> 
> 

También puede utilizar un localizador de streaming a petición para generar direcciones URL que señalen a archivos MP4 que se pueden descargar progresivamente.  

En este tema se muestra cómo crear un localizador de streaming a petición para publicar el recurso y crear direcciones URL de streaming Smooth, MPEG DASH y HLS. También se muestra cómo generar direcciones URL de descarga progresiva. 

## <a name="create-an-ondemand-streaming-locator"></a>Creación de un localizador de streaming a petición
Para crear el localizador de streaming a petición y obtener las direcciones URL, deberá hacer lo siguiente:

1. Si se cifra el contenido, defina una directiva de acceso.
2. Cree un localizador de streaming a petición.
3. Si planea transmitir, obtenga el archivo de manifiesto de streaming (.ism) del recurso. 
   
   Si planea la descarga progresiva, obtenga los nombres de los archivos MP4 del recurso.  
4. Genere direcciones URL para el archivo de manifiesto o archivos MP4. 


>[!NOTE]
>Hay un límite de 1 000 000 directivas para diferentes directivas de AMS (por ejemplo, para la directiva de localizador o ContentKeyAuthorizationPolicy). Utilice el mismo identificador de directiva si siempre usa los mismos días o permisos de acceso. Por ejemplo, directivas para localizadores que van a permanecer durante mucho tiempo (directivas no de carga). Para obtener más información, consulte [este tema](media-services-dotnet-manage-entities.md#limit-access-policies) .

### <a name="use-media-services-net-sdk"></a>Uso del SDK de .NET de Media Services
Generación de direcciones URL de streaming 

```csharp
    private static void BuildStreamingURLs(IAsset asset)
    {

        // Create a 30-day readonly access policy. 
          // You cannot create a streaming locator using an AccessPolicy that includes write or delete permissions.
        IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
            TimeSpan.FromDays(30),
            AccessPermissions.Read);

        // Create a locator to the streaming content on an origin. 
        ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
            policy,
            DateTime.UtcNow.AddMinutes(-5));

        // Display some useful values based on the locator.
        Console.WriteLine("Streaming asset base path on origin: ");
        Console.WriteLine(originLocator.Path);
        Console.WriteLine();

        // Get a reference to the streaming manifest file from the  
        // collection of files in the asset. 
        var manifestFile = asset.AssetFiles.ToList().Where(f => f.Name.ToLower().
                                    EndsWith(".ism")).
                                    FirstOrDefault();

        // Create a full URL to the manifest file. Use this for playback
        // in streaming media clients. 
        string urlForClientStreaming = originLocator.Path + manifestFile.Name + "/manifest";
        Console.WriteLine("URL to manifest for client streaming using Smooth Streaming protocol: ");
        Console.WriteLine(urlForClientStreaming);
        Console.WriteLine("URL to manifest for client streaming using HLS protocol: ");
        Console.WriteLine(urlForClientStreaming + "(format=m3u8-aapl)");
        Console.WriteLine("URL to manifest for client streaming using MPEG DASH protocol: ");
        Console.WriteLine(urlForClientStreaming + "(format=mpd-time-csf)"); 
        Console.WriteLine();
    }
```

Las salidas:

- Dirección URL del manifiesto para streaming de cliente mediante el protocolo de Smooth Streaming:\
  `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest`
- Dirección URL del manifiesto para streaming de cliente mediante el protocolo HLS:\
  `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=m3u8-aapl)`
- Dirección URL del manifiesto para streaming de cliente mediante el protocolo MPEG DASH:\
  `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny.ism/manifest(format=mpd-time-csf)`

> [!NOTE]
> También puede transmitir el contenido por una conexión TLS. Para llevar a cabo este enfoque, asegúrese de que las direcciones URL de streaming comienzan por HTTPS. Actualmente, AMS no admite TLS con dominios personalizados.
> 
> 

Creación de direcciones URL de descarga progresiva 

```csharp
    private static void BuildProgressiveDownloadURLs(IAsset asset)
    {
        // Create a 30-day readonly access policy. 
        IAccessPolicy policy = _context.AccessPolicies.Create("Streaming policy",
            TimeSpan.FromDays(30),
            AccessPermissions.Read);

        // Create an OnDemandOrigin locator to the asset. 
        ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
            policy,
            DateTime.UtcNow.AddMinutes(-5));

        // Display some useful values based on the locator.
        Console.WriteLine("Streaming asset base path on origin: ");
        Console.WriteLine(originLocator.Path);
        Console.WriteLine();

        // Get MP4 files.
        IEnumerable<IAssetFile> mp4AssetFiles = asset
            .AssetFiles
            .ToList()
            .Where(af => af.Name.EndsWith(".mp4", StringComparison.OrdinalIgnoreCase));

        // Create a full URL to the MP4 files. Use this to progressively download files.
        foreach (var pd in mp4AssetFiles)
            Console.WriteLine(originLocator.Path + pd.Name);
    }
```
Las salidas:

- `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4`
- `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_400kbps_AAC_und_ch2_96kbps.mp4`
- `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_3400kbps_AAC_und_ch2_96kbps.mp4`
- `http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_2250kbps_AAC_und_ch2_96kbps.mp4`

    . . . 

### <a name="use-media-services-net-sdk-extensions"></a>Uso de Extensiones del SDK .NET de Media Services
El código siguiente llama a los métodos de extensiones del SDK de .NET que crean un localizador y generan las direcciones URL de Smooth Streaming, HLS y MPEG-DASH para el streaming adaptable.
```csharp
    // Create a loctor.
    _context.Locators.Create(
        LocatorType.OnDemandOrigin,
        inputAsset,
        AccessPermissions.Read,
        TimeSpan.FromDays(30));

    // Get the streaming URLs.
    Uri smoothStreamingUri = inputAsset.GetSmoothStreamingUri();
    Uri hlsUri = inputAsset.GetHlsUri();
    Uri mpegDashUri = inputAsset.GetMpegDashUri();

    Console.WriteLine(smoothStreamingUri);
    Console.WriteLine(hlsUri);
    Console.WriteLine(mpegDashUri);
```

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="next-steps"></a>Pasos siguientes
* [Descarga de activos](media-services-deliver-asset-download.md)
* [Configuración de directivas de entrega de activos](media-services-dotnet-configure-asset-delivery-policy.md)

