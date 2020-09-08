---
title: Configuración de aplicaciones de API web protegidas | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo compilar una API web protegida y configurar el código de la aplicación.
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 07/15/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 50de800c94bd0a65fafcff3ef6613d6f063a3797
ms.sourcegitcommit: b33c9ad17598d7e4d66fe11d511daa78b4b8b330
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88855485"
---
# <a name="protected-web-api-code-configuration"></a>API web protegida: Configuración del código

Para configurar el código de la API web protegida, debe comprender lo siguiente:

- Qué define las API como protegidas.
- Cómo configurar un token de portador.
- Cómo validar el token.

## <a name="what-defines-aspnet-and-aspnet-core-apis-as-protected"></a>¿Qué define que las API de ASP.NET o ASP.NET Core sean protegidas?

Al igual que las aplicaciones web, las API web de ASP.NET y ASP.NET Core están protegidas porque sus acciones de controlador llevan como prefijo el atributo **[Authorize]** . Solo se puede llamar a las acciones de controlador si se llama a la API con una identidad autorizada.

Tenga en cuenta las preguntas siguientes:

- Solo una aplicación puede llamar a una API web. ¿Cómo conoce la API web la identidad de la aplicación que le llama?
- Si la aplicación llama a la API en nombre de un usuario, ¿cuál es la identidad del usuario?

## <a name="bearer-token"></a>Token de portador

El token de portador que se establece en el encabezado cuando se llama a la aplicación contiene información sobre la identidad de la aplicación. También contiene información sobre el usuario a menos que la aplicación web acepte llamadas de servicio a servicio desde una aplicación de demonio.

Este es un ejemplo de código de C# que muestra a un cliente que llama a la API después de adquirir un token con la biblioteca de autenticación de Microsoft para .NET (MSAL.NET):

```csharp
var scopes = new[] {$"api://.../access_as_user"};
var result = await app.AcquireToken(scopes)
                      .ExecuteAsync();

httpClient = new HttpClient();
httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", result.AccessToken);

// Call the web API.
HttpResponseMessage response = await _httpClient.GetAsync(apiUri);
```

> [!IMPORTANT]
> Una aplicación cliente solicita el token de portador al punto de conexión de la plataforma de Microsoft Identity *para la API web*. La API web es la única aplicación que debe comprobar el token y ver las notificaciones que contiene. Las aplicaciones cliente nunca deben intentar inspeccionar las notificaciones de los tokens.
>
> En el futuro, la API web podría requerir que el token esté cifrado. Este requisito podría impedir el acceso a las aplicaciones cliente que pueden ver los tokens de acceso.

## <a name="jwtbearer-configuration"></a>Configuración de JwtBearer

En esta sección se describe cómo configurar un token de portador.

### <a name="config-file"></a>Archivo config

```Json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-api-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    /*
      You need specify the TenantId only if you want to accept access tokens from a single tenant
     (line-of-business app).
      Otherwise, you can leave them set to common.
      This can be:
      - A GUID (Tenant ID = Directory ID)
      - 'common' (any organization and personal accounts)
      - 'organizations' (any organization)
      - 'consumers' (Microsoft personal accounts)
    */
    "TenantId": "common"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

#### <a name="case-where-you-used-a-custom-app-id-uri-for-your-web-api"></a>Caso en el que usó un URI del identificador de aplicación personalizado para la API web

Si ha aceptado el URI del identificador de aplicación propuesto por el portal de registro de aplicaciones, no es necesario especificar el público (consulte [URI y ámbitos del identificador de aplicación](scenario-protected-web-api-app-registration.md#application-id-uri-and-scopes)). De lo contrario, debe agregar una propiedad `Audience` cuyo valor sea el URI del identificador de aplicación para la API web.

```Json
{
  "AzureAd": {
    "Instance": "https://login.microsoftonline.com/",
    "ClientId": "[Client_id-of-web-api-eg-2ec40e65-ba09-4853-bcde-bcb60029e596]",
    "TenantId": "common",
    "Audience": "custom App ID URI for your web API"
  },
  // more lines
}
```

### <a name="code-initialization"></a>Inicialización del código

Cuando se llama a una aplicación en una acción de controlador que contiene un atributo **[Authorize]** , ASP.NET y ASP.NET Core extraen el token de acceso del token de portador del encabezado de autorización. Después, el token se reenvía al middleware JwtBearer, que llama a las extensiones de Microsoft IdentityModel para .NET.

#### <a name="using-microsoftidentityweb-templates"></a>Uso de plantillas de Microsoft.Identity.Web

Puede crear una API web desde cero con plantillas del proyecto Microsoft.Identity.Web. Para obtener más información, consulte [Plantillas de proyecto de API web de Microsoft.Identity.Web](https://aka.ms/ms-id-web/webapi-project-templates).

#### <a name="starting-from-an-existing-aspnet-core-31-application"></a>A partir de una aplicación existente de ASP.NET Core 3.1

En la actualidad, ASP.NET Core 3.1 usa la biblioteca Microsoft.AspNetCoreAspNetCore.AzureAD.UI. Este middleware se inicializa en el archivo Startup.cs.

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
```

