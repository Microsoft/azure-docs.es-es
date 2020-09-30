---
title: Complemento de Microsoft Enterprise SSO para dispositivos Apple
titleSuffix: Microsoft identity platform | Azure
description: Obtenga información sobre el complemento de SSO de Microsoft Azure Active Directory para dispositivos iOS y macOS.
services: active-directory
author: brandwe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/15/2020
ms.author: brandwe
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: e43ce318ca9e9b14ad059dd296799667653e0f95
ms.sourcegitcommit: 6e1124fc25c3ddb3053b482b0ed33900f46464b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90561353"
---
# <a name="microsoft-enterprise-sso-plug-in-for-apple-devices-preview"></a>Complemento de Microsoft Enterprise SSO para dispositivos Apple (versión preliminar)

> [!NOTE]
> Esta característica está en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

El *complemento de Microsoft Enterprise SSO para dispositivos de Apple* proporciona cuentas de inicio de sesión único (SSO) para Azure Active Directory (Azure AD) en todas las aplicaciones que admiten la característica [Enterprise Single Sign-On](https://developer.apple.com/documentation/authenticationservices) de Apple. Microsoft ha trabajado estrechamente con Apple para desarrollar este complemento con el fin de aumentar la facilidad de uso de su aplicación y ofrecer la máxima protección de la que disponen Apple y Microsoft.

En esta versión preliminar pública, el complemento Enterprise SSO solo está disponible para dispositivos iOS y se distribuye en determinadas aplicaciones de Microsoft.

## <a name="features"></a>Características

El complemento Microsoft Enterprise SSO para dispositivos de Apple ofrece las siguientes ventajas:

- Proporciona SSO para las cuentas de Azure AD en todas las aplicaciones que admiten la característica Enterprise Single Sign-On de Apple.
- Se entrega automáticamente en Microsoft Authenticator y se puede habilitar mediante cualquier solución de administración de dispositivos móviles (MDM).

## <a name="requirements"></a>Requisitos

Para usar el complemento de Microsoft Enterprise SSO para dispositivos de Apple:

- iOS 13.0 o posterior debe estar instalado en el dispositivo.
- Debe instalarse una aplicación de Microsoft que proporcione el complemento Microsoft Enterprise SSO para dispositivos de Apple en el dispositivo. En el caso de la versión preliminar pública, estas aplicaciones incluyen la [aplicación Microsoft Authenticator](../user-help/user-help-auth-app-overview.md).
- El dispositivo debe estar inscrito en MDM (por ejemplo, con Microsoft Intune).
- La configuración se debe insertar en el dispositivo para habilitar el complemento Microsoft Enterprise SSO para dispositivos Apple en el dispositivo. Apple requiere esta restricción de seguridad.

## <a name="enable-the-sso-plug-in-with-mobile-device-management-mdm"></a>Habilitación del complemento de inicio de sesión único con administración de dispositivos móviles (MDM)

Para habilitar el complemento de Microsoft Enterprise SSO para dispositivos de Apple, los dispositivos deben enviar una señal a través de un servicio MDM. Puesto que Microsoft incluye el complemento Enterprise SSO en la [aplicación Microsoft Authenticator](..//user-help/user-help-auth-app-overview.md), use su MDM para configurar la aplicación y habilitar el complemento Microsoft Enterprise SSO.

Use los parámetros siguientes para configurar el complemento Microsoft Enterprise SSO para dispositivos de Apple:

- **Tipo**: Redirect
- **Identificador de extensión**: `com.microsoft.azureauthenticator.ssoextension`
- **Id. de equipo**: (este campo no es necesario en iOS).
- **URL**:
  - `https://login.microsoftonline.com`
  - `https://login.microsoft.com`
  - `https://sts.windows.net`
  - `https://login.partner.microsoftonline.cn`
  - `https://login.chinacloudapi.cn`
  - `https://login.microsoftonline.de`
  - `https://login.microsoftonline.us`
  - `https://login.usgovcloudapi.net`
  - `https://login-us.microsoftonline.com`
  
### <a name="additional-configuration-options"></a>Opciones de configuración adicionales
Se pueden agregar opciones de configuración adicionales para ampliar la funcionalidad de inicio de sesión único a otras aplicaciones.

#### <a name="enable-sso-for-apps-that-dont-use-msal"></a>Habilitación del inicio de sesión único para aplicaciones que no usan MSAL

El complemento de inicio de sesión único permite que cualquier aplicación participe en el inicio de sesión único, aunque no se haya desarrollado con un SDK de Microsoft, como la Biblioteca de autenticación de Microsoft (MSAL).

Este complemento se instala automáticamente en los dispositivos que han descargado la aplicación Microsoft Authenticator y han registrado el dispositivo en la organización. Es probable que la organización ya use la aplicación Authenticator en escenarios como la autenticación multifactor, la autenticación sin contraseña y el acceso condicional. Aunque se puede activar en las aplicaciones con cualquier proveedor de MDM, Microsoft ha facilitado su configuración en Microsoft Endpoint Manager de Intune. Para configurar estas aplicaciones de forma que usen el complemento de inicio de sesión único instalado por la aplicación Authenticator, se usa una lista de permitidos.

Solo se admiten las aplicaciones que usan vistas web o tecnologías de red de Apple nativas. Si una aplicación incluye su propia implementación de capa de red, no se admite el complemento Microsoft Enterprise SSO.  

Use los parámetros siguientes para configurar el complemento Microsoft Enterprise SSO en dispositivos que no usan MSAL:

- **Clave**: `AppAllowList`
- **Tipo**: `String`
- **Valor**: lista delimitada por comas de identificadores de lote de aplicaciones que pueden participar en el inicio de sesión único.
- **Ejemplo**: `com.contoso.workapp, com.contoso.travelapp`

Las [aplicaciones consentidas](./application-consent-experience.md) que el administrador de MDM permita participar en el inicio de sesión único pueden obtener un token para el usuario final de forma silenciosa. Por lo tanto, es importante agregar únicamente aplicaciones de confianza a la lista de permitidos. 

No es necesario agregar aplicaciones que usen MSAL o ASWebAuthenticationSession a esta lista. Esas aplicaciones están habilitadas de forma predeterminada. 

#### <a name="allow-creating-sso-session-from-any-application"></a>Permitir la creación de sesiones SSO desde cualquier aplicación

De forma predeterminada, el complemento Microsoft Enterprise SSO proporciona inicio de sesión único en aplicaciones autorizadas solo cuando dicho complemento ya tiene una credencial compartida. El complemento Microsoft Enterprise SSO puede adquirir una credencial compartida cuando otra aplicación basada en ADAL o MSAL la llama durante la adquisición de tokens. La mayoría de las aplicaciones de Microsoft usan Microsoft Authenticator o el complemento de inicio de sesión único. Esto significa que, de forma predeterminada, lo mejor es el inicio de sesión único fuera de los flujos de las aplicaciones nativas.  

Al habilitar la marca `browser_sso_interaction_enabled`, las aplicaciones que no son de MSAL y el explorador Safari pueden realizar el arranque inicial y obtener una credencial compartida. Si el complemento Microsoft Enterprise SSO aún no tiene una credencial compartida, intentará obtener una cada vez que se solicite un inicio de sesión desde una dirección URL de Azure AD en el explorador Safari, ASWebAuthenticationSession, SafariViewController u otra aplicación nativa incluida en la lista de permitidos.  

- **Clave**: `browser_sso_interaction_enabled`
- **Tipo**: `Integer`
- **Valor**: 1 o 0

Se recomienda habilitar esta marca para obtener una experiencia más uniforme entre todas las aplicaciones. Esta opción está deshabilitada de manera predeterminada. 

#### <a name="disable-oauth2-application-prompts"></a>Deshabilitación de los avisos de aplicaciones OAuth2

El complemento Microsoft Enterprise SSO proporciona inicio de sesión único mediante la anexión de credenciales compartidas a las solicitudes de red procedentes de aplicaciones permitidas. Es posible que algunas aplicaciones OAuth2 exijan el aviso del usuario final en la capa del protocolo. En esas aplicaciones, las credenciales compartidas se pasarán por alto.  

Al habilitar la marca `disable_explicit_app_prompt`, se restringe la posibilidad de que las aplicaciones web y nativas fuercen un aviso al usuario final en la capa del protocolo y omitan el inicio de sesión único.

- **Clave**: `disable_explicit_app_prompt`
- **Tipo**: `Integer`
- **Valor**: 1 o 0

Se recomienda habilitar esta marca para obtener una experiencia más uniforme entre todas las aplicaciones. Esta opción está deshabilitada de manera predeterminada. 

#### <a name="use-intune-for-simplified-configuration"></a>Uso de Intune para la configuración simplificada

Puede usar Microsoft Intune como su servicio MDM para facilitar la configuración del complemento de Microsoft Enterprise SSO. Para obtener más información, consulte la [documentación de configuración de Intune](/intune/configuration/ios-device-features-settings).

## <a name="using-the-sso-plug-in-in-your-application"></a>Uso del complemento de inicio de sesión único en la aplicación

La [Biblioteca de autenticación de Microsoft (MSAL) para dispositivos Apple](https://github.com/AzureAD/microsoft-authentication-library-for-objc) de la versión 1.1.0 y versiones posteriores admite el complemento Microsoft Enterprise SSO para dispositivos Apple.

Si va a compilar una aplicación para escenarios de Frontline Worker, consulte [Modo de dispositivo compartido para dispositivos iOS](msal-ios-shared-devices.md) para conocer la configuración adicional de la característica.

## <a name="how-the-sso-plug-in-works"></a>Funcionamiento del complemento de inicio de sesión único

El complemento Microsoft Enterprise SSO se basa en el [marco Enterprise Single Sign-On de Apple](https://developer.apple.com/documentation/authenticationservices/asauthorizationsinglesignonprovider?language=objc). Los proveedores de identidades que se incorporan al marco pueden interceptar el tráfico de red en sus dominios y mejorar o cambiar el modo en que se administran esas solicitudes. Por ejemplo, el complemento de inicio de sesión único puede mostrar una interfaz de usuario adicional para recopilar credenciales de usuario final de forma segura, exigir MFA o proporcionar tokens a la aplicación de forma silenciosa.

Las aplicaciones nativas también pueden implementar operaciones personalizadas y comunicarse directamente con el complemento de inicio de sesión único.
Puede aprender más sobre el marco de inicio de sesión único en este [vídeo de WWDC de 2019 de Apple](https://developer.apple.com/videos/play/tech-talks/301/).

### <a name="applications-that-use-msal"></a>Aplicaciones que usan MSAL

La [Biblioteca de autenticación de Microsoft (MSAL) para dispositivos Apple](https://github.com/AzureAD/microsoft-authentication-library-for-objc) de la versión 1.1.0 y posteriores admite de forma nativa el complemento Microsoft Enterprise SSO para dispositivos Apple con cuentas profesionales y educativas. 

Si ha seguido [todos los pasos recomendados](./quickstart-v2-ios.md) y ha usado el [formato de URI de redirección](./redirect-uris-ios.md) predeterminado, no es necesaria ninguna configuración especial. Cuando se ejecuta en un dispositivo en el que existe el complemento de inicio de sesión único, MSAL lo invoca automáticamente en todas las solicitudes de token interactivas y silenciosas, así como en las operaciones de enumeración y eliminación de cuentas. Dado que MSAL implementa el protocolo del complemento de inicio de sesión único nativo que depende de operaciones personalizadas, esta instalación proporciona la mejor experiencia nativa para el usuario final. 

Si MDM no habilita el complemento de inicio de sesión único, pero la aplicación Microsoft Authenticator existe en el dispositivo, MSAL la usará en su lugar con todas las solicitudes de token interactivas. El complemento de inicio de sesión único comparte el inicio de sesión único con la aplicación Microsoft Authenticator.

### <a name="applications-that-dont-use-msal"></a>Aplicaciones que no usan MSAL

Las aplicaciones que no usen MSAL también pueden obtener el inicio de sesión único si un administrador las agrega explícitamente a la lista de permitidos. 

No es necesario cambiar el código de esas aplicaciones siempre y cuando se cumplan las siguientes condiciones:

- La aplicación usa marcos de Apple para ejecutar las solicitudes de red (por ejemplo, [WKWebView](https://developer.apple.com/documentation/webkit/wkwebview) o [NSURLSession](https://developer.apple.com/documentation/foundation/nsurlsession)) 
- La aplicación usa protocolos estándar para comunicarse con Azure AD (por ejemplo, OAuth2, SAML, WS-Federation)
- La aplicación no recopila el nombre de usuario y la contraseña de texto no cifrado en la interfaz de usuario nativa

En este caso, el inicio de sesión único se proporciona cuando la aplicación crea una solicitud de red y abre un explorador web para iniciar la sesión del usuario. Cuando a un usuario se le redirige a una dirección URL de inicio de sesión de Azure AD, el complemento de inicio de sesión único la valida y comprueba si hay una credencial de inicio de sesión único disponible para ella. Si hay una, el complemento de inicio de sesión único pasa la credencial de inicio de sesión único a Azure AD, que autoriza a la aplicación que lleve a cabo la solicitud de red sin pedir al usuario que escriba sus credenciales. Además, si Azure AD conoce el dispositivo, el complemento de inicio de sesión único también aprueba el certificado de este para satisfacer la comprobación de acceso condicional basada en dispositivo. 

Para admitir el inicio de sesión único en aplicaciones que no son de MSAL, el complemento de inicio de sesión único implementa un protocolo similar al del complemento de explorador de Windows descrito en [¿Qué es un token de actualización principal?](../devices/concept-primary-refresh-token.md#browser-sso-using-prt). 

En comparación con las aplicaciones basadas en MSAL, el complemento de inicio de sesión único funciona de forma más transparente con las aplicaciones que no son de MSAL gracias a la integración con la experiencia de inicio de sesión del explorador existente que proporcionan las aplicaciones. El usuario final verá la experiencia con la que ya está familiarizado, con la ventaja de no tener que realizar inicios de sesión adicionales en cada una de las aplicaciones. Por ejemplo, en lugar de mostrar el selector de cuentas nativo, el complemento de inicio de sesión único agrega sesiones de inicio de sesión único a la experiencia del selector de cuentas basado en web. 

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el modo de dispositivo compartido en iOS, consulte [Modo de dispositivo compartido para dispositivos iOS](msal-ios-shared-devices.md).
