---
title: Configuración de Remote Rendering para Unity
description: Inicialización de Azure Remote Rendering en un proyecto de Unity
author: jakrams
ms.author: jakras
ms.date: 02/27/2020
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: 4a0be44d8709726e159e17e703566c6c576bc18f
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018984"
---
# <a name="set-up-remote-rendering-for-unity"></a>Configuración de Remote Rendering para Unity

Para habilitar Azure Remote Rendering (ARR) en Unity, proporcionamos métodos dedicados que se encargan de algunos aspectos específicos de Unity.

## <a name="startup-and-shutdown"></a>Inicio y apagado

Para inicializar Remote Rendering, use `RemoteManagerUnity`. Esta clase llama al elemento `RemoteManager` genérico, pero implementa detalles específicos de Unity automáticamente. Por ejemplo, Unity usa un sistema de coordenadas específico. Al llamar a `RemoteManagerUnity.Initialize`, se configurará la convención adecuada. La llamada también requiere que proporcione la cámara de Unity que se debe usar para mostrar el contenido representado de forma remota.

```cs
// initialize Azure Remote Rendering for use in Unity:
// it needs to know which camera is used for rendering the scene
RemoteUnityClientInit clientInit = new RemoteUnityClientInit(Camera.main);
RemoteManagerUnity.InitializeManager(clientInit);
```

Para apagar Remote Rendering, llame a `RemoteManagerStatic.ShutdownRemoteRendering()`.

Una vez que se ha creado un elemento `AzureSession` y se ha elegido como sesión de representación principal, se debe registrar con `RemoteManagerUnity`:

```cs
RemoteManagerUnity.CurrentSession = ...
```

### <a name="full-example-code"></a>Ejemplo de código completo

En el código siguiente se muestran todos los pasos necesarios para inicializar Azure Remote Rendering en Unity:

```cs
// initialize Remote Rendering
RemoteUnityClientInit clientInit = new RemoteUnityClientInit(Camera.main);
RemoteManagerUnity.InitializeManager(clientInit);

// create a frontend
AzureFrontendAccountInfo accountInfo = new AzureFrontendAccountInfo();
// ... fill out accountInfo ...
AzureFrontend frontend = new AzureFrontend(accountInfo);

// start a session
AzureSession session = await frontend.CreateNewRenderingSessionAsync(new RenderingSessionCreationParams(RenderingSessionVmSize.Standard, 0, 30)).AsTask();

// let RemoteManagerUnity know about the session we want to use
RemoteManagerUnity.CurrentSession = session;

session.ConnectToRuntime(new ConnectToRuntimeParams());

/// When connected, load and modify content

RemoteManagerStatic.ShutdownRemoteRendering();
```

## <a name="convenience-functions"></a>Funciones de conveniencia

### <a name="session-state-events"></a>Eventos de estado de sesión

`RemoteManagerUnity.OnSessionUpdate` emite eventos para cuando su estado de sesión cambie. Consulte la documentación del código para obtener más información.

### <a name="arrserviceunity"></a>ARRServiceUnity

`ARRServiceUnity` es un componente opcional para simplificar la configuración y la administración de sesiones. Contiene opciones para detener automáticamente la sesión cuando la aplicación se está cerrando o se sale del modo de reproducción en el editor, así como para renovar automáticamente la concesión de la sesión cuando sea necesario. Almacena en la memoria caché datos como las propiedades de sesión (vea su variable `LastProperties`) y expone eventos para los cambios de estado de sesión y los errores de sesión.

No puede haber más de una instancia de `ARRServiceUnity` a la vez. Está pensada para que pueda empezar a trabajar más rápidamente implementando cierta funcionalidad común. Sin embargo, para una aplicación de mayor tamaño, puede ser preferible hacer esas cosas por sí mismo.

Para obtener un ejemplo de configuración y uso de `ARRServiceUnity`, consulte [Tutorial: Visualización de modelos representados de forma remota](../../tutorials/unity/view-remote-models/view-remote-models.md).

## <a name="next-steps"></a>Pasos siguientes

* [Instalación del paquete de Remote Rendering para Unity](install-remote-rendering-unity-package.md)
* [Tutorial: Visualización de modelos representados de forma remota](../../tutorials/unity/view-remote-models/view-remote-models.md)
