---
title: Inicializar las aplicaciones cliente (biblioteca de autenticación de Microsoft para. NET) | Azure
description: Obtenga información sobre la inicialización de cliente público y las aplicaciones de cliente confidencial mediante la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
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
ms.date: 04/12/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 2f22ff41e380a16af2aa45df9a61eefbf293ff83
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544314"
---
# <a name="initialize-client-applications-using-msalnet"></a>Inicializar las aplicaciones de cliente mediante MSAL.NET
Este artículo describe la inicialización de cliente público y las aplicaciones de cliente confidencial mediante la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).  Para obtener más información sobre los tipos de aplicaciones cliente y las opciones de configuración de aplicación, lea el [Introducción](msal-client-applications.md).

Con MSAL.NET 3.x, que es el método recomendado para crear una instancia de una aplicación mediante el uso de los generadores de aplicación: `PublicClientApplicationBuilder` y `ConfidentialClientApplicationBuilder`. Ofrecen un mecanismo eficaz para configurar la aplicación desde el código, o desde un archivo de configuración o incluso mediante la combinación de ambos enfoques.

## <a name="prerequisites"></a>Requisitos previos
Antes de inicializar una aplicación, primero deberá [registrarlo](quickstart-register-app.md) para que la aplicación se puede integrar con la plataforma Microsoft identity.  Después del registro, puede que necesite la información siguiente (que puede encontrarse en el portal de Azure):

- El identificador de cliente (es decir, una cadena que representa un GUID)
- La URL del proveedor de identidades (la instancia con nombre) y la audiencia de inicio de sesión para la aplicación. Estos dos parámetros se conocen colectivamente como la entidad.
- El identificador del inquilino si está escribiendo una aplicación de línea de negocio únicamente para su organización (también aplicación de inquilino único con nombre).
- El secreto de aplicación (cadena de secreto de cliente) o el certificado (del tipo X509Certificate2) si es una aplicación cliente confidencial.
- Para las aplicaciones web y, a veces, para las aplicaciones de cliente público (en particular cuando la aplicación debe usar a un agente), también habrá establecer redirectUri donde el proveedor de identidades pondremos en contacto con back su aplicación con los tokens de seguridad.

## <a name="ways-to-initialize-applications"></a>Formas de inicializar aplicaciones
Hay muchas maneras diferentes para crear instancias de las aplicaciones cliente.

### <a name="initializing-a-public-client-application-from-code"></a>Inicializar una aplicación cliente pública desde el código

El código siguiente crea una instancia de una aplicación de cliente público, los usuarios iniciar sesión en la nube pública de Microsoft Azure, con su trabajo y cuentas educativas o sus cuentas personales de Microsoft.

```csharp
IPublicClientApplication app = PublicClientApplicationBuilder.Create(clientId)
    .Build();
```

### <a name="initializing-a-confidential-client-application-from-code"></a>Inicializar una aplicación cliente confidencial desde el código

En la misma manera, el código siguiente crea una instancia de una aplicación confidencial (una aplicación Web ubicado en `https://myapp.azurewebsites.net`) controla los tokens de los usuarios en la nube pública de Microsoft Azure, con su trabajo y cuentas educativas o sus cuentas personales de Microsoft. La aplicación se identifica con el proveedor de identidades mediante el uso compartido de un secreto de cliente:

```csharp
string redirectUri = "https://myapp.azurewebsites.net";
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithClientSecret(clientSecret)
    .WithRedirectUri(redirectUri )
    .Build();
```

Como probablemente sepa, en producción, en lugar de utilizar un secreto de cliente, desea compartir con Azure AD un certificado. El código sería la siguiente:

```csharp
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithCertificate(certificate)
    .WithRedirectUri(redirectUri )
    .Build();
```

### <a name="initializing-a-public-client-application-from-configuration-options"></a>Inicializar una aplicación cliente pública de las opciones de configuración

El código siguiente crea una instancia de una aplicación cliente pública desde un objeto de configuración que podría rellenarse de mediante programación o leer desde un archivo de configuración:

```csharp
PublicClientApplicationOptions options = GetOptions(); // your own method
IPublicClientApplication app = PublicClientApplicationBuilder.CreateWithApplicationOptions(options)
    .Build();
```

### <a name="initializing-a-confidential-client-application-from-configuration-options"></a>Inicializar una aplicación cliente confidencial de las opciones de configuración

El mismo tipo de patrón se aplica a las aplicaciones cliente confidencial. También puede agregar otros parámetros mediante `.WithXXX` modificadores (aquí un certificado).

```csharp
ConfidentialClientApplicationOptions options = GetOptions(); // your own method
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(options)
    .WithCertificate(certificate)
    .Build();
```

## <a name="builder-modifiers"></a>Modificadores de generador

