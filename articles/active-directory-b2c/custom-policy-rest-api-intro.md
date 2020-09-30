---
title: Intercambios de notificaciones de la API de REST en la directiva personalizada B2C
titleSuffix: Azure AD B2C
description: Introducción a la creación de un recorrido del usuario de Azure AD B2C que interactúa con servicios RESTful.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 05/18/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: bc2b72779460c2b7e3999204ace50ca57388b9a2
ms.sourcegitcommit: 1b320bc7863707a07e98644fbaed9faa0108da97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89594193"
---
# <a name="integrate-rest-api-claims-exchanges-in-your-azure-ad-b2c-custom-policy"></a>Integración de intercambios de notificaciones de la API de REST en la directiva personalizada de Azure AD B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Identity Experience Framework, que subyace a Azure Active Directory B2C (Azure AD B2C), permite la integración en API de RESTful dentro de un recorrido del usuario. En este artículo se muestra cómo crear un recorrido de usuario que interactúe con un servicio RESTful mediante un [perfil técnico de RESTful](restful-technical-profile.md).

Mediante Azure AD B2C, puede agregar su propia lógica de negocios a un recorrido del usuario mediante una llamada a su propio servicio RESTful. Identity Experience Framework puede enviar y recibir datos del servicio RESTful para intercambiar notificaciones. Por ejemplo, puede:

- **Validar los datos de entrada del usuario**. Por ejemplo, puede verificar que la dirección de correo electrónico que proporciona el usuario existe en la base de datos del cliente y, en caso contrario, presentar un error.
- **Procesar notificaciones**. Si un usuario escribe su nombre todo con mayúsculas o minúsculas, la API de REST puede dar formato al nombre para que solo la primera letra esté en mayúscula y lo devuelva a Azure AD B2C.
- **Enriquecer datos de usuario mediante integración adicional en aplicaciones de línea de negocio corporativas**. el servicio RESTful puede recibir la dirección de correo electrónico del usuario, consultar la base de datos del cliente y devolver el número de fidelidad del usuario a Azure AD B2C. Luego, las notificaciones devueltas pueden almacenarse en la cuenta de Azure AD del usuario, evaluarse en los siguientes pasos de orquestación o incluirse en el token de acceso.
- **Ejecutar una lógica de negocios personalizada**. Puede enviar notificaciones push, actualizar las bases de datos corporativas, ejecutar un proceso de migración de usuarios, administrar permisos, auditar bases de datos y realizar otros flujos de trabajo.

![Diagrama de un intercambio de notificaciones del servicio RESTful](media/custom-policy-rest-api-intro/restful-service-claims-exchange.png)

> [!NOTE]
> Si el servicio RESTful es lento o no responde a Azure AD B2C, el tiempo de expiración es de 30 segundos y el número de reintentos es 2 veces (es decir, hay 3 intentos en total). La configuración de tiempo de expiración y el número de reintentos no es configurable actualmente.

## <a name="calling-a-restful-service"></a>Llamada a un servicio RESTful

La interacción incluye un intercambio de notificaciones de información entre las notificaciones de la API de REST y Azure AD B2C. La integración con los servicios REST se pueden diseñar de las siguientes maneras:

- **Perfil técnico de validación**. La llamada al servicio RESTful tiene lugar dentro de un [perfil técnico de validación](validation-technical-profile.md) del [perfil técnico autoafirmado](self-asserted-technical-profile.md) especificado o un [control de visualización de verificación](display-control-verification.md) de un [control de visualización](display-controls.md). El perfil técnico de validación valida los datos de que proporciona el usuario antes de que el recorrido del usuario avance. Con el perfil técnico de validación, puede realizar las siguientes tareas:

  - Enviar notificaciones a la API de REST.
  - Validar las notificaciones y generar mensajes de error personalizados que se muestren al usuario.
  - Devolver notificaciones de la API de REST a los pasos de orquestación posteriores.

- **Intercambio de notificaciones**. Un intercambio de notificaciones directo se puede configurar llamando a un perfil técnico de la API de REST directamente desde un paso de orquestación de un [recorrido del usuario](userjourneys.md). Esta definición se limita a:

  - Enviar notificaciones a la API de REST.
  - Validar las notificaciones y generar mensajes de error personalizados que se devuelvan a la aplicación.
  - Devolver notificaciones de la API de REST a los pasos de orquestación posteriores.

