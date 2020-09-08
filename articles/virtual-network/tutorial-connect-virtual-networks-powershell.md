---
title: 'Conexión de redes virtuales con emparejamiento de VNet: Azure PowerShell'
description: En este artículo, obtendrá información sobre cómo conectar las redes virtuales con emparejamiento de redes virtuales usando Azure PowerShell.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
tags: azure-resource-manager
Customer intent: I want to connect two virtual networks so that virtual machines in one virtual network can communicate with virtual machines in the other virtual network.
ms.assetid: ''
ms.service: virtual-network
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: kumud
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 6ec552ea525abe6a84bb5e34e00ad317cae038bf
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89077864"
---
# <a name="connect-virtual-networks-with-virtual-network-peering-using-powershell"></a>Conexión de redes virtuales con emparejamiento de redes virtuales usando PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Puede conectar redes virtuales entre sí con el emparejamiento de redes virtuales. Una vez que las redes virtuales están emparejadas, los recursos de ambas se pueden comunicar entre sí con el mismo ancho de banda y la misma latencia que si estuvieran en la misma red virtual. En este artículo aprenderá a:

* Crear dos redes virtuales
* Conectar dos redes virtuales con el emparejamiento de redes virtuales
* Implementar una máquina virtual (VM) en cada red virtual
* Comunicarse entre máquinas virtuales

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, para realizar los pasos de este artículo necesita la versión 1.0.0 del módulo de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-virtual-networks"></a>Creación de redes virtuales

Antes de crear una red virtual, cree un grupo de recursos para ella y los demás recursos que se crearon en este artículo. Cree un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName myResourceGroup -Location EastUS
```

Cree una red virtual con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). En el ejemplo siguiente se crea una red virtual denominada *myVirtualNetwork1* con el prefijo de dirección *10.0.0.0/16*.

```azurepowershell-interactive
$virtualNetwork1 = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork1 `
  -AddressPrefix 10.0.0.0/16
```

Cree una configuración de subred con [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). En el ejemplo siguiente se crea una configuración de subred con un prefijo de dirección 10.0.0.0/24:

```azurepowershell-interactive
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
  -Name Subnet1 `
  -AddressPrefix 10.0.0.0/24 `
  -VirtualNetwork $virtualNetwork1
```

Escriba la configuración de subred en la red virtual con [ASet-AzVirtualNetwork](/powershell/module/az.network/Set-azVirtualNetwork), que crea la siguiente subred:

```azurepowershell-interactive
$virtualNetwork1 | Set-AzVirtualNetwork
```

Cree una red virtual con un prefijo de dirección 10.1.0.0/16 y una subred:

```azurepowershell-interactive
# Create the virtual network.
$virtualNetwork2 = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroup `
  -Location EastUS `
  -Name myVirtualNetwork2 `
  -AddressPrefix 10.1.0.0/16

# Create the subnet configuration.
$subnetConfig = Add-AzVirtualNetworkSubnetConfig `
  -Name Subnet1 `
  -AddressPrefix 10.1.0.0/24 `
  -VirtualNetwork $virtualNetwork2

# Write the subnet configuration to the virtual network.
$virtualNetwork2 | Set-AzVirtualNetwork
```

## <a name="peer-virtual-networks"></a>Emparejamiento de redes virtuales

Cree un emparejamiento con [Add-AzVirtualNetworkPeering](/powershell/module/az.network/add-azvirtualnetworkpeering). El siguiente ejemplo empareja *myVirtualNetwork1* con *myVirtualNetwork2*.

```azurepowershell-interactive
Add-AzVirtualNetworkPeering `
  -Name myVirtualNetwork1-myVirtualNetwork2 `
  -VirtualNetwork $virtualNetwork1 `
  -RemoteVirtualNetworkId $virtualNetwork2.Id
```

En la salida que se devuelve al ejecutarse el comando anterior, verá que **PeeringState** está en estado *Iniciado*. El emparejamiento permanece en estado *Iniciado* hasta que cree el emparejamiento de *myVirtualNetwork2* con *myVirtualNetwork1*. Cree un emparejamiento de *myVirtualNetwork2* con *myVirtualNetwork1*.

```azurepowershell-interactive
Add-AzVirtualNetworkPeering `
  -Name myVirtualNetwork2-myVirtualNetwork1 `
  -VirtualNetwork $virtualNetwork2 `
  -RemoteVirtualNetworkId $virtualNetwork1.Id
```

En la salida que se devuelve al ejecutarse el comando anterior, verá que **PeeringState** está en estado *Conectado*. Azure también cambia el estado del emparejamiento *myVirtualNetwork1-myVirtualNetwork2* a *Conectado*. Confirme que el estado del emparejamiento *myVirtualNetwork1-myVirtualNetwork2* cambia a *Conectado* con [Get-AzVirtualNetworkPeering](/powershell/module/az.network/get-azvirtualnetworkpeering).

