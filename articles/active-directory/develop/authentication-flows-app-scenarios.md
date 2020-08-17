---
title: Escenarios de flujos y aplicaciones de autenticación para la plataforma de identidad de Microsoft | Azure
description: Más información sobre los escenarios de aplicación para la Plataforma de identidad de Microsoft, incluida la autenticación de identidades, la adquisición de tokens y la llamada a API protegidas.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 03/03/2020
ms.author: jmprieur
ms.custom: aaddev, identityplatformtop40, scenarios:getting-started
ms.openlocfilehash: 99d16407d1cfadcb7d3a79968fae4436e7c631e3
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88115652"
---
# <a name="authentication-flows-and-application-scenarios"></a>Flujos de autenticación y escenarios de aplicaciones

El punto de conexión de la plataforma de identidad de Microsoft (v 2.0) admite la autenticación para diferentes tipos de arquitecturas de aplicación modernas. Todas ellas se basan en los protocolos estándar del sector [OAuth 2.0 y OpenID Connect](active-directory-v2-protocols.md). Con las [bibliotecas de autenticación de la plataforma de identidad de Microsoft](reference-v2-libraries.md), las aplicaciones autentican las identidades y adquieren tokens para acceder a API protegidas.

En este artículo se describen los flujos de autenticación y los escenarios de aplicaciones en los que se usan.

## <a name="application-categories"></a>Categorías de aplicaciones

Los tokens se pueden adquirir a partir de varios tipos de aplicaciones, entre las que se incluyen:

- Aplicaciones web
- Aplicaciones móviles
- Aplicaciones de escritorio
- API web

Los tokens también pueden ser adquiridos por aplicaciones que se ejecutan en dispositivos sin explorador o en Internet de las cosas (IoT).

En las secciones siguientes se describen las categorías de aplicaciones.

### <a name="protected-resources-vs-client-applications"></a>Recursos protegidos frente a aplicaciones cliente

Los escenarios de autenticación implican dos actividades:

- **Adquisición de tokens de seguridad para una API web protegida**: se recomienda usar [bibliotecas cliente compatibles con Microsoft](reference-v2-libraries.md#microsoft-supported-client-libraries) para adquirir tokens. En concreto, se recomienda la familia de la Biblioteca de autenticación de Microsoft (MSAL).
- **Protección de una API web o una aplicación web**: un reto a la hora de proteger estos recursos es validar el token de seguridad. En algunas plataformas, Microsoft ofrece [bibliotecas de middleware](reference-v2-libraries.md#microsoft-supported-server-middleware-libraries).

### <a name="with-users-or-without-users"></a>Con usuarios o sin usuarios

La mayoría de los escenarios de autenticación adquieren tokens en nombre de un usuario con la sesión iniciada.

![Escenarios con usuarios](media/scenarios/scenarios-with-users.svg)

Sin embargo, también hay aplicaciones de demonio. En estos escenarios, las aplicaciones adquieren los tokens en su propio nombre, sin usuario.

![Escenarios con aplicaciones de demonio](media/scenarios/daemon-app.svg)

### <a name="single-page-public-client-and-confidential-client-applications"></a>Aplicaciones cliente públicas y confidenciales, de página única

Hay varios tipos de aplicaciones que pueden adquirir tokens de seguridad. Estas aplicaciones tienden a dividirse en las tres categorías siguientes. Cada se usa con diferentes bibliotecas y objetos.

- **Aplicaciones de página única**: también conocidas como SPA, son aplicaciones web en las que los tokens son adquiridos por una aplicación de JavaScript o TypeScript que se ejecuta en el explorador. Muchas aplicaciones modernas tienen una aplicación de página única en el front-end, que está escrita principalmente en JavaScript. A menudo, la aplicación usa un marco como Angular, React o Vue. MSAL.js es la única biblioteca de autenticación de Microsoft que admite aplicaciones de página única.

- **Aplicaciones cliente públicas**: las aplicaciones de esta categoría, como los tipos siguientes, siempre inician la sesión de los usuarios:
  - Aplicaciones de escritorio que llaman a API web en nombre de los usuarios que han iniciado sesión
  - Aplicaciones móviles
  - Aplicaciones que se ejecutan en dispositivos sin explorador, como las que se ejecutan en IoT
  
- **Aplicaciones cliente confidenciales**: las aplicaciones de esta categoría incluyen:
  - Aplicaciones web que llaman a una API web
  - API web que llaman a una API web
  - Aplicaciones de demonio, incluso cuando se implementan como servicio de consola, como un demonio de Linux o un servicio de Windows

### <a name="sign-in-audience"></a>Inicio de sesión de la audiencia

los flujos de autenticación disponibles difieren en función de la audiencia de inicio de sesión. Algunos flujos solo están disponibles para las cuentas profesionales o educativas. Otros están disponibles para las cuentas profesionales o educativas y para las cuentas Microsoft personales.

Para más información, consulte [Tipos de cuenta admitidos](v2-supported-account-types.md#account-type-support-in-authentication-flows).

## <a name="application-scenarios"></a>Escenarios de aplicación

El punto de conexión de la plataforma de identidad de Microsoft admite la autenticación en estas arquitecturas de aplicaciones:

- Aplicaciones de una sola página
- Aplicaciones web
- API web
- Aplicaciones móviles
- Aplicaciones nativas
- Aplicaciones de demonio
- Aplicaciones de servidor

Las aplicaciones usan los distintos flujos de autenticación para iniciar la sesión de los usuarios y obtener tokens para llamar a API protegidas.

### <a name="single-page-application"></a>Aplicación de una sola página

Muchas aplicaciones web modernas se compilan como aplicaciones de página única del lado cliente. Estas aplicaciones usan JavaScript o un marco de trabajo como Angular, Vue.js y React.js. Estas aplicaciones se ejecutan en un explorador web.

Las aplicaciones de página única difieren de las aplicaciones web tradicionales del lado servidor en las características de autenticación. Con la plataforma de identidad de Microsoft las aplicaciones de página única pueden iniciar la sesión de los usuarios y obtengan tokens para acceder a los servicios de back-end o a las API web.

![Aplicación de página única](media/scenarios/spa-app.svg)

Para más información, consulte [Aplicaciones de página única](scenario-spa-overview.md).

### <a name="web-app-that-signs-in-a-user"></a>Aplicación web que inicia la sesión de un usuario

![Aplicación web que inicia la sesión de un usuario](media/scenarios/scenario-webapp-signs-in-users.svg)

Para ayudar a proteger una aplicación web que inicia la sesión de un usuario:

- Si se está desarrollando en .NET, se usa ASP.NET o ASP.NET Core con el middleware ASP.NET OpenID Connect. La protección de un recurso conlleva la validación del token de seguridad, que se realiza con las [extensiones de IdentityModel para .NET](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/wiki) y no con las bibliotecas MSAL.

- Si se desarrolla en Node.js, se usa Passport.js.

Para más información, consulte [Aplicación web que permite iniciar sesión a los usuarios](scenario-web-app-sign-user-overview.md).

### <a name="web-app-that-signs-in-a-user-and-calls-a-web-api-on-behalf-of-the-user"></a>Aplicación web que inicia la sesión de un usuario y llama a una API web en nombre del usuario

![Aplicaciones web que llama a API web](media/scenarios/web-app.svg)

Para llamar a una API web desde una aplicación web en nombre de un usuario, use el flujo de código de autorización y almacene los tokens adquiridos en la caché de tokens. Cuando sea necesario, MSAL actualiza los tokens y el controlador adquiere los tokens de la memoria caché de forma silenciosa.

Para más información, consulte [Aplicación web que llama a las API web](scenario-web-app-call-api-overview.md).

### <a name="desktop-app-that-calls-a-web-api-on-behalf-of-a-signed-in-user"></a>Aplicación de escritorio que llama a una API web en nombre de un usuario que ha iniciado sesión

Para que una aplicación de escritorio llame a una API web que inicia la sesión de los usuarios, use los métodos de adquisición de tokens interactivos de MSAL. Estos métodos interactivos permiten controlar la experiencia de inicio de sesión en la interfaz de usuario. MSAL usa un explorador web para esta interacción.

![Aplicación de escritorio que llama a una API web](media/scenarios/desktop-app.svg)

Existe otra posibilidad para las aplicaciones hospedadas en Windows en equipos unidos, ya sea a un dominio de Windows, o mediante Azure Active Directory (Azure AD). Estas aplicaciones pueden adquirir un token de forma silenciosa con la [autenticación integrada de Windows](https://aka.ms/msal-net-iwa).

Las aplicaciones que se ejecutan en un dispositivo sin explorador podrán seguir llamando a una API en nombre de un usuario. Para realizar la autenticación, el usuario debe iniciar sesión en otro dispositivo que tenga un explorador web. En este escenario es necesario usar el [flujo de código del dispositivo](https://aka.ms/msal-net-device-code-flow).

![Flujo de código de dispositivo](media/scenarios/device-code-flow-app.svg)

Aunque no se recomienda usarlo, el [flujo de nombre de usuario y contraseña](scenario-desktop-acquire-token.md#username-and-password) está disponible en las aplicaciones cliente públicas. Este flujo sigue siendo necesario en algunos escenarios, como DevOps.

El uso del flujo de nombre de usuario y contraseña restringe las aplicaciones. Por ejemplo, las aplicaciones no pueden iniciar la sesión de un usuario que tenga que usar autenticación multifactor o la herramienta Acceso condicional de Azure AD. Las aplicaciones tampoco se benefician del inicio de sesión único. La autenticación mediante el flujo de nombre de usuario y contraseña va en contra de los principios de la autenticación moderna y solo se proporciona por motivos de herencia.

En aplicaciones de escritorio, si quiere que la caché de tokens persista, puede personalizar la [serialización de la caché de tokens](scenario-desktop-acquire-token.md#file-based-token-cache). Con la implementación de la [serialización de la caché de tokens dual](scenario-desktop-acquire-token.md#dual-token-cache-serialization-msal-unified-cache--adal-v3), puede usar cachés de tokens compatibles con versiones anteriores y posteriores. Estos tokens admiten generaciones anteriores de bibliotecas de autenticación. Las bibliotecas específicas incluyen las versiones 3 y 4 de la biblioteca de autenticación de Azure AD para .NET (ADAL.NET).

Para más información, consulte [Aplicación de escritorio que llama a las API web](scenario-desktop-overview.md).

### <a name="mobile-app-that-calls-a-web-api-on-behalf-of-an-interactive-user"></a>Aplicación móvil que llama a una API web en nombre de un usuario interactivo

De forma parecida a una aplicación de escritorio, una aplicación móvil llama a los métodos interactivos de adquisición de tokens de MSAL para adquirir un token para llamar a una API web.

![Aplicación móvil que llama a una API web](media/scenarios/mobile-app.svg)

MSAL iOS y MSAL Android usan el explorador web del sistema de forma predeterminada. Pero puede dirigirlos para que usen la vista web insertada en su lugar. Existen algunas características específicas que dependen de la plataforma móvil: Plataforma universal de Windows (UWP), iOS o Android.

Algunos escenarios, como los que implican el acceso condicional relacionado con un identificador de dispositivo o una inscripción de dispositivo, requieren que se instale un agente en el dispositivo. Algunos ejemplos de agentes son Portal de empresa de Microsoft en Android y Microsoft Authenticator en Android e iOS. MSAL ahora puede interactuar con los agentes. Para más información sobre los agentes, consulte [Aprovechamiento de agentes en Android e iOS](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/leveraging-brokers-on-Android-and-iOS).

Para más información, consulte [Aplicación móvil que llama a las API web](scenario-mobile-overview.md).

> [!NOTE]
> Una aplicación móvil que use MSAL.iOS, MSAL.Android o MSAL.NET en Xamarin puede tener aplicadas directivas de protección de aplicaciones. Por ejemplo, las directivas pueden impedir que un usuario copie texto protegido. Intune administra la aplicación móvil y reconoce este servicio como una aplicación administrada. Para más información, consulte [Introducción al SDK para aplicaciones de Microsoft Intune](/intune/app-sdk).
>
> El [SDK de la aplicación Intune](/intune/app-sdk-get-started) es independiente de las bibliotecas de MSAL e interactúa con Azure AD por sí solo.

### <a name="protected-web-api"></a>API web protegida

Puede usar el punto de conexión de la Plataforma de identidad de Microsoft para proteger servicios web como la API web RESTful de la aplicación. Se llama a una API web protegida mediante un token de acceso. El token ayuda a proteger los datos de la API y a autenticar las solicitudes entrantes. El llamador de una API web anexa un token de acceso al encabezado de autorización de una solicitud HTTP.

Si quiere proteger su API web de ASP.NET o ASP.NET Core, debe validar el token de acceso. Para la validación se usa el middleware JWT de ASP.NET. Esta validación se realiza con la biblioteca de [extensiones IdentityModel para .NET](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/wiki), no con MSAL.NET.

Para más información, consulte [API web protegida](scenario-protected-web-api-overview.md).

### <a name="web-api-that-calls-another-web-api-on-behalf-of-a-user"></a>API web que llama a otra API web en nombre de un usuario

Para que la API web protegida llame a otra API web en nombre de un usuario, la aplicación debe adquirir un token para la API web de bajada. Estas llamadas a veces se denominan llamadas de *servicio a servicio*. Las API web que llaman a otras API web tienen que proporcionar serialización de caché personalizada.

![API web que llama a otra API web](media/scenarios/web-api.svg)

Para más información, consulte [API web que llama a las API web](scenario-web-api-call-api-overview.md).

### <a name="daemon-app-that-calls-a-web-api-in-the-daemons-name"></a>Aplicación de demonio que llama a una API web en nombre del demonio

Las aplicaciones que contienen procesos de larga duración o que funcionan sin la interacción de usuario también necesitan un modo de acceder a API web protegidas. Estas aplicaciones pueden autenticarse y obtener tokens mediante la identidad de la aplicación. La aplicación demuestra su identidad mediante un certificado o secreto de cliente.

Puede escribir estas aplicaciones de demonio que adquieren un token para la aplicación que realiza la llamada con los métodos de adquisición de [credenciales de cliente](scenario-daemon-acquire-token.md#acquiretokenforclient-api) de MSAL. Estos métodos requieren un secreto de cliente que se agrega al registro de aplicación en Azure AD. A continuación, la aplicación comparte el secreto con el demonio al que ha llamado. Ejemplos de estos secretos son las contraseñas de aplicación, la aserción de certificado y la aserción de cliente.

![Aplicación de demonio a la que llaman otras aplicaciones y API](media/scenarios/daemon-app.svg)

Para más información, consulte [Aplicación de demonio que llama a las API web](scenario-daemon-overview.md).

## <a name="scenarios-and-supported-authentication-flows"></a>Escenarios y flujos de autenticación compatibles

Puede usar flujos de autenticación para implementar los escenarios de aplicaciones que solicitan tokens. No hay ninguna asignación individual entre los escenarios de aplicaciones y los flujos de autenticación.

Los escenarios que implican la adquisición de tokens también se asignan a flujos de autenticación de OAuth 2.0. Para más información, consulte [Protocolos OAuth 2.0 y OpenID Connect en la Plataforma de identidad de Microsoft](active-directory-v2-protocols.md).

<table>
 <thead>
  <tr><th>Escenario</th> <th>Tutorial detallado de escenarios</th> <th>Flujo y concesión de OAuth 2.0</th> <th>Público</th></tr>
 </thead>
 <tbody>
  <tr>
   <td><a href="scenario-spa-overview.md"><img alt="Single-Page App" src="media/scenarios/spa-app.svg"></a></td>
   <td><a href="scenario-spa-overview.md">Aplicación de una sola página</a></td>
   <td><a href="v2-oauth2-implicit-grant-flow.md">Implícito</a></td>
   <td>Cuentas profesionales o educativas, cuentas personales y Azure Active Directory B2C (Azure AD B2C)</td>
 </tr>

  <tr>
   <td><a href="scenario-web-app-sign-user-overview.md"><img alt="Web app that signs in users" src="media/scenarios/scenario-webapp-signs-in-users.svg"></a></td>
   <td><a href="scenario-web-app-sign-user-overview.md">Aplicación web que permite iniciar sesión a los usuarios</a></td>
   <td><a href="v2-oauth2-auth-code-flow.md">Código de autorización</a></td>
   <td>Cuentas profesionales o educativas, cuentas personales y Azure AD B2C</td>
 </tr>

  <tr>
   <td><a href="scenario-web-app-call-api-overview.md"><img alt="Web app that signs in users" src="media/scenarios/web-app.svg"></a></td>
   <td><a href="scenario-web-app-call-api-overview.md">Aplicación web que llama a las API web</a></td>
   <td><a href="v2-oauth2-auth-code-flow.md">Código de autorización</a></td>
   <td>Cuentas profesionales o educativas, cuentas personales y Azure AD B2C</td>
 </tr>

  <tr>
   <td rowspan="3"><a href="scenario-desktop-overview.md"><img alt=Desktop app that calls web APIs" src="media/scenarios/desktop-app.svg"></a></td>
   <td rowspan="4"><a href="scenario-desktop-overview.md">Aplicación de escritorio que llama a las API web</a></td>
   <td>De forma interactiva mediante un <a href="v2-oauth2-auth-code-flow.md">código de autorización</a> con PKCE</td>
   <td>Cuentas profesionales o educativas, cuentas personales y Azure AD B2C</td>
 </tr>

  <tr>
   <td>Autenticación integrada de Windows</td>
   <td>Cuentas profesionales o educativas</td>
 </tr>

  <tr>
   <td><a href="v2-oauth-ropc.md">Contraseña del propietario del recurso</a></td>
   <td>Cuentas profesionales o educativas y Azure AD B2C</td>
 </tr>

  <tr>
   <td><a href="scenario-desktop-acquire-token.md#command-line-tool-without-a-web-browser"><img alt="Browserless application" src="media/scenarios/device-code-flow-app.svg"></a></td>
   <td><a href="v2-oauth2-device-code.md">Código del dispositivo</a></td>
   <td>Cuentas profesionales o educativas</td>
 </tr>

 <tr>
   <td rowspan="2"><a href="scenario-mobile-overview.md"><img alt="Mobile app that calls web APIs" src="media/scenarios/mobile-app.svg"></a></td>
   <td rowspan="2"><a href="scenario-mobile-overview.md">Aplicación móvil que llama a las API web</a></td>
   <td>De forma interactiva mediante un <a href="v2-oauth2-auth-code-flow.md">código de autorización</a> con PKCE</td>
   <td>Cuentas profesionales o educativas, cuentas personales y Azure AD B2C</td>
 </tr>

  <tr>
   <td><a href="v2-oauth-ropc.md">Contraseña del propietario del recurso</a></td>
   <td>Cuentas profesionales o educativas y Azure AD B2C</td>
 </tr>

  <tr>
   <td><a href="scenario-daemon-overview.md"><img alt="Daemon app that calls web APIs" src="media/scenarios/daemon-app.svg"></a></td>
   <td><a href="scenario-daemon-overview.md">Aplicación de demonio que llama a las API web</a></td>
   <td><a href="v2-oauth2-client-creds-grant-flow.md">Credenciales de cliente</a></td>
   <td>Permisos solo de aplicación sin usuario y que solo se usan en organizaciones de Azure AD</td>
 </tr>

  <tr>
   <td><a href="scenario-web-api-call-api-overview.md"><img alt="Web API that calls web APIs" src="media/scenarios/web-api.svg"></a></td>
   <td><a href="scenario-web-api-call-api-overview.md">API web que llama a API web</a></td>
   <td><a href="v2-oauth2-on-behalf-of-flow.md">En nombre de</a></td>
   <td>Cuentas profesionales o educativas y cuentas personales</td>
 </tr>

 </tbody>
</table>

## <a name="scenarios-and-supported-platforms-and-languages"></a>Escenarios, plataformas y lenguajes compatibles

Las bibliotecas de autenticación de Microsoft admiten varias plataformas:

- JavaScript
- .NET Framework
- .NET Core
- Windows 10/UWP
- Xamarin.iOS
- Xamarin.Android
- iOS nativo
- macOS
- Android nativo
- Java
- Python

También puede usar varios lenguajes para compilar las aplicaciones.

> [!NOTE]
> Algunos tipos de aplicación no están disponibles en todas las plataformas.

En la columna Windows de la tabla siguiente, cada vez que se menciona .NET Core, también es posible .NET Framework. El último se omite para evitar la acumulación en la tabla.

|Escenario  | Windows | Linux | Mac | iOS | Android
|--|--|--|--|--|--|--|
| [Aplicación de una sola página](scenario-spa-overview.md) <br/>[![Aplicación de página única](media/scenarios/spa-app.svg)](scenario-spa-overview.md) | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png) MSAL.js | ![MSAL.js](media/sample-v2-code/small_logo_js.png)<br/>MSAL.js
| [Aplicación web que permite iniciar sesión a los usuarios](scenario-web-app-sign-user-overview.md) <br/>[![Aplicación web que inicia sesión de usuarios](media/scenarios/scenario-webapp-signs-in-users.svg)](scenario-web-app-sign-user-overview.md) | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core
| [Aplicación web que llama a las API web](scenario-web-app-call-api-overview.md) <br/> <br/>[![Aplicación web que llama a las API web](media/scenarios/web-app.svg)](scenario-web-app-call-api-overview.md) | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core y MSAL.NET ![Java de MSAL](media/sample-v2-code/small_logo_java.png) <br/>Java de MSAL<br/>![Python de MSAL](media/sample-v2-code/small_logo_python.png)<br/>Flask y Python de MSAL| ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core y MSAL.NET ![Java de MSAL](media/sample-v2-code/small_logo_java.png)<br/>Java de MSAL<br/>![Python de MSAL](media/sample-v2-code/small_logo_python.png)<br/>Flask y Python de MSAL| ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core y MSAL.NET ![Java de MSAL](media/sample-v2-code/small_logo_java.png)<br/>Java de MSAL<br/> ![Python de MSAL](media/sample-v2-code/small_logo_python.png)<br/>Flask y Python de MSAL
| [Aplicación de escritorio que llama a las API web](scenario-desktop-overview.md) <br/> <br/>[![Aplicación de escritorio que llama a las API web](media/scenarios/desktop-app.svg)](scenario-desktop-overview.md) ![Flujo de código de dispositivo](media/scenarios/device-code-flow-app.svg) | ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![Java de MSAL](media/sample-v2-code/small_logo_java.png)<br/>Java de MSAL<br/> ![Python de MSAL](media/sample-v2-code/small_logo_python.png)<br/>Python de MSAL| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![Java de MSAL](media/sample-v2-code/small_logo_java.png)<br/>Java de MSAL<br/>![Python de MSAL](media/sample-v2-code/small_logo_python.png)<br/>Python de MSAL| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![Java de MSAL](media/sample-v2-code/small_logo_java.png)<br/>Java de MSAL<br/>![Python de MSAL](media/sample-v2-code/small_logo_python.png)<br/>Python de MSAL <br/> ![iOS / Objective C o Swift](media/sample-v2-code/small_logo_iOS.png) MSAL.objc |
| [Aplicación móvil que llama a las API web](scenario-mobile-overview.md) <br/> [![Aplicación móvil que llama a las API web](media/scenarios/mobile-app.svg)](scenario-mobile-overview.md) | ![UWP](media/sample-v2-code/small_logo_windows.png) MSAL.NET ![Xamarin](media/sample-v2-code/small_logo_xamarin.png) MSAL.NET | | | ![iOS / Objective C o Swift](media/sample-v2-code/small_logo_iOS.png) MSAL.objc | ![Android](media/sample-v2-code/small_logo_Android.png) MSAL.Android
| [Aplicación de demonio](scenario-daemon-overview.md) <br/> [![Aplicación de demonio](media/scenarios/daemon-app.svg)](scenario-daemon-overview.md) | ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![Java de MSAL](media/sample-v2-code/small_logo_java.png)<br/>Java de MSAL<br/>![Python de MSAL](media/sample-v2-code/small_logo_python.png)<br/>Python de MSAL| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png) MSAL.NET ![Java de MSAL](media/sample-v2-code/small_logo_java.png)<br/>Java de MSAL<br/>![Python de MSAL](media/sample-v2-code/small_logo_python.png)<br/>Python de MSAL| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)MSAL.NET ![Java de MSAL](media/sample-v2-code/small_logo_java.png)<br/>Java de MSAL<br/>![Python de MSAL](media/sample-v2-code/small_logo_python.png)<br/>Python de MSAL
| [API web que llama a API web](scenario-web-api-call-api-overview.md) <br/><br/> [![API web que llama a API web](media/scenarios/web-api.svg)](scenario-web-api-call-api-overview.md) | ![ASP.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core y MSAL.NET ![Java de MSAL](media/sample-v2-code/small_logo_java.png)<br/>Java de MSAL<br/>![Python de MSAL](media/sample-v2-code/small_logo_python.png)<br/>Python de MSAL| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core y MSAL.NET ![Java de MSAL](media/sample-v2-code/small_logo_java.png)<br/>Java de MSAL<br/>![Python de MSAL](media/sample-v2-code/small_logo_python.png)<br/>Python de MSAL| ![.NET Core](media/sample-v2-code/small_logo_NETcore.png)<br/>ASP.NET Core y MSAL.NET ![Java de MSAL](media/sample-v2-code/small_logo_java.png)<br/>Java de MSAL<br/>![Python de MSAL](media/sample-v2-code/small_logo_python.png)<br/>Python de MSAL

Para más información, consulte [Bibliotecas compatibles con Microsoft por sistema operativo y lenguaje](reference-v2-libraries.md#microsoft-supported-libraries-by-os--language).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre los [aspectos básicos de la autenticación](./authentication-vs-authorization.md) y los [tokens de acceso de la plataforma de identidad de Microsoft](access-tokens.md).
* Más información sobre la [protección del acceso a aplicaciones IoT](/azure/architecture/example-scenario/iot-aad/iot-aad).