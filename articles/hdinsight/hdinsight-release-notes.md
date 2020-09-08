---
title: Notas de la versión de Azure HDInsight.
description: Notas más recientes de la versión de Azure HDInsight. Obtenga sugerencias de desarrollo y detalles sobre Hadoop, Spark, R Server, Hive, etc.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.custom: hdinsightactive
ms.service: hdinsight
ms.topic: conceptual
ms.date: 08/25/2020
ms.openlocfilehash: 0587a179b98a410cdba46b7817d86567f275f25d
ms.sourcegitcommit: ac7ae29773faaa6b1f7836868565517cd48561b2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88826827"
---
# <a name="azure-hdinsight-release-notes"></a>Notas de la versión de Azure HDInsight

Este artículo proporciona información sobre las **últimas** actualizaciones de versión de Azure HDInsight. Para más información sobre versiones anteriores, vea [Notas de la versión (en archivo) de los componentes de Hadoop en Azure HDInsight](hdinsight-release-notes-archive.md).

## <a name="summary"></a>Resumen

Azure HDInsight es uno de los servicios más populares entre los clientes de empresa para el análisis de código abierto en Azure.

## <a name="release-date-08092020"></a>Fecha de lanzamiento: 09/08/2020

Esta versión se aplica solo a HDInsight 4.0. La versión de HDInsight se pone a disposición de todas las regiones durante varios días. Esta fecha de lanzamiento indica la fecha de lanzamiento de la primera región. Si no ve los cambios siguientes, espere unos días a que la versión se active en su región.

## <a name="new-features"></a>Nuevas características
### <a name="support-for-sparkcruise"></a>Soporte técnico de SparkCruise
SparkCruise es un sistema de reutilización automática de cálculos para Spark. Selecciona subexpresiones comunes que se puedan materializar en función de la carga de trabajo de consultas pasadas. SparkCruise materializa estas subexpresiones como parte del procesamiento de consultas, y la reutilización de cálculos se aplica automáticamente en segundo plano. Puede beneficiarse de SparkCruise sin tener que modificar el código de Spark.
 
### <a name="support-hive-view-for-hdinsight-40"></a>Soporte técnico de la vista de Hive para HDInsight 4.0
La vista de Hive de Apache Ambari está diseñada para ayudarle a crear, optimizar y ejecutar consultas de Hive desde el explorador web. La vista de Hive se admite de forma nativa en los clústeres de HDInsight 4.0 a partir de esta versión. No se aplica a los clústeres ya existentes. Para obtener la vista de Hive integrada, debe quitar y volver a crear el clúster.
 
### <a name="support-tez-view-for-hdinsight-40"></a>Soporte técnico de la vista de Tez para HDInsight 4.0
La vista Apache Tez se usa para realizar un seguimiento de la ejecución del trabajo de Hive Tez y depurarlo. La vista de Tez se admite de forma nativa para HDInsight 4.0 a partir de esta versión. No se aplica a los clústeres ya existentes. Para obtener la vista de Tez integrada, debe colocar y volver a crear el clúster.

## <a name="deprecation"></a>Desuso
### <a name="deprecation-of-spark-21-and-22-in-hdinsight-36-spark-cluster"></a>Desuso de Spark 2.1 y 2.2 para el clúster de Spark de HDInsight 3.6
A partir del 1 de julio de 2020, los clientes no podrán crear nuevos clústeres de Spark con Spark 2.1 y 2.2 en HDInsight 3.6. Los clústeres existentes se ejecutarán tal cual sin la compatibilidad de Microsoft. Considere la posibilidad de pasar a Spark 2.3 en HDInight 3.6 a partir del 30 de junio de 2020 para evitar la posible interrupción del sistema o del soporte técnico.
 
### <a name="deprecation-of-spark-23-in-hdinsight-40-spark-cluster"></a>Desuso de Spark 2.3 y 4.0 para el clúster de Spark de HDInsight 4.0
A partir del 1 de julio de 2020, los clientes no podrán crear nuevos clústeres de Spark con Spark 2.3 en HDInsight 4.0. Los clústeres existentes se ejecutarán tal cual sin la compatibilidad de Microsoft. Considere la posibilidad de pasar a Spark 2.4 en HDInight 4.0 a partir del 30 de junio de 2020 para evitar la posible interrupción del sistema o del soporte técnico.
 
### <a name="deprecation-of-kafka-11-in-hdinsight-40-kafka-cluster"></a>Desuso de Kafka 1.1 en el clúster de Kafka de HDInsight 4.0
A partir del 1 de julio de 2020, los clientes no podrán crear nuevos clústeres de Kafka con Kafka 1.1 en HDInsight 4.0. Los clústeres existentes se ejecutarán tal cual sin la compatibilidad de Microsoft. Considere la posibilidad de pasar a Kafka 2.1 en HDInight 4.0 a partir del 30 de junio de 2020 para evitar la posible interrupción del sistema o del soporte técnico.

## <a name="behavior-changes"></a>Cambios de comportamiento
### <a name="ambari-stack-version-change"></a>Cambio de versión de la pila de Ambari
En esta versión, la versión de Ambari se cambia de 2.x.x.x a 4.1. Puede obtener la versión de Ambari en la opción "Acerca de" de la interfaz de usuario de Ambari.

## <a name="upcoming-changes"></a>Próximos cambios
No habrá cambios importantes que requieran su atención.

## <a name="bug-fixes"></a>Corrección de errores
HDInsight continúa realizando mejoras en la confiabilidad y el rendimiento del clúster. 

Las instancias de JIRA siguientes se trasladan a Hive:
* [HIVE-23619](https://issues.apache.org/jira/browse/HIVE-23619)
* [HIVE-21223](https://issues.apache.org/jira/browse/HIVE-21223)
* [HIVE-22599](https://issues.apache.org/jira/browse/HIVE-22599)
* [HIVE-22121](https://issues.apache.org/jira/browse/HIVE-22121)
* [HIVE-22136](https://issues.apache.org/jira/browse/HIVE-22136)
* [HIVE-18786](https://issues.apache.org/jira/browse/HIVE-18786)

Las siguientes instancias de JIRA se llevan a HBase:
* [HBASE-21458](https://issues.apache.org/jira/browse/HBASE-21458)
* [HBASE-24208](https://issues.apache.org/jira/browse/HBASE-24208)
* [HBASE-24205](https://issues.apache.org/jira/browse/HBASE-24205)

## <a name="component-version-change"></a>Cambio de versión de componentes
No hay cambio de versión de componentes para esta versión. En [este documento](https://docs.microsoft.com/azure/hdinsight/hdinsight-component-versioning#apache-hadoop-components-available-with-different-hdinsight-versions) puede encontrar las versiones actuales de los componentes para HDInsight 4.0 y HDInsight 3.6.

## <a name="known-issues"></a>Problemas conocidos

Se ha corregido un problema de Azure Portal, en el que los usuarios experimentaban un error cuando creaban un clúster de Azure HDInsight mediante un tipo de autenticación de SSH de la clave pública. Cuando los usuarios hacían clic en **Revisar y crear**, recibían el error "No debe contener tres caracteres consecutivos del nombre de usuario de SSH". Este problema se ha corregido, pero es posible que deba actualizar la memoria caché del explorador; para ello, presione Ctrl + F5 para cargar la vista corregida. La solución a este problema consistió en crear un clúster con una plantilla de Resource Manager. 
