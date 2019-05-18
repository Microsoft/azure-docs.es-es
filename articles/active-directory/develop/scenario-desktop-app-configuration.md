---
title: Aplicación de escritorio que llama a web API (configuración de código), plataforma Microsoft identity
description: Aprenda a crear una aplicación de escritorio que llama a web API (configuración de código de la aplicación)
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
ms.date: 05/o7/2019
ms.author: jmprieur
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: eadcae393128d9721f2c988f713af07913c5fd1d
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545499"
---
# <a name="desktop-app-that-calls-web-apis---code-configuration"></a>Aplicación de escritorio que llama a web API: configuración de código

Ahora que ha creado la aplicación, obtendrá información sobre cómo configurar el código con las coordenadas de la aplicación.

## <a name="msal-libraries"></a>Bibliotecas MSAL

La única biblioteca MSAL que admiten aplicaciones de escritorio de hoy en día es MSAL.NET

## <a name="public-client-application"></a>Aplicación de cliente público

Desde un punto de vista de código, las aplicaciones de escritorio son aplicaciones de cliente público, y por eso creará y manipular MSAL.NET `IPublicClientApplication`. Vuelva a cosas será un poco diferente si se utiliza la autenticación interactiva o no.

![IPublicClientApplication](media/scenarios/public-client-application.png)

### <a name="exclusively-by-code"></a>Exclusivamente mediante código

El código siguiente crea una instancia de una aplicación de cliente público, los usuarios iniciar sesión en la nube pública de Microsoft Azure, con su trabajo y cuentas educativas o sus cuentas personales de Microsoft.

```CSharp
IPublicClientApplication app = PublicClientApplicationBuilder.Create(clientId)
    .Build();
```

Si piensa utilizar la autenticación interactiva, tal como se muestra anteriormente, que desea utilizar el `.WithRedirectUri` modificador:

```CSharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithRedirectUri(PublicClientApplicationBuilder.DefaultInteractiveDesktopRedirectUri)
         .Build();
```

### <a name="using-configuration-files"></a>Uso de archivos de configuración

El código siguiente crea una instancia de una aplicación cliente pública desde un objeto de configuración que podría rellenarse de mediante programación o leer desde un archivo de configuración

```CSharp
PublicClientApplicationOptions options = GetOptions(); // your own method
IPublicClientApplication app = PublicClientApplicationBuilder.CreateWithApplicationOptions(options)
    .Build();
```

### <a name="more-elaborated-configuration"></a>Configuración más elaborado

Puede proporcionar la información de la aplicación creando mediante la adición de un número de modificadores. Por ejemplo, si desea que la aplicación para que una aplicación de varios inquilinos en una nube nacional (aquí gobierno de Estados Unidos), podría escribir:

```CSharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAadAuthority(AzureCloudInstance.AzureUsGovernment,
                         AadAuthorityAudience.AzureAdMultipleOrgs)
        .Build();
```

MSAL.NET también contiene un modificador de 2019 ADFS:

```CSharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAdfsAuthority("https://consoso.com/adfs")
        .Build();
```

Por último, si desea adquirir tokens para un inquilino de Azure AD B2C, puede especificar al inquilino tal como se muestra en el siguiente fragmento de código:

```CSharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithB2CAuthority("https://fabrikamb2c.b2clogin.com/tfp/{tenant}/{PolicySignInSignUp}")
        .Build();
```

### <a name="learn-more"></a>Obtenga más información

Para más información sobre cómo configurar una aplicación de escritorio MSAL.NET:

- Para obtener la lista de todos los modificadores disponibles en `PublicClientApplicationBuilder`, consulte la documentación de referencia [PublicClientApplicationBuilder](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.publicclientapplicationbuilder#methods)
- Para la descripción de todas las opciones que se muestran en `PublicClientApplicationOptions` vea [PublicClientApplicationOptions](https://docs.microsoft.com/dotnet/api/microsoft.identity.client.publicclientapplicationoptions), en la documentación de referencia

## <a name="complete-example-with-configuration-options"></a>Ejemplo completo con las opciones de configuración

Imagine una aplicación de consola .NET Core que tiene las siguientes `appsettings.json` archivo de configuración:

```JSon
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

Tiene poco código para leer este archivo mediante .NET proporciona el marco de configuración;

```CSharp
public class SampleConfiguration
{
 /// <summary>
 /// Authentication options
 /// </summary>
 public PublicClientApplicationOptions PublicClientApplicationOptions { get; set; }

 /// <summary>
 /// Base URL for Microsoft Graph (it varies depending on whether the application is ran
 /// in Microsoft Azure public clouds or national / sovereign clouds
 /// </summary>
 public string MicrosoftGraphBaseEndpoint { get; set; }

 /// <summary>
 /// Reads the configuration from a json file
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

  // Read the auth and graph endpoint config
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

Ahora, para crear la aplicación, simplemente necesita escribir el código siguiente:

```CSharp
SampleConfiguration config = SampleConfiguration.ReadFromJsonFile("appsettings.json");
var app = PublicClientApplicationBuilder.CreateWithApplicationOptions(config.PublicClientApplicationOptions)
           .Build();
```

y antes de llamar a la `.Build()` método, puede invalidar la configuración con las llamadas a `.WithXXX` métodos tal como se muestra anteriormente.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Adquirir un token para una aplicación de escritorio](scenario-desktop-acquire-token.md)
