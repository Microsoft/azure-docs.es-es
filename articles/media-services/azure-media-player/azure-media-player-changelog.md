---
title: Registro de cambios de Azure Media Player
description: Registro de cambios de Azure Media Player.
author: IngridAtMicrosoft
ms.author: inhenkel
ms.service: media-services
ms.topic: overview
ms.date: 07/27/2020
ms.openlocfilehash: 9642f6d1090ef358cf86241640255c647dea52eb
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432660"
---
# <a name="changelog"></a>Registro de cambios #

## <a name="235-official-update-june-1-2020"></a>2.3.5 (actualización oficial del 1 de junio de 2020)

### <a name="bug-fixes-235"></a>Correcciones de errores 2.3.5

- [Accesibilidad] El cliente de escucha de la tecla Esc en el panel Opciones está conectado al documento.
- [Accesibilidad] La interfaz de usuario del reproductor ya no desaparece cuando la barra de control o el menú de opciones tiene el foco.
- La barra de control muestra el tiempo de reloj incorrecto cuando está habilitada la configuración de pantalla del tiempo de reloj.

### <a name="changes-235"></a>Cambios 2.3.5

- Se ha agregado el mensaje de error para el código de error 0x00400005 y se ha agregado a la documentación.

## <a name="234-official-update-march-4-2020"></a>2.3.4 (actualización oficial del 4 de marzo de 2020)

### <a name="bug-fixes-234"></a>Correcciones de errores 2.3.4

- No se puede establecer el valor de overrideLicenseAcquistionUrl de PlayReady.
- No se puede reproducir contenido con interrupciones.
- [Accesibilidad] El valor del atributo de id. para la alerta del lector de pantalla debe ser único.
- [Accesibilidad] Al navegar por el cuadro de diálogo Configuración de subtítulos, el foco se desplaza fuera del cuadro de diálogo.

### <a name="changes-234"></a>Cambios 2.3.4

- Longitud del contenido del registro después de una descarga correcta para ayudar a analizar los errores de descifrado 2.3.3 (actualización oficial del 12 de noviembre de 2019).

### <a name="features-234"></a>Características 2.3.4

- Se ha agregado compatibilidad para mostrar el tiempo de reloj de un vídeo como superposición, así como en la barra de control.

### <a name="bug-fixes-234"></a>Correcciones de errores 2.3.4

- El cambio de pista de audio funciona, pero genera un error en IE11 y Windows 7 "El objeto no admite la propiedad o el método 'habilitado'".
- El cambio de pista de audio produce un error cuando el búfer se ha cargado por completo.
- El cambio de pista de audio produce un error cuando el usuario pausa el vídeo y cambia entre pistas de audio con mucha rapidez.
- [Accesibilidad] La información sobre herramientas no está definida para el control de vídeo en el reproductor de vídeo.
- Faltan los botones de volumen en HTML5 en función del momento en que se recibe "loadstart".
- [Accesibilidad] No hay forma de establecer el texto alternativo para la imagen de póster.
- [Accesibilidad] Se pierde el foco de la aplicación después de seleccionar "Listo" en el cuadro de diálogo Configuración de subtítulos.
- [Accesibilidad] Se definieron atributos Aria incorrectos para "video" en "segments preview".

### <a name="changes-234"></a>Cambios 2.3.4

- Se ha quitado la etiqueta o pista de subtítulo vacía al reproducir HLS en iOS y macOS Safari.
- Se ha reducido el número de respuestas 412 para los subtítulos IMSC1.
- Advertencia de salida en la consola para 10 respuestas de subtítulo IMSC1 vacíos consecutivas para ayudar a la depuración en directo.

## <a name="232-official-update-october-9-2019"></a>2.3.2 (actualización oficial del 9 de octubre de 2019)

### <a name="features-232"></a>Características de la versión 2.3.2

-Se ha agregado compatibilidad con PlayReady para la reproducción de DASH del explorador Edge basado en Chromium.

### <a name="bug-fixes-232"></a>Correcciones de errores 2.3.2

- La velocidad de reproducción actual no se muestra visualmente en el menú de velocidad de reproducción a menos que el usuario la configure manualmente.
- [Accesibilidad] El panel "Configuración" no se contrae con la tecla "Esc".
- [Accesibilidad] La tecla de método abreviado "M" de AMP no funciona cuando el narrador está activado.

### <a name="changes-232"></a>Cambios 2.3.2

- En el caso de los exploradores que no admiten el códec de audio E-AC3, las pistas de audio E-AC3 están ocultas en el menú de pista de audio.
- En el caso de los exploradores que admiten el códec de audio E-AC3, una pista de audio E-AC3 está seleccionada de forma predeterminada.
- En el caso de los exploradores que no admiten el cambio de códec de audio, las pistas de audio con un códec diferente al de la pista seleccionada se ocultan en el menú de pistas de audio.

## <a name="231-official-update-august-12-2019"></a>2.3.1 (actualización oficial del 12 de agosto de 2019)

### <a name="features-231"></a>Características 2.3.1

- Señalar un evento en el que se reciben cuadros "emsg" en la reproducción DASH: se ha agregado compatibilidad para mostrar las pistas de audio EC-3 en el menú Audio de los exploradores que admiten EC-3 y permitir el cambio de la pista de audio de AAC a EC3 y viceversa solo en el explorador Edge basado en Chromium.

### <a name="bug-fixes-231"></a>Correcciones de errores 2.3.1

- El menú Pista de audio se daña después de quitar las pistas EC-3.
- El tiempo actual puede ser mayor que la duración del vídeo.
- No es posible establecer la velocidad de reproducción mediante initialSpeed.
- A veces después de una búsqueda, el reproductor parece bloqueado.
- En Edge e IE con una pantalla táctil, después de acercar una página, presionar o mantener el mouse sobre la barra de búsqueda no se obtiene de forma precisa el segmento correcto del vídeo.
- [Accesibilidad] La etiqueta Aria para Reproducir/Pausar no describe al reproductor de vídeo. Asignación del error Segmento en directo no encontrado para flashSS al error de AMP correcto.
- [Accesibilidad] Los roles Aria usados para Reproducir/Pausar deben ajustarse a los valores válidos (.vjs-text-track-display).
- [Accesibilidad] Determinados roles Aria deben incluirse en elementos primarios específicos.
- [Accesibilidad] No se ha definido información sobre herramientas para el botón Reproducir/Pausar del reproductor de vídeo. Los subtítulos IMSC1 pueden desaparecer después de realizar búsquedas en el búfer de vídeo/audio actual.

### <a name="changes-231"></a>Cambios 2.3.1

- Al obtener un elemento segmentDecryptError cuando el reproductor ya está en el perímetro activo, el reproductor ahora actualiza el manifiesto en lugar de probar el siguiente segmento.
- Se han agregado más registros para diagnóstico.
- Se ha actualizado la documentación para incluir la compatibilidad de FairPlay con iOS Safari.
- Se ha agregado un ejemplo para la opción "srclang" de IMSC1.
- Se han agregado invalidaciones para los valores de padding, textPadding y boxShadow de las pistas de texto.
- Se ha agregado un código de error (0x0020025B) para diferenciar un error de descarga de segmento debido a que no hay Internet, en lugar de simplemente devolver 0x00200259.

