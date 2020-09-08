---
title: Conceptos de Azure Media Services | Microsoft Docs
description: En este artículo se ofrece una breve introducción a los conceptos de Microsoft Azure Media Services y vínculos a otros artículos para ampliar la información.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/14/2019
ms.author: juliako
ms.openlocfilehash: e75dde14cf7182f5153b3a42b26f1c7e8a4e78ac
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89258034"
---
# <a name="azure-media-services-concepts"></a>Conceptos de Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

En este tema se proporciona información general sobre los conceptos más importantes de Media Services.

## <a name="assets-and-storage"></a><a name="assets"></a>Recursos y almacenamiento
### <a name="assets"></a>Recursos
Un [recurso](/rest/api/media/operations/asset) contiene archivos digitales (como vídeos, audio, imágenes, colecciones de miniaturas, pistas de texto y subtítulos) y metadatos de estos archivos. Una vez que los archivos digitales se cargan en un recurso, se pueden utilizar en los flujos de trabajo de codificación y streaming en Media Services.

Un recurso se asigna a un contenedor de blobs en la cuenta de Azure Storage y los archivos del recurso se almacenan como blobs en bloques en ese contenedor. Azure Media Services no admite los blobs en páginas.

Cuando se decide qué contenido multimedia cargar y almacenar en un recurso, se aplican las siguientes consideraciones:

* Un recurso debe contener solo una instancia única de contenido multimedia. Por ejemplo, una edición única de un episodio de televisión, una película o un anuncio.
* Un recurso no puede contener varias representaciones o ediciones de un archivo audiovisual. Un ejemplo de uso inadecuado de un recurso sería intentar almacenar más de un episodio de televisión, anuncio o varios ángulos de cámara desde una sola producción dentro de un recurso. Almacenar varias representaciones o ediciones de un archivo audiovisual en un recurso puede generar dificultades en el envío de trabajos de codificación, en la transmisión y protección de la entrega del recurso más adelante en el flujo de trabajo.  

### <a name="asset-file"></a>Archivo de recursos
Un [AssetFile](/rest/api/media/operations/assetfile) representa un archivo de vídeo o audio real almacenado en un contenedor de blobs. Un archivo de recursos siempre está asociado con un recurso y un recurso puede contener uno o varios archivos. La tarea de Media Services produce un error si un objeto de archivo de recursos no está asociado a un archivo digital de un contenedor de blobs.

La instancia de **AssetFile** y el archivo multimedia real son dos objetos distintos. La instancia de AssetFile contiene metadatos sobre el archivo multimedia, mientras que el archivo multimedia contiene el contenido multimedia real.

No debe intentar cambiar el contenido de los contenedores de blobs que generó Media Services sin usar las API de Media Services.

### <a name="asset-encryption-options"></a>Opciones de cifrado de recursos
Según el tipo de contenido que desea cargar, almacenar y entregar, Media Services proporciona varias opciones de cifrado de las que puede elegir.

>[!NOTE]
>No se utiliza cifrado. Este es el valor predeterminado. Cuando se utiliza esta opción, el contenido no está protegido en tránsito o en reposo en el almacenamiento.

Si planifica entregar un MP4 mediante una descarga progresiva, use esta opción para cargar el contenido.

**StorageEncrypted** : use esta opción para cifrar el contenido no cifrado de manera local mediante el cifrado AES de 256 bits y, a continuación, cargarlo a Azure Storage donde se almacena cifrado en reposo. Los recursos protegidos con el cifrado de almacenamiento se descifran automáticamente y se colocan en un sistema de archivos cifrados antes de la codificación y, opcionalmente, se vuelven a cifrar antes de volver a cargarlos como un nuevo recurso de salida. El caso de uso principal para el cifrado de almacenamiento es cuando desea proteger los archivos multimedia de entrada de alta calidad con un sólido cifrado en reposo en disco. 

Para entregar a un recurso cifrado de almacenamiento, debe configurar la directiva de entrega del recurso para que Media Services sepa cómo desea entregar el contenido. Antes de poder transmitir el recurso, el servidor de streaming quita el cifrado de almacenamiento y transmite el contenido usando la directiva de entrega especificada (por ejemplo, AES, PlayReady o sin cifrado). 

**CommonEncryptionProtected** : utilice esta opción si desea cifrar contenido o cargar contenido ya cifrado con cifrado común o DRM de PlayReady (por ejemplo, Smooth Streaming protegido con DRM de PlayReady).

