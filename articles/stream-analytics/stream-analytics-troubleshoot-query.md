---
title: Solución de problemas en las consultas de Azure Stream Analytics
description: En este artículo se describen técnicas para solucionar problemas de las consultas en los trabajos de Azure Stream Analytics.
author: sidram
ms.author: sidram
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: troubleshooting
ms.date: 03/31/2020
ms.custom: seodec18
ms.openlocfilehash: ead175cbcaa9467cb5263ad95100facdda096991
ms.sourcegitcommit: 46f8457ccb224eb000799ec81ed5b3ea93a6f06f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87337813"
---
# <a name="troubleshoot-azure-stream-analytics-queries"></a>Solución de problemas en las consultas de Azure Stream Analytics

En este artículo se describen problemas comunes con el desarrollo de consultas a Stream Analytics y cómo solucionarlos.

En este artículo se describen los problemas comunes relacionados con el desarrollo de consultas de Azure Stream Analytics, cómo solucionar problemas con las consultas y cómo corregir estos problemas. Muchos pasos para solucionar problemas requieren que los registros de recursos estén habilitados para el trabajo de Stream Analytics. Si los registros de recursos no están habilitados, vea [Solución de problemas de Azure Stream Analytics mediante registros de recursos](stream-analytics-job-diagnostic-logs.md).

## <a name="query-is-not-producing-expected-output"></a>La consulta no produce el resultado esperado

1.  Examine los errores mediante la comprobación local:

    - En Azure Portal, en la pestaña **Consulta**, seleccione **Probar**. Use los datos de ejemplo que descargó para [probar la consulta](stream-analytics-test-query.md). Examine los errores e intente corregirlos.   
    - También puede [probar la consulta localmente](stream-analytics-live-data-local-testing.md) mediante las herramientas de Stream Analytics para Visual Studio o [Visual Studio Code](visual-studio-code-local-run-live-input.md). 

2.  [Depure las consultas paso a paso localmente mediante un diagrama de trabajo](debug-locally-using-job-diagram-vs-code.md) con las herramientas de Azure Stream Analytics para Visual Studio Code. El diagrama de trabajos muestra cómo fluyen los datos de los orígenes de entrada (centro de eventos, IoT Hub, etc.) por varios pasos de consulta y, por último, a los receptores de salida. Cada paso de la consulta se asigna a un conjunto de resultados temporal definido en el script mediante la instrucción WITH. Puede ver los datos, así como las métricas, en cada uno de los conjuntos de resultados intermedios para encontrar el origen del problema.

    ![Resultado de la vista previa del diagrama de trabajos](./media/debug-locally-using-job-diagram-vs-code/preview-result.png)

