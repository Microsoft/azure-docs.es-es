---
title: Creación de una instancia de Azure Application Gateway y reescritura de los encabezados HTTP
description: En este artículo, se proporciona información sobre cómo crear una instancia de Azure Application Gateway y reescribir encabezados HTTP con Azure PowerShell
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/19/2019
ms.author: absha
ms.openlocfilehash: e18288dbc2a09c7e9dd5b0c0e96dfd04ec192596
ms.sourcegitcommit: 1b320bc7863707a07e98644fbaed9faa0108da97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89595910"
---
# <a name="create-an-application-gateway-and-rewrite-http-headers"></a>Crear una instancia de Application Gateway y reescribir los encabezados HTTP

Puede usar Azure PowerShell para configurar las [reglas de reescritura de encabezados HTTP de solicitud y respuesta](rewrite-http-headers.md) al crear una [SKU de Application Gateway de escalado automático y con redundancia de zona](https://docs.microsoft.com/azure/application-gateway/application-gateway-autoscaling-zone-redundant) existente.

En este artículo aprenderá a:

* Crear una red virtual de escalabilidad automática
* Creación de una dirección IP pública reservada
* Configurar una infraestructura para Application Gateway
* Especificar la configuración de la regla para volver a escribir el encabezado
* Especificación de la escalabilidad automática
* Creación de la puerta de enlace de aplicaciones
* Prueba de la puerta de enlace de aplicaciones

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="prerequisites"></a>Prerequisites

En este artículo es necesario ejecutar Azure PowerShell en local. Necesita tener instalado el módulo de Azure versión 1.0.0 o posterior. Ejecute `Import-Module Az` y, después, `Get-Module Az` para encontrar la versión. Si necesita actualizarla, consulte [Instalación del módulo de Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps). Después de verificar la versión de PowerShell, ejecute `Login-AzAccount` para crear una conexión con Azure.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

```azurepowershell
Connect-AzAccount
Select-AzSubscription -Subscription "<sub name>"
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Cree un grupo de recursos en una de las ubicaciones disponibles.

```azurepowershell
$location = "East US 2"
$rg = "<rg name>"

#Create a new Resource Group
New-AzResourceGroup -Name $rg -Location $location
```

## <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree una red virtual con una subred específica para una instancia de Application Gateway con escalabilidad automática. Actualmente solo se puede implementar una puerta de enlace de aplicaciones con escalabilidad automática en cada subred específica.

```azurepowershell
#Create VNet with two subnets
$sub1 = New-AzVirtualNetworkSubnetConfig -Name "AppGwSubnet" -AddressPrefix "10.0.0.0/24"
$sub2 = New-AzVirtualNetworkSubnetConfig -Name "BackendSubnet" -AddressPrefix "10.0.1.0/24"
$vnet = New-AzvirtualNetwork -Name "AutoscaleVNet" -ResourceGroupName $rg `
       -Location $location -AddressPrefix "10.0.0.0/16" -Subnet $sub1, $sub2
```

## <a name="create-a-reserved-public-ip"></a>Creación de una dirección IP pública reservada

Especifique el método de asignación de PublicIPAddress como **Estático**. La dirección IP virtual de la puerta de enlace de aplicaciones con escalabilidad automática solo puede ser estática. No se admiten direcciones IP dinámicas. Se admite solo la SKU estándar PublicIpAddress.

```azurepowershell
#Create static public IP
$pip = New-AzPublicIpAddress -ResourceGroupName $rg -name "AppGwVIP" `
       -location $location -AllocationMethod Static -Sku Standard
```

## <a name="retrieve-details"></a>Recuperación de los detalles

Recupere los detalles del grupo de recursos, la subred y la dirección IP en un objeto local para crear los detalles de configuración de IP de Application Gateway.

```azurepowershell
$resourceGroup = Get-AzResourceGroup -Name $rg
$publicip = Get-AzPublicIpAddress -ResourceGroupName $rg -name "AppGwVIP"
$vnet = Get-AzvirtualNetwork -Name "AutoscaleVNet" -ResourceGroupName $rg
$gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name "AppGwSubnet" -VirtualNetwork $vnet
```

## <a name="configure-the-infrastructure"></a>Configuración de la infraestructura

Configure la dirección IP, la dirección IP de front-end, el grupo de servidores back-end, los ajustes de HTTP, el certificado, el puerto, el agente de escucha y la regla con un formato idéntico al de la instancia estándar de Application Gateway. La nueva SKU sigue el mismo modelo de objeto que la SKU estándar.

```azurepowershell
$ipconfig = New-AzApplicationGatewayIPConfiguration -Name "IPConfig" -Subnet $gwSubnet
$fip = New-AzApplicationGatewayFrontendIPConfig -Name "FrontendIPCOnfig" -PublicIPAddress $publicip
$pool = New-AzApplicationGatewayBackendAddressPool -Name "Pool1" `
       -BackendIPAddresses testbackend1.westus.cloudapp.azure.com, testbackend2.westus.cloudapp.azure.com
$fp01 = New-AzApplicationGatewayFrontendPort -Name "HTTPPort" -Port 80

$listener01 = New-AzApplicationGatewayHttpListener -Name "HTTPListener" `
             -Protocol Http -FrontendIPConfiguration $fip -FrontendPort $fp01

$setting = New-AzApplicationGatewayBackendHttpSettings -Name "BackendHttpSetting1" `
          -Port 80 -Protocol Http -CookieBasedAffinity Disabled
```

## <a name="specify-your-http-header-rewrite-rule-configuration"></a>Especificar la configuración de la regla de reescritura de encabezados HTTP

Configure los nuevos objetos necesarios para reescribir los encabezados HTTP:

- **RequestHeaderConfiguration**: este objeto se usa para especificar los campos del encabezado de solicitud que va a reescribir y el nuevo valor con el que se reescribirán los encabezados originales.
- **ResponseHeaderConfiguration**: este objeto se usa para especificar los campos del encabezado de respuesta que quiera reescribir y el nuevo valor con el que se reescribirán los encabezados originales.
- **ActionSet**: este objeto contiene las configuraciones de los encabezados de solicitud y respuesta especificados anteriormente. 
- **RewriteRule**: este objeto contiene todos los elementos *actionSets* especificados anteriormente. 
- **RewriteRuleSet**: este objeto contiene todos los elementos *rewriteRules* y tendrá que conectarse a una regla de enrutamiento de solicitudes (básica o basada en ruta de acceso).

   ```azurepowershell
   $requestHeaderConfiguration = New-AzApplicationGatewayRewriteRuleHeaderConfiguration -HeaderName "X-isThroughProxy" -HeaderValue "True"
   $responseHeaderConfiguration = New-AzApplicationGatewayRewriteRuleHeaderConfiguration -HeaderName "Strict-Transport-Security" -HeaderValue "max-age=31536000"
   $actionSet = New-AzApplicationGatewayRewriteRuleActionSet -RequestHeaderConfiguration $requestHeaderConfiguration -ResponseHeaderConfiguration $responseHeaderConfiguration    
   $rewriteRule = New-AzApplicationGatewayRewriteRule -Name rewriteRule1 -ActionSet $actionSet    
   $rewriteRuleSet = New-AzApplicationGatewayRewriteRuleSet -Name rewriteRuleSet1 -RewriteRule $rewriteRule
   ```

## <a name="specify-the-routing-rule"></a>Especificar la regla de enrutamiento

Cree una regla de enrutamiento de solicitudes. Una vez creada, esta configuración de reescritura se conecta al agente de escucha de origen mediante la regla de enrutamiento. Al usar una regla de enrutamiento básica, la configuración de reescritura de encabezados se asocia a un agente de escucha de origen y es una reescritura de encabezados global. Al usar una regla de enrutamiento basada en rutas, la configuración de reescritura de encabezado se define en la asignación de la ruta de URL. Por lo tanto, solo se aplica al área específica de la ruta de acceso de un sitio. A continuación, se crea una regla de enrutamiento básica y se adjunta el conjunto de reglas de reescritura.

```azurepowershell
$rule01 = New-AzApplicationGatewayRequestRoutingRule -Name "Rule1" -RuleType basic `
         -BackendHttpSettings $setting -HttpListener $listener01 -BackendAddressPool $pool -RewriteRuleSet $rewriteRuleSet
```

## <a name="specify-autoscale"></a>Especificación de la escalabilidad automática

Ahora puede especificar la configuración de la escalabilidad automática para Application Gateway. Admite dos tipos de configuración:

* **Modo de capacidad fija**. En este modo, la puerta de enlace de aplicaciones no escala automáticamente y funciona con una unidad de escalado fija.

   ```azurepowershell
   $sku = New-AzApplicationGatewaySku -Name Standard_v2 -Tier Standard_v2 -Capacity 2
   ```

* **Modo de escalabilidad automática**. En este modo, la puerta de enlace de aplicaciones escala automáticamente en función del patrón de tráfico de las aplicaciones.

   ```azurepowershell
   $autoscaleConfig = New-AzApplicationGatewayAutoscaleConfiguration -MinCapacity 2
   $sku = New-AzApplicationGatewaySku -Name Standard_v2 -Tier Standard_v2
   ```

## <a name="create-the-application-gateway"></a>Creación de la puerta de enlace de aplicaciones

Cree la instancia de Application Gateway e incluya las zonas de redundancia y la configuración de escalabilidad automática.

```azurepowershell
$appgw = New-AzApplicationGateway -Name "AutoscalingAppGw" -Zone 1,2,3 -ResourceGroupName $rg -Location $location -BackendAddressPools $pool -BackendHttpSettingsCollection $setting -GatewayIpConfigurations $ipconfig -FrontendIpConfigurations $fip -FrontendPorts $fp01 -HttpListeners $listener01 -RequestRoutingRules $rule01 -Sku $sku -AutoscaleConfiguration $autoscaleConfig -RewriteRuleSet $rewriteRuleSet
```

## <a name="test-the-application-gateway"></a>Prueba de la puerta de enlace de aplicaciones

Use Get-AzureRmPublicIPAddress para obtener la dirección IP pública de la puerta de enlace de aplicaciones. Copie la dirección IP pública o el nombre DNS y péguelo en la barra de direcciones del explorador.

```azurepowershell
Get-AzPublicIPAddress -ResourceGroupName $rg -Name AppGwVIP
```



## <a name="clean-up-resources"></a>Limpieza de recursos

En primer lugar, explore los recursos que se crearon con Application Gateway. Después, cuando ya no se necesiten, puede usar el comando `Remove-AzResourceGroup` para quitar el grupo de recursos, la instancia de Application Gateway y todos los recursos relacionados.

`Remove-AzResourceGroup -Name $rg`

## <a name="next-steps"></a>Pasos siguientes

- [Crear una puerta de enlace de aplicaciones con reglas de enrutamiento basadas en rutas de direcciones URL](./tutorial-url-route-powershell.md)
