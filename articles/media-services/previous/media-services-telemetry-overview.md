---
title: Telemetría de Azure Media Services | Microsoft Docs
description: En este artículo se proporciona información general sobre la telemetría de Microsoft Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 95c20ec4-c782-4063-8042-b79f95741d28
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/01/2019
ms.author: juliako
ms.openlocfilehash: 0701e9c6428283d45cf4b4a2e24c8de99d9a286b
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89265905"
---
# <a name="azure-media-services-telemetry"></a>Sistema de telemetría de Azure Media Services  

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!NOTE]
> No hay características o funcionalidades nuevas para agregar a Media Services, versión 2. <br/>Finalice la compra de la versión más reciente, [Media Services v3](../latest/index.yml). Consulte también la [guía de migración de la versión v2 a v3](../latest/migrate-from-v2-to-v3.md).

Gracias a Azure Media Services (AMS), podrá acceder a los datos de telemetría y métricas de dicha plataforma. La versión actual de AMS permite recopilar datos de telemetría de entidades activas **StreamingEndpoint**, **Channel** y **Archive**. 

Los datos de telemetría se crean en una tabla de almacenamiento de una cuenta de Azure Storage que especifique (normalmente, se usa la cuenta de almacenamiento asociada a la de AMS). 

El sistema de telemetría no administra la retención de datos. Puede quitar los datos antiguos de telemetría eliminando las tablas de almacenamiento.

En este tema se explica cómo configurar y usar el sistema de telemetría de AMS.

## <a name="configuring-telemetry"></a>Configuración del sistema de telemetría

Puede configurar la telemetría en la granularidad de nivel de componente. Hay dos niveles de detalle "Normal" y "Detallado". Actualmente, los dos devuelven la misma información. Se recomienda utilizar el nivel Normal. 

En los temas siguientes se muestra cómo habilitar el sistema de telemetría:

[Habilitación del sistema de telemetría con .NET](media-services-dotnet-telemetry.md) 

[Habilitación del sistema de telemetría con REST](media-services-rest-telemetry.md)

## <a name="consuming-telemetry-information"></a>uso de información de telemetría

Los datos de telemetría se crean en una tabla de Azure Storage de la cuenta de almacenamiento que especificó al configurar el sistema de telemetría para la cuenta de Media Services. En esta sección se describen las tablas de almacenamiento de las métricas.

Puede utilizar los datos de telemetría de una de las maneras siguientes:

- Leer datos directamente desde Azure Table Storage (por ejemplo, mediante el SDK de Storage). Para la descripción de las tablas de almacenamiento de datos de telemetría, consulte la sección sobre **uso de información de telemetría** de [este](/previous-versions/azure/mt742089(v=azure.100)) tema.

Or

- Aproveche la compatibilidad del SDK de .NET de Media Services para leer los datos de almacenamiento, tal y como se explica en [este](media-services-dotnet-telemetry.md) tema. 


El esquema de telemetría que se describe a continuación está diseñado para proporcionar un buen rendimiento dentro de los límites de almacenamiento de Table Storage de Azure:

- Los datos se dividen por identificador de cuenta y de servicio para que se puedan consultar de forma independiente los datos de telemetría de cada servicio.
- Las particiones contienen la fecha para que pueda asignar un límite superior razonable en el tamaño de la partición.
- Las claves de fila están en orden temporal inverso para poder consultar los elementos de telemetría más recientes en un servicio determinado.

Gracias a esto, muchas de las consultas comunes serán eficaces:

- Descarga paralelas e independientes de datos de servicios específicos
- Recuperación de todos los datos de un servicio determinado en un intervalo de fechas
- Recuperación de los datos más recientes de un servicio

### <a name="telemetry-table-storage-output-schema"></a>Esquema de salida de almacenamiento de tablas de telemetría

