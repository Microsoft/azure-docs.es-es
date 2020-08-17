---
title: Ejemplos de Azure PowerShell para Azure Cosmos DB
description: Obtenga ejemplos de Azure PowerShell para realizar tareas comunes en Azure Cosmos DB
author: markjbrown
ms.service: cosmos-db
ms.topic: sample
ms.date: 07/30/2020
ms.author: mjbrown
ms.openlocfilehash: 678e9490fe52a329365174e1a2283a475f3312e5
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87505019"
---
# <a name="azure-powershell-samples-for-azure-cosmos-db"></a>Ejemplos de Azure PowerShell para Azure Cosmos DB

En la tabla siguiente se incluyen vínculos a scripts de Azure PowerShell usados habitualmente para Azure Cosmos DB. Use los vínculos de la derecha para ir a ejemplos específicos de la API. Los ejemplos comunes son los mismos en todas las API. En la [referencia de Azure PowerShell](/powershell/module/az.cosmosdb) hay páginas de referencia para todos los cmdlets de PowerShell de Azure Cosmos DB disponibles. Compruebe periódicamente si hay actualizaciones para `Az.CosmosDB`. También puede bifurcar estos ejemplos de PowerShell para Cosmos DB desde nuestro repositorio de GitHub, [Ejemplos de PowerShell para Cosmos DB en GitHub](https://github.com/Azure/azure-docs-powershell-samples/tree/master/cosmosdb).

## <a name="common-samples"></a>Ejemplos comunes

|Tarea | Descripción |
|---|---|
|[Actualización de una cuenta](scripts/powershell/common/account-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Actualice el nivel de coherencia predeterminado de una cuenta de Cosmos DB. |
|[Actualización de las regiones de una cuenta](scripts/powershell/common/update-region.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Actualice las regiones de una cuenta de Cosmos DB. |
|[Cambio de la prioridad de la conmutación por error o desencadenamiento de la conmutación por error](scripts/powershell/common/failover-priority-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cambie la prioridad de la conmutación por error regional de una cuenta de Azure Cosmos o desencadene una conmutación por error manual. |
|[Claves de cuenta o cadenas de conexión](scripts/powershell/common/keys-connection-strings.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Obtenga las claves principales y secundarias, las cadenas de conexión o vuelva a generar una clave de cuenta de una cuenta de Azure Cosmos DB. |
|[Creación de una cuenta de Cosmos con firewall de IP](scripts/powershell/common/firewall-create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cree una cuenta de Azure Cosmos DB con firewall de IP habilitado. |
|||

## <a name="core-sql-api-samples"></a>Ejemplos de API Core (SQL)

|Tarea | Descripción |
|---|---|
|[Crear una cuenta, base de datos y contenedor](scripts/powershell/sql/create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cree una cuenta, una base de datos y un contenedor de Azure Cosmos DB. |
|[Creación de una cuenta, una base de datos y un contenedor con escalabilidad automática](scripts/powershell/sql/autoscale.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cree una cuenta, una base de datos y un contenedor de Azure Cosmos DB con escalabilidad automática. |
|[Creación de un contenedor con una clave de partición de gran tamaño](scripts/powershell/sql/create-large-partition-key.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cree un contenedor con una clave de partición de gran tamaño. |
|[Creación de un contenedor sin directiva de índice](scripts/powershell/sql/create-index-none.md?toc=%2fpowershell%2fmodule%2ftoc.json) | Cree un contenedor de Azure Cosmos con directiva de índice desactivada.|
|[Enumerar u obtener las bases de datos o contenedores](scripts/powershell/sql/list-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Enumere u obtenga las bases de datos o contenedores. |
|[Obtención de rendimiento](scripts/powershell/sql/throughput-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Obtenga el rendimiento de una base de datos o un contenedor. |
|[Actualización del rendimiento](scripts/powershell/sql/throughput-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Actualice el rendimiento de una base de datos o un contenedor. |
|[Bloquear recursos contra la eliminación](scripts/powershell/sql/lock.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Evite la eliminación de recursos con bloqueos de recursos. |
|||

## <a name="cassandra-api-samples"></a>Ejemplos de Cassandra API

|Tarea | Descripción |
|---|---|
|[Crear una cuenta, un espacio de claves y una tabla](scripts/powershell/cassandra/create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cree una cuenta, un espacio de claves y una tabla de Azure Cosmos. |
|[Creación de una cuenta, un espacio de claves y una tabla con escalabilidad automática](scripts/powershell/cassandra/autoscale.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Crea una cuenta, un espacio de claves y una tabla con escalabilidad automática de Azure Cosmos. |
|[Enumerar u obtener espacios de claves o tablas](scripts/powershell/cassandra/list-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Enumere u obtenga espacios de claves o tablas. |
|[Obtención de rendimiento](scripts/powershell/cassandra/throughput-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Obtenga el rendimiento de un espacio de claves o una tabla. |
|[Actualización del rendimiento](scripts/powershell/cassandra/throughput-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Actualice el rendimiento de un espacio de claves o de una tabla. |
|[Bloquear recursos contra la eliminación](scripts/powershell/cassandra/lock.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Evite la eliminación de recursos con bloqueos de recursos. |
|||

## <a name="mongo-db-api-samples"></a>Ejemplos de Mongo DB API

|Tarea | Descripción |
|---|---|
|[Crear una cuenta, base de datos y colección](scripts/powershell/mongodb/create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cree una cuenta, una base de datos y una colección de Azure Cosmos. |
|[Creación de una cuenta, una base de datos y una colección con escalabilidad automática](scripts/powershell/mongodb/autoscale.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Crea una cuenta, una base de datos y una colección con escalabilidad automática de Azure Cosmos. |
|[Enumerar u obtener las bases de datos o colecciones](scripts/powershell/mongodb/list-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Enumere u obtenga una base de datos o colección. |
|[Obtención de rendimiento](scripts/powershell/mongodb/throughput-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Obtenga el rendimiento de una base de datos o una colección. |
|[Actualización del rendimiento](scripts/powershell/mongodb/throughput-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Actualice el rendimiento de una base de datos o una colección. |
|[Bloquear recursos contra la eliminación](scripts/powershell/mongodb/lock.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Evite la eliminación de recursos con bloqueos de recursos. |
|||

## <a name="gremlin-api-samples"></a>Ejemplos de Gremlin API

|Tarea | Descripción |
|---|---|
|[Crear una cuenta, base de datos y gráfico](scripts/powershell/gremlin/create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cree una cuenta, una base de datos y un gráfico de Azure Cosmos. |
|[Creación de una cuenta, una base de datos y un grafo con escalabilidad automática](scripts/powershell/gremlin/autoscale.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Crea una cuenta, una base de datos y un grafo con escalabilidad automática de Azure Cosmos. |
|[Enumerar u obtener las bases de datos o gráficos](scripts/powershell/gremlin/list-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Enumere u obtenga las bases de datos o gráficos. |
|[Obtención de rendimiento](scripts/powershell/gremlin/throughput-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Obtenga el rendimiento de una base de datos o un grafo. |
|[Actualización del rendimiento](scripts/powershell/gremlin/throughput-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Actualice el rendimiento de una base de datos o un grafo. |
|[Bloquear recursos contra la eliminación](scripts/powershell/gremlin/lock.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Evite la eliminación de recursos con bloqueos de recursos. |
|||

## <a name="table-api-samples"></a>Ejemplos de Table API

|Tarea | Descripción |
|---|---|
|[Crear una cuenta y tabla](scripts/powershell/table/create.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Cree una cuenta y una tabla de Azure Cosmos. |
|[Creación de una cuenta y una tabla con escalabilidad automática](scripts/powershell/table/autoscale.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Crea una cuenta y una tabla con escalabilidad automática de Azure Cosmos. |
|[Enumerar u obtener tablas](scripts/powershell/table/list-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Enumere u obtenga las tablas. |
|[Obtención de rendimiento](scripts/powershell/table/throughput-get.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Obtenga el rendimiento de una tabla. |
|[Actualización del rendimiento](scripts/powershell/table/throughput-update.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Actualice el rendimiento de una tabla. |
|[Bloquear recursos contra la eliminación](scripts/powershell/table/lock.md?toc=%2fpowershell%2fmodule%2ftoc.json)| Evite la eliminación de recursos con bloqueos de recursos. |
|||
