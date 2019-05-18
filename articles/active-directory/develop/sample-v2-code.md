---
title: Ejemplos de código de plataforma de identidad de Microsoft | Microsoft Docs
description: Proporciona un índice de la identidad de Microsoft disponible ejemplos de código de plataforma (punto de conexión V2), organizados por escenario.
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: a242a5ff-7300-40c2-ba83-fb6035707433
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 10/26/2018
ms.author: ryanwi
ms.reviewer: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: f430d712626c5c5f4667d709a27171592f84a150
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65823606"
---
# <a name="microsoft-identity-platform-code-samples-v20-endpoint"></a>Ejemplos de código de plataforma de identidad de Microsoft (punto de conexión v2.0)

[!INCLUDE [active-directory-develop-applies-v2-msal](../../../includes/active-directory-develop-applies-v2-msal.md)]

Puede usar la plataforma de identidad de Microsoft para:

- Agregar autenticación y autorización a sus aplicaciones web y API web.
- Requerir un token de acceso para tener acceso a una API web protegida.

En este artículo se describe brevemente y se proporciona vínculos a ejemplos para el punto de conexión de plataforma de identidad de Microsoft. Estos ejemplos le muestran cómo se hace, junto con fragmentos de código que puede usar en sus aplicaciones. En la página de ejemplos de código, encontrará temas Léame detallados que le ayudarán con los requisitos, la instalación y la configuración. Dispone de comentarios dentro del código para ayudarle a comprender las secciones críticas.

> [!NOTE]
> Si está interesado en los ejemplos de v1.0, consulte los [ejemplos de código de Azure AD (punto de conexión v1.0)](sample-v1-code.md).

Para entender el escenario básico para cada tipo de ejemplo, vea [tipos de aplicación para el punto de conexión de plataforma de identidad de Microsoft](v2-app-types.md).

