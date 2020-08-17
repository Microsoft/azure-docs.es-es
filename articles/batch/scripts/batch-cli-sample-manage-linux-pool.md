---
title: 'Ejemplo de script de la CLI de Azure: grupo de Linux en Batch'
description: Este script muestra algunos de los comandos disponibles en la CLI de Azure para crear y administrar un grupo de nodos de proceso de Linux en Azure Batch.
ms.topic: sample
ms.date: 01/29/2018
ms.custom: devx-track-azurecli
ms.openlocfilehash: 7aba6e64fdf39a069eb010ef032fcc5391fd47ee
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494397"
---
# <a name="cli-example-create-and-manage-a-linux-pool-in-azure-batch"></a>Ejemplo de la CLI: creación y administración de un grupo de Linux en Azure Batch

Este script muestra algunos de los comandos disponibles en la CLI de Azure para crear y administrar un grupo de nodos de proceso de Linux en Azure Batch.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI en un entorno local, para esta guía de inicio rápido es preciso que ejecute la versión 2.0.20 de la CLI de Azure o una versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 

## <a name="example-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/batch/manage-pool/manage-pool-linux.sh "Manage Linux Virtual Machine Pool")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos y todos los recursos asociados a él.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az-group-create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az batch account create](/cli/azure/batch/account#az-batch-account-create) | Crea la cuenta de Batch. |
| [az batch account login](/cli/azure/batch/account#az-batch-account-login) | Realiza la autenticación respecto de la cuenta de Batch especificada para una mayor interacción de la CLI.  |
| [az batch pool node-agent-skus list](../batch-linux-nodes.md#list-of-virtual-machine-images) | Enumera las SKU del agente del nodo disponibles y la información de la imagen.  |
| [az batch pool create](/cli/azure/batch/pool#az-batch-pool-create) | Crea un grupo de nodos de proceso.  |
| [az batch pool resize](/cli/azure/batch/pool#az-batch-pool-resize) | Cambia la cantidad de máquinas virtuales en ejecución en el grupo especificado.  |
| [az batch pool show](/cli/azure/batch/pool#az-batch-pool-show) | Muestra las propiedades de un grupo.  |
| [az batch node list](/cli/azure/batch/node#az-batch-node-list) | Enumera todos los nodos de proceso del grupo especificado.  |
| [az batch node reboot](/cli/azure/batch/node#az-batch-node-reboot) | Reinicia el nodo de proceso especificado.  |
| [az batch node delete](/cli/azure/batch/node#az-batch-node-delete) | Elimina los nodos enumerados del grupo especificado.  |
| [az group delete](/cli/azure/group#az-group-delete) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).
