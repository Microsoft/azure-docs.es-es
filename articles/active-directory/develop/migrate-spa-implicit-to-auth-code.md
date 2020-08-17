---
title: Migración de una aplicación de página única de JavaScript desde una concesión implícita a un flujo de código de autorización | Azure
titleSuffix: Microsoft identity platform
description: Cómo actualizar JavaScript SPA mediante MSAL.js 1.x y el flujo de concesión implícita a MSAL.js 2.x, y el flujo de código de autorización mediante la compatibilidad con PKCE y CORS.
services: active-directory
author: hahamil
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 07/17/2020
ms.author: hahamil
ms.custom: aaddev
ms.openlocfilehash: b7316756aab7875dce50a3783cb95ca42676b970
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87027094"
---
# <a name="migrate-a-javascript-single-page-app-from-implicit-grant-to-auth-code-flow"></a>Migración de una aplicación de página única de JavaScript desde una concesión implícita a un flujo de código de autorización

La biblioteca de autenticación de Microsoft para JavaScript (MSAL. js) v2.0 proporciona compatibilidad con el flujo de código de autorización para PKCE y CORS en aplicaciones de página única en la Plataforma de identidad de Microsoft. Siga los pasos de las secciones siguientes para migrar la aplicación MSAL.js 1.x mediante la concesión implícita a MSAL.js 2.0+ (en adelante *2.x*) y el flujo de código de autenticación.

MSAL.js 2.x mejora MSAL.js 1.x al ser compatible con el flujo de código de autorización en el explorador, en lugar de utilizar el flujo de concesión implícita. MSAL.js 2.x **no** admite el flujo implícito.

## <a name="migration-steps"></a>Pasos de migración

Para actualizar su aplicación a MSAL.js 2.x y al flujo de código de autenticación, hay tres pasos principales:

1. Cambie los URI de redirección del [registro de aplicación](#switch-redirect-uris-to-spa-platform) de la plataforma **web** a la plataforma de **aplicación de página única**.
1. Actualice el [código](#switch-redirect-uris-to-spa-platform) de MSAL.js 1.x a **2.x**.
1. Desactive la [concesión implícita](#disable-implicit-grant-settings) en el registro de aplicación cuando todas las aplicaciones que comparten el registro se hayan actualizado a MSAL.js 2.x y al flujo de código de autenticación.

Las siguientes secciones describen cada opción con más detalle.

## <a name="switch-redirect-uris-to-spa-platform"></a>Cambio de los URI de redirección a la plataforma SPA

Si desea seguir usando el registro de aplicación existente para sus aplicaciones, use Azure Portal para actualizar los URI de redireccionamiento del registro a la plataforma SPA. Hacerlo así habilita el flujo de código de autorización mediante la compatibilidad con PKCE y CORS para las aplicaciones que usan el registro. Todavía es necesario que actualice el código de la aplicación a MSAL.js v2.x.

Siga estos pasos con los registros de aplicaciones que están configurados actualmente con los URI de redirección de la plataforma **web**:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y seleccione su inquilino de **Azure Active Directory**.
1. En **Registros de aplicaciones**, seleccione la aplicación y, a continuación, **Autenticación**.
1. En el icono de plataforma **Web** que hay debajo de **URI de redirección**, seleccione el banner de advertencia que indica que debe migrar los URI.

    :::image type="content" source="media/migrate-spa-implicit-to-auth-code/portal-01-implicit-warning-banner.png" alt-text="Banner de advertencia de flujo implícito en el icono de aplicación web en Azure Portal":::
1. Seleccione *solo* los URI de redirección cuyas aplicaciones van a usar MSAL.js 2.x y, a continuación, **Configurar**.

    :::image type="content" source="media/migrate-spa-implicit-to-auth-code/portal-02-select-redirect-uri.png" alt-text="Seleccione el panel de URI de redirección en el panel SPA de Azure Portal":::.

Estos URI de redirección deberían aparecer ahora en el icono de la plataforma de **aplicación de página única**, lo cual indica que la compatibilidad de CORS con el flujo de código de autorización y PKCE está habilitada para estos URI.

:::image type="content" source="media/migrate-spa-implicit-to-auth-code/portal-03-spa-redirect-uri-tile.png" alt-text="Icono de aplicación de página única en el registro de aplicación en Azure Portal":::

También puede [crear un nuevo registro de aplicación](scenario-spa-app-registration.md) en lugar de actualizar los URI de redirección en el registro existente.

## <a name="update-your-code-to-msaljs-2x"></a>Actualización del código a MSAL.js 2.x

En MSAL 1.x, ha creado una instancia de aplicación al inicializar [UserAgentApplication][msal-js-useragentapplication] de la siguiente manera:

```javascript
// MSAL 1.x
import * as msal from "msal";

const msalInstance = new msal.UserAgentApplication(config);
```

En MSAL 2.x, inicialice [PublicClientApplication][msal-js-publicclientapplication]:

```javascript
// MSAL 2.x
import * as msal from "@azure/msal-browser";

const msalInstance = new msal.PublicClientApplication(config);
```

Para ver una guía paso a paso sobre cómo agregar MSAL 2.x a su aplicación, consulte [Tutorial: Inicio de sesión de usuarios y llamada a Microsoft Graph API desde una aplicación de página única (SPA) de JavaScript mediante un flujo de código de autorización](tutorial-v2-javascript-auth-code.md).

Para informarse sobre otros cambios que puede que deba realizar en el código, consulte la [guía de migración](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/v1-migration.md) en GitHub.

## <a name="disable-implicit-grant-settings"></a>Desactivación de la configuración de concesión implícita

Una vez que haya actualizado todas las aplicaciones de producción que usan este registro de aplicación y su identificador de cliente a MSAL 2.x y el flujo de código de autorización, tiene que desactivar la configuración de concesión implícita en el registro de aplicación.

Al desactivar la configuración de concesión implícita en el registro de aplicación, el flujo implícito se deshabilita para todas las aplicaciones que usan el registro y su identificador de cliente.

**No** deshabilite el flujo de concesión implícita antes de haber actualizado todas sus aplicaciones a MSAL.js 2.x y [PublicClientApplication][msal-js-publicclientapplication].

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el flujo de código de autorización, incluidas las diferencias entre los flujos de código implícito y de autenticación, consulte [Plataforma de identidad y flujo de código de autorización de OAuth 2.0](v2-oauth2-auth-code-flow.md).

Si quiere profundizar más en el desarrollo de aplicaciones de una sola página de JavaScript en la plataforma de Microsoft Identity, la serie de varias partes [Escenario: Aplicación de página única](scenario-spa-overview.md) puede ayudarle a empezar.

<!-- LINKS - external -->
[msal-js-useragentapplication]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/modules/_useragentapplication_.html
[msal-js-publicclientapplication]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-browser/classes/_src_app_publicclientapplication_.publicclientapplication.html
