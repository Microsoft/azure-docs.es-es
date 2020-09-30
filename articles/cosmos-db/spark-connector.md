---
title: Conexión de Apache Spark a Azure Cosmos DB
description: Aprenda sobre el conector de Azure Cosmos DB Spark que le permite conectar Apache Spark a Azure Cosmos DB.
author: tknandu
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/21/2019
ms.author: ramkris
ms.openlocfilehash: 633baafa7f6cb1bc134f07ce64ba98ebd4cb626c
ms.sourcegitcommit: 80b9c8ef63cc75b226db5513ad81368b8ab28a28
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90603057"
---
# <a name="accelerate-big-data-analytics-by-using-the-apache-spark-to-azure-cosmos-db-connector"></a>Aceleración de análisis de macrodatos mediante el conector de Apache Spark a Azure Cosmos DB

Puede ejecutar trabajos de [Spark](https://spark.apache.org/) con los datos almacenados en Azure Cosmos DB mediante el conector Spark de Cosmos DB. Cosmos se puede usar para procesar lotes y secuencias, y puede hacer de capa de servicio para el acceso de baja latencia.

Este conector se puede usar con [Azure Databricks](https://azure.microsoft.com/services/databricks) o con [Azure HDInsight](https://azure.microsoft.com/services/hdinsight/), que proporcionan clústeres de Spark administrados en Azure. En la tabla siguiente se muestran las versiones de Spark admitidas.

| Componente | Versión |
|---------|-------|
| Spark de Apache | 2.4.x, 2.3.x, 2.2.x y 2.1.x |
| Scala | 2.11 |
| Versión del runtime de Azure Databricks | > 3.4 |

> [!WARNING]
> Este conector admite la API Core (SQL) de Azure Cosmos DB.
> Si va a usar Cosmos DB para la API de MongoDB, use el [conector Spark de MongoDB](https://docs.mongodb.com/spark-connector/master/).
> Si va a usar Cosmos DB con Cassandra API, use el [conector Spark de Cassandra](https://github.com/datastax/spark-cassandra-connector).
>

## <a name="quickstart"></a>Guía de inicio rápido

* Siga los pasos de [introducción al SDK de Java](sql-api-async-java-get-started.md) para configurar una cuenta de Cosmos DB y rellenar algunos datos.
* Siga los pasos de [introducción a Azure Databricks](/azure/azure-databricks/quickstart-create-databricks-workspace-portal) para configurar un área de trabajo y un clúster de Azure Databricks.
* Ya puede crear cuadernos e importar la biblioteca de conectores de Cosmos DB. Vaya a [Trabajar con el conector de Cosmos DB](#bk_working_with_connector) para obtener más información sobre cómo configurar el área de trabajo.
* La siguiente sección tiene fragmentos de código sobre cómo leer y escribir con el conector.

### <a name="batch-reads-from-cosmos-db"></a>Lecturas de lotes de Cosmos DB

El siguiente fragmento de código muestra cómo crear una trama de datos de Spark para leer desde Cosmos DB en PySpark.

```python
# Read Configuration
readConfig = {
    "Endpoint": "https://doctorwho.documents.azure.com:443/",
    "Masterkey": "YOUR-KEY-HERE",
    "Database": "DepartureDelays",
    "Collection": "flights_pcoll",
    "query_custom": "SELECT c.date, c.delay, c.distance, c.origin, c.destination FROM c WHERE c.origin = 'SEA'" // Optional
}

# Connect via azure-cosmosdb-spark to create Spark DataFrame
flights = spark.read.format(
    "com.microsoft.azure.cosmosdb.spark").options(**readConfig).load()
flights.count()
```

Y el mismo fragmento de código, en Scala:

```scala
// Import Necessary Libraries
import com.microsoft.azure.cosmosdb.spark.schema._
import com.microsoft.azure.cosmosdb.spark._
import com.microsoft.azure.cosmosdb.spark.config.Config

// Read Configuration
val readConfig = Config(Map(
  "Endpoint" -> "https://doctorwho.documents.azure.com:443/",
  "Masterkey" -> "YOUR-KEY-HERE",
  "Database" -> "DepartureDelays",
  "Collection" -> "flights_pcoll",
  "query_custom" -> "SELECT c.date, c.delay, c.distance, c.origin, c.destination FROM c WHERE c.origin = 'SEA'" // Optional
))

// Connect via azure-cosmosdb-spark to create Spark DataFrame
val flights = spark.read.cosmosDB(readConfig)
flights.count()
```

### <a name="batch-writes-to-cosmos-db"></a>Escrituras de lotes en Cosmos DB

El siguiente fragmento de código muestra cómo escribir una trama de datos en Cosmos DB en PySpark.

```python
# Write configuration
writeConfig = {
    "Endpoint": "https://doctorwho.documents.azure.com:443/",
    "Masterkey": "YOUR-KEY-HERE",
    "Database": "DepartureDelays",
    "Collection": "flights_fromsea",
    "Upsert": "true"
}

# Write to Cosmos DB from the flights DataFrame
flights.write.mode("overwrite").format("com.microsoft.azure.cosmosdb.spark").options(
    **writeConfig).save()
```

Y el mismo fragmento de código, en Scala:

```scala
// Write configuration

val writeConfig = Config(Map(
  "Endpoint" -> "https://doctorwho.documents.azure.com:443/",
  "Masterkey" -> "YOUR-KEY-HERE",
  "Database" -> "DepartureDelays",
  "Collection" -> "flights_fromsea",
  "Upsert" -> "true"
))

// Write to Cosmos DB from the flights DataFrame
import org.apache.spark.sql.SaveMode
flights.write.mode(SaveMode.Overwrite).cosmosDB(writeConfig)
```

### <a name="streaming-reads-from-cosmos-db"></a>Lecturas de streaming desde Cosmos DB

El siguiente fragmento de código muestra cómo conectarse y leer desde la fuente de cambios de Azure Cosmos DB.

```python
# Read Configuration
readConfig = {
    "Endpoint": "https://doctorwho.documents.azure.com:443/",
    "Masterkey": "YOUR-KEY-HERE",
    "Database": "DepartureDelays",
    "Collection": "flights_pcoll",
    "ReadChangeFeed": "true",
    "ChangeFeedQueryName": "Departure-Delays",
    "ChangeFeedStartFromTheBeginning": "false",
    "InferStreamSchema": "true",
    "ChangeFeedCheckpointLocation": "dbfs:/Departure-Delays"
}


# Open a read stream to the Cosmos DB Change Feed via azure-cosmosdb-spark to create Spark DataFrame
changes = (spark
           .readStream
           .format("com.microsoft.azure.cosmosdb.spark.streaming.CosmosDBSourceProvider")
           .options(**readConfig)
           .load())
```
Y el mismo fragmento de código, en Scala:

```scala
// Import Necessary Libraries
import com.microsoft.azure.cosmosdb.spark.schema._
import com.microsoft.azure.cosmosdb.spark._
import com.microsoft.azure.cosmosdb.spark.config.Config

// Read Configuration
val readConfig = Config(Map(
  "Endpoint" -> "https://doctorwho.documents.azure.com:443/",
  "Masterkey" -> "YOUR-KEY-HERE",
  "Database" -> "DepartureDelays",
  "Collection" -> "flights_pcoll",
  "ReadChangeFeed" -> "true",
  "ChangeFeedQueryName" -> "Departure-Delays",
  "ChangeFeedStartFromTheBeginning" -> "false",
  "InferStreamSchema" -> "true",
  "ChangeFeedCheckpointLocation" -> "dbfs:/Departure-Delays"
))

// Open a read stream to the Cosmos DB Change Feed via azure-cosmosdb-spark to create Spark DataFrame
val df = spark.readStream.format(classOf[CosmosDBSourceProvider].getName).options(readConfig).load()
```

### <a name="streaming-writes-to-cosmos-db"></a>Escrituras de streaming en Cosmos DB

El siguiente fragmento de código muestra cómo escribir una trama de datos en Cosmos DB en PySpark.

```python
# Write configuration
writeConfig = {
    "Endpoint": "https://doctorwho.documents.azure.com:443/",
    "Masterkey": "YOUR-KEY-HERE",
    "Database": "DepartureDelays",
    "Collection": "flights_fromsea",
    "Upsert": "true",
    "WritingBatchSize": "500",
    "CheckpointLocation": "/checkpointlocation_write1"
}

# Write to Cosmos DB from the flights DataFrame
changeFeed = (changes
              .writeStream
              .format("com.microsoft.azure.cosmosdb.spark.streaming.CosmosDBSinkProvider")
              .outputMode("append")
              .options(**writeconfig)
              .start())
```

Y el mismo fragmento de código, en Scala:

```scala
// Write configuration

val writeConfig = Config(Map(
  "Endpoint" -> "https://doctorwho.documents.azure.com:443/",
  "Masterkey" -> "YOUR-KEY-HERE",
  "Database" -> "DepartureDelays",
  "Collection" -> "flights_fromsea",
  "Upsert" -> "true",
  "WritingBatchSize" -> "500",
  "CheckpointLocation" -> "/checkpointlocation_write1"
))

// Write to Cosmos DB from the flights DataFrame
df
.writeStream
.format(classOf[CosmosDBSinkProvider].getName)
.options(writeConfig)
.start()
```
Para obtener más fragmentos de código y ejemplos de un extremo a otro, vea [Jupyter](https://github.com/Azure/azure-cosmosdb-spark/tree/master/samples/notebooks).

## <a name="working-with-the-connector"></a><a name="bk_working_with_connector"></a> Trabajo con el conector

Puede crear el conector desde un origen en GitHub, o bien descargar los archivos uber-jar de Maven en los vínculos siguientes.

| Spark | Scala | La versión más reciente |
|---|---|---|
| 2.4.0 | 2.11 | [azure-cosmosdb-spark_lkg_version](https://aka.ms/CosmosDB_OLTP_Spark_2.4_LKG)
| 2.3.0 | 2.11 | [azure-cosmosdb-spark_2.3.0_2.11_1.3.3](https://search.maven.org/artifact/com.microsoft.azure/azure-cosmosdb-spark_2.3.0_2.11/1.3.3/jar)
| 2.2.0 | 2.11 | [azure-cosmosdb-spark_2.2.0_2.11_1.1.1](https://search.maven.org/#artifactdetails%7Ccom.microsoft.azure%7Cazure-cosmosdb-spark_2.2.0_2.11%7C1.1.1%7Cjar)
| 2.1.0 | 2.11 | [azure-cosmosdb-spark_2.1.0_2.11_1.2.2](https://search.maven.org/artifact/com.microsoft.azure/azure-cosmosdb-spark_2.1.0_2.11/1.2.2/jar)

### <a name="using-databricks-notebooks"></a>Uso de cuadernos de Databricks

Para crear una biblioteca mediante el área de trabajo de Databricks, siga las instrucciones en [Usar el conector Spark de Azure Cosmos DB](https://docs.azuredatabricks.net/spark/latest/data-sources/azure/cosmosdb-connector.html) de la guía de Azure Databricks.

> [!NOTE]
> La página **Usar el conector Spark de Azure Cosmos DB** no está actualizada en este momento. En lugar de descargar los seis archivos jar independientes en seis bibliotecas diferentes, puede descargar el archivo uber-jar de Maven en [azure-cosmosdb-spark_lkg_version](https://aka.ms/CosmosDB_OLTP_Spark_2.4_LKG) e instalar solamente esta biblioteca/archivo jar individual.
> 

### <a name="using-spark-cli"></a>Uso de spark-cli

Para trabajar con el conector mediante spark-cli (es decir, `spark-shell`, `pyspark`, `spark-submit`), puede usar el parámetro `--packages` con las [coordenadas de Maven](https://mvnrepository.com/artifact/com.microsoft.azure/azure-cosmosdb-spark_2.4.0_2.11) del conector.

```sh
spark-shell --master yarn --packages "com.microsoft.azure:azure-cosmosdb-spark_2.4.0_2.11:1.4.0"

```

### <a name="using-jupyter-notebooks"></a>Uso de Jupyter Notebooks

Si usa Jupyter Notebook en HDInsight, puede usar la celda `%%configure` de spark-magic para especificar las coordenadas Maven del conector.

```python
{ "name":"Spark-to-Cosmos_DB_Connector",
  "conf": {
    "spark.jars.packages": "com.microsoft.azure:azure-cosmosdb-spark_2.4.0_2.11:1.4.0",
    "spark.jars.excludes": "org.scala-lang:scala-reflect"
   }
   ...
}
```

> Tenga en cuenta que incluir `spark.jars.excludes` sirve expresamente para eliminar posibles conflictos entre el conector, Apache Spark y Livy.

### <a name="build-the-connector"></a>Creación del conector

Actualmente, este proyecto de conector usa `maven`, por lo que, para crear el conector sin dependencias, puede ejecutar:

```sh
mvn clean package
```

## <a name="working-with-our-samples"></a>Trabajo con nuestros ejemplos

El [repositorio de GitHub de Spark de Cosmos DB](https://github.com/Azure/azure-cosmosdb-spark) contiene los siguientes scripts y cuadernos de ejemplo que puede probar a usar.

* **Rendimiento vuelos en hora con Spark y Cosmos DB (Seattle)** [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/On-Time%20Flight%20Performance%20with%20Spark%20and%20Cosmos%20DB%20-%20Seattle.ipynb) | [html](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/On-Time%20Flight%20Performance%20with%20Spark%20and%20Cosmos%20DB%20-%20Seattle.html): conecte Spark a Cosmos DB mediante el servicio Jupyter Notebook de HDInsight para mostrar Spark SQL y GraphFrames, así como para predecir retrasos en los vuelos usando canalizaciones ML.
* **Origen de Twitter con Apache Spark y la fuente de cambios de Azure Cosmos DB**: [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/Twitter%20with%20Spark%20and%20Azure%20Cosmos%20DB%20Change%20Feed.ipynb) | [html](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/Twitter%20with%20Spark%20and%20Azure%20Cosmos%20DB%20Change%20Feed.html)
* **Uso de Apache Spark para consultar gráficos de Cosmos DB**: [ipynb](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/Using%20Apache%20Spark%20to%20query%20Cosmos%20DB%20Graphs.ipynb) | [html](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/notebooks/Using%20Apache%20Spark%20to%20query%20Cosmos%20DB%20Graphs.html)
* **[Conectar Azure Databricks a Azure Cosmos DB](https://docs.databricks.com/spark/latest/data-sources/azure/cosmosdb-connector.html)** mediante `azure-cosmosdb-spark`.  Aquí encontrará vinculada también una versión de Azure Databricks del [cuaderno de rendimiento de vuelos en hora](https://github.com/dennyglee/databricks/tree/master/notebooks/Users/denny%40databricks.com/azure-databricks).
* **[Arquitectura lambda con Azure Cosmos DB y HDInsight (Apache Spark)](https://github.com/Azure/azure-cosmosdb-spark/blob/master/samples/lambda/readme.md)** : puede usar Cosmos DB y Spark para reducir la sobrecarga operativa derivada de mantener canalizaciones de macrodatos.

## <a name="more-information"></a>Más información

Contamos con más información en la `azure-cosmosdb-spark` [wiki](https://github.com/Azure/azure-cosmosdb-spark/wiki), como la siguiente:

* [Guía de usuario del conector Spark de Azure Cosmos DB](https://github.com/Azure/azure-documentdb-spark/wiki/Azure-Cosmos-DB-Spark-Connector-User-Guide)
* [Ejemplos de agregaciones](https://github.com/Azure/azure-documentdb-spark/wiki/Aggregations-Examples)

### <a name="configuration-and-setup"></a>Instalación y configuración

* [Configuración del conector Spark](https://github.com/Azure/azure-cosmosdb-spark/wiki/Configuration-references)
* [Configuración del conector de Spark en Cosmos DB](https://github.com/Azure/azure-documentdb-spark/wiki/Spark-to-Cosmos-DB-Connector-Setup) (en fase de elaboración)
* [Configuración de DirectQuery de Power BI en Azure Cosmos DB a través de Apache Spark (HDI)](https://github.com/Azure/azure-cosmosdb-spark/wiki/Configuring-Power-BI-Direct-Query-to-Azure-Cosmos-DB-via-Apache-Spark-(HDI))

### <a name="troubleshooting"></a>Solución de problemas

* [Uso de agregados de Cosmos DB](https://github.com/Azure/azure-documentdb-spark/wiki/Troubleshooting:-Using-Cosmos-DB-Aggregates)
* [Problemas conocidos](https://github.com/Azure/azure-cosmosdb-spark/wiki/Known-Issues)

### <a name="performance"></a>Rendimiento

* [Consejos de rendimiento](https://github.com/Azure/azure-cosmosdb-spark/wiki/Performance-tips)
* [Ejecuciones de pruebas de consultas](https://github.com/Azure/azure-documentdb-spark/wiki/Query-Test-Runs)
* [Ejecuciones de pruebas de escritura](https://github.com/Azure/azure-cosmosdb-spark/wiki/Writing-Test-Runs)

### <a name="change-feed"></a>Fuente de cambios

* [Procesamiento de flujo de cambios mediante la fuente de cambios de Azure Cosmos DB y Apache Spark](https://github.com/Azure/azure-cosmosdb-spark/wiki/Stream-Processing-Changes-using-Azure-Cosmos-DB-Change-Feed-and-Apache-Spark)
* [Demos de fuentes de cambios](https://github.com/Azure/azure-cosmosdb-spark/wiki/Change-Feed-demos)
* [Demos de flujos estructurados](https://github.com/Azure/azure-cosmosdb-spark/wiki/Structured-Stream-demos)

### <a name="monitoring"></a>Supervisión

* [Supervisión de trabajos de Spark con Application Insights](https://github.com/Azure/azure-cosmosdb-spark/tree/2.3/samples/monitoring)

## <a name="next-steps"></a>Pasos siguientes

Si todavía no lo ha hecho, descargue el conector de Spark a Azure Cosmos DB del repositorio de GitHub [azure-cosmosdb-spark](https://github.com/Azure/azure-cosmosdb-spark). Explore los siguientes recursos adicionales en el repositorio:

* [Ejemplos de agregaciones](https://github.com/Azure/azure-cosmosdb-spark/wiki/Aggregations-Examples)
* [Cuadernos y script de ejemplo](https://github.com/Azure/azure-cosmosdb-spark/tree/master/samples)

Es posible que también quiera consultar la [guía de Apache Spark SQL, DataFrames y conjuntos de datos](https://spark.apache.org/docs/latest/sql-programming-guide.html) y el artículo [Apache Spark en Azure HDInsight](../hdinsight/spark/apache-spark-jupyter-spark-sql.md).
