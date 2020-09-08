---
title: 'Configuración de la cuenta para el streaming sin conexión de contenido protegido de PlayReady: Azure'
description: En este artículo, se muestra cómo configurar la cuenta de Azure Media Services para transmitir por streaming PlayReady para Windows 10 sin conexión.
services: media-services
keywords: DASH, DRM, Modo sin conexión de Widevine, ExoPlayer, Android
documentationcenter: ''
author: willzhan
manager: steveng
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/16/2019
ms.author: willzhan
ms.custom: devx-track-csharp
ms.openlocfilehash: a26be590d5f7b467f57e8e18eac54ce57be24094
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89266602"
---
# <a name="offline-playready-streaming-for-windows-10"></a>Streaming de PlayReady sin conexión para Windows 10

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector" title1="Seleccione la versión de Media Services que se está usando:"]
> * [Versión 3](../latest/offline-plaready-streaming-for-windows-10.md)
> * [Versión 2](offline-playready-streaming-windows-10.md)

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Azure Media Services admite la descarga o reproducción sin conexión con protección DRM. En este artículo, se explica la compatibilidad sin conexión de Azure Media Services para clientes de Windows 10/PlayReady. Para obtener información sobre la compatibilidad del modo sin conexión para iOS/FairPlay y dispositivos Android/Widevine, vea los artículos siguientes:

- [Streaming de FairPlay sin conexión para iOS](media-services-protect-hls-with-offline-fairplay.md)
- [Streaming de Widevine sin conexión para Android](offline-widevine-for-android.md)

## <a name="overview"></a>Información general

En esta sección, se proporciona información sobre la reproducción del modo sin conexión, especialmente por estos motivos:

* En algunos países o regiones, la disponibilidad de Internet o de ancho de banda sigue siendo limitada. Los usuarios pueden decidir descargar primero para poder ver el contenido en una resolución lo suficientemente alta que les permita disfrutar de una experiencia de visualización satisfactoria. En este caso, con mayor frecuencia, el problema no es la disponibilidad de la red, sino que su ancho de banda es limitado. Los proveedores OTT/OVP están solicitando compatibilidad con el modo sin conexión.
* Como se informó en la conferencia de accionistas de Netflix del tercer trimestre de 2016, la descarga de contenido es una “característica muy solicitada” y, según afirma Reed Hastings (consejero delegado de Netflix), “están estudiando su implementación”.
* Algunos proveedores de contenido pueden suspender la entrega de licencias de DRM al cruzar la frontera de un país o una región. Si un usuario necesita viajar al extranjero y quiere ver contenido, se necesita la descarga sin conexión.
 
La dificultad a la que nos enfrentamos al implementar el modo sin conexión es la siguiente:

* MP4 es compatible con una gran variedad de reproductores y herramientas de codificador, pero no hay ningún enlace entre el contenedor de MP4 y DRM.
* A largo plazo, la forma correcta es usar CFF con CENC. Pero, actualmente, el ecosistema de compatibilidad de herramientas o reproductores aún no está preparado. Necesitamos una solución hoy mismo.
 
La idea es esta: el formato de archivo de streaming con velocidad de transmisión adaptable ([PIFF](/iis/media/smooth-streaming/protected-interoperable-file-format)) con H264/AAC tiene un enlace con PlayReady (AES-128 CTR). El archivo .ismv individual de streaming con velocidad de transmisión adaptable (siempre que el audio se multiplexe en el vídeo) es en sí un archivo fMP4 y puede usarse para la reproducción. Si un contenido de streaming con velocidad de transmisión adaptable pasa por un cifrado de PlayReady, cada archivo .ismv se convierte en un MP4 fragmentado protegido por PlayReady. Podemos seleccionar un archivo .ismv con la velocidad de bits preferida y cambiar su extensión a .mp4 para su descarga.

Existen dos opciones para hospedar los archivos MP4 protegidos por PlayReady para su descarga progresiva:

* En una, se puede usar este MP4 en el mismo contenedor o activo de servicio multimedia y usar el punto de conexión de streaming de Azure Media Services para la descarga progresiva.
* En otra, se puede usar el localizador de SAS para realizar la descarga progresiva directamente desde Azure Storage, omitiendo Azure Media Services.
 
Se pueden usar dos tipos de entrega de licencias de PlayReady:

* Servicio de entrega de licencia de PlayReady en Azure Media Services.
* Servidores de licencia de PlayReady hospedados en cualquier lugar.

