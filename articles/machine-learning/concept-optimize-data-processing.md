---
title: Optimización del procesamiento de datos
titleSuffix: Azure Machine Learning
description: Obtenga información sobre los procedimientos recomendados para optimizar las velocidades de procesamiento de datos y qué integraciones admite Azure Machine Learning para el procesamiento de datos a gran escala.
services: machine-learning
ms.service: machine-learning
ms.author: sgilley
author: sdgilley
ms.subservice: core
ms.reviewer: nibaccam
ms.topic: conceptual
ms.date: 06/26/2020
ms.openlocfilehash: 6bb85ada5ab1cd443d47ed85024b45d98354e97f
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500970"
---
# <a name="optimize-data-processing-with-azure-machine-learning"></a>Optimización del procesamiento de datos con Azure Machine Learning

En este artículo, obtendrá información sobre los procedimientos recomendados para ayudarle a optimizar la velocidad de procesamiento de datos de forma local y a gran escala.

Azure Machine Learning se integra en paquetes de código abierto y marcos para el procesamiento de datos. Al usar estas integraciones y aplicar las recomendaciones de procedimiento recomendado de este artículo, podrá mejorar las velocidades de procesamiento de datos tanto de forma local como a gran escala.

## <a name="parquet-and-csv-file-formats"></a>Formatos de archivo Parquet y CSV

Los archivos de valores separados por comas (CSV) son formatos de archivo comunes para el procesamiento de datos. Sin embargo, se recomiendan los formatos de archivo Parquet para las tareas de aprendizaje automático.