## <a name="230-official-release-april-30-2019"></a>2.3.0 (lanzamiento oficial el 30 de abril de 2019)

### <a name="features-230"></a>Características 2.3.0

- Se ha agregado compatibilidad con los subtítulos IMSC1 para DASH.
- Se ha agregado compatibilidad con los recursos solo de vídeo para DASH.
- Se agregado presentationTimeOffsetInSec a la API.

### <a name="bug-fixes-230"></a>Correcciones de errores 2.3.0

- El perfil de heurística LowLatency de AMP interfiere con la reproducción de vídeo de iOS. Las opciones "silenciar" y "reactivar audio" de algunos idiomas tienen traducciones erróneas.
- A veces, el valor Aria "now" del control deslizante de la barra de progreso a veces es incorrecto.
- El valor Aria de "role" de la visualización de la pista de texto es incorrecto.

### <a name="changes-230"></a>Cambios 2.3.0

- Los registros ahora incluyen el tamaño de los fragmentos multimedia descargados.
- Se ha quitado la compatibilidad con IE 9 e IE 10.
- Se ha actualizado el ejemplo CEA708 para mostrar los subtítulos con alineación izquierda.
- Se incluye MediaError.message en los registros de errores de reproducción.

## <a name="224-official-update-february-22-2019"></a>2.2.4 (actualización oficial del 22 de febrero de 2019) ##

### <a name="bug-fixes-224"></a>Correcciones de errores en la versión 2.2.4 ###

- [Corrección de errores][AMP][Accesibilidad] Se ha quitado una pestaña fantasma a la que se puede acceder cuando aparecía la pantalla de error.
- [Corrección de errores][AMP] Se ha corregido la tecla de acceso rápido "M" para IE11 y Edge.
- [Corrección de errores][AMP] Se ha corregido una excepción para los subtítulos de CEA708.
- [Corrección de errores][AMP] Se ha corregido un problema de inmovilización de vídeo en el explorador Edge.

### <a name="changes-224"></a>Cambios en la versión 2.2.4 ###

- [Cambio][AMP] Cuando se produce un error de descifrado de fragmentos, el reproductor reintenta tanto el fragmento actual como otros para recuperar la reproducción.
- [Cambio][AMP] Ahora, AMP es más tolerante a la superposición de fragmentos de vídeo o audio.

## <a name="223-official-update-january-9-2019"></a>2.2.3 (actualización oficial del 9 de enero de 2019) ##

### <a name="features-223"></a>Características 2.2.3 ###

- [Característica][HLS] Se ha agregado el menú de pista de audio para la reproducción de Safari HLS.

### <a name="bug-fixes-223"></a>Correcciones de errores en la versión 2.2.3 ###

- [Corrección de errores][AMP][Accesibilidad] Durante las reproducciones de retransmisión en directo, el botón "live" (en vivo) no se puede seleccionar con el teclado.
- [Corrección de errores][AMP] Se han corregido falsos positivos del error 0x0400003 debido a un error en la prueba de MSE.
- [Corrección de errores][AMP] Se ha corregido el problema por el que el vídeo se queda inmovilizado al iniciar un streaming en vivo.

### <a name="changes-223"></a>Cambios en la versión 2.2.3 ###

- [Cambio][AMP] Se ha agregado más información al registro para mejorar los diagnósticos.
- [Cambio][AMP] Cuando hay varias velocidades de bits disponibles en la misma resolución de pantalla, todas ellas están disponibles y se pueden seleccionar.

## <a name="222-official-update"></a>2.2.2 (actualización oficial) ##

### <a name="bug-fixes-222"></a>Correcciones de errores en la versión 2.2.2 ###

- [Corrección de errores][AMP] Cuando el reproductor encuentra una interrupción de red transitoria, detiene la reproducción de inmediato.
- [Corrección de errores][AMP][Accesibilidad] No se puede acceder al cuadro de diálogo de error desde el teclado.
- [Corrección de errores][AMP] Se muestra un indicador giratorio infinito al reproducir un recurso de solo audio, en lugar de un error de falta de compatibilidad.

### <a name="changes-222"></a>Cambios en la versión 2.2.2 ###

- [Cambio][AMP] Se han agregado cadenas localizadas para la interfaz de usuario de anuncios.

## <a name="221-official-update"></a>2.2.1 (actualización oficial) ##

### <a name="features-221"></a>Características de la versión 2.2.1 ###

- [Característica][CMAF] Se ha agregado compatibilidad con HLS CMAF.

### <a name="bug-fixes"></a>Correcciones de errores ###

- [Corrección de errores][AMP] Temporizadores sin borrar en la lógica de reintentos que producían errores de reproducción.
- [Corrección de errores][AMP][Firefox] El evento Ended no se desencadena en Firefox y Chrome cuando se detiene el programa en vivo.
- [Corrección de errores][AMP] Los controles se muestran después de Setsource, aun cuando los controles se establezcan en false en las opciones del reproductor.

### <a name="changes"></a>Cambios ###

- [Cambio][Subtítulos en vivo] Se ha cambiado el nombre de la API en los subtítulos de CEA de 608 a 708. Para más información, consulte [Configuración de subtítulos de CEA708](/javascript/api/azuremediaplayer/amp.player.cea708captionssettings)-->

## <a name="220-official-release"></a>2.2.0 (versión oficial) ##

### <a name="features-220"></a>Características de versión 2.2.0 ###

- [Característica][Azurehtml5JS][Flash][LiveCaptions] Se ha agregado compatibilidad con subtítulos de CEA 708 en Azurehtml5JS y la tecnología FlashSS para contenido descifrado y AES.

### <a name="bug-fixes-220"></a>Correcciones de errores en la versión 2.2.0 ###

- [Corrección de errores] La detección de la versión de Flash no funciona en Chrome/Edge.

### <a name="changes-220"></a>Cambios en la versión 2.2.0 ###

- [Cambio][AMP][Heurística] Se ha cambiado el nombre de perfil de heurística de QuickStartive a LowLatency.
- [Cambio][Flash] Se ha cambiado la detección de la versión del reproductor de Flash para habilitar la reproducción de contenido AES con la nueva actualización de Adobe Flash.

## <a name="219-official-hotfix"></a>2.1.9 (revisión oficial) ##

### <a name="bug-fixes-219"></a>Correcciones de errores de la versión 2.1.9 ###

- [Corrección de errores][Directo] Se produce una excepción cuando el streaming en vivo pasa a vídeo bajo demanda o archivos en vivo.

### <a name="changes-219"></a>Cambios en la versión 2.1.9 ###

- [Cambio][Flash][AES] Se ha modificado la lógica de la tecnología Flash para que no se use sharedbytearrays para el descifrado de AES, ya que Adobe ha bloqueado el uso a partir de Flash 30. Tenga en cuenta que la reproducción solo funcionará cuando Adobe implemente una nueva versión de Flash debido a un error en la versión 30. Para más información, consulte [Problemas conocidos](azure-media-player-known-issues.md)

