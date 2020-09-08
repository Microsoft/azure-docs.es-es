---
title: Detección de movimiento y objetos del análisis de vídeo de Azure IoT Central | Microsoft Docs
description: Aprenda a compilar una aplicación de IoT Central mediante la plantilla de la aplicación Video analytics - object and motion detection en IoT Central. Esta plantilla usa análisis de vídeo en directo y cámaras conectadas.
author: KishorIoT
ms.author: nandab
ms.date: 07/27/2020
ms.topic: conceptual
ms.service: iot-central
ms.subservice: iot-central-retail
services: iot-central
ms.openlocfilehash: 087c5e5716a4c8c640faebd7470905fde85d0136
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88038149"
---
# <a name="video-analytics---object-and-motion-detection-application-architecture"></a>Arquitectura de la aplicación Video analytics - object and motion detection

La plantilla de la aplicación de **detección de objetos y movimiento de Video Analytics** permite compilar soluciones de IoT entre las que se incluyen las funcionalidades de análisis de vídeo en directo.

:::image type="content" source="media/architecture-video-analytics/architecture.png" alt-text="Architecture":::

Los componentes clave de la solución de análisis de vídeo incluyen:

## <a name="live-video-analytics-lva"></a>Live Video Analytics (LVA)

LVA ofrece una plataforma para compilar aplicaciones de vídeo inteligentes que abarquen el entorno perimetral y la nube. La plataforma permite compilar aplicaciones de vídeo inteligentes que abarquen el perímetro y la nube. La plataforma ofrece la funcionalidad de capturar, grabar, analizar vídeo en directo y publicar los resultados, que podrían ser análisis de vídeo o vídeo, en los servicios de Azure. Los servicios de Azure podrían ejecutarse en la nube o en el dispositivo perimetral. La plataforma se puede usar para mejorar las soluciones de IoT con análisis de vídeo.

Para más información, consulte [Live Video Analytics](https://github.com/Azure/live-video-analytics) en GitHub.

## <a name="iot-edge-lva-gateway-module"></a>Módulo de la puerta de enlace para LVA de IoT Edge

El módulo de la puerta de enlace para LVA de IoT Edge crea instancias de las cámaras como dispositivos nuevos y los conecta directamente a IoT Central mediante el SDK de cliente de dispositivo IoT.

En esta implementación de referencia, los dispositivos se conectan a la solución mediante claves simétricas del dispositivo perimetral. Para más información sobre la conectividad de los dispositivos, consulte [Conexión a Azure IoT Central](../core/concepts-get-connected.md).

## <a name="media-graph"></a>Grafo multimedia

El grafo multimedia permite definir dónde se debe capturar el contenido multimedia, cómo procesarlo y dónde se entregan los resultados. Conecte componentes o nodos de la manera deseada para configurar este grafo multimedia. Para más información, consulte [Grafos multimedia](https://github.com/Azure/live-video-analytics/tree/master/MediaGraph) en GitHub.

## <a name="next-steps"></a>Pasos siguientes

El siguiente paso sugerido es aprender sobre la [Creación de una aplicación de análisis de vídeo en Azure IoT Central](tutorial-video-analytics-create-app.md).
