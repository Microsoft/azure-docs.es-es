---
title: Personalice las notificaciones emitidas en tokens para una determinada aplicación de un inquilino de Azure AD (versión preliminar pública)
description: En esta página se describe la asignación de notificaciones de Azure Active Directory.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/28/2019
ms.author: ryanwi
ms.reviewer: paulgarn, hirsin, jeedes, luleon
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8b770ee476fc5c1c334f53904539cc34cf962c62
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65546198"
---
# <a name="how-to-customize-claims-emitted-in-tokens-for-a-specific-app-in-a-tenant-preview"></a>Procedimientos para: Personalizar las notificaciones emitidas en tokens para una determinada aplicación de un inquilino (versión preliminar)

> [!NOTE]
> Esta característica reemplaza a la [personalización de notificaciones](active-directory-saml-claims-customization.md) que se ofrece actualmente a través del portal. Si en la misma aplicación personaliza las notificaciones mediante el portal y usa al mismo tiempo el método de Graph/PowerShell que se detalla en este documento, los tokens emitidos para esa aplicación harán caso omiso de la configuración del portal. Las configuraciones realizadas mediante los métodos que se detallan en este documento no se reflejarán en el portal.

Los administradores de inquilinos usan esta característica para personalizar las notificaciones que se emiten en tokens para una aplicación específica de su inquilino. Puede usar directivas de asignación de notificaciones para:

- Seleccionar las notificaciones que se incluyen en tokens.
- Crear tipos de notificación que aún no existen.
- Elegir o cambiar el origen de los datos emitidos en notificaciones concretas.

> [!NOTE]
> Esta funcionalidad se encuentra actualmente en versión preliminar pública. Debe estar preparado para deshacer o eliminar los cambios. La característica está disponible en cualquier suscripción de Azure Active Directory (Azure AD) durante el período de versión preliminar pública. Pero cuando ya esté disponible con carácter general, algunos aspectos podrían requerir una suscripción Premium de Azure AD. Esta característica permite configurar directivas de asignación de notificaciones para los protocolos WS-Fed, SAML, OAuth y OpenID Connect.

## <a name="claims-mapping-policy-type"></a>Tipo de directiva de asignación de notificaciones

En Azure AD, un objeto de **directiva** representa un conjunto de reglas que se imponen en algunas o todas las aplicaciones de una organización. Cada tipo de directiva tiene una estructura única con un conjunto de propiedades que luego se aplican a los objetos a los que están asignadas.

Una directiva de asignación de notificaciones es un tipo de objeto de **directiva** que modifica las notificaciones emitidas en tokens para aplicaciones concretas.

## <a name="claim-sets"></a>Conjuntos de notificaciones

Hay ciertos conjuntos de notificaciones que definen cómo y cuándo se usan en los tokens.

| Conjunto de notificaciones | DESCRIPCIÓN |
|---|---|
| Conjunto de notificaciones principales | Están presentes en todos los tokens, independientemente de la directiva. Estas notificaciones se consideran también restringidas y no se pueden modificar. |
| Conjunto de notificaciones básicas | Incluye las notificaciones que se emiten de forma predeterminada para los tokens (además del conjunto de notificaciones principales). Puede omitir o modificar las notificaciones básicas utilizando directivas de asignación de notificaciones. |
| Conjunto de notificaciones restringidas | No se pueden modificar con directivas. No se puede cambiar el origen de datos ni se aplica ninguna transformación al generar estas notificaciones. |

### <a name="table-1-json-web-token-jwt-restricted-claim-set"></a>Tabla 1: Conjunto de notificaciones restringidas de JSON Web Token (JWT)