Los [archivos Parquet](https://parquet.apache.org/) almacenan los datos en un formato de columnas binarias. Este formato es útil si se necesita dividir los datos en varios archivos. Además, este formato le permite dirigirse a los campos pertinentes de los experimentos de aprendizaje automático. Para reducir la carga de datos, en lugar de tener que leer en un archivo de datos de 20 GB, puede seleccionar las columnas necesarias para entrenar el modelo de aprendizaje automático. Los archivos Parquet también se pueden comprimir para minimizar la capacidad de procesamiento necesaria y ocupar menos espacio.

Los archivos CSV se usan normalmente para importar y exportar datos, ya que son fáciles de editar y leer en Excel. Los datos de CSV se almacenan como cadenas en un formato basado en filas, y los archivos se pueden comprimir para reducir las cargas de transferencia de datos. Los archivos CSV sin comprimir pueden expandirse aproximadamente de 2 a 10 veces, y los archivos CSV comprimidos pueden aumentar aún más. De modo que un archivo CSV de 5 GB en la memoria se expande fácilmente hasta los 8 GB de RAM que tenga en la máquina. Este comportamiento de compresión puede aumentar la latencia de transferencia de datos, lo que no es ideal si tiene grandes cantidades de datos para procesar. 

## <a name="pandas-dataframe"></a>Dataframe de Pandas

Los [dataframes de Pandas](https://pandas.pydata.org/pandas-docs/stable/getting_started/overview.html) se usan normalmente para la manipulación y el análisis de datos. `Pandas` funciona bien para tamaños de datos inferiores a 1 GB, pero los tiempos de procesamiento de los dataframes `pandas` se ralentizan cuando los tamaños de archivo se aproximan a 1 GB. Esta ralentización se debe a que el tamaño de los datos en el almacenamiento no es el mismo que en un dataframe. Por ejemplo, los datos de los archivos CSV se pueden expandir hasta 10 veces en un dataframe, por lo que un archivo CSV de 1 GB puede convertirse en 10 GB en un dataframe.

`Pandas` tiene un único subproceso, lo que significa que las operaciones se realizan una a la vez en una sola CPU. Puede paralelizar fácilmente las cargas de trabajo a varias CPU virtuales en una única instancia de proceso de Azure Machine Learning con paquetes como [Modin](https://modin.readthedocs.io/en/latest/) que encapsulan `Pandas` mediante un back-end distribuido.

Para paralelizar las tareas con `Modin` y [Dask](https://dask.org), solo tiene que cambiar esta línea de código de `import pandas as pd` a `import modin.pandas as pd`.

## <a name="dataframe-out-of-memory-error"></a>Dataframe: error de memoria insuficiente 

Normalmente se produce un error de *memoria insuficiente* cuando el dataframe se expande más allá de la RAM disponible en la máquina. Este concepto también se aplica a un marco distribuido, como `Modin` o `Dask`.  Es decir, la operación intenta cargar el dataframe en la memoria de cada nodo del clúster, pero no hay suficiente RAM disponible para ello.

Una solución es aumentar la RAM para ajustarla al dataframe en la memoria. Se recomienda que el tamaño de proceso y la capacidad de procesamiento tengan una RAM del doble del tamaño. Por lo tanto, si el dataframe es de 10 GB, use un destino de proceso con más de 20 GB de RAM para asegurarse de que el dataframe tenga suficiente espacio en la memoria y se puede procesar. 

En el caso de varias CPU virtuales, vCPU, tenga en cuenta que querrá que una de las particiones quepa fácilmente en la memoria RAM que cada vCPU pueda tener en la máquina. Es decir, si tiene 4 vCPU con 16 GB de RAM, querrá más de dataframes de 2 GB por cada vCPU.

### <a name="minimize-cpu-workloads"></a>Minimización de las cargas de trabajo de CPU

Si no puede agregar más RAM a la máquina, puede aplicar las técnicas siguientes para ayudar a minimizar las cargas de trabajo de CPU y optimizar los tiempos de procesamiento. Estas recomendaciones corresponden a sistemas tanto individuales como distribuidos.

Técnica | Descripción
----|----
Compresión | Use una representación diferente para los datos, de forma que use menos memoria y no afecte significativamente a los resultados del cálculo.<br><br>*Ejemplo*: En lugar de almacenar las entradas como cadena con unos 10 bytes o más por cada entrada, guárdelas como valor booleano, true o false, que puede almacenar en 1 byte.
Fragmentación | Cargue datos en la memoria en subconjuntos (fragmentos), y procesar los datos de a un subconjunto a la vez, o varios subconjuntos en paralelo. Este método funciona mejor si tiene que procesar todos los datos, pero no necesita cargar en la memoria todos los datos a la vez. <br><br>*Ejemplo*: En lugar de procesar a la vez los datos de todo un año, cargue y procese los datos un mes cada vez.
Indización | Aplique y usa un índice, un resumen que le indica dónde encontrar los datos que le interesan. La indexación es útil cuando solo necesita usar un subconjunto de los datos, en lugar del conjunto completo.<br><br>*Ejemplo*: Si tiene datos de ventas de todo un año ordenados por mes, un índice le ayuda a buscar rápidamente el mes deseado que quiere procesar.

## <a name="scale-data-processing"></a>Escalado del procesamiento de datos

Si las recomendaciones anteriores no son suficientes y no puede obtener una máquina virtual que se ajuste a sus datos, puede: 

* Usar un marco, como `Spark` o `Dask`, para procesar los datos de "memoria insuficiente" En esta opción, el dataframe se carga en la RAM partición por partición y se procesa, y el resultado final se recopila al final.  

* Escalar horizontalmente a un clúster mediante un marco distribuido. En esta opción, las cargas de procesamiento de datos se dividen y procesan en varias CPU que funcionan en paralelo, y el resultado final se recopila al final.

### <a name="recommended-distributed-frameworks"></a>Marcos distribuidos recomendados

En la tabla siguiente se recomiendan los marcos distribuidos que se integran en Azure Machine Learning en función del tamaño de los datos o de la preferencia del código.

Experiencia o tamaño de los datos | Recomendación
------|------
Si está familiarizado con `Pandas`| Dataframe `Modin` o `Dask`
Si prefiere `Spark` | `PySpark`
Para menos de 1 GB | `Pandas` localmente **o** una instancia de proceso de Azure Machine Learning
Para datos de más de 10 GB| Muévase a un clúster mediante `Ray`, `Dask`o `Spark`

Puede crear clústeres de `Dask` en el clúster de proceso de Azure Machine Learning con el paquete [dask-cloudprovider](https://cloudprovider.dask.org/en/latest/#azure). O bien, puede ejecutar `Dask` localmente en una instancia de proceso.

## <a name="next-steps"></a>Pasos siguientes

* [Opciones de ingesta de datos con Azure Machine Learning](concept-data-ingestion.md).
* [Creación y registro de conjuntos de datos](how-to-create-register-datasets.md)
