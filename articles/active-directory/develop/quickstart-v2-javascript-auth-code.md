---
title: Inicio de sesión de los usuarios en aplicaciones de página única (SPA) de JavaScript con código de autorización | Azure
titleSuffix: Microsoft identity platform
description: Aprenda cómo una aplicación de JavaScript puede llamar a una API que requiera tokens de acceso mediante el uso de la Plataforma de identidad de Microsoft.
services: active-directory
author: hahamil
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 07/17/2020
ms.author: hahamil
ms.custom: aaddev, scenarios:getting-started, languages:JavaScript, devx-track-javascript
ms.openlocfilehash: 461f05b90b79852194d657a5dcbc3ba7583cff8d
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88115193"
---
# <a name="quickstart-sign-in-users-and-get-an-access-token-in-a-javascript-spa-using-the-auth-code-flow"></a>Inicio rápido: Inicio de sesión de los usuarios y obtención de un token de acceso en una SPA de JavaScript mediante el flujo de código de autorización

En este inicio rápido aprenderá a ejecutar un código de ejemplo que muestra cómo una aplicación de página única (SPA) de JavaScript puede iniciar la sesión de los usuarios de cuentas personales, profesionales y educativas mediante el flujo de código de autorización. En el ejemplo de código también se muestra cómo obtener un token de acceso para llamar a una API web, en este caso, Microsoft Graph API. Para ilustrar este tema, consulte el apartado en el que se explica el [funcionamiento del ejemplo](#how-the-sample-works).

En este inicio rápido se usa MSAL.js 2.0 con el flujo de código de autorización. Puede encontrar un inicio rápido similar que usa MSAL.js 1.0 con el flujo implícito en [Inicio rápido: Inicio de sesión de usuarios en aplicaciones de página única de JavaScript](./quickstart-v2-javascript.md).

[!INCLUDE [MSAL.js 2.0 and Azure AD B2C temporary incompatibility notice](../../../includes/msal-b2c-cors-compatibility-notice.md)]

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una de forma gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Node.js](https://nodejs.org/en/download/)
* [Visual Studio Code](https://code.visualstudio.com/download) u otro editor de código

> [!div renderon="docs"]
> ## <a name="register-and-download-your-quickstart-application"></a>Registro y descarga de la aplicación de inicio rápido
> Para iniciar la aplicación de inicio rápido, use cualquiera de las siguientes opciones.
>
> ### <a name="option-1-express-register-and-auto-configure-your-app-and-then-download-your-code-sample"></a>Opción 1 (Rápido): registrar y configurar de modo automático la aplicación y, a continuación, descargar el código de ejemplo
>
> 1. Inicie sesión en [Azure Portal](https://portal.azure.com).
> 1. Si la cuenta le proporciona acceso a más de un inquilino, seleccione la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Active Directory (Azure AD) que desee utilizar.
> 1. Seleccione [App registrations](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType/JavascriptSpaQuickstartPage/sourceType/docs) (Registros de aplicaciones).
> 1. Escriba un nombre para la aplicación.
> 1. En **Supported account types** (Tipos de cuenta compatibles), seleccione **Accounts in any organizational directory and personal Microsoft accounts** (Cuentas en cualquier directorio de organización y cuentas personales de Microsoft).
> 1. Seleccione **Registrar**.
> 1. Vaya al panel de inicio rápido y siga las instrucciones para descargar y configurar automáticamente su nueva aplicación.
>
> ### <a name="option-2-manual-register-and-manually-configure-your-application-and-code-sample"></a>Opción 2 (Manual): registrar y configurar manualmente la aplicación y el código de ejemplo
>
> #### <a name="step-1-register-your-application"></a>Paso 1: Registrar su aplicación
>
> 1. Inicie sesión en [Azure Portal](https://portal.azure.com).
> 1. Si la cuenta proporciona acceso a más de un inquilino, seleccione la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Azure AD que desee utilizar.
> 1. Seleccione [App registrations](https://go.microsoft.com/fwlink/?linkid=2083908) (Registros de aplicaciones).
> 1. Seleccione **Nuevo registro**.
> 1. Cuando se abra la página **Registrar una aplicación**, escriba el nombre de su aplicación.
> 1. En **Supported account types** (Tipos de cuenta compatibles), seleccione **Accounts in any organizational directory and personal Microsoft accounts** (Cuentas en cualquier directorio de organización y cuentas personales de Microsoft).
> 1. Seleccione **Registrar**. En la página de **información general** de la aplicación, anote el valor del **Identificador de aplicación (cliente)** para su uso posterior.
> 1. En el panel izquierdo de la aplicación registrada, seleccione **Autenticación**.
> 1. En **Configuraciones de plataforma**, seleccione **Agregar una plataforma**. En el panel que se abre, seleccione **Aplicación de página única**.
> 1. Establezca el valor de **URI de redireccionamiento** en `http://localhost:3000/`.
> 1. Seleccione **Configurar**.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-your-application-in-the-azure-portal"></a>Paso 1: Configuración de la aplicación en Azure Portal
> Para que el código de ejemplo de este inicio rápido funcione, tiene que agregar un valor de `redirectUri` de `http://localhost:3000/`.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Realizar estos cambios por mí]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Ya configurada](media/quickstart-v2-javascript/green-check.png) La aplicación está configurada con estos atributos.

#### <a name="step-2-download-the-project"></a>Paso 2: Descarga del proyecto

> [!div renderon="docs"]
> Para ejecutar el proyecto con un servidor web con Node.js. [descargue los archivos principales del proyecto](https://github.com/Azure-Samples/ms-identity-javascript-v2/archive/master.zip).

> [!div renderon="portal" class="sxs-lookup"]
> Ejecutar el proyecto con un servidor Web mediante Node.js

> [!div renderon="portal" class="sxs-lookup" id="autoupdate" class="nextstepaction"]
> [Descargar el código de ejemplo](https://github.com/Azure-Samples/ms-identity-javascript-v2/archive/master.zip)

> [!div renderon="docs"]
> #### <a name="step-3-configure-your-javascript-app"></a>Paso 3: Configuración de la aplicación de JavaScript
>
> En la carpeta *app*, abra el archivo *authConfig.js* y actualice los valores `clientID`, `authority` y `redirectUri` del objeto `msalConfig`.
>
> ```javascript
> // Config object to be passed to Msal on creation
> const msalConfig = {
>   auth: {
>     clientId: "Enter_the_Application_Id_Here",
>     authority: "Enter_the_Cloud_Instance_Id_HereEnter_the_Tenant_Info_Here",
>     redirectUri: "Enter_the_Redirect_Uri_Here",
>   },
>   cache: {
>     cacheLocation: "sessionStorage", // This configures where your cache will be stored
>     storeAuthStateInCookie: false, // Set this to "true" if you are having issues on IE11 or Edge
>   }
> };
> ```

> [!div renderon="portal" class="sxs-lookup"]
> > [!NOTE]
> > `Enter_the_Supported_Account_Info_Here`

> [!div renderon="docs"]
>
> Modifique los valores de la sección `msalConfig` como se describe aquí:
>
> - `Enter_the_Application_Id_Here` es el **identificador de aplicación (cliente)** de la aplicación que registró.
> - `Enter_the_Cloud_Instance_Id_Here` es la instancia de la nube de Azure. En el caso de la nube principal o global de Azure, escriba `https://login.microsoftonline.com/`. Para nubes **nacionales** (por ejemplo, China), consulte [Nubes nacionales](authentication-national-cloud.md).
> - `Enter_the_Tenant_info_here` se establece en una de las opciones siguientes:
>   - Si la aplicación admite *cuentas de este directorio organizativo*, reemplace este valor por los valores de **Id. de inquilino** o **Nombre de inquilino**. Por ejemplo, `contoso.microsoft.com`.
>   - Si la aplicación admite *cuentas de cualquier directorio organizativo*, reemplace este valor por `organizations`.
>   - Si la aplicación admite *cuentas de cualquier directorio organizativo y cuentas Microsoft personales*, reemplace este valor por `common`. **En este inicio rápido**, use `common`.
>   - Para restringir la compatibilidad a *solo cuentas de Microsoft personales*, reemplace este valor por `consumers`.
> - `Enter_the_Redirect_Uri_Here` es `http://localhost:3000/`.
>
> El valor `authority` en *authConfig.js* debe ser similar al siguiente si usa la nube principal (global) de Azure:
>
> ```javascript
> authority: "https://login.microsoftonline.com/common",
> ```
>
> > [!TIP]
> > Para encontrar los valores de **Identificador de aplicación (cliente)** , **Identificador de directorio (inquilino)** y **Tipos de cuenta admitidos**, vaya a la página **Información general** del registro de aplicaciones en Azure Portal.
>
> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-3-your-app-is-configured-and-ready-to-run"></a>Paso 3: La aplicación está configurada y lista para ejecutarse
> Hemos configurado el proyecto con los valores de las propiedades de su aplicación.

> [!div renderon="docs"]
>
> Después, en la misma carpeta, edite el archivo *graphConfig.js* y actualice los valores `graphMeEndpoint` y `graphMailEndpoint` del objeto `apiConfig`.
>
> ```javascript
>   // Add here the endpoints for MS Graph API services you would like to use.
>   const graphConfig = {
>     graphMeEndpoint: "Enter_the_Graph_Endpoint_Herev1.0/me",
>     graphMailEndpoint: "Enter_the_Graph_Endpoint_Herev1.0/me/messages"
>   };
>
>   // Add here scopes for access token to be used at MS Graph API endpoints.
>   const tokenRequest = {
>       scopes: ["Mail.Read"]
>   };
> ```
>
> [!div renderon="docs"]
>
> `Enter_the_Graph_Endpoint_Here` es el punto de conexión en el que se realizarán las llamadas API. Como servicio principal de Microsoft Graph API (global), escriba `https://graph.microsoft.com/` (incluya la barra diagonal final). Para más información sobre Microsoft Graph en nubes nacionales, consulte [Implementaciones de nube nacionales](/graph/deployments).
>
> Los valores `graphMeEndpoint` y `graphMailEndpoint` del archivo *graphConfig.js* deben ser similares a los siguientes si usa el servicio principal de Microsoft Graph API (global):
>
> ```javascript
> graphMeEndpoint: "https://graph.microsoft.com/v1.0/me",
> graphMailEndpoint: "https://graph.microsoft.com/v1.0/me/messages"
> ```
>
> #### <a name="step-4-run-the-project"></a>Paso 4: Ejecución del proyecto

Ejecute el proyecto con un servidor web mediante Node.js:

1. Para iniciar el servidor, ejecute los siguientes comandos desde el directorio del proyecto:
    ```console
    npm install
    npm start
    ```
1. Vaya a `http://localhost:3000/`.

1. Seleccione **Iniciar sesión** para iniciar el proceso de inicio de sesión y, luego, llame a Microsoft Graph API.

    La primera vez que inicie sesión, se le pedirá que dé su consentimiento para permitir que la aplicación acceda a su perfil e inicie sesión automáticamente. Después de que haya iniciado sesión correctamente, la información del perfil de usuario debe mostrarse en la página.

## <a name="more-information"></a>Más información

### <a name="how-the-sample-works"></a>Funcionamiento del ejemplo

![Diagrama que muestra el flujo de código de autorización para una aplicación de página única.](media/quickstart-v2-javascript-auth-code/diagram-01-auth-code-flow.png)

### <a name="msaljs"></a>msal.js

La biblioteca MSAL.js inicia la sesión de los usuarios y solicita los tokens que se usan para acceder a una API protegida por la plataforma de identidad de Microsoft. El archivo *index.html* del ejemplo contiene una referencia a la biblioteca:

```html
<script type="text/javascript" src="https://alcdn.msauth.net/browser/2.0.0-beta.0/js/msal-browser.js" integrity=
"sha384-r7Qxfs6PYHyfoBR6zG62DGzptfLBxnREThAlcJyEfzJ4dq5rqExc1Xj3TPFE/9TH" crossorigin="anonymous"></script>
```

Si tiene Node.js instalado, también puede descargar la versión más reciente mediante el administrador de paquetes de Node.js (npm):

```console
npm install @azure/msal-browser
```

## <a name="next-steps"></a>Pasos siguientes

Para encontrar una guía paso a paso más detallada sobre la compilación de la aplicación usada en este inicio rápido, consulte:

> [!div class="nextstepaction"]
> [Tutorial para iniciar sesión y llamar a MS Graph](./tutorial-v2-javascript-auth-code.md).