---
title: 'Streaming en vivo con codificadores locales que crean transmisiones de velocidad de bits múltiple: Azure | Microsoft Docs'
description: En este tema se describe cómo configurar un canal que recibe streaming en vivo de velocidad de bits múltiple desde un codificador local.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: d9f0912d-39ec-4c9c-817b-e5d9fcf1f7ea
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.openlocfilehash: 746fe9132dcb06678e2a0a975c8eed0aba6c3fad
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269631"
---
# <a name="working-with-channels-that-receive-multi-bitrate-live-stream-from-on-premises-encoders"></a>Uso de canales que reciben streaming en vivo con velocidad de bits múltiple de codificadores locales

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> A partir del 12 de mayo de 2018, los canales en directo ya no admitirán el protocolo de ingesta de secuencia de transporte RTP/MPEG-2. Migre de RTP/MPEG-2 a RTMP o protocolos de ingeesta de MP4 fragmentado (Smooth Streaming).

## <a name="overview"></a>Información general
En Azure Media Services, un *canal* representa una canalización para procesar contenido de streaming en vivo. Los canales reciben la transmisión de entrada en vivo de dos maneras posibles:

* Un codificador local en vivo envía una transmisión RTMP o Smooth Streaming (MP4 fragmentado) de velocidad de bits múltiple al canal que no está habilitado para realizar la codificación en vivo con Media Services. Las transmisiones ingeridas pasan a través de canales sin más procesamiento. Este método se llama *paso a través*. El codificador en vivo también envía una transmisión de una sola velocidad de bits a un canal que no está habilitado para la codificación en vivo, pero esto no es recomendable. Media Services entrega la transmisión a los clientes que lo soliciten.

  > [!NOTE]
  > El método de paso a través es la forma más económica de realizar un streaming en vivo.


* Un codificador en directo local envía una secuencia de una sola velocidad de bits al canal que está habilitado para realizar codificación en directo con Media Services, con uno de los siguientes formatos: RTMP o Smooth Streaming (MP4 fragmentado). Después, el canal codifica en vivo la transmisión entrante de una sola velocidad de bits en una de vídeo de velocidad de bits múltiple (adaptable). Media Services entrega la transmisión a los clientes que lo soliciten.

A partir de la versión 2.10 de Media Services, al crear un canal puede especificar cómo desea que este reciba la transmisión entrante. También puede especificar si desea que el canal realice la codificación en vivo de la transmisión. Tiene dos opciones:

* **Pass Through**: especifique este valor si piensa usar un codificador en directo local que genere una transmisión de velocidad de bits múltiple (transmisión de acceso directo) de salida. En este caso, la transmisión entrante pasa hasta la salida sin codificación. Este es el comportamiento de los canales antes de la versión 2.10. En este artículo se proporciona información sobre cómo trabajar con canales de este tipo.
* **Live Encoding**: elija este valor si piensa usar Media Services para codificar transmisiones en vivo con una sola velocidad de bits como transmisión de velocidad de bits múltiple. Dejar un canal con codificación en directo en estado **En ejecución** supone cargos de facturación. Se recomienda detener inmediatamente los canales en ejecución después que termine evento de streaming en vivo para evitar cargos por hora adicionales. Media Services entrega la transmisión a los clientes que lo soliciten.

> [!NOTE]
> En este artículo se describen los atributos de los canales no habilitados para realizar la codificación en directo. Para más información sobre cómo trabajar con los canales habilitados para Live Encoding, consulte [Live Encoding con Azure Media Services para crear velocidades de bits múltiple](media-services-manage-live-encoder-enabled-channels.md).
>
>Para obtener información sobre los codificadores locales recomendados, consulte [Codificadores locales recomendados](media-services-recommended-encoders.md).

El siguiente diagrama representa un flujo de trabajo de streaming en vivo que usa un codificador en vivo local para generar transmisiones de RTMP o MP4 fragmentado de velocidad de bits múltiple (Smooth Streaming) de salida.

