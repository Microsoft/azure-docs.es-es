---
title: Uso de SparkCruise en Azure HDInsight para acelerar las consultas Apache Spark
description: Obtenga información sobre cómo usar la plataforma de optimización SparkCruise para mejorar la eficacia de las consultas Apache Spark.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 07/27/2020
ms.openlocfilehash: 1a73b4707f83d6a23dffc20d95aa7b8a0fa465b3
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88649064"
---
# <a name="sparkcruise-on-azure-hdinsight"></a>SparkCruise en Azure HDInsight

En este documento se describe la característica *SparkCruise* de Azure HDInsight, que reutiliza automáticamente cálculos de Apache Spark para aumentar la eficacia de las consultas.

## <a name="overview"></a>Información general

Las consultas que se ejecutan en una plataforma de análisis, como Apache Spark, se descomponen en un plan de consulta que contiene subconsultas más pequeñas. Estas subconsultas se pueden mostrar repetidamente en los planes de consulta de varias consultas. Cada vez que aparecen, se vuelven a ejecutar para devolver los resultados. Sin embargo, volver a ejecutar la misma consulta puede ser ineficaz y generar costos de cálculo innecesarios.

*SparkCruise* es una plataforma de optimización de la carga de trabajo que puede reutilizar los cálculos comunes, lo que reduce el tiempo de ejecución general de la consulta y los costos de transferencia de datos. La plataforma usa el concepto de una *vista materializada*, que es una consulta cuyos resultados se almacenan en formato precalculado. Estos resultados se pueden volver a usar cuando la consulta se muestra de nuevo más adelante, en lugar de volverlos a calcular.

## <a name="setup-and-installation"></a>Configuración e instalación

SparkCruise está disponible en todos los clústeres de HDInsight 4.0 con Spark 2.3 o 2.4. Los archivos de la biblioteca de SparkCruise se instalan en el directorio `/opt/peregrine/` del clúster de HDInsight. Para que funcione correctamente, *SparkCruise* requiere las siguientes propiedades de configuración, que se establecen de forma predeterminada.

* `spark.sql.queryExecutionListeners` se establece en `com.microsoft.peregrine.spark.listeners.PlanLogListener`, que habilita el registro de los planes de consulta.
* `spark.sql.extensions` se establece en `com.microsoft.peregrine.spark.extensions.SparkExtensionsHdi`, que habilita las reglas del optimizador para la materialización y la reutilización en línea.

## <a name="computation-reuse-in-spark-sql"></a>Reutilización del cálculo en Spark SQL

En el escenario de ejemplo siguiente se muestra cómo usar *SparkCruise* para optimizar las consultas Apache Spark. 

1. SSH en el nodo principal del clúster de Spark.
1. Escriba `spark-shell`.
1. Ejecute el siguiente fragmento de código, que ejecuta algunas consultas básicas con datos de ejemplo en el clúster.

    ```scala
    spark.sql("select count(*) from hivesampletable").collect
    spark.sql("select count(*) from hivesampletable").collect
    spark.sql("select distinct market from hivesampletable where querytime like '11%'").show
    spark.sql("select distinct state, country from hivesampletable where querytime like '11%'").show
    :quit
    ```
1. Use la herramienta de análisis de consultas *SparkCruise* para analizar los planes de consulta de las consultas anteriores, que están almacenadas en los registros de aplicaciones de Spark. 

    ```
    sudo /opt/peregrine/analyze/peregrine.sh analyze views
    ```

1. Vea la salida del proceso de análisis, que es un archivo de comentarios. Este archivo de comentarios contiene anotaciones para futuras consultas Spark SQL. 

    ```
    sudo /opt/peregrine/analyze/peregrine.sh show
    ```

