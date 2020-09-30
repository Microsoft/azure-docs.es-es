---
title: Opciones de inicio de sesión único en Azure AD
description: Conozca las opciones disponibles para el inicio de sesión único (SSO) en Azure Active Directory.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: conceptual
ms.date: 12/03/2019
ms.author: kenwith
ms.reviewer: arvindh, japere
ms.openlocfilehash: ab4d150d79b2c53ad23d892efacef6a848d97854
ms.sourcegitcommit: 4feb198becb7a6ff9e6b42be9185e07539022f17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89469820"
---
# <a name="single-sign-on-options-in-azure-ad"></a>Opciones de inicio de sesión único en Azure AD

El inicio de sesión único ofrece muchas ventajas con respecto a los métodos de inicio de sesión tradicionales.

- **Con el inicio de sesión único**, los usuarios inician sesión una vez con una cuenta para acceder a dispositivos unidos a dominio, recursos de la empresa, aplicaciones de software como servicio (SaaS) y aplicaciones web. Después de iniciar sesión, el usuario puede iniciar las aplicaciones desde el portal de Office 365 o desde Mis aplicaciones. Los administradores pueden centralizar la administración de cuentas de usuario y automáticamente agregar o quitar el acceso de usuario a aplicaciones basadas en la pertenencia a grupos.

- **Sin el inicio de sesión único**, los usuarios deben recordar las contraseñas específicas de las aplicaciones e iniciar sesión en cada aplicación. El personal de TI necesita crear y actualizar las cuentas de usuario para cada aplicación como, por ejemplo, Office 365, Box y Salesforce. Los usuarios tienen que recordar sus contraseñas, además de dedicar tiempo a iniciar sesión en cada aplicación.

Para más información sobre el inicio de sesión único, consulte [¿Qué es el inicio de sesión único?](what-is-single-sign-on.md).

## <a name="choosing-a-single-sign-on-method"></a>Elección de un método de inicio de sesión único

Hay varias maneras de configurar una aplicación para el inicio de sesión único. La elección de un método de inicio de sesión único depende de cómo esté configurada la aplicación para la autenticación.

- Las aplicaciones en la nube pueden usar los métodos OpenID Connect, OAuth, SAML, basado en contraseñas, vinculado o deshabilitado para realizar el inicio de sesión único. 
- Las aplicaciones locales pueden usar métodos de inicio de sesión único basados en contraseñas, de autenticación integrada, basados en encabezados, vinculados o deshabilitados. Las opciones locales funcionan si las aplicaciones están configuradas para el proxy de aplicación.

Este diagrama de flujo le ayuda a decidir qué método de inicio de sesión único es el mejor en su caso.

![Diagrama de flujo de decisiones sobre el método de inicio de sesión único](./media/what-is-single-sign-on/choose-single-sign-on-method-040419.png)

En la tabla siguiente se resumen los métodos de inicio de sesión únicos y los vínculos para obtener más información.

