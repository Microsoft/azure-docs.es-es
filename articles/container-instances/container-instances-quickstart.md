---
title: 'Inicio rápido: implementación de un contenedor de Docker en una instancia de contenedor (CLI de Azure)'
description: En este inicio rápido, usará la CLI de Azure para implementar rápidamente una aplicación web en contenedores que se ejecuta en una instancia de contenedor aislada de Azure
ms.topic: quickstart
ms.date: 03/21/2019
ms.custom:
- seo-python-october2019
- seodec18
- mvc
- devx-track-javascript
- devx-track-azurecli
ms.openlocfilehash: 31e7e7d6785e8fd52f4c76171f6ca9d7f5c8383c
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501328"
---
# <a name="quickstart-deploy-a-container-instance-in-azure-using-the-azure-cli"></a>Inicio rápido: Implementación de una instancia de contenedor en Azure mediante la CLI de Azure

Use Azure Container Instances para ejecutar contenedores de Docker sin servidor en Azure con sencillez y velocidad. Implemente una aplicación en una instancia de contenedor a petición cuando no necesite una plataforma de orquestación de contenedores completa, como Azure Kubernetes Service.

En esta guía de inicio rápido, va a usar la CLI de Azure para implementar un contenedor de Docker aislado y hacer que su aplicación esté disponible con un nombre de dominio completo (FQDN). Unos pocos segundos después de ejecutar un comando de implementación único, puede ir a la aplicación que se ejecuta en el contenedor:

![Visualización de una aplicación implementada en Azure Container Instances en el explorador][aci-app-browser]

Si no tiene una suscripción a Azure, cree una [cuenta gratuita][azure-account] antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Puede usar Azure Cloud Shell o una instalación local de la CLI de Azure para completar esta guía de inicio rápido. Si desea usarlos de forma local, se recomienda la versión 2.0.55 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Las instancias de contenedores de Azure, al igual que otros recursos de Azure, se deben implementar en un grupo de recursos. Los grupos de recursos le permiten organizar y administrar los recursos relacionados de Azure.

En primer lugar, cree un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus* con el comando [az group create][az-group-create] siguiente:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-container"></a>Crear un contenedor

Ahora que tiene un grupo de recursos, puede ejecutar un contenedor en Azure. Para crear una instancia de contenedor con la CLI de Azure, proporcione un nombre de grupo de recursos, un nombre de instancia de contenedor y una imagen de contenedor de Docker al comando [az container create][az-container-create]. En este inicio rápido, usará la imagen `mcr.microsoft.com/azuredocs/aci-helloworld` pública. Esta imagen empaqueta una pequeña aplicación web escrita en Node.js que sirve una página HTML estática.

Puede exponer los contenedores en Internet mediante la especificación para que se abran uno o varios puertos, o la especificación de una etiqueta de nombre DNS o ambas. En este inicio rápido, se implementa un contenedor con una etiqueta de nombre DNS para que la aplicación web sea públicamente accesible.

Ejecute un comando similar al siguiente para iniciar una instancia de contenedor. Establezca un valor `--dns-name-label` que sea único dentro de la región de Azure en la que cree la instancia. Si recibe un mensaje de error "DNS name label not available" (La etiqueta de nombre DNS no está disponible), pruebe otra etiqueta de nombre DNS diferente.

```azurecli-interactive
az container create --resource-group myResourceGroup --name mycontainer --image mcr.microsoft.com/azuredocs/aci-helloworld --dns-name-label aci-demo --ports 80
```

En unos segundos, debería recibir una respuesta de la CLI de Azure que indica que la implementación ha finalizado. Compruebe su estado con el comando [az container show][az-container-show]:

```azurecli-interactive
az container show --resource-group myResourceGroup --name mycontainer --query "{FQDN:ipAddress.fqdn,ProvisioningState:provisioningState}" --out table
```

Al ejecutar el comando, se muestra el nombre completo del dominio (FQDN) y su estado de aprovisionamiento.

```output
FQDN                               ProvisioningState
---------------------------------  -------------------
aci-demo.eastus.azurecontainer.io  Succeeded
```

Si el estado `ProvisioningState` del contenedor es **Correcto**, vaya a su FQDN en el explorador. Si ve una página web parecida a la siguiente, enhorabuena. Ha implementado correctamente una aplicación que se ejecuta en un contenedor de Docker en Azure.

![Visualización de una aplicación implementada en Azure Container Instances en el explorador][aci-app-browser]

Si al principio no aparece la aplicación, espere unos segundos mientras se propaga el DNS y, a continuación, pruebe a actualizar el explorador.

