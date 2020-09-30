---
title: Arquitectura de Apache Hadoop en Azure HDInsight
description: Describe el procesamiento y almacenamiento de Apache Hadoop en clústeres de Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.custom: hdinsightactive
ms.date: 02/07/2020
ms.openlocfilehash: cbd1303417f008da476356a274dde30d7d02d36f
ms.sourcegitcommit: 59ea8436d7f23bee75e04a84ee6ec24702fb2e61
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89505512"
---
# <a name="apache-hadoop-architecture-in-hdinsight"></a>Arquitectura de Apache Hadoop en HDInsight

[Apache Hadoop](https://hadoop.apache.org/) incluye dos componentes principales: el [Sistema de archivos distribuido de Apache Hadoop (HDFS)](https://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html), que proporciona almacenamiento, y [Apache Hadoop Yet Another Resource Negotiator (YARN)](https://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html), que proporciona procesamiento. Con las capacidades de almacenamiento y procesamiento, un clúster puede ejecutar programas [MapReduce](https://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html) para realizar el procesamiento de datos deseado.

> [!NOTE]  
> Normalmente, HDFS no se implementa dentro del clúster HDInsight para proporcionar almacenamiento. En su lugar, los componentes de Hadoop utilizan una capa de interfaz compatible con HDFS. La capacidad de almacenamiento real la proporcionan Azure Storage o Azure Data Lake Storage. Para Hadoop, los trabajos MapReduce que se ejecutan en el clúster HDInsight, lo hacen como si HDFS estuviese presente y no requieren cambios para satisfacer sus necesidades de almacenamiento. En Hadoop en HDInsight, el almacenamiento es externo, pero el procesamiento YARN sigue siendo un componente principal. Para más información, consulte [Introducción a Azure HDInsight](hadoop/apache-hadoop-introduction.md).

En este artículo se introduce YARN y cómo coordina la ejecución de aplicaciones en HDInsight.

## <a name="apache-hadoop-yarn-basics"></a>Conceptos básicos de Apache Hadoop y YARN

YARN rige y organiza el procesamiento de datos en Hadoop. YARN tiene dos servicios principales que se ejecutan como procesos en nodos del clúster:

* ResourceManager
* NodeManager

ResourceManager concede recursos de proceso de clúster a aplicaciones, como trabajos MapReduce. ResourceManager concede estos recursos como contenedores, y cada contenedor se compone de una asignación de núcleos de CPU y memoria RAM. Si se combinan todos los recursos disponibles en un clúster y, después, los núcleos y la memoria se distribuyen en bloques, cada bloque de recursos es un contenedor. Cada nodo del clúster tiene capacidad para un determinado número de contenedores, por lo tanto, el clúster tiene un límite fijo de número de contenedores disponibles. La asignación de recursos en un contenedor es configurable.

Cuando se ejecuta una aplicación MapReduce en un clúster, ResourceManager proporciona a la aplicación los contenedores en los que se puede ejecutar. ResourceManager realiza un seguimiento del estado de las aplicaciones en ejecución y la capacidad de clúster disponible, y realiza un seguimiento de cuándo se completan las aplicaciones y liberan sus recursos.

ResourceManager también ejecuta un proceso de servidor web que proporciona una interfaz de usuario web para supervisar el estado de las aplicaciones.

Cuando un usuario envía una aplicación MapReduce para que se ejecute en el clúster, la aplicación se envía a ResourceManager. A su vez, ResourceManager asigna un contenedor en nodos de NodeManager disponibles. Los nodos de NodeManager son la ubicación en que realmente se ejecuta la aplicación. El primer contenedor asignado ejecuta una aplicación especial denominada ApplicationMaster. ApplicationMaster es responsable de adquirir recursos, en forma de contenedores subsiguientes, necesarios para ejecutar la aplicación enviada. ApplicationMaster examina las fases de la aplicación, como la fase de asignación y la de reducción, y considera la cantidad de datos que deben procesarse. A continuación, ApplicationMaster solicita (*negocia*) los recursos de ResourceManager en nombre de la aplicación. ResourceManager concede, a su vez, recursos de NodeManagers del clúster a ApplicationMaster para que los use al ejecutar la aplicación.

Los servicios NodeManager ejecutan las tareas que componen la aplicación y, a continuación, informan de su progreso y estado a ApplicationMaster. ApplicationMaster, a su vez, informa del estado de la aplicación a ResourceManager. ResourceManager devuelve todos los resultados al cliente.

## <a name="yarn-on-hdinsight"></a>YARN en HDInsight

Todos los tipos de clúster HDInsight implementan YARN. ResourceManager se implementa para lograr alta disponibilidad con una instancia principal y secundaria, que se ejecutan en el primer y segundo nodo principal dentro del clúster, respectivamente. Solo la única instancia de ResourceManager está activa a la vez. Las instancias de NodeManager se ejecutan en todos los nodos de trabajo disponibles en el clúster.

![Apache YARN en HDInsight de Azure](./media/hdinsight-hadoop-architecture/apache-yarn-on-hdinsight.png)

## <a name="soft-delete"></a>Eliminación temporal

Para recuperar un archivo de la cuenta de almacenamiento, consulte:

### <a name="azure-storage"></a>Azure Storage

* [Eliminación temporal de blobs de Azure Storage](../storage/blobs/storage-blob-soft-delete.md)
* [Undelete Blob](https://docs.microsoft.com/rest/api/storageservices/undelete-blob)

### <a name="azure-data-lake-storage-gen-1"></a>Azure Data Lake Storage Gen 1

[Restore-AzDataLakeStoreDeletedItem](https://docs.microsoft.com/powershell/module/az.datalakestore/restore-azdatalakestoredeleteditem)

### <a name="azure-data-lake-storage-gen-2"></a>Azure Data Lake Storage Gen 2

[Problemas conocidos con Azure Data Lake Storage Gen2](../storage/blobs/data-lake-storage-known-issues.md)

## <a name="trash-purging"></a>Purga de la papelera

La propiedad `fs.trash.interval` de **HDFS** > **Advanced core-site** debe permanecer en su valor predeterminado de `0`, ya que no se debe almacenar ningún dato en el sistema de archivos local. Este valor no afecta a las cuentas de almacenamiento remoto (WASB, ADLS GEN1, ABFS)

## <a name="next-steps"></a>Pasos siguientes

* [Uso de MapReduce con Apache Hadoop en HDInsight](hadoop/hdinsight-use-mapreduce.md)
* [Introducción a Azure HDInsight](hadoop/apache-hadoop-introduction.md)
