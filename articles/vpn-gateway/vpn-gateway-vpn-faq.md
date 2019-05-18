---
title: Preguntas más frecuentes acerca de Azure VPN Gateway | Microsoft Docs
description: Preguntas más frecuentes sobre VPN Gateway Preguntas más frecuentes sobre las conexiones entre locales, las conexiones de configuración híbrida y VPN Gateway.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 05/14/2019
ms.author: yushwang
ms.openlocfilehash: 23dc017b6ffcca8761966a10bd5cb45b32c7a602
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65786694"
---
# <a name="vpn-gateway-faq"></a>Preguntas más frecuentes sobre VPN Gateway

## <a name="connecting"></a>Conexión a redes virtuales

### <a name="can-i-connect-virtual-networks-in-different-azure-regions"></a>¿Puedo conectar redes virtuales en diferentes regiones de Azure?

Sí. De hecho, no hay ninguna restricción de región. Puede conectar una red virtual a otra red virtual en la misma región o en una región distinta de Azure. 

### <a name="can-i-connect-virtual-networks-in-different-subscriptions"></a>¿Puedo conectar redes virtuales en diferentes suscripciones?

Sí.

### <a name="can-i-connect-to-multiple-sites-from-a-single-virtual-network"></a>¿Puedo conectar a varios sitios desde una única red virtual?

