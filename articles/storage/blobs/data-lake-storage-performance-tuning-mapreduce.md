---
title: 'Ajustar rendimiento: MapReduce, HDInsight y Azure Data Lake Storage Gen2 | Microsoft Docs'
description: Conozca las directrices para optimizar el rendimiento de los trabajos de MapReduce en Azure Data Lake Storage Gen2.
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 11/18/2019
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: e9d638a7ed17d08b585c71b1dac4a0177f4a2939
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88030527"
---
# <a name="tune-performance-mapreduce-hdinsight--azure-data-lake-storage-gen2"></a>Ajustar rendimiento: MapReduce, HDInsight y Azure Data Lake Storage Gen2

Esta guía le ayuda a comprender los factores que se deben tener en cuenta al optimizar el rendimiento de los trabajos de MapReduce. En este artículo se describen varias directrices de optimización del rendimiento.

## <a name="prerequisites"></a>Requisitos previos

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* **Una cuenta de Azure Data Lake Storage Gen2**. Para obtener instrucciones sobre cómo crear una, consulte [Guía de inicio rápido: Creación de una cuenta de almacenamiento de Azure Data Lake Storage Gen2](data-lake-storage-quickstart-create-account.md).
* **Clúster de Azure HDInsight** con acceso a una cuenta de Data Lake Storage Gen2. Consulte [Use Azure Data Lake Storage Gen2 with Azure HDInsight clusters](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2) (Uso de Azure Data Lake Storage Gen2 con clústeres de Azure HDInsight)
* **Uso de MapReduce en HDInsight**.  Para más información, consulte [Uso de MapReduce en Hadoop en HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-use-mapreduce).
* **Guía para la optimización del rendimiento en Data Lake Storage Gen2**.  Para conocer los conceptos generales sobre rendimiento, consulte [Guía para la optimización del rendimiento en Data Lake Storage Gen2](data-lake-storage-performance-tuning-guidance.md).

## <a name="parameters"></a>Parámetros

Al ejecutar trabajos de MapReduce, estos son los parámetros que se pueden configurar para aumentar el rendimiento en Data Lake Storage Gen2:

* **Mapreduce.map.memory.mb**: la cantidad de memoria que se asignará a cada asignador.
* **MapReduce.job.maps**: el número de tareas de asignación por trabajo.
* **Mapreduce.reduce.memory.mb**: la cantidad de memoria que se asignará a cada reductor.
* **Mapreduce.job.reduces**: el número de tareas de reducción por trabajo.

**Mapreduce.map.memory / Mapreduce.reduce.memory**: este número se debe ajustar según la cantidad de memoria necesaria para la tarea de asignación o reducción.  Los valores predeterminados de mapreduce.map.memory y mapreduce.reduce.memory pueden verse en Ambari a través de la configuración de Yarn.  En Ambari, vaya a YARN y vea la pestaña Configs (Configuraciones).  Se mostrará la memoria YARN.  

**Mapreduce.job.maps / Mapreduce.job.reduces**: determina el número máximo de asignadores o reductores que se crearán.  El número de divisiones determinará cuántos asignadores se crearán para el trabajo de MapReduce.  Por lo tanto, puede que obtenga menos asignadores que los solicitados si hay menos divisiones que el número de asignadores solicitado.       

## <a name="guidance"></a>Guía

> [!NOTE]
> En las instrucciones de este documento se asume que la aplicación es la única aplicación que se ejecuta en el clúster.

**Paso 1: Determinación del número de trabajos que se ejecutan**

De forma predeterminada, MapReduce usará todo el clúster para el trabajo.  Puede usar una cantidad menor del clúster usando menos asignadores que contenedores haya disponibles.        

**Paso 2: Establecimiento de mapreduce.map.memory/mapreduce.reduce.memory**

El tamaño de la memoria para las tareas de asignación y reducción dependerá del trabajo específico.  Puede reducir el tamaño de memoria si quiere aumentar la simultaneidad.  El número de tareas en ejecución al mismo tiempo depende del número de contenedores.  Al reducir la cantidad de memoria por asignador o reductor, pueden crearse más contenedores, que permiten que más asignadores o reductores se ejecuten a la vez.  Si se reduce demasiado la cantidad de memoria puede provocar que algunos procesos se ejecuten con memoria insuficiente.  Si recibe un error de heap cuando se ejecuta el trabajo, debe aumentar la memoria por asignador o reductor.  Debe tener en cuenta que agregar más contenedores supondrá una sobrecarga adicional para cada contenedor de más, lo que podría reducir el rendimiento.  Otra alternativa consiste en obtener más memoria mediante un clúster que tenga mayor cantidad de memoria o aumentar el número de nodos en el clúster.  Más memoria permitirá que se usen más contenedores, lo que se traduce en una mayor simultaneidad.  

