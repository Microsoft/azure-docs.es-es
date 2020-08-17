---
title: Creación de nodos virtuales mediante la CLI de Azure
titleSuffix: Azure Kubernetes Service
description: Aprenda a usar la CLI de Azure para crear un clúster de Azure Kubernetes Service (AKS) que usa los nodos virtuales para ejecutar pods.
services: container-service
ms.topic: conceptual
ms.date: 05/06/2019
ms.custom: references_regions
ms.openlocfilehash: 1e62af4f2ab8233125777bf6edf713758e4f2ec7
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543085"
---
# <a name="create-and-configure-an-azure-kubernetes-services-aks-cluster-to-use-virtual-nodes-using-the-azure-cli"></a>Creación y configuración de un clúster de Azure Kubernetes Service (AKS) para usar nodos virtuales mediante la CLI de Azure

Para escalar rápidamente las cargas de trabajo de aplicaciones en un clúster de Azure Kubernetes Service (AKS), puede usar nodos virtuales. Con los nodos virtuales, tiene un aprovisionamiento rápido de pods y solo paga por segundo para el tiempo de ejecución. No es necesario que espere a que el escalador automático del clúster de Kubernetes implemente nodos de proceso de máquina virtual para ejecutar los pods adicionales. Los nodos virtuales solo son compatibles con los nodos y pods de Linux.

En este artículo se muestra cómo crear y configurar los recursos de red virtual y un clúster de AKS y cómo habilitar después nodos virtuales.

## <a name="before-you-begin"></a>Antes de empezar

Los nodos virtuales permiten la comunicación de red entre los pods que se ejecutan en Azure Container Instances (ACI) y el clúster de AKS. Para proporcionar esta comunicación, se crea una subred de red virtual y se asignan permisos delegados. Los nodos virtuales solo funcionan con clústeres de AKS creados mediante redes *avanzadas*. De forma predeterminada, los clústeres de AKS se crean con redes *básicas*. En este artículo se explica cómo crear una red virtual y subredes y, después, cómo implementar un clúster de AKS que usa redes avanzadas.

Si no ha utilizado anteriormente ACI, registre el proveedor de servicio con su suscripción. Puede comprobar el estado de registro del proveedor de ACI mediante el comando [az provider list][az-provider-list], tal como se muestra en el siguiente ejemplo:

```azurecli-interactive
az provider list --query "[?contains(namespace,'Microsoft.ContainerInstance')]" -o table
```

El proveedor *Microsoft.ContainerInstance* debería notificar como *Registrado*, tal como se muestra en el siguiente ejemplo de salida:

```output
Namespace                    RegistrationState    RegistrationPolicy
---------------------------  -------------------  --------------------
Microsoft.ContainerInstance  Registered           RegistrationRequired
```

Si el proveedor se muestra como *NotRegistered*, registre el proveedor con el comando [az provider register][az-provider-register] tal como se muestra en el siguiente ejemplo:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerInstance
```

## <a name="regional-availability"></a>Disponibilidad regional

Se admiten las siguientes regiones para las implementaciones de nodos virtuales:

* Este de Australia (australiaeast)
* Centro de EE. UU. (centralus)
* Este de EE. UU. (eastus)
* Este de EE. UU. 2 (eastus2)
* Japón Oriental (japaneast)
* Norte de Europa (northeurope)
* Sudeste de Asia (southeastasia)
* Centro-oeste de EE. UU. (westcentralus)
* Oeste de Europa (westeurope)
* Oeste de EE. UU. (westus)
* Oeste de EE. UU. 2 (westus2)

## <a name="known-limitations"></a>Restricciones conocidas
La funcionalidad de nodos virtuales es muy dependiente del conjunto de características de ACI. Además de las [cuotas y los límites de Azure Container Instances](../container-instances/container-instances-quotas.md), los siguientes escenarios no se admiten aún con los nodos virtuales:

* Uso de entidad de servicio para extraer imágenes de ACR. La [solución alternativa](https://github.com/virtual-kubelet/azure-aci/blob/master/README.md#private-registry) consiste en usar [secretos de Kubernetes](https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#create-a-secret-by-providing-credentials-on-the-command-line)
* [Limitaciones de la red virtual](../container-instances/container-instances-vnet.md) entre las que se incluyen el emparejamiento de redes virtuales, las directivas de red de Kubernetes y el tráfico saliente a Internet con grupos de seguridad de red.
* Iniciar contenedores
* [Hospedaje de alias](https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/)
* [Argumentos](../container-instances/container-instances-exec.md#restrictions) para la ejecución en ACI
* [DaemonSets](concepts-clusters-workloads.md#statefulsets-and-daemonsets) no implementará los pods en el nodo virtual
* Los nodos virtuales admiten la programación de pods de Linux. Puede instalar manualmente el proveedor de código abierto [ACI de Kubelet virtual](https://github.com/virtual-kubelet/azure-aci) para programar contenedores de Windows Server en ACI.
* Los nodos virtuales necesitan clústeres de AKS con redes de Azure CNI

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta.

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/bash](https://shell.azure.com/bash) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

Si prefiere instalar y usar la CLI en un entorno local, para este artículo se requiere la versión 2.0.49 de la CLI de Azure o una versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos de Azure es un grupo lógico en el que se implementan y administran recursos de Azure. Para crear un grupo de recursos, use el comando [az group create][az-group-create]. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *westus*.

```azurecli-interactive
az group create --name myResourceGroup --location westus
```

## <a name="create-a-virtual-network"></a>Creación de una red virtual

Cree una red virtual con el comando [az network vnet create][az-network-vnet-create]. En el ejemplo siguiente se crea una red virtual denominada *myVnet* con un prefijo de dirección de *10.0.0.0/8* y una subred llamada *myAKSSubnet*. El valor predeterminado del prefijo de la dirección de esta subred es *10.240.0.0/16*:

```azurecli-interactive
az network vnet create \
    --resource-group myResourceGroup \
    --name myVnet \
    --address-prefixes 10.0.0.0/8 \
    --subnet-name myAKSSubnet \
    --subnet-prefix 10.240.0.0/16
