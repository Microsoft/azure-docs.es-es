---
title: 'Notas de la versión de Live Video Analytics on IoT Edge: Azure'
description: En este tema se proporcionan las notas de la versión sobre las mejoras, las versiones, las correcciones de errores y los problemas conocidos de Live Video Analytics on IoT Edge.
ms.topic: conceptual
ms.date: 08/19/2020
ms.openlocfilehash: 0bdf11cd99d99067dc53dde7d55fd37b96a382c9
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90882730"
---
# <a name="live-video-analytics-on-iot-edge-release-notes"></a>Notas de la versión de Live Video Analytics on IoT Edge

>Reciba notificaciones para volver a visitar esta página y obtener actualizaciones; para ello, copie y pegue esta URL (`https://docs.microsoft.com/api/search/rss?search=%22Live+Video+Analytics+on+IoT+Edge+release+notes%22&locale=en-us`) en el lector de fuentes RSS.

En este artículo se proporciona información acerca de lo siguiente:

* Versiones más recientes
* Problemas conocidos
* Corrección de errores
* Funciones obsoletas

<hr width=100%>

## <a name="september-22-2020"></a>22 de septiembre de 2020

La etiqueta de versión de la actualización de septiembre de 2020 del módulo es:

```
mcr.microsoft.com/media/live-video-analytics:1.0.4
```

> [!NOTE]
> En los inicios rápido y tutoriales, los manifiestos de implementación usan una etiqueta de 1 (live-video-analytics:1). Por lo tanto, con solo repetir la implementación de tales manifiestos se actualizará el módulo en los dispositivos perimetrales.

### <a name="module-updates"></a>Actualizaciones del módulo

* Hay un nuevo nodo de extensión de grafos, [MediaGraphCognitiveServicesVisionExtension](custom-vision-tutorial.md), disponible para integrarse con el [módulo Spatial Analysis](spatial-analysis-tutorial.md)(versión preliminar) de Cognitive Services.
* Se ha agregado compatibilidad con dispositivos ARM64 de Linux: use [pasos manuales](deploy-iot-edge-device.md) para implementar en esos dispositivos.

### <a name="documentation-updates"></a>Actualizaciones de la documentación

* Hay [instrucciones](deploy-azure-stack-edge-how-to.md) disponibles para usar Live Video Analytics en IoT Edge en dispositivos Azure Stack Edge.
* Nuevo tutorial sobre el desarrollo de modelos de visión de proceso específicos de cada escenario mediante el [servicio Custom Vision](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/) y su uso para [analizar vídeo en vivo](custom-vision-tutorial.md) en tiempo real.

<hr width=100%>

## <a name="august-19-2020"></a>19 de agosto de 2020

La etiqueta de versión de la actualización de agosto de 2020 del módulo es la siguiente:

```
mcr.microsoft.com/media/live-video-analytics:1.0.3
```

> [!NOTE]
> En los inicios rápido y tutoriales, los manifiestos de implementación usan una etiqueta de 1 (live-video-analytics:1). Por lo tanto, con solo repetir la implementación de tales manifiestos se actualizará el módulo en los dispositivos perimetrales.

### <a name="module-updates"></a>Actualizaciones del módulo

* Ahora puede obtener un alto rendimiento de la transferencia de contenido de datos entre Live Video Analytics en IoT Edge y su extensión personalizada mediante el marco de gRPC. Consulte [aquí](analyze-live-video-use-your-grpc-model-quickstart.md) para empezar.
* Implementación regional más amplia de Live Video Analytics y solo se ha actualizado el servicio en la nube.  
* Live Video Analytics ahora está disponible en otras 25 regiones de todo el mundo. A continuación, se muestra la [lista](https://azure.microsoft.com/global-infrastructure/services/?products=media-services) de todas las regiones disponibles.  
* La [configuración](https://aka.ms/lva-edge/setup-resources-for-samples) de los inicios rápidos se actualizó también con compatibilidad con regiones nuevas.
    * No hay ninguna llamada a la acción para ningún usuario que ya haya configurado los recursos.

### <a name="bug-fixes"></a>Corrección de errores 

* Eliminación del uso de una extensión de Azure en desuso en el script de configuración.

<hr width=100%>

## <a name="july-13-2020"></a>13 de julio de 2020

La etiqueta de versión de la actualización de julio de 2020 del módulo es la siguiente:

```
mcr.microsoft.com/media/live-video-analytics:1.0.2
```

> [!NOTE]
> En los inicios rápido y tutoriales, los manifiestos de implementación usan una etiqueta de 1 (live-video-analytics:1). Por lo tanto, con solo repetir la implementación de tales manifiestos se actualizará el módulo en los dispositivos perimetrales.

### <a name="module-updates"></a>Actualizaciones del módulo

* Ahora puede crear topologías de grafos que tengan un nodo receptor de recursos, así como un nodo receptor de archivos de nivel inferior de un nodo procesador de puerta de señal. Consulte [aquí](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph/topologies/evr-motion-assets-files) para ver un ejemplo.

### <a name="bug-fixes"></a>Corrección de errores

* Mejoras en la validación de las propiedades deseadas

<hr width=100%>

## <a name="june-1-2020"></a>1 de junio de 2020

Esta versión es la primera versión preliminar pública de Live Video Analytics on IoT Edge. La etiqueta de versión es la siguiente:

```
     mcr.microsoft.com/media/live-video-analytics:1.0.0
```

### <a name="supported-functionalities"></a>Funcionalidades admitidas

* Analice secuencias de vídeo en directo con los módulos de inteligencia artificial que prefiera y, de manera opcional, grabe vídeos en su dispositivo perimetral o en la nube.
* Use este módulo en sistemas operativos Linux AMD64 [compatibles](../../iot-edge/support.md) con IoT Edge.
* Implemente y configure el módulo a través de IoT Hub mediante Azure Portal o Visual Studio Code.
* Administre [topologías e instancias de grafos](media-graph-concept.md#media-graph-topologies-and-instances) de forma remota o local mediante la llamada a los siguientes métodos directos:

    *   GraphTopologyGet
    *   GraphTopologySet
    *   GraphTopologyDelete
    *   GraphTopologyList
    *   GraphInstanceGet
    *   GraphInstanceSet
    *   GraphInstanceDelete
    *   GraphInstanceList

## <a name="next-steps"></a>Pasos siguientes

[Información general](overview.md)
