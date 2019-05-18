---
title: 'Introducción a iOS: plataforma de identidad de Microsoft | Azure'
description: Aprenda la forma en que las aplicaciones de iOS (Swift) pueden llamar a una API que requiera tokens de acceso mediante la plataforma de identidad de Microsoft.
services: active-directory
documentationcenter: dev-center-name
author: danieldobalian
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 04/26/2019
ms.author: dadobali
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: 06b4ecc151f8402bfb8f05968475b0305737ecd9
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65406477"
---
# <a name="sign-in-users-and-call-the-microsoft-graph-from-an-ios-app"></a>Hacer que los usuarios inicien sesión y llamar a Microsoft Graph desde una aplicación de iOS

En este tutorial, obtendrá información sobre cómo compilar una aplicación iOS y cómo integrarla en la plataforma de identidad de Microsoft. En concreto, esta aplicación iniciará la sesión de un usuario, obtendrá un token de acceso para llamar a Microsoft Graph API y realizará una solicitud básica a Microsoft Graph API.  

Cuando haya completado la guía, la aplicación podrá aceptar inicios de sesión de cuentas Microsoft personales (como outlook.com, live.com y otras), y cuentas profesionales y educativas de cualquier empresa u organización que utilice Azure Active Directory.

## <a name="how-this-guide-works"></a>Funcionamiento de esta guía

![Muestra cómo funciona la aplicación de ejemplo generada por este tutorial](../../../includes/media/active-directory-develop-guidedsetup-ios-introduction/iosintro.svg)

La aplicación de este ejemplo iniciará la sesión de los usuarios y obtendrá datos en su nombre.  El acceso a los datos se realizará a través de una API protegida (Microsoft Graph API en este caso) que requiere autorización y también está protegida por la plataforma de identidad de Microsoft.

Más concretamente:

* La aplicación iniciará la sesión del usuario mediante un explorador o Microsoft Authenticator.
* El usuario final aceptará los permisos que la aplicación ha solicitado. 
* Se emitirá un token de acceso a la aplicación para Microsoft Graph API.
* El token de acceso se incluirán en la solicitud HTTP a la API web.
* Se procesa la respuesta de Microsoft Graph.

Este ejemplo utiliza la biblioteca de Microsoft Authentication (MSAL) para implementar la autenticación. MSAL renovará automáticamente los tokens, proporcionará inicio de sesión único entre las otras aplicaciones del dispositivo y administrará las cuentas.

## <a name="prerequisites"></a>Requisitos previos

