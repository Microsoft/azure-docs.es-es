---
title: ¿Qué es Azure Databricks?
description: Obtenga información acerca de qué es Azure Databricks y cómo lleva Spark en Databricks a Azure. Azure Databricks es una plataforma de análisis basada en Apache Spark optimizada para la plataforma de servicios en la nube de Microsoft Azure.
services: azure-databricks
author: mamccrea
ms.reviewer: jasonh
ms.service: azure-databricks
ms.workload: big-data
ms.topic: overview
ms.date: 05/08/2019
ms.author: mamccrea
ms.custom: mvc
ms.openlocfilehash: 0864db682141b1d7f8cb0c9ff14c9897fb02fe1b
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65412755"
---
# <a name="what-is-azure-databricks"></a>¿Qué es Azure Databricks?

Azure Databricks es una plataforma de análisis basada en Apache Spark optimizada para la plataforma de servicios en la nube de Microsoft Azure. Diseñada por los fundadores de Apache Spark, Databricks está integrado con Azure para proporcionar una configuración con un solo clic, flujos de trabajo optimizados y un área de trabajo interactiva que permite la colaboración entre científicos de datos, ingenieros de datos y analistas empresariales.

![¿Qué es Azure Databricks? ](./media/what-is-azure-databricks/azure-databricks-overview.png "¿Qué es Azure Databricks?")

Azure Databricks es un servicio de análisis rápido, sencillo y de colaboración basado en la plataforma de análisis Apache Spark. Para una canalización de macrodatos, los datos (estructurados o sin formato) se ingieren en Azure mediante Azure Data Factory en lotes o transmitidos casi en tiempo real con Kafka, Event Hub o IoT Hub. Estos datos llegan a un lago de datos para un almacenamiento persistente a largo plazo en Azure Blob Storage o Azure Data Lake Storage. Como parte del flujo de trabajo de análisis, use Azure Databricks para leer datos desde varios orígenes de datos, como [Azure Blob Storage](../storage/blobs/storage-blobs-introduction.md), [Azure Data Lake Storage](../data-lake-store/index.md), [Azure Cosmos DB](../cosmos-db/index.yml) o [Azure SQL Data Warehouse](../sql-data-warehouse/index.md), y convertirlos en conclusiones importante sobre el uso de Spark.

![Canalización de Databricks](./media/what-is-azure-databricks/databricks-pipeline.png)

## <a name="apache-spark-based-analytics-platform"></a>Plataforma de análisis basada en Apache Spark

Azure Databricks consta de las tecnologías y funcionalidades de clúster de Apache Spark de código abierto integrales. En Azure Databricks, Spark incluye los siguientes componentes:

![Apache Spark en Azure Databricks](./media/what-is-azure-databricks/apache-spark-ecosystem-databricks.png "Apache Spark en Azure Databricks")

* **Spark SQL y DataFrames**: Spark SQL es el módulo de Spark para trabajar con datos estructurados. Una trama de datos es una colección distribuida de datos que se organizan en columnas con nombre. Es conceptualmente equivalente a una tabla en una base de datos relacional o a una trama de datos en R/Python.

* **Streaming**: procesamiento y análisis de datos en tiempo real para aplicaciones analíticas e interactivas. Se integra con HDFS, Flume y Kafka.

* **MLib**: biblioteca de Machine Learning que consta de algoritmos y utilidades de aprendizaje comunes, como la clasificación, la regresión, la agrupación en clústeres, el filtrado colaborativo, la reducción de dimensionalidad y las primitivas de optimización subyacentes.

* **GraphX**: grafos y cálculo de grafos para una amplia gama de casos de uso, desde los análisis cognitivos hasta la exploración de datos.

* **Spark Core API**: incluye compatibilidad con R, SQL, Python, Scala y Java.

## <a name="apache-spark-in-azure-databricks"></a>Apache Spark en Azure Databricks

