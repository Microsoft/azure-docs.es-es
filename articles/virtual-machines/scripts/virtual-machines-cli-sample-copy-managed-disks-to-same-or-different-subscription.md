---
title: 'Copia de discos administrados en una suscripción: ejemplo de la CLI'
description: 'Ejemplo de script de la CLI de Azure: copia (o transferencia) de discos administrados en la misma suscripción o en otra'
documentationcenter: storage
author: ramankumarlive
manager: kavithag
ms.service: virtual-machines
ms.devlang: azurecli
ms.topic: sample
ms.workload: infrastructure
ms.date: 05/19/2017
ms.author: ramankum
ms.custom: mvc
ms.openlocfilehash: ed593b37de556d5b62cfec6b726322d12f4d8104
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051848"
---
# <a name="copy-managed-disks-to-same-or-different-subscription-with-cli"></a>Copia de discos administrados en la misma suscripción o en otra con CLI

Este script copia un disco administrado en la misma suscripción o en otra, pero dentro de la misma región. La copia solo funciona si las suscripciones forman parte del mismo inquilino de Azure AD.


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli[main](../../../cli_scripts/virtual-machine/copy-managed-disks-to-same-or-different-subscription/copy-managed-disks-to-same-or-different-subscription.sh "Copy managed disk")]


## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un nuevo disco administrado en la suscripción de destino mediante el identificador de disco administrado de origen. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az disk show](/cli/azure/disk) | Obtiene todas las propiedades de un disco administrado usando las propiedades de nombre y grupo de recursos del disco administrado. La propiedad del identificador se usa para copiar el disco administrado en otra suscripción.  |
| [az disk create](/cli/azure/disk) | Copia un disco administrado mediante la creación de un nuevo disco administrado en otra suscripción con el identificador y nombre el disco administrado primario.  |

## <a name="next-steps"></a>Pasos siguientes

[Crear una máquina virtual a partir de un disco administrado](./virtual-machines-linux-cli-sample-create-vm-from-managed-os-disks.md?toc=%2fpowershell%2fmodule%2ftoc.json)

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de la CLI de máquina virtual y discos administrados en la [documentación de Azure sobre máquinas virtuales Linux](../linux/cli-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
