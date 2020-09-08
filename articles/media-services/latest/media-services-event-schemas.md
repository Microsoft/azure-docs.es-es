---
title: Esquemas de Azure Event Grid para eventos de Media Services
description: Describe las propiedades que se proporcionan para los eventos de Media Services con Azure Event Grid
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: reference
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: f7c9f3b000973868c003477e58de14634b139cae
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89267673"
---
# <a name="azure-event-grid-schemas-for-media-services-events"></a>Esquemas de Azure Event Grid para eventos de Media Services

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

En este artículo se proporcionan los esquemas y las propiedades para los eventos de Media Services.

Para obtener una lista de scripts de ejemplo y los tutoriales, consulte [Orígenes de eventos de Media Services](../../event-grid/event-schema-subscriptions.md).

## <a name="job-related-event-types"></a>Tipos de eventos relacionados con el trabajo

Media Services emite los tipos de eventos relacionados con el **trabajo** que se describen a continuación. Existen dos categorías para los eventos relacionados con el **trabajo**: "supervisión de cambios de estado de trabajo" y "supervisión de cambios de estado de salida de trabajo". 

Para registrarse para todos los eventos, puede suscribirse al evento JobStateChange. O bien, puede suscribirse solo a eventos específicos (por ejemplo, estados finales como JobErrored, JobFinished y JobCanceled).   

### <a name="monitoring-job-state-changes"></a>Supervisión de cambios de estado de trabajo

| Tipo de evento | Descripción |
| ---------- | ----------- |
| Microsoft.Media.JobStateChange| Permite obtener un evento para todos los cambios de estado del trabajo. |
| Microsoft.Media.JobScheduled| Permite obtener un evento cuando el trabajo realiza la transición al estado programado. |
| Microsoft.Media.JobProcessing| Permite obtener un evento cuando el trabajo realiza la transición al estado de procesamiento en curso. |
| Microsoft.Media.JobCanceling| Permite obtener un evento cuando el trabajo realiza la transición al estado de cancelación en curso. |
| Microsoft.Media.JobFinished| Permite obtener un evento cuando el trabajo realiza la transición al estado finalizado. Se trata de un estado final que incluye las salidas del trabajo.|
| Microsoft.Media.JobCanceled| Permite obtener un evento cuando el trabajo realiza la transición al estado cancelado. Se trata de un estado final que incluye las salidas del trabajo.|
| Microsoft.Media.JobErrored| Permite obtener un evento cuando el trabajo realiza la transición al estado de error. Se trata de un estado final que incluye las salidas del trabajo.|

