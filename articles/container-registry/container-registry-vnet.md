---
title: Restricción del acceso mediante puntos de conexión de servicio
description: Restricción del acceso a un registro de contenedor de Azure mediante un punto de conexión de servicio en una red virtual de Azure. El acceso de punto de conexión de servicio es una característica del nivel de servicio Premium.
ms.topic: article
ms.date: 05/04/2020
ms.openlocfilehash: 1fc8d54d677112a9c934f9079e953a7389939bde
ms.sourcegitcommit: de2750163a601aae0c28506ba32be067e0068c0c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/04/2020
ms.locfileid: "89488681"
---
# <a name="restrict-access-to-a-container-registry-using-a-service-endpoint-in-an-azure-virtual-network"></a>Restricción del acceso a un registro de contenedor mediante un punto de conexión de servicio en una red virtual de Azure

[Azure Virtual Network](../virtual-network/virtual-networks-overview.md) proporciona acceso de red seguro y privado a los recursos locales y de Azure. Un [punto de conexión de servicio](../virtual-network/virtual-network-service-endpoints-overview.md) le permite proteger la dirección IP pública del registro de contenedor solo en la red virtual. Este punto de conexión proporciona al tráfico una ruta óptima hasta el recurso a través de una red troncal de Azure. Las identidades de la red virtual y la subred también se transmiten con cada solicitud.

En este artículo se muestra cómo configurar un punto de conexión de servicio de registro de contenedor (versión preliminar) en una red virtual. 

> [!IMPORTANT]
> Azure Container Registry admite ahora [Azure Private Link](container-registry-private-link.md), lo que permite colocar los puntos de conexión privados desde una red virtual en un registro. Se puede acceder a los puntos de conexión privados desde dentro de la red virtual mediante direcciones IP privadas. Se recomienda usar puntos de conexión privados en lugar de puntos de conexión de servicio en la mayoría de los escenarios de red.

La configuración de un punto de conexión de servicio de registro está disponible en el nivel de servicio **Premium** del registro de contenedor. Para obtener información sobre los límites y los niveles de servicio de los registros, consulte [SKU de Azure Container Registry](container-registry-skus.md).

## <a name="preview-limitations"></a>Limitaciones de vista previa

* El futuro desarrollo de puntos de conexión de servicio para Azure Container Registry no está planeado actualmente. En su lugar, se recomienda usar [puntos de conexión privados](container-registry-private-link.md).
* No se puede usar Azure Portal para configurar puntos de conexión de servicio en un registro.
* Para acceder a un registro de contenedor en un punto de conexión de servicio, solo se pueden usar como host un clúster de [Azure Kubernetes Service](../aks/intro-kubernetes.md) o una [máquina virtual](../virtual-machines/linux/overview.md) de Azure. *No se admiten otros servicios de Azure, como Azure Container Instances.*
* Cada registro admite un máximo de 100 reglas de acceso de red.
* Los puntos de conexión de servicio para Azure Container Registry no se admiten en la nube de Azure US Government ni en la nube de Azure China.

[!INCLUDE [container-registry-scanning-limitation](../../includes/container-registry-scanning-limitation.md)]

## <a name="prerequisites"></a>Requisitos previos

* Para usar los pasos de la CLI de Azure de este artículo, se requieren la versión 2.0.58 de la CLI de Azure, o cualquier versión posterior. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli].

* Si aún no tiene un registro de contenedor, créelo (se requiere un nivel Premium) e inserte una imagen de ejemplo, como `hello-world`, desde Docker Hub. Por ejemplo, use [Azure Portal][quickstart-portal] o la [CLI de Azure][quickstart-cli] para crear un registro. 

* Si quiere restringir el acceso al registro mediante un punto de conexión de servicio en otra suscripción de Azure, registre el proveedor de recursos para Azure Container Registry en esa suscripción. Por ejemplo:

  ```azurecli
  az account set --subscription <Name or ID of subscription of virtual network>

  az provider register --namespace Microsoft.ContainerRegistry
  ``` 

[!INCLUDE [Set up Docker-enabled VM](../../includes/container-registry-docker-vm-setup.md)]

## <a name="configure-network-access-for-registry"></a>Configuración el acceso de red al registro

