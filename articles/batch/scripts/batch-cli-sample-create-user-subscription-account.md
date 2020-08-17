---
title: 'Ejemplo de script de la CLI de Azure: creación de una cuenta en Batch - suscripción de usuario'
description: Este script crea una cuenta en Azure Batch en modo de suscripción de usuario. Esta cuenta asigna nodos de proceso a la suscripción.
ms.topic: sample
ms.date: 01/29/2018
ms.custom: devx-track-azurecli
ms.openlocfilehash: 1c60b9c3de5f2370d7bbc557d36db61d23d32ccf
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494407"
---
# <a name="cli-example-create-a-batch-account-in-user-subscription-mode"></a>Ejemplo de la CLI: creación de una cuenta en Batch en modo de suscripción de usuario

Este script crea una cuenta en Azure Batch en modo de suscripción de usuario. Una cuenta que asigna nodos de proceso a su suscripción debe estar autenticada mediante un token de Azure Active Directory. Los nodos de proceso asignaban un recuento con respecto a la cuota de vCPU (núcleos) de su suscripción. 

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0.20 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 

## <a name="example-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/batch/create-account/create-account-user-subscription.sh "Create Account using user subscription")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos y todos los recursos asociados a él.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az role assignment create](/cli/azure/role) | Crea una nueva asignación de roles para un usuario, grupo o entidad de servicio. |
| [az group create](/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az keyvault create](/cli/azure/keyvault#az-keyvault-create) | Crea un almacén de claves. |
| [az keyvault set-policy](/cli/azure/keyvault#az-keyvault-set-policy) | Actualiza la directiva de seguridad del almacén de Key Vault especificado. |
| [az batch account create](/cli/azure/batch/account#az-batch-account-create) | Crea la cuenta de Batch.  |
| [az batch account login](/cli/azure/batch/account#az-batch-account-login) | Realiza la autenticación respecto de la cuenta de Batch especificada para una mayor interacción de la CLI.  |
| [az group delete](/cli/azure/group#az-group-delete) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).
