---
title: 'Supervisión y registro: Azure'
description: En este artículo se proporciona información general sobre la supervisión y el registro de Live Video Analytics on IoT Edge.
ms.topic: reference
ms.date: 04/27/2020
ms.openlocfilehash: 82e4a5879e4c88e462edcddb02866ec9b671d7fe
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87060462"
---
# <a name="monitoring-and-logging"></a>Supervisión y registro

En este artículo se describe cómo se pueden recibir eventos del módulo Live Video Analytics on IoT Edge para la supervisión remota. 

También encontrará información sobre cómo se pueden controlar los registros que el módulo genera.

## <a name="taxonomy-of-events"></a>Taxonomía de eventos

Live Video Analytics on IoT Edge emite eventos o datos de telemetría según la siguiente taxonomía.

![Esquema de telemetría de Live Video Analytics on IoT Edge](./media/telemetry-schema/taxonomy.png)

* Operativo: eventos que se generan como parte de las acciones realizadas por un usuario o durante la ejecución de un [gráfico multimedia](media-graph-concept.md).
   
   * Volumen: se espera que sea bajo (unas cuantas veces por minuto, o incluso una tasa inferior).
   * Ejemplos:

      Grabación iniciada (abajo), grabación detenida.
      
      ```
      {
        "body": {
          "outputType": "assetName",
          "outputLocation": "sampleAssetFromEVR-LVAEdge-20200512T233309Z"
        },
        "applicationProperties": {
          "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
          "subject": "/graphInstances/Sample-Graph-2/sinks/assetSink",
          "eventType": "Microsoft.Media.Graph.Operational.RecordingStarted",
          "eventTime": "2020-05-12T23:33:10.392Z",
          "dataVersion": "1.0"
        }
      }
      ```