**EnvelopeEncryptedProtected** : utilice esta opción si desea proteger HTTP Live Streaming (HLS) cifrado (o cargar uno ya protegido) con estándar de cifrado avanzado (AES). Si carga HLS ya cifrado con AES, se debe haber cifrado con Transform Manager.

### <a name="access-policy"></a>Directiva de acceso
Un [AccessPolicy](/rest/api/media/operations/accesspolicy) define los permisos (como lectura, escritura y lista) y la duración de acceso a un recurso. Normalmente pasaría un objeto AccessPolicy a un localizador que luego se usaría para tener acceso a los archivos contenidos en un recurso.

>[!NOTE]
>Hay un límite de 1 000 000 directivas para diferentes directivas de AMS (por ejemplo, para la directiva de localizador o ContentKeyAuthorizationPolicy). Debe usar el mismo identificador de directiva si siempre usa los mismos permisos de acceso y días, por ejemplo, directivas para localizadores que vayan a aplicarse durante mucho tiempo (directivas distintas a carga). Para obtener más información, consulte [este tema](media-services-dotnet-manage-entities.md#limit-access-policies) .

### <a name="blob-container"></a>Contenedor de blobs
Un contenedor de blobs proporciona una agrupación de un conjunto de blobs. Los contenedores de blobs se usan en Media Services como punto limítrofe para el control de acceso y localizadores de firma de acceso compartido (SAS) en los recursos. Una cuenta de Azure Storage puede contener una cantidad ilimitada de contenedores de blobs. Un contenedor puede almacenar un número ilimitado de blobs.

>[!NOTE]
> No debe intentar cambiar el contenido de los contenedores de blobs que generó Media Services sin usar las API de Media Services.
> 
> 

### <a name="locators"></a><a name="locators"></a>Localizadores
Los [localizador](/rest/api/media/operations/locator)es proporcionan un punto de entrada para tener acceso a los archivos que se encuentran en un recurso. Se usa una directiva de acceso para definir los permisos y la duración en que un cliente tiene acceso a un recurso determinado. Los localizadores pueden tener de varias a una relación con una directiva de acceso, de manera tal que distintos localizadores pueden proporcionar distintas horas de inicio y tipos de conexión a distintos clientes, mientras que todos usan la misma configuración de permiso y duración; sin embargo, debido a una restricción en la directiva de acceso compartido definida por los servicios de almacenamiento de Azure, no puede tener más de cinco localizadores únicos asociados con un recurso determinado a la vez. 

Media Services admite dos tipos de localizadores: OnDemandOrigin, que se usan para hacer streaming de elementos multimedia (por ejemplo, MPEG DASH, HLS o Smooth Streaming) o para descargar elementos multimedia de manera progresiva; y SAS URL, que se usan para cargar o descargar archivos multimedia hacia y desde Azure Storage. 

>[!NOTE]
>El permiso de lista (AccessPermissions.List) no se debe usar al crear un localizador OnDemandOrigin. 

### <a name="storage-account"></a>Cuenta de almacenamiento
Todo el acceso a Azure Storage se realiza a través de una cuenta de almacenamiento. Una cuenta de Servicios multimedia se puede asociar con una o más cuentas de almacenamiento. Una cuenta puede contener una cantidad ilimitada de contenedores, siempre que su tamaño total no supere los 500 TB por cuenta de almacenamiento.  Media Services proporciona herramientas del nivel de SDK que le permiten administrar varias cuentas de almacenamiento y equilibrar la carga de la distribución de sus recursos durante la carga a estas cuentas según métricas o una distribución aleatoria. Para obtener más información, consulte Uso de [Azure Storage](/previous-versions/azure/dn767951(v=azure.100)). 

## <a name="jobs-and-tasks"></a>Trabajos y tareas
Un [trabajo](/rest/api/media/operations/job) se usa normalmente para procesar (por ejemplo, indexar o codificar) una presentación de audio/vídeo. Si procesa varios vídeos, cree un trabajo para cada vídeo que se va a codificar.

Un trabajo contiene metadatos acerca del procesamiento que se realizará. Cada trabajo contiene una o varias [tareas](/rest/api/media/operations/task)que especifican una tarea de procesamiento atómica, sus recursos de entrada, recursos de salida, un procesador de multimedia y su configuración asociada. Las tareas dentro de un trabajo se pueden encadenar en conjunto, donde el recurso de salida de una de las tareas se indica como el recurso de entrada de la tarea siguiente. De este modo, un trabajo puede contener todos los procesos necesarios para una presentación multimedia.

## <a name="encoding"></a><a id="encoding"></a>Encoding
Azure Media Services ofrece varias opciones para la codificación de medios en la nube.

Cuando comience con Media Services, es importante comprender la diferencia entre códecs y formatos de archivo.
Los códecs son el software que implementa los algoritmos de compresión/descompresión, mientras que los formatos de archivo son contenedores que contienen el vídeo comprimido.

Media Services proporciona empaquetado dinámico que permite entregar contenido codificado MP4 de velocidad de bits adaptable o Smooth Streaming en formatos de streaming admitidos por Media Services (MPEG-DASH, HLS y Smooth Streaming) sin tener que volver a realizar el empaquetamiento en estos formatos de streaming.

Para aprovechar el [empaquetado dinámico](media-services-dynamic-packaging-overview.md), tiene que modificar o transcodificar el archivo intermedio (origen) en un conjunto de archivos MP4 de velocidad de bits adaptable o de Smooth Streaming de velocidad de bits adaptable y tener al menos un punto de conexión de streaming estándar o premium en estado iniciado.

Media Services admite los siguientes codificadores a petición que se describen en este artículo:

* [Media Encoder Standard](media-services-encode-asset.md#media-encoder-standard)
* [Flujo de trabajo del Codificador multimedia](media-services-encode-asset.md#media-encoder-premium-workflow)

Para obtener información acerca de los codificadores compatibles, consulte [Codificadores](media-services-encode-asset.md).

## <a name="live-streaming"></a>Streaming en directo
En Azure Media Services, un canal representa una canalización para procesar contenido de streaming en directo. Los canales reciben el flujo de entrada en directo de dos maneras posibles:

* Un codificador local en directo envía contenido RTMP o Smooth Streaming (MP4 fragmentado) de varias velocidades de bits al canal. Puede usar los siguientes codificadores en directo que generan Smooth Streaming de velocidad de bits múltiple: MediaExcel, Ateme, Imagine Communications, Envivio, Cisco y Elemental. Los siguientes codificadores en directo generan RTMP: Adobe Flash Live Encoder, [Telestream Wirecast](media-services-configure-wirecast-live-encoder.md), Teradek y Haivision. Las secuencias recopiladas pasan a través de canales sin más procesamiento ni codificación. Cuando se solicita, Media Services entrega la secuencia a los clientes.
* Una secuencia de velocidad de bits única (en uno de los siguientes formatos: RTMP o Smooth Streaming (MP4 fragmentado)) se envía al canal habilitado para realizar la codificación en directo con Media Services. Después, el canal codifica en directo la secuencia entrante de una sola velocidad de bits en una secuencia de vídeo de varias velocidades de bits (adaptable). Cuando se solicita, Media Services entrega la secuencia a los clientes.

### <a name="channel"></a>Canal
En Media Services, los [canal](/rest/api/media/operations/channel)es son los responsables de procesar el contenido de streaming en vivo. Un canal proporciona un extremo de entrada (dirección URL de introducción) que luego se brinda a un transcodificador en vivo. El canal recibe flujos de entrada en vivo desde el transcodificador en vivo y las deja a disposición del streaming a través de uno o más StreamingEndpoints. Los canales también proporcionan un extremo de vista previa (dirección URL de vista previa) que se puede utilizar para obtener una vista previa y validar el flujo antes de mayor procesamiento y entrega.

Puede obtener la dirección URL de introducción y la dirección URL de vista previa cuando crea el canal. Para obtener estas direcciones URL, el canal no puede encontrarse en el estado iniciado. Cuando está listo para comenzar a insertar datos desde un transcodificador en vivo al canal, este debe estar iniciado. Una vez que el transcodificar en vivo comienza a introducir datos, puede tener una vista previa de la transmisión.

Cada cuenta de Media Services puede contener varios canales, varios programas y varios StreamingEndpoints. Según las necesidades de ancho de banda y seguridad, los servicios de StreamingEndpoint pueden dedicarse a uno o más canales. Puede extraer cualquier StreamingEndpoint de cualquier canal.

### <a name="program-event"></a>Programa (evento)
Un [programa (evento)](/rest/api/media/operations/program) permite controlar la publicación y el almacenamiento de segmentos en una secuencia en directo. Los canales administran los programas (eventos). La relación entre canales y programas es similar a los medios tradicionales, donde un canal tiene un flujo constante de contenido y un programa se enfoca en algún evento programado en dicho canal.
Puede especificar la cantidad de horas que desea conservar el contenido grabado del programa en la configuración de la propiedad **ArchiveWindowLength** . Este valor se puede establecer desde un mínimo de cinco minutos a un máximo de 25 horas.

ArchiveWindowLength también indica el tiempo máximo que los clientes pueden buscar hacia atrás a partir de la posición en vivo actual. Los programas pueden transmitirse durante la cantidad de tiempo especificada, pero el contenido que escape de esa longitud de ventana se descartará continuamente. El valor de esta propiedad también determina durante cuánto tiempo los manifiestos de cliente pueden crecer.

Cada programa (evento) está asociado a un recurso. Para publicar el programa, debe crear un localizador para el recurso asociado. Contar con este localizador le permitirá crear una dirección URL de streaming que puede proporcionar a sus clientes.

Un canal es compatible con hasta tres programas en ejecución simultánea, por lo que puede crear varios archivos del mismo flujo entrante. Esto le permite publicar y archivar distintas partes de un evento, según sea necesario. Por ejemplo, el requisito de su empresa es solo archivar seis horas de un programa, pero difundir solo los últimos diez minutos. Para lograrlo, necesita crear dos programas en ejecución simultánea. Un programa está definido para archivar seis horas del evento, pero no está publicado. El otro programa está definido para archivar durante diez minutos y este programa sí se publica.

Para más información, consulte:

* [Uso de canales habilitados para Live Encoding con Azure Media Services](media-services-manage-live-encoder-enabled-channels.md)
* [Uso de canales que reciben streaming en vivo con velocidad de bits múltiple de codificadores locales](media-services-live-streaming-with-onprem-encoders.md)
* [Cuotas y limitaciones](media-services-quotas-and-limitations.md).

## <a name="protecting-content"></a>Protección del contenido
### <a name="dynamic-encryption"></a>Cifrado dinámico
Azure Media Services le permite proteger su contenido multimedia desde el momento en que deja el equipo a través de almacenamiento, procesamiento y entrega. Media Services permite entregar el contenido cifrado de forma dinámica con Estándar de cifrado avanzado (AES) (mediante claves de cifrado de 128 bits) y cifrado común (CENC) mediante DRM de PlayReady o Widevine. Media Services también proporciona un servicio para entregar claves AES y licencias de PlayReady a clientes autorizados.

Actualmente puede cifrar los formatos de streaming siguientes: HLS, MPEG DASH y Smooth Streaming. No se pueden cifrar las descargas progresivas.

Si desea que Media Services cifre un recurso, debe asociar una clave de cifrado (CommonEncryption o EnvelopeEncryption) con su recurso y, además, configurar directivas de autorización para la clave.

Para transmitir un activo de cifrado de almacenamiento, debe configurar la directiva de entrega del activo para especificar cómo desea entregarlo.

Cuando un reproductor solicita una transmisión, Media Services usa la clave especificada para cifrar de forma dinámica el contenido mediante un cifrado de sobre (con AES) o uno común (con PlayReady o Widevine). Para descifrar la secuencia, el reproductor solicitará la clave del servicio de entrega de claves. Para decidir si el usuario está o no autorizado para obtener la clave, el servicio evalúa las directivas de autorización que especificó para la clave.

### <a name="token-restriction"></a>Restricción de token
La directiva de autorización de clave de acceso podría tener una o más restricciones de autorización: abrir, restricción de token o restricción de IP. La directiva con restricción token debe ir acompañada de un token emitido por un Servicio de tokens seguros (STS). Media Services admite tokens en formato Token de web simple (SWT) y en formato Token de web JSON (JWT). Media Services no proporciona Servicios de tokens seguros. Puede crear un STS personalizado. Se debe configurar el STS para crear un token firmado con las notificaciones de clave y emisión que especificó en la configuración de restricción de tokens. El servicio de entrega de claves de Media Services devolverá la clave solicitada (o licencia) al cliente si el token es válido y las reclamaciones del token coinciden con las configuradas para la clave (o licencia).

Al configurar la directiva de restricción de token, debe especificar los parámetros de clave de comprobación principal, emisor y público. La clave de comprobación principal contiene la clave con la que se firmó el token y el emisor es el servicio de tokens seguros que emite el token. El público (a veces denominado ámbito) describe la intención del token o del recurso cuyo acceso está autorizado por el token. El servicio de entrega de claves de los Media Services valida que estos valores del token coincidan con los valores de la plantilla.

Para más información, consulte los siguientes artículos.
- [Información general de protección del contenido](media-services-content-protection-overview.md)
- [Protección con AES-128](media-services-protect-with-aes128.md)
- [Protección con PlayReady/Widevine](media-services-protect-with-playready-widevine.md)

## <a name="delivering"></a>Entrega
### <a name="dynamic-packaging"></a><a name="dynamic_packaging"></a>Empaquetado dinámico
Cuando se trabaja con Media Services, es aconsejable codificar los archivos intermedios en una conjunto MP4 de velocidad de bits adaptable y, a continuación, convertir el conjunto en el formato deseado con el [Empaquetado dinámico](media-services-dynamic-packaging-overview.md).

### <a name="streaming-endpoint"></a>extremo de streaming
Un StreamingEndpoint representa un servicio de streaming que puede entregar contenido directamente a una aplicación de reproductor de cliente o a una instancia de Content Delivery Network (CDN) para su posterior distribución (Azure Media Services ahora proporciona la integración de Azure CDN). La secuencia de salida del servicio de punto de conexión de streaming puede ser streaming en vivo o un recurso de vídeo a petición en la cuenta de Media Services. Los clientes de Media Services eligen un punto de conexión de streaming **estándar** o uno o varios puntos de conexión de streaming **premium**, según sus necesidades. El punto de conexión de streaming estándar es adecuado para la mayoría de las cargas de trabajo de streaming. 

El punto de conexión de streaming estándar es adecuado para la mayoría de las cargas de trabajo de streaming. Los puntos de conexión de streaming estándar ofrecen la flexibilidad de entregar el contenido a prácticamente todos los dispositivos a través de empaquetado dinámico en HLS, MPEG-DASH y Smooth Streaming, así como cifrado dinámico para Microsoft PlayReady, Google Widevine, Apple Fairplay y AES128.  También se adaptan a tamaños de audiencias desde muy pequeñas a muy grandes con miles de personas visualizando simultáneamente a través de la integración de Azure CDN. Si tiene una carga de trabajo avanzada o sus requisitos de capacidad de streaming no se ajustan a los objetivos de rendimiento del punto de conexión de streaming estándar o desea controlar la capacidad del servicio de StreamingEndpoint para administrar las crecientes necesidades de ancho de banda, se recomienda asignar unidades de escalado (conocidas también como unidades de streaming premium).

Se recomienda utilizar el cifrado dinámico o el empaquetado dinámico.

>[!NOTE]
>Cuando se crea la cuenta de AMS, se agrega un punto de conexión de streaming **predeterminado** a la cuenta en estado **Stopped** (Detenido). Para iniciar la transmisión del contenido y aprovechar el empaquetado dinámico y el cifrado dinámico, el punto de conexión de streaming desde el que va a transmitir el contenido debe estar en estado **Running** (En ejecución). 

Para obtener más información, consulte [este tema](media-services-portal-manage-streaming-endpoints.md) .

De forma predeterminada, puede disponer de hasta 2 canales en streaming en su cuenta de Media Services. Para solicitar un límite superior, consulte [Cuotas y limitaciones](media-services-quotas-and-limitations.md).

Solo se le cobrará cuando StreamingEndpoint esté en estado en ejecución.

### <a name="asset-delivery-policy"></a>Directiva de entrega de recursos
Uno de los pasos del flujo de trabajo de entrega de contenido de Media Services consiste en configurar [directivas de entrega para los recursos ](/rest/api/media/operations/assetdeliverypolicy)que quiere transmitir. La directiva de entrega de recursos indica a los Media Services cómo desea usted que se entregue el recurso: en qué protocolo de streaming se debe empaquetar de forma dinámica el recurso (por ejemplo, MPEG DASH, HLS, Smooth Streaming o todos) o si desea o no cifrar de forma dinámica el recurso y de qué manera (cifrado de sobre o común).

Si tiene un recurso cifrado de almacenamiento, antes de poder transmitir el recurso, el servidor de streaming quita el cifrado de almacenamiento y transmite el contenido usando la directiva de entrega especificada. Por ejemplo, para entregar el recurso cifrado con clave de cifrado del estándar de cifrado avanzado (AES), defina el tipo de directiva en DynamicEnvelopeEncryption. Para quitar el cifrado de almacenamiento y transmitir el recurso sin cifrar, establezca el tipo de directiva en NoDynamicEncryption.

### <a name="progressive-download"></a>Descarga progresiva
La descarga progresiva le permite comenzar a reproducir archivos multimedia antes de haber descargado todo el archivo. Solo puede descargar progresivamente un archivo MP4.

>[!NOTE]
>Debe descifrar los recursos cifrados si desea que estén disponibles para la descarga progresiva.

Para proporcionar direcciones URL de descarga progresiva a los usuarios, primero debe crear un localizador OnDemandOrigin. Crear el localizador le brinda la ruta de acceso de base al recurso. Luego debe anexar el nombre del archivo MP4. Por ejemplo:

`http://amstest1.streaming.mediaservices.windows.net/3c5fe676-199c-4620-9b03-ba014900f214/BigBuckBunny_H264_650kbps_AAC_und_ch2_96kbps.mp4`

### <a name="streaming-urls"></a>Direcciones URL de streaming
Transmisión del contenido a los clientes. Para proporcionar direcciones URL de streaming a los usuarios, primero debe crear un localizador OnDemandOrigin. Crear el localizador le brinda la ruta de acceso de base al recurso que contiene el contenido que desea transmitir. Sin embargo, para poder transmitir este contenido, es necesario modificar aún más esta ruta de acceso. Para construir una dirección URL completa al archivo de manifiesto del streaming, debe concatenar el valor de la ruta de acceso del localizador y el nombre de archivo del manifiesto (filename.ism). Luego, anexe /Manifiesto y un formato adecuado (si corresponde) a la ruta de acceso del localizador.

También puede transmitir el contenido por una conexión TLS. Para ello, asegúrese de que las URL de streaming comienzan por HTTPS. Actualmente, AMS no admite TLS con dominios personalizados.  

>[!NOTE]
>Solo puede transmitir por TLS si se creó el punto de conexión de streaming desde el que se entrega el contenido a partir del 10 de septiembre de 2014. Si las direcciones URL de streaming se basan en los puntos de conexión de streaming creados después del 10 de septiembre, la dirección URL contendrá "streaming.mediaservices.windows.net" (el formato nuevo). Las direcciones URL de streaming que contengan "origin.mediaservices.windows.net" (el formato anterior) no son compatibles con TLS. Si la dirección URL tiene un formato antiguo y quiere transmitir a través de TLS, cree un punto de conexión de streaming nuevo. Utilice direcciones URL creadas en función del nuevo extremo de streaming para transmitir el contenido a través de TLS.

En la siguiente lista se describen distintos formatos de streaming y aparecen ejemplos:

* Smooth Streaming

{nombre de extremo de streaming-nombre de cuenta de servicios multimedia}.streaming.mediaservices.windows.net/{Id. de localizador}/{filename}.ism/Manifest

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest

* MPEG DASH

{nombre de extremo de streaming-nombre de cuenta de servicios multimedia}.streaming.mediaservices.windows.net/{Id. de localizador}/{nombre de archivo}.ism/Manifest(formato=mpd-time-csf)

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=mpd-time-csf)

* Apple HTTP Live Streaming (HLS) V4

{nombre de extremo de streaming-nombre de cuenta de servicios multimedia}.streaming.mediaservices.windows.net/{Id. de localizador}/{nombre de archivo}.ism/Manifest(formato=m3u8-aapl)

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl)

* Apple HTTP Live Streaming (HLS) V3

{nombre de extremo de streaming-nombre de cuenta de servicios multimedia}.streaming.mediaservices.windows.net/{Id. de localizador}/{nombre de archivo}.ism/Manifest(formato=m3u8-aapl-v3)

http:\//testendpoint-testaccount.streaming.mediaservices.windows.net/fecebb23-46f6-490d-8b70-203e86b0df58/BigBuckBunny.ism/Manifest(format=m3u8-aapl-v3)

## <a name="additional-notes"></a>Notas adicionales

* Widevine es un servicio que ofrece Google Inc. y que está sujeto a los términos del servicio y la directiva de privacidad de Google, Inc.

## <a name="media-services-learning-paths"></a>Rutas de aprendizaje de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
