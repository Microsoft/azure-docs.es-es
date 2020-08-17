---
title: 'Configuración de conexiones coexistentes de VPN de sitio a sitio y ExpressRoute: clásico'
description: Este artículo le guiará en la configuración de conexiones ExpressRoute y VPN de sitio a sitio que puedan coexistir en el modelo de implementación clásica.
documentationcenter: na
services: expressroute
author: charwen
ms.service: expressroute
ms.topic: how-to
ms.date: 12/06/2019
ms.author: charwen
ms.openlocfilehash: 673c0e7af4be330f3d63d2fe6ed68c73142da5d2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87084813"
---
# <a name="configure-expressroute-and-site-to-site-coexisting-connections-classic"></a>Configuración de conexiones de ExpressRoute y de sitio a sitio coexistentes (modelo de implementación clásica)
> [!div class="op_single_selector"]
> * [PowerShell: administrador de recursos](expressroute-howto-coexist-resource-manager.md)
> * [PowerShell: clásico](expressroute-howto-coexist-classic.md)
> 
> 

En este artículo se le ayuda a configurar conexiones ExpressRoute y VPN de sitio a sitio que coexisten. Tener la posibilidad de configurar VPN de sitio a sitio y ExpressRoute tiene varias ventajas. Puede configurar una VPN de sitio a sitio como una ruta de acceso seguro de conmutación por error para ExpressRoute, o bien usar las VPN de sitio a sitio para conectarse a sitios que no están conectados mediante ExpressRoute. En este artículo trataremos los pasos para configurar ambos escenarios. Este artículo se aplica al modelo de implementación clásico. Esta configuración no está disponible en el portal.

