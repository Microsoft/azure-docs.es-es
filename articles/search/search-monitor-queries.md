---
title: Supervisión de consultas
titleSuffix: Azure Cognitive Search
description: Supervise las métricas de consultas para analizar el rendimiento. Recopile y analice entradas de cadenas de consulta en registros de recursos.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/18/2020
ms.openlocfilehash: a5589a46a63437fb395db280222f8a9e84775df3
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88935082"
---
# <a name="monitor-query-requests-in-azure-cognitive-search"></a>Supervisión de solicitudes de consulta en Azure Cognitive Search

En este artículo se explica cómo medir el rendimiento y el volumen de las consultas mediante métricas y el registro de recursos. También se explica cómo recopilar los términos de entrada que se usan en las consultas: información necesaria cuando se necesita evaluar la utilidad y la eficacia de los corpus de búsqueda.

Los datos históricos que se incluyen en las métricas se conservan durante 30 días. Para una retención más larga o para informar sobre datos operativos y cadenas de consulta, asegúrese de habilitar una [configuración de diagnóstico](search-monitor-logs.md) que especifique una opción de almacenamiento para conservar los eventos y métricas registrados.

Entre las condiciones que maximizan la integridad de la medición de datos se incluyen:

+ Uso de un servicio facturable (un servicio creado en el nivel Básico o Estándar). El servicio gratuito lo comparten entre varios suscriptores, lo cual introduce una cierta volatilidad ya que las cargas se desplazan.

+ Use una réplica y una partición, si es posible, para crear un entorno contenido y aislado. Si usa varias réplicas, se calcula el promedio de las métricas de consulta en varios nodos, lo que puede reducir la precisión de los resultados. Del mismo modo, el hecho de que haya varias particiones implica que los datos se dividen, con la posibilidad de que algunas particiones tengan datos diferentes si también se está llevando a cabo la indexación. A la hora de ajustar el rendimiento de las consultas, un solo nodo y una partición proporcionan un entorno más estable para las pruebas.

> [!Tip]
> Con código de cliente adicional y Application Insights puede también capturar datos click-through para obtener una visión más profunda de lo que atrae el interés de los usuarios de la aplicación. Para más información, consulte [Análisis de tráfico de búsqueda](search-traffic-analytics.md).

## <a name="query-volume-qps"></a>Volumen de consultas (QPS)

El volumen se mide en términos de **consultas de búsqueda por segundo** (QPS), una métrica integrada que se puede presentar como un promedio, un recuento, un mínimo o un número máximo de consultas que se ejecutan en un período de un minuto. Los intervalos de un minuto (TimeGrain = "PT1M") de las métricas se fijan dentro del sistema.

Es habitual que las consultas se ejecuten en cuestión de milisegundos, por lo que solo las consultas que duran segundos aparecerán en las métricas.

| Tipo de agregación | Descripción |
|------------------|-------------|
| Average | El número medio de segundos de un minuto durante el que se produjo la ejecución de la consulta.|
| Count | El número de métricas emitidas en el registro en el intervalo de un minuto. |
| Máxima | Número máximo de consultas de búsqueda por segundo registradas durante un minuto. |
| Mínima | Número mínimo de consultas de búsqueda por segundo registradas durante un minuto.  |
| Sum | La suma de todas las consultas ejecutadas dentro del minuto.  |

Por ejemplo, en el transcurso de un minuto, podría tener un patrón parecido a este: un segundo de carga elevada que es el máximo en SearchQueriesPerSecond, seguido de 58 segundos de carga media y, por último, 1 segundo con solo una consulta (que es el mínimo).

Otro ejemplo: si un nodo emite 100 métricas y el valor de cada una de las métricas es 40, "Count" es 100, "Sum" es 4000, "Average" es 40 y "Max" es 40.

## <a name="query-performance"></a>Rendimiento de las consultas

En todo el servicio, el rendimiento de las consultas se mide en términos de latencia de búsqueda (cuánto tiempo tarda una consulta en completarse) y de consultas limitadas que se quitaron como resultado de la contención de recursos.

### <a name="search-latency"></a>Latencia de búsqueda

| Tipo de agregación | Latencia | 
|------------------|---------|
| Average | Duración promedio de la consulta en milisegundos. | 
| Count | El número de métricas emitidas en el registro en el intervalo de un minuto. |
| Máxima | Consulta con la ejecución más larga del ejemplo. | 
| Mínima | Consulta con la ejecución más corta del ejemplo.  | 
| Total | Tiempo total de ejecución de todas las consultas del ejemplo que se ejecutan en el intervalo (un minuto).  |

Analice el siguiente ejemplo de métricas de **latencia de búsqueda**: se muestrearon 86 consultas, con una duración media de 23,26 milisegundos. Un mínimo de 0 indica que se eliminaron algunas consultas. La consulta con la mayor duración tardó 1000 milisegundos en completarse. El tiempo de ejecución total fue de 2 segundos.

![Agregaciones de latencia](./media/search-monitor-usage/metrics-latency.png "Agregaciones de latencia")

