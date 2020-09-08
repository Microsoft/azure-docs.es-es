---
title: Cree un vínculo privado de Azure mediante Azure PowerShell| Microsoft Docs
description: Aprenda a crear un vínculo privado de Azure mediante Azure PowerShell
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: how-to
ms.date: 09/16/2019
ms.author: allensu
ms.openlocfilehash: 3c808623269b8fabc32134a165b964a3b0747d4b
ms.sourcegitcommit: 54d8052c09e847a6565ec978f352769e8955aead
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88505661"
---
# <a name="create-a-private-link-service-using-azure-powershell"></a>Creación de un servicio Private Link mediante Azure PowerShell
En este artículo se muestra cómo crear un servicio Private Link en Azure mediante Azure PowerShell.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, este artículo requiere la versión del módulo de Azure PowerShell más reciente. Ejecute `Get-Module -ListAvailable Az` para buscar la versión instalada. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Connect-AzAccount` para crear una conexión con Azure.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para poder crear el vínculo privado, debe crear primero un grupo de recursos con [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). En el ejemplo siguiente se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *WestCentralUS*:

```azurepowershell
$location = "westcentralus"
$rgName = "myResourceGroup"
New-AzResourceGroup `
  -ResourceGroupName $rgName `
  -Location $location
```
## <a name="create-a-virtual-network"></a>Creación de una red virtual
Cree una red virtual para su vínculo privado con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). En el ejemplo siguiente se crea una red virtual denominada *myvnet* con la subred para frontend (*frontendSubnet*), backend (*backendSubnet*), vínculo privado (*otherSubnet*):

```azurepowershell
$virtualNetworkName = "myvnet"


# Create subnet config

$frontendSubnet = New-AzVirtualNetworkSubnetConfig `
-Name frontendSubnet `
-AddressPrefix "10.0.1.0/24"

$backendSubnet = New-AzVirtualNetworkSubnetConfig `
-Name backendSubnet `
-AddressPrefix "10.0.2.0/24"

$otherSubnet = New-AzVirtualNetworkSubnetConfig `
-Name otherSubnet `
-AddressPrefix "10.0.3.0/24" `
-PrivateLinkServiceNetworkPolicies "Disabled"

# Create the virtual network
$vnet = New-AzVirtualNetwork `
-Name $virtualNetworkName `
-ResourceGroupName $rgName `
-Location $location `
-AddressPrefix "10.0.0.0/16" `
-Subnet $frontendSubnet,$backendSubnet,$otherSubnet
```
## <a name="create-internal-load-balancer"></a>Creación del equilibrador de carga interno
Cree una instancia interna de Standard Load Balancer con [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer). En el ejemplo siguiente se crea una instancia interna de Standard Load Balancer mediante la configuración IP de front-end, el sondeo, la regla y el grupo de back-end que creó en los pasos anteriores:

```azurepowershell

$lbBackendName = "LB-backend"
$lbFrontName = "LB-frontend"
$lbName = "lb"

#Create Internal Load Balancer
$frontendIP = New-AzLoadBalancerFrontendIpConfig -Name $lbFrontName -PrivateIpAddress 10.0.1.5 -SubnetId $vnet.subnets[0].Id
$beaddresspool= New-AzLoadBalancerBackendAddressPoolConfig -Name $lbBackendName
$probe = New-AzLoadBalancerProbeConfig -Name 'myHealthProbe' -Protocol Http -Port 80 `
  -RequestPath / -IntervalInSeconds 360 -ProbeCount 5
$rule = New-AzLoadBalancerRuleConfig -Name HTTP -FrontendIpConfiguration $frontendIP -BackendAddressPool  $beaddresspool -Probe $probe -Protocol Tcp -FrontendPort 80 -BackendPort 80
$NRPLB = New-AzLoadBalancer -ResourceGroupName $rgName -Name $lbName -Location $location -FrontendIpConfiguration $frontendIP -BackendAddressPool $beAddressPool -Probe $probe -LoadBalancingRule $rule -Sku Standard
```
## <a name="create-a-private-link-service"></a>Creación de un servicio Private Link
Cree un servicio de vínculo privado con [New-AzPrivateLinkService](/powershell/module/az.network/new-azloadbalancer).  En este ejemplo se crea un servicio de vínculo privado denominado *myPLS* con una instancia de Standard Load Balancer en el grupo de recursos llamado *myResourceGroup*.
```azurepowershell

$plsIpConfigName = "PLS-ipconfig"
$plsName = "pls"
$peName = "pe"
 
$IPConfig = New-AzPrivateLinkServiceIpConfig `
-Name $plsIpConfigName `
-Subnet $vnet.subnets[2] `
-PrivateIpAddress 10.0.3.5

