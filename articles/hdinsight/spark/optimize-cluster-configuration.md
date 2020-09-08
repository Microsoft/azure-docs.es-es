---
title: 'Optimización de la configuración del clúster de Apache Spark: Azure HDInsight'
description: Aprenda a configurar el clúster de Apache Spark para maximizar el rendimiento en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 08/21/2020
ms.custom: contperfq1
ms.openlocfilehash: 456b955d40e417c9851734b0acaa86a14c9c83f0
ms.sourcegitcommit: afa1411c3fb2084cccc4262860aab4f0b5c994ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/23/2020
ms.locfileid: "88757853"
---
# <a name="cluster-configuration-optimization-for-apache-spark"></a>Optimización de la configuración de clústeres para Apache Spark

En este artículo se describe cómo optimizar la configuración del clúster de Apache Spark para obtener el máximo rendimiento en Azure HDInsight.

## <a name="overview"></a>Información general

En función de la carga de trabajo del clúster de Spark, puede determinar que una configuración de Spark no predeterminada tendría como resultado una ejecución de trabajos de Spark más optimizada.  Realice pruebas comparativas con cargas de trabajo de ejemplo para validar las configuraciones de clúster no predeterminadas.

Estos son algunos parámetros comunes que puede ajustar:

|Parámetro |Descripción |
|---|---|
|--num-executors|Establece el número adecuado de ejecutores.|
|--executor-cores|Establece el número de núcleos para cada ejecutor. Normalmente, debería tener ejecutores de tamaño medio, dado que otros procesos consumen parte de la memoria disponible.|
|--executor-memory|Establece el tamaño de memoria de cada ejecutor, que controla el tamaño del montón en YARN. Deje algo de memoria para la sobrecarga de ejecución.|

## <a name="select-the-correct-executor-size"></a>Selección del tamaño correcto del ejecutor

A la hora de decidir la configuración del ejecutor, tenga en cuenta la sobrecarga de la recolección de elementos no utilizados (GC) de Java.

* Factores para reducir el tamaño del ejecutor:
    1. Reduzca el tamaño del montón a por debajo de 32 GB para mantener la sobrecarga de GC < 10 %.
    2. Reduzca el número de núcleos para mantener la sobrecarga de GC < 10 %.

* Factores para aumentar el tamaño del ejecutor:
    1. Reduzca la sobrecarga de comunicación entre los ejecutores.
    2. Reduzca el número de conexiones abiertas entre los ejecutores (N2) en clústeres más grandes (> 100 ejecutores).
    3. Aumente el tamaño del montón para dar cabida a las tareas que consumen mucha memoria.
    4. Opcional: reduzca la sobrecarga de memoria previa al ejecutor.
    5. Opcional: Aumente la utilización y la simultaneidad mediante la sobresuscripción de CPU.

Como norma general, al seleccionar el tamaño del ejecutor:

1. Comience con 30 GB por ejecutor y distribuya los núcleos de máquina disponibles.
2. Aumente el número de núcleos de ejecutor para clústeres más grandes (> 100 ejecutores).
3. Modifique el tamaño según las ejecuciones de prueba y los factores anteriores, como la sobrecarga de GC.

Al ejecutar consultas simultáneas, tenga en cuenta lo siguiente:

1. Comience con 30 GB por ejecutor y todos los núcleos de la máquina.
2. Cree varias aplicaciones de Spark paralelas mediante la sobresuscripción de CPU (mejora de la latencia en torno al 30 %).
3. Distribuya las consultas entre aplicaciones paralelas.
4. Modifique el tamaño según las ejecuciones de prueba y los factores anteriores, como la sobrecarga de GC.

Para más información sobre el uso de Ambari para configurar ejecutores, consulte [Configuración de los ejecutores de Apache Spark](apache-spark-settings.md#configuring-spark-executors).

Supervise el rendimiento de las consultas en busca de valores atípicos u otros problemas de rendimiento, para ello, consulte la vista de escala de tiempo. También puede consultar el gráfico SQL, las estadísticas de trabajos, etc. Para información sobre la depuración de trabajos de Spark mediante YARN y el servidor de historial de Spark, consulte [Depuración de trabajos de Apache Spark que se ejecutan en Azure HDInsight](apache-spark-job-debugging.md). Puede encontrar sugerencias sobre el uso de un servidor de escala de tiempo de YARN en [Acceso a los registros de aplicaciones de YARN en Apache Hadoop](../hdinsight-hadoop-access-yarn-app-logs-linux.md).

En ocasiones, uno o algunos de los ejecutores son más lentos que otros y las tareas tardan mucho más en ejecutarse. Esta lentitud sucede con frecuencia en clústeres más grandes (>30 nodos). En este caso, divida el trabajo en un número de tareas más grande para que el programador pueda compensar las tareas lentas. Por ejemplo, debe tener al menos dos veces tantas tareas como el número de núcleos de ejecutor en la aplicación. También puede habilitar la ejecución especulativa de tareas con `conf: spark.speculation = true`.

## <a name="next-steps"></a>Pasos siguientes

* [Optimización del procesamiento de datos para Apache Spark](optimize-cluster-configuration.md)
* [Optimización del almacenamiento de datos para Apache Spark](optimize-data-storage.md)
* [Optimización del uso de la memoria para Apache Spark](optimize-memory-usage.md)
