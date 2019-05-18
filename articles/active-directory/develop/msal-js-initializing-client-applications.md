---
title: Inicializar las aplicaciones cliente (biblioteca de autenticación de Microsoft para JavaScript) | Azure
description: Obtenga información sobre la inicialización de aplicaciones cliente utilizan la biblioteca de autenticación de Microsoft para JavaScript (MSAL.js).
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/12/2019
ms.author: nacanuma
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: cd26f36356affbc8c272bd093757a8482773baf2
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544025"
---
# <a name="initialize-client-applications-using-msaljs"></a>Inicializar aplicaciones cliente que utilizan MSAL.js
Este artículo describe al inicializar Microsoft Authentication Library para JavaScript (MSAL.js) con una instancia de una aplicación de agente de usuario. La aplicación de agente de usuario es un formulario de aplicación de cliente público en el que se ejecuta el código de cliente en un agente de usuario como un explorador web. Estos clientes no almacenan secretos, puesto que el contexto del explorador es accesible públicamente. Para obtener más información sobre los tipos de aplicaciones cliente y las opciones de configuración de aplicación, lea el [Introducción](msal-client-applications.md).

## <a name="prerequisites"></a>Requisitos previos
Antes de inicializar una aplicación, primero deberá [registrarlo en el portal de Azure](scenario-spa-app-registration.md) para que la aplicación se puede integrar con la plataforma Microsoft identity. Después del registro, puede que necesite la información siguiente (que puede encontrarse en el portal de Azure):

- El identificador de cliente (es decir, una cadena que representa un GUID para la aplicación)
- La URL del proveedor de identidades (la instancia con nombre) y la audiencia de inicio de sesión para la aplicación. Estos dos parámetros se conocen colectivamente como la entidad.
- El identificador del inquilino si está escribiendo una aplicación de línea de negocio únicamente para su organización (también aplicación de inquilino único con nombre).
- Para las aplicaciones web, debe establecer también el redirectUri que devolverá el proveedor de identidades a su aplicación con los tokens de seguridad.

## <a name="initializing-applications"></a>Inicialización de aplicaciones

Puede usar MSAL.js como se indica a continuación en una aplicación de JavaScript/Typescript sin formato. Inicializar el contexto de autenticación de MSAL creando `UserAgentApplication` con un objeto de configuración. La configuración mínima necesaria para inicializar MSAL.js es el valor clientID de la aplicación que se debe obtener en el portal de registro de aplicación.

Para los métodos de autenticación con redirección flujos (`loginRedirect` y `acquireTokenRedirect`), deberá registrar una devolución de llamada para el éxito o error a través de forma explícita `handleRedirectCallback()` método. Esto es necesario puesto que los flujos de redirección no devuelven promesas igual que los métodos con una experiencia emergente.

```javascript
// Configuration object constructed
const config = {
    auth: {
        clientId: “abcd-ef12-gh34-ikkl-ashdjhlhsdg”
    }
}

// create UserAgentApplication instance
const myMSALObj = new UserAgentApplication(config);

function authCallback(error, response) {
    //handle redirect response
}

// (optional when using redirect methods) register redirect call back for Success or Error
myMSALObj.handleRedirectCallback(authCallback);
```

MSAL.js está diseñado para tener una sola instancia y la configuración de la `UserAgentApplication` para representar un contexto de autenticación único. No se recomiendan varias instancias porque hacen que las entradas de caché en conflicto y el comportamiento en el explorador.

## <a name="configuration-options"></a>Opciones de configuración

MSAL.js tiene una configuración de objeto que se muestra a continuación que proporciona una agrupación de opciones configurables disponibles para crear una instancia de `UserAgentApplication`.

```javascript
type storage = "localStorage" | "sessionStorage";

// Protocol Support
export type AuthOptions = {
    clientId: string;
    authority?: string;
    validateAuthority?: boolean;
    redirectUri?: string | (() => string);
    postLogoutRedirectUri?: string | (() => string);
    navigateToLoginRequestUrl?: boolean;
};

// Cache Support
export type CacheOptions = {
    cacheLocation?: CacheLocation;
    storeAuthStateInCookie?: boolean;
};

// Library support
export type SystemOptions = {
    logger?: Logger;
    loadFrameTimeout?: number;
    tokenRenewalOffsetSeconds?: number;
};

// Developer App Environment Support
export type FrameworkOptions = {
    isAngular?: boolean;
    unprotectedResources?: Array<string>;
    protectedResourceMap?: Map<string, Array<string>>;
};

// Configuration Object
export type Configuration = {
    auth: AuthOptions,
    cache?: CacheOptions,
    system?: SystemOptions,
    framework?: FrameworkOptions
};
```

A continuación es el conjunto total de opciones configurables que se admiten actualmente en el objeto de configuración:

- **clientID**: Necesario. ClientID de la aplicación, se debe obtener desde el portal de registro de aplicación.