En esta sección se configura el registro de contenedor para permitir el acceso desde una subred de una red virtual de Azure. Se proporcionan los pasos equivalentes si se usa mediante la CLI de Azure y Azure Portal.

### <a name="allow-access-from-a-virtual-network---cli"></a>Concesión de acceso desde una red virtual: CLI

#### <a name="add-a-service-endpoint-to-a-subnet"></a>Incorporación de un punto de conexión de servicio a una subred

Al crear una máquina virtual, Azure crea de manera predeterminada crea una red virtual en el mismo grupo de recursos. El nombre de dicha red virtual se basa en el nombre de la máquina virtual. Por ejemplo, si asigna a la máquina virtual el nombre *myDockerVM*, el nombre predeterminado de la red virtual es *myDockerVMVNET*, con una subred llamada *myDockerVMSubnet*. Compruébelo en Azure Portal o mediante el comando [az network vnet list][az-network-vnet-list]:

```azurecli
az network vnet list \
  --resource-group myResourceGroup \
  --query "[].{Name: name, Subnet: subnets[0].name}"
```

Salida:

```console
[
  {
    "Name": "myDockerVMVNET",
    "Subnet": "myDockerVMSubnet"
  }
]
```

Use el comando [az network vnet subnet update][az-network-vnet-subnet-update] para agregar un punto de conexión de servicio **Microsoft.ContainerRegistry** a la subred. Sustituya los nombres de la red virtual y de la subred en el siguiente comando:

```azurecli
az network vnet subnet update \
  --name myDockerVMSubnet \
  --vnet-name myDockerVMVNET \
  --resource-group myResourceGroup \
  --service-endpoints Microsoft.ContainerRegistry
```

Use el comando [az network vnet subnet show][az-network-vnet-subnet-show] para recuperar el identificador de recurso de la subred. Lo necesitará en un paso posterior para configurar una regla de acceso a la red.

```azurecli
az network vnet subnet show \
  --name myDockerVMSubnet \
  --vnet-name myDockerVMVNET \
  --resource-group myResourceGroup \
  --query "id"
  --output tsv
``` 

Salida:

```
/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myDockerVMVNET/subnets/myDockerVMSubnet
```

#### <a name="change-default-network-access-to-registry"></a>Cambio del acceso de red predeterminado al registro

De manera predeterminada los registros de contenedor Azure permiten conexiones de hosts de cualquier red. Para limitar el acceso a una red seleccionada, cambie la acción predeterminada para denegar el acceso. Sustituya el nombre del registro en el siguiente comando [az acr update][az-acr-update]:

```azurecli
az acr update --name myContainerRegistry --default-action Deny
```

#### <a name="add-network-rule-to-registry"></a>Incorporación de una regla de red al registro

Use el comando [az acr network-rule add][az-acr-network-rule-add] para agregar una regla de red al registro que permita el acceso desde la subred de la máquina virtual. Sustituya el nombre del registro de contenedor y el identificador de recurso de la subred en el siguiente comando: 

 ```azurecli
az acr network-rule add \
  --name mycontainerregistry \
  --subnet <subnet-resource-id>
```

## <a name="verify-access-to-the-registry"></a>Comprobación del acceso al registro

Después de esperar unos minutos para que se actualice la configuración, compruebe que la máquina virtual puede acceder al registro de contenedor. Cree una conexión SSH con la máquina virtual y ejecute el comando [az acr login][az-acr-login] para iniciar sesión en el registro. 

```bash
az acr login --name mycontainerregistry
```

Puede realizar las operaciones de registro, como ejecutar `docker pull` para extraer una imagen de ejemplo del registro. Sustituya una imagen y el valor de la etiqueta pertinente para el registro, y anteponga el nombre del servidor de inicio de sesión del registro (todo en minúsculas):

```bash
docker pull mycontainerregistry.azurecr.io/hello-world:v1
``` 

Docker extrae correctamente la imagen a la máquina virtual.

En este ejemplo se muestra que puede acceder al registro de contenedor privado a través de la regla de acceso a la red. Sin embargo, no se puede acceder al registro desde un host de inicio de sesión que no tenga configurada una regla de acceso de red. Si intenta iniciar sesión desde otro host mediante el comando `az acr login` o el comando `docker login`, el resultado será similar al siguiente:

```Console
Error response from daemon: login attempt to https://xxxxxxx.azurecr.io/v2/ failed with status: 403 Forbidden
```

## <a name="restore-default-registry-access"></a>Restauración del acceso al registro predeterminado

Para restaurar el registro para permitir el acceso de forma predeterminada, quite todas las reglas de red configuradas. Luego, establezca la acción predeterminado en permitir el acceso. Se proporcionan los pasos equivalentes si se usa mediante la CLI de Azure y Azure Portal.

### <a name="restore-default-registry-access---cli"></a>Restauración del acceso al registro predeterminado: CLI

#### <a name="remove-network-rules"></a>Eliminación de reglas de red

Para ver una lista de reglas de red configuradas para el registro, ejecute el siguiente comando [az acr network-rule list][az-acr-network-rule-list]:

```azurecli
az acr network-rule list --name mycontainerregistry 
```

En cada regla que esté configurada, ejecute el comando [az acr network-rule remove][az-acr-network-rule-remove] para quitarla. Por ejemplo:

```azurecli
# Remove a rule that allows access for a subnet. Substitute the subnet resource ID.

az acr network-rule remove \
  --name mycontainerregistry \
  --subnet /subscriptions/ \
  xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Network/virtualNetworks/myDockerVMVNET/subnets/myDockerVMSubnet
```

#### <a name="allow-access"></a>Permitir acceso

Sustituya el nombre del registro en el siguiente comando [az acr update][az-acr-update]:
```azurecli
az acr update --name myContainerRegistry --default-action Allow
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha creado todos los recursos de Azure en el mismo grupo de recursos y ya no los necesita, puede eliminarlos con el comando [az group delete](/cli/azure/group):

```azurecli
az group delete --name myResourceGroup
```

Para limpiar los recursos en el portal, vaya al grupo de recursos myResourceGroup. Una vez que se ha cargado el grupo de recursos, haga clic en **Eliminar grupo de recursos** para quitar el grupo de recursos y los recursos almacenados en él.

## <a name="next-steps"></a>Pasos siguientes

* Para restringir el acceso a un registro mediante un punto de conexión privado de una red virtual, consulte [Configuración de Azure Private Link para un registro de contenedor de Azure](container-registry-private-link.md).
* Si necesita configurar reglas de acceso a un registro desde detrás de un firewall cliente, consulte [Configuración de reglas para acceder a un registro de contenedor de Azure desde detrás de un firewall](container-registry-firewall-access-rules.md).


<!-- IMAGES -->

[acr-subnet-service-endpoint]: ./media/container-registry-vnet/acr-subnet-service-endpoint.png


<!-- LINKS - External -->
[aci-helloworld]: https://hub.docker.com/r/microsoft/aci-helloworld/
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-acr-repository-show]: /cli/azure/acr/repository#az-acr-repository-show
[az-acr-repository-list]: /cli/azure/acr/repository#az-acr-repository-list
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-network-rule-add]: /cli/azure/acr/network-rule/#az-acr-network-rule-add
[az-acr-network-rule-remove]: /cli/azure/acr/network-rule/#az-acr-network-rule-remove
[az-acr-network-rule-list]: /cli/azure/acr/network-rule/#az-acr-network-rule-list
[az-acr-run]: /cli/azure/acr#az-acr-run
[az-acr-update]: /cli/azure/acr#az-acr-update
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-group-create]: /cli/azure/group
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-vm-create]: /cli/azure/vm#az-vm-create
[az-network-vnet-subnet-show]: /cli/azure/network/vnet/subnet/#az-network-vnet-subnet-show
[az-network-vnet-subnet-update]: /cli/azure/network/vnet/subnet/#az-network-vnet-subnet-update
[az-network-vnet-subnet-show]: /cli/azure/network/vnet/subnet/#az-network-vnet-subnet-show
[az-network-vnet-list]: /cli/azure/network/vnet/#az-network-vnet-list
[quickstart-portal]: container-registry-get-started-portal.md
[quickstart-cli]: container-registry-get-started-azure-cli.md
[azure-portal]: https://portal.azure.com
