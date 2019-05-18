---
title: UserJourneys | Microsoft Docs
description: Especifique el elemento UserJourneys de una directiva personalizada en Azure Active Directory B2C.
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: reference
ms.date: 09/10/2018
ms.author: davidmu
ms.subservice: B2C
ms.openlocfilehash: f5e56d4953eecdb488d5dadd4497b1c42b932f35
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65812577"
---
# <a name="userjourneys"></a>UserJourneys

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

Los recorridos del usuario especifican rutas de acceso explícitas con las que una directiva permite que una aplicación de usuario de confianza obtenga las notificaciones deseadas para un usuario. Se conduce al usuario por estas rutas de acceso para recuperar las notificaciones que se van a presentar al usuario de confianza. En otras palabras, los recorridos del usuario definen la lógica de negocios por la que pasa un usuario final mientras el marco de experiencia de identidad de Azure AD B2C procesa la solicitud.

Estos recorridos de usuario pueden considerarse como plantillas disponibles para satisfacer las necesidades de núcleo de los distintos usuarios de confianza de la Comunidad de interés. Recorridos de usuario facilitan la definición de la parte de terceros para usuario autenticado de una directiva. Una directiva puede definir varios recorridos del usuario. Cada recorrido del usuario es una secuencia de pasos de orquestación.

Para definir los recorridos del usuario compatibles con la directiva, se agrega un elemento **UserJourneys** debajo del elemento de nivel superior del archivo de directiva. 

El elemento **UserJourneys** contiene el elemento siguiente:

| Elemento | Repeticiones | DESCRIPCIÓN |
| ------- | ----------- | ----------- |
| UserJourney | 1:n | Un recorrido del usuario que define todas las construcciones necesarias para un flujo de usuario completo. | 

El elemento **UserJourney** contiene los siguientes atributos:

| Atributo | Obligatorio | DESCRIPCIÓN |
| --------- | -------- | ----------- |
| Identificador | Sí | Un identificador de un recorrido del usuario que se puede usar para hacer referencia a él desde otros elementos de la directiva. El elemento **DefaultUserJourney** de la [directiva del usuario de confianza](relyingparty.md) apunta a este atributo. |

El elemento **UserJourney** contiene los siguientes elementos:

| Elemento | Repeticiones | DESCRIPCIÓN |
| ------- | ----------- | ----------- |
| OrchestrationSteps | 1:n | Una secuencia de orquestación por la que hay que pasar para lograr una transacción correcta. Cada recorrido del usuario consta de una lista ordenada de los pasos de orquestación que se ejecutan en secuencia. Si se produce un error en algún paso, la transacción no prospera. |

## <a name="orchestrationsteps"></a>OrchestrationSteps

El recorrido del usuario se representa como una secuencia de orquestación por la que hay que pasar para lograr una transacción correcta. Si se produce un error en algún paso, la transacción no prospera. Estos pasos de orquestación hacen referencia tanto a los bloques de construcción como a los proveedores de notificaciones permitidos en el archivo de directiva. Cualquier paso de orquestación que se encargue de mostrar o representar una experiencia de usuario también hace referencia al identificador de la definición de contenido correspondiente.

Pasos de orquestación se pueden ejecutar condicionalmente en función de las condiciones previas definidas en el elemento de paso de orquestación. Por ejemplo, puede comprobar para llevar a cabo un paso de orquestación sólo si existe un notificaciones específicas, una notificación es igual o no en el valor especificado. 

Para especificar la lista ordenada de los pasos de orquestación, se agrega un elemento **OrchestrationSteps** como parte de la directiva. Este elemento es obligatorio.

El elemento **OrchestrationSteps** contiene el elemento siguiente:

| Elemento | Repeticiones | DESCRIPCIÓN |
| ------- | ----------- | ----------- |
| OrchestrationStep | 1:n | Un paso de orquestación ordenada. | 

El elemento **OrchestrationStep** contiene los siguientes atributos:

| Atributo | Obligatorio | DESCRIPCIÓN |
| --------- | -------- | ----------- |
| `Order` | Sí | El orden de los pasos de orquestación. | 
| `Type` | Sí | El tipo de paso de orquestación. Valores posibles: <ul><li>**ClaimsProviderSelection**: indica que el paso de orquestación presenta diversos proveedores de notificaciones al usuario para que seleccione uno.</li><li>**CombinedSignInAndSignUp**: indica que el paso de orquestación presenta una página combinada de inicio de sesión en el proveedor social y de registro en la cuenta local.</li><li>**ClaimsExchange**: indica que el paso de orquestación intercambia notificaciones con un proveedor de notificaciones.</li><li>**SendClaims**: indica que el paso de orquestación envía las notificaciones al usuario de confianza con un token emitido por un emisor de notificaciones.</li></ul> | 
| ContentDefinitionReferenceId | No | El identificador de la [definición de contenido](contentdefinitions.md) asociada a este paso de orquestación. Normalmente, el identificador de referencia de la definición de contenido se define en el perfil técnico autoafirmado. Pero hay algunos casos en los que Azure AD B2C necesita mostrar contenido sin un perfil técnico. Hay dos ejemplos: si el tipo de paso de orquestación es uno de los siguientes: `ClaimsProviderSelection` o `CombinedSignInAndSignUp`, Azure AD B2C necesita para mostrar la selección del proveedor de identidad sin tener un perfil técnico. | 
| CpimIssuerTechnicalProfileReferenceId | No | El tipo de paso de orquestación es `SendClaims`. Esta propiedad define el identificador de perfil técnico del proveedor de notificaciones que emite el token del usuario de confianza.  Si no aparece, no se crea ningún token para el usuario de confianza. |