Puede agregar una llamada a la API REST en cualquier paso del recorrido del usuario definido por una directiva personalizada. Por ejemplo, puede llamar a una API REST:

- Durante el inicio de sesión, justo antes de que Azure AD B2C valide las credenciales.
- Inmediatamente después del inicio de sesión.
- Antes de que Azure AD B2C cree una cuenta en el directorio.
- Después de que Azure AD B2C cree una cuenta en el directorio.
- Antes de que Azure AD B2C emita un token de acceso.

![Colección de perfiles técnicos de validación](media/custom-policy-rest-api-intro/validation-technical-profile.png)

## <a name="sending-data"></a>Envío de datos

En el [perfil técnico de RESTful](restful-technical-profile.md), el elemento `InputClaims` contiene una lista de notificaciones que se enviarán al servicio RESTful. Puede asignar el nombre de la notificación al nombre definido en el servicio RESTful, establecer un valor predeterminado y usar [solucionadores de notificaciones](claim-resolver-overview.md).

Puede configurar la manera que las notificaciones de entrada se envían al proveedor de notificaciones de RESTful mediante el atributo SendClaimsIn. Los valores posibles son:

- **Body**, se envía en el cuerpo de la solicitud HTTP POST en formato JSON.
- **Form**, se envía en el cuerpo de la solicitud HTTP POST en un formato de valor de clave separado por "&" (Y comercial).
- **Header**, se envía en el encabezado de la solicitud HTTP GET.
- **QueryString**, se envía en la cadena de consulta de la solicitud HTTP GET.

