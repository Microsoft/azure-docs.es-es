---
title: 'Tutorial: Creación de una nueva aplicación de HoloLens Unity'
description: En este tutorial, aprenderá a crear una aplicación de HoloLens Unity mediante Azure Spatial Anchors.
author: craigktreasure
manager: vriveras
services: azure-spatial-anchors
ms.author: crtreasu
ms.date: 08/17/2020
ms.topic: tutorial
ms.service: azure-spatial-anchors
ms.openlocfilehash: 365fe8c330cadcc01fcd24de28b663cd80b55117
ms.sourcegitcommit: c52e50ea04dfb8d4da0e18735477b80cafccc2cf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/08/2020
ms.locfileid: "89535892"
---
# <a name="tutorial-step-by-step-instructions-to-create-a-new-hololens-unity-app-using-azure-spatial-anchors"></a>Tutorial: instrucciones paso a paso para crear una aplicación de HoloLens Unity mediante Azure Spatial Anchors

En este tutorial se muestra cómo crear una aplicación de HoloLens Unity con Azure Spatial Anchors.

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial, asegúrese de disponer de los siguientes elementos:

1. Una máquina Windows con <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2017 o superior</a> instalado, con la carga de trabajo **Desarrollo de la plataforma universal de Windows** y el componente **Windows 10 SDK (10.0.18362.0 o versiones posteriores)** y <a href="https://git-scm.com/download/win" target="_blank">Git para Windows</a>.
2. La [extensión de Visual Studio para C++/WinRT (VSIX)](https://aka.ms/cppwinrt/vsix) para Visual Studio instalada desde [Visual Studio Marketplace](https://marketplace.visualstudio.com/).
3. Un dispositivo HoloLens con el [modo de desarrollador](https://docs.microsoft.com/windows/mixed-reality/using-visual-studio) habilitado. Este artículo requiere un dispositivo HoloLens con la [actualización de octubre de 2018 de Windows 10](https://docs.microsoft.com/windows/mixed-reality/release-notes-october-2018 ) (también conocida como RS5). Para actualizar a la versión más reciente en HoloLens, abra la aplicación **Settings** (Configuración), vaya a **Update & Security** (Actualización y seguridad) y, a continuación, seleccione el botón **Check for updates** (Buscar actualizaciones).

## <a name="getting-started"></a>Introducción

Lo primero es configurar el proyecto y la escena de Unity:
1. Inicie Unity.
2. Seleccione **Nuevo**.
4. Asegúrese de que la opción **3D** está seleccionada.
5. Asigne un nombre al proyecto y escriba una **ubicación** donde guardarlo.
6. Seleccione **Create project** (Crear proyecto).
7. Guarde la escena predeterminada vacía en un nuevo archivo con: **File** > **Save As** (Archivo > Guardar como).
8. Asigne a la nueva escena el nombre **Principal** y presione el botón **Save** (Guardar).

**Configuración del proyecto**

A continuación, se van a establecer algunos valores de configuración del proyecto de Unity que nos ayuden a trabajar con el SDK de Windows Holographic para el desarrollo.

En primer lugar, veremos la configuración de calidad de nuestra aplicación.
1. Seleccione **Edit** > **Project Settings** > **Quality** (Editar > Configuración del proyecto > Calidad).
2. En la columna que aparece bajo el logotipo de **Windows Store** (Tienda Windows), haga clic en la flecha situada en la fila **Default** (Predeterminado) y seleccione **Very Low** (Muy baja). Sabrá que la configuración se aplica correctamente cuando el cuadro de la columna **Windows Store** (Tienda Windows) y la fila **Very Low** (Muy baja) estén en verde.

Hay que configurar la aplicación de Unity con una vista inmersiva, en lugar de una vista 2D. Para crear una vista inmersiva, se puede habilitar la compatibilidad con Virtual Reality en Unity y apuntar a Windows 10 SDK.
1. Vaya a **Edit** > **Project Settings** > **Player** (Editar > Configuración del proyecto > Reproductor).
2. En **Inspector Panel** (Panel de inspector) de **Player Settings** (Configuración del Reproductor), seleccione el icono de **Windows**.
3. Expanda el grupo **XR Settings** (Configuración de XR).
4. En la sección **Rendering** (Representación), active la casilla **Virtual Reality Supported** (Se admite Virtual Reality) para agregar una nueva lista de **SDK de Virtual Reality**.
5. Compruebe que **Windows Mixed Reality** (Mixed Reality de Windows) aparece en la lista. En caso contrario, seleccione el botón **+** situado en la parte inferior de la lista y elija **Windows Mixed Reality** (Mixed Reality de Windows).

> [!NOTE]
> Si no ve el icono de Windows, verifique que ha seleccionado el back-end de scripting de .NET de Windows. Si no, es posible que deba volver a instalar Unity con la instalación correcta de Windows.

**Verificar la configuración de back-end de script**
1. Vaya a **Edit** > **Project Settings** > **Player** (Editar > Configuración del proyecto > Reproductor) (puede que aún tenga abierto el **Reproductor** del paso anterior).
2. En **Inspector Panel** (Panel de inspector) de **Player Settings** (Configuración del Reproductor), seleccione el icono de **Windows Store** (Tienda Windows).
3. En la sección de configuración **Other Settings** (Otra configuración), asegúrese de que **Scripting Backend** (Back-end de scripting) esté establecido en **IL2CPP**.

**Establecimiento de funcionalidades**
1. Vaya a **Edit** > **Project Settings** > **Player** (Editar > Configuración del proyecto > Reproductor) (puede que aún tenga abierto el **Reproductor** del paso anterior).
2. En **Inspector Panel** (Panel de inspector) de **Player Settings** (Configuración del Reproductor), seleccione el icono de **Windows Store** (Tienda Windows).
3. En la sección de configuración **Publishing Settings** (Configuración de publicación), active **InternetClientServer** y **SpatialPerception**.

**Configuración de la cámara virtual principal**
1. En **Hierarchy Panel** (Panel de jerarquía), seleccione **Main Camera** (Cámara principal).
2. En **Inspector**, establezca su posición de transformación en **0,0,0**.
3. Busque la propiedad **Clear Flags** (Borrar marcas) y cambie la lista desplegable de **Skybox** a **Solid Color** (Color sólido).
4. Haga clic en el campo **Background** (Fondo) para abrir un selector de colores.
5. Establezca **R, G, B, and A** (R, G, B y A) en **0**.
6. Seleccione **Add Component** (Agregar componente) y busque **Spatial Mapping Collider**.

**Creación del script**
1. En el panel **Project** (Proyecto), cree una carpeta, **Scripts**, en la carpeta **Assets** (Activos).
2. Haga clic con el botón derecho en la carpeta y seleccione **Create (Crear) >** , **C# Script** (Script de C#). Asígnele el nombre **AzureSpatialAnchorsScript**.
3. Vaya a **GameObject** -> **Create Empty** (Crear vacío).
4. Selecciónelo y, en **Inspector**, cambie su nombre de **GameObject** a **MixedRealityCloud**. Seleccione **Add Component** (Agregar componente) y agregue **AzureSpatialAnchorsScript**.

**Creación del objeto prefabricado de esfera**
1. Vaya a **GameObject** -> **3D Object** -> **Sphere** (GameObject > Objeto 3D > Esfera).
2. En el **Inspector**, establezca su escala en **0,25, 0,25, 0,25**.
3. Buscar el objeto **Sphere** en el panel de **jerarquía**. Haga clic en él y arrástrelo a la carpeta **Assets** carpeta en el panel **Project** (Proyecto).
4. Haga clic con el botón derecho y elija **Eliminar** la esfera original que ha creado en el panel **Hierarchy** (Jerarquía).

Ahora debería tener una esfera prefabricada en su panel **Project** (Proyecto).

## <a name="trying-it-out"></a>Prueba
Para probar que todo funciona, compile la aplicación en **Unity** e impleméntela desde **Visual Studio**. Para ello, siga el capítulo 6 del curso [**MR Basics 100: Introducción a Unity**](https://docs.microsoft.com/windows/mixed-reality/holograms-100#chapter-6---build-and-deploy-to-device-from-visual-studio). Verá la pantalla de inicio de Unity y, luego, una opción para borrar la pantalla.

## <a name="place-an-object-in-the-real-world"></a>Colocación de un objeto en el mundo real
Vamos a crear y colocar un objeto mediante la aplicación. Abra la solución de Visual Studio que se creó cuando se [implementó nuestra aplicación](#trying-it-out).

En primer lugar, agregue las importaciones siguientes a `Assembly-CSharp (Universal Windows)\Scripts\AzureSpatialAnchorsScript.cs`:

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=19-24)]

Luego, agregue las siguientes variables de miembro a la clase `AzureSpatialAnchorsScript`:

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=26-47,53-57,65-84)]

