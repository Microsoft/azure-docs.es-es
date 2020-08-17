---
title: Protección del contenido HLS con Apple FairPlay - Azure sin conexión | Microsoft Docs
description: En este tema se ofrece información general y se muestra cómo usar Azure Media Services para cifrar dinámicamente el contenido HTTP Live Streaming (HLS) con Apple FairPlay en modo sin conexión.
services: media-services
keywords: HLS, DRM, FairPlay Streaming (FPS), Offline, iOS 10
documentationcenter: ''
author: willzhan
manager: steveng
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/07/2020
ms.author: willzhan
ms.reviewer: dwgeo
ms.openlocfilehash: eee718627f35154c878c717e02febe0ea7958fe6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87000557"
---
# <a name="offline-fairplay-streaming-for-ios"></a>FairPlay Streaming sin conexión para iOS 

> [!div class="op_single_selector" title1="Seleccione la versión de Media Services que se está usando:"]
> * [Versión 3](../latest/offline-fairplay-for-ios.md)
> * [Versión 2](media-services-protect-hls-with-offline-fairplay.md)

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Azure Media Services proporciona un conjunto de [servicios de protección de contenido](https://azure.microsoft.com/services/media-services/content-protection/) de alto diseño que abarca:

- Microsoft PlayReady
- Google Widevine
- Apple FairPlay
- Cifrado AES-128

El cifrado de contenido del tipo Administración de derechos digitales (DRM) o Estándar de cifrado avanzado (AES) se realiza dinámicamente a petición para los distintos protocolos de streaming. Media Services también proporciona servicios de entrega de claves de descifrado de AES/licencia de DRM.

Además de la protección del contenido de streaming en línea sobre diversos protocolos de streaming, el modo sin conexión para el contenido protegido también es una característica solicitada con frecuencia. Se requiere compatibilidad con el modo sin conexión para los escenarios siguientes:

* Reproducción cuando la conexión a Internet no está disponible, como durante los viajes.
* Algunos proveedores de contenido pueden suspender la entrega de licencias de DRM al cruzar la frontera de un país o región. Si los usuarios quieren ver contenido mientras viajan fuera del país o la región, es necesario descargar sin conexión.
* En algunos países o regiones, la disponibilidad de Internet o de ancho de banda sigue siendo limitada. Los usuarios pueden decidir descargar primero para poder ver el contenido en una resolución lo suficientemente alta que les permita disfrutar de una experiencia de visualización satisfactoria. En este caso, el problema habitualmente no es la disponibilidad de la red, sino que su ancho de banda es limitado. Los proveedores de OVP y OTT solicitan la compatibilidad con el modo sin conexión.

En este artículo se trata la compatibilidad con el modo sin conexión de FairPlay Streaming (FPS) que está destinado a dispositivos que ejecutan iOS 10 o posterior. Esta característica no es compatible con otras plataformas de Apple como watchOS, tvOS o Safari en macOS.

## <a name="preliminary-steps"></a>Pasos previos
Antes de implementar DRM sin conexión para FairPlay en un dispositivo iOS 10 o superior:

* Familiarizarse con la protección de contenido en línea para FairPlay. Para más información, consulte los artículos y ejemplos siguientes:

    - [Apple FairPlay Streaming for Azure Media Services is generally available](https://azure.microsoft.com/blog/apple-FairPlay-streaming-for-azure-media-services-generally-available/) (Disponibilidad general de Apple FairPlay Streaming para Azure Media Services)
    - [Protección del contenido HLS con Apple FairPlay o Microsoft PlayReady](./media-services-protect-hls-with-fairplay.md)
    - [Un ejemplo de streaming FPS en línea](https://azure.microsoft.com/resources/samples/media-services-dotnet-dynamic-encryption-with-FairPlay/)

* Obtener el SDK de FPS de Apple Developer Network. El SDK de FPS contiene dos componentes:

    - El SDK de FPS Server, que contiene el módulo de seguridad de claves (KSM), ejemplos de cliente, una especificación y un conjunto de vectores de prueba.
    - El Paquete de implementación de FPS, que contiene la especificación de la función D junto con instrucciones sobre cómo generar el certificado de FPS, la clave privada específica del cliente y la clave secreta de aplicación. Apple emite el paquete de implementación de FPS únicamente a los proveedores de contenido con licencia.

## <a name="configuration-in-media-services"></a>Configuración en Media Services
Para la configuración del modo sin conexión de FPS a través del [SDK .NET de Media Services](https://www.nuget.org/packages/windowsazure.mediaservices), use la versión 4.0.0.4 del SDK .NET de Media Services o una versión posterior, que proporciona la API que se necesita para configurar el modo sin conexión de FPS.
También se necesita el código de trabajo para configurar la protección de contenido FPS en el modo en línea. Una vez que obtiene el código para configurar la protección del contenido FPS en el modo en línea, solo necesita los dos cambios siguientes.

## <a name="code-change-in-the-fairplay-configuration"></a>Cambio de código en la configuración de FairPlay
El primer cambio es definir un valor booleano para "habilitar el modo sin conexión", llamado objDRMSettings.EnableOfflineMode, que es "true" cuando habilita el escenario de DRM sin conexión. En función de este indicador, realice el siguiente cambio en la configuración de FairPlay:

```csharp
if (objDRMSettings.EnableOfflineMode)
    {
        FairPlayConfiguration = Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(
            objX509Certificate2,
            pfxPassword,
            pfxPasswordId,
            askId,
            iv, 
            RentalAndLeaseKeyType.PersistentUnlimited,
            0x9999);
    }
    else
    {
        FairPlayConfiguration = Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(
            objX509Certificate2,
            pfxPassword,
            pfxPasswordId,
            askId,
            iv);
    }
```

## <a name="code-change-in-the-asset-delivery-policy-configuration"></a>Cambio de código en la configuración de la directiva de entrega de activos
El segundo cambio consiste en agregar la tercera clave al diccionario Dictionary<AssetDeliveryPolicyConfigurationKey, string>.
Agregue AssetDeliveryPolicyConfigurationKey tal como se muestra a continuación:
 
```csharp
// FPS offline mode
    if (drmSettings.EnableOfflineMode)
    {
        objDictionary_AssetDeliveryPolicyConfigurationKey.Add(AssetDeliveryPolicyConfigurationKey.AllowPersistentLicense, "true");
        Console.WriteLine("FPS OFFLINE MODE: AssetDeliveryPolicyConfigurationKey.AllowPersistentLicense added into asset delivery policy configuration.");
    }

    // for IAssetDelivery for FPS
    IAssetDeliveryPolicy objIAssetDeliveryPolicy = objCloudMediaContext.AssetDeliveryPolicies.Create(
            drmSettings.AssetDeliveryPolicyName,
            AssetDeliveryPolicyType.DynamicCommonEncryptionCbcs,
            AssetDeliveryProtocol.HLS,
            objDictionary_AssetDeliveryPolicyConfigurationKey);
```

Después de este paso, la cadena <Dictionary_AssetDeliveryPolicyConfigurationKey> en la directiva de entrega de activos de FPS contiene las tres entradas siguientes:

* AssetDeliveryPolicyConfigurationKey.FairPlayBaseLicenseAcquisitionUrl o AssetDeliveryPolicyConfigurationKey.FairPlayLicenseAcquisitionUrl, en función de factores como el servidor de claves/KSM de FPS que se usa y si desea volver a usar la misma directiva de entrega de activos en los distintos activos
* AssetDeliveryPolicyConfigurationKey.CommonEncryptionIVForCbcs
* AssetDeliveryPolicyConfigurationKey.AllowPersistentLicense

Ahora la cuenta de Media Services está configurada para entregar licencias FairPlay sin conexión.

## <a name="sample-ios-player"></a>Reproductor iOS de ejemplo
La compatibilidad con el modo sin conexión de FPS solo está disponible en iOS 10 y versiones posteriores. El SDK de FPS Server (versión 3.0 o posterior) contiene el documento y un ejemplo para el modo sin conexión de FPS. En concreto, el SDK de FPS Server (versión 3.0 o posterior) contiene los dos elementos siguientes relacionados con el modo sin conexión:

* Documento: "Offline Playback with FairPlay Streaming and HTTP Live Streaming" ("Reproducción sin conexión con FairPlay Streaming y HTTP Live Streaming"). Apple, 14 de septiembre de 2016. En la versión 4.0 del SDK de FPS Server, este documento se combina con el documento principal de FPS.
* Ejemplo de código: ejemplo de HLSCatalog para el modo sin conexión de FPS en \FairPlay Streaming Server SDK version 3.1\Development\Client\HLSCatalog_With_FPS\HLSCatalog\. En la aplicación de ejemplo HLSCatalog, los archivos de código siguientes se usan para implementar las características del modo sin conexión:

    - Archivo de código AssetPersistenceManager.swift: AssetPersistenceManager es la clase principal de este ejemplo que muestra cómo:

        - Administrar la descarga de transmisiones HLS, como las API que se usan para empezar y cancelar las descargas y para eliminar los activos existentes de los dispositivos.
        - Supervisar el progreso de la descarga.
    - Archivos de código AssetListTableViewController.swift y AssetListTableViewCell.swift: AssetListTableViewController es la interfaz principal de este ejemplo. Proporciona una lista de activos que el ejemplo puede usar para reproducir, descargar, eliminar o cancelar una descarga. 

Estos pasos muestran cómo configurar un reproductor de iOS en ejecución. Si suponemos que comenzó en el ejemplo de HLSCatalog en la versión 4.0.1 del SDK de FPS Server, realice los cambios de código siguientes:

En HLSCatalog\Shared\Managers\ContentKeyDelegate.swift, implemente el método `requestContentKeyFromKeySecurityModule(spcData: Data, assetID: String)` mediante el código siguiente. Permita que "drmUr" sea una variable asignada a esta dirección URL de HLS.

```swift
    var ckcData: Data? = nil
    
    let semaphore = DispatchSemaphore(value: 0)
    let postString = "spc=\(spcData.base64EncodedString())&assetId=\(assetIDString)"
    
    if let postData = postString.data(using: .ascii, allowLossyConversion: true), let drmServerUrl = URL(string: self.drmUrl) {
        var request = URLRequest(url: drmServerUrl)
        request.httpMethod = "POST"
        request.setValue(String(postData.count), forHTTPHeaderField: "Content-Length")
        request.setValue("application/x-www-form-urlencoded", forHTTPHeaderField: "Content-Type")
        request.httpBody = postData
        
        URLSession.shared.dataTask(with: request) { (data, _, error) in
            if let data = data, var responseString = String(data: data, encoding: .utf8) {
                responseString = responseString.replacingOccurrences(of: "<ckc>", with: "").replacingOccurrences(of: "</ckc>", with: "")
                ckcData = Data(base64Encoded: responseString)
            } else {
                print("Error encountered while fetching FairPlay license for URL: \(self.drmUrl), \(error?.localizedDescription ?? "Unknown error")")
            }
            
            semaphore.signal()
            }.resume()
    } else {
        fatalError("Invalid post data")
    }
    
    semaphore.wait()
    return ckcData
```

En HLSCatalog\Shared\Managers\ContentKeyDelegate.swift, implemente el método `requestApplicationCertificate()`. Esta implementación depende de si inserta el certificado (solo clave pública) en el dispositivo u hospeda el certificado en la web. En la implementación siguiente se usa el certificado de aplicación hospedada usado en los ejemplos de prueba. Permita que "certUrl" sea una variable que contiene la dirección URL del certificado de la aplicación.

```swift
func requestApplicationCertificate() throws -> Data {

        var applicationCertificate: Data? = nil
        do {
            applicationCertificate = try Data(contentsOf: URL(string: certUrl)!)
        } catch {
            print("Error loading FairPlay application certificate: \(error)")
        }
        
        return applicationCertificate
    }
```

Para la prueba final integrada, se proporcionará la dirección URL de vídeo y la dirección URL de certificado de la aplicación en la sección "Prueba integrada".

En HLSCatalog\Shared\Resources\Streams.plist, agregue la dirección URL del vídeo de prueba. En el caso del identificador de clave de contenido, use la dirección URL de adquisición de la licencia de FairPlay con el protocolo skd como el valor único.

![Secuencias de aplicación de iOS de FairPlay sin conexión](media/media-services-protect-hls-with-offline-FairPlay/media-services-offline-FairPlay-ios-app-streams.png)

Si ya las configuró, use la dirección URL de su propio vídeo de prueba, la dirección URL de adquisición de la licencia de FairPlay y la dirección URL del certificado de aplicación. O bien puede continuar en la sección siguiente, que incluye ejemplos de pruebas.

## <a name="integrated-test"></a>Prueba integrada
Existen tres ejemplos de pruebas en Media Services que abarcan los tres escenarios siguientes:

* FPS protegido, con pista de vídeo, audio y audio alternativo.
* FPS protegido, con pista de vídeo y audio pero sin audio alternativo.
* FPS protegido, solo con vídeo, sin audio.

Puede encontrar estos ejemplos en [este sitio de demostración](https://aka.ms/poc#22), con el certificado de aplicación correspondiente hospedado en una aplicación web de Azure.
Ya sea con un ejemplo de la versión 3 o de la versión 4 del SDK de FPS Server, si una lista de reproducción maestra contiene audio alternativo, durante el modo sin conexión solo reproduce audio. Por lo tanto, es necesario quitar el audio alternativo. Es decir, tanto el segundo como el tercer ejemplo mencionados anteriormente funcionan en el modo en línea y en el modo sin conexión. En el primer ejemplo, el audio se reproduce solo en el modo sin conexión, mientras que el streaming en línea funciona correctamente.

## <a name="faq"></a>Preguntas más frecuentes
Las preguntas más frecuentes que aparecen a continuación proporcionan ayuda para solucionar los problemas:

- **¿Por qué solo se reproduce el audio pero no el vídeo en el modo sin conexión?** Este comportamiento parece deberse al diseño de la aplicación de ejemplo. Cuando hay una pista de audio alternativo presente (como es el caso de HLS), durante el modo sin conexión, tanto iOS 10 como iOS 11 usan de manera predeterminada la pista de audio alternativo. Para compensar este comportamiento para el modo sin conexión de FPS, quite la pista de audio alternativo de la secuencia. Para hacer esto en Media Services, agregue el filtro de manifiesto dinámico "audio-only=false". En otras palabras, una dirección URL de HLS finaliza con .ism/manifest(format=m3u8-aapl,audio-only=false). 
- **¿Por qué se sigue reproduciendo solo audio sin vídeo durante el modo sin conexión después de agregar audio-only=false?** Según cómo esté diseñada la clave de caché de la red de entrega de contenido (CND), es posible que el contenido se almacene en la memoria caché. Debe purgar la caché.
- **¿El modo sin conexión de FPS es compatible con iOS 11 además de iOS 10?** Sí. El modo sin conexión de FPS es compatible tanto con iOS 10 como con iOS 11.
- **¿Por qué no encuentro el documento "Offline Playback with FairPlay Streaming and HTTP Live Streaming" ("Reproducción sin conexión con FairPlay Streaming y HTTP Live Streaming") en el SDK de FPS Server?** A partir de la versión 4 del SDK de FPS Server, este documento se combinó con el documento relativo a la guía de programación de streaming de FairPlay.
- **¿Qué significa el último parámetro en la siguiente API para el modo sin conexión de FPS?** 
`Microsoft.WindowsAzure.MediaServices.Client.FairPlay.FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration(objX509Certificate2, pfxPassword, pfxPasswordId, askId, iv, RentalAndLeaseKeyType.PersistentUnlimited, 0x9999);`

    Si necesita la configuración correspondiente a esta API, consulte el [método FairPlayConfiguration.CreateSerializedFairPlayOptionConfiguration](/dotnet/api/microsoft.windowsazure.mediaservices.client.fairplay.fairplayconfiguration.createserializedfairplayoptionconfiguration?view=azure-dotnet). El parámetro representa la duración del alquiler sin conexión con el segundo como unidad.
- **¿Cuál es la estructura del archivo descargado o sin conexión en dispositivos iOS?** La estructura del archivo descargado en un dispositivo iOS es similar a la captura de pantalla siguiente. La carpeta `_keys` almacena licencias FPS descargadas, con un archivo de almacén para cada host de servicio de licencia. La carpeta `.movpkg` almacena el contenido de audio y vídeo. La primera carpeta, cuyo nombre finaliza con un guion seguido de un valor numérico, contiene datos de vídeo. El valor numérico es el valor PeakBandwidth de la reproducción de vídeo. La segunda carpeta, cuyo nombre finaliza con un guion seguido de un 0, contiene datos de audio. La tercera carpeta, denominada "Data", contiene la lista de reproducción maestra del contenido FPS. Por último, boot.xml proporciona una descripción completa del contenido de la carpeta `.movpkg`. 

![Estructura del archivo de aplicación de ejemplo de iOS de FairPlay sin conexión](media/media-services-protect-hls-with-offline-FairPlay/media-services-offline-FairPlay-file-structure.png)

Un archivo boot.xml de ejemplo:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<HLSMoviePackage xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://apple.com/IMG/Schemas/HLSMoviePackage" xsi:schemaLocation="http://apple.com/IMG/Schemas/HLSMoviePackage /System/Library/Schemas/HLSMoviePackage.xsd">
  <Version>1.0</Version>
  <HLSMoviePackageType>PersistedStore</HLSMoviePackageType>
  <Streams>
    <Stream ID="1-4DTFY3A3VDRCNZ53YZ3RJ2NPG2AJHNBD-0" Path="1-4DTFY3A3VDRCNZ53YZ3RJ2NPG2AJHNBD-0" NetworkURL="https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/QualityLevels(127000)/Manifest(aac_eng_2_127,format=m3u8-aapl)">
      <Complete>YES</Complete>
    </Stream>
    <Stream ID="0-HC6H5GWC5IU62P4VHE7NWNGO2SZGPKUJ-310656" Path="0-HC6H5GWC5IU62P4VHE7NWNGO2SZGPKUJ-310656" NetworkURL="https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/QualityLevels(161000)/Manifest(video,format=m3u8-aapl)">
      <Complete>YES</Complete>
    </Stream>
  </Streams>
  <MasterPlaylist>
    <NetworkURL>https://willzhanmswest.streaming.mediaservices.windows.net/e7c76dbb-8e38-44b3-be8c-5c78890c4bb4/MicrosoftElite01.ism/manifest(format=m3u8-aapl,audio-only=false)</NetworkURL>
  </MasterPlaylist>
  <DataItems Directory="Data">
    <DataItem>
      <ID>CB50F631-8227-477A-BCEC-365BBF12BCC0</ID>
      <Category>Playlist</Category>
      <Name>master.m3u8</Name>
      <DataPath>Playlist-master.m3u8-CB50F631-8227-477A-BCEC-365BBF12BCC0.data</DataPath>
      <Role>Master</Role>
    </DataItem>
  </DataItems>
</HLSMoviePackage>
```

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="summary"></a>Resumen
En este documento se incluyen la información y los pasos siguientes que puede usar para implementar el modo sin conexión de FPS:

* La configuración de protección de contenido de Media Services a través de la API de .NET de Media Services configura la entrega de licencias de FairPlay y el cifrado dinámico de FairPlay en Media Services.
* Un reproductor de iOS basado en el ejemplo del SDK de FPS Server configura un reproductor iOS que puede reproducir el contenido de FPS en modo de streaming en línea o en modo sin conexión.
* Se usan vídeos de FPS de ejemplo para probar el modo sin conexión y el streaming en línea.
* Una sección de Preguntas más frecuentes en las que puede encontrar respuestas sobre el modo sin conexión de FPS.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]
