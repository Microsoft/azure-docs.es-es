---
title: Creación de una instancia para una aplicación cliente pública (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo crear una instancia de una aplicación cliente pública con opciones de configuración mediante la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 04/30/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 349ec8c99bc9e6768e5ec2ded346893da95c1636
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88166151"
---
# <a name="instantiate-a-public-client-application-with-configuration-options-using-msalnet"></a>Crear una instancia de una aplicación cliente pública con opciones de configuración mediante MSAL.NET

En este artículo se describe cómo crear una instancia de una [aplicación cliente pública](msal-client-applications.md) mediante la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET).  La instancia de la aplicación se crea con opciones de configuración definidas en un archivo de configuración.

Antes de inicializar una aplicación, primero tendrá que [registrarla](quickstart-register-app.md) para que se pueda integrar con la plataforma de identidad de Microsoft. Después del registro, es posible que necesite la información siguiente (que puede encontrar en Azure Portal):

- El identificador de cliente (una cadena que representa un GUID).
- La URL del proveedor de identidades (la instancia) y la audiencia de inicio de sesión para la aplicación. De forma conjunta, estos dos parámetros se conocen como la autoridad.
- El identificador del inquilino si va a escribir una aplicación de línea de negocio exclusivamente para la organización (también denominada aplicación de un único inquilino).
- Para las aplicaciones web y, a veces, para las aplicaciones cliente públicas (concretamente cuando la aplicación debe usar a un agente), también habrá establecido la redirectUri donde el proveedor de identidades contactará con la aplicación mediante los tokens de seguridad.


Una aplicación de consola .NET Core podría tener el siguiente archivo de configuración *appsettings.json*:

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

El siguiente código lee este archivo mediante el marco de configuración de .NET:

```csharp
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
        config.MicrosoftGraphBaseEndpoint = Configuration.GetValue<string>("WebAPI:MicrosoftGraphBaseEndpoint");
        return config;
    }
}
```

El código siguiente crea la aplicación con la configuración del archivo de configuración:

```csharp
SampleConfiguration config = SampleConfiguration.ReadFromJsonFile("appsettings.json");
var app = PublicClientApplicationBuilder.CreateWithApplicationOptions(config.PublicClientApplicationOptions)
           .Build();
```

