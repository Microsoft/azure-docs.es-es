---
title: Ámbitos de plataforma de identidad de Microsoft, permisos y consentimiento | Microsoft Docs
description: Descripción de la autorización en el extremo de plataforma de identidad de Microsoft, incluidos los ámbitos, permisos y consentimiento.
services: active-directory
documentationcenter: ''
author: rwike77
manager: CelesteDG
editor: ''
ms.assetid: 8f98cbf0-a71d-4e34-babf-e644ad9ff423
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 04/12/2019
ms.author: ryanwi
ms.reviewer: hirsin, jesakowi, jmprieur
ms.custom: fasttrack-edit
ms.collection: M365-identity-device-management
ms.openlocfilehash: 717607de6d9d193a7373637d0d0fcd879b54fed0
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544882"
---
# <a name="permissions-and-consent-in-the-microsoft-identity-platform-endpoint"></a>Permisos y consentimiento en el punto de conexión de plataforma de identidad de Microsoft

[!INCLUDE [active-directory-develop-applies-v2](../../../includes/active-directory-develop-applies-v2.md)]

Las aplicaciones que se integran con la plataforma de identidad de Microsoft siguen un modelo de autorización que permite a los usuarios y los administradores controlar el modo en que se puede acceder a los datos. La implementación del modelo de autorización se ha actualizado en el punto de conexión de plataforma de identidad de Microsoft, y cambia de cómo una aplicación debe interactuar con la plataforma Microsoft identity. En este artículo se tratan los conceptos básicos de este modelo de autorización, incluidos los ámbitos, los permisos y el consentimiento.

> [!NOTE]
> El punto de conexión de plataforma de identidad de Microsoft no admite todas las características y escenarios. Para determinar si debe utilizar el punto de conexión de plataforma de identidad de Microsoft, obtenga información sobre [limitaciones de la plataforma de identidad de Microsoft](active-directory-v2-limitations.md).

## <a name="scopes-and-permissions"></a>Permisos y ámbitos

La plataforma de identidad de Microsoft implementa el protocolo de autorización [OAuth 2.0](active-directory-v2-protocols.md). OAuth 2.0 es un método a través del cual una aplicación de terceros puede acceder a recursos hospedados en Web en nombre de un usuario. Cualquier recurso hospedado en web que se integre con la plataforma de identidad de Microsoft tiene un identificador de recursos o *URI de identificador de aplicación*. Por ejemplo, algunos de los recursos de Microsoft hospedados en la web incluyen:

* Microsoft Graph: `https://graph.microsoft.com`
* Office 365 Mail API: `https://outlook.office.com`
* Azure AD Graph: `https://graph.windows.net`

> [!NOTE]
> Se recomienda encarecidamente que utilice Microsoft Graph en lugar de Azure AD Graph, Office 365 Mail API, etc.

