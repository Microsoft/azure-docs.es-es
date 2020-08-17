---
title: Uso de MSAL.js con Azure AD B2C
titleSuffix: Microsoft identity platform
description: La biblioteca de autenticación de Microsoft para JavaScript (MSAL.js) permite que las aplicaciones trabajen con Azure AD B2C y adquieran tokens para llamar a API web protegidas. Estas API web pueden ser Microsoft Graph, otras API de Microsoft, API web de terceros o su propia API web.
services: active-directory
author: negoe
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 06/05/2020
ms.author: negoe
ms.reviewer: nacanuma
ms.custom: aaddev
ms.openlocfilehash: 13b478e85278827258ea2fc25a0ee4298039fb1c
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119798"
---
# <a name="use-microsoft-authentication-library-for-javascript-to-work-with-azure-ad-b2c"></a>Uso de la biblioteca de autenticación de Microsoft para JavaScript a fin de trabajar con Azure AD B2C

La [biblioteca de autenticación de Microsoft para JavaScript (MSAL.js)](https://github.com/AzureAD/microsoft-authentication-library-for-js) permite que los desarrolladores de JavaScript autentiquen a los usuarios con identidades sociales y locales mediante [Azure Active Directory B2C](../../active-directory-b2c/overview.md) (Azure AD B2C).

Al usar Azure AD B2C como un servicio de administración de identidades, puede personalizar y controlar la manera en que los clientes se registran, inician sesión y administran sus perfiles al usar las aplicaciones. Azure AD B2C también permite personalizar la marca y la interfaz de usuario que muestra la aplicación durante el proceso de autenticación.

En las secciones siguientes se muestran los procedimientos para realizar lo siguiente:

- Proteger una API web de Node.Js
- Admitir el inicio de sesión en una aplicación de página única (SPA) y llamar a *esa* API web protegida
- Habilitar la compatibilidad con el restablecimiento de contraseña

## <a name="prerequisites"></a>Requisitos previos

Si aún no ha creado su propio [inquilino de Azure AD B2C](../../active-directory-b2c/tutorial-create-tenant.md), cree uno ahora.

## <a name="nodejs-web-api"></a>API web de Node.js

En los pasos siguientes se muestra cómo una **API web** puede usar Azure AD B2C para protegerse y exponer ámbitos seleccionados en una aplicación cliente.

MSAL.js para Node está actualmente en desarrollo. Para obtener más información, vea la [hoja de ruta](https://github.com/AzureAD/microsoft-authentication-library-for-js/wiki#roadmap) en GitHub. Actualmente se recomienda usar [passport-azure-ad](https://github.com/AzureAD/passport-azure-ad), una biblioteca de autenticación para Node.js que desarrolla y respalda Microsoft.

### <a name="step-1-register-your-application"></a>Paso 1: Registrar su aplicación

Para proteger la API web con Azure AD B2C, primero debe registrarla. Consulte [Registrar la aplicación](../../active-directory-b2c/add-web-api-application.md) para obtener instrucciones detalladas.

### <a name="step-2-download-the-sample-application"></a>Paso 2: Descarga de la aplicación de ejemplo

Descargue el ejemplo como archivo ZIP o clónelo desde GitHub:

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi.git
```

### <a name="step-3-configure-authentication"></a>Paso 3: Configurar la autenticación

1. Abra el archivo `config.js` del ejemplo.

2. Configure el ejemplo con las credenciales de la aplicación que obtuvo anteriormente al registrar la aplicación. Cambie las líneas de código siguiente reemplazando los valores por los nombres de clientID, host, tenantId y nombre de la directiva.

```JavaScript
const clientID = "<Application ID for your Node.js web API - found on Properties page in Azure portal e.g. 93733604-cc77-4a3c-a604-87084dd55348>";
const b2cDomainHost = "<Domain of your B2C host eg. fabrikamb2c.b2clogin.com>";
const tenantId = "<your-tenant-ID>.onmicrosoft.com"; // Alternatively, you can use your Directory (tenant) ID (GUID)
const policyName = "<Name of your sign in / sign up policy, e.g. B2C_1_signupsignin1>";
```

Para más información, consulte este [ejemplo de API web B2C de Node.js](https://github.com/Azure-Samples/active-directory-b2c-javascript-nodejs-webapi).

## <a name="javascript-spa"></a>JavaScript SPA

En los pasos siguientes se muestra cómo una **aplicación de página única** puede usar Azure AD B2C para el registro, el inicio de sesión y las llamadas a una API web protegida.

### <a name="step-1-register-your-application"></a>Paso 1: Registrar su aplicación

Para implementar la autenticación, primero debe registrar la aplicación. Consulte [Registrar la aplicación](../../active-directory-b2c/tutorial-register-applications.md) para obtener instrucciones detalladas.

### <a name="step-2-download-the-sample-application"></a>Paso 2: Descarga de la aplicación de ejemplo

Descargue el [archivo .ZIP](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp/archive/master.zip) del ejemplo de código o clone el repositorio de GitHub:

```console
git clone https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp.git
```

### <a name="step-3-configure-authentication"></a>Paso 3: Configurar la autenticación

Hay dos puntos de interés en la configuración de la aplicación:

- Configuración de un punto de conexión de API y los ámbitos expuestos
- Configuración de los parámetros de autenticación y los ámbitos de token

1. Abra el archivo *apiConfig.js* en el ejemplo.

2. Configure el ejemplo con los parámetros que obtuvo anteriormente al registrar la API web. Cambie las líneas de código siguiente reemplazando los valores por la dirección de la API web y los ámbitos expuestos.

   ```javascript
    // The current application coordinates were pre-registered in a B2C tenant.
    const apiConfig = {
        b2cScopes: ["https://fabrikamb2c.onmicrosoft.com/helloapi/demo.read"], //API scopes you exposed during api registration
        webApi: "https://fabrikamb2chello.azurewebsites.net/hello"
    };
   ```

1. Abra el archivo *authConfig.js* en el ejemplo.

1. Configure el ejemplo con los parámetros que obtuvo anteriormente al registrar la aplicación de página única. Cambie las líneas de código siguientes reemplazando los valores por su ClientId, los metadatos de autoridad y los ámbitos de solicitud de token.

   ```javascript
    // Config object to be passed to Msal on creation.
    const msalConfig = {
        auth: {
            clientId: "e760cab2-b9a1-4c0d-86fb-ff7084abd902",
            authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/B2C_1_signupsignin1",
            validateAuthority: false
        },
        cache: {
            cacheLocation: "localStorage", // This configures where your cache will be stored
            storeAuthStateInCookie: false // Set this to "true" to save cache in cookies
        }
    };

    // Add here scopes for id token to be used at the MS Identity Platform endpoint
    const loginRequest = {
        scopes: ["openid", "profile"],
    };
   ```

Para más información, consulte este [ejemplo de aplicación de página única de B2C de JavaScript](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp).

## <a name="support-password-reset"></a>Compatibilidad con el restablecimiento de contraseña

En esta sección, ampliará la aplicación de página única para usar el flujo de usuario de restablecimiento de contraseña de Azure AD B2C. Aunque MSAL.js no admite actualmente varios flujos de usuarios o directivas personalizadas de forma nativa, se puede usar la biblioteca para administrar casos de uso comunes como el restablecimiento de contraseña.

En los pasos siguientes se da por hecho que ya se han seguido los pasos descritos en la sección anterior [SPA de JavaScript](#javascript-spa).

### <a name="step-1-define-the-authority-string-for-password-reset-user-flow"></a>Paso 1: Definir la cadena de autoridad para el flujo de usuario de restablecimiento de contraseña

1. En primer lugar, cree un objeto en el que se almacenen los URI de autoridad:

    ```javascript
        const b2cPolicies = {
            names: {
                signUpSignIn: "b2c_1_susi",
                forgotPassword: "b2c_1_reset"
            },
            authorities: {
                signUpSignIn: {
                    authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_susi",
                },
                forgotPassword: {
                    authority: "https://fabrikamb2c.b2clogin.com/fabrikamb2c.onmicrosoft.com/b2c_1_reset",
                },
            },
        }
    ```

1. Después, inicialice el objeto MSAL con la directiva `signInSignUp` como predeterminada (vea el fragmento de código anterior). Cuando un usuario intenta iniciar sesión, se le presenta la pantalla siguiente:

    :::image type="content" source="media/msal-b2c-overview/user-journey-01-signin.png" alt-text="Pantalla de inicio de sesión que muestra Azure AD B2C":::

### <a name="step-2-catch-and-handle-authentication-errors-in-your-login-method"></a>Paso 2: Detectar y controlar errores de autenticación en el método de inicio de sesión

Cuando un usuario selecciona **Contraseña olvidada**, la aplicación produce un error que se debe detectar en el código y, después, controlar mediante la presentación del flujo de usuario adecuado. En este caso, el flujo de restablecimiento de contraseña `b2c_1_reset`.

1. Amplíe el método de inicio de sesión de la siguiente manera:

    ```javascript
    function signIn() {
      myMSALObj.loginPopup(loginRequest)
        .then(loginResponse => {
            console.log("id_token acquired at: " + new Date().toString());

            if (myMSALObj.getAccount()) {
              updateUI();
            }

        }).catch(function (error) {
          console.log(error);

          // error handling
          if (error.errorMessage) {
            // check for forgot password error
            if (error.errorMessage.indexOf("AADB2C90118") > -1) {

              //call login method again with the password reset user flow
              myMSALObj.loginPopup(b2cPolicies.authorities.forgotPassword)
                .then(loginResponse => {
                  console.log(loginResponse);
                  window.alert("Password has been reset successfully. \nPlease sign-in with your new password.");
                })
            }
          }
        });
    }
    ```

1. En el fragmento de código anterior se muestra cómo mostrar la pantalla de restablecimiento de contraseña después de detectar el error con el código `AADB2C90118`.

    Después de restablecer la contraseña, se devuelve al usuario a la aplicación para iniciar sesión de nuevo.

    :::image type="content" source="media/msal-b2c-overview/user-journey-02-password-reset.png" alt-text="Pantallas de flujo de restablecimiento de contraseña que muestra Azure AD B2C" border="false":::

    Para obtener más información sobre los códigos de error y el control de excepciones, vea [Códigos de excepción y de error de MSAL](msal-handling-exceptions.md).

## <a name="next-steps"></a>Pasos siguientes

Obtener más información sobre estos conceptos de Azure AD B2C:

- [Flujos de usuario](../../active-directory-b2c/tutorial-create-user-flows.md)
- [Directivas personalizadas](../../active-directory-b2c/custom-policy-get-started.md)
- [Personalización de la experiencia de usuario](../../active-directory-b2c/custom-policy-configure-user-input.md)