### <a name="throttled-queries"></a>Consultas limitadas

Las consultas limitadas se refieren a las consultas que se eliminaron en lugar de procesarse. En la mayoría de los casos, la limitación es una parte normal de la ejecución del servicio.  No necesariamente indica que se produjera algún problema.

La limitación se produce cuando el número de solicitudes procesadas actualmente supera los recursos disponibles. Puede que observe un aumento en las solicitudes limitadas cuando se saca una réplica de la rotación o durante la indexación. El mismo conjunto de recursos controla las solicitudes de consulta y de indexación.

El servicio determina si debe reducir el número de solicitudes según el consumo de los recursos. Se realiza el promedio del porcentaje de los recursos que se consume entre memoria, CPU y E/S de disco durante un período de tiempo. Si este porcentaje supera un umbral, todas las solicitudes al índice se limitan hasta que el volumen de solicitudes se reduce. 

Según el cliente, una solicitud limitada se puede indicar de estas maneras:

+ Un servicio devuelve el error "Está enviando demasiadas solicitudes. Inténtelo de nuevo más tarde." 
+ Un servicio devuelve un código de error 503 que indica que el servicio no está disponible actualmente. 
+ Si está utilizando el portal (por ejemplo, el Explorador de búsqueda), la consulta se eliminará silenciosamente y tendrá que hacer clic de nuevo en Buscar.

Para confirmar las consultas limitadas, use la métrica **Consultas de búsqueda limitadas**. Puede explorar las métricas en el portal o crear una métrica de alerta tal y como se describe en este artículo. En el caso de las consultas que se eliminaron en el intervalo de muestreo, use *Total* para obtener el porcentaje de consultas que no se ejecutaron.

| Tipo de agregación | Limitaciones |
|------------------|-----------|
| Average | Porcentaje de consultas eliminadas en el intervalo. |
| Count | El número de métricas emitidas en el registro en el intervalo de un minuto. |
| Máxima | Porcentaje de consultas eliminadas en el intervalo.|
| Mínima | Porcentaje de consultas eliminadas en el intervalo. |
| Total | Porcentaje de consultas eliminadas en el intervalo. |

En el caso de **Porcentaje de consultas de búsqueda limitadas**, "minimum", "maximum", "average" y "total" tienen el mismo valor: el porcentaje de consultas de búsqueda que se limitaron en comparación con el número total de consultas de búsqueda durante un minuto.

En la siguiente captura de pantalla, el primer número es el valor de "count" (o número de métricas enviadas al registro). Entre las agregaciones adicionales, que aparecen en la parte superior o al mantener el puntero sobre la métrica, se incluyen "average", "maximum" y "total". En este ejemplo, no se eliminó ninguna solicitud.

![Agregaciones limitadas](./media/search-monitor-usage/metrics-throttle.png "Agregaciones limitadas")

## <a name="explore-metrics-in-the-portal"></a>Exploración de las métricas en el portal

Para echar un vistazo a las cifras actuales, la pestaña **Supervisión** de la página de información general del servicio ofrece tres métricas (**Latencia de búsqueda**, **Consultas de búsqueda por segundo (por unidad de búsqueda)** , **Porcentaje de consultas de búsqueda limitadas**) en intervalos fijos que se miden en horas, días y semanas con la posibilidad de cambiar el tipo de agregación.

Para un análisis en profundidad, abra el explorador de métricas en el menú **Supervisión** para que pueda clasificar, ampliar y visualizar los datos en busca de tendencias o anomalías. Para más información sobre el explorador de métricas, realice este [tutorial sobre creación de un gráfico de métricas](../azure-monitor/learn/tutorial-metrics-explorer.md).

1. En la sección Supervisión, seleccione **Métricas** para que se abra el explorador de métricas con el ámbito establecido en el servicio de búsqueda.

1. En Métricas, elija una de la lista desplegable y revise la lista de las agregaciones disponibles para seleccionar el tipo que prefiera. La agregación define cómo se realizará el muestreo de los valores recopilados durante cada intervalo de tiempo.

   ![Explorador de métricas QPS](./media/search-monitor-usage/metrics-explorer-qps.png "Explorador de métricas QPS")

1. En la esquina superior derecha, establezca el intervalo de tiempo.

1. Elija una visualización. La opción predeterminada es un gráfico de líneas.

1. Para clasificar según las agregaciones adicionales, seleccione **Agregar métrica** y seleccione las diferentes agregaciones.

1. Acerque cualquier área de su interés en el gráfico de líneas. Coloque el puntero del mouse al principio del área, haga clic y mantenga presionado el botón izquierdo, arrastre al otro lado del área y suelte el botón. El gráfico acercará el intervalo de tiempo.

## <a name="identify-strings-used-in-queries"></a>Identificación de las cadenas utilizadas en las consultas

Cuando habilita el registro de recursos, el sistema captura las solicitudes de consulta en la tabla **AzureDiagnostics**. Como requisito previo, debe tener habilitado el [registro de recursos](search-monitor-logs.md). Para ello, especifique un área de trabajo de Log Analytics u otra opción de almacenamiento.

