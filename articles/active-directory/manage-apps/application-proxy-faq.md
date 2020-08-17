---
title: Preguntas más frecuentes sobre Azure AD Application Proxy| Microsoft Docs
description: Obtenga respuestas a las preguntas más frecuentes (p+f) sobre el uso de Azure AD Application Proxy para publicar aplicaciones internas y locales en usuarios remotos.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: reference
ms.date: 07/23/2020
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: aa9a930195908671cc0e772fd9643dcbce9dbb1c
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87562419"
---
# <a name="active-directory-azure-ad-application-proxy-frequently-asked-questions"></a>Preguntas más frecuentes sobre Active Directory (Azure AD) Application Proxy

Esta página responde a las preguntas más frecuentes sobre Azure Active Directory (Azure AD) Application Proxy.

## <a name="enabling-azure-ad-application-proxy"></a>Habilitar Azure AD Application Proxy

### <a name="what-license-is-required-to-use-azure-ad-application-proxy"></a>¿Qué licencia se necesita para usar Azure AD Application Proxy?

Para usar Azure AD Application Proxy, debe tener una licencia de Azure AD Premium de tipo P1 o P2. Para obtener más información sobre licencias, consulte la [página de precios de Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/).

### <a name="why-is-the-enable-application-proxy-button-grayed-out"></a>¿Por qué el botón "Habilitar Application Proxy" está atenuado?

Asegúrese de que tiene al menos una licencia de Azure AD Premium de tipo P1 o P2 y un conector de Azure AD Application Proxy instalada. Después de instalar correctamente el primer conector, el servicio de Azure AD Application Proxy se habilitará automáticamente.

## <a name="connector-configuration"></a>Configuración del conector

### <a name="can-application-proxy-connector-services-run-in-a-different-user-context-than-the-default"></a>¿Los servicios del conector de Application Proxy se ejecutan en un contexto de usuario distinto del predeterminado?

No, este escenario no se admite. La configuración predeterminada es la siguiente:

- Conector de Microsoft AAD Application Proxy - WAPCSvc - Servicio de red
- Microsoft AAD Application Proxy Connector Updater - WAPCUpdaterSvc - NT Authority\System

### <a name="can-a-guest-user-with-the-global-administrator-or-the-application-administrator-role-register-the-connector-for-the-guest-tenant"></a>¿Puede un usuario invitado con el rol de administrador global o de administrador de aplicaciones registrar el conector para el inquilino (invitado)?

No, actualmente no es posible. El intento de registro siempre se realiza en el inquilino principal del usuario.

### <a name="my-back-end-application-is-hosted-on-multiple-web-servers-and-requires-user-session-persistence-stickiness-how-can-i-achieve-session-persistence"></a>La aplicación de back-end se hospeda en varios servidores web y requiere la persistencia de la sesión de usuario (permanencia). ¿Cómo puedo conseguir la persistencia de la sesión? 

Para ver más recomendaciones, consulte [Alta disponibilidad y equilibrio de carga de los conectores y las aplicaciones de Application Proxy](application-proxy-high-availability-load-balancing.md).

### <a name="is-tls-termination-tlshttps-inspection-or-acceleration-on-traffic-from-the-connector-servers-to-azure-supported"></a>¿Se admite la terminación TLS (inspección o aceleración de TLS/HTTPS) en el tráfico desde los servidores de conector a Azure?

El conector de Application Proxy realiza la autenticación basada en certificados en Azure. Asimismo, la terminación TLS (inspección o aceleración de TLS/HTTPS) interrumpe este método de autenticación y no se admite. El tráfico del conector a Azure debe omitir todos los dispositivos que realicen la terminación TLS.  

