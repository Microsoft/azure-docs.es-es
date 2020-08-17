---
title: 'CLI de Azure: detención y eliminación una máquina virtual en un laboratorio'
description: En este artículo se proporciona un script de la CLI de Azure que detiene y elimina una máquina virtual de un laboratorio en Azure DevTest Labs.
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/11/2020
ms.openlocfilehash: 45d5b3453e29571760d47d52a1a21288be31ea90
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88136138"
---
# <a name="use-azure-cli-to-stop-and-delete-a-virtual-machine-in-a-lab-in-azure-devtest-labs"></a>Uso de la CLI de Azure para detener y eliminar una máquina virtual en un laboratorio de Azure DevTest Labs

Este script de la CLI de Azure detiene y elimina una máquina virtual en un laboratorio. 

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli-interactive[main](../../../cli_scripts/devtest-lab/stop-delete-virtual-machine-in-lab/stop-delete-virtual-machine-in-lab.sh "Stop and delete a VM in a lab")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [az lab vm stop](/cli/azure/lab/vm?view=azure-cli-latest#az-lab-vm-stop) | Detiene una máquina virtual en un laboratorio. Esta operación puede tardar varios minutos en completarse. |
| [az lab vm delete](/cli/azure/lab/vm?view=azure-cli-latest#az-lab-vm-delete) | Elimina una máquina virtual en un laboratorio. Esta operación puede tardar varios minutos en completarse. |


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de la CLI de Azure Lab Services en los [ejemplos de la CLI de Azure Lab Services](../samples-cli.md).
