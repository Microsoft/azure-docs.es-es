---
title: Problemas conocidos de Azure Media Player
description: La versión actual tiene los siguientes problemas conocidos.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: reference
ms.date: 05/11/2020
ms.openlocfilehash: a31c089971b7e70e70c5906480deb2b17c197b9f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87043608"
---
# <a name="known-issues"></a>Problemas conocidos #

La versión actual tiene los siguientes problemas conocidos.

## <a name="azure-media-player"></a>Azure Media Player ##

- Los codificadores mal configurados pueden causar problemas de reproducción
- Las transmisiones con marcas de tiempo mayores que 2^53 pueden tener problemas de reproducción.
  - Mitigación: Use las escalas de tiempo de vídeo de 90-kHz y de audio de 44,1kHz.
- No hay reproducción automática en los dispositivos móviles sin la interacción del usuario. Está bloqueada por la plataforma.
- La búsqueda de discontinuidades cercanas puede producir un error de reproducción.
- La descarga de presentaciones grandes puede hacer que la interfaz de usuario se bloquee.
- No se puede reproducir automáticamente el mismo origen una vez finalizada la presentación.
  - Para volver a reproducir un origen una vez finalizado, es necesario volver a establecer el origen.
- Los manifiestos vacíos pueden causar problemas con el reproductor.
  - Este problema se puede producir al iniciar un streaming en vivo y no encontrar suficientes fragmentos en el manifiesto.
- La posición de reproducción puede estar fuera de la barra de búsqueda de la interfaz de usuario.
- El orden de los eventos no es coherente en todas las tecnologías.
- La propiedad almacenada en el búfer no es coherente entre las tecnologías.
- El valor de nativeControlsForTouch debe ser false (valor predeterminado) para evitar el error "El objeto no admite la propiedad o el método setControls".
- Los pósteres ahora deben ser direcciones URL absolutas.
- Se pueden producir problemas estéticos secundarios en la interfaz de usuario al usar el modo de contraste alto del dispositivo.
- Las direcciones URL que contienen "%" o "+" en la cadena totalmente descodificada pueden tener problemas al establecer el origen.

## <a name="ad-insertion"></a>Inserción de anuncios ##

- Los anuncios pueden sufrir problemas al insertarse (a petición o en directo) si hay un bloqueador de anuncios instalado en el explorador.
- Los dispositivos móviles pueden tener problemas al reproducir los anuncios.
- Actualmente, no se admiten las cuñas publicitarias intermedias en formato MP4 en Azure Media Player.

## <a name="azurehtml5js"></a>AzureHtml5JS ##

- En la ventana de DVR del contenido en directo, cuando el contenido finaliza, la escala de tiempo seguirá creciendo hasta que se realice una búsqueda en el área o hasta que llegue al final de la presentación.
- Las presentaciones en directo en Firefox con MSE habilitado tienen algunos problemas.

