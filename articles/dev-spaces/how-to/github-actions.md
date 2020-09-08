---
title: Acciones de GitHub y Azure Kubernetes Service (versión preliminar)
services: azure-dev-spaces
ms.date: 04/03/2020
ms.topic: conceptual
description: Revise y pruebe los cambios de una solicitud de incorporación de cambios directamente en Azure Kubernetes Service mediante acciones de GitHub y Azure Dev Spaces.
keywords: Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, contenedores, acciones de GitHub, Helm, service mesh, enrutamiento de service mesh, kubectl, k8s
manager: gwallace
ms.custom: devx-track-javascript
ms.openlocfilehash: 5e3417f16791b71d53a0eec9263532219c779440
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212501"
---
# <a name="github-actions--azure-kubernetes-service-preview"></a>Acciones de GitHub y Azure Kubernetes Service (versión preliminar)

Azure Dev Spaces proporciona un flujo de trabajo con Acciones de GitHub que le permite probar los cambios de una solicitud de incorporación de cambios directamente en AKS antes de que dicha solicitud se combine en la rama principal del repositorio. Tener una aplicación en ejecución para revisar los cambios de una solicitud de incorporación de cambios puede aumentar la confianza del desarrollador y de los miembros del equipo. Esta aplicación en ejecución también puede ayudar a los miembros del equipo, como administradores de productos y diseñadores, a formar parte del proceso de revisión durante las primeras fases del desarrollo.

En esta guía, aprenderá a:

* Configurar Azure Dev Spaces en un clúster de Kubernetes administrado en Azure.
* Implementar una aplicación de gran tamaño con varios microservicios en un espacio de desarrollo.
* Configurar CI/CD con acciones de GitHub.
* Probar un único microservicio en un espacio aislado de desarrollo dentro del contexto de la aplicación completa.