Cuando la opción **Body** está configurada, el perfil técnico de la API de REST le permite enviar una carga de JSON compleja a un punto de conexión. Para obtener más información, consulte [Envío de una carga de JSON](restful-technical-profile.md#send-a-json-payload).

## <a name="receiving-data"></a>Recibir datos

El elemento `OutputClaims` del [perfil técnico de RESTful](restful-technical-profile.md) contiene una lista de las notificaciones devueltas por la API de REST. Es posible que tenga que asignar el nombre de la notificación definida en la directiva al nombre definido en la API REST. También puede incluir notificaciones que no devuelve el proveedor de identidades de la API de REST, siempre que establezca el atributo DefaultValue.

Las notificaciones de salida analizadas por el proveedor de notificaciones RESTful siempre esperan analizar una respuesta de Body de JSON plana, como:

```json
{
  "name": "Emily Smith",
  "email": "emily@outlook.com",
  "loyaltyNumber":  1234
}
```

Las notificaciones de salida deben tener un aspecto similar al siguiente:

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
  <OutputClaim ClaimTypeReferenceId="email" />
  <OutputClaim ClaimTypeReferenceId="loyaltyNumber" />
</OutputClaims>
```

Para analizar una respuesta de Body de JSON anidada, establezca los metadatos de ResolveJsonPathsInJsonTokens en true. En una notificación de salida, establezca PartnerClaimType en el elemento de la ruta de acceso JSON que quiere generar.

```json
"contacts": [
  {
    "id": "MAINCONTACT_1",
    "person": {
      "name": "Emily Smith",
      "loyaltyNumber":  1234,
      "emails": [
        {
          "id": "EMAIL_1",
          "type": "WORK",
          "email": "email@domain.com"
        }
      ]
    }
  }
],
```


Las notificaciones de salida deben tener un aspecto similar al siguiente:

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="contacts[0].person.name" />
  <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="contacts[0].person.emails[0].email" />
  <OutputClaim ClaimTypeReferenceId="loyaltyNumber" PartnerClaimType="contacts[0].person.loyaltyNumber" />
</OutputClaims>
```

## <a name="security-considerations"></a>Consideraciones sobre la seguridad

Debe proteger el punto de conexión de la API de REST para que solo los clientes autenticados puedan comunicarse con él. La API de REST debe usar un punto de conexión HTTPS. Establezca los metadatos de AuthenticationType en uno de los métodos de autenticación siguientes:

- El **certificado de cliente** restringe el acceso mediante una autenticación de certificados de cliente. Solo pueden acceder a la API los servicios que tengan los certificados adecuados. El certificado de cliente se almacena en una clave de directiva de Azure AD B2C. Obtenga más información sobre la [protección del servicio RESTful mediante certificados de cliente](secure-rest-api.md#https-client-certificate-authentication).
- **Básico** protege la API de REST con la autenticación HTTP básica. Solo los usuarios verificados, incluido Azure AD B2C, pueden acceder a la API. El nombre de usuario y la contraseña se almacenan en claves de directiva de Azure AD B2C. Obtenga información sobre la [protección de los servicios RESTful mediante la autenticación HTTP básica](secure-rest-api.md#http-basic-authentication).
- **Portador** restringe el acceso mediante un token de acceso de OAuth2 de cliente. El token de acceso se almacena en una clave de directiva de Azure AD B2C. Obtenga más información sobre la [protección del servicio RESTful mediante un token de portador](secure-rest-api.md#oauth2-bearer-authentication).

## <a name="rest-api-platform"></a>Plataforma de la API de REST
La API de REST puede basarse en cualquier plataforma y escribirse en cualquier lenguaje de programación, siempre y cuando esté protegida y pueda enviar y recibir notificaciones, tal como se especifica en el [perfil técnico de RESTful](restful-technical-profile.md).

## <a name="localize-the-rest-api"></a>Localización de la API de REST
En un perfil técnico de RESTful, puede querer enviar el idioma o la configuración regional de la sesión actual y, si es necesario, producir un mensaje de error localizado. Mediante el [solucionador de notificaciones](claim-resolver-overview.md), puede enviar una notificación contextual, como el idioma del usuario. En el ejemplo siguiente se muestra un perfil técnico de RESTful que muestra este escenario.

```xml
<TechnicalProfile Id="REST-ValidateUserData">
  <DisplayName>Validate user input data</DisplayName>
  <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.RestfulProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
  <Metadata>
    <Item Key="ServiceUrl">https://your-app.azurewebsites.net/api/identity</Item>
    <Item Key="AuthenticationType">None</Item>
    <Item Key="SendClaimsIn">Body</Item>
    <Item Key="IncludeClaimResolvingInClaimsHandling">true</Item>
  </Metadata>
  <InputClaims>
    <InputClaim ClaimTypeReferenceId="userLanguage" DefaultValue="{Culture:LCID}" AlwaysUseDefaultValue="true" />
    <InputClaim ClaimTypeReferenceId="email" PartnerClaimType="emailAddress" />
  </InputClaims>
  <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop" />
</TechnicalProfile>
```

## <a name="handling-error-messages"></a>Control de mensajes de error

Es posible la API de REST tenga que devolver un mensaje de error, como "No se encuentra el usuario en el sistema CRM". Si se produce un error, la API de REST debe devolver un mensaje de error HTTP 409 (código de estado de respuesta de conflicto). Para obtener más información, consulte [Perfil técnico de RESTful](restful-technical-profile.md#returning-validation-error-message).

Esto solo se puede lograr si se llama a un perfil técnico de la API de REST desde un perfil técnico de validación. Esto permite al usuario corregir los datos en la página y volver a ejecutar la validación tras el envío de la página.

Se requiere una respuesta HTTP 409 para evitar el procesamiento de los perfiles técnicos de validación posteriores dentro de este paso de orquestación.

Si hace referencia a un perfil técnico de la API de REST directamente desde un recorrido del usuario, se redirigirá al usuario a la aplicación de usuario de confianza con el mensaje de error correspondiente.

## <a name="publishing-your-rest-api"></a>Publicación de la API de REST

La solicitud al servicio de la API de REST procede de servidores de Azure AD B2C. El servicio de la API de REST se debe publicar en un punto de conexión HTTPS accesible públicamente. Las llamadas a la API de REST llegarán desde una dirección IP del centro de datos de Azure.

Diseñe el servicio de la API de REST y sus componentes subyacentes (como la base de datos y el sistema de archivos) para que tengan alta disponibilidad.

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para obtener ejemplos del uso de un perfil técnico de RESTful:

- [Tutorial: Integración de intercambios de notificaciones de API de REST en el recorrido del usuario de Azure AD B2C como validación de la entrada del usuario](custom-policy-rest-api-claims-validation.md)
- [Tutorial: Agregue los intercambios de notificaciones de la API de REST a directivas personalizadas de Azure Active Directory B2C](custom-policy-rest-api-claims-validation.md)
- [Protección de los servicios de la API de REST](secure-rest-api.md)
- [Referencia: Perfil técnico de RESTful](restful-technical-profile.md)
