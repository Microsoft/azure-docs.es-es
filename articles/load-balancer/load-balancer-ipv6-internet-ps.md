---
title: Creación de un equilibrador de carga con conexión a Internet con IPv6-Azure PowerShell
titleSuffix: Azure Load Balancer
description: Aprenda a crear un equilibrador de carga orientado a Internet con IPv6 mediante el uso de PowerShell para Resource Manager
services: load-balancer
documentationcenter: na
author: asudbring
keywords: IPv6, Azure Load Balancer, pila doble, dirección ip pública, ipv6 nativo, móvil, iot
ms.service: load-balancer
ms.custom: seodec18
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: allensu
ms.openlocfilehash: 43203a756bcb42c7d00de9c11e9223f1d8b9e2a8
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87001594"
---
# <a name="get-started-creating-an-internet-facing-load-balancer-with-ipv6-using-powershell-for-resource-manager"></a>Introducción a la creación de un equilibrador de carga orientado a Internet con IPv6 mediante el uso de PowerShell para Resource Manager

> [!div class="op_single_selector"]
> * [PowerShell](load-balancer-ipv6-internet-ps.md)
> * [CLI de Azure](load-balancer-ipv6-internet-cli.md)
> * [Plantilla](load-balancer-ipv6-internet-template.md)

>[!NOTE] 
>En este artículo se describe una característica de IPv6 introductoria que permite que los equilibradores de carga básicos proporcionen conectividad IPv4 e IPv6. Ahora hay disponible conectividad IPv6 completa con [IPv6 para redes virtuales de Azure](../virtual-network/ipv6-overview.md) que integra conectividad IPv6 con las redes virtuales e incluye características clave como las reglas de grupo de seguridad de red IPv6, el enrutamiento definido por el usuario IPv6, el equilibrio de carga de IPv6 básico y estándar, y mucho más.  IPv6 para redes virtuales de Azure es el estándar recomendado para las aplicaciones IPv6 en Azure. Vea [IPv6 para la implementación de PowerShell de red virtual de Azure](../virtual-network/virtual-network-ipv4-ipv6-dual-stack-standard-load-balancer-powershell.md). 

Azure Load Balancer es un equilibrador de carga de nivel 4 (TCP y UDP) que distribuye proporcionando una alta disponibilidad el tráfico entrante entre las instancias de servicio correctas de los servicios en la nube o las máquinas virtuales de un conjunto de carga equilibrada. Azure Load Balancer también pueden presentar prestar servicios en varios puertos, varias direcciones IP o ambos.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="example-deployment-scenario"></a>Escenario de implementación de ejemplo

En el siguiente diagrama se ilustra la solución de equilibrio de carga que se implementa en este artículo.

![Escenario del equilibrador de carga](./media/load-balancer-ipv6-internet-ps/lb-ipv6-scenario.png)

En este escenario creará los siguientes recursos de Azure:

* un equilibrador de carga orientado a Internet con una dirección IP pública IPv4 e IPv6
* dos reglas de equilibrio de carga para asignar las VIP públicas a los puntos de conexión privados
* un conjunto de disponibilidad con las dos máquinas virtuales
* dos máquinas virtuales (VM)
* una interfaz de red virtual para cada máquina virtual con las direcciones IPv4 e IPv6 asignadas

## <a name="deploying-the-solution-using-the-azure-powershell"></a>Implementación de la solución mediante Azure PowerShell

Los pasos siguientes muestran cómo crear un equilibrador de carga orientado a Internet mediante Azure Resource Manager con PowerShell. Con Azure Resource Manager, cada recurso se crea y configura individualmente y, a continuación, se colocan juntos para crear un recurso.

Para implementar un equilibrador de carga, debe crear y configurar los siguientes objetos:

* Configuración de direcciones IP de front-end: contiene direcciones IP públicas para el tráfico de red entrante.
* Grupo de direcciones de back-end: contiene interfaces de red (NIC) para que las máquinas virtuales reciban tráfico de red del equilibrador de carga.
* Reglas de equilibrio de carga: contiene reglas que asignan un puerto público en el equilibrador de carga al del grupo de direcciones de back-end.
* Reglas NAT de entrada: contiene reglas que asignan un puerto público en el equilibrador de carga a un puerto para una máquina virtual específica en el grupo de direcciones de back-end.
* Sondeos: contiene los sondeos de estado que se usan para comprobar la disponibilidad de las instancias de las máquinas virtuales del grupo de direcciones de back-end.

