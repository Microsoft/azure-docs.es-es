---
title: Inicio de sesión único (biblioteca de autenticación de Microsoft para JavaScript) | Azure
description: Obtenga información sobre la creación de las experiencias de inicio de sesión únicas mediante la biblioteca de autenticación de Microsoft para JavaScript (MSAL.js).
services: active-directory
documentationcenter: dev-center-name
author: navyasric
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/24/2019
ms.author: nacanuma
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 9f1f102307256852ac92616c7fb707e0e2739e5d
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544157"
---
# <a name="single-sign-on-with-msaljs"></a>Inicio de sesión único con MSAL.js

Inicio de sesión único (SSO) permite a los usuarios que escriban sus credenciales una vez para iniciar sesión y establecer una sesión que se puede reutilizar en varias aplicaciones sin necesidad de volver a autenticarse. Esto ofrece una experiencia al usuario y reduce las peticiones repetidas de credenciales.

Azure AD proporciona las capacidades de inicio de sesión único a las aplicaciones mediante el establecimiento de una cookie de sesión cuando el usuario autentica por primera vez. La biblioteca MSAL.js permite que las aplicaciones que se aprovecha de varias maneras.

## <a name="sso-between-browser-tabs"></a>Inicio de sesión único entre las pestañas del explorador

Cuando la aplicación se abre en varias pestañas y primero iniciar sesión el usuario en una ficha, el usuario es también firmado en las demás pestañas sin que se le solicite. MSAL.js almacena en caché el token de identificador para el usuario en el explorador `localStorage` y va a iniciar sesión el usuario la aplicación en las otras pestañas abiertas.

De forma predeterminada, usa MSAL.js `sessionStorage` que no permite la sesión para compartirse entre distintas pestañas. Para configurar el inicio de sesión único entre las fichas, asegúrese de establecer el `cacheLocation` en MSAL.js a `localStorage` tal como se muestra a continuación.

```javascript
const config = {
    auth: {
        clientId: “abcd-ef12-gh34-ikkl-ashdjhlhsdg”
    },
    cache: {
        cacheLocation: 'localStorage'
    }
}

const myMSALObj = new UserAgentApplication(config);
```

## <a name="sso-between-apps"></a>Inicio de sesión único entre aplicaciones

Cuando un usuario se autentica, se establece una cookie de sesión en el dominio de Azure AD en el explorador. MSAL.js se basa en esta cookie de sesión para proporcionar inicio de sesión único para el usuario entre diferentes aplicaciones. MSAL.js también almacena en caché los tokens de identificador y tokens de acceso del usuario en el almacenamiento del explorador por dominio de aplicación. Como resultado, el comportamiento de inicio de sesión único varía para los distintos casos:  

### <a name="applications-on-the-same-domain"></a>Aplicaciones en el mismo dominio

Cuando las aplicaciones se hospedan en el mismo dominio, el usuario puede iniciar sesión en una aplicación una vez y, a continuación, se autentica a las demás aplicaciones sin un símbolo del sistema. MSAL.js aprovecha los tokens en caché para que el usuario en el dominio proporcionar SSO.

### <a name="applications-on-different-domain"></a>Aplicaciones en un dominio diferente

Cuando las aplicaciones se hospedan en dominios diferentes, los tokens en caché en un dominio no pueden tener acceso MSAL.js en el dominio B.

Esto significa que cuando los usuarios sesión en el dominio desplazarse a una aplicación en el dominio B, que se redirigen o se le solicite la página de Azure AD. Puesto que Azure AD tiene todavía la cookie de sesión de usuario, iniciará sesión en el usuario y no tendrá que volver a escribir las credenciales. Si el usuario tiene varias cuentas de usuario en la sesión con Azure AD, se pedirá al usuario para elegir la cuenta para iniciar sesión con pertinente.

### <a name="automatically-select-account-on-azure-ad"></a>Seleccionar automáticamente la cuenta en Azure AD

En algunos casos, la aplicación tenga acceso al contexto de autenticación del usuario y desea evitar el mensaje de selección de la cuenta de Azure AD cuando varias cuentas han iniciado sesión.  Esto puede hacerse varias maneras diferentes:

**Uso de Id. de sesión (SID)**

Identificador de sesión es un [notificación opcional](active-directory-optional-claims.md) que se pueden configurar en los tokens de identificador. Esta notificación permite que la aplicación identificar sesión de AD independiente del nombre de la cuenta del usuario o nombre de usuario de Azure del usuario. Puede pasar el SID de los parámetros de solicitud a la `acquireTokenSilent` llamar. Esto permitirá que Azure AD para omitir la selección de cuenta. SID depende de la cookie de sesión y no cruce contextos de explorador.

```javascript
var request = {
    scopes: ["user.read"],
    sid: sid
}

userAgentApplication.acquireTokenSilent(request).then(function(response) {
        const token = response.accessToken
    }
).catch(function (error) {  
        //handle error
});
```

