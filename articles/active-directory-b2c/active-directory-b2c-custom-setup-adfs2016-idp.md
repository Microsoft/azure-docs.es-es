---
title: Adición de ADFS como proveedor de identidades de SAML mediante directivas personalizadas en Azure Active Directory B2C | Microsoft Docs
description: Configurar ADFS 2016 mediante el protocolo SAML y las directivas personalizadas en Azure Active Directory B2C
services: active-directory-b2c
author: davidmu1
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 11/07/2018
ms.author: davidmu
ms.subservice: B2C
ms.openlocfilehash: df4acf72a1a6ea134e1192512fda1d8cf1e92f0a
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65767990"
---
# <a name="add-adfs-as-a-saml-identity-provider-using-custom-policies-in-azure-active-directory-b2c"></a>Agregar ADFS como proveedor de identidades de SAML mediante las directivas personalizadas de Azure Active Directory B2C

[!INCLUDE [active-directory-b2c-advanced-audience-warning](../../includes/active-directory-b2c-advanced-audience-warning.md)]

En este artículo se muestra cómo habilitar el inicio de sesión para una cuenta de usuario de ADFS mediante el uso de [directivas personalizadas](active-directory-b2c-overview-custom.md) en Azure Active Directory (Azure AD) B2C. Habilite el inicio de sesión mediante la adición de un [perfil técnico de SAML](saml-technical-profile.md) a una directiva personalizada.

## <a name="prerequisites"></a>Requisitos previos

- Siga los pasos de [Introducción a las directivas personalizadas en Azure Active Directory B2C](active-directory-b2c-get-started-custom.md).
- Asegúrese de que tiene acceso a un archivo .pfx del certificado con una clave privada. Puede generar su propio certificado firmado y cargarlo en Azure AD B2C. Azure AD B2C usa este certificado para firmar la solicitud SAML enviada al proveedor de identidades SAML.

## <a name="create-a-policy-key"></a>Creación de una clave de directiva

Debe almacenar el certificado en el inquilino de Azure AD B2C.

