---
title: 'Disco duro virtual de un disco administrado en una cuenta de otra región (Windows): PowerShell'
description: 'Ejemplo de script de Azure PowerShell: Exportación o copia del disco duro virtual de un disco administrado en una cuenta de almacenamiento de la misma región u otra diferente'
services: virtual-machines-windows
documentationcenter: storage
author: ramankumarlive
manager: kavithag
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 09/17/2018
ms.author: ramankum
ms.openlocfilehash: ef760688e2a91da6e2f8ab759dc1858d3e583655
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322769"
---
# <a name="exportcopy-the-vhd-of-a-managed-disk-to-a-storage-account-in-different-region-with-powershell-windows"></a>Exportación o copia del disco duro virtual de un disco administrado en una cuenta de almacenamiento de otra región con PowerShell (Windows)

Este script exporta el disco duro virtual de un disco administrado a una cuenta de almacenamiento de otra región. Primero genera el identificador URI de SAS del disco administrado y, luego, lo usa para copiar el disco duro virtual subyacente en una cuenta de almacenamiento de una región diferente. Use este script para copiar los discos administrados en otra región y realizar la expansión regional.  

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/virtual-machine/copy-managed-disks-vhd-to-storage-account/copy-managed-disks-vhd-to-storage-account.ps1 "Copy the VHD of a managed disk")]


## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para generar el URI de SAS de un disco administrado y copia el disco duro virtual subyacente en una cuenta de almacenamiento mediante dicho URI. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [Grant-AzDiskAccess](/powershell/module/az.compute/grant-azdiskaccess) | Genera el URI de SAS de un disco administrado que se usa para copiar el disco duro virtual subyacente en una cuenta de almacenamiento. |
| [New-AzureStorageContext](/powershell/module/azure.storage/new-azurestoragecontext) | Crea un contexto de cuenta de almacenamiento con el nombre de cuenta y la clave. Este contexto se puede usar para realizar operaciones de lectura y escritura en la cuenta de almacenamiento. |
| [Start-AzureStorageBlobCopy](/powershell/module/azure.storage/start-azurestorageblobcopy) | Copia el VHD subyacente de una instantánea en una cuenta de almacenamiento. |

## <a name="next-steps"></a>Pasos siguientes

[Crear un disco administrado a partir de un VHD](virtual-machines-windows-powershell-sample-create-managed-disk-from-vhd.md?toc=%2fpowershell%2fmodule%2ftoc.json)

[Crear una máquina virtual a partir de un disco administrado](./virtual-machines-powershell-sample-create-vm-from-managed-os-disks.md?toc=%2fpowershell%2fmodule%2ftoc.json)

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de Azure PowerShell de máquina virtual en la [documentación sobre máquinas virtuales Windows de Azure](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
