---
title: Crear una instancia de una aplicación de cliente público con opciones (biblioteca de autenticación de Microsoft para. NET) | Azure
description: Obtenga información sobre cómo crear una instancia de una aplicación cliente pública con las opciones de configuración mediante la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
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
ms.date: 04/30/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 125bbf9aed54fb00f039aeffddd5cc1aad3360a6
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544403"
---
# <a name="instantiate-a-public-client-application-with-configuration-options-using-msalnet"></a>Crear una instancia de una aplicación cliente pública con las opciones de configuración mediante MSAL.NET

En este artículo se describe cómo crear una instancia de un [aplicación cliente pública](msal-client-applications.md) mediante la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).  Se crea una instancia de la aplicación con las opciones de configuración definidas en un archivo de configuración.

Antes de inicializar una aplicación, primero deberá [registrar](quickstart-register-app.md) , para que la aplicación se puede integrar con la plataforma Microsoft identity. Después del registro, puede que necesite la información siguiente (que puede encontrarse en el portal de Azure):

- El identificador de cliente (es decir, una cadena que representa un GUID)
- La URL del proveedor de identidades (la instancia con nombre) y la audiencia de inicio de sesión para la aplicación. Estos dos parámetros se conocen colectivamente como la entidad.
- El identificador del inquilino si está escribiendo una aplicación de línea de negocio únicamente para su organización (también aplicación de inquilino único con nombre).
- Para las aplicaciones web y, a veces, para las aplicaciones de cliente público (en particular cuando la aplicación debe usar a un agente), también habrá establecer redirectUri donde el proveedor de identidades pondremos en contacto con back su aplicación con los tokens de seguridad.


Una aplicación de consola .NET Core podría tener el siguiente *appsettings.json* archivo de configuración:

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

El siguiente código lee este archivo mediante la configuración de .NET framework:

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