Puede conectarse a varios sitios mediante el uso de Windows PowerShell y las API de REST de Azure. Consulte la sección de P+F [Conectividad de red virtual a red virtual y de multisitio](#V2VMulti) .

### <a name="is-there-an-additional-cost-for-setting-up-a-vpn-gateway-as-active-active"></a>¿Hay un costo adicional para configurar una puerta de enlace VPN como activa-activa?

No. 

### <a name="what-are-my-cross-premises-connection-options"></a>¿Cuáles son mis opciones de conexión entre locales?

Se admiten las siguientes conexiones entre locales:

* Sitio a sitio: conexión VPN sobre IPsec (IKE v1 e IKE v2). Este tipo de conexión requiere un dispositivo VPN o RRAS. Para más información, consulte [Sitio a sitio](vpn-gateway-howto-site-to-site-resource-manager-portal.md).
* De punto a sitio: conexión VPN sobre SSTP (Protocolo de túnel de sockets seguros) o IKE v2. Esta conexión no requiere un dispositivo VPN. Para más información, consulte [Punto a sitio](vpn-gateway-howto-point-to-site-resource-manager-portal.md).
* Red virtual a red virtual: este tipo de conexión es el mismo que el de la configuración de sitio a sitio. La conexión de red virtual a red virtual es una conexión VPN sobre IPsec (IKE v1 e IKE v2). No requiere un dispositivo VPN. Para más información, consulte [Red virtual a red virtual](vpn-gateway-howto-vnet-vnet-resource-manager-portal.md).
* Varios sitios: esta es una variación de una configuración de sitio a sitio que le permite conectar varios sitios locales a una red virtual. Para más información, consulte [Varios sitios](vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md).
* ExpressRoute: ExpressRoute es una conexión privada a Azure desde la WAN, no una conexión VPN a través de la red pública de Internet. Para más información, consulte [Información técnica de ExpressRoute](../expressroute/expressroute-introduction.md) y [P+F de ExpressRoute](../expressroute/expressroute-faqs.md).

Para más información acerca de las conexiones de puerta de enlace de VPN, consulte [Acerca de VPN Gateway](vpn-gateway-about-vpngateways.md).

### <a name="what-is-the-difference-between-a-site-to-site-connection-and-point-to-site"></a>¿Cuál es la diferencia entre una conexión de sitio a sitio y una de punto a sitio?

Las configuraciones **de sitio a sitio** (túnel de VPN sobre IPsec/IKE) se encuentran entre la ubicación local y Azure. Esto significa que puede conectar cualquiera de las máquinas de sus instalaciones locales con cualquier máquina virtual o instancia de rol de la red virtual, en función de la configuración del enrutamiento y de los permisos. Es ideal para una conexión entre locales que esté siempre disponible y una opción que se adapta bien a las configuraciones híbridas. Este tipo de conexión se basa en un dispositivo (de hardware o de software) VPN sobre IPsec, que se tiene que implementar en el perímetro de la red. Para crear este tipo de conexión, debe tener una dirección IPv4 externa.

Las configuraciones **de punto a sitio** (VPN sobre SSTP) le permiten conectarse desde una única máquina y desde cualquier lugar con cualquier dispositivo de la red virtual. Usa el cliente VPN incluido en Windows. Como parte de la configuración de punto a sitio, instala un certificado y un paquete de configuración de cliente VPN, que contiene la configuración que permite al equipo conectarse a cualquier máquina virtual o instancia de rol dentro de la red virtual. Es ideal si desea conectarse a una red virtual pero no se encuentra en una ubicación local. También es una buena opción cuando no tenga acceso a un hardware VPN o una dirección IPv4 con orientación externa, ya que ambos son necesarios para una conexión de sitio a sitio.

Puede configurar la red virtual para utilizar las conexiones de sitio a sitio y de punto a sitio simultáneamente, siempre que cree la conexión de sitio a sitio mediante un tipo de VPN basada en enrutamiento para la puerta de enlace. Los tipos de VPN basada en enrutamiento se denominan puertas de enlace dinámicas en el modelo de implementación clásico.

## <a name="gateways"></a>Puertas de enlace de red virtual

### <a name="is-a-vpn-gateway-a-virtual-network-gateway"></a>¿Es la puerta de enlace de VPN una puerta de enlace de red virtual?

Una puerta de enlace de VPN es un tipo de puerta de enlace de red virtual. Una puerta de enlace de VPN envía tráfico cifrado entre la red virtual y la ubicación local a través de una conexión pública. También puede utilizar una puerta de enlace de VPN para enviar tráfico entre redes virtuales. Cuando se crea una puerta de enlace de VPN, se usa el valor de -GatewayType 'Vpn'. Para más información, consulte [Acerca de la configuración de VPN Gateway](vpn-gateway-about-vpn-gateway-settings.md).

### <a name="what-is-a-policy-based-static-routing-gateway"></a>¿Qué es una puerta de enlace basada en directivas (de enrutamiento estático)?

Las puertas de enlace basadas en directivas implementan VPN basadas en directivas. Las VPN basadas en directivas cifran y dirigen los paquetes a través de túneles de IPsec basados en las combinaciones de prefijos de dirección entre su red local y la red virtual de Azure. Normalmente, la directiva (o selector de tráfico) se define como una lista de acceso en la configuración de la VPN.

### <a name="what-is-a-route-based-dynamic-routing-gateway"></a>¿Qué es una puerta de enlace basada en enrutamiento (de enrutamiento dinámico)?

Las puertas de enlace basadas en enrutamiento implementan VPN basadas en enrutamiento. Las VPN basadas en enrutamiento utilizan "rutas" en la dirección IP de reenvío o en la tabla de enrutamiento para dirigir los paquetes a sus correspondientes interfaces de túnel. A continuación, las interfaces de túnel cifran o descifran los paquetes dentro y fuera de los túneles. La directiva o el selector de tráfico para las VPN basadas en enrutamiento se configura como conectividad de tipo cualquiera a cualquier (o caracteres comodín).

### <a name="can-i-update-my-policy-based-vpn-gateway-to-route-based"></a>¿Puedo actualizar mi puerta de enlace VPN basada en directivas a una basada en el enrutamiento?
No. No se puede cambiar un tipo de puerta de enlace de red virtual de Azure basada en directivas a una basada en el enrutamiento o viceversa. Es necesario eliminar la puerta de enlace y volver a crearla, un proceso tarda aproximadamente 60 minutos. La dirección IP de la puerta de enlace no se conserva, ni tampoco la clave precompartida (PSK).
1. Elimine también las conexiones asociadas a la puerta de enlace que se va a eliminar.
1. Elimine la puerta de enlace:
1. [Azure Portal](vpn-gateway-delete-vnet-gateway-portal.md)
1. [Azure PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
1. [Azure PowerShell: clásico](vpn-gateway-delete-vnet-gateway-classic-powershell.md)
1. [Creación de una puerta de enlace del tipo deseado y configuración de VPN completa](vpn-gateway-howto-site-to-site-resource-manager-portal.md#VNetGateway)

### <a name="do-i-need-a-gatewaysubnet"></a>¿Necesito una "GatewaySubnet"?

Sí. La subred de puerta de enlace contiene las direcciones IP que usan los servicios de puerta de enlace de la red virtual. Necesitará crear una subred de puerta de enlace de red virtual para configurar la puerta de enlace de la red virtual. Para que funcionen correctamente, todas las subredes de puerta de enlace se deben llamar "GatewaySubnet". No denomine de ninguna otra forma la subred de puerta de enlace. Y no implemente máquinas virtuales ni nada más en la subred de puerta de enlace.

Al crear la subred de puerta de enlace, especifique el número de direcciones IP que contiene la subred. Las direcciones IP de la subred de puerta de enlace se asignan al servicio de puerta de enlace. Algunas configuraciones requieren la asignación de más direcciones IP a los servicios de puerta de enlace que otras. Debe asegurarse de que la subred de puerta de enlace contiene suficientes direcciones IP para soportar el crecimiento futuro y posibles configuraciones de conexión nuevas. Por lo tanto, aunque es posible crear una puerta de enlace tan pequeña como /29, se recomienda que sea de /27 o mayor (/27, /26, /25, etc.). Consulte los requisitos de configuración que desea crear y compruebe que su subred de puerta de enlace los cumple.

### <a name="can-i-deploy-virtual-machines-or-role-instances-to-my-gateway-subnet"></a>¿Puedo implementar máquinas virtuales o instancias de rol en mi subred de puerta de enlace?

No.

### <a name="can-i-get-my-vpn-gateway-ip-address-before-i-create-it"></a>¿Puedo obtener mi dirección IP de puerta de enlace VPN antes de crearla?

No. Tiene que crear primero la puerta de enlace para obtener la dirección IP. Si elimina y vuelve a crear la puerta de enlace VPN, la dirección IP cambiará.

### <a name="can-i-request-a-static-public-ip-address-for-my-vpn-gateway"></a>¿Se puede solicitar una dirección IP pública estática para una puerta de enlace de VPN?

No. Solo se admite la asignación de direcciones IP dinámicas. Sin embargo, esto no significa que la dirección IP cambia después de que se ha asignado a una puerta de enlace VPN. La única vez que cambia la dirección IP de la puerta de enlace de VPN es cuando se elimina y se vuelve a crear la puerta de enlace. La dirección IP pública de la puerta de enlace de VPN no cambia aunque se cambie el tamaño, se restablezca o se lleven a cabo actualizaciones o u operaciones de mantenimiento interno de la puerta de enlace de VPN. 

### <a name="how-does-my-vpn-tunnel-get-authenticated"></a>¿Cómo se autentica mi túnel VPN?

VPN de Azure usa autenticación PSK (clave previamente compartida). Se genera una clave previamente compartida (PSK) cuando se crea el túnel VPN. Puede cambiar la clave PSK generada automáticamente con la suya propia a través del cmdlet de PowerShell o la API de REST para establecer la clave precompartida.

### <a name="can-i-use-the-set-pre-shared-key-api-to-configure-my-policy-based-static-routing-gateway-vpn"></a>¿Puedo usar la API para establecer la clave precompartida y configurar mi VPN basada en directivas (enrutamiento estático) de la puerta de enlace?

Sí, el cmdlet de PowerShell y la API para establecer la clave precompartida se pueden utilizar para configurar tanto las VPN basadas en directivas (enrutamiento estático) de Azure como las VPN basadas en enrutamiento (enrutamiento dinámico).

### <a name="can-i-use-other-authentication-options"></a>¿Puedo usar otras opciones de autenticación?

Las opciones están limitadas al uso de claves precompartidas (PSK) para la autenticación.

### <a name="how-do-i-specify-which-traffic-goes-through-the-vpn-gateway"></a>¿Cómo especifico qué tráfico pasa a través de la puerta de enlace VPN?

#### <a name="resource-manager-deployment-model"></a>Modelo de implementación del Administrador de recursos

* PowerShell: use "AddressPrefix" para especificar el tráfico de la puerta de enlace de red local.
* Azure Portal: navegue a la puerta de enlace de red local > Configuración > Espacio de direcciones.

#### <a name="classic-deployment-model"></a>Modelo de implementación clásica

* Azure Portal: navegue a la red virtual clásica > Conexiones VPN > Conexiones VPN de sitio a sitio > Nombre del sitio local > Sitio local > Espacio de direcciones del cliente. 

### <a name="can-i-configure-force-tunneling"></a>¿Puedo configurar una tunelización forzada?

Sí. Consulte [Configuración de una tunelización forzada](vpn-gateway-about-forced-tunneling.md).

### <a name="can-i-set-up-my-own-vpn-server-in-azure-and-use-it-to-connect-to-my-on-premises-network"></a>¿Puedo configurar mi propio servidor VPN en Azure y usarlo para conectar a mi red local?

Sí, puede implementar sus propias puertas de enlace o servidores VPN en Azure bien desde Azure Marketplace o creando sus propios enrutadores VPN. Necesita configurar las rutas definidas por el usuario en la red virtual para asegurarse de que el tráfico se enruta correctamente entre las redes locales y las subredes de la red virtual.

### <a name="gatewayports"></a>¿Por qué hay algunos puertos abiertos en mi puerta de enlace de red virtual?

Son necesarios para la comunicación de la infraestructura de Azure. Están protegidos (bloqueados) mediante certificados de Azure. Sin los certificados apropiados, las entidades externas, incluidos los clientes de esas puertas de enlace, no podrán tener ningún efecto en esos puntos de conexión.

Una puerta de enlace de red virtual es básicamente un dispositivo de hosts múltiples con una NIC que accede a la red privada del cliente y una NIC accesible desde la red pública. Las entidades de la infraestructura de Azure no pueden acceder a redes privadas de clientes por motivos de conformidad, por lo que necesitan usar puntos de conexión públicos para la comunicación de infraestructura. Los puntos de conexión públicos se analizan periódicamente mediante auditoría de seguridad de Azure.

### <a name="more-information-about-gateway-types-requirements-and-throughput"></a>Más información acerca de los tipos de puerta de enlace, los requisitos y el rendimiento

Para más información, consulte [Acerca de la configuración de VPN Gateway](vpn-gateway-about-vpn-gateway-settings.md).

## <a name="s2s"></a>Conexiones de sitio a sitio y dispositivos VPN

### <a name="what-should-i-consider-when-selecting-a-vpn-device"></a>¿Qué tengo que tener en cuenta al seleccionar un dispositivo VPN?

Hemos validado un conjunto de dispositivos estándar VPN de sitio a sitio en colaboración con proveedores de dispositivos. El artículo [Acerca de los dispositivos VPN](vpn-gateway-about-vpn-devices.md) contiene una lista de dispositivos VPN de compatibilidad conocida y sus correspondientes instrucciones de configuración o ejemplos, así como especificaciones del dispositivo. Todos los dispositivos dentro de las familias de dispositivos que aparecen como de compatibilidad conocida, deben funcionar con Virtual Network. Con el fin de configurar el dispositivo VPN, consulte el ejemplo de configuración de dispositivo o el vínculo que corresponde a la familia de dispositivos adecuada.

### <a name="where-can-i-find-vpn-device-configuration-settings"></a>¿Dónde puedo encontrar los valores de configuración de los dispositivos VPN?

[!INCLUDE [vpn devices](../../includes/vpn-gateway-configure-vpn-device-rm-include.md)]

### <a name="how-do-i-edit-vpn-device-configuration-samples"></a>Edición de ejemplos de configuración de dispositivos VPN

Para obtener información sobre cómo modificar los ejemplos de configuración de dispositivo, consulte [Edición de ejemplos](vpn-gateway-about-vpn-devices.md#editing).

### <a name="where-do-i-find-ipsec-and-ike-parameters"></a>¿Dónde encuentro los parámetros de IPsec e IKE?

Para los parámetros de IPsec/IKE, vea [Parámetros](vpn-gateway-about-vpn-devices.md#ipsec).

### <a name="why-does-my-policy-based-vpn-tunnel-go-down-when-traffic-is-idle"></a>¿Por qué mi túnel de VPN basado en directivas deja de funcionar cuando el tráfico está inactivo?

Este es el comportamiento esperado para puertas de enlace de VPN basadas en directivas (también conocido como enrutamiento estático). Cuando el tráfico a través del túnel está inactivo durante más de 5 minutos, este túnel se cancelará. Cuanto el tráfico comience a fluir en cualquier dirección, el túnel se restablecerá inmediatamente.

### <a name="can-i-use-software-vpns-to-connect-to-azure"></a>¿Puedo usar una VPN de software para conectarme a Azure?

Para la configuración de sitio a sitio entre locales se admiten los servidores de Windows Server 2012 con servicio de enrutamiento y acceso remoto (RRAS).

Otras soluciones VPN de software deben funcionar con nuestra puerta de enlace siempre que se ajusten a las implementaciones IPsec estándar de la industria. Póngase en contacto con el proveedor del software para obtener instrucciones de configuración y soporte técnico.

## <a name="P2S"></a>Punto a sitio mediante la autenticación de certificados de Azure nativa

Esta sección se aplica al modelo de implementación de Resource Manager.

[!INCLUDE [P2S Azure cert](../../includes/vpn-gateway-faq-p2s-azurecert-include.md)]

## <a name="P2SRADIUS"></a>Punto a sitio mediante la autenticación RADIUS

Esta sección se aplica al modelo de implementación de Resource Manager.

[!INCLUDE [vpn-gateway-point-to-site-faq-include](../../includes/vpn-gateway-faq-p2s-radius-include.md)]

## <a name="V2VMulti"></a>Conexiones entre dos redes virtuales y multisitio

[!INCLUDE [vpn-gateway-vnet-vnet-faq-include](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]

### <a name="can-i-use-azure-vpn-gateway-to-transit-traffic-between-my-on-premises-sites-or-to-another-virtual-network"></a>¿Puedo usar la puerta de enlace de VPN de Azure para el tráfico en tránsito entre mis sitios locales o a otra red virtual?

**Modelo de implementación de Resource Manager**<br>
Sí. Para más información, consulte la sección [BGP](#bgp).

**Modelo de implementación clásica**<br>
 el tráfico en tránsito a través de Puerta de enlace de VPN de Azure es posible mediante el modelo de implementación clásica, pero se basa en espacios de direcciones definidos estáticamente en el archivo de configuración de red. BGP aún no se admite con instancias de Red virtual de Azure y Puerta de enlace de VPN mediante el modelo de implementación clásica. Sin BGP, definir manualmente los espacios de direcciones de tránsito es difícil de hacer sin errores y no se recomienda.

### <a name="does-azure-generate-the-same-ipsecike-pre-shared-key-for-all-my-vpn-connections-for-the-same-virtual-network"></a>¿Azure genera la misma clave precompartida de IPsec/IKE para todas mis conexiones VPN para la misma red virtual?

No, Azure de forma predeterminada genera distintas claves precompartidas para distintas conexiones VPN. Sin embargo, puede utilizar la API de REST para establecer la clave de VPN Gateway o el cmdlet PowerShell para establecer el valor de clave que prefiera. La clave debe ser caracteres ASCII imprimibles.

### <a name="do-i-get-more-bandwidth-with-more-site-to-site-vpns-than-for-a-single-virtual-network"></a>¿Tengo más ancho de banda con más VPN de sitio a sitio que si tengo una única red virtual?

No, todos los túneles VPN, incluidas las VPN de punto a sitio, comparten la misma puerta de enlace de VPN de Azure y el ancho de banda disponible.

### <a name="can-i-configure-multiple-tunnels-between-my-virtual-network-and-my-on-premises-site-using-multi-site-vpn"></a>¿Puedo configurar varios túneles entre mi red virtual y mi sitio local utilizando VPN multisitio?

Sí, pero debe configurar BGP en ambos túneles de la misma ubicación.

### <a name="can-i-use-point-to-site-vpns-with-my-virtual-network-with-multiple-vpn-tunnels"></a>¿Puedo usar VPN de punto a sitio con mi red virtual con varios túneles VPN?

Sí, las VPN de punto a sitio (P2S) se pueden usar con las puertas de enlace de VPN para conectarse a varios sitios locales y a otras redes virtuales.

### <a name="can-i-connect-a-virtual-network-with-ipsec-vpns-to-my-expressroute-circuit"></a>¿Puedo conectar una red virtual con VPN sobre IPsec a mi circuito de ExpressRoute?

Sí, este procedimiento se admite. Para más información, consulte [Configurar conexiones VPN ExpressRoute y sitio a sitio que coexistan](../expressroute/expressroute-howto-coexist-classic.md).

## <a name="ipsecike"></a>Directiva de IPsec o IKE

[!INCLUDE [vpn-gateway-ipsecikepolicy-faq-include](../../includes/vpn-gateway-faq-ipsecikepolicy-include.md)]


## <a name="bgp"></a>BGP

[!INCLUDE [vpn-gateway-faq-bgp-include](../../includes/vpn-gateway-faq-bgp-include.md)]

## <a name="vms"></a>Conectividad entre locales y máquinas virtuales

### <a name="if-my-virtual-machine-is-in-a-virtual-network-and-i-have-a-cross-premises-connection-how-should-i-connect-to-the-vm"></a>Si mi máquina virtual está en una red virtual y tengo una conexión entre locales, ¿cómo debo conectar a la máquina virtual?

Dispone de varias opciones. Si tiene el RDP habilitado en la máquina virtual, puede conectarse a ella mediante la dirección IP privada. En ese caso, debe especificar la dirección IP privada y el puerto al que desea conectarse (normalmente el 3389). Tendrá que configurar el puerto en la máquina virtual para el tráfico.

También puede conectarse a la máquina virtual mediante la dirección IP privada desde otra máquina virtual que se encuentre en la misma red virtual. Si se conecta desde una ubicación que esté fuera de la red virtual, no podrá usar RDP en la máquina virtual mediante la dirección IP privada. Por ejemplo, si tiene configurada una red virtual de punto a sitio y no establece una conexión desde su equipo, no podrá conectarse a la máquina virtual mediante la dirección IP privada.

### <a name="if-my-virtual-machine-is-in-a-virtual-network-with-cross-premises-connectivity-does-all-the-traffic-from-my-vm-go-through-that-connection"></a>¿Si mi máquina virtual está en una red virtual con conectividad entre locales, todo el tráfico de mi máquina virtual pasa a través de esa conexión?

No. Únicamente el tráfico que tiene como destino una IP que se encuentra en los intervalos de direcciones IP de red local de la red virtual que haya especificado pasará a través de la puerta de enlace de red virtual. El tráfico que tenga una IP de destino ubicada dentro de la red virtual permanecerá en la red virtual. El resto del tráfico se envía a través del equilibrador de carga a las redes públicas, o si se usa la tunelización forzada, se envía a través de la puerta de enlace VPN de Azure.

### <a name="how-do-i-troubleshoot-an-rdp-connection-to-a-vm"></a>Cómo se solucionan los problemas de una conexión RDP a una máquina virtual

[!INCLUDE [Troubleshoot VM connection](../../includes/vpn-gateway-connect-vm-troubleshoot-include.md)]


## <a name="faq"></a>Preguntas más frecuentes sobre Virtual Network

Consulte información adicional de redes virtuales adicionales en las [Preguntas frecuentes sobre redes virtuales](../virtual-network/virtual-networks-faq.md).

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre VPN Gateway, consulte [Acerca de VPN Gateway](vpn-gateway-about-vpngateways.md).
* Para más información acerca de la configuración de VPN Gateway, consulte [Acerca de la configuración de VPN Gateway](vpn-gateway-about-vpn-gateway-settings.md).

**"OpenVPN" es una marca de OpenVPN Inc.**
