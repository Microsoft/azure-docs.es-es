---
title: Implementación de un grupo de contenedores en Azure Virtual Network
description: Obtenga información sobre cómo implementar un grupo de contenedores en una instancia de Azure Virtual Network nueva o existente mediante la interfaz de la línea de comandos de Azure.
ms.topic: article
ms.date: 07/02/2020
ms.custom: devx-track-javascript
ms.openlocfilehash: c5eff486299a3974e8fb9b4b12d6bb74880c48d8
ms.sourcegitcommit: d0541eccc35549db6381fa762cd17bc8e72b3423
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/09/2020
ms.locfileid: "89569397"
---
# <a name="deploy-container-instances-into-an-azure-virtual-network"></a>Implementación de instancias de contenedor en una red virtual de Azure

[Azure Virtual Network](../virtual-network/virtual-networks-overview.md) proporciona acceso de red seguro y privado a los recursos locales y de Azure. Al implementar grupos de contenedores en una red virtual de Azure, los contenedores pueden comunicarse de forma segura con otros recursos de la red virtual.

En este artículo se muestra cómo usar el comando [az container create][az-container-create] en la CLI de Azure para implementar grupos de contenedores en una red virtual nueva o existente. 

Para conocer los escenarios y las limitaciones de red, consulte [Escenarios y recursos de redes virtuales para Azure Container Instances](container-instances-virtual-network-concepts.md).

