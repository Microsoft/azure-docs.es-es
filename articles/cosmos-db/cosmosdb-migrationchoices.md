---
title: Opciones de migración de Cosmos DB
description: En este documento se describen las distintas opciones para migrar los datos locales o en la nube a Azure Cosmos DB.
author: SnehaGunda
ms.author: sngun
ms.service: cosmos-db
ms.topic: how-to
ms.date: 09/01/2020
ms.openlocfilehash: 02a2880c42d4eda42c1a96cfaafb04c85358ccb5
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321283"
---
# <a name="options-to-migrate-your-on-premises-or-cloud-data-to-azure-cosmos-db"></a>Opciones para migrar los datos locales o en la nube a Azure Cosmos DB

Puede cargar datos de varios orígenes de datos en Azure Cosmos DB. Además, como Azure Cosmos DB admite varias API, los destinos pueden ser cualquiera de las API existentes. Con el fin de admitir las rutas de migración de los diversos orígenes a las distintas API de Azure Cosmos DB, hay varias soluciones que proporcionan un control especializado para cada ruta de migración. En este documento se enumeran las soluciones disponibles y se describen sus ventajas y limitaciones.

## <a name="factors-affecting-the-choice-of-migration-tool"></a>Factores que afectan a la elección de la herramienta de migración

Los siguientes factores determinan la elección de la herramienta de migración:
* **Migración en línea frente a migración sin conexión** Muchas herramientas de migración proporcionan una ruta para realizar solo una migración única. Esto significa que las aplicaciones que acceden a la base de datos pueden experimentar un período de inactividad. Algunas soluciones de migración proporcionan una forma de realizar una migración en vivo en la que hay una canalización de replicación configurada entre el origen y el destino.

* **Origen de datos**: Los datos existentes pueden estar en distintos orígenes de datos, como Oracle DB2, Datastax Cassanda, Azure SQL Database, PostgreSQL, etc. Los datos también pueden estar en una cuenta de Azure Cosmos DB existente y la intención de la migración puede ser que cambie el modelo de datos o que se vuelvan a particionar los datos de un contenedor con una clave de partición diferente.

* **API de Azure Cosmos DB**: En el caso de SQL API en Azure Cosmos DB, hay diversas herramientas desarrolladas por el equipo de Azure Cosmos DB que ayudan en los distintos escenarios de migración. Todas las demás API tienen su propio conjunto especializado de herramientas desarrolladas y mantenidas por la comunidad. Como Azure Cosmos DB admite estas API en un nivel de protocolo de conexión, estas herramientas también deberían funcionar tal cual al migrar datos a Azure Cosmos DB. Sin embargo, es posible que requieran un control personalizado para las limitaciones, ya que este concepto es específico de Azure Cosmos DB.

* **Tamaño de los datos**: La mayoría de las herramientas de migración funcionan bien para los conjuntos de datos más pequeños. Cuando el conjunto de datos supera unos cientos de gigabytes, las opciones de herramientas de migración son limitadas. 

* **Duración esperada de la migración**: Las migraciones se pueden configurar para que tengan lugar a un ritmo lento e incremental que consume menos capacidad de proceso o puede consumir toda la capacidad de proceso aprovisionada en el contenedor de Azure Cosmos DB de destino y completar la migración en menos tiempo.

## <a name="azure-cosmos-db-sql-api"></a>API de SQL de Azure Cosmos DB

