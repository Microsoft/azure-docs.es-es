---
title: Elija un análisis en tiempo real y la tecnología de procesamiento en Azure de streaming
description: Obtenga información sobre cómo elegir el análisis en tiempo real directamente y la tecnología de procesamiento de streaming para compilar la aplicación en Azure.
author: zhongc
ms.author: zhongc
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/15/2019
ms.openlocfilehash: 7ef8715f23d21a23dc3f37b192314b922847d3e1
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65805912"
---
# <a name="choose-a-real-time-analytics-and-streaming-processing-technology-on-azure"></a>Elija un análisis en tiempo real y la tecnología de procesamiento en Azure de streaming

Hay varios servicios para análisis en tiempo real y el procesamiento de streaming en Azure. En este artículo se proporciona la información que necesita decidir qué tecnología es la mejor opción para la aplicación.

## <a name="when-to-use-azure-stream-analytics"></a>Cuándo usar Azure Stream Analytics

Azure Stream Analytics es el servicio recomendado para stream analytics en Azure. Está destinado a una amplia gama de escenarios que incluyen, pero no se limitan a:

* Paneles de visualización de datos
* En tiempo real [alertas](stream-analytics-set-up-alerts.md) de patrones espaciales y temporales o de anomalías
* Extraer, transformar y cargar (ETL)
* [Patrón Event Sourcing](/azure/architecture/patterns/event-sourcing.md)
* [IoT Edge](stream-analytics-edge.md)

Agregar un Azure Stream Analytics trabajo a la aplicación es la manera más rápida de obtener análisis de transmisiones de y que se ejecuta en Azure mediante el lenguaje SQL que ya conoce. Azure Stream Analytics es un servicio de trabajo, por lo que no tiene que perder clústeres de administración del tiempo y no tiene que preocuparse por el tiempo de inactividad con un SLA del 99,9% en el nivel de trabajo. La facturación también se realiza en el nivel de trabajo que los costos iniciales de baja (una unidad de Streaming), pero escalable (hasta 192 unidades de Streaming). Es mucho más rentable ejecutar algunos trabajos de Stream Analytics diferente ejecutar y mantener un clúster.

Azure Stream Analytics tiene una experiencia de fábrica. Inmediatamente pueden sacar provecho de las siguientes características sin ninguna configuración adicional:

* Los operadores temporales integrados, como [agregados en ventanas](stream-analytics-window-functions.md), combinaciones temporales y funciones analíticas temporales.
* Azure nativo [entrada](stream-analytics-add-inputs.md) y [salida](stream-analytics-define-outputs.md) adaptadores
* Variación lenta [hacen referencia a datos](stream-analytics-use-reference-data.md) (también conocido como un tablas de búsqueda), incluida la unión con datos de referencia geoespaciales de perímetro.
* Integrado, como las soluciones, [detección de anomalías](stream-analytics-machine-learning-anomaly-detection.md)
* Varias ventanas de tiempo en la misma consulta
* Capacidad de crear varios operadores temporales en secuencias arbitrarias.
* Debajo de 100 ms-to-end latencia de entrada que llegan a Event Hubs, a la salida de aterrizaje en Event Hubs, incluidos el retraso de red desde y a Event Hubs, con alto rendimiento sostenido

## <a name="when-to-use-other-technologies"></a>Cuándo usar otras tecnologías

### <a name="you-need-to-input-from-or-output-to-kafka"></a>Necesita para entrada o salida para Kafka

Azure Stream Analytics no tiene una entrada de Apache Kafka o adaptador de salida. Si tiene eventos que llegan a o necesita para enviar a Kafka y no tiene un requisito para ejecutar su propio clúster de Kafka, aún puede usar Stream Analytics mediante el envío de eventos a Event Hubs mediante la API de Event Hubs Kafka sin cambiar el remitente del evento. Si es necesario ejecutar su propio clúster de Kafka, puede usar Spark Structured Streaming, que es totalmente compatible con [Azure Databricks](../azure-databricks/index.yml), o de Storm en [HDInsight de Azure](../hdinsight/storm/apache-storm-tutorial-get-started-linux.md).

### <a name="you-want-to-write-udfs-udas-and-custom-deserializers-in-a-language-other-than-javascript-or-c"></a>Desea escribir deserializadores personalizados, UDA y UDF en un idioma distinto de JavaScript oC#

Azure Stream Analytics admite funciones definidas por el usuario (UDF) o agregados definidos por el usuario (UDA) en JavaScript para los trabajos en la nube y C# para trabajos de IoT Edge. C#También se admiten deserializadores definido por el usuario. Si desea implementar un UDA, una UDF o un deserializador en otros lenguajes, como Java o Python, puede usar Spark Structured Streaming. También puede ejecutar los centros de eventos **EventProcessorHost** en sus propias máquinas virtuales para realizar el procesamiento de transmisión por secuencias arbitrario.

### <a name="your-solution-is-in-a-multi-cloud-or-on-premises-environment"></a>La solución está en un entorno de varios en la nube o local

Azure Stream Analytics es la tecnología de Microsoft y solo está disponible en Azure. Si necesita la solución sea portable entre nubes o de forma local, considere la posibilidad de tecnologías de código abierto como Storm o Spark Structured Streaming.

## <a name="next-steps"></a>Pasos siguientes

* [Crear un trabajo de Stream Analytics mediante el portal de Azure](stream-analytics-quick-create-portal.md)
* [Crear un trabajo de Stream Analytics mediante Azure PowerShell](stream-analytics-quick-create-powershell.md)
* [Crear un trabajo de Stream Analytics mediante el uso de Visual Studio](stream-analytics-quick-create-vs.md)
* [Crear un trabajo de Stream Analytics mediante el uso de Visual Studio Code](quick-create-vs-code.md)