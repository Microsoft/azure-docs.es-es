---
title: Inicialización de aplicaciones cliente de MSAL.js | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre la inicialización de aplicaciones cliente mediante la biblioteca de autenticación de Microsoft para JavaScript (MSAL.js).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/17/2020
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 5af5d3a88262792f4b32e2ce3d8143ac680f083a
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87027043"
---
# <a name="initialize-client-applications-using-msaljs"></a>Inicialización de aplicaciones cliente con MSAL.js

En este artículo se describe la inicialización de la biblioteca de autenticación de Microsoft para JavaScript (MSAL.js) con una instancia de una aplicación de agente de usuario.

La aplicación de agente de usuario es una forma de aplicación cliente pública en la que el código cliente se ejecuta en un agente de usuario, como un explorador web. Tales clientes no almacenan secretos, porque el contexto del explorador es de acceso público.

Para obtener más información sobre los tipos de aplicaciones cliente y las opciones de configuración de las aplicaciones, consulte [Aplicaciones cliente públicas y confidenciales en MSAL](msal-client-applications.md).

## <a name="prerequisites"></a>Prerrequisitos

Antes de inicializar una aplicación, primero tendrá que [registrarla en Azure Portal](scenario-spa-app-registration.md), estableciendo una relación de confianza entre la aplicación y la Plataforma de identidad de Microsoft.

Después de registrar la aplicación, necesitará la totalidad o parte de los valores siguientes que se pueden encontrar en Azure Portal.

| Value | Obligatorio | Descripción |
|:----- | :------: | :---------- |
| Id. de aplicación (cliente) | Obligatorio | GUID que identifica de forma única la aplicación en la Plataforma de identidad de Microsoft. |
| Autoridad | Opcional | La URL del proveedor de identidades (la *instancia*) y la *audiencia de inicio de sesión* para la aplicación. La instancia y la audiencia de inicio de sesión, cuando se concatenan, constituyen la *autoridad*. |
| Id. de directorio (inquilino) | Opcional | Especifíquelo si va a compilar una aplicación de línea de negocio exclusivamente para su organización, en ocasiones denominada *aplicación de un único inquilino*. |
| URI de redireccionamiento | Opcional | Si va a compilar una aplicación web, `redirectUri` especifica a dónde debe devolver el proveedor de identidades (la Plataforma de identidad de Microsoft) los tokens de seguridad que ha emitido. |

## <a name="initialize-msaljs-2x-apps"></a>Inicialización de aplicaciones MSAL.js 2.x

Para inicializar el contexto de autenticación de MSAL, cree una instancia de [PublicClientApplication][msal-js-publicclientapplication] con un objeto [Configuration][msal-js-configuration]. La propiedad de configuración mínima necesaria es el `clientID` de la aplicación, que se muestra como el **Id. de aplicación (cliente)** en la página **Información general** del registro de la aplicación en Azure Portal.

Este es un ejemplo de objeto Configuration y creación de instancias de `PublicClientApplication`:

```javascript
const msalConfig = {
    auth: {
        clientId: "11111111-1111-1111-111111111111",
        authority: "https://login.microsoftonline.com/common",
        knownAuthorities: [],
        redirectUri: "https://localhost:3001",
        postLogoutRedirectUri: "https://localhost:3001/logout",
        navigateToLoginRequestUrl: true
    },
    cache: {
        cacheLocation: "sessionStorage",
        storeAuthStateInCookie: false
    },
    system: {
        loggerOptions: {
            loggerCallback: (level: LogLevel, message: string, containsPii: boolean): void => {
                if (containsPii) {
                    return;
                }
                switch (level) {
                    case LogLevel.Error:
                        console.error(message);
                        return;
                    case LogLevel.Info:
                        console.info(message);
                        return;
                    case LogLevel.Verbose:
                        console.debug(message);
                        return;
                    case LogLevel.Warning:
                        console.warn(message);
                        return;
                }
            },
            piiLoggingEnabled: false
        },
        windowHashTimeout: 60000,
        iframeHashTimeout: 6000,
        loadFrameTimeout: 0
    }
};

// Create an instance of PublicClientApplication
const msalInstance = new PublicClientApplication(msalConfig);

// Handle the redirect flows
msalInstance.handleRedirectPromise().then((tokenResponse) => {
    // Handle redirect response
}).catch((error) => {
    // Handle redirect error
});
```

