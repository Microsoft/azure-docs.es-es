---
title: Configuración de aplicaciones de escritorio que llaman a las API web - Plataforma de identidad de Microsoft | Azure
description: Aprenda a configurar el código de una aplicación de escritorio que llama a las API web.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 10/30/2019
ms.author: jmprieur
ms.custom: aaddev, devx-track-python
ms.openlocfilehash: 56d2e01d35f25ac469e808a805382463d2b23ceb
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88121109"
---
# <a name="desktop-app-that-calls-web-apis-code-configuration"></a>Aplicación de escritorio que llama a las API web: Configuración del código

Ahora que ha creado la aplicación, aprenderá a configurar el código con las coordenadas de la aplicación.

## <a name="microsoft-authentication-libraries"></a>Bibliotecas de autenticación de Microsoft

Las siguientes bibliotecas de autenticación de Microsoft (MSAL) admiten aplicaciones de escritorio.

  Biblioteca de autenticación de Microsoft | Descripción
  ------------ | ----------
  ![MSAL.NET](media/sample-v2-code/logo_NET.png) <br/> MSAL.NET  | Admite la creación de una aplicación de escritorio en varias plataformas, como Linux, Windows y macOS.
  ![Python](media/sample-v2-code/logo_python.png) <br/> Python de MSAL | Admite la creación de una aplicación de escritorio en varias plataformas.
  ![Java](media/sample-v2-code/logo_java.png) <br/> Java de MSAL | Admite la creación de una aplicación de escritorio en varias plataformas.
  ![MSAL iOS](media/sample-v2-code/logo_iOS.png) <br/> MSAL iOS | Admite aplicaciones de escritorio que solo se ejecutan en macOS.

## <a name="public-client-application"></a>Aplicación cliente pública

Desde un punto de vista del código, las aplicaciones de escritorio son aplicaciones cliente públicas. La configuración diferirá un poco en función de si se utiliza o no la autenticación interactiva.

# <a name="net"></a>[.NET](#tab/dotnet)

Deberá compilar y manipular MSAL.NET`IPublicClientApplication`.

![IPublicClientApplication](media/scenarios/public-client-application.png)

### <a name="exclusively-by-code"></a>Exclusivamente mediante código

El siguiente código crea una instancia de una aplicación cliente pública e inicia la sesión de los usuarios en la nube pública de Microsoft Azure con una cuenta profesional o educativa o bien con una cuenta Microsoft personal.

```csharp
IPublicClientApplication app = PublicClientApplicationBuilder.Create(clientId)
    .Build();
```

Si piensa utilizar la autenticación interactiva o el flujo de código del dispositivo, tal como se vio anteriormente, utilice el modificador `.WithRedirectUri`.

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithDefaultRedirectUri()
        .Build();
```

### <a name="use-configuration-files"></a>Uso de archivos de configuración

El siguiente código crea una instancia de una aplicación cliente pública a partir de un objeto de configuración, que podría rellenarse mediante programación o leerse desde un archivo de configuración.

```csharp
PublicClientApplicationOptions options = GetOptions(); // your own method
IPublicClientApplication app = PublicClientApplicationBuilder.CreateWithApplicationOptions(options)
        .WithDefaultRedirectUri()
        .Build();
```

### <a name="more-elaborated-configuration"></a>Configuración más elaborada

Puede elaborar la compilación de la aplicación agregando una serie de modificadores. Por ejemplo, si quiere que la aplicación sea una aplicación de varios inquilinos en una nube nacional, como en la Administración Pública de Estados Unidos que se muestra aquí, podría escribir lo siguiente:

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithDefaultRedirectUri()
        .WithAadAuthority(AzureCloudInstance.AzureUsGovernment,
                         AadAuthorityAudience.AzureAdMultipleOrgs)
        .Build();
```

MSAL.NET también contiene un modificador para Servicios de federación de Active Directory 2019:

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAdfsAuthority("https://consoso.com/adfs")
        .Build();
```

Por último, si quiere adquirir tokens para un inquilino de Azure Active Directory (Azure AD) B2C, especifique el inquilino tal como se muestra en el siguiente fragmento de código:

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithB2CAuthority("https://fabrikamb2c.b2clogin.com/tfp/{tenant}/{PolicySignInSignUp}")
        .Build();
```

### <a name="learn-more"></a>Más información

Para más información sobre cómo configurar una aplicación de escritorio de MSAL.NET:

