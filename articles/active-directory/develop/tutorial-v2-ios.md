---
title: 'Tutorial de MSAL para iOS y macOS: Plataforma de identidad de Microsoft | Azure'
description: Aprenda la forma en que las aplicaciones de iOS y macOS (Swift) pueden llamar a una API que requiera tokens de acceso mediante la Plataforma de identidad de Microsoft.
services: active-directory
author: mmacy
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: tutorial
ms.workload: identity
ms.date: 06/16/2020
ms.author: jmprieur
ms.reviewer: oldalton
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 61a3884ce4d1075dbcac61a0c202c3a393d2d943
ms.sourcegitcommit: b8702065338fc1ed81bfed082650b5b58234a702
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88118151"
---
# <a name="sign-in-users-and-call-the-microsoft-graph-from-an-ios-or-macos-app"></a>Inicio de sesión de los usuarios y llamada a Microsoft Graph desde una aplicación para iOS o macOS

En este tutorial, aprenderá a integrar una aplicación para iOS o macOS con la plataforma de identidad de Microsoft. La aplicación iniciará la sesión de un usuario, obtendrá un token de acceso para llamar a Microsoft Graph API y realizará una solicitud a Microsoft Graph API.  

Cuando haya completado la guía, la aplicación podrá aceptar inicios de sesión de cuentas Microsoft personales (como outlook.com, live.com y otras), y cuentas profesionales y educativas de cualquier empresa u organización que utilice Azure Active Directory.

>[!NOTE]
> Si es la primera vez que usa la plataforma de identidad de Microsoft, es aconsejable que comience con [Inicio de sesión de los usuarios y llamada a Microsoft Graph API desde una aplicación de iOS o macOS](quickstart-v2-ios.md).

## <a name="how-this-tutorial-works"></a>Funcionamiento de este tutorial

![Muestra cómo funciona la aplicación de ejemplo generada por este tutorial](../../../includes/media/active-directory-develop-guidedsetup-ios-introduction/iosintro.svg)

La aplicación de este tutorial iniciará la sesión de los usuarios y obtendrá datos en su nombre.  A estos datos se accederá a través de una API protegida (Microsoft Graph API en este caso) que requiere autorización y que está protegida por la plataforma de identidad de Microsoft.

Más concretamente:

* La aplicación iniciará la sesión del usuario mediante un explorador o Microsoft Authenticator.
* El usuario final aceptará los permisos que la aplicación ha solicitado.
* Se emitirá un token de acceso a la aplicación para Microsoft Graph API.
* El token de acceso se incluirán en la solicitud HTTP a la API web.
* Se procesa la respuesta de Microsoft Graph.

Este ejemplo utiliza la Biblioteca de autenticación de Microsoft (MSAL) para implementar la autenticación. MSAL renovará automáticamente los tokens, proporcionará el inicio de sesión único entre las otras aplicaciones del dispositivo y administrará las cuentas.

Este tutorial es aplicable a las aplicaciones para iOS y macOS. Tenga en cuenta que algunos pasos son diferentes entre esas dos plataformas. 

## <a name="prerequisites"></a>Requisitos previos

