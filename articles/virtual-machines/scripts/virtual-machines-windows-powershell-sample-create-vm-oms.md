---
title: 'Ejemplo de script de Azure PowerShell en Windows: registros de Azure Monitor'
description: 'Ejemplo de script de Azure PowerShell: registros de Azure Monitor'
services: virtual-machines-windows
documentationcenter: virtual-machines
author: cynthn
manager: gwallace
tags: azure-service-management
ms.assetid: ''
ms.service: virtual-machines-windows
ms.topic: sample
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 12/12/2017
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: ee18cffdbb52360a2383c342a9b1133052de57e8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082329"
---
# <a name="create-an-azure-monitor-vm-with-powershell"></a>Creación de una máquina virtual de Azure Monitor con PowerShell 

Este script crea una máquina virtual de Azure, instala el agente de Log Analytics e inscribe el sistema en un área de trabajo de Log Analytics. Después de ejecutar el script, la máquina virtual será visible en Azure Monitor. Además, debe actualizar la clave del área de trabajo y el identificador del área de trabajo de Log Analytics.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/virtual-machine/create-vm-monitor-oms/create-windows-vm-detailed-oms.ps1 "Create VM Log Analytics")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear la implementación. Cada elemento de la tabla incluye vínculos a la documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzVM](/powershell/module/az.compute/new-azvm) | Crea la máquina virtual y la conecta con la tarjeta de red, la red virtual, la subred y el grupo de seguridad de red. Este comando también abre el puerto 80 y establece las credenciales administrativas. |
| [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension) | Agregue una extensión de máquina virtual a la máquina virtual. |
|[Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Quita un grupo de recursos y todos los recursos incluidos en él. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de Azure PowerShell de máquina virtual en la [documentación sobre máquinas virtuales Windows de Azure](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
