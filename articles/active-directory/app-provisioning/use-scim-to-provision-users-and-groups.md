---
title: Desarrollo de un punto de conexión SCIM para el aprovisionamiento de usuarios en aplicaciones desde Azure AD
description: System for Cross-domain Identity Management (SCIM) normaliza el aprovisionamiento automático de usuarios. Aprenda a desarrollar un punto de conexión SCIM, integre la API de SCIM con Azure Active Directory y comience a automatizar el aprovisionamiento de usuarios y grupos en las aplicaciones en la nube.
services: active-directory
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: how-to
ms.date: 03/07/2020
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: a8138f125c55e3b2d76cb680ea48366c5a3e05fd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87051525"
---
# <a name="build-a-scim-endpoint-and-configure-user-provisioning-with-azure-ad"></a>Creación de un punto de conexión SCIM y configuración del aprovisionamiento de usuarios con Azure AD

Como desarrollador de aplicaciones, puede usar la API de administración de usuarios del sistema para la administración de identidades entre dominios (SCIM) para habilitar el aprovisionamiento automático de usuarios y grupos entre la aplicación y Azure AD. En este artículo se describe cómo crear un punto de conexión de SCIM e integrarlo con el servicio de aprovisionamiento de Azure AD. La especificación SCIM proporciona un esquema de usuario común para el aprovisionamiento. Cuando se usa junto con estándares de federación como SAML u OpenID Connect, SCIM ofrece a los administradores una solución de un extremo a otro basada en estándares para la administración del acceso.

SCIM es una definición estándar de dos puntos de conexión: /Users (Usuarios) y /Groups (Grupos). Utiliza verbos de REST comunes para crear, actualizar y eliminar objetos, y un esquema predefinido para atributos comunes como el nombre de grupo, nombre de usuario, nombre, apellidos y correo electrónico. Las aplicaciones que ofrecen una API REST de SCIM 2.0 pueden reducir o eliminar la molestia de trabajar con una API de administración de usuarios propia. Por ejemplo, cualquier cliente SCIM compatible sabe cómo crear una entrada HTTP POST de un objeto JSON para el punto de conexión /Users a fin de crear una nueva entrada de usuario. En lugar de necesitar una API ligeramente diferente para las mismas acciones básicas, las aplicaciones que cumplan con el estándar SCIM pueden aprovechar al instante los clientes, herramientas y código ya existentes. 

![Aprovisionamiento desde Azure AD a una aplicación con SCIM](media/use-scim-to-provision-users-and-groups/scim-provisioning-overview.png)

