---
title: Crear una instancia de una aplicación cliente confidencial con opciones (biblioteca de autenticación de Microsoft para. NET) | Azure
description: Obtenga información sobre cómo crear una instancia de una aplicación cliente confidencial con opciones de configuración mediante la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
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
ms.openlocfilehash: 6f935b1b2815501710444e3f921a157ba02e3215
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544082"
---
# <a name="instantiate-a-confidential-client-application-with-configuration-options-using-msalnet"></a>Crear una instancia de una aplicación cliente confidencial con opciones de configuración mediante MSAL.NET

En este artículo se describe cómo crear una instancia de un [aplicación cliente confidencial](msal-client-applications.md) mediante la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).  Se crea una instancia de la aplicación con las opciones de configuración definidas en un archivo de configuración.

Antes de inicializar una aplicación, primero deberá [registrar](quickstart-register-app.md) , para que la aplicación se puede integrar con la plataforma Microsoft identity. Después del registro, puede que necesite la información siguiente (que puede encontrarse en el portal de Azure):

- El identificador de cliente (es decir, una cadena que representa un GUID)
- La URL del proveedor de identidades (la instancia con nombre) y la audiencia de inicio de sesión para la aplicación. Estos dos parámetros se conocen colectivamente como la entidad.
- El identificador del inquilino si está escribiendo una aplicación de línea de negocio únicamente para su organización (también aplicación de inquilino único con nombre).
- El secreto de aplicación (cadena de secreto de cliente) o el certificado (del tipo X509Certificate2) si es una aplicación cliente confidencial.
- Para las aplicaciones web y, a veces, para las aplicaciones de cliente público (en particular cuando la aplicación debe usar a un agente), también habrá establecer redirectUri donde el proveedor de identidades pondremos en contacto con back su aplicación con los tokens de seguridad.

## <a name="configure-the-application-from-the-config-file"></a>Configurar la aplicación desde el archivo de configuración
El nombre de las propiedades de las opciones de MSAL.NET coincide con el nombre de las propiedades de la `AzureADOptions` en ASP.NET Core, por lo tanto no es necesario escribir ningún código de adherencia.

Se describe una configuración de aplicaciones de ASP.NET Core en un *appsettings.json* archivo:

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

A partir de MSAL.NET v3.x, puede configurar la aplicación cliente confidencial desde el archivo de configuración. Las clases relacionadas con la configuración de la aplicación se encuentran en el `Microsoft.Identity.Client.AppConfig` espacio de nombres.

En la clase donde desee configurar y crear una instancia de la aplicación, debe declarar un `ConfidentialClientApplicationOptions` objeto.  Enlazar la configuración leída desde el origen (incluido el archivo appconfig.json) a la instancia de las opciones de aplicación:

```csharp
using Microsoft.Identity.Client.AppConfig;

private ConfidentialClientApplicationOptions _applicationOptions;
_applicationOptions = new ConfidentialClientApplicationOptions();
configuration.Bind("AzureAD", _applicationOptions);
```

Esto permite que el contenido de la sección "AzureAD" de la *appsettings.json* archivo enlazar las propiedades correspondientes de la `ConfidentialClientApplicationOptions` objeto.  A continuación, compile un `ConfidentialClientApplication` objeto:

```csharp
IConfidentialClientApplication app;
app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(_applicationOptions)
        .Build();
```

## <a name="add-runtime-configuration"></a>Agregar configuración de tiempo de ejecución
En una aplicación cliente confidencial, normalmente dispone de una memoria caché por usuario. Por lo tanto, deberá obtener la memoria caché asociada con el usuario e informar al generador de aplicaciones que desea usar. En la misma manera, podría tener un URI de redirección calculado dinámicamente. En este caso, el código es el siguiente:

```csharp
IConfidentialClientApplication app;
var request = httpContext.Request;
var currentUri = UriHelper.BuildAbsolute(request.Scheme, request.Host, request.PathBase, _azureAdOptions.CallbackPath ?? string.Empty);
app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(_applicationOptions)
       .WithRedirectUri(currentUri)
       .Build();
TokenCache userTokenCache = _tokenCacheProvider.SerializeCache(app.UserTokenCache,httpContext, claimsPrincipal);
```

