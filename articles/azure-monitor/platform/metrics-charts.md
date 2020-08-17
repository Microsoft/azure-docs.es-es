---
title: Características avanzadas del Explorador de métricas de Azure
description: Descubra las características avanzadas del Explorador de métricas de Azure Monitor
author: vgorbenko
services: azure-monitor
ms.topic: conceptual
ms.date: 01/22/2019
ms.author: vitalyg
ms.subservice: metrics
ms.openlocfilehash: e554fe48c3cbf8b744bf2f435f7b51e424d28808
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079560"
---
# <a name="advanced-features-of-azure-metrics-explorer"></a>Características avanzadas del Explorador de métricas de Azure

> [!NOTE]
> En este artículo se da por hecho que está familiarizado con las características básicas del Explorador de métricas. Si es un nuevo usuario y desea aprender a crear su primer gráfico de métricas, consulte [Introducción al Explorador de métricas](metrics-getting-started.md).

## <a name="metrics-in-azure"></a>Métricas en Azure

Las [métricas en Azure Monitor](data-platform-metrics.md) son la serie de valores medidos y recuentos que se recopilan y se almacenan con el tiempo. Hay métricas estándar (o de la “plataforma”) y métricas personalizadas. La misma plataforma de Azure le proporciona las métricas estándares. Las métricas estándares reflejan las estadísticas de uso y mantenimiento de los recursos de Azure. Por otra parte, las métricas personalizadas se envían a Azure por medio de sus aplicaciones con la [API de Application Insights para eventos y métricas personalizados](../app/api-custom-events-metrics.md), la [extensión de Windows Azure Diagnostics (WAD)](./diagnostics-extension-overview.md), o por medio de la [API de REST de Azure Monitor](./metrics-store-custom-rest-api.md).

## <a name="create-views-with-multiple-metrics-and-charts"></a>Creación de vistas con varias métricas y gráficos

Puede crear gráficos que tracen varias líneas de métricas o mostrar varios gráficos de métricas a la vez. Esta funcionalidad le permite:

- poner en correlación métricas relacionadas en el mismo grupo para ver la relación de un valor con otro,
- mostrar métricas con diferentes unidades de medida muy cercanas o
- agregar y comparar visualmente métricas de varios recursos.

Por ejemplo, si tiene cinco cuentas de almacenamiento y desea conocer el espacio total consumido entre ellas, puede crear un gráfico de áreas (apiladas) que muestre el valor individual y la suma de todos los valores en determinados momentos.

### <a name="multiple-metrics-on-the-same-chart"></a>Varias métricas en el mismo gráfico