| Tipo de notificación (nombre) |
| ----- |
| _claim_names |
| _claim_sources |
| access_token |
| account_type |
| acr |
| actor |
| actortoken |
| aio |
| altsecid |
| amr |
| app_chain |
| app_displayname |
| app_res |
| appctx |
| appctxsender |
| appid |
| appidacr |
| Aserción |
| at_hash |
| aud |
| auth_data |
| auth_time |
| authorization_code |
| azp |
| azpacr |
| c_hash |
| ca_enf |
| cc |
| cert_token_use |
| client_id |
| cloud_graph_host_name |
| cloud_instance_name |
| cnf |
| código |
| controls |
| credential_keys |
| csr |
| csr_type |
| deviceid |
| dns_names |
| domain_dns_name |
| domain_netbios_name |
| e_exp |
| email |
| endpoint |
| enfpolids |
| exp |
| expires_on |
| grant_type |
| graph |
| group_sids |
| Grupos |
| hasgroups |
| hash_alg |
| home_oid |
| `http://schemas.microsoft.com/ws/2008/06/identity/claims/authenticationinstant` |
| `http://schemas.microsoft.com/ws/2008/06/identity/claims/authenticationmethod` |
| `http://schemas.microsoft.com/ws/2008/06/identity/claims/expiration` |
| `http://schemas.microsoft.com/ws/2008/06/identity/claims/expired` |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress` |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name` |
| `http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier` |
| iat |
| identityprovider |
| idp |
| in_corp |
| instance |
| ipaddr |
| isbrowserhostedapp |
| iss |
| jwk |
| key_id |
| key_type |
| mam_compliance_url |
| mam_enrollment_url |
| mam_terms_of_use_url |
| mdm_compliance_url |
| mdm_enrollment_url |
| mdm_terms_of_use_url |
| nameid |
| nbf |
| netbios_name |
| valor de seguridad |
| oid |
| on_prem_id |
| onprem_sam_account_name |
| onprem_sid |
| openid2_id |
| password |
| platf |
| polids |
| pop_jwk |
| preferred_username |
| previous_refresh_token |
| primary_sid |
| puid |
| pwd_exp |
| pwd_url |
| redirect_uri |
| refresh_token |
| refreshtoken |
| request_nonce |
| Recurso |
| rol |
| roles |
| ámbito |
| scp |
| sid |
| signature |
| signin_state |
| src1 |
| src2 |
| sub |
| tbid |
| tenant_display_name |
| tenant_region_scope |
| thumbnail_photo |
| tid |
| tokenAutologonEnabled |
| trustedfordelegation |
| unique_name |
| upn |
| user_setting_sync_url |
| username |
| uti |
| ver |
| verified_primary_email |
| verified_secondary_email |
| wids |
| win_ver |

### <a name="table-2-saml-restricted-claim-set"></a>Tabla 2: Conjunto de notificaciones restringidas de SAML

| Tipo de notificación (URI) |
| ----- |
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/expiration`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/expired`|
|`http://schemas.microsoft.com/identity/claims/accesstoken`|
|`http://schemas.microsoft.com/identity/claims/openid2_id`|
|`http://schemas.microsoft.com/identity/claims/identityprovider`|
|`http://schemas.microsoft.com/identity/claims/objectidentifier`|
|`http://schemas.microsoft.com/identity/claims/puid`|
|`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier [MR1]`|
|`http://schemas.microsoft.com/identity/claims/tenantid`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/authenticationinstant`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/authenticationmethod`|
|`http://schemas.microsoft.com/accesscontrolservice/2010/07/claims/identityprovider`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/groups`|
|`http://schemas.microsoft.com/claims/groups.link`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/role`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/wids`|
|`http://schemas.microsoft.com/2014/09/devicecontext/claims/iscompliant`|
|`http://schemas.microsoft.com/2014/02/devicecontext/claims/isknown`|
|`http://schemas.microsoft.com/2012/01/devicecontext/claims/ismanaged`|
|`http://schemas.microsoft.com/2014/03/psso`|
|`http://schemas.microsoft.com/claims/authnmethodsreferences`|
|`http://schemas.xmlsoap.org/ws/2009/09/identity/claims/actor`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/samlissuername`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/confirmationkey`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsaccountname`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/primarygroupsid`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/primarysid`|
|`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/authorizationdecision`|
|`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/authentication`|
|`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/sid`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/denyonlyprimarygroupsid`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/denyonlyprimarysid`|
|`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/denyonlysid`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/denyonlywindowsdevicegroup`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsdeviceclaim`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsdevicegroup`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsfqbnversion`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/windowssubauthority`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/windowsuserclaim`|
|`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/x500distinguishedname`|
|`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid`|
|`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/spn`|
|`http://schemas.microsoft.com/ws/2008/06/identity/claims/ispersistent`|
|`http://schemas.xmlsoap.org/ws/2005/05/identity/claims/privatepersonalidentifier`|
|`http://schemas.microsoft.com/identity/claims/scope`|

