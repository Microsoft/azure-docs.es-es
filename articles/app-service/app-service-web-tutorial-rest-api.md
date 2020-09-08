---
title: 'Tutorial: Hospedaje de API RESTful con CORS'
description: Aprenda cómo Azure App Service le ayuda a hospedar API RESTful con compatibilidad CORS. App Service puede hospedar aplicaciones web de front-end y API de back-end.
ms.assetid: a820e400-06af-4852-8627-12b3db4a8e70
ms.devlang: dotnet
ms.topic: tutorial
ms.date: 04/28/2020
ms.custom: devx-track-csharp, mvc, devcenter, seo-javascript-september2019, seo-javascript-october2019, seodec18
ms.openlocfilehash: 9481b6d2740d27b8c3d1309e205edda6017868fa
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213695"
---
# <a name="tutorial-host-a-restful-api-with-cors-in-azure-app-service"></a>Tutorial: Hospedaje de una API RESTful con CORS en Azure App Service

[Azure App Service](overview.md) proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones de un modo automático. Además, App Service tiene compatibilidad integrada para el [uso compartido de recursos entre orígenes (CORS)](https://wikipedia.org/wiki/Cross-Origin_Resource_Sharing) para API RESTful. En este tutorial se muestra cómo implementar una aplicación de API de ASP. NET Core en App Service con compatibilidad con CORS. Va a configurar la aplicación mediante el uso de herramientas de línea de comandos e implementar la aplicación usando Git. 

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear recursos de App Service mediante la CLI de Azure
> * Implementar una API RESTful en Azure con Git
> * Habilitar la compatibilidad con CORS de App Service

También puede seguir los pasos de este tutorial en macOS, Linux, Windows.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial:

* <a href="https://git-scm.com/" target="_blank">Instalación de Git</a>
 * <a href="https://dotnet.microsoft.com/download/dotnet-core/3.1" target="_blank">Instalación del SDK más reciente de .NET Core 3.1</a>

## <a name="create-local-aspnet-core-app"></a>Creación de una aplicación ASP.NET Core local

En este paso, configurará el proyecto ASP.NET Core local. App Service admite el mismo flujo de trabajo para API escritas en otros lenguajes.

### <a name="clone-the-sample-application"></a>Clonación de la aplicación de ejemplo

En la ventana del terminal, use `cd` para cambiar a un directorio de trabajo.  

Ejecute el comando siguiente para clonar el repositorio de ejemplo. 

```bash
git clone https://github.com/Azure-Samples/dotnet-core-api
```

Este repositorio contiene una aplicación creada según el siguiente tutorial: [Páginas de ayuda de ASP.NET Core Web API mediante Swagger](/aspnet/core/tutorials/web-api-help-pages-using-swagger?tabs=visual-studio). Utiliza un generador de Swagger para atender la [interfaz de usuario de Swagger](https://swagger.io/swagger-ui/) y el punto de conexión JSON de Swagger.

### <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute los comandos siguientes para instalar los paquetes necesarios, ejecutar las migraciones de bases de datos e iniciar la aplicación.

```bash
cd dotnet-core-api
dotnet restore
dotnet run
```

Vaya a `http://localhost:5000/swagger` en un explorador para reproducirla con la interfaz de usuario de Swagger.

![API de ASP.NET Core en ejecución local](./media/app-service-web-tutorial-rest-api/azure-app-service-local-swagger-ui.png)

Vaya a `http://localhost:5000/api/todo` y vea una lista de elementos de JSON de ToDo.

Vaya a `http://localhost:5000` y reprodúzcala con la aplicación del explorador. Más adelante, apuntará la aplicación del explorador a una API remota en App Service para probar la funcionalidad CORS. El código de la aplicación del explorador se encuentra en el directorio _wwwroot_ del repositorio.

Para detener ASP.NET Core en cualquier momento, presione `Ctrl+C` en el terminal.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="deploy-app-to-azure"></a>Implementación de la aplicación en Azure

En este paso, va a implementar la aplicación .NET Core conectada a SQL Database en App Service.

### <a name="configure-local-git-deployment"></a>Configuración de la implementación de Git local

[!INCLUDE [Configure a deployment user](../../includes/configure-deployment-user-no-h.md)]

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-no-h.md)]

### <a name="create-an-app-service-plan"></a>Creación de un plan de App Service

[!INCLUDE [Create app service plan](../../includes/app-service-web-create-app-service-plan-no-h.md)]

### <a name="create-a-web-app"></a>Creación de una aplicación web

[!INCLUDE [Create web app](../../includes/app-service-web-create-web-app-dotnetcore-win-no-h.md)] 

### <a name="push-to-azure-from-git"></a>Inserción en Azure desde Git

[!INCLUDE [app-service-plan-no-h](../../includes/app-service-web-git-push-to-azure-no-h.md)]

<pre>
Enumerating objects: 83, done.
Counting objects: 100% (83/83), done.
Delta compression using up to 8 threads
Compressing objects: 100% (78/78), done.
Writing objects: 100% (83/83), 22.15 KiB | 3.69 MiB/s, done.
Total 83 (delta 26), reused 0 (delta 0)
remote: Updating branch 'master'.
remote: Updating submodules.
remote: Preparing deployment for commit id '509236e13d'.
remote: Generating deployment script.
remote: Project file path: .\TodoApi.csproj
remote: Generating deployment script for ASP.NET MSBuild16 App
remote: Generated deployment script files
remote: Running deployment command...
remote: Handling ASP.NET Core Web Application deployment with MSBuild16.
remote: .
remote: .
remote: .
remote: Finished successfully.
remote: Running post deployment command(s)...
remote: Triggering recycle (preview mode disabled).
remote: Deployment successful.
To https://&lt;app_name&gt;.scm.azurewebsites.net/&lt;app_name&gt;.git
 * [new branch]      master -> master