> [!IMPORTANT]
> La implementación del grupo de contenedores en una red virtual está disponible con carácter general para contenedores de Linux en la mayoría de las regiones donde Azure Container Instances está disponible. Para obtener más información, consulte [Regiones y disponibilidad de recursos](container-instances-virtual-network-concepts.md#where-to-deploy). 

Los ejemplos que aparecen en este artículo están formateados para el shell de Bash. Si prefiere otro shell como PowerShell o el símbolo del sistema, ajuste los caracteres de continuación de línea según corresponda.


## <a name="deploy-to-new-virtual-network"></a>Implementación en una nueva red virtual

Para implementarlos en una nueva red virtual y hacer que Azure cree automáticamente los recursos de red, especifique lo siguiente al ejecutar [az container create][az-container-create]:

* Nombre de la red virtual
* Prefijo de dirección de red virtual en formato CIDR
* Nombre de subred
* Prefijo de dirección de subred en formato CIDR

Los prefijos de direcciones de subred y red virtual especifican los espacios de direcciones para la subred y la red virtual, respectivamente. Estos valores se representan en notación de Enrutamiento de interdominios sin clases (CIDR), por ejemplo `10.0.0.0/16`. Para más información sobre cómo trabajar con las subredes, consulte [Incorporación, cambio o eliminación de una subred de red virtual](../virtual-network/virtual-network-manage-subnet.md).

Cuando haya implementado el primer grupo de contenedores con este método, puede implementarlo en la misma subred; para ello, especifique los nombres de la red virtual y la subred, o el perfil de red que Azure crea automáticamente para usted. Dado que Azure delega la subred en Azure Container Instances, *solo* puede implementar grupos de contenedores en la subred.

### <a name="example"></a>Ejemplo

El comando [az container create][az-container-create] siguiente especifica valores para una nueva red virtual y una subred. Proporcione el nombre de un grupo de recursos creado en una región en la que haya [disponibles](container-instances-region-availability.md#availability---virtual-network-deployment) implementaciones de grupo de contenedores en una red virtual. Este comando implementa el contenedor [aci-helloworld][aci-helloworld] público de Microsoft que ejecuta un pequeño servidor web de Node.js que sirve una página web estática. En la siguiente sección, implementará un segundo grupo de contenedores en la misma subred y probará la comunicación entre las dos instancias de contenedor.

```azurecli
az container create \
  --name appcontainer \
  --resource-group myResourceGroup \
  --image mcr.microsoft.com/azuredocs/aci-helloworld \
  --vnet aci-vnet \
  --vnet-address-prefix 10.0.0.0/16 \
  --subnet aci-subnet \
  --subnet-address-prefix 10.0.0.0/24
```

Al implementar en una nueva red virtual mediante este método, la implementación puede tardar unos minutos mientras se crean los recursos de red. Tras la implementación inicial, las implementaciones de grupos de contenedores adicionales en la misma subred se completan más rápido.

## <a name="deploy-to-existing-virtual-network"></a>Implementación en una red virtual existente

Para implementar un grupo de contenedores en una red virtual existente:

1. Cree una subred en la red virtual existente, use una subred existente en la que ya se haya implementado un grupo de contenedores, o bien use una subred existente vacía de *todos* los demás recursos
1. Implemente un grupo de contenedores con [az container create][az-container-create] y especifique uno de los siguientes:
   * Nombre de red virtual y nombre de subred
   * El identificador de recurso de red virtual y el identificador de recurso de subred, lo permite usar una red virtual desde otro grupo de recursos
   * El identificador o el nombre del perfil de red, que puede obtener mediante el comando [az network profile list][az-network-profile-list]

### <a name="example"></a>Ejemplo

En el ejemplo siguiente se implementa un segundo grupo de contenedores en la misma subred creada anteriormente y se comprueba la comunicación entre las dos instancias de contenedor.

En primer lugar, obtenga la dirección IP del primer grupo de contenedores que implementó, *appcontainer*:

```azurecli
az container show --resource-group myResourceGroup \
  --name appcontainer \
  --query ipAddress.ip --output tsv
```

La salida muestra la dirección IP del grupo de contenedores en la subred privada. Por ejemplo:

```console
10.0.0.4
```

Ahora, establezca `CONTAINER_GROUP_IP` en la dirección IP que recuperó con el comando `az container show` y ejecute el siguiente comando `az container create`. Este segundo contenedor, *commchecker*, ejecuta una imagen basada en Linux de Alpine y `wget` con la dirección IP de la subred privada del primer grupo de contenedores.

```azurecli
CONTAINER_GROUP_IP=<container-group-IP-address>

az container create \
  --resource-group myResourceGroup \
  --name commchecker \
  --image alpine:3.5 \
  --command-line "wget $CONTAINER_GROUP_IP" \
  --restart-policy never \
  --vnet aci-vnet \
  --subnet aci-subnet
```

Una vez completada esta segunda implementación de contenedor, extraiga sus registros para que pueda ver la salida del comando `wget` que ejecutó:

```azurecli
az container logs --resource-group myResourceGroup --name commchecker
```

Si el segundo contenedor se comunicó correctamente con el primero, la salida es similar a la siguiente:

```console
Connecting to 10.0.0.4 (10.0.0.4:80)
index.html           100% |*******************************|  1663   0:00:00 ETA
```

La salida del registro debe mostrar que `wget` fue capaz de conectarse y descargar el archivo de índice del primer contenedor usando su dirección IP privada en la subred local. El tráfico de red entre los dos grupos de contenedores permanece dentro de la red virtual.

### <a name="example---yaml"></a>Ejemplo: YAML

También puede implementar un grupo de contenedores en una red virtual existente mediante un archivo YAML, una [plantilla de Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aci-vnet
) u otro método de programación como con el SDK de Python. 

Por ejemplo, si usa un archivo YAML, puede realizar la implementación en una red virtual con una subred delegada en Azure Container Instances. Especifique las propiedades siguientes:

* `ipAddress`: la configuración de direcciones IP privadas para el grupo de contenedores.
  * `ports`: los puertos que deben abrirse, si hay alguno.
  * `protocol`: el protocolo (TCP o UDP) para el puerto abierto.
* `networkProfile`: configuración de red para la red virtual y la subred.
  * `id`: el identificador de recurso del Administrador de recursos completo de `networkProfile`.

Para obtener el identificador del perfil de red, ejecute el comando [az network profile list][az-network-profile-list] y especifique el nombre del grupo de recursos que contiene la red virtual y la subred delegada.

``` azurecli
az network profile list --resource-group myResourceGroup \
  --query [0].id --output tsv
```

Salida del ejemplo:

```console
/subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkProfiles/aci-network-profile-aci-vnet-aci-subnet
```

Una vez que tenga el identificador del perfil de red, copie el siguiente YAML en un nuevo archivo denominado *vnet-deploy-aci.yaml*. En `networkProfile`, reemplace el valor de `id` con el identificador recién recuperado y, después, guarde el archivo. Este YAML crea un grupo de contenedores denominado *appcontaineryaml* en la red virtual.

```YAML
apiVersion: '2019-12-01'
location: westus
name: appcontaineryaml
properties:
  containers:
  - name: appcontaineryaml
    properties:
      image: mcr.microsoft.com/azuredocs/aci-helloworld
      ports:
      - port: 80
        protocol: TCP
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
  ipAddress:
    type: Private
    ports:
    - protocol: tcp
      port: '80'
  networkProfile:
    id: /subscriptions/<Subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Network/networkProfiles/aci-network-profile-aci-vnet-subnet
  osType: Linux
  restartPolicy: Always
tags: null
type: Microsoft.ContainerInstance/containerGroups
```

Implemente el grupo de contenedores con el comando [az container create][az-container-create] y especifique el nombre del archivo YAML para el parámetro `--file`:

```azurecli
az container create --resource-group myResourceGroup \
  --file vnet-deploy-aci.yaml
```

Una vez completada la implementación, ejecute el comando [az container show][az-container-show] para mostrar su estado. Salida del ejemplo:

```console
Name              ResourceGroup    Status    Image                                       IP:ports     Network    CPU/Memory       OsType    Location
----------------  ---------------  --------  ------------------------------------------  -----------  ---------  ---------------  --------  ----------
appcontaineryaml  myResourceGroup  Running   mcr.microsoft.com/azuredocs/aci-helloworld  10.0.0.5:80  Private    1.0 core/1.5 gb  Linux     westus
```

## <a name="clean-up-resources"></a>Limpieza de recursos

### <a name="delete-container-instances"></a>Eliminación de instancias de contenedor

Cuando termine de trabajar con las instancias de contenedor que creó, elimínelas con los siguientes comandos:

```azurecli
az container delete --resource-group myResourceGroup --name appcontainer -y
az container delete --resource-group myResourceGroup --name commchecker -y
az container delete --resource-group myResourceGroup --name appcontaineryaml -y
```

### <a name="delete-network-resources"></a>Eliminación de recursos de red

Actualmente, esta característica necesita varios comandos adicionales para eliminar los recursos de red que ha creado antes. Si ha usado los comandos de ejemplo en las secciones anteriores de este artículo para crear la red virtual y la subred, puede usar el siguiente script para eliminar esos recursos de red. El script presupone que el grupo de recursos contiene una sola red virtual con un único perfil de red.

Antes de ejecutar el script, establezca la variable `RES_GROUP` en el nombre del grupo de recursos que contenga la red virtual y la subred que se deben eliminar. Actualice el nombre de la red virtual si no usó el nombre `aci-vnet` que se sugirió anteriormente. El script tiene el formato adecuado para el shell de Bash. Si prefiere otro shell como PowerShell o el símbolo del sistema, deberá ajustar los descriptores de acceso y la asignación de variables según corresponda.

> [!WARNING]
> Este script elimina los recursos. Elimina la red virtual y todas las subredes que contiene. Asegúrese de que ya no necesita *ninguno* de los recursos en la red virtual, incluidas las subredes que contiene, antes de ejecutar este script. Una vez eliminado, **estos recursos son irrecuperables**.

```azurecli
# Replace <my-resource-group> with the name of your resource group
# Assumes one virtual network in resource group
RES_GROUP=<my-resource-group>

# Get network profile ID
# Assumes one profile in virtual network
NETWORK_PROFILE_ID=$(az network profile list --resource-group $RES_GROUP --query [0].id --output tsv)

# Delete the network profile
az network profile delete --id $NETWORK_PROFILE_ID -y

# Delete virtual network
az network vnet delete --resource-group $RES_GROUP --name aci-vnet
```

## <a name="next-steps"></a>Pasos siguientes

Para implementar una nueva red virtual, subred, perfil de red y grupo de contenedores mediante una plantilla de Resource Manager, consulte el artículo sobre la [creación de un grupo de contenedores de Azure con VNet](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aci-vnet
).

<!-- IMAGES -->
[aci-vnet-01]: ./media/container-instances-vnet/aci-vnet-01.png

<!-- LINKS - External -->
[aci-helloworld]: https://hub.docker.com/_/microsoft-azuredocs-aci-helloworld

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show
[az-network-vnet-create]: /cli/azure/network/vnet#az-network-vnet-create
[az-network-profile-list]: /cli/azure/network/profile#az-network-profile-list
