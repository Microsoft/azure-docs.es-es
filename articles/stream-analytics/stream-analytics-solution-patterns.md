---
title: Patrones de soluciones de Azure Stream Analytics
description: Obtenga información acerca de los patrones de solución de Azure Stream Analytics.
author: zhongc
ms.author: zhongc
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/06/2019
ms.openlocfilehash: 80843abe130f1388a5d4081adab7b9128446763b
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65761990"
---
# <a name="azure-stream-analytics-solution-patterns"></a>Patrones de soluciones de Azure Stream Analytics

Al igual que muchos otros servicios de Azure Stream Analytics se usa mejor con otros servicios para crear una solución de extremo a otro mayor. Este artículo describen soluciones sencillas de Azure Stream Analytics y diversos patrones de arquitectura. Puede compilar en estos patrones para desarrollar soluciones más complejas. Los modelos descritos en este artículo pueden usarse en una amplia variedad de escenarios. Ejemplos de patrones específicos del escenario de están disponibles en [arquitecturas de soluciones de Azure](https://azure.microsoft.com/solutions/architecture/?product=stream-analytics).

## <a name="create-a-stream-analytics-job-with-a-real-time-dashboard"></a>Crear un trabajo de Stream Analytics con un panel en tiempo real

Con facilitar su Azure Stream Analytics uso, puede crear rápidamente paneles en tiempo real y alertas. Una solución sencilla ingiere eventos de Event Hubs o IoT Hub, y [fuentes en el panel de Power BI con un conjunto de datos de transmisión por secuencias](/power-bi/service-real-time-streaming). Para obtener más información, consulte el tutorial detallado [analizar datos de llamada de teléfono con Stream Analytics y visualizar los resultados en el panel de Power BI](stream-analytics-manage-job.md).

![Panel de Power BI de ASA](media/stream-analytics-solution-patterns/pbidashboard.png)

Esta solución se puede compilar en unos pocos minutos desde Azure portal. No hay que no implicada ninguna codificación extensa y lenguaje SQL se utiliza para expresar la lógica de negocios.

Este patrón de solución de panel en tiempo real ofrece la latencia más baja desde el origen del evento en el panel de Power BI en un explorador. Azure Stream Analytics es el único servicio de Azure con esta capacidad integrada.

## <a name="use-sql-for-dashboard"></a>Uso de SQL para el panel

El panel de Power BI ofrece una baja latencia, pero no se puede usar para generar informes de Power BI puedos completa. Es un patrón común de informes a los datos a una base de datos SQL de salida en primer lugar. A continuación, usar el conector de Power BI SQL para consultar SQL para los datos más recientes.

![Panel de ASA SQL](media/stream-analytics-solution-patterns/sqldashboard.png)

Con SQL database aporta más flexibilidad a costa de una mayor latencia. Esta solución es óptima para los trabajos con los requisitos de latencia mayores que un segundo. Con este método, puede maximizar la utilidad de Power BI a un segmento adicional y desglosar los datos para los informes. También obtendrá la flexibilidad de usar otras soluciones de panel, como Tableau.

SQL no es un almacén de datos de alto rendimiento y el rendimiento máximo para una base de datos SQL de Azure Stream Analytics es 24 MB/s. Si los orígenes de eventos de la solución generan datos a una velocidad mayor, deberá usar la lógica de procesamiento de Stream Analytics para reducir la velocidad de salida a SQL. Técnicas como el filtrado, agregados en ventanas, patrón coincidente con combinaciones temporales y se pueden usar funciones analíticas. La velocidad de salida a SQL puede optimizarse aún más mediante las técnicas descritas en [salida de Azure Stream Analytics a Azure SQL Database](stream-analytics-sql-output-perf.md).

## <a name="incorporate-real-time-insights-into-your-application-with-event-messaging"></a>Incorporar información en tiempo real en la aplicación con la mensajería de eventos

El uso de Stream Analytics en segundo lugar más popular consiste en generar alertas en tiempo real. En este patrón de la solución, la lógica de negocios en Stream Analytics se puede utilizar para detectar [patrones espaciales y temporales](stream-analytics-geospatial-functions.md) o [anomalías](stream-analytics-machine-learning-anomaly-detection.md), a continuación, generar señales de alerta. Sin embargo, a diferencia de la solución de panel donde Stream Analytics usa Power BI como un punto de conexión preferido, puede usar un número de receptores de datos intermedios. Estos receptores incluyen Event Hubs, Service Bus y Azure Functions. Como el generador de aplicación, debe decidir qué receptor de datos funciona mejor para su escenario.

Para generar alertas en el flujo de trabajo de negocio existente, se debe implementar lógica de consumidor de eventos de nivel inferior. Dado que puede implementar lógica personalizada en Azure Functions, Functions es la forma más rápida que puede realizar esta integración. Un tutorial para usar la función de Azure como la salida de trabajo de Stream Analytics puede encontrarse en [ejecutar Azure Functions desde trabajos de Azure Stream Analytics](stream-analytics-with-azure-functions.md). Las funciones de Azure también admite varios tipos de notificaciones, incluidos texto y correo electrónico. Aplicación lógica también puede utilizarse para dicha integración con Event Hubs entre Stream Analytics y la aplicación lógica.

![Aplicación de mensajería de eventos ASA](media/stream-analytics-solution-patterns/eventmessagingapp.png)

Event Hubs, por otro lado, ofrece el punto de integración más flexible. Muchos otros servicios, como el Explorador de datos de Azure y Time Series Insights, pueden consumir eventos desde Event Hubs. Los servicios se pueden conectar directamente a los receptores de Event Hubs desde Azure Stream Analytics para completar la solución. Event Hubs también es el más alto rendimiento de agente de mensajería disponible en Azure para estos escenarios de integración.

## <a name="dynamic-applications-and-websites"></a>Sitios Web y aplicaciones dinámicas

Puede crear visualizaciones personalizadas de en tiempo real, por ejemplo, panel o visualización, se asignan mediante Azure Stream Analytics y Azure SignalR Service. Uso de SignalR, los clientes web pueden actualizarse y mostrar contenido dinámico en tiempo real.

![Aplicación dinámica de ASA](media/stream-analytics-solution-patterns/dynamicapp.png)

## <a name="incorporate-real-time-insights-into-your-application-through-data-stores"></a>Incorporar información en tiempo real en la aplicación a través de los almacenes de datos

La mayoría de los servicios web y aplicaciones web de hoy en día usan un modelo de solicitud/respuesta para atender la capa de presentación. El patrón de solicitud/respuesta es fácil de crear y se pueden escalar fácilmente con el tiempo de respuesta bajos con un front-end sin estado y los almacenes escalables, como Cosmos DB.

Mayor volumen de datos suele crea cuellos de botella de rendimiento en un sistema basado en CRUD. El [solución patrón event sourcing](/azure/architecture/patterns/event-sourcing) se usa para resolver los cuellos de botella de rendimiento. Insights y patrones temporales también son difíciles y eficaz para extraer de un almacén de datos tradicionales. Los datos de gran volumen modernos a menudo aplicaciones controladas por adoptarán una arquitectura basada en el flujo de datos. Análisis de Stream de Azure como el motor de cálculo de datos en movimiento es una pieza clave en esa arquitectura.

![Aplicación de abastecimiento de eventos ASA](media/stream-analytics-solution-patterns/eventsourcingapp.png)

En este patrón de la solución, eventos se procesan y agregan en almacenes de datos mediante Azure Stream Analytics. La capa de aplicación interactúa con almacenes de datos mediante el modelo tradicional de solicitud/respuesta. Debido a la capacidad de Stream Analytics para procesar un gran número de eventos en tiempo real, la aplicación es altamente escalable sin necesidad de seguridad de la capa de almacén de datos de forma masiva. El nivel de almacén de datos es esencialmente una vista materializada en el sistema. [Salida de Azure Stream Analytics a Azure Cosmos DB](stream-analytics-documentdb-output.md) describe cómo se usa Cosmos DB como una salida de Stream Analytics.

En las aplicaciones reales donde la lógica de procesamiento es compleja y ahí está la necesidad de actualizar determinadas partes de la lógica de forma independiente, se pueden crear varios trabajos de Stream Analytics junto con Event Hubs como agente intermediario de eventos.

![Aplicación de abastecimiento de eventos complejos de ASA](media/stream-analytics-solution-patterns/eventsourcingapp2.png)

Este patrón mejora la resistencia y facilidad de uso del sistema. Sin embargo, aunque Stream Analytics garantiza un procesamiento exactamente una vez, hay una pequeña posibilidad de que se pueden conseguir eventos duplicados en los centros de eventos intermediario. Es importante para el trabajo de Stream Analytics descendente para los eventos mediante claves de lógica en una ventana lookback la eliminación de réplicas. Para obtener más información sobre la entrega de eventos, consulte [garantías de entrega de evento](/stream-analytics-query/event-delivery-guarantees-azure-stream-analytics) referencia.

## <a name="use-reference-data-for-application-customization"></a>Usar datos de referencia para la personalización de la aplicación

La característica de datos de referencia de Azure Stream Analytics está diseñada específicamente para la personalización del usuario final, como las alertas de umbral, el procesamiento de reglas, y [las geovallas](geospatial-scenarios.md). La capa de aplicación puede aceptar los cambios de parámetro y almacenarlas en una base de datos SQL. El trabajo de Stream Analytics periódicamente las consultas para los cambios de la base de datos y hace que los parámetros de personalización que sea accesible a través de una combinación de datos de referencia. Para obtener más información sobre cómo usar los datos de referencia para la personalización de la aplicación, consulte [datos de referencia SQL](sql-reference-data.md) y [instrucción join de datos de referencia](/stream-analytics-query/reference-data-join-azure-stream-analytics).

Este patrón también puede usar para implementar un motor de reglas donde se definen los umbrales de las reglas de los datos de referencia. Para obtener más información sobre las reglas, consulte [procesar reglas configurables basadas en umbrales en Azure Stream Analytics](stream-analytics-threshold-based-rules.md).

![Aplicación de datos de referencia ASA](media/stream-analytics-solution-patterns/refdataapp.png)

## <a name="add-machine-learning-to-your-real-time-insights"></a>Agregar aprendizaje automático a la información en tiempo real

Integrada de Azure Stream Analytics [modelo de detección de anomalías](stream-analytics-machine-learning-anomaly-detection.md) es una manera cómoda para introducir el aprendizaje automático a la aplicación en tiempo real. Para un intervalo más amplio de Machine Learning necesita, consulte [Azure Stream Analytics se integra con el servicio de puntuación de Azure Machine Learning](stream-analytics-machine-learning-integration-tutorial.md).

Para los usuarios avanzados que desean incorporar cursos en línea y la puntuación en la misma canalización de Stream Analytics, consulte en este ejemplo de cómo hacerlo con [regresión lineal](stream-analytics-high-frequency-trading.md).

![Aplicación de aprendizaje automático ASA](media/stream-analytics-solution-patterns/mlapp.png)

## <a name="near-real-time-data-warehousing"></a>Cerca de almacenamiento de datos en tiempo real

Otro patrón común es que los datos en tiempo real de almacenamiento, conocido también como almacén de datos de transmisión por secuencias. Además de los eventos que llegan a Event Hubs y IoT Hub desde su aplicación, [Azure Stream Analytics que se ejecutan en IoT Edge](stream-analytics-edge.md) puede utilizarse para cumplir la limpieza de datos, reducción de datos y el almacén de datos y necesidades hacia delante. Análisis de Stream que se ejecutan en IoT Edge pueden administrar correctamente en el sistema de limitación de ancho de banda y problemas de conectividad. Se puede usar el adaptador de salida SQL a la salida a SQL Data Warehouse; Sin embargo, el rendimiento máximo está limitado a 10 MB/s.

![Almacenamiento de datos ASA](media/stream-analytics-solution-patterns/datawarehousing.png)

Es una forma de mejorar el rendimiento con algún inconveniente de latencia archivar los eventos en el almacenamiento de blobs de Azure y, a continuación, [importarlos en SQL Data Warehouse con Polybase](../sql-data-warehouse/load-data-from-azure-blob-storage-using-polybase.md). Deben manualmente unir de Stream Analytics para almacenamiento de blobs de entrada y salida de almacenamiento de blobs a SQL Data Warehouse por [archivar los datos mediante la marca de tiempo](stream-analytics-custom-path-patterns-blob-storage-output.md) e importar de forma periódica.

En este patrón de uso, Azure Stream Analytics se utiliza como un motor ETL casi en tiempo real. Recién los eventos que llegan continuamente se transformar y almacenar el consumo del servicio de análisis de nivel inferior.

![Almacenamiento de datos de alto rendimiento ASA](media/stream-analytics-solution-patterns/datawarehousing2.png)

## <a name="archiving-real-time-data-for-analytics"></a>Archivar datos en tiempo real para el análisis

La mayoría actividades ciencia y análisis de datos todavía ocurrir sin conexión. Azure Stream Analytics, se pueden archivar datos a través de la salida de Azure Data Lake Store Gen2 y formatos de salida Parquet. Esta funcionalidad quita la fricción para introducir datos directamente en Azure Data Lake Analytics, Azure Databricks y Azure HDInsight. Azure Stream Analytics se utiliza como un motor ETL casi en tiempo real, en esta solución. Puede explorar los datos archivados en Data Lake mediante varios motores de proceso.

![Análisis sin conexión de ASA](media/stream-analytics-solution-patterns/offlineanalytics.png)

## <a name="use-reference-data-for-enrichment"></a>Usar datos de referencia de enriquecimiento

Enriquecimiento de datos a menudo es un requisito para los motores de ETL. Azure Stream Analytics admite el enriquecimiento de datos con [hacen referencia a datos](stream-analytics-use-reference-data.md) desde la base de datos SQL y Azure Blob storage. Enriquecimiento de datos puede realizarse para los datos llegan a Azure Data Lake y SQL Data Warehouse.

![Análisis sin conexión de ASA con enriquecimiento de datos](media/stream-analytics-solution-patterns/offlineanalytics.png)

## <a name="operationalize-insights-from-archived-data"></a>Poner la información a partir de los datos archivados

Si combina el patrón de análisis sin conexión con la aplicación en tiempo real casi patrón, puede crear un bucle de comentarios. El bucle de comentarios permite que la aplicación se ajuste automáticamente para cambiar los patrones en los datos. Este bucle de comentarios puede ser tan sencillo como cambiar el valor de umbral para las alertas, o tan complejo como el reciclaje de modelos de Machine Learning. La misma arquitectura de solución se puede aplicar a ambos trabajos ASA que se ejecutan en la nube y en IoT Edge.

![Puesta en marcha ASA insights](media/stream-analytics-solution-patterns/insightsoperationalization.png)

## <a name="how-to-monitor-asa-jobs"></a>Cómo supervisar trabajos de ASA

Se puede ejecutar un trabajo de Azure Stream Analytics 24/7 para procesar eventos de entrada continuamente en tiempo real. Su tiempo de actividad garantizado es fundamental para el estado de la aplicación global. Aunque Stream Analytics es el único servicio de análisis de streaming en el sector que ofrece un [garantía de disponibilidad del 99,9%](https://azure.microsoft.com/support/legal/sla/stream-analytics/v1_0/), todavía puede suponer cierto nivel de tiempo de inactividad. Durante años, Stream Analytics ha introducido las métricas, registros y Estados de trabajo para reflejar el estado de los trabajos. Todos ellos se exponen a través del servicio Azure Monitor y se pueden exportar más a OMS. Para obtener más información, consulte [comprender Stream Analytics del trabajo de supervisión y cómo supervisar consultas](stream-analytics-monitoring.md).

![Supervisión de ASA](media/stream-analytics-solution-patterns/monitoring.png)

Hay dos cuestiones clave para supervisar:

- [Estado de error de trabajo](job-states.md)

    En primer lugar y ante todo, deberá asegurarse de que se ejecuta el trabajo. Sin el trabajo en el estado de ejecución, no hay nuevas métricas o los registros se generan. Pueden cambiar los trabajos a un estado de error por diversos motivos, incluidos tener un alto nivel de utilización de unidades de búsqueda (es decir, se está quedando sin recursos).

- [Métricas de retraso de marca de agua](https://azure.microsoft.com/blog/new-metric-in-azure-stream-analytics-tracks-latency-of-your-streaming-pipeline/)

    Esta métrica refleja cómo lejos el procesamiento de canalización está en tiempo de reloj (segundos). Algunos del retraso se atribuye a la lógica de procesamiento inherente. Como resultado, la supervisión de la creciente tendencia es mucho más importante que el valor absoluto de supervisión. El retraso de estado estable debe tratarse con el diseño de aplicaciones, no por alertas o supervisión.

En caso de error, los registros de actividad y [los registros de diagnóstico](stream-analytics-job-diagnostic-logs.md) son los mejores lugares para empezar la búsqueda de errores.

## <a name="build-resilient-and-mission-critical-applications"></a>Compilación resistente y aplicaciones de misión crítica

Independientemente de la garantía de SLA de Azure Stream Analytics y cómo cuidado que ejecute la aplicación to-end, producen interrupciones. Si la aplicación es crítica, deberá estar preparado para interrupciones con el fin de recuperar sin contratiempos.

Para avisar a las aplicaciones, lo más importante es detectar la alerta siguiente. Puede elegir reiniciar el trabajo desde la actual tiempo cuando se recupera, omitiendo alertas anteriores. La semántica de tiempo de inicio de trabajo es por la primera hora de salida, no la primera hora de entrada. La entrada se rebobina una cantidad apropiada de tiempo para garantizar que la primera salida a la hora especificada está completo y correcto con las versiones anteriores. No obtener agregados parciales y desencadenar alertas inesperadamente como resultado.

También puede iniciar la salida de cierta cantidad de tiempo en el pasado. Las directivas de retención de Event Hubs y del centro de IoT mantenga una cantidad razonable de datos para permitir el procesamiento del pasado. El inconveniente es la rapidez puede ponerse al día a la hora actual y comenzar a generar nuevas alertas oportunas. Datos pierden su valor rápidamente con el tiempo, por lo que es importante para ponerse rápidamente a la hora actual. Hay dos maneras para ponerse al día rápidamente:

- Aprovisionar más recursos (SU) cuando se pone al día.
- Reinicie desde el momento actual.

Reinicio del actual el tiempo es fácil de hacer, con el inconveniente de dejando un vacío durante el procesamiento. Reinicio de este modo podría ser correcta para las alertas de escenarios, pero puede ser problemático para los escenarios de panel y es un que no son de inicio para el archivado y escenarios de almacenamiento de datos.

Aprovisionamiento de recursos más puede acelerar el proceso, pero el efecto de tener un aumento de velocidad de procesamiento es complejo.

- Prueba de que su trabajo es escalable a un mayor número de unidades de streaming. No todas las consultas son escalables. Deberá asegurarse de que la consulta es [paralelizado](stream-analytics-parallelization.md).

- Asegúrese de que hay suficientes particiones en el canal de subida Event Hubs o IoT Hub que se pueden agregar a más unidades de procesamiento (tu) para escalar el rendimiento de entrada. Recuerde que cada ma de Event Hubs llegar al máximo a una velocidad de salida de 2 MB/s.

- Asegúrese de que se ha aprovisionado suficientes recursos en los receptores de salida (es decir, SQL Database, Cosmos DB), por lo que no limitación el auge de salida, que a veces puede provocar que el sistema al bloqueo.

Lo más importante consiste en prever el cambio de velocidad de procesamiento, probar estos escenarios antes de entrar en producción y estar preparado para escalar el procesamiento correctamente durante el tiempo de recuperación de errores.

En el escenario de extremo que son todos los eventos de entrada retrasada [resulta posible todos los eventos retrasados se quitan](stream-analytics-time-handling.md) si ha aplicado una ventana en tiempo de ejecución que llegan a su trabajo. La eliminación de los eventos puede parecer un comportamiento misterioso al principio; Sin embargo, considerar Stream Analytics es un motor de procesamiento en tiempo real, la actividad espera eventos entrantes en esté relacionado con el tiempo de reloj. Tiene que quitar los eventos que infringen estas restricciones.

### <a name="backfilling-process"></a>Proceso de reposición

Afortunadamente, el patrón de archivado de datos anterior puede utilizarse para procesar estos eventos en tiempo de ejecución correctamente. La idea es que el trabajo de archivado procesa los eventos de entrada de hora de llegada y archiva los eventos en el depósito de momento oportuno en Azure Blob o Azure Data Lake Store con la hora del evento. No importa cómo tarde un evento llega, nunca se eliminarán. Llegará siempre en el depósito de momento oportuno. Durante la recuperación, es posible volver a procesar los eventos archivados y la reposición de los resultados en el almacén de elección.

![Reposición ASA](media/stream-analytics-solution-patterns/backfill.png)

El proceso de reposición debe realizarse con un lote sin conexión de procesamiento del sistema, lo que probablemente tiene un modelo de programación diferente de Azure Stream Analytics. Esto significa que tendrá que volver a implementar la lógica de procesamiento completo.

Para recuperar la información, es importante tener por lo menos temporalmente aprovisionar que más recursos a la salida de receptores para controlar un mayor rendimiento que en estado estable necesidades de procesamiento.

|Escenarios  |Reiniciar ahora solo desde  |Reiniciar desde la última vez que se detuvo |Reiniciar ahora + reposición con eventos archivados|
|---------|---------|---------|---------|
|**Paneles**   |Crea la brecha    |Aceptar para una pequeña interrupción    |Uso de una interrupción prolongada |
|**Alerting**   |Aceptable |Aceptar para una pequeña interrupción    |No es necesario |
|**Aplicación de abastecimiento de eventos** |Aceptable |Aceptar para una pequeña interrupción    |Uso de una interrupción prolongada |
|**Almacenamiento de datos**   |Pérdida de datos  |Aceptable |No es necesario |
|**Análisis sin conexión**  |Pérdida de datos  |Aceptable |No es necesario|

## <a name="putting-it-all-together"></a>Resumen

No es difícil imaginar que todos los patrones de soluciones mencionados anteriormente se pueden combinar en un sistema complejo de extremo a otro. El sistema combinado puede incluir paneles, alertas, aplicación de abastecimiento de eventos, almacenamiento de datos y funcionalidades de análisis sin conexión.

La clave consiste en diseñar el sistema en los patrones que admite composición, por lo que se puede compilar cada subsistema, probado, actualizado y recuperar de forma independiente.

## <a name="next-steps"></a>Pasos siguientes

Ahora ha visto una variedad de patrones de la solución mediante Azure Stream Analytics. A continuación, puede profundizar y crear su primer trabajo de Stream Analytics:

* [Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md).
* [Creación de un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md).
* [Creación de un trabajo de Stream Analytics mediante Visual Studio](stream-analytics-quick-create-vs.md).