Ocurre lo mismo con cualquier recurso de terceros integrado en la plataforma de identidad de Microsoft. Cualquiera de estos recursos también puede definir un conjunto de permisos que se puede usar para dividir la funcionalidad de ese recurso en fragmentos menores. Por ejemplo, [Microsoft Graph](https://graph.microsoft.com) tiene permisos definidos para realizar, entre otras, las tareas siguientes:

* Leer el calendario de un usuario
* Escribir en el calendario de un usuario
* Enviar correo como usuario

Mediante la definición de estos tipos de permisos, el recurso tiene control específico sobre sus datos y el modo en que la funcionalidad de la API se expone. Una aplicación de terceros puede solicitar estos permisos a los usuarios y administradores, quienes deben aprobar la solicitud para que la aplicación pueda acceder a los datos o actuar en nombre de un usuario. Al fragmentar la funcionalidad del recurso en conjuntos de permisos más pequeños, se pueden crear aplicaciones de terceros para solicitar solo los permisos concretos que necesitan para realizar sus tareas. Los usuarios y administradores pueden saber exactamente qué datos de la aplicación tenga acceso a, y pueden ser más seguros de que no se comporta de forma malintencionada. Los desarrolladores deben cumplir siempre con el concepto del mínimo privilegio y pedir solo los permisos que necesitan para que sus aplicaciones funcionen.

En OAuth 2.0, estos tipos de permisos se denominan *ámbitos*. También conocen a menudo como *permisos*. Un permiso se representa en la plataforma de identidad de Microsoft como un valor de cadena. Al igual que en el ejemplo de Microsoft Graph, el valor de ámbito cadena para cada permiso es:

* Leer el calendario de un usuario mediante `Calendars.Read`
* Escribir en el calendario de un usuario mediante `Calendars.ReadWrite`
* Enviar correo electrónico con un usuario mediante `Mail.Send`

Normalmente, una aplicación solicita estos permisos especificando los ámbitos en las solicitudes a la plataforma Microsoft identity autorizan un extremo. Sin embargo, ciertos permisos con privilegios elevados solo pueden se concede mediante el consentimiento del administrador y solicita/conceder mediante el [punto de conexión de consentimiento de administrador](v2-permissions-and-consent.md#admin-restricted-permissions). Siga leyendo para obtener más información.

## <a name="permission-types"></a>Tipos de permisos

La plataforma de identidad de Microsoft admite dos tipos de permisos: **permisos delegados** y **permisos de aplicación**.

* **Permisos delegados**: se utilizan en aplicaciones que tienen un usuario con la sesión iniciada. Para estas aplicaciones, el usuario o un administrador da su consentimiento a los permisos que las solicitudes de aplicación y la aplicación es un permiso delegado para que actúe como el usuario que inició sesión al realizar llamadas para el recurso de destino. Algunos permisos delegados pueden ser consentidos por usuarios que no sean administradores, pero otros con más privilegios requieren el [consentimiento del administrador](v2-permissions-and-consent.md#admin-restricted-permissions). Para información sobre qué roles de administrador pueden consentir los permisos delegados, consulte [Permisos de roles de administrador en Azure AD](../users-groups-roles/directory-assign-admin-roles.md).

* **Permisos de aplicación**: los usan las aplicaciones que se ejecutan sin la presencia de un usuario con la sesión iniciada; por ejemplo, las aplicaciones que se ejecutan como demonios o servicios en segundo plano.  Los permisos de aplicación solo pueden ser [aceptados por un administrador](v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant).

_Los permisos efectivos_ son los que la aplicación tendrá al realizar solicitudes al recurso de destino. Es importante comprender la diferencia entre los delegados y permisos de aplicación que se concede a la aplicación y sus permisos efectivos al realizar llamadas para el recurso de destino.

- En el caso de los permisos delegados, los _permisos efectivos_ de la aplicación serán la intersección con menos privilegios de los permisos delegados que se le han concedido a la aplicación (mediante consentimiento) y los privilegios del usuario que tiene iniciada la sesión actualmente. La aplicación nunca puede tener más privilegios que el usuario que tiene la sesión iniciada. Dentro de las organizaciones, los privilegios del usuario que tiene la sesión iniciada pueden determinarse mediante directivas o pertenencia a uno o varios roles de administrador. Para información sobre qué roles de administrador pueden consentir los permisos delegados, consulte [Permisos de roles de administrador en Azure AD](../users-groups-roles/directory-assign-admin-roles.md).

   Por ejemplo, suponga que a su aplicación se le ha concedido el permiso delegado _User.ReadWrite.All_ en Microsoft Graph. Este permiso concede a su aplicación de forma nominal un permiso para leer y actualizar el perfil de cada usuario de una organización. Si el usuario que inició sesión es un administrador global, la aplicación podrá actualizar el perfil de cada usuario de la organización. Sin embargo, si el usuario que inició sesión no está en un rol de administrador, la aplicación podrá actualizar solo el perfil del usuario con sesión iniciada. No podrá actualizar los perfiles de otros usuarios de la organización, porque el usuario para el que tiene permiso para actuar en su nombre no tiene tales privilegios.
  
- Para los permisos de aplicación, los _permisos efectivos_ de la aplicación serán el nivel completo de privilegios que concede el permiso. Por ejemplo, una aplicación que tiene el permiso de aplicación _User.ReadWrite.All_ puede actualizar el perfil de cada usuario de la organización. 

## <a name="openid-connect-scopes"></a>Ámbitos de OpenID Connect

La implementación de plataforma de identidad de Microsoft de OpenID Connect tiene unos cuantos ámbitos bien definidos que no se aplican a un recurso concreto: `openid`, `email`, `profile`, y `offline_access`. Los ámbitos de OpenID Connect `address` y `phone` no son compatibles.

### <a name="openid"></a>openid

Si una aplicación realiza el inicio de sesión mediante [OpenID Connect](active-directory-v2-protocols.md), debe solicitar el ámbito `openid`. El ámbito `openid` aparece en la pantalla de consentimiento de la cuenta profesional como el permiso "Sign you in" (Iniciar sesión) y en la pantalla de consentimiento de la cuenta personal de Microsoft como el permiso "View your profile and connect to apps and services using your Microsoft account" (Ver el perfil y conectarse a las aplicaciones y servicios con la cuenta de Microsoft). Este permiso permite que una aplicación reciba un identificador único para el usuario en forma de la notificación `sub` . También ofrece acceso de la aplicación al punto de conexión UserInfo. El `openid` ámbito puede usarse en el extremo de token de plataforma de identidad de Microsoft para adquirir tokens de identificador que se pueden usar la aplicación para la autenticación.

### <a name="email"></a>email

El ámbito `email` puede usarse con el ámbito `openid` y cualquier otro. Proporciona acceso de la aplicación a la dirección de correo electrónico principal del usuario en forma de la notificación `email`. El `email` notificación se incluye en un token solo si está asociada con la cuenta de usuario, que no siempre es el caso de una dirección de correo electrónico. Si usa el ámbito `email`, la aplicación debe estar preparada para controlar los casos en los que la notificación `email` no exista en el token.

### <a name="profile"></a>perfil

El ámbito `profile` puede usarse con el ámbito `openid` y cualquier otro. Proporciona acceso de la aplicación a una cantidad considerable de información sobre el usuario. Incluye la información que pueda tener acceso, pero no se limita a, nombre dado del usuario, apellido, nombre de usuario preferido e identificador de objeto. Para ver una lista completa de las notificaciones de perfil disponibles en el parámetro id_tokens para un usuario específico, consulte la [`id_tokens`referencia](id-tokens.md).

### <a name="offlineaccess"></a>offline_access

El ámbito [`offline_access` ](https://openid.net/specs/openid-connect-core-1_0.html#OfflineAccess) proporciona acceso de la aplicación a recursos en nombre del usuario durante un periodo de tiempo prolongado. En la página de consentimiento, este ámbito aparece como el permiso "Mantener el acceso a los datos a los que le ha dado acceso". Cuando un usuario aprueba el `offline_access` ámbito, la aplicación puede recibir tokens de actualización desde el extremo de token de plataforma de identidad de Microsoft. Los tokens de actualización tienen una duración larga. La aplicación puede obtener nuevos tokens de acceso cuando expiren los antiguos.

Si la aplicación no solicita explícitamente el ámbito `offline_access`, no recibirá tokens de actualización. Esto significa que cuando se canjea un código de autorización del [flujo del código de autorización de OAuth 2.0](active-directory-v2-protocols.md), solo se recibirá un token de acceso del punto de conexión `/token`. El token de acceso es válido durante un breve período de tiempo. Normalmente, expira al cabo de una hora. En ese momento, la aplicación tiene que redirigir al usuario de vuelta al punto de conexión `/authorize` para que obtenga un nuevo código de autorización. Durante esta redirección y, en función del tipo de aplicación, puede que el usuario tenga que volver a escribir sus credenciales o dar de nuevo el consentimiento a permisos. Mientras el `offline_access` ámbito solicitado automáticamente por el servidor, el cliente debe solicitar todavía para recibir los tokens de actualización.

Para obtener más información acerca de cómo obtener y usar tokens de actualización, consulte el [referencia del protocolo de plataforma de identidad de Microsoft](active-directory-v2-protocols.md).

## <a name="requesting-individual-user-consent"></a>Solicitud de consentimiento de usuario individual

En una solicitud de autorización de [OpenID Connect o OAuth 2.0](active-directory-v2-protocols.md), una aplicación puede solicitar los permisos que necesita mediante el parámetro de consulta `scope`. Por ejemplo, cuando un usuario inicia sesión en una aplicación, la aplicación envía una solicitud como la del ejemplo siguiente (se han agregado saltos de línea para una mayor legibilidad):

```
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&response_type=code
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
&response_mode=query
&scope=
https%3A%2F%2Fgraph.microsoft.com%2Fcalendars.read%20
https%3A%2F%2Fgraph.microsoft.com%2Fmail.send
&state=12345
```

El parámetro `scope` es una lista separada por espacios que incluye los permisos delegados que la aplicación solicita. Cada permiso se indica anexando el valor del permiso al identificador del recurso (URI del identificador de aplicación). En la solicitud de ejemplo, la aplicación necesita permiso para leer el calendario del usuario y enviar correo electrónico en nombre del usuario.

Después de que el usuario escribe sus credenciales, el punto de conexión de plataforma de identidad de Microsoft busca un registro coincidente de *consentimiento del usuario*. Si el usuario no dio su consentimiento a cualquiera de los permisos solicitados en el pasado, ni ha administrador dado su consentimiento a estos permisos en nombre de toda la organización, el punto de conexión de plataforma de identidad de Microsoft le pide al usuario que conceda los permisos solicitados.

> [!NOTE]
> En este momento, los permisos `offline_access` ("Mantener el acceso a los datos a los que le ha dado acceso") y `user.read` ("Iniciar sesión y leer su perfil") se incluyen automáticamente en el consentimiento inicial para una aplicación.  Estos permisos suelen ser necesarios para que la aplicación funcione correctamente: `offline_access` da acceso a la aplicación a los tokens de actualización, críticos para aplicaciones nativas y web, mientras que `user.read` da acceso a la notificación `sub`, lo que permite al cliente o a la aplicación identificar correctamente al usuario con el tiempo y acceder a información de usuario rudimentaria.  

![Consentimiento de la cuenta de trabajo](./media/v2-permissions-and-consent/work_account_consent.png)

Cuando el usuario aprueba la solicitud del permiso, el consentimiento se registra y el usuario no tiene que volver a dar su consentimiento en los sucesivos inicios de sesión en la aplicación.

## <a name="requesting-consent-for-an-entire-tenant"></a>Solicitud de consentimiento para un inquilino al completo

A menudo, cuando una organización adquiere una licencia o una suscripción de una aplicación, quiere configurar de forma proactiva la aplicación para que la usen todos sus miembros. Como parte de este proceso, el administrador puede conceder consentimiento a la aplicación para que actúe en nombre de cualquier usuario del inquilino. Si el administrador concede el consentimiento del inquilino entero, los usuarios de la organización no verán una página de consentimiento para la aplicación.

Para solicitar el consentimiento de los permisos delegados para todos los usuarios de un inquilino, la aplicación puede usar el punto de conexión de consentimiento del administrador.

Además, las aplicaciones deben usar el punto de conexión de consentimiento del administrador para solicitar los permisos de la aplicación.

## <a name="admin-restricted-permissions"></a>Permisos restringidos por el administrador

Algunos permisos de privilegios elevados del ecosistema de Microsoft se pueden establecer en *restringido para los administradores*. Algunos ejemplos de estos tipos de permisos son los siguientes:

* Leer los perfiles completos de todos los usuarios con `User.Read.All`
* Escribir datos en el directorio de una organización mediante `Directory.ReadWrite.All`
* Leer todos los grupos de seguridad del directorio de una organización con: `Groups.Read.All`

Aunque un usuario consumidor podría conceder acceso de la aplicación a este tipo de datos, los usuarios de la organización tienen la restricción de no conceder acceso al mismo conjunto de datos confidenciales de la empresa. Si la aplicación solicita acceso a uno de estos permisos desde un usuario de la organización, el usuario recibe un mensaje de error que dice que no están autorizados para dar su consentimiento a permisos de la aplicación.

Si la aplicación necesita acceso a los ámbitos restringidos para los administradores en una organización, debe solicitarlos directamente a un administrador de la empresa, también mediante el punto de conexión de consentimiento del administrador, que se describe a continuación.

Si la aplicación solicita permisos delegados con altos privilegios y un administrador los concede a través del punto de conexión de consentimiento del administrador, el consentimiento se concede a todos los usuarios del inquilino.

Si la aplicación solicita permisos de la aplicación y un administrador concede que estos permisos mediante el Administrador de punto de conexión de consentimiento, no se realiza esta concesión en nombre de un usuario específico. En su lugar, a la aplicación cliente se le conceden los permisos *directamente*. Estos tipos de permisos solo se usan los servicios de demonio y otras aplicaciones no interactivas que se ejecutan en segundo plano.

## <a name="using-the-admin-consent-endpoint"></a>Uso del punto de conexión de consentimiento del administrador

Cuando el administrador de una empresa use la aplicación y se dirija al punto de conexión de autorización, la plataforma de identidad de Microsoft detectará el rol del usuario y le preguntará si desea dar el consentimiento en nombre del inquilino completo para los permisos que se hayan solicitado. Sin embargo, también hay un punto de conexión de consentimiento de administrador dedicado que puede usar si desea solicitar de forma proactiva que un administrador conceda permiso en nombre de todo el inquilino. Con este punto de conexión también es necesario para solicitar permisos de aplicación (que no se pueden solicitar mediante el extremo authorize).

Si sigue estos pasos, la aplicación puede recopilar los permisos para todos los usuarios de un inquilino, incluso en los ámbitos restringidos para los administradores. Se trata de una operación con privilegios elevados y solo debe realizarse si es necesario.

Para ver un ejemplo de código que implementa lo pasos, consulte el [ejemplo de ámbitos restringidos para los administradores](https://github.com/Azure-Samples/active-directory-dotnet-admin-restricted-scopes-v2).

### <a name="request-the-permissions-in-the-app-registration-portal"></a>Solicitud de los permisos en el portal de registro de aplicaciones

El consentimiento del administrador no acepta un parámetro de ámbito, por lo que ningún permiso que se solicite debe definirse estáticamente en el registro de la aplicación. En general, es una práctica recomendada para asegurarse de que los permisos definidos estáticamente para una determinada aplicación son un superconjunto de los permisos que va a solicitar dinámicamente/incremental.

#### <a name="to-configure-the-list-of-statically-requested-permissions-for-an-application"></a>Para configurar la lista de permisos solicitados estáticamente para una aplicación

1. Vaya a la aplicación en el [Azure portal: registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) experimentar, o [crear una aplicación](quickstart-register-app.md) si no lo ha hecho ya.
2. Busque la sección de **permisos de Microsoft Graph** y agregue los permisos que necesite la aplicación.
3. **Guarde** el registro de aplicaciones.

### <a name="recommended-sign-the-user-into-your-app"></a>Se recomienda: Iniciar la sesión del usuario en la aplicación

Normalmente, cuando se compila una aplicación que usa el punto de conexión de consentimiento del administrador, la aplicación necesita una página o una vista en la que el administrador pueda aprobar los permisos de la aplicación. Esta página puede ser parte del flujo de inicio de sesión de la aplicación o de la configuración de la aplicación, o bien puede ser un flujo de "conexión" dedicado. En muchos casos, tiene sentido que la aplicación muestre esta vista de conexión solo después de que un usuario haya iniciado sesión con una cuenta Microsoft profesional o educativa.

Al iniciar la sesión del usuario en la aplicación, puede identificar la organización a la que pertenece el administrador antes de pedirle que apruebe los permisos necesarios. Aunque no es estrictamente necesario, puede ayudarle a crear una experiencia más intuitiva para los usuarios de la organización. Para iniciar la sesión del usuario, siga nuestro [tutoriales del protocolo de plataforma de identidad de Microsoft](active-directory-v2-protocols.md).

### <a name="request-the-permissions-from-a-directory-admin"></a>Solicitud de los permisos de un administrador de directorios

Cuando esté listo para solicitar permisos de administrador de su organización, puede redirigir al usuario a la plataforma Microsoft identity *punto de conexión de consentimiento de administrador*.

```
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/adminconsent?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&state=12345
&redirect_uri=http://localhost/myapp/permissions
```

```
// Pro tip: Try pasting the below request in a browser!
```

```
https://login.microsoftonline.com/common/adminconsent?client_id=6731de76-14a6-49ae-97bc-6eba6914391e&state=12345&redirect_uri=http://localhost/myapp/permissions
```

| Parámetro | Condición | DESCRIPCIÓN |
| --- | --- | --- |
| `tenant` | Obligatorio | El inquilino de directorio al que quiere solicitar permiso. Puede proporcionarse en formato de GUID o de nombre descriptivo, O puede hacerse referencia genéricamente con `common`, como se muestra en el ejemplo. |
| `client_id` | Obligatorio | El **Id. de aplicación (cliente)** que la [Azure portal: registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) experiencia asignado a la aplicación. |
| `redirect_uri` | Obligatorio |El URI de redirección adonde desea que se envíe la respuesta para que la controle la aplicación. Debe coincidir exactamente con uno de los identificadores URI de redirección que registró el Portal de registro de aplicaciones. |
| `state` | Recomendada | Un valor incluido en la solicitud que se devolverá también en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. Use el estado para codificar información sobre el estado del usuario en la aplicación antes de que se produzca la solicitud de autenticación, por ejemplo, la página o vista en la que estaba. |

En este momento, Azure AD requiere que un administrador de inquilinos inicie sesión para completar la solicitud. Se pedirá al administrador que apruebe todos los permisos que ha solicitado para la aplicación en el Portal de registro de aplicaciones.

#### <a name="successful-response"></a>Respuesta correcta

Si el administrador aprueba los permisos para la aplicación, la respuesta correcta tendrá un aspecto similar al siguiente:

```
GET http://localhost/myapp/permissions?tenant=a8990e1f-ff32-408a-9f8e-78d3b9139b95&state=state=12345&admin_consent=True
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| `tenant` | El inquilino del directorio que concedió los permisos solicitados, en formato GUID. |
| `state` | Un valor incluido en la solicitud que también se devolverá en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. El estado se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, por ejemplo, la página o vista en la que estaban. |
| `admin_consent` | Se establecerá en `True`. |

#### <a name="error-response"></a>Respuesta de error

Si el administrador no aprueba los permisos de la aplicación, la respuesta de error tendrá el aspecto siguiente:

```
GET http://localhost/myapp/permissions?error=permission_denied&error_description=The+admin+canceled+the+request
```

| Parámetro | DESCRIPCIÓN |
| --- | --- |
| `error` | Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y para reaccionar ante ellos. |
| `error_description` | Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error. |

Cuando haya recibido una respuesta correcta del punto de conexión de consentimiento del administrador, la aplicación habrá obtenido los permisos solicitados. Ahora puede solicitar un token para el recurso que desee.

## <a name="using-permissions"></a>Uso de permisos

Después de que el usuario da su consentimiento para los permisos de la aplicación, la aplicación puede obtener tokens de acceso que representan los permisos de la aplicación para acceder a un recurso de alguna manera. Un token de acceso solo se puede usar para un recurso, pero dentro del token de acceso están todos los permisos codificados que se han concedido a la aplicación para ese recurso. Para adquirir un token de acceso, la aplicación puede realizar una solicitud al Microsoft identity platform extremo de token, similar al siguiente:

```
POST common/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/json

{
    "grant_type": "authorization_code",
    "client_id": "6731de76-14a6-49ae-97bc-6eba6914391e",
    "scope": "https://outlook.office.com/mail.read https://outlook.office.com/mail.send",
    "code": "AwABAAAAvPM1KaPlrEqdFSBzjqfTGBCmLdgfSTLEMPGYuNHSUYBrq..."
    "redirect_uri": "https://localhost/myapp",
    "client_secret": "zc53fwe80980293klaj9823"  // NOTE: Only required for web apps
}
```

Puede usar el token de acceso resultante en las solicitudes HTTP al recurso. Dicho token indica al recurso de forma confiable que la aplicación tiene el permiso apropiado para realizar una tarea específica. 

Para obtener más información sobre el protocolo OAuth 2.0 y cómo obtener tokens de acceso, consulte el [referencia del protocolo de punto de conexión de Microsoft identity platform](active-directory-v2-protocols.md).

## <a name="the-default-scope"></a>Ámbito /.default

Puede usar el `/.default` ámbito que ayude a migrar sus aplicaciones desde el punto de conexión v1.0 para el punto de conexión de plataforma de identidad de Microsoft. Se trata de un ámbito integrado para todas las aplicaciones, que hace referencia a la lista estática de permisos configurados en el registro de aplicación. Un valor `scope` de `https://graph.microsoft.com/.default` tiene exactamente la misma funcionalidad que los puntos de conexión v1.0 `resource=https://graph.microsoft.com`; es decir, solicita un token con los ámbitos en Microsoft Graph que la aplicación ha registrado en Azure Portal.

El ámbito de /.default se pueden usar en cualquier flujo de OAuth 2.0, pero es necesario en el [flujo de On-Behalf-Of](v2-oauth2-on-behalf-of-flow.md) y [flujo de credenciales de cliente](v2-oauth2-client-creds-grant-flow.md).  

> [!NOTE]
> Los clientes no pueden combinar estático (`/.default`) y consentimiento dinámico en una sola solicitud. Por tanto, `scope=https://graph.microsoft.com/.default+mail.read` generará un error debido a la combinación de los tipos de ámbitos.

### <a name="default-and-consent"></a>/.default y consentimiento

El ámbito `/.default` también desencadena el comportamiento del punto de conexión v1.0 para `prompt=consent`. Solicita el consentimiento para todos los permisos registrados por la aplicación, con independencia del recurso. Si se incluye como parte de la solicitud, el `/.default` ámbito devuelve un token que contiene los ámbitos para el recurso solicitado.

### <a name="default-when-the-user-has-already-given-consent"></a>/.default cuando el usuario ya ha dado el consentimiento

Dado que `/.default` tiene exactamente la misma funcionalidad que el comportamiento del punto de conexión v1.0 basado en `resource`, también aporta el comportamiento de consentimiento del punto de conexión v1.0. Esto quiere decir que `/.default` solo desencadena una petición de consentimiento si el usuario no ha concedido ningún permiso entre el cliente y el recurso. Si existe tal consentimiento, se devolverá un token que contendrá todos los ámbitos concedidos por el usuario para dicho recurso. Sin embargo, si no se ha concedido ningún permiso o si se ha proporcionado el parámetro `prompt=consent`, se mostrará una petición de consentimiento para todos los ámbitos registrados por la aplicación cliente.

#### <a name="example-1-the-user-or-tenant-admin-has-granted-permissions"></a>Ejemplo 1: El usuario o administrador de inquilinos han concedido permisos

El usuario o un administrador de inquilinos han concedido al cliente los permisos de Microsoft Graph `mail.read` y `user.read`. Si el cliente realiza una solicitud de `scope=https://graph.microsoft.com/.default`, no se mostrará ninguna petición de consentimiento independientemente del contenido de los permisos registrados por las aplicaciones cliente para Microsoft Graph. Se devolvería un token que incluiría los ámbitos `mail.read` y `user.read`.

#### <a name="example-2-the-user-hasnt-granted-permissions-between-the-client-and-the-resource"></a>Ejemplo 2: El usuario no ha concedido permisos entre el cliente y el recurso

No existe ningún consentimiento para el usuario entre el cliente y Microsoft Graph. El cliente se ha registrado en los permisos `user.read` y `contacts.read`, así como en el ámbito `https://vault.azure.net/user_impersonation`. de Azure Key Vault. Cuando el cliente solicita un token para `scope=https://graph.microsoft.com/.default`, el usuario verá una pantalla de consentimiento para `user.read`, `contacts.read` y los ámbitos `user_impersonation` de Key Vault. El token devuelto solo contendrán los ámbitos `user.read` y `contacts.read`.

#### <a name="example-3-the-user-has-consented-and-the-client-requests-additional-scopes"></a>Ejemplo 3: El usuario ha dado su consentimiento y el cliente solicita ámbitos adicionales

El usuario ya ha dado su consentimiento a `mail.read` para el cliente. El cliente se ha registrado en el ámbito `contacts.read` en su registro. Cuando el cliente realiza una solicitud para un token con `scope=https://graph.microsoft.com/.default` y solicita el consentimiento mediante `prompt=consent`, el usuario verá una pantalla de consentimiento para todos los permisos registrados por la aplicación. `contacts.read` aparecerá en la pantalla de consentimiento, pero `mail.read` no se mostrará. El token devuelto será para Microsoft Graph y contendrá `mail.read` y `contacts.read`.

### <a name="using-the-default-scope-with-the-client"></a>Uso del ámbito /.default con el cliente

Existe un caso especial del ámbito `/.default` en el que un cliente solicita su propio ámbito `/.default`. En el siguiente ejemplo se muestra este escenario.

```
// Line breaks are for legibility only.

GET https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
response_type=token            //code or a hybrid flow is also possible here
&client_id=9ada6f8a-6d83-41bc-b169-a306c21527a5
&scope=9ada6f8a-6d83-41bc-b169-a306c21527a5/.default
&redirect_uri=https%3A%2F%2Flocalhost
&state=1234
```

Se abre una pantalla de consentimiento para todos los permisos registrados (si es aplicable en función de las descripciones anteriores de consentimiento y `/.default`); a continuación, se devuelve un token de identificación en lugar de un token de acceso.  Este comportamiento existe para determinados clientes heredados de movimiento de ADAL a MSAL y no debe usarse por los clientes nuevos como destino el punto de conexión de plataforma de identidad de Microsoft.  

## <a name="troubleshooting-permissions-and-consent"></a>Solución de problemas con permisos y consentimientos

Si usted o los usuarios de su aplicación observan errores inesperados durante el proceso de consentimiento, consulte en este artículo los pasos para solucionar el problema: [Error inesperado al otorgar consentimiento a una aplicación](../manage-apps/application-sign-in-unexpected-user-consent-error.md).
