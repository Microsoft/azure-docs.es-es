---
title: Definición de un perfil técnico de SAML en una directiva personalizada
titleSuffix: Azure AD B2C
description: Defina un perfil técnico de SAML en una directiva personalizada en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/04/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 30c64e4cf467f4e505327414e15b23ee2c6d1543
ms.sourcegitcommit: f845ca2f4b626ef9db73b88ca71279ac80538559
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89611652"
---
# <a name="define-a-saml-identity-provider-technical-profile-in-an-azure-active-directory-b2c-custom-policy"></a>Definición de un perfil técnico de proveedor de identidades de SAML en una directiva personalizada en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Azure Active Directory B2C (Azure AD B2C) proporciona compatibilidad con el proveedor de identidades SAML 2.0. En este artículo se describen los detalles para que un perfil técnico interactúe con un proveedor de notificaciones que admita este protocolo estandarizado. Con un perfil técnico de SAML puede federarse con un proveedor de identidades basado en SAML, como [ADFS](identity-provider-adfs2016-custom.md) y [Salesforce](identity-provider-salesforce-custom.md). Esta federación permite a sus usuarios iniciar sesión con sus identidades de redes sociales o de empresa existentes.

## <a name="metadata-exchange"></a>Intercambio de metadatos

Los metadatos son información que se usa en el protocolo SAML para exponer la configuración de una entidad SAML, como un proveedor de servicios o de identidades. Los metadatos definen la ubicación de los servicios, como el inicio y el cierre de sesión, los certificados, el método de inicio de sesión y mucho más. El proveedor de identidades usa los metadatos para saber cómo comunicarse con Azure AD B2C. Los metadatos se configuran en formato XML y pueden estar firmados con una firma digital para que la otra entidad pueda validar la integridad de los metadatos. Cuando Azure AD B2C se federa con un proveedor de identidades SAML, actúa como proveedor de servicios iniciando una solicitud SAML y esperando una respuesta SAML. En algunos casos, acepta la autenticación de SAML no solicitada, que también se conoce como autenticación iniciada por el proveedor de identidad.

Los metadatos se pueden configurar en las dos entidades como “Metadatos estáticos” o “Metadatos dinámicos”. En el modo estático, los metadatos se copian por completo de una entidad y se establecen en la otra. En el modo dinámico, se establece la URL en los metadatos mientras la otra entidad lee la configuración de forma dinámica. Los principios son los mismos: establecer los metadatos del perfil técnico de Azure AD B2C en el proveedor de identidades y establecer los metadatos del proveedor de identidades en Azure AD B2C.

Cada proveedor de identidades SAML sigue distintos pasos para exponer y establecer el proveedor de servicios, en este caso Azure AD B2C, y los metadatos de Azure AD B2C en el proveedor de identidades. Consulte la documentación de su proveedor de identidades para obtener instrucciones sobre cómo hacerlo.

En el ejemplo siguiente se muestra una dirección URL a los metadatos SAML de un perfil técnico de Azure AD B2C:

```
https://your-tenant-name.b2clogin.com/your-tenant-name/your-policy/samlp/metadata?idptp=your-technical-profile
```

Reemplace los siguientes valores:

- **your-tenant-name** por su nombre de inquilino, por ejemplo, fabrikam.b2clogin.com.
- **your-policy** por el nombre de la directiva. Use la directiva donde configura el perfil técnico del proveedor de SAML o una directiva que hereda de esa directiva.
- **your-technical-profile** por el nombre del perfil técnico del proveedor de identidades SAML.

## <a name="digital-signing-certificates-exchange"></a>Intercambio de certificados de firma digital

Para fomentar la confianza entre Azure AD B2C y el proveedor de identidades SAML, debe proporcionar un certificado X509 válido junto con la clave privada. Cargue el certificado con la clave privada (archivo .pfx) en el almacén de claves de directiva de Azure AD B2C. Azure AD B2C firma digitalmente la solicitud de inicio de sesión de SAML usando el certificado que proporcione.

El certificado se usa de las formas siguientes:

- Azure AD B2C genera y firma una solicitud de SAML con la clave privada Azure AD B2C del certificado. La solicitud de SAML se envía al proveedor de identidades, que valida la solicitud con la clave pública Azure AD B2C del certificado. Se puede acceder al certificado público de Azure AD B2C a través de los metadatos del perfil técnico. También puede cargar manualmente el archivo .cer en el proveedor de identidades SAML.
- El proveedor de identidades firma los datos enviados a Azure AD B2C usando la clave privada del certificado del proveedor de identidades. Azure AD B2C valida los datos mediante el certificado público del proveedor de identidades. Cada proveedor de identidades ofrece distintos pasos para la configuración. Consulte la documentación de los proveedores de identidades para obtener instrucciones sobre cómo hacerlo. En Azure AD B2C, la directiva necesita acceso a la clave pública del certificado mediante los metadatos del proveedor de identidades.

En la mayoría de los casos se acepta un certificado autofirmado. En los entornos de producción, se recomienda usar un certificado X509 emitido por una entidad de certificación. Además, como se describe más adelante en este documento, en los entornos que no son de producción, puede deshabilitar la firma de SAML en ambos lados.

En el siguiente diagrama se muestra el intercambio entre metadatos y certificados:

![intercambio entre metadatos y certificados](media/saml-technical-profile/technical-profile-idp-saml-metadata.png)

## <a name="digital-encryption"></a>Cifrado digital

Para cifrar la aserción de respuesta SAML, el proveedor de identidades siempre usa una clave pública de un certificado de cifrado en un perfil técnico de Azure AD B2C. Cuando Azure AD B2C necesita descifrar los datos, usa la parte privada del certificado de cifrado.

Para cifrar la aserción de respuesta SAML:

1. Cargue un certificado X509 con la clave privada (archivo .pfx) en el almacén de claves de directiva de Azure AD B2C.
2. Añada un elemento **CryptographicKey** con un identificador de `SamlAssertionDecryption` a la colección **CryptographicKeys** del perfil técnico. Asigne el nombre de la clave de directiva que creó en el paso 1 a **StorageReferenceId**.
3. Establezca los metadatos del perfil técnico **WantsEncryptedAssertions** en `true`.
4. Actualice el proveedor de identidades con los nuevos metadatos del perfil técnico de Azure AD B2C. La propiedad **use** de **KeyDescriptor** debería estar establecida en `encryption` y contener la clave pública del certificado.

En el ejemplo siguiente se muestra la sección del descriptor de clave de los metadatos de SAML usados para el cifrado:

```xml
<KeyDescriptor use="encryption">
  <KeyInfo xmlns="https://www.w3.org/2000/09/xmldsig#">
    <X509Data>
      <X509Certificate>valid certificate</X509Certificate>
    </X509Data>
  </KeyInfo>
</KeyDescriptor>
```

## <a name="protocol"></a>Protocolo

El atributo **Name** del elemento Protocol se debe establecer en `SAML2`.

## <a name="output-claims"></a>Notificaciones de salida

El elemento **OutputClaims** contiene una lista de notificaciones que devuelve el proveedor de identidades de SAML en la sección `AttributeStatement`. Puede que tenga que asignar el nombre de la notificación definida en la directiva al nombre definido en el proveedor de identidades. También puede incluir notificaciones no especificadas por el proveedor de identidades, siempre que establezca el atributo `DefaultValue`.

### <a name="subject-name-output-claim"></a>Notificación de salida del nombre de sujeto

Para leer la aserción SAML **NamedId** en **Subject** como si fuera una notificación normalizada, establezca la notificación **PartnerClaimType** en el valor del atributo `SPNameQualifier`. No el atributo `SPNameQualifier` no aparece, establezca la notificación **PartnerClaimType** en el valor del atributo `NameQualifier`. 


Aserción SAML: 

```xml
<saml:Subject>
  <saml:NameID SPNameQualifier="http://your-idp.com/unique-identifier" Format="urn:oasis:names:tc:SAML:2.0:nameid-format:transient">david@contoso.com</saml:NameID>
    <SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
      <SubjectConfirmationData InResponseTo="_cd37c3f2-6875-4308-a9db-ce2cf187f4d1" NotOnOrAfter="2020-02-15T16:23:23.137Z" Recipient="https://your-tenant.b2clogin.com/your-tenant.onmicrosoft.com/B2C_1A_TrustFrameworkBase/samlp/sso/assertionconsumer" />
    </SubjectConfirmation>
  </saml:SubjectConfirmation>
</saml:Subject>
```

Notificación de salida:

```xml
<OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="http://your-idp.com/unique-identifier" />
```

Si los atributos `SPNameQualifier` o `NameQualifier` no se encuentran en la aserción SAML, establezca la notificación **PartnerClaimType** en `assertionSubjectName`. Asegúrese de que **NameId** es el primer valor en el XML de la aserción. Cuando defina más de una aserción, Azure AD B2C toma el valor del tema de la última aserción.

El ejemplo siguiente muestra las notificaciones devueltas por el proveedor de identidades de SAML:

- La notificación **issuerUserId** se asigna a la notificación **assertionSubjectName**.
- La notificación **first_name** se asigna a la notificación **givenName**.
- La notificación **last_name** se asigna a la notificación **surname**.
- La notificación **displayName** se asigna a la notificación **name**.
- La notificación **email** sin asignación de nombre.

El perfil técnico también muestra la notificaciones no proporcionadas por el proveedor de identidades:

- La notificación **identityProvider** que contiene el nombre del proveedor de identidades.
- La notificación **authenticationSource** con un valor predeterminado de **socialIdpAuthentication**.

```xml
<OutputClaims>
  <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="assertionSubjectName" />
  <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="first_name" />
  <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="last_name" />
  <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
  <OutputClaim ClaimTypeReferenceId="email"  />
  <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="contoso.com" />
  <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" />
</OutputClaims>
```

El elemento **OutputClaimsTransformations** puede contener una colección de elementos **OutputClaimsTransformation** que se usan para modificar las notificaciones de salida o para generar nuevas.

## <a name="metadata"></a>Metadatos

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| PartnerEntity | Sí | URL de los metadatos del proveedor de identidades SAML. Copie los metadatos del proveedor de identidades y agréguelo en el elemento CDATA `<![CDATA[Your IDP metadata]]>` |
| WantsSignedRequests | No | Indica si el perfil técnico requiere que todas las solicitudes de autenticación de salida estén firmadas. Valores posibles: `true` o `false`. El valor predeterminado es `true`. Cuando el valor se establece en `true`, hay que especificar la clave criptográfica **SamlMessageSigning** y todas las solicitudes de autenticación de salida deben estar firmadas. Si el valor se establece en `false`, se omiten los parámetros **SigAlg** y **Signature** (cadena de consulta o parámetro posterior) de la solicitud. Estos metadatos también controlan el atributo **AuthnRequestsSigned** de los metadatos, que se recogen en los metadatos del perfil técnico de Azure AD B2C que se comparte con el proveedor de identidades. Azure AD B2C no firma la solicitud si el valor de **WantsSignedRequests** en los metadatos del perfil técnico se establece en `false` y los metadatos del proveedor de identidades **WantAuthnRequestsSigned** están establecidos en `false` o no se han especificado. |
| XmlSignatureAlgorithm | No | El método que Azure AD B2C usa para firmar la solicitud SAML. Estos metadatos controlan el valor del parámetro **SigAlg** (cadena de consulta o parámetro posterior) en la solicitud SAML. Valores posibles: `Sha256`, `Sha384`, `Sha512` o `Sha1`. Asegúrese de configurar el algoritmo de firma en ambos lados con el mismo valor. Use solo el algoritmo que admite el certificado. |
| WantsSignedAssertions | No | Indica si el perfil técnico requiere que todas las aserciones entrantes estén firmadas. Valores posibles: `true` o `false`. El valor predeterminado es `true`. Si el valor se establece en `true`, la sección `saml:Assertion` de todas las aserciones enviadas por el proveedor de entidades a Azure AD B2C debe estar firmada. Si el valor se establece en `false`, no es necesario que el proveedor de identidades firme las aserciones, pero aunque lo haga, Azure AD B2C no validará la firma. Estos metadatos también controlan la marca de metadatos **WantsAssertionsSigned**, que se recoge en los metadatos del perfil técnico de Azure AD B2C que se comparte con el proveedor de identidades. Si deshabilita la validación de las aserciones, también puede interesarle deshabilitar la validación de la firma de respuesta (para obtener más información, vea **ResponsesSigned**). |
| ResponsesSigned | No | Valores posibles: `true` o `false`. El valor predeterminado es `true`. Si el valor se establece en `false`, no es necesario que el proveedor de identidades firme la respuesta de SAML, pero incluso si lo hace, Azure AD B2C no validará la firma. Si el valor se establece en `true`, la respuesta de SAML enviada por el proveedor de identidades a Azure AD B2C está firmada y debe validarse. Si deshabilita la validación de la respuesta de SAML, es posible que también le interese deshabilitar la validación de la firma de aserción (para obtener más información, vea **WantsSignedAssertions**). |
| WantsEncryptedAssertions | No | Indica si el perfil técnico requiere que todas las aserciones entrantes estén cifradas. Valores posibles: `true` o `false`. El valor predeterminado es `false`. Si el valor se establece en `true`, las aserciones enviadas por el proveedor de identidades a Azure AD B2C deben estar firmadas y debe especificarse la clave criptográfica **SamlAssertionDecryption**. Si el valor se establece en `true`, los metadatos del perfil técnico de Azure AD B2C incluyen la sección **cifrado**. El proveedor de identidades lee los metadatos y cifra la aserción de respuesta de SAML con la clave pública que se proporciona en los metadatos del perfil técnico de Azure AD B2C. Si habilita el cifrado de aserciones, es posible que también tenga que deshabilitar la validación de firma de respuesta (para obtener más información, vea **ResponsesSigned**). |
| NameIdPolicyFormat | No | Especifica restricciones en el nombre del identificador que se usará para representar el tema solicitado. Si se omite, se puede usar cualquier tipo de identificador compatible con el proveedor de identidades para el tema solicitado. Por ejemplo, `urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified`. **NameIdPolicyFormat** puede usarse con **NameIdPolicyAllowCreate**. Examine la documentación de su proveedor de identidades para obtener instrucciones sobre qué directivas de id. de nombre se admiten. |
| NameIdPolicyAllowCreate | No | Cuando se usa **NameIdPolicyFormat**, también puede especificar la propiedad `AllowCreate` de **NameIDPolicy**. El valor de estos metadatos es `true` o `false`, e indica si el proveedor de identidades tiene permiso para crear una nueva cuenta durante el flujo de inicio de sesión. Consulte la documentación de su proveedor de identidades para obtener instrucciones sobre cómo hacerlo. |
| AuthenticationRequestExtensions | No | Elementos de extensión de mensajes de protocolo opcionales acordados entre Azure AD BC y el proveedor de identidades. La extensión se presenta en formato XML. Debe agregar los datos XML en del elemento CDATA `<![CDATA[Your IDP metadata]]>`. Consulte la documentación del proveedor de identidades para ver si se admite el elemento de extensiones. |
| IncludeAuthnContextClassReferences | No | Especifica una o varias referencias de URI que identifican las clases de contexto de autenticación. Por ejemplo, para permitir que un usuario inicie sesión solo con el nombre de usuario y la contraseña, establezca el valor en `urn:oasis:names:tc:SAML:2.0:ac:classes:Password`. Para permitir el inicio de sesión mediante el nombre de usuario y la contraseña a través de una sesión protegida (SSL/TLS), especifique `PasswordProtectedTransport`. Examine la documentación del proveedor de identidades para obtener instrucciones sobre las URI de **AuthnContextClassRef** que se admiten. Especifique varios URI como una lista delimitada por comas. |
| IncludeKeyInfo | No | Indica si la solicitud de autenticación SAML contiene la clave pública del certificado cuando el enlace se establece en `HTTP-POST`. Valores posibles: `true` o `false`. |
| IncludeClaimResolvingInClaimsHandling  | No | En el caso de las notificaciones de entrada y salida, especifica si se incluye la [resolución de notificaciones](claim-resolver-overview.md) en el perfil técnico. Valores posibles: `true` o `false`  (valor predeterminado). Si desea utilizar un solucionador de notificaciones en el perfil técnico, establézcalo en `true`. |

## <a name="cryptographic-keys"></a>Claves de cifrado

El elemento **CryptographicKeys** contiene los siguientes atributos:

| Atributo |Obligatorio | Descripción |
| --------- | ----------- | ----------- |
| SamlMessageSigning |Sí | El certificado X509 (conjunto de claves RSA) que se va a usar para firmar los mensajes SAML. Azure AD B2C usa esta clave para firmar las solicitudes y enviarlas al proveedor de identidades. |
| SamlAssertionDecryption |Sí | El certificado X509 (conjunto de claves RSA) que se va a usar para descifrar los mensajes SAML. El proveedor de identidades debe proporcionar este certificado. Azure AD B2C usa este certificado para descifrar los datos enviados por el proveedor de identidades. |
| MetadataSigning |No | El certificado X509 (conjunto de claves RSA) que se va a usar para firmar los metadatos SAML. Azure AD B2C usa esta clave para firmar los metadatos.  |

## <a name="saml-entityid-customization"></a>Personalización de entityID de SAML

Si tiene varias aplicaciones SAML que dependen de distintos valores de entityID, puede reemplazar el valor de `issueruri` en el archivo de usuario de confianza. Para ello, copie el perfil técnico con el identificador "Saml2AssertionIssuer" del archivo base y reemplace el valor `issueruri`.

> [!TIP]
> Copie la sección `<ClaimsProviders>` de la base y conserve estos elementos en el proveedor de notificaciones: `<DisplayName>Token Issuer</DisplayName>`, `<TechnicalProfile Id="Saml2AssertionIssuer">` y `<DisplayName>Token Issuer</DisplayName>`.
 
Ejemplo:

```xml
   <ClaimsProviders>   
    <ClaimsProvider>
      <DisplayName>Token Issuer</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Saml2AssertionIssuer">
          <DisplayName>Token Issuer</DisplayName>
          <Metadata>
            <Item Key="IssuerUri">customURI</Item>
          </Metadata>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
  </ClaimsProviders>
  <RelyingParty>
    <DefaultUserJourney ReferenceId="SignUpInSAML" />
    <TechnicalProfile Id="PolicyProfile">
      <DisplayName>PolicyProfile</DisplayName>
      <Protocol Name="SAML2" />
      <Metadata>
     …
```

## <a name="next-steps"></a>Pasos siguientes

Vea los artículos siguientes para obtener ejemplos de cómo trabajar con proveedores de identidades SAML en Azure AD B2C:

- [Agregar ADFS como proveedor de identidades de SAML mediante las directivas personalizadas](identity-provider-adfs2016-custom.md)
- [Inicio de sesión mediante cuentas de Salesforce a través de SAML](identity-provider-salesforce-custom.md)
