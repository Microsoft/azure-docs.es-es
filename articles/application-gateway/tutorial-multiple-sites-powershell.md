---
title: Hospedaje de varios sitios web con PowerShell
titleSuffix: Azure Application Gateway
description: Aprenda a crear una puerta de enlace de aplicaciones que hospede varios sitios web mediante Azure PowerShell.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 07/20/2020
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: dd73dc69fc2d40a0b4c24739dca6ad8174ad1047
ms.sourcegitcommit: 1b320bc7863707a07e98644fbaed9faa0108da97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89595881"
---
# <a name="create-an-application-gateway-that-hosts-multiple-web-sites-using-azure-powershell"></a>Crear una puerta de enlace de aplicaciones que hospede varios sitios web mediante Azure PowerShell

Puede usar Azure PowerShell para [configurar el hospedaje de varios sitios web](multiple-site-overview.md) cuando se crea una [puerta de enlace de aplicaciones](overview.md). En este artículo se definen grupos de direcciones de back-end mediante conjuntos de escalado de máquinas virtuales. Después, configurará agentes de escucha y reglas basados en los dominios que posee para asegurarse de que el tráfico web llega a los servidores adecuados en los grupos. En este artículo se da por supuesto que posee varios dominios y se van a utilizar los ejemplos de *www.contoso.com* y *www.fabrikam.com*.

En este artículo aprenderá a:

* Configuración de la red
* Creación de una puerta de enlace de aplicaciones
* Crear un agente de escucha de back-end
* Crear reglas de enrutamiento
* Crear conjuntos de escalado de máquinas virtuales con los grupos de back-end
* Creación de un registro CNAME en el dominio

:::image type="content" source="./media/tutorial-multiple-sites-powershell/scenario.png" alt-text="Instancia de Application Gateway multisitio":::

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar PowerShell de forma local, la versión del módulo de Azure PowerShell que necesita este artículo es la 1.0.0 u otra posterior. Para encontrar la versión, ejecute `Get-Module -ListAvailable Az`. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps). Si PowerShell se ejecuta localmente, también debe ejecutar `Login-AzAccount` para crear una conexión con Azure.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Cree un grupo de recursos de Azure mediante [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup).  

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroupAG -Location eastus
```

## <a name="create-network-resources"></a>Crear recursos de red

Cree la configuración de subred mediante [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig). Cree la red virtual mediante [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) con las configuraciones de la subred. Y, por último, cree la dirección IP pública mediante [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress). Estos recursos se usan para proporcionar conectividad de red a la puerta de enlace de aplicaciones y sus recursos asociados.

```azurepowershell-interactive
$backendSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
  -AddressPrefix 10.0.1.0/24

$agSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myAGSubnet `
  -AddressPrefix 10.0.2.0/24

$vnet = New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myVNet `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $backendSubnetConfig, $agSubnetConfig

$pip = New-AzPublicIpAddress `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myAGPublicIPAddress `
  -AllocationMethod Dynamic
```

## <a name="create-an-application-gateway"></a>Creación de una puerta de enlace de aplicaciones

### <a name="create-the-ip-configurations-and-frontend-port"></a>Creación de las configuraciones IP y el puerto de front-end

Asocie la subred que creó anteriormente a la puerta de enlace de aplicaciones mediante [New-AzApplicationGatewayIPConfiguration](/powershell/module/az.network/new-azapplicationgatewayipconfiguration). Asigne la dirección IP pública a la puerta de enlace de aplicaciones mediante [New-AzApplicationGatewayFrontendIPConfig](/powershell/module/az.network/new-azapplicationgatewayfrontendipconfig).

```azurepowershell-interactive
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet

$subnet=$vnet.Subnets[0]

$gipconfig = New-AzApplicationGatewayIPConfiguration `
  -Name myAGIPConfig `
  -Subnet $subnet

$fipconfig = New-AzApplicationGatewayFrontendIPConfig `
  -Name myAGFrontendIPConfig `
  -PublicIPAddress $pip

$frontendport = New-AzApplicationGatewayFrontendPort `
  -Name myFrontendPort `
  -Port 80
```

### <a name="create-the-backend-pools-and-settings"></a>Creación de la configuración y los grupos de servidores back-end

Cree el primer grupo de direcciones de back-end para la puerta de enlace de aplicaciones mediante [New-AzApplicationGatewayBackendAddressPool](/powershell/module/az.network/new-azapplicationgatewaybackendaddresspool). Configure los valores del grupo mediante [New-AzApplicationGatewayBackendHttpSettings](/powershell/module/az.network/new-azapplicationgatewaybackendhttpsetting).

```azurepowershell-interactive
$contosoPool = New-AzApplicationGatewayBackendAddressPool `
  -Name contosoPool

$fabrikamPool = New-AzApplicationGatewayBackendAddressPool `
  -Name fabrikamPool

