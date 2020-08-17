---
title: 'Planeamiento de un entorno de Gen1: Azure Time Series Insights | Microsoft Docs'
description: Prácticas recomendadas para preparar, configurar e implementar el entorno de Azure Time Series Insights Gen1.
services: time-series-insights
ms.service: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: seodec18
ms.openlocfilehash: ea243ffd125a527949c4a264177e78de04a22aae
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87046396"
---
# <a name="plan-your-azure-time-series-insights-gen1-environment"></a>Planeamiento del entorno de Azure Time Series Insights Gen1

En este artículo se describe cómo planear el entorno de Azure Time Series Insights Gen1 según la velocidad de entrada esperada y los requisitos de retención de datos.

## <a name="video"></a>Vídeo

**Vea este vídeo para obtener más información sobre la retención de datos en Azure Time Series Insights y cómo planearla**:<br /><br />

> [!VIDEO https://www.youtube.com/embed/03x6zKDQ6DU]

## <a name="best-practices"></a>Procedimientos recomendados

Para empezar a trabajar con Azure Time Series Insights, se recomienda saber cuántos datos se espera insertar por minuto, así como durante cuánto tiempo se deben almacenar.  

Para más información sobre la capacidad y la retención de ambas SKU de Azure Time Series Insights, consulte los [precios de Azure Time Series Insights](https://azure.microsoft.com/pricing/details/time-series-insights/).

Considere estos atributos para planear mejor el entorno de Azure Time Series Insights para el éxito a largo plazo:

- [Capacidad de almacenamiento](#storage-capacity)
- [Período de retención de datos](#data-retention)
- [Capacidad de entrada](#ingress-capacity)
- [Eventos con forma](#shape-your-events)
- [Confirmación de la presencia de datos de referencia](#ensure-that-you-have-reference-data)

## <a name="storage-capacity"></a>Capacidad de almacenamiento

De manera predeterminada, Azure Time Series Insights conserva los datos en función de la cantidad de almacenamiento que ha aprovisionado (unidades &#215; la cantidad de almacenamiento por unidad) y la entrada.

## <a name="data-retention"></a>Retención de datos

Puede cambiar el valor de **Data retention time** (Tiempo de retención de datos) en el entorno de Azure Time Series Insights. Puede habilitar hasta 400 días de retención. 

Azure Time Series Insights tiene dos modos:

* Un modo se optimiza para los datos más actualizados. Aplica una directiva **Purgar datos antiguos** y mantiene los datos recientes disponibles con la instancia. Este modo está activado, de forma predeterminada. 
* El otro optimiza los datos para que permanezcan por debajo de los límites de retención configurados. **Pausar entrada** impide que los nuevos datos entren cuando está seleccionado como **Comportamiento correspondiente a un exceso del límite de almacenamiento** .

Puede ajustar la retención y alternar entre ambos modos en la página de configuración del entorno de Azure Portal.

> [!IMPORTANT]
> Puede configurar un máximo de 400 días de retención de datos en el entorno de Azure Time Series Insights Gen1.

### <a name="configure-data-retention"></a>Configuración de la retención de datos

1. En [Azure Portal](https://portal.azure.com), seleccione el entorno de Time Series Insights.

1. En el panel **Entorno de Time Series Insights**, en **Configuración**, seleccione **Configuración de Storage**.

1. En el cuadro **Tiempo de retención de datos (en días)** , escriba un valor entre 1 y 400.

   [![Configuración de la retención](media/data-retention/configure-data-retention.png)](media/data-retention/configure-data-retention.png#lightbox)

> [!TIP]
> Para obtener más información sobre cómo implementar una directiva de retención de datos adecuada, consulte [Cómo configurar la retención](./time-series-insights-how-to-configure-retention.md).

## <a name="ingress-capacity"></a>Capacidad de entrada

[!INCLUDE [Azure Time Series Insights Gen1 limits](../../includes/time-series-insights-ga-limits.md)]

### <a name="environment-planning"></a>Planeamiento del entorno

La segunda área de interés para planear un entorno de Azure Time Series Insights es la capacidad de entrada. Tanto el almacenamiento de entrada diario como la capacidad de evento se miden por minuto, en bloques de 1 KB. El tamaño de paquete máximo permitido es 32 KB. Los paquetes de datos mayores de 32 KB se truncan.

Puede aumentar la capacidad de una SKU S1 o S2 a 10 unidades en un solo entorno. No puede migrar desde un entorno de S1 a uno de S2. No puede migrar desde un entorno de S2 a uno de S1.

Para la capacidad de entrada, primero determine la entrada total que requiere de manera mensual. A continuación, determine sus necesidades por minuto. 

La limitación y la latencia desempeñan un papel en la capacidad por minuto. Si tiene un incremento en la entrada de datos que dura menos de 24 horas, Azure Time Series Insights puede "ponerse al día" con una velocidad de entrada que doble las velocidades mencionadas en la tabla anterior.

Por ejemplo, si tiene una sola SKU S1, los datos entran a una velocidad de 720 eventos por minuto y hay un incremento de la velocidad de datos durante menos de una hora a una velocidad de 1440 eventos o menos, no habrá una latencia apreciable en el entorno. Pero si supera los 1440 eventos por minuto durante más de una hora, es probable que experimente una latencia con respecto a los datos que se visualizan y están disponibles para consulta en el entorno.

Es posible que no sepa de antemano cuántos datos espera insertar. En este caso, puede encontrar la telemetría de datos para [Azure IoT Hub](../iot-hub/iot-hub-metrics.md) y [Azure Event Hubs](https://blogs.msdn.microsoft.com/cloud_solution_architect/2016/05/25/using-the-azure-rest-apis-to-retrieve-event-hub-metrics/) en la suscripción de Azure Portal. La telemetría puede ayudar a determinar cómo aprovisionar el entorno. Use la página **Métricas** del origen del evento correspondiente en Azure Portal para ver su telemetría. Si comprende las métricas de origen del evento, puede planea de manera más eficaz y aprovisionar el entorno de Azure Time Series Insights.

### <a name="calculate-ingress-requirements"></a>Cálculo de los requisitos de entrada

Para calcular los requisitos de entrada:

- Verifique que la capacidad de entrada está por encima de la velocidad promedio por minuto y que el entorno es suficientemente grande para controlar la entrada anticipada equivalente al doble de la capacidad durante menos de una hora.

- Si se producen incrementos de entrada que duran más de 1 hora, use la tasa de incremento como promedio. Aprovisione un entorno con capacidad para controlar la tasa de incremento.

### <a name="mitigate-throttling-and-latency"></a>Mitigación de la limitación y la latencia

Para información sobre cómo prevenir la limitación y la latencia, consulte el artículo sobre la [mitigación de la limitación y la latencia](time-series-insights-environment-mitigate-latency.md).

## <a name="shape-your-events"></a>Moldear los eventos

Es importante asegurarse de que el método de envío de eventos de Azure Time Series Insights es compatible con el tamaño del entorno que se va a aprovisionar. (De forma inversa, puede asignar el tamaño del entorno respecto al número de eventos que lee Azure Time Series Insights y el tamaño de cada evento). También es importante pensar en los atributos que quiere usar para segmentar y filtrar al consultar los datos.

> [!TIP]
> Revise la documentación de configuración de JSON en [Enviar eventos](time-series-insights-send-events.md).

## <a name="ensure-that-you-have-reference-data"></a>Garantizar la presencia de datos de referencia

Un *conjunto de datos de referencia* es una colección de elementos que aumentan los eventos de un origen de eventos. El motor de entrada de Azure Time Series Insights combina cada evento del origen de eventos con la fila de datos correspondiente en el conjunto de datos de referencia. A partir de ese momento, el evento aumentado está disponible para consultas. La combinación se basa en las columnas de **clave principal** definidas en el conjunto de datos de referencia.

> [!NOTE]
> Los datos de referencia no se combinan de manera retroactiva. Solo los datos de entrada actuales y futuros se combinan y unen con el conjunto de datos de referencia una vez que se han configurado y cargado. Si va a enviar una gran cantidad de datos históricos a Azure Time Series Insights y primero no carga o crea datos de referencia en Azure Time Series Insights, deberá rehacer el trabajo (lo que no resulta nada agradable).  

Para más información sobre cómo crear, cargar y administrar los datos de referencia en Azure Time Series Insights, consulte nuestra [documentación sobre el conjunto de datos de referencia](time-series-insights-add-reference-data-set.md).

[!INCLUDE [business-disaster-recover](../../includes/time-series-insights-business-recovery.md)]

## <a name="next-steps"></a>Pasos siguientes

- Empiece por crear [un entorno de Azure Time Series Insights en Azure Portal](time-series-insights-get-started.md).

- Obtenga información sobre cómo [agregar un origen de evento de Event Hubs](time-series-insights-how-to-add-an-event-source-eventhub.md) al entorno de Azure Time Series Insights.

- Obtenga información sobre cómo [configurar un origen de eventos de IoT Hub](time-series-insights-how-to-add-an-event-source-iothub.md).
