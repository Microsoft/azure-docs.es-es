---
title: Ejemplos de código de Azure Active Directory v1.0 | Microsoft Docs
description: Se proporciona un índice de ejemplos de código de Azure Active Directory (punto de conexión v1.0), organizados por escenario.
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.topic: sample
ms.workload: identity
ms.date: 07/15/2019
ms.author: ryanwi
ms.reviewer: jmprieur
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: 1efae6563b88f82318f9b8358d255b234c6cb7fd
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88116213"
---
# <a name="azure-active-directory-code-samples-v10-endpoint"></a>Ejemplos de código de Azure Active Directory (punto de conexión v1.0)

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

Puede usar Microsoft Azure Active Directory (Azure AD) para agregar autenticación y autorización a sus aplicaciones web y API web.

En esta sección se proporcionan vínculos a ejemplos que puede usar para aprender más sobre el punto de conexión de Azure AD v1.0. En estos ejemplos se muestra cómo se hace, junto con fragmentos de código que puede usar en sus aplicaciones. En la página de ejemplos de código, encontrará temas Léame detallados que le ayudarán con los requisitos, la instalación y la configuración. Y se comenta el código para ayudarle a comprender las secciones críticas.

> [!NOTE]
> Si está interesado en ejemplos de código de Azure AD V2, consulte [ejemplos de código de v2.0 por escenario](../develop/sample-v2-code.md?toc=/azure/active-directory/azuread-dev/toc.json&bc=/azure/active-directory/azuread-dev/breadcrumb/toc.json).

Para comprender los escenarios básicos para cada tipo de ejemplo, consulte [Escenarios de autenticación para Azure AD](v1-authentication-scenarios.md).

