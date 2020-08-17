---
title: Consideraciones sobre Xamarin Android (MSAL.NET) | Azure
titleSuffix: Microsoft identity platform
description: Conozca las consideraciones para usar Xamarin Android con la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/24/2019
ms.author: marsma
ms.reviewer: saeeda
ms.custom: devx-track-csharp, aaddev
ms.openlocfilehash: 601a501114d754dc82991a3c19b977c0c63c5bc0
ms.sourcegitcommit: c28fc1ec7d90f7e8b2e8775f5a250dd14a1622a6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/13/2020
ms.locfileid: "88165743"
---
# <a name="considerations-for-using-xamarin-android-with-msalnet"></a>Consideraciones para usar Xamarin Android con MSAL.NET
En este artículo se describe lo que debe tener en cuenta al usar Xamarin Android con la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET).

## <a name="set-the-parent-activity"></a>Definición de la actividad primaria

En Xamarin Android, establezca la actividad primaria para que el token se devuelva después de la interacción. Este es un ejemplo de código:

```csharp
var authResult = AcquireTokenInteractive(scopes)
 .WithParentActivityOrWindow(parentActivity)
 .ExecuteAsync();
```

En MSAL 4.2 y versiones posteriores, también puede establecer esta funcionalidad en el nivel de `PublicClientApplication`. Para ello, use una devolución de llamada:

```csharp
// Requires MSAL.NET 4.2 or later
var pca = PublicClientApplicationBuilder
  .Create("<your-client-id-here>")
  .WithParentActivityOrWindow(() => parentActivity)
  .Build();
```

