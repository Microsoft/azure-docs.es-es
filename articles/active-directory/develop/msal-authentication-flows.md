---
title: Flujos de autenticación de MSAL | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre los flujos y concesiones de autenticación utilizados por la biblioteca de autenticación de Microsoft (MSAL).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/08/2020
ms.author: marsma
ms.reviewer: saeeda
ms.openlocfilehash: 4a902ed53e92cd073d81626e80bdb3c8629ad072
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89437877"
---
# <a name="authentication-flows"></a>Flujos de autenticación

La biblioteca de autenticación de Microsoft (MSAL) admite varios flujos de autenticación para su uso en distintos escenarios de aplicaciones.

| Flujo | Descripción | Se usa en |
|--|--|--|
| [Código de autorización](#authorization-code) | Se usa en aplicaciones que están instaladas en un dispositivo para obtener acceso a recursos protegidos, como las API web. Permite agregar acceso de inicio de sesión y API a las aplicaciones de escritorio y móviles. | [Aplicaciones de escritorio](scenario-desktop-overview.md), [aplicaciones móviles](scenario-mobile-overview.md), [aplicaciones web](scenario-web-app-call-api-overview.md) |
| [Credenciales de cliente](#client-credentials) | Permite acceder a los recursos hospedados en la Web mediante la identidad de una aplicación. Se usa principalmente para las interacciones entre servidores que se deben ejecutar en segundo plano, sin la interacción inmediata con un usuario. | [Aplicaciones de demonio](scenario-daemon-overview.md) |
| [Código del dispositivo](#device-code) | Permite a los usuarios iniciar sesión en dispositivos con limitaciones de entrada, como un televisor inteligente, dispositivo IoT o impresora. | [Aplicaciones de escritorio o móviles](scenario-desktop-acquire-token.md#command-line-tool-without-a-web-browser) |
| [Concesión implícita](#implicit-grant) | Permite a la aplicación obtener tokens sin tener que realizar un intercambio de credenciales de servidor back-end. Permite a la aplicación iniciar la sesión del usuario, mantenerla y recibir tokens para otras API web, todo dentro del código de cliente de JavaScript. | [Aplicación de página única (SPA)](scenario-spa-overview.md) |
| [En nombre de](#on-behalf-of) | Una aplicación invoca un servicio o API web, que, a su vez, debe llamar a otro servicio o API web. La idea es propagar la identidad y los permisos del usuario delegado a través de la cadena de solicitud. | [API web](scenario-web-api-call-api-overview.md) |
| [Nombre de usuario y contraseña](#usernamepassword) | Permite a una aplicación iniciar la sesión del usuario al controlar directamente la contraseña. Este flujo no es recomendable. | [Aplicaciones de escritorio o móviles](scenario-desktop-acquire-token.md#username-and-password) |
| [Autenticación integrada de Windows](#integrated-windows-authentication) | Permite a los equipos de aplicaciones en equipos unidos a un dominio o a Azure Active Directory (Azure AD) adquirir un token de manera silenciosa (sin ninguna interacción de la interfaz de usuario del usuario). | [Aplicaciones de escritorio o móviles](scenario-desktop-acquire-token.md#integrated-windows-authentication) |

## <a name="how-each-flow-emits-tokens-and-codes"></a>Cómo emite cada flujo los tokens y los códigos

En función de cómo se compile la aplicación cliente, puede usar uno o varios de los flujos de autenticación admitidos por la Plataforma de identidad de Microsoft. Estos flujos pueden generar varios tipos de tokens, así como códigos de autorización, y requieren distintos tokens para que funcionen.

| Flujo                                                                               | Requiere            | ID_token | de la aplicación Twitter | Token de actualización | código de autorización |
|------------------------------------------------------------------------------------|:-------------------:|:--------:|:------------:|:-------------:|:------------------:|
| [Flujo de código de autorización](v2-oauth2-auth-code-flow.md)                             |                     | x        | x            | x             | x                  |
| [Credenciales de cliente](v2-oauth2-client-creds-grant-flow.md)                         |                     |          | x (solo aplicación) |               |                    |
| [Flujo de código de dispositivo](v2-oauth2-device-code.md)                                       |                     | x        | x            | x             |                    |
| [Flujo implícito](v2-oauth2-implicit-grant-flow.md)                                  |                     | x        | x            |               |                    |
| [Flujo en nombre de](v2-oauth2-on-behalf-of-flow.md)                                | de la aplicación Twitter        | x        | x            | x             |                    |
| [Nombre de usuario/contraseña](v2-oauth-ropc.md) (ROPC)                                       | Nombre de usuario y contraseña | x        | x            | x             |                    |
| [Flujo de OIDC híbrido](v2-protocols-oidc.md#protocol-diagram-access-token-acquisition) |                     | x        |              |               | x                  |
| [Redención de token de actualización](v2-oauth2-auth-code-flow.md#refresh-the-access-token)   | Token de actualización       | x        | x            | x             |                    |

### <a name="interactive-and-non-interactive-authentication"></a>Autenticación interactiva y no interactiva

Varios de estos flujos admiten la adquisición interactiva y no interactiva de tokens.

  - La autenticación **interactiva** significa que se puede solicitar la intervención del usuario. Por ejemplo, se le puede pedir al usuario que inicie sesión, realice autenticación multifactor (MFA) o conceda consentimiento adicional a los recursos.
  - La autenticación **no interactiva** o *silenciosa* intenta adquirir un token de manera que el servidor de inicio de sesión *no pueda* solicitar información adicional al usuario.

La aplicación basada en MSAL primero debe intentar adquirir un token de manera *silenciosa* y luego de manera interactiva solo si se produce un error en el método no interactivo. Para más información sobre este patrón, consulte [Adquisición y almacenamiento en caché de tokens con la biblioteca de autenticación de Microsoft (MSAL)](msal-acquire-cache-tokens.md).

## <a name="authorization-code"></a>Código de autorización

La [concesión de un código de autorización de OAuth 2](v2-oauth2-auth-code-flow.md) se puede usar en aplicaciones que se instalan en un dispositivo para obtener acceso a recursos protegidos, como las API web. Esto permite agregar acceso de inicio de sesión y API a las aplicaciones de escritorio y móviles.

Cuando los usuarios inician sesión en aplicaciones web (sitios web), la aplicación web recibe un código de autorización. El código de autorización se canjea para adquirir un token para llamar a las API web.

![Diagrama del flujo de código de autorización](media/msal-authentication-flows/authorization-code.png)

En el diagrama anterior, la aplicación hace lo siguiente:

1. Solicita un código de autorización, que se canjea por un token de acceso.
2. Usa el token de acceso para llamar a una API web.

### <a name="considerations"></a>Consideraciones

- Puede usar el código de autorización una sola vez para canjear un token. No intente adquirir un token varias veces con el mismo código de autorización, porque la especificación del protocolo estándar lo prohíbe explícitamente. Si canjea el código varias veces de manera intencionada o porque no es consciente de que un marco también lo hace, obtendrá el error siguiente:

    `AADSTS70002: Error validating credentials. AADSTS54005: OAuth2 Authorization code was already redeemed, please retry with a new valid code or use an existing refresh token.`

## <a name="client-credentials"></a>Credenciales de cliente

Es [flujo de credenciales de cliente de OAuth 2](v2-oauth2-client-creds-grant-flow.md) permite acceder a los recursos hospedados en la Web mediante la identidad de una aplicación. Este tipo de concesión se usa principalmente para las interacciones entre servidores que se deben ejecutar en segundo plano, sin la interacción inmediata con un usuario. Estos tipos de aplicaciones suelen denominarse demonios o cuentas de servicio.

El flujo de concesión de credenciales de cliente permite que un servicio web (cliente confidencial) use sus propias credenciales para autenticarse al llamar a otro servicio web, en lugar de suplantar a un usuario. En este escenario, el cliente suele ser un servicio web de nivel intermedio, un servicio demonio o un sitio web. Para conseguir un mayor nivel de control, la plataforma de identidad de Microsoft también permite que el servicio que realiza la llamada use un certificado (en lugar de un secreto compartido) como credencial.

> [!NOTE]
> El flujo de cliente confidencial no está disponible en plataformas móviles como UWP, Xamarin.iOS y Xamarin.Android, porque solo admiten aplicaciones de cliente públicas. Las aplicaciones de cliente públicas no saben cómo demostrar la identidad de la aplicación al proveedor de identidades. Una conexión segura se puede lograr en los servidores back-end de aplicación web o API web mediante la implementación de un certificado.

### <a name="application-secrets"></a>Secretos de aplicación

![Diagrama de cliente confidencial con contraseña](media/msal-authentication-flows/confidential-client-password.png)

En el diagrama anterior, la aplicación hace lo siguiente:

1. Adquiere un token al usar un secreto de aplicación o credenciales de contraseña.
2. Usa el token para hacer solicitudes al recurso.

### <a name="certificates"></a>Certificados

![Diagrama de cliente confidencial con certificado](media/msal-authentication-flows/confidential-client-certificate.png)

En el diagrama anterior, la aplicación hace lo siguiente:

1. Adquiere un token mediante el uso de credenciales de certificado.
2. Usa el token para hacer solicitudes al recurso.

Estas credenciales de cliente deben:

- Registrarse en Azure AD.
- Se pasa al construir el objeto de aplicación cliente confidencial en el código.

## <a name="device-code"></a>Código del dispositivo

El [flujo de código del dispositivo de OAuth 2](v2-oauth2-device-code.md) permite que los usuarios inicien sesión en dispositivos con restricción de entrada como televisores inteligentes, dispositivos IoT e impresoras. La autenticación interactiva con Azure AD requiere un explorador web. Cuando el dispositivo o el sistema operativo no proporciona un explorador web, el flujo de código del dispositivo permite que el usuario utilice otro dispositivo, como un equipo o un teléfono móvil para iniciar sesión de manera interactiva.

Al usar el flujo de código del dispositivo, la aplicación obtiene los tokens a través de un proceso de dos pasos diseñado para estos dispositivos y sistemas operativos. Entre algunos ejemplos de estas aplicaciones se incluye aquellas que se ejecutan en dispositivos IoT o herramientas de interfaz de la línea de comandos (CLI).

![Diagrama del flujo de código de dispositivo](media/msal-authentication-flows/device-code.png)

En el diagrama anterior:

1. Cada vez que se requiera autenticación del usuario, la aplicación proporciona un código y pide al usuario que use otro dispositivo, como un smartphone conectado a Internet, para ir a una dirección URL (por ejemplo, `https://microsoft.com/devicelogin`). Luego, se le pide al usuario que especifique el código y el usuario pasa por una experiencia de autenticación normal, como peticiones de consentimiento y [autenticación multifactor](../authentication/concept-mfa-howitworks.md), si es necesario.
1. Tras una autenticación correcta, la aplicación de línea de comandos recibe los tokens necesarios a través de un canal posterior y los usa para realizar las llamadas a la API web que necesita.

### <a name="constraints"></a>Restricciones

- El flujo de código del dispositivo solo está disponible en las aplicaciones cliente públicas.
- La autoridad que se pasa al construir la aplicación cliente pública debe ser una de las siguientes:
  - Con inquilino, con el formato `https://login.microsoftonline.com/{tenant}/,`, donde `{tenant}` es el GUID que representa el identificador del inquilino o un dominio asociado con el inquilino.
  - Para las cuentas profesionales y educativas con el formato `https://login.microsoftonline.com/organizations/`.

## <a name="implicit-grant"></a>Concesión implícita

El flujo de [concesión implícita de OAuth 2](v2-oauth2-implicit-grant-flow.md) permite a la aplicación obtener tokens de la plataforma de identidad de Microsoft sin tener que realizar un intercambio de credenciales del servidor back-end. Este flujo permite que la aplicación inicie la sesión del usuario, la mantenga y reciba tokens para otras API web, todo dentro del código de cliente de JavaScript.

![Diagrama del flujo de concesión implícito](media/msal-authentication-flows/implicit-grant.svg)

Muchas aplicaciones web modernas se crean como aplicaciones cliente, aplicaciones de una sola página (SPA) escritas con JavaScript o un marco de SPA, como Angular, Vue.js y React.js. Estas aplicaciones se ejecutan en un explorador web y tienen características de autenticación diferentes que las aplicaciones web de servidor tradicionales. La plataforma de identidad de Microsoft permite que las aplicaciones de página única inicien la sesión de los usuarios y obtengan tokens para acceder a los servicios back-end o API web, mediante el flujo de concesión implícito. El flujo implícito permite a la aplicación obtener los tokens de identificador para representar al usuario autenticado y también acceder a los tokens que se necesitan para llamar a las API protegidas.

Este flujo de autenticación no incluye escenarios de aplicaciones que usan marcos de JavaScript multiplataforma, como Electron o React-Native, ya que requieren más funcionalidades para la interacción con las plataformas nativas.

Los tokens emitidos a través del modo de flujo implícito tienen una **limitación de longitud**, porque son devueltos al explorador por la dirección URL (donde `response_mode` es `query` o `fragment`). Algunos exploradores limitan la longitud de la dirección URL en la barra del explorador y generan un error cuando es demasiado larga. Por lo tanto, estos tokens de flujo implícitos no contienen notificaciones `groups` o `wids`.

## <a name="on-behalf-of"></a>En nombre de

El flujo de [autenticación delegada de OAuth 2](v2-oauth2-on-behalf-of-flow.md) se usa cuando una aplicación invoca un servicio o una API web que, a su vez, necesita llamar a otro servicio o a otra API web. La idea es propagar la identidad y los permisos del usuario delegado a través de la cadena de solicitud. Para que el servicio de nivel intermedio realice solicitudes autenticadas al servicio de bajada, debe proteger un token de acceso de la plataforma de identidad de Microsoft *en nombre del* usuario.

![Diagrama del flujo "en nombre de"](media/msal-authentication-flows/on-behalf-of.png)

En el diagrama anterior:

1. La aplicación adquiere un token de acceso para la API web.
2. Un cliente (web, aplicación de escritorio, aplicación móvil o una página) llama a una API web protegida, agregando el token de acceso como token de portador en el encabezado de autenticación de la solicitud HTTP. La API web autentica al usuario.
3. Cuando el cliente llama a la API web, esta solicita otro token en nombre del usuario.
4. La API web protegida usa este token para llamar a una API web descendente en nombre del usuario. La API web también puede solicitar tokens más tarde para otras API descendentes (pero todavía en nombre del mismo usuario).

## <a name="usernamepassword"></a>Nombre de usuario y contraseña

La concesión de [credenciales de contraseña de propietario de recurso de OAuth 2](v2-oauth-ropc.md) (ROPC) permite que una aplicación inicie la sesión del usuario al controlar directamente la contraseña. En la aplicación de escritorio, puede usar el flujo de usuario y contraseña para adquirir un token de forma silenciosa. No se requiere ninguna interfaz de usuario cuando se usa la aplicación.

![Diagrama del flujo de usuario y contraseña](media/msal-authentication-flows/username-password.png)

En el diagrama anterior, la aplicación hace lo siguiente:

1. Adquiere un token al enviar el nombre de usuario y contraseña al proveedor de identidades.
2. Llama a una API web mediante el token.

> [!WARNING]
> Este flujo no es recomendable. Requiere un alto grado de confianza y exposición de credencial. Este flujo lo debe usar *solo* cuando no se pueden usar flujos más seguros. Para más información, consulte [What's the solution to the growing problem of passwords?](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/) (¿Cuál es la solución al creciente problema de las contraseñas?).

El flujo preferido para adquirir un token de forma silenciosa en equipos unidos a un dominio de Windows es la [autenticación integrada de Windows](#integrated-windows-authentication). En otros casos, use el [flujo de código del dispositivo](#device-code).

Si bien el flujo de nombre de usuario/contraseña puede ser útil en algunos escenarios como DevOps, evítelo si quiere usar el nombre de usuario/contraseña en escenarios interactivos donde puede proporcionar su propia interfaz de usuario.

Mediante el uso de nombre de usuario y contraseña:

- Los usuarios que necesitan realizar la autenticación multifactor no podrán iniciar sesión, porque no hay ninguna interacción.
- Los usuarios no podrán realizar el inicio de sesión único.

### <a name="constraints"></a>Restricciones

Aparte de la [las restricciones de la autenticación integrada de Windows](#integrated-windows-authentication), también se aplican las siguientes restricciones:

- El flujo de usuario y contraseña no es compatible con el acceso condicional ni la autenticación multifactor. Como consecuencia, si la aplicación se ejecuta en un inquilino de Azure AD donde el administrador de inquilinos requiere la autenticación multifactor, no se puede usar este flujo. Muchas organizaciones hacen eso.
- ROPC solo funciona para las cuentas profesionales y educativas. No se puede usar ROPC para las cuentas de Microsoft (MSA).
- El flujo está disponible en el escritorio de .NET y .NET Core, pero no en la Plataforma universal de Windows.
- En Azure AD B2C, el flujo ROPC solo funciona para las cuentas locales. Para Uso sobre ROPC en MSAL.NET y Azure AD B2C, consulte [Credenciales de contraseña de propietario de recursos con Azure AD B2C](msal-net-aad-b2c-considerations.md#resource-owner-password-credentials-ropc).

## <a name="integrated-windows-authentication"></a>Autenticación integrada de Windows

MSAL admite la Autenticación integrada de Windows (IWA) para aplicaciones de escritorio y aplicaciones móviles que se ejecutan en un equipo Windows unido a un dominio o a Azure AD. Mediante IWA, estas aplicaciones pueden adquirir un token de forma silenciosa, sin necesitar ninguna interacción de la interfaz de usuario del usuario.

![Diagrama de la autenticación integrada de Windows](media/msal-authentication-flows/integrated-windows-authentication.png)

En el diagrama anterior, la aplicación hace lo siguiente:

1. Adquiere un token mediante el uso de la autenticación integrada de Windows.
2. Usa el token para hacer solicitudes al recurso.

### <a name="constraints"></a>Restricciones

La Autenticación integrada de Windows (IWA) admite *solo* usuarios federados, los usuarios creados en Active Directory y respaldados por Azure AD. Los usuarios creados directamente en Azure AD sin el respaldo de Active Directory (usuarios administrados) no pueden usar este flujo de autenticación. Esta limitación no afecta al [flujo de usuario y contraseña](#usernamepassword).

IWA es para las aplicaciones de .NET Framework, .NET Core y de Plataforma universal de Windows.

IWA no omite la autenticación multifactor. Si se ha configurado la autenticación multifactor, podría producirse un error en IWA si se requiere un desafío de autenticación multifactor. La autenticación multifactor requiere la interacción del usuario.

No se puede controlar cuándo el proveedor de identidades solicita que se realice la autenticación en dos fases. Esto lo hace el administrador de inquilinos. Normalmente, la autenticación en dos fases se requiere al iniciar sesión desde un país o región diferente, cuando no está conectado a través de una VPN a una red corporativa y, a veces, incluso cuando se conecta a través de una VPN. Azure AD usa la inteligencia artificial para aprender continuamente si se requiere la autenticación en dos fases. Si IWA produce un error, debe revertir a una [solicitud interactiva de usuario](#interactive-and-non-interactive-authentication).

La autoridad que se pasa al construir la aplicación cliente pública debe ser una de las siguientes:

- Con inquilino, con el formato `https://login.microsoftonline.com/{tenant}/,`, donde `{tenant}` es el GUID que representa el identificador del inquilino o un dominio asociado con el inquilino.
- Para las cuentas profesionales y educativas (`https://login.microsoftonline.com/organizations/`). No se admiten cuentas de Microsoft personales (MSA); no se pueden usar los inquilinos `/common` o `/consumers`.

Dado que IWA es un flujo silencioso, debe cumplirse uno de los siguientes requisitos:

- El usuario de la aplicación debe haber consentido anteriormente para usar la aplicación.
- El administrador de inquilinos debe haber consentido anteriormente a todos los usuarios en el inquilino para que usen la aplicación.

Esto significa que una de las siguientes condiciones es verdad:

- Usted, como desarrollador, seleccionó **Conceder** en Azure Portal para usted mismo.
- Un administrador de inquilinos ha seleccionado la pestaña **Conceder o revocar consentimiento del administrador para {dominio del inquilino}** en la pestaña **Permisos de API** del registro de la aplicación en Azure Portal (consulte [Adición de permisos para acceder a las API web](quickstart-configure-app-access-web-apis.md#add-permissions-to-access-your-web-api)).
- Proporcionó una manera para que los usuarios den su consentimiento a la aplicación. Consulte [Solicitud de consentimiento de usuario individual](v2-permissions-and-consent.md#requesting-individual-user-consent).
- Proporcionó una forma para que el administrador de inquilinos dé su consentimiento para la aplicación. Consulte [Consentimiento de administrador](v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant).

El flujo de IWA está habilitado para el escritorio. NET, .NET Core y aplicaciones de la Plataforma universal de Windows. En .NET Core, debe proporcionar el nombre de usuario a IWA, porque .NET Core no puede obtener los nombres de usuario del sistema operativo.

Para más información sobre el consentimiento, consulte [Permiso y consentimiento de v2.0](v2-permissions-and-consent.md).

## <a name="next-steps"></a>Pasos siguientes

Ahora que revisó los flujos de autenticación admitidos por la biblioteca de autenticación de Microsoft (MSAL), obtenga información sobre cómo adquirir y almacenar en caché los tokens que se usan en estos flujos:

[Adquisición y almacenamiento en caché de tokens con la biblioteca de autenticación de Microsoft (MSAL)](msal-acquire-cache-tokens.md)
