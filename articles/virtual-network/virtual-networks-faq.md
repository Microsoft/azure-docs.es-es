---
title: P+F de Azure Virtual Network
titlesuffix: Azure Virtual Network
description: Las respuestas para las preguntas más frecuentes acerca de las redes virtuales de Microsoft Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: twooley
ms.service: virtual-network
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/26/2020
ms.author: kumud
ms.openlocfilehash: c242b08c598b04c84ab330e846704eace23c4858
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88205480"
---
# <a name="azure-virtual-network-frequently-asked-questions-faq"></a>Preguntas más frecuentes (P+F) acerca de Azure Virtual Network

## <a name="virtual-network-basics"></a>Conceptos básicos de Virtual Network

### <a name="what-is-an-azure-virtual-network-vnet"></a>¿Qué es Azure Virtual Network?
Azure Virtual Network es una representación de su propia red en la nube. Es un aislamiento lógico de la nube de Azure dedicada a su suscripción. Puede usar las redes virtuales para aprovisionar y administrar redes privadas virtuales (VPN) en Azure y, opcionalmente, vincular las redes virtuales con otras redes virtuales en Azure o con sus infraestructura de TI local para crear soluciones híbridas o entre entornos. Cada red virtual que se crea tiene su propio bloque CIDR y se puede vincular a otras redes locales y redes virtuales, siempre que los bloques CIDR no se superpongan. También tiene el control de la configuración del servidor DNS para redes virtuales y de la segmentación de la red virtual en subredes.

Use las redes virtuales para:

* Crear una red virtual dedicada solo en la nube privada. A veces no necesita una configuración entre entornos para su solución. Cuando se crea una red virtual, los servicios y las máquinas virtuales de la red virtual pueden comunicarse de forma directa y segura entre ellas en la nube. Sin embargo, puede configurar conexiones de punto de conexión para las máquinas virtuales y los servicios que requieren la comunicación a Internet, como parte de la solución.

* Extender su centro de datos de forma segura. Con las redes virtuales, puede crear VPN de sitio a sitio (S2S) tradicionales para ampliar la capacidad del centro de datos de forma segura. Las redes virtuales de sitio a sitio (S2S) usan IPSEC para proporcionar una conexión segura entre la puerta de enlace VPN corporativa y Azure.

* Habilitar escenarios de nube híbrida. Las redes virtuales proporcionan la flexibilidad para admitir una variedad de escenarios de nube híbrida. Puede conectar de forma segura aplicaciones basadas en la nube a cualquier tipo de sistema local como grandes sistemas y sistemas Unix.

