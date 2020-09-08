---
title: Patrones de soluciones de Azure Stream Analytics
description: Obtenga información sobre los patrones de soluciones comunes para Azure Stream Analytics, como los paneles, la mensajería de eventos, los almacenes de datos, el enriquecimiento de datos de referencia y la supervisión.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 06/21/2019
ms.openlocfilehash: c3d487c1595a077ac8609813a41d15e28ede0e0b
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87903330"
---
# <a name="azure-stream-analytics-solution-patterns"></a>Patrones de soluciones de Azure Stream Analytics

Al igual que muchos otros servicios de Azure, Stream Analytics se usa mejor con otros servicios para crear una solución de extremo a otro más amplia. En este artículo se describen soluciones sencillas de Azure Stream Analytics y diversos patrones de arquitectura. Puede ampliar estos patrones para desarrollar soluciones más complejas. Los patrones que se describen en este artículo pueden usarse en una amplia variedad de escenarios. Encontrará ejemplos de patrones específicos del escenario en las [arquitecturas de soluciones de Azure](https://azure.microsoft.com/solutions/architecture/?product=stream-analytics).

## <a name="create-a-stream-analytics-job-to-power-real-time-dashboarding-experience"></a>Creación de un trabajo de Stream Analytics para impulsar la experiencia de paneles en tiempo real

Con Azure Stream Analytics, puede crear rápidamente paneles y alertas en tiempo real. Una solución sencilla ingiere eventos desde Event Hubs o IoT Hub y [alimenta el panel de Power BI con un conjunto de datos de streaming](/power-bi/service-real-time-streaming). Para más información, consulte el tutorial detallado [Analizar los datos de llamadas de teléfono con Stream Analytics y visualizar los resultados en un panel de Power BI](stream-analytics-manage-job.md).

![Panel de Power BI ASA](media/stream-analytics-solution-patterns/power-bi-dashboard.png)

Esta solución se puede compilar en unos pocos minutos desde Azure Portal. No se necesita codificación extensa y se usa el lenguaje SQL para expresar la lógica de negocios.

Este patrón de solución ofrece la latencia más baja desde el origen del evento en el panel de Power BI en un explorador. Azure Stream Analytics es el único servicio de Azure con esta capacidad integrada.

## <a name="use-sql-for-dashboard"></a>Uso de SQL para el panel

El panel de Power BI ofrece una baja latencia, pero no se puede usar para generar informes de Power BI completos. Un patrón de informes común consiste en generar primero la salida de los datos en SQL Database. Luego, use el conector SQL de Power BI para consultar SQL en busca de los datos más recientes.

![Panel SQL ASA](media/stream-analytics-solution-patterns/sql-dashboard.png)

El uso de SQL Database ofrece mayor flexibilidad, pero a costa de una latencia ligeramente mayor. Esta solución es óptima para los trabajos con requisitos de latencia mayores que un segundo. Con este método, puede maximizar las capacidades de Power BI para segmentar y desglosar aún más los datos para los informes y disponer de muchas más opciones de visualización. También obtendrá la flexibilidad de usar otras soluciones de panel, como Tableau.

SQL no es un almacén de datos de alto rendimiento. El rendimiento máximo de SQL Database en Azure Stream Analytics es actualmente de aproximadamente 24 MB/s. Si los orígenes de evento de la solución generan datos a mayor velocidad, tendrá que usar la lógica de procesamiento de Stream Analytics para reducir la velocidad de salida a SQL. Se pueden usar técnicas, como el filtrado, agregados en ventanas, coincidencia de patrones con combinaciones temporales y funciones analíticas. La velocidad de salida a SQL puede optimizarse aún más mediante las técnicas que se describen en [Salida de Azure Stream Analytics a Azure SQL Database](stream-analytics-sql-output-perf.md).

## <a name="incorporate-real-time-insights-into-your-application-with-event-messaging"></a>Incorporación de información en tiempo real en la aplicación con la mensajería de eventos

El segundo uso más popular de Stream Analytics es generar alertas en tiempo real. En este patrón de solución, la lógica de negocios en Stream Analytics se puede usar para detectar [patrones espaciales y temporales](stream-analytics-geospatial-functions.md) o [anomalías](stream-analytics-machine-learning-anomaly-detection.md) y luego generar señales de alerta. Sin embargo, a diferencia de la solución de panel donde Stream Analytics usa Power BI como un punto de conexión preferido, se puede usar un número de receptores de datos intermedios. Estos receptores incluyen Event Hubs, Service Bus y Azure Functions. Como el generador de la aplicación, debe decidir qué receptor de datos funciona mejor para su escenario.

Para generar alertas en el flujo de trabajo de negocio existente, se debe implementar una lógica de consumidor de eventos descendente. Dado que puede implementar lógica personalizada en Azure Functions, Azure Functions es la forma más rápida para realizar esta integración. Encontrará un tutorial para usar Azure Function como la salida de un trabajo de Stream Analytics en [Ejecución de Azure Functions desde trabajos de Azure Stream Analytics](stream-analytics-with-azure-functions.md). Azure Functions también admite varios tipos de notificaciones, incluidos texto y correo electrónico. Para este tipo de integración, también se puede usar Logic App, con Event Hubs entre Stream Analytics y Logic App.

![Aplicación de mensajería de eventos ASA](media/stream-analytics-solution-patterns/event-messaging-app.png)

Por otro lado, Event Hubs ofrece el punto de integración más flexible. Muchos otros servicios, como Azure Data Explorer y Time Series Insights, pueden consumir eventos desde Event Hubs. Los servicios se pueden conectar directamente a los receptores de Event Hubs desde Azure Stream Analytics para completar la solución. Event Hubs también es el agente de mensajería de más alto rendimiento disponible en Azure para estos escenarios de integración.

## <a name="dynamic-applications-and-websites"></a>Sitios web y aplicaciones dinámicas

Puede crear visualizaciones personalizadas en tiempo real, por ejemplo, panel o visualización de asignaciones, mediante Azure Stream Analytics y Azure SignalR Service. Mediante SignalR, los clientes web se pueden actualizar y mostrar contenido dinámico en tiempo real.

![Aplicación dinámica ASA](media/stream-analytics-solution-patterns/dynamic-app.png)

## <a name="incorporate-real-time-insights-into-your-application-through-data-stores"></a>Incorporación de información en tiempo real en la aplicación a través de almacenes de datos

La mayoría de los servicios web y aplicaciones web de hoy en día usan un modelo de solicitud y respuesta para procesar la capa de presentación. El patrón de solicitud y respuesta es fácil de crear y se puede escalar fácilmente con un tiempo de respuesta bajo mediante almacenes front-end sin estado y escalables, como Cosmos DB.

Un volumen de datos elevado suele crear cuellos de botella de rendimiento en un sistema basado en CRUD. El [patrón de solución aprovisionamiento de eventos](/azure/architecture/patterns/event-sourcing) se usa para resolver los cuellos de botella de rendimiento. Los patrones y la información temporales también son difíciles e eficaces para extraer de un almacén de datos tradicional. Las aplicaciones modernas impulsadas por datos de alto volumen suelen adoptar una arquitectura basada en el flujo de datos. Azure Stream Analytics como el motor de proceso para los datos en movimiento es una pieza clave de esa arquitectura.

![Aplicación de aprovisionamiento de eventos ASA](media/stream-analytics-solution-patterns/event-sourcing-app.png)

En este patrón de solución, los eventos se procesan y agregan en almacenes de datos mediante Azure Stream Analytics. La capa de aplicación interactúa con los almacenes de datos mediante el modelo tradicional de solicitud y respuesta. Debido a la capacidad de Stream Analytics de procesar un gran número de eventos en tiempo real, la aplicación es altamente escalable sin tener que ampliar la capa de almacén de datos de forma masiva. La capa de almacén de datos es esencialmente una vista materializada en el sistema. En [Salida de Azure Stream Analytics a Azure Cosmos DB](stream-analytics-documentdb-output.md) se describe cómo se usa Cosmos DB como una salida de Stream Analytics.

En las aplicaciones reales donde la lógica de procesamiento es compleja y existe la necesidad de actualizar determinadas partes de la lógica de forma independiente, se pueden crear varios trabajos de Stream Analytics junto con Event Hubs como agente intermediario de eventos.

![Aplicación de aprovisionamiento de eventos complejos ASA](media/stream-analytics-solution-patterns/event-sourcing-app-complex.png)

Este patrón mejora la resistencia y facilidad de uso del sistema. Sin embargo, aunque Stream Analytics garantiza un procesamiento de exactamente una vez, hay una pequeña posibilidad de que eventos duplicados se inserten en las instancias de Event Hubs intermediarias. Es importante para el trabajo de Stream Analytics descendente depurar los eventos mediante claves de lógica en una ventana de retrospectiva. Para más información sobre la entrega de eventos, consulte la referencia [Garantías de entrega de eventos](/stream-analytics-query/event-delivery-guarantees-azure-stream-analytics).

## <a name="use-reference-data-for-application-customization"></a>Uso de datos de referencia para la personalización de la aplicación

La característica de datos de referencia de Azure Stream Analytics está diseñada específicamente para la personalización del usuario final, como las alertas de umbral, el procesamiento de reglas y las [geovallas](geospatial-scenarios.md). La capa de aplicación puede aceptar los cambios de parámetro y almacenarlos en SQL Database. El trabajo de Stream Analytics periódicamente consulta la base de datos en busca de cambios y hace que los parámetros de personalización sean accesibles a través de una combinación de datos de referencia. Para más información sobre cómo usar los datos de referencia para la personalización de la aplicación, consulte [Datos de referencia SQL](sql-reference-data.md) y [Combinación de datos de referencia](/stream-analytics-query/reference-data-join-azure-stream-analytics).

Este patrón también se puede usar para implementar un motor de reglas donde se definen los umbrales de las reglas de los datos de referencia. Para más información sobre las reglas, consulte [Procesar reglas configurables basadas en umbrales en Azure Stream Analytics](stream-analytics-threshold-based-rules.md).

![Aplicación de datos de referencia ASA](media/stream-analytics-solution-patterns/reference-data-app.png)

## <a name="add-machine-learning-to-your-real-time-insights"></a>Adición del aprendizaje automático a la información en tiempo real

El [modelo de detección de anomalías](stream-analytics-machine-learning-anomaly-detection.md) integrado de Azure Stream Analytics es una manera cómoda de introducir el aprendizaje automático a la aplicación en tiempo real. Para una variedad más amplia de necesidades de aprendizaje automático, consulte [Azure Stream Analytics se integra con el servicio de puntuación de Azure Machine Learning](stream-analytics-machine-learning-integration-tutorial.md).

Los usuarios avanzados que desean incorporar cursos en línea y la puntuación en la misma canalización de Stream Analytics pueden consultar este ejemplo de cómo hacerlo con [regresión lineal](stream-analytics-high-frequency-trading.md).

![Aplicación de aprendizaje automático ASA](media/stream-analytics-solution-patterns/machine-learning-app.png)

## <a name="real-time-data-warehousing"></a>Almacenamiento de datos en tiempo real

Otro patrón común es el almacenamiento de datos en tiempo real, también conocido como almacén de datos de streaming. Además de los eventos que llegan a Event Hubs e IoT Hub desde la aplicación, [Azure Stream Analytics que se ejecutan en IoT Edge](stream-analytics-edge.md) puede usarse para satisfacer las necesidades de limpieza de datos, reducción de datos y almacenamiento y reenvío de datos. Stream Analytics que se ejecutan en IoT Edge puede administrar con agilidad los problemas de limitación de ancho de banda y conectividad en el sistema. Stream Analytics puede admitir velocidades de rendimiento de hasta 200 MB/s al escribir en Azure Synapse Analytics.

![Almacenamiento de datos ASA](media/stream-analytics-solution-patterns/data-warehousing.png)


## <a name="archiving-real-time-data-for-analytics"></a>Archivado de datos en tiempo real para el análisis

La mayoría actividades ciencia y análisis de datos aún se realiza sin conexión. Con Azure Stream Analytics, se pueden archivar datos a través de la salida de Azure Data Lake Store Gen2 y formatos de salida Parquet. Esta funcionalidad quita la fricción para alimentar datos directamente en Azure Data Lake Analytics, Azure Databricks y Azure HDInsight. En esta solución, Azure Stream Analytics se usa como un motor ETL casi en tiempo real. Puede explorar los datos archivados en Data Lake mediante distintos motores de proceso.

> [!div class="mx-imgBorder"]
> ![Análisis sin conexión de ASA](media/stream-analytics-solution-patterns/offline-analytics.png)

## <a name="use-reference-data-for-enrichment"></a>Uso de datos de referencia para el enriquecimiento

Con frecuencia, el enriquecimiento de datos es un requisito para los motores de ETL. Azure Stream Analytics admite el enriquecimiento de datos con [datos de referencia](stream-analytics-use-reference-data.md) desde SQL Database y Azure Blob Storage. El enriquecimiento de datos se puede realizar para los datos que llegan a Azure Data Lake y SQL Data Warehouse.


![Análisis sin conexión con enriquecimiento de datos ASA](media/stream-analytics-solution-patterns/offline-analytics-enriched.png)

## <a name="operationalize-insights-from-archived-data"></a>Uso de la información a partir de datos archivados

Si combina el patrón de análisis sin conexión con el patrón de la aplicación casi en tiempo real, puede crear un bucle de comentarios. El bucle de comentarios permite que la aplicación se ajuste automáticamente para cambiar los patrones en los datos. Este bucle de comentarios puede ser tan sencillo como cambiar el valor de umbral para las alertas, o tan complejo como el reciclaje de modelos de Machine Learning. Se puede aplicar la misma arquitectura de solución a ambos trabajos ASA que se ejecutan en la nube y en IoT Edge.

![Uso de información ASA](media/stream-analytics-solution-patterns/insights-operationalization.png)

## <a name="how-to-monitor-asa-jobs"></a>Cómo supervisar trabajos ASA

Un trabajo de Azure Stream Analytics se puede ejecutar las 24 horas, los 7 días de la semana, para procesar eventos entrantes continuamente en tiempo real. La garantía del tiempo de actividad es fundamental para el estado de la aplicación global. Si bien Stream Analytics es el único servicio de análisis de streaming del sector que ofrece una [garantía de disponibilidad del 99,9%](https://azure.microsoft.com/support/legal/sla/stream-analytics/v1_0/), aún puede producirse cierto nivel de tiempo de inactividad. A lo largo de los años, Stream Analytics ha introducido métricas, registros y estados de trabajo para reflejar el estado de los trabajos. Todos ellos se exponen a través del servicio Azure Monitor y se pueden exportar a OMS. Para más información, consulte [Descripción de la supervisión del trabajo de Stream Analytics y cómo supervisar consultas](stream-analytics-monitoring.md).

![Supervisión de ASA](media/stream-analytics-solution-patterns/monitoring.png)

Hay dos elementos clave que se deben supervisar:

- [Estado de error del trabajo](job-states.md)

    En primer lugar y ante todo, debe asegurarse de que el trabajo esté en ejecución. Si trabajo no se encuentra en el estado de ejecución, no se generan nuevas métricas ni registros. Los trabajos pueden cambiar a un estado de error por diversos motivos, como por ejemplo, un uso elevado de la unidad de almacenamiento (es decir, se está quedando sin recursos).

- [Métricas de retraso de marca de agua](https://azure.microsoft.com/blog/new-metric-in-azure-stream-analytics-tracks-latency-of-your-streaming-pipeline/)

    Esta métrica refleja cuán lejos se encuentra el procesamiento de canalización del tiempo de reloj (segundos). Parte del retraso se atribuye a la lógica de procesamiento inherente. Como resultado, la supervisión de la tendencia creciente es mucho más importante que la supervisión del valor absoluto. La aplicación debería abordar el estado de retraso constante en el diseño, no mediante alertas o supervisión.

En caso de error, los registros de actividad y los [registros de diagnóstico](stream-analytics-job-diagnostic-logs.md) son los mejores lugares para empezar a buscar errores.

## <a name="build-resilient-and-mission-critical-applications"></a>Compilación de aplicaciones resistentes y de misión crítica

Independientemente de la garantía del SLA de Azure Stream Analytics y el grado de cuidado con el que se ejecute la aplicación de un extremo a otro, se producen interrupciones. Si la aplicación es de misión crítica, debe estar preparado para interrupciones y poder recuperar sin contratiempos.

Para las aplicaciones de alerta, lo más importante es detectar la alerta siguiente. Al recupera, puede elegir reiniciar el trabajo desde la hora actual, omitiendo las alertas anteriores. La semántica de hora de inicio del trabajo es por la primera hora de salida, no la primera hora de entrada. La entrada retrocede una cantidad de tiempo apropiada para garantizar que la primera salida a la hora especificada está completa y es correcta. No obtendrá agregados parciales y desencadenará alertas inesperadamente como resultado.

También puede optar por iniciar la salida desde una cierta cantidad de tiempo en el pasado. Las directivas de retención de Event Hubs y IoT Hub tienen una cantidad razonable de datos para permitir el procesamiento del pasado. El inconveniente es la rapidez con la que puede ponerse al día y comenzar a generar nuevas alertas oportunas. Los datos pierden su valor rápidamente con el tiempo, por lo que es importante ponerse al día rápidamente. Para ello, hay dos métodos:

- Aprovisionar más recursos (SU) al ponerse al día.
- Reiniciar desde la hora actual.

Reiniciar desde la hora actual es fácil de hacer, con el inconveniente de que deja un vacío durante el procesamiento. Este método podría resultar adecuado para los escenarios de alerta, pero puede ser problemático para los escenarios de panel y no debería usarse en absoluto para los escenarios de archivado y almacenamiento de datos.

El aprovisionamiento de más recursos puede acelerar el proceso, pero el efecto de tener un aumento importante en la velocidad de procesamiento es complejo.

- Realice pruebas del trabajo para comprobar que se pueda escalar a un mayor número de unidades de almacenamiento. No todas las consultas son escalables. Deberá asegurarse de que la consulta sea [paralelizada](stream-analytics-parallelization.md).

- Asegúrese de que haya particiones suficientes en la instancia ascendente de Event Hubs o IoT Hub y que pueda agregar más unidades de procesamiento (TU) para escalar el rendimiento de entrada. Recuerde que la velocidad de salida máxima de cada instancia de unidad de procesamiento de Event Hubs es de 2 MB/s.

- Asegúrese de haber aprovisionado recursos suficientes en los receptores de salida (por ej., SQL Database, Cosmos DB) para que no limiten el auge de salida, que a veces puede provocar un bloqueo del sistema.

Lo más importante es prever el cambio de velocidad de procesamiento, probar estos escenarios antes de entrar en producción y estar preparado para escalar el procesamiento correctamente durante el tiempo de recuperación de errores.

En el escenario extremo en el que se retrasan los eventos de entrada, [es posible que se eliminen todos los eventos retrasados](stream-analytics-time-handling.md) si ha aplicado al trabajo una ventana de llegada retrasada. La eliminación de los eventos puede parecer un comportamiento misterioso al principio. Sin embargo, si se tiene en cuenta que Stream Analytics es un motor de procesamiento en tiempo real, espera que los eventos entrantes estén cerca de la hora de reloj. Tiene que eliminar los eventos que infringen estas restricciones.

### <a name="lambda-architectures-or-backfill-process"></a>Proceso de reposición o arquitecturas lambda

Afortunadamente, se puede usar el patrón de archivado de datos anterior para procesar estos eventos retrasados correctamente. La idea es que el trabajo de archivado procese los eventos de entrada en la hora de llegada y archive los eventos en el cubo temporal oportuno en Azure Blob o Azure Data Lake Store con la hora del evento. No importa lo tarde que llegue un evento, nunca se eliminará. Siempre llegará al cubo temporal oportuno. Durante la recuperación, es posible volver a procesar los eventos archivados y reponer los resultados en el almacén deseado. Esto es similar al modo en que se implementan los patrones lambda.

![Reposición ASA](media/stream-analytics-solution-patterns/back-fill.png)

El proceso de reposición se debe realizar con un sistema de procesamiento por lotes sin conexión, que probablemente tiene un modelo de programación diferente que el de Azure Stream Analytics. Esto significa que tendrá que volver a implementar la lógica de procesamiento completo.

Para la reposición, aún es importante aprovisionar al menos temporalmente más recursos a los receptores de salida para aceptar un mayor rendimiento que durante el procesamiento de estado estable.

|Escenarios  |Reinicio solo desde ahora  |Reinicio desde la última hora de parada |Reinicio desde ahora + reposición con eventos archivados|
|---------|---------|---------|---------|
|**Creación de paneles**   |Crea una brecha    |Aceptar para una interrupción corta    |Usar para una interrupción prolongada |
|**Alertas**   |Aceptable |Aceptar para una interrupción corta    |No es necesario |
|**Aplicación de aprovisionamiento de eventos** |Aceptable |Aceptar para una interrupción corta    |Usar para una interrupción prolongada |
|**Almacenamiento de datos**   |Pérdida de datos  |Aceptable |No es necesario |
|**Análisis sin conexión**  |Pérdida de datos  |Aceptable |No es necesario|

## <a name="putting-it-all-together"></a>Resumen

No es difícil imaginar que todos los patrones de soluciones mencionados anteriormente se pueden combinar en un sistema complejo de un extremo a otro. El sistema combinado puede incluir paneles, alertas, una aplicación de abastecimiento de eventos, almacenamiento de datos y funcionalidades de análisis sin conexión.

La clave consiste en diseñar el sistema en patrones que admite composición, de modo que cada subsistema se pueda compilar, probar, actualizar y recuperar de manera independiente.

## <a name="next-steps"></a>Pasos siguientes

Ahora ha visto una variedad de patrones de solución que usan Azure Stream Analytics. A continuación, puede profundizar y crear su primer trabajo de Stream Analytics:

* [Creación de un trabajo de Stream Analytics mediante Azure Portal](stream-analytics-quick-create-portal.md).
* [Creación de un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md).
* [Creación de un trabajo de Stream Analytics mediante Visual Studio](stream-analytics-quick-create-vs.md).
