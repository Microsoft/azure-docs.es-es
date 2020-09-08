---
title: Adición de aprobaciones personalizadas a flujos de registro de autoservicio - Azure AD
description: 'Incorporación de conectores de API para flujos de trabajo de aprobación personalizados en el registro de autoservicio de identidades externas: Azure Active Directory (Azure AD)'
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: article
ms.date: 06/16/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: d664d7cd169593924917bb02a0220e4047eb0cdb
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165261"
---
# <a name="add-a-custom-approval-workflow-to-self-service-sign-up"></a>Adición de un flujo de trabajo de aprobaciones personalizado al registro de autoservicio

Con [conectores de API](api-connectors-overview.md), puede realizar la integración con sus propios flujos de trabajo de aprobaciones personalizados con el registro de autoservicio, para que pueda administrar qué cuentas de usuario invitado se crean en el inquilino.

En este artículo se proporciona un ejemplo de cómo realizar la integración con un sistema de aprobación. En este ejemplo, el flujo de usuario de registro de autoservicio recopila datos de usuario durante el proceso de registro y los pasa al sistema de aprobación. Después, el sistema de aprobación puede:

- Aprobar automáticamente el usuario y permitir que Azure AD cree la cuenta de usuario.
- Desencadenar una revisión manual. Si se aprueba la solicitud, el sistema de aprobación utiliza Microsoft Graph para aprovisionar la cuenta de usuario. El sistema de aprobación también puede notificar al usuario que se ha creado su cuenta.

## <a name="register-an-application-for-your-approval-system"></a>Registro de una aplicación para el sistema de aprobación