- **authority**: Opcional. Una dirección URL que indica un directorio que MSAL puede solicitar tokens de. El valor predeterminado es `https://login.microsoftonline.com/common`.
    * En Azure AD, es del formato https://&lt;instancia&gt;/&lt;audiencia&gt;, donde &lt;instancia&gt; es el dominio del proveedor de identidades (por ejemplo, `https://login.microsoftonline.com`) y &lt;audiencia&gt; es un identificador que representa la audiencia de inicio de sesión. Esto puede ser los siguientes valores:
        * `https://login.microsoftonline.com/<tenant>`-inquilino es un dominio asociado al inquilino, por ejemplo, contoso.onmicrosoft.com, o el GUID que representa el `TenantID` propiedad del directorio usado solo para iniciar sesión en los usuarios de una organización específica.
        * `https://login.microsoftonline.com/common`: Se usa para iniciar sesión en los usuarios con el trabajo y cuentas educativas o una cuenta personal de Microsoft.
        * `https://login.microsoftonline.com/organizations/`: Se usa para iniciar sesión en los usuarios con cuentas profesionales y educativas.
        * `https://login.microsoftonline.com/consumers/` : Se usa para iniciar sesión en los usuarios con solo cuentas de Microsoft (live).
    * En Azure AD B2C, tiene el formato `https://<instance>/tfp/<tenant>/<policyName>/`, donde instancia es el dominio de Azure AD B2C, inquilino es el nombre del inquilino de Azure AD B2C, policyName es el nombre de la directiva de B2C para aplicar.


- **validateAuthority**: Opcional.  Valide al emisor de tokens. El valor predeterminado es `true`. Para las aplicaciones B2C, ya que el valor de autoridad se conoce y puede ser diferente según la directiva, la validación de la entidad de certificación no funcionará y tiene que establecerse en `false`.

- **redirectUri**: Opcional.  El URI de redireccionamiento de la aplicación, adonde la aplicación puede enviar y recibir las respuestas de autenticación. Debe coincidir exactamente con uno de los URI de redireccionamiento que registró en el portal, con la excepción de que debe estar codificado como URL. Tiene como valor predeterminado `window.location.href`.

- **postLogoutRedirectUri**: Opcional.  Redirige al usuario a `postLogoutRedirectUri` después de cierre de sesión. El valor predeterminado es `redirectUri`.

- **navigateToLoginRequestUrl**: Opcional. Capacidad de desactivar la navegación de forma predeterminada a la página de inicio después de iniciar sesión. El valor predeterminado es true. Esto se usa solo para los flujos de redirección.

- **cacheLocation**: Opcional.  Establece el almacenamiento del explorador en `localStorage` o `sessionStorage`. El valor predeterminado es `sessionStorage`.

- **storeAuthStateInCookie**: Opcional.  Esta marca se introdujo en MSAL.js v0.2.2 como una corrección para el [problemas de autenticación de bucle](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#1-issues-due-to-security-zones) en Microsoft Internet Explorer y Microsoft Edge. Habilitar la marca `storeAuthStateInCookie` a True para aprovechar las ventajas de esta corrección. Cuando esta opción está habilitada, MSAL.js almacenará el estado de la solicitud de autenticación necesario para la validación de los flujos de autenticación en las cookies del explorador. De forma predeterminada, esta marca se establece en `false`.

- **logger**: Opcional.  Un objeto de registrador con una instancia de devolución de llamada que puede proporcionar al desarrollador consuma y publique los registros de una manera personalizada. Para obtener más información sobre cómo pasar el objeto de registrador, vea [registro con msal.js](msal-logging.md).

- **loadFrameTimeout**: Opcional.  Se agotó el número de milisegundos de inactividad antes de que se debe considerar una respuesta de renovación del token de Azure AD. Valor predeterminado es 6 segundos.

- **tokenRenewalOffsetSeconds**: Opcional. El número de milisegundos que establece la ventana de desplazamiento es necesario renovar el token antes de que expire. Valor predeterminado es 300 milisegundos.

Solo son aplicables a pasarse desde la biblioteca MSAL Angular del contenedor:
- **unprotectedResources**: Opcional.  Matriz de identificadores URI que son los recursos desprotegidos. MSAL no asociará un token a las solicitudes salientes que tienen estos URI. Tiene como valor predeterminado `null`.

- **protectedResourceMap**: Opcional.  Esta es la asignación de recursos a los ámbitos que usa MSAL para asociar automáticamente los tokens de acceso en las llamadas de API web. Se obtiene un token de acceso único para el recurso. Para que pueda asignar una ruta de acceso de recurso específico como sigue: {"https://graph.microsoft.com/v1.0/me", ["user.read"]}, o la dirección URL de aplicación del recurso como: {"https://graph.microsoft.com/", ["user.read", "mail.send"]}. Esto es necesario para las llamadas de la CORS. Tiene como valor predeterminado `null`.
