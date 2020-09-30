---
title: Depuración de las consultas de Azure Stream Analytics localmente mediante un diagrama de trabajos de Visual Studio Code
description: En este artículo se describe cómo depurar las consultas localmente mediante un diagrama de trabajos en la extensión de Azure Stream Analytics para Visual Studio Code.
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 06/23/2020
ms.openlocfilehash: f8885229c8ab6784c6c0bf45487812fcc43f9b22
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90903830"
---
# <a name="debug-azure-stream-analytics-queries-locally-using-job-diagram-in-visual-studio-code"></a>Depuración de las consultas de Azure Stream Analytics localmente mediante un diagrama de trabajos de Visual Studio Code

Los trabajos de streaming que no generan ningún resultado ni resultados inesperados suelen requerir la solución de problemas. La extensión de Visual Studio Code para Azure Stream Analytics integra diagramas de trabajos, métricas, registros de diagnóstico y resultados intermedios que le ayudarán a aislar rápidamente el origen de un problema. Puede usar el diagrama de trabajos mientras prueba la consulta localmente para examinar el conjunto de resultados intermedio y las métricas de cada paso.

## <a name="debug-a-query-using-job-diagram"></a>Depuración de una consulta mediante un diagrama de trabajos

Se utiliza un script de Azure Stream Analytics para transformar los datos de entrada en datos de salida. El diagrama de trabajos muestra cómo fluyen los datos de los orígenes de entrada, como el centro de eventos o IoT Hub, por varios pasos de consulta a los receptores de salida. Cada paso de la consulta se asigna a un conjunto de resultados temporal definido en el script mediante una instrucción `WITH`. Puede ver los datos, así como las métricas de cada paso de consulta, en cada uno de los conjuntos de resultados intermedios para encontrar el origen de un problema.

> [!NOTE]
> En este diagrama de trabajos solo se muestran los datos y las métricas para las pruebas locales en un único nodo. No debe usarse para el ajuste del rendimiento y la solución de problemas.

### <a name="start-local-testing"></a>Inicio de las pruebas locales

Use este [inicio rápido](quick-create-visual-studio-code.md) para aprender a crear un trabajo de Stream Analytics mediante Visual Studio Code o [exportar un trabajo existente a un proyecto local](visual-studio-code-explore-jobs.md). Las credenciales para entradas y salidas se rellenan automáticamente para los trabajos exportados.

Si desea probar la consulta con datos de entrada locales, siga estas [instrucciones](visual-studio-code-local-run.md). Si desea probar con una entrada activa, [configure su entrada](stream-analytics-add-inputs.md) vaya al paso siguiente. 

Abra el archivo de script *\.asaql* y seleccione **Ejecutar de forma local**. A continuación, seleccione **Usar entrada local** o **Usar entrada activa**. El diagrama de trabajos aparece a la derecha de la ventana.

### <a name="view-the-output-and-intermediate-result-set"></a>Visualización del conjunto de resultados intermedio y de salida  

1. Todas las salidas de trabajo se muestran en la ventana de resultados en el lado inferior derecho de la ventana de Visual Studio Code.

   > [!div class="mx-imgBorder"]
   > ![Resultados de salida de trabajo](./media/debug-locally-using-job-diagram-vs-code/job-output-results.png)

2. Seleccione el paso de consulta para desplazarse al script. Se le dirigirá automáticamente al script correspondiente en el editor situado a la izquierda. El resultado intermedio aparece en la ventana de resultados en el lado inferior derecho de la ventana de Visual Studio Code.

   > [!div class="mx-imgBorder"]
   > ![Resultado de la vista previa del diagrama de trabajo](./media/debug-locally-using-job-diagram-vs-code/preview-result.png)

### <a name="view-metrics"></a>Visualización de métricas

En esta sección, explorará las métricas disponibles para cada parte del diagrama.

1. Seleccione la pestaña **Métricas** situada junto a la pestaña **Resultado** en el lado inferior derecho de la ventana de Visual Studio Code.

2. Seleccione **Trabajo** en la lista desplegable. Puede seleccionar cualquier espacio vacío en un nodo de gráfico para navegar a las métricas de nivel de trabajo. Esta vista contiene todas las métricas, que se actualizan cada 10 segundos cuando se ejecuta el trabajo. Puede seleccionar o anular la selección de las métricas en el lado derecho para verlas en los gráficos.

   > [!div class="mx-imgBorder"]
   > ![Métricas del diagrama de trabajos](./media/debug-locally-using-job-diagram-vs-code/job-metrics.png)

3. Seleccione el nombre del origen de datos de entrada en la lista desplegable para ver las métricas de entrada. El origen de entrada de la siguiente captura de pantalla se denomina *comillas*. Para más información sobre las métricas de entrada, consulte [Descripción de la supervisión del trabajo de Stream Analytics y cómo supervisar consultas](stream-analytics-monitoring.md).

   > [!div class="mx-imgBorder"]
   > ![Métricas de entrada del diagrama de trabajo](./media/debug-locally-using-job-diagram-vs-code/input-metrics.png)

4. Seleccione un paso de consulta en el diagrama de trabajos o seleccione el nombre del paso en la lista desplegable para ver las métricas de nivel de paso. Retraso de la marca de agua es la única métrica de paso disponible.

   > [!div class="mx-imgBorder"]
   > ![Métricas de los pasos](./media/debug-locally-using-job-diagram-vs-code/step-metrics.png)

5. Seleccione una salida en el diagrama o en la lista desplegable para ver las métricas relacionadas con la salida. Para más información sobre las métricas de salida, consulte [Descripción de la supervisión del trabajo de Stream Analytics y cómo supervisar consultas](stream-analytics-monitoring.md). Los receptores de salida en directo no se admiten.

   > [!div class="mx-imgBorder"]
   > ![Métricas de salida](./media/debug-locally-using-job-diagram-vs-code/output-metrics.png)

### <a name="view-diagnostic-logs"></a>Visualización de los registros de diagnóstico

Los registros de diagnóstico de nivel de trabajo contienen información de diagnóstico para los orígenes de datos de entrada y los receptores de salida. Al seleccionar un nodo de entrada o de salida, solo se mostrarán los registros correspondientes. No se mostrará ningún registro si selecciona un paso de consulta. Puede buscar todos los registros en el nivel de trabajo, así como filtrar los registros por gravedad y hora.

   > [!div class="mx-imgBorder"]
   > ![Registros de diagnóstico](./media/debug-locally-using-job-diagram-vs-code/diagnostic-logs.png)

   Seleccione una entrada de registro para ver todo el mensaje.

   > [!div class="mx-imgBorder"]
   > ![Mensaje de registros de diagnóstico](./media/debug-locally-using-job-diagram-vs-code/diagnostic-logs-message.png)


## <a name="other-job-diagram-features"></a>Otras características del diagrama de trabajos

Puede seleccionar **Detener** o **Pausar** en la barra de herramientas según sea necesario. Una vez que el trabajo se haya pausado, podrá reanudarlo desde la última salida.

> [!div class="mx-imgBorder"]
> ![Detener o pausar el trabajo](./media/debug-locally-using-job-diagram-vs-code/stop-pause-job.png)

Seleccione **Resumen de trabajos** en la parte superior derecha del diagrama de trabajos para ver las propiedades y las configuraciones del trabajo local.

> [!div class="mx-imgBorder"]
> ![Resumen de trabajos locales](./media/debug-locally-using-job-diagram-vs-code/job-summary.png)

## <a name="limitations"></a>Limitaciones

* Los receptores de salida en directo no se admiten en la ejecución local.

* Ejecutar el trabajo localmente con la función de JavaScript solo se admite en el sistema operativo Windows.

* El código personalizado C# y las funciones de Azure Machine Learning no se admiten. 

* Solo las opciones de entrada de nube tienen compatibilidad con [directivas de tiempo](stream-analytics-out-of-order-and-late-events.md), lo que no ocurre con las opciones de entrada locales.

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: Crear un trabajo de Stream Analytics con Visual Studio Code](quick-create-visual-studio-code.md)
* [Exploración de Azure Stream Analytics con Visual Studio Code](visual-studio-code-explore-jobs.md)
* [Prueba de consultas de Stream Analytics localmente con datos de ejemplo mediante Visual Studio Code](visual-studio-code-local-run.md)
* [Prueba de los trabajos de Azure Stream Analytics localmente con una entrada activa mediante Visual Studio Code](visual-studio-code-local-run-live-input.md)
