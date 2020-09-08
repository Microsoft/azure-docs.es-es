---
title: 'Ajustar rendimiento: Spark, HDInsight y Azure Data Lake Storage Gen2 | Microsoft Docs'
description: Conozca las directrices para optimizar el rendimiento de Spark con Azure HDInsight y Azure Data Lake Storage Gen2.
services: storage
author: normesta
ms.subservice: data-lake-storage-gen2
ms.service: storage
ms.topic: how-to
ms.date: 11/18/2019
ms.author: normesta
ms.reviewer: stewu
ms.openlocfilehash: 8ae9f96b42c0eb36a9380589780d141711c7ae4d
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88034741"
---
# <a name="tune-performance-spark-hdinsight--azure-data-lake-storage-gen2"></a>Ajustar rendimiento: Spark, HDInsight y Azure Data Lake Storage Gen2

Cuando ajuste el rendimiento de Spark, necesitará tener en cuenta el número de aplicaciones que se ejecutarán en su clúster.  De forma predeterminada, puede ejecutar 4 aplicaciones simultáneamente en su clúster HDI. (Nota: la configuración predeterminada está sujeta a cambios).  Puede decidir usar menos aplicaciones, por lo que puede sustituir la configuración predeterminada y utilizar más del clúster para esas aplicaciones.  

