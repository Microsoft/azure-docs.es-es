---
title: 'Inicio rápido: Tráfico web directo mediante PowerShell'
titleSuffix: Azure Application Gateway
description: En esta guía de inicio rápido, obtendrá información sobre cómo utilizar Azure PowerShell para crear una instancia de Azure Application Gateway que dirija el tráfico web a las máquinas virtuales de un grupo de back-end.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: quickstart
ms.date: 08/27/2020
ms.author: victorh
ms.custom: mvc
ms.openlocfilehash: 115f01bffc4664798682923cb83a99a23a8ce274
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88958344"
---
# <a name="quickstart-direct-web-traffic-with-azure-application-gateway-using-azure-powershell"></a>Inicio rápido: Tráfico web directo con Azure Application Gateway mediante Azure PowerShell

En este inicio rápido, usará Azure PowerShell para crear una puerta de enlace de aplicaciones. Luego, lo probará para asegurarse de que funciona correctamente. 

La puerta de enlace de aplicaciones dirige el tráfico web de la aplicación a recursos específicos de un grupo de back-end. Se asignan escuchas a los puertos, se crean reglas y se agregan recursos a un grupo de back-end. Para simplificar, en este artículo se usa una configuración sencilla con una dirección IP de front-end pública, una escucha básica que hospede un único sitio en la puerta de enlace de aplicaciones, una regla de enrutamiento de solicitudes básica y dos máquinas virtuales que se usan con el grupo de back-end.

También puede completar este inicio rápido con la [CLI de Azure](quick-create-cli.md) o con [Azure Portal](quick-create-portal.md).

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Azure PowerShell versión 1.0.0 o posterior](/powershell/azure/install-az-ps) (si ejecuta Azure PowerShell localmente).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="connect-to-azure"></a>Conexión con Azure

Para conectarse con Azure, ejecute `Connect-AzAccount`.

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

En Azure, puede asignar recursos relacionados a un grupo de recursos. Puede usar un grupo de recursos existente o crear uno nuevo.

Para crear un nuevo grupo de recursos, use el cmdlet `New-AzResourceGroup`: 

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroupAG -Location eastus
```
## <a name="create-network-resources"></a>Crear recursos de red

Para que Azure se comunique entre los recursos que se crean, se necesita una red virtual.  La subred de la puerta de enlace de aplicaciones solo puede contener puertas de enlace de aplicaciones. No se permite ningún otro recurso.  Puede crear una subred para Application Gateway o usar una existente. En este ejemplo se crean dos subredes: una para la puerta de enlace de aplicaciones y la otra para los servidores back-end. Puede configurar la dirección IP de front-end de Application Gateway para que sea pública o privada, según el caso de uso. En este ejemplo, elegimos una IP de front-end pública.

1. Cree las configuraciones de la subred mediante `New-AzVirtualNetworkSubnetConfig`.
2. Cree la red virtual con las configuraciones de la subred mediante `New-AzVirtualNetwork`. 
3. Cree la dirección IP pública mediante `New-AzPublicIpAddress`. 

```azurepowershell-interactive
$agSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myAGSubnet `
  -AddressPrefix 10.0.1.0/24
$backendSubnetConfig = New-AzVirtualNetworkSubnetConfig `
  -Name myBackendSubnet `
  -AddressPrefix 10.0.2.0/24
New-AzVirtualNetwork `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myVNet `
  -AddressPrefix 10.0.0.0/16 `
  -Subnet $agSubnetConfig, $backendSubnetConfig
New-AzPublicIpAddress `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -Name myAGPublicIPAddress `
  -AllocationMethod Static `
  -Sku Standard
```
## <a name="create-an-application-gateway"></a>Creación de una puerta de enlace de aplicaciones

### <a name="create-the-ip-configurations-and-frontend-port"></a>Creación de las configuraciones IP y el puerto de front-end

