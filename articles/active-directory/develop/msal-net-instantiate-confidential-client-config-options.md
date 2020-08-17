---
title: Creación de una instancia de una aplicación cliente confidencial (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo crear una instancia de una aplicación cliente confidencial con opciones de configuración mediante la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
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
ms.openlocfilehash: 140385a26dbfd754438d632ce327c3c4a3ea1b61
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88166185"
---
# <a name="instantiate-a-confidential-client-application-with-configuration-options-using-msalnet"></a>Creación de una instancia de una aplicación cliente confidencial con opciones de configuración mediante MSAL.NET

En este artículo se describe cómo crear una instancia de una [aplicación cliente confidencial](msal-client-applications.md) mediante la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET).  La instancia de la aplicación se crea con opciones de configuración definidas en un archivo de configuración.

Antes de inicializar una aplicación, primero tendrá que [registrarla](quickstart-register-app.md) para que se pueda integrar con la plataforma de identidad de Microsoft. Después del registro, es posible que necesite la información siguiente (que puede encontrar en Azure Portal):

- El identificador de cliente (una cadena que representa un GUID).
- La URL del proveedor de identidades (la instancia) y la audiencia de inicio de sesión para la aplicación. De forma conjunta, estos dos parámetros se conocen como la autoridad.
- El identificador del inquilino si va a escribir una aplicación de línea de negocio exclusivamente para la organización (también denominada aplicación de un único inquilino).
- El secreto de aplicación (cadena de secreto de cliente) o el certificado (del tipo X509Certificate2) si se trata de una aplicación cliente confidencial.
- Para las aplicaciones web y, a veces, para las aplicaciones cliente públicas (concretamente cuando la aplicación debe usar a un agente), también habrá establecido la redirectUri donde el proveedor de identidades contactará con la aplicación mediante los tokens de seguridad.

## <a name="configure-the-application-from-the-config-file"></a>Configuración de la aplicación a partir del archivo de configuración
El nombre de las propiedades de las opciones de MSAL.NET coincide con el nombre de las propiedades de la `AzureADOptions` en ASP.NET Core, por lo que no tendrá que escribir código de adherencia.

La configuración de una aplicación de ASP.NET Core se describe en un archivo *appsettings.json*:

```json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "Domain": "[Enter the domain of your tenant, e.g. contoso.onmicrosoft.com]",
    "TenantId": "[Enter 'common', or 'organizations' or the Tenant Id (Obtained from the Azure portal. Select 'Endpoints' from the 'App registrations' blade and use the GUID in any of the URLs), e.g. da41245a5-11b3-996c-00a8-4d99re19f292]",
    "ClientId": "[Enter the Client Id (Application ID obtained from the Azure portal), e.g. ba74781c2-53c2-442a-97c2-3d60re42f403]",
    "CallbackPath": "/signin-oidc",
    "SignedOutCallbackPath ": "/signout-callback-oidc",

    "ClientSecret": "[Copy the client secret added to the app from the Azure portal]"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

A partir de MSAL.NET v3.x, puede configurar la aplicación cliente confidencial a partir del archivo de configuración.

En la clase en la que desee configurar y crear la instancia de la aplicación, deberá declarar un objeto `ConfidentialClientApplicationOptions`.  Enlace la configuración leída desde el origen (incluido el archivo appconfig.json) a la instancia de las opciones de la aplicación, mediante el método `IConfigurationRoot.Bind()` desde el [paquete de NuGet Microsoft.Extensions.Configuration.Binder](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Binder):

```csharp
using Microsoft.Identity.Client;

private ConfidentialClientApplicationOptions _applicationOptions;
_applicationOptions = new ConfidentialClientApplicationOptions();
configuration.Bind("AzureAD", _applicationOptions);
```

Esto permite que el contenido de la sección "AzureAD" del archivo *appsettings.json* esté enlazado con las propiedades correspondientes del objeto `ConfidentialClientApplicationOptions`.  A continuación, compile un objeto `ConfidentialClientApplication`:

```csharp
IConfidentialClientApplication app;
app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(_applicationOptions)
        .Build();
```

## <a name="add-runtime-configuration"></a>Adición de la configuración de ejecución
En una aplicación cliente confidencial, normalmente dispone de una caché por usuario. Por lo tanto, deberá obtener la caché asociada con el usuario e informar al generador de aplicaciones de que desea usarla. Asimismo, es posible que tenga un URI de redirección calculado dinámicamente. En este caso, el código es el siguiente:

```csharp
IConfidentialClientApplication app;
var request = httpContext.Request;
var currentUri = UriHelper.BuildAbsolute(request.Scheme, request.Host, request.PathBase, _azureAdOptions.CallbackPath ?? string.Empty);
app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(_applicationOptions)
       .WithRedirectUri(currentUri)
       .Build();
TokenCache userTokenCache = _tokenCacheProvider.SerializeCache(app.UserTokenCache,httpContext, claimsPrincipal);
```