Si usa [CurrentActivityPlugin](https://github.com/jamesmontemagno/CurrentActivityPlugin), el código del generador `PublicClientApplication` es similar al del ejemplo siguiente.

```csharp
// Requires MSAL.NET 4.2 or later
var pca = PublicClientApplicationBuilder
  .Create("<your-client-id-here>")
  .WithParentActivityOrWindow(() => CrossCurrentActivity.Current)
  .Build();
```

## <a name="ensure-that-control-returns-to-msal"></a>Asegurarse de que el control vuelve a MSAL 
Cuando la parte interactiva del flujo de autenticación termine, asegúrese de que el control se devuelve a MSAL. En Android, invalide el método `OnActivityResult` de `Activity`. A continuación, llame al método `SetAuthenticationContinuationEventArgs` de la clase MSAL `AuthenticationContinuationHelper`. 

Este es un ejemplo:

```csharp
protected override void OnActivityResult(int requestCode, 
                                         Result resultCode, Intent data)
{
 base.OnActivityResult(requestCode, resultCode, data);
 AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(requestCode,
                                                                         resultCode,
                                                                         data);
}

```

Esta línea garantiza que el control se devuelve a MSAL cuando se completa la parte interactiva del flujo de autenticación.

## <a name="update-the-android-manifest"></a>Actualización del manifiesto de Android
El archivo *AndroidManifest.xml* debe contener los valores siguientes:

<!--Intent filter to capture System Browser or Authenticator calling back to our app after sign-in-->
```
  <activity
        android:name="com.microsoft.identity.client.BrowserTabActivity">
     <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="msauth"
                android:host="Enter_the_Package_Name"
                android:path="/Enter_the_Signature_Hash" />
     </intent-filter>
 </activity>
```

Sustituya el nombre del paquete que registró en Azure Portal por el valor `android:host=`. Sustituya el hash de clave que registró en Azure Portal por el valor `android:path=`. El hash de firma *no* debe estar codificado como dirección URL. Asegúrese de que aparece un carácter de barra inclinada (`/`) inicial al principio del hash de firma.

Como alternativa, [cree la actividad en el código](/xamarin/android/platform/android-manifest#the-basics) en lugar de editar manualmente el archivo *AndroidManifest.xml*. Para crear la actividad en el código, cree primero una clase que incluya los atributos `Activity` y `IntentFilter`. 

A continuación se muestra un ejemplo de una clase que representa los valores del archivo XML:

```csharp
  [Activity]
  [IntentFilter(new[] { Intent.ActionView },
        Categories = new[] { Intent.CategoryBrowsable, Intent.CategoryDefault },
        DataHost = "auth",
        DataScheme = "msal{client_id}")]
  public class MsalActivity : BrowserTabActivity
  {
  }
```

### <a name="xamarinforms-43x-manifest"></a>Manifiesto Xamarin.Forms 4.3.X

Xamarin.Forms 4.3.x genera código que establece el atributo `package` en `com.companyname.{appName}` en el archivo *AndroidManifest.xml*. Si usa `DataScheme` como `msal{client_id}`, es posible que desee cambiar el valor para que coincida con el del espacio de nombres `MainActivity.cs`.

## <a name="use-the-embedded-web-view-optional"></a>Uso de la vista web incrustada (opcional)

De forma predeterminada, MSAL.NET usa el explorador web del sistema. Este explorador permite obtener el inicio de sesión único (SSO) mediante aplicaciones web y otras aplicaciones. En casos excepcionales, es posible que desee que el sistema use una vista web insertada. 

En este ejemplo de código se muestra cómo configurar una vista web insertada:

```csharp
bool useEmbeddedWebView = !app.IsSystemWebViewAvailable;

var authResult = AcquireTokenInteractive(scopes)
 .WithParentActivityOrWindow(parentActivity)
 .WithEmbeddedWebView(useEmbeddedWebView)
 .ExecuteAsync();
```

Para obtener más información, consulte [Uso de exploradores web (MSAL.NET)](msal-net-web-browsers.md) y [Consideraciones del explorador del sistema de Xamarin Android](msal-net-system-browser-android-considerations.md).


## <a name="troubleshoot"></a>Solución de problemas
Puede crear una nueva aplicación Xamarin.Forms y agregar una referencia al paquete NuGet de MSAL.NET.
Sin embargo, puede experimentar problemas de compilación si actualiza una aplicación Xamarin.Forms existente a la versión preliminar de MSAL.NET 1.1.2 o posterior.

Para solucionar los problemas de compilación:

- Actualice el paquete NuGet de MSAL.NET existente a la versión preliminar de MSAL.NET 1.1.2 o posterior.
- Compruebe que Xamarin.Forms se actualiza automáticamente a la versión 2.5.0.122203. Si es necesario, actualice Xamarin.Forms a esta versión.
- Compruebe que Xamarin.Android.Support.v4 se actualiza automáticamente a la versión 25.4.0.2. Si es necesario, actualícelo a la versión 25.4.0.2.
- Asegúrese de que todos los paquetes de Xamarin.Android.Support apuntan a la versión 25.4.0.2.
- Limpie o vuelva a compilar la aplicación.
- En Visual Studio, pruebe a establecer el número máximo de compilaciones de proyecto paralelas en 1. Para ello, seleccione **Opciones** > **Proyectos y soluciones** > **Compilar y ejecutar** > **Número máximo de compilaciones de proyecto paralelas**.
- Si realiza la compilación desde la línea de comandos y el comando usa `/m`, intente quitar este elemento del comando.

### <a name="error-the-name-authenticationcontinuationhelper-doesnt-exist-in-the-current-context"></a>Error: El nombre AuthenticationContinuationHelper no existe en el contexto actual.

Si un error indica que `AuthenticationContinuationHelper` no existe en el contexto actual, es posible que Visual Studio haya actualizado incorrectamente el archivo Android.csproj*. A veces la ruta de archivo *\<HintPath>* contiene incorrectamente *netstandard13* en lugar de *monoandroid90*.

Este ejemplo contiene una ruta de archivo correcta:

```xml
<Reference Include="Microsoft.Identity.Client, Version=3.0.4.0, Culture=neutral, PublicKeyToken=0a613f4dd989e8ae,
           processorArchitecture=MSIL">
  <HintPath>..\..\packages\Microsoft.Identity.Client.3.0.4-preview\lib\monoandroid90\Microsoft.Identity.Client.dll</HintPath>
</Reference>
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte el ejemplo de una [aplicación móvil de Xamarin que usa la Plataforma de identidad de Microsoft](https://github.com/azure-samples/active-directory-xamarin-native-v2#android-specific-considerations). En la tabla siguiente se resume la información relevante del archivo LÉAME.

| Muestra | Plataforma | Descripción |
| ------ | -------- | ----------- |
|[https://github.com/Azure-Samples/active-directory-xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) | Xamarin.iOS, Android, UWP | Aplicación sencilla de Xamarin.Forms que muestra cómo usar MSAL para autenticar cuentas personales de Microsoft y Azure AD a través del punto de conexión de Azure AD 2.0. La aplicación también muestra cómo acceder a Microsoft Graph y el token resultante. <br>![Topología](media/msal-net-xamarin-android-considerations/topology.png) |