## <a name="claims-mapping-policy-properties"></a>Propiedades de la directiva de asignación de notificaciones

Para controlar qué reclamaciones se emiten y de dónde provienen los datos, utilice las propiedades de una directiva de asignación de notificaciones. Si no hay ninguna directiva establecida, el sistema emite los tokens que contienen el conjunto de notificaciones principales, el conjunto de notificaciones básicas y las [notificaciones opcionales](active-directory-optional-claims.md) que la aplicación ha decidido recibir.

### <a name="include-basic-claim-set"></a>Inclusión del conjunto de notificaciones básicas

**Cadena:** IncludeBasicClaimSet

**Tipo de datos:** Booleanos (true o false)

**Resumen:** Esta propiedad determina si el conjunto de notificaciones principales se incluye en los tokens afectados por esta directiva.

- Si se establece en True, todas las notificaciones del conjunto de notificaciones básicas se emiten en los tokens afectados por esta directiva. 
- Si se establece en False, las notificaciones del conjunto de notificaciones básicas no se incluyen en los tokens a menos que se agreguen individualmente a la propiedad de esquema de notificaciones de la misma directiva.

> [!NOTE] 
> Las notificaciones del conjunto de notificaciones principales están presentes en todos los tokens, independientemente del valor en que se establezca esta propiedad. 

### <a name="claims-schema"></a>Esquema de notificaciones

**Cadena:** ClaimsSchema

**Tipo de datos:** blob de JSON con una o varias entradas de esquema de notificación

**Resumen:** en esta propiedad se definen las notificaciones que están presentes en los tokens afectados por la directiva, además del conjunto de notificaciones básicas y el conjunto de notificaciones principales.
Para cada entrada de esquema de notificación definida en esta propiedad, se requiere cierta información. Especifique de dónde provienen los datos (**valor** o **par origen/identificador**) y qué notificación se usa para emitir los datos (**tipo de notificación**).

### <a name="claim-schema-entry-elements"></a>Elementos de entrada de esquema de notificación

**Valor:** el elemento Valor define un valor estático para los datos que se emiten en la notificación.

**Par origen/Id.:** los elementos Origen e Id. definen en dónde se originan los datos de la notificación. 

Establezca el elemento Source (Origen) en uno de los valores siguientes: 

- "user": los datos de la notificación son una propiedad del objeto User. 
- "application": los datos de la notificación son una propiedad en la entidad de servicio de la aplicación (cliente). 
- "resource": los datos de la notificación son una propiedad en la entidad de servicio del recurso.
- "audience": los datos de la notificación son una propiedad en la entidad de servicio que es la audiencia del token (la entidad de servicio de recurso o cliente).
- “company”: los datos de la notificación son una propiedad en el objeto Company del inquilino del recurso.
- "transformation": los datos de la notificación proceden de la transformación de notificaciones (vea la sección "Transformación de notificaciones" de este mismo artículo).

Si el origen es una transformación, debe incluirse también el elemento **TransformationID** en la definición de esta notificación.

El elemento ID identifica la propiedad en el origen que proporciona el valor de la notificación. En la tabla siguiente se muestran los valores de ID válidos para cada valor de Source.

#### <a name="table-3-valid-id-values-per-source"></a>Tabla 3: Valores de Id. válidos por origen