El elemento **OrchestrationStep** puede contener los siguientes elementos:

| Elemento | Repeticiones | DESCRIPCIÓN |
| ------- | ----------- | ----------- | 
| Preconditions | 0:n | Una lista de las condiciones previas que deben cumplirse para que se ejecute el paso de orquestación. | 
| ClaimsProviderSelections | 0:n | Una lista de las selecciones del proveedor de notificaciones para el paso de orquestación. | 
| ClaimsExchanges | 0:n | Una lista de los intercambios de notificaciones para el paso de orquestación. | 

### <a name="preconditions"></a>Condiciones previas

El elemento **Preconditions** contiene el elemento siguiente:

| Elemento | Repeticiones | DESCRIPCIÓN |
| ------- | ----------- | ----------- | 
| Condición previa | 0:n | En función del perfil técnico que se esté utilizando, puede ser que se redireccione al cliente según la selección del proveedor de notificaciones o que se haga una llamada al servidor para intercambiar notificaciones. | 


#### <a name="precondition"></a>Condición previa

El **Precondition** elemento contiene los siguientes atributos:

| Atributo | Obligatorio | DESCRIPCIÓN |
| --------- | -------- | ----------- |
| `Type` | Sí | El tipo de comprobación o la consulta que hay que llevar a cabo para esta condición previa. El valor puede ser **ClaimsExist**, que especifica que se deben realizar las acciones si existen las notificaciones especificadas en el conjunto de notificaciones actual del usuario, o **ClaimEquals**, que especifica que las acciones deben realizarse si existe la notificación especificada y su valor es igual al valor especificado. |
| `ExecuteActionsIf` | Sí | Use una prueba de true o false para decidir si se deben realizar las acciones de la condición previa. | 

El elemento **Precondition** contiene los siguientes elementos:

| Elemento | Repeticiones | DESCRIPCIÓN |
| ------- | ----------- | ----------- |
| Valor | 1:n | Un valor ClaimTypeReferenceId que se va a consultar. Otro elemento de valor contiene el valor que se va a comprobar.</li></ul>|
| . | 1:1 | La acción que debe realizarse si se cumple la comprobación de condición previa dentro de un paso de orquestación. Si el valor de la `Action` está establecido en `SkipThisOrchestrationStep`, el `OrchestrationStep` asociado no debe ejecutarse. | 

#### <a name="preconditions-examples"></a>Ejemplos de condiciones previas

Con las siguientes condiciones previas se comprueba si existe el objectId del usuario. En el recorrido del usuario, el usuario ha optado por iniciar sesión con una cuenta local. Si existe el valor objectId, omita este paso de orquestación.

```XML
<OrchestrationStep Order="2" Type="ClaimsExchange">
  <Preconditions>
    <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
      <Value>objectId</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
    <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccountSignUpWithLogonEmail" />
  </ClaimsExchanges>
</OrchestrationStep>
```

Con las siguientes condiciones previas se comprueba si el usuario ha iniciado sesión con una cuenta social. Se intenta encontrar la cuenta de usuario en el directorio. Si el usuario inicia sesión o se registra con una cuenta local, omita este paso de orquestación.

```XML
<OrchestrationStep Order="3" Type="ClaimsExchange">
  <Preconditions>
    <Precondition Type="ClaimEquals" ExecuteActionsIf="true">
      <Value>authenticationSource</Value>
      <Value>localAccountAuthentication</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="AADUserReadUsingAlternativeSecurityId" TechnicalProfileReferenceId="AAD-UserReadUsingAlternativeSecurityId-NoError" />
  </ClaimsExchanges>
</OrchestrationStep>
```

Las condiciones previas pueden comprobar varias condiciones previas. En el ejemplo siguiente se comprueba si existe “objectId” o “email”. Si la primera condición es true, se omite el viaje con el siguiente paso de orquestación.

```XML
<OrchestrationStep Order="4" Type="ClaimsExchange">
  <Preconditions>
  <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
      <Value>objectId</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
    <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
      <Value>email</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>      
  <ClaimsExchanges>
    <ClaimsExchange Id="SelfAsserted-SocialEmail" TechnicalProfileReferenceId="SelfAsserted-SocialEmail" />
  </ClaimsExchanges>
</OrchestrationStep>
```

## <a name="claimsproviderselection"></a>ClaimsProviderSelection

