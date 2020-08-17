---
title: 'Transformación de datos: proceso y transformación de datos '
description: Aprenda cómo transformar datos o procesar datos en Azure Data Factory mediante Hadoop, Machine Learning o Azure Data Lake Analytics.
services: data-factory
documentationcenter: ''
author: djpmsft
ms.author: daperlov
manager: jroth
ms.reviewer: maghan
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/10/2018
ms.openlocfilehash: 74cf34438ca142302a6853e8e488b0c7188b162b
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86522370"
---
# <a name="transform-data-in-azure-data-factory-version-1"></a>Transformación de datos en Azure Data Factory versión 1
> [!div class="op_single_selector"]
> * [Hive](data-factory-hive-activity.md)  
> * [Pig](data-factory-pig-activity.md)  
> * [de Hadoop](data-factory-map-reduce.md)  
> * [Hadoop Streaming](data-factory-hadoop-streaming-activity.md)
> * [Machine Learning](data-factory-azure-ml-batch-execution-activity.md) 
> * [Procedimiento almacenado](data-factory-stored-proc-activity.md)
> * [U-SQL de análisis con Data Lake](data-factory-usql-activity.md)
> * [.NET personalizado](data-factory-use-custom-activities.md)

## <a name="overview"></a>Información general
> [!NOTE]
> Este artículo se aplica a la versión 1 de Data Factory. Si usa la versión actual del servicio Data Factory, consulte el artículo acerca de las [actividades de transformación de datos en Azure Data Factory](../transform-data.md).

En este artículo se explican las actividades de transformación de datos de Azure Data Factory que puede usar para transformar y procesar los datos sin procesar en predicciones e información. Una actividad de transformación se ejecuta en un entorno informático, como clúster de Azure HDInsight o una instancia de Azure Batch. Proporciona vínculos a artículos con información detallada sobre cada actividad de transformación.

Data Factory admite las siguientes actividades de transformación de datos que se pueden agregar a las [canalizaciones](data-factory-create-pipelines.md) tanto individualmente como encadenadas a otra actividad.

> [!NOTE]
> Para obtener un tutorial con instrucciones paso a paso, consulte el artículo [Crear una canalización con la transformación de Hive](data-factory-build-your-first-pipeline.md) .  
> 
> 

## <a name="hdinsight-hive-activity"></a>Actividad de HDInsight Hive
La actividad de Hive de HDInsight en una canalización de Data Factory ejecuta consultas de Hive en su propio clúster de HDInsight o en uno a petición basado en Windows/Linux. Consulte el artículo [Actividad de Hive](data-factory-hive-activity.md) para obtener más información acerca de esta actividad. 

## <a name="hdinsight-pig-activity"></a>Actividad de HDInsight Pig
La actividad de Pig de HDInsight en una canalización de Data Factory ejecuta consultas de Pig en su propio clúster de HDInsight o en uno a petición basado en Windows/Linux. Consulte el artículo [Actividad de Pig](data-factory-pig-activity.md) para obtener más información acerca de esta actividad. 

## <a name="hdinsight-mapreduce-activity"></a>Actividad de MapReduce de HDInsight
La actividad MapReduce de HDInsight en una canalización de Data Factory ejecuta programas de MapReduce en su propio clúster de HDInsight o en uno basado en Windows/Linux a petición. Consulte el artículo [Actividad de MapReduce](data-factory-map-reduce.md) para obtener más información acerca de esta actividad.

## <a name="hdinsight-streaming-activity"></a>Actividad de HDInsight Streaming
La actividad de streaming de HDInsight en una canalización de Data Factory ejecuta programas de streaming de Hadoop en su propio clúster de HDInsight o en uno basado en Windows/Linux a petición. Vea [Actividad de HDInsight Streaming](data-factory-hadoop-streaming-activity.md) para obtener información sobre esta actividad.

## <a name="hdinsight-spark-activity"></a>Actividad de HDInsight Spark
La actividad de Spark de HDInsight en una canalización de Data Factory ejecuta consultas de Spark en su propio clúster de HDInsight. Consulte [Invoke Spark programs from Azure Data Factory](data-factory-spark.md) (Invocar programas Spark desde Data Factory de Azure) para obtener información detallada. 