También puede contribuir a nuestros ejemplos en GitHub. Para saber cómo, vea [Ejemplos y documentación de Microsoft Azure Active Directory](https://github.com/Azure-Samples?page=3&query=active-directory).

## <a name="single-page-applications"></a>Aplicación de página única

En este ejemplo se muestra cómo escribir una aplicación de página única protegida con Azure AD.

 Plataforma | Llama a su propia API | Llama a otra API web
 -------- |  --------------------- | ------------------ 
![Esta imagen muestra el logotipo de JavaScript](media/sample-v2-code/logo-js.png) | [javascript-singlepageapp](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi) |
![Esta imagen muestra el logotipo de Angular JS](media/sample-v2-code/logo-angular.png) | [angularjs-singlepageapp](https://github.com/Azure-Samples/active-directory-angularjs-singlepageapp) | [angularjs-singlepageapp-cors](https://github.com/Azure-Samples/active-directory-angularjs-singlepageapp-dotnet-webapi)

## <a name="web-applications"></a>Aplicaciones web

### <a name="web-applications-signing-in-users-calling-microsoft-graph-or-a-web-api-with-the-users-identity"></a>Aplicaciones Web que inician sesión por los usuarios, llaman a Microsoft Graph o una API web con la identidad del usuario

En los ejemplos siguientes se ilustra la firma de aplicaciones web por los usuarios. Algunas de estas aplicaciones también llaman a Microsoft Graph o a su propia API web, en nombre del usuario con sesión iniciada.

 Plataforma | Únicamente inicio de sesión del usuario | Llama a Microsoft Graph | Llama a otra API web de ASP.NET o de ASP.NET Core 2.0
 -------- | ------------------- | --------------------- | -------------------------
![Esta imagen muestra el logotipo de ASP.NET](media/sample-v2-code/logo-netcore.png)</p>ASP.NET Core 2.0 | [dotnet-webapp-openidconnect-aspnetcore](https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-aspnetcore) | [webapp-webapi-multitenant-openidconnect-aspnetcore](https://github.com/Azure-Samples/active-directory-webapp-webapi-multitenant-openidconnect-aspnetcore/) </p>(AAD Graph) | [dotnet-webapp-webapi-openidconnect-aspnetcore](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-openidconnect-aspnetcore)
![Esta imagen muestra el logotipo de ASP.NET](media/sample-v2-code/logo-netframework.png)</p> ASP.NET 4.5 |  </p> [webapp-WSFederation-dotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation) </p> [dotnet-webapp-webapi-oauth2-useridentity](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-oauth2-useridentity) | [dotnet-webapp-multitenant-openidconnect](https://github.com/Azure-Samples/active-directory-dotnet-webapp-multitenant-openidconnect)</p> (AAD Graph) |
![Esta imagen muestra el logotipo de Python](media/sample-v2-code/logo-python.png) | | [python-webapp-graphapi](https://github.com/Azure-Samples/active-directory-python-webapp-graphapi)  |
![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo-java.png)  | | [java-webapp-openidconnect](https://github.com/azure-samples/active-directory-java-webapp-openidconnect)  |
![Esta imagen muestra el logotipo de PHP](media/sample-v2-code/logo-php.png) | | [php-graphapi-web](https://github.com/Azure-Samples/active-directory-php-graphapi-web)  |

### <a name="web-applications-demonstrating-role-based-access-control-authorization"></a>Aplicaciones web que muestran el control de acceso basado en roles (autorización)

En los ejemplos siguientes se muestra cómo implementar el control de acceso basado en rol (RBAC). RBAC se usa para restringir los permisos de determinadas características de una aplicación web a determinados usuarios. Se otorga autorización a los usuarios en función de si pertenecen a un **grupo de Azure AD** o tienen un determinado **rol** de aplicación.

Plataforma | Muestra |
 -------- | ------------------- |
![Esta imagen muestra el logotipo de ASP.NET](media/sample-v2-code/logo-netframework.png)</p> ASP.NET 4.5 | [dotnet-webapp-groupclaims](https://github.com/Azure-Samples/active-directory-dotnet-webapp-groupclaims) </p>  [dotnet-webapp-roleclaims](https://github.com/Azure-Samples/active-directory-dotnet-webapp-roleclaims) | Una aplicación web de .NET 4.5 MVC que usa **roles** de Azure AD para la autorización

## <a name="desktop-and-mobile-public-client-applications-calling-microsoft-graph-or-a-web-api"></a>Aplicaciones de escritorio y móviles de cliente público que llaman a Microsoft Graph o a una API web

En los siguientes ejemplos se muestran aplicaciones cliente públicas (aplicaciones de escritorio o móviles) que acceden a Microsoft Graph o a una API web en nombre de un usuario. Según los dispositivos y plataformas, las aplicaciones pueden iniciar la sesión de los usuarios de diferentes maneras (flujos o concesiones):

- interactivamente,
- en modo silencioso (con autenticación integrada de Windows en Windows, o nombre de usuario y contraseña),
- o mediante la delegación del inicio de sesión interactivo en otro dispositivo (flujo de código de dispositivo usado en dispositivos que no proporcionan controles web).

Aplicación cliente | Plataforma | Flujo y concesión | Llama a Microsoft Graph | Llama a una API web de ASP.NET o ASP.NET Core 2.x
------------------ | -------- | ---------- | -------------------- | -------------------------
Escritorio (WPF)           | ![Esta imagen muestra el logotipo de .NET/C#](media/sample-v2-code/logo-net.png)  | Interactive | Parte de [dotnet-native-multitarget](https://github.com/azure-samples/active-directory-dotnet-native-multitarget) | [Dotnet-native-desktop](https://github.com/Azure-Samples/active-directory-dotnet-native-desktop) </p> [dotnet-native-aspnetcore](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore/)</p> [dotnet-webapi-manual-jwt-validation](https://github.com/azure-samples/active-directory-dotnet-webapi-manual-jwt-validation)
Móvil (UWP)            | .![Esta imagen muestra el logotipo de .NET/C#/UWP](media/sample-v2-code/logo-windows.png)   | Interactive | [dotnet-native-uwp-wam](https://github.com/azure-samples/active-directory-dotnet-native-uwp-wam) </p> En este ejemplo se usa [WAM](/windows/uwp/security/web-account-manager), no [ADAL.NET](https://aka.ms/adalnet).|  [dotnet-windows-store](https://github.com/Azure-Samples/active-directory-dotnet-windows-store) (aplicación de UWP que usa ADAL.NET para llamar a una API web de un único inquilino) </p> [dotnet-webapi-multitenant-windows-store](https://github.com/Azure-Samples/active-directory-dotnet-webapi-multitenant-windows-store) (aplicación de UWP que usa ADAL.NET para llamar a una API web multiinquilino)|
Mobile (Android, iOS, UWP)   | ![Esta imagen muestra el logotipo de .NET/C# (Xamarin)](media/sample-v2-code/logo-xamarin.png) | Interactive | [dotnet-native-multitarget](https://github.com/azure-samples/active-directory-dotnet-native-multitarget) |
Móvil (Android)           | ![Esta imagen muestra el logotipo de Android](media/sample-v2-code/logo-android.png) | Interactive |   [android](https://github.com/Azure-Samples/active-directory-android) |
Mobile (iOS)           | ![Esta imagen muestra iOS/Objective-C o Swift](media/sample-v2-code/logo-ios.png) | Interactive |   [nativeClient-iOS](https://github.com/azureadquickstarts/nativeclient-ios) |
Escritorio (consola)          | ![Esta imagen muestra el logotipo de .NET/C#](media/sample-v2-code/logo-net.png) | Nombre de usuario y contraseña </p>  Autenticación integrada de Windows | | [dotnet-native-headless](https://github.com/azure-samples/active-directory-dotnet-native-headless)
Escritorio (consola)          | ![Esta imagen muestra el logotipo de Java](media/sample-v2-code/logo-java.png) | Nombre de usuario y contraseña | | [java-native-headless](https://github.com/Azure-Samples/active-directory-java-native-headless)
Escritorio (consola)           | ![Esta imagen muestra el logotipo de .NET Core/C#](media/sample-v2-code/logo-netcore.png) | Flujo de código de dispositivo | | [dotnet-deviceprofile](https://github.com/Azure-Samples/active-directory-dotnet-deviceprofile)

## <a name="daemon-applications-accessing-web-apis-with-the-applications-identity"></a>Aplicaciones demonio (que acceden a las API web con la identidad de la aplicación)

Los siguientes ejemplos muestran aplicaciones de escritorio o web que tienen acceso a Microsoft Graph o a una API web sin usuario (con la identidad de la aplicación).

Aplicación cliente | Plataforma | Flujo y concesión | Llama a una API web de ASP.NET o de ASP.NET Core 2.0
------------------ | -------- | ---------- | -------------------- 
Aplicación demonio (consola)          | ![Esta imagen muestra el logotipo de .NET](media/sample-v2-code/logo-netframework.png) | Credenciales del cliente con secreto de la aplicación o certificado | [dotnet-daemon](https://github.com/azure-samples/active-directory-dotnet-daemon)</p> [dotnet-daemon-certificate-credential](https://github.com/azure-samples/active-directory-dotnet-daemon-certificate-credential)
Aplicación demonio (consola)         | ![Esta imagen muestra el logotipo de .NET](media/sample-v2-code/logo-netcore.png) | Credenciales del cliente con certificado| [dotnetcore-daemon-certificate-credential](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-certificate-credential)
Aplicación web para ASP.NET  | ![Esta imagen muestra el logotipo de .NET](media/sample-v2-code/logo-netframework.png) | Credenciales de cliente | [dotnet-webapp-webapi-oauth2-appidentity](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-oauth2-appidentity)

## <a name="web-apis"></a>API web

### <a name="web-api-protected-by-azure-active-directory"></a>API web protegida por Azure Active Directory

En el ejemplo siguiente se muestra cómo proteger una API web de node.js con Azure AD.

En las secciones anteriores de este artículo, puede encontrar también otros ejemplos que ilustran una aplicación cliente **que llama** a una **API web** ASP.NET o ASP.NET Core. Estos ejemplos no se mencionan de nuevo en esta sección, pero puede encontrarlos en la última columna de las tablas, encima o debajo.

| Plataforma | Muestra |
|--------|-------------------|
| ![Esta imagen muestra el logotipo de Node.js](media/sample-v2-code/logo-nodejs.png)  | [node-webapi](https://github.com/Azure-Samples/active-directory-node-webapi) |

### <a name="web-api-calling-microsoft-graph-or-another-web-api"></a>API web llamando a Microsoft Graph o a otra API web

Los ejemplos siguientes muestran una API web que llama a otra API web. El segundo ejemplo muestra cómo controlar el acceso condicional.

| Plataforma |  Llama a Microsoft Graph | Llama a otra API web de ASP.NET o de ASP.NET Core 2.0 |
| -------- |  --------------------- | ------------------------- |
| ![Esta imagen muestra el logotipo de ASP.NET](media/sample-v2-code/logo-netframework.png)</p> ASP.NET 4.5 | [dotnet-webapi-onbehalfof](https://github.com/azure-samples/active-directory-dotnet-webapi-onbehalfof) </p> [dotnet-webapi-onbehalfof-ca](https://github.com/azure-samples/active-directory-dotnet-webapi-onbehalfof-ca) | [dotnet-webapi-onbehalfof](https://github.com/azure-samples/active-directory-dotnet-webapi-onbehalfof) </p> [dotnet-webapi-onbehalfof-ca](https://github.com/azure-samples/active-directory-dotnet-webapi-onbehalfof-ca) |

## <a name="other-microsoft-graph-samples"></a>Otros ejemplos de Microsoft Graph

Para obtener ejemplos y tutoriales en los que se muestren los diferentes patrones de uso de Microsoft Graph API, incluida la autenticación con Azure AD, consulte [Microsoft Graph Community samples & tutorials](https://github.com/microsoftgraph/msgraph-community-samples) (Ejemplos y tutoriales de la comunidad de Microsoft Graph).

## <a name="see-also"></a>Consulte también

- [Guía del desarrollador de Azure Active Directory](v1-overview.md)
- [Bibliotecas de autenticación de Azure Active Directory](active-directory-authentication-libraries.md)
- [Conceptos y referencia de Microsoft Graph API](/graph/use-the-api)