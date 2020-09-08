---
title: Instantánea de VHD para crear muchos discos administrados idénticos (Windows) mediante PowerShell
description: 'Ejemplo de script de Azure PowerShell: creación de una instantánea desde un VHD para crear varios discos administrados idénticos en poco tiempo'
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
ms.openlocfilehash: 47ade94ad2e0be0ce44d346336c6c7eea4fca78b
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89322788"
---
# <a name="create-a-snapshot-from-a-vhd-to-create-multiple-identical-managed-disks-in-small-amount-of-time-with-powershell-windows"></a>Creación de una instantánea desde un VHD para crear varios discos administrados idénticos en poco tiempo con PowerShell (Windows)

Este script crea una instantánea desde un archivo VHD en una cuenta de almacenamiento en una misma suscripción o en una suscripción distinta. Use este script para importar un VHD especializado (no generalizado ni preparado con SysPrep) a una instantánea y luego usar esta instantánea para crear varios discos administrados idénticos en poco tiempo. Además, úsela para importar un VHD de datos a una instantánea y luego use esta instantánea para crear varios discos administrados en poco tiempo. 

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]


 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/virtual-machine/create-snapshots-from-vhd-in-different-subscription/create-snapshots-from-vhd-in-different-subscription.ps1 "Create snapshot from VHD")]


## <a name="next-steps"></a>Pasos siguientes

[Creación de un disco administrado a partir de una instantánea](virtual-machines-powershell-sample-create-managed-disk-from-snapshot.md?toc=%2fpowershell%2fmodule%2ftoc.json)


[Crear una máquina virtual conectando un disco administrado como disco del SO](./virtual-machines-powershell-sample-create-vm-from-managed-os-disks.md)

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de Azure PowerShell de máquina virtual en la [documentación sobre máquinas virtuales Windows de Azure](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
