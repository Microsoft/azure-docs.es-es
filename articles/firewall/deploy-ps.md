---
title: Implementación y configuración de Azure Firewall mediante Azure PowerShell
description: En este artículo, aprenderá a implementar y configurar Azure Firewall mediante Azure PowerShell.
services: firewall
author: vhorne
ms.service: firewall
ms.date: 08/28/2020
ms.author: victorh
ms.topic: how-to
ms.openlocfilehash: c720d7c261421ade9dfce01f0b116123dcab1e55
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89071710"
---
# <a name="deploy-and-configure-azure-firewall-using-azure-powershell"></a>Implementación y configuración de Azure Firewall mediante Azure PowerShell

El control del acceso de red saliente es una parte importante de un plan de seguridad de red de ámbito general. Por ejemplo, es posible que desee limitar el acceso a sitios web. O bien, que desee limitar las direcciones IP de salida y los puertos a los que se puede acceder.

Una manera de controlar el acceso de red saliente desde una subred de Azure es con Azure Firewall. Con Azure Firewall, puede configurar:

* Reglas de aplicación que definen los nombres de dominio completos (FQDN) a los que se puede acceder desde una subred.
* Reglas de red que definen la dirección de origen, el protocolo, el puerto de destino y la dirección de destino.

El tráfico está sujeto a las reglas de firewall configuradas cuando enruta el tráfico al firewall como puerta de enlace predeterminada de la subred.