Un paso de orquestación de tipo `ClaimsProviderSelection` o `CombinedSignInAndSignUp` puede contener una lista de proveedores de notificaciones con los que un usuario puede iniciar sesión. El orden de los elementos dentro de los elementos `ClaimsProviderSelections` controla el orden de los proveedores de identidades que se presentan al usuario.

El elemento **ClaimsProviderSelection** contiene el elemento siguiente:

| Elemento | Repeticiones | DESCRIPCIÓN |
| ------- | ----------- | ----------- |
| ClaimsProviderSelection | 0:n | Proporciona la lista de proveedores de notificaciones que se pueden seleccionar.|

El elemento **ClaimsProviderSelection** contiene los atributos siguientes: 

| Atributo | Obligatorio | DESCRIPCIÓN |
| --------- | -------- | ----------- |
| TargetClaimsExchangeId | No | El identificador del intercambio de notificaciones, que se ejecuta en el siguiente paso de orquestación de la selección del proveedor de notificaciones. Hay que especificar este atributo o el atributo ValidationClaimsExchangeId, pero no ambos. | 
| ValidationClaimsExchangeId | No | El identificador del intercambio de notificaciones, que se ejecuta en el paso de orquestación actual para validar la selección del proveedor de notificaciones. Hay que especificar este atributo o el atributo TargetClaimsExchangeId, pero no ambos. |

### <a name="claimsproviderselection-example"></a>Ejemplo de ClaimsProviderSelection

En el siguiente paso de orquestación, el usuario puede iniciar sesión con Facebook, LinkedIn, Twitter, Google o una cuenta local. Si el usuario selecciona uno de los proveedores de identidades sociales, el segundo paso de orquestación se ejecuta con el intercambio de notificaciones seleccionado especificado en el atributo `TargetClaimsExchangeId`. El segundo paso de orquestación redirige al usuario al proveedor de identidades sociales para completar el proceso de inicio de sesión. Si el usuario decide iniciar sesión con la cuenta local, Azure AD B2C se mantiene en el mismo paso de orquestación (la misma página de registro o de inicio de sesión) y omite el segundo paso de orquestación.

```XML
<OrchestrationStep Order="1" Type="CombinedSignInAndSignUp" ContentDefinitionReferenceId="api.signuporsignin">
    <ClaimsProviderSelections>
    <ClaimsProviderSelection TargetClaimsExchangeId="FacebookExchange" />
    <ClaimsProviderSelection TargetClaimsExchangeId="LinkedInExchange" />
    <ClaimsProviderSelection TargetClaimsExchangeId="TwitterExchange" />
    <ClaimsProviderSelection TargetClaimsExchangeId="GoogleExchange" />
    <ClaimsProviderSelection ValidationClaimsExchangeId="LocalAccountSigninEmailExchange" />
    </ClaimsProviderSelections>
    <ClaimsExchanges>
    <ClaimsExchange Id="LocalAccountSigninEmailExchange" 
                    TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Email" />
    </ClaimsExchanges>
</OrchestrationStep>


<OrchestrationStep Order="2" Type="ClaimsExchange">
  <Preconditions>
    <Precondition Type="ClaimsExist" ExecuteActionsIf="true">
      <Value>objectId</Value>
      <Action>SkipThisOrchestrationStep</Action>
    </Precondition>
  </Preconditions>
  <ClaimsExchanges>
    <ClaimsExchange Id="FacebookExchange" TechnicalProfileReferenceId="Facebook-OAUTH" />
    <ClaimsExchange Id="SignUpWithLogonEmailExchange" TechnicalProfileReferenceId="LocalAccountSignUpWithLogonEmail" />
    <ClaimsExchange Id="GoogleExchange" TechnicalProfileReferenceId="Google-OAUTH" />
    <ClaimsExchange Id="LinkedInExchange" TechnicalProfileReferenceId="LinkedIn-OAUTH" />
    <ClaimsExchange Id="TwitterExchange" TechnicalProfileReferenceId="Twitter-OAUTH1" />
  </ClaimsExchanges>
</OrchestrationStep>
```

## <a name="claimsexchanges"></a>ClaimsExchanges

El elemento **ClaimsExchanges** contiene el elemento siguiente:

| Elemento | Repeticiones | DESCRIPCIÓN |
| ------- | ----------- | ----------- |
| ClaimsExchange | 0:n | En función del perfil técnico que se esté usando, puede ser que se redireccione al cliente según la selección del proveedor de notificaciones que se ha seleccionado o que se haga una llamada al servidor para intercambiar notificaciones. | 

El elemento **ClaimsExchange** contiene los atributos siguientes:

| Atributo | Obligatorio | DESCRIPCIÓN |
| --------- | -------- | ----------- |
| Identificador | Sí | Identificador del paso de intercambio de notificaciones. El identificador se usa para hacer referencia al intercambio de notificaciones a partir de un paso de selección del proveedor de notificaciones en la directiva. | 
| TechnicalProfileReferenceId | Sí | El identificador del perfil técnico que se va a ejecutar. |