En primer lugar, [cree un nuevo gráfico](metrics-getting-started.md#create-your-first-metric-chart). Haga clic en **Agregar métrica** y repita los pasos para agregar otra métrica en el mismo gráfico.

   > [!NOTE]
   > Normalmente no querrá tener en un gráfico métricas con unidades de medida diferentes (es decir, milisegundos y kilobytes) ni con una escala muy diferente. En su lugar, considere el uso de varios gráficos. Haga clic en el botón Agregar gráfico para crear varios gráficos en el explorador de métricas.

### <a name="multiple-charts"></a>Varios gráficos

Haga clic en **Agregar gráfico** y cree otro gráfico con una métrica diferente.

### <a name="order-or-delete-multiple-charts"></a>Ordenación o eliminación de varios gráficos

Para ordenar o eliminar varios gráficos, haga clic en el símbolo de puntos suspensivos ( **...** ) para abrir el menú del gráfico y elija el elemento de menú adecuado de **Subir**, **Bajar** o **Eliminar**.

## <a name="changing-aggregation"></a>Cambio de la agregación

Al agregar una métrica a un gráfico, el Explorador de métricas preselecciona automáticamente la agregación predeterminada. El valor predeterminado tiene sentido en escenarios básicos, pero puede usar una agregación diferente para obtener información adicional sobre la métrica. Para ver diferentes agregaciones en un gráfico, es necesario comprender cómo las administra el Explorador de métricas. 

Las métricas son la serie de medidas (o "valores de métricas") capturadas durante el período de tiempo. Al trazar un gráfico, los valores de la métrica seleccionada se agregan por separado en el *intervalo de agregación*. Puede seleccionar el tamaño del intervalo de agregación [mediante el panel selector de tiempo del Explorador de métricas](metrics-getting-started.md#select-a-time-range). Si no realiza una selección explícita del intervalo de agregación, la granularidad del tiempo se selecciona automáticamente en función del intervalo de tiempo seleccionado actualmente. Una vez que se determina el intervalo de agregación, los valores de métricas que se capturaron durante cada intervalo de agregación se agregan y se colocan en el gráfico: un punto de bits por cada intervalo de agregación.

Por ejemplo, supongamos que el gráfico muestra la métrica **Tiempo de respuesta del servidor** con la agregación **Media** en el intervalo de tiempo **últimas 24 horas**:

- Si la granularidad del tiempo se establece en 30 minutos, el gráfico se dibuja a partir de 48 puntos de datos agregados (por ejemplo, el gráfico de líneas conecta 48 puntos en el área de trazado del gráfico). Es decir, 24 horas x 2 puntos de datos cada hora. Cada punto de datos representa la *media* de todos los tiempos de respuesta capturados para las solicitudes de servidor que se produjeron durante cada uno de los períodos de tiempo de 30 minutos correspondientes.
- Si cambia la granularidad del tiempo a 15 minutos, obtendrá 96 puntos de datos agregados.  Es decir, 24 horas x 4 puntos de datos cada hora.

Hay cinco tipos de agregaciones estadísticas básicas disponibles en el Explorador de métricas: **Suma**, **Recuento**, **Min**, **Max** y **Media**. A veces, se hace referencia a la agregación **Suma** como agregación **Total**. En el caso de muchas métricas, el Explorador de métricas ocultará las agregaciones que no son pertinentes y no se pueden usar.

- **Suma**: suma de todos los valores capturados en el intervalo de agregación.
- **Recuento**: número de medidas capturadas en el intervalo de agregación. Tenga en cuenta que **Recuento** será igual a **Suma** en el caso de que la métrica siempre se capture con el valor 1. Esto es habitual cuando la métrica realiza un seguimiento del recuento de distintos eventos y cada medida representa un evento (es decir, el código activa un registro de métricas cada vez que entra una nueva solicitud).
- **Media**: media de los valores de la métrica capturados en el intervalo de agregación.
- **Min**: el menor valor capturado en el intervalo de agregación.
- **Max**: el mayor valor capturado en el intervalo de agregación.

## <a name="apply-filters-to-charts"></a>Aplicación de filtros a gráficos

Puede aplicar filtros a los gráficos que muestran métricas con dimensiones. Por ejemplo, si la métrica “Transaction count” tiene una dimensión, “Response type”, que indica si la respuesta de las transacciones se realizó correctamente o no, el filtrado en esta dimensión trazaría una línea de gráfico solo para transacciones correctas (o solo con errores). 

### <a name="to-add-a-filter"></a>Para agregar un filtro

1. Seleccione **Agregar filtro** encima del gráfico

2. Seleccione qué dimensión (propiedad) quiere filtrar.

   ![imagen de métrica](./media/metrics-charts/00006.png)

3. Seleccione qué valores de dimensión quiere incluir al trazar el gráfico (en este ejemplo se muestra el filtrado excluyente de las transacciones de almacenamiento correctas):

   ![imagen de métrica](./media/metrics-charts/00007.png)

4. Después de seleccionar los valores de filtro, haga clic fuera del Selector de filtro para cerrarlo. Ahora el gráfico muestra cuántas transacciones de almacenamiento han tenido error:

   ![imagen de métrica](./media/metrics-charts/00008.png)

5. Puede repetir los pasos del 1 al 4 para aplicar varios filtros a los gráficos mismos.



## <a name="apply-splitting-to-a-chart"></a>Aplicación de la división a un gráfico

Puede dividir una métrica por dimensión para visualizar cómo se comparan entre sí los distintos segmentos de la métrica, y para identificar los segmentos no relevantes de una dimensión.

### <a name="apply-splitting"></a>Aplicación de la división

1. Haga clic en **Aplicar separación** encima del gráfico.
 
   > [!NOTE]
   > La división no se podrá usar con aquellos gráficos que tengan varias métricas. Asimismo, puede tener varios filtros, pero solo una dimensión de división aplicada en un único gráfico.

2. Elija una dimensión por la que quiera segmentar el gráfico:

   ![imagen de métrica](./media/metrics-charts/00010.png)

   Ahora el gráfico muestra varias líneas, una para cada segmento de dimensión:

   ![imagen de métrica](./media/metrics-charts/00012.png)

3. Haga clic fuera del **Selector de agrupación** para cerrarlo.

   > [!NOTE]
   > Use el filtrado y la separación en la misma dimensión para ocultar los segmentos que no son pertinentes para su escenario y facilitar la lectura de los gráficos.

## <a name="lock-boundaries-of-chart-y-axis"></a>Bloqueo de límites del eje y del gráfico

El bloqueo del intervalo del eje y cobra importancia cuando el gráfico muestra fluctuaciones pequeñas de valores mayores. 

Por ejemplo, cuando el volumen de solicitudes correctas cae del 99,99 % al 99,5 %, puede representar una reducción significativa de la calidad del servicio. Sin embargo, percibir una pequeña fluctuación de un valor numérico sería difícil, o incluso imposible, desde la configuración predeterminada del gráfico. En ese caso podría bloquear el límite inferior del gráfico al 99 %, lo que haría que esta pequeña caída fuera más evidente. 

Otro ejemplo es una fluctuación en la memoria disponible, donde el valor técnicamente nunca llegará a 0. El ajuste del intervalo a un valor mayor puede facilitar las detección de caídas en la memoria disponible. 

Para controlar el intervalo del eje y, use el menú "..." del gráfico y seleccione **Editar gráfico** para acceder a la configuración avanzada del gráfico. Modifique los valores en la sección Intervalo del eje Y o utilice el botón **Auto** para volver a los valores predeterminados.

![imagen de métrica](./media/metrics-charts/00014-manually-set-granularity.png)

> [!WARNING]
> El bloqueo de los límites del eje Y de los gráficos que realizan el seguimiento de varios recuentos o sumas durante un período (y, por tanto, use recuento de uso, suma y agregaciones mínima o máxima) normalmente requiere que se especifique una granularidad de tiempo fijo, en lugar de usar los valores predeterminados automáticos. Esto es necesario porque los valores de los gráficos cambian cuando la granularidad de tiempo se modifica automáticamente cuando el usuario cambia el tamaño de la ventana del explorador o pasa de una resolución de pantalla a otro. El cambio resultante en la granularidad de tiempo afecta al aspecto del gráfico, lo que invalida la selección actual del eje Y.

## <a name="change-colors-of-chart-lines"></a>Cambio de los colores de las líneas del gráfico

Después de configurar los gráficos, se asigna automáticamente a las líneas del gráfico un color de la paleta predeterminada. Puede cambiar esos colores.

Para cambiar el color de una línea del gráfico, haga clic en la barra coloreada de la leyenda correspondiente al gráfico. Se abrirá el cuadro de diálogo Selector de colores. Use el selector de colores para configurar el color de la línea.

Una vez configurados los colores del gráfico, permanecerán de este modo al anclar el gráfico a un panel. En la sección siguiente se muestra cómo anclar un gráfico.

![imagen de métrica](./media/metrics-charts/018.png)

## <a name="pin-charts-to-dashboards"></a>Anclaje de gráficos a paneles

Después de configurar un gráfico, puede que quiera agregarlo a los paneles para poder volver a verlo, posiblemente en el contexto de otra telemetría de supervisión, o compartirlo con su equipo.

Para anclar un gráfico configurado a un panel:

Después de configurar el gráfico, haga clic en el menú **Chart Actions** (Acciones del gráfico) en la esquina superior derecha del gráfico y haga clic en **Anclar al panel**.

![imagen de métrica](./media/metrics-charts/00013.png)

## <a name="create-alert-rules"></a>Creación de reglas de alerta

También puede usar los criterios que ha establecido para visualizar las métricas como la base de una regla de alerta basada en la métrica. La nueva regla de alerta incluirá su recurso de destino, métrica, división y dimensiones de filtro del gráfico. Podrá modificar esta configuración más adelante en el panel de creación de la regla de alerta.

### <a name="to-create-a-new-alert-rule-click-new-alert-rule"></a>Para crear una nueva regla de alertas, haga clic en **Nueva regla de alertas**.

![Botón de nueva regla de alertas resaltado en rojo](./media/metrics-charts/015.png)

Se le dirigirá al panel de creación de reglas de alertas con las dimensiones de métricas subyacentes desde el gráfico rellenado previamente para que resulte más fácil generar reglas de alertas personalizadas.

![Crear regla de alertas](./media/metrics-charts/016.png)

Consulte este [artículo](alerts-metric.md) para obtener más información sobre cómo configurar alertas de métricas.

## <a name="troubleshooting"></a>Solución de problemas

*No veo ningún dato en el gráfico.*

* Los filtros se aplican a todos los gráficos del panel. Asegúrese de que, al centrarse en un gráfico, no ha establecido un filtro que excluya todos los datos en otro.

* Si desea establecer filtros diferentes en gráficos diferentes, créelos en hojas diferentes y guárdelos como elementos favoritos independientes. Si lo desea, puede anclarlos al panel para verlos en paralelo.

* Si segmenta un gráfico por una propiedad que no está definida en la métrica, no habrá nada en el gráfico. Intente borrar la segmentación (división) o elija otro nombre de propiedad.

## <a name="next-steps"></a>Pasos siguientes

  Lea [Creating custom KPI dashboards](../learn/tutorial-app-dashboards.md) (Creación de paneles de KPI personalizados) para obtener información sobre las prácticas recomendadas para la creación de paneles accionables con métricas.
