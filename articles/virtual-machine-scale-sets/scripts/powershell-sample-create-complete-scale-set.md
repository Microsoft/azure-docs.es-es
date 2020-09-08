---
title: 'Ejemplos de Azure PowerShell: Creación de un conjunto de escalado de máquinas virtuales completo'
description: Este script crea un conjunto de escalado de máquinas virtuales que ejecutan Windows Server 2016, en el que se configuran y crean recursos individuales.
author: mimckitt
ms.author: mimckitt
ms.topic: sample
ms.service: virtual-machine-scale-sets
ms.subservice: powershell
ms.date: 05/29/2018
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: d37f5f624459db6bc336884987a16c60503492a8
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078510"
---
# <a name="create-a-complete-virtual-machine-scale-set-with-powershell"></a>Creación de un conjunto de escalado de máquinas virtuales completo con PowerShell

Este script crea un conjunto de escalado de máquinas virtuales que ejecutan Windows Server 2016. Se configuran y crean recursos individuales, en lugar de usar las [opciones integradas de creación de recursos disponibles aquí en New-AzVmss](powershell-sample-create-simple-scale-set.md). Después de ejecutar el script, puede acceder a las instancias de máquina virtual mediante RDP.


[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/virtual-machine-scale-sets/complete-scale-set/complete-scale-set.ps1 "Create a complete virtual machine scale set")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación
Ejecute el siguiente comando para quitar el grupo de recursos, el conjunto de escalado y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name $resourceGroupName
```

## <a name="script-explanation"></a>Explicación del script
Este script usa los siguientes comandos para crear la implementación. Cada elemento de la tabla incluye vínculos a la documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Crea una configuración de subred. Esta configuración se utiliza con el proceso de creación de la red virtual. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crea una red virtual. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crea una dirección IP pública. |
| [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) | Crea una configuración de dirección IP de Front-End para un equilibrador de carga. |
| [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) | Crea una configuración de grupo de direcciones de back-end para un equilibrador de carga. |
| [New-AzLoadBalancerInboundNatRuleConfig](/powershell/module/az.network/new-azloadbalancerinboundnatruleconfig) | Crea una configuración de regla NAT entrante para un equilibrador de carga. |
| [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer) | Crea un equilibrador de carga. |
| [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/new-azloadbalancerprobeconfig) | Crea una configuración de sondeo para un equilibrador de carga. |
| [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/new-azloadbalancerruleconfig) | Crea una configuración de regla para un equilibrador de carga. |
| [Set-AzLoadBalancer](/powershell/module/az.Network/Set-azLoadBalancer) | Actualice el equilibrador de carga con la información que se proporciona. |
| [New-AzVmssIpConfig](/powershell/module/az.Compute/New-azVmssIpConfig) | Cree una configuración de IP para las instancias de máquina virtual del conjunto de escalado. Las instancias de máquina virtual están conectadas al grupo de back-end del equilibrador de carga, a un grupo de NAT y a una subred de la red virtual. |
| [New-AzVmssConfig](/powershell/module/az.Compute/New-azVmssConfig) | Crea una configuración del conjunto de escalado. Dicha configuración incluye información como el número de instancias de máquina virtual que se crean, la SKU de la máquina virtual (tamaño) y el modo de la directiva de actualización. La configuración se agrega a los cmdlets adicionales y se usa durante la creación del conjunto de escalado. |
| [Set-AzVmssStorageProfile](/powershell/module/az.Compute/Set-azVmssStorageProfile) | Defina la imagen que se va a utilizar para las instancias de máquina virtual y agréguela a la configuración del conjunto de escalado. |
| [Set-AzVmssOsProfile](/powershell/module/az.Compute/Set-azVmssStorageProfile) | Defina las credenciales del nombre de usuario y la contraseña de administración, así como el prefijo de asignación de nombres de máquina virtual. Agregue estos valores a la configuración del conjunto de escalado. |
| [Add-AzVmssNetworkInterfaceConfiguration](/powershell/module/az.Compute/Add-azVmssNetworkInterfaceConfiguration) | Agregue una interfaz de red virtual a las instancias de máquina virtual, en función de la configuración de la IP. Agregue estos valores a la configuración del conjunto de escalado. |
| [New-AzVmss](/powershell/module/az.Compute/New-azVmss) | Cree el conjunto de escalado en función de la información proporcionada en la configuración del conjunto de escalado. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Quita un grupo de recursos y todos los recursos incluidos en él. |

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).