### `handleRedirectPromise`

Invoque [handleRedirectPromise][msal-js-handleredirectpromise] cuando la aplicación usa los flujos de redireccionamiento. Al usar los flujos de redireccionamiento, se debe ejecutar `handleRedirectPromise` en la carga de cada página.

Hay tres posibles resultados de la promesa:

- Se invoca a `.then` y `tokenResponse` es verdadero: La aplicación vuelve de una operación de redireccionamiento que se ha completado correctamente.
- Se invoca a `.then` y `tokenResponse` es falso (`null`): La aplicación no vuelve de una operación de redireccionamiento.
- Se invoca a `.catch`: La aplicación vuelve de una operación de redireccionamiento y hubo un error.

## <a name="initialize-msaljs-1x-apps"></a>Inicialización de aplicaciones MSAL.js 1.x

Para inicializar el contexto de autenticación de MSAL 1.x, cree una instancia de [UserAgentApplication][msal-js-useragentapplication] con un objeto Configuration. La propiedad de configuración mínima necesaria es el `clientID` de la aplicación, que se muestra como el **Id. de aplicación (cliente)** en la página **Información general** del registro de la aplicación en Azure Portal.

Para los métodos de autenticación con flujos de redireccionamiento ([loginRedirect][msal-js-loginredirect] y [acquireTokenRedirect][msal-js-acquiretokenredirect]), en MSAL.js 1.2.x o en versiones anteriores, tendrá que registrar de forma explícita una devolución de llamada para los casos correctos o con error mediante el método `handleRedirectCallback()`. El registro explícito de la devolución de llamada es necesario en MSAL.js 1.2.x y versiones anteriores, ya que los flujos de redireccionamiento no devuelven promesas, como sí lo hacen los métodos con una experiencia emergente. El registro de la devolución de llamada es *opcional* en MSAL.js versión 1.3.x y versiones posteriores.

```javascript
// Configuration object constructed
const msalConfig = {
    auth: {
        clientId: "11111111-1111-1111-111111111111"
    }
}

// Create UserAgentApplication instance
const msalInstance = new UserAgentApplication(msalConfig);

function authCallback(error, response) {
    // Handle redirect response
}

// Register a redirect callback for Success or Error (when using redirect methods)
// **REQUIRED** in MSAL.js 1.2.x and earlier
// **OPTIONAL** in MSAL.js 1.3.x and later
msalInstance.handleRedirectCallback(authCallback);
```

## <a name="single-instance-and-configuration"></a>Instancia y configuración únicas

Tanto MSAL.js 1.x como 2.x están diseñados para tener una sola instancia y configuración de `UserAgentApplication` o `PublicClientApplication`, respectivamente, para representar un único contexto de autenticación.

No se recomienda el uso de varias instancias de `UserAgentApplication` o `PublicClientApplication`, ya que generan entradas de caché y comportamientos en conflicto en el explorador.

## <a name="next-steps"></a>Pasos siguientes

Este ejemplo de código de MSAL.js 2.x en GitHub muestra la creación de instancias de [PublicClientApplication][msal-js-publicclientapplication] con un objeto [Configuration][msal-js-configuration]:

[Azure-Samples/ms-identity-javascript-v2](https://github.com/Azure-Samples/ms-identity-javascript-v2)

<!-- LINKS - External -->
[msal-browser]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-browser/
[msal-core]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/
[msal-js-acquiretokenredirect]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/classes/_useragentapplication_.useragentapplication.html#acquiretokenredirect
[msal-js-configuration]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/modules/_configuration_.html
[msal-js-handleredirectpromise]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-browser/classes/_src_app_publicclientapplication_.publicclientapplication.html#handleredirectpromise
[msal-js-loginredirect]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/classes/_useragentapplication_.useragentapplication.html#loginredirect
[msal-js-publicclientapplication]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-browser/classes/_src_app_publicclientapplication_.publicclientapplication.html
[msal-js-useragentapplication]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/modules/_useragentapplication_.html
