---
title: B2C de Azure AD (biblioteca de autenticación de Microsoft para. NET) | Azure
description: Obtenga información sobre las consideraciones específicas al usar Azure AD B2C con la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
services: active-directory
documentationcenter: dev-center-name
author: rwike77
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/24/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 6c26a5007c2dcaa5d41be46f685f0f259866ca2c
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544071"
---
# <a name="use-msalnet-to-sign-in-users-with-social-identities"></a>Use MSAL.NET para iniciar sesión los usuarios con identidades de redes sociales

Puede usar para iniciar sesión en los usuarios con identidades de redes sociales mediante MSAL.NET [Azure Active Directory B2C (Azure AD B2C)](https://aka.ms/aadb2c). B2C de Azure AD se basa en la noción de directivas. En MSAL.NET, especificar una directiva se convierte en proporcionar una autoridad.

- Cuando crea una instancia de la aplicación de cliente público, deberá especificar la directiva en la entidad.
- Cuando desea aplicar una directiva, debe llamar a una invalidación de `AcquireTokenInteractive` que contiene un `authority` parámetro.

Esta página es para MSAL 3.x. Si está interesado en MSAL 2.x, consulte [las características de Azure AD B2C en MSAL 2.x](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-Specifics-MSAL-2.x).

## <a name="authority-for-a-azure-ad-b2c-tenant-and-policy"></a>Entidad para un inquilino de Azure AD B2C y la directiva

Es la autoridad que se utilizará `https://login.microsoftonline.com/tfp/{tenant}/{policyName}` donde:

- `tenant` es el nombre del inquilino de Azure AD B2C, 
- `policyName` el nombre de la directiva que se aplicará (por ejemplo "b2c_1_susi" para el inicio de sesión-en sesión y registro).

La orientación actual de Azure AD B2C consiste en usar `b2clogin.com` como la entidad. Por ejemplo, `$"https://{your-tenant-name}.b2clogin.com/tfp/{your-tenant-ID}/{policyname}"`. Para obtener más información, consulte este [documentación](/azure/active-directory-b2c/b2clogin).

## <a name="instantiating-the-application"></a>Crear una instancia de la aplicación

Al compilar la aplicación, deberá proporcionar la autoridad.

```csharp
// Azure AD B2C Coordinates
public static string Tenant = "fabrikamb2c.onmicrosoft.com";
public static string ClientID = "90c0fe63-bcf2-44d5-8fb7-b8bbc0b29dc6";
public static string PolicySignUpSignIn = "b2c_1_susi";
public static string PolicyEditProfile = "b2c_1_edit_profile";
public static string PolicyResetPassword = "b2c_1_reset";

public static string AuthorityBase = $"https://fabrikamb2c.b2clogin.com/tfp/{Tenant}/";
public static string Authority = $"{AuthorityBase}{PolicySignUpSignIn}";
public static string AuthorityEditProfile = $"{AuthorityBase}{PolicyEditProfile}";
public static string AuthorityPasswordReset = $"{AuthorityBase}{PolicyResetPassword}";

application = PublicClientApplicationBuilder.Create(ClientID)
               .WithB2CAuthority(Authority)
               .Build();
```

## <a name="acquire-a-token-to-apply-a-policy"></a>Adquirir un token para aplicar una directiva

Adquirir un token para un B2C de Azure AD API protegida en una aplicación cliente pública requiere que use las invalidaciones con una entidad:

```csharp
IEnumerable<IAccount> accounts = await application.GetAccountsAsync();
AuthenticationResult ar = await application .AcquireToken(scopes, parentWindow)
                                            .WithAccount(GetAccountByPolicy(accounts, policy))
                                            .ExecuteAsync();
```

por:

- `policy` que se va a una de las cadenas anteriores (por ejemplo `PolicySignUpSignIn`).
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

Aplicar una directiva (por ejemplo permitiendo que el usuario final editar su perfil o restablezca su contraseña) se realiza actualmente mediante una llamada a `AcquireTokenInteractive`. En el caso de estas dos directivas no use el token devuelto autenticación result.

## <a name="special-case-of-editprofile-and-resetpassword-policies"></a>Caso especial de EditProfile y ResetPassword directivas

Cuando desea proporcionar una experiencia donde los usuarios finales iniciar sesión con una identidad social y, a continuación, edite su perfil que desea aplicar la directiva de Azure AD B2C EditProfile. La manera de hacer esto, es mediante una llamada a `AcquireTokenInteractive` con la autoridad para esa directiva y un símbolo del sistema establecido en `Prompt.NoPrompt` para evitar el cuadro de diálogo de selección de cuenta que se mostrará (como el usuario ya inició sesión)

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
  . . .
 }
}
```
## <a name="resource-owner-password-credentials-ropc-with-azure-ad-b2c"></a>Credenciales de contraseña de propietario de recursos (ROPC) con Azure AD B2C
Para obtener más detalles sobre el flujo ROPC, consulte este [documentación](v2-oauth-ropc.md).

Este flujo está **no recomienda** dado que la aplicación que pide un usuario de la contraseña no es segura. Para obtener más información acerca de este problema, consulte [en este artículo](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/). 

Mediante el uso de nombre de usuario/contraseña, se están dando vertical varias cosas:
- principales inquilinos de identidad moderna: contraseña obtiene pesca, reproducir. Dado que tenemos este concepto de un secreto compartido que se puede interceptar. Esto es incompatible con sin contraseña.
- Los usuarios que necesitan para realizar la MFA no podrán iniciar sesión (ya que no hay ninguna interacción).
- Los usuarios no podrán inicio de sesión único.

### <a name="configure-the-ropc-flow-in-azure-ad-b2c"></a>Configuración del flujo ROPC en Azure AD B2C
En el inquilino de Azure AD B2C, cree un nuevo flujo de usuario y seleccione **inicie sesión con ROPC**. Esto permitirá que la directiva ROPC para el inquilino. Consulte [configurar las credenciales de contraseña de propietario de recursos de flujo](/azure/active-directory-b2c/configure-ropc) para obtener más detalles.

`IPublicClientApplication` contiene un método:
```csharp
AcquireTokenByUsernamePassword(
            IEnumerable<string> scopes,
            string username,
            SecureString password)