Para más información, vea [Componentes de Azure Load Balancer](./components.md).

## <a name="set-up-powershell-to-use-resource-manager"></a>Configuración de PowerShell para usar Resource Manager

Asegúrese de que tiene la última versión de producción del módulo Azure Resource Manager para PowerShell.

1. Inicio de sesión en Azure

    ```azurepowershell-interactive
    Connect-AzAccount
    ```

    Escriba sus credenciales cuando se le solicite.

2. Compruebe las suscripciones para la cuenta.

   ```azurepowershell-interactive
    Get-AzSubscription
    ```

3. Elección de la suscripción de Azure que se va a usar.

    ```azurepowershell-interactive
    Select-AzSubscription -SubscriptionId 'GUID of subscription'
    ```

4. Creación de un grupo de recursos (omitir este paso si se utiliza un grupo de recursos existente)

    ```azurepowershell-interactive
    New-AzResourceGroup -Name NRP-RG -location "West US"
    ```

## <a name="create-a-virtual-network-and-a-public-ip-address-for-the-front-end-ip-pool"></a>Creación de una red virtual y una dirección IP pública para el grupo de direcciones IP front-end

1. Cree una red virtual con una subred

    ```azurepowershell-interactive
    $backendSubnet = New-AzVirtualNetworkSubnetConfig -Name LB-Subnet-BE -AddressPrefix 10.0.2.0/24
    $vnet = New-AzvirtualNetwork -Name VNet -ResourceGroupName NRP-RG -Location 'West US' -AddressPrefix 10.0.0.0/16 -Subnet $backendSubnet
    ```

2. Cree recursos de dirección IP pública (PIP) de Azure para el grupo de direcciones IP de front-end No olvide cambiar el valor de `-DomainNameLabel` antes de ejecutar los comandos siguientes. El valor debe ser único dentro de la región de Azure.

    ```azurepowershell-interactive
    $publicIPv4 = New-AzPublicIpAddress -Name 'pub-ipv4' -ResourceGroupName NRP-RG -Location 'West US' -AllocationMethod Static -IpAddressVersion IPv4 -DomainNameLabel lbnrpipv4
    $publicIPv6 = New-AzPublicIpAddress -Name 'pub-ipv6' -ResourceGroupName NRP-RG -Location 'West US' -AllocationMethod Dynamic -IpAddressVersion IPv6 -DomainNameLabel lbnrpipv6
    ```

    > [!IMPORTANT]
    > El equilibrador de carga usa la etiqueta de dominio de la dirección IP pública como prefijo para su FQDN. En este ejemplo, los FQDN son *lbnrpipv4.westus.cloudapp.azure.com* y *lbnrpipv6.westus.cloudapp.azure.com*.

## <a name="create-a-front-end-ip-configurations-and-a-back-end-address-pool"></a>Creación de configuraciones de direcciones IP de front-end y un grupo de direcciones de back-end

1. Cree la configuración de direcciones de front-end que usa las direcciones IP públicas que ha creado

    ```azurepowershell-interactive
    $FEIPConfigv4 = New-AzLoadBalancerFrontendIpConfig -Name "LB-Frontendv4" -PublicIpAddress $publicIPv4
    $FEIPConfigv6 = New-AzLoadBalancerFrontendIpConfig -Name "LB-Frontendv6" -PublicIpAddress $publicIPv6
    ```

2. Cree grupos de direcciones de back-end

    ```azurepowershell-interactive
    $backendpoolipv4 = New-AzLoadBalancerBackendAddressPoolConfig -Name "BackendPoolIPv4"
    $backendpoolipv6 = New-AzLoadBalancerBackendAddressPoolConfig -Name "BackendPoolIPv6"
    ```

## <a name="create-lb-rules-nat-rules-a-probe-and-a-load-balancer"></a>Crear reglas de equilibrador de carga, reglas NAT, un sondeo y un equilibrador de carga

En este ejemplo se crean los siguientes elementos:

* una regla NAT para trasladar todo el tráfico entrante del puerto 443 al puerto 4443
* Una regla de equilibrador de carga para equilibrar todo el tráfico entrante del puerto 80 al puerto 80 en las direcciones del grupo de back-end.
* Una regla de equilibrador de carga para permitir la conexión RDP a las máquinas virtuales en el puerto 3389.
* una regla de sondeo para comprobar el estado de mantenimiento en una página llamada *HealthProbe.aspx* o un servicio en el puerto 8080
* un equilibrador de carga que usa todos estos objetos