3.  Si usa [**Marca de tiempo por**](https://docs.microsoft.com/stream-analytics-query/timestamp-by-azure-stream-analytics), asegúrese de que los eventos tengan marcas de tiempo mayores que la [hora de inicio del trabajo](stream-analytics-out-of-order-and-late-events.md).

4.  Elimine los errores comunes, como:
    - Una cláusula [**WHERE**](https://docs.microsoft.com/stream-analytics-query/where-azure-stream-analytics) en la consulta filtrada por todos los eventos, que impide que se genere alguna salida.
    - Un error de una función [**CAST**](https://docs.microsoft.com/stream-analytics-query/cast-azure-stream-analytics), que provoca que el trabajo no se realice. Para evitar errores de conversión de tipo, utilice [**TRY_CAST**](https://docs.microsoft.com/stream-analytics-query/try-cast-azure-stream-analytics) en su lugar.
    - Cuando use funciones de ventana, espere la duración entera de la ventana para ver una salida de la consulta.
    - La marca de tiempo de los eventos es anterior a la hora de inicio del trabajo y los eventos se descartan.
    - Las condiciones [**JOIN**](https://docs.microsoft.com/stream-analytics-query/join-azure-stream-analytics) no coinciden. Si no hay ninguna coincidencia, no habrá resultados.

5.  Asegúrese de que las directivas de ordenación de eventos están configuradas según lo previsto. Vaya a **Configuración** y seleccione [**Ordenación de eventos**](stream-analytics-out-of-order-and-late-events.md). La directiva *no* se aplica cuando usa el botón **Probar** para probar la consulta. El resultado es una diferencia entre probar en el explorador frente a ejecutar el trabajo de producción. 

6. Use registros de recursos y actividad para la depuración:
    - Use [Registros de actividad](../azure-resource-manager/resource-group-audit.md) y filtre para identificar y depurar errores.
    - Use [registros de recursos de trabajo](stream-analytics-job-diagnostic-logs.md) para identificar y depurar errores.

## <a name="resource-utilization-is-high"></a>El uso de recursos es alto

Asegúrese de estar aprovechando la paralelización en Azure Stream Analytics. Puede aprender a [escalar con la paralelización de consultas](stream-analytics-parallelization.md) de los trabajos de Stream Analytics mediante la configuración de particiones de entrada y el ajuste de la definición de consultas de análisis.

## <a name="debug-queries-progressively"></a>Depuración gradual de consultas

En el procesamiento de datos en tiempo real, puede ser útil conocer qué aspecto tendrán los datos en el medio de la consulta. Puede verlo mediante el diagrama de trabajos de Visual Studio. Si no tiene Visual Studio, puede seguir otros pasos para generar datos intermedios.

Como las entradas o pasos de un trabajo de Azure Stream Analytics se pueden leer muchas veces, puede escribir instrucciones SELECT INTO adicionales. Si lo hace, se generan daros intermedios en el almacenamiento y puede inspeccionar la integridad de los datos, igual que hacen las *variables de inspección* cuando se depura un programa.

La siguiente consulta de ejemplo en un trabajo de Azure Stream Analytics tiene una entrada de secuencias, dos entradas de datos de referencia y una salida a Azure Table Storage. La consulta combina datos del centro de eventos y dos blobs de referencia para obtener la información de nombre y categoría:

![Ejemplo de consulta SELECT INTO de Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-query1.png)

Tenga en cuenta que el trabajo está en ejecución, pero no se genera ningún evento en la salida. En el icono **Supervisión**, que se muestra aquí, puede ver que la entrada genera datos, pero no sabe qué paso de la cláusula **JOIN** ha hecho que se descarten todos los eventos.

![Icono de Supervisión de Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-monitor.png)

En esta situación, puede agregar unas cuantas instrucciones SELECT INTO adicionales para "registrar" los resultados intermedios de JOIN y los datos que se leen de la entrada.

En este ejemplo, hemos agregado dos nuevas "salidas temporales". Estas pueden ser cualquier receptor que desee. Aquí usamos Azure Storage como ejemplo:

![Adición de instrucciones SELECT INTO adicionales a la consulta de Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-outputs.png)

A continuación, puede volver a escribir la consulta del modo siguiente:

![Consulta SELECT INTO reescrita de Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-query2.png)

Ahora, inicie de nuevo el trabajo y deje que se ejecute durante unos minutos. A continuación, consulte temp1 y temp2 con Visual Studio Cloud Explorer para generar las siguientes tablas:

**Tabla temp1**
![tabla temp1 de la consulta SELECT INTO de Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-temp-table-1.png)

**Tabla temp2**
![tabla temp2 de la consulta SELECT INTO de Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-temp-table-2.png)

Como puede ver, temp1 y temp2 tienen ambas datos, y la columna de nombre está rellenada correctamente en temp2. Sin embargo, dado que todavía no hay ningún dato en la salida, algo sucede:

![Tabla output1 sin datos de la consulta SELECT INTO de Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-out-table-1.png)

Mediante el muestreo de los datos, puede estar casi seguro de que el problema está relacionado con la segunda cláusula JOIN. Puede descargar los datos de referencia del blob y echar un vistazo:

![Tabla de referencia de la consulta SELECT INTO de Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-ref-table-1.png)

Como puede ver, el formato del GUID en los datos de esta referencia es distinto del formato de la columna [from] en temp2. Por eso los datos no llegan a output1 como estaba previsto.

Puede corregir el formato de datos, cargarlo para que haga referencia al blob e intentarlo de nuevo:

![Tabla temporal de la consulta SELECT INTO de Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-ref-table-2.png)

Esta vez, los datos en la salida tienen el formato adecuado y se han rellenado según lo previsto.

![Tabla final de la consulta SELECT INTO de Stream Analytics](./media/stream-analytics-select-into/stream-analytics-select-into-final-table.png)

## <a name="get-help"></a>Obtener ayuda

Para más ayuda, pruebe nuestra [Página de preguntas y respuestas de Microsoft sobre Azure Stream Analytics](https://docs.microsoft.com/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Pasos siguientes

* [Introducción a Azure Stream Analytics](stream-analytics-introduction.md)
* [Introducción al uso de Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Escalación de trabajos de Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Referencia del lenguaje de consulta de Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/stream-analytics-query-language-reference)
* [Referencia de API de REST de administración de Azure Stream Analytics](https://msdn.microsoft.com/library/azure/dn835031.aspx)
