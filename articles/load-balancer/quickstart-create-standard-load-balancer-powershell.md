---
title: 'Guía de inicio rápido: Creación de una instancia de Standard Load Balancer: Azure PowerShell'
titlesuffix: Azure Load Balancer
description: En este inicio rápido se muestra cómo crear una instancia de Standard Load Balancer con PowerShell
services: load-balancer
documentationcenter: na
author: KumudD
manager: twooley
Customer intent: I want to create a Standard Load balancer so that I can load balance internet traffic to VMs.
ms.assetid: ''
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 05/07/2019
ms.author: kumud
ms:custom: seodec18
ms.openlocfilehash: 513d3931c31ca94b4089139992bceb6f679caa98
ms.sourcegitcommit: e6d53649bfb37d01335b6bcfb9de88ac50af23bd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/09/2019
ms.locfileid: "65467691"
---
# <a name="quickstart-create-a-standard-load-balancer-using-azure-powershell"></a>Inicio rápido: Creación de una instancia de Standard Load Balancer mediante Azure PowerShell

En este inicio rápido se muestra cómo crear una instancia de Standard Load Balancer con Azure PowerShell. Para probar el equilibrador de carga, implemente tres máquinas virtuales que ejecuten Windows Server y equilibre la carga de una aplicación web entre ellas. Para más información sobre Standard Load Balancer, consulte [Introducción a Standard Load Balancer](load-balancer-standard-overview.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-powershell.md](../../includes/cloud-shell-powershell.md)]

Si decide instalar y usar PowerShell de forma local, para realizar los pasos de este artículo necesita la versión 5.4.1 del módulo de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para poder crear el equilibrador de carga, debe crear primero un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroupSLB* en la ubicación *EastUS*:

```azurepowershell
$rgName='MyResourceGroupSLB'
$location='eastus'
New-AzResourceGroup -Name $rgName -Location $location
```

## <a name="create-a-public-ip-address"></a>Crear una dirección IP pública

Para obtener acceso a la aplicación en Internet, necesita una dirección IP pública para el equilibrador de carga. Cree una dirección IP pública con [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress). En el ejemplo siguiente, se crea una dirección IP pública denominada *myPublicIP* en el grupo de recursos *myResourceGroupSLB*:

```azurepowershell
$publicIp = New-AzPublicIpAddress `
 -ResourceGroupName $rgName `
 -Name 'myPublicIP' `
 -Location $location `
 -AllocationMethod static `
 -SKU Standard
```

## <a name="create-standard-load-balancer"></a>Creación de Standard Load Balancer

En esta sección, va a configurar la dirección IP de front-end y el grupo de direcciones de back-end para el equilibrador de carga, y, a continuación, creará Standard Load Balancer.

### <a name="create-front-end-ip"></a>Creación de una dirección IP de front-end

Cree una dirección IP de front-end con [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig). En el ejemplo siguiente, se crea una configuración de direcciones IP de front-end llamada *myFrontEnd* y se asocia a la dirección *myPublicIP*:

```azurepowershell
$feip = New-AzLoadBalancerFrontendIpConfig -Name 'myFrontEndPool' -PublicIpAddress $publicIp
```

### <a name="configure-back-end-address-pool"></a>Configuración del grupo de direcciones de back-end

Cree un grupo de direcciones de back-end con [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig). En el resto de los pasos, las máquinas virtuales se conectan a este grupo back-end. En el siguiente ejemplo se crea un grupo de direcciones de back-end denominado *myBackEndPool*:

```azurepowershell-interactive
$bepool = New-AzLoadBalancerBackendAddressPoolConfig -Name 'myBackEndPool'
```

### <a name="create-a-health-probe"></a>Creación de un sondeo de estado
Para permitir que el equilibrador de carga supervise el estado de la aplicación, utilice un sondeo de estado. El sondeo de estado agrega o quita de forma dinámica las máquinas virtuales de la rotación del equilibrador de carga en base a su respuesta a las comprobaciones de estado. De forma predeterminada, una máquina virtual se quita de la distribución del equilibrador de carga después de dos errores consecutivos en un intervalo de 15 segundos. Cree un sondeo de estado en función de un protocolo o una página de comprobación de mantenimiento específica para la aplicación.

En el ejemplo siguiente se crea un sondeo de TCP. También se pueden crear sondeos HTTP personalizados para comprobaciones de estado más específicas. Al usar un sondeo HTTP personalizado, debe crear la página de comprobación de estado, por ejemplo *healthcheck.aspx*. El sondeo debe devolver una respuesta **HTTP 200 OK** para que el equilibrador de carga mantenga el host en rotación.

Para crear un sondeo de estado TCP, se usa [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/add-azloadbalancerprobeconfig). En el ejemplo siguiente, se crea un sondeo de estado llamado *myHealthProbe* que supervisa cada máquina virtual en el puerto *HTTP* *80*:

```azurepowershell
$probe = New-AzLoadBalancerProbeConfig `
 -Name 'myHealthProbe' `
 -Protocol Http -Port 80 `
 -RequestPath / -IntervalInSeconds 360 -ProbeCount 5
```

