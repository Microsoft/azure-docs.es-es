---
title: 'Registro de una API web que permite iniciar sesión a los usuarios: Plataforma de identidad de Microsoft | Azure'
description: Aprenda a registrar una aplicación web que permite iniciar sesión a los usuarios
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/14/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: b16ee98b44e52482423229a0940c8927a94d4c83
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88118678"
---
# <a name="web-app-that-signs-in-users-app-registration"></a>Aplicación web que inicia sesión de usuarios: Registro de aplicación

En este artículo se explican los detalles del registro de la aplicación para una aplicación web que inicia la sesión de los usuarios.

Para registrar la aplicación, puede usar lo siguiente:

- Los [inicios rápidos de aplicación web](#register-an-app-by-using-the-quickstarts). Además de ser una excelente primera experiencia con la creación de una aplicación, las guías de inicio rápido de Azure Portal incluyen un botón denominado **Hacer este cambio por mí**. Puede usar este botón para establecer las propiedades que necesita, incluso para una aplicación existente. Deberá adaptar los valores de estas propiedades a su propio caso. En concreto, la dirección URL de la API web para la aplicación probablemente será diferente de la URL propuesta de forma predeterminada, lo cual afectará también al URI de cierre de sesión.
- Azure Portal para [registrar la aplicación manualmente](#register-an-app-by-using-the-azure-portal).
- PowerShell y herramientas de línea de comandos.

## <a name="register-an-app-by-using-the-quickstarts"></a>Registro de una aplicación con las guías de inicio rápido

Puede usar estos vínculos para el arranque de la creación de la aplicación web:

- [ASP.NET Core](https://aka.ms/aspnetcore2-1-aad-quickstart-v2)
- [ASP.NET](https://ms.portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/applicationsListBlade/quickStartType/AspNetWebAppQuickstartPage/sourceType/docs)

## <a name="register-an-app-by-using-the-azure-portal"></a>Registro de una aplicación mediante Azure Portal

> [!NOTE]
> El portal que se debe usar varía en función de si la aplicación se ejecuta en la nube pública de Microsoft Azure o en una nube soberana o nacional. Para obtener más información, consulte [Nubes nacionales](./authentication-national-cloud.md#app-registration-endpoints).


1. Inicie sesión en [Azure Portal](https://portal.azure.com) con una cuenta personal, profesional o educativa de Microsoft. Como alternativa, inicie sesión en la instancia de [Azure Portal que prefiera](./authentication-national-cloud.md#app-registration-endpoints) de la nube nacional.
2. Si la cuenta proporciona acceso a más de un inquilino, seleccione su cuenta en la esquina superior derecha. A continuación, establezca la sesión del portal en el inquilino de Azure Active Directory (Azure AD) deseado.
3. En el panel izquierdo, seleccione el servicio **Azure Active Directory** y, después, seleccione **Registros de aplicaciones** > **Nuevo registro**.

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

1. Cuando aparece la página **Registrar una aplicación**, escriba la información de registro de la aplicación:
   1. Elija los tipos de cuenta admitidos para la aplicación. (Consulte [Tipos de cuenta admitidos](./v2-supported-account-types.md)).
   1. En la sección **Nombre**, escriba un nombre con sentido para la aplicación, que se mostrará a los usuarios de la aplicación. Por ejemplo, escriba **AspNetCore-WebApp**.
   1. En **URI de redireccionamiento**, agregue el tipo de aplicación y el destino de URI que aceptará las respuestas de token devueltas después de una autenticación correcta. Por ejemplo, escriba: **https://localhost:44321** . Después, seleccione **Registrar**.
   ![registro](media/scenario-webapp/scenario-webapp-app-registration-1.png)
1. Seleccione el menú **Autenticación** y, a continuación, agregue la siguiente información:
   1. En **URL de respuesta**, agregue **https://localhost:44321/signin-oidc** de tipo **Web**.
   1. En la sección **Configuración avanzada**, establezca **Dirección URL de cierre de sesión** en **https://localhost:44321/signout-oidc** .
   1. En **Concesión implícita**, seleccione **Tokens de id.** .
   1. Seleccione **Guardar**.
  ![registro](media/scenario-webapp/scenario-webapp-app-registration-2.png)
 
# <a name="aspnet"></a>[ASP.NET](#tab/aspnet)

1. Cuando aparece la página **Registrar una aplicación**, escriba la información de registro de la aplicación:
   1. Elija los tipos de cuenta admitidos para la aplicación. (Consulte [Tipos de cuenta admitidos](./v2-supported-account-types.md)).
   1. En la sección **Nombre**, escriba un nombre con sentido para la aplicación, que se mostrará a los usuarios de la aplicación. Por ejemplo, escriba **MailApp-openidconnect-v2**.
   1. En la sección **URI de redirección (opcional)** , seleccione **Web** en el cuadro combinado y escriba los siguientes URI de redirección: **https://localhost:44326/** .
1. Seleccione **Registrar** para crear la aplicación.
1. Seleccione el menú **Autenticación**.
1. En la sección **Configuración avanzada** | **Concesión implícita**, seleccione **Tokens de id.** Para este ejemplo, se requiere que habilite el [flujo de concesión implícita](v2-oauth2-implicit-grant-flow.md).
1. Seleccione **Guardar**.

# <a name="java"></a>[Java](#tab/java)

1. Cuando aparezca la página **Registrar una aplicación**, escriba un nombre para mostrar de la aplicación. Por ejemplo, escriba **java-webapp**.
1. Seleccione **Cuentas en cualquier directorio organizativo y cuentas Microsoft personales (por ejemplo, Skype, Xbox, Outlook.com)** y luego seleccione **Aplicación web o API** para **Tipo de aplicación**.
1. Seleccione **Registrar** para registrar la aplicación.
1. En el menú izquierdo, seleccione **Autenticación**. En **URI de redirección**, seleccione **Web**.

1. Escriba dos URI de redirección: uno para la página de inicio de sesión y otro para la página de gráfico. En ambos casos, use el mismo host y número de puerto, seguido de **/msal4jsample/secure/aad** para la página de inicio de sesión y de **msal4jsample/graph/me** para la página de información de usuario.

   De forma predeterminada, en el ejemplo se usa:

   - **http://localhost:8080/msal4jsample/secure/aad**
   - **http://localhost:8080/msal4jsample/graph/me**

  Después, seleccione **Guardar**.

1. Seleccione **Certificados y secretos** en el menú.
1. En la sección **Secretos de cliente**, seleccione **Nuevo secreto de cliente** y, a continuación, haga lo siguiente:

   1. Escriba una descripción de la clave.
   1. Seleccione la duración de clave **En 1 año**.
   1. Seleccione **Agregar**.
   1. Cuando aparezca el valor de clave, cópielo para usarlo más adelante. Este valor no se volverá a mostrar ni se podrá recuperar por otro medio.

# <a name="python"></a>[Python](#tab/python)

1. Cuando aparece la página **Registrar una aplicación**, escriba la información de registro de la aplicación:
   1. En la sección **Nombre**, escriba un nombre con sentido para la aplicación, que se mostrará a los usuarios de la aplicación. Por ejemplo, escriba **python-webapp**.
   1. Cambie **Supported account types** (Tipos de cuenta compatibles) por **Cuentas en cualquier directorio organizativo y cuentas Microsoft personales (por ejemplo, Skype, Xbox, Outlook.com)** .
   1. En la sección **URI de redirección (opcional)** , seleccione **Web** en el cuadro combinado y escriba los siguientes URI de redirección: **http://localhost:5000/getAToken** .
1. Seleccione **Registrar** para crear la aplicación.
1. En la página **Información general** de la aplicación, busque el valor de **Id. de aplicación (cliente)** y regístrelo para usarlo más tarde. Lo necesitará para configurar el archivo de configuración de Visual Studio para este proyecto.
1. En el menú izquierdo, seleccione **Certificados y secretos**.
1. En la sección **Secretos de cliente**, seleccione **Nuevo secreto de cliente** y, a continuación, haga lo siguiente:

   1. Escriba una descripción de la clave.
   1. Seleccione una duración de la clave de **En 1 año**.
   1. Seleccione **Agregar**.
   1. Cuando aparezca el valor de clave, cópielo. Lo necesitará más adelante.
---

## <a name="register-an-app-by-using-powershell"></a>Registro de una aplicación mediante PowerShell

> [!NOTE]
> Actualmente, Azure AD PowerShell crea aplicaciones con solo los siguientes tipos de cuenta compatibles:
>
> - MyOrg (solo las cuentas de este directorio organizativo)
> - AnyOrg (cuentas de cualquier directorio organizativo)
>
> Puede crear una aplicación que inicie la sesión de los usuarios con sus cuentas personales de Microsoft (por ejemplo, Skype, Xbox o Outlook.com). En primer lugar, cree una aplicación de varios inquilinos. Los tipos de cuenta compatibles son cuentas de cualquier directorio organizativo. A continuación, cambie la propiedad [`accessTokenAcceptedVersion`](./reference-app-manifest.md#accesstokenacceptedversion-attribute) a **2** y la propiedad [`signInAudience`](./reference-app-manifest.md#signinaudience-attribute) a `AzureADandPersonalMicrosoftAccount` en el [manifiesto de aplicación](./reference-app-manifest.md) del Azure Portal. Para más información, consulte el [paso 1.3](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/1-WebApp-OIDC/1-3-AnyOrgOrPersonal#step-1-register-the-sample-with-your-azure-ad-tenant) en el tutorial ASP.NET Core. Puede generalizar este paso para las aplicaciones web en cualquier lenguaje.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración del código de la aplicación](scenario-web-app-sign-user-app-configuration.md)