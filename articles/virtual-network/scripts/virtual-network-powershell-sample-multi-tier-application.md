---
title: 'Creación de una red virtual para aplicaciones multinivel: ejemplo de script de Azure PowerShell'
description: 'Ejemplo de script de Azure PowerShell: crear una red virtual para aplicaciones de niveles múltiples.'
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: twooley
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: powershell
ms.topic: sample
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 12/13/2018
ms.author: kumud
ms.openlocfilehash: 773a7d6b12cc515a7525808068080f46d9f79799
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87068709"
---
# <a name="create-a-network-for-multi-tier-applications-script-sample"></a>Creación de una red para aplicaciones de niveles múltiples: script de ejemplo

Este ejemplo de script crea una red virtual con subredes de front-end y back-end. El tráfico a la subred de front-end está limitado a HTTP y SSH, mientras que el tráfico a la subred de back-end está limitado a MySQL, al puerto 3306. Después de ejecutar el script, tendrá dos máquinas virtuales, una en cada subred en la que puede implementar el servidor web y el software MySQL.

Puede ejecutar el script desde Azure [Cloud Shell](https://shell.azure.com/powershell) o desde una instalación de PowerShell local. Si usa PowerShell de forma local, este script requiere la versión del módulo de Azure PowerShell 1.0.0 o posterior. Ejecute `Get-Module -ListAvailable Az` para ver cuál es la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

<!-- gitHub issue https://github.com/MicrosoftDocs/azure-docs/issues/17748 -->
Una vez que se crea una red virtual, se asigna un identificador de subred, específicamente mediante el cmdlet New-AzVirtualNetwork con la opción -Subnet. Si configura la subred mediante el cmdlet New-AzVirtualNetworkSubnetConfig antes de llamar a New-AzVirtualNetwork, no verá el identificador de subred hasta después de llamar a New-AzVirtualNetwork.

[!code-azurepowershell-interactive[main](../../../powershell_scripts/virtual-network/virtual-network-multi-tier-application/virtual-network-multi-tier-application.ps1  "Virtual network for multi-tier application")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos, la VM y todos los recursos relacionados:

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una red virtual y grupos de seguridad de red. Cada comando de la tabla siguiente crea un vínculo a documentación específica del comando:

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crea una subred de front-end y una red virtual de Azure. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Crea una subred de back-end. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crea una dirección IP pública para acceder a la VM desde Internet. |
| [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) | Crea interfaces de red virtual y las asocia a subredes de front-end y back-end de la red virtual. |
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Crea grupos de seguridad de red (NSG) que están asociados a las subredes de front-end y back-end. |
| [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) |Crea reglas NSG que permiten o bloquean puertos específicos para subredes concretas. |
| [New-AzVM](/powershell/module/az.compute/new-azvm) | Crea máquinas virtuales se crea y asocia una NIC a cada VM. Este comando también especifica la imagen de máquina virtual que se usará y las credenciales administrativas. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos y todos los recursos que contiene. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Puede encontrar más ejemplos de script de PowerShell de red virtual en [ejemplos de PowerShell de red virtual](../powershell-samples.md).
