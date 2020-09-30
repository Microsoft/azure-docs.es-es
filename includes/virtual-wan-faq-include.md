---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 09/02/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 2519baa01fa9d8a13dd2e7855f9da3ec7f9093f9
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89570146"
---
### <a name="does-the-user-need-to-have-hub-and-spoke-with-sd-wanvpn-devices-to-use-azure-virtual-wan"></a>¿El usuario necesita disponer de una topología radial con los dispositivos SD-WAN/VPN para usar Azure Virtual WAN?

Virtual WAN proporciona muchas funcionalidades integradas en un único panel, como la conectividad VPN de sitio a sitio, la conectividad de usuario y P2S, la conectividad de ExpressRoute, la conectividad de Virtual Network, la interconectividad de ExpressRoute de VPN, la conectividad transitiva de red virtual a red virtual, el enrutamiento centralizado, Azure Firewall y la seguridad de Firewall Manager, la supervisión, el cifrado de ExpressRoute y muchas otras funcionalidades. No es necesario disponer de todos estos casos de uso para empezar a usar Virtual WAN. Puede empezar a trabajar con uno solo de estos. La arquitectura de Virtual WAN es una arquitectura de concentrador y radio con escalado y rendimiento integrados, donde las ramas (dispositivos VPN/SD-WAN), los usuarios (clientes de VPN de Azure, OpenVPN o IKEv2), los circuitos ExpressRoute y las instancias de Virtual Network sirven como radios para los centros de conectividad virtuales. Todos los centros de conectividad están conectados en una malla completa en una red WAN virtual estándar, lo que facilita al usuario el uso de la red troncal de Microsoft para la conectividad de cualquier tipo (cualquier radio). En el caso de la topología radial de los dispositivos SD-WAN/VPN, los usuarios pueden configurarla manualmente en el portal de Azure Virtual WAN o usar el CPE del asociado de Virtual WAN (SD-WAN/VPN) para configurar la conectividad con Azure. Los asociados de Virtual WAN proporcionan automatización de la conectividad, que es la capacidad de exportar la información del dispositivo a Azure, descargar la configuración de Azure y establecer la conectividad con el centro de conectividad de Azure Virtual WAN. Para la conectividad de punto a sitio/VPN de usuario, se admiten el [cliente de VPN de Azure](https://go.microsoft.com/fwlink/?linkid=2117554), OpenVPN o IKEv2. 

### <a name="can-you-disable-fully-meshed-hubs-in-a-virtual-wan"></a>¿Se pueden deshabilitar los centros de conectividad totalmente en malla en una instancia de Virtual WAN?

Virtual WAN se ofrece en dos variedades: Básico y Estándar. En la instancia de Virtual WAN básica, los centros de conectividad no están en malla. En una instancia de Virtual WAN estándar, los centros de conectividad están en malla y se conectan automáticamente cuando la instancia se configura por primera vez. No es necesario que el usuario haga nada específico. El usuario tampoco tiene que deshabilitar ni habilitar la funcionalidad para obtener concentradores en malla. Virtual WAN ofrece muchas opciones de enrutamiento para dirigir el tráfico entre radios cualesquiera (VNet, VPN o ExpressRoute). Proporciona la facilidad de los centros de conectividad totalmente en malla, así como la flexibilidad de enrutar el tráfico según sus necesidades. 

### <a name="how-are-availability-zones-and-resiliency-handled-in-virtual-wan"></a>¿Cómo se administran Availability Zones y la resistencia en Virtual WAN?

Virtual WAN es una colección de centros de conectividad y servicios que están disponibles en el centro de conectividad. El usuario puede tener tantas instancias de Virtual WAN como necesite. En un centro de conectividad de Virtual WAN existen varios servicios, como VPN, ExpressRoute, etc. Cada uno de estos servicios (excepto Azure Firewall) se implementa en una región de Availability Zones, siempre y cuando esta admita Availability Zones. Si una región se convierte en una zona de disponibilidad después de la implementación inicial en el centro de conectividad, el usuario puede volver a crear las puertas de enlace, lo que desencadenará una implementación de Availability Zones. Todas las puertas de enlace se aprovisionan en un centro de conectividad como activo-activo, lo que implica que hay resistencia integrada dentro de un centro de conectividad. Los usuarios pueden conectarse a varios centros de conectividad si desean resistencia entre regiones. Aunque el concepto de Virtual WAN es global, el recurso de Virtual WAN real se basa en Resource Manager y se implementa de forma regional. En caso de que la propia región de Virtual WAN tenga un problema, todos los centros de conectividad de esa instancia de Virtual WAN seguirán funcionando tal cual, pero el usuario no podrá crear nuevos centros de conectividad hasta que la región de Virtual WAN esté disponible.

### <a name="what-client-does-the-azure-virtual-wan-user-vpn-point-to-site-support"></a>¿Qué cliente admite la red privada virtual de usuario (de punto a sitio) de Azure Virtual WAN?

Virtual WAN admite el [cliente de VPN de Azure](https://go.microsoft.com/fwlink/?linkid=2117554), el cliente de OpenVPN o cualquier cliente de IKEv2. La autenticación de Azure AD se admite con el cliente de VPN de Azure. Como mínimo, se requiere la versión 17763.0 o superior del sistema operativo de cliente de Windows 10.  Los clientes de OpenVPN pueden admitir la autenticación basada en certificados. Una vez que se haya seleccionado la autenticación basada en certificados en la puerta de enlace, verá el archivo *.ovpn* que se va a descargar en el dispositivo. IKEv2 admite la autenticación basada en certificados y RADIUS. 

### <a name="for-user-vpn-point-to-site--why-is-the-p2s-client-pool-split-into-two-routes"></a>En el caso de la red privada virtual de usuario (de punto a sitio), ¿por qué el grupo de clientes de P2S se divide en dos rutas?

Cada puerta de enlace tiene dos instancias; la división se produce para que cada instancia de la puerta de enlace pueda asignar de forma independiente las direcciones IP de cliente para los clientes conectados, y el tráfico de la red virtual se redirija a la instancia de puerta de enlace correcta para evitar el salto de instancias entre puertas de enlace.

### <a name="how-do-i-add-dns-servers-for-p2s-clients"></a>¿Cómo puedo agregar servidores DNS para los clientes de P2S?

Hay dos opciones para agregar servidores DNS para los clientes de P2S. Se prefiere el primer método porque agrega los servidores DNS personalizados a la puerta de enlace en lugar del cliente.

1. Use el siguiente script de PowerShell para agregar los servidores DNS personalizados. Reemplace los valores por los de su entorno.

   ```powershell
   // Define variables
   $rgName = "testRG1"
   $virtualHubName = "virtualHub1"
   $P2SvpnGatewayName = "testP2SVpnGateway1"
   $vpnClientAddressSpaces = 
   $vpnServerConfiguration1Name = "vpnServerConfig1"
   $vpnClientAddressSpaces = New-Object string[] 2
   $vpnClientAddressSpaces[0] = "192.168.2.0/24"
   $vpnClientAddressSpaces[1] = "192.168.3.0/24"
   $customDnsServers = New-Object string[] 2
   $customDnsServers[0] = "7.7.7.7"
   $customDnsServers[1] = "8.8.8.8"
   $virtualHub = $virtualHub = Get-AzVirtualHub -ResourceGroupName $rgName -Name $virtualHubName
   $vpnServerConfig1 = Get-AzVpnServerConfiguration -ResourceGroupName $rgName -Name $vpnServerConfiguration1Name

   // Specify custom dns servers for P2SVpnGateway VirtualHub while creating gateway
   createdP2SVpnGateway = New-AzP2sVpnGateway -ResourceGroupName $rgname -Name $P2SvpnGatewayName -VirtualHub $virtualHub -VpnGatewayScaleUnit 1 -VpnClientAddressPool $vpnClientAddressSpaces -VpnServerConfiguration $vpnServerConfig1 -CustomDnsServer $customDnsServers

   // Specify custom dns servers for P2SVpnGateway VirtualHub while updating existing gateway
   $P2SVpnGateway = Get-AzP2sVpnGateway -ResourceGroupName $rgName -Name $P2SvpnGatewayName
   $updatedP2SVpnGateway = Update-AzP2sVpnGateway -ResourceGroupName $rgName -Name $P2SvpnGatewayName  -CustomDnsServer $customDnsServers 

   // Re-generate Vpn profile either from PS/Portal for Vpn clients to have the specified dns servers
   ```
2. O bien, si usa el cliente de VPN de Azure para Windows 10, puede modificar el archivo XML de perfil descargado y agregar las etiquetas **\<dnsservers>\<dnsserver> \</dnsserver>\</dnsservers>** antes de importarlo.

   ```powershell
      <azvpnprofile>
      <clientconfig>

          <dnsservers>
              <dnsserver>x.x.x.x</dnsserver>
              <dnsserver>y.y.y.y</dnsserver>
          </dnsservers>
    
      </clientconfig>
      </azvpnprofile>
   ```

### <a name="for-user-vpn-point-to-site--how-many-clients-are-supported"></a>En el caso de la red privada virtual de usuario (de punto a sitio), ¿cuántos clientes se admiten?

Cada puerta de enlace de red privada virtual de usuario P2S tiene dos instancias, y cada instancia admite hasta un número determinado de usuarios a medida que cambia la unidad de escalado. La unidad de escalado 1-3 admite 500 conexiones, la unidad de escalado 4-6 admite 1000 conexiones, la unidad de escalado 7-12 admite 5000 conexiones y la unidad de escalado 13-18 admite hasta 10 000 conexiones. 

Como ejemplo, supongamos que el usuario elige 1 unidad de escalado. Cada unidad de escalado implica una puerta de enlace activo-activo implementada, y cada una de las instancias (2, en este caso) admitiría hasta 500 conexiones. Dado que puede obtener 500 conexiones x 2 por puerta de enlace, no significa que tenga previstas 1000 en lugar de 500 para esta unidad de escalado. Es posible que se deban atender las instancias en que la conectividad de las 500 conexiones adicionales pueda sufrir interrupciones si supera el recuento de conexiones recomendado. Además, asegúrese de planear el tiempo de inactividad en caso de que decida escalar o reducir verticalmente en la unidad de escalado o cambiar la configuración de punto a sitio en VPN Gateway.

### <a name="what-is-the-difference-between-an-azure-virtual-network-gateway-vpn-gateway-and-an-azure-virtual-wan-vpn-gateway"></a>¿Cuál es la diferencia entre una puerta de enlace de Azure Virtual Network (VPN Gateway) y una instancia de VPN Gateway de Azure Virtual WAN?

Virtual WAN proporciona conectividad de sitio a sitio y a gran escala, y se ha creado para mejorar el rendimiento, la escalabilidad y la facilidad de uso. Cuando conecta un sitio a una instancia de VPN Gateway de Virtual WAN, esto es diferente a una puerta de enlace de red virtual normal que usa un tipo de puerta de enlace "VPN". Del mismo modo, cuando se conecta un circuito ExpressRoute a un centro de conectividad de Virtual WAN, se usa un recurso para la puerta de enlace de ExpressRoute que es distinto al de la puerta de enlace de red virtual normal que usa el tipo de puerta de enlace "ExpressRoute". 

Virtual WAN admite un rendimiento agregado de hasta 20 Gbps para VPN y ExpressRoute. Virtual WAN también tiene automatización para la conectividad con un ecosistema de asociados de dispositivos de la rama CPE. Los dispositivos de la rama CPE tienen una automatización integrada que se aprovisiona automáticamente y se conecta a Azure Virtual WAN. Estos dispositivos están disponibles en un creciente ecosistema de asociados de VPN y SD-WAN. Consulte la [lista de asociados preferidos](../articles/virtual-wan/virtual-wan-locations-partners.md).

### <a name="how-is-virtual-wan-different-from-an-azure-virtual-network-gateway"></a>¿En qué difiere Virtual WAN de una puerta de enlace de Azure Virtual Network?

Una VPN de puerta de enlace de red virtual se limita a 30 túneles. En las conexiones, debe usar Virtual WAN para VPN a gran escala. Puede conectar hasta 1000 conexiones de rama por región (centro de conectividad virtual) con agregados de 20 Gbps por centro. Una conexión es un túnel de activo a activo del dispositivo VPN local al concentrador virtual. Puede tener un único concentrador por región, lo que significa que puede conectar más de 1000 ramas en total con todos los concentradores.

### <a name="what-is-a-virtual-wan-gateway-scale-unit"></a>¿Qué es una unidad de escalado de puerta de enlace de Virtual WAN?

Una unidad de escalado es una unidad definida para obtener un rendimiento agregado de una puerta de enlace de un centro de conectividad virtual. 1 unidad de escalado de VPN = 500 Mbps. 1 unidad de escalado de ExpressRoute = 2 Gbps. Ejemplo: 10 unidades de escalado de VPN significarían 500 Mbps * 10 = 5 Gbps

### <a name="which-device-providers-virtual-wan-partners-are-supported"></a>¿Qué proveedores de dispositivos (asociados de Virtual WAN) se admiten?

En este momento, muchos asociados admiten la experiencia de Virtual WAN totalmente automatizada. Para más información, consulte [Asociados de Virtual WAN](../articles/virtual-wan/virtual-wan-locations-partners.md).

### <a name="what-are-the-virtual-wan-partner-automation-steps"></a>¿Cuáles son los pasos de automatización de asociados de Virtual WAN?

Para conocer estos pasos, consulte [Automatización de asociados de Virtual WAN](../articles/virtual-wan/virtual-wan-configure-automation-providers.md).

### <a name="am-i-required-to-use-a-preferred-partner-device"></a>¿Debo usar un dispositivo de asociado preferido?

No. Puede usar cualquier dispositivo compatible con VPN que cumpla los requisitos de Azure para la compatibilidad con IPsec de IKEv1 o IKEv2. Virtual WAN también tiene soluciones de asociado de CPE que automatizan la conectividad con Azure Virtual WAN, lo que facilita la configuración de conexiones VPN de IPsec a escala.

### <a name="how-do-virtual-wan-partners-automate-connectivity-with-azure-virtual-wan"></a>¿Cómo automatizan la conectividad con Azure Virtual WAN los asociados de Virtual WAN?

Las soluciones de conectividad definidas por software suelen administrar sus dispositivos de rama con un controlador o un centro de aprovisionamiento de dispositivos. El controlador puede usar las API de Azure para automatizar la conectividad a Azure Virtual WAN. La automatización incluye la carga de información de la rama, la descarga de la configuración de Azure, la configuración de túneles IPSec en los centros de conectividad virtuales de Azure y la configuración automática de la conectividad desde el dispositivo de rama a Azure Virtual WAN. Cuando tiene cientos de ramas, la conexión mediante asociados de CPE de Virtual WAN es fácil ya que la experiencia de incorporación elimina la necesidad de configurar y administrar la conectividad de IPsec a gran escala. Para más información, consulte [Automatización de asociados de Virtual WAN](../articles/virtual-wan/virtual-wan-configure-automation-providers.md).

### <a name="what-if-a-device-i-am-using-is-not-in-the-virtual-wan-partner-list-can-i-still-use-it-to-connect-to-azure-virtual-wan-vpn"></a>¿Qué ocurre si el dispositivo que uso no está en la lista de partners de Virtual WAN? ¿Lo puedo usar a pesar de todo para conectarme a una VPN de Azure Virtual WAN?

Sí, siempre que el dispositivo admita IPsec IKEv1 o IKEv2. Los asociados de Virtual WAN automatizan la conectividad desde el dispositivo hasta los puntos de conexión de VPN de Azure. Esto supone automatizar pasos como "carga de información de la rama", "IPsec y configuración" y "conectividad". Dado que el dispositivo no es del ecosistema de un asociado de Virtual WAN, tendrá que realizar el pesado trabajo de configurar manualmente Azure y actualizar el dispositivo para configurar la conectividad de IPsec.

### <a name="how-do-new-partners-that-are-not-listed-in-your-launch-partner-list-get-onboarded"></a>¿Cómo consiguen los nuevos asociados que no aparecen en la lista de asociados de partida ser incluidos en ella?

Todas las API de Virtual WAN son API abiertas. Puede consultar la documentación de [Automatización de asociados de Virtual WAN](../articles/virtual-wan/virtual-wan-configure-automation-providers.md) para valorar la viabilidad técnica. Un asociado ideal es aquel que tiene un dispositivo que se puede aprovisionar para la conectividad IPsec de IKEv1 o IKEv2. Una vez que la compañía haya completado el trabajo de automatización para su dispositivo CPE según las instrucciones de automatización proporcionadas anteriormente, puede ponerse en contacto con azurevirtualwan@microsoft.com para que se muestre en [Conectividad mediante asociados]( ../articles/virtual-wan/virtual-wan-locations-partners.md#partners). Si es un cliente que desea que una determinada solución de la empresa aparezca como un asociado de Virtual WAN, haga que la empresa se ponga en contacto con Virtual WAN mediante el envío de un correo electrónico a azurevirtualwan@microsoft.com.

### <a name="how-is-virtual-wan-supporting-sd-wan-devices"></a>¿Cómo es que Virtual WAN admite dispositivos SD-WAN?

Los asociados de Virtual WAN automatizan la conectividad IPsec a los puntos de conexión de VPN de Azure. Si el asociado de Virtual WAN es un proveedor de SD-WAN, se supone que el controlador de SD-WAN administra la automatización y la conectividad IPsec a los puntos de conexión de VPN de Azure. Si el dispositivo SD-WAN necesita su propio punto de conexión en lugar de la VPN de Azure para obtener funcionalidad de SD-WAN propietaria, puede implementar el punto de conexión de SD-WAN en una red virtual de Azure y coexistir con Azure Virtual WAN.

### <a name="how-many-vpn-devices-can-connect-to-a-single-hub"></a>¿Cuántos dispositivos VPN pueden conectarse a un único centro?

Se admiten hasta 1000 conexiones por concentrador virtual. Cada conexión consta de cuatro vínculos y cada conexión de vínculo admite dos túneles que están en una configuración activo-activo. Los túneles terminan en un elemento VPN Gateway del centro de conectividad virtual de Azure. Los vínculos representan el vínculo del ISP físico en la sucursal o el dispositivo VPN.

### <a name="what-is-a-branch-connection-to-azure-virtual-wan"></a>¿Qué es una conexión de rama a Azure Virtual WAN?

Una conexión desde una rama o un dispositivo VPN a Azure Virtual WAN no es nada más que una conexión VPN que se conecta de forma virtual al sitio VPN y a Azure VPN Gateway en un centro de conectividad virtual.

### <a name="can-the-on-premises-vpn-device-connect-to-multiple-hubs"></a>¿El dispositivo VPN local puede conectarse a varios concentradores?

Sí. Al principio, el flujo de tráfico sería del dispositivo local al perímetro de red de Microsoft más cercano y, luego, al centro de conectividad virtual.

### <a name="are-there-new-resource-manager-resources-available-for-virtual-wan"></a>¿Hay disponibles nuevos recursos de Resource Manager para Virtual WAN?
  
Sí, Virtual WAN presenta nuevos recursos de Resource Manager. Para más información, consulte la [Información general](../articles/virtual-wan/virtual-wan-about.md).

### <a name="can-i-deploy-and-use-my-favorite-network-virtual-appliance-in-an-nva-vnet-with-azure-virtual-wan"></a>¿Puedo implementar y usar mi dispositivo virtual de red favorito (en una red virtual de NVA) con Azure Virtual WAN?

Sí, puede conectar su aplicación virtual de red (NVA) favorita a Azure Virtual WAN.

### <a name="can-i-create-a-network-virtual-appliance-inside-the-virtual-hub"></a>¿Puedo crear una aplicación virtual de red dentro del centro de conectividad virtual?

Una aplicación virtual de red (NVA) no se puede implementar en un centro de conectividad virtual. Sin embargo, puede crearlo en una red virtual radial que esté conectada al centro de conectividad virtual y habilitar el enrutamiento adecuado para dirigir el tráfico según sus necesidades.

### <a name="can-a-spoke-vnet-have-a-virtual-network-gateway"></a>¿Puede una red virtual de radio tener una puerta de enlace de red virtual?

No. La red virtual de radio no puede tener una puerta de enlace de red virtual si está conectada al centro de conectividad virtual.

### <a name="is-there-support-for-bgp-in-vpn-connectivity"></a>¿Existe compatibilidad con BGP en la conectividad VPN?

Sí, BGP se admite. Al crear una VPN de sitio, puede proporcionar en ella los parámetros de BGP. Como resultado, las conexiones creadas en Azure para ese sitio estarán habilitadas para BGP.

### <a name="is-there-any-licensing-or-pricing-information-for-virtual-wan"></a>¿Hay información de precios o licencias para Virtual WAN?

Sí. Consulte la página [Precios](https://azure.microsoft.com/pricing/details/virtual-wan/).

### <a name="is-it-possible-to-construct-azure-virtual-wan-with-a-resource-manager-template"></a>¿Es posible construir Azure Virtual WAN con una plantilla de Resource Manager?

Se puede crear una configuración simple de una instancia de Virtual WAN con un centro y un sitio VPN mediante una [plantilla de inicio rápido](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network). Virtual WAN es principalmente un servicio REST o basado en el portal.

### <a name="can-spoke-vnets-connected-to-a-virtual-hub-communicate-with-each-other-v2v-transit"></a>¿Pueden las redes virtuales radiales conectadas a un centro virtual comunicarse entre sí (tránsito V2V)?

Sí. Virtual WAN estándar admite la conectividad transitiva entre redes virtuales mediante el centro de conectividad de Virtual WAN al que están conectadas las redes virtuales. En la terminología de Virtual WAN, llamamos a estas rutas de acceso "tránsito local de red virtual de Virtual WAN" para las redes virtuales conectadas a un centro de conectividad de Virtual WAN dentro de una única región, y "tránsito global de red virtual de Virtual WAN" en el caso de las redes virtuales conectadas mediante varios centros de conectividad de Virtual WAN en dos o más zonas. En algunos escenarios, las redes virtuales radiales también se pueden emparejar directamente entre sí mediante el [emparejamiento de red virtual](../articles/virtual-network/virtual-network-peering-overview.md), además del tránsito local o global de red virtual de Virtual WAN. En este caso, el emparejamiento de red virtual tiene prioridad sobre la conexión transitiva del centro de conectividad de Virtual WAN.

### <a name="is-branch-to-branch-connectivity-allowed-in-virtual-wan"></a>¿Se permite la conectividad de rama a rama en Virtual WAN?

Sí, este tipo de conectividad está disponible en Virtual WAN. La rama se aplica conceptualmente a sitios VPN, circuitos ExpressRoute o usuarios de VPN de punto a sitio o de usuario. La conectividad de rama a rama está habilitada de forma predeterminada y se puede encontrar en la configuración de WAN. Permite que los usuarios o las ramas de VPN se conecten a otras ramas de VPN, y la conectividad de tránsito está habilitada entre los usuarios de VPN y de ExpressRoute.

### <a name="does-branch-to-branch-traffic-traverse-through-the-azure-virtual-wan"></a>¿Atraviesa el tráfico de rama a rama Azure Virtual WAN?

Sí.

### <a name="does-virtual-wan-require-expressroute-from-each-site"></a>¿La instancia de Virtual WAN requiere ExpressRoute desde cada sitio?

No. Virtual WAN no requiere ExpressRoute desde cada sitio. Los sitios pueden estar conectados a una red de proveedor mediante un circuito ExpressRoute. En el caso de los sitios conectados mediante ExpressRoute a un centro de conectividad virtual, así como a la VPN de IPsec en el mismo centro de conectividad, el centro de conectividad virtual proporciona conectividad de tránsito entre el usuario de VPN y ExpressRoute.

### <a name="is-there-a-network-throughput-or-connection-limit-when-using-azure-virtual-wan"></a>¿Hay un límite en el rendimiento o la conexión de la red al utilizar Azure Virtual WAN?

El rendimiento de red es por servicio en un centro de conectividad de Virtual WAN. Aunque puede tener tantas instancias de Virtual WAN como desee, cada una permite un centro de conectividad por región. En cada centro de conectividad, el rendimiento agregado de VPN es de hasta 20 Gbps, el rendimiento agregado de ExpressRoute es hasta 20 Gbps y el rendimiento agregado de VPN de punto a sitio o VPN de usuario es de hasta 20 Gbps. El enrutador del centro de conectividad virtual admite hasta 50 Gbps en los flujos de tráfico de red virtual a red virtual y supone una carga de trabajo total de 2000 máquinas virtuales en todas las redes virtuales conectadas a un solo centro de conectividad virtual.

Cuando los sitios VPN se conectan a un centro de conectividad, lo hacen con conexiones. Virtual WAN admite hasta 1000 conexiones o 2000 túneles IPsec por centro de conectividad virtual. Cuando los usuarios remotos se conectan a un centro de conectividad virtual, se conectan a la instancia de VPN Gateway P2S, que admite hasta 10 000 usuarios en función de la unidad de escalado (ancho de banda) elegida para la instancia de VPN Gateway P2S en el centro de conectividad virtual.

### <a name="what-is-the-total-vpn-throughput-of-a-vpn-tunnel-and-a-connection"></a>¿Qué es el rendimiento total de VPN de un túnel y una conexión VPN?

El rendimiento total de VPN de un centro de conectividad es de hasta 20 Gbps en función de la unidad de escalado elegida de VPN Gateway. El rendimiento se comparte entre todas las conexiones existentes. Cada túnel de una conexión puede admitir hasta 1 Gbps.

### <a name="i-dont-see-the-20-gbps-setting-for-the-virtual-hub-in-portal-how-do-i-configure-that"></a>No veo la configuración de 20 Gbps para el centro de conectividad virtual en el portal. ¿Cómo puedo configurarlo?

Vaya a instancia de VPN Gateway de un centro del portal y haga clic en la unidad de escalado para cambiarla a la configuración adecuada.

### <a name="does-virtual-wan-allow-the-on-premises-device-to-utilize-multiple-isps-in-parallel-or-is-it-always-a-single-vpn-tunnel"></a>¿Azure Virtual WAN permite al dispositivo local usar varias ISP en paralelo o es siempre un solo túnel VPN?

Las soluciones en dispositivos locales pueden aplicar directivas de tráfico para dirigir el tráfico entre varios túneles en el centro de conectividad de Azure Virtual WAN (VPN Gateway en el centro de conectividad virtual).

### <a name="what-is-global-transit-architecture"></a>¿Qué es la arquitectura de tránsito global?

Para más información acerca de la arquitectura de tránsito global, consulte [Arquitectura de red de tránsito global y Virtual WAN](../articles/virtual-wan/virtual-wan-global-transit-network-architecture.md).

### <a name="how-is-traffic-routed-on-the-azure-backbone"></a>¿Cómo se enruta el tráfico en la red troncal de Azure?

El tráfico sigue el patrón: dispositivo de rama -> ISP -> perímetro de red de Microsoft -> controlador de dominio de Microsoft (red virtual de centro) -> perímetro de red de Microsoft -> ISP -> dispositivo de rama.

### <a name="in-this-model-what-do-you-need-at-each-site-just-an-internet-connection"></a>En este modelo, ¿qué es necesario en cada sitio? ¿Solo una conexión a Internet?

Sí. Una conexión a Internet y un dispositivo físico que admita IPsec, preferiblemente de nuestros [asociados de Virtual WAN](../articles/virtual-wan/virtual-wan-locations-partners.md) integrados. De forma opcional, puede administrar manualmente la configuración y la conectividad a Azure desde su dispositivo preferido.

### <a name="how-do-i-enable-default-route-00000-in-a-connection-vpn-expressroute-or-virtual-network"></a>¿Cómo habilito la ruta predeterminada (0.0.0.0/0) de una conexión (VPN, ExpressRoute o Virtual Network)?

Un centro de conectividad virtual puede propagar una ruta predeterminada aprendida en una red virtual, una VPN de sitio a sitio y una conexión ExpressRoute si la marca está "habilitada" en la conexión. Esta marca está visible cuando el usuario edita una conexión de red virtual, una conexión VPN o una conexión ExpressRoute. De forma predeterminada, esta marca está deshabilitada cuando un sitio o circuito de ExpressRoute se conecta a un centro. Está habilitada de forma predeterminada cuando se agrega una conexión de red virtual para conectar una red virtual a un centro de conectividad virtual. La ruta predeterminada no se origina en el centro de conectividad de Virtual WAN; la ruta se propaga si ya la ha aprendido el centro de Virtual WAN como resultado de la implementación de un firewall en el centro o en caso de que otro sitio conectado tenga habilitada la tunelización forzada.

### <a name="how-does-the-virtual-hub-in-a-virtual-wan-select-the-best-path-for-a-route-from-multiple-hubs"></a>Cómo selecciona el centro virtual de una instancia de Virtual WAN la mejor ruta de varios centros

Si un centro de conectividad virtual aprende la misma ruta de varios centros de conectividad remotos, el orden en el que decide es el siguiente:

1. Coincidencia de prefijo más larga.
2. Rutas locales en vez de rutas entre centros de conectividad.
3. Rutas estáticas en vez de BGP: en el contexto de la decisión tomada por el enrutador del centro de conectividad virtual. Sin embargo, si la decisión la toma VPN Gateway cuando un sitio anuncia rutas a través de BGP o proporciona prefijos de dirección estáticos, es posible que se prefieran rutas estáticas a rutas BGP.
4. ExpressRoute (ER) en lugar de VPN: se prefiere ER frente a VPN cuando el contexto es un centro de conectividad local. La conectividad de tránsito entre circuitos ExpressRoute solo está disponible a través de Global Reach. Por lo tanto, en escenarios en los que el circuito ExpressRoute está conectado a un centro de conectividad y hay otro circuito ExpressRoute conectado a un centro de conectividad diferente con una conexión VPN, es posible que se prefiera la VPN para escenarios entre centros de conectividad.
5. Longitud de la ruta de acceso del sistema autónomo.

### <a name="does-virtual-wan-hub-allow-connectivity-between-expressroute-circuits"></a>¿El centro de conectividad de Virtual WAN permite la conectividad entre circuitos ExpressRoute?

El tránsito entre ER y ER siempre se realiza a través de Global Reach. Las puertas de enlace de centro de conectividad virtual se implementan en las regiones de Azure o del controlador de dominio. Cuando dos circuitos ExpressRoute se conectan a través de Global Reach, no es necesario que el tráfico llegue desde los enrutadores perimetrales hasta el controlador de dominio del centro de conectividad.

### <a name="is-there-a-concept-of-weight-in-azure-virtual-wan-expressroute-circuits-or-vpn-connections"></a>¿Existe un concepto de peso en los circuitos ExpressRoute de Azure Virtual WAN o en las conexiones VPN?

Cuando varios circuitos ExpressRoute están conectados a un centro de conectividad virtual, el peso del enrutamiento en la conexión proporciona un mecanismo para que ExpressRoute en el centro de conectividad virtual prefiera un circuito antes que el otro. No hay ningún mecanismo para establecer un peso en una conexión VPN. Azure siempre prefiere una conexión ExpressRoute antes que una conexión VPN en un solo centro de conectividad.

### <a name="does-virtual-wan-prefer-expressroute-over-vpn-for-traffic-egressing-azure"></a>¿Virtual WAN prefiere ExpressRoute antes que una VPN para el tráfico que sale de Azure?

Sí.

### <a name="when-a-virtual-wan-hub-has-an-expressroute-circuit-and-a-vpn-site-connected-to-it-what-would-cause-a-vpn-connection-route-to-be-preferred-over-expressroute"></a>Cuando un centro de conectividad de Virtual WAN tiene un circuito ExpressRoute y un sitio VPN conectados a él, ¿qué haría que una ruta de conexión VPN se prefiriera por encima de ExpressRoute?

Cuando un circuito ExpressRoute se conecta a un centro de conectividad virtual, los enrutadores perimetrales de Microsoft son el primer nodo para la comunicación entre el entorno local y Azure. Estos enrutadores perimetrales se comunican con las puertas de enlace de ExpressRoute de Virtual WAN que, a su vez, aprenden las rutas del enrutador del centro de conectividad virtual que controla todas las rutas entre todas las puertas de enlace de Virtual WAN. Los enrutadores perimetrales de Microsoft prefieren procesar mejor las rutas de ExpressRoute del centro de conectividad virtual antes que las rutas aprendidas del entorno local. Si, por algún motivo, la conexión VPN se convierte en el medio principal por el que el centro de conectividad virtual aprenda las rutas (por ejemplo, escenarios de conmutación por error entre ExpressRoute y VPN), a menos que el sitio VPN tenga una longitud de ruta AS mayor, el centro de conectividad virtual seguirá compartiendo las rutas de VPN aprendidas con la puerta de enlace de ExpressRoute, lo que hará que los enrutadores de Microsoft Edge prefieran las rutas VPN antes que las rutas locales.

### <a name="when-two-hubs-hub-1-and-2-are-connected-and-there-is-an-expressroute-circuit-connected-as-a-bow-tie-to-both-the-hubs-what-is-the-path-for-a-vnet-connected-to-hub-1-to-reach-a-vnet-connected-in-hub-2"></a>Cuando dos centros de conectividad (1 y 2) están conectados y hay un circuito ExpressRoute conectado como un lazo para ambos centros de conectividad, ¿cuál es la ruta de acceso de una red virtual conectada al centro de conectividad 1 para llegar a una red virtual conectada en el centro de conectividad 2?

El comportamiento actual es preferir la ruta de acceso del circuito ExpressRoute frente a una conexión entre centros para la conectividad de red virtual a red virtual. Sin embargo, esto no se recomienda en una configuración de Virtual WAN. El equipo de Virtual WAN está trabajando en una corrección para permitir la preferencia de centro de conectividad a centro de conectividad frente a la ruta de acceso de ExpressRoute. La recomendación es para que varios circuitos ExpressRoute (proveedores distintos) se conecten a un centro de conectividad y usen la conectividad entre centros de conectividad que proporciona Virtual WAN para los flujos de tráfico entre regiones.

### <a name="can-hubs-be-created-in-different-resource-group-in-virtual-wan"></a>¿Se pueden crear concentradores en un grupo de recursos diferente en Virtual WAN?
Sí. Esta opción solo está disponible actualmente mediante PowerShell. El portal de Virtual WAN asigna los concentradores en el mismo grupo de recursos que el propio recurso de Virtual WAN.

### <a name="is-there-support-for-ipv6-in-virtual-wan"></a>¿Hay compatibilidad con IPv6 en un Virtual WAN?

IPv6 no se admite en el centro de conectividad de Virtual WAN y sus puertas de enlace. Si tiene una red virtual que tiene compatibilidad con IPv4 e IPv6 y desea conectar la red virtual a Virtual WAN, este escenario no se admite actualmente. 

### <a name="what-is-the-recommended-api-version-to-be-used-by-scripts-automating-various-virtual-wan-functionalities"></a>¿Cuál es la versión de API recomendada para utilizar en los scripts que automatizan varias funcionalidades de Virtual WAN?

Se requiere una versión mínima de 05-01-2020 (1 de mayo de 2020). 

### <a name="are-there-any-virtual-wan-limits"></a>¿Hay algún límite de Virtual WAN?

Consulte la sección [Límites de Virtual WAN](../articles/azure-resource-manager/management/azure-subscription-service-limits.md#virtual-wan-limits) en la página de límites de suscripciones y servicios.

### <a name="what-are-the-differences-between-the-virtual-wan-types-basic-and-standard"></a>¿Cuáles son las diferencias entre los tipos de instancias de Virtual WAN (básico y estándar)?

Consulte [Redes WAN virtuales de tipo Básico y Estándar](../articles/virtual-wan/virtual-wan-about.md#basicstandard). Para ver los precios, consulte la página de [precios](https://azure.microsoft.com/pricing/details/virtual-wan/).
