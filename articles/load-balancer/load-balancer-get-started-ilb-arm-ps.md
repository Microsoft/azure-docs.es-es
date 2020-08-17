---
title: Creación de un equilibrador de carga interno de Azure mediante PowerShell
titleSuffix: Azure Load Balancer
description: Obtenga información acerca de cómo crear un equilibrador de carga interno mediante el módulo de Azure PowerShell con Azure Resource Manager
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/02/2020
ms.author: allensu
ms.openlocfilehash: 7e962b87b1bd7ef200aa276d8dbecffd5d3fed04
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090440"
---
# <a name="create-an-internal-load-balancer-by-using-the-azure-powershell-module"></a>Creación de un equilibrador de carga interno mediante el módulo de Azure PowerShell

> [!div class="op_single_selector"]
> * [Azure Portal](../load-balancer/load-balancer-get-started-ilb-arm-portal.md)
> * [PowerShell](../load-balancer/load-balancer-get-started-ilb-arm-ps.md)
> * [CLI de Azure](../load-balancer/load-balancer-get-started-ilb-arm-cli.md)
> * [Plantilla](../load-balancer/load-balancer-get-started-ilb-arm-template.md)

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [load-balancer-get-started-ilb-intro-include.md](../../includes/load-balancer-get-started-ilb-intro-include.md)]

[!INCLUDE [load-balancer-get-started-ilb-scenario-include.md](../../includes/load-balancer-get-started-ilb-scenario-include.md)]

