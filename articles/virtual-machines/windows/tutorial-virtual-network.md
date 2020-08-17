---
title: 'Tutorial: Creación y administración de redes virtuales de Azure para máquinas virtuales Windows'
description: En este tutorial, aprenderá a usar Azure PowerShell para crear y administrar redes virtuales de Azure para máquinas virtuales Windows
author: cynthn
ms.service: virtual-machines-windows
ms.subservice: networking
ms.topic: tutorial
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 08/04/2020
ms.author: cynthn
ms.custom: mvc
ms.openlocfilehash: 84da38fe71446c54f17c4d4329c7294c5e5176d2
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87800201"
---
# <a name="tutorial-create-and-manage-azure-virtual-networks-for-windows-virtual-machines-with-azure-powershell"></a>Tutorial: Creación y administración de redes virtuales de Azure para máquinas virtuales Windows con Azure PowerShell

Las máquinas virtuales de Azure utilizan las redes de Azure para la comunicación de red interna y externa. Este tutorial le guía a través de la implementación de dos máquinas virtuales y la configuración de redes de Azure para estas máquinas virtuales. Se da por supuesto que en los ejemplos de este tutorial las máquinas virtuales hospedan una aplicación web con un back-end de base de datos, sin embargo, no se implementa ninguna aplicación en el tutorial. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una red virtual y una subred
> * Crear una dirección IP pública
> * Crear una máquina virtual de front-end
> * Protegen el tráfico de red.
> * Creación de la máquina virtual de "back-end"


## <a name="vm-networking-overview"></a>Introducción a las redes de máquinas virtuales

Las redes virtuales de Azure habilitan las conexiones de red seguras entre máquinas virtuales, Internet y otros servicios de Azure como Azure SQL Database. Las redes virtuales se dividen en segmentos lógicos llamados subredes. Las subredes se utilizan para controlar el flujo de red y como límite de seguridad. La implementación de una máquina virtual incluye, por lo general, una interfaz de red virtual, que está conectada a una subred.

Al completar este tutorial, podrá ver que se crearon los siguientes recursos:

![Red virtual con dos subredes](./media/tutorial-virtual-network/networktutorial.png)

- *myVNet*: red virtual que las máquinas virtuales usan para comunicarse entre sí y con Internet.
- *myFrontendSubnet*: subred de *myVNet* que usan los recursos de front-end.
- *myPublicIPAddress*: dirección IP pública que se usa para tener acceso a *myFrontendVM* desde Internet.
- *myFrontendNic*: interfaz de red que usa *myFrontendVM* para comunicarse con *myBackendVM*.
- *myFrontendVM*: máquina virtual que se usa para la comunicación entre Internet y *myBackendVM*.
- *myBackendNSG*: grupo de seguridad de red que controla la comunicación entre *myFrontendVM* y *myBackendVM*.
- *myBackendSubnet*: subred asociada a *myBackendNSG* y que usan los recursos de back-end.
- *myBackendNic*: interfaz de red que usa *myBackendVM* para comunicarse con *myFrontendVM*.
- *myBackendVM*: máquina virtual que usa el puerto 1433 para comunicarse con *myFrontendVM*.


## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.


## <a name="create-subnet"></a>Creación de una subred 

Para este tutorial, se crea una única red virtual con dos subredes: una subred de front-end para hospedar una aplicación web y una subred de back-end para hospedar un servidor de bases de datos.

