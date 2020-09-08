---
title: Configuración de aplicaciones móviles que llaman a las API web | Azure
titleSuffix: Microsoft identity platform
description: Obtenga información sobre cómo compilar una aplicación móvil que llama a las API web (configuración del código de la aplicación)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 06/16/2020
ms.author: jmprieur
ms.custom: aaddev
ms.openlocfilehash: 717b6ed7e711b4db4cb15e55282cfcdea4659178
ms.sourcegitcommit: 8a7b82de18d8cba5c2cec078bc921da783a4710e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89051345"
---
# <a name="configure-a-mobile-app-that-calls-web-apis"></a>Configuración de una aplicación móvil que llama a las API web

Una vez creada la aplicación, aprenderá a configurar el código con los parámetros de registro de la aplicación. Las aplicaciones móviles presentan algunas complejidades relacionadas con la adaptación a su marco de creación.

## <a name="find-msal-support-for-mobile-apps"></a>Búsqueda de compatibilidad con MSAL para aplicaciones móviles

Los siguientes tipos de la Biblioteca de autenticación de Microsoft (MSAL) admiten aplicaciones móviles.

MSAL | Descripción
------------ | ----------
![MSAL.NET](media/sample-v2-code/logo_NET.png) <br/> MSAL.NET  | Se usa para desarrollar aplicaciones portátiles. MSAL.NET admite las siguientes plataformas para compilar una aplicación móvil: Plataforma universal de Windows (UWP), Xamarin.iOS y Xamarin.Android.
![MSAL.iOS](media/sample-v2-code/logo_iOS.png) <br/> MSAL.iOS | Se usa desarrollar aplicaciones de iOS nativas con Objective-C o Swift.
![MSAL.Android](media/sample-v2-code/logo_android.png) <br/> MSAL.Android | Se usa para desarrollar aplicaciones nativas de Android en Java para Android.

## <a name="instantiate-the-application"></a>Creación de una instancia de la aplicación

### <a name="android"></a>Android

Las aplicaciones móviles usan la clase `PublicClientApplication`. Aquí se muestra cómo crear una instancia:

```Java
PublicClientApplication sampleApp = new PublicClientApplication(
                    this.getApplicationContext(),
                    R.raw.auth_config);
```

### <a name="ios"></a>iOS

Las aplicaciones móviles de iOS necesitan crear una instancia de la clase `MSALPublicClientApplication`. Para crear una instancia de la clase, use el código siguiente.

```objc
NSError *msalError = nil;

MSALPublicClientApplicationConfig *config = [[MSALPublicClientApplicationConfig alloc] initWithClientId:@"<your-client-id-here>"];
MSALPublicClientApplication *application = [[MSALPublicClientApplication alloc] initWithConfiguration:config error:&msalError];
```

```swift
let config = MSALPublicClientApplicationConfig(clientId: "<your-client-id-here>")
if let application = try? MSALPublicClientApplication(configuration: config){ /* Use application */}
```

