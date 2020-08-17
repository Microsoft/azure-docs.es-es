---
title: Obtención de un token para llamar a una API web (aplicaciones móviles) | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo compilar una aplicación móvil que llama a las API web. (Obtenga un token para la aplicación).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.reviewer: brandwe
ms.custom: aaddev
ms.openlocfilehash: dfccc274ef920c59d39c160055ab27a6900c839c
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88141285"
---
# <a name="get-a-token-for-a-mobile-app-that-calls-web-apis"></a>Obtenga un token para una aplicación móvil que llama a las API web

Para que la aplicación pueda llamar a las API web protegidas, necesita un token de acceso. Este artículo le guía a través del proceso para obtener un token usando la Biblioteca de autenticación de Microsoft (MSAL).

## <a name="define-a-scope"></a>Definición de un ámbito

Cuando solicite un token, deberá definir un ámbito. El ámbito determina a qué datos puede acceder la aplicación.

La manera más sencilla de definir un ámbito es combinar el `App ID URI` de la API web deseada con el ámbito `.default`. Esta definición indica a la Plataforma de identidad de Microsoft que su aplicación requiere todos los ámbitos establecidos en el portal.

### <a name="android"></a>Android
```Java
String[] SCOPES = {"https://graph.microsoft.com/.default"};
```

### <a name="ios"></a>iOS
```swift
let scopes = ["https://graph.microsoft.com/.default"]
```

### <a name="xamarin"></a>Xamarin
```csharp
var scopes = new [] {"https://graph.microsoft.com/.default"};
```

## <a name="get-tokens"></a>Obtención de tokens

### <a name="acquire-tokens-via-msal"></a>Adquisición de tokens a través de MSAL

MSAL permite que las aplicaciones adquieran tokens de forma silenciosa e interactiva. Cuando llama a `AcquireTokenSilent()` o a `AcquireTokenInteractive()`, MSAL devuelve un token de acceso para los ámbitos solicitados. El patrón correcto consiste en realizar una solicitud silenciosa y, a continuación, revertir a una solicitud interactiva.

#### <a name="android"></a>Android

```Java
String[] SCOPES = {"https://graph.microsoft.com/.default"};
PublicClientApplication sampleApp = new PublicClientApplication(
                    this.getApplicationContext(),
                    R.raw.auth_config);

// Check if there are any accounts we can sign in silently.
// Result is in the silent callback (success or error).
sampleApp.getAccounts(new PublicClientApplication.AccountsLoadedCallback() {
    @Override
    public void onAccountsLoaded(final List<IAccount> accounts) {

        if (accounts.isEmpty() && accounts.size() == 1) {
            // TODO: Create a silent callback to catch successful or failed request.
            sampleApp.acquireTokenSilentAsync(SCOPES, accounts.get(0), getAuthSilentCallback());
        } else {
            /* No accounts or > 1 account. */
        }
    }
});

[...]

// No accounts found. Interactively request a token.
// TODO: Create an interactive callback to catch successful or failed requests.
sampleApp.acquireToken(getActivity(), SCOPES, getAuthInteractiveCallback());
```

#### <a name="ios"></a>iOS

En primer lugar, intente adquirir un token silenciosamente:

```objc

NSArray *scopes = @[@"https://graph.microsoft.com/.default"];
NSString *accountIdentifier = @"my.account.id";

MSALAccount *account = [application accountForIdentifier:accountIdentifier error:nil];

MSALSilentTokenParameters *silentParams = [[MSALSilentTokenParameters alloc] initWithScopes:scopes account:account];
[application acquireTokenSilentWithParameters:silentParams completionBlock:^(MSALResult *result, NSError *error) {

    if (!error)
    {
        // You'll want to get the account identifier to retrieve and reuse the account
        // for later acquireToken calls
        NSString *accountIdentifier = result.account.identifier;

        // Access token to call the web API
        NSString *accessToken = result.accessToken;
    }

    // Check the error
    if (error && [error.domain isEqual:MSALErrorDomain] && error.code == MSALErrorInteractionRequired)
    {
        // Interactive auth will be required, call acquireTokenWithParameters:error:
        return;
    }
}];
```

```swift

let scopes = ["https://graph.microsoft.com/.default"]
let accountIdentifier = "my.account.id"

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

    // You'll want to get the account identifier to retrieve and reuse the account
    // for later acquireToken calls
    let accountIdentifier = authResult.account.identifier

    // Access token to call the web API
    let accessToken = authResult.accessToken
}
```

