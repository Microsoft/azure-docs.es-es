---
title: Bibliotecas de autenticación de Azure Active Directory | Microsoft Docs
description: La biblioteca de autenticación de Azure AD (ADAL) permite a los desarrolladores de aplicaciones cliente autenticar fácilmente a los usuarios en Active Directory (AD) local o en la nube y luego obtener tokens de acceso para proteger las llamadas de API.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: azuread-dev
ms.topic: reference
ms.workload: identity
ms.date: 12/01/2018
ms.author: ryanwi
ms.reviewer: saeeda, jmprieur
ms.custom: aaddev
ROBOTS: NOINDEX
ms.openlocfilehash: cd31171d04dcf0102472e71fdcdc058471d79fb0
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88117726"
---
# <a name="azure-active-directory-authentication-libraries"></a>Bibliotecas de autenticación de Azure Active Directory

[!INCLUDE [active-directory-azuread-dev](../../../includes/active-directory-azuread-dev.md)]

La Biblioteca de autenticación de Azure Active Directory (ADAL) versión 1.0 permite a los desarrolladores de aplicaciones autenticar a los usuarios en Active Directory (AD) local o en la nube y obtener tokens para proteger las llamadas API. ADAL facilita la autenticación para los desarrolladores a través de características como:

- Una memoria caché de tokens configurable que almacena tokens de acceso y tokens de actualización
- Actualización de tokens automática disponible cuando expira un token de acceso y un token de actualización
- Compatibilidad con las llamadas a métodos asincrónicos

> [!NOTE]
> ¿Busca las bibliotecas de Azure AD v2.0 (MSAL)? Revise la [Guía de la biblioteca MSAL](../develop/reference-v2-libraries.md).
>
>

## <a name="microsoft-supported-client-libraries"></a>Bibliotecas de cliente compatibles con Microsoft

