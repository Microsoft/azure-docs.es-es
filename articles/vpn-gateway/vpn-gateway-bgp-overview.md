---
title: 'BGP y Azure VPN Gateway: Información general'
description: Obtenga información sobre el Protocolo de puerta de enlace de borde (BGP) en Azure VPN, el protocolo de Internet estándar para intercambiar información de enrutamiento y disponibilidad entre redes.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: article
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: 464d00cbeddbacd617b1d2c88f9e5f68cc5d996e
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89400881"
---
# <a name="about-bgp-with-azure-vpn-gateway"></a>Acerca de BGP con Azure VPN Gateway
En este artículo se proporciona información general de compatibilidad de BGP (Protocolo de puerta de enlace de borde) en Azure VPN Gateway.

BGP es el protocolo de enrutamiento estándar usado habitualmente en Internet para intercambiar información de enrutamiento y disponibilidad entre dos o más redes. Cuando se utiliza en el contexto de Azure Virtual Network, BGP permite que instancias de Azure VPN Gateway y los dispositivos VPN locales, denominados vecinos o pares BGP, intercambien "rutas" que comunicarán a ambas puertas de enlace la disponibilidad y la posibilidad de que dichos prefijos pasen a través de las puertas de enlace o los enrutadores implicados. BGP también puede permitir el enrutamiento del tránsito entre varias redes mediante la propagación de las rutas que una puerta de enlace de BGP aprende de un par BGP a todos los demás pares BGP. 

## <a name="why-use-bgp"></a><a name="why"></a>¿Por qué usar BGP?
BGP es una característica opcional que puede utilizar con puertas de enlace de VPN basadas en enrutamiento de Azure. También debe asegurarse de que los dispositivos VPN locales admiten BGP antes de habilitar la característica. Puede seguir usando las puertas de enlace de VPN de Azure y los dispositivos VPN locales sin BGP. Es el equivalente a utilizar rutas estáticas (sin BGP) *frente a* usar el enrutamiento dinámico con BGP entre sus redes y Azure.

Hay varias ventajas y nuevas funcionalidades con BGP:

### <a name="support-automatic-and-flexible-prefix-updates"></a><a name="prefix"></a>Uso de actualizaciones de prefijo flexible y automáticas
Con BGP, solo necesita declarar un prefijo mínimo para un par BGP específico a través del túnel VPN S2S de IPsec. Puede ser tan pequeño como un prefijo de host (/32) de la dirección IP del par BGP del dispositivo VPN local. Puede controlar que prefijos de red local desea anunciar a Azure para permitir el acceso de Azure Virtual Network.

También puede anunciar prefijos mayores que pueden incluir algunos de los prefijos de dirección de red virtual, como un espacio de direcciones IP privado grande (por ejemplo, 10.0.0.0/8). Tenga en cuenta que los prefijos no pueden ser idénticos a uno de sus prefijos de red virtual. Se rechazarán dichas rutas idénticas a sus prefijos de red virtual.

### <a name="support-multiple-tunnels-between-a-vnet-and-an-on-premises-site-with-automatic-failover-based-on-bgp"></a><a name="multitunnel"></a>Uso de varios túneles entre una red virtual y un sitio local con la conmutación por error automática basada en BGP
Puede establecer varias conexiones entre la red virtual de Azure y los dispositivos VPN locales en la misma ubicación. Esta funcionalidad proporciona varios túneles (rutas) entre las dos redes en una configuración activa-activa. Si uno de los túneles está desconectado, se retiran las rutas correspondientes a través de BGP y el tráfico se cambiará automáticamente a los túneles restantes.

El siguiente diagrama muestra un ejemplo sencillo de esta configuración de alta disponibilidad:

![Varias rutas de acceso activas](./media/vpn-gateway-bgp-overview/multiple-active-tunnels.png)

### <a name="support-transit-routing-between-your-on-premises-networks-and-multiple-azure-vnets"></a><a name="transitrouting"></a>Uso del enrutamiento de tránsito entre las redes locales y varias redes virtuales de Azure
BGP permite que varias puertas de enlace obtengan y propaguen los prefijos de diferentes redes, si están directa o indirectamente conectadas. Esto puede permitir el enrutamiento de tránsito con instancias de Azure VPN Gateway entre sitios locales o en varias instancias de Azure Virtual Network.

El siguiente diagrama muestra un ejemplo de una topología de múltiples saltos con varias rutas que pueden enrutar el tráfico entre las dos redes locales a través de las puertas de enlace de VPN de Azure en las redes de Microsoft:

![Tránsito de múltiples saltos](./media/vpn-gateway-bgp-overview/full-mesh-transit.png)

## <a name="bgp-faq"></a><a name="faq"></a>Preguntas más frecuentes de BGP
[!INCLUDE [vpn-gateway-faq-bgp-include](../../includes/vpn-gateway-faq-bgp-include.md)]

## <a name="next-steps"></a>Pasos siguientes
Consulte [Cómo configurar BGP en puertas de enlace de VPN de Azure mediante Azure Resource Manager y PowerShell](vpn-gateway-bgp-resource-manager-ps.md) para conocer los pasos necesarios para configurar BGP en las conexiones entre sistemas locales y de red virtual a red virtual.