[!INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**Información sobre los modelos de implementación de Azure**

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

> [!IMPORTANT]
> Los circuitos ExpressRoute deben configurarse previamente antes de seguir las instrucciones siguientes. Asegúrese de que ha seguido las instrucciones para [crear un circuito ExpressRoute](expressroute-howto-circuit-classic.md) y [configurar el enrutamiento](expressroute-howto-routing-classic.md) antes de seguir los pasos que se indican a continuación.
> 
> 

## <a name="limits-and-limitations"></a>Límites y limitaciones
* **No se admite el enrutamiento transitorio.** No se puede realizar un enrutamiento (a través de Azure) entre una red local conectada a través de una VPN sitio a sitio y una red local conectada a través de ExpressRoute.
* **No se admite el punto a sitio.** No se pueden habilitar conexiones VPN de punto a sitio a la misma red virtual que esté conectada a ExpressRoute. VPN de punto a sitio y ExpressRoute no pueden coexistir en la misma red virtual.
* **En VPN Gateway de sitio a sitio no se puede habilitar la tunelización forzada.** Solo se puede "forzar" que todo el tráfico vinculado a Internet vuelva a la red local a través de ExpressRoute.
* **No se admite la puerta de enlace de la SKU de nivel Básico.** Debe utilizar una puerta de enlace de la SKU que no sea de nivel Básico tanto para la [puerta de enlace de ExpressRoute](expressroute-about-virtual-network-gateways.md) como para la [VPN Gateway](../vpn-gateway/vpn-gateway-about-vpngateways.md).
* **Solo se admite la VPN Gateway basada en rutas.** Debe usar una [VPN Gateway](../vpn-gateway/vpn-gateway-about-vpngateways.md) basada en rutas.
* **Se debe configurar una ruta estática para VPN Gateway.** Si la red local está conectada tanto a ExpressRoute como a una VPN de sitio a sitio, debe tener configurada una ruta estática en la red local para enrutar la conexión VPN de sitio a sitio a la red pública de Internet.
* **Primero debe configurarse la puerta de enlace de ExpressRoute.** La puerta de enlace de ExpressRoute se debe crear antes de agregar la puerta de enlace de la VPN de sitio a sitio.

## <a name="configuration-designs"></a>Diseños de configuración
### <a name="configure-a-site-to-site-vpn-as-a-failover-path-for-expressroute"></a>Configuración de una VPN de sitio a sitio como una ruta de acceso de conmutación por error para ExpressRoute
Puede configurar una conexión VPN de sitio a sitio como una copia de seguridad para ExpressRoute. Esto se aplica únicamente a las redes virtuales vinculadas a la ruta de acceso de emparejamiento privada de Azure. No hay ninguna solución de conmutación por error basada en VPN para servicios accesibles a través de emparejamientos de Microsoft y públicos de Azure. El circuito ExpressRoute siempre es el vínculo principal. Los datos fluirán a través de la ruta de acceso de VPN de sitio a sitio solo si se produce un error en el circuito ExpressRoute. 

> [!NOTE]
> Si bien se prefiere el circuito ExpressRoute a la VPN de sitio a sitio cuando ambas rutas son las mismas, Azure utilizará la coincidencia de prefijo más larga para elegir la ruta hacia el destino del paquete.
> 
> 

![Coexistencia](media/expressroute-howto-coexist-classic/scenario1.jpg)

### <a name="configure-a-site-to-site-vpn-to-connect-to-sites-not-connected-through-expressroute"></a>Configuración de una VPN de sitio a sitio para conectarse a sitios no conectados mediante ExpressRoute
Puede configurar la red para sitios que se conectan directamente a Azure mediante una VPN de sitio a sitio y otros que se conectan a través de ExpressRoute. 

![Coexistencia](media/expressroute-howto-coexist-classic/scenario2.jpg)

> [!NOTE]
> No se puede configurar una red virtual como un enrutador de tránsito.
> 
> 

## <a name="selecting-the-steps-to-use"></a>Selección de los pasos a seguir
Hay dos conjuntos diferentes de los procedimientos entre los que elegir para configurar las conexiones que pueden coexistir. El procedimiento de configuración que seleccione dependerá de si ya tiene una red virtual existente a la que quiere conectarse o si desea crear una nueva red virtual.

* No tengo una red virtual y necesito crear una.
  
    Si aún no tiene una red virtual, este procedimiento le guiará en la creación de una nueva red virtual mediante el modelo de implementación clásica y la creación de nuevas conexiones VPN de sitio a sitio y ExpressRoute. Para configurarla, siga los pasos que se describen en la sección del artículo [Creación de una nueva red virtual y conexiones coexistentes](#new).
* Ya tengo una red virtual con el modelo de implementación clásico.
  
    Puede que ya tenga una red virtual con una conexión VPN de sitio a sitio o una conexión ExpressRoute existentes. La sección [Para configurar conexiones coexistentes para una red virtual ya existente](#add) le guiará en la eliminación de la puerta de enlace y la creación de nuevas conexiones VPN de sitio a sitio y ExpressRoute. Tenga en cuenta que al crear las nuevas conexiones, se deben completar los pasos en un orden muy específico. No utilice las instrucciones que aparecen en otros artículos para crear puertas de enlace y conexiones.
  
    En este procedimiento, para crear conexiones que puedan coexistir, tendrá que eliminar la puerta de enlace y luego configurar nuevas puertas de enlace. Esto significa que tendrá tiempo de inactividad para las conexiones entre entornos mientras elimina y vuelve a crear la puerta de enlace y las conexiones, pero no necesitará migrar las máquinas virtuales o servicios a una nueva red virtual. Las máquinas virtuales y los servicios podrán seguir comunicándose con el exterior a través del equilibrador de carga mientras configura la puerta de enlace si están configurados para ello.

## <a name="install-powershell-cmdlets"></a>Instalación de los cmdlets de PowerShell

[!INCLUDE [classic powershell install instructions](../../includes/expressroute-poweshell-classic-install-include.md)]

## <a name="to-create-a-new-virtual-network-and-coexisting-connections"></a><a name="new"></a>Creación de una nueva red virtual y conexiones coexistentes
Este procedimiento le guiará en la creación de una red virtual y conexiones de sitio a sitio y ExpressRoute que coexistirán.

1. Deberá instalar la versión más reciente de los cmdlets de Azure PowerShell. Consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) para obtener más información sobre cómo instalar los cmdlets de PowerShell. Tenga en cuenta que los cmdlets que se van a utilizar en esta configuración pueden ser ligeramente diferentes de aquellos con los que podría estar familiarizado. Asegúrese de usar los cmdlets especificados en estas instrucciones. 
2. Cree un esquema para la red virtual. Para más información sobre el esquema de configuración, consulte [Esquema de configuración de Azure Virtual Network](https://msdn.microsoft.com/library/azure/jj157100.aspx).
   
    Al crear el esquema, asegúrese de que usa los valores siguientes:
   
   * La subred de puerta de enlace para la red virtual debe ser /27 o un prefijo más corto (como /26 o /25).
   * El tipo de conexión de puerta de enlace es "Dedicado".
     
    ```xml
    <VirtualNetworkSite name="MyAzureVNET" Location="Central US">
      <AddressSpace>
        <AddressPrefix>10.17.159.192/26</AddressPrefix>
      </AddressSpace>
      <Subnets>
        <Subnet name="Subnet-1">
          <AddressPrefix>10.17.159.192/27</AddressPrefix>
        </Subnet>
        <Subnet name="GatewaySubnet">
          <AddressPrefix>10.17.159.224/27</AddressPrefix>
          /Subnet>
      </Subnets>
      <Gateway>
        <ConnectionsToLocalNetwork>
          <LocalNetworkSiteRef name="MyLocalNetwork">
            <Connection type="Dedicated" />
          </LocalNetworkSiteRef>
        </ConnectionsToLocalNetwork>
      </Gateway>
    </VirtualNetworkSite>
    ```
3. Después de crear y configurar el archivo de esquema xml, cargue el archivo. Esto creará la red virtual.
   
    Use el cmdlet siguiente para cargar el archivo, reemplazando el valor por el suyo propio.
   
    ```azurepowershell
    Set-AzureVNetConfig -ConfigurationPath 'C:\NetworkConfig.xml'
    ```
4. <a name="gw"></a>Cree una puerta de enlace de ExpressRoute. Asegúrese de especificar GatewaySKU como *Standard*, *HighPerformance* o *UltraPerformance* y GatewayType como *DynamicRouting*.
   
    Use el ejemplo siguiente y sustituya los valores por los suyos propios.

    ```azurepowershell
    New-AzureVNetGateway -VNetName MyAzureVNET -GatewayType DynamicRouting -GatewaySKU HighPerformance
    ```
5. Vincule la puerta de enlace de ExpressRoute al circuito ExpressRoute. Una vez completado este paso, se ha establecido la conexión entre su red local y Azure a través de ExpressRoute.
   
    ```azurepowershell
    New-AzureDedicatedCircuitLink -ServiceKey <service-key> -VNetName MyAzureVNET
    ```
6. <a name="vpngw"></a>A continuación, cree la puerta de enlace de la VPN de sitio a sitio. Los valores de GatewaySKU deben ser *Standard*, *HighPerformance* o *UltraPerformance* y GatewayType debe ser *DynamicRouting*.
   
    ```azurepowershell
    New-AzureVirtualNetworkGateway -VNetName MyAzureVNET -GatewayName S2SVPN -GatewayType DynamicRouting -GatewaySKU  HighPerformance
    ```
   
    Para recuperar la configuración de la puerta de enlace de red virtual, lo que incluye el identificador de puerta de enlace y la dirección IP pública, use el cmdlet `Get-AzureVirtualNetworkGateway`.
   
    ```azurepowershell
    Get-AzureVirtualNetworkGateway
   
    GatewayId            : 348ae011-ffa9-4add-b530-7cb30010565e
    GatewayName          : S2SVPN
    LastEventData        :
    GatewayType          : DynamicRouting
    LastEventTimeStamp   : 5/29/2015 4:41:41 PM
    LastEventMessage     : Successfully created a gateway for the following virtual network: GNSDesMoines
    LastEventID          : 23002
    State                : Provisioned
    VIPAddress           : 104.43.x.y
    DefaultSite          :
    GatewaySKU           : HighPerformance
    Location             :
    VnetId               : 979aabcf-e47f-4136-ab9b-b4780c1e1bd5
    SubnetId             :
    EnableBgp            : False
    OperationDescription : Get-AzureVirtualNetworkGateway
    OperationId          : 42773656-85e1-a6b6-8705-35473f1e6f6a
    OperationStatus      : Succeeded
    ```
7. Cree una entidad de puerta de enlace de VPN de sitio local. Este comando no configura la puerta de enlace de VPN local. En su lugar, permite proporcionar la configuración de puerta de enlace local, como la dirección IP pública y el espacio de direcciones local, para que la puerta de enlace de VPN de Azure pueda conectarse a ella.
   
   > [!IMPORTANT]
   > El sitio local de la VPN de sitio a sitio no está definido en netcfg. En su lugar, es preciso usar este cmdlet para especificar los parámetros del sitio local. No puede definirlos ni con el portal ni con el archivo netcfg.
   > 
   > 
   
    Use el ejemplo siguiente y reemplace los valores por los suyos propios:
   
    ```azurepowershell
    New-AzureLocalNetworkGateway -GatewayName MyLocalNetwork -IpAddress <MyLocalGatewayIp> -AddressSpace <MyLocalNetworkAddress>
    ```
   
   > [!NOTE]
   > Si la red local tiene varias rutas, puede pasar todas ellas en una matriz.  $MyLocalNetworkAddress = @("10.1.2.0/24","10.1.3.0/24","10.2.1.0/24")  
   > 
   > 

    Para recuperar la configuración de la puerta de enlace de red virtual, lo que incluye el identificador de puerta de enlace y la dirección IP pública, use el cmdlet `Get-AzureVirtualNetworkGateway`. Consulte el ejemplo siguiente.

    ```azurepowershell
    Get-AzureLocalNetworkGateway

    GatewayId            : 532cb428-8c8c-4596-9a4f-7ae3a9fcd01b
    GatewayName          : MyLocalNetwork
    IpAddress            : 23.39.x.y
    AddressSpace         : {10.1.2.0/24}
    OperationDescription : Get-AzureLocalNetworkGateway
    OperationId          : ddc4bfae-502c-adc7-bd7d-1efbc00b3fe5
    OperationStatus      : Succeeded
    ```


1. Configure el dispositivo VPN local para que se conecte a la nueva puerta de enlace. Al configurar el dispositivo VPN, use la información que recuperó en el paso 6. Para obtener más información sobre la configuración del dispositivo VPN, vea [Configuración de dispositivos VPN](../vpn-gateway/vpn-gateway-about-vpn-devices.md).
2. Vincule la puerta de enlace de VPN sitio a sitio en Azure a la puerta de enlace local.
   
    En este ejemplo, connectedEntityId es el identificador de puerta de enlace local. Para encontrarlo, es preciso ejecutar `Get-AzureLocalNetworkGateway`. Para encontrar virtualNetworkGatewayId, use el cmdlet `Get-AzureVirtualNetworkGateway`. Después de este paso, se establece la conexión entre la red local y Azure a través de la conexión VPN sitio a sitio.

    ```azurepowershell
    New-AzureVirtualNetworkGatewayConnection -connectedEntityId <local-network-gateway-id> -gatewayConnectionName Azure2Local -gatewayConnectionType IPsec -sharedKey abc123 -virtualNetworkGatewayId <azure-s2s-vpn-gateway-id>
    ```

## <a name="to-configure-coexisting-connections-for-an-already-existing-vnet"></a><a name="add"></a>Para configurar conexiones coexistentes para una red virtual ya existente
Si ya tiene una red virtual, compruebe el tamaño de la subred de puerta de enlace. Si la subred de puerta de enlace es /28 o /29, primero debe eliminar la puerta de enlace de red virtual y aumentar el tamaño de la subred de puerta de enlace. Los pasos de esta sección le mostrarán cómo hacerlo.

Si la puerta de enlace es /27 o mayor y la red virtual está conectada a través de ExpressRoute, puede omitir los pasos siguientes y continuar con ["Paso 6: Creación una puerta de enlace VPN de sitio a sitio"](#vpngw) en la sección anterior.

> [!NOTE]
> Cuando elimine la puerta de enlace existente, las instalaciones locales perderán la conexión a la red virtual mientras trabaja en esta configuración.
> 
> 

1. Necesitará instalar la versión más reciente de los cmdlets de PowerShell del Administrador de recursos de Azure. Consulte [Cómo instalar y configurar Azure PowerShell](/powershell/azure/) para obtener más información sobre cómo instalar los cmdlets de PowerShell. Tenga en cuenta que los cmdlets que se van a utilizar en esta configuración pueden ser ligeramente diferentes de aquellos con los que podría estar familiarizado. Asegúrese de usar los cmdlets especificados en estas instrucciones. 
2. Elimine la puerta de enlace de la VPN de ExpressRoute o de sitio a sitio. Use el siguiente cmdlet, reemplazando los valores por los suyos propios.
   
    ```azurepowershell
    Remove-AzureVNetGateway –VnetName MyAzureVNET
    ```
3. Exporte el esquema de red virtual. Use el siguiente cmdlet de PowerShell, reemplazando los valores por los suyos propios.
   
    ```azurepowershell
    Get-AzureVNetConfig –ExportToFile "C:\NetworkConfig.xml"
    ```
4. Edite el esquema del archivo de configuración de red para que la subred de puerta de enlace sea /27 o un prefijo más corto (como /26 o /25). Consulte el ejemplo siguiente. 
   
   > [!NOTE]
   > Si no tiene suficientes direcciones IP en la red virtual para aumentar el tamaño de la subred de puerta de enlace, debe agregar más espacio de direcciones IP. Para más información sobre el esquema de configuración, consulte [Esquema de configuración de Azure Virtual Network](https://msdn.microsoft.com/library/azure/jj157100.aspx).
   > 
   > 
   
    ```xml
    <Subnet name="GatewaySubnet">
      <AddressPrefix>10.17.159.224/27</AddressPrefix>
    </Subnet>
    ```
5. Si la puerta de enlace anterior era una VPN de sitio a sitio, debe cambiar también el tipo de conexión a **Dedicado**.
   
    ```xml
    <Gateway>
      <ConnectionsToLocalNetwork>
        <LocalNetworkSiteRef name="MyLocalNetwork">
          <Connection type="Dedicated" />
        </LocalNetworkSiteRef>
      </ConnectionsToLocalNetwork>
    </Gateway>
    ```
6. Ya tiene una red virtual sin puertas de enlace. Para crear puertas de enlace y completar las conexiones, puede continuar con el [paso 4 para crear una puerta de enlace de ExpressRoute](#gw), del conjunto de pasos anterior.

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre ExpressRoute, consulte [P+F de ExpressRoute](expressroute-faqs.md)

