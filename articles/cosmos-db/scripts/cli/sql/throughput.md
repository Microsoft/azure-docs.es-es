---
title: Actualización de la capacidad de proceso de una base de datos y un contenedor de SQL (Core) API para Azure Cosmos DB
description: Actualización de la capacidad de proceso de una base de datos y un contenedor de SQL (Core) API para Azure Cosmos DB
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: sample
ms.date: 07/29/2020
ms.openlocfilehash: 23b18d903bfd4e9d723524924e6c31db45c14d62
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87429897"
---
# <a name="update-rus-for-a-sql-core-api-database-and-container-for-azure-cosmos-db-using-azure-cli"></a>Actualización de la capacidad de proceso de una base de datos y un contenedor de SQL (Core) API para Azure Cosmos DB con la CLI de Azure

[!INCLUDE [cloud-shell-try-it.md](../../../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este tema es preciso que ejecute la CLI de Azure, versión 2.9.1 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="sample-script"></a>Script de ejemplo

Este script crea una base de datos de SQL (Core) API con una capacidad de proceso compartida y un contenedor de SQL (Core) API con una capacidad de proceso dedicada y, a continuación, actualiza la capacidad de proceso de la base de datos y el contenedor.

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/sql/throughput.sh "Update RU/s for a SQL database and container.")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Después de ejecutar el script de ejemplo, se puede usar el comando siguiente para quitar el grupo de recursos y todos los recursos asociados.

```azurecli-interactive
az group delete --name $resourceGroupName
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az cosmosdb create](/cli/azure/cosmosdb#az-cosmosdb-create) | Crea una cuenta de Azure Cosmos DB. |
| [az cosmosdb sql database create](/cli/azure/cosmosdb/sql/database#az-cosmosdb-sql-database-create) | Crea una base de datos de SQL (Core) para Azure Cosmos. |
| [az cosmosdb sql container create](/cli/azure/cosmosdb/sql/container#az-cosmosdb-sql-container-create) | Crea un contenedor de SQL (Core) para Azure Cosmos. |
| [az cosmosdb sql database throughput update](/cli/azure/cosmosdb/sql/database/throughput#az-cosmosdb-sql-database-throughput-update) | Actualiza la capacidad de proceso (RU/s) de una base de datos de SQL (Core) para Azure Cosmos. |
| [az cosmosdb sql container throughput update](/cli/azure/cosmosdb/sql/container/throughput#az-cosmosdb-sql-container-throughput-update) | Actualiza la capacidad de proceso (RU/s) de un contenedor de SQL (Core) para Azure Cosmos. |
| [az group delete](/cli/azure/resource#az-resource-delete) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte la [documentación de la CLI de Azure Cosmos DB](/cli/azure/cosmosdb).

Encontrará más ejemplos de scripts de la CLI de Azure Cosmos DB en el [repositorio de la CLI de Azure Cosmos DB en GitHub](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
