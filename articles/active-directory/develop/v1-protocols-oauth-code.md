---
title: Descripción del flujo de código de autorización de OAuth 2.0 en Azure AD
description: En este artículo se describe cómo utilizar mensajes HTTP para autorizar el acceso a aplicaciones y API web en su inquilino con Azure Active Directory y OAuth 2.0.
services: active-directory
documentationcenter: .net
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/5/2019
ms.author: ryanwi
ms.reviewer: hirsin
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: d2e9177f91f016ffc64d542b9fad89d147b96f6d
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545148"
---
# <a name="authorize-access-to-azure-active-directory-web-applications-using-the-oauth-20-code-grant-flow"></a>Autorización del acceso a aplicaciones web de Azure Active Directory mediante el flujo de concesión de código OAuth 2.0

Azure Active Directory (Azure AD) utiliza el protocolo OAuth 2.0 para poder autorizar el acceso a aplicaciones y API web en su inquilino de Azure AD. En esta guía, que es independiente del lenguaje, se describe cómo enviar y recibir mensajes HTTP sin usar ninguna de nuestras [bibliotecas de código abierto](active-directory-authentication-libraries.md).

El flujo de código de autorización de OAuth 2.0 se describe en la [sección 4.1 de la especificación de OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-4.1). Se usa para realizar la autenticación y autorización en la mayoría de los tipos de aplicación, incluidas las aplicaciones web y las aplicaciones instaladas de forma nativa.

[!INCLUDE [active-directory-protocols-getting-started](../../../includes/active-directory-protocols-getting-started.md)]

## <a name="oauth-20-authorization-flow"></a>Flujo de autorización de OAuth 2.0

A nivel general, el flujo de autorización completo de una aplicación tiene un aspecto similar al siguiente:

![Flujo de código de autenticación de OAuth](./media/v1-protocols-oauth-code/active-directory-oauth-code-flow-native-app.png)

## <a name="request-an-authorization-code"></a>Solicitud de un código de autorización

El flujo del código de autorización comienza con el cliente dirigiendo al usuario al punto de conexión `/authorize` . En esta solicitud, el cliente indica los permisos que necesita obtener del usuario. Puede obtener el punto de conexión de autorización de OAuth 2.0 para el inquilino seleccionando **Registros de aplicaciones > Puntos de conexión** en Azure Portal.

```
// Line breaks for legibility only

https://login.microsoftonline.com/{tenant}/oauth2/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%3A12345
&response_mode=query
&resource=https%3A%2F%2Fservice.contoso.com%2F
&state=12345
```

| Parámetro |  | DESCRIPCIÓN |
| --- | --- | --- |
| tenant |requerido |El valor `{tenant}` de la ruta de acceso de la solicitud se puede usar para controlar quién puede iniciar sesión en la aplicación. Los valores permitidos son los identificadores de inquilino; por ejemplo, `8eaef023-2b34-4da1-9baa-8bc8c9d6a490`, `contoso.onmicrosoft.com` o `common` para los tokens independientes del inquilino. |
| client_id |requerido |Identificador de aplicación asignado a la aplicación cuando se registra en Azure AD. Puede encontrarlo en Azure Portal. Haga clic en **Azure Active Directory** en la barra lateral de servicios, haga clic en **Registros de aplicaciones** y elija la aplicación. |
| response_type |requerido |Debe incluir `code` para el flujo de código de autorización. |
| redirect_uri |recomendado |El redirect_uri de su aplicación, a donde su aplicación puede enviar y recibir las respuestas de autenticación. Debe coincidir exactamente con uno de los redirect_uris que registró en el portal, con la excepción de que debe estar codificado como URL. En el caso de las aplicaciones nativas y móviles, es preciso usar el valor predeterminado, `urn:ietf:wg:oauth:2.0:oob`. |
| response_mode |opcional |Especifica el método que debe usarse para enviar el token resultante de nuevo a la aplicación. Puede ser `query`, `fragment` o `form_post`. `query` proporciona el código como un parámetro de cadena de consulta en el URI de redirección. Si solicita un token de identificador con el flujo implícito, no puede usar `query` según lo indicado en la [especificación de OpenID](https://openid.net/specs/oauth-v2-multiple-response-types-1_0.html#Combinations). Si solicita solo el código, puede usar `query`, `fragment` o `form_post`. `form_post` ejecuta una prueba POST que contiene el código para el URI de redirección. El valor predeterminado es `query` para un flujo de código.  |
| state |recomendado |Un valor incluido en la solicitud que también se devolverá en la respuesta del token. Normalmente se usa un valor único generado de forma aleatoria para [evitar los ataques de falsificación de solicitudes entre sitios](https://tools.ietf.org/html/rfc6749#section-10.12). El estado también se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, por ejemplo, la página o vista en la que estaban. |
| Recurso | recomendado |URI del identificador de la aplicación de la API web de destino (recurso seguro). Para buscar el URI del identificador de la aplicación, en Azure Portal, haga clic en **Azure Active Directory** y en **Application registrations** (Registros de aplicaciones), abra la página de **Configuración** de la aplicación y, a continuación, haga clic en **Propiedades**. También puede ser un recurso externo, como `https://graph.microsoft.com`. Esto es necesario en una de las solicitudes de la autorización o del token. A fin de realizar menos solicitudes de confirmación de autenticación, colóquelo en la solicitud de autorización para asegurarse de que se recibe el consentimiento del usuario. |
| ámbito | **ignorado** | En el caso de las aplicaciones de Azure AD v1, los ámbitos se deben configurar estáticamente en Azure Portal en las aplicaciones **Configuración** y **Permisos necesarios**. |
| símbolo del sistema |opcional |Indica el tipo de interacción necesaria con el usuario.<p> Los valores válidos son: <p> *login*: se le solicitará al usuario que vuelva a autenticarse. <p> *select_account*: se solicita al usuario seleccionar una cuenta e interrumpir el inicio de sesión único. El usuario puede seleccionar una cuenta existente en la que ya se haya iniciado sesión, escribir las credenciales de una cuenta guardada o decidir usar una cuenta diferente por completo. <p> *consent*: se le ha concedido el consentimiento al usuario, pero debe actualizarse. Se le solicitará al usuario consentimiento. <p> *admin_consent*: se le solicitará al administrador que dé consentimiento en nombre de todos los usuarios de su organización. |
| login_hint |opcional |Puede usarse para rellenar previamente el campo de nombre de usuario y dirección de correo electrónico de la página de inicio de sesión del usuario, si sabe su nombre de usuario con antelación. A menudo las aplicaciones usan este parámetro durante la reautenticación, dado que ya han extraído el nombre de usuario de un inicio de sesión anterior mediante la notificación `preferred_username`. |
| domain_hint |opcional |Proporciona una sugerencia sobre el inquilino o dominio que el usuario debe utilizar para iniciar sesión. El valor de domain_hint es un dominio registrado para el inquilino. Si el inquilino está federado en un directorio local, AAD le redirige al servidor de federación del inquilino especificado. |
| code_challenge_method | recomendado    | Método utilizado para codificar `code_verifier` para el parámetro `code_challenge`. Puede ser `plain` o `S256`. Si se excluye, se supone que `code_challenge` es texto no cifrado si se incluye `code_challenge`. Azure AAD v1.0 admite `plain` y `S256`. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636). |
| code_challenge        | recomendado    | Se usa para proteger concesiones de código de autorización a través de la clave de prueba para el intercambio de códigos (PKCE) desde un cliente público o nativo. Se requiere si se incluye `code_challenge_method`. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636). |

