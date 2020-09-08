---
title: Llamada a una instancia de ASP.NET Web API protegida por una plataforma de identidad de Microsoft
description: En este inicio rápido, aprenderá a llamar a una instancia de ASP.NET Web API protegida por una plataforma de identidad de Microsoft desde una aplicación del Escritorio de Windows (WPF). El cliente de WPF autentica a un usuario, solicita un token de acceso y llama a la API web.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: quickstart
ms.workload: identity
ms.date: 12/12/2019
ms.author: jmprieur
ms.custom: devx-track-csharp, aaddev, identityplatformtop40, scenarios:getting-started, languages:ASP.NET
ms.openlocfilehash: 0fc31fd397f8206f7c6f0509dd03495631dde609
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165641"
---
# <a name="quickstart-call-an-aspnet-web-api-protected-by-microsoft-identity-platform"></a>Inicio rápido: Llamada a una instancia de ASP.NET Web API protegida por una plataforma de identidad de Microsoft

En este inicio rápido se expone una API web y se protegerá, con el fin de que solo pueda acceder a ella el usuario autenticado. En este ejemplo se muestra cómo exponer una instancia de ASP.NET Web API para que pueda aceptar los tokens que emiten no solo las cuentas personales (como outlook.com, live.com, etc.), sino también las cuentas profesionales y educativas de cualquier empresa u organización que se haya integrado en una plataforma de identidad de Microsoft.

El ejemplo también incluye un cliente de aplicación de escritorio de Windows (WPF) que muestra cómo se puede solicitar un token de acceso para acceder a una API web.

## <a name="prerequisites"></a>Requisitos previos

Para ejecutar este ejemplo, necesitará lo siguiente:

* Visual Studio 2017 o 2019.  Descargue [Visual Studio de forma gratuita](https://www.visualstudio.com/downloads/).

* Una [cuenta de Microsoft](https://www.outlook.com) o el [programa de desarrolladores de Microsoft 365](/office/developer-program/office-365-developer-program).

## <a name="download-or-clone-this-sample"></a>Descarga o clonación de este ejemplo

Puede clonar este ejemplo desde el shell o la línea de comandos:

  ```console
  git clone https://github.com/AzureADQuickStarts/AppModelv2-NativeClient-DotNet.git
  ```

O bien puede [descargar el código de ejemplo como un archivo ZIP](https://github.com/AzureADQuickStarts/AppModelv2-NativeClient-DotNet/archive/complete.zip).

## <a name="register-your-web-api-in-the-application-registration-portal"></a>Registro de una API web en el portal de registro de aplicación

### <a name="choose-the-azure-ad-tenant-where-you-want-to-create-your-applications"></a>Selección del inquilino de Azure AD en el que quiere crear las aplicaciones

Si quiere registrar las aplicaciones manualmente, como primer paso, necesitará:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft.
1. Si su cuenta existe en más de un inquilino de Azure AD, seleccione su perfil en la esquina superior derecha del menú de la parte superior de la página y luego **cambie de directorio**.
   Cambie la sesión del portal por el inquilino de Azure AD deseado.

### <a name="register-the-service-app-todolistservice"></a>Registro de la aplicación de servicio (TodoListService)

1. Vaya a la página [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) de la plataforma de identidad de Microsoft para desarrolladores.
1. Seleccione **Nuevo registro**.
1. Cuando aparece la página **Registrar una aplicación**, escriba la información de registro de la aplicación:
   - En la sección **Nombre**, escriba un nombre significativo para la aplicación, que se mostrará a los usuarios de la aplicación, por ejemplo, `AppModelv2-NativeClient-DotNet-TodoListService`.
   - Cambie los **Tipos de cuenta compatibles** a **Cuentas en cualquier directorio organizativo**.
   - Seleccione **Registrar** para crear la aplicación.

1. En la página **Información general** de la aplicación, busque el valor de **Id. de aplicación (cliente)** y regístrelo para usarlo más tarde. Lo necesitará para ajustar el archivo de configuración de Visual Studio para este proyecto (`ClientId` en `TodoListService\Web.config`).
1. Seleccione la sección **Exponer una API** y:
   - Seleccione **Agregar un ámbito**.
   - Acepte el URI de Id. de aplicación propuesto (api://{clientId}). Para ello, seleccione **Guardar y continuar**.
   - Escriba los siguientes parámetros:
     - para **Nombre del ámbito**, use `access_as_user`.
     - Asegúrese de que esté seleccionada la opción **Administradores y usuarios** para **¿Quién puede dar el consentimiento?**
     - en **Nombre para mostrar del consentimiento del administrador**, escriba `Access TodoListService as a user`.
     - en **Descripción del consentimiento del administrador**, escriba `Accesses the TodoListService web API as a user`.
     - en **Nombre para mostrar del consentimiento del usuario**, escriba `Access TodoListService as a user`.
     - en **Descripción del consentimiento del usuario**, escriba `Accesses the TodoListService web API as a user`.
     - Mantenga el **Estado** como **Habilitado**.
     - Seleccione **Agregar ámbito**.

### <a name="configure-the-service-project-to-match-the-registered-web-api"></a>Configuración del proyecto del servicio para que coincida con la API web registrada

1. Abra la solución en Visual Studio y después abra el archivo **Web.config** en la raíz de proyecto de **TodoListService**.
1. Reemplace el valor del parámetro `ida:ClientId` por el **Id. de cliente (Id. de aplicación)** de la aplicación que acaba de registrar en el portal de registro de aplicaciones.

### <a name="add-the-new-scope-to-the-todolistclients-appconfig"></a>Agregue el nuevo ámbito al archivo app.config de *TodoListClient*.

* Abra el archivo **app.config** ubicado en la carpeta raíz del proyecto **TodoListClient** y después pegue el **Id. de la aplicación** de la aplicación que acaba de registrar para *TodoListService* en el parámetro `TodoListServiceScope`, reemplazando la cadena `{Enter the Application ID of your TodoListService from the app registration portal}`.

  > [!NOTE]
  > Asegúrese de usar el siguiente formato:
  >
  > `api://{TodoListService-Application-ID}/access_as_user`
  >
  >(donde {TodoListService-Application-ID} es el GUID que representa al Id. de aplicación de TodoListService).

## <a name="register-the-client-app-todolistclient"></a>Registro de la aplicación cliente (TodoListClient)

En este paso, configurará el proyecto *TodoListClient* al registrar una nueva aplicación en el portal de registro de aplicaciones. En los casos en los que el cliente y el servidor se consideran *la misma aplicación* también puede volver a usar la misma aplicación registrada en el "Paso 2". Se necesita usar la misma aplicación si quiere que los usuarios inicien sesión con sus cuentas personales de Microsoft.

### <a name="register-the-todolistclient-application-in-the-application-registration-portal"></a>Registre la aplicación *TodoListClient* en el *portal de registro de aplicaciones*.

1. Vaya a la página [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) de la plataforma de identidad de Microsoft para desarrolladores.
1. Seleccione **Nuevo registro**.
1. Cuando aparece la página **Registrar una aplicación**, escriba la información de registro de la aplicación:
   - En la sección **Nombre**, escriba un nombre significativo para la aplicación, que se mostrará a los usuarios de la aplicación, por ejemplo, `NativeClient-DotNet-TodoListClient`.
   - Cambie los **Tipos de cuenta compatibles** a **Cuentas en cualquier directorio organizativo**.
   - Seleccione **Registrar** para crear la aplicación.
   
   > [!NOTE]
   > En el archivo **app.config** del proyecto de *TodoListClient*, el valor predeterminado de `ida:Tenant` se establece en `common`.
   >
   > `common` significa que puede iniciar sesión con una cuenta profesional o educativa o una cuenta personal de Microsoft (porque seleccionó **Cuentas en cualquier directorio de organización**).
   >
   > `organizations` significa que puede iniciar sesión con una cuenta profesional o educativa.
   >
   > `consumers` significa que solo puede iniciar sesión con una cuenta personal de Microsoft.
   >
   
1. En la página Información general de la aplicación, seleccione la sección **Autenticación**.
   1. En **Configuraciones de plataforma**, seleccione el botón **Agregar una plataforma**.
   1. En **Aplicaciones móviles y de escritorio**, seleccione las **aplicaciones móviles y de escritorio**.
   1. En **URI de redirección**, active la casilla **https://login.microsoftonline.com/common/oauth2/nativeclient** .
   1. Seleccione **Configurar**.   
1. Seleccione la sección **Permisos de API**.
   1. Seleccione el botón **Agregar un permiso**.
   1. Seleccione la pestaña **Mis API**.
   1. En la lista de las API, seleccione `AppModelv2-NativeClient-DotNet-TodoListService API` o el nombre que especificó para la API web.
   1. Active el permiso **access_as_user** si aún no está activado. Si es necesario, utilice el cuadro de búsqueda.
   1. Seleccione el botón **Agregar permisos**.

### <a name="configure-your-todolistclient-project"></a>Configuración del proyecto *TodoListClient*

1. En el *portal de registro de aplicaciones*, en la página **Información general**, copie el valor del **Id. de la aplicación (cliente)** .
1. Abra el archivo **app.config** ubicado en la carpeta raíz del proyecto **TodoListClient** y pegue el valor en el valor del parámetro `ida:ClientId`.

## <a name="run-your-project"></a>Ejecución del proyecto

1. Presione `<F5>` para ejecutar el proyecto. El proyecto *TodoListClient* debe abrirse.
1. Seleccione **Iniciar sesión** en la parte superior derecha e inicie sesión con el mismo usuario que ha usado para registrar la aplicación o un usuario en el mismo directorio.
1. En este momento, si inicia sesión por primera vez, es posible que se le pida que dé su consentimiento a la API web de *TodoListService*.
1. El inicio de sesión también solicita el token de acceso al ámbito *access_as_user* para acceder a la API web de *TodoListService* y manipular la lista *To-Do*.

## <a name="pre-authorize-your-client-application"></a>Preautorización de la aplicación cliente

Una de las formas de permitir que los usuarios de otros directorios accedan a la API web es *preautorizar* las aplicaciones cliente para tener acceso a la API web. Para ello, agregue los Id. de aplicación de las aplicaciones cliente en la lista de aplicaciones *preautorizadas* para su API web. Al agregar un cliente preautorizado, no será necesario que el usuario dé su consentimiento para usar la API web. Siga los pasos que se indican a continuación para preautorizar la aplicación web:

1. Vuelva al *portal de registro de aplicaciones* y abra las propiedades del elemento **TodoListService**.
1. En la sección **Exponer una API**, haga clic en **Agregar una aplicación cliente** en la sección *Aplicaciones cliente autorizadas*.
1. En el campo *Id. de cliente*, pegue el Id. de la aplicación `TodoListClient`.
1. En la sección *Ámbitos autorizados*, seleccione el ámbito de esta API web `api://<Application ID>/access_as_user`.
1. Presione el botón **Agregar aplicación** en la parte inferior de la página.

## <a name="run-your-project"></a>Ejecución del proyecto

1. Presione `<F5>` para ejecutar el proyecto. El proyecto *TodoListClient* debe abrirse.
1. Seleccione **Iniciar sesión** en la parte superior derecha (o Borrar caché/Iniciar sesión) y después inicie sesión con un cuenta personal de Microsoft (live.com o hotmail.com) o una cuenta profesional o educativa.

## <a name="optional-restrict-sign-in-access-to-your-application"></a>Opcional: Restringir el acceso de inicio de sesión a la aplicación

De forma predeterminada, cuando descargue este ejemplo de código y configure la aplicación para usar el punto de conexión de Azure Active Directory v2 siguiendo los pasos anteriores, las cuentas personales (como outlook.com, live.com, etc.) y las cuentas profesionales o educativas de cualquier organización integrada con Azure AD pueden solicitar tokens y acceder a la API web.

Para restringir quién puede iniciar sesión a la aplicación, use una de las siguientes opciones:

### <a name="option-1-restrict-access-to-a-single-organization-single-tenant"></a>Opción 1: Restringir el acceso a una sola organización (inquilino único)

Puede restringir el acceso al inicio de sesión de la aplicación para que solo puedan usarlo aquellas cuentas de usuario que forman parte de un solo inquilino de Azure AD, incluidas las *cuentas de invitado* de ese inquilino. Este es un escenario común para las *aplicaciones de línea de negocio*:

1. Abra el archivo **App_Start\Startup.Auth** y cambie el valor del punto de conexión de metadatos que se pasa al `OpenIdConnectSecurityTokenProvider` por `"https://login.microsoftonline.com/{Tenant ID}/v2.0/.well-known/openid-configuration"` (también puede usar el nombre del inquilino, como en `contoso.onmicrosoft.com`).
2. En el mismo archivo, establezca la propiedad `ValidIssuer` de los `TokenValidationParameters` en `"https://sts.windows.net/{Tenant ID}/"` y el argumento `ValidateIssuer` en `true`.

### <a name="option-2-use-a-custom-method-to-validate-issuers"></a>Opción 2: Usar un método personalizado para validar emisores

Puede implementar un método personalizado para validar los emisores con el parámetro **IssuerValidator**. Para más información acerca de cómo usar este parámetro, puede encontrar más detalles en la [clase TokenValidationParameters](/dotnet/api/microsoft.identitymodel.tokens.tokenvalidationparameters?view=azure-dotnet).

[!INCLUDE [Help and support](../../../includes/active-directory-develop-help-support-include.md)]

## <a name="next-steps"></a>Pasos siguientes
Obtenga más información sobre el escenario de API web protegida que admite la plataforma de identidad de Microsoft:
> [!div class="nextstepaction"]
> [escenario con una API web protegida](scenario-protected-web-api-overview.md)
