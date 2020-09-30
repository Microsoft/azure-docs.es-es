---
title: Transición a OpenVPN o IKEv2 desde SSTP | Azure VPN Gateway
description: Este artículo le ayuda a comprender las maneras de superar el límite de 128 conexiones simultáneas de SSTP.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: alzam
ms.openlocfilehash: aec5a08bf56cd9a9ba18159ffc28a129163571b0
ms.sourcegitcommit: ac5cbef0706d9910a76e4c0841fdac3ef8ed2e82
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89426289"
---
# <a name="transition-to-openvpn-protocol-or-ikev2-from-sstp"></a>Transición al protocolo OpenVPN o IKEv2 desde SSTP

Una conexión de puerta de enlace de VPN de punto a sitio (P2S) permite crear una conexión segura a la red virtual desde un equipo cliente individual. Se establece una conexión de punto a sitio al iniciarla desde el equipo cliente. Este artículo se aplica al modelo de implementación del Administrador de recursos y en él se habla de distintas formas de superar el límite de 128 conexiones simultáneas de SSTP mediante la transición al protocolo OpenVPN o IKEv2.

## <a name="what-protocol-does-p2s-use"></a><a name="protocol"></a>¿Qué protocolo utiliza P2S?

La conexión VPN de punto a sitio usa uno de los siguientes protocolos:

* **Protocolo OpenVPN&reg;** , un protocolo VPN basado en SSL/TLS. Una solución de VPN basada en SSL puede penetrar firewalls, puesto que la mayoría de ellos abre el puerto TCP 443 saliente, que utiliza SSL. OpenVPN puede utilizarse para la conexión desde dispositivos Android, iOS (11.0 y versiones posteriores), Windows, Linux y Mac (OSX 10.13 y versiones posteriores).

* El **protocolo de túnel de sockets seguro (SSTP)** , que es un protocolo VPN propio basado en SSL. Una solución de VPN basada en SSL puede penetrar firewalls, puesto que la mayoría de ellos abre el puerto TCP 443 saliente, que utiliza SSL. El protocolo SSTP solo se admite en dispositivos Windows. Azure es compatible con todas las versiones de Windows que tienen SSTP (Windows 7 y versiones posteriores). **SSTP admite como máximo 128 conexiones simultáneas, con independencia de la SKU de puerta de enlace**.

* La conexión VPN IKEv2, una solución de VPN con protocolo de seguridad de Internet basada en estándares. La conexión VPN IKEv2 puede utilizarse para la conexión desde dispositivos Mac (versión de OSX 10.11 y versiones posteriores).


>[!NOTE]
>IKEv2 y OpenVPN para P2S están disponibles para el modelo de implementación de Resource Manager. No están disponibles para el modelo de implementación clásico. La SKU de puerta de enlace básica no es compatible con los protocolos IKEv2 ni OpenVPN. Si usa la SKU básica, tendrá que eliminar y volver a crear una SKU de producción para la puerta de enlace de red virtual.
>

## <a name="migrating-from-sstp-to-ikev2-or-openvpn"></a>Migración de SSTP a IKEv2 o a OpenVPN

Puede haber casos en los que quiera admitir más de 128 conexiones P2S simultáneas con una puerta de enlace VPN pero que usen SSTP. En tal caso, deberá pasarse al protocolo OpenVPN o IKEv2.

### <a name="option-1---add-ikev2-in-addition-to-sstp-on-the-gateway"></a>Opción 1: agregar IKEv2 además de SSTP en la puerta de enlace

Esta es la opción más sencilla. SSTP e IKEv2 pueden coexistir en la misma puerta de enlace y proporcionan un mayor número de conexiones simultáneas. Puede habilitar IKEv2 en la puerta de enlace existente y volver a descargar el cliente.

El hecho de agregar IKEv2 a una puerta de enlace VPN de SSTP existente no afectará a los clientes existentes y podrá configurarlos para usar IKEv2 en lotes pequeños o simplemente configurar los clientes nuevos para que usen IKEv2. Si un cliente de Windows está configurado para SSTP e IKEv2, intentará conectarse primero con IKEV2 y, si no lo consigue, usará SSTP.

**IKEv2 usa puertos UDP no estándar, por lo que deberá asegurarse de que estos puertos no estén bloqueados en el firewall del usuario. Los puertos en uso son el UDP 500 y el 4500.**

Para agregar IKEv2 a una puerta de enlace existente, solo tiene que ir a la pestaña "Configuración de punto a sitio" en la puerta de enlace de red virtual del portal y seleccionar **IKEv2 y SSTP (SSL)** en el cuadro desplegable.

![point-to-site](./media/ikev2-openvpn-from-sstp/sstptoikev2.png "IKEv2")


### <a name="option-2---remove-sstp-and-enable-openvpn-on-the-gateway"></a>Opción 2: quitar SSTP y habilitar OpenVPN en la puerta de enlace

Dado que SSTP y OpenVPN son protocolos basados en TLS, no pueden coexistir en la misma puerta de enlace. Si opta por pasarse de SSTP a OpenVPN, tendrá que deshabilitar SSTP y habilitar OpenVPN en la puerta de enlace. Esta operación hará que los clientes existentes pierdan la conectividad a la puerta de enlace VPN hasta que se haya configurado el perfil nuevo en el cliente.

Si quiere, puede habilitar OpenVPN junto con IKEv2. OpenVPN está basado en TLS y usa el puerto TCP 443 estándar. Para cambiar a OpenVPN, vaya a la pestaña "Configuración de punto a sitio" de la puerta de enlace de red virtual del portal y seleccione **OpenVPN (SSL)** o **IKEv2 y OpenVPN (SSL)** en el cuadro desplegable.

