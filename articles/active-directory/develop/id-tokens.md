---
title: Referencia de token de Id. de Microsoft identity platform | Microsoft Docs
description: Obtenga información sobre cómo usar los id_tokens emitida por la versión 1.0 de Azure AD y los puntos de conexión de la plataforma (v2.0) de Microsoft identity.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/13/2019
ms.author: ryanwi
ms.reviewer: hirsin
ms.custom: aaddev
ms:custom: fasttrack-edit
ms.collection: M365-identity-device-management
ms.openlocfilehash: f2c99caa46522f9b1e5d6334da8f10a0f4039899
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65540307"
---
# <a name="microsoft-identity-platform-id-tokens"></a>Tokens de identificador de plataforma de identidad de Microsoft

Los `id_tokens` se envían a la aplicación cliente como parte de un flujo de [OpenID Connect](v1-protocols-openid-connect-code.md). Se pueden enviar con o en lugar de un token de acceso y el cliente los utiliza para autenticar al usuario.

## <a name="using-the-idtoken"></a>Uso del id_token

Los Tokens de identificador debe usarse para validar que un usuario es quien dice ser y obtener información útil adicional sobre ellos: no debe usarse para la autorización en lugar de un [token de acceso](access-tokens.md). Las notificaciones que proporciona pueden usarse para la experiencia del usuario en la aplicación, seleccionar una base de datos y proporcionar acceso a la aplicación cliente.

## <a name="claims-in-an-idtoken"></a>Notificaciones de un id_token

