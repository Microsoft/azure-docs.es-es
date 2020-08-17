---
title: Migración de las cargas de trabajo de Apache Spark 2.1 o 2.2 a 2.3 o 2.4 (Azure HDInsight)
description: Obtenga información sobre cómo migrar Apache Spark 2.1 y 2.2 a 2.3 o 2.4.
author: ashishthaps1
ms.author: ashishth
ms.reviewer: hrasheed
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/20/2020
ms.openlocfilehash: 681de5d8e4daeaabb865cde3ef93008b2965824e
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86497155"
---
# <a name="migrate-apache-spark-21-and-22-workloads-to-23-and-24"></a>Migración de las cargas de trabajo de Apache Spark 2.1 o 2.2 a 2.3 o 2.4

En este documento se explica cómo migrar las cargas de trabajo de Apache Spark en Spark 2.1 y 2.2 a 2.3 o 2.4.

Tal y como se describe en las [notas de la versión](../hdinsight-release-notes.md#upcoming-changes), a partir del 1 de julio de 2020, no se admitirán las siguientes configuraciones de clúster y los clientes no podrán crear nuevos clústeres con estas configuraciones:
 - Spark 2.1 y 2.2 en un clúster de Spark de HDInsight 3.6
 - Spark 2.3 en un clúster de Spark de HDInsight 4.0

Los clústeres de estas configuraciones se ejecutarán tal cual sin la compatibilidad de Microsoft. Si usa Spark 2.1 o 2.2 en HDInsight 3.6, pase a Spark 2.3 en HDInsight 3.6 a partir del 30 de junio de 2020 para evitar una posible interrupción del sistema o del soporte técnico. Si usa Spark 2.3 en un clúster de HDInsight 4.0, pase a Spark 2.4 en HDInsight 4.0 a partir del 30 de junio de 2020 para evitar una posible interrupción del sistema o del soporte técnico.

Para obtener información general sobre la migración de un clúster de HDInsight de 3.6 a 4.0, consulte [Migración del clúster de HDInsight a una versión más reciente](../hdinsight-upgrade-cluster.md). Para obtener información general sobre cómo realizar la migración a una versión más reciente de Apache Spark, consulte [Apache Spark: directiva de control de versiones](https://spark.apache.org/versioning-policy.html).

## <a name="guidance-on-spark-version-upgrades-on-hdinsight"></a>Instrucciones para las actualizaciones de la versión de Spark en HDInsight

| Escenario de actualización | Mechanism | Aspectos que se deben tener en cuenta: | Integración de Spark/Hive |
|------------------|-----------|--------------------|------------------------|
|De HDInsight 3.6 Spark 2.1 a HDInsight 3.6 Spark 2.3| Nueva creación de clústeres con HDInsight Spark 2.3 | Consulte los artículos siguientes: <br> [Apache Spark: actualización de Spark SQL 2.2 a 2.3](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-22-to-23) <br><br> [Apache Spark: actualización de Spark SQL 2.1 a 2.2](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-21-to-22) | Sin cambios |
|De HDInsight 3.6 Spark 2.2 a HDInsight 3.6 Spark 2.3 | Nueva creación de clústeres con HDInsight Spark 2.3 | Consulte los artículos siguientes: <br> [Apache Spark: actualización de Spark SQL 2.2 a 2.3](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-22-to-23) | Sin cambios |
| De HDInsight 3.6 Spark 2.1 a HDInsight 4.0 Spark 2.4 | Nueva creación de clústeres con HDInsight 4.0 Spark 2.4 | Consulte los artículos siguientes: <br> [Apache Spark: actualización de Spark SQL 2.3 a 2.4](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-23-to-24) <br><br> [Apache Spark: actualización de Spark SQL 2.2 a 2.3](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-22-to-23) <br><br> [Apache Spark: actualización de Spark SQL 2.1 a 2.2](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-21-to-22) | La integración de Spark y Hive ha cambiado en HDInsight 4.0. <br><br> En HDInsight 4,0, Spark y Hive usan catálogos independientes para acceder a tablas de SparkSQL o Hive. Una tabla creada por Spark se hospeda en el catálogo de Spark. Una tabla creada por Hive se hospeda en el catálogo de Hive. Este comportamiento es diferente a HDInsight 3.6, donde Hive y Spark compartían el catálogo común. La integración de Hive y Spark en HDInsight 4.0 se basa en el conector de Hive Warehouse (HWC). HWC funciona como un puente entre Spark y Hive. Obtenga más información sobre Hive Warehouse Connector. <br> En HDInsight 4.0, puede compartir el metastore entre Hive y Spark cambiando la propiedad metastore.catalog.default a Hive en el clúster de Spark, si así lo desea. Esta propiedad se encuentra en spark2-hive-site-override de las opciones avanzadas de Ambari. Es importante comprender que el uso compartido del metastore solo funciona para las tablas de Hive externas, de modo que no funcionará si tiene tablas de Hive o ACID administradas o internas. <br><br>Consulte [Migración de cargas de trabajo de Hive de Azure HDInsight 3.6 a HDInsight 4.0](../interactive-query/apache-hive-migrate-workloads.md) para obtener más información.<br><br> |
| De HDInsight 3.6 Spark 2.2 a HDInsight 4.0 Spark 2.4 | Nueva creación de clústeres con HDInsight 4.0 Spark 2.4 | Consulte los artículos siguientes: <br> [Apache Spark: actualización de Spark SQL 2.3 a 2.4](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-23-to-24) <br><br> [Apache Spark: actualización de Spark SQL 2.2 a 2.3](https://spark.apache.org/docs/latest/sql-migration-guide.html#upgrading-from-spark-sql-22-to-23) | La integración de Spark y Hive ha cambiado en HDInsight 4.0. <br><br> En HDInsight 4,0, Spark y Hive usan catálogos independientes para acceder a tablas de SparkSQL o Hive. Una tabla creada por Spark se hospeda en el catálogo de Spark. Una tabla creada por Hive se hospeda en el catálogo de Hive. Este comportamiento es diferente a HDInsight 3.6, donde Hive y Spark compartían el catálogo común. La integración de Hive y Spark en HDInsight 4.0 se basa en el conector de Hive Warehouse (HWC). HWC funciona como un puente entre Spark y Hive. Obtenga más información sobre Hive Warehouse Connector. <br> En HDInsight 4.0, puede compartir el metastore entre Hive y Spark cambiando la propiedad metastore.catalog.default a Hive en el clúster de Spark, si así lo desea. Esta propiedad se encuentra en spark2-hive-site-override de las opciones avanzadas de Ambari. Es importante comprender que el uso compartido del metastore solo funciona para las tablas de Hive externas, de modo que no funcionará si tiene tablas de Hive o ACID administradas o internas. <br><br>Consulte [Migración de cargas de trabajo de Hive de Azure HDInsight 3.6 a HDInsight 4.0](../interactive-query/apache-hive-migrate-workloads.md) para obtener más información.|

## <a name="next-steps"></a>Pasos siguientes

* [Migración del clúster de HDInsight a una versión más reciente](../hdinsight-upgrade-cluster.md)
* [Migración de cargas de trabajo de Hive de Azure HDInsight 3.6 a HDInsight 4.0](../interactive-query/apache-hive-migrate-workloads.md)
