---
title: 'Creación de una aplicación de funciones de Python sin servidor: CLI de Azure'
description: Creación de una aplicación de funciones de Python sin servidor con la CLI de Azure
ms.assetid: 0e221db6-ee2d-4e16-9bf6-a456cd05b6e7
ms.topic: sample
ms.date: 11/23/2019
ms.custom: devx-track-python, devx-track-azurecli
ms.openlocfilehash: bbc302194d4dc1922344138e057b456b63c18bbf
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87848135"
---
# <a name="create-a-serverless-python-function-app-using-azure-cli"></a>Creación de una aplicación de funciones de Python sin servidor con la CLI de Azure

Este script de ejemplo de Azure Functions crea una aplicación de función, que es un contenedor para las funciones. 

>[!NOTE]
>La aplicación de funciones creada se ejecuta en la versión 3.6 de Python. La versión 3.7 de Python también es compatible con Azure Functions.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 

## <a name="sample-script"></a>Script de ejemplo

Este script crea una aplicación de funciones de Azure con el [Plan de consumo](../functions-scale.md#consumption-plan).

[!code-azurecli-interactive[main](../../../cli_scripts/azure-functions/create-function-app-consumption-python/create-function-app-consumption-python.sh "Create an Azure Function on a Consumption plan")]

[!INCLUDE [cli-script-clean-up](../../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>Explicación del script

Cada comando de la tabla crea un vínculo a documentación específica del comando. Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az storage account create](/cli/azure/storage/account#az-storage-account-create) | Crea una cuenta de Azure Storage. |
| [az functionapp create](/cli/azure/functionapp#az-functionapp-create) | Crea una aplicación de función. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de CLI para Azure Functions en la [documentación de Azure Functions](../functions-cli-samples.md).
