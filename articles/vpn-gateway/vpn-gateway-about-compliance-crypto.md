---
title: 'Azure VPN Gateway: Requisitos criptográficos'
description: Aprenda a configurar las puertas de enlace de VPN de Azure para satisfacer los requisitos criptográficos para los túneles VPN de sitio a sitio entre entornos locales y las conexiones de red virtual a red virtual de Azure.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: article
ms.date: 01/10/2020
ms.author: yushwang
ms.openlocfilehash: 499d184d373b896ba7bffcf990693ef5e6ac466b
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88032431"
---
# <a name="about-cryptographic-requirements-and-azure-vpn-gateways"></a>Acerca de los requisitos criptográficos y las puertas de enlace de VPN de Azure

En este artículo se explica cómo configurar puertas de enlace de VPN de Azure para satisfacer los requisitos criptográficos para los túneles de VPN de sitio a sitio entre entornos y las conexiones entre redes virtuales dentro de Azure.

## <a name="about-ikev1-and-ikev2-for-azure-vpn-connections"></a>Acerca de IKEv1 e IKEv2 para conexiones VPN de Azure

Tradicionalmente permitíamos conexiones IKEv1 solo para SKU básicas y conexiones IKEv2 para todas las SKU de VPN Gateway que no fueran SKU básicas. Las SKU básicas solo permiten 1 conexión y, junto con otras limitaciones, como el rendimiento, los clientes que usaban dispositivos heredados que solo admiten protocolos IKEv1 tenían una experiencia limitada. Con el fin de mejorar la experiencia de los clientes que usan protocolos IKEv1, ahora se permiten conexiones IKEv1 para todas las SKU de VPN Gateway, excepto SKU de nivel Básico. Consulte [SKU de VPN Gateway](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-about-vpn-gateway-settings#gwsku) para más información.

![Conexiones IKEv1 e IKEv2 de Azure VPN Gateway](./media/vpn-gateway-about-compliance-crypto/ikev1-ikev2-connections.png)

Cuando las conexiones IKEv1 e IKEv2 se aplican a la misma instancia de VPN Gateway, el tránsito entre estas dos conexiones se habilita automáticamente.

## <a name="about-ipsec-and-ike-policy-parameters-for-azure-vpn-gateways"></a>Acerca de los parámetros de la directiva de IPsec e IKE para puertas de enlace de VPN de Azure

El protocolo IPsec e IKE estándar admite una gran variedad de algoritmos criptográficos en diversas combinaciones. Si no solicita una combinación específica de algoritmos y parámetros criptográficos, las instancias de Azure VPN Gateway usan un conjunto de propuestas predeterminadas. Los conjuntos de directivas predeterminados se eligieron para maximizar la interoperabilidad con una amplia gama de dispositivos VPN de terceros en las configuraciones predeterminadas. Como resultado, las directivas y el número de propuestas no pueden cubrir todas las posibles combinaciones de algoritmos criptográficos disponibles y puntos fuertes clave.

La directiva predeterminada establecida para Azure VPN Gateway se muestra en el artículo: [Acerca de los dispositivos VPN y los parámetros de IPsec/IKE para conexiones de VPN Gateway de sitio a sitio](vpn-gateway-about-vpn-devices.md).

## <a name="cryptographic-requirements"></a>Requisitos criptográficos

Para las comunicaciones que requieren determinados algoritmos o parámetros criptográficos, normalmente debido a requisitos de seguridad o de conformidad, ahora puede configurar las instancias de Azure VPN Gateway para usar una directiva personalizada de IPsec o IKE con determinados algoritmos criptográficos y severidades de clave, en lugar de conjuntos de directivas predeterminadas de Azure.

Por ejemplo, las directivas del modo principal de IKEv2 para las instancias de Azure VPN Gateway usan solo el Grupo Diffie-Hellman 2 (1024 bits), mientras que es posible que necesite especificar grupos más sólidos para su uso en IKE, por ejemplo, el Grupo 14 (2048 bits), el Grupo 24 (grupo de MODP de 2048 bits) o ECP (grupos de curva elíptica) de 256 o 384 bits (Grupo 19 y Grupo 20, respectivamente). También se aplican requisitos similares a las directivas de modo rápido de IPsec.

## <a name="custom-ipsecike-policy-with-azure-vpn-gateways"></a>Directiva personalizada de IPsec o IKE con puertas de enlace de VPN de Azure

Ahora las puertas de enlace de VPN de Azure admiten directivas de IPsec o IKE personalizadas y por conexión. Para una conexión entre sitios o entre redes virtuales puede elegir una combinación específica de algoritmos criptográficos para IPsec e IKE con el nivel de clave deseado, como se muestra en el ejemplo siguiente:

![ipsec-ike-policy](./media/vpn-gateway-about-compliance-crypto/ipsecikepolicy.png)

Puede crear una directiva de IPsec o IKE y aplicarla a una conexión nueva o existente.

### <a name="workflow"></a>Flujo de trabajo

1. Cree las redes virtuales, las puertas de enlace de VPN o las puertas de enlace de red local para su topología de conectividad, como se describe en otros documentos y procedimientos.
2. Cree una directiva IPsec/IKE.
3. Puede aplicar la directiva cuando se crea una conexión de S2S o entre redes virtuales
4. Si ya se ha creado la conexión, puede aplicar la directiva a una conexión existente o actualizarla.

## <a name="ipsecike-policy-faq"></a>P+F sobre directivas de IPsec o IKE

[!INCLUDE [vpn-gateway-ipsecikepolicy-faq-include](../../includes/vpn-gateway-faq-ipsecikepolicy-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Consulte [Configure IPsec/IKE policy](vpn-gateway-ipsecikepolicy-rm-powershell.md) (Configuración de la directiva de IPsec o IKE) para obtener instrucciones paso a paso acerca de cómo configurar directivas personalizadas de IPsec o IKE en una conexión.

Consulte también [Conexión de varios dispositivos VPN basados en directivas](vpn-gateway-connect-multiple-policybased-rm-ps.md) para obtener más información sobre la opción UsePolicyBasedTrafficSelectors.