</pre>

### <a name="browse-to-the-azure-app"></a>Navegación hasta la aplicación de Azure

Vaya a `http://<app_name>.azurewebsites.net/swagger` en un explorador y reprodúzcala con la interfaz de usuario de Swagger.

![API de ASP.NET Core que se ejecuta en Azure App Service](./media/app-service-web-tutorial-rest-api/azure-app-service-browse-app.png)

Vaya a `http://<app_name>.azurewebsites.net/swagger/v1/swagger.json` para ver el archivo _swagger.json_ de la API implementada.

Vaya a `http://<app_name>.azurewebsites.net/api/todo` para ver cómo trabaja la API implementada.

## <a name="add-cors-functionality"></a>Adición de funcionalidad CORS

Después, habilite la compatibilidad integrada de CORS en App Service para la API.

### <a name="test-cors-in-sample-app"></a>Prueba de CORS en la aplicación de ejemplo

En el repositorio local, abra _wwwroot/index.html_.

En la línea 51, establezca la variable `apiEndpoint` en la dirección URL de la API implementada (`http://<app_name>.azurewebsites.net`). Reemplace _\<appname>_ por el nombre de la aplicación de App Service.

En la ventana de terminal local, vuelva a ejecutar la aplicación de ejemplo.

```bash
dotnet run
```

Vaya a la aplicación del explorador en `http://localhost:5000`. Abra la ventana de herramientas del desarrollador en el explorador (`Ctrl`+`Shift`+`i` en Chrome para Windows) e inspeccione la pestaña **Consola**. Ahora debería ver el mensaje de error `No 'Access-Control-Allow-Origin' header is present on the requested resource`.

![Error de CORS en el cliente del explorador](./media/app-service-web-tutorial-rest-api/azure-app-service-cors-error.png)

Debido al error de coincidencia del dominio entre la aplicación del explorador (`http://localhost:5000`) y el recurso remoto (`http://<app_name>.azurewebsites.net`), y al hecho de que la API de App Service no está enviando el encabezado `Access-Control-Allow-Origin`, el explorador ha impedido que el contenido entre dominios se cargue en la aplicación del explorador.

En producción, la aplicación del explorador tendría una dirección URL pública en lugar de la dirección URL del host local, pero la forma de habilitar CORS en una dirección URL del host local es la misma que en una dirección URL pública.

### <a name="enable-cors"></a>Habilitación de CORS 

En Cloud Shell, habilite CORS en la dirección URL del cliente mediante el comando [`az webapp cors add`](/cli/azure/webapp/cors#az-webapp-cors-add). Reemplace el marcador de posición _&lt;app-name>_ .

```azurecli-interactive
az webapp cors add --resource-group myResourceGroup --name <app-name> --allowed-origins 'http://localhost:5000'
```

Puede establecer más de una dirección URL de cliente en `properties.cors.allowedOrigins` (`"['URL1','URL2',...]"`). También puede habilitar todas las direcciones URL de cliente con `"['*']"`.

> [!NOTE]
> Si la aplicación requiere que se envíen credenciales, como cookies o tokens de autenticación, el explorador puede requerir el encabezado `ACCESS-CONTROL-ALLOW-CREDENTIALS` en la respuesta. Para habilitarlo en App Service, establezca `properties.cors.supportCredentials` en `true` en la configuración de CORS. No se puede habilitar cuando `allowedOrigins` incluye `'*'`.

### <a name="test-cors-again"></a>Otra prueba de CORS

Actualice la aplicación del explorador en `http://localhost:5000`. El mensaje de error de la ventana **Consola** ha desaparecido y puede ver los datos de la API implementada e interactuar con ella. La API remota ahora admite CORS en la aplicación del explorador que se ejecuta localmente. 

![CORS implementado correctamente en el cliente de explorador](./media/app-service-web-tutorial-rest-api/azure-app-service-cors-success.png)

Enhorabuena, ya está ejecutando una API en Azure App Service con compatibilidad CORS.

## <a name="app-service-cors-vs-your-cors"></a>CORS de App Service frente a CORS del usuario

Puede usar sus propias utilidades CORS en lugar de CORS de App Service para una mayor flexibilidad. Por ejemplo, es posible que desee especificar diferentes orígenes permitidos para rutas o métodos distintos. Dado que CORS de App Service le permite especificar un conjunto de orígenes aceptados para todos los métodos y rutas de API, debería utilizar su propio código CORS. Consulte cómo lo realiza ASP.NET Core en [Habilitación de solicitudes entre orígenes (CORS)](/aspnet/core/security/cors).

> [!NOTE]
> No intente utilizar juntos CORS de App Service y su propio código CORS. Cuando se usan juntos, CORS de App Service tiene prioridad y su propio código CORS no tiene ningún efecto.
>
>

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

<a name="next"></a>
## <a name="next-steps"></a>Pasos siguientes

¿Qué ha aprendido?

> [!div class="checklist"]
> * Crear recursos de App Service mediante la CLI de Azure
> * Implementar una API RESTful en Azure con Git
> * Habilitar la compatibilidad con CORS de App Service

Avance al siguiente tutorial para aprender a autenticar y autorizar a los usuarios.

> [!div class="nextstepaction"]
> [Tutorial: Autenticación y autorización de usuarios de un extremo a otro](tutorial-auth-aad.md)