En este artículo, creará una red virtual única simplificada con tres subredes para facilitar la implementación. Para las implementaciones de producción, se recomienda un [modelo de concentrador y radio](https://docs.microsoft.com/azure/architecture/reference-architectures/hybrid-networking/hub-spoke), en el que el firewall está en su propia red virtual. Los servidores de las cargas de trabajo están en redes virtuales emparejadas en la misma región con una o varias subredes.

* **AzureFirewallSubnet**: el firewall está en esta subred.
* **Workload-SN**: el servidor de carga de trabajo está en esta subred. El tráfico de red de esta subred va a través del firewall.
* **Jump-SN**: el servidor de "salto" está en esta subred. El servidor de salto tiene una dirección IP pública a la que puede conectarse mediante Escritorio remoto. Desde allí se puede conectar posteriormente al servidor de carga de trabajo (mediante otro Escritorio remoto).

![Infraestructura de red del tutorial](media/tutorial-firewall-rules-portal/Tutorial_network.png)

En este artículo aprenderá a:


* Configurar un entorno de red de prueba
* Implementar un firewall
* Crear una ruta predeterminada
* Configurar una regla de aplicación para permitir el acceso a www.google.com
* Configuración de una regla de red para permitir el acceso a los servidores DNS externos
* Probar el firewall

Si lo prefiere, puede realizar los pasos de este procedimiento mediante [Azure Portal](tutorial-firewall-deploy-portal.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

En este procedimiento es necesario ejecutar PowerShell de forma local. Debe tener instalados el módulo de Azure PowerShell. Ejecute `Get-Module -ListAvailable Az` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-Az-ps). Después de verificar la versión de PowerShell, ejecute `Connect-AzAccount` para crear una conexión con Azure.

## <a name="set-up-the-network"></a>Configuración de la red

En primer lugar, cree un grupo de recursos para que contenga los recursos necesarios para implementar el firewall. A continuación, cree una red virtual, subredes y pruebe los servidores.

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

El grupo de recursos contiene todos los recursos necesarios para la implementación.

```azurepowershell
New-AzResourceGroup -Name Test-FW-RG -Location "East US"
```

### <a name="create-a-vnet"></a>Creación de una red virtual

Esta red virtual tiene tres subredes:

> [!NOTE]
> El tamaño de la subred AzureFirewallSubnet es /26. Para más información sobre el tamaño de la subred, consulte [Preguntas más frecuentes sobre Azure Firewall](firewall-faq.md#why-does-azure-firewall-need-a-26-subnet-size).

```azurepowershell
$FWsub = New-AzVirtualNetworkSubnetConfig -Name AzureFirewallSubnet -AddressPrefix 10.0.1.0/26
$Worksub = New-AzVirtualNetworkSubnetConfig -Name Workload-SN -AddressPrefix 10.0.2.0/24
$Jumpsub = New-AzVirtualNetworkSubnetConfig -Name Jump-SN -AddressPrefix 10.0.3.0/24
```
Ahora cree la red virtual:

```azurepowershell
$testVnet = New-AzVirtualNetwork -Name Test-FW-VN -ResourceGroupName Test-FW-RG `
-Location "East US" -AddressPrefix 10.0.0.0/16 -Subnet $FWsub, $Worksub, $Jumpsub
```

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Ahora cree las máquinas virtuales de salto y de cargas de trabajo, y colóquelas en las subredes adecuadas.
Cuando se le solicite, escriba el nombre de usuario y la contraseña de la máquina virtual.

Cree la máquina virtual Srv-Jump.

```azurepowershell
New-AzVm `
    -ResourceGroupName Test-FW-RG `
    -Name "Srv-Jump" `
    -Location "East US" `
    -VirtualNetworkName Test-FW-VN `
    -SubnetName Jump-SN `
    -OpenPorts 3389 `
    -Size "Standard_DS2"
```

Cree una máquina virtual de carga de trabajo sin ninguna dirección IP pública.
Cuando se le solicite, escriba el nombre de usuario y la contraseña de la máquina virtual.

```azurepowershell
#Create the NIC
$NIC = New-AzNetworkInterface -Name Srv-work -ResourceGroupName Test-FW-RG `
 -Location "East US" -Subnetid $testVnet.Subnets[1].Id 

#Define the virtual machine
$VirtualMachine = New-AzVMConfig -VMName Srv-Work -VMSize "Standard_DS2"
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName Srv-Work -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2016-Datacenter' -Version latest

#Create the virtual machine
New-AzVM -ResourceGroupName Test-FW-RG -Location "East US" -VM $VirtualMachine -Verbose
```

## <a name="deploy-the-firewall"></a>Implementación del firewall

Ahora, implemente el firewall en la red virtual.

```azurepowershell
# Get a Public IP for the firewall
$FWpip = New-AzPublicIpAddress -Name "fw-pip" -ResourceGroupName Test-FW-RG `
  -Location "East US" -AllocationMethod Static -Sku Standard
# Create the firewall
$Azfw = New-AzFirewall -Name Test-FW01 -ResourceGroupName Test-FW-RG -Location "East US" -VirtualNetworkName Test-FW-VN -PublicIpName fw-pip

#Save the firewall private IP address for future use

$AzfwPrivateIP = $Azfw.IpConfigurations.privateipaddress
$AzfwPrivateIP
```

Anote la dirección IP privada. Se usará más adelante al crear la ruta predeterminada.

## <a name="create-a-default-route"></a>Crear una ruta predeterminada

Creación de una tabla con la propagación de la ruta BGP deshabilitada

```azurepowershell
$routeTableDG = New-AzRouteTable `
  -Name Firewall-rt-table `
  -ResourceGroupName Test-FW-RG `
  -location "East US" `
  -DisableBgpRoutePropagation

#Create a route
 Add-AzRouteConfig `
  -Name "DG-Route" `
  -RouteTable $routeTableDG `
  -AddressPrefix 0.0.0.0/0 `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress $AzfwPrivateIP `
 | Set-AzRouteTable

#Associate the route table to the subnet

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $testVnet `
  -Name Workload-SN `
  -AddressPrefix 10.0.2.0/24 `
  -RouteTable $routeTableDG | Set-AzVirtualNetwork
```

## <a name="configure-an-application-rule"></a>Configuración de una regla de aplicación

La regla de aplicación permite el acceso saliente a www.google.com.

```azurepowershell
$AppRule1 = New-AzFirewallApplicationRule -Name Allow-Google -SourceAddress 10.0.2.0/24 `
  -Protocol http, https -TargetFqdn www.google.com

$AppRuleCollection = New-AzFirewallApplicationRuleCollection -Name App-Coll01 `
  -Priority 200 -ActionType Allow -Rule $AppRule1

$Azfw.ApplicationRuleCollections.Add($AppRuleCollection)

Set-AzFirewall -AzureFirewall $Azfw
```

Azure Firewall incluye una colección de reglas integradas para FQDN de infraestructura que están permitidos de forma predeterminada. Estos FQDN son específicos para la plataforma y no se pueden usar para otros fines. Para más información, consulte [Nombres de dominio completos de infraestructura](infrastructure-fqdns.md).

## <a name="configure-a-network-rule"></a>Configurar una regla de red

La regla de red permite el acceso saliente a dos direcciones IP en el puerto 53 (DNS).

```azurepowershell
$NetRule1 = New-AzFirewallNetworkRule -Name "Allow-DNS" -Protocol UDP -SourceAddress 10.0.2.0/24 `
   -DestinationAddress 209.244.0.3,209.244.0.4 -DestinationPort 53

$NetRuleCollection = New-AzFirewallNetworkRuleCollection -Name RCNet01 -Priority 200 `
   -Rule $NetRule1 -ActionType "Allow"

$Azfw.NetworkRuleCollections.Add($NetRuleCollection)

Set-AzFirewall -AzureFirewall $Azfw
```

### <a name="change-the-primary-and-secondary-dns-address-for-the-srv-work-network-interface"></a>Cambio de la dirección DNS principal y secundaria para la interfaz de red **Srv-Work**

Con fines de prueba para este procedimiento, configure las direcciones DNS principal y secundaria del servidor. Esto no es un requisito general de Azure Firewall.

```azurepowershell
$NIC.DnsSettings.DnsServers.Add("209.244.0.3")
$NIC.DnsSettings.DnsServers.Add("209.244.0.4")
$NIC | Set-AzNetworkInterface
```

## <a name="test-the-firewall"></a>Probar el firewall

Ahora, pruebe el firewall para confirmar que funciona según lo previsto.

1. Anote la dirección IP privada de la máquina virtual **Srv-Work**:

   ```
   $NIC.IpConfigurations.PrivateIpAddress
   ```

1. Conecte un escritorio remoto a la máquina virtual **Srv-Jump** e inicie sesión. Desde ahí, abra una conexión del escritorio remoto a la dirección IP privada de **Srv-Work** e inicie sesión.

3. En **SRV-Work**, abra una ventana de PowerShell y ejecute los comandos siguientes:

   ```
   nslookup www.google.com
   nslookup www.microsoft.com
   ```

   Ambos comandos deben devolver respuestas en las que se muestre que sus consultas de DNS están pasando por el firewall.

1. Ejecute los comandos siguientes:

   ```
   Invoke-WebRequest -Uri https://www.google.com
   Invoke-WebRequest -Uri https://www.google.com

   Invoke-WebRequest -Uri https://www.microsoft.com
   Invoke-WebRequest -Uri https://www.microsoft.com
   ```

   Las solicitudes de `www.google.com` deberían realizarse correctamente, mientras que las solicitudes de `www.microsoft.com` deberían producir un error. Esto demostraría que las reglas de firewall están funcionando según lo previsto.

Con ello, ha comprobado que las reglas de firewall funcionan:

* Puede resolver nombres DNS con el servidor DNS externo configurado.
* Puede navegar al FQDN permitido pero no a ningún otro.

## <a name="clean-up-resources"></a>Limpieza de recursos

Puede conservar los recursos relacionados con el firewall para el siguiente tutorial o, si ya no los necesita, eliminar el grupo de recursos **Test-FW-RG** para eliminarlos todos:

```azurepowershell
Remove-AzResourceGroup -Name Test-FW-RG
```

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Supervisión de los registros de Azure Firewall](./tutorial-diagnostics.md)