| Método de inicio de sesión único | Tipos de aplicación | Cuándo se usa |
| :------ | :------- | :----- |
| [OpenID Connect y OAuth](#openid-connect-and-oauth) | En la nube y locales | OpenID Connect y OAuth se usan cuando se desarrollan aplicaciones nuevas. Este protocolo simplifica la configuración de la aplicación, tiene SDK fáciles de usar y permite que la aplicación use MS Graph.
| [SAML](#saml-sso) | En la nube y locales | Elija SAML siempre que sea posible para las aplicaciones existentes que no utilizan OpenID Connect o OAuth. SAML funciona con las aplicaciones realizan la autenticación mediante uno de los protocolos SAML.|
| [Basado en contraseñas](#password-based-sso) | En la nube y locales | Elija el método basado en contraseña cuando la aplicación se autentique con nombre de usuario y contraseña. El inicio de sesión único basado en contraseña permite el almacenamiento seguro de contraseñas de las aplicaciones y la reproducción mediante una extensión de explorador web o aplicación móvil. Este método usa el proceso de inicio de sesión existente proporcionado por la aplicación, pero permite que un administrador administre las contraseñas. |
| [Vinculado](#linked-sign-on) | En la nube y locales | Elija el inicio de sesión vinculado si la aplicación está configurada para el inicio de sesión único en otro servicio de proveedor de identidades. Esta opción no agrega el inicio de sesión único a la aplicación. No obstante, es posible que ya se haya implementado el inicio de sesión único en la aplicación mediante otro servicio, como los Servicios de federación de Active Directory.|
| [Deshabilitada](#disabled-sso) | En la nube y locales | Elija un inicio de sesión único deshabilitado si la aplicación no está lista para configurarse para el inicio de sesión único. Este modo es la opción predeterminada cuando se crea la aplicación.|
| [Autenticación integrada de Windows (IWA)](#integrated-windows-authentication-iwa-sso) | Solo en entornos locales | Elija el inicio de sesión único IWA para aplicaciones que usen la [autenticación integrada de Windows (IWA)](/aspnet/web-api/overview/security/integrated-windows-authentication) o aplicaciones compatibles con notificaciones. Para la autenticación integrada de Windows, los conectores del proxy de aplicación utilizan la delegación restringida de Kerberos (KCD) para autenticar a los usuarios en la aplicación. |
| [Basado en encabezados](#header-based-sso) | Solo en entornos locales | Use el inicio de sesión único basado en encabezados si la aplicación usa encabezados para la autenticación. El inicio de sesión único basado en encabezados requiere PingAccess para Azure AD. El proxy de aplicación usa Azure AD para autenticar al usuario y, a continuación, pasa el tráfico a través del servicio de conector.  |

## <a name="openid-connect-and-oauth"></a>OpenID Connect y OAuth

Al desarrollar nuevas aplicaciones, use protocolos modernos como OpenID Connect y OAuth para lograr la mejor experiencia de inicio de sesión única para la aplicación en varias plataformas de dispositivo. OAuth permite a los usuarios o administradores [conceder consentimiento](configure-user-consent.md) para recursos protegidos como [Microsoft Graph](/graph/overview). Se proporciona un [SDK](../develop/reference-v2-libraries.md) de fácil adopción para la aplicación y, además, la aplicación estará lista para usar [Microsoft Graph](/graph/overview).

Para más información, consulte:

- [OAuth 2.0](../develop/v2-oauth2-auth-code-flow.md)
- [OpenID Connect 1.0](../develop/v2-protocols-oidc.md)
- [Guía para desarrolladores de la plataforma de identidad de Microsoft](https://docs.microsoft.com/azure/active-directory/develop/active-directory-developers-guide).

## <a name="saml-sso"></a>SSO de SAML

Con **el inicio de sesión único de SAML**, Azure AD se autentica en la aplicación mediante el uso de la cuenta de Azure AD del usuario. Azure AD comunica la información de inicio de sesión a la aplicación a través de un protocolo de conexión. Con el inicio de sesión único basado en SAML puede asignar usuarios a roles de aplicación específicos según las reglas que defina en las notificaciones SAML.

Elija el inicio de sesión único basado en SAML cuando la aplicación lo admita.

El inicio de sesión único basado en SAML es compatible con aplicaciones que usan cualquiera de estos protocolos:

- SAML 2.0
- El certificado del proveedor de identidades de WS-Federation

Para configurar una aplicación SaaS para el inicio de sesión único basado en SAML, consulte [Configuración del inicio de sesión único basado en SAML](configure-saml-single-sign-on.md). Además, muchas aplicaciones de software como servicio (SaaS) tienen un [tutorial específico de la aplicación](../saas-apps/tutorial-list.md) que le guía por la configuración del inicio de sesión único basado en SAML.

Para configurar una aplicación para WS-Federation, siga las mismas instrucciones que para configurar la aplicación para el inicio de sesión único basado en SAML. En el paso para configurar la aplicación para usar Azure AD, tendrá que reemplazar la dirección URL de inicio de sesión de Azure AD para el punto de conexión de WS-Federation `https://login.microsoftonline.com/<tenant-ID>/wsfed`.

Para configurar una aplicación local para el inicio de sesión único basado en SAML, consulte [Inicio de sesión único en SAML para aplicaciones locales con Application Proxy](application-proxy-configure-single-sign-on-on-premises-apps.md).

Para más información acerca del protocolo SAML, consulte [Protocolo SAML de inicio de sesión único](../develop/single-sign-on-saml-protocol.md).

## <a name="password-based-sso"></a>Inicio de sesión único basado en contraseñas

Con el inicio de sesión con contraseña, los usuarios inician sesión con nombre de usuario y contraseña la primera vez que acceden a ella. Después del primer inicio de sesión, Azure AD proporciona el nombre de usuario y la contraseña a la aplicación.

El inicio de sesión único basado en contraseñas usa el proceso de autenticación existente que proporciona la aplicación. Cuando se habilita el inicio de sesión único con contraseña para una aplicación, Azure AD recopila y almacena de forma segura los nombres de usuario y contraseñas de la aplicación. Las credenciales de usuario se almacenan en un estado cifrado en el directorio.

Elija el inicio de sesión único basado en contraseñas si:

- Una aplicación no admite el protocolo de inicio de sesión único de SAML.
- Una aplicación se autentica con un nombre de usuario y una contraseña en lugar de con tokens de acceso y encabezados.

>[!NOTE]
>No se pueden aplicar directivas de acceso condicional ni la autenticación multifactor para el inicio de sesión único basado en contraseña.

Se admite el inicio de sesión único basado en contraseñas para cualquier aplicación basada en la nube cuya página de inicio de sesión esté basada en HTML. Puede utilizar cualquiera de los siguientes exploradores:

- Internet Explorer 11 en Windows 7 o posterior
   > [!NOTE]
   > Internet Explorer se encuentra en un periodo de compatibilidad limitada y ya no recibe nuevas actualizaciones de software. Microsoft Edge es el explorador recomendado.

- Microsoft Edge en Windows 10 Anniversary Edition o versión posterior
- Microsoft Edge para iOS y Android
- Explorador administrado con Intune
- Chrome en Windows 7 o versiones posteriores y en MacOS X o versiones posteriores.
- Firefox 26.0 o versiones posteriores: en Windows XP SP2 o versiones posteriores y en Mac OS X 10.6 o versiones posteriores.

Para configurar una aplicación en la nube para el inicio de sesión único basado en contraseñas, consulte [Configuración del inicio de sesión único con contraseña](configure-password-single-sign-on-non-gallery-applications.md).

Para configurar una aplicación local para el inicio de sesión único a través del proxy de aplicación, consulte [Almacén de contraseñas para el inicio de sesión único con el proxy de aplicación](application-proxy-configure-single-sign-on-password-vaulting.md)

### <a name="how-authentication-works-for-password-based-sso"></a>Funcionamiento de la autenticación en el inicio de sesión único mediante contraseña

Para autenticar un usuario en una aplicación, Azure AD recupera sus credenciales del directorio y las especifica en la página de inicio de sesión de la aplicación.  Azure AD pasa de forma segura las credenciales del usuario a través de una extensión de explorador web o una aplicación móvil. Este proceso permite que un administrador administre las credenciales de usuario y no requiere que los usuarios recuerden sus contraseñas.

> [!IMPORTANT]
> Durante el proceso de inicio de sesión automático el usuario no ve las credenciales. Sin embargo, las credenciales son detectables mediante el uso de herramientas de depuración web. Los usuarios y administradores deben seguir las mismas directivas de seguridad como si el usuario especificara directamente las credenciales.

### <a name="managing-credentials-for-password-based-sso"></a>Administración de credenciales para el inicio de sesión único basado en contraseñas

El administrador de Azure AD o los usuarios pueden administrar las contraseñas de cada aplicación.

Si el administrador de Azure AD administra las credenciales:  

- El usuario no necesita restablecer ni recordar el nombre de usuario y la contraseña. El usuario puede acceder a la aplicación si hace clic en ella en Mis aplicaciones o a través del vínculo que se proporcione.
- El administrador puede realizar tareas de administración de las credenciales. Por ejemplo, el administrador puede actualizar el acceso a las aplicaciones según la pertenencia a grupos de usuarios y el estado de los empleados.
- El administrador puede usar credenciales administrativas para proporcionar acceso a las aplicaciones que comparten muchos usuarios. Por ejemplo, el administrador puede permitir que todos los usuarios que pueden acceder a una aplicación tengan acceso a una red social o aplicación que comparta documentos.

Si el usuario final administra las credenciales:

- Los usuarios pueden administrar sus contraseñas, actualizándolas o eliminándolas según sea necesario.
- Los administradores pueden seguir estableciendo nuevas credenciales para la aplicación.

## <a name="linked-sign-on"></a>Inicio de sesión vinculado
El inicio de sesión único vinculado permite a Azure AD proporcionar inicio de sesión único a una aplicación que ya está configurada para el inicio de sesión único en otro servicio. La aplicación vinculada puede aparecer a los usuarios finales en el portal de Office 365 o el portal Mis aplicaciones de Azure AD. Por ejemplo, un usuario puede iniciar una aplicación que está configurada para el inicio de sesión único en Servicios de federación de Active Directory (AD FS) 2.0 desde el portal de Office 365. También hay informes adicionales disponibles para aplicaciones vinculadas que se inician desde el portal de Office 365 o el portal Mis aplicaciones de Azure AD. Para configurar una aplicación para el inicio de sesión vinculado, consulte [Configuración del inicio de sesión vinculado](configure-linked-sign-on.md).

### <a name="linked-sign-on-for-application-migration"></a>Inicio de sesión vinculado para la migración de aplicaciones

El inicio de sesión vinculado puede proporcionar una experiencia de usuario coherente durante la migración de aplicaciones durante un período de tiempo. Si va a migrar aplicaciones a Azure Active Directory, puede usar un inicio de sesión vinculado para publicar rápidamente los vínculos a todas las aplicaciones que pretende migrar.  Los usuarios pueden encontrar todos los vínculos en el [portal Mis aplicaciones](../user-help/active-directory-saas-access-panel-introduction.md) o en el [iniciador de aplicaciones de Office 365](https://support.office.com/article/meet-the-office-365-app-launcher-79f12104-6fed-442f-96a0-eb089a3f476a). Los usuarios no saben que están accediendo a una aplicación vinculada o una aplicación migrada.  

Una vez que el usuario se ha autenticado con una aplicación vinculada, se debe crear un registro de cuenta antes de proporcionar al usuario final el acceso de inicio de sesión único. El aprovisionamiento de este registro de cuenta puede producirse automáticamente o bien lo puede realizar manualmente un administrador.

>[!NOTE]
>No se pueden aplicar directivas de acceso condicional ni la autenticación multifactor a una aplicación vinculada. Esto se debe a que las aplicaciones vinculadas no proporcionan funcionalidades de inicio de sesión único mediante Azure AD. Al configurar una aplicación vinculada, simplemente se agrega un vínculo que aparecerá en el iniciador de aplicaciones o en el portal Aplicaciones. 

## <a name="disabled-sso"></a>Inicio de sesión único deshabilitado

El modo deshabilitado significa que el inicio de sesión único no se usa para la aplicación. Si el inicio de sesión único está deshabilitado, es posible que los usuarios tengan que autenticarse dos veces. Primero, los usuarios se autentican en Azure AD y, posteriormente, inician sesión en la aplicación.

Use el modo de inicio de sesión único deshabilitado en estos casos:

- Si no está preparado para integrar esta aplicación con el inicio de sesión único de Azure AD, o
- Si va a probar otros aspectos de la aplicación, o
- Como una capa de seguridad en una aplicación local que no requiere que los usuarios se autentiquen. Con el modo deshabilitado, el usuario debe autenticarse.

Tenga en cuenta que si ha configurado la aplicación para el inicio de sesión único basado en SAML iniciado por SP y cambia el modo SSO para deshabilitarlo, no impedirá que los usuarios firmen a la aplicación fuera del portal Mis aplicaciones. Para ello, debe [deshabilitar la capacidad de inicio de sesión de los usuarios](disable-user-sign-in-portal.md).

## <a name="integrated-windows-authentication-iwa-sso"></a>Inicio de sesión único con autenticación integrada de Windows (IWA)

El [proxy de la aplicación](/aspnet/web-api/overview/security/integrated-windows-authentication) proporciona el inicio de sesión único (SSO) a las aplicaciones que usan la [autenticación integrada de Windows (IWA)](application-proxy.md), o bien a las aplicaciones compatibles con notificaciones. Si la aplicación utiliza IWA, el proxy de la aplicación se autentica en la aplicación mediante la delegación restringida de Kerberos (KCD). Para una aplicación compatible con notificaciones que confía en Azure Active Directory, el inicio de sesión único funciona porque el usuario ya se ha autenticado con Azure AD.

Elija el modo de inicio de sesión único de autenticación integrada para proporcionar inicio de sesión único a una aplicación local que se autentica con IWA.

Para configurar una aplicación local para IWA, consulte [Delegación restringida de Kerberos para el inicio de sesión único para las aplicaciones con Proxy de aplicación](application-proxy-configure-single-sign-on-with-kcd.md).

### <a name="how-single-sign-on-with-kcd-works"></a>Cómo funciona el inicio de sesión único con KCD
En este diagrama se explica el flujo cuando un usuario accede a una aplicación local que usa IWA.

![Diagrama de flujos de autenticación de Microsoft Azure AD](./media/application-proxy-configure-single-sign-on-with-kcd/AuthDiagram.png)

1. El usuario escribe la dirección URL para tener acceso a la aplicación local mediante Application Proxy.
1. Proxy de aplicación redirige la solicitud a los servicios de autenticación de Azure AD para realizar la autenticación previa. En este momento, Azure AD aplica cualquier autenticación correspondiente, así como directivas de autorización, como la autenticación multifactor. Si se valida el usuario, Azure AD crea un token y lo envía al usuario.
1. El usuario pasa el token a Proxy de aplicación.
1. El proxy de aplicación valida el token y recupera el nombre principal de usuario (UPN) del token. A continuación, envía la solicitud, el nombre principal de usuario y el nombre de entidad de seguridad de servicio (SPN) al conector mediante un canal seguro con autenticación dual.
1. El conector usa la negociación de la delegación restringida de Kerberos (KCD) con el AD local, suplantando al usuario para obtener un token de Kerberos para la aplicación.
1. Active Directory envía el token de Kerberos para la aplicación al conector.
1. El conector envía la solicitud original al servidor de aplicaciones, con el token de Kerberos que recibió de AD.
1. La aplicación envía la respuesta al conector y, después, se devuelve al servicio del proxy de aplicación y, por último, al usuario.

## <a name="header-based-sso"></a>Inicio de sesión único basado en encabezados

El inicio de sesión único basado en encabezados funciona con aquellas aplicaciones que usan encabezados para la autenticación. Este método de inicio de sesión usa un servicio de autenticación de terceros denominado PingAccess. Un usuario solo necesita autenticarse en Azure AD.

Elija el inicio de sesión único basado en encabezados cuando el proxy de aplicación y PingAccess estén configurados para la aplicación.

Para configurar la autenticación basada en encabezados, consulte [Autenticación basada en el encabezado para el inicio de sesión único con el proxy de aplicación](application-proxy-configure-single-sign-on-with-ping-access.md).

### <a name="what-is-pingaccess-for-azure-ad"></a>¿Qué es PingAccess para Azure AD?

Mediante el uso de PingAccess para Azure AD los usuarios pueden acceder y realizar un inicio de sesión único en las aplicaciones que usan encabezados para la autenticación. El proxy de aplicación trata estas aplicaciones como a las demás, usa Azure AD para autenticar el acceso y, después, pasa el tráfico a través del servicio de conector. Después de que se produce la autenticación, el servicio PingAccess traslada el token de acceso de Azure AD a un formato de encabezado que se envía a la aplicación.

Los usuarios no notarán ninguna diferencia al iniciar sesión para usar las aplicaciones corporativas. Podrán seguir trabajando desde cualquier lugar y en cualquier dispositivo. Los conectores del proxy de aplicación dirigen el tráfico remoto a todas las aplicaciones y seguirán equilibrando la carga de forma automática.

### <a name="how-do-i-get-a-license-for-pingaccess"></a>¿Cómo se puede obtener una licencia de PingAccess?

Dado que este escenario se ofrece a través de una asociación entre Azure AD y PingAccess, se necesitarán licencias de ambos servicios. Sin embargo, las suscripciones Azure AD Premium incluyen una licencia básica de PingAccess que abarca hasta 20 aplicaciones. Si tiene que publicar más de 20 aplicaciones basadas en encabezados, puede adquirir una licencia adicional de PingAccess.

Para obtener más información, consulte [Ediciones de Azure Active Directory](../fundamentals/active-directory-whatis.md).

## <a name="next-steps"></a>Pasos siguientes
* [Serie de guías de inicio rápido sobre la administración de aplicaciones](view-applications-portal.md)
* [Planeación de una implementación de inicio de sesión único](plan-sso-deployment.md)
