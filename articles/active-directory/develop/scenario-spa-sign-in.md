---
title: 'Inicio y cierre de sesión en una aplicación de página única: Plataforma de identidad de Microsoft | Azure'
description: Aprenda a crear una aplicación de página única (inicio de sesión)
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 02/11/2020
ms.author: nacanuma
ms.custom: aaddev
ms.openlocfilehash: c8ec151c813bfb0b9777e583a4ea5144e3b2079a
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297066"
---
# <a name="single-page-application-sign-in-and-sign-out"></a>Aplicación de página única: Inicio y cierre de sesión

Aprenda a agregar el inicio de sesión al código de la aplicación de página única.

Antes de obtener tokens para acceder a las API de la aplicación, necesita un contexto de usuario autenticado. Puede iniciar la sesión de los usuarios en la aplicación con MSAL.js de dos maneras:

* [Ventana emergente](#sign-in-with-a-pop-up-window), mediante el método `loginPopup`
* [Redirección](#sign-in-with-redirect), mediante el método `loginRedirect`

Opcionalmente, también puede pasar los ámbitos de las API para las que necesita que el usuario dé su consentimiento en el momento de iniciar sesión.

> [!NOTE]
> Si su aplicación ya tiene acceso a un contexto de usuario autenticado o un token de identificación, puede omitir el paso de inicio de sesión y adquirir los tokens directamente. Para más información, consulte [Inicio de sesión único sin inicio de sesión en MSAL.js](msal-js-sso.md#sso-without-msaljs-login).

## <a name="choosing-between-a-pop-up-or-redirect-experience"></a>Elegir entre una experiencia de elemento emergente o redireccionamiento

No puede usar juntos los métodos de ventana emergente y redirección en la aplicación. La elección de una u otra experiencia depende del flujo de la aplicación:

* Si no quiere que los usuarios salgan de la página principal de la aplicación durante la autenticación, se recomienda el método de ventana emergente. Dado que la redirección de la autenticación tiene lugar en una ventana emergente, se conserva el estado de la aplicación principal.

* Si los usuarios tienen restricciones del explorador o directivas donde las ventanas emergentes están deshabilitadas, puede usar el método de redirección. Use el método de redirección con Internet Explorer, ya que hay [problemas conocidos con las ventanas emergentes en este explorador](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser).

## <a name="sign-in-with-a-pop-up-window"></a>Inicio de sesión con una ventana emergente


# <a name="javascript-msaljs-2x"></a>[JavaScript (MSAL.js 2.x)](#tab/javascript2)

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

let username = "";

const myMsal = new PublicClientApplication(config);

myMsal.loginPopup(loginRequest)
    .then(function (loginResponse) {
        //login success

        // In case multiple accounts exist, you can select
        const currentAccounts = myMsal.getAllAccounts();
    
        if (currentAccounts === null) {
            // no accounts detected
        } else if (currentAccounts.length > 1) {
            // Add choose account code here
        } else if (currentAccounts.length === 1) {
            username = currentAccounts[0].username;
        }
    
    }).catch(function (error) {
        //login failure
        console.log(error);
    });
```

# <a name="javascript-msaljs-1x"></a>[JavaScript (MSAL.js 1.x)](#tab/javascript1)

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

const myMsal = new UserAgentApplication(config);

myMsal.loginPopup(loginRequest)
    .then(function (loginResponse) {
        //login success
    }).catch(function (error) {
        //login failure
        console.log(error);
    });
```

# <a name="angular"></a>[Angular](#tab/angular)

El contenedor MSAL Angular le permite proteger rutas específicas de la aplicación con solo agregar el valor `MsalGuard` a la definición de ruta. Esta protección invocará al método para iniciar sesión cuando se tenga acceso a esa ruta.

```javascript
// In app-routing.module.ts
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';
import { ProfileComponent } from './profile/profile.component';
import { MsalGuard } from '@azure/msal-angular';
import { HomeComponent } from './home/home.component';

const routes: Routes = [
  {
    path: 'profile',
    component: ProfileComponent,
    canActivate: [
      MsalGuard
    ]
  },
  {
    path: '',
    component: HomeComponent
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes, { useHash: false })],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

Para usar la experiencia de ventana emergente, habilite la opción de configuración `popUp`. También puede pasar los ámbitos que requieren el consentimiento tal como se detalla a continuación:

```javascript
// In app.module.ts
@NgModule({
    imports: [
        MsalModule.forRoot({
            auth: {
                clientId: 'your_app_id',
            }
        }, {
            popUp: true,
            consentScopes: ["User.ReadWrite"]
        })
    ]
})
```
---

## <a name="sign-in-with-redirect"></a>Inicio de sesión con redirección

# <a name="javascript-msaljs-2x"></a>[JavaScript (MSAL.js 2.x)](#tab/javascript2)

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

let username = "";

const myMsal = new PublicClientApplication(config);

function handleResponse(response) {
    //handle redirect response

    // In case multiple accounts exist, you can select
    const currentAccounts = myMsal.getAllAccounts();

    if (currentAccounts === null) {
        // no accounts detected
    } else if (currentAccounts.length > 1) {
        // Add choose account code here
    } else if (currentAccounts.length === 1) {
        username = currentAccounts[0].username;
    }
}

myMsal.handleRedirectPromise(handleResponse);

myMsal.loginRedirect(loginRequest);
```

# <a name="javascript-msaljs-1x"></a>[JavaScript (MSAL.js 1.x)](#tab/javascript1)

Los métodos de redirección no devuelven una promesa, dado que se abandona la aplicación principal. Para procesar los tokens devueltos y acceder a ellos, debe registrar las devoluciones de llamada correctas y con error antes de llamar a los métodos de redirección.

```javascript

const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const loginRequest = {
    scopes: ["User.ReadWrite"]
}

const myMsal = new UserAgentApplication(config);

function authCallback(error, response) {
    //handle redirect response
}

myMsal.handleRedirectCallback(authCallback);

myMsal.loginRedirect(loginRequest);
```

# <a name="angular"></a>[Angular](#tab/angular)

El código aquí es el mismo que el que se describió anteriormente en la sección sobre el inicio de sesión con una ventana emergente. El flujo predeterminado es de redirección.

---

## <a name="sign-out"></a>Cierre de sesión

La biblioteca MSAL proporciona un método `logout` que borra la caché en el almacenamiento del explorador y envía una solicitud de cierre de sesión a Azure Active Directory (Azure AD). Después de cerrar la sesión, la biblioteca redirige de vuelta a la página de inicio de la aplicación de forma predeterminada.

Puede configurar el URI de redirección tras el cierre de sesión mediante el establecimiento del valor `postLogoutRedirectUri`. Este URI también se debe registrar como URI de cierre de sesión en el registro de la aplicación.

# <a name="javascript-msaljs-2x"></a>[JavaScript (MSAL.js 2.x)](#tab/javascript2)

```javascript
const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const myMsal = new PublicClientApplication(config);

// you can select which account application should sign out
const logoutRequest = {
    account: myMsal.getAccountByUsername(username)
}

myMsal.logout(logoutRequest);
```

# <a name="javascript-msaljs-1x"></a>[JavaScript (MSAL.js 1.x)](#tab/javascript1)

```javascript
const config = {
    auth: {
        clientId: 'your_app_id',
        redirectUri: "your_app_redirect_uri", //defaults to application start page
        postLogoutRedirectUri: "your_app_logout_redirect_uri"
    }
}

const myMsal = new UserAgentApplication(config);

myMsal.logout();
```

# <a name="angular"></a>[Angular](#tab/angular)

```javascript
//In app.module.ts
@NgModule({
    imports: [
        MsalModule.forRoot({
            auth: {
                clientId: 'your_app_id',
                postLogoutRedirectUri: "your_app_logout_redirect_uri"
            }
        })
    ]
})

// In app.component.ts
this.authService.logout();
```

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Adquisición de un token para la aplicación](scenario-spa-acquire-token.md)
