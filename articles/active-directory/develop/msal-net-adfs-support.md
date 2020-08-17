---
title: Compatibilidad de AD FS en MSAL.NET | Azure
titleSuffix: Microsoft identity platform
description: Aprenda sobre la compatibilidad para los Servicios de federación de Active Directory (AD FS) en la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/16/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 4107b0785ef4635e0e5e745a35b605d2634a220d
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88166236"
---
# <a name="active-directory-federation-services-support-in-msalnet"></a>Compatibilidad para los Servicios de federación de Active Directory (AD FS) en MSAL.NET.
Los Servicios de federación de Active Directory (AD FS) en Windows Server le permiten agregar autenticación y autorización basadas en OpenID Connect y OAuth 2.0 a las aplicaciones que desarrolle. Estas aplicaciones pueden autenticar a los usuarios directamente en AD FS. Para más información, consulte [Escenarios de AD FS para desarrolladores](/windows-server/identity/ad-fs/overview/ad-fs-openid-connect-oauth-flows-scenarios).

La biblioteca de autenticación de Microsoft para .NET (MSAL.NET) admite dos escenarios de autenticación en AD FS:

- MSAL.NET se comunica con Azure Active Directory, que a su vez está *federado* con AD FS.
- MSAL.NET se comunica **directamente** con una entidad de ADFS. Esto solo se admite a partir de la versión AD FS 2019 y posteriores. Un escenario destacable es la compatibilidad con [Azure Stack](https://azure.microsoft.com/overview/azure-stack/)


## <a name="msal-connects-to-azure-ad-which-is-federated-with-ad-fs"></a>Conexión de MSAL a Azure AD, federado con AD FS
MSAL.NET admite la conexión a Azure AD, que inicia sesión de usuarios administrados (usuarios administrados en Azure AD) o de usuarios federados (usuarios administrados por otro proveedor de identidades, como AD FS). MSAL.NET no reconoce la federación de los usuarios. En lo que a él respecta, se comunica con Azure AD.

La [autoridad](msal-client-application-configuration.md#authority) que usa en este caso es la habitual (nombre de host de la autoridad + inquilino, common u organizations).

### <a name="acquiring-a-token-interactively"></a>Adquisición de un token de forma interactiva
Cuando se llama al método `AcquireTokenInteractive`, la experiencia del usuario suele ser:

1. El usuario especifica sus identificador de cuenta.
2. Azure AD muestra brevemente el mensaje "Taking you to your organization's page" (Le redirigimos a la página de su organización).
3. Al usuario se le redirige a la página de inicio de sesión del proveedor de identidades. Normalmente, la página de inicio de sesión se personaliza con el logotipo de la organización.

Las versiones compatibles de AD FS en este escenario federado son AD FS v2 y AD FS v3 (Windows Server 2012 R2) y AD FS v4 (AD FS 2016).

### <a name="acquiring-a-token-using-acquiretokenbyintegratedauthentication-or-acquiretokenbyusernamepassword"></a>Adquisición de un token mediante AcquireTokenByIntegratedAuthentication o AcquireTokenByUsernamePassword
Al adquirir un token mediante los métodos `AcquireTokenByIntegratedAuthentication` o `AcquireTokenByUsernamePassword`, MSAL.NET hace que la conexión del proveedor de identidades se base en el nombre de usuario.  MSAL.NET recibe un [token SAML 1.1](reference-saml-tokens.md) después de ponerse en contacto con el proveedor de identidades.  A continuación, proporciona el token SAML a Azure AD como aserción de usuario (similar al [flujo con derechos delegados](msal-authentication-flows.md#on-behalf-of)) para obtener un token JWT.

## <a name="msal-connects-directly-to-ad-fs"></a>Conexión directa de MSLA a AD FS
MSAL.NET admite la conexión a AD FS 2019, conforme con Open ID Connect y comprende los elementos PKCE y los ámbitos. Esta compatibilidad requiere que se aplique un Service Pack de [KB 4490481](https://support.microsoft.com/en-us/help/4490481/windows-10-update-kb4490481) en Windows Server. Con la conexión directa a AD FS, la autoridad que desee usar para compilar la aplicación se parece a `https://mysite.contoso.com/adfs/`.

Actualmente, no hay planes para admitir una conexión directa a:

- AD FS 16, ya que no admite PKCE y aún usa recursos y no ámbitos.
- AD FS v2, que no es compatible con OIDC.

 Si necesita admitir los escenarios que requieran una conexión directa con AD FS 2016, use la versión más reciente de la [biblioteca de autenticación de Azure Active Directory](../azuread-dev/active-directory-authentication-libraries.md#microsoft-supported-client-libraries). Podrá usar MSAL.NET una vez actualizado el sistema local a AD FS 2019.

## <a name="next-steps"></a>Pasos siguientes

En cuanto al caso federado, consulte [Configuración del comportamiento de inicio de sesión de Azure Active Directory de una aplicación mediante una directiva de detección del dominio de inicio](../manage-apps/configure-authentication-for-federated-users-portal.md).