Debe registrar el sistema de aprobación como una aplicación en el inquilino de Azure AD para que pueda autenticarse en Azure AD y tener permiso para crear usuarios. Obtenga más información en [Aspectos básicos de autorización y autenticación de Microsoft Graph](https://docs.microsoft.com/graph/auth/auth-concepts).

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de Azure AD.
2. En **Servicios de Azure**, seleccione **Azure Active Directory**.
3. En el menú de la izquierda, seleccione **Registros de aplicaciones** y luego **Nuevo registro**.
4. Escriba un **Nombre** para la aplicación; por ejemplo, _Sign-up Approvals_.

   <!-- ![Register an application for the approval system](./self-service-sign-up-add-approvals/approvals/register-an-approvals-application.png) -->

5. Seleccione **Registrar**. Puede dejar todos los demás campos con sus valores predeterminados.

   ![Página de registro de una aplicación](media/self-service-sign-up-add-approvals/register-approvals-app.png)

6. En **Administrar**, en el menú de la izquierda, seleccione **Permisos de API** y, después, **Agregar un permiso**.
7. En la página **Solicitud de permisos de API**, seleccione **Microsoft Graph** y, después, **Permisos de la aplicación**.
8. En **Seleccionar permisos**, expanda **Usuario** y, después, seleccione la casilla **User.ReadWrite.All**. Este permiso permite al sistema de aprobación crear el usuario tras su aprobación. Seleccione **Agregar permisos**.

   ![Página de registro de una aplicación](media/self-service-sign-up-add-approvals/request-api-permissions.png)

9. En la página **Permisos de API**, seleccione **Conceder consentimiento de administrador para (nombre del inquilino)** y, después, seleccione **Sí**.
10. En **Administrar**, en el menú de la izquierda, seleccione **Certificados y secretos** y luego **Nuevo secreto de cliente**.
11. Escriba una **Descripción** para el secreto, por ejemplo, _Secreto de cliente de aprobaciones_ y seleccione la duración para la **Expiración** del secreto de cliente. A continuación, seleccione **Agregar**.
12. Copie el valor del secreto de cliente.

    ![Copia del secreto de cliente para su uso en el sistema de aprobación](media/self-service-sign-up-add-approvals/client-secret-value-copy.png)

13. Configure el sistema de aprobación para usar el **Id. de la aplicación** como el identificador de cliente y el **secreto de cliente** generado para autenticarse en Azure AD.

## <a name="create-the-api-connectors"></a>Creación de los conectores de API

A continuación, [creará los conectores de API](self-service-sign-up-add-api-connector.md#create-an-api-connector) para el flujo de usuario de registro de autoservicio. La API del sistema de aprobación necesita dos conectores y los puntos de conexión correspondientes, como los ejemplos que se muestran a continuación. Estos conectores de API hacen lo siguiente:

- **Comprobar el estado de aprobación**. Enviar una llamada al sistema de aprobación inmediatamente después de que un usuario inicie sesión con un proveedor de identidades, para comprobar si el usuario tiene una solicitud de aprobación existente o si ya se ha denegado. Si el sistema de aprobación solo toma decisiones de aprobación automáticas, es posible que este conector de API no sea necesario. Ejemplo de un conector de API "Check approval status".

  ![Configuración del conector de API Check approval status](./media/self-service-sign-up-add-approvals/check-approval-status-api-connector-config-alt.png)

- **Solicitar aprobación**. Enviar una llamada al sistema de aprobación después de que un usuario complete la página de la colección de atributos, pero antes de que se cree la cuenta de usuario, para solicitar la aprobación. La solicitud de aprobación se puede conceder automáticamente o revisar de forma manual. Ejemplo de un conector de API "Request approval". 

  ![Configuración del conector de API Request approval](./media/self-service-sign-up-add-approvals/create-approval-request-api-connector-config-alt.png)

Para crear estos conectores, siga los pasos descritos en [Creación de un conector de API](self-service-sign-up-add-api-connector.md#create-an-api-connector).

## <a name="enable-the-api-connectors-in-a-user-flow"></a>Habilitación de los conectores de API en un flujo de usuario

Ahora, agregará los conectores de API a un flujo de usuario de registro de autoservicio con estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de Azure AD.
2. En **Servicios de Azure**, seleccione **Azure Active Directory**.
3. En el menú de la izquierda, seleccione **External Identities**.
4. Seleccione **Flujos de usuario (versión preliminar)** y, después, seleccione el flujo de usuario para el que desea habilitar el conector de API.
5. Seleccione **Conectores de API** y, después, seleccione los puntos de conexión de API que desea invocar en los pasos siguientes del flujo de usuario:

   - **Después de iniciar sesión con un proveedor de identidades**: seleccione el conector de API del estado de aprobación; por ejemplo, _Check approval status_.
   - **Antes de crear el usuario**: seleccione el conector de API de solicitud de aprobación; por ejemplo, _Request approval_.

   ![Adición de API al flujo de usuario](./media/self-service-sign-up-add-approvals/api-connectors-user-flow-api.png)

6. Seleccione **Guardar**.

## <a name="control-the-sign-up-flow-with-api-responses"></a>Control del flujo de registro con respuestas de API

El sistema de aprobación puede usar sus respuestas cuando se llama para controlar el flujo de registro. 

### <a name="request-and-responses-for-the-check-approval-status-api-connector"></a>Solicitud y respuestas del conector de API "Check approval status"

Ejemplo de la solicitud recibida por la API desde el conector de API "Check approval status":

```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [ //Sent for Google and Facebook identity providers
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "givenName":"John",
 "lastName":"Smith",
 "ui_locales":"en-US"
}
```

Las notificaciones exactas enviadas a la API dependen de la información proporcionada por el proveedor de identidades. Siempre se envía "email".

#### <a name="continuation-response-for-check-approval-status"></a>Respuesta de continuación para "Check approval status"

El punto de conexión de la API **Check approval status** debe devolver una respuesta de continuación si:

- El usuario no ha solicitado previamente una aprobación.

Ejemplo de la respuesta de continuación:

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "Continue"
}
```

#### <a name="blocking-response-for-check-approval-status"></a>Respuesta de bloqueo para "Check approval status"

El punto de conexión de la API **Check approval status** debe devolver una respuesta de bloqueo si:

- La aprobación del usuario está pendiente.
- Se denegó el usuario y no se le debería permitir volver a solicitar la aprobación.

Estos son ejemplos de respuestas de bloqueo:

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "Your access request is already processing. You'll be notified when your request has been approved.",
    "code": "CONTOSO-APPROVAL-PENDING"
}
```

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "Your sign up request has been denied. Please contact an administrator if you believe this is an error",
    "code": "CONTOSO-APPROVAL-DENIED"
}
```

### <a name="request-and-responses-for-the-request-approval-api-connector"></a>Solicitud y respuestas del conector de API "Request approval"

Ejemplo de una solicitud HTTP recibida por la API desde el conector de API "Request approval":

```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [ //Sent for Google and Facebook identity providers
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "givenName":"John",
 "surname":"Smith",
 "jobTitle":"Supplier",
 "streetAddress":"1000 Microsoft Way",
 "city":"Seattle",
 "postalCode": "12345",
 "state":"Washington",
 "country":"United States",
 "extension_<extensions-app-id>_CustomAttribute1": "custom attribute value",
 "extension_<extensions-app-id>_CustomAttribute2": "custom attribute value",
 "ui_locales":"en-US"
}
```

Las notificaciones exactas enviadas a la API dependen de la información recopilada por el usuario o proporcionada por el proveedor de identidades.

#### <a name="continuation-response-for-request-approval"></a>Respuesta de continuación para "Request approval"

El punto de conexión de la API **Request approval** debe devolver una respuesta de continuación si:

- El usuario se puede **_aprobar automáticamente_**.

Ejemplo de la respuesta de continuación:

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "Continue"
}
```

> [!IMPORTANT]
> Si se recibe una respuesta de continuación, Azure AD crea una cuenta de usuario y dirige al usuario a la aplicación.

#### <a name="blocking-response-for-request-approval"></a>Respuesta de bloqueo para "Request approval"

El punto de conexión de la API **Request approval** debe devolver una respuesta de bloqueo si:

- Se ha creado una solicitud de aprobación de usuario y está pendiente.
- Se denegó automáticamente una solicitud de aprobación de usuario.

Estos son ejemplos de respuestas de bloqueo:

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "Your account is now waiting for approval. You'll be notified when your request has been approved.",
    "code": "CONTOSO-APPROVAL-REQUESTED"
}
```

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "Your sign up request has been denied. Please contact an administrator if you believe this is an error",
    "code": "CONTOSO-APPROVAL-AUTO-DENIED"
}
```

