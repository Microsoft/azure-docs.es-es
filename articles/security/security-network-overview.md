---
title: Conceptos y requisitos de seguridad de red en Azure| Microsoft Docs
description: En este artículo se proporcionan explicaciones básicas de los conceptos y requisitos fundamentales de la seguridad de red, así como información sobre lo que puede ofrecer Azure en cada una de estas áreas.
services: security
documentationcenter: na
author: TomShinder
manager: barbkess
editor: TomSh
ms.assetid: bedf411a-0781-47b9-9742-d524cf3dbfc1
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/29/2018
ms.author: terrylan
ms.openlocfilehash: 30a7eea751a9fbfa0ee95a7edb039a13860be7f8
ms.sourcegitcommit: fec0e51a3af74b428d5cc23b6d0835ed0ac1e4d8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/12/2019
ms.locfileid: "56115477"
---
# <a name="azure-network-security-overview"></a>Introducción a Azure Network Security

La seguridad de red se puede definir como el proceso de protección de recursos contra un acceso no autorizado o un ataque mediante la aplicación de controles para el tráfico de red. El objetivo es asegurarse de que solo se permita el tráfico legítimo. Azure incluye una sólida infraestructura de red que respalda sus requisitos de conectividad de aplicaciones y servicios. La conectividad de red es posible entre recursos ubicados en Azure, entre recursos locales y hospedados en Azure y entre Internet y Azure.

En este artículo se explican algunas de las opciones que Azure ofrece en el área de la seguridad de red. Puede obtener información acerca de:

* Redes de Azure
* Control de acceso de red
* Azure Firewall
* Protección del acceso remoto y la conectividad local
* Disponibilidad
* Resolución de nombres
* Arquitectura de red perimetral (DMZ)
* Azure DDoS Protection
* Azure Front Door
* Traffic Manager
* Detección de amenazas y supervisión

## <a name="azure-networking"></a>Redes de Azure

Es necesario que las máquinas virtuales de Azure estén conectadas a una instancia de Azure Virtual Network. Una red virtual es una construcción lógica creada sobre la estructura física de la red de Azure. Cada red virtual está aislada de todas las demás redes virtuales. Esto contribuye a garantizar que otros clientes de Azure no puedan obtener acceso al tráfico de red de sus implementaciones.

Más información:

* [Información general sobre redes virtuales](../virtual-network/virtual-networks-overview.md)

## <a name="network-access-control"></a>Control de acceso de red

El control de acceso de red es el acto de limitar la conectividad entre subredes o dispositivos específicos dentro de una red virtual. El objetivo del control de acceso de red es limitar el acceso a las máquinas virtuales y los servicios a los usuarios y dispositivos aprobados. Los controles de acceso se basan en decisiones que permiten o deniegan la conexión a o desde la máquina virtual o el servicio.

Azure admite varios tipos de controles de acceso de red, como:

* Control de capa de red
* Control de ruta y tunelización forzada
* Dispositivos de seguridad de red virtual

### <a name="network-layer-control"></a>Control de capa de red

Toda implementación segura requiere alguna medida de control del acceso a la red. El objetivo del control de acceso de red es restringir la comunicación de la máquina virtual con los sistemas necesarios. Se bloquean los demás intentos de comunicación.

>[!NOTE]
Los firewalls de Storage se tratan en el artículo [Introducción a la seguridad de Azure Storage](security-storage-overview.md)

#### <a name="network-security-rules-nsgs"></a>Reglas de seguridad de red (NSG)

