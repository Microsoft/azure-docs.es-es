---
title: Introducción al uso de Delta Lake de Linux Foundation en Apache Spark para Azure Synapse Analytics
description: Aprenda a usar Delta Lake en Apache Spark para Azure Synapse Analytics para crear y usar tablas con las propiedades ACID.
services: synapse-analytics
author: euangMS
ms.service: synapse-analytics
ms.reviewer: euang
ms.topic: overview
ms.subservice: spark
ms.date: 07/28/2020
ms.author: euang
ms.custom: devx-track-csharp
zone_pivot_groups: programming-languages-spark-all-minus-sql
ms.openlocfilehash: 655daeb0149228d78d5288b0e5d0d705a5743d28
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89008648"
---
# <a name="linux-foundation-delta-lake-overview"></a>Introducción a Delta Lake de Linux Foundation

Con el fin de proporcionar más claridad, este artículo se ha adaptado de su homólogo original, que se encuentra [aquí](https://docs.delta.io/latest/quick-start.html). Este artículo le ayuda a explorar rápidamente las principales características de [Delta Lake](https://delta.io). En el artículo se proporcionan fragmentos de código que muestran cómo leer tablas de Delta Lake y escribir en ellas desde consultas interactivas, por lotes y de streaming. Los fragmentos de código también están disponibles en un conjunto de cuadernos [PySpark aquí](https://github.com/Azure-Samples/Synapse/blob/master/Notebooks/PySpark/Hitchikers%20Guide%20to%20Delta%20Lake%20-%20Python.ipynb), [Scala aquí](https://github.com/Azure-Samples/Synapse/blob/master/Notebooks/Scala/Hitchikers%20Guide%20to%20Delta%20Lake%20-%20Scala.ipynb) y [C# aquí](https://github.com/Azure-Samples/Synapse/blob/master/Notebooks/Spark.NET%20C%23/Hitchikers%20Guide%20to%20Delta%20Lake%20-%20CSharp.ipynb)

Esto es lo que trataremos:

* Creación de una tabla
* Lectura de datos
* Actualización de los datos de una tabla
* Sobrescritura de los datos de una tabla
* Actualización condicional sin sobrescribir
* Lectura de las versiones anteriores de datos mediante Viaje en el tiempo
* Escritura de un flujo de datos en una tabla
* Lectura de un flujo de cambios de una tabla
* Soporte técnico de SQL

## <a name="configuration"></a>Configuración

Asegúrese de modificar el siguiente código según corresponda a su entorno.

:::zone pivot = "programming-language-python"

```python
import random

session_id = random.randint(0,1000000)
delta_table_path = "/delta/delta-table-{0}".format(session_id)

delta_table_path
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var sessionId = (new Random()).Next(10000000);
var deltaTablePath = $"/delta/delta-table-{sessionId}";

deltaTablePath
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val sessionId = scala.util.Random.nextInt(1000000)
val deltaTablePath = s"/delta/delta-table-$sessionId";
```

::: zone-end

El resultado es:

"/delta/delta-table-335323"

## <a name="create-a-table"></a>Creación de una tabla

Para crear una tabla de Delta Lake, escriba un DataFrame en el formato Delta. Puede cambiar el formato Parquet, CSV, JSON, etc., a Delta.

El siguiente código muestra cómo crear una tabla de Delta Lake mediante el esquema deducido de su DataFrame.

:::zone pivot = "programming-language-python"

```python
data = spark.range(0,5)
data.show()
data.write.format("delta").save(delta_table_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var data = spark.Range(0,5);
data.Show();
data.Write().Format("delta").Save(deltaTablePath);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val data = spark.range(0, 5)
data.show
data.write.format("delta").save(deltaTablePath)
```

::: zone-end

El resultado es:

| ID|
|---|
|  0|
|  1|
|  2|
|  3|
|  4|

## <a name="read-data"></a>Lectura de datos

Para leer los datos de su tabla de Delta Lake, especifique la ruta de acceso a los archivos y el formato Delta.

:::zone pivot = "programming-language-python"

```python
df = spark.read.format("delta").load(delta_table_path)
df.show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var df = spark.Read().Format("delta").Load(deltaTablePath);
df.Show()
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val df = spark.read.format("delta").load(deltaTablePath)
df.show()
```

::: zone-end

El resultado es:

| ID|
|---|
|  1|
|  3|
|  4|
|  0|
|  2|

El orden de los resultados es distinto del anterior, ya que antes de generar los resultados no había ningún orden especificado explícitamente.

## <a name="update-table-data"></a>Actualización de los datos de una tabla

Delta Lake admite varias operaciones para modificar tablas mediante las API estándar de DataFrame, este es una de las grandes mejoras que agrega el formato Delta. En el ejemplo siguiente se ejecuta un trabajo por lotes para sobrescribir los datos de la tabla.

:::zone pivot = "programming-language-python"

```python
data = spark.range(5,10)
data.write.format("delta").mode("overwrite").save(delta_table_path)
df.show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var data = spark.Range(5,10);
data.Write().Format("delta").Mode("overwrite").Save(deltaTablePath);
df.Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val data = spark.range(5, 10)
data.write.format("delta").mode("overwrite").save(deltaTablePath)
df.show()
```

::: zone-end

El resultado es:

| ID|
|---|
|  7|
|  8|
|  5|
|  9|
|  6|

Aquí se puede ver que los cinco registros se han actualizado para contener nuevos valores.

## <a name="save-as-catalog-tables"></a>Guardar como tablas de catálogo

Delta Lake puede escribir en tablas de catálogo administradas o externas.

:::zone pivot = "programming-language-python"

```python
data.write.format("delta").saveAsTable("ManagedDeltaTable")
spark.sql("CREATE TABLE ExternalDeltaTable USING DELTA LOCATION '{0}'".format(delta_table_path))
spark.sql("SHOW TABLES").show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
data.Write().Format("delta").SaveAsTable("ManagedDeltaTable");
spark.Sql($"CREATE TABLE ExternalDeltaTable USING DELTA LOCATION '{deltaTablePath}'");
spark.Sql("SHOW TABLES").Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
data.write.format("delta").saveAsTable("ManagedDeltaTable")
spark.sql(s"CREATE TABLE ExternalDeltaTable USING DELTA LOCATION '$deltaTablePath'")
spark.sql("SHOW TABLES").show
```

::: zone-end

El resultado es:

|database|         tableName|isTemporary|
|--------|------------------|-----------|
| default|externaldeltatable|      false|
| default| manageddeltatable|      false|

Con este código, ha creado una tabla en el catálogo a partir de una dataframe existente, que se denomina tabla administrada. Luego, ha definido una nueva tabla externa en el catálogo que utiliza una ubicación existente, que se denomina tabla externa. En la salida puede ver que ambas tablas, con independencia de cómo se hayan creado, se muestran en el catálogo.

Ahora puede ver las propiedades extendidas de las dos tablas

:::zone pivot = "programming-language-python"

```python
spark.sql("DESCRIBE EXTENDED ManagedDeltaTable").show(truncate=False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
spark.Sql("DESCRIBE EXTENDED ManagedDeltaTable").Show(truncate: 0);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
spark.sql("DESCRIBE EXTENDED ManagedDeltaTable").show(truncate=false)
```

::: zone-end

El resultado es:

|col_name                    |data_type                                                                                                    |comment|
|----------------------------|-------------------------------------------------------------------------------------------------------------|-------|
|id                          |bigint                                                                                                       |null   |
|                            |                                                                                                             |       |
|Información detallada de la tabla  |                                                                                                             |       |
|Base de datos                    |default                                                                                                      |       |
|Tabla                       |manageddeltatable                                                                                            |       |
|Propietario                       |trusted-service-user                                                                                         |       |
|Hora de creación                |Sábado, 25 de abril 00:35:34 UTC 2020                                                                                 |       |
|Último acceso                 |Jueves, 01 de enero 00:00:00 UTC 1970                                                                                 |       |
|Creado por                  |Spark 2.4.4.2.6.99.201-11401300                                                                              |       |
|Tipo                        |ADMINISTRADO                                                                                                      |       |
|Proveedor                    |delta                                                                                                        |       |
|Propiedades de tabla            |[transient_lastDdlTime=1587774934]                                                                           |       |
|Estadísticas                  |2407 bytes                                                                                                   |       |
|Location                    |abfss://data@<data lake>.dfs.core.windows.net/synapse/workspaces/<workspace name>/warehouse/manageddeltatable|       |
|Biblioteca Serde               |org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe                                                           |       |
|InputFormat                 |org.apache.hadoop.mapred.SequenceFileInputFormat                                                             |       |
|OutputFormat                |org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat                                                    |       |
|Propiedades de almacenamiento          |[serialization.format=1]                                                                                     |       |

:::zone pivot = "programming-language-python"

```python
spark.sql("DESCRIBE EXTENDED ExternalDeltaTable").show(truncate=False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
spark.Sql("DESCRIBE EXTENDED ExternalDeltaTable").Show(truncate: 0);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
spark.sql("DESCRIBE EXTENDED ExternalDeltaTable").show(truncate=false)
```

::: zone-end

El resultado es:

|col_name                    |data_type                                                             |comment|
|----------------------------|----------------------------------------------------------------------|-------|
|id                          |bigint                                                                |null   |
|                            |                                                                      |       |
|Información detallada de la tabla  |                                                                      |       |
|Base de datos                    |default                                                               |       |
|Tabla                       |externaldeltatable                                                    |       |
|Propietario                       |trusted-service-user                                                  |       |
|Hora de creación                |Sábado, 25 de abril 00:35:38 UTC 2020                                          |       |
|Último acceso                 |Jueves, 01 de enero 00:00:00 UTC 1970                                          |       |
|Creado por                  |Spark 2.4.4.2.6.99.201-11401300                                       |       |
|Tipo                        |EXTERNAL                                                              |       |
|Proveedor                    |DELTA                                                                 |       |
|Propiedades de tabla            |[transient_lastDdlTime=1587774938]                                    |       |
|Location                    |abfss://data@<data lake>.dfs.core.windows.net/delta/delta-table-587152|       |
|Biblioteca Serde               |org.apache.hadoop.hive.serde2.lazy.LazySimpleSerDe                    |       |
|InputFormat                 |org.apache.hadoop.mapred.SequenceFileInputFormat                      |       |
|OutputFormat                |org.apache.hadoop.hive.ql.io.HiveSequenceFileOutputFormat             |       |
|Propiedades de almacenamiento          |[serialization.format=1]                                              |       |

## <a name="conditional-update-without-overwrite"></a>Actualización condicional sin sobrescribir

Delta Lake proporciona API de programación para la actualización condicional, la eliminación y la combinación (que se conoce normalmente como upsert) de datos en las tablas.

:::zone pivot = "programming-language-python"

```python
from delta.tables import *
from pyspark.sql.functions import *

delta_table = DeltaTable.forPath(spark, delta_table_path)

delta_table.update(
  condition = expr("id % 2 == 0"),
  set = { "id": expr("id + 100") })
delta_table.toDF().show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
using Microsoft.Spark.Extensions.Delta;
using Microsoft.Spark.Extensions.Delta.Tables;
using Microsoft.Spark.Sql;
using static Microsoft.Spark.Sql.Functions;

var deltaTable = DeltaTable.ForPath(deltaTablePath);

deltaTable.Update(
  condition: Expr("id % 2 == 0"),
  set: new Dictionary<string, Column>(){{ "id", Expr("id + 100") }});
deltaTable.ToDF().Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
import io.delta.tables._
import org.apache.spark.sql.functions._

val deltaTable = DeltaTable.forPath(deltaTablePath)

// Update every even value by adding 100 to it
deltaTable.update(
  condition = expr("id % 2 == 0"),
  set = Map("id" -> expr("id + 100")))
deltaTable.toDF.show
```

::: zone-end

El resultado es:

| ID|
|---|
|106|
|108|
|  5|
|  7|
|  9|

Acaba de agregar 100 a cada identificador par.

:::zone pivot = "programming-language-python"

```python
delta_table.delete("id % 2 == 0")
delta_table.toDF().show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
deltaTable.Delete(condition: Expr("id % 2 == 0"));
deltaTable.ToDF().Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
deltaTable.delete(condition = expr("id % 2 == 0"))
deltaTable.toDF.show
```

::: zone-end

El resultado es:

| ID|
|---|
|  5|
|  7|
|  9|

Observe que se han eliminado todas las filas pares.

:::zone pivot = "programming-language-python"

```python
new_data = spark.range(0,20).alias("newData")

delta_table.alias("oldData")\
    .merge(new_data.alias("newData"), "oldData.id = newData.id")\
    .whenMatchedUpdate(set = { "id": lit("-1")})\
    .whenNotMatchedInsert(values = { "id": col("newData.id") })\
    .execute()

delta_table.toDF().show(100)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var newData = spark.Range(20).As("newData");

deltaTable
    .As("oldData")
    .Merge(newData, "oldData.id = newData.id")
    .WhenMatched()
        .Update(new Dictionary<string, Column>() {{"id", Lit("-1")}})
    .WhenNotMatched()
        .Insert(new Dictionary<string, Column>() {{"id", Col("newData.id")}})
    .Execute();

deltaTable.ToDF().Show(100);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val newData = spark.range(0, 20).toDF

deltaTable.as("oldData").
  merge(
    newData.as("newData"),
    "oldData.id = newData.id").
  whenMatched.
  update(Map("id" -> lit(-1))).
  whenNotMatched.
  insert(Map("id" -> col("newData.id"))).
  execute()

deltaTable.toDF.show()
```

::: zone-end

El resultado es:

| ID|
|---|
| 18|
| 15|
| 19|
|  2|
|  1|
|  6|
|  8|
|  3|
| -1|
| 10|
| 13|
|  0|
| 16|
|  4|
| -1|
| 12|
| 11|
| 14|
| -1|
| 17|

Aquí tiene una combinación de los datos existentes. A los datos existentes se les ha asignado el valor-1 en la ruta de acceso del código de actualización (WhenMatched). También se agregaron los nuevos datos que se crearon en al principio del fragmento a través de la ruta de acceso al código de inserción (WhenNotMatched).

### <a name="history"></a>Historial

Delta Lake tiene la capacidad de permitir examinar el historial de una tabla. Es decir, los cambios que se realizaron en la tabla Delta subyacente. En la celda siguiente se muestra lo sencillo que es inspeccionar el historial.

:::zone pivot = "programming-language-python"

```python
delta_table.history().show(20, 1000, False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
deltaTable.History().Show(20, 1000, false);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
deltaTable.history.show(false)
```

::: zone-end

El resultado es:

|version|          timestamp|userId|userName|operation|                                                operationParameters| trabajo|notebook|clusterId|readVersion|isolationLevel|isBlindAppend|
|-------|-------------------|------|--------|---------|-------------------------------------------------------------------|----|--------|---------|-----------|--------------|-------------|
|      4|25-04-2020 00:36:27|  null|    null|    MERGE|                       [predicate -> (oldData.`ID` = newData.`ID`)]|null|    null|     null|          3|          null|        false|
|      3|25-04-2020 00:36:08|  null|    null|   Delete|[predicate -> ["((`ID` % CAST(2 AS BIGINT)) = CAST(0 AS BIGINT))"]]|null|    null|     null|          2|          null|        false|
|      2|25-04-2020 00:35:51|  null|    null|   UPDATE| [predicate -> ((ID#744L % cast(2 as bigint)) = cast(0 as bigint))]|null|    null|     null|          1|          null|        false|
|      1|25-04-2020 00:35:05|  null|    null|    WRITE|                             [mode -> Overwrite, partitionBy -> []]|null|    null|     null|          0|          null|        false|
|      0|25-04-2020 00:34:34|  null|    null|    WRITE|                         [mode -> ErrorIfExists, partitionBy -> []]|null|    null|     null|       null|          null|         true|

Aquí puede ver todas las modificaciones realizadas en los fragmentos de código anteriores.

## <a name="read-older-versions-of-data-using-time-travel"></a>Lectura de las versiones anteriores de datos mediante Viaje en el tiempo

La característica Viaje en el tiempo permite consultar instantáneas anteriores de la tabla de Delta Lake. Si desea acceder a los datos que ha sobrescrito, puede consultar una instantánea de la tabla antes de sobrescribir el primer conjunto de datos mediante la opción versionAsOf.

Una vez que ejecute la celda siguiente, debería ver el primer conjunto de datos de antes de que se sobrescribiera. Time Travel es una característica muy eficaz que aprovecha la eficacia del registro de transacciones de Delta Lake para acceder a los datos que ya no están en la tabla. La eliminación de la opción de la versión 0 (o la especificación de la versión 1) le permitiría volver a ver los datos más recientes. Para más información, consulte [Consulta de una instantánea anterior de una tabla](https://docs.delta.io/latest/delta-batch.html#deltatimetravel).

:::zone pivot = "programming-language-python"

```python
df = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
df.show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var df = spark.Read().Format("delta").Option("versionAsOf", 0).Load(deltaTablePath);
df.Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val df = spark.read.format("delta").option("versionAsOf", 0).load(deltaTablePath)
df.show()
```

::: zone-end

El resultado es:

| ID|
|---|
|  0|
|  1|
|  4|
|  3|
|  2|

Aquí puede ver que ha vuelto a la versión más antigua de los datos.

## <a name="write-a-stream-of-data-to-a-table"></a>Escritura de un flujo de datos en una tabla

También puede escribir en una tabla de Delta Lake mediante Structured Streaming de Spark. El registro de transacciones de Delta Lake garantiza un procesamiento exactamente una vez, incluso cuando haya otras secuencias o consultas por lotes que se ejecutan simultáneamente en la tabla. De forma predeterminada, las secuencias se ejecutan en modo Anexar, que agrega nuevos registros a la tabla.

Para más información sobre la integración de Delta Lake en Structured Streaming, consulte [Lecturas y escrituras de streaming de tablas](https://docs.delta.io/latest/delta-streaming.html).

En las celdas siguientes, esto es lo que hacemos:

* En la celda 30, mostrar los datos recién anexados
* En la celda 31, comprobar el historial
* En la celda 32, detener el trabajo de Structured Streaming
* En la celda 33, inspeccionar el historial <-- Observará que los anexos se han detenido

En primer lugar, va a configurar un sencillo trabajo de Spark Streaming para generar una secuencia y hacer que el trabajo escriba en una tabla de Delta.

:::zone pivot = "programming-language-python"

```python
streaming_df = spark.readStream.format("rate").load()
stream = streaming_df\
    .selectExpr("value as id")\
    .writeStream\
    .format("delta")\
    .option("checkpointLocation", "/tmp/checkpoint-{0}".format(session_id))\
    .start(delta_table_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var streamingDf = spark.ReadStream().Format("rate").Load();
var stream = streamingDf.SelectExpr("value as id").WriteStream().Format("delta").Option("checkpointLocation", $"/tmp/checkpoint-{sessionId}").Start(deltaTablePath);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val streamingDf = spark.readStream.format("rate").load()
val stream = streamingDf.select($"value" as "id").writeStream.format("delta").option("checkpointLocation", s"/tmp/checkpoint-$sessionId").start(deltaTablePath)
```

::: zone-end

## <a name="read-a-stream-of-changes-from-a-table"></a>Lectura de un flujo de cambios de una tabla

Mientras el flujo se escribe en la tabla de Delta Lake, también puede leer dicha tabla como un origen de streaming. Por ejemplo, puede iniciar otra consulta de streaming que imprima todos los cambios realizados en la tabla de Delta Lake.

:::zone pivot = "programming-language-python"

```python
delta_table.toDF().sort(col("id").desc()).show(100)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
deltaTable.ToDF().Sort(Col("id").Desc()).Show(100);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
deltaTable.toDF.sort($"id".desc).show
```

::: zone-end

El resultado es:

| ID|
|---|
| 19|
| 18|
| 17|
| 16|
| 15|
| 14|
| 13|
| 12|
| 11|
| 10|
|  8|
|  6|
|  4|
|  3|
|  2|
|  1|
|  0|
| -1|
| -1|
| -1|

:::zone pivot = "programming-language-python"

```python
delta_table.history().drop("userId", "userName", "job", "notebook", "clusterId", "isolationLevel", "isBlindAppend").show(20, 1000, False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
deltaTable.History().Drop("userId", "userName", "job", "notebook", "clusterId", "isolationLevel", "isBlindAppend").Show(20, 1000, false);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
deltaTable.history.show
```

::: zone-end

El resultado es:

|version|          timestamp|       operation|                                                                  operationParameters|readVersion|
|-------|-------------------|----------------|-------------------------------------------------------------------------------------|-----------|
|      5|25-04-2020 00:37:09|STREAMING UPDATE|[outputMode -> Append, queryId -> d26b4f8a-7e5a-44f2-a5fb-23a7bd02aef7, epochId -> 0]|          4|
|      4|25-04-2020 00:36:27|           MERGE|                                         [predicate -> (oldData.`id` = newData.`id`)]|          3|
|      3|25-04-2020 00:36:08|          DELETE|                  [predicate -> ["((`id` % CAST(2 AS BIGINT)) = CAST(0 AS BIGINT))"]]|          2|
|      2|25-04-2020 00:35:51|          UPDATE|                   [predicate -> ((id#744L % cast(2 as bigint)) = cast(0 as bigint))]|          1|
|      1|25-04-2020 00:35:05|           WRITE|                                               [mode -> Overwrite, partitionBy -> []]|          0|
|      0|25-04-2020 00:34:34|           WRITE|                                           [mode -> ErrorIfExists, partitionBy -> []]|       null|

Aquí va a quitar algunas de las columnas menos interesantes para simplificar la experiencia de visualización del historial.

:::zone pivot = "programming-language-python"

```python
stream.stop()
delta_table.history().drop("userId", "userName", "job", "notebook", "clusterId", "isolationLevel", "isBlindAppend").show(100, 1000, False)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
stream.Stop();
deltaTable.History().Drop("userId", "userName", "job", "notebook", "clusterId", "isolationLevel", "isBlindAppend").Show(100, 1000, false);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
stream.stop
deltaTable.history.show
```

El resultado es:

|version|          timestamp|       operation|                                                                  operationParameters|readVersion|
|-------|-------------------|----------------|-------------------------------------------------------------------------------------|-----------|
|      5|25-04-2020 00:37:09|STREAMING UPDATE|[outputMode -> Append, queryId -> d26b4f8a-7e5a-44f2-a5fb-23a7bd02aef7, epochId -> 0]|          4|
|      4|25-04-2020 00:36:27|           MERGE|                                         [predicate -> (oldData.`id` = newData.`id`)]|          3|
|      3|25-04-2020 00:36:08|          Delete|                  [predicate -> ["((`id` % CAST(2 AS BIGINT)) = CAST(0 AS BIGINT))"]]|          2|
|      2|25-04-2020 00:35:51|          UPDATE|                   [predicate -> ((id#744L % cast(2 as bigint)) = cast(0 as bigint))]|          1|
|      1|25-04-2020 00:35:05|           WRITE|                                               [mode -> Overwrite, partitionBy -> []]|          0|
|      0|25-04-2020 00:34:34|           WRITE|                                           [mode -> ErrorIfExists, partitionBy -> []]|       null|

::: zone-end

## <a name="convert-parquet-to-delta"></a>Conversión de Parquet en Delta

Puede realizar una conversión en contexto desde el formato Parquet a Delta.

Aquí va a probar si la tabla existente está en formato Delta.
:::zone pivot = "programming-language-python"

```python
parquet_path = "/parquet/parquet-table-{0}".format(session_id)
data = spark.range(0,5)
data.write.parquet(parquet_path)
DeltaTable.isDeltaTable(spark, parquet_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var parquetPath = $"/parquet/parquet-table-{sessionId}";
var data = spark.Range(0,5);
data.Write().Parquet(parquetPath);
DeltaTable.IsDeltaTable(parquetPath)
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val parquetPath = s"/parquet/parquet-table-$sessionId"
val data = spark.range(0,5)
data.write.parquet(parquetPath)
DeltaTable.isDeltaTable(parquetPath)
```

::: zone-end

El resultado es:

Falso

Ahora va a convertir los datos al formato Delta y a comprobar que la operación ha funcionado.

:::zone pivot = "programming-language-python"

```python
DeltaTable.convertToDelta(spark, "parquet.`{0}`".format(parquet_path))
DeltaTable.isDeltaTable(spark, parquet_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
DeltaTable.ConvertToDelta(spark, $"parquet.`{parquetPath}`");
DeltaTable.IsDeltaTable(parquetPath)
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
DeltaTable.convertToDelta(spark, s"parquet.`$parquetPath`")
DeltaTable.isDeltaTable(parquetPath)
```

::: zone-end

El resultado es:

Verdadero

## <a name="sql-support"></a>Soporte técnico de SQL

Delta admite comandos de la utilidad de tablas a través de SQL. SQL se puede usar para:

* Obtener el historial de una instancia de DeltaTable.
* Crear el vacío en una instancia de DeltaTable.
* Convertir un archivo Parquet en Delta.

:::zone pivot = "programming-language-python"

```python
spark.sql("DESCRIBE HISTORY delta.`{0}`".format(delta_table_path)).show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
spark.Sql($"DESCRIBE HISTORY delta.`{deltaTablePath}`").Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
spark.sql(s"DESCRIBE HISTORY delta.`$deltaTablePath`").show()
```

::: zone-end

El resultado es:

|version|          timestamp|userId|userName|       operation| operationParameters| trabajo|notebook|clusterId|readVersion|isolationLevel|isBlindAppend|
|-------|-------------------|------|--------|----------------|--------------------|----|--------|---------|-----------|--------------|-------------|
|      5|25-04-2020 00:37:09|  null|    null|STREAMING UPDATE|[outputMode -> Ap...|null|    null|     null|          4|          null|         true|
|      4|25-04-2020 00:36:27|  null|    null|           MERGE|[predicate -> (ol...|null|    null|     null|          3|          null|        false|
|      3|25-04-2020 00:36:08|  null|    null|          Delete|[predicate -> ["(...|null|    null|     null|          2|          null|        false|
|      2|25-04-2020 00:35:51|  null|    null|          UPDATE|[predicate -> ((i...|null|    null|     null|          1|          null|        false|
|      1|25-04-2020 00:35:05|  null|    null|           WRITE|[mode -> Overwrit...|null|    null|     null|          0|          null|        false|
|      0|25-04-2020 00:34:34|  null|    null|           WRITE|[mode -> ErrorIfE...|null|    null|     null|       null|          null|         true|

:::zone pivot = "programming-language-python"

```python
spark.sql("VACUUM delta.`{0}`".format(delta_table_path)).show()
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
spark.Sql($"VACUUM delta.`{deltaTablePath}`").Show();
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
spark.sql(s"VACUUM delta.`$deltaTablePath`").show()
```

::: zone-end

El resultado es:

|                path|
|--------------------|
|abfss://data@arca...|

Ahora va a comprobar que una tabla no está en formato Delta y, luego, va a convertirla en ese formato mediante Spark SQL y a confirmar que se ha convertido correctamente.

:::zone pivot = "programming-language-python"

```python
parquet_id = random.randint(0,1000)
parquet_path = "/parquet/parquet-table-{0}-{1}".format(session_id, parquet_path)
data = spark.range(0,5)
data.write.parquet(parquet_path)
DeltaTable.isDeltaTable(spark, parquet_path)
spark.sql("CONVERT TO DELTA parquet.`{0}`".format(parquet_path))
DeltaTable.isDeltaTable(spark, parquet_path)
```

::: zone-end

:::zone pivot = "programming-language-csharp"

```csharp
var parquetId =  (new Random()).Next(10000000);
var parquetPath = $"/parquet/parquet-table-{sessionId}-{parquetId}";
var data = spark.Range(0,5);
data.Write().Parquet(parquetPath);
DeltaTable.IsDeltaTable(parquetPath);
spark.Sql($"CONVERT TO DELTA parquet.`{parquetPath}`");
DeltaTable.IsDeltaTable(parquetPath);
```

::: zone-end

:::zone pivot = "programming-language-scala"

```scala
val parquetId = scala.util.Random.nextInt(1000)
val parquetPath = s"/parquet/parquet-table-$sessionId-$parquetId"
val data = spark.range(0,5)
data.write.parquet(parquetPath)
DeltaTable.isDeltaTable(parquetPath)
spark.sql(s"CONVERT TO DELTA parquet.`$parquetPath`")
DeltaTable.isDeltaTable(parquetPath)
```

::: zone-end

El resultado es:

Verdadero

Para ver la documentación completa, consulte la [página de la documentación de Delta Lake](https://docs.delta.io/latest/delta-intro.html).

Para obtener más información, consulte [Proyecto Delta Lake](https://github.com/delta-io/delta).

## <a name="next-steps"></a>Pasos siguientes

* [Documentación de .NET para Apache Spark](/dotnet/spark?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
* [Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics)
