---
title: Conexión de una puerta de enlace de red virtual a una de Azure Virtual WAN
description: Este artículo le ayuda a conectar una puerta de enlace de red virtual de Azure a una puerta de enlace de VPN de Azure Virtual WAN
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: how-to
ms.date: 07/28/2020
ms.author: cherylmc
ms.openlocfilehash: 9d94904e580cefb53b2c71d21259bebfc07c1ad6
ms.sourcegitcommit: 0b8320ae0d3455344ec8855b5c2d0ab3faa974a3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87431281"
---
# <a name="connect-a-vpn-gateway-virtual-network-gateway-to-virtual-wan"></a>Conexión de una VPN Gateway (puerta de enlace de red virtual) a una Virtual WAN

Este artículo le ayuda a configurar la conectividad de una VPN Gateway de Azure (puerta de enlace de red) a una Azure Virtual WAN (puerta de enlace de VPN). La creación de una conexión desde un VPN Gateway (puerta de enlace de red virtual) a una Virtual WAN (puerta de enlace de VPN) es similar a la configuración de la conectividad a una WAN virtual desde sitios VPN de rama.

Con el fin de minimizar la posible confusión entre dos características, se dirigirá la puerta de enlace con el nombre de la característica a la que se hace referencia. Por ejemplo, puerta de enlace de red virtual de VPN Gateway  y puerta de enlace de VPN Virtual WAN.

## <a name="before-you-begin"></a>Antes de empezar

Antes de empezar, cree los siguientes recursos:

Azure Virtual WAN

