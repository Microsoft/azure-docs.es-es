---
title: Plataforma de identidad de Microsoft y flujo de código de autorización de OAuth | Azure
description: Creación de aplicaciones web mediante la implementación de plataforma de identidad de Microsoft del protocolo de autenticación OAuth 2.0.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: ae1d7d86-7098-468c-aa32-20df0a10ee3d
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
ms.openlocfilehash: 0d3ab6f53fdb11b0b8d643868d0692667c8672f9
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545178"
---
# <a name="microsoft-identity-platform-and-oauth-20-authorization-code-flow"></a>Plataforma de identidad de Microsoft y flujo de código de autorización de OAuth 2.0

[!INCLUDE [active-directory-develop-applies-v2](../../../includes/active-directory-develop-applies-v2.md)]

La concesión de un código de autorización de OAuth 2.0 se puede usar en aplicaciones que se instalan en un dispositivo para obtener acceso a recursos protegidos, como las API web. Mediante la implementación de plataforma de identidad de Microsoft de OAuth 2.0, puede agregar iniciar sesión y tener acceso API a las aplicaciones de escritorio y móviles. En esta guía, que es independiente del lenguaje, se describe cómo enviar y recibir mensajes HTTP sin usar ninguna de las [bibliotecas de autenticación de código abierto de Azure](active-directory-authentication-libraries.md).

> [!NOTE]
> No todas las características y escenarios de Azure Active Directory se admiten el punto de conexión de plataforma de identidad de Microsoft. Para determinar si debe usar el punto de conexión de plataforma de identidad de Microsoft, obtenga información sobre [limitaciones de la plataforma de identidad de Microsoft](active-directory-v2-limitations.md).