$poolSettings = New-AzApplicationGatewayBackendHttpSettings `
  -Name myPoolSettings `
  -Port 80 `
  -Protocol Http `
  -CookieBasedAffinity Enabled `
  -RequestTimeout 120
```

### <a name="create-the-listeners-and-rules"></a>Creación de las reglas y los agentes de escucha

Los agentes de escucha son necesarios para permitir que la puerta de enlace de aplicaciones enrute el tráfico de forma adecuada a los grupos de direcciones de back-end. En este artículo, creará dos agentes de escucha para los dos dominios. Se crean agentes de escucha para los dominios *contoso.com* y *fabrikam.com*.

Cree el primer agente de escucha mediante [New-AzApplicationGatewayHttpListener](/powershell/module/az.network/new-azapplicationgatewayhttplistener) con la configuración de front-end y el puerto de front-end creados anteriormente. Es necesaria una regla para que el agente de escucha sepa qué grupo de servidores back-end se usa para el tráfico entrante. Cree una regla básica denominada *contosoRule* mediante [New-AzApplicationGatewayRequestRoutingRule](/powershell/module/az.network/new-azapplicationgatewayrequestroutingrule).

>[!NOTE]
> Con la SKU de Application Gateway o WAF v2, también puede configurar hasta cinco nombres de host por cliente de escucha y puede usar caracteres comodín en el nombre de host. Consulte los [nombres de host comodín en el cliente de escucha](multiple-site-overview.md#wildcard-host-names-in-listener-preview) para obtener más información.
>Para usar varios nombres de host y caracteres comodín en un cliente de escucha mediante Azure PowerShell, debe usar `-HostNames`, en lugar de `-HostName`. Con los nombres de host, puede mencionar un máximo de cinco nombres de host como valores separados por comas. Por ejemplo: `-HostNames "*.contoso.com,*.fabrikam.com"`

```azurepowershell-interactive
$contosolistener = New-AzApplicationGatewayHttpListener `
  -Name contosoListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendport `
  -HostName "www.contoso.com"

$fabrikamlistener = New-AzApplicationGatewayHttpListener `
  -Name fabrikamListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendport `
  -HostName "www.fabrikam.com"

$contosoRule = New-AzApplicationGatewayRequestRoutingRule `
  -Name contosoRule `
  -RuleType Basic `
  -HttpListener $contosoListener `
  -BackendAddressPool $contosoPool `
  -BackendHttpSettings $poolSettings

$fabrikamRule = New-AzApplicationGatewayRequestRoutingRule `
  -Name fabrikamRule `
  -RuleType Basic `
  -HttpListener $fabrikamListener `
  -BackendAddressPool $fabrikamPool `
  -BackendHttpSettings $poolSettings
```

### <a name="create-the-application-gateway"></a>Creación de la puerta de enlace de aplicaciones

Ahora que ha creado los recursos auxiliares necesarios, especifique parámetros para la puerta de enlace de aplicaciones mediante [New-AzApplicationGatewaySku](/powershell/module/az.network/new-azapplicationgatewaysku) y, luego, cree dicha puerta mediante [New-AzApplicationGateway](/powershell/module/az.network/new-azapplicationgateway).

```azurepowershell-interactive
$sku = New-AzApplicationGatewaySku `
  -Name Standard_Medium `
  -Tier Standard `
  -Capacity 2

$appgw = New-AzApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -BackendAddressPools $contosoPool, $fabrikamPool `
  -BackendHttpSettingsCollection $poolSettings `
  -FrontendIpConfigurations $fipconfig `
  -GatewayIpConfigurations $gipconfig `
  -FrontendPorts $frontendport `
  -HttpListeners $contosoListener, $fabrikamListener `
  -RequestRoutingRules $contosoRule, $fabrikamRule `
  -Sku $sku
```

## <a name="create-virtual-machine-scale-sets"></a>Creación de conjuntos de escalado de máquinas virtuales

En este ejemplo, creará dos conjuntos de escalado de máquinas virtuales que admiten los dos grupos de back-end que ha creado. Los conjuntos de escalado que se crean se llaman *myvmss1* y *myvmss2*. Cada conjunto de escalado contiene dos instancias de máquina virtual en las que se instala IIS. Asignará el conjunto de escalado al grupo de servidores back-end cuando configure los valores de IP.

```azurepowershell-interactive
$vnet = Get-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Name myVNet

$appgw = Get-AzApplicationGateway `
  -ResourceGroupName myResourceGroupAG `
  -Name myAppGateway

$contosoPool = Get-AzApplicationGatewayBackendAddressPool `
  -Name contosoPool `
  -ApplicationGateway $appgw

$fabrikamPool = Get-AzApplicationGatewayBackendAddressPool `
  -Name fabrikamPool `
  -ApplicationGateway $appgw

for ($i=1; $i -le 2; $i++)
{
  if ($i -eq 1) 
  {
    $poolId = $contosoPool.Id
  }
  if ($i -eq 2)
  {
    $poolId = $fabrikamPool.Id
  }

  $ipConfig = New-AzVmssIpConfig `
    -Name myVmssIPConfig$i `
    -SubnetId $vnet.Subnets[1].Id `
    -ApplicationGatewayBackendAddressPoolsId $poolId

  $vmssConfig = New-AzVmssConfig `
    -Location eastus `
    -SkuCapacity 2 `
    -SkuName Standard_DS2 `
    -UpgradePolicyMode Automatic

  Set-AzVmssStorageProfile $vmssConfig `
    -ImageReferencePublisher MicrosoftWindowsServer `
    -ImageReferenceOffer WindowsServer `
    -ImageReferenceSku 2016-Datacenter `
    -ImageReferenceVersion latest `
    -OsDiskCreateOption FromImage

  Set-AzVmssOsProfile $vmssConfig `
    -AdminUsername azureuser `
    -AdminPassword "Azure123456!" `
    -ComputerNamePrefix myvmss$i

  Add-AzVmssNetworkInterfaceConfiguration `
    -VirtualMachineScaleSet $vmssConfig `
    -Name myVmssNetConfig$i `
    -Primary $true `
    -IPConfiguration $ipConfig

  New-AzVmss `
    -ResourceGroupName myResourceGroupAG `
    -Name myvmss$i `
    -VirtualMachineScaleSet $vmssConfig
}
```

### <a name="install-iis"></a>Instalación de IIS

```azurepowershell-interactive
$publicSettings = @{ "fileUris" = (,"https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/appgatewayurl.ps1"); 
  "commandToExecute" = "powershell -ExecutionPolicy Unrestricted -File appgatewayurl.ps1" }

for ($i=1; $i -le 2; $i++)
{
  $vmss = Get-AzVmss `
    -ResourceGroupName myResourceGroupAG `
    -VMScaleSetName myvmss$i

  Add-AzVmssExtension -VirtualMachineScaleSet $vmss `
    -Name "customScript" `
    -Publisher "Microsoft.Compute" `
    -Type "CustomScriptExtension" `
    -TypeHandlerVersion 1.8 `
    -Setting $publicSettings

  Update-AzVmss `
    -ResourceGroupName myResourceGroupAG `
    -Name myvmss$i `
    -VirtualMachineScaleSet $vmss
}
```

## <a name="create-cname-record-in-your-domain"></a>Creación de un registro CNAME en el dominio

Después de crear la puerta de enlace de aplicaciones con la dirección IP pública, puede obtener la dirección DNS y usarla para crear un registro CNAME en el dominio. Puede usar [Get-AzPublicIPAddress](/powershell/module/az.network/get-azpublicipaddress) para obtener la dirección DNS de la puerta de enlace de aplicaciones. Copie el valor de *fqdn* de DNSSettings y úselo como valor del registro CNAME que creó. No se recomienda el uso de registros A, ya que la IP virtual puede cambiar al reiniciarse la puerta de enlace de aplicaciones en la SKU V1.

```azurepowershell-interactive
Get-AzPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

## <a name="test-the-application-gateway"></a>Prueba de la puerta de enlace de aplicaciones

Escriba el nombre de dominio en la barra de direcciones del explorador. Por ejemplo, http:\//www.contoso.com.

![Prueba del sitio de contoso en la puerta de enlace de aplicaciones](./media/tutorial-multiple-sites-powershell/application-gateway-iistest.png)

Cambie la dirección al otro dominio. Debería ver algo parecido al ejemplo siguiente:

![Prueba del sitio de fabrikam en la puerta de enlace de aplicaciones](./media/tutorial-multiple-sites-powershell/application-gateway-iistest2.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la puerta de enlace de aplicaciones y todos los recursos relacionados.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroupAG
```

## <a name="next-steps"></a>Pasos siguientes

[Crear una puerta de enlace de aplicaciones con reglas de enrutamiento basadas en rutas de direcciones URL](./tutorial-url-route-powershell.md)
