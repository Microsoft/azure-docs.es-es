---
title: 'Publicación de aplicaciones cliente nativas: Azure AD | Microsoft Docs'
description: Explica cómo habilitar las aplicaciones cliente nativas para comunicarse con el conector del proxy de la aplicación de Azure AD para proporcionar acceso remoto seguro a las aplicaciones locales.
services: active-directory
documentationcenter: ''
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/12/2020
ms.author: kenwith
ms.reviewer: japere
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3f8494852bcff49602645c940470b529302f119f
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165080"
---
# <a name="how-to-enable-native-client-applications-to-interact-with-proxy-applications"></a>Habilitación de las aplicaciones cliente nativas para interactuar con el proxy de aplicaciones

Puede usar Azure Active Directory (Azure AD) Application Proxy para publicar aplicaciones web, pero también se puede utilizar para publicar las aplicaciones cliente nativas que se configuran con la Biblioteca de autenticación de Microsoft (MSAL). Las aplicaciones cliente nativas son distintas de las aplicaciones web porque se instalan en un dispositivo, mientras a las aplicaciones web se accede mediante un explorador.

Para admitir aplicaciones cliente nativas, Application Proxy acepta tokens emitidos por Azure AD que se envían en el encabezado. El servicio Application Proxy realiza la autenticación para los usuarios. Esta solución no utiliza tokens de aplicación para la autenticación.

![Relación entre los usuarios finales, Azure AD y las aplicaciones publicadas](./media/application-proxy-configure-native-client-application/richclientflow.png)

Utilice la biblioteca de autenticación de Microsoft, que se encarga de la autenticación y admite muchos de los entornos de cliente, para publicar aplicaciones nativas. El proxy de aplicación encaja en la [aplicación de escritorio que llama a una API web en nombre de un escenario de usuario con sesión iniciada](https://docs.microsoft.com/azure/active-directory/develop/authentication-flows-app-scenarios#desktop-app-that-calls-a-web-api-on-behalf-of-a-signed-in-user).

Este artículo le guiará por los cuatro pasos para publicar una aplicación nativa con el proxy de aplicación y la biblioteca de Autenticación de Azure AD.

## <a name="step-1-publish-your-proxy-application"></a>Paso 1: Publique su aplicación de proxy

Publique su aplicación de proxy al igual que haría con cualquier otra aplicación y asigne a los usuarios el acceso a la aplicación. Para más información, consulte [Publicación de aplicaciones mediante el proxy de aplicación de Azure AD](application-proxy-add-on-premises-application.md).

## <a name="step-2-register-your-native-application"></a>Paso 2: Registre la aplicación nativa

Ahora debe registrar la aplicación en Azure AD:

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com/). Aparecerá el **panel** del **Centro de administración de Azure Active Directory**.
1. En la barra lateral, seleccione **Azure Active Directory**. Se mostrará la página de introducción de **Azure Active Directory**.
1. En la barra lateral de introducción de Azure AD, seleccione **Registros de aplicaciones**. Aparece la lista de todos los registros de aplicaciones.
1. Seleccione **Nuevo registro**. Se muestra la página **Registrar una aplicación**.

   ![Creación de un nuevo registro de aplicaciones en Azure Portal](./media/application-proxy-configure-native-client-application/create.png)

1. En el encabezado **Nombre**, especifique un nombre para mostrar a los usuarios en la aplicación.
1. En el encabezado **Tipos de cuentas admitidos**, seleccione un nivel de acceso mediante estas instrucciones:

   - Para seleccionar como destino únicamente las cuentas que son internas de su organización, seleccione **Solo las cuentas de este directorio organizativo**.
   - Seleccione **Cuentas en cualquier directorio organizativo** si desea tener como destino todos los clientes de negocios y del sector educativo.
   - Para establecer como destino el mayor conjunto posible de clientes, seleccione **Cuentas en cualquier directorio organizativo y cuentas Microsoft personales**.
1. En **URI de redireccionamiento**, seleccione **Cliente público (móvil y de escritorio)** y, a continuación, escriba el URI de redireccionamiento `https://login.microsoftonline.com/common/oauth2/nativeclient` de su aplicación.
1. Seleccione y lea las **directivas de la plataforma de Microsoft** y, a continuación, seleccione **Registrar**. Se crea el registro de aplicación y se muestra la página de introducción.

