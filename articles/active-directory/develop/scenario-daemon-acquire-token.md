---
title: Adquisición de tokens para llamar a una API web (aplicación de demonio) - Plataforma de identidad de Microsoft | Azure
description: Obtenga información sobre cómo compilar una aplicación de demonio que llame a las API web (adquisición de tokens)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/30/2019
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 13000c5a61dc2c4d49aa395271beddef64d32245
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88119222"
---
# <a name="daemon-app-that-calls-web-apis---acquire-a-token"></a>Aplicación de demonio que llama a las API web: adquisición de un token

Después de que se ha construido una aplicación cliente confidencial, puede llamar a `AcquireTokenForClient` y pasar el ámbito y, opcionalmente, forzar una actualización del token para adquirir un token para la aplicación.

## <a name="scopes-to-request"></a>Solicitud de ámbitos

El ámbito que va a solicitar para un flujo de credenciales de cliente es el nombre del recurso seguido de `/.default`. Esta notación indica a Azure Active Directory (Azure AD) que use los *permisos de nivel de aplicación* declarados estáticamente durante el registro de la aplicación. Además, un administrador de inquilinos debe conceder estos permisos de API.

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
ResourceId = "someAppIDURI";
var scopes = new [] {  ResourceId+"/.default"};
```

# <a name="python"></a>[Python](#tab/python)

En MSAL Python, el archivo de configuración es similar a este fragmento de código:

```Json
{
    "scope": ["https://graph.microsoft.com/.default"],
}
```

# <a name="java"></a>[Java](#tab/java)

```Java
final static String GRAPH_DEFAULT_SCOPE = "https://graph.microsoft.com/.default";
```

---

### <a name="azure-ad-v10-resources"></a>Recursos de Azure AD (v1.0)

El ámbito usado para las credenciales del cliente siempre debe ser el identificador del recurso seguido de `/.default`.

> [!IMPORTANT]
> Cuando MSAL solicita un token de acceso para un recurso que acepta tokens de acceso de la versión 1.0, Azure AD analiza la audiencia deseada del ámbito solicitado, para lo cual toma todo lo que hay delante de la última barra diagonal y lo usa como identificador del recurso.
> Por tanto si, al igual que Azure SQL Database (**https:\//database.windows.net**), el recuso espera una audiencia que finaliza con una barra diagonal (en el caso de Azure SQL Database, `https://database.windows.net/`), deberá solicitar un ámbito de `https://database.windows.net//.default`. (Tenga en cuenta la barra diagonal doble). Consulte también el problema de MSAL.NET [747: falta la barra diagonal final de la dirección URL del recurso, lo que ha provocado un error de autenticación de SQL](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/issues/747)

## <a name="acquiretokenforclient-api"></a>API AcquireTokenForClient

Para adquirir un token para la aplicación, usará `AcquireTokenForClient` o el equivalente, según la plataforma.

# <a name="net"></a>[.NET](#tab/dotnet)

```csharp
using Microsoft.Identity.Client;

// With client credentials flows, the scope is always of the shape "resource/.default" because the
// application permissions need to be set statically (in the portal or by PowerShell), and then granted by
// a tenant administrator.
string[] scopes = new string[] { "https://graph.microsoft.com/.default" };

AuthenticationResult result = null;
try
{
 result = await app.AcquireTokenForClient(scopes)
                  .ExecuteAsync();
}
catch (MsalUiRequiredException ex)
{
    // The application doesn't have sufficient permissions.
    // - Did you declare enough app permissions during app creation?
    // - Did the tenant admin grant permissions to the application?
}
catch (MsalServiceException ex) when (ex.Message.Contains("AADSTS70011"))
{
    // Invalid scope. The scope has to be in the form "https://resourceurl/.default"
    // Mitigation: Change the scope to be as expected.
}
```

# <a name="python"></a>[Python](#tab/python)

```Python
# The pattern to acquire a token looks like this.
result = None

# First, the code looks up a token from the cache.
# Because we're looking for a token for the current app, not for a user,
# use None for the account parameter.
result = app.acquire_token_silent(config["scope"], account=None)

if not result:
    logging.info("No suitable token exists in cache. Let's get a new one from AAD.")
    result = app.acquire_token_for_client(scopes=config["scope"])

if "access_token" in result:
    # Call a protected API with the access token.
    print(result["token_type"])
else:
    print(result.get("error"))
    print(result.get("error_description"))
    print(result.get("correlation_id"))  # You might need this when reporting a bug.
```

# <a name="java"></a>[Java](#tab/java)

Este código se extrae de los [ejemplos de desarrollo de Java de MSAL](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/confidential-client/).