> [!NOTE]
> Si el usuario forma parte de una organización, un administrador de la organización puede mostrar su aprobación o rechazo en nombre del usuario, o bien permitir que el usuario dé su consentimiento. El usuario tiene la opción de dar su consentimiento solo cuando el administrador lo permite.
>
>

En este punto, se pide al usuario que escriba sus credenciales y dé su consentimiento a los permisos solicitados por la aplicación en Azure Portal. Una vez que el usuario se autentica y da su consentimiento, Azure AD envía una respuesta a la aplicación en la dirección `redirect_uri` de la solicitud, junto con el código.

### <a name="successful-response"></a>Respuesta correcta
Una respuesta correcta podría tener el siguiente aspecto:

```
GET  HTTP/1.1 302 Found
Location: http://localhost:12345/?code= AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrqqf_ZT_p5uEAEJJ_nZ3UmphWygRNy2C3jJ239gV_DBnZ2syeg95Ki-374WHUP-i3yIhv5i-7KU2CEoPXwURQp6IVYMw-DjAOzn7C3JCu5wpngXmbZKtJdWmiBzHpcO2aICJPu1KvJrDLDP20chJBXzVYJtkfjviLNNW7l7Y3ydcHDsBRKZc3GuMQanmcghXPyoDg41g8XbwPudVh7uCmUponBQpIhbuffFP_tbV8SNzsPoFz9CLpBCZagJVXeqWoYMPe2dSsPiLO9Alf_YIe5zpi-zY4C3aLw5g9at35eZTfNd0gBRpR5ojkMIcZZ6IgAA&session_state=7B29111D-C220-4263-99AB-6F6E135D75EF&state=D79E5777-702E-4260-9A62-37F75FF22CCE
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| admin_consent |El valor es True si un administrador ha dado su consentimiento a una solicitud de consentimiento. |
| código |El código de autorización que solicitó la aplicación. La aplicación puede utilizar el código de autorización con el fin de solicitar un token de acceso para el recurso de destino. |
| session_state |Un valor único que identifica la sesión del usuario actual. Este valor es un GUID, pero se debe tratar como un valor opaco que se pasa sin analizar. |
| state |Si se incluye un parámetro de estado en la solicitud, debería aparecer el mismo valor en la respuesta. Se recomienda que la aplicación compruebe que los valores de estado de la solicitud y la respuesta son idénticos antes de usar la respuesta. Esto ayuda a detectar los [ataques de falsificación de solicitud entre sitios (CSRF)](https://tools.ietf.org/html/rfc6749#section-10.12) que se producen en el cliente. |

### <a name="error-response"></a>Respuesta de error
Las respuestas de error también pueden enviarse al elemento `redirect_uri` para que la aplicación pueda controlarlas adecuadamente.

```
GET http://localhost:12345/?
error=access_denied
&error_description=the+user+canceled+the+authentication
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| error |Valor de código de error definido en la sección 5.2 del [marco de autorización de OAuth 2.0](https://tools.ietf.org/html/rfc6749). En la tabla siguiente se describen los códigos de error que devuelve Azure AD. |
| error_description |Descripción más detallada del error. Este mensaje no está diseñado para que el usuario final lo comprenda sin problemas. |
| state |El valor de estado es un valor no reutilizado y generado aleatoriamente que se envía en la solicitud y que se devuelve en la respuesta con el fin de evitar ataques de falsificación de solicitud entre sitios (CSRF). |

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

## <a name="use-the-authorization-code-to-request-an-access-token"></a>Uso del código de autorización para solicitar un token de acceso
Ahora que ha obtenido un código de autorización y el usuario le ha concedido permiso, puede canjear el código por un token de acceso al recurso deseado mediante el envío de una solicitud POST al punto de conexión `/token` :

```
// Line breaks for legibility only

POST /{tenant}/oauth2/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded
grant_type=authorization_code
&client_id=2d4d11a2-f814-46a7-890a-274a72a7309e
&code=AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrqqf_ZT_p5uEAEJJ_nZ3UmphWygRNy2C3jJ239gV_DBnZ2syeg95Ki-374WHUP-i3yIhv5i-7KU2CEoPXwURQp6IVYMw-DjAOzn7C3JCu5wpngXmbZKtJdWmiBzHpcO2aICJPu1KvJrDLDP20chJBXzVYJtkfjviLNNW7l7Y3ydcHDsBRKZc3GuMQanmcghXPyoDg41g8XbwPudVh7uCmUponBQpIhbuffFP_tbV8SNzsPoFz9CLpBCZagJVXeqWoYMPe2dSsPiLO9Alf_YIe5zpi-zY4C3aLw5g9at35eZTfNd0gBRpR5ojkMIcZZ6IgAA
&redirect_uri=https%3A%2F%2Flocalhost%3A12345
&resource=https%3A%2F%2Fservice.contoso.com%2F
&client_secret=p@ssw0rd

//NOTE: client_secret only required for web apps
```

| Parámetro |  | DESCRIPCIÓN |
| --- | --- | --- |
| tenant |requerido |El valor `{tenant}` de la ruta de acceso de la solicitud se puede usar para controlar quién puede iniciar sesión en la aplicación. Los valores permitidos son los identificadores de inquilino; por ejemplo, `8eaef023-2b34-4da1-9baa-8bc8c9d6a490`, `contoso.onmicrosoft.com` o `common` para los tokens independientes del inquilino. |
| client_id |requerido |Identificador de aplicación asignado a la aplicación cuando se registra en Azure AD. Puede encontrarlo en Azure Portal. El identificador de aplicación se muestra en la configuración del registro de la aplicación. |
| grant_type |requerido |Debe ser `authorization_code` para el flujo de código de autorización. |
| código |requerido |El elemento `authorization_code` que obtuvo en la sección anterior. |
| redirect_uri |requerido | Un `redirect_uri`registrado en la aplicación cliente. |
| client_secret |necesario para aplicaciones web, no se permite para clientes públicos |Secreto de la aplicación que creó en Azure Portal para su aplicación en **Claves**. No puede utilizarse en una aplicación nativa (cliente público), porque los client_secrets no se pueden almacenar de forma confiable en los dispositivos. Es necesario para aplicaciones y API web (todos los clientes confidenciales), que tienen la capacidad de almacenar `client_secret` de forma segura en el servidor. El secreto de cliente debería codificarse como dirección URL antes de enviarse. |
| Recurso | recomendado |URI del identificador de la aplicación de la API web de destino (recurso seguro). Para buscar el URI del identificador de la aplicación, en Azure Portal, haga clic en **Azure Active Directory** y en **Application registrations** (Registros de aplicaciones), abra la página de **Configuración** de la aplicación y, a continuación, haga clic en **Propiedades**. También puede ser un recurso externo, como `https://graph.microsoft.com`. Esto es necesario en una de las solicitudes de la autorización o del token. A fin de realizar menos solicitudes de confirmación de autenticación, colóquelo en la solicitud de autorización para asegurarse de que se recibe el consentimiento del usuario. Si está tanto en la solicitud de autorización como en la solicitud de token, los parámetros del recurso deben coincidir. | 
| code_verifier | opcional | El mismo valor de code_verifier que usó para obtener el valor de authorization_code. Se requiere si PKCE se utilizó en la solicitud de concesión de código de autorización. Para obtener más información, consulte [PKCE RFC](https://tools.ietf.org/html/rfc7636).   |

Para buscar el URI del identificador de la aplicación, en Azure Portal, haga clic en **Azure Active Directory** y en **Application registrations** (Registros de aplicaciones), abra la página de **Configuración** de la aplicación y, a continuación, haga clic en **Propiedades**.

### <a name="successful-response"></a>Respuesta correcta
Azure AD devuelve un [token de acceso](access-tokens.md) tras una respuesta correcta. Para minimizar las llamadas de red realizadas desde la aplicación cliente y la latencia asociada, la aplicación cliente debe almacenar en caché los tokens de acceso durante el tiempo que se especifica en la respuesta de OAuth 2.0. Para determinar la duración del token, utilice los valores de parámetro `expires_in` o `expires_on`.

Si un recurso de API web devuelve un código de error `invalid_token` , podría significar que el recurso ha determinado que el token ha expirado. Si las horas de reloj del cliente y los recursos son diferentes (fenómeno conocido como "sesgo horario"), el recurso podría considerar que el token expire antes de que se borre de la memoria caché del cliente. Si esto ocurre, borre el token de la caché, aunque se encuentre todavía dentro de la duración calculada.

Una respuesta correcta podría tener el siguiente aspecto:

```
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1THdqcHdBSk9NOW4tQSJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0MDg2MywibmJmIjoxMzg4NDQwODYzLCJleHAiOjEzODg0NDQ3NjMsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6IjY4Mzg5YWUyLTYyZmEtNGIxOC05MWZlLTUzZGQxMDlkNzRmNSIsInVwbiI6ImZyYW5rbUBjb250b3NvLmNvbSIsInVuaXF1ZV9uYW1lIjoiZnJhbmttQGNvbnRvc28uY29tIiwic3ViIjoiZGVOcUlqOUlPRTlQV0pXYkhzZnRYdDJFYWJQVmwwQ2o4UUFtZWZSTFY5OCIsImZhbWlseV9uYW1lIjoiTWlsbGVyIiwiZ2l2ZW5fbmFtZSI6IkZyYW5rIiwiYXBwaWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJhcHBpZGFjciI6IjAiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJhY3IiOiIxIn0.JZw8jC0gptZxVC-7l5sFkdnJgP3_tRjeQEPgUn28XctVe3QqmheLZw7QVZDPCyGycDWBaqy7FLpSekET_BftDkewRhyHk9FW_KeEz0ch2c3i08NGNDbr6XYGVayNuSesYk5Aw_p3ICRlUV1bqEwk-Jkzs9EEkQg4hbefqJS6yS1HoV_2EsEhpd_wCQpxK89WPs3hLYZETRJtG5kvCCEOvSHXmDE6eTHGTnEgsIk--UlPe275Dvou4gEAwLofhLDQbMSjnlV5VLsjimNBVcSRFShoxmQwBJR_b2011Y5IuD6St5zPnzruBbZYkGNurQK63TJPWmRd3mbJsGM0mf3CUQ",
  "token_type": "Bearer",
  "expires_in": "3600",
  "expires_on": "1388444763",
  "resource": "https://service.contoso.com/",
  "refresh_token": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4rTfgV29ghDOHRc2B-C_hHeJaJICqjZ3mY2b_YNqmf9SoAylD1PycGCB90xzZeEDg6oBzOIPfYsbDWNf621pKo2Q3GGTHYlmNfwoc-OlrxK69hkha2CF12azM_NYhgO668yfcUl4VBbiSHZyd1NVZG5QTIOcbObu3qnLutbpadZGAxqjIbMkQ2bQS09fTrjMBtDE3D6kSMIodpCecoANon9b0LATkpitimVCrl-NyfN3oyG4ZCWu18M9-vEou4Sq-1oMDzExgAf61noxzkNiaTecM-Ve5cq6wHqYQjfV9DOz4lbceuYCAA",
  "scope": "https%3A%2F%2Fgraph.microsoft.com%2Fmail.read",
  "id_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJhdWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJpc3MiOiJodHRwczovL3N0cy53aW5kb3dzLm5ldC83ZmU4MTQ0Ny1kYTU3LTQzODUtYmVjYi02ZGU1N2YyMTQ3N2UvIiwiaWF0IjoxMzg4NDQwODYzLCJuYmYiOjEzODg0NDA4NjMsImV4cCI6MTM4ODQ0NDc2MywidmVyIjoiMS4wIiwidGlkIjoiN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlIiwib2lkIjoiNjgzODlhZTItNjJmYS00YjE4LTkxZmUtNTNkZDEwOWQ3NGY1IiwidXBuIjoiZnJhbmttQGNvbnRvc28uY29tIiwidW5pcXVlX25hbWUiOiJmcmFua21AY29udG9zby5jb20iLCJzdWIiOiJKV3ZZZENXUGhobHBTMVpzZjd5WVV4U2hVd3RVbTV5elBtd18talgzZkhZIiwiZmFtaWx5X25hbWUiOiJNaWxsZXIiLCJnaXZlbl9uYW1lIjoiRnJhbmsifQ."
}

```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| access_token |El [token de acceso](access-tokens.md) solicitado como un JSON Web Token (JWT) firmado. La aplicación puede utilizar este token para autenticar a los recursos protegidos, como una API web. |
| token_type |Indica el valor de tipo de token. El único tipo que admite Azure AD es portador. Para más información sobre los tokens de portador, consulte [The OAuth2.0 Authorization Framework: Bearer Token Usage (RFC 6750)](https://www.rfc-editor.org/rfc/rfc6750.txt) (Marco de autorización de OAuth2.0: uso del token de portador [RFC 6750]). |
| expires_in |Durante cuánto tiempo es válido el token de acceso (en segundos). |
| expires_on |La hora a la que expira el token de acceso. La fecha se representa como el número de segundos desde 1970-01-01T0:0:0Z UTC hasta la fecha de expiración. Este valor se utiliza para determinar la duración de los tokens almacenados en caché. |
| Recurso |El URI del identificador de la aplicación de la API web (recurso seguro). |
| ámbito |Permisos de suplantación concedidos a la aplicación cliente. El permiso predeterminado es `user_impersonation`. El propietario del recurso protegido puede registrar valores adicionales en Azure AD. |
| refresh_token |Un token de actualización de OAuth 2.0. La aplicación puede utilizar este token para obtener más tokens de acceso una vez que expire el token de acceso actual. Los tokens de actualización son de larga duración y pueden usarse para conservar el acceso a los recursos durante largos periodos. |
| ID_token |Un JSON Web Token (JWT) sin signo que representa un [identificador de token](id-tokens.md). La aplicación puede descodificar base64Url en los segmentos de este token para solicitar información acerca del usuario que ha iniciado sesión. La aplicación puede almacenar en caché los valores y mostrarlos, pero no debe confiar en ellos para cualquier autorización o límite de seguridad. |

Para más información sobre los tokens web JSON, consulte la [especificación de borrador de JWT IETF](https://go.microsoft.com/fwlink/?LinkId=392344).   Para más información acerca de `id_tokens`, consulte el [flujo de conexión de OpenID de v1.0](v1-protocols-openid-connect-code.md).

### <a name="error-response"></a>Respuesta de error
Los errores de puntos de conexión de emisión de tokens son los códigos de error HTTP, ya que el cliente llama directamente al punto de conexión de emisión de tokens. Además del código de estado HTTP, el punto de conexión de emisión de tokens de Azure AD también devuelve un documento JSON con objetos que describen el error.

Una respuesta de error de ejemplo podría tener este aspecto:

```
{
  "error": "invalid_grant",
  "error_description": "AADSTS70002: Error validating credentials. AADSTS70008: The provided authorization code or refresh token is expired. Send a new interactive authorization request for this user and resource.\r\nTrace ID: 3939d04c-d7ba-42bf-9cb7-1e5854cdce9e\r\nCorrelation ID: a8125194-2dc8-4078-90ba-7b6592a7f231\r\nTimestamp: 2016-04-11 18:00:12Z",
  "error_codes": [
    70002,
    70008
  ],
  "timestamp": "2016-04-11 18:00:12Z",
  "trace_id": "3939d04c-d7ba-42bf-9cb7-1e5854cdce9e",
  "correlation_id": "a8125194-2dc8-4078-90ba-7b6592a7f231"
}
```
| Parámetro | DESCRIPCIÓN |
| --- | --- |
| error |Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| error_description |Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error de autenticación. |
| error_codes |Una lista de los códigos de error específicos de STS que pueden ayudar en los diagnósticos. |
|  timestamp |La hora a la que se produjo el error. |
| trace_id |Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos. |
| correlation_id |Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos entre componentes. |

#### <a name="http-status-codes"></a>Códigos de estado HTTP
En la tabla siguiente se enumeran los códigos de estado HTTP que devuelve el punto de conexión de emisión de tokens. En algunos casos, con el código de error basta para describir la respuesta, pero, si hay errores, se debe analizar el documento JSON que la acompaña y examinar su código de error.

| Código HTTP | DESCRIPCIÓN |
| --- | --- |
| 400 |Código HTTP predeterminado. Se utiliza en la mayoría de los casos y suele originarse debido a una solicitud con formato incorrecto. Corrija el error y vuelva a enviar la solicitud. |
| 401 |Error de autenticación. Por ejemplo, la solicitud no tiene el parámetro client_secret. |
| 403 |Error de autorización. Por ejemplo, el usuario no tiene permiso para acceder al recurso. |
| 500 |Se ha producido un error interno en el servicio. Vuelva a intentarlo. |

#### <a name="error-codes-for-token-endpoint-errors"></a>Códigos de error correspondientes a errores de puntos de conexión de token
| Código de error | DESCRIPCIÓN | Acción del cliente |
| --- | --- | --- |
| invalid_request |Error de protocolo, como un parámetro obligatorio que falta. |Corrija el error y vuelva a enviar la solicitud. |
| invalid_grant |El código de autorización no es válido o ha expirado. |Trate de iniciar una nueva solicitud al punto de conexión `/authorize`. |
| unauthorized_client |El cliente autenticado no está autorizado para usar este tipo de concesión de autorización. |Este error suele producirse cuando la aplicación cliente no está registrada en Azure AD o no se ha agregado al inquilino de Azure AD del usuario. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |
| invalid_client |Se produjo un error de autenticación de cliente. |Las credenciales del cliente no son válidas. Para corregirlo, el administrador de la aplicación actualiza las credenciales. |
| unsupported_grant_type |El servidor de autorización no admite el tipo de concesión de autorización. |Cambie el tipo de concesión de la solicitud. Este tipo de error solo debe producirse durante el desarrollo y detectarse en las pruebas iniciales. |
| invalid_resource |El recurso de destino no es válido porque no existe, Azure AD no lo encuentra o no está configurado correctamente. |Este error indica que el recurso, en caso de que exista, no se ha configurado en el inquilino. La aplicación puede pedir al usuario consentimiento para instalar la aplicación y agregarla a Azure AD. |
| interaction_required |La solicitud requiere la interacción del usuario. Por ejemplo, hay que realizar un paso de autenticación más. | En lugar de una solicitud no interactiva, reintente con una solicitud de autorización interactiva para el mismo recurso. |
| temporarily_unavailable |De manera temporal, el servidor está demasiado ocupado para atender la solicitud. |Vuelva a intentarlo. La aplicación podría explicar al usuario que su respuesta se retrasó debido a una condición temporal. |

## <a name="use-the-access-token-to-access-the-resource"></a>Uso del token de acceso para acceder al recurso
Ahora que ha adquirido correctamente un `access_token`, puede usar el token en solicitudes a las API web mediante su inclusión en el encabezado `Authorization`. En la especificación [RFC 6750](https://www.rfc-editor.org/rfc/rfc6750.txt) se explica cómo utilizar los tokens de portador en solicitudes HTTP para acceder a recursos protegidos.

### <a name="sample-request"></a>Solicitud de ejemplo
```
GET /data HTTP/1.1
Host: service.contoso.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1THdqcHdBSk9NOW4tQSJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0MDg2MywibmJmIjoxMzg4NDQwODYzLCJleHAiOjEzODg0NDQ3NjMsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6IjY4Mzg5YWUyLTYyZmEtNGIxOC05MWZlLTUzZGQxMDlkNzRmNSIsInVwbiI6ImZyYW5rbUBjb250b3NvLmNvbSIsInVuaXF1ZV9uYW1lIjoiZnJhbmttQGNvbnRvc28uY29tIiwic3ViIjoiZGVOcUlqOUlPRTlQV0pXYkhzZnRYdDJFYWJQVmwwQ2o4UUFtZWZSTFY5OCIsImZhbWlseV9uYW1lIjoiTWlsbGVyIiwiZ2l2ZW5fbmFtZSI6IkZyYW5rIiwiYXBwaWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJhcHBpZGFjciI6IjAiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJhY3IiOiIxIn0.JZw8jC0gptZxVC-7l5sFkdnJgP3_tRjeQEPgUn28XctVe3QqmheLZw7QVZDPCyGycDWBaqy7FLpSekET_BftDkewRhyHk9FW_KeEz0ch2c3i08NGNDbr6XYGVayNuSesYk5Aw_p3ICRlUV1bqEwk-Jkzs9EEkQg4hbefqJS6yS1HoV_2EsEhpd_wCQpxK89WPs3hLYZETRJtG5kvCCEOvSHXmDE6eTHGTnEgsIk--UlPe275Dvou4gEAwLofhLDQbMSjnlV5VLsjimNBVcSRFShoxmQwBJR_b2011Y5IuD6St5zPnzruBbZYkGNurQK63TJPWmRd3mbJsGM0mf3CUQ
```

### <a name="error-response"></a>Respuesta de error
Los recursos protegidos que implementan esta especificación emiten códigos de error HTTP. Si la solicitud no incluye las credenciales de autenticación o si falta el token, la respuesta incluirá un encabezado `WWW-Authenticate` . Cuando se produce un error en una solicitud, el servidor de recursos responde con el código de estado HTTP y un código de error.

A continuación, se muestra un ejemplo de una respuesta incorrecta cuando la solicitud de cliente no incluye el token de portador:

```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: Bearer authorization_uri="https://login.microsoftonline.com/contoso.com/oauth2/authorize",  error="invalid_token",  error_description="The access token is missing.",
```

#### <a name="error-parameters"></a>Parámetros de error
| Parámetro | DESCRIPCIÓN |
| --- | --- |
| authorization_uri |URI (punto de conexión físico) del servidor de autorización. Este valor también se utiliza como clave de búsqueda con el fin de obtener más información del servidor a partir de un punto de conexión de detección. <p><p>  El cliente debe validar que el servidor de autorización sea de confianza. Cuando el recurso está protegido por Azure AD, basta con comprobar que la dirección URL comienza por https://login.microsoftonline.com u otro nombre de host que admita Azure AD. Los recursos específicos del inquilino siempre deben devolver un URI de autorización exclusivo del inquilino. |
| error |Valor de código de error definido en la sección 5.2 del [marco de autorización de OAuth 2.0](https://tools.ietf.org/html/rfc6749). |
| error_description |Descripción más detallada del error. Este mensaje no está diseñado para que el usuario final lo comprenda sin problemas. |
| resource_id |Devuelve el identificador único del recurso. La aplicación cliente puede utilizar este identificador como valor del parámetro `resource` cuando se solicita un token para el recurso. <p><p> Es importante para la aplicación cliente comprobar este valor; de lo contrario, un servicio malintencionado puede inducir un ataque por **elevación de privilegios**. <p><p> La estrategia recomendada para prevenir ataques consiste en comprobar que `resource_id` coincide con la base de la dirección URL de la API web que tiene acceso. Por ejemplo, si se accede a https://service.contoso.com/data, `resource_id` puede ser https://service.contoso.com/. La aplicación cliente debe rechazar un parámetro `resource_id` que no empiece por la dirección URL base, salvo que exista una alternativa confiable para comprobar el id. |

#### <a name="bearer-scheme-error-codes"></a>Códigos de error del esquema de portador
La especificación RFC 6750 define los siguientes errores de los recursos que utilizan el encabezado WWW-Authenticate y el esquema de portador en la respuesta.

| Código de estado HTTP | Código de error | DESCRIPCIÓN | Acción del cliente |
| --- | --- | --- | --- |
| 400 |invalid_request |La solicitud no tiene el formato correcto. Por ejemplo, podría faltar un parámetro o se está utilizando el mismo parámetro dos veces. |Corrija el error y vuelva a tratar de realizar la solicitud. Este tipo de error solo debe producirse durante el desarrollo y detectarse en las pruebas iniciales. |
| 401 |invalid_token |El token de acceso no es válido, falta o se ha revocado. El valor del parámetro error_description proporciona más información. |Solicite un nuevo token desde el servidor de autorización. Si el nuevo token no funciona, significa que se ha producido un error inesperado. Envíe un mensaje de error para el usuario y vuelva a intentarlo después de los retrasos aleatorios. |
| 403 |insufficient_scope |El token de acceso no contiene los permisos de suplantación necesarios para acceder al recurso. |Envíe una nueva solicitud de autorización al punto de conexión de autorización. Si la respuesta contiene el parámetro de ámbito, utilice el valor de ámbito de la solicitud en el recurso. |
| 403 |insufficient_access |El asunto del token no tiene los permisos necesarios para acceder al recurso. |Pida al usuario que utilice una cuenta diferente o que solicite permisos al recurso especificado. |

## <a name="refreshing-the-access-tokens"></a>Actualización de los tokens de acceso

Los tokens de acceso tienen una duración breve y debe actualizarlos una vez expiren para seguir teniendo acceso a los recursos. Puede hacerlo el elemento `access_token` enviando otra solicitud `POST` al punto de conexión `/token`, pero esta vez proporcionando `refresh_token` en lugar de `code`.  Los tokens de actualización son válidos para todos los recursos para los que su cliente ya haya obtenido consentimiento de acceso; por tanto, un token de actualización emitido en una solicitud de `resource=https://graph.microsoft.com` se puede usar para solicitar un nuevo token de acceso para `resource=https://contoso.com/api`. 

Los tokens de actualización no tienen una duración especificada. Normalmente, las duraciones de este tipo de tokens son relativamente largas. Sin embargo, en algunos casos, los tokens de actualización expiran, se revocan o carecen de privilegios suficientes para realizar la acción deseada. Su aplicación debe esperar y controlar los errores que devuelve el punto de conexión de emisión de tokens correctamente.

Cuando reciba una respuesta con un error de actualización de token, descarte el token de actualización actual y solicite un nuevo código de autorización o token de acceso. En concreto, al utilizar un token de actualización en el flujo de concesión del código de autorización, si recibe una respuesta con los códigos de error `interaction_required` o `invalid_grant`, descarte el token de actualización y solicite un nuevo código de autorización.

Una solicitud de ejemplo al punto de conexión **específico del inquilino** (también puede utilizar el punto de conexión **común**) para obtener un nuevo token de acceso con un token de actualización sería similar a la siguiente:

```
// Line breaks for legibility only

POST /{tenant}/oauth2/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&refresh_token=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq...
&grant_type=refresh_token
&resource=https%3A%2F%2Fservice.contoso.com%2F
&client_secret=JqQX2PNo9bpM0uEihUPzyrh    // NOTE: Only required for web apps
```

### <a name="successful-response"></a>Respuesta correcta
Una respuesta de token correcta tendrá un aspecto similar al siguiente:

```
{
  "token_type": "Bearer",
  "expires_in": "3600",
  "expires_on": "1460404526",
  "resource": "https://service.contoso.com/",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1THdqcHdBSk9NOW4tQSJ9.eyJhdWQiOiJodHRwczovL3NlcnZpY2UuY29udG9zby5jb20vIiwiaXNzIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvN2ZlODE0NDctZGE1Ny00Mzg1LWJlY2ItNmRlNTdmMjE0NzdlLyIsImlhdCI6MTM4ODQ0MDg2MywibmJmIjoxMzg4NDQwODYzLCJleHAiOjEzODg0NDQ3NjMsInZlciI6IjEuMCIsInRpZCI6IjdmZTgxNDQ3LWRhNTctNDM4NS1iZWNiLTZkZTU3ZjIxNDc3ZSIsIm9pZCI6IjY4Mzg5YWUyLTYyZmEtNGIxOC05MWZlLTUzZGQxMDlkNzRmNSIsInVwbiI6ImZyYW5rbUBjb250b3NvLmNvbSIsInVuaXF1ZV9uYW1lIjoiZnJhbmttQGNvbnRvc28uY29tIiwic3ViIjoiZGVOcUlqOUlPRTlQV0pXYkhzZnRYdDJFYWJQVmwwQ2o4UUFtZWZSTFY5OCIsImZhbWlseV9uYW1lIjoiTWlsbGVyIiwiZ2l2ZW5fbmFtZSI6IkZyYW5rIiwiYXBwaWQiOiIyZDRkMTFhMi1mODE0LTQ2YTctODkwYS0yNzRhNzJhNzMwOWUiLCJhcHBpZGFjciI6IjAiLCJzY3AiOiJ1c2VyX2ltcGVyc29uYXRpb24iLCJhY3IiOiIxIn0.JZw8jC0gptZxVC-7l5sFkdnJgP3_tRjeQEPgUn28XctVe3QqmheLZw7QVZDPCyGycDWBaqy7FLpSekET_BftDkewRhyHk9FW_KeEz0ch2c3i08NGNDbr6XYGVayNuSesYk5Aw_p3ICRlUV1bqEwk-Jkzs9EEkQg4hbefqJS6yS1HoV_2EsEhpd_wCQpxK89WPs3hLYZETRJtG5kvCCEOvSHXmDE6eTHGTnEgsIk--UlPe275Dvou4gEAwLofhLDQbMSjnlV5VLsjimNBVcSRFShoxmQwBJR_b2011Y5IuD6St5zPnzruBbZYkGNurQK63TJPWmRd3mbJsGM0mf3CUQ",
  "refresh_token": "AwABAAAAv YNqmf9SoAylD1PycGCB90xzZeEDg6oBzOIPfYsbDWNf621pKo2Q3GGTHYlmNfwoc-OlrxK69hkha2CF12azM_NYhgO668yfcUl4VBbiSHZyd1NVZG5QTIOcbObu3qnLutbpadZGAxqjIbMkQ2bQS09fTrjMBtDE3D6kSMIodpCecoANon9b0LATkpitimVCrl PM1KaPlrEqdFSBzjqfTGAMxZGUTdM0t4B4rTfgV29ghDOHRc2B-C_hHeJaJICqjZ3mY2b_YNqmf9SoAylD1PycGCB90xzZeEDg6oBzOIPfYsbDWNf621pKo2Q3GGTHYlmNfwoc-OlrxK69hkha2CF12azM_NYhgO668yfmVCrl-NyfN3oyG4ZCWu18M9-vEou4Sq-1oMDzExgAf61noxzkNiaTecM-Ve5cq6wHqYQjfV9DOz4lbceuYCAA"
}
```
| Parámetro | DESCRIPCIÓN |
| --- | --- |
| token_type |El tipo de token. El único valor admitido es **bearer**. |
| expires_in |Duración restante del token en segundos. Un valor típico es 3600 (una hora). |
| expires_on |La fecha y hora en que expira el token. La fecha se representa como el número de segundos desde 1970-01-01T0:0:0Z UTC hasta la fecha de expiración. |
| Recurso |Identifica el recurso protegido que el token de acceso puede utilizar para acceder. |
| ámbito |Permisos de suplantación concedidos a la aplicación cliente nativa. El permiso predeterminado es **user_impersonation**. El propietario del recurso de destino puede registrar valores alternativos en Azure AD. |
| access_token |El nuevo token de acceso que se solicitó. |
| refresh_token |Nuevo refresh_token de OAuth 2.0 que puede usarse para solicitar nuevos tokens de acceso cuando expira en esta respuesta. |

### <a name="error-response"></a>Respuesta de error
Una respuesta de error de ejemplo podría tener este aspecto:

```
{
  "error": "invalid_resource",
  "error_description": "AADSTS50001: The application named https://foo.microsoft.com/mail.read was not found in the tenant named 295e01fc-0c56-4ac3-ac57-5d0ed568f872. This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant. You might have sent your authentication request to the wrong tenant.\r\nTrace ID: ef1f89f6-a14f-49de-9868-61bd4072f0a9\r\nCorrelation ID: b6908274-2c58-4e91-aea9-1f6b9c99347c\r\nTimestamp: 2016-04-11 18:59:01Z",
  "error_codes": [
    50001
  ],
  "timestamp": "2016-04-11 18:59:01Z",
  "trace_id": "ef1f89f6-a14f-49de-9868-61bd4072f0a9",
  "correlation_id": "b6908274-2c58-4e91-aea9-1f6b9c99347c"
}
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| error |Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| error_description |Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error de autenticación. |
| error_codes |Una lista de los códigos de error específicos de STS que pueden ayudar en los diagnósticos. |
|  timestamp |La hora a la que se produjo el error. |
| trace_id |Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos. |
| correlation_id |Un identificador exclusivo para la solicitud que puede ayudar en los diagnósticos entre componentes. |

Para ver una descripción de los códigos de error y la acción recomendada que tiene que realizar el cliente, consulte la sección [Códigos de error correspondientes a errores de puntos de conexión de token](#error-codes-for-token-endpoint-errors).
