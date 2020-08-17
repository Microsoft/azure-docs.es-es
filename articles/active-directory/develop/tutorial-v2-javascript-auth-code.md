---
title: 'Tutorial de aplicaciones de página única de JavaScript: flujo de código de autorización | Azure'
titleSuffix: Microsoft identity platform
description: Se describe cómo las aplicaciones SPA de JavaScript pueden usar el flujo de código de autorización para llamar a una API que requiere tokens de acceso mediante el punto de conexión de Azure Active Directory v2.0
services: active-directory
author: hahamil
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 07/17/2020
ms.author: hahamil
ms.custom: aaddev, devx-track-javascript
ms.openlocfilehash: 4613e22193de8dc374d1a9e1a293c317fb9c1b9b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87311556"
---
# <a name="tutorial-sign-in-users-and-call-the-microsoft-graph-api-from-a-javascript-single-page-app-spa-using-auth-code-flow"></a>Tutorial: Inicio de sesión de usuarios y llamada a Microsoft Graph API desde una aplicación de página única (SPA) de JavaScript mediante un flujo de código de autorización

En este tutorial se muestra cómo crear una aplicación de página única (SPA) de JavaScript que usa la biblioteca de autenticación de Microsoft (MSAL) para JavaScript v2.0 para lo siguiente:

> [!div class="checklist"]
> * Realización del flujo de código de autorización de OAuth 2.0 con PKCE
> * Inicio de sesión en cuentas personales, profesionales y educativas de Microsoft
> * Adquisición de un token de acceso
> * Llamada a Microsoft Graph o a su propia API que requiera tokens de acceso obtenidos del punto de conexión de la plataforma de identidad de Microsoft

MSAL.js 2.0 mejora MSAL.js 1.0 al admitir el flujo de código de autorización en el explorador en lugar del flujo de concesión implícito. MSAL.js 2.0 **no** admite el flujo implícito.

[!INCLUDE [MSAL.js 2.0 and Azure AD B2C temporary incompatibility notice](../../../includes/msal-b2c-cors-compatibility-notice.md)]

## <a name="how-the-tutorial-app-works"></a>Cómo funciona la aplicación tutorial

:::image type="content" source="media/tutorial-v2-javascript-auth-code/diagram-01-auth-code-flow.png" alt-text="Diagrama que muestra el flujo de código de autorización para una aplicación de página única":::

La aplicación que se crea en este tutorial permite que una aplicación de página única de JavaScript consulte Microsoft Graph API mediante la adquisición de tokens de seguridad del punto de conexión de la plataforma de identidad de Microsoft. En este escenario, después de que un usuario inicia sesión, se solicita un token de acceso y se agrega a las solicitudes HTTP mediante el encabezado de autorización. La adquisición y la renovación de los tokens se realiza por medio de la biblioteca de autenticación de Microsoft para JavaScript (MSAL.js).

En este tutorial se usa la siguiente biblioteca:

[msal.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser) Biblioteca de autenticación de Microsoft para el paquete del explorador de JavaScript v2.0

## <a name="get-the-completed-code-sample"></a>Obtención del ejemplo de código completado

