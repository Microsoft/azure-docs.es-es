---
ms.openlocfilehash: 870b5b54c4a5afb39b43063bd00d3cb73e8ee0f9
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88682332"
---
Cuando se ejecuta el grafo multimedia, los resultados del nodo del procesador de detección del movimiento pasan a través del nodo de receptor de IoT Hub al centro de IoT. Los mensajes que aparecen en la ventana **SALIDA** de Visual Studio Code contienen una sección `body` y una sección `applicationProperties`. Para más información, consulte [Creación y lectura de mensajes de IoT Hub](../../../../../iot-hub/iot-hub-devguide-messages-construct.md).

En los mensajes siguientes, se definen las propiedades de la aplicación y el contenido del cuerpo mediante el módulo Live Video Analytics.

### <a name="mediasessionestablished-event"></a>Evento MediaSessionEstablished

Cuando se crean instancias del grafo multimedia, el nodo de origen RTSP intenta conectarse al servidor RTSP que se ejecuta en el contenedor rtspsim-live555. Si la conexión se realiza correctamente, se imprime el evento siguiente.

```
[IoTHubMonitor] [05:37:21 AM] Message received from [lva-sample-device/lvaEdge]:
{  
"body": {
"sdp": "SDP:\nv=0\r\no=- 1586450538111534 1 IN IP4 xxx.xxx.xxx.xxx\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/camera-300s.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.03.06\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-300.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/camera-300s.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=4D0029;sprop-parameter-sets={SPS}\r\na=control:track1\r\n"  
},  
"applicationProperties": {  
    "dataVersion": "1.0",  
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",  
    "subject": "/graphInstances/Sample-Graph-1/sources/rtspSource",  
    "eventType": "Microsoft.Media.MediaGraph.Diagnostics.MediaSessionEstablished",  
    "eventTime": "2020-05-21T05:37:21.398Z",
    }  
}
```

En la salida anterior: 

* El mensaje es un evento de diagnóstico `MediaSessionEstablished`. Indica que el nodo de origen RTSP (el sujeto) ha establecido una conexión con el simulador RTSP y ha comenzado a recibir una fuente en directo (simulada).
* En `applicationProperties`, `subject` hace referencia al nodo de la topología del grafo de elementos multimedia desde el que se generó el mensaje. En este caso, el mensaje se origina en el nodo de origen RTSP.
* En `applicationProperties`, `eventType` indica que este evento es un evento de diagnóstico.
* El valor `eventTime` es la hora en que se produjo el evento.
* La sección `body` contiene datos sobre el evento de diagnóstico. En este caso, los datos incluyen detalles sobre el [protocolo de descripción de sesiones (SDP)](https://en.wikipedia.org/wiki/Session_Description_Protocol).

### <a name="recordingstarted-event"></a>Evento RecordingStarted

Cuando se detecta movimiento, se activa el nodo de procesador de la puerta de señales y el nodo de receptor de archivos del grafo multimedia empieza a escribir un archivo MP4. El nodo del receptor de archivos envía un evento operativo. El `type` se establece en `motion` para indicar que se trata de un resultado del procesador de detección del movimiento. El valor `eventTime` es la hora UTC en la que se produjo el movimiento. Para más información sobre este proceso, consulte la sección de [Introducción](#overview) de este inicio rápido.

A continuación se muestra un ejemplo de este mensaje:

```
[IoTHubMonitor] [05:37:27 AM] Message received from [lva-sample-device/lvaEdge]:
{
  "body": {
    "outputType": "filePath",
    "outputLocation": "/var/media/sampleFilesFromEVR-filesinkOutput-20200521T053726Z.mp4"
  },
  "applicationProperties": {
    "topic": "/subscriptions/{subscriptionID}/resourceGroups/{name}/providers/microsoft.media/mediaservices/hubname",  
    "subject": "/graphInstances/Sample-Graph-1/sinks/fileSink",
    "eventType": "Microsoft.Media.Graph.Operational.RecordingStarted",
    "eventTime": "2020-05-21T05:37:27.713Z",
    "dataVersion": "1.0"
  }
}
```

En el mensaje anterior: 

* En `applicationProperties`, `subject` hace referencia al nodo del grafo multimedia desde el que se generó el mensaje. En este caso, el mensaje se origina en el nodo del receptor de archivos.
* En `applicationProperties`, `eventType` indica que este evento es operativo.
* El valor `eventTime` es la hora en que se produjo el evento. Esta hora es entre 5 y 6 segundos posterior a `MediaSessionEstablished` y posterior al inicio del flujo de vídeo. Esta hora se corresponde a la marca de 5 o 6 segundos cuando el [coche empezó a moverse](#review-the-sample-video) por el aparcamiento.
* La sección `body` contiene datos sobre el evento operativo. En este caso, los datos incluyen `outputType` y `outputLocation`.
* La variable `outputType` indica que esta información es sobre la ruta de acceso del archivo.
* El valor `outputLocation` es la ubicación del archivo MP4 en el módulo de Edge.

### <a name="recordingstopped-and-recordingavailable-events"></a>Eventos RecordingStopped y RecordingAvailable

Si examina las propiedades del nodo de procesador de la puerta de señales en la [topología del grafo](https://github.com/Azure/live-video-analytics/blob/master/MediaGraph/topologies/evr-motion-files/topology.json) puede ver que los tiempos de activación se han establecido en 5 segundos. Por tanto, unos 5 segundos después de que se reciba el evento `RecordingStarted`, obtendrá:

* Un evento `RecordingStopped`, que indica que la grabación se ha detenido.
* Un evento `RecordingAvailable`, que indica que ahora se puede usar el archivo MP4 para su visualización.

Normalmente, los dos eventos se emiten con unos segundos de diferencia.
