---
title: Características en vista previa (GB) de Azure Stream Analytics
description: En este artículo se enumeran las características de Azure Stream Analytics que están actualmente en versión preliminar.
services: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.reviewer: jasonh
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 05/07/2019
ms.openlocfilehash: f3838bf6b9f7daa24c0cdb5b0c5a08d41d164530
ms.sourcegitcommit: 179918af242d52664d3274370c6fdaec6c783eb6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65561135"
---
# <a name="azure-stream-analytics-preview-features"></a>Características en vista previa (GB) de Azure Stream Analytics

En este artículo se resumen todas las características actualmente en versión preliminar de Azure Stream Analytics. No se recomienda el uso de características en vista previa en un entorno de producción.

## <a name="public-previews"></a>Versiones preliminares públicas

Las características siguientes se encuentran en la versión preliminar pública. Puede aprovechar las ventajas de estas características hoy mismo, pero no las use en su entorno de producción.

### <a name="visual-studio-code-for-azure-stream-analytics-released-may-2019"></a>Visual Studio Code para análisis de Azure Stream (publicado en mayo de 2019)

Los trabajos de Azure Stream Analytics se pueden crear en Visual Studio Code. Consulte nuestra [tutorial de introducción de VS Code a](https://docs.microsoft.com/azure/stream-analytics/quick-create-vs-code).

### <a name="anomaly-detection"></a>Detección de anomalías

Azure Stream Analytics introduce nuevos modelos de aprendizaje automático con compatibilidad para a detección de *picos* y *caídas*, además de detección de tendencias bidireccionales, positivas lentas y negativas lentas. Para obtener más información, visite [detección de anomalías en Azure Stream Analytics](stream-analytics-machine-learning-anomaly-detection.md).

### <a name="sql-database-reference-data"></a>Datos de referencia de SQL Database

Azure Stream Analytics admite Azure SQL Database como origen de la entrada de datos de referencia. Puede usar SQL Database como datos de referencia para su trabajo de Stream Analytics en Azure Portal y en Visual Studio con herramientas de Stream Analytics. Para más información, visite [Use reference data from a SQL Database for an Azure Stream Analytics job](sql-reference-data.md) (Uso de datos de referencia de una instancia de SQL Database para un trabajo de Azure Stream Analytics).

### <a name="integration-with-azure-machine-learning"></a>Integración con Azure Machine Learning

Puede escalar los trabajos de Stream Analytics con las funciones de Machine Learning (ML). Para obtener más información acerca de cómo puede usar las funciones de Machine Learning en un trabajo de Stream Analytics, visite [Escalado del trabajo de Análisis de transmisiones con funciones de Azure Machine Learning](stream-analytics-scale-with-machine-learning-functions.md). Eche un vistazo a un escenario realista con [Análisis de opiniones mediante Azure Stream Analytics y Azure Machine Learning](stream-analytics-machine-learning-integration-tutorial.md).

### <a name="javascript-user-defined-aggregate"></a>Agregado definido por el usuario en JavaScript

Azure Stream Analytics admite agregados definidos por el usuario (UDA) escritos en JavaScript, que le permiten implementar lógica empresarial con estado compleja. Obtenga información sobre cómo usar los UDA en la documentación de [Agregados definidos por el usuario en JavaScript para Azure Stream Analytics](stream-analytics-javascript-user-defined-aggregates.md). 

### <a name="live-data-testing-in-visual-studio"></a>Prueba de datos activos en Visual Studio

Las herramientas de Visual Studio para Azure Stream Analytics mejoran la función de pruebas locales que le permite realizar pruebas en las consultas en comparación con los flujos de datos de eventos activos desde fuentes en la nube como un centro de eventos o un centro de IoT. Aprenda cómo realizar una [Prueba local de datos activos mediante herramientas de Azure Stream Analytics para Visual Studio](stream-analytics-live-data-local-testing.md).

### <a name="net-user-defined-functions-on-iot-edge"></a>Funciones definidas por el usuario de .NET en IoT Edge

Con las funciones definidas por el usuario de . NET Standard, puede ejecutar código de .NET Standard como parte de la canalización de transmisiones de datos. Puede crear clases de C# simples o importar bibliotecas y proyectos completos. Las funciones de creación y depuración completas se admiten en Visual Studio. Para obtener más información, visite [Desarrollar funciones definidas por el usuario de .NET Standard para trabajos perimetrales de Azure Stream Analytics](stream-analytics-edge-csharp-udf-methods.md).

## <a name="other-previews"></a>Otras versiones preliminares

Las siguientes características también están disponibles en versión preliminar.

### <a name="c-custom-deserializer-for-azure-stream-analytics-on-iot-edge-and-cloud-announced-may-2019"></a>C#deserializador personalizado para Azure Stream Analytics en IoT Edge y en la nube (Announced de mayo de 2019)

Los desarrolladores pueden implementar deserializadores personalizados en C# para deserializar los eventos recibidos por Azure Stream Analytics. Parquet, Protobuf, XML y todos los formatos binarios son algunos ejemplos de formatos que se pueden deserializar. Regístrese para esta versión preliminar [aquí](https://aka.ms/asapreview1).

### <a name="parquet-output-announced-may-2019"></a>Salida de parquet (anunciada en mayo de 2019)
Parquet es un formato de columnas habilitar eficaz procesamiento de macrodatos. Al generar datos en formato Parquet en un lago de datos, puede aprovechar las ventajas de Azure Stream Analytics para ETL de streaming de gran escala de energía y ejecutar procesamiento por lotes, entrenar algoritmos de aprendizaje automático o ejecutar consultas interactivas en los datos históricos. Regístrese para esta versión preliminar [aquí](https://aka.ms/asapreview1).

### <a name="one-click-integration-with-event-hubs-announced-may-2019"></a>Integración con Event Hubs (Announced de mayo de 2019) con un solo clic 
Con esta integración, ahora podrá visualizar los datos entrantes y empezar a escribir una consulta de Stream Analytics con un solo clic desde el portal del centro de eventos. Una vez que la consulta está lista, podrá la incluya en pocos clics y comenzar a obtener perspectivas en tiempo real. Esto reducirá significativamente el tiempo y costo para desarrollar soluciones de análisis en tiempo real. Regístrese para esta versión preliminar [aquí](https://aka.ms/asapreview1).

### <a name="support-for-azure-stack-announced-may-2019"></a>Compatibilidad con Azure Stack (anunciada en mayo de 2019)
Esta característica está habilitada en el tiempo de ejecución de Azure IoT Edge aprovecha las características personalizadas de Azure Stack, como la compatibilidad nativa con entradas locales y da como resultado que se ejecuta en Azure Stack (por ejemplo, Event Hubs, IoT Hub, Blob Storage). Esta nueva integración le permite crear arquitecturas híbridas que se pueden analizar los datos cerca de donde se genera, disminuye la latencia y maximizar insights.
Regístrese para esta versión preliminar [aquí](https://aka.ms/asapreview1).