Si necesita un control de acceso de nivel de red básico (basado en la dirección IP y los protocolos TCP o UDP), puede usar los grupos de seguridad de red (NSG). Un grupo de seguridad de red (NSG) es un firewall de filtrado de paquetes básico y con estado que le permite controlar el acceso basado en una [5-tupla](https://www.techopedia.com/definition/28190/5-tuple). Los NSG incluyen funcionalidad para simplificar la administración y reducir las posibilidades de errores de configuración:

* Las **reglas de seguridad aumentada** simplifican la definición de la regla de NSG y permiten crear reglas complejas en lugar de tener que crear varias reglas sencillas para lograr el mismo resultado.
* Las **etiquetas de servicio** son etiquetas creadas por Microsoft que representan un grupo de direcciones IP. Se actualizan dinámicamente para incluir los intervalos IP que cumplen las condiciones que definen su inclusión en la etiqueta. Por ejemplo, si quiere crear una regla que se aplique a todo Azure Storage en la región Este puede usar Storage.EastUS
* Los **grupos de seguridad de la aplicación** permiten implementar recursos en grupos de aplicaciones y controlar el acceso a dichos recursos mediante la creación de reglas que usan los grupos de aplicaciones. Por ejemplo, si ha implementado servidores web en el grupo de aplicaciones "Webservers", puede crear una regla que se aplique a un NSG que permita el tráfico 443 desde Internet a todos los sistemas del grupo de aplicaciones "Webservers".

Los NSG no proporcionan inspección de nivel de aplicación ni controles de acceso autenticados.

Más información:

* [Grupos de seguridad de red](../virtual-network/security-overview.md)

#### <a name="asc-just-in-time-vm-access"></a>Acceso ASC Just-In-Time a la máquina virtual

[Azure Security Center](../security-center/security-center-intro.md) puede administrar los NSG en máquinas virtuales y bloquear el acceso a la máquina virtual hasta que un usuario con los permisos adecuados del control de acceso basado en rol [RBAC](../role-based-access-control/overview.md) solicite acceso. Cuando el usuario está correctamente autorizado, ASC realiza modificaciones en los NSG para permitir el acceso a los puertos seleccionados durante el tiempo especificado. Cuando expira el tiempo, los NSG se restauran a su estado seguro anterior.

Más información:

* [Acceso a Azure Security Center Just in Time](../security-center/security-center-just-in-time.md)

#### <a name="service-endpoints"></a>Puntos de conexión de servicio

Los puntos de conexión de servicio son otra forma de controlar el tráfico. Puede limitar la comunicación con los servicios admitidos a únicamente las redes virtuales a través de una conexión directa. El tráfico desde la red virtual al servicio de Azure especificado permanece en la red troncal de Microsoft Azure.  

Más información:

* [Puntos de conexión de servicio](../virtual-network/virtual-network-service-endpoints-overview.md#securing-azure-services-to-virtual-networks)

### <a name="route-control-and-forced-tunneling"></a>Control de ruta y tunelización forzada

La capacidad de controlar el comportamiento de enrutamiento en las redes virtuales es fundamental. Si el enrutamiento no está configurado correctamente, las aplicaciones y los servicios hospedados en la máquina virtual podrían conectarse a dispositivos no autorizados, incluyendo sistemas que son propiedad de atacantes potenciales o que están operados por ellos.

Las redes de Azure ofrecen la posibilidad de personalizar el comportamiento de enrutamiento del tráfico de red en las redes virtuales. Gracias a ello, puede modificar las entradas de tabla de enrutamiento predeterminadas en la red virtual. El control del comportamiento de enrutamiento le ayuda a garantizar que todo el tráfico procedente de un determinado dispositivo o grupo de dispositivos entra o sale de la red virtual a través de una ubicación específica.

Por ejemplo, suponga que tiene un dispositivo de seguridad de red virtual en la red virtual. Quiere asegurarse de que todo el tráfico que entra y sale de la red virtual pase por el dispositivo de seguridad virtual. Para ello, puede configurar [rutas definidas por el usuario](../virtual-network/virtual-networks-udr-overview.md) (UDR) en Azure.

La [tunelización forzada](https://www.petri.com/azure-forced-tunneling) es un mecanismo que puede usar para tener la seguridad de que no se permite que sus servicios inicien una conexión con dispositivos en Internet. Tenga en cuenta que este proceso no es lo mismo que aceptar conexiones entrantes y luego responder a ellas. En este caso, los servidores web de front-end tienen que responder a las solicitudes de los hosts de Internet, así que se permite que el tráfico cuyo origen es Internet entre en estos servidores web y que dichos servidores respondan.

Lo que no quiere es permitir que un servidor web front-end inicie una solicitud saliente. Estas solicitudes pueden representar un riesgo para la seguridad porque estas conexiones podrían usarse para descargar software malintencionado. Incluso si quiere que estos servidores de front-end inicien solicitudes salientes a Internet, puede que quiera obligarles a que pasen por los proxies web locales. Así podrá aprovechar las ventajas del filtrado y el registro de direcciones URL.

En su lugar, y para evitar esto, querrá usar la tunelización forzada. Cuando se habilita la tunelización forzada, todas las conexiones a Internet tienen que pasar a la fuerza por la puerta de enlace local. Puede configurar la tunelización forzada aprovechando las rutas que definió el usuario.

Más información:

* [¿Qué son las rutas definidas por el usuario y el reenvío IP?](../virtual-network/virtual-networks-udr-overview.md)

### <a name="virtual-network-security-appliances"></a>Dispositivos de seguridad de red virtual

Aunque los grupos de seguridad de red y la tunelización forzada proporcionan un nivel de seguridad en las capas de red y transporte del [modelo OSI](https://en.wikipedia.org/wiki/OSI_model), habrá ocasiones en las que quiera habilitar la seguridad en niveles más altos que la red.

Por ejemplo, sus requisitos de seguridad podrían incluir:

* Autenticación y autorización antes de permitir el acceso a la aplicación
* Detección de intrusiones y respuesta a estas
* Inspección de la capa de aplicación para comprobar la existencia de protocolos de alto nivel
* Filtrado para direcciones URL
* Antimalware y antivirus de nivel de red
* Protección contra robots
* Control de acceso a las aplicaciones
* Protección adicional de DDoS (por encima de la protección de DDoS que proporciona el mismo tejido de Azure)

Puede tener acceso a estas características de seguridad de red mejoradas mediante el uso de una solución de socio de Azure. Para encontrar las soluciones de seguridad de red más actuales de los asociados de Azure, visite [Azure Marketplace](https://azure.microsoft.com/marketplace/) y busque "seguridad" y "seguridad de red".

## <a name="azure-firewall"></a>Azure Firewall

Azure Firewall es un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure Virtual Network. Se trata de un firewall como servicio con estado completo que incorpora alta disponibilidad y escalabilidad a la nube sin restricciones. Algunas características incluyen:

* Alta disponibilidad
* Escalabilidad a la nube
* Reglas de filtrado de FQDN de aplicación
* Reglas de filtrado de tráfico de red

Más información:

* [Información general de Azure Firewall](../firewall/overview.md)

## <a name="secure-remote-access-and-cross-premises-connectivity"></a>Protección del acceso remoto y la conectividad local

La instalación, la configuración y la administración de los recursos de Azure se han de realizar de forma remota. Además, puede que quiera implementar soluciones de [TI híbrida](https://social.technet.microsoft.com/wiki/contents/articles/18120.hybrid-cloud-infrastructure-design-considerations.aspx) que tengan componentes locales y en la nube pública de Azure. Estos escenarios requieren acceso remoto seguro.

Las redes de Azure admiten los siguientes escenarios de acceso remoto seguro:

* Conexión de estaciones de trabajo individuales a una red virtual.
* Conexión de la red local a una red virtual con una VPN.
* Conexión de la red local a una red virtual con un vínculo WAN dedicado.
* Conexión de redes virtuales entre sí.

### <a name="connect-individual-workstations-to-a-virtual-network"></a>Conexión de estaciones de trabajo individuales a una red virtual.

También es posible que quiera que determinados desarrolladores o miembros del personal de operaciones administren máquinas virtuales y servicios en Azure. Por ejemplo, supongamos que necesita obtener acceso a una máquina virtual en una red virtual. Pero la directiva de seguridad no permite el acceso remoto de RDP o SSH a máquinas virtuales individuales. En este caso, puede usar una conexión [VPN de punto a sitio](../vpn-gateway/point-to-site-about.md).

La conexión VPN de punto a sitio le permite configurar una conexión privada y segura entre el usuario y la red virtual. Cuando se establece la conexión VPN, el usuario puede aplicar el protocolo de RDP o SSH a través del vínculo VPN en cualquier máquina virtual de la red virtual. (Se presupone que el usuario puede autenticarse y que está autorizado). VPN de punto a sitio admite:

* El protocolo de túnel de sockets seguro (SSTP), que es un protocolo VPN propio basado en SSL. Una solución de VPN basada en SSL puede penetrar firewalls, puesto que la mayoría de ellos abre el puerto TCP 443, que utiliza SSL. El protocolo SSTP solo se admite en dispositivos Windows. Azure es compatible con todas las versiones de Windows que tienen SSTP (Windows 7 y versiones posteriores).

* La conexión VPN IKEv2, una solución de VPN con protocolo de seguridad de Internet basada en estándares. La conexión VPN IKEv2 puede utilizarse para la conexión desde dispositivos Mac (versión de OSX 10.11 y versiones posteriores).

* [OpenVPN](https://azure.microsoft.com/updates/openvpn-support-for-azure-vpn-gateways/)

Más información:

* [Configuración de una conexión punto a sitio a una red virtual mediante PowerShell](../vpn-gateway/vpn-gateway-howto-point-to-site-rm-ps.md)

### <a name="connect-your-on-premises-network-to-a-virtual-network-with-a-vpn"></a>Conexión de la red local a una red virtual con una VPN.

Quizás quiera conectar la red corporativa completa, o algunas de sus partes, a una red virtual. Esto es habitual en escenarios de TI híbridos donde las organizaciones [amplían su centro de datos local a Azure](https://gallery.technet.microsoft.com/Datacenter-extension-687b1d84). En muchos casos, las organizaciones hospedan partes de un servicio en Azure y otras partes de forma local. Por ejemplo, pueden hacerlo cuando una solución incluye servidores web de front-end en Azure y bases de datos de back-end locales. Estos tipos de conexiones "entre locales" también permiten que la administración de recursos ubicados en Azure sea más segura y obtienen escenarios como la ampliación de controladores de dominio de Active Directory a Azure.

Una manera de lograr esto es usar una [VPN de sitio a sitio](https://www.techopedia.com/definition/30747/site-to-site-vpn). La diferencia entre una VPN de sitio a sitio y una VPN de punto a sitio es que la última conecta un dispositivo a una red virtual. Una VPN de sitio a sitio conecta toda una red (por ejemplo, una red local) a una red virtual. Las VPN de sitio a sitio a una red virtual emplean el protocolo VPN de modo de túnel IPsec de alta seguridad.

Más información:

* [Crear una red virtual con una conexión VPN de sitio a sitio mediante Azure Portal y Azure Resource Manager](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md)
* [Planeamiento y diseño de puerta de enlace de VPN](../vpn-gateway/vpn-gateway-plan-design.md)

### <a name="connect-your-on-premises-network-to-a-virtual-network-with-a-dedicated-wan-link"></a>Conexión de la red local a una red virtual con un vínculo WAN dedicado.

Las conexiones VPN de punto a sitio y de sitio a sitio son eficaces para permitir la conectividad entre locales. Sin embargo, algunas organizaciones consideran que presentan las siguientes desventajas:

* Las conexiones VPN mueven los datos a través de Internet. Como consecuencia, estas conexiones se exponen a posibles problemas de seguridad relacionados con el movimiento de los datos a través de una red pública. Además, no se puede garantizar la confiabilidad y disponibilidad de las conexiones a Internet.
* Es posible que las conexiones VPN a redes virtuales no tengan el ancho de banda necesario para algunas aplicaciones y fines, ya que agotan el límite en torno a los 200 Mbps.

Las organizaciones que necesitan el más alto nivel de seguridad y disponibilidad para sus conexiones entre locales, suelen usar vínculos WAN dedicados para la conexión a sitios remotos. Azure ofrece la posibilidad de usar un vínculo WAN dedicado que se puede emplear para conectar la red local a una red virtual. Azure ExpressRoute, ExpressRoute Direct y ExpressRoute Global Reach permiten esto.

Más información:

* [Información técnica de ExpressRoute](../expressroute/expressroute-introduction.md)
* [ExpressRoute Direct](../expressroute/expressroute-erdirect-about.md)
* [ExpressRoute Global Reach](..//expressroute/expressroute-global-reach.md)

### <a name="connect-virtual-networks-to-each-other"></a>Conexión de redes virtuales entre sí.

Es posible utilizar varias redes virtuales en las implementaciones. Hay muchas razones para hacer esto. Es posible que quiera simplificar la administración o aumentar la seguridad. Con independencia de los motivos o razones para colocar los recursos en diferentes redes virtuales, puede haber ocasiones en que quiera que los recursos de cada una de las redes se conecten entre sí.

Una opción sería conectar los servicios en una red virtual con los servicios de otra creando un "bucle de retroceso" a través de Internet. La conexión comienza en una red virtual, pasa por Internet y vuelve a la red virtual de destino. Esta opción expone la conexión a los problemas de seguridad inherentes a cualquier comunicación basada en Internet.

Una opción mejor podría ser crear una VPN de sitio a sitio que se conecte entre dos redes virtuales. Este método usa el mismo protocolo de [modo de túnel IPSec](https://technet.microsoft.com/library/cc786385.aspx) que la conexión VPN de sitio a sitio entre locales mencionada anteriormente.

La ventaja de esta opción, es que la conexión VPN se establece a través del tejido de red de Azure y no mediante una conexión a través de Internet. Gracias a ello, tiene un nivel adicional de seguridad en comparación con las VPN de sitio a sitio que se conectan a través de Internet.

Más información:

* [Configuración de una conexión entre dos redes virtuales mediante Azure Resource Manager y PowerShell](../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)

Otra manera de conectar las redes virtuales es el [Emparejamiento de VNET](../virtual-network/virtual-network-peering-overview.md). Esta característica permite conectar dos redes de Azure para que la comunicación entre ellas se realice a través de la infraestructura de red troncal de Microsoft sin ni siquiera pasar a través de Internet. El Emparejamiento de VNET puede conectar dos redes virtuales dentro de la misma región o dos redes virtuales entre regiones de Azure. Los NSG pueden utilizarse para limitar la conectividad entre diferentes sistemas o subredes.

## <a name="availability"></a>Disponibilidad

La disponibilidad es un componente clave de cualquier programa de seguridad. Si los usuarios y los sistemas no pueden acceder a lo que necesitan a través de la red, el servicio puede considerarse en peligro. Azure ofrece tecnologías de red que admiten los siguientes mecanismos de alta disponibilidad:

* Equilibrio de carga basado en HTTP
* Equilibrio de carga de nivel de red
* Equilibrio de carga global

El equilibrio de carga es un mecanismo diseñado para distribuir equitativamente las conexiones entre varios dispositivos. Los objetivos del equilibrio de carga son:

* Para aumentar la disponibilidad. Cuando se equilibra la carga de las conexiones entre varios dispositivos, uno o más de los dispositivos pueden dejar de estar disponibles sin comprometer el servicio. Los servicios que se ejecuten en los dispositivos en línea restantes pueden seguir proporcionando contenido desde el servicio.
* Para aumentar el rendimiento. Cuando se equilibra la carga de las conexiones entre varios dispositivos, un solo dispositivo no tiene que asumir toda la carga de la operación de procesamiento. En su lugar, las demandas de procesamiento y memoria para el suministro de contenido se reparten entre varios dispositivos.

### <a name="http-based-load-balancing"></a>Equilibrio de carga basado en HTTP

Las organizaciones que ejecutan servicios basados en web, a menudo necesitan tener un equilibrador de carga basado en HTTP en esos servicios web. Con ello, se garantizan unos niveles adecuados de rendimiento y alta disponibilidad. Normalmente, los equilibradores de carga basados en una red se basan en los protocolos de capa de transporte y de red. Por otro lado, los equilibradores de carga basados en HTTP toman decisiones en función de las características del protocolo HTTP.

Azure Application Gateway se encarga de proporcionar equilibrio de carga basado en HTTP a los servicios basados en la web. Application Gateway admite:

* Afinidad de sesión basada en cookies. Esta funcionalidad garantiza que las conexiones establecidas con uno de los servidores detrás del equilibrador de carga permanecen intactas entre el cliente y el servidor. Gracias a ello, las transacciones permanecen estables.
* Descarga de SSL. Cuando un cliente se conecta con el equilibrador de carga, dicha sesión se cifra mediante el protocolo HTTPS (SSL). Sin embargo, para aumentar el rendimiento, tiene la opción de usar el protocolo HTTP para realizar una conexión entre el equilibrador de carga y el servidor web que está detrás de ese equilibrador de carga. Esto se conoce como "descarga de SSL" porque los servidores web que hay detrás del equilibrador de carga no experimentan la sobrecarga del procesador implicada en el cifrado. Por lo tanto, deberían poder atender las solicitudes con mayor rapidez.
* Enrutamiento de contenido basado en direcciones URL. Esta característica hace posible que el equilibrador de carga tome decisiones sobre dónde reenviar las conexiones en función de la dirección URL de destino. Este método proporciona mucha más flexibilidad que las soluciones que toman decisiones sobre el equilibrio de carga según las direcciones IP.

Más información:

* [Introducción a Application Gateway](../application-gateway/application-gateway-introduction.md)

### <a name="network-level-load-balancing"></a>Equilibrio de carga de nivel de red

A diferencia del equilibrio de carga basado en HTTP, el equilibrio de carga de nivel de red toma las decisiones en función de la dirección IP y los números de puerto (TCP o UDP).
Puede beneficiarse del equilibrio de carga de nivel de red en Azure gracias a Azure Load Balancer. Algunas características clave de Azure Load Balancer son:

* El equilibrio de carga de nivel de red basado en la dirección IP y los números de puerto.
* La compatibilidad con cualquier protocolo de nivel de aplicación.
* La posibilidad de equilibrar la carga en máquinas virtuales e instancias de rol de servicios en la nube de Azure.
* Se puede usar en aplicaciones y máquinas virtuales accesibles desde Internet (equilibrio de carga externo) y no accesibles desde Internet (equilibrio de carga interno).
* La supervisión de puntos de conexión, que se utiliza para determinar si alguno de los servicios que hay detrás del equilibrador de carga ha dejado de estar disponible.

Más información:

* [Equilibrador de carga accesible desde Internet entre varias máquinas virtuales o servicios](../load-balancer/load-balancer-internet-overview.md)
* [Información general sobre el equilibrador de carga interno](../load-balancer/load-balancer-internal-overview.md)

### <a name="global-load-balancing"></a>Equilibrio de carga global

Algunas organizaciones querrán el nivel más alto de disponibilidad posible. Una manera de lograr este objetivo es hospedar aplicaciones en centros de datos distribuidos globalmente. Cuando una aplicación está hospedada en centros de datos repartidos por todo el mundo, una región geopolítica entera puede dejar de estar disponible, pero la aplicación puede seguir funcionando.

Esta estrategia de equilibrio de carga también puede producir mejoras en el rendimiento. Puede dirigir las solicitudes para el servicio en el centro de datos que esté más próximo al dispositivo con el que está realizando la solicitud.

En Azure, puede conseguir los beneficios del equilibrio de carga global mediante el Administrador de tráfico de Azure.

Más información:

* [¿Qué es el Administrador de tráfico?](../traffic-manager/traffic-manager-overview.md)

## <a name="name-resolution"></a>Resolución de nombres

La resolución de nombres es una función crítica para todos los servicios hospedados en Azure. Desde una perspectiva de la seguridad, poner en peligro esta función puede dar lugar a que un atacante redirija las solicitudes de sus sitios al sitio de dicho individuo. Proteger la resolución de nombres es un requisito de todos los servicios hospedados en la nube.

Hay dos tipos de resolución de nombres que debe abordar:

* Resolución de nombres interna. Los servicios de las redes virtuales, las redes locales o ambos usan esta opción. Los nombres que se usan para la resolución de nombres interna no son accesibles a través de Internet. Para lograr una seguridad óptima, es importante que el esquema de la resolución de nombres interna no sea accesible a usuarios externos.
* Resolución de nombres externa. La usan personas y dispositivos que se encuentran fuera del alcance de las redes locales y virtuales. Son los nombres que son visibles en Internet y que se usan para dirigir la conexión a los servicios basados en la nube.

Para la resolución de nombres interna, tiene dos opciones:

* Un servidor DNS de red virtual. Cuando crea una nueva red virtual, se crea un servidor DNS automáticamente. Este servidor DNS puede resolver los nombres de las máquinas ubicadas en esa red virtual. Dicho servidor DNS no es configurable y lo administra el administrador de tejido de Azure, por lo que puede ayudarle a proteger la resolución de nombres.
* Traiga su propio servidor DNS. Tiene la opción de colocar un servidor DNS de su elección en una red virtual. Este podría ser un servidor DNS integrado de Active Directory o una solución de servidor DNS dedicada que proporcione un socio de Azure que puede obtener en Azure Marketplace.

Más información:

* [Información general sobre redes virtuales](../virtual-network/virtual-networks-overview.md)
* [Manage DNS Servers used by a virtual network](../virtual-network/manage-virtual-network.md#change-dns-servers) (Administrar los servidores DNS que use una red virtual)

En cuanto a la resolución de nombres externa, tiene dos opciones:

* Hospedar su propio servidor DNS externo en el entorno local.
* Hospedar su propio servidor DNS externo con un proveedor de servicios.

Muchas organizaciones de gran tamaño hospedan sus propios servidores DNS en el entorno local. Pueden hacerlo porque tienen la experiencia en redes y la presencia global para ello.

En la mayoría de los casos, es mejor hospedar los servicios de resolución de nombres DNS con un proveedor de servicios. Estos proveedores de servicios cuentan con la experiencia en redes y la presencia global para garantizar una disponibilidad muy alta de los servicios de resolución de nombres. La disponibilidad es esencial para los servicios DNS, ya que si se produce un error en los servicios de resolución de nombres, nadie podrá establecer comunicación con los servicios accesibles desde Internet.

Azure proporciona una solución DNS externa de alta disponibilidad y elevado rendimiento en forma de Azure DNS. Esta solución de resolución de nombres externa aprovecha la infraestructura de DNS de Azure en todo el mundo. Le permite hospedar un dominio en Azure con las mismas credenciales, API, herramientas y facturación que los demás servicios de Azure. Al ser parte de Azure, también hereda los fuertes controles de seguridad incorporados en la plataforma.

Más información:

* [Introducción a Azure DNS](../dns/dns-overview.md)
* Las [zonas privadas de Azure DNS](../dns/private-dns-overview.md) le permiten configurar nombres DNS privados para los recursos de Azure en lugar de los nombres asignados automáticamente sin necesidad de agregar una solución DNS personalizada.

## <a name="perimeter-network-architecture"></a>Arquitectura de red perimetral

Muchas organizaciones de gran tamaño usan redes perimetrales para segmentar sus redes y crear una zona de protección entre Internet y sus servicios. La parte perimetral de la red se considera una zona de baja seguridad y ningún recurso de alto valor se coloca en ese segmento de red. Por lo general, verá dispositivos de seguridad de red que tienen una interfaz de red en el segmento de red perimetral. Otra interfaz de red se conecta a una red que tiene máquinas virtuales y servicios que aceptan las conexiones entrantes de Internet.

Puede diseñar redes perimetrales de diferentes maneras. La decisión de implementar una red perimetral y qué tipo de red perimetral usará, si decide usar una, depende de los requisitos de seguridad de la red.

Más información:

* [Cloud Services de Microsoft y seguridad de red](../best-practices-network-security.md)

## <a name="azure-ddos-protection"></a>Azure DDoS Protection

Los ataques por denegación de servicio distribuido (DDoS) son uno de los problemas de seguridad y disponibilidad más extendidos a los que se enfrentan los clientes que mueven sus aplicaciones a la nube. Un ataque DDoS intenta agotar los recursos de una aplicación haciendo que esta no esté disponible para los usuarios legítimos. Los ataques DDoS pueden ir dirigidos a cualquier punto de conexión que sea públicamente accesible a través de Internet.
Microsoft proporciona protección contra DDoS conocida como **Básica** como parte de la plataforma Azure. No supone ningún costo adicional e incluye la supervisión siempre activada y la mitigación en tiempo real de ataques comunes de nivel de red. Además de las protecciones incluidas con la protección contra DDoS **Básica**, se puede habilitar la opción **Estándar**. Entre las características de Protección contra DDoS estándar se incluyen:

* **Integración de plataforma nativa:** integrado de forma nativa en Azure. Incluye la configuración a través de Azure Portal. Protección contra DDoS estándar comprende sus recursos y la configuración de recursos.
* **Protección llave en mano:** la configuración simplificada protege de inmediato todos los recursos de una red virtual desde el momento en que se habilita DDoS Protection Estándar. No se requiere intervención ni definición del usuario. Protección contra DDoS estándar mitiga el ataque de forma instantánea y automática una vez detectado.
* **Supervisión continua del tráfico:** los patrones de tráfico de la aplicación se supervisan de forma ininterrumpida en busca de indicadores de ataques DDoS. La mitigación se realiza cuando se sobrepasan las directivas de protección.
* **Informes de mitigación de ataques** Los informes de mitigación de ataques usan datos de flujo de red agregados para brindar información detallada sobre los ataques dirigidos a sus recursos.
* **Registros de flujo de mitigación de ataques** Los registros de flujo de mitigación de ataques le permiten revisar el tráfico descartado, el tráfico reenviado y otros datos de ataque en tiempo real durante un ataque de DDoS.
* **Ajuste adaptable:** la generación de perfiles de tráfico inteligente va conociendo con el tiempo el tráfico de la aplicación y selecciona y actualiza el perfil que resulta más adecuado para el servicio. El perfil se ajusta a medida que el tráfico cambia con el tiempo. Protección de capa 3 a capa 7: proporciona protección contra DDoS de pila completa cuando se usa con un firewall de aplicaciones web.
* **Escala de mitigación amplia:** se pueden mitigar más de 60 tipos de ataque diferentes con capacidad global para protegerse contra los ataques DDoS más conocidos.
* **Métricas de ataques:** con Azure Monitor se puede acceder a un resumen de métricas de cada ataque.
* **Alertas de ataques:** las alertas se pueden configurar en el inicio y la detención de un ataque y a lo largo de la duración del ataque mediante métricas de ataque integradas. Las alertas se integran en el software operativo como Microsoft Azure Log Analytics, Splunk, Azure Storage, correo electrónico y Azure Portal.
* **Garantía de costo:**  créditos para servicio de escalado horizontal de aplicaciones y transferencia de datos para ataques de DDoS documentados.
* **Capacidad de respuesta rápida ante DDoS** Los clientes de DDoS Protection Standard ahora tienen acceso al equipo de Rapid Response durante un ataque activo. DDR ayuda con la realización de una investigación sobre los ataques, la personalización de mitigaciones durante un ataque y la publicación de análisis sobre tales ataques.


Más información:

* [Información general sobre la protección contra DDoS](../virtual-network/ddos-protection-overview.md)

## <a name="azure-front-door"></a>Azure Front Door

Azure Front Door Service le permite definir, administrar y supervisar el enrutamiento global del tráfico web. Optimiza el enrutamiento del tráfico para obtener un mejor rendimiento y alta disponibilidad. Azure Front Door permite crear reglas de firewall de aplicaciones web (WAF) personalizadas para el control de acceso con el fin de proteger la carga de trabajo HTTP/HTTPS frente a técnicas de explotación basadas en direcciones IP de cliente, código de país y parámetros HTTP. Además, Front Door también le permite crear reglas de limitación de velocidad para enfrentar el tráfico de bots malintencionados, lo que incluye la descarga SSL y el procesamiento de niveles de aplicación por solicitud HTTP/HTTPS.

La plataforma misma de Front Door está protegida con el nivel básico de Azure DDoS Protection. Para lograr una mayor protección, se puede habilitar Azure DDoS Protection estándar en las redes virtuales y proteger los recursos contra ataques de nivel de red (TCP/UDP) a través de la optimización automática y la mitigación. Front Door es un proxy inverso de nivel 7, solo permite que el tráfico web pase a los servidores back-end y bloquea otros tipos de tráfico de forma predeterminada.

Más información:

* Para más información sobre todo el conjunto de funcionalidades de Azure Front Door, puede revisar la [información general sobre Azure Front Door](../frontdoor/front-door-overview.md).

## <a name="azure-traffic-manager"></a>Azure Traffic Manager

Azure Traffic Manager es un equilibrador de carga de tráfico basado en DNS que le permite distribuir el tráfico de forma óptima a servicios de regiones de Azure globales, al tiempo que proporciona una alta disponibilidad y capacidad de respuesta. Traffic Manager usa DNS para dirigir las solicitudes del cliente al punto de conexión de servicio más adecuado en función de un método de enrutamiento del tráfico y el mantenimiento de los puntos de conexión. Un punto de conexión es cualquier servicio accesible desde Internet hospedado dentro o fuera de Azure. Traffic Manager supervisa los puntos de conexión y no dirige el tráfico a ningún punto de conexión que no esté disponible.

Más información:

* [Información general sobre Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md)

## <a name="monitoring-and-threat-detection"></a>Detección de amenazas y supervisión

Azure proporciona funcionalidades para ayudarle en esta área clave con la detección temprana, la supervisión y la capacidad de recopilar y revisar el tráfico de red.

### <a name="azure-network-watcher"></a>Azure Network Watcher

Gracias a Azure Network Watcher no solo puede solucionar problemas, también tendrá en sus manos un conjunto completamente nuevo de herramientas para poder identificar problemas de seguridad.

La [vista del grupo de seguridad ](../network-watcher/network-watcher-security-group-view-overview.md) le ayudará a cumplir los requisitos de seguridad y auditoría de las máquinas virtuales. Use esta característica para realizar auditorías mediante programación y así comparar las directivas de línea de base que haya definido la organización con las reglas efectivas de cada una de sus máquinas virtuales. Esto puede ayudarle a identificar cualquier cambio en la configuración.

La [captura de paquetes](../network-watcher/network-watcher-packet-capture-overview.md) le permite capturar el tráfico de red hacia y desde la máquina virtual. Puede recopilar estadísticas de red y solucionar problemas de aplicaciones, lo que puede ser una ventaja inestimable a la hora de investigar intrusiones de red. También puede usar esta característica junto con Azure Functions para iniciar las capturas de red en respuesta a alertas específicas de Azure.

Para obtener más información sobre Network Watcher y cómo comenzar a probar algunas de las funcionalidades en los laboratorios, eche un vistazo a la [introducción sobre la supervisión de Azure Network Watcher](../network-watcher/network-watcher-monitoring-overview.md).

>[!NOTE]
Para obtener las notificaciones más recientes sobre la disponibilidad y el estado de este servicio, consulte la página de [actualizaciones de Azure](https://azure.microsoft.com/updates/?product=network-watcher).

### <a name="azure-security-center"></a>Azure Security Center

Azure Security Center ayuda a evitar, detectar y responder a amenazas, al tiempo que proporciona más visibilidad y control de la seguridad de los recursos de Azure. Proporciona administración de directivas y supervisión de la seguridad integrada en las suscripciones de Azure, ayuda a detectar las amenazas que podrían pasar desapercibidas y funciona con un amplio conjunto de soluciones de seguridad.

Security Center le ayuda a optimizar y controlar la seguridad de la red realizando lo siguiente:

* Proporcionar recomendaciones de seguridad de la red.
* Supervisar el estado de la configuración de seguridad de la red.
* Alertar de las amenazas basadas en la red, tanto en los niveles de red como en el punto de conexión.

Más información:

* [Introducción al Centro de seguridad de Azure](../security-center/security-center-intro.md)

### <a name="virtual-network-tap"></a>Virtual Network TAP

Azure Virtual Network TAP (punto de acceso del terminal) permite transmitir continuamente el tráfico de red de la máquina virtual a un recopilador de paquetes de red o a una herramienta de análisis. Un asociado de la aplicación virtual de red proporciona el recopilador o la herramienta de análisis de la herramienta. Puede usar el mismo recurso de Virtual Red TAP para agregar tráfico de diferentes interfaces de red en la misma suscripción o en suscripciones distintas.

Más información:

* [Virtual network TAP](../virtual-network/virtual-network-tap-overview.md)

### <a name="logging"></a>Registro

El registro en el nivel de red es una función clave en cualquier escenario de seguridad de red. En Azure, puede registrar la información obtenida de los grupos de seguridad de red (NSG) a fin de obtener información del registro de nivel de red. Con el registro de NSG, obtiene información de:

* [Registros de actividad](../azure-monitor/platform/activity-logs-overview.md). Use estos registros para ver todas las operaciones enviadas a las suscripciones de Azure. Estos registros están habilitados de forma predeterminada y se pueden ver en Azure Portal. Anteriormente se les llamaba registros de "auditoría" o "registros operativos".
* Registros de eventos. Estos registros proporcionan información sobre las reglas de NSG que se aplicaron.
* Registro de contadores. Estos registros le permiten saber cuántas veces se aplica cada regla de NSG para denegar o permitir el tráfico.

También puede usar [Microsoft Power BI](https://powerbi.microsoft.com/what-is-power-bi/), una eficaz herramienta de visualización de datos, para ver y analizar estos registros.
Más información:

* [Análisis del registro para grupos de seguridad de red (NSG)](../virtual-network/virtual-network-nsg-manage-log.md)