- En esta guía se requiere la versión 11.x o posterior de XCode para compilar la aplicación. Puede descargar XCode desde [Mac App Store](https://geo.itunes.apple.com/us/app/xcode/id497799835?mt=12 "URL de descarga de XCode").
- Biblioteca de autenticación de Microsoft ([MSAL.framework](https://github.com/AzureAD/microsoft-authentication-library-for-objc)). Puede usar un administrador de dependencias o agregar la biblioteca manualmente. Las instrucciones siguientes muestran cómo hacerlo.

En este tutorial, vamos a crear un nuevo proyecto. Si desea descargar el tutorial completado en su lugar, descargue el código:
- [Código de ejemplo de iOS](https://github.com/Azure-Samples/active-directory-ios-swift-native-v2/archive/master.zip)
- [Código de ejemplo de macOS](https://github.com/Azure-Samples/active-directory-macOS-swift-native-v2/archive/master.zip)

## <a name="create-a-new-project"></a>Creación de un nuevo proyecto

1. Abra Xcode y seleccione **Create a new Xcode project** (Crear un nuevo proyecto en Xcode).
2. En el caso de las aplicaciones para iOS, seleccione **iOS** > **Aplicación de vista única** y seleccione **Siguiente**.
3. En el caso de las aplicaciones para macOS, seleccione **macOS** > **Aplicación de Cocoa** y seleccione **Siguiente**.
4. Especifique un nombre de producto.
5. En **Lenguaje**, seleccione **Swift** y seleccione **Siguiente**.
6. Seleccione una carpeta para crear la aplicación y haga clic en **Crear**.

## <a name="register-your-application"></a>Registrar su aplicación

1. Vaya a [Azure Portal](https://aka.ms/MobileAppReg).
2. Abra la hoja Registros de aplicaciones y haga clic en **+Nuevo registro**.
3. Escriba un **Nombre** para la aplicación y deje sin configurar el URI de redirección.
4. En **Tipos de cuenta compatibles**, seleccione **Cuentas en cualquier directorio organizativo (cualquier directorio de Azure AD: multiinquilino) y cuentas de Microsoft personales (como Skype o Xbox)** .
5. Haga clic en **Registrar**.
6. En la sección **Administrar** del panel que aparece, seleccione **Autenticación**.

7. Haga clic en **Probar la nueva experiencia**  cerca de la parte superior de la pantalla para abrir el registro de aplicaciones y, después, haga clic en **+Nuevo registro** >  **+ Agregar una plataforma** > **iOS/macOS**.
    - Escriba el identificador de agrupación del proyecto. Si descargó el código, es `com.microsoft.identitysample.MSALiOS`. Si va a crear su propio proyecto, seleccione el proyecto en Xcode y abra la pestaña **General**. El identificador de la agrupación aparece en la sección **Identidad**.
8. Haga clic en `Configure` y guarde el valor de **Configuración de MSAL** que aparece en la página **Configuración de MSAL** para que pueda escribirlo cuando configure la aplicación más tarde.  Haga clic en **Done**(Listo).

## <a name="add-msal"></a>Adición de MSAL

Elija una de las siguientes formas de instalar la biblioteca de MSAL en su aplicación:

### <a name="cocoapods"></a>CocoaPods

1. Si usa [CocoaPods](https://cocoapods.org/), instale `MSAL`, para lo que primero debe crear un archivo vacío llamado `podfile` en la misma carpeta que el archivo `.xcodeproj` del proyecto. Agregue lo siguiente a `podfile`:

   ```
   use_frameworks!
   
   target '<your-target-here>' do
      pod 'MSAL'
   end
   ```

2. Reemplace `<your-target-here>` por el nombre del proyecto.
3. En una ventana de terminal, vaya a la carpeta que contiene el archivo `podfile` que ha creado y ejecute `pod install` para instalar la biblioteca MSAL.
4. Cierre Xcode y abra `<your project name>.xcworkspace` para volver a cargar el proyecto en Xcode.

### <a name="carthage"></a>Carthage

Si usa [Carthage](https://github.com/Carthage/Carthage), instale `MSAL`, para lo que debe agregarlo a `Cartfile`:

```
github "AzureAD/microsoft-authentication-library-for-objc" "master"
```

En una ventana de terminal, en el mismo directorio que el archivo `Cartfile` actualizado, ejecute el siguiente comando para que Carthage actualice las dependencias del proyecto.

iOS:

```bash
carthage update --platform iOS
```

macOS:

```bash
carthage update --platform macOS
```

### <a name="manually"></a>Manualmente

También puede usar un submódulo de Git o consultar la versión más reciente y usarla como marco en su aplicación.

## <a name="add-your-app-registration"></a>Adición del registro de aplicaciones

A continuación, agregaremos el registro de aplicaciones al código. 

En primer lugar, agregue la siguiente instrucciones de importación al principio del archivo `ViewController.swift`, así como de los archivos `AppDelegate.swift` o `SceneDelegate.swift`:

```swift
import MSAL
```

Después agregue el siguiente código a `ViewController.swift`, antes de `viewDidLoad()`:

```swift
// Update the below to your client ID you received in the portal. The below is for running the demo only
let kClientID = "Your_Application_Id_Here"
let kGraphEndpoint = "https://graph.microsoft.com/" // the Microsoft Graph endpoint
let kAuthority = "https://login.microsoftonline.com/common" // this authority allows a personal Microsoft account and a work or school account in any organization’s Azure AD tenant to sign in
    
let kScopes: [String] = ["user.read"] // request permission to read the profile of the signed-in user
    
var accessToken = String()
var applicationContext : MSALPublicClientApplication?
var webViewParameters : MSALWebviewParameters?
var currentAccount: MSALAccount?
```

El único valor que debe modificar arriba es el asignado a `kClientID` como [Identificador de aplicación](./developer-glossary.md#application-id-client-id). Este valor forma parte de los datos de Configuración de MSAL que guardó al principio de este tutorial para registrar la aplicación en Azure Portal.

## <a name="configure-xcode-project-settings"></a>Configuración de los valores del proyecto de Xcode

Agregue un nuevo grupo de cadena de claves al proyecto **Signing & Capabilities** (Firma y funcionalidades). El grupo de cadena de claves debe ser `com.microsoft.adalcache` en iOS y `com.microsoft.identity.universalstorage` en macOS.

![Interfaz de usuario de Xcode que muestra cómo se debe configurar el grupo de cadena de claves](../../../includes/media/active-directory-develop-guidedsetup-ios-introduction/iosintro-keychainShare.png)

## <a name="for-ios-only-configure-url-schemes"></a>Solo para iOS, configure los esquemas de dirección URL.

En este paso, registrará `CFBundleURLSchemes` para que se pueda redirigir al usuario a la aplicación después de iniciar sesión. Por cierto, `LSApplicationQueriesSchemes` también permite a la aplicación hacer uso de Microsoft Authenticator.

En Xcode, abra `Info.plist` como un archivo de código fuente y agregue lo siguiente en la sección `<dict>`. Reemplace `[BUNDLE_ID]` por el valor que usó en Azure Portal que, si descargó el código, es `com.microsoft.identitysample.MSALiOS`. Si va a crear su propio proyecto, seleccione el proyecto en Xcode y abra la pestaña **General**. El identificador de la agrupación aparece en la sección **Identidad**.

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>msauth.[BUNDLE_ID]</string>
        </array>
    </dict>
</array>
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>msauthv2</string>
    <string>msauthv3</string>
</array>
```

## <a name="for-macos-only-configure-app-sandbox"></a>Solo para macOS, configure App Sandbox.

1. Vaya a la configuración del proyecto de Xcode > **pestaña Funcionalidad** > **App Sandbox**
2. Active la casilla **Conexiones salientes (cliente)** . 

## <a name="create-your-apps-ui"></a>Creación de la interfaz de usuario de la aplicación

Ahora cree una interfaz de usuario que incluya un botón para llamar a Microsoft Graph API, otro para cerrar la sesión y una vista de texto para parte de la salida agregando el siguiente código a la clase `ViewController`:

### <a name="ios-ui"></a>Interfaz de usuario de iOS

```swift
var loggingText: UITextView!
var signOutButton: UIButton!
var callGraphButton: UIButton!
var usernameLabel: UILabel!

func initUI() {
        
    usernameLabel = UILabel()
    usernameLabel.translatesAutoresizingMaskIntoConstraints = false
    usernameLabel.text = ""
    usernameLabel.textColor = .darkGray
    usernameLabel.textAlignment = .right
        
    self.view.addSubview(usernameLabel)
        
    usernameLabel.topAnchor.constraint(equalTo: view.topAnchor, constant: 50.0).isActive = true
    usernameLabel.rightAnchor.constraint(equalTo: view.rightAnchor, constant: -10.0).isActive = true
    usernameLabel.widthAnchor.constraint(equalToConstant: 300.0).isActive = true
    usernameLabel.heightAnchor.constraint(equalToConstant: 50.0).isActive = true
        
    // Add call Graph button
    callGraphButton  = UIButton()
    callGraphButton.translatesAutoresizingMaskIntoConstraints = false
    callGraphButton.setTitle("Call Microsoft Graph API", for: .normal)
    callGraphButton.setTitleColor(.blue, for: .normal)
    callGraphButton.addTarget(self, action: #selector(callGraphAPI(_:)), for: .touchUpInside)
    self.view.addSubview(callGraphButton)
        
    callGraphButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    callGraphButton.topAnchor.constraint(equalTo: view.topAnchor, constant: 120.0).isActive = true
    callGraphButton.widthAnchor.constraint(equalToConstant: 300.0).isActive = true
    callGraphButton.heightAnchor.constraint(equalToConstant: 50.0).isActive = true
        
    // Add sign out button
    signOutButton = UIButton()
    signOutButton.translatesAutoresizingMaskIntoConstraints = false
    signOutButton.setTitle("Sign Out", for: .normal)
    signOutButton.setTitleColor(.blue, for: .normal)
    signOutButton.setTitleColor(.gray, for: .disabled)
    signOutButton.addTarget(self, action: #selector(signOut(_:)), for: .touchUpInside)
    self.view.addSubview(signOutButton)
        
    signOutButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    signOutButton.topAnchor.constraint(equalTo: callGraphButton.bottomAnchor, constant: 10.0).isActive = true
    signOutButton.widthAnchor.constraint(equalToConstant: 150.0).isActive = true
    signOutButton.heightAnchor.constraint(equalToConstant: 50.0).isActive = true
        
    let deviceModeButton = UIButton()
    deviceModeButton.translatesAutoresizingMaskIntoConstraints = false
    deviceModeButton.setTitle("Get device info", for: .normal);
    deviceModeButton.setTitleColor(.blue, for: .normal);
    deviceModeButton.addTarget(self, action: #selector(getDeviceMode(_:)), for: .touchUpInside)
    self.view.addSubview(deviceModeButton)
        
    deviceModeButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    deviceModeButton.topAnchor.constraint(equalTo: signOutButton.bottomAnchor, constant: 10.0).isActive = true
    deviceModeButton.widthAnchor.constraint(equalToConstant: 150.0).isActive = true
    deviceModeButton.heightAnchor.constraint(equalToConstant: 50.0).isActive = true
        
    // Add logging textfield
    loggingText = UITextView()
    loggingText.isUserInteractionEnabled = false
    loggingText.translatesAutoresizingMaskIntoConstraints = false
        
    self.view.addSubview(loggingText)
        
    loggingText.topAnchor.constraint(equalTo: deviceModeButton.bottomAnchor, constant: 10.0).isActive = true
    loggingText.leftAnchor.constraint(equalTo: self.view.leftAnchor, constant: 10.0).isActive = true
    loggingText.rightAnchor.constraint(equalTo: self.view.rightAnchor, constant: -10.0).isActive = true
    loggingText.bottomAnchor.constraint(equalTo: self.view.bottomAnchor, constant: 10.0).isActive = true
}

func platformViewDidLoadSetup() {
                
    NotificationCenter.default.addObserver(self,
                        selector: #selector(appCameToForeGround(notification:)),
                        name: UIApplication.willEnterForegroundNotification,
                        object: nil)
        
}
    
@objc func appCameToForeGround(notification: Notification) {
    self.loadCurrentAccount()
}

```

### <a name="macos-ui"></a>Interfaz de usuario de macOS

```swift

var callGraphButton: NSButton!
var loggingText: NSTextView!
var signOutButton: NSButton!
    
var usernameLabel: NSTextField!

func initUI() {
        
    usernameLabel = NSTextField()
    usernameLabel.translatesAutoresizingMaskIntoConstraints = false
    usernameLabel.stringValue = ""
    usernameLabel.isEditable = false
    usernameLabel.isBezeled = false
    self.view.addSubview(usernameLabel)
        
    usernameLabel.topAnchor.constraint(equalTo: view.topAnchor, constant: 30.0).isActive = true
    usernameLabel.rightAnchor.constraint(equalTo: view.rightAnchor, constant: -10.0).isActive = true
        
    // Add call Graph button
    callGraphButton  = NSButton()
    callGraphButton.translatesAutoresizingMaskIntoConstraints = false
    callGraphButton.title = "Call Microsoft Graph API"
    callGraphButton.target = self
    callGraphButton.action = #selector(callGraphAPI(_:))
    callGraphButton.bezelStyle = .rounded
    self.view.addSubview(callGraphButton)
        
    callGraphButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    callGraphButton.topAnchor.constraint(equalTo: view.topAnchor, constant: 50.0).isActive = true
    callGraphButton.heightAnchor.constraint(equalToConstant: 34.0).isActive = true
        
    // Add sign out button
    signOutButton = NSButton()
    signOutButton.translatesAutoresizingMaskIntoConstraints = false
    signOutButton.title = "Sign Out"
    signOutButton.target = self
    signOutButton.action = #selector(signOut(_:))
    signOutButton.bezelStyle = .texturedRounded
    self.view.addSubview(signOutButton)
        
    signOutButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
    signOutButton.topAnchor.constraint(equalTo: callGraphButton.bottomAnchor, constant: 10.0).isActive = true
    signOutButton.heightAnchor.constraint(equalToConstant: 34.0).isActive = true
    signOutButton.isEnabled = false
        
    // Add logging textfield
    loggingText = NSTextView()
    loggingText.translatesAutoresizingMaskIntoConstraints = false
        
    self.view.addSubview(loggingText)
        
    loggingText.topAnchor.constraint(equalTo: signOutButton.bottomAnchor, constant: 10.0).isActive = true
    loggingText.leftAnchor.constraint(equalTo: self.view.leftAnchor, constant: 10.0).isActive = true
    loggingText.rightAnchor.constraint(equalTo: self.view.rightAnchor, constant: -10.0).isActive = true
    loggingText.bottomAnchor.constraint(equalTo: self.view.bottomAnchor, constant: -10.0).isActive = true
    loggingText.widthAnchor.constraint(equalToConstant: 500.0).isActive = true
    loggingText.heightAnchor.constraint(equalToConstant: 300.0).isActive = true
}

func platformViewDidLoadSetup() {}

```

A continuación, también en la clase `ViewController`, reemplace el método `viewDidLoad()` por:

```swift
    override func viewDidLoad() {

        super.viewDidLoad()

        initUI()
        
        do {
            try self.initMSAL()
        } catch let error {
            self.updateLogging(text: "Unable to create Application Context \(error)")
        }
        
        self.loadCurrentAccount()
        self.platformViewDidLoadSetup()
    }
```

## <a name="use-msal"></a>Uso de MSAL

### <a name="initialize-msal"></a>Inicializar MSAL

Agregue el siguiente método `initMSAL` a la clase `ViewController`:

```swift
    func initMSAL() throws {
        
        guard let authorityURL = URL(string: kAuthority) else {
            self.updateLogging(text: "Unable to create authority URL")
            return
        }
        
        let authority = try MSALAADAuthority(url: authorityURL)
        
        let msalConfiguration = MSALPublicClientApplicationConfig(clientId: kClientID, redirectUri: nil, authority: authority)
        self.applicationContext = try MSALPublicClientApplication(configuration: msalConfiguration)
        self.initWebViewParams()
    }
```

Agregue lo siguiente después del método `initMSAL` a la clase `ViewController`.

### <a name="ios-code"></a>Código para iOS:

```swift
func initWebViewParams() {
        self.webViewParameters = MSALWebviewParameters(authPresentationViewController: self)
    }
```

### <a name="macos-code"></a>Código para macOS:

```swift
func initWebViewParams() {
        self.webViewParameters = MSALWebviewParameters()
    }
```

### <a name="for-ios-only-handle-the-sign-in-callback"></a>Solo para iOS, controle la devolución de llamada del inicio de sesión

Abra el archivo `AppDelegate.swift` . Para controlar la devolución de llamada después de iniciar sesión, agregue `MSALPublicClientApplication.handleMSALResponse` a la clase `appDelegate` de la siguiente forma:

```swift
// Inside AppDelegate...
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        
        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String)
}

```

**Si usa Xcode 11**, debe colocar la devolución de llamada de MSAL en `SceneDelegate.swift` en su lugar.
Si admite UISceneDelegate y UIApplicationDelegate para la compatibilidad con sistemas operativos iOS anteriores, la devolución de llamada de MSAL debe colocarse en ambos archivos.

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        
        guard let urlContext = URLContexts.first else {
            return
        }
        
        let url = urlContext.url
        let sourceApp = urlContext.options.sourceApplication
        
        MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApp)
    }
```

#### <a name="acquire-tokens"></a>Adquisición de tokens

Ahora, podemos implementar la lógica de procesamiento de la interfaz de usuario de la aplicación y obtener los tokens interactivamente a través de MSAL.

MSAL expone dos métodos principales para obtener tokens: `acquireTokenSilently()` y `acquireTokenInteractively()`: 

- `acquireTokenSilently()` intenta iniciar la sesión de un usuario y obtener tokens sin interacción del usuario, siempre que haya una cuenta. `acquireTokenSilently()` requiere que se indique un valor de `MSALAccount` válido, que se puede recuperar mediante una de las API de enumeración de cuentas de MSAL. En este ejemplo se usa `applicationContext.getCurrentAccount(with: msalParameters, completionBlock: {})` para recuperar la cuenta actual. 

- `acquireTokenInteractively()` siempre muestra la interfaz de usuario al intentar iniciar la sesión del usuario. Puede utilizar las cookies de sesión en el explorador o una cuenta de Microsoft Authenticator para proporcionar una experiencia de inicio de sesión único interactivo.

Agregue el siguiente código a la clase `ViewController`:

```swift
    func getGraphEndpoint() -> String {
        return kGraphEndpoint.hasSuffix("/") ? (kGraphEndpoint + "v1.0/me/") : (kGraphEndpoint + "/v1.0/me/");
    }

    @objc func callGraphAPI(_ sender: AnyObject) {
        
        self.loadCurrentAccount { (account) in
            
            guard let currentAccount = account else {
                
                // We check to see if we have a current logged in account.
                // If we don't, then we need to sign someone in.
                self.acquireTokenInteractively()
                return
            }
            
            self.acquireTokenSilently(currentAccount)
        }
    }

    typealias AccountCompletion = (MSALAccount?) -> Void

    func loadCurrentAccount(completion: AccountCompletion? = nil) {
        
        guard let applicationContext = self.applicationContext else { return }
        
        let msalParameters = MSALParameters()
        msalParameters.completionBlockQueue = DispatchQueue.main
                
        applicationContext.getCurrentAccount(with: msalParameters, completionBlock: { (currentAccount, previousAccount, error) in
            
            if let error = error {
                self.updateLogging(text: "Couldn't query current account with error: \(error)")
                return
            }
            
            if let currentAccount = currentAccount {
                
                self.updateLogging(text: "Found a signed in account \(String(describing: currentAccount.username)). Updating data for that account...")
                
                self.updateCurrentAccount(account: currentAccount)
                
                if let completion = completion {
                    completion(self.currentAccount)
                }
                
                return
            }
            
            self.updateLogging(text: "Account signed out. Updating UX")
            self.accessToken = ""
            self.updateCurrentAccount(account: nil)
            
            if let completion = completion {
                completion(nil)
            }
        })
    }
```

#### <a name="get-a-token-interactively"></a>Obtención de un token interactivamente

El código siguiente obtiene un token por primera vez mediante la creación de un objeto `MSALInteractiveTokenParameters` y la llamada a `acquireToken`. A continuación, agregará código que:

1. Crea `MSALInteractiveTokenParameters` con ámbitos.
2. Llama a `acquireToken()` con los parámetros creados.
3. Controla los errores. Para más información, consulte [la guía de control de errores de MSAL para iOS y macOS](msal-handling-exceptions.md).
4. Controla el caso de éxito.

Agregue el siguiente código a la clase `ViewController` .

```swift
func acquireTokenInteractively() {
        
    guard let applicationContext = self.applicationContext else { return }
    guard let webViewParameters = self.webViewParameters else { return }
        
    // #1
    let parameters = MSALInteractiveTokenParameters(scopes: kScopes, webviewParameters: webViewParameters)
    parameters.promptType = .selectAccount
        
    // #2
    applicationContext.acquireToken(with: parameters) { (result, error) in
            
        // #3
        if let error = error {
                
            self.updateLogging(text: "Could not acquire token: \(error)")
            return
        }
            
        guard let result = result else {
                
            self.updateLogging(text: "Could not acquire token: No result returned")
            return
        }
            
        // #4
        self.accessToken = result.accessToken
        self.updateLogging(text: "Access token is \(self.accessToken)")
        self.updateCurrentAccount(account: result.account)
        self.getContentWithToken()
    }
}    
```


#### <a name="get-a-token-silently"></a>Obtención de un token en silencio

Para adquirir un token actualizado en modo silencioso, agregue el código siguiente a la clase`ViewController`. Crea un objeto `MSALSilentTokenParameters` y llama a `acquireTokenSilent()`:

```swift
    
    func acquireTokenSilently(_ account : MSALAccount!) {
        
        guard let applicationContext = self.applicationContext else { return }
        
        /**
         
         Acquire a token for an existing account silently
         
         - forScopes:           Permissions you want included in the access token received
         in the result in the completionBlock. Not all scopes are
         guaranteed to be included in the access token returned.
         - account:             An account object that we retrieved from the application object before that the
         authentication flow will be locked down to.
         - completionBlock:     The completion block that will be called when the authentication
         flow completes, or encounters an error.
         */
        
        let parameters = MSALSilentTokenParameters(scopes: kScopes, account: account)
        
        applicationContext.acquireTokenSilent(with: parameters) { (result, error) in
            
            if let error = error {
                
                let nsError = error as NSError
                
                // interactionRequired means we need to ask the user to sign-in. This usually happens
                // when the user's Refresh Token is expired or if the user has changed their password
                // among other possible reasons.
                
                if (nsError.domain == MSALErrorDomain) {
                    
                    if (nsError.code == MSALError.interactionRequired.rawValue) {
                        
                        DispatchQueue.main.async {
                            self.acquireTokenInteractively()
                        }
                        return
                    }
                }
                
                self.updateLogging(text: "Could not acquire token silently: \(error)")
                return
            }
            
            guard let result = result else {
                
                self.updateLogging(text: "Could not acquire token: No result returned")
                return
            }
            
            self.accessToken = result.accessToken
            self.updateLogging(text: "Refreshed Access token is \(self.accessToken)")
            self.updateSignOutButton(enabled: true)
            self.getContentWithToken()
        }
    }
```

### <a name="call-the-microsoft-graph-api"></a>Llamar a Microsoft Graph API 

Una vez que tenga un token, la aplicación puede usarlo este valor en el encabezado de HTTP para realizar una solicitud autorizada a Microsoft Graph:

| clave del encabezado    | value                 |
| ------------- | --------------------- |
| Authorization | Portador \<access-token> |

Agregue el siguiente código a la clase `ViewController`:

```swift
    func getContentWithToken() {
        
        // Specify the Graph API endpoint
        let graphURI = getGraphEndpoint()
        let url = URL(string: graphURI)
        var request = URLRequest(url: url!)
        
        // Set the Authorization header for the request. We use Bearer tokens, so we specify Bearer + the token we got from the result
        request.setValue("Bearer \(self.accessToken)", forHTTPHeaderField: "Authorization")
        
        URLSession.shared.dataTask(with: request) { data, response, error in
            
            if let error = error {
                self.updateLogging(text: "Couldn't get graph result: \(error)")
                return
            }
            
            guard let result = try? JSONSerialization.jsonObject(with: data!, options: []) else {
                
                self.updateLogging(text: "Couldn't deserialize result JSON")
                return
            }
            
            self.updateLogging(text: "Result from Graph: \(result))")
            
            }.resume()
    }
```

Consulte [Microsoft Graph API](https://graph.microsoft.com) para más información acerca de Microsoft Graph API.

### <a name="use-msal-for-sign-out"></a>Uso de MSAL para cierre de sesión

A continuación, agregue compatibilidad para el cierre de sesión.

> [!Important]
> Al cerrar sesión con MSAL, se elimina de la aplicación toda la información conocida sobre un usuario, así como al eliminar una sesión activa en el dispositivo, cuando lo permite la configuración de este. Opcionalmente, también puede cerrar la sesión del usuario desde el explorador.

Para agregar la funcionalidad de cierre de sesión, agregue el siguiente código a la clase `ViewController`. 

```swift 
@objc func signOut(_ sender: AnyObject) {
        
        guard let applicationContext = self.applicationContext else { return }
        
        guard let account = self.currentAccount else { return }
        
        do {
            
            /**
             Removes all tokens from the cache for this application for the provided account
             
             - account:    The account to remove from the cache
             */
            
            let signoutParameters = MSALSignoutParameters(webviewParameters: self.webViewParameters!)
            signoutParameters.signoutFromBrowser = false // set this to true if you also want to signout from browser or webview
            
            applicationContext.signout(with: account, signoutParameters: signoutParameters, completionBlock: {(success, error) in
                
                if let error = error {
                    self.updateLogging(text: "Couldn't sign out account with error: \(error)")
                    return
                }
                
                self.updateLogging(text: "Sign out completed successfully")
                self.accessToken = ""
                self.updateCurrentAccount(account: nil)
            })
            
        }
    }
```

### <a name="enable-token-caching"></a>Habilitación del almacenamiento en caché de tokens

De forma predeterminada, MSAL almacena en caché los tokens de la aplicación en el llavero de iOS o macOS. 

Para habilitar el almacenamiento en caché de tokens:
1. Asegúrese de que la aplicación está firmada correctamente.
2. Vaya a la configuración del proyecto de Xcode > **Pestaña Capabilities** (Funcionalidades) > **Enable Keychain Sharing** (Habilitar uso compartido de la cadena de claves)
3. Haga clic en **+** y, en **Keychain Groups** (Grupos de llaveros), escriba el valor siguiente: 3.a Para iOS, escriba `com.microsoft.adalcache` 3.b Para macOS, escriba `com.microsoft.identity.universalstorage`.

### <a name="add-helper-methods"></a>Adición de métodos auxiliares
Agregue los siguientes métodos auxiliares a la clase `ViewController` para completar el ejemplo.

### <a name="ios-ui"></a>Interfaz de usuario de iOS:

``` swift
    
    func updateLogging(text : String) {
        
        if Thread.isMainThread {
            self.loggingText.text = text
        } else {
            DispatchQueue.main.async {
                self.loggingText.text = text
            }
        }
    }
    
    func updateSignOutButton(enabled : Bool) {
        if Thread.isMainThread {
            self.signOutButton.isEnabled = enabled
        } else {
            DispatchQueue.main.async {
                self.signOutButton.isEnabled = enabled
            }
        }
    }
    
    func updateAccountLabel() {
        
        guard let currentAccount = self.currentAccount else {
            self.usernameLabel.text = "Signed out"
            return
        }
        
        self.usernameLabel.text = currentAccount.username
    }
    
    func updateCurrentAccount(account: MSALAccount?) {
        self.currentAccount = account
        self.updateAccountLabel()
        self.updateSignOutButton(enabled: account != nil)
    }
```

### <a name="macos-ui"></a>Interfaz de usuario de macOS:

```swift
    func updateLogging(text : String) {
        
        if Thread.isMainThread {
            self.loggingText.string = text
        } else {
            DispatchQueue.main.async {
                self.loggingText.string = text
            }
        }
    }
    
    func updateSignOutButton(enabled : Bool) {
        if Thread.isMainThread {
            self.signOutButton.isEnabled = enabled
        } else {
            DispatchQueue.main.async {
                self.signOutButton.isEnabled = enabled
            }
        }
    }
    
     func updateAccountLabel() {

         guard let currentAccount = self.currentAccount else {
            self.usernameLabel.stringValue = "Signed out"
            return
        }

        self.usernameLabel.stringValue = currentAccount.username ?? ""
        self.usernameLabel.sizeToFit()
     }

     func updateCurrentAccount(account: MSALAccount?) {
        self.currentAccount = account
        self.updateAccountLabel()
        self.updateSignOutButton(enabled: account != nil)
    }
```

### <a name="for-ios-only-get-additional-device-information"></a>Solo para iOS, obtención de información adicional del dispositivo

Use el código siguiente para leer la configuración actual del dispositivo, incluido el dato de si el dispositivo está configurado como compartido:

```swift
    @objc func getDeviceMode(_ sender: AnyObject) {
        
        if #available(iOS 13.0, *) {
            self.applicationContext?.getDeviceInformation(with: nil, completionBlock: { (deviceInformation, error) in
                
                guard let deviceInfo = deviceInformation else {
                    self.updateLogging(text: "Device info not returned. Error: \(String(describing: error))")
                    return
                }
                
                let isSharedDevice = deviceInfo.deviceMode == .shared
                let modeString = isSharedDevice ? "shared" : "private"
                self.updateLogging(text: "Received device info. Device is in the \(modeString) mode.")
            })
        } else {
            self.updateLogging(text: "Running on older iOS. GetDeviceInformation API is unavailable.")
        }
    }
```

### <a name="multi-account-applications"></a>Aplicaciones de varias cuentas

Esta aplicación se ha creado para un escenario de una única cuenta. MSAL también admite escenarios de varias cuentas, pero requiere algún trabajo adicional por parte de las aplicaciones. Deberá crear una interfaz de usuario para ayudar a los usuarios a seleccionar qué cuenta desean usar para cada acción que requiera tokens. Como alternativa, la aplicación puede implementar un método heurístico para seleccionar qué cuenta se usará consultando todas las cuentas de MSAL. Por ejemplo, consulte la [API](https://azuread.github.io/microsoft-authentication-library-for-objc/Classes/MSALPublicClientApplication.html#/c:objc(cs)MSALPublicClientApplication(im)accountsFromDeviceForParameters:completionBlock:) `accountsFromDeviceForParameters:completionBlock:`.

## <a name="test-your-app"></a>Prueba de la aplicación

### <a name="run-locally"></a>Ejecución en modo local

Cree e implemente la aplicación en un emulador o dispositivo de prueba. Debe ser capaz de iniciar sesión y obtener tokens para Azure AD o cuentas personales de Microsoft.

La primera vez que un usuario inicie sesión en la aplicación, Microsoft Identity le pedirá que acepte los permisos solicitados.  Aunque la mayoría de los usuarios pueden dar su consentimiento, algunos inquilinos de Azure AD tienen el consentimiento del usuario deshabilitado, lo que requiere que los administradores den su consentimiento en nombre de todos los usuarios. Para admitir este escenario, registre los ámbitos de la aplicación en Azure Portal.

Después de iniciar sesión, esta aplicación mostrará los datos devueltos desde el punto de conexión `/me` de Microsoft Graph.

## <a name="get-help"></a>Obtener ayuda

Visite [Ayuda y soporte técnico](./developer-support-help-options.md) si tienes algún problema con este tutorial o con la plataforma de identidad de Microsoft.