> [!Note]
> SID puede utilizarse solo con las solicitudes de autenticación en modo silencioso realizadas por `acquireTokenSilent` llamar en MSAL.js.
Puede encontrar los pasos para configurar notificaciones opcionales en el manifiesto de aplicación [aquí](active-directory-optional-claims.md).

**Usar la sugerencia de inicio de sesión**

Si no tienen SID configurado de notificación o deberá omitir la confirmación de selección de cuenta en las llamadas de autenticación interactiva, puede hacerlo al proporcionar un `login_hint` en los parámetros de solicitud y, opcionalmente, un `domain_hint` como `extraQueryParameters` en el MSAL.js los métodos interactivos (`loginPopup`, `loginRedirect`, `acquireTokenPopup` y `acquireTokenRedirect`). Por ejemplo:

```javascript
var request = {
    scopes: ["user.read"],
    loginHint: preferred_username,
    extraQueryParameters: {domain_hint: 'organizations'}
}

userAgentApplication.loginRedirect(request);
```

Puede obtener los valores para login_hint y domain_hint mediante la lectura de las notificaciones devueltas en el token de identificador para el usuario.

* **loginHint** debe establecerse en el `preferred_username` del token de identificador de notificación.

* **domain_hint** solo es necesario que se pasarán al utilizar el/Common autoridad. La sugerencia de dominio viene determinada por el inquilino ID(tid).  Si el `tid` notificación del token de identificador es `9188040d-6c67-4c5b-b112-36a304b66dad` es los consumidores. En caso contrario, es que las organizaciones.

Lectura [aquí](v2-oauth2-implicit-grant-flow.md) para obtener más información sobre los valores de la sugerencia de inicio de sesión y la sugerencia de dominio.

> [!Note]
> No se puede pasar el SID y login_hint al mismo tiempo. Esto dará como resultado de la respuesta de error.

## <a name="sso-without-msaljs-login"></a>Inicio de sesión único sin inicio de sesión MSAL.js

Por diseño, MSAL.js requiere que se llama a un método de inicio de sesión para establecer un contexto de usuario antes de obtener tokens para las API. Puesto que los métodos de inicio de sesión interactivos, el usuario ve un símbolo del sistema.

Hay algunos casos en que las aplicaciones tienen acceso al contexto del usuario autenticado o token de identificador mediante la autenticación se inicia en otra aplicación y desea aprovechar SSO para adquirir tokens sin la primera sesión a través de MSAL.js.

Un ejemplo de esto es: Un usuario ha iniciado sesión en una aplicación web de principal que hospeda a otra aplicación de JavaScript que se ejecuta como un complemento o un complemento.

La experiencia de inicio de sesión único en este escenario puede lograrse como sigue:

Pase el `sid` si está disponible (o `login_hint` y, opcionalmente, `domain_hint`) como solicitar parámetros a la MSAL.js `acquireTokenSilent` llame como sigue:

```javascript
var request = {
    scopes: ["user.read"],
    loginHint: preferred_username,
    extraQueryParameters: {domain_hint: 'organizations'}
}

userAgentApplication.acquireTokenSilent(request).then(function(response) {
        const token = response.accessToken
    }
).catch(function (error) {  
        //handle error
});
```

## <a name="sso-in-adaljs-to-msaljs-update"></a>Inicio de sesión único en ADAL.js MSAL.js Update

MSAL.js aporta paridad de características con ADAL.js para escenarios de autenticación de Azure AD. Para facilitar la migración de ADAL.js a MSAL.js y evitar pedir a los usuarios volver a iniciar sesión, la biblioteca lee el token de identificador que representa la sesión del usuario en la memoria caché de ADAL.js y perfectamente inicia sesión el usuario MSAL.js.  

Para poder aprovechar el comportamiento de inicio de sesión único (SSO) cuando se actualizan desde ADAL.js, deberá asegurarse de que están utilizando las bibliotecas `localStorage` para almacenar en caché los tokens. Establecer el `cacheLocation` a `localStorage` en la MSAL.js ADAL.js configuración y en la inicialización de la manera siguiente:


```javascript
// In ADAL.js
window.config = {
   clientId: 'g075edef-0efa-453b-997b-de1337c29185',
   cacheLocation: 'localStorage'
};

var authContext = new AuthenticationContext(config);


// In latest MSAL.js version
const config = {
    auth: {
        clientId: “abcd-ef12-gh34-ikkl-ashdjhlhsdg”
    },
    cache: {
        cacheLocation: 'localStorage'
    }
}

const myMSALObj = new UserAgentApplication(config);
```

Una vez configurado, MSAL.js podrá leer el estado almacenado en caché del usuario autenticado en ADAL.js y usarlo para proporcionar inicio de sesión único en MSAL.js.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre la [único inicio de sesión y la duración del token](active-directory-configurable-token-lifetimes.md) valores en Azure AD.
