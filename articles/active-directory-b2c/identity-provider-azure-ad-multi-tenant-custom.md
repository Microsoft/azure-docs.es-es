---
title: Configuración del inicio de sesión para Azure AD multiinquilino mediante directivas personalizadas
titleSuffix: Azure AD B2C
description: Adición de un proveedor de identidades de Azure AD multiinquilino mediante directivas personalizadas en Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/10/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 645a0d21fc25cb45914eed02e023a0076c457ffb
ms.sourcegitcommit: 0820c743038459a218c40ecfb6f60d12cbf538b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87116293"
---
# <a name="set-up-sign-in-for-multi-tenant-azure-active-directory-using-custom-policies-in-azure-active-directory-b2c"></a>Configuración del inicio de sesión para Azure Active Directory multiinquilino mediante directivas personalizadas en Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

En este artículo se muestra cómo habilitar el inicio de sesión de los usuarios con el punto de conexión de multiempresa de Azure Active Directory (Azure AD) mediante [directivas personalizadas](custom-policy-overview.md) en Azure AD B2C. Esto permite a los usuarios de varios inquilinos de Azure AD iniciar sesión en Azure AD B2C sin tener que configurar un proveedor técnico para cada inquilino. Sin embargo, los miembros invitados en cualquiera de estos inquilinos **no** podrán iniciar sesión. Para ello, tendrá que [configurar individualmente cada inquilino](identity-provider-azure-ad-single-tenant-custom.md).

## <a name="prerequisites"></a>Prerrequisitos

Siga los pasos de [Introducción a las directivas personalizadas en Azure Active Directory B2C](custom-policy-get-started.md).

## <a name="register-an-application"></a>Registro de una aplicación

Para habilitar el inicio de sesión para los usuarios de una organización específica de Azure AD, es preciso registrar una aplicación en el inquilino de Azure AD de la organización.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD de la organización (por ejemplo, contoso.com). Seleccione el **filtro de directorio y suscripción** en el menú superior y elija el directorio que contiene la suscripción.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Registros de aplicaciones**.
1. Seleccione **Nuevo registro**.
1. Escriba el **nombre** de la aplicación. Por ejemplo, `Azure AD B2C App`.
1. Seleccione **Cuentas en cualquier directorio organizativo** para esta aplicación.
1. Para la **URI de redirección**, acepte el valor de **Web** y escriba la siguiente dirección URL en minúscula, donde `your-B2C-tenant-name` se reemplaza por el nombre del inquilino de Azure AD B2C.

    ```
    https://your-B2C-tenant-name.b2clogin.com/your-B2C-tenant-name.onmicrosoft.com/oauth2/authresp
    ```

    Por ejemplo, `https://fabrikam.b2clogin.com/fabrikam.onmicrosoft.com/oauth2/authresp`.

1. Seleccione **Registrar**. Anote el **Id. de aplicación (cliente)** para usarlo en un paso posterior.
1. Seleccione **Certificados y secretos** y luego seleccione **Nuevo secreto de cliente**.
1. En **Descripción**, escriba una descripción para el secreto, seleccione una fecha de expiración y seleccione **Agregar**. Registre el valor **Value** del secreto para usarlo en un paso posterior.

## <a name="configuring-optional-claims"></a>Configuración de notificaciones opcionales

Si quiere obtener las notificaciones `family_name` y `given_name` de Azure AD, puede configurar notificaciones opcionales para la aplicación en la interfaz de usuario de Azure Portal o el manifiesto de aplicación. Para obtener más información, consulte [Procedimientos: Proporcionar notificaciones opcionales a la aplicación de Azure AD](../active-directory/develop/active-directory-optional-claims.md).

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Busque y seleccione **Azure Active Directory**.
1. En la sección **Administrar**, seleccione **Registros de aplicaciones**.
1. Seleccione en la lista la aplicación para la que desea configurar notificaciones opcionales.
1. En la sección **Administrar**, seleccione **Configuración del token**.
1. Seleccione **Agregar notificación opcional**.
1. En **Tipo de token**, seleccione **ID**.
1. Seleccione las notificaciones opcionales que va a agregar, `family_name` y `given_name`.
1. Haga clic en **Agregar**.