1. Use `New-AzApplicationGatewayIPConfiguration` para crear la configuración que asocia la subred que creó con la instancia la puerta de enlace de aplicaciones. 
2. Use `New-AzApplicationGatewayFrontendIPConfig` para crear la configuración que asigna la dirección IP pública que ha creado previamente con la una puerta de enlace de aplicaciones. 
3. Use `New-AzApplicationGatewayFrontendPort` para asignar el puerto 80 para acceder a la puerta de enlace de aplicaciones.

```azurepowershell-interactive
$vnet   = Get-AzVirtualNetwork -ResourceGroupName myResourceGroupAG -Name myVNet
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name myAGSubnet
$pip    = Get-AzPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress 
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

### <a name="create-the-backend-pool"></a>Creación del grupo de servidores back-end

1. Use `New-AzApplicationGatewayBackendAddressPool` para crear el grupo de back-end para la puerta de enlace de aplicaciones. El grupo de back-end estará vacío por ahora. Cuando cree las tarjetas de interfaz de red del servidor back-end en la sección siguiente, las agregará al grupo de back-end.
2. Configure los valores para el grupo de back-end con `New-AzApplicationGatewayBackendHttpSetting`.

```azurepowershell-interactive
$backendPool = New-AzApplicationGatewayBackendAddressPool `
  -Name myAGBackendPool
$poolSettings = New-AzApplicationGatewayBackendHttpSetting `
  -Name myPoolSettings `
  -Port 80 `
  -Protocol Http `
  -CookieBasedAffinity Enabled `
  -RequestTimeout 30
```

### <a name="create-the-listener-and-add-a-rule"></a>Creación del agente de escucha e incorporación de una regla

Azure necesita un agente de escucha para que la puerta de enlace de aplicaciones enrute el tráfico de forma adecuada al grupo de servidores back-end. También necesita una regla para que el agente de escucha sepa qué grupo de servidores back-end se usa para el tráfico entrante. 

1. Cree una escucha mediante `New-AzApplicationGatewayHttpListener` con la configuración de front-end y el puerto de front-end que creó anteriormente. 
2. Use `New-AzApplicationGatewayRequestRoutingRule` para crear una regla denominada *rule1*. 

```azurepowershell-interactive
$defaultlistener = New-AzApplicationGatewayHttpListener `
  -Name myAGListener `
  -Protocol Http `
  -FrontendIPConfiguration $fipconfig `
  -FrontendPort $frontendport
$frontendRule = New-AzApplicationGatewayRequestRoutingRule `
  -Name rule1 `
  -RuleType Basic `
  -HttpListener $defaultlistener `
  -BackendAddressPool $backendPool `
  -BackendHttpSettings $poolSettings
```

### <a name="create-the-application-gateway"></a>Creación de la puerta de enlace de aplicaciones

Ahora que ha creado los recursos auxiliares necesarios, cree la puerta de enlace de aplicaciones:

1. Use `New-AzApplicationGatewaySku` para especificar parámetros para la puerta de enlace de aplicaciones.
2. Use `New-AzApplicationGateway` para crear la puerta de enlace de aplicaciones.

```azurepowershell-interactive
$sku = New-AzApplicationGatewaySku `
  -Name Standard_v2 `
  -Tier Standard_v2 `
  -Capacity 2
New-AzApplicationGateway `
  -Name myAppGateway `
  -ResourceGroupName myResourceGroupAG `
  -Location eastus `
  -BackendAddressPools $backendPool `
  -BackendHttpSettingsCollection $poolSettings `
  -FrontendIpConfigurations $fipconfig `
  -GatewayIpConfigurations $gipconfig `
  -FrontendPorts $frontendport `
  -HttpListeners $defaultlistener `
  -RequestRoutingRules $frontendRule `
  -Sku $sku
```

### <a name="backend-servers"></a>Servidores back-end

Ahora que ha creado la instancia de Application Gateway, cree las máquinas virtuales de back-end que hospedarán los sitios web. Los servidores back-end pueden constar de NIC, conjuntos de escalado de máquinas virtuales, direcciones IP públicas e internas, nombres de dominio completos (FQDN) y servidores back-end multiinquilino como Azure App Service. En este ejemplo, se crean dos máquinas virtuales que usará Azure como servidores back-end para la puerta de enlace de aplicaciones. También se instala IIS en las máquinas virtuales para comprobar que Azure ha instalado correctamente la puerta de enlace de aplicaciones.

