---
title: 'Conexión de una red virtual a varios sitios mediante VPN Gateway: Clásico'
description: Conecte varios sitios locales a una red virtual clásica con una instancia de VPN Gateway.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: yushwang
ms.openlocfilehash: 93916bb1eb92447da45ebae7ccb2e61c5af1dd9b
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89441585"
---
# <a name="add-a-site-to-site-connection-to-a-vnet-with-an-existing-vpn-gateway-connection-classic"></a>Agregar una conexión de sitio a sitio a una red virtual con una conexión de VPN Gateway existente (clásico)

[!INCLUDE [deployment models](../../includes/vpn-gateway-classic-deployment-model-include.md)]

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)
> * [PowerShell (clásico)](vpn-gateway-multi-site.md)
>
>

Este artículo le guiará a través del uso de PowerShell para agregar conexiones de sitio a sitio (S2S) a VPN Gateway con una conexión existente. Este tipo de conexión se denomina con frecuencia, configuración "multisitio". Los pasos de este artículo se aplican a las redes virtuales creadas con el modelo de implementación clásica (también conocido como Service Management). Estos pasos no se aplican a las configuraciones de conexión coexistentes de ExpressRoute/sitio a sitio.

### <a name="deployment-models-and-methods"></a>Modelos de implementación y métodos

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

Esta tabla se actualiza cada vez que hay nuevos artículos y nuevas herramientas disponibles para esta configuración. Cuando aparezca un artículo, creamos un vínculo directo a él desde esta tabla.

[!INCLUDE [vpn-gateway-table-multi-site](../../includes/vpn-gateway-table-multisite-include.md)]

## <a name="about-connecting"></a>Acerca de la conexión

Puede conectar varios sitios locales a una única red virtual. Esto resulta especialmente atractivo para crear soluciones híbridas en la nube. La creación de una conexión de varios sitios en la puerta de enlace de red virtual de Azure es similar a la creación de otras conexiones de sitio a sitio. De hecho, puede utilizar una Azure VPN Gateway existente, siempre que la puerta de enlace sea dinámica (basada en ruta).

Si ya se ha conectado una puerta de enlace estática a la red virtual, puede cambiar el tipo de puerta de enlace a dinámico sin tener que recompilar la red virtual para dar cabida a varios sitios. Antes de cambiar el tipo de enrutamiento, asegúrese de que la VPN Gateway local admite configuraciones de VPN basadas en ruta.

![Diagrama de multisitio](./media/vpn-gateway-multi-site/multisite.png "multisitio")

## <a name="points-to-consider"></a>Puntos que se deben tener en cuenta

**No podrá usar el portal para realizar cambios en esta red virtual.** Tiene que realizar los cambios en el archivo de configuración de red en lugar de usar el portal. Si realiza cambios en el portal, sobrescribirán la configuración de referencia a varios sitios de esta red virtual.

Debe sentirse cómodo al usar el archivo de configuración de red en el momento en el que complete el procedimiento de varios sitios. Sin embargo, si hay más personas que trabajan en la configuración de red, tendrá que asegurarse de que todos conocen esta limitación. Esto no significa que no pueda usar el portal en absoluto. Puede usarlo para todo lo demás menos para hacer cambios de configuración en esta red virtual concreta.

## <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar la configuración, compruebe que dispone de lo siguiente:

* Hardware VPN compatible para cada ubicación local. Consulte [Acerca de los dispositivos VPN para conectividad de red virtual](vpn-gateway-about-vpn-devices.md) para comprobar si el dispositivo que quiere usar es un dispositivo que se sabe que es compatible.
* Una dirección IP IPv4 pública orientada externamente para cada dispositivo VPN. La dirección IP no se puede ubicar detrás de un NAT. Esto es un requisito.
* Alguna persona con experiencia en configuración de hardware de VPN Necesitará un conocimiento amplio de cómo configurar el dispositivo VPN o trabajar con alguien que lo tenga.
* Los intervalos de dirección IP que desea usar para la red virtual (si aún no ha creado uno).
* Los intervalos de direcciones IP para cada uno de los sitios de red locales a los que se va a conectar. Tendrá que asegurarse de que los intervalos de dirección IP para cada uno de los sitios de red locales a los que desea conectarse no se solapan. De lo contrario, el portal o la API de REST rechazarán la configuración que se carga.<br>Por ejemplo, si dispone de dos sitios de red locales que contienen el intervalo de dirección IP 10.2.3.0/24 y dispone de un paquete con una dirección de destino 10.2.3.3, Azure no sabrá a qué sitio desea enviar el paquete porque se solapan los intervalos de dirección. Para evitar problemas de enrutamiento, Azure no loe permite cargar un archivo de configuración que disponga de intervalos que se solapan.

