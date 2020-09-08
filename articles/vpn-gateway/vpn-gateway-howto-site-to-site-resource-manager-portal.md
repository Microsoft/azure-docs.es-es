---
title: 'Conexión de una red local a una instancia de Azure Virtual Network: VPN de sitio a sitio: Portal'
description: Cree una conexión de puerta de enlace VPN de sitio a sitio de IPsec desde la red local a una red virtual de Azure a través de la red pública de Internet mediante el portal.
services: vpn-gateway
titleSuffix: Azure VPN Gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 03/03/2020
ms.author: cherylmc
ms.openlocfilehash: ebfd03935f5189a544f11e5b8bbdd4b46e2aa989
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88037072"
---
# <a name="create-a-site-to-site-connection-in-the-azure-portal"></a>Creación de una conexión de sitio a sitio mediante Azure Portal

Este artículo muestra cómo usar Azure Portal para crear una conexión de puerta de enlace VPN de sitio a sitio desde la red local a la red virtual. Los pasos descritos en este artículo se aplican al modelo de implementación de Resource Manager. También se puede crear esta configuración con una herramienta o modelo de implementación distintos, mediante la selección de una opción diferente en la lista siguiente:

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-howto-site-to-site-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md)
> * [CLI](vpn-gateway-howto-site-to-site-resource-manager-cli.md)
> * [Portal de Azure clásico](vpn-gateway-howto-site-to-site-classic-portal.md)
> 
>

Se utiliza una conexión de puerta de enlace VPN de sitio a sitio para conectar su red local a una red virtual de Azure a través de un túnel VPN de IPsec/IKE (IKEv1 o IKEv2). Este tipo de conexión requiere un dispositivo VPN local que tenga una dirección IP pública asignada. Para más información acerca de las puertas de enlace VPN, consulte [Acerca de VPN Gateway](vpn-gateway-about-vpngateways.md).

![Diagrama de la conexión entre locales de VPN Gateway de sitio a sitio](./media/vpn-gateway-howto-site-to-site-resource-manager-portal/site-to-site-diagram.png)

## <a name="before-you-begin"></a>Antes de empezar

Antes de comenzar con la configuración, compruebe que se cumplen los criterios siguientes:

* Asegúrese de tener un dispositivo VPN compatible y alguien que pueda configurarlo. Para más información acerca de los dispositivos VPN compatibles y su configuración, consulte [Acerca de los dispositivos VPN](vpn-gateway-about-vpn-devices.md).
* Compruebe que tiene una dirección IPv4 pública externa para el dispositivo VPN.
* Si no está familiarizado con los intervalos de direcciones IP ubicados en la red local, necesita trabajar con alguien que pueda proporcionarle estos detalles. Al crear esta configuración, debe especificar los prefijos del intervalo de direcciones IP al que Azure enrutará la ubicación local. Ninguna de las subredes de la red local puede superponerse con las subredes de la red virtual a la que desea conectarse. 

### <a name="example-values"></a><a name="values"></a>Valores del ejemplo

Los ejemplos de este artículo utilizan los valores siguientes. Puede usar estos valores para crear un entorno de prueba o hacer referencia a ellos para comprender mejor los ejemplos de este artículo. Para más información sobre la configuración de VPN Gateway en general, consulte [Acerca de la configuración de VPN Gateway](vpn-gateway-about-vpn-gateway-settings.md).

* **Nombre de la red virtual:** VNet1
* **Espacio de direcciones:** 10.1.0.0/16
* **Suscripción:** seleccione la suscripción que desea usar
* **Grupos de recursos:** TestRG1
* **Región:** Este de EE. UU.
* **Subred:** front-end: 10.1.0.0/24, back-end: 10.1.1.0/24 (opcional para este ejercicio)
* **Intervalo de direcciones de subred de puerta de enlace:** 10.1.255.0/27
* **Nombre de la puerta de enlace de red virtual:** VNet1GW
* **Dirección IP pública:** VNet1GWpip
* **Tipo de VPN:** basada en rutas
* **Tipo de conexión**: de sitio a sitio (IPsec)
* **Tipo de puerta de enlace:** VPN
* **Nombre de la puerta de enlace de red local:** Site1
* **Nombre de la conexión:** VNet1toSite1
* **Clave compartida:** en este ejemplo, usamos abc123. Sin embargo, puede usar cualquiera compatible con el hardware VPN. Lo importante es que los valores coincidan en ambos lados de la conexión.

