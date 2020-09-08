---
title: 'Azure ExpressRoute: Acerca del cifrado'
description: Aprenda acerca del cifrado de ExpressRoute.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 05/05/2020
ms.author: duau
ms.openlocfilehash: 46f0a0e86c5db612f440bcf631329d2800251dab
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89397804"
---
# <a name="expressroute-encryption"></a>Cifrado de ExpressRoute
 
ExpressRoute admite un par de tecnologías de cifrado para garantizar la confidencialidad y la integridad de los datos que atraviesan su red y la red de Microsoft.

## <a name="point-to-point-encryption-by-macsec-faq"></a>Preguntas frecuentes sobre el cifrado punto a punto mediante MACsec
MACsec es un [estándar IEEE](https://1.ieee802.org/security/802-1ae/). Cifra los datos en el nivel de Media Access Control (MAC) o Network Layer 2. Puede usar MACsec para cifrar los vínculos físicos entre los dispositivos de red y los dispositivos de red de Microsoft cuando se conecte a Microsoft a través de [ExpressRoute Direct](expressroute-erdirect-about.md). De forma predeterminada, MACsec está deshabilitado en los puertos de ExpressRoute Direct. Traiga su propia clave de MACsec para el cifrado y almacénela en [Azure Key Vault](../key-vault/general/overview.md). Decida cuándo desea rotar la clave. Vea otras preguntas frecuentes a continuación.
### <a name="can-i-enable-macsec-on-my-expressroute-circuit-provisioned-by-an-expressroute-provider"></a>¿Puedo habilitar MACsec en mi circuito ExpressRoute aprovisionado por un proveedor de ExpressRoute?
No. MACsec cifra todo el tráfico de un vínculo físico con una clave que pertenece a una entidad (es decir, un cliente). Por tanto, solo está disponible en ExpressRoute Direct.
### <a name="can-i-encrypt-some-of-the-expressroute-circuits-on-my-expressroute-direct-ports-and-leave-other-circuits-on-the-same-ports-unencrypted"></a>¿Puedo cifrar algunos circuitos ExpressRoute en mis puertos de ExpressRoute Direct y dejar otros circuitos en los mismos puertos sin cifrar? 
No. Una vez que MACsec está habilitado, se cifra todo el tráfico de control de red, por ejemplo, el tráfico de datos de BGP y el tráfico de datos de cliente. 
### <a name="when-i-enabledisable-macsec-or-update-macsec-key-will-my-on-premises-network-lose-connectivity-to-microsoft-over-expressroute"></a>Al habilitar o deshabilitar MACsec o actualizar la clave de MACsec, ¿pierde mi red del entorno local la conectividad con Microsoft a través de ExpressRoute?
Sí. Para la configuración de MACsec, solo se admite el modo de clave previamente compartida. Significa que debe actualizar la clave en sus dispositivos y en los de Microsoft (a través de nuestra API). Este cambio no es atómico, por lo que perderá la conectividad cuando haya una discrepancia de claves entre los dos lados. Se recomienda encarecidamente programar una ventana de mantenimiento para el cambio de configuración. Para minimizar el tiempo de inactividad, sugerimos que actualice la configuración en un vínculo de ExpressRoute Direct a la vez después de cambiar el tráfico de red al otro vínculo.  
### <a name="will-traffic-continue-to-flow-if-theres-a-mismatch-in-macsec-key-between-my-devices-and-microsofts"></a>¿Continuará el flujo del tráfico si hay una discrepancia en la clave MACsec entre mis dispositivos y los de Microsoft?
No. Si MACsec se configura y se produce una discrepancia en la clave, se pierde la conectividad con Microsoft. En otras palabras, no se revertirá a una conexión no cifrada, con lo que se exponen los datos. 
### <a name="will-enabling-macsec-on-expressroute-direct-degrade-network-performance"></a>¿Al habilitarse MACsec en ExpressRoute Direct se degradará el rendimiento de la red?
El cifrado y el descifrado de MACsec se produce en el hardware de los enrutadores que usamos. No hay ningún efecto en el rendimiento por nuestra parte. Sin embargo, debe consultar al proveedor de la red los dispositivos que usa y comprobar si MACsec tiene alguna implicación en su rendimiento.
### <a name="which-cipher-suites-are-supported-for-encryption"></a>¿Qué conjuntos de cifrado se admiten para el cifrado?
Solo se admite la versión de [números de paquetes extendidos](https://1.ieee802.org/security/802-1aebw/) de AES128 y AES256. Además, deshabilite el [identificador de canal seguro (SCI)](https://en.wikipedia.org/wiki/IEEE_802.1AE) en la configuración MACsec del dispositivo. 

## <a name="end-to-end-encryption-by-ipsec-faq"></a>Preguntas frecuentes sobre el cifrado de un extremo a otro de IPsec
IPsec es un [estándar de IETF](https://tools.ietf.org/html/rfc6071). Cifra los datos en el nivel Protocolo de Internet (IP) o Capa 3 de red. Puede usar IPsec para cifrar una conexión de un extremo a otro entre la red local y la red virtual (VNET) en Azure. Vea otras preguntas frecuentes a continuación.
### <a name="can-i-enable-ipsec-in-addition-to-macsec-on-my-expressroute-direct-ports"></a>¿Puedo habilitar IPsec además de MACsec en mis puertos de ExpressRoute Direct?
Sí. MACsec protege las conexiones físicas entre el usuario y Microsoft. IPsec protege la conexión de un extremo a otro entre el usuario y las redes virtuales en Azure. Puede habilitarlos de forma independiente. 
### <a name="can-i-use-azure-vpn-gateway-to-set-up-the-ipsec-tunnel-between-my-on-premises-network-and-my-azure-virtual-network"></a>¿Puedo usar la puerta de enlace de Azure VPN para configurar el túnel IPsec entre mi red local y mi red virtual de Azure?
Sí. Puede configurar este túnel IPsec a través de Emparejamiento de Microsoft del circuito ExpressRoute. Siga nuestra [guía de configuración](site-to-site-vpn-over-microsoft-peering.md).
### <a name="can-i-use-azure-vpn-gateway-to-set-up-the-ipsec-tunnel-over-azure-private-peering"></a>¿Puedo usar la puerta de enlace de Azure VPN para configurar el túnel IPsec a través de Emparejamiento privado de Azure?
Si adopta Azure Virtual WAN, puede seguir [estos pasos](../virtual-wan/vpn-over-expressroute.md) para cifrar la conexión de un extremo a otro. Si tiene una VNET de Azure normal, puede implementar una puerta de enlace de VPN de terceros en la VNET y establecer un túnel IPsec entre ella y la puerta de enlace de VPN en el entorno local.
### <a name="what-is-the-throughput-i-will-get-after-enabling-ipsec-on-my-expressroute-connection"></a>¿Cuál es el rendimiento que se obtendría después de habilitar IPsec en mi conexión de ExpressRoute?
Si se usa la puerta de enlace de Azure VPN, compruebe las [cifras de rendimiento aquí](../vpn-gateway/vpn-gateway-about-vpngateways.md). Si se usa una puerta de enlace de VPN de terceros, consulte con el proveedor de las cifras de rendimiento.

## <a name="next-steps"></a>Pasos siguientes
Consulte [Configuración de MACsec](expressroute-howto-macsec.md) para más información sobre la configuración de MACsec.

Consulte [Configuración de IPsec](site-to-site-vpn-over-microsoft-peering.md) para más información sobre la configuración de IPsec.