### <a name="create-a-load-balancer-rule"></a>Creación de una regla de equilibrador de carga
Las reglas de equilibrador de carga se utilizan para definir cómo se distribuye el tráfico a las máquinas virtuales. Se define la configuración de IP front-end para el tráfico entrante y el grupo IP de back-end para recibir el tráfico, junto con el puerto de origen y destino requeridos. Para asegurarse de que solo las máquinas virtuales correctas reciban tráfico, también hay que definir el sondeo de estado que se va usar.

Cree una regla del equilibrador de carga con [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/add-azloadbalancerruleconfig). En el ejemplo siguiente se crea una regla del equilibrador de carga llamada *myLoadBalancerRule* y se equilibra el tráfico en el puerto *TCP* *80*:

```azurepowershell
$rule = New-AzLoadBalancerRuleConfig `
  -Name 'myLoadBalancerRuleWeb' -Protocol Tcp `
  -Probe $probe -FrontendPort 80 -BackendPort 80 `
  -FrontendIpConfiguration $feip `
  -BackendAddressPool $bePool
```

### <a name="create-the-nat-rules"></a>Creación de las reglas NAT

Cree las reglas NAT con [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/new-azloadbalancerinboundnatruleconfig). En el ejemplo siguiente, se crean unas reglas NAT denominadas *myLoadBalancerRDP1* y *myLoadBalancerRDP2* para permitir conexiones RDP con los servidores back-end a través de los puertos 4221 y 4222:

```azurepowershell
$natrule1 = New-AzLoadBalancerInboundNatRuleConfig `
  -Name 'myLoadBalancerRDP1' `
  -FrontendIpConfiguration $feip `
  -Protocol tcp -FrontendPort 4221 `
  -BackendPort 3389

$natrule2 = New-AzLoadBalancerInboundNatRuleConfig `
  -Name 'myLoadBalancerRDP2' `
  -FrontendIpConfiguration $feip `
  -Protocol tcp `
  -FrontendPort 4222 `
  -BackendPort 3389

$natrule3 = New-AzLoadBalancerInboundNatRuleConfig `
  -Name 'myLoadBalancerRDP3' `
  -FrontendIpConfiguration $feip `
  -Protocol tcp `
  -FrontendPort 4223 `
  -BackendPort 3389
```

### <a name="create-load-balancer"></a>Creación de un equilibrador de carga

Cree la instancia de Standard Load Balancer con [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer). En el ejemplo siguiente, se crea una instancia de Standard Load Balancer pública denominada myLoadBalancer usando la configuración de direcciones IP de front-end, el grupo de servidores back-end, el sondeo de estado, la regla de equilibrio de carga y las reglas NAT que creó en los pasos anteriores:

```azurepowershell
$lb = New-AzLoadBalancer `
  -ResourceGroupName $rgName `
  -Name 'MyLoadBalancer' `
  -SKU Standard `
  -Location $location `
  -FrontendIpConfiguration $feip `
  -BackendAddressPool $bepool `
  -Probe $probe `
  -LoadBalancingRule $rule `
  -InboundNatRule $natrule1,$natrule2,$natrule3
```

## <a name="create-network-resources"></a>Crear recursos de red
Para poder implementar algunas máquinas virtuales y probar el equilibrador, debe crear unos recursos de red que lo permitan: una red virtual y varias NIC virtuales. 

### <a name="create-a-virtual-network"></a>Creación de una red virtual
Cree una red virtual con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). En el ejemplo siguiente se crea una red virtual denominada con una subred llamada *myVnet* con *mySubnet*:

```azurepowershell
# Create subnet config
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name "mySubnet" `
  -AddressPrefix 10.0.2.0/24

# Create the virtual network
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName "myResourceGroupSLB" `
  -Location $location `
  -Name "myVnet" `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $subnetConfig
```
### <a name="create-public-ip-addresses-for-the-vms"></a>Creación de direcciones IP públicas para las máquinas virtuales

Para acceder a las máquinas virtuales mediante una conexión RDP, se necesita una dirección IP pública para las máquinas virtuales. Como en este escenario se usa Standard Load Balancer, debe crear las direcciones IP públicas estándar para las máquinas virtuales con [New AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress).

```azurepowershell
$RdpPublicIP_1 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_1" `
  -ResourceGroupName $RgName `
  -Location $location  `
  -SKU Standard `
  -AllocationMethod static
 

$RdpPublicIP_2 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_2" `
  -ResourceGroupName $RgName `
  -Location $location  `
  -SKU Standard `
  -AllocationMethod static


$RdpPublicIP_3 = New-AzPublicIpAddress `
  -Name "RdpPublicIP_3" `
  -ResourceGroupName $RgName `
  -Location $location  `
  -SKU Standard `
  -AllocationMethod static

