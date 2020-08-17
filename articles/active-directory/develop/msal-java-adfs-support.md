---
title: Compatibilidad con AD FS (MSAL para Java)
titleSuffix: Microsoft identity platform
description: Aprenda sobre la compatibilidad para los Servicios de federación de Active Directory (AD FS) en la biblioteca de autenticación de Microsoft para Java (MSAL4j).
services: active-directory
author: sangonzal
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 11/21/2019
ms.author: sagonzal
ms.reviewer: nacanuma
ms.custom: aaddev, devx-track-java
ms.openlocfilehash: 5d01d1143563637e21aaa06a3f997c1507e4d8f1
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88114785"
---
# <a name="active-directory-federation-services-support-in-msal-for-java"></a>Compatibilidad para los Servicios de federación de Active Directory (AD FS) en MSAL para Java

Los Servicios de federación de Active Directory (AD FS) en Windows Server le permiten agregar autenticación y autorización basadas en OpenID Connect y OAuth 2.0 a la aplicación de la biblioteca de autenticación de Microsoft para Java (MSAL para Java). Una vez integrada, la aplicación puede autenticar usuarios en AD FS federada mediante Azure AD. Para obtener más información sobre los escenarios, consulte [Escenarios de AD FS para desarrolladores](/windows-server/identity/ad-fs/ad-fs-development).

Una aplicación que usa MSAL para Java se comunicará con Azure Active Directory (Azure AD), que luego se federa con AD FS.

MSAL para Java se conecta con Azure AD, que inicia sesión de usuarios que se administran en Azure AD (usuarios administrados) o de usuarios que administra otro proveedor de identidades como AD FS (usuarios federados). MSAL para Java no sabe que un usuario está federado. Simplemente, se comunica con Azure AD.

La [autoridad](msal-client-application-configuration.md#authority) que usa en este caso es la habitual (nombre de host de la autoridad + inquilino, common u organizations).

## <a name="acquire-a-token-interactively-for-a-federated-user"></a>Adquisición interactiva de un token para un usuario federado

Cuando se llama a `ConfidentialClientApplication.AcquireToken()` o `PublicClientApplication.AcquireToken()` con `AuthorizationCodeParameters` o `DeviceCodeParameters`, la experiencia del usuario suele ser:

1. El usuario especifica sus identificador de cuenta.
2. Azure AD muestra brevemente el mensaje "Taking you to your organization's page" (Le estamos dirigiendo a la página de su organización) y se redirige al usuario a la página de inicio de sesión del proveedor de identidades. Normalmente, la página de inicio de sesión se personaliza con el logotipo de la organización.

Las versiones de AD FS admitidas en este escenario federado son:
- Servicios de federación de Active Directory (AD FS) FS V2
- Servicios de federación de Active Directory (AD FS) V3 (Windows Server 2012 R2)
- Servicios de federación de Active Directory (AD FS) V4 (AD FS 2016)

## <a name="acquire-a-token-via-username-and-password"></a>Adquisición de un token mediante el nombre de usuario y la contraseña

Al adquirir un token con `ConfidentialClientApplication.AcquireToken()` o `PublicClientApplication.AcquireToken()` con `IntegratedWindowsAuthenticationParameters` o `UsernamePasswordParameters`, MSAL para Java hace que el proveedor de identidades se ponga en contacto con él en función del nombre de usuario. MSAL para Java obtiene un [token de SAML 1.1](reference-saml-tokens.md) del proveedor de identidades, que luego proporciona a Azure AD que, a su vez, devuelve el JSON Web Token (JWT).

## <a name="next-steps"></a>Pasos siguientes

En cuanto al caso federado, consulte [Configuración del comportamiento de inicio de sesión de Azure Active Directory de una aplicación mediante una directiva de detección del dominio de inicio](../manage-apps/configure-authentication-for-federated-users-portal.md).