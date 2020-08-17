---
title: Equilibrio de carga en varias configuraciones de IP en la CLI de Azure
titleSuffix: Azure Load Balancer
description: En este artículo, obtendrá información sobre el equilibrio de carga en las configuraciones de IP principales y secundarias mediante la CLI de Azure.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/25/2017
ms.author: allensu
ms.openlocfilehash: e8fedad40c18818932bf37dfe93c1b236357c30b
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87001612"
---
# <a name="load-balancing-on-multiple-ip-configurations-using-powershell"></a>Equilibrio de carga en varias configuraciones de IP mediante PowerShell

> [!div class="op_single_selector"]
> * [Portal](load-balancer-multiple-ip.md)
> * [CLI](load-balancer-multiple-ip-cli.md)
> * [PowerShell](load-balancer-multiple-ip-powershell.md)


En este artículo, se explica cómo se utiliza Azure Load Balancer con varias direcciones IP en una interfaz de red secundaria (NIC). En este escenario, tenemos dos máquinas virtuales que ejecutan Windows. Cada una de ellas cuenta con una NIC principal y otra secundaria. Cada NIC secundario tiene dos configuraciones IP. Cada máquina virtual hospeda dos sitios web: contoso.com y fabrikam.com. Cada uno de los sitios web está enlazado a una de las configuraciones de IP de la NIC secundaria. Usamos Azure Load Balancer para exponer dos direcciones IP front-end, una por cada sitio web, que van a distribuir el tráfico a la configuración de IP correspondiente del sitio web. En este escenario, se utiliza el mismo número de puerto en los dos front-end, así como en las dos direcciones IP del grupo de back-end.

![Imagen del escenario de equilibrio de carga](./media/load-balancer-multiple-ip/lb-multi-ip.PNG)

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="steps-to-load-balance-on-multiple-ip-configurations"></a>Pasos para equilibrar la carga en varias configuraciones de IP

Siga estos pasos para reproducir el escenario que se describe en este artículo:

1. Instale Azure PowerShell. Consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) para obtener más información sobre cómo instalar la versión más reciente de Azure PowerShell, seleccionar la suscripción que quiere usar e iniciar sesión en su cuenta.
2. Cree un grupo de recursos con el siguiente comando:

    ```powershell
    $location = "westcentralus".
    $myResourceGroup = "contosofabrikam"
    ```

    Para más información, consulte el [Paso 2: Creación de un grupo de recursos](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fazure%2fload-balancer%2ftoc.json).

3. [Cree un conjunto de disponibilidad](../virtual-machines/windows/tutorial-availability-sets.md?toc=%2fazure%2fload-balancer%2ftoc.json) que contenga sus máquinas virtuales. En este caso, utilice el siguiente comando:

    ```powershell
    New-AzAvailabilitySet -ResourceGroupName "contosofabrikam" -Name "myAvailset" -Location "West Central US"
    ```

4. Siga las instrucciones de los pasos 3 a 5 del artículo [Creación de una máquina virtual Windows](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fazure%2fload-balancer%2ftoc.json) para preparar la creación de una máquina virtual con una sola NIC. Realice el paso 6.1 y use el siguiente código en lugar del paso 6.2:

    ```powershell
    $availset = Get-AzAvailabilitySet -ResourceGroupName "contosofabrikam" -Name "myAvailset"
    New-AzVMConfig -VMName "VM1" -VMSize "Standard_DS1_v2" -AvailabilitySetId $availset.Id
    ```

    Después, complete los pasos 6.3 a 6.8 de [Creación de una máquina virtual Windows](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fazure%2fload-balancer%2ftoc.json).

