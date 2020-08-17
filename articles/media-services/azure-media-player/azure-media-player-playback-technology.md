---
title: Tecnología de reproducción de Azure Media Player
description: Más información sobre la tecnología de reproducción que se usa para reproducir contenidos de vídeo o audio.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 04/20/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: 56fd644e43b704eced4f5a97b82e4b07ab1b4db9
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87424092"
---
# <a name="playback-technology-tech"></a>Tecnología de reproducción #

La tecnología de reproducción hace referencia a la tecnología específica de explorador o de complementos que se usa para reproducir el contenido de vídeo o audio.

- **azureHtml5JS**: emplea estándares de MSE y EME junto con el elemento de vídeo para la reproducción sin complementos de contenido de DASH con compatibilidad con contenido de cifrado de sellado de AES de 128 bits o contenido cifrado normal de DRM (mediante PlayReady y Widevine si lo admite el explorador) procedente de Azure Media Services.
- **flashSS**: emplea tecnología de Flash Player para reproducir contenido de Smooth con compatibilidad con el descifrado de sellado de AES de 128 bits de Azure Media Services. Esto requiere la versión de Flash de 11.4 o posterior.
- **html5FairPlayHLS**: emplea tecnología de reproducción específica basada en exploradores mediante HLS con el elemento de vídeo. Esta tecnología es necesaria para reproducir el contenido protegido de FairPlay de Azure Media Services y se ha agregado a techOrder desde el 19 de octubre de 2016
- **silverlightSS**: emplea la tecnología de Silverlight para reproducir contenido de Smooth con compatibilidad con el contenido protegido de PlayReady de Azure Media Services.
- **html5**: utiliza tecnología de reproducción basada en exploradores con el elemento de vídeo.  Si está en un dispositivo Apple iOS o Android, esta tecnología permite la reproducción de transmisiones HLS con cierta compatibilidad básica con el cifrado de sellado de AES de 128 bits o el contenido DRM (mediante FairPlay si el explorador lo admite).

## <a name="tech-order"></a>Orden tecnológico ##

Para asegurarse de que el recurso se puede reproducir en una amplia variedad de dispositivos, se recomienda el siguiente orden tecnológico y es el valor predeterminado si: `techOrder: ["azureHtml5JS", "flashSS", "html5FairPlayHLS","silverlightSS", "html5"]` y se puede establecer directamente en el `<video>` o mediante programación en las opciones:

`<video data-setup='{"techOrder": ["azureHtml5JS", "flashSS", "html5FairPlayHLS, "silverlightSS", "html5"]}`

or

```javascript
    amp("vid1", {
          techOrder: ["azureHtml5JS", "flashSS", "html5FairPlayHLS, "silverlightSS", "html5"]
    });
```

## <a name="compatibility-matrix"></a>Matriz de compatibilidad ##

Dado el orden tecnológico recomendado con el contenido de streaming de Azure Media Services, se espera la siguiente matriz de reproducción de compatibilidad.

| Browser        | SO                                                       | Tecnología esperada (Clear)  | Tecnología esperada (AES)  | Tecnología esperada (DRM)          |
|----------------|----------------------------------------------------------|------------------------|----------------------|------------------------------|
| EdgeIE 11      | Windows 10, Windows 8.1, Windows Phone 101               | azureHtml5JS           | azureHtml5JS         | azureHtml5JS (PlayReady)     |
| IE 11IE 9-101  | Windows 7, Windows Vista<sup>1</sup>                     | flashSS                | flashSS              | silverlightSS (PlayReady)    |
| IE 11          | Windows Phone 8,1                                        | azureHtml5JS           | azureHtml5JS         | no admitido                |
| perimetral           | Xbox One<sup>1</sup> (actualización de noviembre de 2015)                   | azureHtml5JS           | azureHtml5JS         | no admitido                |
| Chrome 37+     | Windows 10, Windows 8.1, macOS X Yosemite<sup>1</sup>   | azureHtml5JS           | azureHtml5JS         | azureHtml5JS (Widevine)      |
| Firefox 47+    | Windows 10, Windows 8.1, macOS X Yosemite+<sup>1</sup>  | azureHtml5JS           | azureHtml5JS         | azureHtml5JS (Widevine)      |
| Firefox 42-46  | Windows 10, Windows 8.1, macOS X Yosemite+<sup>1</sup>  | azureHtml5JS           | azureHtml5JS         | silverlightSS (PlayReady)    |
| Firefox 35-41  | Windows 10, Windows 8.1                                  | flashSS                | flashSS              | silverlightSS (PlayReady)    |
| Safari         | iOS 6+                                                   | html5                  | html5 (sin token)3    | no admitido                |
| Safari 8+      | OS X Yosemite+                                           | azureHtml5JS           | azureHtml5JS         | html5FairPlayHLS (FairPlay)  |
| Safari 6       | OS X Mountain Lion<sup>1</sup>                           | flashSS                | flashSS              | silverlightSS (PlayReady)    |
| Chrome 37+     | Android 4.4.4+<sup>2</sup>                               | azureHtml5JS           | azureHtml5JS         | azureHtml5JS (Widevine)      |
| Chrome 37+     | Android 4.02                                             | html5                  | html5 (sin token)<sup>3</sup>    | no admitido                |
| Firefox 42+    | Android 5.0+<sup>2</sup>                                 | azureHtml5JS           | azureHtml5JS         | no admitido                |
| IE 8           | Windows                                                  | no admitido          | no admitido        | no admitido                |

<sup>1</sup> Configuración no compatible o no probada; aparece como referencia para su finalización.

<sup>2</sup> Para una reproducción correcta en dispositivos Android, es necesaria una combinación de funcionalidades del dispositivo, compatibilidad con gráficos, representación de códecs y compatibilidad del sistema operativo, entre otros requisitos. Como Android es una plataforma de código abierto que permite a los fabricantes de teléfonos cambiar el sistema operativo Vanilla Android que proporciona Google, esto provoca una compatibilidad irregular en Android, y puede que algunos dispositivos no sean compatibles debido a una falta de características. Además, algunos dispositivos Android no son compatibles con todos los códecs.  

<sup>3</sup> En los casos en los que no se admiten tokens, se puede usar un proxy para agregar esta funcionalidad. Consulte este [blog](https://azure.microsoft.com/blog/2015/03/06/how-to-make-token-authorized-aes-encrypted-hls-stream-working-in-safari/) para más información sobre esta solución.

> [!NOTE]
> Si la tecnología esperada requiere la instalación de un complemento (Flash, por ejemplo), y este no está instalado en la máquina del usuario, AMP seguirá comprobando las funcionalidades de la siguiente tecnología, junto con los tipos de origen y la información de protección, de la lista de tecnologías. Por ejemplo, si se intenta ver una transmisión a petición no protegida de Safari 8 en OS X Yosemite, y Flash y Silverlight no están instalados, AMP seleccionará Html5 nativo para la reproducción.<br/><br/>Aparecen nuevas tecnologías de explorador a diario y, por ello, esta matriz podría verse afectada.

## <a name="next-steps"></a>Pasos siguientes ##

- [Inicio rápido de Azure Media Player](azure-media-player-quickstart.md)