1. En la sección Supervisión, seleccione **Registros** para que se abra una ventana de consulta vacía en Log Analytics.

1. Ejecute la siguiente expresión para buscar operaciones Query.Search que devuelvan un conjunto de resultados tabular que conste de nombre de la operación, cadena de consulta, el índice consultado y el número de documentos que se encontró. Las dos últimas instrucciones excluyen las cadenas de consulta que consisten en una búsqueda vacía o no específica, en un índice de búsqueda, lo cual reduce el ruido en los resultados.

   ```
   AzureDiagnostics
   | project OperationName, Query_s, IndexName_s, Documents_d
   | where OperationName == "Query.Search"
   | where Query_s != "?api-version=2020-06-30&search=*"
   | where IndexName_s != "realestate-us-sample-index"
   ```

1. Opcionalmente, puede establecer *Query_s* para que busque una cadena o sintaxis específica. Por ejemplo, puede filtrar por *es igual a* `?api-version=2020-06-30&search=*&%24filter=HotelName`).

   ![Cadenas de consulta registradas](./media/search-monitor-usage/log-query-strings.png "Cadenas de consulta registradas")

Aunque esta técnica funciona para la investigación ad hoc, la creación de un informe le permite consolidar y presentar las cadenas de consulta en un diseño que facilita el análisis.

## <a name="identify-long-running-queries"></a>Identificación de consultas de larga duración

Agregue la columna de duración para obtener las cifras de todas las consultas, no solo de aquellas que se han seleccionado como métrica. La ordenación de estos datos le permite observar qué consultas tardaron más tiempo en completarse.

1. En la sección Supervisión, seleccione **Registros** para consultar la información sobre registros.

1. Ejecute la siguiente consulta para que devuelva las consultas ordenadas según la duración en milisegundos. Las consultas de mayor duración se encuentran en la parte superior.

   ```
   AzureDiagnostics
   | project OperationName, resultSignature_d, DurationMs, Query_s, Documents_d, IndexName_s
   | where OperationName == "Query.Search"
   | sort by DurationMs
   ```

   ![Ordenación de consultas según la duración](./media/search-monitor-usage/azurediagnostics-table-sortby-duration.png "Ordenación de consultas según la duración")

## <a name="create-a-metric-alert"></a>Creación de una alerta de métrica

Una alerta de métrica permite establecer un umbral a partir del cual recibirá una notificación o se desencadenará una acción correctiva definida de antemano. 

En el caso de un servicio de búsqueda, es habitual crear una alerta de métrica para la latencia de búsqueda y para las consultas limitadas. Si sabe cuándo se eliminan las consultas, puede buscar soluciones que reduzcan la carga o aumenten la capacidad. Por ejemplo, si las consultas limitadas aumentan durante la indexación, podría posponer esta hasta que la actividad de consulta disminuya.

A la hora de disponer los límites de una configuración de réplica o partición, puede resultar también útil la configuración de alertas para umbrales de volumen de consultas (QPS).

1. En la sección Supervisión, seleccione **Alertas** y, después, haga clic en **+ Nueva regla de alertas**. Asegúrese de que el servicio de búsqueda está seleccionado como recurso.

1. En Condición, haga clic en **Agregar**.

1. Configure la lógica de señal. En tipo de señal, seleccione **métricas** y, a continuación, seleccione la señal.

1. Después de seleccionar la señal, puede usar un gráfico para visualizar los datos históricos para poder tomar una decisión informada sobre cómo proceder con las condiciones de configuración.

1. A continuación, desplácese hacia abajo hasta Lógica de alerta. Para la prueba de concepto, puede especificar un valor intencionadamente bajo con fines de prueba.

   ![Lógica de alerta](./media/search-monitor-usage/alert-logic-qps.png "Lógica de alerta")

1. Después, especifique o cree un grupo de acciones. Esta será la respuesta que se invoque cuando se alcance el umbral. Puede tratarse del envío de una notificación o una respuesta automática.

1. Por último, especifique los detalles de la alerta. Asigne un nombre a la alerta y descríbala, asigne un valor de gravedad y especifique si se debe crear la alerta en un estado habilitado o deshabilitado.

   ![Detalles de alerta](./media/search-monitor-usage/alert-details.png "Detalles de alertas")

Si ha especificado el envío de una notificación por correo electrónico, recibirá un correo de "Microsoft Azure" con la línea de asunto "Azure: Gravedad activada: 3 `<your rule name>`".

<!-- ## Report query data

Power BI is an analytical reporting tool useful for visualizing data, including log information. If you are collecting data in Blob storage, a Power BI template makes it easy to spot anomalies or trends. Use this link to download the template. -->

## <a name="next-steps"></a>Pasos siguientes

Si aún no lo ha hecho, revise los aspectos fundamentales de la supervisión en el servicio de búsqueda para más información sobre la gama completa de funcionalidades de supervisión.

> [!div class="nextstepaction"]
> [Supervisión de operaciones y actividades en Azure Cognitive Search](search-monitor-usage.md)