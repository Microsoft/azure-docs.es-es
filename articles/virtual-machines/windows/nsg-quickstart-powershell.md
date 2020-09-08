---
title: Apertura de puertos para una VM mediante Azure PowerShell
description: Aprenda a abrir un puerto o crear un punto de conexión a la máquina virtual Windows con el modelo de implementación de Azure Resource Manager y Azure PowerShell
author: cynthn
ms.service: virtual-machines-windows
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 12/13/2017
ms.author: cynthn
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 0eca3f2fea3cf72b05b301ae307c1179f9bca0be
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89072373"
---
# <a name="how-to-open-ports-and-endpoints-to-a-vm-in-azure-using-powershell"></a>Apertura de puertos y puntos de conexión para una máquina virtual en Azure mediante PowerShell
[!INCLUDE [virtual-machines-common-nsg-quickstart](../../../includes/virtual-machines-common-nsg-quickstart.md)]

## <a name="quick-commands"></a>Comandos rápidos
Para crear reglas de los grupos de seguridad de red y ACL necesitará [que esté instalada la versión más reciente de Azure PowerShell](/powershell/azure/). También puede [llevar a cabo estos pasos con Azure Portal](nsg-quickstart-portal.md).

Inicie sesión en una cuenta de Azure:

```powershell
Connect-AzAccount
```

En los ejemplos siguientes, reemplace los nombres de parámetros por los suyos propios. Los nombres de parámetro de ejemplo incluían *myResourceGroup*, *myNetworkSecurityGroup* y *myVnet*.

Cree una regla con [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecurityruleconfig). En el siguiente ejemplo crea una regla denominada *myNetworkSecurityGroupRule* para permitir el tráfico *TCP* en el puerto *80*:

```powershell
$httprule = New-AzNetworkSecurityRuleConfig `
    -Name "myNetworkSecurityGroupRule" `
    -Description "Allow HTTP" `
    -Access "Allow" `
    -Protocol "Tcp" `
    -Direction "Inbound" `
    -Priority "100" `
    -SourceAddressPrefix "Internet" `
    -SourcePortRange * `
    -DestinationAddressPrefix * `
    -DestinationPortRange 80
```

A continuación, cree un grupo de seguridad de red con [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) y asigne la regla HTTP que acaba de crear como se indica a continuación. En el ejemplo siguiente se crea un grupo de seguridad de red denominado *myNetworkSecurityGroup*:

```powershell
$nsg = New-AzNetworkSecurityGroup `
    -ResourceGroupName "myResourceGroup" `
    -Location "EastUS" `
    -Name "myNetworkSecurityGroup" `
    -SecurityRules $httprule
```

Ahora hay que asignar el grupo de seguridad de red a una subred. En el ejemplo siguiente se asigna una red virtual existente denominada *myVnet* a la variable *$vnet* con [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork):

```powershell
$vnet = Get-AzVirtualNetwork `
    -ResourceGroupName "myResourceGroup" `
    -Name "myVnet"
```

Asocie el grupo de seguridad de red con la subred con [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig). En el ejemplo siguiente, se asocia la subred denominada *mySubnet* con un grupo de seguridad de red:

```powershell
$subnetPrefix = $vnet.Subnets|?{$_.Name -eq 'mySubnet'}

Set-AzVirtualNetworkSubnetConfig `
    -VirtualNetwork $vnet `
    -Name "mySubnet" `
    -AddressPrefix $subnetPrefix.AddressPrefix `
    -NetworkSecurityGroup $nsg
```

Por último, actualice la red virtual con [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) para que los cambios surtan efecto:

```powershell
Set-AzVirtualNetwork -VirtualNetwork $vnet
```


## <a name="more-information-on-network-security-groups"></a>Más información sobre los grupos de seguridad de red
Los comandos rápidos que se describen aquí le permiten ponerse a trabajar con el flujo de tráfico a la máquina virtual. Los grupos de seguridad de red proporcionan muchas características excelentes y un gran nivel de detalle para controlar el acceso a sus recursos. Puede leer más sobre la [creación de un grupo de seguridad de red y las reglas de ACL aquí](tutorial-virtual-network.md#secure-network-traffic).

Para las aplicaciones web de alta disponibilidad, debe colocar las máquinas virtuales detrás de una instancia de Azure Load Balancer. El equilibrador de carga distribuye el tráfico a las máquinas virtuales, con un grupo de seguridad de red que proporciona el filtrado del tráfico. Para más información, consulte [Equilibrio de la carga de máquinas virtuales Linux en Azure para crear una aplicación de alta disponibilidad](tutorial-load-balancer.md).

## <a name="next-steps"></a>Pasos siguientes
En este ejemplo, se ha creado una regla sencilla para permitir tráfico HTTP. Puede encontrar información sobre la creación de entornos más detallados en los siguientes artículos:

* [Información general sobre Azure Resource Manager](../../azure-resource-manager/management/overview.md)
* [¿Qué es un grupo de seguridad de red?](../../virtual-network/security-overview.md)
* [Información general sobre Azure Load Balancer](../../load-balancer/load-balancer-overview.md)
