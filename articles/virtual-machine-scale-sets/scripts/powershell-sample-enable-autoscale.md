---
title: 'Ejemplos de Azure PowerShell: Habilitación del escalado automático basado en el host'
description: Este script crea un conjunto de escalado de máquinas virtuales que ejecuta Windows Server 2016 y usa métricas basadas en el host para escalar automáticamente a medida que la carga de CPU cambia.
author: ju-shim
ms.author: jushiman
ms.topic: sample
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 06/25/2020
ms.custom: avverma
ms.openlocfilehash: 9ee64f86cba7287af371f856c442d93e9a03d85b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088917"
---
# <a name="automatically-scale-a-virtual-machine-scale-set-with-powershell"></a>Escalado automático de un conjunto de escalado de máquinas virtuales con PowerShell
Este script crea un conjunto de escalado de máquinas virtuales que ejecuta Windows Server 2016 y usa métricas basadas en el host para escalar automáticamente a medida que la carga de CPU cambia.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="sample-script"></a>Script de ejemplo


[!code-powershell[main](../../../powershell_scripts/virtual-machine-scale-sets/auto-scale-host-metrics/auto-scale-host-metrics.ps1 "Automatically scale a virtual machine scale set")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación
Ejecute el siguiente comando para quitar el grupo de recursos, el conjunto de escalado y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="script-explanation"></a>Explicación del script
Este script usa los siguientes comandos para crear la implementación. Cada elemento de la tabla incluye vínculos a la documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzVmss](/powershell/module/az.compute/new-azvmss) | Crea el conjunto de escalado de máquinas virtuales y todos los recursos auxiliares, como la red virtual, el equilibrador de carga y las reglas NAT. |
| [Get-AzVmss](/powershell/module/az.compute/get-azvmss) | Obtiene información sobre un conjunto de escalado de máquinas virtuales. |
| [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension) | Agrega una extensión de máquina virtual para script personalizado para instalar una aplicación web básica. |
| [Update-AzVmss](/powershell/module/az.compute/update-azvmss) | Actualiza el modelo de conjunto de escalado de máquinas virtuales para aplicar la extensión de máquina virtual. |
| [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) | Obtiene información sobre la dirección IP pública asignada que usa el equilibrador de carga. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Quita un grupo de recursos y todos los recursos incluidos en él. |

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