El valor `userMessage` se muestra al usuario en la respuesta; por ejemplo:

![Ejemplo de página de aprobación pendiente](./media/self-service-sign-up-add-approvals/approval-pending.png)

## <a name="user-account-creation-after-manual-approval"></a>Creación de cuentas de usuario después de la aprobación manual

Después de obtener la aprobación manual, el sistema de aprobación personalizado crea una cuenta de [usuario](https://docs.microsoft.com/graph/azuread-users-concept-overview) mediante [Microsoft Graph](https://docs.microsoft.com/graph/use-the-api). La forma en que el sistema de aprobación aprovisiona la cuenta de usuario depende del proveedor de identidades que el usuario utiliza.

### <a name="for-a-federated-google-or-facebook-user"></a>Para un usuario federado de Google o Facebook

> [!IMPORTANT]
> El sistema de aprobación debe comprobar explícitamente que `identities`, `identities[0]` y `identities[0].issuer` están presentes y que `identities[0].issuer` es igual a "facebook"' o "google" para usar este método.

Si el usuario ha iniciado sesión con una cuenta de Google o Facebook, puede usar la [API de creación de usuarios](https://docs.microsoft.com/graph/api/user-post-users?view=graph-rest-1.0&tabs=http).

1. El sistema de aprobación recibe la solicitud HTTP del flujo de usuario.

```http
POST <Approvals-API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@outlook.com",
 "identities": [
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "city": "Redmond",
 "extension_<extensions-app-id>_CustomAttribute": "custom attribute value",
 "ui_locales":"en-US"
}
```

2. El sistema de aprobación utiliza Microsoft Graph para crear una cuenta de usuario.

```http
POST https://graph.microsoft.com/v1.0/users
Content-type: application/json

{
 "userPrincipalName": "johnsmith_outlook.com#EXT@contoso.onmicrosoft.com",
 "accountEnabled": true,
 "mail": "johnsmith@outlook.com",
 "userType": "Guest",
 "identities": [
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "city": "Redmond",
 "extension_<extensions-app-id>_CustomAttribute": "custom attribute value"
}
```

| Parámetro                                           | Obligatorio | Descripción                                                                                                                                                            |
| --------------------------------------------------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| userPrincipalName                                   | Sí      | Se puede generar con la utilización de la notificación `email` enviada a la API, la sustitución del carácter `@` por `_` y anteponiéndolo a `#EXT@<tenant-name>.onmicrosoft.com`. |
| accountEnabled                                      | Sí      | Se debe establecer en `true`.                                                                                                                                                 |
| mail                                                | Sí      | Equivalente a la notificación `email` enviada a la API.                                                                                                               |
| userType                                            | Sí      | Debe ser `Guest`. Designa a este usuario como un usuario invitado.                                                                                                                 |
| Identidades                                          | Sí      | Información de la identidad federada.                                                                                                                                    |
| \<otherBuiltInAttribute>                            | No       | Otros atributos integrados, como `displayName`, `city` y otros. Los nombres de los parámetros son los mismos que los que envía el conector de API.                            |
| \<extension\_\{extensions-app-id}\_CustomAttribute> | No       | Atributos personalizados sobre el usuario. Los nombres de los parámetros son los mismos que los que envía el conector de API.                                                            |

### <a name="for-a-federated-azure-active-directory-user"></a>Para un usuario de Azure Active Directory federado

Si un usuario inicia sesión con una cuenta de Azure Active Directory federada, debe usar la [API de invitación](https://docs.microsoft.com/graph/api/invitation-post?view=graph-rest-1.0) para crear el usuario y, opcionalmente, la [API de actualización de usuario](https://docs.microsoft.com/graph/api/user-update?view=graph-rest-1.0) para asignar más atributos al usuario.

1. El sistema de aprobación recibe la solicitud HTTP del flujo de usuario.

```http
POST <Approvals-API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "displayName": "John Smith",
 "city": "Redmond",
 "extension_<extensions-app-id>_CustomAttribute": "custom attribute value",
 "ui_locales":"en-US"
}
```

2. El sistema de aprobación crea la invitación mediante el atributo `email` proporcionado por el conector de API.

```http
POST https://graph.microsoft.com/v1.0/invitations
Content-type: application/json

{
    "invitedUserEmailAddress":"johnsmith@fabrikam.onmicrosoft.com",
    "inviteRedirectUrl" : "https://myapp.com"
}
```

Ejemplo de la respuesta:

```http
HTTP/1.1 201 OK
Content-type: application/json

{
    ...
    "invitedUser": {
        "id": "<generated-user-guid>"
    }
}
```

3. El sistema de aprobación usa el identificador del usuario invitado para actualizar la cuenta del usuario con los atributos de usuario recopilados (opcional).

```http
PATCH https://graph.microsoft.com/v1.0/users/<generated-user-guid>
Content-type: application/json

{
    "displayName": "John Smith",
    "city": "Redmond",
    "extension_<extensions-app-id>_AttributeName": "custom attribute value"
}
```

## <a name="next-steps"></a>Pasos siguientes

- Introducción a los [ejemplos de inicio rápido de Azure Functions](code-samples-self-service-sign-up.md#api-connector-azure-function-quickstarts).
- Compruebe el [registro de autoservicio para usuarios invitados con el ejemplo de aprobación manual](code-samples-self-service-sign-up.md#custom-approval-workflows). 