## <a name="218-official-update"></a>2.1.8 (actualización oficial) ##

### <a name="bug-fixes-218"></a>Correcciones de errores en la versión 2.1.8 ###

- [Corrección de errores] El indicador giratorio a veces no se muestra después de la búsqueda y antes de la reproducción.
- [Corrección de errores] El reproductor no comienza en silencio cuando la opción de silencio está habilitada.
- [Corrección de errores] Se muestra el control del volumen cuando los controles se establecen en false.
- [Corrección de errores] La reproducción se repite ocasionalmente cuando el usuario salta al extremo en vivo.
- [Corrección de errores][Firefox] Ocasionalmente, el reproductor inicia una excepción de JavaScript al cargarse.
- [Corrección de errores][Accesibilidad] El botón Reproducir/Pausa/Volumen pierde el contorno del foco cuando se selecciona mediante los controles del teclado.
- [Corrección de errores] Se ha corregido una pérdida de memoria en el reproductor cuando se deshecha.
- [Corrección de errores] Al llamar a src() después de los errores de salida del reproductor no se restablece el origen.
- [Corrección de errores][En vivo] AMP está en estado de carga constante cuando el usuario hace clic en el botón Live (En vivo) después de que la retransmisión haya terminado.
- [Corrección de errores][Chrome] El reproductor se detiene y se produce un error en la reproducción cuando el explorador se minimiza y queda en segundo plano.

### <a name="changes-218"></a>Cambios en la versión 2.1.8 ###

- [Cambio] Se ha actualizado el error 0x0600001 para que se muestre al reproducir contenido AES con Flash 30, ya que no se admite actualmente. Para más información, consulte [Problemas conocidos](azure-media-player-known-issues.md)
- [Cambio] Se han agregado más reintentos para los escenarios en vivo cuando el manifiesto solicita 404 o devuelve manifiestos vacíos.

## <a name="217-official-update"></a>2.1.7 (actualización oficial) ##

### <a name="features-217"></a>Características de la versión 2.1.7 ###

- [Característica][AzureHtml5JS] Se ha agregado una opción de configuración para vaciar los datos obsoletos del búfer de origen de los elementos multimedia.

### <a name="bug-fixes-217"></a>Correcciones de errores en la versión 2.1.7 ###

- [Corrección de errores][Accesibilidad][Lector de pantalla] Se ha eliminado el encabezado en blanco que incluía el reproductor cuando no se establece ningún título.
- [Corrección de errores][UWA] AMP inicia una excepción al reproducir en la Aplicación universal de Windows.
- [Corrección de errores][OSX] setActiveTextTrack() no funciona en Safari en OSx.
- [Corrección de errores][En vivo] Al hacer clic en el borde en vivo después de desecharlo y reiniciar el reproductor, se produce una excepción.
- [Corrección de errores][Máscara] La hora actual está truncada en ciertos recursos.
- [Corrección de errores][DRM] Se ha incluido una corrección para admitir la reproducción en exploradores que admiten varios DRM de CENC.

### <a name="changes-217"></a>Cambios en la versión 2.1.7 ###

- [Cambio][Ejemplos][Accesibilidad] Se ha agregado una etiqueta de idioma en todos los ejemplos.

## <a name="216-official-update"></a>2.1.6 (actualización oficial) ##

### <a name="bug-fixes-216"></a>Correcciones de errores en la versión 2.1.6 ###

- [Corrección de errores] AMP muestra una duración incorrecta para el recurso específico.
- [Corrección de errores][FairPlay-HLS] Los errores de Fairplay no se propagan a la interfaz de usuario.
- [Corrección de errores] Las propiedades heurísticas personalizadas se omiten en AMP 2.1.5.

### <a name="changes-216"></a>Cambios en la versión 2.1.6 ###

- [Cambio][FairPlayDRM] Se ha quitado el tiempo de espera tanto de la solicitud de certificación como de la solicitud de licencia de FairPlay, con el fin de mantener la paridad con las implementaciones de PlayReady y Widevine.
- [Cambio] Distintas mejoras heurísticas para combatir el contenido borroso.

### <a name="features-216"></a>Características de la versión 2.1.6 ###

- [Característica] Se ha agregado compatibilidad con el formato mpd-time-cmaf.

## <a name="215-official-hotfix"></a>2.1.5 (revisión oficial) ##

### <a name="bug-fixes-215"></a>Correcciones de errores en la versión 2.1.5 ###

- [Corrección de errores][Subtítulos] El reproductor no representa correctamente los estilos de VTT.
- [Corrección de errores][Accesibilidad] El botón Live (En vivo) no tiene la etiqueta aria.

## <a name="214-official-update"></a>2.1.4 (actualización oficial) ##

### <a name="bug-fixes-214"></a>Correcciones de errores en la versión 2.1.4 ###

- [Corrección de errores][Accesibilidad][Foco] Los usuarios no pueden usar el tabulador para cambiar el foco a los botones personalizados que se agregan a la derecha del botón de pantalla completa de la barra de control.
- [Corrección de errores][IE11][Barra de volumen] Al desplazarse mediante el tabulador al elemento emergente de volumen, toda la pantalla de vídeo parpadea en IE11 cuando está en pantalla completa.
- [Corrección de errores][Máscara|Vaciar] Se muestra un espacio entre la barra de control y el elemento emergente de la barra de volumen.
- [Corrección de errores][AMP][Subtítulos] Las pistas antiguas insertadas no se borran cuando se cambia el origen en un reproductor existente.
- [Corrección de errores][Accesibilidad][Narrador] El lector de pantalla lee el control de volumen de forma incorrecta.
- [Corrección de errores][FlashSS] En ocasiones, el evento Play no se desencadena desde la tecnología Flash.
- [Corrección de errores][AMP][Foco] Para reproducir/pausar se requieren dos clics cuando el reproducir tiene el foco y está en modo de pantalla completa.
- [Corrección de errores][AMP][Máscara] Se muestra una duración incorrecta en la barra de progreso de un recurso específico.
- [Corrección de errores][Anuncios][AdButler] El analizador VAST no controla el archivo VAST que no tiene el evento de progreso.
- [Corrección de errores][SDN][AMP 2.1.1] Se ha corregido el problema de compatibilidad con el complemento SDN de Hive.
- [Corrección de errores][Accesibilidad] El narrador lee "Midnight Mute Button" (Botón silenciar a media noche) cuando el usuario tiene el foco en el botón de volumen.

### <a name="changes-214"></a>Cambios en la versión 2.1.4 ###

- [Cambio][Accesibilidad][Tecnología de asistencia] Ahora, los botones ahora tienen la propiedad aria-live para mejorar la experiencia con la tecnología de asistencia.
- [Cambio][Accesibilidad][Botón de volumen|Narrador] Se ha mejorado la accesibilidad del botón de volumen mediante la modificación de la funcionalidad de tabulación y el comportamiento del control deslizante. Estos cambios facilitan a los usuarios del teclado la modificación del volumen del reproductor.
- [Cambio] Se ha aumentado de 3 a 5 segundos el tiempo que tarda el menú contextual en estar inactivo.
- [Cambio][Accesibilidad][Luminosidad] Se ha mejorado la relación de contraste de luminosidad en los menús desplegables de la configuración de los subtítulos.

