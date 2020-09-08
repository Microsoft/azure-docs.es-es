---
title: Bibliotecas de autenticación de la Plataforma de identidad de Microsoft
description: Bibliotecas de cliente y de middleware de servidor compatibles, junto con vínculos de bibliotecas, orígenes y muestras relacionados, para el punto de conexión de la Plataforma de identidad de Microsoft.
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: reference
ms.workload: identity
ms.date: 07/25/2019
ms.author: negoe
ms.reviewer: jmprieur, saeeda
ms.custom: aaddev
ms.openlocfilehash: 58b31618ad1edd3eb9d71a56529b8618edf4e43e
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88688494"
---
# <a name="microsoft-identity-platform-authentication-libraries"></a>Bibliotecas de autenticación de la Plataforma de identidad de Microsoft

El [punto de conexión de la Plataforma de identidad de Microsoft](../azuread-dev/azure-ad-endpoint-comparison.md) admite los protocolos OAuth 2.0 y OpenID Connect 1.0 estándar del sector. Microsoft Authentication Library (MSAL) está diseñada para funcionar con el punto de conexión de la Plataforma de identidad de Microsoft. También puede usar las bibliotecas de código abierto que admiten OAuth 2.0 y OpenID Connect 1.0.

Se recomienda usar bibliotecas escritas por expertos en los dominios del protocolo que sigan una metodología de ciclo de vida de desarrollo de seguridad (SDL). Estas metodologías incluyen [la que sigue Microsoft][Microsoft-SDL]. Si codifica manualmente los protocolos, debe seguir una metodología como la de SDL de Microsoft. Preste mucha atención a las consideraciones de seguridad en las especificaciones de estándares para cada protocolo.

> [!NOTE]
> ¿Busca la Biblioteca de autenticación de Azure Active Directory (ADAL)? Revise la [Guía de la biblioteca ADAL](../azuread-dev/active-directory-authentication-libraries.md).

## <a name="types-of-libraries"></a>Tipos de bibliotecas

El punto de conexión de la Plataforma de identidad de Microsoft funciona con dos tipos de bibliotecas:

* **Bibliotecas cliente**: Los servidores y clientes nativos usan bibliotecas de clientes para adquirir tokens de acceso para llamar a un recurso, como Microsoft Graph.
* **Bibliotecas de middleware de servidores**: Las aplicaciones web usan bibliotecas de middleware de servidor para el inicio de sesión de usuario. Las API de web utilizan bibliotecas de middleware de servidor para validar los tokens que se envían mediante clientes nativos o mediante otros servidores.

## <a name="library-support"></a>Compatibilidad con bibliotecas

Las bibliotecas se dividen en dos categorías de soporte técnico:

* **Soporte técnico de Microsoft**: Microsoft proporciona soluciones para estas bibliotecas y ha hecho las diligencias necesarias con SDL para dichas bibliotecas.
* **Compatible**: Microsoft ha probado estas bibliotecas en escenarios básicos y ha confirmado que funcionan con el punto de conexión de la Plataforma de identidad de Microsoft. Microsoft no proporciona correcciones para estas bibliotecas y no ha realizado una revisión de estas bibliotecas. Los problemas y las solicitudes de características deben dirigirse al proyecto de código abierto de la biblioteca.

Para obtener una lista de bibliotecas que funcionan con el punto de conexión de la Plataforma de identidad de Microsoft, lea las secciones siguientes.

## <a name="microsoft-supported-client-libraries"></a>Bibliotecas de cliente compatibles con Microsoft

Use las bibliotecas de autenticación de clientes para adquirir un token para llamar a una API web protegida.