El esquema de objetos de usuario estándar y las API REST para administración definidas en SCIM 2.0 (RFC [7642](https://tools.ietf.org/html/rfc7642), [7643](https://tools.ietf.org/html/rfc7643), [7644](https://tools.ietf.org/html/rfc7644)) permiten que los proveedores de identidades y las aplicaciones se integren más fácilmente entre sí. Los desarrolladores de aplicaciones que crean un punto de conexión SCIM se pueden integrar con cualquier cliente compatible con SCIM sin tener que realizar ningún trabajo personalizado.

Para automatizar el aprovisionamiento de una aplicación, es necesario crear e integrar un punto de conexión SCIM con el cliente SCIM de Azure AD. Realice los pasos siguientes para iniciar el aprovisionamiento de usuarios y grupos en la aplicación. 
    
  * **[Paso 1: Diseñe el esquema de grupos y usuarios.](#step-1-design-your-user-and-group-schema)** Identifique los objetos y atributos que necesite la aplicación, y determine cómo se asignan al esquema de usuarios y de grupos que admite la implementación de SCIM de Azure AD.

  * **[Paso 2: Información sobre la implementación de SCIM de Azure AD.](#step-2-understand-the-azure-ad-scim-implementation)** Averigüe cómo se implementa el cliente de SCIM de Azure AD y modele las respuestas y el control de solicitudes del protocolo SCIM.

  * **[Paso 3: Cree un punto de conexión SCIM.](#step-3-build-a-scim-endpoint)** Un punto de conexión debe ser compatible con SCIM 2.0 para integrarse con el servicio de aprovisionamiento de Azure AD. Si lo desea, puede usar las bibliotecas de Microsoft Common Language Infrastructure (CLI) y los ejemplos de código para crear el punto de conexión. Estos ejemplos se ofrecen solo como referencia y para realizar pruebas; se recomienda no codificar la aplicación de producción de modo que dependa de ellos.

  * **[Paso 4: Integre el punto de conexión SCIM con el cliente de SCIM de Azure AD.](#step-4-integrate-your-scim-endpoint-with-the-azure-ad-scim-client)** Si la organización usa una aplicación de terceros que implementa el perfil de SCIM 2.0 que admite Azure AD, puede empezar a automatizar tanto el aprovisionamiento como el desaprovisionamiento de usuarios y grupos de forma inmediata.

  * **[Paso 5: Publique su aplicación en la galería de aplicaciones de Azure AD.](#step-5-publish-your-application-to-the-azure-ad-application-gallery)** Facilite a los clientes la detección de la aplicación y la configuración del aprovisionamiento. 

![Pasos para la integración de un punto de conexión SCIM con Azure AD](media/use-scim-to-provision-users-and-groups/process.png)

## <a name="step-1-design-your-user-and-group-schema"></a>Paso 1: Diseñe el esquema de grupos y usuarios

Cada aplicación requiere atributos diferentes para crear un usuario o un grupo. Para iniciar la integración, identifique los objetos (usuarios, grupos) y atributos (nombre, administrador, puesto, etc.) que la aplicación requiera. El estándar SCIM define un esquema para administrar usuarios y grupos. El esquema de usuario principal solo requiere tres atributos: **id** (identificador definido por el proveedor de servicios), **externalId** (identificador definido por el cliente) y **meta** (metadatos de solo lectura que mantiene el proveedor de servicios). El resto de atributos son opcionales. Además del esquema de usuario principal, el estándar SCIM define una extensión de usuario empresarial y un modelo para extender el esquema de usuario para satisfacer las necesidades de su aplicación. Si, por ejemplo, la aplicación requiere un administrador de usuario, puede usar el esquema de usuario empresarial para recopilar el administrador del usuario y el esquema principal para recopilar el correo electrónico del usuario. Para diseñar su esquema, siga estos pasos:
  1. Enumere los atributos que requiere la aplicación. Puede ser útil dividir sus requisitos en los atributos necesarios para la autenticación (por ejemplo, loginName y email), los atributos necesarios para administrar el ciclo de vida del usuario (por ejemplo, status o active) y otros atributos necesarios para que funcione la aplicación en particular (por ejemplo, manager, tag).
  2. Compruebe si esos atributos ya están definidos en el esquema de usuario principal o en el esquema de usuario empresarial. Si los atributos que necesita no se cubren en los esquemas de usuario principal o empresarial, deberá definir una extensión para el esquema de usuario que cubra los atributos que necesita. En el ejemplo siguiente, hemos agregado una extensión al usuario para permitir el aprovisionamiento de un atributo "tag" en un usuario. Lo mejor es comenzar con los esquemas de usuario principal y empresarial, y ampliarlos con esquemas personalizados más adelante.  
  3. Asigne los atributos SCIM a los atributos de usuario en Azure AD. Si uno de los atributos que ha definido en el punto de conexión de SCIM no tiene un homólogo claro en el esquema de usuario de Azure AD, existe la posibilidad de que los datos no se almacenen en el objeto de usuario en la mayoría de los inquilinos. Considere si este atributo puede ser opcional para crear un usuario. Si el atributo es fundamental para que la aplicación funcione, guíe al administrador de inquilinos para que amplíe su esquema o use un atributo de extensión, como se muestra a continuación para la propiedad "tags".

### <a name="table-1-outline-the-attributes-that-you-need"></a>Tabla 1: Definición de un esquema de los atributos que necesita 
| Paso 1: Determinación de los atributos que requiere la aplicación| Paso 2: Asignación de los requisitos de la aplicación al estándar SCIM| Paso 3: Asignación de los atributos SCIM a los atributos de Azure AD|
|--|--|--|
|loginName|userName|userPrincipalName|
|firstName|name.givenName|givenName|
|lastName|name.lastName|lastName|
|workMail|Emails[type eq “work”].value|Correo|
|manager|manager|manager|
|etiqueta|urn:ietf:params:scim:schemas:extension:2.0:CustomExtension:tag|extensionAttribute1|
|status|active|isSoftDeleted (valor calculado no almacenado en el usuario)|

El esquema definido anteriormente se representaría mediante la carga JSON siguiente. Tenga en cuenta que, además de los atributos necesarios para la aplicación, la representación JSON incluye los atributos `id`, `externalId` y `meta` necesarios.

```json
{
     "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User",
      "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User",
      "urn:ietf:params:scim:schemas:extension:CustomExtensionName:2.0:User"],
     "userName":"bjensen",
     "externalId":"bjensen",
     "name":{
       "familyName":"Jensen",
       "givenName":"Barbara"
     },
     "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User": {
     "Manager": "123456"
   },
     "urn:ietf:params:scim:schemas:extension:CustomExtensionName:2.0:CustomAttribute:User": {
     "tag": "701984",
   },
   "meta": {
     "resourceType": "User",
     "created": "2010-01-23T04:56:22Z",
     "lastModified": "2011-05-13T04:42:34Z",
     "version": "W\/\"3694e05e9dff591\"",
     "location":
 "https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646"
   }
 ```

### <a name="table-2-default-user-attribute-mapping"></a>Tabla 2: Asignación de atributos de usuario predeterminada
A continuación, puede usar la tabla siguiente para entender cómo los atributos de su aplicación se pueden asignar a un atributo en Azure AD y en el RFC de SCIM. Puede [personalizar](customize-application-attributes.md) el modo en que los atributos se asignan entre Azure AD y el punto de conexión SCIM. Tenga en cuenta que no es necesario que admita los usuarios ni los grupos, ni todos los atributos que se muestran a continuación. Son una referencia para el modo en que los atributos de Azure AD se suelen asignar a las propiedades del protocolo SCIM. 

| Usuario de Azure Active Directory | "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User" |
| --- | --- |
| IsSoftDeleted |active |
|department|urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:department|
| DisplayName |DisplayName |
|employeeId|urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:employeeNumber|
| Facsimile-TelephoneNumber |phoneNumbers[type eq "fax"].value |
| givenName |name.givenName |
| jobTitle |title |
| mail |emails[type eq "work"].value |
| mailNickname |externalId |
| manager |urn:ietf:params:scim:schemas:extension:enterprise:2.0:User:manager |
| mobile |phoneNumbers[type eq "mobile"].value |
| postalCode |addresses[type eq "work"].postalCode |
| proxy-Addresses |emails[type eq "other"].Value |
| physical-Delivery-OfficeName |addresses[type eq "other"].Formatted |
| streetAddress |addresses[type eq "work"].streetAddress |
| surname |name.familyName |
| telephone-Number |phoneNumbers[type eq "work"].value |
| user-PrincipalName |userName |


### <a name="table-3-default-group-attribute-mapping"></a>Tabla 3: Asignación de atributos de grupo predeterminada

| Grupo de Azure Active Directory | urn:ietf:params:scim:schemas:core:2.0:Group |
| --- | --- |
| DisplayName |DisplayName |
| mail |emails[type eq "work"].value |
| mailNickname |DisplayName |
| members |members |
| objectId |externalId |
| proxyAddresses |emails[type eq "other"].Value |

Hay varios puntos de conexión definidos en el RFC de SCIM. Puede empezar con el punto de conexión /User y después ampliarlo desde allí. El punto de conexión /Schemas es útil cuando se usan atributos personalizados o si el esquema cambia con frecuencia. Permite a un cliente recuperar el esquema más reciente automáticamente. El punto de conexión /Bulk es especialmente útil cuando se admiten grupos. En la tabla siguiente se describen los distintos puntos de conexión definidos en el estándar SCIM. El punto de conexión /Schemas es útil cuando se usan atributos personalizados o si el esquema cambia con frecuencia. Permite a un cliente recuperar el esquema más reciente automáticamente. El punto de conexión /Bulk es especialmente útil cuando se admiten grupos. En la tabla siguiente se describen los distintos puntos de conexión definidos en el estándar SCIM. 
 
### <a name="table-4-determine-the-endpoints-that-you-would-like-to-develop"></a>Tabla 4: Determinación de los puntos de conexión que quiere desarrollar
|ENDPOINT|DESCRIPTION|
|--|--|
|/User|Realiza operaciones CRUD en un objeto de usuario.|
|/Group|Realiza operaciones CRUD en un objeto de grupo.|
|/ServiceProviderConfig|Proporciona detalles sobre las características del estándar SCIM que se admiten; por ejemplo, los recursos que se admiten y el método de autenticación.|
|/ResourceTypes|Especifica los metadatos de cada recurso.|
|/Schemas|El conjunto de atributos que admite cada cliente y proveedor de servicios puede variar. Un proveedor de servicios puede incluir `name`, `title` y `emails`, mientras que otro usa `name`, `title` y `phoneNumbers`. El punto de conexión de esquemas permite la detección de los atributos admitidos.|
|/Bulk|Las operaciones masivas le permiten realizar operaciones en una gran colección de objetos de recursos en una sola operación (por ejemplo, actualizar las pertenencias para un grupo de gran tamaño).|


## <a name="step-2-understand-the-azure-ad-scim-implementation"></a>Paso 2: Información sobre la implementación de SCIM de Azure AD
> [!IMPORTANT]
> El comportamiento de la implementación de SCIM de Azure AD se actualizó por última vez el 18 de diciembre de 2018. Para obtener información sobre los cambios que se han producido, consulte [SCIM 2.0 protocol compliance of the Azure AD User Provisioning service](application-provisioning-config-problem-scim-compatibility.md) (Cumplimiento del protocolo SCIM 2.0 del servicio de aprovisionamiento de usuarios de Azure AD).

Si está creando una aplicación que admite una API de administración de usuario de SCIM 2.0, en esta sección se describe en detalle cómo se implementa el cliente de SCIM de Azure AD. También se muestra cómo modelar las respuestas y el control de solicitudes del protocolo SCIM. Una vez que haya implementado el punto de conexión SCIM, puede probarlo con el procedimiento descrito en la sección anterior.

Dentro de la [especificación del protocolo SCIM 2.0](http://www.simplecloud.info/#Specification), la aplicación tiene que cumplir estos requisitos:

* Admitir la creación de usuarios y de forma opcional también de grupos, según la sección [3.3 del protocolo SCIM](https://tools.ietf.org/html/rfc7644#section-3.3).  
* Admitir la modificación de usuarios o grupos con solicitudes PATCH, según la [sección 3.5.2 del protocolo SCIM](https://tools.ietf.org/html/rfc7644#section-3.5.2).  
* Admitir la recuperación de un recurso conocido, para un usuario o un grupo creado anteriormente, según la [sección 3.4.1 del protocolo SCIM](https://tools.ietf.org/html/rfc7644#section-3.4.1).  
* Admitir la consulta de usuarios o grupos, según la sección [3.4.2 del protocolo SCIM](https://tools.ietf.org/html/rfc7644#section-3.4.2).  De forma predeterminada, los usuarios se recuperan por sus `id` y se consultan por sus `username` y `externalId`, y los grupos por su `displayName`.  
* Admitir la consulta de usuarios por identificador y por administrador, según la sección 3.4.2 del protocolo SCIM.  
* Admitir la consulta de grupos por Id. y miembro, según la sección 3.4.2 del protocolo SCIM.  
* Acepta un token de portador único para la autenticación y autorización de Azure AD para la aplicación.

Siga estas directrices generales al implementar un punto de conexión SCIM para garantizar la compatibilidad con Azure AD:

* `id` es una propiedad obligatoria para todos los recursos. Todas las respuestas que devuelve un recurso deben asegurarse de que cada recurso tiene esta propiedad, excepto `ListResponse` con cero miembros.
* La respuesta a una solicitud de consulta o filtrado siempre debe ser una `ListResponse`.
* Los grupos son opcionales, pero solo se admiten si la implementación de SCIM admite las solicitudes PATCH.
* No es necesario incluir todo el recurso completo en la respuesta PATCH.
* Microsoft Azure AD solo usa los siguientes operadores:  
    - `eq`
    - `and`
* No solicite una coincidencia entre mayúsculas y minúsculas en los elementos estructurales de SCIM, en particular en los valores de operación PATCH `op`, como se define en https://tools.ietf.org/html/rfc7644#section-3.5.2. Azure AD emite los valores de "op" como `Add`, `Replace`, y `Remove`.
* Microsoft Azure AD realiza solicitudes para recuperar un usuario y un grupo aleatorio para asegurarse de que el punto de conexión y las credenciales son válidas. También se realiza como parte del flujo de la **Prueba de conexión** en [Azure Portal](https://portal.azure.com). 
* El atributo sobre el que se pueden consultar los recursos debe establecerse como un atributo coincidente en la aplicación en [Azure Portal](https://portal.azure.com). Para más información, consulte [Personalización de asignaciones de atributos de aprovisionamiento de usuarios](customize-application-attributes.md)

### <a name="user-provisioning-and-deprovisioning"></a>Aprovisionamiento y desaprovisionamiento de usuarios

La siguiente ilustración muestra los mensajes que Azure Active Directory envía a un servicio SCIM para administrar el ciclo de vida de un usuario en su almacén de identidades.  

![Muestra la secuencia de aprovisionamiento y desaprovisionamiento de usuarios](media/use-scim-to-provision-users-and-groups/scim-figure-4.png)<br/>
*Secuencia de aprovisionamiento y desaprovisionamiento de usuarios*

### <a name="group-provisioning-and-deprovisioning"></a>Aprovisionamiento y desaprovisionamiento de grupos

El aprovisionamiento y desaprovisionamiento de grupos son opcionales. Cuando se implementa y habilita, la siguiente ilustración muestra los mensajes que Azure AD envía a un servicio SCIM para administrar el ciclo de vida de un usuario en el almacén de identidades de la aplicación.  Esos mensajes se diferencian de los mensajes sobre los usuarios de dos maneras:

* Las solicitudes para recuperar grupos especifican que el atributo members se excluya de cualquier recurso proporcionado en respuesta a la solicitud.  
* Las solicitudes para determinar si un atributo de referencia tiene un valor determinado son solicitudes sobre el atributo members.  

![Muestra la secuencia de aprovisionamiento y desaprovisionamiento de grupos](media/use-scim-to-provision-users-and-groups/scim-figure-5.png)<br/>
*Secuencia de aprovisionamiento y desaprovisionamiento de grupos*

### <a name="scim-protocol-requests-and-responses"></a>Modificación de solicitudes y respuestas de protocolo SCIM
Esta sección proporciona solicitudes SCIM de ejemplo emitidas por el cliente de SCIM de Azure AD y el ejemplo respuestas esperadas. Para obtener mejores resultados, debe codificar la aplicación para controlar estas solicitudes en este formato y emitir las respuestas esperadas.

> [!IMPORTANT]
> Para comprender cómo y cuándo el servicio de aprovisionamiento de usuarios de Azure AD emite las operaciones que se describen a continuación, consulte [Ciclos de aprovisionamiento: inicial e incremental](how-provisioning-works.md#provisioning-cycles-initial-and-incremental) en [Funcionamiento del aprovisionamiento](how-provisioning-works.md).

[Operaciones de usuario](#user-operations)
  - [Crear usuario](#create-user) ([Solicitud](#request) / [Respuesta](#response))
  - [Obtener usuario](#get-user) ([Solicitud](#request-1) / [Respuesta](#response-1))
  - [Obtener usuario por consulta](#get-user-by-query) ([Solicitud](#request-2) / [Respuesta](#response-2))
  - [Obtener usuario por consulta: cero resultados](#get-user-by-query---zero-results) ([Solicitud](#request-3)
/ [Respuesta](#response-3))
  - [Actualizar usuario [propiedades con varios valores]](#update-user-multi-valued-properties) ([Solicitud](#request-4) /  [Respuesta](#response-4))
  - [Actualizar usuario [propiedades con un solo valor]](#update-user-single-valued-properties) ([Solicitud](#request-5)
/ [Respuesta](#response-5)) 
  - [Deshabilitar usuario](#disable-user) ([Solicitud](#request-14) / 
[Respuesta](#response-14))
  - [Eliminar usuario](#delete-user) ([Solicitud](#request-6) / 
[Respuesta](#response-6))


[Operaciones de grupo](#group-operations)
  - [Crear grupo](#create-group) ([Solicitud](#request-7) / [Respuesta](#response-7))
  - [Obtener grupo](#get-group) ([Solicitud](#request-8) / [Respuesta](#response-8))
  - [Obtener grupo por displayName](#get-group-by-displayname) ([Solicitud](#request-9) / [Respuesta](#response-9))
  - [Actualizar grupo [Atributos no de miembro]](#update-group-non-member-attributes) ([Solicitud](#request-10) /
 [Respuesta](#response-10))
  - [Actualizar grupo [Agregar miembros]](#update-group-add-members) ([Solicitud](#request-11) /
[Respuesta](#response-11))
  - [Actualizar grupo [Quitar miembros]](#update-group-remove-members) ( [Solicitud](#request-12) /
[Respuesta](#response-12))
  - [Eliminar grupo](#delete-group) ([Solicitud](#request-13) /
[Respuesta](#response-13))

### <a name="user-operations"></a>Operaciones de usuario

* Los usuarios pueden ser consultados por atributos `userName` o `email[type eq "work"]`.  

#### <a name="create-user"></a>Crear usuario

###### <a name="request"></a>Solicitud

*POST /Users*
```json
{
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User",
        "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"],
    "externalId": "0a21f0f2-8d2a-4f8e-bf98-7363c4aed4ef",
    "userName": "Test_User_ab6490ee-1e48-479e-a20b-2d77186b5dd1",
    "active": true,
    "emails": [{
        "primary": true,
        "type": "work",
        "value": "Test_User_fd0ea19b-0777-472c-9f96-4f70d2226f2e@testuser.com"
    }],
    "meta": {
        "resourceType": "User"
    },
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName"
    },
    "roles": []
}
```

##### <a name="response"></a>Response

*HTTP/1.1 201 Created*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "48af03ac28ad4fb88478",
    "externalId": "0a21f0f2-8d2a-4f8e-bf98-7363c4aed4ef",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "userName": "Test_User_ab6490ee-1e48-479e-a20b-2d77186b5dd1",
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName",
    },
    "active": true,
    "emails": [{
        "value": "Test_User_fd0ea19b-0777-472c-9f96-4f70d2226f2e@testuser.com",
        "type": "work",
        "primary": true
    }]
}
```

#### <a name="get-user"></a>Obtener usuario

###### <a name="request"></a><a name="request-1"></a>Solicitud
*GET /Users/5d48a0a8e9f04aa38008* 

###### <a name="response-user-found"></a><a name="response-1"></a>Respuesta (Se encontró el usuario)
*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "5d48a0a8e9f04aa38008",
    "externalId": "58342554-38d6-4ec8-948c-50044d0a33fd",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "userName": "Test_User_feed3ace-693c-4e5a-82e2-694be1b39934",
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName",
    },
    "active": true,
    "emails": [{
        "value": "Test_User_22370c1a-9012-42b2-bf64-86099c2a1c22@testuser.com",
        "type": "work",
        "primary": true
    }]
}
```

###### <a name="request"></a>Solicitud
*GET /Users/5171a35d82074e068ce2* 

###### <a name="response-user-not-found-note-that-the-detail-is-not-required-only-status"></a>Respuesta (No se encontró el usuario. Observe que no se requiere el detalle, solo el estado).

```json
{
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:Error"
    ],
    "status": "404",
    "detail": "Resource 23B51B0E5D7AE9110A49411D@7cca31655d49f3640a494224 not found"
}
```

#### <a name="get-user-by-query"></a>Obtener usuario por consulta

##### <a name="request"></a><a name="request-2"></a>Solicitud

*GET /Users?filter=userName eq "Test_User_dfeef4c5-5681-4387-b016-bdf221e82081"*

##### <a name="response"></a><a name="response-2"></a>Respuesta

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
    "totalResults": 1,
    "Resources": [{
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
        "id": "2441309d85324e7793ae",
        "externalId": "7fce0092-d52e-4f76-b727-3955bd72c939",
        "meta": {
            "resourceType": "User",
            "created": "2018-03-27T19:59:26.000Z",
            "lastModified": "2018-03-27T19:59:26.000Z"
            
        },
        "userName": "Test_User_dfeef4c5-5681-4387-b016-bdf221e82081",
        "name": {
            "familyName": "familyName",
            "givenName": "givenName"
        },
        "active": true,
        "emails": [{
            "value": "Test_User_91b67701-697b-46de-b864-bd0bbe4f99c1@testuser.com",
            "type": "work",
            "primary": true
        }]
    }],
    "startIndex": 1,
    "itemsPerPage": 20
}

```

#### <a name="get-user-by-query---zero-results"></a>Obtener usuario por consulta: cero resultados

##### <a name="request"></a><a name="request-3"></a>Solicitud

*GET /Users?filter=userName eq "non-existent user"*

##### <a name="response"></a><a name="response-3"></a>Respuesta

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
    "totalResults": 0,
    "Resources": [],
    "startIndex": 1,
    "itemsPerPage": 20
}

```

#### <a name="update-user-multi-valued-properties"></a>Actualizar usuario [propiedades con varios valores]

##### <a name="request"></a><a name="request-4"></a>Solicitud

*PATCH /Users/6764549bef60420686bc HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [
            {
            "op": "Replace",
            "path": "emails[type eq \"work\"].value",
            "value": "updatedEmail@microsoft.com"
            },
            {
            "op": "Replace",
            "path": "name.familyName",
            "value": "updatedFamilyName"
            }
    ]
}
```

##### <a name="response"></a><a name="response-4"></a>Respuesta

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "6764549bef60420686bc",
    "externalId": "6c75de36-30fa-4d2d-a196-6bdcdb6b6539",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "userName": "Test_User_fbb9dda4-fcde-4f98-a68b-6c5599e17c27",
    "name": {
        "formatted": "givenName updatedFamilyName",
        "familyName": "updatedFamilyName",
        "givenName": "givenName"
    },
    "active": true,
    "emails": [{
        "value": "updatedEmail@microsoft.com",
        "type": "work",
        "primary": true
    }]
}
```

#### <a name="update-user-single-valued-properties"></a>Actualizar usuario [propiedades de un solo valor]

##### <a name="request"></a><a name="request-5"></a>Solicitud

*PATCH /Users/5171a35d82074e068ce2 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Replace",
        "path": "userName",
        "value": "5b50642d-79fc-4410-9e90-4c077cdd1a59@testuser.com"
    }]
}
```

##### <a name="response"></a><a name="response-5"></a>Respuesta

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
    "id": "5171a35d82074e068ce2",
    "externalId": "aa1eca08-7179-4eeb-a0be-a519f7e5cd1a",
    "meta": {
        "resourceType": "User",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
        
    },
    "userName": "5b50642d-79fc-4410-9e90-4c077cdd1a59@testuser.com",
    "name": {
        "formatted": "givenName familyName",
        "familyName": "familyName",
        "givenName": "givenName",
    },
    "active": true,
    "emails": [{
        "value": "Test_User_49dc1090-aada-4657-8434-4995c25a00f7@testuser.com",
        "type": "work",
        "primary": true
    }]
}
```

### <a name="disable-user"></a>Deshabilitar usuario

##### <a name="request"></a><a name="request-14"></a>Solicitud

*PATCH /Users/5171a35d82074e068ce2 HTTP/1.1*
```json
{
    "Operations": [
        {
            "op": "Replace",
            "path": "active",
            "value": false
        }
    ],
    "schemas": [
        "urn:ietf:params:scim:api:messages:2.0:PatchOp"
    ]
}
```

##### <a name="response"></a><a name="response-14"></a>Respuesta

```json
{
    "schemas": [
        "urn:ietf:params:scim:schemas:core:2.0:User"
    ],
    "id": "CEC50F275D83C4530A495FCF@834d0e1e5d8235f90a495fda",
    "userName": "deanruiz@testuser.com",
    "name": {
        "familyName": "Harris",
        "givenName": "Larry"
    },
    "active": false,
    "emails": [
        {
            "value": "gloversuzanne@testuser.com",
            "type": "work",
            "primary": true
        }
    ],
    "addresses": [
        {
            "country": "ML",
            "type": "work",
            "primary": true
        }
    ],
    "meta": {
        "resourceType": "Users",
        "location": "/scim/5171a35d82074e068ce2/Users/CEC50F265D83B4530B495FCF@5171a35d82074e068ce2"
    }
}
```
#### <a name="delete-user"></a>Eliminar usuario

##### <a name="request"></a><a name="request-6"></a>Solicitud

*DELETE /Users/5171a35d82074e068ce2 HTTP/1.1*

##### <a name="response"></a><a name="response-6"></a>Respuesta

*HTTP/1.1 204 No Content*

### <a name="group-operations"></a>Operaciones de grupo

* Los grupos deben crearse siempre con una lista de miembros vacía.
* Los grupos pueden ser consultados por el atributo `displayName`.
* La actualización de la solicitud de PATCH del grupo debe producir un *HTTP 204 No Content* en la respuesta. La devolución de un cuerpo con una lista de todos los miembros no es aconsejable.
* No es necesario admitir la devolución de todos los miembros del grupo.

#### <a name="create-group"></a>Crear grupo

##### <a name="request"></a><a name="request-7"></a>Solicitud

*POST /Groups HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group", "http://schemas.microsoft.com/2006/11/ResourceManagement/ADSCIM/2.0/Group"],
    "externalId": "8aa1a0c0-c4c3-4bc0-b4a5-2ef676900159",
    "displayName": "displayName",
    "meta": {
        "resourceType": "Group"
    }
}
```

##### <a name="response"></a><a name="response-7"></a>Respuesta

*HTTP/1.1 201 Created*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
    "id": "927fa2c08dcb4a7fae9e",
    "externalId": "8aa1a0c0-c4c3-4bc0-b4a5-2ef676900159",
    "meta": {
        "resourceType": "Group",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
        
    },
    "displayName": "displayName",
    "members": []
}
```

#### <a name="get-group"></a>Obtener grupo

##### <a name="request"></a><a name="request-8"></a>Solicitud

*GET /Groups/40734ae655284ad3abcc?excludedAttributes=members HTTP/1.1*

##### <a name="response"></a><a name="response-8"></a>Respuesta
*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
    "id": "40734ae655284ad3abcc",
    "externalId": "60f1bb27-2e1e-402d-bcc4-ec999564a194",
    "meta": {
        "resourceType": "Group",
        "created": "2018-03-27T19:59:26.000Z",
        "lastModified": "2018-03-27T19:59:26.000Z"
    },
    "displayName": "displayName",
}
```

#### <a name="get-group-by-displayname"></a>Obtener grupo por displayName

##### <a name="request"></a><a name="request-9"></a>Solicitud
*GET /Groups?excludedAttributes=members&filter=displayName eq "displayName" HTTP/1.1*

##### <a name="response"></a><a name="response-9"></a>Respuesta

*HTTP/1.1 200 OK*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
    "totalResults": 1,
    "Resources": [{
        "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
        "id": "8c601452cc934a9ebef9",
        "externalId": "0db508eb-91e2-46e4-809c-30dcbda0c685",
        "meta": {
            "resourceType": "Group",
            "created": "2018-03-27T22:02:32.000Z",
            "lastModified": "2018-03-27T22:02:32.000Z",
            
        },
        "displayName": "displayName",
    }],
    "startIndex": 1,
    "itemsPerPage": 20
}
```

#### <a name="update-group-non-member-attributes"></a>Actualizar grupo [atributos no de miembro]

##### <a name="request"></a><a name="request-10"></a>Solicitud

*PATCH /Groups/fa2ce26709934589afc5 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Replace",
        "path": "displayName",
        "value": "1879db59-3bdf-4490-ad68-ab880a269474updatedDisplayName"
    }]
}
```

##### <a name="response"></a><a name="response-10"></a>Respuesta

*HTTP/1.1 204 No Content*

### <a name="update-group-add-members"></a>Actualizar grupo [Agregar miembros]

##### <a name="request"></a><a name="request-11"></a>Solicitud

*PATCH /Groups/a99962b9f99d4c4fac67 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Add",
        "path": "members",
        "value": [{
            "$ref": null,
            "value": "f648f8d5ea4e4cd38e9c"
        }]
    }]
}
```

##### <a name="response"></a><a name="response-11"></a>Respuesta

*HTTP/1.1 204 No Content*

#### <a name="update-group-remove-members"></a>Actualizar grupo [Eliminar miembros]

##### <a name="request"></a><a name="request-12"></a>Solicitud

*PATCH /Groups/a99962b9f99d4c4fac67 HTTP/1.1*
```json
{
    "schemas": ["urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations": [{
        "op": "Remove",
        "path": "members",
        "value": [{
            "$ref": null,
            "value": "f648f8d5ea4e4cd38e9c"
        }]
    }]
}
```

##### <a name="response"></a><a name="response-12"></a>Respuesta

*HTTP/1.1 204 No Content*

#### <a name="delete-group"></a>Eliminar grupo

##### <a name="request"></a><a name="request-13"></a>Solicitud

*DELETE /Groups/cdb1ce18f65944079d37 HTTP/1.1*

##### <a name="response"></a><a name="response-13"></a>Respuesta

*HTTP/1.1 204 No Content*

### <a name="security-requirements"></a>Requisitos de seguridad
**Versiones del protocolo SSL**

Las únicas versiones aceptables del protocolo TLS son TLS 1.2 y TLS 1.3. No se permiten otras versiones de TLS. No se permite ninguna versión de SSL. 
- Las claves RSA deben ser de al menos 2048 bits.
- Las claves ECC deben ser de al menos 256 bits, generadas mediante una curva elíptica aprobada.


**Longitud de las claves**

Todos los servicios deben usar certificados X.509 generados con claves criptográficas de una longitud suficiente, lo que significa:

**Conjuntos de cifrado**

Todos los servicios deben configurarse para usar los siguientes conjuntos de cifrado, en el orden exacto que se especifica a continuación. Tenga en cuenta que si solo tiene un certificado RSA instalado, los conjuntos de cifrado ECDSA no tienen ningún efecto. </br>

Barra mínima de conjuntos de cifrado TLS 1.2:

- TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384

### <a name="ip-ranges"></a>Intervalos IP
Actualmente, el servicio de aprovisionamiento de Azure AD puede funcionar con cualquier intervalo de direcciones IP de Azure. Estamos trabajando para consolidar el conjunto de intervalos IP en los que opera el servicio. Este documento se actualizará una vez que la lista de intervalos IP esté consolidada. 

## <a name="step-3-build-a-scim-endpoint"></a>Paso 3: Cree un punto de conexión SCIM

Ahora que ha diseñado el esquema y comprendido la implementación de SCIM de Azure AD, puede empezar a desarrollar el punto de conexión de SCIM. En lugar de comenzar desde cero y compilar la implementación totalmente por su cuenta, puede confiar en una serie de bibliotecas de SCIM de código abierto publicadas por la comunidad de SCIM.

El [código de referencia](https://aka.ms/SCIMReferenceCode) de .NET Core de código abierto publicado por el equipo de aprovisionamiento de Azure AD es uno de esos recursos que puede iniciar su desarrollo. Una vez que haya creado el punto de conexión de SCIM, querrá probarlo. Puede usar la colección de [pruebas de Postman](https://github.com/AzureAD/SCIMReferenceCode/wiki/Test-Your-SCIM-Endpoint) proporcionadas como parte del código de referencia o ejecutar las solicitudes o respuestas de ejemplo proporcionadas [anteriormente](https://docs.microsoft.com/azure/active-directory/app-provisioning/use-scim-to-provision-users-and-groups#user-operations).  

   > [!Note]
   > El código de referencia está pensado para ayudarle a empezar a crear el punto de conexión de SCIM y se proporciona "tal cual". Las contribuciones de la comunidad le ayudarán a crear y mantener el código.

La solución se compone de dos proyectos: _Microsoft.SCIM_ y _Microsoft.SCIM.WebHostSample_.

El proyecto _Microsoft.SCIM_ es la biblioteca que define los componentes del servicio web que se ajusta a la especificación SCIM. Declara la interfaz _Microsoft.SCIM.IProvider_. Las solicitudes se traducen en llamadas a los métodos del proveedor, que se programan para funcionar en un almacén de identidades.

![Desglose: Una solicitud traducida en llamadas a los métodos del proveedor](media/use-scim-to-provision-users-and-groups/scim-figure-3.png)

El proyecto _Microsoft.SCIM.WebHostSample_ es una aplicación web ASP.NET Core de Visual Studio basada en la plantilla _Empty_. Esto permite implementar el código de ejemplo como independiente, hospedado en contenedores o en Internet Information Services. También implementa la interfaz _Microsoft.SCIM.IProvider_, que mantiene las clases en memoria como almacén de identidades de ejemplo.

```csharp
    public class Startup
    {
        ...
        public IMonitor MonitoringBehavior { get; set; }
        public IProvider ProviderBehavior { get; set; }

        public Startup(IWebHostEnvironment env, IConfiguration configuration)
        {
            ...
            this.MonitoringBehavior = new ConsoleMonitor();
            this.ProviderBehavior = new InMemoryProvider();
        }
        ...
```

### <a name="building-a-custom-scim-endpoint"></a>Creación de un punto de conexión SCIM personalizado

El servicio SCIM debe tener una dirección HTTP y un certificado de autenticación de servidor para el que la entidad de certificación raíz tenga uno de los siguientes nombres:

* CNNIC
* Comodo
* CyberTrust
* DigiCert
* GeoTrust
* GlobalSign
* Go Daddy
* VeriSign
* WoSign

El SDK de .NET Core incluye un certificado de desarrollo HTTPS que se puede usar durante el desarrollo. El certificado se instala como parte de la experiencia de primera ejecución. En función de cómo ejecute la aplicación web ASP.NET Core, escuchará en un puerto diferente:

* Microsoft.SCIM.WebHostSample: https://localhost:5001
* IIS Express: https://localhost:44359/

Para obtener más información sobre HTTPS en ASP.NET Core, use el siguiente vínculo: [Aplicar HTTPS en ASP.NET Core](https://docs.microsoft.com/aspnet/core/security/enforcing-ssl)

### <a name="handling-endpoint-authentication"></a>Control de la autenticación de puntos de conexión

Las solicitudes de Azure Active Directory incluyen un token de portador de OAuth 2.0. Cualquier servicio que reciba la solicitud debe autenticar al emisor como Azure Active Directory para el inquilino de Azure Active Directory esperado.

En el token, el emisor se identifica mediante una notificación de ISS; por ejemplo, `"iss":"https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/"`. En este ejemplo, la dirección base del valor de notificación, `https://sts.windows.net`, identifica a Azure Active Directory como el emisor, mientras que el segmento de la dirección relativa, _cbb1a5ac-f33b-45fa-9bf5-f37db0fed422_, es un identificador único del inquilino de Azure Active Directory para el que se emitió el token.

La audiencia del token será el identificador de la plantilla de aplicación de la aplicación en la galería. Cada una de las aplicaciones registradas en un solo inquilino puede recibir la misma notificación `iss` con solicitudes SCIM. El identificador de la plantilla de aplicación para todas las aplicaciones personalizadas es _8adf8e6e-67b2-4cf2-a259-e3dc5476c621_. El token generado por el servicio de aprovisionamiento de Azure AD solo se debe usar para realizar pruebas. No se debe usar en entornos de producción.

En el código de ejemplo, las solicitudes se autentican mediante el paquete Microsoft.AspNetCore.Authentication.JwtBearer. El código siguiente exige que las solicitudes a cualquiera de los puntos de conexión del servicio se autentiquen mediante el token de portador emitido por Azure Active Directory para un inquilino concreto:

```csharp
        public void ConfigureServices(IServiceCollection services)
        {
            if (_env.IsDevelopment())
            {
                ...
            }
            else
            {
                services.AddAuthentication(options =>
                {
                    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                })
                    .AddJwtBearer(options =>
                    {
                        options.Authority = " https://sts.windows.net/cbb1a5ac-f33b-45fa-9bf5-f37db0fed422/";
                        options.Audience = "8adf8e6e-67b2-4cf2-a259-e3dc5476c621";
                        ...
                    });
            }
            ...
        }

        public void Configure(IApplicationBuilder app)
        {
            ...
            app.UseAuthentication();
            app.UseAuthorization();
            ...
       }
```

También se requiere un token de portador para usar las [pruebas de Postman ](https://github.com/AzureAD/SCIMReferenceCode/wiki/Test-Your-SCIM-Endpoint) proporcionadas y realizar la depuración local con localhost. El código de ejemplo usa entornos ASP.NET Core para cambiar las opciones de autenticación durante la fase de desarrollo y habilitar el uso de un token autofirmado.

Para obtener más información sobre varios entornos en ASP.NET Core, use el siguiente vínculo: [Usar varios entornos en ASP.NET Core](
https://docs.microsoft.com/aspnet/core/fundamentals/environments)

El código siguiente exige que las solicitudes a cualquier punto de conexión del servicio se autentiquen mediante un token de portador firmado con una clave personalizada:

```csharp
        public void ConfigureServices(IServiceCollection services)
        {
            if (_env.IsDevelopment())
            {
                services.AddAuthentication(options =>
                {
                    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
                    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
                })
                    .AddJwtBearer(options =>
                    {
                        options.TokenValidationParameters =
                            new TokenValidationParameters
                            {
                                ValidateIssuer = false,
                                ValidateAudience = false,
                                ValidateLifetime = false,
                                ValidateIssuerSigningKey = false,
                                ValidIssuer = "Microsoft.Security.Bearer",
                                ValidAudience = "Microsoft.Security.Bearer",
                                IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("A1B2C3D4E5F6A1B2C3D4E5F6"))
                            };
                    });
            }
        ...
```

Envíe una solicitud GET al controlador de tokens para obtener un token de portador válido. El método _GenerateJSONWebToken_ es el responsable de crear un token que coincida con los parámetros configurados para el desarrollo:

```csharp
        private string GenerateJSONWebToken()
        {
            // Create token key
            SymmetricSecurityKey securityKey =
                new SymmetricSecurityKey(Encoding.UTF8.GetBytes("A1B2C3D4E5F6A1B2C3D4E5F6"));
            SigningCredentials credentials =
                new SigningCredentials(securityKey, SecurityAlgorithms.HmacSha256);

            // Set token expiration
            DateTime startTime = DateTime.UtcNow;
            DateTime expiryTime = startTime.AddMinutes(120);

            // Generate the token
            JwtSecurityToken token =
                new JwtSecurityToken(
                    "Microsoft.Security.Bearer",
                    "Microsoft.Security.Bearer",
                    null,
                    notBefore: startTime,
                    expires: expiryTime,
                    signingCredentials: credentials);

            string result = new JwtSecurityTokenHandler().WriteToken(token);
            return result;
        }
```

### <a name="handling-provisioning-and-deprovisioning-of-users"></a>Control del aprovisionamiento y desaprovisionamiento de usuarios

***Ejemplo 1. Consulta del servicio para buscar un usuario coincidente***

Azure Active Directory consulta el servicio para buscar un usuario con un valor de atributo `externalId` que coincida con el valor de atributo mailNickname de un usuario de Azure AD. La consulta se expresa como una solicitud de Protocolo de transferencia de hipertexto (HTTP) como la de este ejemplo, donde jyoung es un ejemplo de un mailNickname de un usuario en Azure Active Directory.

>[!NOTE]
> Esto es solo un ejemplo. No todos los usuarios tendrán un atributo mailNickname, y el valor que tiene un usuario no puede ser único en el directorio. Además, el atributo utilizado para la coincidencia (que en este caso es `externalId`) es configurable en las [asignaciones de atributos de Azure AD](customize-application-attributes.md).

```
GET https://.../scim/Users?filter=externalId eq jyoung HTTP/1.1
 Authorization: Bearer ...
```

En el código de ejemplo, la solicitud se traduce en una llamada al método QueryAsync del proveedor del servicio. Esta es la firma de ese método: 

```csharp
 // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
 // Microsoft.SCIM.IRequest is defined in 
 // Microsoft.SCIM.Service.  
 // Microsoft.SCIM.Resource is defined in 
 // Microsoft.SCIM.Schemas.  
 // Microsoft.SCIM.IQueryParameters is defined in 
 // Microsoft.SCIM.Protocol.  

 Task<Resource[]> QueryAsync(IRequest<IQueryParameters> request);
```

En la consulta de ejemplo, en el caso de un usuario con un valor especificado para el atributo `externalId`, los valores de los argumentos pasados al método QueryAsync serán los siguientes:

* parameters.AlternateFilters.Count: 1
* parameters.AlternateFilters.ElementAt(0).AttributePath: "externalId"
* parameters.AlternateFilters.ElementAt(0).ComparisonOperator: ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(0).ComparisonValue: "jyoung"

***Ejemplo 2. Aprovisionamiento de un usuario***

Si la respuesta a una consulta al servicio web relativa a un usuario con un valor de atributo `externalId` que coincide con el valor de atributo mailNickname de un usuario no devuelve ningún usuario, Azure Active Directory solicita al servicio que aprovisione un usuario correspondiente al de Azure Active Directory.  Este es un ejemplo de dicha solicitud: 

```
 POST https://.../scim/Users HTTP/1.1
 Authorization: Bearer ...
 Content-type: application/scim+json
 {
   "schemas":
   [
     "urn:ietf:params:scim:schemas:core:2.0:User",
     "urn:ietf:params:scim:schemas:extension:enterprise:2.0User"],
   "externalId":"jyoung",
   "userName":"jyoung",
   "active":true,
   "addresses":null,
   "displayName":"Joy Young",
   "emails": [
     {
       "type":"work",
       "value":"jyoung@Contoso.com",
       "primary":true}],
   "meta": {
     "resourceType":"User"},
    "name":{
     "familyName":"Young",
     "givenName":"Joy"},
   "phoneNumbers":null,
   "preferredLanguage":null,
   "title":null,
   "department":null,
   "manager":null}
```

En el código de ejemplo, la solicitud se traduce en una llamada al método CreateAsync del proveedor del servicio. Esta es la firma de ese método: 

```csharp
 // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
 // Microsoft.SCIM.IRequest is defined in 
 // Microsoft.SCIM.Service.  
 // Microsoft.SCIM.Resource is defined in 
 // Microsoft.SCIM.Schemas.  

 Task<Resource> CreateAsync(IRequest<Resource> request);
```

En una solicitud para aprovisionar un usuario, el valor del argumento del recurso es una instancia de la clase Microsoft.SCIM.Core2EnterpriseUser, que se define en la biblioteca Microsoft.SCIM.Schemas.  Si la solicitud para aprovisionar el usuario se realiza correctamente, se espera que la implementación del método devuelva una instancia de la clase Microsoft.SCIM.Core2EnterpriseUser, con el valor de la propiedad Identifier establecida en el identificador único del usuario recién aprovisionado.  

***Ejemplo 3. Consulta del estado actual de un usuario*** 

Para actualizar un usuario que se sabe que existe en un almacén de identidades dirigido por un SCIM, Azure Active Directory solicita el estado actual de dicho usuario desde el servicio con una solicitud como esta: 

```
 GET ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
 Authorization: Bearer ...
```

En el código de ejemplo, la solicitud se traduce en una llamada al método RetrieveAsync del proveedor del servicio. Esta es la firma de ese método: 

```csharp
 // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
 // Microsoft.SCIM.IRequest is defined in 
 // Microsoft.SCIM.Service.  
 // Microsoft.SCIM.Resource and 
 // Microsoft.SCIM.IResourceRetrievalParameters 
 // are defined in Microsoft.SCIM.Schemas 

 Task<Resource> RetrieveAsync(IRequest<IResourceRetrievalParameters> request);
```

En el ejemplo anterior de una solicitud para recuperar el estado actual de un usuario, los valores de las propiedades del objeto proporcionados como el valor del argumento parameters son los siguientes: 
  
* Identificador: "54D382A4-2050-4C03-94D1-E769F1D15682"
* SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

***Ejemplo 4. Consulta del valor de un atributo de referencia que se va a actualizar*** 

Si se va a actualizar un atributo de referencia, Azure Active Directory consulta el servicio para determinar si el valor actual del atributo de referencia en el almacén de identidades dirigido por el servicio, ya coincide con el valor de dicho atributo en Azure Active Directory. Para los usuarios, el único atributo del que se va a consultar el valor actual de esta manera es el atributo manager. Este es un ejemplo de una solicitud para determinar si el atributo de administrador de un objeto de usuario tiene, actualmente, un determinado valor: En el código de ejemplo, la solicitud se traduce en una llamada al método QueryAsync del proveedor del servicio. El valor de las propiedades del objeto proporcionado como el valor del argumento parameters es el siguiente: 
  
* parameters.AlternateFilters.Count: 2
* parameters.AlternateFilters.ElementAt(x).AttributePath: "ID"
* parameters.AlternateFilters.ElementAt(x).ComparisonOperator: ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(x).ComparisonValue: "54D382A4-2050-4C03-94D1-E769F1D15682"
* parameters.AlternateFilters.ElementAt(y).AttributePath: "manager"
* parameters.AlternateFilters.ElementAt(y).ComparisonOperator: ComparisonOperator.Equals
* parameters.AlternateFilter.ElementAt(y).ComparisonValue: "2819c223-7f76-453a-919d-413861904646"
* parameters.RequestedAttributePaths.ElementAt(0): "ID"
* parameters.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

En este caso, el valor del índice x puede ser 0 y el valor del índice y puede ser 1, o bien el valor de x puede ser 1 y el valor de y puede ser 0, en función del orden de las expresiones del parámetro de consulta filter.   

***Ejemplo 5. Solicitud de Azure AD a un servicio SCIM para actualizar un usuario*** 

A continuación se proporciona un ejemplo de una solicitud de Azure Active Directory a un servicio SCIM para actualizar un usuario: 

```
  PATCH ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
  Authorization: Bearer ...
  Content-type: application/scim+json
  {
    "schemas": 
    [
      "urn:ietf:params:scim:api:messages:2.0:PatchOp"],
    "Operations":
    [
      {
        "op":"Add",
        "path":"manager",
        "value":
          [
            {
              "$ref":"http://.../scim/Users/2819c223-7f76-453a-919d-413861904646",
              "value":"2819c223-7f76-453a-919d-413861904646"}]}]}
```

En el código de ejemplo, la solicitud se traduce en una llamada al método UpdateAsync del proveedor del servicio. Esta es la firma de ese método: 

```csharp
 // System.Threading.Tasks.Tasks and 
 // System.Collections.Generic.IReadOnlyCollection<T>  // are defined in mscorlib.dll.  
 // Microsoft.SCIM.IRequest is defined in
 // Microsoft.SCIM.Service.
 // Microsoft.SCIM.IPatch, 
 // is defined in Microsoft.SCIM.Protocol. 

 Task UpdateAsync(IRequest<IPatch> request);
```

En el ejemplo de una solicitud para actualizar un usuario, el objeto proporcionado como el valor del argumento patch tiene estos valores de propiedad: 
  
* ResourceIdentifier.Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
* ResourceIdentifier.SchemaIdentifier:  "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"
* (PatchRequest as PatchRequest2).Operations.Count: 1
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).OperationName: OperationName.Add
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).Path.AttributePath: "manager"
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.Count: 1
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.ElementAt(0).Reference: http://.../scim/Users/2819c223-7f76-453a-919d-413861904646
* (PatchRequest as PatchRequest2).Operations.ElementAt(0).Value.ElementAt(0).Value: 2819c223-7f76-453a-919d-413861904646

***Ejemplo 6. Desaprovisionamiento de un usuario***

Para desaprovisionar un usuario de un almacén de identidades dirigido por un servicio SCIM, Azure AD envía una solicitud como esta:

```
  DELETE ~/scim/Users/54D382A4-2050-4C03-94D1-E769F1D15682 HTTP/1.1
  Authorization: Bearer ...
```

En el código de ejemplo, la solicitud se traduce en una llamada al método DeleteAsync del proveedor del servicio. Esta es la firma de ese método: 

```csharp
 // System.Threading.Tasks.Tasks is defined in mscorlib.dll.  
 // Microsoft.SCIM.IRequest is defined in 
 // Microsoft.SCIM.Service.  
 // Microsoft.SCIM.IResourceIdentifier, 
 // is defined in Microsoft.SCIM.Protocol. 

 Task DeleteAsync(IRequest<IResourceIdentifier> request);
```

El objeto proporcionado como el valor del argumento resourceIdentifier tiene estos valores de propiedad en el ejemplo de una solicitud para desaprovisionar a un usuario: 

* ResourceIdentifier.Identifier: "54D382A4-2050-4C03-94D1-E769F1D15682"
* ResourceIdentifier.SchemaIdentifier: "urn:ietf:params:scim:schemas:extension:enterprise:2.0:User"

## <a name="step-4-integrate-your-scim-endpoint-with-the-azure-ad-scim-client"></a>Paso 4: Integre el punto de conexión SCIM con el cliente de SCIM de Azure AD

Azure AD se puede configurar para que aprovisione automáticamente usuarios y grupos asignados a aplicaciones que implementan un perfil específico del [protocolo SCIM 2.0](https://tools.ietf.org/html/rfc7644). Los detalles del perfil se documentan en [Paso 2: Información sobre la implementación de SCIM de Azure AD](#step-2-understand-the-azure-ad-scim-implementation).

Consulte a su proveedor de la aplicación o la documentación que se incluye con la aplicación para ver si existen declaraciones de compatibilidad con estos requisitos.

> [!IMPORTANT]
> La implementación de SCIM de Azure AD se basa en el servicio de aprovisionamiento de usuarios de Azure AD, que está diseñado para mantener a los usuarios constantemente sincronizados entre Azure AD y la aplicación de destino, y que implementa un conjunto muy específico de operaciones estándar. Es importante comprender estos comportamientos para entender el comportamiento del cliente de SCIM de Azure AD. Para obtener más información, consulte la sección [Ciclos de aprovisionamiento: inicial e incremental](how-provisioning-works.md#provisioning-cycles-initial-and-incremental) en [Funcionamiento del aprovisionamiento](how-provisioning-works.md).

### <a name="getting-started"></a>Introducción

Las aplicaciones que admiten el perfil SCIM descrito en este artículo se pueden conectar a Azure Active Directory mediante la característica "de aplicación situada fuera de la galería" de la galería de aplicaciones de Azure AD. Una vez conectadas, Azure AD ejecuta un proceso de sincronización cada 40 minutos en el que consulta el punto de conexión SCIM de la aplicación para ver los usuarios y grupos asignados, y los crea o modifica según los detalles de asignación.

**Para conectar una aplicación que admite SCIM:**

1. Inicie sesión en el [portal de Azure Active Directory](https://aad.portal.azure.com). Tenga en cuenta que puede obtener acceso a una evaluación gratuita de Azure Active Directory con licencias P2 si se suscribe al [programa para programadores](https://developer.microsoft.com/office/dev-program).
2. En el panel izquierdo, seleccione **Aplicaciones empresariales**. Se muestra una lista de las aplicaciones configuradas, incluidas aquellas que se han agregado desde la galería.
3. Seleccione **+ Nueva aplicación** > **Todas** > **Aplicación situada fuera de la galería**.
4. Escriba un nombre para la aplicación y seleccione **Agregar** para crear un objeto de aplicación. La nueva aplicación se agrega a la lista de aplicaciones empresariales y se abre en su pantalla de administración de la aplicación.

   ![Captura de pantalla que muestra la galería de aplicaciones de Azure AD](media/use-scim-to-provision-users-and-groups/scim-figure-2a.png)<br/>
   *Galería de aplicaciones de Azure AD*

5. En la pantalla de administración de la aplicación, seleccione **Aprovisionamiento** en el panel izquierdo.
6. En el menú **Modo de aprovisionamiento**, seleccione **Automático**.

   ![Ejemplo: Página Aprovisionamiento de una aplicación en Azure Portal](media/use-scim-to-provision-users-and-groups/scim-figure-2b.png)<br/>
   *Configuración del aprovisionamiento en Azure Portal*

7. En el campo **Dirección URL del inquilino**, escriba la dirección URL del punto de conexión SCIM de la aplicación. Ejemplo: `https://api.contoso.com/scim/`
8. Si el punto de conexión SCIM requiere un token de portador OAuth de un emisor que no sea Azure AD, copie el token de portador OAuth necesario en el campo **Token secreto**. Si este campo se deja en blanco, Azure AD incluye un token de portador OAuth emitido desde Azure AD con cada solicitud. Las aplicaciones que usan Azure AD como un proveedor de identidades pueden validar este token que emitió Azure AD. 
   > [!NOTE]
   > ***No*** se recomienda dejar este campo en blanco y basarse en un token generado por Azure AD. Esta opción está disponible principalmente para fines de prueba.
9. Seleccione **Probar conexión** para que Azure Active Directory intente conectarse al punto de conexión SCIM. Si se produce un error en el intento, se muestra la información de error.  

    > [!NOTE]
    > La **prueba de conexión** consulta el punto de conexión SCIM de un usuario que no existe mediante un GUID aleatorio, como la propiedad de coincidencia seleccionada en la configuración de Azure AD. La respuesta correcta esperada es HTTP 200 OK con un mensaje ListResponse de SCIM vacío.

10. Si la conexión se realiza con éxito, a continuación, seleccione **Guardar** para guardar las credenciales de administrador.
11. En la sección **Asignaciones**, hay dos conjuntos seleccionables de [asignaciones de atributos](customize-application-attributes.md): uno para los objetos de usuario y otro para los objetos de grupo. Seleccione cada uno de ellos para revisar los atributos que se sincronizan desde Azure Active Directory a la aplicación. Los atributos seleccionados como propiedades de **Coincidencia** se usan para buscar coincidencias con los usuarios y grupos de la aplicación con el objetivo de realizar operaciones de actualización. Para confirmar los cambios, seleccione **Guardar**.

    > [!NOTE]
    > Opcionalmente, puede deshabilitar la sincronización de objetos de grupo deshabilitando la asignación de "grupos".

12. En **Settings** (Configuración), el campo **Scope** (Ámbito) define qué usuarios y grupos se sincronizan. Seleccione **Sincronizar solo los usuarios y grupos asignados** (recomendado) para que se sincronicen solamente los usuarios y los grupos asignados en la pestaña **Usuarios y grupos**.
13. Una vez completada la configuración, cambie el **Estado de aprovisionamiento** a **Activado**.
14. Seleccione **Guardar** para iniciar el servicio de aprovisionamiento de Azure AD.
15. Al sincronizar solo los usuarios y grupos asignados (recomendado), no olvide seleccionar la pestaña **Usuarios y grupos** y asignar los usuarios o grupos que quiere sincronizar.

Una vez que haya empezado el ciclo inicial, puede seleccionar **Registros de aprovisionamiento** en el panel izquierdo para supervisar el progreso; con esto se muestran todas las acciones realizadas por el servicio de aprovisionamiento en la aplicación. Para más información sobre cómo leer los registros de aprovisionamiento de Azure AD, consulte el tutorial de [Creación de informes sobre el aprovisionamiento automático de cuentas de usuario](check-status-user-account-provisioning.md).

> [!NOTE]
> El ciclo inicial tarda más tiempo en realizarse que las sincronizaciones posteriores, que se producen aproximadamente cada 40 minutos si se está ejecutando el servicio.

## <a name="step-5-publish-your-application-to-the-azure-ad-application-gallery"></a>Paso 5: Publique la aplicación en la galería de aplicaciones de Azure AD

Si va a crear una aplicación que usará más de un inquilino, puede hacer que esté disponible en la galería de aplicaciones de Azure AD. De este modo, facilitará a las organizaciones la detección de la aplicación y la configuración del aprovisionamiento. Publicar la aplicación en la galería de Azure AD y hacer que el aprovisionamiento esté disponible para otros usuarios es fácil. Consulte los pasos [aquí](../develop/howto-app-gallery-listing.md). Microsoft colaborará con usted a fin de integrar su aplicación en nuestra galería, probar su punto de conexión y publicar la [documentación](../saas-apps/tutorial-list.md) de incorporación de versiones para que los clientes la usen. 

### <a name="gallery-onboarding-checklist"></a>Lista de comprobación de la incorporación a la galería
Siga la lista de comprobación siguiente para asegurarse de que la aplicación se incorpora con rapidez y que los clientes tienen una experiencia de implementación sin problemas. La información se recopilará en el momento en que se incorpore a la galería. 
> [!div class="checklist"]
> * Compatibilidad con un punto de conexión de grupo y de usuario de [SCIM 2.0 ](https://docs.microsoft.com/azure/active-directory/app-provisioning/use-scim-to-provision-users-and-groups#step-2-understand-the-azure-ad-scim-implementation) (solo se requiere uno, pero se recomiendan los dos)
> * Admisión de al menos 25 solicitudes por segundo por inquilino (obligatorio)
> * Establecimiento de contactos de ingeniería y soporte técnico para guiar la incorporación de clientes a la galería (obligatorio)
> * Tres credenciales de prueba sin expiración para la aplicación (obligatorio)
> * Compatibilidad con la concesión de código de autorización de OAuth o un token de larga duración, tal como se describe a continuación (obligatorio)
> * Establecimiento de un punto de contacto de ingeniería y soporte técnico para respaldar la incorporación de clientes a la galería (obligatorio)
> * Compatibilidad con la actualización de varias pertenencias a grupos con una única revisión (recomendado) 
> * Documentación del punto de conexión de SCIM públicamente (recomendado) 
> * [Compatibilidad con la detección de esquemas](https://tools.ietf.org/html/rfc7643#section-6) (recomendado)


### <a name="authorization-for-provisioning-connectors-in-the-application-gallery"></a>Autorización para aprovisionar conectores en la galería de aplicaciones
La especificación SCIM no define un esquema específico de SCIM para la autenticación y la autorización. Se basa en el uso de los estándares del sector actuales. El cliente de aprovisionamiento de Azure AD admite dos métodos de autorización para las aplicaciones de la galería. 

|Método de autorización|Ventajas|Desventajas|Soporte técnico|
|--|--|--|--|
|Nombre de usuario y contraseña (no recomendado ni compatible con Azure AD)|Fácil de implementar|No seguro: [Tu contraseña no importa](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/your-pa-word-doesn-t-matter/ba-p/731984)|Se admite según cada caso en las aplicaciones de la galería. No se admite para las aplicaciones que no son de la galería.|
|Token de portador de larga duración|Los tokens de larga duración no requieren que haya un usuario presente. Son fáciles de usar para los administradores al configurar el aprovisionamiento.|Los tokens de larga duración pueden ser difíciles de compartir con un administrador sin usar métodos no seguros como el correo electrónico. |Compatibles con las aplicaciones de la galería y las que no forman parte de ella. |
|Concesión de código de autorización de OAuth|Los tokens de acceso tienen una duración muy inferior a las contraseñas y tienen un mecanismo de actualización automatizado que los tokens de portador de larga duración no tienen.  Un usuario real debe estar presente durante la autorización inicial, lo que añade un nivel de responsabilidad. |Requiere que haya un usuario presente. Si el usuario deja la organización, el token no es válido y será necesario volver a realizar la autorización.|Compatible con aplicaciones de la galería, pero no con aplicaciones que no son de la galería. La compatibilidad con aplicaciones que no son de la galería es un trabajo pendiente.|
|Concesión de credenciales del cliente de OAuth|Los tokens de acceso tienen una duración muy inferior a las contraseñas y tienen un mecanismo de actualización automatizado que los tokens de portador de larga duración no tienen. Tanto la concesión de código de autorización como la concesión de credenciales de cliente crean el mismo tipo de token de acceso, por lo que el cambio entre estos métodos es transparente para la API.  El aprovisionamiento se puede automatizar completamente y los nuevos tokens se pueden solicitar silenciosamente sin la interacción del usuario. ||No compatible con las aplicaciones de la galería y las que no forman parte de ella. La compatibilidad se encuentra en nuestro trabajo pendiente.|

> [!NOTE]
> No se recomienda dejar en blanco el campo de token en la interfaz de usuario de la aplicación personalizada de la configuración de aprovisionamiento de Azure AD. El token generado está disponible principalmente para fines de prueba.

**Flujo de concesión de código de autorización OAuth:** el servicio de aprovisionamiento admite la [concesión de código de autorización](https://tools.ietf.org/html/rfc6749#page-24). Después de enviar la solicitud para publicar la aplicación en la galería, nuestro equipo trabajará con usted para recopilar la información siguiente:
*  Dirección URL de autorización: una dirección URL del cliente para obtener la autorización del propietario del recurso a través de la redirección del agente de usuario. Se redirige al usuario a esta dirección URL para autorizar el acceso. Tenga en cuenta que esta dirección URL no es actualmente configurable por inquilino.
*  Dirección URL de intercambio de token: una dirección URL por parte del cliente para intercambiar una concesión de autorización para un token de acceso, normalmente con autenticación del cliente. Tenga en cuenta que esta dirección URL no es actualmente configurable por inquilino.
*  Identificador de cliente: el servidor de autorización emite al cliente registrado un identificador de cliente, que es una cadena única que representa la información de registro que proporciona el cliente.  El identificador de cliente no es un secreto; se expone al propietario del recurso y **no debe** usarse solo para la autenticación del cliente.  
*  Secreto del cliente: un secreto generado por el servidor de autorización. Debe ser un valor único conocido solo para el servidor de autorización. 

Tenga en cuenta que OAuth v1 no se admite debido a la exposición del secreto de cliente. Se admite OAuth v2.  

Procedimientos recomendados (recomendables, pero no obligatorios):
* Se admiten varias direcciones URL de redireccionamiento. Los administradores pueden configurar el aprovisionamiento tanto desde "portal.azure.com" como desde "aad.portal.azure.com". La compatibilidad con varias direcciones URL de redireccionamiento garantizará que los usuarios puedan autorizar el acceso desde cualquier portal.
* Se admiten varios secretos para garantizar una renovación de secretos sin problemas ni tiempo de inactividad. 

**Tokens de portador OAuth de larga duración:** si la aplicación no admite el flujo de concesión de código de autorización de OAuth, también puede generar un token de portador de OAuth de larga duración que un administrador puede usar para configurar la integración del aprovisionamiento. El token debe ser perpetuo o, de lo contrario, el trabajo de aprovisionamiento se [pondrá en cuarentena](application-provisioning-quarantine-status.md) cuando expire el token. Este token debe tener un tamaño inferior a 1 KB.  

Puede solicitar más métodos de autenticación y autorización en [UserVoice](https://aka.ms/appprovisioningfeaturerequest).

### <a name="gallery-go-to-market-launch-check-list"></a>Lista de comprobación del lanzamiento de la comercialización en la galería
Para ayudar a impulsar el reconocimiento y la demanda de nuestra integración conjunta, se recomienda actualizar la documentación existente y ampliar la integración en los canales de marketing.  A continuación, se muestra un conjunto de actividades de lista de comprobación que se recomienda completar para respaldar el lanzamiento

* **Preparación de las ventas y del soporte al cliente.** Asegúrese de que sus equipos de ventas y soporte técnico son conscientes y pueden hablar con las funcionalidades de integración. Informe al equipo de ventas y de soporte técnico, proporcióneles las preguntas más frecuentes e incluya la integración en sus materiales de ventas. 
* **Entrada de blog y comunicado de prensa.** Cree una entrada de blog o un comunicado de prensa que describan la integración conjunta, las ventajas y cómo empezar. [Ejemplo: Comunicado de Imprivata y Azure Active Directory](https://www.imprivata.com/company/press/imprivata-introduces-iam-cloud-platform-healthcare-supported-microsoft) 
* **Redes sociales.** Aproveche sus redes sociales como Twitter, Facebook o LinkedIn para promover la integración en sus clientes. Asegúrese de incluir @AzureAD para que podamos retweettear la publicación. [Ejemplo: Publicación de Twitter de Imprivata](https://twitter.com/azuread/status/1123964502909779968)
* **Sitio web de marketing.** Cree o actualice las páginas de marketing (por ejemplo, la página de integración, la página de asociados, la página de precios, etc.) para incluir la disponibilidad de la integración conjunta. [Ejemplo: Página de integración de Pingboard](https://pingboard.com/org-chart-for), [Página de integración de Smartsheet](https://www.smartsheet.com/marketplace/apps/microsoft-azure-ad), [Página de precios de Monday.com](https://monday.com/pricing/) 
* **Documentación técnica.** Cree un artículo en el centro de ayuda o documentación técnica sobre cómo pueden empezar los clientes. [Ejemplo: Integración de Envoy + Microsoft Azure Active Directory.](https://envoy.help/en/articles/3453335-microsoft-azure-active-directory-integration/
) 
* **Comunicación al cliente.** Avise a los clientes de la nueva integración a través de la comunicación al cliente (boletines mensuales, campañas por correo electrónico, notas de la versión del producto). 

## <a name="related-articles"></a>Artículos relacionados

* [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS](user-provisioning.md)
* [Personalización de asignaciones de atributos para el aprovisionamiento de usuarios](customize-application-attributes.md)
* [Escritura de expresiones para la asignación de atributos](functions-for-customizing-application-data.md)
* [Filtros de ámbito para el aprovisionamiento de usuarios](define-conditional-rules-for-provisioning-user-accounts.md)
* [Notificaciones de aprovisionamiento de cuentas](user-provisioning.md)
* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS](../saas-apps/tutorial-list.md)