## <a name="machine-learning-activities"></a>Actividades de Machine Learning
Azure Data Factory permite crear fácilmente canalizaciones que utilizan un servicio web de Azure Machine Learning publicado para realizar análisis predictivos. Mediante la [actividad de ejecución de Batch](data-factory-azure-ml-batch-execution-activity.md#invoking-a-web-service-using-batch-execution-activity) en una canalización de Azure Data Factory, puede invocar un servicio web de Machine Learning para realizar predicciones sobre los datos en el lote.

Pasado algún tiempo, los modelos predictivos en los experimentos de puntuación de Machine Learning tienen que volver a entrenarse con nuevos conjuntos de datos de entrada. Después de terminar con el nuevo entrenamiento, tendrá que actualizar el servicio web de puntuación con el modelo de Machine Learning que volvió a entrenar. Puede usar la [Actividad de recursos de actualización](data-factory-azure-ml-batch-execution-activity.md#updating-models-using-update-resource-activity) para actualizar el servicio web con el modelo recién entrenado.  

Consulte [Usar actividades de Machine Learning](data-factory-azure-ml-batch-execution-activity.md) para obtener más información sobre estas actividades de Machine Learning. 

## <a name="stored-procedure-activity"></a>Actividad de procedimiento almacenado
Puede usar la actividad de procedimiento almacenado de SQL Server en una canalización de Data Factory para invocar un procedimiento almacenado en uno de los siguientes almacenes de datos: Azure SQL Database, Azure SQL Data Warehouse y base de datos de SQL Server en una empresa o una máquina virtual de Azure. Consulte el artículo [Actividad de procedimiento almacenado](data-factory-stored-proc-activity.md) para obtener más información.  

## <a name="data-lake-analytics-u-sql-activity"></a>Actividad de U-SQL de Data Lake Analytics
La actividad de U-SQL de Data Lake Analytics ejecuta un script de U-SQL en un clúster de Azure Data Lake Analytics. Consulte el artículo [Actividad de U-SQL de Data Analytics](data-factory-usql-activity.md) para obtener más información. 

## <a name="net-custom-activity"></a>Actividad personalizada de .NET
Si necesita transformar datos de algún modo no compatible con Data Factory, puede crear una actividad personalizada con su propia lógica de procesamiento de datos y usarla en la canalización. Puede configurar una actividad de .NET personalizada para ejecutarse mediante un servicio Azure Batch o un clúster de Azure HDInsight. Consulte el artículo [Utilizar actividades personalizadas](data-factory-use-custom-activities.md) para obtener más información. 

Puede crear una actividad personalizada para ejecutar scripts de R en su clúster de HDInsight con R instalado. Consulte [Run R Script using Azure Data Factory](https://github.com/Azure/Azure-DataFactory/tree/master/SamplesV1/RunRScriptUsingADFSample)(Ejecutar script de R con Data Factory de Azure). 

## <a name="compute-environments"></a>Entornos de proceso
Deberá crear un servicio vinculado para el entorno de proceso y después usar el servicio vinculado al definir una actividad de transformación. La Factoría de datos admite dos tipos de entornos de proceso. 

1. **A petición**:  en este caso, Data Factory administra completamente el entorno informático. El servicio Factoría de datos lo crea automáticamente antes de que se envíe un trabajo para procesar los datos y que se quite cuando finalice el trabajo. Los usuarios pueden configurar y controlar la configuración granular del entorno de proceso a petición para la ejecución del trabajo, la administración del clúster y las acciones de arranque. 
2. **Traiga el suyo propio**: en este caso, puede registrar su propio entorno informático (por ejemplo, clúster de HDInsight) como servicio vinculado en Data Factory. El usuario administra el entorno de procesos y el servicio Factoría de datos lo usa para ejecutar las actividades. 

Vea el artículo [Servicios vinculados de procesos](data-factory-compute-linked-services.md) para obtener información sobre los servicios vinculados de proceso compatibles con Data Factory. 

## <a name="summary"></a>Resumen
Azure Data Factory admite las siguientes actividades de transformación de datos y los entornos de proceso para las actividades. Las actividades de transformación se pueden agregar a las canalizaciones tanto individualmente como encadenadas a otra actividad.

| Actividad de transformación de datos | Entorno de procesos |
|:--- |:--- |
| [Hive](data-factory-hive-activity.md) |HDInsight [Hadoop] |
| [Pig](data-factory-pig-activity.md) |HDInsight [Hadoop] |
| [de Hadoop](data-factory-map-reduce.md) |HDInsight [Hadoop] |
| [Hadoop Streaming](data-factory-hadoop-streaming-activity.md) |HDInsight [Hadoop] |
| [Actividades de Machine Learning: ejecución de lotes y recurso de actualización](data-factory-azure-ml-batch-execution-activity.md) |Azure VM |
| [Procedimiento almacenado](data-factory-stored-proc-activity.md) |SQL Azure, Azure SQL Data Warehouse o SQL Server |
| [U-SQL de análisis con Data Lake](data-factory-usql-activity.md) |Análisis con Azure Data Lake |
| [DotNet](data-factory-use-custom-activities.md) |HDInsight [Hadoop] o Azure Batch |

