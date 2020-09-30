---
title: Interacción con Azure Cosmos DB mediante Apache Spark en Azure Synapse Link (versión preliminar)
description: Cómo interactuar con Azure Cosmos DB mediante Apache Spark en Azure Synapse Link
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: synapse-link
ms.date: 09/15/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: 663c07795926b17eb42ff185ca248454c5bc459c
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90881843"
---
# <a name="interact-with-azure-cosmos-db-using-apache-spark-in-azure-synapse-link-preview"></a>Interacción con Azure Cosmos DB mediante Apache Spark en Azure Synapse Link (versión preliminar)

En este artículo, aprenderá a interactuar con Azure Cosmos DB mediante Synapse Apache Spark. Gracias a su compatibilidad completa con Scala, Python, SparkSQL y C#, Synapse Apache Spark es fundamental para los escenarios de análisis, ingeniería de datos, ciencia de datos y exploración de datos en [Azure Synapse Link para Azure Cosmos DB](../../cosmos-db/synapse-link.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json).

Al interactuar con Azure Cosmos DB, se admiten las siguientes funcionalidades:
* Apache Spark de Synapse permite analizar los datos de los contenedores de Azure Cosmos DB que están habilitados con Azure Synapse Link casi en tiempo real, sin afectar al rendimiento de las cargas de trabajo transaccionales. Las dos opciones siguientes están disponibles para consultar el [almacén analítico](../../cosmos-db/analytical-store-introduction.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) de Azure Cosmos DB desde Spark:
    + Cargar a DataFrame de Spark
    + Crear una tabla de Spark
* Apache Spark de Synapse también le permite ingerir datos en Azure Cosmos DB. Es importante tener en cuenta que los datos siempre se ingieren en contenedores de Azure Cosmos DB a través del almacén transaccional. Cuando el Synapse Link está habilitado, las nuevas inserciones, actualizaciones y eliminaciones se sincronizan automáticamente con el almacén analítico.
* Apache Spark de Synapse también admite Spark Structured Streaming con Azure Cosmos DB como origen y receptor. 

Las secciones siguientes le guiarán por la sintaxis de las funcionalidades anteriores. Los gestos en el área de trabajo de Azure Synapse Analytics están diseñados para proporcionarle una experiencia de introducción fácil de usar. Los gestos se muestran al hacer clic con el botón derecho en un contenedor de Azure Cosmos DB en la pestaña **Datos** del área de trabajo de Synapse. Con ellos, puede generar código rápidamente y adaptarlo a sus necesidades. También son ideales para detectar datos con un solo clic.

## <a name="query-azure-cosmos-db-analytical-store"></a>Consulta al almacén analítico de Azure Cosmos DB

Antes de obtener información acerca de los dos métodos posibles para consultar al almacén analítico de Azure Cosmos DB, así como para cargar a un DataFrame de Spark y crear una tabla de Spark, merece la pena explorar las diferencias entre experiencias para que pueda elegir la opción que mejor se adapte a sus necesidades.

La diferencia en la experiencia radica en si los cambios de datos subyacentes en el contenedor de Azure Cosmos DB se deben reflejar automáticamente en el análisis realizado en Spark. Cuando se registra un DataFrame de Spark o se crea una tabla de Spark en el almacén analítico de un contenedor, los metadatos de la instantánea de datos actual en el almacén analítico se capturan en Spark para un aplicación eficaz del análisis subsiguiente. Es importante tener en cuenta que, ya que Spark usa una directiva de evaluación diferida, a menos que se invoque una acción en el DataFrame de Spark o se ejecute una consulta SparkSQL en la tabla de Spark, no se capturarán datos reales desde el almacén analítico del contenedor subyacente.

En el caso de la **carga a DataFrame de Spark**, los metadatos capturados se almacenan en caché durante la sesión de Spark y, por tanto, las siguientes acciones invocadas en el DataFrame se evalúan con según la instantánea del almacén analítico en el momento de creación del DataFrame.

Por otro lado, en el caso de la **creación de una tabla de Spark**, los metadatos del estado del almacén analítico no se almacenan en caché en Spark y se recargan en cada ejecución de la consulta de SparkSQL en la tabla de Spark.

Por lo tanto, puede elegir entre cargar a DataFrame de Spark y crear una tabla de Spark en función de si quiere que el análisis de Spark se evalúe con una instantánea fija del almacén analítico o con la instantánea más reciente del almacén analítico, respectivamente.

