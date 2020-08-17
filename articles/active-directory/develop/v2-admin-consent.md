---
title: Protocolos de consentimiento del administrador de la Plataforma de identidad de Microsoft
description: Descripción de la autorización en el punto de conexión de la Plataforma de identidad de Microsoft, incluidos los ámbitos, los permisos y el consentimiento.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 12/3/2019
ms.author: ryanwi
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: cb629b80958ed2897f76eb099f738c33b48c3696
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119613"
---
# <a name="admin-consent-on-the-microsoft-identity-platform"></a>Consentimiento del administrador en la Plataforma de identidad de Microsoft

Algunos permisos requieren el consentimiento de un administrador antes de poder concederlos en un inquilino.  También puede usar el punto de conexión de consentimiento del administrador para conceder permisos a todo un inquilino.

## <a name="recommended-sign-the-user-into-your-app"></a>Se recomienda: Iniciar la sesión del usuario en la aplicación

Normalmente, cuando se compila una aplicación que usa el punto de conexión de consentimiento del administrador, la aplicación necesita una página o una vista en la que el administrador pueda aprobar los permisos de la aplicación. Esta página puede ser parte del flujo de inicio de sesión de la aplicación o de la configuración de la aplicación, o bien puede ser un flujo de "conexión" dedicado. En muchos casos, tiene sentido que la aplicación muestre esta vista de conexión solo después de que un usuario haya iniciado sesión con una cuenta Microsoft profesional o educativa.

Al iniciar la sesión del usuario en la aplicación, puede identificar la organización a la que pertenece el administrador antes de pedirle que apruebe los permisos necesarios. Aunque no es estrictamente necesario, puede ayudarle a crear una experiencia más intuitiva para los usuarios de la organización. Para iniciar la sesión del usuario, siga nuestros [tutoriales del protocolo de la Plataforma de identidad de Microsoft](active-directory-v2-protocols.md).

## <a name="request-the-permissions-from-a-directory-admin"></a>Solicitud de los permisos de un administrador de directorios

Cuando esté listo para solicitar permisos al administrador de la organización, puede redirigir al usuario al *punto de conexión de consentimiento del administrador* de la Plataforma de identidad de Microsoft.

```HTTP
// Line breaks are for legibility only.
GET https://login.microsoftonline.com/{tenant}/v2.0/adminconsent?
client_id=6731de76-14a6-49ae-97bc-6eba6914391e
&state=12345
&redirect_uri=http://localhost/myapp/permissions
&scope=
https://graph.microsoft.com/calendars.read
https://graph.microsoft.com/mail.send
```


| Parámetro     | Condición     | Descripción                                                                               |
|--------------:|--------------:|:-----------------------------------------------------------------------------------------:|
| `tenant` | Obligatorio | El inquilino de directorio al que quiere solicitar permiso. Puede proporcionarse en formato de GUID o de nombre descriptivo, O puede hacerse referencia genéricamente con `organizations`, como se muestra en el ejemplo. No use "común", ya que las cuentas personales no pueden proporcionar consentimiento del administrador salvo en el contexto de un inquilino. Para garantizar una mejor compatibilidad con las cuentas personales que administran los inquilinos, use el identificador de inquilino cuando sea posible. |
| `client_id` | Obligatorio | El **identificador de aplicación (cliente)** que la experiencia [Azure Portal: Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) asignó a la aplicación. |
| `redirect_uri` | Obligatorio |El URI de redireccionamiento adonde desea que se envíe la respuesta para que la controle la aplicación. Debe coincidir exactamente con uno de los identificadores URI de redirección que registró el Portal de registro de aplicaciones. |
| `state` | Recomendado | Un valor incluido en la solicitud que se devolverá también en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. Use el estado para codificar información sobre el estado del usuario en la aplicación antes de que se produzca la solicitud de autenticación, por ejemplo, la página o vista en la que estaba. |
|`scope`        | Obligatorio      | Define el conjunto de permisos que la aplicación solicita. Puede ser estático (mediante /.default) o ámbitos dinámicos.  Puede incluir ámbitos de OIDC (`openid`, `profile`, `email`). |


