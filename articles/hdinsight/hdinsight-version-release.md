---
title: Introducción a HDInsight 4.0 (Azure)
description: Comparación entre las características, limitaciones y recomendaciones de actualización de HDInsight 3.6 y HDInsight 4.0.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: hrasheed
ms.service: hdinsight
ms.topic: conceptual
ms.date: 08/21/2020
ms.openlocfilehash: 37f0a8d1f70fa96db505973d097febabe99ab7a8
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "88749168"
---
# <a name="azure-hdinsight-40-overview"></a>Introducción a Azure HDInsight 4.0

Azure HDInsight es uno de los servicios más populares entre los clientes de empresa para Apache Hadoop y Apache Spark. HDInsight 4.0 es una distribución de nube de componentes de Apache Hadoop. Este artículo proporciona información sobre la última versión de Azure HDInsight y cómo actualizarla.

## <a name="whats-new-in-hdinsight-40"></a>Novedades de HDInsight 4.0

### <a name="apache-hive-30-and-low-latency-analytical-processing"></a>Apache Hive 3.0 y procesamiento analítico de baja latencia

El procesamiento analítico de baja latencia (LLAP) de Apache Hive usa servidores de consultas persistentes y almacenamiento en memoria caché. Este proceso ofrece resultados rápidos de consultas de SQL realizadas en datos de un almacenamiento en la nube remoto. Este tipo de procesamiento de Hive usa un conjunto de demonios persistentes que ejecutan fragmentos de consultas de Hive. La ejecución de consultas en LLAP es similar a la de Hive sin LLAP, con las tareas de trabajo ejecutándose en los demonios de LLAP en lugar de en contenedores.

Entre las ventajas del procesamiento LLAP de Hive se pueden citar las siguientes:

* Capacidad para realizar análisis SQL profundos sin sacrificar el rendimiento y la capacidad de adaptación. Por ejemplo, combinaciones complejas, subconsultas, funciones de ventanas, ordenación, funciones definidas por el usuario y agregaciones complejas.

* Consultas interactivas en datos en el mismo almacenamiento en el que se preparan los datos, eliminando la necesidad de mover los datos de un almacenamiento a otro motor para el procesamiento analítico.

* Almacenar en caché los resultados de la consulta permite volver a usar los resultados de la consulta calculados anteriormente. Esta memoria caché ahorra tiempo y recursos invertidos en ejecutar las tareas de clúster necesarias para la consulta.

### <a name="hive-dynamic-materialized-views"></a>Vistas materializadas dinámicas de Hive

Hive admite ahora vistas materializadas dinámicas o procesamiento previo de los resúmenes correspondientes. Las vistas aceleran el procesamiento de consultas en los almacenamientos de datos. Las vistas materializadas pueden almacenarse de forma nativa en Hive y pueden utilizar sin problemas la aceleración de LLAP.

### <a name="hive-transactional-tables"></a>Tablas transaccionales de Hive

HDI 4.0 incluye Apache Hive 3. Hive 3 requiere el cumplimiento con la atomicidad, coherencia, aislamiento y durabilidad de las tablas transaccionales que residen en el almacén de Hive. Se puede administrar y acceder a las tablas compatibles con ACID y a los datos de estas mediante Hive. Los datos de las tablas CRUD (crear, recuperar, actualizar y eliminar) deben tener el formato de archivo con columnas y filas optimizadas (ORC). Las tablas de solo inserción admiten todos los formatos de archivos.

* ACID v2 tiene mejoras de rendimiento en el formato de almacenamiento y el motor de ejecución.

* ÁCIDO está habilitado de forma predeterminada para permitir la compatibilidad total de las actualizaciones de datos.

* Las funcionalidades mejoradas de ACID le permiten actualizar y eliminar en el nivel de fila.

* No hay sobrecarga de rendimiento.

* La creación de depósitos no es necesaria.

* Spark puede leer y escribir en las tablas ACID de Hive mediante Hive Warehouse Connector.

Más información sobre [Apache Hive 3](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.0.0/hive-overview/content/hive_whats_new_in_this_release_hive.html).

### <a name="apache-spark"></a>Spark de Apache

