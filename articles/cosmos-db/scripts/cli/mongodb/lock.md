---
title: Creación de un bloqueo de recurso para una base de datos y una colección de MongoDB API para Azure Cosmos DB
description: Creación de un bloqueo de recurso para una base de datos y una colección de MongoDB API para Azure Cosmos DB
author: markjbrown
ms.author: mjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-mongo
ms.topic: sample
ms.date: 07/29/2020
ms.openlocfilehash: 879dd060f3de7a638a9191178c3bc1fb8b371de2
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87432134"
---
# <a name="create-a-resource-lock-for-azure-cosmos-dbs-api-for-mongodb-using-azure-cli"></a>Creación de un bloqueo de recurso para la API de Azure Cosmos DB para MongoDB mediante la CLI de Azure

[!INCLUDE [cloud-shell-try-it.md](../../../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este tema es preciso que ejecute la CLI de Azure, versión 2.9.1 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

> [!IMPORTANT]
> Los bloqueos de recursos no funcionarán para los cambios que realicen los usuarios que se conectan mediante cualquier SDK de MongoDB, Mongoshell, cualquier otra herramienta o Azure Portal, a menos que se bloquee en primer lugar la cuenta de Cosmos DB con la propiedad `disableKeyBasedMetadataWriteAccess` habilitada. Para más información sobre cómo habilitar esta propiedad, consulte [Evitar cambios de SDK](../../../role-based-access-control.md#prevent-sdk-changes).

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../../../cli_scripts/cosmosdb/mongodb/lock.sh "Create a resource lock for an Azure Cosmos DB MongoDB API database and collection.")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az lock create](/cli/azure/lock#az-lock-create) | Crea un bloqueo. |
| [az lock list](/cli/azure/lock#az-lock-list) | Muestra información del bloqueo. |
| [az lock show](/cli/azure/lock#az-lock-show) | Muestra las propiedades de un bloqueo. |
| [az lock delete](/cli/azure/lock#az-lock-delete) | Elimina un bloqueo. |

## <a name="next-steps"></a>Pasos siguientes

-[Bloqueo de recursos para impedir cambios inesperados](../../../../azure-resource-manager/management/lock-resources.md)

-[Documentación sobre la CLI de Azure Cosmos DB](/cli/azure/cosmosdb).

-[Repositorio de GitHub para la CLI de Azure Cosmos DB](https://github.com/Azure-Samples/azure-cli-samples/tree/master/cosmosdb).
