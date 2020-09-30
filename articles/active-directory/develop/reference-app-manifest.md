---
title: Descripción del manifiesto de la aplicación de Azure Active Directory
titleSuffix: Microsoft identity platform
description: Cobertura detallada del manifiesto de aplicación de Azure Active Directory, que representa la configuración de identidad de una aplicación en un inquilino de Azure AD, y se usa para facilitar la autorización de OAuth, la experiencia de consentimiento y mucho más.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: reference
ms.workload: identity
ms.date: 04/15/2020
ms.author: ryanwi
ms.custom: aaddev
ms.reviewer: sureshja
ms.openlocfilehash: 554e5a022dcb49cd861ad7198a2c375634db6d10
ms.sourcegitcommit: 7374b41bb1469f2e3ef119ffaf735f03f5fad484
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/16/2020
ms.locfileid: "90705752"
---
# <a name="azure-active-directory-app-manifest"></a>Manifiesto de aplicación de Azure Active Directory

El manifiesto de la aplicación contiene una definición de todos los atributos de un objeto de la aplicación en la plataforma de identidad de Microsoft. También sirve como mecanismo para actualizar el objeto de la aplicación. Para más información sobre la entidad Application y su esquema, consulte la [documentación sobre la entidad Application de Graph API](/graph/api/resources/application).

