---
title: 'Tutorial: Protección de una API web de Node.js mediante Azure AD B2C y concesión de acceso a una aplicación de página única (SPA)'
titleSuffix: Azure AD B2C
description: En este tutorial aprenderá a usar Active Directory B2C para proteger una API web de Node.js y llamarla desde una aplicación de página única.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.author: mimart
ms.date: 04/04/2020
ms.custom: mvc, devx-track-javascript
ms.topic: tutorial
ms.service: active-directory
ms.subservice: B2C
ms.openlocfilehash: f01ef1a4cf5bc5b805da3dd4d825ef17f81ce53e
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87170190"
---
# <a name="tutorial-protect-and-grant-access-to-a-nodejs-web-api-from-a-single-page-application-with-azure-ad-b2c"></a>Tutorial: Protección y concesión de acceso a una API web de Node.js desde una aplicación de página única con Azure AD B2C

En este tutorial se muestra cómo llamar a una API web de Node.js protegida mediante Azure Active Directory B2C (Azure AD B2C) desde una aplicación de página única.

Este tutorial es la segunda parte de una serie de dos:

> [!div class="checklist"]
> * Crear un registro de aplicación de API web en el inquilino de Azure AD B2C
> * Configurar los ámbitos para la API web
> * Conceder permisos a la API web
> * Modificar un código de ejemplo de la API web para que funcione con el inquilino

En el [primer tutorial](tutorial-single-page-app.md) de esta serie, ha descargado el código de ejemplo y lo ha modificado para iniciar sesión con usuarios mediante un flujo de usuario en el inquilino de Azure AD B2C.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

