---
title: Ejemplos de código de la Plataforma de identidad de Microsoft
description: Se proporciona un índice de los ejemplos de código disponibles para la Plataforma de identidad de Microsoft (punto de conexión v2.0), organizados por escenario.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: sample
ms.workload: identity
ms.date: 06/01/2020
ms.author: marsma
ms.reviewer: jmprieur
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 142037366e9309f8087a2e42fa46cd0b4137f6b8
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119409"
---
# <a name="microsoft-identity-platform-code-samples-v20-endpoint"></a>Ejemplos de código de la Plataforma de identidad de Microsoft (punto de conexión v2.0)

Puede usar la plataforma de identidad de Microsoft para:

- Agregar autenticación y autorización a sus aplicaciones web y API web.
- Requerir un token de acceso para tener acceso a una API web protegida.

Este artículo describe brevemente y proporciona vínculos a ejemplos para el punto de conexión de la Plataforma de identidad de Microsoft. Estos ejemplos le muestran cómo se hace y también proporciona fragmentos de código que puede usar en sus aplicaciones. En la página de ejemplos de código, encontrará temas Léame detallados que le ayudarán con los requisitos, la instalación y la configuración. Los comentarios dentro del código lo ayudarán a comprender las secciones críticas.

> [!NOTE]
> Si está interesado en los ejemplos de v1.0, consulte los [ejemplos de código de Azure AD (punto de conexión v1.0)](../azuread-dev/sample-v1-code.md).

Para entender el escenario básico de cada tipo de ejemplo, consulte el artículo sobre los [tipos de aplicaciones para el punto de conexión de la Plataforma de identidad de Microsoft](v2-app-types.md).