5. Agregue una segunda configuración de IP a cada una de las máquinas virtuales. Siga las instrucciones del artículo [Asignación de varias direcciones IP a máquinas virtuales](../virtual-network/virtual-network-multiple-ip-addresses-powershell.md#add). Use las opciones de configuración siguientes:

    ```powershell
    $NicName = "VM1-NIC2"
    $RgName = "contosofabrikam"
    $NicLocation = "West Central US"
    $IPConfigName4 = "VM1-ipconfig2"
    $Subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "mySubnet" -VirtualNetwork $myVnet
    ```

    Para los fines de este tutorial, no es necesario asociar las configuraciones de IP secundarias a direcciones IP públicas. Edite el comando para quitar la parte pública de la asociación de IP.

6. Complete de nuevo los pasos 4 a 6 de este artículo para VM2. Al hacerlo, asegúrese de reemplazar el nombre de la máquina virtual por VM2. Tenga en cuenta que no es necesario crear una red virtual para la segunda máquina virtual. Según sus necesidades de uso, puede crear o no una nueva subred.

7. Cree dos direcciones IP públicas y almacénelas en las variables correspondientes, tal y como se muestra:

    ```powershell
    $publicIP1 = New-AzPublicIpAddress -Name PublicIp1 -ResourceGroupName contosofabrikam -Location 'West Central US' -AllocationMethod Dynamic -DomainNameLabel contoso
    $publicIP2 = New-AzPublicIpAddress -Name PublicIp2 -ResourceGroupName contosofabrikam -Location 'West Central US' -AllocationMethod Dynamic -DomainNameLabel fabrikam

    $publicIP1 = Get-AzPublicIpAddress -Name PublicIp1 -ResourceGroupName contosofabrikam
    $publicIP2 = Get-AzPublicIpAddress -Name PublicIp2 -ResourceGroupName contosofabrikam
    ```

8. Cree dos configuraciones de IP de front-end:

    ```powershell
    $frontendIP1 = New-AzLoadBalancerFrontendIpConfig -Name contosofe -PublicIpAddress $publicIP1
    $frontendIP2 = New-AzLoadBalancerFrontendIpConfig -Name fabrikamfe -PublicIpAddress $publicIP2
    ```

9. Cree los grupos de direcciones de back-end, un sondeo y las reglas de equilibrio de carga:

    ```powershell
    $beaddresspool1 = New-AzLoadBalancerBackendAddressPoolConfig -Name contosopool
    $beaddresspool2 = New-AzLoadBalancerBackendAddressPoolConfig -Name fabrikampool

    $healthProbe = New-AzLoadBalancerProbeConfig -Name HTTP -RequestPath 'index.html' -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

    $lbrule1 = New-AzLoadBalancerRuleConfig -Name HTTPc -FrontendIpConfiguration $frontendIP1 -BackendAddressPool $beaddresspool1 -Probe $healthprobe -Protocol Tcp -FrontendPort 80 -BackendPort 80
    $lbrule2 = New-AzLoadBalancerRuleConfig -Name HTTPf -FrontendIpConfiguration $frontendIP2 -BackendAddressPool $beaddresspool2 -Probe $healthprobe -Protocol Tcp -FrontendPort 80 -BackendPort 80
    ```

10. Una vez que tenga estos recursos creados, cree el equilibrador de carga:

    ```powershell
    $mylb = New-AzLoadBalancer -ResourceGroupName contosofabrikam -Name mylb -Location 'West Central US' -FrontendIpConfiguration $frontendIP1 -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe
    ```

11. Agregue el segundo grupo de direcciones de back-end y la configuración de IP de front-end al equilibrador de carga recién creado:

    ```powershell
    $mylb = Get-AzLoadBalancer -Name "mylb" -ResourceGroupName $myResourceGroup | Add-AzLoadBalancerBackendAddressPoolConfig -Name fabrikampool | Set-AzLoadBalancer

    $mylb | Add-AzLoadBalancerFrontendIpConfig -Name fabrikamfe -PublicIpAddress $publicIP2 | Set-AzLoadBalancer
    
    Add-AzLoadBalancerRuleConfig -Name HTTP -LoadBalancer $mylb -FrontendIpConfiguration $frontendIP2 -BackendAddressPool $beaddresspool2 -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80 | Set-AzLoadBalancer
    ```

12. Los siguientes comandos obtienen las NIC y agregan ambas configuraciones de IP de cada NIC secundaria al grupo de direcciones de back-end del equilibrador de carga:

    ```powershell
    $nic1 = Get-AzNetworkInterface -Name "VM1-NIC2" -ResourceGroupName "MyResourcegroup";
    $nic2 = Get-AzNetworkInterface -Name "VM2-NIC2" -ResourceGroupName "MyResourcegroup";

    $nic1.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[0]);
    $nic1.IpConfigurations[1].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[1]);
    $nic2.IpConfigurations[0].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[0]);
    $nic2.IpConfigurations[1].LoadBalancerBackendAddressPools.Add($mylb.BackendAddressPools[1]);

    $mylb = $mylb | Set-AzLoadBalancer

    $nic1 | Set-AzNetworkInterface
    $nic2 | Set-AzNetworkInterface
    ```

13. Por último, debe configurar los registros de recursos DNS para que apunten a la dirección IP de front-end correspondiente del equilibrador de carga. Puede hospedar los dominios en Azure DNS. Para más información sobre el uso de Azure DNS con Load Balancer, consulte [Uso de Azure DNS con otros servicios de Azure](../dns/dns-for-azure-services.md).

## <a name="next-steps"></a>Pasos siguientes
- Aprenda más sobre cómo combinar servicios de equilibrio de carga en Azure en [Uso de servicios de equilibrio de carga de Azure](../traffic-manager/traffic-manager-load-balancing-azure.md).
- Obtenga información sobre cómo usar diferentes tipos de registros en Azure para administrar y solucionar los problemas del equilibrador de carga en [Registros de Azure Monitor para Azure Load Balancer](../load-balancer/load-balancer-monitor-log.md).