## <a name="213-official-update"></a>2.1.3 (actualización oficial) ##

### <a name="bug-fixes-213"></a>Correcciones de errores en la versión 2.1.3 ###

- [Corrección de errores][Complementos|Title Overlay] El complemento Title Overlay produce excepciones de JS con AMP V2.X+.
- [Corrección de errores] El evento Source Set se envía a la consola de JavaScript aunque el registro esté desactivado.
- [Corrección de errores][Máscara] La información sobre el tiempo del reproductor se representa fuera del contexto del reproductor al desplazarse sobre ambos extremos de la barra de duración.
- [Corrección de errores][Accesibilidad][Lector de pantalla] El narrador lee "Region Landmark" (Punto de referencia regional) o "Video Player Region Landmark" (Punto de referencia regional de reproductor de vídeo) cuando el visor tiene el foco en el reproductor.
- [Corrección de errores][AMP] No se puede deshabilitar el contorno del reproductor mediante CSS.
- [Corrección de errores][Accesibilidad] No se puede usar el tabulador para centrarse en el reproductor completo cuando el usuario está en modo de pantalla completa.
- [Corrección de errores][Máscara][En vivo] La máscara no responde al texto LIVE (EN VIVO) localizado en japonés.
- [Corrección de errores][Máscara] La duración y la hora actual se recortan cuando la transmisión dura más de 60 minutos-[Corrección de errores][iPhone|En vivo] El reproductor muestra el texto de la hora o duración actuales en la barra de control.
- [Corrección de errores][AMP] La llamada a las API de heurística del reproductor produce excepciones de JavaScript.
- [Corrección de errores][HTML5 nativo|iOS] La propiedad de Videotag "playsinline" no se propaga al reproductor.
- [Corrección de errores][iOS|iframe] El reproductor no puede entrar en modo de pantalla completa en iPhone si el reproductor está cargado en un iFrame.
- [Corrección de errores][AMP][Heurística] AMP siempre funciona con el perfil híbrido, independientemente de las opciones del reproductor.
- [Corrección de errores][AMP|Win8.1] Se inicia cuando se hospeda en una aplicación Win8.1 con una vista web.

### <a name="changes-213"></a>Cambios en la versión 2.1.3 ###

- [Cambio][AMP] Se ha agregado información del punto de conexión de CDN al evento FragmentDownloadComplete.
- [Cambio][AMP][En vivo] Se ha mejorado y optimizado la latencia del streaming en vivo.

## <a name="212-official-hotfix"></a>2.1.2 (revisión oficial) ##

### <a name="bug-fixes-212"></a>Correcciones de errores en la versión 2.1.2 ####

- [Corrección de errores][Accesibilidad][Narrador de Windows] El narrador lee "Progress midnight" (Progreso a medianoche) cuando el usuario tiene el contexto de la barra de progreso y la hora actual es 0:00.
- [Corrección de errores][Máscara] El tamaño del logotipo está codificado de forma rígida en el código JavaScript.
- [Accesibilidad][Teclas de acceso rápido] Las teclas de acceso rápido no se habilitan al hacer clic en el reproductor.

### <a name="changes-212"></a>Cambios en la versión 2.1.2 ####

- [Cambios][Registro] Se registra la dirección URL del manifiesto cuando el reproductor no carga el manifiesto.

### <a name="features-212"></a>Características de la versión 2.1.2 ###

- [Cambio][Rendimiento][Optimización] Se han mejorado los tiempos de inicio y carga del reproductor.

## <a name="211-official-update"></a>2.1.1 (actualización oficial) ##

### <a name="bug-fixes-211"></a>Correcciones de errores en la versión 2.1.1 ####

- [Corrección de errores][iOS] Al establecer la opción de reproducción automática en false, se genera un indicador giratorio infinito en Safari para iOS.
- [Corrección de errores] Si se busca un tiempo mayor que la duración del contenido, se genera un indicador giratorio infinito.
- [Corrección de errores] Las teclas de acceso rápido requieren varias tabulaciones del teclado para que el contexto del reproductor funcione.
- [Corrección de errores] El vídeo se queda inmóvil unos segundos después de cambiar el tamaño del reproductor en ciertos recursos.
- [Corrección de errores] Se muestra un indicador giratorio infinito (una vez que se completa la búsqueda) cuando el usuario realiza varias búsquedas rápidamente.
- [Corrección de errores] La barra de control no se oculta durante la inactividad.
- [Corrección de errores] La apertura de una aplicación web que hospeda AMP puede causar que la página web se cargue dos veces.
- [Corrección de errores] Se muestra un indicador giratorio infinito al reproducir contenido de determinados recursos mediante tecnología Flash.
- [Corrección de errores] El menú More Options (Más opciones) no se muestra con complementos de terceros.
- [Corrección de errores][Máscara|Tube][En vivo] Se muestran dos iconos dinámicos cuando el reproductor está en el extremo en vivo de un programa.
- [Corrección de errores][Máscara] El logotipo no se puede deshabilitar.
- [Corrección de errores][DD + contenido] Se muestra un indicador giratorio continuo en los recursos que contienen una pista de audio Dolby Digital.
- [Corrección de errores] El AMP más reciente se inmoviliza al cambiar las pistas del idioma de audio durante el streaming en vivo.
- [Corrección de errores] Desaparece el fondo fijo del indicador giratorio.
- [Corrección de errores] Se han corregido los errores por lo que se mostraba un indicador giratorio infinito en los ejemplos estáticos de tokens de Flash AES.

### <a name="changes-211"></a>Cambios en la versión 2.1.1 ####

- [Cambio] Se ha agregado código de error para el requisito de HTTPS de Widevine: a partir de la versión 58 de Chrome, el contenido de Widevine se debe cargar o reproducir mediante el protocolo `https://`; de lo contrario, se producirá un error de reproducción.
- [Cambio] Se ha agregado la etiqueta aria para cargar el indicador giratorio, de modo que la tecnología de asistencia pueda narrar "video loading" (cargando vídeo) cuando se carga el contenido.  

## <a name="210-official-release"></a>2.1.0 (versión oficial) ##

### <a name="features-210"></a>Características de la versión 2.1.0 ###

- [Característica][AzureHtml5JS] Se ha agregado compatibilidad con anuncios VoD al principio, a la mitad y al final de la reproducción.
- [Característica][Beta][AzureHtml5JS] Se ha agregado compatibilidad con anuncios en vivo a la mitad y al final de la reproducción.
- [Característica] Se ha agregado una nueva opción de máscara: vaciado de AMP.
- [Característica] Se han agregado etiquetas aria mejoradas para una mejor integración tanto con los lectores de pantalla como con la tecnología de asistencia.
- [Característica][Máscara] La máscara ahora muestra todos los iconos y botones con claridad en el modo de contraste alto.

### <a name="bug-fixes-210"></a>Correcciones de errores en la versión 2.1.0 ###

- [Corrección de errores] Diversas correcciones de accesibilidad y de la interfaz de usuario.
- [Corrección de errores] AMP no se carga correctamente en IE9.

