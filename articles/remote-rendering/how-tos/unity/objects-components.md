---
title: Componentes y objetos de juegos de Unity
description: Describe los métodos específicos de Unity para trabajar con componentes y entidades de Remote Rendering.
author: jakrams
ms.author: jakras
ms.date: 02/28/2020
ms.topic: how-to
ms.custom: devx-track-csharp
ms.openlocfilehash: 662c28196b06f5fbe49f69cb7145fdd33805e000
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019052"
---
# <a name="interact-with-unity-game-objects-and-components"></a>Interacción con componentes y objetos de juegos de Unity

Azure Remote Rendering (ARR) está optimizado para grandes cantidades de objetos (consulte [Límites](../../reference/limits.md)). Aunque es posible administrar jerarquías grandes y complejas en el host, la replicación de todas ellas en Unity en dispositivos de bajo consumo es inviable.

Por lo tanto, cuando se carga un modelo en el host, Azure Remote Rendering refleja la información sobre la estructura del modelo en el dispositivo cliente (que incurrirá en tráfico de red), pero no replica los objetos y componentes en Unity. En su lugar, espera que solicite manualmente los objetos y componentes de juego de Unity necesarios, de modo que pueda limitar la sobrecarga a la que realmente se necesita. De este modo, tendrá más control sobre el rendimiento del cliente.

Por lo tanto, la integración de Unity de Azure Remote Rendering incluye una funcionalidad adicional para replicar la estructura de Remote Rendering a petición.

## <a name="load-a-model-in-unity"></a>Carga de un modelo en Unity

Cuando se carga un modelo, se obtiene una referencia al objeto raíz del modelo cargado. Esta referencia no es un objeto de juego Unity, pero puede convertirla en uno con el método de extensión `Entity.GetOrCreateGameObject()`. Esa función espera un argumento de tipo `UnityCreationMode`. Si pasa `CreateUnityComponents`, el objeto de juego Unity recién creado también se rellenará con los componentes de proxy para todos los componentes de Remote Rendering que existan en el host. Sin embargo, se recomienda usar `DoNotCreateUnityComponents` para mantener la sobrecarga mínima.

### <a name="load-model-with-task"></a>Carga de un modelo con tarea

```cs
LoadModelAsync _pendingLoadTask = null;
void LoadModelWithTask()
{
    _pendingLoadTask = RemoteManagerUnity.CurrentSession.Actions.LoadModelFromSASAsync(new LoadModelFromSASParams("builtin://Engine"));

    _pendingLoadTask.Completed += (LoadModelAsync res) =>
    {
        // turn the root object into a Unity game object
        var gameObject = res.Result.Root?.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
        _pendingLoadTask = null;
    };

    // also listen to progress updates:
    _pendingLoadTask.ProgressUpdated += (float progress) =>
    {
        // progress is a fraction in [0..1] range
        int percentage = (int)(progress * 100.0f);
        // do something...
        // Since the updates are triggered by the main thread, we may access unity objects here.
    };
}
```

### <a name="load-model-with-unity-coroutines"></a>Carga de un modelo con las corrutinas de Unity

```cs
IEnumerator LoadModelWithCoroutine()
{
    LoadModelAsync task = RemoteManagerUnity.CurrentSession.Actions.LoadModelFromSASAsync(new LoadModelFromSASParams("builtin://Engine"));

    while (!task.IsCompleted)
    {
        int percentage = (int)(task.Progress * 100.0f);
        yield return null;
    }

    if (!task.IsFaulted)
    {
        var gameObject = task.Result.Root?.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
    }

    task = null;
}
```

### <a name="load-model-with-await-pattern"></a>Carga de un modelo con el patrón Await

```cs
async void LoadModelWithAwait()
{
    var result = await RemoteManagerUnity.CurrentSession.Actions.LoadModelFromSASAsync(new LoadModelFromSASParams("builtin://Engine")).AsTask();
    var gameObject = result.Root?.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
}
```

Los ejemplos de código anteriores usaban la ruta de acceso de carga del modelo a través de SAS porque el modelo integrado está cargado. El direccionamiento del modelo a través de contenedores de blobs (mediante `LoadModelAsync` y `LoadModelParams`) funciona de forma similar.

## <a name="remoteentitysyncobject"></a>RemoteEntitySyncObject

La creación de un objeto de juego Unity agrega implícitamente un componente `RemoteEntitySyncObject` al objeto de juego. Este componente se utiliza para sincronizar la transformación de entidad en el servidor. De forma predeterminada `RemoteEntitySyncObject` requiere que el usuario llame explícitamente a `SyncToRemote()` para sincronizar el estado de Unity local con el servidor. Al habilitar `SyncEveryFrame`, el objeto se sincronizará automáticamente.

Los objetos con un `RemoteEntitySyncObject` pueden tener instancias de sus elementos secundarios remotos y mostrarse en el editor de Unity mediante el botón **:::no-loc text="Show children":::** .

![RemoteEntitySyncObject](media/remote-entity-sync-object.png)

## <a name="wrapper-components"></a>Componentes de contenedor

[Los componentes](../../concepts/components.md) adjuntados a las entidades de Remote Rendering se exponen a Unity mediante el proxy `MonoBehavior`. Estos servidores proxy representan el componente remoto en Unity y reenvían todas las modificaciones al host.

Para crear componentes de Remote Rendering de proxy, use el método de extensión `GetOrCreateArrComponent`:

```cs
var cutplane = gameObject.GetOrCreateArrComponent<ARRCutPlaneComponent>(RemoteManagerUnity.CurrentSession);
```

## <a name="coupled-lifetimes"></a>Duraciones acopladas

La duración de una [entidad](../../concepts/entities.md) remota y un objeto de juego de Unity se acopla mientras se enlazan con un objeto `RemoteEntitySyncObject`. Si llama a `UnityEngine.Object.Destroy(...)` con este tipo de objeto de juego, también se quitará la entidad remota.

Para destruir el objeto de juego de Unity sin afectar a la entidad remota, primero debe llamar a `Unbind()` en el objeto `RemoteEntitySyncObject`.

Lo mismo se aplica a todos los componentes de proxy. Para destruir solo la representación del cliente, primero debe llamar a `Unbind()` en el componente del proxy:

```cs
var cutplane = gameObject.GetComponent<ARRCutPlaneComponent>();
if (cutplane != null)
{
    cutplane.Unbind();
    UnityEngine.Object.Destroy(cutplane);
}
```

## <a name="next-steps"></a>Pasos siguientes

* [Configuración de Remote Rendering para Unity](unity-setup.md)
* [Tutorial: Trabajo con entidades remotas en Unity](../../tutorials/unity/manipulate-models/manipulate-models.md)