Si MSAL devuelve `MSALErrorInteractionRequired`, intente adquirir tokens de forma interactiva:

```objc
UIViewController *viewController = ...; // Pass a reference to the view controller that should be used when getting a token interactively
MSALWebviewParameters *webParameters = [[MSALWebviewParameters alloc] initWithAuthPresentationViewController:viewController];
MSALInteractiveTokenParameters *interactiveParams = [[MSALInteractiveTokenParameters alloc] initWithScopes:scopes webviewParameters:webParameters];
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

```swift
let viewController = ... // Pass a reference to the view controller that should be used when getting a token interactively
let webviewParameters = MSALWebviewParameters(authPresentationViewController: viewController)
let interactiveParameters = MSALInteractiveTokenParameters(scopes: scopes, webviewParameters: webviewParameters)
application.acquireToken(with: interactiveParameters, completionBlock: { (result, error) in

    guard let authResult = result, error == nil else {
        print(error!.localizedDescription)
        return
    }

    // Get access token from result
    let accessToken = authResult.accessToken
})
```

MSAL para iOS y macOS admite varios modificadores al obtener un token de forma interactiva o silenciosa:
* [Parámetros comunes para obtener un token](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALTokenParameters.html#/Configuration%20parameters)
* [Parámetros para la obtención interactiva de un token](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALInteractiveTokenParameters.html#/Configuring%20MSALInteractiveTokenParameters)
* [Parámetros para la obtención silenciosa de un token](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALSilentTokenParameters.html)

#### <a name="xamarin"></a>Xamarin

En el ejemplo siguiente se muestra el código mínimo para obtener un token de forma interactiva. En el ejemplo se usa Microsoft Graph para leer el perfil del usuario.

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

#### <a name="mandatory-parameters-in-msalnet"></a>Parámetros obligatorios en MSAL.NET

`AcquireTokenInteractive` solo tiene un parámetro obligatorio: `scopes`. El parámetro `scopes` enumera las cadenas que definen los ámbitos para los que se requiere un token. Si el token es para Microsoft Graph, los ámbitos necesarios se pueden encontrar en la referencia de API de cada instancia de Microsoft Graph API. En la referencia, vaya a la sección "Permisos".

Por ejemplo, para [enumerar los contactos del usuario](/graph/api/user-list-contacts), se debe usar el ámbito "User.Read", "Contacts.Read". Para más información, consulte [Referencia de permisos de Microsoft Graph](/graph/permissions-reference).

En Android, puede especificar la actividad primaria al crear la aplicación mediante `PublicClientApplicationBuilder`. Si no especifica la actividad primaria en ese momento, puede especificarla más adelante mediante `.WithParentActivityOrWindow` como se indica en la sección siguiente. Si especifica la actividad primaria, el token vuelve a la actividad primaria después de la interacción. Si no se especifica, la llamada a `.ExecuteAsync()` inicia una excepción.

#### <a name="specific-optional-parameters-in-msalnet"></a>Parámetros opcionales específicos de MSAL.NET

En las secciones siguientes se explican los parámetros opcionales de MSAL.NET.

##### <a name="withprompt"></a>WithPrompt

El parámetro `WithPrompt()` controla la interactividad con el usuario mediante la especificación de un mensaje.

![Imagen que muestra los campos de la estructura del mensaje. Estos valores constantes controlan la interactividad con el usuario al definir el tipo de mensaje que muestra el parámetro WithPrompt().](https://user-images.githubusercontent.com/13203188/53438042-3fb85700-39ff-11e9-9a9e-1ff9874197b3.png)

La clase define las constantes siguientes:

- `SelectAccount` obliga al servicio de token de seguridad (STS) a presentar el cuadro de diálogo de selección de cuenta. El cuadro de diálogo contiene las cuentas para las que el usuario tiene una sesión. Puede usar esta opción si desea permitir que el usuario elija entre distintas identidades. Esta opción dirige a MSAL para que envíe `prompt=select_account` al proveedor de identidades.

    La constante `SelectAccount` es el valor predeterminado y proporciona de forma eficaz la mejor experiencia posible en función de la información disponible. La información disponible puede incluir la cuenta, la presencia de una sesión del usuario, etc. No cambie este valor predeterminado a menos que tenga buena razón para hacerlo.
- `Consent` le permite pedir consentimiento al usuario, aunque ya se haya concedido antes. En este caso, MSAL envía `prompt=consent` al proveedor de identidades.

    Es posible que desee usar la constante `Consent` en aplicaciones centradas en la seguridad donde la gobernanza de la organización requiere que los usuarios vean el cuadro de diálogo de consentimiento cada vez que utilicen la aplicación.
- `ForceLogin` permite al servicio solicitar credenciales al usuario, aunque no sea necesario.

    Esta opción puede ser útil si se produce un error en la adquisición del token y desea permitir que el usuario vuelva a iniciar sesión. En este caso, MSAL envía `prompt=login` al proveedor de identidades. Puede utilizar esta opción en aplicaciones centradas en la seguridad donde la gobernanza de la organización exige que el usuario vuelva a iniciar sesión cada vez que accede a determinadas partes de una aplicación.
- `Never` es solo para .NET 4.5 y Windows Runtime (WinRT). Esta constante no preguntará al usuario, pero intentará usar la cookie almacenada en la vista web incrustada oculta. Para más información, consulte [Uso de exploradores web con MSAL.NET](./msal-net-web-browsers.md).

    Si esta opción genera un error, `AcquireTokenInteractive` inicia una excepción para notificar que se necesita una interacción de la interfaz de usuario. Entonces, debe usar otro parámetro `Prompt`.
- `NoPrompt` no envía ningún mensaje al proveedor de identidades.

    Esta opción solo es útil para directivas de edición de perfil de Azure Active Directory B2C. Para más información, consulte [Información específica sobre B2C](https://aka.ms/msal-net-b2c-specificities).

##### <a name="withextrascopetoconsent"></a>WithExtraScopeToConsent

Utilice el modificador `WithExtraScopeToConsent` en un escenario avanzado donde desee que el usuario proporcione consentimiento por adelantado a varios recursos. Puede usar este modificador si no desea usar el consentimiento incremental, que normalmente se usa con MSAL.NET o la Plataforma de identidad de Microsoft 2.0. Para más información, consulte [Cómo tener el consentimiento del usuario por adelantado para varios recursos](scenario-desktop-production.md#have-the-user-consent-upfront-for-several-resources).

Este es un ejemplo de código:

```csharp
var result = await app.AcquireTokenInteractive(scopesForCustomerApi)
                     .WithExtraScopeToConsent(scopesForVendorApi)
                     .ExecuteAsync();
