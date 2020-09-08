---
title: Streaming de FairPlay sin conexión para iOS con Azure Media Services v3
description: En este tema se ofrece información general y se muestra cómo usar Azure Media Services para cifrar dinámicamente el contenido HTTP Live Streaming (HLS) con Apple FairPlay en modo sin conexión.
services: media-services
keywords: HLS, DRM, FairPlay Streaming (FPS), Offline, iOS 10
documentationcenter: ''
author: willzhan
manager: steveng
editor: ''
ms.assetid: 7c3b35d9-1269-4c83-8c91-490ae65b0817
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 08/31/2020
ms.author: willzhan
ms.custom: devx-track-csharp
ms.openlocfilehash: c46fe5a4199e73cd22aeef4df62daeed4b58db06
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89291320"
---
# <a name="offline-fairplay-streaming-for-ios-with-media-services-v3"></a>Streaming de FairPlay sin conexión para iOS con Media Services v3

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

 Azure Media Services proporciona un conjunto de [servicios de protección de contenido](https://azure.microsoft.com/services/media-services/content-protection/) de alto diseño que abarca:

- Microsoft PlayReady
- Google Widevine
    
    Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.
- Apple FairPlay
- Cifrado AES-128

El cifrado de contenido del tipo Administración de derechos digitales (DRM) o Estándar de cifrado avanzado (AES) se realiza dinámicamente a petición para los distintos protocolos de streaming. Media Services también proporciona servicios de entrega de claves de descifrado de AES/licencia de DRM.

Además de la protección del contenido de streaming en línea sobre diversos protocolos de streaming, el modo sin conexión para el contenido protegido también es una característica solicitada con frecuencia. Se requiere compatibilidad con el modo sin conexión para los escenarios siguientes:

* Reproducción cuando la conexión a Internet no está disponible, como durante los viajes.
* Algunos proveedores de contenido pueden suspender la entrega de licencias de DRM al cruzar la frontera de un país o región. Si los usuarios quieren ver contenido mientras viajan fuera del país o la región, es necesario descargar sin conexión.
* En algunos países o regiones, la disponibilidad de Internet o de ancho de banda sigue siendo limitada. Los usuarios pueden decidir descargar primero para poder ver el contenido en una resolución lo suficientemente alta que les permita disfrutar de una experiencia de visualización satisfactoria. En este caso, el problema habitualmente no es la disponibilidad de la red, sino que su ancho de banda es limitado. Los proveedores de OVP y OTT solicitan la compatibilidad con el modo sin conexión.

En este artículo se trata la compatibilidad con el modo sin conexión de FairPlay Streaming (FPS) que está destinado a dispositivos que ejecutan iOS 10 o posterior. Esta característica no es compatible con otras plataformas de Apple como watchOS, tvOS o Safari en macOS.

> [!NOTE]
> DRM sin conexión solo se factura para hacer una única solicitud de una licencia al descargar el contenido. Los errores no se facturan.

## <a name="prerequisites"></a>Prerrequisitos

Antes de implementar DRM sin conexión para FairPlay en un dispositivo iOS 10 o superior:

* Revise la protección de contenido en línea para FairPlay: 

    - [Configuración y requisitos de licencia de FairPlay de Apple](fairplay-license-overview.md)
    - [Uso del cifrado dinámico de DRM y el servicio de entrega de licencias](protect-with-drm.md)
    - Un ejemplo de .NET que incluye configuración de streaming de FPS en línea: [ConfigureFairPlayPolicyOptions](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L505)
* Obtener el SDK de FPS de Apple Developer Network. El SDK de FPS contiene dos componentes:

    - El SDK de FPS Server, que contiene el módulo de seguridad de claves (KSM), ejemplos de cliente, una especificación y un conjunto de vectores de prueba.
    - El Paquete de implementación de FPS, que contiene la especificación de la función D junto con instrucciones sobre cómo generar el certificado de FPS, la clave privada específica del cliente y la clave secreta de aplicación. Apple emite el paquete de implementación de FPS únicamente a los proveedores de contenido con licencia.
* Clone https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials.git. 

    Necesitará modificar el código en [Codificación con DRM mediante .NET](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/tree/master/AMSV3Tutorials/EncryptWithDRM) para agregar configuraciones de FairPlay.  

## <a name="configure-content-protection-in-azure-media-services"></a>Configuración de la protección de contenido en Azure Media Services

En el método [GetOrCreateContentKeyPolicyAsync](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L189), haga lo siguiente:

Quite la marca de comentario del código que configura la opción de directiva de FairPlay:

```csharp
ContentKeyPolicyFairPlayConfiguration fairplayConfig = ConfigureFairPlayPolicyOptions();
```

Además, quite la marca de comentario del código que agrega CBCS ContentKeyPolicyOption en la lista de ContentKeyPolicyOptions.

```csharp
options.Add(
    new ContentKeyPolicyOption()
    {
        Configuration = fairplayConfig,
        Restriction = restriction,
        Name = "ContentKeyPolicyOption_CBCS"
    });
```

## <a name="enable-offline-mode"></a>Habilitación del modo sin conexión

Para habilitar el modo sin conexión, cree un objeto StreamingPolicy personalizado y use su nombre al crear un objeto StreamingLocator en [CreateStreamingLocatorAsync](https://github.com/Azure-Samples/media-services-v3-dotnet-tutorials/blob/master/AMSV3Tutorials/EncryptWithDRM/Program.cs#L561).
 
```csharp
CommonEncryptionCbcs objStreamingPolicyInput= new CommonEncryptionCbcs()
{
    Drm = new CbcsDrmConfiguration()
    {
        FairPlay = new StreamingPolicyFairPlayConfiguration()
        {
            AllowPersistentLicense = true // This enables offline mode
        }
    },
    EnabledProtocols = new EnabledProtocols()
    {
        Hls = true,
        Dash = true // Even though DASH under CBCS is not supported for either CSF or CMAF, HLS-CMAF-CBCS uses DASH-CBCS fragments in its HLS playlist
    },

    ContentKeys = new StreamingPolicyContentKeys()
    {
        // Default key must be specified if keyToTrackMappings is present
        DefaultKey = new DefaultKey()
        {
            Label = "CBCS_DefaultKeyLabel"
        }
    }
}

```

Ahora la cuenta de Media Services está configurada para entregar licencias FairPlay sin conexión.

## <a name="sample-ios-player"></a>Reproductor iOS de ejemplo

La compatibilidad con el modo sin conexión de FPS solo está disponible en iOS 10 y versiones posteriores. El SDK de FPS Server (versión 3.0 o posterior) contiene el documento y un ejemplo para el modo sin conexión de FPS. En concreto, el SDK de FPS Server (versión 3.0 o posterior) contiene los dos elementos siguientes relacionados con el modo sin conexión:

* Documento: "Offline Playback with FairPlay Streaming and HTTP Live Streaming" ("Reproducción sin conexión con FairPlay Streaming y HTTP Live Streaming"). Apple, 14 de septiembre de 2016. En la versión 4.0 del SDK de FPS Server, este documento se combina con el documento principal de FPS.
* Ejemplo de código: ejemplo de HLSCatalog (parte del SDK FPS Server de Apple) para el modo sin conexión de FPS en \FairPlay Streaming Server SDK version 3.1\Development\Client\HLSCatalog_With_FPS\HLSCatalog\. En la aplicación de ejemplo HLSCatalog, los archivos de código siguientes se usan para implementar las características del modo sin conexión:

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

![Secuencias de aplicación de iOS de FairPlay sin conexión](media/offline-fairplay-for-ios/offline-fairplay-ios-app-streams.png)

Si ya las configuró, use la dirección URL de su propio vídeo de prueba, la dirección URL de adquisición de la licencia de FairPlay y la dirección URL del certificado de aplicación. O bien puede continuar en la sección siguiente, que incluye ejemplos de pruebas.

## <a name="integrated-test"></a>Prueba integrada

Existen tres ejemplos de pruebas en Media Services que abarcan los tres escenarios siguientes:

* FPS protegido, con pista de vídeo, audio y audio alternativo.
* FPS protegido, con pista de vídeo y audio pero sin audio alternativo.
* FPS protegido, solo con vídeo, sin audio.

Puede encontrar estos ejemplos en [este sitio de demostración](https://aka.ms/poc#22), con el certificado de aplicación correspondiente hospedado en una aplicación web de Azure.
Ya sea con un ejemplo de la versión 3 o de la versión 4 del SDK de FPS Server, si una lista de reproducción maestra contiene audio alternativo, durante el modo sin conexión solo reproduce audio. Por lo tanto, es necesario quitar el audio alternativo. Es decir, tanto el segundo como el tercer ejemplo mencionados anteriormente funcionan en el modo en línea y en el modo sin conexión. En el primer ejemplo, el audio se reproduce solo en el modo sin conexión, mientras que el streaming en línea funciona correctamente.

## <a name="faq"></a>Preguntas más frecuentes

Consulte las [preguntas más frecuentes para obtener ayuda para solucionar los problemas](frequently-asked-questions.md#why-does-only-audio-play-but-not-video-during-offline-mode).

## <a name="next-steps"></a>Pasos siguientes

Consulte cómo [proteger con AES-128](protect-with-aes128.md).
