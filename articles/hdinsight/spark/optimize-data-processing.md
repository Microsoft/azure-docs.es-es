---
title: 'Optimización del procesamiento de datos para Apache Spark: Azure HDInsight'
description: Obtenga información sobre cómo elegir las operaciones más eficaces para procesar los datos en Apache Spark con Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/20/2020
ms.custom: contperfq1
ms.openlocfilehash: 3ed4f8d4d8ca0a68a4ccf01a38ae5f8e66cc26df
ms.sourcegitcommit: afa1411c3fb2084cccc4262860aab4f0b5c994ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/23/2020
ms.locfileid: "88757836"
---
# <a name="data-processing-optimization-for-apache-spark"></a>Optimización del procesamiento de datos para Apache Spark

En este artículo se describe cómo optimizar la configuración del clúster de Apache Spark para obtener el máximo rendimiento en Azure HDInsight.

## <a name="overview"></a>Información general

Si tiene trabajos lentos en una combinación o un orden aleatorio, es probable que la causa sea la *asimetría de datos*. La asimetría de datos es la asimetría en los datos del trabajo. Por ejemplo, un trabajo de asignación puede tardar 20 segundos. Por ejecutar un trabajo donde los datos se combinan o se ordenan de forma aleatoria puede tardar horas. Para corregir la asimetría de los datos, debe usar el valor de salt en toda la clave o un *valor de salt aislado* únicamente para algunos subconjuntos de claves. Si usa un valor de salt aislado, debe filtrar adicionalmente para aislar el subconjunto de claves con el valor de salt en las combinaciones de asignación. Otra opción consiste en insertar una columna de depósito y primero agregarla en los depósitos.

Otro factor causante de las combinaciones lentas podría ser el tipo de combinación. De forma predeterminada, Spark usa el tipo de combinación `SortMerge`. Este tipo de combinación es más adecuada para conjuntos de datos grandes, pero tiene la desventaja de que es cara desde el punto de vista computacional, porque primero debe ordenar los lados izquierdo y derecho de los datos antes de combinarlos.

Una combinación `Broadcast` es más adecuada para conjuntos de datos más pequeños o donde un lado de la combinación sea mucho más pequeño que el otro. Este tipo de combinación difunde un lado a todos los ejecutores y, por tanto, requiere en general más memoria para las difusiones.

Se puede cambiar el tipo de combinación en la configuración si establece `spark.sql.autoBroadcastJoinThreshold`, o puede establecer una sugerencia de combinación con las API de DataFrame (`dataframe.join(broadcast(df2))`).

```scala
// Option 1
spark.conf.set("spark.sql.autoBroadcastJoinThreshold", 1*1024*1024*1024)

// Option 2
val df1 = spark.table("FactTableA")
val df2 = spark.table("dimMP")
df1.join(broadcast(df2), Seq("PK")).
    createOrReplaceTempView("V_JOIN")

sql("SELECT col1, col2 FROM V_JOIN")
```

Si usa tablas en depósitos, tiene un tercer tipo de combinación, la combinación `Merge`. Un conjunto de datos particionado previamente y ordenado previamente de forma correcta omitirá la fase cara de ordenación de una combinación `SortMerge`.

El orden de las combinaciones importa, en especial en consultas más complejas. Comience con las combinaciones más selectivas. Además, mueva las combinaciones que aumentan el número de filas después de las agregaciones cuando sea posible.

Para administrar el paralelismo en el caso de las combinaciones cartesianas, puede agregar estructuras anidadas, ventanas y, quizás, omitir uno o varios pasos del trabajo de Spark.

## <a name="optimize-job-execution"></a>Optimización de la ejecución de trabajos

* Almacene en caché como sea necesario; por ejemplo, si usa los datos dos veces, almacénelos en caché.
* Difunda variables a todos los ejecutores. Las variables se serializan una sola vez, lo que da lugar a búsquedas más rápidas.
* Use el grupo de subprocesos en el controlador, lo que tendrá como resultado operaciones más rápidas para muchas tareas.

Supervise los trabajos en ejecución con regularidad en busca de problemas de rendimiento. Si necesita más información sobre determinados problemas, podría usar una de las siguientes herramientas de generación de perfiles de rendimiento:

* [Intel PAL Tool](https://github.com/intel-hadoop/PAT) supervisa el uso de CPU, de almacenamiento y de ancho de banda de red.
* [Oracle Java 8 Mission Control](https://www.oracle.com/technetwork/java/javaseproducts/mission-control/java-mission-control-1998576.html) crea perfiles del código de Spark y del ejecutor.

Esencial para el rendimiento de consultas de Spark 2.x es el motor de Tungsten, que depende de la generación de código whole stage. En algunos casos, se puede deshabilitar la generación de código whole stage. Por ejemplo, si se usa un tipo no mutable (`string`) en la expresión de agregación, aparece `SortAggregate` en lugar de `HashAggregate`. Por ejemplo, para mejorar el rendimiento, pruebe lo siguiente y, a continuación, habilite de nuevo la generación de código:

```sql
MAX(AMOUNT) -> MAX(cast(AMOUNT as DOUBLE))
```

## <a name="next-steps"></a>Pasos siguientes

* [Optimización del almacenamiento de datos para Apache Spark](optimize-data-storage.md)
* [Optimización del uso de la memoria para Apache Spark](optimize-memory-usage.md)
* [Optimización de la configuración de clústeres para Apache Spark](optimize-cluster-configuration.md)
