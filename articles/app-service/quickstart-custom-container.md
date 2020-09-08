---
title: 'Inicio rápido: Ejecución de un contenedor personalizado en App Service'
description: 'Para comenzar a usar contenedores en Azure App Service: implemente su primer contenedor personalizado.'
author: msangapu-msft
ms.author: msangapu
ms.date: 08/28/2019
ms.topic: quickstart
ms.custom: devx-track-csharp
zone_pivot_groups: app-service-containers-windows-linux
ms.openlocfilehash: 33eaf6274f2da09ab98a21e6028b0103df817744
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961370"
---
# <a name="run-a-custom-container-in-azure"></a>Ejecución de un contenedor personalizado en Azure

::: zone pivot="container-windows"
[Azure App Service](overview.md) proporciona las pilas de aplicaciones predefinidas en Windows, como ASP.NET o Node.js, que se ejecutan en IIS. El entorno preconfigurado del contenedor Windows (versión preliminar) impide el acceso administrativo, las instalaciones de software administrativas, los cambios en la caché global de ensamblados, etc. en el sistema operativo. Para más información, consulte [Funcionalidad del sistema operativo en Azure App Service](operating-system-functionality.md). Si la aplicación requiere más acceso que el que permite el entorno preconfigurado, puede implementar un contenedor de Windows personalizado en su lugar.

