---
title: ¿Qué es Azure Synapse Analytics?
description: Información general de Azure Synapse Analytics
services: synapse-analytics
author: saveenr
ms.service: synapse-analytics
ms.topic: overview
ms.subservice: overview
ms.date: 04/15/2020
ms.author: saveenr
ms.reviewer: jrasnick
ms.openlocfilehash: f9277842339d7c4fa74e2e0f1febcf5916d86f4a
ms.sourcegitcommit: 3fb5e772f8f4068cc6d91d9cde253065a7f265d6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/31/2020
ms.locfileid: "89180786"
---
# <a name="what-is-azure-synapse-analytics-workspaces-preview"></a>¿Qué es Azure Synapse Analytics (versión preliminar para área de trabajo)?

[!INCLUDE [preview](includes/note-preview.md)]

Hoy en día, el análisis empresarial requiere un funcionamiento a gran escala en cualquier tipo de datos, ya sea sin procesar, modificado o sumamente mantenidos. En el pasado, la creación de estos tipos de soluciones de análisis requerían que las empresas combinaran tecnologías de almacenamiento de datos y macrodatos, como Spark y SQL. A continuación, debían integrarse en canalizaciones de datos enriquecidas que funcionan en los datos de almacenes relacionales y lagos de datos.  

Soluciones como esta son difíciles de compilar, configurar, proteger y mantener, lo que retrasa la extracción rápida de la información inteligente.

**Azure Synapse** es un servicio de análisis integrado que acelera el tiempo para obtener información de todos los datos a cualquier escala, entre los almacenamientos de datos y los sistemas de análisis de macrodatos. Reúne lo mejor de las tecnologías **SQL** que se usan en el almacenamiento de datos de empresa, las tecnologías **Spark** que se utilizan en el análisis de macrodatos y **Pipelines** para la integración de datos y la extracción, transformación y carga de datos.

Azure Synapse incluye una experiencia de usuario web nativa de **Studio** que proporciona una experiencia y un modelo únicos de administración, supervisión, codificación y seguridad.

Azure Synapse proporciona la forma más sencilla y rápida para que una empresa recopile información sobre los datos a cualquier tamaño, mediante el análisis con el que está familiarizado. Se integra profundamente con **Power BI** para permitir a los ingenieros de datos crear soluciones de análisis que funcionen de un extremo a otro para proporcionar inteligencia empresarial.

Además, Azure Synapse facilita la creación de modelos predictivos y el análisis avanzado con aprendizaje automático a través de la compatibilidad integrada con **AzureML**.

## <a name="key-features--benefits"></a>Principales características y ventajas

### <a name="industry-leading-sql"></a>SQL líder del sector

* **Synapse SQL** es un sistema de consultas distribuidas que permite a las empresas implementar escenarios de virtualización de datos y almacenamiento de datos mediante experiencias estándar de T-SQL conocidas para los ingenieros de datos. También amplía las funcionalidades de SQL para abordar los escenarios de streaming y aprendizaje automático.

* Synapse SQL ofrece modelos de recursos **sin servidor** y **aprovisionados**, que ofrecen opciones de facturación y consumo para satisfacer sus necesidades. Para obtener un rendimiento y un costo predecibles, aprovisione los grupos para reservar la capacidad de procesamiento de los datos almacenados en tablas SQL. En el caso de cargas de trabajo no planeadas o en ráfagas, utilice el punto de conexión SQL sin servidor siempre disponible.
* Use las funcionalidades integradas de **streaming** para destinar datos de los orígenes de datos en la nube a tablas SQL.
* Integración de IA con SQL mediante el uso de modelos de **aprendizaje automático** para puntuar los datos mediante la [función PREDICT de T-SQL](https://docs.microsoft.com/sql/t-sql/queries/predict-transact-sql?view=azure-sqldw-latest)

### <a name="industry-standard-apache-spark"></a>Apache Spark estándar del sector

**Apache Spark para Azure Synapse** se integra profundamente y sin fisuras en Apache Spark, el motor de macrodatos de código abierto más popular que se usa para la preparación de datos, la ingeniería de datos, ETL y el aprendizaje automático.

* Modelos de aprendizaje automático con algoritmos de SparkML e integración de AzureML para Apache Spark 2.4 con compatibilidad integrada con el modelo delta de Linux Foundation.
* Modelo de recursos simplificado que evita tener que preocuparse por la administración de clústeres.
* Inicio rápido de Spark y escalado automático agresivo.
* Compatibilidad integrada de .NET para Spark que le permite volver a usar la experiencia en C# y el código .NET existente en una aplicación de Spark.

### <a name="interop-of-sql-and-apache-spark-on-your-data-lake"></a>Interoperabilidad de SQL y Apache Spark en la instancia de Data Lake

Azure Synapse elimina las barreras de la tecnología tradicionales entre el uso conjunto de SQL y Spark. Puede combinar y comparar sin problemas en función de sus necesidades y conocimientos.

* Un sistema de metadatos compatible con Hive compartido permite que las tablas definidas en los archivos del lago de datos se consuman sin problemas mediante Spark o Hive.
* SQL y Spark pueden explorar y analizar directamente los archivos Parquet, CSV, TSV y JSON almacenados en el lago de datos.
* Carga y descarga rápida y escalable para los datos que fluyen entre las bases de datos SQL y Spark

### <a name="built-in-data-integration-via-pipelines"></a>Integración de datos integrada a través de canalizaciones

Azure Synapse incluye el mismo motor de integración de datos y experiencias que Azure Data Factory, lo que permite crear canalizaciones de ELT a escala enriquecidas sin tener que salir de Synapse Analytics.

* Ingesta de datos de más de 90 orígenes de datos
* ETL sin código con actividades de flujo de datos
* Orquestación de cuadernos, trabajos de Spark, procedimientos almacenados, scripts de SQL, etc

### <a name="unified-management-monitoring-and-security"></a>Administración, supervisión y seguridad unificadas

Azure Synapse proporciona a las empresas una única forma de administrar los recursos de análisis, supervisar el uso y la actividad, y aplicar la seguridad.

* Asignación de usuarios a un rol para simplificar el acceso a los recursos de Analytics
* Control de acceso específico en los datos y el código
* Un solo panel para supervisar los recursos, el uso y los usuarios en SQL y Spark

### <a name="synapse-studio"></a>Synapse Studio

**Synapse Studio** es la experiencia web-nativa que relaciona todo con los ingenieros de datos, lo que les permite realizar todas las tareas necesarias para crear una solución completa.

* Creación de una solución de análisis de un extremo a otro en un solo lugar: ingesta, exploración, preparación, orquestación, visualización
* Productividad líder del sector para ingenieros de datos que escriben código SQL o Spark: creación, depuración y optimización del rendimiento
* Integración con procesos de CI/CD empresariales

## <a name="next-steps"></a>Pasos siguientes

* [Creación de un área de trabajo](quickstart-create-workspace.md)
* [Inicio rápido: Uso de Synapse Studio (versión preliminar)](quickstart-synapse-studio.md)
* [Creación de un grupo de SQL](quickstart-create-sql-pool-portal.md)
* [Uso de SQL a petición](quickstart-sql-on-demand.md)
* [Creación de un grupo de Apache Spark](quickstart-create-apache-spark-pool-portal.md)
