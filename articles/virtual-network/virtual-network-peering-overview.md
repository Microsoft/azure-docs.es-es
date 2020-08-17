---
title: Emparejamiento de Azure Virtual Network
titlesuffix: Azure Virtual Network
description: Obtenga información sobre el emparejamiento de red virtual en Azure, incluido cómo permite conectar redes en Azure Virtual Network.
services: virtual-network
documentationcenter: na
author: altambaw
ms.service: virtual-network
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/15/2019
ms.author: kumud
ms.openlocfilehash: 6ba0caa2546fdb9d4845e2c7adf87a06d906dd12
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87265167"
---
# <a name="virtual-network-peering"></a>Emparejamiento de redes virtuales de Azure

El emparejamiento de redes virtuales permite conectar redes sin problemas en [Azure Virtual Network](virtual-networks-overview.md). A efectos de conectividad las redes virtuales aparecen como una sola. El tráfico entre máquinas virtuales usa la infraestructura de la red troncal de Microsoft. Al igual que el tráfico entre las máquinas virtuales de la misma red, el tráfico solo se enruta a través de la red *privada* de Microsoft.

La directiva admite los siguientes tipos de emparejamiento:

* Emparejamiento de redes virtuales: conecte redes virtuales que se encuentren en la misma región de Azure.
* Emparejamiento global de redes virtuales: conexión de redes virtuales en distintas regiones de Azure.

Las ventajas del uso del emparejamiento de redes virtuales, ya sean locales o globales, son las siguientes:

* Baja latencia, conexión de gran ancho de banda entre los recursos de redes virtuales diferentes.
* La capacidad de los recursos de una red virtual para comunicarse con los de otra red virtual.
* La capacidad de transferir datos entre redes virtuales de distintas suscripciones de Azure, inquilinos de Azure Active Directory, modelos de implementación y regiones de Azure.
* La capacidad de emparejar redes virtuales creadas mediante Azure Resource Manager.
* La capacidad de emparejar una red virtual creada mediante Resource Manager con otra creada mediante el modelo de implementación clásica. Para más información sobre los modelos de implementación de Azure, consulte [Descripción de los modelos de implementación de Azure](../azure-resource-manager/management/deployment-models.md?toc=%2fazure%2fvirtual-network%2ftoc.json).
* No tienen tiempo de inactividad ni los recursos de la red virtual al crear el emparejamiento, o después.

El tráfico de red entre redes virtuales emparejadas es privado. El tráfico entre las redes virtuales se mantiene en la red troncal de Microsoft. No se requieren ninguna red pública de Internet, puertas de enlace o cifrado en la comunicación entre las redes virtuales.

## <a name="connectivity"></a>Conectividad

En el caso de las redes virtuales emparejadas, los recursos de cualquiera de ellas se pueden conectar directamente con los de la red virtual emparejada.

La latencia de red entre las máquinas virtuales de redes virtuales emparejadas en la misma región es la misma que la de una única red virtual. El rendimiento de la red se basa en el ancho de banda que se permite para la máquina virtual, en proporción a su tamaño. No hay restricciones adicionales con respecto al ancho de banda en el emparejamiento.

El tráfico entre las máquinas virtuales en las redes virtuales emparejadas se enruta directamente a través de la infraestructura de red troncal de Microsoft, no de una puerta de enlace ni de una red Internet pública.

En cualquier red virtual, se pueden aplicar grupos de seguridad de red para bloquear el acceso a otras redes o subredes virtuales.
Al configurar el emparejamiento de red virtual, abra o cierre las reglas del grupo de seguridad de red entre las redes virtuales. Si abre la conectividad completa entre redes virtuales emparejadas, puede aplicar grupos de seguridad de red a subredes o máquinas virtuales concretas para bloquear o denegar el acceso específico. La opción predeterminada es la conectividad total. Para obtener más información sobre los grupos de seguridad de red, consulte [Grupos de seguridad](security-overview.md).

## <a name="service-chaining"></a>Encadenamiento de servicios

El encadenamiento de servicios permite dirigir el tráfico de una red virtual a una aplicación virtual o puerta de enlace de una red emparejada a través de rutas definidas por el usuario.

