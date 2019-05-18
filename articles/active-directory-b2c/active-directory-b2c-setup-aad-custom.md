---
title: Configurar el inicio de sesión con una cuenta de Azure Active Directory en Azure Active Directory B2C mediante directivas personalizadas | Microsoft Docs
description: Configure el inicio de sesión con una cuenta de Azure Active Directory en Azure Active Directory B2C mediante directivas personalizadas.
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 09/20/2018
ms.author: davidmu
ms.subservice: B2C
ms.openlocfilehash: 9ad4a1a9fff8e011638eb7c532bf7619c87c1996
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65785202"
---
# <a name="set-up-sign-in-with-an-azure-active-directory-account-using-custom-policies-in-azure-active-directory-b2c"></a>Configurar el inicio de sesión con una cuenta de Azure Active Directory mediante directivas personalizadas en Azure Active Directory B2C 

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

En este artículo se muestra cómo habilitar el inicio de sesión para los usuarios desde una organización de Azure Active Directory (Azure AD) mediante [directivas personalizadas](active-directory-b2c-overview-custom.md) en Azure Active Directory (Azure AD) B2C.

## <a name="prerequisites"></a>Requisitos previos

Siga los pasos de [Introducción a las directivas personalizadas en Azure Active Directory B2C](active-directory-b2c-get-started-custom.md).

## <a name="register-an-application"></a>Registrar una aplicación

Para habilitar el inicio de sesión para los usuarios de una organización específica de Azure AD, es preciso registrar una aplicación en el inquilino de Azure AD de la organización.

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
2. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD de la organización (contoso.com). Para ello, haga clic en el **filtro de directorio y suscripción** que se encuentra en el menú superior y elija el directorio que contiene el inquilino.
3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Registros de aplicaciones**.
4. Seleccione **Nuevo registro de aplicaciones**.
5. Escriba un nombre para la aplicación. Por ejemplo, `Azure AD B2C App`.
6. En **Tipo de aplicación**, seleccione `Web app / API`.
7. Para la **dirección URL de inicio de sesión**, escriba la siguiente dirección URL en minúscula, donde `your-B2C-tenant-name` se reemplaza por el nombre del inquilino de Azure AD B2C:

    ```
    https://your-B2C-tenant-name.b2clogin.com/your-B2C-tenant-name.onmicrosoft.com/oauth2/authresp
    ```

    Por ejemplo, `https://contoso.b2clogin.com/contoso.onmicrosoft.com/oauth2/authresp`.

8. Haga clic en **Create**(Crear). Copie el **Identificador de aplicación**, ya que lo usará más adelante.
9. Seleccione la aplicación y **Configuración**.
10. Seleccione **Claves**, escriba la descripción de la clave, seleccione una duración y haga clic en **Guardar**. Copie el valor de la clave que se muestra, ya que lo usará más adelante.

## <a name="create-a-policy-key"></a>Crear una clave de directiva

Debe almacenar la clave de la aplicación que creó en el inquilino de Azure AD B2C.

1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Para ello, haga clic en el **filtro de directorio y suscripción** en el menú superior y elija el directorio que contiene el inquilino.
2. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
3. En la página de introducción, seleccione **Identity Experience Framework**.
4. Seleccione **Claves de directiva** y luego **Agregar**.
5. En **Opciones**, elija `Manual`.
6. Escriba un **nombre** para la clave de directiva. Por ejemplo, `ContosoAppSecret`.  Se agregará el prefijo `B2C_1A_` automáticamente al nombre de la clave.
7. En **Secreto**, escriba la clave de la aplicación que guardó previamente.
8. En **Uso de claves**, seleccione `Signature`.
9. Haga clic en **Create**(Crear).

## <a name="add-a-claims-provider"></a>Incorporación de un proveedor de notificaciones

Si quiere que los usuarios inicien sesión con Azure AD, deberá definir Azure AD como proveedor de notificaciones con el que Azure AD B2C se comunique mediante un punto de conexión. El punto de conexión proporciona un conjunto de notificaciones que Azure AD B2C usa para comprobar que un usuario concreto se ha autenticado. 

Para definir Azure AD como proveedor de notificaciones, agregue el elemento **ClaimsProvider** al archivo de extensión de la directiva.