### <a name="is-tls-12-required-for-all-connections"></a>¿Se requiere TLS 1.2 para todas las conexiones?
Sí. Para proporcionar el mejor cifrado a nuestros clientes, el servicio Application Proxy solo permite el acceso a los protocolos TLS 1.2. Estos cambios se han implementado gradualmente y han entrado definitivamente en vigor el 31 de agosto de 2019. Asegúrese de que todas las combinaciones de cliente-servidor y explorador-servidor están actualizadas para usar TLS 1.2 para mantener la conexión al servicio Application Proxy. Entre ellos, los clientes que los usuarios utilizan para tener acceso a las aplicaciones publicadas a través de Application Proxy. Vea [Preparación para usar TLS 1.2 en Office 365](https://docs.microsoft.com/microsoft-365/compliance/prepare-tls-1.2-in-office-365) para obtener referencias y recursos útiles.

### <a name="can-i-place-a-forward-proxy-device-between-the-connector-servers-and-the-back-end-application-server"></a>¿Puedo colocar un dispositivo proxy de reenvío entre los servidores del conector y el servidor de aplicaciones back-end?
Sí, este escenario se admite a partir de la versión del conector 1.5.1526.0. Consulte [Trabajo con servidores proxy locales existentes](application-proxy-configure-connectors-with-proxy-servers.md).

### <a name="should-i-create-a-dedicated-account-to-register-the-connector-with-azure-ad-application-proxy"></a>¿Debo crear una cuenta dedicada para registrar el conector con Azure AD Application Proxy?

No hay ninguna razón para ello. Cualquier cuenta de administrador global o de administrador de la aplicación funcionará. Recuerde que las credenciales especificadas durante la instalación no se usan después del proceso de registro. En su lugar, se emite un certificado para el conector, que se usa para la autenticación a partir de ese punto.

### <a name="how-can-i-monitor-the-performance-of-the-azure-ad-application-proxy-connector"></a>¿Cómo se puede supervisar el rendimiento del conector de Azure AD Application Proxy?

Hay contadores del monitor de rendimiento que se instalan junto con el conector. Para verlos haga lo siguiente:  

1. Seleccione **Start** (Inicio), escriba "Perfmon" y presione ENTRAR.
2. Seleccione el **monitor de rendimiento**  y haga clic en el icono verde **+** .
3. Agregue los contadores del **conector de Microsoft AAD Application Proxy** que quiera supervisar.

### <a name="does-the-azure-ad-application-proxy-connector-have-to-be-on-the-same-subnet-as-the-resource"></a>¿El conector de Azure AD Application Proxy debe estar en la misma subred que el recurso?

No es necesario que el conector esté en la misma subred. Sin embargo, este necesita tener la resolución de nombres (DNS, archivo de hosts) en el recurso y la conectividad de red necesaria (enrutamiento al recurso, puertos abiertos en el recurso, etc.). Para obtener más información, consulte [Consideraciones sobre la topología de red al utilizar el Azure Active Directory Application Proxy](application-proxy-network-topology.md).

### <a name="what-versions-of-windows-server-can-i-install-a-connector-on"></a>¿En qué versiones de Windows Server se puede instalar un conector?
El proxy de aplicación requiere Windows Server 2012 R2 o versiones posteriores. Actualmente hay una limitación en HTTP2 para Windows Server 2019. Para usar correctamente el conector en Windows Server 2019, deberá agregar la siguiente clave del Registro y reiniciar el servidor:
    ```
    HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Internet Settings\WinHttp\EnableDefaultHttp2 (DWORD) Value: 0 
    ```


## <a name="application-configuration"></a>Configuración de aplicaciones

### <a name="what-is-the-length-of-the-default-and-long-back-end-timeout-can-the-timeout-be-extended"></a>¿Cuál es la duración del tiempo de expiración de back-end predeterminado y "largo"? ¿Se puede ampliar el tiempo de expiración?

La duración predeterminada es de 85 segundos. La duración "larga" es de 180 segundos. No se puede extender el límite del tiempo de expiración.

### <a name="how-do-i-change-the-landing-page-my-application-loads"></a>¿Cómo puedo cambiar la página de aterrizaje que carga la aplicación?

En la página de registros de aplicaciones puede cambiar la dirección URL de la página principal a la dirección URL externa que quiera ver en la página de aterrizaje. La página especificada se cargará cuando se inicie la aplicación desde Mis aplicaciones o desde el portal de Office 365. Para obtener los pasos de configuración, consulte [Establecer una página principal personalizada para aplicaciones publicadas mediante el Azure AD Application Proxy](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-configure-custom-home-page).

### <a name="can-only-iis-based-applications-be-published-what-about-web-applications-running-on-non-windows-web-servers-does-the-connector-have-to-be-installed-on-a-server-with-iis-installed"></a>¿Solo se pueden publicar aplicaciones basadas en IIS? ¿Qué ocurre con las aplicaciones web que se ejecutan en servidores web que no son de Windows? ¿El conector tiene que estar instalado en un servidor que tenga a su vez IIS instalado?

No, no hay ningún requisito de IIS para las aplicaciones que se publican. Puede publicar aplicaciones web que se ejecutan en servidores distintos de Windows Server. Sin embargo, es posible que no pueda usar la autenticación previa con un servidor que no sea de Windows Server, dependiendo de si el servidor web admite Negotiate (la autenticación de Kerberos). IIS no es necesario en el servidor en el que está instalado el conector.

### <a name="can-i-configure-application-proxy-to-add-the-hsts-header"></a>¿Puedo configurar Application Proxy para agregar el encabezado HSTS?
Application Proxy no agrega automáticamente el encabezado HTTP Strict-Transport-Security a las respuestas HTTPS, pero mantendrá el encabezado si está en la respuesta original enviada por la aplicación publicada. La incorporación de una configuración para habilitar esta funcionalidad está en el mapa de ruta. Si está interesado en una versión preliminar que permite agregar esto a las respuestas, póngase en contacto con aadapfeedback@microsoft.com para más información.

## <a name="integrated-windows-authentication"></a>Autenticación integrada de Windows

### <a name="when-should-i-use-the-principalsallowedtodelegatetoaccount-method-when-setting-up-kerberos-constrained-delegation-kcd"></a>¿Cuándo debo usar el método PrincipalsAllowedToDelegateToAccount al configurar la delegación restringida de Kerberos (KCD)?

El método PrincipalsAllowedToDelegateToAccount se usa cuando los servidores del conector están en un dominio diferente de la cuenta de servicio de la aplicación web. Requiere el uso de la delegación restringida basada en recursos.
Si los servidores del conector y la cuenta de servicio de la aplicación web se encuentran en el mismo dominio, puede usar los usuarios y equipos de Active Directory para configurar las opciones de delegación en cada una de las cuentas de equipo del conector, para que así puedan delegar en el SPN de destino.

Si los servidores del conector y la cuenta de servicio de la aplicación web están en dominios diferentes, se usará la delegación basada en recursos. Los permisos de delegación se configuran en el servidor web de destino y la cuenta de servicio de la aplicación web. Tenga en cuenta que este método de delegación restringida es relativamente nuevo. El método se presentó en Windows Server 2012, que admite la delegación entre dominios al permitir que el propietario del recurso (servicio web) controle el equipo y las cuentas de servicio que pueden delegar contenido en él. No hay ninguna interfaz de usuario que le ayude con esta configuración, por lo que tendrá que usar PowerShell.
Para obtener más información, consulte [Información sobre la delegación restringida de Kerberos con Application Proxy](https://aka.ms/kcdpaper).

### <a name="does-ntlm-authentication-work-with-azure-ad-application-proxy"></a>¿Funciona la autenticación NTLM con Azure AD Application Proxy?

No se puede usar la autenticación NTLM como método de autenticación previa o de inicio de sesión único. La autenticación NTLM solo se puede usar cuando se puede negociar directamente entre el cliente y la aplicación web publicada. El uso de la autenticación NTLM normalmente hace que aparezca un mensaje de inicio de sesión en el explorador.

## <a name="pass-through-authentication"></a>Autenticación de paso a través

### <a name="can-i-use-conditional-access-policies-for-applications-published-with-pass-through-authentication"></a>¿Puedo usar directivas de acceso condicional en aplicaciones publicadas con la autenticación de paso a través?

Las directivas de acceso condicional solo se aplican a los usuarios que hayan realizado correctamente una autenticación previa en Azure AD. La autenticación de paso a través no desencadena la autenticación Azure AD, por lo que no se pueden aplicar las directivas de acceso condicional. Si se usa la autenticación de paso a través, las directivas de MFA se deben implementar en el servidor local (si es posible) o se debe habilitar la autenticación previa con Azure AD Application Proxy.

### <a name="can-i-publish-a-web-application-with-client-certificate-authentication-requirement"></a>¿Se puede publicar una aplicación web con el requisito de autenticación de certificado de cliente?

No, este escenario no se admite porque Application Proxy finalizaría el tráfico TLS.  

## <a name="remote-desktop-gateway-publishing"></a>Publicación de la Puerta de enlace de Escritorio remoto

### <a name="how-can-i-publish-remote-desktop-gateway-over-azure-ad-application-proxy"></a>¿Cómo se puede publicar la Puerta de enlace de Escritorio remoto con Azure AD Application Proxy?

Consulte [Publicación del Escritorio Remoto con Azure AD Application Proxy](application-proxy-integrate-with-remote-desktop-services.md).

### <a name="can-i-use-kerberos-constrained-delegation-single-sign-on---windows-integrated-authentication-in-the-remote-desktop-gateway-publishing-scenario"></a>¿Puedo usar la delegación restringida de Kerberos (inicio de sesión único: autenticación integrada de Windows) en el escenario de publicación de la Puerta de enlace de Escritorio remoto?

No, este escenario no se admite.  

### <a name="my-users-dont-use-internet-explorer-11-and-the-pre-authentication-scenario-doesnt-work-for-them-is-this-expected"></a>Mis usuarios no usan Internet Explorer 11 y el escenario de autenticación previa no les funciona. ¿Es normal?

Sí, es normal. El escenario de autenticación previa requiere un control ActiveX, que no se admite en exploradores de terceros.

### <a name="is-the-remote-desktop-web-client-html5-supported"></a>¿Se admite el cliente web de Escritorio remoto (HTML5)?

Sí, este escenario se encuentra actualmente en versión preliminar pública. Consulte [Publicación del Escritorio Remoto con Azure AD Application Proxy](application-proxy-integrate-with-remote-desktop-services.md).

### <a name="after-i-configured-the-pre-authentication-scenario-i-realized-that-the-user-has-to-authenticate-twice-first-on-the-azure-ad-sign-in-form-and-then-on-the-rdweb-sign-in-form-is-this-expected-how-can-i-reduce-this-to-one-sign-in"></a>Después de configurar el escenario de autenticación previa, me di cuenta de que el usuario tiene que autenticarse dos veces: primero en el formulario de inicio de sesión de Azure AD y, a continuación, en el formulario de inicio de sesión de RDWeb. ¿Es normal? ¿Cómo puedo reducir esto a un único inicio de sesión?

Sí, es normal. Si el equipo del usuario está unido Azure AD, el usuario inicia sesión en Azure AD automáticamente. El usuario solo tiene que proporcionar sus credenciales en el formulario de inicio de sesión RDWeb.

## <a name="sharepoint-publishing"></a>Publicación de SharePoint

### <a name="how-can-i-publish-sharepoint-over-azure-ad-application-proxy"></a>¿Cómo se puede publicar SharePoint en Azure AD Application Proxy?

Consulte [Habilitar el acceso remoto a SharePoint con Azure AD Application Proxy](application-proxy-integrate-with-sharepoint-server.md).

### <a name="can-i-use-the-sharepoint-mobile-app-ios-android-to-access-a-published-sharepoint-server"></a>¿Puedo usar la aplicación móvil de SharePoint (iOS/Android) para tener acceso a un servidor de SharePoint publicado?

Actualmente, la [aplicación móvil de SharePoint](https://docs.microsoft.com/sharepoint/administration/supporting-the-sharepoint-mobile-apps-online-and-on-premises) no admite la autenticación previa con Azure Active Directory.

## <a name="active-directory-federation-services-ad-fs-publishing"></a>Publicación de Servicios de federación de Active Directory (AD FS) 

### <a name="can-i-use-azure-ad-application-proxy-as-ad-fs-proxy-like-web-application-proxy"></a>¿Puedo usar Azure AD Application Proxy como proxy de AD FS (esto es, como un proxy de aplicación web)?

No. Azure AD Application Proxy está diseñado para funcionar con Azure AD y no cumple los requisitos para funcionar como un proxy de AD FS.

## <a name="websocket"></a>WebSocket

### <a name="does-websocket-support-work-for-applications-other-than-qliksense"></a>¿Admite WebSocket el trabajo de aplicaciones distintas a QlikSense?

Actualmente, la compatibilidad con el protocolo de WebSocket todavía está en su versión preliminar pública y es posible que no funcione en otras aplicaciones. Algunos clientes han podido usar correctamente el protocolo WebSocket con otras aplicaciones. Si prueba ese tipo de escenarios, nos encantaría saber más acerca de sus resultados. Envíenos sus comentarios a aadapfeedback@microsoft.com.

Las características (EventLog, PowerShell y Servicios de Escritorio remoto) en Windows Admin Center (WAC) o el cliente web de Escritorio remoto (HTML5) no funcionan actualmente a través de Azure AD Application Proxy.

## <a name="link-translation"></a>Traducción de vínculos

### <a name="does-using-link-translation-affect-performance"></a>¿El uso de la traducción de vínculos afecta al rendimiento?

Sí. El uso de la traducción de vínculos afecta al rendimiento. El servicio Application Proxy escanea la aplicación en busca de enlaces codificados y los reemplaza con sus respectivas direcciones URL externas publicadas, antes de presentarlas al usuario. 

Para obtener el mejor rendimiento, se recomienda usar direcciones URL internas y externas idénticas mediante la configuración de los [dominios personalizados](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-configure-custom-domain). Si no es posible usar dominios personalizados, puede mejorar el rendimiento de la traducción de vínculos mediante la extensión del inicio de sesión seguro de Mis aplicaciones o el explorador Microsoft Edge en dispositivos móviles. Consulte [Redirección de los vínculos codificados de manera rígida para las aplicaciones publicadas con Azure AD Application Proxy](application-proxy-configure-hard-coded-link-translation.md).

## <a name="wildcards"></a>Caracteres comodín

### <a name="how-do-i-use-wildcards-to-publish-two-applications-with-the-same-custom-domain-name-but-with-different-protocols-one-for-http-and-one-for-https"></a>¿Cómo puedo usar caracteres comodín para publicar dos aplicaciones con el mismo nombre de dominio personalizado, pero con diferentes protocolos, uno para HTTP y otro para HTTPS?

Este escenario no se admite directamente. Pero tiene varias opciones para el mismo:

1. Puede publicar las direcciones URL HTTP y HTTPS como aplicaciones independientes con un carácter comodín, pero deberá asignarles un dominio personalizado diferente. Esta configuración funcionará porque tiene direcciones URL externas diferentes.

2. Puede publicar la dirección URL HTTPS a través de una aplicación con caracteres comodín. Publique las aplicaciones HTTP por separado mediante estos cmdlets de PowerShell de Application Proxy:
   - [Administración de aplicaciones de Application Proxy](https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0#application_proxy_application_management)
   - [Administración de conectores de Application Proxy](https://docs.microsoft.com/powershell/module/azuread/?view=azureadps-2.0#application_proxy_connector_management)