| Plataforma | Biblioteca | Descargar | Código fuente | Muestra | Referencia
| --- | --- | --- | --- | --- | --- |
| Cliente .NET, Tienda Windows, UWP, Xamarin iOS y Android |ADAL .NET v3 |[NuGet](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet) | [Aplicación de escritorio](../develop/quickstart-v2-windows-desktop.md) |[Referencia](/dotnet/api/microsoft.identitymodel.clients.activedirectory?view=azure-dotnet) |
| JavaScript |ADAL.js |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-js) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-js) |[Aplicación de una sola página](https://github.com/Azure-Samples/active-directory-javascript-singlepageapp-dotnet-webapi) | |
| iOS, macOS |ADAL |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-objc/releases) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-objc) |[Aplicación para iOS](../develop/quickstart-v2-ios.md) | [Referencia](http://cocoadocs.org/docsets/ADAL/2.5.1/)|
| Android |ADAL |[Maven](https://search.maven.org/search?q=g:com.microsoft.aad+AND+a:adal&core=gav) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-android) |[Aplicación para Android](../develop/quickstart-v2-android.md) | [JavaDocs](https://javadoc.io/doc/com.microsoft.aad/adal/)|
| Node.js |ADAL |[npm](https://www.npmjs.com/package/adal-node) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-nodejs) | [Aplicación web para Node.js](https://github.com/Azure-Samples/active-directory-node-webapp-openidconnect)|[Referencia](/javascript/api/overview/azure/activedirectory) |
| Java |ADAL4J |[Maven](https://search.maven.org/#search%7Cga%7C1%7Ca%3Aadal4j%20g%3Acom.microsoft.azure) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-java) |[Aplicaciones web de Java](https://github.com/Azure-Samples/active-directory-java-webapp-openidconnect) |[Referencia](https://javadoc.io/doc/com.microsoft.azure/adal4j) |
| Python |ADAL |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-python) |[GitHub](https://github.com/AzureAD/azure-activedirectory-library-for-python) |[Aplicación web de Python](https://github.com/Azure-Samples/active-directory-python-webapp-graphapi) |[Referencia](https://adal-python.readthedocs.io/) |

## <a name="microsoft-supported-server-libraries"></a>Bibliotecas de servidor compatibles con Microsoft

| Plataforma | Biblioteca | Descargar | Código fuente | Muestra | Referencia
| --- | --- | --- | --- | --- | --- |
| .NET |OWIN para AzureAD|[NuGet](https://www.nuget.org/packages/Microsoft.Owin.Security.ActiveDirectory/) |[GitHub](https://github.com/aspnet/AspNetKatana/tree/dev/src/Microsoft.Owin.Security.ActiveDirectory) |[Aplicación MVC](../develop/quickstart-v2-aspnet-webapp.md) | |
| .NET |OWIN para OpenIDConnect |[NuGet](https://www.nuget.org/packages/Microsoft.Owin.Security.OpenIdConnect) |[GitHub](https://github.com/aspnet/AspNetKatana/tree/dev/src/Microsoft.Owin.Security.OpenIdConnect) |[Aplicación web](https://github.com/AzureADSamples/WebApp-OpenIDConnect-DotNet) | |
| .NET |OWIN para WS-Federation |[NuGet](https://www.nuget.org/packages/Microsoft.Owin.Security.WsFederation) |[GitHub](https://github.com/aspnet/AspNetKatana/tree/dev/src/Microsoft.Owin.Security.WsFederation) |[Aplicación web MVC](https://github.com/AzureADSamples/WebApp-WSFederation-DotNet) | |
| .NET |Extensiones de protocolo de identidad para .NET Framework 4.5 |[NuGet](https://www.nuget.org/packages/Microsoft.IdentityModel.Protocol.Extensions) |[GitHub](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet) | | |
| .NET |Controlador JWT para .NET 4.5 |[NuGet](https://www.nuget.org/packages/System.IdentityModel.Tokens.Jwt) |[GitHub](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet) | | |
| Node.js |Azure AD Passport |[npm](https://www.npmjs.com/package/passport-azure-ad) |[GitHub](https://github.com/AzureAD/passport-azure-ad) | [API web](../develop/authentication-flows-app-scenarios.md)| |

## <a name="scenarios"></a>Escenarios

Presentamos tres escenarios comunes para el uso de ADAL en un cliente que accede a un recurso remoto:

### <a name="authenticating-users-of-a-native-client-application-running-on-a-device"></a>Autenticación de los usuarios de una aplicación cliente nativa que se ejecuta en un dispositivo

En este escenario, un desarrollador tiene un cliente móvil o una aplicación de escritorio que necesita acceso a un recurso remoto, como una API web. La API web no admite llamadas anónimas y se debe llamar en el contexto de un usuario autenticado. La API web está preconfigurada para confiar en los tokens de acceso emitidos por un inquilino de Azure AD determinado. Azure AD está preconfigurado para emitir tokens de acceso para ese recurso. Para invocar la API web desde el cliente, el desarrollador usa ADAL a fin de facilitar la autenticación con Azure AD. El modo más seguro de usar ADAL es hacer que represente la interfaz de usuario para recopilar las credenciales de usuario (se representa como una ventana del explorador).

ADAL hace que resulte fácil autenticar el usuario, obtener un token de acceso y un token de actualización de Azure AD, y luego llamar a la API web mediante el token de acceso.

Para un código de ejemplo que muestra este escenario con autenticación en Azure AD, vea [Aplicación WPF cliente nativa para API web](https://github.com/azureadsamples/nativeclient-dotnet).

### <a name="authenticating-a-confidential-client-application-running-on-a-web-server"></a>Autenticación de una aplicación cliente confidencial que se ejecuta en un servidor web

En este escenario, un desarrollador tiene una aplicación en ejecución en un servidor que necesita acceso a un recurso remoto, como una API web. La API no admite llamadas anónimas, por lo que debe llamarse desde un servicio autorizado. La API web está preconfigurada para confiar en los tokens de acceso emitidos por un inquilino de Azure AD determinado. Azure AD está preconfigurado para emitir tokens de acceso para ese recurso a un servicio con las credenciales del cliente (identificador y secreto de cliente). ADAL facilita la autenticación del servicio con Azure AD al devolver un token de acceso que se puede usar para llamar a la API web. ADAL también controla la administración de la duración del token de acceso almacenándolo en la caché y renovándolo cuando sea necesario. Para un código de ejemplo que muestra este escenario, vea [Aplicación de consola demonio para API web](https://github.com/AzureADSamples/Daemon-DotNet).

### <a name="authenticating-a-confidential-client-application-running-on-a-server-on-behalf-of-a-user"></a>Autenticación de una aplicación cliente confidencial que se ejecuta en un servidor web en nombre de un usuario

En este escenario, un desarrollador tiene una aplicación web en ejecución en un servidor que necesita acceso a un recurso remoto, como una API web. La API no admite llamadas anónimas, por lo que debe llamarse desde un servicio autorizado en nombre de un usuario autenticado. La API web está preconfigurada para confiar en los tokens de acceso emitidos por un inquilino de Azure determinado, y Azure AD está preconfigurado para emitir tokens de acceso para ese recurso a un servicio con las credenciales del cliente. Cuando el usuario se autentica en la aplicación web, la aplicación puede obtener un código de autorización para el usuario de Azure AD. La aplicación web puede luego usar ADAL para obtener un token de acceso y un token de actualización en nombre del usuario con las credenciales de cliente y el código de autorización asociados a la aplicación de Azure AD. Cuando la aplicación web está en posesión del token de acceso, puede llamar a la API web hasta que el token expire. Cuando el token expira, la aplicación web puede usar ADAL para obtener un nuevo token de acceso con el token de actualización que recibió anteriormente. Para un código de ejemplo que muestra este escenario, vea [Cliente nativo de API web a API web](https://github.com/Azure-Samples/active-directory-dotnet-webapi-onbehalfof).

## <a name="see-also"></a>Consulte también

- [Guía del desarrollador de Azure Active Directory](v1-overview.md)
- [Escenarios de autenticación para Azure Active Directory](v1-authentication-scenarios.md)
- [Ejemplos de código de Azure Active Directory](sample-v1-code.md)