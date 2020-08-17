---
title: 'Grabación de vídeo continua: Azure'
description: Grabación de vídeo continua (CVR) hace referencia al proceso de grabación continua del vídeo desde un origen de vídeo. En este tema se trata qué es CVR.
ms.topic: conceptual
ms.date: 04/27/2020
ms.openlocfilehash: 76af97fe1398421f5f37cfca32127d926ce56bac
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87043307"
---
# <a name="continuous-video-recording"></a>Grabación de vídeo continua  

## <a name="suggested-pre-reading"></a>Sugerencias antes de la lectura  

* [Concepto de grafo de elementos multimedia](media-graph-concept.md)
* [Concepto de grabación de vídeo](video-recording-concept.md)

## <a name="overview"></a>Información general

Grabación de vídeo continua (CVR) hace referencia al proceso de grabación continua del vídeo desde un origen de vídeo. Live Video Analytics on IoT Edge admite la grabación de vídeo continua, de forma ininterrumpida, de una cámara de videovigilancia a través de un [grafo de elementos multimedia](media-graph-concept.md) que se compone de un nodo de origen RTSP y un nodo receptor de recursos. El diagrama siguiente muestra una representación gráfica de ese grafo. La representación JSON de la [topología de grafo](media-graph-concept.md?branch=release-preview-media-services-lva#media-graph-topologies-and-instances) de este tipo de grafo de elementos multimedia se puede encontrar [aquí](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph/topologies/cvr-asset).

![Grabación de vídeo continua](./media/continuous-video-recording/continuous-video-recording-overview.png)

El grafo de elementos multimedia que se ha descrito anteriormente se puede ejecutar en un dispositivo perimetral, con el vídeo de grabación del receptor de recursos en un [recurso](terminology.md#asset) de Azure Media Services. El vídeo se grabará durante todo el tiempo que el grafo de elementos multimedia permanezca en estado activado. Al grabarse como un recurso, para reproducirlo se pueden usar las funcionalidades de streaming existentes de Media Services. Para más información, consulte [Reproducción de contenido grabado](video-playback-concept.md).

## <a name="resilient-recording"></a>Grabación resistente

El análisis de vídeo en directo en IoT Edge admite el funcionamiento cuando la red no se encuentra en condiciones óptimas, en los casos en que el dispositivo perimetral puede perder ocasionalmente la conectividad con la nube o experimentar una reducción del ancho de banda disponible. Para tener todo esto en cuenta, el vídeo del origen se registra localmente en una caché y se sincroniza automáticamente con el recurso de forma periódica. Si examina el [JSON de la topología del grafo](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph/topologies/cvr-asset/topology.json), observará que tiene definidas las siguientes propiedades:

```
    "segmentLength": "PT30S",
    "localMediaCacheMaximumSizeMiB": "2048",
    "localMediaCachePath": "/var/lib/azuremediaservices/tmp/",
```
Las dos últimas propiedades son apropiadas para la grabación resistente (también son propiedades obligatorias para un nodo de receptor de recursos). La propiedad localMediaCachePath indica al receptor de recursos que use la ruta de acceso de la carpeta para almacenar en caché los datos multimedia antes de cargarlos en el recurso. Puede consultar [este](../../iot-edge/how-to-access-host-storage-from-module.md) artículo para entender de qué forma puede el módulo IoT Edge usar el almacenamiento local del dispositivo. La propiedad localMediaCacheMaximumSizeMiB define cuánto espacio en disco puede usar el receptor de recursos como memoria caché (1 MiB = 1024 * 1024 bytes). 

Si el módulo IoT Edge pierde la conectividad durante mucho tiempo y el contenido almacenado en la carpeta de la caché alcanza el valor localMediaCacheMaximumSizeMiB, el receptor de recursos empezará a descartar los datos de la caché, empezando por los más antiguos. Por ejemplo, si el dispositivo perdió la conectividad a las 10 a. m. y la caché alcanza el límite máximo a las 6 p. m., el receptor de recursos empieza a eliminar los datos grabados a las 10 a. m. 

Cuando se restaure la conectividad de red, el receptor de recursos comenzará la carga desde la memoria caché y volverá a empezar por los datos más antiguos. En el ejemplo anterior, supongamos que es necesario descartar de la memoria caché 5 minutos de vídeo antes de que se restaurar la conectividad (por ejemplo, a las 6:02 p. m.), el receptor de recursos empezará a realizar la carga desde la marca 10:05 a. m.

Si más adelante examina el recurso mediante [estas](playback-recordings-how-to.md) API, verá que hay un hueco en el recurso de aproximadamente las 10:00 a las 10:05 a. m.

## <a name="segmented-recording"></a>Grabación segmentada  

Como ya se ha explicado, el nodo receptor de recursos grabará vídeo en una caché local y cargará periódicamente el vídeo en la nube. La propiedad segmentLength (que se muestra en la sección anterior) le ayudará a controlar el costo de las transacciones de escritura asociado a la escritura de datos en la cuenta de almacenamiento en la que se registra el recurso. Por ejemplo, si aumenta el período de carga de 30 segundos a 5 minutos, el número de transacciones de almacenamiento se reducirá 10 veces (5 * 60/30).

La propiedad segmentLength garantiza que el módulo IoT Edge cargará vídeo como máximo una vez cada segmentLength segundos. Esta propiedad tiene un valor mínimo de 30 segundos (también el valor predeterminado) y se puede aumentar en incrementos de 30 segundos hasta un máximo de 5 minutos.

>[!NOTE]
>En [este artículo](playback-recordings-how-to.md) puede ver el efecto que segmentLength tiene en la reproducción.


## <a name="see-also"></a>Consulte también

* [Grabación de vídeo basada en eventos](event-based-video-recording-concept.md)
* [Reproducción de contenido grabado](video-playback-concept.md)


## <a name="next-steps"></a>Pasos siguientes

[Tutorial: grabación continua de vídeo](continuous-video-recording-tutorial.md)