| Origen | ID | DESCRIPCIÓN |
|-----|-----|-----|
| Usuario | surname | Nombre de familia |
| Usuario | givenname | Nombre propio |
| Usuario | displayname | Nombre para mostrar |
| Usuario | objectid | ObjectID |
| Usuario | correo | Dirección de correo electrónico |
| Usuario | userprincipalname | Nombre principal de usuario |
| Usuario | department|department|
| Usuario | onpremisessamaccountname | Nombre de cuenta SAM local |
| Usuario | netbiosname| Nombre de NetBios |
| Usuario | dnsdomainname | Nombre de dominio DNS |
| Usuario | onpremisesecurityidentifier | Identificador de seguridad local |
| Usuario | companyname| Nombre de la organización |
| Usuario | streetaddress | Dirección |
| Usuario | postalcode | Código postal |
| Usuario | preferredlanguange | Idioma preferido |
| Usuario | onpremisesuserprincipalname | UPN local |
| Usuario | mailNickname | Alias de correo |
| Usuario | extensionattribute1 | Atributo de extensión 1 |
| Usuario | extensionattribute2 | Atributo de extensión 2 |
| Usuario | extensionattribute3 | Atributo de extensión 3 |
| Usuario | extensionattribute4 | Atributo de extensión 4 |
| Usuario | extensionattribute5 | Atributo de extensión 5 |
| Usuario | extensionattribute6 | Atributo de extensión 6 |
| Usuario | extensionattribute7 | Atributo de extensión 7 |
| Usuario | extensionattribute8 | Atributo de extensión 8 |
| Usuario | extensionattribute9 | Atributo de extensión 9 |
| Usuario | extensionattribute10 | Atributo de extensión 10 |
| Usuario | extensionattribute11 | Atributo de extensión 11 |
| Usuario | extensionattribute12 | Atributo de extensión 12 |
| Usuario | extensionattribute13 | Atributo de extensión 13 |
| Usuario | extensionattribute14 | Atributo de extensión 14 |
| Usuario | extensionattribute15 | Atributo de extensión 15 |
| Usuario | othermail | Otro correo |
| Usuario | country | País |
| Usuario | city | City |
| Usuario | state | Estado |
| Usuario | jobtitle | Puesto |
| Usuario | employeeid | Id. de empleado |
| Usuario | facsimiletelephonenumber | Número de teléfono de fax |
| application, resource, audience | displayname | Nombre para mostrar |
| application, resource, audience | objected | ObjectID |
| application, resource, audience | tags | Etiqueta de entidad de servicio |
| Empresa | tenantcountry | País del inquilino |

**TransformationID:** el elemento TransformationID solo debe indicarse si el elemento Source está establecido en “transformation”.

- Este elemento debe coincidir con el elemento ID de la entrada de transformación en la propiedad **ClaimsTransformation** que define cómo se generan los datos de esta notificación.

**Claim Type:** los elementos **JwtClaimType** y **SamlClaimType** definen la notificación a la que esta entrada de esquema de notificación hace referencia.

- El elemento JwtClaimType tiene que contener el nombre de la notificación que se va a emitir en los token JWT.
- El elemento SamlClaimType debe contener el identificador URI de la notificación que se va a emitir en los token SAML.

> [!NOTE]
> Los nombres e identificadores URI de las notificaciones del conjunto de notificaciones restringidas no se pueden usar en los elementos de tipo de notificación. Para más información, vea la sección "Excepciones y restricciones" más adelante en este artículo.

### <a name="claims-transformation"></a>Transformación de notificaciones

**Cadena:** ClaimsTransformation

**Tipo de datos:** blob de JSON, con una o varias entradas de transformación 

**Resumen:** use esta propiedad para aplicar transformaciones comunes a datos de origen para generar los datos de salida de las notificaciones especificadas en el esquema de notificaciones.

**ID:** use el elemento ID para hacer referencia a esta entrada de transformación en el elemento TransformationID del esquema de notificaciones. Este valor debe ser único para cada entrada de transformación comprendida en la directiva.

**TransformationMethod:** el elemento TransformationMethod identifica la operación que se ejecuta para generar los datos de la notificación.

En función del método elegido, se espera un conjunto de entradas y salidas. Defina las entradas y salidas utilizando los elementos **InputClaims**, **InputParameters** y **OutputClaims**.

#### <a name="table-4-transformation-methods-and-expected-inputs-and-outputs"></a>Tabla 4: Métodos de transformación y entradas y salidas previstas

|TransformationMethod|Entrada prevista|Salida prevista|DESCRIPCIÓN|
|-----|-----|-----|-----|
|Unir|string1, string2, separator|outputClaim|Se combinan las cadenas de entrada mediante un separador entre ellas. Por ejemplo: string1:"foo@bar.com" , string2:"sandbox" , separator:"." da como resultado outputClaim:"foo@bar.com.sandbox"|
|ExtractMailPrefix|correo|outputClaim|Extrae la parte local de una dirección de correo electrónico. Por ejemplo: mail:"foo@bar.com" da como resultado outputClaim:"foo". Si no existe ningún signo \@, la cadena de entrada original se devuelve tal y como está.|