También puede contribuir a los ejemplos en GitHub. Para saber cómo, vea [Ejemplos y documentación de Microsoft Azure Active Directory](https://github.com/Azure-Samples?page=3&query=active-directory).

## <a name="single-page-applications"></a>Aplicación de página única

En estos ejemplos se muestra cómo escribir una aplicación de página única protegida con la Plataforma de identidad de Microsoft. Estos ejemplos usan uno de los tipos de MSAL.js.

| Plataforma | Descripción | Vínculo |
| -------- | --------------------- | -------- |
| ![Esta imagen muestra el logotipo de JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core) | SPA llama a Microsoft Graph. |[javascript-graphapi-v2](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2) |
| ![Esta imagen muestra el logotipo de JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | SPA llama a Microsoft Graph mediante el flujo de código de autenticación con PKCE. |[javascript-v2](https://github.com/Azure-Samples/ms-identity-javascript-v2) |
| ![Esta imagen muestra el logotipo de JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core) | SPA llama a B2C. |[b2c-javascript-msal-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp) |
| ![Esta imagen muestra el logotipo de Angular](media/sample-v2-code/logo_angular.png) [JavaScript (MSAL-Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)| SPA llama a Microsoft Graph.  | [active-directory-javascript-singlepageapp-angular](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular) |
| ![Esta imagen muestra el logotipo de Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL-Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)| SPA llama a la API Web personalizada. | [ms-identity-javascript-angular-spa-aspnetcore-webapi](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnetcore-webapi) |
| ![Esta imagen muestra el logotipo de Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL-Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | SPA llama a B2C. |[active-directory-b2c-javascript-angular-spa](https://github.com/Azure-Samples/active-directory-b2c-javascript-angular-spa) |
| ![Esta imagen muestra el logotipo de React](media/sample-v2-code/logo_react.png) [React (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core)| SPA llama a su propia API web que, a su vez, llama a Microsoft Graph.  | [ms-identity-javascript-react-spa-dotnetcore-webapi-obo](https://github.com/Azure-Samples/ms-identity-javascript-react-spa-dotnetcore-webapi-obo) |
| ![Esta imagen muestra el logotipo de Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL-Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | SPA llama a la API Web personalizada con roles de aplicación y grupos de seguridad |[ms-identity-javascript-angular-spa-dotnetcore-webapi-roles-groups](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-dotnetcore-webapi-roles-groups) |

## <a name="web-applications"></a>Aplicaciones web

Los ejemplos siguientes ilustran las aplicaciones web que inician la sesión de los usuarios. Algunos ejemplos también muestran la aplicación que llama a Microsoft Graph o a su propia API web con la identidad del usuario.

| Plataforma | Únicamente inicio de sesión del usuario | Inicio de sesión del usuario y llamada a Microsoft Graph |
| -------- | ------------------- | --------------------------------- |
| ![Esta imagen muestra el logotipo de ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core | [ASP.NET Core WebApp signs-in users tutorial](https://aka.ms/aspnetcore-webapp-sign-in) (Tutorial de inicio de sesión para los usuarios de WebApp de ASP.NET Core) | Mismo ejemplo en la fase en la que la [aplicación web de ASP.NET Core llama a Microsoft Graph](https://aka.ms/aspnetcore-webapp-call-msgraph) |
| ![Esta imagen muestra el logotipo de ASP.NET](media/sample-v2-code/logo_NETframework.png)</p> ASP.NET | [Inicio rápido de ASP.NET](https://github.com/AzureAdQuickstarts/AppModelv2-WebApp-OpenIDConnect-DotNet) </p> [dotnet-webapp-openidconnect-v2](https://github.com/azure-samples/active-directory-dotnet-webapp-openidconnect-v2)  |  [dotnet-admin-restricted-scopes-v2](https://github.com/azure-samples/active-directory-dotnet-admin-restricted-scopes-v2) </p> |[msgraph-training-aspnetmvcapp](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp)
| ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png)  |                   | [ms-identity-java-webapp](https://github.com/Azure-Samples/ms-identity-java-webapp) |
| ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png)  |                   | [ms-identity-python-webapp](https://github.com/Azure-Samples/ms-identity-python-webapp) |
| ![Esta imagen muestra el logotipo de Ruby](media/sample-v2-code/logo_ruby.png) |                   | [msgraph-training-rubyrailsapp](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp) |

## <a name="desktop-and-mobile-public-client-apps"></a>Aplicaciones de escritorio y móviles de cliente público

Los siguientes ejemplos muestran aplicaciones cliente público (aplicaciones de escritorio o móviles) que tienen acceso a Microsoft Graph API o a una API web en nombre de un usuario. Además del ejemplo de *Escritorio (consola) con WAM*, todas estas aplicaciones cliente usan la Biblioteca de autenticación de Microsoft (MSAL).

| Aplicación cliente | Plataforma | Flujo y concesión | Llama a Microsoft Graph | Llama a una API web de ASP.NET Core. |
| ------------------ | -------- |  ----------| ---------- | ------------------------- |
| Escritorio (WPF)      | ![Esta imagen muestra el logotipo de .NET/C#](media/sample-v2-code/logo_NET.png) | [Código de autorización](msal-authentication-flows.md#authorization-code)| [dotnet-desktop-msgraph-v2](https://github.com/azure-samples/active-directory-dotnet-desktop-msgraph-v2) | [dotnet-native-aspnetcore-v2](https://aka.ms/msidentity-aspnetcore-webapi) |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de .NET/C# (escritorio)](media/sample-v2-code/logo_NET.png) | [Autenticación integrada de Windows](msal-authentication-flows.md#integrated-windows-authentication) | [dotnet-iwa-v2](https://github.com/azure-samples/active-directory-dotnet-iwa-v2) |  |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png) | [Autenticación integrada de Windows](msal-authentication-flows.md#integrated-windows-authentication) |[ms-identity-java-desktop](https://github.com/Azure-Samples/ms-identity-java-desktop/) |  |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de .NET/C# (escritorio)](media/sample-v2-code/logo_NETcore.png) | [Nombre de usuario/contraseña](msal-authentication-flows.md#usernamepassword) |[dotnetcore-up-v2](https://github.com/azure-samples/active-directory-dotnetcore-console-up-v2) |  |
| Escritorio (consola) con WAM  | ![Esta imagen muestra el logotipo de .NET/C# (escritorio)](media/sample-v2-code/logo_NETcore.png) | Interactivo con el [Administrador de cuentas web](/windows/uwp/security/web-account-manager) (WAM) |[dotnet-native-uwp-wam](https://github.com/azure-samples/active-directory-dotnet-native-uwp-wam) |  |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png) | [Nombre de usuario/contraseña](msal-authentication-flows.md#usernamepassword) |[ms-identity-java-desktop](https://github.com/Azure-Samples/ms-identity-java-desktop/) |  |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png) | [Nombre de usuario/contraseña](msal-authentication-flows.md#usernamepassword) |[ms-identity-python-desktop](https://github.com/Azure-Samples/ms-identity-python-desktop) |  |
| Mobile (Android, iOS, UWP)   | ![Esta imagen muestra el logotipo de .NET/C# (Xamarin)](media/sample-v2-code/logo_xamarin.png) | [Código de autorización](msal-authentication-flows.md#authorization-code) |[xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) |  |
| Mobile (iOS)       | ![Esta imagen muestra iOS/Objective-C o Swift](media/sample-v2-code/logo_iOS.png) | [Código de autorización](msal-authentication-flows.md#authorization-code) |[ios-swift-objc-native-v2](https://github.com/azure-samples/active-directory-ios-swift-native-v2) </p> [ios-native-nxoauth2-v2](https://github.com/azure-samples/active-directory-ios-native-nxoauth2-v2) |  |
| Escritorio (macOS)       | macOS | [Código de autorización](msal-authentication-flows.md#authorization-code) |[macOS-swift-objc-native-v2](https://github.com/Azure-Samples/ms-identity-macOS-swift-objc) |  |
| Móvil (Android-Java)   | ![Esta imagen muestra el logotipo de Android](media/sample-v2-code/logo_Android.png) | [Código de autorización](msal-authentication-flows.md#authorization-code) |  [android-Java](https://github.com/Azure-Samples/ms-identity-android-java) |  |
| Móvil (Android-Kotlin)   | ![Esta imagen muestra el logotipo de Android](media/sample-v2-code/logo_Android.png) | [Código de autorización](msal-authentication-flows.md#authorization-code) |  [android-Kotlin](https://github.com/Azure-Samples/ms-identity-android-kotlin) |  |

## <a name="daemon-applications"></a>Aplicaciones demonio

En el siguiente ejemplo se muestra una aplicación que accede a Microsoft Graph API con su propia identidad (sin ningún usuario).

| Aplicación cliente | Plataforma | Flujo y concesión | Llama a Microsoft Graph |
| ------------------ | -------- | ---------- | -------------------- |
| Consola | ![Esta imagen muestra el logotipo de .NET Core](media/sample-v2-code/logo_NETcore.png)</p> ASP.NET  | [Credenciales de cliente](msal-authentication-flows.md#client-credentials) | [dotnetcore-daemon-v2](https://github.com/azure-samples/active-directory-dotnetcore-daemon-v2) |
| Aplicación web | ![Esta imagen muestra el logotipo de ASP.NET](media/sample-v2-code/logo_NETframework.png)</p> ASP.NET  | [Credenciales de cliente](msal-authentication-flows.md#client-credentials) | [dotnet-daemon-v2](https://github.com/azure-samples/active-directory-dotnet-daemon-v2) |
| Consola | ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png) | [Credenciales de cliente](msal-authentication-flows.md#client-credentials) | [ms-identity-java-daemon](https://github.com/Azure-Samples/ms-identity-java-daemon) |
| Consola | ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png) | [Credenciales de cliente](msal-authentication-flows.md#client-credentials) | [ms-identity-python-daemon](https://github.com/Azure-Samples/ms-identity-python-daemon) |

## <a name="headless-applications"></a>Aplicaciones sin periféricos

El siguiente ejemplo muestra aplicaciones cliente públicas que se ejecutan en un dispositivo sin un explorador web. La aplicación puede ser una herramienta de línea de comandos, una aplicación que se ejecuta en Linux o Mac o una aplicación de IoT. El ejemplo incluye aplicaciones que acceden a Microsoft Graph API en nombre de un usuario que inicia sesión de manera interactiva en otro dispositivo (como un teléfono móvil). Esta aplicación cliente usa la Biblioteca de autenticación de Microsoft (MSAL).

| Aplicación cliente | Plataforma | Flujo y concesión | Llama a Microsoft Graph |
| ------------------ | -------- |  ----------| ---------- |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de .NET/C# (escritorio)](media/sample-v2-code/logo_NETcore.png) | [Flujo de código de dispositivo](msal-authentication-flows.md#device-code) |[dotnetcore-devicecodeflow-v2](https://github.com/azure-samples/active-directory-dotnetcore-devicecodeflow-v2) |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png) | [Flujo de código de dispositivo](msal-authentication-flows.md#device-code) |[ms-identity-java-devicecodeflow](https://github.com/Azure-Samples/ms-identity-java-devicecodeflow) |
| Escritorio (consola)   | ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png) | [Flujo de código de dispositivo](msal-authentication-flows.md#device-code) |[ms-identity-python-devicecodeflow](https://github.com/Azure-Samples/ms-identity-python-devicecodeflow) |

## <a name="multi-tenant-saas-applications"></a>Aplicaciones SaaS multiinquilino

En los siguientes ejemplos se muestra cómo configurar la aplicación para que acepte inicios de sesión de cualquier inquilino de Azure Active Directory (Azure AD). La configuración de la aplicación para que sea *multiinquilino* significa que puede ofrecer una aplicación de **software como servicio** (SaaS) a muchas organizaciones, lo que permite a sus usuarios iniciar sesión en la aplicación después de dar su consentimiento para usar su cuenta.

| Plataforma | Descripción | Vínculo |
| -------- | --------------------- | -------- |
| ![Esta imagen muestra el logotipo de JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser) | SPA multiinquilino llama a Graph API |[ms-identity-javascript-angular-spa-aspnet-webapi-multitenant](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnet-webapi-multitenant/tree/master/Chapter1) |
| ![Esta imagen muestra el logotipo de Angular](media/sample-v2-code/logo_angular.png) [Angular (MSAL-Angular)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular) | SPA multiinquilino llama a la API Web personalizada multiinquilino. |[ms-identity-javascript-angular-spa-aspnet-webapi-multitenant](https://github.com/Azure-Samples/ms-identity-javascript-angular-spa-aspnet-webapi-multitenant/tree/master/Chapter2) |
| ![Esta imagen muestra el logotipo de ASP.NET Core](media/sample-v2-code/logo_NETcore.png) [.NET Core (MSAL.NET)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) | La aplicación web ASP.NET Core MVC llama a Graph API |[active-directory-aspnetcore-webapp-openidconnect-v2](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-3-Multi-Tenant) |
| ![Esta imagen muestra el logotipo de ASP.NET Core](media/sample-v2-code/logo_NETcore.png) [.NET Core (MSAL.NET)](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) | La aplicación web ASP.NET Core MVC llama a la API web de ASP.NET Core |[active-directory-aspnetcore-webapp-openidconnect-v2](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/4-WebApp-your-API/4-3-AnyOrg) |

## <a name="web-apis"></a>API web

En los ejemplos siguientes se muestra cómo proteger una API web con el punto de conexión de la Plataforma de identidad de Microsoft y cómo llamar a una API de nivel inferior desde la API web.

| Plataforma | Muestra |
| -------- | ------------------- |
| ![Esta imagen muestra el logotipo de ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core | API web (servicio) de ASP.NET Core de [dotnet-native-aspnetcore-v2](https://aka.ms/msidentity-aspnetcore-webapi-calls-msgraph)  |
| ![Esta imagen muestra el logotipo de ASP.NET](media/sample-v2-code/logo_NET.png)</p>ASP.NET MVC | API web (servicio) de [ms-identity-aspnet-webapi-onbehalfof](https://github.com/Azure-Samples/ms-identity-aspnet-webapi-onbehalfof) |
| ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo_java.png) | Web API (servicio) de [ms-identity-java-webapi](https://github.com/Azure-Samples/ms-identity-java-webapi) |
| ![Esta imagen muestra el logotipo de Node.js](media/sample-v2-code/logo_nodejs.png) | API web (servicio) de [active-directory-javascript-nodejs-webapi-v2](https://github.com/Azure-Samples/active-directory-javascript-nodejs-webapi-v2) |
| ![Esta imagen muestra el logotipo de Node.js](media/sample-v2-code/logo_nodejs.png) | API web B2C (servicio) de [active-directory-b2c-javascript-nodejs-webapi](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi) |

## <a name="azure-functions-as-web-apis"></a>Azure Functions como API web

En los ejemplos siguientes se muestra cómo proteger una función de Azure mediante el desencadenador HTTP y exponer una API web con el punto de conexión de la plataforma de identidad de Microsoft; también se muestra cómo llamar a una API de nivel inferior desde la API web.

| Plataforma | Muestra |
| -------- | ------------------- |
| ![Esta imagen muestra el logotipo de ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core | Función de Azure de API web de ASP.NET Core (servicio) de [dotnet-native-aspnetcore-v2](https://github.com/Azure-Samples/ms-identity-dotnet-webapi-azurefunctions)  |
| ![Esta imagen muestra el logotipo de Node.js](media/sample-v2-code/logo_nodejs.png)</p>NodeJS | API web (servicio) de [NodeJS y passport-azure-ad](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-azurefunctions) |
| ![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo_python.png)</p>Python | API web API (servicio) de [Python](https://github.com/Azure-Samples/ms-identity-python-webapi-azurefunctions) |
| ![Esta imagen muestra el logotipo de Node.js](media/sample-v2-code/logo_nodejs.png)</p>NodeJS | API web (servicio) de [NodeJS y passport-azure-ad mediante behalf of](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-onbehalfof-azurefunctions) |

## <a name="other-microsoft-graph-samples"></a>Otros ejemplos de Microsoft Graph

Para ver [ejemplos](https://github.com/microsoftgraph/msgraph-community-samples/tree/master/samples#aspnet) y tutoriales que muestren los diferentes patrones de uso de Microsoft Graph API, incluida la autenticación con Azure AD, consulte [Microsoft Graph Community samples & tutorials](https://github.com/microsoftgraph/msgraph-community-samples) (Ejemplos y tutoriales de la comunidad de Microsoft Graph).

## <a name="see-also"></a>Consulte también

- [Introducción a Azure Active Directory para desarrolladores (v1.0)](../azuread-dev/v1-overview.md)
- [Conceptos y referencia de Microsoft Graph API](/graph/use-the-api?context=graph%2fapi%2fbeta&view=graph-rest-beta)