```
### <a name="create-network-security-group"></a>Creación de un grupo de seguridad de red
Cree un grupo de seguridad de red para definir las conexiones entrantes a la red virtual.

#### <a name="create-a-network-security-group-rule-for-port-3389"></a>Creación de una regla de grupo de seguridad de red para el puerto 3389
Cree una regla de grupo de seguridad de red para permitir conexiones RDP en el puerto 3389 con [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig).

```azurepowershell

$rule1 = New-AzNetworkSecurityRuleConfig -Name 'myNetworkSecurityGroupRuleRDP' -Description 'Allow RDP' `
  -Access Allow -Protocol Tcp -Direction Inbound -Priority 1000 `
  -SourceAddressPrefix Internet -SourcePortRange * `
  -DestinationAddressPrefix * -DestinationPortRange 3389
```

#### <a name="create-a-network-security-group-rule-for-port-80"></a>Creación de una regla de grupo de seguridad de red para el puerto 80
Cree una regla de grupo de seguridad de red para permitir conexiones de entrada en el puerto 80 con [New-AzNetworkSecurityRuleConfig](/powershell/module/az.network/new-aznetworksecurityruleconfig).

```azurepowershell
$rule2 = New-AzNetworkSecurityRuleConfig -Name 'myNetworkSecurityGroupRuleHTTP' -Description 'Allow HTTP' `
  -Access Allow -Protocol Tcp -Direction Inbound -Priority 2000 `
  -SourceAddressPrefix Internet -SourcePortRange * `
  -DestinationAddressPrefix * -DestinationPortRange 80
```

#### <a name="create-a-network-security-group"></a>Crear un grupo de seguridad de red

Cree un grupo de seguridad de red con [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup).

```azurepowershell
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $RgName -Location $location `
-Name 'myNetworkSecurityGroup' -SecurityRules $rule1,$rule2
```

### <a name="create-nics"></a>Creación de tarjetas NIC
Crear NIC virtuales y asócielas a la dirección IP pública y a los grupos de seguridad de red creados en los pasos anteriores con [New AzNetworkInterface](/powershell/module/az.network/new-aznetworkinterface). En el ejemplo siguiente se crean tres NIC. (Una NIC virtual para cada máquina virtual que cree para la aplicación en los pasos siguientes). Puede crear NIC virtuales y máquinas virtuales adicionales en cualquier momento y agregarlas al equilibrador de carga:

```azurepowershell
# Create NIC for VM1
$nicVM1 = New-AzNetworkInterface -ResourceGroupName $rgName -Location $location `
  -Name 'MyNic1' -PublicIpAddress $RdpPublicIP_1 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg `
  -LoadBalancerInboundNatRule $natrule1 -Subnet $vnet.Subnets[0]

$nicVM2 = New-AzNetworkInterface -ResourceGroupName $rgName -Location $location `
  -Name 'MyNic2' -PublicIpAddress $RdpPublicIP_2 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg `
  -LoadBalancerInboundNatRule $natrule2 -Subnet $vnet.Subnets[0]

$nicVM3 = New-AzNetworkInterface -ResourceGroupName $rgName -Location $location `
  -Name 'MyNic3' -PublicIpAddress $RdpPublicIP_3 -LoadBalancerBackendAddressPool $bepool -NetworkSecurityGroup $nsg `
  -LoadBalancerInboundNatRule $natrule3 -Subnet $vnet.Subnets[0]
```

### <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Establezca un nombre de usuario de administrador y una contraseña para las máquinas virtuales con [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential):

```azurepowershell
$cred = Get-Credential
```

Ahora puede crear las máquinas virtuales con [New-AzVM](/powershell/module/az.compute/new-azvm). En el ejemplo siguiente, se crean dos máquinas virtuales y los componentes de red virtual necesarios, si aún no existen. En este ejemplo, las NIC (*MyNic1*, *MyNic2*, y *MyNic3*) creadas en el paso anterior se asignan a las máquinas virtuales *myVM1*, *myVM2* y *VM3*. Además, como las NIC se asocian al grupo de servidores back-end del equilibrador de carga, las máquinas virtuales se agregan automáticamente al grupo de servidores back-end.

