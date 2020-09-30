---
title: 'Configuración de una conexión de VPN Gateway entre redes virtuales: Portal de Azure'
description: Crear una conexión de VPN Gateway entre redes virtuales mediante Resource Manager y Azure Portal.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: cherylmc
ms.openlocfilehash: b14f3c241fd21d3d4f9c4e17ae95d8048139cec1
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89442878"
---
# <a name="configure-a-vnet-to-vnet-vpn-gateway-connection-by-using-the-azure-portal"></a>Configuración de una conexión de VPN Gateway de red virtual a red virtual mediante Azure Portal

Este artículo le ayuda a conectarse a redes virtuales mediante el tipo de conexión de red virtual a red virtual. Las redes virtuales pueden estar en distintas regiones y provenir de distintas suscripciones. Al conectar redes virtuales provenientes de distintas suscripciones, no es necesario que las suscripciones estén asociadas con el mismo inquilino de Active Directory. 

![diagrama de v2v](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/v2vrmps.png)

Los pasos descritos en este artículo se aplican al modelo de implementación de Azure Resource Manager y utilizan Azure Portal. Puede crear esta configuración con un modelo o una herramienta de implementación diferente a través de las opciones que se describen en estos artículos:

> [!div class="op_single_selector"]
> * [Azure Portal](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md)
> * [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md)
> * [CLI de Azure](vpn-gateway-howto-vnet-vnet-cli.md)
> * [Portal de Azure clásico](vpn-gateway-howto-vnet-vnet-portal-classic.md)
> * [Conexión de diferentes modelos de implementación - Azure Portal](vpn-gateway-connect-different-deployment-models-portal.md)
> * [Conexión de diferentes modelos de implementación - PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
>
>

## <a name="about-connecting-vnets"></a>Acerca de la conexión de redes virtuales

En las secciones siguientes se describen las distintas formas de conectar redes virtuales.

### <a name="vnet-to-vnet"></a>De red virtual a red virtual

Configurar una conexión de red virtual a red virtual es una forma sencilla de conectar redes virtuales. Cuando conecta una red virtual a otra mediante el tipo de conexión entre redes virtuales (VNet2VNet) es parecida a la creación de una conexión IPsec de sitio a sitio en una ubicación local. Ambos tipos de conectividad usan una puerta de enlace de VPN para proporcionar un túnel seguro mediante IPsec/IKE y funcionan de la misma forma en lo relativo a la comunicación. Sin embargo, la manera en que se configura la puerta de enlace de red local es distinta. 

Cuando se crea una conexión de red virtual a red virtual, el espacio de direcciones de la puerta de enlace de red local se crea y rellena automáticamente. Si actualiza el espacio de direcciones de una de las redes virtuales, la otra enruta automáticamente al espacio de direcciones actualizado. Por lo general, es más rápido y sencillo crear una conexión de red virtual a red virtual que una conexión de sitio a sitio.

### <a name="site-to-site-ipsec"></a>De sitio a sitio (IPsec)

Si trabaja con una configuración de red complicada, puede que, en su lugar, prefiera conectar las redes virtuales mediante una [conexión de sitio a sitio](vpn-gateway-howto-site-to-site-resource-manager-portal.md). Cuando se siguen las instrucciones para la conexión IPsec de sitio a sitio, las puertas de enlace de red locales se crean y se configuran manualmente. La puerta de enlace de red local de cada red virtual trata a la otra red virtual como un sitio local. Estos pasos le permiten especificar espacios de direcciones adicionales para que la puerta de enlace de red local enrute el tráfico. Si el espacio de direcciones de una red virtual cambia, debe actualizar automáticamente la puerta de enlace de red local correspondiente.

### <a name="vnet-peering"></a>Emparejamiento de VNET

También puede conectar las redes virtuales a través del emparejamiento de redes virtuales. El emparejamiento de VNET no utiliza una puerta de enlace de VPN y tiene distintas restricciones. Además, [los precios del emparejamiento de VNET](https://azure.microsoft.com/pricing/details/virtual-network) se calculan de forma diferente que los [precios de VPN Gateway entre redes virtuales](https://azure.microsoft.com/pricing/details/vpn-gateway). Para más información, consulte [Emparejamiento de VNET](../virtual-network/virtual-network-peering-overview.md).

## <a name="why-create-a-vnet-to-vnet-connection"></a>¿Por qué crear una conexión de red virtual a red virtual?

Puede que desee conectar redes virtuales con una conexión entre redes virtuales por las siguientes razones:

### <a name="cross-region-geo-redundancy-and-geo-presence"></a>Presencia geográfica y redundancia geográfica entre regiones

  * Puede configurar su propia replicación geográfica o sincronización con conectividad segura sin recurrir a los puntos de conexión a Internet.
  * Con Azure Traffic Manager y Azure Load Balancer, puede configurar cargas de trabajo de alta disponibilidad con redundancia geográfica en varias regiones de Azure. Por ejemplo, puede configurar grupos de disponibilidad AlwaysOn de SQL Server en varias regiones de Azure.

### <a name="regional-multi-tier-applications-with-isolation-or-administrative-boundaries"></a>Aplicaciones regionales de niveles múltiples con aislamiento o límites administrativos

  * Dentro de la misma región, se pueden configurar aplicaciones de niveles múltiples con varias redes virtuales conectadas entre sí debido a requisitos de aislamiento o administrativos.

Se puede combinar la comunicación entre redes virtuales con configuraciones de varios sitios. Estas configuraciones permiten establecer topologías de red que combinan la conectividad entre entornos locales con la conectividad entre redes virtuales, como se muestra en el diagrama siguiente:

![Acerca de las conexiones](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/aboutconnections.png "Acerca de las conexiones")

En este artículo se muestra cómo conectar redes virtuales mediante el tipo de conexión de red virtual a red virtual. Cuando sigue estos pasos como ejercicio, puede usar estos valores de configuración de ejemplo. En el ejemplo, las redes virtuales se encuentran en la misma suscripción, pero en distintos grupos de recursos. Si las redes virtuales se encuentran en distintas suscripciones, no se puede crear la conexión en el portal. En su lugar, use [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md) o la [CLI](vpn-gateway-howto-vnet-vnet-cli.md). Para más información sobre las conexiones de red virtual a red virtual, consulte las [preguntas más frecuentes sobre red virtual a red virtual](#vnet-to-vnet-faq).

### <a name="example-settings"></a>Configuración de ejemplo

**Valores de VNet1:**

- **Configuración de la red virtual**
    - **Name**: VNet1
    - **Espacio de direcciones**: 10.1.0.0/16
    - **Suscripción**: Seleccione la suscripción que quiere usar.
    - **Grupo de recursos**: TestRG1
    - **Ubicación**: Este de EE. UU.
    - **Subred**
        - **Name**: FrontEnd
        - **Intervalo de direcciones**: 10.1.0.0/24
    - **Subred de puerta de enlace**:
        - **Name**: *GatewaySubnet* se rellena automáticamente
        - **Intervalo de direcciones**: 10.1.255.0/27

- **Configuración de puerta de enlace de red virtual**
    - **Name**: VNet1GW
    - **Tipo de puerta de enlace**: Seleccione **VPN**.
    - **Tipo de VPN**: seleccione **Basada en rutas**.
    - **SKU**: Seleccione la SKU de puerta de enlace que quiere usar.
    - **Nombre de dirección IP pública**: VNet1GWpip
    - **Connection**
       - **Name**: VNet1toVNet4
       - **Clave compartida**: Puede crear usted mismo la clave compartida. Cuando crea la conexión entre las redes virtuales, los valores deben coincidir. Para este ejercicio, use abc123.

**Valores de VNet4:**

- **Configuración de la red virtual**
   - **Name**: VNet4
   - **Espacio de direcciones**: 10.41.0.0/16
   - **Suscripción**: Seleccione la suscripción que quiere usar.
   - **Grupo de recursos**: TestRG4
   - **Ubicación**: Oeste de EE. UU.
   - **Subred** 
      - **Name**: FrontEnd
      - **Intervalo de direcciones**: 10.41.0.0/24
   - **GatewaySubnet** 
      - **Name**: *GatewaySubnet* se rellena automáticamente
      - **Intervalo de direcciones**: 10.41.255.0/27

- **Configuración de puerta de enlace de red virtual** 
    - **Name**: VNet4GW
    - **Tipo de puerta de enlace**: Seleccione **VPN**.
    - **Tipo de VPN**: seleccione **Basada en rutas**.
    - **SKU**: Seleccione la SKU de puerta de enlace que quiere usar.
    - **Nombre de dirección IP pública**: VNet4GWpip
    - **Connection** 
       - **Name**: VNet4toVNet1
       - **Clave compartida**: Puede crear usted mismo la clave compartida. Cuando crea la conexión entre las redes virtuales, los valores deben coincidir. Para este ejercicio, use abc123.

## <a name="create-and-configure-vnet1"></a>Creación y configuración de VNet1
Si ya dispone de una red virtual, compruebe que la configuración sea compatible con el diseño de la puerta de enlace de VPN. Preste especial atención a las subredes que se pueden superponer con otras redes. La conexión no funcionará correctamente si tiene subredes superpuestas.

### <a name="to-create-a-virtual-network"></a>Creación de una red virtual
[!INCLUDE [vpn-gateway-basic-vnet-rm-portal](../../includes/vpn-gateway-basic-vnet-rm-portal-include.md)]

## <a name="create-the-vnet1-gateway"></a>Creación de la puerta de enlace de VNet1
En este paso, se crea la puerta de enlace para la red virtual. La creación de una puerta de enlace suele tardar 45 minutos o más, según la SKU de la puerta de enlace seleccionada. Si va a crear esta configuración como ejercicio, consulte la [configuración de ejemplo](#example-settings).

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-portal-include.md)]

### <a name="to-create-a-virtual-network-gateway"></a>Creación de una puerta de enlace de red virtual
[!INCLUDE [vpn-gateway-add-gw-rm-portal](../../includes/vpn-gateway-add-gw-rm-portal-include.md)]

[!INCLUDE [vpn-gateway-no-nsg](../../includes/vpn-gateway-no-nsg-include.md)]

## <a name="create-and-configure-vnet4"></a>Creación y configuración de VNet4
Después de configurar VNet1, cree VNet4 y la puerta de enlace de VNet4 repitiendo los pasos anteriores, pero reemplazando los valores por los de VNet4. No es preciso esperar a que la puerta de enlace de red virtual de VNet1 haya terminado de crearse para configurar VNet4. Si usa sus propios valores, asegúrese de que los espacios de direcciones no se superponen con las redes virtuales a las que quiere conectarse.

## <a name="configure-the-vnet1-gateway-connection"></a>Configuración de la conexión de puerta de enlace de VNet1
Cuando se hayan completado las puertas de enlace de red virtual de VNet1 y VNet4, puede crear las conexiones de dichas puertas de enlace. En esta sección, se crea una conexión de VNet1 a VNet4. Estos pasos solo funcionan para las redes virtuales en la misma suscripción. Si las redes virtuales se encuentran en distintas suscripciones, debe usar [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md) para realizar la conexión. Sin embargo, si las redes virtuales se encuentran en distintos grupos de recursos de la misma suscripción, puede conectarlas mediante el portal.

1. En Azure Portal, seleccione **Todos los recursos**, escriba *puerta de enlace de red virtual* en el cuadro de búsqueda y vaya a la puerta de enlace de su red virtual. Por ejemplo, **VNet1GW**. Seleccione la puerta de enlace para abrir la página **Puerta de enlace de red virtual**. En **Configuración**, seleccione **Conexiones**.

   ![Página Conexiones](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/connections.png "Página Conexiones")
2. Seleccione **+ Agregar** para abrir la página **Agregar conexión**.

   ![Agregar conexión](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/vnet1-vnet4-connection.png "Agregar una conexión")
3. En la página **Agregar conexión**, rellene los valores de la conexión:

   - **Name**: Escriba un nombre para la conexión. Por ejemplo, *VNet1toVNet4*.

   - **Tipo de conexión**: Seleccione **De red virtual a red virtual** en la lista desplegable.

   - **Primera puerta de enlace de red virtual**: Este valor de campo se rellena automáticamente porque esta conexión se crea desde la puerta de enlace de red virtual especificada.

   - **Segunda puerta de enlace de red virtual**: Este campo es la puerta de enlace de la red virtual con la que quiere crear una conexión. Seleccione **Elegir otra puerta de enlace de red virtual** para abrir la página **Elegir puerta de enlace de red virtual**.

     - Fíjese en las puertas de enlace de red virtual que se enumeran en esta página. Observe que solo se muestran las que se encuentran en su suscripción. Si quiere conectarse a una puerta de enlace de red virtual que no está en su suscripción, use [PowerShell](vpn-gateway-vnet-vnet-rm-ps.md).

     - Seleccione la puerta de enlace de red virtual con la que quiere conectarse.

     - **Clave compartida (PSK)** : En este campo, escriba una clave compartida para la conexión. Dicha clave puede generarla o crearla manualmente. En una conexión de sitio a sitio, la clave que usa es la misma del dispositivo local y de la conexión de puerta de enlace de red virtual. Aquí el concepto es similar, salvo en que en lugar de conectarse a un dispositivo VPN, la conexión se establece con otra puerta de enlace de red virtual.
    
4. Seleccione **Aceptar** para guardar los cambios.

## <a name="configure-the-vnet4-gateway-connection"></a>Configuración de la conexión de puerta de enlace de VNet4
A continuación, cree una conexión de VNet4 a VNet1. En el portal, busque la puerta de enlace de red virtual asociada a VNet4. Siga los pasos de la sección anterior, reemplazando los valores para crear una conexión de VNet4 a VNet1. Asegúrese de que utiliza la misma clave compartida.

## <a name="verify-your-connections"></a>Comprobación de las conexiones

1. Busque la puerta de enlace de red virtual en Azure Portal. 
2. En la página **Puerta de enlace de red virtual**, seleccione **Conexiones** para ver la página **Conexiones** de la puerta de enlace de red virtual. Una vez que se establece la conexión, verá que los valores de **Estado** cambian a **Conectado**.

   ![Comprobación de conexiones](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/view-connections.png "Comprobación de conexiones")
3. En la columna **Nombre**, seleccione una de las conexiones para ver más información. Cuando se inicia el flujo de datos, aparecerán los valores para **Datos de entrada** y **Datos de salida**.

   ![Estado](./media/vpn-gateway-howto-vnet-vnet-resource-manager-portal/status.png "Status")

## <a name="add-additional-connections"></a>Incorporación de conexiones adicionales

Si quiere agregar más conexiones, vaya a la puerta de enlace de red virtual desde la que quiere crear la conexión y, luego, seleccione **Conexiones**. Puede crear otra conexión entre redes virtual o bien crear una conexión de IPsec de sitio a sitio en una ubicación local. Asegúrese de ajustar el **Tipo de conexión** para coincidir con el tipo de conexión que desea crear. Antes de crear más conexiones, compruebe que el espacio de direcciones de la red virtual no se superponga con ninguno de los espacios de direcciones con los que quiere conectarse. Para saber qué pasos son necesarios para crear una conexión de sitio a sitio, consulte [Creación de una conexión de sitio a sitio](vpn-gateway-howto-site-to-site-resource-manager-portal.md).

## <a name="vnet-to-vnet-faq"></a>P+F sobre conexiones de red virtual a red virtual
Vea los detalles de preguntas más frecuentes para más información acerca de las conexiones de red virtual a red virtual.

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Para información sobre cómo limitar el tráfico de red a los recursos de una red virtual, consulte [Seguridad de red](../virtual-network/security-overview.md).

Para información acerca de cómo enruta Azure el tráfico entre los recursos locales, de Internet y de Azure, vea [Enrutamiento del tráfico de redes virtuales](../virtual-network/virtual-networks-udr-overview.md).
