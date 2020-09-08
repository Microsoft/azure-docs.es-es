---
title: 'Copia de un disco administrado en una cuenta de almacenamiento: CLI'
description: 'Ejemplo de la CLI de Azure: exporte o copie un disco administrado en una cuenta de almacenamiento.'
documentationcenter: storage
author: ramankumarlive
manager: kavithag
ms.service: virtual-machines
ms.subservice: disks
ms.topic: sample
ms.workload: infrastructure
ms.date: 05/09/2019
ms.author: ramankum
ms.custom: mvc,seodec18, devx-track-azurecli
ms.openlocfilehash: c43a18f1dcb4122eb6c1407ca11b7c60653594c4
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322799"
---
# <a name="exportcopy-a-managed-disk-to-a-storage-account-using-the-azure-cli"></a>Exportar o copiar un disco administrado en una cuenta de almacenamiento mediante la CLI de Azure

Este script exporta el disco duro virtual (VHD) subyacente de un disco administrado a una cuenta de almacenamiento de la misma región u otra diferente. Primero se genera el URI de SAS del disco administrado y, luego, se usa para copiar el disco duro virtual (VHD) en una cuenta de almacenamiento. Use este script para copiar los discos administrados en otra región y realizar la expansión regional. Si desea publicar el archivo VHD de un disco administrado en Azure Marketplace, puede utilizar este script para copiar el archivo VHD a una cuenta de almacenamiento y después generar un identificador URI de SAS del VHD copiado para publicarlo en Marketplace.   


[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli[main](../../../cli_scripts/virtual-machine/copy-managed-disks-vhd-to-storage-account/copy-managed-disks-vhd-to-storage-account.sh "Copy the VHD of a managed disk")]


## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para generar el URI de SAS de un disco administrado y copia el disco duro virtual (VHD) subyacente en una cuenta de almacenamiento mediante dicho URI. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az disk grant-access](/cli/azure/disk?view=azure-cli-latest#az-disk-grant-access) | Genera la SAS de solo lectura que se usa para copiar el archivo de VHD subyacente en una cuenta de almacenamiento o descargarlo localmente.  |
| [az storage blob copy start](/cli/azure/storage/blob/copy) | Copia un blob de forma asincrónica desde una cuenta de almacenamiento a otra. |

## <a name="next-steps"></a>Pasos siguientes

[Crear un disco administrado a partir de un VHD](virtual-machines-cli-sample-create-managed-disk-from-vhd.md)

[Crear una máquina virtual a partir de un disco administrado](virtual-machines-linux-cli-sample-create-vm-from-managed-os-disks.md)

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Encontrará más ejemplos de scripts de la CLI de máquina virtual y discos administrados en la [documentación de Azure sobre máquinas virtuales Linux](../linux/cli-samples.md).
