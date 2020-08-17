---
title: Interfaces y modelos personalizados
description: Adición de controladores de vista e ingesta de modelos personalizados para representarlos mediante Azure Remote Rendering
author: florianborn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.openlocfilehash: ae3d0ac6fb332fa17fbe938572b94c51e0785089
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449013"
---
# <a name="tutorial-interfaces-and-custom-models"></a>Tutorial: Interfaces y modelos personalizados

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Agregar la utilidad Mixed Reality Toolkit al proyecto
> * Administrar el estado del modelo
> * Configurar Azure Blob Storage para la ingesta de modelos
> * Cargar y procesar modelos para representarlos

## <a name="prerequisites"></a>Requisitos previos

* Este tutorial se basa en el [Tutorial: Visualización de un modelo representado de forma remota](../view-remote-models/view-remote-models.md).

## <a name="get-started-with-the-mixed-reality-toolkit-mrtk"></a>Introducción a Mixed Reality Toolkit (MRTK)

Mixed Reality Toolkit (MRTK) es un kit de herramientas multiplataforma para la creación de experiencias de realidad mixta. Usaremos MRTK 2.3 por sus características de interacción y visualización.

Para agregar MRTK, siga los [pasos necesarios](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.3.0/Documentation/GettingStartedWithTheMRTK.html#required) enumerados en [Introducción a MRTK](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.3.0/Documentation/GettingStartedWithTheMRTK.html).

Estos pasos son:
 - [Obtención de los paquetes de Unity para MRTK más recientes](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.3.0/Documentation/GettingStartedWithTheMRTK.html#get-the-latest-mrtk-unity-packages)
     - Aunque dice "más reciente", es la versión 2.3.
     - En este tutorial, solo se empleará el paquete *Foundation*. No se necesitan los paquetes *Extensions*, *Tools* ni *Examples*.
 - [Importación de los paquetes de MRTK en el proyecto de Unity](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.3.0/Documentation/GettingStartedWithTheMRTK.html#import-mrtk-packages-into-your-unity-project)
 - [Cambio del proyecto de Unity a la plataforma de destino](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.3.0/Documentation/GettingStartedWithTheMRTK.html#switch-your-unity-project-to-the-target-platform)
     - Aunque ya debería haber realizado este paso en el primer capítulo, es un buen momento para comprobarlo.
 - [Adición de MRTK a una escena o un proyecto nuevos](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.3.0/Documentation/GettingStartedWithTheMRTK.html#add-mrtk-to-a-new-scene-or-new-project)
     - Puede agregar MRTK a una nueva escena y volver a agregar los objetos o scripts de coordinador y modelo, o puede agregar MRTK a la escena existente con el comando de menú *Mixed Reality Toolkit-> Add to Scene and Configure* (Mixed Reality Toolkit > Agregar a escena y configurar).

## <a name="import-assets-used-by-this-tutorial"></a>Importación de los recursos usados en este tutorial

A partir de este capítulo, se implementará un [patrón modelo-vista-controlador](https://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller) sencillo en gran parte del material incluido. La parte *modelo* del patrón es el código específico de Azure Remote Rendering y la administración de estados relacionada con este servicio. Las partes *vista* y *controlador* del patrón se implementan mediante recursos de MRTK y algunos scripts personalizados. En este tutorial se puede usar el *modelo* sin la parte *vista-controlador* aquí implementada. Esta separación le permite integrar fácilmente el código de este tutorial en su propia aplicación, donde se ocupará de la parte *vista-controlador* del modelo de diseño.

Con la introducción de MRTK, hay varios scripts, elementos prefabricados y recursos que ahora se pueden agregar al proyecto para admitir interacciones e información visual. Estos recursos, conocidos como **recursos del tutorial**, se agrupan en un [paquete de recursos de Unity](https://docs.unity3d.com/Manual/AssetPackages.html), que se incluye en el [repositorio de GitHub Azure Remote Rendering](https://github.com/Azure/azure-remote-rendering) en "\Unity\TutorialAssets\TutorialAssets.unitypackage".

1. Clone o descargue el repositorio de Git [Azure Remote Rendering](https://github.com/Azure/azure-remote-rendering), si la descarga extrae el archivo ZIP en una ubicación conocida.
1. En el proyecto de Unity, elija *Assets -> Import Package -> Custom Package* (Recursos > Importar paquete > Paquete personalizado).
1. En el explorador de archivos, vaya al directorio en el que ha clonado o descomprimido el repositorio Azure Remote Rendering y, después, seleccione el archivo .unitypackage que se encuentra en **Unity -> TutorialAssets -> TutorialAssets.unitypackage**
1. Seleccione el botón **Import** (Importar) para importar el contenido del paquete en el proyecto.
1. En el editor de Unity, seleccione *Mixed Reality Toolkit -> Utilities -> Upgrade MRTK Standard Shader for Lightweight Render Pipeline* (Mixed Reality Toolkit > Utilidades > Actualizar el sombreador estándar de MRTK para la canalización de presentación ligera) en la barra de menús superior y siga las indicaciones para actualizar el sombreador.

Una vez que MRTK y los recursos del tutorial estén incluidos en el proyecto, cambiaremos el perfil de MRTK por uno más adecuado para el tutorial.

1. Seleccione el elemento GameObject **MixedRealityToolkit** en la jerarquía de la escena.
1. En el inspector, en el componente **MixedRealityToolkit**, cambie el perfil de configuración a *ARRMixedRealityToolkitConfigurationProfile*.
1. Presione *Ctrl+S* para guardar los cambios.

Se configurará MRTK, principalmente con los perfiles de HoloLens 2 predeterminados. Los perfiles proporcionados están preconfigurados de las siguientes maneras:
 - Desactive Profiler (presione 9 para activarlo o desactivarlo o diga "Mostrar u ocultar Profiler" en el dispositivo).
 - Desactive el cursor de mirada.
 - Habilite los clics del mouse de Unity para poder hacer clic en los elementos de la interfaz de usuario de MRTK con el mouse en lugar de con la mano simulada.

## <a name="add-the-app-menu"></a>Adición del menú de la aplicación

La mayoría de los controladores de vista de este tutorial funcionan con clases base abstractas en lugar de con clases concretas. Este patrón ofrece mayor flexibilidad y nos permite proporcionar los controladores de vista automáticamente, al tiempo que le ayuda a conocer el código de Azure Remote Rendering. Por motivos de simplicidad, la clase **RemoteRenderingCoordinator** no tiene una clase abstracta y su controlador de vista funciona directamente con la clase concreta.

Ahora puede agregar el elemento prefabricado **AppMenu** a la escena para obtener información visual del estado de sesión actual. Este controlador de vista revelará un submenú con más controladores de vista a medida que implementemos e integremos más características de ARR en la escena. Por ahora, **AppMenu** tendrá una indicación visual del estado de ARR y presentará el panel modal que el usuario emplea para autorizar a la aplicación a conectarse a ARR.

1. Busque el elemento prefabricado **AppMenu** en *Assets/RemoteRenderingTutorial/Prefabs/AppMenu*.
1. Arrastre el elemento prefabricado **AppMenu** a la escena.
1. Es probable que vea un cuadro de diálogo **TMP Importer** (Importador de TMP), ya que es la primera vez que se incluyen recursos de *Text Mesh Pro* en la escena. Siga las indicaciones para **importar la información esencial de TMP**. A continuación, cierre el cuadro de diálogo del importador; los ejemplos y los elementos adicionales no son necesarios.
1. **AppMenu** está configurado para enlazarse automáticamente y proporcionar el modal de consentimiento para conectarse a una sesión, de modo que podemos quitar la omisión colocada anteriormente. En el objeto GameObject **RemoteRenderingCoordinator**, quite la omisión de autorización que hemos implementado anteriormente; para ello, presione el botón "-" en el evento **On Requesting Authorization** (Al solicitar autorización).
 ![Quitar omisión](./media/remove-bypass-event.png).
1. Para probar el controlador de vista, presione **Play** (Reproducir) en el editor de Unity.
1. En el editor, ahora que está configurado MRTK, puede usar las teclas WASD para cambiar la posición de la vista, o mantener presionado el botón derecho del mouse y mover el mouse para cambiar su dirección. Muévase un poco por la escena para hacerse una idea de los controles.
1. En el dispositivo, puede subir la palma para llamar a **AppMenu**; en el editor de Unity, use la tecla de acceso directo "M".
1. Si ha perdido de vista el menú, presione la tecla "M" para llamar al menú. El menú se colocará cerca de la cámara para facilitar la interacción.
1. La autorización se mostrará ahora en forma de solicitud a la derecha de **AppMenu** y, de aquí en adelante, se usará para autorizar a la aplicación la administración de las sesiones de representación remota.
 ![Autorización de la interfaz de usuario](./media/authorize-request-ui.png)
1. Detenga la reproducción de Unity para continuar con el tutorial.

## <a name="manage-model-state"></a>Administrar el estado del modelo

Ahora implementaremos un nuevo script, **RemoteRenderedModel** que se usa para el seguimiento del estado, la respuesta a eventos y la activación de estos y la configuración. Básicamente, **RemoteRenderedModel** almacena la ruta de acceso remota a los datos del modelo en `modelPath`. Para comprobar si debe cargar o descargar automáticamente el modelo que define, escucha los cambios de estado en **RemoteRenderingCoordinator**. El elemento GameObject que tiene asociado **RemoteRenderedModel** será el elemento primario local del contenido remoto.

Tenga en cuenta que el script **RemoteRenderedModel** implementa **BaseRemoteRenderedModel**, que procede de los **recursos del tutorial**. De esta forma, el controlador de vista del modelo remoto se enlazará con el script.

1. Cree un script llamado **RemoteRenderedModel** en la misma carpeta que **RemoteRenderingCoordinator**. Reemplace todo el contenido por el código siguiente:

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;
    using UnityEngine.Events;

    public class RemoteRenderedModel : BaseRemoteRenderedModel
    {
        [SerializeField]
        [Tooltip("The friendly name for this model")]
        private string modelDisplayName;
        [SerializeField]
        [Tooltip("The URI for this model")]
        private string modelPath;

        public bool AutomaticallyLoad = true;

        private ModelState currentModelState = ModelState.NotReady;

        public override string ModelDisplayName { get => modelDisplayName; set => modelDisplayName = value; }
        public override string ModelPath { get => modelPath; set => modelPath = value; }

        public override ModelState CurrentModelState
        {
            get => currentModelState;
            protected set
            {
                if (currentModelState != value)
                {
                    currentModelState = value;
                    ModelStateChange?.Invoke(value);
                }
            }
        }

        public override event Action<ModelState> ModelStateChange;
        public override event Action<float> LoadProgress;
        public override Entity ModelEntity { get; protected set; }

        public UnityEvent OnModelNotReady = new UnityEvent();
        public UnityEvent OnModelReady = new UnityEvent();
        public UnityEvent OnStartLoading = new UnityEvent();
        public UnityEvent OnModelLoaded = new UnityEvent();
        public UnityEvent OnModelUnloading = new UnityEvent();

        public UnityFloatEvent OnLoadProgress = new UnityFloatEvent();

        public void Awake()
        {
            // Hook up the event to the Unity event
            LoadProgress += (progress) => OnLoadProgress?.Invoke(progress);

            ModelStateChange += HandleUnityStateEvents;
        }

        private void HandleUnityStateEvents(ModelState modelState)
        {
            switch (modelState)
            {
                case ModelState.NotReady:  OnModelNotReady?.Invoke();  break;
                case ModelState.Ready:     OnModelReady?.Invoke();     break;
                case ModelState.Loading:   OnStartLoading?.Invoke();   break;
                case ModelState.Loaded:    OnModelLoaded?.Invoke();    break;
                case ModelState.Unloading: OnModelUnloading?.Invoke(); break;
            }
        }

        private void Start()
        {
            //Attach to and initialize current state (in case we're attaching late)
            RemoteRenderingCoordinator.CoordinatorStateChange += Instance_CoordinatorStateChange;
            Instance_CoordinatorStateChange(RemoteRenderingCoordinator.instance.CurrentCoordinatorState);
        }

        /// <summary>
        /// Listen for state changes on the coordinator, clean up this model's remote objects if we're no longer connected.
        /// Automatically load if required
        /// </summary>
        private void Instance_CoordinatorStateChange(RemoteRenderingCoordinator.RemoteRenderingState state)
        {
            switch (state)
            {
                case RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected:
                    CurrentModelState = ModelState.Ready;
                    if (AutomaticallyLoad)
                        LoadModel();
                    break;
                default:
                    UnloadModel();
                    break;
            }
        }

        private void OnDestroy()
        {
            RemoteRenderingCoordinator.CoordinatorStateChange -= Instance_CoordinatorStateChange;
            UnloadModel();
        }

        /// <summary>
        /// Asks the coordinator to create a model entity and listens for coordinator state changes
        /// </summary>
        [ContextMenu("Load Model")]
        public override async void LoadModel()
        {
            if (CurrentModelState != ModelState.Ready)
                return; //We're already loaded, currently loading, or not ready to load

            CurrentModelState = ModelState.Loading;

            ModelEntity = await RemoteRenderingCoordinator.instance?.LoadModel(ModelPath, this.transform, SetLoadingProgress);

            if (ModelEntity != null)
                CurrentModelState = ModelState.Loaded;
            else
                CurrentModelState = ModelState.Error;
        }

        /// <summary>
        /// Clean up the local model instances
        /// </summary>
        [ContextMenu("Unload Model")]
        public override void UnloadModel()
        {
            CurrentModelState = ModelState.Unloading;

            if (ModelEntity != null)
            {
                var modelGameObject = ModelEntity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
                Destroy(modelGameObject);
                ModelEntity.Destroy();
                ModelEntity = null;
            }

            if (RemoteRenderingCoordinator.instance.CurrentCoordinatorState == RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected)
                CurrentModelState = ModelState.Ready;
            else
                CurrentModelState = ModelState.NotReady;
        }

        /// <summary>
        /// Update the Unity progress event
        /// </summary>
        /// <param name="progressValue"></param>
        public override void SetLoadingProgress(float progressValue)
        {
            LoadProgress?.Invoke(progressValue);
        }
    }
    ```

Básicamente, **RemoteRenderedModel** contiene los datos necesarios para cargar un modelo (en este caso, el SAS o el URI *builtin://* ) y realiza un seguimiento del estado del modelo remoto. Cuando es el momento de la carga, se llama al método `LoadModel` en **RemoteRenderingCoordinator** y la entidad que contiene el modelo se devuelve como referencia y se descarga.

## <a name="load-the-test-model"></a>Carga del modelo de prueba

Vamos a volver a cargar el modelo de prueba para probar el nuevo script. Crearemos un elemento GameObject que contenga el script y que será un elemento primario del modelo de prueba.

1. Cree un elemento GameObject vacío en la escena y llámelo **TestModel**.
1. Agregue el script *RemoteRenderedModel* a **TestModel**.
![Adición del componente RemoteRenderedModel](./media/add-remote-rendered-model-script.png)
1. Rellene `Model Display Name` y `Model Path` con "*TestModel*" y "*builtin://Engine*", respectivamente.
![Especificación de los detalles del modelo](./media/add-model-script.png)
1. Coloque el objeto **TestModel** delante de la cámara, en la posición **x = 0, y = 0, z = 3**.
![Colocación del objeto](./media/test-model-position.png)
1. Asegúrese de que **AutomaticallyLoad** está activado.
1. Presione **Play** (Reproducir) en el editor de Unity para probar la aplicación.
1. Para conceder autorización, haga clic en el botón *Connect* (Conectar) para permitir que la aplicación cree una sesión, se conecte a ella y cargue automáticamente el modelo.

Vea en la consola como la aplicación avanza por sus estados. Tenga en cuenta que algunos estados pueden tardar algún tiempo en completarse y no mostrarán el progreso. Finalmente, verá los registros de la carga del modelo y, luego, el modelo de prueba se representará en la escena.

Pruebe a mover y girar el elemento GameObject **TestModel** a través de la transformación en el inspector o en la vista de la escena. Verá que el modelo se mueve y gira en la vista de juego.

![Registro de Unity](./media/unity-loading-log.png)

## <a name="provision-blob-storage-in-azure-and-custom-model-ingestion"></a>Aprovisionamiento de Blob Storage en Azure e ingesta de modelos personalizados

Ahora podemos intentar cargar su propio modelo. Para ello, deberá configurar Blob Storage; luego, en Azure, cargará y convertirá un modelo y, después, lo cargará con el script **RemoteRenderedModel**. Los pasos de carga de modelos personalizados se pueden omitir de forma segura si no tiene su propio modelo para cargar en este momento.

Siga los pasos especificados en el [Inicio rápido: Conversión de un modelo para su representación](../../../quickstarts/convert-model.md). Para los fines de este tutorial, omita la sección **Insertar un nuevo modelo en la aplicación de ejemplo del inicio rápido**. Cuando tenga el URI de *firma de acceso compartido (SAS)* del modelo ingerido, continúe con el siguiente paso que se indica a continuación.

## <a name="load-and-rendering-a-custom-model"></a>Carga y representación de un modelo personalizado

1. Cree un elemento GameObject vacío en la escena y llámelo de forma parecida al modelo personalizado.
1. Agregue el script *RemoteRenderedModel* al objeto GameObject recién creado.
 ![Adición del componente RemoteRenderedModel](./media/add-remote-rendered-model-script.png)
1. Rellene `Model Display Name` con un nombre adecuado para el modelo.
1. Rellene `Model Path` con el URI de *firma de acceso compartido (SAS)* del modelo que creó en los pasos de ingesta anteriores.
1. Coloque el elemento GameObject delante de la cámara, en la posición **x = 0, y = 0, z = 3.**
1. Asegúrese de que **AutomaticallyLoad** está activado.
1. Presione **Play** (Reproducir) en el editor de Unity para probar la aplicación.

    Verá que la consola comienza a rellenarse con el estado actual y, finalmente, los mensajes de progreso de la carga del modelo. A continuación, el modelo personalizado se carga en la escena.

1. Quite el objeto de modelo personalizado de la escena. Lo mejor para este tutorial será usar el modelo de prueba. Aunque se admiten varios modelos en ARR, este tutorial se ha escrito para uno solo modelo remoto cada vez.

## <a name="next-steps"></a>Pasos siguientes

Ahora puede cargar sus propios modelos en Azure Remote Rendering y verlos en la aplicación. A continuación, le guiaremos por la manipulación de los modelos.

> [!div class="nextstepaction"]
> [Siguiente: Manipulación de modelos](../manipulate-models/manipulate-models.md)