## <a name="pull-the-container-logs"></a>Extracción de los registros del contenedor

Cuando necesite solucionar problemas de un contenedor o de la aplicación que este ejecuta (o simplemente ver la salida), empiece por ver los registros de la instancia de contenedor.

Extraiga los registros de la instancia de contenedor con el comando [az container logs][az-container-logs]:

```azurecli-interactive
az container logs --resource-group myResourceGroup --name mycontainer
```

En la salida se muestran los registros del contenedor y se mostrarán las solicitudes HTTP GET generadas al ver la aplicación en el explorador.

```output
listening on port 80
::ffff:10.240.255.55 - - [21/Mar/2019:17:43:53 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [21/Mar/2019:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [21/Mar/2019:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
```

## <a name="attach-output-streams"></a>Flujos de salida como adjuntos

Además de ver los registros, puede asociar la salida estándar local y los flujos de error estándar a los del contenedor.

En primer lugar, ejecute el comando [az container attach][az-container-attach] para asociar la consola local a los flujos de salida del contenedor:

```azurecli-interactive
az container attach --resource-group myResourceGroup --name mycontainer
```

Una vez asociados, actualice el explorador varias veces para generar algunas salidas adicionales. Cuando haya terminado, desconecte la consola con `Control+C`. Debería ver un resultado similar al siguiente:

```output
Container 'mycontainer' is in state 'Running'...
(count: 1) (last timestamp: 2019-03-21 17:27:20+00:00) pulling image "mcr.microsoft.com/azuredocs/aci-helloworld"
(count: 1) (last timestamp: 2019-03-21 17:27:24+00:00) Successfully pulled image "mcr.microsoft.com/azuredocs/aci-helloworld"
(count: 1) (last timestamp: 2019-03-21 17:27:27+00:00) Created container
(count: 1) (last timestamp: 2019-03-21 17:27:27+00:00) Started container

Start streaming logs:
listening on port 80

::ffff:10.240.255.55 - - [21/Mar/2019:17:43:53 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [21/Mar/2019:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [21/Mar/2019:17:44:36 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.55 - - [21/Mar/2019:17:47:01 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
::ffff:10.240.255.56 - - [21/Mar/2019:17:47:12 +0000] "GET / HTTP/1.1" 304 - "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36"
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con el contenedor, puede eliminarlo con el comando [az container delete][az-container-delete]:

```azurecli-interactive
az container delete --resource-group myResourceGroup --name mycontainer
```

Para comprobar que se ha eliminado el contenedor, ejecute el comando [az container list](/cli/azure/container#az-container-list):

```azurecli-interactive
az container list --resource-group myResourceGroup --output table
```

El contenedor **mycontainer** no debe aparecer en la salida del comando. Si no cuenta con ningún otro contenedor en el grupo de recursos, no se muestra ninguna salida.

Si ha terminado con el grupo de recursos *myResourceGroup* y todos los recursos que contiene, elimínelo con el comando [az group delete][az-group-delete]:

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una instancia de contenedor de Azure a partir de una imagen de Microsoft pública. Si quiere compilar una imagen de contenedor e implementarla desde un registro de contenedor privado de Azure, vaya al tutorial de Azure Container Instances.

> [!div class="nextstepaction"]
> [Tutorial de Azure Container Instances](./container-instances-tutorial-prepare-app.md)

Para probar opciones de ejecución de contenedores en un sistema de orquestación en Azure, consulte los inicios rápidos de [Azure Kubernetes Service (AKS)][container-service].

<!-- IMAGES -->
[aci-app-browser]: ./media/container-instances-quickstart/view-an-application-running-in-an-azure-container-instance.png

<!-- LINKS - External -->
[app-github-repo]: https://github.com/Azure-Samples/aci-helloworld.git
[azure-account]: https://azure.microsoft.com/free/
[node-js]: https://nodejs.org

<!-- LINKS - Internal -->
[az-container-attach]: /cli/azure/container#az-container-attach
[az-container-create]: /cli/azure/container#az-container-create
[az-container-delete]: /cli/azure/container#az-container-delete
[az-container-list]: /cli/azure/container#az-container-list
[az-container-logs]: /cli/azure/container#az-container-logs
[az-container-show]: /cli/azure/container#az-container-show
[az-group-create]: /cli/azure/group#az-group-create
[az-group-delete]: /cli/azure/group#az-group-delete
[azure-cli-install]: /cli/azure/install-azure-cli
[container-service]: ../aks/kubernetes-walkthrough.md
