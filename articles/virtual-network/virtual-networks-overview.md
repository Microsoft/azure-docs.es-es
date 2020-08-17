---
title: Azure Virtual Network | Microsoft Docs
description: Conozca los conceptos y las características de Azure Virtual Network, como el espacio de direcciones, las subredes, las regiones y las subscripciones.
services: virtual-network
documentationcenter: na
author: anavinahar
tags: azure-resource-manager
Customer intent: As someone with a basic network background that is new to Azure, I want to understand the capabilities of Azure Virtual Network, so that my Azure resources such as VMs, can securely communicate with each other, the internet, and my on-premises resources.
ms.service: virtual-network
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 06/19/2019
ms.author: anavin
ms.openlocfilehash: db3eae5cd31fff0db465389ea4a09b1666453634
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87387003"
---
# <a name="what-is-azure-virtual-network"></a>¿Qué es Azure Virtual Network?

Azure Virtual Network (VNet) es el bloque de creación fundamental de una red privada en Azure. VNet permite muchos tipos de recursos de Azure, como Azure Virtual Machines (máquinas virtuales), para comunicarse de forma segura entre usuarios, con Internet y con las redes locales. VNet es similar a una red tradicional que funcionaría en su propio centro de datos, pero aporta las ventajas adicionales de la infraestructura de Azure, como la escala, la disponibilidad y el aislamiento.

## <a name="vnet-concepts"></a>Conceptos de VNet

