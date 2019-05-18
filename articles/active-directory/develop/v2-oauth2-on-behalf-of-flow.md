---
title: Plataforma de identidad de Microsoft y flujo de On-Behalf-Of OAuth2.0 | Azure
description: En este artículo se describe cómo usar los mensajes HTTP para implementar la autenticación entre servicios mediante el flujo en nombre de OAuth 2.0.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 09f6f318-e88b-4024-9ee1-e7f09fb19a82
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/05/2019
ms.author: ryanwi
ms.reviewer: hirsin
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: ce0c1c4dcf7e4ff0c82157af83aa15544cf092e2
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544745"
---
# <a name="microsoft-identity-platform-and-oauth-20-on-behalf-of-flow"></a>Plataforma de identidad de Microsoft y flujo en nombre de OAuth 2.0

[!INCLUDE [active-directory-develop-applies-v2](../../../includes/active-directory-develop-applies-v2.md)]

El flujo con derechos delegados (OBO) de OAuth 2.0 se usa en los casos en que una aplicación invoca un servicio o una API web que a su vez debe llamar a otro servicio o API web. La idea es propagar la identidad y los permisos del usuario delegado a través de la cadena de solicitud. Para que el servicio de nivel intermedio realice solicitudes autenticadas al servicio de bajada, debe proteger un token de acceso de la plataforma Microsoft identity, en nombre del usuario.

