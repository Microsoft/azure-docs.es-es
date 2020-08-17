---
title: Seguridad de red para Azure Relay
description: En este artículo se describe cómo configurar el acceso desde puntos de conexión privados
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: bf49eff29385b5b72639420416df87b9187845e8
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87532992"
---
# <a name="network-security-for-azure-relay"></a>Seguridad de red para Azure Relay 
En este artículo se describe cómo usar las siguientes características de seguridad con Azure Relay: 

- Reglas de firewall de IP (versión preliminar)
- Puntos de conexión privados (versión preliminar)

> [!NOTE]
> Azure Relay no admite puntos de conexión del servicio de red. 


## <a name="ip-firewall"></a>Firewall de dirección IP 
De forma predeterminada, se puede acceder en Internet a los espacios de nombres de Relay, siempre que la solicitud tenga una autenticación y una autorización válidas. Con el firewall de IP, puede restringirlo aún más a solo un conjunto de direcciones o intervalos de direcciones IPv4 en notación [CIDR (Enrutamiento de interdominios sin clases)](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing).

Esta característica es útil en escenarios en los que Azure Relay debe ser accesible únicamente desde ciertos sitios conocidos. Las reglas de firewall permiten configurar reglas para aceptar el tráfico procedente de direcciones IPv4 concretas. Por ejemplo, si usa Relay con [Azure Express Route](../expressroute/expressroute-faqs.md#supported-services), puede crear una **regla de firewall** para permitir el tráfico procedente única y exclusivamente de las direcciones IP de la infraestructura local. 

Las reglas de firewall de IP se aplican en el nivel del espacio de nombres de Relay. Por lo tanto, las reglas se aplican a todas las conexiones de clientes que usan cualquier protocolo admitido. Cualquier intento de conexión desde una dirección IP que no coincida con una regla IP admitida en el espacio de nombres de Relay se rechaza como no autorizado. La respuesta no menciona la regla IP. Las reglas de filtro IP se aplican en orden y la primera regla que coincida con la dirección IP determina la acción de aceptar o rechazar.

Para obtener más información, consulte [Configuración de reglas de firewall de IP para un espacio de nombres de Relay](ip-firewall-virtual-networks.md).

## <a name="private-endpoints"></a>Puntos de conexión privados

El **servicio Azure Private Link** le permite acceder a los servicios de Azure (por ejemplo, Azure Relay, Azure Service Bus, Azure Event Hubs, Azure Storage y Azure Cosmos DB) y a los servicios de asociados o clientes hospedados por Azure mediante un punto de conexión privado de la red virtual. Para más información, consulte [¿Qué es Azure Private Link?](../private-link/private-link-overview.md)

Un **punto de conexión privado** es una interfaz de red que permite que las cargas de trabajo que se ejecutan en una red virtual se conecten de forma privada y segura a un servicio que tiene un **recurso de vínculo privado** (por ejemplo, un espacio de nombres de Relay). El punto de conexión privado usa una dirección IP privada de la red virtual para incorporar el servicio de manera eficaz a su red virtual. Todo el tráfico dirigido al servicio se puede enrutar mediante el punto de conexión privado, por lo que no se necesitan puertas de enlace, dispositivos NAT, ExpressRoute, conexiones de VPN ni direcciones IP públicas. El tráfico entre la red virtual y el servicio atraviesa la red troncal de Microsoft y elimina la exposición a la red pública de Internet. Puede proporcionar un nivel de granularidad en el control de acceso si permite conexiones a espacios de nombres de Azure Relay específicos.

> [!NOTE]
> Esta funcionalidad actualmente está en su **versión preliminar**. 

Para obtener más información, consulte [Configuración de los puntos de conexión privados](private-link-service.md).


## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes:

- [Configuración de la dirección IP del firewall](ip-firewall-virtual-networks.md)
- [Configuración los puntos de conexión privados](private-link-service.md)