Para habilitar el encadenamiento de servicios, configure las rutas definidas por el usuario que apuntan a máquinas virtuales de redes virtuales emparejadas como la dirección IP del *próximo salto*. Las rutas definidas por el usuario también pueden apuntar a puertas de enlace de redes virtuales para habilitar el encadenamiento de servicios.

Puede implementar redes del tipo de concentrador y radio, en los que la red virtual del concentrador hospeda componentes de la infraestructura, como una aplicación virtual de red o una puerta de enlace de VPN. Todas las redes virtuales de radio se pueden emparejar con la red virtual de concentrador. El tráfico fluye por las aplicaciones virtuales de red o las puertas de enlace de VPN de la red virtual del concentrador.

El emparejamiento de red virtual permite que el próximo salto de una ruta definida por el usuario sea la dirección IP de una máquina virtual de la red virtual emparejada o en una puerta de enlace de VPN. No puede realizar el enrutamiento entre redes virtuales con una ruta definida por el usuario que especifique una puerta de enlace de Azure ExpressRoute como del tipo de próximo salto. Para obtener más información sobre las rutas definidas por el usuario, consulte [Introducción a las rutas definidas por el usuario](virtual-networks-udr-overview.md#user-defined). Para aprender a crear una topología de red en estrella tipo hub-and-spoke, consulte [Topología en estrella tipo hub-and-spoke en Azure](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="gateways-and-on-premises-connectivity"></a>Puertas de enlace y conectividad local

Cada red virtual, incluidas las redes virtuales emparejadas, puede tener su propia puerta de enlace. Las redes virtuales pueden usar su puerta de enlace para conectarse a una red local. También puede configurar [conexiones entre redes virtuales](../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md?toc=%2fazure%2fvirtual-network%2ftoc.json) mediante puertas de enlace, incluso para redes virtuales emparejadas.

Cuando se configuran las dos opciones para la interconectividad de redes virtuales, el tráfico entre las redes virtuales se propaga a través de la configuración del emparejamiento. El tráfico usa la red troncal de Azure.

La puerta de enlace de la red virtual emparejada también se puede configurar como un punto de tránsito a una red local. En ese caso, la red virtual que usa una puerta de enlace remota no puede tener su propia puerta de enlace. Una red virtual no puede tener más de una puerta de enlace. En la red virtual emparejada, la puerta de enlace puede ser local o remota, como se muestra en el diagrama siguiente:

![tránsito de emparejamiento de redes virtuales](./media/virtual-networks-peering-overview/local-or-remote-gateway-in-peered-virual-network.png)

Tanto el emparejamiento de red virtual como el emparejamiento de red virtual global admiten el tránsito de puerta de enlace.

Se admite el tránsito de puerta de enlace entre redes virtuales creadas mediante diferentes modelos de implementación. La red virtual debe estar en la red virtual del modelo de Resource Manager. Para más información sobre el uso de una puerta de enlace de tránsito, consulte [Configure a VPN gateway for transit in a virtual network peering](../vpn-gateway/vpn-gateway-peering-gateway-transit.md?toc=%2fazure%2fvirtual-network%2ftoc.json) (Configuración de una puerta de enlace de VPN de tránsito en un emparejamiento de red virtual).

Si se emparejan redes virtuales que comparten la misma conexión de Azure ExpressRoute, el tráfico entre ellas pasa por la relación de emparejamiento. Ese tráfico usa la red troncal de Azure. Puede seguir usando las puertas de enlace locales de cada red virtual para conectarse al circuito local. De lo contrario, puede utilizar una puerta de enlace compartida y configurar el tránsito para una conectividad local.

## <a name="troubleshoot"></a>Solución de problemas

Para confirmar que las redes virtuales están emparejadas, puede comprobar las rutas eficaces. Las rutas de una interfaz de red se comprueban en cualquier subred de una red virtual. Si el emparejamiento de redes virtuales existe, todas las subredes de la red virtual tienen rutas con próximo salto del tipo *emparejamiento de VNet*, para cada espacio de direcciones en cada red virtual emparejada. Para más información, consulte [Diagnóstico de problemas de enrutamiento en una máquina virtual](diagnose-network-routing-problem.md).

En una red virtual emparejada, los problemas de conectividad con una máquina virtual se pueden solucionar mediante Azure Network Watcher. Una comprobación de conectividad permite ver cómo se enruta el tráfico desde la interfaz de red de una máquina virtual de origen a la de una máquina virtual de destino. Para más información, consulte [Solución de problemas en las conexiones con Azure Network Watcher desde Azure Portal](../network-watcher/network-watcher-connectivity-portal.md#check-connectivity-to-a-virtual-machine).

También puede probar la [solución de problemas de emparejamiento de red virtual](virtual-network-troubleshoot-peering-issues.md).

## <a name="constraints-for-peered-virtual-networks"></a>Restricciones de las redes virtuales emparejadas<a name="requirements-and-constraints"></a>

Cuando las redes virtuales están emparejadas globalmente, se aplican las siguientes restricciones:

* Los recursos de una red virtual no pueden comunicarse con la dirección IP de front-end de un equilibrador de carga interno (ILB) básico en una red virtual emparejada globalmente.
* Algunos servicios que usan un equilibrador de carga básico no funcionan en el emparejamiento de red virtual global. Para más información, consulte [¿Cuáles son las restricciones relacionadas con el emparejamiento de red virtual global y los equilibradores de carga?](virtual-networks-faq.md#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers).

Para más información, consulte [Requisitos y restricciones](virtual-network-manage-peering.md#requirements-and-constraints). Para más información sobre el número de emparejamientos que se admiten, consulte [Límites de la red](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

## <a name="permissions"></a>Permisos

Para más información sobre los permisos necesarios para crear un emparejamiento de red virtual, consulte [Permisos](virtual-network-manage-peering.md#permissions).

## <a name="pricing"></a>Precios

Hay un cargo nominal para el tráfico de entrada y salida que usa una conexión de emparejamiento de red virtual. Para más información, consulte [Precios de redes virtuales](https://azure.microsoft.com/pricing/details/virtual-network).

El tránsito de puerta de enlace es una propiedad del emparejamiento que permite a las redes virtuales utilizar una puerta de enlace de VPN o ExpressRoute de una red virtual emparejada. El tránsito de puerta de enlace funciona tanto para la conectividad entre entornos locales como para la conectividad entre redes. El tráfico a la puerta de enlace (de entrada o salida) en la red virtual emparejada incurrirá en cargos de emparejamiento de red virtual en la red virtual de radio (o en la red virtual que no sea de la puerta de enlace). Para más información, consulte [Precios de VPN Gateway](https://azure.microsoft.com/pricing/details/vpn-gateway/), donde podrá ver los cargos que supone una puerta de enlace de red privada virtual y los precios de la puerta de enlace de ExpressRoute, donde podrá ver los costos de la puerta de enlace de ExpressRoute.

>[!NOTE]
> En una versión anterior de este documento se indicaba que no se aplicarían cargos de emparejamiento de red virtual a la red virtual de radio (o la red virtual que no sea de puerta de enlace) con el tránsito de puerta de enlace. Ahora se refleja un precio preciso en la página de precios.

## <a name="next-steps"></a>Pasos siguientes

* Puede crear un emparejamiento entre dos redes virtuales. Las redes pueden pertenecer a la misma suscripción, a diferentes modelos de implementación de la misma suscripción o a diferentes suscripciones. Realice el tutorial para uno de los siguientes escenarios:

    |Modelo de implementación de Azure             | Subscription  |
    |---------                          |---------|
    |Ambas mediante Resource Manager              |[La misma](tutorial-connect-virtual-networks-portal.md)|
    |                                   |[Diferente](create-peering-different-subscriptions.md)|
    |Una mediante Resource Manager y la otra clásica  |[La misma](create-peering-different-deployment-models.md)|
    |                                   |[Diferente](create-peering-different-deployment-models-subscriptions.md)|

* Para aprender a crear una topología de red en estrella tipo hub-and-spoke, consulte [Topología en estrella tipo hub-and-spoke en Azure](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json).
* Para obtener información acerca de la configuración del emparejamiento de red virtual, consulte [Creación, cambio o eliminación de un emparejamiento de red virtual](virtual-network-manage-peering.md).
* Para conocer las respuestas a las preguntas habituales acerca del emparejamiento de red virtual y del emparejamiento de red virtual global, consulte[Emparejamiento de red virtual](virtual-networks-faq.md#vnet-peering).