```azurepowershell-interactive
Get-AzVirtualNetworkPeering `
  -ResourceGroupName myResourceGroup `
  -VirtualNetworkName myVirtualNetwork1 `
  | Select PeeringState
```

Los recursos de una red virtual no se comunican con los de la otra hasta que el estado **PeeringState** de los emparejamientos de ambas redes virtuales es *Conectado*.

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree una máquina virtual en cada red virtual para que puedan comunicarse entre ellas en un paso posterior.

### <a name="create-the-first-vm"></a>Creación de la primera máquina virtual

Cree una máquina virtual con [New-AzVM](/powershell/module/az.compute/new-azvm). En el ejemplo siguiente se crea una máquina virtual llamada *myVm1* en la red virtual *myVirtualNetwork1*. La opción `-AsJob` crea la máquina virtual en segundo plano, así que puede continuar con el siguiente paso. Cuando se le solicite, escriba el nombre de usuario y la contraseña con los que quiere iniciar sesión en la máquina virtual.

```azurepowershell-interactive
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -VirtualNetworkName "myVirtualNetwork1" `
  -SubnetName "Subnet1" `
  -ImageName "Win2016Datacenter" `
  -Name "myVm1" `
  -AsJob
```

### <a name="create-the-second-vm"></a>Creación de la segunda máquina virtual

```azurepowershell-interactive
New-AzVm `
  -ResourceGroupName "myResourceGroup" `
  -Location "East US" `
  -VirtualNetworkName "myVirtualNetwork2" `
  -SubnetName "Subnet1" `
  -ImageName "Win2016Datacenter" `
  -Name "myVm2"
```

La máquina virtual tarda en crearse unos minutos. No continúe con los pasos siguientes hasta que Azure cree la máquina virtual y devuelve el resultado a PowerShell.

## <a name="communicate-between-vms"></a>Comunicarse entre máquinas virtuales

Puede conectarse a la dirección IP pública de una máquina virtual desde Internet. Use [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) para devolver la dirección IP pública de una máquina virtual. En el siguiente ejemplo se devuelve la dirección IP pública de la máquina virtual *myVm1*:

```azurepowershell-interactive
Get-AzPublicIpAddress `
  -Name myVm1 `
  -ResourceGroupName myResourceGroup | Select IpAddress
```

Ejecute el comando siguiente en el equipo local para crear una sesión de Escritorio remoto con la máquina virtual *myVm1* desde sus sistema local. Reemplace `<publicIpAddress>` con la dirección IP que devolvió el comando anterior.

```
mstsc /v:<publicIpAddress>
```

A continuación se crea, se descarga y se abre en el equipo un archivo de Protocolo de Escritorio remoto (.rdp). Escriba el nombre de usuario y la contraseña (puede que tenga que seleccionar **More choices** [Más opciones] y luego **Use a different account** [Usar una cuenta diferente], para especificar las credenciales que escribió cuando creó la máquina virtual). A continuación, seleccione **Aceptar**. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Haga clic en **Sí** o **Conectar** para continuar con la conexión.

En la máquina virtual *myVm1*, habilite el Protocolo de mensajes de control de Internet (ICMP) a través del Firewall de Windows para que pueda hacer ping en esta máquina virtual desde *myVm2* en un paso posterior, mediante PowerShell:

```powershell
New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
```

Aunque en este artículo se usa ping para comunicarse entre máquinas virtuales, no se recomienda permitir ICMP mediante el Firewall de Windows para las implementaciones de producción.

Para conectar con la máquina virtual *myVm2*, escriba el siguiente comando desde el símbolo del sistema en la máquina virtual *myVm1*:

```
mstsc /v:10.1.0.4
```

Como ha habilitado ping en *myVm1*, ahora puede hacer ping en él mediante una dirección IP desde un símbolo del sistema en la máquina virtual *myVm2*:

```
ping 10.0.0.4
```

Recibirá cuatro respuestas. Desconecte las sesiones RDP para ambos *myVm1* y *myVm2*.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, utilice [Remove-AzResourcegroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos y todos los recursos que contiene.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a conectar dos redes, de la misma región de Azure, con el emparejamiento de redes virtuales. También puede emparejar redes virtuales de distintas [regiones compatibles](virtual-network-manage-peering.md#cross-region) y en [distintas suscripciones de Azure](create-peering-different-subscriptions.md#powershell), además de crear [diseños de red de concentrador y radio](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke#virtual-network-peering) con emparejamiento. Para más información sobre el emparejamiento de redes virtuales, consulte los artículos sobre el [emparejamiento de redes virtuales](virtual-network-peering-overview.md) y la [administración de emparejamientos de redes virtuales](virtual-network-manage-peering.md).

Puede [conectar su equipo a una red virtual](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json) mediante VPN e interactuar con recursos en una red virtual, o en redes virtuales emparejadas. En el caso de los scripts reutilizables para completar muchas de las tareas descritas en los artículos sobre las redes virtuales, consulte los [ejemplos de script](powershell-samples.md).