### <a name="changes-210"></a>Cambios en la versión 2.1.0 ###

- [Cambio] Se han reestructurado los elementos de DOM en el reproductor para que los anuncios funcionen.
- [Cambio] Se ha cambiado de CSS a SCSS para el desarrollo de máscaras.
- [Cambio][Ejemplo] Se ha agregado un ejemplo de anuncios VoD.
- [Cambio][Ejemplo] Se ha agregado un ejemplo de velocidad de reproducción.
- [Cambio][Ejemplo] Se ha agregado un ejemplo de máscara de vaciado.

## <a name="200-beta-release"></a>2.0.0 (versión beta) ##

- [Cambio] Se ha actualizado a VJS5.
- [Característica] Se ha agregado una nueva API para que la capacidad de respuesta del reproductor sea más fluida.
- [Característica] Velocidad de reproducción.
- [Característica] Se ha cambiado de CSS a SCSS para la máscara.

## <a name="183-official-hotfix-update"></a>1.8.3 (actualización de revisión oficial) ##

### <a name="bug-fixes-183"></a>Correcciones de errores en la versión 1.8.3 ###

- [Corrección de errores][AzureHtml5JS] Determinados recursos con DTS negativo no se reproducirán en Chrome.

## <a name="182-official-hotfix-update"></a>1.8.2 (actualización de revisión oficial) ##

### <a name="bug-fixes-182"></a>Correcciones de errores en la versión 1.8.2 ###

- [Corrección de errores][AzureHtml5JS] Las velocidades de bits de audio altas no se reproducirán a través de AzureHtml5JS.

## <a name="181-official-update"></a>1.8.1 (actualización oficial) ##

### <a name="bug-fixes-181"></a>Correcciones de errores en la versión 1.8.1 ###

- [Corrección de errores][iOS] Las leyendas y los subtítulos no se muestran en el reproductor nativo.
- [Corrección de errores][AMP] No se reproducen direcciones URL de streaming con el respaldo de CDN anexadas con tokens de autenticación.
- [Corrección de errores][FairPlay] Al código de error de Fairplay le falta el identificador de Tech (bits 31 a 28 del código de error); para más información, consulte los códigos de error.
- [Corrección de errores][Safari][PlayReady] El contenido de PlayReady en Safari genera un indicador giratorio infinito.

### <a name="changes-181"></a>Cambios en la versión 1.8.1 ###

- [Cambio][Html5] Se han cambiado los registros detallados nativos de la tecnología Html5 para que contengan eventos de VideoTag.

## <a name="180-official-update"></a>1.8.0 (actualización oficial) ##

### <a name="features-180"></a>Características de la versión 1.8.0 ###

- [Features][DRM] Se ha agregado compatibilidad con FairPlay (para más información, consulte el artículo sobre [contenido protegido](azure-media-player-protected-content.md)).

### <a name="bug-fixes-180"></a>Correcciones de errores de la versión 1.8.0 ###

- [Corrección de errores][AMP] La búsqueda del usuario no desencadena un evento de espera cuando la red está limitada.
- [Corrección de errores][FlashSS] La selección de calidad en la tecnología Flash produce una excepción.
- [Corrección de errores][AMP] La selección dinámica de la calidad se muestra en un menú contextual.
- [Corrección de errores][Máscara] Es difícil seleccionar el último elemento de menú de los menús contextuales.

### <a name="changes-180"></a>Cambios en la versión 1.8.0 ###

- [Cambio] El reproductor se ha actualizado a los requisitos de EME de Chrome actuales.
- [Cambio] El valor predeterminado de techOrder ha cambiado para acomodar la nueva tecnología, html5FairPlayHLS (para más información, consulte el artículo sobre [contenido protegido](azure-media-player-protected-content.md)).
- [Cambio][AzureHtml5JS] Se ha habilitado la reproducción de MPEG-Dash en Safari.
- [Cambio][Ejemplos] Se han cambiado los ejemplos con varios DRM para acomodar FairPlay.

## <a name="174-official-hotfix-update"></a>1.7.4 (actualización de revisión oficial) ##

### <a name="bug-fixes-174"></a>Correcciones de errores de la versión 1.7.4 ###

- [Corrección de errores][Chrome] Se muestra un contorno azul alrededor del controlador de búsqueda cuando el usuario tiene el contexto del reproductor.
- [Corrección de errores][IE9] Se iniciaba una excepción de JavaScript cuando el reproductor se cargaba en IE9.

## <a name="173-official-hotfix-update"></a>1.7.3 (actualización de revisión oficial) ##

### <a name="bug-fixes-173"></a>Correcciones de errores de la versión 1.7.3 ###

- [Corrección de errores][AzureHtml5JS] Se agotaba el tiempo de espera del reproductor en redes restringidas.

### <a name="changes-173"></a>Cambios en la versión 1.7.3 ###

- [Cambio] Se ha habilitado Webcrypto en Edge para descifrar contenido AES.
- [Cambio] Se ha optimizado la heurística de AMP para tener en cuenta los fragmentos almacenados en caché.
- [Cambio][AzureHtml5JS] Se ha optimizado la heurística mediante la reducción de la latencia de estimación del ancho de banda.

## <a name="172-official-hotfix-update"></a>1.7.2 (actualización de revisión oficial) ##

