---
title: 'Ejemplo de script de la CLI de Azure: ejecución de un trabajo con Batch'
description: Este script crea un trabajo de Batch y agrega una serie de tareas a dicho trabajo. También muestra cómo supervisar un trabajo y sus tareas.
ms.topic: sample
ms.date: 12/12/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: d8efce1bc8dc32070b770769104b7fdc38e683cd
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500238"
---
# <a name="cli-example-run-a-job-and-tasks-with-azure-batch"></a>Ejemplo de la CLI: ejecución de un trabajo y tareas con Azure Batch

Este script crea un trabajo de Batch y agrega una serie de tareas a dicho trabajo. También muestra cómo supervisar un trabajo y sus tareas. 

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, para este artículo es preciso que ejecute la versión 2.0.20 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 

## <a name="example-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/batch/run-job/run-job.sh "Run Job")]

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
| [az batch pool create](/cli/azure/batch/pool#az-batch-pool-create) | Crea un grupo de nodos de proceso.  |
| [az batch job create](/cli/azure/batch/job#az-batch-job-create) | Crea un trabajo de Batch.  |
| [az batch task create](/cli/azure/batch/task#az-batch-task-create) | Agrega una tarea al trabajo de Batch especificado.  |
| [az batch job set](/cli/azure/batch/job#az-batch-job-set) | Actualiza las propiedades de un trabajo de Batch.  |
| [az batch job show](/cli/azure/batch/job#az-batch-job-show) | Recupera los detalles de un trabajo de Batch especificado.  |
| [az batch task show](/cli/azure/batch/task#az-batch-task-show) | Recupera los detalles de una tarea del trabajo de Batch especificado.  |
| [az group delete](/cli/azure/group#az-group-delete) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).
