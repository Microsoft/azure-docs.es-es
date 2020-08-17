---
title: Migración de aplicaciones de Xamarin mediante agentes a MSAL.NET
titleSuffix: Microsoft identity platform
description: Aprenda a migrar aplicaciones de Xamarin iOS que usan Microsoft Authenticator de ADAL.NET a MSAL.NET.
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 09/08/2019
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: bc94c7be4e3979cf9aa7624a9aeadf156cc48035
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88166083"
---
# <a name="migrate-ios-applications-that-use-microsoft-authenticator-from-adalnet-to-msalnet"></a>Migración de aplicaciones de iOS que usan Microsoft Authenticator de ADAL.NET a MSAL.NET

Ha estado usando la biblioteca de autenticación de Azure Active Directory para .NET (ADAL.NET) y el agente de iOS. Ahora es el momento de migrar a la [biblioteca de autenticación de Microsoft](msal-overview.md) para .NET (MSAL.net), que es compatible con el agente de iOS de la versión 4.3 en adelante. 

¿Por dónde debe empezar? Este artículo le ayuda a migrar la aplicación de Xamarin iOS de ADAL a MSAL.

## <a name="prerequisites"></a>Prerrequisitos
En este documento se supone que ya tiene una aplicación de Xamarin iOS que se integra con el agente de iOS. Si no es así, pase directamente a MSAL.NET y comience ahí la implementación del agente. Para información sobre cómo invocar el agente de iOS en MSAL.net con una nueva aplicación, consulte [esta documentación](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Leveraging-the-broker-on-iOS#why-use-brokers-on-xamarinios-and-xamarinandroid-applications).

## <a name="background"></a>Información previa

### <a name="what-are-brokers"></a>¿Qué son los agentes?

Los agentes son aplicaciones proporcionadas por Microsoft en Android e iOS. (Consulte la aplicación [Microsoft Authenticator](https://www.microsoft.com/p/microsoft-authenticator/9nblgggzmcj6) en iOS y Android y la aplicación del Portal de empresa de Intune en Android). 

Habilitan lo siguiente:

- Inicio de sesión único.
- Identificación del dispositivo, que requieren algunas [directivas de acceso condicional](../conditional-access/overview.md). Para más información, consulte [Administración de dispositivos](../conditional-access/concept-conditional-access-conditions.md#device-platforms).
- Comprobación de identificación de la aplicación, que también es necesaria en algunos escenarios empresariales. Para más información, consulte [Administración de aplicaciones móviles (MAM) de Intune](/intune/mam-faq).

## <a name="migrate-from-adal-to-msal"></a>Migración de ADAL a MSAL

### <a name="step-1-enable-the-broker"></a>Paso 1: Habilitar el agente

<table>
<tr><td>Código de ADAL actual:</td><td>Homólogo de MSAL:</td></tr>
<tr><td>
En ADAL.NET, la compatibilidad con el agente se habilitaba en cada contexto de autenticación. De forma predeterminada, está deshabilitada. Tenía que establecer una marca 

`useBroker` en true en el constructor `PlatformParameters` para llamar al agente:

```csharp
public PlatformParameters(
        UIViewController callerViewController, 
        bool useBroker)
```
Además, en el código específico de la plataforma de este ejemplo, en el representador de páginas para iOS, establezca el valor de la marca `useBroker` 
en true:
```csharp
page.BrokerParameters = new PlatformParameters(
          this, 
          true, 
          PromptBehavior.SelectAccount);
```

A continuación, incluya los parámetros en la llamada de adquisición de token:
```csharp
 AuthenticationResult result =
                    await
                        AuthContext.AcquireTokenAsync(
                              Resource, 
                              ClientId, 
                              new Uri(RedirectURI), 
                              platformParameters)
                              .ConfigureAwait(false);
```

</td><td>
En MSAL.NET, la compatibilidad con el agente se habilita según el elemento PublicClientApplication. De forma predeterminada, está deshabilitada. Para habilitarla, use el 

parámetro `WithBroker()` (establecido en true de manera predeterminada) para llamar al agente:

```csharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithReplyUri(redirectUriOnIos)
                .Build();
```
En la llamada al token de adquisición:
```csharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow(App.RootViewController)
             .ExecuteAsync();
```
</table>

### <a name="step-2-set-a-uiviewcontroller"></a>Paso 2: Establecer un UIViewController()
En ADAL.NET, se pasaba UIViewController como parte de `PlatformParameters`. (Vea el ejemplo del paso 1). Sin embargo, en MSAL.NET, para ofrecer a los desarrolladores una mayor flexibilidad, se usa una ventana de objeto, pero no hace falta con el uso normal de iOS. Para usar el agente, establezca la ventana de objeto para enviar y recibir respuestas del agente. 
<table>
<tr><td>Código de ADAL actual:</td><td>Homólogo de MSAL:</td></tr>
<tr><td>
Se pasaba UIViewController a 

`PlatformParameters` en la plataforma específica de iOS.

```csharp
page.BrokerParameters = new PlatformParameters(
          this, 
          true, 
          PromptBehavior.SelectAccount);
```
</td><td>
En MSAL.NET, deberá hacer dos cosas para establecer la ventana de objeto para iOS:

1. En `AppDelegate.cs`, establezca `App.RootViewController` en un valor de `UIViewController()` nuevo. Esta asignación garantiza que hay un elemento UIViewController con la llamada al agente. Si no se establece correctamente, puede obtener este error: `"uiviewcontroller_required_for_ios_broker":"UIViewController is null, so MSAL.NET cannot invoke the iOS broker. See https://aka.ms/msal-net-ios-broker"`.
1. En la llamada a AcquireTokenInteractive, use `.WithParentActivityOrWindow(App.RootViewController)` y pase la referencia a la ventana de objeto que usará.

**Por ejemplo:**

En `App.cs`:
```csharp
   public static object RootViewController { get; set; }
```
En `AppDelegate.cs`:
```csharp
   LoadApplication(new App());
   App.RootViewController = new UIViewController();
```
En la llamada al token de adquisición:
```csharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow(App.RootViewController)
             .ExecuteAsync();
```

</table>

### <a name="step-3-update-appdelegate-to-handle-the-callback"></a>Paso 3: Actualizar AppDelegate para controlar la devolución de llamada
Tanto ADAL como MSAL llaman al agente y, el agente, a su vez, devuelve la llamada a la aplicación con el método `OpenUrl` de la clase `AppDelegate`. Para obtener más información, consulte [esta documentación](msal-net-use-brokers-with-xamarin-apps.md#step-3-update-appdelegate-to-handle-the-callback).

No hay ningún cambio aquí entre ADAL.NET y MSAL.NET.

### <a name="step-4-register-a-url-scheme"></a>Paso 4: Registrar un esquema de dirección URL
ADAL.NET y MSAL.NET usan direcciones URL para invocar al agente y devolver la respuesta del agente a la aplicación. Registre el esquema de dirección URL en el archivo `Info.plist` de la aplicación de la siguiente manera:

<table>
<tr><td>Código de ADAL actual:</td><td>Homólogo de MSAL:</td></tr>
<tr><td>
El esquema de dirección URL es exclusivo de la aplicación.
</td><td>
Tenga en cuenta que 

el nombre de `CFBundleURLSchemes` debe incluir 

`msauth.`

como prefijo, seguido del `CFBundleURLName`.

Por ejemplo: `$"msauth.(BundleId")`

```csharp
 <key>CFBundleURLTypes</key>
    <array>
      <dict>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>CFBundleURLName</key>
        <string>com.yourcompany.xforms</string>
        <key>CFBundleURLSchemes</key>
        <array>
          <string>msauth.com.yourcompany.xforms</string>
        </array>
      </dict>
    </array>
```

> [!NOTE]
> Este esquema de dirección URL se convierte en parte del URI de redirección que se usa para identificar de forma única la aplicación cuando recibe la respuesta del agente.

</table>

### <a name="step-5-add-the-broker-identifier-to-the-lsapplicationqueriesschemes-section"></a>Paso 5: Agregar el identificador de agente a la sección LSApplicationQueriesSchemes

ADAL.NET y MSAL.NET usan `-canOpenURL:` para comprobar si el agente está instalado en el dispositivo. Agregue el identificador correcto para el agente de iOS a la sección LSApplicationQueriesSchemes del archivo info.plist de la siguiente manera:

<table>
<tr><td>Código de ADAL actual:</td><td>Homólogo de MSAL:</td></tr>
<tr><td>
Usos 

`msauth`


```csharp
<key>LSApplicationQueriesSchemes</key>
<array>
     <string>msauth</string>
</array>
```
</td><td>
Usos 

`msauthv2`


```csharp
<key>LSApplicationQueriesSchemes</key>
<array>
     <string>msauthv2</string>
     <string>msauthv3</string>
</array>
```
</table>

### <a name="step-6-register-your-redirect-uri-in-the-portal"></a>Paso 6: Registrar el URI de redirección en el portal

ADAL.NET y MSAL.NET agregan un requisito adicional al URI de redirección cuando se dirige al agente. Registre el URI de redirección con la aplicación en el portal.
<table>
<tr><td>Código de ADAL actual:</td><td>Homólogo de MSAL:</td></tr>
<tr><td>

`"<app-scheme>://<your.bundle.id>"`

Ejemplo: 

`mytestiosapp://com.mycompany.myapp`
</td><td>

`$"msauth.{BundleId}://auth"`

Ejemplo:

`public static string redirectUriOnIos = "msauth.com.yourcompany.XForms://auth"; `

</table>

Para más información sobre cómo registrar el URI de redirección en el portal, consulte [Uso del agente en las aplicaciones de Xamarin.iOS](msal-net-use-brokers-with-xamarin-apps.md#step-8-make-sure-the-redirect-uri-is-registered-with-your-app).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre las [Consideraciones específicas de Xamarin iOS con MSAL.NET](msal-net-xamarin-ios-considerations.md).