---
title: 'Creación de una aplicación de funciones en un plan Premium: CLI de Azure'
description: Creación de una aplicación de funciones en un plan Premium escalable en Azure mediante la CLI de Azure
ms.service: azure-functions
ms.topic: sample
ms.date: 11/23/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 70a5a29a251590580ac787efe8460caf07598898
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498283"
---
# <a name="create-a-function-app-in-a-premium-plan---azure-cli"></a>Creación de una aplicación de funciones en un plan Premium: CLI de Azure

Este script de ejemplo de Azure Functions crea una aplicación de función, que es un contenedor para las funciones. La aplicación de funciones que se crea usa un [plan Premium escalable](../functions-premium-plan.md).

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 

## <a name="sample-script"></a>Script de ejemplo

Este script crea una aplicación de funciones con un [plan Premium](../functions-premium-plan.md).

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/create-function-app-premium-plan/create-function-app-premium-plan.sh "Create an Azure Function on an App Service plan")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Cada comando de la tabla crea un vínculo a documentación específica del comando. Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az storage account create](/cli/azure/storage/account#az-storage-account-create) | Crea una cuenta de Azure Storage. |
| [az functionapp plan create](/cli/azure/functionapp/plan#az-functionapp-plan-create) | Crea un plan Premium en una [SKU específica](../functions-premium-plan.md#available-instance-skus). |
| [az functionapp create](/cli/azure/functionapp#az-functionapp-create) | Crea una aplicación de función en el plan de App Service. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de CLI para Azure Functions en la [documentación de Azure Functions](../functions-cli-samples.md).
