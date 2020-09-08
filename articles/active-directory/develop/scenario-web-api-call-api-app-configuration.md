---
title: Configuración de una API web que llama a otras API web | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo compilar una API web que llama a otras API web (configuración del código de la aplicación)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 08/05/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: e9faea3462ae953e474b5053b651808b03f07c23
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855449"
---
# <a name="a-web-api-that-calls-web-apis-code-configuration"></a>API web que llama a API web: Configuración del código

Una vez registrada su API web, puede configurar el código de la aplicación.

El código que usa para configurar su API web para que llame a las API web de bajada se basa en el código usado para proteger una API web. Para más información, consulte [API web protegida: configuración de la aplicación](scenario-protected-web-api-app-configuration.md).

# <a name="aspnet-core"></a>[ASP.NET Core](#tab/aspnetcore)

## <a name="client-secrets-or-client-certificates"></a>Secretos de cliente o certificados de cliente

Dado que la API web ahora llama a una API web de nivel inferior, debe proporcionar un secreto de cliente o un certificado de cliente en el archivo *appsettings.json*.

```JSON
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-api-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    "TenantId": "common"
  
   // To call an API
   "ClientSecret": "[Copy the client secret added to the app from the Azure portal]",
   "ClientCertificates": [
  ]
 }
}
```

En lugar de un secreto de cliente, puede proporcionar un certificado de cliente. En el fragmento de código siguiente se muestra el uso de un certificado almacenado en Azure Key Vault.

```JSON
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-api-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    "TenantId": "common"
  
   // To call an API
   "ClientCertificates": [
      {
        "SourceType": "KeyVault",
        "KeyVaultUrl": "https://msidentitywebsamples.vault.azure.net",
        "KeyVaultCertificateName": "MicrosoftIdentitySamplesCert"
      }
  ]
 }
}
```

Microsoft.Identity.Web proporciona varias maneras de describir certificados, tanto mediante configuración como mediante código. Para obtener más información, consulte el [wiki de Microsoft.Identity.Web: uso de certificados](https://github.com/AzureAD/microsoft-identity-web/wiki/Using-certificates) en GitHub.

## <a name="startupcs"></a>Startup.cs

Con Microsoft.Identity.Web, si desea que la API web llame a las API web de nivel inferior, agregue la línea `.EnableTokenAcquisitionToCallDownstreamApi()` después de `.AddMicrosoftIdentityWebApi(Configuration)` y, después, elija una implementación de la caché de tokens, por ejemplo `.AddInMemoryTokenCaches()`, en *Startup.cs*:

```csharp
using Microsoft.Identity.Web;

public class Startup
{
  ...
  public void ConfigureServices(IServiceCollection services)
  {
   // ...
    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddMicrosoftIdentityWebApi(Configuration, "AzureAd")
                .EnableTokenAcquisitionToCallDownstreamApi()
                .AddInMemoryTokenCaches();
  // ...
  }
  // ...
}
```

Al igual que con las aplicaciones web, puede elegir varias implementaciones de la caché de tokens. Para obtener más información, consulte el [wiki de Microsoft.Identity.Web: Serialización de la caché de tokens](https://aka.ms/ms-id-web/token-cache-serialization) en GitHub.

# <a name="java"></a>[Java](#tab/java)

El flujo "en nombre de" (OBO) se usa para obtener un token para llamar a la API web de nivel inferior. En este flujo, la API web recibe un token de portador con permisos delegados de usuario de la aplicación cliente y, a continuación, intercambia este token por otro token de acceso para llamar a la API web de nivel inferior.

En el código siguiente se usa el valor `SecurityContextHolder` de Spring Security Framework en la API web para obtener el token de portador validado. A continuación, se usa la biblioteca de MSAL Java para obtener un token para la API de nivel inferior mediante la llamada a `acquireToken` con `OnBehalfOfParameters`. MSAL almacena en caché el token para que las siguientes llamadas a la API puedan usar `acquireTokenSilently` para obtener el token almacenado en caché.

```Java
@Component
class MsalAuthHelper {

    @Value("${security.oauth2.client.authority}")
    private String authority;

    @Value("${security.oauth2.client.client-id}")
    private String clientId;

    @Value("${security.oauth2.client.client-secret}")
    private String secret;

    @Autowired
    CacheManager cacheManager;

    private String getAuthToken(){
        String res = null;
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();

        if(authentication != null){
            res = ((OAuth2AuthenticationDetails) authentication.getDetails()).getTokenValue();
        }
        return res;
    }

    String getOboToken(String scope) throws MalformedURLException {
        String authToken = getAuthToken();

        ConfidentialClientApplication application =
                ConfidentialClientApplication.builder(clientId, ClientCredentialFactory.create(secret))
                        .authority(authority).build();

        String cacheKey = Hashing.sha256()
                .hashString(authToken, StandardCharsets.UTF_8).toString();

        String cachedTokens = cacheManager.getCache("tokens").get(cacheKey, String.class);
        if(cachedTokens != null){
            application.tokenCache().deserialize(cachedTokens);
        }

        IAuthenticationResult auth;
        SilentParameters silentParameters =
                SilentParameters.builder(Collections.singleton(scope))
                        .build();
        auth = application.acquireTokenSilently(silentParameters).join();

        if (auth == null){
            OnBehalfOfParameters parameters =
                    OnBehalfOfParameters.builder(Collections.singleton(scope),
                            new UserAssertion(authToken))
                            .build();

            auth = application.acquireToken(parameters).join();
        }

        cacheManager.getCache("tokens").put(cacheKey, application.tokenCache().serialize());

        return auth.accessToken();
    }
}
```

# <a name="python"></a>[Python](#tab/python)

El flujo "en nombre de" (OBO) se usa para obtener un token para llamar a la API web de nivel inferior. En este flujo, la API web recibe un token de portador con permisos delegados de usuario de la aplicación cliente y, a continuación, intercambia este token por otro token de acceso para llamar a la API web de nivel inferior.

Una API web de Python deberá usar algún middleware para validar el token de portador recibido del cliente. A continuación, la API web podrá obtener el token de acceso para la API de nivel inferior mediante la biblioteca de MSAL para Python al llamar al método [`acquire_token_on_behalf_of`](https://msal-python.readthedocs.io/en/latest/?badge=latest#msal.ConfidentialClientApplication.acquire_token_on_behalf_of). Para obtener un ejemplo del uso de esta API, consulte [Código de prueba para la biblioteca de autenticación de Microsoft para Python en GitHub](https://github.com/AzureAD/microsoft-authentication-library-for-python/blob/1.2.0/tests/test_e2e.py#L429-L472). Consulte también la explicación del [problema 53](https://github.com/AzureAD/microsoft-authentication-library-for-python/issues/53) en ese mismo repositorio para obtener un enfoque que omita la necesidad de una aplicación de nivel intermedio.

---

También puede ver un ejemplo de implementación del flujo con derechos delegados en [Node.js y Azure Functions](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-onbehalfof-azurefunctions/blob/master/MiddleTierAPI/MyHttpTrigger/index.js#L61).

## <a name="protocol"></a>Protocolo

Para más información acerca del protocolo OBO, consulte [Plataforma de identidad de Microsoft y flujo con derechos delegados de OAuth 2.0](./v2-oauth2-on-behalf-of-flow.md).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [API web que llama a las API web: adquisición de un token para la aplicación](scenario-web-api-call-api-acquire-token.md)