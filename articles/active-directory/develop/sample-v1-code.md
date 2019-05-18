---
title: Ejemplos de código de Azure Active Directory v1.0 | Microsoft Docs
description: Se proporciona un índice de ejemplos de código de Azure Active Directory (punto de conexión v1.0), organizados por escenario.
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.assetid: a242a5ff-7300-40c2-ba83-fb6035707433
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/24/2018
ms.author: ryanwi
ms.reviewer: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: b14dcb5f5af971b10c83ea7fd9064a08cde3ab35
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545430"
---
# <a name="azure-active-directory-code-samples-v10-endpoint"></a>Ejemplos de código de Azure Active Directory (punto de conexión v1.0)

[!INCLUDE [active-directory-develop-applies-v1-adal](../../../includes/active-directory-develop-applies-v1-adal.md)]

Puede usar Microsoft Azure Active Directory (Azure AD) para agregar autenticación y autorización a sus aplicaciones web y API web.

En esta sección se proporcionan vínculos a ejemplos que puede usar para aprender más sobre el punto de conexión de Azure AD v1.0. En estos ejemplos se muestra cómo se hace, junto con fragmentos de código que puede usar en sus aplicaciones. En la página de ejemplos de código, encontrará temas Léame detallados que le ayudarán con los requisitos, la instalación y la configuración. Y se comenta el código para ayudarle a comprender las secciones críticas.

> [!NOTE]
> Si está interesado en ejemplos de código de Azure AD V2, consulte [ejemplos de código de v2.0 por escenario](sample-v2-code.md).

Para comprender los escenarios básicos para cada tipo de ejemplo, consulte [Escenarios de autenticación para Azure AD](authentication-scenarios.md).

