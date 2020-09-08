---
title: 'Creación de un espacio de desarrollo de Kubernetes: Visual Studio y .NET Core'
services: azure-dev-spaces
ms.custom: vs-azure, devx-track-azurecli, devx-track-csharp
ms.workload: azure-vs
ms.date: 07/09/2018
ms.topic: tutorial
description: En este tutorial se muestra cómo usar Azure Dev Spaces y Visual Studio para depurar e iterar rápidamente una aplicación .NET Core en Azure Kubernetes Service
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores, Helm, service mesh, enrutamiento de service mesh, kubectl, k8s
ms.openlocfilehash: 78ad3c643b839d2a0901ac2c1d930d73b718cd8d
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88999264"
---
# <a name="create-a-kubernetes-dev-space-visual-studio-and-net-core-with-azure-dev-spaces"></a>Creación de un espacio de desarrollo de Kubernetes: Visual Studio y .NET Core con Azure Dev Spaces

En esta guía, aprenderá a:

- Configurar Azure Dev Spaces con un clúster de Kubernetes administrado en Azure.
- Desarrollar código de forma iterativa en contenedores con Visual Studio
- Desarrollar de forma independiente dos servicios distintos y usar la detección de servicios DNS de Kubernetes para realizar una llamada a otro servicio.
- Desarrollar y probar de forma productiva el código en un entorno de equipo.

> [!NOTE]
> **Si se queda bloqueado** en cualquier momento, consulte la sección [Solución de problemas](troubleshooting.md).

