---
title: 'Inicio rápido: Creación de una aplicación de varios contenedores'
description: Para empezar a trabajar con aplicaciones de varios contenedores en Azure App Service, primero es preciso implementar la primera de estas aplicaciones.
keywords: servicio de aplicación de azure, aplicación web, linux, docker, compose, multicontenedor, varios contenedores, aplicación web para contenedores, contenedor, wordpress, azure db for mysql, base de datos de producción con contenedores
author: msangapu-msft
ms.topic: quickstart
ms.date: 08/23/2019
ms.author: msangapu
ms.custom: mvc, seodec18
ms.openlocfilehash: 562185697f8f80fc52b06be80a9f15271f16c0a3
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961472"
---
# <a name="create-a-multi-container-preview-app-using-a-docker-compose-configuration"></a>Creación de una aplicación de varios contenedores (versión preliminar) mediante una configuración de Docker Compose

> [!NOTE]
> La aplicación de varios contenedores está en versión preliminar.

[Web App for Containers](overview.md#app-service-on-linux) proporciona una manera flexible de utilizar imágenes de Docker. En este inicio rápido se muestra cómo implementar una aplicación de varios contenedores (versión preliminar) en Web App for Containers en [Cloud Shell](../cloud-shell/overview.md) mediante una configuración de Docker Compose.

Completará esta guía de inicio rápido en Cloud Shell, pero también puede ejecutar estos comandos localmente con la [CLI de Azure](/cli/azure/install-azure-cli) (2.0.32 o posterior). 

![Aplicación de varios contenedores de ejemplo en Web App for Containers][1]

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="download-the-sample"></a>Descarga del ejemplo

En esta guía de inicio rápido se usa el archivo de Compose de [Docker](https://docs.docker.com/compose/wordpress/#define-the-project). El archivo de configuración se pueden encontrar en los [ejemplos de Azure](https://github.com/Azure-Samples/multicontainerwordpress).

[!code-yml[Main](../../azure-app-service-multi-container/docker-compose-wordpress.yml)]

En Cloud Shell, cree un directorio de inicio rápido y luego cambie a él.

```bash
mkdir quickstart

cd $HOME/quickstart
```

A continuación, ejecute el comando siguiente para clonar el repositorio de la aplicación de ejemplo en el directorio de inicio rápido. Después, cambie al directorio `multicontainerwordpress`.

```bash
git clone https://github.com/Azure-Samples/multicontainerwordpress

cd multicontainerwordpress
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

[!INCLUDE [resource group intro text](../../includes/resource-group.md)]

En Cloud Shell, cree un grupo de recursos con el comando [`az group create`](/cli/azure/group?view=azure-cli-latest#az-group-create). En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *Centro-sur de EE. UU.* Para ver todas las ubicaciones de App Service que se admiten en Linux en el nivel **Estándar**, ejecute el comando [`az appservice list-locations --sku S1 --linux-workers-enabled`](/cli/azure/appservice?view=azure-cli-latest#az-appservice-list-locations).

```azurecli-interactive
az group create --name myResourceGroup --location "South Central US"
```

Generalmente se crean el grupo de recursos y los recursos en una región cercana.

Cuando finaliza el comando, una salida de JSON muestra las propiedades del grupo de recursos.

## <a name="create-an-azure-app-service-plan"></a>Crear un plan de Azure App Service

En Cloud Shell, cree un plan de App Service en el grupo de recursos con el comando [`az appservice plan create`](/cli/azure/appservice/plan?view=azure-cli-latest#az-appservice-plan-create).

En el ejemplo siguiente se crea un plan de App Service denominado `myAppServicePlan` en el plan de tarifa **Estándar** (`--sku S1`) y en un contenedor Linux (`--is-linux`).

```azurecli-interactive
az appservice plan create --name myAppServicePlan --resource-group myResourceGroup --sku S1 --is-linux
```

Cuando se crea el plan de App Service, la CLI de Azure muestra información similar al ejemplo siguiente:

<pre>
{
  "adminSiteName": null,
  "appServicePlanName": "myAppServicePlan",
  "geoRegion": "South Central US",
  "hostingEnvironmentProfile": null,
  "id": "/subscriptions/0000-0000/resourceGroups/myResourceGroup/providers/Microsoft.Web/serverfarms/myAppServicePlan",
  "kind": "linux",
  "location": "South Central US",
  "maximumNumberOfWorkers": 1,
  "name": "myAppServicePlan",
  &lt; JSON data removed for brevity. &gt;
  "targetWorkerSizeId": 0,
  "type": "Microsoft.Web/serverfarms",
  "workerTierName": null
}
</pre>

## <a name="create-a-docker-compose-app"></a>Creación de una aplicación de Docker Compose

> [!NOTE]
> En este momento, Docker Compose en Azure App Services tiene un límite de 4.000 caracteres.

En el terminal de Cloud Shell, cree una [aplicación web](overview.md#app-service-on-linux) de varios contenedores en el plan de App Service `myAppServicePlan` con el comando [az webapp create](/cli/azure/webapp?view=azure-cli-latest#az-webapp-create). No olvide reemplazar _\<app_name>_ por un nombre de aplicación único (los caracteres válidos son `a-z`, `0-9` y `-`).

```azurecli
az webapp create --resource-group myResourceGroup --plan myAppServicePlan --name <app_name> --multicontainer-config-type compose --multicontainer-config-file compose-wordpress.yml
```

Cuando se haya creado la aplicación web, la CLI de Azure mostrará información similar a la del ejemplo siguiente:

<pre>
{
  "additionalProperties": {},
  "availabilityState": "Normal",
  "clientAffinityEnabled": true,
  "clientCertEnabled": false,
  "cloningInfo": null,
  "containerSize": 0,
  "dailyMemoryTimeQuota": 0,
  "defaultHostName": "&lt;app_name&gt;.azurewebsites.net",
  "enabled": true,
  &lt; JSON data removed for brevity. &gt;
}
</pre>

### <a name="browse-to-the-app"></a>Navegación hasta la aplicación

Vaya a la aplicación implementada en (`http://<app_name>.azurewebsites.net`). La aplicación puede tardar unos minutos en cargarse. Si recibe un error, deje pasar unos minutos, y actualice el explorador.

![Aplicación de varios contenedores de ejemplo en Web App for Containers][1]

**Enhorabuena**, ha creado una aplicación de varios contenedores en Web App for Containers.

[!INCLUDE [Clean-up section](../../includes/cli-script-clean-up.md)]

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Aplicación de WordPress con varios contenedores](tutorial-multi-container-app.md)

> [!div class="nextstepaction"]
> [Configurar un contenedor personalizado](configure-custom-container.md)

<!--Image references-->
[1]: media/tutorial-multi-container-app/azure-multi-container-wordpress-install.png