1. Cree las reglas NAT.

    ```azurepowershell-interactive
    $inboundNATRule1v4 = New-AzLoadBalancerInboundNatRuleConfig -Name "NicNatRulev4" -FrontendIpConfiguration $FEIPConfigv4 -Protocol TCP -FrontendPort 443 -BackendPort 4443
    $inboundNATRule1v6 = New-AzLoadBalancerInboundNatRuleConfig -Name "NicNatRulev6" -FrontendIpConfiguration $FEIPConfigv6 -Protocol TCP -FrontendPort 443 -BackendPort 4443
    ```

2. Cree un sondeo de estado. Hay dos formas de configurar un sondeo:

    Sondeo HTTP

    ```azurepowershell-interactive
    $healthProbe = New-AzLoadBalancerProbeConfig -Name 'HealthProbe-v4v6' -RequestPath 'HealthProbe.aspx' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2
    ```

    o sondeo TCP

    ```azurepowershell-interactive
    $healthProbe = New-AzLoadBalancerProbeConfig -Name 'HealthProbe-v4v6' -Protocol Tcp -Port 8080 -IntervalInSeconds 15 -ProbeCount 2
    $RDPprobe = New-AzLoadBalancerProbeConfig -Name 'RDPprobe' -Protocol Tcp -Port 3389 -IntervalInSeconds 15 -ProbeCount 2
    ```

    En este ejemplo, vamos a usar los sondeos TCP.

3. Cree una regla de equilibrador de carga.

    ```azurepowershell-interactive
    $lbrule1v4 = New-AzLoadBalancerRuleConfig -Name "HTTPv4" -FrontendIpConfiguration $FEIPConfigv4 -BackendAddressPool $backendpoolipv4 -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 8080
    $lbrule1v6 = New-AzLoadBalancerRuleConfig -Name "HTTPv6" -FrontendIpConfiguration $FEIPConfigv6 -BackendAddressPool $backendpoolipv6 -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 8080
    $RDPrule = New-AzLoadBalancerRuleConfig -Name "RDPrule" -FrontendIpConfiguration $FEIPConfigv4 -BackendAddressPool $backendpoolipv4 -Probe $RDPprobe -Protocol Tcp -FrontendPort 3389 -BackendPort 3389
    ```

4. Cree el equilibrador de carga mediante los objetos creados anteriormente

    ```azurepowershell-interactive
    $NRPLB = New-AzLoadBalancer -ResourceGroupName NRP-RG -Name 'myNrpIPv6LB' -Location 'West US' -FrontendIpConfiguration $FEIPConfigv4,$FEIPConfigv6 -InboundNatRule $inboundNATRule1v6,$inboundNATRule1v4 -BackendAddressPool $backendpoolipv4,$backendpoolipv6 -Probe $healthProbe,$RDPprobe -LoadBalancingRule $lbrule1v4,$lbrule1v6,$RDPrule
    ```

## <a name="create-nics-for-the-back-end-vms"></a>Creación de tarjetas NIC para las máquinas virtuales de back-end

1. Obtenga la red virtual y la subred de red virtual, donde deben crearse las NIC.

    ```azurepowershell-interactive
    $vnet = Get-AzVirtualNetwork -Name VNet -ResourceGroupName NRP-RG
    $backendSubnet = Get-AzVirtualNetworkSubnetConfig -Name LB-Subnet-BE -VirtualNetwork $vnet
    ```

