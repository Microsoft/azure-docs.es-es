---
title: ¿Qué es Apache HBase en HDInsight de Azure?
description: Una introducción a Apache HBase en HDInsight, una base de datos NoSQL basada en Hadoop. Obtenga información acerca de los casos de uso y compare HBase con otros clústeres de Hadoop.
author: hrasheed-msft
ms.reviewer: jasonh
ms.service: hdinsight
ms.custom: hdinsightactive,hdiseo17may2017
ms.topic: conceptual
ms.date: 02/22/2018
ms.author: hrasheed
ms.openlocfilehash: a492adbf1c01271ceea736bde9de999190fc2da4
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65789198"
---
# <a name="what-is-apache-hbase-in-azure-hdinsight"></a>¿Qué es Apache HBase en HDInsight de Azure

[Apache HBase](https://hbase.apache.org/) es una base de datos NoSQL de código abierto construida sobre [Apache Hadoop](https://hadoop.apache.org/) y modelada según [Google BigTable](https://cloud.google.com/bigtable/). HBase proporciona acceso aleatorio y enorme coherencia para grandes cantidades de datos no estructurados y semiestructurados en una base de datos sin esquemas organizada por familias de columnas.

Desde la perspectiva del usuario, HBase es similar a una base de datos. Los datos se almacenan en las filas y las columnas de una tabla, mientras que los datos de una fila se agrupan por familia de columnas. HBase es una base de datos sin esquemas en el sentido de que no es preciso que ni las columnas ni el tipo de datos almacenados en ellas se definan antes de usarlos. El código abierto se escala linealmente para controlar petabytes de datos en miles de nodos. Puede basarse en la redundancia de datos, el procesamiento por lotes y otras características proporcionadas por aplicaciones distribuidas en el ecosistema Hadoop.

[!INCLUDE [hdinsight-price-change](../../../includes/hdinsight-enhancements.md)]

## <a name="how-is-apache-hbase-implemented-in-azure-hdinsight"></a>¿Cómo se implementa Apache HBase en Azure HDInsight?

HBase de HDInsight se ofrece como un clúster administrado que se integra en el entorno de Azure. Los clústeres están configurados para almacenar datos directamente en [Azure Storage](./../hdinsight-hadoop-use-blob-storage.md), que proporciona una baja latencia y una mayor elasticidad en las opciones de costo y rendimiento. Esto permite que los clientes creen sitios web interactivos que trabajen con grandes conjuntos de datos, creen servicios que almacenen datos del sensor y telemetría desde millones de extremos y analicen estos datos con trabajos de Hadoop. HBase y Hadoop son buenos puntos de partida para proyectos con Big Data en Azure y, en concreto, pueden permitir que aplicaciones en tiempo real funcionen con grandes conjuntos de datos.

La implementación de HDInsight aprovecha la arquitectura de escala de HBase para proporcionar particionamiento automático de tablas, una coherencia fuerte para lecturas y escrituras, y una conmutación automática por error. Se mejora el rendimiento mediante el almacenamiento en caché de memoria para lecturas y streaming de alto rendimiento para escrituras. Se puede crear un clúster de HBase dentro de la red virtual. Para obtener detalles, consulte [Creación de clústeres de HDInsight en Azure Virtual Network](./apache-hbase-provision-vnet.md).

## <a name="how-is-data-managed-in-hdinsight-hbase"></a>¿Cómo se administran los datos en HBase de HDInsight?
En HBase, los datos pueden administrarse mediante los comandos `create`, `get`, `put` y `scan` del shell de HBase. Los datos se escriben en la base de datos mediante `put` y se leen con `get`. El comando de `scan` se usa para obtener datos de varias filas de una tabla. Los datos también se pueden administrar usando la API de C# de HBase, que proporciona una biblioteca de cliente en la parte superior de la API de REST de HBase. Asimismo, una base de datos de HBase se puede consultar usando [Apache Hive](https://hive.apache.org/). Para obtener una introducción a estos modelos de programación, consulte [Introducción a un ejemplo de Apache HBase con Apache Hadoop en HDInsight](./apache-hbase-tutorial-get-started-linux.md). También existen coprocesadores, que permiten el procesamiento de datos en los nodos que hospedan la base de datos.

> [!NOTE]  
> Thrift no es compatible con HBase en HDInsight.

## <a name="scenarios-use-cases-for-apache-hbase"></a>Escenarios: casos de uso de Apache HBase
El caso de uso canónico para el que se creó BigTable y, por extensión, HBase, fue la búsqueda web. Los motores de búsqueda crean índices que asignan términos a las páginas web que los contienen. Sin embargo, hay muchos otros casos de uso a los que se adapta HBase, varios de los cuales se muestran en esta sección.

* almacén de pares clave-valor
  
    HBase se puede usar como almacén de pares clave-valor y es adecuado para la administración de sistemas de mensajes. Facebook usa HBase para su sistema de mensajería y es ideal para almacenar y administrar comunicaciones de Internet. WebTable usa HBase para buscar y administrar tablas extraídas de páginas web.
* datos del sensor
  
    HBase es útil para capturar datos que se recopilan cada vez más a partir de diversas fuentes. Entre estos se incluyen análisis sociales, serie temporal, actualización de paneles interactivos con tendencias y contadores y administración de sistemas de registro de auditoría. Entre los ejemplos se incluyen un terminal comercial de Bloombergy y la base de datos de series temporales abierta (OpenTSDB), que almacena y proporciona acceso a las métricas recopiladas en beneficio de los sistemas de servidor.
* consulta en tiempo real
  
    [Apache Phoenix](https://phoenix.apache.org/) es un motor de consultas SQL para HBase Apache. Se obtiene acceso a él como controlador de JDBC y permite consultar y administrar tablas de HBase usando SQL.
* HBase como plataforma
  
    Las aplicaciones pueden ejecutarse en de HBase usándolo como almacén de datos. Entre los ejemplos se incluyen Phoenix, [OpenTSDB](http://opentsdb.net/), Kiji y Titan. Las aplicaciones también pueden integrarse con HBase. Algunos ejemplos son [Apache Hive](https://hive.apache.org/), [Apache Pig](https://pig.apache.org/), [Solr](https://lucene.apache.org/solr/), [Apache Storm](https://storm.apache.org/), [Apache Flume](https://flume.apache.org/), [Apache Impala](https://impala.apache.org/), [Apache Spark](https://spark.apache.org/), [Ganglia](http://ganglia.info/) y [Apache Drill](https://drill.apache.org/).

## <a name="next-steps"></a>Pasos siguientes
* [Introducción al uso de Apache HBase con Apache Hadoop en HDInsight](./apache-hbase-tutorial-get-started-linux.md)
* [Create HDInsight clusters on Azure Virtual Network](./apache-hbase-provision-vnet.md) (Creación de clústeres de HDInsight en Azure Virtual Network)
* [Configuración de la replicación de Apache HBase en HDInsight](apache-hbase-replication.md)
* [Uso de Apache Maven para crear aplicaciones Java que utilizan Apache HBase con HDInsight (Hadoop)](./apache-hbase-build-java-maven-linux.md)

## <a name="see-also"></a>Consulte también
* [Apache HBase](https://hbase.apache.org/)
* [Guía de referencia de Apache HBase](https://hbase.apache.org/book.html)
* [BigTable: A Distributed Storage System for Structured Data](https://research.google.com/archive/bigtable.html) (BigTable: un sistema de almacenamiento distribuido para datos estructurados)