El flujo de código de autorización de OAuth 2.0 se describe en la [sección 4.1 de la especificación de OAuth 2.0](https://tools.ietf.org/html/rfc6749). Se utiliza para realizar la autenticación y autorización en la mayoría de los tipos de aplicación, incluidos [aplicaciones web](v2-app-types.md#web-apps) y [aplicaciones instaladas de forma nativa](v2-app-types.md#mobile-and-native-apps). El flujo permite que las aplicaciones adquieran de forma segura access_tokens que se puede usar para tener acceso a recursos protegidos por el punto de conexión de plataforma de identidad de Microsoft.

## <a name="protocol-diagram"></a>Diagrama de protocolo

En un nivel alto, el flujo de autenticación completo para una aplicación nativa o móvil es algo parecido a esto:

![Flujo de código de autenticación de OAuth](./media/v2-oauth2-auth-code-flow/convergence-scenarios-native.svg)

## <a name="request-an-authorization-code"></a>Solicitud de un código de autorización

El flujo del código de autorización comienza con el cliente dirigiendo al usuario al punto de conexión `/authorize` . En esta solicitud, el cliente indica los permisos que necesita adquirir del usuario:

```
// Line breaks for legibility only

https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fuser.read
&state=12345
```

> [!TIP]
> Para ejecutar esta solicitud, haga clic en el vínculo siguiente. Después de iniciar sesión, el explorador se redirigirá a `https://localhost/myapp/` con un elemento `code` en la barra de direcciones.
> <a href="https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&response_type=code&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F&response_mode=query&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2Fuser.read&state=12345" target="_blank">https://login.microsoftonline.com/common/oauth2/v2.0/authorize...</a>

| Parámetro    | Obligatorio/opcional | DESCRIPCIÓN |
|--------------|-------------|--------------|
| `tenant`    | requerido    | El valor `{tenant}` de la ruta de acceso de la solicitud se puede usar para controlar quién puede iniciar sesión en la aplicación. Los valores permitidos son `common`, `organizations`, `consumers` y los identificadores de inquilinos. Para obtener más información, consulte los [conceptos básicos sobre el protocolo](active-directory-v2-protocols.md#endpoints).  |
| `client_id`   | requerido    | El **Id. de aplicación (cliente)** que la [Azure portal: registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) experiencia asignado a la aplicación.  |
| `response_type` | requerido    | Debe incluir `code` para el flujo de código de autorización.       |
| `redirect_uri`  | requerido | El redirect_uri de su aplicación, a donde su aplicación puede enviar y recibir las respuestas de autenticación. Debe coincidir exactamente con uno de los redirect_uris que registró en el portal, con la excepción de que debe estar codificado como URL. En el caso de las aplicaciones nativas y móviles, es preciso usar el valor predeterminado, `https://login.microsoftonline.com/common/oauth2/nativeclient`.   |
| `scope`  | requerido    | Una lista separada por espacios de [ámbitos](v2-permissions-and-consent.md) que desea que el usuario consienta. |
| `response_mode`   | recomendado | Especifica el método que debe usarse para enviar el token resultante de nuevo a la aplicación. Puede ser uno de los siguientes:<br/><br/>- `query`<br/>- `fragment`<br/>- `form_post`<br/><br/>`query` proporciona el código como un parámetro de cadena de consulta en el URI de redirección. Si se solicita un token de identificador con el flujo implícito, no puede usar `query` según lo especificado en el [especificación de OpenID](https://openid.net/specs/oauth-v2-multiple-response-types-1_0.html#Combinations). Si solicita solo el código, puede usar `query`, `fragment` o `form_post`. `form_post` ejecuta una prueba POST que contiene el código para el URI de redirección. Para más información, consulte [Protocolo OpenID Connect](https://docs.microsoft.com/azure/active-directory/develop/active-directory-protocols-openid-connect-code).  |
| `state`                 | recomendado | Un valor incluido en la solicitud que se devolverá también en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. Normalmente se usa un valor único generado de forma aleatoria para [evitar los ataques de falsificación de solicitudes entre sitios](https://tools.ietf.org/html/rfc6749#section-10.12). El valor también puede codificar información sobre el estado del usuario en la aplicación antes de que se produzca la solicitud de autenticación, por ejemplo, la página o vista en la que estaba. |
| `prompt`  | opcional    | Indica el tipo de interacción necesaria con el usuario. Los únicos valores válidos en este momento son `login`, `none` y `consent`.<br/><br/>- `prompt=login` obligará al usuario a escribir sus credenciales en esa solicitud, negando el inicio de sesión único.<br/>- `prompt=none` es la opuesta - garantizará que el usuario no se le presenta ninguna solicitud interactiva del tipo que sea. Si la solicitud no se puede completar en modo silencioso a través de inicio de sesión único, el punto de conexión de plataforma de identidad de Microsoft devolverá un `interaction_required` error.<br/>- `prompt=consent` desencadenará el cuadro de diálogo de consentimiento de OAuth después de que el usuario inicie sesión y solicitará a este que conceda permisos a la aplicación. |
| `login_hint`  | opcional    | Puede usarse para rellenar previamente el campo de nombre de usuario y dirección de correo electrónico de la página de inicio de sesión del usuario, si sabe su nombre de usuario con antelación. A menudo las aplicaciones usarán este parámetro durante la reautenticación, dado que ya han extraído el nombre de usuario de un inicio de sesión anterior mediante la notificación `preferred_username`.   |
| `domain_hint`  | opcional    | Puede ser `consumers` o `organizations`.<br/><br/>Si se incluye, omitirá el proceso de detección basada en correo electrónico que pasa el usuario en la página de inicio de sesión, dando lugar a una experiencia de usuario será ligeramente más sencilla. A menudo las aplicaciones usarán este parámetro durante la reautenticación, para lo que extraen `tid` de un inicio de sesión anterior. Si el valor de la notificación `tid` es `9188040d-6c67-4c5b-b112-36a304b66dad`, debe usar `domain_hint=consumers`. De lo contrario, use `domain_hint=organizations`.  |
| `code_challenge_method` | opcional    | Método utilizado para codificar `code_verifier` para el parámetro `code_challenge`. Puede presentar uno de los siguientes valores:<br/><br/>- `plain` <br/>- `S256`<br/><br/>Si se excluye, se supone que `code_challenge` es texto no cifrado si se incluye `code_challenge`. Plataforma de identidad de Microsoft es compatible con ambos `plain` y `S256`. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636). |
| `code_challenge`  | opcional | Se usa para proteger concesiones de código de autorización a través de la clave de prueba para intercambio de códigos (PKCE) desde un cliente nativo. Se requiere si se incluye `code_challenge_method`. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636). |

En este punto, se le pedirá al usuario que escriba sus credenciales y que complete la autenticación. El punto de conexión de plataforma de identidad de Microsoft también se asegurará de que el usuario ha dado su consentimiento a los permisos indicados en el `scope` parámetro de consulta. Si el usuario no ha dado su consentimiento a alguno de esos permisos, se le solicitará al usuario su consentimiento para los permisos necesarios. Aquí puede encontrar información detallada sobre los [permisos, el consentimiento y las aplicaciones multiempresa](v2-permissions-and-consent.md).

Una vez que el usuario se autentica y da su consentimiento, el punto de conexión de plataforma de identidad de Microsoft devolverá una respuesta a la aplicación en el indicado `redirect_uri`, mediante el método especificado en el `response_mode` parámetro.

#### <a name="successful-response"></a>Respuesta correcta

Una respuesta correcta al usar `response_mode=query` tiene el siguiente aspecto:

```
GET https://login.microsoftonline.com/common/oauth2/nativeclient?
code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq...
&state=12345
```

| . | DESCRIPCIÓN  |
|-----------|--------------|
| `code` | El authorization_code que solicitó la aplicación. La aplicación puede utilizar el código de autorización para solicitar un token de acceso para el recurso de destino. Los Authorization_codes son de corta duración, normalmente expiran después de unos 10 minutos. |
| `state` | Si se incluye un parámetro de estado en la solicitud, debería aparecer el mismo valor en la respuesta. La aplicación debería comprobar que los valores de estado de la solicitud y la respuesta son idénticos. |

#### <a name="error-response"></a>Respuesta de error

Las respuestas de error también pueden enviarse al `redirect_uri` para que la aplicación pueda controlarlas adecuadamente:

```
GET https://login.microsoftonline.com/common/oauth2/nativeclient?
error=access_denied
&error_description=the+user+canceled+the+authentication
```

| Parámetro | DESCRIPCIÓN  |
|----------|------------------|
| `error`  | Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error de autenticación. |

#### <a name="error-codes-for-authorization-endpoint-errors"></a>Códigos de error correspondientes a errores de puntos de conexión de autorización

En la tabla siguiente se describen los distintos códigos de error que pueden devolverse en el parámetro `error` de la respuesta de error.

| Código de error  | DESCRIPCIÓN    | Acción del cliente   |
|-------------|----------------|-----------------|
| `invalid_request` | Error de protocolo, como un parámetro obligatorio que falta. | Corrija el error y vuelva a enviar la solicitud. Se trata de un error de desarrollo que suele detectarse durante las pruebas iniciales. |
| `unauthorized_client` | La aplicación cliente no se puede solicitar un código de autorización. | Este error suele producirse cuando la aplicación cliente no está registrada en Azure AD o no se agrega al inquilino de Azure AD del usuario. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |
| `access_denied`  | El propietario del recurso ha denegado el consentimiento.  | La aplicación cliente puede notificar al usuario que no puede continuar a menos que el usuario da su consentimiento. |
| `unsupported_response_type` | El servidor de autorización no admite el tipo de respuesta de la solicitud. | Corrija el error y vuelva a enviar la solicitud. Se trata de un error de desarrollo que suele detectarse durante las pruebas iniciales.  |
| `server_error`  | El servidor ha detectado un error inesperado.| Vuelva a intentarlo. Estos errores pueden deberse a condiciones temporales. La aplicación cliente podría explicar al usuario que su respuesta se retrasó debido a un error temporal. |
| `temporarily_unavailable`   | De manera temporal, el servidor está demasiado ocupado para atender la solicitud. | Vuelva a intentarlo. La aplicación cliente podría explicar al usuario que su respuesta se retrasó debido a una condición temporal. |
| `invalid_resource`  | El recurso de destino no es válido porque no existe, Azure AD no lo encuentra o está configurado incorrectamente. | Este error indica que el recurso, en caso de que exista, no se ha configurado en el inquilino. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |
| `login_required` | Se han encontrado demasiados usuarios o no se ha encontrado ninguno | El cliente solicitó la autenticación en modo silencioso (`prompt=none`), pero no se pudo encontrar un único usuario. Esto puede significar que hay varios usuarios activos en la sesión o que no hay ninguno. Esto tiene en cuenta que el inquilino elegido (por ejemplo, si hay dos cuentas de Azure AD Directory y una cuenta de Microsoft, y `consumers` elegido, funcionará la autenticación en modo silencioso). |
| `interaction_required` | La solicitud requiere la interacción del usuario. | Es necesario realizar un paso de autenticación o consentimiento más. Vuelva a intentar realizar la solicitud sin `prompt=none`. |

## <a name="request-an-access-token"></a>Solicitud de un token de acceso

Ahora que ha adquirido un código de autorización y el usuario le ha concedido permiso, puede canjear el `code` por un `access_token` al recurso deseado. Puede hacer esto mediante el envío de una solicitud `POST` al punto de conexión `/token`:

```
// Line breaks for legibility only

POST /{tenant}/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fuser.read
&code=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq3n8b2JRLk4OxVXr...
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&grant_type=authorization_code
&client_secret=JqQX2PNo9bpM0uEihUPzyrh    // NOTE: Only required for web apps
```

> [!TIP]
> Pruebe a ejecutar esta solicitud en Postman (No olvide reemplazar `code`) [![Ejecución en Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)

| Parámetro  | Obligatorio/opcional | DESCRIPCIÓN     |
|------------|-------------------|----------------|
| `tenant`   | requerido   | El valor `{tenant}` de la ruta de acceso de la solicitud se puede usar para controlar quién puede iniciar sesión en la aplicación. Los valores permitidos son `common`, `organizations`, `consumers` y los identificadores de inquilinos. Para obtener más información, consulte los [conceptos básicos sobre el protocolo](active-directory-v2-protocols.md#endpoints).  |
| `client_id` | requerido  | IDENTIFICADOR de la aplicación (cliente) que el [Azure portal: registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) página asignada a la aplicación. |
| `grant_type` | requerido   | Debe ser `authorization_code` para el flujo de código de autorización.   |
| `scope`      | requerido   | Una lista de ámbitos separada por espacios. Los ámbitos solicitados en esta fase deben ser un subconjunto de los ámbitos solicitados en el primer segmento o un equivalente de este. Si los ámbitos especificados en esta solicitud abarcan varios servidores de recursos, el punto de conexión de plataforma de identidad de Microsoft devolverá un token para el recurso especificado en el primer ámbito. Para obtener una explicación más detallada de los ámbitos, consulte [permisos, consentimiento y ámbitos](v2-permissions-and-consent.md). |
| `code`          | requerido  | El authorization_code que adquirió en el primer segmento del flujo. |
| `redirect_uri`  | requerido  | El mismo valor redirect_uri usado para adquirir el código de autorización. |
| `client_secret` | necesario para las aplicaciones web | El secreto de la aplicación que creó en el portal de registro de aplicaciones para su aplicación. No debe usar el secreto de aplicación en una aplicación nativa, porque los client_secrets no se puede almacenar de forma confiable en los dispositivos. Es obligatorio para las aplicaciones web y API web, que tienen la capacidad de almacenar el client_secret de forma segura en el servidor.  El secreto de cliente debe codificarse como dirección URL antes de enviarse.  |
| `code_verifier` | opcional  | El mismo valor de code_verifier que usó para obtener el valor de authorization_code. Se requiere si PKCE se utilizó en la solicitud de concesión de código de autorización. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636). |

### <a name="successful-response"></a>Respuesta correcta

Una respuesta de token correcta tendrá un aspecto similar al siguiente:

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "token_type": "Bearer",
    "expires_in": 3599,
    "scope": "https%3A%2F%2Fgraph.microsoft.com%2Fuser.read",
    "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD...",
}
```

| Parámetro     | DESCRIPCIÓN   |
|---------------|------------------------------|
| `access_token`  | El token de acceso solicitado. La aplicación puede utilizar este token para autenticarse en el recursos protegido, como una API web.  |
| `token_type`    | Indica el valor de tipo de token. El único tipo que admite Azure AD es el portador |
| `expires_in`    | Durante cuánto tiempo es válido el token de acceso (en segundos). |
| `scope`         | Los ámbitos para los que el access_token es válido. |
| `refresh_token` | Un token de actualización de OAuth 2.0. La aplicación puede utilizar este token para adquirir tokens de acceso adicionales una vez que expire el token de acceso actual. Los refresh_tokens son de larga duración y pueden usarse para conservar el acceso a los recursos durante largos períodos de tiempo. Para más información acerca de la actualización de un token de acceso, consulte la [siguiente sección](#refresh-the-access-token). <br> **Nota:** Solo se proporciona si se solicitó el ámbito `offline_access`. |
| `id_token`      | Un JSON Web Token (JWT). La aplicación puede decodificar los segmentos de este token para solicitar información acerca del usuario que ha iniciado sesión. La aplicación puede almacenar en caché los valores y mostrarlos, pero no debe confiar en ellos para cualquier autorización o límite de seguridad. Para más información sobre los parámetros id_tokens, consulte [`id_token reference`](id-tokens.md). <br> **Nota:** Solo se proporciona si se solicitó el ámbito `openid`. |

### <a name="error-response"></a>Respuesta de error

Las respuestas de error tendrán un aspecto similar al siguiente:

```json
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/mail.read is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| Parámetro         | DESCRIPCIÓN    |
|-------------------|----------------|
| `error`       | Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error de autenticación. |
| `error_codes` | Una lista de los códigos de error específicos de STS que pueden ayudar en los diagnósticos.  |
| `timestamp`   | La hora a la que se produjo el error. |
| `trace_id`    | Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos. |
| `correlation_id` | Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos entre componentes. |

### <a name="error-codes-for-token-endpoint-errors"></a>Códigos de error correspondientes a errores de puntos de conexión de token

| Código de error         | DESCRIPCIÓN        | Acción del cliente    |
|--------------------|--------------------|------------------|
| `invalid_request`  | Error de protocolo, como un parámetro obligatorio que falta. | Corrija el error y vuelva a enviar la solicitud.   |
| `invalid_grant`    | El código de autorización o el comprobador de código PKCE no son válidos o han expirado. | Intente una nueva solicitud para el punto de conexión `/authorize` y compruebe que el parámetro code_verifier es correcto.  |
| `unauthorized_client` | El cliente autenticado no está autorizado para usar este tipo de concesión de autorización. | Esto suele ocurrir cuando la aplicación cliente no está registrada en Azure AD o no se agrega al inquilino de Azure AD del usuario. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |
| `invalid_client` | Se produjo un error de autenticación de cliente.  | Las credenciales del cliente no son válidas. Para corregirlo, el administrador de la aplicación actualiza las credenciales.   |
| `unsupported_grant_type` | El servidor de autorización no admite el tipo de concesión de autorización. | Cambie el tipo de concesión de la solicitud. Este tipo de error solo debe producirse durante el desarrollo y detectarse en las pruebas iniciales. |
| `invalid_resource` | El recurso de destino no es válido porque no existe, Azure AD no lo encuentra o está configurado incorrectamente. | Este error indica que el recurso, en caso de que exista, no se ha configurado en el inquilino. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD.  |
| `interaction_required` | La solicitud requiere la interacción del usuario. Por ejemplo, hay que realizar un paso de autenticación más. | Vuelva a tratar de realizar la solicitud con el mismo recurso.  |
| `temporarily_unavailable` | De manera temporal, el servidor está demasiado ocupado para atender la solicitud. | Vuelva a intentarlo. La aplicación cliente podría explicar al usuario que su respuesta se retrasó debido a una condición temporal. |

## <a name="use-the-access-token"></a>Uso del token de acceso

Ahora que ha adquirido correctamente un `access_token`, puede usar el token en solicitudes a las API web mediante su inclusión en el encabezado `Authorization`:

> [!TIP]
> Ejecute esta solicitud en Postman (Reemplace primero el encabezado `Authorization`) [![Ejecución en Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)

```
GET /v1.0/me/messages
Host: https://graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

## <a name="refresh-the-access-token"></a>Actualización del token de acceso

Los Access_tokens tienen una duración breve y debe actualizarlos una vez expiren para seguir teniendo acceso a los recursos. Puede hacerlo mediante el envío de otra solicitud `POST` al punto de conexión `/token`, esta vez proporcionando el elemento `refresh_token` en lugar del elemento `code`.  Los tokens de actualización son válidos para todos los permisos para los que el cliente ya haya obtenido consentimiento; por tanto, un token de actualización emitido en una solicitud de `scope=mail.read` se puede usar para solicitar un nuevo token de acceso para `scope=api://contoso.com/api/UseResource`.  

Los tokens de actualización no tienen una duración especificada. Normalmente, las duraciones de este tipo de tokens son relativamente largas. Sin embargo, en algunos casos, los tokens de actualización expiran, se revocan o carecen de privilegios suficientes para realizar la acción deseada. La aplicación debe esperar y controlar los [errores que devuelve el punto de conexión de emisión de tokens](#error-codes-for-token-endpoint-errors) correctamente. 

Aunque no revocan los tokens de actualización cuando se usa para adquirir nuevos tokens de acceso, se espera que descarte el token de actualización anterior. El [especificación de OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-6) dice: "El servidor de autorización puede emitir un nuevo token de actualización, en el que caso el cliente debe descartar el token de actualización antiguo y reemplazarlo con el nuevo token de actualización. El servidor de autorización puede revocar el token de actualización antiguos después de emitir un nuevo token de actualización al cliente."  

```
// Line breaks for legibility only

POST /{tenant}/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&scope=https%3A%2F%2Fgraph.microsoft.com%2Fuser.read
&refresh_token=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq...
&grant_type=refresh_token
&client_secret=JqQX2PNo9bpM0uEihUPzyrh      // NOTE: Only required for web apps
```

> [!TIP]
> Pruebe a ejecutar esta solicitud en Postman (No olvide reemplazar `refresh_token`) [![Ejecución en Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)
> 

| Parámetro     |                | DESCRIPCIÓN        |
|---------------|----------------|--------------------|
| `tenant`        | requerido     | El valor `{tenant}` de la ruta de acceso de la solicitud se puede usar para controlar quién puede iniciar sesión en la aplicación. Los valores permitidos son `common`, `organizations`, `consumers` y los identificadores de inquilinos. Para obtener más información, consulte los [conceptos básicos sobre el protocolo](active-directory-v2-protocols.md#endpoints).   |
| `client_id`     | requerido    | El **Id. de aplicación (cliente)** que la [Azure portal: registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) experiencia asignado a la aplicación. |
| `grant_type`    | requerido    | Debe ser `refresh_token` para este segmento del flujo de código de autorización. |
| `scope`         | requerido    | Una lista de ámbitos separada por espacios. Los ámbitos solicitados en este segmento deben ser un subconjunto de los ámbitos solicitados en el segmento de la solicitud del  authorization_code original o un equivalente de este. Si los ámbitos especificados en esta solicitud abarcan varios servidores de recursos, el punto de conexión de plataforma de identidad de Microsoft devolverá un token para el recurso especificado en el primer ámbito. Para obtener una explicación más detallada de los ámbitos, consulte [permisos, consentimiento y ámbitos](v2-permissions-and-consent.md). |
| `refresh_token` | requerido    | El refresh_token que adquirió en el segundo segmento del flujo. |
| `client_secret` | necesario para las aplicaciones web | El secreto de la aplicación que creó en el portal de registro de aplicaciones para su aplicación. No se debe usar en una aplicación nativa, porque los client_secrets no se puede almacenar de forma confiable en los dispositivos. Es obligatorio para las aplicaciones web y API web, que tienen la capacidad de almacenar el client_secret de forma segura en el servidor. |

#### <a name="successful-response"></a>Respuesta correcta

Una respuesta de token correcta tendrá un aspecto similar al siguiente:

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...",
    "token_type": "Bearer",
    "expires_in": 3599,
    "scope": "https%3A%2F%2Fgraph.microsoft.com%2Fuser.read",
    "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4...",
    "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctOD...",
}
```
| Parámetro     | DESCRIPCIÓN         |
|---------------|-------------------------------------------------------------|
| `access_token`  | El token de acceso solicitado. La aplicación puede utilizar este token para autenticarse en el recursos protegido, como una API web. |
| `token_type`    | Indica el valor de tipo de token. El único tipo que admite Azure AD es el portador |
| `expires_in`    | Durante cuánto tiempo es válido el token de acceso (en segundos).   |
| `scope`         | Los ámbitos para los que el access_token es válido.    |
| `refresh_token` | Un nuevo token de actualización de OAuth 2.0. Debe reemplazar el token de actualización antiguo con este token de actualización recientemente adquirido para asegurar que los tokens de actualización siguen siendo válidos durante tanto tiempo como sea posible. <br> **Nota:** Solo se proporciona si se solicitó el ámbito `offline_access`.|
| `id_token`      | Un token web JSON (JWT) sin firmar. La aplicación puede decodificar los segmentos de este token para solicitar información acerca del usuario que ha iniciado sesión. La aplicación puede almacenar en caché los valores y mostrarlos, pero no debe confiar en ellos para cualquier autorización o límite de seguridad. Para más información sobre los parámetros id_tokens, consulte [`id_token reference`](id-tokens.md). <br> **Nota:** Solo se proporciona si se solicitó el ámbito `openid`. |

#### <a name="error-response"></a>Respuesta de error

```json
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/mail.read is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| Parámetro         | DESCRIPCIÓN                                                                                        |
|-------------------|----------------------------------------------------------------------------------------------------|
| `error`           | Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error de autenticación.           |
| `error_codes` |Una lista de los códigos de error específicos de STS que pueden ayudar en los diagnósticos. |
| `timestamp` | La hora a la que se produjo el error. |
| `trace_id` | Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos. |
| `correlation_id` | Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos entre componentes. |

Para ver una descripción de los códigos de error y la acción recomendada que tiene que realizar el cliente, consulte la sección [Códigos de error correspondientes a errores de puntos de conexión de token](#error-codes-for-token-endpoint-errors).
