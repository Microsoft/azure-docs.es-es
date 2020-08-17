---
title: Creación de un entorno de prueba de Azure Firewall
description: Este script de ejemplo crea un firewall y un entorno de red de prueba. La red tiene una red virtual con tres subredes.
services: virtual-network
author: vhorne
ms.service: firewall
ms.devlang: powershell
ms.topic: sample
ms.date: 11/19/2019
ms.author: victorh
ms.openlocfilehash: 87ca8591be42e34a52f15913e0ec4058505cc0b0
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87079053"
---
# <a name="create-an-azure-firewall-test-environment"></a>Creación de un entorno de prueba de Azure Firewall

Este script de ejemplo crea un firewall y un entorno de red de prueba. La red tiene una red virtual, con tres subredes: *AzureFirewallSubnet*, *ServersSubnet* y *JumpboxSubnet*. Las subredes ServersSubnet y JumpboxSubnet tienen un servidor de Windows Server de dos núcleos en ellas.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

El firewall está en AzureFirewallSubnet y está configurado con una colección de reglas de aplicación con una única regla que permite el acceso a `www.microsoft.com`.

Se crea una ruta definida por el usuario que señala el tráfico de red desde ServersSubnet a través del firewall, en donde se aplican las reglas de firewall.

Puede ejecutar el script desde Azure [Cloud Shell](https://shell.azure.com/powershell) o desde una instalación de PowerShell local. 

Si ejecuta PowerShell de forma local, este script requiere Azure PowerShell. Ejecute `Get-Module -ListAvailable Az` para ver cuál es la versión instalada. 

Puede usar `PowerShellGet` si necesita actualizar. Esta versión está integrada en Windows 10 y Windows Server 2016.

> [!NOTE]
>Otras versiones de Windows le pedirán que instale `PowerShellGet` antes de utilizarla. Puede ejecutar `Get-Module -Name PowerShellGet -ListAvailable | Select-Object -Property Name,Version,Path` para determinar si está instalada en el sistema. Si la salida está en blanco, deberá instalar la versión más reciente de [Windows Management Framework](https://www.microsoft.com/download/details.aspx?id=54616).

Para más información, consulte [Azure PowerShell](/powershell/azure/install-Az-ps)

Cualquier instalación de Azure PowerShell existente hecha con el instalador de plataforma web entrará en conflicto con la instalación de PowerShellGet y se debe eliminar.

Recuerde que si PowerShell se ejecuta localmente, también deberá ejecutar `Connect-AzAccount` para crear una conexión con Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo


[!code-azurepowershell-interactive[main](../../../powershell_scripts/firewall/create-fw-test.ps1  "Create a firewall test environment")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Ejecute el siguiente comando para quitar el grupo de recursos, la VM y todos los recursos relacionados:

```powershell
Remove-AzResourceGroup -Name AzfwSampleScriptEastUS -Force
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una red virtual y grupos de seguridad de red. Cada comando de la tabla siguiente crea un vínculo a documentación específica del comando:

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Crea un objeto de configuración de subred. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Crea una subred de front-end y una red virtual de Azure. |
| [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig) | Crea reglas de seguridad que se asignarán a un grupo de seguridad de red. |
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) |Crea reglas NSG que permiten o bloquean puertos específicos para subredes concretas. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Asocia NSG a subredes. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crea una dirección IP pública para acceder a la VM desde Internet. |
| [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) | Crea interfaces de red virtual y las asocia a subredes de front-end y back-end de la red virtual. |
| [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig) | Crea una configuración de máquina virtual. Esta configuración incluye diversa información, como el nombre de la máquina virtual, sistema el operativo y las credenciales administrativas. La configuración se utiliza durante la creación de las máquinas virtuales. |
| [New-AzVM](/powershell/module/az.compute/new-azvm) | Cree una máquina virtual. |
|[Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Quita un grupo de recursos y todos los recursos incluidos en él. |
|[New-AzFirewall](https://docs.microsoft.com/powershell/module/az.network/new-azfirewall)| Crea una nueva instancia de Azure Firewall.|
|[Get-AzFirewall](https://docs.microsoft.com/powershell/module/az.network/get-azfirewall)|Obtiene un objeto de Azure Firewall.|
|[New-AzFirewallApplicationRule](https://docs.microsoft.com/powershell/module/az.network/new-azfirewallapplicationrule)|Crea una nueva regla de aplicación de Azure Firewall.|
|[Set-AzFirewall](https://docs.microsoft.com/powershell/module/az.network/set-azfirewall)|Confirma los cambios en el objeto de Azure Firewall.|

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

