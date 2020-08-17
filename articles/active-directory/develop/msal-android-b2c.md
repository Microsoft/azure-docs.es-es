---
title: Azure AD B2C (MSAL para Android) | Azure
titleSuffix: Microsoft identity platform
description: Aprenda las consideraciones específicas de uso de Azure AD B2C con la biblioteca de autenticación de Microsoft para Android (MSAL.Android)
services: active-directory
author: brianmel
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 9/18/2019
ms.author: brianmel
ms.reviewer: rapong
ms.custom: aaddev
ms.openlocfilehash: 0ad5fab685757d2efd91cd1df0e48a5f1258d17e
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119885"
---
# <a name="use-msal-for-android-with-b2c"></a>Uso de MSAL para Android con B2C

La biblioteca de autenticación de Microsoft (MSAL) permite a los desarrolladores de aplicaciones autenticar a los usuarios con identidades sociales y locales mediante [Azure Active Directory B2C (Azure AD B2C)](../../active-directory-b2c/index.yml). Azure AD B2C es un servicio de administración de identidades. Utilícelo para personalizar y controlar el modo en que los clientes se suscriben, inician sesión y administran sus perfiles cuando usan las aplicaciones.

## <a name="configure-known-authorities-and-redirect-uri"></a>Configuración de autoridades conocidas y URI de redireccionamiento

En MSAL para Android, las directivas de B2C (recorridos de usuario) se configuran como autoridades individuales.

Dada una aplicación B2C que tiene dos directivas:
- Registro / inicio de sesión
    * Llamada `B2C_1_SISOPolicy`
- Edición de perfil
    * Llamada `B2C_1_EditProfile`

El archivo de configuración para la aplicación declararía dos `authorities`. Una para cada directiva. La propiedad `type` de cada autoridad es `B2C`.

### `app/src/main/res/raw/msal_config.json`
```json
{
    "client_id": "<your_client_id_here>",
    "redirect_uri": "<your_redirect_uri_here>",
    "authorities": [{
            "type": "B2C",
            "authority_url": "https://contoso.b2clogin.com/tfp/contoso.onmicrosoft.com/B2C_1_SISOPolicy/",
            "default": true
        },
        {
            "type": "B2C",
            "authority_url": "https://contoso.b2clogin.com/tfp/contoso.onmicrosoft.com/B2C_1_EditProfile/"
        }
    ]
}
```

`redirect_uri` debe estar registrado en la configuración de la aplicación y también en `AndroidManifest.xml` para admitir el redireccionamiento durante el [flujo de concesión de código de autorización](../../active-directory-b2c/authorization-code-flow.md).

## <a name="initialize-ipublicclientapplication"></a>Inicialización de IPublicClientApplication

`IPublicClientApplication` se construye mediante Factory Method para que la configuración de la aplicación se pueda analizar de forma asincrónica.

```java
PublicClientApplication.createMultipleAccountPublicClientApplication(
    context, // Your application Context
    R.raw.msal_config, // Id of app JSON config
    new IPublicClientApplication.ApplicationCreatedListener() {
        @Override
        public void onCreated(IMultipleAccountPublicClientApplication pca) {
            // Application has been initialized.
        }

        @Override
        public void onError(MsalException exception) {
            // Application could not be created.
            // Check Exception message for details.
        }
    }
);
```

## <a name="interactively-acquire-a-token"></a>Adquisición de un token de forma interactiva

Para adquirir un token de forma interactiva con MSAL, cree una instancia de `AcquireTokenParameters` y proporciónesela al método `acquireToken`. La solicitud de token siguiente usa la autoridad `default`.

```java
IMultipleAccountPublicClientApplication pca = ...; // Initialization not shown

AcquireTokenParameters parameters = new AcquireTokenParameters.Builder()
    .startAuthorizationFromActivity(activity)
    .withScopes(Arrays.asList("https://contoso.onmicrosoft.com/contosob2c/read")) // Provide your registered scope here
    .withPrompt(Prompt.LOGIN)
    .callback(new AuthenticationCallback() {
        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            // Token request was successful, inspect the result
        }

        @Override
        public void onError(MsalException exception) {
            // Token request was unsuccessful, inspect the exception
        }

        @Override
        public void onCancel() {
            // The user cancelled the flow
        }
    }).build();

pca.acquireToken(parameters);
```

## <a name="silently-renew-a-token"></a>Renovación de un token de forma silenciosa

Para adquirir un token de forma silenciosa con MSAL, cree una instancia de `AcquireTokenSilentParameters` y proporciónesela al método `acquireTokenSilentAsync`. A diferencia del método `acquireToken`, se debe especificar `authority` para adquirir un token de forma silenciosa.

```java
IMultilpeAccountPublicClientApplication pca = ...; // Initialization not shown
AcquireTokenSilentParameters parameters = new AcquireTokenSilentParameters.Builder()
    .withScopes(Arrays.asList("https://contoso.onmicrosoft.com/contosob2c/read")) // Provide your registered scope here
    .forAccount(account)
    // Select a configured authority (policy), mandatory for silent auth requests
    .fromAuthority("https://contoso.b2clogin.com/tfp/contoso.onmicrosoft.com/B2C_1_SISOPolicy/")
    .callback(new SilentAuthenticationCallback() {
        @Override
        public void onSuccess(IAuthenticationResult authenticationResult) {
            // Token request was successful, inspect the result
        }

        @Override
        public void onError(MsalException exception) {
            // Token request was unsuccesful, inspect the exception
        }
    })
    .build();

pca.acquireTokenSilentAsync(parameters);
```

