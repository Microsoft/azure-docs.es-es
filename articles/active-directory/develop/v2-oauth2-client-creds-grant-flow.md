---
title: Plataforma de identidad de Microsoft de uso para tener acceso a recursos seguros sin interacción del usuario | Azure
description: Crear aplicaciones web mediante la implementación de plataforma de identidad de Microsoft del protocolo de autenticación OAuth 2.0.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 9b7cfbd7-f89f-4e33-aff2-414edd584b07
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
ms.openlocfilehash: 485f79f79c52067e89fa0a606e76a533c312fb84
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545089"
---
# <a name="microsoft-identity-platform-and-the-oauth-20-client-credentials-flow"></a>Plataforma de identidad de Microsoft y el flujo de credenciales de cliente de OAuth 2.0

[!INCLUDE [active-directory-develop-applies-v2](../../../includes/active-directory-develop-applies-v2.md)]

Puede usar la [concesión de credenciales de cliente de OAuth 2.0](https://tools.ietf.org/html/rfc6749#section-4.4) especificada en RFC 6749 y a veces denominada *OAuth de dos días*, para acceder a recursos hospedados en la web mediante la identidad de una aplicación. Este tipo de concesión se usa principalmente para las interacciones entre servidores que se deben ejecutar en segundo plano, sin la interacción inmediata con un usuario. Estos tipos de aplicaciones suelen denominarse *demonios* o *cuentas de servicio*.

El flujo de concesión de credenciales de cliente de OAuth 2.0 permite que un servicio web (cliente confidencial) use sus propias credenciales para autenticarse al llamar a otro servicio web, en lugar de suplantar a un usuario. En este escenario, el cliente es normalmente un servicio web de nivel intermedio, un servicio demonio o un sitio web. Para conseguir un mayor nivel de control, la plataforma de identidad de Microsoft también permite que el servicio que realiza la llamada use un certificado (en lugar de un secreto compartido) como credencial.

> [!NOTE]
> El punto de conexión de plataforma de identidad de Microsoft no es compatible con todas las características y escenarios de Azure AD. Para determinar si debe utilizar el punto de conexión de plataforma de identidad de Microsoft, obtenga información sobre [limitaciones de la plataforma de identidad de Microsoft](active-directory-v2-limitations.md).

En el escenario más típico de *OAuth de tres vías*, una aplicación cliente tiene permiso para obtener acceso a un recurso en nombre de un usuario específico. El permiso se delega desde el usuario a la aplicación, habitualmente durante el proceso de [consentimiento](v2-permissions-and-consent.md). Sin embargo, en el flujo de credenciales de cliente (*OAuth con dos patas*), los permisos se conceden directamente en la propia aplicación. Cuando la aplicación presenta un token a un recurso, este exige que sea la propia aplicación la que tenga autorización para realizar una acción, no el usuario.

## <a name="protocol-diagram"></a>Diagrama de protocolo

El flujo completo de credenciales de cliente tiene un aspecto similar al diagrama siguiente. Más adelante en este artículo se describe cada uno de los pasos.

![Flujo de credenciales de cliente](./media/v2-oauth2-client-creds-grant-flow/convergence-scenarios-client-creds.svg)

## <a name="get-direct-authorization"></a>Obtención de autorización directa

Una aplicación recibe habitualmente autorización directa para acceder a un recurso de una de dos maneras:

* [Mediante una lista de control de acceso (ACL) en el recurso](#access-control-lists)
* [Mediante la asignación de permisos de aplicación en Azure AD](#application-permissions)

Estos dos métodos son los más comunes en Azure AD y se recomiendan para los clientes y recursos que realizan el flujo de credenciales de cliente. Un recurso puede elegir autorizar a sus clientes de otras formas. Cada servidor de recurso puede elegir el mejor método para su aplicación.

### <a name="access-control-lists"></a>Listas de control de acceso

Un proveedor de recursos podría exigir una comprobación de autorización basada en una lista de identificadores de aplicación (cliente) que conoce, y concede un nivel de acceso específico. Cuando el recurso recibe un token desde el punto de conexión de plataforma de identidad de Microsoft, puede descodificar el token y extraer el identificador de aplicación del cliente desde el `appid` y `iss` notificaciones. Luego, compara la aplicación con una lista de control de acceso (ACL) que mantiene. El método y la granularidad de la ACL podrían variar considerablemente entre los recursos.

Un caso de uso común es utilizar una ACL para ejecutar pruebas para una aplicación web o para una API web. La API web podría conceder solo un subconjunto de permisos completos a un cliente específico. Para ejecutar pruebas de extremo a extremo en la API, cree a un cliente de prueba que adquiere tokens desde el punto de conexión de plataforma de identidad de Microsoft y, a continuación, los envía a la API. Luego, la API comprueba la ACL del identificador de la aplicación del cliente de prueba para tener acceso completo a toda la funcionalidad de la API. Si usa este tipo de ACL, asegúrese de validar no solo el valor `appid` de la persona que llama, sino también que el valor de `iss` del token sea de confianza.

Este tipo de autorización es común para las cuentas de servicio y los demonios que necesitan tener acceso a datos que pertenecen a los usuarios consumidores con cuentas personales de Microsoft. En el caso de los datos que pertenecen a organizaciones, se recomienda obtener la autorización necesaria a través de los permisos de aplicación.

### <a name="application-permissions"></a>Permisos de aplicación

En lugar de usar ACL, puede usar las API para exponer un conjunto de permisos de aplicación. El administrador de una organización concede un permiso de aplicación a una aplicación, el que solo se puede usar para obtener acceso a los datos que pertenecen a esa organización y sus empleados. Por ejemplo, Microsoft Graph expone varios permisos de aplicación para hacer lo siguiente:

* Leer correo en todos los buzones de correo
* Leer y escribir correo en todos los buzones de correo
* Enviar correo como cualquier usuario
* Leer datos de directorio

Para más información sobre los permisos de aplicación, vaya a [Microsoft Graph](https://developer.microsoft.com/graph).

Para usar permisos de aplicación en la aplicación, siga los pasos que se describen en las secciones siguientes.

#### <a name="request-the-permissions-in-the-app-registration-portal"></a>Solicitud de los permisos en el portal de registro de aplicaciones

1. Registrar y crear una aplicación a través del nuevo [registros de aplicaciones (versión preliminar) experimentan](quickstart-register-app.md).
2. Vaya a la aplicación en la experiencia de registros (versión preliminar) de la aplicación. Navegue hasta la **certificados y secretos** sección y agregue un **nuevo secreto de cliente**, ya que lo necesitará al menos un secreto de cliente para solicitar un token.
3. Busque la sección de **permisos de API** y agregue los **permisos de aplicación**  que esta requiere.
4. **Guarde** el registro de aplicaciones.

#### <a name="recommended-sign-the-user-into-your-app"></a>Se recomienda: Iniciar la sesión del usuario en la aplicación

Habitualmente, cuando compila una aplicación que usa permisos de aplicación, la aplicación requiere una página o vista en la que el administrador aprueba los permisos de la aplicación. Esta página puede ser parte del flujo de inicio de sesión de la aplicación o de la configuración de la aplicación, o bien puede ser un flujo de "conexión" dedicado. En muchos casos, tiene sentido que la aplicación muestre esta vista de conexión solo después de que un usuario haya iniciado sesión con una cuenta Microsoft profesional o educativa.

Si el usuario inicia sesión en su aplicación, puede identificar la organización a la que pertenece el usuario antes de que pregunte al usuario que apruebe los permisos de aplicación. Aunque no es estrictamente necesario, puede ayudarlo a crear una experiencia más intuitiva para los usuarios. Para iniciar la sesión del usuario, siga nuestro [tutoriales del protocolo de plataforma de identidad de Microsoft](active-directory-v2-protocols.md).

#### <a name="request-the-permissions-from-a-directory-admin"></a>Solicitud de los permisos de un administrador de directorios

Cuando esté listo para solicitar permisos de administrador de su organización, puede redirigir al usuario a la plataforma Microsoft identity *punto de conexión de consentimiento de administrador*.

> [!TIP]
> Pruebe a ejecutar esta solicitud en Postman (Utilizar su propio identificador de aplicación para obtener mejores resultados: la aplicación del tutorial no solicita permisos útiles). [![Ejecución en Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)

```
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/adminconsent?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&state=12345
&redirect_uri=http://localhost/myapp/permissions
```

```
// Pro tip: Try pasting the following request in a browser.
```

```
https://login.microsoftonline.com/common/adminconsent?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&state=12345&redirect_uri=http://localhost/myapp/permissions
```

| Parámetro | Condición | DESCRIPCIÓN |
| --- | --- | --- |
| `tenant` | Obligatorio | El inquilino de directorio al que quiere solicitar permiso. Puede estar en formato de nombre descriptivo o GUID. Si no sabe a qué inquilino pertenece el usuario y desea permitirle iniciar sesión con cualquier inquilino, use `common`. |
| `client_id` | Obligatorio | El **Id. de aplicación (cliente)** que la [Azure portal: registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) experiencia asignado a la aplicación. |
| `redirect_uri` | Obligatorio | El URI de redireccionamiento adonde desea que se envíe la respuesta para que la controle la aplicación. Debe coincidir exactamente con uno de los URI de redireccionamiento que registró en el portal, con la excepción de que debe estar codificado como dirección URL y puede tener segmentos de trazado adicionales. |
| `state` | Recomendada | Un valor incluido en la solicitud que también se devolverá en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. El estado se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, por ejemplo, la página o vista en la que estaban. |

En este momento, Azure AD exige solo un administrador de inquilinos puede firmar en completar la solicitud. Se pedirá al administrador que apruebe todos los permisos de aplicación directos que solicitó para la aplicación en el portal de registro de aplicaciones.

##### <a name="successful-response"></a>Respuesta correcta

Si el administrador aprueba los permisos para la aplicación, la respuesta correcta tendrá un aspecto similar al siguiente:

```
GET http://localhost/myapp/permissions?tenant=a8990e1f-ff32-408a-9f8e-78d3b9139b95&state=state=12345&admin_consent=True
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| `tenant` | El inquilino de directorio que concedió los permisos solicitados a la aplicación, en formato GUID. |
| `state` | Un valor incluido en la solicitud que también se devuelve en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. El estado se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, por ejemplo, la página o vista en la que estaban. |
| `admin_consent` | Se establece en **True**. |

##### <a name="error-response"></a>Respuesta de error

Si el administrador no aprueba los permisos de la aplicación, la respuesta de error tendrá el aspecto siguiente:

```
GET http://localhost/myapp/permissions?error=permission_denied&error_description=The+admin+canceled+the+request
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| `error` | Una cadena de código de error que puede usar para clasificar los tipos de errores y que puede usar para reaccionar ante los errores. |
| `error_description` | Un mensaje de error específico que puede ayudarle a identificar la causa raíz de un error. |

Una vez que reciba una respuesta correcta desde el punto de conexión de aprovisionamiento de la aplicación, esta habrá obtenido los permisos de aplicación directos que solicitó. Ahora puede solicitar un token para el recurso que desee.

## <a name="get-a-token"></a>Obtención de un token

Una vez que obtenga la autorización necesaria para la aplicación, siga con el proceso de adquisición de tokens de acceso para las API. Para obtener un token mediante el cliente de concesión de credenciales, envíe una solicitud POST a la `/token` extremos de plataforma de identidad de Microsoft:

> [!TIP]
> Pruebe a ejecutar esta solicitud en Postman (Utilizar su propio identificador de aplicación para obtener mejores resultados: la aplicación del tutorial no solicita permisos útiles). [![Ejecución en Postman](./media/v2-oauth2-auth-code-flow/runInPostman.png)](https://app.getpostman.com/run-collection/f77994d794bab767596d)

### <a name="first-case-access-token-request-with-a-shared-secret"></a>Primer caso: solicitud de token de acceso con un secreto compartido

```
POST /{tenant}/oauth2/v2.0/token HTTP/1.1           //Line breaks for clarity
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=535fb089-9ff3-47b6-9bfb-4f1264799865
&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_secret=qWgdYAmab0YSkuL1qKv5bPX
&grant_type=client_credentials
```

```
curl -X POST -H "Content-Type: application/x-www-form-urlencoded" -d 'client_id=535fb089-9ff3-47b6-9bfb-4f1264799865&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default&client_secret=qWgdYAmab0YSkuL1qKv5bPX&grant_type=client_credentials' 'https://login.microsoftonline.com/common/oauth2/v2.0/token'
```

| Parámetro | Condición | DESCRIPCIÓN |
| --- | --- | --- |
| `tenant` | Obligatorio | El inquilino del directorio en el que va a funcionar la aplicación, con el formato de GUID o de nombre de dominio. |
| `client_id` | Obligatorio | El identificador de la aplicación que está asignado a la aplicación. Puede encontrar esta información en el portal donde registró la aplicación. |
| `scope` | Obligatorio | El valor pasado del parámetro `scope` en esta solicitud debe ser el identificador de recurso (URI de identificador de aplicación) del recurso que desea, con el sufijo `.default`. Para el ejemplo de Microsoft Graph, el valor es `https://graph.microsoft.com/.default`. <br/>Este valor indica el punto de conexión de plataforma de identidad de Microsoft de todos los permisos de aplicación directos que ha configurado para la aplicación, el punto de conexión debe emitir un token para los que están asociados con el recurso que desee utilizar. Para más información sobre el ámbito `/.default`, consulte la [documentación de consent](v2-permissions-and-consent.md#the-default-scope). |
| `client_secret` | Obligatorio | El secreto de cliente que ha generado para la aplicación en el portal de registro de aplicación. El secreto de cliente debe codificarse como dirección URL antes de enviarse. |
| `grant_type` | Obligatorio | Se debe establecer en `client_credentials`. |

### <a name="second-case-access-token-request-with-a-certificate"></a>Segundo caso: solicitud de token de acceso con un certificado

```
POST /{tenant}/oauth2/v2.0/token HTTP/1.1               // Line breaks for clarity
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_id=97e0a5b7-d745-40b6-94fe-5f77d35c6e05
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&grant_type=client_credentials
```

| Parámetro | Condición | DESCRIPCIÓN |
| --- | --- | --- |
| `tenant` | Obligatorio | El inquilino del directorio en el que va a funcionar la aplicación, con el formato de GUID o de nombre de dominio. |
| `client_id` | Obligatorio |El identificador de aplicación (cliente) que se asigna a la aplicación. |
| `scope` | Obligatorio | El valor pasado del parámetro `scope` en esta solicitud debe ser el identificador de recurso (URI de identificador de aplicación) del recurso que desea, con el sufijo `.default`. Para el ejemplo de Microsoft Graph, el valor es `https://graph.microsoft.com/.default`. <br/>Este valor indica el punto de conexión de plataforma de identidad de Microsoft que de todos los permisos de aplicación directos que ha configurado para la aplicación, debe emitir un token para los que están asociados con el recurso que desee utilizar. Para más información sobre el ámbito `/.default`, consulte la [documentación de consent](v2-permissions-and-consent.md#the-default-scope). |
| `client_assertion_type` | Obligatorio | El valor se debe establecer en `urn:ietf:params:oauth:client-assertion-type:jwt-bearer`. |
| `client_assertion` | Obligatorio | Una aserción (JSON Web Token) que debe crear y firmar con el certificado que ha registrado como credenciales de la aplicación. Lea el artículo sobre las [credenciales de certificado](active-directory-certificate-credentials.md) para información sobre cómo registrar el certificado y el formato de la aserción.|
| `grant_type` | Obligatorio | Se debe establecer en `client_credentials`. |

Tenga en cuenta que los parámetros son casi iguales que en el caso de solicitud con un secreto compartido, salvo que el parámetro client_secret se sustituye por dos parámetros: client_assertion_type y client_assertion.

### <a name="successful-response"></a>Respuesta correcta

Una respuesta correcta tiene el siguiente aspecto:

```
{
  "token_type": "Bearer",
  "expires_in": 3599,
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBP..."
}
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| `access_token` | El token de acceso solicitado. La aplicación puede usar este token para autenticarse en el recurso protegido, como una API web. |
| `token_type` | Indica el valor de tipo de token. El único tipo que Microsoft admite de plataforma de identidad es `bearer`. |
| `expires_in` | La cantidad de tiempo que un token de acceso es válido (en segundos). |

### <a name="error-response"></a>Respuesta de error

Una respuesta de error tiene el aspecto siguiente:

```
{
  "error": "invalid_scope",
  "error_description": "AADSTS70011: The provided value for the input parameter 'scope' is not valid. The scope https://foo.microsoft.com/.default is not valid.\r\nTrace ID: 255d1aef-8c98-452f-ac51-23d051240864\r\nCorrelation ID: fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7\r\nTimestamp: 2016-01-09 02:02:12Z",
  "error_codes": [
    70011
  ],
  "timestamp": "2016-01-09 02:02:12Z",
  "trace_id": "255d1aef-8c98-452f-ac51-23d051240864",
  "correlation_id": "fb3d2015-bc17-4bb9-bb85-30c5cf1aaaa7"
}
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| `error` | Una cadena de código de error que puede usar para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que podría ayudarlo a identificar la causa raíz de un error de autenticación. |
| `error_codes` | Una lista de los códigos de error específicos de STS que podrían ayudar en los diagnósticos. |
| `timestamp` | La hora a la que se produjo el error. |
| `trace_id` | Un identificador único de la solicitud para ayudar con los diagnósticos. |
| `correlation_id` | Un identificador único de la solicitud para ayudar con los diagnósticos entre componentes. |

> [!NOTE]
> Puede actualizar el archivo de manifiesto de la aplicación del portal de azure para su aplicación poder recibir el token de v2. Puede agregar el atributo `accessTokenAcceptedVersion` y establezca el valor en 2 como `"accessTokenAcceptedVersion": 2`. Consulte el artículo [manifiesto de aplicación](https://docs.microsoft.com/azure/active-directory/develop/reference-app-manifest#manifest-reference) para más información sobre el mismo. De forma predeterminada, la aplicación actualmente recibe un token de v1. Si esto no se define dentro del manifiesto de aplicación o Web API, que el valor de este atributo en el manifiesto predeterminado es 1 y, por tanto, la aplicación recibe el token de v1.  


## <a name="use-a-token"></a>Uso de un token

Ahora que adquirió un token, úselo para hacer solicitudes al recurso. Cuando expire el token, repita la solicitud al punto de conexión `/token` para adquirir un token de acceso nuevo.

```
GET /v1.0/me/messages
Host: https://graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q...
```

```
// Pro tip: Try the following command! (Replace the token with your own.)
```

```
curl -X GET -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik5HVEZ2ZEstZnl0aEV1Q" 'https://graph.microsoft.com/v1.0/me/messages'
```

## <a name="code-samples-and-other-documentation"></a>Ejemplos de código y otra documentación

Lea la [documentación en la que se describen de forma general las credenciales del cliente](https://aka.ms/msal-net-client-credentials) en la Biblioteca de autenticación de Microsoft

| Muestra | Plataforma |DESCRIPCIÓN |
|--------|----------|------------|
|[active-directory-dotnetcore-daemon-v2](https://github.com/Azure-Samples/active-directory-dotnetcore-daemon-v2) | Consola de .NET Core 2.1 | Una aplicación de .NET Core sencilla que muestra los usuarios de un inquilino que consultan Microsoft Graph mediante la identidad de la aplicación, en lugar hacerlo en nombre de un usuario. El ejemplo también muestra la variación del uso de certificados para la autenticación. |
|[active-directory-dotnet-daemon-v2](https://github.com/Azure-Samples/active-directory-dotnet-daemon-v2)|ASP.NET MVC | Una aplicación web que sincroniza datos de Microsoft Graph mediante la identidad de la aplicación, en lugar hacerlo en nombre de un usuario. |
