---
title: ¿Qué es Azure SQL Edge (versión preliminar)?
description: Información sobre Azure SQL Edge (versión preliminar)
keywords: introducción a SQL Edge, qué es SQL Edge, información general sobre SQL Edge
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: 2c96e4b7baa2c463c42db9440cadb3cb396fde1b
ms.sourcegitcommit: 628be49d29421a638c8a479452d78ba1c9f7c8e4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88642476"
---
# <a name="what-is-azure-sql-edge-preview"></a>¿Qué es Azure SQL Edge (versión preliminar)?

Azure SQL Edge (versión preliminar) es un motor de base de datos relacional optimizado y orientado a implementaciones de IoT e IoT Edge. Proporciona funcionalidades para crear una capa de procesamiento y almacenamiento de datos de alto rendimiento para las aplicaciones y soluciones de IoT. Azure SQL Edge proporciona funcionalidades para transmitir, procesar y analizar datos relacionales y no relacionales como JSON, gráficos y datos de series temporales, lo que hace que sea la opción adecuada para una variedad de aplicaciones de IoT modernas.

Azure SQL Edge se basa en las versiones más recientes del Motor de base de datos de Microsoft SQL Server (/sql/sql-server/sql-server-technical-documentation?toc=/azure/azure-sql-edge/toc.json), que ofrece funcionalidades de rendimiento, seguridad y procesamiento de consultas líderes en el sector. Dado que Azure SQL Edge se basa en el mismo motor que [SQL Server](/sql/sql-server/sql-server-technical-documentation?toc=/azure/azure-sql-edge/toc.json) y [Azure SQL](https://docs.microsoft.com/azure/azure-sql/), proporciona la misma área expuesta de programación de T-SQL, la cual hace que el desarrollo de aplicaciones o soluciones sea más fácil y rápido, y al mismo tiempo hace que la portabilidad de las aplicaciones entre los dispositivos IoT Edge, los centros de datos y la nube sea más directa.

> [!NOTE]
> Azure SQL Edge está actualmente en versión preliminar y NO debe usarse en entornos de producción.

## <a name="deployment-models"></a>Modelos de implementación

Azure SQL Edge está disponible en Azure Marketplace y se puede implementar como un módulo para [Azure IoT Edge](../iot-edge/about-iot-edge.md). Para obtener más información, consulte [Implementación de Azure SQL Edge](deploy-portal.md).<br>

![Diagrama general de SQL Edge](media/overview/overview.png)

## <a name="editions-of-sql-edge"></a>Ediciones de SQL Edge

SQL Edge está disponible en dos ediciones o planes de software diferentes. Estas ediciones tienen conjuntos de características idénticos y solo difieren en cuanto a sus derechos de uso y la cantidad de CPU o memoria que admiten.

   |**Plan**  |**Descripción**  |
   |---------|---------|
   |Desarrollador de Azure SQL Edge  |  SKU solo de desarrollo; cada contenedor de SQL Edge está limitado a hasta 4 núcleos y 32 GB de memoria  |
   |Azure SQL Edge    |  SKU de producción; cada contenedor de SQL Edge está limitado a hasta 8 núcleos y 64 GB de memoria. |

## <a name="pricing-and-availability"></a>Precios y disponibilidad

Actualmente, Azure SQL Edge se encuentra en versión preliminar. Para obtener más información sobre los precios y la disponibilidad, consulte [Azure SQL Edge](https://azure.microsoft.com/services/sql-edge/).

> [!IMPORTANT]
> Para comprender las diferencias en las características de Azure SQL Edge y SQL Server, así como las que existen entre distintas opciones de Azure SQL Edge, consulte [Características admitidas de Azure SQL Edge](features.md).

## <a name="streaming-capabilities"></a>Capacidades de streaming  

Azure SQL Edge ofrece funcionalidades de streaming integradas para el análisis en tiempo real y el procesamiento de eventos complejos. La funcionalidad de streaming se basa en las mismas construcciones que [Azure Stream Analytics](../stream-analytics/stream-analytics-introduction.md) y en funcionalidades similares a [Azure Stream Analytics en IoT Edge](../stream-analytics/stream-analytics-edge.md).

El motor de streaming de Azure SQL Edge está diseñado para ofrecer baja latencia, resistencia, uso eficiente del ancho de banda y cumplimiento. 

Para obtener más información sobre el streaming de datos en SQL Edge, consulte [Streaming de datos](stream-data.md).

## <a name="machine-learning-and-artificial-intelligence-capabilities"></a>Funcionalidades de aprendizaje automático e inteligencia artificial

Azure SQL Edge proporciona funcionalidades de análisis y aprendizaje automático integradas mediante la integración del runtime de formato abierto de ONNX (Open Neuronal Network Exchange), que permite el intercambio de modelos de red neuronal y de aprendizaje profundo entre diferentes marcos. Para obtener más información sobre ONNX, vaya [aquí](https://onnx.ai/). El runtime de ONNX proporciona la flexibilidad para desarrollar modelos en el lenguaje o las herramientas de su elección, que luego se pueden convertir al formato ONNX para su ejecución en SQL Edge. Para obtener más información, consulte [Aprendizaje automático e inteligencia artificial con ONNX en SQL Edge](onnx-overview.md).

## <a name="working-with-azure-sql-edge"></a>Uso de Azure SQL Edge

Azure SQL Edge facilita el desarrollo y el mantenimiento de aplicaciones y aumenta su productividad. Los usuarios pueden usar todas las herramientas y los conocimientos que ya conocen para crear excelentes aplicaciones y soluciones para sus necesidades de IoT Edge. El usuario puede desarrollar en SQL Edge mediante herramientas como las siguientes:

- [Azure Portal](https://portal.azure.com/): una aplicación web para administrar todos los servicios de Azure.
- [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms/): una aplicación cliente gratuita que se puede descargar para administrar cualquier infraestructura de SQL, desde SQL Server hasta SQL Database.
- [SQL Server Data Tools en Visual Studio](/sql/ssdt/download-sql-server-data-tools-ssdt/): una aplicación cliente gratuita que se puede descargar para desarrollar bases de datos relacionales de SQL Server, bases de datos de SQL, paquetes de Integration Services, modelos de datos de Analysis Services e informes de Reporting Services.
- [Azure Data Studio](/sql/azure-data-studio/what-is/): una herramienta multiplataforma gratuita de base de datos que se puede descargar, ideal para profesionales de datos que utilizan la familia Microsoft de plataformas de datos locales y en nube para Windows, macOS y Linux.
- [Visual Studio Code](https://code.visualstudio.com/docs): editor de código abierto gratuito que se puede descargar para Windows, macOS y Linux. Admite extensiones, como la [extensión mssql](https://aka.ms/mssql-marketplace) para consultar Microsoft SQL Server, Azure SQL Database y SQL Azure Data Warehouse.


## <a name="next-steps"></a>Pasos siguientes

- [Implementación de SQL Edge mediante Azure Portal](deploy-portal.md)
- [Aprendizaje automático e inteligencia artificial con SQL Edge](onnx-overview.md)
- [Creación de una solución de IoT global con SQL Edge](tutorial-deploy-azure-resources.md)