> [!NOTE]
> Para consultar a la API de Azure Cosmos DB de las cuentas de Mongo DB, obtenga más información sobre la [representación de esquemas de fidelidad completa](../../cosmos-db/analytical-store-introduction.md#analytical-schema) en el almacén analítico y los nombres de propiedad extendidos que se van a usar.

### <a name="load-to-spark-dataframe"></a>Cargar a DataFrame de Spark

En este ejemplo, creará un DataFrame de Spark que apunte al almacén analítico de Azure Cosmos DB. A continuación, puede realizar un análisis adicional mediante la invocación de las acciones de Spark en el DataFrame. Esta operación no afecta al almacén transaccional.

La sintaxis en **Python** sería la siguiente:
```python
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

df = spark.read.format("cosmos.olap")\
    .option("spark.synapse.linkedService", "<enter linked service name>")\
    .option("spark.cosmos.container", "<enter container name>")\
    .load()
```

La sintaxis equivalente en **Scala** sería la siguiente:
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

val df_olap = spark.read.format("cosmos.olap").
    option("spark.synapse.linkedService", "<enter linked service name>").
    option("spark.cosmos.container", "<enter container name>").
    load()
```

### <a name="create-spark-table"></a>Crear una tabla de Spark

En este ejemplo, creará una tabla de Spark que apunte al almacén analítico de Azure Cosmos DB. A continuación, puede realizar un análisis adicional mediante la invocación de las consultas de SparkSQL en la tabla. Esta operación no afecta al almacén de transacciones ni provoca ningún movimiento de datos. Si decide eliminar esta tabla de Spark, el contenedor subyacente de Azure Cosmos DB y el almacén analítico correspondiente no se verán afectados. 

Este escenario resulta práctico para volver a usar las tablas de Spark a través de herramientas de terceros y proporcionar accesibilidad a los datos subyacentes durante el tiempo de ejecución.

La sintaxis para crear una tabla de Spark es la siguiente:
```sql
%%sql
-- To select a preferred list of regions in a multi-region Azure Cosmos DB account, add spark.cosmos.preferredRegions '<Region1>,<Region2>' in the config options

create table call_center using cosmos.olap options (
    spark.synapse.linkedService '<enter linked service name>',
    spark.cosmos.container '<enter container name>'
)
```

> [!NOTE]
> Si hay escenarios en los que el esquema del contenedor de Azure Cosmos DB subyacente cambia con el tiempo; y quiere que el esquema actualizado se refleje automáticamente en las consultas a las tabla de Spark, puede lograrlo al establecer la opción `spark.cosmos.autoSchemaMerge` en `true` entre las opciones de la tabla de Spark.


## <a name="write-spark-dataframe-to-azure-cosmos-db-container"></a>Escritura de DataFrame de Spark en el contenedor de Azure Cosmos DB

En este ejemplo, escribirá un DataFrame de Spark en un contenedor de Azure Cosmos DB. Esta operación afectará al rendimiento de las cargas de trabajo transaccionales y consumirá unidades de solicitud aprovisionadas en el contenedor de Azure Cosmos DB o en la base de datos compartida.

La sintaxis en **Python** sería la siguiente:
```python
# Write a Spark DataFrame into an Azure Cosmos DB container
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

YOURDATAFRAME.write.format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "<enter linked service name>")\
    .option("spark.cosmos.container", "<enter container name>")\
    .option("spark.cosmos.write.upsertEnabled", "true")\
    .mode('append')\
    .save()
```

La sintaxis equivalente en **Scala** sería la siguiente:
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

import org.apache.spark.sql.SaveMode

df.write.format("cosmos.oltp").
    option("spark.synapse.linkedService", "<enter linked service name>").
    option("spark.cosmos.container", "<enter container name>"). 
    option("spark.cosmos.write.upsertEnabled", "true").
    mode(SaveMode.Overwrite).
    save()
```