```azurepowershell

# ############## VM1 ###############

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig -VMName 'myVM1' -VMSize Standard_DS1_v2 `
 | Set-AzVMOperatingSystem -Windows -ComputerName 'myVM1' -Credential $cred `
 | Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2019-Datacenter -Version latest `
 | Add-AzVMNetworkInterface -Id $nicVM1.Id

# Create a virtual machine
$vm1 = New-AzVM -ResourceGroupName $rgName -Zone 1 -Location $location -VM $vmConfig

# ############## VM2 ###############

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig -VMName 'myVM2' -VMSize Standard_DS1_v2 `
 | Set-AzVMOperatingSystem -Windows -ComputerName 'myVM2' -Credential $cred `
 | Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2019-Datacenter -Version latest `
 | Add-AzVMNetworkInterface -Id $nicVM2.Id

# Create a virtual machine
$vm2 = New-AzVM -ResourceGroupName $rgName -Zone 2 -Location $location -VM $vmConfig

# ############## VM3 ###############

# Create a virtual machine configuration
$vmConfig = New-AzVMConfig -VMName 'myVM3' -VMSize Standard_DS1_v2 `
 | Set-AzVMOperatingSystem -Windows -ComputerName 'myVM3' -Credential $cred `
 | Set-AzVMSourceImage -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2019-Datacenter -Version latest `
| Add-AzVMNetworkInterface -Id $nicVM3.Id

# Create a virtual machine
$vm3 = New-AzVM -ResourceGroupName $rgName -Zone 3 -Location $location -VM $vmConfig
```

Se tarda unos minutos en crear y configurar las tres máquinas virtuales.

### <a name="install-iis-with-a-custom-web-page"></a>Instalación de IIS con una página web personalizada

Siga estos pasos para instalar IIS con una página web personalizada en las dos máquinas virtuales de back-end:

1. Obtenga las direcciones IP públicas de las tres máquinas virtuales mediante `Get-AzPublicIPAddress`.

   ```azurepowershell
     $vm1_rdp_ip = (Get-AzPublicIPAddress -ResourceGroupName $rgName -Name "RdpPublicIP_1").IpAddress
     $vm2_rdp_ip = (Get-AzPublicIPAddress -ResourceGroupName $rgName -Name "RdpPublicIP_2").IpAddress
     $vm3_rdp_ip = (Get-AzPublicIPAddress -ResourceGroupName $rgName -Name "RdpPublicIP_3").IpAddress
    ```
2. Cree conexiones a Escritorio remoto con *myVM1*, *myVM2* y *myVM3* mediante las direcciones IP públicas de las máquinas virtuales como se indica a continuación: 

   ```azurepowershell    
     mstsc /v:$vm1_rdp_ip
     mstsc /v:$vm2_rdp_ip
     mstsc /v:$vm3_rdp_ip
   
    ```

3. Escriba las credenciales de cada máquina virtual para iniciar la sesión de RDP.
4. Inicie Windows PowerShell en cada una de las máquinas virtuales y use los siguientes comandos para instalar el servidor IIS y actualizar el archivo HTM predeterminado.

    ```azurepowershell
    # Install IIS
     Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom htm file
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from host " + $env:computername)
    ```

5. Cierre las conexiones del RDP con *myVM1*, *myVM2* y *myVM3*.


## <a name="test-load-balancer"></a>Prueba del equilibrador de carga
Obtenga la dirección IP pública del equilibrador de carga con [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress). En el ejemplo siguiente se obtiene la dirección IP de *myPublicIP* que se ha creado anteriormente:

```azurepowershell
Get-AzPublicIPAddress `
  -ResourceGroupName "myResourceGroupSLB" `
  -Name "myPublicIP" | select IpAddress
```

A continuación, puede escribir la dirección IP pública en un explorador web. Se muestra el sitio web, incluido el nombre de host de la máquina virtual a la que el equilibrador de carga distribuye el tráfico como en el ejemplo siguiente:

![Prueba del equilibrador de carga](media/quickstart-create-basic-load-balancer-powershell/load-balancer-test.png)

Para ver cómo el equilibrador de carga distribuye el tráfico entre las tres máquinas virtuales que ejecutan la aplicación, puede realizar una actualización forzada del explorador web. 

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no se necesiten, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurepowershell
Remove-AzResourceGroup -Name myResourceGroupSLB
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha creado un equilibrador de carga estándar, le ha asociado máquinas virtuales, ha configurado la regla de tráfico del equilibrador de carga y el sondeo de estado y, después, ha probado el equilibrador de carga. Para más información acerca de Azure Load Balancer, diríjase a los tutoriales de Azure Load Balancer.

> [!div class="nextstepaction"]
> [Tutoriales de Azure Load Balancer](tutorial-load-balancer-basic-internal-portal.md)
