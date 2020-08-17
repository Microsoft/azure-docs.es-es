---
title: 'Copia de un disco administrado en una cuenta de almacenamiento: ejemplo de la CLI de Windows'
description: 'Ejemplo de la CLI de Azure: exporte o copie un disco administrado en una cuenta de almacenamiento.'
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
ms.date: 09/17/2018
ms.author: ramankum
ms.custom: mvc,seodec18, devx-track-azurecli
ms.openlocfilehash: 00b6f0d79427cf1f4af2c1669f514aaf6fe7558a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500341"
---
# <a name="exportcopy-a-managed-disk-to-a-storage-account-using-the-azure-cli"></a>Exportar o copiar un disco administrado en una cuenta de almacenamiento mediante la CLI de Azure

Este script exporta el disco duro virtual (VHD) subyacente de un disco administrado a una cuenta de almacenamiento de la misma región u otra diferente. Primero se genera el URI de SAS del disco administrado y, luego, se usa para copiar el disco duro virtual (VHD) en una cuenta de almacenamiento. Use este script para copiar los discos administrados y realizar la expansión regional.

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

[Crear un disco administrado a partir de un VHD](virtual-machines-windows-cli-sample-create-managed-disk-from-vhd.md?toc=%2fcli%2fmodule%2ftoc.json)

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Se pueden encontrar ejemplos de scripts adicionales de la CLI de máquina virtual y discos administrados en la [documentación de Azure sobre máquinas virtuales Windows](../windows/cli-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