1. Inicie sesión en el [Azure Portal](https://portal.azure.com/).
2. Asegúrese de que usa el directorio que contiene al inquilino de Azure AD B2C. Seleccione el **filtro de directorio y suscripción** en el menú superior y elija el directorio que contiene el inquilino.
3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
4. En la página de introducción, seleccione **Identity Experience Framework**.
5. Seleccione **Claves de directiva** y luego **Agregar**.
6. En **Opciones**, elija `Upload`.
7. Escriba un **nombre** para la clave de directiva. Por ejemplo, `SamlCert`. Se agregará el prefijo `B2C_1A_` automáticamente al nombre de la clave.
8. Busque el archivo .pfx de certificado con la clave privada y selecciónelo.
9. Haga clic en **Create**(Crear).

## <a name="add-a-claims-provider"></a>Incorporación de un proveedor de notificaciones

Si desea que los usuarios inicien sesión con una cuenta de ADFS, deberá definirla como un proveedor de notificaciones con el que Azure AD B2C pueda comunicarse mediante un punto de conexión. El punto de conexión proporciona un conjunto de notificaciones que Azure AD B2C usa para comprobar que un usuario concreto se ha autenticado. 

Puede definir una cuenta de ADFS como proveedor de notificaciones; para ello, agréguela al elemento **ClaimsProvider** en el archivo de extensión de la directiva.

1. Abra el archivo *TrustFrameworkExtensions.xml*.
2. Busque el elemento **ClaimsProviders**. Si no existe, agréguelo debajo del elemento raíz.
3. Agregue un nuevo elemento **ClaimsProvider** tal como se muestra a continuación:

    ```xml
    <ClaimsProvider>
      <Domain>contoso.com</Domain>
      <DisplayName>Contoso ADFS</DisplayName>
      <TechnicalProfiles>
        <TechnicalProfile Id="Contoso-SAML2">
          <DisplayName>Contoso ADFS</DisplayName>
          <Description>Login with your ADFS account</Description>
          <Protocol Name="SAML2"/>
          <Metadata>
            <Item Key="WantsEncryptedAssertions">false</Item>
            <Item Key="PartnerEntity">https://your-ADFS-domain/federationmetadata/2007-06/federationmetadata.xml</Item>
            <Item Key="XmlSignatureAlgorithm">Sha256</Item>
          </Metadata>
          <CryptographicKeys>
            <Key Id="SamlAssertionSigning" StorageReferenceId="B2C_1A_ADFSSamlCert"/>
            <Key Id="SamlMessageSigning" StorageReferenceId="B2C_1A_ADFSSamlCert"/>
          </CryptographicKeys>
          <OutputClaims>
            <OutputClaim ClaimTypeReferenceId="issuerUserId" PartnerClaimType="userPrincipalName" />
            <OutputClaim ClaimTypeReferenceId="givenName" PartnerClaimType="given_name"/>
            <OutputClaim ClaimTypeReferenceId="surname" PartnerClaimType="family_name"/>
            <OutputClaim ClaimTypeReferenceId="email" PartnerClaimType="email"/>
            <OutputClaim ClaimTypeReferenceId="displayName" PartnerClaimType="name"/>
            <OutputClaim ClaimTypeReferenceId="identityProvider" DefaultValue="contoso.com" />
            <OutputClaim ClaimTypeReferenceId="authenticationSource" DefaultValue="socialIdpAuthentication"/>
          </OutputClaims>
          <OutputClaimsTransformations>
            <OutputClaimsTransformation ReferenceId="CreateRandomUPNUserName"/>
            <OutputClaimsTransformation ReferenceId="CreateUserPrincipalName"/>
            <OutputClaimsTransformation ReferenceId="CreateAlternativeSecurityId"/>
            <OutputClaimsTransformation ReferenceId="CreateSubjectClaimFromAlternativeSecurityId"/>
          </OutputClaimsTransformations>
          <UseTechnicalProfileForSessionManagement ReferenceId="SM-Noop"/>
        </TechnicalProfile>
      </TechnicalProfiles>
    </ClaimsProvider>
    ```

4. Reemplace `your-ADFS-domain` por el nombre de su dominio de AD FS y reemplace el valor de la notificación de salida **identityProvider** por su DNS (valor arbitrario que indica el dominio).
5. Guarde el archivo.

### <a name="upload-the-extension-file-for-verification"></a>Carga del archivo de extensión para su comprobación

Por el momento, ha configurado la directiva para que Azure AD B2C sepa cómo comunicarse con la cuenta de ADFS. Pruebe a cargar el archivo de extensión de la directiva para confirmar que no tiene problemas.

1. En la página **Directivas personalizadas** del inquilino de Azure AD B2C, seleccione **Cargar directiva**.
2. Habilite **Sobrescribir la directiva, si existe**, y busque y seleccione el archivo *TrustFrameworkExtensions.xml*.
3. Haga clic en **Cargar**.

## <a name="register-the-claims-provider"></a>Registro del proveedor de notificaciones

En este punto, el proveedor de identidades ya se ha configurado, pero no está disponible en ninguna de las pantallas de registro o de inicio de sesión. Para que esté disponible, debe crear un duplicado de un recorrido del usuario de plantilla existente que modificaremos a continuación para que también tenga el proveedor de identidades de ADFS.

1. Abra el archivo *TrustFrameworkBase.xml* del paquete de inicio.
2. Busque y copie todo el contenido del elemento **UserJourney** que incluye `Id="SignUpOrSignIn"`.
3. Abra el archivo *TrustFrameworkExtensions.xml* y busque el elemento **UserJourneys**. Si el elemento no existe, agréguelo.
4. Pegue todo el contenido del elemento **UserJourney** que ha copiado como elemento secundario del elemento **UserJourneys**.
5. Cambie el identificador del recorrido del usuario. Por ejemplo, `SignUpSignInADFS`.

### <a name="display-the-button"></a>Visualización del botón

El elemento **ClaimsProviderSelection** es análogo a un botón de proveedor de identidades en una pantalla de registro o de inicio de sesión. Si agrega un elemento **ClaimsProviderSelection** para una cuenta de ADFS, se muestra un nuevo botón cuando un usuario llega a la página.

1. Busque el elemento **OrchestrationStep** que incluye `Order="1"` en el recorrido del usuario que ha creado.
2. En **ClaimsProviderSelections**, agregue el siguiente elemento. Establezca un valor adecuado en **TargetClaimsExchangeId**, por ejemplo, `ContosoExchange`:

    ```XML
    <ClaimsProviderSelection TargetClaimsExchangeId="ContosoExchange" />
    ```

### <a name="link-the-button-to-an-action"></a>Vincular el botón a una acción

Ahora que hay un botón colocado, es preciso vincularlo a una acción. En este caso, la acción es para que Azure AD B2C se comunique con una cuenta de ADFS para recibir un token.

1. Busque el elemento **OrchestrationStep** que incluye `Order="2"` en el recorrido del usuario.
2. Al agregar el siguiente elemento **ClaimsExchange**, asegúrese de usar el mismo valor para el identificador que usó para **TargetClaimsExchangeId**:

    ```XML
    <ClaimsExchange Id="ContosoExchange" TechnicalProfileReferenceId="Contoso-SAML2" />
    ```
    
    Cambie el valor de **TechnicalProfileReferenceId** para el identificador del perfil técnico que creó anteriormente. Por ejemplo, `Contoso-SAML2`.

3. Guarde el archivo *TrustFrameworkExtensions.xml* y cárguelo de nuevo a fin de verificarlo.


## <a name="configure-an-adfs-relying-party-trust"></a>Configuración de una relación de confianza para usuarios de confianza de ADFS

Para usar ADFS como proveedor de identidades en Azure AD B2C, debe crear una relación de confianza para usuarios de confianza de ADFS con los metadatos de SAML de Azure AD B2C. En el ejemplo siguiente se muestra una dirección URL a los metadatos SAML de un perfil técnico de Azure AD B2C:

```
https://login.microsoftonline.com/te/your-tenant/your-policy/samlp/metadata?idptp=your-technical-profile
```

Reemplace los valores siguientes:

- **your-tenant** por el nombre del inquilino, como inquilino.onmicrosoft.com.
- **your-policy** por el nombre de la directiva. Por ejemplo, B2C_1A_signup_signin_adfs.
- **su perfil de técnica** con el nombre de su perfil técnico del proveedor SAML identity. Por ejemplo, Contoso-SAML2.
 
Abra un explorador y vaya a esta dirección URL. Asegúrese de escribir la dirección URL correcta y que tenga acceso al archivo XML de metadatos. Para agregar una nueva relación de confianza para usuarios de confianza mediante el complemento de administración de ADFS y configurar manualmente los valores, realice el procedimiento siguiente en un servidor de federación. Para realizar este procedimiento, se requiere al menos la pertenencia al grupo **Administradores**, o equivalente, en el equipo local.

1. En el Administrador del servidor, seleccione **Herramientas** y luego **ADFS Management** (Administración de ADFS).
2. Seleccione **Agregar confianza para usuario de confianza**.
3. En la **página principal**, elija **Claims aware** (Compatible con notificaciones) y haga clic en **Iniciar**.
4. En la página **Seleccionar origen de datos**, seleccione **Import data about the relying party publish online or on a local network** (Importar datos sobre el usuario de confianza para publicar en línea o en una red local), proporcione la dirección URL de metadatos de Azure AD B2C y luego haga clic en **Siguiente**.
5. En la página **Especificar nombre para mostrar**, escriba un nombre en **Nombre para mostrar**. En **Notas** escriba una descripción de esta relación de confianza de usuario y luego haga clic en **Siguiente**.
6. En **Choose Access Control Policy** (Elegir directiva de control de acceso), seleccione una directiva y haga clic en **Siguiente**.
7. En la página **Ready to Add Trust** (Listo para agregar confianza), revise la configuración y luego haga clic en **Siguiente** para guardar la información de la relación de confianza para usuario autenticado.
8. En la página **Finalizar**, haga clic en **Cerrar**; esta acción muestra automáticamente el cuadro de diálogo **Edit Claim Rules** (Editar reglas de notificación).
9. Seleccione **Agregar regla**.  
10. En **Claim rule template** (Plantilla de regla de notificación), seleccione **Send LDAP attributes as claims** (Enviar atributos LDAP como notificaciones).
11. Proporcione un valor en **Claim rule name** (Nombre de la regla de notificación). Para **Attribute store** (Almacén de atributos), seleccione **Select Active Directory** (Seleccionar Active Directory), agregue las siguientes notificaciones y haga clic en **Finalizar** y luego en **Aceptar**.

    | Atributo LDAP | Tipo de notificación saliente |
    | -------------- | ------------------- |
    | User-Principal-Name | userPrincipalName |
    | Apellido | family_name |
    | Given-Name | given_name |
    | E-Mail-Address | email |
    | Display-Name | Nombre |
    
12.  Según el tipo de certificado, es posible que tenga que establecer el algoritmo HASH. En la ventana de propiedades de la relación de usuario de confianza (demo de B2C), haga clic en la pestaña **Avanzado** y cambie el valor de **Algoritmo hash seguro** a `SHA-256` y luego haga clic en **Aceptar**.  
13. En el Administrador del servidor, seleccione **Herramientas** y luego **ADFS Management** (Administración de ADFS).
14. Seleccione la relación de usuario de confianza que creó, seleccione **Actualizar a partir de los metadatos de federación** y luego haga clic en **Actualizar**. 

## <a name="create-an-azure-ad-b2c-application"></a>Creación de una aplicación de Azure AD B2C

La comunicación con Azure AD B2C se produce mediante una aplicación que se crea en el inquilino. En esta sección se enumeran los pasos opcionales que puede llevar a cabo para crear una aplicación de prueba, si aún no lo ha hecho.

1. Inicie sesión en el [Azure Portal](https://portal.azure.com).
2. Asegúrese de que usa el directorio que contiene el inquilino de Azure AD B2C. Para ello, haga clic en el **filtro de directorio y suscripción** en el menú superior y elija el directorio que contiene el inquilino.
3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.
4. Seleccione **Aplicaciones** y **Agregar**.
5. Escriba un nombre para la aplicación; por ejemplo, *testapp1*.
6. En **Aplicación web/API web**, seleccione `Yes` y escriba `https://jwt.ms` como **URL de respuesta**.
7. Haga clic en **Create**(Crear).

### <a name="update-and-test-the-relying-party-file"></a>Actualización y prueba del archivo del usuario de confianza

Actualice el archivo de usuario de confianza (RP) que inicia el recorrido del usuario que ha creado.

1. Realice una copia del archivo *SignUpOrSignIn.xml* en el directorio de trabajo y cámbiele el nombre. Por ejemplo, cambie su nombre a *SignUpSignInADFS.xml*.
2. Abra el nuevo archivo y actualice el valor del atributo **PolicyId** del elemento **TrustFrameworkPolicy** con un valor único. Por ejemplo, `SignUpSignInADFS`.
3. Actualice el valor de **PublicPolicyUri** con el URI para la directiva. Por ejemplo: `http://contoso.com/B2C_1A_signup_signin_adfs`
4. Cambie el valor del atributo **ReferenceId** del elemento **DefaultUserJourney** para que coincida con el identificador del nuevo recorrido del usuario que ha creado (SignUpSignInADFS).
5. Guarde los cambios, cargue el archivo y, a continuación, seleccione la nueva directiva en la lista.
6. Asegúrese de que la aplicación de Azure AD B2C que creó está seleccionada en el campo **Seleccionar aplicación** y pruébela; para ello, haga clic en **Ejecutar ahora**.