![point-to-site](./media/ikev2-openvpn-from-sstp/sstptoopenvpn.png "OpenVPN")

Una vez configurada la puerta de enlace, los clientes existentes no podrán conectarse hasta que haya [implementado y configurado los clientes de OpenVPN](https://docs.microsoft.com/azure/vpn-gateway/vpn-gateway-howto-openvpn-clients).

Si usa Windows 10, también puede usar el [cliente VPN de Azure para Windows](https://docs.microsoft.com/azure/vpn-gateway/openvpn-azure-ad-client#to-download-the-azure-vpn-client)


## <a name="frequently-asked-questions"></a>Preguntas más frecuentes
### <a name="what-are-the-client-configuration-requirements"></a>¿Cuáles son los requisitos de configuración del cliente?

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

### <a name="which-gateway-skus-support-p2s-vpn"></a><a name="gwsku"></a>¿Qué SKU de puerta de enlace admite VPN de P2S?

[!INCLUDE [aggregate throughput sku](../../includes/vpn-gateway-table-gwtype-aggtput-include.md)]

* Para obtener recomendaciones de la SKU de puerta de enlace, consulte [Acerca de la configuración de VPN Gateway](vpn-gateway-about-vpn-gateway-settings.md#gwsku).

>[!NOTE]
>La SKU básica no admite la autenticación de IKEv2 o RADIUS.
>

### <a name="what-ikeipsec-policies-are-configured-on-vpn-gateways-for-p2s"></a><a name="IKE/IPsec policies"></a>¿Qué directivas IPsec/IKE se configuran en las puertas de enlace de VPN de P2S?


**IKEv2**

|**Cifrado** | **Integridad** | **PRF** | **Grupo DH** |
|---        | ---            | ---        | ---     |
|GCM_AES256 |    GCM_AES256    | SHA384    | GROUP_24 |
|GCM_AES256 |    GCM_AES256    | SHA384    | GROUP_14 |
|GCM_AES256 |    GCM_AES256    | SHA384    | GROUP_ECP384 |
|GCM_AES256 |    GCM_AES256    | SHA384    | GROUP_ECP256 |
|GCM_AES256 |    GCM_AES256    | SHA256    | GROUP_24 |
|GCM_AES256 |    GCM_AES256    | SHA256    | GROUP_14 |
|GCM_AES256 |    GCM_AES256    | SHA256    | GROUP_ECP384 |
|GCM_AES256 |    GCM_AES256    | SHA256    | GROUP_ECP256 |
|AES256     |   SHA384        | SHA384    | GROUP_24 |
|AES256     |   SHA384        | SHA384    | GROUP_14 |
|AES256     |   SHA384        | SHA384    | GROUP_ECP384 |
|AES256     |   SHA384        | SHA384    | GROUP_ECP256 |
|AES256     |   SHA256        | SHA256    | GROUP_24 |
|AES256     |   SHA256        | SHA256    | GROUP_14 |
|AES256     |   SHA256        | SHA256    | GROUP_ECP384 |
|AES256     |   SHA256        | SHA256    | GROUP_ECP256 |
|AES256     |   SHA256        | SHA256    | GROUP_2 |

**IPsec**

|**Cifrado** | **Integridad** | **Grupo PFS** |
|---        | ---            | ---        |
|GCM_AES256    | GCM_AES256 | GROUP_NONE |
|GCM_AES256    | GCM_AES256 | GROUP_24 |
|GCM_AES256    | GCM_AES256 | GROUP_14 |
|GCM_AES256    | GCM_AES256 | GROUP_ECP384 |
|GCM_AES256    | GCM_AES256 | GROUP_ECP256 |
| AES256    | SHA256 | GROUP_NONE |
| AES256    | SHA256 | GROUP_24 |
| AES256    | SHA256 | GROUP_14 |
| AES256    | SHA256 | GROUP_ECP384 |
| AES256    | SHA256 | GROUP_ECP256 |
| AES256    | SHA1 | GROUP_NONE |

### <a name="what-tls-policies-are-configured-on-vpn-gateways-for-p2s"></a><a name="TLS policies"></a>¿Qué directivas TLS se configuran en las puertas de enlace de VPN para P2S?
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

### <a name="how-do-i-configure-a-p2s-connection"></a><a name="configure"></a>¿Cómo se puede configurar una conexión de P2S?

En una configuración de P2S es necesario realizar unos pocos pasos específicos. En los siguientes artículos encontrará los pasos necesarios para realizar una configuración de P2S, así como vínculos para configurar los dispositivos de cliente VPN:

* [Configure a P2S connection - RADIUS authentication](point-to-site-how-to-radius-ps.md) (Configurar una conexión P2S: autenticación RADIUS)

* [Configure a P2S connection - Azure native certificate authentication](vpn-gateway-howto-point-to-site-rm-ps.md) (Configurar una conexión P2S: autenticación de certificados nativos de Azure)

* [Configuración de OpenVPN](vpn-gateway-howto-openvpn.md)

## <a name="next-steps"></a>Pasos siguientes

* [Configure a P2S connection - RADIUS authentication](point-to-site-how-to-radius-ps.md) (Configurar una conexión P2S: autenticación RADIUS)

* [Configure a P2S connection - Azure native certificate authentication](vpn-gateway-howto-point-to-site-rm-ps.md) (Configurar una conexión P2S: autenticación de certificados nativos de Azure)

**"OpenVPN" es una marca comercial de OpenVPN Inc.**
