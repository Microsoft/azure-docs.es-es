---
title: Streaming en vivo con Azure Media Services para crear transmisiones con velocidad de bits múltiple | Microsoft Docs
description: En este tema se describe cómo configurar un canal que recibe una transmisión en vivo de una sola velocidad de bits desde un codificador local y, a continuación, realiza la codificación en directo a una transmisión de velocidad de bits adaptable con Media Services.
services: media-services
documentationcenter: ''
author: anilmur
manager: femila
editor: ''
ms.assetid: 30ce6556-b0ff-46d8-a15d-5f10e4c360e2
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: anilmur
ms.reviewer: juliako
ms.openlocfilehash: 8d103e6a0f7a47aadce524325e58fbb7069a1e13
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87042815"
---
# <a name="live-streaming-using-azure-media-services-to-create-multi-bitrate-streams"></a>Streaming en vivo con Azure Media Services para crear transmisiones con velocidad de bits múltiple

> [!NOTE]
> A partir del 12 de mayo de 2018, los canales en directo ya no admitirán el protocolo de ingesta de secuencia de transporte RTP/MPEG-2. Migre de RTP/MPEG-2 a RTMP o protocolos de ingeesta de MP4 fragmentado (Smooth Streaming).

## <a name="overview"></a>Información general
En Azure Media Services (AMS), un **canal** representa una canalización para procesar contenido de streaming en vivo. Los **canales** reciben el flujo de entrada en directo de dos maneras posibles:

* Un codificador en directo local envía una secuencia de una sola velocidad de bits al canal que está habilitado para realizar codificación en directo con Media Services, con uno de los siguientes formatos: RTMP o Smooth Streaming (Fragmented MP4). Después, el canal codifica en directo la secuencia entrante de una sola velocidad de bits en una secuencia de vídeo de varias velocidades de bits (adaptable). Cuando se solicita, Media Services entrega la secuencia a los clientes.
* Un codificador local en vivo envía contenido **RTMP** o **Smooth Streaming** (MP4 fragmentado) de velocidad de bits múltiple al canal que no está habilitado para realizar la codificación en directo con AMS. Las secuencias recopiladas pasan a través de **canales**sin más procesamiento. Este método se llama **paso a través**. Puede usar los siguientes codificadores en directo que generan Smooth Streaming de velocidad de bits múltiple: MediaExcel, Ateme, Imagine Communications, Envivio, Cisco y Elemental. Los siguientes codificadores en directo generan RTMP: [Telestream Wirecast](media-services-configure-wirecast-live-encoder.md), Haivision y Teradek.  El codificador en directo también puede enviar una secuencia de una sola velocidad de bits a un canal que no está habilitado para la codificación en directo, pero esto no es recomendable. Cuando se solicita, Media Services entrega la secuencia a los clientes.

  > [!NOTE]
  > El método de paso a través es la forma más económica de realizar un streaming en vivo.
  > 
  > 

A partir de la versión 2.10 de Media Services, al crear un canal, puede especificar la forma en que desea que este reciba el flujo de entrada y si quiere que el canal realice la codificación en directo de la secuencia. Tiene dos opciones:

* **Ninguna** : especifique este valor si piensa usar un codificador en directo local que genere una secuencia de varias velocidades de bits (una transmisión de paso a través). En este caso, el flujo entrante pasa hasta la salida sin codificación alguna. Este es el comportamiento de un canal antes de la versión 2.10.  Para más información sobre cómo trabajar con los canales de este tipo, consulte [Transmisión en vivo con codificadores locales que crean transmisiones de velocidad de bits múltiple](media-services-live-streaming-with-onprem-encoders.md).
* **Estándar**: elija este valor si piensa usar Media Services para codificar transmisiones en directo con una sola velocidad de bits en transmisiones de varias velocidades de bits. Tenga en cuenta que hay un impacto en la facturación para la codificación en directo y debe recordar que salir de un canal de codificación en directo en el estado "En ejecución" supondrá un coste adicional de facturación.  Se recomienda detener inmediatamente sus canales de ejecución después que se complete su evento de transmisión en directo para evitar cargos por hora adicionales.