### <a name="features-172"></a>Características de la versión 1.7.2 ###
<!---API needs onboarding. Removed link to API until remedied.--->
- [Característica][AzureHtml5JS|Firefox] Se ha habilitado la reproducción de Widevine con EME para Firefox 47+.
- [Característica] Se ha agregado un evento para desechar el reproductor.
<!-- ([disposing](index.html#static-amp.eventname.disposing)) -->

### <a name="bug-fixes-172"></a>Correcciones de errores en la versión 1.7.2 ###

- [Corrección de errores] Los parámetros de consulta de URL de CDN de Akamai codificados no se descodificaban correctamente.
- [Corrección de errores] Se inicia una excepción en manifestPlayableWindowLength().
- [Corrección de errores] El usuario no siempre puede hacer clic en reproducir en el vídeo para volver a verlo una vez que este ha terminado.
- [Corrección de errores] El cambio dinámico de tamaño no se ajusta a los cambios de tamaño rápidos de una ventana.
- [Corrección de errores][Edge|IE] El cambio dinámico de tamaño no surte efecto al cargar la página con width=x, height=auto.
- [Corrección de errores][Android][Chrome] Chrome pide permiso para reproducir contenido DRM cuando el contenido no está cifrado.
- [Corrección de errores][Accesibilidad][Edge] Los controles de teclado no seleccionan correctamente los elementos del menú contextual.
- [Corrección de errores][Accesibilidad] El borde no se muestra en el modo de contraste alto.
- [Corrección de errores][FlashSS] No quitar el agente de escucha del evento Mouse Up después de desechar el reproductor provoca una excepción.
- [Corrección de errores][FlashSS] Se produce un problema al analizar la dirección URL del manifiesto con espacios codificados.
- [Corrección de errores][iOS] Error de tipo al evaluar tech.featuresVolumeControl.

### <a name="changes-172"></a>Cambios en la versión 1.7.2 ###

- [Cambio][DRM] Se han cambiado las comprobaciones de DRM después de establecer el origen a solo comprobación cuando el contenido está cifrado.
- [Cambio][AES] Se ha quitado el cuerpo no definido de tipo/sin formato de la solicitud de entrega de clave.
- [Cambio][Accesibilidad] El narrador de Windows ahora lee "Media Player" cuando el contexto está en el reproductor en lugar de en las propiedades.

## <a name="171-official-hotfix-update"></a>1.7.1 (actualización de revisión oficial) ##

### <a name="features-171"></a>Características de la versión 1.7.1 ###

- [Característica] Se ha agregado una opción para el perfil híbrido heurístico (este perfil se establece de forma predeterminada).

### <a name="bug-fixes-171"></a>Correcciones de errores en la versión 1.7.1 ###

- [Corrección de errores] El diseño dinámico no funciona según el estándar HTML5 (width=100%, height=auto).
- [Corrección de errores] Los valores porcentuales de ancho y alto no se comportan como se esperaba en la versión 1.7.0.

## <a name="170-official-update"></a>1.7.0 (actualización oficial) ##

### <a name="features-170"></a>Características de la versión 1.7.0 ###
<!---API needs onboarding. Removed link until remedied.--->
- [Característica][AzureHtml5JS][FlashSS] Se ha agregado currentMediaTime() para obtener el tiempo actual de los elementos multimedia del codificador en segundos.
- [Característica][FlashSS] Se han implementado las API de telemetría de descarga con videoBufferData() y audioBufferData().<!-- (see [BufferData](index.html#amp.bufferdata) for more details) -->
- [Característica][FlashSS] Se ha agregado el evento "downloadbitratechanged".
- [Característica] Se ha mejorado el tiempo de carga en comparación con las versiones anteriores del reproductor.
- [Característica] Los errores se registran en la consola de JavaScript.

### <a name="bug-fixes-170"></a>Correcciones de errores en la versión 1.7.0 ###

- [Corrección de errores] La dirección URL de póster codificada con parámetros de cadena de consulta no se muestra en el reproductor.
- [Corrección de errores] Se produce una excepción cuando no se carga ninguna tecnología y se llama a API amp.Player.poster().
- [Corrección de errores] Se produce una excepción cuando las funciones intentan acceder al reproductor después de desecharlo.
- [Corrección de errores][Accesibilidad] Falta el contorno en el foco, en el encabezado de búsqueda de la barra de progreso.
- [Corrección de errores][Accesibilidad] Los menús contextuales tienen una sombra en el modo de contraste alto.
- [Corrección de errores][iOS] No funciona la reproducción de subtítulos WebVTT del reproductor nativo.
- [Corrección de errores][AzureHtml5JS] Debe mostrarse el error 0x0100002 al reproducir una secuencia HTTP en un sitio HTTPS que genera un indicador giratorio infinito como resultado de un contenido mixto.
- [Corrección de errores][AzureHtml5JS] Falta el segmento final, lo que provoca que un error de comprobación del estado en bucle muestre lo que parece un estado de almacenamiento en búfer infinito.
- [Corrección de errores][AzureHtml5JS] Nombre de pista de audio incorrecto cuando se usan useManifestForLabel=false y códigos de idioma de tres letras.
- [Corrección de errores][AzureHtml5JS|Chrome] Estado de almacenamiento en búfer que se percibe como infinito al final del contenido causado por una imprecisión de número de punto flotante en la duración con JavaScript en Chrome.
- [Corrección de errores][FlashSS] Se muestra momentáneamente un error intermitente que no es grave al crear un reproductor Flash.
- [Corrección de errores][FlashSS] Se produce un error en la reproducción cuando las secuencias de vídeo y audio usan escalas de tiempo diferentes debido a que una imprecisión de redondeo provoca un error "Fragment url (...) is failed to generate FLVTags"[URL de fragmento (...) no genera FLVTags].
- [Corrección de errores][FlashSS] Problemas al analizar direcciones URL de manifiesto con espacios codificados.
- [Corrección de errores][FlashSS] Falta una comprobación para determinar si la versión del reproductor de Flash es 11.4 o anterior, lo que provoca un error en la reproducción en lugar de revertir a la siguiente tecnología de techOrder.
- [Corrección de errores][FlashSS][AES] Problemas al aceptar tokens de AES que tenga caracteres de subrayado.
- [Corrección de errores][SilverlightSS|OSX] Cuando se utiliza "//" como prefijo de un manifiesto, en lugar del protocolo (HTTP o HTTPS), se reconoce como un archivo local que genera un indicador giratorio infinito.

### <a name="changes-170"></a>Cambios en la versión 1.7.0 ###

- [Cambio][FlashSS] Se han combinado los scripts SWF (MSAdaptiveStreamingPlugin-osmf 2.0.swf" y "StrobeMediaPlayback. 2.0.swf") en un solo archivo SWF llamado "StrobeMediaPlayback.2.0.swf".
- [Cambio][FlashSS] Se ha actualizado la propagación de códigos de error para obtener códigos de error más precisos (por ejemplo, los errores 404 ahora producen 0x30200194, en lugar de un error genérico 0x30200000).

## <a name="163-official-hotfix-update"></a>1.6.3 (actualización de revisión oficial) ##

### <a name="bug-fixes-163"></a>Correcciones de errores en la versión 1.6.3 ###

- [Corrección de errores] Excepción en tiempo de ejecución de JavaScript cuando se ejecuta el controlador de eventos de las teclas de acceso rápido después de desechar el reproductor.
- [Corrección de errores][Android][AzureHtml5JS] No se realiza la reproducción en un dispositivo móvil cuando se utiliza la conexión de datos móviles.
- [Corrección de errores] Se ha actualizado Forge para que se ejecute como un trabajo web para liberar la interfaz de usuario.

## <a name="162-official-hotfix-update"></a>1.6.2 (actualización de revisión oficial) ##

### <a name="features-162"></a>Características de la versión 1.6.2 ###

- [Característica] Se han agregado idiomas adicionales para la localización (para más detalles, consulte la documentación).

### <a name="bug-fixes-162"></a>Correcciones de errores en la versión 1.6.2 ###

- [Corrección de errores][IE9-10] Al hacer clic en las zonas situadas alrededor del reproductor se minimizaba la ventana del explorador debido al error IE9/IE10 que se minimiza cuando window.blur().
- [Corrección de errores][FlashSS] No se aceptan tokens AES con guiones bajos.

## <a name="161-official-hotfix-update"></a>1.6.1 (actualización de revisión oficial) ##

### <a name="bug-fixes-161"></a>Correcciones de errores en la versión 1.6.1 ###

- [Corrección de errores][FlashSS|Edge,IE][SilverlightSS|IE] No es mover el foco a otros elementos de la interfaz de usuario para la entrada de datos u otras operaciones en IE/Edge.
- [Corrección de errores] Error de reproducción de AES cuando Forge no está definido.
- [Corrección de errores][Android][AzureHtml5JS|Chrome] El indicador giratorio continuo no reproduce contenido cuando esta en el bucle de comprobación del estado.
- [Corrección de errores][IE9] IE9 no admite console.log(), lo que causa una excepción.

## <a name="160-official-update"></a>1.6.0 (actualización oficial) ##

### <a name="features-160"></a>Características de la versión 1.6.0 ###

- [Característica] Se ha reducido un 33 % el tamaño de azuremediaplayer.min.js.
- [Característica][AzureHtml5JS|Edge][Sin probar] Se ha agregado compatibilidad con DD+secuencias de audio en Edge (no se puede cambiar de códecs después de la elección inicial). La aplicación debe seleccionar la secuencia de audio correcta en este momento.
- [Característica] Controles de las teclas de acceso rápido (para más información, consulte los documentos).
- [Característica] Se muestra información sobre el tiempo de progreso al pasar el cursor por encima para una búsqueda precisa del tiempo.
- [Característica] Se permite la detección asincrónica de complementos si el método setupDone existe en el complemento.

### <a name="bug-fixes-160"></a>Correcciones de errores en la versión 1.6.0 ###

- [Corrección de errores] El registro de memoria no se vacía cuando getMemoryLog(true)
- [Corrección de errores] El cuadro de selección de la velocidad de bits se restablece al mover el mouse, lo que causa que se seleccionen velocidades de bits menores mediante el control del mouse.
- [Corrección de errores] Office para Mac en la aplicación se bloquea al realizar la comprobación de DRM.
- [Corrección de errores] Las clases CSS se sobrescriben fácilmente por accidente.
- [Corrección de errores][Chrome] Se ha actualizado la identificación de la cadena user-agent, el explorador es Edge.
- [Corrección de errores][AzureHtml5JS] El botón de título no se muestra en la barra de herramientas en Edge(Win10) o Chrome(Mac).
- [Corrección de errores][Android][AzureHtml5JS|Chrome] Excepción InvalidStateError en la llamada endOfStream() en vídeos cortos.
- [Corrección de errores][Firefox] Se ha quitado la advertencia de DRM causada por Firefox al comprobar las funcionalidades del explorador.
- [Corrección de errores][Html5] Los subtítulos o las leyendas no se muestran con contenido mp4 progresivo.
- [Corrección de errores][FlashSS] Los mensajes con marcas de tiempo coincidentes se registraban en orden inverso.
- [Corrección de errores][Accesibilidad][Chrome|Firefox] El tabulador y los controles de selección seleccionan automáticamente el primer elemento del menú.
- [Corrección de errores][Accesibilidad] Uso del tabulador para controlar el botón de volumen.

### <a name="changes-160"></a>Cambios en la versión 1.6.0 ###

- [Cambio] Uso del tiempo del descifrado de AES al seleccionar el nivel de calidad.
- [Cambio] Actualización del sistema de reescritura de direcciones URL para que use la versión 4 de HLS antes que la versión 3 de HLS para secuencias con varios audios.
- [Cambio] Se ha establecido nativeControlsForTouch en false como valor predeterminado (para que funcione correctamente, debe ser false).

## <a name="150-official-update"></a>1.5.0 (actualización oficial) ##

### <a name="features-150"></a>Características de la versión 1.5.0 ###

- [Característica] Mejoras de la seguridad web general (prevención de inyección, XSS, etc.).
- [Característica] La integración del complemento SDN enlaza el evento sourceset y options.sdn.
- [Característica] Se ha mejorado el control de errores 5XX y 4XX durante la reproducción.

### <a name="bug-fixes-150"></a>Correcciones de errores en la versión 1.5.0 ###

- [Corrección de errores] Se ha actualizado la minificación de CSS para usar códigos fuente de entidad HTML para los botones, en lugar de Unicode.
- [Corrección de errores][AzureHtml5JS] El contenido con varios DRM siempre selecciona el token del primer elemento en protectionInfo, lo que causa un error en el segundo DRM.
- [Corrección de errores][AzureHtml5JS] La búsqueda nunca se completa al buscar en un área en la que faltan segmentos.
- [Corrección de errores][AzureHtml5JS|Edge] Se han agregado compatibilidad con EME con prefijos en la actualización de Edge para la reproducción de PlayReady.
- [Corrección de errores][AzureHtml5JS|Firefox] Se ha actualizado la comprobación de EME para permitir que, a partir de la versión 42 de Firefox (con MSE), use Silverlight como reserva para el contenido protegido.
- [Corrección de errores][FlashSS] Se ha actualizado error.message de número a cadena detallada.

### <a name="changes-150"></a>Cambios en la versión 1.5.0 ###

- [Cambio] Actualmente, los pósteres solo funcionan como direcciones URL absolutas.

## <a name="140-official-update"></a>1.4.0 (actualización oficial) ##

### <a name="features-140"></a>Características de la versión 1.4.0 ###

- [Característica][AzureHtml5JS|Chrome] Se ha agregado compatibilidad simple con DRM de Widevine.
- [Característica][AzureHtml5JS] Se ha mejorado el control de los errores 404 o 412 durante la reproducción.

### <a name="bug-fixes-140"></a>Correcciones de errores en la versión 1.4.0 ###

- [Corrección de errores][FlashSS] Se ha mejorado la validación de parámetros.

## <a name="130-official-update"></a>1.3.0 (actualización oficial) ##

### <a name="features-130"></a>Características de la versión 1.3.0 ###

- [Característica][AzureHtml5JS][FlashSS] Conmutación de audio del contenido con audio múltiple del mismo códec.

### <a name="bug-fixes-130"></a>Correcciones de errores en la versión 1.3.0 ###

- [Corrección de errores][AzureHtml5JS|Chrome] Indicador rotatorio infinito intermitente.
- [Corrección de errores][AzureHtml5JS|IE][Windows Phone] Excepción que hace que Windows Phone tenga problemas de reproducción.
- [Corrección de errores][FlashSS] Si la opción de reproducción automática se establece en false, se producen errores en las instancias adicionales.
- [Corrección de errores] Problemas de cambio de tamaño de los menús de la interfaz de usuario.

## <a name="120-official-update"></a>1.2.0 (actualización oficial) ##

### <a name="features-120"></a>Características de la versión 1.2.0 ###

- [Característica][AzureHtml5JS|Firefox] Se ha agregado compatibilidad cuando MSE está habilitado.
- [Característica] Ya no se necesita que la aplicación para proporcione rutas de acceso para los archivos binarios de reserva de la tecnología (swf y xap). La ruta de acceso es relativa al script de Azure Media Player.

### <a name="bug-fixes-120"></a>Correcciones de errores en la versión 1.2.0 ###

- [Corrección de errores][AzureHtml5JS|Chrome] El reproductor va por detrás del extremo en vivo cuando está en segundo plano.
- [Corrección de errores][AzureHtml5JS|Edge] No funciona el modo de pantalla completa.
- [Corrección de errores][AzureHtml5JS] El registro no se habilitaba correctamente cuando se establecía en las opciones.
- [Corrección de errores][Flash] Tanto el "almacenamiento en búfer" como el icono del almacenamiento en búfer se muestran durante el evento de espera.
- [Corrección de errores] Se permite que la reproducción continúe si se produce un error en la solicitud de ancho de banda inicial.
- [Corrección de errores] El reproductor no se carga cuando se inicializa con opciones sin definir.
- [Corrección de errores] Si se intenta desechar el reproductor cuando ya está desechado, se produce una excepción de vdata.
- [Corrección de errores] Los iconos de la barra de calidad se asignaron de forma incorrecta.

## <a name="111-official-hotfix-update"></a>1.1.1 (actualización de revisión oficial) ##

### <a name="bug-fixes-111"></a>Correcciones de errores en la versión 1.1.1 ###

- [Corrección de errores] Problema antiguo de pantalla completa en IE.
- [Corrección de errores] Los complementos ya no se sobrescriben.

## <a name="110-official-update"></a>1.1.0 (actualización oficial) ##

### <a name="features-110"></a>Características de la versión 1.1.0 ###

- [Característica] Se han actualizado las cadenas de localización de la interfaz de usuario.

### <a name="bug-fixes-110"></a>Correcciones de errores en la versión 1.1.0 ###

- [Corrección de errores] El botón de reproducción grande no tiene suficiente contraste.
- [Corrección de errores] Indicador visual del foco en la pestaña.
- [Corrección de errores] El menú de selección de velocidad de bits ahora incluye la información de resolución correcta.
- [Corrección de errores] El ajuste de tamaño del menú More options (Más opciones) ahora se realiza dinámicamente.
- [Corrección de errores] Varios problemas de la interfaz de usuario.

## <a name="100-official-release"></a>1.0.0 (versión oficial) ##

### <a name="features-100"></a>Características de la versión 1.0.0 ###

- [Característica] Pruebas de accesibilidad básicas para el control del tabulador, el control del foco, el lector de pantalla y la interfaz de usuario de alto contraste.
- [Característica] Interfaz de usuario actualizada.
- [Característica] Registro de desarrollo.
- [Característica] API para establecer dinámicamente las pistas de subtítulos y leyendas.
- [Característica] Características de localización básicas.
- [Característica] Consolidación de códigos de error entre tecnologías.
- [Característica] Nuevo código de error si los complementos (como Flash o Silverlight) no están instalados.
- [Característica][AzureHtml5JS] Se han implementado eventos de diagnóstico básicos.

### <a name="bug-fixes-100"></a>Correcciones de errores en la versión 1.0.0 ###
<!---What is that actually supposed to say?--->
- [Corrección de errores][AzureHtml5JS] La reproducción en vivo se inmoviliza en las actualizaciones de MPD cuando hay pequeñas imprecisiones en la marca de tiempo.
- [Corrección de errores][AzureHtml5JS] Se han mitigado varios problemas de la reproducción en vivo.
- [Corrección de errores][AzureHtml5JS] Los búferes se vacían cuando la heurística del tamaño de la ventana está activa y se va a una pantalla con mayor resolución.
- [Corrección de errores][AzureHtml5JS] Chrome ahora muestra correctamente el evento finalizado. Vinculado a un problema conocido anterior de *Chrome por el que no se enviaba correctamente un evento terminado cuando se usa AzureHtml5JS. Hay un problema en el explorador subyacente.*
- [Corrección de errores][AzureHtml5JS] Se ha deshabilitado Safari para esta tecnología, con el fin de solucionar un *problema de reproducción con OSX Yosemite con la tecnología AzureHtml5JS. Hay problemas de implementación de MSE. Mitigación temporal: forzar flashSS, silverlightSS como orden de las tecnologías para estos agentes de usuario*
- [Corrección de errores][FlashSS] Se activaba loadstart después de que se produjera un error.

## <a name="020-beta"></a>0.2.0 (Beta) ##

### <a name="features-020"></a>Características de la versión 0.2.0 ###

- [Característica] Pruebas completadas de PlayReady y AES tanto para la versión en vivo como la versión a petición (vea la matriz de compatibilidad).
- [Característica] Control de discontinuidades.
- [Característica] Se ha agregado compatibilidad con marcas de tiempo mayores que 2^53.
- [Característica] El parámetro de consulta de URL persiste en la solicitud del manifiesto.
- [Característica][Sin probar] Se ha agregado compatibilidad con los perfiles heurísticos `QuickStart` y `HighQuality`.
- [Característica][Sin probar] Se expone la información relativa a secuencias de vídeo sobre velocidades de bits, ancho y alto en AzureHtml5JS y FlashSS.
- [Característica][Sin probar] Selección de la velocidad de bits en AzureHtml5JS y FlashSS (consulte la documentación de la API).

### <a name="bug-fixes-020"></a>Correcciones de errores en la versión 0.2.0 ###

- [Corrección de errores] El botón de reproducción grande ya se ve en WP8.1.
- [Corrección de errores] Se han reparado varios problemas de reproducción en vivo.
- [Corrección de errores] El botón para reactivar el audio ahora funciona en la interfaz de usuario.
- [Corrección de errores] Se ha actualizado la experiencia de carga de la interfaz de usuario en el modo de reproducción automática.
- [Corrección de errores] Problema con el cargador de AMD y conflictos del método de definición.
- [Corrección de errores] Problema de carga de la aplicación Cordova en WP 8.1.
- [Corrección de errores] La plataforma o la tecnología de consultas de contenido protegidas admitían ProtectionType para seleccionar la tecnología adecuada para la reproducción.  Corrige un problema ya conocido de "_El contenido de PlayReady en Chrome (escritorio)/Safari 8 (en OSX Yosemite) no utiliza el reproductor Silverlight como reserva_".
- [Corrección de errores] Excepción no detectada en WinServer 2012 R2 debido a que Media Foundation no está instalado en esa máquina de forma predeterminada.  Al intentar usar de las API de etiquetas de vídeo HTML que no están implementadas, se produce un error. La mitigación actual es detectar el error y devolver false, en lugar de producir el error.
- [Corrección de errores] Se obtiene siempre el segmento inicial después del error de búsqueda o de http para evitar problemas durante la reproducción.
- [Corrección de errores] Se desactiva el proceso simulado del seguimiento y las actualizaciones del tiempo en caso de error.
- [Corrección de errores] Se ha eliminado el menú contextual.
- [Corrección de errores][AzureHtml5JS] El mensaje de error no se muestra cuando se ha establecido un token no válido para el contenido de PlayReady.
- [Corrección de errores][AzureHtml5JS] Si se pasaba al modo de pantalla completa durante la reproducción en vivo, no se tenía en cuenta la heurística del tamaño de la ventana.
- [Corrección de errores][FlashSS] Se han eliminado los mensajes que mostraba Strobe Media Player y solo se muestran los mensajes de Azure Media Player.
- [Corrección de errores][SilverlightSS] No se obtiene el evento "seeked" cuando se busca en un tiempo mayor que la duración o menor que 0.

## <a name="010-beta-release"></a>0.1.0 (versión beta) ##

Versión preliminar inicial

## <a name="next-steps"></a>Pasos siguientes ##

- [Inicio rápido de Azure Media Player](azure-media-player-quickstart.md)