**Paso 3: Determinación de la memoria de YARN total**

Para optimizar mapreduce.job.maps/mapreduce.job.reduces, debería considerar la cantidad de memoria YARN total disponible para su uso.  Esta información está disponible en Ambari.  Vaya a YARN y vea la pestaña Configs (Configuraciones).  En esta ventana se muestra el tamaño de la memoria de YARN.  Para obtener la memoria de YARN total, debe multiplicar la memoria de YARN por el número de nodos que tiene en el clúster.

Memoria total de YARN = nodos * memoria de YARN por nodo

Si va a usar un clúster vacío, la memoria puede ser la memoria de YARN total para en el clúster.  Si otras aplicaciones usan memoria, puede elegir usar solamente una parte de la memoria del clúster; para ello, reduzca el número de asignadores o reductores al número de contenedores que quiere usar.  

**Paso 4: Cálculo del número de contenedores YARN**

Los contenedores YARN dictaminan la cantidad de simultaneidad disponible para el trabajo.  Tome la memoria de YARN total y divídala entre el valor de mapreduce.map.memory.  

\# de contenedores de YARN = memoria total de YARN / mapreduce.map.memory

**Paso 5: Establecer mapreduce.job.maps/mapreduce.job.reduces**

Establece mapreduce.job.maps/mapreduce.job.reduces en al menos el número de contenedores disponibles.  Puede seguir experimentando y aumentar el número de asignadores y reductores para ver si obtiene mejor rendimiento.  Tenga en cuenta que cuantos más asignadores más sobrecarga, por lo que tener demasiados asignadores puede reducir el rendimiento.  

Nota: la programación de CPU y el aislamiento de CPU están desactivadas de forma predeterminada, de modo que el número de contenedores de YARN está restringido por la memoria.

## <a name="example-calculation"></a>Cálculo de ejemplo

Supongamos que actualmente hay un clúster compuesto de 8 nodos D14 y se quiere ejecutar un trabajo de uso intensivo de E/S.  Estos son los cálculos que se debe realizar:

**Paso 1: Determinación del número de trabajos que se ejecutan**

En este ejemplo, supongamos que nuestro trabajo es el único trabajo que se está ejecutando.  

**Paso 2: Establecimiento de mapreduce.map.memory/mapreduce.reduce.memory**

En este ejemplo, se ejecuta un trabajo con uso intensivo de E/S y decide que 3 GB de memoria para las tareas de asignación serán suficientes.

mapreduce.map.memory = 3 GB

**Paso 3: Determinación de la memoria de YARN total**

La memoria total del clúster es de 8 nodos * 96 GB de memoria de YARN para un D14 = 768 GB

**Paso 4: Cálculo del número de contenedores de YARN**

\# de contenedores de YARN = 768 GB de memoria disponible /3 GB de memoria = 256

**Paso 5: Establecer mapreduce.job.maps/mapreduce.job.reduces**

mapreduce.map.jobs = 256

## <a name="examples-to-run"></a>Ejemplos de ejecución

Para demostrar cómo MapReduce se ejecuta en Data Lake Storage Gen2, a continuación se muestra código de ejemplo que se ejecutó en un clúster con la siguiente configuración:

* 16 nodos D14v2
* Clúster de Hadoop con HDI 3.6

Como punto de partida, estos son algunos comandos de ejemplo para ejecutar MapReduce: Teragen, Terasort y Teravalidate.  Puede ajustar estos comandos en función de los recursos.

**Teragen**

```cmd
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teragen -Dmapreduce.job.maps=2048 -Dmapreduce.map.memory.mb=3072 10000000000 abfs://example/data/1TB-sort-input
```

**Terasort**

```cmd
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar terasort -Dmapreduce.job.maps=2048 -Dmapreduce.map.memory.mb=3072 -Dmapreduce.job.reduces=512 -Dmapreduce.reduce.memory.mb=3072 abfs://example/data/1TB-sort-input abfs://example/data/1TB-sort-output
```

**Teravalidate**

```cmd
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teravalidate -Dmapreduce.job.maps=512 -Dmapreduce.map.memory.mb=3072 abfs://example/data/1TB-sort-output abfs://example/data/1TB-sort-validate
```
