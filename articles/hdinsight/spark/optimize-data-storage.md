---
title: 'Optimización del almacenamiento de datos para Apache Spark: Azure HDInsight'
description: Aprenda a optimizar el almacenamiento de datos para su uso con Apache Spark en Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 05/20/2020
ms.custom: contperfq1
ms.openlocfilehash: 092757728e791f60616d9dceca43e109e7f0019e
ms.sourcegitcommit: afa1411c3fb2084cccc4262860aab4f0b5c994ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/23/2020
ms.locfileid: "88757819"
---
# <a name="data-storage-optimization-for-apache-spark"></a>Optimización del almacenamiento de datos para Apache Spark

En este artículo se describen las estrategias para optimizar el almacenamiento de datos para la ejecución eficaz de trabajos de Apache Spark en Azure HDInsight.

## <a name="overview"></a>Información general

Spark admite muchos formatos, como csv, json, xml, parquet, orc y avro. Spark se puede ampliar para la compatibilidad con muchos más formatos con orígenes de datos externos; para más información, consulte los [paquetes de Apache Spark](https://spark-packages.org).

El mejor formato para el rendimiento es parquet con *compresión eficiente*, que es el valor predeterminado en Spark 2.x. Parquet almacena datos en formato de columnas y está muy optimizado en Spark.

## <a name="choose-data-abstraction"></a>Elección de la abstracción de datos

Las versiones anteriores de Spark usan RDD para abstraer datos; Spark 1.3 y 1.6 introdujeron DataFrames y DataSets, respectivamente. Tenga en cuenta las siguientes ventajas relativas:

* **DataFrames**
    * La mejor opción en la mayoría de los casos.
    * Proporciona optimización de consultas gracias a Catalyst.
    * Generación de código en tiempo de ejecución.
    * Acceso directo a memoria.
    * Baja sobrecarga de recolección de elementos no utilizados (GC).
    * No es tan sencillo para los desarrolladores como DataSets, dado que no hay comprobaciones en tiempo de compilación ni programación de objetos de dominio.
* **DataSets**
    * Adecuado en canalizaciones ETL complejas en las que el impacto sobre el rendimiento es aceptable.
    * No adecuado en agregaciones donde el impacto sobre el rendimiento puede ser considerable.
    * Proporciona optimización de consultas gracias a Catalyst.
    * Es sencillo de usar para los programadores al proporcionar programación de objetos de dominio y comprobaciones de tiempo de compilación.
    * Agrega sobrecarga de serialización/deserialización.
    * Gran sobrecarga de GC.
    * Interrumpe la generación de código en tiempo de ejecución.
* **RDD**
    * No es necesario usar RDD, a menos que deba crear un nuevo RDD personalizado.
    * Sin optimización de consultas mediante Catalyst.
    * Sin generación de código en tiempo de ejecución.
    * Gran sobrecarga de GC.
    * Se deben usar las API heredadas de Spark 1.x.

## <a name="select-default-storage"></a>Selección del almacenamiento predeterminado

Cuando se crea un nuevo clúster de Spark, puede seleccionar Azure Blob Storage o Azure Data Lake Storage como almacenamiento predeterminado del clúster. Ambas opciones ofrecen la ventaja de almacenamiento a largo plazo para clústeres transitorios. Por lo tanto, los datos no se eliminan automáticamente cuando elimina el clúster. Puede volver a crear un clúster transitorio y seguir teniendo acceso a los datos.

| Tipo de almacén | Sistema de archivos | Velocidad | Transitorio | Casos de uso |
| --- | --- | --- | --- | --- |
| Azure Blob Storage | **wasb:** //url/ | **Estándar** | Sí | Clúster transitorio |
| Azure Blob Storage (seguro) | **wasbs:** //url/ | **Estándar** | Sí | Clúster transitorio |
| Azure Data Lake Storage Gen 2| **abfs:** //url/ | **Más rápido** | Sí | Clúster transitorio |
| Azure Data Lake Storage Gen 1| **adl:** //url/ | **Más rápido** | Sí | Clúster transitorio |
| HDFS local | **hdfs:** //url/ | **El más rápido** | No | Clúster 24/7 interactivo |

Para encontrar una descripción completa de las opciones de almacenamiento, consulte [Comparación de opciones de almacenamiento para los clústeres de Azure HDInsight](../hdinsight-hadoop-compare-storage-options.md).

## <a name="use-the-cache"></a>Uso de la caché

Spark proporciona sus propios mecanismos nativos de almacenamiento, que se pueden usar a través de diferentes métodos, como `.persist()`, `.cache()` y `CACHE TABLE`. Este almacenamiento en caché nativo es efectivo con pequeños conjuntos de datos y en canalizaciones ETL donde tenga que almacenar en caché los resultados intermedios. Sin embargo, el almacenamiento en caché nativo de Spark no funciona bien actualmente con la creación de particiones, dado que la tabla almacenada en caché no conserva los datos de la creación de particiones. Una técnica de almacenamiento en caché más genérica y confiable es el *almacenamiento en caché de capas de almacenamiento*.

* Almacenamiento en caché nativo de Spark (no se recomienda)
    * Adecuado para conjuntos de datos pequeños
    * No funciona con la creación de particiones; sin embargo, esto puede cambiar en futuras versiones de Spark.

* Almacenamiento en caché de capas de almacenamiento (recomendado)
    * Se puede implementar en HDInsight mediante la característica [Caché de E/S](apache-spark-improve-performance-iocache.md).
    * Se usa en el almacenamiento en caché SSD y en memoria

* HDFS local (recomendado)
    * Ruta de acceso de `hdfs://mycluster`
    * Usa el almacenamiento en caché SSD
    * Los datos en caché se pierden si se elimina el clúster, por lo que será necesario volver a generar la caché

## <a name="optimize-data-serialization"></a>Optimización de la serialización de datos

Los trabajos de Spark son distribuidos, así que es importante la serialización adecuada de los datos para obtener el mejor rendimiento.  Hay dos opciones de serialización para Spark:

* La serialización Java es la predeterminada.
* La serialización `Kryo` es un formato más reciente y puede dar como resultado una serialización más rápida y compacta que la de Java.  `Kryo` requiere que se registren las clases en el programa y aún no admite todos los tipos serializables.

## <a name="use-bucketing"></a>Uso de la creación de depósitos

La creación de cubos es similar a la creación de particiones de datos. Pero cada cubo puede contener un conjunto de valores de columna en lugar de solo uno. Este método funciona bien cuando se crean particiones en números grandes de valores (millones o más), como es el caso de los identificadores de productos. Para determinar un depósito, se usa hash en la clave de depósito de la fila. Las tablas en depósitos ofrecen optimizaciones únicas porque almacenan metadatos sobre cómo se incluyeron en depósitos y se ordenaron.

Algunas características avanzadas de creación de depósitos son:

* Optimización de consultas basada en la creación de depósitos de información de metadatos.
* Agregaciones optimizadas.
* Combinaciones optimizadas.

Puede usar creación de particiones y creación de depósitos al mismo tiempo.

## <a name="next-steps"></a>Pasos siguientes

* [Optimización del procesamiento de datos para Apache Spark](optimize-cluster-configuration.md)
* [Optimización del uso de la memoria para Apache Spark](optimize-memory-usage.md)
* [Optimización de la configuración de clústeres para Apache Spark](optimize-cluster-configuration.md)