2. Cree configuraciones de direcciones IP y tarjetas NIC para las máquinas virtuales

    ```azurepowershell-interactive
    $nic1IPv4 = New-AzNetworkInterfaceIpConfig -Name "IPv4IPConfig" -PrivateIpAddressVersion "IPv4" -Subnet $backendSubnet -LoadBalancerBackendAddressPool $backendpoolipv4 -LoadBalancerInboundNatRule $inboundNATRule1v4
    $nic1IPv6 = New-AzNetworkInterfaceIpConfig -Name "IPv6IPConfig" -PrivateIpAddressVersion "IPv6" -LoadBalancerBackendAddressPool $backendpoolipv6 -LoadBalancerInboundNatRule $inboundNATRule1v6
    $nic1 = New-AzNetworkInterface -Name 'myNrpIPv6Nic0' -IpConfiguration $nic1IPv4,$nic1IPv6 -ResourceGroupName NRP-RG -Location 'West US'

    $nic2IPv4 = New-AzNetworkInterfaceIpConfig -Name "IPv4IPConfig" -PrivateIpAddressVersion "IPv4" -Subnet $backendSubnet -LoadBalancerBackendAddressPool $backendpoolipv4
    $nic2IPv6 = New-AzNetworkInterfaceIpConfig -Name "IPv6IPConfig" -PrivateIpAddressVersion "IPv6" -LoadBalancerBackendAddressPool $backendpoolipv6
    $nic2 = New-AzNetworkInterface -Name 'myNrpIPv6Nic1' -IpConfiguration $nic2IPv4,$nic2IPv6 -ResourceGroupName NRP-RG -Location 'West US'
    ```

## <a name="create-virtual-machines-and-assign-the-newly-created-nics"></a>Creación de máquinas virtuales y asignación de las tarjetas NIC recién creadas

Para obtener más información sobre la creación de una máquina virtual, consulte [Creación de una máquina virtual de Windows con Resource Manager y Azure PowerShell](../virtual-machines/windows/quick-create-powershell.md?toc=%2fazure%2fload-balancer%2ftoc.json)

1. Cree un conjunto de disponibilidad y una cuenta de almacenamiento

    ```azurepowershell-interactive
    New-AzAvailabilitySet -Name 'myNrpIPv6AvSet' -ResourceGroupName NRP-RG -location 'West US'
    $availabilitySet = Get-AzAvailabilitySet -Name 'myNrpIPv6AvSet' -ResourceGroupName NRP-RG
    New-AzStorageAccount -ResourceGroupName NRP-RG -Name 'mynrpipv6stacct' -Location 'West US' -SkuName "Standard_LRS"
    $CreatedStorageAccount = Get-AzStorageAccount -ResourceGroupName NRP-RG -Name 'mynrpipv6stacct'
    ```

2. Cree cada máquina virtual y asigne las tarjetas NIC creadas con anterioridad

    ```azurepowershell-interactive
    $mySecureCredentials= Get-Credential -Message "Type the username and password of the local administrator account."

    $vm1 = New-AzVMConfig -VMName 'myNrpIPv6VM0' -VMSize 'Standard_G1' -AvailabilitySetId $availabilitySet.Id
    $vm1 = Set-AzVMOperatingSystem -VM $vm1 -Windows -ComputerName 'myNrpIPv6VM0' -Credential $mySecureCredentials -ProvisionVMAgent -EnableAutoUpdate
    $vm1 = Set-AzVMSourceImage -VM $vm1 -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
    $vm1 = Add-AzVMNetworkInterface -VM $vm1 -Id $nic1.Id -Primary
    $osDisk1Uri = $CreatedStorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/myNrpIPv6VM0osdisk.vhd"
    $vm1 = Set-AzVMOSDisk -VM $vm1 -Name 'myNrpIPv6VM0osdisk' -VhdUri $osDisk1Uri -CreateOption FromImage
    New-AzVM -ResourceGroupName NRP-RG -Location 'West US' -VM $vm1

    $vm2 = New-AzVMConfig -VMName 'myNrpIPv6VM1' -VMSize 'Standard_G1' -AvailabilitySetId $availabilitySet.Id
    $vm2 = Set-AzVMOperatingSystem -VM $vm2 -Windows -ComputerName 'myNrpIPv6VM1' -Credential $mySecureCredentials -ProvisionVMAgent -EnableAutoUpdate
    $vm2 = Set-AzVMSourceImage -VM $vm2 -PublisherName MicrosoftWindowsServer -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"
    $vm2 = Add-AzVMNetworkInterface -VM $vm2 -Id $nic2.Id -Primary
    $osDisk2Uri = $CreatedStorageAccount.PrimaryEndpoints.Blob.ToString() + "vhds/myNrpIPv6VM1osdisk.vhd"
    $vm2 = Set-AzVMOSDisk -VM $vm2 -Name 'myNrpIPv6VM1osdisk' -VhdUri $osDisk2Uri -CreateOption FromImage
    New-AzVM -ResourceGroupName NRP-RG -Location 'West US' -VM $vm2
    ```