```

Ahora cree una subred adicional para los nodos virtuales mediante el comando [az network vnet subnet create][az-network-vnet-subnet-create]. En el ejemplo siguiente se crea una subred llamada *myVirtualNodeSubnet* con el prefijo de dirección *10.241.0.0/16*.

```azurecli-interactive
az network vnet subnet create \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name myVirtualNodeSubnet \
    --address-prefixes 10.241.0.0/16
```

## <a name="create-a-service-principal-or-use-a-managed-identity"></a>Creación de una entidad de servicio o uso de una identidad administrada

Para permitir que un clúster de AKS interactúe con otros recursos de Azure, se usa una entidad de servicio de Azure Active Directory. Esta entidad de servicio puede crearse automáticamente mediante la CLI de Azure o el portal, o puede crear una previamente y asignar permisos adicionales. También puede usar una identidad administrada para los permisos en lugar de una entidad de servicio. Para más información, consulte [Uso de identidades administradas](use-managed-identity.md).

Cree una entidad de servicio mediante el comando [az ad sp create-for-rbac][az-ad-sp-create-for-rbac]. El parámetro `--skip-assignment` impide que se asignen permisos adicionales.

```azurecli-interactive
az ad sp create-for-rbac --skip-assignment
```

La salida es similar a la del ejemplo siguiente:

```output
{
  "appId": "bef76eb3-d743-4a97-9534-03e9388811fc",
  "displayName": "azure-cli-2018-11-21-18-42-00",
  "name": "http://azure-cli-2018-11-21-18-42-00",
  "password": "1d257915-8714-4ce7-a7fb-0e5a5411df7f",
  "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db48"
}
```

Anote el valor de *appId* y *password*. Estos valores se usan en los pasos siguientes.

## <a name="assign-permissions-to-the-virtual-network"></a>Asignación de permisos a la red virtual

Para permitir que el clúster use y administre la red virtual, debe conceder a la entidad de servicio de AKS los derechos apropiados para usar los recursos de red.

En primer lugar, obtenga el identificador de recurso de red virtual mediante [az network vnet show][az-network-vnet-show]:

```azurecli-interactive
az network vnet show --resource-group myResourceGroup --name myVnet --query id -o tsv
```

Para conceder el acceso correcto al clúster de AKS para usar la red virtual, cree una asignación de roles mediante el comando [az role assignment create][az-role-assignment-create]. Reemplace `<appId`> y `<vnetId>` por los valores recopilados en los dos pasos anteriores.

```azurecli-interactive
az role assignment create --assignee <appId> --scope <vnetId> --role Contributor
```

## <a name="create-an-aks-cluster"></a>Creación de un clúster de AKS

Implemente el clúster de AKS en la subred de AKS creada en un paso anterior. Obtenga el identificador de esta subred mediante [az network vnet subnet show][az-network-vnet-subnet-show]:

```azurecli-interactive
az network vnet subnet show --resource-group myResourceGroup --vnet-name myVnet --name myAKSSubnet --query id -o tsv
```

Use el comando [az aks create][az-aks-create] para crear un clúster de AKS. En el siguiente ejemplo se crea un clúster denominado *myAKSCluster* con un nodo. Reemplace `<subnetId>` con el identificador obtenido en el paso anterior y, a continuación, `<appId>` y `<password>` con los valores recopilados en la sección anterior.

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-count 1 \
    --network-plugin azure \
    --service-cidr 10.0.0.0/16 \
    --dns-service-ip 10.0.0.10 \
    --docker-bridge-address 172.17.0.1/16 \
    --vnet-subnet-id <subnetId> \
    --service-principal <appId> \
    --client-secret <password>
```

