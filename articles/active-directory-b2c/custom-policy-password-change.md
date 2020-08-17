---
title: Configuración del cambio de contraseñas mediante directivas personalizadas
titleSuffix: Azure AD B2C
description: Obtenga información sobre cómo habilitar a los usuarios para el cambio de contraseñas mediante directivas personalizadas en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 12/13/2018
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 6faf9fb3d15709d3897db9a77bf14ebf238e8fbf
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87116370"
---
# <a name="configure-password-change-using-custom-policies-in-azure-active-directory-b2c"></a>Configuración del cambio de contraseñas con directivas personalizadas en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

En Azure Active Directory B2C (Azure AD B2C), puede permitir que los usuarios que han iniciado sesión con una cuenta local cambien su contraseña sin tener que demostrar su autenticidad por comprobación de correo electrónico. Si la sesión expira cuando el usuario llega al flujo de cambio de contraseña, se le solicita que inicie sesión de nuevo. En este artículo se muestra cómo configurar el cambio de contraseñas en [directivas personalizadas](custom-policy-overview.md). También es posible configurar el [restablecimiento de contraseña de autoservicio](user-flow-self-service-password-reset.md) para flujos de usuario.

## <a name="prerequisites"></a>Prerequisites

Siga los pasos de [Introducción a las directivas personalizadas en Active Directory B2C](custom-policy-get-started.md).

## <a name="add-the-elements"></a>Adición de los elementos

1. Abra el archivo *TrustframeworkExtensions.xml* y agregue el siguiente elemento **ClaimType** con un identificador de `oldPassword` al elemento [ClaimsSchema](claimsschema.md):

    ```xml
    <BuildingBlocks>
      <ClaimsSchema>
        <ClaimType Id="oldPassword">
          <DisplayName>Old Password</DisplayName>
          <DataType>string</DataType>
          <UserHelpText>Enter password</UserHelpText>
          <UserInputType>Password</UserInputType>
        </ClaimType>
      </ClaimsSchema>
    </BuildingBlocks>
    ```

2. Un elemento [ClaimsProvider](claimsproviders.md) contiene el perfil técnico que autentica al usuario. Agregue los siguientes proveedores de notificaciones al elemento **ClaimsProviders**:

    ```xml
    <ClaimsProviders>
      <ClaimsProvider>
        <DisplayName>Local Account SignIn</DisplayName>
        <TechnicalProfiles>
          <TechnicalProfile Id="login-NonInteractive-PasswordChange">
            <DisplayName>Local Account SignIn</DisplayName>
            <Protocol Name="OpenIdConnect" />
            <Metadata>
              <Item Key="UserMessageIfClaimsPrincipalDoesNotExist">We can't seem to find your account</Item>
              <Item Key="UserMessageIfInvalidPassword">Your password is incorrect</Item>
              <Item Key="UserMessageIfOldPasswordUsed">Looks like you used an old password</Item>
              <Item Key="ProviderName">https://sts.windows.net/</Item>
              <Item Key="METADATA">https://login.microsoftonline.com/{tenant}/.well-known/openid-configuration</Item>
              <Item Key="authorization_endpoint">https://login.microsoftonline.com/{tenant}/oauth2/token</Item>
              <Item Key="response_types">id_token</Item>
              <Item Key="response_mode">query</Item>
              <Item Key="scope">email openid</Item>
              <Item Key="grant_type">password</Item>
              <Item Key="UsePolicyInRedirectUri">false</Item>
              <Item Key="HttpBinding">POST</Item>
              <Item Key="client_id">ProxyIdentityExperienceFrameworkAppId</Item>
              <Item Key="IdTokenAudience">IdentityExperienceFrameworkAppId</Item>
            </Metadata>
            <InputClaims>
              <InputClaim ClaimTypeReferenceId="signInName" PartnerClaimType="username" Required="true" />
              <InputClaim ClaimTypeReferenceId="oldPassword" PartnerClaimType="password" Required="true" />
              <InputClaim ClaimTypeReferenceId="grant_type" DefaultValue="password" />
              <InputClaim ClaimTypeReferenceId="scope" DefaultValue="openid" />
              <InputClaim ClaimTypeReferenceId="nca" PartnerClaimType="nca" DefaultValue="1" />
              <InputClaim ClaimTypeReferenceId="client_id" DefaultValue="ProxyIdentityExperienceFrameworkAppID" />
              <InputClaim ClaimTypeReferenceId="resource_id" PartnerClaimType="resource" DefaultValue="IdentityExperienceFrameworkAppID" />
            </InputClaims>
            <OutputClaims>
              <OutputClaim ClaimTypeReferenceId="objectId" PartnerClaimType="oid" />
              <OutputClaim ClaimTypeReferenceId="tenantId" PartnerClaimType="tid" />
              <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
              <OutputClaim ClaimTypeReferenceId="surName" PartnerClaimType="family_name" />
              <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
              <OutputClaim ClaimTypeReferenceId="userPrincipalName" PartnerClaimType="upn" />
              <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="localAccountAuthentication" />
            </OutputClaims>
          </TechnicalProfile>
        </TechnicalProfiles>
      </ClaimsProvider>
      <ClaimsProvider>
        <DisplayName>Local Account Password Change</DisplayName>
        <TechnicalProfiles>
          <TechnicalProfile Id="LocalAccountWritePasswordChangeUsingObjectId">
            <DisplayName>Change password (username)</DisplayName>
            <Protocol Name="Proprietary" Handler="Web.TPEngine.Providers.SelfAssertedAttributeProvider, Web.TPEngine, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null" />
            <Metadata>
              <Item Key="ContentDefinitionReferenceId">api.selfasserted</Item>
            </Metadata>
            <CryptographicKeys>
              <Key Id="issuer_secret" StorageReferenceId="B2C_1A_TokenSigningKeyContainer" />
            </CryptographicKeys>
            <InputClaims>
              <InputClaim ClaimTypeReferenceId="objectId" />
            </InputClaims>
            <OutputClaims>
              <OutputClaim ClaimTypeReferenceId="oldPassword" Required="true" />
              <OutputClaim ClaimTypeReferenceId="newPassword" Required="true" />
              <OutputClaim ClaimTypeReferenceId="reenterPassword" Required="true" />
            </OutputClaims>
            <ValidationTechnicalProfiles>
              <ValidationTechnicalProfile ReferenceId="login-NonInteractive-PasswordChange" />
              <ValidationTechnicalProfile ReferenceId="AAD-UserWritePasswordUsingObjectId" />
            </ValidationTechnicalProfiles>
          </TechnicalProfile>
        </TechnicalProfiles>
      </ClaimsProvider>
    </ClaimsProviders>
    ```

    Reemplace `IdentityExperienceFrameworkAppId` por el identificador de aplicación de la aplicación IdentityExperienceFramework que creó en el tutorial de requisitos previos. Reemplace `ProxyIdentityExperienceFrameworkAppId` por el identificador de aplicación de la aplicación ProxyIdentityExperienceFramework que creó anteriormente.

