---
title: 'Creación de una aplicación de funciones con implementación de DevOps: CLI de Azure'
description: Creación de una instancia de Function App e implementación de código de función desde Azure DevOps
ms.date: 07/03/2018
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 773a08646f7a69e1ed828621bad48a6c6729eb88
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87498538"
---
# <a name="create-a-function-in-azure-that-is-deployed-from-azure-devops"></a>Creación de una función en Azure que se implementa desde Azure DevOps

En este tema se explica cómo usar Azure Functions para crear una aplicación de función [sin servidor](https://azure.microsoft.com/solutions/serverless/) con el [plan de consumo](../functions-scale.md#consumption-plan). La aplicación de función, que es un contenedor para las funciones, se implementa continuamente desde un repositorio de Azure DevOps. 

Para realizar este tema, necesita lo siguiente:

* Un repositorio de Azure DevOps que contenga el proyecto de aplicación de función y para el que disponga de permisos administrativos.
* Un [token de acceso personal (PAT)](/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate) para acceder al repositorio de Azure DevOps.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si prefiere usar la CLI de Azure de forma local, debe instalar y utilizar la versión 2.0 o posterior. Para determinar la versión de la CLI de Azure, ejecute `az --version`. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 

## <a name="sample-script"></a>Script de ejemplo

Este ejemplo crea una instancia de Azure Function App e implementa código de la función desde Azure DevOps.

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/deploy-function-app-with-function-vsts/deploy-function-app-with-function-vsts.sh?highlight=3-4 "Azure Service")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una cuenta de almacenamiento, una aplicación de función y todos los recursos relacionados. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az storage account create](/cli/azure/storage/account#az-storage-account-create) | Crea la cuenta de almacenamiento necesaria para la aplicación de función. |
| [az functionapp create](/cli/azure/functionapp#az-functionapp-create) | Crea una aplicación de función en el [Plan de consumo](../functions-scale.md#consumption-plan) sin servidor. |
| [az functionapp deployment source config](/cli/azure/functionapp/deployment/source#az-functionapp-deployment-source-config) | Asocia una aplicación de función con un repositorio GIT o Mercurial. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de CLI para Azure Functions en la [documentación de Azure Functions](../functions-cli-samples.md).