|**Tipo de migración**|**Solución**|**Consideraciones**|
|---------|---------|---------|
|Sin conexión|[Herramienta de migración de datos](import-data.md)|&bull; Fácil de configurar y admite varios orígenes <br/>&bull; No es adecuada para grandes conjuntos de datos|
|Sin conexión|[Azure Data Factory](../data-factory/connector-azure-cosmos-db.md)|&bull; Fácil de configurar y admite varios orígenes <br/>&bull; Hace uso de la biblioteca BulkExecutor de Azure Cosmos DB <br/>&bull; Adecuada para grandes conjuntos de datos <br/>&bull; Faltan puntos de comprobación: significa que, si se produce un problema durante el transcurso de la migración, es necesario reiniciar todo el proceso de migración.<br/>&bull; Falta una cola de mensajes con problemas de entrega: significa que algunos archivos erróneos pueden detener todo el proceso de migración.|
|Sin conexión|[Conector de Spark de Azure Cosmos DB](spark-connector.md)|&bull; Hace uso de la biblioteca BulkExecutor de Azure Cosmos DB <br/>&bull; Adecuada para grandes conjuntos de datos <br/>&bull; Necesita una instalación personalizada de Spark <br/>&bull; Spark es sensible a las incoherencias del esquema y esto puede ser un problema durante la migración |
|Sin conexión|[Herramienta personalizada con la biblioteca BulkExecutor de Cosmos DB](migrate-cosmosdb-data.md)|&bull; Proporciona funcionalidades de punto de comprobación y de mensajes fallidos que aumenta la resistencia de la migración <br/>&bull; Adecuada para conjuntos de datos de gran tamaño (+10 TB)  <br/>&bull; Requiere la instalación personalizada de esta herramienta para su ejecución como App Service |
|En línea|[Funciones + API de fuente de cambios de Cosmos DB](change-feed-functions.md)|&bull; Fácil de configurar <br/>&bull; Solo funciona si el origen es un contenedor de Azure Cosmos DB <br/>&bull; No es adecuada para grandes conjuntos de datos <br/>&bull; No captura eliminaciones del contenedor de origen |
|En línea|[Servicio de migración personalizado con la fuente de cambios](https://github.com/nomiero/CosmosDBLiveETLSample)|&bull; Proporciona seguimiento del progreso <br/>&bull; Solo funciona si el origen es un contenedor de Azure Cosmos DB <br/>&bull; Funciona también para conjuntos de datos de mayor tamaño <br/>&bull; Requiere que el usuario configure una instancia de App Service para hospedar el procesador de la fuente de cambios <br/>&bull; No captura eliminaciones del contenedor de origen|
|En línea|[Striim](cosmosdb-sql-api-migrate-data-striim.md)|&bull; Funciona con una gran variedad de orígenes, como Oracle, DB2, SQL Server <br/>&bull; Fácil de crear canalizaciones de ETL y proporciona un panel para la supervisión <br/>&bull; Admite conjuntos de datos de mayor tamaño <br/>&bull; Dado que se trata de una herramienta de terceros, debe adquirirse en Marketplace e instalarse en el entorno del usuario|

## <a name="azure-cosmos-db-mongo-api"></a>Mongo API de Azure Cosmos DB

|**Tipo de migración**|**Solución**|**Consideraciones**|
|---------|---------|---------|
|En línea|[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db-online.md)|&bull; Hace uso de la biblioteca BulkExecutor de Azure Cosmos DB <br/>&bull; Adecuada para grandes conjuntos de valores y se encarga de replicar los cambios en directo <br/>&bull; Solo funciona con otros orígenes de MongoDB|
|Sin conexión|[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db-online.md)|&bull; Hace uso de la biblioteca BulkExecutor de Azure Cosmos DB <br/>&bull; Adecuada para grandes conjuntos de valores y se encarga de replicar los cambios en directo <br/>&bull; Solo funciona con otros orígenes de MongoDB|
|Sin conexión|[Azure Data Factory](../data-factory/connector-azure-cosmos-db.md)|&bull; Fácil de configurar y admite varios orígenes <br/>&bull; Hace uso de la biblioteca BulkExecutor de Azure Cosmos DB <br/>&bull; Adecuada para grandes conjuntos de datos <br/>&bull; Faltan puntos de comprobación: significa que, si se produce un problema durante el transcurso de la migración, es necesario reiniciar todo el proceso de migración.<br/>&bull; Falta una cola de mensajes con problemas de entrega: significa que algunos archivos erróneos pueden detener todo el proceso de migración. <br/>&bull; Necesita código personalizado para aumentar el rendimiento de lectura de determinados orígenes de datos|
|Sin conexión|[Herramientas existentes de Mongo (mongodump, mongorestore, Studio3T)](https://azure.microsoft.com/resources/videos/using-mongodb-tools-with-azure-cosmos-db/)|&bull; Fácil de configurar e integrar <br/>&bull; Necesita un control personalizado para las limitaciones|


## <a name="azure-cosmos-db-cassandra-api"></a>API Cassandra de Azure Cosmos DB

|**Tipo de migración**|**Solución**|**Consideraciones**|
|---------|---------|---------|
|Sin conexión|[Comando COPY de cqlsh](cassandra-import-data.md#migrate-data-using-cqlsh-copy-command)|&bull; Fácil de configurar <br/>&bull; No es adecuada para grandes conjuntos de datos <br/>&bull; Solo funciona cuando el origen es una tabla de Cassandra|
|Sin conexión|[Copia de tablas con Spark](cassandra-import-data.md#migrate-data-using-spark) |&bull; Puede hacer uso de las funcionalidades de Spark para paralelizar la transformación y la ingesta <br/>&bull; Necesita configuración con una directiva de reintentos personalizada para controlar las limitaciones|
|En línea|[Striim (desde Oracle DB/Apache Cassandra)](cosmosdb-cassandra-api-migrate-data-striim.md)|&bull; Funciona con una gran variedad de orígenes, como Oracle, DB2, SQL Server <br/>&bull; Fácil de crear canalizaciones de ETL y proporciona un panel para la supervisión <br/>&bull; Admite conjuntos de datos de mayor tamaño <br/>&bull; Dado que se trata de una herramienta de terceros, debe adquirirse en Marketplace e instalarse en el entorno del usuario|
|En línea|[Blitzz (desde Oracle DB/Apache Cassandra)](oracle-migrate-cosmos-db-blitzz.md)|<br/>&bull; Admite conjuntos de datos de mayor tamaño <br/>&bull; Dado que se trata de una herramienta de terceros, debe adquirirse en Marketplace e instalarse en el entorno del usuario|

## <a name="other-apis"></a>Otras API

En el caso de las API que no sean SQL API, Mongo API y Cassandra API, se admiten varias herramientas en cada uno de los ecosistemas existentes de la API. 

**Table API** 

* [Herramienta de migración de datos](table-import.md#data-migration-tool)
* [AzCopy](table-import.md#migrate-data-by-using-azcopy)

**Gremlin API**

* [Biblioteca BulkExecutor de Graph](bulk-executor-graph-dotnet.md)
* [Gremlin Spark](https://github.com/Azure/azure-cosmosdb-spark/blob/2.4/samples/graphframes/main.scala) 

## <a name="next-steps"></a>Pasos siguientes

* Puede aprender más si prueba las aplicaciones de ejemplo que usan la biblioteca BulkExecutor en [.NET](bulk-executor-dot-net.md) y [Java](bulk-executor-java.md). 
* La biblioteca BulkExecutor está integrada en el conector de Spark a Cosmos DB; para más información, vea el artículo sobre el [conector de Spark a Azure Cosmos DB](spark-connector.md).  
* Póngase en contacto con el equipo de producto de Azure Cosmos DB abriendo una incidencia de soporte técnico en el tipo de problema "Asesoramiento general" y en el subtipo "Grandes migraciones (TB+)" para obtener ayuda adicional con las migraciones a gran escala.
