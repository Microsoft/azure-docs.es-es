---
title: Visualización de un modelo representado de forma remota
description: En el tutorial sobre la aplicación "Hola mundo" de Azure Remote Rendering, se muestra cómo ver un modelo representado de forma remota por Azure.
author: florianborn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: 7509a17127f04220a8e8450a81627354b28bdacd
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89006472"
---
# <a name="tutorial-viewing-a-remotely-rendered-model"></a>Tutorial: Visualización de un modelo representado de forma remota

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Aprovisionar una instancia de Azure Remote Rendering (ARR)
> * Crear y detener una sesión de representación
> * Reutilizar una sesión de representación existente
> * Conectarse a sesiones y desconectarse de ellas
> * Cargar modelos en una sesión de representación

## <a name="prerequisites"></a>Requisitos previos

Para este tutorial, necesitará:

* Una suscripción de pago por uso activa a Azure. Consulte [Creación de una cuenta](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go/).
* Windows SDK 10.0.18362.0 [(descargar)](https://developer.microsoft.com/windows/downloads/windows-10-sdk)
* La versión más reciente de Visual Studio 2019 [(descargar)](https://visualstudio.microsoft.com/vs/older-downloads/)
* GIT [(descargar)](https://git-scm.com/downloads)
* La versión de Unity más reciente (2019.3); en este tutorial, se recomienda el uso de Unity Hub [(descargar)](https://unity3d.com/get-unity/download)
  * Instale estos módulos en Unity:
    * **UWP**: compatibilidad con la compilación de la Plataforma universal de Windows
    * **IL2CPP**: compatibilidad con la compilación de Windows (IL2CPP)
* Conocimientos intermedios de Unity y del lenguaje C# (por ejemplo, creación de scripts y objetos, uso de elementos prefabricados, configuración de eventos de Unity, etc.)

## <a name="provision-an-azure-remote-rendering-arr-instance"></a>Aprovisionar una instancia de Azure Remote Rendering (ARR)

Para obtener acceso al servicio Azure Remote Rendering, primero debe [crear una cuenta](../../../how-tos/create-an-account.md#create-an-account).

## <a name="create-a-new-unity-project"></a>Creación de un proyecto de Unity

> [!TIP]
> El [repositorio de muestras de ARR](https://github.com/Azure/azure-remote-rendering) contiene un proyecto con todos los tutoriales completados, que se puede usar como referencia. Mire en *Unity\Tutorial-Complete* el proyecto de Unity completo.

En Unity Hub, cree un proyecto.
En este ejemplo, supondremos que el proyecto se crea en una carpeta denominada **RemoteRendering**.

:::image type="content" source="./media/unity-new-project.PNG" alt-text="Nuevo proyecto de Unity":::

## <a name="include-the-azure-remote-rendering-package"></a>Inclusión del paquete de Azure Remote Rendering

Debe modificar el archivo `Packages/manifest.json` que se encuentra en la carpeta del proyecto de Unity. Abra el archivo en un editor de texto y agregue las siguientes líneas a la parte superior del manifiesto:

```json
{
    "scopedRegistries": [
    {
        "name": "Azure Mixed Reality Services",
        "url": "https://api.bintray.com/npm/microsoft/AzureMixedReality-NPM/",
        "scopes": ["com.microsoft.azure"]
    }
    ],
    "dependencies": {
        "com.unity.render-pipelines.universal": "7.3.1",
        "com.microsoft.azure.remote-rendering": "0.1.31",
        ...existing dependencies...
    }
}
```

Después de modificar y guardar el manifiesto, Unity se actualiza automáticamente. Confirme que los paquetes se han cargado en la ventana *Project*:

:::image type="content" source="./media/confirm-packages.png" alt-text="Confirmación de las importaciones del paquete":::

Si los paquetes no se están cargando, compruebe la consola de Unity para ver si hay errores. Si no tiene errores y sigue sin ver ningún paquete en la carpeta **Packages**, active el botón de alternancia de visibilidad de paquetes.
![Propiedades de la cámara de Unity](./media/unity-package-visibility.png)

## <a name="ensure-you-have-the-latest-version-of-the-package"></a>Asegúrese de que tiene la versión más reciente del paquete

Los siguientes pasos garantizan que en el proyecto se usa la versión más reciente del paquete de representación remota.

1. En el menú superior del editor de Unity, abra *Window->Package Manager* (Ventana > Administrador de paquetes).
1. Seleccione el paquete **Microsoft Azure Remote Rendering**.
1. En la página del administrador de paquetes del paquete **Azure Remote Rendering**, vea si está disponible el botón **Update** (Actualizar). Si es así, haga clic en él para actualizar el paquete a la última versión disponible:
![El paquete de Azure Remote Rendering en el administrador de paquetes](./media/package-manager.png)
1. La actualización del paquete puede provocar errores ocasionales en la consola. En ese caso, prueba a cerrar y volver a abrir el proyecto.
1. Cuando el paquete esté actualizado, el administrador de paquetes mostrará **Up to date** (Actualizado) en lugar de un botón Upate (Actualizar).
![Paquete actualizado](./media/package-up-to-date.png)
## <a name="configure-the-camera"></a>Configuración de la cámara

1. Seleccione el nodo **Main Camera** (Cámara principal).

1. Abra el menú contextual, para lo cual debe hacer clic con el botón derecho en el componente *Transform* (Transformación) y seleccionar la opción **Reset** (Restablecer):

    ![restablecer transformación de la cámara](./media/camera-reset-transform.png)

1. En **Clear flags** (Borrar marcas), seleccione *Solid Color* (Color sólido).

1. Establezca  **Fondo** en *Negro* (#000000), con un alfa (A) totalmente transparente (0)

    ![Rueda de colores](./media/color-wheel-black.png)

1. Establezca **Clipping Planes** (Planos de recorte) en *Near = 0.3* (Cerca = 0,3) y *Far = 20* (Lejos = 20). Esto significa que la representación recortará la geometría que está a menos de 30 cm o a más de 20 m.

    ![Propiedades de la cámara de Unity](./media/camera-properties.png)

## <a name="adjust-the-project-settings"></a>Ajuste de la configuración del proyecto

1. Abra *Edit > Project Settings...* (Editar > Configuración del proyecto).
1. Seleccione **Quality** (Calidad) en el menú de lista izquierdo.
1. Cambie el valor de **Default Quality Level** (Nivel de calidad predeterminado) de todas las plataformas a *Low* (Bajo). Esta configuración permitirá una representación más eficaz del contenido local y no afectará a la calidad del contenido representado de forma remota.

    ![cambiar la configuración de la calidad del proyecto](./media/settings-quality.png)

1. Seleccione **Graphics** (Gráficos) en el menú de lista izquierdo.
1. Cambie el valor de **Scriptable Rendering Pipeline** (Canalización de representación mediante script) a *HybridRenderingPipeline*.
    ![Cambio de la configuración de gráficos del proyecto](./media/settings-graphics-render-pipeline.png)\
    A veces, la interfaz de usuario no rellena la lista de tipos de canalización disponibles a partir de los paquetes. Si esto ocurre, se debe arrastrar el recurso *HybridRenderingPipeline* al campo manualmente:
    ![Cambio de la configuración de gráficos del proyecto](./media/hybrid-rendering-pipeline.png)

    > [!NOTE]
    > Si no puede arrastrar y colocar el recurso *HybridRenderingPipeline* en el campo Render Pipeline Asset (Recurso de canalización de representación) (posiblemente porque el campo no existe), asegúrese de que la configuración de paquetes contenga el paquete `com.unity.render-pipelines.universal`.

1. Seleccione **Player** (Reproductor) en el menú de lista izquierdo.
1. Seleccione la pestaña **Universal Windows Platform settings** (Configuración de la Plataforma universal de Windows), representada como un icono de Windows.
1. Cambie el valor de **XR Settings** (Configuración de XR) para admitir Windows Mixed Reality, como se muestra a continuación:
    1. Habilite **Virtual Reality Supported** (Se admite realidad virtual)
    1. Presione el botón "+" y agregue **Windows Mixed Reality**.
    1. En **Depth Format** (Formato de profundidad), seleccione *16-Bit Depth* (Profundidad de 16 bits)
    1. Asegúrese de que **Depth Buffer Sharing** (Uso compartido del búfer de profundidad) está habilitado.
    1. En **Stereo Rendering Mode** (Modo de representación estéreo), seleccione *Single Pass Instanced* (Instancia de paso único)

    ![Configuración del reproductor](./media/xr-player-settings.png)

1. En la misma ventana, encima de **XR Settings** (Configuración de XR), expanda **Publishing Settings** (Configuración de publicación)
1. Desplácese hacia abajo hasta **Capabilities** (Funcionalidades) y seleccione:
    * **InternetClient**
    * **InternetClientServer**
    * **SpatialPerception**
    * **PrivateNetworkClientServer** (*opcional*). Seleccione esta opción si quiere conectar el depurador remoto de Unity al dispositivo.

1. En **Supported Device Families** (Familias de dispositivos admitidos), habilite **Holographic** (Holográfico) y **Desktop** (Escritorio).
1. Cierre o acople el panel **Project Settings** (Configuración del proyecto).
1. Abra *File->Build Settings* (Archivo > Configuración de compilación).
1. Seleccione **Universal Windows Platform** (Plataforma universal de Windows).
1. Configure los valores para que coincidan con los que se encuentran a continuación.
1. Presione el botón **Switch Platform** (Cambiar plataforma).
![Configuración de la compilación](./media/build-settings.png)
1. Después de que Unity cambie las plataformas, cierre el panel de compilación.

## <a name="validate-project-setup"></a>Validar la configuración del proyecto

Realice los siguientes pasos para validar que la configuración del proyecto es correcta.

1. Elija la entrada **ValidateProject** del menú **RemoteRendering** en la barra de herramientas del editor de Unity.
1. Revise la ventana **ValidateProject** para ver los errores y corregir la configuración del proyecto cuando sea necesario.

    ![Validación de un proyecto en el Editor de Unity](./media/remote-render-unity-validation.png)

## <a name="create-a-script-to-coordinate-azure-remote-rendering-connection-and-state"></a>Creación de un script para coordinar la conexión y el estado de Azure Remote Rendering

Hay cuatro fases básicas para mostrar los modelos representados de forma remota, que se describen en el diagrama de flujo siguiente. Cada fase debe realizarse en orden. El siguiente paso consiste en crear un script que administre el estado de la aplicación y avance por las fases necesarias.

![Pila de ARR 0](./media/remote-render-stack-0.png)

1. En el panel *Project* (Proyecto), en **Assets** (Recursos), cree una carpeta llamada *RemoteRenderingCore*. Luego, en *RemoteRenderingCore*, cree otra carpeta llamada *Scripts*.

1. Cree un [script de C#](https://docs.unity3d.com/Manual/CreatingAndUsingScripts.html) llamado **RemoteRenderingCoordinator**.
El proyecto se parecerá a esta imagen:

    ![Jerarquía de proyectos](./media/project-structure.png)

    Este script de coordinador realizará un seguimiento del estado de representación remota y lo administrará. Tenga en cuenta que parte de este código se usa para mantener el estado, exponer la funcionalidad a otros componentes, desencadenar eventos y almacenar datos específicos de la aplicación que no están relacionados *directamente* con Azure Remote Rendering. Use el código siguiente como punto de partida; más adelante en el tutorial, abordaremos e implementaremos el código específico de Azure Remote Rendering.

1. Abra **RemoteRenderingCoordinator** en el editor de código y reemplace todo el contenido por el código siguiente:

```csharp
// Copyright (c) Microsoft Corporation. All rights reserved.
// Licensed under the MIT License. See LICENSE in the project root for license information.

using Microsoft.Azure.RemoteRendering;
using Microsoft.Azure.RemoteRendering.Unity;
using System;
using System.Linq;
using System.Threading.Tasks;
using UnityEngine;
using UnityEngine.Events;

#if UNITY_WSA
using UnityEngine.XR.WSA;
#endif

/// <summary>
/// Remote Rendering Coordinator is the controller for all Remote Rendering operations.
/// </summary>

// Require the GameObject with a RemoteRenderingCoordinator to also have the ARRServiceUnity component
[RequireComponent(typeof(ARRServiceUnity))]
public class RemoteRenderingCoordinator : MonoBehaviour
{
    public enum RemoteRenderingState
    {
        NotSet,
        NotInitialized,
        NotAuthorized,
        NoSession,
        ConnectingToExistingRemoteSession,
        ConnectingToNewRemoteSession,
        RemoteSessionReady,
        ConnectingToRuntime,
        RuntimeConnected
    }

    public static RemoteRenderingCoordinator instance;

    // AccountDomain must be '<region>.mixedreality.azure.com' - if no '<region>' is specified, connections will fail
    // For most people '<region>' is either 'westus2' or 'westeurope'
    [SerializeField]
    private string accountDomain = "westus2.mixedreality.azure.com";
    public string AccountDomain
    {
        get => accountDomain.Trim();
        set => accountDomain = value;
    }

    [Header("Development Account Credentials")]
    [SerializeField]
    private string accountId = "<enter your account id here>";
    public string AccountId {
        get => accountId.Trim();
        set => accountId = value;
    }

    [SerializeField]
    private string accountKey = "<enter your account key here>";
    public string AccountKey {
        get => accountKey.Trim();
        set => accountKey = value;
    }

    // These settings are important. All three should be set as low as possible, while maintaining a good user experience
    // See the documentation around session management and the technical differences in session VM size
    [Header("New Session Defaults")]

    public RenderingSessionVmSize renderingSessionVmSize = RenderingSessionVmSize.Standard;
    public uint maxLeaseHours = 0;
    public uint maxLeaseMinutes = 20;

    [Header("Other Configuration")]

    [Tooltip("If you have a known active SessionID, you can fill it in here before connecting")]
    [SerializeField]
    private string sessionIDOverride;
    public string SessionIDOverride {
        get => sessionIDOverride.Trim();
        set => sessionIDOverride = value;
    }

    // When Automatic Mode is true, the coordinator will attempt to automatically proceed through the process of connecting and loading a model
    public bool automaticMode = true;

    public event Action RequestingAuthorization;
    public UnityEvent OnRequestingAuthorization = new UnityEvent();

    public event Action AuthorizedChanged;
    public UnityEvent OnAuthorizationChanged = new UnityEvent();
    private bool authorized;
    public bool Authorized
    {
        get => authorized;
        set
        {
            if (value == true) //This is a one-way value, once we're authorized it lasts until the app is shutdown.
            {
                authorized = value;
                AuthorizedChanged?.Invoke();
            }
        }
    }

    public delegate Task<AzureFrontendAccountInfo> AccountInfoGetter();

    public static AccountInfoGetter ARRCredentialGetter
    {
        private get;
        set;
    }

    private RemoteRenderingState currentCoordinatorState = RemoteRenderingState.NotSet;
    public RemoteRenderingState CurrentCoordinatorState
    {
        get => currentCoordinatorState;
        private set
        {
            if (currentCoordinatorState != value)
            {
                currentCoordinatorState = value;
                Debug.Log($"State changed to: {currentCoordinatorState}");
                CoordinatorStateChange?.Invoke(currentCoordinatorState);
            }
        }
    }

    public static event Action<RemoteRenderingState> CoordinatorStateChange;

    public static AzureSession CurrentSession => instance?.ARRSessionService?.CurrentActiveSession;

    private ARRServiceUnity arrSessionService;

    private ARRServiceUnity ARRSessionService
    {
        get
        {
            if (arrSessionService == null)
                arrSessionService = GetComponent<ARRServiceUnity>();
            return arrSessionService;
        }
    }

    private async Task<AzureFrontendAccountInfo> GetDevelopmentCredentials()
    {
        Debug.LogWarning("Using development credentials! Not recommended for production.");
        return await Task.FromResult(new AzureFrontendAccountInfo(AccountDomain, AccountId, AccountKey));
    }

    /// <summary>
    /// Keep the last used SessionID, when launching, connect to this session if its available
    /// </summary>
    private string LastUsedSessionID
    {
        get
        {
            if (!string.IsNullOrEmpty(SessionIDOverride))
                return SessionIDOverride;

            if (PlayerPrefs.HasKey("LastUsedSessionID"))
                return PlayerPrefs.GetString("LastUsedSessionID");
            else
                return null;
        }
        set
        {
            PlayerPrefs.SetString("LastUsedSessionID", value);
        }
    }

    public void Awake()
    {
        //Forward events to Unity events
        RequestingAuthorization += () => OnRequestingAuthorization?.Invoke();
        AuthorizedChanged += () => OnAuthorizationChanged?.Invoke();

        //Attach to event
        AuthorizedChanged += RemoteRenderingCoordinator_AuthorizedChanged;

        if (instance == null)
            instance = this;
        else
            Destroy(this);

        CoordinatorStateChange += AutomaticMode;

        CurrentCoordinatorState = RemoteRenderingState.NotInitialized;
    }

    private void RemoteRenderingCoordinator_AuthorizedChanged()
    {
        if (CurrentCoordinatorState != RemoteRenderingState.NotAuthorized)
            return; //This isn't valid from any other state than NotAuthorized


        //We just became authorized to connect to Azure
        InitializeSessionService();
    }

    /// <summary>
    /// Automatic mode attempts to automatically progress through the connection and loading steps. Doesn't handle error states.
    /// </summary>
    /// <param name="currentState">The current state</param>
    private async void AutomaticMode(RemoteRenderingState currentState)
    {
        if (!automaticMode)
            return;

        //Add a small delay for visual effect
        await Task.Delay(1500);
        switch (currentState)
        {
            case RemoteRenderingState.NotInitialized:
                InitializeARR();
                break;
            case RemoteRenderingState.NotAuthorized:
                RequestAuthorization();
                break;
            case RemoteRenderingState.NoSession:
                JoinRemoteSession();
                break;
            case RemoteRenderingState.RemoteSessionReady:
                ConnectRuntimeToRemoteSession();
                break;
        }
    }

    /// <summary>
    /// Initializes ARR, associating the main camera
    /// Note: This must be called on the main Unity thread
    /// </summary>
    public void InitializeARR()
    {
        //Implement me
    }

    /// <summary>
    /// Create a new remote session manager
    /// If the ARRCredentialGetter is set, use it as it, otherwise use the development credentials 
    /// </summary>
    public async void InitializeSessionService()
    {
        //Implement me
    }

    /// <summary>
    /// Trigger the event for checking authorization, respond to this event by prompting the user for authentication
    /// If authorized, set Authorized = true
    /// </summary>
    public void RequestAuthorization()
    {
        RequestingAuthorization?.Invoke();
    }

    public void BypassAuthorization()
    {
        Authorized = true;
    }

    /// <summary>
    /// Attempts to join an existing session or start a new session
    /// </summary>
    public async void JoinRemoteSession()
    {
        //Implement me
    }

    public void StopRemoteSession()
    {
        //Implement me
    }

    private async Task<bool> IsSessionAvailable(string sessionID)
    {
        var allSessions = await ARRSessionService.Frontend.GetCurrentRenderingSessionsAsync().AsTask();
        return allSessions.Any(x => x.Id == sessionID && (x.Status == RenderingSessionStatus.Ready || x.Status == RenderingSessionStatus.Starting));
    }

    /// <summary>
    /// Connects the local runtime to the current active session, if there's a session available
    /// </summary>
    public void ConnectRuntimeToRemoteSession()
    {
        //Implement me
    }

    public void DisconnectRuntimeFromRemoteSession()
    {
        //Implement me
    }

    /// <summary>
    /// The session must have its runtime pump updated.
    /// The Actions.Update() will push messages to the server, receive messages, and update the frame-buffer with the remotely rendered content.
    /// </summary>
    private void LateUpdate()
    {
        ARRSessionService?.CurrentActiveSession?.Actions?.Update();
    }

    /// <summary>
    /// Loads a model into the remote session for rendering
    /// </summary>
    /// <param name="modelPath">The model's path</param>
    /// <param name="progress">A call back method that accepts a float progress value [0->1]</param>
    /// <param name="parent">The parent Transform for this remote entity</param>
    /// <returns>An awaitable Remote Rendering Entity</returns>
    public async Task<Entity> LoadModel(string modelPath, Transform parent = null, ProgressHandler progress = null)
    {
        //Implement me
        return null;
    }

    private async void OnRemoteSessionStatusChanged(ARRServiceUnity caller, AzureSession session)
    {
        var properties = await session.GetPropertiesAsync().AsTask();

        switch (properties.Status)
        {
            case RenderingSessionStatus.Error:
            case RenderingSessionStatus.Expired:
            case RenderingSessionStatus.Stopped:
            case RenderingSessionStatus.Unknown:
                CurrentCoordinatorState = RemoteRenderingState.NoSession;
                break;
            case RenderingSessionStatus.Starting:
                CurrentCoordinatorState = RemoteRenderingState.ConnectingToNewRemoteSession;
                break;
            case RenderingSessionStatus.Ready:
                CurrentCoordinatorState = RemoteRenderingState.RemoteSessionReady;
                break;
        }
    }

    private void OnLocalRuntimeStatusChanged(ConnectionStatus status, Result error)
    {
        switch (status)
        {
            case ConnectionStatus.Connected:
                CurrentCoordinatorState = RemoteRenderingState.RuntimeConnected;
                break;
            case ConnectionStatus.Connecting:
                CurrentCoordinatorState = RemoteRenderingState.ConnectingToRuntime;
                break;
            case ConnectionStatus.Disconnected:
                CurrentCoordinatorState = RemoteRenderingState.RemoteSessionReady;
                break;
        }
    }
}
```

## <a name="create-the-azure-remote-rendering-gameobject"></a>Creación del elemento GameObject de Azure Remote Rendering

El coordinador de representación remota y su script necesario (*ARRServiceUnity*) son ambos elementos MonoBehaviour que se deben asociar a un elemento GameObject de la escena. El script *ARRServiceUnity* se proporciona a través de ARR para exponer gran parte de la funcionalidad de ARR para conectarse a las sesiones remotas y administrarlas.

1. Cree un elemento GameObject en la escena (Ctrl+Mayús+N o *GameObject-> Create Empty* [GameObject->Crear vacío]) y llámelo **RemoteRenderingCoordinator**.
1. Agregue el script *RemoteRenderingCoordinator* al elemento GameObject **RemoteRenderingCoordinator**.
![Adición del componente RemoteRenderingCoordinator](./media/add-coordinator-script.png)
1. Confirme que el script *ARRServiceUnity*, que aparece como *Service* (Servicio) en el inspector, se agrega automáticamente a GameObject. Por si se lo pregunta, este es el resultado de tener `[RequireComponent(typeof(ARRServiceUnity))]` en la parte superior del script **RemoteRenderingCoordinator**.
1. Agregue las credenciales de Azure Remote Rendering y su cuenta de dominio al script de coordinador:
![Adición de las credenciales](./media/configure-coordinator-script.png)

## <a name="initialize-azure-remote-rendering"></a>Inicialización de Azure Remote Rendering

Ahora que tenemos el marco de trabajo de nuestro coordinador, implementaremos cada una de las cuatro fases empezando por **Inicialización de Remote Rendering**.

![Pila de ARR 1](./media/remote-render-stack-1.png)

La acción **Initialize** (Inicializar) indica a Azure Remote Rendering el objeto de la cámara que se va a usar para la representación y lleva la máquina de estados a **NotAuthorized**. Esto significa que se ha inicializado el servicio, pero aún no está autorizado para conectarse a una sesión. Dado que el inicio de una sesión de ARR conlleva un costo, es necesario confirmar que el usuario quiera continuar.

Al entrar en el estado **NotAuthorized**, se llama a **CheckAuthorization**, que invoca el evento **RequestingAuthorization** y determina las credenciales de cuenta que se usarán (**AccountInfo** se define cerca de la parte superior de la clase y usa las credenciales definidas mediante el inspector de Unity en el paso anterior).

   > [!NOTE]
   > ARR no admite la recompilación en tiempo de ejecución. Si el script se modifica y se guarda con el modo de reproducción activo, Unity podría bloquearse y para cerrarlo tendrá que usar el administrador de tareas. Asegúrese siempre de que ha detenido el modo de reproducción antes de modificar los scripts.

1. Reemplace el contenido de **InitializeARR** e **InitializeSessionService** por el siguiente código completado:

 ```csharp
/// <summary>
/// Initializes ARR, associating the main camera
/// Note: This must be called on the main Unity thread
/// </summary>
public void InitializeARR()
{
    RemoteManagerUnity.InitializeManager(new RemoteUnityClientInit(Camera.main));

    CurrentCoordinatorState = RemoteRenderingState.NotAuthorized;
}

/// <summary>
/// Create a new remote session manager
/// If the ARRCredentialGetter is set, use it as it, otherwise use the development credentials 
/// </summary>
public async void InitializeSessionService()
{
    if (ARRCredentialGetter == null)
        ARRCredentialGetter = GetDevelopmentCredentials;

    var accountInfo = await ARRCredentialGetter.Invoke();

    ARRSessionService.OnSessionStatusChanged += OnRemoteSessionStatusChanged;

    ARRSessionService.Initialize(accountInfo);

    CurrentCoordinatorState = RemoteRenderingState.NoSession;
}
```

Para pasar de **NotAuthorized** a **NoSession**, normalmente presentaríamos al usuario un cuadro de diálogo modal para que pueda elegir (y eso haremos en otro capítulo). Por ahora, omitiremos automáticamente la comprobación de autorización mediante una llamada a **ByPassAuthentication** en cuanto se desencadene el evento **RequestingAuthorization**.

1. Seleccione el elemento GameObject **RemoteRenderingCoordinator** y busque el evento de Unity **OnRequestingAuthorization** expuesto en el inspector del componente **RemoteRenderingCoordinator**.

1. Para agregar un nuevo evento, presione el botón "+" en la parte inferior derecha.
1. Arrastre el componente a su propio evento para que se haga referencia a sí mismo.
![Omisión de la autenticación](./media/bypass-authorization-add-event.png)\
1. En el menú desplegable, seleccione **RemoteRenderingCoordinator-> BypassAuthorization**.
![Omisión de la autenticación](./media/bypass-authorization-event.png)

## <a name="create-or-join-a-remote-session"></a>Creación de una sesión remota o unión a ella

La segunda fase consiste en crear una sesión de Remote Rendering o unirse a ella (consulte [Sesiones de Remote Rendering](../../../concepts/sessions.md) para más información).

![Pila de ARR 2](./media/remote-render-stack-2.png)

La sesión remota es donde se representarán los modelos. El método **JoinRemoteSession()** intentará la unión a una sesión existente, a la que se realiza un seguimiento con la propiedad **LastUsedSessionID** o si hay un identificador de sesión activo asignado en **sessionIDOverride**. **sessionIDOverride** solo se usa en tareas de depuración y solo si sabe que la sesión existe y desea conectarse explícitamente a ella.

Si no hay ninguna sesión disponible, se crea una nueva. Sin embargo, la creación de una sesión es una operación muy laboriosa. Por lo tanto, debe intentar crear sesiones solo cuando sea necesario y reutilizarlas siempre que sea posible (consulte [Preparación comercial: agrupación de sesiones, programación y prácticas recomendadas](../commercial-ready/commercial-ready.md#fast-startup-time-strategies) para más información sobre la administración de sesiones).

> [!TIP]
> **StopRemoteSession()** finalizará la sesión activa. Para evitar cargos innecesarios, siempre debe detener las sesiones cuando ya no se necesiten.

La máquina de estados pasará ahora a **ConnectingToNewRemoteSession** o **ConnectingToExistingRemoteSession**, en función de las sesiones disponibles. Si se abre una sesión existente o se crea una sesión, se desencadenará el evento **ARRSessionService.OnSessionStatusChanged**, y se ejecutará el método **OnRemoteSessionStatusChanged**. Lo normal es que la máquina de estados avance a **RemoteSessionReady**.

1. Para unirse a una nueva sesión, modifique el código para reemplazar los métodos **JoinRemoteSession()** y **StopRemoteSession()** por los ejemplos completados siguientes:

```csharp
/// <summary>
/// Attempts to join an existing session or start a new session
/// </summary>
public async void JoinRemoteSession()
{
    //If there's a session available that previously belonged to us, and it's ready, use it. Otherwise start a new session.
    RenderingSessionProperties joinResult;
    if (await IsSessionAvailable(LastUsedSessionID))
    {
        CurrentCoordinatorState = RemoteRenderingState.ConnectingToExistingRemoteSession;
        joinResult = await ARRSessionService.OpenSession(LastUsedSessionID);
    }
    else
    {
        CurrentCoordinatorState = RemoteRenderingState.ConnectingToNewRemoteSession;
        joinResult = await ARRSessionService.StartSession(new RenderingSessionCreationParams(renderingSessionVmSize, maxLeaseHours, maxLeaseMinutes));
    }

    if (joinResult.Status == RenderingSessionStatus.Ready || joinResult.Status == RenderingSessionStatus.Starting)
    {
        LastUsedSessionID = joinResult.Id;
    }
    else
    {
        //The session should be ready or starting, if it's not, something went wrong
        await ARRSessionService.StopSession();
        if(LastUsedSessionID == SessionIDOverride)
            SessionIDOverride = "";
        CurrentCoordinatorState = RemoteRenderingState.NoSession;
    }
}

public void StopRemoteSession()
{
    if (ARRSessionService.CurrentActiveSession != null)
    {
        ARRSessionService.CurrentActiveSession.StopAsync();
    }
}
```

Si quiere reutilizar las sesiones para ahorrar tiempo, asegúrese de desactivar la opción **Auto-Stop Session** (Detener sesión automáticamente) en el componente *ARRServiceUnity*. Tenga en cuenta que, en esta situación, las sesiones quedarán en ejecución, aunque no haya nadie conectado a ellas. La sesión se puede ejecutar durante el tiempo que se indique en *MaxLeaseTime* antes de que la cierre el servidor (el valor de *MaxLeaseTime* se puede modificar en el coordinador de Remote Rendering, en *New Session Defaults* [Nuevos valores predeterminados de sesión]). Por otro lado, si apaga automáticamente cada sesión durante la desconexión, tendrá que esperar a que se inicie una nueva sesión cada vez, lo que puede ser un proceso bastante largo.

> [!NOTE]
> La acción de detener una sesión tendrá un efecto inmediato y no se puede deshacer. Una vez que se haya detenido, tiene que crear una sesión con la misma sobrecarga de inicio.

## <a name="connect-the-local-runtime-to-the-remote-session"></a>Conexión del entorno de ejecución local a la sesión remota

A continuación, la aplicación debe conectar su entorno de ejecución local a la sesión remota.

![Pila de ARR 3](./media/remote-render-stack-3.png)

La aplicación también debe escuchar eventos sobre la conexión entre el entorno de ejecución y la sesión actual; esos cambios de estado se administran en **OnLocalRuntimeStatusChanged**. Con este código el estado avanzará a **ConnectingToRuntime**. Una vez conectado en **OnLocalRuntimeStatusChanged**, el estado avanzará a **RuntimeConnected**. La conexión al entorno de ejecución es el último estado que concierne al coordinador, lo que significa que la aplicación se crea con toda la configuración común y está lista para comenzar el trabajo específico de la sesión de carga y representación de los modelos.

 1. Reemplace los métodos **ConnectRuntimeToRemoteSession()** y **DisconnectRuntimeFromRemoteSession()** por las versiones completadas siguientes.
 1. Es importante observar el método de Unity **LateUpdate** y ver que está actualizando la sesión activa actual. De esta forma, la sesión actual puede enviar y recibir mensajes y actualizar el búfer de fotogramas con los fotogramas recibidos de la sesión remota. Ese proceso es fundamental para que ARR funcione correctamente.

```csharp
/// <summary>
/// Connects the local runtime to the current active session, if there's a session available
/// </summary>
public void ConnectRuntimeToRemoteSession()
{
    if (ARRSessionService == null || ARRSessionService.CurrentActiveSession == null)
    {
        Debug.LogError("Not ready to connect runtime");
        return;
    }

    //Connect the local runtime to the currently connected session
    //This session is set when connecting to a new or existing session

    ARRSessionService.CurrentActiveSession.ConnectionStatusChanged += OnLocalRuntimeStatusChanged;
    ARRSessionService.CurrentActiveSession.ConnectToRuntime(new ConnectToRuntimeParams());
    CurrentCoordinatorState = RemoteRenderingState.ConnectingToRuntime;
}

public void DisconnectRuntimeFromRemoteSession()
{
    if (ARRSessionService == null || ARRSessionService.CurrentActiveSession == null || ARRSessionService.CurrentActiveSession.ConnectionStatus != ConnectionStatus.Connected)
    {
        Debug.LogError("Runtime not connected!");
        return;
    }

    ARRSessionService.CurrentActiveSession.DisconnectFromRuntime();
    ARRSessionService.CurrentActiveSession.ConnectionStatusChanged -= OnLocalRuntimeStatusChanged;
    CurrentCoordinatorState = RemoteRenderingState.RemoteSessionReady;
}

/// <summary>
/// The session must have its runtime pump updated.
/// The Actions.Update() will push messages to the server, receive messages, and update the frame-buffer with the remotely rendered content.
/// </summary>
private void LateUpdate()
{
    ARRSessionService?.CurrentActiveSession?.Actions?.Update();
}
```

> [!NOTE]
> La conexión del entorno de ejecución local a una sesión remota depende de que se llame a la opción **Update** (Actualizar) en la sesión activa en ese momento. Si observa que la aplicación no progresa nunca más allá del estado **ConnectingToRuntime**, asegúrese de llamar a **Update** (Actualizar) de forma periódica en la sesión activa.

## <a name="load-a-model"></a>Carga de un modelo

Con la base ya establecida, está listo para cargar un modelo en la sesión remota y comenzar a recibir fotogramas.

![Pila de ARR 4](./media/remote-render-stack-4.png)

El método **LoadModel** está diseñado para aceptar una ruta de acceso de modelo, un controlador de progreso y una transformación principal. Estos argumentos se usarán para cargar un modelo en la sesión remota, actualizar el usuario durante el progreso de la carga y orientar el modelo representado de forma remota en función de la transformación principal.

1. Reemplace todo el método **LoadModel** por el código siguiente:

    ```csharp
    /// <summary>
    /// Loads a model into the remote session for rendering
    /// </summary>
    /// <param name="modelName">The model's path</param>
    /// <param name="parent">The parent Transform for this remote entity</param>
    /// <param name="progress">A call back method that accepts a float progress value [0->1]</param>
    /// <returns>An awaitable Remote Rendering Entity</returns>
    public async Task<Entity> LoadModel(string modelPath, Transform parent = null, ProgressHandler progress = null)
    {
        //Create a root object to parent a loaded model to
        var modelEntity = ARRSessionService.CurrentActiveSession.Actions.CreateEntity();

        //Get the game object representation of this entity
        var modelGameObject = modelEntity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);

        //Ensure the entity will sync its transform with the server
        var sync = modelGameObject.GetComponent<RemoteEntitySyncObject>();
        sync.SyncEveryFrame = true;

        //Parent the new object under the defined parent
        if (parent != null)
        {
            modelGameObject.transform.SetParent(parent, false);
            modelGameObject.name = parent.name + "_Entity";
        }

    #if UNITY_WSA
        //Anchor the model in the world, prefer anchoring parent if there is one
        if (parent != null)
        {
            parent.gameObject.AddComponent<WorldAnchor>();
        }
        else
        {
            modelGameObject.AddComponent<WorldAnchor>();
        }
    #endif

        //Load a model that will be parented to the entity
        var loadModelParams = new LoadModelFromSASParams(modelPath, modelEntity);
        var loadModelAsync = ARRSessionService.CurrentActiveSession.Actions.LoadModelFromSASAsync(loadModelParams);
        if(progress != null)
            loadModelAsync.ProgressUpdated += progress;
        var result = await loadModelAsync.AsTask();
        return modelEntity;
    }
    ```

El código anterior está realizando los pasos siguientes:

1. Crear una [entidad remota](../../../concepts/entities.md)
1. Crear un elemento GameObject local para representar la entidad remota
1. Configurar el elemento GameObject local para sincronizar su estado (es decir, transformarlo) con la entidad remota con cada fotograma
1. Establecer un nombre y agregar un objeto [**WorldAnchor**](https://docs.unity3d.com/ScriptReference/XR.WSA.WorldAnchor.html) para ayudar a la estabilización
1. Cargar los datos del modelo de Blob Storage en la entidad remota
1. Devolver la entidad primaria, para hacer referencia a ella posteriormente

## <a name="view-the-test-model"></a>Visualización del modelo de prueba

Ahora tenemos todo el código necesario para ver un modelo representado de forma remota, y se han completado las cuatro fases necesarias para la representación remota. Lo siguiente es agregar algo de código para iniciar el proceso de carga del modelo.

![Pila de ARR 4](./media/remote-render-stack-5.png)

1. Agregue el código siguiente a la clase **RemoteRenderingCoordinator** justo debajo del método **LoadModel**:

    ```csharp
    private bool loadingTestModel = false;
    [ContextMenu("Load Test Model")]
    public async void LoadTestModel()
    {
        if(CurrentCoordinatorState != RemoteRenderingState.RuntimeConnected)
        {
            Debug.LogError("Please wait for the runtime to connect before loading the test model. Try again later.");
            return;
        }
        if(loadingTestModel)
        {
            Debug.Log("Test model already loading or loaded!");
            return;
        }
        loadingTestModel = true;
    
        // Create a parent object to use for positioning
        GameObject testParent = new GameObject("TestModelParent");
        testParent.transform.position = new Vector3(0f, 0f, 3f);
    
        // The 'built in engine path' is a special path that references a test model built into Azure Remote Rendering.
        await LoadModel("builtin://Engine", testParent.transform, (progressValue) => Debug.Log($"Loading Test Model progress: {Math.Round(progressValue * 100, 2)}%"));
    }
    ```
    
    Este código crea un elemento GameObject para que funcione como elemento primario del modelo de prueba. Luego, llama al método **LoadModel** para cargar el modelo "builtin://Engine", que es un recurso integrado en Azure Remote Rendering para probar la representación.

1. Guarde el código.
1. Presione el botón de reproducción en el editor de Unity para iniciar el proceso de conectarse a Azure Remote Rendering y crear una sesión.
1. Aunque no verá mucho en la vista de juego, en la consola se muestra cómo la aplicación va cambiando de estado. Probablemente llegará a `ConnectingToNewRemoteSession` y es posible que permanezca hasta cinco minutos.
1. Seleccione el elemento GameObject **RemoteRenderingCoordinator** para ver sus scripts asociados en el inspector. Examine la actualización del componente **Service** (Servicio) a medida que avanza por los pasos de inicialización y conexión.
1. Supervise la salida de la consola, en espera de que el estado cambie a **RuntimeConnected**.
1. Una vez conectado el entorno de ejecución, haga clic con el botón derecho en **RemoteRenderingCoordinator** en el inspector para exponer el menú contextual. Luego, haga clic en la opción **Load Test Model** (Cargar modelo de prueba) en el menú contextual, agregado por la parte `[ContextMenu("Load Test Model")]` del código anterior.

    ![Carga desde el menú contextual](./media/load-test-model.png)

1. Examine en la consola la salida de **ProgressHandler** que hemos pasado al método **LoadModel**.
1. Ya puede ver el modelo representado de forma remota.

> [!NOTE]
> El modelo remoto nunca será visible en la vista de escena, solo en la vista de juego. El motivo es que ARR está representando los fotogramas de forma remota en concreto para la perspectiva de la cámara de vista de juego y no tiene en cuenta la cámara del editor (que se usa para representar la vista de escena).

## <a name="next-steps"></a>Pasos siguientes

![Modelo cargado](./media/test-model-rendered.png)

Felicidades. Ha creado una aplicación básica capaz de ver los modelos representados de forma remota mediante Azure Remote Rendering. En el siguiente tutorial, integraremos MRTK e importaremos nuestros propios modelos.

> [!div class="nextstepaction"]
> [Siguiente: Interfaces y modelos personalizados](../custom-models/custom-models.md)