### <a name="working-with-azure-powershell"></a>Trabajo con Azure PowerShell

[!INCLUDE [vpn-gateway-classic-powershell](../../includes/vpn-gateway-powershell-classic-locally.md)]

## <a name="1-create-a-site-to-site-vpn"></a>1. Crear una VPN de sitio a sitio
Si ya tiene una VPN de sitio a sitio con una puerta de enlace de enrutamiento dinámico, perfecto. Puede pasar a [Exportación de la configuración de la red virtual](#export). De lo contrario, haga lo siguiente:

### <a name="if-you-already-have-a-site-to-site-virtual-network-but-it-has-a-static-policy-based-routing-gateway"></a>Si ya dispone de una red virtual de sitio a sitio, pero con una puerta de enlace de enrutamiento estático (basada en directivas):
1. Cambie el tipo de puerta de enlace a enrutamiento dinámico. Una VPN de varios sitios requiere una puerta de enlace de enrutamiento dinámico (también denominada basada en ruta). Para cambiar el tipo de puerta de enlace, tendrá que eliminar primero la puerta de enlace existente y, a continuación, crear una nueva.
2. Configure la nueva puerta de enlace y cree un túnel de VPN. Para obtener instrucciones, consulte [Especificación del tipo de VPN y SKU](vpn-gateway-howto-site-to-site-classic-portal.md#sku). Asegúrese de especificar el tipo de enrutamiento como "Dinámico".

### <a name="if-you-dont-have-a-site-to-site-virtual-network"></a>Si no dispone de una red virtual de sitio a sitio:
1. Cree la red virtual de sitio a sitio mediante las instrucciones que se describen en: [Creación de una red virtual con una conexión VPN de sitio a sitio](vpn-gateway-site-to-site-create.md).  
2. Configure una puerta de enlace de enrutamiento mediante estas instrucciones: [Configuración de una instancia de VPN Gateway](vpn-gateway-configure-vpn-gateway-mp.md). Asegúrese de seleccionar **enrutamiento dinámico** como tipo de puerta de enlace.

## <a name="2-export-the-network-configuration-file"></a><a name="export"></a>2. Exportar el archivo de configuración de red

Abra la consola de PowerShell con privilegios elevados. Para cambiar a la administración de servicios, use este comando:

```powershell
azure config mode asm
```

Conéctese a su cuenta. Use el siguiente ejemplo para conectarse:

```powershell
Add-AzureAccount
```

Exporte el archivo de configuración de red de Azure mediante la ejecución del comando siguiente. Puede cambiar la ubicación del archivo que se va a exportar a una ubicación diferente si es necesario.

```powershell
Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
```

## <a name="3-open-the-network-configuration-file"></a>3. Abrir el archivo de configuración de red
Abra el archivo de configuración de red que descargó en el último paso. Use el editor xml que desee. El archivo debe tener un aspecto similar al siguiente:

```xml
<NetworkConfiguration xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/ServiceHosting/2011/07/NetworkConfiguration">
  <VirtualNetworkConfiguration>
    <LocalNetworkSites>
      <LocalNetworkSite name="Site1">
        <AddressSpace>
          <AddressPrefix>10.0.0.0/16</AddressPrefix>
          <AddressPrefix>10.1.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>131.2.3.4</VPNGatewayAddress>
      </LocalNetworkSite>
      <LocalNetworkSite name="Site2">
        <AddressSpace>
          <AddressPrefix>10.2.0.0/16</AddressPrefix>
          <AddressPrefix>10.3.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>131.4.5.6</VPNGatewayAddress>
      </LocalNetworkSite>
    </LocalNetworkSites>
    <VirtualNetworkSites>
      <VirtualNetworkSite name="VNet1" AffinityGroup="USWest">
        <AddressSpace>
          <AddressPrefix>10.20.0.0/16</AddressPrefix>
          <AddressPrefix>10.21.0.0/16</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="FE">
            <AddressPrefix>10.20.0.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="BE">
            <AddressPrefix>10.20.1.0/24</AddressPrefix>
          </Subnet>
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.20.2.0/29</AddressPrefix>
          </Subnet>
        </Subnets>
        <Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="Site1">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
          </ConnectionsToLocalNetwork>
        </Gateway>
      </VirtualNetworkSite>
    </VirtualNetworkSites>
  </VirtualNetworkConfiguration>
</NetworkConfiguration>
```

## <a name="4-add-multiple-site-references"></a>4. Agregar referencias de varios sitios
Cuando agregue o quite la información de referencia del sitio, realizará cambios de configuración en ConnectionsToLocalNetwork/LocalNetworkSiteRef. Si se agrega una nueva referencia a sitio local se activa Azure para la creación de un nuevo túnel. En el ejemplo siguiente, la configuración de red es para una conexión de sitio único. Cuando haya terminado de realizar los cambios, guarde el archivo.

```xml
  <Gateway>
    <ConnectionsToLocalNetwork>
      <LocalNetworkSiteRef name="Site1"><Connection type="IPsec" /></LocalNetworkSiteRef>
    </ConnectionsToLocalNetwork>
  </Gateway>
```

Para agregar referencias a sitios adicionales (crear una configuración de varios sitios), basta con agregar líneas "LocalNetworkSiteRef" adicionales, como se muestra en el ejemplo siguiente:

```xml
  <Gateway>
    <ConnectionsToLocalNetwork>
      <LocalNetworkSiteRef name="Site1"><Connection type="IPsec" /></LocalNetworkSiteRef>
      <LocalNetworkSiteRef name="Site2"><Connection type="IPsec" /></LocalNetworkSiteRef>
    </ConnectionsToLocalNetwork>
  </Gateway>
```

## <a name="5-import-the-network-configuration-file"></a>5. Importar el archivo de configuración de red
Importe un archivo de configuración de red. Cuando importe este archivo con los cambios, se agregarán nuevos túneles. Los túneles usarán la puerta de enlace dinámica que creó anteriormente. Puede usar PowerShell para importar el archivo.

## <a name="6-download-keys"></a>6. Descargar las claves
Una vez que se han agregado los nuevos túneles, use el cmdlet de PowerShell 'Get-AzureVNetGatewayKey' para obtener las claves IPsec/IKE compartidas previamente para cada túnel.

Por ejemplo:

```powershell
Get-AzureVNetGatewayKey –VNetName "VNet1" –LocalNetworkSiteName "Site1"
Get-AzureVNetGatewayKey –VNetName "VNet1" –LocalNetworkSiteName "Site2"
```

Si lo prefiere, también puede usar la API de REST de *Obtención de la clave compartida de puerta de enlace de la red virtual* para obtener claves compartidas previamente.

## <a name="7-verify-your-connections"></a>7. Comprobación de las conexiones
Compruebe el estado del túnel de varios sitios. Después de descargar las claves para cada túnel, querrá comprobar las conexiones. Use 'Get-AzureVnetConnection' para obtener una lista de túneles de redes virtuales, como se muestra en el siguiente ejemplo. VNet1 es el nombre de la red virtual.

```powershell
Get-AzureVnetConnection -VNetName VNET1
```

Valor devuelto del ejemplo:

```
    ConnectivityState         : Connected
    EgressBytesTransferred    : 661530
    IngressBytesTransferred   : 519207
    LastConnectionEstablished : 5/2/2014 2:51:40 PM
    LastEventID               : 23401
    LastEventMessage          : The connectivity state for the local network site 'Site1' changed from Not Connected to Connected.
    LastEventTimeStamp        : 5/2/2014 2:51:40 PM
    LocalNetworkSiteName      : Site1
    OperationDescription      : Get-AzureVNetConnection
    OperationId               : 7f68a8e6-51e9-9db4-88c2-16b8067fed7f
    OperationStatus           : Succeeded

    ConnectivityState         : Connected
    EgressBytesTransferred    : 789398
    IngressBytesTransferred   : 143908
    LastConnectionEstablished : 5/2/2014 3:20:40 PM
    LastEventID               : 23401
    LastEventMessage          : The connectivity state for the local network site 'Site2' changed from Not Connected to Connected.
    LastEventTimeStamp        : 5/2/2014 2:51:40 PM
    LocalNetworkSiteName      : Site2
    OperationDescription      : Get-AzureVNetConnection
    OperationId               : 7893b329-51e9-9db4-88c2-16b8067fed7f
    OperationStatus           : Succeeded
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las VPN Gateway, consulte [Acerca de las VPN Gateway](vpn-gateway-about-vpngateways.md).
