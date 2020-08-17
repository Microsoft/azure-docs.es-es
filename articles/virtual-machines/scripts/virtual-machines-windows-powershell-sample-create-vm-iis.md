---
title: 'Ejemplo de script de Azure PowerShell: IIS'
description: 'Ejemplo de script de Azure PowerShell: IIS'
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
ms.openlocfilehash: cd10beb52a4bee4161d3bb608d203769cdba0df4
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87082354"
---
# <a name="create-an-iis-vm-with-powershell"></a>Creación de una máquina virtual IIS con PowerShell

Este script crea una máquina virtual de Azure con Windows Server 2016 y, a continuación, usa la extensión de scripts personalizados para máquinas virtuales de Azure para instalar IIS. Una vez ejecutado el script, puedo acceder al sitio web IIS predeterminado en la dirección IP pública de la máquina virtual.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/virtual-machine/create-vm-iis/create-windows-vm-iis.ps1 "Create VM IIS")]

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
| [Set-AzVMExtension](/powershell/module/az.compute/set-azvmextension) | Agregue una extensión de máquina virtual a la máquina virtual. En este ejemplo, se utiliza la extensión de scripts personalizados para instalar IIS. |
|[Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Quita un grupo de recursos y todos los recursos incluidos en él. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Encontrará más ejemplos de scripts de Azure PowerShell de máquina virtual en la [documentación sobre máquinas virtuales Windows de Azure](../windows/powershell-samples.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json).
