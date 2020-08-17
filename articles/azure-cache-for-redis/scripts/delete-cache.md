---
title: Eliminación de una instancia de Azure Cache for Redis mediante la CLI de Azure
description: Este ejemplo de código de la CLI de Azure muestra cómo eliminar una instancia de Azure Cache for Redis mediante el comando az redis delete.
author: yegu-ms
ms.author: yegu
tags: azure-service-management
ms.service: cache
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/30/2017
ms.custom: devx-track-azurecli
ms.openlocfilehash: b1e044c6f61e51de1d8f2ba07f64918307109e46
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494730"
---
# <a name="delete-an-azure-cache-for-redis"></a>Eliminación de una instancia de Azure Redis Cache

En este escenario, aprenderá a eliminar una instancia de Azure Cache for Redis.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli[main](../../../cli_scripts/redis-cache/delete-cache/delete-cache.sh "Azure Cache for Redis")]

[!INCLUDE [cli-script-clean-up](../../../includes/redis-cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para eliminar una instancia de Azure Cache for Redis. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [eliminación de az redis](https://docs.microsoft.com/cli/azure/redis) | Eliminación de una instancia de Azure Cache for Redis. |


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure).

Encontrará más ejemplos de scripts de CLI de Azure Cache for Redis en la [documentación de Azure Cache for Redis](../cli-samples.md).
