---
title: Adquisición de un token para llamar a una API web (aplicación de escritorio) | Azure
titleSuffix: Microsoft identity platform
description: Aprenda a compilar una aplicación de escritorio que llame a API web para adquirir un token para la aplicación.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/18/2020
ms.author: jmprieur
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: 300bc6acbe7821841b578dcc2166ecfc498ad750
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88141302"
---
# <a name="desktop-app-that-calls-web-apis-acquire-a-token"></a>Aplicación de escritorio que llama a API web: Adquisición de un token

Una vez que se ha compilado una instancia de la aplicación cliente pública, se usará para adquirir un token que, a su vez, se emplea para llamar a una API web.

## <a name="recommended-pattern"></a>Patrón recomendado

La API web se define mediante sus `scopes`. Sea cual sea la experiencia que proporcione en la aplicación, el patrón que debe usar es:

- Intente obtener de forma sistemática un token de la caché de tokens mediante una llamada a `AcquireTokenSilent`.
- Si se produce un error en esta llamada, use el flujo `AcquireToken` que quiera emplear, que aquí está representado por `AcquireTokenXX`.

# <a name="net"></a>[.NET](#tab/dotnet)

### <a name="in-msalnet"></a>En MSAL.NET

```csharp
AuthenticationResult result;
var accounts = await app.GetAccountsAsync();
IAccount account = ChooseAccount(accounts); // for instance accounts.FirstOrDefault
                                            // if the app manages is at most one account  
try
{
 result = await app.AcquireTokenSilent(scopes, account)
                   .ExecuteAsync();
}
catch(MsalUiRequiredException ex)
{
  result = await app.AcquireTokenXX(scopes, account)
                    .WithOptionalParameterXXX(parameter)
                    .ExecuteAsync();
}
```

# <a name="java"></a>[Java](#tab/java)

```java

Set<IAccount> accountsInCache = pca.getAccounts().join();
// Take first account in the cache. In a production application, you would filter
// accountsInCache to get the right account for the user authenticating.
IAccount account = accountsInCache.iterator().next();

IAuthenticationResult result;
try {
    SilentParameters silentParameters =
            SilentParameters
                    .builder(SCOPE, account)
                    .build();

    // try to acquire token silently. This call will fail since the token cache
    // does not have any data for the user you are trying to acquire a token for
    result = pca.acquireTokenSilently(silentParameters).join();
} catch (Exception ex) {
    if (ex.getCause() instanceof MsalException) {

        InteractiveRequestParameters parameters = InteractiveRequestParameters
                .builder(new URI("http://localhost"))
                .scopes(SCOPE)
                .build();

        // Try to acquire a token interactively with system browser. If successful, you should see
        // the token and account information printed out to console
        result = pca.acquireToken(parameters).join();
    } else {
        // Handle other exceptions accordingly
        throw ex;
    }
}
return result;

```

# <a name="python"></a>[Python](#tab/python)

```Python
result = None

# Firstly, check the cache to see if this end user has signed in before
accounts = app.get_accounts(username=config["username"])
if accounts:
    result = app.acquire_token_silent(config["scope"], account=accounts[0])

if not result:
    result = app.acquire_token_by_xxx(scopes=config["scope"])
```

# <a name="macos"></a>[macOS](#tab/macOS)

### <a name="in-msal-for-ios-and-macos"></a>En MSAL para iOS y macOS

Objective-C:

```objc
MSALAccount *account = [application accountForIdentifier:accountIdentifier error:nil];

MSALSilentTokenParameters *silentParams = [[MSALSilentTokenParameters alloc] initWithScopes:scopes account:account];
[application acquireTokenSilentWithParameters:silentParams completionBlock:^(MSALResult *result, NSError *error) {

    // Check the error
    if (error && [error.domain isEqual:MSALErrorDomain] && error.code == MSALErrorInteractionRequired)
    {
        // Interactive auth will be required, call acquireTokenWithParameters:error:
    }
}];
```
Swift:

```swift
guard let account = try? application.account(forIdentifier: accountIdentifier) else { return }
let silentParameters = MSALSilentTokenParameters(scopes: scopes, account: account)
application.acquireTokenSilent(with: silentParameters) { (result, error) in

    guard let authResult = result, error == nil else {

    let nsError = error! as NSError

        if (nsError.domain == MSALErrorDomain &&
            nsError.code == MSALError.interactionRequired.rawValue) {

            // Interactive auth will be required, call acquireToken()
            return
        }
        return
    }
}
```
---

Estas son las distintas formas de adquirir tokens en una aplicación de escritorio.

## <a name="acquire-a-token-interactively"></a>Adquisición de un token de forma interactiva

El ejemplo siguiente muestra un código mínimo para obtener un token de forma interactiva para leer el perfil del usuario con Microsoft Graph.

# <a name="net"></a>[.NET](#tab/dotnet)
### <a name="in-msalnet"></a>En MSAL.NET

```csharp
string[] scopes = new string[] {"user.read"};
var app = PublicClientApplicationBuilder.Create(clientId).Build();
var accounts = await app.GetAccountsAsync();
AuthenticationResult result;
try
{
 result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
             .ExecuteAsync();
}
catch(MsalUiRequiredException)
{
 result = await app.AcquireTokenInteractive(scopes)
             .ExecuteAsync();
}
```

### <a name="mandatory-parameters"></a>Parámetros obligatorios

`AcquireTokenInteractive` solo tiene un parámetro obligatorio, ``scopes``, que contiene una enumeración de cadenas que definen los ámbitos en los que se necesita un token. Si el token es para Microsoft Graph, los ámbitos necesarios se pueden encontrar en la sección denominada "Permisos" de la referencia de API de cada instancia de Microsoft Graph API. Por ejemplo, para [enumerar los contactos del usuario](/graph/api/user-list-contacts), se debe usar el ámbito "User.Read", "Contacts.Read". Para más información, consulte [Referencia de permisos de Microsoft Graph](/graph/permissions-reference).

En Android, también debe especificar la actividad principal mediante `.WithParentActivityOrWindow`, como se muestra, para que el token vuelva a esa actividad principal tras la interacción. Si no se especifica, se produce una excepción al llamar a `.ExecuteAsync()`.

### <a name="specific-optional-parameters-in-msalnet"></a>Parámetros opcionales específicos de MSAL.NET

#### <a name="withparentactivityorwindow"></a>WithParentActivityOrWindow

La interfaz de usuario es importante porque es interactiva. `AcquireTokenInteractive` tiene un parámetro opcional específico que permite determinar, en las plataformas que lo admiten, la interfaz de usuario principal. Cuando se usa en una aplicación de escritorio, `.WithParentActivityOrWindow` tiene un tipo diferente que depende de la plataforma.

```csharp
// net45
WithParentActivityOrWindow(IntPtr windowPtr)
WithParentActivityOrWindow(IWin32Window window)

// Mac
WithParentActivityOrWindow(NSWindow window)

// .Net Standard (this will be on all platforms at runtime, but only on NetStandard at build time)
WithParentActivityOrWindow(object parent).
```

Comentarios:

- En .NET Standard, el elemento `object` esperado es `Activity` en Android, `UIViewController` en iOS, `NSWindow` en MAC y `IWin32Window` o `IntPr` en Windows.
- En Windows, debe llamar a `AcquireTokenInteractive` desde el subproceso de interfaz de usuario para que el explorador integrado obtenga el contexto de sincronización de la interfaz de usuario adecuado. Si no se llama desde el subproceso de interfaz de usuario, es posible que los mensajes no se suministren correctamente o que se produzcan escenarios de interbloqueo con la interfaz de usuario. Una manera de llamar a bibliotecas de autenticación de Microsoft (MSAL) desde el subproceso de interfaz de usuario (si aún no está en él) es usar `Dispatcher` en WPF.
- Si usa WPF, para obtener una ventana de un control WPF puede usar la clase `WindowInteropHelper.Handle`. Así, la llamada procede de un control WPF (`this`):

  ```csharp
  result = await app.AcquireTokenInteractive(scopes)
                    .WithParentActivityOrWindow(new WindowInteropHelper(this).Handle)
                    .ExecuteAsync();
  ```

#### <a name="withprompt"></a>WithPrompt

`WithPrompt()` se usa para controlar la interactividad con el usuario mediante la especificación de un símbolo del sistema.

