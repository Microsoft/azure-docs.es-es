---
title: Exploradores en la biblioteca de autenticación de Microsoft Web para .NET | Azure
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
ms.date: 05/06/2019
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 4f7f91e6ab1fb12132068b839e66fafd3ab1bc73
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65543955"
---
# <a name="using-web-browsers-in-msalnet"></a>Uso de los exploradores web en MSAL.NET
Los exploradores Web son necesarios para la autenticación interactiva. De forma predeterminada, admite MSAL.NET el [explorador web del sistema](#system-web-browser-on-xamarinios-and-xamarinandroid) en Xamarin.iOS y [Xamarin.Android](https://github.com/AzureAD/microsoft-authentication-library-for-dotnet/wiki/system-browser). Pero [también puede habilitar el explorador Web incrustado](#enable-embedded-webviews) dependiendo de los requisitos (UX, necesario para single sign-on (SSO), seguridad) en [Xamarin.iOS](#choosing-between-embedded-web-browser-or-system-browser-on-xamarinios) y [Xamarin.Android](#choosing-between-embedded-web-browser-or-system-browser-on-xamarinandroid) aplicaciones. Y puede incluso [elija dinámicamente](#detecting-the-presence-of-custom-tabs-on-xamarinandroid) qué explorador web para usar en función de la presencia de Chrome o un explorador compatible con las pestañas personalizadas de Chrome en Android.

## <a name="web-browsers-in-msalnet"></a>Exploradores Web en MSAL.NET

Es importante comprender que, al adquirir un token de forma interactiva, el contenido del cuadro de diálogo no se proporciona en la biblioteca, pero el STS (servicio de Token de seguridad). El punto de conexión de autenticación devuelve algún HTML y JavaScript que controla la interacción, que se representa en un explorador web o un control web. Permitir que el STS controlar la interacción de HTML tiene muchas ventajas:

- Nunca se almacena la contraseña (si se ha escrito uno) por la aplicación ni la biblioteca de autenticación.
- Permite las redirecciones a otros proveedores de identidades (por ejemplo inicio de sesión de con una cuenta educativa o una cuenta personal con MSAL, o con una cuenta social con Azure AD B2C).
- Permite que el STS controlar el acceso condicional, por ejemplo, haciendo que el usuario realice varias autenticación multifactor (MFA) durante la fase de autenticación (escribir un pin de Windows Hello o que se llama en su teléfono o en una aplicación en su teléfono de autenticación). En casos donde la autenticación multifactor necesarios aún no está establecida, seguridad, el usuario puede configurar, justo a tiempo en el mismo cuadro de diálogo.  El usuario escribe su número de teléfono móvil y se le guiará para instalar una aplicación de autenticación y analizar una etiqueta QR para agregar su cuenta. Este servidor controlado por la interacción es una gran experiencia.
- Permite al usuario cambiar la contraseña en este mismo cuadro de diálogo cuando la contraseña ha expirado (proporcionar campos adicionales para la contraseña antigua y la nueva contraseña).
- Permite la personalización de marca del inquilino, o la aplicación (imágenes) controlado por el Administrador de inquilinos de Azure AD / propietario de la aplicación.
- Permite a los usuarios den su consentimiento para permitir que la aplicación acceder a los recursos / ámbitos en su nombre justo después de la autenticación.

## <a name="system-web-browser-on-xamarinios-and-xamarinandroid"></a>Explorador web del sistema en Xamarin.iOS y Xamarin.Android

De forma predeterminada, MSAL.NET es compatible con el explorador web del sistema en Xamarin.iOS y Xamarin.Android. Para todas las plataformas que proporcionan la interfaz de usuario (es decir, .NET Core), se proporciona un cuadro de diálogo en la biblioteca de incrustación de un control de explorador Web. MSAL.NET también usa una vista web incrustado para el escritorio de .NET y WAB para la plataforma UWP. Sin embargo, aprovecha de forma predeterminada el **explorador web del sistema** para Xamarin aplicaciones de iOS y Xamarin Android. En iOS, incluso elige la vista web para utilizar según la versión del sistema operativo (iOS12, iOS11 y versiones anteriores).

Mediante el explorador del sistema tiene la gran ventaja de uso compartido del estado de inicio de sesión único con otras aplicaciones y aplicaciones web sin necesidad de un agente (portal de empresa / Authenticator). Se utilizó el explorador del sistema, de forma predeterminada, en el MSAL.NET para Xamarin plataformas iOS y Xamarin Android porque, en estas plataformas, el explorador web del sistema ocupa toda la pantalla y la experiencia del usuario es mejor. La vista web del sistema no es discernible de un cuadro de diálogo. Sin embargo, en iOS, el usuario podría tener que dar su consentimiento para el explorador para volver a llamar la aplicación, que puede resultar molesta.

### <a name="uwp-does-not-use-the-system-webview"></a>UWP no utiliza el Webview del sistema

Para aplicaciones de escritorio, sin embargo, iniciar un Webview del sistema da lugar a una experiencia de usuario de una, como el usuario ve el explorador, donde es posible que ya tienen otras fichas al abrir. Y cuando se ha producido la autenticación, los usuarios obtiene una página solicitándoles cerrar esta ventana. Si el usuario no presta atención, puede cerrar todo el proceso (incluidos otras pestañas, que no están relacionadas con la autenticación). Aprovechando el explorador del sistema en el escritorio también requeriría la apertura de puertos locales y escuchando en ellos, que podría requerir permisos avanzados para la aplicación. Usted, como desarrollador, el usuario o el administrador, podría ser reacios sobre este requisito.

## <a name="enable-embedded-webviews"></a>Habilitar insertadas 
También puede habilitar insertadas en aplicaciones de Xamarin.iOS y Xamarin.Android. A partir de MSAL.NET 2.0.0-preview, MSAL.NET también admite el uso de la **incrustado** opción webview. Para ADAL.NET, webview incrustado es la única opción admitida.

Como desarrollador mediante MSAL.NET tienen como destino Xamarin, puede usar insertadas o exploradores de sistema. Se trata de su elección, dependiendo de las preocupaciones de experiencia y seguridad de usuario que desea dirigirse.

Actualmente, MSAL.NET no admite aún los agentes de iOS y Android. Por lo tanto, si tiene que proporcionar el inicio de sesión único (SSO), el explorador del sistema todavía podría ser una mejor opción. Compatibilidad con los agentes con el explorador web incrustado es en el trabajo pendiente MSAL.NET.

### <a name="differences-between-embedded-webview-and-system-browser"></a>Diferencias entre webview incrustado y explorador del sistema 
Hay algunas diferencias visuales entre webview incrustado y explorador del sistema en MSAL.NET.

**Inicio de sesión interactivo con MSAL.NET mediante la vista Web incrustado:**

![embedded](media/msal-net-web-browsers/embedded-webview.png)

**Inicio de sesión interactivo con MSAL.NET mediante el explorador del sistema:**

![Explorador del sistema](media/msal-net-web-browsers/system-browser.png)

### <a name="developer-options"></a>Opciones del desarrollador

Como desarrollador mediante MSAL.NET, tiene varias opciones para mostrar el cuadro de diálogo interactivo de STS:

- **Explorador del sistema.** El explorador del sistema se establece de forma predeterminada en la biblioteca. Si usa Android, lea [exploradores sistema](msal-net-system-browser-android-considerations.md) para obtener información específica acerca de qué exploradores se admiten para la autenticación. Al utilizar el explorador del sistema en Android, se recomienda que el dispositivo tiene un explorador compatible con las pestañas personalizadas de Chrome.  En caso contrario, puede producir un error de autenticación.
- **Webview incrustado.** Para usar sólo incrustado webview en MSAL.NET, el `AcquireTokenInteractively` generador de parámetros contiene un `WithUseEmbeddedWebView()` método.

    iOS

    ```csharp
    AuthenticationResult authResult;
    authResult = app.AcquireTokenInteractively(scopes)
                    .WithUseEmbeddedWebView(useEmbeddedWebview)
                    .ExecuteAsync();
    ```

    Android:

    ```csharp
    authResult = app.AcquireTokenInteractively(scopes)
                .WithParentActivityOrWindow(activity)
                .WithUseEmbeddedWebView(useEmbeddedWebview)
                .ExecuteAsync();
    ```

#### <a name="choosing-between-embedded-web-browser-or-system-browser-on-xamarinios"></a>Elegir entre el explorador web incrustado o explorador del sistema en Xamarin.iOS

En la aplicación de iOS en `AppDelegate.cs` se puede inicializar el `ParentWindow` a `null`. No se usa en iOS

```csharp
App.ParentWindow = null; // no UI parent on iOS
```

#### <a name="choosing-between-embedded-web-browser-or-system-browser-on-xamarinandroid"></a>Elegir entre el explorador web incrustado o explorador del sistema en Xamarin.Android

En su aplicación de Android en `MainActivity.cs` puede establecer la actividad primaria, por lo que los resultados de la autenticación se vuelve a ella:

```csharp
 App.ParentWindow = this;
```

A continuación, en el `MainPage.xaml.cs`:

```csharp
authResult = await App.PCA.AcquireTokenInteractive(App.Scopes)
                      .WithParentActivityOrWindow(App.ParentWindow)
                      .WithUseEmbeddedWebView(true)
                      .ExecuteAsync();
```

#### <a name="detecting-the-presence-of-custom-tabs-on-xamarinandroid"></a>Detectar la presencia de las pestañas personalizadas en Xamarin.Android

Si desea usar el explorador web del sistema para habilitar SSO con las aplicaciones que se ejecutan en el explorador, pero se preocupa de la experiencia del usuario para dispositivos Android no tiene un explorador con compatibilidad de la ficha personalizada, tiene la opción de decidir mediante una llamada a la `IsSystemWebViewAvailable()` método < c 2 > `IPublicClientApplication` . Este método devuelve `true` si el PackageManager detecta pestañas personalizadas y `false` si no se detectan en el dispositivo.

Según el valor devuelto por este método y sus requisitos, puede tomar una decisión:

- Puede devolver un mensaje de error personalizado al usuario. Por ejemplo: "Instale Chrome para continuar con la autenticación" - OR-
- Puede recurrir a la opción de webview incrustado e iniciar la interfaz de usuario como una vista Web incrustado.

El código siguiente muestra la opción de webview incrustado:

```csharp
bool useSystemBrowser = app.IsSystemWebviewAvailable();

authResult = await App.PCA.AcquireTokenInteractive(App.Scopes)
                      .WithParentActivityOrWindow(App.ParentWindow)
                      .WithUseEmbeddedWebView(!useSystemBrowser)
                      .ExecuteAsync();
```

## <a name="net-core-does-not-support-interactive-authentication-out-of-the-box"></a>.NET core no admite la autenticación interactiva de fábrica

Para .NET Core, adquisición de tokens de forma interactiva no está disponible. De hecho, .NET Core no proporciona aún la interfaz de usuario. Si desea proporcionar inicio de sesión interactivo para una aplicación .NET Core, puede permitir que la aplicación presentar al usuario un código y una dirección URL para ir a la sesión de forma interactiva (consulte [flujo de código de dispositivo](msal-authentication-flows.md#device-code)).

Como alternativa puede implementar la [IWithCustomUI](scenario-desktop-acquire-token.md#withcustomwebui) interfaz y proporcionar su propio explorador