Antes de continuar, necesitamos establecer la esfera prefabricada que creamos en nuestra variable miembro de spherePrefab. Vuelva a **Unity**.
1. En **Unity**, seleccione el objeto **MixedRealityCloud** en el panel **Hierarchy** (Jerarquía).
2. Haga clic en el objeto prefabricado **Sphere** que guardó en el panel de **proyecto**. Arrastre la **esfera** en la que hizo clic en el área del **objeto prefabricado esfera** bajo **Azure Spatial Anchors Script (Script)** en el panel **Inspector**.

Ahora debería tener el objeto **Sphere** establecido como prefabricado en el script. Compílelos en **Unity** y, luego, abra de nuevo la solución de **Visual Studio** resultante, como ha hecho en [Prueba](#trying-it-out).

En **Visual Studio**, abra el archivo `AzureSpatialAnchorsScript.cs`. Agregue el código siguiente al método `Start()`. Este código enlazará `GestureRecognizer`, que llamará a `HandleTap` cuando detecte una pulsación en el aire.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=86-95,98&highlight=4-10)]

Ahora es necesario agregar el siguiente método `HandleTap()` debajo de `Update()`. Realizará un lanzamiento de rayos (ray casting) y obtendrá un punto de posicionamiento en el que colocar una esfera.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=273-283,305-306,310-318)]

