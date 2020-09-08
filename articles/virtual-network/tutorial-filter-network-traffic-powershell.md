---
title: Filtrado del tráfico de red en Azure PowerShell | Microsoft Docs
description: En este artículo, aprenderá a filtrar el tráfico de red a una subred, con un grupo de seguridad de red, mediante PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: twooley
editor: ''
tags: azure-resource-manager
Customer intent: I want to filter network traffic to virtual machines that perform similar functions, such as web servers.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/30/2018
ms.author: kumud
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: c0b0d366518b786b33048d2d2f182abd787856ad
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89075110"
---
# <a name="filter-network-traffic-with-a-network-security-group-using-powershell"></a>Filtrado del tráfico de red con un grupo de seguridad de red mediante PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Puede filtrar el tráfico de red entrante y saliente de una subred de una red virtual con un grupo de seguridad de red. Los grupos de seguridad de red contienen reglas de seguridad que filtran el tráfico de red por dirección IP, puerto y protocolo. Las reglas de seguridad se aplican a los recursos implementados en una subred. En este artículo aprenderá a:

* Crear un grupo de seguridad de red y reglas de seguridad
* Crear una red virtual y asociar un grupo de seguridad de red a una subred
* Implementar máquinas virtuales (VM) en una subred
* Probar los filtros de tráfico

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, para realizar los pasos de este artículo necesita la versión 1.0.0 del módulo de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Un grupo de seguridad de red contiene reglas de seguridad. Las reglas de seguridad especifican un origen y destino. Los orígenes y destinos pueden ser grupos de seguridad de aplicaciones.

### <a name="create-application-security-groups"></a>Creación de grupos de seguridad de aplicaciones

En primer lugar, cree un grupo para todos los recursos generados en este artículo con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el ejemplo siguiente se crea un grupo de recursos en la ubicación *eastus*:

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

Cree un grupo de seguridad de aplicaciones con [New-AzApplicationSecurityGroup](/powershell/module/az.network/new-azapplicationsecuritygroup). Un grupo de seguridad de aplicaciones permite agrupar servidores con requisitos de filtrado de puertos similar. El ejemplo siguiente crea dos grupos de seguridad de aplicaciones.

```azurepowershell-interactive
$webAsg = New-AzApplicationSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Name myAsgWebServers `
  -Location eastus

$mgmtAsg = New-AzApplicationSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Name myAsgMgmtServers `
  -Location eastus
```

### <a name="create-security-rules"></a>Creación de reglas de seguridad

Cree una regla de seguridad con [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig). En el ejemplo siguiente se crea una regla que permite el tráfico entrante desde Internet al grupo de seguridad de aplicaciones *myWebServers* a través de los puertos 80 y 443:

```azurepowershell-interactive
$webRule = New-AzNetworkSecurityRuleConfig `
  -Name "Allow-Web-All" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 100 `
  -SourceAddressPrefix Internet `
  -SourcePortRange * `
  -DestinationApplicationSecurityGroupId $webAsg.id `
  -DestinationPortRange 80,443

The following example creates a rule that allows traffic inbound from the internet to the *myMgmtServers* application security group over port 3389:

$mgmtRule = New-AzNetworkSecurityRuleConfig `
  -Name "Allow-RDP-All" `
  -Access Allow `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 110 `
  -SourceAddressPrefix Internet `
  -SourcePortRange * `
  -DestinationApplicationSecurityGroupId $mgmtAsg.id `
  -DestinationPortRange 3389
```

