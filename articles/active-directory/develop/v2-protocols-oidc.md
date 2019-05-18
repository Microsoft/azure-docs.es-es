---
title: Plataforma de identidad de Microsoft y el protocolo OpenID Connect | Azure
description: Crear aplicaciones web mediante la implementación de plataforma de identidad de Microsoft del protocolo de autenticación OpenID Connect.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: a4875997-3aac-4e4c-b7fe-2b4b829151ce
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/12/2019
ms.author: ryanwi
ms.reviewer: hirsin
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 23a8eaaf095be1d59944791bd793047886dda40c
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544806"
---
# <a name="microsoft-identity-platform-and-openid-connect-protocol"></a>Plataforma de identidad de Microsoft y el protocolo OpenID Connect

OpenID Connect es un protocolo de autenticación basado en OAuth 2.0 que se puede usar para que un usuario inicie sesión de forma segura en una aplicación web. Cuando se usa la implementación de Microsoft identity platform del punto de conexión de OpenID Connect, puede agregar el inicio de sesión y acceso a API a las aplicaciones basadas en web. En este artículo se muestra cómo hacer esto de forma independiente del lenguaje y se describe cómo enviar y recibir mensajes HTTP sin usar ninguna biblioteca de código abierto de Microsoft.

> [!NOTE]
> El punto de conexión de plataforma de identidad de Microsoft no admite todas las características y escenarios de Azure Active Directory (Azure AD). Para determinar si debe utilizar el punto de conexión de plataforma de identidad de Microsoft, obtenga información sobre [limitaciones de la plataforma de identidad de Microsoft](active-directory-v2-limitations.md).

[OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html) amplía el protocolo de *autorización* de OAuth 2.0 para usarlo como un protocolo de *autenticación*, lo que le permite realizar inicios de sesión únicos mediante OAuth. OpenID Connect presenta el concepto de un *token de identificador*, que es un token de seguridad que permite al cliente comprobar la identidad del usuario. El token de identificador también obtiene información de perfil básica sobre el usuario. Como OpenID Connect amplía OAuth 2.0, las aplicaciones pueden adquirir de forma segura *tokens de acceso*, que se pueden usar para obtener acceso a los recursos protegidos mediante un [servidor de autorización](active-directory-v2-protocols.md#the-basics). El punto de conexión de plataforma de identidad de Microsoft también permite que las aplicaciones de terceros que están registradas con Azure AD para emitir tokens de acceso para los recursos protegidos como las API Web. Para obtener más información acerca de cómo configurar una aplicación para emitir tokens de acceso, consulte [cómo registrar una aplicación con el punto de conexión de plataforma de identidad de Microsoft](quickstart-register-app.md). Se recomienda usar OpenID Connect si compila una [aplicación web](v2-app-types.md#web-apps) hospedada en un servidor y a la que se obtiene acceso a través de un explorador.

## <a name="protocol-diagram-sign-in"></a>Diagrama de protocolo: Inicio de sesión

El diagrama siguiente muestra los pasos de un flujo de inicio de sesión más básico. En este artículo se describe detalladamente cada paso.

![Protocolo OpenID Connect: Inicio de sesión](./media/v2-protocols-oidc/convergence-scenarios-webapp.svg)

## <a name="fetch-the-openid-connect-metadata-document"></a>Captura del documento de metadatos de OpenID Connect

OpenID Connect describe un documento de metadatos que contiene la mayor parte de la información necesaria para que una aplicación para iniciar sesión. Esta información incluye las direcciones URL que se usan y la ubicación de las claves de firma pública del servicio. Para el punto de conexión de plataforma de identidad de Microsoft, trata el documento de metadatos OpenID Connect, que debe usar:

```
https://login.microsoftonline.com/{tenant}/v2.0/.well-known/openid-configuration
```
> [!TIP]
> Pruébela. Haga clic en [https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration](https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration) para ver la configuración de inquilinos `common`.

`{tenant}` puede tomar uno de estos cuatro valores:

| Valor | DESCRIPCIÓN |
| --- | --- |
| `common` |Los usuarios con una cuenta Microsoft personal y una cuenta profesional o educativa de Azure AD pueden iniciar sesión la aplicación. |
| `organizations` |Solo los usuarios con cuentas profesionales o educativas de Azure AD pueden iniciar sesión en la aplicación. |
| `consumers` |Solo los usuarios con una cuenta de Microsoft personal pueden iniciar sesión en la aplicación. |
| `8eaef023-2b34-4da1-9baa-8bc8c9d6a490` o `contoso.onmicrosoft.com` | Solo los usuarios con una cuenta profesional o educativa de un inquilino específico de Azure AD puede iniciar sesión en la aplicación. Puede usarse el nombre de dominio descriptivo del inquilino de Azure AD o bien el identificador de GUID del inquilino. También puede usar el inquilino consumidor `9188040d-6c67-4c5b-b112-36a304b66dad`, en lugar de la `consumers` inquilino.  |

Los metadatos son un documento de notación de objetos JavaScript (JSON) simple. Consulte el fragmento de código siguiente para ver un ejemplo. El contenido del fragmento de código se describe detalladamente en la [especificación de OpenID Connect](https://openid.net/specs/openid-connect-discovery-1_0.html#rfc.section.4.2).

```
{
  "authorization_endpoint": "https:\/\/login.microsoftonline.com\/{tenant}\/oauth2\/v2.0\/authorize",
  "token_endpoint": "https:\/\/login.microsoftonline.com\/{tenant}\/oauth2\/v2.0\/token",
  "token_endpoint_auth_methods_supported": [
    "client_secret_post",
    "private_key_jwt"
  ],
  "jwks_uri": "https:\/\/login.microsoftonline.com\/{tenant}\/discovery\/v2.0\/keys",

  ...

}
```

Si la aplicación tiene las claves de firma personalizadas como resultado de utilizar el [asignación de notificaciones](active-directory-claims-mapping.md) característica, debe anexar un `appid` que contiene el identificador de aplicación con el fin de obtener el parámetro de consulta un `jwks_uri` que apunta a la aplicación de la clave de firma información. Por ejemplo: `https://login.microsoftonline.com/{tenant}/.well-known/v2.0/openid-configuration?appid=6731de76-14a6-49ae-97bc-6eba6914391e` contiene un `jwks_uri` de `https://login.microsoftonline.com/{tenant}/discovery/v2.0/keys?appid=6731de76-14a6-49ae-97bc-6eba6914391e`.

Normalmente, este documento de metadatos se usa para configurar una biblioteca o SDK de OpenID Connect; la biblioteca usa los metadatos para realizar su trabajo. Sin embargo, si no usa una biblioteca precompilada OpenID Connect, puede seguir los pasos descritos en el resto de este artículo para iniciar sesión en una aplicación web utilizando el punto de conexión de plataforma de identidad de Microsoft.

## <a name="send-the-sign-in-request"></a>Envío de la solicitud de inicio de sesión

Cuando su aplicación web deba autenticar al usuario, puede dirigirlo al punto de conexión `/authorize` . Esta solicitud es similar al primer segmento del [flujo de código de autorización de OAuth 2.0](v2-oauth2-auth-code-flow.md), con estas importantes diferencias:

* La solicitud debe incluir el ámbito `openid` en el parámetro `scope`.
* El parámetro `response_type` debe incluir `id_token`.
* La solicitud debe incluir el parámetro `nonce` .

> [!IMPORTANT]
> Con el fin de solicitar un token de identificador correctamente desde el punto de conexión /authorization, el registro de aplicación en el [portal de registro](https://portal.azure.com) debe tener la concesión implícita de los id_tokens habilitada en la ficha autenticación (que establece el `oauth2AllowIdTokenImplicitFlow`marca en el [manifiesto de aplicación](reference-app-manifest.md) a `true`). Si aún no está habilitado, un `unsupported_response` , se devolverá el error: "El valor proporcionado para el parámetro de entrada 'response_type' no se permite para este cliente. Expected value is 'code'" ("No se permite el valor proporcionado para el parámetro de entrada "response_type" para este cliente. El valor esperado es "code"")

Por ejemplo:

```
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=id_token
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=form_post
&scope=openid
&state=12345
&nonce=678910
```

> [!TIP]
> Haga clic en el vínculo siguiente para ejecutar esta solicitud. Una vez que inicie sesión, el explorador se redirigirá a `https://localhost/myapp/`, con un token de identificador en la barra de direcciones. Tenga en cuenta que esta solicitud usa `response_mode=fragment` (solo con fines de demostración). Se recomienda que use `response_mode=form_post`.
> <a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=id_token&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&scope=openid&response_mode=fragment&state=12345&nonce=678910" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>

| Parámetro | Condición | DESCRIPCIÓN |
| --- | --- | --- |
| `tenant` | Obligatorio | Puede usar el valor `{tenant}` en la ruta de acceso de la solicitud para controlar quién puede iniciar sesión en la aplicación. Los valores permitidos son `common`, `organizations`, `consumers` y los identificadores de inquilinos. Para más información, consulte los [conceptos básicos sobre el protocolo](active-directory-v2-protocols.md#endpoints). |
| `client_id` | Obligatorio | El **Id. de aplicación (cliente)** que la [Azure portal: registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) experiencia asignado a la aplicación. |
| `response_type` | Obligatorio | Debe incluir `id_token` para el inicio de sesión en OpenID Connect. También puede incluir otros valores `response_type`, como `code`. |
| `redirect_uri` | Recomendada | El URI de redireccionamiento de la aplicación, adonde la aplicación puede enviar y recibir las respuestas de autenticación. Debe coincidir exactamente con uno de los URI de redireccionamiento que registró en el portal, con la excepción de que debe estar codificado como URL. Si no se presente, el punto de conexión realizará una selección de redirect_uri registrados una al azar a enviar al usuario a. |
| `scope` | Obligatorio | Una lista de ámbitos separada por espacios. Asegúrese de incluir el ámbito `openid`para OpenID Connect, lo que se traduce en el permiso de inicio de sesión en la interfaz de usuario de consentimiento. También puede incluir otros ámbitos en esta solicitud para solicitar el consentimiento. |
| `nonce` | Obligatorio | Un valor incluido en la solicitud, generado por la aplicación, que se incluirá en el valor id_token resultante como una notificación. La aplicación puede comprobar este valor para mitigar los ataques de reproducción de token. Habitualmente, el valor es una cadena única aleatoria que se puede usar para identificar el origen de la solicitud. |
| `response_mode` | Recomendada | Especifica el método que se debe usar para enviar el código de autorización resultante de nuevo a la aplicación. Puede ser `form_post` o `fragment`. En el caso de las aplicaciones web, se recomienda usar `response_mode=form_post` para asegurar la transferencia más segura de tokens a la aplicación. |
| `state` | Recomendada | Un valor incluido en la solicitud que también se devolverá en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. Se usa normalmente un valor único generado de forma aleatoria para [evitar los ataques de falsificación de solicitudes entre sitios](https://tools.ietf.org/html/rfc6749#section-10.12). El estado también se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, como la página o la vista en la que estaba el usuario. |
| `prompt` | Opcional | Indica el tipo de interacción necesaria con el usuario. Los únicos valores válidos en este momento son `login`, `none` y `consent`. La notificación `prompt=login` obliga al usuario a escribir sus credenciales en esa solicitud, lo que niega el inicio de sesión único. La notificación `prompt=none` es lo contrario. Esta notificación se asegura de que no se presenta al usuario ninguna solicitud interactiva en. Si la solicitud no se puede completar en modo silencioso a través de inicio de sesión único, el punto de conexión de plataforma de identidad de Microsoft devuelve un error. La notificación `prompt=consent` desencadena el cuadro de diálogo de consentimiento de OAuth después de que el usuario inicia sesión. El cuadro de diálogo le pide al usuario que conceda permisos a la aplicación. |
| `login_hint` | Opcional | Puede usar este parámetro para rellenar previamente el campo de nombre de usuario y dirección de correo electrónico de la página de inicio de sesión del usuario, si sabe el nombre de usuario con anticipación. A menudo, las aplicaciones usan este parámetro durante la reautenticación, una vez que ya se extrajo el nombre de usuario de un inicio de sesión anterior mediante la notificación `preferred_username`. |
| `domain_hint` | Opcional | El dominio Kerberos del usuario en un directorio federado.  Esto omite el proceso de detección basada en correo electrónico que pasa el usuario en la página de inicio de sesión, para una experiencia de usuario será ligeramente más sencilla. Para los inquilinos que están federados a través de un directorio local como AD FS, esto suele ocurrir en un inicio de sesión sin problemas debido a la sesión de inicio de sesión existente. |

En este punto, se le pide al usuario que escriba sus credenciales y que complete la autenticación. El punto de conexión de plataforma de identidad de Microsoft comprueba que el usuario ha dado su consentimiento a los permisos indicados en el `scope` parámetro de consulta. Si el usuario no ha dado su consentimiento a cualquiera de estos permisos, el punto de conexión de plataforma de identidad de Microsoft le pide al usuario su consentimiento a los permisos necesarios. Puede leer más sobre [permisos, consentimiento y aplicaciones de varios inquilinos](v2-permissions-and-consent.md).

Después de que el usuario se autentica y da su consentimiento, el punto de conexión de plataforma de identidad de Microsoft devuelve una respuesta a la aplicación en el URI de redireccionamiento mediante el método especificado en el `response_mode` parámetro.

### <a name="successful-response"></a>Respuesta correcta

Una respuesta correcta cuando usa `response_mode=form_post` tiene el aspecto siguiente:

```
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNB...&state=12345
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| `id_token` | El token de identificador que la aplicación solicitó. Puede usar el parámetro `id_token` para comprobar la identidad del usuario y comenzar una sesión con el usuario. Para más información sobre los tokens de identificador y su contenido, consulte la [`id_tokens` referencia](id-tokens.md). |
| `state` | Si un parámetro `state` está incluido en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debería comprobar que los valores de estado de la solicitud y la respuesta son idénticos. |

### <a name="error-response"></a>Respuesta de error

Las respuestas de error también se pueden enviar al URI de redireccionamiento para que la aplicación pueda controlarlas. Una respuesta de error tiene el aspecto siguiente:

```
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

error=access_denied&error_description=the+user+canceled+the+authentication
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| `error` | Una cadena de código de error que puede usar para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que puede ayudarlo a identificar la causa raíz de un error de autenticación. |

### <a name="error-codes-for-authorization-endpoint-errors"></a>Códigos de error correspondientes a errores de puntos de conexión de autorización

En la tabla siguiente se describen los códigos de error que puede devolver el parámetro `error` de la respuesta de error:

| Código de error | DESCRIPCIÓN | Acción del cliente |
| --- | --- | --- |
| `invalid_request` | Error de protocolo, como un parámetro obligatorio que falta. |Corrija el error y vuelva a enviar la solicitud. Se trata de un error de desarrollo que habitualmente se detecta durante las pruebas iniciales. |
| `unauthorized_client` | La aplicación cliente no puede solicitar un código de autorización. |Esto suele ocurrir cuando la aplicación cliente no está registrada en Azure AD o no se agrega al inquilino de Azure AD del usuario. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |
| `access_denied` | El propietario del recurso denegó el consentimiento. |La aplicación cliente puede notificar al usuario que no puede continuar a menos que el usuario da su consentimiento. |
| `unsupported_response_type` |El servidor de autorización no admite el tipo de respuesta de la solicitud. |Corrija el error y vuelva a enviar la solicitud. Se trata de un error de desarrollo que habitualmente se detecta durante las pruebas iniciales. |
| `server_error` | El servidor ha detectado un error inesperado. |Vuelva a intentarlo. Estos errores pueden deberse a condiciones temporales. La aplicación cliente podría explicar al usuario que su respuesta se retrasó debido a un error temporal. |
| `temporarily_unavailable` | De manera temporal, el servidor está demasiado ocupado para atender la solicitud. |Vuelva a intentarlo. La aplicación cliente podría explicar al usuario que su respuesta se retrasó debido a una condición temporal. |
| `invalid_resource` | El recurso de destino no es válido porque no existe, Azure AD no lo encuentra, o bien no está configurado correctamente. |Esto indica que el recurso, si existe, no se ha configurado en el inquilino. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |

## <a name="validate-the-id-token"></a>Validar el token de identificador

Recibir un solo id_token no es suficiente para autenticar al usuario; debe validar la firma del id_token y comprobar las notificaciones en el token según los requisitos de la aplicación. El punto de conexión de plataforma de identidad de Microsoft usa [Tokens Web JSON (Jwt)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) y criptografía de clave pública para firmar los tokens y comprobar si es válidos.

Puede elegir validar la `id_token` en cliente de código, pero una práctica común consiste en enviar la `id_token` a un servidor back-end y saque allí la validación. Una vez que haya validado la firma del id_token, hay algunas notificaciones que se le pide a comprobar. Para más información, consulte la [Referencia de `id_token`](id-tokens.md), incluidas la sección [Validación de los tokens](id-tokens.md#validating-an-id_token) y [Sustitución de claves de firma de Azure Active Directory](active-directory-signing-key-rollover.md). Hay al menos una disponible para la mayoría de los lenguajes y las plataformas.

Se recomienda que valide notificaciones adicionales según su escenario. Algunas validaciones comunes incluyen:

* Asegurarse de que la organización/el usuario se ha registrado en la aplicación.
* Asegurarse de que el usuario tiene la autorización/los privilegios adecuados
* Asegurarse de que se haya producido un determinado nivel de autenticación, como la autenticación multifactor.

Una vez haya validado el id_token, puede iniciar una sesión con el usuario y usar las notificaciones en id_token para obtener información sobre el usuario en la aplicación. Esta información puede usarse para visualización, registros, autorizaciones, etc.

## <a name="send-a-sign-out-request"></a>Envío de una solicitud de cierre de sesión

Si desea cerrar la sesión del usuario en la aplicación, no basta con borrar las cookies de la aplicación o finalizar la sesión del usuario. También debe redirigir al usuario para el punto de conexión de la plataforma de identidad de Microsoft para cerrar la sesión. Si no lo hace, el usuario vuelve a autenticar a su aplicación sin escribir sus credenciales de nuevo, ya que tienen una única inicio de sesión de sesión válida con el punto de conexión de plataforma de identidad de Microsoft.

Puede redirigir al usuario al objeto `end_session_endpoint` que aparece en el documento de metadatos OpenID Connect:

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/logout?
post_logout_redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
```

| Parámetro | Condición | DESCRIPCIÓN |
| ----------------------- | ------------------------------- | ------------ |
| `post_logout_redirect_uri` | Recomendada | La dirección URL al que se redirige al usuario tras cerrar sesión correctamente. Si el parámetro no se incluye, se muestra al usuario un mensaje genérico que se genera el punto de conexión de plataforma de identidad de Microsoft. Esta dirección URL debe coincidir con uno de los URI de redirección registrados para su aplicación en el portal de registro de aplicaciones. |

## <a name="single-sign-out"></a>Cierre de sesión único

Cuando redirige al usuario la `end_session_endpoint`, el punto de conexión de plataforma de identidad de Microsoft borra la sesión del usuario desde el explorador. Sin embargo, el usuario puede permanecer con la sesión iniciada en otras aplicaciones que usan cuentas de Microsoft para la autenticación. Para permitir que esas aplicaciones cerrar la sesión del usuario al mismo tiempo, Microsoft, punto de conexión de plataforma de identidad envía una solicitud HTTP GET a registrado `LogoutUrl` de todas las aplicaciones que el usuario actualmente ha iniciado sesión en. Las aplicaciones deben responder a esta solicitud mediante la eliminación de la sesión que identifica al usuario y la devolución de una respuesta `200`. Si quiere admitir el inicio de sesión único en la aplicación, debe implementar este elemento `LogoutUrl` en el código de la aplicación. Para configurar `LogoutUrl`, hágalo en el portal de registro de aplicaciones.

## <a name="protocol-diagram-access-token-acquisition"></a>Diagrama de protocolo: Obtención de tokens de acceso

Muchas aplicaciones web no solo deben iniciar la sesión del usuario, sino también obtener acceso a un servicio web en nombre del usuario mediante OAuth. En este escenario se combina OpenID Connect para la autenticación de usuario, al tiempo que se obtiene, a la vez, un código de autorización que puede usar para obtener tokens de acceso si usa el flujo de código de autorización de OAuth.

El flujo completo de adquisición de tokens e inicio de sesión de OpenID Connect es similar al diagrama siguiente. En las secciones siguientes del artículo se describe detalladamente cada paso.

![Protocolo OpenID Connect: Obtención de tokens](./media/v2-protocols-oidc/convergence-scenarios-webapp-webapi.svg)

## <a name="get-access-tokens"></a>Obtención de tokens de acceso
Para obtener tokens de acceso, modifique la solicitud de inicio de sesión:

```
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e        // Your registered Application ID
&response_type=id_token%20code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F       // Your registered redirect URI, URL encoded
&response_mode=form_post                              // 'form_post' or 'fragment'
&scope=openid%20                                      // Include both 'openid' and scopes that your app needs  
offline_access%20                                         
https%3A%2F%2Fgraph.microsoft.com%2Fuser.read
&state=12345                                          // Any value, provided by your app
&nonce=678910                                         // Any value, provided by your app
```

> [!TIP]
> Haga clic en el vínculo siguiente para ejecutar esta solicitud. Una vez que inicie sesión, el explorador se redirigirá a `https://localhost/myapp/`, con un token de identificador y un código en la barra de direcciones. Tenga en cuenta que esta solicitud usa `response_mode=fragment` solo con fines de demostración. Se recomienda que use `response_mode=form_post`.
> <a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=id_token%20code&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_mode=fragment&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fuser.read&state=12345&nonce=678910" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>

Mediante la inclusión de los ámbitos de permiso en la solicitud y mediante el uso de `response_type=id_token code`, el punto de conexión de plataforma de identidad de Microsoft garantiza que el usuario ha dado su consentimiento a los permisos indicados en el `scope` parámetro de consulta. Devuelve un código de autorización a la aplicación para intercambiarla por un token de acceso.

### <a name="successful-response"></a>Respuesta correcta

Una respuesta correcta al usar `response_mode=form_post` tiene el siguiente aspecto:

```
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

id_token=eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNB...&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...&state=12345
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| `id_token` | El token de identificador que la aplicación solicitó. Puede usar el token de identificador para comprobar la identidad del usuario y comenzar una sesión con el usuario. Encontrará más detalles sobre los tokens de identificador y su contenido en la [`id_tokens` referencia](id-tokens.md). |
| `code` | El código de autorización que la aplicación solicitó. La aplicación puede utilizar el código de autorización para solicitar un token de acceso para el recurso de destino. Un código de autorización es corta duración. Habitualmente, un código de autorización expira en alrededor de 10 minutos. |
| `state` | Si se incluye un parámetro de estado en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debería comprobar que los valores de estado de la solicitud y la respuesta son idénticos. |

### <a name="error-response"></a>Respuesta de error

Las respuestas de error también se pueden enviar al URI de redireccionamiento para que la aplicación pueda controlarlas correctamente. Una respuesta de error tiene el aspecto siguiente:

```
POST /myapp/ HTTP/1.1
Host: localhost
Content-Type: application/x-www-form-urlencoded

error=access_denied&error_description=the+user+canceled+the+authentication
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| `error` | Una cadena de código de error que puede usar para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que puede ayudarlo a identificar la causa raíz de un error de autenticación. |

Para una descripción de los posibles códigos de error y las respuestas de cliente recomendadas, consulte [Códigos de error correspondientes a errores de puntos de conexión de autorización](#error-codes-for-authorization-endpoint-errors).

Cuando tiene un código de autorización y un token de identificador, puede iniciar la sesión del usuario y obtener tokens de acceso en su nombre. Para iniciar la sesión del usuario, debe validar el token de identificador [exactamente como se describió](id-tokens.md#validating-an-id_token). Para obtener tokens de acceso, siga los pasos descritos en la [documentación del flujo de código de OAuth](v2-oauth2-auth-code-flow.md#request-an-access-token).