* Diagnóstico: eventos que ayudan a diagnosticar problemas o incidencias de rendimiento.

   * Volumen: puede ser elevado (varias veces por minuto).
   * Ejemplos:
   
      Información de [Session Description Protocol](https://en.wikipedia.org/wiki/Session_Description_Protocol) de RTSP (abajo) o saltos en la fuente de vídeo entrante.

      ```
      {
        "body": {
          "sdp": "SDP:\nv=0\r\no=- 1589326384077235 1 IN IP4 XXX.XX.XX.XXX\r\ns=Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\ni=media/lots_015.mkv\r\nt=0 0\r\na=tool:LIVE555 Streaming Media v2020.04.12\r\na=type:broadcast\r\na=control:*\r\na=range:npt=0-73.000\r\na=x-qt-text-nam:Matroska video+audio+(optional)subtitles, streamed by the LIVE555 Media Server\r\na=x-qt-text-inf:media/lots_015.mkv\r\nm=video 0 RTP/AVP 96\r\nc=IN IP4 0.0.0.0\r\nb=AS:500\r\na=rtpmap:96 H264/90000\r\na=fmtp:96 packetization-mode=1;profile-level-id=640028;sprop-parameter-sets=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX\r\na=control:track1\r\n"
        },
        "applicationProperties": {
          "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
          "subject": "/graphInstances/Sample-Graph-2/sources/rtspSource",
          "eventType": "Microsoft.Media.Graph.Diagnostics.MediaSessionEstablished",
          "eventTime": "2020-05-12T23:33:04.077Z",
          "dataVersion": "1.0"
        }
      }
      ```
* Análisis: eventos que se generan como parte del análisis de vídeo.

   * Volumen: puede ser elevado (varias veces por minuto o más a menudo).
   * Ejemplos:
      
      Movimiento detectado (abajo), resultado de la inferencia.
   ```      
   {
     "body": {
       "timestamp": 143039375044290,
       "inferences": [
         {
           "type": "motion",
           "motion": {
             "box": {
               "l": 0.48954,
               "t": 0.140741,
               "w": 0.075,
               "h": 0.058824
             }
           }
         }
       ]
     },
     "applicationProperties": {
       "topic": "/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX/resourceGroups/<my-resource-group>/providers/microsoft.media/mediaservices/<ams-account-name>",
       "subject": "/graphInstances/Sample-Graph-2/processors/md",
       "eventType": "Microsoft.Media.Graph.Analytics.Inference",
       "eventTime": "2020-05-12T23:33:09.381Z",
       "dataVersion": "1.0"
     }
   }
   ```
Los eventos emitidos por el módulo se envían al [centro de IoT Edge](../../iot-edge/iot-edge-runtime.md#iot-edge-hub) y, desde allí, se pueden enrutar a otros destinos. 

## <a name="controlling-events"></a>Control de eventos

Se pueden usar las siguientes propiedades de módulo gemelo (como se documenta en [Esquema JSON de módulo gemelo](module-twin-configuration-schema.md)) para controlar los eventos operativos y de diagnóstico publicados por el módulo Live Video Analytics on IoT Edge.

`diagnosticsEventsOutputName` : incluya y proporcione cualquier valor en esta propiedad para obtener eventos de diagnóstico del módulo. Omítalo o déjelo en blanco si prefiere que el módulo deje de publicar eventos de diagnóstico.
   
`operationalEventsOutputName` : incluya y proporcione cualquier valor en esta propiedad para obtener eventos operativos del módulo. Omítalo o déjelo en blanco si prefiere que el módulo deje de publicar eventos operativos.
   
Los eventos de análisis los generan nodos como, por ejemplo, el procesador de detección de movimiento o el procesador de extensión HTTP, y el receptor de IoT Hub se usa para enviarlos al centro de IoT Edge. 

Puede controlar el [enrutamiento de todos los eventos anteriores](../../iot-edge/module-composition.md#declare-routes) a través de una propiedad deseada del módulo gemelo $edgeHub (en el manifiesto de implementación):

```
 "$edgeHub": {
   "properties.desired": {
     "schemaVersion": "1.0",
     "routes": {
       "moduleToHub": "FROM /messages/modules/lvaEdge/outputs/* INTO $upstream"
     },
     "storeAndForwardConfiguration": {
       "timeToLiveSecs": 7200
     }
   }
 }
```

Arriba, lvaEdge es el nombre del módulo Live Video Analytics on IoT Edge, y la regla de enrutamiento sigue el esquema definido en [Declaración de rutas](../../iot-edge/module-composition.md#declare-routes).

> [!NOTE]
> Para garantizar que los eventos de análisis llegan al centro de IoT Edge, debe haber un nodo receptor de IoT Hub en sentido descendente con respecto a cualquier nodo procesador de detección de movimiento y/o nodo procesador de extensión HTTP.

## <a name="event-schema"></a>Esquema del evento

Los eventos se originan en el dispositivo Edge y se pueden consumir tanto en Edge como en la nube. Los eventos generados por Live Video Analytics on IoT Edge siguen el [patrón de mensajería de streaming](../../iot-hub/iot-hub-devguide-messages-construct.md) establecido por Azure IoT Hub, con propiedades de sistema, propiedades de la aplicación y un cuerpo.

### <a name="summary"></a>Resumen

Cada evento, cuando se observa a través de IoT Hub, tendrá un conjunto de propiedades comunes, como se describe a continuación.

|Propiedad   |Tipo de propiedad| Tipo de datos   |Descripción|
|---|---|---|---|
|message-id |sistema |guid|  Identificador de evento único|
|topic| applicationProperty |string|    Ruta de acceso de Azure Resource Manager de la cuenta de Media Services|
|subject|   applicationProperty |string|    Subruta de acceso a la entidad que emite el evento|
|eventTime| applicationProperty|    string| Hora a la que se generó el evento|
|eventType| applicationProperty |string|    Identificador de tipo de evento (ver más abajo)|
|body|body  |object|    Datos de evento concretos|
|dataVersion    |applicationProperty|   string  |{Major}.{Minor}|

### <a name="properties"></a>Propiedades

#### <a name="message-id"></a>message-id

Es el identificador único global (GUID) del evento.

#### <a name="topic"></a>topic

Representa la cuenta de Azure Media Services asociada al gráfico.

`/subscriptions/{subId}/resourceGroups/{rgName}/providers/Microsoft.Media/mediaServices/{accountName}`

#### <a name="subject"></a>subject

Es la entidad que emite el evento:

`/graphInstances/{graphInstanceName}`<br/>
`/graphInstances/{graphInstanceName}/sources/{sourceName}`<br/>
`/graphInstances/{graphInstanceName}/processors/{processorName}`<br/>
`/graphInstances/{graphInstanceName}/sinks/{sinkName}`

La propiedad subject permite asignar eventos genéricos al módulo que lo ha generado. Por ejemplo, si el nombre de usuario o la contraseña de RTSP no son válidos, el evento generado sería `Microsoft.Media.Graph.Diagnostics.ProtocolError` en el nodo `/graphInstances/myGraph/sources/myRtspSource`.

#### <a name="event-types"></a>Tipos de eventos

Los tipos de evento se asignan a un espacio de nombres de acuerdo con el siguiente esquema:

`Microsoft.Media.Graph.{EventClass}.{EventType}`

#### <a name="event-classes"></a>Clases de eventos

|Class Name (Nombre de clase)|Descripción|
|---|---|
|Análisis  |Eventos generados como parte del análisis de contenido|
|Diagnóstico    |Eventos que ayudan a diagnosticar problemas y el rendimiento|
|Operativos    |Eventos generados como parte de las operaciones de recursos|

Los tipos de evento son específicos de cada clase de evento.

Ejemplos:

* Microsoft.Media.Graph.Analytics.Inference
* Microsoft.Media.Graph.Diagnostics.AuthorizationError
* Microsoft.Media.Graph.Operational.GraphInstanceStarted

### <a name="event-time"></a>Hora del evento

La hora del evento se describe en la cadena ISO8601 y es la hora a la que dicho evento se produjo.

## <a name="logging"></a>Registro

Al igual que sucede con otros módulos de IoT Edge, también se pueden [examinar los registros de contenedor](../../iot-edge/troubleshoot.md#check-container-logs-for-issues) en el dispositivo Edge. La información que se escribe en estos registros se puede controlar por medio de las [siguientes propiedades de módulo gemelo](module-twin-configuration-schema.md):

* logLevel

   * Los valores permitidos son Verbose, Information, Warning, Error y None.
   * El valor predeterminado es Information: los registros contendrán mensajes de error, de advertencia y de información.
   * Si el valor se establece en Warning, los registros contendrán mensajes de error y de advertencia.
   * Si el valor se establece en Error, los registros solo contendrán mensajes de error.
   * Si el valor se establece en None, no se generará ningún registro (no se recomienda).
   * La opción Verbose solo debe usarse en caso de que sea necesario compartir registros con Soporte técnico de Azure para diagnosticar un problema.
* logCategories

   * Lista separada por comas de uno o varios de los siguientes elementos: Application, Events, MediaPipeline.
   * Valor predeterminado: Application, Events.
   * Application: se trata de información de alto nivel del módulo, como mensajes de inicio del módulo, errores de entorno y llamadas a métodos directos.
   * Events: se trata de todos los eventos descritos anteriormente en este artículo.
   * MediaPipeline: se trata de una serie de registros de bajo nivel que pueden arrojar más información al solucionar problemas, como dificultades para establecer una conexión con una cámara compatible con RTSP.
   
### <a name="generating-debug-logs"></a>Generación de registros de depuración

Algunas veces, puede que tenga que generar registros más detallados que los descritos arriba para ayudar a Soporte técnico de Azure a resolver un problema. Para ello, es necesario realizar dos pasos.

En primer lugar, [vincular el almacenamiento del módulo al almacenamiento del dispositivo](../../iot-edge/how-to-access-host-storage-from-module.md#link-module-storage-to-device-storage) a través de createOptions. Si examina una [plantilla de manifiesto de implementación](https://github.com/Azure-Samples/live-video-analytics-iot-edge-csharp/blob/master/src/edge/deployment.template.json) de los inicios rápidos, verá lo siguiente:

```
"createOptions": {
   …
   "Binds": [
     "/var/local/mediaservices/:/var/lib/azuremediaservices/"
   ]
 }
```

El código anterior permite al módulo Edge escribir registros en la ruta de acceso de almacenamiento (del dispositivo) "/var/local/mediaservices/". Si agrega la siguiente propiedad deseada al módulo:

`"debugLogsDirectory": "/var/lib/azuremediaservices/debuglogs/",`

El módulo escribirá registros de depuración en formato binario en la ruta de acceso de almacenamiento (del dispositivo) /var/local/mediaservices/debuglogs/, que puede compartir con Soporte técnico de Azure.

## <a name="faq"></a>Preguntas más frecuentes

[Preguntas más frecuentes](faq.md#monitoring-and-metrics)

## <a name="next-steps"></a>Pasos siguientes

[Grabación de vídeo continua](continuous-video-recording-tutorial.md)
