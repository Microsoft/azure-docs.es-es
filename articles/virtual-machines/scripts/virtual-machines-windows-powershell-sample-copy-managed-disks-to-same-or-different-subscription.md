---
title: 'Copia de discos administrados en una suscripción (Windows): ejemplo de PowerShell'
description: 'Ejemplo de script de Azure PowerShell: copia o transferencia de discos administrados en la misma suscripción o en otra'
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
ms.date: 06/06/2017
ms.author: ramankum
ms.openlocfilehash: 6f191eaf5c0a02f77d267999c307d34e083b6cdf
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87069179"
---
# <a name="copy-managed-disks-in-the-same-subscription-or-different-subscription-with-powershell-windows"></a>Copia de discos administrados en la misma suscripción o en otra con PowerShell (Windows)

Este script crea una copia de un disco administrado existente en la misma suscripción o en otra. El nuevo disco se crea en la misma región que el disco administrado primario.   

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/virtual-machine/copy-managed-disks-to-same-or-different-subscription/copy-managed-disks-to-same-or-different-subscription.ps1 "Copy managed disk")]


## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un nuevo disco administrado en la suscripción de destino mediante el identificador de disco administrado de origen. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) | Crea la configuración de disco que se usa para la creación del disco. Incluye el identificador de recurso del disco primario y una ubicación igual a la del disco primario.  |
| [New-AzDisk](/powershell/module/az.compute/new-azdisk) | Crea un disco mediante la configuración de disco, el nombre del disco y el nombre del grupo de recursos que se pasan como parámetros. |


## <a name="next-steps"></a>Pasos siguientes

[Crear una máquina virtual a partir de un disco administrado](./virtual-machines-windows-powershell-sample-create-vm-from-managed-os-disks.md?toc=%2fpowershell%2fmodule%2ftoc.json)

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de Azure PowerShell de máquina virtual en la [documentación sobre máquinas virtuales Windows de Azure](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
