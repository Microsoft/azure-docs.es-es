---
title: Contenido protegido de Azure Media Player
description: En la actualidad, Azure Media Player admite contenido cifrado de sobre de AES de 128 bits y contenido cifrado común.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 04/20/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: da79556b466e3511845724e969c76477ad2ba0a8
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87423021"
---
# <a name="protected-content"></a>Contenido protegido #

En la actualidad, Azure Media Player admite contenido cifrado de sobre de AES de 128 bits y contenido cifrado común (a través de PlayReady y Widevine), o contenido cifrado mediante FairPlay. Para reproducir el contenido protegido correctamente, debe indicar a Azure Media Player la `protectionInfo`. Esta información existe por origen y se puede agregar directamente en la etiqueta `<source>` mediante `data-setup`.  También puede agregar `protectionInfo` directamente como parámetro si establece el origen dinámicamente.

`protectionInfo` acepta un objeto JSON e incluye:

- `type`: `AES` o `PlayReady` o `Widevine` o `FairPlay`
- `certificateUrl`: debe ser un vínculo directo a su certificado de FairPlay hospedado.

- `authenticationToken`: se trata de un campo de opción para agregar un token de autenticación sin codificar

> [!IMPORTANT]
> El objeto **certificateUrl** solo es necesario para la administración de derechos digitales de FairPlay.***
>[!NOTE]
> El valor predeterminado de techOrder se ha cambiado para dar cabida a la nueva tecnología `html5FairPlayHLS` para reproducir contenido de FairPlay de forma nativa en los exploradores compatibles (Safari en OSX 8+). Si tiene contenido de FairPlay para la reproducción **Y** ha cambiado el valor predeterminado de techOrder a uno personalizado en la aplicación, deberá agregar esta nueva tecnología en el objeto techOrder. Se recomienda incluirlo antes de silverlightSS para que el contenido no se reproduzca mediante PlayReady.

## <a name="code-sample"></a>Código de ejemplo ##

```html
Ex:

    <video id="vid1" class="azuremediaplayer amp-default-skin">
        <source
            src="//example/path/to/myVideo.ism/manifest"
            type="application/vnd.ms-sstr+xml"
            data-setup='{"protectionInfo": [{"type": "AES", "authenticationToken": "Bearer=urn%3amicrosoft%3aazure%3amediaservices%3acontentkeyidentifier=8130520b-c116-45a9-824e-4a0082f3cb3c&Audience=urn%3atest&ExpiresOn=1450207516&Issuer=http%3a%2f%2ftestacs.com%2f&HMACSHA256=eV7HDgZ9msp9H9bnEPGN91sBdU7XsZ9OyB6VgFhKBAU%3d"}]}'
        />
    </video>
or

```javascript
    var myPlayer = amp("vid1", /* Options */);
    myPlayer.src([{
        src: "//example/path/to/myVideo.ism/manifest",
        type: "application/vnd.ms-sstr+xml",
        protectionInfo: [{
            type: "PlayReady",
            authenticationToken: "Bearer=urn%3amicrosoft%3aazure%3amediaservices%3acontentkeyidentifier=d5646e95-63ee-4fbe-ba4e-295c8d9502e0&Audience=urn%3atest&ExpiresOn=1450222961&Issuer=http%3a%2f%2ftestacs.com%2f&HMACSHA256=4Jop3kNJdzVI8L5IZLgFtPdImyE%2fHTRil0x%2bEikSdPs%3d"
        }] }, ]
    );
```

o, con varias DRM

```javascript
    var myPlayer = amp("vid1", /* Options */);
    myPlayer.src([{
        src: "//example/path/to/myVideo.ism/manifest",
        type: "application/vnd.ms-sstr+xml",
        protectionInfo: [{
                type: "PlayReady",
                authenticationToken: "Bearer=urn%3amicrosoft%3aazure%3amediaservices%3acontentkeyidentifier=d5646e95-63ee-4fbe-ba4e-295c8d9502e0&Audience=urn%3atest&ExpiresOn=1450222961&Issuer=http%3a%2f%2ftestacs.com%2f&HMACSHA256=4Jop3kNJdzVI8L5IZLgFtPdImyE%2fHTRil0x%2bEikSdPs%3d"
            },
            {
                type: "Widevine",
                authenticationToken: "Bearer=urn%3amicrosoft%3aazure%3amediaservices%3acontentkeyidentifier=d5646e95-63ee-4fbe-ba4e-295c8d9502e0&Audience=urn%3atest&ExpiresOn=1450222961&Issuer=http%3a%2f%2ftestacs.com%2f&HMACSHA256=4Jop3kNJdzVI8L5IZLgFtPdImyE%2fHTRil0x%2bEikSdPs%3d"
            },
            {
                   type: "FairPlay",
                  certificateUrl: "//example/path/to/myFairplay.der",
                   authenticationToken: "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1cm46bWljcm9zb2Z0OmF6dXJlOm1lZGlhc2VydmljZXM6Y29udGVudGtleWlkZW50aWZpZXIiOiIyMTI0M2Q2OC00Yjc4LTRlNzUtYTU5MS1jZWMzMDI0NDNhYWMiLCJpc3MiOiJodHRwOi8vY29udG9zbyIsImF1ZCI6InVybjp0ZXN0IiwiZXhwIjoxNDc0NTkyNDYzLCJuYmYiOjE0NzQ1ODg1NjN9.mE7UxgNhkieMMqtM_IiYQj-FK1KKIzB6lAptw4Mi67A"
        }] } ]
    );
```

> [!NOTE]
> No todos los exploradores y plataformas son capaces de reproducir contenido protegido. Consulte la sección [Tecnología de reproducción](azure-media-player-playback-technology.md) para más información sobre las tecnologías compatibles.
> [!IMPORTANT]
> El token que se pasa al reproductor está pensado para el contenido protegido y solo se usa para los usuarios autenticados. Se supone que la aplicación usa SSL o algún otro tipo de medida de seguridad. Además, se da por supuesta la confianza en que el usuario final no hará un mal uso del token. Si ese no es el caso, pida ayuda a los expertos en seguridad.

## <a name="next-steps"></a>Pasos siguientes ##

- [Inicio rápido de Azure Media Player](azure-media-player-quickstart.md)