En los fragmentos de código con los generadores de aplicación, un número de `.With` se pueden aplicar a métodos como modificadores (por ejemplo, `.WithCertificate` y `.WithRedirectUri`). 

### <a name="modifiers-common-to-public-and-confidential-client-applications"></a>Modificadores comunes a las aplicaciones cliente públicos y confidenciales

Los modificadores que se puede establecer en un cliente público o el generador de la aplicación cliente confidencial son:

|Parámetro | DESCRIPCIÓN|
|--------- | --------- |
|`.WithAuthority()` 7 invalidaciones | Establece la entidad de aplicación predeterminado a una autoridad de Azure AD, con la posibilidad de elegir la nube de Azure, la audiencia, el inquilino (inquilino identificador o nombre de dominio), o proporcionar directamente el URI de autoridad.|
|`.WithAdfsAuthority(string)` | Establece la entidad de aplicación predeterminado sea una autoridad ADFS.|
|`.WithB2CAuthority(string)` | Establece la entidad de aplicación predeterminado sea una autoridad de Azure AD B2C.|
|`.WithClientId(string)` | Invalida el identificador de cliente.|
|`.WithComponent(string)` | Establece el nombre de la biblioteca mediante MSAL.NET (por motivos de telemetría). |
|`.WithDebugLoggingCallback()` | Si se llama, la aplicación llamará `Debug.Write` simplemente habilitar seguimientos de depuración. Consulte [registro](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/logging) para obtener más información.|
|`.WithExtraQueryParameters(IDictionary<string,string> eqp)` | Establezca los parámetros de consulta adicional de nivel de aplicación que se enviarán en todas las solicitudes de autenticación. Esto es reemplazable en cada nivel de método de adquisición de tokens (con el mismo `.WithExtraQueryParameters pattern`).|
|`.WithHttpClientFactory(IMsalHttpClientFactory httpClientFactory)` | Habilita las avanzadas escenarios tales como la configuración para un proxy HTTP o para forzar la MSAL para usar un HttpClient determinado (por ejemplo, en ASP.NET Core web apps o API).|
|`.WithLogging()` | Si se llama, la aplicación llamará a una devolución de llamada con seguimientos de depuración. Consulte [registro](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/logging) para obtener más información.|
|`.WithRedirectUri(string redirectUri)` | Invalida el URI de redirección de forma predeterminada. En el caso de las aplicaciones de cliente público, esto será útil para escenarios que implican al agente.|
|`.WithTelemetry(TelemetryCallback telemetryCallback)` | Establece al delegado utilizado para enviar datos de telemetría.|
|`.WithTenantId(string tenantId)` | Invalida el identificador del inquilino o la descripción del inquilino.|

### <a name="modifiers-specific-to-xamarinios-applications"></a>Modificadores específicos de las aplicaciones de Xamarin.iOS

Los modificadores que se puede establecer en un generador de aplicaciones de cliente público en Xamarin.iOS son:

|Parámetro | DESCRIPCIÓN|
|--------- | --------- |
|`.WithIosKeychainSecurityGroup()` | **Xamarin.iOS solo**: Establece el grupo de seguridad de la cadena de claves de iOS (para la persistencia de caché).|

### <a name="modifiers-specific-to-confidential-client-applications"></a>Modificadores específicos de las aplicaciones cliente confidencial

Los modificadores que se puede establecer en un generador de aplicaciones de cliente confidencial son:

|Parámetro | DESCRIPCIÓN|
|--------- | --------- |
|`.WithCertificate(X509Certificate2 certificate)` | Establece el certificado que identifica la aplicación con Azure AD.|
|`.WithClientSecret(string clientSecret)` | Establece el secreto de cliente (contraseña de aplicación) que identifica la aplicación con Azure AD.|

Estos modificadores son mutuamente excluyentes. Si proporciona ambos, MSAL producirá una excepción significativa.

### <a name="example-of-usage-of-modifiers"></a>Ejemplo de uso de modificadores

Supongamos que la aplicación es una aplicación de línea de negocio, que es solo para su organización.  A continuación, puede escribir:

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAadAuthority(AzureCloudInstance.AzurePublic, tenantId)
        .Build();
```

Donde resulta interesante es que ahora ha simplificado la programación para nubes nacionales. Si desea que la aplicación para que una aplicación de varios inquilinos en una nube nacional, podría escribir, por ejemplo:

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAadAuthority(AzureCloudInstance.AzureUsGovernment, AadAuthorityAudience.AzureAdMultipleOrgs)
        .Build();
```

También hay una invalidación para AD FS (2019 ADFS no se admite):
```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAdfsAuthority("https://consoso.com/adfs")
        .Build();
```

Por último, si es un desarrollador de Azure AD B2C, puede especificar al inquilino similar al siguiente:

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithB2CAuthority("https://fabrikamb2c.b2clogin.com/tfp/{tenant}/{PolicySignInSignUp}")
        .Build();
```
