---
title: 'Soporte técnico para las licencias de Media Services y FairPlay de Apple: Azure | Microsoft Docs'
description: En este tema se proporciona información general sobre los requisitos de licencia y la configuración de FairPlay de Apple.
author: juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/08/2018
ms.author: juliako
ms.custom: seodec18
ms.openlocfilehash: 433768ddea6e250c66483ec3da4c7737b3538a1d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87092055"
---
# <a name="apple-fairplay-license-requirements-and-configuration"></a>Configuración y requisitos de licencia de FairPlay de Apple 

Azure Media Services le permite cifrar el contenido HLS con **FairPlay de Apple** (AES-128 CBC). Asimismo, Media Services también proporciona un servicio para entregar licencias de FairPlay. Cuando el reproductor intenta reproducir contenido protegido de FairPlay, se envía una solicitud al servicio de entrega de licencias para obtener una licencia. Si el servicio de licencia aprueba la solicitud, emite la licencia, la que se envía al cliente y se usa para descifrar y reproducir el contenido especificado.

Media Services también proporciona las API que puede usar para configurar sus licencias de FairPlay. En este tema se describen los requisitos de licencia de FairPlay y se muestra cómo puede configurar una licencia de **FairPlay** mediante las API de Media Services. 

## <a name="requirements"></a>Requisitos

Cuando se usa Media Services para proporcionar HLS cifrado con **FairPlay de Apple** y entregar licencias de FairPlay, se necesita lo siguiente:

