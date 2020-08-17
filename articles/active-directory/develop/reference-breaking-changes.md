---
title: Referencia de cambios importantes en Azure Active Directory
description: Obtenga información sobre los cambios realizados en los protocolos de Azure AD que pueden afectar a la aplicación.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: reference
ms.date: 5/4/2020
ms.author: ryanwi
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 0c5abf345fda9db4cc5123360245e42ea0ef40e1
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88115040"
---
# <a name="whats-new-for-authentication"></a>Novedades en la autenticación

> Para recibir notificaciones de las actualizaciones de esta página, pegue esta dirección URL en el lector de fuentes RSS:<br/>`https://docs.microsoft.com/api/search/rss?search=%22whats%20new%20for%20authentication%22&locale=en-us`

El sistema de autenticación altera y agrega características constantemente para aumentar la seguridad y mejorar el cumplimiento de normas. Para mantenerse al día con los avances más recientes, este artículo proporciona información acerca de los detalles siguientes:

- Características más recientes
- Problemas conocidos
- Cambios en el protocolo
- Funciones obsoletas

> [!TIP]
> Esta página se actualiza regularmente, por lo que se recomienda visitarla a menudo. Salvo que se indique lo contrario, estos cambios solo entrarán en vigor en las aplicaciones recién registradas.

## <a name="upcoming-changes"></a>Próximos cambios

No hay ninguno programado en este momento.  Consulte a continuación los cambios que están o estarán próximamente en producción.

## <a name="may-2020"></a>Mayo de 2020

### <a name="azure-government-endpoints-are-changing"></a>Los puntos de conexión de Azure Government están cambiando

**Fecha efectiva**: 5 de mayo (finaliza en junio de 2020) 

**Puntos de conexión afectados**: All

**Protocolo afectado**: Todos los flujos

El 1 de junio de 2018, la autoridad oficial de Azure Active Directory (AAD) para Azure Government cambió de `https://login-us.microsoftonline.com` a `https://login.microsoftonline.us`. Este cambio también se aplica a Microsoft 365 GCC High y DoD, a los que también presta servicio Azure Government AAD. Si es propietario de una aplicación en un inquilino de la Administración Pública de EE. UU., debe actualizar la aplicación para que la sesión de los usuarios se inicie en el punto de conexión `.us`.  

A partir del 5 de mayo, Azure AD comenzará a aplicar el cambio del punto de conexión, lo que impide que los usuarios gubernamentales inicien sesión en aplicaciones hospedadas en inquilinos de la Administración Pública de EE. UU. mediante el punto de conexión público (`microsoftonline.com`).  Las aplicaciones afectadas comenzarán a experimentar un error `AADSTS900439` - `USGClientNotSupportedOnPublicEndpoint`. Este error indica que la aplicación está intentando iniciar la sesión de un usuario de la Administración Pública de EE. UU. en el punto de conexión de nube pública. Si su aplicación se encuentra en un inquilino en la nube pública y tiene previsto prestar servicio a usuarios de la Administración Pública de EE. UU., deberá [actualizar la aplicación para que los admita de forma explícita](./authentication-national-cloud.md). Esto puede requerir la creación de un nuevo registro de aplicaciones en la nube de la Administración Pública de EE. UU. 

La aplicación de este cambio se realizará mediante un lanzamiento gradual basado en la frecuencia con que los usuarios de la nube de la Administración Pública de EE. UU. inician sesión en la aplicación: se aplicará antes a las aplicaciones que inician sesión de usuarios de la Administración Pública de EE. UU. con poca frecuencia, mientras que las aplicaciones que los usuarios de la Administración Pública de EE. UU. usan con frecuencia serán las últimas a las que se aplicará. Esperamos que el cambio se haya completado en todas las aplicaciones en junio de 2020. 

