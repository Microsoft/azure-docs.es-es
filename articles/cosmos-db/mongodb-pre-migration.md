---
title: Pasos previos a la migración para migraciones de datos a la API de Azure Cosmos DB para MongoDB
description: Este documento proporciona información general sobre los requisitos previos para una migración de datos de MongoDB a Cosmos DB.
author: LuisBosquez
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: how-to
ms.date: 09/01/2020
ms.author: lbosq
ms.openlocfilehash: be38b1cfa698907f44c6deee77bb9b8ca88b77b7
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89318223"
---
# <a name="pre-migration-steps-for-data-migrations-from-mongodb-to-azure-cosmos-dbs-api-for-mongodb"></a>Pasos previos a la migración para migraciones de datos de MongoDB a la API de Azure Cosmos DB para MongoDB

Antes de migrar los datos de MongoDB (ya sea local o en la nube) a la API de Azure Cosmos DB para MongoDB, se debe hacer lo siguiente:

1. [Leer las consideraciones clave sobre el uso de la API de Azure Cosmos DB para MongoDB](#considerations)
2. [Elegir una opción para migrar sus datos](#options)
3. [Estimar el rendimiento necesario para las cargas de trabajo](#estimate-throughput).
4. [Elegir una clave de partición óptima para los datos](#partitioning).
5. [Comprender la directiva de indexación que se puede establecer en los datos](#indexing).

Si ya se han completado los anteriores requisitos previos para la migración, puede [migrar los datos de MongoDB a la API de Azure Cosmos DB para MongoDB mediante Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db.md). Además, si no ha creado una cuenta, puede examinar cualquiera de los [inicios rápidos](create-mongodb-dotnet.md) que muestran los pasos para crear una cuenta.

## <a name="considerations-when-using-azure-cosmos-dbs-api-for-mongodb"></a><a id="considerations"></a>Consideraciones al usar la API de Azure Cosmos DB para MongoDB

A continuación se muestran características específicas sobre la API de Azure Cosmos DB para MongoDB:

- **Modelo de capacidad**: la capacidad de bases de datos en Azure Cosmos DB se basa en un modelo basado a su vez en el rendimiento. Este modelo se basa en [Unidades de solicitud por segundo](request-units.md), que es una unidad que representa el número de operaciones de base de datos que se pueden ejecutar en una colección por segundo. Esta capacidad se puede asignar en [un nivel de base de datos o de colección](set-throughput.md) y se puede aprovisionar en un modelo de asignación o mediante el [rendimiento aprovisionado de escalabilidad automática](provision-throughput-autoscale.md).

- **Unidades de solicitud**: Cada operación de base de datos tiene un costo de las Unidades de solicitud (RU) en Azure Cosmos DB. Al ejecutarse, se resta del nivel de unidades de solicitud disponible en un segundo dado. Si una solicitud requiere más RU que las RU/s asignadas actualmente, existen dos opciones para resolver el problema: aumentar la cantidad de RU o esperar a que se inicie el próximo segundo y volver a intentar la operación.

- **Capacidad elástica**: la capacidad de una colección o base de datos determinada puede cambiar en cualquier momento. Esto permite a la base de datos adaptarse de forma elástica a los requisitos de rendimiento de su carga de trabajo.

- **Particionamiento automático**: Azure Cosmos DB proporciona un sistema de particionamiento automático que solo requiere una partición (o una clave de partición). El [mecanismo de particionamiento automático](partition-data.md) se comparte entre todas las API de Azure Cosmos DB y permite datos sin problemas y en todo el escalado a través de la distribución horizontal.

## <a name="migration-options-for-azure-cosmos-dbs-api-for-mongodb"></a><a id="options"></a>Opciones de migración para la API de Azure Cosmos DB para MongoDB

[Azure Database Migration Service para la API de Azure Cosmos DB para MongoDB](../dms/tutorial-mongodb-cosmos-db.md) proporciona un mecanismo que simplifica la migración de datos facilitando una plataforma de hospedaje totalmente administrada, opciones de supervisión de la migración y control de limitaciones automático. La lista completa de opciones es la siguiente:

|**Tipo de migración**|**Solución**|**Consideraciones**|
|---------|---------|---------|
|En línea|[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db-online.md)|&bull; Hace uso de la biblioteca BulkExecutor de Azure Cosmos DB <br/>&bull; Adecuada para grandes conjuntos de valores y se encarga de replicar los cambios en directo <br/>&bull; Solo funciona con otros orígenes de MongoDB|
|Sin conexión|[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db-online.md)|&bull; Hace uso de la biblioteca BulkExecutor de Azure Cosmos DB <br/>&bull; Adecuada para grandes conjuntos de valores y se encarga de replicar los cambios en directo <br/>&bull; Solo funciona con otros orígenes de MongoDB|
|Sin conexión|[Azure Data Factory](../data-factory/connector-azure-cosmos-db.md)|&bull; Fácil de configurar y admite varios orígenes <br/>&bull; Hace uso de la biblioteca BulkExecutor de Azure Cosmos DB <br/>&bull; Adecuada para grandes conjuntos de datos <br/>&bull; Faltan puntos de comprobación: significa que, si se produce un problema durante el transcurso de la migración, es necesario reiniciar todo el proceso de migración.<br/>&bull; Falta una cola de mensajes con problemas de entrega: significa que algunos archivos erróneos pueden detener todo el proceso de migración. <br/>&bull; Necesita código personalizado para aumentar el rendimiento de lectura de determinados orígenes de datos|
|Sin conexión|[Herramientas existentes de Mongo (mongodump, mongorestore, Studio3T)](https://azure.microsoft.com/resources/videos/using-mongodb-tools-with-azure-cosmos-db/)|&bull; Fácil de configurar e integrar <br/>&bull; Necesita un control personalizado para las limitaciones|

## <a name="estimate-the-throughput-need-for-your-workloads"></a><a id="estimate-throughput"></a> Estimación del rendimiento necesario para las cargas de trabajo

En Azure Cosmos DB, el rendimiento se aprovisiona de antemano y se mide en Unidades de solicitud (RU) por segundo. A diferencia de las máquinas virtuales o servidores locales, las Unidades de solicitud se pueden escalar y reducir verticalmente fácilmente en cualquier momento. Puede cambiar el número de RU aprovisionadas de forma inmediata. Para más información, consulte [Unidades de solicitud en Azure Cosmos DB](request-units.md).

Puede usar la [calculadora de capacidad de Azure Cosmos DB](https://cosmos.azure.com/capacitycalculator/) para determinar la cantidad de Unidades de solicitud en función de la configuración de la cuenta de base de datos, la cantidad de datos, el tamaño del documento y las lecturas y escrituras requeridas por segundo.

Estos son factores clave que afectan al número de Unidades de solicitud necesarias:
- **Tamaño del documento**: a medida que el tamaño de un elemento o documento aumenta, también lo hace el número de Unidades de solicitud consumidas para leer o escribir el elemento o documento.

- **Recuento de propiedades del documento**: el número de RU consumidas para crear o actualizar un documento está relacionado con el número, la complejidad y la longitud de sus propiedades. Puede reducir el consumo de Unidades de solicitud para operaciones de escritura [limitando el número de las propiedades indexadas](mongodb-indexing.md).

- **Patrones de consultas**: la complejidad de una consulta afecta a la cantidad de Unidades de solicitud que una consulta consume. 

La mejor manera de comprender el costo de las consultas es usar los datos de ejemplo en Azure Cosmos DB [y ejecutar consultas de ejemplo desde el Shell de MongoDB](connect-mongodb-account.md) mediante el comando `getLastRequestStastistics` para obtener el cargo de solicitud, lo que generará el número de Unidades de solicitud consumidas:

`db.runCommand({getLastRequestStatistics: 1})`

Este comando generará un documento JSON similar al siguiente:

```{  "_t": "GetRequestStatisticsResponse",  "ok": 1,  "CommandName": "find",  "RequestCharge": 10.1,  "RequestDurationInMilliSeconds": 7.2}```

También puede usar [la configuración de diagnóstico](cosmosdb-monitor-resource-logs.md) para comprender la frecuencia y los patrones de las consultas ejecutadas en Azure Cosmos DB. Los resultados de los registros de diagnóstico se pueden enviar a una cuenta de almacenamiento, una instancia de EventHub o [Azure Log Analytics](../azure-monitor/log-query/get-started-portal.md).  

## <a name="choose-your-partition-key"></a><a id="partitioning"></a>Elección de la clave de partición
La creación de particiones, también conocida como particionamiento, es un punto clave de consideración antes de migrar los datos. Azure Cosmos DB usa una creación de particiones totalmente administrada para aumentar la capacidad de una base de datos a fin de cumplir los requisitos de almacenamiento y rendimiento. Esta característica no necesita el hospedaje ni la configuración de los servidores de enrutamiento.   

De forma similar, la funcionalidad de creación de particiones agrega capacidad automáticamente y vuelve a equilibrar los datos en consecuencia. Para obtener detalles y recomendaciones sobre cómo elegir la clave de partición correcta para los datos, consulte el [artículo Elección de una clave de partición](partitioning-overview.md#choose-partitionkey). 

## <a name="index-your-data"></a><a id="indexing"></a>Indexación de los datos

La API de Azure Cosmos DB para el servidor MongoDB versión 3.6 solo indexa automáticamente el campo `_id`. Este campo no se puede quitar. Aplica automáticamente la unicidad del campo `_id` por clave de partición. Para indexar campos adicionales, se aplican los comandos de administración de índices de MongoDB. Esta directiva de indexación predeterminada difiere de la API SQL de Azure Cosmos DB, que indexa todos los campos de manera predeterminada.

Entre las funcionalidades de indexación proporcionadas por Azure Cosmos DB se incluye la adición de índices compuestos, índices únicos e índices del período de vida (TTL). La interfaz de administración de índices se asigna al comando `createIndex()`. Obtenga más información en el artículo [Indexación en la API de Azure Cosmos DB para MongoDB](mongodb-indexing.md).

[Azure Database Migration Service](../dms/tutorial-mongodb-cosmos-db.md) migra automáticamente las colecciones de MongoDB con índices únicos. Sin embargo, los índices únicos deben crearse antes de la migración. Azure Cosmos DB no admite la creación de índices únicos cuando ya hay datos en las colecciones. Para más información, consulte [Claves únicas en Azure Cosmos DB](unique-keys.md).

## <a name="next-steps"></a>Pasos siguientes
* [Migración de los datos de MongoDB a Cosmos DB mediante Database Migration Service](../dms/tutorial-mongodb-cosmos-db.md) 
* [Aprovisionamiento del rendimiento en contenedores y bases de datos de Azure Cosmos](set-throughput.md)
* [Creación de particiones en Azure Cosmos DB](partition-data.md)
* [Distribución global en Azure Cosmos DB](distribute-data-globally.md)
* [Indexación en Azure Cosmos DB](index-overview.md)
* [Unidades de solicitud en Azure Cosmos DB](request-units.md)
