---
title: Implementación y configuración de Azure Firewall en una red híbrida con PowerShell
description: En este artículo, aprenderá a implementar y configurar Azure Firewall mediante Azure PowerShell.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 08/28/2020
ms.author: victorh
customer intent: As an administrator, I want to control network access from an on-premises network to an Azure virtual network.
ms.openlocfilehash: a91d0e11c44657a2d4cdd267ffa6490ca89532a9
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89069415"
---
# <a name="deploy-and-configure-azure-firewall-in-a-hybrid-network-using-azure-powershell"></a>Implementación y configuración de Azure Firewall en una red híbrida con Azure PowerShell

Cuando conecta la red local a una red virtual de Azure para crear una red híbrida, la capacidad de controlar el acceso a los recursos de la red de Azure es parte importante de un plan de seguridad global.

Puede usar Azure Firewall para controlar el acceso de red en una red híbrida con reglas que definen el tráfico de red que se permite o que se rechaza.

En este artículo se crearán tres redes virtuales:

- **VNet-Hub**: el firewall está en esta red virtual.
- **VNet-Spoke**: la red virtual Spoke representa la carga de trabajo ubicada en Azure.
- **VNet-Onprem**: la red virtual local representa una red local. En una implementación real, se puede conectar mediante una conexión VPN o ExpressRoute. Para simplificar, este artículo usa una conexión de puerta de enlace de VPN y una red virtual ubicada en Azure para representar una red local.

![Firewall en una red híbrida](media/tutorial-hybrid-ps/hybrid-network-firewall.png)

En este artículo aprenderá a:

* Declaración de las variables
* Crear la red virtual del centro de firewall
* Crear la red virtual de tipo hub-and-spoke
* Crear la red virtual local
* Configuración e implementación del firewall
* Creación y conexión de las puertas de enlace de VPN
* Emparejar las redes virtuales de tipo hub-and-spoke
* Creación de las rutas
* Creación de las máquinas virtuales
* Probar el firewall

Si desea usar Azure Portal en lugar de completar este tutorial, consulte [Tutorial: Implementación y configuración de Azure Firewall en una red híbrida con Azure Portal](tutorial-hybrid-portal.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

En este artículo es necesario ejecutar PowerShell en local. Debe tener instalados el módulo de Azure PowerShell. Ejecute `Get-Module -ListAvailable Az` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-Az-ps). Después de verificar la versión de PowerShell, ejecute `Login-AzAccount` para crear una conexión con Azure.

Hay tres requisitos clave para que este escenario funcione correctamente:

- Una ruta definida por el usuario en la subred de radio que apunte a la dirección IP de Azure Firewall como puerta de enlace predeterminada. La propagación de las rutas de puerta de enlace de red virtual se debe **deshabilitar** en esta tabla de rutas.
- Una ruta definida por el usuario en la subred de la puerta de enlace del centro debe apuntar a la dirección IP del firewall como próximo salto para las redes de radio.

   No se requiere ninguna ruta definida por el usuario en la subred de Azure Firewall, ya que obtiene las rutas de BGP.
- Asegúrese de establecer **AllowGatewayTransit** al emparejar VNet-Hub con VNet-Spoke y **UseRemoteGateways** al emparejar VNet-Spoke con VNet-Hub.

Consulte la sección [Creación de rutas](#create-the-routes) en este artículo para ver cómo se crean estas rutas.

>[!NOTE]
>Azure Firewall debe tener conectividad directa a Internet. Si AzureFirewallSubnet aprende una ruta predeterminada a la red local mediante BGP, debe reemplazarla por una UDR 0.0.0.0/0 con el valor **NextHopType** establecido como **Internet** para mantener la conectividad directa a Internet.
>
>Azure Firewall puede configurarse para admitir la tunelización forzada. Para más información, consulte [Tunelización forzada de Azure Firewall](forced-tunneling.md).

>[!NOTE]
>El tráfico entre redes virtuales emparejadas directamente se enruta directamente aunque una ruta definida por el usuario apunte a Azure Firewall como puerta de enlace predeterminada. Para enviar tráfico de subred a subred al firewall en este escenario, una UDR debe contener el prefijo de red de la subred de destino de forma explícita en ambas subredes.

Para revisar la documentación de referencia de Azure PowerShell relacionada, consulte [Referencia de Azure PowerShell](https://docs.microsoft.com/powershell/module/az.network/new-azfirewall).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="declare-the-variables"></a>Declaración de las variables

En el ejemplo siguiente se declaran las variables con los valores para este artículo. En algunos casos, puede que tenga que reemplazar algunos valores por los suyos propios para trabajar en su suscripción. Si es necesario, modifique las variables y después cópielas y péguelas en la consola de PowerShell.

```azurepowershell
$RG1 = "FW-Hybrid-Test"
$Location1 = "East US"

# Variables for the firewall hub VNet

$VNetnameHub = "VNet-hub"
$SNnameHub = "AzureFirewallSubnet"
$VNetHubPrefix = "10.5.0.0/16"
$SNHubPrefix = "10.5.0.0/24"
$SNGWHubPrefix = "10.5.1.0/24"
$GWHubName = "GW-hub"
$GWHubpipName = "VNet-hub-GW-pip"
$GWIPconfNameHub = "GW-ipconf-hub"
$ConnectionNameHub = "hub-to-Onprem"

# Variables for the spoke virtual network

$VnetNameSpoke = "VNet-Spoke"
$SNnameSpoke = "SN-Workload"
$VNetSpokePrefix = "10.6.0.0/16"
$SNSpokePrefix = "10.6.0.0/24"
$SNSpokeGWPrefix = "10.6.1.0/24"

# Variables for the on-premises virtual network

$VNetnameOnprem = "Vnet-Onprem"
$SNNameOnprem = "SN-Corp"
$VNetOnpremPrefix = "192.168.0.0/16"
$SNOnpremPrefix = "192.168.1.0/24"
$SNGWOnpremPrefix = "192.168.2.0/24"
$GWOnpremName = "GW-Onprem"
$GWIPconfNameOnprem = "GW-ipconf-Onprem"
$ConnectionNameOnprem = "Onprem-to-hub"
$GWOnprempipName = "VNet-Onprem-GW-pip"

$SNnameGW = "GatewaySubnet"
```


## <a name="create-the-firewall-hub-virtual-network"></a>Crear la red virtual del centro de firewall

En primer lugar, cree el grupo de recursos en el que se incluirán los recursos de este artículo:

```azurepowershell
  New-AzResourceGroup -Name $RG1 -Location $Location1
  ```

Defina las subredes que se incluirán en la red virtual:

```azurepowershell
$FWsub = New-AzVirtualNetworkSubnetConfig -Name $SNnameHub -AddressPrefix $SNHubPrefix
$GWsub = New-AzVirtualNetworkSubnetConfig -Name $SNnameGW -AddressPrefix $SNGWHubPrefix
```

Ahora, cree la red virtual del centro de firewall:

```azurepowershell
$VNetHub = New-AzVirtualNetwork -Name $VNetnameHub -ResourceGroupName $RG1 `
-Location $Location1 -AddressPrefix $VNetHubPrefix -Subnet $FWsub,$GWsub
```

Solicite que se asigne una dirección IP pública a la puerta de enlace de VPN que creará para la red virtual. Observe que *AllocationMethod* es **Dynamic**. No puede especificar la dirección IP que desea usar. Se asigna dinámicamente a la puerta de enlace de VPN.

  ```azurepowershell
  $gwpip1 = New-AzPublicIpAddress -Name $GWHubpipName -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Dynamic
```

## <a name="create-the-spoke-virtual-network"></a>Crear la red virtual de tipo hub-and-spoke

Defina las subredes que se incluirán en la red virtual de tipo hub-and-spoke:

```azurepowershell
$Spokesub = New-AzVirtualNetworkSubnetConfig -Name $SNnameSpoke -AddressPrefix $SNSpokePrefix
$GWsubSpoke = New-AzVirtualNetworkSubnetConfig -Name $SNnameGW -AddressPrefix $SNSpokeGWPrefix
```

Cree la red virtual de tipo hub-and-spoke:

```azurepowershell
$VNetSpoke = New-AzVirtualNetwork -Name $VnetNameSpoke -ResourceGroupName $RG1 `
-Location $Location1 -AddressPrefix $VNetSpokePrefix -Subnet $Spokesub,$GWsubSpoke
```

## <a name="create-the-on-premises-virtual-network"></a>Crear la red virtual local

Defina las subredes que se incluirán en la red virtual:

```azurepowershell
$Onpremsub = New-AzVirtualNetworkSubnetConfig -Name $SNNameOnprem -AddressPrefix $SNOnpremPrefix
$GWOnpremsub = New-AzVirtualNetworkSubnetConfig -Name $SNnameGW -AddressPrefix $SNGWOnpremPrefix
```

Ahora, cree la red virtual local:

```azurepowershell
$VNetOnprem = New-AzVirtualNetwork -Name $VNetnameOnprem -ResourceGroupName $RG1 `
-Location $Location1 -AddressPrefix $VNetOnpremPrefix -Subnet $Onpremsub,$GWOnpremsub
```

Solicite que se asigne una dirección IP pública a la puerta de enlace que creará para la red virtual. Observe que *AllocationMethod* es **Dynamic**. No puede especificar la dirección IP que desea usar. Se asigna dinámicamente a la puerta de enlace.

  ```azurepowershell
  $gwOnprempip = New-AzPublicIpAddress -Name $GWOnprempipName -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Dynamic
```

## <a name="configure-and-deploy-the-firewall"></a>Configuración e implementación del firewall

Ahora, implemente el firewall en la red virtual de centro.

```azurepowershell
# Get a Public IP for the firewall
$FWpip = New-AzPublicIpAddress -Name "fw-pip" -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Static -Sku Standard
# Create the firewall
$Azfw = New-AzFirewall -Name AzFW01 -ResourceGroupName $RG1 -Location $Location1 -VirtualNetworkName $VNetnameHub -PublicIpName fw-pip

#Save the firewall private IP address for future use

$AzfwPrivateIP = $Azfw.IpConfigurations.privateipaddress
$AzfwPrivateIP

```

### <a name="configure-network-rules"></a>Configuración de reglas de red

<!--- $Rule3 = New-AzFirewallNetworkRule -Name "AllowPing" -Protocol ICMP -SourceAddress $SNOnpremPrefix `
   -DestinationAddress $VNetSpokePrefix -DestinationPort *--->

```azurepowershell
$Rule1 = New-AzFirewallNetworkRule -Name "AllowWeb" -Protocol TCP -SourceAddress $SNOnpremPrefix `
   -DestinationAddress $VNetSpokePrefix -DestinationPort 80

$Rule2 = New-AzFirewallNetworkRule -Name "AllowRDP" -Protocol TCP -SourceAddress $SNOnpremPrefix `
   -DestinationAddress $VNetSpokePrefix -DestinationPort 3389

$NetRuleCollection = New-AzFirewallNetworkRuleCollection -Name RCNet01 -Priority 100 `
   -Rule $Rule1,$Rule2 -ActionType "Allow"
$Azfw.NetworkRuleCollections = $NetRuleCollection
Set-AzFirewall -AzureFirewall $Azfw
```

## <a name="create-and-connect-the-vpn-gateways"></a>Creación y conexión de las puertas de enlace de VPN

Las redes virtuales de centro y local están conectadas mediante puertas de enlace VPN.

### <a name="create-a-vpn-gateway-for-the-hub-virtual-network"></a>Creación de una puerta de enlace VPN para la red virtual de centro

Establezca la configuración de la puerta de enlace de VPN. La configuración de la puerta de enlace de VPN define la subred y la dirección IP pública.

  ```azurepowershell
  $vnet1 = Get-AzVirtualNetwork -Name $VNetnameHub -ResourceGroupName $RG1
  $subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
  $gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfNameHub `
  -Subnet $subnet1 -PublicIpAddress $gwpip1
  ```

Cree la puerta de enlace VPN para la red virtual de centro. Las configuraciones de red a red requieren un VpnType RouteBased. La creación de una puerta de enlace de VPN suele tardar 45 minutos o más, según la SKU de la puerta de enlace de VPN seleccionada.

```azurepowershell
New-AzVirtualNetworkGateway -Name $GWHubName -ResourceGroupName $RG1 `
-Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn `
-VpnType RouteBased -GatewaySku basic
```

### <a name="create-a-vpn-gateway-for-the-on-premises-virtual-network"></a>Creación de una puerta de enlace VPN para la red virtual local

Establezca la configuración de la puerta de enlace de VPN. La configuración de la puerta de enlace de VPN define la subred y la dirección IP pública.

  ```azurepowershell
$vnet2 = Get-AzVirtualNetwork -Name $VNetnameOnprem -ResourceGroupName $RG1
$subnet2 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gwipconf2 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfNameOnprem `
  -Subnet $subnet2 -PublicIpAddress $gwOnprempip
  ```

Cree la puerta de enlace VPN para la red virtual local. Las configuraciones de red a red requieren un VpnType RouteBased. La creación de una puerta de enlace de VPN suele tardar 45 minutos o más, según la SKU de la puerta de enlace de VPN seleccionada.

```azurepowershell
New-AzVirtualNetworkGateway -Name $GWOnpremName -ResourceGroupName $RG1 `
-Location $Location1 -IpConfigurations $gwipconf2 -GatewayType Vpn `
-VpnType RouteBased -GatewaySku basic
```

### <a name="create-the-vpn-connections"></a>Creación de las conexiones

Ahora puede crear las conexiones de VPN entre las puertas de enlace de centro y local.

#### <a name="get-the-vpn-gateways"></a>Obtención de las puertas de enlace de VPN

```azurepowershell
$vnetHubgw = Get-AzVirtualNetworkGateway -Name $GWHubName -ResourceGroupName $RG1
$vnetOnpremgw = Get-AzVirtualNetworkGateway -Name $GWOnpremName -ResourceGroupName $RG1
```

#### <a name="create-the-connections"></a>Crear las conexiones

En este paso va a crear la conexión entre la red virtual de centro y la red virtual local. Verá una clave compartida a la que se hace referencia en los ejemplos. Puede utilizar sus propios valores para la clave compartida. Lo importante es que la clave compartida coincida en ambas conexiones. Se tardará unos momentos en terminar de crear la conexión.

```azurepowershell
New-AzVirtualNetworkGatewayConnection -Name $ConnectionNameHub -ResourceGroupName $RG1 `
-VirtualNetworkGateway1 $vnetHubgw -VirtualNetworkGateway2 $vnetOnpremgw -Location $Location1 `
-ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
```
Cree la conexión de red entre la red virtual local y la red virtual de centro. Este paso es similar al anterior, excepto que se crea una conexión desde la red virtual local a la red virtual de centro. Asegúrese de que coincidan las claves compartidas. Después de unos minutos, se habrá establecido la conexión.

  ```azurepowershell
  New-AzVirtualNetworkGatewayConnection -Name $ConnectionNameOnprem -ResourceGroupName $RG1 `
  -VirtualNetworkGateway1 $vnetOnpremgw -VirtualNetworkGateway2 $vnetHubgw -Location $Location1 `
  -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
  ```

#### <a name="verify-the-connection"></a>Comprobación de la conexión

Para comprobar que la conexión se realizó correctamente, use el cmdlet *Get-AzVirtualNetworkGatewayConnection*, con o sin *-Debug*. Puede usar el siguiente ejemplo de cmdlet, configurando los valores para que coincidan con los tuyos. Cuando se le pida, seleccione **A** para ejecutar **Todo**. En el ejemplo, *-Name* es el nombre de la conexión que desea probar.

```azurepowershell
Get-AzVirtualNetworkGatewayConnection -Name $ConnectionNameHub -ResourceGroupName $RG1
```

Cuando el cmdlet termine, consulte los valores. En el ejemplo siguiente, el estado de conexión aparece como *Connected* (Conectado) y pueden verse los bytes de entrada y salida.

```
"connectionStatus": "Connected",
"ingressBytesTransferred": 33509044,
"egressBytesTransferred": 4142431
```

## <a name="peer-the-hub-and-spoke-virtual-networks"></a>Emparejar las redes virtuales de tipo hub-and-spoke

Ahora, empareje las redes virtuales de tipo hub-and-spoke.

```azurepowershell
# Peer hub to spoke
Add-AzVirtualNetworkPeering -Name HubtoSpoke -VirtualNetwork $VNetHub -RemoteVirtualNetworkId $VNetSpoke.Id -AllowGatewayTransit

# Peer spoke to hub
Add-AzVirtualNetworkPeering -Name SpoketoHub -VirtualNetwork $VNetSpoke -RemoteVirtualNetworkId $VNetHub.Id -AllowForwardedTraffic -UseRemoteGateways
```

## <a name="create-the-routes"></a>Creación de las rutas

A continuación, cree un par de rutas:

- Una ruta desde la subred de puerta de enlace de concentrador a la subred de radio mediante la dirección IP del firewall.
- Una ruta predeterminada desde la subred de radio mediante la dirección IP del firewall.

```azurepowershell
#Create a route table
$routeTableHubSpoke = New-AzRouteTable `
  -Name 'UDR-Hub-Spoke' `
  -ResourceGroupName $RG1 `
  -location $Location1

#Create a route
Get-AzRouteTable `
  -ResourceGroupName $RG1 `
  -Name UDR-Hub-Spoke `
  | Add-AzRouteConfig `
  -Name "ToSpoke" `
  -AddressPrefix $VNetSpokePrefix `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress $AzfwPrivateIP `
 | Set-AzRouteTable

#Associate the route table to the subnet

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNetHub `
  -Name $SNnameGW `
  -AddressPrefix $SNGWHubPrefix `
  -RouteTable $routeTableHubSpoke | `
Set-AzVirtualNetwork

#Now create the default route

#Create a table, with BGP route propagation disabled. The property is now called "Virtual network gateway route propagation," but the API still refers to the parameter as "DisableBgpRoutePropagation."
$routeTableSpokeDG = New-AzRouteTable `
  -Name 'UDR-DG' `
  -ResourceGroupName $RG1 `
  -location $Location1 `
  -DisableBgpRoutePropagation

#Create a route
Get-AzRouteTable `
  -ResourceGroupName $RG1 `
  -Name UDR-DG `
  | Add-AzRouteConfig `
  -Name "ToFirewall" `
  -AddressPrefix 0.0.0.0/0 `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress $AzfwPrivateIP `
 | Set-AzRouteTable

#Associate the route table to the subnet

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNetSpoke `
  -Name $SNnameSpoke `
  -AddressPrefix $SNSpokePrefix `
  -RouteTable $routeTableSpokeDG | `
Set-AzVirtualNetwork
```

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Ahora, cree la carga de trabajo de tipo hub-and-spoke y las máquinas virtuales locales, y colóquelas en las subredes adecuadas.

### <a name="create-the-workload-virtual-machine"></a>Creación de la máquina virtual de cargas de trabajo

Cree una máquina virtual en la red virtual de tipo hub-and-spoke, que ejecute IIS, sin ninguna dirección IP pública y que permita que se haga ping en ella.
Cuando se le solicite, escriba el nombre de usuario y la contraseña de la máquina virtual.

```azurepowershell
# Create an inbound network security group rule for ports 3389 and 80
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name Allow-RDP  -Protocol Tcp `
  -Direction Inbound -Priority 200 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix $SNSpokePrefix -DestinationPortRange 3389 -Access Allow
$nsgRuleWeb = New-AzNetworkSecurityRuleConfig -Name Allow-web  -Protocol Tcp `
  -Direction Inbound -Priority 202 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix $SNSpokePrefix -DestinationPortRange 80 -Access Allow

# Create a network security group
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $RG1 -Location $Location1 -Name NSG-Spoke02 -SecurityRules $nsgRuleRDP,$nsgRuleWeb

#Create the NIC
$NIC = New-AzNetworkInterface -Name spoke-01 -ResourceGroupName $RG1 -Location $Location1 -SubnetId $VnetSpoke.Subnets[0].Id -NetworkSecurityGroupId $nsg.Id

#Define the virtual machine
$VirtualMachine = New-AzVMConfig -VMName VM-Spoke-01 -VMSize "Standard_DS2"
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName Spoke-01 -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2016-Datacenter' -Version latest

#Create the virtual machine
New-AzVM -ResourceGroupName $RG1 -Location $Location1 -VM $VirtualMachine -Verbose

#Install IIS on the VM
Set-AzVMExtension `
    -ResourceGroupName $RG1 `
    -ExtensionName IIS `
    -VMName VM-Spoke-01 `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server"}' `
    -Location $Location1
```

<!---#Create a host firewall rule to allow ping in
Set-AzVMExtension `
    -ResourceGroupName $RG1 `
    -ExtensionName IIS `
    -VMName VM-Spoke-01 `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4"}' `
    -Location $Location1--->

### <a name="create-the-on-premises-virtual-machine"></a>Creación de la máquina virtual local

Se trata de una máquina virtual sencilla con una dirección IP pública a la que se puede conectar mediante Escritorio remoto. Desde allí, puede conectarse al servidor local a través del firewall. Cuando se le solicite, escriba el nombre de usuario y la contraseña de la máquina virtual.

```azurepowershell
New-AzVm `
    -ResourceGroupName $RG1 `
    -Name "VM-Onprem" `
    -Location $Location1 `
    -VirtualNetworkName $VNetnameOnprem `
    -SubnetName $SNNameOnprem `
    -OpenPorts 3389 `
    -Size "Standard_DS2"
```

## <a name="test-the-firewall"></a>Probar el firewall

En primer lugar, obtenga y anote la dirección IP privada de la máquina virtual **VM-spoke-01**.

```azurepowershell
$NIC.IpConfigurations.privateipaddress
```

En Azure Portal, conéctese a la máquina virtual **VM-Onprem**.
<!---2. Open a Windows PowerShell command prompt on **VM-Onprem**, and ping the private IP for **VM-spoke-01**.

   You should get a reply.--->
Abra un explorador web en **VM-Onprem** y vaya a http://\<VM-spoke-01 private IP\>.

Verá la página predeterminada de Internet Information Services.

En **VM-Onprem**, abra un escritorio remoto a **VM-spoke-01** en la dirección IP privada.

Se realizará la conexión y podrá iniciar sesión con el nombre de usuario y la contraseña elegidos.

Con ello, ha comprobado que las reglas de firewall funcionan:

<!---- You can ping the server on the spoke VNet.--->
- Puede examinar el servidor web en la red virtual de tipo hub-and-spoke.
- Puede conectarse al servidor en la red virtual de tipo hub-and-spoke mediante RDP.

A continuación, cambie la acción de recopilación de la regla de red del firewall a **Deny** (Denegar) para comprobar que las reglas del firewall funcionan según lo previsto. Ejecute el siguiente script para cambiar la acción de recopilación de la regla a **Deny** (Denegar).

```azurepowershell
$rcNet = $azfw.GetNetworkRuleCollectionByName("RCNet01")
$rcNet.action.type = "Deny"

Set-AzFirewall -AzureFirewall $azfw
```

Ahora, ejecute de nuevo las pruebas. Esta vez, todas producirán un error. Cierre los escritorios remotos existentes antes de probar las reglas modificadas.

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede conservar los recursos relacionados con el firewall para el siguiente tutorial o, si ya no los necesita, eliminar el grupo de recursos **FW-Hybrid-Test** para eliminarlos todos.

## <a name="next-steps"></a>Pasos siguientes

A continuación, puede supervisar los registros de Azure Firewall.

[Tutorial: Supervisión de los registros de Azure Firewall](./tutorial-diagnostics.md)
