---
title: Ejemplos de la CLI de Azure para Azure Cosmos DB
description: Ejemplos de la CLI de Azure para Azure Cosmos DB
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 07/29/2020
ms.author: mjbrown
ms.custom: devx-track-azurecli
ms.openlocfilehash: 954215f04525e850151fdad93af6e7272b41b3df
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498470"
---
# <a name="azure-cli-samples-for-azure-cosmos-db"></a>Ejemplos de la CLI de Azure para Azure Cosmos DB

En la tabla siguiente se incluyen vínculos a scripts de ejemplo de la CLI de Azure para Azure Cosmos DB. Use los vínculos de la derecha para ir a ejemplos específicos de la API. Los ejemplos comunes son los mismos en todas las API. Hay páginas de referencia para todos los comandos de CLI de Azure Cosmos DB disponibles en la [referencia de la CLI de Azure](/cli/azure/cosmosdb). Encontrará más ejemplos de scripts de la CLI de Azure Cosmos DB en el [repositorio de la CLI de Azure Cosmos DB en GitHub](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).

Estos ejemplos requieren la versión 2.9.1 de la CLI de Azure, o cualquier versión superior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli)

## <a name="common-samples"></a>Ejemplos comunes

Estos ejemplos se aplican a todas las API de Azure Cosmos DB

|Tarea | Descripción |
|---|---|
| [Agregar o regiones de conmutación por error](scripts/cli/common/regions.md?toc=%2fcli%2fazure%2ftoc.json) | Agrega una región, cambia la prioridad de conmutación por error y desencadena una conmutación por error manual.|
| [Claves de cuenta y cadenas de conexión](scripts/cli/common/keys.md?toc=%2fcli%2fazure%2ftoc.json) | Muestra las claves de cuenta y las claves de solo lectura, vuelve a generar las claves y enumera las cadenas de conexión.|
| [Protección con firewall de IP](scripts/cli/common/ipfirewall.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una cuenta de Cosmos con firewall de IP configurado.|
| [Proteger una cuenta nueva con puntos de conexión de servicio](scripts/cli/common/service-endpoints.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una cuenta de Cosmos y la protege con puntos de conexión de servicio.|
| [Proteger una cuenta existente con puntos de conexión de servicio](scripts/cli/common/service-endpoints-ignore-missing-vnet.md?toc=%2fcli%2fazure%2ftoc.json)| Actualiza una cuenta de Cosmos para protegerla con puntos de conexión de servicio cuando la subred se configura finalmente.|
|||

## <a name="core-sql-api-samples"></a>Ejemplos de API Core (SQL)

|Tarea | Descripción |
|---|---|
| [Crear una cuenta, una base de datos y un contenedor de Azure Cosmos](scripts/cli/sql/create.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una cuenta, una base de datos y un contenedor de API Core (SQL) de Azure Cosmos DB. |
| [Creación de una cuenta de Azure Cosmos, una base de datos y un contenedor con escalabilidad automática](scripts/cli/sql/autoscale.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una cuenta, una base de datos y un contenedor con escalabilidad automática para API Core (SQL) de Azure Cosmos DB. |
| [Cambiar la capacidad de proceso](scripts/cli/sql/throughput.md?toc=%2fcli%2fazure%2ftoc.json) | Actualiza la capacidad de proceso (RU/s) de una base de datos y un contenedor.|
| [Bloquear recursos contra la eliminación](scripts/cli/sql/lock.md?toc=%2fcli%2fazure%2ftoc.json)| Bloquee los recursos para impedir que se eliminen.|
|||

## <a name="mongodb-api-samples"></a>Ejemplos de MongoDB API

|Tarea | Descripción |
|---|---|
| [Crear una cuenta, una base de datos y una colección de Azure Cosmos](scripts/cli/mongodb/create.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una cuenta de Azure Cosmos DB y una base de datos y una colección de MongoDB API. |
| [Creación de una cuenta, una base de datos con escalabilidad automática y dos colecciones con rendimiento compartido de Azure Cosmos](scripts/cli/mongodb/autoscale.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una cuenta, una base de datos con escalabilidad automática y dos colecciones con rendimiento compartido para MongoDB API de Azure Cosmos DB. |
| [Cambiar la capacidad de proceso](scripts/cli/mongodb/throughput.md?toc=%2fcli%2fazure%2ftoc.json) | Actualiza la capacidad de proceso (RU/s) de una base de datos y una colección.|
| [Bloquear recursos contra la eliminación](scripts/cli/mongodb/lock.md?toc=%2fcli%2fazure%2ftoc.json)| Bloquee los recursos para impedir que se eliminen.|
|||

## <a name="cassandra-api-samples"></a>Ejemplos de Cassandra API

|Tarea | Descripción |
|---|---|
| [Crear una cuenta, un espacio de claves y una tabla de Azure Cosmos](scripts/cli/cassandra/create.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una cuenta, un espacio de claves y una tabla de Cassandra API para Azure Cosmos DB. |
| [Creación de una cuenta de Azure Cosmos, un espacio de claves y una tabla con escalabilidad automática](scripts/cli/cassandra/autoscale.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una cuenta, un espacio de claves y una tabla de Cassandra API con escalabilidad automática para Azure Cosmos DB. |
| [Cambiar la capacidad de proceso](scripts/cli/cassandra/throughput.md?toc=%2fcli%2fazure%2ftoc.json) | Actualiza la capacidad de proceso (RU/s) de un espacio de claves y una tabla.|
| [Bloquear recursos contra la eliminación](scripts/cli/cassandra/lock.md?toc=%2fcli%2fazure%2ftoc.json)| Bloquee los recursos para impedir que se eliminen.|
|||

## <a name="gremlin-api-samples"></a>Ejemplos de Gremlin API

|Tarea | Descripción |
|---|---|
| [Crear una cuenta, una base de datos y un grafo de Azure Cosmos](scripts/cli/gremlin/create.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una cuenta de Azure Cosmos DB y una base de datos y un grafo de Gremlin API. |
| [Creación de una cuenta, una base de datos y un grafo con escalabilidad automática de Azure Cosmos](scripts/cli/gremlin/autoscale.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una cuenta de Azure Cosmos DB y una base de datos y un grafo con escalabilidad automática para Gremlin API. |
| [Cambiar la capacidad de proceso](scripts/cli/gremlin/throughput.md?toc=%2fcli%2fazure%2ftoc.json) | Actualiza la capacidad de proceso (RU/s) de una base de datos y un grafo.|
| [Bloquear recursos contra la eliminación](scripts/cli/gremlin/lock.md?toc=%2fcli%2fazure%2ftoc.json)| Bloquee los recursos para impedir que se eliminen.|
|||

## <a name="table-api-samples"></a>Ejemplos de Table API

|Tarea | Descripción |
|---|---|
| [Crear una cuenta y una tabla de Azure Cosmos](scripts/cli/table/create.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una cuenta de Azure Cosmos DB y una tabla de Table API. |
| [Creación de una cuenta y una tabla con escalabilidad automática de Azure Cosmos](scripts/cli/table/autoscale.md?toc=%2fcli%2fazure%2ftoc.json)| Crea una cuenta y una tabla con escalabilidad automática para Table API de Azure Cosmos DB. |
| [Cambiar la capacidad de proceso](scripts/cli/table/throughput.md?toc=%2fcli%2fazure%2ftoc.json) | Actualiza la capacidad de proceso (RU/s) de una tabla.|
| [Bloquear recursos contra la eliminación](scripts/cli/table/lock.md?toc=%2fcli%2fazure%2ftoc.json)| Evite la eliminación de recursos con bloqueos de recursos.|
|||
