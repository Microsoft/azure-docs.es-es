---
title: Refinamiento de materiales, iluminación y efectos
description: Modifique los materiales y la iluminación del modelo. Agregue otros efectos, contornos y planos de corte.
author: florianborn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: 0580614468d4003b3640fd4df08ff02f3a1c8476
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021075"
---
# <a name="tutorial-refining-materials-lighting-and-effects"></a>Tutorial: Refinamiento de materiales, iluminación y efectos

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Resaltar y contornear los modelos y sus componentes
> * Aplicar materiales diferentes a los modelos
> * Segmentar modelos con planos de corte
> * Agregar animaciones sencillas para objetos representados de forma remota

## <a name="prerequisites"></a>Requisitos previos

* Este tutorial se basa en el [Tutorial: Manipulación de modelos](..\manipulate-models\manipulate-models.md).

## <a name="highlighting-and-outlining"></a>Resaltado y contorno

Proporcionar información visual al usuario es una parte importante de la experiencia del usuario en cualquier aplicación. Azure Remote Rendering proporciona mecanismos de información visual mediante [invalidaciones de estado jerárquicas](../../../overview/features/override-hierarchical-state.md). Las invalidaciones de estado jerárquicas se implementan con componentes asociados a las instancias locales de los modelos. Hemos aprendido a crear estas instancias locales en [Sincronización del gráfico de objetos remotos en la jerarquía de Unity](../manipulate-models/manipulate-models.md#synchronizing-the-remote-object-graph-into-the-unity-hierarchy).

En primer lugar, vamos a crear un contenedor en torno al componente [**HierarchicalStateOverrideComponent**](https://docs.microsoft.com/dotnet/api/microsoft.azure.remoterendering.hierarchicalstateoverridecomponent). **HierarchicalStateOverrideComponent** es el script local que controla las invalidaciones en la entidad remota. Los [**recursos del tutorial**](../custom-models/custom-models.md#import-assets-used-by-this-tutorial) incluyen una clase base abstracta llamada **BaseEntityOverrideController**, que ampliaremos para crear el contenedor.

1. Cree un script llamado **EntityOverrideController** y reemplace su contenido por el código siguiente:

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;

    public class EntityOverrideController : BaseEntityOverrideController
    {
        public override event Action<HierarchicalStates> FeatureOverrideChange;

        private ARRHierarchicalStateOverrideComponent localOverride;
        public override ARRHierarchicalStateOverrideComponent LocalOverride
        {
            get
            {
                if (localOverride == null)
                {
                    localOverride = gameObject.GetComponent<ARRHierarchicalStateOverrideComponent>();
                    if (localOverride == null)
                    {
                        localOverride = gameObject.AddComponent<ARRHierarchicalStateOverrideComponent>();
                    }

                    var remoteStateOverride = TargetEntity.Entity.FindComponentOfType<HierarchicalStateOverrideComponent>();

                    if (remoteStateOverride == null)
                    {
                        // if there is no HierarchicalStateOverrideComponent on the remote side yet, create one
                        localOverride.Create(RemoteManagerUnity.CurrentSession);
                    }
                    else
                    {
                        // otherwise, bind our local stateOverride component to the remote component
                        localOverride.Bind(remoteStateOverride);

                    }
                }
                return localOverride;
            }
        }

        private RemoteEntitySyncObject targetEntity;
        public override RemoteEntitySyncObject TargetEntity
        {
            get
            {
                if (targetEntity == null)
                    targetEntity = gameObject.GetComponent<RemoteEntitySyncObject>();
                return targetEntity;
            }
        }

        private HierarchicalEnableState ToggleState(HierarchicalStates feature)
        {
            HierarchicalEnableState setToState = HierarchicalEnableState.InheritFromParent;
            switch (LocalOverride.RemoteComponent.GetState(feature))
            {
                case HierarchicalEnableState.ForceOff:
                case HierarchicalEnableState.InheritFromParent:
                    setToState = HierarchicalEnableState.ForceOn;
                    break;
                case HierarchicalEnableState.ForceOn:
                    setToState = HierarchicalEnableState.InheritFromParent;
                    break;
            }

            return SetState(feature, setToState);
        }

        private HierarchicalEnableState SetState(HierarchicalStates feature, HierarchicalEnableState enableState)
        {
            if (GetState(feature) != enableState) //if this is actually different from the current state, act on it
            {
                LocalOverride.RemoteComponent.SetState(feature, enableState);
                FeatureOverrideChange?.Invoke(feature);
            }

            return enableState;
        }

        public override HierarchicalEnableState GetState(HierarchicalStates feature) => LocalOverride.RemoteComponent.GetState(feature);

        public override void ToggleHidden() => ToggleState(HierarchicalStates.Hidden);

        public override void ToggleSelect() => ToggleState(HierarchicalStates.Selected);

        public override void ToggleSeeThrough() => ToggleState(HierarchicalStates.SeeThrough);

        public override void ToggleTint(Color tintColor = default)
        {
            if (tintColor != default) LocalOverride.RemoteComponent.TintColor = tintColor.toRemote();
            ToggleState(HierarchicalStates.UseTintColor);
        }

        public override void ToggleDisabledCollision() => ToggleState(HierarchicalStates.DisableCollision);

        public override void RemoveOverride()
        {
            var remoteStateOverride = TargetEntity.Entity.FindComponentOfType<HierarchicalStateOverrideComponent>();
            if (remoteStateOverride != null)
            {
                remoteStateOverride.Destroy();
            }

            if (localOverride == null)
                localOverride = gameObject.GetComponent<ARRHierarchicalStateOverrideComponent>();

            if (localOverride != null)
            {
                Destroy(localOverride);
            }
        }
    }
    ```

La misión principal de **LocalOverride** es crear un vínculo entre sí mismo y su valor de `RemoteComponent`. **LocalOverride** nos permite establecer marcas de estado en el componente local, que están enlazadas a la entidad remota. Las invalidaciones y sus estados se describen en la página [Invalidaciones de estado jerárquicas](../../../overview/features/override-hierarchical-state.md). 

Esta implementación alterna un estado cada vez. Sin embargo, es totalmente posible combinar varias invalidaciones en entidades únicas y crear combinaciones en distintos niveles de la jerarquía. Por ejemplo, la combinación de `Selected` y `SeeThrough` en un único componente le daría un contorno y, al mismo tiempo, lo haría transparente. También, establecer la invalidación `Hidden` de la entidad raíz en `ForceOn` y, al mismo tiempo, convertir la invalidación `Hidden` de una entidad secundaria en `ForceOff`, ocultaría todo excepto el elemento secundario con la invalidación.

Para aplicar estados a las entidades, podemos modificar el elemento **RemoteEntityHelper** creado anteriormente.

1. Modifique la clase **RemoteEntityHelper** para implementar la clase abstracta **BaseRemoteEntityHelper**. Esta modificación permitirá el uso de un controlador de vista proporcionado en los **recursos del tutorial**. Cuando se modifique, debería tener este aspecto:

    ```csharp
    public class RemoteEntityHelper : BaseRemoteEntityHelper
    ```

2. Invalide los métodos abstractos mediante el código siguiente:

    ```csharp
    public override BaseEntityOverrideController EnsureOverrideComponent(Entity entity)
    {
        var entityGameObject = entity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
        var overrideComponent = entityGameObject.GetComponent<EntityOverrideController>();
        if (overrideComponent == null)
            overrideComponent = entityGameObject.AddComponent<EntityOverrideController>();
        return overrideComponent;
    }

    public override HierarchicalEnableState GetState(Entity entity, HierarchicalStates feature)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        return overrideComponent.GetState(feature);
    }

    public override void ToggleHidden(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleHidden();
    }

    public override void ToggleSelect(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleSelect();
    }

    public override void ToggleSeeThrough(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleSeeThrough();
    }

    public Color TintColor = new Color(0.0f, 1.0f, 0.0f, 0.1f);
    public override void ToggleTint(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleTint(TintColor);
    }

    public override void ToggleDisableCollision(Entity entity)
    {
        var overrideComponent = EnsureOverrideComponent(entity);
        overrideComponent.ToggleHidden();
    }

    public override void RemoveOverrides(Entity entity)
    {
        var entityGameObject = entity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
        var overrideComponent = entityGameObject.GetComponent<EntityOverrideController>();
        if (overrideComponent != null)
        {
            overrideComponent.RemoveOverride();
            Destroy(overrideComponent);
        }
    }
    ```

Este código garantiza que se agrega un componente **EntityOverrideController** a la entidad de destino y que luego se llama a uno de los métodos de alternancia. Si lo desea, en el elemento GameObject **TestModel**, puede hacer una llamada a estos métodos auxiliares agregando **RemoteEntityHelper** como devolución de llamada al evento `OnRemoteEntityClicked` en el componente **RemoteRayCastPointerHandler**.

![Devoluciones de llamada del puntero](./media/pointer-event-callbacks.png)

Ahora que estos scripts se han agregado al modelo, una vez que se ha conectado al entorno de ejecución, el controlador de vista **AppMenu** debe tener habilitadas interfaces adicionales para interactuar con el script **EntityOverrideController**. Consulte el menú **Model Tools** (Herramientas de modelo) para ver los controladores de vista desbloqueados.

En este momento, los componentes del elemento GameObject **TestModel** deben tener un aspecto parecido a este:

![Modelo de prueba con scripts adicionales](./media/test-model-updated.png)

Este es un ejemplo de apilamiento de invalidaciones en una sola entidad. Hemos usado `Select` y `Tint` para proporcionar un contorno y un color:

![Selección del matiz del modelo de prueba](./media/select-tint-test-model.png)

## <a name="cut-planes"></a>Corte de planos

Los [planos de corte](../../../overview/features/cut-planes.md) son una característica que se puede agregar a cualquier entidad remota. Normalmente, creará una entidad remota que no esté asociada a ningún dato de la malla para que contenga el componente de plano de corte. La posición y la orientación del plano de corte se determinan por la posición y la orientación de la entidad remota a la que está asociada.

Vamos a crear un script que cree automáticamente una entidad remota, agregue un componente de plano de corte y sincronice la transformación de un objeto local con la entidad del plano de corte. Luego, podemos usar **CutPlaneViewController** para encapsular el plano de corte en una interfaz que nos permita manipularlo.

1. Cree un script llamado **RemoteCutPlane** y reemplace su código por el siguiente:

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;

    public class RemoteCutPlane : BaseRemoteCutPlane
    {
        public Color SliceColor = new Color(0.5f, 0f, 0f, .5f);
        public float FadeLength = 0.01f;
        public Axis SliceNormal = Axis.Y_Neg;

        public bool AutomaticallyCreate = true;

        private CutPlaneComponent remoteCutPlaneComponent;
        private bool cutPlaneReady = false;

        public override bool CutPlaneReady 
        { 
            get => cutPlaneReady;
            set 
            { 
                cutPlaneReady = value;
                CutPlaneReadyChanged?.Invoke(cutPlaneReady);
            }
        }

        public override event Action<bool> CutPlaneReadyChanged;

        public UnityBoolEvent OnCutPlaneReadyChanged = new UnityBoolEvent();

        public void Start()
        {
            // Hook up the event to the Unity event
            CutPlaneReadyChanged += (ready) => OnCutPlaneReadyChanged?.Invoke(ready);

            RemoteRenderingCoordinator.CoordinatorStateChange += RemoteRenderingCoordinator_CoordinatorStateChange;
            RemoteRenderingCoordinator_CoordinatorStateChange(RemoteRenderingCoordinator.instance.CurrentCoordinatorState);
        }

        private void RemoteRenderingCoordinator_CoordinatorStateChange(RemoteRenderingCoordinator.RemoteRenderingState state)
        {
            switch (state)
            {
                case RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected:
                    if (AutomaticallyCreate)
                        CreateCutPlane();
                    break;
                default:
                    DestroyCutPlane();
                    break;
            }
        }

        public override void CreateCutPlane()
        {
            //Implement me
        }

        public override void DestroyCutPlane()
        {
            //Implement me
        }
    }
    ```

    Este código amplía la clase **BaseRemoteCutPlane** incluida en los **recursos del tutorial**. Al igual que el modelo representado de forma remota, este script asocia y escucha los cambios de `RemoteRenderingState` desde el coordinador remoto. Cuando el coordinador alcanza el estado `RuntimeConnected`, intentará conectarse automáticamente en caso de que deba hacerlo. También hay una variable `CutPlaneComponent` a la que haremos un seguimiento. Este es el componente de Azure Remote Rendering que se sincroniza con el plano de corte en la sesión remota. Vamos a ver lo que debemos hacer para crear el plano de corte.

2. Reemplace el método `CreateCutPlane()` por la versión completada siguiente:

    ```csharp
    public override void CreateCutPlane()
    {
        if (remoteCutPlaneComponent != null)
            return; //Nothing to do!

        //Create a root object for the cut plane
        var cutEntity = RemoteRenderingCoordinator.CurrentSession.Actions.CreateEntity();

        //Bind the remote entity to this game object
        cutEntity.BindToUnityGameObject(this.gameObject);

        //Sync the transform of this object so we can move the cut plane
        var syncComponent = this.gameObject.GetComponent<RemoteEntitySyncObject>();
        syncComponent.SyncEveryFrame = true;

        //Add a cut plane to the entity
        remoteCutPlaneComponent = RemoteRenderingCoordinator.CurrentSession.Actions.CreateComponent(ObjectType.CutPlaneComponent, cutEntity) as CutPlaneComponent;

        //Configure the cut plane
        remoteCutPlaneComponent.Normal = SliceNormal;
        remoteCutPlaneComponent.FadeColor = SliceColor.toRemote();
        remoteCutPlaneComponent.FadeLength = FadeLength;
        CutPlaneReady = true;
    }
    ```

    Aquí vamos a crear una entidad remota y a enlazarla a un elemento GameObject local. Para asegurarnos de que la entidad remota tenga sincronizada su transformación con la transformación local, estableceremos `SyncEveryFrame` en `true`. Luego, usaremos la llamada `CreateComponent` para agregar un elemento `CutPlaneComponent` al objeto remoto. Por último, configuraremos el plano de corte con la configuración definida en la parte superior de MonoBehaviour. Vamos a ver lo que se tarda en limpiar un plano de corte con la implementación del método `DestroyCutPlane()`.

3. Reemplace el método `DestroyCutPlane()` por la versión completada siguiente:

    ```csharp
    public override void DestroyCutPlane()
    {
        if (remoteCutPlaneComponent == null)
            return; //Nothing to do!

        remoteCutPlaneComponent.Owner.Destroy();
        remoteCutPlaneComponent = null;
        CutPlaneReady = false;
    }
    ```

Dado que el objeto remoto es bastante simple y solo estamos limpiando el extremo remoto (y manteniendo nuestro objeto local), basta con llamar a `Destroy` en el objeto remoto y borrar nuestra referencia a él.

**AppMenu** incluye un controlador de vista que se asociará automáticamente al plano de corte y le permitirá interactuar con él. Aunque no es necesario usar **AppMenu** ni ninguno de los controladores de vista, mejoran la experiencia. Ahora pruebe el plano de corte y su controlador de vista.

1. Cree un elemento GameObject vacío en la escena y llámelo **CutPlane**.
1. Agregue el componente **RemoteCutPlane** al elemento GameObject **CutPlane**.

   ![Configuración del componente de plano de corte](./media/cut-plane-config.png)

1. Presione el botón de reproducción en el editor de Unity para cargarlo y conectarse a una sesión remota.
1. Con la simulación de manos de MRTK, agarre y gire (mantenga presionada la tecla Ctrl para girar) el elemento CutPlane para moverlo por la escena. Vea cómo se segmenta en **TestModel** para revelar los componentes internos.

![Ejemplo de plano de corte](./media/cut-plane-example-engine.png)

## <a name="configuring-the-remote-lighting"></a>Configuración de la iluminación remota

La sesión de representación remota admite un espectro completo de [opciones de iluminación](../../../overview/features/lights.md). Vamos a crear scripts para la [textura del cielo](../../../overview/features/sky.md) y un mapa sencillo para dos tipos de luz de Unity que se usarán con la representación remota.

### <a name="sky-texture"></a>Textura del cielo

Al cambiar la textura del cielo, se puede elegir entre una serie de mapas de cubo integrados. Estos se cargan en la sesión y se aplican a la textura del cielo. También es posible [cargar sus propias texturas](../../../concepts/textures.md) y usarlas como luces del cielo.

Vamos a crear un script **RemoteSky** que tenga una lista de los mapas de cubo disponibles integrados en forma de parámetros de carga. Luego, dejaremos que el usuario seleccione y cargue una de las opciones.

1. Cree un script llamado **RemoteSky** y reemplace todo su contenido por el código siguiente:

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using System;
    using System.Collections.Generic;
    using UnityEngine;

    public class RemoteSky : BaseRemoteSky
    {
        public override Dictionary<string, LoadTextureFromSASParams> AvailableCubemaps => builtInTextures;

        private bool canSetSky;
        public override bool CanSetSky
        {
            get => canSetSky;
            set
            {
                canSetSky = value;
                CanSetSkyChanged?.Invoke(canSetSky);
            }
        }

        private string currentSky = "DefaultSky";
        public override string CurrentSky
        {
            get => currentSky;
            protected set
            {
                currentSky = value;
                SkyChanged?.Invoke(value);
            }
        }

        private Dictionary<string, LoadTextureFromSASParams> builtInTextures = new Dictionary<string, LoadTextureFromSASParams>()
        {
            {"Autoshop",new LoadTextureFromSASParams("builtin://Autoshop", TextureType.CubeMap)},
            {"BoilerRoom",new LoadTextureFromSASParams("builtin://BoilerRoom", TextureType.CubeMap)},
            {"ColorfulStudio",new LoadTextureFromSASParams("builtin://ColorfulStudio", TextureType.CubeMap)},
            {"Hangar",new LoadTextureFromSASParams("builtin://Hangar", TextureType.CubeMap)},
            {"IndustrialPipeAndValve",new LoadTextureFromSASParams("builtin://IndustrialPipeAndValve", TextureType.CubeMap)},
            {"Lebombo",new LoadTextureFromSASParams("builtin://Lebombo", TextureType.CubeMap)},
            {"SataraNight",new LoadTextureFromSASParams("builtin://SataraNight", TextureType.CubeMap)},
            {"SunnyVondelpark",new LoadTextureFromSASParams("builtin://SunnyVondelpark", TextureType.CubeMap)},
            {"Syferfontein",new LoadTextureFromSASParams("builtin://Syferfontein", TextureType.CubeMap)},
            {"TearsOfSteelBridge",new LoadTextureFromSASParams("builtin://TearsOfSteelBridge", TextureType.CubeMap)},
            {"VeniceSunset",new LoadTextureFromSASParams("builtin://VeniceSunset", TextureType.CubeMap)},
            {"WhippleCreekRegionalPark",new LoadTextureFromSASParams("builtin://WhippleCreekRegionalPark", TextureType.CubeMap)},
            {"WinterRiver",new LoadTextureFromSASParams("builtin://WinterRiver", TextureType.CubeMap)},
            {"DefaultSky",new LoadTextureFromSASParams("builtin://DefaultSky", TextureType.CubeMap)}
        };

        public UnityBoolEvent OnCanSetSkyChanged;
        public override event Action<bool> CanSetSkyChanged;

        public UnityStringEvent OnSkyChanged;
        public override event Action<string> SkyChanged;

        public void Start()
        {
            // Hook up the event to the Unity event
            CanSetSkyChanged += (canSet) => OnCanSetSkyChanged?.Invoke(canSet);
            SkyChanged += (key) => OnSkyChanged?.Invoke(key);

            RemoteRenderingCoordinator.CoordinatorStateChange += ApplyStateToView;
            ApplyStateToView(RemoteRenderingCoordinator.instance.CurrentCoordinatorState);
        }

        private void ApplyStateToView(RemoteRenderingCoordinator.RemoteRenderingState state)
        {
            switch (state)
            {
                case RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected:
                    CanSetSky = true;
                    break;
                default:
                    CanSetSky = false;
                    break;
            }
        }

        public override async void SetSky(string skyKey)
        {
            if (!CanSetSky)
            {
                Debug.Log("Unable to set sky right now");
                return;
            }

            if (AvailableCubemaps.ContainsKey(skyKey))
            {
                Debug.Log("Setting sky to " + skyKey);
                //Load the texture into the session
                var texture = await RemoteRenderingCoordinator.CurrentSession.Actions.LoadTextureFromSASAsync(AvailableCubemaps[skyKey]).AsTask();

                //Apply the texture to the SkyReflectionSettings
                RemoteRenderingCoordinator.CurrentSession.Actions.SkyReflectionSettings.SkyReflectionTexture = texture;
                SkyChanged?.Invoke(skyKey);
            }
            else
            {
                Debug.Log("Invalid sky key");
            }
        }
    }
    ```

    La parte más importante de este código son solo unas pocas líneas:

    ```csharp
    //Load the texture into the session
    var texture = await RemoteRenderingCoordinator.CurrentSession.Actions.LoadTextureFromSASAsync(AvailableCubemaps[skyKey]).AsTask();

    //Apply the texture to the SkyReflectionSettings
    RemoteRenderingCoordinator.CurrentSession.Actions.SkyReflectionSettings.SkyReflectionTexture = texture;
    ```

    Aquí, se obtiene una referencia a la textura que se va a usar, para lo cual se carga en la sesión desde el almacenamiento de blobs integrado. Después, solo hará falta asignar esa textura al elemento `SkyReflectionTexture` de la sesión para aplicarla.

1. Cree un elemento GameObject vacío en la escena y llámelo **SkyLight**.

1. Agregue el script **RemoteSky** al elemento GameObject **SkyLight**.

    El cambio entre las luces del cielo puede realizarse mediante la llamada a `SetSky` con una de las claves de cadena definidas en `AvailableCubemaps`. El controlador de vista integrado en **AppMenu** crea automáticamente botones y enlaza sus eventos para llamar a `SetSky` con su clave respectiva.
1. Presione el botón de reproducción en el editor de Unity y autorice una conexión.
1. Después de conectar el entorno de ejecución local a una sesión remota, vaya a **AppMenu -> Session Tools -> Remote Sky** (AppMenú > Herramientas de sesión > Cielo remoto), para explorar las diferentes opciones de cielo y ver cómo afectan a **TestModel**.

### <a name="scene-lights"></a>Iluminación de escenas

Entre las luces de las escenas remotas están las puntuales, las focales y las direccionales. Al igual que el plano de corte que hemos creado anteriormente, estas luces de escena son entidades remotas con componentes asociados. Una cuestión importante al iluminar la escena remota es intentar buscar una coincidencia con la iluminación de la escena local. Esta estrategia no siempre es posible porque muchas aplicaciones de Unity para HoloLens 2 no usan la representación física de los objetos representados localmente. Sin embargo, en un nivel determinado, podemos simular la iluminación predeterminada más sencilla de Unity.

1. Cree un script llamado **RemoteLight** y reemplace su código por el siguiente:

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;

    [RequireComponent(typeof(Light))]
    public class RemoteLight : BaseRemoteLight
    {
        public bool AutomaticallyCreate = true;

        private bool lightReady = false;
        public override bool LightReady 
        {
            get => lightReady;
            set
            {
                lightReady = value;
                LightReadyChanged?.Invoke(lightReady);
            }
        }

        private ObjectType remoteLightType = ObjectType.Invalid;
        public override ObjectType RemoteLightType => remoteLightType;

        public UnityBoolEvent OnLightReadyChanged;

        public override event Action<bool> LightReadyChanged;

        private Light localLight; //Unity Light

        private Entity lightEntity;
        private LightComponentBase remoteLightComponent; //Remote Rendering Light

        private void Awake()
        {
            localLight = GetComponent<Light>();
            switch (localLight.type)
            {
                case LightType.Directional:
                    remoteLightType = ObjectType.DirectionalLightComponent;
                    break;
                case LightType.Point:
                    remoteLightType = ObjectType.PointLightComponent;
                    break;
                case LightType.Spot:
                case LightType.Area:
                    //Not supported in tutorial
                case LightType.Disc:
                    // No direct analog in remote rendering
                    remoteLightType = ObjectType.Invalid;
                    break;
            }
        }

        public void Start()
        {
            // Hook up the event to the Unity event
            LightReadyChanged += (ready) => OnLightReadyChanged?.Invoke(ready);

            RemoteRenderingCoordinator.CoordinatorStateChange += RemoteRenderingCoordinator_CoordinatorStateChange;
            RemoteRenderingCoordinator_CoordinatorStateChange(RemoteRenderingCoordinator.instance.CurrentCoordinatorState);
        }

        public void OnDestroy()
        {
            lightEntity?.Destroy();
        }

        private void RemoteRenderingCoordinator_CoordinatorStateChange(RemoteRenderingCoordinator.RemoteRenderingState state)
        {
            switch (state)
            {
                case RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected:
                    if (AutomaticallyCreate)
                        CreateLight();
                    break;
                default:
                    DestroyLight();
                    break;
            }
        }

        public override void CreateLight()
        {
            if (remoteLightComponent != null)
                return; //Nothing to do!

            //Create a root object for the light
            if(lightEntity == null)
                lightEntity = RemoteRenderingCoordinator.CurrentSession.Actions.CreateEntity();

            //Bind the remote entity to this game object
            lightEntity.BindToUnityGameObject(this.gameObject);

            //Sync the transform of this object so we can move the light
            var syncComponent = this.gameObject.GetComponent<RemoteEntitySyncObject>();
            syncComponent.SyncEveryFrame = true;

            //Add a light to the entity
            switch (RemoteLightType)
            {
                case ObjectType.DirectionalLightComponent:
                    var remoteDirectional = RemoteRenderingCoordinator.CurrentSession.Actions.CreateComponent(ObjectType.DirectionalLightComponent, lightEntity) as DirectionalLightComponent;
                    //No additional properties
                    remoteLightComponent = remoteDirectional;
                    break;

                case ObjectType.PointLightComponent:
                    var remotePoint = RemoteRenderingCoordinator.CurrentSession.Actions.CreateComponent(ObjectType.PointLightComponent, lightEntity) as PointLightComponent;
                    remotePoint.Radius = 0;
                    remotePoint.Length = localLight.range;
                    //remotePoint.AttenuationCutoff = //No direct analog in Unity legacy lights
                    //remotePoint.ProjectedCubeMap = //No direct analog in Unity legacy lights

                    remoteLightComponent = remotePoint;
                    break;
                default:
                    LightReady = false;
                    return;
            }

            // Set the common values for all light types
            UpdateRemoteLightSettings();

            LightReady = true;
        }

        public override void UpdateRemoteLightSettings()
        {
            remoteLightComponent.Color = localLight.color.toRemote();
            remoteLightComponent.Intensity = localLight.intensity;
        }

        public override void DestroyLight()
        {
            if (remoteLightComponent == null)
                return; //Nothing to do!

            remoteLightComponent.Destroy();
            remoteLightComponent = null;
            LightReady = false;
        }

        [ContextMenu("Sync Remote Light Configuration")]
        public override void RecreateLight()
        {
            DestroyLight();
            CreateLight();
        }

        public override void SetIntensity(float intensity)
        {
            localLight.intensity = Mathf.Clamp(intensity, 0, 1);
            UpdateRemoteLightSettings();
        }

        public override void SetColor(Color color)
        {
            localLight.color = color;
            UpdateRemoteLightSettings();
        }
    }
    ```

    Este script crea distintos tipos de luces remotas según el tipo de la luz local de Unity a la que está asociado el script. La luz remota duplicará la luz local en su posición, rotación, color e intensidad. Siempre que sea posible, la luz remota también establecerá una configuración adicional. Esta no es una coincidencia perfecta, ya que las luces de Unity no son luces PBR.

1. Busque el elemento GameObject **DirectionalLight** en la escena. Si ha quitado el valor predeterminado de **DirectionalLight** de la escena, en la barra de menús superior, seleccione *GameObject-> Light-> DirectionalLight* (GameObject-> Luz-> DirectionalLight) para crear una luz en la escena.

1. Seleccione el elemento GameObject **DirectionalLight** y, mediante el botón **Add Component** (Agregar componente), agregue el script **RemoteLight**.

1. Dado que este script implementa la clase base `BaseRemoteLight`, puede usar el controlador de vista de **AppMenu** proporcionado para interactuar con la luz remota. Vaya a **AppMenu -> Session Tools -> Directional Light** (AppMenu -> Herramientas de sesión -> Luz direccional).

    > [!NOTE]
    > Por motivos de simplicidad, la interfaz de usuario de **AppMenu** se ha limitado a una sola luz direccional. Sin embargo, todavía es posible, y se recomienda, agregar luces puntuales y asociarles el script **RemoteLight**. Esas luces adicionales se pueden modificar editando las propiedades de la luz de Unity en el editor. Deberá sincronizar manualmente los cambios locales en la luz remota mediante el menú contextual **RemoteLight** del inspector:
    >
    > ![Sincronización manual de las luces remotas](./media/sync-remote-light.png)

1. Presione el botón de reproducción en el editor de Unity y autorice una conexión.

1. Después de conectar el entorno de ejecución a una sesión remota, coloque y apunte la cámara (use WASD y haga clic con el botón derecho y mueva el mouse) para que aparezca el controlador de vista de luz direccional. 
1. Use el controlador de vista de luz remota para modificar las propiedades de la luz. Con la simulación de manos de MRTK, agarre y gire (mantenga presionada la tecla Ctrl para girar) la luz direccional para ver el efecto en la iluminación de la escena.

    ![Luz direccional](./media/directional-light-test.png)

## <a name="editing-materials"></a>Modificación de materiales

Los [materiales](../../../concepts/materials.md) representados de forma remota pueden modificarse para proporcionar efectos visuales adicionales, ajustar los objetos visuales de los modelos representados o proporcionar información adicional a los usuarios. Existen muchas formas de modificar un material y muchas razones para hacerlo. Aquí le mostraremos cómo cambiar el color de albedo de un material y modificar la rugosidad y la composición metálica de un material PBR.

> [!NOTE]
> En muchos casos, si una característica o un efecto se pueden implementar con **HierarchicalStateOverrideComponent**, es mejor usarlos que modificar el material.

Vamos a crear un script que acepte una entidad de destino y configure algunos objetos `OverrideMaterialProperty` para cambiar las propiedades del material de dicha entidad. Para comenzar, obtendremos el elemento [**MeshComponent**](../../../concepts/meshes.md#meshcomponent) de la entidad, que contiene una lista de materiales usados en la malla. Para simplificar las cosas, usaremos solo el primer material encontrado. Esta sencilla estrategia es fácil que no funcione bien según cómo se haya creado el contenido, por lo que probablemente quiera adoptar un enfoque más complejo para seleccionar el material adecuado.

A partir del material, podemos acceder a valores comunes como albedo. En primer lugar, los materiales deben convertirse en su tipo adecuado, `PbrMaterial` o `ColorMaterial`, para recuperar sus valores, como se observa en el método **GetMaterialColor**. Cuando tengamos una referencia al material deseado, basta con establecer los valores y ARR administrará la sincronización entre las propiedades del material local y el material remoto.

1. Cree un script llamado **EntityMaterialController** y reemplace su contenido por el código siguiente:

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using System.Linq;
    using UnityEngine;
    // to prevent namespace conflicts
    using ARRMaterial = Microsoft.Azure.RemoteRendering.Material;

    public class EntityMaterialController : BaseEntityMaterialController
    {
        public override bool RevertOnEntityChange { get; set; } = true;

        public override OverrideMaterialProperty<Color> ColorOverride { get; set; }
        public override OverrideMaterialProperty<float> RoughnessOverride { get; set; }
        public override OverrideMaterialProperty<float> MetalnessOverride { get; set; }

        private Entity targetEntity;
        public override Entity TargetEntity
        {
            get => targetEntity;
            set
            {
                if (targetEntity != value)
                {
                    if (targetEntity != null && RevertOnEntityChange)
                    {
                        Revert();
                    }

                    targetEntity = value;
                    ConfigureTargetEntity();
                    TargetEntityChanged?.Invoke(value);
                }
            }
        }

        private ARRMaterial targetMaterial;
        private ARRMeshComponent meshComponent;

        public override event Action<Entity> TargetEntityChanged;
        public UnityRemoteEntityEvent OnTargetEntityChanged;

        public void Start()
        {
            // Forward events to Unity events
            TargetEntityChanged += (entity) => OnTargetEntityChanged?.Invoke(entity);

            // If there happens to be a remote RayCaster on this object, assume we should listen for events from it
            if (GetComponent<BaseRemoteRayCastPointerHandler>() != null)
                GetComponent<BaseRemoteRayCastPointerHandler>().RemoteEntityClicked += (entity) => TargetEntity = entity;
        }

        protected override void ConfigureTargetEntity()
        {
            //Get the Unity object, to get the sync object, to get the mesh component, to get the material.
            var targetEntityGameObject = TargetEntity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);

            var localSyncObject = targetEntityGameObject.GetComponent<RemoteEntitySyncObject>();
            meshComponent = targetEntityGameObject.GetComponent<ARRMeshComponent>();
            if (meshComponent == null)
            {
                var mesh = localSyncObject.Entity.FindComponentOfType<MeshComponent>();
                if (mesh != null)
                {
                    targetEntityGameObject.BindArrComponent<ARRMeshComponent>(mesh);
                    meshComponent = targetEntityGameObject.GetComponent<ARRMeshComponent>();
                }
            }

            meshComponent.enabled = true;

            targetMaterial = meshComponent.RemoteComponent.Mesh.Materials.FirstOrDefault();
            if (targetMaterial == default)
            {
                return;
            }

            ColorOverride = new OverrideMaterialProperty<Color>(
                GetMaterialColor(targetMaterial), //The original value
                targetMaterial, //The target material
                ApplyMaterialColor); //The action to take to apply the override

            //If the material is a PBR material, we can override some additional values
            if (targetMaterial.MaterialSubType == MaterialType.Pbr)
            {
                var firstPBRMaterial = (PbrMaterial)targetMaterial;

                RoughnessOverride = new OverrideMaterialProperty<float>(
                    firstPBRMaterial.Roughness, //The original value
                    targetMaterial, //The target material
                    ApplyRoughnessValue); //The action to take to apply the override

                MetalnessOverride = new OverrideMaterialProperty<float>(
                    firstPBRMaterial.Metalness, //The original value
                    targetMaterial, //The target material
                    ApplyMetalnessValue); //The action to take to apply the override
            }
            else //otherwise, ensure the overrides are cleared out from any previous entity
            {
                RoughnessOverride = null;
                MetalnessOverride = null;
            }
        }

        public override void Revert()
        {
            if (ColorOverride != null)
                ColorOverride.OverrideActive = false;

            if (RoughnessOverride != null)
                RoughnessOverride.OverrideActive = false;

            if (MetalnessOverride != null)
                MetalnessOverride.OverrideActive = false;
        }

        private Color GetMaterialColor(ARRMaterial material)
        {
            if (material == null)
                return default;

            if (material.MaterialSubType == MaterialType.Color)
                return ((ColorMaterial)material).AlbedoColor.toUnity();
            else
                return ((PbrMaterial)material).AlbedoColor.toUnity();
        }

        private void ApplyMaterialColor(ARRMaterial material, Color color)
        {
            if (material == null)
                return;

            if (material.MaterialSubType == MaterialType.Color)
                ((ColorMaterial)material).AlbedoColor = color.toRemoteColor4();
            else
                ((PbrMaterial)material).AlbedoColor = color.toRemoteColor4();
        }

        private void ApplyRoughnessValue(ARRMaterial material, float value)
        {
            if (material == null)
                return;

            if (material.MaterialSubType == MaterialType.Pbr) //Only PBR has Roughness
                ((PbrMaterial)material).Roughness = value;
        }

        private void ApplyMetalnessValue(ARRMaterial material, float value)
        {
            if (material == null)
                return;

            if (material.MaterialSubType == MaterialType.Pbr) //Only PBR has Metalness
                ((PbrMaterial)material).Metalness = value;
        }
    }
    ```

El tipo `OverrideMaterialProperty` debe ser lo suficientemente flexible como para permitir que se cambien algunos otros valores del material, si así se desea. El tipo `OverrideMaterialProperty` realiza un seguimiento del estado de una invalidación, mantiene los valores anterior y nuevo, y usa un delegado para establecer la invalidación. Como ejemplo, examine el elemento `ColorOverride`:

```csharp
ColorOverride = new OverrideMaterialProperty<Color>(
    GetMaterialColor(targetMaterial), //The original value
    targetMaterial, //The target material
    ApplyMaterialColor); //The action to take to apply the override
```

Se crea un elemento `OverrideMaterialProperty` donde la invalidación encapsulará el tipo `Color`. Proporcionamos el color actual u original en el momento en que se crea la invalidación. También le damos un material ARR para usar. Por último, se proporciona un delegado que aplicará la invalidación. El delegado es un método que acepta un material ARR y el tipo que encapsula la invalidación. Este método es la parte más importante para saber cómo ARR ajusta los valores del material.

Para realizar su trabajo, `ColorOverride` usa el método `ApplyMaterialColor`:

```csharp
private void ApplyMaterialColor(ARRMaterial material, Color color)
{
    if (material.MaterialSubType == MaterialType.Color)
        ((ColorMaterial)material).AlbedoColor = color.toRemoteColor4();
    else
        ((PbrMaterial)material).AlbedoColor = color.toRemoteColor4();
}
```

Este código acepta un material y un color. Comprueba qué tipo de material es y, luego, hace una conversión del material para aplicar el color.

`RoughnessOverride` y `MetalnessOverride` funcionan de forma similar: ambos usan los métodos `ApplyRoughnessValue` y `ApplyMetalnessValue` para hacer su trabajo.

A continuación, vamos a probar el controlador de materiales.

1. Agregue el script **EntityMaterialController** al elemento GameObject **TestModel**.
1. Presione el botón de reproducción en Unity para iniciar la escena y conectarse a ARR.
1. Después de conectar el entorno de ejecución a una sesión remota y cargar el modelo, vaya a **AppMenu -> Model Tools -> Edit Material** (AppMenu -> Herramientas de modelo -> Editar material.
1. Seleccione una entidad del modelo con las manos simuladas para hacer clic en **TestModel**.
1. Confirme que el controlador de vista de material (**AppMenu-> Model Tools-> Edit Material** [AppMenu > Herramientas de modelo > Editar material]) se ha actualizado con la entidad de destino.
1. Use el controlador de vista de material para ajustar el material en la entidad de destino.

Dado que solo estamos modificando el primer material de la malla, es posible cómo cambia el material. Use la invalidación jerárquica **SeeThrough** para ver si el material que está cambiando está dentro de la malla.

![Ejemplo de modificación de materiales](./media/material-edit-example.png)

## <a name="next-steps"></a>Pasos siguientes

Felicidades. Ahora ha implementado toda la funcionalidad básica de Azure Remote Rendering. En el siguiente capítulo, veremos cómo proteger Azure Remote Rendering y Blob Storage. Estos serán los primeros pasos para lanzar una aplicación comercial que use Azure Remote Rendering.

> [!div class="nextstepaction"]
> [Siguiente: Protección de Azure Remote Rendering y el almacenamiento de modelos](../security/security.md)