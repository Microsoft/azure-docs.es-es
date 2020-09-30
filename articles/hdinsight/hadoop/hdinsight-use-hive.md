---
title: ¿Qué son Apache Hive y HiveQL en Azure HDInsight?
description: Apache Hive es un sistema de almacenamiento de datos para Apache Hadoop. Puede consultar datos almacenados en Hive mediante HiveQL, que se parece a Transact-SQL. En este documento, aprenda a usar Hive y HiveQL con Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive,hdiseo17may2017
ms.date: 02/28/2020
ms.openlocfilehash: 37fdf863d29015bba7015fcff1ae49a34aebd785
ms.sourcegitcommit: 4a7a4af09f881f38fcb4875d89881e4b808b369b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89462282"
---
# <a name="what-is-apache-hive-and-hiveql-on-azure-hdinsight"></a>¿Qué son Apache Hive y HiveQL en Azure HDInsight?

[Apache Hive](https://hive.apache.org/) es un sistema de almacenamiento de datos para Apache Hadoop. Hive hace posibles el resumen de los datos, las consultas y el análisis de datos. Las consultas de Hive se escriben en HiveQL, que es un lenguaje de consulta similar a SQL.

Hive le permite proyectar la estructura del proyecto en datos que en gran medida no están estructurados. Después de definir la estructura, puede usar HiveQL para consultar los datos sin conocimientos de Java ni MapReduce.

HDInsight proporciona varios tipos de clúster, que están optimizados para cargas de trabajo específicas. Se suelen usar los siguientes tipos de clúster para consultas de Hive:

|Tipo de clúster |Descripción|
|---|---|
|Interactive Query|un clúster de Hadoop que proporciona funcionalidad de [procesamiento analítico de baja latencia (LLAP)](https://cwiki.apache.org/confluence/display/Hive/LLAP) para mejorar los tiempos de respuesta de las consultas interactivas. Para más información, vea el documento [Uso de Hive interactivo con HDInsight (versión preliminar)](../interactive-query/apache-interactive-query-get-started.md).|
|Hadoop|un clúster de Hadoop que está optimizado para cargas de trabajo de procesamiento por lotes. Para más información, consulte el documento [Introducción a Apache Hadoop en HDInsight](../hadoop/apache-hadoop-linux-tutorial-get-started.md).|
|Spark|Apache Spark tiene funcionalidad integrada para trabajar con Hive. Para más información, consulte el documento [Introducción a Apache Spark en HDInsight](../spark/apache-spark-jupyter-spark-sql.md).|
|HBase|se puede usar HiveQL para consultar datos almacenados en Apache HBase. Para más información, consulte el documento [Introducción a Apache HBase en HDInsight](../hbase/apache-hbase-tutorial-get-started-linux.md).|

## <a name="how-to-use-hive"></a>Uso de Hive

Utilice la siguiente tabla para descubrir distintas formas de usar Hive con HDInsight:

| **Use este método** si desea… | ... consultas **interactivas** | ...procesamiento por**lotes** | ...desde este **sistema operativo de cliente** |
|:--- |:---:|:---:|:--- |:--- |
| [Herramientas de HDInsight para Visual Studio Code](../hdinsight-for-vscode.md) |✔ |✔ | Linux, Unix, Mac OS X o Windows |
| [Herramientas de HDInsight para Visual Studio](../hadoop/apache-hadoop-use-hive-visual-studio.md) |✔ |✔ |Windows |
| [Vista de Hive](../hadoop/apache-hadoop-use-hive-ambari-view.md) |✔ |✔ |Cualquiera (en función del explorador) |
| [Cliente Beeline](../hadoop/apache-hadoop-use-hive-beeline.md) |✔ |✔ |Linux, Unix, Mac OS X o Windows |
| [REST API](../hadoop/apache-hadoop-use-hive-curl.md) |&nbsp; |✔ |Linux, Unix, Mac OS X o Windows |
| [Windows PowerShell](../hadoop/apache-hadoop-use-hive-powershell.md) |&nbsp; |✔ |Windows |

## <a name="hiveql-language-reference"></a>Referencia del lenguaje HiveQL

La referencia del lenguaje de HiveQL está disponible en el [manual del lenguaje](https://cwiki.apache.org/confluence/display/Hive/LanguageManual).

## <a name="hive-and-data-structure"></a>Hive y estructura de datos

Hive entiende cómo trabajar con los datos estructurados y semiestructurados. Por ejemplo, archivos de texto donde los campos están delimitados por caracteres específicos. La siguiente instrucción HiveQL crea una tabla de datos delimitados por espacios:

```hiveql
CREATE EXTERNAL TABLE log4jLogs (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE LOCATION '/example/data/';
```

Hive también admite **serializador/deserializadores (SerDe)** personalizados para datos estructurados irregularmente o complejos. Para obtener más información, consulte el documento [Uso de un SerDe de JSON personalizado con HDInsight](https://web.archive.org/web/20190217104719/https://blogs.msdn.microsoft.com/bigdatasupport/2014/06/18/how-to-use-a-custom-json-serde-with-microsoft-azure-hdinsight/).

Para obtener más información sobre formatos de archivo compatibles con Hive, consulte el [manual del lenguaje (https://cwiki.apache.org/confluence/display/Hive/LanguageManual)](https://cwiki.apache.org/confluence/display/Hive/LanguageManual)

### <a name="hive-internal-tables-vs-external-tables"></a>Tablas internas frente a tablas externas de Hive.

Hay dos tipos de tablas que puede crear con Hive:

* __Internas__: los datos se almacenan en el almacenamiento de datos de Hive. El almacenamiento de datos se encuentra en `/hive/warehouse/` en el almacenamiento predeterminado para el clúster.

    Use las tablas internas cuando se cumpla alguna de las condiciones siguientes:

    * Los datos sean temporales.
    * Desee que Hive administre el ciclo de vida de la tabla y los datos.

* __Externas__: los datos se almacenan fuera del almacenamiento de datos. Los datos se pueden almacenar en cualquier almacenamiento accesible desde el clúster.

    Use las tablas externas cuando se cumpla alguna de las condiciones siguientes:

    * Los datos también se utilicen fuera de Hive. Por ejemplo, los archivos de datos se actualizan en otro proceso (que no bloquea los archivos).
    * Los datos deban permanecer en la ubicación subyacente, incluso después de eliminarse la tabla.
    * Necesite una ubicación personalizada, como una cuenta de almacenamiento no predeterminada.
    * Un programa distinto de Hive administre el formato de datos, la ubicación, etc.

Para más información, consulte la entrada de blog [Hive Internal and External Tables Intro](https://blogs.msdn.microsoft.com/cindygross/2013/02/05/hdinsight-hive-internal-and-external-tables-intro/) (Introducción a las tablas internas y externas de Hive).

## <a name="user-defined-functions-udf"></a>Funciones definidas por el usuario (UDF)

Hive también puede extenderse a través de **funciones definidas por el usuario (UDF)** . Una UDF le permite implementar la funcionalidad o la lógica que no se modela con facilidad en HiveQL. Para obtener un ejemplo del uso de UDF con Hive, vea los siguientes documentos:

* [Utilización de una función definida por el usuario de Java con Apache Hive](../hadoop/apache-hadoop-hive-java-udf.md)

* [Uso de funciones definidas por el usuario de Python con Apache Hive](../hadoop/python-udf-hdinsight.md)

* [Usar funciones definidas por el usuario de C# con Apache Hive](../hadoop/apache-hadoop-hive-pig-udf-dotnet-csharp.md)

* [Cómo agregar una función definida por el usuario de Apache Hive personalizada a HDInsight](https://docs.microsoft.com/archive/blogs/bigdatasupport/how-to-add-custom-hive-udfs-to-hdinsight)

* [Un ejemplo de función definida por el usuario de Apache Hive para convertir formatos de fecha y hora en una marca de tiempo de Hive](https://github.com/Azure-Samples/hdinsight-java-hive-udf)

## <a name="example-data"></a>Datos de ejemplo

Hive en HDInsight tiene ya cargada una tabla interna denominada `hivesampletable`. Además, HDInsight proporciona conjuntos de datos de ejemplo que se pueden usar con Hive. Estos conjuntos de datos se almacenan en los directorios `/example/data` y `/HdiSamples`. Estos directorios existen en el almacenamiento predeterminado del clúster.

## <a name="example-hive-query"></a>Ejemplo de consulta de Hive

Las siguientes instrucciones de HiveQL proyectan columnas sobre el archivo `/example/data/sample.log`:

```hiveql
DROP TABLE log4jLogs;
CREATE EXTERNAL TABLE log4jLogs (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ' '
STORED AS TEXTFILE LOCATION '/example/data/';
SELECT t4 AS sev, COUNT(*) AS count FROM log4jLogs
    WHERE t4 = '[ERROR]' AND INPUT__FILE__NAME LIKE '%.log'
    GROUP BY t4;
```

En el ejemplo anterior, las instrucciones de HiveQL realizan las acciones siguientes:

|. |Descripción |
|---|---|
|DROP TABLE|si la tabla ya existe, la elimina.|
|CREATE EXTERNAL TABLE|crea una nueva tabla **externa** en Hive. Las tablas externas solo almacenan la definición de Tabla en Hive. Los datos permanecen en la ubicación y formato originales.|
|ROW FORMAT|indica a Hive cómo se da formato a los datos. En este caso, los campos de cada registro se separan mediante un espacio.|
|STORED AS TEXTFILE LOCATION|indica a Hive dónde se almacenan los datos (el directorio `example/data`) y que se almacenen como texto. Los datos pueden estar en un archivo o distribuidos en varios archivos dentro del directorio.|
|SELECT|selecciona el número total de filas donde la columna **t4** contiene el valor **[ERROR]** . Esta instrucción devuelve un valor de **3** porque hay tres filas que contienen este valor.|
|INPUT__FILE__NAME LIKE '%.log'|Hive intenta aplicar el esquema a todos los archivos en el directorio. En este caso, el directorio contiene archivos que no coinciden con el esquema. Para evitar que haya datos inservibles en los resultados, esta instrucción indica a Hive que solo se deben devolver datos de archivos que terminen en .log.|

> [!NOTE]  
> Las tablas externas se deben utilizar cuando se espera que un origen externo actualice los datos subyacentes. Por ejemplo, un proceso de carga de datos automatizado o una operación de MapReduce.
>
> La eliminación de una tabla externa **no** elimina los datos, solamente la definición de tabla.

Para crear una tabla **interna** en lugar de una externa, use el siguiente HiveQL:

```hiveql
CREATE TABLE IF NOT EXISTS errorLogs (
    t1 string,
    t2 string,
    t3 string,
    t4 string,
    t5 string,
    t6 string,
    t7 string)
STORED AS ORC;
INSERT OVERWRITE TABLE errorLogs
SELECT t1, t2, t3, t4, t5, t6, t7 
    FROM log4jLogs WHERE t4 = '[ERROR]';
```

Estas instrucciones realizan las acciones siguientes:

|. |Descripción |
|---|---|
|CREATE TABLE IF NOT EXISTS|Si la tabla no existe, créela. Dado que no se utiliza la palabra clave **EXTERNAL**, esta instrucción crea una tabla interna. La tabla se almacena en el almacenamiento de datos de Hive y Hive la administra por completo.|
|STORED AS ORC|almacena los datos en el formato de columnas de filas optimizadas (ORC, Optimized Row Columnar). ORC es un formato altamente optimizado y eficiente para almacenar datos de Hive.|
|INSERT OVERWRITE... SELECT|selecciona filas de la tabla **log4jLogs** que contiene **[ERROR]** y luego inserta los datos en la tabla **errorLogs**.|

> [!NOTE]  
> A diferencia de las tablas externas, la eliminación de una tabla interna también eliminará los datos subyacentes.

## <a name="improve-hive-query-performance"></a>Mejora del rendimiento de las consultas de Hive

### <a name="apache-tez"></a>Apache Tez

[Apache Tez](https://tez.apache.org) es un marco que permite que aplicaciones con uso intensivo de datos, como Hive, se ejecuten con mucha más eficacia a escala. Tez está habilitado de manera predeterminada.  Los [documentos de diseño de Apache Hive en Tez](https://cwiki.apache.org/confluence/display/Hive/Hive+on+Tez) incluyen detalles sobre las opciones de implementación y las configuraciones de ajuste.

### <a name="low-latency-analytical-processing-llap"></a>Procesamiento analítico de baja latencia (LLAP)

[LLAP](https://cwiki.apache.org/confluence/display/Hive/LLAP) (conocido a veces como Larga vida y procesamiento) es una característica nueva de Hive 2.0 que permite el almacenamiento en memoria caché de las consultas. LLAP hace que las consultas de Hive sean mucho más rápidas, hasta [26 más que Hive 1.x en algunos casos](https://hortonworks.com/blog/announcing-apache-hive-2-1-25x-faster-queries-much/).

HDInsight proporciona LLAP en el tipo de clúster Interactive Query. Para más información, vea el documento [Uso de Hive interactivo con HDInsight (versión preliminar)](../interactive-query/apache-interactive-query-get-started.md).

## <a name="scheduling-hive-queries"></a>Programación de consultas de Hive

Hay varios servicios que se pueden usar para ejecutar consultas de Hive como parte de un flujo de trabajo programado o a petición.

### <a name="azure-data-factory"></a>Azure Data Factory

Azure Data Factory permite usar HDInsight como parte de una canalización de Data Factory. Para más información sobre el uso de Hive desde una canalización, consulte el documento [Transformación de datos mediante una actividad de Hive en Azure Data Factory](../../data-factory/transform-data-using-hadoop-hive.md).

### <a name="hive-jobs-and-sql-server-integration-services"></a>Trabajos de Hive y servicios de integración de SQL Server

Puede usar SQL Server Integration Services (SSIS) para ejecutar un trabajo de Hive. El paquete de características de Azure para SSIS proporciona los siguientes componentes que funcionan con trabajos de Hive en HDInsight.

* [Tarea de Hive de Azure HDInsight](https://docs.microsoft.com/sql/integration-services/control-flow/azure-hdinsight-hive-task)

* [Administrador de conexiones de suscripción de Azure](https://docs.microsoft.com/sql/integration-services/connection-manager/azure-subscription-connection-manager)

Para más información, consulte la documentación del [Azure Feature Pack](https://docs.microsoft.com/sql/integration-services/azure-feature-pack-for-integration-services-ssis).

### <a name="apache-oozie"></a>Apache Oozie

Apache Oozie es un sistema de coordinación y flujos de trabajo que administra trabajos de Hadoop. Para más información sobre el uso de Oozie con Hive, consulte el documento [Uso de Apache Oozie para definir y ejecutar un flujo de trabajo](../hdinsight-use-oozie-linux-mac.md).

## <a name="next-steps"></a>Pasos siguientes

Ahora que aprendió qué es Hive y cómo usarlo con Hadoop en HDInsight, use los siguientes vínculos para explorar otras formas de trabajar con HDInsight de Azure.

* [Carga de datos en HDInsight](../hdinsight-upload-data.md)
* [Uso de funciones definidas por el usuario (UDF) de Python con Apache Hive y Apache Pig en HDInsight](./python-udf-hdinsight.md)
* [Uso de trabajos de MapReduce con HDInsight](hdinsight-use-mapreduce.md)