A continuación, se muestran dos conjuntos de activos de prueba; el primero usa la entrega de licencia de PlayReady en AMS, mientras que el segundo usa el servidor de licencias de PlayReady hospedados en una VM de Azure:

Recurso 1:

* URL de descarga progresiva: [https://willzhanmswest.streaming.mediaservices.windows.net/8d078cf8-d621-406c-84ca-88e6b9454acc/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4](https://willzhanmswest.streaming.mediaservices.windows.net/8d078cf8-d621-406c-84ca-88e6b9454acc/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4)
* PlayReady LA_URL (AMS): `https://willzhanmswest.keydelivery.mediaservices.windows.net/PlayReady/`

Recurso 2:

* URL de descarga progresiva: [https://willzhanmswest.streaming.mediaservices.windows.net/7c085a59-ae9a-411e-842c-ef10f96c3f89/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4](https://willzhanmswest.streaming.mediaservices.windows.net/7c085a59-ae9a-411e-842c-ef10f96c3f89/20150807-bridges-2500_H264_1644kbps_AAC_und_ch2_256kbps.mp4)
* PlayReady LA_URL (entorno local): `https://willzhan12.cloudapp.net/playready/rightsmanager.asmx`

Para la prueba de reproducción, he usado una aplicación universal de Windows en Windows 10. En los [ejemplos universales de Windows 10](https://github.com/Microsoft/Windows-universal-samples), encontrará un reproductor básico de ejemplo denominado [Ejemplo de streaming adaptable](https://github.com/Microsoft/Windows-universal-samples/tree/master/Samples/AdaptiveStreaming). Solo tenemos que agregar el código para seleccionar el vídeo descargado y usarlo como el origen, en lugar del origen de streaming adaptable. Los cambios se realizan en el controlador de eventos de clic del botón:

```csharp
private async void LoadUri_Click(object sender, RoutedEventArgs e)
{
    //Uri uri;
    //if (!Uri.TryCreate(UriBox.Text, UriKind.Absolute, out uri))
    //{
    // Log("Malformed Uri in Load text box.");
    // return;
    //}
    //LoadSourceFromUriTask = LoadSourceFromUriAsync(uri);
    //await LoadSourceFromUriTask;

    //willzhan change start
    // Create and open the file picker
    FileOpenPicker openPicker = new FileOpenPicker();
    openPicker.ViewMode = PickerViewMode.Thumbnail;
    openPicker.SuggestedStartLocation = PickerLocationId.ComputerFolder;
    openPicker.FileTypeFilter.Add(".mp4");
    openPicker.FileTypeFilter.Add(".ismv");
    //openPicker.FileTypeFilter.Add(".mkv");
    //openPicker.FileTypeFilter.Add(".avi");

    StorageFile file = await openPicker.PickSingleFileAsync();

    if (file != null)
    {
       //rootPage.NotifyUser("Picked video: " + file.Name, NotifyType.StatusMessage);
       this.mediaPlayerElement.MediaPlayer.Source = MediaSource.CreateFromStorageFile(file);
       this.mediaPlayerElement.MediaPlayer.Play();
       UriBox.Text = file.Path;
    }
    else
    {
       // rootPage.NotifyUser("Operation cancelled.", NotifyType.ErrorMessage);
    }

    // On small screens, hide the description text to make room for the video.
    DescriptionText.Visibility = (ActualHeight < 500) ? Visibility.Collapsed : Visibility.Visible;
}
```

 ![Reproducción del modo sin conexión de fMP4 protegido por PlayReady](./media/offline-playready/offline-playready1.jpg)

Como el vídeo se encuentra bajo la protección de PlayReady, en la captura de pantalla no se puede incluir el vídeo.

En resumen, hemos obtenido el modo sin conexión en Azure Media Services:

* La transcodificación de contenido y el cifrado de PlayReady pueden realizarse en Azure Media Services o en otras herramientas.
* El contenido se puede hospedar en Azure Media Services o Azure Storage para su descarga progresiva.
* La entrega de licencia de PlayReady puede realizarse desde Azure Media Services o desde otra ubicación.
* El contenido preparado de streaming con velocidad de transmisión adaptable puede usarse para el streaming en línea mediante DASH o transmisión suave con PlayReady como el DRM.

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="next-steps"></a>Pasos siguientes

[Diseño del sistema DRM híbrido](hybrid-design-drm-sybsystem.md)
