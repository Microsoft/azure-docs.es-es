---
title: 'Tutorial de aplicaciones de página única de JavaScript: Plataforma de identidad de Microsoft | Azure'
description: Cómo pueden llamar las aplicaciones SPA de JavaScript a una API que requiera tokens de acceso mediante el punto de conexión de Azure Active Directory v2.0
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 03/20/2019
ms.author: nacanuma
ms.custom: aaddev, identityplatformtop40, devx-track-javascript
ms.openlocfilehash: 745132284ee48270b46b6bd2f785d8bec19404fb
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87129685"
---
# <a name="sign-in-users-and-call-the-microsoft-graph-api-from-a-javascript-single-page-application-spa"></a>Inicio de sesión de usuarios y llamada a Microsoft Graph API desde una aplicación de página única (SPA) de JavaScript

En esta guía se muestra cómo una aplicación de una sola página (SPA) de JavaScript puede:
- Iniciar sesión en cuentas personales, profesionales y educativas
- Adquisición de un token de acceso
- Llamar a Microsoft Graph API o a otras API que requieran tokens de acceso del *punto de conexión de la Plataforma de identidad de Microsoft*.

>[!NOTE]
> Si es la primera vez que usa la Plataforma de identidad de Microsoft, se recomienda que comience con [Inicio de sesión de los usuarios y obtención de un token de acceso en un inicio rápido de SPA de JavaScript](quickstart-v2-javascript.md).

## <a name="how-the-sample-app-generated-by-this-guide-works"></a>Funcionamiento de la aplicación de ejemplo generada por esta guía

![Muestra cómo funciona la aplicación de ejemplo generada por este tutorial](media/active-directory-develop-guidedsetup-javascriptspa-introduction/javascriptspa-intro.svg)

### <a name="more-information"></a>Más información

La aplicación de ejemplo que se crea con esta guía permite que una aplicación de página única de JavaScript haga consultas a Microsoft Graph API o a una API web que acepte tokens de un punto de conexión de la Plataforma de identidad de Microsoft. En este escenario, después de que un usuario inicia sesión, se solicita un token de acceso y se agrega a las solicitudes HTTP mediante el encabezado de autorización. Este token se usará para adquirir el perfil y los mensajes de correo electrónico del usuario mediante **MS Graph API**. La adquisición y la renovación de los tokens se realiza por medio de la **Biblioteca de autenticación de Microsoft (MSAL) para JavaScript**.

### <a name="libraries"></a>Bibliotecas

Esta guía utiliza la siguiente biblioteca:

|Biblioteca|Descripción|
|---|---|
|[msal.js](https://github.com/AzureAD/microsoft-authentication-library-for-js)|Biblioteca de autenticación de Microsoft para JavaScript|

## <a name="set-up-your-web-server-or-project"></a>Configuración de un proyecto o servidor web

> ¿Prefiere descargar este proyecto de ejemplo en su lugar? [Descargue los archivos del proyecto](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/archive/quickstart.zip).
>
> Para configurar el ejemplo de código antes de ejecutarlo, vaya al [paso de configuración](#register-your-application).

## <a name="prerequisites"></a>Requisitos previos

* Para ejecutar este tutorial, necesita un servidor web local, como [Node.js](https://nodejs.org/en/download/), [.NET Core](https://www.microsoft.com/net/core) o la integración de IIS Express con [Visual Studio 2017](https://www.visualstudio.com/downloads/).

* Las instrucciones de esta guía se basan en un servidor web integrado en Node.js. Se recomienda usar [Visual Studio Code](https://code.visualstudio.com/download) como entorno de desarrollo integrado (IDE).

* Un explorador web moderno. En este ejemplo de JavaScript se usan [convenciones ES6](http://www.ecma-international.org/ecma-262/6.0/) y, por lo tanto, **no** es compatible con **Internet Explorer**.

## <a name="create-your-project"></a>Creación del proyecto

Asegúrese de que tiene instalado [Node.js](https://nodejs.org/en/download/) y, a continuación, cree una carpeta para hospedar la aplicación. Allí, se implementará un servidor web [Express](https://expressjs.com/) sencillo para servir el archivo `index.html`.

1. Mediante un terminal (como el terminal integrado de Visual Studio Code), busque la carpeta del proyecto y, a continuación, escriba:

   ```console
   npm init
   ```

2. Después, instale las dependencias necesarias:

   ```console
   npm install express --save
   npm install morgan --save
   ```

1. Ahora, cree un archivo .js llamado `index.js` y agregue el código siguiente:

   ```JavaScript
   const express = require('express');
   const morgan = require('morgan');
   const path = require('path');

   //initialize express.
   const app = express();

   // Initialize variables.
   const port = 3000; // process.env.PORT || 3000;

   // Configure morgan module to log all requests.
   app.use(morgan('dev'));

   // Set the front-end folder to serve public assets.
   app.use(express.static('JavaScriptSPA'))

   // Set up a route for index.html.
   app.get('*', function (req, res) {
       res.sendFile(path.join(__dirname + '/index.html'));
   });

   // Start the server.
   app.listen(port);
   console.log('Listening on port ' + port + '...');
   ```

Ahora tiene un servidor simple para dar servicio a la SPA. La estructura de carpetas deseada al final de este tutorial es la siguiente:

![representación de texto de la estructura de carpetas de la SPA deseada](./media/tutorial-v2-javascript-spa/single-page-application-folder-structure.png)

## <a name="create-the-spa-ui"></a>Creación de la interfaz de usuario de SPA

1. Cree un archivo `index.html` para JavaScript SPA. Este archivo implementa una interfaz de usuario creada con **Bootstrap 4 Framework** e importa archivos de script para la configuración, la autenticación y la llamada a la API.

   En el archivo `index.html`, agregue el código siguiente:

   ```html
   <!DOCTYPE html>
   <html lang="en">
     <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0, shrink-to-fit=no">
       <title>Quickstart | MSAL.JS Vanilla JavaScript SPA</title>

       <!-- msal.js with a fallback to backup CDN -->
       <script type="text/javascript" src="https://alcdn.msauth.net/lib/1.2.1/js/msal.js" integrity="sha384-9TV1245fz+BaI+VvCjMYL0YDMElLBwNS84v3mY57pXNOt6xcUYch2QLImaTahcOP" crossorigin="anonymous"></script>
       <script type="text/javascript">
         if(typeof Msal === 'undefined')document.write(unescape("%3Cscript src='https://alcdn.msftauth.net/lib/1.2.1/js/msal.js' type='text/javascript' integrity='sha384-m/3NDUcz4krpIIiHgpeO0O8uxSghb+lfBTngquAo2Zuy2fEF+YgFeP08PWFo5FiJ' crossorigin='anonymous'%3E%3C/script%3E"));
       </script>

       <!-- adding Bootstrap 4 for UI components  -->
       <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css" integrity="sha384-Vkoo8x4CGsO3+Hhxv8T/Q5PaXtkKtu6ug5TOeNV6gBiFeWPGFN9MuhOf23Q9Ifjh" crossorigin="anonymous">
     </head>
     <body>
       <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
         <a class="navbar-brand" href="/">MS Identity Platform</a>
         <div class="btn-group ml-auto dropleft">
           <button type="button" id="signIn" class="btn btn-secondary" onclick="signIn()">Sign In</button>
           <button type="button" id="signOut" class="btn btn-success d-none" onclick="signOut()">Sign Out</button>
       </div>
       </nav>
       <br>
       <h5 class="card-header text-center">Vanilla JavaScript SPA calling MS Graph API with MSAL.JS</h5>
       <br>
       <div class="row" style="margin:auto" >
       <div id="card-div" class="col-md-3 d-none">
       <div class="card text-center">
         <div class="card-body">
           <h5 class="card-title" id="welcomeMessage">Please sign-in to see your profile and read your mails</h5>
           <div id="profile-div"></div>
           <br>
           <br>
           <button class="btn btn-primary" id="seeProfile" onclick="seeProfile()">See Profile</button>
           <br>
           <br>
           <button class="btn btn-primary d-none" id="readMail" onclick="readMail()">Read Mails</button>
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

       <!-- replace next line with authRedirect.js if you would like to use the redirect flow -->
       <!-- <script type="text/javascript" src="./authRedirect.js"></script>   -->
       <script type="text/javascript" src="./authPopup.js"></script>
       <script type="text/javascript" src="./graph.js"></script>
     </body>
   </html>
   ```

   > [!TIP]
   > Puede reemplazar la versión de MSAL.js del script anterior con la versión más reciente en la sección de [versiones de MSAL.js](https://github.com/AzureAD/microsoft-authentication-library-for-js/releases).

2. Ahora, cree un archivo .js llamado `ui.js`, que tendrá acceso a los elementos del DOM y los actualizará, y agregue el código siguiente:

   ```JavaScript
   // Select DOM elements to work with
   const welcomeDiv = document.getElementById("welcomeMessage");
   const signInButton = document.getElementById("signIn");
   const signOutButton = document.getElementById('signOut');
   const cardDiv = document.getElementById("card-div");
   const mailButton = document.getElementById("readMail");
   const profileButton = document.getElementById("seeProfile");
   const profileDiv = document.getElementById("profile-div");

   function showWelcomeMessage(account) {
     // Reconfiguring DOM elements
     cardDiv.classList.remove('d-none');
     welcomeDiv.innerHTML = `Welcome ${account.name}`;
     signInButton.classList.add('d-none');
     signOutButton.classList.remove('d-none');
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

Antes de continuar con la autenticación, registre la aplicación en **Azure Active Directory**.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Si la cuenta proporciona acceso a más de un inquilino, selecciónela en la esquina superior derecha y establezca la sesión del portal en el inquilino de Azure AD que desee utilizar.
1. Vaya a la página [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) de la plataforma de identidad de Microsoft para desarrolladores.
1. Cuando se abra la página **Registrar una aplicación**, escriba el nombre de su aplicación.
1. En **Supported account types** (Tipos de cuenta compatibles), seleccione **Accounts in any organizational directory and personal Microsoft accounts** (Cuentas en cualquier directorio de organización y cuentas personales de Microsoft).
1. En la sección **URI de redireccionamiento**, seleccione la plataforma **Web** en la lisa desplegable y, luego, establezca el valor en la dirección URL de la aplicación que se basa en el servidor web.
1. Seleccione **Registrar**.
1. En la página de **información general** de la aplicación, anote el valor del **Identificador de aplicación (cliente)** para su uso posterior.
1. Para esta guía, se requiere que habilite el [flujo de concesión implícita](v2-oauth2-implicit-grant-flow.md). En el panel izquierdo de la aplicación registrada, seleccione **Autenticación**.
1. En **Configuración avanzada**, en **Concesión implícita**, active las casillas **Tokens de id.** y **Tokens de acceso**. Los tokens de identificador y los tokens de acceso son obligatorios, ya que esta aplicación tiene que iniciar la sesión de los usuarios y llamar a una API.
1. Seleccione **Guardar**.

> ### <a name="set-a-redirect-url-for-nodejs"></a>Configuración de una dirección URL de redireccionamiento para Node.js
>
> Para Node.js, puede establecer el puerto del servidor web en el archivo *index.js*. En este tutorial se usa el puerto 3000, pero puede usar cualquier otro disponible.
>
> Para configurar una dirección URL de redireccionamiento en la información de registro de aplicación, vuelva al panel **Registro de aplicación** y realice una de las acciones siguientes:
>
> - Establezca *`http://localhost:3000/`* como la **dirección URL de redireccionamiento**.
> - Si usa un puerto TCP personalizado, use *`http://localhost:<port>/`* (donde *\<port>* es el número de puerto TCP personalizado).
>   1. Copie el valor **URL**.
>   1. Vuelva al panel **Registro de aplicación** y pegue el valor copiado como la **Dirección URL de redireccionamiento**.
>

### <a name="configure-your-javascript-spa"></a>Configuración de JavaScript SPA

Cree un nuevo archivo .js llamado `authConfig.js`, que contendrá los parámetros de configuración para la autenticación y agregue el código siguiente:

```javascript
  const msalConfig = {
    auth: {
      clientId: "Enter_the_Application_Id_Here",
      authority: "Enter_the_Cloud_Instance_Id_HereEnter_the_Tenant_Info_Here",
      redirectUri: "Enter_the_Redirect_Uri_Here",
    },
    cache: {
      cacheLocation: "sessionStorage", // This configures where your cache will be stored
      storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
    }
  };

  // Add here scopes for id token to be used at MS Identity Platform endpoints.
  const loginRequest = {
    scopes: ["openid", "profile", "User.Read"]
  };

  // Add here scopes for access token to be used at MS Graph API endpoints.
  const tokenRequest = {
    scopes: ["Mail.Read"]
  };
```

 Donde:
 - *\<Enter_the_Application_Id_Here>* es el **identificador de aplicación (cliente)** de la aplicación que registró.
 - *\<Enter_the_Cloud_Instance_Id_Here>* es la instancia de la nube de Azure. En el caso de la nube de Azure principal o global, simplemente escriba *https://login.microsoftonline.com* . Para nubes **nacionales** (por ejemplo, China), consulte [Nubes nacionales](https://docs.microsoft.com/azure/active-directory/develop/authentication-national-cloud).
 - *\<Enter_the_Tenant_info_here>* se establece en una de las opciones siguientes:
   - Si la aplicación admite *solo las cuentas de este directorio organizativo*, reemplace este valor por el **identificador de inquilino** o el **nombre de inquilino** (por ejemplo, *contoso.microsoft.com*).
   - Si la aplicación admite *cuentas en cualquier directorio organizativo*, reemplace este valor por **organizaciones**.
   - Si la aplicación admite *cuentas en cualquier directorio organizativo y cuentas Microsoft personales*, reemplace este valor por **común**. Para restringir la compatibilidad a *Personal Microsoft accounts only* (Solo cuentas Microsoft personales), reemplace este valor por **consumidores**.


## <a name="use-the-microsoft-authentication-library-msal-to-sign-in-the-user"></a>Uso de la biblioteca de autenticación de Microsoft (MSAL) para iniciar la sesión del usuario

Cree un nuevo archivo .js llamado `authPopup.js`, que contendrá la lógica de la autenticación y de la adquisición del token y agregue el código siguiente:

   ```JavaScript
   const myMSALObj = new Msal.UserAgentApplication(msalConfig);

   function signIn() {
     myMSALObj.loginPopup(loginRequest)
       .then(loginResponse => {
         console.log('id_token acquired at: ' + new Date().toString());
         console.log(loginResponse);

         if (myMSALObj.getAccount()) {
           showWelcomeMessage(myMSALObj.getAccount());
         }
       }).catch(error => {
         console.log(error);
       });
   }

   function signOut() {
     myMSALObj.logout();
   }

   function callMSGraph(theUrl, accessToken, callback) {
       var xmlHttp = new XMLHttpRequest();
       xmlHttp.onreadystatechange = function () {
           if (this.readyState == 4 && this.status == 200) {
              callback(JSON.parse(this.responseText));
           }
       }
       xmlHttp.open("GET", theUrl, true); // true for asynchronous
       xmlHttp.setRequestHeader('Authorization', 'Bearer ' + accessToken);
       xmlHttp.send();
   }

   function getTokenPopup(request) {
     return myMSALObj.acquireTokenSilent(request)
       .catch(error => {
         console.log(error);
         console.log("silent token acquisition fails. acquiring token using popup");

         // fallback to interaction when silent call fails
           return myMSALObj.acquireTokenPopup(request)
             .then(tokenResponse => {
               return tokenResponse;
             }).catch(error => {
               console.log(error);
             });
       });
   }

   function seeProfile() {
     if (myMSALObj.getAccount()) {
       getTokenPopup(loginRequest)
         .then(response => {
           callMSGraph(graphConfig.graphMeEndpoint, response.accessToken, updateUI);
           profileButton.classList.add('d-none');
           mailButton.classList.remove('d-none');
         }).catch(error => {
           console.log(error);
         });
     }
   }

   function readMail() {
     if (myMSALObj.getAccount()) {
       getTokenPopup(tokenRequest)
         .then(response => {
           callMSGraph(graphConfig.graphMailEndpoint, response.accessToken, updateUI);
         }).catch(error => {
           console.log(error);
         });
     }
   }
   ```

### <a name="more-information"></a>Más información

Cuando un usuario selecciona el botón **Iniciar sesión** por primera vez, el método `signIn` llama a `loginPopup` para iniciar la sesión del usuario. Este método hace que se abra una ventana emergente con el *punto de conexión de la Plataforma de identidad de Microsoft* para pedir y validar las credenciales del usuario. Después de iniciar sesión correctamente, el usuario se redirige de nuevo a la página *index.html* original. `msal.js` recibe y procesa un token, y la información contenida en él se almacena en caché. Este token se conoce como el *token de identificador* y contiene información básica sobre el usuario, como su nombre. Si piensa utilizar los datos proporcionados por este token para algún propósito, debe asegurarse de que el servidor backend lo valide para garantizar que el token se emitió a un usuario válido para la aplicación.

La instancia de SPA generada por esta guía llama a `acquireTokenSilent` o a `acquireTokenPopup` para adquirir un *token de acceso* que se usa para consultar la información del perfil de usuario a Microsoft Graph API. Si necesita un ejemplo que valide el token de identificador, eche un vistazo a [esta](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2 "Ejemplo active-directory-javascript-singlepageapp-dotnet-webapi-v2 de GitHub") aplicación de ejemplo en GitHub. En el ejemplo se usa una API web de ASP.NET para la validación de tokens.

#### <a name="get-a-user-token-interactively"></a>Obtención de un token de usuario interactivamente

Después del inicio de sesión inicial, no desea pedir a los usuarios que se vuelvan a autenticar cada vez que tengan que solicitar un token para acceder a un recurso. De modo que se debe usar *acquireTokenSilent* la mayor parte del tiempo para adquirir tokens. Sin embargo, hay situaciones en las que es necesario hacer que los usuarios interactúen con el punto de conexión de la Plataforma de identidad de Microsoft. Algunos ejemplos son:

- Los usuarios deben volver a escribir las credenciales porque la contraseña expiró.
- La aplicación solicita acceso a un recurso para el cual el usuario necesita consentimiento.
- Se requiere la autenticación en dos fases.

A llamar a *acquireTokenPopup*, se abre una ventana emergente (o *acquireTokenRedirect* redirige a los usuarios al punto de conexión de la Plataforma de identidad de Microsoft). En esa ventana, los usuarios tienen que interactuar confirmando sus credenciales, dándole el consentimiento al recurso requerido o completando la autenticación en dos fases.

#### <a name="get-a-user-token-silently"></a>Obtención de un token de usuario en silencio

El método `acquireTokenSilent` controla la renovación y las adquisiciones de tokens sin la interacción del usuario. Después de ejecutar `loginPopup` (o `loginRedirect`) por primera vez, `acquireTokenSilent` es el método que se usa habitualmente para obtener los tokens que se utilizan para acceder a los recursos protegidos en las llamadas posteriores. (Las llamadas para solicitar o renovar tokens se realizan en modo silencioso). `acquireTokenSilent` puede dar error en algunos casos. Por ejemplo, la contraseña del usuario puede haber expirado. La aplicación puede abordar esta excepción de dos maneras:

1. Realice una llamada a `acquireTokenPopup` inmediatamente, lo que desencadena un mensaje de inicio de sesión de usuario. Este patrón se da comúnmente en aplicaciones en línea en las que no hay ningún contenido no autenticado en la aplicación disponible para el usuario. El ejemplo generado por esta instalación guiada usa este patrón.

1. Las aplicaciones también pueden hacer una indicación visual al usuario de que se requiere un inicio de sesión interactivo, de manera que el usuario pueda seleccionar el momento adecuado para iniciar sesión, o la aplicación puede reintentar `acquireTokenSilent` en un momento posterior. Esto se suele usar cuando el usuario puede utilizar otra funcionalidad de la aplicación sin ser interrumpido. Por ejemplo, podría haber contenido no autenticado disponible en la aplicación. En esta situación, el usuario puede decidir cuándo desea iniciar sesión para acceder al recurso protegido o para actualizar la información obsoleta.

> [!NOTE]
> Esta guía de inicio rápido usa los métodos `loginPopup` y `acquireTokenPopup` de forma predeterminada. Si usa Internet Explorer como explorador, se recomienda usar los métodos `loginRedirect` y `acquireTokenRedirect`, debido a un [problema conocido](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues-on-IE-and-Edge-Browser#issues) relacionado con la forma en la que Internet Explorer controla las ventanas emergentes. Si desea ver cómo lograr el mismo resultado mediante `Redirect methods`, consulte [aquí](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2/blob/quickstart/JavaScriptSPA/authRedirect.js).

## <a name="call-the-microsoft-graph-api-by-using-the-token-you-just-acquired"></a>Llamada a Microsoft Graph API con el token que acaba de obtener

1. En primer lugar, cree un archivo .js llamado `graphConfig.js`, que almacenará los puntos de conexión REST. Agregue el siguiente código:

   ```JavaScript
      const graphConfig = {
        graphMeEndpoint: "Enter_the_Graph_Endpoint_Herev1.0/me",
        graphMailEndpoint: "Enter_the_Graph_Endpoint_Herev1.0/me/messages"
      };
   ```

   Donde:
   - *\<Enter_the_Graph_Endpoint_Here>* es la instancia de Microsoft Graph API. En el caso del punto de conexión global de MS Graph API, basta con reemplazar esta cadena por `https://graph.microsoft.com`. En el caso de las implementaciones de nube nacional, consulte la [Documentación de Graph API](https://docs.microsoft.com/graph/deployments).

1. A continuación, cree un archivo .js llamado `graph.js`, que realizará una llamada REST a Microsoft Graph API y agregue el código siguiente:

   ```javascript
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
       .catch(error => console.log(error))
   }
   ```

### <a name="more-information-about-making-a-rest-call-against-a-protected-api"></a>Más información acerca de cómo realizar una llamada de REST a una API protegida

En esta aplicación de ejemplo que se crea en esta guía, el método `callMSGraph()` se usa para realizar una solicitud HTTP `GET` a un recurso protegido que requiere un token. A continuación, el método devuelve el contenido al autor de la llamada. Este método agrega el token adquirido al *encabezado de autorización HTTP*. En este ejemplo de aplicación creada en esta guía, el recurso es el punto de conexión *me* de Microsoft Graph API, que muestra información del perfil del usuario.

## <a name="test-your-code"></a>Prueba del código

1. Configure el servidor para que escuche un puerto TCP que esté basado en la ubicación del archivo *index.html*. Para Node.js, inicie el servidor web para que escuche el puerto; para ello, ejecute los siguientes comandos desde la carpeta de la aplicación en un símbolo del sistema de la línea de comandos:

   ```bash
   npm install
   npm start
   ```
1. En el explorador, escriba **http://localhost:3000** o **http://localhost:{port}** , donde *port* es el puerto en el que escucha el servidor web. Verá el contenido del archivo *index.html* y el botón **Iniciar sesión**.

## <a name="test-your-application"></a>Prueba de la aplicación

Cuando el explorador haya cargado el archivo *index.html*, seleccione **Iniciar sesión**. Se le pedirá que inicie sesión con el punto de conexión de la plataforma de identidad de Microsoft:

![La ventana de inicio de sesión en la cuenta de SPA de JavaScript](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptspascreenshot1.png)

### <a name="provide-consent-for-application-access"></a>Consentimiento para el acceso a la aplicación

La primera vez que inicie sesión en la aplicación, se le pedirá que le conceda acceso a su perfil e inicie su sesión:

![La ventana "Permisos solicitados"](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptspaconsent.png)

### <a name="view-application-results"></a>Visualización de los resultados de la aplicación

Una vez iniciada la sesión, la información del perfil de usuario se devuelve en la respuesta de Microsoft Graph API que se muestra en la página:

![Resultados de la llamada a Microsoft Graph API](media/active-directory-develop-guidedsetup-javascriptspa-test/javascriptsparesults.png)

### <a name="more-information-about-scopes-and-delegated-permissions"></a>Más información sobre los ámbitos y permisos delegados

Microsoft Graph API requiere el ámbito *user.read* para leer el perfil del usuario. De forma predeterminada, este ámbito se agrega automáticamente en todas las aplicaciones que se registran en el portal de registro. Otras API de Microsoft Graph, así como las API personalizadas para el servidor back-end, pueden requerir ámbitos adicionales. Por ejemplo, Microsoft Graph API requiere el ámbito *Mail.Read* para mostrar los correos electrónicos del usuario.

> [!NOTE]
> Es posible que se pida al usuario algún consentimiento adicional a medida que aumente el número de ámbitos.

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]