Apache Spark obtiene tablas actualizables y transacciones ACID con Hive Warehouse Connector. Hive Warehouse Connector le permite registrar las tablas transaccionales de Hive como tablas externas en Spark para acceder a una funcionalidad transaccional completa. Las versiones anteriores solo admitían la manipulación de particiones de tablas. Hive Warehouse Connector también admite DataFrames de streaming.  Este proceso transmite lecturas y escrituras en las tablas transaccionales y de streaming de Hive desde Spark.

Los ejecutores de Spark se pueden conectar directamente a los demonios de LLAP de Hive para recuperar y actualizar datos de una forma transaccional, lo que permite a Hive mantener el control de los datos.

Apache Spark en HDInsight 4.0 admite los siguientes escenarios:

* Ejecute el aprendizaje del modelo de Machine Learning sobre la misma tabla transaccional usada para los informes.
* Use transacciones ACID para agregar de forma segura columnas del aprendizaje automático de Spark a una tabla de Hive.
* Ejecute un trabajo de streaming de Spark en la fuente de cambios desde una tabla de streaming de Hive.
* Cree archivos ORC directamente desde un trabajo de Spark Structured Streaming.

Ya no tiene que preocuparse por intentar acceder accidentalmente a tablas transaccionales de Hive directamente desde Spark. Esto provocaba resultados incoherentes y datos duplicados o dañados. En HDInsight 4.0, las tablas de Spark y Hive se mantienen en distintas instancias de Metastore. Use Hive Data Warehouse Connector para registrar explícitamente tablas transaccionales de Hive como tablas externas de Spark.

Más información sobre [Apache Spark](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.0.0/spark-overview/content/analyzing_data_with_apache_spark.html).

### <a name="apache-oozie"></a>Apache Oozie

Apache Oozie 4.3.1 se incluye en HDI 4.0 con los cambios siguientes:

* Oozie ya no ejecuta acciones de Hive. La CLI de Hive se ha eliminado y se ha reemplazado por BeeLine.

* Puede excluir las dependencias no deseadas de ShareLib incluyendo un patrón de exclusión en el archivo **job.properties**.

Más información sobre [Apache Oozie](https://docs.hortonworks.com/HDPDocuments/HDP3/HDP-3.0.0/release-notes/content/patch_oozie.html).

## <a name="how-to-upgrade-to-hdinsight-40"></a>Actualización a HDInsight 4.0

Pruebe concienzudamente los componentes antes de implementar la versión más reciente en un entorno de producción. HDInsight 4.0 está disponible para comenzar el proceso de actualización. HDInsight 3.6 es la opción predeterminada para evitar contratiempos accidentales.

No hay ninguna ruta de actualización a HDInsight 4.0 que se admita desde versiones anteriores de HDInsight. Como han cambiado los formatos de metastore y de datos de blob, 4.0 no es compatible con versiones anteriores. Es importante que mantenga su nuevo entorno de HDInsight 4.0 separado del entorno de producción actual. Si implementa HDInsight 4.0 en el entorno actual, metastore se actualizará de forma permanente.  

## <a name="limitations"></a>Limitaciones

* HDInsight 4.0 no admite MapReduce para Apache Hive. En su lugar, use Apache Tez. Más información sobre [Apache Tez](https://tez.apache.org/).
* HDInsight 4.0 no es compatible con Apache Storm.
* HDInsight 4.0 no es compatible con el tipo de clúster de ML Services.
* La vista de Hive solo está disponible en clústeres de HDInsight 4.0 con un número de versión igual o superior a 4.1. Este número de versión está disponible en Ambari Admin -> Versions (Administración de Ambari > Versiones).
* No se admite el intérprete de shell de Apache Zeppelin en clústeres de Spark e Interactive Query.
* No puede *deshabilitar* LLAP en un clúster LLAP de Spark. Solo puede desactivarlo.
* Azure Data Lake Storage Gen2 no puede guardar cuadernos de Jupyter en un clúster de Spark.
* Apache Pig se ejecuta en Tez de forma predeterminada, pero puede cambiarlo a MapReduce.
* La integración con Spark SQL Ranger para la seguridad de filas y columnas está en desuso
* Spark 2.4 y Kafka 2.1 están disponibles en HDInsight 4.0, por lo que Spark 2.3 y Kafka 1.1 ya no se admiten. Se recomienda usar Spark 2.4 y Kafka 2.1 y versiones posteriores en HDInsight 4.0.

## <a name="next-steps"></a>Pasos siguientes

* [Documentación de HDInsight de Azure](index.yml)
* [Notas de la versión](hdinsight-release-notes.md)