* Suscríbase al [programa de desarrollo de Apple](https://developer.apple.com/).
* En Apple es obligatorio que el propietario del contenido obtenga el [paquete de implementación](https://developer.apple.com/contact/fps/). Indique que ya ha implementado el módulo principal de seguridad (KSM) con Media Services y que está solicitando el paquete FPS final. Hay instrucciones que aparecen en el paquete FPS final para generar certificados y obtener la clave secreta de la aplicación (ASK). Utilice la ASK para configurar FairPlay.
* En la entrega de claves o licencias de Media Services se debe establecer lo siguiente:

    * **Certificado de aplicación (CA)** : Se trata de un archivo .pfx que contiene la clave privada. Puede crear este archivo y cifrarlo con una contraseña. El archivo .pfx debe estar en formato Base64.

        En los pasos siguientes se describe cómo generar un certificado pfx para FairPlay:

        1. Instale OpenSSL desde https://slproweb.com/products/Win32OpenSSL.html.

            Vaya a la carpeta donde están los certificados FairPlay y otros archivos cuyo emisor sea Apple.
        2. Ejecute el siguiente comando desde la línea de comando. Esto convierte el archivo .cer en un archivo .pem.

            "C:\OpenSSL-Win32\bin\openssl.exe" x509 -inform der -in FairPlay.cer -out FairPlay-out.pem
        3. Ejecute el siguiente comando desde la línea de comando. Esto convierte el archivo .pem a un archivo .pfx con la clave privada. A continuación, OpenSSL pide la contraseña para el archivo .pfx.

            "C:\OpenSSL-Win32\bin\openssl.exe" pkcs12 -export -out FairPlay-out.pfx -inkey privatekey.pem -in FairPlay-out.pem -passin file:privatekey-pem-pass.txt
            
    * **Contraseña de certificado de la aplicación**: La contraseña para crear el archivo .pfx.
    * **ASK**: Esta clave se recibe cuando se genera la certificación mediante el portal para desarrolladores de Apple (Apple Developer). Cada equipo de desarrollo recibe una única ASK. Guarde una copia de la ASK y almacénela en un lugar seguro. Necesita configurar la ASK como FairPlayAsk con Media Services.
    
* En el lado cliente FPS se debe establecer lo siguiente:

  * **Certificado de aplicación (CA)** : Se trata de un archivo .cer/.der que contiene la clave pública que el sistema operativo usa para cifrar alguna carga útil. Media Services necesita tener información sobre él porque lo necesita el reproductor. El servicio de entrega de claves lo descifra utilizando la clave privada correspondiente.

* Para reproducir una transmisión cifrada FairPlay, obtenga primero la ASK real y luego genere un certificado real. Este proceso crea todas las tres partes:

  * archivo .der
  * archivo .pfx
  * la contraseña para el archivo .pfx
  
> [!NOTE]
> Azure Media Services no comprueba la fecha de expiración del certificado durante el empaquetado ni la entrega de claves. Seguirá funcionando después de que expire el certificado.

## <a name="fairplay-and-player-apps"></a>Aplicaciones de reproductor y de FairPlay

Cuando el contenido está cifrado con **FairPlay de Apple**, las muestras de audio y vídeo individuales se cifran con el modo **AES-128 CBC**. **FairPlay Streaming** (FPS) se integra en los sistemas operativos de dispositivos, con compatibilidad nativa en iOS y TV de Apple. Safari en OS X habilita FPS utilizando la compatibilidad con la interfaz Encrypted Media Extensions (EME).

Azure Media Player admite la reproducción de FairPlay. Consulte la [documentación de Azure Media Player](https://amp.azure.net/libs/amp/latest/docs/index.html) para obtener más información.

Puede desarrollar sus propias aplicaciones de reproducción gracias al SDK de iOS. Para poder reproducir el contenido de FairPlay, tiene que implementar el protocolo de intercambio de licencias. Apple no especifica este protocolo. Depende de cada aplicación decidir cómo enviar las solicitudes de entrega de claves. El servicio de entrega de claves de FairPlay de Media Services espera que SPC funcione como mensaje de publicación codificado www-form-url con el formato siguiente:

```
spc=<Base64 encoded SPC>
```

## <a name="fairplay-configuration-net-example"></a>Ejemplo .NET de la configuración de FairPlay

Puede usar la API de Media Services para configurar las licencias de FairPlay. Cuando el reproductor intenta reproducir contenido protegido de FairPlay, se envía una solicitud al servicio de entrega de licencias para obtener una licencia. Si el servicio de licencias aprueba la solicitud, el servicio emite la licencia. A continuación, se envía al cliente y se usa para descifrar y reproducir el contenido especificado.

> [!NOTE]
> En general, lo más seguro es que configure las opciones de directiva de FairPlay solo una vez, ya que solo tendrá un conjunto de certificados y una ASK.

En el ejemplo siguiente se usa el [SDK de .NET de Media Services](/dotnet/api/microsoft.azure.management.media.models?view=azure-dotnet) para configurar la licencia.

```csharp
private static ContentKeyPolicyFairPlayConfiguration ConfigureFairPlayPolicyOptions()
{

    string askHex = "";
    string FairPlayPfxPassword = "";

    var appCert = new X509Certificate2("FairPlayPfxPath", FairPlayPfxPassword, X509KeyStorageFlags.Exportable);

    byte[] askBytes = Enumerable
        .Range(0, askHex.Length)
        .Where(x => x % 2 == 0)
        .Select(x => Convert.ToByte(askHex.Substring(x, 2), 16))
        .ToArray();

    ContentKeyPolicyFairPlayConfiguration fairPlayConfiguration =
    new ContentKeyPolicyFairPlayConfiguration
    {
        Ask = askBytes,
        FairPlayPfx =
                Convert.ToBase64String(appCert.Export(X509ContentType.Pfx, FairPlayPfxPassword)),
        FairPlayPfxPassword = FairPlayPfxPassword,
        RentalAndLeaseKeyType =
                ContentKeyPolicyFairPlayRentalAndLeaseKeyType
                .PersistentUnlimited,
        RentalDuration = 2249 // in seconds
    };

    return fairPlayConfiguration;
}
```

## <a name="next-steps"></a>Pasos siguientes

Consulte cómo [proteger con DRM](protect-with-drm.md).