Los `id_tokens` de una identidad de Microsoft son elementos [JWT](https://tools.ietf.org/html/rfc7519), lo que significa que están formados por una parte de encabezado, carga y firma. Puede usar el encabezado y la firma para comprobar la autenticidad del token, mientras que la carga contiene la información sobre el usuario que solicitó el cliente. Excepto donde se indique, todas las notificaciones que se muestran aquí aparecen en los tokens de las versiones 1.0 y 2.0.

### <a name="v10"></a>v1.0

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IjdfWnVmMXR2a3dMeFlhSFMzcTZsVWpVWUlHdyIsImtpZCI6IjdfWnVmMXR2a3dMeFlhSFMzcTZsVWpVWUlHdyJ9.eyJhdWQiOiJiMTRhNzUwNS05NmU5LTQ5MjctOTFlOC0wNjAxZDBmYzljYWEiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC9mYTE1ZDY5Mi1lOWM3LTQ0NjAtYTc0My0yOWYyOTU2ZmQ0MjkvIiwiaWF0IjoxNTM2Mjc1MTI0LCJuYmYiOjE1MzYyNzUxMjQsImV4cCI6MTUzNjI3OTAyNCwiYWlvIjoiQVhRQWkvOElBQUFBcXhzdUIrUjREMnJGUXFPRVRPNFlkWGJMRDlrWjh4ZlhhZGVBTTBRMk5rTlQ1aXpmZzN1d2JXU1hodVNTajZVVDVoeTJENldxQXBCNWpLQTZaZ1o5ay9TVTI3dVY5Y2V0WGZMT3RwTnR0Z2s1RGNCdGsrTExzdHovSmcrZ1lSbXY5YlVVNFhscGhUYzZDODZKbWoxRkN3PT0iLCJhbXIiOlsicnNhIl0sImVtYWlsIjoiYWJlbGlAbWljcm9zb2Z0LmNvbSIsImZhbWlseV9uYW1lIjoiTGluY29sbiIsImdpdmVuX25hbWUiOiJBYmUiLCJpZHAiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMWRiNDcvIiwiaXBhZGRyIjoiMTMxLjEwNy4yMjIuMjIiLCJuYW1lIjoiYWJlbGkiLCJub25jZSI6IjEyMzUyMyIsIm9pZCI6IjA1ODMzYjZiLWFhMWQtNDJkNC05ZWMwLTFiMmJiOTE5NDQzOCIsInJoIjoiSSIsInN1YiI6IjVfSjlyU3NzOC1qdnRfSWN1NnVlUk5MOHhYYjhMRjRGc2dfS29vQzJSSlEiLCJ0aWQiOiJmYTE1ZDY5Mi1lOWM3LTQ0NjAtYTc0My0yOWYyOTU2ZmQ0MjkiLCJ1bmlxdWVfbmFtZSI6IkFiZUxpQG1pY3Jvc29mdC5jb20iLCJ1dGkiOiJMeGVfNDZHcVRrT3BHU2ZUbG40RUFBIiwidmVyIjoiMS4wIn0=.UJQrCA6qn2bXq57qzGX_-D3HcPHqBMOKDPx4su1yKRLNErVD8xkxJLNLVRdASHqEcpyDctbdHccu6DPpkq5f0ibcaQFhejQNcABidJCTz0Bb2AbdUCTqAzdt9pdgQvMBnVH1xk3SCM6d4BbT4BkLLj10ZLasX7vRknaSjE_C5DI7Fg4WrZPwOhII1dB0HEZ_qpNaYXEiy-o94UJ94zCr07GgrqMsfYQqFR7kn-mn68AjvLcgwSfZvyR_yIK75S_K37vC3QryQ7cNoafDe9upql_6pB2ybMVlgWPs_DmbJ8g0om-sPlwyn74Cc1tW3ze-Xptw_2uVdPgWyqfuWAfq6Q
```

Puede ver este token de ejemplo de la versión v1.0 en [jwt.ms](https://jwt.ms/#id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6IjdfWnVmMXR2a3dMeFlhSFMzcTZsVWpVWUlHdyIsImtpZCI6IjdfWnVmMXR2a3dMeFlhSFMzcTZsVWpVWUlHdyJ9.eyJhdWQiOiJiMTRhNzUwNS05NmU5LTQ5MjctOTFlOC0wNjAxZDBmYzljYWEiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC9mYTE1ZDY5Mi1lOWM3LTQ0NjAtYTc0My0yOWYyOTU2ZmQ0MjkvIiwiaWF0IjoxNTM2Mjc1MTI0LCJuYmYiOjE1MzYyNzUxMjQsImV4cCI6MTUzNjI3OTAyNCwiYWlvIjoiQVhRQWkvOElBQUFBcXhzdUIrUjREMnJGUXFPRVRPNFlkWGJMRDlrWjh4ZlhhZGVBTTBRMk5rTlQ1aXpmZzN1d2JXU1hodVNTajZVVDVoeTJENldxQXBCNWpLQTZaZ1o5ay9TVTI3dVY5Y2V0WGZMT3RwTnR0Z2s1RGNCdGsrTExzdHovSmcrZ1lSbXY5YlVVNFhscGhUYzZDODZKbWoxRkN3PT0iLCJhbXIiOlsicnNhIl0sImVtYWlsIjoiYWJlbGlAbWljcm9zb2Z0LmNvbSIsImZhbWlseV9uYW1lIjoiTGluY29sbiIsImdpdmVuX25hbWUiOiJBYmUiLCJpZHAiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83MmY5ODhiZi04NmYxLTQxYWYtOTFhYi0yZDdjZDAxMWRiNDcvIiwiaXBhZGRyIjoiMTMxLjEwNy4yMjIuMjIiLCJuYW1lIjoiYWJlbGkiLCJub25jZSI6IjEyMzUyMyIsIm9pZCI6IjA1ODMzYjZiLWFhMWQtNDJkNC05ZWMwLTFiMmJiOTE5NDQzOCIsInJoIjoiSSIsInN1YiI6IjVfSjlyU3NzOC1qdnRfSWN1NnVlUk5MOHhYYjhMRjRGc2dfS29vQzJSSlEiLCJ0aWQiOiJmYTE1ZDY5Mi1lOWM3LTQ0NjAtYTc0My0yOWYyOTU2ZmQ0MjkiLCJ1bmlxdWVfbmFtZSI6IkFiZUxpQG1pY3Jvc29mdC5jb20iLCJ1dGkiOiJMeGVfNDZHcVRrT3BHU2ZUbG40RUFBIiwidmVyIjoiMS4wIn0=.UJQrCA6qn2bXq57qzGX_-D3HcPHqBMOKDPx4su1yKRLNErVD8xkxJLNLVRdASHqEcpyDctbdHccu6DPpkq5f0ibcaQFhejQNcABidJCTz0Bb2AbdUCTqAzdt9pdgQvMBnVH1xk3SCM6d4BbT4BkLLj10ZLasX7vRknaSjE_C5DI7Fg4WrZPwOhII1dB0HEZ_qpNaYXEiy-o94UJ94zCr07GgrqMsfYQqFR7kn-mn68AjvLcgwSfZvyR_yIK75S_K37vC3QryQ7cNoafDe9upql_6pB2ybMVlgWPs_DmbJ8g0om-sPlwyn74Cc1tW3ze-Xptw_2uVdPgWyqfuWAfq6Q).

### <a name="v20"></a>v2.0

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IjFMVE16YWtpaGlSbGFfOHoyQkVKVlhlV01xbyJ9.eyJ2ZXIiOiIyLjAiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vOTE4ODA0MGQtNmM2Ny00YzViLWIxMTItMzZhMzA0YjY2ZGFkL3YyLjAiLCJzdWIiOiJBQUFBQUFBQUFBQUFBQUFBQUFBQUFJa3pxRlZyU2FTYUZIeTc4MmJidGFRIiwiYXVkIjoiNmNiMDQwMTgtYTNmNS00NmE3LWI5OTUtOTQwYzc4ZjVhZWYzIiwiZXhwIjoxNTM2MzYxNDExLCJpYXQiOjE1MzYyNzQ3MTEsIm5iZiI6MTUzNjI3NDcxMSwibmFtZSI6IkFiZSBMaW5jb2xuIiwicHJlZmVycmVkX3VzZXJuYW1lIjoiQWJlTGlAbWljcm9zb2Z0LmNvbSIsIm9pZCI6IjAwMDAwMDAwLTAwMDAtMDAwMC02NmYzLTMzMzJlY2E3ZWE4MSIsInRpZCI6IjMzMzgwNDBkLTZjNjctNGM1Yi1iMTEyLTM2YTMwNGI2NmRhZCIsIm5vbmNlIjoiMTIzNTIzIiwiYWlvIjoiRGYyVVZYTDFpeCFsTUNXTVNPSkJjRmF0emNHZnZGR2hqS3Y4cTVnMHg3MzJkUjVNQjVCaXN2R1FPN1lXQnlqZDhpUURMcSFlR2JJRGFreXA1bW5PcmNkcUhlWVNubHRlcFFtUnA2QUlaOGpZIn0=.1AFWW-Ck5nROwSlltm7GzZvDwUkqvhSQpm55TQsmVo9Y59cLhRXpvB8n-55HCr9Z6G_31_UbeUkoz612I2j_Sm9FFShSDDjoaLQr54CreGIJvjtmS3EkK9a7SJBbcpL1MpUtlfygow39tFjY7EVNW9plWUvRrTgVk7lYLprvfzw-CIqw3gHC-T7IK_m_xkr08INERBtaecwhTeN4chPC4W3jdmw_lIxzC48YoQ0dB1L9-ImX98Egypfrlbm0IBL5spFzL6JDZIRRJOu8vecJvj1mq-IUhGt0MacxX8jdxYLP-KUu2d9MbNKpCKJuZ7p8gwTL5B7NlUdh_dmSviPWrw
```

Puede ver este token de ejemplo de la versión v2.0 en [jwt.ms](https://jwt.ms/#id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6IjFMVE16YWtpaGlSbGFfOHoyQkVKVlhlV01xbyJ9.eyJ2ZXIiOiIyLjAiLCJpc3MiOiJodHRwczovL2xvZ2luLm1pY3Jvc29mdG9ubGluZS5jb20vOTE4ODA0MGQtNmM2Ny00YzViLWIxMTItMzZhMzA0YjY2ZGFkL3YyLjAiLCJzdWIiOiJBQUFBQUFBQUFBQUFBQUFBQUFBQUFJa3pxRlZyU2FTYUZIeTc4MmJidGFRIiwiYXVkIjoiNmNiMDQwMTgtYTNmNS00NmE3LWI5OTUtOTQwYzc4ZjVhZWYzIiwiZXhwIjoxNTM2MzYxNDExLCJpYXQiOjE1MzYyNzQ3MTEsIm5iZiI6MTUzNjI3NDcxMSwibmFtZSI6IkFiZSBMaW5jb2xuIiwicHJlZmVycmVkX3VzZXJuYW1lIjoiQWJlTGlAbWljcm9zb2Z0LmNvbSIsIm9pZCI6IjAwMDAwMDAwLTAwMDAtMDAwMC02NmYzLTMzMzJlY2E3ZWE4MSIsInRpZCI6IjMzMzgwNDBkLTZjNjctNGM1Yi1iMTEyLTM2YTMwNGI2NmRhZCIsIm5vbmNlIjoiMTIzNTIzIiwiYWlvIjoiRGYyVVZYTDFpeCFsTUNXTVNPSkJjRmF0emNHZnZGR2hqS3Y4cTVnMHg3MzJkUjVNQjVCaXN2R1FPN1lXQnlqZDhpUURMcSFlR2JJRGFreXA1bW5PcmNkcUhlWVNubHRlcFFtUnA2QUlaOGpZIn0.1AFWW-Ck5nROwSlltm7GzZvDwUkqvhSQpm55TQsmVo9Y59cLhRXpvB8n-55HCr9Z6G_31_UbeUkoz612I2j_Sm9FFShSDDjoaLQr54CreGIJvjtmS3EkK9a7SJBbcpL1MpUtlfygow39tFjY7EVNW9plWUvRrTgVk7lYLprvfzw-CIqw3gHC-T7IK_m_xkr08INERBtaecwhTeN4chPC4W3jdmw_lIxzC48YoQ0dB1L9-ImX98Egypfrlbm0IBL5spFzL6JDZIRRJOu8vecJvj1mq-IUhGt0MacxX8jdxYLP-KUu2d9MbNKpCKJuZ7p8gwTL5B7NlUdh_dmSviPWrw).

### <a name="header-claims"></a>Notificaciones de encabezado

|Notificación | Formato | DESCRIPCIÓN |
|-----|--------|-------------|
|`typ` | Cadena: siempre "JWT" | Indica que el token es un token JWT.|
|`alg` | string | Indica el algoritmo que se usó para firmar el token. Ejemplo: "RS256" |
|`kid` | String | Huella digital de la clave pública utilizada para firmar este token. Se emite en los `id_tokens` de las versiones 1.0 y 2.0. |
|`x5t` | String | Es igual (en uso y valor) a `kid`. Sin embargo, esta es una notificación heredada emitida solo en los `id_tokens` de la versión 1.0 para fines de compatibilidad. |

### <a name="payload-claims"></a>Notificaciones de carga

Esta lista muestra las notificaciones que se encuentran en la mayoría de los id_tokens (excepto donde se indique) de forma predeterminada.  Sin embargo, puede usar la aplicación [notificaciones opcionales](active-directory-optional-claims.md) para solicitar notificaciones adicionales en el id_token.  Estas pueden ir desde el `groups` de notificación para obtener información sobre el nombre del usuario.

|Notificación | Formato | DESCRIPCIÓN |
|-----|--------|-------------|
|`aud` |  Cadena, un URI de identificador de aplicación | Identifica al destinatario previsto del token. En los `id_tokens`, la audiencia es el identificador de aplicación de la aplicación, asignado a la aplicación en Azure Portal. La aplicación debe validar este valor y rechazar el token si el valor no coincide. |
|`iss` |  Cadena, un identificador URI de STS | Identifica el servicio de token de seguridad (STS) que construye y devuelve el token, así como el inquilino de Azure AD en el que se autenticó al usuario. Si el token fue emitido por el punto de conexión de la versión 2.0, el identificador URI finalizará con `/v2.0`.  El GUID que indica que el usuario es un usuario consumidor desde una cuenta de Microsoft es `9188040d-6c67-4c5b-b112-36a304b66dad`. La aplicación debe usar la parte del GUID de la notificación para restringir el conjunto de inquilinos que pueden iniciar sesión en la aplicación, si procede. |
|`iat` |  entero, una marca de tiempo de UNIX | La notificación "iat" (emitido a las) indica cuándo se produjo la autenticación de este token.  |
|`idp`|Cadena, normalmente un identificador URI de STS | Registra el proveedor de identidades que autenticó al firmante del token. Este valor es idéntico al valor de la notificación del emisor, a menos que la cuenta de usuario no esté en el mismo inquilino que el emisor: los invitados, por ejemplo. Si la notificación no está presente, significa que el valor de `iss` se puede usar en su lugar.  Para las cuentas personales que se usan en un contexto de la organización (por ejemplo, una cuenta personal invitada a un inquilino de Azure AD), la notificación `idp` puede ser "live.com" o un identificador URI de STS que contenga al inquilino de la cuenta Microsoft `9188040d-6c67-4c5b-b112-36a304b66dad`. |
|`nbf` |  entero, una marca de tiempo de UNIX | La notificación "nbf" (no antes de) identifica la hora antes de la cual no debe ser aceptado el token JWT para su procesamiento.|
|`exp` |  entero, una marca de tiempo de UNIX | La notificación "exp" (fecha de expiración) identifica la hora de expiración en la que o después de la que el token JWT no debe ser aceptado para su procesamiento.  Es importante tener en cuenta que un recurso puede rechazar el token antes de esta hora: si, por ejemplo, es necesario un cambio en la autenticación o se ha detectado una revocación del token. |
| `c_hash`| string |El hash de código se incluye en los tokens de identificador solo cuando se emite el token de identificador con un código de autorización de OAuth 2.0. Se puede usar para validar la autenticidad de un código de autorización. Para detalles sobre cómo realizar esta validación, consulte la [especificación de OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html). |
|`at_hash`| String |El hash de token de acceso se incluye en los tokens de identificador solo cuando el token de identificador se emite con un token de acceso de OAuth 2.0. Se puede usar para validar la autenticidad de un token de acceso. Para detalles sobre cómo realizar esta validación, consulte la [especificación de OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html). |
|`aio` | Cadena opaca | Una notificación interna usada por Azure AD para registrar los datos para la reutilización de tokens. Se debe omitir.|
|`preferred_username` | String | El nombre de usuario principal que representa al usuario. Puede ser una dirección de correo electrónico, un número de teléfono o un nombre de usuario genérico sin un formato especificado. Su valor es mutable y podría cambiar en el tiempo. Puesto que es mutable, este valor no debe usarse para tomar decisiones de autorización. El `profile` ámbito se requiere para recibir esta notificación.|
|`email` | String | La notificación `email` está presente de manera predeterminada para las cuentas de invitado que tengan una dirección de correo electrónico.  La aplicación puede solicitar la notificación de correo electrónico para los usuarios administrados (del mismo inquilino que el recurso) mediante la [notificación opcional](active-directory-optional-claims.md) `email`.  En el punto de conexión v2.0, la aplicación también puede solicitar el ámbito `email` de OpenID Connect. No es necesario que solicite ambos, la notificación opcional y el ámbito para obtener la notificación.  La notificación de correo electrónico solo es compatible con correo direccionable desde la información del perfil del usuario. |
|`name` | String | La notificación `name` proporciona un valor en lenguaje natural que identifica al firmante del token. El valor no está garantizado que es único, es mutable y está diseñado para usarse solo con fines de visualización. El `profile` ámbito se requiere para recibir esta notificación. |
|`nonce`| string | El valor nonce coincide con el parámetro incluido en la solicitud /authorize original al IDP. Si no coincide, la aplicación debe rechazar el token. |
|`oid` | Cadena, un identificador GUID | El identificador inmutable de un objeto en el sistema de identidades Microsoft, en este caso, una cuenta de usuario. Este identificador identifica de forma única el usuario entre aplicaciones: dos aplicaciones diferentes que inician sesión con el mismo usuario recibirán el mismo valor en la notificación `oid`. Microsoft Graph devuelve este identificador como la propiedad `id` para una cuenta de usuario determinada. Dado que el `oid` permite que varias aplicaciones pongan en correlación los usuarios, el `profile` ámbito se requiere para recibir esta notificación. Tenga en cuenta que si un usuario existe en varios inquilinos, el usuario contendrá un identificador de objeto distinto en cada inquilino, se consideran cuentas diferentes, incluso si el usuario inicia sesión en todas las cuentas con las mismas credenciales. |
|`roles`| Matriz de cadenas | El conjunto de roles que se asignaron al usuario que ha iniciado sesión. |
|`rh` | Cadena opaca |Una notificación interna que Azure usa para volver a validar los tokens. Se debe omitir. |
|`sub` | Cadena, un identificador GUID | La entidad de seguridad sobre la que el token declara información como, por ejemplo, el usuario de una aplicación. Este valor es inmutable y no se puede reasignar ni volver a usar. El firmante es un identificador en pares: es único para un identificador de aplicación determinado. Si un usuario inicia sesión en dos aplicaciones diferentes con dos identificadores de cliente diferente, esas aplicaciones recibirán dos valores diferentes para la notificación del firmante. Esto puede o no es posible que se desee dependiendo de los requisitos de arquitectura y privacidad. |
|`tid` | Cadena, un identificador GUID | Un GUID que representa el inquilino de Azure AD de donde proviene el usuario. En el caso de las cuentas profesionales y educativas, el GUID es el identificador del inquilino inmutable de la organización a la que pertenece el usuario. En el caso de las cuentas personales, el valor es `9188040d-6c67-4c5b-b112-36a304b66dad`. El `profile` ámbito se requiere para recibir esta notificación. |
|`unique_name` | String | Proporciona un valor en lenguaje natural que identifica al firmante del token. Este valor no está garantizado que sea único dentro de un inquilino y debe usarse solo con fines de visualización. Solo se emite en los `id_tokens` de la versión 1.0. |
|`uti` | Cadena opaca | Una notificación interna que Azure usa para volver a validar los tokens. Se debe omitir. |
|`ver` | Cadena, 1.0 o 2.0 | Indica la versión del id_token. |

## <a name="validating-an-idtoken"></a>Validación de un id_token

Validar un `id_token` es similar al primer paso de [validar un token de acceso](access-tokens.md#validating-tokens) : el cliente debe validar que el emisor correcto ha enviado el token y que no se ha manipulado. Dado que los `id_tokens` siempre son un token JWT, existen muchas bibliotecas para validar estos tokens: se recomienda usar una de ellas en lugar de hacerlo usted mismo.

Para validar manualmente el token, consulte los pasos detallados en [Validación de un token de acceso](access-tokens.md#validating-tokens). Después de validar la firma del token, se deben validar las siguientes notificaciones del id_token (esto también se podría hacer con la biblioteca de validación de tokens):

* Marcas de tiempo: las marcas de tiempo `iat`, `nbf` y `exp` deben estar todas antes o después de la hora actual, según corresponda. 
* Audiencia: la notificación `aud` debe coincidir con el identificador de aplicación de la aplicación.
* Nonce: la notificación `nonce` de la carga debe coincidir con el parámetro nonce pasado en el punto de conexión /authorize durante la solicitud inicial.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre [tokens de acceso](access-tokens.md)
* Personalizar las notificaciones del id_token utilizando [notificaciones opcionales](active-directory-optional-claims.md).