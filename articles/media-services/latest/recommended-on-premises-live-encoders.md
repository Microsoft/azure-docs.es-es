---
title: Codificadores de streaming en vivo recomendados por Media Services en Azure | Microsoft Docs
description: Información sobre los codificadores locales de streaming en vivo recomendados por Media Services
services: media-services
keywords: encoding;encoders;media
author: johndeu
manager: johndeu
ms.author: johndeu
ms.date: 01/17/2019
ms.topic: article
ms.service: media-services
ms.openlocfilehash: 50b22cefccf620d7b79202a5c432e2e6a4e3e3be
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65550050"
---
# <a name="recommended-live-streaming-encoders"></a>Codificadores de streaming en vivo recomendados

En Azure Media Services, un [evento en directo](https://docs.microsoft.com/rest/api/media/liveevents) (canal) representa una canalización para procesar contenido de streaming en vivo. El evento en directo recibe las secuencias de entrada en vivo de una de estas dos formas:

* Un codificador en directo local envía una secuencia RTMP o Smooth Streaming (MP4 fragmentado) de velocidad de bits múltiple al canal Evento en directo que no está habilitado para realizar la codificación en vivo con Media Services. Las secuencias recopiladas pasan a través de Eventos en directo sin más procesamiento. Este método se llama **paso a través**. Un codificador en directo puede enviar una transmisión de velocidad de bits única a un canal de tránsito. No se recomienda esta configuración, ya que no permite al cliente utilizar un streaming con velocidad de bits adaptable.

  > [!NOTE]
  > El método de paso a través es la forma más económica de realizar un streaming en vivo.

* Un codificador en directo local envía una secuencia de velocidad de bits única al Evento en directo que está habilitado para realizar la codificación en vivo con Media Services, con uno de los siguientes formatos: RTMP o Smooth Streaming (MP4 fragmentado). Después, el Evento en directo codifica en vivo la secuencia entrante de velocidad de bits única en una secuencia de vídeo de velocidad de bits múltiple (adaptable).

Para obtener información detallada sobre la codificación en vivo con Media Services, vea [Streaming en vivo con Azure Media Services v3](live-streaming-overview.md).

## <a name="live-encoders-that-output-rtmp"></a>Codificadores en directo que generan una salida RTMP

Media Services recomienda usar uno de los siguientes codificadores en directo que tienen RTMP como salida. Los esquemas URL admitidos son `rtmp://` o `rtmps://`.

> [!NOTE]
> Al hacer el streaming mediante RTMP, compruebe la configuración del firewall o del proxy para confirmar que los puertos TCP salientes 1935 y 1936 están abiertos.

- Adobe Flash Media Live Encoder 3.2
- Haivision KB
- Haivision Makito X HEVC
- OBS Studio
- Switcher Studio (iOS)
- Telestream Wirecast 8.1+
- Telestream Wirecast S
- Teradek Slice 756
- TriCaster 8000
- Tricaster Mini HD-4
- VMIX
- xStream

## <a name="live-encoders-that-output-fragmented-mp4"></a>Codificadores en directo que generan una salida de MP4 fragmentado

Media Services recomienda usar uno de los codificadores en directo siguientes que tienen Smooth Streaming de velocidad de bits múltiple (MP4 fragmentado) como salida. Los esquemas URL admitidos son `http://` o `https://`.

- Ateme TITAN Live
- Cisco Digital Media Encoder 2200
- Elemental Live
- Envivio 4Caster C4 Gen III
- Imagine Communications Selenio MCP3
- Media Excel Hero Live y Hero 4K (UHD/HEVC)

> [!TIP]
>  Si está realizando streaming de eventos en directo en varios idiomas (por ejemplo, una pista de audio inglés y una pista de audio español), puede hacerlo con el codificador en directo de Media Excel configurado para enviar la fuente directa a un evento en directo paso a través.

## <a name="configuring-on-premises-live-encoder-settings"></a>Configuración de los valores del codificador en directo local

Para información acerca de qué valores son válidos para el tipo de evento en directo, consulte [Comparación de tipos de objetos LiveEvent](live-event-types-comparison.md).

### <a name="playback-requirements"></a>Requisitos de reproducción

Para volver a reproducir un contenido, debe haber una secuencia de audio y una secuencia de vídeo. No se admite la reproducción de la secuencia solo de vídeo.

### <a name="configuration-tips"></a>Sugerencias de configuración

- Siempre que sea posible, use una conexión a Internet por cable.
- Cuando tenga que determinar los requisitos de ancho de banda, duplique las velocidades de bits de streaming. Aunque no es obligatorio, esta sencilla regla ayuda a mitigar el impacto de la congestión de la red.
- Cuando se usen codificadores por software, cierre todos los programas innecesarios.
- Si se modifica la configuración del codificador una vez iniciada la inserción, se producirán efectos negativos en el evento. Los cambios de configuración pueden provocar que el evento se vuelva inestable. 
- Asegúrese de dejar tiempo suficiente para configurar el evento. En el caso de los eventos a gran escala, se recomienda iniciar la configuración una hora antes del evento.

## <a name="becoming-an-on-premises-encoder-partner"></a>Convertirse en un asociado de codificador local

Como asociado de codificador local de Azure Media Services, Media Services promueve su producto al recomendar su codificador a clientes empresariales. Para convertirse en un asociado de codificador local, debe comprobar la compatibilidad del codificador local con Media Services. Para ello, complete las comprobaciones siguientes:

### <a name="pass-through-live-event-verification"></a>Verificación del Evento en directo con paso a través

1. En la cuenta de Media Services, asegúrese de que el **Punto de conexión de streaming** esté en ejecución. 
2. Cree e inicie el Evento en directo de **paso a través**. <br/> Para más información, consulte [Estados y facturación de LiveEvent](live-event-states-billing.md).
3. Obtenga las direcciones URL de ingesta y configure el codificador local de forma que utilice la dirección URL para enviar una secuencia en directo de velocidad de bits múltiple a Media Services.
4. Obtenga la dirección URL de versión preliminar y úsela para verificar que la entrada del codificador se está recibiendo realmente.
5. Cree un nuevo objeto de **recurso**.
6. Cree un objeto de **salida en directo** y use el nombre del recurso que ha creado.
7. Cree un objeto **Streaming Locator** con los tipos del objeto **Streaming Policy** integrados.
8. Enumere las rutas de acceso en el **localizador de streaming** para recuperar las direcciones URL que se van a usar.
9. Obtenga el nombre de host del **punto de conexión de streaming** desde el que quiere hacer el streaming.
10. Combine la dirección URL del paso 8 con el nombre de host del paso 9 para obtener la dirección URL completa.
11. Ejecute el codificador en directo durante aproximadamente diez minutos.
12. Detenga el objeto LiveEvent. 
13. Use un reproductor, como [Azure Media Player](https://ampdemo.azureedge.net/azuremediaplayer.html), para ver el recurso archivado y asegurarse de que la reproducción no tenga problemas visibles. También, puede ver el recurso y validarlo utilizando la dirección URL de vista previa durante la sesión activa.
14. Registre el identificador del recurso, la dirección URL de streaming publicada en el archivo dinámico y la configuración y la versión que utilizó en el codificador en directo.
15. Restablezca el estado del Evento en directo después de crear cada ejemplo.
16. Repita los pasos 5 a 15 con todas las configuraciones admitidas por el codificador (con y sin señalización, subtítulos o velocidades de codificación diferentes).

### <a name="live-encoding-live-event-verification"></a>Verificación del Evento en directo con codificación en directo

1. En la cuenta de Media Services, asegúrese de que el **Punto de conexión de streaming** esté en ejecución. 
2. Cree e inicie el Evento en directo de **codificación en directo**. <br/> Para más información, consulte [Estados y facturación de LiveEvent](live-event-states-billing.md).
3. Obtenga las direcciones URL de ingesta y configure el codificador para insertar una secuencia en directo de velocidad de bits única en Media Services.
4. Obtenga la dirección URL de versión preliminar y úsela para verificar que la entrada del codificador se está recibiendo realmente.
5. Cree un nuevo objeto de **recurso**.
6. Cree un objeto de **salida en directo** y use el nombre del recurso que ha creado.
7. Cree un objeto **Streaming Locator** con los tipos del objeto **Streaming Policy** integrados.
8. Enumere las rutas de acceso en el **localizador de streaming** para recuperar las direcciones URL que se van a usar.
9. Obtenga el nombre de host del **punto de conexión de streaming** desde el que quiere hacer el streaming.
10. Combine la dirección URL del paso 8 con el nombre de host del paso 9 para obtener la dirección URL completa.
11. Ejecute el codificador en directo durante aproximadamente diez minutos.
12. Detenga el objeto LiveEvent.
13. Use un reproductor, como [Azure Media Player](https://ampdemo.azureedge.net/azuremediaplayer.html), para ver el recurso archivado y asegurarse de que la reproducción no tenga problemas visibles en todos los niveles de calidad. También, puede ver el recurso y validarlo utilizando la dirección URL de vista previa durante la sesión activa.
14. Registre el identificador del recurso, la dirección URL de streaming publicada en el archivo dinámico y la configuración y la versión que utilizó en el codificador en directo.
15. Restablezca el estado del Evento en directo después de crear cada ejemplo.
16. Repita los pasos 5 a 15 con todas las configuraciones admitidas por el codificador (con y sin señalización, subtítulos o velocidades de codificación diferentes).

### <a name="longevity-verification"></a>Comprobación de duración

Siga los mismos pasos que se indican en [Verificación del evento en directo con paso a través](#pass-through-live-event-verification), salvo el paso 11. <br/>En lugar de 10 minutos, ejecute el codificador en directo durante una semana o más. Use un reproductor, como [Azure Media Player](https://ampdemo.azureedge.net/azuremediaplayer.html), para ver el streaming en vivo cada cierto tiempo (o un recurso archivado) y asegurarse de que la reproducción no tenga problemas visibles.

### <a name="email-your-recorded-settings"></a>Envío de la configuración anotada por correo electrónico

Por último, envíe la configuración registrada y los parámetros archivados por correo electrónico a Azure Media Services (amsstreaming@microsoft.com) para notificar que se han superado todos los pasos de la comprobación automática. Incluya también sus datos de contacto para que pueda recibir las actualizaciones que haya. Puede ponerse en contacto con el equipo de Azure Media Services si tiene alguna pregunta sobre este proceso.

## <a name="next-steps"></a>Pasos siguientes

[Streaming en vivo con Media Services v3](live-streaming-overview.md)
