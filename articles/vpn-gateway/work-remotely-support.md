---
title: 'Trabajo remoto con P2S: Azure VPN Gateway'
description: En esta página se describe cómo puede sacar provecho de VPN Gateway para habilitar el trabajo remoto debido a la pandemia de la COVID-19.
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: conceptual
ms.date: 09/03/2020
ms.author: alzam
ms.openlocfilehash: a70f7ccd1c3ca078bab0ff473283b505c0ded316
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89441568"
---
# <a name="remote-work-using-azure-vpn-gateway-point-to-site"></a>Trabajo remoto de punto a sitio de Azure VPN Gateway

>[!NOTE]
>En este artículo se describe cómo puede aprovechar Azure VPN Gateway, Azure, la red de Microsoft y el ecosistema de asociados de Azure para trabajar de forma remota y mitigar los problemas de red que está experimentando debido a la crisis del COVID-19.
>

En este artículo se describen las opciones de que disponen las organizaciones para configurar el acceso remoto para sus usuarios o para complementar sus soluciones existentes con capacidad adicional durante la epidemia de la COVID-19.

La solución de punto a sitio de Azure se basa en la nube y se puede aprovisionar rápidamente para satisfacer el aumento de la demanda de usuarios para trabajar desde casa. Puede escalarse verticalmente y desactivarse de la manera más fácil y rápida cuando ya no se necesite la mayor capacidad.

## <a name="about-point-to-site-vpn"></a><a name="p2s"></a>Acerca de las conexiones VPN de punto a sitio

Una conexión de puerta de enlace de VPN de punto a sitio (P2S) permite crear una conexión segura a la red virtual desde un equipo cliente individual. Se establece una conexión de punto a sitio al iniciarla desde el equipo cliente. Esta solución resulta útil para los teletrabajadores que quieran conectarse a redes virtuales de Azure o a centros de datos locales desde una ubicación remota, por ejemplo, desde casa o un congreso. En este artículo se describe cómo permitir que los usuarios trabajen de forma remota en función de varios casos.

En la tabla siguiente se muestran los sistemas operativos cliente y las opciones de autenticación que están a su disposición. Sería útil seleccionar el método de autenticación en función del sistema operativo cliente que ya está en uso. Por ejemplo, seleccione OpenVPN con una autenticación basada en certificados si tiene una combinación de sistemas operativos cliente que necesitan conectarse. Tenga en cuenta también que la VPN de punto a sitio solo se admite en puertas de enlace de VPN basadas en rutas.

![point-to-site](./media/working-remotely-support/ostable.png "SO")

## <a name="scenario-1---users-need-access-to-resources-in-azure-only"></a><a name="scenario1"></a>Escenario 1: los usuarios solo necesitan acceder a los recursos de Azure

En este caso, los usuarios remotos solo necesitan obtener acceso a los recursos que se encuentran en Azure.

![point-to-site](./media/working-remotely-support/scenario1.png "Escenario 1")

En un nivel alto, se deben seguir estos pasos para que los usuarios puedan conectarse a los recursos de Azure de forma segura:

1. Cree una puerta de enlace de red virtual (solo si no existe).
2. Configure una conexión VPN de punto a sitio en la puerta de enlace.

   * Para la autenticación de certificados, siga [este vínculo](vpn-gateway-howto-point-to-site-resource-manager-portal.md#creategw).
   * Para OpenVPN, siga [este vínculo](vpn-gateway-howto-openvpn.md).
   * Para la autenticación de Azure AD, siga [este vínculo](openvpn-azure-ad-tenant.md).
   * Para solucionar los problemas de las conexiones de punto a sitio, siga [este vínculo](vpn-gateway-troubleshoot-vpn-point-to-site-connection-problems.md).
3. Descargue y distribuya la configuración del cliente VPN.
4. Distribuya los certificados (si se selecciona la autenticación de certificados) a los clientes.
5. Conéctese a una VPN de Azure.

## <a name="scenario-2---users-need-access-to-resources-in-azure-andor-on-prem-resources"></a><a name="scenario2"></a>Escenario 2: los usuarios necesitan acceder a los recursos de Azure o a recursos locales

En este caso, los usuarios remotos necesitan obtener acceso a los recursos que se encuentran en Azure y en los centros de datos locales.

![point-to-site](./media/working-remotely-support/scenario2.png "Escenario 2")

En un nivel alto, se deben seguir estos pasos para que los usuarios puedan conectarse a los recursos de Azure de forma segura:

1. Cree una puerta de enlace de red virtual (solo si no existe).
2. Configure una conexión VPN de punto a sitio en la puerta de enlace (consulte el [escenario 1](#scenario1)).
3. Configure un túnel de sitio a sitio en la puerta de enlace de red virtual de Azure con el protocolo BGP habilitado.
4. Configure un dispositivo local para conectarse a la puerta de enlace de red virtual de Azure.
5. Descargar el perfil de punto a sitio desde Azure Portal y distribuirlo a los clientes

Para aprender a configurar un túnel VPN de sitio a sitio, consulte [este vínculo](vpn-gateway-howto-site-to-site-resource-manager-portal.md).

## <a name="faq-for-native-azure-certificate-authentication"></a><a name="faqcert"></a>Preguntas más frecuentes acerca de la autenticación de certificado nativa de Azure

[!INCLUDE [vpn-gateway-point-to-site-faq-include](../../includes/vpn-gateway-faq-p2s-azurecert-include.md)]

## <a name="faq-for-radius-authentication"></a><a name="faqradius"></a>Preguntas más frecuentes acerca de la autenticación RADIUS

[!INCLUDE [vpn-gateway-point-to-site-faq-include](../../includes/vpn-gateway-faq-p2s-radius-include.md)]

## <a name="next-steps"></a>Pasos siguientes

* [Configure a P2S connection - Azure AD authentication](openvpn-azure-ad-tenant.md) (Configurar una conexión P2S: autenticación Azure AD)

* [Configure a P2S connection - RADIUS authentication](point-to-site-how-to-radius-ps.md) (Configurar una conexión P2S: autenticación RADIUS)

* [Configure a P2S connection - Azure native certificate authentication](vpn-gateway-howto-point-to-site-rm-ps.md) (Configurar una conexión P2S: autenticación de certificados nativos de Azure)

**"OpenVPN" es una marca comercial de OpenVPN Inc.**