Consulte los [Ejemplos de esquema](#event-schema-examples) siguientes.

### <a name="monitoring-job-output-state-changes"></a>Supervisión de cambios de estado de salida de trabajo

Un trabajo puede contener varias salidas de trabajo (si ha configurado la transformación para que tenga varias salidas de trabajo) Si quiere realizar el seguimiento de los detalles de salida de un trabajo individual, escuche un evento de cambio de salida del trabajo.

Cada elemento **Job** estará en un nivel superior al de **JobOutput**, por lo que los eventos de salida de trabajo se activan dentro de un trabajo correspondiente. 

Los mensajes de error de `JobFinished`, `JobCanceled`, `JobError` muestran los resultados agregados para cada salida de trabajo, cuando todos han finalizado. Por su parte, los eventos de salida de trabajo se activan cuando finaliza cada tarea. Por ejemplo, si tiene una salida de codificación, seguida de una de análisis de vídeo, obtendría dos eventos que se desencadenan como eventos de salida de trabajo antes de que el evento JobFinished final se desencadene con los datos agregados.

| Tipo de evento | Descripción |
| ---------- | ----------- |
| Microsoft.Media.JobOutputStateChange| Permite obtener un evento para todos los cambios de estado de salida del trabajo. |
| Microsoft.Media.JobOutputScheduled| Permite obtener un evento cuando la salida del trabajo realiza la transición al estado programado. |
| Microsoft.Media.JobOutputProcessing| Permite obtener un evento cuando la salida del trabajo realiza la transición al estado de procesamiento en curso. |
| Microsoft.Media.JobOutputCanceling| Permite obtener un evento cuando la salida del trabajo realiza la transición al estado de cancelación en curso.|
| Microsoft.Media.JobOutputFinished| Permite obtener un evento cuando la salida del trabajo realiza la transición al estado finalizado.|
| Microsoft.Media.JobOutputCanceled| Permite obtener un evento cuando la salida del trabajo realiza la transición al estado cancelado.|
| Microsoft.Media.JobOutputErrored| Permite obtener un evento cuando la salida del trabajo realiza la transición al estado de error.|

Consulte los [Ejemplos de esquema](#event-schema-examples) siguientes.

### <a name="monitoring-job-output-progress"></a>Supervisión del progreso de salida del trabajo

| Tipo de evento | Descripción |
| ---------- | ----------- |
| Microsoft.Media.JobOutputProgress| Este evento refleja el progreso del procesamiento de trabajo del 0 % al 100 %. El servicio intenta enviar un evento si se ha producido un aumento del 5 % o mayor en el valor de progreso o han pasado más de 30 segundos desde el último evento (latido). No se garantiza que el valor de progreso se inicie en el 0 %, o que alcance el 100 %, ni tampoco que aumente a una velocidad constante a lo largo del tiempo. No debe utilizar este evento para determinar si se ha completado el procesamiento; en su lugar, utilice los eventos de cambio de estado.|

Consulte los [Ejemplos de esquema](#event-schema-examples) siguientes.

## <a name="live-event-types"></a>Tipos de eventos en directo

Media Services emite los tipos de eventos **en directo** que se describen a continuación. Existen dos categorías para los eventos en **directo**: eventos a nivel de transmisión y eventos a nivel de pista. 

### <a name="stream-level-events"></a>Eventos de nivel de transmisión

Los eventos de nivel de transmisión se generan en función de la transmisión o conexión. Cada evento tiene un parámetro `StreamId` que identifica la conexión o la transmisión. Cada transmisión o conexión tiene una o más pistas de diferentes tipos. Por ejemplo, una conexión de un codificador puede tener una pista de audio y cuatro pistas de vídeo. Los tipos de evento de transmisión son:

| Tipo de evento | Descripción |
| ---------- | ----------- |
| Microsoft.Media.LiveEventConnectionRejected | Se rechazó el intento de conexión del codificador. |
| Microsoft.Media.LiveEventEncoderConnected | El codificador se conectó al evento en directo. |
| Microsoft.Media.LiveEventEncoderDisconnected | El codificador se desconecta. |

Consulte los [Ejemplos de esquema](#event-schema-examples) siguientes.

### <a name="track-level-events"></a>Eventos de nivel de seguimiento

Los eventos de nivel de seguimiento se generan por pista. 

> [!NOTE]
> Todos los eventos de nivel de seguimiento se generan después de conectarse un codificador en directo.

Los tipos de evento de nivel de seguimiento son:

| Tipo de evento | Descripción |
| ---------- | ----------- |
| Microsoft.Media.LiveEventIncomingDataChunkDropped | El servidor multimedia elimina el fragmento de datos porque es demasiado tarde o porque tiene una marca de tiempo superpuesta (la marca de tiempo del nuevo fragmento de datos es menor que el tiempo de finalización del fragmento de datos anterior). |
| Microsoft.Media.LiveEventIncomingStreamReceived | El servidor multimedia recibe el primer fragmento de datos de cada pista en la transmisión o la conexión. |
| Microsoft.Media.LiveEventIncomingStreamsOutOfSync | El servidor multimedia detecta que las transmisiones de audio y vídeo no están sincronizadas. Se usa como advertencia porque la experiencia del usuario no se verá afectada. |
| Microsoft.Media.LiveEventIncomingVideoStreamsOutOfSync | El servidor multimedia detecta que cualquiera de las dos transmisiones de vídeo provenientes del codificador externo no está sincronizada. Se usa como advertencia porque la experiencia del usuario no se verá afectada. |
| Microsoft.Media.LiveEventIngestHeartbeat | Se publica cada 20 segundos para cada pista cuando se está ejecutando el evento en directo. Proporciona resumen de mantenimiento de la ingesta.<br/><br/>Después de conectarse inicialmente el codificador, el evento de latido continúa emitiéndose cada 20 seg. independientemente de si el codificador sigue conectado o no. |
| Microsoft.Media.LiveEventTrackDiscontinuityDetected | El servidor multimedia detecta una discontinuidad en la pista entrante. |

Consulte los [Ejemplos de esquema](#event-schema-examples) siguientes.

## <a name="event-schema-examples"></a>Ejemplos de esquema de eventos

### <a name="jobstatechange"></a>JobStateChange

En el ejemplo siguiente, se muestra el esquema de un evento **JobStateChange**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
    "eventType": "Microsoft.Media.JobStateChange",
    "eventTime": "2018-04-20T21:26:13.8978772",
    "id": "b9d38923-9210-4c2b-958f-0054467d4dd7",
    "data": {
      "previousState": "Processing",
      "state": "Finished"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| previousState | string | El estado del trabajo antes del evento. |
| state | string | El nuevo estado del trabajo que se notifica en este evento. Por ejemplo, "Programado: el trabajo está listo para comenzar" o "Finalizado: el trabajo ha finalizado".|

Donde el estado del trabajo puede ser uno de los valores siguientes: *Queued*, *Scheduled*, *Processing*, *Finished*, *Error*, *Canceled*, *Canceling*.

> [!NOTE]
> *Queued* solo estará presente en la propiedad **previousState** pero no en la propiedad **state**.

### <a name="jobscheduled-jobprocessing-jobcanceling"></a>JobScheduled, JobProcessing, JobCanceling

Para cada cambio de estado de trabajo no final (por ejemplo, JobScheduled, JobProcessing y JobCanceling), el esquema de ejemplo tiene un aspecto similar al siguiente:

```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "eventType": "Microsoft.Media.JobProcessing",
  "eventTime": "2018-10-12T16:12:18.0839935",
  "id": "a0a6efc8-f647-4fc2-be73-861fa25ba2db",
  "data": {
    "previousState": "Scheduled",
    "state": "Processing",
    "correlationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

### <a name="jobfinished-jobcanceled-joberrored"></a>JobFinished, JobCanceled, JobErrored

Para cada cambio de estado de trabajo final (por ejemplo, JobFinished, JobCanceled y JobErrored), el esquema de ejemplo tiene un aspecto similar al siguiente:

```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "eventType": "Microsoft.Media.JobFinished",
  "eventTime": "2018-10-12T16:25:56.4115495",
  "id": "9e07e83a-dd6e-466b-a62f-27521b216f2a",
  "data": {
    "outputs": [
      {
        "@odata.type": "#Microsoft.Media.JobOutputAsset",
        "assetName": "output-7640689F",
        "error": null,
        "label": "VideoAnalyzerPreset_0",
        "progress": 100,
        "state": "Finished"
      }
    ],
    "previousState": "Processing",
    "state": "Finished",
    "correlationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| outputs | Array | Permite obtener las salidas del trabajo.|

### <a name="joboutputstatechange"></a>JobOutputStateChange

En este ejemplo se muestra el esquema del evento **JobOutputStateChange**:

```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "eventType": "Microsoft.Media.JobOutputStateChange",
  "eventTime": "2018-10-12T16:25:56.0242854",
  "id": "dde85f46-b459-4775-b5c7-befe8e32cf90",
  "data": {
    "previousState": "Processing",
    "output": {
      "@odata.type": "#Microsoft.Media.JobOutputAsset",
      "assetName": "output-7640689F",
      "error": null,
      "label": "VideoAnalyzerPreset_0",
      "progress": 100,
      "state": "Finished"
    },
    "jobCorrelationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

### <a name="joboutputscheduled-joboutputprocessing-joboutputfinished-joboutputcanceling-joboutputcanceled-joboutputerrored"></a>JobOutputScheduled, JobOutputProcessing, JobOutputFinished, JobOutputCanceling, JobOutputCanceled, JobOutputErrored

Para cada cambio de estado JobOutput, el esquema de ejemplo tiene un aspecto similar al siguiente:

```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/<job-id>",
  "eventType": "Microsoft.Media.JobOutputProcessing",
  "eventTime": "2018-10-12T16:12:18.0061141",
  "id": "f1fd5338-1b6c-4e31-83c9-cd7c88d2aedb",
  "data": {
    "previousState": "Scheduled",
    "output": {
      "@odata.type": "#Microsoft.Media.JobOutputAsset",
      "assetName": "output-7640689F",
      "error": null,
      "label": "VideoAnalyzerPreset_0",
      "progress": 0,
      "state": "Processing"
    },
    "jobCorrelationData": {
      "testKey1": "testValue1",
      "testKey2": "testValue2"
    }
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```
### <a name="joboutputprogress"></a>JobOutputProgress

El esquema de ejemplo es similar al siguiente:

 ```json
[{
  "topic": "/subscriptions/<subscription-id>/resourceGroups/belohGroup/providers/Microsoft.Media/mediaservices/<account-name>",
  "subject": "transforms/VideoAnalyzerTransform/jobs/job-5AB6DE32",
  "eventType": "Microsoft.Media.JobOutputProgress",
  "eventTime": "2018-12-10T18:20:12.1514867",
  "id": "00000000-0000-0000-0000-000000000000",
  "data": {
    "jobCorrelationData": {
      "TestKey1": "TestValue1",
      "testKey2": "testValue2"
    },
    "label": "VideoAnalyzerPreset_0",
    "progress": 86
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}]
```

### <a name="liveeventconnectionrejected"></a>LiveEventConnectionRejected

En el ejemplo siguiente, se muestra el esquema de un evento **LiveEventConnectionRejected**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/MyLiveEvent1",
    "eventType": "Microsoft.Media.LiveEventConnectionRejected",
    "eventTime": "2018-01-16T01:57:26.005121Z",
    "id": "b303db59-d5c1-47eb-927a-3650875fded1",
    "data": { 
      "streamId":"Mystream1",
      "ingestUrl": "http://abc.ingest.isml",
      "encoderIp": "118.238.251.xxx",
      "encoderPort": 52859,
      "resultCode": "MPE_INGEST_CODEC_NOT_SUPPORTED"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| streamId | string | Identificador de la transmisión o la conexión. El codificador o el cliente es responsable de agregar este id. en la URL de introducción. |  
| ingestUrl | string | Introducir la dirección URL que proporcionó el evento en directo. |  
| encoderIp | string | Dirección IP del codificador. |
| encoderPort | string | Puerto del codificador de donde procede esta transmisión. |
| resultCode | string | El motivo por el que se rechazó la conexión. En la tabla siguiente se muestran los códigos de resultados: |

Puede encontrar los códigos de resultado de error en [Códigos de error de eventos en directo de Media Services](live-event-error-codes.md).

### <a name="liveeventencoderconnected"></a>LiveEventEncoderConnected

En el ejemplo siguiente, se muestra el esquema de un evento **LiveEventEncoderConnected**: 

```json
[
  { 
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventEncoderConnected",
    "eventTime": "2018-08-07T23:08:09.1710643",
    "id": "<id>",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml",
      "streamId": "15864-stream0",
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| streamId | string | Identificador de la transmisión o la conexión. El codificador o el cliente es responsable de proporcionar este id. en la URL de introducción. |
| ingestUrl | string | Introducir la dirección URL que proporcionó el evento en directo. |
| encoderIp | string | Dirección IP del codificador. |
| encoderPort | string | Puerto del codificador de donde procede esta transmisión. |

### <a name="liveeventencoderdisconnected"></a>LiveEventEncoderDisconnected

En el ejemplo siguiente, se muestra el esquema de un evento **LiveEventEncoderDisconnected**: 

```json
[
  { 
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventEncoderDisconnected",
    "eventTime": "2018-08-07T23:08:09.1710872",
    "id": "<id>",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml",
      "streamId": "15864-stream0",
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485",
      "resultCode": "S_OK"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| streamId | string | Identificador de la transmisión o la conexión. El codificador o el cliente es responsable de agregar este id. en la URL de introducción. |  
| ingestUrl | string | Introducir la dirección URL que proporcionó el evento en directo. |  
| encoderIp | string | Dirección IP del codificador. |
| encoderPort | string | Puerto del codificador de donde procede esta transmisión. |
| resultCode | string | La razón por la cual el codificador se desconecta. Podría ser un cierre estable o un error. En la tabla siguiente se muestran los códigos de resultados: |

Puede encontrar los códigos de resultado de error en [Códigos de error de eventos en directo de Media Services](live-event-error-codes.md).

Los códigos de resultado del cierre estable son:

| Código de resultado | Descripción |
| ----------- | ----------- |
| S_OK | El codificador se desconectó correctamente. |
| MPE_CLIENT_TERMINATED_SESSION | Codificador desconectado (RTMP). |
| MPE_CLIENT_DISCONNECTED | Codificador desconectado (FMP4). |
| MPI_REST_API_CHANNEL_RESET | Se recibió el comando de restablecimiento de canal. |
| MPI_REST_API_CHANNEL_STOP | Se recibió el comando de parada de canal. |
| MPI_REST_API_CHANNEL_STOP | Canal en mantenimiento. |
| MPI_STREAM_HIT_EOF | La transmisión EOF que envía el codificador. |

### <a name="liveeventincomingdatachunkdropped"></a>LiveEventIncomingDataChunkDropped

En el ejemplo siguiente, se muestra el esquema de un evento **LiveEventIncomingDataChunkDropped**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/MyLiveEvent1",
    "eventType": "Microsoft.Media.LiveEventIncomingDataChunkDropped",
    "eventTime": "2018-01-16T01:57:26.005121Z",
    "id": "03da9c10-fde7-48e1-80d8-49936f2c3e7d",
    "data": { 
      "trackType": "Video",
      "trackName": "Video",
      "bitrate": 300000,
      "timestamp": 36656620000,
      "timescale": 10000000,
      "resultCode": "FragmentDrop_OverlapTimestamp"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| trackType | string | Tipo de pista (audio/vídeo). |
| trackName | string | Nombre de la pista. |
| bitrate | integer | Velocidad de bits de la pista. |
| timestamp | string | Marca de tiempo del fragmento de datos proporcionado. |
| timescale | string | Escala temporal de la marca de tiempo. |
| resultCode | string | Razón del fragmento de datos proporcionado. **FragmentDrop_OverlapTimestamp** o **FragmentDrop_NonIncreasingTimestamp**. |

### <a name="liveeventincomingstreamreceived"></a>LiveEventIncomingStreamReceived

En el ejemplo siguiente, se muestra el esquema de un evento **LiveEventIncomingStreamReceived**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventIncomingStreamReceived",
    "eventTime": "2018-08-07T23:08:10.5069288Z",
    "id": "7f939a08-320c-47e7-8250-43dcfc04ab4d",
    "data": {
      "ingestUrl": "http://mle1-amsts03mediaacctgndos-ts031.channel.media.azure-test.net:80/ingest.isml/Streams(15864-stream0)15864-stream0",
      "trackType": "video",
      "trackName": "video",
      "bitrate": 2962000,
      "encoderIp": "131.107.147.xxx",
      "encoderPort": "27485",
      "timestamp": "15336831655032322",
      "duration": "20000000",
      "timescale": "10000000"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| trackType | string | Tipo de pista (audio/vídeo). |
| trackName | string | Nombre de la pista (lo puede proporcionar el codificador o, en el caso de RTMP, el servidor se genera en formato *TrackType_Bitrate*). |
| bitrate | integer | Velocidad de bits de la pista. |
| ingestUrl | string | Introducir la dirección URL que proporcionó el evento en directo. |
| encoderIp | string  | Dirección IP del codificador. |
| encoderPort | string | Puerto del codificador de donde procede esta transmisión. |
| timestamp | string | Primera marca de tiempo del fragmento de datos recibido. |
| timescale | string | Escala temporal en la que se representa la marca de tiempo. |

### <a name="liveeventincomingstreamsoutofsync"></a>LiveEventIncomingStreamsOutOfSync

En el ejemplo siguiente, se muestra el esquema de un evento **LiveEventIncomingStreamsOutOfSync**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventIncomingStreamsOutOfSync",
    "eventTime": "2018-08-10T02:26:20.6269183Z",
    "id": "b9d38923-9210-4c2b-958f-0054467d4dd7",
    "data": {
      "minLastTimestamp": "319996",
      "typeOfStreamWithMinLastTimestamp": "Audio",
      "maxLastTimestamp": "366000",
      "typeOfStreamWithMaxLastTimestamp": "Video",
      "timescaleOfMinLastTimestamp": "10000000", 
      "timescaleOfMaxLastTimestamp": "10000000"       
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| minLastTimestamp | string | Mínimo de las últimas marcas de tiempo entre todas las pistas (audio o vídeo). |
| typeOfTrackWithMinLastTimestamp | string | Tipo de pista (audio o vídeo) con la última marca de tiempo mínima. |
| maxLastTimestamp | string | Máximo de las todas las marcas de tiempo entre todas las pistas (audio o vídeo). |
| typeOfTrackWithMaxLastTimestamp | string | Tipo de pista (audio o vídeo) con la última marca de tiempo máxima. |
| timescaleOfMinLastTimestamp| string | Permite obtener la escala temporal en la que se representa “MinLastTimestamp”.|
| timescaleOfMaxLastTimestamp| string | Permite obtener la escala temporal en la que se representa “MaxLastTimestamp”.|

### <a name="liveeventincomingvideostreamsoutofsync"></a>LiveEventIncomingVideoStreamsOutOfSync

En el ejemplo siguiente, se muestra el esquema de un evento **LiveEventIncomingVideoStreamsOutOfSync**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaServices/<account-name>",
    "subject": "/LiveEvents/LiveEvent1",
    "eventType": "Microsoft.Media.LiveEventIncomingVideoStreamsOutOfSync",
    "eventTime": "2018-01-16T01:57:26.005121Z",
    "id": "6dd4d862-d442-40a0-b9f3-fc14bcf6d750",
    "data": {
      "firstTimestamp": "2162058216",
      "firstDuration": "2000",
      "secondTimestamp": "2162057216",
      "secondDuration": "2000",
      "timescale": "10000000"      
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| firstTimestamp | string | Marca de tiempo recibida para una de las pistas o niveles de calidad de tipo de vídeo. |
| firstDuration | string | Duración del fragmento de datos con la primera marca de tiempo. |
| secondTimestamp | string  | Marca de tiempo recibida para algún otro nivel de pista o calidad del tipo de vídeo. |
| secondDuration | string | Duración del fragmento de datos con la segunda marca de tiempo. |
| timescale | string | Escala de tiempo de marcas de tiempo y duración.|

### <a name="liveeventingestheartbeat"></a>LiveEventIngestHeartbeat

En el ejemplo siguiente, se muestra el esquema de un evento **LiveEventIngestHeartbeat**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventIngestHeartbeat",
    "eventTime": "2018-08-07T23:17:57.4610506",
    "id": "7f450938-491f-41e1-b06f-c6cd3965d786",
    "data": {
      "trackType": "audio",
      "trackName": "audio",
      "bitrate": 160000,
      "incomingBitrate": 155903,
      "lastTimestamp": "15336837535253637",
      "timescale": "10000000",
      "overlapCount": 0,
      "discontinuityCount": 0,
      "nonincreasingCount": 0,
      "unexpectedBitrate": false,
      "state": "Running",
      "healthy": true
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| trackType | string | Tipo de pista (audio/vídeo). |
| trackName | string | Nombre de la pista (lo puede proporcionar el codificador o, en el caso de RTMP, el servidor se genera en formato *TrackType_Bitrate*). |
| bitrate | integer | Velocidad de bits de la pista. |
| incomingBitrate | integer | Velocidad de bits calculada en función de los fragmentos de datos procedentes del codificador. |
| lastTimestamp | string | Última marca de tiempo recibida para una pista en los últimos 20 segundos. |
| timescale | string | Escala temporal en la que se expresan las marcas de tiempo. |
| overlapCount | integer | Número de fragmentos de datos que tienen marcas de tiempo superpuestas en los últimos 20 segundos. |
| discontinuityCount | integer | Número de interrupciones observada en los últimos 20 segundos. |
| nonIncreasingCount | integer | Número de fragmentos de datos con marcas de tiempo en el pasado que se recibieron en los últimos 20 segundos. |
| unexpectedBitrate | bool | Si las velocidades de bits esperadas y reales superan el límite permitido en los últimos 20 segundos. Es "true" si, y solo si, la velocidad de bits incomingBitrate >= 2* O incomingBitrate <= bitrate/2 O IncomingBitrate = 0. |
| state | string | Estado del evento en directo. |
| healthy | bool | Indica si la ingesta está en buen estado según los recuentos y las marcas. Estará en buen estado si este tiene el valor "true" y si overlapCount = 0 && discontinuityCount = 0 && nonIncreasingCount = 0 && unexpectedBitrate = false. |

### <a name="liveeventtrackdiscontinuitydetected"></a>LiveEventTrackDiscontinuityDetected

En el ejemplo siguiente, se muestra el esquema de un evento **LiveEventTrackDiscontinuityDetected**: 

```json
[
  {
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<rg-name>/providers/Microsoft.Media/mediaservices/<account-name>",
    "subject": "liveEvent/mle1",
    "eventType": "Microsoft.Media.LiveEventTrackDiscontinuityDetected",
    "eventTime": "2018-08-07T23:18:06.1270405Z",
    "id": "5f4c510d-5be7-4bef-baf0-64b828be9c9b",
    "data": {
      "trackName": "video",
      "previousTimestamp": "15336837615032322",
      "trackType": "video",
      "bitrate": 2962000,
      "newTimestamp": "15336837619774273",
      "discontinuityGap": "575284",
      "timescale": "10000000"
    },
    "dataVersion": "1.0",
    "metadataVersion": "1"
  }
]
```

El objeto data tiene las siguientes propiedades:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| trackType | string | Tipo de pista (audio/vídeo). |
| trackName | string | Nombre de la pista (lo puede proporcionar el codificador o, en el caso de RTMP, el servidor se genera en formato *TrackType_Bitrate*). |
| bitrate | integer | Velocidad de bits de la pista. |
| previousTimestamp | string | Marca de tiempo del fragmento anterior. |
| newTimestamp | string | Marca de tiempo del fragmento actual. |
| discontinuityGap | string | Intervalo entre las dos marcas de tiempo anteriores. |
| timescale | string | Escala temporal en la que se representan tanto la marca de tiempo como el intervalo de discontinuidad. |

### <a name="common-event-properties"></a>Propiedades personalizadas del evento

Un evento tiene los siguientes datos de nivel superior:

| Propiedad | Tipo | Descripción |
| -------- | ---- | ----------- |
| topic | string | El tema EventGrid. Esta propiedad tiene el id. de recurso de la cuenta de Media Services. |
| subject | string | La ruta de acceso del recurso del canal de Media Services en la cuenta de Media Services. Al concatenar los temas, obtendrá la identificación del recurso para el trabajo. |
| eventType | string | Uno de los tipos de eventos registrados para este origen de eventos. Por ejemplo, "Microsoft.Media.JobStateChange". |
| eventTime | string | La hora de generación del evento en función de la hora UTC del proveedor. |
| id | string | Identificador único para el evento |
| datos | object | Datos del evento de Media Services. |
| dataVersion | string | Versión del esquema del objeto de datos. El publicador define la versión del esquema. |
| metadataVersion | string | Versión del esquema de los metadatos del evento. Event Grid define el esquema de las propiedades de nivel superior. Event Grid proporciona este valor. |

## <a name="next-steps"></a>Pasos siguientes

[Regístrese para los eventos de cambio de estado del trabajo](job-state-events-cli-how-to.md)

## <a name="see-also"></a>Consulte también

- [EventGrid .NET SDK that includes Media Service events](https://www.nuget.org/packages/Microsoft.Azure.EventGrid/) (SDK de .NET en EventGrid que incluye eventos de Media Services)
- [Definitions of Media Services events](https://github.com/Azure/azure-rest-api-specs/blob/master/specification/eventgrid/data-plane/Microsoft.Media/stable/2018-01-01/MediaServices.json) (Definición de eventos de Media Services)
- [Códigos de error de eventos en directo de Media Services](live-event-error-codes.md)
