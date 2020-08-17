---
title: Creación de una instancia de Azure Cache for Redis prémium con agrupación en clústeres mediante la CLI de Azure
description: Este ejemplo de código de la CLI de Azure muestra cómo crear una instancia de Azure Cache for Redis prémium de 6 GB con la agrupación en clústeres habilitada y dos particiones.
author: yegu-ms
ms.author: yegu
tags: azure-service-management
ms.service: cache
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/30/2017
ms.custom: devx-track-azurecli
ms.openlocfilehash: ad29c7d12428d8f010017f9ef3a66cecb82db43a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502841"
---
# <a name="create-a-premium-azure-cache-for-redis-with-clustering"></a>Creación de una instancia de Azure Cache for Redis de nivel Premium con agrupación en clústeres

En este escenario, aprenderá cómo crear Azure Cache for Redis de nivel Premium de 6 GB con la agrupación en clústeres habilitada y dos particiones.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli[main](../../../cli_scripts/redis-cache/create-premium-cache-cluster/create-premium-cache-cluster.sh "Azure Cache for Redis")]

[!INCLUDE [cli-script-clean-up](../../../includes/redis-cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos y una instancia de Azure Cache for Redis de nivel Premium con la habilitación de agrupación en clústeres. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az redis create](https://docs.microsoft.com/cli/azure/redis) | Creación de una instancia de Azure Cache for Redis. |


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](https://docs.microsoft.com/cli/azure).

Encontrará más ejemplos de scripts de CLI de Azure Cache for Redis en la [documentación de Azure Cache for Redis](../cli-samples.md).
