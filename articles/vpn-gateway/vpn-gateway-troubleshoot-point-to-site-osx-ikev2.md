---
title: 'Azure VPN Gateway: solución de problemas de las conexiones de punto a sitio: clientes de Mac OS X'
description: Aprenda a solucionar problemas de conectividad de punto a sitio de Mac OS X mediante el cliente VPN nativo e IKEv2.
services: vpn-gateway
author: kumudD
ms.service: vpn-gateway
ms.topic: troubleshooting
ms.date: 03/27/2018
ms.author: alzam
ms.openlocfilehash: 58250980081388b78ebb32fb01b84beb8d86c1c2
ms.sourcegitcommit: bfeae16fa5db56c1ec1fe75e0597d8194522b396
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88030672"
---
# <a name="troubleshoot-point-to-site-vpn-connections-from-mac-os-x-vpn-clients"></a>Solución de problemas de conexiones VPN de punto a sitio desde clientes de VPN de Mac OS X

Este artículo le ayuda a solucionar problemas de conectividad de punto a sitio de Mac OS X con el cliente VPN nativo e IKEv2. El cliente VPN de Mac para IKEv2 es muy básico y no permite muchas personalizaciones. Hay solo cuatro opciones de configuración que deben comprobarse:

* Dirección del servidor
* Id. remoto
* Id. local
* Configuración de autenticación
* Versión del SO (10.11 o posterior)


## <a name="troubleshoot-certificate-based-authentication"></a><a name="VPNClient"></a> Solución de problemas de autenticación basada en certificados
1. Compruebe la configuración del cliente de VPN. Vaya a la **Configuración de red** presionando Comando + Mayús y, a continuación, escriba "VPN" para comprobar la configuración del cliente de VPN. En la lista, haga clic en la entrada VPN que debe investigarse.

   ![Autenticación basada en certificados IKEv2](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/ikev2cert1.jpg)
2. Compruebe que la **dirección del servidor** es el FQDN completo e incluye cloudapp.net.
3. El **id. remoto** debe ser el mismo que la dirección del servidor (FQDN de puerta de enlace).
4. El **id. local** debe ser el mismo que el **asunto** del certificado de cliente.
5. Haga clic en **Configuración de autenticación** para abrir la página Configuración de autenticación.

   ![Configuración de autenticación](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/ikev2auth2.jpg)
6. Compruebe que la opción **Certificado** está seleccionada en la lista desplegable.
7. Haga clic en el botón **Seleccionar** y compruebe que se ha seleccionado el certificado correcto. Haga clic en **Aceptar** para guardar los cambios.

## <a name="troubleshoot-username-and-password-authentication"></a><a name="ikev2"></a>Solución de problemas de autenticación de nombre de usuario y contraseña

1. Compruebe la configuración del cliente de VPN. Vaya a la **Configuración de red** presionando Comando + Mayús y, a continuación, escriba "VPN" para comprobar la configuración del cliente de VPN. En la lista, haga clic en la entrada VPN que debe investigarse.

   ![Contraseña del nombre de usuario de IKEv2](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/ikev2user3.jpg)
2. Compruebe que la **dirección del servidor** es el FQDN completo e incluye cloudapp.net.
3. El **id. remoto** debe ser el mismo que la dirección del servidor (FQDN de puerta de enlace).
4. El **id. local** puede estar en blanco.
5. Haga clic en el botón **Configuración de autenticación** y compruebe que se ha seleccionado "Nombre de usuario" en la lista desplegable.

   ![Configuración de autenticación](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/ikev2auth4.png)
6. Compruebe que se han introducido las credenciales correctas.

## <a name="additional-steps"></a><a name="additional"></a>Pasos adicionales

Si ha llevado a cabo los pasos anteriores y todo está configurado correctamente, descargue [Wireshark](https://www.wireshark.org/#download) y realice una captura de paquetes.

1. Filtre por *isakmp* y examine los paquetes **IKE_SA**. Podrá consultar los detalles de la propuesta de asociación de seguridad en **Payload: Security Association**. 
2. Compruebe que el cliente y el servidor disponen de un conjunto común.

   ![paquete](./media/vpn-gateway-troubleshoot-point-to-site-osx-ikev2/packet5.jpg) 
  
3. Si no hay ninguna respuesta del servidor en los seguimientos de red, compruebe que ha habilitado el protocolo IKEv2 en la página de configuración de la puerta de enlace de Azure en el sitio web de Azure Portal.

## <a name="next-steps"></a>Pasos siguientes
Para obtener más ayuda, consulte el [Soporte técnico de Microsoft](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade).
