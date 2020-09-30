---
title: Flujo con derechos delegados de OAuth 2.0 y Plataforma de identidad de Microsoft | Azure
titleSuffix: Microsoft identity platform
description: En este artículo se describe cómo usar los mensajes HTTP para implementar la autenticación entre servicios mediante el flujo en nombre de OAuth 2.0.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 08/7/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 018d67b3e4e730cd46eb524a8927b3a6d68d74e8
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88958667"
---
# <a name="microsoft-identity-platform-and-oauth-20-on-behalf-of-flow"></a>Flujo con derechos delegados de OAuth 2.0 y Plataforma de identidad de Microsoft


El flujo con derechos delegados (OBO) de OAuth 2.0 se usa en los casos en que una aplicación invoca un servicio o una API web que a su vez debe llamar a otro servicio o API web. La idea es propagar la identidad y los permisos del usuario delegado a través de la cadena de solicitud. Para que el servicio de nivel intermedio realice solicitudes autenticadas al servicio de bajada, debe proteger un token de acceso de la plataforma de identidad de Microsoft en nombre del usuario.

En este artículo se describe cómo programar directamente con el protocolo de la aplicación.  Cuando sea posible, se recomienda usar las bibliotecas de autenticación de Microsoft (MSAL) admitidas, en lugar de [adquirir tokens y API web protegidas por llamadas](authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows).  Además, eche un vistazo a las [aplicaciones de ejemplo que usan MSAL](sample-v2-code.md).