¿Prefiere descargar en su lugar este proyecto de ejemplo completado del tutorial? Para ejecutar el proyecto mediante un servidor web local, como Node.js, clone el repositorio [ms-identity-javascript-v2](https://github.com/Azure-Samples/ms-identity-javascript-v2):

`git clone https://github.com/Azure-Samples/ms-identity-javascript-v2`

Después, para configurar el ejemplo de código antes de ejecutarlo, vaya al [paso de configuración](#register-your-application).

Para continuar con el tutorial y crear la aplicación, vaya a la sección siguiente, [Prerrequisitos](#prerequisites).

## <a name="prerequisites"></a>Requisitos previos

* [Node.js](https://nodejs.org/en/download/) para ejecutar un servidor web local
* [Visual Studio Code](https://code.visualstudio.com/download) u otro editor de código

## <a name="create-your-project"></a>Creación del proyecto

Una vez que haya instalado [Node.js](https://nodejs.org/en/download/), cree una carpeta para hospedar la aplicación, por ejemplo *msal-spa-tutorial*.

A continuación, implemente un servidor web de [Express](https://expressjs.com/) pequeño para atender el archivo *index.html*.

1. En primer lugar, cambie al directorio del proyecto en el terminal y, a continuación, ejecute los siguientes comandos de `npm`:
    ```console
    npm init -y
    npm install @azure/msal-browser
    npm install express
    npm install morgan
    npm install yargs
    ```
2. Después, cree un archivo llamado *server.js* y agregue el código siguiente:

   ```JavaScript
   const express = require('express');
   const morgan = require('morgan');
   const path = require('path');
   const argv = require('yargs')
      .usage('Usage: $0 -p [PORT]')
      .alias('p', 'port')
      .describe('port', '(Optional) Port Number - default is 3000')
      .strict()
      .argv;

   const DEFAULT_PORT = 3000;

   //initialize express.
   const app = express();

   // Initialize variables.
   let port = DEFAULT_PORT; // -p {PORT} || 3000;
   if (argv.p) {
      port = argv.p;
   }

   // Configure morgan module to log all requests.
   app.use(morgan('dev'));

   // Set the front-end folder to serve public assets.
   app.use("/lib", express.static(path.join(__dirname, "../../lib/msal-browser/lib")));

   // Setup app folders
   app.use(express.static('app'));

   // Set up a route for index.html.
   app.get('*', function (req, res) {
      res.sendFile(path.join(__dirname + '/index.html'));
   });

   // Start the server.
   app.listen(port);
   console.log(`Listening on port ${port}...`);
    ```

Ahora tiene un servidor web pequeño para dar servicio a la aplicación de página única. Después de completar el resto del tutorial, la estructura de archivos y carpetas del proyecto será similar a la siguiente:

```
msal-spa-tutorial/
├── app
│   ├── authConfig.js
│   ├── authPopup.js
│   ├── authRedirect.js
│   ├── graphConfig.js
│   ├── graph.js
│   ├── index.html
│   └── ui.js
└── server.js
```

## <a name="create-the-spa-ui"></a>Creación de la interfaz de usuario de SPA

1. Cree una carpeta *app* en el directorio del proyecto y, en ella, cree un archivo *index.html* para la aplicación de página única de JavaScript. Este archivo implementa una interfaz de usuario creada con **Bootstrap 4 Framework** e importa archivos de script para la configuración, la autenticación y las llamadas API.

    En el archivo *index.html*, agregue el código siguiente:

    ```html
    <!DOCTYPE html>
    <html lang="en">
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0, shrink-to-fit=no">
        <title>Tutorial | MSAL.js JavaScript SPA</title>

        <!-- IE support: add promises polyfill before msal.js  -->
        <script type="text/javascript" src="//cdn.jsdelivr.net/npm/bluebird@3.7.2/js/browser/bluebird.min.js"></script>
        <script type="text/javascript" src="https://alcdn.msauth.net/browser/2.0.0-beta.4/js/msal-browser.js" integrity="sha384-7sxY2tN3GMVE5jXH2RL9AdbO6s46vUh9lUid4yNCHJMUzDoj+0N4ve6rLOmR88yN" crossorigin="anonymous"></script>

        <!-- adding Bootstrap 4 for UI components  -->
        <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
        <link rel="SHORTCUT ICON" href="https://c.s-microsoft.com/favicon.ico?v2" type="image/x-icon">
      </head>
      <body>
        <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
          <a class="navbar-brand" href="/">Microsoft identity platform</a>
          <div class="btn-group ml-auto dropleft">
              <button type="button" id="SignIn" class="btn btn-secondary" onclick="signIn()">
                Sign In
              </button>
          </div>
        </nav>
        <br>
        <h5 class="card-header text-center">JavaScript SPA calling Microsoft Graph API with MSAL.js</h5>
        <br>
        <div class="row" style="margin:auto" >
        <div id="card-div" class="col-md-3" style="display:none">
        <div class="card text-center">
          <div class="card-body">
            <h5 class="card-title" id="WelcomeMessage">Please sign-in to see your profile and read your mails</h5>
            <div id="profile-div"></div>
            <br>
            <br>
            <button class="btn btn-primary" id="seeProfile" onclick="seeProfile()">See Profile</button>
            <br>
            <br>
            <button class="btn btn-primary" id="readMail" onclick="readMail()">Read Mail</button>
          </div>
        </div>
        </div>
        <br>
        <br>
          <div class="col-md-4">
            <div class="list-group" id="list-tab" role="tablist">
            </div>
          </div>
          <div class="col-md-5">
            <div class="tab-content" id="nav-tabContent">
            </div>
          </div>
        </div>
        <br>
        <br>

        <!-- importing bootstrap.js and supporting js libraries -->
        <script src="https://code.jquery.com/jquery-3.4.1.slim.min.js" integrity="sha384-J6qa4849blE2+poT4WnyKhv5vZF5SrPo0iEjwBvKU7imGFAV0wwj1yYfoRSJoZ+n" crossorigin="anonymous"></script>
        <script src="https://cdn.jsdelivr.net/npm/popper.js@1.16.0/dist/umd/popper.min.js" integrity="sha384-Q6E9RHvbIyZFJoft+2mJbHaEWldlvI9IOYy5n3zV9zzTtmI3UksdQRVvoxMfooAo" crossorigin="anonymous"></script>
        <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/js/bootstrap.min.js" integrity="sha384-wfSDF2E50Y2D1uUdj0O3uMBJnjuUD4Ih7YwaYd1iqfktj0Uod8GCExl3Og8ifwB6" crossorigin="anonymous"></script>

        <!-- importing app scripts (load order is important) -->
        <script type="text/javascript" src="./authConfig.js"></script>
        <script type="text/javascript" src="./graphConfig.js"></script>
        <script type="text/javascript" src="./ui.js"></script>

        <!-- <script type="text/javascript" src="./authRedirect.js"></script>   -->
        <!-- uncomment the above line and comment the line below if you would like to use the redirect flow -->
        <script type="text/javascript" src="./authPopup.js"></script>
        <script type="text/javascript" src="./graph.js"></script>
      </body>
    </html>
    ```

2. A continuación, también en la carpeta *app*, cree un archivo denominado *ui.js* y agregue el código siguiente. Este archivo tendrá acceso a los elementos DOM y los actualizará.

    ```JavaScript
    // Select DOM elements to work with
    const welcomeDiv = document.getElementById("WelcomeMessage");
    const signInButton = document.getElementById("SignIn");
    const cardDiv = document.getElementById("card-div");
    const mailButton = document.getElementById("readMail");
    const profileButton = document.getElementById("seeProfile");
    const profileDiv = document.getElementById("profile-div");

    function showWelcomeMessage(account) {
        // Reconfiguring DOM elements
        cardDiv.style.display = 'initial';
        welcomeDiv.innerHTML = `Welcome ${account.username}`;
        signInButton.setAttribute("onclick", "signOut();");
        signInButton.setAttribute('class', "btn btn-success")
        signInButton.innerHTML = "Sign Out";
    }

    function updateUI(data, endpoint) {
        console.log('Graph API responded at: ' + new Date().toString());

        if (endpoint === graphConfig.graphMeEndpoint) {
            const title = document.createElement('p');
            title.innerHTML = "<strong>Title: </strong>" + data.jobTitle;
            const email = document.createElement('p');
            email.innerHTML = "<strong>Mail: </strong>" + data.mail;
            const phone = document.createElement('p');
            phone.innerHTML = "<strong>Phone: </strong>" + data.businessPhones[0];
            const address = document.createElement('p');
            address.innerHTML = "<strong>Location: </strong>" + data.officeLocation;
            profileDiv.appendChild(title);
            profileDiv.appendChild(email);
            profileDiv.appendChild(phone);
            profileDiv.appendChild(address);

        } else if (endpoint === graphConfig.graphMailEndpoint) {
            if (data.value.length < 1) {
                alert("Your mailbox is empty!")
            } else {
                const tabList = document.getElementById("list-tab");
                tabList.innerHTML = ''; // clear tabList at each readMail call
                const tabContent = document.getElementById("nav-tabContent");

                data.value.map((d, i) => {
                    // Keeping it simple
                    if (i < 10) {
                        const listItem = document.createElement("a");
                        listItem.setAttribute("class", "list-group-item list-group-item-action")
                        listItem.setAttribute("id", "list" + i + "list")
                        listItem.setAttribute("data-toggle", "list")
                        listItem.setAttribute("href", "#list" + i)
                        listItem.setAttribute("role", "tab")
                        listItem.setAttribute("aria-controls", i)
                        listItem.innerHTML = d.subject;
                        tabList.appendChild(listItem)

                        const contentItem = document.createElement("div");
                        contentItem.setAttribute("class", "tab-pane fade")
                        contentItem.setAttribute("id", "list" + i)
                        contentItem.setAttribute("role", "tabpanel")
                        contentItem.setAttribute("aria-labelledby", "list" + i + "list")
                        contentItem.innerHTML = "<strong> from: " + d.from.emailAddress.address + "</strong><br><br>" + d.bodyPreview + "...";
                        tabContent.appendChild(contentItem);
                    }
                });
            }
        }
    }
    ```

## <a name="register-your-application"></a>Registrar su aplicación

Siga los pasos descritos en [Aplicación de una sola página: Registro de aplicación](scenario-spa-app-registration.md) para crear un registro de aplicación para la aplicación de página única.

En el paso [URI de redireccionamiento: MSAL.js 2.0 con flujo de código de autorización](scenario-spa-app-registration.md#redirect-uri-msaljs-20-with-auth-code-flow), escriba `http://localhost:3000`, la ubicación predeterminada donde se ejecuta la aplicación de este tutorial.

Si desea utilizar un puerto diferente, escriba `http://localhost:<port>`, donde `<port>` es el número de puerto TCP preferido. Si especifica un número de puerto distinto de `3000`, actualice también *server.js* con el número de puerto que prefiera.

### <a name="configure-your-javascript-spa"></a>Configuración de JavaScript SPA

Cree un nuevo archivo llamado *authConfig.js*, en la carpeta *app* que contendrá los parámetros de configuración de la autenticación, y agregue el código siguiente:

```javascript
const msalConfig = {
  auth: {
    clientId: "Enter_the_Application_Id_Here",
    authority: "Enter_the_Cloud_Instance_Id_Here/Enter_the_Tenant_Info_Here",
    redirectUri: "Enter_the_Redirect_Uri_Here",
  },
  cache: {
    cacheLocation: "sessionStorage", // This configures where your cache will be stored
    storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
  }
};

// Add scopes here for ID token to be used at Microsoft identity platform endpoints.
const loginRequest = {
 scopes: ["openid", "profile", "User.Read"]
};

// Add scopes here for access token to be used at Microsoft Graph API endpoints.
const tokenRequest = {
 scopes: ["User.Read", "Mail.Read"]
};
```

Modifique los valores de la sección `msalConfig` como se describe aquí:

- `Enter_the_Application_Id_Here`: el **identificador de la aplicación (cliente)** de la aplicación que registró.
- `Enter_the_Cloud_Instance_Id_Here`: Instancia en la nube de Azure en la que se registra la aplicación.
  - En el caso de la nube principal (o *global*) de Azure, escriba `https://login.microsoftonline.com`.
  - En el caso de las nubes **nacionales** (por ejemplo, China), puede encontrar los valores adecuados en las [nubes nacionales](authentication-national-cloud.md).
- `Enter_the_Tenant_info_here` debe ser una de las siguientes:
  - Si la aplicación admite *cuentas de este directorio organizativo*, reemplace este valor por los valores de **Id. de inquilino** o **Nombre de inquilino**. Por ejemplo, `contoso.microsoft.com`.
  - Si la aplicación admite *cuentas de cualquier directorio organizativo*, reemplace este valor por `organizations`.
  - Si la aplicación admite *cuentas de cualquier directorio organizativo y cuentas Microsoft personales*, reemplace este valor por `common`.
  - Para restringir la compatibilidad a *solo cuentas de Microsoft personales*, reemplace este valor por `consumers`.
- `Enter_the_Redirect_Uri_Here` es `http://localhost:3000`.

El valor de `authority` en *authConfig.js* debe ser similar al siguiente si usa la nube global de Azure:

```javascript
authority: "https://login.microsoftonline.com/common",
```

Todavía en la carpeta *app*, cree un archivo denominado *graphConfig.js*. Agregue el código siguiente para proporcionar a la aplicación los parámetros de configuración para llamar a Microsoft Graph API:

```javascript
// Add the endpoints here for Microsoft Graph API services you'd like to use.
const graphConfig = {
    graphMeEndpoint: "Enter_the_Graph_Endpoint_Here/v1.0/me",
    graphMailEndpoint: "Enter_the_Graph_Endpoint_Here/v1.0/me/messages"
};
```

Modifique los valores de la sección `graphConfig` como se describe aquí:

- `Enter_the_Graph_Endpoint_Here` es la instancia de Microsoft Graph API con la que la aplicación debe comunicarse.
  - En el caso del punto de conexión de Microsoft Graph API **global**, reemplace ambas instancias de esta cadena por `https://graph.microsoft.com`.
  - Para obtener información sobre los puntos de conexión en las implementaciones de nubes **nacionales**, consulte [Implementaciones de nube nacionales](https://docs.microsoft.com/graph/deployments) en la documentación de Microsoft Graph.

Los valores `graphMeEndpoint` y `graphMailEndpoint` de *graphConfig.js* deben ser similares a los siguientes si usa el punto de conexión global:

```javascript
graphMeEndpoint: "https://graph.microsoft.com/v1.0/me",
graphMailEndpoint: "https://graph.microsoft.com/v1.0/me/messages"
```

## <a name="use-microsoft-authentication-library-msal-to-sign-in-user"></a>Uso de la biblioteca de autenticación de Microsoft (MSAL) para iniciar la sesión del usuario

### <a name="pop-up"></a>Elemento emergente

En la carpeta *app*, cree un archivo denominado *authPopup.js* y agregue el siguiente código de autenticación y de adquisición de tokens para la ventana emergente de inicio de sesión:

```JavaScript
// Create the main myMSALObj instance
// configuration parameters are located at authConfig.js
const myMSALObj = new msal.PublicClientApplication(msalConfig);

let username = "";

function loadPage() {
    /**
     * See here for more info on account retrieval:
     * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
     */
    const currentAccounts = myMSALObj.getAllAccounts();
    if (currentAccounts === null) {
        return;
    } else if (currentAccounts.length > 1) {
        // Add choose account code here
        console.warn("Multiple accounts detected.");
    } else if (currentAccounts.length === 1) {
        username = currentAccounts[0].username;
        showWelcomeMessage(currentAccounts[0]);
    }
}

function handleResponse(resp) {
    if (resp !== null) {
        username = resp.account.username;
        showWelcomeMessage(resp.account);
    } else {
        loadPage();
    }
}

function signIn() {
    myMSALObj.loginPopup(loginRequest).then(handleResponse).catch(error => {
        console.error(error);
    });
}

function signOut() {
    const logoutRequest = {
        account: myMSALObj.getAccountByUsername(username)
    };

    myMSALObj.logout(logoutRequest);
}

function getTokenPopup(request) {
    /**
     * See here for more info on account retrieval:
     * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
     */
    request.account = myMSALObj.getAccountByUsername(username);
    return myMSALObj.acquireTokenSilent(request).catch(error => {
        console.warn("silent token acquisition fails. acquiring token using redirect");
        if (error instanceof msal.InteractionRequiredAuthError) {
            // fallback to interaction when silent call fails
            return myMSALObj.acquireTokenPopup(request).then(tokenResponse => {
                console.log(tokenResponse);

                return tokenResponse;
            }).catch(error => {
                console.error(error);
            });
        } else {
            console.warn(error);
        }
    });
}

function seeProfile() {
    getTokenPopup(loginRequest).then(response => {
        callMSGraph(graphConfig.graphMeEndpoint, response.accessToken, updateUI);
        profileButton.classList.add('d-none');
        mailButton.classList.remove('d-none');
    }).catch(error => {
        console.error(error);
    });
}

function readMail() {
    getTokenPopup(tokenRequest).then(response => {
        callMSGraph(graphConfig.graphMailEndpoint, response.accessToken, updateUI);
    }).catch(error => {
        console.error(error);
    });
}

loadPage();
```

### <a name="redirect"></a>Redirect

Cree un archivo denominado *authRedirect.js* en la carpeta *app* y agregue el siguiente código de autenticación y de adquisición de tokens para la redirección de inicio de sesión:

```javascript
// Create the main myMSALObj instance
// configuration parameters are located at authConfig.js
const myMSALObj = new msal.PublicClientApplication(msalConfig);

let accessToken;
let username = "";

// Redirect: once login is successful and redirects with tokens, call Graph API
myMSALObj.handleRedirectPromise().then(handleResponse).catch(err => {
    console.error(err);
});

function handleResponse(resp) {
    if (resp !== null) {
        username = resp.account.username;
        showWelcomeMessage(resp.account);
    } else {
        /**
         * See here for more info on account retrieval:
         * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
         */
        const currentAccounts = myMSALObj.getAllAccounts();
        if (currentAccounts === null) {
            return;
        } else if (currentAccounts.length > 1) {
            // Add choose account code here
            console.warn("Multiple accounts detected.");
        } else if (currentAccounts.length === 1) {
            username = currentAccounts[0].username;
            showWelcomeMessage(currentAccounts[0]);
        }
    }
}

function signIn() {
    myMSALObj.loginRedirect(loginRequest);
}

function signOut() {
    const logoutRequest = {
        account: myMSALObj.getAccountByUsername(username)
    };

    myMSALObj.logout(logoutRequest);
}

function getTokenRedirect(request) {
    /**
     * See here for more info on account retrieval:
     * https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-common/docs/Accounts.md
     */
    request.account = myMSALObj.getAccountByUsername(username);
    return myMSALObj.acquireTokenSilent(request).catch(error => {
            console.warn("silent token acquisition fails. acquiring token using redirect");
            if (error instanceof msal.InteractionRequiredAuthError) {
                // fallback to interaction when silent call fails
                return myMSALObj.acquireTokenRedirect(request);
            } else {
                console.warn(error);
            }
        });
}

function seeProfile() {
    getTokenRedirect(loginRequest).then(response => {
        callMSGraph(graphConfig.graphMeEndpoint, response.accessToken, updateUI);
        profileButton.classList.add('d-none');
        mailButton.classList.remove('d-none');
    }).catch(error => {
        console.error(error);
    });
}

function readMail() {
    getTokenRedirect(tokenRequest).then(response => {
        callMSGraph(graphConfig.graphMailEndpoint, response.accessToken, updateUI);
    }).catch(error => {
        console.error(error);
    });
}
```

### <a name="how-the-code-works"></a>Funcionamiento del código

Cuando un usuario selecciona el botón **Iniciar sesión** por primera vez, el método `signIn` llama a `loginPopup` para iniciar la sesión del usuario. Este método `loginPopup` hace que se abra una ventana emergente con el *punto de conexión de la Plataforma de identidad de Microsoft* para pedir y validar las credenciales del usuario. Después de un inicio de sesión correcto, *msal.js* inicia el [flujo de código de autorización](v2-oauth2-auth-code-flow.md).

En este momento, se envía un código de autorización protegido por PKCE al punto de conexión del token protegido con CORS y se intercambia por tokens. Se reciben un token de identificador, un token de acceso y un token de actualización por la aplicación y *msal.js* los procesa, y la información contenida en los tokens se almacena en caché.

El token de identificador contiene información básica sobre el usuario, como su nombre para mostrar. Si piensa usar los datos proporcionados por este token de identificador, *debe* validarse en el servidor back-end, para asegurarse de que el token se emitió para un usuario válido en la aplicación. El token de actualización tiene una duración limitada y expira después de 24 horas. Este token se puede usar para adquirir nuevos tokens de acceso de forma silenciosa.

La instancia de aplicación de página única generada por este tutorial llama a `acquireTokenSilent` o a `acquireTokenPopup` para adquirir un *token de acceso* que se usa para consultar la información del perfil de usuario a Microsoft Graph API. Si necesita obtener un ejemplo que valide el token de identificador, consulte la aplicación de ejemplo [active-directory-javascript-singlepageapp-dotnet-webapi-v2](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2) en GitHub. En el ejemplo se usa una API web de ASP.NET para la validación de tokens.

#### <a name="get-a-user-token-interactively"></a>Obtención de un token de usuario interactivamente

Después del inicio de sesión inicial, la aplicación no solicitará a los usuarios que se vuelvan a autenticar cada vez que necesiten acceder a un recurso protegido (es decir, para solicitar un token). Para evitar estas solicitudes de reautenticación, llame a `acquireTokenSilent`. Sin embargo, puede haber situaciones en las que sea necesario hacer que los usuarios interactúen con el punto de conexión de la Plataforma de identidad de Microsoft. Por ejemplo:

- Los usuarios deben volver a escribir las credenciales porque la contraseña expiró.
- La aplicación solicita acceso a un recurso para el que se necesita el consentimiento del usuario.
- Se requiere la autenticación en dos fases.

La llamada a `acquireTokenPopup` abre una ventana emergente (o `acquireTokenRedirect` redirige a los usuarios al punto de conexión de la Plataforma de identidad de Microsoft). En esa ventana, los usuarios tienen que interactuar confirmando sus credenciales, dándole el consentimiento al recurso requerido o completando la autenticación en dos fases.

#### <a name="get-a-user-token-silently"></a>Obtención de un token de usuario en silencio

El método `acquireTokenSilent` controla la renovación y las adquisiciones de tokens sin la interacción del usuario. Después de ejecutar `loginPopup` (o `loginRedirect`) por primera vez, `acquireTokenSilent` es el método que se usa habitualmente para obtener los tokens que se utilizan para acceder a los recursos protegidos en las llamadas posteriores. (Las llamadas para solicitar o renovar tokens se realizan en modo silencioso). `acquireTokenSilent` puede dar error en algunos casos. Por ejemplo, la contraseña del usuario puede haber expirado. La aplicación puede abordar esta excepción de dos maneras:

1. Puede realizar una llamada de inmediato a `acquireTokenPopup` para desencadenar un mensaje de inicio de sesión de usuario. Este patrón se da comúnmente en aplicaciones en línea en las que no hay ningún contenido no autenticado en la aplicación disponible para el usuario. El ejemplo generado por esta instalación guiada usa este patrón.
1. Puede indicar visualmente al usuario que se requiere un inicio de sesión interactivo, de manera que el usuario pueda seleccionar el momento adecuado para iniciar sesión, o bien puede reintentar ejecutar el método `acquireTokenSilent` en un momento posterior. Esta técnica se suele usar cuando el usuario puede utilizar otra funcionalidad de la aplicación sin ser interrumpido. Por ejemplo, podría haber contenido no autenticado disponible en la aplicación. En esta situación, el usuario puede decidir cuándo desea iniciar sesión para acceder al recurso protegido o para actualizar la información obsoleta.

> [!NOTE]
> En este tutorial se usan los métodos `loginPopup` y `acquireTokenPopup` de forma predeterminada. Si usa Internet Explorer, se recomienda usar los métodos `loginRedirect` y `acquireTokenRedirect` debido a un [problema conocido](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues) con Internet Explorer y ventanas emergentes. Para obtener un ejemplo de cómo lograr el mismo resultado mediante métodos de redirección, consulte [*authRedirect.js*](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/blob/quickstart/JavaScriptSPA/authRedirect.js) en GitHub.

## <a name="call-the-microsoft-graph-api"></a>Llamar a Microsoft Graph API

Cree un archivo denominado *graph.js* en la carpeta *app* y agregue el código siguiente para realizar llamadas de REST a Microsoft Graph API:

```javascript
// Helper function to call Microsoft Graph API endpoint
// using authorization bearer token scheme
function callMSGraph(endpoint, token, callback) {
    const headers = new Headers();
    const bearer = `Bearer ${token}`;

    headers.append("Authorization", bearer);

    const options = {
        method: "GET",
        headers: headers
    };

    console.log('request made to Graph API at: ' + new Date().toString());

    fetch(endpoint, options)
        .then(response => response.json())
        .then(response => callback(response, endpoint))
        .catch(error => console.log(error));
}
```

En esta aplicación de ejemplo que se crea en este tutorial, el método `callMSGraph()` se usa para realizar una solicitud HTTP `GET` a un recurso protegido que requiere un token. A continuación, el método devuelve el contenido al autor de la llamada. Este método agrega el token adquirido al *encabezado de autorización HTTP*. En este ejemplo de aplicación creada en este tutorial, el recurso protegido es el punto de conexión *me* de Microsoft Graph API, que muestra información del perfil del usuario que ha iniciado sesión.

## <a name="test-your-application"></a>Prueba de la aplicación

Ha completado la creación de la aplicación y ya está listo para iniciar el servidor web de Node.js y probar la funcionalidad de la aplicación.

1. Inicie el servidor web de Node.js mediante la ejecución del siguiente comando desde la raíz de la carpeta del proyecto:

   ```console
   npm start
   ```
1. En el explorador, escriba `http://localhost:3000` o `http://localhost:<port>`, donde `<port>` es el puerto en el que escucha el servidor web. Verá el contenido del archivo *index.html* y el botón **Iniciar sesión**.

### <a name="sign-in-to-the-application"></a>Inicie sesión en la aplicación.

Cuando el explorador haya cargado el archivo *index.html*, seleccione **Iniciar sesión**. Se le pedirá que inicie sesión con el punto de conexión de la plataforma de identidad de Microsoft:

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-01-signin-dialog.png" alt-text="Explorador web que muestra el cuadro de diálogo de inicio de sesión":::

### <a name="provide-consent-for-application-access"></a>Consentimiento para el acceso a la aplicación

La primera vez que inicie sesión en la aplicación, se le pedirá que le conceda acceso a su perfil e inicie su sesión:

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-02-consent-dialog.png" alt-text="Cuadro de diálogo de contenido que se muestra en el explorador web":::

Si da su consentimiento a los permisos solicitados, las aplicaciones web muestran el nombre de usuario, lo que significa que el inicio de sesión se ha realizado correctamente:

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-03-signed-in.png" alt-text="Resultados de un inicio de sesión correcto en el explorador web":::

### <a name="call-the-graph-api"></a>Llamada a Graph API

Después de iniciar sesión, seleccione **Ver perfil** para ver la información de perfil de usuario devuelta en la respuesta de la llamada a Microsoft Graph API:

:::image type="content" source="media/tutorial-v2-javascript-auth-code/spa-04-see-profile.png" alt-text="Información de perfil de Microsoft Graph que se muestra en el explorador":::

### <a name="more-information-about-scopes-and-delegated-permissions"></a>Más información sobre los ámbitos y permisos delegados

Microsoft Graph API requiere el ámbito *user.read* para leer el perfil del usuario. De forma predeterminada, este ámbito se agrega automáticamente en todas las aplicaciones que se registran en Azure Portal. Otras API de Microsoft Graph, así como las API personalizadas para el servidor back-end, pueden requerir ámbitos adicionales. Por ejemplo, Microsoft Graph API requiere el ámbito *Mail.Read* para mostrar el correo electrónico del usuario.

A medida que se agregan ámbitos, puede que se pida a los usuarios que proporcionen consentimiento adicional para los ámbitos agregados.

Si una API de back-end no requiere un ámbito, que no se recomienda, puede usar `clientId` como ámbito en las llamadas de adquisición de tokens.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una aplicación de página única (SPA) de JavaScript que usa la biblioteca de autenticación de Microsoft (MSAL) para JavaScript v2.0 para lo siguiente:

> [!div class="checklist"]
> * Realización del flujo de código de autorización de OAuth 2.0 con PKCE
> * Inicio de sesión en cuentas personales, profesionales y educativas de Microsoft
> * Adquisición de un token de acceso
> * Llamada a Microsoft Graph o a su propia API que requiera tokens de acceso obtenidos del punto de conexión de la plataforma de identidad de Microsoft

Para más información sobre el flujo de código de autorización, incluidas las diferencias entre los flujos de código implícito y de autenticación, consulte [Plataforma de identidad y flujo de código de autorización de OAuth 2.0](v2-oauth2-auth-code-flow.md).

Si quiere profundizar más en el desarrollo de aplicaciones de una sola página de JavaScript en la plataforma de Microsoft Identity, la serie de varias partes [Escenario: Aplicación de página única](scenario-spa-overview.md) puede ayudarle a empezar.