```Java
private static IAuthenticationResult acquireToken() throws Exception {

     // Load token cache from file and initialize token cache aspect. The token cache will have
     // dummy data, so the acquireTokenSilently call will fail.
     TokenCacheAspect tokenCacheAspect = new TokenCacheAspect("sample_cache.json");

     IClientCredential credential = ClientCredentialFactory.createFromSecret(CLIENT_SECRET);
     ConfidentialClientApplication cca =
             ConfidentialClientApplication
                     .builder(CLIENT_ID, credential)
                     .authority(AUTHORITY)
                     .setTokenCacheAccessAspect(tokenCacheAspect)
                     .build();

     IAuthenticationResult result;
     try {
         SilentParameters silentParameters =
                 SilentParameters
                         .builder(SCOPE)
                         .build();

         // try to acquire token silently. This call will fail since the token cache does not
         // have a token for the application you are requesting an access token for
         result = cca.acquireTokenSilently(silentParameters).join();
     } catch (Exception ex) {
         if (ex.getCause() instanceof MsalException) {

             ClientCredentialParameters parameters =
                     ClientCredentialParameters
                             .builder(SCOPE)
                             .build();

             // Try to acquire a token. If successful, you should see
             // the token information printed out to console
             result = cca.acquireToken(parameters).join();
         } else {
             // Handle other exceptions accordingly
             throw ex;
         }
     }
     return result;
 }
```

---

### <a name="protocol"></a>Protocolo

Si no tiene todavía una biblioteca para el lenguaje elegido, es posible que quiera usar el protocolo directamente:

#### <a name="first-case-access-the-token-request-by-using-a-shared-secret"></a>Primer caso: Acceso a la solicitud de token mediante un secreto compartido

```HTTP
POST /{tenant}/oauth2/v2.0/token HTTP/1.1           //Line breaks for clarity.
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

client_id=535fb089-9ff3-47b6-9bfb-4f1264799865
&scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_secret=qWgdYAmab0YSkuL1qKv5bPX
&grant_type=client_credentials
```

#### <a name="second-case-access-the-token-request-by-using-a-certificate"></a>Segundo caso: Acceso a la solicitud de token mediante un certificado

```HTTP
POST /{tenant}/oauth2/v2.0/token HTTP/1.1               // Line breaks for clarity.
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded

scope=https%3A%2F%2Fgraph.microsoft.com%2F.default
&client_id=97e0a5b7-d745-40b6-94fe-5f77d35c6e05
&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer
&client_assertion=eyJhbGciOiJSUzI1NiIsIng1dCI6Imd4OHRHeXN5amNScUtqRlBuZDdSRnd2d1pJMCJ9.eyJ{a lot of characters here}M8U3bSUKKJDEg
&grant_type=client_credentials
```

Para más información, consulte la documentación del protocolo: [La Plataforma de identidad de Microsoft y el flujo de credenciales de cliente de OAuth 2.0](v2-oauth2-client-creds-grant-flow.md)

## <a name="application-token-cache"></a>Caché de tokens de aplicación

En MSAL.NET, `AcquireTokenForClient` usa la caché de tokens de aplicación. (Todos los demás métodos AcquireToken*XX* usan la caché de tokens de usuario). No llame a `AcquireTokenSilent` antes de llamar a `AcquireTokenForClient`, ya que `AcquireTokenSilent` usa la caché de tokens de *usuario*. `AcquireTokenForClient` comprueba la propia caché de tokens de *aplicación* y la actualiza.

## <a name="troubleshooting"></a>Solución de problemas

### <a name="did-you-use-the-resourcedefault-scope"></a>¿Ha usado el ámbito de recurso/.default?

Si recibe un mensaje de error que indica que ha usado un ámbito no válido, probablemente no ha usado el ámbito `resource/.default`.

### <a name="did-you-forget-to-provide-admin-consent-daemon-apps-need-it"></a>¿Olvidó proporcionar el consentimiento del administrador? Las aplicaciones de demonio lo necesitan.

Si se produce un error **No tiene privilegios suficientes para completar la operación** al llamar a la API, el administrador de inquilinos debe conceder permisos a la aplicación. Consulte el paso 6 del registro de la aplicación de cliente anterior.
Normalmente verá un error similar al siguiente:

```json
Failed to call the web API: Forbidden
Content: {
  "error": {
    "code": "Authorization_RequestDenied",
    "message": "Insufficient privileges to complete the operation.",
    "innerError": {
      "request-id": "<guid>",
      "date": "<date>"
    }
  }
}
```

## <a name="next-steps"></a>Pasos siguientes

# <a name="net"></a>[.NET](#tab/dotnet)

> [!div class="nextstepaction"]
> [Llamada de demonio: llamar a una API web](./scenario-daemon-call-api.md?tabs=dotnet)

# <a name="python"></a>[Python](#tab/python)

> [!div class="nextstepaction"]
> [Llamada de demonio: llamar a una API web](./scenario-daemon-call-api.md?tabs=python)

# <a name="java"></a>[Java](#tab/java)

> [!div class="nextstepaction"]
> [Llamada de demonio: llamar a una API web](./scenario-daemon-call-api.md?tabs=java)

---