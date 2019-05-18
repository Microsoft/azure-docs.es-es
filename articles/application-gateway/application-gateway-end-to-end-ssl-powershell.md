---
title: Configuración de SSL de un extremo a otro con Azure Application Gateway
description: En este artículo se describe cómo configurar SSL de un extremo a otro con Azure Application Gateway mediante PowerShell
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: article
ms.date: 4/8/2019
ms.author: victorh
ms.openlocfilehash: 8c715cb84dff6e2e739de59aba33041ec1b8db52
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65786282"
---
# <a name="configure-end-to-end-ssl-by-using-application-gateway-with-powershell"></a>Configuración de SSL de un extremo a otro con Application Gateway mediante PowerShell

## <a name="overview"></a>Información general

Azure Application Gateway admite el cifrado de un extremo a otro del tráfico. Application Gateway termina la conexión SSL en la puerta de enlace de aplicaciones. La puerta de enlace aplica entonces las reglas de enrutamiento al tráfico, vuelve a cifrar el paquete y lo reenvía al servidor back-end adecuado según las reglas de enrutamiento definidas. Cualquier respuesta del servidor web pasa por el mismo proceso en su regreso al usuario final.

Application Gateway admite la definición de opciones SSL personalizadas. También admite deshabilitar las siguientes versiones de protocolo: **TLSv1.0**, **TLSv1.1** y **TLSv1.2**, así como definir qué conjuntos de cifrado usar y el orden de preferencia. Para más información sobre las opciones configurables de SSL, consulte la [introducción a la directiva SSL](application-gateway-SSL-policy-overview.md).

> [!NOTE]
> SSL 2.0 y SSL 3.0 están deshabilitados de manera predeterminada y no se pueden habilitar. Se considera que no son seguros y no se pueden usar con Application Gateway.

![imagen de escenario][scenario]

## <a name="scenario"></a>Escenario

En este escenario, aprenderá a crear una puerta de enlace de aplicaciones mediante SSL de un extremo a otro con PowerShell.

En este escenario:

* Creará un grupo de recursos llamado **appgw-rg**.
* Creará una red virtual denominada **appgwvnet** con un espacio de direcciones de **10.0.0.0/16**.
* Creará dos subredes llamadas **appgwsubnet** y **appsubnet**.
* Creará una puerta de enlace de aplicaciones pequeña que admite el cifrado SSL de un extremo a otro que limita las versiones del protocolo SSL y los conjuntos de cifrado.

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Para configurar SSL de un extremo a otro con una puerta de enlace de aplicaciones, hacen falta certificados para la puerta de enlace y los servidores back-end. El certificado de puerta de enlace se usa para derivar una clave simétrica según la especificación del protocolo SSL. A continuación, la clave simétrica se usa para cifrar y descifrar el tráfico que se envía a la puerta de enlace. El certificado de la puerta de enlace debe estar en formato de Intercambio de información personal (PFX). Este formato de archivo permite la exportación de la clave privada, lo que es necesario para que la puerta de enlace de aplicaciones pueda realizar el cifrado y descifrado del tráfico.

Para el cifrado SSL de un extremo a otro, el back-end debe estar en la lista de permitidos junto con la puerta de enlace de aplicaciones. Cargue el certificado público de los servidores back-end en la puerta de enlace de aplicaciones. Al agregar el certificado, se garantiza que la puerta de enlace de aplicaciones solo se comunique con instancias back-end conocidas. Esto protege aún más la comunicación de un extremo a otro.

El proceso de configuración se describe en las secciones siguientes.

## <a name="create-the-resource-group"></a>Creación del grupo de recursos

Esta sección le explica cómo crear un grupo de recursos que contiene la puerta de enlace de aplicaciones.

1. Inicie sesión en la cuenta de Azure.

   ```powershell
   Connect-AzAccount
   ```

