---
title: 'Optimización del rendimiento en Azure Data Lake Storage Gen1: MapReduce'
description: Obtenga información sobre la optimización del rendimiento de MapReduce en Azure Data Lake Storage Gen1, incluidos parámetros, instrucciones, un cálculo de ejemplo y limitaciones.
author: stewu
ms.service: data-lake-store
ms.topic: how-to
ms.date: 12/19/2016
ms.author: stewu
ms.openlocfilehash: 03c35d0af97cf24d1683d0ff21f10a0371391616
ms.sourcegitcommit: 9ce0350a74a3d32f4a9459b414616ca1401b415a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88190207"
---
# <a name="performance-tuning-guidance-for-mapreduce-on-hdinsight-and-azure-data-lake-storage-gen1"></a>Guía para la optimización del rendimiento de MapReduce en HDInsight y Azure Data Lake Storage Gen1

## <a name="prerequisites"></a>Prerrequisitos

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* **Una cuenta de Azure Data Lake Storage Gen1**. Para instrucciones sobre cómo crear una, consulte la [introducción a Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* **Clúster de Azure HDInsight** con acceso a una cuenta de Data Lake Storage Gen1. Consulte [Creación de un clúster de HDInsight con Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md). Asegúrese de habilitar el Escritorio remoto para el clúster.
* **Uso de MapReduce en HDInsight**. Para más información, consulte [Uso de MapReduce en Hadoop en HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-use-mapreduce).
* **Revisión de las directrices para la optimización del rendimiento en Data Lake Storage Gen1**. Para conocer los conceptos generales sobre rendimiento, consulte [Guía para la optimización del rendimiento de Data Lake Storage Gen1](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-performance-tuning-guidance).

## <a name="parameters"></a>Parámetros

Al ejecutar trabajos de MapReduce, estos son los parámetros más importantes que se pueden configurar para aumentar el rendimiento en Data Lake Storage Gen1:

|Parámetro      | Descripción  |
|---------|---------|
|`Mapreduce.map.memory.mb`  |  Cantidad de memoria que se va a asignar a cada asignador.  |
|`Mapreduce.job.maps`     |  Número de tareas de asignación por trabajo.  |
|`Mapreduce.reduce.memory.mb`     |  Cantidad de memoria que se va a asignar a cada reductor.  |
|`Mapreduce.job.reduces`    |   Número de tareas de reducción por trabajo.  |

### <a name="mapreducemapmemory--mapreducereducememory"></a>Mapreduce.map.memory/Mapreduce.reduce.memory

Ajuste este número en función de la cantidad de memoria necesaria para la tarea de asignación o reducción. Puede ver los valores predeterminados de `mapreduce.map.memory` y `mapreduce.reduce.memory` en Ambari a través de la configuración de Yarn. En Ambari, navegue a YARN y examine la pestaña **Configs** (Configuraciones). Se mostrará la memoria YARN.

### <a name="mapreducejobmaps--mapreducejobreduces"></a>Mapreduce.job.maps/Mapreduce.job.reduces

Determina el número máximo de asignadores o reductores que se van a crear. El número de divisiones determinará cuántos asignadores se crearán para el trabajo de MapReduce. Por lo tanto, puede que obtenga menos asignadores que los que solicitó si hay menos divisiones que el número de asignadores solicitados.

## <a name="guidance"></a>Guía

### <a name="step-1-determine-number-of-jobs-running"></a>Paso 1: Determinación del número de trabajos en ejecución

De forma predeterminada, MapReduce usará todo el clúster para el trabajo. Puede usar una cantidad menor del clúster usando menos asignadores que los contenedores disponibles. En las instrucciones de este documento se asume que la aplicación es la única aplicación que se ejecuta en el clúster.

### <a name="step-2-set-mapreducemapmemorymapreducereducememory"></a>Paso 2: Establecimiento de mapreduce.map.memory/mapreduce.reduce.memory

El tamaño de la memoria para las tareas de asignación y reducción dependerá del trabajo específico. Puede reducir el tamaño de memoria si quiere aumentar la simultaneidad. El número de tareas en ejecución al mismo tiempo depende del número de contenedores. Al reducir la cantidad de memoria por asignador o reductor, pueden crearse más contenedores, que permiten que más asignadores o reductores se ejecuten a la vez. Si se reduce demasiado la cantidad de memoria puede provocar que algunos procesos se ejecuten con memoria insuficiente. Si recibe un error de montón al ejecutar el trabajo, aumente la memoria por asignador o reductor. Tenga en cuenta que la adición de más contenedores supondrá una sobrecarga adicional para cada contenedor de más, lo que podría reducir el rendimiento. Otra alternativa consiste en obtener más memoria mediante un clúster que tenga mayor cantidad de memoria o aumentar el número de nodos en el clúster. Más memoria permitirá que se usen más contenedores, lo que se traduce en una mayor simultaneidad.

### <a name="step-3-determine-total-yarn-memory"></a>Paso 3: Determinación de la memoria de YARN total

Para optimizar mapreduce.job.maps/mapreduce.job.reduces, considere la cantidad de memoria de YARN total disponible para su uso. Esta información está disponible en Ambari. Navegue a YARN y examine la pestaña **Configs** (Configuraciones). En esta ventana se muestra el tamaño de la memoria de YARN. Para obtener la memoria de YARN total, multiplique la memoria de YARN por el número de nodos que tiene en el clúster.

`Total YARN memory = nodes * YARN memory per node`

Si va a usar un clúster vacío, la memoria puede ser la memoria de YARN total para el clúster. Si otras aplicaciones usan memoria, puede elegir usar solamente una parte de la memoria del clúster; para ello, reduzca el número de asignadores o reductores al número de contenedores que quiere usar.

### <a name="step-4-calculate-number-of-yarn-containers"></a>Paso 4: Cálculo del número de contenedores de YARN

Los contenedores YARN dictaminan la cantidad de simultaneidad disponible para el trabajo. Tome la memoria de YARN total y divídala entre el valor de mapreduce.map.memory.

`# of YARN containers = total YARN memory / mapreduce.map.memory`

### <a name="step-5-set-mapreducejobmapsmapreducejobreduces"></a>Paso 5: Establecimiento de mapreduce.job.maps/mapreduce.job.reduces

Establece mapreduce.job.maps/mapreduce.job.reduces en al menos el número de contenedores disponibles. Puede seguir experimentando y aumentar el número de asignadores y reductores para ver si obtiene mejor rendimiento. Tenga en cuenta que cuantos más asignadores más sobrecarga, por lo que tener demasiados asignadores puede reducir el rendimiento.

Nota: la programación de CPU y el aislamiento de CPU están desactivadas de forma predeterminada, de modo que el número de contenedores de YARN está restringido por la memoria.

## <a name="example-calculation"></a>Cálculo de ejemplo

Supongamos que tiene actualmente un clúster compuesto de 8 nodos D14 y quiere ejecutar un trabajo de uso intensivo de E/S. Estos son los cálculos que se debe realizar:

### <a name="step-1-determine-number-of-jobs-running"></a>Paso 1: Determinación del número de trabajos en ejecución

En nuestro ejemplo, suponemos que nuestro trabajo es el único en ejecución.

### <a name="step-2-set-mapreducemapmemorymapreducereducememory"></a>Paso 2: Establecimiento de mapreduce.map.memory/mapreduce.reduce.memory

En nuestro ejemplo, ejecuta un trabajo con uso intensivo de operaciones de E/S y decide que 3 GB de memoria para las tareas de asignación serán suficientes.

`mapreduce.map.memory = 3GB`

### <a name="step-3-determine-total-yarn-memory"></a>Paso 3: Determinación de la memoria de YARN total

`total memory from the cluster is 8 nodes * 96GB of YARN memory for a D14 = 768GB`

### <a name="step-4-calculate--of-yarn-containers"></a>Paso 4: Cálculo del número de contenedores de YARN

`# of YARN containers = 768 GB of available memory / 3 GB of memory = 256`

### <a name="step-5-set-mapreducejobmapsmapreducejobreduces"></a>Paso 5: Establecimiento de mapreduce.job.maps/mapreduce.job.reduces

`mapreduce.map.jobs = 256`

## <a name="limitations"></a>Limitaciones

**Límite de Data Lake Storage Gen1**

Como servicio multiinquilino, Data Lake Storage Gen1 establece los límites de ancho de banda de nivel de cuenta. Si se alcanzan estos límites, comenzará a ver errores de tarea. Para identificar esta situación, observe los errores de limitación en los registros de tareas. Si necesita más ancho de banda para su trabajo, póngase en contacto con nosotros.

Para comprobar si le están aplicando limitaciones, debe habilitar el registro de depuración en el lado del cliente. Así es cómo debe hacerlo:

1. Coloque la siguiente propiedad en las propiedades log4j en Ambari > YARN > Config (Configuración) > Advanced (Opciones avanzadas) yarn-log4j: log4j.logger.com.microsoft.azure.datalake.store=DEBUG

2. Reinicie todos los nodos/servicios para que la configuración surta efecto.

3. Si se le están aplicando límites, verá el código de error HTTP 429 en el archivo de registro de YARN. El archivo de registro de YARN se encuentra en /tmp/&lt;user&gt;/yarn.log

## <a name="examples-to-run"></a>Ejemplos de ejecución

Para demostrar cómo MapReduce se ejecuta en Data Lake Storage Gen1, a continuación se muestra código de ejemplo que se ejecutó en un clúster con la siguiente configuración:

* 16 nodos D14v2
* Clúster de Hadoop con HDI 3.6

Como punto de partida, estos son algunos comandos de ejemplo para ejecutar MapReduce: Teragen, Terasort y Teravalidate. Puede ajustar estos comandos en función de los recursos.

### <a name="teragen"></a>Teragen

```
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teragen -Dmapreduce.job.maps=2048 -Dmapreduce.map.memory.mb=3072 10000000000 adl://example/data/1TB-sort-input
```

### <a name="terasort"></a>Terasort

```
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar terasort -Dmapreduce.job.maps=2048 -Dmapreduce.map.memory.mb=3072 -Dmapreduce.job.reduces=512 -Dmapreduce.reduce.memory.mb=3072 adl://example/data/1TB-sort-input adl://example/data/1TB-sort-output
```

### <a name="teravalidate"></a>Teravalidate

```
yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teravalidate -Dmapreduce.job.maps=512 -Dmapreduce.map.memory.mb=3072 adl://example/data/1TB-sort-output adl://example/data/1TB-sort-validate
```
