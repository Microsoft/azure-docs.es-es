---
title: Adquisición y almacenamiento en caché de tokens con la biblioteca de autenticación de Microsoft (MSAL) | Azure
titleSuffix: Microsoft identity platform
description: Más información sobre la adquisición y el almacenamiento en caché de tokens mediante MSAL.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/28/2020
ms.author: marsma
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 47af4015fa5c6d9a73ee597146890a29b4b9ef9d
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119902"
---
# <a name="acquire-and-cache-tokens-using-the-microsoft-authentication-library-msal"></a>Adquisición y almacenamiento en caché de tokens con la biblioteca de autenticación de Microsoft (MSAL)

Los [tokens de acceso](access-tokens.md) permiten a los clientes llamar a las API web protegidas por Azure de forma segura. Hay muchas maneras de adquirir un token mediante la biblioteca de autenticación de Microsoft (MSAL). En algunos casos, se requiere la interacción del usuario a través de un explorador web, mientras que en otros no es necesario. En general, la manera de adquirir un token depende de si la aplicación es una aplicación cliente pública (por ejemplo, una aplicación de escritorio o móvil) o una aplicación cliente confidencial (por ejemplo, una aplicación web, API web o de demonio).

Después de adquirir un token, MSAL lo almacena en caché. El código de su aplicación debería intentar primero obtener un token en modo silencioso desde la caché antes de intentarlo por otros medios.

También puede borrar la memoria caché de tokens, lo cual se consigue mediante la eliminación de las cuentas de la memoria caché. No obstante, este paso no elimina la cookie de sesión del explorador.

## <a name="scopes-when-acquiring-tokens"></a>Ámbitos al adquirir tokens

Los [ámbitos](v2-permissions-and-consent.md) son los permisos que una API web expone para que las aplicaciones cliente soliciten acceso a ellos. Las aplicaciones cliente solicitan el consentimiento del usuario para estos ámbitos al realizar solicitudes de autenticación para obtener los tokens de acceso a las API web. MSAL le permite obtener tokens para acceder a Azure AD para desarrolladores (v1.0) y a las API de la plataforma de identidad de Microsoft (v2.0). El protocolo de la versión v2.0 usa ámbitos en lugar de recursos en las solicitudes. Para más información, lea [la comparación entre las versiones 1.0 y 2.0](../azuread-dev/azure-ad-endpoint-comparison.md). Según la configuración de la API web de la versión de token que acepta, el punto de conexión v2.0 devolverá el token de acceso a MSAL.

Algunos de los métodos de adquisición de tokens de MSAL requieren un parámetro `scopes`. El parámetro `scopes` consiste en una lista de cadenas que declaran los permisos deseados y los recursos solicitados. Algunos ejemplos de ámbitos bien conocidos son los [permisos de Microsoft Graph](/graph/permissions-reference).

En MSAL también se puede acceder a los recursos de v1.0. Para más información, consulte el artículo sobre los [ámbitos para una aplicación que acepta tokens v1.0](msal-v1-app-scopes.md).

### <a name="request-scopes-for-a-web-api"></a>Solicitud de ámbitos para una API web

Si su aplicación necesita solicitar un token de acceso con permisos específicos para una API de recursos, transfiera los ámbitos que contienen el URI del identificador de la aplicación con el formato `<app ID URI>/<scope>`.

Valores de ámbito de ejemplo para distintos recursos:

- Microsoft Graph API: `https://graph.microsoft.com/User.Read`
- API web personalizada: `api://11111111-1111-1111-1111-111111111111/api.read`

El formato del valor de ámbito varía en función del recurso (la API) que reciba el token de acceso y los valores de notificación `aud` que acepte.

En el caso de Microsoft Graph únicamente, el ámbito `user.read` se asigna a `https://graph.microsoft.com/User.Read`, y ambos formatos de ámbito se pueden utilizar indistintamente.