1. Abra el archivo *TrustFrameworkExtensions.xml*.
2. Busque el elemento **ClaimsProviders**. Si no existe, agréguelo debajo del elemento raíz.
3. Agregue un nuevo elemento **ClaimsProvider** tal como se muestra a continuación:

    ```XML
    <ClaimsProvider>
      <Domain>Contoso</Domain>
      <DisplayName>Login using Contoso</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="ContosoProfile">
          <DisplayName>Contoso Employee</DisplayName>
          <Description>Login with your Contoso account</Description>
          <Protocol Name="OpenIdConnect"/>
          <OutputTokenFormat>JWT</OutputTokenFormat>
          <Metadata>
            <Item Key="METADATA">https://login.windows.net/your-AD-tenant-name.onmicrosoft.com/.well-known/openid-configuration</Item>
            <Item Key="ProviderName">https://sts.windows.net/00000000-0000-0000-0000-000000000000/</Item>
            <Item Key="client_id">00000000-0000-0000-0000-000000000000</Item>
            <Item Key="IdTokenAudience">00000000-0000-0000-0000-000000000000</Item>
            <Item Key="UsePolicyInRedirectUri">false</Item>
            <Item Key="response_types">code</Item>
            <Item Key="scope">openid</Item>
            <Item Key="response_mode">form_post</Item>
            <Item Key="HttpBinding">POST</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_ContosoAppSecret"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="oid"/>
            <OutputClaim ClaimTypeReferenceId="tenantId" PartnerClaimType="tid"/>
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
            <OutputClaim ClaimTypeReferenceId="surName" PartnerClaimType="family_name" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" AlwaysUseDefaultValue="true" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="AzureADContoso" AlwaysUseDefaultValue="true" />
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-SocialLogin"/>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

4. En el elemento **ClaimsProvider**, actualice el valor de **Domain** a un valor único que pueda usarse para distinguirlo de otros proveedores de identidades. Por ejemplo: `Contoso`. No incluya un `.com` al final de esta configuración de dominio.
5. En el elemento **ClaimsProvider**, actualice el valor de **DisplayName** a un nombre descriptivo del proveedor de notificaciones. Este valor no se utiliza actualmente.

### <a name="update-the-technical-profile"></a>Actualización del perfil técnico

Para obtener un token del punto de conexión de Azure AD es preciso definir los protocolos que Azure AD B2C debe usar para comunicarse con Azure AD. Esto se realiza dentro del elemento **TechnicalProfile** de **ClaimsProvider**.

1. Actualice el identificador del elemento **TechnicalProfile**. Este identificador se usa para hacer referencia a este perfil técnico desde otras partes de la directiva.
2. Actualice el valor de **DisplayName**. Este valor se mostrará en el botón de inicio de sesión de la pantalla de inicio de sesión.
3. Actualice el valor de **Description**.
4. Azure AD usa el protocolo OpenID Connect, por lo que debe asegurarse de que el valor de **Protocol** es `OpenIdConnect`.
5. Establezca el valor de **METADATA** como `https://login.windows.net/your-AD-tenant-name.onmicrosoft.com/.well-known/openid-configuration`, donde `your-AD-tenant-name` es el nombre del inquilino de Azure AD. Por ejemplo, `https://login.windows.net/fabrikam.onmicrosoft.com/.well-known/openid-configuration`
6. Abra el explorador y vaya a la dirección URL **METADATA** que acaba de actualizar, busque el objeto del **emisor** y copie y pegue el valor en **ProviderName** en el archivo XML.
8. Establezca **client_id** y **IdTokenAudience** en el identificador de la aplicación desde el registro de aplicación.
9. En **CryptograhicKeys**, actualice el valor de **StorageReferenceId** a la clave de directiva que ha definido. Por ejemplo, `ContosoAppSecret`.

### <a name="upload-the-extension-file-for-verification"></a>Carga del archivo de extensión para su comprobación

Por el momento, ha configurado la directiva para que Azure AD B2C sepa cómo comunicarse con su directorio de Azure AD. Pruebe a cargar el archivo de extensión de la directiva para confirmar que no tiene problemas.

1. En la página **Directivas personalizadas** del inquilino de Azure AD B2C, seleccione **Cargar directiva**.
2. Habilite **Sobrescribir la directiva, si existe**, y busque y seleccione el archivo *TrustFrameworkExtensions.xml*.
3. Haga clic en **Cargar**.

## <a name="register-the-claims-provider"></a>Registro del proveedor de notificaciones

El proveedor de identidades ya se ha configurado, pero no está disponible en ninguna de las pantallas de registro/inicio de sesión. Para que esté disponible, debe crear un duplicado de un recorrido del usuario de plantilla existente que modificaremos a continuación para que también tenga el proveedor de identidades de Azure AD:

1. Abra el archivo *TrustFrameworkBase.xml* del paquete de inicio.
2. Busque y copie todo el contenido del elemento **UserJourney** que incluye `Id="SignUpOrSignIn"`.
3. Abra el archivo *TrustFrameworkExtensions.xml* y busque el elemento **UserJourneys**. Si el elemento no existe, agréguelo.
4. Pegue todo el contenido del elemento **UserJourney** que ha copiado como elemento secundario del elemento **UserJourneys**.
5. Cambie el identificador del recorrido del usuario. Por ejemplo, `SignUpSignInContoso`.

### <a name="display-the-button"></a>Visualización del botón

El elemento **ClaimsProviderSelection** es análogo a un botón del proveedor de identidades en una pantalla de registro o de inicio de sesión. Si agrega un elemento **ClaimsProviderSelection** a Azure AD, se muestra un nuevo botón cuando un usuario llega a la página.

1. Busque el elemento **OrchestrationStep** que incluye `Order="1"` en el recorrido del usuario que ha creado.
2. En **ClaimsProviderSelections**, agregue el siguiente elemento. Establezca un valor adecuado en **TargetClaimsExchangeId**, por ejemplo, `ContosoExchange`:

    ```XML
    <ClaimsProviderSelection TargetClaimsExchangeId="ContosoExchange" />
    ```

### <a name="link-the-button-to-an-action"></a>Vincular el botón a una acción

Ahora que hay un botón colocado, es preciso vincularlo a una acción. En este caso, la acción es para que Azure AD B2C se comunique con Azure AD para recibir un token. Para vincular un botón a una acción, vincule el perfil técnico del proveedor de notificaciones de Azure AD:

1. Busque el elemento **OrchestrationStep** que incluye `Order="2"` en el recorrido del usuario.
2. Al agregar el siguiente elemento **ClaimsExchange**, asegúrese de usar el mismo valor para el elemento **Id** que el que usó en **TargetClaimsExchangeId**:

    ```XML
    <ClaimsExchange Id="ContosoExchange" TechnicalProfileReferenceId="ContosoProfile" />
    ```
    
    Actualice el valor de **TechnicalProfileReferenceId** al elemento **Id** del perfil técnico que creó anteriormente. Por ejemplo, `ContosoProfile`.

3. Guarde el archivo *TrustFrameworkExtensions.xml* y cárguelo de nuevo a fin de verificarlo.

## <a name="create-an-azure-ad-b2c-application"></a>Creación de una aplicación de Azure AD B2C

La comunicación con Azure AD B2C se produce mediante una aplicación que se crea en el inquilino. En esta sección se enumeran los pasos opcionales que puede llevar a cabo para crear una aplicación de prueba, si aún no lo ha hecho.

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
2. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Para ello, haga clic en el **filtro de directorio y suscripción** en el menú superior y elija el directorio que contiene el inquilino.
3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
4. Seleccione **Aplicaciones** y **Agregar**.
5. Escriba un nombre para la aplicación; por ejemplo, *testapp1*.
6. En **Aplicación web/API web**, seleccione `Yes` y escriba `https://jwt.ms` como **URL de respuesta**.
7. Haga clic en **Create**(Crear).

## <a name="update-and-test-the-relying-party-file"></a>Actualización y prueba del archivo del usuario de confianza

Actualice el archivo de usuario de confianza (RP) que inicia el recorrido del usuario que ha creado.

1. Realice una copia del archivo *SignUpOrSignIn.xml* en el directorio de trabajo y cámbiele el nombre. Por ejemplo, cambie su nombre a *SignUpSignInContoso.xml*.
2. Abra el nuevo archivo y actualice el valor del atributo **PolicyId** del elemento **TrustFrameworkPolicy** con un valor único. Por ejemplo, `SignUpSignInContoso`.
3. Actualice el valor de **PublicPolicyUri** con el URI para la directiva. Por ejemplo: `http://contoso.com/B2C_1A_signup_signin_contoso`.
4. Actualice el valor del atributo **ReferenceId** del elemento **DefaultUserJourney** para que coincida con el identificador del nuevo recorrido del usuario que ha creado (SignUpSignInContoso).
5. Guarde los cambios, cargue el archivo y, a continuación, seleccione la nueva directiva en la lista.
6. Asegúrese de que la aplicación de Azure AD B2C que creó está seleccionada en el campo **Seleccionar aplicación** y pruébela; para ello, haga clic en **Ejecutar ahora**.

