---
title: 'Creación de un disco administrado a partir de un archivo de disco duro virtual en una cuenta de almacenamiento: Ejemplo de PowerShell'
description: 'Script de Azure PowerShell de ejemplo: crear un disco administrado desde un archivo VHD en una cuenta de almacenamiento en la misma o distinta suscripción'
services: virtual-machines-linux
documentationcenter: storage
author: ramankumarlive
manager: kavithag
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-linux
ms.topic: sample
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 06/05/2017
ms.author: ramankum
ms.openlocfilehash: 60621d57313a6d407391dffcf047d2c4e7852854
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079951"
---
# <a name="create-a-managed-disk-from-a-vhd-file-in-a-storage-account-in-same-or-different-subscription-with-powershell"></a>Creación de un disco administrado desde un archivo VHD en una cuenta de almacenamiento en la misma o distinta suscripción con PowerShell

Este script crea un disco administrado desde un archivo VHD en una cuenta de almacenamiento en la misma o distinta suscripción. Utilice este script para importar un archivo VHD especializado (no generalizado o preparado con sysprep) a un disco de sistema operativo administrado para crear una máquina virtual. También puede utilizarlo para importar datos de un archivo VHD a un disco de datos administrado.

No debe crear varios discos administrados idénticos desde un archivo VHD en un período de tiempo corto. Para crear discos administrados desde un archivo VHD, se crear una instantánea de blob del archivo VHD y, a continuación, se utiliza para crear discos administrados. Solo se puede crear una instantánea de blob en un minuto, lo que provoca errores de creación de disco debido a la limitación. Para evitar esta limitación, cree una [instantánea administrada desde el archivo VHD](virtual-machines-linux-powershell-sample-create-snapshot-from-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) y, a continuación, use la instantánea administrada para crear varios discos administrados en un corto período de tiempo.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/virtual-machine/create-managed-disks-from-vhd-in-different-subscription/create-managed-disks-from-vhd-in-different-subscription.ps1 "Create managed disk from VHD")]

## <a name="script-explanation"></a>Explicación del script

Este script utiliza los comandos siguientes para crear un disco administrado desde un archivo VHD en una suscripción distinta. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzDiskConfig](/powershell/module/az.compute/new-azdiskconfig) | Crea la configuración de disco que se usa para la creación del disco. Incluye el tipo de almacenamiento, ubicación, identificador de recurso de la cuenta de almacenamiento donde se almacena el archivo VHD primario, identificador URI del archivo VHD primario. |
| [New-AzDisk](/powershell/module/az.compute/new-azdisk) | Crea un disco mediante la configuración de disco, el nombre del disco y el nombre del grupo de recursos que se pasan como parámetros. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de PowerShell de máquina virtual en la [documentación sobre máquinas virtuales Linux de Azure](../linux/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