Ahora debemos crear la esfera. La esfera inicialmente será blanca, pero este valor se ajustará más adelante. Agregue el siguiente método `CreateAndSaveSphere()`:

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=320-331,396)]

Ejecute la aplicación desde **Visual Studio** para validarla una vez más. Luego, toque la pantalla para crear la esfera blanca y colocarla sobre la superficie que haya elegido.

## <a name="set-up-the-dispatcher-pattern"></a>Configuración del patrón de distribuidor

Cuando se trabaja con Unity, todas las API de Unity (por ejemplo, las API que se usan para realizar actualizaciones de la interfaz de usuario) deben tener lugar en el subproceso principal. Sin embargo, en el código que vamos a escribir, obtendremos devoluciones de llamada en otros subprocesos. Queremos actualizar la interfaz de usuario en estas devoluciones de llamada, por lo que necesitamos una manera de pasar de un subproceso secundario al subproceso principal. Para ejecutar código en el subproceso principal desde un subproceso secundario, vamos a usar el patrón de distribuidor.

Utilizaremos la variable de miembro `dispatchQueue`, que consiste una cola de acciones. Insertaremos acciones en la cola y, luego, las quitaremos de la cola y ejecutaremos las acciones en el subproceso principal.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=43-56&highlight=6-9)]

Luego, agregaremos una manera de incorporar una acción a la cola. Agregaremos `QueueOnUpdate()` justo después de `Update()`:

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=112-122)]

Podemos usar el bucle Update() para comprobar si hay una acción en cola. En caso afirmativo, la quitaremos de la cola y la ejecutaremos.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=100-110&highlight=4-10)]

## <a name="get-the-azure-spatial-anchors-sdk"></a>Obtención del SDK de Azure Spatial Anchors

## <a name="via-unity-package-manager-upm-package"></a>[Mediante el paquete del administrador de paquetes de Unity (UPM)](#tab/UPMPackage)

Este método es compatible con las versiones de Unity 2019.1 y posteriores.