- Para obtener una lista de todos los modificadores disponibles en `PublicClientApplicationBuilder`, consulte la documentación de referencia de [PublicClientApplicationBuilder](/dotnet/api/microsoft.identity.client.publicclientapplicationbuilder#methods).
- Para obtener la descripción de todas las opciones que se muestran en `PublicClientApplicationOptions`, consulte [PublicClientApplicationOptions](/dotnet/api/microsoft.identity.client.publicclientapplicationoptions) en la documentación de referencia.

### <a name="complete-example-with-configuration-options"></a>Ejemplo completo con opciones de configuración

Imagine una aplicación de consola .NET Core que tiene el siguiente archivo de configuración `appsettings.json`:

```json
{
  "Authentication": {
    "AzureCloudInstance": "AzurePublic",
    "AadAuthorityAudience": "AzureAdMultipleOrgs",
    "ClientId": "ebe2ab4d-12b3-4446-8480-5c3828d04c50"
  },

  "WebAPI": {
    "MicrosoftGraphBaseEndpoint": "https://graph.microsoft.com"
  }
}
```

Tiene algún código que leer en este archivo mediante el marco de configuración de .NET proporcionado:

```csharp
public class SampleConfiguration
{
 /// <summary>
 /// Authentication options
 /// </summary>
 public PublicClientApplicationOptions PublicClientApplicationOptions { get; set; }

 /// <summary>
 /// Base URL for Microsoft Graph (it varies depending on whether the application runs
 /// in Microsoft Azure public clouds or national or sovereign clouds)
 /// </summary>
 public string MicrosoftGraphBaseEndpoint { get; set; }

 /// <summary>
 /// Reads the configuration from a JSON file
 /// </summary>
 /// <param name="path">Path to the configuration json file</param>
 /// <returns>SampleConfiguration as read from the json file</returns>
 public static SampleConfiguration ReadFromJsonFile(string path)
 {
  // .NET configuration
  IConfigurationRoot Configuration;
  var builder = new ConfigurationBuilder()
                    .SetBasePath(Directory.GetCurrentDirectory())
                    .AddJsonFile(path);
  Configuration = builder.Build();

  // Read the auth and graph endpoint configuration
  SampleConfiguration config = new SampleConfiguration()
  {
   PublicClientApplicationOptions = new PublicClientApplicationOptions()
  };
  Configuration.Bind("Authentication", config.PublicClientApplicationOptions);
  config.MicrosoftGraphBaseEndpoint =
  Configuration.GetValue<string>("WebAPI:MicrosoftGraphBaseEndpoint");
  return config;
 }
}
```

Ahora, para crear la aplicación, escriba el siguiente código:

```csharp
SampleConfiguration config = SampleConfiguration.ReadFromJsonFile("appsettings.json");
var app = PublicClientApplicationBuilder.CreateWithApplicationOptions(config.PublicClientApplicationOptions)
           .WithDefaultRedirectUri()
           .Build();
```

Antes de llamar al método `.Build()` puede invalidar la configuración con llamadas a los métodos `.WithXXX`, tal como se mostró anteriormente.

# <a name="java"></a>[Java](#tab/java)

Esta es la clase que se usa en los ejemplos de desarrollo de Java de MSAL para configurar los ejemplos: [TestData](https://github.com/AzureAD/microsoft-authentication-library-for-java/blob/dev/src/samples/public-client/).

```Java
PublicClientApplication pca = PublicClientApplication.builder(CLIENT_ID)
        .authority(AUTHORITY)
        .build();
```

# <a name="python"></a>[Python](#tab/python)

```Python
config = json.load(open(sys.argv[1]))

app = msal.PublicClientApplication(
    config["client_id"], authority=config["authority"],
    # token_cache=...  # Default cache is in memory only.
                       # You can learn how to use SerializableTokenCache from
                       # https://msal-python.rtfd.io/en/latest/#msal.SerializableTokenCache
    )
```

# <a name="macos"></a>[macOS](#tab/macOS)

El siguiente código crea una instancia de una aplicación cliente pública e inicia la sesión de los usuarios en la nube pública de Microsoft Azure con una cuenta profesional o educativa o bien con una cuenta Microsoft personal.

### <a name="quick-configuration"></a>Configuración rápida

Objective-C:

```objc
NSError *msalError = nil;

MSALPublicClientApplicationConfig *config = [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"];
MSALPublicClientApplication *application = [[MSALPublicClientApplication alloc] initWithConfiguration:config error:&msalError];
```

Swift:
```swift
let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>")
if let application = try? MSALPublicClientApplication(configuration: config){ /* Use application */}
```

### <a name="more-elaborated-configuration"></a>Configuración más elaborada

Puede elaborar la compilación de la aplicación agregando una serie de modificadores. Por ejemplo, si quiere que la aplicación sea una aplicación de varios inquilinos en una nube nacional, como en la Administración Pública de Estados Unidos que se muestra aquí, podría escribir lo siguiente:

Objective-C:

```objc
MSALAADAuthority *aadAuthority =
                [[MSALAADAuthority alloc] initWithCloudInstance:MSALAzureUsGovernmentCloudInstance
                                                   audienceType:MSALAzureADMultipleOrgsAudience
                                                      rawTenant:nil
                                                          error:nil];

MSALPublicClientApplicationConfig *config =
                [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"
                                                                redirectUri:@"<your-redirect-uri-here>"
                                                                  authority:aadAuthority];

NSError *applicationError = nil;
MSALPublicClientApplication *application =
                [[MSALPublicClientApplication alloc] initWithConfiguration:config error:&applicationError];
```

Swift:

```swift
let authority = try? MSALAADAuthority(cloudInstance: .usGovernmentCloudInstance, audienceType: .azureADMultipleOrgsAudience, rawTenant: nil)

let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>", redirectUri: "<your-redirect-uri-here>", authority: authority)
if let application = try? MSALPublicClientApplication(configuration: config) { /* Use application */}
```
---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Aplicación de escritorio que llama a las API web: adquisición de un token](scenario-desktop-acquire-token.md)