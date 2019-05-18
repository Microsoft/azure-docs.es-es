---
title: Bibliotecas de autenticación de plataforma de identidad de Microsoft | Microsoft Docs
description: Bibliotecas de cliente compatible y bibliotecas de middleware de servidor y biblioteca relacionado, origen y vínculos de ejemplos, para el extremo de plataforma de identidad de Microsoft.
services: active-directory
documentationcenter: ''
author: negoe
manager: CelesteDG
editor: ''
ms.assetid: 19cec615-e51f-4141-9f8c-aaf38ff9f746
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: negoe
ms.reviewer: jmprieur, saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 19718dc884685f6c1e1a588eea6a5de36e446fb1
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545533"
---
# <a name="microsoft-identity-platform-authentication-libraries"></a>Bibliotecas de autenticación de plataforma de identidad de Microsoft

El [extremos de plataforma de identidad de Microsoft](active-directory-v2-compare.md) admite los protocolos de OAuth 2.0 y OpenID Connect 1.0 estándar del sector. La biblioteca de autenticación de Microsoft (MSAL) está diseñado para trabajar con el punto de conexión de plataforma de identidad de Microsoft. También es posible usar las bibliotecas de código abierto que admiten OAuth 2.0 y OpenID Connect 1.0.

Se recomienda usar bibliotecas escritas por expertos en los dominios del protocolo que sigan una metodología de ciclo de vida de desarrollo de seguridad (SDL), como [a la que sigue Microsoft][Microsoft-SDL]. Si decide codificar manualmente los protocolos, siga una metodología como SDL de Microsoft y observe detenidamente las consideraciones de seguridad de las especificaciones de los estándares de cada protocolo.

> [!NOTE]
> ¿Busca la biblioteca de autenticación de Azure AD (ADAL)? Revise la [Guía de la biblioteca ADAL](active-directory-authentication-libraries.md).

## <a name="types-of-libraries"></a>Tipos de bibliotecas

Punto de conexión de plataforma de identidad de Microsoft funciona con dos tipos de bibliotecas:

* **Bibliotecas cliente**: Los servidores y clientes nativos utilizan bibliotecas de clientes para obtener tokens de acceso para llamar a un recurso, como Microsoft Graph.
* **Bibliotecas de middleware de servidores**: Las aplicaciones web usan bibliotecas de middleware de servidor para el inicio de sesión de usuario. Las API de web utilizan bibliotecas de middleware de servidor para validar los tokens que se envían mediante clientes nativos o mediante otros servidores.

## <a name="library-support"></a>Compatibilidad con bibliotecas

Las bibliotecas se dividen en dos categorías de soporte técnico:

* **Soporte técnico de Microsoft**: Microsoft proporciona soluciones para estas bibliotecas y ha hecho las diligencias necesarias con SDL para dichas bibliotecas.
* **Compatible**: Microsoft ha probado estas bibliotecas en escenarios básicos y ha confirmado que funcionan con el punto de conexión de plataforma de identidad de Microsoft. Microsoft no proporciona correcciones para estas bibliotecas y no ha realizado una revisión de estas bibliotecas. Los problemas y las solicitudes de características deben dirigirse al proyecto de código abierto de la biblioteca.

Para obtener una lista de bibliotecas que funcionan con el punto de conexión de plataforma de identidad de Microsoft, consulte las secciones siguientes de este artículo.

## <a name="microsoft-supported-client-libraries"></a>Bibliotecas de cliente compatibles con Microsoft

Las bibliotecas de autenticación de clientes se utilizan para adquirir un token para llamar a una API web protegida.