También puede contribuir a los ejemplos en GitHub. Para saber cómo, vea [Ejemplos y documentación de Microsoft Azure Active Directory](https://github.com/Azure-Samples?page=3&query=active-directory).

## <a name="single-page-applications-spa"></a>Aplicación de página única (SPA)

Estos ejemplos muestran cómo escribir una aplicación de una página protegida con la plataforma Microsoft identity. Estos ejemplos usan uno de los tipos de MSAL.js:

| Plataforma | DESCRIPCIÓN | Vincular |
| -------- | --------------------- | -------- |
| ![JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (msal.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core) | Llama a Microsoft Graph |[javascript-graphapi-web-v2](https://github.com/Azure-Samples/active-directory-javascript-graphapi-web-v2) |
| ![JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (msal.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core) | Las llamadas a las llamadas a B2C |[b2c-javascript-msal-singlepageapp](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp) |
| ![JavaScript](media/sample-v2-code/logo_js.png) [JavaScript (msal.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core) | Web propio de las llamadas API |[javascript-singlepageapp-dotnet-webapi-v2](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2) |
| ![Angular JS](media/sample-v2-code/logo_angular.png) [JavaScript (AngularJS MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angularjs)| Llama a Microsoft Graph  | [MsalAngularjsDemoApp](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-angularjs/samples/MsalAngularjsDemoApp)
| ![Angular](media/sample-v2-code/logo_angular.png) [JavaScript (Angular MSAL)](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular)| Llama a Microsoft Graph  | [MSALAngularDemoApp](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-angular/samples/MSALAngularDemoApp) |

## <a name="web-applications"></a>Aplicaciones web

Los ejemplos siguientes ilustran las aplicaciones web que inician la sesión de los usuarios. Algunos ejemplos también muestran la aplicación que llama a Microsoft Graph o a su propia API web con la identidad del usuario.

| Plataforma | Únicamente inicio de sesión del usuario | Inicio de sesión del usuario y llamada a Microsoft Graph |
| -------- | ------------------- | --------------------------------- |
| ![ASP.NET Core](media/sample-v2-code/logo_NETcore.png)</p>ASP.NET Core 2.1 | [Tutorial ASP.NET Core WebApp inicia sesión a los usuarios](https://aka.ms/aspnetcore-webapp-sign-in) | Mismo ejemplo en el [aplicación Web ASP.NET Core llama a Microsoft Graph](https://aka.ms/aspnetcore-webapp-call-msgraph) fase |
| ![ASP.NET](media/sample-v2-code/logo_NETframework.png)</p> ASP.NET | [Inicio rápido de ASP.NET](https://github.com/AzureAdQuickstarts/AppModelv2-WebApp-OpenIDConnect-DotNet) </p> [dotnet-webapp-openidconnect-v2](https://github.com/azure-samples/active-directory-dotnet-webapp-openidconnect-v2)  |  [dotnet-admin-restricted-scopes-v2](https://github.com/azure-samples/active-directory-dotnet-admin-restricted-scopes-v2) </p> |[msgraph-training-aspnetmvcapp](https://github.com/microsoftgraph/msgraph-training-aspnetmvcapp)
| ![Node.js](media/sample-v2-code/logo_nodejs.png)  |                   | [Inicio rápido de Node.js](https://github.com/azureadquickstarts/appmodelv2-webapp-openidconnect-nodejs) |
| ![Ruby](media/sample-v2-code/logo_ruby.png) |                   | [msgraph-training-rubyrailsapp](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp) |

## <a name="desktop-and-mobile-public-client-apps"></a>Aplicaciones de escritorio y móviles de cliente público

Los siguientes ejemplos muestran aplicaciones cliente público (aplicaciones de escritorio o móviles) que tienen acceso a Microsoft Graph API o a una API web en nombre de un usuario. Todas estas aplicaciones cliente usan bibliotecas de autenticación de Microsoft (MSAL).

| Aplicación cliente | Plataforma | Flujo y concesión | Llama a Microsoft Graph | Llamada a una API web de ASP.NET Core 2.0 |
| ------------------ | -------- |  ----------| ---------- | ------------------------- |
| Escritorio (WPF)      | ![.NET/C#](media/sample-v2-code/logo_NET.png) | [interactive](msal-authentication-flows.md#interactive)| [dotnet-desktop-msgraph-v2](https://github.com/azure-samples/active-directory-dotnet-desktop-msgraph-v2) | [dotnet-native-aspnetcore-v2](https://aka.ms/msidentity-aspnetcore-webapi) |
| Escritorio (consola)   | ![.NET/C# (escritorio)](media/sample-v2-code/logo_NET.png) | [Autenticación integrada de Windows](msal-authentication-flows.md#integrated-windows-authentication) | [dotnet-iwa-v2](https://github.com/azure-samples/active-directory-dotnet-iwa-v2) |  |
| Escritorio (consola)   | ![.NET/C# (escritorio)](media/sample-v2-code/logo_NETcore.png) | [Nombre de usuario/contraseña](msal-authentication-flows.md#usernamepassword) |[dotnetcore-up-v2](https://github.com/azure-samples/active-directory-dotnetcore-console-up-v2) |  |
| Mobile (Android, iOS, UWP)   | ![.NET/C# (Xamarin)](media/sample-v2-code/logo_xamarin.png) | [interactive](msal-authentication-flows.md#interactive) |[xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) |  |
| Mobile (iOS)       | ![iOS / Objective C o Swift](media/sample-v2-code/logo_iOS.png) | [interactive](msal-authentication-flows.md#interactive) |[ios-swift-native-v2](https://github.com/azure-samples/active-directory-ios-swift-native-v2) </p> [ios-native-nxoauth2-v2](https://github.com/azure-samples/active-directory-ios-native-nxoauth2-v2) |  |
| Móvil (Android)   | ![Android / Java](media/sample-v2-code/logo_Android.png) | [interactive](msal-authentication-flows.md#interactive) |  [android-native-v2](https://github.com/azure-samples/active-directory-android-native-v2 ) |  |

## <a name="daemon-applications"></a>Aplicaciones demonio

En el siguiente ejemplo se muestra una aplicación que accede a Microsoft Graph API con su propia identidad (sin ningún usuario).

| Aplicación cliente | Plataforma | Flujo y concesión | Llama a Microsoft Graph |
| ------------------ | -------- | ---------- | -------------------- |
| Consola | ![.NET Core](media/sample-v2-code/logo_NETcore.png)</p> ASP.NET  | [Credenciales del cliente](msal-authentication-flows.md#client-credentials) | [dotnetcore-daemon-v2](https://github.com/azure-samples/active-directory-dotnetcore-daemon-v2) |
| Aplicación web | ![ASP.NET](media/sample-v2-code/logo_NETframework.png)</p> ASP.NET  | [Credenciales del cliente](msal-authentication-flows.md#client-credentials) | [dotnet-daemon-v2](https://github.com/azure-samples/active-directory-dotnet-daemon-v2) |

## <a name="headless-applications"></a>Aplicaciones sin periféricos

El siguiente ejemplo muestra aplicaciones cliente públicas que se ejecutan en un dispositivo sin un explorador web. La aplicación puede ser una herramienta de línea de comandos, se puede ejecutar en Linux/Mac o puede ser una aplicación de IoT. El ejemplo incluye aplicaciones que acceden a Microsoft Graph API en nombre de un usuario que inicia sesión de manera interactiva en otro dispositivo (como un teléfono móvil). Esta aplicación cliente usa bibliotecas de autenticación de Microsoft (MSAL).

| Aplicación cliente | Plataforma | Flujo y concesión | Llama a Microsoft Graph |
| ------------------ | -------- |  ----------| ---------- |
| Escritorio (consola)   | ![.NET/C# (escritorio)](media/sample-v2-code/logo_NETcore.png) | [Flujo de código de dispositivo](msal-authentication-flows.md#device-code) |[dotnetcore-devicecodeflow-v2](https://github.com/azure-samples/active-directory-dotnetcore-devicecodeflow-v2) |

## <a name="web-apis"></a>API web

El ejemplo siguiente muestra cómo proteger una API web con el punto de conexión de plataforma de identidad de Microsoft. Esta API la ejecuta una aplicación de WPF, pero podría llamarla cualquier aplicación. La API web también llama a Microsoft Graph.

| Plataforma | Muestra |
| -------- | ------------------- |
| ![.NET/C#](media/sample-v2-code/logo_NET.png) | WebAPI (servicio) de [dotnet-native-aspnetcore-v2](https://aka.ms/msidentity-aspnetcore-webapi-calls-msgraph) |

## <a name="other-microsoft-graph-samples"></a>Otros ejemplos de Microsoft Graph

Para ver [ejemplos](https://github.com/microsoftgraph/msgraph-community-samples/tree/master/samples#aspnet) y tutoriales que muestren los diferentes patrones de uso de Microsoft Graph API, incluida la autenticación con Azure AD, consulte [Microsoft Graph Community samples & tutorials](https://github.com/microsoftgraph/msgraph-community-samples) (Ejemplos y tutoriales de la comunidad de Microsoft Graph).

## <a name="see-also"></a>Vea también

- [Guía del desarrollador de Azure Active Directory (v1.0)](v1-overview.md)
- [Conceptos y referencia de Graph API de Azure AD](https://msdn.microsoft.com/library/azure/hh974476.aspx)
- [Biblioteca auxiliar de Graph API de Azure AD](https://www.nuget.org/packages/Microsoft.Azure.ActiveDirectory.GraphClient)