| Plataforma | Biblioteca | Descargar | Código fuente | Muestra | Referencia | Documentación conceptual | Plan de desarrollo |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ![JavaScript](media/sample-v2-code/logo_js.png) | MSAL.js  | [NPM](https://www.npmjs.com/package/msal) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/README.md) |  [Aplicación de una sola página](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2) | [Referencia](https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/) | [Documentación conceptual](msal-overview.md)| [Roadmap](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki#roadmap)
![Angular](media/sample-v2-code/logo_angular.png) | MSAL Angular | [NPM](https://www.npmjs.com/package/@azure/msal-angular) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/README.md) | [Angular SPA](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-angular) | [Referencia](https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-angular/) | [Documentación conceptual](msal-overview.md) | [Roadmap](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki#roadmap)
| ![.NET Framework](media/sample-v2-code/logo_NET.png) ![UWP](media/sample-v2-code/logo_windows.png) ![Xamarin](media/sample-v2-code/logo_xamarin.png) | MSAL.NET  |[NuGet](https://www.nuget.org/packages/Microsoft.Identity.Client) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) | [Aplicación de escritorio](/windows/apps/desktop/) | [MSAL.NET](/dotnet/api/microsoft.identity.client?view=azure-dotnet-preview) |[Documentación conceptual](msal-overview.md) | [Roadmap](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki#roadmap)
| ![Python](media/sample-v2-code/logo_python.png) | Python de MSAL | [PyPI](https://pypi.org/project/msal) | [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-python) | [Muestras](https://github.com/AzureAD/microsoft-authentication-library-for-python/tree/dev/sample) | [ReadTheDocs](https://msal-python.rtfd.io/) | [Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki) | [Roadmap](https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki/Roadmap)
| ![Java](media/sample-v2-code/logo_java.png) | Java de MSAL | [Maven](https://mvnrepository.com/artifact/com.microsoft.azure/msal4j) | [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-java) | [Muestras](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples) | [Referencia](https://javadoc.io/doc/com.microsoft.azure/msal4j/latest/index.html) | [Wiki](https://github.com/AzureAD/microsoft-authentication-library-for-java/wiki) | [Roadmap](https://github.com/AzureAD/microsoft-authentication-library-for-java/wiki)
| iOS y macOS | MSAL para iOS y macOS | [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-objc) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-objc) | [Aplicación iOS](https://github.com/Azure-Samples/ms-identity-mobile-apple-swift-objc), [Aplicación macOS](https://github.com/Azure-Samples/ms-identity-macOS-swift-objc) | [Referencia](https://azuread.github.io/microsoft-authentication-library-for-objc/index.html)  | [Documentación conceptual](msal-overview.md) | |
|![Android / Java](media/sample-v2-code/logo_Android.png) | MSAL de Android | [Central repository](https://repo1.maven.org/maven2/com/microsoft/identity/client/msal/) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-android) | [Aplicación para Android](quickstart-v2-android.md) | [JavaDocs](https://javadoc.io/doc/com.microsoft.identity.client/msal) | [Documentación conceptual](msal-overview.md) |[Roadmap](https://github.com/AzureAD/microsoft-authentication-library-for-android/wiki/Roadmap)

## <a name="microsoft-supported-server-middleware-libraries"></a>Bibliotecas de middleware de servidor compatibles de Microsoft

Use las bibliotecas de middleware para ayudar a proteger las aplicaciones web y las API web. Las aplicaciones web o las API web escritas con ASP.NET o ASP.NET Core usan las bibliotecas de middleware.

| Plataforma | Biblioteca | Descargar | Código fuente | Muestra | Referencia
| --- | --- | --- | --- | --- | --- |
| ![.NET](media/sample-v2-code/logo_NET.png) ![.NET Core](media/sample-v2-code/logo_NETcore.png) | Seguridad de ASP.NET |[NuGet](https://www.nuget.org/packages/Microsoft.AspNet.Mvc/) |[GitHub](https://github.com/aspnet/AspNetCore) |[Aplicación MVC](quickstart-v2-aspnet-webapp.md) |[Referencia de API de ASP.NET](/dotnet/api/?view=aspnetcore-2.0) |
| ![.NET](media/sample-v2-code/logo_NET.png)| Extensiones de modelo de identidad para .NET| |[GitHub](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet) | [Aplicación MVC](quickstart-v2-aspnet-webapp.md) |[Referencia](/dotnet/api/overview/azure/activedirectory/client?view=azure-dotnet) |
| ![Node.js](media/sample-v2-code/logo_nodejs.png) | Azure AD Passport |[NPM](https://www.npmjs.com/package/passport-azure-ad) |[GitHub](https://github.com/AzureAD/passport-azure-ad) | [Aplicación web](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-nodejs) | |

## <a name="microsoft-supported-libraries-by-os--language"></a>Bibliotecas compatibles con Microsoft por sistema operativo y lenguaje

En términos de compatibilidad de los sistemas operativos frente a los lenguajes, la asignación es la siguiente:

| Plataforma    | Windows    | Linux      | macOS      | iOS | Android    |
|-------------|------------|------------|------------|------------|------------|
| ![JavaScript](media/sample-v2-code/logo_js.png)  |  MSAL.js | MSAL.js | MSAL.js | MSAL.js |  MSAL.js |
| <img alt="C#" src="../../cognitive-services/speech-service/media/index/logo_csharp.svg" width="64px" height="64px" /> | ASP.NET, ASP.NET Core, MSAL.Net (.NET FW,  Core, UWP)| ASP.NET Core, MSAL.Net (.NET Core) | ASP.NET Core, MSAL.Net (macOS)       | MSAL.Net (Xamarin.iOS) | MSAL.Net (Xamarin.Android)|
| Swift <br> Objective-C |            |            | [MSAL para iOS y macOS](msal-overview.md) | [MSAL para iOS y macOS](msal-overview.md) |            |
| ![Java](media/sample-v2-code/logo_java.png) Java | msal4j | msal4j | msal4j | | MSAL de Android |
| ![Python](media/sample-v2-code/logo_python.png) Python | Python de MSAL | Python de MSAL | Python de MSAL |
| ![Node.Js](media/sample-v2-code/logo_nodejs.png) Node.JS | Passport.node | Passport.node | Passport.node |

Consulte también [Escenarios por plataformas y lenguajes compatibles](authentication-flows-app-scenarios.md#scenarios-and-supported-platforms-and-languages)

## <a name="compatible-client-libraries"></a>Bibliotecas de cliente compatibles

| Plataforma | Nombre de la biblioteca | Versión probada | Código fuente | Muestra |
|:---:|:---:|:---:|:---:|:---:|
|![JavaScript](media/sample-v2-code/logo_js.png)|[Hello.js](https://adodson.com/hello.js/) | Versión 1.13.5 |[Hello.js](https://github.com/MrSwitch/hello.js) |[SPA](https://github.com/Azure-Samples/active-directory-javascript-graphapi-v2) |
|![Vue](media/sample-v2-code/logo_vue.png)|[MSAL para Vue](https://github.com/mvertopoulos/vue-msal) | Versión 3.0.3 |[vue-msal](https://github.com/mvertopoulos/vue-msal) | |
| ![Java](media/sample-v2-code/logo_java.png) | [Scribe Java](https://github.com/scribejava/scribejava) | [Versión 3.2.0](https://github.com/scribejava/scribejava/releases/tag/scribejava-3.2.0) | [ScribeJava](https://github.com/scribejava/scribejava/) | |
| ![Java](media/sample-v2-code/logo_java.png) | [Biblioteca de Gluu OpenID Connect](https://github.com/GluuFederation/oxAuth) | [Versión 3.0.2](https://github.com/GluuFederation/oxAuth/releases/tag/3.0.2) | [Biblioteca de Gluu OpenID Connect](https://github.com/GluuFederation/oxAuth) | |
| ![Python](media/sample-v2-code/logo_python.png) | [Requests-OAuthlib](https://github.com/requests/requests-oauthlib) | [Versión 1.2.0](https://github.com/requests/requests-oauthlib/releases/tag/v1.2.0) | [Requests-OAuthlib](https://github.com/requests/requests-oauthlib) | |
| ![Node.js](media/sample-v2-code/logo_nodejs.png) | [openid-client](https://github.com/panva/node-openid-client) | [Versión 2.4.5](https://github.com/panva/node-openid-client/releases/tag/v2.4.5) | [openid-client](https://github.com/panva/node-openid-client) | |
| ![PHP](media/sample-v2-code/logo_php.png) | [PHP League oauth2-cliente](https://github.com/thephpleague/oauth2-client) | [Versión 1.4.2](https://github.com/thephpleague/oauth2-client/releases/tag/1.4.2) | [oauth2-client](https://github.com/thephpleague/oauth2-client/) | |
| ![Ruby](media/sample-v2-code/logo_ruby.png) |[OmniAuth](https://github.com/omniauth/omniauth/wiki) |omniauth: 1.3.1<br />omniauth-oauth2: 1.4.0 |[OmniAuth](https://github.com/omniauth/omniauth)<br />[OmniAuth OAuth2](https://github.com/intridea/omniauth-oauth2) |  |
| iOS, macOS, y Android  | [React Native App Auth](https://github.com/FormidableLabs/react-native-app-auth) | [Versión 4.2.0](https://github.com/FormidableLabs/react-native-app-auth/releases/tag/v4.2.0) | [React Native App Auth](https://github.com/FormidableLabs/react-native-app-auth) | |

Para cualquier biblioteca que cumpla con los estándares, puede usar el punto de conexión de la Plataforma de identidad de Microsoft. Es importante saber adónde acudir para obtener soporte técnico:

* Para problemas y solicitudes de nuevas características en el código de la biblioteca, póngase en contacto con el propietario de la biblioteca.
* Para problemas y solicitudes de nuevas características de la implementación del protocolo en el servicio, póngase en contacto con Microsoft.
* [Archive una solicitud de característica](https://feedback.azure.com/forums/169401-azure-active-directory) para características adicionales que le gustaría ver en el protocolo.
* [Cree una solicitud de soporte técnico](../../azure-portal/supportability/how-to-create-azure-support-request.md) si encuentra un problema en el que el punto de conexión de la Plataforma de identidad de Microsoft no es compatible con OAuth 2.0 u OpenID Connect 1.0.

## <a name="related-content"></a>Contenido relacionado

Para más información sobre el punto de conexión de la Plataforma de identidad de Microsoft, consulte la [Introducción a la Plataforma de identidad de Microsoft][AAD-App-Model-V2-Overview].

<!--Image references-->

<!--Reference style links -->
[AAD-App-Model-V2-Overview]: v2-overview.md
[ClientLib-NET-Lib]: https://www.nuget.org/packages/Microsoft.Identity.Client
[ClientLib-NET-Repo]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet
[ClientLib-NET-Sample]: ./tutorial-v2-windows-desktop.md
[ClientLib-Node-Lib]: https://www.npmjs.com/package/passport-azure-ad
[ClientLib-Node-Repo]: https://github.com/AzureAD/passport-azure-ad
[ClientLib-Node-Sample]:/
[ClientLib-Iosmac-Lib]:/
[ClientLib-Iosmac-Repo]:/
[ClientLib-Iosmac-Sample]:/
[ClientLib-Android-Lib]:/
[ClientLib-Android-Repo]:/
[ClientLib-Android-Sample]:/
[ClientLib-Js-Lib]:/
[ClientLib-Js-Repo]:/
[ClientLib-Js-Sample]:/

[Microsoft-SDL]: https://www.microsoft.com/sdl/default.aspx
[ServerLib-Net4-Owin-Oidc-Lib]: https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect/
[ServerLib-Net4-Owin-Oidc-Repo]: https://katanaproject.codeplex.com/
[ServerLib-Net4-Owin-Oidc-Sample]: ./tutorial-v2-asp-webapp.md
[ServerLib-Net4-Owin-Oauth-Lib]: https://www.nuget.org/packages/Microsoft.Owin.Security.OAuth/
[ServerLib-Net4-Owin-Oauth-Repo]: https://katanaproject.codeplex.com/
[ServerLib-Net4-Owin-Oauth-Sample]: https://azure.microsoft.com/documentation/articles/active-directory-v2-devquickstarts-dotnet-api/
[ServerLib-Net-Jwt-Lib]: https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt
[ServerLib-Net-Jwt-Repo]: https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet
[ServerLib-Net-Jwt-Sample]:/
[ServerLib-NetCore-Owin-Oidc-Lib]: https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.OpenIdConnect/
[ServerLib-NetCore-Owin-Oidc-Repo]: https://github.com/aspnet/Security
[ServerLib-NetCore-Owin-Oidc-Sample]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-aspnetcore-v2
[ServerLib-NetCore-Owin-Oauth-Lib]: https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.OAuth/
[ServerLib-NetCore-Owin-Oauth-Repo]: https://github.com/aspnet/Security
[ServerLib-NetCore-Owin-Oauth-Sample]:/
[ServerLib-Node-Lib]: https://www.npmjs.com/package/passport-azure-ad
[ServerLib-Node-Repo]: https://github.com/AzureAD/passport-azure-ad/
[ServerLib-Node-Sample]: https://azure.microsoft.com/documentation/articles/active-directory-v2-devquickstarts-node-web/