---
title: 'Inicio de sesión de usuarios en aplicaciones de página única de Angular: Azure'
titleSuffix: Microsoft identity platform
description: Obtenga información sobre la forma en que una aplicación de Angular puede llamar a una API que requiera tokens de acceso mediante el uso de la plataforma de identidad de Microsoft.
services: active-directory
author: jasonnutter
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started, languages:JavaScript, devx-track-javascript
ms.topic: quickstart
ms.workload: identity
ms.date: 03/18/2020
ms.author: janutter
ms.openlocfilehash: bf921ce4ce3ad65f0787ccab2f0a064da8973af1
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88115244"
---
# <a name="quickstart-sign-in-users-and-get-an-access-token-in-an-angular-single-page-application"></a>Inicio rápido: Inicio de sesión de usuarios y obtención de un token de acceso en un aplicación de página única de Angular

En este inicio rápido se usa un código de ejemplo que muestra cómo una aplicación de página única (SPA) de Angular puede iniciar sesión en usuarios que tienen cuentas personales, profesionales o educativas de Microsoft. Una SPA de Angular también puede obtener un token de acceso para llamar a Microsoft Graph API o a cualquier API web.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción de Azure. [cree una de forma gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Node.js](https://nodejs.org/en/download/).
* [Visual Studio Code](https://code.visualstudio.com/download) para editar los archivos del proyecto o [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) para ejecutar el proyecto.

> [!div renderon="docs"]
> ## <a name="register-and-download-the-quickstart-app"></a>Registro y descarga de la aplicación de inicio rápido
> Para iniciar la aplicación de inicio rápido, use cualquiera de las siguientes opciones.
>
> ### <a name="option-1-express-register-and-automatically-configure-the-app-and-then-download-the-code-sample"></a>Opción 1 (rápido): Registrar y configurar de modo automático la aplicación y, luego, descargar el código de ejemplo
>
> 1. Inicie sesión en [Azure Portal](https://portal.azure.com).
> 1. Si la cuenta tiene acceso a más de un inquilino, seleccione la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Active Directory (Azure AD) que desee utilizar.
> 1. Abra el nuevo panel de [Registros de aplicaciones](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/ApplicationsListBlade/quickStartType/JavascriptSpaQuickstartPage/sourceType/docs) de Azure Portal.
> 1. Escriba un nombre para la aplicación y seleccione **Registrar**.
> 1. Vaya al panel de inicio rápido y vea el inicio rápido de Angular. Siga las instrucciones para descargar y configurar automáticamente la nueva aplicación.
>
> ### <a name="option-2-manual-register-and-manually-configure-the-application-and-code-sample"></a>Opción 2 (manual): registrar y configurar manualmente tanto la aplicación como el código de ejemplo.
>
> #### <a name="step-1-register-the-application"></a>Paso 1: Registro de la aplicación
>
> 1. Inicie sesión en [Azure Portal](https://portal.azure.com).
> 1. Si la cuenta tiene acceso a más de un inquilino, seleccione la cuenta en la esquina superior derecha y establezca la sesión del portal en el inquilino de Azure AD que desee utilizar.
> 1. Siga las instrucciones para [registrar una aplicación de página única](./scenario-spa-app-registration.md) en Azure Portal.
> 1. Agregue una nueva plataforma en el panel **Autenticación** del registro de la aplicación y registre el identificador URI de redireccionamiento: `http://localhost:4200/`.
> 1. En este inicio rápido se usa el [flujo de concesión implícita](v2-oauth2-implicit-grant-flow.md). Seleccione la configuración de **Concesión implícita** para **Tokens de identificador** y **Tokens de acceso**. Tanto los tokens de identificador como los tokens de acceso son obligatorios, ya que esta aplicación inicia la sesión de los usuarios y llama a una API.

> [!div class="sxs-lookup" renderon="portal"]
> #### <a name="step-1-configure-the-application-in-the-azure-portal"></a>Paso 1: Configurar la aplicación en Azure Portal
> Para que el código de ejemplo de este inicio rápido funcione, tiene que agregar un identificador URI de redirección como **http://localhost:4200/** y habilitar **Concesión implícita**.
> > [!div renderon="portal" id="makechanges" class="nextstepaction"]
> > [Realizar estos cambios por mí]()
>
> > [!div id="appconfigured" class="alert alert-info"]
> > ![Ya configurada](media/quickstart-v2-javascript/green-check.png) La aplicación está configurada con estos atributos.

#### <a name="step-2-download-the-code-sample"></a>Paso 2: Descargar el código de ejemplo
>[!div renderon="docs"]
>Para ejecutar el proyecto con un servidor web mediante Node.js, [clone el repositorio de ejemplo](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular) o [descargue los archivos principales del proyecto](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular/archive/master.zip). Abra los archivos mediante un editor como Visual Studio Code.

> [!div renderon="portal" id="autoupdate" class="sxs-lookup nextstepaction"]
> [Descargar el código de ejemplo](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular/archive/master.zip)

> [!div renderon="docs"]
>#### <a name="step-3-configure-the-javascript-app"></a>Paso 3: Configurar la aplicación de JavaScript
>
> En la carpeta *src/app*, edite *app.module.ts* y establezca los valores `clientId` y `authority` en `MsalModule.forRoot`.
>
>```javascript
>MsalModule.forRoot({
>    auth: {
>        clientId: 'Enter_the_Application_Id_here', // This is your client ID
>        authority: 'https://login.microsoftonline.com/Enter_the_Tenant_Info_Here', // This is your tenant info
>        redirectUri: 'Enter_the_Redirect_Uri_Here' // This is your redirect URI
>    },
>    cache: {
>        cacheLocation: 'localStorage',
>        storeAuthStateInCookie: isIE, // set to true for IE 11
>    },
>},
> //... )
>```

> [!div class="sxs-lookup" renderon="portal"]
> > [!NOTE]
> > Enter_the_Supported_Account_Info_Here


> [!div renderon="docs"]
>
> Reemplace estos valores:
>
>|Nombre del valor|Descripción|
>|---------|---------|
>|Enter_the_Application_Id_Here|En la página de **información general** del registro de la aplicación, corresponde al valor de **Id. de aplicación (cliente)** . |
>|Enter_the_Cloud_Instance_Id_Here|Esta es la instancia de la nube de Azure. En el caso de la nube principal o global de Azure, escriba **https://login.microsoftonline.com** . Para las nubes nacionales (por ejemplo, China) consulte [Nubes nacionales](./authentication-national-cloud.md).|
>|Enter_the_Tenant_Info_Here| Seleccione una de las siguientes opciones: Si la aplicación admite las *cuentas de este directorio organizativo*, reemplace este valor por el identificador del directorio (inquilino) o por el nombre del inquilino (por ejemplo, **contoso.microsoft.com**). Si la aplicación admite *cuentas en cualquier directorio organizativo*, reemplace este valor por **organizaciones**. Si la aplicación admite *cuentas en cualquier directorio organizativo y cuentas Microsoft personales*, reemplace este valor por **común**. Para restringir la compatibilidad a *Personal Microsoft accounts only* (Solo cuentas Microsoft personales), reemplace este valor por **consumidores**. |
>|Enter_the_Redirect_Uri_Here|Reemplácelo por **http://localhost:4200** .|
>|cacheLocation  | (Opcional) Establece el almacenamiento del explorador para el estado de autenticación. El valor predeterminado es **sessionStorage**.   |
>|storeAuthStateInCookie  | (Opcional) Identifica la biblioteca que almacena el estado de la solicitud de autenticación. Este estado es necesario para validar los flujos de autenticación en las cookies del explorador. Esta cookie se establece para Internet Explorer y Edge, para aceptar esos dos exploradores. Para más información, consulte el artículo en el que se explican los [problemas conocidos](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki/Known-issues->on-IE-and-Edge-Browser#issues). |
> > [!TIP]
> > Para buscar los valores de **Identificador de aplicación (cliente)** , **Identificador de directorio (inquilino)** y **Tipos de cuenta admitidos**, vaya a la página **Información general** en Azure Portal.

Para más información acerca de las opciones configurables disponibles, consulte [Inicializar aplicaciones cliente](msal-js-initializing-client-applications.md).

Puede encontrar el código fuente de la biblioteca MSAL.js en el repositorio [AzureAD/microsoft-authentication-library-for-js](https://github.com/AzureAD/microsoft-authentication-library-for-js) de GitHub.

>[!div class="sxs-lookup" renderon="portal"]
>#### <a name="step-3-run-the-project"></a>Paso 3: Ejecución del proyecto

>[!div renderon="docs"]
>#### <a name="step-4-run-the-project"></a>Paso 4: Ejecución del proyecto

Si usa Node.js:

1. Inicie el servidor mediante la ejecución de los siguientes comandos desde el directorio del proyecto:

   ```console
   npm install
   npm start
   ```

1. Vaya a **http://localhost:4200/** .
1. Seleccione **Login** (Iniciar sesión).
1. Seleccione **Perfil** para llamar a Microsoft Graph.

Cuando el explorador haya cargado la aplicación, seleccione **Iniciar sesión**. La primera vez que empiece a iniciar sesión, se le pedirá que dé su consentimiento para permitir que la aplicación acceda a su perfil e inicie sesión automáticamente. Después de que haya iniciado sesión correctamente, seleccione **Perfil** y la información del perfil de usuario se mostrará en la página.

## <a name="how-the-sample-works"></a>Funcionamiento del ejemplo

![Diagrama del funcionamiento de la aplicación de ejemplo en este inicio rápido](./media/quickstart-v2-angular/diagram-auth-flow-spa-angular.svg)


## <a name="next-steps"></a>Pasos siguientes

A continuación, aprenda a iniciar sesión en un usuario y adquirir tokens en el tutorial de Angular:

> [!div class="nextstepaction"]
> [Tutorial de Angular](./tutorial-v2-angular.md)