3. El elemento [UserJourney](userjourneys.md) define la ruta de acceso que el usuario toma al interactuar con la aplicación. Agregue el elemento **UserJourneys**, si no existe, con el elemento **UserJourney** identificado como `PasswordChange`:

    ```xml
    <UserJourneys>
      <UserJourney Id="PasswordChange">
        <OrchestrationSteps>
          <OrchestrationStep Order="1" Type="ClaimsProviderSelection" ContentDefinitionReferenceId="api.idpselections">
            <ClaimsProviderSelections>
              <ClaimsProviderSelection TargetClaimsExchangeId="LocalAccountSigninEmailExchange" />
            </ClaimsProviderSelections>
          </OrchestrationStep>
          <OrchestrationStep Order="2" Type="ClaimsExchange">
            <ClaimsExchanges>
              <ClaimsExchange Id="LocalAccountSigninEmailExchange" TechnicalProfileReferenceId="SelfAsserted-LocalAccountSignin-Email" />
            </ClaimsExchanges>
          </OrchestrationStep>
          <OrchestrationStep Order="3" Type="ClaimsExchange">
            <ClaimsExchanges>
              <ClaimsExchange Id="NewCredentials" TechnicalProfileReferenceId="LocalAccountWritePasswordChangeUsingObjectId" />
            </ClaimsExchanges>
          </OrchestrationStep>
          <OrchestrationStep Order="4" Type="SendClaims" CpimIssuerTechnicalProfileReferenceId="JwtIssuer" />
        </OrchestrationSteps>
        <ClientDefinition ReferenceId="DefaultWeb" />
      </UserJourney>
    </UserJourneys>
    ```

4. Guarde el archivo de directiva *TrustFrameworkExtensions.xml*.
5. Copie el archivo *ProfileEdit.xml* que descargó con el paquete de inicio y asígnele el nombre de archivo *ProfileEditPasswordChange.xml*.
6. Abra el archivo nuevo y actualice el atributo **PolicyId** con un valor único. Este valor es el nombre de la directiva. Por ejemplo, *B2C_1A_profile_edit_password_change*.
7. Modifique el atributo **ReferenceId** de `<DefaultUserJourney>` para que coincida con el identificador del nuevo recorrido del usuario que ha creado. Por ejemplo, *PasswordChange*.
8. Guarde los cambios.

Puede encontrar la directiva de ejemplo [aquí](https://github.com/Azure-Samples/active-directory-b2c-custom-policy-starterpack/tree/master/scenarios/password-change).

## <a name="test-your-policy"></a>Prueba de la directiva

Al probar las aplicaciones en Azure AD B2C, puede ser útil que el token de Azure AD B2C se devuelva a `https://jwt.ms` para poder revisar las notificaciones en él.

### <a name="upload-the-files"></a>Carga de los archivos

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino.
3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
4. Seleccione **Marco de experiencia de identidad**.
5. En la página Directivas personalizadas, haga clic en **Cargar directiva**.
6. Seleccione **Sobrescribir la directiva, si existe** y busque y seleccione el archivo *TrustframeworkExtensions.xml*.
7. Haga clic en **Cargar**.
8. Repita los pasos 5 a 7 para el archivo del usuario de confianza, como *ProfileEditPasswordChange.xml*.

### <a name="run-the-policy"></a>Ejecución de la directiva

1. Abra la directiva que ha cambiado. Por ejemplo, *B2C_1A_profile_edit_password_change*.
2. En **Aplicación**, seleccione la aplicación que registró anteriormente. Para ver el token, **URL de respuesta** debe mostrar `https://jwt.ms`.
3. Haga clic en **Ejecutar ahora**. Inicie sesión con la cuenta que creó anteriormente. Ahora debería tener la oportunidad de cambiar la contraseña.

## <a name="next-steps"></a>Pasos siguientes

- Más información sobre [Configuración de la complejidad de las contraseñas con directivas personalizadas en Azure Active Directory B2C](custom-policy-password-complexity.md).