El middleware se agrega a la API web mediante esta instrucción:

```csharp
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
  services.AddAuthentication(AzureADDefaults.JwtBearerAuthenticationScheme)
          .AddAzureADBearer(options => Configuration.Bind("AzureAd", options));
}
```

 Actualmente, las plantillas de ASP.NET Core crean API web de Azure Active Directory (Azure AD) a las que se conectan usuarios dentro de su organización o de cualquier organización. No inician la sesión de los usuarios con cuentas personales. Sin embargo, puede cambiar las plantillas para usar el punto de conexión de la Plataforma de identidad de Microsoft mediante [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web), disponible como paquete NuGet, y reemplazar el código en *Startup.cs*:

```csharp
using Microsoft.Identity.Web;
```

```csharp
public void ConfigureServices(IServiceCollection services)
{
 // Adds Microsoft Identity platform (AAD v2.0) support to protect this API
 services.AddMicrosoftIdentityWebApiAuthentication(Configuration, "AzureAd");

 services.AddControllers();
}
```

también puede escribir lo siguiente (que es equivalente).

```csharp
public void ConfigureServices(IServiceCollection services)
{
 // Adds Microsoft Identity platform (AAD v2.0) support to protect this API
 services.AddMicrosoftIdentityWebApiAuthentication(Configuration, "AzureAd");

 services.AddControllers();
}
```

