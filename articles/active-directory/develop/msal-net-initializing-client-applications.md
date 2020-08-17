---
title: Inicialización de aplicaciones cliente de MSAL.NET | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre la inicialización de aplicaciones cliente confidenciales y públicas mediante la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/12/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 30f9f1998ee133c2546c9f4de7a99c51feb8740f
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88166202"
---
# <a name="initialize-client-applications-using-msalnet"></a>Inicialización de aplicaciones cliente con MSAL.NET
En este artículo se describe la inicialización de aplicaciones cliente confidenciales y públicas mediante la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET).  Para obtener más información sobre los tipos de aplicaciones cliente y las opciones de configuración de la aplicación, lea la [Introducción](msal-client-applications.md).

Con MSAL.NET 3.x, el método recomendado para crear una instancia de una aplicación es mediante el uso de los generadores de aplicaciones: `PublicClientApplicationBuilder` y `ConfidentialClientApplicationBuilder`. Ofrecen un mecanismo eficaz para configurar la aplicación, ya sea a partir del código o de un archivo de configuración, o incluso combinando ambos enfoques.

## <a name="prerequisites"></a>Prerrequisitos
Antes de inicializar una aplicación, primero tendrá que [registrarla](quickstart-register-app.md) para que se pueda integrar con la plataforma de identidad de Microsoft.  Después del registro, es posible que necesite la información siguiente (que puede encontrar en Azure Portal):

- El identificador de cliente (una cadena que representa un GUID).
- La URL del proveedor de identidades (la instancia) y la audiencia de inicio de sesión para la aplicación. De forma conjunta, estos dos parámetros se conocen como la autoridad.
- El identificador del inquilino si va a escribir una aplicación de línea de negocio exclusivamente para la organización (también denominada aplicación de un único inquilino).
- El secreto de aplicación (cadena de secreto de cliente) o el certificado (del tipo X509Certificate2) si se trata de una aplicación cliente confidencial.
- Para las aplicaciones web y, a veces, para las aplicaciones cliente públicas (concretamente cuando la aplicación debe usar a un agente), también habrá establecido la redirectUri donde el proveedor de identidades contactará con la aplicación mediante los tokens de seguridad.

## <a name="ways-to-initialize-applications"></a>Formas de inicializar aplicaciones
Hay muchas maneras diferentes para crear instancias de aplicaciones cliente.

### <a name="initializing-a-public-client-application-from-code"></a>Inicialización de una aplicación cliente pública a partir del código

El código siguiente crea una instancia de una aplicación cliente pública e inicia la sesión de los usuarios en la nube pública de Microsoft Azure con sus cuentas profesionales o educativas o bien con sus cuentas Microsoft personales.

```csharp
IPublicClientApplication app = PublicClientApplicationBuilder.Create(clientId)
    .Build();
```

### <a name="initializing-a-confidential-client-application-from-code"></a>Inicialización de una aplicación cliente confidencial a partir del código

En la misma manera, el código siguiente crea una instancia de una aplicación confidencial (una aplicación web ubicada en `https://myapp.azurewebsites.net`) y controla los tokens de los usuarios en la nube pública de Microsoft Azure con sus cuentas profesionales o educativas o bien con sus cuentas Microsoft personales. La aplicación se identifica con el proveedor de identidades compartiendo un secreto de cliente:

```csharp
string redirectUri = "https://myapp.azurewebsites.net";
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithClientSecret(clientSecret)
    .WithRedirectUri(redirectUri )
    .Build();
```

Como probablemente sepa, en producción, en lugar de utilizar un secreto de cliente, es posible que le interese compartir un certificado con Azure AD. En ese caso, el código sería el siguiente:

```csharp
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.Create(clientId)
    .WithCertificate(certificate)
    .WithRedirectUri(redirectUri )
    .Build();
```

### <a name="initializing-a-public-client-application-from-configuration-options"></a>Inicialización de una aplicación cliente pública a partir de las opciones de configuración

El código siguiente crea una instancia de una aplicación cliente pública a partir de un objeto de configuración, que podría rellenarse mediante programación o leerse desde un archivo de configuración:

```csharp
PublicClientApplicationOptions options = GetOptions(); // your own method
IPublicClientApplication app = PublicClientApplicationBuilder.CreateWithApplicationOptions(options)
    .Build();
```

### <a name="initializing-a-confidential-client-application-from-configuration-options"></a>Inicialización de una aplicación cliente confidencial a partir de las opciones de configuración

El mismo tipo de patrón se aplica a las aplicaciones cliente confidenciales. También puede agregar otros parámetros mediante modificadores de `.WithXXX` (en este caso, un certificado).

```csharp
ConfidentialClientApplicationOptions options = GetOptions(); // your own method
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder.CreateWithApplicationOptions(options)
    .WithCertificate(certificate)
    .Build();
```

## <a name="builder-modifiers"></a>Modificadores del generador

En los fragmentos de código que usan generadores de aplicaciones, pueden aplicarse como modificadores varios métodos de `.With` (por ejemplo, `.WithCertificate` y `.WithRedirectUri`). 

