---
title: 'Solicitud de token de acceso: Azure Active Directory B2C | Microsoft Docs'
description: Obtenga información sobre cómo solicitar un token de acceso de Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 05/12/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: be43b74e7128f9b250d25f8bdb2642c6f7b41d2a
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87115541"
---
# <a name="request-an-access-token-in-azure-active-directory-b2c"></a>Solicitud de token de acceso en Azure Active Directory B2C

Un *token de acceso* contiene notificaciones que puede usar en Azure Active Directory B2C (Azure AD B2C) para identificar los permisos concedidos a las API. Al llamar a un servidor de recursos, debe incluirse un token de acceso en la solicitud HTTP. Un token de acceso se indica como **access_token** en las respuestas de Azure AD B2C.

Este artículo muestra cómo solicitar un token de acceso para una aplicación web y una API web. Para más información sobre los tokens en Azure AD B2C, consulte [Información general de tokens de Azure Active Directory B2C](tokens-overview.md).

> [!NOTE]
> **Las cadenas de la API web (en nombre de) no son compatibles con Azure AD B2C.** - Muchas arquitecturas incluyen una API web que necesita llamar a otra API web de nivel inferior, ambas protegidas mediante Azure AD B2C. Este escenario es común en clientes que tienen un back-end de API web, que a su vez llama a otro servicio. Este escenario de API web encadenadas puede admitirse mediante la concesión de credenciales de portador JWT de OAuth 2.0, también conocido como flujo "en nombre de". Sin embargo, el flujo "en nombre de" no está implementado actualmente en Azure AD B2C.

## <a name="prerequisites"></a>Prerrequisitos

- [Cree un flujo de usuario](tutorial-create-user-flows.md) para permitir a los usuarios registrarse e iniciar sesión en la aplicación.
- Si aún no lo ha hecho, [agregue una aplicación de API web al inquilino de Azure Active Directory B2C](add-web-api-application.md).

## <a name="scopes"></a>Ámbitos

Los ámbitos proporcionan una manera de administrar permisos para los recursos protegidos. Cuando se selecciona un token de acceso, la aplicación cliente debe especificar los permisos que desea en el parámetro **scope** de la solicitud. Por ejemplo, para determinar el **Valor de ámbito** de `read` de la API que tiene el **identificador URI de id. de aplicación** de `https://contoso.onmicrosoft.com/api`, el ámbito sería `https://contoso.onmicrosoft.com/api/read`.

La API web utiliza los ámbitos para implementar el control de acceso basado en el ámbito. Por ejemplo, los usuarios de la API web pueden tener ambos accesos de lectura y de escritura, o pueden tener solo acceso de lectura. Para adquirir varios permisos en la misma solicitud, puede agregar varias entradas en el parámetro **scope** único de la solicitud, separadas por espacios.

El ejemplo siguiente muestra ámbitos descodificados en una dirección URL:

```
scope=https://contoso.onmicrosoft.com/api/read openid offline_access
```

El ejemplo siguiente muestra ámbitos codificados en una dirección URL:

```
scope=https%3A%2F%2Fcontoso.onmicrosoft.com%2Fapi%2Fread%20openid%20offline_access
```

Si solicita más ámbitos que los que se conceden para la aplicación cliente, la llamada se realiza correctamente si se concede al menos un permiso. La notificación **scp** del token de acceso resultante se rellena solo con los permisos que se concedieron correctamente. El estándar OpenID Connect especifica varios valores de ámbito especiales. Los ámbitos siguientes representan el permiso para acceder al perfil del usuario:

- **openid**: solicita un token de identificador.
- **offline_access**: solicita un token de actualización con [flujos de código de autenticación](authorization-code-flow.md).

Si el parámetro **response_type** de una solicitud de `/authorize` incluye `token`, el parámetro **scope** debe incluir al menos un ámbito de recurso (distinto de `openid` y `offline_access`) para su concesión. En caso contrario, se produce un error en la solicitud `/authorize`.

## <a name="request-a-token"></a>Solicitud de un token

Para solicitar un token de acceso, necesita un código de autorización. A continuación se muestra un ejemplo de solicitud de código de autorización al punto de conexión `/authorize`. Los dominios personalizados no se admiten para usarse con tokens de acceso. Use su dominio tenant-name.onmicrosoft.com en la URL de solicitud.

En el ejemplo siguiente, reemplace estos valores:

- `<tenant-name>`: el nombre de su inquilino de Azure AD B2C.
- `<policy-name>`: el nombre de su directiva o flujo de usuarios personalizados.
- `<application-ID>`: el identificador de la aplicación web que ha registrado para admitir el flujo de usuario.
- `<redirect-uri>`: el **identificador URI de redirección** que especificó al registrar la aplicación cliente.

```http
GET https://<tenant-name>.b2clogin.com/tfp/<tenant-name>.onmicrosoft.com/<policy-name>/oauth2/v2.0/authorize?
client_id=<application-ID>
&nonce=anyRandomValue
&redirect_uri=https://jwt.ms
&scope=https://<tenant-name>.onmicrosoft.com/api/read
&response_type=code
```

La respuesta con el código de autorización debe ser similar a este ejemplo:

```
https://jwt.ms/?code=eyJraWQiOiJjcGltY29yZV8wOTI1MjAxNSIsInZlciI6IjEuMC...
```

Después de recibir correctamente el código de autorización, puede usarlo para solicitar un token de acceso:

```http
POST <tenant-name>.onmicrosoft.com/<policy-name>/oauth2/v2.0/token HTTP/1.1
Host: <tenant-name>.b2clogin.com
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&client_id=<application-ID>
&scope=https://<tenant-name>.onmicrosoft.com/api/read
&code=eyJraWQiOiJjcGltY29yZV8wOTI1MjAxNSIsInZlciI6IjEuMC...
&redirect_uri=https://jwt.ms
&client_secret=2hMG2-_:y12n10vwH...
```

Debe ver algo parecido a la siguiente respuesta:

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsImtpZCI6Ilg1ZVhrN...",
    "token_type": "Bearer",
    "not_before": 1549647431,
    "expires_in": 3600,
    "expires_on": 1549651031,
    "resource": "f2a76e08-93f2-4350-833c-965c02483b11",
    "profile_info": "eyJ2ZXIiOiIxLjAiLCJ0aWQiOiJjNjRhNGY3ZC0zMDkxLTRjNzMtYTcyMi1hM2YwNjk0Z..."
}
```

Cuando se usa https://jwt.ms para examinar el token de acceso devuelto, debe ver algo similar al ejemplo siguiente:

```json
{
  "typ": "JWT",
  "alg": "RS256",
  "kid": "X5eXk4xyojNFum1kl2Ytv8dl..."
}.{
  "iss": "https://contoso0926tenant.b2clogin.com/c64a4f7d-3091-4c73-a7.../v2.0/",
  "exp": 1549651031,
  "nbf": 1549647431,
  "aud": "f2a76e08-93f2-4350-833c-965...",
  "oid": "1558f87f-452b-4757-bcd1-883...",
  "sub": "1558f87f-452b-4757-bcd1-883...",
  "name": "David",
  "tfp": "B2C_1_signupsignin1",
  "nonce": "anyRandomValue",
  "scp": "read",
  "azp": "38307aee-303c-4fff-8087-d8d2...",
  "ver": "1.0",
  "iat": 1549647431
}.[Signature]
```

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre cómo [configurar tokens en Azure AD B2C](configure-tokens.md)