```

##### <a name="other-optional-parameters"></a>Otros parámetros opcionales

Para obtener más información sobre los demás parámetros opcionales para `AcquireTokenInteractive`, consulte la [documentación de referencia de AcquireTokenInteractiveParameterBuilder](/dotnet/api/microsoft.identity.client.acquiretokeninteractiveparameterbuilder?view=azure-dotnet-preview#methods).

### <a name="acquire-tokens-via-the-protocol"></a>Adquisición de tokens mediante el protocolo

No se recomienda usar el protocolo directamente para obtener tokens. Si lo hace, la aplicación no admitirá algunos escenarios que impliquen el inicio de sesión único (SSO), la administración de dispositivos ni el acceso condicional.

Si usa el protocolo para obtener tokens para aplicaciones móviles, realice dos solicitudes:

* Obtenga un código de autorización.
* Intercambie el código de un token.

#### <a name="get-an-authorization-code"></a>Obtener un código de autorización

```
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize?
client_id=<CLIENT_ID>
&response_type=code
&redirect_uri=<ENCODED_REDIRECT_URI>
&response_mode=query
&scope=openid%20offline_access%20https%3A%2F%2Fgraph.microsoft.com%2F.default
&state=12345
```

#### <a name="get-access-and-refresh-the-token"></a>Obtener acceso y actualizar el token

```HTTP
POST /{tenant}/oauth2/v2.0/token HTTP/1.1
Host: https://login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=<CLIENT_ID>
&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&code=OAAABAAAAiL9Kn2Z27UubvWFPbm0gLWQJVzCTE9UkP3pSx1aXxUjq3n8b2JRLk4OxVXr...
&redirect_uri=<ENCODED_REDIRECT_URI>
&grant_type=authorization_code
```

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Llamada a una API web](scenario-mobile-call-api.md)