En unos minutos, terminará de ejecutarse el comando, que devuelve información con formato JSON sobre el clúster.

## <a name="enable-virtual-nodes-addon"></a>Habilitar complemento de nodos virtuales

Para habilitar los nodos virtuales, ahora use el comando [az aks enable-addons][az-aks-enable-addons]. En el ejemplo siguiente se usa la subred denominada *myVirtualNodeSubnet* creada en un paso anterior:

```azurecli-interactive
az aks enable-addons \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --addons virtual-node \
    --subnet-name myVirtualNodeSubnet
```

## <a name="connect-to-the-cluster"></a>Conectarse al clúster

Para configurar `kubectl` para conectarse a su clúster de Kubernetes, use el comando [az aks get-credentials][az-aks-get-credentials]. Con este paso se descargan las credenciales y se configura la CLI de Kubernetes para usarlas.

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Para comprobar la conexión al clúster, use el comando [kubectl get][kubectl-get] para devolver una lista de los nodos del clúster.

```console
kubectl get nodes
```

La salida de ejemplo siguiente muestra el nodo de máquina virtual único creado y luego el nodo virtual para Linux, *virtual-node-aci-linux*:

```output
NAME                          STATUS    ROLES     AGE       VERSION
virtual-node-aci-linux        Ready     agent     28m       v1.11.2
aks-agentpool-14693408-0      Ready     agent     32m       v1.11.2
```

## <a name="deploy-a-sample-app"></a>Implementación de una aplicación de ejemplo

Cree un archivo denominado `virtual-node.yaml` y cópielo en el siguiente código YAML. Para programar el contenedor en el nodo, se definen [nodeSelector][node-selector] y [toleration][toleration].

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aci-helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: aci-helloworld
  template:
    metadata:
      labels:
        app: aci-helloworld
    spec:
      containers:
      - name: aci-helloworld
        image: microsoft/aci-helloworld
        ports:
        - containerPort: 80
      nodeSelector:
        kubernetes.io/role: agent
        beta.kubernetes.io/os: linux
        type: virtual-kubelet
      tolerations:
      - key: virtual-kubelet.io/provider
        operator: Exists
      - key: azure.com/aci
        effect: NoSchedule
```

Ejecute la aplicación con el comando [kubectl apply][kubectl-apply].

```console
kubectl apply -f virtual-node.yaml
```

Use el comando [kubectl get pods][kubectl-get] con el argumento `-o wide` para generar una lista de los pods y el nodo programado. Observe que el pod `aci-helloworld` se ha programado en el nodo `virtual-node-aci-linux`.

```console
kubectl get pods -o wide
```

```output
NAME                            READY     STATUS    RESTARTS   AGE       IP           NODE
aci-helloworld-9b55975f-bnmfl   1/1       Running   0          4m        10.241.0.4   virtual-node-aci-linux
```

Se asigna una dirección IP interna al pod de la subred de red virtual de Azure delegada para su uso con los nodos virtuales.

> [!NOTE]
> Si usa imágenes almacenadas en Azure Container Registry, [configure y use un secreto de Kubernetes][acr-aks-secrets]. Una limitación actual de los nodos virtuales es que no se puede usar la autenticación de entidad de servicio de Azure AD integrada. Si no usa un secreto, los pods programados en los nodos virtuales no se pueden iniciar y se notifica el error `HTTP response status code 400 error code "InaccessibleImage"`.

## <a name="test-the-virtual-node-pod"></a>Prueba del pod del nodo virtual

Para probar el pod que se ejecuta en el nodo virtual, vaya a la aplicación de demostración con un cliente web. Como se asigna una dirección IP interna al pod, puede probar rápidamente esta conectividad desde otro pod en el clúster de AKS. Cree un pod de prueba y asóciele una sesión de terminal:

```console
kubectl run -it --rm testvk --image=debian
```

Instale `curl` en el pod mediante `apt-get`:

```console
apt-get update && apt-get install -y curl
```

Ahora acceda a la dirección de su pod mediante `curl`, como *http://10.241.0.4* . Proporcione su propia dirección IP interna mostrada en el comando `kubectl get pods` anterior:

```console
curl -L http://10.241.0.4
```

Se muestra la aplicación de demostración, tal como se muestra en la siguiente salida de ejemplo reducida:

```output
<html>
<head>
  <title>Welcome to Azure Container Instances!</title>
