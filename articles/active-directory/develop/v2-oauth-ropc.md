---
title: Inicio de sesión con la concesión de credenciales de contraseña del propietario del recurso | Azure
titleSuffix: Microsoft identity platform
description: Admita flujos de autenticación sin explorador mediante la concesión de credenciales de contraseña de propietario de recursos (ROPC).
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 05/18/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 24d50635efb4d7fe18db9836311cf0a85dfcc734
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88118627"
---
# <a name="microsoft-identity-platform-and-oauth-20-resource-owner-password-credentials"></a>Plataforma de identidad de Microsoft y credenciales de contraseña de propietario de recursos de OAuth 2.0

La Plataforma de identidad de Microsoft admite la [concesión de credenciales de contraseña de propietario de recursos (ROPC) de OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-4.3), que permite que, para que una aplicación inicie la sesión del usuario, se pueda controlar directamente su contraseña.  En este artículo se describe cómo programar directamente con el protocolo de la aplicación.  Cuando sea posible, se recomienda usar las bibliotecas de autenticación de Microsoft (MSAL) admitidas, en lugar de [adquirir tokens y API web protegidas por llamadas](authentication-flows-app-scenarios.md#scenarios-and-supported-authentication-flows).  Además, eche un vistazo a las [aplicaciones de ejemplo que usan MSAL](sample-v2-code.md).

> [!WARNING]
> Microsoft recomienda que _no_ use el flujo de ROPC. En la mayoría de los escenarios, hay alternativas más seguras y recomendables. Este flujo requiere un alto grado de confianza en la aplicación y conlleva riesgos que no están presentes en otros flujos. Solo debe usar este flujo cuando no se puedan usar otros más seguros.

> [!IMPORTANT]
>
> * El punto de conexión de la Plataforma de identidad de Microsoft solo admite ROPC para inquilinos de Azure AD, no cuentas personales. Esto significa que debe usar un punto de conexión específico del inquilino (`https://login.microsoftonline.com/{TenantId_or_Name}`) o el punto de conexión `organizations`.
> * Las cuentas personales que se invitan a un inquilino de Azure AD no pueden usar ROPC.
> * Las cuentas sin contraseña no pueden iniciar sesión a través de ROPC. En este escenario, se recomienda usar en su lugar un flujo distinto para la aplicación.
> * Si los usuarios deben usar la [autenticación multifactor (MFA)](../authentication/concept-mfa-howitworks.md) para iniciar sesión en la aplicación, se les bloqueará.
> * ROPC no se admite en escenarios de [federación de identidades híbridas](../hybrid/whatis-fed.md) (por ejemplo, Azure AD y ADFS que se usan para autenticar cuentas locales). Si los usuarios se redirigen a página completa a proveedores de identidades locales, Azure AD no puede probar el nombre de usuario y la contraseña en el proveedor de identidades. Sin embargo, la [autenticación de paso a través](../hybrid/how-to-connect-pta.md) se admite con ROPC.

## <a name="protocol-diagram"></a>Diagrama de protocolo

En el diagrama siguiente se muestra el flujo de ROPC.

![Diagrama que muestra el flujo de credenciales de contraseña de propietario del recurso](./media/v2-oauth2-ropc/v2-oauth-ropc.svg)

## <a name="authorization-request"></a>Solicitud de autorización

El flujo de ROPC es una solicitud única que envía la identificación del cliente y las credenciales del usuario al punto de distribución de emisión (IDP) y después, a cambio, recibe tokens. El cliente debe solicitar la dirección de correo electrónico (UPN) y la contraseña del usuario antes de hacerlo. Inmediatamente después de que una solicitud se realice correctamente, el cliente deberá liberar de manera segura las credenciales del usuario de la memoria. Nunca debe guardarlas.

> [!TIP]
> Pruebe a ejecutar esta solicitud en Postman
> [![Pruebe a ejecutar esta solicitud en Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)


```HTTP
// Line breaks and spaces are for legibility only.  This is a public client, so no secret is required.

POST {tenant}/oauth2/v2.0/token
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&scope=user.read%20openid%20profile%20offline_access
&username=MyUsername@myTenant.com
&password=SuperS3cret
&grant_type=password
```

| Parámetro | Condición | Descripción |
| --- | --- | --- |
| `tenant` | Obligatorio | El inquilino del directorio en el que desea iniciar la sesión del usuario. Puede estar en formato de nombre descriptivo o GUID. Este parámetro no se puede establecer en `common` ni en `consumers`, pero sí se puede establecer en `organizations`. |
| `client_id` | Obligatorio | El identificador de aplicación (cliente) que la página [Azure Portal: Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) asignó a la aplicación. |
| `grant_type` | Obligatorio | Se debe establecer en `password`. |
| `username` | Obligatorio | La dirección de correo electrónico del usuario. |
| `password` | Obligatorio | La contraseña del usuario. |
| `scope` | Recomendado | Una lista de [ámbitos](v2-permissions-and-consent.md) o permisos separada por espacios que requiere la aplicación. En un flujo interactivo, el administrador o el usuario deben dar su consentimiento a estos ámbitos de antemano. |
| `client_secret`| A veces es necesario | Si la aplicación es un cliente público, no se puede incluir el valor de `client_secret` o `client_assertion`.  Si la aplicación es un cliente confidencial, se debe incluir. |
| `client_assertion` | A veces es necesario | Una forma diferente de `client_secret`, que se genera mediante un certificado.  Consulte [Credenciales de certificado](active-directory-certificate-credentials.md) para más información. |

### <a name="successful-authentication-response"></a>Respuesta de autenticación correcta

En el ejemplo siguiente se muestra una respuesta de token correcta:

```json
{
    "token_type": "Bearer",
    "scope": "User.Read profile openid email",
    "expires_in": 3599,
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD..."
}
```

| Parámetro | Formato | Descripción |
| --------- | ------ | ----------- |
| `token_type` | String | Siempre se establece en `Bearer`. |
| `scope` | Cadenas separadas por espacios | Si se devolvió un token de acceso, este parámetro muestra los ámbitos para los que es válido el token de acceso. |
| `expires_in`| int | Número de segundos durante los que el token de acceso incluido es válido. |
| `access_token`| Cadena opaca | Se emite para los [ámbitos](v2-permissions-and-consent.md) solicitados. |
| `id_token` | JWT | Se emite si el parámetro `scope` original incluye el ámbito `openid`. |
| `refresh_token` | Cadena opaca | Se emite si el parámetro `scope` original incluye `offline_access`. |

Puede usar el token de actualización para adquirir nuevos tokens de acceso y tokens de actualización con el mismo flujo que se describe en la [Documentación del flujo de código de OAuth](v2-oauth2-auth-code-flow.md#refresh-the-access-token).

### <a name="error-response"></a>Respuesta de error

Si el usuario no ha proporcionado la contraseña o el nombre de usuario adecuado o si el cliente no ha recibido el consentimiento solicitado, la autenticación no se realizará.

| Error | Descripción | Acción del cliente |
|------ | ----------- | -------------|
| `invalid_grant` | Error de autenticación | Las credenciales no eran las correctas o el cliente no tiene consentimiento para los ámbitos solicitados. Si no se conceden los ámbitos, se devolverá un error `consent_required`. Si esto ocurre, el cliente debería enviar el usuario a una solicitud interactiva mediante una vista web o un explorador. |
| `invalid_request` | La solicitud se construyó de manera inadecuada. | El tipo de concesión no es compatible con los contextos de autenticación `/common` ni `/consumers`.  Use `/organizations` o un identificador de inquilino en su lugar. |

## <a name="learn-more"></a>Más información

* Pruebe usted mismo ROPC con la [aplicación de consola de ejemplo](https://github.com/azure-samples/active-directory-dotnetcore-console-up-v2).
* Para determinar si debe usar la versión 2.0 del punto de conexión, consulte la información sobre la [Plataforma de identidad de Microsoft](../azuread-dev/azure-ad-endpoint-comparison.md).