Antes de poder crear una red virtual, debe crear un grupo de recursos mediante [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el ejemplo siguiente, se crea un grupo de recursos denominado *myRGNetwork* en la ubicación *EastUS*:

```azurepowershell-interactive
New-AzResourceGroup -ResourceGroupName myRGNetwork -Location EastUS
```

Cree una configuración de subred llamada *myFrontendSubnet* mediante [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig):

```azurepowershell-interactive
$frontendSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name myFrontendSubnet `
  -AddressPrefix 10.0.0.0/24
```

Además, debe crear una configuración de subred llamada *myBackendSubnet*:

```azurepowershell-interactive
$backendSubnet = New-AzVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
  -AddressPrefix 10.0.1.0/24
```

## <a name="create-virtual-network"></a>Creación de una red virtual

Cree una red virtual denominada *myVNet* mediante *myFrontendSubnet* y *myBackendSubnet* mediante [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork):

```azurepowershell-interactive
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myVNet `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $frontendSubnet, $backendSubnet
```

Entonces, se ha creado una red y se ha segmentado en dos subredes, una para servicios front-end y otra para servicios back-end. En la siguiente sección, se crean máquinas virtuales y se conectan a estas subredes.

## <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Una dirección IP pública permite que los recursos de Azure estén accesibles en Internet. El método de asignación de la dirección IP pública se puede configurar como dinámico o estático. De forma predeterminada, la dirección IP pública se asigna dinámicamente. Las direcciones IP dinámicas se liberan al desasignar una máquina virtual. Este comportamiento hace que la dirección IP cambie durante cualquier operación que incluya una desasignación de máquina virtual.

El método de asignación se puede establecer en estático, lo que garantiza que la dirección IP siga asignada a una máquina virtual, incluso durante un estado de desasignación. Si usa una dirección IP estática, no se puede especificar la propia dirección IP, sino que se asigna desde un grupo de direcciones disponibles.

Cree una dirección IP pública denominada *myPublicIPAddress* mediante [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress):

```azurepowershell-interactive
$pip = New-AzPublicIpAddress `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -AllocationMethod Dynamic `
  -Name myPublicIPAddress
```

Puede cambiar el parámetro -AllocationMethod a `Static`, para asignar una dirección IP pública estática.

## <a name="create-a-front-end-vm"></a>Crear una máquina virtual de front-end

Para que una máquina virtual se comunique en una red virtual, debe tener una interfaz de red virtual (NIC). Cree una NIC con [New-AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface):

```azurepowershell-interactive
$frontendNic = New-AzNetworkInterface `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myFrontend `
  -SubnetId $vnet.Subnets[0].Id `
  -PublicIpAddressId $pip.Id
```

Establezca el nombre de usuario y la contraseña que se necesitan para la cuenta de administrador en la máquina virtual con [Get-Credential](/powershell/module/microsoft.powershell.security/get-credential?view=powershell-5.1). Use estas credenciales para conectarse a la máquina virtual en pasos adicionales:

```azurepowershell-interactive
$cred = Get-Credential
```

Cree máquinas virtuales mediante [New-AzVM](/powershell/module/az.compute/new-azvm).

```azurepowershell-interactive
New-AzVM `
   -Credential $cred `
   -Name myFrontend `
   -PublicIpAddressName myPublicIPAddress `
   -ResourceGroupName myRGNetwork `
   -Location "EastUS" `
   -Size Standard_D1 `
   -SubnetName myFrontendSubnet `
   -VirtualNetworkName myVNet
```

## <a name="secure-network-traffic"></a>Protegen el tráfico de red.

Un grupo de seguridad de red (NSG) contiene una lista de reglas de seguridad que permiten o deniegan el tráfico de red a recursos conectados a Azure Virtual Network (VNet). Los NSG se pueden asociar a subredes o a interfaces de red individuales. Cuando un NSG está asociado a una interfaz de red, se aplica solo a la máquina virtual asociada. Cuando un grupo de seguridad de red está asociado a una subred, las reglas se aplican a todos los recursos conectados a la subred.

### <a name="network-security-group-rules"></a>Reglas del grupo de seguridad de red

Las reglas de NSG definen puertos de redes a través de los que se permite o deniega el tráfico. Las reglas pueden incluir intervalos de direcciones IP de origen y destino para que el tráfico se controle entre sistemas o subredes específicos. Las reglas de NSG también incluyen una prioridad (entre 1 y 4096). Las reglas se evalúan por orden de prioridad. Una regla con una prioridad de 100 se evalúa antes que una regla con prioridad de 200.

Todos los grupos de seguridad de red contienen un conjunto de reglas predeterminadas. Las reglas predeterminadas no se pueden eliminar, pero dado que tienen asignada la mínima prioridad, pueden reemplazarse por las reglas que cree.

- **Red virtual:** el tráfico que se origina y termina en una red virtual se permite en las direcciones tanto de entrada como de salida.
- **Internet:** se permite el tráfico saliente pero se bloquea el entrante.
- **Equilibrador de carga:** se permite que el equilibrador de carga de Azure sondee el estado de las máquinas virtuales y las instancias de rol. Si no va a usar un conjunto con equilibrio de carga, puede invalidar esta regla.

### <a name="create-network-security-groups"></a>Creación de grupos de seguridad de red

Cree una regla de entrada denominada *myFrontendNSGRule* para permitir el tráfico web entrante en *myFrontendVM* mediante [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig):

```azurepowershell-interactive
$nsgFrontendRule = New-AzNetworkSecurityRuleConfig `
  -Name myFrontendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 200 `
  -SourceAddressPrefix * `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 80 `
  -Access Allow
```

Puede limitar el tráfico interno a *myBackendVM* solo desde *myFrontendVM* creando un NSG para la subred de "back-end". En el ejemplo siguiente, se crea una regla NSG denominada *myBackendNSGRule*:

```azurepowershell-interactive
$nsgBackendRule = New-AzNetworkSecurityRuleConfig `
  -Name myBackendNSGRule `
  -Protocol Tcp `
  -Direction Inbound `
  -Priority 100 `
  -SourceAddressPrefix 10.0.0.0/24 `
  -SourcePortRange * `
  -DestinationAddressPrefix * `
  -DestinationPortRange 1433 `
  -Access Allow
```

Agregue un nuevo grupo de seguridad de red denominado *myFrontendNSG* mediante [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup):

```azurepowershell-interactive
$nsgFrontend = New-AzNetworkSecurityGroup `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myFrontendNSG `
  -SecurityRules $nsgFrontendRule
```

Agregue un nuevo grupo de seguridad de red denominado *myBackendNSG* mediante New-AzNetworkSecurityGroup:

```azurepowershell-interactive
$nsgBackend = New-AzNetworkSecurityGroup `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myBackendNSG `
  -SecurityRules $nsgBackendRule
```

Agregue los grupos de seguridad de red a las subredes:

```azurepowershell-interactive
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName myRGNetwork `
  -Name myVNet
$frontendSubnet = $vnet.Subnets[0]
$backendSubnet = $vnet.Subnets[1]
$frontendSubnetConfig = Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name myFrontendSubnet `
  -AddressPrefix $frontendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgFrontend
$backendSubnetConfig = Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $vnet `
  -Name myBackendSubnet `
  -AddressPrefix $backendSubnet.AddressPrefix `
  -NetworkSecurityGroup $nsgBackend
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

## <a name="create-a-back-end-vm"></a>Crear una máquina virtual de back-end

La manera más fácil de crear la máquina virtual de back-end que se detalla en este tutorial es usar una imagen de SQL Server. Con este tutorial solo se crea la máquina virtual con el servidor de base de datos, pero no se proporciona información del acceso a la base de datos.

Creación de *myBackendNic*:

```azurepowershell-interactive
$backendNic = New-AzNetworkInterface `
  -ResourceGroupName myRGNetwork `
  -Location EastUS `
  -Name myBackend `
  -SubnetId $vnet.Subnets[1].Id
```

Establezca el nombre de usuario y la contraseña que se necesitan para la cuenta de administrador en la máquina virtual con Get-Credential:

```azurepowershell-interactive
$cred = Get-Credential
```

Creación de *myBackendVM*.

```azurepowershell-interactive
New-AzVM `
   -Credential $cred `
   -Name myBackend `
   -ImageName "MicrosoftSQLServer:SQL2016SP1-WS2016:Enterprise:latest" `
   -ResourceGroupName myRGNetwork `
   -Location "EastUS" `
   -SubnetName MyBackendSubnet `
   -VirtualNetworkName myVNet
```

La imagen de este ejemplo tiene instalado SQL Server, pero no se usa en este tutorial. Se incluye para mostrar cómo se pueden configurar una máquina virtual para controlar el tráfico de web y otra para controlar la administración de bases de datos.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado y protegido redes de Azure cuando están relacionadas con máquinas virtuales. 

> [!div class="checklist"]
> * Creación de una red virtual y una subred
> * Crear una dirección IP pública
> * Crear una máquina virtual de front-end
> * Protegen el tráfico de red.
> * Crear una máquina virtual de back-end

Prosiga con el siguiente tutorial para aprender a supervisar la protección de datos en máquinas virtuales mediante Azure Backup.

> [!div class="nextstepaction"]
> [Copia de seguridad de máquinas virtuales Windows en Azure](./tutorial-backup-vms.md)
