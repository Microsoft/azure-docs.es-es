---
title: Supervisión y reducción de la limitación, Azure Time Series Insights | Microsoft Docs
description: Aprenda a supervisar, diagnosticar y mitigar los problemas de rendimiento que causan la latencia y la limitación en Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: v-mamcge, jasonh, kfile
ms.devlang: csharp
ms.workload: big-data
ms.topic: troubleshooting
ms.date: 06/30/2020
ms.custom: seodec18
ms.openlocfilehash: 9fa47c81aede9de5d083f16f9e1705f687ad39a4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87046442"
---
# <a name="monitor-and-mitigate-throttling-to-reduce-latency-in-azure-time-series-insights-gen1"></a>Supervisión y reducción de la limitación para reducir la latencia en Azure Time Series Insights Gen1

Si la cantidad de datos de entrada supera la configuración del entorno, puede experimentar latencia o limitación en Azure Time Series Insights.

Para evitarlo, configure correctamente el entorno para la cantidad de datos que desee analizar.

Es más probable que experimente latencia y limitación si:

- Agrega un origen del evento que contiene datos antiguos que pueden superar la tasa de entrada asignada (ya que Azure Time Series Insights tendrá que leer todos los datos).
- Agrega más orígenes de eventos a un entorno, lo que produce un pico de eventos adicionales (que podría superar la capacidad del entorno).
- Inserta grandes cantidades de eventos históricos en un origen del evento, lo cual resulta en un retraso (ya que Azure Time Series Insights tendrá que leer todos los datos).
- Une los datos de referencia con la telemetría, lo que da lugar a un evento de mayor tamaño. El tamaño de paquete máximo permitido es 32 KB. Los paquetes de datos con más de 32 KB se truncan.

## <a name="video"></a>Vídeo

### <a name="learn-about-azure-time-series-insights-data-ingress-behavior-and-how-to-plan-for-itbr"></a>Obtenga más información sobre la entrada de datos de Azure Time Series Insights y cómo planearla.</br>

