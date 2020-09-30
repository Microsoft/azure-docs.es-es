---
title: Acerca de las conexiones VPN de punto a sitio de Azure | VPN Gateway
description: Este artículo le ayudará a comprender las conexiones de punto a sitio y a decidir qué tipo de autenticación de puerta de enlace de VPN P2S utilizar.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 09/02/2020
ms.author: cherylmc
ms.openlocfilehash: 9ad0ad7b1ff2f0b706595daf8b8c93f480f0e00b
ms.sourcegitcommit: 9c262672c388440810464bb7f8bcc9a5c48fa326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89421720"
---
# <a name="about-point-to-site-vpn"></a>Acerca de las conexiones VPN de punto a sitio

Una conexión de puerta de enlace de VPN de punto a sitio (P2S) permite crear una conexión segura a la red virtual desde un equipo cliente individual. Se establece una conexión de punto a sitio al iniciarla desde el equipo cliente. Esta solución resulta útil para los teletrabajadores que deseen conectarse a redes virtuales de Azure desde una ubicación remota, por ejemplo, desde casa o un congreso. La conexión VPN de punto a sitio también es una solución útil en comparación con la conexión VPN de sitio a sitio cuando solo necesitan conectarse a la red virtual algunos clientes. Este artículo se aplica al modelo de implementación de Resource Manager.

## <a name="what-protocol-does-p2s-use"></a><a name="protocol"></a>¿Qué protocolo utiliza P2S?

La conexión VPN de punto a sitio usa uno de los siguientes protocolos:

* **Protocolo OpenVPN®**, un protocolo VPN basado en SSL/TLS. Una solución de VPN basada en TLS puede penetrar firewalls, puesto que la mayoría de ellos abre el puerto TCP 443 saliente, que usa TLS. OpenVPN puede utilizarse para la conexión desde dispositivos Android, iOS (11.0 y versiones posteriores), Windows, Linux y Mac (OSX 10.13 y versiones posteriores).

* El protocolo de túnel de sockets seguro (SSTP), que es un protocolo VPN propio basado en TLS. Una solución de VPN basada en TLS puede penetrar firewalls, puesto que la mayoría de ellos abre el puerto TCP 443 saliente, que usa TLS. El protocolo SSTP solo se admite en dispositivos Windows. Azure es compatible con todas las versiones de Windows que tienen SSTP (Windows 7 y versiones posteriores).

* La conexión VPN IKEv2, una solución de VPN con protocolo de seguridad de Internet basada en estándares. La conexión VPN IKEv2 puede utilizarse para la conexión desde dispositivos Mac (versión de OSX 10.11 y versiones posteriores).


>[!NOTE]
>IKEv2 y OpenVPN para P2S están disponibles para el modelo de implementación de Resource Manager. No están disponibles para el modelo de implementación clásico.
>

## <a name="how-are-p2s-vpn-clients-authenticated"></a><a name="authentication"></a>¿Cómo se autentican los clientes VPN de punto a sitio?

Para que Azure acepte una conexión VPN de punto a sitio, el usuario primero debe autenticarse. Azure ofrece dos mecanismos para autenticar los usuarios que se conectan.

### <a name="authenticate-using-native-azure-certificate-authentication"></a>Autenticación con certificados nativos de Azure

Con la autenticación con certificados nativos de Azure, para autenticar el usuario que se conecta, se usa un certificado de cliente presente en el dispositivo. Los certificados de cliente se generan a partir de un certificado raíz de confianza y se instalan en cada equipo cliente. Puede, o bien, usar un certificado raíz generado mediante una solución empresarial, o bien, generar un certificado autofirmado.

La validación del certificado de cliente se realiza mediante la puerta de enlace de VPN durante el establecimiento de la conexión VPN P2S. El certificado raíz es necesario para la validación y se debe cargar en Azure.

### <a name="authenticate-using-native-azure-active-directory-authentication"></a>Autenticación mediante Azure Active Directory de forma nativa

