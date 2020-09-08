---
title: Copia de los datos de Synapse Link para Azure Cosmos DB en un grupo de SQL mediante Apache Spark
description: Cargue los datos en un dataframe de Spark, manténgalos y cárguelos en una tabla de un grupo de SQL
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: synapse-link
ms.date: 08/10/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: 88962d63519cfeb78be694c4f702b05ed4e7d3df
ms.sourcegitcommit: 271601d3eeeb9422e36353d32d57bd6e331f4d7b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88658376"
---
# <a name="copy-data-from-azure-cosmos-db-into-a-sql-pool-using-apache-spark"></a>Copia de datos de Azure Cosmos DB en un grupo de SQL mediante Apache Spark

Azure Synapse Link para Azure Cosmos DB permite a los usuarios ejecutar análisis casi en tiempo real con datos operativos en Azure Cosmos DB. Sin embargo, hay ocasiones en que es necesario agregar y enriquecer algunos datos para servir a los usuarios del almacenamiento de datos. Tanto el mantenimiento como la exportación de datos de Synapse Link se pueden realizar con unas pocas celdas de un cuaderno.

## <a name="prerequisites"></a>Requisitos previos
* [Aprovisione un área de trabajo de Synapse](../quickstart-create-workspace.md)con:
    * [Grupo de Spark](../quickstart-create-apache-spark-pool-studio.md)
    * [Grupo de SQL](../quickstart-create-sql-pool-studio.md)
* [Aprovisione una cuenta de Cosmos DB con un contenedor HTAP con datos](../../cosmos-db/configure-synapse-link.md)
* [Conecte el contenedor HTAP de Azure Cosmos DB al área de trabajo](./how-to-connect-synapse-link-cosmos-db.md)
* [Tenga la configuración adecuada para importar datos a un grupo de SQL desde Spark](../spark/synapse-spark-sql-pool-import-export.md)

## <a name="steps"></a>Pasos
En este tutorial, se conectará al almacén de análisis para que no haya ningún impacto en el almacén de transacciones (no consumirá ninguna unidad de solicitud). Seguiremos los pasos indicados a continuación:
1. Leer el contenedor HTAP de Cosmos DB en un dataframe de Spark.
2. Agregar los resultados a un dataframe nuevo.
3. Ingerir los datos en un grupo de SQL.

[![Pasos de Spark a SQL](../media/synapse-link-spark-to-sql/synapse-spark-to-sql.png)](../media/synapse-link-spark-to-sql/synapse-spark-to-sql.png#lightbox)

## <a name="data"></a>Datos
En ese ejemplo, se usa un contenedor HTAP denominado **RetailSales**. Forma parte de un servicio vinculado denominado **ConnectedData** y tiene el siguiente esquema:
* _rid: string (nullable = true)
* _ts: long (nullable = true)
* logQuantity: double (nullable = true)
* productCode: string (nullable = true)
* quantity: long (nullable = true)
* price: long (nullable = true)
* id: string (nullable = true)
* advertising: long (nullable = true)
* storeId: long (nullable = true)
* weekStarting: long (nullable = true)
* _etag: string (nullable = true)

Se agregarán las ventas (*de cantidad de*, *ingresos* [precio x cantidad] por *productCode* y *weekStarting*) para la elaboración de informes. Por último, se exportarán los datos en una tabla de grupo de SQL denominada **dbo.productsales**.

## <a name="configure-a-spark-notebook"></a>Configuración de un cuaderno de Spark
Cree un cuaderno de Spark con Scala and Spark (Scala) como lenguaje principal. Se usa la configuración predeterminada del cuaderno para la sesión.

## <a name="read-the-data-in-spark"></a>Lectura de datos en Spark
Lea el contenedor HTAP de Cosmos DB con Spark en un dataframe de la primera celda.

```java
val df_olap = spark.read.format("cosmos.olap").
    option("spark.synapse.linkedService", "ConnectedData").
    option("spark.cosmos.container", "RetailSales").
    load()
```

## <a name="aggregate-the-results-in-a-new-dataframe"></a>Agregación de los resultados a un dataframe nuevo

En la segunda celda, se ejecutan la transformación y los agregados necesarios para el nuevo dataframe antes de cargarlo en una base de datos del grupo de SQL.

```java
// Select relevant columns and create revenue
val df_olap_step1 = df_olap.select("productCode","weekStarting","quantity","price").withColumn("revenue",col("quantity")*col("price"))
//Aggregate revenue, quantity sold and avg. price by week and product ID
val df_olap_aggr = df_olap_step1.groupBy("productCode","weekStarting").agg(sum("quantity") as "Sum_quantity",sum("revenue") as "Sum_revenue").
    withColumn("AvgPrice",col("Sum_revenue")/col("Sum_quantity"))
```

## <a name="load-the-results-into-a-sql-pool"></a>Carga de los resultados en un grupo de SQL

En la tercera celda, se cargan los datos en un grupo de SQL. Se creará automáticamente una tabla externa temporal, un origen de datos externo y un formato de archivo externo que se eliminarán una vez que se haya completado el trabajo.

```java
df_olap_aggr.write.sqlanalytics("arnaudpool.dbo.productsales", Constants.INTERNAL)
```

## <a name="query-the-results-with-sql"></a>Consulta de los resultados con SQL

El resultado se puede mediante una sencilla consulta SQL, como el siguiente script de SQL:
```sql
SELECT  [productCode]
,[weekStarting]
,[Sum_quantity]
,[Sum_revenue]
,[AvgPrice]
 FROM [dbo].[productsales]
```

La consulta presentará los resultados siguientes en modo de gráfico: [![Pasos de Spark a SQL](../media/synapse-link-spark-to-sql/sql-script-spark-sql.png)](../media/synapse-link-spark-to-sql/sql-script-spark-sql.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes
* [Consulta de un almacén analítico de Azure Cosmos DB con Apache Spark](./how-to-query-analytical-store-spark.md)