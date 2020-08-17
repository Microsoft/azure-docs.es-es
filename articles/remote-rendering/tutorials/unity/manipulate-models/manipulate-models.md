---
title: Manipulación de modelos
description: 'Manipule los modelos representados de forma remota: muévalos, gírelos, cambie su escala, etc.'
author: florianborn71
ms.author: flborn
ms.date: 06/15/2020
ms.topic: tutorial
ms.openlocfilehash: ea951943c3f48443e4348d633c16ed61303f7aa8
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449055"
---
# <a name="tutorial-manipulating-models"></a>Tutorial: Manipulación de modelos

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Agregar objetos visuales y límites de manipulación alrededor de los modelos representados de forma remota
> * Mover, girar y cambiar la escala
> * Proyectar rayos con consultas espaciales
> * Agregar animaciones sencillas para objetos representados de forma remota

## <a name="prerequisites"></a>Requisitos previos

* Este tutorial se basa en el [Tutorial: Interfaces y modelos personalizados](../custom-models/custom-models.md).

## <a name="query-remote-object-bounds-and-apply-to-local-bounds"></a>Consulta de los límites de objetos remotos y aplicación a los límites locales

Para interactuar con objetos remotos, se necesita primero una representación local con la que interactuar. Los [límites de objetos](../../../concepts/object-bounds.md) son útiles para la manipulación rápida de un objeto remoto. Los límites remotos se pueden consultar desde ARR, usando como referencia la entidad local. Los límites se consultan después de que el modelo se haya cargado en la sesión remota.