[!INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## <a name="get-started-with-the-configuration"></a>Introducción a la configuración

En este artículo se explica cómo crear un equilibrador de carga interno mediante Azure Resource Manager con el módulo de Azure PowerShell. En el modelo de implementación de Resource Manager, los objetos que son necesarios para crear un equilibrador de carga interno se configuran individualmente. Una vez que los objetos se crean y configuran, se combinan para crear un equilibrador de carga.

Para implementar un equilibrador de carga, se deben crear los objetos siguientes:

* Grupo de direcciones IP de front-end: la dirección IP privada de todo tráfico de red entrante.
* Grupo de direcciones de back-end: las interfaces de red que reciben el tráfico con equilibrio de carga de la dirección IP front-end.
* Reglas de equilibrio de carga: la configuración del puerto (de origen y local) del equilibrador de carga.
* Configuración de sondeo: el estado de mantenimiento sondea si hay máquinas virtuales.
* Reglas NAT de entrada: las reglas de puerto para el acceso directo a las máquinas virtuales.

Para más información sobre los componentes del equilibrador de carga, vea [Componentes de Azure Load Balancer](components.md).

Los pasos siguientes explican cómo configurar un equilibrador de carga entre dos máquinas virtuales.

## <a name="set-up-powershell-to-use-resource-manager"></a>Configuración de PowerShell para usar Resource Manager

Asegúrese de que tiene la última versión de producción del módulo de Azure PowerShell. PowerShell debe estar configurado correctamente para acceder a su suscripción de Azure.

### <a name="step-1-start-powershell"></a>Paso 1: Iniciar PowerShell

Inicie el módulo de PowerShell para Azure Resource Manager.

```azurepowershell-interactive
Connect-AzAccount
```

### <a name="step-2-view-your-subscriptions"></a>Paso 2: Visualizar las suscripciones

Compruebe las suscripciones de Azure disponibles.

```azurepowershell-interactive
Get-AzSubscription
```

Escriba sus credenciales cuando se le pida para la autenticación.

### <a name="step-3-select-the-subscription-to-use"></a>Paso 3: Seleccione la suscripción que se va a usar

Elija qué suscripciones de Azure va a usar para implementar el equilibrador de carga.

```azurepowershell-interactive
Select-AzSubscription -Subscriptionid "GUID of subscription"
```

### <a name="step-4-choose-the-resource-group-for-the-load-balancer"></a>Paso 4: Elegir el grupo de recursos del equilibrador de carga

Cree un nuevo grupo de recursos para el equilibrador de carga. Si utiliza un grupo de recursos existente, omita este paso.

```azurepowershell-interactive
New-AzResourceGroup -Name NRP-RG -location "West US"
```

Azure Resource Manager requiere que todos los grupos de recursos especifiquen una ubicación. Esta se utiliza como valor predeterminado para los recursos del grupo de recursos. Utilice siempre el mismo grupo de recursos para todos los comandos relacionados con la creación del equilibrador de carga.

En el ejemplo anterior, hemos creado un grupo de recursos denominado **NRP-RG** en la ubicación Oeste de EE. UU.

## <a name="create-the-virtual-network-and-ip-address-for-the-front-end-ip-pool"></a>Creación de la red virtual y la dirección IP del grupo de direcciones IP de front-end

Cree una subred para la red virtual y asígnela a la variable **$backendSubnet**.

```azurepowershell-interactive
$backendSubnet = New-AzVirtualNetworkSubnetConfig -Name LB-Subnet-BE -AddressPrefix 10.0.2.0/24
```

Cree una red virtual.

```azurepowershell-interactive
$vnet= New-AzVirtualNetwork -Name NRPVNet -ResourceGroupName NRP-RG -Location "West US" -AddressPrefix 10.0.0.0/16 -Subnet $backendSubnet
```

La red virtual está creada. La subred **LB-Subnet-BE** se agrega a la red virtual **NRPVNet**. Estos valores se asignan a la variable **$vnet**.

## <a name="create-the-front-end-ip-pool-and-back-end-address-pool"></a>Creación del grupo de direcciones IP de front-end y el grupo de direcciones de back-end

Cree un grupo de direcciones IP de front-end para el tráfico entrante y un grupo de direcciones de back-end para recibir el tráfico con equilibrio de carga.

### <a name="step-1-create-a-front-end-ip-pool"></a>Paso 1: Creación de un grupo de direcciones IP de front-end

Cree un grupo de direcciones IP de front-end con la dirección IP privada 10.0.2.5 para la subred 10.0.2.0/24. Esta dirección es el punto de conexión del tráfico de red entrante.

```azurepowershell-interactive
$frontendIP = New-AzLoadBalancerFrontendIpConfig -Name LB-Frontend -PrivateIpAddress 10.0.2.5 -SubnetId $vnet.subnets[0].Id
```

### <a name="step-2-create-a-back-end-address-pool"></a>Paso 2: Creación del grupo de direcciones de back-end

Cree un grupo de direcciones de back-end para recibir el tráfico entrante proveniente del grupo de direcciones IP de front-end:

```powershell
$beaddresspool= New-AzLoadBalancerBackendAddressPoolConfig -Name "LB-backend"
```

## <a name="create-the-configuration-rules-probe-and-load-balancer"></a>Creación de las reglas de configuración, el sondeo y el equilibrador de carga

Una vez que se han creado el grupo de direcciones IP de front-end y el grupo de direcciones de back-end, especifique las reglas del recurso de equilibrador de carga.

### <a name="step-1-create-the-configuration-rules"></a>Paso 1: Creación de las reglas de configuración

En el ejemplo se crean los cuatro objetos de regla siguientes:

* Una regla NAT de entrada para el protocolo de escritorio remoto (RDP): redirige todo el tráfico entrante del puerto 3441 al puerto 3389.
* Una segunda regla NAT de entrada para RDP: redirige todo el tráfico entrante del puerto 3442 al puerto 3389.
* Una regla de sondeo de mantenimiento: comprueba el estado del mantenimiento de la ruta de acceso de HealthProbe.aspx.
* Una regla del equilibrador de carga: equilibra la carga de todo el tráfico entrante en el puerto público 80 al puerto local 80 del grupo de direcciones de back-end.
* Una [regla del equilibrador de carga para los puertos de alta disponibilidad](load-balancer-ha-ports-overview.md) para equilibrar la carga de todo el tráfico entrante en todos los puertos, a fin de simplificar los escenarios de alta disponibilidad para el equilibrador de carga interno estándar.

```azurepowershell-interactive
$inboundNATRule1= New-AzLoadBalancerInboundNatRuleConfig -Name "RDP1" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3441 -BackendPort 3389

$inboundNATRule2= New-AzLoadBalancerInboundNatRuleConfig -Name "RDP2" -FrontendIpConfiguration $frontendIP -Protocol TCP -FrontendPort 3442 -BackendPort 3389

$healthProbe = New-AzLoadBalancerProbeConfig -Name "HealthProbe" -RequestPath "HealthProbe.aspx" -Protocol http -Port 80 -IntervalInSeconds 15 -ProbeCount 2

$lbrule = New-AzLoadBalancerRuleConfig -Name "HTTP" -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol Tcp -FrontendPort 80 -BackendPort 80

$haportslbrule = New-AzLoadBalancerRuleConfig -Name "HAPortsRule" -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $healthProbe -Protocol "All" -FrontendPort 0 -BackendPort 0
```

### <a name="step-2-create-the-load-balancer"></a>Paso 2: Creación del equilibrador de carga

Crear el equilibrador de carga y combine los objetos de regla (NAT de entrada para RDP, equilibrador de carga y sondeo de estado):

```azurepowershell-interactive
$NRPLB = New-AzLoadBalancer -ResourceGroupName "NRP-RG" -Name "NRP-LB" -SKU Standard -Location "West US" -FrontendIpConfiguration $frontendIP -InboundNatRule $inboundNATRule1,$inboundNatRule2 -LoadBalancingRule $lbrule -BackendAddressPool $beAddressPool -Probe $healthProbe
```

Utilice `-SKU Basic` para crear una instancia de Load Balancer Básico. Microsoft recomienda el uso de Standard Load Balancer para cargas de trabajo de producción.

## <a name="create-the-network-interfaces"></a>Creación de las interfaces de red

Después de crear el equilibrador de carga interno, defina las interfaces de red (NIC) que van a recibir el tráfico de red entrante con equilibrio de carga, las reglas NAT y el sondeo. Cada interfaz de red se configura individualmente y se asigna posteriormente a una máquina virtual.

### <a name="step-1-create-the-first-network-interface"></a>Paso 1: Creación de la primera de interfaz de red

Obtenga la red virtual y la subred de recursos. Estos valores se utilizan para crear las interfaces de red:

```azurepowershell-interactive
$vnet = Get-AzVirtualNetwork -Name NRPVNet -ResourceGroupName NRP-RG

$backendSubnet = Get-AzVirtualNetworkSubnetConfig -Name LB-Subnet-BE -VirtualNetwork $vnet
```

Cree la primera interfaz de red con el nombre **lb-nic1-be**. Asigne la interfaz al grupo de back-end del equilibrador de carga. Asocie la primera regla NAT para RDP a esta NIC:

```azurepowershell-interactive
$backendnic1= New-AzNetworkInterface -ResourceGroupName "NRP-RG" -Name lb-nic1-be -Location "West US" -PrivateIpAddress 10.0.2.6 -Subnet $backendSubnet -LoadBalancerBackendAddressPool $nrplb.BackendAddressPools[0] -LoadBalancerInboundNatRule $nrplb.InboundNatRules[0]
```

### <a name="step-2-create-the-second-network-interface"></a>Paso 2: Creación de la segunda interfaz de red

Cree la segunda interfaz de red con el nombre **lb-nic2-be**. Asigne la segunda interfaz al mismo grupo de back-end del equilibrador de carga que la primera interfaz. Asocie el segundo NIC a la segunda regla NAT para RDP:

```azurepowershell-interactive
$backendnic2= New-AzNetworkInterface -ResourceGroupName "NRP-RG" -Name lb-nic2-be -Location "West US" -PrivateIpAddress 10.0.2.7 -Subnet $backendSubnet -LoadBalancerBackendAddressPool $nrplb.BackendAddressPools[0] -LoadBalancerInboundNatRule $nrplb.InboundNatRules[1]
```

Revise la configuración:

```azurepowershell-interactive
$backendnic1
```

La configuración debe se como esta:

```output
Name                 : lb-nic1-be
ResourceGroupName    : NRP-RG
Location             : westus
Id                   : /subscriptions/[Id]/resourceGroups/NRP-RG/providers/Microsoft.Network/networkInterfaces/lb-nic1-be
Etag                 : W/"d448256a-e1df-413a-9103-a137e07276d1"
ProvisioningState    : Succeeded
Tags                 :
VirtualMachine       : null
IpConfigurations     : [
                     {
                       "PrivateIpAddress": "10.0.2.6",
                       "PrivateIpAllocationMethod": "Static",
                       "Subnet": {
                         "Id": "/subscriptions/[Id]/resourceGroups/NRP-RG/providers/Microsoft.Network/virtualNetworks/NRPVNet/subnets/LB-Subnet-BE"
                       },
                       "PublicIpAddress": {
                         "Id": null
                       },
                       "LoadBalancerBackendAddressPools": [
                         {
                           "Id": "/subscriptions/[Id]/resourceGroups/NRP-RG/providers/Microsoft.Network/loadBalancers/NRPlb/backendAddressPools/LB-backend"
                         }
                       ],
                       "LoadBalancerInboundNatRules": [
                         {
                           "Id": "/subscriptions/[Id]/resourceGroups/NRP-RG/providers/Microsoft.Network/loadBalancers/NRPlb/inboundNatRules/RDP1"
                         }
                       ],
                       "ProvisioningState": "Succeeded",
                       "Name": "ipconfig1",
                       "Etag": "W/\"d448256a-e1df-413a-9103-a137e07276d1\"",
                       "Id": "/subscriptions/[Id]/resourceGroups/NRP-RG/providers/Microsoft.Network/networkInterfaces/lb-nic1-be/ipConfigurations/ipconfig1"
                     }
                   ]
DnsSettings          : {
                     "DnsServers": [],
                     "AppliedDnsServers": []
                   }
AppliedDnsSettings   :
NetworkSecurityGroup : null
Primary              : False
```

### <a name="step-3-assign-the-nic-to-a-vm"></a>Paso 3: Asignación de la NIC a una máquina virtual

Para asignar la NIC a una máquina virtual, use el comando `Add-AzVMNetworkInterface`.

Para obtener instrucciones detalladas para crear una máquina virtual y asignarla a una NIC, consulte [Creación de una máquina virtual de Azure con PowerShell](../virtual-machines/scripts/virtual-machines-windows-powershell-sample-create-vm.md?toc=%2fazure%2fload-balancer%2ftoc.json).

## <a name="add-the-network-interface"></a>Incorporación de la interfaz de red

Una vez creada la máquina virtual, agregue la interfaz de red.

### <a name="step-1-store-the-load-balancer-resource"></a>Paso 1: Almacenamiento del recurso del equilibrador de carga

Almacene el recurso del equilibrador de carga en una variable (si no lo ha hecho aún). Vamos a usar el nombre de variable **$lb**. Para los valores de atributo del script, use los nombres de los recursos del equilibrador de carga que se han creado en los pasos anteriores.

```azurepowershell-interactive
$lb = Get-AzLoadBalancer –name NRP-LB -resourcegroupname NRP-RG
```

### <a name="step-2-store-the-back-end-configuration"></a>Paso 2: Almacenamiento de la configuración de back-end

Almacene la configuración de back-end en la variable **$backend**.

```azurepowershell-interactive
$backend = Get-AzLoadBalancerBackendAddressPoolConfig -name LB-backend -LoadBalancer $lb
```

### <a name="step-3-store-the-network-interface"></a>Paso 3: Almacenamiento de la interfaz de red

Almacene la interfaz de red en otra variable. Esta interfaz se ha creado en el "paso 1 de Creación de las interfaces de red". Vamos a usar el nombre de variable **$nic1**. Use el mismo nombre de la interfaz de red que en ejemplo anterior.

```azurepowershell-interactive
$nic = Get-AzNetworkInterface –name lb-nic1-be -resourcegroupname NRP-RG
```

### <a name="step-4-change-the-back-end-configuration"></a>Paso 4: Cambio de la configuración de back-end

Cambie la configuración de back-end en la interfaz de red.

```azurepowershell-interactive
$nic.IpConfigurations[0].LoadBalancerBackendAddressPools=$backend
```

### <a name="step-5-save-the-network-interface-object"></a>Paso 5: Guardado del objeto de interfaz de red

Guarde el objeto de interfaz de red.

```azurepowershell-interactive
Set-AzNetworkInterface -NetworkInterface $nic
```

Después de que la interfaz se agrega al grupo de back-end, se equilibra la carga del tráfico de red según las reglas. Dichas reglas se han configurado en "Creación de las reglas de configuración, el sondeo y el equilibrador de carga".

## <a name="update-an-existing-load-balancer"></a>Actualización de un equilibrador de carga existente

### <a name="step-1-assign-the-load-balancer-object-to-a-variable"></a>Paso 1: Asignación del objeto de equilibrador de carga a una variable

Asigne el objeto de equilibrador de carga (del ejemplo anterior) a la variable **$slb** mediante el comando `Get-AzLoadBalancer`:

```azurepowershell-interactive
$slb = Get-AzLoadBalancer -Name NRP-LB -ResourceGroupName NRP-RG
```

### <a name="step-2-add-a-nat-rule"></a>Paso 2: Incorporación de una regla NAT

Agregue una nueva regla NAT de entrada a un equilibrador de carga existente. Utilice el puerto 81 para el grupo de front-end y el puerto 8181 para el grupo de back-end:

```azurepowershell-interactive
$slb | Add-AzLoadBalancerInboundNatRuleConfig -Name NewRule -FrontendIpConfiguration $slb.FrontendIpConfigurations[0] -FrontendPort 81  -BackendPort 8181 -Protocol Tcp
```

### <a name="step-3-save-the-configuration"></a>Paso 3: Guardado de la configuración

Guarde la nueva configuración mediante el comando `Set-AzureLoadBalancer`:

```azurepowershell-interactive
$slb | Set-AzLoadBalancer
```

## <a name="remove-an-existing-load-balancer"></a>Actualización de un equilibrador de carga existente

Elimine el equilibrador de carga **NRP LB** del grupo de recursos **NRP RG** mediante el comando `Remove-AzLoadBalancer`:

```azurepowershell-interactive
Remove-AzLoadBalancer -Name NRP-LB -ResourceGroupName NRP-RG
```

> [!NOTE]
> Usar el modificador **-Force** opcional para evitar la eliminación del mensaje de confirmación.

## <a name="next-steps"></a>Pasos siguientes

* [Configuración del modo de distribución en el equilibrador de carga](load-balancer-distribution-mode.md)
* [Configuración de opciones de tiempo de espera de inactividad de TCP para el equilibrador de carga](load-balancer-tcp-idle-timeout.md)
