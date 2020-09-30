---
title: 'Inicio rápido: adición de llamadas a una aplicación de iOS mediante Azure Communication Services'
description: En este inicio rápido, aprenderá a usar la biblioteca cliente de llamadas de Azure Communication Services para iOS.
author: matthewrobertson
ms.author: marobert
ms.date: 07/24/2020
ms.topic: quickstart
ms.service: azure-communication-services
ms.openlocfilehash: c67440453e5ca8395464369d75bfac418a564764
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90945963"
---
En este inicio rápido, aprenderá a iniciar una llamada mediante la biblioteca cliente de llamadas de Azure Communication Services para iOS.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, debe cumplir los siguientes requisitos previos:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F). 
- Mac con [Xcode](https://go.microsoft.com/fwLink/p/?LinkID=266532), junto con un certificado de desarrollador válido instalado en el Llavero.
- Un recurso de Communication Services implementado. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- Un [token de acceso de usuario](../../access-tokens.md) para su instancia de Azure Communication Services.

## <a name="setting-up"></a>Instalación

### <a name="creating-the-xcode-project"></a>Creación del proyecto de Xcode

En Xcode, cree un nuevo proyecto de iOS y seleccione la plantilla **Aplicación de una vista**. En este tutorial se usa el [marco SwiftUI](https://developer.apple.com/xcode/swiftui/), por lo que debe establecer el **lenguaje** en **Swift** y la **interfaz de usuario** en **SwiftUI**. Durante este inicio rápido, no va a crear pruebas unitarias ni pruebas de interfaz de usuario. No dude en desactivar la opción **Include Unit Tests** (Incluir pruebas unitarias) y desactivar también la opción **Include UI Tests** (Incluir pruebas de interfaz de usuario).

:::image type="content" source="../media/ios/xcode-new-ios-project.png" alt-text="Captura de pantalla que muestra la ventana de creación de un nuevo proyecto en Xcode.":::

### <a name="install-the-package"></a>Instalar el paquete

Agregue la biblioteca cliente de llamadas de Azure Communication Services y sus dependencias (AzureCore.framework and AzureCommunication.framework) al proyecto.

> [!NOTE]
> Con el lanzamiento del SDK de AzureCommunicationCalling, encontrará un script de Bash `BuildAzurePackages.sh`. Al ejecutar `sh ./BuildAzurePackages.sh`, el script le proporcionará la ruta de acceso a los paquetes generados del marco que se deben importar en la aplicación de ejemplo en el paso siguiente. Tenga en cuenta que deberá configurar las herramientas de la línea de comandos de Xcode si no lo ha hecho antes de ejecutar el script: Inicie Xcode, seleccione "Preferencias -> Ubicaciones". Elija la versión de Xcode para las herramientas de la línea de comandos.

1. Descargue la biblioteca cliente de llamadas de Azure Communication Services para iOS.
2. En Xcode, haga clic en el archivo del proyecto y seleccione el destino de compilación para abrir el editor de configuración del proyecto.
3. En la pestaña **General**, desplácese a la sección **Frameworks, Libraries, and Embedded Content** (Marcos, bibliotecas y contenido insertado) y haga clic en el icono **"+"** .
4. En la parte inferior izquierda del cuadro de diálogo, elija **Agregar archivos**, desplácese hasta el directorio **AzureCommunicationCalling.framework** del paquete de la biblioteca cliente sin comprimir.
    1. Repita el último paso para agregar **AzureCore.framework** y **AzureCommunication.framework**.
5. Abra la pestaña **Configuración de la compilación** del editor de configuración del proyecto y desplácese hasta la sección **Rutas de búsqueda**. Agregue la entrada **Framework Search Paths** (Rutas de acceso de búsqueda del marco) para el directorio que contiene **AzureCommunicationCalling.framework**.
    1. Agregue otra entrada de rutas de acceso de búsqueda del marco que apunte a la carpeta que contiene las dependencias.

:::image type="content" source="../media/ios/xcode-framework-search-paths.png" alt-text="Captura de pantalla que muestra cómo actualizar las rutas de acceso de búsqueda del marco en Xcode.":::

### <a name="request-access-to-the-microphone"></a>Solicitud de acceso al micrófono

Para acceder al micrófono del dispositivo, debe actualizar la lista de propiedades de información de la aplicación con el elemento `NSMicrophoneUsageDescription`. Puede establecer el valor asociado al elemento `string` que se incluirá en el cuadro de diálogo que el sistema usa para solicitar acceso al usuario.

Haga clic con el botón derecho en la entrada `Info.plist` del árbol del proyecto y seleccione **Abrir como** > **Código fuente**. Agregue las líneas siguientes a la sección `<dict>` de nivel superior y guarde el archivo.

```xml
<key>NSMicrophoneUsageDescription</key>
<string>Need microphone access for VOIP calling.</string>
```

### <a name="set-up-the-app-framework"></a>Instalación del marco de la aplicación

Abra el archivo **ContentView.swift** del proyecto y agregue una declaración `import` en la parte superior del archivo para importar el elemento `AzureCommunicationCalling library`. Además, importe `AVFoundation`; lo necesitaremos para la solicitud de permiso de audio en el código.

```swift
import AzureCommunicationCalling
import AVFoundation
```

Reemplace la implementación de la estructura `ContentView` por algunos controles de interfaz de usuario simples que permitan a los usuarios iniciar y finalizar una llamada. En este inicio rápido, asociaremos la lógica de negocios a estos controles.

```swift
struct ContentView: View {
    @State var callee: String = ""
    @State var callClient: ACSCallClient?
    @State var callAgent: ACSCallAgent?
    @State var call: ACSCall?

    var body: some View {
        NavigationView {
            Form {
                Section {
                    TextField("Who would you like to call?", text: $callee)
                    Button(action: startCall) {
                        Text("Start Call")
                    }.disabled(callAgent == nil)
                    Button(action: endCall) {
                        Text("End Call")
                    }.disabled(call == nil)
                }
            }
            .navigationBarTitle("Calling Quickstart")
        }.onAppear {
            // Initialize call agent
        }
    }

    func startCall() {
        // Ask permissions
        AVAudioSession.sharedInstance().requestRecordPermission { (granted) in
            if granted {
                // Add start call logic
            }
        }
    }

    func endCall() {
        // Add end call logic
    }
}
```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases e interfaces controlan algunas de las características principales de la biblioteca cliente de llamadas de Azure Communication Services:

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| ACSCallClient | CallClient es el punto de entrada principal a la biblioteca cliente de llamadas.|
| ACSCallClient | CallAgent se usa para iniciar y administrar llamadas. |
| CommunicationUserCredential | CommunicationUserCredential se usa como la credencial de token para crear una instancia de CallAgent.| 
| CommunicationIndentifier | CommunicationIndentifier se usa para representar la identidad del usuario, que puede ser una de las opciones siguientes: CommunicationUser/PhoneNumber/CallingApplication. |

## <a name="authenticate-the-client"></a>Autenticar el cliente

Inicialice una instancia de `CallAgent` con un token de acceso de usuario que nos permita realizar y recibir llamadas. Agregue el código siguiente a la devolución de llamada `onAppear` en **ContentView.swift**:

```swift
var userCredential: CommunicationUserCredential?
do {
    userCredential = try CommunicationUserCredential(token: "<USER ACCESS TOKEN>")
} catch {
    print("ERROR: It was not possible to create user credential.")
    return
}

self.callClient = ACSCallClient()

// Creates the call agent
self.callClient?.createCallAgent(userCredential) { (agent, error) in
    if error != nil {
        print("ERROR: It was not possible to create a call agent.")
    }

    if let agent = agent {
        self.callAgent = agent
        print("Call agent successfully created.")
    }
}
```

Debe reemplazar `<USER ACCESS TOKEN>` por un token de acceso de usuario válido para el recurso. Consulte la documentación de [token de acceso de usuario](../../access-tokens.md) si aún no tiene ningún token disponible.

## <a name="start-a-call"></a>Inicio de una llamada

El método `startCall` se establece como la acción que se llevará a cabo cuando se toque el botón *Iniciar llamada*. Actualice la implementación para iniciar una llamada con `ASACallAgent`:

```swift
func startCall()
{
    // Ask permissions
    AVAudioSession.sharedInstance().requestRecordPermission { (granted) in
        if granted {
            // start call logic
            let callees:[CommunicationIdentifier] = [CommunicationUser(identifier: self.callee)]
            self.call = self.callAgent?.call(callees, options: ACSStartCallOptions())
        }
    }
}
```

También puede usar las propiedades de `ACSStartCallOptions` para establecer las opciones iniciales de la llamada (es decir, permite iniciar la llamada con el micrófono silenciado).

## <a name="end-a-call"></a>Finalización de una llamada

Implemente el método `endCall` para finalizar la llamada actual cuando se toque el botón *Finalizar llamada*.

```swift
func endCall()
{    
    self.call!.hangup(ACSHangupOptions()) { (error) in
        if (error != nil) {
            print("ERROR: It was not possible to hangup the call.")
        }
    }
}
```

## <a name="run-the-code"></a>Ejecución del código

Para compilar y ejecutar la aplicación en el simulador de iOS, seleccione **Producto** > **Ejecutar** o use el método abreviado de teclado (&#8984;-R).

:::image type="content" source="../media/ios/quick-start-make-call.png" alt-text="Aspecto final de la aplicación de inicio rápido":::

Para hacer una llamada de VoIP saliente, proporcione un identificador de usuario en el campo de texto y toque el botón **Iniciar llamada**. La llamada a `8:echo123` le conecta a un bot de eco, lo que resulta ideal como introducción y para verificar que los dispositivos de audio funcionan. 

> [!NOTE]
> La primera vez que realice una llamada, el sistema le solicitará acceso al micrófono. En una aplicación de producción, debe usar la API `AVAudioSession`, [comprobar el estado del permiso](https://developer.apple.com/documentation/uikit/protecting_the_user_s_privacy/requesting_access_to_protected_resources) y actualizar correctamente el comportamiento de la aplicación cuando no se conceda el permiso.

## <a name="sample"></a>Ejemplo

Puede descargar la aplicación de ejemplo de [GitHub](https://github.com/Azure/Communication/tree/master/samples/AzureCommunicationCalling/iOS/Swift).
