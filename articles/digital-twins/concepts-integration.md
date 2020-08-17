---
title: Integración con otros servicios
titleSuffix: Azure Digital Twins
description: Conozca los requisitos de entrada y salida al implementar Azure Digital Twins.
author: baanders
ms.author: baanders
ms.date: 3/16/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: d0bca2ad14216873836c751dc002ea915baa3a66
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87800490"
---
# <a name="integrate-azure-digital-twins-with-other-services"></a>Integre Azure Digital Twins con otros servicios

Azure Digital Twins se suele usar junto con otros servicios. Mediante [**rutas de evento**](concepts-route-events.md), Azure Digital Twins recibe datos de los servicios de nivel superior como [IOT Hub](../iot-hub/about-iot-hub.md), que se usa para proporcionar telemetría y notificaciones. Azure Digital Twins también puede enrutar los datos a servicios de bajada para el almacenamiento, la integración de flujos de trabajo, el análisis y otros usos. 

## <a name="data-ingress"></a>Entrada de datos

Azure Digital Twins se puede controlar con datos y eventos de cualquier servicio: IoT Hub, Logic Apps, su propio servicio personalizado, etc. Esto le permite recopilar datos de telemetría de dispositivos físicos en el entorno y procesarlos mediante el grafo de Azure Digital Twins en la nube.

Azure Digital Twins no tiene un centro de IoT Hub integrado. Puede usar un centro de IoT Hub existente que tenga actualmente en producción o bien implementar uno nuevo. Así, obtiene acceso completo a todas las funcionalidades de administración de los dispositivos de IoT Hub.

Para ingerir datos de cualquier origen en Azure Digital Twins, use una [función de Azure Functions](../azure-functions/functions-overview.md). Más información sobre este patrón en [*Procedimiento: Ingesta de telemetría de IoT Hub*](how-to-ingest-iot-hub-data.md) o pruébelo usted mismo en Azure Digital Twins: [*Tutorial: Conexión de una solución de un extremo a otro*](tutorial-end-to-end.md).

## <a name="data-egress-services"></a>Servicios de salida de datos

Azure Digital Twins puede enviar datos a los **puntos de conexión** conectados. Los puntos de conexión admitidos pueden ser:
* [Event Hubs](../event-hubs/event-hubs-about.md)
* [Event Grid](../event-grid/overview.md)
* [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md)

Los puntos de conexión se asocian a Azure Digital Twins mediante las API de administración o Azure Portal. Más información sobre cómo adjuntar un punto de conexión a Azure Digital Twins en [*Procedimiento: Administración de puntos de conexión y rutas*](how-to-manage-routes-apis-cli.md).

Hay muchos otros servicios en los que es posible que desee dirigir los datos en última instancia, como [Azure Storage](../storage/common/storage-introduction.md) o [Time Series Insights](../time-series-insights/time-series-insights-update-overview.md). Para enviar los datos a servicios como estos, asocie el servicio de destino a un punto de conexión.

Por ejemplo, si también usa [Azure Maps](../azure-maps/about-azure-maps.md) y desea poner en correlación la ubicación con el [grafo gemelo](concepts-twins-graph.md) de Azure Digital Twins, puede usar Azure Functions con Event Grid para establecer la comunicación entre todos los servicios de la implementación.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre los puntos de conexión y el enrutamiento de eventos a servicios externos:
* [*Conceptos: Enrutamiento de eventos en Azure Digital Twins*](concepts-route-events.md)

Vea cómo configurar Azure Digital Twins para ingerir datos de IoT Hub:
* [*Procedimiento: Ingesta de telemetría de IoT Hub*](how-to-ingest-iot-hub-data.md)
