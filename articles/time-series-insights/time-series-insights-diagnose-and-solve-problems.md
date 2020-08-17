---
title: 'Diagnóstico y resolución de problemas: Azure Time Series Insights'
description: En este artículo se describe cómo diagnosticar y solucionar problemas comunes en el entorno de Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: v-mamcge
ms.workload: big-data
ms.topic: troubleshooting
ms.date: 06/30/2020
ms.custom: seodec18
ms.openlocfilehash: 0630e4dfcfc01e5c20fa6fcc3a516dbea6f6f53b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87046461"
---
# <a name="diagnose-and-solve-issues-in-your-azure-time-series-insights-gen1-environment"></a>Diagnóstico y solución de problemas en el entorno de Azure Time Series Insights Gen1

En este artículo se describen algunos problemas que podría encontrar en el entorno de Azure Time Series Insights. Se ofrecen posibles causas y soluciones para la resolución.

## <a name="video"></a>Vídeo

### <a name="learn-about-common-azure-time-series-insights-challenges-and-mitigationsbr"></a>Conozca los desafíos y las mitigaciones comunes de Azure Time Series Insights.</br>

> [!VIDEO https://www.youtube.com/embed/7U0SwxAVSKw]

## <a name="problem-no-data-is-shown"></a>Problema: no se muestra ningún dato

Si no aparece ningún dato en el [explorador de Azure Time Series Insights](https://insights.timeseries.azure.com), considere estas causas comunes.

### <a name="cause-a-event-source-data-isnt-in-json-format"></a>Causa A: los datos de origen del evento no tienen formato JSON

Azure Time Series Insights solo admite datos JSON. Para ver ejemplos de JSON, consulte [Formas de JSON admitidas](./how-to-shape-query-json.md).

### <a name="cause-b-the-event-source-key-is-missing-a-required-permission"></a>Causa B: a la clave de origen del evento le falta un permiso necesario

* Para un centro de IoT en Azure IoT Hub, debe proporcionar la clave con los permisos de conexión de servicio. Seleccione la directiva **iothubowner** o **service**. Ambas tienen permisos de conexión de servicio.

   [![Permisos de conexión de servicio de IoT Hub](media/diagnose-and-solve-problems/iothub-serviceconnect-permissions.png)](media/diagnose-and-solve-problems/iothub-serviceconnect-permissions.png#lightbox)

* Para un centro de eventos de Azure Event Hubs, debe proporcionar la clave con permisos de escucha. Servirán las directivas **read** y **manage**, dado que ambas tienen permisos de escucha.

   [![Permisos de escucha de centro de eventos](media/diagnose-and-solve-problems/eventhub-listen-permissions.png)](media/diagnose-and-solve-problems/eventhub-listen-permissions.png#lightbox)

### <a name="cause-c-the-provided-consumer-group-isnt-exclusive-to-azure-time-series-insights"></a>Causa C: el grupo de consumidores proporcionado no es exclusivo de Azure Time Series Insights.

Al registrar un centro de IoT o un centro de eventos, es importante establecer el grupo de consumidores que quiere usar para leer los datos. Este grupo de consumidores *no puede ser compartido*. Si el grupo de consumidores es compartido, el centro de IoT o el centro de eventos subyacente desconectará automáticamente uno de los lectores de forma aleatoria. Proporcione un grupo de consumidores exclusivo que pueda leer Azure Time Series Insights.

### <a name="cause-d-the-environment-has-just-been-provisioned"></a>Causa D: el entorno se ha aprovisionado recientemente

Los datos aparecerán en el Explorador de Azure Time Series Insights al cabo de unos minutos, después de que se creen por primera vez el entorno y sus datos.

## <a name="problem-some-data-is-shown-but-data-is-missing"></a>Problema: se muestran algunos datos, pero faltan otros

Cuando los datos solo aparecen parcialmente y parece que están atrasados, tenga en cuenta estos posibles problemas.

### <a name="cause-a-your-environment-is-being-throttled"></a>Causa A: puede que se esté limitando el entorno

La [limitación](time-series-insights-environment-mitigate-latency.md) es un problema común cuando se aprovisionan entornos después de crear un origen de eventos que tiene datos. Azure IoT Hub y Azure Events Hubs almacenan datos durante siete días como máximo. Azure Time Series Insights siempre empieza con el evento más antiguo del origen de eventos (primero en entrar, primero en salir o *FIFO*).

Por ejemplo, si tiene 5 millones de eventos en un origen de eventos al conectarse a un único entorno S1 de Azure Time Series Insights, Azure Time Series Insights lee aproximadamente 1 millón de eventos al día. Podría parecer que Azure Time Series Insights está experimentando cinco días de latencia. Sin embargo, lo que sucede es que el entorno está limitado.

Si tiene los eventos antiguos en el origen de eventos, puede enfocarlo de dos maneras:

- Cambie los límites de retención del origen de eventos para ayudar a eliminar los eventos antiguos que no quiere que aparezcan en Azure Time Series Insights.
- Aprovisione un tamaño mayor de entorno (número de unidades) para aumentar el rendimiento de los eventos antiguos. Siguiendo con el ejemplo anterior, si aumenta ese mismo entorno de S1 a cinco unidades durante un día, debería actualizarse en el plazo de un día. Si la producción de eventos de estado estable es de 1 millón o menos eventos al día, puede reducir la capacidad de eventos a una unidad cuando Azure Time Series Insights se ponga al día.

El límite de aplicación se basa en el tipo y la capacidad de la SKU del entorno. Todos los orígenes de eventos del entorno comparten esta capacidad. Si el origen de eventos de la instancia de IoT Hub o del centro de evento inserta datos por encima de los límites exigidos, experimentará limitaciones y retrasos.

En la siguiente ilustración se muestra un entorno de Azure Time Series Insights con una SKU de S1 y una capacidad de 3. Este entorno registra 3 millones de eventos al día.

[![Capacidad del entorno](media/diagnose-and-solve-problems/environment-sku-current-capacity.png)](media/diagnose-and-solve-problems/environment-sku-current-capacity.png#lightbox)

Imagine un entorno que ingiere mensajes de un centro de eventos. Tiene una tasa de entrada diaria de aproximadamente 67 000 mensajes. Esta velocidad se traduce aproximadamente en 46 mensajes por minuto.

* Si cada mensaje del centro de eventos se acopla a un evento de Azure Time Series Insights, no se produce ninguna limitación.
* Si cada mensaje del centro de eventos se acopla a 100 eventos de Azure Time Series Insights, se deben ingerir 4600 eventos por minuto.

Un entorno de SKU S1 que tiene una capacidad de 3 solo puede admitir la entrada de 2100 eventos por minuto (1 millón de eventos por día = 700 eventos por minuto por 3 unidades = 2100 eventos por minuto).

Para comprender de una manera más profunda cómo funciona la lógica de aplanamiento, consulte [Formas JSON admitidas](./how-to-shape-query-json.md).

#### <a name="recommended-resolutions-for-excessive-throttling"></a>Resoluciones recomendadas para la limitación excesiva

Para corregir el retraso, aumente la capacidad de la SKU de su entorno. Para más información, consulte [Escalado de su entorno de Azure Time Series Insights](time-series-insights-how-to-scale-your-environment.md).

### <a name="cause-b-initial-ingestion-of-historical-data-slows-ingress"></a>Causa B: la ingesta inicial de los datos históricos ralentiza la entrada

Si va a conectar un origen de eventos existente, es probable que su centro de eventos o instancia de IoT Hub ya contengan datos. El entorno comienza a extraer datos del principio del período de retención del mensaje de origen del evento. Este proceso predeterminado no se puede invalidar. Puede exponerse a limitaciones. La limitación puede tardar un rato en alcanzarse ya que ingiere datos históricos.

#### <a name="recommended-resolutions-for-large-initial-ingestion"></a>Resoluciones recomendadas para una ingesta inicial grande

Para corregir el retraso:

1. Aumente la capacidad de la SKU al valor máximo permitido (10 en este caso). Después de aumentar la capacidad, se inicia el proceso de entrada para ponerse al día mucho más rápidamente. El aumento de capacidad se le cobra. Para visualizar lo rápido que se pone al día, puede ver el gráfico de disponibilidad en el [Explorador de Azure Time Series Insights](https://insights.timeseries.azure.com).

2. Cuando se actualice el retraso, reduzca la capacidad de la SKU a la velocidad de entrada normal.

## <a name="problem-data-was-showing-previously-but-is-no-longer-showing"></a>Problema: los datos se mostraban anteriormente, pero ya no lo hacen

Si Azure Time Series Insights ya no ingiere datos, pero se siguen transmitiendo eventos a IoT Hub o al centro de eventos, tenga en cuenta esta posible causa.

### <a name="cause-a-your-hub-access-key-was-regenerated-and-your-environment-needs-to-be-updated"></a>Causa A: se ha regenerado la clave de acceso del centro y su entorno necesita actualizarse.

Este problema se produce cuando la clave proporcionada al crear el origen del evento ya no es válida. Verá la telemetría en el centro, pero ningún mensaje de entrada recibido en Azure Time Series Insights. Si no está seguro de si se ha regenerado la clave, puede buscar "crear o actualizar reglas de autorización de espacio de nombres" en el registro de actividad del centro de eventos. En el caso de una instancia de IoT Hub, busque "crear o actualizar el recurso IotHub".

Para actualizar el entorno de Azure Time Series Insights con la nueva clave, abra el recurso del centro en Azure Portal y copie la nueva clave. Vaya a su recurso de Azure Time Series Insights y seleccione **Event Sources** (Orígenes de eventos):

   [![Seleccionar orígenes de eventos](media/diagnose-and-solve-problems/update-hub-key-step-1.png)](media/diagnose-and-solve-problems/update-hub-key-step-1.png#lightbox)

Seleccione el origen o los orígenes de eventos en los que se ha detenido la ingesta, pegue la nueva clave y seleccione **Guardar**:

   [![Pegar la nueva clave](media/diagnose-and-solve-problems/update-hub-key-step-2.png)](media/diagnose-and-solve-problems/update-hub-key-step-2.png#lightbox)

## <a name="problem-the-event-sources-timestamp-property-name-setting-doesnt-work"></a>Problema: la configuración de nombre de propiedad timestamp del origen de eventos no funciona.

Asegúrese de que el valor de la propiedad timestamp que procede del origen del evento, como una cadena JSON, esté en el formato _yyyy-MM-ddTHH:mm:ss.FFFFFFFK_. Este es un ejemplo: **2008-04-12T12:53Z**.

Recuerde que el nombre de la propiedad timestamp distingue mayúsculas de minúsculas.

La manera más fácil de asegurarse de que el nombre de la propiedad timestamp se captura y funciona correctamente consiste en utilizar el Explorador de Azure Time Series Insights. En el Explorador de Azure Time Series Insights, valiéndose del gráfico, seleccione un período de tiempo después de proporcionar el nombre de la propiedad timestamp. Haga clic con el botón derecho en la selección y, a continuación, seleccione la opción **Explorar eventos**.

El primer encabezado de columna debe ser el nombre de propiedad timestamp. Junto a la palabra **Timestamp**, se mostrará **($ts)** .

No se mostrarán estos valores:

- *(abc)* : indica que Azure Time Series Insights está leyendo los valores de datos como cadenas.
- *Icono de calendario*: indica que Azure Time Series Insights está leyendo los valores de datos como valores datetime.
- *#* : indica que Azure Time Series Insights está leyendo los valores de datos como enteros.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre la [mitigación de la latencia en Azure Time Series Insights](time-series-insights-environment-mitigate-latency.md).

- Aprenda a [escalar el entorno de Azure Time Series Insights](time-series-insights-how-to-scale-your-environment.md).