## <a name="prerequisites"></a>Requisitos previos

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* **Una cuenta de Azure Data Lake Storage Gen2**. Para obtener instrucciones sobre cómo crear una, consulte [Guía de inicio rápido: Creación de una cuenta de almacenamiento de Azure Data Lake Storage Gen2](data-lake-storage-quickstart-create-account.md).
* **Clúster de Azure HDInsight** con acceso a una cuenta de Data Lake Storage Gen2. Consulte [Uso de Data Lake Storage Gen2 con clústeres de Azure HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-use-data-lake-storage-gen2). Asegúrese de habilitar el Escritorio remoto para el clúster.
* **Ejecutar el clúster de Spark en Data Lake Storage Gen2**.  Para más información, vea [Usar un clúster de HDInsight Spark para analizar los datos en Data Lake Storage Gen2](https://docs.microsoft.com/azure/hdinsight/hdinsight-apache-spark-use-with-data-lake-store).
* **Guía para la optimización del rendimiento en Data Lake Storage Gen2**.  Para conocer los conceptos generales sobre rendimiento, consulte [Guía para la optimización del rendimiento en Data Lake Storage Gen2](data-lake-storage-performance-tuning-guidance.md). 

## <a name="parameters"></a>Parámetros

Estos son los valores más importantes que se pueden optimizar para aumentar el rendimiento en Data Lake Storage Gen2 cuando ejecute trabajos de Spark:

* **num-executors**: número de tareas simultáneas que se pueden ejecutar.

* **executor-memory**: cantidad de memoria asignada a cada ejecutor.

* **executor-cores**: número de núcleos asignados a cada ejecutor.                     

**num-executors**. En num-executors se define el número máximo de tareas que se pueden ejecutar en paralelo.  El número real de tareas que se pueden ejecutar en paralelo está limitado por los recursos de CPU y memoria disponibles en el clúster.

**executor-memory**. Es la cantidad de memoria que se asigna a cada ejecutor.  La memoria necesaria para cada ejecutor depende del trabajo.  En las operaciones complejas, la memoria debe ser mayor.  En cambio, para las simples como la lectura y escritura, los requisitos de memoria serán menores.  La cantidad de memoria para cada ejecutor puede verse en Ambari.  En Ambari, vaya a Spark y vea la pestaña Configs (Configuraciones).  

**Executor-cores** Establece el número de núcleos que se usan por ejecutor, lo que determina el número de subprocesos paralelos que se pueden ejecutar por ejecutor.  Por ejemplo, si executor-cores = 2, entonces cada ejecutor puede ejecutar 2 tareas en paralelo.  Los núcleos de ejecutor necesarios dependerán del trabajo.  Los trabajos con muchas E/S no requieren una gran cantidad de memoria por tarea, por lo que cada ejecutor puede administrar más tareas paralelas.

De forma predeterminada, se definen dos núcleos YARN virtuales para cada núcleo físico cuando se ejecuta Spark en HDInsight.  Este número proporciona un buen equilibrio entre simultaneidad y la cantidad de cambios de contexto desde varios subprocesos.  

## <a name="guidance"></a>Guía

Si va a ejecute cargas de trabajo de análisis de Spark para trabajar con datos de Data Lake Storage Gen2, se recomienda usar la versión más reciente de HDInsight para conseguir el máximo rendimiento con Data Lake Storage Gen2. Si el trabajo tiene un uso intensivo de E/S, hay determinados parámetros que puede configurar para mejorar el rendimiento.  Data Lake Storage Gen2 es una plataforma de almacenamiento altamente escalable que puede controlar un alto rendimiento.  Si el trabajo está compuesto principalmente por lecturas o escrituras, aumentar la simultaneidad de E/S hacia y desde Data Lake Storage Gen2 podría aumentar el rendimiento.

Para aumentar la simultaneidad de trabajos de uso intensivo de E/S, existen algunos métodos generales:

**Paso 1: Determine cuántas aplicaciones se ejecutan en el clúster**. Debe conocer cuántas aplicaciones se están ejecutando en el clúster, incluida la actual.  Los valores predeterminados para cada ajuste de Spark asume que hay 4 aplicaciones ejecutándose simultáneamente.  Por lo tanto, solo habrá un 25 % del clúster disponible para cada aplicación.  Para obtener un mejor rendimiento, puede sustituir los valores predeterminados cambiando el número de ejecutores.  

**Paso 2: Establezca executor-memory**. Lo primero que se debe establecer es executor-memory.  La memoria dependerá el trabajo que va a ejecutar.  Puede aumentar la simultaneidad asignando menos memoria a cada ejecutor.  Si ve excepciones por memoria insuficiente cuando ejecuta el trabajo, aumente el valor para este parámetro.  Otra alternativa consiste en obtener más memoria mediante un clúster que tenga mayor cantidad de la misma o aumentar el número de nodos en el clúster.  Mientras más memoria tenga, más ejecutores podrá utilizar, lo que se traduce en una mayor simultaneidad.

**Paso 3: Establezca executor-cores**. En cargas de trabajo con uso intensivo de E/S que no tengan operaciones complejas, es recomendable empezar con un gran número de núcleos de ejecutor para aumentar el número de tareas paralelas por cada ejecutor.  Un buen punto de partida sería establecer executor-cores en 4.   

--executor-cores = 4

Si aumenta el número de núcleos de ejecutor podrá tener mayor paralelismo, con lo que puede experimentar con diferentes valores para executor-cores.  Para los trabajos que tengan operaciones más complejas, reduzca el número de núcleos por ejecutor.  Si executor-cores se ha establecido en un número mayor que 4, puede que la recolección de elementos no utilizados sea ineficaz y el rendimiento se vea degradado.

**Paso 4: Determine la cantidad de memoria YARN en el clúster**. Esta información está disponible en Ambari.  Vaya a YARN y vea la pestaña Configs (Configuraciones).  En esta ventana se muestra el tamaño de la memoria de YARN.  
Tenga en cuenta que mientras está en la ventana puede ver también el tamaño predeterminado del contenedor YARN.  El tamaño del contenedor YARN es igual que la memoria por cada parámetro de ejecutor.

Memoria total de YARN = nodos * memoria de YARN por nodo

**Paso 5: Calcule num-executors**

**Calcule la restricción de memoria**: el parámetro num-executors está restringido por la memoria o por la CPU.  La restricción de memoria se determina por la cantidad de memoria YARN disponible para la aplicación.  Tome la memoria total de YARN y divídala por la memoria del ejecutor.  La restricción debe adaptarse al número de aplicaciones, por lo que dividiremos entre dicho número.

Restricción de memoria = (memoria de YARN total / memoria del ejecutor) / número de aplicaciones

**Calcule la restricción de CPU**: la restricción de la CPU se calcula como el total de núcleos virtuales divididos por el número de núcleos por ejecutor.  Hay 2 núcleos virtuales para cada núcleo físico.  Al igual que en la restricción de memoria, se divide por el número de aplicaciones.

- núcleos virtuales = (nodos del clúster * n.º de núcleos físicos en el nodo * 2)
- Restricción de CPU = (número total de núcleos virtuales / número de núcleos por ejecutor) / número de aplicaciones

**Establezca num-executors**: el parámetro num-executors se determina con la restricción de memoria y la restricción de CPU mínimas. 

num-executors = Min (número total de núcleos virtuales / número de núcleos por ejecutor, memoria de YARN disponible / executor-memory)

El hecho de establecer un número mayor de ejecutores no tiene por qué aumentar el rendimiento.  Debe tener en cuenta que al agregar más ejecutores pondrá una sobrecarga adicional a cada ejecutor que agregue, lo que podría reducir el rendimiento.  El valor de num-executors está limitado por los recursos del clúster.    

## <a name="example-calculation"></a>Cálculo de ejemplo

Supongamos que tiene actualmente un clúster compuesto de 8 nodos D4v2 que están ejecutando 2 aplicaciones, incluida la que se va a ejecutar.  

**Paso 1: Determine cuántas aplicaciones se ejecutan en el clúster**. Sabe que tiene 2 aplicaciones en el clúster, incluida la que se va a ejecutar.  

**Paso 2: Establezca executor-memory**. En este ejemplo, determinamos que 6 GB de memoria de ejecutor serán suficientes para trabajos con uso intensivo de E/S.  

executor-memory = 6 GB

**Paso 3: Establezca executor-cores**. Puesto que se trata de un trabajo con uso intensivo de E/S, podemos establecer el número de núcleos para cada ejecutor a 4.  Si establece los núcleos por ejecutor en un número mayor que 4 puede causar problemas de colección de elementos no utilizados.  

--executor-cores = 4

**Paso 4: Determine la cantidad de memoria YARN en clúster**. Navegamos a Ambari para averiguar que cada D4v2 tiene 25 GB de memoria YARN.  Dado que hay 8 nodos, la memoria YARN disponible se multiplica por 8.

- Memoria de YARN total = nodos * memoria de YARN * por nodo
- Memoria de YARN total = 8 nodos * 25 GB = 200 GB

**Paso 5: Calcule num-executors**. El parámetro num-executors se determina a partir de la restricción mínima de memoria y de CPU, dividida por el número de aplicaciones que están ejecutando en Spark.    

**Calcule la restricción de memoria**. La restricción de memoria se calcula dividiendo la memoria YARN total entre la memoria por ejecutor.

- Restricción de memoria = (memoria de YARN total / memoria del ejecutor) / número de aplicaciones
- Restricción de memoria = (200 GB / 6 GB)/ 2
- Restricción de memoria = 16 (redondeado)

**Calcule la restricción de CPU**. La restricción de la CPU se calcula dividiendo el total de núcleos YARN entre el número de núcleos por ejecutor.

- Núcleos de YARN = nodos del clúster * número de núcleos por nodo * 2
- Núcleos de YARN = 8 nodos * 8 núcleos por D14 * 2 = 128
- Restricción de CPU = (Total de núcleos de YARN / número de núcleos por ejecutor) / número de aplicaciones
- Restricción de CPU = (128 / 4) / 2
- Restricción de CPU = 16

**Establezca num-executors**

- num-executors = min (restricción de memoria, restricción de CPU)
- num-executors = Min (16, 16)
- num-executors = 16

