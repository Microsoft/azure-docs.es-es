---
title: 'Azure AD Connect: consideraciones al respecto de la identidad híbrida para la nube de Azure Government'
description: Consideraciones especiales para implementar Azure AD Connect con la nube de Azure Government.
services: active-directory
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 04/14/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 39acc0373f5748f57f4fef5a5cee5fb038071523
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88080188"
---
# <a name="hybrid-identity-considerations-for-the-azure-government-cloud"></a>Consideraciones al respecto de la identidad híbrida para la nube de Azure Government

En este artículo se describen las consideraciones para la integración de un entorno híbrido con la nube de Microsoft Azure Government. Esta información se proporciona como referencia para los administradores y arquitectos que trabajan con la nube de Azure Government.

> [!NOTE]
> Para integrar un entorno de Microsoft Azure Active Directory (Azure AD) local con la nube de Azure Government, debe actualizar a la versión más reciente de [Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594).

Para obtener una lista completa de los puntos de conexión del Departamento de Defensa del Gobierno de Estados Unidos, consulte la [documentación](https://docs.microsoft.com/office365/enterprise/office-365-u-s-government-dod-endpoints).

## <a name="azure-ad-pass-through-authentication"></a>Autenticación de paso a través de Azure AD

La siguiente información describe la implementación de la autenticación de paso a través (PTA) y la nube de Azure Government.

### <a name="allow-access-to-urls"></a>Permiso de acceso a direcciones URL

Antes de implementar el agente de autenticación de paso a través, compruebe si hay un firewall entre los servidores y Azure AD. Si su firewall o proxy permite programas bloqueados o seguros del Sistema de nombres de dominio, agregue las siguientes conexiones.

> [!NOTE]
> Las siguientes instrucciones también se aplican a la instalación del [conector de Application Proxy de Azure AD](https://aka.ms/whyappproxy) para entornos de Azure Government.

|URL |Cómo se usa|
|-----|-----|
|&#42;.msappproxy.us</br>&#42;.servicebus.usgovcloudapi.net|El agente utiliza estas direcciones URL para comunicarse con el servicio en la nube de Azure AD. |
|`mscrl.microsoft.us:80` </br>`crl.microsoft.us:80` </br>`ocsp.msocsp.us:80` </br>`www.microsoft.us:80`| El agente utiliza estas direcciones URL para verificar los certificados.|
|login.windows.us </br>secure.aadcdn.microsoftonline-p.com </br>&#42;.microsoftonline.us </br>&#42;.microsoftonline-p.us </br>&#42;.msauth.net </br>&#42;.msauthimages.net </br>&#42;.msecnd.net</br>&#42;.msftauth.net </br>&#42;.msftauthimages.net</br>&#42;.phonefactor.net </br>enterpriseregistration.windows.net</br>management.azure.com </br>policykeyservice.dc.ad.msft.net</br>ctldl.windowsupdate.us:80| El agente utiliza estas direcciones URL durante el proceso de registro.

### <a name="install-the-agent-for-the-azure-government-cloud"></a>Instalación del agente para la nube de Azure Government

Siga estos pasos para instalar el agente para la nube de Azure Government:

1. En el terminal de la línea de comandos, vaya a la carpeta que contiene el archivo ejecutable que instala el agente.
1. Ejecute el siguiente comando, que especifica que la instalación es para Azure Government.

   Para la autenticación de paso a través:

   ```
   AADConnectAuthAgentSetup.exe ENVIRONMENTNAME="AzureUSGovernment"
   ```

   Para el proxy de aplicación:

   ```
   AADApplicationProxyConnectorInstaller.exe ENVIRONMENTNAME="AzureUSGovernment" 
   ```

## <a name="single-sign-on"></a>Inicio de sesión único

### <a name="set-up-your-azure-ad-connect-server"></a>Configuración del servidor de Azure AD Connect

Si usa la autenticación de paso a través como método de inicio de sesión, no es necesaria ninguna otra comprobación de requisitos previos. Si usa la sincronización de hash de contraseña como método de inicio de sesión y hay un firewall entre Azure AD Connect y Azure AD, asegúrese de que:

- Usa Azure AD Connect, versión 1.1.644.0 o posterior.
- Si el firewall o proxy permite programas de DNS bloqueados o seguros, agregue las conexiones a las direcciones URL de &#42;.msappproxy.us a través del puerto 443.

  En caso contrario, permita el acceso a los intervalos de direcciones IP del centro de datos de Azure, que se actualizan cada semana. Este requisito solo es aplicable cuando se habilita la característica. No es necesario para los inicios de sesión de usuarios reales.

### <a name="roll-out-seamless-single-sign-on"></a>Implementación del inicio de sesión único de conexión directa

Puede implementar gradualmente el inicio de sesión único de conexión directa de Azure AD a los usuarios mediante las instrucciones siguientes. Para empezar, agregue la dirección URL de Azure AD `https://autologon.microsoft.us` a la configuración de la zona de la intranet de todos los usuarios o de los seleccionados mediante la directiva de grupo de Active Directory.

Además, debe habilitar una configuración de directiva de zona de intranet llamada **Permitir actualizaciones en la barra de estado a través de script mediante la directiva de grupo**.

## <a name="browser-considerations"></a>Consideraciones sobre el explorador

### <a name="mozilla-firefox-all-platforms"></a>Mozilla Firefox (todas las plataformas)

Mozilla Firefox no realiza automáticamente la autenticación Kerberos. Cada usuario debe agregar manualmente la dirección URL de Azure AD a su configuración de Firefox mediante estos pasos:

1. Ejecute Firefox y escriba  **about:config**  en la barra de direcciones. Descarte las notificaciones que vea.
1. Busque la preferencia  **network.negotiate-auth.trusted-uris** . Esta preferencia enumera los sitios de confianza de Firefox para la autenticación Kerberos.
1. Haga clic con el botón derecho en el nombre de la preferencia y luego seleccione  **Modificar**.
1. Escriba `https://autologon.microsoft.us` en el cuadro.
1. Seleccione  **Aceptar**  y vuelva a abrir el explorador.

### <a name="microsoft-edge-based-on-chromium-all-platforms"></a>Microsoft Edge basado en Chromium (todas las plataformas)

Si ha reemplazado la configuración de las directivas  `AuthNegotiateDelegateAllowlist` o`AuthServerAllowlist`  en el entorno, asegúrese de agregarle la dirección URL de Azure AD `https://autologon.microsoft.us`.

### <a name="google-chrome-all-platforms"></a>Google Chrome (todas las plataformas)

Si ha reemplazado la configuración de las directivas `AuthNegotiateDelegateWhitelist` o`AuthServerWhitelist`  en el entorno, asegúrese de agregarle la dirección URL de Azure AD `https://autologon.microsoft.us`.

## <a name="next-steps"></a>Pasos siguientes

- [Autenticación de paso a través](how-to-connect-pta-quick-start.md#step-1-check-the-prerequisites)
- [Inicio de sesión único](how-to-connect-sso-quick-start.md#step-1-check-the-prerequisites)