Los atributos de una aplicación pueden configurarse en Azure Portal o mediante programación con la [API de REST](/graph/api/resources/application) o [PowerShell](/powershell/module/azuread/?view=azureadps-2.0#applications). Sin embargo, hay algunos escenarios en los que será necesario editar el manifiesto de la aplicación para poder configurar los atributos de una aplicación. Entre los escenarios se incluyen los siguientes:

* Si registró la aplicación como una cuenta multiinquilino de Azure AD y una cuenta de Microsoft personal, no podrá cambiar las cuentas de Microsoft compatibles en la interfaz de usuario. En su lugar, tendrá que utilizar el editor de manifiesto de la aplicación para cambiar los tipos de cuenta compatibles.
* Si necesita definir permisos y roles compatibles con la aplicación, tendrá que modificar el manifiesto de la aplicación.

## <a name="configure-the-app-manifest"></a>Configuración del manifiesto de la aplicación

Para configurar el manifiesto de la aplicación:

1. Vaya a [Azure Portal](https://portal.azure.com). Busque y seleccione el servicio **Azure Active Directory**.
1. Seleccione **App registrations** (Registros de aplicaciones).
1. Seleccione la aplicación que desee configurar.
1. Desde la página **Introducción** de la aplicación, seleccione la sección **Manifiesto**. Se abrirá un editor de manifiestos basado en la Web que le permitirá editar el manifiesto desde el portal. Si lo desea, también puede seleccionar **Descargar** para editar el manifiesto de forma local y usar después **Cargar** para aplicarlo de nuevo a la aplicación.

## <a name="manifest-reference"></a>Referencia de manifiesto

En esta sección se describen los atributos que se encuentran en el manifiesto de la aplicación.

### <a name="id-attribute"></a>Atributo id

| Clave | Tipo de valor |
| :--- | :--- |
| id | String |

El identificador único de la aplicación en el directorio. Este identificador no es el que se usa para identificar la aplicación en cualquier transacción del protocolo. Se usa para hacer referencia al objeto en las consultas del directorio.

Ejemplo:

```json
    "id": "f7f9acfc-ae0c-4d6c-b489-0a81dc1652dd",
```

### <a name="accesstokenacceptedversion-attribute"></a>Atributo accessTokenAcceptedVersion

| Clave | Tipo de valor |
| :--- | :--- |
| accessTokenAcceptedVersion | Int32 que acepta valores null |

Especifica la versión del token de acceso que espera el recurso. Este parámetro cambia la versión y el formato de JWT generado independientemente del punto de conexión o cliente utilizado para solicitar el token de acceso.

El punto de conexión usado, v1.0 o v2.0, lo elige el cliente y solo afecta a la versión de id_tokens. Los recursos deben configurar explícitamente `accesstokenAcceptedVersion` para indicar el formato del token de acceso admitido.

Los valores posibles de `accesstokenAcceptedVersion` son 1, 2 o un valor null. Si el valor es NULL, el valor predeterminado es 1, lo que corresponde al punto de conexión v1.0.

Si `signInAudience` es `AzureADandPersonalMicrosoftAccount`, el valor debe ser `2`.

Ejemplo:

```json
    "accessTokenAcceptedVersion": 2,
```

### <a name="addins-attribute"></a>Atributo addIns

| Clave | Tipo de valor |
| :--- | :--- |
| addIns | Colección |

Define el comportamiento personalizado que puede usar un servicio de consumo para llamar a una aplicación en contextos concretos. Por ejemplo, las aplicaciones que pueden representar secuencias de archivos pueden establecer la propiedad `addIns` para su funcionalidad "FileHandler". Este parámetro permitirá que servicios como Microsoft 365 llamen a la aplicación en el contexto de un documento en el que esté trabajando el usuario.

Ejemplo:

```json
    "addIns": [
       {
        "id": "968A844F-7A47-430C-9163-07AE7C31D407",
        "type":" FileHandler",
        "properties": [
           {
              "key": "version",
              "value": "2"
           }
        ]
       }
    ],
```

### <a name="allowpublicclient-attribute"></a>Atributo allowPublicClient

| Clave | Tipo de valor |
| :--- | :--- |
| allowPublicClient | Boolean |

Especifica el tipo de aplicación de reserva. Azure AD deduce el tipo de aplicación a partir de replyUrlsWithType, de forma predeterminada. Hay algunos escenarios en los que Azure AD no puede determinar el tipo de aplicación cliente. Por ejemplo, uno de estos escenarios es el flujo [ROPC](https://tools.ietf.org/html/rfc6749#section-4.3) en el que la solicitud HTTP se realiza sin redireccionamiento de direcciones URL). En esos casos, Azure AD interpretará el tipo de aplicación en función del valor de esta propiedad. Si este valor se establece en true, el tipo de aplicación de reserva se establece como cliente público, como una aplicación instalada que se ejecuta en un dispositivo móvil. El valor predeterminado es false, lo que significa que el tipo de aplicación de reserva es un cliente confidencial, como una aplicación web.

Ejemplo:

```json
    "allowPublicClient": false,
```

### <a name="availabletoothertenants-attribute"></a>Atributo availableToOtherTenants

| Clave | Tipo de valor |
| :--- | :--- |
| availableToOtherTenants | Boolean |

Es "true" si la aplicación se comparte con otros inquilinos; de lo contrario, es "false".

> [!NOTE]
> Este atributo solo está disponible en la experiencia **Registros de aplicaciones (heredados)** . Se reemplaza por `signInAudience` en la experiencia [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908).

### <a name="appid-attribute"></a>Atributo appId

| Clave | Tipo de valor |
| :--- | :--- |
| appId | String |

Especifica el identificador único de la aplicación que Azure AD asigna a una aplicación.

Ejemplo:

```json
    "appId": "601790de-b632-4f57-9523-ee7cb6ceba95",
```

### <a name="approles-attribute"></a>Atributo appRoles

| Clave | Tipo de valor |
| :--- | :--- |
| appRoles | Colección |

Especifica la colección de roles que una aplicación puede declarar. Dichos roles se pueden asignar a usuarios, grupos o entidades de servicio. Para ver más ejemplos e información, consulte el artículo [Agregar roles de aplicación en la aplicación y recibirlos en el token](howto-add-app-roles-in-azure-ad-apps.md).

Ejemplo:

```json
    "appRoles": [
        {
           "allowedMemberTypes": [
               "User"
           ],
           "description": "Read-only access to device information",
           "displayName": "Read Only",
           "id": "601790de-b632-4f57-9523-ee7cb6ceba95",
           "isEnabled": true,
           "value": "ReadOnly"
        }
    ],
```

### <a name="displayname-attribute"></a>Atributo displayName

| Clave | Tipo de valor |
| :--- | :--- |
| DisplayName | String |

El nombre para mostrar de la aplicación.

> [!NOTE]
> Este atributo solo está disponible en la experiencia **Registros de aplicaciones (heredados)** . Se reemplaza por `name` en la experiencia [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908).

### <a name="errorurl-attribute"></a>Atributo errorUrl

| Clave | Tipo de valor |
| :--- | :--- |
| errorUrl | String |

No compatible.

### <a name="groupmembershipclaims-attribute"></a>Atributo groupMembershipClaims

| Clave | Tipo de valor |
| :--- | :--- |
|groupMembershipClaims | String |

Configura la notificación `groups` emitida en un token de acceso OAuth 2.0 o de usuario que la aplicación espera. Para establecer este atributo, use uno de los siguientes valores de cadena válidos:

- `"None"`
- `"SecurityGroup"` (para grupos de seguridad y roles de Azure AD)
- `"All"` (se obtendrán todos los grupos de seguridad, grupos de distribución y los roles de directorio de Azure AD a los que pertenezca el usuario que inició sesión.

Ejemplo:

```json
    "groupMembershipClaims": "SecurityGroup",
```

### <a name="homepage-attribute"></a>Atributo homepage

| Clave | Tipo de valor |
| :--- | :--- |
| página principal |String |

URL a la página principal de la aplicación.

> [!NOTE]
> Este atributo solo está disponible en la experiencia **Registros de aplicaciones (heredados)** . Se reemplaza por `signInUrl` en la experiencia [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908).

### <a name="objectid-attribute"></a>Atributo objectId

| Clave | Tipo de valor |
| :--- | :--- |
|objectId | String |

El identificador único de la aplicación en el directorio.

Esto solo está disponible en la experiencia **Registros de aplicaciones (heredados)** . Se reemplaza por `id` en la experiencia [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908).

Ejemplo:

```json
    "objectId": "f7f9acfc-ae0c-4d6c-b489-0a81dc1652dd",
```

### <a name="optionalclaims-attribute"></a>Atributo optionalClaims

| Clave | Tipo de valor |
| :--- | :--- |
| optionalClaims | String |

Las notificaciones opcionales que el servicio de token de seguridad devuelve en el token para esta aplicación específica.

En este momento, las aplicaciones que admiten cuentas personales y cuentas de Azure AD (registradas mediante el portal de registro de aplicaciones) no pueden usar notificaciones opcionales. Sin embargo, las aplicaciones registradas solo para Azure AD mediante el punto de conexión v2.0 pueden obtener las notificaciones opcionales que han solicitado en el manifiesto. Para obtener más información, consulte las [notificaciones opcionales](active-directory-optional-claims.md).

Ejemplo:

```json
    "optionalClaims": null,
```



### <a name="identifieruris-attribute"></a>Atributo identifierUris

| Clave | Tipo de valor |
| :--- | :--- |
| identifierUris | Matriz de cadenas |

Los URI definidos por el usuario que identifican de manera única una aplicación web en su inquilino de Azure AD o en un dominio personalizado comprobado si la aplicación tiene varios inquilinos.

Ejemplo:

```json
    "identifierUris": "https://MyRegisteredApp",
```

### <a name="informationalurls-attribute"></a>Atributo informationalUrls

| Clave | Tipo de valor |
| :--- | :--- |
| informationalUrls | String |

Especifica los vínculos a los términos del servicio y la declaración de privacidad de la aplicación. Las condiciones del servicio y la declaración de privacidad se exponen a los usuarios mediante la experiencia de consentimiento del usuario. Para obtener más información, consulte [Cómo agregar condiciones del servicio y declaración de privacidad de las aplicaciones registradas de Azure AD](howto-add-terms-of-service-privacy-statement.md).

Ejemplo:

```json
    "informationalUrls": {
        "termsOfService": "https://MyRegisteredApp/termsofservice",
        "support": "https://MyRegisteredApp/support",
        "privacy": "https://MyRegisteredApp/privacystatement",
        "marketing": "https://MyRegisteredApp/marketing"
    },
```

### <a name="keycredentials-attribute"></a>Atributo keyCredentials

| Clave | Tipo de valor |
| :--- | :--- |
| keyCredentials | Colección |

Contiene referencias a credenciales asignadas por la aplicación, secretos compartidos basados en cadena y certificados X.509. Estas credenciales se usan cuando se solicitan tokens de acceso (cuando la aplicación actúa como cliente, en lugar de como recurso).

Ejemplo:

```json
    "keyCredentials": [
        {
           "customKeyIdentifier":null,
           "endDate":"2018-09-13T00:00:00Z",
           "keyId":"<guid>",
           "startDate":"2017-09-12T00:00:00Z",
           "type":"AsymmetricX509Cert",
           "usage":"Verify",
           "value":null
        }
    ],
```

### <a name="knownclientapplications-attribute"></a>Atributo knownClientApplications

| Clave | Tipo de valor |
| :--- | :--- |
| knownClientApplications | Matriz de cadenas |

Se usa para el consentimiento de unión si dispone de una solución que contenga dos partes, una aplicación cliente y una aplicación de API web personalizada. Si especifica el valor de appID de la aplicación cliente en este valor, el usuario solo tendrá que dar su consentimiento una vez a la aplicación cliente. Azure AD sabrá que el consentimiento al cliente significa consentir implícitamente a la API web. Aprovisionará automáticamente las entidades de servicio para el cliente y la API web al mismo tiempo. Tanto el cliente como la aplicación de API web deben estar registrados en el mismo inquilino.

Ejemplo:

```json
    "knownClientApplications": ["f7f9acfc-ae0c-4d6c-b489-0a81dc1652dd"],
```

### <a name="logourl-attribute"></a>Atributo logoUrl

| Clave | Tipo de valor |
| :--- | :--- |
| logoUrl | String |

Leer el valor solo que apunta a la dirección URL de CDN para el logotipo que se cargó en el portal.

Ejemplo:

```json
    "logoUrl": "https://MyRegisteredAppLogo",
```

### <a name="logouturl-attribute"></a>Atributo logoutUrl

| Clave | Tipo de valor |
| :--- | :--- |
| logoutUrl | String |

La dirección URL para cerrar la sesión de la aplicación.

Ejemplo:

```json
    "logoutUrl": "https://MyRegisteredAppLogout",
```

### <a name="name-attribute"></a>Atributo name

| Clave | Tipo de valor |
| :--- | :--- |
| name | String |

El nombre para mostrar de la aplicación.

Ejemplo:

```json
    "name": "MyRegisteredApp",
```

### <a name="oauth2allowimplicitflow-attribute"></a>Atributo oauth2AllowImplicitFlow

| Clave | Tipo de valor |
| :--- | :--- |
| oauth2AllowImplicitFlow | Boolean |

Especifica si esta aplicación web puede solicitar tokens de acceso de flujo implícitos de OAuth2.0. El valor predeterminado es false. Esta marca se usa para las aplicaciones basadas en explorador, como las aplicaciones de página única de JavaScript. Para más información, escriba `OAuth 2.0 implicit grant flow` en la tabla de contenido y vea los temas sobre el flujo implícito.

Ejemplo:

```json
    "oauth2AllowImplicitFlow": false,
```

### <a name="oauth2allowidtokenimplicitflow-attribute"></a>Atributo oauth2AllowIdTokenImplicitFlow

| Clave | Tipo de valor |
| :--- | :--- |
| oauth2AllowIdTokenImplicitFlow | Boolean |

Especifica si esta aplicación web puede solicitar tokens de identificador de flujo implícitos de OAuth2.0. El valor predeterminado es false. Esta marca se usa para las aplicaciones basadas en explorador, como las aplicaciones de página única de JavaScript.

Ejemplo:

```json
    "oauth2AllowIdTokenImplicitFlow": false,
```

### <a name="oauth2permissions-attribute"></a>Atributo oauth2Permissions

| Clave | Tipo de valor |
| :--- | :--- |
| oauth2Permissions | Colección |

Especifica la colección de ámbitos de permiso de OAuth 2.0 que la aplicación de API (recurso) web expone a las aplicaciones cliente. Estos ámbitos de permisos pueden concederse a las aplicaciones cliente durante el consentimiento.

Ejemplo:

```json
    "oauth2Permissions": [
       {
          "adminConsentDescription": "Allow the app to access resources on behalf of the signed-in user.",
          "adminConsentDisplayName": "Access resource1",
          "id": "<guid>",
          "isEnabled": true,
          "type": "User",
          "userConsentDescription": "Allow the app to access resource1 on your behalf.",
          "userConsentDisplayName": "Access resources",
          "value": "user_impersonation"
        }
    ],
```

### <a name="oauth2requiredpostresponse-attribute"></a>Atributo oauth2RequiredPostResponse

| Clave | Tipo de valor |
| :--- | :--- |
| oauth2RequiredPostResponse | Boolean |

Especifica si, como parte de las solicitudes de tokens de OAuth 2.0, Azure AD permitirá solicitudes POST, frente a las solicitudes GET. El valor predeterminado es false, que especifica solo se permitirán solicitudes GET.

Ejemplo:

```json
    "oauth2RequirePostResponse": false,
```

### <a name="parentalcontrolsettings-attribute"></a>Atributo parentalControlSettings

| Clave | Tipo de valor |
| :--- | :--- |
| parentalControlSettings | String |

- `countriesBlockedForMinors` especifica los países o regiones en los que la aplicación se bloquea para menores de edad.
- `legalAgeGroupRule` especifica la regla de grupo de edad legal que se aplica a los usuarios de la aplicación. Se puede establecer en `Allow`, `RequireConsentForPrivacyServices`, `RequireConsentForMinors`, `RequireConsentForKids` o `BlockMinors`.

Ejemplo:

```json
    "parentalControlSettings": {
        "countriesBlockedForMinors": [],
        "legalAgeGroupRule": "Allow"
    },
```

### <a name="passwordcredentials-attribute"></a>Atributo passwordCredentials

| Clave | Tipo de valor |
| :--- | :--- |
| passwordCredentials | Colección |

Consulte la descripción de la propiedad `keyCredentials`.

Ejemplo:

```json
    "passwordCredentials": [
      {
        "customKeyIdentifier": null,
        "endDate": "2018-10-19T17:59:59.6521653Z",
        "keyId": "<guid>",
        "startDate":"2016-10-19T17:59:59.6521653Z",
        "value":null
      }
    ],
```

### <a name="preauthorizedapplications-attribute"></a>Atributo preAuthorizedApplications

| Clave | Tipo de valor |
| :--- | :--- |
| preAuthorizedApplications | Colección |

Enumera las aplicaciones y los permisos solicitados para el consentimiento implícito. Requiere que un administrador haya proporcionado su consentimiento a la aplicación. preAuthorizedApplications no requiere que el usuario dé su consentimiento para los permisos solicitados. Los permisos enumerados en preAuthorizedApplications no requieren el consentimiento del usuario. Sin embargo, los permisos solicitados adicionales que no aparecen en preAuthorizedApplications requieren el consentimiento del usuario.

Ejemplo:

```json
    "preAuthorizedApplications": [
       {
          "appId": "abcdefg2-000a-1111-a0e5-812ed8dd72e8",
          "permissionIds": [
             "8748f7db-21fe-4c83-8ab5-53033933c8f1"
            ]
        }
    ],
```

### <a name="publicclient-attribute"></a>Atributo publicClient

| Clave | Tipo de valor |
| :--- | :--- |
| publicClient | Boolean|

Especifica si esta aplicación es un cliente público (como una aplicación instalada que se ejecuta en un dispositivo móvil).

Esta propiedad solo está disponible en la experiencia **Registros de aplicaciones (heredados)** . Se reemplaza por `allowPublicClient` en la experiencia [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908).

### <a name="publisherdomain-attribute"></a>Atributo publisherDomain

| Clave | Tipo de valor |
| :--- | :--- |
| publisherDomain | String |

El dominio del publicador comprobado para la aplicación. Solo lectura.

Ejemplo:

```json
    "publisherDomain": "https://www.contoso.com",
````

### <a name="replyurls-attribute"></a>Atributo replyUrls

| Clave | Tipo de valor |
| :--- | :--- |
| replyUrls | Matriz de cadena |

Esta propiedad multivalor contiene la lista de valores de redirect_uri registrados que Azure AD aceptará como destinos cuando se devuelvan tokens.

Esta propiedad solo está disponible en la experiencia **Registros de aplicaciones (heredados)** . Se reemplaza por `replyUrlsWithType` en la experiencia [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908).

### <a name="replyurlswithtype-attribute"></a>Atributo replyUrlsWithType

| Clave | Tipo de valor |
| :--- | :--- |
| replyUrlsWithType | Colección |

Esta propiedad multivalor contiene la lista de valores de redirect_uri registrados que Azure AD aceptará como destinos cuando se devuelvan tokens. Cada valor del identificador URI debe contener un valor de tipo de aplicación asociado. Los valores de tipo admitidos son:

- `Web`
- `InstalledClient`
- `Spa`

Para obtener más información, consulte las [limitaciones y restricciones de las URL de respuesta](./reply-url.md).

Ejemplo:

```json
    "replyUrlsWithType": [
       {
          "url": "https://localhost:4400/services/office365/redirectTarget.html",
          "type": "InstalledClient"
       }
    ],
```

### <a name="requiredresourceaccess-attribute"></a>Atributo requiredResourceAccess

| Clave | Tipo de valor |
| :--- | :--- |
| requiredResourceAccess | Colección |

Con el consentimiento dinámico, `requiredResourceAccess` controla la experiencia de consentimiento del administrador y la experiencia de consentimiento del usuario para usuarios que usan consentimiento estático. Aunque este parámetro no controla la experiencia de consentimiento del usuario del caso general.

- `resourceAppId` es el único identificador del recurso al que la aplicación necesita acceso. Este valor debe ser igual que el valor de appId declarado en la aplicación del recurso de destino.
- `resourceAccess` es una matriz que enumera los ámbitos de permiso de OAuth 2.0 y los roles de aplicación que necesita la aplicación del recurso especificado. Contiene los valores `id` y `type` de los recursos especificados.

Ejemplo:

```json
    "requiredResourceAccess": [
        {
            "resourceAppId": "00000002-0000-0000-c000-000000000000",
            "resourceAccess": [
                {
                    "id": "311a71cc-e848-46a1-bdf8-97ff7156d8e6",
                    "type": "Scope"
                }
            ]
        }
    ],
```

### <a name="samlmetadataurl-attribute"></a>Atributo samlMetadataUrl

| Clave | Tipo de valor |
| :--- | :--- |
| samlMetadataUrl | String |

La dirección URL de los metadatos de SAML de la aplicación.

Ejemplo:

```json
    "samlMetadataUrl": "https://MyRegisteredAppSAMLMetadata",
```

### <a name="signinurl-attribute"></a>Atributo signInUrl

| Clave | Tipo de valor |
| :--- | :--- |
| signInUrl | String |

Especifica la dirección URL a la página principal de la aplicación.

Ejemplo:

```json
    "signInUrl": "https://MyRegisteredApp",
```

### <a name="signinaudience-attribute"></a>Atributo signInAudience

| Clave | Tipo de valor |
| :--- | :--- |
| signInAudience | String |

Especifica qué cuentas de Microsoft se admiten en la aplicación actual. Los valores admitidos son:
- `AzureADMyOrg`: usuarios con una cuenta profesional o educativa de Microsoft en el inquilino de Azure AD de la organización (por ejemplo, un inquilino único).
- `AzureADMultipleOrgs`: usuarios con una cuenta profesional o educativa de Microsoft en el inquilino de Azure AD de una organización (por ejemplo, un multiinquilino).
- `AzureADandPersonalMicrosoftAccount`: usuarios con una cuenta de Microsoft personal o una cuenta profesional o educativa en el inquilino de Azure AD de una organización.
- `PersonalMicrosoftAccount`: cuentas personales que se usan para iniciar sesión en servicios como Xbox y Skype.

Ejemplo:

```json
    "signInAudience": "AzureADandPersonalMicrosoftAccount",
```

### <a name="tags-attribute"></a>Atributo tags

| Clave | Tipo de valor |
| :--- | :--- |
| etiquetas | Matriz de cadenas  |

Cadenas personalizadas que pueden utilizarse para clasificar e identificar la aplicación.

Ejemplo:

```json
    "tags": [
       "ProductionApp"
    ],
```

## <a name="common-issues"></a>Problemas comunes

### <a name="manifest-limits"></a>Límites de manifiesto

Un manifiesto de aplicación tiene varios atributos que se conocen como colecciones; por ejemplo, appRoles, keyCredentials, knownClientApplications, identifierUris, redirectUris, requiredResourceAccess y oauth2Permissions. Dentro del manifiesto de aplicación completa para cualquier aplicación, el número total de entradas de todas las colecciones combinadas se ha limitado a 1200. Si previamente especifica 100 URI de redireccionamiento en el manifiesto de aplicación, solo le quedarán 1100 entradas restantes para usar entre todas las demás colecciones combinadas que componen el manifiesto.

> [!NOTE]
> Si intenta agregar más de 1200 entradas en el manifiesto de aplicación, es posible que aparezca un error **"No se pudo actualizar la aplicación xxxxxx. Detalles del error: El tamaño del manifiesto ha excedido su límite. Reduzca el número de valores y vuelva a intentar la solicitud".**

### <a name="unsupported-attributes"></a>Atributos no admitidos

El manifiesto de aplicación representa el esquema del modelo de aplicación subyacente de Azure AD. A medida que evoluciona el esquema subyacente, el editor de manifiestos se actualizará para reflejar el nuevo esquema de vez en cuando. Como resultado, es posible que observe nuevos atributos que se muestran en el manifiesto de aplicación. En raras ocasiones, es posible que observe un cambio semántico o sintáctico en los atributos existentes o quizás encontrará un atributo que existía previamente, pero ya no se admite. Por ejemplo, verá los nuevos atributos en los [Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) que se conocen por otro nombre en la experiencia de Registros de aplicaciones (heredados).

| Registros de aplicaciones (heredados)| Registros de aplicaciones           |
|---------------------------|-----------------------------|
| `availableToOtherTenants` | `signInAudience`            |
| `displayName`             | `name`                      |
| `errorUrl`                | -                           |
| `homepage`                | `signInUrl`                 |
| `objectId`                | `Id`                        |
| `publicClient`            | `allowPublicClient`         |
| `replyUrls`               | `replyUrlsWithType`         |

Para obtener descripciones de estos atributos, consulte la sección de [referencia de manifiesto](#manifest-reference).

Al intentar cargar un manifiesto descargado anteriormente, es posible que vea uno de los siguientes errores. Este error probablemente se deba a que el editor de manifiestos ahora admite una versión más reciente del esquema, que no coincide con el que está intentando cargar.

* "No se pudo actualizar la aplicación xxxxxx. Detalle del error: Identificador de objeto no válido "undefined". []".
* "No se pudo actualizar la aplicación xxxxxx. Detalle del error: Uno o más de los valores de propiedad especificados no son válidos. []".
* "No se pudo actualizar la aplicación xxxxxx. Detalle del error: No se permite establecer availableToOtherTenants en esta versión de API para la actualización. []".
* "No se pudo actualizar la aplicación xxxxxx. Detalle del error: No se permiten actualizaciones de la propiedad "replyUrls" para esta aplicación. Use la propiedad "replyUrlsWithType" en su lugar. []".
* "No se pudo actualizar la aplicación xxxxxx. Detalle del error: Se encontró un valor sin nombre de tipo y no ningún tipo esperado disponible. Cuando se especifica el modelo, cada valor de la carga debe tener un tipo que el autor puede especificar en la carga de forma explícita o que se puede inferir de forma implícita a partir del valor primario. []"

Cuando vea uno de estos errores, se recomienda realizar las acciones siguientes:

1. Edite los atributos de forma individualmente en el editor de manifiestos en lugar de cargar un manifiesto descargado anteriormente. Use la tabla de [referencia de manifiesto](#manifest-reference) para comprender la sintaxis y semántica de atributos antiguos y nuevos para que se puedan editar correctamente los atributos que le interesan.
1. Si el flujo de trabajo requiere guardar los manifiestos en el repositorio de origen para su uso posterior, se recomienda reorganizar los manifiestos guardados en el repositorio con el que ve en la experiencia **Registros de aplicaciones**.

## <a name="next-steps"></a>Pasos siguientes

* Para más información sobre la relación entre los objetos de aplicación y entidad de servicio de una aplicación, consulte [Objetos de aplicación y de entidad de servicio de Azure Active Directory](app-objects-and-service-principals.md).
* Consulte el [Glosario para desarrolladores de la plataforma de identidad de Microsoft](developer-glossary.md) para obtener las definiciones de algunos de los conceptos básicos para desarrolladores de la Plataforma de identidad de Microsoft.

Use la siguiente sección de comentarios para especificar opiniones que ayuden a afinar y dar forma al contenido.

<!--article references -->
[AAD-APP-OBJECTS]:app-objects-and-service-principals.md
[AAD-DEVELOPER-GLOSSARY]:developer-glossary.md
[AAD-GROUPS-FOR-AUTHORIZATION]: http://www.dushyantgill.com/blog/2014/12/10/authorization-cloud-applications-using-ad-groups/
[ADD-UPD-RMV-APP]:quickstart-v1-integrate-apps-with-azure-ad.md
[APPLICATION-ENTITY]: /previous-versions/azure/ad/graph/api/entity-and-complex-type-reference#application-entity
[APPLICATION-ENTITY-APP-ROLE]: /previous-versions/azure/ad/graph/api/entity-and-complex-type-reference#approle-type
[APPLICATION-ENTITY-OAUTH2-PERMISSION]: /previous-versions/azure/ad/graph/api/entity-and-complex-type-reference#oauth2permission-type
[AZURE-PORTAL]: https://portal.azure.com
[DEV-GUIDE-TO-AUTH-WITH-ARM]: http://www.dushyantgill.com/blog/2015/05/23/developers-guide-to-auth-with-azure-resource-manager-api/
[GRAPH-API]: active-directory-graph-api.md
[IMPLICIT-GRANT]:v1-oauth2-implicit-grant-flow.md
[INTEGRATING-APPLICATIONS-AAD]: ./quickstart-register-app.md
[O365-PERM-DETAILS]: /graph/permissions-reference
[O365-SERVICE-DAEMON-APPS]: /previous-versions/office/office-365-api/
[RBAC-CLOUD-APPS-AZUREAD]: http://www.dushyantgill.com/blog/2014/12/10/roles-based-access-control-in-cloud-applications-using-azure-ad/