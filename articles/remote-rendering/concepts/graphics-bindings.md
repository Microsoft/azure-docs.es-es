---
title: Enlace de gráficos
description: Configuración de casos de uso y enlaces de gráficos
author: florianborn71
manager: jlyons
services: azure-remote-rendering
titleSuffix: Azure Remote Rendering
ms.author: flborn
ms.date: 12/11/2019
ms.topic: conceptual
ms.service: azure-remote-rendering
ms.custom: devx-track-csharp
ms.openlocfilehash: f769036ac9e5a6945e7ecad30e021d377cabd358
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020276"
---
# <a name="graphics-binding"></a>Enlace de gráficos

Para poder usar Azure Remote Rendering en una aplicación personalizada, debe integrarse en la canalización de representación de la aplicación. Esta integración es responsabilidad del enlace de gráficos.

Una vez configurado, el enlace de gráficos proporciona acceso a varias funciones que afectan a la imagen representada. Estas funciones se pueden dividir en dos categorías: funciones generales que siempre están disponibles y funciones específicas que solo son relevantes para el tipo `Microsoft.Azure.RemoteRendering.GraphicsApiType` seleccionado.

## <a name="graphics-binding-in-unity"></a>Enlace de gráficos en Unity

En Unity, el enlace completo se controla mediante el struct `RemoteUnityClientInit` pasado a `RemoteManagerUnity.InitializeManager`. Para establecer el modo gráfico, el campo `GraphicsApiType` tiene que establecerse en el enlace elegido. El campo se rellenará automáticamente en función de si un objeto XRDevice está presente. El comportamiento se puede invalidar manualmente con los comportamientos siguientes:

* **HoloLens 2**: siempre se usa el enlace de gráficos [Windows Mixed Reality](#windows-mixed-reality).
* **Aplicación de escritorio de UWP plana**: siempre se usa [Simulación](#simulation).
* **Editor de Unity**: siempre se usa [Simulación](#simulation), a menos que se conecte un casco de realidad virtual de WMR, en cuyo caso se deshabilitará ARR para permitir que depure las partes no relacionadas con ARR de la aplicación. Vea también el artículo sobre cómo [usar Holographic Remoting](../how-tos/unity/holographic-remoting.md).

La otra parte relevante para Unity es el acceso al [enlace básico](#access); se pueden saltar todas las otras secciones siguientes.

## <a name="graphics-binding-setup-in-custom-applications"></a>Configuración del enlace de gráficos en aplicaciones personalizadas

Para seleccionar un enlace de gráficos, siga estos dos pasos: En primer lugar, el enlace de gráficos debe inicializarse estáticamente cuando se inicializa el programa:

```cs
RemoteRenderingInitialization managerInit = new RemoteRenderingInitialization;
managerInit.graphicsApi = GraphicsApiType.WmrD3D11;
managerInit.connectionType = ConnectionType.General;
managerInit.right = ///...
RemoteManagerStatic.StartupRemoteRendering(managerInit);
```

```cpp
RemoteRenderingInitialization managerInit;
managerInit.graphicsApi = GraphicsApiType::WmrD3D11;
managerInit.connectionType = ConnectionType::General;
managerInit.right = ///...
StartupRemoteRendering(managerInit); // static function in namespace Microsoft::Azure::RemoteRendering
```

La llamada anterior es necesaria para inicializar Azure Remote Rendering en las API holográficas. Se debe llamar a esta función antes de que se llame a cualquier API holográfica y antes de que se tenga acceso a cualquier otra API de Remote Rendering. Del mismo modo, se debe llamar a la función -init `RemoteManagerStatic.ShutdownRemoteRendering();` correspondiente después de que ya no se llame a ninguna API holográfica.

## <a name="span-idaccessaccessing-graphics-binding"></a><span id="access">Acceso al enlace de gráficos

Una vez configurado un cliente, se puede tener acceso al enlace de gráficos básico con el captador `AzureSession.GraphicsBinding`. Por ejemplo, las últimas estadísticas de fotogramas se pueden recuperar de la siguiente manera:

```cs
AzureSession currentSession = ...;
if (currentSession.GraphicsBinding)
{
    FrameStatistics frameStatistics;
    if (currentSession.GraphicsBinding.GetLastFrameStatistics(out frameStatistics) == Result.Success)
    {
        ...
    }
}
```

```cpp
ApiHandle<AzureSession> currentSession = ...;
if (ApiHandle<GraphicsBinding> binding = currentSession->GetGraphicsBinding())
{
    FrameStatistics frameStatistics;
    if (*binding->GetLastFrameStatistics(&frameStatistics) == Result::Success)
    {
        ...
    }
}
```

## <a name="graphic-apis"></a>API de gráficos

Actualmente hay dos API de gráficos que se pueden seleccionar, `WmrD3D11` y `SimD3D11`. Hay una tercera `Headless`, pero todavía no se admite en el lado cliente.

### <a name="windows-mixed-reality"></a>Windows Mixed Reality

`GraphicsApiType.WmrD3D11` es el enlace predeterminado que se ejecuta en HoloLens 2. Creará el enlace `GraphicsBindingWmrD3d11`. En este modo, Azure Remote Rendering enlaza directamente a las API holográficas.

Para tener acceso a los enlaces de gráficos derivados, es necesario convertir la base `GraphicsBinding`.
Hay dos tareas que deben realizarse para usar el enlace de WMR:

#### <a name="inform-remote-rendering-of-the-used-coordinate-system"></a>Informar a Remote Rendering del sistema de coordenadas usado

```cs
AzureSession currentSession = ...;
IntPtr ptr = ...; // native pointer to ISpatialCoordinateSystem
GraphicsBindingWmrD3d11 wmrBinding = (currentSession.GraphicsBinding as GraphicsBindingWmrD3d11);
if (wmrBinding.UpdateUserCoordinateSystem(ptr) == Result.Success)
{
    ...
}
```

```cpp
ApiHandle<AzureSession> currentSession = ...;
void* ptr = ...; // native pointer to ISpatialCoordinateSystem
ApiHandle<GraphicsBindingWmrD3d11> wmrBinding = currentSession->GetGraphicsBinding().as<GraphicsBindingWmrD3d11>();
if (*wmrBinding->UpdateUserCoordinateSystem(ptr) == Result::Success)
{
    //...
}
```


El puntero anterior `ptr` debe ser un puntero a un objeto `ABI::Windows::Perception::Spatial::ISpatialCoordinateSystem` nativo que define el sistema de coordenadas del espacio universal en el que se expresan las coordenadas de la API.

#### <a name="render-remote-image"></a>Representación de la imagen remota

Al principio de cada fotograma, el marco remoto debe representarse en el búfer de reserva. Esto se hace mediante una llamada a `BlitRemoteFrame`, que rellenará la información de color y de profundidad en el destino de representación enlazado actualmente. Por lo tanto, es importante que esto se realice después de enlazar el búfer de reserva como destino de representación.

```cs
AzureSession currentSession = ...;
GraphicsBindingWmrD3d11 wmrBinding = (currentSession.GraphicsBinding as GraphicsBindingWmrD3d11);
wmrBinding.BlitRemoteFrame();
```

```cpp
ApiHandle<AzureSession> currentSession = ...;
ApiHandle<GraphicsBindingWmrD3d11> wmrBinding = currentSession->GetGraphicsBinding().as<GraphicsBindingWmrD3d11>();
wmrBinding->BlitRemoteFrame();
```

### <a name="simulation"></a>Simulation

`GraphicsApiType.SimD3D11` es el enlace de simulación y, si se selecciona, crea el enlace de gráficos de `GraphicsBindingSimD3d11`. Esta interfaz se utiliza para simular el movimiento principal, por ejemplo, en una aplicación de escritorio, y representa una imagen monoscópica.
La configuración es un poco más complicada y funciona de la siguiente manera:

#### <a name="create-proxy-render-target"></a>Creación del destino de representación proxy

El contenido remoto y local debe representarse en un destino de representación de color o profundidad fuera de pantalla, denominado "proxy", con los datos de la cámara proxy proporcionados por la función `GraphicsBindingSimD3d11.Update`. El proxy debe coincidir con la resolución del búfer de reserva. Una vez que una sesión está lista, es necesario llamar a `GraphicsBindingSimD3d11.InitSimulation` antes de conectarse a ella:

```cs
AzureSession currentSession = ...;
IntPtr d3dDevice = ...; // native pointer to ID3D11Device
IntPtr color = ...; // native pointer to ID3D11Texture2D
IntPtr depth = ...; // native pointer to ID3D11Texture2D
float refreshRate = 60.0f; // Monitor refresh rate up to 60hz.
bool flipBlitRemoteFrameTextureVertically = false;
bool flipReprojectTextureVertically = false;
GraphicsBindingSimD3d11 simBinding = (currentSession.GraphicsBinding as GraphicsBindingSimD3d11);
simBinding.InitSimulation(d3dDevice, depth, color, refreshRate, flipBlitRemoteFrameTextureVertically, flipReprojectTextureVertically);
```

```cpp
ApiHandle<AzureSession> currentSession = ...;
void* d3dDevice = ...; // native pointer to ID3D11Device
void* color = ...; // native pointer to ID3D11Texture2D
void* depth = ...; // native pointer to ID3D11Texture2D
float refreshRate = 60.0f; // Monitor refresh rate up to 60hz.
bool flipBlitRemoteFrameTextureVertically = false;
bool flipReprojectTextureVertically = false;
ApiHandle<GraphicsBindingSimD3d11> simBinding = currentSession->GetGraphicsBinding().as<GraphicsBindingSimD3d11>();
simBinding->InitSimulation(d3dDevice, depth, color, refreshRate, flipBlitRemoteFrameTextureVertically, flipReprojectTextureVertically);
```

La función init debe proporcionarse con punteros al dispositivo d3d nativo, así como a la textura de color y profundidad del destino de representación proxy. Una vez inicializado, se puede llamar a `AzureSession.ConnectToRuntime` y `DisconnectFromRuntime` varias veces, pero cuando se cambia a una sesión diferente, se debe llamar primero a `GraphicsBindingSimD3d11.DeinitSimulation` en la sesión anterior antes de que se pueda llamar a `GraphicsBindingSimD3d11.InitSimulation` en otra sesión.

#### <a name="render-loop-update"></a>Actualización del bucle de representación

El proceso de actualización del bucle de representación consta de varios pasos:

1. En cada fotograma, antes de que tenga lugar la representación, se llama a `GraphicsBindingSimD3d11.Update` con la transformación de la cámara actual que se envía al servidor con el fin de representarse. Al mismo tiempo, la transformación del proxy devuelto debe aplicarse a la cámara proxy para representarse en el destino de representación proxy.
Si el valor de la actualización del proxy devuelto `SimulationUpdate.frameId` es NULL, todavía no hay datos remotos. En ese caso, en lugar de representarse en el destino de representación proxy, cualquier contenido local se debe representar directamente en el búfer de reserva, utilizando los datos de la cámara actuales e ignorando los dos pasos siguientes.
1. Ahora la aplicación debe enlazar el destino de representación proxy y llamar a `GraphicsBindingSimD3d11.BlitRemoteFrameToProxy`. Esta acción rellenará la información de profundidad y color remota en el destino de representación proxy. Ahora se puede representar cualquier contenido local en el proxy mediante la transformación de la cámara proxy.
1. Después, el búfer de reserva debe estar enlazado como un destino de representación y llamarse a `GraphicsBindingSimD3d11.ReprojectProxy` en el punto en el que se puede presentar el búfer de reserva.

```cs
AzureSession currentSession = ...;
GraphicsBindingSimD3d11 simBinding = (currentSession.GraphicsBinding as GraphicsBindingSimD3d11);
SimulationUpdate update = new SimulationUpdate();
// Fill out camera data with current camera data
...
SimulationUpdate proxyUpdate = new SimulationUpdate();
simBinding.Update(update, out proxyUpdate);
// Is the frame data valid?
if (proxyUpdate.frameId != 0)
{
    // Bind proxy render target
    simBinding.BlitRemoteFrameToProxy();
    // Use proxy camera data to render local content
    ...
    // Bind back buffer
    simBinding.ReprojectProxy();
}
else
{
    // Bind back buffer
    // Use current camera data to render local content
    ...
}
```

```cpp
ApiHandle<AzureSession> currentSession;
ApiHandle<GraphicsBindingSimD3d11> simBinding = currentSession->GetGraphicsBinding().as<GraphicsBindingSimD3d11>();

SimulationUpdate update;
// Fill out camera data with current camera data
...
SimulationUpdate proxyUpdate;
simBinding->Update(update, &proxyUpdate);
// Is the frame data valid?
if (proxyUpdate.frameId != 0)
{
    // Bind proxy render target
    simBinding->BlitRemoteFrameToProxy();
    // Use proxy camera data to render local content
    ...
    // Bind back buffer
    simBinding->ReprojectProxy();
}
else
{
    // Bind back buffer
    // Use current camera data to render local content
    ...
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Tutorial: Visualización de modelos representados de forma remota](../tutorials/unity/view-remote-models/view-remote-models.md)
