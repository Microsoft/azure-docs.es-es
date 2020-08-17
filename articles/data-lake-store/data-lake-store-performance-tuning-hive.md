---
title: 'Optimización del rendimiento: Hive en Azure Data Lake Storage Gen1'
description: Directrices de optimización del rendimiento de Hive en HDInsight y Azure Data Lake Storage Gen1.
author: stewu
ms.service: data-lake-store
ms.topic: how-to
ms.date: 12/19/2016
ms.author: stewu
ms.openlocfilehash: 9e05773955fb79007a5d5899babf8680c69aa654
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87281725"
---
# <a name="performance-tuning-guidance-for-hive-on-hdinsight-and-azure-data-lake-storage-gen1"></a>Guía para la optimización del rendimiento de Hive en HDInsight y Azure Data Lake Storage Gen1

La configuración predeterminada se ha establecido para proporcionar un buen rendimiento en muchos casos de uso diferentes.  Para las consultas de uso intensivo de E/S, Hive se puede optimizar para obtener un mejor rendimiento con Azure Data Lake Storage Gen1.  

## <a name="prerequisites"></a>Requisitos previos

* **Una suscripción de Azure**. Consulte [Obtención de una versión de evaluación gratuita](https://azure.microsoft.com/pricing/free-trial/).
* **Cuenta de Data Lake Storage Gen1**. Para instrucciones sobre cómo crear una, consulte la [introducción a Azure Data Lake Storage Gen1](data-lake-store-get-started-portal.md)
* **Clúster de Azure HDInsight** con acceso a una cuenta de Data Lake Storage Gen1. Consulte [Creación de un clúster de HDInsight con Data Lake Storage Gen1](data-lake-store-hdinsight-hadoop-use-portal.md). Asegúrese de habilitar el Escritorio remoto para el clúster.
* **Ejecución de Hive en HDInsight**.  Para obtener información sobre cómo ejecutar trabajos de Hive en HDInsight, vea [Uso de Hive en HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-use-hive).
* **Guía para la optimización del rendimiento en Data Lake Storage Gen1**.  Para conocer los conceptos generales sobre rendimiento, consulte [Guía para la optimización del rendimiento de Data Lake Storage Gen1](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-performance-tuning-guidance).

## <a name="parameters"></a>Parámetros

Estos son los parámetros de configuración más importantes que se deben ajustar para mejorar el rendimiento de Data Lake Storage Gen1:

* **hive.tez.container.size**: la cantidad de memoria usada en cada tarea.

* **tez.grouping.min-size**: tamaño mínimo de cada asignador.

* **tez.grouping.min-size**: tamaño mínimo de cada asignador.

* **hive.exec.reducer.bytes.per.reducer**: tamaño de cada reductor.

**hive.tez.container.size**: el tamaño del contenedor determina cuánta memoria está disponible para cada tarea.  Esta es la entrada principal para controlar la simultaneidad en Hive.  

**tez.grouping.min-size**: este parámetro permite establecer el tamaño mínimo de cada asignador.  Si el número de asignadores que Tez elige es inferior al valor de este parámetro, Tez usará el valor establecido aquí.

**tez.grouping.max-size**: el parámetro le permite establecer el tamaño máximo de cada asignador.  Si el número de asignadores que elige Tez es superior al valor de este parámetro, Tez usará entonces el valor establecido aquí.

**hive.exec.reducer.bytes.per.reducer**: este parámetro establece el tamaño de cada reductor.  De forma predeterminada, cada reductor tiene 256 MB.  

## <a name="guidance"></a>Guía

**Set hive.exec.reducer.bytes.per.reducer**: el valor predeterminado funciona bien cuando los datos están descomprimidos.  En el caso de los datos que están comprimidos, debe reducir el tamaño del reductor.  

**Set hive.tez.container.size**: en cada nodo, la memoria se especifica mediante yarn.nodemanager.resource.memory-mb y se debe establecer correctamente en el clúster de HDI de forma predeterminada.  Para más información sobre la configuración de la memoria adecuada en YARN, consulte esta [entrada](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-hive-out-of-memory-error-oom).

Las cargas de trabajo que hacen un uso intensivo de E/S pueden beneficiarse de un mayor paralelismo disminuyendo el tamaño del contenedor de Tez. Con ello el usuario tiene más contenedores, lo que aumenta la simultaneidad.  Sin embargo, algunas consultas de Hive requieren una cantidad significativa de memoria (p. ej. MapJoin).  Si la tarea no tiene suficiente memoria, obtendrá una excepción de memoria insuficiente durante el tiempo de ejecución.  Si recibe excepciones de memoria insuficiente, debe aumentar la memoria.   

El número de tareas simultáneas en ejecución o con paralelismo estará enlazado por la cantidad total de memoria YARN.  El número de contenedores YARN determina cuántas tareas simultáneas se pueden ejecutar.  Para buscar la memoria YARN por nodo, puede ir a Ambari.  Vaya a YARN y vea la pestaña Configs (Configuraciones).  En esta ventana se muestra el tamaño de la memoria de YARN.  

> Total de memoria de YARN = nodos * memoria de YARN por nodo Número de contenedores de YARN = total de memoria de YARN/tamaño de contenedor de Tez

La clave para mejorar el rendimiento mediante Data Lake Storage Gen1 es aumentar la simultaneidad tanto como sea posible.  Tez calcula automáticamente el número de tareas que se deben crear por lo que no es necesario establecerlo.   

## <a name="example-calculation"></a>Cálculo de ejemplo

Supongamos que tiene un clúster D14 de 8 nodos.  

> Total de memoria de YARN = nodos * memoria de YARN por nodo Cantidad total de memoria de YARN= 8 nodos * 96 GB = 768 GB Número de contenedores de YARN = 768 GB/3072 MB = 256

## <a name="limitations"></a>Limitaciones

**Límite de Data Lake Storage Gen1** 

Si se alcanzan los límites de ancho de banda proporcionados por Data Lake Storage Gen1, es posible que experimente errores en las tareas. Esto puede identificarse observando los errores de limitación en los registros de tarea.  Puede reducir el paralelismo mediante un aumento del tamaño del contenedor de Tez.  Si necesita más simultaneidad para su trabajo, póngase en contacto con nosotros.

Para comprobar si le están aplicando limitaciones, debe habilitar el registro de depuración en el lado del cliente. Así es cómo debe hacerlo:

1. Coloque la siguiente propiedad en las propiedades de log4j en la configuración de Hive. Puede hacerlo desde la vista de Ambari: log4j.logger.com.microsoft.azure.datalake.store=DEBUG. Reinicie todos los nodos/servicios para que la configuración surta efecto.

2. Si se le están aplicando límites, verá el código de error HTTP 429 en el archivo de registro de Hive. El archivo de registro de Hive se encuentra en /tmp/&lt;user&gt;/hive.log

## <a name="further-information-on-hive-tuning"></a>Información adicional sobre la optimización de Hive

Estos son algunos blogs que le ayudará a optimizar las consultas de Hive:
* [Optimizar consultas de Hive para Hadoop en HDInsight](https://azure.microsoft.com/documentation/articles/hdinsight-hadoop-optimize-hive-query/)
* [Codificación del archivo de consulta de Hive en Azure HDInsight](https://docs.microsoft.com/archive/blogs/bigdatasupport/encoding-the-hive-query-file-in-azure-hdinsight)
* [Ignite talk on optimize Hive on HDInsight](https://channel9.msdn.com/events/Machine-Learning-and-Data-Sciences-Conference/Data-Science-Summit-2016/MSDSS25) (Acalorada charla sobre la optimización de Hive en HDInsight)
