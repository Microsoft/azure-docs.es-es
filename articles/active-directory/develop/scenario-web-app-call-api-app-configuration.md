---
title: Aplicación Web que llama a web API (configuración de código), plataforma Microsoft identity
description: Obtenga información sobre cómo compilar una aplicación Web que llama a web API (configuración de código de la aplicación)
services: active-directory
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: bd7f393f889facf147cf25625d5c3b20f886ddf5
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65784938"
---
# <a name="web-app-that-calls-web-apis---code-configuration"></a>Aplicación Web que llama a web API: configuración de código

Tal como se muestra en el [escenario inicia sesión a los usuarios de Web app](scenario-web-app-sign-user-overview.md), dado que la sesión de usuario se delega al identificador abierto connect (OIDC) middleware, desea enlace de seguridad en el proceso OIDC. La manera de hacerlo es diferente según el marco que usa (aquí ASP.NET y ASP.NET Core), pero al final, deberá suscribirse a eventos de middleware OIDC. El principio es:

- Permitirá a ASP.NET o ASP.NET core solicitar un código de autorización. Realizando este ASP.NET/ASP.NET core permitirá al usuario iniciar sesión y dar su consentimiento,
- Deberá suscribirse a la recepción del código de autorización de la aplicación Web.
- Cuando se recibe el código de autenticación, usará las bibliotecas MSAL para canjear el código y los tokens de acceso resultante y actualizar el almacén de símbolos (tokens) en la caché de tokens. Desde allí, la memoria caché puede utilizarse en otras partes de la aplicación para adquirir otros tokens de forma silenciosa.

## <a name="libraries-supporting-web-app-scenarios"></a>Bibliotecas que admiten escenarios de aplicación Web

Las bibliotecas que admiten el flujo de código de autorización para aplicaciones Web son:

| Biblioteca MSAL | DESCRIPCIÓN |
|--------------|-------------|
| ![MSAL.NET](media/sample-v2-code/logo_NET.png) <br/> MSAL.NET  | Las plataformas admitidas son las plataformas .NET Framework y .NET Core (no de UWP, Xamarin.iOS y Xamarin.Android como esas plataformas se usan para crear aplicaciones de cliente público) |
| ![Python](media/sample-v2-code/logo_python.png) <br/> MSAL.Python | Desarrollo en curso: en versión preliminar pública |
| ![Java](media/sample-v2-code/logo_java.png) <br/> MSAL.Java | Desarrollo en curso: en versión preliminar pública |

## <a name="aspnet-core-configuration"></a>Configuración de ASP.NET Core

En ASP.NET Core, suceden cosas en el `Startup.cs` archivo. Desea suscribirse a la `OnAuthorizationCodeReceived` abrir el Id. de evento connect y desde este evento, puede llamar a MSAL. Método de la red `AcquireTokenFromAuthorizationCode` que tiene el efecto de almacenar en la caché de tokens, el token de acceso para los ámbitos solicitados y un token de actualización que se usará para actualizar el token de acceso cuando está cerca de expiración u obtener un token para el mismo usuario , pero para un recurso diferente.

Los comentarios en el código siguiente le ayudará a entender algunos aspectos complicados de weaving MSAL.NET y ASP.NET Core