**InputClaims:** use un elemento InputClaims para pasar los datos de una entrada de esquema de notificación a una transformación. Tiene dos atributos: **ClaimTypeReferenceId** y **TransformationClaimType**.

- **ClaimTypeReferenceId** se combina con el elemento ID de la entrada de esquema de notificación para buscar la notificación de entrada adecuada. 
- **TransformationClaimType** se usa para asignar un nombre único a esta entrada. Este nombre debe coincidir con una de las entradas esperadas para el método de transformación.

**InputParameters:** use un elemento InputParameters para pasar un valor constante a una transformación. Tiene dos atributos: **Valor** e **ID**.

- **Value** es el valor constante real que se pasa.
- **ID** se usa para asignar un nombre único a esta entrada. Este nombre debe coincidir con una de las entradas esperadas del método de transformación.

**OutputClaims:** use un elemento OutputClaims para contener los datos generados por una transformación y vincularlos a una entrada de esquema de notificación. Tiene dos atributos: **ClaimTypeReferenceId** y **TransformationClaimType**.

- **ClaimTypeReferenceId** se combina con el elemento ID de la entrada de esquema de notificación para buscar la notificación de salida adecuada.
- **TransformationClaimType** se usa para asignar un nombre único a la salida. Este nombre debe coincidir con una de las salidas esperadas del método de transformación.

### <a name="exceptions-and-restrictions"></a>Excepciones y restricciones

**NameID y UPN de SAML:** Los atributos desde los que se originan los valores de NameID y UPN, y las transformaciones de notificaciones que se permiten, están limitados. Consulte las tablas 5 y 6 para ver los valores permitidos.

#### <a name="table-5-attributes-allowed-as-a-data-source-for-saml-nameid"></a>Tabla 5: Atributos permitidos como origen de datos en NameID de SAML

|Origen|ID|DESCRIPCIÓN|
|-----|-----|-----|
| Usuario | correo|Dirección de correo electrónico|
| Usuario | userprincipalname|Nombre principal de usuario|
| Usuario | onpremisessamaccountname|Nombre de cuenta SAM local|
| Usuario | employeeid|Id. de empleado|
| Usuario | extensionattribute1 | Atributo de extensión 1 |
| Usuario | extensionattribute2 | Atributo de extensión 2 |
| Usuario | extensionattribute3 | Atributo de extensión 3 |
| Usuario | extensionattribute4 | Atributo de extensión 4 |
| Usuario | extensionattribute5 | Atributo de extensión 5 |
| Usuario | extensionattribute6 | Atributo de extensión 6 |
| Usuario | extensionattribute7 | Atributo de extensión 7 |
| Usuario | extensionattribute8 | Atributo de extensión 8 |
| Usuario | extensionattribute9 | Atributo de extensión 9 |
| Usuario | extensionattribute10 | Atributo de extensión 10 |
| Usuario | extensionattribute11 | Atributo de extensión 11 |
| Usuario | extensionattribute12 | Atributo de extensión 12 |
| Usuario | extensionattribute13 | Atributo de extensión 13 |
| Usuario | extensionattribute14 | Atributo de extensión 14 |
| Usuario | extensionattribute15 | Atributo de extensión 15 |

#### <a name="table-6-transformation-methods-allowed-for-saml-nameid"></a>Tabla 6: Métodos de transformación permitidos en NameID de SAML

| TransformationMethod | Restricciones |
| ----- | ----- |
| ExtractMailPrefix | None |
| Unir | El sufijo que se combine debe ser un dominio comprobado del inquilino del recurso. |

### <a name="custom-signing-key"></a>Clave de firma de personalizada