* Complete los pasos y requisitos previos en [Tutorial: Habilitación de la autenticación en una aplicación de página única mediante Azure AD B2C](tutorial-single-page-app.md)
* [Visual Studio Code](https://code.visualstudio.com/) u otro editor de código
* [Node.js](https://nodejs.org/en/download/)

## <a name="add-a-web-api-application"></a>Incorporar una aplicación de API web

[!INCLUDE [active-directory-b2c-appreg-webapi](../../includes/active-directory-b2c-appreg-webapi.md)]

## <a name="configure-scopes"></a>Configuración de ámbitos

Los ámbitos proporcionan una manera de controlar el acceso a los recursos protegidos. La API web utiliza los ámbitos para implementar el control de acceso basado en el ámbito. Por ejemplo, algunos usuarios pueden tener tanto acceso de lectura como de escritura, mientras que otros usuarios pueden tener permisos de solo lectura. En este tutorial, definirá los permisos de lectura y escritura para la API web.

[!INCLUDE [active-directory-b2c-scopes](../../includes/active-directory-b2c-scopes.md)]

Anote el valor indicado bajo **Ámbitos** correspondiente al ámbito `demo.read`; lo usará en un paso posterior al configurar la aplicación de página única. El valor de ámbito completo es similar a `https://contosob2c.onmicrosoft.com/api/demo.read`.

## <a name="grant-permissions"></a>Concesión de permisos

Para llamar a una API web protegida desde otra aplicación, deberá conceder permisos de la API web a la aplicación.

En el tutorial de requisitos previos, ha creado una aplicación web denominada *webapp1*. En este tutorial, configurará esa aplicación para que llame a la API web que creó en un apartado anterior, *webapi1*.

[!INCLUDE [active-directory-b2c-permissions-api](../../includes/active-directory-b2c-permissions-api.md)]

Ahora, la aplicación web de página única tiene permisos concedidos a la API web protegida para los ámbitos especificados. Un usuario se autentica con Azure AD B2C para utilizar la aplicación de página única. La aplicación de página única usa el flujo de concesión de autorización para acceder a la API web protegida con un token de acceso devuelto por Azure AD B2C.

## <a name="configure-the-sample"></a>Configuración del ejemplo

Ahora que se ha registrado la API web y ha definido ámbitos, configure el código de la API web para que funcione con el inquilino de Azure AD B2C. En este tutorial, configurará una API web de Node.js de ejemplo que descargará desde GitHub.

[Descargue un \*archivo zip](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi/archive/master.zip) o clone el proyecto de API web de ejemplo desde GitHub. También puede ir directamente al proyecto [Azure-Samples/active-directory-b2c-javascript-nodejs-webapi](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) en GitHub.

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi.git
```

### <a name="configure-the-web-api"></a>Configuración de la API web

1. Abra el archivo *config.js* en el editor de código.
1. Modifique los valores de las variables para que reflejen los del registro de aplicación creado anteriormente. Actualice igualmente `policyName` con el flujo de usuario que ha creado como parte de los requisitos previos. Por ejemplo, *B2C_1_signupsignin1*.

    ```javascript
    const clientID = "<your-webapi-application-ID>"; // Application (client) ID
    const b2cDomainHost = "<your-tenant-name>.b2clogin.com";
    const tenantId = "<your-tenant-ID>.onmicrosoft.com"; // Alternatively, you can use your Directory (tenant) ID (a GUID)
    const policyName = "B2C_1_signupsignin1";
    ```

#### <a name="enable-cors"></a>Habilitación de CORS

Para permitir que la aplicación de página única llame a la API web de Node.js, debe habilitar [CORS](https://expressjs.com/en/resources/middleware/cors.html) en la API web. En una aplicación de producción, tenga cuidado con el dominio que realiza la solicitud, no obstante, para este tutorial, permita las solicitudes de cualquier dominio.

Para habilitar CORS, utilice el siguiente middleware. En el código de ejemplo de la API web de Node.js de este tutorial, ya se ha agregado al archivo *index.js*.

```javascript
app.use((req, res, next) => {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "Authorization, Origin, X-Requested-With, Content-Type, Accept");
    next();
});
```

### <a name="configure-the-single-page-application"></a>Configuración de la aplicación de página única

La aplicación de página única (SPA) del [tutorial anterior](tutorial-single-page-app.md) de la serie usa Azure AD B2C para el registro y el inicio de sesión de usuario y, de forma predeterminada, llama a la API web de Node.js protegida por el inquilino de demostración *fabrikamb2c*.

En esta sección actualizará la aplicación web de página única para que llame a la API web de Node.js protegida por *su* inquilino de Azure AD B2C (y que ejecuta en la máquina local).

Para cambiar la configuración en la aplicación de página única:

1. En el proyecto [active-directory-b2c-javascript-msal-singlepageapp][github-js-spa] que ha descargado o clonado en el tutorial anterior, abra el archivo *apiConfig.js* en la carpeta *JavaScriptSPA*.
1. Configure el ejemplo con el URI para el ámbito *demo.read* que creó anteriormente y la dirección URL de la API web.
    1. En la definición de `apiConfig`, reemplace el valor `b2cScopes` por el URI completo del ámbito *demo.read* (el valor **Ámbito** que ha anotado anteriormente).
    1. Cambie el dominio del valor `webApi` por el URI de redirección que agregó al registrar la API web en un paso anterior.

    Dado que la API es accesible en el punto de conexión de `/hello`, deje */hello* en el URI.

    La definición de `apiConfig` debe ser similar al siguiente bloque de código, pero con el nombre del inquilino de B2C en el lugar de `<your-tenant-name>`:

    ```javascript
    // The current application coordinates were pre-registered in a B2C tenant.
    const apiConfig = {
      b2cScopes: ["https://<your-tenant-name>.onmicrosoft.com/api/demo.read"],
      webApi: "http://localhost:5000/hello" // '/hello' should remain in the URI
    };
    ```

## <a name="run-the-spa-and-web-api"></a>Ejecución de la aplicación de página única y la API web

Ya está preparado para probar el acceso de ámbito de la aplicación de página única a la API. Ejecute la API web de Node.js y la aplicación de página única de JavaScript de ejemplo en la máquina local. A continuación, inicie sesión en la aplicación de página única y seleccione el botón **Call API** (Llamar a la API) para iniciar una solicitud a la API protegida.

Aunque ambas aplicaciones se ejecutan localmente cuando sigue este tutorial, las ha configurado a fin de utilizar Azure AD B2C para el registro o el inicio de sesión seguros, y para conceder acceso a la API web protegida.

### <a name="run-the-nodejs-web-api"></a>Ejecución de la API web de Node.js

1. Abra una ventana de consola y cambie al directorio que contiene la API web de Node.js de ejemplo. Por ejemplo:

    ```console
    cd active-directory-b2c-javascript-nodejs-webapi
    ```

1. Ejecute los comandos siguientes:

    ```console
    npm install && npm update
    node index.js
    ```

    La ventana de la consola muestra el número de puerto en el que se hospeda la aplicación.

    ```console
    Listening on port 5000...
    ```

### <a name="run-the-single-page-app"></a>Ejecución de una aplicación de página única

1. Abra otra ventana de consola y cambie al directorio que contiene la aplicación de página única de JavaScript de ejemplo. Por ejemplo:

    ```console
    cd active-directory-b2c-javascript-msal-singlepageapp
    ```

1. Ejecute los comandos siguientes:

    ```console
    npm install && npm update
    npm start
    ```

    La ventana de la consola muestra el número de puerto donde se hospeda la aplicación.

    ```console
    Listening on port 6420...
    ```

1. Vaya a `http://localhost:6420` en el explorador para ver la aplicación.

    ![Aplicación de página única de ejemplo en el explorador](./media/tutorial-single-page-app-webapi/tutorial-01-sample-app-browser.png)

1. Inicie sesión con la dirección de correo electrónico y la contraseña que usó en el [tutorial anterior](tutorial-single-page-app.md). Tras iniciar sesión correctamente, debería ver el mensaje `User 'Your Username' logged-in`.
1. Seleccione el botón **Llamada a API**. La aplicación de página única obtiene una concesión de autorización de Azure AD B2C y accede a la API web protegida para mostrar el nombre del usuario que ha iniciado sesión:

    ![Aplicación de página única en el explorador que muestra el resultado JSON del nombre de usuario devuelto por la API](./media/tutorial-single-page-app-webapi/tutorial-02-call-api.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Crear un registro de aplicación de la API web en el inquilino de Azure AD B2C
> * Configurar ámbitos para la API web
> * Conceder permisos a la API web
> * Modificar un código de ejemplo de la API web para que funcione con el inquilino

Ahora que ha visto una aplicación de página única solicitar un recurso de una API web protegida, entremos en detalles sobre cómo interactúan estos tipos de aplicaciones entre sí y con Azure AD B2C.

> [!div class="nextstepaction"]
> [Tipos de aplicaciones que se pueden usar en Active Directory B2C >](application-types.md)

<!-- Links - EXTERNAL -->
[github-js-spa]: https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp
