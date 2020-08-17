---
title: 'Localization: Azure Active Directory B2C'
description: Especifique el elemento Localization de una directiva personalizada en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 04/20/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 84850b7d44033a2759c51c5c6b9c53d1c945a99d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87005385"
---
# <a name="localization-element"></a>Elemento Localization

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

El elemento **Localization** permite incluir varios idiomas o configuraciones regionales en la directiva para los recorridos del usuario. Admitir la localización en las directivas le permite:

- Configurar la lista explícita de los idiomas admitidos en una directiva y seleccionar un idioma predeterminado.
- Proporcionar cadenas y colecciones específicas del idioma.

```xml
<Localization Enabled="true">
  <SupportedLanguages DefaultLanguage="en" MergeBehavior="ReplaceAll">
    <SupportedLanguage>en</SupportedLanguage>
    <SupportedLanguage>es</SupportedLanguage>
  </SupportedLanguages>
  <LocalizedResources Id="api.localaccountsignup.en">
  <LocalizedResources Id="api.localaccountsignup.es">
  ...
```

El elemento **Localization** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| habilitado | No | Valores posibles: `true` o `false`. |

El elemento **Localization** contiene los siguientes elementos XML:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| SupportedLanguages | 1:n | Lista de idiomas admitidos. |
| LocalizedResources | 0:n | Lista de recursos localizados. |

## <a name="supportedlanguages"></a>SupportedLanguages

El elemento **SupportedLanguages** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| DefaultLanguage | Sí | El idioma que se usará como el predeterminado para los recursos localizados. |
| MergeBehavior | No | Valores de enumeración de los valores combinados con cualquier elemento ClaimType presente en una directiva principal con el mismo identificador. Use este atributo al sobrescribir una notificación especificada en la directiva de base. Valores posibles: `Append`, `Prepend` o `ReplaceAll`. El valor `Append` especifica que la colección de datos presente tiene que anexarse al final de la colección especificada en la directiva principal. El valor `Prepend` especifica que la colección de datos presente tiene que agregarse antes de la colección especificada en la directiva principal. El valor `ReplaceAll` especifica que la colección de datos definidos en la directiva principal tiene que omitirse y, en su lugar, tienen que usarse los datos definidos en la directiva actual. |

### <a name="supportedlanguages"></a>SupportedLanguages

El elemento **SupportedLanguages** contiene los elementos siguientes:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| SupportedLanguage | 1:n | Muestra contenido que cumple con una etiqueta de idioma según RFC 5646 (etiquetas para identificar idiomas). |

## <a name="localizedresources"></a>LocalizedResources

El elemento **LocalizedResources** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Identificador | Sí | Identificador único para recursos localizados. |

El elemento **LocalizedResources** contiene los elementos siguientes:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| LocalizedCollections | 0:n | Define colecciones enteras de distintas referencias culturales. Una colección puede tener un número distinto de elementos y cadenas diferentes para varias referencias culturales. Entre los ejemplos de colecciones, se incluyen las enumeraciones que aparecen en los tipos de notificación. Por ejemplo, una lista de países o regiones se muestra al usuario en una lista desplegable. |
| LocalizedStrings | 0:n | Define todas las cadenas en distintas referencias culturales, excepto las que aparecen en colecciones. |

### <a name="localizedcollections"></a>LocalizedCollections

El elemento **LocalizedCollections** contiene los elementos siguientes:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| LocalizedCollection | 1:n | Lista de idiomas admitidos. |

#### <a name="localizedcollection"></a>LocalizedCollection

El elemento **LocalizedCollection** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ElementType | Sí | Hace referencia a un elemento ClaimType o a un elemento de la interfaz de usuario en el archivo de directiva. |
| ElementId | Sí | Cadena que contiene una referencia a un tipo de notificación que ya se ha definido en la sección ClaimsSchema y que se usa si se establece **ElementType** en un elemento ClaimType. |
| TargetCollection | Sí | La colección de destino. |

El elemento **LocalizedCollection** contiene los elementos siguientes:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| Elemento | 0:n | Define una opción disponible para que el usuario seleccione una notificación en la interfaz de usuario, como un valor en una lista desplegable. |

El elemento **Item** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| Texto | Sí | Cadena para mostrar fácil de identificar que se mostrará al usuario en la interfaz de usuario para esta opción. |
| Value | Sí | Valor de notificación de cadena asociado a la selección de esta opción. |
| SelectByDefault | No | Indica si esta opción se puede seleccionar o no de forma predeterminada en la interfaz de usuario. Valores posibles: True o False. |

En el ejemplo siguiente, se muestra cómo usar el elemento **LocalizedCollections**. Contiene dos elementos **LocalizedCollection**, uno para inglés y otro para español. Ambos establecen la colección **Restriction** de la notificación `Gender` con una lista de elementos para inglés y español.

```xml
<LocalizedResources Id="api.selfasserted.en">
 <LocalizedCollections>
   <LocalizedCollection ElementType="ClaimType" ElementId="Gender" TargetCollection="Restriction">
      <Item Text="Female" Value="F" />
      <Item Text="Male" Value="M" />
    </LocalizedCollection>
</LocalizedCollections>

<LocalizedResources Id="api.selfasserted.es">
 <LocalizedCollections>
   <LocalizedCollection ElementType="ClaimType" ElementId="Gender" TargetCollection="Restriction">
      <Item Text="Femenino" Value="F" />
      <Item Text="Masculino" Value="M" />
    </LocalizedCollection>
</LocalizedCollections>
```

### <a name="localizedstrings"></a>LocalizedStrings

El elemento **LocalizedStrings** contiene los elementos siguientes:

| Elemento | Repeticiones | Descripción |
| ------- | ----------- | ----------- |
| LocalizedString | 1:n | Cadena localizada. |

El elemento **LocalizedString** contiene los atributos siguientes:

| Atributo | Obligatorio | Descripción |
| --------- | -------- | ----------- |
| ElementType | Sí | Valores posibles: [ClaimsProvider](#claimsprovider), [ClaimType](#claimtype), [ErrorMessage](#errormessage), [GetLocalizedStringsTransformationClaimType](#getlocalizedstringstransformationclaimtype), [Predicate](#predicate), [InputValidation](#inputvalidation) o [UxElement](#uxelement).   | 
| ElementId | Sí | Si **ElementType** se establece en `ClaimType`, `Predicate` o `InputValidation`, este elemento contiene una referencia a un tipo de notificación que ya se ha definido en la sección ClaimsSchema. |
| StringId | Sí | Si **ElementType** se establece en `ClaimType`, este elemento contiene una referencia a un atributo de un tipo de notificación. Valores posibles: `DisplayName`, `AdminHelpText` o `PatternHelpText`. El valor `DisplayName` se usa para establecer el nombre para mostrar de la notificación. El valor `AdminHelpText` se usa para establecer el nombre del texto de ayuda del usuario de la notificación. El valor `PatternHelpText` se usa para establecer el texto de ayuda del patrón de la notificación. Si **ElementType** se establece en `UxElement`, este elemento contiene una referencia a un atributo de un elemento de la interfaz de usuario. Si **ElementType** se establece en `ErrorMessage`, este elemento especifica el identificador de un mensaje de error. Vea la lista completa de los identificadores de `UxElement` en [Id. de cadena Localization](localization-string-ids.md).|

## <a name="elementtype"></a>ElementType

Referencia de ElementType a un tipo de notificación, una transformación de notificaciones o un elemento de la interfaz de usuario de la directiva que se va a localizar.

| Elemento que se va a localizar | ElementType | ElementId |StringId |
| --------- | -------- | ----------- |----------- |
| Nombre del proveedor de identidades |`ClaimsProvider`| | Identificador del elemento ClaimsExchange|
| Atributos de tipo de notificación|`ClaimType`|Nombre del tipo de notificación| Atributo de la notificación que se va a localizar. Valores posibles: `AdminHelpText`, `DisplayName`, `PatternHelpText` y `UserHelpText`.|
|Mensaje de error|`ErrorMessage`||Identificador del mensaje de error |
|Copia las cadenas localizadas en las notificaciones|`GetLocalizedStringsTra nsformationClaimType`||Nombre de la notificación de salida|
|Mensaje de usuario de predicado|`Predicate`|Nombre del predicado| Atributo del predicado que se va a localizar. Valores posibles: `HelpText`.|
|Mensaje de usuario del grupo de predicados|`InputValidation`|Identificador del elemento PredicateValidation.|Identificador del elemento PredicateGroup. El grupo de predicados debe ser un elemento secundario del elemento de validación de predicado, tal como se define en ElementId.|
|Elementos de la interfaz de usuario |`UxElement` | | Identificador del elemento de la interfaz de usuario que se va a localizar.|

## <a name="examples"></a>Ejemplos

### <a name="claimsprovider"></a>ClaimsProvider

El valor de ClaimsProvider se usa para localizar el nombre para mostrar de uno de los proveedores de notificaciones. 

```xml
<OrchestrationStep Order="2" Type="ClaimsExchange">
  ...
  <ClaimsExchanges>
    <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
    <ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAUTH" />
    <ClaimsExchange Id="LinkedInExchange" TechnicalProfileReferenceId="LinkedIn-OAUTH" />
  </ClaimsExchanges>
</OrchestrationStep>

```

En el ejemplo siguiente se muestra cómo localizar el nombre para mostrar de los proveedores de notificaciones.

```xml
<LocalizedString ElementType="ClaimsProvider" StringId="FacebookExchange">Facebook</LocalizedString>
<LocalizedString ElementType="ClaimsProvider" StringId="GoogleExchange">Google</LocalizedString>
<LocalizedString ElementType="ClaimsProvider" StringId="LinkedInExchange">LinkedIn</LocalizedString>
```

### <a name="claimtype"></a>ClaimType

El valor de ClaimType se usa para localizar uno de los atributos de la notificación. 

```xml
<ClaimType Id="email">
  <DisplayName>Email Address</DisplayName>
  <DataType>string</DataType>
  <UserHelpText>Email address that can be used to contact you.</UserHelpText>
  <UserInputType>TextBox</UserInputType>
</ClaimType>
```

En el ejemplo siguiente se muestra cómo localizar los atributos DisplayName, UserHelpText y PatternHelpText del tipo de notificación de correo electrónico.

```xml
<LocalizedString ElementType="ClaimType" ElementId="email" StringId="DisplayName">Email</LocalizedString>
<LocalizedString ElementType="ClaimType" ElementId="email" StringId="UserHelpText">Please enter your email</LocalizedString>
<LocalizedString ElementType="ClaimType" ElementId="email" StringId="PatternHelpText">Please enter a valid email address</LocalizedString>
```

### <a name="errormessage"></a>ErrorMessage

El valor de ErrorMessage se usa para localizar uno de los mensajes de error del sistema. 

```xml
<TechnicalProfile Id="AAD-UserWriteUsingAlternativeSecurityId">
  <Metadata>
    <Item Key="RaiseErrorIfClaimsPrincipalAlreadyExists">true</Item>
    <Item Key="UserMessageIfClaimsPrincipalAlreadyExists">You are already registered, please press the back button and sign in instead.</Item>
  </Metadata>
  ...
</TechnicalProfile>
```

En el ejemplo siguiente se muestra cómo localizar el mensaje de error UserMessageIfClaimsPrincipalAlreadyExists.


```xml
<LocalizedString ElementType="ErrorMessage" StringId="UserMessageIfClaimsPrincipalAlreadyExists">The account you are trying to create already exists, please sign-in.</LocalizedString>
```

### <a name="getlocalizedstringstransformationclaimtype"></a>GetLocalizedStringsTransformationClaimType

El valor de GetLocalizedStringsTransformationClaimType se usa para copiar las cadenas localizadas en las notificaciones. Para más información, consulte la [transformación de notificaciones GetLocalizedStringsTransformation](string-transformations.md#getlocalizedstringstransformation).


```xml
<ClaimsTransformation Id="GetLocalizedStringsForEmail" TransformationMethod="GetLocalizedStringsTransformation">
  <OutputClaims>
    <OutputClaim ClaimTypeReferenceId="subject" TransformationClaimType="email_subject" />
    <OutputClaim ClaimTypeReferenceId="message" TransformationClaimType="email_message" />
    <OutputClaim ClaimTypeReferenceId="codeIntro" TransformationClaimType="email_code" />
    <OutputClaim ClaimTypeReferenceId="signature" TransformationClaimType="email_signature" />
   </OutputClaims>
</ClaimsTransformation>
```

En el ejemplo siguiente se muestra cómo localizar las notificaciones de salida de la transformación de notificaciones GetLocalizedStringsTransformation.

```xml
<LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_subject">Contoso account email verification code</LocalizedString>
<LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_message">Thanks for verifying your account!</LocalizedString>
<LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_code">Your code is</LocalizedString>
<LocalizedString ElementType="GetLocalizedStringsTransformationClaimType" StringId="email_signature">Sincerely</LocalizedString>
```

### <a name="predicate"></a>Predicate

El valor de Predicate se usa para localizar uno de los mensajes de error del elemento [Predicate](predicates.md). 

```xml
<Predicates>
  <Predicate Id="LengthRange" Method="IsLengthRange"  HelpText="The password must be between 6 and 64 characters.">
    <Parameters>
      <Parameter Id="Minimum">6</Parameter>
      <Parameter Id="Maximum">64</Parameter>
    </Parameters>
  </Predicate>
  <Predicate Id="Lowercase" Method="IncludesCharacters" HelpText="a lowercase letter">
    <Parameters>
      <Parameter Id="CharacterSet">a-z</Parameter>
    </Parameters>
  </Predicate>
  <Predicate Id="Uppercase" Method="IncludesCharacters" HelpText="an uppercase letter">
    <Parameters>
      <Parameter Id="CharacterSet">A-Z</Parameter>
    </Parameters>
  </Predicate>
</Predicates>
```

En el ejemplo siguiente se muestra cómo localizar el texto de ayuda de los predicados.

```xml
<LocalizedString ElementType="Predicate" ElementId="LengthRange" StringId="HelpText">The password must be between 6 and 64 characters.</LocalizedString>
<LocalizedString ElementType="Predicate" ElementId="Lowercase" StringId="HelpText">a lowercase letter</LocalizedString>
<LocalizedString ElementType="Predicate" ElementId="Uppercase" StringId="HelpText">an uppercase letter</LocalizedString>
```

### <a name="inputvalidation"></a>InputValidation

El valor de InputValidation se usa para localizar uno de los mensajes de error del grupo [PredicateValidation](predicates.md). 

```xml
<PredicateValidations>
  <PredicateValidation Id="CustomPassword">
    <PredicateGroups>
      <PredicateGroup Id="LengthGroup">
        <PredicateReferences MatchAtLeast="1">
          <PredicateReference Id="LengthRange" />
        </PredicateReferences>
      </PredicateGroup>
      <PredicateGroup Id="CharacterClasses">
        <UserHelpText>The password must have at least 3 of the following:</UserHelpText>
        <PredicateReferences MatchAtLeast="3">
          <PredicateReference Id="Lowercase" />
          <PredicateReference Id="Uppercase" />
          <PredicateReference Id="Number" />
          <PredicateReference Id="Symbol" />
        </PredicateReferences>
      </PredicateGroup>
    </PredicateGroups>
  </PredicateValidation>
</PredicateValidations>
```

En el ejemplo siguiente se muestra cómo localizar el texto de ayuda de un grupo de validación de predicados.

```xml
<LocalizedString ElementType="InputValidation" ElementId="CustomPassword" StringId="CharacterClasses">The password must have at least 3 of the following:</LocalizedString>
```

### <a name="uxelement"></a>UxElement

El valor de UxElement se usa para localizar uno de los elementos de la interfaz de usuario. En el ejemplo siguiente se muestra cómo localizar los botones Continuar y Cancelar.

```xml
<LocalizedString ElementType="UxElement" StringId="button_continue">Create new account</LocalizedString>
<LocalizedString ElementType="UxElement" StringId="button_cancel">Cancel</LocalizedString>
```

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para obtener ejemplos de localización:

- [Personalización de idioma con directiva personalizada en Azure Active Directory B2C](custom-policy-localization.md)
- [Personalización de idioma con flujos de usuario en Azure Active Directory B2C](user-flow-language-customization.md)