> [!NOTE]
> En este tema se describen los atributos de los canales habilitados para realizar la codificación en directo (tipo de codificación**Estándar** ). Para más información sobre cómo trabajar con canales no habilitados para realizar la codificación en directo, consulte [Transmisión en vivo con codificadores locales que crean transmisiones de velocidad de bits múltiple](media-services-live-streaming-with-onprem-encoders.md).
> 
> Asegúrese de revisar la sección [Consideraciones](media-services-manage-live-encoder-enabled-channels.md#Considerations) .
> 
> 

## <a name="billing-implications"></a>Implicaciones de facturación
Un canal de codificación en directo comienza la facturación tan pronto como su estado realiza la transición a "En ejecución" a través de la API.   También puede ver el estado en Azure Portal o en la herramienta del Explorador de Azure Media Services (https://aka.ms/amse) ).

En la tabla siguiente se muestra cómo se asignan los estados del canal a los estados de facturación en la API y Azure Portal. Los estados son ligeramente diferentes en la API y en la experiencia de usuario de Azure Portal. En cuanto un canal se encuentre en el estado "En ejecución" a través de la API, o en el estado "Listo" o "Streaming" en Azure Portal, la facturación estará activa.
Para hacer que el canal deje de facturarle, tendrá que detener el canal a través de la API o en Azure Portal.
Usted es responsable de detener sus canales cuando haya terminado con el canal de codificación en directo.  Si no se detiene un canal de codificación, la facturación continuará.

### <a name="channel-states-and-how-they-map-to-the-billing-mode"></a><a id="states"></a>Estados del canal y cómo se asignan al modo de facturación
El estado actual de un canal. Los valores posibles son:

* **Detenido**. Este es el estado inicial del canal después de su creación (a menos que seleccionara el inicio automático en el portal.) No se produce ninguna facturación en este estado. En este estado, se pueden actualizar las propiedades del canal pero no se permite el streaming.
* **Iniciando**. El canal se está iniciando. No se produce ninguna facturación en este estado. No se permiten actualizaciones ni streaming durante este estado. Si se produce un error, el canal vuelve al estado Detenido.
* **En ejecución**. El canal es capaz de procesar secuencias en directo. Está ahora el uso de facturación. Debe detener el canal para evitar más facturación. 
* **Deteniéndose**. El canal se está deteniendo. No se produce facturación en este estado transitorio. No se permiten actualizaciones ni streaming durante este estado.
* **Eliminando**. El canal se está eliminando. No se produce facturación en este estado transitorio. No se permiten actualizaciones ni streaming durante este estado.

En la tabla siguiente se muestra cómo se asignan los estados del canal al modo de facturación. 

| Estado del canal | Indicadores IU del portal | ¿Es la facturación? |
| --- | --- | --- |
| Iniciando |Iniciando |No (estado transitorio) |
| En ejecución |Listo (no hay programas en ejecución)<br/>or<br/>Streaming (al menos un programa en ejecución) |SÍ |
| Deteniéndose |Deteniéndose |No (estado transitorio) |
| Detenido |Detenido |No |

### <a name="automatic-shut-off-for-unused-channels"></a>Cierre automático para canales no utilizados
A partir del 25 de enero de 2016, Media Services implementó una actualización que detiene automáticamente un canal (con Live Encoding habilitado), después de haber estado ejecutándose en un estado no usado durante un largo período. Esto se aplica a los canales que no tienen ningún programa activo y que no han recibido una fuente de contribución de entrada durante un largo período de tiempo.

El umbral para un período sin usar nominalmente es de 12 horas, pero está sujeta a cambios.

## <a name="live-encoding-workflow"></a>Flujo de trabajo de Live Encoding
El siguiente diagrama representa un flujo de trabajo de streaming en vivo donde un canal recibe una secuencia de una sola velocidad de bits en uno de los siguientes protocolos: RTMP o Smooth Streaming. A continuación, la codifica como secuencia de velocidad de bits múltiple. 

![Flujo de trabajo activo][live-overview]

## <a name="common-live-streaming-scenario"></a><a id="scenario"></a>Escenario común de streaming en vivo
A continuación se indican los pasos generales para crear aplicaciones comunes de streaming en vivo.

> [!NOTE]
> Actualmente, la duración máxima recomendada de un evento en directo es de 8 horas.
>
> La codificación en directo afecta a la facturación y debe recordar que salir de un canal de codificación en directo en estado "En ejecución" supondrá un costo de facturación por hora. Se recomienda detener inmediatamente sus canales de ejecución después que se complete su evento de transmisión en directo para evitar cargos por hora adicionales. 

1. Conecte una cámara de vídeo a un equipo. Inicie y configure un codificador local en directo que pueda generar una secuencia de una **sola** velocidad de bits en uno de los siguientes protocolos: RTMP o Smooth Streaming. 

    Este paso también puede realizarse después de crear el canal.
2. Cree e inicie un canal. 
3. Recupere la URL de ingesta de canales. 

    El codificador en directo usa la URL de ingesta para enviar la secuencia al canal.
4. Recupere la URL de vista previa de canal. 

    Use esta dirección URL para comprobar que el canal recibe correctamente la secuencia en vivo.
5. Cree un programa. 

    Con Azure Portal, al crear un programa también se crea un recurso. 

    Con el SDK de .NET o REST, debe crear un recurso y especificar que este se use al crear un programa. 
6. Publique el recurso asociado al programa.   

    >[!NOTE]
    >Cuando se crea la cuenta de AMS, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Stopped** (Detenido). El punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado **Running** (En ejecución). 

7. Inicie el programa cuando esté listo para iniciar el streaming y el archivo.
8. Si lo desea, puede señalar el codificador en directo para iniciar un anuncio. El anuncio se inserta en el flujo de salida.
9. Detenga el programa cuando quiera detener el streaming y el archivo del evento.
10. Elimine el programa (y, opcionalmente, elimine el recurso).   

> [!NOTE]
> Es muy importante no olvidar detener un canal de Live Encoding. Tenga en cuenta que hay un impacto en la facturación por hora para la codificación en directo y debe recordar que salir de un canal de codificación en directo en el estado "En ejecución" supondrá un coste adicional de facturación.  Se recomienda detener inmediatamente sus canales de ejecución después que se complete su evento de transmisión en directo para evitar cargos por hora adicionales. 
> 
> 

## <a name="channels-input-ingest-configurations"></a><a id="channel"></a>Configuraciones de entrada de canal (ingesta)
### <a name="ingest-streaming-protocol"></a><a id="Ingest_Protocols"></a>Protocolo de streaming de ingesta
Si el **Tipo de codificador** está establecido en **Estándar**, las opciones válidas son:

* **RTMP**
* **MP4 fragmentado** de una sola velocidad de bits (Smooth Streaming)

#### <a name="single-bitrate-rtmp"></a><a id="single_bitrate_RTMP"></a>RTMP de una sola velocidad de bits
Consideraciones:

* La secuencia de entrada no puede contener vídeo de varias velocidades de bits.
* La secuencia de vídeo debe tener una velocidad de bits media inferior a 15 Mbps.
* La secuencia de audio debe tener una velocidad de bits media inferior a 1 Mbps.
* A continuación, se indican los códecs admitidos:
* MPEG-4 AVC/H.264 Video
* Perfil Línea de base, Principal, Alta (4:2:0 de 8 bits)
* Perfil Alta 10 (4:2:0 de 10 bits)
* Perfil Alta 422 (4:2:2 de 10 bits)
* MPEG-2 AAC-LC Audio
* Mono, estéreo, Surround (5.1, 7.1)
* Velocidad de muestreo 44,1 kHz
* Empaquetado ADTS estilo MPEG-2
* Entre los codificadores recomendados se incluyen:
* [Telestream Wirecast](media-services-configure-wirecast-live-encoder.md)
* Flash Media Live Encoder

#### <a name="single-bitrate-fragmented-mp4-smooth-streaming"></a>MP4 fragmentado de una sola velocidad de bits (Smooth Streaming)
Caso de uso típico:

Use codificadores locales en directo de proveedores como Elemental Technologies, Ericsson, Ateme y Envivio para enviar el flujo de entrada a través de Internet abierto a un centro de datos de Azure cercano.

Consideraciones:

Las mismas aplicables al apartado [RTMP de una sola velocidad de bits](media-services-manage-live-encoder-enabled-channels.md#single_bitrate_RTMP).

#### <a name="other-considerations"></a>Otras consideraciones
* No se puede cambiar el protocolo de entrada mientras el canal o sus programas asociados se están ejecutando. Si necesita diferentes protocolos, debe crear canales independientes para cada protocolo de entrada.
* La resolución máxima de la secuencia de vídeo entrante es de 1920 x 1080, con un máximo de 60 campos por segundo, cuando es entrelazada, o 30 fotogramas por segundo, si es progresiva.

### <a name="ingest-urls-endpoints"></a>Direcciones URL de ingesta (extremos)
Un canal proporciona un extremo de entrada (dirección URL de ingesta) que usted especifica en el codificador en directo, de modo que el codificador puede insertar secuencias en sus canales.

Puede obtener las direcciones URL de ingesta al crear un canal. Para obtener estas direcciones URL, el canal no puede encontrarse en el estado **En ejecución** . Cuando esté listo para comenzar a insertar datos en el canal, este debe estar **En ejecución** . Una vez que el canal empieza a consumir datos, puede obtener una vista previa de la secuencia a través de la dirección URL de vista previa.

Tiene la opción de consumir una secuencia en directo de MP4 fragmentado (Smooth Streaming) a través de una conexión TLS. Para ingerir en TLS, asegúrese de actualizar la dirección URL de introducción a HTTPS. Actualmente, AMS no admite TLS con dominios personalizados.  

### <a name="allowed-ip-addresses"></a>Direcciones IP permitidas
Puede definir las direcciones IP permitidas para publicar vídeo en el canal. Las direcciones IP permitidas se pueden especificar como una dirección IP única (por ejemplo, "10.0.0.1"), un intervalo IP que usa una dirección IP y una máscara de subred CIDR (por ejemplo, "10.0.0.1/22") o un intervalo de IP que usa una máscara de subred decimal con puntos; por ejemplo, "10.0.0.1(255.255.252.0)".

Si no se especifican direcciones IP y no hay ninguna definición de regla, no se permitirá ninguna dirección IP. Para permitir las direcciones IP, cree una regla y establezca 0.0.0.0/0.

## <a name="channel-preview"></a>Vista previa de canal
### <a name="preview-urls"></a>Direcciones URL de vista previa
Los canales proporcionan un extremo de vista previa (dirección URL de vista previa) que se puede utilizar para obtener una vista previa y validar la secuencia antes de mayor procesamiento y entrega.

Puede obtener la dirección URL de vista previa al crear el canal. Para obtenerla, el canal no puede encontrarse en el estado **En ejecución** .

Una vez que el canal empieza a consumir datos, puede obtener una vista previa de la secuencia.

> [!NOTE]
> Actualmente la secuencia de vista previa solo se puede entregar en formato MP4 fragmentado (Smooth Streaming), independientemente del tipo de entrada especificado.  Puede usar un reproductor hospedado en Azure Portal para ver la transmisión.
> 
> 

### <a name="allowed-ip-addresses"></a>Direcciones IP permitidas
Puede definir las direcciones IP permitidas para conectarse al extremo de vista previa. Si no se especifica ninguna dirección IP, se permitirá cualquier dirección IP. Las direcciones IP permitidas se pueden especificar como una dirección IP única (por ejemplo, "10.0.0.1"), un intervalo IP que usa una dirección IP y una máscara de subred CIDR (por ejemplo, "10.0.0.1/22") o un intervalo de IP que usa una máscara de subred decimal con puntos; por ejemplo, "10.0.0.1(255.255.252.0)".

## <a name="live-encoding-settings"></a>Configuración de la codificación en directo
En esta sección se describe cómo configurar los valores del codificador en directo dentro del canal cuando el **Tipo de codificación** del canal se establece en **Estándar**.

> [!NOTE]
> La fuente de contribución solo puede contener una pista de audio: la introducción de varias pistas de audio no es compatible actualmente. Al realizar la codificación en directo con [codificaciones en directo locales](media-services-live-streaming-with-onprem-encoders.md), puede enviar una fuente de contribución a través del protocolo Smooth Streaming que contenga varias pistas de audio.
> 
> 

### <a name="ad-marker-source"></a>Origen de marcador de anuncio
Puede especificar el origen de las señales de los marcadores de anuncio. El valor predeterminado es **Api**, que indica que el codificador en directo del canal debe escuchar una **API de marcadores de anuncio** asincrónica.

### <a name="cea-708-closed-captions"></a>Subtítulos CEA 708
Marca opcional que indica al codificador en directo que omita cualquier dato de los subtítulos CEA 708 insertado en el vídeo entrante. Si la marca está establecida en false (valor predeterminado), el codificador detectará y volverá a insertar los datos de CEA 708 en las secuencias de vídeo salientes.

#### <a name="index"></a>Índice
Se recomienda enviar una secuencia de transporte de un solo programa (SPTS). Si la secuencia de entrada contiene varios programas, el codificador en directo del canal analiza la tabla de asignación de programas (PMT) en la entrada, identifica las entradas con un nombre de tipo de secuencia MPEG-2 AAC ADTS, AC-3 System-A, AC-3 System-B, MPEG-2 Private PES, MPEG-1 Audio o MPEG-2 Audio y las organiza en el orden especificado en la tabla PMT. A continuación, se usa el índice basado en cero para seleccionar la entrada concreta en esa disposición.

#### <a name="language"></a>Idioma
El identificador de idioma de la secuencia de audio, conforme a ISO 639-2, por ejemplo, ENG. Si no aparece, el valor predeterminado es UND (sin definir).

### <a name="system-preset"></a><a id="preset"></a>Valor preestablecido del sistema
Especifica el valor preestablecido que usará el codificador en directo dentro de este canal. Actualmente, el único valor permitido es **Default720p** (valor predeterminado).

**Default720p** codificará el vídeo en las 6 capas siguientes.

#### <a name="output-video-stream"></a>Secuencia de vídeo de salida

| Velocidad de bits | Ancho | Alto | Fotogramas/seg. máx. | Perfil | Nombre secuencia salida |
| --- | --- | --- | --- | --- | --- |
| 3500 |1280 |720 |30 |Alto |Video_1280x720_3500kbps |
| 2200 |960 |540 |30 |Alto |Video_960x540_2200kbps |
| 1350 |704 |396 |30 |Alto |Video_704x396_1350kbps |
| 850 |512 |288 |30 |Alto |Video_512x288_850kbps |
| 550 |384 |216 |30 |Alto |Video_384x216_550kbps |
| 200 |340 |192 |30 |Alto |Video_340x192_200kbps |

#### <a name="output-audio-stream"></a>Secuencia de audio de salida

El audio se codifica como estéreo AAC-LC a 128 kbps, con una frecuencia de muestreo de 48 kHz.

## <a name="signaling-advertisements"></a>Señalización de anuncios
Si el canal tiene habilitado Live Encoding, dispone de un componente en la canalización de procesamiento de vídeo y puede manipularlo. Puede señalar que el canal inserte pizarras o anuncios en la secuencia de velocidad de bits adaptable saliente. Las pizarras son imágenes estáticas que puede usar para cubrir la fuente de entrada directa en determinados casos (por ejemplo, durante una pausa comercial). Las señales de anuncio son señales sincronizadas temporalmente que se insertan en la secuencia saliente para indicar al reproductor de vídeo que realice una acción determinada, por ejemplo, cambiar a un anuncio en el momento adecuado. Consulte este [blog](https://codesequoia.wordpress.com/2014/02/24/understanding-scte-35/) para obtener información general sobre el mecanismo de señalización SCTE-35 usado para este fin. A continuación se muestra un escenario típico que puede implementar en el evento en directo.

1. Haga que los visores obtengan una imagen ANTERIOR AL EVENTO antes de que este empiece.
2. Haga que los visores obtengan una imagen POSTERIOR AL EVENTO una vez que este finalice.
3. Haga que los visores obtengan una imagen de ERROR DEL EVENTO si hay algún problema durante el mismo (por ejemplo, un corte de luz en el estadio).
4. Envíe una imagen de PAUSA COMERCIAL para ocultar la fuente del evento en directo durante una pausa publicitaria.

A continuación se indican las propiedades que puede establecer al señalar anuncios. 

### <a name="duration"></a>Duration
La duración, en segundos, de la pausa comercial. Para que se inicie la pausa comercial, este debe ser un valor positivo distinto de cero. Si hay una pausa comercial en curso, la duración está establecida en cero y el identificador de pila coincide con el de dicha pausa comercial, la pausa se cancela.

### <a name="cueid"></a>Identificador de pila
Identificador único de la pausa comercial que la aplicación correspondiente usará para emprender las acciones adecuadas. Debe ser un entero positivo. Puede establecer este valor en un número entero positivo aleatorio o usar un sistema ascendente para realizar un seguimiento de los identificadores de pila. Asegúrese de normalizar los identificadores como enteros positivos antes de enviar a través de la API.

### <a name="show-slate"></a>Mostrar pizarra
Opcional. Indica al codificador en directo que cambie a la [careta predeterminada](media-services-manage-live-encoder-enabled-channels.md#default_slate) durante una pausa comercial y oculte la fuente de vídeo entrante. El audio también está desactivado durante el uso de la pizarra. El valor predeterminado es **false**. 

La imagen usada será la especificada mediante la propiedad Id del recurso de pizarra predeterminado en el momento de la creación del canal. La pizarra se estira para ajustarse al tamaño de la imagen en pantalla. 

## <a name="insert-slate--images"></a>Insertar imágenes de pizarra
Puede señalarse al codificador en directo del canal que cambie a una imagen de pizarra. También puede se le puede indicar que finalice una pizarra en curso. 

El codificador en directo puede configurarse para cambiar a una imagen de pizarra y ocultar la señal de vídeo entrante en determinadas situaciones, por ejemplo, durante una pausa de anuncio. Si no se ha configurado ninguna pizarra de este tipo, el vídeo de entrada no se enmascara durante esa pausa.

### <a name="duration"></a>Duration
La duración de la pizarra en segundos. Para iniciar la pizarra, este debe ser un valor positivo distinto de cero. Si hay una pizarra en curso y se especifica una duración de cero, dicha pizarra se terminará.

### <a name="insert-slate-on-ad-marker"></a>Insertar pizarra en marcador de anuncio
Cuando está establecido en true, este valor configura el codificador en directo para insertar una imagen de pizarra durante una pausa de anuncio. El valor predeterminado es true. 

### <a name="default-slate-asset-id"></a><a id="default_slate"></a>Identificador de activo de activo de tableta táctil predeterminado

Opcional. Especifica el identificador del recurso de Media Services que contiene la imagen de pizarra. El valor predeterminado es null. 


> [!NOTE] 
> Antes de crear el canal, la imagen de careta debe cargarse con las siguientes restricciones como activo dedicado (no debe haber ningún otro archivo en este activo). Esta imagen solo se utiliza cuando el codificador en directo está insertando una careta debido a una pausa publicitaria o porque se ha indicado expresamente. Actualmente, no existe la posibilidad de utilizar una imagen personalizada cuando el codificador en directo entra en un estado de 'pérdida de señal de entrada'. Puede votar por esta característica [aquí](https://feedback.azure.com/forums/169396-azure-media-services/suggestions/10190457-define-custom-slate-image-on-a-live-encoder-channel).

* Máximo 1920x1080 de resolución.
* Al menos 3 MB de tamaño.
* El nombre de archivo debe tener una extensión *.jpg.
* La imagen se debe cargar en un recurso como el único AssetFile en ese recurso y este AssetFile debe marcarse como el archivo principal. El recurso no se puede almacenar cifrado.

Si no se especifica el **identificador del recurso de pizarra predeterminado** y el valor de **Marcador para insertar pizarra en anuncio** está establecido en **True**, se usará una imagen de Azure Media Services predeterminada para ocultar la secuencia de vídeo de entrada. El audio también está desactivado durante el uso de la pizarra. 

## <a name="channels-programs"></a>Programas del canal
Un canal está asociado a programas que le permiten controlar la publicación y el almacenamiento de segmentos en una secuencia en directo. Los canales administran los programas. La relación entre canales y programas es muy similar a los medios tradicionales, donde un canal tiene un flujo constante de contenido y un programa se enfoca a algún evento programado en dicho canal.

Puede especificar la cantidad de horas que desea conservar el contenido grabado del programa en la configuración de la duración de **Ventana de archivo** . Este valor se puede establecer desde un mínimo de cinco minutos a un máximo de 25 horas. La duración de la ventana de archivo también determina el número máximo de veces que los clientes pueden buscar hacia atrás a partir de la posición en directo actual. Los programas pueden transmitirse durante la cantidad de tiempo especificada, pero el contenido que escape de esa longitud de ventana se descartará continuamente. El valor de esta propiedad también determina durante cuánto tiempo los manifiestos de cliente pueden crecer.

Cada programa se asocia a un recurso que almacena el contenido transmitido por streaming. Un recurso se asigna a un contenedor de blobs en bloques de la cuenta de Azure Storage y los archivos del recurso se almacenan como blobs en ese contenedor. Para publicar el programa a fin de que los clientes puedan ver la secuencia, debe crear un localizador a petición para el recurso asociado. Contar con este localizador le permitirá crear una dirección URL de streaming que puede proporcionar a sus clientes.

Un canal es compatible con hasta tres programas en ejecución simultánea, por lo que puede crear varios archivos del mismo flujo entrante. Esto le permite publicar y archivar distintas partes de un evento, según sea necesario. Por ejemplo, el requisito de su empresa es solo archivar seis horas de un programa, pero difundir solo los últimos diez minutos. Para lograrlo, necesita crear dos programas en ejecución simultánea. Un programa está definido para archivar seis horas del evento, pero no está publicado. El otro programa está definido para archivar durante diez minutos y este programa sí se publica.

No debe volver a usar programas existentes para eventos nuevos. En su lugar, cree e inicie un programa nuevo para cada evento como se describe en la sección Programación de aplicaciones de streaming en vivo.

Inicie el programa cuando esté listo para iniciar el streaming y el archivo. Detenga el programa cuando quiera detener el streaming y el archivo del evento. 

Para eliminar contenido archivado, detenga y elimine el programa y, a continuación, elimine el recurso asociado. No se puede eliminar un recurso si se usa un programa; primero se debe eliminar el programa. 

Incluso después de detener y eliminar el programa, los usuarios podrán transmitir el contenido archivado como un vídeo a petición siempre que no elimine el recurso.

Si desea conservar el contenido archivado, pero no hacerlo disponible para la transmisión, elimine el localizador de streaming.

## <a name="getting-a-thumbnail-preview-of-a-live-feed"></a>Obtención de una vista previa en miniatura de una fuente directa
Si Live Encoding está habilitado, puede obtener una vista previa de la fuente directa cuando llega al canal. Esta puede ser una herramienta muy valiosa para comprobar si la fuente directa llega realmente al canal. 

## <a name="channel-states-and-how-states-map-to-the-billing-mode"></a><a id="states"></a>Estados del canal y cómo se asignan los estados al modo de facturación
El estado actual de un canal. Los valores posibles son:

* **Detenido**. Este es el estado inicial del canal después de su creación. En este estado, se pueden actualizar las propiedades del canal pero no se permite el streaming.
* **Iniciando**. El canal se está iniciando. No se permiten actualizaciones ni streaming durante este estado. Si se produce un error, el canal vuelve al estado Detenido.
* **En ejecución**. El canal es capaz de procesar secuencias en directo.
* **Deteniéndose**. El canal se está deteniendo. No se permiten actualizaciones ni streaming durante este estado.
* **Eliminando**. El canal se está eliminando. No se permiten actualizaciones ni streaming durante este estado.

En la tabla siguiente se muestra cómo se asignan los estados del canal al modo de facturación. 

| Estado del canal | Indicadores IU del portal | ¿Facturado? |
| --- | --- | --- |
| Iniciando |Iniciando |No (estado transitorio) |
| En ejecución |Listo (no hay programas en ejecución)<br/>or<br/>Streaming (al menos un programa en ejecución) |Sí |
| Deteniéndose |Deteniéndose |No (estado transitorio) |
| Detenido |Detenido |No |

> [!NOTE]
> Actualmente, el promedio de inicio de canal es de aproximadamente 2 minutos, pero a veces puede tardar hasta más de 20 minutos. Los restablecimientos de canal pueden tardar hasta 5 minutos.
> 
> 

## <a name="considerations"></a><a id="Considerations"></a>Consideraciones
* Cuando un canal de tipo de codificación **estándar** experimenta una pérdida de origen de entrada/fuente de contribución, la compensa reemplazando el audio y vídeo de origen por una careta de error y silencio. El canal continuará emitiendo una careta hasta que se reanude la fuente de contribución/entrada. Se recomienda no dejar un canal en directo en este estado durante más de dos horas. A partir de este momento, no se garantiza el comportamiento del canal en la reconexión de entrada, ni su comportamiento en respuesta a un comando de restablecimiento. Tendrá que detener el canal, eliminarlo y crear uno nuevo.
* No se puede cambiar el protocolo de entrada mientras el canal o sus programas asociados se están ejecutando. Si necesita diferentes protocolos, debe crear canales independientes para cada protocolo de entrada.
* Cada vez que vuelva a configurar el codificador en directo, llame al método **Restablecer** en el canal. Antes de restablecer el canal, debe detener el programa. Después de restablecer el canal, reinicie el programa.
* Un canal se puede detener solo cuando está en el estado En ejecución y se han detenido todos los programas en el canal.
* De forma predeterminada solo puede agregar 5 canales a su cuenta de Media Services. Esta es una cuota de advertencia a todas las cuentas nuevas. Para obtener más información, consulte [Cuotas y limitaciones](media-services-quotas-and-limitations.md).
* No se puede cambiar el protocolo de entrada mientras el canal o sus programas asociados se están ejecutando. Si necesita diferentes protocolos, debe crear canales independientes para cada protocolo de entrada.
* Solo se le cobrará cuando el canal esté en estado **En ejecución** . Para obtener más información, consulte [esta](media-services-manage-live-encoder-enabled-channels.md#states) sección.
* Actualmente, la duración máxima recomendada de un evento en directo es de 8 horas. 
* Asegúrese de que el punto de conexión de streaming desde el que va a transmitir el contenido tenga el estado **En ejecución**.
* El valor predeterminado de codificación usa la noción de "velocidad de fotogramas máxima" de 30 fps. Por tanto, si la entrada es de 60fps 59.94i, los fotogramas de entrada se quitan o se elimina su entrelazado a 30/29.97 fps. Si la entrada es 50fps/50i, los fotogramas de entrada se quitan o se elimina su entrelazado a 25 fps. Si la entrada es de 25 fps, la salida permanece a 25 fps.
* No olvide DETENER SUS CANALES cuando haya terminado. Si no lo hace, la facturación continuará.

## <a name="known-issues"></a>Problemas conocidos
* El tiempo de inicio del canal se ha mejorado en un promedio de 2 minutos, pero cuando se produce mayor demanda, podría tardar hasta más de 20 minutos.
* Las imágenes de careta deben cumplir las restricciones descritas [aquí](media-services-manage-live-encoder-enabled-channels.md#default_slate). Si intenta crear un canal con una pizarra predeterminada que sea superior a 1920x1080, la solicitud terminará por producir un error.
* Una vez más... no se olvide de DETENER SUS CANALES cuando haya terminado de realizar el streaming. Si no lo hace, la facturación continuará.

## <a name="need-help"></a>¿Necesita ayuda?

Puede abrir una incidencia de soporte técnico si se desplaza a la [nueva solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="next-step"></a>Paso siguiente

Consulte las rutas de aprendizaje de Media Services.

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

## <a name="related-topics"></a>Temas relacionados
[Entrega de eventos de streaming en vivo con Azure Media Services](media-services-overview.md)

[Creación de canales que realizan la codificación en directo de una velocidad de bits única a una secuencia de velocidad de bits adaptable a través de Portal](media-services-portal-creating-live-encoder-enabled-channel.md)

[Creación de canales que realizan la codificación en directo de una velocidad de bits única a una secuencia de velocidad de bits adaptable a través del SDK de .NET](media-services-dotnet-creating-live-encoder-enabled-channel.md)

[Administración de canales con la API de REST](/rest/api/media/operations/channel)

[Conceptos de Media Services](media-services-concepts.md)

[Especificación de la introducción en directo de MP4 fragmentado de Azure Media Services](../media-services-fmp4-live-ingest-overview.md)

[live-overview]: ./media/media-services-manage-live-encoder-enabled-channels/media-services-live-streaming-new.png