## <a name="create-a-policy-key"></a>Creación de una clave de directiva

Debe almacenar la clave de la aplicación que creó en el inquilino de Azure AD B2C.

1. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Seleccione el filtro **Directorio y suscripción** en el menú superior y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
1. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
1. En **Directivas**, seleccione **Identity Experience Framework**.
1. Seleccione **Claves de directiva** y luego **Agregar**.
1. En **Opciones**, elija `Manual`.
1. Escriba un **nombre** para la clave de directiva. Por ejemplo, `AADAppSecret`.  El prefijo `B2C_1A_` se agrega automáticamente al nombre de la clave cuando se crea, por lo que hace referencia a *B2C_1A_AADAAppSecret* en el código XML de la siguiente sección.
1. En **Secreto**, escriba el secreto de cliente que registró previamente.
1. En **Uso de claves**, seleccione `Signature`.
1. Seleccione **Crear**.

## <a name="add-a-claims-provider"></a>Incorporación de un proveedor de notificaciones

Si quiere que los usuarios inicien sesión con Azure AD, deberá definir Azure AD como proveedor de notificaciones con el que Azure AD B2C se comunique mediante un punto de conexión. El punto de conexión proporciona un conjunto de notificaciones que Azure AD B2C usa para comprobar que un usuario concreto se ha autenticado.

Para definir Azure AD como proveedor de notificaciones, agregue el elemento **ClaimsProvider** al archivo de extensión de la directiva.

1. Abra el archivo *TrustFrameworkExtensions.xml*.
1. Busque el elemento **ClaimsProviders**. Si no existe, agréguelo debajo del elemento raíz.
1. Agregue un nuevo elemento **ClaimsProvider** tal como se muestra a continuación:

    ```xml
    <ClaimsProvider>
      <Domain>commonaad</Domain>
      <DisplayName>Common AAD</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Common-AAD">
          <DisplayName>Multi-Tenant AAD</DisplayName>
          <Description>Login with your Contoso account</Description>
          <Protocol Name="OpenIdConnect"/>
          <Metadata>
            <Item Key="METADATA">https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration</Item>
            <!-- Update the Client ID below to the Application ID -->
            <Item Key="client_id">00000000-0000-0000-0000-000000000000</Item>
            <Item Key="response_types">code</Item>
            <Item Key="scope">openid profile</Item>
            <Item Key="response_mode">form_post</Item>
            <Item Key="HttpBinding">POST</Item>
            <Item Key="UsePolicyInRedirectUri">false</Item>
            <Item Key="DiscoverMetadataByTokenIssuer">true</Item>
            <!-- The key below allows you to specify each of the Azure AD tenants that can be used to sign in. Update the GUIDs below for each tenant. -->
            <Item Key="ValidTokenIssuerPrefixes">https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000,https://login.microsoftonline.com/11111111-1111-1111-1111-111111111111</Item>
            <!-- The commented key below specifies that users from any tenant can sign-in. Uncomment if you would like anyone with an Azure AD account to be able to sign in. -->
            <!-- <Item Key="ValidTokenIssuerPrefixes">https://login.microsoftonline.com/</Item> -->
          </Metadata>
          <CryptographicKeys>
            <Key Id="client_secret" StorageReferenceId="B2C_1A_AADAppSecret"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="oid"/>
            <OutputClaim ClaimTypeReferenceId="tenantId" PartnerClaimType="tid"/>
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name" />
            <OutputClaim ClaimTypeReferenceId="surName" PartnerClaimType="family_name" />
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication" AlwaysUseDefaultValue="true" />
            <OutputClaim ClaimTypeReferenceId="identityProvider" PartnerClaimType="iss" />
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

1. En el elemento **ClaimsProvider**, actualice el valor de **Domain** a un valor único que pueda usarse para distinguirlo de otros proveedores de identidades.
1. En el elemento **TechnicalProfile**, actualice el valor de **DisplayName**, por ejemplo, `Contoso Employee`. Este valor se muestra en el botón de inicio de sesión de la página de inicio de sesión.
1. Establezca **client_id** en el identificador de aplicación de la aplicación multiinquilino de Azure AD que registró anteriormente.
1. En **CryptographicKeys**, actualice el valor de **StorageReferenceId** con el nombre de la clave de directiva que creó anteriormente. Por ejemplo, `B2C_1A_AADAppSecret`.

### <a name="restrict-access"></a>Restricción del acceso

> [!NOTE]
> Al usar `https://login.microsoftonline.com/` como valor para **ValidTokenIssuerPrefixes**, se permite a todos los usuarios de Azure AD iniciar sesión en la aplicación.

