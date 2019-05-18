---
title: Consideraciones de Xamarin Android (biblioteca de autenticación de Microsoft para. NET) | Azure
description: Obtenga información sobre las consideraciones específicas al usar Xamarin Android con la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
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
ms.date: 04/24/2019
ms.author: ryanwi
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: cb0cfb06e95cadbb549f669e5d59bdb0d795c896
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65545882"
---
# <a name="xamarin-android-specific-considerations-with-msalnet"></a>Consideraciones específicas de Android de Xamarin con MSAL.NET
En este artículo se describe algunas consideraciones al usar Xamarin Android con la biblioteca de autenticación de Microsoft para .NET (MSAL.NET).

Este artículo es para MSAL.NET 3.x. Si está interesado en MSAL.NET 2.x, consulte [las características de Xamarin para Android en MSAL.NET 2.x](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/Xamarin-Android-specifics-2x).

## <a name="set-the-parent-activity"></a>Establece la actividad primaria

En Xamarin.Android, deberá establecer la actividad primaria para que el token que se obtiene una vez que ha ocurrido la interacción.

```csharp
var authResult = AcquireTokenInteractive(scopes)
 .WithParentActivityOrWindow(parentActivity)
 .ExecuteAsync();
```

## <a name="ensuring-control-goes-back-to-msal-once-the-interactive-portion-of-the-authentication-flow-ends"></a>Garantizar el control vuelve a MSAL una vez que la parte interactiva de los extremos de flujo de autenticación
En Android, deberá reemplazar el `OnActivityResult` método de la `Activity` y llame al método SetAuthenticationContinuationEventArgs de la clase AuthenticationContinuationHelper MSAL.

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
Esa línea garantiza que el control vuelve a MSAL una vez que la parte interactiva del flujo de autenticación ha finalizado.

## <a name="update-the-android-manifest"></a>Actualizar el manifiesto de Android
El `AndroidManifest.xml` debe contener los siguientes valores:
```csharp
<activity android:name="microsoft.identity.client.BrowserTabActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="msal{client_id}" android:host="auth" />
         </intent-filter>
</activity>
```

## <a name="use-the-embedded-web-view-optional"></a>Utilice la vista web incrustado (opcional)

De forma predeterminada MSAL.NET usa el explorador web del sistema, que le permite obtener el SSO con aplicaciones Web y otras aplicaciones. En algunos casos excepcionales, puede especificar que desea utilizar la vista web incrustado. Para obtener más información, consulte [MSAL.NET utiliza un explorador Web](msal-net-web-browsers.md) y [explorador del sistema Android](msal-net-system-browser-android-considerations.md).

```csharp
bool useEmbeddedWebView = !app.IsSystemWebViewAvailable;

var authResult = AcquireTokenInteractive(scopes)
 .WithParentActivityOrWindow(parentActivity)
 .WithEmbeddedWebView(useEmbeddedWebView)
 .ExecuteAsync();
```

## <a name="troubleshooting"></a>solución de problemas
Si crea una nueva aplicación de Xamarin.Forms y agregue una referencia al paquete MSAL.Net NuGet, esto, simplemente funcionan.
Sin embargo, si desea actualizar una aplicación existente de Xamarin.Forms para MSAL.NET una vista previa 1.1.2 o posterior puede experimentar problemas de compilación.

Para solucionar estos problemas, debe:
- Actualizar el paquete NuGet MSAL.NET existente a la versión preliminar MSAL.NET 1.1.2 o posterior
- Compruebe que Xamarin.Forms se actualizan automáticamente a la versión 2.5.0.122203 (si no, la actualización a esta versión)
- Compruebe que Xamarin.Android.Support.v4 se actualizan automáticamente a la versión 25.4.0.2 (si no, la actualización a esta versión)
- Todos los paquetes de Xamarin.Android.Support deben estar destinado a la versión 25.4.0.2
- Limpiar o vuelve a generar
- Intente establecer el proyecto paralelas max se basa en 1 en Visual Studio (las opciones -> proyectos y soluciones -> compilación y ejecución -> número máximo de compilaciones de proyectos paralelos)
- Como alternativa, si va a compilar desde la línea de comandos, pruebe a quitar /m desde el comando si lo está utilizando.


### <a name="error-the-name-authenticationcontinuationhelper-does-not-exist-in-the-current-context"></a>Error: El nombre 'AuthenticationContinuationHelper' no existe en el contexto actual

Probablemente esto es porque Visual Studio no ha actualizado correctamente el archivo Android.csproj*. A veces la **<HintPath>** filepath incorrectamente contiene netstandard13 en lugar de **monoandroid90**.

```xml
<Reference Include="Microsoft.Identity.Client, Version=3.0.4.0, Culture=neutral, PublicKeyToken=0a613f4dd989e8ae,
           processorArchitecture=MSIL">
  <HintPath>..\..\packages\Microsoft.Identity.Client.3.0.4-preview\lib\monoandroid90\Microsoft.Identity.Client.dll</HintPath>
</Reference>
```

## <a name="next-steps"></a>Pasos siguientes

Se proporcionan más detalles y ejemplos en los [Android consideraciones específicas](https://github.com/azure-samples/active-directory-xamarin-native-v2#android-specific-considerations) párrafo del archivo del archivo readme.md del ejemplo siguiente:

| Muestra | Plataforma | DESCRIPCIÓN |
| ------ | -------- | ----------- |
|[https://github.com/Azure-Samples/active-directory-xamarin-native-v2](https://github.com/azure-samples/active-directory-xamarin-native-v2) | Xamarin iOS, Android, UWP | Una aplicación de Xamarin Forms sencilla que muestra cómo usar MSAL para autenticar MSA y Azure AD a través del punto de conexión v2.0 agregue y tener acceso a Microsoft Graph con el token resultante. <br>![Topología](media/msal-net-xamarin-android-considerations/topology.png) |