Los datos de telemetría se almacenan en una tabla, TelemetryMetrics20160321, donde 20160321 es la fecha de la tabla creada. El sistema de telemetría creará una tabla independiente para cada día nuevo con el formato de hora 00:00 UTC. La tabla se utiliza para almacenar valores de repetición, como la velocidad de bits de ingesta en una determinada ventana de tiempo, los bytes enviados, etc. 

Propiedad|Value|Ejemplos y notas
---|---|---
PartitionKey|{IDdeCuenta}_{IDdeEntidad}|e49bef329c29495f9b9570989682069d_64435281c50a4dd8ab7011cb0f4cdf66<br/<br/>El identificador de cuenta se incluye en la clave de partición para simplificar los flujos de trabajo, donde se crearán varias cuentas de Media Services en la misma cuenta de almacenamiento.
RowKey|{segundosParaLaMedianoche}_{valorAleatorio}|01688_00199<br/><br/>La clave de fila comienza con el número de segundos para la medianoche con el fin de que puedan realizarse N consultas de estilo superiores en una partición. Para obtener más información, consulte [este](../../cosmos-db/table-storage-design-guide.md#log-tail-pattern) artículo. 
Timestamp|Fecha y hora|Marca de tiempo creada automáticamente en la tabla de Azure: 2016-09-09T22:43:42.241Z.
Tipo|El tipo de la entidad que proporciona los datos de telemetría|Channel, StreamingEndpoint y Archive<br/><br/>El tipo de evento es un valor de cadena.
Nombre|El nombre del evento de telemetría|ChannelHeartbeat y StreamingEndpointRequestLog
ObservedTime|La hora a la que se produjo el evento de telemetría (UTC)|2016-09-09T22:42:36.924Z<br/><br/>La entidad que envía los datos de telemetría (por ejemplo, un canal) proporciona la hora observada. Puede haber problemas de sincronización en los componentes, así que este valor es aproximado.
ServiceID|{IDdeServicio}|f70bd731-691d-41c6-8f2d-671d0bdc9c7e
Propiedades específicas de la entidad|Tal y como se define en el evento|StreamName: stream1, Bitrate 10123, etc.<br/><br/>Las propiedades restantes se definen para el tipo de evento determinado. El contenido de la tabla de Azure son pares clave-valor  (es decir, las diferentes filas de la tabla tienen distintos conjuntos de propiedades).

### <a name="entity-specific-schema"></a>Esquema específico de la entidad

Hay tres tipos específicos de entradas de datos de telemetría específicas de la entidad que se insertan con la siguiente frecuencia:

- Puntos de conexión de streaming: cada 30 segundos
- Canales activos: cada minuto
- Archivo activo: cada minuto

**Punto de conexión de streaming**

Propiedad|Value|Ejemplos
---|---|---
PartitionKey|PartitionKey|e49bef329c29495f9b9570989682069d_64435281c50a4dd8ab7011cb0f4cdf66
RowKey|RowKey|01688_00199
Timestamp|Timestamp|Marca de tiempo creada automáticamente en la tabla de Azure: 2016-09-09T22:43:42.241Z.
Tipo|Tipo|StreamingEndpoint
Nombre|Nombre|StreamingEndpointRequestLog
ObservedTime|ObservedTime|2016-09-09T22:42:36.924Z
ServiceID|Identificador de servicio|f70bd731-691d-41c6-8f2d-671d0bdc9c7e
HostName|Nombre de host del punto de conexión|builddemoserver.origin.mediaservices.windows.net
StatusCode|Estado HTTP de los registros|200
ResultCode|Detalles del código de resultado|S_OK
RequestCount|Total de solicitudes en la agregación|3
BytesSent|Bytes agregados enviados|2987358
ServerLatency|Latencia media del servidor (incluido el almacenamiento)|129
E2ELatency|Latencia media de extremo a extremo|250

**Canal activo**

Propiedad|Value|Ejemplos y notas
---|---|---
PartitionKey|PartitionKey|e49bef329c29495f9b9570989682069d_64435281c50a4dd8ab7011cb0f4cdf66
RowKey|RowKey|01688_00199
Timestamp|Timestamp|Marca de tiempo creada automáticamente en la tabla de Azure: 2016-09-09T22:43:42.241Z.
Tipo|Tipo|Canal
Nombre|Nombre|ChannelHeartbeat
ObservedTime|ObservedTime|2016-09-09T22:42:36.924Z
ServiceID|Identificador de servicio|f70bd731-691d-41c6-8f2d-671d0bdc9c7e
TrackType|Tipo de pista: audio, vídeo o texto|Audio y vídeo
TrackName|Nombre de la pista|vídeo/audio_1
Bitrate|Velocidad de bits de la pista|785000
CustomAttributes||   
IncomingBitrate|Velocidad de bits entrante real|784548
OverlapCount|Superposición en la ingesta|0
DiscontinuityCount|Interrupción de la pista|0
LastTimestamp|Última marca de tiempo de los datos integrados|1800488800
NonincreasingCount|Número de fragmentos descartados debido a una marca de tiempo que no aumenta|2
UnalignedKeyFrames|Si se reciben fragmentos (en los distintos niveles de calidad) en los que los fotogramas clave no están alineados |True
UnalignedPresentationTime|Si se reciben fragmentos (en los distintos niveles de calidad y pistas) en los que no se encuentra el tiempo de presentación|True
UnexpectedBitrate|True, si la velocidad de bits calculada o real de la pista de audio o vídeo es superior a 40 000 bps, y si el valor de IncomingBitrate es 0 o los valores de IncomingBitrate y actualBitrate difieren en un 50 % |True
Healthy|True, si los valores de <br/>overlapCount, <br/>DiscontinuityCount, <br/>NonIncreasingCount, <br/>UnalignedKeyFrames, <br/>UnalignedPresentationTime y <br/>UnexpectedBitrate<br/> son 0|True<br/><br/>Healthy es una función compuesta que devuelve el valor False cuando se da una de las siguientes condiciones:<br/><br/>- OverlapCount > 0<br/>- DiscontinuityCount > 0<br/>- NonincreasingCount > 0<br/>- UnalignedKeyFrames == True<br/>- UnalignedPresentationTime == True<br/>- UnexpectedBitrate == True

**Archivo activo**

Propiedad|Value|Ejemplos y notas
---|---|---
PartitionKey|PartitionKey|e49bef329c29495f9b9570989682069d_64435281c50a4dd8ab7011cb0f4cdf66
RowKey|RowKey|01688_00199
Timestamp|Timestamp|Marca de tiempo creada automáticamente en la tabla de Azure: 2016-09-09T22:43:42.241Z.
Tipo|Tipo|Archivar
Nombre|Nombre|ArchiveHeartbeat
ObservedTime|ObservedTime|2016-09-09T22:42:36.924Z
ServiceID|Identificador de servicio|f70bd731-691d-41c6-8f2d-671d0bdc9c7e
ManifestName|URL de programa|asset-eb149703-ed0a-483c-91c4-e4066e72cce3/a0a5cfbf-71ec-4bd2-8c01-a92a2b38c9ba.ism
TrackName|Nombre de la pista|audio_1
TrackType|Tipo de pista|Audio y vídeo
CustomAttribute|Cadena hexadecimal que distingue entre pistas diferentes con el mismo nombre y la misma velocidad de bits (ángulos multicámara)|
Bitrate|Velocidad de bits de la pista|785000
Healthy|True, si el valor de FragmentDiscardedCount es 0 y el de ArchiveAcquisitionError, False|True (estos dos valores no están presentes en la métrica, pero sí en el evento de origen)<br/><br/>Healthy es una función compuesta que devuelve el valor False cuando se da una de las siguientes condiciones:<br/><br/>- FragmentDiscardedCount > 0<br/>- ArchiveAcquisitionError == True

## <a name="general-qa"></a>Preguntas y respuestas generales

### <a name="how-to-consume-metrics-data"></a>¿Cómo se utilizan los datos de métricas?

Los datos de métricas se almacenan como una serie de tablas de Azure en la cuenta de almacenamiento del cliente. Además, pueden utilizarse con las siguientes herramientas:

- SDK DE AMS
- El Explorador de Microsoft Azure Storage (puede exportar la información en un archivo con formato de valores separados por comas y procesarse en Excel)
- API DE REST

### <a name="how-to-find-average-bandwidth-consumption"></a>¿Cómo puedo buscar los datos de uso medio del ancho de banda?

El uso medio de ancho de banda es el promedio del valor de BytesSent un intervalo de tiempo.

### <a name="how-to-define-streaming-unit-count"></a>¿Cómo puedo definir el número de unidades de streaming?

El número de unidades de streaming se puede definir como el rendimiento máximo de los puntos de conexión de streaming del servicio dividido por el rendimiento máximo de un punto de conexión de streaming. El rendimiento máximo que puede aprovechar un punto de conexión de streaming es de 160 Mbps.
Por ejemplo, supongamos que el rendimiento máximo del servicio de un cliente es de 40 MBps el (valor máximo de BytesSent a lo largo de un intervalo de tiempo). Después, el número de unidades de streaming es igual a (40 MBps) * (8 bits/byte)/(160 Mbps) = 2 unidades de streaming.

### <a name="how-to-find-average-requestssecond"></a>¿Cómo puedo buscar el valor medio de solicitudes por segundo?

Para buscar la cantidad media de solicitudes por segundo, calcule el promedio de solicitudes (RequestCount) a lo largo de un intervalo de tiempo.

### <a name="how-to-define-channel-health"></a>¿Cómo puedo definir el estado de un canal?

El estado de un canal se puede definir como una función booleana compuesta que tiene un valor False cuando se dé una de las siguientes condiciones:

- OverlapCount > 0
- DiscontinuityCount > 0
- NonincreasingCount > 0
- UnalignedKeyFrames == True 
- UnalignedPresentationTime == True 
- UnexpectedBitrate == True


### <a name="how-to-detect-discontinuities"></a>¿Cómo puedo detectar interrupciones?

Para detectar interrupciones, busque todas las entradas de la entidad Channel en las que el valor de DiscontinuityCount sea mayor que 0. La marca de tiempo de ObservedTime correspondiente indica las horas en las que se produjeron las interrupciones.

### <a name="how-to-detect-timestamp-overlaps"></a>¿Cómo puedo detectar superposiciones de marcas de tiempo?

Para detectar superposiciones de marcas de tiempo, busque todas las entradas de datos de la entidad Channel en las que el valor de OverlapCount sea mayor que 0. La marca de tiempo de ObservedTime correspondiente indica las horas en las que se produjeron las superposiciones de marcas de tiempo.

### <a name="how-to-find-streaming-request-failures-and-reasons"></a>¿Cómo puedo buscar errores en las solicitudes de streaming y los motivos?

Para encontrar errores en las solicitudes de streaming y los motivos, busque todas las entradas de datos de la entidad StreamingEndpoint en las que el valor de ResultCode no sea igual a S_OK. El campo StatusCode correspondiente indica el motivo del error en una solicitud.

### <a name="how-to-consume-data-with-external-tools"></a>¿Cómo puedo utilizar los datos con herramientas externas?

Los datos de telemetría se pueden procesar y visualizar con las siguientes herramientas:

- PowerBI
- Application Insights
- Azure Monitor (anteriormente conocido como Shoebox)
- El panel en tiempo real de AMS
- Azure Portal (pendiente de publicación)

### <a name="how-to-manage-data-retention"></a>¿Cómo puedo administrar la retención de datos?

El sistema de telemetría no proporciona funcionalidades de administración de retención de datos ni de eliminación automática de registros antiguos. Por lo tanto, debe administrar y eliminar manualmente los registros antiguos de la tabla de almacenamiento. Puede consultar el SDK de almacenamiento para saber cómo hacerlo.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Envío de comentarios

[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
