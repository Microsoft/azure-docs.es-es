---
title: Compatibilidad con Azure AD FS (MSAL Python)
titleSuffix: Microsoft identity platform
description: Más información sobre la compatibilidad para los Servicios de federación de Active Directory (AD FS) en la biblioteca de autenticación de Microsoft para Python
services: active-directory
author: abhidnya13
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/23/2019
ms.author: abpati
ms.reviewer: nacanuma
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: ad874a4b117b7a7ccecea0e1800ca18c6fce8aaa
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87846214"
---
# <a name="active-directory-federation-services-support-in-msal-for-python"></a>Compatibilidad para los Servicios de federación de Active Directory (AD FS) en MSAL para Python

Los Servicios de federación de Active Directory (AD FS) en Windows Server le permiten agregar autenticación y autorización basadas en OpenID Connect y OAuth 2.0 a las aplicaciones mediante la biblioteca de autenticación de Microsoft (MSAL) para Python. Gracias a la biblioteca MSAL para Python, la aplicación puede autenticar a los usuarios directamente en AD FS. Para obtener más información sobre los escenarios, consulte [Escenarios de AD FS para desarrolladores](/windows-server/identity/ad-fs/ad-fs-development).

Normalmente hay dos maneras de autenticarse en AD FS:

- MSAL Python se comunica con Azure Active Directory, que a su vez está federado con otros proveedores de identidades. La federación se produce a través de AD FS. MSAL Python se conecta con Azure AD, que inicia sesión de usuarios que se administran en Azure AD (usuarios administrados) o de usuarios que administra otro proveedor de identidades como AD FS (usuarios federados). MSAL Python no sabe que un usuario está federado. Simplemente, se comunica con Azure AD. La [autoridad](msal-client-application-configuration.md#authority) que usa en este caso es la habitual (nombre de host de la autoridad + inquilino, common u organizations).
- MSAL Python se comunica directamente con una entidad de certificación de AD FS. Esto solo se admite a partir de la versión AD FS 2019 y posteriores.

## <a name="connect-to-active-directory-federated-with-ad-fs"></a>Conexión a Active Directory federado con AD FS

### <a name="acquire-a-token-interactively-for-a-federated-user"></a>Adquisición interactiva de un token para un usuario federado

Lo siguiente se aplica tanto si se conecta directamente a los Servicios de federación de Active Directory (AD FS) o a través de Active Directory.

Cuando se llama a `acquire_token_by_authorization_code` o`acquire_token_by_device_flow`, la experiencia del usuario suele ser la siguiente:

1. El usuario especifica sus identificador de cuenta.
2. Azure AD muestra brevemente el mensaje "Taking you to your organization's page" (Le estamos dirigiendo a la página de su organización) y se redirige al usuario a la página de inicio de sesión del proveedor de identidades. Normalmente, la página de inicio de sesión se personaliza con el logotipo de la organización.

Las versiones de AD FS admitidas en este escenario federado son:
- Servicios de federación de Active Directory (AD FS) FS V2
- Servicios de federación de Active Directory (AD FS) V3 (Windows Server 2012 R2)
- Servicios de federación de Active Directory (AD FS) V4 (AD FS 2016)

### <a name="acquire-a-token-via-username-and-password"></a>Adquisición de un token mediante el nombre de usuario y la contraseña

Lo siguiente se aplica tanto si se conecta directamente a los Servicios de federación de Active Directory (AD FS) o a través de Active Directory.

Al adquirir un token con `acquire_token_by_username_password`, MSAL Python hace que el proveedor de identidades se ponga en contacto con él en función del nombre de usuario. MSAL Python obtiene un [token de SAML 1.1](reference-saml-tokens.md) del proveedor de identidades, que luego proporciona a Azure AD que, a su vez, devuelve el JSON Web Token (JWT).

## <a name="connecting-directly-to-ad-fs"></a>Conexión directa a AD FS

Al conectar el directorio a AD FS, la autoridad que querrá usar para compilar la aplicación se parecerá a `https://somesite.contoso.com/adfs/`.

MSAL Python admite ADFS 2019.

No admite una conexión directa a ADFS 2016 o ADFS V2. Si tiene que admitir escenarios que requieran una conexión directa a ADFS 2016, use la versión más reciente de ADAL para Python. Podrá usar MSAL Python una vez actualizado el sistema local a ADFS 2019.

## <a name="next-steps"></a>Pasos siguientes

- En cuanto al caso federado, consulte [Configuración del comportamiento de inicio de sesión de Azure Active Directory de una aplicación mediante una directiva de detección del dominio de inicio](../manage-apps/configure-authentication-for-federated-users-portal.md).