En este inicio rápido se muestra cómo implementar una aplicación ASP.NET, que está en una imagen de Windows, en [Docker Hub](https://hub.docker.com/) desde Visual Studio. La aplicación se ejecuta en un contenedor personalizado en Azure App Service.

> [!NOTE]
> App Service en los contenedores Windows se encuentra en la versión preliminar.
>

## <a name="prerequisites"></a>Requisitos previos

Para completar este tutorial:

- <a href="https://hub.docker.com/" target="_blank">Registrarse para obtener una cuenta de Docker Hub</a>
- Instalar <a href="https://docs.docker.com/docker-for-windows/install/" target="_blank">Docker para Windows</a>.
- <a href="https://docs.microsoft.com/virtualization/windowscontainers/quick-start/quick-start-windows-10" target="_blank">Cambiar Docker para ejecutar contenedores de Windows</a>.
- <a href="https://www.visualstudio.com/downloads/" target="_blank">Instalar Visual Studio 2019</a> con las cargas de trabajo **ASP.NET y desarrollo web** y **desarrollo de Azure**. Si ya ha instalado Visual Studio 2019:

    - Para instalar las actualizaciones más recientes de Visual Studio, seleccione **Ayuda** > **Buscar actualizaciones**.
    - Para agregar las cargas de trabajo en Visual Studio, seleccione **Herramientas** > **Obtener herramientas y características**.

## <a name="create-an-aspnet-web-app"></a>Creación de una aplicación web de ASP.NET

Cree una aplicación web ASP.NET siguiendo estos pasos:

1. Abra Visual Studio y seleccione **Crear un proyecto**.

1. En **Crear un proyecto**, busque y elija **Aplicación web ASP.NET Core (.NET Framework)** para C# y, a continuación, seleccione **Siguiente**.

1. En **Configurar el nuevo proyecto**, asigne al proyecto el nombre _myfirstazurewebapp_ y luego seleccione **Crear**.

   ![Configuración del proyecto de aplicación web](./media/quickstart-custom-container/configure-web-app-project-container.png)

1. Puede implementar cualquier tipo de aplicación web de ASP.NET en Azure. Para este inicio rápido, elija la plantilla **MVC**.

1. Seleccione **Compatibilidad con Docker**  y asegúrese de que la autenticación se establece en **Sin autenticación**. Seleccione **Crear**.

   ![Creación de una aplicación web ASP.NET](./media/quickstart-custom-container/select-mvc-template-for-container.png)

1. Si el archivo _Dockerfile_ no se abre automáticamente, ábralo desde el **Explorador de soluciones**.

1. Necesita una [imagen principal compatible](#use-a-different-parent-image). Cambie la imagen primaria reemplazando la línea `FROM` con el código siguiente y guarde el archivo:

   ```dockerfile
   FROM mcr.microsoft.com/dotnet/framework/aspnet:4.7.2-windowsservercore-ltsc2019
   ```

1. En el menú de Visual Studio, seleccione **Depurar** > **Iniciar sin depurar** para ejecutar la aplicación web localmente.

   ![Ejecución de la aplicación de forma local](./media/quickstart-custom-container/local-web-app.png)

## <a name="publish-to-docker-hub"></a>Publicación en Docker Hub

1. En **Explorador de soluciones**, haga clic con el botón derecho en el proyecto **myfirstazurewebapp** y seleccione **Publicar**.

1. Elija **App Service** y seleccione **Publicar**.

1. En **Elegir un destino de publicación**, seleccione **Container Registry** y **Docker Hub**y, haga clic en **Publicar**.

   ![Publicar desde la página de información general del proyecto](./media/quickstart-custom-container/publish-to-docker-vs2019.png)

1. Escriba las credenciales de su cuenta de Docker Hub y seleccione **Guardar**.

   Espere a que la implementación se complete. La página **Publicar** muestra ahora el nombre del repositorio que se va a usar más adelante.

   ![Publicar desde la página de información general del proyecto](./media/quickstart-custom-container/published-docker-repository-vs2019.png)

1. Copie este nombre de repositorio para su uso posterior.

## <a name="create-a-windows-container-app"></a>Creación de una aplicación de contenedor de Windows

1. Inicie sesión en [Azure Portal]( https://portal.azure.com).

1. Elija **Crear un recurso** en la esquina superior izquierda de Azure Portal.

1. En el cuadro de búsqueda que está encima de la lista de recursos de Azure Marketplace, busque **Web App for Containers** y seleccione **Crear**.

1. En  **Web App Create** (Creación de aplicación web), elija su suscripción y un **grupo de recursos**. Si es necesario, puede crear un grupo de recursos.

1. Especifique un nombre de aplicación, como *win-container-demo* y elija **Windows** en **Sistema operativo**. Seleccione **Siguiente: Docker** para continuar.

   ![Crear una instancia de Web App for Containers](media/quickstart-custom-container/create-web-app-continer.png)

1. En **Origen de imagen**, elija **Docker Hub** y en **Imagen y etiqueta**, escriba el nombre del repositorio que copió en [Publicar en Docker Hub](#publish-to-docker-hub).

   ![Configurar una instancia de Web App for Containers](media/quickstart-custom-container/configure-web-app-continer.png)

    Si tiene una imagen personalizada de la aplicación web en otra parte como, por ejemplo, en [Azure Container Registry](../container-registry/index.yml) o en cualquier otro repositorio privado, puede configurarla aquí.

1. Seleccione **Revisar y crear** y, después, **Crear** y espere que Azure cree los recursos necesarios.

## <a name="browse-to-the-container-app"></a>Desplazamiento a la aplicación de contenedor

Una vez completada la operación de Azure, se muestra un cuadro de notificación.

![Implementación correcta](media/quickstart-custom-container/portal-create-finished.png)

1. Haga clic en **Ir al recurso**.

1. En la información general de este recurso, siga el vínculo situado junto a **URL**.

Se abre una nueva página del explorador en la página siguiente:

![Inicio de la aplicación de contenedor de Windows](media/quickstart-custom-container/app-starting.png)

Espere unos minutos e inténtelo de nuevo, hasta que llegue a la página principal predeterminada de ASP.NET:

![Aplicación de contenedor de Windows en ejecución](media/quickstart-custom-container/app-running-vs.png)

**¡Enhorabuena!** Ya está ejecutando el primer contenedor de Windows personalizado en Azure App Service.

## <a name="see-container-start-up-logs"></a>Consulta de los registros de inicio del contenedor

El contenedor de Windows puede tardar un tiempo en cargarse. Para ver el progreso, vaya a la siguiente dirección URL sustituyendo *\<app_name>* por el nombre de la aplicación.
```
https://<app_name>.scm.azurewebsites.net/api/logstream
```

Los registros transmitidos tienen este aspecto:

```
2018-07-27T12:03:11  Welcome, you are now connected to log-streaming service.
27/07/2018 12:04:10.978 INFO - Site: win-container-demo - Start container succeeded. Container: facbf6cb214de86e58557a6d073396f640bbe2fdec88f8368695c8d1331fc94b
27/07/2018 12:04:16.767 INFO - Site: win-container-demo - Container start complete
27/07/2018 12:05:05.017 INFO - Site: win-container-demo - Container start complete
27/07/2018 12:05:05.020 INFO - Site: win-container-demo - Container started successfully
```

## <a name="update-locally-and-redeploy"></a>Actualización local y nueva implementación

1. En Visual Studio, en el **Explorador de soluciones**, abra **Vistas** > **Inicio** > **Index.cshtml**.

1. Busque la etiqueta HTML `<div class="jumbotron">` en la parte superior y reemplace el elemento entero por el código siguiente:

   ```html
   <div class="jumbotron">
       <h1>ASP.NET in Azure!</h1>
       <p class="lead">This is a simple app that we've built that demonstrates how to deploy a .NET app to Azure App Service.</p>
   </div>
   ```

1. Para volver a realizar la implementación en Azure, haga clic con el botón derecho en el proyecto **myfirstazurewebapp**, en el **Explorador de soluciones** y elija **Publicar**.

1. En la página de publicación, seleccione **Publicar** y espere hasta que la publicación se complete.

1. Para indicar a App Service que extraiga la nueva imagen de Docker Hub, reinicie la aplicación. Volviendo a la página de aplicación en el portal, haga clic en **Reiniciar** > **Sí**.

   ![Reinicio de la aplicación web en Azure](./media/quickstart-custom-container/portal-restart-app.png)

Vuelva a [desplazarse a la aplicación de contenedor](#browse-to-the-container-app). Al actualizar la página web, la aplicación debe volver a la página "Iniciando" al principio y luego volver a mostrar la página web actualizada transcurridos unos minutos.

![Aplicación web actualizada en Azure](./media/quickstart-custom-container/azure-web-app-updated.png)

## <a name="use-a-different-parent-image"></a>Uso de una imagen primaria diferente

Puede usar otra imagen personalizada de Docker para ejecutar la aplicación. Sin embargo, debe elegir la [imagen primaria (imagen base)](https://docs.docker.com/develop/develop-images/baseimages/) correcta para la plataforma que desee:

- Para implementar aplicaciones de .NET Framework, use una imagen primaria basada en la versión del [canal de mantenimiento a largo plazo (LTSC)](/windows-server/get-started-19/servicing-channels-19#long-term-servicing-channel-ltsc) de Windows Server Core 2019. 
- Para implementar aplicaciones de .NET Core, use una imagen primaria basada en la versión del [canal de servicio semianual (SAC)](/windows-server/get-started-19/servicing-channels-19#semi-annual-channel) de Windows Server Nano 1809. 

La descarga de una imagen primaria tarda un tiempo en completarse durante el inicio de la aplicación. Sin embargo, puede reducir el tiempo de inicio mediante una de las siguientes imágenes primarias que ya están almacenadas en caché en Azure App Service:

- [mcr.microsoft.com/dotnet/framework/aspnet](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet/):4.7.2-windowsservercore-ltsc2019
- [mcr.microsoft.com/windows/nanoserver](https://hub.docker.com/_/microsoft-windows-nanoserver/):1809 (esta imagen es el contenedor base que se usa en las imágenes de Microsoft Windows Nano Server de Microsoft [ASP.NET Core](https://hub.docker.com/_/microsoft-dotnet-core-aspnet/)).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Migrar al contenedor de Windows en Azure](tutorial-custom-container.md)
::: zone-end  

::: zone pivot="container-linux"
App Service en Linux proporciona pilas de aplicaciones predefinidas en Linux con compatibilidad con lenguajes como .NET, PHP o Node.js entre otros. También puede usar una imagen personalizada de Docker para ejecutar la aplicación web en una pila de aplicaciones aún sin definir en Azure. En este inicio rápido se muestra cómo implementar una imagen desde [Azure Container Registry](../container-registry/index.yml) (ACR) en App Service.

## <a name="prerequisites"></a>Requisitos previos

* Una [cuenta de Azure](https://azure.microsoft.com/free/?utm_source=campaign&utm_campaign=vscode-tutorial-docker-extension&mktingSource=vscode-tutorial-docker-extension)
* [Docker](https://www.docker.com/community-edition)
* [Visual Studio Code](https://code.visualstudio.com/)
* La [extensión de Azure App Service para VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azureappservice). Puede usar esta extensión para crear, administrar e implementar Web Apps de Linux en la Plataforma como servicio (PaaS) de Azure.
* La [extensión de Docker para VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-docker). Puede usar esta extensión para simplificar la administración de imágenes y comandos locales de Docker e implementar imágenes de aplicaciones compiladas en Azure.

## <a name="create-an-image"></a>Crear una imagen

Para completar este inicio rápido, necesitará una imagen de aplicación web adecuada almacenada en [Azure Container Registry](../container-registry/index.yml). Siga las instrucciones del artículo [Guía de inicio rápido: Creación de un registro de contenedor privado con Azure Portal](../container-registry/container-registry-get-started-portal.md), pero use la imagen `mcr.microsoft.com/azuredocs/go` en lugar de `hello-world`. Como referencia, el [ejemplo de Dockerfile se encuentra en el repositorio Azure Samples](https://github.com/Azure-Samples/go-docs-hello-world).

> [!IMPORTANT]
> Asegúrese de establecer la opción **Usuario administrador** en **Habilitar** al crear el registro de contenedor. También puede establecerla en la sección **Claves de acceso** de la página de registro en Azure Portal. Esta opción de configuración es necesaria para el acceso a App Service.

## <a name="sign-in"></a>Iniciar sesión

A continuación, inicie VS Code e inicie sesión en su cuenta de Azure con la extensión App Service. Para ello, seleccione el logotipo de Azure en la barra de actividades, vaya al explorador **APP SERVICE**, después, seleccione **Iniciar sesión en Azure** y siga las instrucciones.

![inicio de sesión en Azure](./media/quickstart-docker/sign-in.png)

## <a name="check-prerequisites"></a>Comprobación de los requisitos previos

Ahora puede comprobar si ha instalado y configurado todos los requisitos previos.

En VS Code, verá su dirección de correo electrónico de Azure en la barra de estado y la suscripción en el explorador de **APP SERVICE**.

A continuación, compruebe que tiene Docker instalado y en ejecución. El siguiente comando mostrará la versión de Docker si se está ejecutando.

```bash
docker --version
```

Por último, asegúrese de que Azure Container Registry está conectado. Para ello, seleccione el logotipo de Docker en la barra de actividad y, a continuación, vaya a **REGISTRIES** (Registros).

![Registros](./media/quickstart-docker/registries.png)

## <a name="deploy-the-image-to-azure-app-service"></a>Implementación de la imagen en Azure App Service

Ahora que todo está configurado, puede implementar la imagen en [Azure App Service](https://azure.microsoft.com/services/app-service/) directamente desde el explorador de extensiones de Docker.

Busque la imagen debajo del nodo **Registries** en el Explorador de **DOCKER** y expándalo para mostrar sus etiquetas. Haga clic con el botón derecho en una etiqueta y, después, seleccione **Implementar imagen en Azure App Service**.

Desde aquí, siga las indicaciones para elegir una suscripción, un nombre de aplicación único global, un grupo de recursos y un plan de App Service. Elija **B1 básico** como plan de tarifa y una región.

Después de la implementación, la aplicación está disponible en `http://<app name>.azurewebsites.net`.

Un **grupo de recursos** es una colección con nombre de todos los recursos de la aplicación en Azure. Por ejemplo, un grupo de recursos puede contener una referencia a un sitio web, una base de datos y una función de Azure.

Un **plan de App Service** define los recursos físicos que se van a usar para hospedar el sitio web. Este inicio rápido usa un plan de hospedaje **básico** en la infraestructura de **Linux**, lo que significa que el sitio se hospedará en una máquina Linux junto con otros sitios web. Si empieza con el plan **básico**, puede usar Azure Portal para escalar verticalmente de modo que el suyo sea el único sitio que se ejecute en una máquina.

## <a name="browse-the-website"></a>Examinar el sitio web

El panel **Salida** se abrirá durante la implementación para indicar el estado de la operación. Cuando se complete la operación, busque la aplicación que creó en el explorador de **APP SERVICE**, haga clic con el botón derecho en ella y, después, seleccione **Examinar sitio web** para abrir el sitio en el explorador.

> [!div class="nextstepaction"]
> [He tenido un problema](https://www.research.net/r/PWZWZ52?tutorial=quickstart-docker&step=deploy-app)

## <a name="next-steps"></a>Pasos siguientes

Ha completado correctamente este inicio rápido.

A continuación, vea otras extensiones de Azure.

* [Cosmos DB](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-cosmosdb)
* [Funciones de Azure](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions)
* [Herramientas de la CLI de Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azurecli)
* [Herramientas de Azure Resource Manager](https://marketplace.visualstudio.com/items?itemName=msazurermtools.azurerm-vscode-tools)

O bien, puede conseguirlas todas si instala el paquete de extensiones [Azure Tools](https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-node-azure-pack).

::: zone-end