- Para el ejemplo que se crea en esta guía, se requiere XCode versión 10.x. Puede descargar XCode desde [el sitio web de iTunes](https://geo.itunes.apple.com/us/app/xcode/id497799835?mt=12 "XCode Download URL").
- Biblioteca de autenticación de Microsoft ([MSAL.framework](https://github.com/AzureAD/microsoft-authentication-library-for-objc)). Puede usar el administrador de dependencias o agregar los elementos manualmente. Hay una sección a continuación con [más información](#add-msal). 

## <a name="set-up-your-project"></a>Configurar su proyecto

En este tutorial, vamos a crear un nuevo proyecto. Si desea descargar el tutorial completado en su lugar, [descargue el código](https://github.com/Azure-Samples/active-directory-ios-swift-native-v2/archive/master.zip).

### <a name="create-a-new-project"></a>Creación de un nuevo proyecto

1. Abra Xcode y seleccione **Create a new Xcode project** (Crear un nuevo proyecto en Xcode).
2. Seleccione **iOS > Single view Application** (Aplicación de vista sencilla) y luego **Next** (Siguiente).
3. Asigne un nombre al producto y seleccione **Next** (Siguiente).
4. Seleccione una carpeta para crear la aplicación y haga clic en *Crear*.

## <a name="register-your-application"></a>Registrar su aplicación 

Puede registrar su aplicación de dos maneras distintas, como se describe en las dos secciones siguientes.

### <a name="register-your-app"></a>Registro de la aplicación

1. Vaya a [Azure Portal](https://aka.ms/MobileAppReg) > Seleccione `New registration`. 
2. Escriba un **nombre** para la aplicación > `Register`. **No establezca un URI de redireccionamiento en esta etapa**. 
3. En la sección `Manage`, vaya a `Authentication` > `Add a platform` > `iOS`.
    - Escriba el identificador de agrupación del proyecto. Si descargó el código, es `com.microsoft.identitysample.MSALiOS`.
4. Presione `Configure` y almacene `MSAL Configuration` para su uso posterior. 

## <a name="add-msal"></a>Adición de MSAL

### <a name="get-msal"></a>Obtención de MSAL

#### <a name="cocoapods"></a>CocoaPods

Puede usar [CocoaPods](https://cocoapods.org/) para instalar `MSAL` agregándolo a su `Podfile` debajo del objetivo:

```
use_frameworks!

target '<your-target-here>' do
   pod 'MSAL', '~> 0.4.0'
end
```

#### <a name="carthage"></a>Carthage

Puede usar [Carthage](https://github.com/Carthage/Carthage) para instalar `MSAL` agregándolo a su `Cartfile`: 

```
github "AzureAD/microsoft-authentication-library-for-objc" "master"
```

#### <a name="manually"></a>Manualmente

También puede usar un submódulo de Git o consultar la última versión y usarla como marco en su aplicación.

### <a name="add-your-app-registration"></a>Adición del registro de aplicaciones

A continuación, agregue el registro de aplicaciones al código. Agregue su ***identificador de cliente/aplicación*** a `ViewController.swift`:

```swift
let kClientID = "Your_Application_Id_Here"

// Additional variables for Auth and Graph API
let kGraphURI = "https://graph.microsoft.com/v1.0/me/"
let kScopes: [String] = ["https://graph.microsoft.com/user.read"]
let kAuthority = "https://login.microsoftonline.com/common"
var accessToken = String()
var applicationContext : MSALPublicClientApplication?
```

### <a name="configure-url-schemes"></a>Configuración de esquemas de direcciones URL

Registre `CFBundleURLSchemes` para permitir una devolución de llamada de manera que el usuario pueda redirigirse a la aplicación después del inicio de sesión.

`LSApplicationQueriesSchemes` permite el uso de la aplicación para usar Microsoft Authenticator, si está disponible. 

Para ello, abra `Info.plist` como origen de código y agregue lo siguiente, reemplazando ***BUNDLE_ID*** por lo que haya configurado en Azure Portal.

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
    <string>msauth</string>
    <string>msauthv2</string>
</array>
```

### <a name="import-msal"></a>Importación de MSAL

Importación del marco de MSAL en los archivos `ViewController.swift` y `AppDelegate.swift`:

```swift
import MSAL
```

## <a name="create-your-apps-ui"></a>Creación de la interfaz de usuario de la aplicación

En este tutorial, necesitará crear:

- Botón para llamar a Graph API
- Botón para cerrar sesión
- Vista de registro de texto

En `ViewController.swift`, defina las propiedades y `initUI()` como sigue:

```swift

var loggingText: UITextView!
var signOutButton: UIButton!
var callGraphButton: UIButton!

func initUI() {
        // Add call Graph button
        callGraphButton  = UIButton()
        callGraphButton.translatesAutoresizingMaskIntoConstraints = false
        callGraphButton.setTitle("Call Microsoft Graph API", for: .normal)
        callGraphButton.setTitleColor(.blue, for: .normal)
        callGraphButton.addTarget(self, action: #selector(callGraphAPI(_:)), for: .touchUpInside)
        self.view.addSubview(callGraphButton)
        
        callGraphButton.centerXAnchor.constraint(equalTo: view.centerXAnchor).isActive = true
        callGraphButton.topAnchor.constraint(equalTo: view.topAnchor, constant: 50.0).isActive = true
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
        signOutButton.isEnabled = false
        
        // Add logging textfield
        loggingText = UITextView()
        loggingText.isUserInteractionEnabled = false
        loggingText.translatesAutoresizingMaskIntoConstraints = false
        
        self.view.addSubview(loggingText)
        
        loggingText.topAnchor.constraint(equalTo: signOutButton.bottomAnchor, constant: 10.0).isActive = true
        loggingText.leftAnchor.constraint(equalTo: self.view.leftAnchor, constant: 10.0).isActive = true
        loggingText.rightAnchor.constraint(equalTo: self.view.rightAnchor, constant: 10.0).isActive = true
        loggingText.bottomAnchor.constraint(equalTo: self.view.bottomAnchor, constant: 10.0).isActive = true
    }
```

A continuación, agregue a `viewDidLoad()` de ViewController.swift:

```swift
    override func viewDidLoad() {
        super.viewDidLoad()
        initUI()
        do {
            try self.initMSAL()
        } catch let error {
            self.loggingText.text = "Unable to create Application Context \(error)"
        }
    }
```

## <a name="use-msal"></a>Uso de MSAL

### <a name="initialize-msal"></a>Inicializar MSAL

En primer lugar, deberá crear `MSALPublicClientApplication` con una instancia de `MSALPublicClientConfiguration` para la aplicación:

```swift
    func initMSAL() throws {
        
        guard let authorityURL = URL(string: kAuthority) else {
            self.loggingText.text = "Unable to create authority URL"
            return
        }
        
        let authority = try MSALAADAuthority(url: authorityURL)
        
        let msalConfiguration = MSALPublicClientApplicationConfig(clientId: kClientID, redirectUri: nil, authority: authority)
        self.applicationContext = try MSALPublicClientApplication(configuration: msalConfiguration)
    }
```

### <a name="handle-the-callback"></a>Control de la devolución de llamada

Para controlar la devolución de llamada después de iniciar sesión, agregue `MSALPublicClientApplication.handleMSALResponse` en `appDelegate`:

```swift
    func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
        
        guard let sourceApplication = options[UIApplication.OpenURLOptionsKey.sourceApplication] as? String else {
            return false
        }
        
        return MSALPublicClientApplication.handleMSALResponse(url, sourceApplication: sourceApplication)
    }
```

#### <a name="acquire-tokens"></a>Adquisición de tokens

Ahora, podemos implementar la lógica de procesamiento de la interfaz de usuario de la aplicación y obtener tokens interactivamente a través de MSAL. 

MSAL expone dos métodos principales para obtener los tokens: `acquireTokenSilently` y `acquireTokenInteractively`.  

`acquireTokenSilently()` trata de iniciar la sesión de un usuario y obtiene tokens sin interacción del usuario si hay una cuenta.

`acquireTokenInteractively` siempre mostrará la interfaz de usuario cuando intente iniciar sesión en el usuario y obtener tokens; sin embargo, puede usar cookies de sesión en el navegador o una cuenta en Microsoft Authenticator para brindar una experiencia interactiva de SSO. 

```swift
    @objc func callGraphAPI(_ sender: UIButton) {
        
        guard let currentAccount = self.currentAccount() else {
            // We check to see if we have a current logged in account.
            // If we don't, then we need to sign someone in.
            acquireTokenInteractively()
            return
        }
        
        acquireTokenSilently(currentAccount)
    }

    func currentAccount() -> MSALAccount? {
        
        guard let applicationContext = self.applicationContext else { return nil }
        
        // We retrieve our current account by getting the first account from cache
        // In multi-account applications, account should be retrieved by home account identifier or username instead
        
        do {
            let cachedAccounts = try applicationContext.allAccounts()
            if !cachedAccounts.isEmpty {
                return cachedAccounts.first
            }
        } catch let error as NSError {
            self.updateLogging(text: "Didn't find any accounts in cache: \(error)")
        }
        
        return nil
    }
```

#### <a name="get-a-token-interactively"></a>Obtención de un token interactivamente

Para adquirir un token por primera vez, deberá crear `MSALInteractiveTokenParameters` y llamar a `acquireToken`.

1. Cree `MSALInteractiveTokenParameters` con ámbitos.
2. Llame a `acquireToken` con los parámetros creados.
3. Controle el error en consecuencia. Para obtener más información, consulte la [guía de control de errores de iOS](https://github.com/AzureAD/microsoft-authentication-library-for-objc/wiki/Error-Handling).
4. Controle el caso de éxito. 

```swift
    func acquireTokenInteractively() {
   
        guard let applicationContext = self.applicationContext else { return }
     // #1    
        let parameters = MSALInteractiveTokenParameters(scopes: kScopes)
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
            self.updateSignOutButton(enabled: true)
            self.getContentWithToken()
        }
    }
```



#### <a name="get-a-token-silently"></a>Obtención de un token en silencio

Para adquirir un token actualizado en modo silencioso, deberá crear `MSALSilentTokenParameters` y llamar a `acquireTokenSilent`:

```swift
    
    func acquireTokenSilently(_ account : MSALAccount!) {
        guard let applicationContext = self.applicationContext else { return }
        let parameters = MSALSilentTokenParameters(scopes: kScopes, account: account)
        
        applicationContext.acquireTokenSilent(with: parameters) { (result, error) in    
            if let error = error {
                let nsError = error as NSError
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

Una vez que tenga un token a través de `self.accessToken`, la aplicación puede usar este valor en el encabezado HTTP para realizar una solicitud autorizada a Microsoft Graph:

| clave del encabezado    | value                 |
| ------------- | --------------------- |
| Autorización | Portador <token_de_acceso> |

Agregue lo siguiente a `ViewController.swift`:

```swift
    func getContentWithToken() {        
        // Specify the Graph API endpoint
        let url = URL(string: kGraphURI)
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

Obtenga más información acerca de [Microsoft Graph API](https://graph.microsoft.com).

### <a name="use-msal-for-sign-out"></a>Uso de MSAL para cierre de sesión

A continuación, vamos a agregar compatibilidad para el cierre de sesión a nuestra aplicación. 

Es importante tener en cuenta que el cierre de sesión con MSAL elimina de esta aplicación toda la información conocida acerca de un usuario, pero el usuario seguirá teniendo una sesión activa en su dispositivo. Si el usuario intenta iniciar sesión nuevamente, puede ver la interacción, pero es posible que no tenga que volver a escribir sus credenciales debido a que la sesión del dispositivo está activa. 

Para agregar el cierre de sesión, copie el siguiente método en su `ViewController.swift`:

```swift 
    @objc func signOut(_ sender: UIButton) {
        
        guard let applicationContext = self.applicationContext else { return }
        
        guard let account = self.currentAccount() else { return }
        
        do {
            
            /**
             Removes all tokens from the cache for this application for the provided account
             
             - account:    The account to remove from the cache
             */
            
            try applicationContext.remove(account)
            self.loggingText.text = ""
            self.signOutButton.isEnabled = false
            
        } catch let error as NSError {
            
            self.updateLogging(text: "Received error signing account out: \(error)")
        }
    }
```

### <a name="enable-token-caching"></a>Habilitación del almacenamiento en caché de tokens

De forma predeterminada, MSAL almacena en caché los tokens de la aplicación en la aplicación Llaveros de iOS. 

Para habilitar el almacenamiento en caché de tokens, vaya a la configuración del proyecto de Xcode `Capabilities tab` > `Enable Keychain Sharing` > Haga clic en `Plus` > Escriba **com.microsoft.adalcache**.

### <a name="add-helper-methods"></a>Adición de métodos auxiliares

Agregue estos métodos auxiliares para completar el ejemplo:

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
```

### <a name="multi-account-applications"></a>Aplicaciones de varias cuentas

Esta aplicación se ha creado para un escenario de una única cuenta. MSAL admite también escenarios de varias cuentas, pero requiere algún trabajo adicional por parte de las aplicaciones. Deberá crear una interfaz de usuario para ayudar al usuario a seleccionar qué cuenta desea usar para cada acción que requiera tokens. Como alternativa, la aplicación puede implementar un método heurístico para seleccionar qué cuenta se usará mediante el método `getAllAccounts(...)`.

## <a name="test-your-app"></a>Prueba de la aplicación

### <a name="run-locally"></a>Ejecución en modo local

Si ha seguido el código anterior, pruebe a compilar e implementar la aplicación en un emulador o dispositivo de prueba. Debe ser capaz de iniciar sesión y obtener tokens para Azure AD o cuentas personales de Microsoft. Después de que un usuario inicie sesión, esta aplicación mostrará los datos devueltos desde el punto de conexión `/me` de Microsoft Graph. 

Si tiene algún problema, no dude abrir una incidencia en este documento o en la biblioteca MSAL y comunicárnoslo. 

### <a name="consent-to-your-app"></a>Consentimiento para su aplicación

La primera vez que un usuario inicie sesión en su aplicación, Microsoft Identity le pedirá que acepte los permisos solicitados.  Si bien la mayoría de los usuarios pueden dar su consentimiento, algunos inquilinos de Azure AD tienen deshabilitado el consentimiento del usuario, lo que requiere que los administradores den su consentimiento en nombre de todos los usuarios.  Para admitir este escenario, asegúrese de registrar los ámbitos de la aplicación en Azure Portal.

## <a name="help-and-support"></a>Ayuda y soporte técnico

¿Ha tenido algún problema con este tutorial o con la plataforma de identidad de Microsoft? Vea la sección de [ayuda y soporte técnico](https://docs.microsoft.com/azure/active-directory/develop/developer-support-help-options).