2. Seleccione la suscripción que se usará en este escenario.

   ```powershell
   Select-Azsubscription -SubscriptionName "<Subscription name>"
   ```

3. Cree un grupo de recursos. (Si utiliza un grupo de recursos existente, puede omitir este paso).

   ```powershell
   New-AzResourceGroup -Name appgw-rg -Location "West US"
   ```

## <a name="create-a-virtual-network-and-a-subnet-for-the-application-gateway"></a>Creación de una red virtual y una subred para la puerta de enlace de aplicaciones

En el ejemplo siguiente se crea una red virtual y dos subredes. Una subred se usa para alojar la puerta de enlace de aplicaciones. La otra se usa para los back-ends que hospedan la aplicación web.

1. Asigne un rango de direcciones para la subred que se va a utiliza para la puerta de enlace de aplicaciones.

   ```powershell
   $gwSubnet = New-AzVirtualNetworkSubnetConfig -Name 'appgwsubnet' -AddressPrefix 10.0.0.0/24
   ```

   > [!NOTE]
   > Las subredes configuradas para una puerta de enlace de aplicaciones deben tener el tamaño correcto. Las puertas de enlace de aplicaciones se puede configurar para un máximo de diez instancias. Cada instancia toma una dirección IP de la subred. Una subred demasiado pequeña puede afectar negativamente el escalado horizontal de una puerta de enlace de aplicaciones.
   >

2. Asigne un rango de direcciones para el grupo de direcciones de back-end.

   ```powershell
   $nicSubnet = New-AzVirtualNetworkSubnetConfig  -Name 'appsubnet' -AddressPrefix 10.0.2.0/24
   ```

3. Cree una red virtual con las subredes definidas en los pasos anteriores.

   ```powershell
   $vnet = New-AzvirtualNetwork -Name 'appgwvnet' -ResourceGroupName appgw-rg -Location "West US" -AddressPrefix 10.0.0.0/16 -Subnet $gwSubnet, $nicSubnet
   ```

4. Recupere el recurso de red virtual y los recursos de subred que se usarán en los pasos siguientes.

   ```powershell
   $vnet = Get-AzvirtualNetwork -Name 'appgwvnet' -ResourceGroupName appgw-rg
   $gwSubnet = Get-AzVirtualNetworkSubnetConfig -Name 'appgwsubnet' -VirtualNetwork $vnet
   $nicSubnet = Get-AzVirtualNetworkSubnetConfig -Name 'appsubnet' -VirtualNetwork $vnet
   ```

## <a name="create-a-public-ip-address-for-the-front-end-configuration"></a>Creación de una dirección IP pública para la configuración del front-end

Cree un recurso de IP pública para la puerta de enlace de aplicaciones. Esta dirección IP pública se usa en uno de los siguientes pasos.

```powershell
$publicip = New-AzPublicIpAddress -ResourceGroupName appgw-rg -Name 'publicIP01' -Location "West US" -AllocationMethod Dynamic
```

> [!IMPORTANT]
> Application Gateway no admite el uso de una dirección IP pública creada con una etiqueta de dominio definida. Solo se admite una dirección IP pública con una etiqueta de dominio creada dinámicamente. Si necesita un nombre DNS descriptivo para la puerta de enlace de aplicaciones, se recomienda usar un registro CNAME como alias.

## <a name="create-an-application-gateway-configuration-object"></a>Creación de un objeto de configuración de la Puerta de enlace de aplicaciones

Se deben establecer todos los elementos de configuración antes de crear la puerta de enlace de aplicaciones. En los pasos siguientes, se crean los elementos de configuración necesarios para un recurso de puerta de enlace de aplicaciones.

1. Cree una configuración de dirección IP de la puerta de enlace de aplicaciones. Este valor configura qué subredes utiliza la puerta de enlace de aplicaciones. Cuando se inicia Application Gateway, elige una dirección IP de la subred configurada y redirige el tráfico de red a las direcciones IP en el grupo IP de back-end. Tenga en cuenta que cada instancia toma una dirección IP.

   ```powershell
   $gipconfig = New-AzApplicationGatewayIPConfiguration -Name 'gwconfig' -Subnet $gwSubnet
   ```

