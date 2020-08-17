---
title: 'Enrutamiento del tráfico mediante una aplicación virtual de red: ejemplo de script de Azure PowerShell'
description: 'Ejemplo de script de Azure PowerShell: redirigir el tráfico a través de una aplicación virtual de red de firewall.'
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: powershell
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/20/2018
ms.author: kumud
ms.openlocfilehash: 6fb16b9839a5f4078347e51647b1f1b916941409
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87068580"
---
# <a name="route-traffic-through-a-network-virtual-appliance-script-sample"></a>Ejemplo de script para el enrutamiento del tráfico mediante una aplicación virtual de red

Este ejemplo de script crea una red virtual con subredes de front-end y back-end. También crea una VM con el reenvío IP habilitado para redirigir el tráfico entre las dos subredes. Después de ejecutar el script puede implementar software de red, como una aplicación de firewall, en la VM.

Puede ejecutar el script desde Azure [Cloud Shell](https://shell.azure.com/powershell) o desde una instalación de PowerShell local. Si usa PowerShell de forma local, este script requiere la versión del módulo Az de PowerShell 5.4.1 o posterior. Ejecute `Get-Module -ListAvailable Az` para ver cuál es la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/virtual-network/route-traffic-through-nva/route-traffic-through-nva.ps1 "Route traffic through a network virtual appliance")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos, la VM y todos los recursos relacionados:

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una red virtual y grupos de seguridad de red. Cada comando de la tabla siguiente crea un vínculo a documentación específica del comando:

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup)  | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crea una subred de front-end y una red virtual de Azure. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Crea subredes de back-end y de red perimetral. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crea una dirección IP pública para acceder a la VM desde Internet. |
| [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) | Crea una interfaz de red virtual y habilita el reenvío IP para ella. |
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Crea un grupo de seguridad de red (NSG). |
| [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) | Crea reglas NSG que permiten puertos HTTP y HTTPS entrantes en la VM. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig)| Asocia los NSG y tablas de rutas a las subredes. |
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable)| Crea una tabla de rutas para todas las rutas. |
| [New-AzRouteConfig](/powershell/module/az.network/new-azrouteconfig)| Crea rutas para redirigir el tráfico entre subredes e Internet a través de la VM. |
| [New-AzVM](/powershell/module/az.compute/new-azvm) | Crea una máquina virtual y conecta la NIC a ella. Este comando también especifica la imagen de máquina virtual que se usará y las credenciales administrativas. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup)  | Elimina un grupo de recursos y todos los recursos que contiene. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](https://docs.microsoft.com/powershell/azure/).

Puede encontrar más ejemplos de script de PowerShell de red virtual en [ejemplos de PowerShell de red virtual](../powershell-samples.md).