### <a name="how-do-i-get-started"></a>¿Cómo empiezo?
Para comenzar, consulte [Documentación de Virtual Network](https://docs.microsoft.com/azure/virtual-network/). En este artículo encontrará información general y de implementación de todas las características de una red virtual.

### <a name="can-i-use-vnets-without-cross-premises-connectivity"></a>¿Puedo usar redes virtuales sin conectividad entre entornos?
Sí. Puede usar una red virtual sin necesidad de conectarse a su entorno local. Por ejemplo, podría ejecutar controladores de dominio de Active Directory para Microsoft Windows Server y granjas de servidores de SharePoint únicamente en una red virtual de Azure.

### <a name="can-i-perform-wan-optimization-between-vnets-or-a-vnet-and-my-on-premises-data-center"></a>¿Se puede realizar la optimización de una WAN entre redes virtuales o entre una red virtual y un centro de datos local?
Sí. A través de Azure Marketplace es posible implementar una [aplicación virtual de red de optimización de la WAN](https://azuremarketplace.microsoft.com/en-us/marketplace/?term=wan%20optimization) de varios proveedores.

## <a name="configuration"></a>Configuración

### <a name="what-tools-do-i-use-to-create-a-vnet"></a>¿Qué herramientas debo usar para crear una red virtual?
Para crear o configurar una red virtual se pueden usar las siguientes herramientas:

* Azure portal
* PowerShell
* Azure CLI
* Un archivo de configuración de red (netcfg; solo para redes virtuales clásicas). Consulte el artículo [Configuración de una red virtual con un archivo de configuración de red](virtual-networks-using-network-configuration-file.md).

### <a name="what-address-ranges-can-i-use-in-my-vnets"></a>¿Qué intervalos de direcciones puedo usar en mis redes virtuales?
Se recomienda usar los intervalos de direcciones enumerados en [RFC 1918](https://tools.ietf.org/html/rfc1918), que el IETF ha reservado para los espacios de direcciones privados y no enrutables:
* 10.0.0.0 a 10.255.255.255 (prefijo 10/8)
* 172.16.0.0 a 172.31.255.255 (prefijo 172.16/12)
* 192.168.0.0 a 192.168.255.255 (prefijo 192.168/16)

Otros espacios de direcciones pueden funcionar, pero es posible que tengan efectos secundarios no deseados.

Además, no se pueden agregar los intervalos de direcciones siguientes:
* 224.0.0.0/4 (multidifusión)
* 255.255.255.255/32 (difusión)
* 127.0.0.0/8 (bucle invertido)
* 169.254.0.0/16 (local de vínculo)
* 168.63.129.16/32 (DNS interno)

### <a name="can-i-have-public-ip-addresses-in-my-vnets"></a>¿Puedo tener direcciones IP públicas en mis redes virtuales?
Sí. Para más información sobre los intervalos de direcciones IP públicas, consulte [Creación de una red virtual](manage-virtual-network.md#create-a-virtual-network). Las direcciones IP públicas no son accesibles directamente desde Internet.

### <a name="is-there-a-limit-to-the-number-of-subnets-in-my-vnet"></a>¿Existe algún límite en el número de subredes de que puede tener una red virtual?
Sí. Para más información, consulte los [límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits). Los espacios de direcciones de las subredes no pueden superponerse entre sí.

### <a name="are-there-any-restrictions-on-using-ip-addresses-within-these-subnets"></a>¿Hay alguna restricción en el uso de direcciones IP dentro de estas subredes?
Sí. Azure reserva 5 direcciones IP dentro de cada subred. Son x.x.x.0-x.x.x.3 y la última dirección de la subred. x.x.x.1-x.x.x.3 está reservada en cada subred para los servicios de Azure.   
- x.x.x.0: Dirección de red
- x.x.x.1: Reservado por Azure para la puerta de enlace predeterminada
- x.x.x.2, x.x.x.3: Reservado por Azure para asignar las direcciones IP de Azure DNS al espacio de red virtual
- x.x.x.255: Dirección de difusión de red

### <a name="how-small-and-how-large-can-vnets-and-subnets-be"></a>¿Qué tamaños mínimo y máximo pueden tener las redes virtuales y las subredes?
La subred IPv4 más pequeña admitida es /29 y la más grande /8 (mediante definiciones de subred CIDR).  Las subredes IPv6 deben tener un tamaño exactamente de /64.  

### <a name="can-i-bring-my-vlans-to-azure-using-vnets"></a>¿Puedo llevar mis VLAN a Azure mediante redes virtuales?
No. Las redes virtuales son superposiciones de nivel 3. Azure no admite ninguna semántica de nivel 2.

### <a name="can-i-specify-custom-routing-policies-on-my-vnets-and-subnets"></a>¿Puedo especificar directivas de enrutamiento personalizadas en mis redes virtuales y subredes?
Sí. Puede crear una tabla de rutas y asociarla a una subred. Para más información sobre el enrutamiento en Azure, consulte [Introducción al enrutamiento](virtual-networks-udr-overview.md#custom-routes).

### <a name="do-vnets-support-multicast-or-broadcast"></a>¿Las redes virtuales admiten la multidifusión o la difusión?
No. No se admite difusión ni multidifusión.

### <a name="what-protocols-can-i-use-within-vnets"></a>¿Qué protocolos puedo usar en las redes virtuales?
En las redes virtuales se pueden usar los protocolos TCP, UDP e ICMP TCP/IP. La unidifusión se admite dentro de las redes virtuales, a excepción del Protocolo de configuración dinámica de host (DCHP) a través de unidifusión (puerto de origen UDP/68 / puerto de destino UDP/67) y el puerto de origen UDP 65330, que se reserva para el anfitrión. La multidifusión, la difusión, los paquetes encapsulados IP en IP y los paquetes de encapsulación de enrutamiento genérico (GRE) se bloquean en las subredes. 

### <a name="can-i-ping-my-default-routers-within-a-vnet"></a>¿Puedo hacer ping a mis enrutadores predeterminados dentro de una red virtual?
No.

### <a name="can-i-use-tracert-to-diagnose-connectivity"></a>¿Puedo usar tracert para diagnosticar la conectividad?
No.

### <a name="can-i-add-subnets-after-the-vnet-is-created"></a>¿Puedo agregar subredes una vez creada la red virtual?
Sí. Se pueden agregar subredes a redes virtuales en cualquier momento siempre y cuando el intervalo de direcciones de subred no sea parte de otra subred y se haya dejado espacio disponible en el intervalo de direcciones de la red virtual.

### <a name="can-i-modify-the-size-of-my-subnet-after-i-create-it"></a>¿Puedo modificar el tamaño de la subred después de crearla?
Sí. Puede agregar, quitar, expandir o contraer una subred si no hay máquinas virtuales ni servicios implementados en ella.

### <a name="can-i-modify-vnet-after-i-created-them"></a>¿Puedo modificar subredes después de crearlas?
Sí. Puede agregar, quitar y modificar los bloques CIDR usados por una red virtual.

### <a name="if-i-am-running-my-services-in-a-vnet-can-i-connect-to-the-internet"></a>¿Puedo conectarme a Internet si ejecuto mis servicios en una red virtual?
Sí. Todos los servicios implementados dentro de una red virtual pueden establecer una conexión a Internet saliente. Para más información sobre las conexiones a Internet salientes en Azure, consulte [Conexiones salientes en Azure](../load-balancer/load-balancer-outbound-connections.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Si quiere establecer una conexión entrante a un recurso implementado mediante Resource Manager, el recurso debe tener asignada una dirección IP pública. Para más información sobre las direcciones IP públicas, consulte [Direcciones IP publicas](virtual-network-public-ip-address.md). Cada servicio en la nube de Azure implementado en Azure tiene asignada una dirección IP virtual direccionable de forma pública. Para permitir que estos servicios acepten conexiones de Internet, se definen puntos de conexión de entrada para roles y puntos de conexión de PaaS en las máquinas virtuales.

### <a name="do-vnets-support-ipv6"></a>¿Las redes virtuales admiten IPv6?
Sí, las redes virtuales pueden ser solo IPv4 o de pila dual (IPv4 + IPv6).  Para obtener más información, consulte [Información general de IPv6 para Azure Virtual Networks](./ipv6-overview.md).

### <a name="can-a-vnet-span-regions"></a>¿Puede una red virtual abarcar varias regiones?
No. Una red virtual está limitada a una única región. Una red virtual, sin embargo, abarca zonas de disponibilidad. Para más información sobre las zonas de disponibilidad, consulte [Introducción a las zonas de disponibilidad](../availability-zones/az-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json). Puede conectar redes virtuales de diferentes regiones con el emparejamiento de redes virtuales. Para más información, consulte [Emparejamiento de redes virtuales](virtual-network-peering-overview.md).

### <a name="can-i-connect-a-vnet-to-another-vnet-in-azure"></a>¿Puedo conectar una red virtual a otra red virtual en Azure?
Sí. Puede conectar una red virtual a otra mediante los métodos siguientes:
- **Emparejamiento de redes virtuales**: para obtener más información, consulte la [VNet peering overview](virtual-network-peering-overview.md) (Introducción al emparejamiento de redes virtuales).
- **Una instancia de Azure VPN Gateway.** : para obtener más información, consulte [Configure a VNet-to-VNet connection](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (Configuración de una conexión de red virtual a red virtual). 

## <a name="name-resolution-dns"></a>resolución de nombres DNS

### <a name="what-are-my-dns-options-for-vnets"></a>¿Qué opciones de DNS hay para las redes virtuales?
Use la tabla de decisiones de la página [Resolución de nombres para las máquinas virtuales e instancias de rol](virtual-networks-name-resolution-for-vms-and-role-instances.md) ; le guiará por todas las opciones de DNS disponibles.

### <a name="can-i-specify-dns-servers-for-a-vnet"></a>¿Puedo especificar servidores DNS para una red virtual?
Sí. Puede especificar direcciones IP de servidor DNS en la configuración de la red virtual. La configuración se aplica al servidor DNS predeterminado en todas las máquinas virtuales de la red virtual.

### <a name="how-many-dns-servers-can-i-specify"></a>¿Cuántos servidores DNS puedo especificar?
Consulte los [límites de Azure](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#networking-limits).

### <a name="can-i-modify-my-dns-servers-after-i-have-created-the-network"></a>¿Puedo modificar mis servidores DNS después de haber creado la red?
Sí. Puede cambiar la lista de servidores DNS de la red virtual en cualquier momento. Si cambia la lista de servidores DNS, debe realizar una renovación de la concesión DHCP en todas las máquinas virtuales afectadas de la red virtual para que la nueva configuración de DNS surta efecto. En el caso de las máquinas virtuales que ejecutan el sistema operativo Windows, puede hacerlo escribiendo `ipconfig /renew` directamente en la máquina virtual. Para otros tipos de sistema operativo, consulte la documentación respectiva sobre la renovación de las concesiones DHCP. 

### <a name="what-is-azure-provided-dns-and-does-it-work-with-vnets"></a>¿Qué es un DNS proporcionado por Azure? ¿Funciona con las redes virtuales?
Un DNS proporcionado por Azure es un servicio DNS multiempresa ofrecido por Microsoft. Azure registra en este servicio todas las máquinas virtuales y las instancias de rol de servicio en la nube. Este servicio proporciona la resolución de nombres mediante nombre de host para las máquinas virtuales y las instancias de rol contenidas en el mismo servicio en la nube y mediante FQDN para las máquinas virtuales y las instancias de rol en la misma red virtual. Para más información sobre DNS, consulte [Resolución de nombres para las máquinas virtuales y las instancias de rol de Cloud Services](virtual-networks-name-resolution-for-vms-and-role-instances.md).

Existe una limitación de los 100 primeros servicios en la nube en una red virtual para la resolución de nombres entre inquilinos mediante DNS proporcionado por Azure. Si usa  su propio servidor DNS, esta limitación no es aplicable.

### <a name="can-i-override-my-dns-settings-on-a-per-vm-or-cloud-service-basis"></a>¿Puedo invalidar mi configuración de DNS por máquina virtual o servicio en la nube?
Sí. Puede configurar los servidores DNS por máquina virtual o servicio en la nube para invalidar la configuración de red predeterminada. Sin embargo, se recomienda usar DNS en toda la red siempre que sea posible.

### <a name="can-i-bring-my-own-dns-suffix"></a>¿Puedo usar mi propio sufijo DNS?
No. No puede especificar un sufijo DNS personalizado para sus redes virtuales.

## <a name="connecting-virtual-machines"></a>Conexión de máquinas virtuales

### <a name="can-i-deploy-vms-to-a-vnet"></a>¿Puedo implementar máquinas virtuales en una red virtual?
Sí. Todas las interfaces de red (NIC) conectadas a una máquina virtual implementada a través del modelo de implementación de Resource Manager deben estar conectadas a una red virtual. Las máquinas virtuales implementadas a través del modelo de implementación clásica también se pueden conectar a una red virtual.

### <a name="what-are-the-different-types-of-ip-addresses-i-can-assign-to-vms"></a>¿Cuáles son los distintos tipos de direcciones IP que se pueden asignar a máquinas virtuales?
* **Privada:** se asigna a cada uno de los NIC de todas las máquinas virtuales. Para asignar la dirección se puede usar el método estático o el dinámico. La direcciones IP privadas se asignan del intervalo especificado en la configuración de subred de la red virtual. A los recursos implementados a través del modelo de implementación clásica se les asignan direcciones IP privadas, aunque no estén conectadas a una red virtual. El comportamiento del método de asignación es diferente en función de si se ha implementado un recurso con el modelo de implementación clásica o de Resource Manager: 

  - **Resource Manager**: una dirección IP privada asignada con el método dinámico o estático permanece asignada a una máquina virtual (Resource Manager) hasta que se elimina el recurso. La diferencia es que cuando se usa el método estático el usuario selecciona la dirección que se asigna y cuando se usa el dinámico es Azure quien la elige. 
  - **Clásica**: las direcciones IP privadas asignadas con el método dinámico pueden cambiar cuando se reinicia una máquina virtual (clásica) después de haber estado en un estado detenido (desasignada). Si necesita asegurarse de que la dirección IP privada de un recurso implementado mediante el modelo de implementación clásica no cambie nunca, asigne una dirección IP privada con el método estático.

* **Pública:** opcionalmente, se puede asignar a NIC conectadas a máquinas virtuales implementadas a través del modelo de implementación de Azure Resource Manager. La dirección se puede asignar con el método de asignación estática o el de asignación dinámica. Todas las máquinas virtuales y las instancias de rol de Cloud Services implementadas a través del modelo de implementación clásica existen en un servicio en la nube, al que se asigna una dirección IP virtual (VIP) pública y *dinámica*. Si se desea, una dirección IP pública *estática*, que se denomina [dirección IP reservada](virtual-networks-reserved-public-ip.md), puede asignarse como si fuera una VIP. Las direcciones IP públicas se pueden asignar a máquinas virtuales o instancias de rol de Cloud Services individuales implementadas mediante el modelo de implementación clásica. Estas direcciones se denominan direcciones [IP públicas a nivel de instancia (ILPIP)](virtual-networks-instance-level-public-ip.md) y se puede asignar dinámicamente.

### <a name="can-i-reserve-a-private-ip-address-for-a-vm-that-i-will-create-at-a-later-time"></a>¿Puedo reservar una dirección IP interna para una máquina virtual que crearé más adelante?
No. Las direcciones IP privadas no se pueden reservar. Si hay una dirección IP privada disponible, el servidor DHCP la asigna a una máquina virtual o una instancia de rol. La máquina virtual puede ser, o no, aquella a la que quiere que se asigne la dirección IP privada. Sin embargo, la dirección IP privada de una máquina virtual ya creada se puede cambiar por cualquier dirección IP privada disponible.

### <a name="do-private-ip-addresses-change-for-vms-in-a-vnet"></a>¿Cambian las direcciones IP privadas de las máquinas virtuales en una red virtual?
Depende. Si se ha implementado la máquina virtual mediante Resource Manager, no, con independencia de si la dirección IP se asignó con el método de asignación estático o dinámico. Si la máquina virtual se implementó mediante el modelo de implementación clásica, se pueden cambiar las direcciones IP dinámicas cuando se inicia una máquina virtual después de haber estado en estado detenido (desasignado). La dirección se libera de una máquina virtual implementada con cualquiera de los modelos de implementación cuando se elimina la máquina.

### <a name="can-i-manually-assign-ip-addresses-to-nics-within-the-vm-operating-system"></a>¿Se pueden asignar manualmente direcciones IP a las NIC del sistema operativo de la máquina virtual?
Sí, pero no se recomienda a menos que es necesario, por ejemplo, al asignar varias direcciones IP a una máquina virtual. Para más información, consulte [Adición de varias direcciones IP a una máquina virtual](virtual-network-multiple-ip-addresses-portal.md#os-config). Si la dirección IP asignada a una NIC de Azure asociada a una máquina virtual cambia, y la dirección IP en el sistema operativo de la máquina virtual es diferente, se pierde la conectividad a la máquina virtual.

### <a name="if-i-stop-a-cloud-service-deployment-slot-or-shutdown-a-vm-from-within-the-operating-system-what-happens-to-my-ip-addresses"></a>¿Qué les sucede a mis direcciones IP si se detiene una ranura de implementación de un servicio en la nube o se apaga una máquina virtual desde el sistema operativo?
Nada. Las direcciones IP (VIP pública, pública y privada) siguen estando asignadas a la ranura de implementación de un servicio en la nube o a la máquina virtual.

### <a name="can-i-move-vms-from-one-subnet-to-another-subnet-in-a-vnet-without-redeploying"></a>¿Puedo mover las máquinas virtuales de una subred a otra en una red virtual sin volver a implementarla?
Sí. Puede encontrar más información en el artículo [Traslado de una máquina virtual o una instancia de rol a una subred diferente](virtual-networks-move-vm-role-to-subnet.md).

### <a name="can-i-configure-a-static-mac-address-for-my-vm"></a>¿Puedo configurar una dirección MAC estática para mi máquina virtual?
No. Una dirección MAC no se puede configurar de forma estática.

### <a name="will-the-mac-address-remain-the-same-for-my-vm-once-its-created"></a>¿Seguirá siendo la dirección MAC la misma en mi máquina virtual una vez que se ha creado?
Sí, la dirección MAC de una máquina virtual no cambia, independientemente de que esta se haya implementado a través del modelo de implementación clásica o de Resource Manager, hasta que se elimina. Antes, la dirección MAC se liberaba si la máquina virtual se detenía (se desasignaba), pero ahora la dirección MAC se conserva aunque la máquina virtual se encuentre en estado desasignada. La dirección MAC permanece asignada a la interfaz de red hasta que esta se elimina o se cambia la dirección IP privada asignada a la configuración de IP principal de la interfaz de red principal. 

### <a name="can-i-connect-to-the-internet-from-a-vm-in-a-vnet"></a>¿Puedo conectarme a Internet desde una máquina virtual de una red virtual?
Sí. Todas las máquinas virtuales y las instancias de rol de Cloud Services implementadas dentro de una red virtual pueden conectarse a Internet.

## <a name="azure-services-that-connect-to-vnets"></a>Servicios de Azure que se conectan a redes virtuales

### <a name="can-i-use-azure-app-service-web-apps-with-a-vnet"></a>¿Se puede usar Azure App Service Web Apps con una red virtual?
Sí. Puede implementar Web Apps dentro de una red virtual mediante un ASE (App Service Environment), conectar el back-end de las aplicaciones a una red virtual con Integración con red virtual y bloquear el tráfico entrante a la aplicación con puntos de conexión de servicio. Para más información, consulte los siguientes artículos.

* [Características de redes de App Service](../app-service/networking-features.md)
* [Creación de Web Apps en un entorno de App Service Environment](../app-service/environment/app-service-web-how-to-create-a-web-app-in-an-ase.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
* [Integración de una aplicación con Azure Virtual Network](../app-service/web-sites-integrate-with-vnet.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
* [Restricciones de acceso de Azure App Service](../app-service/app-service-ip-restrictions.md)

### <a name="can-i-deploy-cloud-services-with-web-and-worker-roles-paas-in-a-vnet"></a>¿Puedo implementar Cloud Services con los roles web y de trabajo (PaaS) en una red virtual?
Sí. Opcionalmente, es posible implementar instancias de rol de Cloud Services en redes virtuales. Para hacerlo, es preciso especificar el nombre de la red virtual y las asignaciones de rol/subred en la sección de configuración de red de la configuración del servicio. No es necesario actualizar ninguno de los archivos binarios.

### <a name="can-i-connect-a-virtual-machine-scale-set-to-a-vnet"></a>¿Se puede conectar un conjunto de escalado de máquinas virtuales a una red virtual?
Sí. Debe conectar un conjunto de escalado de máquinas virtuales a una red virtual.

### <a name="is-there-a-complete-list-of-azure-services-that-can-i-deploy-resources-from-into-a-vnet"></a>¿Hay una lista completa de servicios de Azure de la que pueda implementar recursos desde una red virtual?
Sí. Para más información, consulte [Integración de red virtual para los servicios de Azure](virtual-network-for-azure-services.md).

### <a name="how-can-i-restrict-access-to-azure-paas-resources-from-a-vnet"></a>¿Cómo se puede restringir el acceso a recursos de PaaS de Azure desde una red virtual?

Los recursos implementados a través de algunos servicios PaaS de Azure (como Azure Storage y Azure SQL Database) pueden restringir el acceso de red a la red virtual mediante puntos de conexión de servicio de red virtual o Azure Private Link. Para obtener más información, consulte [la información general sobre puntos de conexión de servicio de red virtual](virtual-network-service-endpoints-overview.md) y [Azure Private Link](../private-link/private-link-overview.md)

### <a name="can-i-move-my-services-in-and-out-of-vnets"></a>¿Puedo mover mis servicios dentro y fuera de las redes virtuales?
No. No se pueden mover los servicios dentro y fuera de las redes virtuales. Para mover un recurso a otra red virtual, tendrá que eliminar el recurso y volver a implementarlo.

## <a name="security"></a>Seguridad

### <a name="what-is-the-security-model-for-vnets"></a>¿Cuál es el modelo de seguridad de las redes virtuales?
Las redes virtuales están aisladas unas de otras y de otros servicios hospedados en la infraestructura de Azure. Una máquina virtual es un límite de confianza.

### <a name="can-i-restrict-inbound-or-outbound-traffic-flow-to-vnet-connected-resources"></a>¿Puedo restringir el flujo de tráfico de entrada o salida a los recursos conectados a la red virtual?
Sí. Puede aplicar [grupos de seguridad de red](security-overview.md) a subredes individuales de una red virtual, a los NIC conectados a una red virtual, o a ambos.

### <a name="can-i-implement-a-firewall-between-vnet-connected-resources"></a>¿Se puede implementar un firewall entre los recursos conectados a una red virtual?
Sí. A través de Azure Marketplace es posible implementar una [aplicación virtual de red de firewall](https://azure.microsoft.com/marketplace/?term=firewall) de varios proveedores.

### <a name="is-there-information-available-about-securing-vnets"></a>¿Hay información disponible acerca la protección de las redes virtuales?
Sí. Para más información, consulte [Información general sobre Azure Network Security](../security/fundamentals/network-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="apis-schemas-and-tools"></a>API, esquemas y herramientas

### <a name="can-i-manage-vnets-from-code"></a>¿Puedo administrar redes virtuales mediante programación?
Sí. Puede usar API de REST en redes virtuales en los modelos de implementación de [Azure Resource Manager](/rest/api/virtual-network) y [clásica](https://go.microsoft.com/fwlink/?LinkId=296833).

### <a name="is-there-tooling-support-for-vnets"></a>¿Hay compatibilidad con las herramientas para redes virtuales?
Sí. Más información acerca del uso de:
- Azure Portal para implementar redes virtuales a través de los modelos de implementación con [Azure Resource Manager](manage-virtual-network.md#create-a-virtual-network) y [clásica](virtual-networks-create-vnet-classic-pportal.md).
- PowerShell para administrar redes virtuales que se implementan a través de los modelos de implementación con [Resource Manager](/powershell/module/az.network) y [clásica](/powershell/module/servicemanagement/azure.service/?view=azuresmps-3.7.0).
- La interfaz de la línea de comandos (CLI) de Azure para implementar y administrar redes virtuales implementadas mediante los modelos de implementación de [Resource Manager](/cli/azure/network/vnet) y [clásica](../virtual-machines/azure-cli-arm-commands.md?toc=%2fazure%2fvirtual-network%2ftoc.json#network-resources).  

## <a name="vnet-peering"></a>Emparejamiento de VNET

### <a name="what-is-vnet-peering"></a>¿Qué es el emparejamiento de VNet?
El emparejamiento de VNet (o emparejamiento de redes virtuales) permite conectar redes virtuales. Una conexión de emparejamiento de VNet entre redes virtuales permite enrutar el tráfico entre ellas de manera privada a través de direcciones IPv4. Las máquinas virtuales de las VNet emparejadas pueden comunicarse entre sí como si estuvieran dentro de la misma red. Estas redes virtuales pueden estar en la misma región o en regiones diferentes (también conocidas como emparejamiento de VNet global). También se pueden crear conexiones de emparejamiento de VNet a través de las suscripciones a Azure.

### <a name="can-i-create-a-peering-connection-to-a-vnet-in-a-different-region"></a>¿Puedo crear una conexión de emparejamiento a una red virtual en una región diferente?
Sí. El emparejamiento de VNET global permite emparejar redes virtuales en diferentes regiones. Emparejamiento de VNET global está disponible en todas las regiones públicas de Azure, en las regiones de nube de China y en regiones de nube gubernamentales. No se puede emparejar globalmente desde las regiones públicas de Azure a las regiones de nube nacionales.

### <a name="what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers"></a>¿Cuáles son las restricciones relacionadas con Emparejamiento de VNET Global y los equilibradores de carga?
Si las dos redes virtuales en dos regiones diferentes están emparejadas a través del emparejamiento de VNet global, no se puede conectar a los recursos que están detrás de una instancia básica de Load Balancer a través de la dirección IP de front-end de Load Balancer. Esta restricción no existe para una instancia de Load Balancer estándar.
Los siguientes recursos pueden usar instancias básicas de Load Balancer, lo que significa que no puede acceder a ellos desde la dirección IP de front-end de Load Balancer a través del emparejamiento de red virtual global. Sin embargo, puede usar el emparejamiento de red virtual global para llegar a los recursos directamente desde sus direcciones IP de red virtual privada, si se permite. 
- Máquinas virtuales detrás de equilibradores de carga básicos
- Conjuntos de escalado de máquinas virtuales con equilibradores de carga básicos 
- Redis Cache 
- SKU de Application Gateway (versión 1)
- Service Fabric
- MI de SQL
- API Management
- Active Directory Domain Services (ADDS)
- Logic Apps
- HDInsight
-   Azure Batch
- Entorno de App Service

Se puede conectar a estos recursos a través de ExpressRoute o de red virtual a red virtual a través de puertas de enlace de red virtuales.

### <a name="can-i-enable-vnet-peering-if-my-virtual-networks-belong-to-subscriptions-within-different-azure-active-directory-tenants"></a>¿Puedo habilitar el emparejamiento de VNET si mis redes virtuales pertenecen a suscripciones de diferentes inquilinos de Azure Active Directory?
Sí. No es posible establecer el emparejamiento de VNET (ya sea local o global) si las suscripciones pertenecen a diferentes inquilinos de Azure Active Directory. Puede hacerlo a través de PowerShell o CLI. Aún no se admite el Portal.

### <a name="my-vnet-peering-connection-is-in-initiated-state-why-cant-i-connect"></a>Mi conexión de emparejamiento de VNET se encuentra en estado *Iniciado*, ¿por qué no puedo conectarme?
Si la conexión de emparejamiento está en estado *Iniciado*, esto significa que ha creado un solo vínculo. Se debe crear un vínculo bidireccional con el fin de establecer una conexión correcta. Por ejemplo, para emparejar VNET A a VNET B, debe crearse un vínculo de VNET A a VNET B y de VNET B a VNET A. La creación de ambos vínculos cambiará el estado a *Conectado*.

### <a name="my-vnet-peering-connection-is-in-disconnected-state-why-cant-i-create-a-peering-connection"></a>Mi conexión de emparejamiento de VNet se encuentra en estado *Desconectado*, ¿por qué no puedo crear una conexión de emparejamiento?
Si la conexión de emparejamiento de VNet está en estado *Desconectado*, significa que se ha eliminado uno de los vínculos creados. Para volver a establecer una conexión de emparejamiento, deberá eliminar el vínculo y volverlo a crear.

### <a name="can-i-peer-my-vnet-with-a-vnet-in-a-different-subscription"></a>¿Puedo emparejar mi red virtual con otra en una suscripción diferente?
Sí. Puede emparejar redes virtuales entre suscripciones y entre regiones.

### <a name="can-i-peer-two-vnets-with-matching-or-overlapping-address-ranges"></a>¿Puedo emparejar dos redes virtuales con rangos de direcciones coincidentes o superpuestas?
No. Los espacios de direcciones no deben solaparse para habilitar el emparejamiento de redes virtuales.

### <a name="how-much-do-vnet-peering-links-cost"></a>¿Cuánto cuestan los vínculos de emparejamiento de redes virtuales?
La creación de una conexión de emparejamiento VNET es gratuita. Se cobra la transferencia de datos a través de conexiones de emparejamiento. [Consulte aquí](https://azure.microsoft.com/pricing/details/virtual-network/).

### <a name="is-vnet-peering-traffic-encrypted"></a>¿Está cifrado el tráfico de emparejamiento de VNET?
No. El tráfico entre recursos en redes virtuales emparejadas es privado y aislado. Sigue estando en la columna vertebral de Microsoft.

### <a name="why-is-my-peering-connection-in-a-disconnected-state"></a>¿Por qué mi conexión de emparejamiento está en estado *Desconectado*?
Las conexiones de emparejamiento de redes virtuales pasan a un estado *Desconectado* cuando se elimina un vínculo de emparejamiento de red virtual. Debe eliminar ambos vínculos para restablecer una conexión de emparejamiento correcta.

### <a name="if-i-peer-vneta-to-vnetb-and-i-peer-vnetb-to-vnetc-does-that-mean-vneta-and-vnetc-are-peered"></a>Si se empareja VNETA con VNETB y se empareja VNETB con VNETC, ¿significa que VNETA y VNETC están emparejadas?
No. No se admite el emparejamiento transitivo. Se deben emparejar VNETA y VNETC para que esto se produzca.

### <a name="are-there-any-bandwidth-limitations-for-peering-connections"></a>¿Hay alguna limitación de ancho de banda para las conexiones de emparejamiento?
No. El emparejamiento de VNET, ya sea local o global, no impone ninguna restricción de ancho de banda. El ancho de banda solo está limitado por el recurso de proceso o de máquina virtual.

### <a name="how-can-i-troubleshoot-vnet-peering-issues"></a>¿Cómo se pueden solucionar problemas de emparejamiento de redes virtuales?
Pruebe con esta [guía de solucionador de problemas](https://support.microsoft.com/en-us/help/4486956/troubleshooter-for-virtual-network-peering-issues).

## <a name="virtual-network-tap"></a>TAP de red virtual

### <a name="which-azure-regions-are-available-for-virtual-network-tap"></a>¿Qué regiones de Azure están disponibles para TAP de red virtual?
La versión preliminar de TAP de red virtual está disponible en todas las regiones de Azure. Las interfaces de red supervisadas, el recurso TAP de la red virtual y el recopilador o solución de análisis deben implementarse en la misma región.

### <a name="does-virtual-network-tap-support-any-filtering-capabilities-on-the-mirrored-packets"></a>¿El TAP de red virtual admite las funcionalidades de filtrado de los paquetes reflejados?
Las funcionalidades de filtrado no son compatibles con la versión preliminar de TAP de la red virtual. Cuando se agrega una configuración de TAP a una interfaz de red, se transmite una copia en profundidad de todo el tráfico de entrada y salida de la interfaz de red al destino de TAP.

### <a name="can-multiple-tap-configurations-be-added-to-a-monitored-network-interface"></a>¿Se pueden agregar varias configuraciones de TAP a una interfaz de red supervisada?
Una interfaz de red supervisada puede tener solo una configuración de TAP. Compruebe con la [solución de asociados](virtual-network-tap-overview.md#virtual-network-tap-partner-solutions) individual la funcionalidad de transmitir varias copias del tráfico de TAP a las herramientas de análisis de su elección.

### <a name="can-the-same-virtual-network-tap-resource-aggregate-traffic-from-monitored-network-interfaces-in-more-than-one-virtual-network"></a>¿Puede el mismo recurso TAP de red virtual agregar tráfico desde las interfaces de red supervisadas en más de una red virtual?
Sí. El mismo recurso de TAP de red virtual se puede utilizar para agregar tráfico reflejado desde las interfaces de red supervisadas en redes virtuales emparejadas en la misma suscripción o en otra. El recurso TAP de red virtual y el equilibrador de carga de destino o la interfaz de red de destino deben estar en la misma suscripción. Todas las suscripciones deben estar bajo el mismo inquilino de Azure Active Directory.

### <a name="are-there-any-performance-considerations-on-production-traffic-if-i-enable-a-virtual-network-tap-configuration-on-a-network-interface"></a>¿Existen consideraciones de rendimiento en el tráfico de producción si se habilita una configuración de TAP de red virtual en una interfaz de red?

TAP de red virtual está en versión preliminar. Durante la versión preliminar no hay ningún Acuerdo de Nivel de Servicio. La funcionalidad no debe usarse para cargas de trabajo de producción. Cuando se habilita una interfaz de red de máquina virtual con una configuración de TAP, se utilizan los mismos recursos en el host de Azure asignados a la máquina virtual para enviar el tráfico de producción para realizar la función de creación de reflejo y enviar los paquetes reflejados. Seleccione el tamaño correcto de la máquina virtual [Linux](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [Windows](../virtual-machines/linux/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) para asegurarse de que dispone de recursos suficientes para que la máquina virtual envíe el tráfico de producción y el tráfico reflejado.

### <a name="is-accelerated-networking-for-linux-or-windows-supported-with-virtual-network-tap"></a>¿Se admiten redes aceleradas para [Linux](create-vm-accelerated-networking-cli.md) o [Windows](create-vm-accelerated-networking-powershell.md) con TAP de red virtual?

Podrá agregar una configuración de TAP en una interfaz de red asociada a una máquina virtual que esté habilitada con redes aceleradas. Pero el rendimiento y la latencia en la máquina virtual se verán afectados por la adición de la configuración de TAP, ya que la descarga para reflejar el tráfico no se admite actualmente por la red acelerada de Azure.

## <a name="virtual-network-service-endpoints"></a>Puntos de conexión de servicio de red virtual

### <a name="what-is-the-right-sequence-of-operations-to-set-up-service-endpoints-to-an-azure-service"></a>¿Cuál es la secuencia correcta de operaciones para configurar los puntos de conexión de servicio en un servicio de Azure?
Existen dos pasos para asegurar un recurso de servicio de Azure mediante puntos de conexión de servicio:
1. Active los puntos de conexión de servicio para el servicio de Azure.
2. Configure las ACL de red virtual en el servicio de Azure.

El primer paso es realizar una operación del lado de red y, el segundo, es realizar una operación del lado del recurso de servicio. Según los permisos RBAC otorgados al rol de administrador, el mismo administrador o varios administradores diferentes pueden realizar ambos pasos. Le recomendamos que primero active los puntos de conexión de servicio de la red virtual antes de configurar las ACL de red virtual en el lado del servicio de Azure. Por lo tanto, los pasos deben realizarse en la secuencia que se detalló anteriormente para configurar los puntos de conexión de servicio de la red virtual.

>[!NOTE]
> Debe completar las dos operaciones descritas anteriormente antes de poder limitar el acceso del servicio de Azure a la red virtual y a la subred permitidas. Si solo activa los puntos de conexión de servicio del servicio de Azure en el lado de red no obtendrá el acceso limitado. Asimismo, también debe configurar las ACL de red virtual en el lado del servicio de Azure.

Ciertos servicios (como SQL y CosmosDB) permiten excepciones en la secuencia anterior si usa la marca **IgnoreMissingVnetServiceEndpoint**. Una vez que la marca se establece en **True**, las ACL de la red virtual pueden establecerse en el lado del servicio de Azure antes de configurar los puntos de conexión de servicio en el lado de red. Los servicios de Azure proporcionan esta marca para ayudar a los clientes en los casos en que los firewalls de IP específicos estén configurados en los servicios de Azure y la activación de los puntos de conexión de servicio en el lado de la red pueda provocar una caída de la conectividad, ya que la IP de origen cambia de una dirección IPv4 pública a una dirección privada. La configuración de las ACL de la red virtual en el lado del servicio de Azure antes de configurar los puntos de conexión de servicio en el lado de red puede ayudarle a evitar que la conectividad se vea afectada.

### <a name="do-all-azure-services-reside-in-the-azure-virtual-network-provided-by-the-customer-how-does-vnet-service-endpoint-work-with-azure-services"></a>¿Todos los servicios de Azure residen en la red virtual de Azure que proporciona el cliente? ¿Cómo funciona el punto de conexión de servicio de la red virtual con los servicios de Azure?

No, no todos los servicios de Azure residen en la red virtual del cliente. La mayoría de los servicios de datos de Azure, como Azure Storage, Azure SQL y Azure Cosmos DB, son servicios de varios inquilinos a los que se puede obtener acceso a través de direcciones IP públicas. Puede obtener más información sobre la integración de redes virtuales para los servicios de Azure [aquí](virtual-network-for-azure-services.md). 

Cuando usa la característica de puntos de conexión de servicio de la red virtual (debe activar el punto de conexión de servicio de la red virtual en el lado de la red y configurar las ACL de red virtual adecuadas en el lado del servicio de Azure), el acceso a un servicio de Azure está restringido desde una red y una subred permitidas.

### <a name="how-does-vnet-service-endpoint-provide-security"></a>¿Cómo proporciona seguridad el punto de conexión de servicio de la red virtual?

La característica de puntos de conexión de servicio de la red virtual (debe activar el punto de conexión de servicio de la red virtual en el lado de la red y configurar las ACL de red virtual adecuadas en el lado del servicio de Azure) limita el acceso a un servicio de Azure a la red virtual y la subred permitidas, lo que proporciona seguridad a nivel de red y el aislamiento del tráfico de los servicios de Azure. Todo el tráfico que usa los puntos de conexión de servicio de la red virtual fluye sobre la red troncal de Microsoft, proporcionando así otra capa de aislamiento de la red pública de Internet. Además, los clientes pueden optar por eliminar completamente el acceso público de Internet a los recursos del servicio de Azure y permitir el tráfico solo desde su red virtual a través de una combinación de firewall de IP y ACL de red virtual, lo que les permitirá proteger los recursos del servicio de Azure de accesos no autorizados.      

### <a name="what-does-the-vnet-service-endpoint-protect---vnet-resources-or-azure-service"></a>¿Qué protege el punto de conexión de servicio de la red virtual, los recursos de red virtual o el servicio de Azure?
Los puntos de conexión de servicio de la red virtual le ayudan a proteger los recursos del servicio de Azure. Los recursos de red virtual están protegidos mediante grupos de seguridad de red (NSG).

### <a name="is-there-any-cost-for-using-vnet-service-endpoints"></a>¿Hay algún costo por usar los puntos de conexión de servicio de la red virtual?

No hay ningún cargo adicional para el uso de puntos de conexión de servicio.

### <a name="can-i-turn-on-vnet-service-endpoints-and-set-up-vnet-acls-if-the-virtual-network-and-the-azure-service-resources-belong-to-different-subscriptions"></a>¿Puedo activar los puntos de conexión de servicio de la red virtual y configurar las ACL de red virtual si esta y los recursos del servicio de Azure pertenecen a suscripciones diferentes?

Sí, es posible. Las redes virtuales y los recursos del servicio de Azure pueden estar en la misma o en diferentes suscripciones. El único requisito es que tanto la red virtual como los recursos del servicio de Azure deben estar bajo el mismo inquilino de Active Directory (AD).

### <a name="can-i-turn-on-vnet-service-endpoints-and-set-up-vnet-acls-if-the-virtual-network-and-the-azure-service-resources-belong-to-different-ad-tenants"></a>¿Puedo activar los puntos de conexión de servicio de la red virtual y configurar las ACL de red virtual si esta y los recursos del servicio de Azure pertenecen a diferentes inquilinos de AD?
Sí, es posible cuando se usan puntos de conexión de servicio para Azure Storage y Azure Key Vault. Para el resto de servicio, los puntos de conexión de servicio de la red virtual y las ACL de red virtual no se admiten con los inquilinos de AD.

### <a name="can-an-on-premises-devices-ip-address-that-is-connected-through-azure-virtual-network-gateway-vpn-or-expressroute-gateway-access-azure-paas-service-over-vnet-service-endpoints"></a>¿La dirección IP de un dispositivo local que está conectada mediante la puerta de enlace de Azure Virtual Network (VPN) o la puerta de enlace de ExpressRoute puede obtener acceso al servicio de Azure PaaS mediante los puntos de conexión de servicio de la red virtual?
De forma predeterminada, los recursos de servicio de Azure protegidos para las redes virtuales no son accesibles desde redes locales. Si quiere permitir el tráfico desde el entorno local, también debe permitir las direcciones IP públicas (normalmente NAT) desde el entorno local o ExpressRoute. Estas direcciones IP se pueden agregar a través de la configuración del firewall de IP para los recursos de los servicios de Azure.

### <a name="can-i-use-vnet-service-endpoint-feature-to-secure-azure-service-to-multiple-subnets-within-a-virtual-network-or-across-multiple-virtual-networks"></a>¿Puedo usar la característica de punto de conexión de servicio de la red virtual para proteger el servicio de Azure en varias subredes de una o varias redes virtuales?
Para proteger los servicios de Azure en varias subredes que se encuentren en una o varias redes virtuales, habilite los puntos de conexión de servicio en el lado de red en cada una de las subredes de manera independiente y, a continuación, proteja los recursos del servicio de Azure en todas las subredes mediante la configuración de las ACL de red virtual adecuadas en el lado del servicio de Azure.
 
### <a name="how-can-i-filter-outbound-traffic-from-a-virtual-network-to-azure-services-and-still-use-service-endpoints"></a>¿Cómo puedo filtrar el tráfico saliente de una red virtual a los servicios de Azure y seguir usando los puntos de conexión de servicio?
Si quiere inspeccionar o filtrar el tráfico destinado a un servicio de Azure desde una red virtual, puede implementar una aplicación virtual de red dentro de la red virtual. Después, puede aplicar los puntos de conexión de servicio a la subred donde se implementa la aplicación virtual de red y se protegen los recursos de servicio de Azure solo para esta subred mediante las ACL de red virtual. Este escenario también puede resultar útil si quiere restringir el acceso de servicio de Azure desde la red virtual solo a recursos específicos de Azure, mediante el filtrado de la aplicación virtual de red. Para más información, consulte el artículo sobre la [salida con las aplicaciones de redes virtuales](https://docs.microsoft.com/azure/architecture/reference-architectures/dmz/nva-ha).

### <a name="what-happens-when-you-access-an-azure-service-account-that-has-a-virtual-network-access-control-list-acl-enabled-from-outside-the-vnet"></a>¿Qué ocurre si accede a una cuenta de servicio de Azure que tiene habilitada la lista de control de acceso (ACL) de una red virtual que está fuera de la misma red virtual?
Se devuelve el error HTTP 403 o 404.

### <a name="are-subnets-of-a-virtual-network-created-in-different-regions-allowed-to-access-an-azure-service-account-in-another-region"></a>¿Las subredes de una máquina virtual creada en regiones distintas pueden obtener acceso a una cuenta de servicio de Azure en otra región? 
Sí, para la mayoría de los servicios de Azure, las redes virtuales creadas en diferentes regiones pueden obtener acceso a los servicios de Azure en otra región a través de los puntos de conexión de servicio de la red virtual. Por ejemplo, si una cuenta de Azure Cosmos DB está en el Oeste de EE. UU. o el Este de EE. UU. y las redes virtuales están en varias regiones, la red virtual puede obtener acceso a Azure Cosmos DB. Storage y SQL son excepciones que son de naturaleza regional, y tanto la red virtual como el servicio de Azure deben estar en la misma región.
  
### <a name="can-an-azure-service-have-both-a-vnet-acl-and-an-ip-firewall"></a>¿Puede un servicio de Azure tener una ACL de red virtual y un firewall de IP?
Sí, una ACL de red virtual y el firewall de IP pueden coexistir. Ambas características se complementan entre sí para garantizar el aislamiento y la seguridad.
 
### <a name="what-happens-if-you-delete-a-virtual-network-or-subnet-that-has-service-endpoint-turned-on-for-azure-service"></a>¿Qué sucede si se elimina una red virtual o subred que tiene el punto de conexión de servicio activado para el servicio de Azure?
La eliminación de redes virtuales y subredes son operaciones independientes y se admiten incluso cuando los puntos de conexión de servicio están activados para los servicios de Azure. En los casos en que los servicios de Azure tienen configuradas las ACL de red virtual, para esas redes virtuales y subredes, la información de las ACL de la red virtual asociada con ese servicio de Azure se desactiva cuando se elimina una red virtual o subred que tiene el punto de conexión de servicio de la red virtual activado.
 
### <a name="what-happens-if-an-azure-service-account-that-has-a-vnet-service-endpoint-enabled-is-deleted"></a>¿Qué sucede si se elimina una cuenta de servicio de Azure que tiene un punto de conexión de servicio de la red virtual habilitado?
La eliminación de una cuenta de servicio de Azure es una operación independiente y se admite incluso cuando el punto de conexión de servicio está habilitado en el lado de la red y las ACL de la red virtual se configuran en el lado del servicio de Azure. 

### <a name="what-happens-to-the-source-ip-address-of-a-resource-like-a-vm-in-a-subnet-that-has-vnet-service-endpoint-enabled"></a>¿Qué sucede con la dirección IP de origen de un recurso (como una máquina virtual en una subred) que tiene habilitado el punto de conexión de servicio de la red virtual?
Si los puntos de conexión de servicio de red virtual están habilitados, las direcciones IP de los recursos de la subred de la red virtual pasarán de usar las direcciones IPV4 públicas a usar las direcciones IP privadas de Azure Virtual Network para el tráfico que fluye hacia el servicio de Azure. Tenga en cuenta que esto puede provocar un error en los firewalls de IP específicos que se configuran en una dirección IPV4 pública que estaba anteriormente en los servicios de Azure. 

### <a name="does-the-service-endpoint-route-always-take-precedence"></a>¿La ruta del punto de conexión de servicio siempre tiene prioridad?
Los puntos de conexión de servicio agregan una ruta de sistema que tiene prioridad sobre las rutas BGP y que proporciona un enrutamiento óptimo para el tráfico del punto de conexión de servicio. Los puntos de conexión de servicio siempre toman el tráfico del servicio directamente de la red virtual al servicio en la red troncal de Microsoft Azure. Para más información sobre cómo Azure selecciona una ruta, vea [Enrutamiento del tráfico de Azure Virtual Network](virtual-networks-udr-overview.md).
 
### <a name="how-does-nsg-on-a-subnet-work-with-service-endpoints"></a>¿Cómo funciona NSG en una subred con puntos de conexión de servicio?
Para alcanzar el servicio de Azure, los NSG deben permitir la conectividad de salida. Si los NSG están abiertos a todo el tráfico saliente de Internet, entonces el tráfico del punto de conexión de servicio debería funcionar. También puede limitar el tráfico saliente a las IP de servicio mediante las etiquetas de servicio.  
 
### <a name="what-permissions-do-i-need-to-set-up-service-endpoints"></a>¿Qué permisos necesito para configurar los puntos de conexión de servicio?
Un usuario con acceso de escritura a la red virtual puede configurar los puntos de conexión de servicio de forma independiente en redes virtuales. Para proteger los recursos de servicio de Azure en una red virtual, el usuario debe tener el permiso **Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/action** en las subredes que se agreguen. De forma predeterminada, este permiso se incluye en los roles de administrador de servicios integrado y puede modificarse mediante la creación de roles personalizados. Obtenga más información sobre los roles integrados y la asignación de permisos específicos a [roles personalizados](https://docs.microsoft.com/azure/role-based-access-control/custom-roles?toc=%2fazure%2fvirtual-network%2ftoc.json).
 

### <a name="can-i-filter-virtual-network-traffic-to-azure-services-allowing-only-specific-azure-service-resources-over-vnet-service-endpoints"></a>¿Puedo filtrar el tráfico de red virtual a los servicios de Azure, permitiendo únicamente recursos de servicio específicos de Azure, sobre los puntos de conexión de servicio de la red virtual? 

Las directivas de punto de conexión de servicio de la red virtual (VNet) le permiten filtrar el tráfico de red virtual a los servicios de Azure, lo que permite únicamente recursos de servicio específicos de Azure, sobre los puntos de conexión de servicio. Las directivas de punto de conexión de servicio ofrecen un control de acceso pormenorizado para el tráfico de red virtual a los servicios de Azure. Puede obtener más información acerca de las directivas de punto de conexión de servicio [aquí](virtual-network-service-endpoint-policies-overview.md).

### <a name="does-azure-active-directory-azure-ad-support-vnet-service-endpoints"></a>¿Admite Azure Active Directory (Azure AD) puntos de conexión de servicio de red virtual?

Azure Active Directory (Azure AD) no admite los puntos de conexión de servicio de forma nativa. [Aquí](https://docs.microsoft.com/azure/virtual-network/virtual-network-service-endpoints-overview) encontrará una lista completa de los servicios de Azure que admiten puntos de conexión de servicio de red virtual. Cabe mencionar que la etiqueta "Microsoft.AzureActiveDirectory" que aparece bajo los servicios compatibles con los puntos de conexión de servicio se usa para admitir los puntos de conexión de servicio para ADLS Gen 1. En el caso de ADLS Gen 1, la integración de red virtual de Azure Data Lake Storage Gen1 emplea la seguridad del punto de conexión de servicio de red virtual entre la red virtual y Azure Active Directory (Azure AD) para generar notificaciones de seguridad adicionales en el token de acceso. Estas notificaciones se usan entonces para autenticar la red virtual en la cuenta de Data Lake Storage Gen1 y permitir el acceso. Más información sobre la [integración con redes virtuales de Azure Data Lake Store Gen 1](../data-lake-store/data-lake-store-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json)

### <a name="are-there-any-limits-on-how-many-vnet-service-endpoints-i-can-set-up-from-my-vnet"></a>¿Hay algún límite en la cantidad de puntos de conexión de servicio de la red virtual que puedo configurar desde mi red virtual?
No hay límite en el número total de puntos de conexión de servicio de la red virtual en una red virtual. Para un recurso de servicio de Azure (por ejemplo, una cuenta de Azure Storage), los servicios pueden exigir límites en el número de subredes que se usan para proteger el recurso. En la tabla siguiente se muestran algunos límites de ejemplo: 

|Servicio de Azure| Límites en las reglas de red virtual|
|---|---|
|Azure Storage| 100|
|Azure SQL| 128|
|Azure SQL Data Warehouse|  128|
|Azure KeyVault|    127|
|Azure Cosmos DB|   64|
|Centro de eventos de Azure|   128|
|Azure Service Bus| 128|
|Azure Data Lake Store V1|  100|
 
>[!NOTE]
> Los límites están sujetos a cambios a discreción del servicio de Azure. Consulte la documentación de servicio correspondiente para obtener detalles acerca de los servicios. 




  