| Plataforma | Biblioteca | Descargar | Código fuente | Muestra | Referencia | Documentación conceptual | Roadmap |
| --- | --- | --- | --- | --- | --- | --- | ---|
| ![JavaScript](media/sample-v2-code/logo_js.png) | MSAL.js  | [NPM](https://www.npmjs.com/package/msal) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angularjs/README.md) |  [Aplicación de una sola página](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi-v2) | [Referencia](https://htmlpreview.github.io/? https://raw.githubusercontent.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-core/docs/classes/_useragentapplication_.useragentapplication.html) | [Docs conceptuales](msal-overview.md)| [Roadmap](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki#roadmap)
|![Angular JS](media/sample-v2-code/logo_angular.png) | MSAL Angular JS | [NPM](https://www.npmjs.com/package/@azure/msal-angularjs) | [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angularjs/README.md) |  |  | |
![Angular](media/sample-v2-code/logo_angular.png) | MSAL Angular (versión preliminar) | [NPM](https://www.npmjs.com/package/@azure/msal-angular) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-angular/README.md) | | | |
| ![.NET Framework](media/sample-v2-code/logo_NET.png) ![UWP](media/sample-v2-code/logo_windows.png) ![Xamarin](media/sample-v2-code/logo_xamarin.png) | MSAL .NET  |[NuGet](https://www.nuget.org/packages/Microsoft.Identity.Client) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet) | [Aplicación de escritorio](guidedsetups/active-directory-mobileanddesktopapp-windowsdesktop-intro.md) | [MSAL.NET](https://docs.microsoft.com/dotnet/api/microsoft.identity.client?view=azure-dotnet-preview) |[wiki](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki#conceptual-documentation) | [Roadmap](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki#roadmap)
| ![Python](media/sample-v2-code/logo_python.png) | MSAL Python (versión preliminar) | [PyPI](https://pypi.org/project/msal) | [Github](https://github.com/AzureAD/microsoft-authentication-library-for-python) | [Muestras](https://github.com/AzureAD/microsoft-authentication-library-for-python/tree/dev/sample) | [ReadTheDocs](https://msal-python.rtfd.io/) | [wiki](https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki) | [Roadmap](https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki/Roadmap)
| ![Java](media/sample-v2-code/logo_java.png) | Java MSAL (versión preliminar) | [Maven](https://mvnrepository.com/artifact/com.microsoft.azure/msal4j) | [Github](https://github.com/AzureAD/microsoft-authentication-library-for-java) | [Muestras](https://github.com/AzureAD/microsoft-authentication-library-for-java/tree/dev/src/samples) | | | [Roadmap](https://github.com/AzureAD/microsoft-authentication-library-for-java/wiki)
| ![iOS / Objective C o Swift](media/sample-v2-code/logo_iOS.png) | MSAL obj_c (versión preliminar) | [GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-objc) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-objc) | [Aplicación para iOS](https://github.com/Azure-Samples/active-directory-msal-ios-swift) |  |
|![Android / Java](media/sample-v2-code/logo_Android.png) | MSAL (versión preliminar) | [Repositorio central](https://repo1.maven.org/maven2/com/microsoft/identity/client/msal/) |[GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-android) | [Aplicación para Android](quickstart-v2-android.md) | [JavaDocs](https://javadoc.io/doc/com.microsoft.identity.client/msal) | | |

## <a name="microsoft-supported-server-middleware-libraries"></a>Bibliotecas de middleware de servidor compatibles de Microsoft

Las bibliotecas de middleware se usan para proteger las aplicaciones web y las API web. Para aplicaciones web o API web escritas con ASP.NET o ASP.NET Core, las bibliotecas de middleware las utilizan ASP.NET o ASP.NET Core

| Plataforma | Biblioteca | Descargar | Código fuente | Muestra | Referencia
| --- | --- | --- | --- | --- | --- |
| ![.NET](media/sample-v2-code/logo_NET.png) ![.NET Core](media/sample-v2-code/logo_NETcore.png) | Seguridad de ASP.NET |[NuGet](https://www.nuget.org/packages/Microsoft.AspNet.Mvc/) |[GitHub](https://github.com/aspnet/AspNetCore) |[Aplicación MVC](quickstart-v2-aspnet-webapp.md) |[Referencia de API de ASP.NET](https://docs.microsoft.com/dotnet/api/?view=aspnetcore-2.0) |
| ![.NET](media/sample-v2-code/logo_NET.png)| Extensiones de modelo de identidad para .NET| |[GitHub](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet) | [Aplicación MVC](quickstart-v2-aspnet-webapp.md) |[Referencia](https://docs.microsoft.com/dotnet/api/overview/azure/activedirectory/client?view=azure-dotnet) |
| ![Node.js](media/sample-v2-code/logo_nodejs.png) | Azure AD Passport |[NPM](https://www.npmjs.com/package/passport-azure-ad) |[GitHub](https://github.com/AzureAD/passport-azure-ad) | [Aplicación web](https://github.com/AzureADQuickStarts/AppModelv2-WebApp-OpenIDConnect-nodejs) | |

## <a name="compatible-client-libraries"></a>Bibliotecas de cliente compatibles

| Plataforma | Nombre de la biblioteca | Versión probada | Código fuente | Muestra |
|:---:|:---:|:---:|:---:|:---:|
|![JavaScript](media/sample-v2-code/logo_js.png)|[Hello.js](https://adodson.com/hello.js/) | 1.13.5 |[Hello.js](https://github.com/MrSwitch/hello.js) |[SPA](https://github.com/Azure-Samples/active-directory-javascript-graphapi-web-v2) |
| ![Java](media/sample-v2-code/logo_java.png) | [Scribe Java](https://github.com/scribejava/scribejava) | [Versión 3.2.0](https://github.com/scribejava/scribejava/releases/tag/scribejava-3.2.0) | [ScribeJava](https://github.com/scribejava/scribejava/) | |
| ![Java](media/sample-v2-code/logo_java.png) | [Biblioteca de Gluu OpenID Connect](https://github.com/GluuFederation/oxAuth) | [Versión 3.0.2](https://github.com/GluuFederation/oxAuth/releases/tag/3.0.2) | [Biblioteca de Gluu OpenID Connect](https://github.com/GluuFederation/oxAuth) | |
| ![Python](media/sample-v2-code/logo_python.png) | [Requests-OAuthlib](https://github.com/requests/requests-oauthlib) | [Versión 1.2.0](https://github.com/requests/requests-oauthlib/releases/tag/v1.2.0) | [Requests-OAuthlib](https://github.com/requests/requests-oauthlib) | |
| ![Node.js](media/sample-v2-code/logo_nodejs.png) | [openid-client](https://github.com/panva/node-openid-client) | [Versión 2.4.5](https://github.com/panva/node-openid-client/releases/tag/v2.4.5) | [openid-client](https://github.com/panva/node-openid-client) | |
| ![PHP](media/sample-v2-code/logo_php.png) | [PHP League oauth2-cliente](https://github.com/thephpleague/oauth2-client) | [Versión 1.4.2](https://github.com/thephpleague/oauth2-client/releases/tag/1.4.2) | [oauth2-client](https://github.com/thephpleague/oauth2-client/) | |
| ![Ruby](media/sample-v2-code/logo_ruby.png) |[OmniAuth](https://github.com/omniauth/omniauth/wiki) |omniauth:1.3.1<br />omniauth-oauth2:1.4.0 |[OmniAuth](https://github.com/omniauth/omniauth)<br />[OmniAuth OAuth2](https://github.com/intridea/omniauth-oauth2) |  |
| ![iOS](media/sample-v2-code/logo_iOS.png) ![Android](media/sample-v2-code/logo_Android.png) | [React Native App Auth](https://github.com/FormidableLabs/react-native-app-auth) | [Versión 4.2.0](https://github.com/FormidableLabs/react-native-app-auth/releases/tag/v4.2.0) | [React Native App Auth](https://github.com/FormidableLabs/react-native-app-auth) | |

Puede usar el punto de conexión de plataforma de identidad de Microsoft, para cualquier biblioteca que cumpla los estándares, por tanto, es importante saber dónde obtener soporte técnico.

* Para problemas y solicitudes de nuevas características en el código de la biblioteca, póngase en contacto con el propietario de la biblioteca.
* Para problemas y solicitudes de nuevas características de la implementación del protocolo en el servicio, póngase en contacto con Microsoft.
* [Archive una solicitud de característica](https://feedback.azure.com/forums/169401-azure-active-directory) para características adicionales que le gustaría ver en el protocolo.
* [Crear una solicitud de soporte técnico](https://docs.microsoft.com/azure/azure-supportability/how-to-create-azure-support-request) si encuentra un problema donde el punto de conexión de plataforma de identidad de Microsoft no es compatible con OAuth 2.0 u OpenID Connect 1.0.

## <a name="related-content"></a>Contenido relacionado

Para obtener más información sobre el punto de conexión de plataforma de identidad de Microsoft, consulte el [información general de plataforma de Microsoft identity][AAD-App-Model-V2-Overview].

<!--Image references-->

<!--Reference style links -->
[AAD-App-Model-V2-Overview]: v2-overview.md
[ClientLib-NET-Lib]: https://www.nuget.org/packages/Microsoft.Identity.Client
[ClientLib-NET-Repo]: https://github.com/AzureAD/microsoft-authentication-library-for-dotnet
[ClientLib-NET-Sample]: active-directory-v2-devquickstarts-wpf.md
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
[ServerLib-Net4-Owin-Oidc-Sample]: active-directory-v2-devquickstarts-dotnet-web.md
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
