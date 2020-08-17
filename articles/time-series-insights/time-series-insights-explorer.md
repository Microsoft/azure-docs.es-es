---
title: Exploración de los datos con el Explorador de Azure Time Series Insights | Microsoft Docs
description: Aprenda a usar el Explorador de Azure Time Series Insights para ver los datos de IoT.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: v-mamcge, jasonh, kfile
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: seodec18
ms.openlocfilehash: e78b5118a2525fbf10a3089712f75e44cb7deceb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87080699"
---
# <a name="azure-time-series-insights-gen1-explorer"></a>Explorador de Azure Time Series Insights Gen1

En este artículo se describen las características y opciones de la [aplicación web del Explorador](https://insights.timeseries.azure.com/) de Azure Time Series Insights Gen1. El Explorador de Azure Time Series Insights demuestra las poderosas funciones de visualización de datos que proporciona el servicio y que son accesibles dentro de su propio entorno.

Azure Time Series Insights es un servicio de análisis, almacenamiento y administración totalmente administrado que facilita el análisis y la exploración de miles de millones de eventos de IoT de forma simultánea. Ofrece una visión global de los datos que permite validar rápidamente la solución de IoT y evitar el costoso tiempo de inactividad de los dispositivos críticos. Puede detectar las tendencias ocultas, descubrir anomalías y realizar análisis de causa principal casi en tiempo real. 

> [!TIP]
> Para obtener una visita guiada a través del entorno de demostración, lea el [Inicio rápido de Azure Time Series Insights](time-series-quickstart.md).

## <a name="video"></a>Vídeo

### <a name="learn-about-querying-data-by-using-the-azure-time-series-insights-explorer-br"></a>Aprenda más sobre la consulta de datos mediante el Explorador de Azure Time Series Insights. </br>

> [!VIDEO https://www.youtube.com/embed/SHFPZvrR71s]

>[!NOTE]
>Reproduzca el vídeo anterior, <a href="https://www.youtube.com/watch?v=6ehNf6AJkFo">"Introducción a Azure Time Series Insights mediante un acelerador de soluciones de Azure IoT"</a>.

## <a name="prerequisites"></a>Requisitos previos

Antes de poder usar el Explorador de Azure Time Series Insights, debe hacer lo siguiente:

- Crear un entorno de Azure Time Series Insights. Para más información, lea [Introducción a Azure Time Series Insights](./time-series-insights-get-started.md).
- [Proporcionar acceso](time-series-insights-data-access.md) a la cuenta en el entorno.
- Agregarle como origen de evento un [centro de IoT](time-series-insights-how-to-add-an-event-source-iothub.md) o un [centro de eventos](time-series-insights-how-to-add-an-event-source-eventhub.md).

## <a name="explore-and-query-data"></a>Explorar y consultar los datos

Minutos después de conectar el origen del evento al entorno de Azure Times Series Insights, puede explorar y consultar los datos de serie temporal.

1. Para comenzar, abra el [Explorador de Azure Time Series Insights](https://insights.timeseries.azure.com/) en el explorador web. En el lado izquierdo de la ventana, seleccione un entorno. Todos los entornos a los que tiene acceso se muestran en orden alfabético.

1. Una vez que selecciona un entorno, use las configuraciones **From** (Desde) y **To** (Hasta) de la parte superior, o bien seleccione y arrastre hasta el intervalo de tiempo que quiera. Seleccione la lupa que se encuentra en la parte superior derecha o haga clic con el botón derecho en el intervalo de tiempo seleccionado y seleccione **Search** (Buscar).

1. También puede actualizar automáticamente la disponibilidad cada minuto si selecciona el botón **Auto On** (Activación automática). El botón **Auto On** (Activación automática) solo se aplica al gráfico de disponibilidad, no al contenido de la visualización principal.

1. El icono de la nube de Azure lo dirige a su entorno en Azure Portal.

   [![Selección del entorno de Azure Time Series Insights](media/time-series-insights-explorer/tsi-ga-explorer-environments.png)](media/time-series-insights-explorer/tsi-ga-explorer-environments.png#lightbox)

1. Después aparece un gráfico en el que se muestra un recuento de todos los eventos durante el intervalo de tiempo seleccionado. A continuación tiene varios controles:

    - **Panel Editor de términos**: el espacio para los términos es donde consulta al entorno. Se encuentra en el lado izquierdo de la pantalla:
      - **MEDIDA**: este menú desplegable muestra todas las columnas numéricas (**Doubles**).
      - **DIVIDIR POR**: este menú desplegable muestra las columnas de categoría (**Strings**).
      - Puede habilitar la interpolación escalonada, mostrar el valor mínimo y el valor máximo, y ajustar el eje Y desde el panel de control que está junto a **MEDIDA**. También puede ajustar si los datos que se muestran son un recuento, un promedio o la suma de los datos.
      - Puede agregar hasta cinco términos para verlos en el mismo eje X. Seleccione **Agregar** para agregar un término nuevo o use el botón **Clonar este término** para agregar una copia de un término existente.

        [![Selección de términos, filtrado y panel de consulta](media/time-series-insights-explorer/tsi-ga-explorer-add-or-clone.png)](media/time-series-insights-explorer/tsi-ga-explorer-add-or-clone.png#lightbox)

      - **Predicado**: use el predicado para filtrar rápidamente los eventos mediante el conjunto de operandos que se enumeran en la siguiente tabla. Si realiza una búsqueda mediante selección o clic, el predicado se actualizará automáticamente según esa búsqueda. Los tipos de operando compatibles incluyen:

         |Operación  |Tipos admitidos  |Notas  |
         |---------|---------|---------|
         |**<**, **>**, **<=**, **>=**    |  **Double**, **DateTime**, **TimeSpan**       |         |
         |**=** , **!=** , **<>**     | **String**, **Bool**, **Double**, **DateTime**, **TimeSpan**, **NULL**        |         |
         |**IN**     | **String**, **Bool**, **Double**, **DateTime**, **TimeSpan**, **NULL**        |  Todos los operandos deben ser del mismo tipo o deben ser una constante **NULL**.        |
         |**HAS**     | **String**        |  Solo los literales de cadena constantes se permiten en el lado derecho. No se permiten cadenas vacías ni con valores **NULL**.       |

      - **Consultas de ejemplo**

         [![Ejemplo de consultas de Gen1](media/time-series-insights-explorer/tsi-ga-example-queries.png)](media/time-series-insights-explorer/tsi-ga-example-queries.png#lightbox)

1. Puede usar la herramienta de control deslizante **Interval Size** (Tamaño del intervalo) para acercar y alejar los intervalos del mismo intervalo de tiempo. El control deslizante proporciona un control más preciso del movimiento entre intervalos de tiempo de gran tamaño que muestran tendencias uniformes hasta intervalos tan pequeños como un milisegundo, lo que permite mostrar y analizar cortes pormenorizados y de alta resolución de los datos. El punto inicial predeterminado del control deslizante está establecido como la vista más óptima de los datos de la selección para equilibrar la resolución, la velocidad de la consulta y la granularidad.

1. La herramienta **Time brush** (Pincel de tiempo) facilita el paso de un intervalo de tiempo a otro.

1. Seleccione el icono **Guardar** para guardar la consulta actual y compartirla con otros usuarios del entorno. Al seleccionar el icono **Abrir**, se pueden revisar todas las consultas guardadas y cualquier consulta compartida de otros usuarios en los entornos a los que tiene acceso.

   [![Consultas](media/time-series-insights-explorer/tsi-ga-explorer-saved-queries.png)](media/time-series-insights-explorer/tsi-ga-explorer-saved-queries.png#lightbox)

## <a name="visualize-data"></a>Visualización de datos

1. Use la herramienta **Perspective View** (Vista de perspectiva) para obtener una vista simultánea de hasta cuatro consultas únicas. El botón **Perspective View** (Vista de perspectiva) está en la esquina superior derecha del gráfico.

   [![Selección de las consultas que se van a agregar al panel perspectiva](media/time-series-insights-explorer/tsi-ga-explorer-perspective-panes.png)](media/time-series-insights-explorer/tsi-ga-explorer-perspective-panes.png#lightbox)

1. Para ver un gráfico para explorar los datos visualmente y usar las herramientas **Chart** (Gráfico):

    - **Seleccione** o **haga clic** en un intervalo de tiempo específico o una serie de datos.
    - Dentro de la selección de un intervalo de tiempo, puede acercar o explorar los eventos.
    - Dentro de una serie de datos, puede dividir la serie por otra columna, agregar la serie como término nuevo, mostrar solo la serie seleccionada, excluir la serie seleccionada, anclar esa serie o explorar los eventos de la serie seleccionada.
    - En el área de filtro situada a la izquierda del gráfico, puede revisar todas las series de datos que aparecen y reordenar por nombre o valor. También puede ver todas las series de datos o las series ancladas o desancladas. Puede seleccionar una serie de datos única y dividirla por otra columna, agregar la serie como un término nuevo, mostrar solo la serie seleccionada, excluirla, anclar esa serie o explorar los eventos de la serie seleccionada.
    - Cuando vea varios términos de manera simultánea, puede apilarlos, desapilarlos, revisar datos adicionales sobre una serie de datos y usar el mismo eje Y en todos los términos. Use los botones de la esquina superior derecha del gráfico.

    [![Configuración de la opción de esquina superior derecha de la herramienta de gráficos](media/time-series-insights-explorer/tsi-ga-example-chart-options.png)](media/time-series-insights-explorer/tsi-ga-example-chart-options.png#lightbox)

1. Use el **mapa térmico** para detectar rápidamente una serie de datos única o anómala en una consulta determinada. Solo un término de búsqueda se puede visualizar como un mapa térmico.

    [![Gráficos de mapa térmico del Explorador de Gen1](media/time-series-insights-explorer/tsi-ga-example-heatmap-charting.png)](media/time-series-insights-explorer/tsi-ga-example-heatmap-charting.png#lightbox)

1. Cuando se exploran eventos seleccionándolos o haciendo clic en ellos con el botón derecho, aparece disponible el panel **EVENTOS**. Aquí, puede revisar todos los eventos sin procesar y exportar los eventos como archivos JSON o CSV. Azure Time Series Insights almacena todos los datos sin procesar.

    [![Eventos](media/time-series-insights-explorer/tsi-ga-explorer-events-panel.png)](media/time-series-insights-explorer/tsi-ga-explorer-events-panel.png#lightbox)

1. Seleccione la pestaña **STATS** (Estadísticas) después de explorar los eventos para exponer los patrones y las estadísticas de columna.

    - **Patrones**: esta característica muestra de manera proactiva los patrones con mayor importancia estadística de una región de datos seleccionada. No es necesario buscar en miles de eventos para comprender cuáles son los patrones que requieren más tiempo y energía. Con Azure Time Series Insights, puede ir directamente a estos patrones con importancia estadística para seguir realizando el análisis. Esta característica también resulta útil para las investigaciones de análisis final de datos históricos.
    - **Estadísticas de columna**: las estadísticas de columna proporcionan gráficos y tablas que desglosan los datos de cada columna de la serie de datos seleccionada durante el intervalo de tiempo seleccionado.

      [![Opciones y gráficos de columnas de ESTADÍSTICAS](media/time-series-insights-explorer/tsi-ga-explorer-stat-column.png)](media/time-series-insights-explorer/tsi-ga-explorer-stat-column.png#lightbox)

Ahora ya conoce las características clave, la configuración y las opciones de visualización que están disponibles en la aplicación web del Explorador de Azure Time Series Insights.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [diagnosticar y solucionar problemas](time-series-insights-diagnose-and-solve-problems.md) en el entorno de Azure Time Series Insights.

- Siga la visita guiada del [Inicio rápido de Azure Time Series Insights](time-series-quickstart.md).
