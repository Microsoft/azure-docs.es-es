---
title: Comparación de características de Azure Stream Analytics
description: En este artículo se comparan las características admitidas en los trabajos en la nube de Azure Stream Analytics y en los trabajos de IoT Edge en Azure Portal, Visual Studio y Visual Studio Code.
author: mamccrea
ms.author: mamccrea
ms.service: stream-analytics
ms.topic: conceptual
ms.date: 06/27/2019
ms.openlocfilehash: 2ba7dc3b0f2bc4f62234c480da0af1061dea1f91
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90885560"
---
# <a name="azure-stream-analytics-feature-comparison"></a>Comparación de características de Azure Stream Analytics

Con Azure Stream Analytics, puede crear soluciones de streaming en la nube y en IoT Edge mediante [Azure Portal](stream-analytics-quick-create-portal.md), [Visual Studio](stream-analytics-quick-create-vs.md) y [Visual Studio Code](quick-create-visual-studio-code.md). En las tablas de este artículo se muestran las características que se admiten con cada plataforma en ambos tipos de trabajos.

> [!NOTE]
> Las herramientas Visual Studio y Visual Studio Code no admiten trabajos de las regiones de Este de China, Norte de China, Centro de Alemania y Noreste de Alemania.

## <a name="cloud-job-features"></a>Características del trabajo en la nube


|Característica  |Portal  |Visual Studio  |Visual Studio Code  |
|---------|---------|---------|---------|
|Multiplataforma     |Mac</br>Linux</br>Windows         |Windows        |Mac</br>Linux</br>Windows          |
|Creación de scripts     |Sí         |Sí         |Sí         |
|Script Intellisense     |Resaltado de sintaxis         |Resaltado de sintaxis</br>Finalización de código</br>Marcador de errores         |Resaltado de sintaxis</br>Finalización de código</br>Marcador de errores         |
|Definir todo tipo de entradas, salidas y configuraciones de trabajos     |Sí         |Sí         |Sí         |
|Control de código fuente     |No         |Sí         |Sí         |
|Compatibilidad con CI/CD     |Parcial         |Sí         |Sí         |
|Compartir entradas y salidas entre varias consultas     |No         |Sí         |Sí         |
|Pruebas de consulta con un archivo de ejemplo     |Sí         |Sí        |Sí         |
|Pruebas locales de datos en vivo     |No         |Sí       |Sí      |
|Enumerar los trabajos y ver las entidades job     |Sí         |Sí        |Sí         |
|Exportar un trabajo a un proyecto local     |No         |Sí         |Sí         |
|Enviar, iniciar y detener trabajos     |Sí         |Sí         |Sí         |
|Ver métricas y diagramas de trabajos     |Sí         |Sí         |Abrir en el portal         |
|Ver errores en tiempo de ejecución de trabajos     |Sí         |Sí         |No         |
|Registros del recurso     |Sí         |No         |No         |
|Propiedades de mensajes personalizados     |Sí         |Sí         |No       |
|Deserializador y función de código personalizada de C#|Modo de solo lectura|Sí|No|
|UDF y UDA de JavaScript     |Sí         |Sí         |Solo Windows         |
|Machine Learning Service     |Sí        |Sí         |No         |
|Machine Learning Studio     |Sí, pero la consulta no se puede probar        |Sí |No         |
|Nivel de compatibilidad     |1.0</br>1.1</br>1.2 (valor predeterminado)         |1.0</br>1.1</br>1.2 (valor predeterminado)           |1.0</br>1.1</br>1.2 (valor predeterminado)           |
|Funciones integradas de detección de anomalías basadas en ML     |Sí         |Sí         |Sí         |
|Funciones geoespaciales integradas     |Sí         |Sí         |Sí         |



## <a name="iot-edge-job-features"></a>Características de trabajos de IoT Edge

|Característica  |Portal  |Visual Studio  |Visual Studio Code  |
|---------|---------|---------|---------|
|Creación de trabajos     |Sí         |Sí         |No         |
|Control de código fuente     |No         |Sí         |No         |
|Exportar un trabajo a un proyecto local     |No         |Sí         |No         |
|Pruebas de consulta con un archivo de ejemplo     |Sí         |Sí         |No         |
|Compartir entradas y salidas entre varias consultas     |No         |Sí         |No         |
|UDF de C#     |No         |Sí         |No         |
|Envío de trabajos     |Sí         |Sí         |No         |
|Enumerar los trabajos y ver las entidades job     |Sí         |Sí         |No         |
|Ver métricas y diagramas de trabajos     |Sí         |Parcial         |No         |
|Ver errores en tiempo de ejecución de trabajos     |Sí         |Parcial         |No         |
|Compatibilidad con CI/CD     |No         |No         |No         |


## <a name="next-steps"></a>Pasos siguientes

* [Azure Stream Analytics en IoT Edge](stream-analytics-edge.md)
* [Tutorial: Escritura de una función definida por el usuario en C# para un trabajo de Azure Stream Analytics en IoT Edge (versión preliminar)](stream-analytics-edge-csharp-udf.md)
* [Desarrollo de trabajos para Stream Analytics en IoT Edge mediante las herramientas de Visual Studio](stream-analytics-tools-for-visual-studio-edge-jobs.md)
* [Uso de Visual Studio para ver trabajos de Azure Stream Analytics](stream-analytics-vs-tools.md)
* [Exploración de Azure Stream Analytics con Visual Studio Code (versión preliminar)](visual-studio-code-explore-jobs.md)


