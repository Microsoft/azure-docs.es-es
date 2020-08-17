---
title: 'Creación de un sondeo personalizado mediante el modelo de implementación clásica: Azure Application Gateway'
description: Aprenda a crear un sondeo personalizado para la puerta de enlace de aplicaciones mediante PowerShell en el modelo de implementación clásica.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/13/2019
ms.author: victorh
ms.openlocfilehash: 1445d1418bde6d5d15e365c59ceb56e7661faccb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87088077"
---
# <a name="create-a-custom-probe-for-azure-application-gateway-classic-by-using-powershell"></a>Creación de un sondeo personalizado para la Puerta de enlace de aplicaciones de Azure (clásica) mediante PowerShell

> [!div class="op_single_selector"]
> * [Azure Portal](application-gateway-create-probe-portal.md)
> * [PowerShell del Administrador de recursos de Azure](application-gateway-create-probe-ps.md)
> * [Azure Classic PowerShell](application-gateway-create-probe-classic-ps.md)

En este artículo, agregará un sondeo personalizado a una puerta de enlace de aplicaciones existente con PowerShell. Los sondeos personalizados son útiles para aplicaciones que tienen una página de comprobación del estado o para aplicaciones que no proporcionan una respuesta correcta en la aplicación web predeterminada.

> [!IMPORTANT]
> Azure tiene dos modelos de implementación diferentes para crear recursos y trabajar con ellos: [Resource Manager y el clásico](../azure-resource-manager/management/deployment-models.md). En este artículo se trata el modelo de implementación clásico. Microsoft recomienda que las implementaciones más recientes usen el modelo de Resource Manager. Obtenga información sobre cómo [realizar estos pasos con el modelo de Resource Manager](application-gateway-create-probe-ps.md).