### <a name="add-the-registry-to-your-unity-project"></a>Incorporación del registro al proyecto de Unity

1. Desde un explorador de archivos, acceda a la carpeta `Packages` del proyecto de Unity. Abra el archivo de manifiesto del proyecto `manifest.json` en un editor de texto.
2. En la parte superior del archivo, en el nivel de la sección `dependencies`, agregue la entrada siguiente para incluir el registro de Azure Spatial Anchors al proyecto. La entrada `scopedRegistries` indica a Unity dónde debe buscar los paquetes del SDK de Azure Spatial Anchors.

    [!code-json[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-unity-scoped-registry-setup.md?range=9-19&highlight=2-10)]

### <a name="add-the-sdk-package-to-your-unity-project"></a>Incorporación del paquete de SDK al proyecto de Unity

1. Agregue una entrada con el nombre del paquete de SDK de Azure Spatial Anchors (`com.microsoft.azure.spatial-anchors-sdk.windows`) y la versión del paquete en la sección `dependencies` del manifiesto del proyecto. Consulte un ejemplo a continuación.

    [!code-json[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-unity-scoped-registry-setup.md?range=9-20&highlight=12)]

2. Guarde y cierre el archivo `manifest.json`. Cuando vuelva a Unity, la plataforma debería detectar automáticamente el cambio de manifiesto del proyecto y recuperar los paquetes especificados. Puede expandir la carpeta `Packages` en la vista de proyecto para comprobar que se hayan importado los paquetes correctos.

## <a name="via-unity-asset-package"></a>[Mediante el paquete de recursos de Unity](#tab/UnityAssetPackage)

> [!WARNING]
> La distribución del paquete de activos de Unity del SDK de Azure Spatial Anchors dejará de utilizarse después de la versión 2.5.0 del SDK.

Ahora, descargaremos el SDK de Azure Spatial Anchors. Vaya a la [página de versiones de GitHub de Azure Spatial Anchors](https://github.com/Azure/azure-spatial-anchors-samples/releases). En **Assets** (Recursos), descargue **AzureSpatialAnchors.unitypackage**. En Unity, vaya a **Assets** (Recursos), seleccione en **Import Package** > **Custom Package...** (Importar paquete > Paquete personalizado...). Vaya al paquete y seleccione **Open** (Abrir).

En la nueva ventana **Import Unity Package** (Importar paquete de Unity) que aparece, anule la selección de **Plugins** (Complementos) y seleccione **Import** (Importar) en la esquina inferior derecha.

---

En la solución de **Visual Studio**, agregue la siguiente importación a `<ProjectName>\Assets\Scripts\AzureSpatialAnchorsScript.cs`:

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=18-21&highlight=1)]

Luego, agregue las siguientes variables de miembro a la clase `AzureSpatialAnchorsScript`:

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=53-68&highlight=6-11)]

## <a name="attach-a-local-azure-spatial-anchor-to-the-local-anchor"></a>Asociación de una instancia local de Azure Spatial Anchors con un anclaje local

Vamos a configurar CloudSpatialAnchorSession de Azure Spatial Anchors. Para comenzar, agregaremos el siguiente método `InitializeSession()` dentro de su clase `AzureSpatialAnchorsScript`. Una vez que se le llama, garantiza que se crea una sesión de Azure Spatial Anchors y que se inicializa correctamente durante el inicio de la aplicación.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=179-208,211-215)]

Ahora es necesario escribir código para controlar las llamadas de delegado. Les agregaremos más código mientras continuamos.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=217-232)]

Ahora, vamos a enlazar el método `initializeSession()` a su método `Start()`.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=86-98&highlight=12)]

Por último, agregue el código siguiente al método `CreateAndSaveSphere()`. Se asociará una instancia local de Azure Spatial Anchors a la esfera que vamos a colocar en el mundo real.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=320-344,396&highlight=14-25)]

Antes de continuar, tiene que crear una cuenta de Azure Spatial Anchors para obtener el identificador, la clave y el dominio de la cuenta. Si aún no dispone de esos valores, vaya a la sección siguiente para obtenerlos.