```

Este método toma como parámetros:
- El *ámbitos* para solicitar un token de acceso.
- Un *username*.
- SecureString *contraseña* para el usuario.

Recuerde que debe usar la entidad que contiene la directiva ROPC.

### <a name="limitations-of-the-ropc-flow"></a>Limitaciones del flujo ROPC
 - El flujo ROPC **solo funciona para las cuentas locales** (donde se registra con Azure AD B2C mediante un correo electrónico o nombre de usuario). Este flujo no funciona si realizando una federación a cualquiera de los proveedores de identidad admitidos por Azure AD B2C (Facebook, Google, etcetera.).
 - Actualmente, no hay **no devuelve ningún valor de id_token de Azure AD B2C** al implementar el flujo ROPC de MSAL. Esto significa que no se puede crear un objeto de cuenta, por lo que en la memoria caché, habrá ninguna cuenta y ningún usuario. El flujo de AcquireTokenSilent no funcionará en este escenario. Sin embargo, ROPC no muestra una interfaz de usuario, por lo que no habrá ningún impacto en la experiencia del usuario.

## <a name="google-auth-and-embedded-webview"></a>Autenticación de Google y Embedded Webview

Si es un desarrollador de Azure AD B2C usa Google como proveedor de identidades se recommand utilizar el explorador del sistema, como Google no permite [autenticación desde insertadas](https://developers.googleblog.com/2016/08/modernizing-oauth-interactions-in-native-apps.html). Actualmente, `login.microsoftonline.com` es una autoridad de confianza con Google. Uso de esta entidad funcionará con webview incrustado. Sin embargo con `b2clogin.com` no es una autoridad de confianza con Google, por lo que los usuarios no podrán autenticarse.

Proporcionamos una actualización para el sitio wiki y esto [problema](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues/688) si las cosas cambian.

## <a name="caching-with-azure-ad-b2c-in-msalnet"></a>Almacenamiento en caché con B2C de Azure AD en MSAL.Net 

### <a name="known-issue-with-azure-ad-b2c"></a>Problema conocido con Azure AD B2C

MSAL.Net admite un [memoria caché de tokens](/dotnet/api/microsoft.identity.client.tokencache?view=azure-dotnet). El token de clave de almacenamiento en caché se basa en las notificaciones devueltas por el proveedor de identidades. Actualmente MSAL.Net necesita dos notificaciones para generar una clave de caché de tokens:  
- `tid` que es el identificador de inquilino de Azure AD, y 
- `preferred_username` 

Faltan ambos estas notificaciones en muchos de los escenarios de Azure AD B2C. 

¿El impacto en el cliente es que al intentar mostrar el campo de nombre de usuario, están obteniendo "Falta en la respuesta del token" como el valor? Si es así, esto es porque Azure AD B2C no devuelve un valor en el IdToken para el preferred_username debido a limitaciones con las cuentas sociales y los proveedores de identidades externo (IDP). Azure AD devuelve un valor para preferred_username porque sabe quién es el usuario, pero para Azure AD B2C, dado que el usuario puede iniciar sesión con una cuenta local, Facebook, Google, GitHub, etc. allí no es un valor constante para Azure AD B2C que se usará para preferred_username. Para desbloquear MSAL de implementación de compatibilidad de la memoria caché con ADAL, decidimos usar "Falta en la respuesta del token" en nuestro extremo cuando se trabaja con las cuentas de Azure AD B2C cuando el IdToken devuelve nothing para preferred_username. MSAL debe devolver un valor para preferred_username mantener la compatibilidad de la memoria caché a través de bibliotecas.

### <a name="workarounds"></a>Soluciones alternativas

#### <a name="mitigation-for-the-missing-tenant-id"></a>Mitigación para el identificador del inquilino que faltan

La solución recomendada es usar el [por la directiva de almacenamiento en caché](#acquire-a-token-to-apply-a-policy)

Como alternativa, puede usar el `tid` de notificación, si usas el [directivas personalizadas B2C](https://aka.ms/ief), ya que proporciona la capacidad de devolver notificaciones adicionales a la aplicación. Para obtener más información acerca de [transformación de notificaciones](/azure/active-directory-b2c/claims-transformation-technical-profile)

#### <a name="mitigation-for-missing-from-the-token-response"></a>Mitigación de "Falta en la respuesta del token"
Una opción es usar la notificación "name" como el nombre de usuario preferido. El proceso se menciona en este [B2C doc](/azure/active-directory-b2c/active-directory-b2c-reference-policies#frequently-asked-questions) -> "en la columna de valor devuelto de notificación, elija las notificaciones que quiere que se devuelvan en los tokens de autorización enviados de vuelta a su aplicación después de una experiencia de edición de perfiles correcta. Por ejemplo, seleccione nombre para mostrar, código Postal".

## <a name="next-steps"></a>Pasos siguientes 

En el ejemplo siguiente, se proporcionan más detalles sobre la adquisición de tokens de forma interactiva con MSAL.NET para aplicaciones de Azure AD B2C.

| Muestra | Plataforma | DESCRIPCIÓN|
|------ | -------- | -----------|
|[active-directory-b2c-xamarin-native](https://github.com/Azure-Samples/active-directory-b2c-xamarin-native) | Xamarin iOS, Xamarin Android, UWP | Una aplicación de Xamarin Forms sencilla que muestra cómo usar MSAL.NET para autenticar a los usuarios a través de Azure AD B2C y acceder a una API Web con los tokens resultantes.|