2. Cree una configuración de dirección IP de front-end. Esta configuración asigna una dirección IP pública o privada al front-end de la puerta de enlace de aplicaciones. El paso siguiente asocia la dirección IP pública del paso anterior con la configuración IP de front-end.

   ```powershell
   $fipconfig = New-AzApplicationGatewayFrontendIPConfig -Name 'fip01' -PublicIPAddress $publicip
   ```

3. Configure el grupo de direcciones IP de back-end con las direcciones IP de los servidores web back-end. Estas direcciones IP son las direcciones que reciben el tráfico de red procedente del punto de conexión de la IP del front-end. Reemplace las direcciones IP del ejemplo por sus propios puntos de conexión de direcciones IP de la aplicación.

   ```powershell
   $pool = New-AzApplicationGatewayBackendAddressPool -Name 'pool01' -BackendIPAddresses 1.1.1.1, 2.2.2.2, 3.3.3.3
   ```

   > [!NOTE]
   > Un nombre de dominio completo (FQDN) es también un valor válido para utilizarlo en lugar de una dirección IP para los servidores back-end. Se habilita mediante el conmutador **-BackendFqdns**. 

4. Configure el puerto IP de front-end para el punto de conexión de IP pública. Este puerto es el puerto al que se conectan los usuarios finales.

   ```powershell
   $fp = New-AzApplicationGatewayFrontendPort -Name 'port01'  -Port 443
   ```

5. Configure el certificado de la puerta de enlace de aplicaciones. Este certificado se usa para descifrar y volver a cifrar el tráfico de la puerta de enlace de aplicaciones.

   ```powershell
   $passwd = ConvertTo-SecureString  <certificate file password> -AsPlainText -Force 
   $cert = New-AzApplicationGatewaySSLCertificate -Name cert01 -CertificateFile <full path to .pfx file> -Password $passwd 
   ```

   > [!NOTE]
   > En este ejemplo se configura el certificado que se usa para la conexión SSL. Es preciso que el certificado tenga el formato .pfx, y que la contraseña tenga entre 4 y 12 caracteres.

6. Cree el agente de escucha HTTP para la puerta de enlace de aplicaciones. Asigne la configuración IP de front-end, el puerto y el certificado SSL que se usarán.

   ```powershell
   $listener = New-AzApplicationGatewayHttpListener -Name listener01 -Protocol Https -FrontendIPConfiguration $fipconfig -FrontendPort $fp -SSLCertificate $cert
   ```