[!INCLUDE [Create Spatial Anchors resource](../../../includes/spatial-anchors-get-started-create-resource.md)]

## <a name="upload-your-local-anchor-into-the-cloud"></a>Carga del anclaje local en la nube

Una vez que tenga el identificador, la clave y el dominio de la cuenta de Azure Spatial Anchors, vaya y pegue el valor de `Account Id` en `SpatialAnchorsAccountId`, el valor de `Account Key` en `SpatialAnchorsAccountKey` y el valor de `Account Domain` en `SpatialAnchorsAccountDomain`.

Por último, vamos a enlazar todos los elementos. En el método `SpawnNewAnchoredObject()`, agregue la siguiente línea. Esta línea invocará el método `CreateAnchorAsync()` en cuanto se cree la esfera. Una vez que el método devuelva resultados, el código siguiente actualizará la esfera una última vez y el color cambiará a azul.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=320-397&highlight=26-77)]

Ejecute la aplicación desde **Visual Studio** una vez más. Mueva la cabeza y, luego, pulse en el aire para colocar la esfera. Una vez que tengamos suficientes fotogramas, la esfera se pondrá amarilla y se iniciará la carga en la nube. Cuando finalice la carga, la esfera pasará a ser azul. También podría usar la ventana de salida dentro de **Visual Studio** para supervisar los mensajes de registro que envía la aplicación. Puede observar `RecommendedForCreateProgress` y, una vez completada la carga, verá el identificador de delimitador que ha devuelto la nube.

> [!NOTE]
> Si aparece la excepción "DllNotFoundException: No se puede cargar el archivo DLL "AzureSpatialAnchors": No se ha podido encontrar el módulo especificado", debe seleccionar **Clean** (Limpiar) y **Build** (Compilar) para limpiar y volver a compilar la solución.

## <a name="locate-your-cloud-spatial-anchor"></a>Búsqueda del anclaje espacial de la nube

Una vez que el anclaje se carga en la nube, ya es posible volver a intentar localizarlo. Vamos a agregar el siguiente código al método `HandleTap()`. Este código hará lo siguiente:

* Llame a `ResetSession()`, que detendrá `CloudSpatialAnchorSession` y quitará la esfera azul existente de la pantalla.
* Inicialice de nuevo `CloudSpatialAnchorSession`. Esta acción garantiza que el anclaje que vamos a buscar proviene de la nube, y no es el anclaje local que hemos creado.
* Cree un elemento **Watcher** (Monitor) que buscará el anclaje que hemos cargado en Azure Spatial Anchors.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=273-311&highlight=13-31,35-36)]

Ahora agreguemos nuestros métodos `ResetSession()` y `CleanupObjects()`. Puede colocarlos debajo de `QueueOnUpdate()`.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=124-177)]

A continuación, debemos enlazar el código que se invocará cuando se encuentre el anclaje que estamos consultando. En `InitializeSession()`, agregue las siguientes devoluciones de llamada:

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=206-212&highlight=4-5)]


Ahora vamos a agregar código con el que se creará y colocará una esfera verde una vez que se encuentre CloudSpatialAnchor. También volverá a habilitar la opción de tocar la pantalla, con el fin de que pueda repetir el escenario completo una vez más: crear otro anclaje local, cargarlo y volver a buscarlo.

[!code-csharp[AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md?range=234-271)]

Eso es todo. Ejecute la aplicación desde **Visual Studio** una última vez para probar el escenario completo de extremo a extremo. Mueva el dispositivo y coloque la esfera negra. Luego, siga moviendo la cabeza para capturar los datos del entorno hasta que la esfera se vuelva amarilla. Se cargará el anclaje local y la esfera volverá a ser azul. Por último, toque la pantalla una vez más para quitar el delimitador local e iniciar una consulta para su equivalente en la nube. Siga desplazando el dispositivo hasta que se encuentre el anclaje espacial en la nube. Debe mostrarse una esfera verde en la ubicación correcta, y todo el escenario se puede volver a repetir.

[!INCLUDE [AzureSpatialAnchorsScript](../../../includes/spatial-anchors-new-unity-hololens-app-finished.md)]