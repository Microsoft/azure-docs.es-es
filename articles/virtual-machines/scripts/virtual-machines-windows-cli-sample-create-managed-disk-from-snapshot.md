---
title: 'Creación de un disco administrado a partir de una instantánea (Windows): ejemplo de la CLI'
description: 'Ejemplo de script de la CLI de Azure: creación de un disco administrado a partir de una instantánea'
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
ms.openlocfilehash: 68435af52a9593d6b8c9fef0de66e867048919de
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87028505"
---
# <a name="create-a-managed-disk-from-a-snapshot-with-cli-windows"></a>Creación de un disco administrado a partir de una instantánea con la CLI (Windows)

Este script crea un disco administrado a partir de una instantánea. Úselo para restaurar una máquina virtual a partir de las instantáneas de discos de sistema operativo y datos. Cree los discos de sistema operativo y datos a partir de las instantáneas correspondientes y, luego, cree una nueva máquina virtual conectando los discos administrados. También puede restaurar los discos de datos de una máquina virtual existente conectando los discos de datos creados a partir de las instantáneas.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli[main](../../../cli_scripts/virtual-machine/create-managed-disks-from-snapshot/create-managed-disks-from-snapshot.sh "Create managed disk from snapshot")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un disco administrado a partir de una instantánea. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az snapshot show](/cli/azure/snapshot) | Obtiene todas las propiedades de una instantánea usando las propiedades de nombre y grupo de recursos de la instantánea. La propiedad de identificador se utiliza para crear disco administrado.  |
| [az disk create](/cli/azure/disk) | Crea un disco administrado usando un identificador de una instantánea administrada |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Se pueden encontrar ejemplos de scripts adicionales de la CLI de máquina virtual y discos administrados en la [documentación de Azure sobre máquinas virtuales Windows](../windows/cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
