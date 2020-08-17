---
title: 'Ejemplos de PowerShell: Instalación de aplicaciones'
description: Este script crea un conjunto de escalado de máquinas virtuales que ejecutan Windows Server 2016 y usa la extensión de script personalizado para instalar una aplicación web básica.
author: mimckitt
ms.author: mimckitt
ms.topic: sample
ms.service: virtual-machine-scale-sets
ms.subservice: powershell
ms.date: 06/25/2020
ms.reviewer: jushiman
ms.custom: mimckitt
ms.openlocfilehash: aff871562c366836906d5125b0cd675a51f9a20c
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087074"
---
# <a name="install-applications-into-a-virtual-machine-scale-set-with-powershell"></a>Instalación de aplicaciones en un conjunto de escalado de máquinas virtuales con PowerShell
Este script crea un conjunto de escalado de máquinas virtuales que ejecutan Windows Server 2016 y usa la extensión de script personalizado para instalar una aplicación web básica. Después de ejecutar el script, puede tener acceso a la aplicación web mediante un explorador web.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/virtual-machine-scale-sets/install-apps/install-apps.ps1 "Install apps into a scale set")]

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
|  [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Quita un grupo de recursos y todos los recursos incluidos en él. |

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).