![Imagen que muestra los campos de la estructura del mensaje. Estos valores constantes controlan la interactividad con el usuario al definir el tipo de mensaje que muestra el método WithPrompt().](https://user-images.githubusercontent.com/13203188/53438042-3fb85700-39ff-11e9-9a9e-1ff9874197b3.png)

La clase define las constantes siguientes:

- ``SelectAccount`` obliga a STS a presentar el cuadro de diálogo de selección de cuenta que contiene las cuentas para las que el usuario tiene una sesión. Esta opción es útil cuando los desarrolladores de aplicaciones quieren permitir a los usuarios elegir entre diferentes identidades. Esta opción dirige a MSAL para que envíe ``prompt=select_account`` al proveedor de identidades. Es la opción predeterminada. Funciona bien para proporcionar la mejor experiencia posible en función de la información disponible, como cuenta y presencia de una sesión para el usuario. No la cambie a menos que tenga buena razón para hacerlo.
- ``Consent`` permite que el desarrollador de la aplicación obligue al usuario a pedir consentimiento, aunque ya lo haya otorgado antes. En este caso, MSAL envía `prompt=consent` al proveedor de identidades. Esta opción se puede usar en algunas aplicaciones centradas en la seguridad donde la gobernanza de la organización exige que se presente al usuario el cuadro de diálogo de consentimiento cada vez que se usa la aplicación.
- ``ForceLogin`` permite que el desarrollador de la aplicación obligue al servicio a solicitar las credenciales al usuario, incluso si este mensaje de usuario no es necesario. Esta opción puede ser útil para permitir que el usuario vuelva a iniciar sesión si se produce un error al adquirir un token. En este caso, MSAL envía `prompt=login` al proveedor de identidades. A veces se usa en aplicaciones centradas en la seguridad donde la gobernanza de la organización exige que el usuario vuelva a iniciar sesión cada vez que accede a determinadas partes de una aplicación.
- ``Never`` (solo para .NET 4.5 y WinRT) no pregunta al usuario, sino que, en su lugar, intenta usar la cookie almacenada en la vista web insertada oculta. Para más información, consulte las vistas web de MSAL.NET. Se puede producir un error al usar esta opción. En ese caso, `AcquireTokenInteractive` produce una excepción para notificar que se necesita una interacción de la interfaz de usuario. Debe usar otro parámetro `Prompt`.
- ``NoPrompt`` no envía ningún símbolo del sistema al proveedor de identidades. Esta opción solo es útil para directivas de perfil de edición de Azure Active Directory (Azure AD) B2C. Para más información, consulte [Información específica sobre Azure AD B2C](https://aka.ms/msal-net-b2c-specificities).

#### <a name="withextrascopetoconsent"></a>WithExtraScopeToConsent

Este modificador se usa en un escenario avanzado en el que se quiere que el usuario otorgue su consentimiento por adelantado a varios recursos y no se quiere emplear el consentimiento incremental, que normalmente se usa con MSAL.NET o la plataforma de identidad de Microsoft. Para más información, consulte [Cómo tener el consentimiento del usuario por adelantado para varios recursos](scenario-desktop-production.md#have-the-user-consent-upfront-for-several-resources).

```csharp
var result = await app.AcquireTokenInteractive(scopesForCustomerApi)
                     .WithExtraScopeToConsent(scopesForVendorApi)
                     .ExecuteAsync();
```

#### <a name="withcustomwebui"></a>WithCustomWebUi

Una interfaz de usuario web es un mecanismo para invocar un explorador. Este mecanismo puede ser un control WebBrowser de interfaz de usuario dedicado o una forma de delegar la apertura del explorador.
MSAL proporciona implementaciones de la interfaz de usuario web para la mayoría de las plataformas, aunque hay casos en los que es posible que quiera hospedar el explorador por su cuenta:

- Plataformas no admitidas explícitamente por MSAL, por ejemplo, Blazor, Unity o Mono en equipos de escritorio.
- Quiere probar la aplicación en la interfaz de usuario y usar un explorador automatizado que se pueda emplear con Selenium.
- El explorador y la aplicación que ejecutan MSAL están en procesos independientes.

##### <a name="at-a-glance"></a>En un vistazo

Para ello, debe proporcionar a MSAL el elemento `start Url`, que debe mostrarse en el explorador seleccionado para que el usuario final pueda escribir elementos como su nombre de usuario.
Una vez terminada la autenticación, la aplicación debe pasar a MSAL el elemento `end Url`, que contiene un código proporcionado por Azure AD.
El host de `end Url` siempre es `redirectUri`. Para interceptar `end Url`, realice una de las siguientes acciones:

- Supervise los redireccionamientos del explorador hasta que se alcance `redirect Url`.
- Haga que el explorador redirija a una dirección URL que supervise.

##### <a name="withcustomwebui-is-an-extensibility-point"></a>WithCustomWebUi es un punto de extensibilidad.

`WithCustomWebUi` es un punto de extensibilidad que se puede usar para proporcionar una interfaz de usuario propia en aplicaciones cliente públicas. También puede permitir que el usuario pase por el punto de conexión /Authorize del proveedor de identidades y dejarle iniciar sesión y dar su consentimiento. Después, MSAL.NET puede canjear el código de autenticación y obtener un token. Por ejemplo, se usa en Visual Studio para que las aplicaciones de Electron (por ejemplo, Visual Studio Feedback) proporcionen la interacción web, pero se deja que MSAL.NET realice la mayor parte del trabajo. También puede usarlo si quiere proporcionar automatización de la interfaz de usuario. En las aplicaciones cliente públicas, MSAL.NET usa la norma Proof Key for Code Exchange (PKCE) para garantizar que se respeta la seguridad. Solo MSAL.NET puede canjear el código. Para más información, consulte [RFC 7636 - Proof Key for Code Exchange por parte de clientes públicos de OAuth](https://tools.ietf.org/html/rfc7636).

  ```csharp
  using Microsoft.Identity.Client.Extensions;
  ```

##### <a name="use-withcustomwebui"></a>Uso de WithCustomWebUi

Para usar `.WithCustomWebUI`, siga estos pasos.

  1. Implemente la interfaz `ICustomWebUi`. Para más información, consulte [este sitio web](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/blob/053a98d16596be7e9ca1ab916924e5736e341fe8/src/Microsoft.Identity.Client/Extensibility/ICustomWebUI.cs#L32-L70). Implemente un método `AcquireAuthorizationCodeAsync` y acepte la dirección URL de código de autorización procesada por MSAL.NET. Luego, permita que el usuario lleve a cabo la interacción con el proveedor de identidades y devuelva la dirección URL con la que este hubiera llamado a la implementación junto con el código de autorización. Si tiene problemas, la implementación debería lanzar una excepción `MsalExtensionException` para cooperar bien con MSAL.
  2. En la llamada a `AcquireTokenInteractive`, use el modificador `.WithCustomUI()` que pasa la instancia de la interfaz de usuario web personalizada.

     ```csharp
     result = await app.AcquireTokenInteractive(scopes)
                       .WithCustomWebUi(yourCustomWebUI)
                       .ExecuteAsync();
     ```

##### <a name="examples-of-implementation-of-icustomwebui-in-test-automation-seleniumwebui"></a>Ejemplos de implementación de ICustomWebUi en automatización de prueba: SeleniumWebUI

El equipo de MSAL.NET ha vuelto a escribir las pruebas de interfaz de usuario para usar este mecanismo de extensibilidad. Si le interesa, vea la clase [SeleniumWebUI](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/blob/053a98d16596be7e9ca1ab916924e5736e341fe8/tests/Microsoft.Identity.Test.Integration/Infrastructure/SeleniumWebUI.cs#L15-L160) en el código fuente de MSAL.NET.

##### <a name="provide-a-great-experience-with-systemwebviewoptions"></a>Una gran experiencia con SystemWebViewOptions

En [`SystemWebViewOptions`](/dotnet/api/microsoft.identity.client.systemwebviewoptions?view=azure-dotnet) de MSAL.NET 4.1, puede especificar:

- El URI al que se va a ir (`BrowserRedirectError`) o el fragmento HTML que se va a mostrar (`HtmlMessageError`) en caso de errores de inicio de sesión o consentimiento en el explorador web del sistema.
- El URI al que se va a ir (`BrowserRedirectSuccess`) o el fragmento HTML que se va a mostrar (`HtmlMessageSuccess`) en caso de inicio de sesión o consentimiento correctos.
- La acción que se va a ejecutar para iniciar el explorador del sistema. Puede proporcionar una implementación propia si establece el delegado `OpenBrowserAsync`. La clase también proporciona una implementación predeterminada para dos exploradores: `OpenWithEdgeBrowserAsync` y `OpenWithChromeEdgeBrowserAsync` para Microsoft Edge y [Microsoft Edge en Chromium](https://www.windowscentral.com/faq-edge-chromium), respectivamente.

Para usar esta estructura, escriba algo como esto:

```csharp
IPublicClientApplication app;
...

options = new SystemWebViewOptions
{
 HtmlMessageError = "<b>Sign-in failed. You can close this tab ...</b>",
 BrowserRedirectSuccess = "https://contoso.com/help-for-my-awesome-commandline-tool.html"
};

var result = app.AcquireTokenInteractive(scopes)
                .WithEmbeddedWebView(false)       // The default in .NET Core
                .WithSystemWebViewOptions(options)
                .Build();
```

#### <a name="other-optional-parameters"></a>Otros parámetros opcionales

Para más información sobre los demás parámetros opcionales de `AcquireTokenInteractive`, consulte [AcquireTokenInteractiveParameterBuilder](/dotnet/api/microsoft.identity.client.acquiretokeninteractiveparameterbuilder?view=azure-dotnet-preview#methods).

# <a name="java"></a>[Java](#tab/java)

```java
private static IAuthenticationResult acquireTokenInteractive() throws Exception {

    // Load token cache from file and initialize token cache aspect. The token cache will have
    // dummy data, so the acquireTokenSilently call will fail.
    TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

    PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
            .authority(AUTHORITY)
            .setTokenCacheAccessAspect(tokenCacheAspect)
            .build();

    Set<IAccount> accountsInCache = pca.getAccounts().join();
    // Take first account in the cache. In a production application, you would filter
    // accountsInCache to get the right account for the user authenticating.
    IAccount account = accountsInCache.iterator().next();

    IAuthenticationResult result;
    try {
        SilentParameters silentParameters =
                SilentParameters
                        .builder(SCOPE, account)
                        .build();

        // try to acquire token silently. This call will fail since the token cache
        // does not have any data for the user you are trying to acquire a token for
        result = pca.acquireTokenSilently(silentParameters).join();
    } catch (Exception ex) {
        if (ex.getCause() instanceof MsalException) {

            InteractiveRequestParameters parameters = InteractiveRequestParameters
                    .builder(new URI("http://localhost"))
                    .scopes(SCOPE)
                    .build();

            // Try to acquire a token interactively with system browser. If successful, you should see
            // the token and account information printed out to console
            result = pca.acquireToken(parameters).join();
        } else {
            // Handle other exceptions accordingly
            throw ex;
        }
    }
    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

Python de MSAL no proporciona ningún método de adquisición de tokens interactivo directamente, sino que exige que la aplicación envíe una solicitud de autorización en su implementación del flujo de interacción del usuario a fin de obtener un código de autorización. Luego, este código se puede pasar al método `acquire_token_by_authorization_code` a fin de obtener el token.

```Python
result = None

# Firstly, check the cache to see if this end user has signed in before
accounts = app.get_accounts(username=config["username"])
if accounts:
    result = app.acquire_token_silent(config["scope"], account=accounts[0])

if not result:
    result = app.acquire_token_by_authorization_code(
         request.args['code'],
         scopes=config["scope"])    

```

# <a name="macos"></a>[macOS](#tab/macOS)

### <a name="in-msal-for-ios-and-macos"></a>En MSAL para iOS y macOS

Objective-C:

```objc
MSALInteractiveTokenParameters *interactiveParams = [[MSALInteractiveTokenParameters alloc] initWithScopes:scopes webviewParameters:[MSALWebviewParameters new]];
[application acquireTokenWithParameters:interactiveParams completionBlock:^(MSALResult *result, NSError *error) {
    if (!error)
    {
        // You'll want to get the account identifier to retrieve and reuse the account
        // for later acquireToken calls
        NSString *accountIdentifier = result.account.identifier;

        NSString *accessToken = result.accessToken;
    }
}];
```

Swift:

```swift
let interactiveParameters = MSALInteractiveTokenParameters(scopes: scopes, webviewParameters: MSALWebviewParameters())
application.acquireToken(with: interactiveParameters, completionBlock: { (result, error) in

    guard let authResult = result, error == nil else {
        print(error!.localizedDescription)
        return
    }

    // Get access token from result
    let accessToken = authResult.accessToken
})
```
---

## <a name="integrated-windows-authentication"></a>Autenticación integrada de Windows

Para iniciar la sesión de un usuario de dominio en un equipo unido a un dominio o a Azure AD, use la autenticación integrada de Windows (IWA).

### <a name="constraints"></a>Restricciones

- La autenticación integrada de Windows solo es útil para usuarios *federados*, es decir, los usuarios creados en Active Directory y respaldados por Azure AD. Los usuarios creados directamente en Azure AD sin el respaldo de Active Directory, conocidos como usuarios *administrados*, no pueden usar este flujo de autenticación. Esta limitación no afecta al flujo de nombre de usuario y contraseña.
- IWA es para aplicaciones escritas para las plataformas .NET Framework, .NET Core y Plataforma universal de Windows (UPW).
- IWA no omite [la autenticación multifactor (MFA)](../authentication/concept-mfa-howitworks.md). Si se ha configurado MFA, IWA podría producir un error si es necesario un desafío de MFA, porque MFA necesita interacción del usuario.
  > [!NOTE]
  > Esto es complicado. IWA no es interactiva, pero MFA requiere interactividad del usuario. El usuario no es quien controla si el proveedor de identidades solicita que se realice MFA, sino el administrador de inquilinos. Por lo que se ha observado, MFA se requiere al iniciar sesión desde otro país o región, cuando no se está conectado a través de VPN a una red corporativa y, a veces, incluso cuando se está. No espere un conjunto determinista de reglas. Azure AD usa inteligencia artificial para aprender continuamente si se requiere MFA. Si se produce un error de IWA, recurra a un mensaje de usuario como la autenticación interactiva o el flujo de código de dispositivo.

- La autoridad pasada en `PublicClientApplicationBuilder` debe ser:
  - Con inquilino con formato `https://login.microsoftonline.com/{tenant}/`, donde `tenant` es el GUID que representa el identificador de inquilino o un dominio asociado al inquilino.
  - Para cualquier cuenta profesional y educativa: `https://login.microsoftonline.com/organizations/`.
  - No se admiten cuentas personales de Microsoft. No se pueden usar inquilinos /common ni /consumers.

- Debido a que la autenticación integrada de Windows es un flujo silencioso:
  - El usuario de la aplicación debe haber consentido anteriormente para usar la aplicación.
  - O bien, el administrador de inquilinos debe haber consentido anteriormente que todos los usuarios del inquilino usen la aplicación.
  - En otras palabras:
    - Usted como desarrollador ha hecho clic en el botón **Conceder** de Azure Portal.
    - O bien, un administrador de inquilinos ha hecho clic en el botón **Conceder o revocar consentimiento del administrador para {dominio del inquilino}** de la pestaña **Permisos de API** del registro de la aplicación. Para más información, consulte [Adición de permisos para acceder a las API web](./quickstart-configure-app-access-web-apis.md#add-permissions-to-access-web-apis).
    - O bien, ha proporcionado una manera para que los usuarios den su consentimiento a la aplicación. Para más información, consulte [Solicitud de consentimiento de usuario individual](./v2-permissions-and-consent.md#requesting-individual-user-consent).
    - O bien, ha proporcionado una forma para que el administrador de inquilinos otorgue su consentimiento a la aplicación. Para más información, consulte [Consentimiento del administrador](./v2-permissions-and-consent.md#requesting-consent-for-an-entire-tenant).

- Este flujo está habilitado para aplicaciones de escritorio de .NET, .NET Core y UWP.

Para más información sobre el consentimiento, consulte [Permisos y consentimiento en el punto de conexión de la Plataforma de identidad de Microsoft](./v2-permissions-and-consent.md).

### <a name="learn-how-to-use-it"></a>Aprenda a usarlo

# <a name="net"></a>[.NET](#tab/dotnet)

En MSAL.NET, debe usar:

```csharp
AcquireTokenByIntegratedWindowsAuth(IEnumerable<string> scopes)
```

Normalmente solo necesita un parámetro (`scopes`). Según la forma en que el administrador de Windows configure las directivas, es posible que no se permita que las aplicaciones de la máquina Windows busquen al usuario que ha iniciado sesión. En ese caso, use un segundo método, `.WithUsername()`, y pase el nombre de usuario del usuario que ha iniciado sesión con un formato UPN, por ejemplo, `joe@contoso.com`. En .NET Core, solo está disponible la sobrecarga que toma el nombre de usuario, ya que la plataforma .NET Core no puede pedir el nombre de usuario al sistema operativo.

El ejemplo siguiente presenta el caso más reciente, con explicaciones de la clase de excepciones que se pueden obtener y sus mitigaciones.

```csharp
static async Task GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication app = PublicClientApplicationBuilder
      .Create(clientId)
      .WithAuthority(authority)
      .Build();

 var accounts = await app.GetAccountsAsync();

 AuthenticationResult result = null;
 if (accounts.Any())
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
      .ExecuteAsync();
 }
 else
 {
  try
  {
   result = await app.AcquireTokenByIntegratedWindowsAuth(scopes)
      .ExecuteAsync(CancellationToken.None);
  }
  catch (MsalUiRequiredException ex)
  {
   // MsalUiRequiredException: AADSTS65001: The user or administrator has not consented to use the application
   // with ID '{appId}' named '{appName}'.Send an interactive authorization request for this user and resource.

   // you need to get user consent first. This can be done, if you are not using .NET Core (which does not have any Web UI)
   // by doing (once only) an AcquireToken interactive.

   // If you are using .NET core or don't want to do an AcquireTokenInteractive, you might want to suggest the user to navigate
   // to a URL to consent: https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id={clientId}&response_type=code&scope=user.read

   // AADSTS50079: The user is required to use multi-factor authentication.
   // There is no mitigation - if MFA is configured for your tenant and AAD decides to enforce it,
   // you need to fallback to an interactive flows such as AcquireTokenInteractive or AcquireTokenByDeviceCode
   }
   catch (MsalServiceException ex)
   {
    // Kind of errors you could have (in ex.Message)

    // MsalServiceException: AADSTS90010: The grant type is not supported over the /common or /consumers endpoints. Please use the /organizations or tenant-specific endpoint.
    // you used common.
    // Mitigation: as explained in the message from Azure AD, the authority needs to be tenanted or otherwise organizations

    // MsalServiceException: AADSTS70002: The request body must contain the following parameter: 'client_secret or client_assertion'.
    // Explanation: this can happen if your application was not registered as a public client application in Azure AD
    // Mitigation: in the Azure portal, edit the manifest for your application and set the `allowPublicClient` to `true`
   }
   catch (MsalClientException ex)
   {
      // Error Code: unknown_user Message: Could not identify logged in user
      // Explanation: the library was unable to query the current Windows logged-in user or this user is not AD or AAD
      // joined (work-place joined users are not supported).

      // Mitigation 1: on UWP, check that the application has the following capabilities: Enterprise Authentication,
      // Private Networks (Client and Server), User Account Information

      // Mitigation 2: Implement your own logic to fetch the username (e.g. john@contoso.com) and use the
      // AcquireTokenByIntegratedWindowsAuth form that takes in the username

      // Error Code: integrated_windows_auth_not_supported_managed_user
      // Explanation: This method relies on an a protocol exposed by Active Directory (AD). If a user was created in Azure
      // Active Directory without AD backing ("managed" user), this method will fail. Users created in AD and backed by
      // AAD ("federated" users) can benefit from this non-interactive method of authentication.
      // Mitigation: Use interactive authentication
   }
 }


 Console.WriteLine(result.Account.Username);
}
```

Para obtener la lista de posibles modificadores en AcquireTokenByIntegratedWindowsAuthentication, consulte [AcquireTokenByIntegratedWindowsAuthParameterBuilder](/dotnet/api/microsoft.identity.client.acquiretokenbyintegratedwindowsauthparameterbuilder?view=azure-dotnet-preview#methods).

# <a name="java"></a>[Java](#tab/java)

Este extracto procede de los [ejemplos de desarrollo de Java de MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/public-client/).

```Java
private static IAuthenticationResult acquireTokenIwa() throws Exception {

    // Load token cache from file and initialize token cache aspect. The token cache will have
    // dummy data, so the acquireTokenSilently call will fail.
    TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

    PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
            .authority(AUTHORITY)
            .setTokenCacheAccessAspect(tokenCacheAspect)
            .build();

    Set<IAccount> accountsInCache = pca.getAccounts().join();
    // Take first account in the cache. In a production application, you would filter
    // accountsInCache to get the right account for the user authenticating.
    IAccount account = accountsInCache.iterator().next();

    IAuthenticationResult result;
    try {
        SilentParameters silentParameters =
                SilentParameters
                        .builder(SCOPE, account)
                        .build();

        // try to acquire token silently. This call will fail since the token cache
        // does not have any data for the user you are trying to acquire a token for
        result = pca.acquireTokenSilently(silentParameters).join();
    } catch (Exception ex) {
        if (ex.getCause() instanceof MsalException) {

            IntegratedWindowsAuthenticationParameters parameters =
                    IntegratedWindowsAuthenticationParameters
                            .builder(SCOPE, USER_NAME)
                            .build();

            // Try to acquire a IWA. You will need to generate a Kerberos ticket.
            // If successful, you should see the token and account information printed out to
            // console
            result = pca.acquireToken(parameters).join();
        } else {
            // Handle other exceptions accordingly
            throw ex;
        }
    }
    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

Este flujo aún no se admite en Python de MSAL.

# <a name="macos"></a>[macOS](#tab/macOS)

Este flujo no se aplica a macOS.

---

## <a name="username-and-password"></a>Nombre de usuario y contraseña

También puede adquirir un token proporcionando el nombre de usuario y la contraseña. Este flujo es limitado y no se recomienda, pero hay casos de uso donde es necesario.

### <a name="this-flow-isnt-recommended"></a>Este flujo no es recomendable.

*No se recomienda* este flujo porque no es seguro que la aplicación le pida a un usuario la contraseña. Para más información, consulte [What's the solution to the growing problem of passwords?](https://news.microsoft.com/features/whats-solution-growing-problem-passwords-says-microsoft/) (¿Cuál es la solución al creciente problema de las contraseñas?). El flujo preferido para adquirir un token de forma silenciosa en equipos unidos a un dominio de Windows es la [autenticación integrada de Windows](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Integrated-Windows-Authentication). También puede usar [flujo de código de dispositivo](https://aka.ms/msal-net-device-code-flow).

> [!NOTE]
> El empleo de un nombre de usuario y una contraseña resulta útil en algunos casos, como los escenarios de DevOps. Pero si quiere usar un nombre de usuario y una contraseña en escenarios interactivos en los que proporciona su propia interfaz de usuario, piense en cómo dejar de hacerlo. Al usar un nombre de usuario y una contraseña, está renunciando a una serie de cosas:
>
> - Principios básicos de identidad moderna. Una contraseña puede ser objeto de suplantación de identidad (phishing) y reproducción porque un secreto compartido se puede interceptar. No es compatible con el inicio de sesión sin contraseña.
> - Los usuarios que necesitan realizar MFA no pueden iniciar sesión porque no hay interacción.
> - Los usuarios no pueden realizar inicio de sesión único (SSO).

### <a name="constraints"></a>Restricciones

También se aplican las restricciones siguientes:

- El flujo de nombre de usuario y contraseña no es compatible con el acceso condicional ni la autenticación multifactor. Como consecuencia, si la aplicación se ejecuta en un inquilino de Azure AD donde el administrador de inquilinos requiere la autenticación multifactor, no se puede usar este flujo. Muchas organizaciones hacen eso.
- Solo funciona para las cuentas profesionales y educativas (no MSA).
- El flujo está disponible en el escritorio de .NET y .NET Core, pero no en UWP.

### <a name="b2c-specifics"></a>Características específicas de B2C

Para más información, consulte [Credenciales de contraseña del propietario del recurso (ROPC) con B2C](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/AAD-B2C-specifics#resource-owner-password-credentials-ropc-with-b2c).

### <a name="use-it"></a>Uso

# <a name="net"></a>[.NET](#tab/dotnet)

`IPublicClientApplication` contiene el método `AcquireTokenByUsernamePassword`.

El ejemplo siguiente presenta un caso simplificado.

```csharp
static async Task GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication app;
 app = PublicClientApplicationBuilder.Create(clientId)
       .WithAuthority(authority)
       .Build();
 var accounts = await app.GetAccountsAsync();

 AuthenticationResult result = null;
 if (accounts.Any())
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
                    .ExecuteAsync();
 }
 else
 {
  try
  {
   var securePassword = new SecureString();
   foreach (char c in "dummy")        // you should fetch the password
    securePassword.AppendChar(c);  // keystroke by keystroke

   result = await app.AcquireTokenByUsernamePassword(scopes,
                                                    "joe@contoso.com",
                                                     securePassword)
                      .ExecuteAsync();
  }
  catch(MsalException)
  {
   // See details below
  }
 }
 Console.WriteLine(result.Account.Username);
}
```

El ejemplo siguiente presenta el caso más reciente, con explicaciones de la clase de excepciones que se pueden obtener y sus mitigaciones.

```csharp
static async Task GetATokenForGraph()
{
 string authority = "https://login.microsoftonline.com/contoso.com";
 string[] scopes = new string[] { "user.read" };
 IPublicClientApplication app;
 app = PublicClientApplicationBuilder.Create(clientId)
                                   .WithAuthority(authority)
                                   .Build();
 var accounts = await app.GetAccountsAsync();

 AuthenticationResult result = null;
 if (accounts.Any())
 {
  result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
                    .ExecuteAync();
 }
 else
 {
  try
  {
   var securePassword = new SecureString();
   foreach (char c in "dummy")        // you should fetch the password keystroke
    securePassword.AppendChar(c);  // by keystroke

   result = await app.AcquireTokenByUsernamePassword(scopes,
                                                    "joe@contoso.com",
                                                    securePassword)
                    .ExecuteAsync();
  }
  catch (MsalUiRequiredException ex) when (ex.Message.Contains("AADSTS65001"))
  {
   // Here are the kind of error messages you could have, and possible mitigations

   // ------------------------------------------------------------------------
   // MsalUiRequiredException: AADSTS65001: The user or administrator has not consented to use the application
   // with ID '{appId}' named '{appName}'. Send an interactive authorization request for this user and resource.

   // Mitigation: you need to get user consent first. This can be done either statically (through the portal),
   /// or dynamically (but this requires an interaction with Azure AD, which is not possible with
   // the username/password flow)
   // Statically: in the portal by doing the following in the "API permissions" tab of the application registration:
   // 1. Click "Add a permission" and add all the delegated permissions corresponding to the scopes you want (for instance
   // User.Read and User.ReadBasic.All)
   // 2. Click "Grant/revoke admin consent for <tenant>") and click "yes".
   // Dynamically, if you are not using .NET Core (which does not have any Web UI) by
   // calling (once only) AcquireTokenInteractive.
   // remember that Username/password is for public client applications that is desktop/mobile applications.
   // If you are using .NET core or don't want to call AcquireTokenInteractive, you might want to:
   // - use device code flow (See https://aka.ms/msal-net-device-code-flow)
   // - or suggest the user to navigate to a URL to consent: https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id={clientId}&response_type=code&scope=user.read
   // ------------------------------------------------------------------------

   // ------------------------------------------------------------------------
   // ErrorCode: invalid_grant
   // SubError: basic_action
   // MsalUiRequiredException: AADSTS50079: The user is required to use multi-factor authentication.
   // The tenant admin for your organization has chosen to oblige users to perform multi-factor authentication.
   // Mitigation: none for this flow
   // Your application cannot use the Username/Password grant.
   // Like in the previous case, you might want to use an interactive flow (AcquireTokenInteractive()),
   // or Device Code Flow instead.
   // Note this is one of the reason why using username/password is not recommended;
   // ------------------------------------------------------------------------

   // ------------------------------------------------------------------------
   // ex.ErrorCode: invalid_grant
   // subError: null
   // Message = "AADSTS70002: Error validating credentials.
   // AADSTS50126: Invalid username or password
   // In the case of a managed user (user from an Azure AD tenant opposed to a
   // federated user, which would be owned
   // in another IdP through ADFS), the user has entered the wrong password
   // Mitigation: ask the user to re-enter the password
   // ------------------------------------------------------------------------

   // ------------------------------------------------------------------------
   // ex.ErrorCode: invalid_grant
   // subError: null
   // MsalServiceException: ADSTS50034: To sign into this application the account must be added to
   // the {domainName} directory.
   // or The user account does not exist in the {domainName} directory. To sign into this application,
   // the account must be added to the directory.
   // The user was not found in the directory
   // Explanation: wrong username
   // Mitigation: ask the user to re-enter the username.
   // ------------------------------------------------------------------------
  }
  catch (MsalServiceException ex) when (ex.ErrorCode == "invalid_request")
  {
   // ------------------------------------------------------------------------
   // AADSTS90010: The grant type is not supported over the /common or /consumers endpoints.
   // Please use the /organizations or tenant-specific endpoint.
   // you used common.
   // Mitigation: as explained in the message from Azure AD, the authority you use in the application needs
   // to be tenanted or otherwise "organizations". change the
   // "Tenant": property in the appsettings.json to be a GUID (tenant Id), or domain name (contoso.com)
   // if such a domain is registered with your tenant
   // or "organizations", if you want this application to sign-in users in any Work and School accounts.
   // ------------------------------------------------------------------------

  }
  catch (MsalServiceException ex) when (ex.ErrorCode == "unauthorized_client")
  {
   // ------------------------------------------------------------------------
   // AADSTS700016: Application with identifier '{clientId}' was not found in the directory '{domain}'.
   // This can happen if the application has not been installed by the administrator of the tenant or consented
   // to by any user in the tenant.
   // You may have sent your authentication request to the wrong tenant
   // Cause: The clientId in the appsettings.json might be wrong
   // Mitigation: check the clientId and the app registration
   // ------------------------------------------------------------------------
  }
  catch (MsalServiceException ex) when (ex.ErrorCode == "invalid_client")
  {
   // ------------------------------------------------------------------------
   // AADSTS70002: The request body must contain the following parameter: 'client_secret or client_assertion'.
   // Explanation: this can happen if your application was not registered as a public client application in Azure AD
   // Mitigation: in the Azure portal, edit the manifest for your application and set the `allowPublicClient` to `true`
   // ------------------------------------------------------------------------
  }
  catch (MsalServiceException)
  {
   throw;
  }

  catch (MsalClientException ex) when (ex.ErrorCode == "unknown_user_type")
  {
   // Message = "Unsupported User Type 'Unknown'. Please see https://aka.ms/msal-net-up"
   // The user is not recognized as a managed user, or a federated user. Azure AD was not
   // able to identify the IdP that needs to process the user
   throw new ArgumentException("U/P: Wrong username", ex);
  }
  catch (MsalClientException ex) when (ex.ErrorCode == "user_realm_discovery_failed")
  {
   // The user is not recognized as a managed user, or a federated user. Azure AD was not
   // able to identify the IdP that needs to process the user. That's for instance the case
   // if you use a phone number
   throw new ArgumentException("U/P: Wrong username", ex);
  }
  catch (MsalClientException ex) when (ex.ErrorCode == "unknown_user")
  {
   // the username was probably empty
   // ex.Message = "Could not identify the user logged into the OS. See https://aka.ms/msal-net-iwa for details."
   throw new ArgumentException("U/P: Wrong username", ex);
  }
  catch (MsalClientException ex) when (ex.ErrorCode == "parsing_wstrust_response_failed")
  {
   // ------------------------------------------------------------------------
   // In the case of a Federated user (that is owned by a federated IdP, as opposed to a managed user owned in an Azure AD tenant)
   // ID3242: The security token could not be authenticated or authorized.
   // The user does not exist or has entered the wrong password
   // ------------------------------------------------------------------------
  }
 }

 Console.WriteLine(result.Account.Username);
}
```

Para más información sobre todos los modificadores que se pueden aplicar a `AcquireTokenByUsernamePassword`, consulte [AcquireTokenByUsernamePasswordParameterBuilder](/dotnet/api/microsoft.identity.client.acquiretokenbyusernamepasswordparameterbuilder?view=azure-dotnet-preview#methods).

# <a name="java"></a>[Java](#tab/java)

El siguiente extracto procede de los [ejemplos de desarrollo de Java de MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/public-client/).

```Java
private static IAuthenticationResult acquireTokenUsernamePassword() throws Exception {

    // Load token cache from file and initialize token cache aspect. The token cache will have
    // dummy data, so the acquireTokenSilently call will fail.
    TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

    PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
            .authority(AUTHORITY)
            .setTokenCacheAccessAspect(tokenCacheAspect)
            .build();

    Set<IAccount> accountsInCache = pca.getAccounts().join();
    // Take first account in the cache. In a production application, you would filter
    // accountsInCache to get the right account for the user authenticating.
    IAccount account = accountsInCache.iterator().next();

    IAuthenticationResult result;
    try {
        SilentParameters silentParameters =
                SilentParameters
                        .builder(SCOPE, account)
                        .build();
        // try to acquire token silently. This call will fail since the token cache
        // does not have any data for the user you are trying to acquire a token for
        result = pca.acquireTokenSilently(silentParameters).join();
    } catch (Exception ex) {
        if (ex.getCause() instanceof MsalException) {

            UserNamePasswordParameters parameters =
                    UserNamePasswordParameters
                            .builder(SCOPE, USER_NAME, USER_PASSWORD.toCharArray())
                            .build();
            // Try to acquire a token via username/password. If successful, you should see
            // the token and account information printed out to console
            result = pca.acquireToken(parameters).join();
        } else {
            // Handle other exceptions accordingly
            throw ex;
        }
    }
    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

Este extracto procede de los [ejemplos de desarrollo de Python de MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/dev/sample/).

```Python
# Create a preferably long-lived app instance which maintains a token cache.
app = msal.PublicClientApplication(
    config["client_id"], authority=config["authority"],
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )

# The pattern to acquire a token looks like this.
result = None

# Firstly, check the cache to see if this end user has signed in before
accounts = app.get_accounts(username=config["username"])
if accounts:
    logging.info("Account(s) exists in cache, probably with token too. Let's try.")
    result = app.acquire_token_silent(config["scope"], account=accounts[0])

if not result:
    logging.info("No suitable token exists in cache. Let's get a new one from AAD.")
    # See this page for constraints of Username Password Flow.
    # https://github.com/AzureAD/microsoft-authentication-library-for-python/wiki/Username-Password-Authentication
    result = app.acquire_token_by_username_password(
        config["username"], config["password"], scopes=config["scope"])
```

# <a name="macos"></a>[macOS](#tab/macOS)

Este flujo no se admite en MSAL para macOS.

---

## <a name="command-line-tool-without-a-web-browser"></a>Herramienta de línea de comandos sin un explorador web

### <a name="device-code-flow"></a>Flujo de código de dispositivo

Si va a escribir una herramienta de línea de comandos que no tenga controles web y no puede o no quiere usar los flujos anteriores, debe usar el flujo de código de dispositivo.

La autenticación interactiva con Azure AD requiere un explorador web. Para más información, consulte [Uso de exploradores web](https://aka.ms/msal-net-uses-web-browser). Para autenticar a los usuarios en dispositivos o sistemas operativos que no proporcionan un explorador web, el flujo de código de dispositivo permite al usuario usar otro dispositivo, como un equipo o un teléfono móvil, para iniciar sesión de forma interactiva. Con el flujo de código de dispositivo, la aplicación obtiene tokens mediante un proceso de dos pasos diseñado para estos dispositivos o sistemas operativos. Ejemplos de estas aplicaciones son aquellas que se ejecutan en iOT o herramientas de línea de comandos (CLI). La idea es la siguiente:

1. Cada vez que se necesita autenticación de usuario, la aplicación proporciona un código para el usuario. Se pide al usuario que use otro dispositivo, como un smartphone conectado a Internet, para ir a una dirección URL, por ejemplo, `https://microsoft.com/devicelogin`. Luego se pide al usuario que escriba el código. Una vez hecho eso, la página web lleva al usuario a una experiencia de autenticación normal, que incluye peticiones de consentimiento y autenticación multifactor, en caso necesario.

2. Tras una autenticación correcta, la aplicación de línea de comandos recibe los tokens necesarios a través de un canal posterior y los usa para realizar las llamadas a la API web que necesita.

### <a name="use-it"></a>Uso

# <a name="net"></a>[.NET](#tab/dotnet)

`IPublicClientApplication` contiene un método denominado `AcquireTokenWithDeviceCode`.

```csharp
 AcquireTokenWithDeviceCode(IEnumerable<string> scopes,
                            Func<DeviceCodeResult, Task> deviceCodeResultCallback)
```

Este método toma como parámetros:

- Los elementos `scopes` para los que solicitar un token de acceso.
- Una devolución de llamada que recibe `DeviceCodeResult`.

  ![Propiedades de DeviceCodeResult](https://user-images.githubusercontent.com/13203188/56024968-7af1b980-5d11-11e9-84c2-5be2ef306dc5.png)

El código de ejemplo siguiente presenta el caso más reciente, con explicaciones de la clase de excepciones que se pueden obtener y su mitigación.

```csharp
private const string ClientId = "<client_guid>";
private const string Authority = "https://login.microsoftonline.com/contoso.com";
private readonly string[] Scopes = new string[] { "user.read" };

static async Task<AuthenticationResult> GetATokenForGraph()
{
    IPublicClientApplication pca = PublicClientApplicationBuilder
            .Create(ClientId)
            .WithAuthority(Authority)
            .WithDefaultRedirectUri()
            .Build();

    var accounts = await pca.GetAccountsAsync();

    // All AcquireToken* methods store the tokens in the cache, so check the cache first
    try
    {
        return await pca.AcquireTokenSilent(Scopes, accounts.FirstOrDefault())
            .ExecuteAsync();
    }
    catch (MsalUiRequiredException ex)
    {
        // No token found in the cache or AAD insists that a form interactive auth is required (e.g. the tenant admin turned on MFA)
        // If you want to provide a more complex user experience, check out ex.Classification

        return await AcquireByDeviceCodeAsync(pca);
    }         
}

private async Task<AuthenticationResult> AcquireByDeviceCodeAsync(IPublicClientApplication pca)
{
    try
    {
        var result = await pca.AcquireTokenWithDeviceCode(scopes,
            deviceCodeResult =>
            {
                    // This will print the message on the console which tells the user where to go sign-in using
                    // a separate browser and the code to enter once they sign in.
                    // The AcquireTokenWithDeviceCode() method will poll the server after firing this
                    // device code callback to look for the successful login of the user via that browser.
                    // This background polling (whose interval and timeout data is also provided as fields in the
                    // deviceCodeCallback class) will occur until:
                    // * The user has successfully logged in via browser and entered the proper code
                    // * The timeout specified by the server for the lifetime of this code (typically ~15 minutes) has been reached
                    // * The developing application calls the Cancel() method on a CancellationToken sent into the method.
                    //   If this occurs, an OperationCanceledException will be thrown (see catch below for more details).
                    Console.WriteLine(deviceCodeResult.Message);
                return Task.FromResult(0);
            }).ExecuteAsync();

        Console.WriteLine(result.Account.Username);
        return result;
    }
    // TODO: handle or throw all these exceptions depending on your app
    catch (MsalServiceException ex)
    {
        // Kind of errors you could have (in ex.Message)

        // AADSTS50059: No tenant-identifying information found in either the request or implied by any provided credentials.
        // Mitigation: as explained in the message from Azure AD, the authoriy needs to be tenanted. you have probably created
        // your public client application with the following authorities:
        // https://login.microsoftonline.com/common or https://login.microsoftonline.com/organizations

        // AADSTS90133: Device Code flow is not supported under /common or /consumers endpoint.
        // Mitigation: as explained in the message from Azure AD, the authority needs to be tenanted

        // AADSTS90002: Tenant <tenantId or domain you used in the authority> not found. This may happen if there are
        // no active subscriptions for the tenant. Check with your subscription administrator.
        // Mitigation: if you have an active subscription for the tenant this might be that you have a typo in the
        // tenantId (GUID) or tenant domain name.
    }
    catch (OperationCanceledException ex)
    {
        // If you use a CancellationToken, and call the Cancel() method on it, then this *may* be triggered
        // to indicate that the operation was cancelled.
        // See https://docs.microsoft.com/dotnet/standard/threading/cancellation-in-managed-threads
        // for more detailed information on how C# supports cancellation in managed threads.
    }
    catch (MsalClientException ex)
    {
        // Possible cause - verification code expired before contacting the server
        // This exception will occur if the user does not manage to sign-in before a time out (15 mins) and the
        // call to `AcquireTokenWithDeviceCode` is not cancelled in between
    }
}
```
# <a name="java"></a>[Java](#tab/java)

Este extracto procede de los [ejemplos de desarrollo de Java de MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/public-client/).

```java
private static IAuthenticationResult acquireTokenDeviceCode() throws Exception {

    // Load token cache from file and initialize token cache aspect. The token cache will have
    // dummy data, so the acquireTokenSilently call will fail.
    TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

    PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
            .authority(AUTHORITY)
            .setTokenCacheAccessAspect(tokenCacheAspect)
            .build();

    Set<IAccount> accountsInCache = pca.getAccounts().join();
    // Take first account in the cache. In a production application, you would filter
    // accountsInCache to get the right account for the user authenticating.
    IAccount account = accountsInCache.iterator().next();

    IAuthenticationResult result;
    try {
        SilentParameters silentParameters =
                SilentParameters
                        .builder(SCOPE, account)
                        .build();

        // try to acquire token silently. This call will fail since the token cache
        // does not have any data for the user you are trying to acquire a token for
        result = pca.acquireTokenSilently(silentParameters).join();
    } catch (Exception ex) {
        if (ex.getCause() instanceof MsalException) {

            Consumer<DeviceCode> deviceCodeConsumer = (DeviceCode deviceCode) ->
                    System.out.println(deviceCode.message());

            DeviceCodeFlowParameters parameters =
                    DeviceCodeFlowParameters
                            .builder(SCOPE, deviceCodeConsumer)
                            .build();

            // Try to acquire a token via device code flow. If successful, you should see
            // the token and account information printed out to console, and the sample_cache.json
            // file should have been updated with the latest tokens.
            result = pca.acquireToken(parameters).join();
        } else {
            // Handle other exceptions accordingly
            throw ex;
        }
    }
    return result;
}
```

# <a name="python"></a>[Python](#tab/python)

Este extracto procede de los [ejemplos de desarrollo de Python de MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/dev/sample/).

```Python
# Create a preferably long-lived app instance which maintains a token cache.
app = msal.PublicClientApplication(
    config["client_id"], authority=config["authority"],
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )

# The pattern to acquire a token looks like this.
result = None

# Note: If your device-flow app does not have any interactive ability, you can
#   completely skip the following cache part. But here we demonstrate it anyway.
# We now check the cache to see if we have some end users signed in before.
accounts = app.get_accounts()
if accounts:
    logging.info("Account(s) exists in cache, probably with token too. Let's try.")
    print("Pick the account you want to use to proceed:")
    for a in accounts:
        print(a["username"])
    # Assuming the end user chose this one
    chosen = accounts[0]
    # Now let's try to find a token in cache for this account
    result = app.acquire_token_silent(config["scope"], account=chosen)

if not result:
    logging.info("No suitable token exists in cache. Let's get a new one from AAD.")

    flow = app.initiate_device_flow(scopes=config["scope"])
    if "user_code" not in flow:
        raise ValueError(
            "Fail to create device flow. Err: %s" % json.dumps(flow, indent=4))

    print(flow["message"])
    sys.stdout.flush()  # Some terminal needs this to ensure the message is shown

    # Ideally you should wait here, in order to save some unnecessary polling
    # input("Press Enter after signing in from another device to proceed, CTRL+C to abort.")

    result = app.acquire_token_by_device_flow(flow)  # By default it will block
        # You can follow this instruction to shorten the block time
        #    https://msal-python.readthedocs.io/en/latest/#msal.PublicClientApplication.acquire_token_by_device_flow
        # or you may even turn off the blocking behavior,
        # and then keep calling acquire_token_by_device_flow(flow) in your own customized loop
```

# <a name="macos"></a>[macOS](#tab/macOS)

Este flujo no se aplica a macOS.

---

## <a name="file-based-token-cache"></a>Caché de tokens basada en archivos

En MSAL.NET, se proporciona una caché de tokens en memoria de forma predeterminada.

### <a name="serialization-is-customizable-in-windows-desktop-apps-and-web-apps-or-web-apis"></a>La serialización de las aplicaciones de escritorio de Windows y las aplicaciones web o API web es personalizable.

En el caso de .NET Framework y .NET Core, si no hace nada más, la caché de tokens en memoria permanece mientras dura la aplicación. Para entender por qué no se proporciona serialización de serie, recuerde que las aplicaciones de escritorio de MSAL.NET o .NET Core pueden ser aplicaciones de consola o Windows (que tendrían acceso al sistema de archivos), *pero también* aplicaciones web o API web. Estas aplicaciones web y API web pueden usar algunos mecanismos de caché específicos, como bases de datos, cachés distribuidas y cachés en Redis. Para que haya una aplicación de caché de tokens persistente en el escritorio de .NET o .NET Core, es preciso personalizar la serialización.

Las clases e interfaces implicadas en la serialización de la caché de tokens son de los tipos siguientes:

- ``ITokenCache``, que define eventos para suscribirse a las solicitudes de serialización de la caché de tokens, así como métodos para serializar o deserializar la caché en diversos formatos (ADAL v3.0, MSAL 2.x y MSAL 3.x = ADAL v5.0).
- ``TokenCacheCallback`` es una devolución de llamada que se pasa a los eventos para que pueda controlar la serialización. Se les llamará con argumentos del tipo ``TokenCacheNotificationArgs``.
- ``TokenCacheNotificationArgs`` solo proporciona el elemento ``ClientId`` de la aplicación y una referencia al usuario para el que está disponible el token.

  ![Diagrama de serialización de la caché de tokens](https://user-images.githubusercontent.com/13203188/56027172-d58d1480-5d15-11e9-8ada-c0292f1800b3.png)

> [!IMPORTANT]
> MSAL.NET crea automáticamente las cachés de tokens y le ofrece la caché de `IToken` cuando se llama a las propiedades `UserTokenCache` y `AppTokenCache` de una aplicación. Se supone que no va a implementar la interfaz usted mismo. Su responsabilidad al implementar una serialización de caché de tokens personalizada es:
>
> - Reaccionar a los eventos `BeforeAccess` y `AfterAccess`, o sus homólogos *asincrónicos*. El delegado `BeforeAccess` es responsable de deserializar la caché. El delegado `AfterAccess` es responsable de serializar la caché.
> - Entender que parte de estos eventos almacenan o cargan blobs que se pasan a través del argumento del evento al almacenamiento que se quiera.

Las estrategias varían en función de si se escribe una serialización de la caché de tokens para una aplicación cliente pública, como un equipo de escritorio, o una aplicación cliente confidencial, como una aplicación web, API web o aplicación de demonio.

A partir de MSAL v2.x, tiene varias opciones. La elección depende de si quiere serializar la caché solo al formato de MSAL.NET, que es una caché de formato unificado común con MSAL pero también el resto de plataformas. O bien, es posible que además quiera admitir la serialización de la caché de tokens [heredada](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/wiki/Token-cache-serialization) de ADAL v3.

La personalización de la serialización de la caché de tokens para compartir el estado de SSO entre ADAL.NET 3.x, ADAL.NET 5.x y MSAL.NET se explica en parte del ejemplo [active-directory-dotnet-v1-to-v2](https://github.com/Azure-Samples/active-directory-dotnet-v1-to-v2).

### <a name="simple-token-cache-serialization-msal-only"></a>Serialización de la caché de tokens simple (solo MSAL)

El siguiente ejemplo es una implementación sencilla de serialización personalizada de una caché de tokens para aplicaciones de escritorio. En él, la caché de tokens de usuario está en un archivo de la misma carpeta que la aplicación.

Después de compilar la aplicación, se llama a ``TokenCacheHelper.EnableSerialization()`` y se pasa la aplicación `UserTokenCache` para habilitar la serialización.

```csharp
app = PublicClientApplicationBuilder.Create(ClientId)
    .Build();
TokenCacheHelper.EnableSerialization(app.UserTokenCache);
```

Esta clase auxiliar es similar al siguiente fragmento de código:

```csharp
static class TokenCacheHelper
 {
  public static void EnableSerialization(ITokenCache tokenCache)
  {
   tokenCache.SetBeforeAccess(BeforeAccessNotification);
   tokenCache.SetAfterAccess(AfterAccessNotification);
  }

  /// <summary>
  /// Path to the token cache
  /// </summary>
  public static readonly string CacheFilePath = System.Reflection.Assembly.GetExecutingAssembly().Location + ".msalcache.bin3";

  private static readonly object FileLock = new object();


  private static void BeforeAccessNotification(TokenCacheNotificationArgs args)
  {
   lock (FileLock)
   {
    args.TokenCache.DeserializeMsalV3(File.Exists(CacheFilePath)
            ? ProtectedData.Unprotect(File.ReadAllBytes(CacheFilePath),
                                      null,
                                      DataProtectionScope.CurrentUser)
            : null);
   }
  }

  private static void AfterAccessNotification(TokenCacheNotificationArgs args)
  {
   // if the access operation resulted in a cache update
   if (args.HasStateChanged)
   {
    lock (FileLock)
    {
     // reflect changesgs in the persistent store
     File.WriteAllBytes(CacheFilePath,
                         ProtectedData.Protect(args.TokenCache.SerializeMsalV3(),
                                                 null,
                                                 DataProtectionScope.CurrentUser)
                         );
    }
   }
  }
 }
```

Hay una versión preliminar de un serializador basado en archivos de la caché de tokens con calidad de producto para aplicaciones cliente públicas para aplicaciones de escritorio que se ejecutan en Windows, Mac y Linux disponible en la biblioteca de código abierto [Microsoft.Identity.Client.Extensions.Msal](https://github.com/AzureAD/microsoft-authentication-extensions-for-dotnet/tree/master/src/Microsoft.Identity.Client.Extensions.Msal). La puede incluir en las aplicaciones desde el siguiente paquete NuGet: [Microsoft.Identity.Client.Extensions.Msal](https://www.nuget.org/packages/Microsoft.Identity.Client.Extensions.Msal/).

> [!NOTE]
> Declinación de responsabilidades: La biblioteca Microsoft.Identity.Client.Extensions.Msal es una extensión de MSAL.NET. Las clases de estas bibliotecas podrían llegar a MSAL.NET en el futuro, tal como están o con cambios importantes.

### <a name="dual-token-cache-serialization-msal-unified-cache--adal-v3"></a>Serialización de la caché de tokens dual (caché unificada de MSAL y ADAL v3)

Es posible que quiera implementar la serialización de la caché de tokens con el formato de caché unificada. Este formato es común a ADAL.NET 4.x y MSAL.NET 2.x, además de otras MSAL de la misma generación o anterior, de la misma plataforma. Inspírese con el siguiente código:

```csharp
string appLocation = Path.GetDirectoryName(Assembly.GetEntryAssembly().Location;
string cacheFolder = Path.GetFullPath(appLocation) + @"..\..\..\..");
string adalV3cacheFileName = Path.Combine(cacheFolder, "cacheAdalV3.bin");
string unifiedCacheFileName = Path.Combine(cacheFolder, "unifiedCache.bin");

IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
                                    .Build();
FilesBasedTokenCacheHelper.EnableSerialization(app.UserTokenCache,
                                               unifiedCacheFileName,
                                               adalV3cacheFileName);

```

Esta vez la clase auxiliar es similar al siguiente código:

```csharp
using System;
using System.IO;
using System.Security.Cryptography;
using Microsoft.Identity.Client;

namespace CommonCacheMsalV3
{
 /// <summary>
 /// Simple persistent cache implementation of the dual cache serialization (ADAL V3 legacy
 /// and unified cache format) for a desktop applications (from MSAL 2.x)
 /// </summary>
 static class FilesBasedTokenCacheHelper
 {
  /// <summary>
  /// Get the user token cache
  /// </summary>
  /// <param name="adalV3CacheFileName">File name where the cache is serialized with the
  /// ADAL V3 token cache format. Can
  /// be <c>null</c> if you don't want to implement the legacy ADAL V3 token cache
  /// serialization in your MSAL 2.x+ application</param>
  /// <param name="unifiedCacheFileName">File name where the cache is serialized
  /// with the Unified cache format, common to
  /// ADAL V4 and MSAL V2 and above, and also across ADAL/MSAL on the same platform.
  ///  Should not be <c>null</c></param>
  /// <returns></returns>
  public static void EnableSerialization(ITokenCache cache, string unifiedCacheFileName, string adalV3CacheFileName)
  {
   UnifiedCacheFileName = unifiedCacheFileName;
   AdalV3CacheFileName = adalV3CacheFileName;

   cache.SetBeforeAccess(BeforeAccessNotification);
   cache.SetAfterAccess(AfterAccessNotification);
  }

  /// <summary>
  /// File path where the token cache is serialized with the unified cache format
  /// (ADAL.NET V4, MSAL.NET V3)
  /// </summary>
  public static string UnifiedCacheFileName { get; private set; }

  /// <summary>
  /// File path where the token cache is serialized with the legacy ADAL V3 format
  /// </summary>
  public static string AdalV3CacheFileName { get; private set; }

  private static readonly object FileLock = new object();

  public static void BeforeAccessNotification(TokenCacheNotificationArgs args)
  {
   lock (FileLock)
   {
    args.TokenCache.DeserializeAdalV3(ReadFromFileIfExists(AdalV3CacheFileName));
    try
    {
     args.TokenCache.DeserializeMsalV3(ReadFromFileIfExists(UnifiedCacheFileName));
    }
    catch(Exception ex)
    {
     // Compatibility with the MSAL v2 cache if you used one
     args.TokenCache.DeserializeMsalV2(ReadFromFileIfExists(UnifiedCacheFileName));
    }
   }
  }

  public static void AfterAccessNotification(TokenCacheNotificationArgs args)
  {
   // if the access operation resulted in a cache update
   if (args.HasStateChanged)
   {
    lock (FileLock)
    {
     WriteToFileIfNotNull(UnifiedCacheFileName, args.TokenCache.SerializeMsalV3());
     if (!string.IsNullOrWhiteSpace(AdalV3CacheFileName))
     {
      WriteToFileIfNotNull(AdalV3CacheFileName, args.TokenCache.SerializeAdalV3());
     }
    }
   }
  }

  /// <summary>
  /// Read the content of a file if it exists
  /// </summary>
  /// <param name="path">File path</param>
  /// <returns>Content of the file (in bytes)</returns>
  private static byte[] ReadFromFileIfExists(string path)
  {
   byte[] protectedBytes = (!string.IsNullOrEmpty(path) && File.Exists(path))
       ? File.ReadAllBytes(path) : null;
   byte[] unprotectedBytes = encrypt ?
       ((protectedBytes != null) ? ProtectedData.Unprotect(protectedBytes, null, DataProtectionScope.CurrentUser) : null)
       : protectedBytes;
   return unprotectedBytes;
  }

  /// <summary>
  /// Writes a blob of bytes to a file. If the blob is <c>null</c>, deletes the file
  /// </summary>
  /// <param name="path">path to the file to write</param>
  /// <param name="blob">Blob of bytes to write</param>
  private static void WriteToFileIfNotNull(string path, byte[] blob)
  {
   if (blob != null)
   {
    byte[] protectedBytes = encrypt
      ? ProtectedData.Protect(blob, null, DataProtectionScope.CurrentUser)
      : blob;
    File.WriteAllBytes(path, protectedBytes);
   }
   else
   {
    File.Delete(path);
   }
  }

  // Change if you want to test with an un-encrypted blob (this is a json format)
  private static bool encrypt = true;
 }
}
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Llamada a una API web desde la aplicación de escritorio](scenario-desktop-call-api.md)