![Flujo de trabajo activo][live-overview]

## <a name="common-live-streaming-scenario"></a><a id="scenario"></a>Escenario típico de streaming en vivo
En los pasos siguientes se describen las tareas que intervienen en la creación de aplicaciones típicas de streaming en vivo.

1. Conecte una cámara de vídeo a un equipo. Inicie y configure un codificador en vivo local que genera una transmisión de RTMP o MP4 fragmentado de velocidad de bits múltiple (Smooth Streaming) de salida. Para obtener más información, consulte [Compatibilidad con RTMP de Azure Media Services y codificadores en directo](https://go.microsoft.com/fwlink/?LinkId=532824).

    También puede realizar este paso después de crear el canal.
2. Cree e inicie un canal.

3. Recupere la URL de ingesta de canales.

    El codificador en vivo usa la URL de ingesta para enviar la transmisión al canal.
4. Recupere la URL de vista previa de canal.

    Use esta dirección URL para comprobar que el canal recibe correctamente la secuencia en vivo.
5. Cree un programa.

    Cuando se usa Azure Portal, al crear un programa también se crea un recurso.

    Cuando use el SDK de .NET o REST, debe crear un recurso y especificar que este se use al crear un programa.
6. Publique el recurso asociado al programa.   

    >[!NOTE]
    >Cuando se crea la cuenta de Azure Media Services, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Stopped** (Detenido). El punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado **Running** (En ejecución).

7. Inicie el programa cuando esté listo para iniciar el streaming y el archivo.

8. Si lo desea, puede señalar el codificador en directo para iniciar un anuncio. El anuncio se inserta en el flujo de salida.

9. Detenga el programa cuando quiera detener el streaming y el archivo del evento.

10. Elimine el programa (y, opcionalmente, también el recurso).     

## <a name="description-of-a-channel-and-its-related-components"></a><a id="channel"></a>Descripción de un canal y sus componentes relacionados
### <a name="channel-input-ingest-configurations"></a><a id="channel_input"></a>Configuraciones de entrada de canal (introducción)
#### <a name="ingest-streaming-protocol"></a><a id="ingest_protocols"></a>Protocolo de streaming de ingesta
Media Services admite la ingesta de fuentes en vivo mediante MP4 fragmentado y RTMP de velocidad de bits múltiple como protocolos de streaming. Al seleccionar el protocolo de streaming de ingesta RTMP, se crean dos puntos de conexión de ingesta (entrada) para el canal:

* **URL principal**: especifica la dirección URL completa del extremo de introducción RTMP principal del canal.
* **URL secundaria** (opcional): especifica la dirección URL completa del extremo de introducción RTMP secundario del canal.

Use la dirección URL secundaria si quiere mejorar la durabilidad y la tolerancia a errores de la transmisión de ingesta, así como la conmutación por error y la tolerancia a errores del codificador, especialmente en los siguientes escenarios:

- Codificador único que realiza inserciones dobles en las URL principal y secundaria:

    El objetivo principal de este escenario es proporcionar más resistencia a las fluctuaciones de la red y a las vibraciones. Algunos codificadores RTMP no controlan bien las desconexiones de red. Cuando se produce una desconexión de red, un codificador podría detener la codificación y no enviar los datos almacenados en búfer cuando vuelva a conectarse. Esto provoca discontinuidades y pérdida de datos. Las desconexiones de red pueden suceder por una red en mal estado o un mantenimiento inadecuado por parte de Azure. Las URL principal y secundaria reducen los problemas de red y proporcionan un proceso de actualización controlado. Cada vez que se produce una desconexión de red programada, Media Services administra la desconexión principal y secundaria, y proporciona un retardo entre las dos. Así los codificadores tienen tiempo para seguir enviando los datos y volver a conectarse. El orden de las desconexiones puede ser aleatorio, pero siempre habrá un retardo entre la URL principal y la secundaria, o viceversa. En este escenario, el codificador sigue siendo el único punto de error.

- Varios codificadores, cada uno con inserción en un punto específico:

    Este escenario proporciona redundancia tanto para el codificador como para la ingesta. En este escenario, encoder1 inserta en la dirección URL principal y encoder2 inserta en la dirección URL secundaria. Cuando se produce un error en un codificador, el otro puede seguir enviando datos. Se puede mantener la redundancia de datos porque Media Services no desconecta la URL principal y secundaria al mismo tiempo. En este escenario se da por hecho que los codificadores tienen sincronización temporal y proporcionan exactamente los mismos datos.  

- Varios codificadores con doble inserción en las URL principal y secundaria:

    En este escenario, ambos codificadores insertan datos en las URL principal y secundaria. Esto proporciona la mejor confiabilidad y tolerancia a errores, así como redundancia de datos. Este escenario puede tolerar errores y desconexiones en ambos codificadores, aunque un codificador deje de funcionar. Se da por hecho que los codificadores tienen sincronización temporal y proporcionan exactamente los mismos datos.  

Para obtener información sobre los codificadores en directo de RTMP, consulte [Compatibilidad con RTMP de Azure Media Services y codificadores en directo](https://go.microsoft.com/fwlink/?LinkId=532824).

#### <a name="ingest-urls-endpoints"></a>Direcciones URL de ingesta (extremos)
Un canal proporciona un punto de conexión de entrada (dirección URL de ingesta) que usted especifica en el codificador en vivo, de modo que este pueda insertar transmisiones en los canales.   

Puede obtener las direcciones URL de ingesta al crear el canal. Para obtener estas direcciones URL, el estado del canal no puede ser **En ejecución**. Cuando esté listo para comenzar a insertar datos en el canal, su estado debe ser **En ejecución**. Una vez que el canal empieza a ingerir datos, puede obtener una vista previa de la transmisión a través de la dirección URL de vista previa.

Tiene la opción de ingerir una transmisión en vivo de MP4 fragmentado (Smooth Streaming) sobre una conexión TLS. Para ingerir en TLS, asegúrese de actualizar la dirección URL de ingesta a HTTPS. Actualmente, no se puede ingerir RTMP sobre TLS.

#### <a name="keyframe-interval"></a><a id="keyframe_interval"></a>Intervalo de fotogramas clave
Cuando se usa un codificador en vivo local para generar una transmisión de velocidad de bits múltiple, el intervalo de fotogramas clave especifica la duración de grupo de imágenes (GOP) tal como la usa el codificador externo. Una vez que el canal recibe esta transmisión de entrada, puede entregar su transmisión en vivo a las aplicaciones de reproducción del cliente en cualquiera de estos formatos: Smooth Streaming, Streaming adaptable dinámico a través de HTTP (DASH) y HTTP Live Streaming (HLS). Cuando se realiza el streaming en vivo, HLS siempre se empaqueta dinámicamente. De forma predeterminada, Media Services calcula automáticamente la proporción de empaquetado por segmento HLS (fragmentos por segmento) según el intervalo de fotogramas clave que se recibe del codificador en vivo.

En la tabla siguiente se muestra cómo se calcula la duración de los segmentos:

| Intervalo de fotogramas clave | Relación de empaquetado por segmento HLS (fragmento por segmento) | Ejemplo |
| --- | --- | --- |
| Menor o igual que 3 segundos |3:1 |Si KeyFrameInterval (o GOP) es de 2 segundos, la proporción de empaquetado de segmento HLS predeterminada es de 3 a 1. Esto crea un segmento HLS de 6 segundos. |
| de 3 a 5 segundos |2:1 |Si KeyFrameInterval (o GOP) es de 4 segundos, la proporción de empaquetado de segmento HLS predeterminada es de 2 a 1. Esto crea un segmento HLS de 8 segundos. |
| Mayor que 5 segundos |1:1 |Si KeyFrameInterval (o GOP) es de 6 segundos, la proporción de empaquetado de segmento HLS predeterminada es de 1 a 1. Esto crea un segmento HLS de 6 segundos. |

Puede cambiar la proporción de fragmentos por segmento al configurar la salida del canal y establecer FragmentsPerSegment en ChannelOutputHls.

También puede cambiar el valor del intervalo de fotogramas clave si establece la propiedad KeyFrameInterval en ChannelInput. Si establece explícitamente KeyFrameInterval, la proporción de empaquetado por segmento HLS FragmentsPerSegment se calcula según las reglas descritas anteriormente.  

Si establece explícitamente KeyFrameInterval y FragmentsPerSegment, Media Services usa los valores que usted establezca.

#### <a name="allowed-ip-addresses"></a>Direcciones IP permitidas
Puede definir las direcciones IP permitidas para publicar vídeo en el canal. Puede especificarse una dirección IP permitida como:

* Una única dirección IP, por ejemplo, 10.0.0.1.
* Un intervalo de direcciones IP que use una dirección IP y una máscara de subred CIDR, por ejemplo, 10.0.0.1/22.
* Un intervalo de direcciones IP que use una dirección IP y una máscara de subred decimal con puntos, por ejemplo, 10.0.0.1(255.255.252.0).

Si no se especifica ninguna dirección IP y no hay ninguna definición de regla, no se permite ninguna dirección IP. Para permitir las direcciones IP, cree una regla y establezca 0.0.0.0/0.

### <a name="channel-preview"></a>Vista previa de canal
#### <a name="preview-urls"></a>Direcciones URL de vista previa
Los canales proporcionan un extremo de vista previa (dirección URL de vista previa) que se puede utilizar para obtener una vista previa y validar la secuencia antes de mayor procesamiento y entrega.

Puede obtener la dirección URL de vista previa al crear el canal. Para ello, el estado del canal no puede ser **En ejecución**. Cuando el canal haya empezado a ingerir datos, puede obtener una vista previa de la transmisión.

Actualmente, la transmisión de vista previa solo se puede entregar en formato MP4 fragmentado (Smooth Streaming), independientemente del tipo de entrada especificado. Puede usar el reproductor [Smooth Streaming Health Monitor](https://playready.directtaps.net/smoothstreaming/) para probar el formato Smooth Stream. También puede usar un reproductor hospedado en Azure Portal para ver la transmisión.

#### <a name="allowed-ip-addresses"></a>Direcciones IP permitidas
Puede definir las direcciones IP permitidas para conectarse al extremo de vista previa. Si no se especifica ninguna dirección IP, se permitirá cualquier dirección IP. Puede especificarse una dirección IP permitida como:

* Una única dirección IP, por ejemplo, 10.0.0.1.
* Un intervalo de direcciones IP que use una dirección IP y una máscara de subred CIDR, por ejemplo, 10.0.0.1/22.
* Un intervalo de direcciones IP que use una dirección IP y una máscara de subred decimal con puntos, por ejemplo, 10.0.0.1(255.255.252.0).

### <a name="channel-output"></a>Salida del canal
Para información acerca de la salida del canal, consulte la sección [Intervalo de fotogramas clave](#keyframe_interval).

### <a name="channel-managed-programs"></a>Programas administrados por canal
Un canal está asociado a programas que puede usar para controlar la publicación y el almacenamiento de segmentos en una transmisión en vivo. Los canales administran los programas. La relación entre canales y programas es similar a los medios tradicionales, donde un canal tiene un flujo constante de contenido y un programa se enfoca en algún evento programado en dicho canal.

Puede especificar la cantidad de horas que desea conservar el contenido grabado del programa en la configuración de la duración de **Ventana de archivo** . Este valor se puede establecer desde un mínimo de cinco minutos a un máximo de 25 horas. La duración de la ventana de archivo también determina el número máximo de veces que los clientes pueden buscar hacia atrás a partir de la posición en directo actual. Los programas pueden transmitirse durante la cantidad de tiempo especificada, pero el contenido que escape de esa longitud de ventana se descartará continuamente. El valor de esta propiedad también determina durante cuánto tiempo los manifiestos de cliente pueden crecer.

Cada programa se asocia a un recurso que almacena el contenido transmitido. Un recurso se asigna a un contenedor de blobs en bloques de la cuenta de almacenamiento de Azure y los archivos del recurso se almacenan como blobs en ese contenedor. Para publicar el programa y que los clientes puedan ver la transmisión, debe crear un localizador a petición para el recurso asociado. Puede usar este localizador para crear una dirección URL de streaming que proporcionar a sus clientes.

Un canal es compatible con hasta tres programas en ejecución simultánea, por lo que puede crear varios archivos del mismo flujo entrante. Puede publicar y archivar distintas partes de un evento, según sea necesario. Por ejemplo, imagine que el requisito de su empresa es archivar seis horas de un programa, pero difundir solo los últimos diez minutos. Para lograrlo, necesita crear dos programas en ejecución simultánea. Un programa está definido para archivar seis horas del evento, pero no está publicado. El otro programa está definido para archivar diez minutos y este sí se publica.

No debe volver a usar programas existentes para eventos nuevos. En su lugar, cree un programa nuevo para cada evento. Inicie el programa cuando esté listo para iniciar el streaming y el archivo. Detenga el programa cuando quiera detener el streaming y el archivo del evento.

Para eliminar contenido archivado, detenga y elimine el programa y, a continuación, elimine el recurso asociado. No se puede eliminar un recurso si un programa lo está usando. Debe eliminarse primero el programa.

Incluso después de detener y eliminar el programa, los usuarios podrán transmitir el contenido archivado como un vídeo a petición, hasta que elimine el recurso. Si desea conservar el contenido archivado, pero no hacerlo disponible para streaming, elimine el localizador de streaming.

## <a name="channel-states-and-billing"></a><a id="states"></a>Estados de los canales y facturación
Los valores posibles para el estado actual de un canal incluyen:

* **Detenido**: este es el estado inicial del canal después de su creación. En este estado, se pueden actualizar las propiedades del canal pero no se permite el streaming.
* **Starting**: el canal se está iniciando. No se permiten actualizaciones ni streaming durante este estado. Si se produce un error, el canal vuelve al estado **Stopped** (Detenido).
* **En ejecución**: el canal puede procesar transmisiones en vivo.
* **Stopping**: el canal se está deteniendo. No se permiten actualizaciones ni streaming durante este estado.
* **Deleting**: el canal se está eliminando. No se permiten actualizaciones ni streaming durante este estado.

En la tabla siguiente se muestra cómo se asignan los estados del canal al modo de facturación.

| Estado del canal | Indicadores de la IU del portal | ¿Facturado? |
| --- | --- | --- |
| **Starting** (iniciándose) |**Starting** (iniciándose) |No (estado transitorio) |
| **Ejecución** |**Ready** (Listo) (sin programas en ejecución)<p><p>or<p>**Streaming** (al menos un programa en ejecución) |Sí |
| **Stopping** (Deteniéndose) |**Stopping** (Deteniéndose) |No (estado transitorio) |
| **Stopped** |**Stopped** |No |

## <a name="closed-captioning-and-ad-insertion"></a><a id="cc_and_ads"></a>Subtítulos e inserción de anuncios
En la siguiente tabla se muestran los estándares de subtítulos e inserción de anuncios compatibles.

| Estándar | Notas |
| --- | --- |
| CEA-708 y EIA-608 (708/608) |Estándares de subtítulos para Estados Unidos y Canadá.<p><p>Actualmente, solo se admiten subtítulos de transmisión de entrada codificada. Debe usar un codificador multimedia en vivo que pueda insertar subtítulos 608 o 708 en la transmisión codificada que se envía a Media Services. Media Services ofrecerá el contenido con subtítulos insertados en los visores. |
| TTML dentro de .ismt (pistas de texto Smooth Streaming) |El empaquetado dinámico de Media Services permite a los clientes transmitir contenido de cualquiera de los siguientes formatos: DASH, HLS o Smooth Streaming. Sin embargo, si ingiere MP4 fragmentado (Smooth Streaming) con subtítulos dentro de .ismt (pistas de texto Smooth Streaming), solo podrá entregar la transmisión a clientes de Smooth Streaming. |
| SCTE-35 |Sistema de señalización digital utilizado para poner en cola la inserción de publicidad. Los receptores descendentes usan la señal para unir la publicidad a la secuencia por el tiempo asignado. SCTE-35 se debe enviar como una pista dispersa en la secuencia de entrada.<p><p>En la actualidad, el único formato de transmisión de entrada admitido con señales de publicidad es el formato MP4 fragmentado (Smooth Streaming). El único formato de salida admitido también es Smooth Streaming. |

## <a name="considerations"></a><a id="considerations"></a>Consideraciones
Cuando se usa un codificador en vivo local para enviar una transmisión de velocidad de bits múltiple en un canal, se aplican las siguientes restricciones:

* Asegúrese de que tiene suficiente conectividad a Internet disponible para enviar datos a los puntos de ingesta.
* El uso de una dirección URL de ingesta secundaria requiere más ancho de banda.
* La transmisión entrante de velocidad de bits múltiple puede tener un máximo de 10 niveles de calidad de vídeo (capas) y un máximo de 5 pistas de audio.
* La mayor velocidad de bits promedio para cualquiera de los niveles de calidad de vídeo debe ser inferior a 10 Mbps.
* La suma de las velocidades de bits promedio para todas las secuencias de audio y vídeo debe ser inferior a 25 Mbps.
* No se puede cambiar el protocolo de entrada mientras el canal o sus programas asociados se están ejecutando. Si necesita diferentes protocolos, debe crear canales independientes para cada protocolo de entrada.
* Puede ingerir una velocidad de bits única en el canal. Pero, dado que el canal no procesa la transmisión, las aplicaciones cliente también recibirán una transmisión de velocidad de bits única. (Esta opción no se recomienda).

Aquí encontrará otras consideraciones sobre el funcionamiento de los canales y los componentes relacionados:

* Cada vez que vuelva a configurar el codificador en directo, llame al método **Restablecer** en el canal. Antes de restablecer el canal, debe detener el programa. Después de restablecer el canal, reinicie el programa.

  > [!NOTE]
  > Cuando se reinicie el programa, debe asociarlo a un nuevo recurso y crear un localizador nuevo. 
  
* Un canal se puede detener solo cuando está en el estado **En ejecución** y se han detenido todos los programas del canal.
* Solo puede agregar cinco canales a su cuenta de Media Services de forma predeterminada. Para más información, consulte [Cuotas y limitaciones](media-services-quotas-and-limitations.md).
* Solo se le cobrará cuando el canal esté en estado **En ejecución**. Para más información, consulte la sección [Estados de los canales y facturación](media-services-live-streaming-with-onprem-encoders.md#states).

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="suggestions-and-feedback"></a>Sugerencias y comentarios

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-topics"></a>Temas relacionados
[Codificadores locales recomendados](media-services-recommended-encoders.md)

[Especificación de la introducción en directo de MP4 fragmentado de Azure Media Services](../media-services-fmp4-live-ingest-overview.md)

[Información general y escenarios comunes de Azure Media Services](media-services-overview.md)

[Conceptos de Azure Media Services](media-services-concepts.md)

[live-overview]: ./media/media-services-manage-channels-overview/media-services-live-streaming-current.png
