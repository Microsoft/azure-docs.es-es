---
title: 'Tutorial de aplicación de página única de Angular: Azure'
titleSuffix: Microsoft identity platform
description: Aprenda cómo las aplicaciones de página única de Angular pueden llamar a una API que requiere tokens de acceso de un punto de conexión de la Plataforma de identidad de Microsoft.
services: active-directory
author: hamiltonha
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 03/05/2020
ms.author: hahamil
ms.custom: aaddev, identityplatformtop40, devx-track-javascript
ms.openlocfilehash: a58da8b11876d662173ae83de43d8ed74ab43e93
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88118304"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-api-from-an-angular-single-page-application"></a>Tutorial: Inicio de sesión de usuarios y llamada a Microsoft Graph API desde una aplicación de página única de Angular

En este tutorial se muestra cómo una aplicación de página única (SPA) de Angular puede realizar lo siguiente:
- Iniciar sesión en cuentas personales, profesionales y educativas.
- Obtener un token de acceso.
- Llamar a Microsoft Graph API o a otras API que requieran tokens de acceso del *punto de conexión de la Plataforma de identidad de Microsoft*.

>[!NOTE]
>En este tutorial se explica cómo crear una aplicación de página única de Angular mediante la biblioteca de autenticación de Microsoft (MSAL). Si desea descargar una aplicación de ejemplo, consulte el [inicio rápido](quickstart-v2-angular.md).

## <a name="how-the-sample-app-works"></a>Funcionamiento de la aplicación de ejemplo

![Diagrama que muestra cómo funciona la aplicación de ejemplo generada en este tutorial](./media/tutorial-v2-angular/diagram-auth-flow-spa-angular.svg)

### <a name="more-information"></a>Más información

La aplicación de ejemplo que se crea con este tutorial permite que una aplicación de página única de Angular haga consultas a Microsoft Graph API o a una API web que acepte tokens de un punto de conexión de la Plataforma de identidad de Microsoft. La biblioteca de MSAL para Angular es un contenedor de la biblioteca principal MSAL.js. Permite que las aplicaciones de Angular (6 +) autentiquen a usuarios empresariales mediante Microsoft Azure Active Directory, a usuarios de cuenta de Microsoft y a usuarios de identidades sociales (como Facebook, Google y LinkedIn). La biblioteca también permite que las aplicaciones obtengan acceso a los servicios en la nube de Microsoft o a Microsoft Graph.

En este escenario, después de que un usuario inicia sesión, se solicita un token de acceso y se agrega a las solicitudes HTTP mediante el encabezado de autorización. MSAL controla la adquisición y la renovación de tokens.

### <a name="libraries"></a>Bibliotecas

En este tutorial se usa la siguiente biblioteca:

|Biblioteca|Descripción|
|---|---|
|[msal.js](https://github.com/AzureAD/microsoft-authentication-library-for-js)|Biblioteca de autenticación de Microsoft para el contenedor de Angular de JavaScript|

Puede encontrar el código fuente de la biblioteca MSAL.js en el repositorio [AzureAD/microsoft-authentication-library-for-js](https://github.com/AzureAD/microsoft-authentication-library-for-js) de GitHub.

## <a name="prerequisites"></a>Prerrequisitos

Para ejecutar este tutorial, necesitará:

* Un servidor web local, como [Node.js](https://nodejs.org/en/download/). Las instrucciones de este tutorial se basan en Node.js.
* Un entorno de desarrollo integrado (IDE), como [Visual Studio Code](https://code.visualstudio.com/download), para editar los archivos del proyecto.

## <a name="create-your-project"></a>Creación del proyecto

Genere una nueva aplicación de Angular con los siguientes comandos de npm:

```bash
npm install -g @angular/cli@8                    # Install the Angular CLI
ng new my-application --routing=true --style=css # Generate a new Angular app
cd my-application                                # Change to the app directory
npm install @angular/material@8 @angular/cdk@8   # Install the Angular Material component library (optional, for UI)
npm install msal @azure/msal-angular             # Install MSAL and MSAL Angular in your application
ng generate component page-name                  # To add a new page (such as a home or profile page)
```

## <a name="register-your-application"></a>Registrar su aplicación

Siga las [instrucciones para registrar una aplicación de página única](./scenario-spa-app-registration.md) en Azure Portal.

En la página de **información general** del registro de la aplicación, anote el valor de **Id. de aplicación (cliente)** para su uso posterior.

Registre el valor de **URI de redirección** como **http://localhost:4200/** y habilite la configuración de concesión implícita.

## <a name="configure-the-application"></a>Configuración de la aplicación

1. En la carpeta *src/app*, edite *app.module.ts* y agregue la información de `MSALModule` a `imports`, así como la constante `isIE`:

    ```javascript
    const isIE = window.navigator.userAgent.indexOf('MSIE ') > -1 || window.navigator.userAgent.indexOf('Trident/') > -1;
    @NgModule({
      declarations: [
        AppComponent
      ],
      imports: [
        BrowserModule,
        AppRoutingModule,
        MsalModule.forRoot({
          auth: {
            clientId: 'Enter_the_Application_Id_here', // This is your client ID
            authority: 'Enter_the_Cloud_Instance_Id_Here'/'Enter_the_Tenant_Info_Here', // This is your tenant ID
            redirectUri: 'Enter_the_Redirect_Uri_Here'// This is your redirect URI
          },
          cache: {
            cacheLocation: 'localStorage',
            storeAuthStateInCookie: isIE, // Set to true for Internet Explorer 11
          },
        }, {
          popUp: !isIE,
          consentScopes: [
            'user.read',
            'openid',
            'profile',
          ],
          unprotectedResources: [],
          protectedResourceMap: [
            ['https://graph.microsoft.com/v1.0/me', ['user.read']]
          ],
          extraQueryParameters: {}
        })
      ],
      providers: [],
      bootstrap: [AppComponent]
    })
    ```

    Reemplace estos valores:

    |Nombre del valor|Información|
    |---------|---------|
    |Enter_the_Application_Id_Here|En la página de **información general** del registro de la aplicación, corresponde al valor de **Id. de aplicación (cliente)** . |
    |Enter_the_Cloud_Instance_Id_Here|Esta es la instancia de la nube de Azure. En el caso de la nube principal o global de Azure, escriba **https://login.microsoftonline.com** . Para las nubes nacionales (por ejemplo, China) consulte [Nubes nacionales](./authentication-national-cloud.md).|
    |Enter_the_Tenant_Info_Here| Seleccione una de las siguientes opciones: Si la aplicación admite las *cuentas de este directorio organizativo*, reemplace este valor por el identificador del directorio (inquilino) o por el nombre del inquilino (por ejemplo, **contoso.microsoft.com**). Si la aplicación admite *cuentas en cualquier directorio organizativo*, reemplace este valor por **organizaciones**. Si la aplicación admite *cuentas en cualquier directorio organizativo y cuentas Microsoft personales*, reemplace este valor por **común**. Para restringir la compatibilidad a *Personal Microsoft accounts only* (Solo cuentas Microsoft personales), reemplace este valor por **consumidores**. |
    |Enter_the_Redirect_Uri_Here|Reemplace este valor por **http://localhost:4200** .|

    Para más información acerca de las opciones configurables disponibles, consulte [Inicializar aplicaciones cliente](msal-js-initializing-client-applications.md).

2. En la parte superior del mismo archivo, agregue la siguiente instrucción import:

    ```javascript
    import { MsalModule, MsalInterceptor } from '@azure/msal-angular';
    ```

3. Agregue las siguientes instrucciones import en la parte superior del archivo `src/app/app.component.ts`:

    ```javascript
    import { MsalService, BroadcastService } from '@azure/msal-angular';
    import { Component, OnInit } from '@angular/core';
    ```
## <a name="sign-in-a-user"></a>Inicio de sesión de un usuario

Agregue el código siguiente a `AppComponent` para iniciar sesión con un usuario:

```javascript
export class AppComponent implements OnInit {
    constructor(private broadcastService: BroadcastService, private authService: MsalService) { }

    ngOnInit() { }

    login() {
        const isIE = window.navigator.userAgent.indexOf('MSIE ') > -1 || window.navigator.userAgent.indexOf('Trident/') > -1;

        if (isIE) {
          this.authService.loginRedirect({
            extraScopesToConsent: ["user.read", "openid", "profile"]
          });
        } else {
          this.authService.loginPopup({
            extraScopesToConsent: ["user.read", "openid", "profile"]
          });
        }
    }
}
```

> [!TIP]
> Se recomienda usar `loginRedirect` para los usuarios de Internet Explorer.

## <a name="acquire-a-token"></a>Adquisición de un token

### <a name="angular-interceptor"></a>Interceptor de Angular

MSAL Angular proporciona una clase `Interceptor` que adquiere automáticamente tokens para las solicitudes salientes que usan el cliente `http` de Angular con los recursos protegidos conocidos.

En primer lugar, incluya la clase `Interceptor` como proveedor de la aplicación:

```javascript
import { MsalInterceptor, MsalModule } from "@azure/msal-angular";
import { HTTP_INTERCEPTORS, HttpClientModule } from "@angular/common/http";

@NgModule({
    // ...
    providers: [
        {
            provide: HTTP_INTERCEPTORS,
            useClass: MsalInterceptor,
            multi: true
        }
    ]
}
```

A continuación, proporcione un mapa de los recursos protegidos a `MsalModule.forRoot()` como `protectedResourceMap` e incluya dichos ámbitos en `consentScopes`:

```javascript
@NgModule({
  // ...
  imports: [
    // ...
    MsalModule.forRoot({
      auth: {
        clientId: 'Enter_the_Application_Id_here', // This is your client ID
        authority: 'https://login.microsoftonline.com/Enter_the_Tenant_Info_Here', // This is your tenant info
        redirectUri: 'Enter_the_Redirect_Uri_Here' // This is your redirect URI
      },
      cache: {
        cacheLocation: 'localStorage',
        storeAuthStateInCookie: isIE, // Set to true for Internet Explorer 11
      },
    },
    {
      popUp: !isIE,
      consentScopes: [
        'user.read',
        'openid',
        'profile',
      ],
      unprotectedResources: [],
      protectedResourceMap: [
        ['https://graph.microsoft.com/v1.0/me', ['user.read']]
      ],
      extraQueryParameters: {}
    })
  ],
});
```

Por último, recupere el perfil de un usuario con una solicitud HTTP:

```JavaScript
const graphMeEndpoint = "https://graph.microsoft.com/v1.0/me";

getProfile() {
  this.http.get(graphMeEndpoint).toPromise()
    .then(profile => {
      this.profile = profile;
    });
}
```

### <a name="acquiretokensilent-acquiretokenpopup-acquiretokenredirect"></a>acquireTokenSilent, acquireTokenPopup, acquireTokenRedirect
MSAL utiliza tres métodos para adquirir tokens: `acquireTokenRedirect`, `acquireTokenPopup` y `acquireTokenSilent`. Sin embargo, se recomienda usar la clase `MsalInterceptor` en lugar de las aplicaciones de Angular, tal como se mostró en la sección anterior.

#### <a name="get-a-user-token-silently"></a>Obtención de un token de usuario en silencio

El método `acquireTokenSilent` controla las adquisiciones y la renovación de tokens sin la interacción del usuario. Después de ejecutar el método `loginRedirect` o `loginPopup` por primera vez, se suele utilizar `acquireTokenSilent` para obtener los tokens que se usan para acceder a los recursos protegidos en las llamadas posteriores. Las llamadas para solicitar o renovar tokens se realizan en modo silencioso.

```javascript
const requestObj = {
    scopes: ["user.read"]
};

this.authService.acquireTokenSilent(requestObj).then(function (tokenResponse) {
    // Callback code here
    console.log(tokenResponse.accessToken);
}).catch(function (error) {
    console.log(error);
});
```

En el código anterior, `scopes` contiene los ámbitos cuya devolución se solicita en el token de acceso de la API.

Por ejemplo:

* `["user.read"]` para Microsoft Graph
* `["<Application ID URL>/scope"]` para las API web personalizadas (es decir, `api://<Application ID>/access_as_user`)

#### <a name="get-a-user-token-interactively"></a>Obtención de un token de usuario interactivamente

En ocasiones, necesitará que el usuario interactúe con el punto de conexión de la Plataforma de identidad de Microsoft. Por ejemplo:

* Es posible que los usuarios tengan que volver a escribir las credenciales porque su contraseña ha expirado.
* La aplicación solicita acceso a ámbitos de recursos adicionales para los que se necesita el consentimiento del usuario.
* Se requiere la autenticación en dos fases.

El patrón recomendado para la mayoría de las aplicaciones es llamar primero a `acquireTokenSilent`, después detectar la excepción y, por último, llamar a `acquireTokenPopup` (o a `acquireTokenRedirect`) para iniciar una solicitud interactiva.

La llamada a `acquireTokenPopup` da como resultado una ventana emergente de inicio de sesión. O bien, `acquireTokenRedirect` redirige a los usuarios al punto de conexión de la Plataforma de identidad de Microsoft. En esa ventana, los usuarios tienen que confirmar sus credenciales, dar el consentimiento al recurso requerido o completar la autenticación en dos fases.

```javascript
  const requestObj = {
      scopes: ["user.read"]
  };

  this.authService.acquireTokenPopup(requestObj).then(function (tokenResponse) {
      // Callback code here
      console.log(tokenResponse.accessToken);
  }).catch(function (error) {
      console.log(error);
  });
```

> [!NOTE]
> En este inicio rápido se usan los métodos `loginRedirect` y `acquireTokenRedirect` con Microsoft Internet Explorer, debido a un [problema conocido](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues) relacionado con la manera en que Internet Explorer controla las ventanas emergentes.

## <a name="log-out"></a>Cerrar la sesión

Agregue el código siguiente para cerrar la sesión de un usuario:

```javascript
logout() {
  this.authService.logout();
}
```

## <a name="add-ui"></a>Incorporación de la interfaz de usuario
Consulte en la [aplicación de ejemplo](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular) una muestra de cómo agregar una interfaz de usuario mediante la biblioteca de componentes Angular Material.

## <a name="test-your-code"></a>Prueba del código

1.  Inicie el servidor web para que escuche el puerto; para ello, ejecute los siguientes comandos desde la carpeta de la aplicación en un símbolo del sistema de la línea de comandos:

    ```bash
    npm install
    npm start
    ```
1. En el explorador, escriba **http://localhost:4200** o **http://localhost:{port}** , donde *port* es el puerto en el que escucha el servidor web.


### <a name="provide-consent-for-application-access"></a>Consentimiento para el acceso a la aplicación

La primera vez que inicie sesión en la aplicación, se le pedirá que le conceda acceso a su perfil y que le permita iniciar sesión:

![La ventana "Permisos solicitados"](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptspaconsent.png)

## <a name="add-scopes-and-delegated-permissions"></a>Incorporación de ámbitos y permisos delegados

Microsoft Graph API requiere el ámbito *user.read* para leer el perfil del usuario. De forma predeterminada, este ámbito se agrega automáticamente en todas las aplicaciones que se registran en el portal de registro. Otras API de Microsoft Graph, así como las API personalizadas para el servidor back-end, pueden requerir ámbitos adicionales. Por ejemplo, Microsoft Graph API requiere el ámbito *Calendars.Read* para mostrar los calendarios del usuario.

Para acceder a los calendarios del usuario en el contexto de una aplicación, agregue el permiso delegado *Calendars.Read* a la información del registro de la aplicación. A continuación, agregue el ámbito *Calendars.Read* a la llamada `acquireTokenSilent`.

>[!NOTE]
>Es posible que se pida al usuario algún consentimiento adicional a medida que aumente el número de ámbitos.

Si una API de back-end no requiere un ámbito (no se recomienda), puede usar *clientId* como ámbito en las llamadas de adquisición de tokens.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Pasos siguientes

Si no está familiarizado con la administración de identidades y la administración del acceso, disponemos de varios artículos que le ayudarán a conocer los conceptos de autenticación modernos, empezando por [autenticación frente a autorización](authentication-vs-authorization.md).

Si quiere profundizar más en el desarrollo de aplicaciones de una sola página en la Plataforma de identidad de Microsoft, la serie de varias partes [Escenario: Aplicación de página única](scenario-spa-overview.md) puede ayudarle a empezar.