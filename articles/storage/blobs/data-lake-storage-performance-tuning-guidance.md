---
title: Optimizar Azure Data Lake Storage Gen2 para el rendimiento | Microsoft Docs
description: Guía para la optimización del rendimiento en Azure Data Lake Storage Gen2
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 11/18/2019
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: f38dc72a45d84b549baa17c34f2c64ac35b83e3e
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87285397"
---
# <a name="optimize-azure-data-lake-storage-gen2-for-performance"></a>Optimizar Azure Data Lake Storage Gen2 para el rendimiento

Azure Data Lake Storage Gen2 permite un alto rendimiento en el movimiento de datos y el análisis de consumo de la entrada y salida.  En Data Lake Storage Gen2, usar toda la capacidad de proceso disponible (la cantidad de datos que se pueden leer o escribir por segundo) es importante para obtener el mejor rendimiento.  Se consigue realizando tantas lecturas y escrituras en paralelo como sea posible.

![Rendimiento de Data Lake Storage Gen2](./media/data-lake-storage-performance-tuning-guidance/throughput.png)

Data Lake Storage Gen2 puede escalar para ofrecer la capacidad de proceso necesaria para todos los escenarios de análisis. De forma predeterminada, una cuenta Data Lake Storage Gen2 proporciona automáticamente la capacidad suficiente para satisfacer las necesidades de una amplia variedad de casos de uso. Cuando los clientes llegan al límite predeterminado, la cuenta Data Lake Storage Gen2 se puede configurar para proporcionar más rendimiento, si se ponen en contacto con [Soporte técnico de Azure](https://azure.microsoft.com/support/faq/).

## <a name="data-ingestion"></a>Ingesta de datos

Cuando se ingieren datos de un sistema de origen para Data Lake Storage Gen2, es importante tener en cuenta que el hardware de origen, el hardware de red de origen y la conectividad de red con Data Lake Storage Gen2 pueden constituir un cuello de botella.  

![Rendimiento de Data Lake Storage Gen2](./media/data-lake-storage-performance-tuning-guidance/bottleneck.png)

Es importante asegurarse de que el movimiento de datos no se ve afectado por estos factores.

### <a name="source-hardware"></a>Hardware de origen

Si usa máquinas locales o máquinas virtuales en Azure, debe seleccionar cuidadosamente el hardware apropiado. Como hardware del disco de origen, elija SSD antes que HDD y seleccione el hardware de disco con los ejes más rápidos. Como hardware de la red de origen, use los NIC más rápidos que sea posible.  En Azure, se recomienda que las máquinas virtuales de Azure D14 tengan el hardware de red y de disco con la capacidad apropiada.

### <a name="network-connectivity-to-data-lake-storage-gen2"></a>Conectividad de red para Data Lake Storage Gen2

La conectividad de red entre los datos de origen y Data Lake Storage Gen2 a veces puede constituir un cuello de botella. Cuando los datos de origen están en local, considere el uso de un vínculo dedicado con [Azure ExpressRoute](https://azure.microsoft.com/services/expressroute/). Si los datos de origen están en Azure, el rendimiento será el mejor cuando los datos se encuentren en la misma región de Azure que Data Lake Storage Gen2.

### <a name="configure-data-ingestion-tools-for-maximum-parallelization"></a>Configuración de herramientas de ingesta de datos para lograr una paralelización máxima

Una vez haya solucionado los cuellos de botella de la conectividad de la red y del hardware de origen, estará listo para configurar las herramientas de ingesta. En la tabla siguiente se resume la configuración básica de varias herramientas de ingesta populares y se ofrecen artículos detallados sobre la optimización del rendimiento relacionados con ellas.  Para más información sobre la herramienta adecuada para su escenario, consulte este [artículo](data-lake-storage-data-scenarios.md).

| Herramienta               | Configuración     | Más detalles                                                                 |
|--------------------|------------------------------------------------------|------------------------------|
| DistCp            | -m (mapper)   | [Vínculo](data-lake-storage-use-distcp.md#performance-considerations-while-using-distcp)                             |
| Azure Data Factory| parallelCopies    | [Vínculo](../../data-factory/copy-activity-performance.md)                          |
| Sqoop           | fs.azure.block.size, -m (mapper)    |   [Vínculo](https://docs.microsoft.com/archive/blogs/shanyu/performance-tuning-for-hdinsight-storm-and-microsoft-azure-eventhubs)        |

## <a name="structure-your-data-set"></a>Estructuración del conjunto de datos

Cuando los datos se almacenan en Data Lake Storage Gen2, el tamaño de los archivos, su número y la estructura de carpetas afectan al rendimiento.  En la siguiente sección se describen los procedimientos recomendados en estas áreas.  

### <a name="file-size"></a>Tamaño de archivo

Normalmente, los motores de análisis como HDInsight y Azure Data Lake Analytics tienen una sobrecarga por cada archivo. Si los datos se almacenan en muchos archivos pequeños, puede afectar desfavorablemente al rendimiento. En general, organice los datos en archivos de un tamaño mayor para mejorar el rendimiento (tamaño de 256 MB a 100 GB). Algunos motores y aplicaciones podrían tener problemas para procesar eficazmente los archivos que tienen un tamaño superior a 100 GB.

A veces, las canalizaciones de datos ejercen un control limitado sobre los datos sin procesar que tienen una gran cantidad de archivos pequeños. Se recomienda disponer de un proceso de "cocinado" que genere archivos de mayor tamaño para usarlos en las aplicaciones de bajada.

### <a name="organizing-time-series-data-in-folders"></a>Organización de los datos de serie temporal en carpetas

En las cargas de trabajo de Hive, la eliminación de las particiones de los datos de serie temporal puede ayudar a algunas consultas a leer solo un subconjunto de los datos, lo que mejora el rendimiento.    

Aquellas canalizaciones que ingieren datos de serie temporal suelen ubicar sus archivos con una nomenclatura muy estructurada para los archivos y las carpetas. A continuación se muestra un ejemplo muy común para los datos estructurados por fecha:

*\DataSet\YYYY\MM\DD\datafile_YYYY_MM_DD.tsv*

Observe que la información de fecha y hora aparece tanto en las carpetas como en el nombre de archivo.

Para la fecha y la hora, el siguiente es un patrón común

*\DataSet\YYYY\MM\DD\HH\mm\datafile_YYYY_MM_DD_HH_mm.tsv*

De nuevo, su elección de organización de los archivos y carpetas debería ser la que consiga un tamaño de archivo mayor y un número razonable de archivos en cada carpeta.

## <a name="optimizing-io-intensive-jobs-on-hadoop-and-spark-workloads-on-hdinsight"></a>Optimización de trabajos con muchas operaciones de E/S en las cargas de trabajo de Hadoop y Spark en HDInsight

Los trabajos se dividen en una de las siguientes tres categorías:

* **Gran uso de CPU.**  Estos trabajos tienen tiempos de cálculo largos con tiempos de E/S mínimos.  Algunos ejemplos son los trabajos de procesamiento del lenguaje natural y el aprendizaje automático.  
* **Gran uso de memoria.**  Estos trabajos usan una gran cantidad de memoria.  Algunos ejemplos son los trabajos de análisis en tiempo real y PageRank.  
* **Gran uso de operaciones de E/S.**  Estos trabajos dedican la mayor parte de su tiempo a realizar operaciones de E/S.  Un ejemplo común es un trabajo de copia que solo realice operaciones de lectura y escritura.  Otros ejemplos son los trabajos de preparación de datos que leen una gran cantidad de datos, realizan alguna transformación en ellos y luego los escriben de nuevo en el almacén.  

Las siguientes instrucciones solo se pueden aplicar a los trabajos que usan mucho las operaciones de E/S.

## <a name="general-considerations"></a>Consideraciones generales

Puede tener un trabajo que lee o escribe hasta 100 MB en una sola operación, pero almacenar en búfer dicho tamaño puede comprometer el rendimiento.
Para optimizar el rendimiento, intente mantener el tamaño de una operación de E/S entre 4 MB y 16 MB.

### <a name="general-considerations-for-an-hdinsight-cluster"></a>Consideraciones generales para un clúster de HDInsight

* **Versiones de HDInsight.** Para obtener el mejor rendimiento, utilice la versión más reciente de HDInsight.
* **Regiones.** Coloque la cuenta de Data Lake Storage Gen2 en la misma región que el clúster de HDInsight.  

Un clúster de HDInsight se compone de dos nodos principales y algunos nodos de trabajo. Cada nodo de trabajo proporciona una cantidad específica de núcleos y de memoria, lo que se determina según el tipo de máquina virtual.  Cuando se ejecuta un trabajo, YARN es el negociador de recursos que asigna la memoria disponible y los núcleos para crear los contenedores.  Cada contenedor ejecuta las tareas necesarias para completar el trabajo.  Los contenedores se ejecutan en paralelo para procesar las tareas rápidamente. Por lo tanto, el rendimiento se mejora ejecutando tantos contenedores en paralelo como sea posible.

Hay tres niveles dentro de un clúster de HDInsight que se pueden optimizar para aumentar el número de contenedores y usar toda la capacidad de proceso disponible.  

* **Nivel físico**
* **Nivel de YARN**
* **Nivel de carga de trabajo**

### <a name="physical-layer"></a>Nivel físico

**Ejecute el clúster con más nodos o máquinas virtuales de un tamaño mayor.**  Un clúster mayor le permitirá ejecutar más contenedores de YARN, tal como se muestra en la ilustración siguiente.

![Rendimiento de Data Lake Storage Gen2](./media/data-lake-storage-performance-tuning-guidance/VM.png)

**Use máquinas virtuales con más ancho de banda de red.**  La cantidad de ancho de banda de red puede ser un cuello de botella, si hay menos de lo que necesita la capacidad de proceso de Data Lake Storage Gen2.  Diferentes máquinas virtuales tendrán varios tamaños de ancho de banda de red.  Elija un tipo de máquina virtual que tenga el mayor ancho de banda de red posible.

### <a name="yarn-layer"></a>Nivel de YARN

**Use contenedores de YARN menores.**  Reduzca el tamaño de cada contenedor de YARN para crear más contenedores con la misma cantidad de recursos.

![Rendimiento de Data Lake Storage Gen2](./media/data-lake-storage-performance-tuning-guidance/small-containers.png)

En función de la carga de trabajo, siempre habrá un tamaño de contenedor de YARN mínimo que se necesite. Si elige un contenedor demasiado pequeño, los trabajos no tendrán memoria suficiente. Normalmente, los contenedores de YARN no deben ser menores de 1 GB. Es habitual ver contenedores de YARN de 3 GB. Para algunas cargas de trabajo, puede que necesite contenedores de YARN mayores.  

**Aumente los núcleos para cada contenedor de YARN.**  Aumente el número de núcleos asignados a cada contenedor para incrementar las tareas en paralelo que se ejecutan en cada uno.  Funciona en las aplicaciones como Spark que ejecutan varias tareas por contenedor.  En las aplicaciones como Hive que ejecutan un único subproceso en cada contenedor, es mejor tener varios contenedores en lugar de más núcleos por contenedor.

### <a name="workload-layer"></a>Nivel de carga de trabajo

**Use todos los contenedores disponibles.**  Establezca el número de tareas para que sea igual o mayor que el número de contenedores disponibles de modo que se usen todos los recursos.

![Rendimiento de Data Lake Storage Gen2](./media/data-lake-storage-performance-tuning-guidance/use-containers.png)

**Los errores de las tareas son costosos.** Si cada tarea tiene una gran cantidad de datos que se van a procesar, el error de una tarea provoca un reintento que resulta caro.  Por lo tanto, es mejor que cree más tareas y que cada una procese una pequeña cantidad de datos.

Además de las directrices generales anteriores, cada aplicación tiene diferentes parámetros disponibles que pueden optimizarse para ella en concreto. En la tabla siguiente se enumeran algunos de los parámetros y los vínculos para empezar a optimizar el rendimiento de cada aplicación.

| Carga de trabajo | Parámetro para establecer tareas |
|----------|------------------------|
| [Spark en HDInsight](data-lake-storage-performance-tuning-spark.md) | <ul><li>Num-executors</li><li>Memoria del ejecutor</li><li>Executor-cores</li></ul> |
| [Hive en HDInsight](data-lake-storage-performance-tuning-hive.md) | <ul><li>hive.tez.container.size</li></ul> |
| [MapReduce en HDInsight](data-lake-storage-performance-tuning-mapreduce.md) | <ul><li>Mapreduce.map.memory</li><li>Mapreduce.job.maps</li><li>Mapreduce.reduce.memory</li><li>Mapreduce.job.reduces</li></ul> |
| [Storm en HDInsight](data-lake-storage-performance-tuning-storm.md)| <ul><li>Número de procesos de trabajo</li><li>Número de instancias de ejecutor de spout</li><li>Número de instancias de ejecutor de bolt </li><li>Número de tareas de spout</li><li>Número de tareas de bolt</li></ul>|

## <a name="see-also"></a>Consulte también
* [Introducción a Azure Data Lake Storage Gen2](data-lake-storage-introduction.md)
