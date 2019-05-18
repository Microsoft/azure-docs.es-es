---
title: 'Aplicación móvil que llama a web API: Introducción | Plataforma de identidad de Microsoft'
description: Aprenda a crear una aplicación móvil que llama a web API (información general)
services: active-directory
documentationcenter: dev-center-name
author: danieldobalian
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: dadobali
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: d9978905f8ae0719f6a77ca2351f2c43bbf94ee0
ms.sourcegitcommit: f013c433b18de2788bf09b98926c7136b15d36f1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2019
ms.locfileid: "65550363"
---
# <a name="scenario-mobile-application-that-calls-web-apis"></a>Escenario: Aplicaciones móviles que las llamadas API web

Aprenda todo lo que necesita saber para crear una aplicación móvil que llama a las API web.

## <a name="prerequisites"></a>Requisitos previos

[!INCLUDE [Prerequisites](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="getting-started"></a>Introducción

Cree su primera aplicación móvil y pruebe un inicio rápido.

> [!div class="nextstepaction"]
> [Inicio rápido: Adquirir un token y llamar a Microsoft Graph API desde una aplicación de Android](./quickstart-v2-android.md)
>
> [Inicio rápido: Adquirir un token y llamar a Microsoft Graph API desde una aplicación de iOS](./quickstart-v2-ios.md)
>
> [Inicio rápido: Adquirir un token y llamar a Microsoft Graph API desde un aplicación de Android y Xamarin iOS](https://github.com/Azure-Samples/active-directory-xamarin-native-v2)

## <a name="overview"></a>Información general

Una experiencia de usuario personalizada y sin problemas es esencial para las aplicaciones móviles.  Plataforma de identidad de Microsoft permite a los desarrolladores móviles crear esa misma experiencia para iOS y Android a los usuarios. La aplicación puede iniciar sesión en los usuarios de Azure Active Directory (Azure AD), los usuarios de cuentas de Microsoft personales y los usuarios de Azure AD B2C y adquirir tokens para llamar a una API web en su nombre. Para implementar estos flujos, vamos a usar la biblioteca de autenticación de Microsoft (MSAL), que implementa el estándar del sector [flujo de código de autorización de OAuth2.0](v2-oauth2-auth-code-flow.md).

![Aplicaciones de demonio](./media/scenarios/mobile-app.svg)

Consideraciones para las aplicaciones móviles:

- **Experiencia del usuario es clave**: Permitir a los usuarios ver el valor de la aplicación antes de solicitar el inicio de sesión y solicitar solo los permisos necesarios.
- **Admite todas las configuraciones de usuario**: Muchos usuarios empresariales móviles están bajo el acceso condicional y las directivas de cumplimiento del dispositivo. No olvide admitir estos escenarios claves.
- **Implementar el inicio de sesión único (SSO)**: MSAL y la plataforma Microsoft identity simplifican habilitar inicio de sesión único a través del explorador del dispositivo o el Microsoft Authenticator (Portal de empresa de Intune en Android).

## <a name="specifics"></a>Características específicas

Tenga en cuenta estas consideraciones al compilar una aplicación móvil en la plataforma Microsoft identity:

- Según la plataforma, una interacción del usuario podría ser necesaria la primera vez que los usuarios iniciar sesión. Por ejemplo, iOS requiere que las aplicaciones para mostrar la interacción del usuario cuando se usa inicio de sesión único en la primera vez a través de Microsoft Authenticator (Portal de empresa de Intune en Android).
- En iOS y Android, MSAL podría utilizar un explorador externo (que es posible que aparecen encima de la aplicación) para los usuarios inicien sesión. Puede personalizar la configuración para usar vistas Web en la aplicación en su lugar.
- Nunca use un secreto en una aplicación móvil. Será accesible para todos los usuarios.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Registro de aplicaciones](scenario-mobile-app-registration.md)
