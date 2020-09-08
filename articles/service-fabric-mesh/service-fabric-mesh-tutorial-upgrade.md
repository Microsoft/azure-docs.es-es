---
title: 'Tutorial: Actualización de una aplicación de Azure Service Fabric Mesh'
description: Este tutorial es la cuarta parte de una serie y muestra cómo actualizar una aplicación de Azure Service Fabric Mesh directamente desde Visual Studio.
author: dkkapur
ms.topic: conceptual
ms.date: 11/29/2018
ms.author: dekapur
ms.custom: mvc, devcenter, devx-track-csharp
ms.openlocfilehash: 876359b45daa280a1f5701f2490ce505327f309b
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89011011"
---
# <a name="tutorial-learn-how-to-upgrade-a-service-fabric-application-using-visual-studio"></a>Tutorial: Actualización de una aplicación de Service Fabric con Visual Studio

Este tutorial es la cuarta parte de una serie y muestra cómo actualizar una aplicación de Azure Service Fabric Mesh directamente desde Visual Studio. La actualización incluirá tanto una actualización de código como una actualización de configuración. Verá que los pasos para actualizar y publicar desde Visual Studio son los mismos.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> * Actualización de un servicio de Service Fabric Mesh mediante Visual Studio

En esta serie de tutoriales, se aprende a:
> [!div class="checklist"]
> * [Crear una aplicación de Service Fabric Mesh en Visual Studio](service-fabric-mesh-tutorial-create-dotnetcore.md)
> * [Depurar una aplicación de Service Fabric Mesh que se ejecuta en el clúster de desarrollo local](service-fabric-mesh-tutorial-debug-service-fabric-mesh-app.md)
> * [Implementar una aplicación de Service Fabric Mesh](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md)
> * Actualizar una aplicación de Service Fabric Mesh
> * [Limpiar los recursos de Service Fabric Mesh](service-fabric-mesh-tutorial-cleanup-resources.md)

