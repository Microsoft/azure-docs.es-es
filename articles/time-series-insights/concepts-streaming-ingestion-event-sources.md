---
title: 'Origen del evento de ingesta de streaming: Azure Time Series Insights Gen2 | Microsoft Docs'
description: Obtenga información sobre el streaming de datos en Azure Time Series Insights Gen2.
author: lyrana
ms.author: lyhughes
manager: deepakpalled
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: 9ef87027bcda6c645d1239598c849f57fb0c8992
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87491976"
---
# <a name="azure-time-series-insights-gen2-event-sources"></a>Orígenes de eventos de Azure Time Series Insights Gen2

 El entorno de Azure Time Series Insights 2 puede tener hasta dos orígenes de eventos de streaming. Se admiten dos tipos de recursos de Azure como entradas:

- [Azure IoT Hub](../iot-hub/about-iot-hub.md)
- [Azure Event Hubs](../event-hubs/event-hubs-about.md)

Los eventos se deben enviar como JSON con codificación UTF8.

## <a name="create-or-edit-event-sources"></a>Crear o editar orígenes de eventos

Los recursos de origen del evento pueden residir en la misma suscripción de Azure que el entorno de Azure Time Series Insights Gen2 o en una suscripción diferente. Puede usar [Azure Portal](time-series-insights-update-create-environment.md#create-a-preview-payg-environment), la [CLI de Azure](https://github.com/Azure/azure-cli-extensions/tree/master/src/timeseriesinsights), las [plantillas de Resource Manager](time-series-insights-manage-resources-using-azure-resource-manager-template.md) y la [API de REST](/rest/api/time-series-insights/management(gen1/gen2)/eventsources) para crear, editar o quitar orígenes de eventos de su entorno.

Cuando conecte un origen de eventos, el entorno Azure Time Series Insights Gen2 leerá todos los eventos almacenados actualmente en su lote o en Event Hub, empezando por el evento más antiguo.

> [!IMPORTANT]
>
> * Al conectar un origen del evento al entorno de Azure Time Series Insights Gen2, puede experimentar una latencia inicial elevada.
> La latencia del origen del evento depende del número de eventos que se encuentren actualmente en IoT Hub o en el Centro de eventos.
> * Tras la primera ingesta de datos del origen del evento la latencia elevada disminuirá. Envíe una incidencia de soporte técnico a través de Azure Portal si sufre una latencia elevada de forma continuada.

## <a name="streaming-ingestion-best-practices"></a>Procedimientos recomendados para la ingesta de streaming

* Cree siempre un grupo de consumidores exclusivo para que el entorno de Azure Time Series Insights Gen2 consuma datos del origen de eventos. La reutilización de grupos de consumidores puede causar desconexiones aleatorias y puede provocar la pérdida de datos.

* Configure el entorno de Azure Time Series Insights Gen2 e IoT Hub o Event Hubs en la misma región de Azure. Aunque es posible configurar un origen de eventos en una región separada, este escenario no es compatible y no podemos garantizar una alta disponibilidad.

* No exceda el [límite de velocidad de rendimiento](./concepts-streaming-ingress-throughput-limits.md) para su entorno o por límite de partición.

* Configure una [alerta](https://review.docs.microsoft.com/azure/time-series-insights/time-series-insights-environment-mitigate-latency?branch=pr-en-us-117938#monitor-latency-and-throttling-with-alerts) de retraso para recibir una notificación si el entorno está experimentando problemas al procesar datos.

* Use la ingesta de streaming solo para los datos recientes y casi en tiempo real, no se admite el streaming de datos históricos.

* Comprenda cómo se aplicarán las secuencias de escape a las propiedades y los datos de JSON [se acoplarán y almacenarán.](./concepts-json-flattening-escaping-rules.md)

* Siga el principio de privilegio mínimo al proporcionar cadenas de conexión de origen de eventos. Para los centros de eventos, configure una directiva de acceso compartido solo con la notificación de *envío* y, para IoT Hub, use solo el permiso de *conexión de servicio*.

### <a name="historical-data-ingestion"></a>Ingesta de datos históricos

Actualmente no se admite el uso de la canalización de streaming para importar datos históricos en Azure Time Series Insights Gen2. Si necesita importar datos pasados a su entorno, siga estas instrucciones:

* No transmita datos en directo e históricos en paralelo. La ingesta de datos desordenados provocará un rendimiento degradado de las consultas.
* Para obtener el mejor rendimiento, ingiera los datos históricos de manera ordenada en el tiempo.
* Manténgase dentro de los límites de rendimiento de ingesta que se indican a continuación.
* Deshabilite el almacenamiento intermedio si los datos son más antiguos que el período de retención de almacenamiento intermedio.

## <a name="event-source-timestamp"></a>Marca de tiempo de origen del evento

Al configurar un origen de eventos, se le pedirá que proporcione una propiedad de identificador de marca de tiempo. La propiedad de marca de tiempo se usa para seguir eventos a lo largo del tiempo, este es el tiempo que se usará como $event.$ts en las [API de consulta](https://docs.microsoft.com/rest/api/time-series-insights/dataaccessgen2/query/execute) y la serie de gráficos en el Explorador de Azure Time Series Insights Gen2. Si no se proporcionan propiedades en el momento de la creación o si la propiedad Marca de tiempo no se encuentra en un evento, la instancia de IoT Hub del evento o los centros de eventos en cola se utilizarán como predeterminados. Los valores de propiedad de marca de tiempo se almacenan en UTC.

En general, los usuarios elegirán personalizar la propiedad de marca de tiempo y usar el tiempo cuando el sensor o la etiqueta generaron la lectura, en lugar de usar el tiempo en cola estándar del centro. Esto es particularmente necesario cuando los dispositivos experimentan una pérdida intermitente de conectividad y un lote de mensajes retrasados se reenvía a Azure Time Series Insights Gen2.

Si la marca de tiempo personalizada está dentro de un objeto JSON anidado o una matriz, deberá proporcionar el nombre de propiedad correcto siguiendo [nuestras convenciones de nomenclatura de acoplamiento y escape](concepts-json-flattening-escaping-rules.md). Por ejemplo, la marca de tiempo de origen del evento para la carga de JSON que se muestra [aquí](concepts-json-flattening-escaping-rules.md#example-a) debe introducirse como `"values.time"`.

### <a name="time-zone-offsets"></a>Desplazamiento de zona horaria

Las marcas de tiempo deben enviarse en formato ISO 8601 y se almacenarán en UTC. Si se proporciona un desplazamiento de zona horaria, se aplica el desplazamiento y, luego, la hora se almacena y se devuelve en formato UTC. Si el desplazamiento tiene un formato incorrecto, se ignorará. En situaciones en las que la solución puede no tener el contexto del desplazamiento original, puede enviar los datos del desplazamiento en una propiedad de evento adicional separada para asegurarse de que se conserva y de que su aplicación puede hacer referencia a ella en una respuesta de consulta.

El desplazamiento de zona horaria debe tener el formato de uno de los siguientes:

±HHMMZ</br>
±HH:MM</br>
±HH:MMZ</br>

## <a name="next-steps"></a>Pasos siguientes

* Lea las [Reglas de acoplamiento y de escape de JSON](./concepts-json-flattening-escaping-rules.md) para comprender cómo se almacenarán los eventos. 

* Comprender las [limitaciones de rendimiento](./concepts-streaming-ingress-throughput-limits.md) del entorno