> [!VIDEO https://www.youtube.com/embed/npeZLAd9lxo]

## <a name="monitor-latency-and-throttling-with-alerts"></a>Supervisión de la latencia y la limitación con alertas

Las alertas pueden ayudarle a diagnosticar y mitigar los problemas de latencia que se producen en su entorno.

1. En Azure Portal, seleccione el entorno de Azure Time Series Insights. A continuación, seleccione **Alertas**.

   [![Incorporación de una alerta al entorno de Azure Time Series Insights](media/environment-mitigate-latency/mitigate-latency-add-alert.png)](media/environment-mitigate-latency/mitigate-latency-add-alert.png#lightbox)

1. Seleccione **+ Nueva regla de alertas**. Aparecerá el panel **Crear regla**. Seleccione **Agregar** en **CONDICIÓN**.

   [![Panel Agregar alerta](media/environment-mitigate-latency/mitigate-latency-add-pane.png)](media/environment-mitigate-latency/mitigate-latency-add-pane.png#lightbox)

1. A continuación, configure las condiciones exactas para la lógica de la señal.

   [![Configuración de la lógica de señal](media/environment-mitigate-latency/configure-alert-rule.png)](media/environment-mitigate-latency/configure-alert-rule.png#lightbox)

   Desde allí, puede configurar alertas mediante algunas de las siguientes condiciones:

   |Métrica  |Descripción  |
   |---------|---------|
   |**Bytes de entrada recibidos**     | Número de bytes sin procesar leídos desde los orígenes de eventos. El número sin procesar normalmente incluye el nombre y el valor de la propiedad.  |  
   |**Mensajes no válidos de entrada recibidos**     | Número de mensajes no válidos leídos desde todos los orígenes de eventos de Azure Event Hubs o Azure IoT Hub.      |
   |**Mensajes de entrada recibidos**   | Número de mensajes leídos desde todos los orígenes de eventos de Event Hubs o IoT Hub.        |
   |**Bytes de entrada almacenados**     | Tamaño total de los eventos almacenados y disponibles para su consulta. Se calcula el tamaño solo sobre el valor de propiedad.        |
   |**Eventos de entrada almacenados**     |   Número de eventos planos almacenados y disponibles para su consulta.      |
   |**Tiempo de retardo de los mensajes de entrada recibidos**    |  Diferencia en segundos entre el momento en el que el mensaje se pone en la cola en el origen del evento y el momento en que se procesa en la entrada.      |
   |**Retardo de recuento de los mensajes de entrada recibidos**    |  Diferencia entre el número de secuencia del último mensaje en cola en la partición del origen del evento y el del mensaje que se está procesando en la entrada.      |

   Seleccione **Listo**.

1. Después de configurar la lógica de señal deseada, revise visualmente la regla de alerta elegida.

   [![Vista de latencia y gráficos](media/environment-mitigate-latency/mitigate-latency-view-and-charting.png)](media/environment-mitigate-latency/mitigate-latency-view-and-charting.png#lightbox)

## <a name="throttling-and-ingress-management"></a>Limitación y administración de entrada

* Si experimenta alguna limitación, se registrará un valor en *Ingress Received Message Time Lag* (Tiempo de retardo de los mensajes de entrada recibidos) que indica cuántos segundos de retraso tiene el entorno de Azure Time Series Insights respecto al momento real en el que el mensaje llega al origen del evento (sin contar el tiempo de indexación, unos 30-60 segundos).  

  *Ingress Received Message Count Lag* (Retardo de recuento de los mensajes de entrada recibidos) también debe tener un valor para que pueda determinar los mensajes que tiene detrás.  La manera más fácil de mantenerse al día es aumentar la capacidad del entorno a un tamaño que le permita compensar la diferencia.  

  Por ejemplo, si el entorno S1 muestra un retraso de 5 millones de mensajes, puede aumentar el tamaño del entorno a seis unidades durante un día aproximadamente para que se actualice.  Podría aumentarlo más para ponerse al día más rápidamente. Este período para ponerse al día es común al aprovisionar inicialmente un entorno, en particular al conectarlo a un origen de eventos que no esté vacío o con la carga masiva de numerosos datos históricos.

* Otra de estas técnicas consiste en establecer una alerta de **eventos de entrada almacenados** >= un umbral ligeramente por debajo de la capacidad total del entorno durante un período de 2 horas.  Esta alerta le ayudará a conocer si se encuentra continuamente en el límite de la capacidad, lo cual indica una alta probabilidad de latencia. 

  Por ejemplo, si tiene tres unidades de nivel S1 aprovisionadas (o una capacidad de entrada de 2100 eventos por minuto), puede establecer una alerta de **eventos de entrada almacenados** >= 1900 eventos durante 2 horas. Si continuamente supera este umbral y, como consecuencia, se desencadena la alerta, es posible que necesite un mayor aprovisionamiento.  

* Si sospecha que está experimentando alguna limitación, puede comparar los **mensajes de entrada recibidos** con los mensajes de origen del evento de salida.  Si la entrada en el centro de eventos es mayor que los **mensajes de entrada recibidos**, es probable que se esté limitando su instancia de Azure Time Series Insights.

## <a name="improving-performance"></a>Mejora del rendimiento

Para reducir la limitación o la posibilidad de experimentar latencia, la mejor forma de solucionarlo es aumentar la capacidad del entorno.

Para evitarlo, configure correctamente el entorno para la cantidad de datos que desee analizar. Para obtener más información acerca de cómo agregar capacidad al entorno, lea [Escalado del entorno](time-series-insights-how-to-scale-your-environment.md).

## <a name="next-steps"></a>Pasos siguientes

- Más información en [Diagnóstico y solución de problemas de su entorno Azure Time Series Insights](time-series-insights-diagnose-and-solve-problems.md).

- Aprenda a [escalar el entorno de Azure Time Series Insights](time-series-insights-how-to-scale-your-environment.md).