[!INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## <a name="create-an-application-gateway"></a>Creación de una puerta de enlace de aplicaciones

Para crear una Puerta de enlace de aplicaciones:

1. Cree un recurso de Application Gateway.
2. Cree un archivo de configuración XML o un objeto de configuración.
3. Confirme la configuración para el recurso de la Puerta de enlace de aplicaciones recién creado.

### <a name="create-an-application-gateway-resource-with-a-custom-probe"></a>Creación de un recurso de puerta de enlace de aplicaciones con un sondeo personalizado

Para crear la puerta de enlace, use el cmdlet `New-AzureApplicationGateway` y reemplace los valores por los suyos. La facturación de la puerta de enlace no se inicia en este momento. La facturación comienza en un paso posterior, cuando la puerta de enlace se ha iniciado correctamente.

En el ejemplo siguiente se crea una puerta de enlace de aplicaciones nueva mediante una red virtual denominada testvnet1 y una subred llamada subnet-1.

```powershell
New-AzureApplicationGateway -Name AppGwTest -VnetName testvnet1 -Subnets @("Subnet-1")
```

Para validar la creación de la puerta de enlace, puede usar el cmdlet `Get-AzureApplicationGateway`.

```powershell
Get-AzureApplicationGateway AppGwTest
```

> [!NOTE]
> El valor predeterminado de *InstanceCount* es 2, con un valor máximo de 10. El valor predeterminado de *GatewaySize* es Medium. Puede elegir entre Pequeño, Mediano y Grande.
> 
> 

*VirtualIPs* y *DnsName* se muestran en blanco porque todavía no se ha iniciado la puerta de enlace. Estos valores se crearán una vez que la puerta de enlace esté en estado de ejecución.

### <a name="configure-an-application-gateway-by-using-xml"></a>Configuración de una puerta de enlace de aplicaciones mediante XML

En el ejemplo siguiente, se usa un archivo XML para configurar todos los valores de la puerta de enlace de aplicaciones y confirmarlos en el recurso de dicha puerta de enlace.  

Copie el texto siguiente y péguelo en el Bloc de notas.

```xml
<ApplicationGatewayConfiguration xmlns:i="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/windowsazure">
<FrontendIPConfigurations>
    <FrontendIPConfiguration>
        <Name>fip1</Name>
        <Type>Private</Type>
    </FrontendIPConfiguration>
</FrontendIPConfigurations>
<FrontendPorts>
    <FrontendPort>
        <Name>port1</Name>
        <Port>80</Port>
    </FrontendPort>
</FrontendPorts>
<Probes>
    <Probe>
        <Name>Probe01</Name>
        <Protocol>Http</Protocol>
        <Host>contoso.com</Host>
        <Path>/path/custompath.htm</Path>
        <Interval>15</Interval>
        <Timeout>15</Timeout>
        <UnhealthyThreshold>5</UnhealthyThreshold>
    </Probe>
    </Probes>
    <BackendAddressPools>
    <BackendAddressPool>
        <Name>pool1</Name>
        <IPAddresses>
            <IPAddress>1.1.1.1</IPAddress>
            <IPAddress>2.2.2.2</IPAddress>
        </IPAddresses>
    </BackendAddressPool>
</BackendAddressPools>
<BackendHttpSettingsList>
    <BackendHttpSettings>
        <Name>setting1</Name>
        <Port>80</Port>
        <Protocol>Http</Protocol>
        <CookieBasedAffinity>Enabled</CookieBasedAffinity>
        <RequestTimeout>120</RequestTimeout>
        <Probe>Probe01</Probe>
    </BackendHttpSettings>
</BackendHttpSettingsList>
<HttpListeners>
    <HttpListener>
        <Name>listener1</Name>
        <FrontendIP>fip1</FrontendIP>
    <FrontendPort>port1</FrontendPort>
        <Protocol>Http</Protocol>
    </HttpListener>
</HttpListeners>
<HttpLoadBalancingRules>
    <HttpLoadBalancingRule>
        <Name>lbrule1</Name>
        <Type>basic</Type>
        <BackendHttpSettings>setting1</BackendHttpSettings>
        <Listener>listener1</Listener>
        <BackendAddressPool>pool1</BackendAddressPool>
    </HttpLoadBalancingRule>
</HttpLoadBalancingRules>
</ApplicationGatewayConfiguration>
```

Edite los valores entre paréntesis de los elementos de configuración. Guarde el archivo con extensión .xml.

En el ejemplo siguiente se muestra cómo usar un archivo de configuración para configurar la puerta de enlace de aplicaciones para que equilibre la carga de tráfico HTTP en el puerto público 80 y envíe el tráfico de red al puerto back-end 80 entre dos direcciones IP mediante sondeo personalizado.

> [!IMPORTANT]
> El elemento de protocolo Http o Https distingue mayúsculas de minúsculas.

Se agrega el nuevo elemento de configuración \<Probe\> para configurar sondeos personalizados.

Los parámetros de configuración son:

|Parámetro|Descripción|
|---|---|
|**Nombre** |Nombre de referencia del sondeo personalizado. |
| **Protocolo** | Protocolo usado (los valores posibles son HTTP o HTTPS).|
| **Host** y **Path** | Dirección URL completa que invoca la puerta de enlace de aplicaciones para determinar el mantenimiento de la instancia. Por ejemplo, si tiene el sitio web http:\//contoso.com/, el sondeo personalizado se puede configurar para "http:\//contoso.com/path/custompath.htm", con el fin de que las comprobaciones del sondeo tengan una respuesta HTTP satisfactoria.|
| **Intervalo** | Configura las comprobaciones de intervalo de sondeo en segundos.|
| **Tiempo de espera** | Define el tiempo de espera de sondeo para una comprobación de respuesta HTTP.|
| **UnhealthyThreshold** | El número de respuestas HTTP con error que es necesario para marcar la instancia del back-end como *incorrecta*.|

Se hace referencia al nombre del sondeo en la configuración de \<BackendHttpSettings\> para asignar el grupo de back-end que usa la configuración de sondeo personalizado.

## <a name="add-a-custom-probe-to-an-existing-application-gateway"></a>Adición de un sondeo personalizado a una puerta de enlace de aplicaciones existente

El cambio de la configuración actual de una puerta de enlace de aplicaciones requiere tres pasos: obtener el archivo de configuración XML actual, modificarlo para que tenga un sondeo personalizado y configurar la puerta de enlace de aplicaciones con la nueva configuración XML.

1. Obtenga el archivo XML mediante `Get-AzureApplicationGatewayConfig`. De esta forma, el cmdlet exportará el XML de configuración que se debe modificar para agregar una configuración de sondeo.

   ```powershell
   Get-AzureApplicationGatewayConfig -Name "<application gateway name>" -Exporttofile "<path to file>"
   ```

1. Abra el archivo XML en un editor de texto. Agregue una sección `<probe>` después de `<frontendport>`.

   ```xml
   <Probes>
    <Probe>
        <Name>Probe01</Name>
        <Protocol>Http</Protocol>
        <Host>contoso.com</Host>
        <Path>/path/custompath.htm</Path>
        <Interval>15</Interval>
        <Timeout>15</Timeout>
        <UnhealthyThreshold>5</UnhealthyThreshold>
    </Probe>
   </Probes>
   ```

   En la sección backendHttpSettings del XML, agregue el nombre del sondeo tal y como se muestra en el ejemplo siguiente:

   ```xml
    <BackendHttpSettings>
        <Name>setting1</Name>
        <Port>80</Port>
        <Protocol>Http</Protocol>
        <CookieBasedAffinity>Enabled</CookieBasedAffinity>
        <RequestTimeout>120</RequestTimeout>
        <Probe>Probe01</Probe>
    </BackendHttpSettings>
   ```

   Guarde el archivo XML.

1. Actualice la configuración de la instancia de Application Gateway con el nuevo archivo XML mediante `Set-AzureApplicationGatewayConfig`. De esta forma, el cmdlet actualizará la instancia de Application Gateway con la nueva configuración.

```powershell
Set-AzureApplicationGatewayConfig -Name "<application gateway name>" -Configfile "<path to file>"
```

## <a name="next-steps"></a>Pasos siguientes

Si quiere configurar la descarga de Seguridad de la capa de transporte (TLS), previamente conocida como Capa de sockets seguros (SSL), vea [Configuración de una puerta de enlace de aplicaciones para la descarga TLS](application-gateway-ssl.md).

Si quiere configurar una puerta de enlace de aplicaciones para usarla con el equilibrador de carga interno, consulte [Creación de una puerta de enlace de aplicaciones con un equilibrador de carga interno (ILB)](application-gateway-ilb.md).

