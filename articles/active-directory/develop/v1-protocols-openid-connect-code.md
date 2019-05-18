---
title: Descripción del flujo de código de autenticación de OpenID Connect en Azure AD | Microsoft Docs
description: En este artículo se describe cómo utilizar mensajes HTTP para autorizar el acceso a aplicaciones y API web en su inquilino con Azure Active Directory y OpenID Connect.
services: active-directory
documentationcenter: .net
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 29142f7e-d862-4076-9a1a-ecae5bcd9d9b
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/4/2019
ms.author: ryanwi
ms.reviewer: hirsin
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 051d3faf5cea24e33f1e6560abc2d039c1059c91
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784981"
---
# <a name="authorize-access-to-web-applications-using-openid-connect-and-azure-active-directory"></a>Autorización del acceso a aplicaciones web con OpenID Connect y Azure Active Directory

[OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) es una capa de identidad sencilla creada basándose en el protocolo OAuth 2.0. OAuth 2.0 define los mecanismos para obtener y usar [**tokens de acceso**](access-tokens.md) para acceder a recursos protegidos, pero no determina métodos estándares para proporcionar información de identidad. OpenID Connect implementa la autenticación como una extensión al proceso de autorización de OAuth 2.0. Proporciona información sobre el usuario final con el formato [`id_token`](id-tokens.md) que comprueba la identidad del usuario y proporciona información de perfil básica sobre dicho usuario.

Nosotros recomendamos OpenID Connect si va a crear una aplicación web que está hospedada en un servidor y a la que se accede mediante un explorador.


[!INCLUDE [active-directory-protocols-getting-started](../../../includes/active-directory-protocols-getting-started.md)] 

## <a name="authentication-flow-using-openid-connect"></a>Flujo de autenticación con OpenID Connect

El flujo de inicio de sesión más básico contiene los pasos siguientes, cada uno de los cuales se describe en detalle a continuación.

![Flujo de autenticación de OpenID Connect](./media/v1-protocols-openid-connect-code/active-directory-oauth-code-flow-web-app.png)

## <a name="openid-connect-metadata-document"></a>Documento de metadatos de OpenID Connect

OpenID Connect describe un documento de metadatos que contiene la mayor parte de la información necesaria para que una aplicación realice el inicio de sesión. Esta información incluye las direcciones URL que se usan y la ubicación de las claves de firma pública del servicio. Se puede encontrar el documento de metadatos de OpenID Connect en:

```
https://login.microsoftonline.com/{tenant}/.well-known/openid-configuration
```
Los metadatos son un documento de notación de objetos JavaScript (JSON) simple. Consulte el fragmento de código siguiente para ver un ejemplo. El contenido del fragmento de código se describe detalladamente en la [especificación de OpenID Connect](https://openid.net). Tenga en cuenta que proporciona un identificador de inquilino en lugar de `common` en lugar de {tenant} anterior dará como resultado en los URI específico del inquilino en el objeto JSON devuelto.

```
{
    "authorization_endpoint": "https://login.microsoftonline.com/{tenant}/oauth2/authorize",
    "token_endpoint": "https://login.microsoftonline.com/{tenant}/oauth2/token",
    "token_endpoint_auth_methods_supported":
    [
        "client_secret_post",
        "private_key_jwt",
        "client_secret_basic"
    ],
    "jwks_uri": "https://login.microsoftonline.com/common/discovery/keys"
    "userinfo_endpoint":"https://login.microsoftonline.com/{tenant}/openid/userinfo",
    ...
}
```

Si la aplicación tiene las claves de firma personalizadas como resultado de utilizar el [asignación de notificaciones](active-directory-claims-mapping.md) característica, debe anexar un `appid` que contiene el identificador de aplicación con el fin de obtener el parámetro de consulta un `jwks_uri` que apunta a la aplicación de la clave de firma información. Por ejemplo: `https://login.microsoftonline.com/{tenant}/.well-known/openid-configuration?appid=6731de76-14a6-49ae-97bc-6eba6914391e` contiene un `jwks_uri` de `https://login.microsoftonline.com/{tenant}/discovery/keys?appid=6731de76-14a6-49ae-97bc-6eba6914391e`.

## <a name="send-the-sign-in-request"></a>Envío de la solicitud de inicio de sesión

Cuando la aplicación web deba autenticar al usuario, debe dirigirlo al punto de conexión `/authorize` . Esta solicitud es similar al primer segmento del [flujo de código de autorización de OAuth 2.0](v1-protocols-oauth-code.md), con algunas diferencias importantes:

* La solicitud debe incluir el ámbito `openid` en el parámetro `scope`.
* El parámetro `response_type` debe incluir `id_token`.
* La solicitud debe incluir el parámetro `nonce` .

Por tanto, una solicitud de muestra tendría el siguiente aspecto:

```
// Line breaks for legibility only

GET https://login.microsoftonline.com/{tenant}/oauth2/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=id_token
&redirect_uri=http%3A%2F%2Flocalhost%3a12345
&response_mode=form_post
&scope=openid
&state=12345
&nonce=7362CAEA-9CA5-4B43-9BA3-34D7C303EBA7
```

| Parámetro |  | DESCRIPCIÓN |
| --- | --- | --- |
| tenant |requerido |El valor `{tenant}` de la ruta de acceso de la solicitud se puede usar para controlar quién puede iniciar sesión en la aplicación. Los valores permitidos son los identificadores de inquilino; por ejemplo, `8eaef023-2b34-4da1-9baa-8bc8c9d6a490`, `contoso.onmicrosoft.com` o `common` para los tokens independientes del inquilino. |
| client_id |requerido |Identificador de aplicación asignado a la aplicación cuando se registra en Azure AD. Puede encontrarlo en Azure Portal. Haga clic en **Azure Active Directory**, haga clic en **registros de aplicaciones**, elija la aplicación y busque el identificador de aplicación en la página de aplicación. |
| response_type |requerido |Debe incluir `id_token` para el inicio de sesión en OpenID Connect. También puede incluir otros elementos response_type como `code` o `token`. |
| ámbito | recomendado | La especificación de OpenID Connect requiere el ámbito `openid`, que se convierte en el permiso "Iniciar sesión" en la IU de consentimiento. Este y otros ámbitos OIDC se omiten en el punto de conexión v1.0, pero sigue siendo una práctica recomendada para los clientes compatibles con los estándares. |
| valor de seguridad |requerido |Un valor incluido en la solicitud que ha generado la aplicación y que se incluye en el elemento `id_token` resultante como una notificación. La aplicación puede comprobar este valor para mitigar los ataques de reproducción de token. Normalmente, el valor es un GUID o una cadena única aleatorios que puede utilizarse para identificar el origen de la solicitud. |
| redirect_uri | recomendado |El redirect_uri de su aplicación, a donde su aplicación puede enviar y recibir las respuestas de autenticación. Debe coincidir exactamente con uno de los redirect_uris que registró en el portal, con la excepción de que debe estar codificado como URL. Si faltan, se enviará al agente de usuario a uno de lo URI de redirección registrados para la aplicación, de forma aleatoria. La longitud máxima es 255 bytes |
| response_mode |opcional |Especifica el método que debe usarse para enviar el authorization_code resultante de nuevo a tu aplicación. Los valores admitidos son `form_post` para *envíos de formulario HTTP* y `fragment` para *fragmentos de dirección URL*. Para las aplicaciones web, se recomienda usar `response_mode=form_post` para asegurar la transferencia más segura de tokens a la aplicación. El valor predeterminado para cualquier flujo que incluye un elemento id_token es `fragment`.|
| state |recomendado |Un valor incluido en la solicitud que se devolverá en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. Normalmente se usa un valor único generado de forma aleatoria para [evitar los ataques de falsificación de solicitudes entre sitios](https://tools.ietf.org/html/rfc6749#section-10.12). El estado también se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, por ejemplo, la página o vista en la que estaban. |
| símbolo del sistema |opcional |Indica el tipo de interacción necesaria con el usuario. Actualmente, los únicos valores válidos son 'login', 'none' y 'consent'. `prompt=login` obliga al usuario a escribir sus credenciales en esa solicitud, negando el inicio de sesión único. `prompt=none` se asegura de que al usuario no se le presenta ninguna solicitud interactiva del tipo que sea. Si la solicitud no se puede completar sin notificaciones mediante el inicio de sesión único, el punto de conexión devuelve un error. `prompt=consent` desencadena el cuadro de diálogo de consentimiento de OAuth después de que el usuario inicia sesión y solicita a este que conceda permisos a la aplicación. |
| login_hint |opcional |Puede usarse para rellenar previamente el campo de nombre de usuario y dirección de correo electrónico de la página de inicio de sesión del usuario, si sabe su nombre de usuario con antelación. A menudo las aplicaciones usan este parámetro durante la reautenticación, dado que ya han extraído el nombre de usuario de un inicio de sesión anterior mediante la notificación `preferred_username`. |

En este punto, se le pide al usuario que escriba sus credenciales y que complete la autenticación.

### <a name="sample-response"></a>Respuesta de muestra

Una respuesta de ejemplo enviada a la `redirect_uri` especificado en la solicitud de inicio de sesión después de que el usuario se ha autenticado, podría tener este aspecto:

```
POST / HTTP/1.1
Host: localhost:12345
Content-Type: application/x-www-form-urlencoded

id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNB...&state=12345
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| ID_token |El `id_token` que solicitó la aplicación. Puede usar el `id_token` para comprobar la identidad del usuario y comenzar una sesión con el usuario. |
| state |Un valor incluido en la solicitud que también se devolverá en la respuesta del token. Normalmente se usa un valor único generado de forma aleatoria para [evitar los ataques de falsificación de solicitudes entre sitios](https://tools.ietf.org/html/rfc6749#section-10.12). El estado también se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, por ejemplo, la página o vista en la que estaban. |

### <a name="error-response"></a>Respuesta de error

Las respuestas de error también pueden enviarse al `redirect_uri` para que la aplicación pueda controlarlas adecuadamente:

```
POST / HTTP/1.1
Host: localhost:12345
Content-Type: application/x-www-form-urlencoded

error=access_denied&error_description=the+user+canceled+the+authentication
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| error |Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| error_description |Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error de autenticación. |

#### <a name="error-codes-for-authorization-endpoint-errors"></a>Códigos de error correspondientes a errores de puntos de conexión de autorización

En la tabla siguiente se describen los distintos códigos de error que pueden devolverse en el parámetro `error` de la respuesta de error.

| Código de error | DESCRIPCIÓN | Acción del cliente |
| --- | --- | --- |
| invalid_request |Error de protocolo, como un parámetro obligatorio que falta. |Corrija el error y vuelva a enviar la solicitud. Se trata de un error de desarrollo que suele detectarse durante las pruebas iniciales. |
| unauthorized_client |La aplicación cliente no puede solicitar un código de autorización. |Este error suele producirse cuando la aplicación cliente no está registrada en Azure AD o no se ha agregado al inquilino de Azure AD del usuario. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |
| access_denied |El propietario del recurso ha denegado el consentimiento. |La aplicación cliente puede notificar al usuario que no puede continuar, salvo que este dé su consentimiento. |
| unsupported_response_type |El servidor de autorización no admite el tipo de respuesta de la solicitud. |Corrija el error y vuelva a enviar la solicitud. Se trata de un error de desarrollo que suele detectarse durante las pruebas iniciales. |
| server_error |El servidor ha detectado un error inesperado. |Vuelva a intentarlo. Estos errores pueden deberse a condiciones temporales. La aplicación cliente podría explicar al usuario que su respuesta se retrasó debido a un error temporal. |
| temporarily_unavailable |De manera temporal, el servidor está demasiado ocupado para atender la solicitud. |Vuelva a intentarlo. La aplicación podría explicar al usuario que su respuesta se retrasó debido a una condición temporal. |
| invalid_resource |El recurso de destino no es válido porque no existe, Azure AD no lo encuentra o no está configurado correctamente. |Este error indica que el recurso, en caso de que exista, no se ha configurado en el inquilino. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |

## <a name="validate-the-idtoken"></a>Validar el id_token

Recibir un solo `id_token` no es suficiente para autenticar al usuario; debe validar la firma del id_token y comprobar las notificaciones del `id_token` según los requisitos de su aplicación. El punto de conexión de Azure AD usa tokens web JSON (JWT) y criptografía de clave pública para firmar los tokens y comprobar que son válidos.

Puede elegir validar el `id_token` en el código de cliente, pero lo habitual es enviar el `id_token` a un servidor back-end y realizar allí la validación. 

Se recomienda que valide notificaciones adicionales según su escenario. Algunas validaciones comunes incluyen:

* Asegurarse de que la organización/el usuario se ha registrado en la aplicación.
* Lo que garantiza el usuario tiene autorización/los privilegios adecuados mediante la `wids` o `roles` notificaciones. 
* Asegurarse de que se haya producido un determinado nivel de autenticación, como la autenticación multifactor.

Cuando haya validado `id_token`, puede iniciar una sesión con el usuario y usar las notificaciones de `id_token` para más información sobre el usuario de la aplicación. Esta información puede usarse para visualización, registros, autorizaciones, etc. Para más información sobre `id_tokens` y notificaciones, lea [Elementos id_tokens de AAD](id-tokens.md).

## <a name="send-a-sign-out-request"></a>Envío de una solicitud de cierre de sesión

Si desea cerrar la sesión del usuario de la aplicación, no es suficiente borrar las cookies de su aplicación; de lo contrario, termine la sesión con el usuario. También debe redirigir al usuario a `end_session_endpoint` para que cierre la sesión. Si no lo hace, el usuario tendrá que volver a autenticar la aplicación sin escribir sus credenciales de nuevo, ya que tienen un inicio de sesión único válido con el punto de conexión de Azure AD.

Simplemente puede redirigir al usuario al `end_session_endpoint` que aparece en el mismo documento de metadatos OpenID Connect:

```
GET https://login.microsoftonline.com/common/oauth2/logout?
post_logout_redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F

```

| Parámetro |  | DESCRIPCIÓN |
| --- | --- | --- |
| post_logout_redirect_uri |recomendado |La dirección URL que se debe redirigir al usuario después de cierre de sesión correcto. Si no se incluye, se muestra un mensaje genérico al usuario. |

## <a name="single-sign-out"></a>Cierre de sesión único

Cuando redirige al usuario a `end_session_endpoint`, Azure AD borra la sesión del usuario del explorador. Sin embargo, el usuario podrá permanecer conectado a otras aplicaciones que utilizan Azure AD para la autenticación. Para permitir que esas aplicaciones cierren la sesión del usuario de manera simultánea, Azure AD envía una solicitud HTTP GET al elemento `LogoutUrl` registrado de todas las aplicaciones en las que tiene una sesión iniciada. Las aplicaciones deben responder a esta solicitud mediante la eliminación de la sesión que identifica al usuario y la devolución de una respuesta `200`. Si quiere admitir el inicio de sesión único en la aplicación, debe implementar este elemento `LogoutUrl` en el código de la aplicación. Puede habilitar `LogoutUrl` desde Azure Portal:

1. Acceda a [Azure Portal](https://portal.azure.com).
2. Para elegir la instancia de Active Directory, haga clic en su cuenta en la esquina superior derecha de la página.
3. En el panel de navegación izquierdo, elija **Azure Active Directory**, a continuación, elija **Registros de aplicaciones** y seleccione la aplicación.
4. Haga clic en **Configuración**, después, en **Propiedades** y busque el cuadro de texto **URL de cierre de sesión**. 

## <a name="token-acquisition"></a>Obtención de tokens
Muchas aplicaciones web no solo deben iniciar la sesión del usuario, sino también obtener acceso a un servicio web en nombre de ese usuario mediante OAuth. En este escenario se combina OpenID Connect para la autenticación de usuario, al tiempo que se obtiene a la vez un `authorization_code` que puede usarse para obtener `access_tokens` mediante el [flujo de código de autorización de OAuth](v1-protocols-oauth-code.md#use-the-authorization-code-to-request-an-access-token).

## <a name="get-access-tokens"></a>Obtención de tokens de acceso
Para obtener tokens de acceso, debe modificar la solicitud anterior de inicio de sesión:

```
// Line breaks for legibility only

GET https://login.microsoftonline.com/{tenant}/oauth2/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e        // Your registered Application ID
&response_type=id_token+code
&redirect_uri=http%3A%2F%2Flocalhost%3a12345          // Your registered Redirect Uri, url encoded
&response_mode=form_post                              // `form_post' or 'fragment'
&scope=openid
&resource=https%3A%2F%2Fservice.contoso.com%2F        // The identifier of the protected resource (web API) that your application needs access to
&state=12345                                          // Any value, provided by your app
&nonce=678910                                         // Any value, provided by your app
```

Con la inclusión de los ámbitos de permiso en la solicitud y el uso de `response_type=code+id_token`, el punto de conexión de `authorize` garantiza que el usuario ha dado su consentimiento a los permisos indicados en el parámetro de consulta `scope`. Asimismo, devuelve a la aplicación un código de autorización para intercambiarlo por un token de acceso.

### <a name="successful-response"></a>Respuesta correcta

Una respuesta correcta, enviada a la `redirect_uri` mediante `response_mode=form_post`, aspecto:

```
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNB...&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&state=12345
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| ID_token |El `id_token` que solicitó la aplicación. Puede usar el `id_token` para comprobar la identidad del usuario y comenzar una sesión con el usuario. |
| código |El authorization_code que solicitó la aplicación. La aplicación puede utilizar el código de autorización para solicitar un token de acceso para el recurso de destino. Los authorization_codes son de corta duración y normalmente expiran después de unos 10 minutos. |
| state |Si se incluye un parámetro de estado en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debería comprobar que los valores de estado de la solicitud y la respuesta son idénticos. |

### <a name="error-response"></a>Respuesta de error

Las respuestas de error también pueden enviarse al `redirect_uri` para que la aplicación pueda controlarlas adecuadamente:

```
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

error=access_denied&error_description=the+user+canceled+the+authentication
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| error |Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| error_description |Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error de autenticación. |

Para ver una descripción de los posibles códigos de error y la acción recomendada que tiene que realizar el cliente, consulte la sección [Códigos de error correspondientes a errores de puntos de conexión de autorización](#error-codes-for-authorization-endpoint-errors).

Una vez que ha obtenido un `code` de autorización y un `id_token`, puede iniciar la sesión del usuario y obtener [tokens de acceso](access-tokens.md) en su nombre. Para iniciar la sesión del usuario, debe validar `id_token` exactamente como se ha descrito anteriormente. Para obtener tokens de acceso, puede seguir los pasos que se describen en la sección "Uso del código de autorización para solicitar un token de acceso" de nuestra [documentación de flujo de código del protocolo OAuth](v1-protocols-oauth-code.md#use-the-authorization-code-to-request-an-access-token).

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre los [tokens de acceso](access-tokens.md).
* Más información sobre [`id_token` y las notificaciones](id-tokens.md).