### <a name="modifiers-common-to-public-and-confidential-client-applications"></a>Modificadores comunes a las aplicaciones cliente públicas y confidenciales

Los modificadores que puede establecer en un generador de aplicaciones cliente públicas o confidenciales son:

|Modificador | Descripción|
|--------- | --------- |
|7 invalidaciones de `.WithAuthority()` | Establece la autoridad predeterminada de la aplicación en una autoridad de Azure AD, con la posibilidad de elegir Azure Cloud, la audiencia o el inquilino (identificador de inquilino o nombre de dominio) o de proporcionar directamente la URI de la autoridad.|
|`.WithAdfsAuthority(string)` | Establece la autoridad predeterminada de la aplicación en una autoridad de ADFS.|
|`.WithB2CAuthority(string)` | Establece la autoridad predeterminada de la aplicación en una autoridad de Azure AD B2C.|
|`.WithClientId(string)` | Invalida el identificador de cliente.|
|`.WithComponent(string)` | Establece el nombre de la biblioteca mediante MSAL.NET (por motivos de telemetría). |
|`.WithDebugLoggingCallback()` | Si se llama, la aplicación llamará a `Debug.Write` simplemente habilitando seguimientos de depuración. Consulte [Registro](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/logging) para obtener más información.|
|`.WithExtraQueryParameters(IDictionary<string,string> eqp)` | Establezca los parámetros de consulta adicionales del nivel de la aplicación que se enviarán en todas las solicitudes de autenticación. Esto se puede reemplazar en cada nivel de método de adquisición de tokens (con el mismo `.WithExtraQueryParameters pattern`).|
|`.WithHttpClientFactory(IMsalHttpClientFactory httpClientFactory)` | Habilita escenarios avanzados, como, por ejemplo, configurar un proxy HTTP o forzar que MSAL use un Http concreto (por ejemplo, en las API o aplicaciones web de ASP.NET Core).|
|`.WithLogging()` | Si se llama, la aplicación llamará a una devolución de llamada con seguimientos de depuración. Consulte [Registro](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/logging) para obtener más información.|
|`.WithRedirectUri(string redirectUri)` | Invalida la URI de redirección predeterminada. En el caso de las aplicaciones cliente públicas, será de utilidad para los escenarios que implican al agente.|
|`.WithTelemetry(TelemetryCallback telemetryCallback)` | Establece al delegado utilizado para enviar datos de telemetría.|
|`.WithTenantId(string tenantId)` | Invalida el identificador del inquilino o la descripción del inquilino.|

### <a name="modifiers-specific-to-xamarinios-applications"></a>Modificadores específicos de las aplicaciones de Xamarin.iOS

Los modificadores que puede establecer en un generador de aplicaciones cliente públicas en Xamarin.iOS son los siguientes:

|Modificador | Descripción|
|--------- | --------- |
|`.WithIosKeychainSecurityGroup()` | **Solo para Xamarin.iOS**: Establece el grupo de seguridad de la cadena de claves de iOS (para la persistencia de la caché).|

### <a name="modifiers-specific-to-confidential-client-applications"></a>Modificadores específicos de las aplicaciones cliente confidenciales

Los modificadores que puede establecer en un generador de aplicaciones cliente confidenciales son los siguientes:

|Modificador | Descripción|
|--------- | --------- |
|`.WithCertificate(X509Certificate2 certificate)` | Establece el certificado que identifica la aplicación con Azure AD.|
|`.WithClientSecret(string clientSecret)` | Establece el secreto de cliente (contraseña de la aplicación) que identifica la aplicación con Azure AD.|

Estos modificadores se excluyen mutuamente. Si proporciona ambos modificadores, MSAL producirá una excepción significativa.

### <a name="example-of-usage-of-modifiers"></a>Ejemplo de uso de los modificadores

Supongamos que la aplicación es una aplicación de línea de negocio, que es solo para su organización.  En ese caso, puede escribir:

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAadAuthority(AzureCloudInstance.AzurePublic, tenantId)
        .Build();
```

Donde resulta interesante que ahora ha simplificado la programación para nubes nacionales. Si desea que la aplicación sea una aplicación de varios inquilinos en una nube nacional, podría escribir, por ejemplo:

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAadAuthority(AzureCloudInstance.AzureUsGovernment, AadAuthorityAudience.AzureAdMultipleOrgs)
        .Build();
```

También hay una invalidación para ADFS (actualmente no se admite ADFS 2019):
```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithAdfsAuthority("https://consoso.com/adfs")
        .Build();
```

Por último, si es un desarrollador de Azure AD B2C, puede especificar al inquilino de una forma similar a la siguiente:

```csharp
IPublicClientApplication app;
app = PublicClientApplicationBuilder.Create(clientId)
        .WithB2CAuthority("https://fabrikamb2c.b2clogin.com/tfp/{tenant}/{PolicySignInSignUp}")
        .Build();
```