$fe = Get-AzLoadBalancer -Name $lbName | Get-AzLoadBalancerFrontendIpConfig

$privateLinkService = New-AzPrivateLinkService `
-ServiceName $plsName `
-ResourceGroupName $rgName `
-Location $location `
-LoadBalancerFrontendIpConfiguration $frontendIP `
-IpConfiguration $IPConfig
```

### <a name="get-private-link-service"></a>Obtención del servicio de vínculo pribado
Obtenga detalles sobre el servicio de vínculo privado [Get-AzPrivateLinkService](/powershell/module/az.network/get-azprivatelinkservice) como se indica a continuación:

```azurepowershell
$pls = Get-AzPrivateLinkService -Name $plsName -ResourceGroupName $rgName
```

En esta fase, el servicio Private Link se ha creado correctamente y está listo para recibir el tráfico. Tenga en cuenta que en el ejemplo anterior solo se muestra la creación de un servicio Private Link mediante PowerShell.  No hemos configurado los grupos de back-end del equilibrador de carga ni ninguna aplicación en los grupos de back-end para escuchar el tráfico. Si desea ver los flujos de tráfico de un extremo a otro, le recomendamos que configure la aplicación detrás de la instancia local de Standard Load Balancer.

A continuación, se mostrará cómo asignar este servicio a un punto de conexión privado en una red virtual diferente mediante PowerShell. De nuevo, el ejemplo se limita a crear el punto de conexión privado y conectarlo al servicio Private Link creado anteriormente. Puede crear máquinas virtuales en la red virtual para enviar tráfico al punto de conexión privado o recibirlo de él a fin de crear su escenario.

## <a name="create-a-private-endpoint"></a>Creación de un punto de conexión privado
### <a name="create-a-virtual-network"></a>Creación de una red virtual
Cree una red virtual para su punto de conexión privado con [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork). En este ejemplo se crea una red virtual denominada  *vnetPE*  en el grupo de recursos *myResourceGroup*:

```azurepowershell
$virtualNetworkNamePE = "vnetPE"

# Create VNet for private endpoint
$peSubnet = New-AzVirtualNetworkSubnetConfig `
-Name peSubnet `
-AddressPrefix "11.0.1.0/24" `
-PrivateEndpointNetworkPolicies "Disabled"

$vnetPE = New-AzVirtualNetwork `
-Name $virtualNetworkNamePE `
-ResourceGroupName $rgName `
-Location $location `
-AddressPrefix "11.0.0.0/16" `
-Subnet $peSubnet
```

### <a name="create-a-private-endpoint"></a>Creación de un punto de conexión privado
Cree un punto de conexión privado para consumir el servicio de vínculo privado creado anteriormente en la red virtual:

```azurepowershell

$plsConnection= New-AzPrivateLinkServiceConnection `
-Name plsConnection `
-PrivateLinkServiceId  $privateLinkService.Id

$privateEndpoint = New-AzPrivateEndpoint -ResourceGroupName $rgName -Name $peName -Location $location -Subnet $vnetPE.subnets[0] -PrivateLinkServiceConnection $plsConnection -ByManualRequest
```

### <a name="get-private-endpoint"></a>Obtener punto de conexión privado
Obtenga la dirección IP del punto de conexión privado con `Get-AzPrivateEndpoint` como se indica a continuación:

```azurepowershell
# Get Private Endpoint and its IP Address
$pe =  Get-AzPrivateEndpoint `
-Name $peName `
-ResourceGroupName $rgName  `
-ExpandResource networkinterfaces

$pe.NetworkInterfaces[0].IpConfigurations[0].PrivateIpAddress

```

### <a name="approve-the-private-endpoint-connection"></a>Aprobación de la conexión del punto de conexión privado
Apruebe la conexión de punto de conexión privado al servicio de vínculo privado con "Approve-AzPrivateEndpointConnection".

```azurepowershell

$pls = Get-AzPrivateLinkService `
-Name $plsName `
-ResourceGroupName $rgName

Approve-AzPrivateEndpointConnection -ResourceId $pls.PrivateEndpointConnections[0].Id -Description "Approved"

```

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre [Azure Private Link](private-link-overview.md).

