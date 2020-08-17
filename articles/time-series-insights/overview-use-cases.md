---
title: 'Casos de uso de Gen2: Azure Time Series Insights Gen2 | Microsoft Docs'
description: Obtenga más información sobre casos de uso de Azure Time Series Insights Gen2.
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.workload: big-data
ms.service: time-series-insights
services: time-series-insights
ms.topic: conceptual
ms.date: 07/07/2020
ms.custom: seodec18
ms.openlocfilehash: d2d3c94985f099c7a0ab92efab564415f5cd81bb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095414"
---
# <a name="azure-time-series-insights-gen2-use-cases"></a>Casos de uso de Azure Time Series Insights Gen2

En este artículo se resumen varios casos de uso comunes de Azure Time Series Insights Gen2. Las recomendaciones de este artículo le servirán como punto de partida para desarrollar aplicaciones y soluciones con Azure Time Series Insights Gen2.

Concretamente, en este artículo se responde a las siguientes preguntas:

* ¿Cuáles son los casos de uso comunes de Azure Time Series Insights Gen2?
* ¿Cuáles son las ventajas de usar Azure Time Series Insights Gen2 para la [detección de anomalías visuales y la exploración de datos](#data-exploration-and-visual-anomaly-detection)?
* ¿Cuáles son las ventajas de usar Azure Time Series Insights Gen2 para realizar [análisis operativos y garantizar la eficacia del proceso](#operational-analysis-and-driving-process-efficiency)?
* ¿Cuáles son las ventajas de usar Azure Time Series Insights Gen2 para realizar [análisis avanzados](#advanced-analytics)?

En las secciones siguientes se incluye información general de estos escenarios de uso.

## <a name="introduction"></a>Introducción

Azure Time Series Insights Gen2 es una oferta de plataforma como servicio de un extremo a otro. Se usa para recopilar, procesar, almacenar, analizar y consultar datos a escala de IoT optimizados para series temporales y muy contextualizados. Es ideal para la exploración de datos ad hoc y para el análisis operativo. Igualmente, Azure Time Series Insights Gen2 es una oferta de servicio de extensibilidad única y personalizada que satisface las amplias necesidades de las implementaciones de IoT industriales.

## <a name="data-exploration-and-visual-anomaly-detection"></a>Exploración de datos y detección anomalías visuales

Explore y analice al instante miles de millones de eventos para detectar anomalías y tendencias ocultas en sus datos. Azure Time Series Insights Gen2 ofrece un rendimiento casi en tiempo real para las cargas de trabajo de análisis de IoT y DevOps.

[![Explorador de datos](media/v2-update-use-cases/data-explorer.png)](media/v2-update-use-cases/data-explorer.png#lightbox)

La mayoría de los clientes aceptan que la cantidad de tiempo mínima necesaria para obtener información es una de las principales características de Azure Time Series Insights Gen2:

* Azure Time Series Insights Gen2 no requiere la preparación inicial de los datos.
* Funciona rápido para conectarse en minutos a miles de millones de eventos en instancias de Azure IoT Hub o de Azure Event Hubs.
* Una vez esté conectado, puede visualizar y analizar al instante miles de millones de eventos para detectar anomalías y tendencias ocultas en sus datos.

Azure Time Series Insights Gen2 es intuitivo y fácil de usar. Además, puede interactuar con sus datos sin tener que escribir una sola línea de código. Tampoco se le exige que aprenda ningún lenguaje nuevo, aunque Azure Time Series Insights Gen2 proporciona un lenguaje de consulta basado en texto granular para usuarios avanzados familiarizados con SQL. En cuanto a los principiantes, también proporciona la opción de explorar seleccionando y haciendo clic en el contenido.

Los clientes pueden aprovechar esta velocidad para poder diagnosticar rápidamente problemas relacionados con los recursos. Asimismo, pueden realizar análisis de operaciones de DevOps para llegar a la raíz de un error en una solución de IoT. También pueden identificar las áreas que se van a marcar para una investigación más exhaustiva como parte de sus iniciativas de ciencia de datos.

Existen tres formas principales de interactuar con los datos almacenados en Azure Time Series Insights Gen2:

* La primera y la forma más fácil de comenzar es con el Explorador de Azure Time Series Insights Gen2. Puede usarlo para visualizar rápidamente todos sus datos de IoT en un solo lugar. Además, le proporciona herramientas como el mapa térmico, que le ayudarán a detectar anomalías en sus datos. También proporciona una vista en perspectiva. Puede usar esta vista para comparar hasta cuatro vistas de uno o más entornos de Azure Time Series Insights Gen2 en un único panel. Este panel le ofrece una vista de sus datos de serie temporal en todas sus ubicaciones. Obtenga más información sobre el [Explorador de Azure Time Series Insights Gen2](./time-series-insights-update-explorer.md). Para planificar su entorno, lea la [planificación de Azure Time Series Insights Gen2](./time-series-insights-update-plan.md).

* La segunda forma de comenzar es usar el SDK de JavaScript para insertar rápidamente eficaces tablas y gráficos en su aplicación web. Con solo unas pocas líneas de código, puede crear consultas realmente eficaces. Úselas para rellenar gráficos de líneas, gráficos circulares, gráficos de barras, mapas térmicos, cuadrículas de datos y muchos más. Si usa el SDK, tiene todos estos elementos listos para usar. El SDK también sintetiza las API de consulta de Azure Time Series Insights Gen2. Puede usarlas para crear predicados similares a SQL para consultar los datos que quiera mostrar en un panel. En cuanto a las soluciones de la capa de presentación híbridas, Azure Time Series Insights Gen2 le ofrece direcciones URL con parámetros. Estas le proporcionan puntos de conexión sencillos con el explorador de Azure Time Series Insights Gen2 para que pueda profundizar en los datos.

  * Para más información sobre el SDK de JavaScript, lea la [biblioteca de cliente de JS](https://github.com/microsoft/tsiclient/blob/master/docs/API.md) y la documentación del [cliente de muestras](https://github.com/Microsoft/tsiclient).

  * Para obtener más información sobre el uso compartido de direcciones URL y la nueva IU, consulte [Visualización de datos en el Explorador de Azure Time Series Insights Gen2](time-series-insights-update-explorer.md).

* La tercera forma de comenzar es usar las eficaces API para consultar los datos almacenados en Azure Time Series Insights Gen2. Azure Time Series Insights Gen2 tiene operadores temporales, como `from`, `to`, `first` y `last`, y tiene agregaciones y transformaciones, como `average`, `sum`, `min`, `max`, `time-weighted average`, `time-weighted sum`, etc. También permite filtros, operadores aritméticos y booleanos, funciones escalares, etc. Todos estos operadores permiten que las aplicaciones posteriores encuentren rápidamente tendencias y patrones interesantes en sus datos. Puede usarlos para rellenar las visualizaciones locales y así poder detectar anomalías.

## <a name="operational-analysis-and-driving-process-efficiency"></a>Análisis operativo y procesos eficientes

Use Azure Time Series Insights Gen2 para supervisar el mantenimiento, el uso y el rendimiento del equipo a escala y para medir la eficacia operativa. Igualmente, Azure Time Series Insights Gen2 facilita la administración de cargas de trabajo de IoT diversas e impredecibles, sin sacrificar el rendimiento de la ingesta o de las consultas.

[![Información general](media/v2-update-use-cases/overview.png)](media/v2-update-use-cases/overview.png#lightbox)

La transmisión y el procesamiento continuo de datos provenientes de procesos operativos pueden transformar con éxito cualquier negocio si se combinan con la tecnología o la solución adecuada. A menudo, estas soluciones son una combinación de varios sistemas. Permiten la exploración y el análisis de datos que cambian constantemente, especialmente en el ámbito de IoT, y comparten un patrón común.

Estos patrones a menudo comienzan con plataformas habilitadas para IoT que ingieren miles de millones de eventos de dispositivos y sensores que abarcan varias configuraciones regionales. Asimismo, estos sistemas procesan y analizan los datos de streaming para obtener detalles y acciones en tiempo real. Los datos normalmente se archivan mediante un almacenamiento en reposo e intermedio para realizar un análisis casi en tiempo real y por lotes.

Los datos que se recopilan pasan por una serie de procesos de limpieza y contextualización para agregarlos a escenarios de análisis y consultas posteriores. Azure ofrece sofisticados servicios que se pueden aplicar a los escenarios de IoT, como el mantenimiento y la fabricación de recursos. Estos servicios incluyen Azure Time Series Insights Gen2, IoT Hub, Event Hubs, Azure Stream Analytics, Azure Functions, Azure Logic Apps, Azure Databricks, Azure Machine Learning y Power BI.

La arquitectura de la solución se puede obtener de la siguiente manera:

* Ingerir datos a través de IoT Hub o Event Hubs para obtener seguridad, rendimiento y latencia de vanguardia.
* Realizar el procesamiento de datos y los cálculos. Canalice los datos ingeridos a través de servicios como Stream Analytics, Logic Apps y Azure Functions. El servicio que use dependerá de las necesidades específicas de procesamiento de datos.
* Las señales calculadas de la canalización de procesamiento se insertan en Azure Time Series Insights Gen2 para su almacenaje y análisis.

Azure Time Series Insights Gen2 le ofrece la oportunidad de explorar los datos casi en tiempo real, así como detalles basados en recursos sobre datos históricos. Dependiendo de las necesidades de su negocio, los trabajos de MapReduce y Hive pueden ejecutarse en datos almacenados en Azure Time Series Insights Gen2 si se conecta Azure Time Series Insights Gen2 a Azure HDInsight. Los datos almacenados en Azure Time Series Insights Gen2 están disponibles para Power BI y otras aplicaciones de cliente a través de las API de consulta de superficie públicas de Azure Time Series Insights Gen2. Estos datos se pueden usar en escenarios de inteligencia profunda operativa y empresarial.

## <a name="advanced-analytics"></a>Análisis avanzado

Integre sus soluciones con servicios de análisis avanzado, como Azure Machine Learning y Azure Databricks. Azure Time Series Insights Gen2 incorpora datos sin procesar de millones de dispositivos. Además, agrega datos contextuales que un conjunto de servicios de análisis de Azure pueden consumir de forma sencilla.

[![Análisis](media/v2-update-use-cases/advanced-analytics.png)](media/v2-update-use-cases/advanced-analytics.png#lightbox)

El análisis avanzado y el aprendizaje automático consumen y procesan grandes volúmenes de datos. Estos datos se usan para tomar decisiones basadas en datos y realizar análisis predictivos. En los casos de uso de IoT, los algoritmos de análisis avanzados aprenden de los datos recopilados de millones de dispositivos. Estos dispositivos transmiten datos varias veces cada segundo. Los datos recopilados de los dispositivos de IoT están sin procesar. Carecen de información contextual, como la ubicación del dispositivo y la unidad de lectura del sensor. Como resultado, los datos sin procesar son difíciles de consumir directamente para realizar análisis avanzados.

Azure Time Series Insights Gen2 cierra esta brecha entre los datos de IoT y el análisis avanzado de dos maneras simples y rentables:

* Primero, Azure Time Series Insights Gen2 recopila los datos de telemetría sin procesar de millones de dispositivos mediante IoT Hub. A continuación, enriquece los datos con información contextual y los transforma en un formato parquet. Este formato se puede integrar fácilmente con otros servicios analíticos avanzados, como Machine Learning, Azure Databricks y aplicaciones de terceros.

    Azure Time Series Insights Gen2 puede servir como fuente de datos para todos los datos de toda la organización. Crea un repositorio central para que las cargas de trabajo analíticas descendentes se consuman. Como Azure Time Series Insights Gen2 es un servicio de almacenamiento casi en tiempo real, los modelos analíticos avanzados pueden aprender continuamente de los datos de telemetría entrantes de IoT. Gracias a ello, los modelos pueden hacer predicciones más precisas.

* En segundo lugar, Azure Time Series Insights Gen2 puede beneficiarse de la salida del aprendizaje automático y los modelos de predicción para visualizar y almacenar sus resultados. Este procedimiento permite que las organizaciones tengan la oportunidad de optimizar y ajustar sus modelos. Azure Time Series Insights Gen2 simplifica la visualización de los datos de telemetría de streaming en el mismo plano que las salidas del modelo entrenado. De esta manera, ayuda a los equipos de ciencia de datos a detectar anomalías e identificar patrones.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre el [Explorador de Azure Time Series Insights Gen2](./time-series-insights-update-explorer.md).
* Para planificar su entorno, lea los [procedimientos recomendados de Azure Time Series Insights Gen2](./time-series-insights-update-plan.md).
* Lea la documentación del [cliente de muestras](https://github.com/Microsoft/tsiclient).