## <a name="specify-a-policy"></a>Especificación de una directiva

Dado que las directivas de B2C se representan como autoridades independientes, la invocación de una directiva distinta de la predeterminada se logra especificando una cláusula `fromAuthority` al construir los parámetros `acquireToken` o `acquireTokenSilent`.  Por ejemplo:

```java
AcquireTokenParameters parameters = new AcquireTokenParameters.Builder()
    .startAuthorizationFromActivity(activity)
    .withScopes(Arrays.asList("https://contoso.onmicrosoft.com/contosob2c/read")) // Provide your registered scope here
    .withPrompt(Prompt.LOGIN)
    .callback(...) // provide callback here
    .fromAuthority("<url_of_policy_defined_in_configuration_json>")
    .build();
```

## <a name="handle-password-change-policies"></a>Administración de directivas de cambio de contraseña

El flujo de usuario de registro o de inicio de sesión de la cuenta local muestra el vínculo "¿Ha olvidado la contraseña?" Al hacer clic en este vínculo, no se desencadena automáticamente ningún flujo de usuario de restablecimiento de contraseña,

sino que se devuelve a la aplicación el código de error `AADB2C90118`. La aplicación debería controlar este código de error mediante la ejecución de un flujo de usuario específico que restablezca la contraseña.

Para detectar un código de error de restablecimiento de contraseña, se puede usar la siguiente implementación en `AuthenticationCallback`:

```java
new AuthenticationCallback() {

    @Override
    public void onSuccess(IAuthenticationResult authenticationResult) {
        // ..
    }

    @Override
    public void onError(MsalException exception) {
        final String B2C_PASSWORD_CHANGE = "AADB2C90118";

        if (exception.getMessage().contains(B2C_PASSWORD_CHANGE)) {
            // invoke password reset flow
        }
    }

    @Override
    public void onCancel() {
        // ..
    }
}
```

## <a name="use-iauthenticationresult"></a>Uso de IAuthenticationResult

Una adquisición correcta del token da como resultado un objeto `IAuthenticationResult`. Contiene el token de acceso, las notificaciones de usuario y los metadatos.

### <a name="get-the-access-token-and-related-properties"></a>Obtención del token de acceso y propiedades relacionadas

```java
// Get the raw bearer token
String accessToken = authenticationResult.getAccessToken();

// Get the scopes included in the access token
String[] accessTokenScopes = authenticationResult.getScope();

// Gets the access token's expiry
Date expiry = authenticationResult.getExpiresOn();

// Get the tenant for which this access token was issued
String tenantId = authenticationResult.getTenantId();
```

### <a name="get-the-authorized-account"></a>Obtención de la cuenta autorizada

```java
// Get the account from the result
IAccount account = authenticationResult.getAccount();

// Get the id of this account - note for B2C, the policy name is a part of the id
String id = account.getId();

// Get the IdToken Claims
//
// For more information about B2C token claims, see reference documentation
// https://docs.microsoft.com/azure/active-directory-b2c/active-directory-b2c-reference-tokens
Map<String, ?> claims = account.getClaims();

// Get the 'preferred_username' claim through a convenience function
String username = account.getUsername();

// Get the tenant id (tid) claim through a convenience function
String tenantId = account.getTenantId();
```

### <a name="idtoken-claims"></a>Notificaciones de IdToken

Las notificaciones devueltas en IdToken las rellena el servicio de token de seguridad (STS) y no MSAL. En función del proveedor de identidades (IdP) utilizado, puede que falten algunas notificaciones. Algunos proveedores de identidades no proporcionan actualmente la notificación `preferred_username`. Como MSAL usa esta notificación para el almacenamiento en caché, se utiliza un valor de marcador de posición, `MISSING FROM THE TOKEN RESPONSE`, en su lugar. Para más información sobre las notificaciones de IdToken de B2C, consulte [Información general de tokens de Azure Active Directory B2C](../../active-directory-b2c/tokens-overview.md#claims).

## <a name="managing-accounts-and-policies"></a>Administración de cuentas y directivas

B2C trata cada directiva como una autoridad independiente. Por lo tanto, los tokens de acceso, los tokens de actualización y los tokens de identificador devueltos por cada directiva no son intercambiables. Esto significa que cada directiva devuelve un objeto `IAccount` independiente cuyos tokens no se pueden usar para invocar a otras directivas.

Cada directiva agrega una `IAccount` a la memoria caché para cada usuario. Si un usuario inicia sesión en una aplicación e invoca a dos directivas, tendrá dos `IAccount`. Para quitar este usuario de la memoria caché, debe llamar a `removeAccount()` para cada directiva.

Al renovar los tokens de una directiva con `acquireTokenSilent`, proporcione la misma `IAccount` devuelta por las invocaciones anteriores de la directiva a `AcquireTokenSilentParameters`. Proporcionar una cuenta devuelta por otra directiva producirá un error.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre Azure Active Directory B2C (Azure AD B2C) en [¿Qué es Azure Active Directory B2C?](../../active-directory-b2c/overview.md)