- **Espacio de direcciones**: Al crear una instancia de VNet tiene que especificar un espacio de direcciones IP privado personalizado mediante direcciones públicas y privadas (RFC 1918). Azure asigna a los recursos de una red virtual una dirección IP privada desde el espacio de direcciones que asigne. Por ejemplo, si implementa una máquina virtual en una red virtual con el espacio de direcciones, 10.0.0.0/16, la máquina virtual se asignará a una dirección IP privada como 10.0.0.4.
- **Subredes:** Las subredes le permiten segmentar la red virtual en una o varias subredes y asignar una parte del espacio de direcciones de la red virtual para cada subred. A continuación, puede implementar los recursos de Azure en una subred específica. Al igual que en una red tradicional, las subredes permiten segmentar el espacio de direcciones de red virtual en segmentos que sean adecuados para la red interna de la organización. Esto también mejora la eficacia de la asignación de dirección. Puede proteger los recursos dentro de las subredes mediante grupos de seguridad de red. Para más información, consulte[Grupo de seguridad de red](security-overview.md).
- **Regiones**: El ámbito de VNet es una sola región o ubicación; sin embargo, se pueden conectar entre sí varias redes virtuales de diferentes regiones mediante el emparejamiento de red virtual.
- **Subscription** (Suscripción): VNet limita su ámbito a una suscripción. Puede implementar varias redes virtuales dentro de cada [suscripción](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#subscription) y [región](../azure-glossary-cloud-terminology.md?toc=%2fazure%2fvirtual-network%2ftoc.json#region) de Azure.

## <a name="best-practices"></a>Procedimientos recomendados

Al crear la red en Azure, es importante tener en cuenta los siguientes principios universales de diseño:

- Asegúrese de que los espacios de dirección no se superponen. Asegúrese de que el espacio de direcciones (bloque CIDR) de red virtual no se superponen con otros intervalos de red de la organización.
- Las subredes no deben cubrir el espacio de direcciones completo de la red virtual. Planee con antelación y reserve algún espacio de direcciones para el futuro.
- Se recomienda que tenga menos redes virtuales de gran tamaño que múltiples redes virtuales pequeñas. Esto impedirá la sobrecarga de administración.
- Asegure las redes virtual mediante la asignación de grupos de seguridad de red (NSG) a sus subredes.

## <a name="communicate-with-the-internet"></a>Comunicación con internet

De manera predeterminada, todos los recursos de una red virtual tienen comunicación de salida hacia Internet. Para comunicarse con un recurso entrante, asígnele una dirección IP pública o un equilibrador de carga público. También puede usar la dirección IP pública o el equilibrador de carga público para administrar las conexiones salientes.  Para más información sobre las conexiones salientes en Azure, consulte [Conexiones salientes](../load-balancer/load-balancer-outbound-connections.md), [Direcciones IP públicas](virtual-network-public-ip-address.md) y [Equilibrador de carga](../load-balancer/load-balancer-overview.md).

>[!NOTE]
>Cuando se usa solo una instancia interna de [Standard Load Balancer](../load-balancer/load-balancer-standard-overview.md), la conectividad de salida no está disponible hasta que define cómo desea que las [conexiones salientes](../load-balancer/load-balancer-outbound-connections.md) trabajen con una dirección IP pública o un equilibrador de carga público en el nivel de instancia.

## <a name="communicate-between-azure-resources"></a>Comunicación entre recursos de Azure

Los recursos de Azure se comunican de manera segura entre sí de una de las maneras siguientes:

- **Mediante una red virtual**: tanto las máquinas virtuales como otros tipos de recursos de Azure se pueden implementar en una red virtual, como Azure App Service Environment, Azure Kubernetes Service (AKS) y Azure Virtual Machine Scale Sets. Para ver una lista completa de los recursos de Azure que puede implementar en una red virtual, consulte [Integración de red virtual para los servicios de Azure](virtual-network-for-azure-services.md).
- **Mediante un punto de conexión de servicio de red virtual**: extienda el espacio de direcciones privadas de la red virtual y la identidad de la red virtual a los recursos del servicio de Azure, como cuentas de Azure Storage y Azure SQL Database, mediante una conexión directa. Los puntos de conexión de los servicios permiten proteger los recursos críticos de los servicios de Azure únicamente en una red virtual. Para más información, consulte [Puntos de conexión de servicio de red virtual](virtual-network-service-endpoints-overview.md).
- **Mediante emparejamiento de VNet**: Las redes virtuales se pueden conectar entre sí, lo que permite que los recursos de cualquiera de ellas se comuniquen entre sí mediante el emparejamiento de red virtual. Las redes virtuales que conecte pueden estar en la misma región de Azure o en regiones distintas. Para más información, consulte [Emparejamiento de redes virtuales de Azure](virtual-network-peering-overview.md).

## <a name="communicate-with-on-premises-resources"></a>Comunicación con recursos locales

Puede conectar sus equipos y redes local a una red virtual mediante cualquier combinación de las siguientes opciones:

- **Red privada virtual (VPN) de punto a sitio**: se establece entre una red virtual y un equipo individual de la red. Cada equipo que desea establecer conectividad con una red virtual debe configurar su conexión. Este tipo de conexión es muy útil cuando se está comenzando con Azure, o para los desarrolladores, porque requiere poco o ningún cambio en la red existente. La comunicación entre el equipo y una red virtual se envía mediante un túnel cifrado a través de internet. Para más información, consulte [VPN de punto a sitio](../vpn-gateway/point-to-site-about.md?toc=%2fazure%2fvirtual-network%2ftoc.json#).
- **VPN de sitio a sitio**: se establece entre un dispositivo VPN local y una instancia de Azure VPN Gateway que está implementada en una red virtual. Este tipo de conexión permite que cualquier recurso local que autorice acceda a una red virtual. La comunicación entre un dispositivo VPN local y una puerta de enlace de VPN de Azure se envía mediante un túnel cifrado a través de internet. Para más información, consulte [VPN de sitio a sitio](../vpn-gateway/design.md?toc=%2fazure%2fvirtual-network%2ftoc.json#s2smulti).
- **Azure ExpressRoute:** se establece entre la red y Azure, mediante un asociado de ExpressRoute. Esta conexión es privada. El tráfico no pasa por Internet. Para más información, consulte [ExpressRoute](../expressroute/expressroute-introduction.md?toc=%2fazure%2fvirtual-network%2ftoc.json).

## <a name="filter-network-traffic"></a>Filtrado del tráfico de red

Puede filtrar el tráfico de red entre subredes mediante una o ambas de las siguientes opciones:

- **Grupos de seguridad de red:** los grupos de seguridad de red y los grupos de seguridad de aplicaciones pueden contener varias reglas de seguridad de entrada y salida que le permiten filtrar el tráfico que llega y sale de los recursos por dirección IP, puerto y protocolo de origen y destino. Para más información, vea [grupos de seguridad de red](security-overview.md#network-security-groups) y [grupos de seguridad de aplicaciones](security-overview.md#application-security-groups).
- **Aplicaciones virtuales de red**: una aplicación de red virtual es una máquina virtual que ejecuta una función de red, como un firewall, la optimización de WAN u otra función de red. Para ver una lista de las aplicaciones virtuales de red que se pueden implementar en una red virtual, consulte [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking?page=1&subcategories=appliances).

## <a name="route-network-traffic"></a>Enrutado del tráfico de red

De forma predeterminada Azure enruta el tráfico entre subredes, redes virtuales conectadas, redes locales e Internet. Puede implementar una o ambas de las siguientes opciones para reemplazar las rutas predeterminadas que crea Azure:

- **Tablas de ruta**: puede crear tablas de ruta personalizadas con las rutas que controlan a dónde se enruta el tráfico para cada subred. Más información sobre las [tablas de rutas](virtual-networks-udr-overview.md#user-defined).
- **Rutas de Protocolo de puerta de enlace de borde (BGP)** : si conecta la red virtual a su red local mediante una conexión de Azure VPN Gateway o ExpressRoute, puede propagar las rutas BGP locales a sus redes virtuales. Más información sobre el uso de BGP con [Azure VPN Gateway](../vpn-gateway/vpn-gateway-bgp-overview.md?toc=%2fazure%2fvirtual-network%2ftoc.json) y [ExpressRoute](../expressroute/expressroute-routing.md?toc=%2fazure%2fvirtual-network%2ftoc.json#dynamic-route-exchange).

## <a name="virtual-network-integration-for-azure-services"></a>Integración de red virtual para los servicios de Azure

La integración de servicios de Azure en una red virtual de Azure permite el acceso privado al servicio desde las máquinas virtuales o los recursos de proceso de la red virtual.
Puede integrar los servicios de Azure en la red virtual con las siguientes opciones:
- Implementación de [instancias dedicadas del servicio](virtual-network-for-azure-services.md) en una red virtual. Luego se puede acceder de forma privada a los servicios dentro de la red virtual y desde redes locales.
- Usando [Private Link](../private-link/private-link-overview.md) para acceder de forma privada a una instancia específica del servicio desde la red virtual y desde las redes locales.
- También puede acceder al servicio mediante puntos de conexión públicos si amplía una red virtual al servicio a través de los [puntos de conexión de servicio](virtual-network-service-endpoints-overview.md). Los puntos de conexión de servicio permiten que los recursos de servicio se protejan en la red virtual.
 

## <a name="azure-vnet-limits"></a>Límites de Azure VNet

Hay ciertos límites en torno al número de recursos de Azure que puede implementar. La mayoría de los límites de redes de Azure están en los valores máximos. Sin embargo, puede [aumentar ciertos límites de red](../azure-portal/supportability/networking-quota-requests.md) como se especifica en la [página de límites de red virtual](../azure-resource-manager/management/azure-subscription-service-limits.md#networking-limits). 

## <a name="pricing"></a>Precios

No hay ningún cargo por usar Azure VNet, es libre de costo. Los cargos estándar son aplicables para recursos como Virtual Machines (máquinas virtuales) y otros productos. Para más información, consulte [Precios de Virtual Network](https://azure.microsoft.com/pricing/details/virtual-network/) y la [Calculadora de precios](https://azure.microsoft.com/pricing/calculator/) de Azure.

## <a name="next-steps"></a>Pasos siguientes

 Para empezar a usar una red virtual, créela, implemente en ella varias máquinas virtuales y establezca comunicación entre las máquinas virtuales. Para aprender a hacerlo, consulte la guía de inicio rápido [Creación de una red virtual](quick-create-portal.md).
