---
title: 'Qué son Apache Hadoop y MapReduce: Azure HDInsight'
description: Una introducción a HDInsight y la pila de tecnología y los componentes de Apache Hadoop.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: overview
ms.custom: hdinsightactive,hdiseo17may2017,mvc,seodec18
ms.date: 02/27/2020
ms.openlocfilehash: 5e5f02b1684e56496778ab677aa9dc46e7dcd9aa
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086530"
---
# <a name="what-is-apache-hadoop-in-azure-hdinsight"></a>¿Qué es Apache Hadoop en Azure HDInsight?

[Apache Hadoop](https://hadoop.apache.org/) era el entorno de trabajo de código abierto original para el procesamiento distribuido y análisis de macrodatos en clústeres. El ecosistema de Hadoop incluye utilidades y software relacionados, como Apache Hive, Apache HBase, Spark, Kafka, entre otros muchos.

Azure HDInsight es un servicio de análisis, de código abierto, espectro completo y totalmente administrado en la nube para empresas. El tipo de clúster de Apache Hadoop en Azure HDInsight permite usar HDFS, la administración de recursos de YARN y un modelo sencillo de programación de MapReduce para procesar y analizar datos por lotes en paralelo.

Para ver los componentes disponibles de la pila de tecnología de Hadoop en HDInsight, consulte [¿Cuáles son los componentes y versiones de Hadoop disponibles con HDInsight?](../hdinsight-component-versioning.md). Para más información sobre Hadoop en HDInsight, consulte la [página de características de Azure para HDInsight](https://azure.microsoft.com/services/hdinsight/).

## <a name="what-is-mapreduce"></a>¿Qué es MapReduce?

MapReduce de Apache Hadoop es un marco de software para escribir trabajos que procesan enormes cantidades de datos. Los datos de entrada se dividen en fragmentos independientes. Cada fragmento se procesa en paralelo en todos los nodos del clúster. Un trabajo de MapReduce consta de dos funciones:

* **Asignador**: consume datos de entrada, los analiza (normalmente con un filtro y operaciones de ordenación) y emite tuplas (pares de clave-valor)

* **Reductor**: consume tuplas emitidas por el asignador y realiza una operación de resumen que crea un resultado combinado más pequeño de los datos del asignador

En el siguiente diagrama se muestra un ejemplo de trabajo de MapReduce de recuento de palabras básico:

 ![HDI.ProgramaRecuentoPalabras](./media/apache-hadoop-introduction/hdi-word-count-diagram.gif)

La salida de este trabajo es un recuento de las veces que aparece cada palabra en el texto.

* El asignador utiliza cada línea del texto de entrada como una entrada y la desglosa en palabras. Emite un par clave-valor cada vez que se detecta una palabra, seguida por un 1. La salida se ordena antes de enviarla al reductor.
* El reductor suma estos recuentos individuales de cada palabra y emite un solo par clave-valor que contiene la palabra seguido de la suma de sus apariciones.

MapReduce se puede implementar en varios lenguajes. Java es la implementación más común y se utiliza para fines de demostración en este documento.

## <a name="development-languages"></a>Lenguajes de desarrollo

Los lenguajes o marcos de trabajo basados en Java y la máquina virtual de Java se pueden ejecutar directamente como un trabajo de MapReduce. El ejemplo usado en este documento es una aplicación de MapReduce de Java. Los lenguajes que no son Java, como C# o Python, o ejecutables independientes, deben usar **streaming de Hadoop**.

El streaming de Hadoop se comunica con el asignador y el reductor a través de STDIN y STDOUT. El asignador y reductor leen datos línea a línea desde STDIN y escriben el resultado en STDOUT. Cada línea leída o emitida por el asignador y el reductor debe estar en el formato de un par de clave-valor delimitado por un carácter de tabulación:

`[key]\t[value]`

Para obtener más información, consulte [Streaming de Hadoop](https://hadoop.apache.org/docs/current/hadoop-streaming/HadoopStreaming.html).

Para obtener ejemplos del uso del streaming de Hadoop con HDInsight, consulte el siguiente documento:

* [Desarrollar trabajos de MapReduce de C#](apache-hadoop-dotnet-csharp-mapreduce-streaming.md)

## <a name="next-steps"></a>Pasos siguientes

* [Create Apache Hadoop cluster in HDInsight](apache-hadoop-linux-create-cluster-get-started-portal.md) (Creación de un clúster de Apache Hadoop en HDInsight)
