---
title: Uso de agentes con Xamarin iOS y Android | Azure
titleSuffix: Microsoft identity platform
description: Aprenda a configurar aplicaciones de Xamarin iOS que puedan usar Microsoft Authenticator y la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET). Obtenga información sobre cómo migrar de la Biblioteca de autenticación de Azure AD para .NET (ADAL.NET) a la Biblioteca de autenticación de Microsoft para .NET (MSAL.NET).
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
ms.openlocfilehash: 7fa13a328a55b0e9eaa546e70bf0711f4f011cf1
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89068547"
---
# <a name="use-microsoft-authenticator-or-intune-company-portal-on-xamarin-applications"></a>Uso de Microsoft Authenticator o el Portal de empresa de Intune en aplicaciones de Xamarin

En iOS y Android, agentes como Microsoft Authenticator y el Portal de empresa de Microsoft Intune específico para Android permiten lo siguiente:

- **Inicio de sesión único (SSO)** : los usuarios no tienen que iniciar sesión en cada aplicación.
- **Identificación de dispositivos**: el agente tiene acceso al certificado del dispositivo. Este certificado se crea en el dispositivo cuando se une al área de trabajo.
- **Comprobación de identificación de la aplicación**: cuando una aplicación llama al agente, pasa su URL de redireccionamiento. El agente comprueba la URL.

Para habilitar una de estas características, use el parámetro `WithBroker()` al llamar al método `PublicClientApplicationBuilder.CreateApplication`. El parámetro `.WithBroker()` se establece en true de forma predeterminada.

La configuración de la autenticación asincrónica en la biblioteca de autenticación de Microsoft para .NET (MSAL.NET) varía según la plataforma:

* [Aplicaciones de iOS](#brokered-authentication-for-ios)
* [Aplicaciones de Android](#brokered-authentication-for-android)

## <a name="brokered-authentication-for-ios"></a>Autenticación asincrónica para iOS

Realice los pasos siguientes para permitir que la aplicación de Xamarin.iOS se comunique con la aplicación [Microsoft Authenticator](https://itunes.apple.com/us/app/microsoft-authenticator/id983156458). Si tiene como destino iOS 13, considere la posibilidad de leer sobre el [cambio importante en la API de Apple](./msal-net-xamarin-ios-considerations.md).

### <a name="step-1-enable-broker-support"></a>Paso 1: Habilitar la compatibilidad con el agente

Debe habilitar la compatibilidad del agente para las instancias individuales de `PublicClientApplication`. La compatibilidad está deshabilitada de manera predeterminada. Al crear `PublicClientApplication` a través de `PublicClientApplicationBuilder`, utilice el parámetro `WithBroker()` como se muestra en el ejemplo siguiente. El parámetro `WithBroker()` se establece en true de forma predeterminada.

```csharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithReplyUri(redirectUriOnIos) // $"msauth.{Bundle.Id}://auth" (see step 6 below)
                .Build();
```

### <a name="step-2-enable-keychain-access"></a>Paso 2: Habilitación de acceso a la cadena de claves

Para habilitar el acceso a la cadena de claves, debe tener un grupo de acceso a la cadena de claves para su aplicación. Puede usar la API de `WithIosKeychainSecurityGroup()` para establecer el grupo de acceso a cadenas de claves al crear la aplicación:

```csharp
var builder = PublicClientApplicationBuilder
     .Create(ClientId)
     .WithIosKeychainSecurityGroup("com.microsoft.adalcache")
     .Build();
```

Para más información, consulte [Habilitación de acceso a la cadena de claves](msal-net-xamarin-ios-considerations.md#enable-keychain-access).

### <a name="step-3-update-appdelegate-to-handle-the-callback"></a>Paso 3: Actualizar AppDelegate para controlar la devolución de llamada

Cuando MSAL.NET llama al agente, este, a su vez, llama de vuelta a la aplicación con el método `OpenUrl` de la clase `AppDelegate`. Dado que MSAL espera la respuesta del agente, la aplicación debe cooperar para volver a llamar a MSAL.NET. Para permitir esta cooperación, actualice el archivo *AppDelegate.cs* para invalidar el método siguiente.

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

Este método se invoca cada vez que se inicia la aplicación. Se usa como una oportunidad para procesar la respuesta del agente y finalizar el proceso de autenticación iniciado por MSAL.NET.

### <a name="step-4-set-uiviewcontroller"></a>Paso 4: Establecer UIViewController()

En el archivo *AppDelegate.cs*, debe establecer una ventana de objeto. Normalmente, no es necesario que establezca la ventana de objeto para Xamarin iOS, pero necesitará una para enviar y recibir respuestas del agente.

Para configurar la ventana de objeto:

1. En el archivo *AppDelegate.cs*, establezca `App.RootViewController` en un nuevo `UIViewController()`. Esta asignación garantiza que la llamada al agente incluye `UIViewController`. Si esta configuración se asigna de forma incorrecta, puede obtener este error:

      `"uiviewcontroller_required_for_ios_broker":"UIViewController is null, so MSAL.NET cannot invoke the iOS broker. See https://aka.ms/msal-net-ios-broker"`

1. En la llamada a `AcquireTokenInteractive`, use `.WithParentActivityOrWindow(App.RootViewController)` y, a continuación, pase la referencia a la ventana de objeto que usará.

    En *App.cs*:

    ```csharp
       public static object RootViewController { get; set; }
    ```

    En *AppDelegate.cs*:

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

### <a name="step-5-register-a-url-scheme"></a>Paso 5: Registrar un esquema de dirección URL

MSAL.NET usa direcciones URL para invocar al agente y devolver la respuesta del agente a la aplicación. Para completar el recorrido de ida y vuelta, registre un esquema de dirección URL para la aplicación en el archivo *Info.plist*.

El nombre `CFBundleURLSchemes` debe incluir el prefijo `msauth.`. El prefijo debe ir seguido de `CFBundleURLName`.

En el esquema de dirección URL, `BundleId` identifica de forma única la aplicación: `$"msauth.(BundleId)"`. Por tanto, si `BundleId` es `com.yourcompany.xforms`, el esquema de dirección URL es `msauth.com.yourcompany.xforms`.

> [!NOTE]
> Este esquema de dirección URL se convierte en parte del URI de redirección que identifica de forma única la aplicación cuando recibe la respuesta del agente.

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

### <a name="step-6-add-the-broker-identifier-to-the-lsapplicationqueriesschemes-section"></a>Paso 6: Agregar el identificador de agente a la sección LSApplicationQueriesSchemes

MSAL usa `–canOpenURL:` para comprobar si el agente está instalado en el dispositivo. En iOS 9, Apple ha bloqueado los esquemas que puede consultar una aplicación.

Agregue `msauthv2` a la sección `LSApplicationQueriesSchemes` del archivo *Info.plist*, como en el ejemplo siguiente:

```XML
<key>LSApplicationQueriesSchemes</key>
    <array>
      <string>msauthv2</string>
      <string>msauthv3</string>
    </array>
```

### <a name="step-7-add-a-redirect-uri-to-your-app-registration"></a>Paso 7: Adición de un identificador URI de redirección al registro de la aplicación

SI usa el agente, se agrega un requisito adicional al URI de redirección. El URI de redirección _debe_ tener el siguiente formato:

```csharp
$"msauth.{BundleId}://auth"
```

Este es un ejemplo:

```csharp
public static string redirectUriOnIos = "msauth.com.yourcompany.XForms://auth";
```

Observe que el URI de redirección coincide con el nombre `CFBundleURLSchemes` que incluyó en el archivo *Info.plist*.

Agregue el URI de redirección al registro de la aplicación en [Azure Portal](https://portal.azure.com). Para generar un URI de redirección con el formato correcto, use **Registros de aplicaciones** en Azure Portal para generar el URI de redirección asincrónica a partir del id. de paquete.

**Para generar el URI de redirección:**

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Azure Active Directory** > **Registros de aplicaciones** > su aplicación registrada.
1. Seleccione **Autenticación** > **Agregar una plataforma** > **iOS/macOS**.
1. Escriba el id. del paquete y, a continuación, seleccione **Configurar**.

    Copie el URI de redirección generado que aparece en el cuadro de texto **URI de redirección** para incluirlo en el código:

    :::image type="content" source="media/msal-net-use-brokers-with-xamarin-apps/portal-01-ios-platform-settings.png" alt-text="Configuración de la plataforma iOS con el URI de redirección generado en Azure Portal":::
1. Seleccione **Listo** para completar la generación del identificador URI de redirección.

## <a name="brokered-authentication-for-android"></a>Autenticación asincrónica para Android

### <a name="step-1-enable-broker-support"></a>Paso 1: Habilitar la compatibilidad con el agente

La compatibilidad con el agente se habilita de forma `PublicClientApplication`. De forma predeterminada, está deshabilitada. Utilice el parámetro `WithBroker()` (establecido en true de forma predeterminada) para crear `IPublicClientApplication` con `PublicClientApplicationBuilder`.

```csharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithRedirectUri(redirectUriOnAndroid) // See step #4
                .Build();
```

### <a name="step-2-update-appdelegate-to-handle-the-callback"></a>Paso 2: Actualizar AppDelegate para controlar la devolución de llamada

Cuando MSAL.NET llame al agente, este, a su vez, volverá a llamar a la aplicación con el método `OnActivityResult()`. Como MSAL espera la respuesta del agente, la aplicación debe cooperar para enrutar el resultado a MSAL.NET.

Enrute el resultado al método `SetAuthenticationContinuationEventArgs(int requestCode, Result resultCode, Intent data)`; para ello, reemplace el método `OnActivityResult()` como se muestra aquí:

```csharp
protected override void OnActivityResult(int requestCode, Result resultCode, Intent data)
{
   base.OnActivityResult(requestCode, resultCode, data);
   AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(requestCode, resultCode, data);
}
```

Este método se invoca cada vez que se inicia la aplicación del agente y se usa como una oportunidad para procesar la respuesta del agente y completar el proceso de autenticación iniciado por MSAL.NET.

### <a name="step-3-set-an-activity"></a>Paso 3: Establecer una actividad

Para habilitar la autenticación asincrónica, establezca una actividad de forma que MSAL pueda enviar y recibir las respuestas del agente. Para ello, proporcione la actividad (normalmente, `MainActivity`) para el objeto primario `WithParentActivityOrWindow(object parent)`.

Por ejemplo, en la llamada a `AcquireTokenInteractive()`:

```csharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow((Activity)context))
             .ExecuteAsync();
```

### <a name="step-4-add-a-redirect-uri-to-your-app-registration"></a>Paso 4: Adición de un identificador URI de redirección al registro de la aplicación

MSAL usa las direcciones URL para invocar al agente y regresar a la aplicación. Para completar este recorrido de ida y vuelta, debe registrar un **URI de redirección** para la aplicación mediante [Azure Portal](https://portal.azure.com).

El formato del URI de redirección para la aplicación depende del certificado que se use para firmar el archivo APK. Por ejemplo:

```
msauth://com.microsoft.xforms.testApp/hgbUYHVBYUTvuvT&Y6tr554365466=
```

La última parte del URI, `hgbUYHVBYUTvuvT&Y6tr554365466=`, es la versión codificada en Base64 de la firma con la que se ha firmado el APK. Durante el desarrollo de la aplicación en Visual Studio, si está depurando el código sin firmar el APK con un certificado específico, Visual Studio lo firmará automáticamente con fines de depuración. Cuando Visual Studio firma el APK de esta manera, le asigna una firma única para la máquina en la que se compiló. Por lo tanto, cada vez que compile la aplicación en otra máquina, deberá actualizar el URI de redireccionamiento en el código de la aplicación y el registro de la aplicación de Azure Portal para poder autenticarse con MSAL.

Durante la depuración, es posible que se produzca una excepción de MSAL (o un mensaje de registro) que indique que el URI de redireccionamiento proporcionado no es correcto. **Esta excepción o mensaje de registros también le indica el URI de redirección que debe usar** con la máquina actual en la que está realizando la depuración. Puede usar el URI de redirección proporcionado para seguir desarrollando la aplicación siempre que actualice el URI de redirección en el código y agregue el URI de redirección proporcionado al registro de la aplicación en Azure Portal.

Una vez que esté preparado para finalizar el código, actualice el URI de redireccionamiento en el código y en el registro de la aplicación de Azure Portal para usar la firma del certificado con el que se firma el APK.

En la práctica, esto significa que debería considerar la posibilidad de agregar un URI de redireccionamiento para cada miembro del equipo de desarrollo *y* otro para la versión del APK firmada para producción.

Puede procesar esta firma usted mismo, del mismo modo que lo hace MSAL:

```csharp
   private string GetRedirectUriForBroker()
   {
      string packageName = Application.Context.PackageName;
      string signatureDigest = this.GetCurrentSignatureForPackage(packageName);
      if (!string.IsNullOrEmpty(signatureDigest))
      {
            return string.Format(CultureInfo.InvariantCulture, "{0}://{1}/{2}", RedirectUriScheme,
               packageName.ToLowerInvariant(), signatureDigest);
      }

      return string.Empty;
   }

   private string GetCurrentSignatureForPackage(string packageName)
   {
            PackageInfo info = Application.Context.PackageManager.GetPackageInfo(packageName,
               PackageInfoFlags.Signatures);
            if (info != null && info.Signatures != null && info.Signatures.Count > 0)
            {
               // First available signature. Applications can be signed with multiple signatures.
               // The order of Signatures is not guaranteed.
               Signature signature = info.Signatures[0];
               MessageDigest md = MessageDigest.GetInstance("SHA");
               md.Update(signature.ToByteArray());
               return Convert.ToBase64String(md.Digest(), Base64FormattingOptions.None);
               // Server side needs to register all other tags. ADAL will
               // send one of them.
            }
   }
```

Si quiere, también puede adquirir la firma del paquete mediante **keytool** con los siguientes comandos:

* Windows:
    ```console
    keytool.exe -list -v -keystore "%LocalAppData%\Xamarin\Mono for Android\debug.keystore" -alias androiddebugkey -storepass android -keypass android
    ````
* macOS:
    ```console
    keytool -exportcert -alias androiddebugkey -keystore ~/.android/debug.keystore | openssl sha1 -binary | openssl base64
    ````

### <a name="step-5-optional-fall-back-to-the-system-browser"></a>Paso 5 (opcional): Reversión al explorador del sistema

Si MSAL está configurado para usar el agente pero este no está instalado, MSAL revertirá al uso de una vista web (un explorador). MSAL intentará realizar la autenticación con el explorador del sistema predeterminado en el dispositivo, lo que produce un error porque el URI de redirección está configurado para el agente y el explorador del sistema no sabe cómo usarlo para regresar a MSAL. Para evitar el error, puede configurar un *filtro de intención* con el URI de redirección del agente que usó en el paso 4.

Modifique el manifiesto de la aplicación para agregar el filtro de intención:

```xml
<!-- NOTE the SLASH (required) that prefixes the signature value in the path attribute.
     The signature value is the Base64-encoded signature discussed above. -->
<intent-filter>
      <data android:scheme="msauth"
                    android:host="Package Name"
                    android:path="/Package Signature"/>
```

Por ejemplo, si tiene un URI de redirección de `msauth://com.microsoft.xforms.testApp/hgbUYHVBYUTvuvT&Y6tr554365466=`, el manifiesto debe tener un aspecto similar al siguiente fragmento de código XML.

La barra diagonal (`/`) delante de la firma en el valor `android:path` es **obligatoria**.

```xml
<!-- NOTE the SLASH (required) that prefixes the signature value in the path attribute.
     The signature value is the Base64-encoded signature discussed above. -->
<intent-filter>
      <data android:scheme="msauth"
                    android:host="com.microsoft.xforms.testApp"
                    android:path="/hgbUYHVBYUTvuvT&Y6tr554365466="/>
```

También puede configurar MSAL para revertir al explorador incrustado, que no depende de un URI de redirección:

```csharp
.WithUseEmbeddedWebUi(true)
```

## <a name="troubleshooting-tips-for-android-brokered-authentication"></a>Sugerencias para la solución de problemas de autenticación asincrónica de Android

A continuación se ofrecen algunas sugerencias sobre cómo evitar problemas al implementar la autenticación asincrónica en Android:

- **URI de redirección**: agregue un URI de redirección al registro de la aplicación en [Azure Portal](https://portal.azure.com/). Un URI de redirección faltante o incorrecto es un problema común que los desarrolladores experimentan.
- **Versión de agente**: instale la versión mínima necesaria de las aplicaciones de agente. Cualquiera de estas dos aplicaciones se puede usar para la autenticación asincrónica en Android.
  - [Portal de empresa de Intune](https://play.google.com/store/apps/details?id=com.microsoft.windowsintune.companyportal) (versión 5.0.4689.0 o superior).
  - [Microsoft Authenticator](https://play.google.com/store/apps/details?id=com.azure.authenticator) (versión 6.2001.0140 o superior).
- **Precedencia de agente**: MSAL se comunica con el *primer agente instalado* en el dispositivo cuando se instalan varios.

    Ejemplo: Si primero instala Microsoft Authenticator y, a continuación, instala Portal de empresa de Intune, la autenticación asincrónica *solo* se produce en Microsoft Authenticator.
- **Registros**: si encuentra un problema con la autenticación asincrónica, ver los registros del agente podría ayudarle a diagnosticar la causa.
  - Vea los registros de Microsoft Authenticator:

    1. Seleccione el botón de menú en la esquina superior derecha de la aplicación.
    1. Seleccione **Ayuda** > **Enviar registros** > **Ver registros**.
    1. Seleccione **Copiar todo** para copiar los registros del agente en el Portapapeles del dispositivo.

    La mejor manera de depurar con estos registros consiste en enviarlos por correo electrónico y verlos en el equipo de desarrollo. Quizá le resulte más fácil analizar los registros en el equipo, en lugar de hacerlo en el dispositivo que los genera. También puede usar un editor de prueba en Android para guardar los registros como un archivo de texto y, a continuación, usar un cable USB para copiar el archivo en un equipo.

  - Vea los registros del Portal de empresa de Intune:

    1. Seleccione el botón de menú en la esquina superior izquierda de la aplicación.
    1. Seleccione **Configuración** > **Datos de diagnóstico**.
    1. Seleccione **Copiar registros** para copiar los registros del agente en la tarjeta SD del dispositivo.
    1. Conecte el dispositivo a un equipo mediante un cable USB para ver los registros en el equipo de desarrollo.

    Una vez que tenga los registros, podrá buscar los intentos de autenticación en ellos mediante el id. de correlación. El id. de correlación se adjunta a todas las solicitudes de autenticación. Para buscar los errores que ha devuelto el punto de conexión de autenticación de la plataforma de identidad de Microsoft, busque `AADSTS`.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información, consulte [Consideraciones para usar la Plataforma universal de Windows con MSAL.NET](msal-net-uwp-considerations.md).
