---
title: 'Creación de una aplicación web de Node.js: Azure App Service | Microsoft Docs'
description: Implementación de su primera aplicación Hola mundo de Node.js en Azure App Service Web Apps en cuestión de minutos.
services: app-service\web
documentationcenter: ''
author: cephalin
manager: jeconnoc
editor: ''
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.service: app-service-web
ms.workload: web
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 02/15/2019
ms.author: cephalin
ms.custom: seodec18
ms.openlocfilehash: c103e6cb3626750414ee5083dad3e34b6be4986c
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408942"
---
# <a name="create-a-nodejs-web-app-in-azure"></a>Creación de una aplicación web de Node.js en Azure

> [!NOTE]
> En este artículo se implementa una aplicación en App Service en Windows. Para realizar implementaciones en App Service en _Linux_, consulte [Creación de una aplicación web de Node.js en Azure App Service en Linux](./containers/quickstart-nodejs.md).
>

[Azure App Service](overview.md) proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones de un modo automático.  En este tutorial rápido se explica cómo se implementa una aplicación de Node.js en Azure App Service. Se crea la aplicación web con la [CLI de Azure](https://docs.microsoft.com/cli/azure/get-started-with-azure-cli) y se usa ZipDeploy para implementar el código Node.js de ejemplo en la aplicación web.

![Aplicación de ejemplo que se ejecuta en Azure](media/app-service-web-get-started-nodejs-poc/hello-world-in-browser.png)

Estos pasos se pueden realizar en este caso con una máquina Mac, Windows o Linux. Una vez instalados los requisitos previos, tardará aproximadamente cinco minutos en completar los pasos.   

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Requisitos previos

Para completar esta guía de inicio rápido:

* <a href="https://nodejs.org/" target="_blank">Instalación de Node.js y NPM</a>

## <a name="download-the-sample"></a>Descarga del ejemplo

Descargue el proyecto de ejemplo Node.js de [https://github.com/Azure-Samples/nodejs-docs-hello-world/archive/master.zip](https://github.com/Azure-Samples/nodejs-docs-hello-world/archive/master.zip) y extraiga el archivo ZIP.

Abra _index.js_ y busque la línea siguiente:

```javascript
const port = process.env.PORT || 1337;
```

App Service inserta process.env.PORT en su aplicación, por lo que el código usa la variable para saber qué puerto debe escuchar. 

En una ventana de terminal, vaya al **directorio raíz** del proyecto de ejemplo de Node.js (el que contiene _index.js_).

## <a name="run-the-app-locally"></a>Ejecución de la aplicación de forma local

Ejecute la aplicación localmente para ver cómo debería ser si se implementara en Azure. Abra una ventana de terminal y use el script `npm start` para iniciar el servidor HTTP de Node.js integrado.

```bash
npm start
```

Abra un explorador web y vaya a la aplicación de ejemplo en `http://localhost:1337`.

Verá el mensaje **Hola mundo** de la aplicación de ejemplo que aparece en la página.

![Aplicación de ejemplo que se ejecuta localmente](media/app-service-web-get-started-nodejs-poc/localhost-hello-world-in-browser.png)

En la ventana de terminal, presione **Ctrl + C** para salir del servidor web.

> [!NOTE]
> En Azure App Service, la aplicación se ejecuta en IIS mediante [iisnode](https://github.com/Azure/iisnode). Para habilitar que la aplicación se ejecute con iisnode, el directorio de la aplicación raíz contiene el archivo web.config. IIS puede leer dicho archivo y la configuración de iisnode se documenta en [el repositorio de GitHub de iisnode](https://github.com/Azure/iisnode/blob/master/src/samples/configuration/web.config).

## <a name="create-a-project-zip-file"></a>Creación de un archivo ZIP de proyecto

Asegúrese de que está todavía en el **directorio raíz** del proyecto de ejemplo (el que contiene _index.js_). Cree un archivo ZIP con todo el contenido del proyecto. El siguiente comando usa la herramienta predeterminada de su terminal:

```
# Bash
zip -r myAppFiles.zip .

# PowerShell
Compress-Archive -Path * -DestinationPath myAppFiles.zip
```

Posteriormente, cargue el archivo ZIP en Azure e impleméntelo en App Service.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

[!INCLUDE [Create resource group](../../includes/app-service-web-create-resource-group-scus.md)] 

[!INCLUDE [Create app service plan](../../includes/app-service-web-create-app-service-plan-scus.md)] 

## <a name="create-a-web-app"></a>Creación de una aplicación web

En Cloud Shell, cree una aplicación web en el plan de App Service `myAppServicePlan` con el comando [`az webapp create`](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create). 

En el siguiente ejemplo, reemplace `<app_name>` por un nombre único global de aplicación (los caracteres válidos son `a-z`, `0-9` y `-`).

```azurecli-interactive
# Bash and Powershell
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name>
```

Cuando se haya creado la aplicación web, la CLI de Azure mostrará información similar a la del ejemplo siguiente:

```json
{
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "<app_name>.azurewebsites.net",
  "enabled": true,
  < JSON data removed for brevity. >
}
```

### <a name="set-nodejs-runtime"></a>Tiempo de ejecución de Node.js

Establezca el entorno de ejecución de Node en 10.14.1. Para ver todos los entornos en tiempo de ejecución admitidos, ejecute [`az webapp list-runtimes`](/cli/azure/webapp?view=azure-cli-latest#az-webapp-list-runtimes).

```azurecli-interactive
# Bash and Powershell
az webapp config appsettings set --resource-group myResourceGroup --name <app_name> --settings WEBSITE_NODE_DEFAULT_VERSION=10.14.1
```

Vaya a la aplicación web recién creada. Reemplace `<app_name>` por un nombre de aplicación único.

```bash
http://<app_name>.azurewebsites.net
```

Este es el aspecto que debería tener su nueva aplicación web:

![Página de la aplicación web vacía](media/app-service-web-get-started-nodejs-poc/app-service-web-service-created.png)

[!INCLUDE [Deploy ZIP file](../../includes/app-service-web-deploy-zip.md)]

## <a name="browse-to-the-app"></a>Navegación hasta la aplicación

Vaya a la aplicación implementada mediante el explorador web.

```
http://<app_name>.azurewebsites.net
```

El código de ejemplo de Node.js se está ejecutando en una aplicación web de Azure App Service.

![Aplicación de ejemplo que se ejecuta en Azure](media/app-service-web-get-started-nodejs-poc/hello-world-in-browser.png)

**¡Enhorabuena!** Ha implementado la primera aplicación de Node.js en App Service.

## <a name="update-and-redeploy-the-code"></a>Actualización del código y nueva implementación

Con un editor de texto, abra el archivo `index.js` en la aplicación de Node.js y realice un pequeño cambio en el texto en la llamada a `response.end`:

```javascript
response.end("Hello Azure!");
```

En la ventana de terminal local, desplácese al **directorio raíz** (el que contiene _index.js_) de la aplicación y cree un nuevo archivo ZIP para el proyecto actualizado.

```azurecli-interactive
# Bash
zip -r myUpdatedAppFiles.zip .

# PowerShell
Compress-Archive -Path * -DestinationPath myUpdatedAppFiles.zip
```

Implemente este nuevo archivo ZIP en App Service mediante los mismos pasos de [Implementación de un archivo ZIP](#deploy-zip-file).

Vuelva a la ventana del explorador que se abrió en el paso **Navegación hasta la aplicación** y actualice la página.

![Aplicación de ejemplo actualizada que se ejecuta en Azure](media/app-service-web-get-started-nodejs-poc/hello-azure-in-browser.png)

## <a name="manage-your-new-azure-app"></a>Administración de la nueva aplicación de Azure

Vaya a <a href="https://portal.azure.com" target="_blank">Azure Portal</a> para administrar la aplicación web que ha creado.

En el menú izquierdo, haga clic en **App Services** y, luego, en el nombre de la aplicación de Azure.

![Navegación en el portal a la aplicación de Azure](./media/app-service-web-get-started-nodejs-poc/nodejs-docs-hello-world-app-service-list.png)

Podrá ver la página de información general de la aplicación web. En este caso, puede realizar tareas de administración básicas como examinar, detener, iniciar, reiniciar y eliminar. 

![Página de App Service en Azure Portal](media/app-service-web-get-started-nodejs-poc/nodejs-docs-hello-world-app-service-detail.png)

El menú izquierdo proporciona distintas páginas para configurar la aplicación. 

[!INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Node.js con MongoDB](app-service-web-tutorial-nodejs-mongodb-app.md)
