---
title: Web API que web de bajada llamadas API (configuración de código de la aplicación), plataforma Microsoft identity
description: Obtenga información sobre cómo crear una web API que llama a web API (configuración de código de la aplicación)
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
ms.openlocfilehash: f62cf65e275d8a9b909bf60103ccbd84e91e4574
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65785054"
---
# <a name="web-api-that-calls-web-apis---code-configuration"></a>Web API que llama a web API: configuración de código

Una vez registrado su API web, puede configurar el código de la aplicación.

El código para configurar su API web para que llame a API web de bajada se basa en el código usado para proteger una API web. Para obtener más información, consulte [Protected web API: configuración de la aplicación](scenario-protected-web-api-app-configuration.md).

## <a name="code-subscribed-to-ontokenvalidated"></a>Código suscrito a OnTokenValidated

Encima de la configuración de código para cualquier API web protegidas, debe suscribirse a la validación del token de portador que se recibe cuando se llama a la API:

```CSharp
/// <summary>
/// Protects the web API with Microsoft Identity Platform v2.0 (AAD v2.0)
/// This supposes that the configuration files have a section named "AzureAD"
/// </summary>
/// <param name="services">Service collection to which to add authentication</param>
/// <param name="configuration">Configuration</param>
/// <returns></returns>
public static IServiceCollection AddProtectedApiCallsWebApis(this IServiceCollection services,
                                                             IConfiguration configuration,
                                                             IEnumerable<string> scopes)
{
    services.AddTokenAcquisition();
    services.Configure<JwtBearerOptions>(AzureADDefaults.JwtBearerAuthenticationScheme, options =>
    {
        // When an access token for our own web API is validated, we add it 
        // to MSAL.NET's cache so that it can be used from the controllers.
        options.Events = new JwtBearerEvents();

        options.Events.OnTokenValidated = async context =>
        {
            context.Success();

            // Adds the token to the cache, and also handles the incremental consent 
            // and claim challenges
            AddAccountToCacheFromJwt(context, scopes);
            await Task.FromResult(0);
        };
    });
    return services;
}
```

## <a name="on-behalf-of-flow"></a>Flujos en nombre de

El método AddAccountToCacheFromJwt() debe:

- Crear una instancia de una aplicación de cliente confidencial MSAL.
- Llame a `AcquireTokenOnBehalf` para intercambiar el token de portador que se ha adquirido por el cliente para la API web, con un token de portador para el mismo usuario, pero para nuestra API para llamar a una API de nivel inferior.

### <a name="instantiate-a-confidential-client-application"></a>Crear una instancia de una aplicación cliente confidencial

Este flujo solo está disponible en el flujo de cliente confidencial, por lo que la API web protegida proporciona las credenciales del cliente (secreto de cliente o certificado) para el [ConfidentialClientApplicationBuilder](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.confidentialclientapplicationbuilder) a través de la `WithClientSecret` o `WithCertificate`métodos, respectivamente.

![image](https://user-images.githubusercontent.com/13203188/55967244-3d8e1d00-5c7a-11e9-8285-a54b05597ec9.png)

```CSharp
IConfidentialClientApplication app;

#if !VariationWithCertificateCredentials
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
           .WithClientSecret(config.ClientSecret)
           .Build();
#else
// Building the client credentials from a certificate
X509Certificate2 certificate = ReadCertificate(config.CertificateName);
app = ConfidentialClientApplicationBuilder.Create(config.ClientId)
    .WithCertificate(certificate)
    .Build();
#endif
```

### <a name="how-to-call-on-behalf-of"></a>Cómo llamar a on-behalf-of

Se realiza la llamada en nombre de (OBO) mediante una llamada a la [AcquireTokenOnBehalf](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.acquiretokenonbehalfofparameterbuilder) método en el `IConfidentialClientApplication` interfaz.

El `ClientAssertion` se crea a partir del token de portador recibido por la API web desde sus propios clientes. Hay [dos constructores](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.clientcredential.-ctor?view=azure-dotnet), una que toma un portador JWT token y otra que toma cualquier tipo de aserción de usuario (otro tipo de token de seguridad, el tipo, a continuación, se especifica en un parámetro adicional llamado `assertionType`).

![image](https://user-images.githubusercontent.com/13203188/37082180-afc4b708-21e3-11e8-8af8-a6dcbd2dfba8.png)

En la práctica, el flujo OBO a menudo se usa para adquirir un token para una API de nivel inferior y almacenarlos en la caché de tokens de usuario MSAL.NET para que otras partes de la API web pueden llamar posteriormente en el [invalida](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.clientapplicationbase.acquiretokensilent?view=azure-dotnet) de ``AcquireTokenOnSilent`` para llamar a las API de nivel inferiores. Esto tiene el efecto de actualizar los tokens, si es necesario.

```CSharp
private void AddAccountToCacheFromJwt(IEnumerable<string> scopes, JwtSecurityToken jwtToken, ClaimsPrincipal principal, HttpContext httpContext)
{
 try
 {
  UserAssertion userAssertion;
  IEnumerable<string> requestedScopes;
  if (jwtToken != null)
  {
   userAssertion = new UserAssertion(jwtToken.RawData, "urn:ietf:params:oauth:grant-type:jwt-bearer");
   requestedScopes = scopes ?? jwtToken.Audiences.Select(a => $"{a}/.default");
  }
  else
  {
   throw new ArgumentOutOfRangeException("tokenValidationContext.SecurityToken should be a JWT Token");
  }

  // Create the application
  var application = BuildConfidentialClientApplication(httpContext, principal);

  // .Result to make sure that the cache is filled-in before the controller tries to get access tokens
  var result = application.AcquireTokenOnBehalfOf(requestedScopes.Except(scopesRequestedByMsalNet),
                                                  userAssertion)
                                        .ExecuteAsync()
                                        .GetAwaiter().GetResult();
 }
 catch (MsalException ex)
 {
  Debug.WriteLine(ex.Message);
  throw;
 }
}
```

## <a name="protocol"></a>Protocol

Para obtener más información acerca del protocolo on-behalf-of, vea [plataforma de identidad de Microsoft y flujo en nombre de OAuth 2.0](https://docs.microsoft.com/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Adquirir un token para la aplicación](scenario-web-api-call-api-acquire-token.md)