La autenticación mediante Azure AD permite a los usuarios conectarse a Azure con sus credenciales de Azure Active Directory. La autenticación de Azure AD nativa solo se admite para el protocolo OpenVPN y Windows 10, y requiere el uso del [Cliente VPN de Azure](https://go.microsoft.com/fwlink/?linkid=2117554).

Con la autenticación de Azure AD nativa, puede aprovechar el acceso condicional de Azure AD, así como las características de Multi-Factor Authentication (MFA) para VPN.

En un nivel alto, para configurar la autenticación de Azure AD es preciso seguir estos pasos:

1. [Configurar un inquilino de Azure AD](openvpn-azure-ad-tenant.md)

2. [Habilitar la autenticación de Azure AD en la puerta de enlace](openvpn-azure-ad-tenant.md#enable-authentication)

3. [Descargar y configurar el Cliente VPN de Azure](https://go.microsoft.com/fwlink/?linkid=2117554)


### <a name="authenticate-using-active-directory-ad-domain-server"></a>Autenticación mediante el servidor de dominio de Active Directory (AD)

La autenticación de dominio de AD permite a los usuarios conectarse a Azure con sus credenciales de dominio de la organización. Requiere un servidor RADIUS que se integre con el servidor de AD. Las organizaciones también pueden aprovechar las implementaciones existentes de RADIUS.
  
El servidor RADIUS puede implementarse de forma local o en la red virtual de Azure. Durante la autenticación, la puerta de enlace de VPN de Azure actúa como acceso directo y reenvía los mensajes de autenticación entre el servidor RADIUS y el dispositivo que se conecta. Por lo tanto, la disponibilidad de la puerta de enlace del servidor RADIUS es importante. Si el servidor RADIUS está presente de forma local, se necesita una conexión VPN S2S de Azure en el sitio local para la disponibilidad.  
  
El servidor RADIUS también se integra con los servicios de certificados de AD. Esto le permite usar el servidor RADIUS y la implementación de certificados de empresa para la autenticación de certificados de P2S como alternativa a la autenticación de certificados de Azure. La ventaja es que no es necesario cargar certificados raíz y revocados en Azure.

Los servidores RADIUS también se integran con otros sistemas de identidad externos. Esto abre una gran cantidad de opciones de autenticación para VPN P2S, incluidas las opciones de multifactor.

![point-to-site](./media/point-to-site-about/p2s.png "De punto a sitio")

## <a name="what-are-the-client-configuration-requirements"></a>¿Cuáles son los requisitos de configuración del cliente?

>[!NOTE]
>Para los clientes de Windows, debe tener derechos de administrador en el dispositivo de cliente para poder iniciar la conexión VPN desde el dispositivo cliente en Azure.
>

Los usuarios utilizan los clientes VPN nativos en los dispositivos Windows y Mac para P2S. Azure proporciona un archivo zip de configuración de cliente VPN con la configuración que necesitan estos clientes nativos para conectarse a Azure.

* Para los dispositivos Windows, la configuración de cliente VPN consta de un paquete de instalación que los usuarios instalan en sus dispositivos.
* Para los dispositivos Mac, consta del archivo mobileconfig que los usuarios instalan en sus dispositivos.

El archivo zip también proporciona los valores de algunas de las opciones de configuración importantes del lado de Azure que puede usar para crear su propio perfil para estos dispositivos. Algunos de los valores incluyen la dirección de puerta de enlace de VPN, los tipos de túnel configurados, las rutas y el certificado raíz para la validación de la puerta de enlace.

>[!NOTE]
>[!INCLUDE [TLS version changes](../../includes/vpn-gateway-tls-change.md)]
>

## <a name="which-gateway-skus-support-p2s-vpn"></a><a name="gwsku"></a>¿Qué SKU de puerta de enlace admite VPN de P2S?

[!INCLUDE [aggregate throughput sku](../../includes/vpn-gateway-table-gwtype-aggtput-include.md)]

* Para obtener recomendaciones de la SKU de puerta de enlace, consulte [Acerca de la configuración de VPN Gateway](vpn-gateway-about-vpn-gateway-settings.md#gwsku).

>[!NOTE]
>La SKU básica no admite la autenticación de IKEv2 o RADIUS.
>

## <a name="what-ikeipsec-policies-are-configured-on-vpn-gateways-for-p2s"></a><a name="IKE/IPsec policies"></a>¿Qué directivas IPsec/IKE se configuran en las puertas de enlace de VPN de P2S?


**IKEv2**

|**Cifrado** | **Integridad** | **PRF** | **Grupo DH** |
|---        | ---           | ---       | ---   |
|GCM_AES256 |   GCM_AES256  | SHA384    | GROUP_24 |
|GCM_AES256 |   GCM_AES256  | SHA384    | GROUP_14 |
|GCM_AES256 |   GCM_AES256  | SHA384    | GROUP_ECP384 |
|GCM_AES256 |   GCM_AES256  | SHA384    | GROUP_ECP256 |
|GCM_AES256 |   GCM_AES256  | SHA256    | GROUP_24 |
|GCM_AES256 |   GCM_AES256  | SHA256    | GROUP_14 |
|GCM_AES256 |   GCM_AES256  | SHA256    | GROUP_ECP384 |
|GCM_AES256 |   GCM_AES256  | SHA256    | GROUP_ECP256 |
|AES256     |   SHA384      | SHA384    | GROUP_24 |
|AES256     |   SHA384      | SHA384    | GROUP_14 |
|AES256     |   SHA384      | SHA384    | GROUP_ECP384 |
|AES256     |   SHA384      | SHA384    | GROUP_ECP256 |
|AES256     |   SHA256      | SHA256    | GROUP_24 |
|AES256     |   SHA256      | SHA256    | GROUP_14 |
|AES256     |   SHA256      | SHA256    | GROUP_ECP384 |
|AES256     |   SHA256      | SHA256    | GROUP_ECP256 |
|AES256     |   SHA256      | SHA256    | GROUP_2 |

**IPsec**

|**Cifrado** | **Integridad** | **Grupo PFS** |
|---        | ---           | ---       |
|GCM_AES256 | GCM_AES256 | GROUP_NONE |
|GCM_AES256 | GCM_AES256 | GROUP_24 |
|GCM_AES256 | GCM_AES256 | GROUP_14 |
|GCM_AES256 | GCM_AES256 | GROUP_ECP384 |
|GCM_AES256 | GCM_AES256 | GROUP_ECP256 |
| AES256    | SHA256 | GROUP_NONE |
| AES256    | SHA256 | GROUP_24 |
| AES256    | SHA256 | GROUP_14 |
| AES256    | SHA256 | GROUP_ECP384 |
| AES256    | SHA256 | GROUP_ECP256 |
| AES256    | SHA1 | GROUP_NONE |

## <a name="what-tls-policies-are-configured-on-vpn-gateways-for-p2s"></a><a name="TLS policies"></a>¿Qué directivas TLS se configuran en las puertas de enlace de VPN para P2S?
**TLS**

|**Directivas** |
|---| 
|TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256 |
|TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384 |
|TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 |
|TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 |
|TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256 |
|TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384 |
|TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256 |
|TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384 |
|TLS_RSA_WITH_AES_128_GCM_SHA256 |
|TLS_RSA_WITH_AES_256_GCM_SHA384 |
|TLS_RSA_WITH_AES_128_CBC_SHA256 |
|TLS_RSA_WITH_AES_256_CBC_SHA256 |

## <a name="how-do-i-configure-a-p2s-connection"></a><a name="configure"></a>¿Cómo se puede configurar una conexión de P2S?

En una configuración de P2S es necesario realizar unos pocos pasos específicos. En los siguientes artículos encontrará los pasos necesarios para realizar una configuración de P2S, así como vínculos para configurar los dispositivos de cliente VPN:

* [Configure a P2S connection - RADIUS authentication](point-to-site-how-to-radius-ps.md) (Configurar una conexión P2S: autenticación RADIUS)

* [Configure a P2S connection - Azure native certificate authentication](vpn-gateway-howto-point-to-site-rm-ps.md) (Configurar una conexión P2S: autenticación de certificados nativos de Azure)

* [Configuración de OpenVPN](vpn-gateway-howto-openvpn.md)

### <a name="to-remove-the-configuration-of-a-p2s-connection"></a>Procedimiento para quitar la configuración de una conexión P2S

Para conocer los pasos, consulte las [preguntas más frecuentes](#removeconfig) a continuación.
 
## <a name="faq-for-native-azure-certificate-authentication"></a><a name="faqcert"></a>Preguntas más frecuentes acerca de la autenticación de certificado nativa de Azure

[!INCLUDE [vpn-gateway-point-to-site-faq-include](../../includes/vpn-gateway-faq-p2s-azurecert-include.md)]

## <a name="faq-for-radius-authentication"></a><a name="faqradius"></a>Preguntas más frecuentes acerca de la autenticación RADIUS

[!INCLUDE [vpn-gateway-point-to-site-faq-include](../../includes/vpn-gateway-faq-p2s-radius-include.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Configure a P2S connection - RADIUS authentication](point-to-site-how-to-radius-ps.md) (Configurar una conexión P2S: autenticación RADIUS)

* [Configure a P2S connection - Azure native certificate authentication](vpn-gateway-howto-point-to-site-rm-ps.md) (Configurar una conexión P2S: autenticación de certificados nativos de Azure)

**"OpenVPN" es una marca comercial de OpenVPN Inc.**