</head>
[...]
```

Cierre la sesión de terminal en su pod de prueba con `exit`. Cuando la sesión finaliza, el pod se elimina.

## <a name="remove-virtual-nodes"></a>Quitar nodos virtuales

Si ya no desea usar los nodos virtuales, puede deshabilitarlos mediante el comando [az aks disable-addons][az aks disable-addons]. 

Si es necesario, vaya a [https://shell.azure.com](https://shell.azure.com) para abrir Azure Cloud Shell en el explorador.

En primer lugar, elimine el pod `aci-helloworld` que se ejecuta en el nodo virtual:

```console
kubectl delete -f virtual-node.yaml
```

En el comando de ejemplo siguiente se deshabilitan los nodos virtuales de Linux:

```azurecli-interactive
az aks disable-addons --resource-group myResourceGroup --name myAKSCluster --addons virtual-node
```

Ahora, quite los recursos de red virtual y el grupo de recursos:

```azurecli-interactive
# Change the name of your resource group, cluster and network resources as needed
RES_GROUP=myResourceGroup
AKS_CLUSTER=myAKScluster
AKS_VNET=myVnet
AKS_SUBNET=myVirtualNodeSubnet

# Get AKS node resource group
NODE_RES_GROUP=$(az aks show --resource-group $RES_GROUP --name $AKS_CLUSTER --query nodeResourceGroup --output tsv)

# Get network profile ID
NETWORK_PROFILE_ID=$(az network profile list --resource-group $NODE_RES_GROUP --query [0].id --output tsv)

# Delete the network profile
az network profile delete --id $NETWORK_PROFILE_ID -y

# Delete the subnet delegation to Azure Container Instances
az network vnet subnet update --resource-group $RES_GROUP --vnet-name $AKS_VNET --name $AKS_SUBNET --remove delegations 0
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo, se programó un pod en el nodo virtual y se le asignó una dirección IP privada o interna. En su lugar, podría crear una implementación de servicio y enrutar el tráfico a su pod a través de un equilibrador de carga o controlador de entrada. Para más información, consulte [Creación de un controlador de entrada en Azure Kubernetes Service (AKS)][aks-basic-ingress].

Los nodos virtuales suelen ser un componente de una solución de escalado en AKS. Para más información sobre soluciones de escalado, consulte los siguientes artículos:

- [Escalador horizontal automático de pods de Kubernetes][aks-hpa]
- [Escalador automático de clústeres de Kubernetes][aks-cluster-autoscaler]
- [Ejemplo de escalador automático para nodos virtuales][virtual-node-autoscale]
- [Más información sobre la biblioteca virtual de código abierto de Kubelet][virtual-kubelet-repo]

<!-- LINKS - external -->
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[node-selector]:https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[toleration]: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
[aks-github]: https://github.com/azure/aks/issues
[virtual-node-autoscale]: https://github.com/Azure-Samples/virtual-node-autoscale
[virtual-kubelet-repo]: https://github.com/virtual-kubelet/virtual-kubelet
[acr-aks-secrets]: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

<!-- LINKS - internal -->
[azure-cli-install]: /cli/azure/install-azure-cli
[az-group-create]: /cli/azure/group#az-group-create
[az-network-vnet-create]: /cli/azure/network/vnet#az-network-vnet-create
[az-network-vnet-subnet-create]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-create
[az-ad-sp-create-for-rbac]: /cli/azure/ad/sp#az-ad-sp-create-for-rbac
[az-network-vnet-show]: /cli/azure/network/vnet#az-network-vnet-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-network-vnet-subnet-show]: /cli/azure/network/vnet/subnet#az-network-vnet-subnet-show
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-enable-addons]: /cli/azure/aks#az-aks-enable-addons
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az aks disable-addons]: /cli/azure/aks#az-aks-disable-addons
[aks-hpa]: tutorial-kubernetes-scale.md
[aks-cluster-autoscaler]: ./cluster-autoscaler.md
[aks-basic-ingress]: ingress-basic.md
[az-provider-list]: /cli/azure/provider#az-provider-list
[az-provider-register]: /cli/azure/provider#az-provider-register