En este artículo, se expone RDP (puerto 3389) a Internet para la máquina virtual *myAsgMgmtServers*. Para entornos de producción, en lugar de exponer el puerto 3389 a Internet, se recomienda conectarse a los recursos de Azure que desea administrar mediante una [VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o una conexión de red [privada](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Cree un grupo de seguridad de red con [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup). En el ejemplo siguiente se crea un grupo de seguridad de red llamado *myNsg*:

```powershell-interactive
$nsg = New-AzNetworkSecurityGroup `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -Name myNsg `
  -SecurityRules $webRule,$mgmtRule
```

## <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree una red virtual con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). En el ejemplo siguiente se crea una red virtual llamada *myVirtualNetwork*:

```azurepowershell-interactive
$virtualNetwork = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork `
  -AddressPrefix 10.0.0.0/16
```

Cree una configuración de subred con [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) y escríbala en la red virtual con [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork). En el ejemplo siguiente se agrega una subred llamada *mySubnet* a la red virtual y se asocia el grupo de seguridad de red *myNsg* a dicha subred:

```azurepowershell-interactive
Add-AzVirtualNetworkSubnetConfig `
  -Name mySubnet `
  -VirtualNetwork $virtualNetwork `
  -AddressPrefix "10.0.2.0/24" `
  -NetworkSecurityGroup $nsg
$virtualNetwork | Set-AzVirtualNetwork
```

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Antes de crear las máquinas virtuales, recupere el objeto de red virtual con la subred mediante [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork):

```powershell-interactive
$virtualNetwork = Get-AzVirtualNetwork `
 -Name myVirtualNetwork `
 -Resourcegroupname myResourceGroup
```

Cree una dirección IP pública para cada máquina virtual con [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress):

```powershell-interactive
$publicIpWeb = New-AzPublicIpAddress `
  -AllocationMethod Dynamic `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -Name myVmWeb

$publicIpMgmt = New-AzPublicIpAddress `
  -AllocationMethod Dynamic `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -Name myVmMgmt
```

Cree dos interfaces de red con [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface) y asigne una dirección IP pública a la interfaz de red. En el ejemplo siguiente se crea una interfaz de red, se asocia la dirección IP pública *myVmWeb* y se convierte en un miembro del grupo de seguridad de aplicaciones *myAsgWebServers*:

```powershell-interactive
$webNic = New-AzNetworkInterface `
  -Location eastus `
  -Name myVmWeb `
  -ResourceGroupName myResourceGroup `
  -SubnetId $virtualNetwork.Subnets[0].Id `
  -ApplicationSecurityGroupId $webAsg.Id `
  -PublicIpAddressId $publicIpWeb.Id
```

En el ejemplo siguiente se crea una interfaz de red, se asocia la dirección IP pública *myVmMgmt* y se convierte en un miembro del grupo de seguridad de aplicaciones *myAsgWebServers*:

```powershell-interactive
$mgmtNic = New-AzNetworkInterface `
  -Location eastus `
  -Name myVmMgmt `
  -ResourceGroupName myResourceGroup `
  -SubnetId $virtualNetwork.Subnets[0].Id `
  -ApplicationSecurityGroupId $mgmtAsg.Id `
  -PublicIpAddressId $publicIpMgmt.Id
```

Cree dos máquinas virtuales en la red virtual para que pueda validar el filtrado de tráfico en un paso posterior.

Cree una configuración de máquina virtual con [New-AzVMConfig](/powershell/module/az.compute/new-azvmconfig) y, a continuación, cree la máquina virtual con [New-AzVM](/powershell/module/az.compute/new-azvm). En el ejemplo siguiente se crea una máquina virtual que actúa como un servidor web. La opción `-AsJob` crea la máquina virtual en segundo plano, por lo tanto, puede continuar con el siguiente paso:

```azurepowershell-interactive
# Create user object
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."

$webVmConfig = New-AzVMConfig `
  -VMName myVmWeb `
  -VMSize Standard_DS1_V2 | `
Set-AzVMOperatingSystem -Windows `
  -ComputerName myVmWeb `
  -Credential $cred | `
Set-AzVMSourceImage `
  -PublisherName MicrosoftWindowsServer `
  -Offer WindowsServer `
  -Skus 2016-Datacenter `
  -Version latest | `
Add-AzVMNetworkInterface `
  -Id $webNic.Id
New-AzVM `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -VM $webVmConfig `
  -AsJob
```

Cree una máquina virtual para que actúe como un servidor de administración:

```azurepowershell-interactive
# Create user object
$cred = Get-Credential -Message "Enter a username and password for the virtual machine."

# Create the web server virtual machine configuration and virtual machine.
$mgmtVmConfig = New-AzVMConfig `
  -VMName myVmMgmt `
  -VMSize Standard_DS1_V2 | `
Set-AzVMOperatingSystem -Windows `
  -ComputerName myVmMgmt `
  -Credential $cred | `
Set-AzVMSourceImage `
  -PublisherName MicrosoftWindowsServer `
  -Offer WindowsServer `
  -Skus 2016-Datacenter `
  -Version latest | `
Add-AzVMNetworkInterface `
  -Id $mgmtNic.Id
New-AzVM `
  -ResourceGroupName myResourceGroup `
  -Location eastus `
  -VM $mgmtVmConfig
```

La creación de la máquina virtual tarda algunos minutos. No continúe con el paso siguiente hasta que Azure finalice la creación de la máquina virtual.

## <a name="test-traffic-filters"></a>Probar los filtros de tráfico

Use [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) para devolver la dirección IP pública de una máquina virtual. En el siguiente ejemplo se devuelve la dirección IP pública de la máquina virtual *myVmMgmt*:

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmMgmt `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

Use el comando siguiente para crear una sesión de Escritorio remoto con la máquina virtual *myVmMgmt* desde su equipo local. Reemplace `<publicIpAddress>` con la dirección IP que devolvió el comando anterior.

```
mstsc /v:<publicIpAddress>
```

Abra el archivo RDP descargado. Cuando se le pida, seleccione **Conectar**.

Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual (puede que deba seleccionar **Más opciones** y luego **Usar una cuenta diferente**, para especificar las credenciales que escribió cuando creó la máquina virtual). A continuación, seleccione **Aceptar**. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Seleccione **Sí** para continuar con la conexión.

La conexión se realiza correctamente porque el puerto 3389 tiene permitida la entrada desde Internet al grupo de seguridad de aplicaciones *myAsgMgmtServers* en el que se encuentra la interfaz de red conectada a la máquina virtual *myVmMgmt*.

Use el siguiente comando para crear una conexión de escritorio remoto con la máquina virtual *myVmWeb* desde la máquina virtual *myVmMgmt* desde PowerShell:

``` 
mstsc /v:myvmWeb
```

La conexión se realiza correctamente porque una regla de seguridad predeterminada dentro de cada grupo de seguridad de red permite el tráfico a través de todos los puertos entre todas las direcciones IP de una red virtual. No es posible crear una conexión de escritorio remoto con la máquina virtual *myVmWeb* desde Internet porque la regla de seguridad para *myAsgWebServers* no permite el puerto 3389 de entrada desde Internet.

Use el siguiente comando para instalar Microsoft IIS en la máquina virtual *myVmWeb* desde PowerShell:

```powershell
Install-WindowsFeature -name Web-Server -IncludeManagementTools
```

Una vez completada la instalación de IIS, desconecte la sesión de la máquina virtual *myVmWeb*, lo que le deja en la conexión de escritorio remoto de la máquina virtual *myVmMgmt*. Para ver la pantalla de bienvenida de IIS, abra un explorador de Internet y vaya a http:\//myVmWeb.

Desconéctese de la máquina virtual *myVmMgmt*.

En su equipo, escriba el siguiente comando desde PowerShell para recuperar la dirección IP pública del servidor *myVmWeb*:

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVmWeb `
  -ResourceGroupName myResourceGroup `
  | Select IpAddress
```

Para confirmar que tiene acceso al servidor web *myVmWeb* desde fuera de Azure, abra un explorador de Internet en el equipo y vaya a `http://<public-ip-address-from-previous-step>`. La conexión se realiza correctamente porque el puerto 80 tiene permitida la entrada desde Internet al grupo de seguridad de aplicaciones *myAsgMgmtServers* en el que se encuentra la interfaz de red conectada a la máquina virtual *myVmWeb*.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, puede usar [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos y todos los recursos que contiene:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha creado un grupo de seguridad de red y lo ha asociado a una subred de una red virtual. Para más información acerca de los grupos de seguridad de red, consulte [Introducción a los grupos de seguridad de red](security-overview.md) y [Administración de un grupo de seguridad de red](manage-network-security-group.md).

De forma predeterminada, Azure enruta el tráfico entre subredes. En su lugar, puede elegir enrutar el tráfico entre subredes a través de una máquina virtual, que actúa como un firewall, por ejemplo. Para obtener información sobre cómo hacerlo, consulte [Creación de una tabla de rutas](tutorial-create-route-table-powershell.md).
