---
title: Flujos de autenticación (biblioteca de autenticación de Microsoft) | Azure
description: Obtenga información sobre lo flujos de autenticación/concede utilizado por la biblioteca de autenticación de Microsoft (MSAL).
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/25/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: cb9a6f162a10408469669cf40b29efc6d2903944
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546049"
---
# <a name="authentication-flows"></a>Flujos de autenticación

En este artículo se describe los flujos de autenticación diferente que proporciona la biblioteca de autenticación de Microsoft (MSAL).  Estos flujos se pueden usar en una variedad de escenarios de aplicación diferente.

| Flujo | DESCRIPCIÓN | Se usa en|  
| ---- | ----------- | ------- | 
| [Interactivo](#interactive) | Obtiene el token a través de un proceso interactivo que solicita al usuario las credenciales a través de un explorador o una ventana emergente. | [Las aplicaciones de escritorio](scenario-desktop-overview.md), [aplicaciones móviles](scenario-mobile-overview.md) |
| [Concesión implícita](#implicit-grant) | Permite que la aplicación obtener tokens sin necesidad de realizar un intercambio de credenciales del servidor back-end. De esta forma, la aplicación puede iniciar la sesión del usuario, mantenerla y recibir tokens para otras API web, y todo dentro del código de cliente de JavaScript.| [Aplicaciones de página única (SPA)](scenario-spa-overview.md) |
| [Código de autorización](#authorization-code) | Se utiliza en aplicaciones que están instaladas en un dispositivo para tener acceso a recursos protegidos, como las API web. Esto le permite agregar iniciar sesión y tener acceso API a las aplicaciones de escritorio y móviles. | [Las aplicaciones de escritorio](scenario-desktop-overview.md), [aplicaciones móviles](scenario-mobile-overview.md), [aplicaciones Web](scenario-web-app-call-api-overview.md) | 
| [On-behalf-of](#on-behalf-of) | Una aplicación llama a un servicio o API web, que a su vez debe llamar a otro servicio o web API. La idea es propagar la identidad y los permisos del usuario delegado a través de la cadena de solicitud. | [API web](scenario-web-api-call-api-overview.md) |
| [Credenciales de cliente](#client-credentials) | Permite obtener acceso a los recursos hospedados en web utilizando la identidad de una aplicación. Normalmente se utiliza para las interacciones de servidor a servidor que se deben ejecutar en segundo plano, sin la interacción inmediata con un usuario. | [Aplicaciones de demonio](scenario-daemon-overview.md) |
| [Código del dispositivo](#device-code) | Permite que los usuarios inicien sesión dispositivos con limitaciones de entrada como un Televisor inteligente, de dispositivo IoT o impresora. | [Aplicaciones de escritorio o móviles](scenario-desktop-acquire-token.md#command-line-tool-without-web-browser) |
| [Autenticación integrada de Windows](scenario-desktop-acquire-token.md#integrated-windows-authentication) | Permite que las aplicaciones en Azure AD o de dominio unido los equipos para adquirir un token de forma silenciosa (sin ninguna interacción de la interfaz de usuario del usuario).| [Aplicaciones de escritorio o móviles](scenario-desktop-acquire-token.md#integrated-windows-authentication) |
| [Nombre de usuario/contraseña](scenario-desktop-acquire-token.md#username--password) | Permite que una aplicación iniciar sesión en el usuario al controlar directamente la contraseña. No se recomienda este flujo. | [Aplicaciones de escritorio o móviles](scenario-desktop-acquire-token.md#username--password) | 

## <a name="interactive"></a>Interactivo
MSAL es compatible con la capacidad de preguntar interactivamente al usuario sus credenciales para iniciar sesión y obtener un token mediante esas credenciales.

![Flujo interactivo](media/msal-authentication-flows/interactive.png)

Para obtener más información sobre el uso de MSAL.NET interactivamente adquirir tokens en plataformas específicas, lea la información siguiente:
- [Xamarin Android](msal-net-xamarin-android-considerations.md)
- [Xamarin iOS](msal-net-xamarin-ios-considerations.md)
- [Plataforma universal de Windows](msal-net-uwp-considerations.md)

Para obtener más información sobre las llamadas interactivas en MSAL.js, lea [comportamiento en solicitudes interactivas MSAL.js](msal-js-prompt-behavior.md)

## <a name="implicit-grant"></a>Concesión implícita

MSAL es compatible con la [flujo de concesión implícita de OAuth 2](v2-oauth2-implicit-grant-flow.md), que permite que la aplicación obtuviera los tokens de la plataforma de identidad de Microsoft sin necesidad de realizar un servidor back-end intercambio de credenciales. De esta forma, la aplicación puede iniciar la sesión del usuario, mantenerla y recibir tokens para otras API web, y todo dentro del código de cliente de JavaScript.

![Flujo de concesión implícita](media/msal-authentication-flows/implicit-grant.svg)

Muchas aplicaciones web modernas se crean como aplicaciones de página única del lado cliente escritas con JavaScript o un marco de SPA como Angular, Vue.js y React.js. Estas aplicaciones se ejecutan en un explorador web y tienen las características de autenticación diferente que las aplicaciones web tradicionales de lado servidor. La plataforma Microsoft identity permite que las aplicaciones de página única para los usuarios inicien sesión y obtener tokens para tener acceso a servicios back-end o las API web mediante el flujo de concesión implícita. El flujo implícito permite que la aplicación obtener los tokens de identificador para representar al usuario autenticado y también tener acceso a los tokens es necesarios llamar a las API protegidas.

Este flujo de autenticación no incluye escenarios de aplicaciones con marcos de JavaScript de desarrollo multiplataforma como Electron y React-Native, puesto que requieren más funcionalidades para la interacción con las plataformas nativas.

## <a name="authorization-code"></a>Código de autorización
MSAL es compatible con la [concesión de código de autorización de OAuth 2](v2-oauth2-auth-code-flow.md), que puede utilizarse en aplicaciones que están instaladas en un dispositivo para tener acceso a recursos protegidos, como las API web. Esto le permite agregar iniciar sesión y tener acceso API a las aplicaciones de escritorio y móviles. 

Cuando los usuarios inician sesión en aplicaciones web (sitios web), la aplicación web recibe un código de autorización.  El código de autorización se canjea para adquirir un token para llamar a las API web. En ASP.NET o ASP.NET core web apps, el único objetivo del `AcquireTokenByAuthorizationCode` consiste en Agregar un token a la caché del token, por lo que, a continuación, se puede usar la aplicación (normalmente en los controladores) que llevaba un token para una API mediante `AcquireTokenSilent`.

![Flujo de código de autorización](media/msal-authentication-flows/authorization-code.png)

1. Solicita un código de autorización, que se puede canjear por un token de acceso.
2. Usa el token de acceso para llamar a una API web.

### <a name="considerations"></a>Consideraciones
- El código de autorización es utilizable una sola vez canjear un token. No intente adquirir un token varias veces con el mismo código de autorización (se prohíbe explícitamente mediante la especificación del protocolo estándar). Si se canjea el código varias veces intencionadamente o porque no es consciente de que un marco de trabajo también encarga de todo, obtendrá un error: `AADSTS70002: Error validating credentials. AADSTS54005: OAuth2 Authorization code was already redeemed, please retry with a new valid code or use an existing refresh token.`

- Si está escribiendo una aplicación ASP.NET/ASP.NET Core, esto puede ocurrir que si no indica el marco de trabajo ASP.NET/Core que ya has canjeado el código de autorización. Para ello, debe llamar a `context.HandleCodeRedemption()` método de la `AuthorizationCodeReceived` controlador de eventos.

- Evite el uso compartido el token de acceso con ASP.NET, lo que podría impedir el consentimiento incremental que se realice correctamente.  Para obtener más información, consulte [emitir 693 #](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues/693).

## <a name="on-behalf-of"></a>On-behalf-of

MSAL es compatible con la [flujo en nombre de la autenticación de OAuth 2](v2-oauth2-on-behalf-of-flow.md).  Este flujo se usa cuando una aplicación invoca un servicio o API web, que a su vez debe llamar a otro servicio o web API. La idea es propagar la identidad y los permisos del usuario delegado a través de la cadena de solicitud. Para que el servicio de nivel intermedio realice solicitudes autenticadas al servicio de bajada, debe proteger un token de acceso de la plataforma Microsoft identity, en nombre del usuario.

![Flujos en nombre de](media/msal-authentication-flows/on-behalf-of.png)

1. Adquiere un token de acceso para la API Web
2. Un cliente (Web, aplicación de escritorio, móvil, una página) llama a una API Web protegida, agregar el token de acceso como token de portador en el encabezado de autenticación de la solicitud HTTP. La API Web autentica al usuario.
3. Cuando el cliente llama a la API Web, la API Web solicita otro token en nombre de usuario.  
4. La API Web protegida usa este token para llamar a una API Web de nivel inferior en nombre de usuario.  La API Web también más adelante puede solicitar tokens para otras API de nivel inferior (pero todavía en nombre del mismo usuario).

## <a name="client-credentials"></a>Credenciales de cliente

MSAL es compatible con la [flujo de credenciales de cliente de OAuth 2](v2-oauth2-client-creds-grant-flow.md). Este flujo le permite acceder a recursos hospedados en web con la identidad de una aplicación. Este tipo de concesión se usa principalmente para las interacciones entre servidores que se deben ejecutar en segundo plano, sin la interacción inmediata con un usuario. Estos tipos de aplicaciones a menudo se conocen como demonios o cuentas de servicio. 

Las credenciales del cliente concesión flujo permite un servicio web (cliente confidencial) para usar sus propias credenciales, en lugar de suplantar a un usuario, para autenticarse al llamar a otro servicio web. En este escenario, el cliente es normalmente un servicio web de nivel intermedio, un servicio demonio o un sitio web. Para conseguir un mayor nivel de control, la plataforma de identidad de Microsoft también permite que el servicio que realiza la llamada use un certificado (en lugar de un secreto compartido) como credencial.

> [!NOTE]
> No está disponible en las plataformas móviles (UWP, Xamarin.iOS y Xamarin.Android), el flujo de cliente confidencial, ya que estos solo admiten aplicaciones de cliente público.  Las aplicaciones de cliente público no saben cómo demostrar la identidad de la aplicación para el proveedor de identidades. Se puede lograr una conexión segura en la aplicación web o API web back-end mediante la implementación de un certificado.

MSAL.NET admite dos tipos de credenciales de cliente. Estas credenciales de cliente deben estar registrada con Azure AD. Las credenciales se pasan en a los constructores de la aplicación cliente confidencial en el código.

### <a name="application-secrets"></a>Secretos de aplicación 

![Cliente confidencial con contraseña](media/msal-authentication-flows/confidential-client-password.png)

1. Adquiere un token con las credenciales del secreto y la contraseña de aplicación.
2. Usa el token para realizar solicitudes del recurso.

### <a name="certificates"></a>Certificados 

![Cliente confidencial con certificado](media/msal-authentication-flows/confidential-client-certificate.png)

1. Adquiere un token mediante credenciales de certificado.
2. Usa el token para realizar solicitudes del recurso.

Estas credenciales de cliente deben ser:
- Registrado con Azure AD.
- Pasa en la construcción de la aplicación cliente confidencial en el código.


## <a name="device-code"></a>Código del dispositivo
MSAL es compatible con la [OAuth 2 flujo de código de dispositivo](v2-oauth2-device-code.md), lo que permite que los usuarios inicien sesión dispositivos con limitaciones de entrada como un Televisor inteligente, de dispositivo IoT o impresora. Autenticación interactiva con Azure AD requiere un explorador web. El flujo de código de dispositivo permite al usuario usar otro dispositivo (por ejemplo, otro equipo o un teléfono móvil) para iniciar sesión interactivamente en el dispositivo o sistema operativo no proporciona un explorador Web.

Mediante el uso del flujo de código de dispositivo, la aplicación obtiene los tokens a través de un proceso de dos pasos diseñado especialmente para estos dispositivos/OS. Ejemplos de estas aplicaciones son aplicaciones que se ejecutan en dispositivos de iOT o herramientas de línea de comandos (CLI). 

![Flujo de código de dispositivo](media/msal-authentication-flows/device-code.png)

1. Cada vez que se requiere autenticación de usuario, la aplicación proporciona un código y pide al usuario que use otro dispositivo (por ejemplo, un smartphone conectado a internet) para navegar a una dirección URL (por ejemplo, https://microsoft.com/devicelogin), donde se pedirá al usuario que escriba el código. Que hace, la página web llevará al usuario a través de una experiencia de autenticación normal, incluidas las peticiones de consentimiento y autenticación multifactor si es necesario.

2. Tras una autenticación correcta, la aplicación de línea de comandos recibirá los tokens necesarios a través de un canal y utiliza para realizar las llamadas a API web que necesita.

### <a name="constraints"></a>Restricciones

- Flujo de código de dispositivo solo está disponible en las aplicaciones de cliente público.
- La entidad pasa al construir la aplicación de cliente público debe ser:
  - inquilinos (del formulario `https://login.microsoftonline.com/{tenant}/` donde `{tenant}` es el GUID que representa el identificador del inquilino o un dominio asociado con el inquilino.
  - o bien, las cuentas profesionales y educativas (`https://login.microsoftonline.com/organizations/`).
- Cuentas personales de Microsoft no son compatibles aún con el punto de conexión de v2.0 de Azure AD (no se puede usar el `/common` o `/consumers` inquilinos).

## <a name="integrated-windows-authentication"></a>Autenticación integrada de Windows
MSAL admite autenticación integrada de Windows (IWA) para escritorio o aplicaciones móviles que se ejecutan en un dominio o Azure AD Unidos a un equipo de Windows. Usa IWA, estas aplicaciones pueden adquirir un token de forma silenciosa (sin ninguna interacción de la interfaz de usuario del usuario). 

![Autenticación integrada de Windows](media/msal-authentication-flows/integrated-windows-authentication.png)

1. Adquiere un token mediante la autenticación integrada de Windows.
2. Usa el token para realizar solicitudes del recurso.

### <a name="constraints"></a>Restricciones

Es compatible con IWA **federados** solo a los usuarios.  Los usuarios crean en Active Directory y respaldada por Azure Active Directory. Los usuarios creados directamente en Azure AD, sin el respaldo de AD (**administrada** usuarios) no se puede usar este flujo de autenticación. Esta limitación no afecta a la [flujo de usuario y contraseña](#usernamepassword).

IWA es para aplicaciones escritas para las plataformas .NET Framework, .NET Core y plataforma Universal de Windows.

IWA no omitir MFA (autenticación multifactor). Si se ha configurado MFA, IWA puede producir un error si un desafío de MFA es necesario porque MFA necesita interacción del usuario. Éste es complicado. IWA es no interactivo, pero requiere la autorización de dos fases (2FA) interactividad del usuario. No se controlan cuando el proveedor de identidades solicita 2FA para realizarse, hace el Administrador de inquilinos. En nuestras observaciones, 2FA es necesaria cuando inicie sesión desde otro país diferente, cuando no está conectado a través de VPN a una red corporativa y, a veces, incluso cuando se conecta a través de VPN. No espere que un conjunto de reglas determinista, Azure Active Directory usa inteligencia artificial para aprender continuamente si se requiere 2FA. Se debe reservar a un símbolo del sistema del usuario (https://aka.ms/msal-net-interactive) si se produce un error de IWA.

La entidad pasa al construir la aplicación de cliente público debe ser:
- inquilinos (del formulario `https://login.microsoftonline.com/{tenant}/` donde `tenant` es el guid que representa el identificador del inquilino o un dominio asociado con el inquilino.
- para cualquier trabajo cuentas profesionales y educativas (`https://login.microsoftonline.com/organizations/`). No se admiten cuentas personales de Microsoft (no se puede usar `/common` o `/consumers` inquilinos).

Dado que IWA es un flujo silencioso:
- el usuario de la aplicación debe haya consentido previamente para usar la aplicación. 
- o bien, el Administrador de inquilinos debe haya consentido previamente a todos los usuarios en el inquilino para usar la aplicación.
- Esto significa que:
    - ya sea como desarrolladora de haber presionado el **Grant** en el portal de Azure por sí mismo, botón 
    - o un administrador de inquilinos ha presionado el **conceder o revocar el consentimiento del Administrador de {dominio del inquilino}** situado en la **permisos de API** ficha del registro para la aplicación (consulte [agregar permisos para las API de web Access](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-web-apis))
    - o bien, ha proporcionado a los usuarios den su consentimiento a la aplicación (consulte [solicitar el consentimiento de usuario individual](v2-permissions-and-consent.md#requesting-individual-user-consent))
    - o bien, ha proporcionado una forma para el Administrador de inquilinos dé su consentimiento para la aplicación (consulte [consentimiento del administrador](v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant))

El flujo IWA está habilitado para el escritorio. NET, .NET Core y aplicaciones de la plataforma Universal de Windows. En .NET Core solo la sobrecarga que toma el nombre de usuario está disponible como la plataforma .NET Core no puede pedir el nombre de usuario para el sistema operativo.
  
Para obtener más información sobre el consentimiento, consulte [v2.0 permisos y consentimiento](v2-permissions-and-consent.md).

## <a name="usernamepassword"></a>Nombre de usuario/contraseña 
MSAL es compatible con la [concesión de credenciales de contraseña de propietario de recurso OAuth 2](v2-oauth-ropc.md), que permite que una aplicación iniciar sesión en el usuario al controlar directamente la contraseña. En su aplicación de escritorio, puede usar el flujo de usuario y contraseña para adquirir un token de forma silenciosa. Ninguna interfaz de usuario es necesario cuando se usa la aplicación.

![Flujo de usuario y contraseña](media/msal-authentication-flows/username-password.png)

1. Adquiere un token mediante el envío el nombre de usuario y la contraseña para el proveedor de identidades.
2. Llama a una API web mediante el token.

> [!WARNING]
> Este flujo está **no recomienda** porque requiere un alto grado de exposición de confianza y el usuario.  Solo se debe usar este flujo cuando no se puede usar flujos de otros y la más seguros. Para obtener más información acerca de este problema, consulte [en este artículo](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/). 

El flujo preferido para adquirir un token de forma silenciosa en equipos unidos a un dominio de Windows es [autenticación integrada de Windows](#integrated-windows-authentication). En caso contrario, también puede usar [flujo de código de dispositivo](#device-code)

Aunque esto es útil en algunos casos (escenarios de DevOps), si desea usar el nombre de usuario y contraseña en escenarios interactivos donde debe proporcionar su propia interfaz de usuario, se debe considerar realmente sobre cómo mover fuera de él. Mediante el uso de nombre de usuario/contraseña, se están dando vertical varias cosas:
- principales inquilinos de identidad moderna: contraseña obtiene pesca, reproducir. Dado que tenemos este concepto de un secreto compartido que se puede interceptar.
Esto es incompatible con sin contraseña.
- los usuarios que necesitan para realizar la MFA no podrán iniciar sesión (ya no hay ninguna interacción)
- Los usuarios no podrán inicio de sesión único

### <a name="constraints"></a>Restricciones

Aparte de la [las restricciones de la autenticación integrada de Windows](#integrated-windows-authentication), también se aplican las siguientes restricciones:

- El flujo de usuario y contraseña no es compatible con el acceso condicional y Multi-factor authentication: Como consecuencia, si la aplicación se ejecuta en un inquilino de Azure AD donde el Administrador de inquilinos requiere autenticación multifactor, no se puede usar este flujo. Muchas organizaciones hacer eso.
- Solo funciona con trabajo cuentas profesionales y educativas (no MSA)
- El flujo está disponible en el escritorio de .NET y .NET core, pero no en la plataforma Universal de Windows.

### <a name="azure-ad-b2c-specifics"></a>Obtener información específica de Azure AD B2C

Para obtener más información sobre el uso de MSAL.NET y Azure AD B2C, lea [utilizando ROPC con B2C de Azure AD (MSAL.NET)](msal-net-aad-b2c-considerations.md#resource-owner-password-credentials-ropc-with-azure-ad-b2c).