Debe asignarse una clave de firma personalizada al objeto de entidad de servicio para que una directiva de asignación de notificaciones surta efecto. Esto garantiza que el creador de la directiva de asignación de notificaciones es el que ha modificado los tokens y protege a las aplicaciones frente a directivas de asignación de notificaciones creadas por actores malintencionados.  Las aplicaciones que tienen notificaciones asignación habilitada debe comprobar un URI especial para su token de claves de firma anexando `appid={client_id}` a sus [las solicitudes de metadatos de OpenID Connect](v2-protocols-oidc.md#fetch-the-openid-connect-metadata-document).  

### <a name="cross-tenant-scenarios"></a>Escenarios de varios inquilinos

Las directivas de asignación de notificaciones no se aplican a los usuarios invitados. Si un usuario invitado intenta acceder a una aplicación que tiene una directiva de asignación de notificaciones asignada a su entidad de servicio, se emite el token predeterminado (la directiva no tiene ningún efecto).

## <a name="claims-mapping-policy-assignment"></a>Asignación de directivas de asignación de notificaciones

Las directivas de asignación de notificaciones solo se pueden asignar a objetos de entidades de servicio.

### <a name="example-claims-mapping-policies"></a>Directivas de asignación de notificaciones de ejemplo

En Azure AD hay muchos escenarios posibles en los que se pueden personalizar las notificaciones emitidas en tokens para entidades de servicio concretas. En esta sección se abordan algunos escenarios comunes que pueden ayudarle a entender cómo usar el tipo de directiva de asignación de notificaciones.

#### <a name="prerequisites"></a>Requisitos previos

En los ejemplos siguientes, va a crear, actualizar, vincular y eliminar directivas de entidades de servicio. Si no está familiarizado con Azure AD, es conveniente que [aprenda a obtener un inquilino de Azure AD](quickstart-create-new-tenant.md) antes de continuar con estos ejemplos.

Para comenzar, realice uno de los pasos siguientes:

1. Descargue la [versión preliminar pública más reciente del módulo de PowerShell de Azure AD](https://www.powershellgallery.com/packages/AzureADPreview).
1. Ejecute el comando Connect para iniciar sesión en la cuenta de administrador de Azure AD. Ejecute este comando cada vez que inicie una nueva sesión.

   ``` powershell
   Connect-AzureAD -Confirm
   ```
1. Ejecute el siguiente comando para ver todas las directivas que se han creado en la organización. Se recomienda ejecutar este comando después de la mayoría de las operaciones en los escenarios siguientes, para comprobar que las directivas se van a crear según lo previsto.

   ``` powershell
   Get-AzureADPolicy
   ```

#### <a name="example-create-and-assign-a-policy-to-omit-the-basic-claims-from-tokens-issued-to-a-service-principal"></a>Ejemplo: Crear y asignar una directiva que omita las notificaciones básicas de los tokens emitidos para una entidad de servicio

En este ejemplo se crea una directiva que quita el conjunto de notificaciones básicas de los tokens emitidos para entidades de servicio vinculadas.

1. Cree una directiva de asignación de notificaciones. Esta directiva, que se vincula a entidades de servicio concretas, quita el conjunto de notificaciones básicas de los tokens.
   1. Ejecute este comando para crear la directiva: 
    
      ``` powershell
      New-AzureADPolicy -Definition @('{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"false"}}') -DisplayName "OmitBasicClaims" -Type "ClaimsMappingPolicy"
      ```
   2. Para ver la nueva directiva y obtener su ObjectID, ejecute el siguiente comando:
    
      ``` powershell
      Get-AzureADPolicy
      ```
1. Asigne la directiva a su entidad de servicio. También necesita obtener el valor de ObjectId de su entidad de servicio.
   1. Para ver todas las entidades de servicio de su organización, puede consultar Microsoft Graph. O bien, en Azure AD Graph Explorer, inicie sesión en su cuenta de Azure AD.
   2. Cuando tenga el valor de ObjectId de la entidad de servicio, ejecute el siguiente comando:  
     
      ``` powershell
      Add-AzureADServicePrincipalPolicy -Id <ObjectId of the ServicePrincipal> -RefObjectId <ObjectId of the Policy>
      ```

#### <a name="example-create-and-assign-a-policy-to-include-the-employeeid-and-tenantcountry-as-claims-in-tokens-issued-to-a-service-principal"></a>Ejemplo: creación y asignación de una directiva que incluye EmployeeID y TenantCountry como notificaciones en los tokens emitidos para una entidad de servicio

En este ejemplo se crea una directiva que agrega EmployeeID y TenantCountry a los tokens emitidos para entidades de servicio vinculadas. EmployeeID se emite como tipo de notificación de nombre en los tokens SAML y JWT. TenantCountry se emite como tipo de notificación de país en los tokens SAML y JWT. En este ejemplo se sigue incluyendo el conjunto de notificaciones básicas en los tokens.

1. Cree una directiva de asignación de notificaciones. Esta directiva, que se vincula a entidades de servicio concretas, agrega las notificaciones EmployeeID y TenantCountry a los tokens.
   1. Ejecute el siguiente comando para crear la directiva:  
     
      ``` powershell
      New-AzureADPolicy -Definition @('{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"true", "ClaimsSchema": [{"Source":"user","ID":"employeeid","SamlClaimType":"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name","JwtClaimType":"name"},{"Source":"company","ID":"tenantcountry","SamlClaimType":"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/country","JwtClaimType":"country"}]}}') -DisplayName "ExtraClaimsExample" -Type "ClaimsMappingPolicy"
      ```
    
   2. Para ver la nueva directiva y obtener su ObjectID, ejecute el siguiente comando:
     
      ``` powershell  
      Get-AzureADPolicy
      ```
1. Asigne la directiva a su entidad de servicio. También necesita obtener el valor de ObjectId de su entidad de servicio. 
   1. Para ver todas las entidades de servicio de su organización, puede consultar Microsoft Graph. O bien, en Azure AD Graph Explorer, inicie sesión en su cuenta de Azure AD.
   2. Cuando tenga el valor de ObjectId de la entidad de servicio, ejecute el siguiente comando:  
     
      ``` powershell
      Add-AzureADServicePrincipalPolicy -Id <ObjectId of the ServicePrincipal> -RefObjectId <ObjectId of the Policy>
      ```

#### <a name="example-create-and-assign-a-policy-that-uses-a-claims-transformation-in-tokens-issued-to-a-service-principal"></a>Ejemplo: creación y asignación de una directiva que usa una transformación de notificaciones en los tokens emitidos para una entidad de servicio

En este ejemplo se crea una directiva que emite una notificación "JoinedData" personalizada para los tokens JWT emitidos para entidades de servicio vinculadas. Esta notificación contiene un valor creado mediante la combinación de los datos almacenados en el atributo extensionattribute1 en el objeto de usuario que incluye ".sandbox". En este ejemplo se excluye el conjunto de notificaciones básicas en los tokens.

1. Cree una directiva de asignación de notificaciones. Esta directiva, que se vincula a entidades de servicio concretas, agrega las notificaciones EmployeeID y TenantCountry a los tokens.
   1. Ejecute el siguiente comando para crear la directiva:
     
      ``` powershell
      New-AzureADPolicy -Definition @('{"ClaimsMappingPolicy":{"Version":1,"IncludeBasicClaimSet":"true", "ClaimsSchema":[{"Source":"user","ID":"extensionattribute1"},{"Source":"transformation","ID":"DataJoin","TransformationId":"JoinTheData","JwtClaimType":"JoinedData"}],"ClaimsTransformations":[{"ID":"JoinTheData","TransformationMethod":"Join","InputClaims":[{"ClaimTypeReferenceId":"extensionattribute1","TransformationClaimType":"string1"}], "InputParameters": [{"ID":"string2","Value":"sandbox"},{"ID":"separator","Value":"."}],"OutputClaims":[{"ClaimTypeReferenceId":"DataJoin","TransformationClaimType":"outputClaim"}]}]}}') -DisplayName "TransformClaimsExample" -Type "ClaimsMappingPolicy"
      ```
    
   2. Para ver la nueva directiva y obtener su ObjectID, ejecute el siguiente comando: 
     
      ``` powershell
      Get-AzureADPolicy
      ```
1. Asigne la directiva a su entidad de servicio. También necesita obtener el valor de ObjectId de su entidad de servicio. 
   1. Para ver todas las entidades de servicio de su organización, puede consultar Microsoft Graph. O bien, en Azure AD Graph Explorer, inicie sesión en su cuenta de Azure AD.
   2. Cuando tenga el valor de ObjectId de la entidad de servicio, ejecute el siguiente comando: 
     
      ``` powershell
      Add-AzureADServicePrincipalPolicy -Id <ObjectId of the ServicePrincipal> -RefObjectId <ObjectId of the Policy>
      ```

## <a name="see-also"></a>Vea también

Para obtener información sobre cómo personalizar las notificaciones emitidas en el token SAML mediante el portal de Azure, consulte [Cómo: Personalizar las notificaciones emitidas en el token SAML para aplicaciones empresariales](active-directory-saml-claims-customization.md)