También puede contribuir a nuestros ejemplos en GitHub. Para saber cómo, vea [Ejemplos y documentación de Microsoft Azure Active Directory](https://github.com/Azure-Samples?page=3&query=active-directory).

## <a name="single-page-applications"></a>Aplicación de página única

En este ejemplo se muestra cómo escribir una aplicación de página única protegida con Azure AD.

 Plataforma | Llama a su propia API | Llama a otra API web
 -------- |  --------------------- | ------------------ 
![JavaScript](media/sample-v2-code/logo_js.png) | [javascript-singlepageapp](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi) |
![Angular JS](media/sample-v2-code/logo_angular.png) | [angularjs-singlepageapp](https://github.com/Azure-Samples/active-directory-angularjs-singlepageapp) | [angularjs-singlepageapp-cors](https://github.com/Azure-Samples/active-directory-angularjs-singlepageapp-dotnet-webapi)

## <a name="web-applications"></a>Aplicaciones web

### <a name="web-applications-signing-in-users-calling-microsoft-graph-or-a-web-api-with-the-users-identity"></a>Aplicaciones Web que inician sesión por los usuarios, llaman a Microsoft Graph o una API web con la identidad del usuario

En los ejemplos siguientes se ilustra la firma de aplicaciones web por los usuarios. Algunas de estas aplicaciones también llaman a Microsoft Graph o a su propia API web, en nombre del usuario con sesión iniciada.

 Plataforma | Únicamente inicio de sesión del usuario | Llama a Microsoft Graph o a AAD Graph| Llama a otra API web de ASP.NET o de ASP.NET Core 2.0
 -------- | ------------------- | --------------------- | -------------------------
![ASP.NET](media/sample-v2-code/logo_NETcore.png)<p/>ASP.NET Core 2.0 | [dotnet-webapp-openidconnect-aspnetcore](https://github.com/Azure-Samples/active-directory-dotnet-webapp-openidconnect-aspnetcore) | [webapp-webapi-multitenant-openidconnect-aspnetcore](https://github.com/Azure-Samples/active-directory-webapp-webapi-multitenant-openidconnect-aspnetcore/) <p/>(AAD Graph) | [dotnet-webapp-webapi-openidconnect-aspnetcore](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-openidconnect-aspnetcore)
![ASP.NET 4.5](media/sample-v2-code/logo_NETframework.png)<p/> ASP.NET 4.5 | [webApp-openidconnect-dotnet](https://docs.microsoft.com/azure/active-directory/develop/guidedsetups/active-directory-aspnetwebapp-v1) <p/> [webapp-WSFederation-dotNet](https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation) <p/> [dotnet-webapp-webapi-oauth2-useridentity](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-oauth2-useridentity) | [dotnet-webapp-multitenant-openidconnect](https://github.com/Azure-Samples/active-directory-dotnet-webapp-multitenant-openidconnect)<p/> (AAD Graph) |
![Python](media/sample-v2-code/logo_python.png) | | [python-webapp-graphapi](https://github.com/Azure-Samples/active-directory-python-webapp-graphapi)  |
![Java](media/sample-v2-code/logo_java.png)  | | [java-webapp-openidconnect](https://github.com/azure-samples/active-directory-java-webapp-openidconnect)  |
![Php](media/sample-v2-code/logo_php.png) | | [php-graphapi-web](https://github.com/Azure-Samples/active-directory-php-graphapi-web)  |

### <a name="web-applications-demonstrating-role-based-access-control-authorization"></a>Aplicaciones web que muestran el control de acceso basado en roles (autorización)

En los ejemplos siguientes se muestra cómo implementar el control de acceso basado en rol (RBAC). RBAC se usa para restringir los permisos de determinadas características de una aplicación web a determinados usuarios. Se otorga autorización a los usuarios en función de si pertenecen a un **grupo de Azure AD** o tienen un determinado **rol** de aplicación.

Plataforma | Muestra |
 -------- | ------------------- |
![ASP.NET 4.5](media/sample-v2-code/logo_NETframework.png)<p/> ASP.NET 4.5 | [dotnet-webapp-groupclaims](https://github.com/Azure-Samples/active-directory-dotnet-webapp-groupclaims) <p/>  [dotnet-webapp-roleclaims](https://github.com/Azure-Samples/active-directory-dotnet-webapp-roleclaims) | Una aplicación web de .NET 4.5 MVC que usa **roles** de Azure AD para la autorización

## <a name="desktop-and-mobile-public-client-applications-calling-microsoft-graph-or-a-web-api"></a>Aplicaciones de escritorio y móviles de cliente público que llaman a Microsoft Graph o a una API web

En los siguientes ejemplos se muestran aplicaciones cliente públicas (aplicaciones de escritorio o móviles) que acceden a Microsoft Graph o a una API web en nombre de un usuario. Según los dispositivos y plataformas, las aplicaciones pueden iniciar la sesión de los usuarios de diferentes maneras (flujos o concesiones): 

- interactivamente,
- en modo silencioso (con autenticación integrada de Windows en Windows, o nombre de usuario y contraseña), 
- o incluso mediante la delegación del inicio de sesión interactivo en otro dispositivo (flujo de código de dispositivo usado en dispositivos que no proporcionan controles web).

Aplicación cliente | Plataforma | Flujo y concesión | Llama a Microsoft Graph | Llama a una API web de ASP.NET o ASP.NET Core 2.x
------------------ | -------- | ---------- | -------------------- | -------------------------
Escritorio (WPF)           | ![.NET/C#](media/sample-v2-code/logo_NET.png)  | Interactivo | Parte de [dotnet-native-multitarget](https://github.com/azure-samples/active-directory-dotnet-native-multitarget) | [Dotnet-native-desktop](https://github.com/Azure-Samples/active-directory-dotnet-native-desktop) </p> [dotnet-native-aspnetcore](https://azure.microsoft.com/resources/samples/active-directory-dotnet-native-aspnetcore/)</p> [dotnet-webapi-manual-jwt-validation](https://github.com/azure-samples/active-directory-dotnet-webapi-manual-jwt-validation)
Móvil (UWP)            | .![.NET/C#/UWP](media/sample-v2-code/logo_Windows.png)   | Interactivo | [dotnet-native-uwp-wam](https://github.com/azure-samples/active-directory-dotnet-native-uwp-wam) </p> En este ejemplo se usa [WAM](https://docs.microsoft.com/windows/uwp/security/web-account-manager), no [ADAL.NET](https://aka.ms/adalnet).|  [dotnet-windows-store](https://github.com/Azure-Samples/active-directory-dotnet-windows-store) (aplicación de UWP que usa ADAL.NET para llamar a una API web de un único inquilino) </p> [dotnet-webapi-multitenant-windows-store](https://github.com/Azure-Samples/active-directory-dotnet-webapi-multitenant-windows-store) (aplicación de UWP que usa ADAL.NET para llamar a una API web multiinquilino)|
Mobile (Android, iOS, UWP)   | ![.NET/C# (Xamarin)](media/sample-v2-code/logo_xamarin.png) | Interactivo | [dotnet-native-multitarget](https://github.com/azure-samples/active-directory-dotnet-native-multitarget) |
Móvil (Android)           | ![Android / Java](media/sample-v2-code/logo_Android.png) | Interactivo |   [android](https://github.com/Azure-Samples/active-directory-android) |
Mobile (iOS)           | ![iOS / Objective C o Swift](media/sample-v2-code/logo_iOS.png) | Interactivo |   [nativeClient-iOS](https://github.com/azureadquickstarts/nativeclient-ios) |
Escritorio (consola)          | ![.NET/C#](media/sample-v2-code/logo_NET.png) | Nombre de usuario y contraseña </p>  Autenticación integrada de Windows | | [dotnet-native-headless](https://github.com/azure-samples/active-directory-dotnet-native-headless)
Escritorio (consola)          | ![Consola de Java](media/sample-v2-code/logo_Java.png) | Nombre de usuario y contraseña | | [java-native-headless](https://github.com/Azure-Samples/active-directory-java-native-headless)
Escritorio (consola)           | ![.NET Core/C#](media/sample-v2-code/logo_NETcore.png) | Flujo de código de dispositivo | | [dotnet-deviceprofile](https://github.com/Azure-Samples/active-directory-dotnet-deviceprofile)

## <a name="daemon-applications-accessing-web-apis-with-the-applications-identity"></a>Aplicaciones demonio (que acceden a las API web con la identidad de la aplicación)

Los siguientes ejemplos muestran aplicaciones de escritorio o web que tienen acceso a Microsoft Graph o a una API web sin usuario (con la identidad de la aplicación).

Aplicación cliente | Plataforma | Flujo y concesión | Llama a una API web de ASP.NET o de ASP.NET Core 2.0
------------------ | -------- | ---------- | -------------------- 
Aplicación demonio (consola)          | ![.NET](media/sample-v2-code/logo_NETframework.png) | Credenciales del cliente con secreto de la aplicación o certificado | [dotnet-daemon](https://github.com/azure-samples/active-directory-dotnet-daemon)</p> [dotnet-daemon-certificate-credential](https://github.com/azure-samples/active-directory-dotnet-daemon-certificate-credential)
Aplicación demonio (consola)         | ![.NET](media/sample-v2-code/logo_NETcore.png) | Credenciales del cliente con certificado| [dotnetcore-daemon-certificate-credential](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-certificate-credential)
Aplicación web para ASP.NET  | ![.NET](media/sample-v2-code/logo_NETframework.png) | Credenciales de cliente | [dotnet-webapp-webapi-oauth2-appidentity](https://github.com/Azure-Samples/active-directory-dotnet-webapp-webapi-oauth2-appidentity)

## <a name="web-apis"></a>API web

### <a name="web-api-protected-by-azure-active-directory"></a>API web protegida por Azure Active Directory

En el ejemplo siguiente se muestra cómo proteger una API web de node.js con Azure AD.

En las secciones anteriores de este artículo, puede encontrar también otros ejemplos que ilustran una aplicación cliente **que llama** a una **API web** ASP.NET o ASP.NET Core. Estos ejemplos no se mencionan de nuevo en esta sección, pero puede encontrarlos en la última columna de las tablas, encima o debajo.

Plataforma | Muestra
 -------- | -------------------
![Php](media/sample-v2-code/logo_nodejs.png)  | [node-webapi](https://github.com/Azure-Samples/active-directory-node-webapi)

### <a name="web-api-calling-microsoft-graph-or-another-web-api"></a>API web llamando a Microsoft Graph o a otra API web

Los ejemplos siguientes muestran una API web que llama a otra API web. El segundo ejemplo muestra cómo controlar el acceso condicional.

 Plataforma |  Llama a Microsoft Graph | Llama a otra API web de ASP.NET o de ASP.NET Core 2.0
 -------- |  --------------------- | -------------------------
![ASP.NET 4.5](media/sample-v2-code/logo_NETframework.png)<p/> ASP.NET 4.5 | [dotnet-webapi-onbehalfof](https://github.com/azure-samples/active-directory-dotnet-webapi-onbehalfof) <p/> [dotnet-webapi-onbehalfof-ca](https://github.com/azure-samples/active-directory-dotnet-webapi-onbehalfof-ca) |[dotnet-webapi-onbehalfof](https://github.com/azure-samples/active-directory-dotnet-webapi-onbehalfof) <p/> [dotnet-webapi-onbehalfof-ca](https://github.com/azure-samples/active-directory-dotnet-webapi-onbehalfof-ca)

## <a name="other-microsoft-graph-samples"></a>Otros ejemplos de Microsoft Graph

Para obtener ejemplos y tutoriales en los que se muestren los diferentes patrones de uso de Microsoft Graph API, incluida la autenticación con Azure AD, consulte [Microsoft Graph Community samples & tutorials](https://github.com/microsoftgraph/msgraph-community-samples) (Ejemplos y tutoriales de la comunidad de Microsoft Graph).

## <a name="see-also"></a>Vea también

[Guía del desarrollador de Azure Active Directory](v1-overview.md)

[Bibliotecas de autenticación de Azure Active Directory](active-directory-authentication-libraries.md)

[Conceptos y referencia de Graph API de Azure AD](https://msdn.microsoft.com/library/azure/hh974476.aspx)

[Biblioteca auxiliar de Graph API de Azure AD](https://www.nuget.org/packages/Microsoft.Azure.ActiveDirectory.GraphClient)
