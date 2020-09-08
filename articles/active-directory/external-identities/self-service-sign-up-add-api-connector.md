---
title: 'Adición de conectores de API a flujos de autoservicio de registro: Azure AD'
description: Configurar una API Web para usarla en un flujo de usuario.
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
ms.openlocfilehash: 5f241fd038d0d7309d8e1e5578dd77f950261b68
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165182"
---
# <a name="add-an-api-connector-to-a-user-flow"></a>Adición de un conector de API a un flujo de usuario

Para usar un [conector de API](api-connectors-overview.md), primero debe crear el conector de API y, después, habilitarlo en un flujo de usuario.

## <a name="create-an-api-connector"></a>Creación de un conector de API

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de Azure AD.
2. En **Servicios de Azure**, seleccione **Azure Active Directory**.
3. En el menú de la izquierda, seleccione **External Identities**.
4. Seleccione **All API connectors (Preview)** (Todos los conectores de API [versión preliminar]) y, después, seleccione **New API connector** (Nuevo conector de API).

   ![Adición de un conector de API nuevo](./media/self-service-sign-up-add-api-connector/api-connector-new.png)

5. Escriba el nombre para mostrar de la llamada. Por ejemplo, **Comprobar el estado de aprobación**.
6. Proporcione el valor de **Dirección URL del punto de conexión** de la llamada API.
7. Indique la información de autenticación de la API.

   - Actualmente solo se admite la autenticación básica. Si desea usar una API sin autenticación básica con fines de desarrollo, solo tiene que escribir un **Nombre de usuario** y una **Contraseña** ficticios que la API pueda omitir. Si se utiliza con una función de Azure y una clave de API, puede incluir el código como un parámetro de consulta en **Dirección URL del punto de conexión** (por ejemplo, https[]()://contoso.azurewebsites.net/api/endpoint<b>?code=0123456789</b>).

   ![Adición de un conector de API nuevo](./media/self-service-sign-up-add-api-connector/api-connector-config.png)
8. Seleccione **Guardar**.

> [!IMPORTANT]
> Anteriormente, tenía que configurar los atributos de usuario que se enviaban a la API ("Claims to send") y los atributos de usuario que se aceptaban de la API ("Claims to receive"). Ahora, todos los atributos de usuario se envían de forma predeterminada si tienen un valor y la API puede devolver cualquier atributo de usuario en una respuesta de "continuación".

## <a name="the-request-sent-to-your-api"></a>Solicitud enviada a la API
Un conector de API se materializa como una solicitud **HTTP POST** y envía los atributos de usuario ("claims") como pares de clave y valor en un cuerpo JSON. Los atributos se serializan de forma similar a las propiedades de usuario de [Microsoft Graph](https://docs.microsoft.com/graph/api/resources/user?view=graph-rest-1.0#properties). 

**Solicitud de ejemplo**
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

Solo se pueden enviar en la solicitud las propiedades de usuario y los atributos personalizados que se enumeran en la experiencia **Azure Active Directory** > **Identidades externas** > **Atributos de usuario personalizados**.

Los atributos personalizados existen en el formato **extension_\<extensions-app-id>_AttributeName** en el directorio. La API esperará recibir las notificaciones con este mismo formato serializado. Para más información acerca de los atributos personalizados, consulte cómo [definir atributos personalizados para flujos de autoservicio de registro](user-flow-add-custom-attributes.md).

Además, en todas las solicitudes se envía de forma predeterminada la notificación de **configuración regional de UI ("ui_locales")** . Proporciona la configuración o configuraciones regionales de un usuario, tal como están definidas en su dispositivo que la API puede utilizar para devolver respuestas internacionalizadas.

> [!IMPORTANT]
> Si una notificación para enviar no tiene un valor en el momento en que se llama al punto de conexión de la API, la notificación no se enviará a la API. La API debe estar diseñada para comprobar explícitamente el valor que espera.

> [!TIP] 
> La API puede utilizar notificaciones de [**identidades ("identities")** ](https://docs.microsoft.com/graph/api/resources/objectidentity?view=graph-rest-1.0) y de **dirección de correo electrónico ("email")** para identificar a un usuario antes de que tenga una cuenta en el inquilino. La notificación "identities" se envía cuando un usuario se autentica con un proveedor de identidades como Google o Facebook. Siempre se envía "email".

## <a name="enable-the-api-connector-in-a-user-flow"></a>Habilitación del conector de API en un flujo de usuario

Siga estos pasos para agregar el conector de API a un flujo de usuario de autoservicio de registro.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de Azure AD.
2. En **Servicios de Azure**, seleccione **Azure Active Directory**.
3. En el menú de la izquierda, seleccione **External Identities**.
4. Seleccione **Flujos de usuario (versión preliminar)** y, después, seleccione el flujo de usuario para el que desea habilitar el conector de API.
5. Seleccione **Conectores de API** y, después, seleccione los puntos de conexión de API que desea invocar en los pasos siguientes del flujo de usuario:

   - **Después de iniciar sesión con un proveedor de identidades**
   - **Antes de crear el usuario**

   ![Adición de API al flujo de usuario](./media/self-service-sign-up-add-api-connector/api-connectors-user-flow-select.png)

6. Seleccione **Guardar**.

## <a name="after-signing-in-with-an-identity-provider"></a>Después de iniciar sesión con un proveedor de identidades

Inmediatamente después de que el usuario se autentique con un proveedor de identidades (Google, Facebook, Azure AD), se invoca un conector de API en este paso del proceso de registro. Este paso precede a la ***página de recopilación de atributos***, que es el formulario que se muestra al usuario para recopilar los atributos de usuario. 

<!-- The following are examples of API connector scenarios you may enable at this step:
- Use the email or federated identity that the user provided to look up claims in an existing system. Return these claims from the existing system, pre-fill the attribute collection page, and make them available to return in the token.
- Validate whether the user is included in an allow or deny list, and control whether they can continue with the sign-up flow. -->

### <a name="example-request-sent-to-the-api-at-this-step"></a>Solicitud de ejemplo enviada a la API en este paso
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

### <a name="expected-response-types-from-the-web-api-at-this-step"></a>Tipos de respuesta esperados de la API web en este paso

Cuando la API web recibe una solicitud HTTP de Azure AD durante un flujo de usuario, puede devolver estas respuestas:

- Respuesta de continuación
- Respuesta de bloqueo

#### <a name="continuation-response"></a>Respuesta de continuación

Una respuesta de continuación indica que el flujo de usuario debe continuar en el paso siguiente: la página de colección de atributos.

En una respuesta de continuación, la API puede devolver notificaciones. Si la API devuelve una notificación, esta realiza lo siguiente:

- Rellena previamente el campo de entrada en la página de colección de atributos.

Vea un ejemplo de una [respuesta de continuación](#example-of-a-continuation-response).

#### <a name="blocking-response"></a>Respuesta de bloqueo

Una respuesta de bloqueo termina el flujo de usuario. La API puede emitirla intencionadamente para detener el flujo de usuario y mostrar una página de bloqueo al usuario. La página de bloqueo muestra el mensaje `userMessage` proporcionado por la API.

Vea un ejemplo de una [respuesta de bloqueo](#example-of-a-blocking-response).

## <a name="before-creating-the-user"></a>Antes de crear el usuario

Después de la página de recopilación de atributos, se invoca un conector de API en este paso del proceso de registro, si se incluye uno. Este paso siempre se invoca antes de crear una cuenta de usuario en Azure AD. 

<!-- The following are examples of scenarios you might enable at this point during sign-up: -->
<!-- 
- Validate user input data and ask a user to resubmit data.
- Block a user sign-up based on data entered by the user.
- Perform identity verification.
- Query external systems for existing data about the user and overwrite the user-provided value. -->

### <a name="example-request-sent-to-the-api-at-this-step"></a>Solicitud de ejemplo enviada a la API en este paso

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

### <a name="expected-response-types-from-the-web-api-at-this-step"></a>Tipos de respuesta esperados de la API web en este paso

Cuando la API web recibe una solicitud HTTP de Azure AD durante un flujo de usuario, puede devolver estas respuestas:

- Respuesta de continuación
- Respuesta de bloqueo
- Respuesta de validación

#### <a name="continuation-response"></a>Respuesta de continuación

Una respuesta de continuación indica que el flujo de usuario debe continuar en el paso siguiente: creación del usuario en el directorio.

En una respuesta de continuación, la API puede devolver notificaciones. Si la API devuelve una notificación, esta realiza lo siguiente:

- Invalida cualquier valor que ya se haya asignado a la notificación de la página de colección de atributos.

Vea un ejemplo de una [respuesta de continuación](#example-of-a-continuation-response).

#### <a name="blocking-response"></a>Respuesta de bloqueo
Una respuesta de bloqueo termina el flujo de usuario. La API puede emitirla intencionadamente para detener el flujo de usuario y mostrar una página de bloqueo al usuario. La página de bloqueo muestra el mensaje `userMessage` proporcionado por la API.

Vea un ejemplo de una [respuesta de bloqueo](#example-of-a-blocking-response).

### <a name="validation-error-response"></a>Respuesta de error de validación
 Cuando la API responde con una respuesta de error de validación, el flujo de usuario permanece en la página de colección de atributos y se muestra `userMessage` al usuario. El usuario puede editar el formulario y volver a enviarlo. Este tipo de respuesta se puede usar para la validación de datos de entrada.

Vea un ejemplo de una [respuesta de error de validación](#example-of-a-validation-error-response).

## <a name="example-responses"></a>Respuestas de ejemplo

### <a name="example-of-a-continuation-response"></a>Ejemplo de una respuesta de continuación

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "Continue",
    "postalCode": "12349", // return claim
    "extension_<extensions-app-id>_CustomAttribute": "value" // return claim
}
```

| Parámetro                                          | Tipo              | Obligatorio | Descripción                                                                                                                                                                                                                                                                            |
| -------------------------------------------------- | ----------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| version                                            | String            | Sí      | La versión de la API.                                                                                                                                                                                                                                                                |
| action                                             | String            | Sí      | El valor debe ser `Continue`.                                                                                                                                                                                                                                                              |
| \<builtInUserAttribute>                            | \<attribute-type> | No       | Los valores se pueden almacenar en el directorio si están seleccionados como **Claim to receive** (Notificación para recibir) en la configuración del conector de API y **User attribute** (Atributo de usuario) de un flujo de usuario. Los valores se pueden devolver en el token si están seleccionados como **Application claim** (Notificación de aplicación).                                              |
| \<extension\_{extensions-app-id}\_CustomAttribute> | \<attribute-type> | No       | No es necesario que la notificación devuelta contenga `_<extensions-app-id>_`. Los valores se almacenan almacenar en el directorio si están seleccionados como **Claim to receive** (Notificación para recibir) en la configuración del conector de API y **User attribute** (Atributo de usuario) de un flujo de usuario. Los atributos personalizados no se pueden devolver en el token. |

### <a name="example-of-a-blocking-response"></a>Ejemplo de una respuesta de bloqueo

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "There was a problem with your request. You are not able to sign up at this time.",
    "code": "CONTOSO-BLOCK-00"
}

```

| Parámetro   | Tipo   | Obligatorio | Descripción                                                                |
| ----------- | ------ | -------- | -------------------------------------------------------------------------- |
| version     | String | Sí      | La versión de la API.                                                    |
| action      | String | Sí      | El valor debe ser `ShowBlockPage`.                                              |
| userMessage | String | Sí      | Mensaje que se va a mostrar al usuario.                                            |
| código        | String | No       | Código de error. Se puede usar con fines de depuración. No se muestra al usuario. |

**Experiencia del usuario final con una respuesta de bloqueo**

![Página de bloqueo de ejemplo](./media/api-connectors-overview/blocking-page-response.png)

### <a name="example-of-a-validation-error-response"></a>Ejemplo de una respuesta de error de validación

```http
HTTP/1.1 400 Bad Request
Content-type: application/json

{
    "version": "1.0.0",
    "status": 400,
    "action": "ValidationError",
    "userMessage": "Please enter a valid Postal Code.",
    "code": "CONTOSO-VALIDATION-00"
}
```

| Parámetro   | Tipo    | Obligatorio | Descripción                                                                |
| ----------- | ------- | -------- | -------------------------------------------------------------------------- |
| version     | String  | Sí      | La versión de la API.                                                    |
| action      | String  | Sí      | El valor debe ser `ValidationError`.                                           |
| status      | Entero | Sí      | Debe ser un valor `400` para una respuesta ValidationError.                        |
| userMessage | String  | Sí      | Mensaje que se va a mostrar al usuario.                                            |
| código        | String  | No       | Código de error. Se puede usar con fines de depuración. No se muestra al usuario. |

**Experiencia del usuario final con una respuesta de error de validación**

![Página de validación de ejemplo](./media/api-connectors-overview/validation-error-postal-code.png)

## <a name="using-azure-functions"></a>Uso de Azure Functions
Puede utilizar un desencadenador HTTP en Azure Functions como una forma sencilla de crear un punto de conexión de API para usarlo con el conector de API. Puede usar la función de Azure para, [por ejemplo](code-samples-self-service-sign-up.md#api-connector-azure-function-quickstarts), crear la lógica de validación y limitar los registros a dominios específicos. También puede llamar e invocar otras API web, almacenes de usuarios y otros servicios en la nube de Azure Functions para amplios escenarios.

## <a name="next-steps"></a>Pasos siguientes

<!-- - Learn [where you can enable an API connector](api-connectors-overview.md#where-you-can-enable-an-api-connector-in-a-user-flow) -->
- Obtenga información sobre cómo [agregar un flujo de trabajo personalizado de aprobación al autoservicio de registro](self-service-sign-up-add-approvals.md).
- Vea una introducción a los [ejemplos de inicio rápido de Azure Functions](code-samples-self-service-sign-up.md#api-connector-azure-function-quickstarts).
<!-- - Learn how to [use API connectors to verify a user identity](code-samples-self-service-sign-up.md#identity-verification) -->