> [!NOTE]
> Si quiere hacer referencia a bibliotecas externas en Apache Spark de Synapse, obtenga más información [aquí](#external-library-management). Por ejemplo, si quiere ingerir un DataFrame de Spark en un contenedor de la API de Cosmos DB para Mongo DB, puede aprovechar el conector de Mongo DB para Spark [aquí](https://docs.mongodb.com/spark-connector/master/).

## <a name="load-streaming-dataframe-from-azure-cosmos-db-container"></a>Cargar un DataFrame de streaming desde un contenedor de Azure Cosmos DB
En este ejemplo, usará la funcionalidad Structured Streaming de Spark para cargar datos de un contenedor de Azure Cosmos DB a un DataFrame de streaming de Spark mediante la funcionalidad de fuente de cambios en Azure Cosmos DB. Los datos del punto de comprobación que usó Spark se almacenarán en la cuenta principal del lago de datos (y en el sistema de archivos) que conectó al área de trabajo.

Si no se creó la carpeta */localReadCheckpointFolder* (en el siguiente ejemplo), se creará automáticamente. Esta operación afectará al rendimiento de las cargas de trabajo transaccionales y consumirá unidades de solicitud aprovisionadas en el contenedor de Azure Cosmos DB o la base de datos compartida.

La sintaxis en **Python** sería la siguiente:
```python
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

dfStream = spark.readStream\
    .format("cosmos.oltp")\
    .option("spark.synapse.linkedService", "<enter linked service name>")\
    .option("spark.cosmos.container", "<enter container name>")\
    .option("spark.cosmos.changeFeed.readEnabled", "true")\
    .option("spark.cosmos.changeFeed.startFromTheBeginning", "true")\
    .option("spark.cosmos.changeFeed.checkpointLocation", "/localReadCheckpointFolder")\
    .option("spark.cosmos.changeFeed.queryName", "streamQuery")\
    .load()
```

La sintaxis equivalente en **Scala** sería la siguiente:
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

val dfStream = spark.readStream.
    format("cosmos.oltp").
    option("spark.synapse.linkedService", "<enter linked service name>").
    option("spark.cosmos.container", "<enter container name>").
    option("spark.cosmos.changeFeed.readEnabled", "true").
    option("spark.cosmos.changeFeed.startFromTheBeginning", "true").
    option("spark.cosmos.changeFeed.checkpointLocation", "/localReadCheckpointFolder").
    option("spark.cosmos.changeFeed.queryName", "streamQuery").
    load()
```

## <a name="write-streaming-dataframe-to-azure-cosmos-db-container"></a>Escritura de DataFrame de streaming a un contenedor de Azure Cosmos DB
En este ejemplo, escribirá un DataFrame de streaming en un contenedor de Azure Cosmos DB. Esta operación afectará al rendimiento de las cargas de trabajo transaccionales y consumirá unidades de solicitud aprovisionadas en el contenedor de Azure Cosmos DB o la base de datos compartida. Si no se creó la carpeta */localWriteCheckpointFolder* (en el siguiente ejemplo), se creará automáticamente. 

La sintaxis en **Python** sería la siguiente:
```python
# To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

streamQuery = dfStream\
        .writeStream\
        .format("cosmos.oltp")\
        .outputMode("append")\
        .option("checkpointLocation", "/localWriteCheckpointFolder")\
        .option("spark.synapse.linkedService", "<enter linked service name>")\
        .option("spark.cosmos.container", "<enter container name>")\
        .option("spark.cosmos.connection.mode", "gateway")\
        .start()

streamQuery.awaitTermination()
```

La sintaxis equivalente en **Scala** sería la siguiente:
```java
// To select a preferred list of regions in a multi-region Azure Cosmos DB account, add .option("spark.cosmos.preferredRegions", "<Region1>,<Region2>")

val query = dfStream.
            writeStream.
            format("cosmos.oltp").
            outputMode("append").
            option("checkpointLocation", "/localWriteCheckpointFolder").
            option("spark.synapse.linkedService", "<enter linked service name>").
            option("spark.cosmos.container", "<enter container name>").
            option("spark.cosmos.connection.mode", "gateway").
            start()

query.awaitTermination()
```

## <a name="external-library-management"></a>Módulo de administración de biblioteca externa

En este ejemplo, aprenderá a hacer referencia a las bibliotecas externas desde archivos JAR al usar los cuadernos de Spark en las áreas de trabajo de Apache Spark de Synpase. Puede colocar los archivos JAR en un contenedor en la cuenta principal de lago de datos que ha conectado al área de trabajo y, a continuación, agregar la siguiente instrucción `%configure` en el cuaderno de Spark:

```cmd
%%configure -f
{
    "jars": [
        "abfss://<storage container name>@<data lake account name>.dfs.core.windows.net/<path to jar>"
    ]
}
```
Si quiere enviar definiciones de trabajos de Spark remotos a un grupo de Spark de Synapse, puede obtener información sobre cómo hacer referencia a las bibliotecas externas con este [tutorial](../spark/apache-spark-job-definitions.md).

## <a name="next-steps"></a>Pasos siguientes

* [Ejemplos para empezar a trabajar con Azure Synapse Link en GitHub](https://aka.ms/cosmosdb-synapselink-samples)
* [Más información sobre características admitidas de Azure Synapse Link para Azure Cosmos DB](./concept-synapse-link-cosmos-db-support.md)
* [Conexión a Synapse Link para Azure Cosmos DB](../quickstart-connect-synapse-link-cosmos-db.md)
