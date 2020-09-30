---
title: Integración de aplicaciones con Azure Virtual Network
description: Integre aplicaciones en Azure App Service con Azure Virtual Network.
author: ccompy
ms.assetid: 90bc6ec6-133d-4d87-a867-fcf77da75f5a
ms.topic: article
ms.date: 08/05/2020
ms.author: ccompy
ms.custom: seodec18
ms.openlocfilehash: 8f356cb935f1cf63408b6fbc604f139439022a4f
ms.sourcegitcommit: 3be3537ead3388a6810410dfbfe19fc210f89fec
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "89646617"
---
# <a name="integrate-your-app-with-an-azure-virtual-network"></a>Integración de su aplicación con una instancia de Azure Virtual Network

En este artículo, se describe la característica Integración con red virtual de Azure App Service y se explica cómo configurarla con aplicaciones en [Azure App Service](https://go.microsoft.com/fwlink/?LinkId=529714). Con [Azure Virtual Network][VNETOverview] (VNet), puede colocar muchos de los recursos de Azure en una red que se puede enrutar, distinta de Internet. La característica Integración con red virtual permite a las aplicaciones acceder a los recursos de una red virtual o mediante esta. La integración con redes virtuales no permite el acceso privado a las aplicaciones.

Azure App Service tiene dos variaciones en la característica de integración de la red virtual:

[!INCLUDE [app-service-web-vnet-types](../../includes/app-service-web-vnet-types.md)]

## <a name="enable-vnet-integration"></a>Habilitación de Integración con red virtual

1. Vaya a la interfaz de usuario de **Redes** en el portal de App Service. En **Integración de red virtual**, seleccione **Haga clic aquí para configurar**.

1. Seleccione **Agregar VNET**.

   ![Selección de Integración con red virtual][1]

1. La lista desplegable contiene todas las redes virtuales de Azure Resource Manager de la suscripción en la misma región. Debajo, se muestra una lista de las redes virtuales de Resource Manager en todas las demás regiones. Seleccione la VNet con la que quiere realizar la integración.

   ![Seleccione la red virtual.][2]

   * Si la VNet está en la misma región, cree una nueva subred o seleccione una subred vacía existente.
   * Para seleccionar una VNet de otra región, debe tener una puerta de enlace de VNet aprovisionada con la conectividad de punto a sitio habilitada.
   * Para integrar con una red virtual clásica, en lugar de seleccionar la lista desplegable **Virtual Network**, seleccione **Haga clic aquí para conectarse a una red virtual clásica**. Seleccione la red virtual clásica que desee usar. La VNet de destino debe tener una puerta de enlace de Virtual Network aprovisionada con la conectividad de punto a sitio habilitada.

    ![Seleccione una red virtual clásica.][3]

Durante la integración, la aplicación se reinicia. Una vez finalizada la integración, verá los detalles de la VNet con la que está integrada.

## <a name="regional-vnet-integration"></a>Integración con red virtual regional

[!INCLUDE [app-service-web-vnet-types](../../includes/app-service-web-vnet-regional.md)]

### <a name="how-regional-vnet-integration-works"></a>Funcionamiento de la versión regional de Integración con red virtual

Las aplicaciones en App Service se hospedan en roles de trabajo. Los planes de precios básico y más elevados son planes de hospedaje dedicados donde no hay ninguna otra carga de trabajo de cliente ejecutándose en los mismos trabajos. La versión regional de Integración con red virtual funciona montando interfaces virtuales con las direcciones de la subred delegada. Como la dirección de origen está en la VNet, tiene acceso a la mayoría de las cosas que están en esa VNet o a las que se puede acceder a través de ella, como cualquier máquina virtual que estuviera en la VNet. La implementación de redes es diferente de la ejecución de una máquina virtual en la VNet. Este es el motivo por el que algunas características de red todavía no están disponibles para esta característica.

![Funcionamiento de la versión regional de Integración con red virtual][5]

Cuando se habilita la versión regional de Integración con red virtual, la aplicación sigue realizando llamadas salientes a Internet utilizando los mismos canales, como hace habitualmente. Las direcciones salientes que se muestran en el portal de propiedades de la aplicación siguen siendo las direcciones usadas por la aplicación. Lo que es distinto para la aplicación es que las llamadas a servicios protegidos de punto de conexión de servicio o direcciones de RFC 1918 tienen lugar en la VNet. Si WEBSITE_VNET_ROUTE_ALL está establecido en 1, todo el tráfico de salida puede enviarse a la VNet.

La característica solo admite una interfaz virtual por trabajo. Una interfaz virtual por trabajo significa una característica Integración con red virtual regional por plan de App Service. Todas las aplicaciones en el mismo plan de App Service pueden usar la misma Integración con red virtual. Si necesita que una aplicación se conecte a una VNet adicional, debe crear otro plan de App Service. La interfaz virtual utilizada no es un recurso al que los clientes tengan acceso directo.

Dada la forma en que esta tecnología funciona, el tráfico que se usa con Integración con red virtual no se muestra en los registros de flujos de NSG o Azure Network Watcher.

## <a name="gateway-required-vnet-integration"></a>Integración con red virtual con requisito de puerta de enlace

La versión de Integración con red virtual que necesita una puerta de enlace permite conectarse a una red virtual de otra región o a una red virtual clásica. Integración con red virtual con requisito de puerta de enlace:

* Permite que una aplicación se conecte solo a una VNet a la vez.
* Permite la integración de hasta cinco VNet dentro de un plan de App Service.
* Permite que varias aplicaciones de un mismo plan de App Service usen la misma VNet sin que ello afecte al número total que se puede utilizar en un plan de App Service. Si tiene seis aplicaciones que usan la misma VNet en el mismo plan de App Service, solo se contabilizará una sola VNet en uso.
* Admite un SLA del 99,9 % debido al SLA de la puerta de enlace.
* Permite que las aplicaciones utilicen el DNS con el que se ha configurado la VNet.
* Para poder conectarse a la aplicación, necesita una puerta de enlace basada en rutas de Virtual Network que esté configurada con una VPN de punto a sitio SSTP.

La versión de Integración con red virtual que necesita una puerta de enlace no puede utilizarse en las siguientes circunstancias:

* Con una VNet conectada a Azure ExpressRoute.
* Desde una aplicación para Linux
* Para acceder a recursos protegidos por puntos de conexión de servicio.
* Con una puerta de enlace coexistente que admita tanto VPN de ExpressRoute como VPN de punto a sitio o de sitio a sitio.

### <a name="set-up-a-gateway-in-your-azure-virtual-network"></a>Configuración de una puerta de enlace en Azure Virtual Network ###

Para crear una puerta de enlace:

1. [Cree una subred de puerta de enlace][creategatewaysubnet] en su red virtual.

1. [Cree la instancia de VPN Gateway][creategateway]. Seleccione un tipo de VPN basada en rutas.

1. [Establezca las direcciones de punto a sitio][setp2saddresses]. Si la puerta de enlace no está en la SKU básica, IKEV2 debe estar deshabilitado en la configuración de punto a sitio y SSTP debe estar seleccionado. El espacio de direcciones de punto a sitio debe especificarse en bloques de direcciones de RFC 1918 (10.0.0.0/8, 172.16.0.0/12 y 192.168.0.0/16).

Si crea la puerta de enlace para su uso con Integración con red virtual de App Service, no es necesario cargar un certificado. La creación de la puerta de enlace puede tardar 30 minutos. No podrá integrar su aplicación con la VNet hasta que la puerta de enlace esté aprovisionada.

### <a name="how-gateway-required-vnet-integration-works"></a>Funcionamiento de la versión de Integración con red virtual que necesita una puerta de enlace

La versión de Integración con red virtual que necesita una puerta de enlace se basa en la tecnología VPN de punto a sitio. Las VPN de punto a sitio limitan el acceso de red a la máquina virtual que hospeda la aplicación. Las aplicaciones están restringidas para enviar tráfico a Internet solo a través de conexiones híbridas o de la integración con red virtual. Cuando la aplicación se ha configurado con el portal para que utilice la versión de Integración con red virtual que necesita una puerta de enlace, se entabla una compleja negociación en su nombre para crear y asignar certificados en la puerta de enlace y en la aplicación. El resultado es que los trabajos que se usan para hospedar las aplicaciones pueden conectarse directamente a la puerta de enlace de red virtual de la VNet seleccionada.

![Funcionamiento de la versión de Integración con red virtual que necesita una puerta de enlace][6]

### <a name="access-on-premises-resources"></a>Acceso a recursos locales

Las aplicaciones pueden acceder a los recursos locales mediante la integración con redes virtuales que tengan conexiones de sitio a sitio. Si usa la Integración con red virtual con requisito de puerta de enlace, actualice las rutas de puerta de enlace de red virtual locales con los bloques de direcciones de punto a sitio. En la configuración inicial de la VPN de sitio a sitio, los scripts que se usan para configurarla deben configurar las rutas correctamente. Si agrega las direcciones de punto a sitio después de crear la VPN de sitio a sitio, deberá actualizar las rutas manualmente. El procedimiento varía según la puerta de enlace y no se describe aquí. No se puede tener BGP configurado con una conexión VPN de sitio a sitio.

No se requiere ninguna configuración adicional para que la característica Integración con red virtual regional acceda a través de la red virtual a los recursos locales. Basta con conectar la red virtual a los recursos de entorno local mediante ExpressRoute o una VPN de sitio a sitio.

> [!NOTE]
> La característica Integración con red virtual con requisito de puerta de enlace no integra una aplicación con una red virtual que tiene una puerta de enlace de ExpressRoute. Aunque la puerta de enlace de ExpressRoute esté configurada en [modo de coexistencia][VPNERCoex], la Integración con red virtual no funciona. Si necesita acceder a los recursos mediante una conexión de ExpressRoute, utilice la característica Integración con red virtual regional o un entorno [App Service Environment][ASE] que se ejecute en la red virtual.
>
>

### <a name="peering"></a>Emparejamiento

Si usa el emparejamiento con Integración con red virtual regional, no es necesario configurar nada más.

Si usa la Integración con red virtual con requisito de puerta de enlace con el emparejamiento, deberá configurar algunos elementos más. Para configurar el emparejamiento para que funcione con su aplicación:

1. Agregue una conexión de emparejamiento en la red virtual a la que se conecta su aplicación. Al agregar la conexión de emparejamiento, habilite **Permitir acceso a red virtual** y active **Permitir tráfico reenviado** y **Permitir tránsito de puerta de enlace**.
1. Agregue una conexión de emparejamiento en la VNet que se está emparejando con la VNet a la que está conectado. Al agregar la conexión de emparejamiento en la VNet de destino, habilite **Permitir acceso a red virtual** y seleccione **Permitir tráfico reenviado** y **Allow remote gateways** (Permitir puertas de enlace remotas).
1. Vaya a la interfaz de usuario **Plan de App Service** > **Redes** > **Integración con red virtual** en el portal. Seleccione la red virtual a la que está conectada su aplicación. En la sección de enrutamiento, agregue el intervalo de direcciones de la VNet que está emparejada con la VNet a la que está conectada su aplicación.

## <a name="manage-vnet-integration"></a>Administración de Integración con red virtual

La conexión y la desconexión con una red virtual tiene lugar en el nivel de la aplicación. Las operaciones que pueden afectar a la integración con red virtual en varias aplicaciones se encuentran en un nivel del plan de App Service. En el portal de la aplicación > **Redes** > **Integración con red virtual**, puede obtener detalles sobre la red virtual. Puede encontrar información similar en el nivel del plan de App Service en el portal **Plan de App Service** > **Redes** > **Integración con red virtual**.

La única operación que puede realizar en la vista de aplicación de la instancia de Integración con red virtual es desconectar la aplicación de la red virtual si está conectada a ella. Para desconectar la aplicación de una red virtual, seleccione **Desconectar**. La aplicación se reinicia cuando se desconecta de una red virtual. La desconexión no cambia la red virtual. La subred o la puerta de enlace no se quitan. Si después quiere eliminar la red virtual, primero desconecte la aplicación de la red virtual y elimine los recursos que hay en ella, como las puertas de enlace.

La interfaz de usuario de Integración con red virtual del plan de App Service muestra todas las integraciones de redes virtuales usadas por las aplicaciones de su plan de App Service. Para ver más detalles sobre cada red virtual, seleccione la red virtual que le interese. Hay dos acciones que puede realizar aquí en relación con la versión de Integración con red virtual que necesita una puerta de enlace:

* **Sincronizar red**: la operación de sincronización de red solo se usa con la característica Integración con red virtual con requisito de puerta de enlace. Realizar una operación de sincronización de red garantiza que los certificados y la información de red estén sincronizados. Si agrega o cambia el DNS de la red virtual, debe ejecutar una operación de sincronización de la red. Esta operación reinicia todas las aplicaciones que usen esta red virtual. Esta operación no funcionará si usa una aplicación y una red virtual que pertenezcan a distintas suscripciones.
* **Agregar rutas**: la adición de rutas impulsa el tráfico de salida a la red virtual.

### <a name="gateway-required-vnet-integration-routing"></a>Enrutamiento de Integración con red virtual con requisito de puerta de enlace
Las rutas definidas en la red virtual se usan para dirigir el tráfico desde la aplicación hacia la red virtual. Para enviar más tráfico de salida a la red virtual, agregue aquí esos bloques de direcciones. Esta característica solo funciona con la Integración con red virtual con requisito de puerta de enlace. Las tablas de enrutamiento no afectan del mismo modo al tráfico de la aplicación cuando se utiliza la versión de Integración con red virtual que necesita una puerta de enlace del mismo modo que cuando se utiliza la versión regional.

### <a name="gateway-required-vnet-integration-certificates"></a>Certificados de Integración con red virtual con requisito de puerta de enlace
Cuando la Integración con red virtual con requisito de puerta de enlace está habilitada, existe un intercambio necesario de certificados para garantizar la seguridad de la conexión. Junto con los certificados, se incluyen la configuración de DNS, las rutas y otros elementos similares que describen la red.

Si los certificados o la información de red cambian, es necesario hacer clic en **Sincronizar red**. Al seleccionar **Sincronizar red**, se producirá una breve interrupción en la conectividad entre la aplicación y la red virtual. Aunque no se reinicia la aplicación, la pérdida de conectividad podría hacer que su sitio no funcione correctamente.

## <a name="pricing-details"></a>Detalles de precios
La característica Integración con red virtual regional no tiene ningún cargo extra por uso más allá de los cargos del plan de tarifa del plan de App Service.

Hay tres cargos relacionados con el uso de la característica Integración con red virtual con requisito de puerta de enlace:

* **Cargos de plan de tarifa del plan de App Service**: Las aplicaciones deben estar en un plan de App Service Estándar, Premium o PremiumV2. Para obtener más información sobre estos costes, consulte [Precios de App Service][ASPricing].
* **Costos de la transferencia de datos**: Se aplica un cargo de salida de datos, aunque la red virtual esté en el mismo centro de datos. Estos cargos se describen en la página de [detalles de precios de Transferencia de datos][DataPricing].
* **Costos de VPN Gateway**: Existe un costo para la puerta de enlace de red virtual necesario para la VPN de punto a sitio. Para obtener más información, vea [Precios de VPN Gateway][VNETPricing].

## <a name="troubleshooting"></a>Solución de problemas

[!INCLUDE [app-service-web-vnet-troubleshooting](../../includes/app-service-web-vnet-troubleshooting.md)]

## <a name="automation"></a>Automation

Hay disponible compatibilidad con la CLI para la versión regional de Integración con red virtual. Para acceder a los comandos siguientes, [instale la CLI de Azure][installCLI].

```azurecli
az webapp vnet-integration --help

Group
    az webapp vnet-integration : Methods that list, add, and remove virtual network
    integrations from a webapp.
        This command group is in preview. It may be changed/removed in a future release.
Commands:
    add    : Add a regional virtual network integration to a webapp.
    list   : List the virtual network integrations on a webapp.
    remove : Remove a regional virtual network integration from webapp.

az appservice vnet-integration --help

Group
    az appservice vnet-integration : A method that lists the virtual network
    integrations used in an appservice plan.
        This command group is in preview. It may be changed/removed in a future release.
Commands:
    list : List the virtual network integrations used in an appservice plan.
```

PowerShell también ofrece compatibilidad para la integración con una red virtual regional, pero es necesario crear un recurso genérico con una matriz de propiedades de la subred resourceID.

```azurepowershell
# Parameters
$sitename = 'myWebApp'
$resourcegroupname = 'myRG'
$VNetname = 'myVNet'
$location = 'myRegion'
$integrationsubnetname = 'myIntegrationSubnet'
$subscriptionID = 'aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee'

#Property array with the SubnetID
$properties = @{
  subnetResourceId = "/subscriptions/$subscriptionID/resourceGroups/$resourcegroupname/providers/Microsoft.Network/virtualNetworks/$VNetname/subnets/$integrationsubnetname"
}

#Creation of the VNet integration
$vNetParams = @{
  ResourceName = "$sitename/VirtualNetwork"
  Location = $location
  ResourceGroupName = $resourcegroupname
  ResourceType = 'Microsoft.Web/sites/networkConfig'
  PropertyObject = $properties
}
New-AzResource @vNetParams
```


En la versión de Integración con red virtual que necesita una puerta de enlace, puede integrar App Service con una instancia de Azure Virtual Network utilizando PowerShell. Para obtener un script que esté listo para ejecutarse, consulte [Conexión de una aplicación de Azure App Service a una red virtual de Azure](https://gallery.technet.microsoft.com/scriptcenter/Connect-an-app-in-Azure-ab7527e3).


<!--Image references-->
[1]: ./media/web-sites-integrate-with-vnet/vnetint-app.png
[2]: ./media/web-sites-integrate-with-vnet/vnetint-addvnet.png
[3]: ./media/web-sites-integrate-with-vnet/vnetint-classic.png
[5]: ./media/web-sites-integrate-with-vnet/vnetint-regionalworks.png
[6]: ./media/web-sites-integrate-with-vnet/vnetint-gwworks.png


<!--Links-->
[VNETOverview]: ../virtual-network/virtual-networks-overview.md
[AzurePortal]: https://portal.azure.com/
[ASPricing]: https://azure.microsoft.com/pricing/details/app-service/
[VNETPricing]: https://azure.microsoft.com/pricing/details/vpn-gateway/
[DataPricing]: https://azure.microsoft.com/pricing/details/data-transfers/
[V2VNETP2S]: ../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md
[ILBASE]: environment/create-ilb-ase.md
[V2VNETPortal]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md
[VPNERCoex]: ../expressroute/expressroute-howto-coexist-resource-manager.md
[ASE]: environment/intro.md
[creategatewaysubnet]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#creategw
[creategateway]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#creategw
[setp2saddresses]: ../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md#addresspool
[VNETRouteTables]: ../virtual-network/manage-route-table.md
[installCLI]: /cli/azure/install-azure-cli?view=azure-cli-latest%2f
[privateendpoints]: networking/private-endpoint.md