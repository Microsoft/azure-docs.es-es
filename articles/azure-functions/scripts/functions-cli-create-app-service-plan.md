---
title: 'Creación de una aplicación de funciones en un plan de App Service: CLI de Azure'
description: 'Ejemplo de script de la CLI de Azure: creación de una instancia de Function App en un plan de App Service'
ms.assetid: 0e221db6-ee2d-4e16-9bf6-a456cd05b6e7
ms.topic: sample
ms.date: 07/03/2018
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: a2faef6162a1a9fe09082330bf52f25dde5f75f6
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498589"
---
# <a name="create-a-function-app-in-an-app-service-plan"></a>Creación de una instancia de Function App en un plan de App Service

Este script de ejemplo de Azure Functions crea una aplicación de función, que es un contenedor para las funciones. La aplicación de función que se crea utiliza un plan de App Service dedicado, lo que significa que los recursos del servidor están siempre disponibles.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso la CLI de Azure versión 2.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 

## <a name="sample-script"></a>Script de ejemplo

Este script crea una aplicación de Azure Function App mediante un [plan de App Service](../functions-scale.md#app-service-plan) dedicado.

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/create-function-app-app-service-plan/create-function-app-app-service-plan.sh "Create an Azure Function on an App Service plan")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Cada comando de la tabla crea un vínculo a documentación específica del comando. Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az storage account create](/cli/azure/storage/account#az-storage-account-create) | Crea una cuenta de Azure Storage. |
| [az functionapp plan create](/cli/azure/functionapp/plan#az-functionapp-plan-create) | Crea un plan Premium. |
| [az functionapp create](/cli/azure/functionapp#az-functionapp-create) | Crea una aplicación de función en el plan de App Service. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de CLI para Azure Functions en la [documentación de Azure Functions](../functions-cli-samples.md).