Determinadas API web como la de Azure Resource Manager (https://management.core.windows.net/) ) esperan que haya una barra diagonal ("/") en la notificación de "audience" (`aud` ) del token de acceso. En este caso, pase el ámbito como `https://management.core.windows.net//user_impersonation`, incluida la barra diagonal doble ("//").

Puede que otras API requieran que *no scheme or host* se incluya en el valor de ámbito y que solo se espere el identificador de la aplicación (GUID) y el nombre de ámbito. Por ejemplo:

`11111111-1111-1111-1111-111111111111/api.read`

> [!TIP]
> Si el recurso de bajada no está bajo su control, es posible que tenga que probar distintos formatos de valor de ámbito (por ejemplo, con/sin esquema y host) si recibe el error `401` u otros errores al pasar el token de acceso al recurso.

### <a name="request-dynamic-scopes-for-incremental-consent"></a>Solicitud de ámbitos dinámicos para el consentimiento incremental

A medida que cambien las características proporcionadas por la aplicación o sus requisitos, puede solicitar permisos adicionales mediante el parámetro de ámbito. Por ejemplo, *ámbitos dinámicos* permite a los usuarios dar su consentimiento incremental en relación con los ámbitos.

De este modo, puede permitir que los usuarios inicien sesión, pero denegarles inicialmente el acceso a los recursos. Más adelante, puede darles la posibilidad de ver su calendario al solicitar el ámbito de calendario en el método de adquisición de tokens y obtener el consentimiento del usuario para hacerlo. Por ejemplo, mediante la solicitud de los ámbitos `https://graph.microsoft.com/User.Read` y `https://graph.microsoft.com/Calendar.Read`.

## <a name="acquiring-tokens-silently-from-the-cache"></a>Adquisición de tokens de forma automática (desde la memoria caché)

MSAL mantiene una memoria caché de tokens, o dos en el caso de las aplicaciones cliente confidenciales, y almacena en caché un token después de haberlo adquirido. En muchos casos, al intentar obtener de forma automática un token adquirirá otro token con más ámbitos en función de un token de la caché. También puede actualizar un token cuando va a expirar (ya que la caché de tokens también contiene un token de actualización).

### <a name="recommended-call-pattern-for-public-client-applications"></a>Patrón de llamada recomendado para aplicaciones cliente públicas

El código de la aplicación debería intentar primero obtener un token en modo silencioso desde la caché. Si la llamada al método devuelve un error o una excepción que indica que "se requiere una interfaz de usuario", intente adquirir un token por otros medios.

Sin embargo, hay dos flujos en los que **no debería** intentar adquirir un token en modo silencioso:

- El [flujo de credenciales del cliente](msal-authentication-flows.md#client-credentials), que no usa la caché de tokens de usuario sino una caché de tokens de aplicación. Este método comprueba la caché de tokens de aplicación antes de enviar una solicitud al servicio de token de seguridad (STS).
- El [flujo de código de autorización](msal-authentication-flows.md#authorization-code) de las aplicaciones web, ya que canjea un código que la aplicación obtuvo al iniciar sesión el usuario y dar el consentimiento a más ámbitos. Puesto que un código se pasa como un parámetro, y no como una cuenta, el método no puede hacer búsquedas en la caché antes de canjear el código, lo cual implica que se llame al servicio.

### <a name="recommended-call-pattern-in-web-apps-using-the-authorization-code-flow"></a>Patrón de llamada recomendado en aplicaciones web mediante el flujo de código de autorización

Para las aplicaciones web que usan el [flujo de código de autorización de OpenID Connect](v2-protocols-oidc.md), el patrón recomendado en los controladores consiste en:

- Crear una instancia de una aplicación cliente confidencial con una caché de tokens con serialización personalizada.
- Adquirir el token mediante el flujo de código de autorización

## <a name="acquiring-tokens"></a>Adquisición de tokens

Por lo general, el método de adquisición de un token depende de si es una aplicación cliente pública o confidencial.

### <a name="public-client-applications"></a>Aplicaciones cliente públicas

Para las aplicaciones cliente públicas (aplicación de escritorio o móvil):

- A menudo adquiere tokens de forma interactiva, haciendo que el usuario inicie sesión mediante una interfaz de usuario o una ventana emergente.
- Puede [obtener un token de forma automática](msal-authentication-flows.md#integrated-windows-authentication) mediante la autenticación integrada (IWA/Kerberos) si la aplicación de escritorio se ejecuta en un equipo de Windows unido a un dominio o a Azure.
- Puede [obtener un token con un nombre de usuario y contraseña](msal-authentication-flows.md#usernamepassword) en las aplicaciones cliente de escritorio de .NET Framework, aunque no es recomendable. No utilice un nombre de usuario y contraseña en aplicaciones cliente confidenciales.
- Puede adquirir un token mediante el [flujo de código de dispositivo](msal-authentication-flows.md#device-code) en aplicaciones que se ejecutan en dispositivos que no tienen un explorador web. Se proporciona al usuario una dirección URL y un código y, a continuación, el usuario va a un explorador web de otro dispositivo y escribe el código e inicia sesión. Después, Azure AD envía un token de nuevo al dispositivo sin explorador.

### <a name="confidential-client-applications"></a>Aplicaciones cliente confidenciales

En el caso de las aplicaciones cliente confidenciales (por ejemplo, una aplicación web, API web o de demonio, como un servicio de Windows), puede:

- Adquiere tokens **para la aplicación en sí** y no para un usuario mediante el [flujo de credenciales de cliente](msal-authentication-flows.md#client-credentials). Este enfoque se puede usar para las herramientas de sincronización o las herramientas que procesan usuarios en general y no un usuario específico.
- Use el [flujo con derechos delegados](msal-authentication-flows.md#on-behalf-of) para que una API web llame a una API en nombre del usuario. La aplicación se identifica con las credenciales del cliente para adquirir un token basado en una aserción de usuario (por ejemplo, SAML o un token de JWT). Este flujo lo usan las aplicaciones que necesitan acceder a recursos de un usuario determinado en llamadas de servicio a servicio.
- Adquiere tokens mediante el [flujo de código de autorización](msal-authentication-flows.md#authorization-code) en aplicaciones web después de que el usuario inicia sesión mediante la dirección URL de solicitud de autorización. La aplicación OpenID Connect usa este mecanismo normalmente, lo cual permite al usuario iniciar sesión mediante OpenID Connect y acceder, posteriormente, a las API web en nombre del usuario.

## <a name="authentication-results"></a>Resultados de autenticación

Cuando el cliente solicita un token de acceso, Azure AD también devuelve un resultado de autenticación que incluye metadatos sobre el token de acceso. Esta información incluye la hora de expiración del token de acceso y los ámbitos para los que es válido. Estos datos permiten a la aplicación realizar un almacenamiento inteligente en caché de los tokens de acceso sin tener que analizar el mismo token de acceso. El resultado de la autenticación expone:

- El [token de acceso](access-tokens.md) para que la API web acceda a los recursos. Esta cadena es normalmente de tipo JWT codificada en Base64, pero el cliente nunca debería mirar dentro del token de acceso. No se garantiza que el formato permanezca estable y se pueda cifrar para el recurso. Los usuarios que escriben código en función del contenido de un token de acceso del cliente contribuyen habitualmente a crear errores e interrumpir la lógica de cliente.
- El [token de identificador](id-tokens.md) del usuario (JWT).
- La expiración del token, que le indica la fecha y hora cuando expira el token.
- El identificador de inquilino indica el inquilino en el que se encontró el usuario. Para los usuarios invitados (escenarios de Azure AD B2B), el identificador de inquilino es el inquilino invitado, no el inquilino único. Cuando el token se entrega en nombre de un usuario, el resultado de la autenticación también contiene información sobre este usuario. Para los flujos de cliente confidenciales en los que se solicitan tokens sin usuario (para la aplicación), esta información de usuario está vacía.
- Los ámbitos para los que el token se emitió.
- El identificador único del usuario.

## <a name="next-steps"></a>Pasos siguientes

Si usa MSAL para Java, obtenga información sobre la [serialización de la memoria caché de tokens personalizada en MSAL para Java](msal-java-token-cache-serialization.md).

Más información sobre el [control de errores y excepciones](msal-handling-exceptions.md).