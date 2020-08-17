---
title: Uso de API Management en una red virtual con Application Gateway
titleSuffix: Azure API Management
description: Aprenda a instalar y configurar Azure API Management en una red virtual interna con Application Gateway (WAF) como front-end.
services: api-management
documentationcenter: ''
author: solankisamir
manager: kjoshi
editor: vlvinogr
ms.assetid: a8c982b2-bca5-4312-9367-4a0bbc1082b1
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/04/2019
ms.author: sasolank
ms.openlocfilehash: eb2ce196687b2ca6a762a879570e4f8ebac788df
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87025122"
---
# <a name="integrate-api-management-in-an-internal-vnet-with-application-gateway"></a>Integración de API Management en una red virtual interna con Application Gateway

## <a name="overview"></a><a name="overview"> </a> Información general

El servicio API Management se puede configurar en una red virtual en modo interno, para que sea accesible únicamente desde dentro de la red virtual. Azure Application Gateway es un servicio PAAS que proporciona un equilibrador de carga de nivel 7. Actúa como un servicio de proxy inverso y proporciona entre su oferta un firewall de aplicaciones web (WAF).

La combinación de una instancia de API Management aprovisionada en una red virtual interna con el front-end de Application Gateway permite los siguientes escenarios:

* Utilizar el mismo recurso de API Management para su uso por los consumidores internos y los consumidores externos.
* Utilizar un único recurso de API Management y tener definido un subconjunto de API en API Management disponible para los consumidores externos.
* Proporcionar una solución de llave en mano para activar y desactivar el acceso a API Management desde la red Internet pública.

[!INCLUDE [premium-dev.md](../../includes/api-management-availability-premium-dev.md)]

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para seguir los pasos que se describen en este artículo, debe tener:

* Una suscripción de Azure activa.

    [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

* Certificados pfx y cer para el nombre de host de API y pfx para el nombre de host del portal para desarrolladores.

## <a name="scenario"></a><a name="scenario"> </a> Escenario 9:

En este artículo se explica cómo usar un único servicio de API Management para los consumidores tanto internos como externos y hacer que actúe como un único servidor de front-end para las API locales y en la nube. También verá cómo exponer solamente un subconjunto de las API (resaltado en verde en el ejemplo) para permitir su uso externo, usando para ello la funcionalidad de enrutamiento disponible en Application Gateway.

En el primer ejemplo de la configuración, todas las API se administran únicamente desde dentro de la red virtual. Los consumidores internos (resaltados en color naranja) pueden tener acceso a todas las API internas y externas. El tráfico nunca sale a Internet. La conectividad de alto rendimiento se proporciona mediante circuitos ExpressRoute.

![ruta de dirección URL](./media/api-management-howto-integrate-internal-vnet-appgateway/api-management-howto-integrate-internal-vnet-appgateway.png)

## <a name="before-you-begin"></a><a name="before-you-begin"> </a> Antes de empezar

* Asegúrese de que está usando la versión más reciente de Azure PowerShell. Consulte las instrucciones de instalación en [Instalación de Azure PowerShell](/powershell/azure/install-az-ps). 

## <a name="what-is-required-to-create-an-integration-between-api-management-and-application-gateway"></a>¿Qué se necesita para crear una integración entre API Management y Application Gateway?

* **Grupo de servidores back-end:** se trata de la dirección IP virtual interna del servicio API Management.
* **Configuración del grupo de servidores back-end:** cada grupo tiene una configuración como el puerto, el protocolo y la afinidad basada en las cookies. Estos valores se aplican a todos los servidores del grupo.
* **Puerto de front-end:** es el puerto público que se abre en la puerta de enlace de aplicaciones. El tráfico que llega se redirige a uno de los servidores back-end.
* **Agente de escucha:** tiene un puerto front-end, un protocolo (Http o Https, estos valores distinguen mayúsculas de minúsculas) y el nombre del certificado TLS/SSL (si se configura una descarga de TLS).
* **Regla:** la regla enlaza un agente de escucha con un grupo de servidores back-end.
* **Sondeo de estado personalizado:** de forma predeterminada, Application Gateway usa sondeos basados en direcciones IP para determinar cuáles son los servidores de BackendAddressPool que están activos. El servicio API Management responde solo a las solicitudes con el encabezado de host correcto, por lo tanto, los sondeos predeterminados no podrán completarse. Es necesario definir el sondeo de mantenimiento personalizado para ayudar a la puerta de enlace de aplicaciones a determinar que el servicio está activo y debe reenviar las solicitudes.
* **Certificados de dominio personalizado:** para tener acceso a API Management desde Internet, debe crear una asignación de CNAME entre el nombre de host y el nombre DNS del front-end de Application Gateway. Esto garantiza que el encabezado de nombre de host y el certificado enviados a Application Gateway que se reenvían a API Management pueden ser reconocidos como válidos por APIM. En este ejemplo, usaremos dos certificados: uno para el back-end y otro para el portal para desarrolladores.  

## <a name="steps-required-for-integrating-api-management-and-application-gateway"></a><a name="overview-steps"></a> Pasos necesarios para integrar API Management y Application Gateway

1. Cree un grupo de recursos para Resource Manager.
2. Cree una red virtual, una subred y una IP pública para Application Gateway. Cree otra subred para API Management.
3. Cree un servicio API Management en la subred de la red virtual creada anteriormente y asegúrese de que usa el modo interno.
4. Configure un nombre de dominio personalizado en el servicio API Management.
5. Cree un objeto de configuración de Application Gateway.
6. Cree un recurso de Application Gateway.
7. Cree un CNAME del nombre DNS público de Application Gateway en el nombre de host de proxy de API Management.

## <a name="exposing-the-developer-portal-externally-through-application-gateway"></a>Exposición del portal para desarrolladores externamente mediante Application Gateway

En esta guía también se expondrá el **portal para desarrolladores** a audiencias externas mediante Application Gateway. Para ello es necesario realizar pasos adicionales para crear el agente de escucha del portal para desarrolladores, el sondeo, la configuración y las reglas. Todos los detalles se proporcionan en los pasos correspondientes.

> [!WARNING]
> Si usa Azure AD o un método de autenticación de terceros, habilite la característica de [afinidad de sesión basada en cookies](../application-gateway/features.md#session-affinity) de Application Gateway.

> [!WARNING]
> Para evitar que WAF de Application Gateway interrumpa la descarga de la especificación de OpenAPI en el portal para desarrolladores, debe deshabilitar la regla de firewall `942200 - "Detects MySQL comment-/space-obfuscated injections and backtick termination"`.
> 
> Las reglas de WAF de Application Gateway, que pueden interrumpir la funcionalidad del portal, incluyen:
> 
> - `920330`, `931130`, `942100`, `942110`, `942180`, `942200`, `942260`, `942370`, `949110`, `980130` para el modo administrativo
> - `942200`, `942260`, `942370`, `942430`, `942440` para el portal publicado

## <a name="create-a-resource-group-for-resource-manager"></a>Creación de un grupo de recursos para Resource Manager

### <a name="step-1"></a>Paso 1

Inicio de sesión en Azure

```powershell
Connect-AzAccount
```

Autentíquese con sus credenciales.

### <a name="step-2"></a>Paso 2

Seleccione la suscripción deseada.

```powershell
$subscriptionId = "00000000-0000-0000-0000-000000000000" # GUID of your Azure subscription
Get-AzSubscription -Subscriptionid $subscriptionId | Select-AzSubscription
```

### <a name="step-3"></a>Paso 3

Cree un grupo de recursos (omita este paso si usa uno existente).

```powershell
$resGroupName = "apim-appGw-RG" # resource group name
$location = "West US"           # Azure region
New-AzResourceGroup -Name $resGroupName -Location $location
```

Azure Resource Manager requiere que todos los grupos de recursos especifiquen una ubicación. Esta se utiliza como ubicación predeterminada para los recursos de ese grupo de recursos. Asegúrese de que todos los comandos para crear una puerta de enlace de aplicaciones usan el mismo grupo de recursos.

## <a name="create-a-virtual-network-and-a-subnet-for-the-application-gateway"></a>Creación de una red virtual y una subred para la puerta de enlace de aplicaciones

En el ejemplo siguiente se muestra cómo crear una red virtual con Resource Manager.

### <a name="step-1"></a>Paso 1

Asigne el intervalo de direcciones 10.0.0.0/24 a la variable subnet que se va a usar para Application Gateway al crear la red virtual.

```powershell
$appgatewaysubnet = New-AzVirtualNetworkSubnetConfig -Name "apim01" -AddressPrefix "10.0.0.0/24"
```

### <a name="step-2"></a>Paso 2

Asigne el intervalo de direcciones 10.0.1.0/24 a la variable subnet que se va a usar para API Management al crear la red virtual.

```powershell
$apimsubnet = New-AzVirtualNetworkSubnetConfig -Name "apim02" -AddressPrefix "10.0.1.0/24"
```

### <a name="step-3"></a>Paso 3

Cree una red virtual llamada **appgwvnet** en el grupo de recursos **apim-appGw-RG** para la región Oeste de EE. UU. Use el prefijo 10.0.0.0/16 con las subredes 10.0.0.0/24 y 10.0.1.0/24.

```powershell
$vnet = New-AzVirtualNetwork -Name "appgwvnet" -ResourceGroupName $resGroupName -Location $location -AddressPrefix "10.0.0.0/16" -Subnet $appgatewaysubnet,$apimsubnet
```

### <a name="step-4"></a>Paso 4

Asigne una variable de subred para los pasos siguientes.

```powershell
$appgatewaysubnetdata = $vnet.Subnets[0]
$apimsubnetdata = $vnet.Subnets[1]
```

## <a name="create-an-api-management-service-inside-a-vnet-configured-in-internal-mode"></a>Cree un servicio de API Management dentro de una red virtual configurada en modo interno.

En el ejemplo siguiente se muestra cómo crear un servicio de API Management en una red virtual configurada únicamente para el acceso interno.

### <a name="step-1"></a>Paso 1

Cree un objeto de red virtual de API Management con la subred $apimsubnetdata creada anteriormente.

```powershell
$apimVirtualNetwork = New-AzApiManagementVirtualNetwork -SubnetResourceId $apimsubnetdata.Id
```

### <a name="step-2"></a>Paso 2

Cree un servicio de API Management dentro de la red virtual.

```powershell
$apimServiceName = "ContosoApi"       # API Management service instance name
$apimOrganization = "Contoso"         # organization name
$apimAdminEmail = "admin@contoso.com" # administrator's email address
$apimService = New-AzApiManagement -ResourceGroupName $resGroupName -Location $location -Name $apimServiceName -Organization $apimOrganization -AdminEmail $apimAdminEmail -VirtualNetwork $apimVirtualNetwork -VpnType "Internal" -Sku "Developer"
```

Cuando el comando anterior se complete con éxito, consulte [la configuración de DNS necesaria para tener acceso al servicio de API Management en una red virtual interna](api-management-using-with-internal-vnet.md#apim-dns-configuration) para tener acceso a él. Este paso puede tardar más de media hora.

## <a name="set-up-a-custom-domain-name-in-api-management"></a>Configuración de un nombre de dominio personalizado en API Management

> [!IMPORTANT]
> El [nuevo portal para desarrolladores](api-management-howto-developer-portal.md) también requiere la habilitación de la conectividad con el punto de conexión de administración de API Management, además de los pasos siguientes.

### <a name="step-1"></a>Paso 1

Inicialice las variables siguientes con los detalles de los certificados con claves privadas para los dominios. En este ejemplo, usaremos `api.contoso.net` y `portal.contoso.net`.  

```powershell
$gatewayHostname = "api.contoso.net"                 # API gateway host
$portalHostname = "portal.contoso.net"               # API developer portal host
$gatewayCertCerPath = "C:\Users\Contoso\gateway.cer" # full path to api.contoso.net .cer file
$gatewayCertPfxPath = "C:\Users\Contoso\gateway.pfx" # full path to api.contoso.net .pfx file
$portalCertPfxPath = "C:\Users\Contoso\portal.pfx"   # full path to portal.contoso.net .pfx file
$gatewayCertPfxPassword = "certificatePassword123"   # password for api.contoso.net pfx certificate
$portalCertPfxPassword = "certificatePassword123"    # password for portal.contoso.net pfx certificate

$certPwd = ConvertTo-SecureString -String $gatewayCertPfxPassword -AsPlainText -Force
$certPortalPwd = ConvertTo-SecureString -String $portalCertPfxPassword -AsPlainText -Force
```

### <a name="step-2"></a>Paso 2

Cree y establezca los objetos de configuración del nombre de host para el servidor proxy y el portal.  

```powershell
$proxyHostnameConfig = New-AzApiManagementCustomHostnameConfiguration -Hostname $gatewayHostname -HostnameType Proxy -PfxPath $gatewayCertPfxPath -PfxPassword $certPwd
$portalHostnameConfig = New-AzApiManagementCustomHostnameConfiguration -Hostname $portalHostname -HostnameType DeveloperPortal -PfxPath $portalCertPfxPath -PfxPassword $certPortalPwd

$apimService.ProxyCustomHostnameConfiguration = $proxyHostnameConfig
$apimService.PortalCustomHostnameConfiguration = $portalHostnameConfig
Set-AzApiManagement -InputObject $apimService
```

> [!NOTE]
> Para configurar la conectividad del portal para desarrolladores heredada, debe reemplazar `-HostnameType DeveloperPortal` por `-HostnameType Portal`.

## <a name="create-a-public-ip-address-for-the-front-end-configuration"></a>Creación de una dirección IP pública para la configuración del front-end

Cree un recurso de IP pública **publicIP01** en el grupo de recursos.

```powershell
$publicip = New-AzPublicIpAddress -ResourceGroupName $resGroupName -name "publicIP01" -location $location -AllocationMethod Dynamic
```

Se asigna una dirección IP a la puerta de enlace de aplicaciones cuando se inicia el servicio.

## <a name="create-application-gateway-configuration"></a>Creación de una configuración de puerta de enlace de aplicaciones

Se deben haber definido todos los elementos de configuración antes de crear la puerta de enlace de aplicaciones. En los pasos siguientes, se crean los elementos de configuración necesarios para un recurso de puerta de enlace de aplicaciones.

### <a name="step-1"></a>Paso 1

Cree una configuración de IP de puerta de enlace de aplicaciones denominada **gatewayIP01**. Cuando se inicia Application Gateway, elige una dirección IP de la subred configurada y redirige el tráfico de red a las direcciones IP en el grupo IP de back-end. Tenga en cuenta que cada instancia toma una dirección IP.

```powershell
$gipconfig = New-AzApplicationGatewayIPConfiguration -Name "gatewayIP01" -Subnet $appgatewaysubnetdata
```

### <a name="step-2"></a>Paso 2

Configure el puerto IP de front-end para el punto de conexión de IP pública. Este puerto es el puerto al que se conectan los usuarios finales.

```powershell
$fp01 = New-AzApplicationGatewayFrontendPort -Name "port01"  -Port 443
```

### <a name="step-3"></a>Paso 3

Configuración de la dirección IP de front-end con el punto de conexión de IP pública

```powershell
$fipconfig01 = New-AzApplicationGatewayFrontendIPConfig -Name "frontend1" -PublicIPAddress $publicip
```

### <a name="step-4"></a>Paso 4

Configure el certificado para Application Gateway, que se usará para descifrar y volver a cifrar el tráfico que pasa por ella.

```powershell
$cert = New-AzApplicationGatewaySslCertificate -Name "cert01" -CertificateFile $gatewayCertPfxPath -Password $certPwd
$certPortal = New-AzApplicationGatewaySslCertificate -Name "cert02" -CertificateFile $portalCertPfxPath -Password $certPortalPwd
```

### <a name="step-5"></a>Paso 5

Cree los agentes de escucha HTTP para Application Gateway. Asígneles la configuración IP de front-end, el puerto y los certificados TSL/SSL que se usarán.

```powershell
$listener = New-AzApplicationGatewayHttpListener -Name "listener01" -Protocol "Https" -FrontendIPConfiguration $fipconfig01 -FrontendPort $fp01 -SslCertificate $cert -HostName $gatewayHostname -RequireServerNameIndication true
$portalListener = New-AzApplicationGatewayHttpListener -Name "listener02" -Protocol "Https" -FrontendIPConfiguration $fipconfig01 -FrontendPort $fp01 -SslCertificate $certPortal -HostName $portalHostname -RequireServerNameIndication true
```

### <a name="step-6"></a>Paso 6

Cree sondeos personalizados para el punto de conexión de dominio del proxy `ContosoApi` del servicio API Management. La ruta de acceso `/status-0123456789abcdef` es un punto de conexión de mantenimiento predeterminado hospedado en todos los servicios de API Management. Establezca `api.contoso.net` como un nombre de host de sondeo personalizado para protegerlo con el certificado TLS/SSL.

> [!NOTE]
> El nombre de host `contosoapi.azure-api.net` es el nombre de host de proxy predeterminado configurado cuando se crea un servicio denominado `contosoapi` en el ámbito público de Azure.
>

```powershell
$apimprobe = New-AzApplicationGatewayProbeConfig -Name "apimproxyprobe" -Protocol "Https" -HostName $gatewayHostname -Path "/status-0123456789abcdef" -Interval 30 -Timeout 120 -UnhealthyThreshold 8
$apimPortalProbe = New-AzApplicationGatewayProbeConfig -Name "apimportalprobe" -Protocol "Https" -HostName $portalHostname -Path "/signin" -Interval 60 -Timeout 300 -UnhealthyThreshold 8
```

### <a name="step-7"></a>Paso 7

Cargue el certificado que se usará en los recursos del grupo de back-end habilitado para TLS/SSL. Este es el mismo certificado que ha proporcionado en el paso 4 anterior.

```powershell
$authcert = New-AzApplicationGatewayAuthenticationCertificate -Name "whitelistcert1" -CertificateFile $gatewayCertCerPath
```

### <a name="step-8"></a>Paso 8

Configure las opciones de back-end HTTP para Application Gateway. Esto incluye la configuración de un límite de tiempo de espera para la solicitud de back-end después del cual se cancela. Este valor es distinto del tiempo de espera del sondeo.

```powershell
$apimPoolSetting = New-AzApplicationGatewayBackendHttpSettings -Name "apimPoolSetting" -Port 443 -Protocol "Https" -CookieBasedAffinity "Disabled" -Probe $apimprobe -AuthenticationCertificates $authcert -RequestTimeout 180
$apimPoolPortalSetting = New-AzApplicationGatewayBackendHttpSettings -Name "apimPoolPortalSetting" -Port 443 -Protocol "Https" -CookieBasedAffinity "Disabled" -Probe $apimPortalProbe -AuthenticationCertificates $authcert -RequestTimeout 180
```

### <a name="step-9"></a>Paso 9:

Configure el grupo de direcciones IP de back-end denominado **apimbackend** con dirección IP virtual interna para el servicio de API Management creado anteriormente.

```powershell
$apimProxyBackendPool = New-AzApplicationGatewayBackendAddressPool -Name "apimbackend" -BackendIPAddresses $apimService.PrivateIPAddresses[0]
```

### <a name="step-10"></a>Paso 10

Cree reglas para Application Gateway para usar el enrutamiento básico.

```powershell
$rule01 = New-AzApplicationGatewayRequestRoutingRule -Name "rule1" -RuleType Basic -HttpListener $listener -BackendAddressPool $apimProxyBackendPool -BackendHttpSettings $apimPoolSetting
$rule02 = New-AzApplicationGatewayRequestRoutingRule -Name "rule2" -RuleType Basic -HttpListener $portalListener -BackendAddressPool $apimProxyBackendPool -BackendHttpSettings $apimPoolPortalSetting
```

> [!TIP]
> Cambie el tipo de regla y el enrutamiento para restringir el acceso a determinadas páginas del portal para desarrolladores.

### <a name="step-11"></a>Paso 11

Configure el número de instancias y el tamaño de Application Gateway. En este ejemplo, usamos [WAF SKU](../web-application-firewall/ag/ag-overview.md) para aumentar la seguridad de los recursos de API Management.

```powershell
$sku = New-AzApplicationGatewaySku -Name "WAF_Medium" -Tier "WAF" -Capacity 2
```

### <a name="step-12"></a>Paso 12

Configuración de WAFS para que esté en modo "Prevención".

```powershell
$config = New-AzApplicationGatewayWebApplicationFirewallConfiguration -Enabled $true -FirewallMode "Prevention"
```

## <a name="create-application-gateway"></a>Creación de la puerta de enlace de aplicaciones

Cree una instancia de Application Gateway con todos los objetos de configuración de los pasos anteriores.

```powershell
$appgwName = "apim-app-gw"
$appgw = New-AzApplicationGateway -Name $appgwName -ResourceGroupName $resGroupName -Location $location -BackendAddressPools $apimProxyBackendPool -BackendHttpSettingsCollection $apimPoolSetting, $apimPoolPortalSetting  -FrontendIpConfigurations $fipconfig01 -GatewayIpConfigurations $gipconfig -FrontendPorts $fp01 -HttpListeners $listener, $portalListener -RequestRoutingRules $rule01, $rule02 -Sku $sku -WebApplicationFirewallConfig $config -SslCertificates $cert, $certPortal -AuthenticationCertificates $authcert -Probes $apimprobe, $apimPortalProbe
```

## <a name="cname-the-api-management-proxy-hostname-to-the-public-dns-name-of-the-application-gateway-resource"></a>Aplicación de un CNAME al nombre de host del proxy de API Management para el nombre DNS público del recurso de Application Gateway

Una vez creada la puerta de enlace, el siguiente paso es configurar el front-end para la comunicación. Cuando se utiliza una dirección IP pública, Application Gateway requiere un nombre DNS asignado dinámicamente, que puede no ser fácil de usar.

El nombre DNS de Application Gateway se debe utilizar para crear un registro CNAME, que apunta el nombre de host del proxy (por ejemplo, `api.contoso.net` en los ejemplos anteriores) a este nombre de DNS. Para configurar el registro IP CNAME de front-end, recupere los detalles de Application Gateway y su nombre DNS o IP asociados mediante el elemento PublicIPAddress. No se recomienda el uso de registros A, ya que la VIP puede cambiar al reiniciarse la puerta de enlace.

```powershell
Get-AzPublicIpAddress -ResourceGroupName $resGroupName -Name "publicIP01"
```

## <a name="summary"></a><a name="summary"> </a> Resumen
El servicio Azure API Management configurado en una red virtual proporciona una interfaz de puerta de enlace única para todas las API configuradas, independientemente de si están hospedadas de forma local o en la nube. La integración de Application Gateway con API Management proporciona la flexibilidad de habilitar de manera selectiva API determinadas para que estén accesibles en Internet, así como la provisión de un firewall de aplicación web como front-end para la instancia de API Management.

## <a name="next-steps"></a><a name="next-steps"> </a> Pasos siguientes
* Obtenga más información sobre Azure Application Gateway.
  * [Introducción a Puerta de enlace de aplicaciones](../application-gateway/overview.md)
  * [Firewall de aplicaciones web de Application Gateway](../web-application-firewall/ag/ag-overview.md)
  * [Application Gateway mediante enrutamiento basado en rutas de acceso](../application-gateway/tutorial-url-route-powershell.md)
* Más información acerca de API Management y redes virtuales
  * [El uso de API Management solo está disponible en la red virtual](api-management-using-with-internal-vnet.md)
  * [Usar Azure API Management con redes virtuales](api-management-using-with-vnet.md)