* [Creación de una WAN virtual](virtual-wan-site-to-site-portal.md#openvwan).
* [Creación de un centro de conectividad](virtual-wan-site-to-site-portal.md#hub). El centro de conectividad virtual contiene la puerta de enlace de VPN de Virtual WAN.

Azure Virtual Network

* Cree una red virtual sin ninguna puerta de enlace de red virtual. Compruebe que ninguna de las subredes de sus redes locales se superpone a las redes virtuales a las que quiere conectarse. Para crear una red virtual en Azure Portal consulte este [Inicio rápido](../virtual-network/quick-create-portal.md).

## <a name="1-create-a-vpn-gateway-virtual-network-gateway"></a><a name="vnetgw"></a>1. Creación de una puerta de enlace de red virtual de VPN Gateway

Cree una puerta de enlace de red virtual de **VPN Gateway** en modo activo-activo para la red virtual. Al crear la puerta de enlace, puede usar las direcciones IP públicas existentes para las dos instancias de la puerta de enlace o puede crear nuevas direcciones IP públicas. Estas direcciones IP públicas se usarán al configurar los sitios de Virtual WAN. Para más información sobre los pasos de configuración y las puertas de enlace VPN activo-activo, consulte [Configuración de puertas de enlace VPN activo-activo](../vpn-gateway/vpn-gateway-activeactive-rm-powershell.md#aagateway).

### <a name="active-active-mode-setting"></a><a name="active-active"></a>Configuración del modo activo/activo

En la página **Configuración** de la puerta de enlace de red virtual, habilite el modo activo/activo.

![activo-activo](./media/connect-virtual-network-gateway-vwan/active.png "activa-activa")

### <a name="bgp-setting"></a><a name="BGP"></a>Configuración de BGP

En la página **Configuración** de la puerta de enlace de red virtual, puede configurar **ASN BGP**. Cambie el ASN BGP. El ASN del protocolo de puerta de enlace de borde (BGP) no puede ser 65515. Azure Virtual WAN usará 66515.

![BGP](./media/connect-virtual-network-gateway-vwan/bgp.png "bgp")

### <a name="public-ip-addresses"></a><a name="pip"></a>Direcciones IP públicas

Después de crea la puerta de enlace, vaya a la página **Propiedades**. Las propiedades y los valores de configuración serán similares al ejemplo siguiente. Tenga en cuenta las dos direcciones IP públicas que se usan para la puerta de enlace.

![properties](./media/connect-virtual-network-gateway-vwan/publicip.png "properties")

## <a name="2-create-virtual-wan-vpn-sites"></a><a name="vwansite"></a>2. Creación de sitios VPN de Virtual WAN

Para crear sitios VPN de Virtual WAN, vaya a su WAN virtual y, en **Conectividad**, elija **Sitios VPN**. En esta sección, creará dos sitios VPN de Virtual WAN que corresponden a las puertas de enlace de red virtual que creó en la sección anterior.

1. Elija **+Crear sitio**.
2. En la página **Crear sitios VPN**, escriba los valores siguientes:

   * **Región**: la misma región que la puerta de enlace de red virtual de Azure VPN Gateway.
   * **Proveedor del dispositivo**: escriba el proveedor del dispositivo (cualquier nombre).
   * **Espacio de direcciones privadas**: escriba un valor o déjelo en blanco si el protocolo de puerta de enlace de borde está habilitado.
   * **Protocolo de puerta de enlace de borde**: seleccione **Habilitar** si la puerta de enlace de red virtual de Azure VPN Gateway tiene este protocolo habilitado.
   * **Conectar a centros de conectividad**: seleccione en la lista desplegable el centro de conectividad que creó en los requisitos previos. Si no ve ningún centro, compruebe que ha creado una puerta de enlace de VPN de sitio a sitio para el centro.
3. En **Vínculos**, escriba los siguientes valores:

   * **Nombre del proveedor**: escriba un nombre de vínculo y un nombre de proveedor (cualquier nombre).
   * **Velocidad**: velocidad (cualquier número).
   * **Dirección IP**: escriba la dirección IP (igual que la primera dirección IP pública que se muestra en las propiedades de la puerta de enlace de red virtual [VPN Gateway]).
   * **Direcciones BGP** y **ASN**: dirección BGP y ASN. Deben ser las mismas que una de las direcciones IP del mismo nivel BGP y ASN de la puerta de enlace de red virtual VPN Gateway que configuró en el [Paso 1](#vnetgw).
4. Revise y haga clic en **Confirmar** para crear el sitio.
5. Repita los pasos anteriores para crear el segundo sitio para que coincida con la segunda instancia de la puerta de enlace de red virtual VPN Gateway. Conservará la misma configuración, excepto el uso de la segunda dirección IP pública y la segunda dirección IP del mismo nivel BGP de la configuración de VPN Gateway.
6. Ahora tiene dos sitios aprovisionados correctamente y puede continuar con la sección siguiente para descargar los archivos de configuración.

## <a name="3-download-the-vpn-configuration-files"></a><a name="downloadconfig"></a>3. Descargar los archivos de configuración de VPN

En esta sección, descargará el archivo de configuración de VPN para cada uno de los sitios que creó en la sección anterior.

1. En la parte superior de la página **Sitios VPN** de Virtual WAN, elija el **Sitio** y luego seleccione **Descargar la configuración de VPN de sitio a sitio**. Azure crea un archivo de configuración con los valores.

   ![descargar archivo de configuración](./media/connect-virtual-network-gateway-vwan/download.png "descarga")
2. Descargue y abra el archivo de configuración.
3. Repita estos pasos para el segundo sitio. Una vez que tenga abiertos ambos archivos de configuración, puede continuar con la siguiente sección.

## <a name="4-create-the-local-network-gateways"></a><a name="createlocalgateways"></a>4. Creación de las puertas de enlace de red local

En esta sección, creará dos puertas de enlace de red local de Azure VPN Gateway. Los archivos de configuración del paso anterior contienen los valores de configuración de la puerta de enlace. Use estos valores para crear y configurar las puertas de enlace de red local de Azure VPN Gateway.

1. Cree la puerta de enlace de red local con estos valores. Para más información sobre cómo crear una puerta de enlace de red local VPN Gateway, consulte el artículo de VPN Gateway [Creación de una puerta de enlace de red local](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md#LocalNetworkGateway).

   * **Dirección IP**: Use la dirección IP de Instance0 que se muestra para *gatewayconfiguration* del archivo de configuración.
   * **BGP**: Si la conexión es a través de BGP, seleccione **Configurar los valores de BGP** y escriba el ASN '65515'. Escriba la dirección IP del mismo nivel BGP. Use 'Instance0 BgpPeeringAddresses' para *gatewayconfiguration* del archivo de configuración.
   * **Suscripción, Grupo de recursos y Ubicación** son los mismos para el centro de conectividad Virtual WAN.
2. Revise y cree la puerta de enlace de red local. La puerta de enlace de red local debe ser similar a la de este ejemplo.

   ![descargar archivo de configuración](./media/connect-virtual-network-gateway-vwan/lng1.png "instance0")
3. Repita estos pasos para crear otra puerta de enlace de red local, pero esta vez use los valores de 'Instance1' en lugar de los valores de 'Instance0' del archivo de configuración.

   ![descargar archivo de configuración](./media/connect-virtual-network-gateway-vwan/lng2.png "instance1")

## <a name="5-create-connections"></a><a name="createlocalgateways"></a>5. Crear conexiones

En esta sección, creará una conexión entre las puertas de enlace de red local VPN Gateway y la puerta de enlace de red virtual. Para conocer los pasos para crear una conexión VPN Gateway, consulte [Configurar una conexión](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md#CreateConnection).

1. En el portal, vaya a la puerta de enlace de red virtual y haga clic en **Conexiones**. En la parte superior de la página Conexiones, haga clic en **+Agregar** para abrir la página **Agregar conexión**.
2. En la página **Agregar conexión**, configure los valores de la conexión:

   * **Nombre:** asigne un nombre a la conexión.
   * **Tipo de conexión**: Seleccione **Sitio a sitio (IPSec)**
   * **Puerta de enlace de red virtual:** el valor es fijo porque se conecta desde esta puerta de enlace.
   * **Puerta de enlace de red local:** Esta conexión conectará la puerta de enlace de red virtual y la puerta de enlace de red local. Elija una de las puertas de enlace de red local que creó anteriormente.
   * **Clave compartida**: Ingrese una clave compartida.
   * **Protocolo TLS:** Elija el protocolo IKE.
3. Haga clic en **Aceptar** para crear la conexión.
4. La conexión se puede ver en la página **Conexiones** de la puerta de enlace de red virtual.

   ![Connection](./media/connect-virtual-network-gateway-vwan/connect.png "conexión")
5. Repita los pasos anteriores para crear otra red virtual. Para la segunda conexión, elija la otra puerta de enlace de red local que creó.
6. Si las conexiones han usado el protocolo de puerta de enlace de borde, después de crearlas vaya a una conexión y seleccione **Configuración**. En la página **Configuración**, en **BGP**, seleccione **Habilitado**. A continuación, haga clic en **Guardar**. Repita la operación en la segunda conexión.

## <a name="6-test-connections"></a><a name="test"></a>6. Prueba de conexiones

Para probar la conectividad, puede crear dos máquinas virtuales, una en el lateral de la puerta de enlace de red virtual de VPN Gateway y otra en una red virtual para la Virtual WAN y luego hacer ping en las dos máquinas virtuales.

1. Cree una máquina virtual en la red virtual (Test1-VNet) para Azure VPN Gateway (Test1-VNG). No cree la máquina virtual en GatewaySubnet.
2. Cree otra red virtual para conectarse a la WAN virtual. Cree una máquina virtual en una subred de esta red virtual. Esta red virtual no puede contener ninguna puerta de enlace de red virtual. Puede crear rápidamente una red virtual con los pasos de PowerShell que se describen en el artículo [Conexión de sitio a sitio](virtual-wan-site-to-site-portal.md#vnet). Asegúrese de cambiar los valores antes de ejecutar los cmdlets.
3. Conecte la VNet a la central de Virtual WAN. En la página de la red WAN virtual, haga clic en **Conexiones de red virtual** luego en **+Agregar conexión**. En la página **Agregar conexión**, rellene los campos siguientes:

    * **Nombre de la conexión**: asigne un nombre a la conexión.
    * **Centros**: seleccione el concentrador que desea asociar a esta conexión.
    * **Suscripción**: compruebe la suscripción.
    * **Red virtual**: seleccione la red virtual que quiere conectar con este concentrador. La red virtual no puede tener una puerta de enlace de red virtual ya existente.
4. Haga clic en **Aceptar** para crear la conexión de red virtual.
5. Ahora se establece la conectividad entre las máquinas virtuales. Debe poder hacer ping a una máquina virtual desde la otra, a menos que haya firewalls u otras directivas que bloqueen la comunicación.

## <a name="next-steps"></a>Pasos siguientes

Para conocer los pasos para configurar una directiva IPsec personalizada, consulte [Configuración de una directiva IPsec personalizada para Virtual WAN](virtual-wan-custom-ipsec-portal.md).
Para obtener más información sobre Virtual WAN, vea [Acerca de Azure Virtual WAN](virtual-wan-about.md) y [Preguntas más frecuentes sobre Azure Virtual WAN](virtual-wan-faq.md).