- Los recursos que son de audio no se reproducirán con la tecnología AzureHtml5JS.
  - Si desea reproducir recursos sin audio, puede hacerlo insertando audio en blanco con la [herramienta Explorador de Azure Media Services](https://aka.ms/amse).
  - [Aquí](../previous/media-services-advanced-encoding-with-mes.md#silent_audio) encontrará instrucciones sobre cómo insertar audio silencioso

## <a name="flash"></a>Intermitente ##

- El contenido de AES no se reproduce en la versión 30.0.0.134 de Flash debido a un error en la lógica de almacenamiento en caché de Adobe. Adobe ha corregido el problema y ha incluido la corrección en la versión 30.0.0.154
- Errores tecnológicos y de http (como los errores 404 de tiempos de expiración de la red), el reproductor tardará más en recuperarse que con otras tecnologías.
- Aunque haga clic en el área de vídeo con la tecnología flashSS, no se iniciará la reproducción ni se pausará el reproductor.
- Si el usuario tiene Flash instalado pero no concede permiso para cargarlo en el sitio, puede producirse un bucle infinito. Esto se debe a que el reproductor cree que el complemento está instalado y disponible, y considera que el complemento está ejecutando dicho contenido. El código de JavaScript se ha enviado pero la configuración del explorador ha bloqueado la ejecución del complemento hasta que el usuario conceda permiso para ello. Esto puede ocurrir en todos los exploradores.  

## <a name="silverlight"></a>Silverlight ##

- Características que faltan
- Errores tecnológicos y de http (como los errores 404 de tiempos de expiración de la red), el reproductor tardará más en recuperarse que con otras tecnologías.
- La reproducción de Safari y Firefox en Mac con Silverlight requiere definir explícitamente `"http://` o `https://` para el origen.
- Si falta una API para esta tecnología, normalmente devolverá null.
- Si el usuario tiene Flash instalado pero no concede permiso para cargarlo en el sitio, puede producirse un bucle infinito. Esto se debe a que el reproductor cree que el complemento está instalado y disponible, y considera que el complemento está ejecutando dicho contenido. El código de JavaScript se ha enviado pero la configuración del explorador ha bloqueado la ejecución del complemento hasta que el usuario conceda permiso para ello. Esto puede ocurrir en todos los exploradores.  

## <a name="native-html5"></a>HTML5 nativo ##

- La tecnología HTML5 solo está enviando el evento canplaythrough para el primer origen del conjunto.
- Esta tecnología solo admite lo que el explorador ha implementado.  Es posible que falten algunas características en esta tecnología.  
- Si falta una API para esta tecnología, normalmente devolverá null.
- Hay problemas con los títulos y subtítulos en esta tecnología. En esta tecnología, puede que estén disponibles y se puedan ver, o puede que no sea así.
- Tener un ancho de banda limitado en un escenario con la tecnología HLS/Html5 puede provocar la reproducción de audio sin vídeo.

### <a name="microsoft"></a>Microsoft ###

- La reproducción en IE8 no funciona actualmente debido a la incompatibilidad con ECMAScript 5.
- En Internet Explorer y algunas versiones de Edge, no se puede seleccionar la pantalla completa avanzando con el tabulador hasta el botón y seleccionándolo ni mediante la tecla de acceso rápido F/f.

## <a name="google"></a>Google ##

- Varios perfiles de codificación del mismo manifiesto tienen algunos problemas de reproducción en Chrome por lo que este no se recomienda.
- Chrome no puede reproducir HE-AAC con AzureHtml5JS. Conozca los detalles en el [seguimiento de errores](https://bugs.chromium.org/p/chromium/issues/detail?id=534301).
- A partir de la versión 58 de Chrome, el contenido de Widevine se debe cargar o reproducir mediante el protocolo https:// o, de lo contrario, se producirá un error de reproducción.

## <a name="mozilla"></a>Mozilla ##

- El conmutador de secuencias de audio requiere que estas tengan los mismos datos privados de códec cuando utilizan AzureHtml5JS. Esto es obligatorio para la plataforma FireFox.

## <a name="apple"></a>Apple ##

- Safari en Mac suele habilitar el modo de ahorro de energía de forma predeterminada con la opción "Stop plug-ins to save power" (Detener complementos para ahorrar energía), que bloquea complementos como Flash y Silverlight cuando considera que estos perjudican al usuario. Este bloqueo no bloquea el complemento en sí, solo sus funcionalidades. Si se proporciona el valor predeterminado de techOrder, esto puede causar problemas al intentar la reproducción.
  - Mitigación 1: Si el reproductor de vídeo es el foco de atención activo (dentro de un límite de 3000 x 3000 píxeles que comienza en la esquina superior izquierda del documento), la reproducción debería funcionar.
  - Mitigación2: Cambie techOrder para Safari para que sea ["azureHtml5JS", "HTML5"]. Esta mitigación implica la no obtención de todas las características que están disponibles en la tecnología de FlashSS.
- El contenido de PlayReady a través de Silverlight puede tener problemas al reproducirse en Safari.
- AES y el contenido restringido de los tokens no se reproducen con dispositivos iOS ni con los dispositivos más antiguos de Android.
  - Para lograr este escenario, se debe agregar un proxy al servicio.
- Se muestra la máscara predeterminada del teléfono iOS.
- La reproducción en iPhone siempre se produce en el modo de pantalla completa del reproductor nativo.
  - Es posible que las características, incluidos los subtítulos, no se conserven en esta reproducción no insertada.
- Cuando finaliza la presentación en directo, los dispositivos iOS no finalizan la presentación correctamente.
- iOS no permite funcionalidades de DVR.
- El conmutador de secuencias de audio de iOS solo se puede efectuar mediante la interfaz de usuario del reproductor nativo de iOS y requiere que las secuencias de audio tengan los mismos datos privados de códec.
- Las versiones más antiguas de Safari pueden tener problemas al reproducir secuencias en directo.

## <a name="older-android"></a>Versiones antiguas de Android ##

- AES y el contenido restringido de los tokens no se reproducen con dispositivos iOS ni con los dispositivos más antiguos de Android.
  - Para lograr este escenario, se debe agregar un proxy al servicio.

## <a name="next-steps"></a>Pasos siguientes ##

- [Inicio rápido de Azure Media Player](azure-media-player-quickstart.md)
