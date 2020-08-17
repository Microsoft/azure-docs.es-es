---
title: 'Copia de discos administrados en una suscripción: Ejemplo de la CLI'
description: 'Ejemplo de script de CLI de Azure: copia (transferencia) de discos administrados en la misma suscripción o en otra'
services: virtual-machines-windows
documentationcenter: storage
author: ramankumarlive
manager: kavithag
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.devlang: azurecli
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 05/19/2017
ms.author: ramankum
ms.custom: mvc
ms.openlocfilehash: 4b1516e492b13426bed6cae8637abd5c713bcd63
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87010145"
---
# <a name="copy-managed-disks-to-same-or-different-subscription-with-cli"></a>Copia de discos administrados en la misma suscripción o en otra con CLI

Este script copia un disco administrado en la misma suscripción o en otra, pero dentro de la misma región. La copia solo funciona si las suscripciones forman parte del mismo inquilino de AAD.

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

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Se pueden encontrar ejemplos de scripts adicionales de la CLI de máquina virtual y discos administrados en la [documentación de Azure sobre máquinas virtuales Windows](../windows/cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