Tiene que actualizar la lista de emisores de tokens válidos y restringir el acceso a la lista específica de usuarios inquilinos de Azure AD que pueden iniciar sesión.

Para obtener los valores, examine los metadatos de descubrimiento de OpenID Connect de cada uno de los inquilinos específicos de Azure AD desde los que le gustaría que los usuarios iniciaran sesión. El formato de la dirección URL de metadatos es similar a `https://login.microsoftonline.com/your-tenant/v2.0/.well-known/openid-configuration`, donde `your-tenant` es el nombre del inquilino de Azure AD. Por ejemplo:

`https://login.microsoftonline.com/fabrikam.onmicrosoft.com/v2.0/.well-known/openid-configuration`

Siga estos pasos para cada inquilino de Azure AD que se deba usar para iniciar sesión:

1. Abra el explorador y vaya a la dirección URL de metadatos de OpenID Connect para el inquilino. Busque el objeto **issuer** y anote su valor. Debe tener un aspecto similar a `https://login.microsoftonline.com/00000000-0000-0000-0000-000000000000/`.
1. Copie y pegue el valor en la clave **ValidTokenIssuerPrefixes**. Separe varios emisores con una coma. Un ejemplo con dos emisores aparece en el ejemplo de código XML `ClaimsProvider` anterior.

### <a name="upload-the-extension-file-for-verification"></a>Carga del archivo de extensión para su comprobación

Por el momento, ha configurado la directiva para que Azure AD B2C sepa cómo comunicarse con su directorio de Azure AD. Pruebe a cargar el archivo de extensión de la directiva para confirmar que no tiene problemas.

1. En la página **Directivas personalizadas** del inquilino de Azure AD B2C, seleccione **Cargar directiva**.
2. Habilite **Sobrescribir la directiva, si existe**, y busque y seleccione el archivo *TrustFrameworkExtensions.xml*.
3. Seleccione **Cargar**.

## <a name="register-the-claims-provider"></a>Registro del proveedor de notificaciones

El proveedor de identidades ya se ha configurado, pero no está disponible en ninguna de las pantallas de registro/inicio de sesión. Para que esté disponible, debe crear un duplicado de un recorrido del usuario de plantilla existente que modificaremos a continuación para que también tenga el proveedor de identidades de Azure AD.

1. Abra el archivo *TrustFrameworkBase.xml* del paquete de inicio.
2. Busque y copie todo el contenido del elemento **UserJourney** que incluye `Id="SignUpOrSignIn"`.
3. Abra el archivo *TrustFrameworkExtensions.xml* y busque el elemento **UserJourneys**. Si el elemento no existe, agréguelo.
4. Pegue todo el contenido del elemento **UserJourney** que ha copiado como elemento secundario del elemento **UserJourneys**.
5. Cambie el identificador del recorrido del usuario. Por ejemplo, `SignUpSignInContoso`.

### <a name="display-the-button"></a>Visualización del botón

El elemento **ClaimsProviderSelection** es análogo a un botón del proveedor de identidades en una pantalla de registro o de inicio de sesión. Si agrega un elemento **ClaimsProviderSelection** a Azure AD, se muestra un nuevo botón cuando un usuario llega a la página.

1. Busque el elemento **OrchestrationStep** que incluya `Order="1"` en el recorrido del usuario que creó en *TrustFrameworkExtensions.xml*.
1. En **ClaimsProviderSelects**, agregue el siguiente elemento. Establezca un valor adecuado en **TargetClaimsExchangeId**, por ejemplo, `AzureADExchange`:

    ```xml
    <ClaimsProviderSelection TargetClaimsExchangeId="AzureADExchange" />
    ```

### <a name="link-the-button-to-an-action"></a>Vincular el botón a una acción

