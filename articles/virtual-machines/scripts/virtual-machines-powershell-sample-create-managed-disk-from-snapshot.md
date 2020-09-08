---
title: 'Creación de un disco administrado a partir de una instantánea: ejemplo de PowerShell'
description: 'Ejemplo de script de Azure PowerShell: creación de un disco administrado a partir de una instantánea'
services: virtual-machines
documentationcenter: storage
author: ramankumarlive
manager: kavithag
ms.service: virtual-machines
ms.subservice: disks
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 06/05/2017
ms.author: ramankum
ms.openlocfilehash: f3a890083e4763bcff95a1361bed69907b80e033
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322763"
---
# <a name="create-a-managed-disk-from-a-snapshot-with-powershell"></a>Creación de un disco administrado a partir de una instantánea con PowerShell 

Este script crea un disco administrado a partir de una instantánea. Úselo para restaurar una máquina virtual a partir de las instantáneas de discos de sistema operativo y datos. Cree los discos de sistema operativo y datos a partir de las instantáneas correspondientes y, luego, cree una nueva máquina virtual conectando los discos administrados. También puede restaurar los discos de datos de una máquina virtual existente conectando los discos de datos creados a partir de las instantáneas.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/virtual-machine/create-managed-disk-from-snapshot/create-managed-disk-from-snapshot.ps1 "Create managed disk from snapshot")]


## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un disco administrado a partir de una instantánea. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [Get-AzSnapshot](/powershell/module/az.compute/get-azsnapshot) | Obtiene las propiedades de la instantánea.  |
| [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) | Crea la configuración de disco que se usa para la creación del disco. Incluye el identificador de recurso de la instantánea principal y una ubicación igual a la de la instantánea principal y el tipo de almacenamiento.  |
| [New-AzDisk](/powershell/module/az.compute/new-azdisk) | Crea un disco mediante la configuración de disco, el nombre del disco y el nombre del grupo de recursos que se pasan como parámetros. |


## <a name="next-steps"></a>Pasos siguientes

[Crear una máquina virtual a partir de un disco administrado](./virtual-machines-powershell-sample-create-vm-from-managed-os-disks.md?toc=%2fpowershell%2fmodule%2ftoc.json)

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de Azure PowerShell de máquina virtual en la [documentación sobre máquinas virtuales Windows de Azure](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