Para obtener más información sobre cómo crear un registro de aplicación, consulte [Integración de aplicaciones con Azure Active Directory](../develop/quickstart-register-app.md).

## <a name="step-3-grant-access-to-your-proxy-application"></a>Paso 3: Conceda acceso a la aplicación de proxy

Ahora que ha registrado su aplicación nativa, puede darle acceso a otras aplicaciones de su directorio, en este caso, para tener acceso a la aplicación de proxy. Para permitir que la aplicación nativa se exponga en el proxy de aplicación:

1. En la barra lateral de la nueva página de registro de aplicaciones, seleccione **Permisos de API**. Se muestra la página **Permisos de API** del nuevo registro de aplicaciones.
1. Seleccione **Agregar un permiso**. Aparece la página **Solicitud de permisos de API**.
1. En el valor **Seleccionar una API**, elija **API usadas en mi organización**. Aparece una lista que contiene las aplicaciones del directorio que exponen las API.
1. Escriba en el cuadro de búsqueda o desplácese para buscar el proxy de aplicación que publicó en [Paso 1: Publique su aplicación de proxy](https://docs.microsoft.com/azure/active-directory/manage-apps/application-proxy-configure-native-client-application#step-1-publish-your-proxy-application) y, a continuación, seleccione el proxy de aplicación.
1. En el encabezado **¿Qué tipo de permiso necesita la aplicación web?** , seleccione el tipo de permiso. Si su aplicación nativa necesita acceder a la API de proxy de aplicación como el usuario que inició sesión, elija **Permisos delegados**.
1. En el encabezado **Seleccionar permisos**, seleccione el permiso deseado y seleccione **Agregar permisos**. La página **Permisos de API** de su aplicación nativa ahora muestra el proxy de aplicación y el permiso de API que ha agregado.

## <a name="step-4-add-the-microsoft-authentication-library-to-your-code-net-c-sample"></a>Paso 4: Agregar la biblioteca de autenticación de Microsoft al código (muestra de .NET C#)

Edite el código de la aplicación nativa en el contexto de autenticación de la biblioteca de autenticación de Microsoft (MSAL) para incluir el siguiente texto: 

```         
// Acquire Access Token from AAD for Proxy Application
IPublicClientApplication clientApp = PublicClientApplicationBuilder
.Create(<App ID of the Native app>)
.WithDefaultRedirectUri() // will automatically use the default Uri for native app
.WithAuthority("https://login.microsoftonline.com/{<Tenant ID>}")
.Build();

AuthenticationResult authResult = null;
var accounts = await clientApp.GetAccountsAsync();
IAccount account = accounts.FirstOrDefault();

IEnumerable<string> scopes = new string[] {"<Scope>"};

try
 {
    authResult = await clientApp.AcquireTokenSilent(scopes, account).ExecuteAsync();
 }
    catch (MsalUiRequiredException ex)
 {
     authResult = await clientApp.AcquireTokenInteractive(scopes).ExecuteAsync();                
 }

if (authResult != null)
 {
  //Use the Access Token to access the Proxy Application

  HttpClient httpClient = new HttpClient();
  HttpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", authResult.AccessToken);
  HttpResponseMessage response = await httpClient.GetAsync("<Proxy App Url>");
 }
```

La información necesaria del código de ejemplo puede encontrarse en el portal de Azure AD:

| Información necesaria | Cómo encontrarla en el portal de Azure AD |
| --- | --- |
| \<Tenant ID> | **Azure Active Directory** > **Propiedades** > **Id. de directorio** |
| \<App ID of the Native app> | **Registro de aplicaciones** > *la aplicación nativa* > **Información general** > **Identificador de la aplicación** |
| \<Scope> | **Registro de aplicaciones** > *la aplicación nativa* > **Permisos de API** > Haga clic en la API de permisos (user_impersonation) > Un panel con el título **user_impersonation** aparece en el lado derecho. > El ámbito es la dirección URL en el cuadro de edición.
| \<Proxy App Url> | la dirección URL externa y la ruta de acceso a la API

Cuando se edita el código de MSAL con estos parámetros, los usuarios podrán autenticarse en las aplicaciones cliente nativas incluso si están fuera de la red corporativa.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el flujo de la aplicación nativa, consulte [Aplicaciones nativas en Azure Active Directory](../azuread-dev/native-app.md).

Obtenga información sobre la configurar el [inicio de sesión único en aplicaciones de Azure Active Directory](sso-options.md#choosing-a-single-sign-on-method).