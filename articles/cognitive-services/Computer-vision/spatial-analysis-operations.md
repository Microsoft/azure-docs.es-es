---
title: Operaciones de análisis espacial
titleSuffix: Azure Cognitive Services
description: Las operaciones de análisis espacial.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: aahi
ms.openlocfilehash: 5955bc35acfaf8f877e68db083871c353a3ce326
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90984919"
---
# <a name="spatial-analysis-operations"></a>Operaciones de análisis espacial

El análisis espacial permite analizar vídeo de streaming en tiempo real para dispositivos de cámara. Para cada dispositivo de cámara que configure, las operaciones para el análisis espacial generarán un flujo de salida de mensajes JSON enviados a la instancia de Azure IoT Hub. 

El contenedor de análisis espacial implementa las operaciones siguientes:

| Identificador de operación| Descripción|
|---------|---------|
| cognitiveservices.vision.spatialanalysis-personcount | Cuenta a las personas que se encuentran en una zona designada del campo visual de la cámara. <br> Emite un evento _personCountEvent_ inicial y, luego, eventos _personCountEvent_ cuando el recuento cambia.  |
| cognitiveservices.vision.spatialanalysis-personcrossingline | Hace un seguimiento de cuando una persona cruza una línea designada en el campo visual de la cámara. <br>Emite un evento _personLineEvent_ cuando la persona cruza la línea y proporciona información direccional. 
| cognitiveservices.vision.spatialanalysis-personcrossingpolygon | Hace un seguimiento de cuando una persona cruza una línea designada en el campo visual de la cámara. <br> Emite un evento _personLineEvent_ cuando la persona cruza la zona y proporciona información direccional. |
| cognitiveservices.vision.spatialanalysis-persondistance | Hace un seguimiento de cuando las personas infringen una regla de distancia. <br> Emite un evento _personDistanceEvent_ de manera periódica con la ubicación de cada infracción de distancia. |

Todas las operaciones anteriores también están disponibles en la versión `.debug`, que tienen la capacidad de visualizar los fotogramas de vídeo a medida que se procesan. Deberá ejecutar `xhost +` en el equipo host para habilitar la visualización de los eventos y los fotogramas de vídeo.

| Identificador de operación| Descripción|
|---------|---------|
| cognitiveservices.vision.spatialanalysis-personcount.debug | Cuenta a las personas que se encuentran en una zona designada del campo visual de la cámara. <br> Emite un evento _personCountEvent_ inicial y, luego, eventos _personCountEvent_ cuando el recuento cambia.  |
| cognitiveservices.vision.spatialanalysis-personcrossingline.debug | Hace un seguimiento de cuando una persona cruza una línea designada en el campo visual de la cámara. <br>Emite un evento _personLineEvent_ cuando la persona cruza la línea y proporciona información direccional. 
| cognitiveservices.vision.spatialanalysis-personcrossingpolygon.debug | Hace un seguimiento de cuando una persona cruza una línea designada en el campo visual de la cámara. <br> Emite un evento _personLineEvent_ cuando la persona cruza la zona y proporciona información direccional. |
| cognitiveservices.vision.spatialanalysis-persondistance.debug | Hace un seguimiento de cuando las personas infringen una regla de distancia. <br> Emite un evento _personDistanceEvent_ de manera periódica con la ubicación de cada infracción de distancia. |

El análisis espacial también se puede ejecutar con [Live Video Analytics](https://azure.microsoft.com/services/media-services/live-video-analytics/) como su módulo de IA de vídeo. 

<!--more details on the setup can be found in the [LVA Setup page](LVA-Setup.md). Below is the list of the operations supported with Live Video Analytics. -->

| Identificador de operación| Descripción|
|---------|---------|
| cognitiveservices.vision.spatialanalysis-personcount.livevideoanalytics | Cuenta a las personas que se encuentran en una zona designada del campo visual de la cámara. <br> Emite un evento _personCountEvent_ inicial y, luego, eventos _personCountEvent_ cuando el recuento cambia.  |
| cognitiveservices.vision.spatialanalysis-personcrossingline.livevideoanalytics | Hace un seguimiento de cuando una persona cruza una línea designada en el campo visual de la cámara. <br>Emite un evento _personLineEvent_ cuando la persona cruza la línea y proporciona información direccional. 
| cognitiveservices.vision.spatialanalysis-personcrossingpolygon.livevideoanalytics | Hace un seguimiento de cuando una persona cruza una línea designada en el campo visual de la cámara. <br> Emite un evento _personLineEvent_ cuando la persona cruza la zona y proporciona información direccional. |
| cognitiveservices.vision.spatialanalysis-persondistance.livevideoanalytics | Hace un seguimiento de cuando las personas infringen una regla de distancia. <br> Emite un evento _personDistanceEvent_ de manera periódica con la ubicación de cada infracción de distancia. |

Las operaciones de Live Video Analytics también están disponibles en la versión `.debug` (por ejemplo, cognitiveservices.vision.spatialanalysis-personcount.livevideoanalytics.debug), que tiene la capacidad de visualizar los fotogramas de vídeo a medida que se procesan. Deberá ejecutar `xhost +` en el equipo host para habilitar la visualización de los eventos y los fotogramas de vídeo.

> [!IMPORTANT]
> Los modelos de IA de Computer Vision detectan y localizan la presencia humana en salida y secuencias de vídeo con un rectángulo de selección alrededor de un cuerpo humano. Los modelos de IA no intentan detectar caras ni las identidades o datos demográficos de los individuos.

Estos son los parámetros necesarios para cada una de estas operaciones de análisis espacial.

| Parámetros de la operación.| Descripción|
|---------|---------|
| Id. de operación | Identificador de operación de la tabla anterior.|
| enabled | Booleano: true o false|
| VIDEO_URL| Dirección URL de RTSP del dispositivo de cámara (ejemplo: `rtsp://username:password@url`). El análisis espacial admite la secuencia con codificación H.264 a través de RTSP, HTTP o mp4 |
| VIDEO_SOURCE_ID | Nombre descriptivo para el dispositivo de cámara o la secuencia de vídeo. Se devolverá con la salida JSON del evento.|
| VIDEO_IS_LIVE| True para dispositivos de cámara; false para vídeos grabados.|
| VIDEO_DECODE_GPU_INDEX| GPU con la que se va a decodificar el fotograma de vídeo. De manera predeterminada, es 0. Esta información debe corresponder a la misma de `gpu_index` en otra configuración de nodo, como `VICA_NODE_CONFIG`, `DETECTOR_NODE_CONFIG`.|
| DETECTOR_NODE_CONFIG | JSON que indica en qué GPU se va a ejecutar el nodo detector. Debe tener el formato siguiente: `"{ \"gpu_index\": 0 }",`|
| SPACEANALYTICS_CONFIG | Configuración JSON para la zona y la línea tal como se describe a continuación.|

### <a name="zone-configuration-for-cognitiveservicesvisionspatialanalysis-personcount"></a>Configuración de zona para cognitiveservices.vision.spatialanalysis-personcount

 Ejemplo de una entrada JSON para el parámetro SPACEANALYTICS_CONFIG que configura una zona. Puede configurar varias zonas para esta operación.

```json
{
"zones":[{
    "name": "lobbycamera"
    "polygon": [[0.3,0.3], [0.3,0.9], [0.6,0.9], [0.6,0.3], [0.3,0.3]],
    "threshold": 50.00,
    "events":[{
        "type": "count",
        "config":{
            "trigger": "event",
            "output_frequency": 1
      }
    }]
}
```

| Nombre | Tipo| Descripción|
|---------|---------|---------|
| `zones` | list| Lista de zonas. |
| `name` | string| Nombre descriptivo de esta zona.|
| `polygon` | list| Cada par de valores representa la x e y de los vértices de un polígono. El polígono representa las áreas en las que se hace un seguimiento o un recuento de las personas y los puntos de polígono se basan en coordenadas normalizadas (0-1), donde la esquina superior izquierda es (0.0, 0.0) y la esquina inferior derecha es (1.0, 1.0).   
| `threshold` | FLOAT| Los eventos se generan cuando la confianza de los modelos de IA es mayor o igual que este valor. |
| `type` | string| Para **cognitiveservices.vision.spatialanalysis-personcount**, debe ser `count`.|
| `trigger` | string| Tipo de desencadenador para enviar un evento. Los valores admitidos son `event` para enviar eventos cuando el recuento cambia o `interval` para enviar eventos de manera periódica, independientemente de si el recuento ha cambiado o no.
| `interval` | string| Tiempo en segundos que se agregará al recuento de personas antes de que se active un evento. La operación seguirá analizando la escena a una velocidad constante y devolverá el recuento más común en ese intervalo. El intervalo de agregación se aplica tanto a `event` como a `interval`.|

### <a name="line-configuration-for-cognitiveservicesvisionspatialanalysis-personcrossingline"></a>Configuración de línea para cognitiveservices.vision.spatialanalysis-personcrossingline

Ejemplo de una entrada JSON para el parámetro SPACEANALYTICS_CONFIG que configura una línea. Puede configurar varias líneas de cruce para esta operación.

```json
{
"lines":[{
    "name": "doorcamera" 
    "line": {
        "start": {"x": 0, "y": 0.5},
        "end": {"x": 1, "y": 0.5}
            },
    "threshold": 50.00,
    "events":[{
        "type": "linecrossing",
        "config":{
            "trigger": "event"
            }
        }]
    }]
}
```

| Nombre | Tipo| Descripción|
|---------|---------|---------|
| `lines` | list| Lista de líneas.|
| `name` | string| Nombre descriptivo de esta línea.|
| `line` | list| Definición de la línea. Se trata de una línea direccional que le permite comprender "entrada" frente a "salida".|
| `start` | value pair| Coordenadas x, y para el punto inicial de la línea. Los valores flotantes representan la posición del vértice relativa a la esquina superior izquierda. Para calcular los valores x, y absolutos, debe multiplicar estos valores por el tamaño del fotograma. |
| `end` | value pair| Coordenadas x, y para el punto final de la línea. Los valores flotantes representan la posición del vértice relativa a la esquina superior izquierda. Para calcular los valores x, y absolutos, debe multiplicar estos valores por el tamaño del fotograma. |
| `threshold` | FLOAT| Los eventos se generan cuando la confianza de los modelos de IA es mayor o igual que este valor. |
| `type` | string| Para **cognitiveservices.vision.spatialanalysis-personcrossingline**, debe ser `linecrossing`.|
|`trigger`|string|Tipo de desencadenador para enviar un evento.<br>Valores admitidos: "event": se activa cuando alguna persona cruza la línea.|

### <a name="zone-configuration-for-cognitiveservicesvisionspatialanalysis-personcrossingpolygon"></a>Configuración de zona para cognitiveservices.vision.spatialanalysis-personcrossingpolygon

Ejemplo de una entrada JSON para el parámetro SPACEANALYTICS_CONFIG que configura una zona. Puede configurar varias zonas para esta operación.

 ```json
{
"zones":[{
    "name": "queuecamera"
    "polygon": [[0.3,0.3], [0.3,0.9], [0.6,0.9], [0.6,0.3], [0.3,0.3]],
    "threshold": 50.00,
    "events":[{
        "type": "zone_crossing",
        "config":{
            "trigger": "event"
            }
        }]
    }]
}
```

| Nombre | Tipo| Descripción|
|---------|---------|---------|
| `zones` | list| Lista de zonas. |
| `name` | string| Nombre descriptivo de esta zona.|
| `polygon` | list| Cada par de valores representa la x e y de los vértices del polígono. El polígono representa las áreas en las que se realiza el seguimiento o el recuento de personas. Los valores flotantes representan la posición del vértice relativa a la esquina superior izquierda. Para calcular los valores x, y absolutos, debe multiplicar estos valores por el tamaño del fotograma. 
| `threshold` | FLOAT| Los eventos se generan cuando la confianza de los modelos de IA es mayor o igual que este valor. |
| `type` | string| Para **cognitiveservices.vision.spatialanalysis-personcrossingpolygon**, debe ser `enter` o `exit`.|
| `trigger`|string|Tipo de desencadenador para enviar un evento.<br>Valores admitidos: "event": se activa cuando alguna persona entra o sale de la zona.|

### <a name="zone-configuration-for-cognitiveservicesvisionspatialanalysis-persondistance"></a>Configuración de zona para cognitiveservices.vision.spatialanalysis-persondistance

Este es un ejemplo de una entrada JSON para el parámetro SPACEANALYTICS_CONFIG que configura una zona para **cognitiveservices.vision.spatialanalysis-persondistance**. Puede configurar varias zonas para esta operación.

```json
{
"zones":[{
    "name": "lobbycamera",
    "polygon": [[0.3,0.3], [0.3,0.9], [0.6,0.9], [0.6,0.3], [0.3,0.3]],
    "threshold": 35.00,
    "events":[{
        "type": "persondistance",
        "config":{
            "trigger": "event",
            "output_frequency":1,
            "minimum_distance_threshold":6.0,
            "maximum_distance_threshold":35.0
            }
        }]
    }]
}
```

| Nombre | Tipo| Descripción|
|---------|---------|---------|
| `zones` | list| Lista de zonas. |
| `name` | string| Nombre descriptivo de esta zona.|
| `polygon` | list| Cada par de valores representa la x e y de los vértices del polígono. El polígono representa las áreas en las que se cuentan las personas y se mide la distancia entre ellas. Los valores flotantes representan la posición del vértice relativa a la esquina superior izquierda. Para calcular los valores x, y absolutos, debe multiplicar estos valores por el tamaño del fotograma. 
| `threshold` | FLOAT| Los eventos se generan cuando la confianza de los modelos de IA es mayor o igual que este valor. |
| `type` | string| Para **cognitiveservices.vision.spatialanalysis-persondistance**, debe ser `people_distance`.|
| `trigger` | string| Tipo de desencadenador para enviar un evento. Los valores admitidos son `event` para enviar eventos cuando el recuento cambia o `interval` para enviar eventos de manera periódica, independientemente de si el recuento ha cambiado o no.
| `interval` | string | Tiempo en segundos que se agregará a las infracciones antes de que se active un evento. El intervalo de agregación se aplica tanto a `event` como a `interval`.|
| `output_frequency` | int | Velocidad a la que se generan los eventos. Cuando `output_frequency` = X, se genera cada X evento, por ejemplo, `output_frequency` = 2 significa que se genera evento por medio. La frecuencia output_frequency se aplica tanto a `event` como a `interval`.|
| `minimum_distance_threshold` | FLOAT| Distancia en pies que desencadenará un evento "TooClose" cuando las personas estén más cerca de esa distancia entre sí.|
| `maximum_distance_threshold` | FLOAT| Distancia en pies que desencadenará un evento "TooFar" cuando las personas estén más lejos de esa distancia entre sí.|

Este es un ejemplo de una entrada JSON para el parámetro DETECTOR_NODE_CONFIG que configura una zona para **cognitiveservices.vision.spatialanalysis-persondistance**.

```json
{ 
"gpu_index": 0, 
"do_calibration": true
}
```

| Nombre | Tipo| Descripción|
|---------|---------|---------|
| `gpu_index` | string| Índice de la GPU en que se ejecutará esta operación.|
| `do_calibration` | string | Indica que la calibración está activada. `do_calibration` debe ser true para que **cognitiveservices.vision.spatialanalysis-persondistance** funcione correctamente.|

Consulte las instrucciones para la [selección de ubicación de la cámara](spatial-analysis-camera-placement.md) para información sobre las configuraciones de zonas y líneas.

## <a name="spatial-analysis-operation-output"></a>Salida de las operaciones de análisis espacial

Los eventos de cada operación se emiten a Azure IoT Hub en formato JSON.

### <a name="json-format-for-cognitiveservicesvisionspatialanalysis-personcount-ai-insights"></a>Formato JSON para Conclusiones de IA de cognitiveservices.vision.spatialanalysis-personcount

JSON de ejemplo de un evento generado por esta operación.

```json
{
    "events": [
        {
            "id": "b013c2059577418caa826844223bb50b",
            "type": "personCountEvent",
            "detectionIds": [
                "bc796b0fc2534bc59f13138af3dd7027",
                "60add228e5274158897c135905b5a019"
            ],
            "properties": {
                "personCount": 2
            },
            "zone": "lobbycamera",
            "trigger": "event"
        }
    ],
    "sourceInfo": {
        "id": "camera_id",
        "timestamp": "2020-08-24T06:06:57.224Z",
        "width": 608,
        "height": 342,
        "frameId": "1400",
        "cameraCalibrationInfo": {
            "status": "Complete",
            "cameraHeight": 10.306597709655762,
            "focalLength": 385.3199462890625,
            "tiltupAngle": 1.0969393253326416
        },
        "imagePath": ""
    },
    "detections": [
        {
            "type": "person",
            "id": "bc796b0fc2534bc59f13138af3dd7027",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.612683747944079,
                        "y": 0.25340268765276636
                    },
                    {
                        "x": 0.7185954043739721,
                        "y": 0.6425260577285499
                    }
                ]
            },
            "confidence": 0.9559211134910583,
            "centerGroundPoint": {
                "x": 0.0,
                "y": 0.0
            },
            "metadataType": ""
        },
        {
            "type": "person",
            "id": "60add228e5274158897c135905b5a019",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.22326200886776573,
                        "y": 0.17830915618361087
                    },
                    {
                        "x": 0.34922296122500773,
                        "y": 0.6297955429344847
                    }
                ]
            },
            "confidence": 0.9389744400978088,
            "centerGroundPoint": {
                "x": 0.0,
                "y": 0.0
            },
            "metadataType": ""
        }
    ],
    "schemaVersion": "1.0"
}
```

| Nombre de campo de evento | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de evento|
| `type` | string| Tipo de evento|
| `detectionsId` | array| Matriz de tamaño 1 del identificador único de la detección de personas que desencadenó este evento|
| `properties` | collection| Colección de valores|
| `trackinId` | string| Identificador único de la persona detectada|
| `status` | string| "Enter" o "Exit"|
| `side` | int| Número del lado del polígono que cruzó la persona|
| `zone` | string | Campo "name" del polígono que representa la zona que se cruzó|
| `trigger` | string| El tipo de desencadenador es "event" o "interval" en función del valor de `trigger` en SPACEANALYTICS_CONFIG|

| Nombre del campo de detecciones | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de detección|
| `type` | string| Tipo de detección|
| `region` | collection| Colección de valores|
| `type` | string| Tipo de región|
| `points` | collection| Puntos superior izquierdo e inferior derecho cuando el tipo de región es RECTANGLE (RECTÁNGULO) |
| `confidence` | FLOAT| Confianza del algoritmo|

| Nombre del campo SourceInfo | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Camera ID (Identificador de la cámara)|
| `timestamp` | Fecha| Fecha UTC cuando se emitió la carga de JSON|
| `width` | int | Ancho del fotograma de vídeo|
| `height` | int | Alto del fotograma de vídeo|
| `frameId` | int | Identificador del fotograma|
| `cameraCallibrationInfo` | collection | Colección de valores|
| `status` | string | Indica si la calibración de la cámara en el plano de tierra está "Complete" (Completa)|
| `cameraHeight` | FLOAT | Alto de la cámara sobre el suelo en pies. Se deduce a partir de la calibración automática. |
| `focalLength` | FLOAT | Longitud focal de la cámara en píxeles. Se deduce a partir de la calibración automática. |
| `tiltUpAngle` | FLOAT | Ángulo de inclinación de la cámara desde el vertical. Se deduce a partir de la calibración automática.|

| Nombre del campo SourceInfo | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Camera ID (Identificador de la cámara)|
| `timestamp` | Fecha| Fecha UTC cuando se emitió la carga de JSON|
| `width` | int | Ancho del fotograma de vídeo|
| `height` | int | Alto del fotograma de vídeo|
| `frameId` | int | Identificador del fotograma|


### <a name="json-format-for-cognitiveservicesvisionspatialanalysis-personcrossingline-ai-insights"></a>Formato JSON para Conclusiones de IA de cognitiveservices.vision.spatialanalysis-personcrossingline

Ejemplo de JSON para las detecciones generadas por esta operación.

```json
{
    "events": [
        {
            "id": "3733eb36935e4d73800a9cf36185d5a2",
            "type": "personLineEvent",
            "detectionIds": [
                "90d55bfc64c54bfd98226697ad8445ca"
            ],
            "properties": {
                "trackingId": "90d55bfc64c54bfd98226697ad8445ca",
                "status": "CrossLeft"
            },
            "zone": "doorcamera"
        }
    ],
    "sourceInfo": {
        "id": "camera_id",
        "timestamp": "2020-08-24T06:06:53.261Z",
        "width": 608,
        "height": 342,
        "frameId": "1340",
        "imagePath": ""
    },
    "detections": [
        {
            "type": "person",
            "id": "90d55bfc64c54bfd98226697ad8445ca",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.491627341822574,
                        "y": 0.2385801348769874
                    },
                    {
                        "x": 0.588894994635331,
                        "y": 0.6395559924387793
                    }
                ]
            },
            "confidence": 0.9005028605461121,
            "metadataType": ""
        }
    ],
    "schemaVersion": "1.0"
}
```
| Nombre de campo de evento | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de evento|
| `type` | string| Tipo de evento|
| `detectionsId` | array| Matriz de tamaño 1 del identificador único de la detección de personas que desencadenó este evento|
| `properties` | collection| Colección de valores|
| `trackinId` | string| Identificador único de la persona detectada|
| `status` | string| Dirección de los cruces de líneas, ya sea "CrossLeft" o "CrossRight"|
| `zone` | string | Campo "name" de la línea que se cruzó|

| Nombre del campo de detecciones | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de detección|
| `type` | string| Tipo de detección|
| `region` | collection| Colección de valores|
| `type` | string| Tipo de región|
| `points` | collection| Puntos superior izquierdo e inferior derecho cuando el tipo de región es RECTANGLE (RECTÁNGULO) |
| `confidence` | FLOAT| Confianza del algoritmo|

| Nombre del campo SourceInfo | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Camera ID (Identificador de la cámara)|
| `timestamp` | Fecha| Fecha UTC cuando se emitió la carga de JSON|
| `width` | int | Ancho del fotograma de vídeo|
| `height` | int | Alto del fotograma de vídeo|
| `frameId` | int | Identificador del fotograma|


> [!IMPORTANT]
> El modelo de IA detecta a una persona independientemente de si el rostro de la persona está frente a la cámara o no. El modelo de IA no ejecuta la detección ni el reconocimiento de caras y no emite información biométrica alguna. 

### <a name="json-format-for-cognitiveservicesvisionspatialanalysis-personcrossingpolygon-ai-insights"></a>Formato JSON para Conclusiones de IA de cognitiveservices.vision.spatialanalysis-personcrossingpolygon

Ejemplo de JSON para las detecciones generadas por esta operación.

```json
{
    "events": [
        {
            "id": "f095d6fe8cfb4ffaa8c934882fb257a5",
            "type": "personZoneEvent",
            "detectionIds": [
                "afcc2e2a32a6480288e24381f9c5d00e"
            ],
            "properties": {
                "trackingId": "afcc2e2a32a6480288e24381f9c5d00e",
                "status": "Enter",
                "side": ""
            },
            "zone": "queuecamera"
        }
    ],
    "sourceInfo": {
        "id": "camera_id",
        "timestamp": "2020-08-24T06:15:09.680Z",
        "width": 608,
        "height": 342,
        "frameId": "428",
        "imagePath": ""
    },
    "detections": [
        {
            "type": "person",
            "id": "afcc2e2a32a6480288e24381f9c5d00e",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.8135572734631991,
                        "y": 0.6653949670624315
                    },
                    {
                        "x": 0.9937645761590255,
                        "y": 0.9925406829655519
                    }
                ]
            },
            "confidence": 0.6267998814582825,
            "metadataType": ""
        }
    ],
    "schemaVersion": "1.0"
}
```

| Nombre de campo de evento | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de evento|
| `type` | string| Tipo de evento|
| `detectionsId` | array| Matriz de tamaño 1 del identificador único de la detección de personas que desencadenó este evento|
| `properties` | collection| Colección de valores|
| `trackinId` | string| Identificador único de la persona detectada|
| `status` | string| Dirección de los cruces de los polígonos, ya sea "Enter" "Exit"|
| `zone` | string | Campo "name" del polígono que representa la zona que se cruzó|

| Nombre del campo de detecciones | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de detección|
| `type` | string| Tipo de detección|
| `region` | collection| Colección de valores|
| `type` | string| Tipo de región|
| `points` | collection| Puntos superior izquierdo e inferior derecho cuando el tipo de región es RECTANGLE (RECTÁNGULO) |
| `confidence` | FLOAT| Confianza del algoritmo|

### <a name="json-format-for-cognitiveservicesvisionspatialanalysis-persondistance-ai-insights"></a>Formato JSON para Conclusiones de IA de cognitiveservices.vision.spatialanalysis-persondistance

Ejemplo de JSON para las detecciones generadas por esta operación.

```json
{
    "events": [
        {
            "id": "9c15619926ef417aa93c1faf00717d36",
            "type": "personDistanceEvent",
            "detectionIds": [
                "9037c65fa3b74070869ee5110fcd23ca",
                "7ad7f43fd1a64971ae1a30dbeeffc38a"
            ],
            "properties": {
                "personCount": 5,
                "averageDistance": 20.807043981552123,
                "minimumDistanceThreshold": 6.0,
                "maximumDistanceThreshold": "Infinity",
                "eventName": "TooClose",
                "distanceViolationPersonCount": 2
            },
            "zone": "lobbycamera",
            "trigger": "event"
        }
    ],
    "sourceInfo": {
        "id": "camera_id",
        "timestamp": "2020-08-24T06:17:25.309Z",
        "width": 608,
        "height": 342,
        "frameId": "1199",
        "cameraCalibrationInfo": {
            "status": "Complete",
            "cameraHeight": 12.9940824508667,
            "focalLength": 401.2800598144531,
            "tiltupAngle": 1.057669997215271
        },
        "imagePath": ""
    },
    "detections": [
        {
            "type": "person",
            "id": "9037c65fa3b74070869ee5110fcd23ca",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.39988183975219727,
                        "y": 0.2719132942065858
                    },
                    {
                        "x": 0.5051516984638414,
                        "y": 0.6488402517218339
                    }
                ]
            },
            "confidence": 0.948630690574646,
            "centerGroundPoint": {
                "x": -1.4638760089874268,
                "y": 18.29732322692871
            },
            "metadataType": ""
        },
        {
            "type": "person",
            "id": "7ad7f43fd1a64971ae1a30dbeeffc38a",
            "region": {
                "type": "RECTANGLE",
                "points": [
                    {
                        "x": 0.5200299714740954,
                        "y": 0.2875368218672903
                    },
                    {
                        "x": 0.6457497446160567,
                        "y": 0.6183311060855263
                    }
                ]
            },
            "confidence": 0.8235412240028381,
            "centerGroundPoint": {
                "x": 2.6310102939605713,
                "y": 18.635927200317383
            },
            "metadataType": ""
        }
    ],
    "schemaVersion": "1.0"
}
```

| Nombre de campo de evento | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de evento|
| `type` | string| Tipo de evento|
| `detectionsId` | array| Matriz de tamaño 1 del identificador único de la detección de personas que desencadenó este evento|
| `properties` | collection| Colección de valores|
| `personCount` | int| Número de personas detectadas cuando se emitió el evento|
| `averageDistance` | FLOAT| Distancia promedio en pies entre todas las personas detectadas|
| `minimumDistanceThreshold` | FLOAT| Distancia en pies que desencadenará un evento "TooClose" cuando las personas estén más cerca de esa distancia entre sí.|
| `maximumDistanceThreshold` | FLOAT| Distancia en pies que desencadenará un evento "TooFar" cuando las personas estén más lejos de esa distancia entre sí.|
| `eventName` | string| El nombre de evento es `TooClose` cuando se infringe `minimumDistanceThreshold`, `TooFar` cuando se infringe `maximumDistanceThreshold` o `unknown` cuando no se ha completado la calibración automática|
| `distanceViolationPersonCount` | int| Número de personas detectadas en infracción de `minimumDistanceThreshold` o `maximumDistanceThreshold`|
| `zone` | string | Campo "name" del polígono que representa la zona en la que se supervisó la distancia entre las personas|
| `trigger` | string| El tipo de desencadenador es "event" o "interval" en función del valor de `trigger` en SPACEANALYTICS_CONFIG|

| Nombre del campo de detecciones | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Id. de detección|
| `type` | string| Tipo de detección|
| `region` | collection| Colección de valores|
| `type` | string| Tipo de región|
| `points` | collection| Puntos superior izquierdo e inferior derecho cuando el tipo de región es RECTANGLE (RECTÁNGULO) |
| `confidence` | FLOAT| Confianza del algoritmo|
| `centerGroundPoint` | 2 valores flotantes| Valores `x`, `y` con las coordenadas de la ubicación deducida de la persona en el suelo, en pies. `x` es la distancia desde la cámara perpendicular al plano de la imagen de cámara que se proyecta en el suelo, en pies. `y` es la distancia desde la cámara paralela al plano de la imagen que se proyecta en el suelo, en pies.|

| Nombre del campo SourceInfo | Tipo| Descripción|
|---------|---------|---------|
| `id` | string| Camera ID (Identificador de la cámara)|
| `timestamp` | Fecha| Fecha UTC cuando se emitió la carga de JSON|
| `width` | int | Ancho del fotograma de vídeo|
| `height` | int | Alto del fotograma de vídeo|
| `frameId` | int | Identificador del fotograma|
| `cameraCallibrationInfo` | collection | Colección de valores|
| `status` | string | Indica si la calibración de la cámara en el plano de tierra está "Complete" (Completa)|
| `cameraHeight` | FLOAT | Alto de la cámara sobre el suelo en pies. Se deduce a partir de la calibración automática. |
| `focalLength` | FLOAT | Longitud focal de la cámara en píxeles. Se deduce a partir de la calibración automática. |
| `tiltUpAngle` | FLOAT | Ángulo de inclinación de la cámara desde el vertical. Se deduce a partir de la calibración automática.|


## <a name="use-the-output-generated-by-the-container"></a>Uso de la salida generada por el contenedor

Puede que quiera integrar eventos o la detección del análisis espacial en la aplicación. Estos son algunos de los enfoques que se deben tener en cuenta: 

* Use el SDK del Centro de eventos de Azure del lenguaje de programación elegido para conectarse al punto de conexión de Azure IoT Hub y recibir los eventos. Para más información, consulte [Leer mensajes del dispositivo a la nube desde el punto de conexión integrado](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-read-builtin). 
* Configure **Enrutamiento de mensajes** en la instancia de Azure IoT Hub para enviar los eventos a otros puntos de conexión o guardar los eventos en el almacenamiento de datos. Para más información, consulte [Enrutamiento de mensajes de IoT Hub](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-d2c). 
* Configure un trabajo de Azure Stream Analytics para procesar los eventos en tiempo real a medida que llegan y crear visualizaciones. 

## <a name="deploying-spatial-analysis-operations-at-scale-multiple-cameras"></a>Implementación de operaciones de análisis espacial a escala (varias cámaras)

Con el fin de obtener el mejor rendimiento y el uso de las GPU, puede implementar cualquier operación de análisis espacial en varias cámaras mediante instancias de grafos. A continuación se muestra un ejemplo de ejecución de la operación cognitiveservices.vision.spatialanalysis-personcount en cinco (5) cámaras.

```json
 "properties.desired": {
      "globalSettings": {
          "PlatformTelemetryEnabled": false,
          "CustomerTelemetryEnabled": true
      },
      "graphs": {
          "personcount": {
              "operationId": "cognitiveservices.vision.spatialanalysis-personcount",
              "version": 1,
              "enabled": true,
              "sharedNodes": {
                  "shared_detector1": {
                      "node": "PersonCountGraph.detector",
                      "parameters": {
                          "DETECTOR_NODE_CONFIG": "{ \"gpu_index\": 0, \"batch_size\": 5}",
                      }
                  }
              },
              "parameters": {
                  "VIDEO_DECODE_GPU_INDEX": 0,
                  "VIDEO_IS_LIVE": true
              },
              "instances": {
                  "1": {
                      "sharedNodeMap": {
                          "PersonCountGraph/detector": "shared_detector1"
                      },
                      "parameters": {
                          "VIDEO_URL": "<Replace RTSP URL for camera 1>",
                          "VIDEO_SOURCE_ID": "camera 1",
                          "SPACEANALYTICS_CONFIG": "{\"zones\":[{\"name\":\"zone5\",\"polygon\":[[0,0],[1,0],[0,1],[1,1],[0,0]],\"threshold\":50.0, \"events\":[{\"type\":\"count\", \"output_frequency\": 1}]}]}"
                      }
                  },
                  "2": {
                      "sharedNodeMap": {
                          "PersonCountGraph/detector": "shared_detector1"
                      },
                      "parameters": {
                          "VIDEO_URL": "<Replace RTSP URL for camera 2>",
                          "VIDEO_SOURCE_ID": "camera 2",
                          "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                      }
                  },
                  "3": {
                      "sharedNodeMap": {
                          "PersonCountGraph/detector": "shared_detector1"
                      },
                      "parameters": {
                          "VIDEO_URL": "<Replace RTSP URL for camera 3>",
                          "VIDEO_SOURCE_ID": "camera 3",
                          "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                      }
                  },
                  "4": {
                      "sharedNodeMap": {
                          "PersonCountGraph/detector": "shared_detector1"
                      },
                      "parameters": {
                          "VIDEO_URL": "<Replace RTSP URL for camera 4>",
                          "VIDEO_SOURCE_ID": "camera 4",
                          "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                      }
                  },
                  "5": {
                      "sharedNodeMap": {
                          "PersonCountGraph/detector": "shared_detector1"
                      },
                      "parameters": {
                          "VIDEO_URL": "<Replace RTSP URL for camera 5>",
                          "VIDEO_SOURCE_ID": "camera 5",
                          "SPACEANALYTICS_CONFIG": "<Replace the zone config value, same format as above>"
                      }
                  }
              }
          }
      }
  }
  ```
| Nombre | Tipo| Descripción|
|---------|---------|---------|
| `batch_size` | int | Indica el número de cámaras que se utilizarán en esa operación.|

## <a name="next-steps"></a>Pasos siguientes

* [Implementación de una aplicación web de recuento de personas](spatial-analysis-web-app.md)
* [Registro y solución de problemas](spatial-analysis-logging.md)
* [Guía de selección de ubicación de la cámara](spatial-analysis-camera-placement.md)
* [Guía de selección de ubicación de zonas y líneas](spatial-analysis-zone-line-placement.md)