> [!IMPORTANT]
> Esta funcionalidad actualmente está en su versión preliminar. Las versiones preliminares están a su disposición con la condición de que acepte los [términos de uso adicionales](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Es posible que algunos de los aspectos de esta característica cambien antes de ofrecer disponibilidad general.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción a Azure, puede crear una [cuenta gratuita](https://azure.microsoft.com/free).
* [La CLI de Azure instalada][azure-cli-installed].
* [Helm 3 instalado][helm-installed].
* Una cuenta de GitHub con [acciones de GitHub habilitadas][github-actions-beta-signup].
* La [aplicación de ejemplo Bike Sharing de Azure Dev Spaces](https://github.com/Azure/dev-spaces/tree/master/samples/BikeSharingApp/README.md) que se ejecuta en un clúster de AKS.

## <a name="create-an-azure-container-registry"></a>Creación de una instancia de Azure Container Registry

Crear una instancia de Azure Container Registry (ACR):

```azurecli
az acr create --resource-group MyResourceGroup --name <acrName> --sku Basic
```

> [!IMPORTANT]
> El nombre de la instancia de ACR debe ser único dentro de Azure y contener entre 5 y 50 caracteres alfanuméricos. Todas las letras que use deben estar en minúsculas.

Guarde el valor *loginServer* de la salida, ya que se usará en un paso posterior.

## <a name="create-a-service-principal-for-authentication"></a>Creación de una entidad de servicio para la autenticación

Use [az ad sp crear-de-rbac][az-ad-sp-create-for-rbac] para crear una entidad de servicio. Por ejemplo:

```azurecli
az ad sp create-for-rbac --sdk-auth --skip-assignment
```

Guarde la salida de JSON porque se usará en un paso posterior.

Use [az aks show][az-aks-show] para mostrar el *identificador* del clúster de AKS:

```azurecli
az aks show -g MyResourceGroup -n MyAKS  --query id
```

Use [az aks show][az-acr-show] para mostrar el *identificador* de la instancia de ACR:

```azurecli
az acr show --name <acrName> --query id
```

Use [az role assignment create][az-role-assignment-create] para conceder acceso de *colaborador* al clúster de AKS y acceso a *AcrPush* a la instancia de ACR.

```azurecli
az role assignment create --assignee <ClientId> --scope <AKSId> --role Contributor
az role assignment create --assignee <ClientId>  --scope <ACRId> --role AcrPush
```

> [!IMPORTANT]
> Debe ser propietario del clúster de AKS y la instancia de ACR para poder conceder a la entidad de servicio acceso a esos recursos.

## <a name="configure-your-github-action"></a>Configuración de la acción de GitHub

> [!IMPORTANT]
> Debe tener habilitadas las acciones de GitHub para el repositorio. Para habilitar las acciones de GitHub para el repositorio, vaya al repositorio en GitHub, haga clic en la pestaña Acciones y habilite las acciones para este repositorio.

Navegue al repositorio bifurcado y haga clic en *Configuración*. Haga clic en *Secretos* en la barra lateral izquierda. Haga clic en *Agregar un nuevo secreto* para agregar cada secreto nuevo a continuación:

1. *AZURE_CREDENTIALS*: toda la salida de la creación de la entidad de servicio.
1. *RESOURCE_GROUP*: el grupo de recursos para el clúster de AKS, que en este ejemplo es *MyResourceGroup*.
1. *CLUSTER_NAME*: el nombre del clúster de AKS, que en este ejemplo es *MyAKS*.
1. *CONTAINER_REGISTRY*: el *loginServer* para la instancia de ACR.
1. *HOST*: el host del espacio de desarrollo, que adopta la forma *<MASTER_SPACE>.<APP_NAME>.<HOST_SUFFIX>* , y en este ejemplo es *dev.bikesharingweb.fedcab0987.eus.azds.io*.
1. *IMAGE_PULL_SECRET*: el nombre del secreto que desea usar, por ejemplo *demo-secret*.
1. *MASTER_SPACE*: el nombre del espacio de desarrollo primario, que en este ejemplo es *dev*.
1. *REGISTRY_USERNAME*: el *clientId* de la salida de JSON en la creación de la entidad de servicio.
1. *REGISTRY_PASSWORD*: el *clientSecret* de la salida de JSON en la creación de la entidad de servicio.

> [!NOTE]
> Todos estos secretos los usa la acción de GitHub y se configuran en [.github/workflows/bikes.yml][github-action-yaml].

De manera opcional, si desea actualizar el espacio maestro después de la combinación de la solicitud de incorporación de cambios, agregue el secreto *GATEWAY_HOST*, que toma la forma *<ESPACIO_MAESTRO>. Gateway. <SUFIJO_DE_HOST>* , que en este ejemplo es *dev.gateway.fedcab0987.eus.azds.io*. Una vez que combine los cambios en la rama maestra de la bifurcación, se ejecutará otra acción para volver a compilar y ejecutar toda la aplicación en el espacio de desarrollo maestro. En este ejemplo, el espacio maestro es *dev*. Esta acción se configura en [.github/workflows/bikesharing.yml][github-action-bikesharing-yaml].

Además, si desea que los cambios de la solicitud de incorporación de cambios se ejecuten en un espacio terciario, actualice los secretos *MASTER_SPACE* y *HOST*. Por ejemplo, si la aplicación se ejecuta en *dev* con un espacio secundario *dev/azureuser1*, para que la solicitud de incorporación de cambios se ejecute en un espacio secundario de *dev/azureuser1*:

* Actualice *MASTER_SPACE* al espacio secundario que desee como espacio primario, en este ejemplo *azureuser1*.
* Actualice *HOST* a *<GRANDPARENT_SPACE>.<APP_NAME>.<HOST_SUFFIX>* , en este ejemplo *dev.bikesharingweb.fedcab0987.eus.azds.io*.

## <a name="create-a-new-branch-for-code-changes"></a>Crear una nueva rama para los cambios de código

Vaya a `BikeSharingApp/` y cree una nueva rama llamada *bike-images*.

```cmd
cd dev-spaces/samples/BikeSharingApp/
git checkout -b bike-images
```

Edite [Bikes/server.js][bikes-server-js] para quitar las líneas 232 y 233:

```javascript
    // Hard code image url *FIX ME*
    theBike.imageUrl = "/static/logo.svg";
```

Ahora, la sección debe tener un aspecto similar al siguiente:

```javascript
    var theBike = result;
    theBike.id = theBike._id;
    delete theBike._id;
```

Guarde el archivo y, después, use `git add` y `git commit` para almacenar provisionalmente los cambios.

```cmd
git add Bikes/server.js 
git commit -m "Removing hard coded imageUrl from /bikes/:id route"
```

## <a name="push-your-changes"></a>Enviar los cambios

Use `git push` para enviar los cambios de la nueva rama al repositorio bifurcado:

```cmd
git push origin bike-images
```

Una vez completado el envío de cambios, vaya al repositorio bifurcado de GitHub y cree una solicitud de incorporación de cambios con la rama *master* del repositorio bifurcado como rama base en lugar de la rama *bike-images*.

Una vez abierta la solicitud de incorporación de cambios, vaya a la pestaña *Acciones*. Compruebe que se haya iniciado una nueva acción y se esté creando el servicio *Bikes*.

## <a name="view-the-child-space-with-your-changes"></a>Ver el espacio secundario con los cambios

Una vez completada la acción, verá un comentario con una dirección URL para el nuevo espacio secundario basado en los cambios de la solicitud de incorporación de cambios.

> [!div class="mx-imgBorder"]
> ![URL de la acción de GitHub](../media/github-actions/github-action-url.png)

Para navegar al servicio *bikesharingweb*, abra la dirección URL pública en el comentario. Seleccione *Aurelia Briggs (customer)* (Aurelia Briggs [cliente]) como usuario y, después, seleccione una bicicleta para alquilar. Compruebe que ya no ve la imagen de marcador de posición de la bicicleta.

Si combina los cambios en la rama *master* de la bifurcación, se ejecutará otra acción para volver a compilar y ejecutar toda la aplicación en el espacio de desarrollo primario. En este ejemplo, el espacio primario es *dev*. Esta acción se configura en [.github/workflows/bikesharing.yml][github-action-bikesharing-yaml].

## <a name="clean-up-your-azure-resources"></a>Limpieza de los recursos de Azure

```azurecli
az group delete --name MyResourceGroup --yes --no-wait
```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre cómo funciona Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Funcionamiento de Azure Dev Spaces](../how-dev-spaces-works.md)

[azure-cli-installed]: /cli/azure/install-azure-cli?view=azure-cli-latest
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-aks-show]: /cli/azure/aks?view=azure-cli-latest#az-aks-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[bikes-server-js]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/Bikes/server.js#L232-L233
[bike-sharing-gh]: https://github.com/Azure/dev-spaces/
[bike-sharing-values-yaml]: https://github.com/Azure/dev-spaces/blob/master/samples/BikeSharingApp/charts/values.yaml
[github-actions-beta-signup]: https://github.com/features/actions
[github-action-yaml]: https://github.com/Azure/dev-spaces/blob/master/.github/workflows/bikes.yml
[github-action-bikesharing-yaml]: https://github.com/Azure/dev-spaces/blob/master/.github/workflows/bikesharing.yml
[helm-installed]: https://helm.sh/docs/intro/install/
[supported-regions]: https://azure.microsoft.com/global-infrastructure/services/?products=kubernetes-service
[sp-acr]: ../../container-registry/container-registry-auth-service-principal.md
[sp-aks]: ../../aks/kubernetes-service-principal.md
