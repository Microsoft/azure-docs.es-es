---
title: Registro de aplicaciones de página única (SPA) | Azure
titleSuffix: Microsoft identity platform
description: Aprenda a compilar una aplicación de página única (registro de la aplicación).
services: active-directory
author: hahamil
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/19/2020
ms.author: hahamil
ms.custom: aaddev
ms.openlocfilehash: 2d596b6d0e291d197146bf5e00827d82b74c125c
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89434544"
---
# <a name="single-page-application-app-registration"></a>Aplicación de página única: Registro de aplicación

Para registrar una aplicación de página única (SPA) en la Plataforma de identidad de Microsoft, complete los pasos siguientes. Los pasos de registro difieren entre MSAL.js 1.0, que admite el flujo de concesión implícita, y MSAL.js 2.0, que admite el flujo de código de autorización con PKCE.

[!INCLUDE [MSAL.js 2.0 and Azure AD B2C temporary incompatibility notice](../../../includes/msal-b2c-cors-compatibility-notice.md)]

## <a name="create-the-app-registration"></a>Creación del registro de aplicaciones

Tanto para las aplicaciones basadas en MSAL.js 1.0 como 2.0, empiece por completar los pasos siguientes para crear el registro de aplicación inicial.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Si su cuenta tiene acceso a varios inquilinos, seleccione el filtro **Directorio + suscripción** en el menú superior y, a continuación, seleccione el inquilino que debe contener el registro de aplicación que va a crear.
1. Busque y seleccione **Azure Active Directory**.
1. En **Administrar**, seleccione **Registros de aplicaciones**.
1. Seleccione **Nuevo registro**, escriba un **Nombre** para la aplicación y elija los **Tipos de cuenta admitidos** para la aplicación. **NO** escriba un **URI de redirección**. Para obtener una descripción de los distintos tipos de cuenta, consulte [Registro de una aplicación nueva mediante Azure Portal](quickstart-register-app.md).
1. Seleccione **Registrar** para crear el registro de aplicación.

Después, configure el registro de la aplicación con un **URI de redirección** para especificar a dónde la Plataforma de identidad de Microsoft debe redirigir al cliente, junto con los tokens de seguridad. Siga los pasos adecuados para la versión de MSAL.js que usa en la aplicación:

- [MSAL.js 2.0 con flujo de código de autenticación](#redirect-uri-msaljs-20-with-auth-code-flow) (recomendado)
- [MSAL.js 1.0 con flujo implícito](#redirect-uri-msaljs-10-with-implicit-flow)

## <a name="redirect-uri-msaljs-20-with-auth-code-flow"></a>URI de redirección: [MSAL.js 2.0 con flujo de código de autenticación](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

Siga estos pasos para agregar un URI de redirección para una aplicación que use MSAL.js 2.0 o posterior. MSAL.js 2.0 o posterior admite el flujo de código de autorización con PKCE y CORS en respuesta a [restricciones de cookies de terceros en el explorador](reference-third-party-cookies-spas.md). El flujo de concesión implícita no se admite en MSAL.js 2.0 o posterior.

1. En Azure Portal, seleccione el registro de la aplicación que creó anteriormente en [Creación del registro de aplicaciones](#create-the-app-registration).
1. En **Administrar**, seleccione **Autenticación** y luego seleccione **Agregar una plataforma**.
1. En **Aplicaciones web**, seleccione el icono **Aplicación de página única**.
1. En **URI de redirección**, escriba un [URI de redirección](reply-url.md). **NO** seleccione ninguna de las casillas en **Concesión implícita**.
1. Seleccione **Configurar** para terminar de agregar el URI de redirección.

Ya ha completado el registro de aplicación de la aplicación de página única (SPA) y ha configurado un URI de redirección al que se redirigirá al cliente y se enviarán los tokens de seguridad. Al configurar el URI de redirección con el icono **Aplicación de página única** en el panel **Agregar una plataforma**, el registro de aplicación se configura para admitir el flujo de código de autorización con PKCE y CORS.

Siga el [tutorial](tutorial-v2-javascript-auth-code.md) para obtener más información.

## <a name="redirect-uri-msaljs-10-with-implicit-flow"></a>URI de redirección: [MSAL.js 1.0 con flujo implícito](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-core)

Siga estos pasos para agregar un URI de redirección para una aplicación de página única que usa MSAL.js 1.3 o anterior y el flujo de concesión implícita. Las aplicaciones que usan MSAL.js 1.3 o anterior no admiten el flujo de código de autenticación.

1. En Azure Portal, seleccione el registro de la aplicación que creó anteriormente en [Creación del registro de aplicaciones](#create-the-app-registration).
1. En **Administrar**, seleccione **Autenticación** y luego seleccione **Agregar una plataforma**.
1. En **Aplicaciones web**, seleccione el icono **Aplicación de página única**.
1. En **URI de redirección**, escriba un [URI de redirección](reply-url.md).
1. Habilite el **Flujo implícito**:
    - Si los usuarios pueden iniciar sesión en la aplicación, seleccione **Tokens de id.**
    - Si la aplicación también necesita llamar a una API web protegida, seleccione **Tokens de acceso**. Para obtener más información sobre estos tipos de token, consulte [Tokens de id.](id-tokens.md) y [Tokens de acceso](access-tokens.md).
1. Seleccione **Configurar** para terminar de agregar el URI de redirección.

Ya ha completado el registro de aplicación de la aplicación de página única (SPA) y ha configurado un URI de redirección al que se redirigirá al cliente y se enviarán los tokens de seguridad. Al seleccionar uno o ambos entre **Tokens de id.** y **Tokens de acceso**, ha habilitado el flujo de concesión implícita.

Siga el [tutorial](tutorial-v2-javascript-spa.md) para obtener más información.

## <a name="note-about-authorization-flows"></a>Nota sobre los flujos de autorización

De forma predeterminada, un registro de aplicación creado mediante la configuración de la plataforma de aplicaciones de página única habilita el flujo de código de autorización. Para aprovechar este flujo, la aplicación debe usar MSAL.js 2.0 o posterior.

Como se mencionó anteriormente, las aplicaciones de página única que usan MSAL.js 1.3 están restringidas al flujo de concesión implícita. Los [procedimientos recomendados de OAuth 2.0](v2-oauth2-auth-code-flow.md) actuales indican el uso del flujo de código de autorización en lugar del flujo implícito para las aplicaciones de página única. Contar con tokens de actualización de duración limitada también ayuda a que la aplicación se adapte a las [limitaciones de privacidad de cookies de los exploradores modernos](reference-third-party-cookies-spas.md), como ITP de Safari.

Cuando todas sus aplicaciones de página única en producción representadas por un registro de aplicación usen MSAL.js 2.0 y el flujo de código de autorización, desactive la configuración de concesión implícita del panel **Autenticación** del registro de aplicación en Azure Portal. Sin embargo, las aplicaciones que usen MSAL.js 1.x y el flujo implícito pueden seguir funcionando si deja el flujo implícito habilitado (activado).

## <a name="next-steps"></a>Pasos siguientes

A continuación, configure el código de la aplicación para que use el registro de aplicación que creó en los pasos anteriores:.

> [!div class="nextstepaction"]
> [Configuración del código de la aplicación](scenario-spa-app-configuration.md)