```CSharp
  services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
  {
   // Response type. We ask ASP.NET to request an Auth Code, and an IDToken
   options.ResponseType = OpenIdConnectResponseType.CodeIdToken;

   // This "offline_access" scope is needed to get a refresh token when users sign in with
   // their Microsoft personal accounts
   // (it's required by MSAL.NET and automatically provided by Azure AD when users
   // sign in with work or school accounts, but not with their Microsoft personal accounts)
   options.Scope.Add(OidcConstants.ScopeOfflineAccess);
   options.Scope.Add("user.read"); // for instance

   // Handling the auth redemption by MSAL.NET so that a token is available in the token cache
   // where it will be usable from Controllers later (through the TokenAcquisition service)
   var handler = options.Events.OnAuthorizationCodeReceived;
   options.Events.OnAuthorizationCodeReceived = async context =>
   {
    // As AcquireTokenByAuthorizationCode is asynchronous we want to tell ASP.NET core
    // that we are handing the code even if it's not done yet, so that it does 
    // not concurrently call the Token endpoint.
    context.HandleCodeRedemption();

    // Call MSAL.NET AcquireTokenByAuthorizationCode
    var application = BuildConfidentialClientApplication(context.HttpContext,
                                                         context.Principal);
    var result = await application.AcquireTokenByAuthorizationCode(scopes.Except(scopesRequestedByMsalNet), context.ProtocolMessage.Code)
                                  .ExecuteAsync();

    // Do not share the access token with ASP.NET Core otherwise ASP.NET will cache it
    // and will not send the OAuth 2.0 request in case a further call to
    // AcquireTokenByAuthorizationCodeAsync in the future for incremental consent 
    // (getting a code requesting more scopes)
    // Share the ID Token so that the identity of the user is known in the application (in 
    // HttpContext.User)
    context.HandleCodeRedemption(null, result.IdToken);

    // Call the previous handler if any
    await handler(context);
   };
```

En ASP.NET Core, compilar la aplicación cliente confidencial usa la información que se encuentra en el elemento HttpContext. Este elemento HttpContext conoce la dirección URL de la aplicación Web y el usuario con sesión iniciada (en un `ClaimsPrincipal`). También usa la configuración de ASP.NET Core, que tiene una sección "AzureAD", y que se enlaza a la `_applicationOptions` estructura de datos. Por último, la aplicación debe mantener las cachés de tokens.

```CSharp
/// <summary>
/// Creates an MSAL Confidential client application
/// </summary>
/// <param name="httpContext">HttpContext associated with the OIDC response</param>
/// <param name="claimsPrincipal">Identity for the signed-in user</param>
/// <returns></returns>
private IConfidentialClientApplication BuildConfidentialClientApplication(HttpContext httpContext, ClaimsPrincipal claimsPrincipal)
{
 var request = httpContext.Request;

 // Find the URI of the application)
 string currentUri = UriHelper.BuildAbsolute(request.Scheme, request.Host, request.PathBase, azureAdOptions.CallbackPath ?? string.Empty);

 // Updates the authority from the instance (including national clouds) and the tenant
 string authority = $"{azureAdOptions.Instance}{azureAdOptions.TenantId}/";

 // Instantiates the application based on the application options (including the client secret)
 var app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(_applicationOptions)
               .WithRedirectUri(currentUri)
               .WithAuthority(authority)
               .Build();

 // Initialize token cache providers. In the case of Web applications, there must be one
 // token cache per user (here the key of the token cache is in the claimsPrincipal which
 // contains the identity of the signed-in user)
 if (this.UserTokenCacheProvider != null)
 {
  this.UserTokenCacheProvider.Initialize(app.UserTokenCache, httpContext, claimsPrincipal);
 }
 if (this.AppTokenCacheProvider != null)
 {
  this.AppTokenCacheProvider.Initialize(app.AppTokenCache, httpContext);
 }
 return app;
}
```

`AcquireTokenByAuthorizationCode` realmente canjea el código de autorización solicitado por ASP.NET y obtiene los tokens que se agregan a la caché de tokens de usuario MSAL.NET. Desde allí, a continuación, está utilizando, en los controladores de ASP.NET Core.

## <a name="aspnet-configuration"></a>Configuración de ASP.NET

ASP.NET controla las cosas de forma similar, salvo que la configuración de OpenIdConnect y la suscripción a la `OnAuthorizationCodeReceived` evento se produce en el `App_Start\Startup.Auth.cs` archivo. Encontrará conceptos similares, salvo que aquí deberá especificar la RedirectUri, en el archivo de configuración, que es un poco menos sólida:

```CSharp
private void ConfigureAuth(IAppBuilder app)
{
 app.SetDefaultSignInAsAuthenticationType(CookieAuthenticationDefaults.AuthenticationType);

 app.UseCookieAuthentication(new CookieAuthenticationOptions { });

 app.UseOpenIdConnectAuthentication(
 new OpenIdConnectAuthenticationOptions
 {
  Authority = Globals.Authority,
  ClientId = Globals.ClientId,
  RedirectUri = Globals.RedirectUri,
  PostLogoutRedirectUri = Globals.RedirectUri,
  Scope = Globals.BasicSignInScopes, // a basic set of permissions for user sign in & profile access
  TokenValidationParameters = new TokenValidationParameters
  {
  // We'll inject our own issuer validation logic below.
  ValidateIssuer = false,
  NameClaimType = "name",
  },
  Notifications = new OpenIdConnectAuthenticationNotifications()
  {
   AuthorizationCodeReceived = OnAuthorizationCodeReceived,
  }
 });
}
```

```CSharp
private async Task OnAuthorizationCodeReceived(AuthorizationCodeReceivedNotification context)
{
 // Upon successful sign in, get & cache a token using MSAL
 string userId = context.AuthenticationTicket.Identity.FindFirst(ClaimTypes.NameIdentifier).Value;
 IConfidentialClientApplication clientapp;
 clientApp = ConfidentialClientApplicationBuilder.Create(Globals.ClientId)
                  .WithClientSecret(Globals.ClientSecret)
                  .WithRedirectUri(Globals.RedirectUri)
                  .WithAuthority(new Uri(Globals.Authority))
                  .Build();

  EnablePersistence(HttpContext, clientApp.UserTokenCache, clientApp.AppTokenCache);

 AuthenticationResult result = await clientapp.AcquireTokenByAuthorizationCode(scopes, context.Code)
                                              .ExecuteAsync();
}
```

### <a name="msalnet-token-cache-for-a-aspnet-core-web-app"></a>Caché de Token MSAL.NET para una aplicación Web ASP.NET (núcleos)

En las aplicaciones web (o web API como de hecho), la implementación de caché de tokens es diferente de las implementaciones de caché de tokens de aplicaciones de escritorio (que a menudo son [basados en archivos](scenario-desktop-acquire-token.md#file-based-token-cache). Puede usar la sesión ASP.NET/ASP.NET Core, o una caché en Redis, una base de datos o incluso almacenamiento de blobs de Azure. En el código de fragmento de código encima de esto es el objeto de la `EnablePersistence(HttpContext, clientApp.UserTokenCache, clientApp.AppTokenCache);` llamada de método, que se enlaza a un servicio de caché. Los detalles de lo que sucede aquí está fuera del ámbito de esta guía del escenario, pero se proporcionan vínculos a continuación.

> [!IMPORTANT]
> Algo muy importante comprender es que para las aplicaciones web y API web, debe haber una caché de tokens por usuario (por cuenta). Es preciso serializar la caché de tokens de cada cuenta.

Ejemplos de cómo usar las cachés de tokens para aplicaciones Web y API web están disponibles en el [tutorial de aplicación Web de ASP.NET Core](https://github.com/Azure-Samples/ms-identity-aspnetcore-webapp-tutorial) en la fase [caché de tokens de 2-2](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/2-WebApp-graph-user/2-2-TokenCache). Para ver las implementaciones, diríjase a la siguiente carpeta [TokenCacheProviders](https://github.com/AzureAD/microsoft-authentication-extensions-for-dotnet/tree/master/src/Microsoft.Identity.Client.Extensions.Web/TokenCacheProviders) de la biblioteca [microsoft-authentication-extensions-for-dotnet](https://github.com/AzureAD/microsoft-authentication-extensions-for-dotnet) (en la carpeta [ Microsoft.Identity.Client.Extensions.Web](https://github.com/AzureAD/microsoft-authentication-extensions-for-dotnet/tree/master/src/Microsoft.Identity.Client.Extensions.Web) carpeta).

## <a name="next-steps"></a>Pasos siguientes

En este momento, cuando el usuario inicia sesión un token se almacena en la caché de tokens. Vamos a ver cómo se usa, a continuación, en otras partes de la aplicación Web.

> [!div class="nextstepaction"]
> [Inicie sesión en la aplicación Web](scenario-web-app-call-api-sign-in.md)