## <a name="install-the-azure-cli"></a>Instalación de la CLI de Azure
Azure Dev Spaces requiere una configuración mínima de la máquina local. La mayor parte de la configuración del espacio de desarrollo se almacena en la nube y se puede compartir con otros usuarios. Para comenzar, descargue y ejecute la [CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

### <a name="sign-in-to-azure-cli"></a>Inicio de sesión en la CLI de Azure
Inicie sesión en Azure. Escriba el siguiente comando en una ventana de terminal:

```azurecli
az login
```

> [!NOTE]
> Si no tiene una suscripción a Azure, puede crear una [cuenta gratuita](https://azure.microsoft.com/free).

#### <a name="if-you-have-multiple-azure-subscriptions"></a>Si tiene varias suscripciones a Azure...
Para ver sus suscripciones, ejecute: 

```azurecli
az account list --output table
```

Busque la suscripción que tiene *True* para *IsDefault*.
Si esta no es la suscripción que desea usar, puede cambiar la suscripción predeterminada:

```azurecli
az account set --subscription <subscription ID>
```

## <a name="create-a-kubernetes-cluster-enabled-for-azure-dev-spaces"></a>Creación de un clúster de Kubernetes habilitado para Azure Dev Spaces

En el símbolo del sistema, cree el grupo de recursos en una [región que admita Azure Dev Spaces][supported-regions].

```azurecli
az group create --name MyResourceGroup --location <region>
```

Crear un clúster de Kubernetes con el siguiente comando:

```azurecli
az aks create -g MyResourceGroup -n MyAKS --location <region> --generate-ssh-keys
```

La operación de creación del clúster tarda unos minutos.

### <a name="configure-your-aks-cluster-to-use-azure-dev-spaces"></a>Configuración del clúster de AKS para usar Azure Dev Spaces

Escriba el siguiente comando de la CLI de Azure, mediante el grupo de recursos que contiene el clúster de AKS, y su nombre del clúster de AKS. El comando configura el clúster para la compatibilidad con Azure Dev Spaces.

   ```azurecli
   az aks use-dev-spaces -g MyResourceGroup -n MyAKS
   ```
   
> [!IMPORTANT]
> El proceso de configuración de Azure Dev Spaces quitará el espacio de nombres `azds` del clúster, si existe.

## <a name="get-the-visual-studio-tools"></a>Obtención de las herramientas de Visual Studio
Instale la versión más reciente de [Visual Studio 2019](https://www.visualstudio.com/vs/) en Windows con la carga de trabajo Desarrollo de Azure.

## <a name="create-a-web-app-running-in-a-container"></a>Crear una aplicación web que se ejecuta en un contenedor

En esta sección, va a crear una aplicación web de ASP.NET Core y a ejecutarla en un contenedor de Kubernetes.

### <a name="create-an-aspnet-web-app"></a>Creación de una aplicación web de ASP.NET

En Visual Studio, cree un nuevo proyecto. Actualmente, el proyecto debe ser una **aplicación web de ASP.NET Core**. Llame al proyecto "**webfrontend**".

![El cuadro de diálogo "Nuevo proyecto" muestra la creación de una aplicación web de C Sharp denominada "webfrontend" en la ubicación C:\Source\Repos. La lista desplegable "Solución" muestra "Crear nueva solución" y la casilla "Crear directorio para la solución" está activada.](media/get-started-netcore-visualstudio/NewProjectDialog1.png)

Seleccione la plantilla **Aplicación web (controlador de vista de modelos)** y asegúrese de que apunta a **.NET Core** y **ASP.NET Core 2.0** en los dos menús desplegables en la parte superior del cuadro de diálogo. Haga clic en **Aceptar** para crear el proyecto.

![En el cuadro de diálogo "Nueva aplicación web de ASP.NET Core", dos cuadros de lista desplegable muestran ".NET Core" y "ASP.NET Core 2.0". En una matriz de botones de plantilla de proyecto situada debajo de los cuadros de lista, está seleccionada la plantilla "Aplicación web (Modelo-Vista-Controlador)". La casilla "Habilitar compatibilidad con Docker" no está activada.](media/get-started-netcore-visualstudio/NewProjectDialog2.png)

### <a name="enable-dev-spaces-for-an-aks-cluster"></a>Habilitación de Dev Spaces para un clúster de AKS

Con el proyecto que acaba de crear, seleccione **Azure Dev Spaces**  en el menú desplegable de configuración de inicio, como se muestra a continuación.

![El cuadro de lista desplegable está en la parte superior de una ventana con la etiqueta Microsoft Visual Studio Int Preview. La opción "Azure Dev Spaces" está seleccionada.](media/get-started-netcore-visualstudio/LaunchSettings.png)

En el cuadro de diálogo que se muestra a continuación, asegúrese de que ha iniciado sesión con la cuenta adecuada y, a continuación, seleccione un clúster de Kubernetes existente.

![El cuadro de diálogo Azure Dev Spaces tiene estas casillas: "Suscripción", "Clúster de Azure Kubernetes Service" y "Espacio".](media/get-started-netcore-visualstudio/Azure-Dev-Spaces-Dialog.PNG)

Por ahora, deje el menú desplegable **Espacio** con un valor predeterminado de `default`. Más adelante, aprenderá más acerca de esta opción. Active la casilla **Publicly Accessible** (Acceso público) para que la aplicación web esté accesible mediante un punto de conexión público. Esta configuración no es necesaria, pero será útil mostrar algunos conceptos más adelante en este tutorial. Pero no se preocupe, en cualquier caso podrá depurar el sitio web con Visual Studio.

![La casilla Accesible públicamente está activada.](media/get-started-netcore-visualstudio/Azure-Dev-Spaces-Dialog2.png)

Haga clic en **Aceptar** para seleccionar o crear el clúster.

Si elige un clúster que no ha sido habilitado para trabajar con Azure Dev Spaces, verá un mensaje que le pregunta si desea configurarlo.

![El mensaje que aparece es: "Add Azure Dev Spaces Resource? The selected AKS cluster must be configured to use Azure Dev Spaces before it can be used. Would you like to do that?" (¿Agregar recurso de Azure Dev Spaces? El clúster de AKS seleccionado tiene que estar configurado para usar Azure Dev Spaces antes de poder usarlo. ¿Quiere hacerlo ahora?). Hay botones "Aceptar" y "Cancelar".](media/get-started-netcore-visualstudio/Add-Azure-Dev-Spaces-Resource.png)

Elija **Aceptar**.

> [!IMPORTANT]
> El proceso de configuración de Azure Dev Spaces quitará el espacio de nombres `azds` del clúster, si existe.

 Se inicia una tarea en segundo plano para realizar esto. Esta operación tarda unos minutos en completarse. Para ver si se ha creado, sitúe el puntero sobre el icono **Tareas en segundo plano**  en la esquina inferior izquierda de la barra de estado, como se muestra en la siguiente imagen.

![La ventana emergente que aparece al pasar el cursor muestra "Creando "My AKS" ' en el grupo de recursos".](media/get-started-netcore-visualstudio/BackgroundTasks.PNG)

> [!NOTE]
> Hasta que el espacio de desarrollo no se haya creado correctamente, no se puede depurar la aplicación.

### <a name="look-at-the-files-added-to-project"></a>Examen de los archivos agregados al proyecto
Mientras espera a que se cree el espacio de desarrollo, mire los archivos que se han agregado al proyecto cuando eligió usar un espacio de desarrollo.

Primero, puede ver que se ha agregado una carpeta con el nombre `charts` y, dentro de ella, un [gráfico Helm](https://docs.helm.sh) para la aplicación a la que se ha aplicado la técnica scaffolding. Estos archivos se usan para implementar la aplicación en el espacio de desarrollo.

Verá que se ha agregado un archivo denominado `Dockerfile`. Este archivo contiene la información necesaria para empaquetar la aplicación en el formato estándar de Docker.

Por último, verá un archivo llamado `azds.yaml`, que contiene la configuración de tiempo de desarrollo necesaria para el espacio de desarrollo.

![Aparece el archivo "azds.yaml" en la solución "webfrontend", en la ventana Explorador de soluciones.](media/get-started-netcore-visualstudio/ProjectFiles.png)

## <a name="debug-a-container-in-kubernetes"></a>Depuración de un contenedor en Kubernetes
Cuando el espacio de desarrollo se haya creado correctamente, puede depurar la aplicación. Establezca un punto de interrupción en el código, por ejemplo, en la línea 20 del archivo `HomeController.cs`, donde se establece la variable `Message`. Pulse **F5** para iniciar la depuración. 

Visual Studio se comunicará con el espacio de desarrollo para compilar e implementar la aplicación y, después, abrirá un explorador con la aplicación web en ejecución. Puede parecer que el contenedor se ejecuta localmente, pero en realidad lo hace en el espacio de desarrollo de Azure. La razón de la dirección de localhost es porque Azure Dev Spaces crea un túnel SSH temporal al contenedor que se ejecuta en AKS.

Haga clic en el vínculo **Acerca de** en la parte superior de la página para desencadenar el punto de interrupción. Tiene acceso completo a la información de depuración, tal como lo haría si el código se ejecutara localmente, como la pila de llamadas, las variables locales o la información de excepción, por ejemplo.

## <a name="iteratively-develop-code"></a>Desarrollo de código de forma iterativa

Con Azure Dev Spaces no se trata solo de conseguir que el código se ejecute en Kubernetes, sino de permitirle ver rápida e iterativamente que los cambios de código surtan efecto en un entorno de Kubernetes en la nube.

### <a name="update-a-content-file"></a>Actualización de un archivo de contenido


1. Busque el archivo `./Views/Home/Index.cshtml` y realice una edición en el código HTML. Por ejemplo, cambie la [línea 73, que dice `<h2>Application uses</h2>`](https://github.com/Azure/dev-spaces/blob/master/samples/dotnetcore/getting-started/webfrontend/Views/Home/Index.cshtml#L73), a algo como: 
  
    ```html
    <h2>Hello k8s in Azure!</h2>`
    ```

2. Guarde el archivo.
3. Regrese al explorador y actualice la página. Debería ver la página web que muestra el código HTML actualizado.

¿Qué ha ocurrido? Las ediciones de archivos de contenido, como HTML y CSS, no requieren compilarlos de nuevo en una aplicación web de .NET Core, por lo que un sesión de F5 activa sincroniza automáticamente cualquier archivo de contenido modificado en el contenedor que se está ejecutando en AKS, para que pueda ver sus ediciones de contenido de inmediato.

### <a name="update-a-code-file"></a>Actualización de un archivo de código
Actualizar archivos de código requiere un poco más de trabajo, porque una aplicación .NET Core necesita volver a crear y producir archivos binarios de aplicaciones actualizados.

1. Detenga el depurador en Visual Studio.
1. Abra el archivo de código denominado `Controllers/HomeController.cs` y edite el mensaje que se mostrará en la página Acerca de: `ViewData["Message"] = "Your application description page.";`
1. Guarde el archivo.
1. Pulse **F5** para iniciar la depuración de nuevo. 

En lugar de volver a crear e implementar una nueva imagen de contenedor cada vez que se realizan modificaciones en el código, lo que a menudo lleva un tiempo considerable, Azure Dev Spaces volverá a compilar el código de manera incremental dentro del contenedor existente para proporcionar un bucle de modificación/depuración más rápido.

Actualice la aplicación web en el explorador y vaya a la página Acerca de. Verá que aparece el mensaje personalizado en la interfaz de usuario.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre cómo funciona Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Funcionamiento de Azure Dev Spaces](how-dev-spaces-works.md)


[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