> [!NOTE]
> Si usa Microsoft.Identity.Web y no establece `Audience` en *appSettings.json*, se usa lo siguiente:
> -  `$"{ClientId}"` si ha establecido la [versión aceptada por el token de acceso](scenario-protected-web-api-app-registration.md#accepted-token-version) en `2` o para las API web de Azure AD B2C.
> - `$"api://{ClientId}` en todos los demás casos (para [tokens de acceso](access-tokens.md) v1.0).
> Para obtener información detallada, consulte [Código fuente](https://github.com/AzureAD/microsoft-identity-web/blob/d2ad0f5f830391a34175d48621a2c56011a45082/src/Microsoft.Identity.Web/Resource/RegisterValidAudience.cs#L70-L83) de Microsoft.Identity.Web.

El fragmento de código anterior se extrae del [Tutorial incremental de la API web de ASP.NET Core](https://github.com/Azure-Samples/active-directory-dotnet-native-aspnetcore-v2/blob/63087e83326e6a332d05fee6e1586b66d840b08f/1.%20Desktop%20app%20calls%20Web%20API/TodoListService/Startup.cs#L23-L28). El detalle de **AddMicrosoftWebApiAuthentication** está disponible en [Microsoft.Identity.Web](https://github.com/AzureAD/microsoft-identity-web/blob/d2ad0f5f830391a34175d48621a2c56011a45082/src/Microsoft.Identity.Web/WebApiExtensions/WebApiServiceCollectionExtensions.cs#L27). Este método llama a [AddMicrosoftWebAPI](https://github.com/AzureAD/microsoft-identity-web/blob/d2ad0f5f830391a34175d48621a2c56011a45082/src/Microsoft.Identity.Web/WebApiExtensions/WebApiAuthenticationBuilderExtensions.cs#L58), que, a su vez, indica al middleware cómo validar el token. Para obtener más información, consulte el [código fuente](https://github.com/AzureAD/microsoft-identity-web/blob/d2ad0f5f830391a34175d48621a2c56011a45082/src/Microsoft.Identity.Web/WebApiExtensions/WebApiAuthenticationBuilderExtensions.cs#L104-L122).

## <a name="token-validation"></a>Validación de tokens

En el fragmento de código anterior, el middleware JwtBearer, al igual que el middleware OpenID Connect en Web Apps, valida el token en función del valor de `TokenValidationParameters`. El token se descifra según sea necesario, se extraen las notificaciones y se comprueba la firma. Después, el middleware valida el token mediante la comprobación de estos datos:

- Audience: El token está destinado a la API web.
- Sub: se ha emitido para una aplicación que puede llamar a la API web.
- Emisor: lo ha emitido un servicio de token de seguridad (STS) de confianza.
- Expiry: su duración está dentro del intervalo.
- Signature: no se ha alterado.

También puede haber validaciones especiales. Por ejemplo, es posible validar que las claves de firma (cuando se insertan en un token) sean de confianza y que el token no se esté reproduciendo. Por último, algunos protocolos requieren validaciones específicas.

### <a name="validators"></a>Validadores

Los pasos de validación se capturan en los validadores, que se proporcionan en la biblioteca de código abierto [extensiones de Microsoft IdentityModel para .NET](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet). Los validadores se definen en el archivo de origen de biblioteca [Microsoft.IdentityModel.Tokens/Validators.cs](https://github.com/AzureAD/azure-activedirectory-identitymodel-extensions-for-dotnet/blob/master/src/Microsoft.IdentityModel.Tokens/Validators.cs).

En esta tabla se describen los validadores:

| Validador | Descripción |
|---------|---------|
| **ValidateAudience** | Garantiza que el token es para la aplicación que valida el token en su nombre. |
| **ValidateIssuer** | Garantiza que el token lo ha emitido un STS de confianza, lo que significa que se trata de un usuario de confianza. |
| **ValidateIssuerSigningKey** | Garantiza que la aplicación que valida el token confía en la clave que se ha usado para firmar el token. Hay un caso especial en el que la clave se inserta en el token. Este caso no es habitual. |
| **ValidateLifetime** | Garantiza que el token ya sea o siga siendo válido. El validador comprueba que la duración del token esté dentro del intervalo especificado por las notificaciones **notBefore** y **expires**. |
| **ValidateSignature** | Garantiza que el token no se ha alterado. |
| **ValidateTokenReplay** | Garantiza que el token no se ha reproducido. Hay un caso especial para algunos protocolos de un solo uso. |

#### <a name="customizing-token-validation"></a>Personalización de la validación de tokens

Los validadores se asocian con las propiedades de la clase **TokenValidationParameters**. Las propiedades se inicializan a partir de la configuración de ASP.NET y ASP.NET Core.

En la mayoría de los casos, no es necesario cambiar los parámetros. Las aplicaciones que no son de inquilino único son excepciones. Estas aplicaciones web aceptan usuarios de cualquier organización o de cuentas de Microsoft personales. En este caso, se deben validar los emisores. Microsoft.Identity.Web también se encarga de la validación del emisor. Para obtener información detallada, consulte [AadIssuerValidator](https://github.com/AzureAD/microsoft-identity-web/blob/master/src/Microsoft.Identity.Web/Resource/AadIssuerValidator.cs) de Microsoft.Identity.Web.

En ASP.NET Core, si desea personalizar los parámetros de validación de tokens, use el siguiente fragmento de código en *Startup.cs*:

```c#
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddMicrosoftIdentityWebApi(Configuration);
services.Configure<JwtBearerOptions>(JwtBearerDefaults.AuthenticationScheme, options =>
{
  var existingOnTokenValidatedHandler = options.Events.OnTokenValidated;
  options.Events.OnTokenValidated = async context =>
  {
       await existingOnTokenValidatedHandler(context);
      // Your code to add extra configuration that will be executed after the current event implementation.
      options.TokenValidationParameters.ValidIssuers = new[] { /* list of valid issuers */ };
      options.TokenValidationParameters.ValidAudiences = new[] { /* list of valid audiences */};
  }
});
```

En el caso de ASP.NET MVC, en el ejemplo de código siguiente se muestra cómo realizar la validación de tokens personalizada:

https://github.com/azure-samples/active-directory-dotnet-webapi-manual-jwt-validation

## <a name="token-validation-in-azure-functions"></a>Validación de tokens en Azure Functions

También puede validar los tokens de acceso entrantes en Azure Functions. Puede encontrar ejemplos de esta validación en los ejemplos de código siguientes en GitHub:

- .NET: [Azure-Samples/ms-identity-dotnet-webapi-azurefunctions](https://github.com/Azure-Samples/ms-identity-dotnet-webapi-azurefunctions)
- Node.js: [Azure-Samples/ms-identity-nodejs-webapi-azurefunctions](https://github.com/Azure-Samples/ms-identity-nodejs-webapi-azurefunctions)
- Python: [Azure-Samples/ms-identity-python-webapi-azurefunctions)](https://github.com/Azure-Samples/ms-identity-python-webapi-azurefunctions)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Comprobación de los ámbitos y roles de aplicación en el código](scenario-protected-web-api-verification-scope-app-roles.md)