Para obtener más información, consulte la [entrada de blog de Azure Government sobre esta migración](https://devblogs.microsoft.com/azuregov/azure-government-aad-authority-endpoint-update/). 

## <a name="march-2020"></a>Marzo de 2020

### <a name="user-passwords-will-be-restricted-to-256-characters"></a>Las contraseñas de usuario tendrán un límite de 256 caracteres.

**Fecha efectiva**: 13 de marzo de 2020

**Puntos de conexión afectados**: All

**Protocolo afectado**: Todos los flujos de usuario.

Los usuarios con contraseñas de más de 256 caracteres que inicien sesión directamente en Azure AD (en lugar de en un IDP federado como ADFS) no podrán iniciar sesión a partir del 13 de marzo de 2020 y se les pedirá que restablezcan su contraseña.  Es posibles que los administradores reciban solicitudes de ayuda para restablecer la contraseña de los usuarios.

El error en los registros de inicio de sesión será AADSTS 50052: InvalidPasswordExceedsMaxLength

Mensaje: `The password entered exceeds the maximum length of 256. Please reach out to your admin to reset the password.`

Corrección:

El usuario no puede iniciar sesión porque su contraseña supera la longitud máxima permitida. Debe ponerse en contacto con su administrador para restablecer la contraseña. Si SSPR está habilitado en su inquilino, puede restablecer la contraseña siguiendo el vínculo "¿Ha olvidado la contraseña?".



## <a name="february-2020"></a>Febrero de 2020

### <a name="empty-fragments-will-be-appended-to-every-http-redirect-from-the-login-endpoint"></a>Los fragmentos vacíos se anexarán a cada redirección HTTP desde el punto de conexión de inicio de sesión.

**Fecha efectiva**: 8 de febrero de 2020

**Puntos de conexión afectados**: v1.0 y v2.0

**Protocolo afectado**: Flujos de OAuth y OIDC que usan response_type=query: incluye el [flujo de código de autorización](v2-oauth2-auth-code-flow.md) en algunos casos y el [flujo implícito](v2-oauth2-implicit-grant-flow.md).

Cuando se envía una respuesta de autenticación desde login.microsoftonline.com a una aplicación a través de la redirección HTTP, el servicio anexará un fragmento vacío a la dirección URL de respuesta.  Esto evita una clase de ataques de redireccionamiento, ya que se asegura de que el explorador borre todo fragmento existente en la solicitud de autenticación.  Ninguna aplicación debe tener una dependencia de este comportamiento.


## <a name="august-2019"></a>Agosto de 2019

### <a name="post-form-semantics-will-be-enforced-more-strictly---spaces-and-quotes-will-be-ignored"></a>La semántica del formulario POST se aplicará más estrictamente y se omitirán los espacios y comillas.

**Fecha efectiva**: 2 de septiembre de 2019

**Puntos de conexión afectados**: v1.0 y v2.0

**Protocolo afectado**: Se usa POST en cualquier lugar([credenciales de cliente](./v2-oauth2-client-creds-grant-flow.md), [canje de código de autorización](./v2-oauth2-auth-code-flow.md), [ROPC](./v2-oauth-ropc.md), [OBO](./v2-oauth2-on-behalf-of-flow.md) y [canje de token de actualización](./v2-oauth2-auth-code-flow.md#refresh-the-access-token)).

A partir de la semana 9/2, las solicitudes de autenticación que usan el método POST se validarán con estándares HTTP más estrictos.  Concretamente, los espacios y las comillas dobles (") ya no se quitarán de los valores del formulario de solicitud. No se espera que estos cambios interrumpan ningún cliente existente y se asegurará de que las solicitudes enviadas a Azure AD se controlan de forma confiable cada vez. En el futuro (consulte más arriba), tenemos previsto rechazar además los parámetros duplicados y omitir la marca BOM dentro de las solicitudes.

Ejemplo:

En la actualidad, `?e=    "f"&g=h` se analiza exactamente igual que `?e=f&g=h`, por tanto `e` == `f`.  Con este cambio, ahora se analizaría para que `e` == `    "f"`; esto es improbable que sea un argumento válido y la solicitud no se realizará correctamente.


## <a name="july-2019"></a>Julio de 2019

### <a name="app-only-tokens-for-single-tenant-applications-are-only-issued-if-the-client-app-exists-in-the-resource-tenant"></a>Los tokens solo de aplicación para aplicaciones de inquilino único solo se emiten si la aplicación cliente existe en el inquilino de recursos.

**Fecha efectiva**: 26 de julio de 2019

**Puntos de conexión afectados**: [v1.0](../azuread-dev/v1-oauth2-client-creds-grant-flow.md) y [v2.0](./v2-oauth2-client-creds-grant-flow.md)

**Protocolo afectado**: [credenciales de cliente (tokens de solo aplicación)](../azuread-dev/v1-oauth2-client-creds-grant-flow.md)

El 26 de julio se aplicó un cambio de seguridad que modifica la manera en la que se emiten los tokens de solo aplicación (a través de la concesión de credenciales de cliente). Anteriormente, las aplicaciones podían obtener tokens para llamar a cualquier otra aplicación, sin tener en cuenta su presencia en el inquilino o los roles con consentimiento para esa aplicación.  Este comportamiento se ha actualizado de modo que, en el caso de los recursos (a veces denominados API web) configurados para ser inquilino único (el valor predeterminado), la aplicación cliente debe existir en el inquilino de recursos.  Tenga en cuenta que aún no se requiere que exista consentimiento entre el cliente y la API, y que las aplicaciones todavía deben realizar sus propias comprobaciones de autorización para asegurarse de que existe una notificación `roles` y de que contiene el valor esperado para la API.

El mensaje de error de este escenario indica actualmente:

`The service principal named <appName> was not found in the tenant named <tenant_name>. This can happen if the application has not been installed by the administrator of the tenant.`

Para solucionar este problema, use la experiencia de consentimiento del administrador con el fin de crear la entidad de servicio de la aplicación cliente en el inquilino o bien créela manualmente.  Este requisito garantiza que el inquilino ha dado permiso a la aplicación para que opere dentro del inquilino.

#### <a name="example-request"></a>Solicitud de ejemplo

`https://login.microsoftonline.com/contoso.com/oauth2/authorize?resource=https://gateway.contoso.com/api&response_type=token&client_id=14c88eee-b3e2-4bb0-9233-f5e3053b3a28&...` En este ejemplo, el inquilino de recursos (autoridad) es contoso.com, la aplicación de recursos es una aplicación de un solo inquilino denominada `gateway.contoso.com/api` para el inquilino de Contoso y la aplicación cliente es `14c88eee-b3e2-4bb0-9233-f5e3053b3a28`.  Si la aplicación cliente tiene una entidad de servicio en Contoso.com, esta solicitud puede continuar.  Si no la tiene, se producirá el error anterior en la solicitud.

Si la aplicación de puerta de enlace de Contoso fuera una aplicación de varios inquilinos, la solicitud continuaría, independientemente de que la aplicación cliente tuviera o no una entidad de servicio en Contoso.com.

### <a name="redirect-uris-can-now-contain-query-string-parameters"></a>Los identificadores URI de redireccionamiento ahora pueden contener parámetros de cadena de consulta.

**Fecha efectiva**: 22 de julio de 2019

**Puntos de conexión afectados**: v1.0 y v2.0

**Protocolo afectado**: Todos los flujos

Según la solicitud [RFC 6749](https://tools.ietf.org/html/rfc6749#section-3.1.2), las aplicaciones de Azure AD ahora pueden registrar y usar identificadores URI de redireccionamiento (respuesta) con parámetros de consulta estáticos (como `https://contoso.com/oauth2?idp=microsoft`) para solicitudes de OAuth 2.0.  Los identificadores URI de redireccionamiento dinámico siguen estando prohibidos, ya que representan un riesgo de seguridad, y no se pueden usar para conservar la información de estado a través de una solicitud de autenticación; para ello, use el parámetro `state`.

El parámetro de consulta estático está sujeto a la coincidencia de cadenas para los identificadores URI de redireccionamiento, al igual que cualquier otra parte del URI de redireccionamiento; si no hay ninguna cadena registrada que coincida con el parámetro redirect_uri descodificado con URI, se rechazará la solicitud.  Si se encuentra el identificador URI en el registro de la aplicación, se usará toda la cadena para redirigir al usuario, incluido el parámetro de consulta estático.

Tenga en cuenta que en este momento (final de julio de 2019), la experiencia de usuario del registro de aplicaciones en Azure Portal sigue bloqueando los parámetros de consulta.  Sin embargo, puede editar el manifiesto de aplicación manualmente para agregar parámetros de consulta y probar este procedimiento en la aplicación.


## <a name="march-2019"></a>Marzo de 2019

### <a name="looping-clients-will-be-interrupted"></a>Se interrumpirán los clientes en bucle

**Fecha efectiva**: 25 de marzo de 2019

**Puntos de conexión afectados**: v1.0 y v2.0

**Protocolo afectado**: Todos los flujos

En ocasiones, las aplicaciones cliente pueden comportarse de manera incorrecta, emitiendo cientos de veces la misma solicitud de inicio de sesión durante un breve período de tiempo.  Estas solicitudes pueden completarse satisfactoriamente o no, pero todas contribuyen a una deficiente experiencia para el usuario y a un incremento de las cargas de trabajo para el IDP, lo que aumenta la latencia para todos los usuarios y reduce la disponibilidad del IDP.  Estas aplicaciones operan fuera de los límites de uso normal y deben actualizarse para que se comporten correctamente.

A los clientes que emitan solicitudes duplicadas varias veces se les enviará un error `invalid_grant`: `AADSTS50196: The server terminated an operation because it encountered a loop while processing a request`.

La mayoría de los clientes no tendrán que cambiar su comportamiento para evitar este error.  Solo los clientes incorrectamente configurados (aquellos sin almacenamiento en caché de tokens o los que ya presentan bucles de mensajes) se verán afectados por este error.  Se realiza un seguimiento de los clientes por instancia a nivel local (mediante cookie) sobre los siguientes factores:

* Sugerencia de usuario, si existe

* Ámbitos o recursos solicitados

* Id. de cliente

* URI de redireccionamiento

* Tipo y modo de respuesta

Las aplicaciones que realizan varias solicitudes (más de 15) en un breve período de tiempo (5 minutos) recibirán un error `invalid_grant` que explica que están en bucle.  Los tokens solicitados tienen una vigencia de una duración suficiente (10 minutos como mínimo, 60 minutos de forma predeterminada), por lo que las solicitudes repetidas durante este período de tiempo son innecesarias.

Todas las aplicaciones deben controlar `invalid_grant` mostrando un aviso interactivo, en lugar de solicitar un token de forma silenciosa.  Para evitar este error, los clientes deberían asegurarse de que almacenan en caché correctamente los tokens que reciben.


## <a name="october-2018"></a>Octubre de 2018

### <a name="authorization-codes-can-no-longer-be-reused"></a>No se podrán reutilizar los códigos de autorización

**Fecha efectiva**: 15 de noviembre de 2018

**Puntos de conexión afectados**: v1.0 y v2.0

**Protocolo afectado**: [flujo de código](v2-oauth2-auth-code-flow.md)

A partir del 15 de noviembre de 2018, Azure AD dejará de aceptar los códigos de autenticación que se usaban anteriormente para las aplicaciones. Este cambio de seguridad ayuda a poner Azure AD en consonancia con la especificación de OAuth y se aplicará en los puntos de conexión v1 y v2.

Si la aplicación reutiliza códigos de autorización para obtener tokens para varios recursos, es recomendable que use el código para obtener un token de actualización y, a continuación, utilice este para adquirir tokens adicionales para otros recursos. Los códigos de autorización solo se pueden usar una vez, pero los tokens de actualización se pueden usar varias veces en varios recursos. Cualquier nueva aplicación que intente reutilizar un código de autenticación durante el flujo de código de OAuth obtendrá el error invalid_grant.

Para más información acerca de los tokens de actualización, consulte [Actualización de los tokens de acceso](v2-oauth2-auth-code-flow.md#refresh-the-access-token).  Si usa ADAL o MSAL, la biblioteca lo controla automáticamente. Sustituya la segunda instancia de 'AcquireTokenByAuthorizationCodeAsync' por 'AcquireTokenSilentAsync'.

## <a name="may-2018"></a>Mayo de 2018

### <a name="id-tokens-cannot-be-used-for-the-obo-flow"></a>No se puede usar los tokens de identificador para el flujo de OBO

**Fecha**: 1 de mayo de 2018

**Puntos de conexión afectados**: v1.0 y v2.0

**Protocolos afectados**: flujo implícito y [flujo con derechos delegados](v2-oauth2-on-behalf-of-flow.md)

A partir del 1 de mayo de 2018, id_tokens no se puede utilizar como instrucción de aserción en un flujo de OBO en las aplicaciones nuevas. En su lugar deben usarse tokens de acceso para proteger las API, incluso entre un cliente y el nivel intermedio de la misma aplicación. Las aplicaciones registradas antes del 1 de mayo de 2018 seguirán funcionando y podrán intercambiar id_tokens por un token de acceso (pero tenga en cuenta que este patrón no se considera un procedimiento recomendado).

Para ofrecer una solución alternativa al problema que pueda provocar este cambio, puede hacer lo siguiente:

1. Cree una API web para la aplicación con uno o más ámbitos. Este punto de entrada explícito le permitirá mayor control y seguridad.
1. En el manifiesto de la aplicación, [Azure Portal](https://portal.azure.com) o el [portal de registro de aplicaciones](https://apps.dev.microsoft.com), asegúrese de que la aplicación tiene permiso para emitir tokens de acceso mediante el flujo implícito, lo que se controla mediante la clave `oauth2AllowImplicitFlow`.
1. Cuando la aplicación cliente solicita un valor de id_token a través de `response_type=id_token`, también solicita un token de acceso (`response_type=token`) para la API web creada anteriormente. Por consiguiente, cuando se usa el punto de conexión v2.0 el parámetro `scope` debe ser similar a `api://GUID/SCOPE`. En el punto de conexión v1.0, el parámetro `resource` debe ser el identificador URI de la aplicación de la API web.
1. Pase este token de acceso al nivel intermedio en lugar de id_token.