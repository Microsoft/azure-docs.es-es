---
title: Consulta de un almacén analítico de Azure Cosmos DB (versión preliminar) con Apache Spark
description: Cómo consultar un almacén analítico de Azure Cosmos DB con Apache Spark para Azure Synapse Analytics
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: synapse-link
ms.date: 05/06/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: a7ee04c922e4373414dc27ed2b7c98be605280e5
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089114"
---
# <a name="query-azure-cosmos-db-analytical-store-preview-with-apache-spark-for-azure-synapse-analytics"></a>Consulta de un almacén analítico de Azure Cosmos DB (versión preliminar) con Apache Spark para Azure Synapse Analytics

En este artículo se ofrecen varios ejemplos de cómo puede interactuar con el almacén analítico desde los gestos de Synapse. Dichos gestos se muestran cuando se hace clic con el botón secundario en un contenedor. Con ellos, puede generar código rápidamente y adaptarlo a sus necesidades. También son ideales para detectar datos con un solo clic.

## <a name="load-to-dataframe"></a>Cargar en DataFrame

En este paso, leerá datos del almacén analítico de Azure Cosmos DB en un DataFrame de Spark. Se mostrarán 10 filas del DataFrame denominado ***df***. Una vez que los datos estén en DataFrame, puede realizar análisis adicionales.

Esta operación no afecta al almacén transaccional.

```python
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

df = spark.read.format("cosmos.olap")\
    .option("spark.synapse.linkedService", "INFERRED")\
    .option("spark.cosmos.container", "INFERRED")\
    .load()

df.show(10)
```

El gesto de código equivalente en **Scala** sería el código siguiente:
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

val df_olap = spark.read.format("cosmos.olap").
    option("spark.synapse.linkedService", "pySparkSamplesDb").
    option("spark.cosmos.container", "trafficSourceColl").
    load()
```

## <a name="create-spark-table"></a>Crear una tabla de Spark

En este gesto, creará una tabla de Spark que apunte al contenedor seleccionado. Esa operación no incurre en ningún movimiento de datos. Si decide eliminar esa tabla, el contenedor subyacente (y el almacén analítico correspondiente) no se verá afectado. 

Este escenario resulta práctico para volver a usar las tablas a través de herramientas de terceros y proporcionar accesibilidad a los datos para el tiempo de ejecución.

```sql
%%sql
-- To select a preferred list of regions in a multi-region Azure Cosmos DB account, add spark.cosmos.preferredRegions '<Region1>,<Region2>' in the config options

create table call_center using cosmos.olap options (
    spark.synapse.linkedService 'INFERRED',
    spark.cosmos.container 'INFERRED'
)
```

## <a name="write-dataframe-to-container"></a>Escribir un DataFrame en el contenedor

En este gesto, escribirá un DataFrame en un contenedor. Esta operación afectará al rendimiento transaccional y consumirá unidades de solicitud. El uso del rendimiento transaccional de Azure Cosmos DB es idóneo para las transacciones de escritura. Asegúrese de reemplazar **YOURDATAFRAME** por el DataFrame en el que quiera volver a escribir contenido.

```python
# Write a Spark DataFrame into an Azure Cosmos DB container
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")


YOURDATAFRAME.write.format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "INFERRED")\
    .option("spark.cosmos.container", "INFERRED")\
    .option("spark.cosmos.write.upsertEnabled", "true")\
    .mode('append')\
    .save()
```

El gesto de código equivalente en **Scala** sería el código siguiente:
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

import org.apache.spark.sql.SaveMode

df.write.format("cosmos.oltp").
    option("spark.synapse.linkedService", "pySparkSamplesDb").
    option("spark.cosmos.container", "trafficSourceColl"). 
    option("spark.cosmos.write.upsertEnabled", "true").
    mode(SaveMode.Overwrite).
    save()
```

## <a name="load-streaming-dataframe-from-container"></a>Cargar un DataFrame de streaming desde un contenedor
En este gesto, usará la funcionalidad de streaming de Spark para cargar datos de un contenedor en un DataFrame. Los datos se almacenarán en la cuenta principal del lago de datos (y en el sistema de archivos) que conectó al área de trabajo. 

Si no se creó la carpeta */localReadCheckpointFolder*, se creará automáticamente. Esta operación afectará al rendimiento transaccional de Azure Cosmos DB.

```python
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

dfStream = spark.readStream\
    .format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "INFERRED")\
    .option("spark.cosmos.container", "INFERRED")\
    .option("spark.cosmos.changeFeed.readEnabled", "true")\
    .option("spark.cosmos.changeFeed.startFromTheBeginning", "true")\
    .option("spark.cosmos.changeFeed.checkpointLocation", "/localReadCheckpointFolder")\
    .option("spark.cosmos.changeFeed.queryName", "streamQuery")\
    .load()
```

El gesto de código equivalente en **Scala** sería el código siguiente:
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

val dfStream = spark.readStream.
    format("cosmos.oltp").
    option("spark.synapse.linkedService", "pySparkSamplesDb").
    option("spark.cosmos.container", "trafficSourceColl").
    option("spark.cosmos.changeFeed.readEnabled", "true").
    option("spark.cosmos.changeFeed.startFromTheBeginning", "true").
    option("spark.cosmos.changeFeed.checkpointLocation", "/localReadCheckpointFolder").
    option("spark.cosmos.changeFeed.queryName", "streamTestRevin2").
    load()
```

## <a name="write-streaming-dataframe-to-container"></a>Escribir un DataFrame de streaming en un contenedor
En este gesto, escribirá un DataFrame de streaming en el contenedor de Azure Cosmos DB que seleccionó. Si no se creó la carpeta */localReadCheckpointFolder*, se creará automáticamente. Esta operación afectará al rendimiento transaccional de Azure Cosmos DB.

```python
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

streamQuery = dfStream\
        .writeStream\
        .format("cosmos.oltp")\
        .outputMode("append")\
        .option("checkpointLocation", "/localWriteCheckpointFolder")\
        .option("spark.synapse.linkedService", "INFERRED")\
        .option("spark.cosmos.container", "trafficSourceColl_sink")\
        .option("spark.cosmos.connection.mode", "gateway")\
        .start()

streamQuery.awaitTermination()
```

El gesto de código equivalente en **Scala** sería el código siguiente:
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

val query = dfStream.
            writeStream.
            format("cosmos.oltp").
            outputMode("append").
            option("checkpointLocation", "/localWriteCheckpointFolder").
            option("spark.synapse.linkedService", "pySparkSamplesDb").
            option("spark.cosmos.container", "test2").
            option("spark.cosmos.connection.mode", "gateway").
            start()

query.awaitTermination()
```
## <a name="next-steps"></a>Pasos siguientes

* [Información sobre compatibilidad entre Synapse y Azure Cosmos DB](./concept-synapse-link-cosmos-db-support.md)
* [Conexión a Synapse Link para Azure Cosmos DB](../quickstart-connect-synapse-link-cosmos-db.md)
