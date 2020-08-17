---
title: Azure AD B2C y MSAL.NET
titleSuffix: Microsoft identity platform
description: Consideraciones de uso de Azure AD B2C con la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2020
ms.author: jeferrie
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: ed3e9da628ab779ab47673fa2ce728c5c25539be
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88166440"
---
# <a name="use-msalnet-to-sign-in-users-with-social-identities"></a>Use MSAL.NET para que los usuarios inicien sesión con identidades de redes sociales

Puede usar MSAL.NET para que los usuarios inicien sesión con identidades de redes sociales con [Azure Active Directory B2C (Azure AD B2C)](https://aka.ms/aadb2c). Azure AD B2C se basa en la noción de directivas. En MSAL.NET, especificar una directiva se traduce en proporcionar una autoridad.

- Al crear una instancia de la aplicación cliente pública, debe especificar la directiva como parte de la autoridad.
- Cuando quiera aplicar una directiva, invoque una invalidación de `AcquireTokenInteractive` que acepte el parámetro `authority`.

Este artículo se aplica a MSAL.NET 3.x. Para MSAL.NET 2.x, consulte los [detalles de Azure AD B2C en MSAL 2.x](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-Specifics-MSAL-2.x) en la wiki de MSAL.NET en GitHub.

## <a name="authority-for-an-azure-ad-b2c-tenant-and-policy"></a>Autoridad para una directiva e inquilino de Azure AD B2C

El formato de autoridad para Azure AD B2C es: `https://{azureADB2CHostname}/tfp/{tenant}/{policyName}`

- `azureADB2CHostname`: nombre del inquilino de Azure AD B2C más el host. Por ejemplo, *contosob2c.b2clogin.com*.
- `tenant`: nombre de dominio o identificador del directorio (inquilino) del inquilino de Azure AD B2C. Por ejemplo, *contosob2c.onmicrosoft.com* o un GUID, respectivamente.
- `policyName`: nombre del flujo de usuario o directiva personalizada que se aplicará. Por ejemplo, una directiva de registro o inicio de sesión como *b2c_1_susi*.

Para obtener más información acerca de las autoridades de Azure AD B2C, consulte [Establecimiento de direcciones URL de redireccionamiento en b2clogin.com](../../active-directory-b2c/b2clogin.md).

## <a name="instantiating-the-application"></a>Creación de instancias de la aplicación

Proporcione la autoridad mediante una llamada a `WithB2CAuthority()` al crear el objeto de aplicación:

```csharp
// Azure AD B2C Coordinates
public static string Tenant = "fabrikamb2c.onmicrosoft.com";
public static string AzureADB2CHostname = "fabrikamb2c.b2clogin.com";
public static string ClientID = "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6";
public static string PolicySignUpSignIn = "b2c_1_susi";
public static string PolicyEditProfile = "b2c_1_edit_profile";
public static string PolicyResetPassword = "b2c_1_reset";

public static string AuthorityBase = $"https://{AzureADB2CHostname}/tfp/{Tenant}/";
public static string Authority = $"{AuthorityBase}{PolicySignUpSignIn}";
public static string AuthorityEditProfile = $"{AuthorityBase}{PolicyEditProfile}";
public static string AuthorityPasswordReset = $"{AuthorityBase}{PolicyResetPassword}";

application = PublicClientApplicationBuilder.Create(ClientID)
               .WithB2CAuthority(Authority)
               .Build();
```

## <a name="acquire-a-token-to-apply-a-policy"></a>Adquisición de un token para aplicar la directiva

Adquirir un token para una API protegida de Azure AD B2C en una aplicación cliente pública requiere usar las invalidaciones con una autoridad:

```csharp
IEnumerable<IAccount> accounts = await application.GetAccountsAsync();
AuthenticationResult ar = await application.AcquireTokenInteractive(scopes)
                                           .WithAccount(GetAccountByPolicy(accounts, policy))
                                           .WithParentActivityOrWindow(ParentActivityOrWindow)
                                           .ExecuteAsync();
```

En el fragmento de código anterior:

- `policy` es una cadena que contiene el nombre del flujo de usuarios de Azure AD B2C o directiva personalizada (por ejemplo, `PolicySignUpSignIn`).
- `ParentActivityOrWindow` es necesario para Android (la actividad) y es opcional para otras plataformas que admiten una UI principal, como Windows en Microsoft Windows y UIViewController en iOS. Para obtener más información sobre el cuadro de diálogo de la interfaz de usuario, consulte [WithParentActivityOrWindow](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Acquiring-tokens-interactively#withparentactivityorwindow) en la wiki de MSAL.
- `GetAccountByPolicy(IEnumerable<IAccount>, string)` es un método que encuentra una cuenta para una directiva determinada. Por ejemplo:

  ```csharp
  private IAccount GetAccountByPolicy(IEnumerable<IAccount> accounts, string policy)
  {
      foreach (var account in accounts)
      {
          string userIdentifier = account.HomeAccountId.ObjectId.Split('.')[0];
          if (userIdentifier.EndsWith(policy.ToLower()))
              return account;
      }
      return null;
  }
  ```

Actualmente, para aplicar una directiva personalizada o flujo de usuario (por ejemplo, para permitir que el usuario edite su perfil o restablezca su contraseña), se usa una llamada a `AcquireTokenInteractive`. Para estas dos directivas, no usa el token devuelto ni el resultado de autenticación.

## <a name="profile-edit-policies"></a>Directivas de edición de perfiles

Para permitir que los usuarios inicien sesión con una identidad social y editen su perfil, aplique la directiva de edición de perfiles de Azure AD B2C.

Para ello, llame a `AcquireTokenInteractive` con la autoridad para esa directiva. Dado que el usuario ya ha iniciado sesión y tiene una sesión de cookies activa, use `Prompt.NoPrompt` para impedir que se muestre el cuadro de diálogo de selección de cuentas.

```csharp
private async void EditProfileButton_Click(object sender, RoutedEventArgs e)
{
    IEnumerable<IAccount> accounts = await app.GetAccountsAsync();
    try
    {
        var authResult = await app.AcquireToken(scopes:App.ApiScopes)
                            .WithAccount(GetUserByPolicy(accounts, App.PolicyEditProfile)),
                            .WithPrompt(Prompt.NoPrompt),
                            .WithB2CAuthority(App.AuthorityEditProfile)
                            .ExecuteAsync();
        DisplayBasicTokenInfo(authResult);
    }
    catch
    {
    }
}
```

## <a name="resource-owner-password-credentials-ropc"></a>Credenciales de contraseña del propietario del recurso (ROPC)

Para obtener más información sobre el flujo de ROPC, consulte [Iniciar sesión con la concesión de credenciales de contraseña del propietario del recurso](v2-oauth-ropc.md).

El flujo de ROPC **no se recomienda** porque no es seguro pedir la contraseña de un usuario en la aplicación. Para obtener más información acerca de este problema, consulte [¿Cuál es la solución al creciente problema de las contraseñas?](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/).

Al usar un nombre de usuario y contraseña en un flujo de ROPC, sacrifica varias cosas:

- Principios básicos de identidad moderna: La contraseña se puede buscar y reproducir porque el secreto compartido se puede interceptar. Por definición, ROPC es incompatible con flujos sin contraseña.
- Los usuarios que necesitan MFA no podrán iniciar sesión (ya que no hay ninguna interacción).
- Los usuarios no podrán usar el inicio de sesión único (SSO).

### <a name="configure-the-ropc-flow-in-azure-ad-b2c"></a>Configuración del flujo ROPC en Azure AD B2C

En el inquilino de Azure AD B2C, cree un nuevo flujo de usuario y seleccione **Iniciar sesión mediante ROPC** para habilitar ROPC para el flujo de usuario. Para obtener más información, consulte [Configuración del flujo de credenciales de contraseña de propietario del recurso](../../active-directory-b2c/configure-ropc.md).

`IPublicClientApplication` contiene el método `AcquireTokenByUsernamePassword`:

```csharp
AcquireTokenByUsernamePassword(
            IEnumerable<string> scopes,
            string username,
            SecureString password)
```

El método `AcquireTokenByUsernamePassword` toma los parámetros siguientes:

- Los *ámbitos* para los que se va a obtener un token de acceso.
- Un *nombre de usuario*.
- Una *contraseña* de Securestring para el usuario.

### <a name="limitations-of-the-ropc-flow"></a>Limitaciones del flujo ROPC

El flujo de ROPC **solo funciona para las cuentas locales**, donde los usuarios se han registrado con Azure AD B2C mediante una dirección de correo electrónico o un nombre de usuario. Este flujo no funciona si existe federación con un proveedor de identidades externo que Azure AD B2C admite (Facebook, Google, etc.).

## <a name="google-auth-and-embedded-webview"></a>Autenticación de Google y vista web insertada

Si usa Google como proveedor de identidades, le recomendamos utilizar el explorador del sistema, ya que Google no permite la [autenticación desde vistas web insertadas](https://developers.googleblog.com/2016/08/modernizing-oauth-interactions-in-native-apps.html). Actualmente, `login.microsoftonline.com` es una autoridad de confianza con Google y funcionará con la vista web integrada. Sin embargo, `b2clogin.com` no es una autoridad de confianza para Google, por lo que los usuarios no podrán autenticarse.

Si las cosas cambian, proporcionaremos una actualización de esta [incidencia](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues/688).

## <a name="token-caching-in-msalnet"></a>Almacenamiento en caché de tokens en MSAL.NET

### <a name="known-issue-with-azure-ad-b2c"></a>Problema conocido con Azure AD B2C

MSAL.NET admite una [memoria caché de tokens](/dotnet/api/microsoft.identity.client.tokencache?view=azure-dotnet). La clave de la caché de tokens se basa en las notificaciones que devuelve el proveedor de identidades (IdP).

Actualmente, MSAL.NET necesita dos notificaciones para generar una clave de caché de tokens:

- `tid` (id. de inquilino de Azure AD)
- `preferred_username`

Ambas notificaciones pueden faltar en escenarios de Azure AD B2C porque no todos los proveedores de identidades sociales (Facebook, Google y otros) las devuelven en los tokens que devuelven a Azure AD B2C.

Un síntoma de este escenario es que MSAL.NET devuelve `Missing from the token response` cuando se tiene acceso al valor de notificación de `preferred_username` en los tokens que emite Azure AD B2C. MSAL usa el valor de `Missing from the token response` para `preferred_username` con el fin de mantener la compatibilidad cruzada de la memoria caché entre bibliotecas.

### <a name="workarounds"></a>Soluciones alternativas

#### <a name="mitigation-for-missing-tenant-id"></a>Mitigación para el identificador del inquilino que falta

La solución recomendada es usar la [directiva de caché](#acquire-a-token-to-apply-a-policy) descrita anteriormente.

Como alternativa, puede usar la notificación `tid` si usa [directivas personalizadas](../../active-directory-b2c/custom-policy-get-started.md) en Azure AD B2C. Las directivas personalizadas pueden devolver notificaciones adicionales a la aplicación mediante la [transformación de notificaciones](../../active-directory-b2c/claims-transformation-technical-profile.md).

#### <a name="mitigation-for-missing-from-the-token-response"></a>Mitigación de "Missing from token response" (Falta en la respuesta del token)

Una opción es usar la notificación `name` en lugar de `preferred_username`. Para incluir la notificación `name` en los tokens de identificador que emite Azure AD B2C, seleccione **Nombre para mostrar** al configurar el flujo de usuario.

Para obtener más información sobre cómo especificar qué notificaciones devuelven los flujos de usuario, consulte [Tutorial: Creación de flujos de usuario en Azure AD B2C](../../active-directory-b2c/tutorial-create-user-flows.md).

## <a name="next-steps"></a>Pasos siguientes

En el siguiente ejemplo se ofrecen más detalles sobre la adquisición de tokens de forma interactiva con MSAL.NET para aplicaciones de Azure AD B2C.

| Muestra | Plataforma | Descripción|
|------ | -------- | -----------|
|[active-directory-b2c-xamarin-native](https://github.com/Azure-Samples/active-directory-b2c-xamarin-native) | Xamarin iOS, Xamarin Android, UWP | Una aplicación de Xamarin Forms que usa MSAL.NET para autenticar a los usuarios mediante Azure AD B2C y acceder a una API web con los tokens que se devuelven.|