---
title: 'Tutorial: Implementación de una aplicación de contenedor en una instancia de contenedor'
description: 'Tutorial de Azure Container Instances, parte 3 de 3: implementación de una aplicación de contenedor en Azure Container Instances'
ms.topic: tutorial
ms.date: 03/21/2018
ms.custom: seodec18, mvc, devx-track-azurecli
ms.openlocfilehash: 2ea3d285f00d38df84587d9a7c15242fff38453b
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500579"
---
# <a name="tutorial-deploy-a-container-application-to-azure-container-instances"></a>Tutorial: Implementación de una aplicación de contenedor en Azure Container Instances

Este es el último tutorial de una serie de tres partes. En las series anteriores, [se creó una imagen de contenedor](container-instances-tutorial-prepare-app.md) y [se insertó en una instancia de Azure Container Registry](container-instances-tutorial-prepare-acr.md). En este artículo se completa la serie con la implementación del contenedor en Azure Container Instances.

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Implementar el contenedor de Azure Container Registry a Azure Container Instances
> * Visualizar la aplicación en ejecución en el explorador
> * Mostrar los registros del contenedor

## <a name="before-you-begin"></a>Antes de empezar

[!INCLUDE [container-instances-tutorial-prerequisites](../../includes/container-instances-tutorial-prerequisites.md)]

## <a name="deploy-the-container-using-the-azure-cli"></a>Implementación del contenedor mediante la CLI de Azure

En esta sección, se utiliza la CLI de Azure para implementar la imagen integrada en el [primer tutorial](container-instances-tutorial-prepare-app.md) e insertarla en Azure Container Registry del [segundo tutorial](container-instances-tutorial-prepare-acr.md). Asegúrese de haber completado estos tutoriales antes de continuar.

### <a name="get-registry-credentials"></a>Obtención de las credenciales del registro

Al implementar una imagen que se hospeda en una instancia privada de Azure Container Registry como la que se creó en el [segundo tutorial](container-instances-tutorial-prepare-acr.md), debe proporcionar las credenciales de acceso al registro. 

En muchos escenarios, uno de los procedimientos recomendados es crear y configurar una entidad de servicio de Azure Active Directory con permisos de *extracción* en el registro. Consulte [Autenticación con Azure Container Registry desde Azure Container Instances](../container-registry/container-registry-auth-aci.md) para obtener scripts de ejemplo con el fin de crear una entidad de servicio con los permisos necesarios. Anote el *identificador* y la *contraseña de la entidad de servicio*. Usará estas credenciales para acceder al registro al implementar el contenedor.

También necesita el nombre completo del servidor de inicio de sesión del registro de contenedor (reemplace `<acrName>` por el nombre del registro):

```azurecli
az acr show --name <acrName> --query loginServer
```

### <a name="deploy-container"></a>Implementación de un contenedor

Ahora, utilice el comando [az container create][az-container-create] para implementar el contenedor. Reemplace `<acrLoginServer>` por el valor obtenido con el comando anterior. Reemplace `<service-principal-ID>` y `<service-principal-password>` por el identificador de la entidad de servicio y la contraseña que creó para acceder al registro. Reemplace `<aciDnsLabel>` por el nombre de DNS que desee.

```azurecli
az container create --resource-group myResourceGroup --name aci-tutorial-app --image <acrLoginServer>/aci-tutorial-app:v1 --cpu 1 --memory 1 --registry-login-server <acrLoginServer> --registry-username <service-principal-ID> --registry-password <service-principal-password> --dns-name-label <aciDnsLabel> --ports 80
```

Al cabo de unos segundos, debe recibir una respuesta inicial de Azure. El valor `--dns-name-label` debe ser único dentro de la región de Azure en la que crea la instancia de contenedor. Modifique el valor del comando anterior si recibe un mensaje de error de **etiqueta de nombre DNS** al ejecutar el comando.

### <a name="verify-deployment-progress"></a>Comprobación del progreso de la implementación

Para ver el estado de la implementación, use [az container show][az-container-show]:

```azurecli
az container show --resource-group myResourceGroup --name aci-tutorial-app --query instanceView.state
```

Repita el comando [az container show][az-container-show] hasta que el estado cambie de *Pendiente* a *En ejecución*, lo que debe tardar menos de un minuto. Cuando el contenedor esté en *En ejecución*, continúe con el paso siguiente.

## <a name="view-the-application-and-container-logs"></a>Visualización de los registros de contenedor y aplicación

Cuando se complete correctamente la implementación, utilice el comando [az container show][az-container-show] para mostrar el nombre de dominio completo (FQDN) del contenedor con el comando:

```azurecli
az container show --resource-group myResourceGroup --name aci-tutorial-app --query ipAddress.fqdn
```

Por ejemplo:
```output
"aci-demo.eastus.azurecontainer.io"
```

Para ver la aplicación en ejecución, vaya al nombre DNS que aparece en su explorador favorito:

![Aplicación Hola mundo en el explorador][aci-app-browser]

También puede ver la salida de registro del contenedor:

```azurecli
az container logs --resource-group myResourceGroup --name aci-tutorial-app
```

Salida de ejemplo:

```output
listening on port 80
::ffff:10.240.0.4 - - [21/Jul/2017:06:00:02 +0000] "GET / HTTP/1.1" 200 1663 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36"
::ffff:10.240.0.4 - - [21/Jul/2017:06:00:02 +0000] "GET /favicon.ico HTTP/1.1" 404 150 "http://aci-demo.eastus.azurecontainer.io/" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36"
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ya no necesita ninguno de los recursos creados en esta serie de tutoriales, puede ejecutar el comando [az group delete][az-group-delete] para quitar el grupo de recursos y todos los recursos que contiene. Este comando elimina el registro de contenedor que se ha creado, así como el contenedor en ejecución y todos los recursos relacionados.

```azurecli-interactive
az group delete --name myResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial se completó el proceso de implementar los contenedores en Azure Container Instances. Se han completado los siguientes pasos:

> [!div class="checklist"]
> * Implementación del contenedor desde Azure Container Registry mediante la CLI de Azure
> * Visualización de la aplicación en el explorador
> * Visualización de los registros del contenedor

Ahora que ya tiene los conceptos básicos, va a aprender más acerca de Azure Container Instances, como por ejemplo, cómo funcionan los grupos de contenedores:

> [!div class="nextstepaction"]
> [Grupos de contenedores en Azure Container Instances](container-instances-container-groups.md)

<!-- IMAGES -->
[aci-app-browser]: ./media/container-instances-quickstart/aci-app-browser.png

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show
[az-group-delete]: /cli/azure/group#az-group-delete
[azure-cli-install]: /cli/azure/install-azure-cli
[prepare-app]: ./container-instances-tutorial-prepare-app.md