7. Cargue el certificado que se utilizará en los recursos del grupo de back-end habilitado para SSL.

   > [!NOTE]
   > El sondeo predeterminado obtiene la clave pública del enlace SSL *predeterminado* en la dirección IP del back-end y compara el valor de la clave pública que recibe con el valor de la clave pública que se proporciona aquí. 
   > 
   > Si usa encabezados de host y de Indicación de nombre de servidor (SNI) en el back-end, es posible que la clave pública recuperada no sea el sitio previsto al que el tráfico fluirá. En caso de duda, visite https://127.0.0.1/ en los servidores back-end para confirmar qué certificado se usa para el enlace SSL *predeterminado*. Utilice la clave pública de dicha solicitud en esta sección. Si usa encabezados de host y SNI en enlaces HTTPS y no recibe una respuesta y un certificado de una solicitud manual de un explorador a https://127.0.0.1/ en los servidores back-end, debe configurar un enlace SSL de forma predeterminada en ellos. Si no lo hace, se producirán errores en los sondeos y el back-end no estará en la lista de permitidos.

   ```powershell
   $authcert = New-AzApplicationGatewayAuthenticationCertificate -Name 'whitelistcert1' -CertificateFile C:\cert.cer
   ```

   > [!NOTE]
   > El certificado proporcionado en el paso anterior debe ser la clave pública del certificado .pfx presente en el back-end. Exporte el certificado (no el certificado raíz) instalado en el servidor back-end en formato Afirmación, Evidencia y Razonamiento (CER) y utilícelo en este paso. En este paso se coloca el back-end en la lista de permitidos con la puerta de enlace de aplicaciones.

   Si usa la SKU V2 de Application Gateway, cree un certificado raíz de confianza en lugar de un certificado de autenticación. Para obtener más información, consulte [Introducción a SSL de extremo a extremo con Application Gateway](ssl-overview.md#end-to-end-ssl-with-the-v2-sku):

   ```powershell
   $trustedRootCert01 = New-AzApplicationGatewayTrustedRootCertificate -Name "test1" -CertificateFile  <path to root cert file>
   ```

8. Configure los valores HTTP para el back-end de la puerta de enlace de aplicaciones. Asigne el certificado cargado en el paso anterior a la configuración HTTP.

   ```powershell
   $poolSetting = New-AzApplicationGatewayBackendHttpSettings -Name 'setting01' -Port 443 -Protocol Https -CookieBasedAffinity Enabled -AuthenticationCertificates $authcert
   ```

   Para la SKU V2 de Application Gateway, use el siguiente comando:

   ```powershell
   $poolSetting01 = New-AzApplicationGatewayBackendHttpSettings -Name “setting01” -Port 443 -Protocol Https -CookieBasedAffinity Disabled -TrustedRootCertificate $trustedRootCert01 -HostName "test1"
   ```

9. Cree una regla de enrutamiento del equilibrador de carga que configure el comportamiento de este último. En este ejemplo, se crea una regla básica de round robin.

   ```powershell
   $rule = New-AzApplicationGatewayRequestRoutingRule -Name 'rule01' -RuleType basic -BackendHttpSettings $poolSetting -HttpListener $listener -BackendAddressPool $pool
   ```

10. Configure el tamaño de la instancia de la Puerta de enlace de aplicaciones. Los tamaños disponibles son **Standard\_Small**, **Standard\_Medium** y **Standard\_Large**.  Para la capacidad, los valores disponibles son de **1** a **10**.

    ```powershell
    $sku = New-AzApplicationGatewaySku -Name Standard_Small -Tier Standard -Capacity 2
    ```

    > [!NOTE]
    > Para las pruebas se puede elegir 1 en Número de instancias. Es importante saber que el Acuerdo de Nivel de Servicio no cubre ningún número de instancias que esté por debajo de las dos instancias y, por consiguiente, no se recomienda. Las puertas de enlace pequeñas se deben usar para pruebas de desarrollo, no con fines de producción.

11. Configure la directiva SSL que se usará en la puerta de enlace de aplicaciones. Application Gateway admite la posibilidad de establecer una versión mínima para las versiones del protocolo SSL.

    Los valores siguientes son una lista de versiones de protocolo que se pueden definir:

    - **TLSV1_0**
    - **TLSV1_1**
    - **TLSV1_2**
    
    El siguiente ejemplo establece la versión mínima del protocolo en **TLSv1_2** y solo habilita **TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256**, **TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384** y **TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256**.

    ```powershell
    $SSLPolicy = New-AzApplicationGatewaySSLPolicy -MinProtocolVersion TLSv1_2 -CipherSuite "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384", "TLS_RSA_WITH_AES_128_GCM_SHA256" -PolicyType Custom
    ```

## <a name="create-the-application-gateway"></a>Creación de la puerta de enlace de aplicaciones

Con todos los pasos anteriores, cree la puerta de enlace de aplicaciones. La creación de la puerta de enlace es un proceso que tarda mucho tiempo en ejecutarse.

```powershell
$appgw = New-AzApplicationGateway -Name appgateway -SSLCertificates $cert -ResourceGroupName "appgw-rg" -Location "West US" -BackendAddressPools $pool -BackendHttpSettingsCollection $poolSetting -FrontendIpConfigurations $fipconfig -GatewayIpConfigurations $gipconfig -FrontendPorts $fp -HttpListeners $listener -RequestRoutingRules $rule -Sku $sku -SSLPolicy $SSLPolicy -AuthenticationCertificates $authcert -Verbose
```

## <a name="apply-a-new-certificate-if-the-back-end-certificate-is-expired"></a>Aplicar un certificado nuevo si ha caducado el certificado de back-end

Utilice este procedimiento para aplicar un nuevo certificado si ha caducado el certificado de back-end.

1. Recupere la puerta de enlace de aplicaciones que se actualizará.

   ```powershell
   $gw = Get-AzApplicationGateway -Name AdatumAppGateway -ResourceGroupName AdatumAppGatewayRG
   ```
   
2. Agregue el nuevo recurso de certificado desde el archivo .cer, que contiene la clave pública del certificado y también puede ser el mismo certificado que se agrega al agente de escucha para la terminación SSL en application gateway.

   ```powershell
   Add-AzApplicationGatewayAuthenticationCertificate -ApplicationGateway $gw -Name 'NewCert' -CertificateFile "appgw_NewCert.cer" 
   ```
    
3. Obtenga el nuevo objeto de certificado de autenticación en una variable (TypeName: Microsoft.Azure.Commands.Network.Models.PSApplicationGatewayAuthenticationCertificate).

   ```powershell
   $AuthCert = Get-AzApplicationGatewayAuthenticationCertificate -ApplicationGateway $gw -Name NewCert
   ```
 
 4. Asignar el nuevo certificado en el **BackendHttp** configuración y hacer referencia a él con la variable $AuthCert. (Especifique el nombre de la configuración de HTTP que desea cambiar.)
 
   ```powershell
   $out= Set-AzApplicationGatewayBackendHttpSetting -ApplicationGateway $gw -Name "HTTP1" -Port 443 -Protocol "Https" -CookieBasedAffinity Disabled -AuthenticationCertificates $Authcert
   ```
    
 5. Confirmar el cambio en la puerta de enlace de la aplicación y pasar la nueva configuración contenida en la variable $out.
 
   ```powershell
   Set-AzApplicationGateway -ApplicationGateway $gw  
   ```

## <a name="remove-an-unused-expired-certificate-from-http-settings"></a>Quitar un certificado expirado sin usar de la configuración de HTTP

Utilice este procedimiento para quitar un certificado expirado sin usar de la configuración de HTTP.

1. Recupere la puerta de enlace de aplicaciones que se actualizará.

   ```powershell
   $gw = Get-AzApplicationGateway -Name AdatumAppGateway -ResourceGroupName AdatumAppGatewayRG
   ```
   
2. Enumera el nombre del certificado de autenticación que desea quitar.

   ```powershell
   Get-AzApplicationGatewayAuthenticationCertificate -ApplicationGateway $gw | select name
   ```
    
3. Quite el certificado de autenticación de una puerta de enlace de la aplicación.

   ```powershell
   $gw=Remove-AzApplicationGatewayAuthenticationCertificate -ApplicationGateway $gw -Name ExpiredCert
   ```
 
 4. Confirmar el cambio.
 
   ```powershell
   Set-AzApplicationGateway -ApplicationGateway $gw
   ```

   
## <a name="limit-ssl-protocol-versions-on-an-existing-application-gateway"></a>Límite de las versiones del protocolo SSL en una puerta de aplicaciones existente

Los pasos anteriores le han llevado por la creación de una aplicación con SSL de un extremo a otro y la deshabilitación de determinadas versiones del protocolo SSL. En el ejemplo siguiente se deshabilitan determinadas directivas SSL en una puerta de enlace de aplicaciones existente.

1. Recupere la puerta de enlace de aplicaciones que se actualizará.

   ```powershell
   $gw = Get-AzApplicationGateway -Name AdatumAppGateway -ResourceGroupName AdatumAppGatewayRG
   ```

2. Defina una directiva SSL. En el ejemplo siguiente, se deshabilitan **TLSv1.0** y **TLSv1.1** y los conjuntos de cifrado **TLS\_ECDHE\_ECDSA\_WITH\_AES\_128\_GCM\_SHA256**, **TLS\_ECDHE\_ECDSA\_WITH\_AES\_256\_GCM\_SHA384** y **TLS\_RSA\_WITH\_AES\_128\_GCM\_SHA256** son los únicos permitidos.

   ```powershell
   Set-AzApplicationGatewaySSLPolicy -MinProtocolVersion TLSv1_2 -PolicyType Custom -CipherSuite "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256", "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384", "TLS_RSA_WITH_AES_128_GCM_SHA256" -ApplicationGateway $gw

   ```

3. Por último, actualice la puerta de enlace. Este último paso es una tarea de ejecución prolongada. Una vez terminado, SSL de un extremo a otro está configurado en la puerta de enlace de aplicaciones.

   ```powershell
   $gw | Set-AzApplicationGateway
   ```

## <a name="get-an-application-gateway-dns-name"></a>Obtención de un nombre DNS de una puerta de enlace de aplicaciones

Una vez creada la puerta de enlace, el siguiente paso consiste en configurar el front-end para la comunicación. La puerta de enlace de aplicaciones requiere un nombre DNS asignado dinámicamente, que no es descriptivo, cuando se utiliza una dirección IP pública. Para asegurarse de que los usuarios finales puedan llegar a la puerta de enlace de aplicaciones, puede utilizar un registro CNAME para que apunte al punto de conexión público de dicha puerta. Para más información, consulte [Configuración de un nombre de dominio personalizado en Azure](../cloud-services/cloud-services-custom-domain-name-portal.md). 

Para configurar un alias, recupere los detalles de la puerta de enlace de aplicaciones y su nombre de IP o DNS asociado mediante el elemento **PublicIPAddress** asociado a dicha puerta de enlace de aplicaciones. Utilice el nombre DNS de la puerta de enlace de aplicaciones para crear un registro CNAME, que hace que las dos aplicaciones web apunten a este nombre DNS. No se recomienda el uso de registros A, ya que la IP virtual puede cambiar al reiniciarse la puerta de enlace de aplicaciones.

```powershell
Get-AzPublicIpAddress -ResourceGroupName appgw-RG -Name publicIP01
```

```
Name                     : publicIP01
ResourceGroupName        : appgw-RG
Location                 : westus
Id                       : /subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/publicIPAddresses/publicIP01
Etag                     : W/"00000d5b-54ed-4907-bae8-99bd5766d0e5"
ResourceGuid             : 00000000-0000-0000-0000-000000000000
ProvisioningState        : Succeeded
Tags                     : 
PublicIpAllocationMethod : Dynamic
IpAddress                : xx.xx.xxx.xx
PublicIpAddressVersion   : IPv4
IdleTimeoutInMinutes     : 4
IpConfiguration          : {
                                "Id": "/subscriptions/<subscription_id>/resourceGroups/appgw-RG/providers/Microsoft.Network/applicationGateways/appgwtest/frontendIP
                            Configurations/frontend1"
                            }
DnsSettings              : {
                                "Fqdn": "00000000-0000-xxxx-xxxx-xxxxxxxxxxxx.cloudapp.net"
                            }
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la protección de la seguridad de las aplicaciones web con el firewall de aplicaciones web mediante Application Gateway, consulte [Información general sobre el firewall de aplicaciones web](application-gateway-webapplicationfirewall-overview.md).

[scenario]: ./media/application-gateway-end-to-end-SSL-powershell/scenario.png
