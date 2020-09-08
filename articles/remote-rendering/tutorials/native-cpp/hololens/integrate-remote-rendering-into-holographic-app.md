---
title: Integración de Remote Rendering en una aplicación holográfica de C++/DirectX11
description: Aquí se explica cómo integrar Remote Rendering en una aplicación holográfica de C++/DirectX11 sin formato creada con el asistente para proyectos de Visual Studio.
author: florianborn71
ms.author: flborn
ms.date: 05/04/2020
ms.topic: tutorial
ms.openlocfilehash: 9457323b2642d0e7c5a623c13ec8440520bd5b8b
ms.sourcegitcommit: c6b9a46404120ae44c9f3468df14403bcd6686c1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88891783"
---
# <a name="tutorial-integrate-remote-rendering-into-a-hololens-holographic-app"></a>Tutorial: Integración de Remote Rendering en una aplicación de HoloLens Holographic

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Uso de Visual Studio para crear una aplicación holográfica que se puede implementar en HoloLens
> * Agregue los fragmentos de código y la configuración de proyecto necesarios para combinar la representación local con contenido representado de forma remota.

Este tutorial se centra en agregar los fragmentos necesarios a un ejemplo nativo de `Holographic App` para combinar la representación local con Azure Remote Rendering. El único tipo de comentarios de estado en esta aplicación se ofrece mediante el panel de salida de depuración de Visual Studio, por lo que se recomienda iniciar el ejemplo desde Visual Studio. La adición de comentarios dentro de la aplicación queda fuera del ámbito de este ejemplo, porque la creación de un panel de texto dinámico desde cero implica una gran cantidad de código. Un buen punto de partida es la clase `StatusDisplay`, que forma parte del [proyecto de ejemplo de Remoting Player en GitHub](https://github.com/microsoft/MixedReality-HolographicRemoting-Samples/tree/master/player/common/Content). De hecho, la versión predefinida de este tutorial usa una copia local de esa clase.

> [!TIP]
> El [repositorio de ejemplos de Azure Remote Rendering](https://github.com/Azure/azure-remote-rendering) contiene el resultado de este tutorial como un proyecto de Visual Studio que está listo para usarse. También se enriquece con informes de estado y de error adecuados mediante la clase de interfaz de usuario `StatusDisplay`. En el tutorial, todas las adiciones específicas de Azure Remote Rendering se encuentran en el ámbito de `#ifdef USE_REMOTE_RENDERING` / `#endif`, por lo que es fácil identificar las adiciones de Remote Rendering.

## <a name="prerequisites"></a>Requisitos previos

Para este tutorial, necesitará:

* La información de su cuenta (identificador de cuenta, clave de cuenta, identificador de suscripción). Si no tiene una cuenta, [cree una](../../../how-tos/create-an-account.md).
* Windows SDK 10.0.18362.0 [(descargar)](https://developer.microsoft.com/windows/downloads/windows-10-sdk).
* La versión más reciente de Visual Studio 2019 [(descargar)](https://visualstudio.microsoft.com/vs/older-downloads/).
* [Las herramientas de Visual Studio para Mixed Reality](https://docs.microsoft.com/windows/mixed-reality/install-the-tools). En concreto, las instalaciones de *carga de trabajo* siguientes son obligatorias:
  * **Desarrollo para el escritorio con C++**
  * **Desarrollo de Plataforma universal de Windows (UWP)**
* Las plantillas de aplicación de Windows Mixed Reality para Visual Studio [(descargar)](https://marketplace.visualstudio.com/items?itemName=WindowsMixedRealityteam.WindowsMixedRealityAppTemplatesVSIX).

## <a name="create-a-new-holographic-app-sample"></a>Creación de una nueva aplicación holográfica de ejemplo

Como primer paso, se crea un inventario de ejemplo que constituye la base de la integración de Remote Rendering. Abra Visual Studio, seleccione "Crear un proyecto" y busque "Holographic DirectX 11 App (Universal Windows) (C++/WinRT)".

![Creación de un proyecto](media/new-project-wizard.png)

Escriba el nombre de proyecto que desee, elija una ruta de acceso y seleccione el botón "Crear".
En el nuevo proyecto, cambie la configuración a **"Debug / ARM64"** . Ahora podrá compilarlo e implementarlo en un dispositivo HoloLens 2 conectado. Si lo ejecuta en HoloLens, verá un cubo girando delante.

## <a name="add-remote-rendering-dependencies-through-nuget"></a>Adición de dependencias de Remote Rendering mediante NuGet

El primer paso para agregar funcionalidades de Remote Rendering es agregar las dependencias del lado cliente. Las dependencias pertinentes están disponibles como un paquete NuGet.
En el Explorador de soluciones, haga clic con el botón derecho en el nombre del proyecto y seleccione **Administrar paquetes NuGet…** en el menú contextual.

En el cuadro de diálogo solicitado, busque el paquete NuGet denominado **"Microsoft.Azure.RemoteRendering.Cpp"** :

![Búsqueda del paquete NuGet](media/add-nuget.png)

y agréguelo al proyecto seleccionando el paquete y, a continuación, presionando el botón "Instalar".

El paquete NuGet agrega las dependencias de Remote Rendering al proyecto. Concretamente:
* Vincule la biblioteca cliente (RemoteRenderingClient.lib).
* Configure las dependencias .dll.
* Establezca la ruta de acceso correcta al directorio include.

## <a name="project-preparation"></a>Preparación del proyecto

Necesitamos realizar pequeños cambios en el proyecto existente. Estos cambios son mínimos, pero sin ellos Remote Rendering no funcionaría.

### <a name="enable-multithread-protection-on-directx-device"></a>Habilitar la protección multiproceso en el dispositivo de DirectX
El dispositivo `DirectX11` debe tener habilitada la protección multiproceso. Para cambiar esa configuración, abra el archivo DeviceResources.cpp de la carpeta "Common" e inserte el código siguiente al final de la función `DeviceResources::CreateDeviceResources()`:

```cpp
// Enable multi thread protection as now multiple threads use the immediate context.
Microsoft::WRL::ComPtr<ID3D11Multithread> contextMultithread;
if (context.As(&contextMultithread) == S_OK)
{
    contextMultithread->SetMultithreadProtected(true);
}
```

### <a name="enable-network-capabilities-in-the-app-manifest"></a>Habilitar las funcionalidades de red en el manifiesto de la aplicación
Las funcionalidades de red se deben habilitar explícitamente para la aplicación implementada. Si esto no se configura, las consultas de conexión acabarán agotando el tiempo de espera. Para habilitarlas, haga doble clic en el elemento `package.appxmanifest` del Explorador de soluciones. En la siguiente interfaz de usuario, vaya a la pestaña **Funcionalidades** y seleccione:
* Internet (cliente y servidor)
* Internet (cliente)

![Funcionalidades de red](media/appx-manifest-caps.png)


## <a name="integrate-remote-rendering"></a>Integración de Remote Rendering

Ahora que el proyecto está preparado, podemos empezar con el código. Un buen punto de entrada en la aplicación es la clase `HolographicAppMain` (archivo HolographicAppMain.h/cpp) porque tiene todos los enlaces necesarios para la inicialización, la desinicialización y la representación.

### <a name="includes"></a>Includes

En primer lugar, agregaremos las instrucciones include necesarias. Agregue la siguiente instrucción include al archivo HolographicAppMain.h:

```cpp
#include <AzureRemoteRendering.h>
```

...y estas directivas `include` adicionales al archivo HolographicAppMain.cpp:

```cpp
#include <AzureRemoteRendering.inl>
#include <RemoteRenderingExtensions.h>
#include <windows.perception.spatial.h>
```

Para simplificar el código, definimos el siguiente acceso directo al espacio de nombres en la parte superior del archivo HolographicAppMain.h, después de las directivas `include`:

```cpp
namespace RR = Microsoft::Azure::RemoteRendering;
```

Este acceso directo es útil, ya que no tenemos que escribir el espacio de nombres completo en todas partes pero podremos reconocer las estructuras de datos específicas de Azure Remote Rendering. Por supuesto, también podríamos usar la directiva `using namespace...`.

### <a name="remote-rendering-initialization"></a>Inicialización de Remote Rendering
 
Necesitamos almacenar algunos objetos para la sesión durante la vigencia de la aplicación. La vigencia coincide con la duración del objeto `HolographicAppMain` de la aplicación, por lo que agregaremos los objetos como miembros a la clase `HolographicAppMain`. El paso siguiente consiste en agregar los siguientes miembros de clase al archivo HolographicAppMain.h:

```cpp
class HolographicAppMain
{
    ...
    // members:
    std::string m_sessionOverride;                // if we have a valid session ID, we specify it here. Otherwise a new one is created
    RR::ApiHandle<RR::AzureFrontend> m_frontEnd;  // the front end instance
    RR::ApiHandle<RR::AzureSession> m_session;    // the current remote rendering session
    RR::ApiHandle<RR::RemoteManager> m_api;       // the API instance, that is used to perform all the actions. This is just a shortcut to m_session->Actions()
    RR::ApiHandle<RR::GraphicsBindingWmrD3d11> m_graphicsBinding; // the graphics binding instance
}
```

Un buen lugar para realizar la implementación real es el constructor de la clase `HolographicAppMain`. Aquí debemos realizar tres tipos de inicialización:
1. Inicialización única del sistema de Remote Rendering
1. Creación del front-end
1. Creación de la sesión

Todo ello se realiza secuencialmente en el constructor. Sin embargo, en casos de uso reales, podría ser adecuado realizar estos pasos por separado.

Agregue el código siguiente al principio del cuerpo del constructor en el archivo HolographicAppMain.cpp:

```cpp
HolographicAppMain::HolographicAppMain(std::shared_ptr<DX::DeviceResources> const& deviceResources) :
    m_deviceResources(deviceResources)
{
    // 1. One time initialization
    {
        RR::RemoteRenderingInitialization clientInit;
        clientInit.connectionType = RR::ConnectionType::General;
        clientInit.graphicsApi = RR::GraphicsApiType::WmrD3D11;
        clientInit.toolId = "<sample name goes here>"; // <put your sample name here>
        clientInit.unitsPerMeter = 1.0f;
        clientInit.forward = RR::Axis::Z_Neg;
        clientInit.right = RR::Axis::X;
        clientInit.up = RR::Axis::Y;
        if (RR::StartupRemoteRendering(clientInit) != RR::Result::Success)
        {
            // something fundamental went wrong with the initialization
            throw std::exception("Failed to start remote rendering. Invalid client init data.");
        }
    }


    // 2. Create front end
    {
        // Users need to fill out the following with their account data and model
        RR::AzureFrontendAccountInfo init;
        init.AccountId = "00000000-0000-0000-0000-000000000000";
        init.AccountKey = "<account key>";
        init.AccountDomain = "westus2.mixedreality.azure.com"; // <change to your region>
        m_modelURI = "builtin://Engine";
        m_sessionOverride = ""; // If there is a valid session ID to re-use, put it here. Otherwise a new one is created

        m_frontEnd = RR::ApiHandle(RR::AzureFrontend(init));
    }

    // 3. Open/create rendering session
    {
        bool createNewSession = true;

        // If we had an old (valid) session that we can recycle, we call synchronous function m_frontEnd->OpenRenderingSession
        if (!m_sessionOverride.empty())
        {
            auto openSessionRes = m_frontEnd->OpenRenderingSession(m_sessionOverride);
            if (openSessionRes->valid())
            {
                SetNewSession(*openSessionRes);
                createNewSession = false;
            }
        }

        if (createNewSession)
        {
            // create a new session
            RR::RenderingSessionCreationParams init;
            init.MaxLease.minute = 10; // session is leased for 10 minutes
            init.Size = RR::RenderingSessionVmSize::Standard;
            auto createSessionAsync = *m_frontEnd->CreateNewRenderingSessionAsync(init);
            createSessionAsync->Completed([&](auto handler)
                {
                    if (handler->GetStatus() == RR::Result::Success)
                    {
                        SetNewSession(handler->GetResult());
                    }
                    else
                    {
                        SetNewState(AppConnectionStatus::ConnectionFailed, "failed");
                    }
                });
            SetNewState(AppConnectionStatus::CreatingSession, nullptr);
        }
    }

    // Rest of constructor code:
    ...
}
```

El código llama a las funciones miembro `SetNewSession` y `SetNewState`, que se implementarán en el párrafo siguiente junto con el resto del código máquina de estado.

Tenga en cuenta que las credenciales están codificadas de forma rígida en el ejemplo y que deben rellenarse ([identificador de cuenta, clave de cuenta](../../../how-tos/create-an-account.md#retrieve-the-account-information) y [dominio](../../../reference/regions.md)).

La desinicialización se realiza de forma simétrica y en orden inverso al final del cuerpo del destructor:

```cpp
HolographicAppMain::~HolographicAppMain()
{
    // Existing destructor code:
    ...
    
    // Destroy session:
    if (m_session != nullptr)
    {
        m_session->DisconnectFromRuntime();
        m_session = nullptr;
    }

    // Destroy front end:
    m_frontEnd = nullptr;

    // One-time de-initialization:
    RR::ShutdownRemoteRendering();
}
```

## <a name="state-machine"></a>Máquina de estados

En Remote Rendering, las funciones clave para crear una sesión y cargar un modelo son funciones asincrónicas. Por ello, se necesita una máquina de estado simple que, esencialmente, pase por los siguientes estados automáticamente:

*Inicialización > Creación de sesión > Inicio de sesión > Carga de modelos (con progreso)*

En consecuencia, en el paso siguiente, se agrega un fragmento de control de la máquina de estados a la clase. Declaramos nuestra propia enumeración `AppConnectionStatus` para los distintos estados en los que la aplicación puede encontrarse. Es similar a `RR::ConnectionStatus`, pero tiene un estado adicional para una conexión con errores.

Agregue los siguientes miembros y funciones a la declaración de clase:

```cpp
namespace HolographicApp
{
    // Our application's possible states:
    enum class AppConnectionStatus
    {
        Disconnected,

        CreatingSession,
        StartingSession,
        Connecting,
        Connected,

        // error state:
        ConnectionFailed,
    };

    class HolographicAppMain
    {
        ...
        // Member functions for state transition handling
        void OnConnectionStatusChanged(RR::ConnectionStatus status, RR::Result error);
        void SetNewState(AppConnectionStatus state, const char* statusMsg);
        void SetNewSession(RR::ApiHandle<RR::AzureSession> newSession);
        void StartModelLoading();

        // Members for state handling:

        // Model loading:
        std::string m_modelURI;
        RR::ApiHandle<RR::LoadModelAsync> m_loadModelAsync;

        // Connection state machine:
        AppConnectionStatus m_currentStatus = AppConnectionStatus::Disconnected;
        std::string m_statusMsg;
        RR::Result m_connectionResult = RR::Result::Success;
        RR::Result m_modelLoadResult = RR::Result::Success;
        bool m_isConnected = false;
        bool m_sessionStarted = false;
        RR::ApiHandle<RR::SessionPropertiesAsync> m_sessionPropertiesAsync;
        bool m_modelLoadTriggered = false;
        float m_modelLoadingProgress = 0.f;
        bool m_modelLoadFinished = false;
        double m_timeAtLastRESTCall = 0;
        bool m_needsCoordinateSystemUpdate = true;
    }
```

En el lado de implementación en el archivo .cpp, agregue estos cuerpos de función:

```cpp
void HolographicAppMain::StartModelLoading()
{
    m_modelLoadingProgress = 0.f;

    RR::LoadModelFromSASParams params;
    params.ModelUrl = m_modelURI.c_str();
    params.Parent = nullptr;

    // Start the async model loading
    if (auto loadModel = m_api->LoadModelFromSASAsync(params))
    {
        m_loadModelAsync = *loadModel;
        m_loadModelAsync->Completed([this](const RR::ApiHandle<RR::LoadModelAsync>& async)
        {
            m_modelLoadResult = async->GetStatus();
            m_modelLoadFinished = true; // successful if m_modelLoadResult==RR::Result::Success
            m_loadModelAsync = nullptr;
            char buffer[1024];
            sprintf_s(buffer, "Remote Rendering: Model loading completed. Result: %s\n", RR::ResultToString(m_modelLoadResult));
            OutputDebugStringA(buffer);
            });
        m_loadModelAsync->ProgressUpdated([this](float progress)
        {
            // Progress callback
            m_modelLoadingProgress = progress;

            // Output progress percentage to VS output
            char buffer[1024];
            sprintf_s(buffer, "Remote Rendering: Model loading progress: %.1f\n", m_modelLoadingProgress * 100.f);
            OutputDebugStringA(buffer);
        });
    }
}



void HolographicAppMain::SetNewState(AppConnectionStatus state, const char* statusMsg)
{
    m_currentStatus = state;
    m_statusMsg = statusMsg ? statusMsg : "";

    // Some log for the VS output panel:
    const char* appStatus = nullptr;

    switch (state)
    {
        case AppConnectionStatus::Disconnected: appStatus = "Disconnected"; break;
        case AppConnectionStatus::CreatingSession: appStatus = "CreatingSession"; break;
        case AppConnectionStatus::StartingSession: appStatus = "StartingSession"; break;
        case AppConnectionStatus::Connecting: appStatus = "Connecting"; break;
        case AppConnectionStatus::Connected: appStatus = "Connected"; break;
        case AppConnectionStatus::ConnectionFailed: appStatus = "ConnectionFailed"; break;
    }

    char buffer[1024];
    sprintf_s(buffer, "Remote Rendering: New status: %s, result: %s\n", appStatus, m_statusMsg.c_str());
    OutputDebugStringA(buffer);
}

void HolographicAppMain::SetNewSession(RR::ApiHandle<RR::AzureSession> newSession)
{
    SetNewState(AppConnectionStatus::StartingSession, nullptr);

    m_timeAtLastRESTCall = m_timer.GetTotalSeconds();
    m_session = newSession;
    m_api = m_session->Actions();
    m_graphicsBinding = m_session->GetGraphicsBinding().as<RR::GraphicsBindingWmrD3d11>();
    m_session->ConnectionStatusChanged([this](auto status, auto error)
        {
            OnConnectionStatusChanged(status, error);
        });

};

void HolographicAppMain::OnConnectionStatusChanged(RR::ConnectionStatus status, RR::Result error)
{
    const char* asString = RR::ResultToString(error);
    m_connectionResult = error;

    switch (status)
    {
    case RR::ConnectionStatus::Connecting:
        SetNewState(AppConnectionStatus::Connecting, asString);
        break;
    case RR::ConnectionStatus::Connected:
        if (error == RR::Result::Success)
        {
            SetNewState(AppConnectionStatus::Connected, asString);
        }
        else
        {
            SetNewState(AppConnectionStatus::ConnectionFailed, asString);
        }
        m_modelLoadTriggered = m_modelLoadFinished = false;
        m_isConnected = error == RR::Result::Success;
        break;
    case RR::ConnectionStatus::Disconnected:
        if (error == RR::Result::Success)
        {
            SetNewState(AppConnectionStatus::Disconnected, asString);
        }
        else
        {
            SetNewState(AppConnectionStatus::ConnectionFailed, asString);
        }
        m_modelLoadTriggered = m_modelLoadFinished = false;
        m_isConnected = false;
        break;
    default:
        break;
    }
    
}
```

### <a name="per-frame-update"></a>Actualización por fotograma

Tenemos que actualizar el cliente una vez por cada paso de simulación y realizar algunas actualizaciones de estado adicionales. La clase `HolographicAppMain::Update` proporciona un buen enlace para las actualizaciones por fotograma.

#### <a name="state-machine-update"></a>Actualización de la máquina de estados

Es necesario sondear el estado de la sesión para ver si ha pasado al estado `Ready`. Si la conexión se ha establecido correctamente, finalmente iniciamos la carga del modelo a través de `StartModelLoading`.

Agregue el siguiente código al cuerpo de la función`HolographicAppMain::Update`:

```cpp
// Updates the application state once per frame.
HolographicFrame HolographicAppMain::Update()
{
    if (m_session != nullptr)
    {
        // Tick the client to receive messages
        m_api->Update();

        if (!m_sessionStarted)
        {
            // Important: To avoid server-side throttling of the requests, we should call GetPropertiesAsync very infrequently:
            const double delayBetweenRESTCalls = 10.0;

            // query session status periodically until we reach 'session started'
            if (m_sessionPropertiesAsync == nullptr && m_timer.GetTotalSeconds() - m_timeAtLastRESTCall > delayBetweenRESTCalls)
            {
                m_timeAtLastRESTCall = m_timer.GetTotalSeconds();
                if (auto propAsync = m_session->GetPropertiesAsync())
                {
                    m_sessionPropertiesAsync = *propAsync;
                    m_sessionPropertiesAsync->Completed([this](const RR::ApiHandle<RR::SessionPropertiesAsync>& async)
                        {
                            if (async->GetStatus() == RR::Result::Success)
                            {
                                auto res = async->GetResult();
                                switch (res.Status)
                                {
                                case RR::RenderingSessionStatus::Ready:
                                {
                                    // The following is async, but we'll get notifications via OnConnectionStatusChanged
                                    m_sessionStarted = true;
                                    SetNewState(AppConnectionStatus::Connecting, nullptr);
                                    RR::ConnectToRuntimeParams init;
                                    init.ignoreCertificateValidation = false;
                                    init.mode = RR::ServiceRenderMode::Default;
                                    m_session->ConnectToRuntime(init);
                                }
                                break;
                                case RR::RenderingSessionStatus::Error:
                                    SetNewState(AppConnectionStatus::ConnectionFailed, "Session error");
                                    break;
                                case RR::RenderingSessionStatus::Stopped:
                                    SetNewState(AppConnectionStatus::ConnectionFailed, "Session stopped");
                                    break;
                                case RR::RenderingSessionStatus::Expired:
                                    SetNewState(AppConnectionStatus::ConnectionFailed, "Session expired");
                                    break;
                                }
    
                            }
                            else
                            {
                                SetNewState(AppConnectionStatus::ConnectionFailed, "Failed to retrieve session status");
                            }
                            m_sessionPropertiesAsync = nullptr; // next try
                            m_needsStatusUpdate = true;
                        });
                }
            }
        }
        if (m_isConnected && !m_modelLoadTriggered)
        {
            m_modelLoadTriggered = true;
            StartModelLoading();
        }
    }

    if (m_needsCoordinateSystemUpdate && m_stationaryReferenceFrame && m_graphicsBinding)
    {
        // Set the coordinate system once. This must be called again whenever the coordinate system changes.
        winrt::com_ptr<ABI::Windows::Perception::Spatial::ISpatialCoordinateSystem> ptr{ m_stationaryReferenceFrame.CoordinateSystem().as<ABI::Windows::Perception::Spatial::ISpatialCoordinateSystem>() };
        m_graphicsBinding->UpdateUserCoordinateSystem(ptr.get());
        m_needsCoordinateSystemUpdate = false;
    }

    // Rest of the body:
    ...
}
```

#### <a name="coordinate-system-update"></a>Actualización del sistema de coordenadas

Es necesario acordar el servicio de representación que se va a usar en un sistema de coordenadas. Para acceder al sistema de coordenadas que queremos usar, necesitamos el elemento `m_stationaryReferenceFrame` que se crea al final de la función `HolographicAppMain::OnHolographicDisplayIsAvailableChanged`.

Normalmente, este sistema de coordenadas no cambia, por lo que se trata de una inicialización puntual. Se debe llamar de nuevo si la aplicación cambia el sistema de coordenadas.

El código anterior establece el sistema de coordenadas una vez dentro de la función `Update` en cuanto ambos tienen un sistema de coordenadas de referencia y una sesión conectada.

#### <a name="camera-update"></a>Actualización de la cámara

Necesitamos actualizar los planos de clip de la cámara para que la cámara del servidor se mantenga sincronizada con la cámara local. Podemos hacerlo al final de la función `Update`:

```cpp
    ...
    if (m_isConnected)
    {
        // Any near/far plane values of your choosing.
        constexpr float fNear = 0.1f;
        constexpr float fFar = 10.0f;
        for (HolographicCameraPose const& cameraPose : prediction.CameraPoses())
        {
            // Set near and far to the holographic camera as normal
            cameraPose.HolographicCamera().SetNearPlaneDistance(fNear);
            cameraPose.HolographicCamera().SetFarPlaneDistance(fFar);
        }

        // The API to inform the server always requires near < far. Depth buffer data will be converted locally to match what is set on the HolographicCamera.
        auto settings = m_api->GetCameraSettings();
        settings->SetNearAndFarPlane(std::min(fNear, fFar), std::max(fNear, fFar));
        settings->SetEnableDepth(true);
    }

    // The holographic frame will be used to get up-to-date view and projection matrices and
    // to present the swap chain.
    return holographicFrame;
}

```

### <a name="rendering"></a>Representación

El último paso es invocar la representación del contenido remoto. Tenemos que realizar esta llamada en la posición exacta dentro de la canalización de representación, después de la función clear de destino de la representación y de establecer la ventanilla. Inserte el siguiente fragmento de código en el bloqueo `UseHolographicCameraResources` dentro de la función `HolographicAppMain::Render`:

```cpp
        ...
        // Existing clear function:
        context->ClearDepthStencilView(depthStencilView, D3D11_CLEAR_DEPTH | D3D11_CLEAR_STENCIL, 1.0f, 0);
        
        // ...

        // Exiting check to test for valid camera:
        bool cameraActive = pCameraResources->AttachViewProjectionBuffer(m_deviceResources);


        // Inject remote rendering: as soon as we are connected, start blitting the remote frame.
        // We do the blitting after the Clear, and before cube rendering.
        if (m_isConnected && cameraActive)
        {
            m_graphicsBinding->BlitRemoteFrame();
        }

        ...
```

## <a name="run-the-sample"></a>Ejecución del ejemplo

El ejemplo debería estar ahora en un estado en el que se compile y se ejecute.

Cuando el ejemplo se ejecute correctamente, verá el cubo girando justo delante y, después de la creación de la sesión y la carga del modelo, representará el modelo de motor que se encuentra en la posición principal actual. La creación de la sesión y la carga del modelo pueden tardar unos minutos. El estado actual solo se escribe en el panel de resultados de Visual Studio. Por lo tanto, se recomienda iniciar el ejemplo desde Visual Studio.

> [!CAUTION]
> El cliente se desconecta del servidor cuando no se llama a la función de tic durante unos segundos. Por lo tanto, desencadenar puntos de interrupción puede hacer que la aplicación se desconecte fácilmente.

Para mostrar el estado correcto con un panel de texto, consulte la versión predefinida de este tutorial en GitHub.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido todos los pasos necesarios para agregar Remote Rendering a una **aplicación holográfica** de C++/DirectX11 de inventario de ejemplo.
Para convertir su propio modelo, consulte el siguiente inicio rápido:

> [!div class="nextstepaction"]
> [Inicio rápido: Conversión de un modelo para su representación](../../../quickstarts/convert-model.md)