> [!NOTE]
> A partir de mayo de 2018, algún token `id_token` derivado del flujo implícito no se puede usar para el flujo OBO. Las aplicaciones de una sola página (SPA) deben pasar un token de **acceso** a un cliente confidencial de nivel intermedio para ejecutar flujos de OBO en su lugar. Para más información sobre los clientes que pueden realizar llamadas OBO, vea [Limitaciones](#client-limitations).

## <a name="protocol-diagram"></a>Diagrama de protocolo

Suponga que el usuario se ha autenticado en una aplicación mediante el [flujo de concesión del código de autorización de OAuth 2.0](v2-oauth2-auth-code-flow.md) u otro flujo de inicio de sesión. En este punto, la aplicación tiene un token de acceso *para la API A* (token A) con las notificaciones del usuario y su consentimiento para acceder a la API web de nivel intermedio (API A). Ahora, la API A debe realizar una solicitud autenticada a la API web de bajada (API B).

Los pasos siguientes constituyen el flujo con derechos delegados y se explican con la ayuda del diagrama siguiente.

![Muestra el flujo con derechos delegados de OAuth 2.0](./media/v2-oauth2-on-behalf-of-flow/protocols-oauth-on-behalf-of-flow.png)

1. La aplicación cliente realiza una solicitud a la API A con el token A (con una notificación `aud` de la API A).
1. La API A se autentica en el punto de conexión de emisión de tokens de la Plataforma de identidad de Microsoft y solicita un token para obtener acceso a la API B.
1. El punto de conexión de emisión de tokens de la Plataforma de identidad de Microsoft valida las credenciales de la API A con el token A y emite el token de acceso para la API B (token B).
1. El token B lo establece la API A en el encabezado de autorización de la solicitud a la API B.
1. La API B devuelve los datos del recurso protegido a la API A y desde allí hasta el cliente.

> [!NOTE]
> En este caso, el servicio de nivel intermedio no tiene interacción con el usuario para obtener el consentimiento del usuario para obtener acceso a la API de bajada. Por lo tanto, la opción para conceder acceso a la API de bajada se presenta inicialmente como parte del paso de autorización durante la autenticación. Para obtener información sobre cómo configurar esta opción para la aplicación, vea [Obtener consentimiento para la aplicación de nivel intermedio](#gaining-consent-for-the-middle-tier-application).

## <a name="middle-tier-access-token-request"></a>Solicitud de token de acceso de nivel intermedio

Para solicitar un token de acceso, realice una solicitud HTTP POST al punto de conexión de la Plataforma de identidad de Microsoft específico del inquilino con los parámetros siguientes.

```
https://login.microsoftonline.com/<tenant>/oauth2/v2.0/token
```

Se pueden dar dos casos en función de si la aplicación cliente elige un secreto compartido o un certificado para su protección.

### <a name="first-case-access-token-request-with-a-shared-secret"></a>Primer caso: solicitud de token de acceso con un secreto compartido

Cuando se utiliza un secreto compartido, una solicitud de token de acceso entre servicios contiene los parámetros siguientes:

| Parámetro | Tipo | Descripción |
| --- | --- | --- |
| `grant_type` | Obligatorio | Tipo de la solicitud de token. Para una solicitud mediante un JWT, el valor debe ser `urn:ietf:params:oauth:grant-type:jwt-bearer`. |
| `client_id` | Obligatorio | El identificador de aplicación (cliente) que la página [Azure Portal: Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) asignó a la aplicación. |
| `client_secret` | Obligatorio | El secreto de cliente que generó para la aplicación en la página Azure Portal: Registros de aplicaciones. |
| `assertion` | Obligatorio | Token de acceso que se envió a la API de nivel intermedio.  Este token debe tener una notificación de audiencia (`aud`) de la aplicación que realiza esta solicitud delegada (la aplicación indicada por el campo `client-id`). Las aplicaciones no pueden canjear un token por una aplicación diferente (por ejemplo, si un cliente envía a una API un token diseñado para MS Graph, la API no puede canjearlo mediante la delegación.  De lo contrario, debe rechazar el token).  |
| `scope` | Obligatorio | Lista de ámbitos separados por un espacio para la solicitud de token. Para obtener más información, vea [Ámbitos](v2-permissions-and-consent.md). |
| `requested_token_use` | Obligatorio | Especifica cómo se debe procesar la solicitud. En el flujo OBO, el valor se debe establecer en `on_behalf_of`. |

#### <a name="example"></a>Ejemplo

El siguiente HTTP POST solicita un token de acceso y un token de actualización con el ámbito `user.read` para la API web https://graph.microsoft.com.

```HTTP
//line breaks for legibility only

POST /oauth2/v2.0/token HTTP/1.1
Host: login.microsoftonline.com/<tenant>
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
&client_id=2846f71b-a7a4-4987-bab3-760035b2f389
&client_secret=BYyVnAt56JpLwUcyo47XODd
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiIyO{a lot of characters here}
&scope=https://graph.microsoft.com/user.read+offline_access
&requested_token_use=on_behalf_of
```

### <a name="second-case-access-token-request-with-a-certificate"></a>Segundo caso: solicitud de token de acceso con un certificado

Una solicitud de token de acceso entre servicios con un certificado contiene los parámetros siguientes:

| Parámetro | Tipo | Descripción |
| --- | --- | --- |
| `grant_type` | Obligatorio | Tipo de la solicitud de token. Para una solicitud mediante un JWT, el valor debe ser `urn:ietf:params:oauth:grant-type:jwt-bearer`. |
| `client_id` | Obligatorio |  El identificador de aplicación (cliente) que la página [Azure Portal: Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) asignó a la aplicación. |
| `client_assertion_type` | Obligatorio | El valor tiene que ser `urn:ietf:params:oauth:client-assertion-type:jwt-bearer`. |
| `client_assertion` | Obligatorio | Una aserción (JSON Web Token) que debe crear y firmar con el certificado que ha registrado como credenciales de la aplicación. Para información sobre cómo registrar el certificado y el formato de la aserción, lea el artículo sobre las [credenciales de certificado](active-directory-certificate-credentials.md). |
| `assertion` | Obligatorio |  Token de acceso que se envió a la API de nivel intermedio.  Este token debe tener una notificación de audiencia (`aud`) de la aplicación que realiza esta solicitud delegada (la aplicación indicada por el campo `client-id`). Las aplicaciones no pueden canjear un token por una aplicación diferente (por ejemplo, si un cliente envía a una API un token diseñado para MS Graph, la API no puede canjearlo mediante la delegación.  De lo contrario, debe rechazar el token).  |
| `requested_token_use` | Obligatorio | Especifica cómo se debe procesar la solicitud. En el flujo OBO, el valor se debe establecer en `on_behalf_of`. |
| `scope` | Obligatorio | Lista de ámbitos separados por un espacio para la solicitud de token. Para obtener más información, vea [Ámbitos](v2-permissions-and-consent.md).|

Tenga en cuenta que los parámetros son casi iguales que en el caso de solicitud con un secreto compartido, salvo que el parámetro `client_secret` se sustituye por dos parámetros: `client_assertion_type` y `client_assertion`.

#### <a name="example"></a>Ejemplo

El siguiente elemento HTTP POST solicita un token de acceso con el ámbito `user.read` para la API web https://graph.microsoft.com con un certificado.

```HTTP
// line breaks for legibility only

POST /oauth2/v2.0/token HTTP/1.1
Host: login.microsoftonline.com/<tenant>
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiO{a lot of characters here}
&requested_token_use=on_behalf_of
&scope=https://graph.microsoft.com/user.read+offline_access
```

## <a name="middle-tier-access-token-response"></a>Respuesta de token de acceso de nivel intermedio

Una respuesta correcta es una respuesta de OAuth 2.0 de JSON con los parámetros siguientes.

| Parámetro | Descripción |
| --- | --- |
| `token_type` | Indica el valor de tipo de token. El único tipo que la Plataforma de identidad de Microsoft admite es `Bearer`. Para más información sobre los tokens de portador, vea [OAuth 2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) (Marco de autorización de OAuth2.0: uso del token de portador [RFC 6750]). |
| `scope` | Ámbito de acceso concedido en el token. |
| `expires_in` | El período de validez, en segundos, del token de acceso. |
| `access_token` | El token de acceso solicitado. El servicio de llamada puede usar este token para autenticarse en el servicio de recepción. |
| `refresh_token` | Token de actualización para el token de acceso solicitado. El servicio de llamada puede usar este token para solicitar otro token de acceso después de que expire el token de acceso actual. El token de actualización solo se proporciona si se solicitó el ámbito `offline_access`. |

### <a name="success-response-example"></a>Ejemplo de respuesta correcta

En el ejemplo siguiente se muestra una respuesta correcta a una solicitud de un token de acceso para la API web https://graph.microsoft.com.

```json
{
  "token_type": "Bearer",
  "scope": "https://graph.microsoft.com/user.read",
  "expires_in": 3269,
  "ext_expires_in": 0,
  "access_token": "eyJ0eXAiOiJKV1QiLCJub25jZSI6IkFRQUJBQUFBQUFCbmZpRy1tQTZOVGFlN0NkV1c3UWZkQ0NDYy0tY0hGa18wZE50MVEtc2loVzRMd2RwQVZISGpnTVdQZ0tQeVJIaGlDbUN2NkdyMEpmYmRfY1RmMUFxU21TcFJkVXVydVJqX3Nqd0JoN211eHlBQSIsImFsZyI6IlJTMjU2IiwieDV0IjoiejAzOXpkc0Z1aXpwQmZCVksxVG4yNVFIWU8wIiwia2lkIjoiejAzOXpkc0Z1aXpwQmZCVksxVG4yNVFIWU8wIn0.eyJhdWQiOiJodHRwczovL2dyYXBoLm1pY3Jvc29mdC5jb20iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMWRiNDcvIiwiaWF0IjoxNDkzOTMwMzA1LCJuYmYiOjE0OTM5MzAzMDUsImV4cCI6MTQ5MzkzMzg3NSwiYWNyIjoiMCIsImFpbyI6IkFTUUEyLzhEQUFBQU9KYnFFWlRNTnEyZFcxYXpKN1RZMDlYeDdOT29EMkJEUlRWMXJ3b2ZRc1k9IiwiYW1yIjpbInB3ZCJdLCJhcHBfZGlzcGxheW5hbWUiOiJUb2RvRG90bmV0T2JvIiwiYXBwaWQiOiIyODQ2ZjcxYi1hN2E0LTQ5ODctYmFiMy03NjAwMzViMmYzODkiLCJhcHBpZGFjciI6IjEiLCJmYW1pbHlfbmFtZSI6IkNhbnVtYWxsYSIsImdpdmVuX25hbWUiOiJOYXZ5YSIsImlwYWRkciI6IjE2Ny4yMjAuMC4xOTkiLCJuYW1lIjoiTmF2eWEgQ2FudW1hbGxhIiwib2lkIjoiZDVlOTc5YzctM2QyZC00MmFmLThmMzAtNzI3ZGQ0YzJkMzgzIiwib25wcmVtX3NpZCI6IlMtMS01LTIxLTIxMjc1MjExODQtMTYwNDAxMjkyMC0xODg3OTI3NTI3LTI2MTE4NDg0IiwicGxhdGYiOiIxNCIsInB1aWQiOiIxMDAzM0ZGRkEwNkQxN0M5Iiwic2NwIjoiVXNlci5SZWFkIiwic3ViIjoibWtMMHBiLXlpMXQ1ckRGd2JTZ1JvTWxrZE52b3UzSjNWNm84UFE3alVCRSIsInRpZCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsInVuaXF1ZV9uYW1lIjoibmFjYW51bWFAbWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hY2FudW1hQG1pY3Jvc29mdC5jb20iLCJ1dGkiOiJWR1ItdmtEZlBFQ2M1dWFDaENRSkFBIiwidmVyIjoiMS4wIn0.cubh1L2VtruiiwF8ut1m9uNBmnUJeYx4x0G30F7CqSpzHj1Sv5DCgNZXyUz3pEiz77G8IfOF0_U5A_02k-xzwdYvtJUYGH3bFISzdqymiEGmdfCIRKl9KMeoo2llGv0ScCniIhr2U1yxTIkIpp092xcdaDt-2_2q_ql1Ha_HtjvTV1f9XR3t7_Id9bR5BqwVX5zPO7JMYDVhUZRx08eqZcC-F3wi0xd_5ND_mavMuxe2wrpF-EZviO3yg0QVRr59tE3AoWl8lSGpVc97vvRCnp4WVRk26jJhYXFPsdk4yWqOKZqzr3IFGyD08WizD_vPSrXcCPbZP3XWaoTUKZSNJg",
  "refresh_token": "OAQABAAAAAABnfiG-mA6NTae7CdWW7QfdAALzDWjw6qSn4GUDfxWzJDZ6lk9qRw4An{a lot of characters here}"
}
```

> [!NOTE]
> El token de acceso anterior es un token con formato v1.0 para Microsoft Graph. Esto se debe a que el formato del token se basa en el **recurso** al que se accede y no está relacionado con los puntos de conexión usados para solicitarlo. Microsoft Graph se configura para aceptar tokens v1.0, por lo que la Plataforma de identidad de Microsoft genera tokens de acceso v1.0 cuando un cliente solicita tokens para Microsoft Graph. Otras aplicaciones pueden indicar que quieren tokens con formato v2.0, tokens con formato v1.0 o incluso formatos de tokens de propiedad o cifrados.  Los puntos de conexión v1.0 y v2.0 pueden emitir cualquier formato de token: de este modo, el recurso siempre puede obtener el formato correcto del token, independientemente de cómo o dónde el cliente solicitó el token. 
>
> Solo las aplicaciones deben revisar los tokens de acceso. Los clientes **no deben** inspeccionarlos. Al inspeccionar los tokens de acceso de otras aplicaciones en el código, la aplicación se interrumpirá inesperadamente cuando la aplicación cambie el formato de sus tokens o empiece a cifrarlos. 

### <a name="error-response-example"></a>Ejemplo de respuesta de error

El punto de conexión del token devuelve una respuesta de error al intentar adquirir un token de acceso para la API de bajada si dicha API tiene establecida una directiva de acceso condicional, como la [autenticación multifactor](../authentication/concept-mfa-howitworks.md). El servicio de nivel intermedio debe exponer el error a la aplicación cliente para que esta pueda proporcionar la interacción del usuario necesaria para cumplir la directiva de acceso condicional.

```json
{
    "error":"interaction_required",
    "error_description":"AADSTS50079: Due to a configuration change made by your administrator, or because you moved to a new location, you must enroll in multi-factor authentication to access 'bf8d80f9-9098-4972-b203-500f535113b1'.\r\nTrace ID: b72a68c3-0926-4b8e-bc35-3150069c2800\r\nCorrelation ID: 73d656cf-54b1-4eb2-b429-26d8165a52d7\r\nTimestamp: 2017-05-01 22:43:20Z",
    "error_codes":[50079],
    "timestamp":"2017-05-01 22:43:20Z",
    "trace_id":"b72a68c3-0926-4b8e-bc35-3150069c2800",
    "correlation_id":"73d656cf-54b1-4eb2-b429-26d8165a52d7",
    "claims":"{\"access_token\":{\"polids\":{\"essential\":true,\"values\":[\"9ab03e19-ed42-4168-b6b7-7001fb3e933a\"]}}}"
}
```

## <a name="use-the-access-token-to-access-the-secured-resource"></a>Usar el token de acceso para obtener acceso al recurso protegido

Ahora, el servicio de nivel intermedio puede usar el token adquirido anteriormente para realizar solicitudes autenticadas a la API web de bajada mediante el establecimiento del token en el encabezado `Authorization`.

### <a name="example"></a>Ejemplo

```HTTP
GET /v1.0/me HTTP/1.1
Host: graph.microsoft.com
Authorization: Bearer eyJ0eXAiO ... 0X2tnSQLEANnSPHY0gKcgw
```

## <a name="saml-assertions-obtained-with-an-oauth20-obo-flow"></a>Aserciones de SAML obtenidas con un flujo de OBO de OAuth2.0

Algunos servicios web basados en OAuth necesitan tener acceso a otras API de servicio web que aceptan aserciones de SAML en flujos no interactivos. Azure Active Directory puede proporcionar una aserción de SAML en respuesta a un flujo de On-Behalf-Of que usa un servicio web basado en SAML como un recurso de destino.

>[!NOTE]
>Se trata de una extensión no estándar del flujo de On-Behalf-Of de OAuth 2.0 que permite a una aplicación basada en OAuth2 acceder a puntos de conexión de API de servicio web que consumen tokens SAML.

> [!TIP]
> Cuando llama a un servicio web SAML protegido desde una aplicación web front-end, solo puede llamar a la API e iniciar un flujo de autenticación interactiva normal que usará la sesión existente del usuario. Solo debe considerar el uso de un flujo de OBO cuando una llamada entre servicios requiere que un token SAML proporcione el contexto del usuario.

## <a name="gaining-consent-for-the-middle-tier-application"></a>Obtener consentimiento para la aplicación de nivel intermedio

Según la arquitectura o el uso de la aplicación, puede plantearse estrategias distintas para garantizar que el flujo de OBO sea correcto. En todos los casos, el objetivo final es asegurarse de que la aplicación cliente puede llamar a la aplicación de nivel intermedio, y la aplicación de nivel intermedio tiene permiso para llamar al recurso de back-end.

> [!NOTE]
> Anteriormente, el sistema de cuentas de Microsoft (cuentas personales) no admitía el campo "aplicación cliente conocida" ni podía mostrar el consentimiento combinado.  Esto se ha agregado y todas las aplicaciones de la Plataforma de identidad de Microsoft pueden usar el enfoque de aplicación cliente conocida para obtener el consentimiento para llamadas a OBO.

### <a name="default-and-combined-consent"></a>Consentimiento /.default y combinado

La aplicación de nivel intermedio agrega el cliente a la lista de aplicaciones cliente conocidas en su manifiesto y, a continuación, el cliente puede desencadenar un flujo de consentimiento combinado tanto para sí mismo como para la aplicación de nivel intermedio. En el punto de conexión de la Plataforma de identidad de Microsoft, esto se hace con el ámbito[`/.default`](v2-permissions-and-consent.md#the-default-scope). Cuando se desencadena una pantalla de consentimiento con aplicaciones cliente conocidas y `/.default`, la pantalla de consentimiento mostrará los permisos de **ambos** al cliente y a la API de nivel intermedio, y también solicitará los permisos que requiera la API de nivel intermedio. El usuario da su consentimiento para ambas aplicaciones y, a continuación, el flujo OBO funcionará.

### <a name="pre-authorized-applications"></a>Aplicaciones preautorizadas

Los recursos pueden indicar que una determinada aplicación siempre tiene permiso para recibir determinados ámbitos. Esto es principalmente útil para realizar conexiones más directas entre un cliente front-end y un recurso back-end. Un recurso puede declarar varias aplicaciones preautorizadas: cualquier aplicación puede solicitar estos permisos en un flujo OBO y recibirlos sin necesidad de que el usuario dé el consentimiento.

### <a name="admin-consent"></a>Consentimiento de administrador

Un administrador de inquilinos puede garantizar que las aplicaciones tienen permiso para llamar a las API necesarias al proporcionar el consentimiento del administrador para la aplicación de nivel intermedio. Para ello, el administrador puede encontrar la aplicación de nivel intermedio en su inquilino, abrir la página de los permisos necesarios y elegir la opción para dar permiso a la aplicación. Para más información sobre el consentimiento del administrador, vea la [documentación sobre consentimientos y permisos](v2-permissions-and-consent.md).

### <a name="use-of-a-single-application"></a>Uso de una sola aplicación

En algunos escenarios, solo puede haber un único emparejamiento entre el nivel intermedio y el cliente front-end. En este escenario, le resultará más fácil hacer esto en una sola aplicación, sin la necesidad de que exista una aplicación de nivel intermedio. Para realizar la autenticación entre el cliente front-end y la API web, puede usar cookies, un token de identificador o un token de acceso solicitado para la propia aplicación. A continuación, solicite el consentimiento a esta aplicación única para el recurso back-end.

## <a name="client-limitations"></a>Limitaciones del cliente

Si un cliente usa el flujo implícito para obtener un id_token y ese cliente también incluye caracteres comodín en una dirección URL de respuesta, el id_token no se puede usar con un flujo OBO.  Sin embargo, los tokens de acceso adquiridos mediante del flujo de concesión implícita todavía pueden ser canjeados por un cliente confidencial, aunque el cliente iniciador tenga registrada una dirección URL de respuesta con caracteres comodín.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el protocolo OAuth 2.0 y conozca otra manera de realizar la autenticación entre servicios con las credenciales del cliente.

* [La Plataforma de identidad de Microsoft y el flujo de credenciales de cliente de OAuth 2.0](v2-oauth2-client-creds-grant-flow.md)
* [OAuth 2.0 code flow in Microsoft identity platform](v2-oauth2-auth-code-flow.md) (Flujo de código de OAuth 2.0 en la Plataforma de identidad de Microsoft)
* [Uso del ámbito `/.default`](v2-permissions-and-consent.md#the-default-scope)