En este momento, Azure AD requiere que un administrador de inquilinos inicie sesión para completar la solicitud. Se solicita al administrador que apruebe todos los permisos solicitados en el parámetro `scope`.  Si ha usado un valor estático (`/.default`), funcionará como el punto de conexión de consentimiento del administración de la versión v1.0 y el consentimiento de solicitud para todos los ámbitos que se encuentran en los permisos necesarios para la aplicación.

### <a name="successful-response"></a>Respuesta correcta

Si el administrador aprueba los permisos para la aplicación, la respuesta correcta tendrá un aspecto similar al siguiente:

```
http://localhost/myapp/permissions?admin_consent=True&tenant=fa00d692-e9c7-4460-a743-29f2956fd429&state=12345&scope=https%3a%2f%2fgraph.microsoft.com%2fCalendars.Read+https%3a%2f%2fgraph.microsoft.com%2fMail.Send
```

| Parámetro         | Descripción                                                                                       |
|------------------:|:-------------------------------------------------------------------------------------------------:|
| `tenant`| El inquilino del directorio que concedió los permisos solicitados, en formato GUID.|
| `state`           | Un valor incluido en la solicitud que también se devolverá en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. El estado se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, por ejemplo, la página o vista en la que estaban.|
| `scope`          | Conjunto de permisos al que se concedió acceso a la aplicación.|
| `admin_consent`   | Se establecerá en `True`.|

### <a name="error-response"></a>Respuesta de error

`http://localhost/myapp/permissions?error=consent_required&error_description=AADSTS65004%3a+The+resource+owner+or+authorization+server+denied+the+request.%0d%0aTrace+ID%3a+d320620c-3d56-42bc-bc45-4cdd85c41f00%0d%0aCorrelation+ID%3a+8478d534-5b2c-4325-8c2c-51395c342c89%0d%0aTimestamp%3a+2019-09-24+18%3a34%3a26Z&admin_consent=True&tenant=fa15d692-e9c7-4460-a743-29f2956fd429&state=12345`

Cuando se agregan a los parámetros que se ven en una respuesta correcta, los parámetros de error se ven como se muestra a continuación.

| Parámetro          | Descripción                                                                                      |
|-------------------:|:-------------------------------------------------------------------------------------------------:|
| `error`            | Una cadena de código de error que puede utilizarse para clasificar los tipos de errores que se producen y para reaccionar ante ellos.|
| `error_description`| Un mensaje de error específico que puede ayudar a un desarrollador a identificar la causa de un error.|
| `tenant`| El inquilino del directorio que concedió los permisos solicitados, en formato GUID.|
| `state`           | Un valor incluido en la solicitud que también se devolverá en la respuesta del token. Puede ser una cadena de cualquier contenido que desee. El estado se usa para codificar información sobre el estado del usuario en la aplicación antes de que se haya producido la solicitud de autenticación, por ejemplo, la página o vista en la que estaban.|
| `admin_consent`   | Se establecerá en `True` para indicar que esta respuesta se produjo en un flujo de consentimiento del administrador.|

## <a name="next-steps"></a>Pasos siguientes
- Consulte [Conversión de una aplicación de inquilino único en una aplicación multiinquilino](howto-convert-app-to-be-multi-tenant.md)
- Obtenga información sobre [cómo se admite el consentimiento en la capa del protocolo OAuth 2.0 durante el flujo de concesión del código de autorización](v2-oauth2-auth-code-flow.md#request-an-authorization-code).
- Aprenda [cómo una aplicación multiinquilino puede usar la plataforma de consentimiento ](./howto-convert-app-to-be-multi-tenant.md) para implementar el consentimiento de "usuario" y "administrador", que admite patrones de aplicación multinivel más avanzados.
- Descripción de las [experiencias de consentimiento de aplicaciones de Azure AD](application-consent-experience.md)