---
title: Descripción de los formatos de datos de Apache Spark para los desarrolladores de U-SQL de Azure Data Lake Analytics
description: En este artículo se describen los conceptos de Apache Spark que ayudan a los desarrolladores de U-SQL a entender las diferencias entre los formatos de datos de Spark y U-SQL.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.custom: understand-apache-spark-data-formats
ms.date: 01/31/2019
ms.openlocfilehash: bff8c89dcdcbb7c319e04e5e7518985badf5a5ff
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132320"
---
# <a name="understand-differences-between-u-sql-and-spark-data-formats"></a>Descripción de las diferencias entre los formatos de datos de Spark y U-SQL

Si quiere usar [Azure Databricks](../azure-databricks/what-is-azure-databricks.md) o [Azure HDInsight Spark](../hdinsight/spark/apache-spark-overview.md), le recomendamos migrar los datos desde [Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md) a [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md).

Además de migrar los archivos, también querrá que Spark pueda acceder a sus datos, que están almacenados en tablas de U-SQL.

## <a name="move-data-stored-in-azure-data-lake-storage-gen1-files"></a>Migración de los datos almacenados en archivos de Azure Data Lake Storage Gen1

Los datos almacenados en archivos de pueden migrar de varias maneras:

- Escriba una canalización de [Azure Data Factory](../data-factory/introduction.md) para copiar los datos desde la cuenta de [Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md) a la cuenta de [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md).
- Escriba un trabajo de Spark que lea los datos de la cuenta de [Azure Data Lake Storage Gen1](../data-lake-store/data-lake-store-overview.md) y los escriba en la cuenta de [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md). Según el caso de uso, puede que desee escribirlos en un formato distinto, como Parquet, si no necesita conservar el formato de archivo original.

Le recomendamos revisar el artículo [Actualización de las soluciones de análisis de macrodatos de Azure Data Lake Storage Gen1 a Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-upgrade.md).

## <a name="move-data-stored-in-u-sql-tables"></a>Migración de los datos almacenados en tablas de U-SQL

Spark no entiende las tablas de U-SQL. Si tiene datos almacenados en tablas de U-SQL, ejecutará un trabajo de U-SQL que extrae los datos de la tabla y los guarda en un formato comprensible para Spark. El formato más adecuado es crear un conjunto de archivos Parquet según el diseño de la carpeta del metastore de Hive.

La salida se puede lograr en U-SQL con el outputter de Parquet integrado y el uso de la creación de particiones de salida dinámica con conjuntos de archivos para crear las carpetas de particiones. El artículo [Procesar más archivos que nunca y usar Parquet](https://blogs.msdn.microsoft.com/azuredatalake/2018/06/11/process-more-files-than-ever-and-use-parquet-with-azure-data-lake-analytics) proporciona un ejemplo de cómo crear estos datos consumibles de Spark.

Después de esta transformación, puede copiar los datos como se describe en el capítulo [Migración de los datos almacenados en archivos de Azure Data Lake Storage Gen1](#move-data-stored-in-azure-data-lake-storage-gen1-files).

## <a name="caveats"></a>Advertencias

- Semántica de los datos Al copiar los archivos, la copia se producirá en el nivel de bytes. Por lo tanto, los mismos datos deberían aparecer en la cuenta de [Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-introduction.md). Sin embargo, tenga en cuenta que Spark puede interpretar algunos caracteres de manera diferente. Por ejemplo, puede usar un valor predeterminado distinto para un delimitador de fila en un archivo .csv.
    Además, si va a copiar datos con tipo (desde tablas), es posible que Parquet y Spark tengan una precisión y una escala diferentes para algunos de los valores con tipo (por ejemplo, un tipo float) y pueden tratar distinto los valores NULL. Por ejemplo, U-SQL tiene la semántica de C# para los valores NULL, mientras que Spark tiene una lógica de tres valores para los valores NULL.

- Las tablas de U-SQL de una organización de datos (particionamiento) proporcionan una partición de dos niveles. El nivel externo (`PARTITIONED BY`) es por valor y se asigna principalmente al esquema de partición de Hive/Spark mediante jerarquías de carpetas. Deberá asegurarse de que los valores NULL estén asignados a la carpeta adecuada. El nivel interno (`DISTRIBUTED BY`) de U-SQL ofrece 4 esquemas de distribución: round robin, intervalo, hash y hash directo.
    Las tablas de Hive/Spark solo admiten particiones de valores o creación de particiones por hash, con una función hash distinta de U-SQL. Cuando se generan los datos de la tabla de U-SQL, es probable que solo se puedan asignar a las particiones de valor para Spark y que sea necesario realizar una optimización adicional del diseño de los datos en función de las consultas de Spark finales.

## <a name="next-steps"></a>Pasos siguientes

- [Descripción de los conceptos de código de Spark para desarrolladores de U-SQL](understand-spark-code-concepts.md)
- [Actualización de las soluciones de análisis de macrodatos de Azure Data Lake Storage Gen1 a Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-upgrade.md)
- [.NET para Apache Spark](https://docs.microsoft.com/dotnet/spark/what-is-apache-spark-dotnet)
- [Transformación de datos mediante la actividad de Spark en Azure Data Factory](../data-factory/transform-data-using-spark.md)
- [Transformación de datos mediante la actividad de Hive de Hadoop en Azure Data Factory](../data-factory/transform-data-using-hadoop-hive.md)
- [Qué es Apache Spark en Azure HDInsight](../hdinsight/spark/apache-spark-overview.md)
