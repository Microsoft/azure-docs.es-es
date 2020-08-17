---
title: Uso de Apache Pig
titleSuffix: Azure HDInsight
description: Aprenda a usar Pig con Apache Hadoop en HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive
ms.topic: how-to
ms.date: 01/28/2020
ms.openlocfilehash: 8fd42169ab6ccb144215e1b6ad2ece09ff55c07f
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87072841"
---
# <a name="use-apache-pig-with-apache-hadoop-on-hdinsight"></a>Uso de Pig con Apache Hadoop en HDInsight

Aprenda a usar [Apache Pig](https://pig.apache.org/) con HDInsight.

Apache Pig es una plataforma para crear programas de Apache Hadoop mediante un lenguaje de procedimientos que se conoce como *Pig Latin*. Pig es una alternativa a Java para crear soluciones *MapReduce* y se incluye con HDInsight de Azure. Utilice la tabla siguiente para conocer las distintas formas de usar Pig con HDInsight:

## <a name="why-use-apache-pig"></a><a id="why"></a>Motivos para usar Apache Pig

Uno de los desafíos del procesamiento de datos mediante el uso de MapReduce en Hadoop es la implementación de la lógica de procesamiento únicamente con un mapa y una función de reducción. Para el procesamiento complejo, a menudo debe interrumpir el procesamiento en varias operaciones de MapReduce que están encadenadas para lograr el resultado deseado.

Pig permite definir los procesos como una serie de transformaciones por la que fluyen los datos para producir el resultado deseado.

El lenguaje de Pig Latin le permite describir el flujo de datos desde entrada sin formato, a través de una o varias transformaciones, para producir el resultado deseado. Los programas de Pig Latin siguen este patrón general:

* **Carga**: Lectura de los datos que se van a manipular desde el sistema de archivos.

* **Transformación**: Manipulación de los datos.

* **Volcado o almacén**: Generación de los datos en la pantalla o almacenamiento para su posterior procesamiento.

### <a name="user-defined-functions"></a>Funciones definidas por el usuario

Pig Latin también admite las funciones definidas por el usuario (UDF), que le permiten invocar componentes externos que implementan la lógica que es difícil de modelar en Pig Latin.

Para más información sobre Pig Latin, consulte el [manual de referencia de Pig Latin 1](https://archive.cloudera.com/cdh/3/pig/piglatin_ref1.html) y el [manual de referencia de Pig Latin 2](https://archive.cloudera.com/cdh/3/pig/piglatin_ref2.html).

## <a name="example-data"></a><a id="data"></a>Datos de ejemplo

HDInsight proporciona diversos conjuntos de datos de ejemplo que se almacenan en los directorios `/example/data` y `/HdiSamples`. Estos directorios están en el almacenamiento predeterminado de su clúster. El ejemplo de Pig de este documento utiliza el archivo *log4j* de `/example/data/sample.log`.

Cada registro del archivo consta de una línea de campos que contiene un campo `[LOG LEVEL]` para mostrar el tipo y la gravedad, por ejemplo:

```output
2012-02-03 20:26:41 SampleClass3 [ERROR] verbose detail for id 1527353937
```

En el ejemplo anterior, el nivel de registro es ERROR.

> [!NOTE]  
> También puede generar un archivo log4j con la herramienta de registro [Apache Log4j](https://en.wikipedia.org/wiki/Log4j) y luego cargarlo en el blob. Consulte [Carga de datos en HDInsight](hdinsight-upload-data.md) para obtener instrucciones. Para obtener más información sobre el uso de los blobs en Azure Storage con HDInsight, consulte [Uso de Azure Blob Storage con HDInsight](hdinsight-hadoop-use-blob-storage.md).

## <a name="example-job"></a><a id="job"></a>Trabajo de ejemplo

El siguiente trabajo de Pig Latin carga el archivo `sample.log` desde el almacenamiento predeterminado para el clúster de HDInsight. A continuación, realiza una serie de transformaciones que generan un recuento de cuántas veces se ha producido cada nivel de registro en los datos de entrada. Los resultados se escriben en STDOUT.

```output
LOGS = LOAD 'wasb:///example/data/sample.log';
LEVELS = foreach LOGS generate REGEX_EXTRACT($0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;
FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;
GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;
FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;
RESULT = order FREQUENCIES by COUNT desc;
DUMP RESULT;
```

En la siguiente imagen se muestra un resumen de lo que hace cada transformación a los datos.

![Representación gráfica de las transformaciones][image-hdi-pig-data-transformation]

## <a name="run-the-pig-latin-job"></a><a id="run"></a>Ejecutar el trabajo de Pig Latin

HDInsight puede ejecutar trabajos de Pig Latin mediante una variedad de métodos. Use la tabla siguiente para decidir cuál es el método adecuado para usted luego siga el vínculo para ver un tutorial.

## <a name="pig-and-sql-server-integration-services"></a>Pig y SQL Server Integration Services

Puede usar SQL Server Integration Services (SSIS) para ejecutar un trabajo de Pig. El paquete de características de Azure para SSIS proporciona los siguientes componentes que funcionan con trabajos de Pig en HDInsight.

* [Tarea de Pig de Azure HDInsight][pigtask]

* [Administrador de conexiones de suscripción de Azure][connectionmanager]

Obtenga más información sobre el paquete de características de Azure para SSIS [aquí][ssispack].

## <a name="next-steps"></a><a id="nextsteps"></a>Pasos siguientes

Ahora que aprendió a usar Pig con HDInsight, use los siguientes vínculos para explorar otras formas de trabajar con HDInsight de Azure.

* [Carga de datos en HDInsight](hdinsight-upload-data.md)
* [Uso de Apache Hive con HDInsight](./hadoop/hdinsight-use-hive.md)
* [Uso de Apache Sqoop con HDInsight](hdinsight-use-sqoop.md)
* [Uso de trabajos de MapReduce con HDInsight](./hadoop/hdinsight-use-mapreduce.md)

[apachepig-home]: https://pig.apache.org/
[putty]: https://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
[curl]: https://curl.haxx.se/
[pigtask]: https://msdn.microsoft.com/library/mt146781(v=sql.120).aspx
[connectionmanager]: https://msdn.microsoft.com/library/mt146773(v=sql.120).aspx
[ssispack]: https://msdn.microsoft.com/library/mt146770(v=sql.120).aspx
[hdinsight-admin-powershell]: hdinsight-administer-use-powershell.md

[hdinsight-use-hive]:../hdinsight-use-hive.md

[hdinsight-provision]: hdinsight-hadoop-provision-linux-clusters.md
[hdinsight-submit-jobs]:submit-apache-hadoop-jobs-programmatically.md#mapreduce-sdk

[Powershell-install-configure]: /powershell/azure/

[powershell-start]: https://technet.microsoft.com/library/hh847889.aspx


[image-hdi-pig-data-transformation]: ./media/use-pig/hdi-data-transformation.gif