Los límites de un modelo se definen mediante el cuadro que contiene el modelo completo, igual que [**BoxCollider**](https://docs.unity3d.com/Manual/class-BoxCollider.html) de Unity, que tiene un centro y un tamaño definidos para los ejes x, y, z. De hecho, usaremos **BoxCollider** de Unity para representar los límites del modelo remoto.

1. Cree un script en el mismo directorio que **RemoteRenderedModel** y llámelo **RemoteBounds**.
1. Reemplace el contenido del script por el código siguiente:

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System;
    using UnityEngine;

    [RequireComponent(typeof(BaseRemoteRenderedModel))]
    public class RemoteBounds : BaseRemoteBounds
    {
        //Remote bounds works with a specific remotely rendered model
        private BaseRemoteRenderedModel targetModel = null;

        private BoundsQueryAsync remoteBoundsQuery = null;

        private RemoteBoundsState currentBoundsState = RemoteBoundsState.NotReady;

        public override RemoteBoundsState CurrentBoundsState
        {
            get => currentBoundsState;
            protected set
            {
                if (currentBoundsState != value)
                {
                    currentBoundsState = value;
                    BoundsStateChange?.Invoke(value);
                }
            }
        }

        public override event Action<RemoteBoundsState> BoundsStateChange;

        public void Awake()
        {
            BoundsStateChange += HandleUnityEvents;
            targetModel = GetComponent<BaseRemoteRenderedModel>();

            targetModel.ModelStateChange += TargetModel_OnModelStateChange;
            TargetModel_OnModelStateChange(targetModel.CurrentModelState);
        }

        private void TargetModel_OnModelStateChange(ModelState state)
        {
            switch (state)
            {
                case ModelState.Loaded:
                    QueryBounds();
                    break;
                default:
                    BoundsBoxCollider.enabled = false;
                    CurrentBoundsState = RemoteBoundsState.NotReady;
                    break;
            }
        }

        // Create a query using the model entity
        private void QueryBounds()
        {
            //Implement me
        }

        // Check the result and apply it to the local Unity bounding box if it was successful
        private void ProcessQueryResult(BoundsQueryAsync remoteBounds)
        {
            //Implement me
        }
    }
    ```

    > [!NOTE]
    > Si ve un error en la declaración de Visual Studio que dice *La característica "X" no está disponible en C# 6, use la versión de idioma 7.0 o posterior*, puede omitirlo de forma segura. Está relacionado con la generación de soluciones y proyectos de Unity.

    Este script debe agregarse al mismo elemento GameObject que el script que implementa **BaseRemoteRenderedModel**. En este caso, eso significa **RemoteRenderedModel**. Al igual que en los scripts anteriores, este código inicial controlará todos los cambios de estado, los eventos y los datos relacionados con los límites remotos.

    Hay dos métodos que quedan por implementar: **QueryBounds** y **ProcessQueryResult**. **QueryBounds** captura los límites y **ProcessQueryResult** toma el resultado de la consulta y lo aplica al elemento **BoxCollider** local.

    El método **QueryBounds** es sencillo: se envía una consulta a la sesión de representación remota y se escucha el evento `Completed`.

1. Reemplace el método **QueryBounds** por el siguiente método completado:

    ```csharp
    // Create a query using the model entity
    private void QueryBounds()
    {
        remoteBoundsQuery = targetModel.ModelEntity.QueryLocalBoundsAsync();
        CurrentBoundsState = RemoteBoundsState.Updating;
        remoteBoundsQuery.Completed += ProcessQueryResult;
    }
    ```

    **ProcessQueryResult** también es sencillo. Vamos a comprobar el resultado para ver si es correcto. En caso afirmativo, convierta y aplique los límites devueltos en un formato que pueda aceptar **BoxCollider**.    

1. Reemplace el método **ProcessQueryResult** por el siguiente método completado:

    ```csharp
    // Check the result and apply it to the local Unity bounding box if it was successful
    private void ProcessQueryResult(BoundsQueryAsync remoteBounds)
    {
        if (remoteBounds.IsRanToCompletion)
        {
            var newBounds = remoteBounds.Result.toUnity();
            BoundsBoxCollider.center = newBounds.center;
            BoundsBoxCollider.size = newBounds.size;
            BoundsBoxCollider.enabled = true;
            CurrentBoundsState = RemoteBoundsState.Ready;
        }
        else
        {
            CurrentBoundsState = RemoteBoundsState.Error;
        }
    }
    ```

Ahora, cuando se agregue el script **RemoteBounds** al mismo objeto de juego que **RemoteRenderedModel**, se agregará un elemento **BoxCollider** si es necesario y cuando el modelo llegue al estado `Loaded`, los límites se consultarán y se aplicarán automáticamente a **BoxCollider**.

1. Con el elemento GameObject **TestModel** creado anteriormente, agregue el componente **RemoteBounds**.
1. Confirme que se ha agregado el script.

     ![Adición del componente RemoteBounds](./media/remote-bounds-script.png)

1. Vuelva a ejecutar la aplicación. Poco después de que se cargue el modelo, verá los límites del objeto remoto. Verá algo parecido a los valores siguientes:

     ![Límites actualizados](./media/updated-bounds.png)

Ahora tenemos un elemento **BoxCollider** local configurado con límites precisos en el objeto de Unity. Los límites permiten la visualización y la interacción mediante las mismas estrategias que usamos para un objeto representado localmente. Por ejemplo, los scripts que modifican la transformación, la física, etc.

## <a name="move-rotate-and-scale"></a>Mover, girar y cambiar la escala  

Mover, girar y cambiar la escala de los objetos representados de forma remota funciona igual que con cualquier otro objeto de Unity. **RemoteRenderingCoordinator**, en su método `LateUpdate`, llama a `Update` en la sesión activa. Parte de lo que hace `Update` es sincronizar las transformaciones de la entidad del modelo local con sus homólogos remotos. Para mover, girar o cambiar la escala de un modelo representado de forma remota, solo tiene que mover, girar o escalar la transformación del elemento GameObject que representa el modelo remoto. Aquí, vamos a modificar la transformación del elemento GameObject primario que tiene asociado el script **RemoteRenderedModel**.

En este tutorial se usa MRTK para la interacción de los objetos. La mayor parte de la implementación específica de MRTK para mover, girar y cambiar la escala de un objeto escapa del ámbito de este tutorial. Hay un controlador de vista de modelo que viene preconfigurado dentro de **AppMenu**, en el menú **Model Tools** (Herramientas de modelo).

1. Asegúrese de que el elemento GameObject **TestModel** creado anteriormente se encuentra en la escena.
1. Asegúrese de que el elemento prefabricado **AppMenu** está en la escena.
1. Presione el botón de reproducción de Unity para reproducir la escena y abra el menú **Model Tools** (Herramientas de modelo) dentro de **AppMenu**.
![Controlador de vista](./media/model-with-view-controller.png)

**AppMenu** tiene un submenú **Model Tools** (Herramientas de modelo) que implementa un controlador de vista para enlazar con el modelo. Cuando el elemento GameObject contiene un componente **RemoteBounds**, el controlador de vista agrega un componente [**BoundingBox**](https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/README_BoundingBox.html), que es un componente de MRTK que representa un cuadro de límite alrededor de un objeto con **BoxCollider**. [**ObjectManipulator**](https://microsoft.github.io/MixedRealityToolkit-Unity/version/releases/2.3.0/api/Microsoft.MixedReality.Toolkit.Experimental.UI.ObjectManipulator.html?q=ObjectManipulator), que es responsable de las interacciones con las manos. Estos scripts, combinados, nos permiten mover, girar y cambiar la escala del modelo representado de forma remota.

1. Mueva el mouse al panel de juego y haga clic dentro para que reciba el foco.
1. Mediante la [simulación de manos de MRTK](https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/InputSimulation/InputSimulationService.html#hand-simulation), mantenga presionada la tecla Mayús izquierda.
1. Guíe la mano simulada para que el rayo de la mano se dirija al modelo de prueba.

    ![Rayo de la mano dirigido](./media/handray-engine.png)

1. Mantenga presionado el botón izquierdo y arrastre el modelo para moverlo.

Verá que el contenido representado de forma remota se mueve junto con el cuadro de límite. Es posible que observe algún retraso o retardo entre el cuadro de límite y el contenido remoto. Este retraso dependerá de la latencia y del ancho de banda de Internet.

## <a name="ray-cast-and-spatial-queries-of-remote-models"></a>Proyección de rayos y consultas espaciales de modelos remotos

Un colisionador de cuadros alrededor de los modelos resulta adecuado para interactuar con el modelo completo, pero no es lo suficientemente detallado como para interactuar con sus partes. Para solucionar este problema, se puede usar la [proyección de rayos remota](../../../overview/features/spatial-queries.md#ray-casts). La proyección de rayos remota es una API que proporciona Azure Remote Rendering para proyectar rayos sobre la escena remota y devolver los resultados de acierto de forma local. Esta técnica se puede usar para seleccionar entidades secundarias de un modelo grande o para obtener información de los resultados de acierto, como la posición, la normal de superficie y la distancia.

El modelo de prueba tiene varias subentidades que se pueden consultar y seleccionar. Por ahora, la selección generará el nombre de la entidad seleccionada en la consola de Unity. Compruebe el capítulo [Materiales, iluminación y efectos](../materials-lighting-effects/materials-lighting-effects.md#highlighting-and-outlining) para saber cómo resaltar la entidad seleccionada.

En primer lugar, vamos a crear un contenedor estático en torno a las consultas de proyección de rayos remota. Este script aceptará una posición y una dirección en el espacio de Unity, las convertirá en los tipos de datos aceptados por la proyección de rayos remota y devolverá los resultados. El script hará uso de la API `RayCastQueryAsync`.

1. Cree un script llamado **RemoteRayCaster** y reemplace su contenido por el código siguiente:

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using System.Linq;
    using System.Threading.Tasks;
    using UnityEngine;

    /// <summary>
    /// Wraps the Azure Remote Rendering RayCast queries to easily send requests using Unity data types
    /// </summary>
    public class RemoteRayCaster
    {
        public static double maxDistance = 30.0;

        public static async Task<RayCastHit[]> RemoteRayCast(Vector3 origin, Vector3 dir, HitCollectionPolicy hitPolicy = HitCollectionPolicy.ClosestHit)
        {
            if(RemoteRenderingCoordinator.instance.CurrentCoordinatorState == RemoteRenderingCoordinator.RemoteRenderingState.RuntimeConnected)
            {
                var rayCast = new RayCast(origin.toRemotePos(), dir.toRemoteDir(), maxDistance, hitPolicy);
                return await RemoteRenderingCoordinator.CurrentSession.Actions.RayCastQueryAsync(rayCast).AsTask();
            }
            else
            {
                return new RayCastHit[0];
            }
        }

        public static async Task<Entity[]> RemoteRayCastEntities(Vector3 origin, Vector3 dir, HitCollectionPolicy hitPolicy = HitCollectionPolicy.ClosestHit)
        {
            var hits = await RemoteRayCast(origin, dir, hitPolicy);
            return hits.Select(hit => hit.HitEntity).Where(entity => entity != null).ToArray();
        }
    }
    ```

    > [!NOTE]
    > Unity tiene una clase denominada [**RaycastHit**](https://docs.unity3d.com/ScriptReference/RaycastHit.html), y Azure Remote Rendering tiene una clase denominada [**RayCastHit**](https://docs.microsoft.com/dotnet/api/microsoft.azure.remoterendering.raycasthit). La **C** mayúscula es una diferencia importante para evitar errores de compilación.

    **RemoteRayCaster** proporciona un punto de acceso común para proyectar los rayos remotos en la sesión actual. Para ser más específico, implementaremos a continuación un controlador de puntero de MRTK. El script implementará la interfaz `IMixedRealityPointerHandler`, que indicará a MRTK que queremos que este script escuche los eventos del [puntero de Mixed Reality](https://microsoft.github.io/MixedRealityToolkit-Unity/Documentation/Input/Pointers.html).

1. Cree un script llamado **RemoteRayCastPointerHandler** y reemplace el código por el código siguiente:

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.

    using Microsoft.Azure.RemoteRendering;
    using Microsoft.MixedReality.Toolkit.Input;
    using System;
    using System.Linq;
    using System.Threading.Tasks;
    using UnityEngine;

    public class RemoteRayCastPointerHandler : BaseRemoteRayCastPointerHandler, IMixedRealityPointerHandler
    {
        public UnityRemoteEntityEvent OnRemoteEntityClicked = new UnityRemoteEntityEvent();

        public override event Action<Entity> RemoteEntityClicked;

        public void Awake()
        {
            // Forward events to Unity events
            RemoteEntityClicked += (entity) => OnRemoteEntityClicked?.Invoke(entity);
        }

        public async void OnPointerClicked(MixedRealityPointerEventData eventData)
        {
            if (RemoteEntityClicked != null) //Ensure someone is listening before we do the work
            {
                var firstHit = await PointerDataToRemoteRayCast(eventData.Pointer);
                if (firstHit.success)
                    RemoteEntityClicked.Invoke(firstHit.hit.HitEntity);
            }
        }

        public void OnPointerDown(MixedRealityPointerEventData eventData) { }

        public void OnPointerDragged(MixedRealityPointerEventData eventData) { }

        public void OnPointerUp(MixedRealityPointerEventData eventData) { }

        private async Task<(bool success, RayCastHit hit)> PointerDataToRemoteRayCast(IMixedRealityPointer pointer, HitCollectionPolicy hitPolicy = HitCollectionPolicy.ClosestHit)
        {
            RayCastHit hit;
            var result = pointer.Result;
            if (result != null)
            {
                var endPoint = result.Details.Point;
                var direction = pointer.Rays[pointer.Result.RayStepIndex].Direction;
                Debug.DrawRay(endPoint, direction, Color.green, 0);
                hit = (await RemoteRayCaster.RemoteRayCast(endPoint, direction, hitPolicy)).FirstOrDefault();
            }
            else
            {
                hit = new RayCastHit();
            }
            return (hit.HitEntity != null, hit);
        }
    }
    ```

MRTK llama al método `OnPointerClicked` de **RemoteRayCastPointerHandler** cuando un puntero "hace clic" en un colisionador, como nuestro colisionador de cuadros. Después, se llama a `PointerDataToRemoteRayCast` para convertir el resultado del puntero en un punto y una dirección. A continuación, ese punto y esa dirección se usan para proyectar un rayo remoto en la sesión remota.

![Límites actualizados](./media/raycast-local-remote.png)

El envío de solicitudes para la proyección de rayos al hacer clic es una estrategia eficaz para consultar objetos remotos. Sin embargo, no es una experiencia de usuario idónea porque el cursor colisiona con el colisionador de cuadros y no con el modelo en sí.

También puede crear un puntero de MRTK que proyecte sus rayos en la sesión remota con mayor frecuencia. Aunque se trata de un enfoque más complejo, la experiencia del usuario sería mejor. Esta estrategia escapa del ámbito de este tutorial, pero se puede ver un ejemplo de este enfoque en la aplicación de presentación que se encuentra en el [repositorio de muestras de ARR](https://github.com/Azure/azure-remote-rendering/tree/master/Unity/AzureRemoteRenderingShowcase).

Cuando una proyección de rayos se completa correctamente en **RemoteRayCastPointerHandler**, el elemento `Entity` afectado se emite desde el evento de Unity `OnRemoteEntityClicked`. Para responder a ese evento, crearemos un script auxiliar que acepte el elemento `Entity` y realice una acción en él. Para comenzar, vamos a obtener el script para imprimir el nombre del elemento `Entity` en el registro de depuración.

1. Cree un script llamado **RemoteEntityHelper** y reemplace su contenido por el siguiente:

    ```csharp
    // Copyright (c) Microsoft Corporation. All rights reserved.
    // Licensed under the MIT License. See LICENSE in the project root for license information.
    
    using Microsoft.Azure.RemoteRendering;
    using Microsoft.Azure.RemoteRendering.Unity;
    using UnityEngine;
    
    public class RemoteEntityHelper : MonoBehaviour
    {
        public void EntityToDebugLog(Entity entity)
        {
            Debug.Log(entity.Name);
        }
    }
    ```

2. En el elemento GameObject **TestModel** creado anteriormente, agregue los componentes **RemoteRayCastPointerHandler** y **RemoteEntityHelper**.
1. Asigne el método `EntityToDebugLog` al evento `OnRemoteEntityClicked`. Cuando el tipo de salida del evento y el tipo de entrada del método coinciden, podemos usar el enlace de eventos dinámicos de Unity, que pasará automáticamente el valor del evento al método.
    1. Cree un campo de devolución de llamada, ![Agregar devolución de llamada](./media/add-callback-remote-entity-clicked.png)
    1. Arrastre el componente **RemoteEntityHelper** al campo Object (Objeto) para hacer referencia al objeto primario GameObject ![Asignar objeto](./media/assign-object.png)
    1. Asigne `EntityToDebugLog` como devolución de llamada ![Asignar devolución de llamada](./media/remote-entity-event.png)
1. Presione el botón de reproducción del editor de Unity para iniciar la escena, conéctese a una sesión remota y cargue el modelo de prueba.
1. Con la simulación de manos de MRTK, mantenga presionada la tecla Mayús izquierda.
1. Guíe la mano simulada para que el rayo de la mano se dirija al modelo de prueba.
1. Realice un clic largo para simular una espita de aire, con la ejecución el evento `OnPointerClicked`.
1. Observe que la consola de Unity muestra un mensaje de registro con el nombre de la entidad secundaria seleccionada. Por ejemplo: ![Ejemplo de entidad secundaria](./media/child-entity-example.png)

## <a name="synchronizing-the-remote-object-graph-into-the-unity-hierarchy"></a>Sincronización del gráfico de objetos remotos en la jerarquía de Unity

Hasta el momento, solo hemos visto un elemento GameObject local que representa todo el modelo. Este enfoque funciona bien para representar y manipular el modelo entero. Sin embargo, si queremos aplicar efectos o manipular subentidades específicas, tendremos que crear elementos GameObject locales para representar esas entidades. En primer lugar, podemos explorar manualmente el modelo de prueba.

1. Inicie la escena y cargue el modelo de prueba.
1. Expanda los elementos secundarios del elemento GameObject **TestModel** de la jerarquía de Unity y seleccione el elemento GameObject **TestModel_Entity**.
1. En el Inspector, haga clic en el botón *Mostrar elementos secundarios*.
![Show children (Mostrar elementos secundarios)](./media/show-remote-children.png)
1. Continúe con la expansión de elementos secundarios en la jerarquía y haciendo clic en *Mostrar elementos secundarios* hasta que aparezca una lista grande de elementos secundarios.
![Todos los elementos secundarios](./media/test-model-children.png)

La jerarquía se rellenará ahora con una lista de decenas de entidades. Al seleccionar una de ellas, se mostrarán los componentes `Transform` y `RemoteEntitySyncObject` en el inspector. De forma predeterminada, cada entidad no se sincroniza automáticamente con cada fotograma, por lo que los cambios locales en `Transform` no se sincronizan con el servidor. Si activa *Sync Every Frame* (Sincronizar cada fotograma) y, luego, mueve, gira o cambia la escala de la transformación en la vista de escena, no verá el modelo representado en ella. Inspeccione la vista de juego para ver que la posición y la rotación del modelo se actualizan visualmente.

El mismo proceso se puede realizar mediante programación y es el primer paso para modificar entidades remotas específicas.

1. Modifique el script **RemoteEntityHelper** para que contenga también el método siguiente:

    ```csharp
    public void MakeSyncedGameObject(Entity entity)
    {
        var entityGameObject = entity.GetOrCreateGameObject(UnityCreationMode.DoNotCreateUnityComponents);
        var sync = entityGameObject.GetComponent<RemoteEntitySyncObject>();
        sync.SyncEveryFrame = true;
    }
    ```

1. Agregue una devolución de llamada adicional al evento `OnRemoteEntityClicked` de **RemoteRayCastPointerHandler** y establézcalo en `MakeSyncedGameObject`.
![Devolución de llamada adicional](./media/additional-callback.png)
1. Con la simulación de manos de MRTK, mantenga presionada la tecla Mayús izquierda.
1. Guíe la mano simulada para que el rayo de la mano se dirija al modelo de prueba.
1. Realice un clic largo para simular una espita de aire, con la ejecución el evento `OnPointerClicked`.
1. Compruebe y expanda la jerarquía para ver un nuevo objeto secundario que representa la entidad en la que se ha hecho clic.
![Representación de GameObject](./media/gameobject-representing-entity.png)
1. Después de la prueba, quite la devolución de llamada de `MakeSyncedGameObject`, ya que se incorporará como parte de otros efectos más adelante.

> [!NOTE]
> La sincronización de cada fotograma solo es necesaria cuando se deben sincronizar los datos de transformación. Existe cierta sobrecarga en la sincronización de las transformaciones, por lo que debe usarse con moderación.

Crear una instancia local y sincronizarla automáticamente es el primer paso para manipular subentidades. Las mismas técnicas que hemos usado para manipular el modelo como un todo se pueden usar también en las subentidades. Por ejemplo, después de crear una instancia local sincronizada de una entidad, podría consultar sus límites y agregar controladores de manipulación para que se mueva con los rayos de las manos del usuario.

## <a name="next-steps"></a>Pasos siguientes

Ahora puede manipular los modelos representados de forma remota e interactuar con ellos. En el siguiente tutorial, trataremos la modificación de materiales, la alteración de la iluminación y la aplicación de efectos a los modelos representados de forma remota.

> [!div class="nextstepaction"]
> [Siguiente: Refinamiento de materiales, iluminación y efectos](../materials-lighting-effects/materials-lighting-effects.md)