> [!NOTE]
>
> - El punto de conexión de plataforma de identidad de Microsoft no es compatible con todas las características y escenarios. Para determinar si debe utilizar el punto de conexión de plataforma de identidad de Microsoft, obtenga información sobre [limitaciones de la plataforma de identidad de Microsoft](active-directory-v2-limitations.md). En concreto, no se admiten las aplicaciones cliente conocidas para las aplicaciones con la cuenta de Microsoft (MSA) y públicos de Azure AD. Por lo tanto, un patrón común de consentimiento para OBO no funcionará para los clientes que inician sesión en cuentas tanto personales como profesionales o educativas. Para más información sobre cómo controlar este paso del flujo, vea [Obtener consentimiento para la aplicación de nivel intermedio](#gaining-consent-for-the-middle-tier-application).
> - A partir de mayo de 2018, algún token `id_token` derivado del flujo implícito no se puede usar para el flujo OBO. Las aplicaciones de una sola página (SPA) deben pasar un token de **acceso** a un cliente confidencial de nivel intermedio para ejecutar flujos de OBO en su lugar. Para más información sobre los clientes que pueden realizar llamadas OBO, vea [Limitaciones](#client-limitations).

## <a name="protocol-diagram"></a>Diagrama de protocolo

Suponga que el usuario se ha autenticado en una aplicación mediante el [flujo de concesión del código de autorización de OAuth 2.0](v2-oauth2-auth-code-flow.md). En este punto, la aplicación tiene un token de acceso *para la API A* (token A) con las notificaciones del usuario y su consentimiento para tener acceso a la API web de nivel intermedio (API A). Ahora, la API A debe realizar una solicitud autenticada a la API web de bajada (API B).

Los pasos siguientes constituyen el flujo con derechos delegados y se explican con la ayuda del diagrama siguiente.

![Flujo con derechos delegados de OAuth 2.0](./media/v2-oauth2-on-behalf-of-flow/protocols-oauth-on-behalf-of-flow.png)

1. La aplicación cliente realiza una solicitud a la API A con el token A (con una notificación `aud` de la API A).
1. API A se autentica en el extremo de emisión de tokens de plataforma de identidad de Microsoft y solicita un token para acceder a la API B.
1. El extremo de emisión de tokens de plataforma de identidad de Microsoft valida las credenciales de la API A con el token A y emite el token de acceso para la API B (token B).
1. El token B se establece en el encabezado de autorización de la solicitud a la API B.
1. La API B devuelve los datos del recurso protegido.

> [!NOTE]
> En este caso, el servicio de nivel intermedio no tiene interacción con el usuario para obtener el consentimiento del usuario para obtener acceso a la API de bajada. Por lo tanto, la opción para conceder acceso a la API de bajada se presenta inicialmente como parte del paso de autorización durante la autenticación. Para obtener información sobre cómo configurar esta opción para la aplicación, vea [Obtener consentimiento para la aplicación de nivel intermedio](#gaining-consent-for-the-middle-tier-application).

## <a name="service-to-service-access-token-request"></a>Solicitud de token de acceso entre servicios

Para solicitar un token de acceso, realice una solicitud HTTP POST para el extremo específico del inquilino Microsoft identity platform token con los siguientes parámetros.

```
https://login.microsoftonline.com/<tenant>/oauth2/v2.0/token
```

Se pueden dar dos casos en función de si la aplicación cliente elige un secreto compartido o un certificado para su protección.

### <a name="first-case-access-token-request-with-a-shared-secret"></a>Primer caso: solicitud de token de acceso con un secreto compartido

Cuando se utiliza un secreto compartido, una solicitud de token de acceso entre servicios contiene los parámetros siguientes:

| Parámetro |  | DESCRIPCIÓN |
| --- | --- | --- |
| `grant_type` | Obligatorio | Tipo de la solicitud de token. Para una solicitud mediante un JWT, el valor debe ser `urn:ietf:params:oauth:grant-type:jwt-bearer`. |
| `client_id` | Obligatorio | IDENTIFICADOR de la aplicación (cliente) que [Azure portal: registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) página ha asignado a la aplicación. |
| `client_secret` | Obligatorio | El secreto de cliente que ha generado para la aplicación en Azure portal: página de registros de aplicación. |
| `assertion` | Obligatorio | Valor del token usado en la solicitud. |
| `scope` | Obligatorio | Lista de ámbitos separados por un espacio para la solicitud de token. Para obtener más información, vea [Ámbitos](v2-permissions-and-consent.md). |
| `requested_token_use` | Obligatorio | Especifica cómo se debe procesar la solicitud. En el flujo OBO, el valor se debe establecer en `on_behalf_of`. |

#### <a name="example"></a>Ejemplo

El siguiente HTTP POST solicita un token de acceso y un token de actualización con el ámbito `user.read` para la API web https://graph.microsoft.com.

```
//line breaks for legibility only

POST /oauth2/v2.0/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:jwt-bearer
&client_id=2846f71b-a7a4-4987-bab3-760035b2f389
&client_secret=BYyVnAt56JpLwUcyo47XODd
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiIyODQ2ZjcxYi1hN2E0LTQ5ODctYmFiMy03NjAwMzViMmYzODkiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vNzJmOTg4YmYtODZmMS00MWFmLTkxYWItMmQ3Y2QwMTFkYjQ3L3YyLjAiLCJpYXQiOjE0OTM5MjA5MTYsIm5iZiI6MTQ5MzkyMDkxNiwiZXhwIjoxNDkzOTI0ODE2LCJhaW8iOiJBU1FBMi84REFBQUFnZm8vNk9CR0NaaFV2NjJ6MFFYSEZKR0VVYUIwRUlIV3NhcGducndMMnVrPSIsIm5hbWUiOiJOYXZ5YSBDYW51bWFsbGEiLCJvaWQiOiJkNWU5NzljNy0zZDJkLTQyYWYtOGYzMC03MjdkZDRjMmQzODMiLCJwcmVmZXJyZWRfdXNlcm5hbWUiOiJuYWNhbnVtYUBtaWNyb3NvZnQuY29tIiwic3ViIjoiZ1Q5a1FMN2hXRUpUUGg1OWJlX1l5dVZNRDFOTEdiREJFWFRhbEQzU3FZYyIsInRpZCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsInV0aSI6IjN5U3F4UHJweUVPd0ZsTWFFMU1PQUEiLCJ2ZXIiOiIyLjAifQ.TPPJSvpNCSCyUeIiKQoLMixN1-M-Y5U0QxtxVkpepjyoWNG0i49YFAJC6ADdCs5nJXr6f-ozIRuaiPzy29yRUOdSz_8KqG42luCyC1c951HyeDgqUJSz91Ku150D9kP5B9-2R-jgCerD_VVuxXUdkuPFEl3VEADC_1qkGBiIg0AyLLbz7DTMp5DvmbC09DhrQQiouHQGFSk2TPmksqHm3-b3RgeNM1rJmpLThis2ZWBEIPx662pjxL6NJDmV08cPVIcGX4KkFo54Z3rfwiYg4YssiUc4w-w3NJUBQhnzfTl4_Mtq2d7cVlul9uDzras091vFy32tWkrpa970UvdVfQ
&scope=https://graph.microsoft.com/user.read+offline_access
&requested_token_use=on_behalf_of
```

### <a name="second-case-access-token-request-with-a-certificate"></a>Segundo caso: solicitud de token de acceso con un certificado

Una solicitud de token de acceso entre servicios con un certificado contiene los parámetros siguientes:

| Parámetro |  | DESCRIPCIÓN |
| --- | --- | --- |
| `grant_type` | Obligatorio | Tipo de la solicitud de token. Para una solicitud mediante un JWT, el valor debe ser `urn:ietf:params:oauth:grant-type:jwt-bearer`. |
| `client_id` | Obligatorio |  IDENTIFICADOR de la aplicación (cliente) que [Azure portal: registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) página ha asignado a la aplicación. |
| `client_assertion_type` | Obligatorio | El valor tiene que ser `urn:ietf:params:oauth:client-assertion-type:jwt-bearer`. |
| `client_assertion` | Obligatorio | Aserción (un JSON Web Token) que debe crear y firmar con el certificado que ha registrado como credenciales de la aplicación. Para información sobre cómo registrar el certificado y el formato de la aserción, lea el artículo sobre las [credenciales de certificado](active-directory-certificate-credentials.md). |
| `assertion` | Obligatorio | Valor del token usado en la solicitud. |
| `requested_token_use` | Obligatorio | Especifica cómo se debe procesar la solicitud. En el flujo OBO, el valor se debe establecer en `on_behalf_of`. |
| `scope` | Obligatorio | Lista de ámbitos separados por un espacio para la solicitud de token. Para obtener más información, vea [Ámbitos](v2-permissions-and-consent.md).|

Tenga en cuenta que los parámetros son casi iguales que en el caso de solicitud con un secreto compartido, salvo que el parámetro `client_secret` se sustituye por dos parámetros: `client_assertion_type` y `client_assertion`.

#### <a name="example"></a>Ejemplo

El siguiente elemento HTTP POST solicita un token de acceso con el ámbito `user.read` para la API web https://graph.microsoft.com con un certificado.

```
// line breaks for legibility only

POST /oauth2/v2.0/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn%3Aietf%3Aparams%3Aoauth%3Agrant-type%3Ajwt-bearer
&client_id=625391af-c675-43e5-8e44-edd3e30ceb15
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&assertion=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCIsImtpZCI6InowMzl6ZHNGdWl6cEJmQlZLMVRuMjVRSFlPMCJ9.eyJhdWQiOiJodHRwczovL2Rkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tLzE5MjNmODYyLWU2ZGMtNDFhMy04MWRhLTgwMmJhZTAwYWY2ZCIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzI2MDM5Y2NlLTQ4OWQtNDAwMi04MjkzLTViMGM1MTM0ZWFjYi8iLCJpYXQiOjE0OTM0MjMxNTIsIm5iZiI6MTQ5MzQyMzE1MiwiZXhwIjoxNDkzNDY2NjUyLCJhY3IiOiIxIiwiYWlvIjoiWTJaZ1lCRFF2aTlVZEc0LzM0L3dpQndqbjhYeVp4YmR1TFhmVE1QeG8yYlN2elgreHBVQSIsImFtciI6WyJwd2QiXSwiYXBwaWQiOiJiMzE1MDA3OS03YmViLTQxN2YtYTA2YS0zZmRjNzhjMzI1NDUiLCJhcHBpZGFjciI6IjAiLCJlX2V4cCI6MzAyNDAwLCJmYW1pbHlfbmFtZSI6IlRlc3QiLCJnaXZlbl9uYW1lIjoiTmF2eWEiLCJpcGFkZHIiOiIxNjcuMjIwLjEuMTc3IiwibmFtZSI6Ik5hdnlhIFRlc3QiLCJvaWQiOiIxY2Q0YmNhYy1iODA4LTQyM2EtOWUyZi04MjdmYmIxYmI3MzkiLCJwbGF0ZiI6IjMiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJzdWIiOiJEVXpYbkdKMDJIUk0zRW5pbDFxdjZCakxTNUllQy0tQ2ZpbzRxS1MzNEc4IiwidGlkIjoiMjYwMzljY2UtNDg5ZC00MDAyLTgyOTMtNWIwYzUxMzRlYWNiIiwidW5pcXVlX25hbWUiOiJuYXZ5YUBkZG9iYWxpYW5vdXRsb29rLm9ubWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hdnlhQGRkb2JhbGlhbm91dGxvb2sub25taWNyb3NvZnQuY29tIiwidmVyIjoiMS4wIn0.R-Ke-XO7lK0r5uLwxB8g5CrcPAwRln5SccJCfEjU6IUqpqcjWcDzeDdNOySiVPDU_ZU5knJmzRCF8fcjFtPsaA4R7vdIEbDuOur15FXSvE8FvVSjP_49OH6hBYqoSUAslN3FMfbO6Z8YfCIY4tSOB2I6ahQ_x4ZWFWglC3w5mK-_4iX81bqi95eV4RUKefUuHhQDXtWhrSgIEC0YiluMvA4TnaJdLq_tWXIc4_Tq_KfpkvI004ONKgU7EAMEr1wZ4aDcJV2yf22gQ1sCSig6EGSTmmzDuEPsYiyd4NhidRZJP4HiiQh-hePBQsgcSgYGvz9wC6n57ufYKh2wm_Ti3Q
&requested_token_use=on_behalf_of
&scope=https://graph.microsoft.com/user.read+offline_access
```

## <a name="service-to-service-access-token-response"></a>Respuesta de token de acceso entre servicios

Una respuesta correcta es una respuesta de OAuth 2.0 de JSON con los parámetros siguientes.

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| `token_type` | Indica el valor de tipo de token. El único tipo que Microsoft admite de plataforma de identidad es `Bearer`. Para más información sobre los tokens de portador, vea [OAuth 2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) (Marco de autorización de OAuth2.0: uso del token de portador [RFC 6750]). |
| `scope` | Ámbito de acceso concedido en el token. |
| `expires_in` | El período de validez, en segundos, del token de acceso. |
| `access_token` | El token de acceso solicitado. El servicio de llamada puede usar este token para autenticarse en el servicio de recepción. |
| `refresh_token` | Token de actualización para el token de acceso solicitado. El servicio de llamada puede usar este token para solicitar otro token de acceso después de que expire el token de acceso actual. El token de actualización solo se proporciona si se solicitó el ámbito `offline_access`. |

### <a name="success-response-example"></a>Ejemplo de respuesta correcta

En el ejemplo siguiente se muestra una respuesta correcta a una solicitud de un token de acceso para la API web https://graph.microsoft.com.

```
{
  "token_type": "Bearer",
  "scope": "https://graph.microsoft.com/user.read",
  "expires_in": 3269,
  "ext_expires_in": 0,
  "access_token": "eyJ0eXAiOiJKV1QiLCJub25jZSI6IkFRQUJBQUFBQUFCbmZpRy1tQTZOVGFlN0NkV1c3UWZkQ0NDYy0tY0hGa18wZE50MVEtc2loVzRMd2RwQVZISGpnTVdQZ0tQeVJIaGlDbUN2NkdyMEpmYmRfY1RmMUFxU21TcFJkVXVydVJqX3Nqd0JoN211eHlBQSIsImFsZyI6IlJTMjU2IiwieDV0IjoiejAzOXpkc0Z1aXpwQmZCVksxVG4yNVFIWU8wIiwia2lkIjoiejAzOXpkc0Z1aXpwQmZCVksxVG4yNVFIWU8wIn0.eyJhdWQiOiJodHRwczovL2dyYXBoLm1pY3Jvc29mdC5jb20iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMWRiNDcvIiwiaWF0IjoxNDkzOTMwMzA1LCJuYmYiOjE0OTM5MzAzMDUsImV4cCI6MTQ5MzkzMzg3NSwiYWNyIjoiMCIsImFpbyI6IkFTUUEyLzhEQUFBQU9KYnFFWlRNTnEyZFcxYXpKN1RZMDlYeDdOT29EMkJEUlRWMXJ3b2ZRc1k9IiwiYW1yIjpbInB3ZCJdLCJhcHBfZGlzcGxheW5hbWUiOiJUb2RvRG90bmV0T2JvIiwiYXBwaWQiOiIyODQ2ZjcxYi1hN2E0LTQ5ODctYmFiMy03NjAwMzViMmYzODkiLCJhcHBpZGFjciI6IjEiLCJmYW1pbHlfbmFtZSI6IkNhbnVtYWxsYSIsImdpdmVuX25hbWUiOiJOYXZ5YSIsImlwYWRkciI6IjE2Ny4yMjAuMC4xOTkiLCJuYW1lIjoiTmF2eWEgQ2FudW1hbGxhIiwib2lkIjoiZDVlOTc5YzctM2QyZC00MmFmLThmMzAtNzI3ZGQ0YzJkMzgzIiwib25wcmVtX3NpZCI6IlMtMS01LTIxLTIxMjc1MjExODQtMTYwNDAxMjkyMC0xODg3OTI3NTI3LTI2MTE4NDg0IiwicGxhdGYiOiIxNCIsInB1aWQiOiIxMDAzM0ZGRkEwNkQxN0M5Iiwic2NwIjoiVXNlci5SZWFkIiwic3ViIjoibWtMMHBiLXlpMXQ1ckRGd2JTZ1JvTWxrZE52b3UzSjNWNm84UFE3alVCRSIsInRpZCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsInVuaXF1ZV9uYW1lIjoibmFjYW51bWFAbWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hY2FudW1hQG1pY3Jvc29mdC5jb20iLCJ1dGkiOiJWR1ItdmtEZlBFQ2M1dWFDaENRSkFBIiwidmVyIjoiMS4wIn0.cubh1L2VtruiiwF8ut1m9uNBmnUJeYx4x0G30F7CqSpzHj1Sv5DCgNZXyUz3pEiz77G8IfOF0_U5A_02k-xzwdYvtJUYGH3bFISzdqymiEGmdfCIRKl9KMeoo2llGv0ScCniIhr2U1yxTIkIpp092xcdaDt-2_2q_ql1Ha_HtjvTV1f9XR3t7_Id9bR5BqwVX5zPO7JMYDVhUZRx08eqZcC-F3wi0xd_5ND_mavMuxe2wrpF-EZviO3yg0QVRr59tE3AoWl8lSGpVc97vvRCnp4WVRk26jJhYXFPsdk4yWqOKZqzr3IFGyD08WizD_vPSrXcCPbZP3XWaoTUKZSNJg",
  "refresh_token": "OAQABAAAAAABnfiG-mA6NTae7CdWW7QfdAALzDWjw6qSn4GUDfxWzJDZ6lk9qRw4AnqPnvFqnzS3GiikHr5wBM1bV1YyjH3nUeIhKhqJWGwqJFRqs2sE_rqUfz7__3J92JDpi6gDdCZNNaXgreQsH89kLCVNYZeN6kGuFGZrjwxp1wS2JYc97E_3reXBxkHrA09K5aR-WsSKCEjf6WI23FhZMTLhk_ZKOe_nWvcvLj13FyvSrTMZV2cmzyCZDqEHtPVLJgSoASuQlD2NXrfmtcmgWfc3uJSrWLIDSn4FEmVDA63X6EikNp9cllH3Gp7Vzapjlnws1NQ1_Ff5QrmBHp_LKEIwfzVKnLLrQXN0EzP8f6AX6fdVTaeKzm7iw6nH0vkPRpUeLc3q_aNsPzqcTOnFfgng7t2CXUsMAGH5wclAyFCAwL_Cds7KnyDLL7kzOS5AVZ3Mqk2tsPlqopAiHijZaJumdTILDudwKYCFAMpUeUwEf9JmyFjl2eIWPmlbwU7cHKWNvuRCOYVqbsTTpJthwh4PvsL5ov5CawH_TaV8omG_tV6RkziHG9urk9yp2PH9gl7Cv9ATa3Vt3PJWUS8LszjRIAJmyw_EhgHBfYCvEZ8U9PYarvgqrtweLcnlO7BfnnXYEC18z_u5wemAzNBFUje2ttpGtRmRic4AzZ708tBHva2ePJWGX6pgQbiWF8esOrvWjfrrlfOvEn1h6YiBW291M022undMdXzum6t1Y1huwxHPHjCAA"
}
```

> [!NOTE]
> El token de acceso anterior es un token con formato v1.0. Esto es porque el token se proporciona en función del recurso al que se accede. Microsoft Graph solicita tokens v1.0, por lo que la plataforma de identidad de Microsoft genera tokens de acceso de v1.0 cuando un cliente solicita tokens para Microsoft Graph. Solo las aplicaciones deben revisar los tokens de acceso. Los clientes no deberían necesitar examinarlos.

### <a name="error-response-example"></a>Ejemplo de respuesta de error

El punto de conexión del token devuelve una respuesta de error cuando intenta adquirir un token de acceso para la API de bajada si dicha API tiene establecida una directiva de acceso condicional, como la autenticación multifactor. El servicio de nivel intermedio debe exponer el error a la aplicación cliente para que esta pueda proporcionar la interacción del usuario necesaria para cumplir la directiva de acceso condicional.

```
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

```
GET /v1.0/me HTTP/1.1
Host: graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJub25jZSI6IkFRQUJBQUFBQUFCbmZpRy1tQTZOVGFlN0NkV1c3UWZkSzdNN0RyNXlvUUdLNmFEc19vdDF3cEQyZjNqRkxiNlVrcm9PcXA2cXBJclAxZVV0QktzMHEza29HN3RzXzJpSkYtQjY1UV8zVGgzSnktUHZsMjkxaFNBQSIsImFsZyI6IlJTMjU2IiwieDV0IjoiejAzOXpkc0Z1aXpwQmZCVksxVG4yNVFIWU8wIiwia2lkIjoiejAzOXpkc0Z1aXpwQmZCVksxVG4yNVFIWU8wIn0.eyJhdWQiOiJodHRwczovL2dyYXBoLm1pY3Jvc29mdC5jb20iLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMWRiNDcvIiwiaWF0IjoxNDkzOTMwMDE2LCJuYmYiOjE0OTM5MzAwMTYsImV4cCI6MTQ5MzkzMzg3NSwiYWNyIjoiMCIsImFpbyI6IkFTUUEyLzhEQUFBQUlzQjN5ZUljNkZ1aEhkd1YxckoxS1dlbzJPckZOUUQwN2FENTVjUVRtems9IiwiYW1yIjpbInB3ZCJdLCJhcHBfZGlzcGxheW5hbWUiOiJUb2RvRG90bmV0T2JvIiwiYXBwaWQiOiIyODQ2ZjcxYi1hN2E0LTQ5ODctYmFiMy03NjAwMzViMmYzODkiLCJhcHBpZGFjciI6IjEiLCJmYW1pbHlfbmFtZSI6IkNhbnVtYWxsYSIsImdpdmVuX25hbWUiOiJOYXZ5YSIsImlwYWRkciI6IjE2Ny4yMjAuMC4xOTkiLCJuYW1lIjoiTmF2eWEgQ2FudW1hbGxhIiwib2lkIjoiZDVlOTc5YzctM2QyZC00MmFmLThmMzAtNzI3ZGQ0YzJkMzgzIiwib25wcmVtX3NpZCI6IlMtMS01LTIxLTIxMjc1MjExODQtMTYwNDAxMjkyMC0xODg3OTI3NTI3LTI2MTE4NDg0IiwicGxhdGYiOiIxNCIsInB1aWQiOiIxMDAzM0ZGRkEwNkQxN0M5Iiwic2NwIjoiVXNlci5SZWFkIiwic3ViIjoibWtMMHBiLXlpMXQ1ckRGd2JTZ1JvTWxrZE52b3UzSjNWNm84UFE3alVCRSIsInRpZCI6IjcyZjk4OGJmLTg2ZjEtNDFhZi05MWFiLTJkN2NkMDExZGI0NyIsInVuaXF1ZV9uYW1lIjoibmFjYW51bWFAbWljcm9zb2Z0LmNvbSIsInVwbiI6Im5hY2FudW1hQG1pY3Jvc29mdC5jb20iLCJ1dGkiOiJzUVlVekYxdUVVS0NQS0dRTVFVRkFBIiwidmVyIjoiMS4wIn0.Hrn__RGi-HMAzYRyCqX3kBGb6OS7z7y49XPVPpwK_7rJ6nik9E4s6PNY4XkIamJYn7tphpmsHdfM9lQ1gqeeFvFGhweIACsNBWhJ9Nx4dvQnGRkqZ17KnF_wf_QLcyOrOWpUxdSD_oPKcPS-Qr5AFkjw0t7GOKLY-Xw3QLJhzeKmYuuOkmMDJDAl0eNDbH0HiCh3g189a176BfyaR0MgK8wrXI_6MTnFSVfBePqklQeLhcr50YTBfWg3Svgl6MuK_g1hOuaO-XpjUxpdv5dZ0SvI47fAuVDdpCE48igCX5VMj4KUVytDIf6T78aIXMkYHGgW3-xAmuSyYH_Fr0yVAQ
```

## <a name="gaining-consent-for-the-middle-tier-application"></a>Obtener consentimiento para la aplicación de nivel intermedio

Dependiendo de la audiencia para su aplicación, puede considerar estrategias distintas para garantizar que el flujo OBO es correcto. En cualquier caso, el objetivo final es garantizar que se concede el consentimiento adecuado. La forma en que se realiza, sin embargo, depende de qué usuarios admite la aplicación.

### <a name="consent-for-azure-ad-only-applications"></a>Consentimiento para aplicaciones que solo usan Azure AD

#### <a name="default-and-combined-consent"></a>Consentimiento /.default y combinado

Para las aplicaciones que solo necesitan iniciar sesión en cuentas profesionales o educativas, es suficiente con el enfoque tradicional "Aplicaciones cliente conocidas". La aplicación de nivel intermedio agrega el cliente a la lista de aplicaciones cliente conocidas en su manifiesto y, a continuación, el cliente puede desencadenar un flujo de consentimiento combinado tanto para sí mismo como para la aplicación de nivel intermedio. En el punto de conexión de plataforma de identidad de Microsoft, esto se realiza mediante el [ `/.default` ámbito](v2-permissions-and-consent.md#the-default-scope). Cuando se desencadena una pantalla de consentimiento con aplicaciones cliente conocidas y `/.default`, la pantalla de consentimiento mostrará los permisos para el cliente a la API de nivel intermedio y también solicitará los permisos requeridos por la API de nivel intermedio. El usuario da su consentimiento para ambas aplicaciones y, a continuación, el flujo OBO funcionará.

En este momento, el sistema de cuentas personales de Microsoft no admite el consentimiento combinado y, por tanto, este enfoque no funciona para las aplicaciones que desean iniciar sesión específicamente en las cuentas personales. Las cuentas personales de Microsoft utilizadas como cuentas de invitado en un inquilino se controlan con el sistema de Azure AD y pueden admitir un consentimiento combinado.

#### <a name="pre-authorized-applications"></a>Aplicaciones preautorizadas

Una característica del portal de la aplicación es "aplicaciones preautorizadas". De este modo, un recurso puede indicar que una determinada aplicación siempre tiene permiso para recibir determinados ámbitos. Esto es principalmente útil para realizar conexiones más directas entre un cliente front-end y un recurso back-end. Un recurso puede declarar varias aplicaciones preautorizadas: cualquier aplicación puede solicitar estos permisos en un flujo OBO y recibirlos sin necesidad de que el usuario dé el consentimiento.

#### <a name="admin-consent"></a>Consentimiento del administrador

Un administrador de inquilinos puede garantizar que las aplicaciones tienen permiso para llamar a las API necesarias al proporcionar el consentimiento del administrador para la aplicación de nivel intermedio. Para ello, el administrador puede encontrar la aplicación de nivel intermedio en su inquilino, abrir la página de los permisos necesarios y elegir la opción para dar permiso a la aplicación. Para más información sobre el consentimiento del administrador, vea la [documentación sobre consentimientos y permisos](v2-permissions-and-consent.md).

### <a name="consent-for-azure-ad--microsoft-account-applications"></a>Consentimiento para aplicaciones de cuentas de Microsoft + Azure AD

Debido a restricciones en el modelo de permisos para las cuentas personales y la falta de un inquilino de gobierno, los requisitos de consentimiento para las cuentas personales son un poco diferentes de Azure AD. No existe ningún inquilino para el que dar consentimiento a nivel de inquilino ni existe la posibilidad de aplicar un consentimiento combinado. Por tanto, se presentan otras estrategias; tenga en cuenta, que estas funcionan para aplicaciones que solo necesitan admitir también cuentas de Azure AD.

#### <a name="use-of-a-single-application"></a>Uso de una sola aplicación

En algunos escenarios, solo puede haber un único emparejamiento entre el nivel intermedio y el cliente front-end. En este escenario, le resultará más fácil hacer esto en una sola aplicación, sin la necesidad de que exista una aplicación de nivel intermedio. Para realizar la autenticación entre el cliente front-end y la API web, puede usar cookies, un token de identificador o un token de acceso solicitado para la propia aplicación. A continuación, solicite el consentimiento a esta aplicación única para el recurso back-end.

## <a name="client-limitations"></a>Limitaciones del cliente

Si un cliente usa el flujo implícito para obtener un id_token, y que el cliente también tiene los caracteres comodín en una dirección URL de respuesta, no se puede usar el id_token para un flujo delegada.  Sin embargo, los tokens de acceso adquiridos mediante del flujo de concesión implícita todavía pueden ser canjeados por un cliente confidencial, aunque el cliente iniciador tenga registrada una dirección URL de respuesta con caracteres comodín.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre el protocolo OAuth 2.0 y conozca otra manera de realizar la autenticación entre servicios con las credenciales del cliente.

* [Concesión las credenciales del cliente de OAuth 2.0 en la plataforma Microsoft identity](v2-oauth2-client-creds-grant-flow.md)
* [Flujo de código de OAuth 2.0 en la plataforma Microsoft identity](v2-oauth2-auth-code-flow.md)
* [Uso del ámbito `/.default`](v2-permissions-and-consent.md#the-default-scope)