#### <a name="create-two-virtual-machines"></a>Creación de dos máquinas virtuales

1. Obtenga la configuración del grupo de back-end de Application Gateway recién creada con `Get-AzApplicationGatewayBackendAddressPool`.
2. Cree una interfaz de red con `New-AzNetworkInterface`.
3. Cree una configuración de máquina virtual con `New-AzVMConfig`.
4. Cree la máquina virtual con `New-AzVM`.

Al ejecutar el ejemplo de código siguiente para crear las máquinas virtuales, Azure le pide las credenciales. Escriba *azureuser* como nombre de usuario y una contraseña:
    
```azurepowershell-interactive
$appgw = Get-AzApplicationGateway -ResourceGroupName myResourceGroupAG -Name myAppGateway
$backendPool = Get-AzApplicationGatewayBackendAddressPool -Name myAGBackendPool -ApplicationGateway $appgw
$vnet   = Get-AzVirtualNetwork -ResourceGroupName myResourceGroupAG -Name myVNet
$subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name myBackendSubnet
$cred = Get-Credential
for ($i=1; $i -le 2; $i++)
{
  $nic = New-AzNetworkInterface `
    -Name myNic$i `
    -ResourceGroupName myResourceGroupAG `
    -Location EastUS `
    -Subnet $subnet `
    -ApplicationGatewayBackendAddressPool $backendpool
  $vm = New-AzVMConfig `
    -VMName myVM$i `
    -VMSize Standard_DS2_v2
  Set-AzVMOperatingSystem `
    -VM $vm `
    -Windows `
    -ComputerName myVM$i `
    -Credential $cred
  Set-AzVMSourceImage `
    -VM $vm `
    -PublisherName MicrosoftWindowsServer `
    -Offer WindowsServer `
    -Skus 2016-Datacenter `
    -Version latest
  Add-AzVMNetworkInterface `
    -VM $vm `
    -Id $nic.Id
  Set-AzVMBootDiagnostic `
    -VM $vm `
    -Disable
  New-AzVM -ResourceGroupName myResourceGroupAG -Location EastUS -VM $vm
  Set-AzVMExtension `
    -ResourceGroupName myResourceGroupAG `
    -ExtensionName IIS `
    -VMName myVM$i `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
    -Location EastUS
}
```

## <a name="test-the-application-gateway"></a>Prueba de la puerta de enlace de aplicaciones

No es necesario instalar IIS para crear la puerta de enlace de aplicaciones, pero se instaló en este inicio rápido para comprobar si la puerta de enlace de aplicaciones se creó correctamente. Use IIS para probar la puerta de enlace de aplicaciones:

1. Ejecute `Get-AzPublicIPAddress` para obtener la dirección IP pública de la puerta de enlace de aplicaciones. 
2. Copie la dirección IP pública y péguela en la barra de direcciones del explorador. Al actualizar el explorador, verá aparecer el nombre de la máquina virtual. Una respuesta válida corrobora que la puerta de enlace de aplicaciones se ha creado correctamente y puede conectarse correctamente con el back-end.

```azurepowershell-interactive
Get-AzPublicIPAddress -ResourceGroupName myResourceGroupAG -Name myAGPublicIPAddress
```

![Prueba de la puerta de enlace de aplicaciones](./media/quick-create-powershell/application-gateway-iistest.png)


## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite los recursos que ha creado con la puerta de enlace de aplicaciones, elimine el grupo de recursos. Al hacerlo, también elimina la puerta de enlace de aplicaciones y todos sus recursos relacionados. 

Para eliminar el grupo de recursos, llame al cmdlet `Remove-AzResourceGroup`:

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroupAG
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administrar el tráfico web con una puerta de enlace de aplicaciones mediante Azure PowerShell](./tutorial-manage-web-traffic-powershell.md)