Azure Databricks proporciona una plataforma en la nube sin administración basada en las funcionalidades de Spark que incluye:

- Clústeres de Spark completamente administrados
- Un área de trabajo interactiva de exploración y visualización
- Una plataforma para activar las aplicaciones favoritas basadas en Spark

### <a name="fully-managed-apache-spark-clusters-in-the-cloud"></a>Clústeres de Apache Spark completamente administrados en la nube

Azure Databricks tiene un entorno de producción seguro y confiable en la nube, administrado y con el soporte técnico de expertos en Spark. Puede:

* Crear clústeres en segundos.
* Escalar y reducir clústeres automática y dinámicamente en vertical (clústeres sin servidor incluidos) y compartirlos entre equipos. 
* Usar clústeres mediante programación con las API de REST. 
* Utilizar funcionalidades seguras de integración de datos basadas en Spark que permiten unificar los datos sin centralización. 
* Acceder inmediatamente a las últimas características de Apache Spark con cada versión.

### <a name="databricks-runtime"></a>Entorno de tiempo de ejecución de Databricks
El entorno de tiempo de ejecución de Databricks se basa en Apache Spark y se ha creado de forma nativa para la nube de Azure. 

Con la opción **Serverless** (Sin servidor), Azure Databricks elimina completamente la complejidad de la infraestructura y la necesidad de experiencia especializada para instalar y configurar la infraestructura de datos. La opción Serverless (Sin servidor) ayuda a los científicos de datos a iterar rápidamente como equipo.

Para los ingenieros de datos, a quienes interesa el rendimiento de las tareas de producción, Azure Databricks proporciona un motor de Spark más rápido y eficiente gracias a varias optimizaciones en la capa de E/S y la de procesamiento (Databricks E/S).

### <a name="workspace-for-collaboration"></a>Área de trabajo para la colaboración

A través de un entorno de colaboración integrado, Azure Databricks optimiza el proceso de exploración de datos, la creación de prototipos y la ejecución de aplicaciones basadas en datos en Spark.

* Determine cómo usar los datos con una exploración de datos sencilla.
* Documente el progreso en blocs de notas en R, Python, Scala o SQL.
* Visualice los datos en unos clics y use herramientas conocidas como Matplotlib, ggplot o d3.
* Use paneles interactivos para crear informes dinámicos.
* Use Spark e interactúe con los datos al mismo tiempo.

## <a name="enterprise-security"></a>Seguridad de la empresa

Azure Databricks proporciona seguridad empresarial de Azure, incluida la integración de Azure Active Directory, controles basados en roles y Acuerdos de Nivel de Servicio que protegen los datos y la empresa.

* La integración con Azure Active Directory permite ejecutar soluciones completas basadas en Azure con Azure Databricks.
* El acceso basado en roles de Azure Databricks permite permisos de usuario específicos para los cuadernos, los clústeres, los trabajos y los datos.
* Acuerdos de Nivel de Servicio de clase empresarial. 

## <a name="integration-with-azure-services"></a>Integración con servicios de Azure

Azure Databricks se integra totalmente con los almacenes y las bases de datos de Azure: SQL Data Warehouse, Cosmos DB, Data Lake Store y Blob Storage. 

## <a name="integration-with-power-bi"></a>Integración con Power BI
Mediante la integración enriquecida con Power BI, Azure Databricks permite detectar y compartir información potente de manera rápida y sencilla. También puede usar otras herramientas de BI, como Tableau Software, a través de puntos de conexión de clúster de JDBC/ODBC.

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido: ejecución de un trabajo de Spark en Azure Databricks](quickstart-create-databricks-workspace-portal.md)
* [Uso de los clústeres de Spark](https://docs.azuredatabricks.net/user-guide/clusters/index.html)
* [Uso de cuadernos](https://docs.azuredatabricks.net/user-guide/notebooks/index.html)
* [Creación de trabajos de Spark](https://docs.azuredatabricks.net/user-guide/jobs.html)

 