## <a name="1-create-a-virtual-network"></a><a name="CreatVNet"></a>1. Creación de una red virtual

[!INCLUDE [Create a virtual network](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="2-create-the-vpn-gateway"></a><a name="VNetGateway"></a>2. Creación de la puerta de enlace VPN

En este paso, se crea la puerta de enlace para la red virtual. La creación de una puerta de enlace suele tardar 45 minutos o más, según la SKU de la puerta de enlace seleccionada.

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-portal-include.md)]

### <a name="example-settings"></a>Configuración de ejemplo

* **Detalles de instancia > Región:** Este de EE. UU.
* **Red virtual > Red virtual:** VNet1
* **Detalles de instancia > Nombre:** VNet1GW
* **Detalles de instancia > Tipo de puerta de enlace:** VPN
* **Detalles de instancia > Tipo de VPN:** basada en rutas
* **Red virtual > Intervalo de direcciones de subred de la puerta de enlace:** 10.1.255.0/27
* **Dirección IP pública > Nombre de dirección IP pública:** VNet1GWpip

[!INCLUDE [Create a vpn gateway](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

[!INCLUDE [NSG warning](../../includes/vpn-gateway-no-nsg-include.md)]


## <a name="3-create-the-local-network-gateway"></a><a name="LocalNetworkGateway"></a>3. Creación de la puerta de enlace de red local

La puerta de enlace de red local suele hacer referencia a la ubicación local. Asigne al sitio un nombre al que Azure pueda hacer referencia y, luego, especifique la dirección IP del dispositivo VPN local con la que creará una conexión. Especifique también los prefijos de dirección IP que se enrutarán a través de la puerta de enlace VPN al dispositivo VPN. Los prefijos de dirección que especifique son los prefijos que se encuentran en la red local. Si la red local cambia o necesita cambiar la dirección IP pública del dispositivo VPN, puede actualizar fácilmente los valores más adelante.

**Valores de ejemplo**

* **Nombre:** Site1
* **Grupos de recursos:** TestRG1
* **Ubicación:** Este de EE. UU.


[!INCLUDE [Add a local network gateway](../../includes/vpn-gateway-add-local-network-gateway-portal-include.md)]

## <a name="4-configure-your-vpn-device"></a><a name="VPNDevice"></a>4. Configurar el dispositivo VPN

Las conexiones de sitio a sitio a una red local requieren un dispositivo VPN. En este paso, se configura el dispositivo VPN. Al configurar el dispositivo VPN, necesita lo siguiente:

- Una clave compartida. Se trata de la misma clave compartida que se especifica al crear la conexión VPN de sitio a sitio. En estos ejemplos se utiliza una clave compartida básica. Se recomienda que genere y utilice una clave más compleja.
- La dirección IP pública de la puerta de enlace de red virtual. Puede ver la dirección IP pública mediante Azure Portal, PowerShell o la CLI. Para buscar la dirección IP pública de la puerta de enlace de la VPN mediante Azure Portal, vaya a **Puertas de enlace de red virtual** y haga clic en el nombre de la puerta de enlace.

[!INCLUDE [Configure a VPN device](../../includes/vpn-gateway-configure-vpn-device-include.md)]

## <a name="5-create-the-vpn-connection"></a><a name="CreateConnection"></a>5. Creación de la conexión VPN

Creación de la conexión VPN de sitio a sitio entre la puerta de enlace de la red virtual y el dispositivo VPN local.

[!INCLUDE [Add a site-to-site connection](../../includes/vpn-gateway-add-site-to-site-connection-portal-include.md)]

## <a name="6-verify-the-vpn-connection"></a><a name="VerifyConnection"></a>6. Comprobación de la conexión VPN

[!INCLUDE [Verify the connection](../../includes/vpn-gateway-verify-connection-portal-include.md)]

## <a name="to-connect-to-a-virtual-machine"></a><a name="connectVM"></a>Conexión a una máquina virtual

[!INCLUDE [Connect to a VM](../../includes/vpn-gateway-connect-vm-s2s-include.md)]

## <a name="how-to-reset-a-vpn-gateway"></a><a name="reset"></a>Procedimientos para restablecer una puerta de enlace de VPN

Restablecer una puerta de enlace de VPN de Azure es útil si se pierde la conectividad VPN entre locales en uno o varios túneles VPN de sitio a sitio. En esta situación, todos tus dispositivos VPN locales funcionan correctamente, pero no pueden establecer túneles IPsec con las Puertas de enlace de VPN de Azure. Para conocer los pasos, consulte [Restablecimiento de una puerta de enlace de VPN](vpn-gateway-resetgw-classic.md).

## <a name="how-to-change-a-gateway-sku-resize-a-gateway"></a><a name="resize"></a>Procedimientos para cambiar la SKU de una puerta de enlace (cambiar el tamaño de una puerta de enlace)

Para que conocer los pasos para cambiar la SKU de una puerta de enlace, consulte [SKU de puerta de enlace](vpn-gateway-about-vpn-gateway-settings.md#gwsku).

## <a name="how-to-add-an-additional-connection-to-a-vpn-gateway"></a><a name="addconnect"></a>Procedimiento para agregar una conexión adicional a una puerta de enlace VPN

Puede agregar conexiones adicionales, siempre que ninguno de los espacios de direcciones se superpongan entre las conexiones.

1. Para agregar una conexión adicional, vaya a la puerta de enlace VPN y, a continuación, haga clic en **Conexiones** para abrir la página Conexiones.
2. Haga clic en **+Agregar** para agregar la conexión. Ajuste el tipo de conexión para reflejar una conexión entre redes virtuales (si se conecta a otra puerta de enlace de red virtual), o de sitio a sitio.
3. Si se conecta de sitio a sitio y aún no ha creado una puerta de enlace de red local para el sitio al que desea conectarse, puede crear una.
4. Especifique la clave compartida que desee usar y, después, haga clic en **Aceptar** para crear la conexión.

## <a name="next-steps"></a>Pasos siguientes

* Para más información acerca de BGP, consulte [Información general de BGP](vpn-gateway-bgp-overview.md) y [Configuración de BGP](vpn-gateway-bgp-resource-manager-ps.md).
* Para información acerca de la tunelización forzada, consulte la [Acerca de la tunelización forzada](vpn-gateway-forced-tunneling-rm.md).
* Para obtener información acerca de las conexiones activo/activo de alta disponibilidad, consulte [Conectividad de alta disponibilidad entre locales y de red virtual a red virtual](vpn-gateway-highlyavailable.md).
* Para información acerca de cómo limitar el tráfico de red a los recursos de una red virtual, vea [Seguridad de red](../virtual-network/security-overview.md).
* Para información acerca de cómo enruta Azure el tráfico entre los recursos locales, de Internet y de Azure, vea [Enrutamiento del tráfico de redes virtuales](../virtual-network/virtual-networks-udr-overview.md).
* Para información acerca de cómo crear una conexión VPN de sitio a sitio mediante una plantilla de Azure Resource Manager, consulte [Create a Site-to-Site VPN Connection](https://azure.microsoft.com/resources/templates/101-site-to-site-vpn-create/) (Creación de una conexión VPN de sitio a sitio).
* Para información acerca de cómo crear una conexión VPN entre redes virtuales mediante una plantilla de Azure Resource Manager, consulte [Deploy HBase geo replication](https://azure.microsoft.com/resources/templates/101-hdinsight-hbase-replication-geo/) (Implementación de replicación geográfica de HBase).