[!INCLUDE [preview note](./includes/include-preview-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Antes de empezar este tutorial:

* Si no ha implementado la aplicación de tareas pendientes, siga las instrucciones de [Publicación de una aplicación web de Service Fabric Mesh](service-fabric-mesh-tutorial-deploy-service-fabric-mesh-app.md).

## <a name="upgrade-a-service-fabric-mesh-service-by-using-visual-studio"></a>Actualización de un servicio de Service Fabric Mesh mediante Visual Studio

En este artículo se muestra cómo actualizar un microservicio dentro de una aplicación. En este ejemplo, modificaremos el servicio `WebFrontEnd` para mostrar una categoría de tareas y aumentar la cantidad de CPU que se le asigna. Después, se actualizará el servicio implementado.

## <a name="modify-the-config"></a>Modificación de la configuración

Cuando se crea una aplicación de Service Fabric Mesh, Visual Studio agrega un archivo **parameters.yaml** para cada entorno de implementación (en la nube y local). En estos archivos, puede definir parámetros y sus valores, a los que luego se harán referencia desde los archivos *.yaml de Mesh como service.yaml o network.yaml.  Visual Studio proporciona algunas de las variables, por ejemplo, la cantidad de CPU que puede usar el servicio.

Actualizaremos el parámetro `WebFrontEnd_cpu` para actualizar los recursos de CPU para `1.5` antes de que el servicio **WebFrontEnd** se use con más asiduidad.

1. En el proyecto **todolistapp**, en **Entornos** > **Nube**, abra el archivo **parameters.yaml**. Modifique el valor de `WebFrontEnd_cpu` a `1.5`. El nombre del parámetro está precedido por el nombre del servicio `WebFrontEnd_` como procedimiento recomendado para distinguirlo de los parámetros del mismo nombre que se aplican a distintos servicios.

    ```xml
    WebFrontEnd_cpu: 1.5
    ```

2. Abra el archivo **service.yaml** del proyecto **WebFrontEnd** en **WebFrontEnd** > **Service Resources** (Recursos de servicios).

    Tenga en cuenta que en la sección `resources:`, `cpu:` está establecido en `"[parameters('WebFrontEnd_cpu')]"`. Si el proyecto se va a compilar para la nube, el valor de `'WebFrontEnd_cpu` se tomará de **Entornos** > **Nube** >  archivo **parameters.yaml**, y será `1.5`. Si el proyecto se compila para ejecutarse localmente, se tendrá el valor de la **entornos** > **Local** > **parameters.yaml** archivo, y será "0,5".

> [!Tip]
> De forma predeterminada, el archivo de parámetros que es del mismo nivel que el archivo profile.yaml se usará para proporcionar los valores para ese archivo profile.yaml.
> Por ejemplo, en Entornos > Nube > archivo parameters.yaml se proporcionan los valores de Entornos > Nube > profile.yaml.
>
> Puede invalidar esta situación agregando lo siguiente al archivo profile.yaml:`parametersFilePath=”relative or full path to the parameters file”`. Por ejemplo, `parametersFilePath=”C:\MeshParms\CustomParameters.yaml”` o `parametersFilePath=”..\CommonParameters.yaml”`.

## <a name="modify-the-model"></a>Modificación del modelo

Para introducir un cambio de código, agregue una propiedad `Category` a la clase `ToDoItem` en el archivo `ToDoItem.cs`.

```csharp
public class ToDoItem
{
    public string Category { get; set; }
    ...
}
```

A continuación, actualice el método `Load()`, en el mismo archivo, para establecer la categoría en una cadena predeterminada:

```csharp
public static ToDoItem Load(string description, int index, bool completed)
{
    ToDoItem newItem = new ToDoItem(description)
    {
        Index = index,
        Completed = completed, 
        Category = "none"    // <-- add this line
    };

    return newItem;
}
```

## <a name="modify-the-service"></a>Modificación del servicio

El proyecto `WebFrontEnd` es una aplicación de ASP.NET Core con una página web en la que se muestran elementos de la lista de tareas pendientes. En el proyecto `WebFrontEnd`, abra `Index.cshtml` y agregue las dos líneas siguientes, indicadas a continuación, para mostrar la categoría de la tarea:

```HTML
<div>
    <table class="table-bordered">
        <thead>
            <tr>
                <th>Description</th>
                <th>Category</th>           @*add this line*@
                <th>Done?</th>
            </tr>
        </thead>
        <tbody>
            @foreach (var item in Model.Items)
            {
                <tr>
                    <td>@item.Description</td>
                    <td>@item.Category</td>           @*add this line*@
                    <td>@item.Completed</td>
                </tr>
            }
        </tbody>
    </table>
</div>
```

Cree y ejecute la aplicación para comprobar que ve una nueva columna de categoría en la página web que enumera las tareas.

## <a name="upgrade-the-app-from-visual-studio"></a>Actualización de la aplicación desde Visual Studio

Tanto si hace una actualización de código como una actualización de configuración (en este caso vamos a hacer ambas cosas), para actualizar la aplicación Service Fabric Mesh en Azure haga clic con el botón derecho en **todolistapp** en Visual Studio y seleccione **Publicar...**

A continuación, verá un cuadro de diálogo **Publicación de la aplicación de Service Fabric**.

Use la lista desplegable **Perfil objetivo** para seleccionar el archivo profile.yaml que se usará para esta implementación. Como vamos a actualizar la aplicación en la nube, seleccionamos **cloud.yaml** en la lista desplegable, que usará el valor `WebFrontEnd_cpu` de 1.0 definido en ese archivo.

![Cuadro de diálogo de publicación de Service Fabric mesh en Visual Studio](./media/service-fabric-mesh-tutorial-deploy-dotnetcore/visual-studio-publish-dialog.png)

Seleccione la cuenta y la suscripción de Azure. Establezca **Ubicación** en la ubicación que usó cuando se publicó originalmente la aplicación de tareas pendientes en Azure. Este artículo utiliza **Este de EE. UU**.

Establezca **Grupo de recursos** en el grupo de recursos que usó cuando se publicó originalmente la aplicación de tareas pendientes en Azure.

Establezca **Azure Container Registry** en el nombre del registro de contenedores de Azure que creó cuando se publicó originalmente la aplicación de tareas pendientes en Azure.

En el cuadro de diálogo Publicar, presione el botón **Publicar** para actualizar la aplicación de tareas pendientes en Azure.

Para supervisar el progreso de la actualización, seleccione el panel **Herramientas de Service Fabric** en la ventana **Salida** de Visual Studio. 

Después de que la imagen compila y se inserta en Azure Container Registry, aparece un vínculo **Para el estado** en la salida en el que puede hacer clic para supervisar la implementación en Azure Portal.

Una vez finalizada la actualización, la salida de **Herramientas de Service Fabric** mostrará la dirección IP y el puerto de la aplicación en forma de dirección URL.

```json
The application was deployed successfully and it can be accessed at http://10.000.38.000:20000.
```

Abra un explorador web y vaya a la dirección URL para ver el sitio web que se ejecuta en Azure. Ahora debería ver una página web que contiene una columna de categoría.

## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha aprendido a:
> [!div class="checklist"]
> * Cómo actualizar una aplicación de Service Fabric Mesh mediante Visual Studio

Avance hasta el siguiente tutorial:
> [!div class="nextstepaction"]
> [Limpiar los recursos de Service Fabric Mesh](service-fabric-mesh-tutorial-cleanup-resources.md)