El comando `analyze` analiza los planes de consulta y crea una representación tabular de la carga de trabajo. Esta tabla de cargas de trabajo se puede consultar mediante el cuaderno *WorkloadInsights* que se incluye en el repositorio [Ejemplos de HDInsight SparkCruise](https://github.com/Azure-Samples/azure-sparkcruise-samples). A continuación, el comando `views` identifica expresiones de subplan comunes y selecciona expresiones de subplan interesantes para su posterior materialización y reutilización. La salida es un archivo de comentarios que contiene anotaciones para futuras consultas Spark SQL. 

El comando `show` muestra una salida parecida al siguiente texto:

```bash
Feedback file -->

1593761760087311271 Materialize /peregrine/views/1593761760087311271
1593761760087311271 Reuse /peregrine/views/1593761760087311271
18446744073621796959 Materialize /peregrine/views/18446744073621796959
18446744073621796959 Reuse /peregrine/views/18446744073621796959
11259615723090744908 Materialize /peregrine/views/11259615723090744908
11259615723090744908 Reuse /peregrine/views/11259615723090744908
9409467400931056980 Materialize /peregrine/views/9409467400931056980
9409467400931056980 Reuse /peregrine/views/9409467400931056980

Materialized subexpressions -->

Found 4 items
-rw-r--r--   1 sshuser sshuser     113445 2020-07-24 16:46 /peregrine/views/logical_ir.csv
-rw-r--r--   1 sshuser sshuser     169458 2020-07-24 16:46 /peregrine/views/physical_ir.csv
-rw-r--r--   1 sshuser sshuser      25730 2020-07-24 16:46 /peregrine/views/views.csv
-rw-r--r--   1 sshuser sshuser        536 2020-07-24 16:46 /peregrine/views/views.stp
```

El archivo de comentarios contiene entradas con el siguiente formato: `subplan-identifier [Materialize|Reuse] input/path/to/action`. En este ejemplo, hay dos firmas únicas: una que representa las dos primeras consultas repetidas y la segunda que representa el predicado de filtro de las dos últimas consultas. Con este archivo de comentarios, cuando se envían las siguientes consultas de nuevo, se materializarán automáticamente y reutilizarán los subplanes comunes. 

Para probar las optimizaciones, ejecute otro conjunto de consultas de ejemplo.

1. Escriba `spark-shell`.
1. Ejecute el siguiente fragmento de código

    ```scala
    spark.sql("select count(*) from hivesampletable").collect
    spark.sql("select count(*) from hivesampletable").collect
    spark.sql("select distinct state, country from hivesampletable where querytime like '12%'").show
    spark.sql("select distinct market from hivesampletable where querytime like '12%'").show
    :quit
    ```

1. Vuelva a consultar el archivo de comentarios para ver las firmas de las consultas que se han reutilizado.

    ```
    sudo /opt/peregrine/analyze/peregrine.sh show
    ```

El comando `show` genera una salida similar al texto siguiente:

```bash
Feedback file -->

1593761760087311271 Materialize /peregrine/views/1593761760087311271
1593761760087311271 Reuse /peregrine/views/1593761760087311271
18446744073621796959 Materialize /peregrine/views/18446744073621796959
18446744073621796959 Reuse /peregrine/views/18446744073621796959
11259615723090744908 Materialize /peregrine/views/11259615723090744908
11259615723090744908 Reuse /peregrine/views/11259615723090744908
9409467400931056980 Materialize /peregrine/views/9409467400931056980
9409467400931056980 Reuse /peregrine/views/9409467400931056980

Materialized subexpressions -->

Found 8 items
drwxr-xr-x   - root root          0 2020-07-24 17:21 /peregrine/views/11259615723090744908
drwxr-xr-x   - root root          0 2020-07-24 17:21 /peregrine/views/1593761760087311271
drwxr-xr-x   - root root          0 2020-07-24 17:22 /peregrine/views/18446744073621796959
drwxr-xr-x   - root root          0 2020-07-24 17:21 /peregrine/views/9409467400931056980
-rw-r--r--   1 root root     113445 2020-07-24 16:46 /peregrine/views/logical_ir.csv
-rw-r--r--   1 root root     169458 2020-07-24 16:46 /peregrine/views/physical_ir.csv
-rw-r--r--   1 root root      25730 2020-07-24 16:46 /peregrine/views/views.csv
-rw-r--r--   1 root root        536 2020-07-24 16:46 /peregrine/views/views.stp

```

Aunque el valor literal de la consulta ha cambiado de `'11%'` a `'12%'`, *SparkCruise* puede hacer coincidir las consultas anteriores con las nuevas con pequeñas variaciones, como la evolución de los valores literales y las versiones de conjuntos de datos. Si hay cambios importantes en una consulta, puede volver a ejecutar la herramienta de análisis para identificar nuevas consultas que se pueden reutilizar.

En segundo plano, *SparkCruise* desencadena una subconsulta para materializar el subplan seleccionado de la primera consulta que lo contiene. Las consultas posteriores pueden leer directamente los subplanes materializados en lugar de recalcularlos. En esta carga de trabajo, los subplanes se materializarán en modo en línea para la primera y la tercera consulta. Podemos ver el cambio de plan de las consultas una vez que se materializan los subplanes comunes.

Puede ver que ahora hay cuatro subexpresiones materializadas nuevas disponibles para reutilizarlas en consultas posteriores.

## <a name="clean-up"></a>Limpieza

Los archivos de comentarios, los subplanes materializados y los registros de consultas se conservan entre sesiones de Spark. Para quitar estos archivos, ejecute el siguiente comando:

```bash
sudo /opt/peregrine/analyze/peregrine.sh clean
```

## <a name="next-steps"></a>Pasos siguientes

* [Uso del cuaderno de conclusiones de la carga de trabajo para determinar las ventajas de SparkCruise](https://github.com/Azure-Samples/azure-sparkcruise-samples/tree/main/SparkCruise)
* [Mejora del rendimiento de las cargas de trabajo de Apache Spark con la memoria caché de E/S de Azure HDInsight](apache-spark-improve-performance-iocache.md)
* [Optimización de trabajos de Apache Spark en HDInsight](./apache-spark-perf.md)
* [SparkCruise: Reutilización del cálculo manos libres en Spark](https://people.cs.umass.edu/~aroy/sparkcruise-vldb19.pdf)
* [Instrucciones de Apache Spark en Azure HDInsight](./spark-best-practices.md)