Ahora que hay un botón colocado, es preciso vincularlo a una acción. En este caso, la acción es para que Azure AD B2C se comunique con Azure AD para recibir un token. Para vincular un botón a una acción, vincule el perfil técnico del proveedor de notificaciones de Azure AD.

1. Busque el elemento **OrchestrationStep** que incluye `Order="2"` en el recorrido del usuario.
2. Al agregar el siguiente elemento **ClaimsExchange**, asegúrese de usar el mismo valor para el elemento **Id** que el que usó en **TargetClaimsExchangeId**:

    ```xml
    <ClaimsExchange Id="AzureADExchange" TechnicalProfileReferenceId="Common-AAD" />
    ```

    Actualice el valor de **TechnicalProfileReferenceId** al elemento **Id** del perfil técnico que creó anteriormente. Por ejemplo, `Common-AAD`.

3. Guarde el archivo *TrustFrameworkExtensions.xml* y cárguelo de nuevo a fin de verificarlo.

## <a name="create-an-azure-ad-b2c-application"></a>Creación de una aplicación de Azure AD B2C

La comunicación con Azure AD B2C se produce mediante una aplicación que se registra en el inquilino B2C. En esta sección se enumeran los pasos opcionales que puede llevar a cabo para crear una aplicación de prueba, si aún no lo ha hecho.

[!INCLUDE [active-directory-b2c-appreg-idp](../../includes/active-directory-b2c-appreg-idp.md)]

## <a name="update-and-test-the-relying-party-file"></a>Actualización y prueba del archivo del usuario de confianza

Actualice el archivo de usuario de confianza (RP) que inicia el recorrido del usuario que ha creado:

1. Realice una copia del archivo *SignUpOrSignIn.xml* en el directorio de trabajo y cámbiele el nombre. Por ejemplo, cámbiele el nombre a *SignUpSignContoso.xml*.
1. Abra el nuevo archivo y actualice el valor del atributo **PolicyId** del elemento **TrustFrameworkPolicy** con un valor único. Por ejemplo, `SignUpSignInContoso`.
1. Actualice el valor de **PublicPolicyUri** con el URI para la directiva. Por ejemplo, `http://contoso.com/B2C_1A_signup_signin_contoso`.
1. Actualice el valor del atributo **ReferenceId** en **DefaultUserJourney** para que coincida con el identificador del recorrido del usuario que creó anteriormente. Por ejemplo, *SignUpSignInContoso*.
1. Guarde los cambios y cargue el archivo.
1. En las **directivas personalizadas** cargadas, seleccione la directiva recién creada de la lista.
1. En el menú desplegable **Seleccionar aplicación**, seleccione la aplicación de Azure AD B2C que creó anteriormente. Por ejemplo, *testapp1*.
1. Copie el valor de **Ejecutar punto de conexión ahora** y ábralo en una ventana privada del explorador web, por ejemplo, en modo incógnito en Google Chrome o en una ventana de InPrivate en Microsoft Edge. De esta forma, podrá probar el recorrido completo del usuario sin usar ninguna credencial de Azure AD actualmente almacenada en caché.
1. Seleccione el botón de inicio de sesión de Azure AD, por ejemplo *Empleado de Contoso* y, luego, escriba las credenciales de un usuario en uno de sus inquilinos de la organización de Azure AD. Se le pedirá que autorice la aplicación y que luego escriba la información de su perfil.

Si el proceso de inicio de sesión se realiza correctamente, el explorador se redirige a `https://jwt.ms`, que muestra el contenido del token devuelto por Azure AD B2C.

Para probar la funcionalidad de inicio de sesión de varios inquilinos, realice los dos últimos pasos con las credenciales de un usuario que exista en otro inquilino de Azure AD.

## <a name="next-steps"></a>Pasos siguientes

Al trabajar con directivas personalizadas, en ocasiones es posible que necesite información adicional al solucionar problemas de una directiva durante su desarrollo.

Para ayudar a diagnosticar problemas, puede poner temporalmente la directiva en "modo de programador" y recopilar registros con Azure Application Insights. Descubra cómo en [Azure Active Directory B2C: Recopilación de registros](troubleshoot-with-application-insights.md).