Las [propiedades MSALPublicClientApplicationConfig adicionales](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALPublicClientApplicationConfig.html#/Configuration%20options) pueden invalidar la autoridad predeterminada, especificar un URI de redirección o cambiar el comportamiento del almacenamiento en caché del token de MSAL.

### <a name="xamarin-or-uwp"></a>Xamarin o UWP

En la siguiente sección se explica cómo crear instancias de la aplicación para las aplicaciones de Xamarin.iOS, de Xamarin.Android y para UWP.

#### <a name="instantiate-the-application"></a>Creación de una instancia de la aplicación

En Xamarin o UWP, la manera más sencilla de crear una instancia de la aplicación es mediante el código siguiente. En este código, `ClientId` es el GUID de la aplicación registrada.

```csharp
var app = PublicClientApplicationBuilder.Create(clientId)
                                        .Build();
```

Otros métodos `With<Parameter>` establecen el elemento primario de la interfaz de usuario, invalidan la autoridad predeterminada, especifican un nombre de cliente y una versión para la telemetría, especifican un URI de redirección y especifican el generador HTTP que se va a usar. El generador HTTP podría usarse, por ejemplo, para controlar los servidores proxy y para especificar la telemetría y el registro.

En las secciones siguientes se proporciona más información sobre la creación de instancias de la aplicación.

##### <a name="specify-the-parent-ui-window-or-activity"></a>Especificación de la ventana, la actividad o la interfaz de usuario primaria

En Android, debe pasar la actividad primaria antes de realizar la autenticación interactiva. En iOS, cuando se usa un agente, debe pasar `ViewController`. Del mismo modo en UWP, puede que quiera pasar la ventana primaria. Se pasa al adquirir el token. No obstante, al crear la aplicación, también puede especificar una devolución de llamada como un delegado que devuelve `UIParent`.

```csharp
IPublicClientApplication application = PublicClientApplicationBuilder.Create(clientId)
  .ParentActivityOrWindowFunc(() => parentUi)
  .Build();
```

En Android, se recomienda que use [`CurrentActivityPlugin`](https://github.com/jamesmontemagno/CurrentActivityPlugin). El código del generador `PublicClientApplication` resultante es similar al de este ejemplo:

```csharp
// Requires MSAL.NET 4.2 or above
var pca = PublicClientApplicationBuilder
  .Create("<your-client-id-here>")
  .WithParentActivityOrWindow(() => CrossCurrentActivity.Current)
  .Build();
```

##### <a name="find-more-app-building-parameters"></a>Búsqueda de más parámetros de compilación de aplicaciones

Para obtener una lista de todos los métodos que están disponibles en `PublicClientApplicationBuilder`, consulte la [lista de métodos](/dotnet/api/microsoft.identity.client.publicclientapplicationbuilder#methods).

Para obtener una descripción de todas las opciones que se exponen en `PublicClientApplicationOptions`, consulte la [documentación de referencia](/dotnet/api/microsoft.identity.client.publicclientapplicationoptions).

## <a name="tasks-for-xamarin-ios"></a>Tareas para Xamarin iOS

Si usa MSAL.NET en Xamarin iOS, realice las siguientes tareas.

* [Reemplazo e implementación de la función `OpenUrl` en `AppDelegate`](msal-net-xamarin-ios-considerations.md#implement-openurl)
* [Habilitación de grupos de cadenas de claves](msal-net-xamarin-ios-considerations.md#enable-keychain-access)
* [Habilitación del uso compartido de caché de tokens](msal-net-xamarin-ios-considerations.md#enable-token-cache-sharing-across-ios-applications)
* [Habilitación del acceso a la cadena de claves](msal-net-xamarin-ios-considerations.md#enable-keychain-access)

Para obtener más información, consulte [Consideraciones sobre Xamarin iOS](msal-net-xamarin-ios-considerations.md).

## <a name="tasks-for-msal-for-ios-and-macos"></a>Tareas de MSAL para iOS y macOS

Estas tareas son necesarias cuando se usa MSAL para iOS y macOS:

* [Implementación de la devolución de llamada `openURL`](#brokered-authentication-for-msal-for-ios-and-macos)
* [Habilitación de los grupos de acceso al llavero](howto-v2-keychain-objc.md)
* [Personalización de los exploradores y las vistas web](customize-webviews.md)

## <a name="tasks-for-xamarinandroid"></a>Tareas para Xamarin.Android

Si usa Xamarin.Android, realice las tareas siguientes:

- [Devolución del control a MSAL al finalizar la parte interactiva del flujo de autenticación](msal-net-xamarin-android-considerations.md#ensure-that-control-returns-to-msal)
- [Actualización del manifiesto de Android](msal-net-xamarin-android-considerations.md#update-the-android-manifest)
- [Uso de la vista web incrustada (opcional)](msal-net-xamarin-android-considerations.md#use-the-embedded-web-view-optional)
- [Solución de problemas según sea necesario](msal-net-xamarin-android-considerations.md#troubleshooting)

Para obtener más información, consulte [Consideraciones sobre Xamarin.Android](msal-net-xamarin-android-considerations.md).

Para conocer las consideraciones sobre los exploradores en Android, consulte [Consideraciones específicas de Xamarin.Android con MSAL.NET](msal-net-system-browser-android-considerations.md).

#### <a name="tasks-for-uwp"></a>Tareas para UWP

En UWP, puede usar redes corporativas. En las siguientes secciones se explican las tareas que debe completar en el escenario corporativo.

Para obtener más información, consulte [Consideraciones específicas de UWP con MSAL.NET](msal-net-uwp-considerations.md).

## <a name="configure-the-application-to-use-the-broker"></a>Configuración de la aplicación para usar el agente

En Android e iOS, los agentes permiten:

- **Inicio de sesión único (SSO)** : puede usar el inicio de sesión único (SSO) para dispositivos registrados con Azure Active Directory (Azure AD). Al usar SSO, los usuarios no tienen que iniciar sesión en cada aplicación.
- **Identificación de dispositivos**: esta opción habilita las directivas de acceso condicional relacionadas con los dispositivos de Azure AD. El proceso de autenticación usa el certificado de dispositivo que se creó al unir el dispositivo al área de trabajo.
- **Comprobación de identificación de la aplicación**: cuando una aplicación llama al agente, pasa su URL de redireccionamiento. A continuación, el agente lo comprueba.

### <a name="enable-the-broker-on-xamarin"></a>Habilitación del agente en Xamarin

Para habilitar el agente en Xamarin, use el parámetro `WithBroker()` al llamar al método `PublicClientApplicationBuilder.CreateApplication`. De manera predeterminada, `.WithBroker()` se establece en true.

Para habilitar la autenticación asincrónica para Xamarin.iOS, siga los pasos de la [sección Xamarin.iOS](#enable-brokered-authentication-for-xamarin-ios) de este artículo.

### <a name="enable-the-broker-for-msal-for-android"></a>Habilitación del agente para MSAL para Android

Consulte [Autenticación asincrónica en Android](brokered-auth.md) para obtener información sobre cómo habilitar un agente en Android.

### <a name="enable-the-broker-for-msal-for-ios-and-macos"></a>Habilitación del agente para MSAL para iOS y macOS

La autenticación asincrónica está habilitada de forma predeterminada para los escenarios de Azure AD en MSAL para iOS y macOS.

En las secciones siguientes se proporcionan instrucciones para configurar la aplicación para la compatibilidad con la autenticación asincrónica para MSAL para Xamarin.iOS o MSAL para iOS y macOS. En los dos conjuntos de instrucciones, algunos de los pasos difieren.

### <a name="enable-brokered-authentication-for-xamarin-ios"></a>Habilitación de la autenticación asincrónica para Xamarin iOS

Siga los pasos de esta sección para permitir que la aplicación de Xamarin.iOS pueda hablar con la aplicación [Microsoft Authenticator](https://itunes.apple.com/us/app/microsoft-authenticator/id983156458).

#### <a name="step-1-enable-broker-support"></a>Paso 1: Habilitar la compatibilidad con el agente

La compatibilidad con el agente está deshabilitada de manera predeterminada. Se habilita para una clase `PublicClientApplication` individual. Use el parámetro `WithBroker()` cuando cree la clase `PublicClientApplication` a través de `PublicClientApplicationBuilder`. El parámetro `WithBroker()` se establece en true de forma predeterminada.

```csharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithReplyUri(redirectUriOnIos) // $"msauth.{Bundle.Id}://auth" (see step 6 below)
                .Build();
```

#### <a name="step-2-update-appdelegate-to-handle-the-callback"></a>Paso 2: Actualizar AppDelegate para controlar la devolución de llamada

Cuando MSAL.NET llama al agente, el agente vuelve a llamar a la aplicación. Vuelve a llamar mediante el método `AppDelegate.OpenUrl`. Como MSAL espera la respuesta del agente, la aplicación debe cooperar para volver a llamar a MSAL.NET. Para configurar este comportamiento, actualice el archivo `AppDelegate.cs` para invalidar el método, como se muestra en el código siguiente.

```csharp
public override bool OpenUrl(UIApplication app, NSUrl url,
                             string sourceApplication,
                             NSObject annotation)
{
 if (AuthenticationContinuationHelper.IsBrokerResponse(sourceApplication))
 {
  AuthenticationContinuationHelper.SetBrokerContinuationEventArgs(url);
  return true;
 }
 else if (!AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url))
 {
  return false;
 }
 return true;
}
```

Este método se invoca cada vez que se inicia la aplicación. Es una oportunidad para procesar la respuesta del agente y finalizar el proceso de autenticación iniciado por MSAL.NET.

#### <a name="step-3-set-a-uiviewcontroller"></a>Paso 3: Establecer un UIViewController()

Normalmente, para Xamarin iOS, no es necesario establecer una ventana de objeto. Pero en este caso se debe establecer para poder enviar y recibir respuestas de un agente. Para establecer una ventana de objeto, en `AppDelegate.cs`, deberá establecer un elemento `ViewController`.

Para establecer la ventana de objeto, siga estos pasos:

1. En `AppDelegate.cs`, establezca `App.RootViewController` en un nuevo `UIViewController()`. Esta configuración garantiza que la llamada al agente incluye `UIViewController`. Si no se establece correctamente, puede obtener este error:

    `"uiviewcontroller_required_for_ios_broker":"UIViewController is null, so MSAL.NET cannot invoke the iOS broker. See https://aka.ms/msal-net-ios-broker."`

1. En la llamada `AcquireTokenInteractive`, use `.WithParentActivityOrWindow(App.RootViewController)`. Pase la referencia a la ventana de objeto que va a usar. Este es un ejemplo:

    En `App.cs`:
    ```csharp
       public static object RootViewController { get; set; }
    ```
    En `AppDelegate.cs`:
    ```csharp
       LoadApplication(new App());
       App.RootViewController = new UIViewController();
    ```
    En la llamada `AcquireToken`:
    ```csharp
    result = await app.AcquireTokenInteractive(scopes)
                 .WithParentActivityOrWindow(App.RootViewController)
                 .ExecuteAsync();
    ```

#### <a name="step-4-register-a-url-scheme"></a>Paso 4: Registrar un esquema de dirección URL

MSAL.NET usa direcciones URL para invocar al agente y devolver la respuesta del agente a la aplicación. Para finalizar el recorrido de ida y vuelta, registre el esquema de dirección URL de la aplicación en el archivo `Info.plist`.

Para registrar el esquema de dirección URL de la aplicación, siga estos pasos:

1. Agregue a `CFBundleURLSchemes` el prefijo `msauth`.
1. Agregue `CFBundleURLName` al final. Siga este patrón:

   `$"msauth.(BundleId)"`

   Aquí, `BundleId` identifica su dispositivo de manera exclusiva. Por ejemplo, si `BundleId` es `yourcompany.xforms`, el esquema de dirección URL es `msauth.com.yourcompany.xforms`.

   > [!NOTE]
   > Este esquema de dirección URL se convertirá en parte del URI de redirección que se usa para identificar de forma única la aplicación cuando recibe la respuesta del agente.

   ```XML
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

#### <a name="step-5-add-to-the-lsapplicationqueriesschemes-section"></a>Paso 5: Agregarlo a la sección LSApplicationQueriesSchemes

MSAL usa `–canOpenURL:` para comprobar si el agente está instalado en el dispositivo. En iOS 9, Apple ha bloqueado los esquemas que puede consultar una aplicación.

Agregue `msauthv2` a la sección `LSApplicationQueriesSchemes` del archivo `Info.plist`, como en el ejemplo de código siguiente:

```XML
<key>LSApplicationQueriesSchemes</key>
    <array>
      <string>msauthv2</string>
    </array>
```

### <a name="brokered-authentication-for-msal-for-ios-and-macos"></a>Autenticación asincrónica de MSAL para iOS y macOS

La autenticación asincrónica está habilitada de forma predeterminada para los escenarios de Azure AD.

#### <a name="step-1-update-appdelegate-to-handle-the-callback"></a>Paso 1: Actualizar AppDelegate para controlar la devolución de llamada

Cuando MSAL para iOS y macOS llama al agente, este a su vez vuelve a llamar a la aplicación con el método `openURL`. Dado que MSAL espera la respuesta del agente, la aplicación debe cooperar para volver a llamar a MSAL. Para configurar este comportamiento, actualice el archivo `AppDelegate.m` para invalidar el método, como se muestra en los ejemplos de código siguientes.

```objc
- (BOOL)application:(UIApplication *)app
            openURL:(NSURL *)url
            options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options
{
    return [MSALPublicClientApplication handleMSALResponse:url
                                         sourceApplication:options[UIApplicationOpenURLOptionsSourceApplicationKey]];
}
```

```swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {

        guard let sourceApplication = options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String else {
            return false
        }

        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApplication)
    }
```

> [!NOTE]
> Si adoptó `UISceneDelegate` en iOS 13 o posterior, coloque la devolución de llamada de MSAL en `scene:openURLContexts:` de `UISceneDelegate` en su lugar. MSAL `handleMSALResponse:sourceApplication:` solo debe llamarse una vez para cada dirección URL.
>
> Para más información, consulte la [documentación de Apple](https://developer.apple.com/documentation/uikit/uiscenedelegate/3238059-scene?language=objc).

#### <a name="step-2-register-a-url-scheme"></a>Paso 2: Registrar un esquema de dirección URL

MSAL para iOS y macOS usa direcciones URL para invocar al agente y devolver su respuesta a la aplicación. Para finalizar el recorrido de ida y vuelta, registre un esquema de dirección URL para la aplicación en el archivo `Info.plist`.

Para registrar un esquema para la aplicación:

1. Prefije el esquema de la dirección URL personalizada con `msauth`.

1. Agregue el identificador del paquete al final del esquema. Siga este patrón:

   `$"msauth.(BundleId)"`

   Aquí, `BundleId` identifica su dispositivo de manera exclusiva. Por ejemplo, si `BundleId` es `yourcompany.xforms`, el esquema de dirección URL es `msauth.com.yourcompany.xforms`.

   > [!NOTE]
   > Este esquema de dirección URL se convertirá en parte del URI de redirección que se usa para identificar de forma única la aplicación cuando recibe la respuesta del agente. Asegúrese de que el URI de redirección con el formato `msauth.(BundleId)://auth` está registrado para la aplicación en [Azure Portal](https://portal.azure.com).

   ```XML
   <key>CFBundleURLTypes</key>
   <array>
       <dict>
           <key>CFBundleURLSchemes</key>
           <array>
               <string>msauth.[BUNDLE_ID]</string>
           </array>
       </dict>
   </array>
   ```

#### <a name="step-3-add-lsapplicationqueriesschemes"></a>Paso 3: Agregar LSApplicationQueriesSchemes

Agregue `LSApplicationQueriesSchemes` para permitir llamadas a la aplicación Microsoft Authenticator, si está instalada.

> [!NOTE]
> El esquema `msauthv3` es necesario si la aplicación se ha compilado con Xcode 11 y versiones posteriores.

Este es un ejemplo sobre cómo agregar `LSApplicationQueriesSchemes`:

```XML
<key>LSApplicationQueriesSchemes</key>
<array>
  <string>msauthv2</string>
  <string>msauthv3</string>
</array>
```

### <a name="brokered-authentication-for-xamarinandroid"></a>Autenticación asincrónica para Xamarin.Android

Vea [Autenticación con agente en Xamarin.Android](msal-net-use-brokers-with-xamarin-apps.md#brokered-authentication-for-android) para más información sobre cómo habilitar un agente en Android.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Adquirir un token](scenario-mobile-acquire-token.md)