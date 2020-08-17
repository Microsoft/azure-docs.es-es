---
title: Creación de nodos virtuales mediante el portal en Azure Kubernetes Service (AKS)
description: Aprenda a usar el Azure Portal para crear un clúster de Azure Kubernetes Service (AKS) que usa los nodos virtuales para ejecutar pods.
services: container-service
ms.topic: conceptual
ms.date: 05/06/2019
ms.custom: references_regions
ms.openlocfilehash: 8748e4f78582ab133d7e527daba1c126dcb7e7e2
ms.sourcegitcommit: 8def3249f2c216d7b9d96b154eb096640221b6b9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87543714"
---
# <a name="create-and-configure-an-azure-kubernetes-services-aks-cluster-to-use-virtual-nodes-in-the-azure-portal"></a>Creación y configuración de un clúster de Azure Kubernetes Service (AKS) para usar nodos virtuales en Azure Portal

Para implementar rápidamente las cargas de trabajo en un clúster de Azure Kubernetes Service (AKS), puede usar nodos virtuales. Con los nodos virtuales, tiene un aprovisionamiento rápido de pods y solo paga por segundo para el tiempo de ejecución. En un escenario de escalado, no es necesario que espere a que el escalador automático del clúster de Kubernetes implemente nodos de proceso de máquina virtual para ejecutar los pods adicionales. Los nodos virtuales solo son compatibles con los nodos y pods de Linux.

En este artículo se muestra cómo crear y configurar los recursos de red virtual y un clúster de AKS con nodos virtuales habilitados.

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

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-an-aks-cluster"></a>Creación de un clúster de AKS

En la esquina superior izquierda de Azure Portal, seleccione **Crear un recurso** > **Kubernetes Service**.

En la página **Datos básicos**, configure las siguientes opciones:

- *DETALLES DEL PROYECTO*: seleccione una suscripción de Azure y, a continuación, seleccione o cree un grupo de recursos de Azure, como *myResourceGroup*. Escriba un **Nombre del clúster de Kubernetes**, como *myAKSCluster*.
- *DETALLES DEL CLÚSTER*: seleccione la región, la versión de Kubernetes y el prefijo del nombre DNS para el clúster de AKS.
- *GRUPO DE NODOS PRINCIPAL*: seleccione un tamaño de máquina virtual para los nodos de AKS. El tamaño de VM **no** puede cambiarse una vez que se ha implementado un clúster de AKS.
     - Seleccione el número de nodos que se van a implementar en el clúster. Para este artículo, establezca **Número de nodos** en *1*. El número de nodos **puede** ajustarse después de implementar el clúster.

Haga clic en **Siguiente: Escalado**.

En la página **Escalado**, seleccione *Habilitado* en **Nodos virtuales**.

![Creación del clúster de AKS y habilitación de los nodos virtuales](media/virtual-nodes-portal/enable-virtual-nodes.png)

De forma predeterminada, se crea una entidad de servicio de Azure Active Directory. Esta entidad de servicio se usa para la comunicación del clúster y la integración con otros servicios de Azure. También puede usar una identidad administrada para los permisos en lugar de una entidad de servicio. Para más información, consulte [Uso de identidades administradas](use-managed-identity.md).

El clúster también se configurado para redes avanzadas. Los nodos virtuales están configurados para usar su propia subred de red virtual de Azure. Esta subred tiene permisos delegados para conectarse a recursos de Azure entre el clúster de AKS. Si aún no tiene una subred delegada, Azure Portal crea y configura la red virtual de Azure y la subred para su uso con los nodos virtuales.

Seleccione **Revisar + crear**. Una vez completada la validación, seleccione **Crear**.

Se tardan unos minutos en crear el clúster de AKS y en prepararlo para usarlo.

## <a name="connect-to-the-cluster"></a>Conectarse al clúster

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. Para administrar un clúster de Kubernetes, use [kubectl][kubectl], el cliente de línea de comandos de Kubernetes. El cliente `kubectl` viene preinstalado en Azure Cloud Shell.

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/bash](https://shell.azure.com/bash) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

Use el comando [az aks get-credentials][az-aks-get-credentials] para configurar `kubectl` para conectarse a su clúster de Kubernetes. En el ejemplo siguiente se obtienen credenciales para el nombre de clúster *myAKSCluster* del grupo de recursos denominado *myResourceGroup*:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

Para comprobar la conexión al clúster, use el comando [kubectl get][kubectl-get] para devolver una lista de los nodos del clúster.

```console
kubectl get nodes
```

La salida de ejemplo siguiente muestra el nodo de máquina virtual único creado y luego el nodo virtual para Linux, *virtual-node-aci-linux*:

```output
NAME                           STATUS    ROLES     AGE       VERSION
virtual-node-aci-linux         Ready     agent     28m       v1.11.2
aks-agentpool-14693408-0       Ready     agent     32m       v1.11.2
```

## <a name="deploy-a-sample-app"></a>Implementación de una aplicación de ejemplo

En Azure Cloud Shell, cree un archivo denominado `virtual-node.yaml` y cópielo en el siguiente código YAML. Para programar el contenedor en el nodo, se definen [nodeSelector][node-selector] y [toleration][toleration]. Estas configuraciones permiten que el pod se programe en el nodo virtual y confirmen que la característica está habilitada correctamente.

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
```

Ejecute la aplicación con el comando [kubectl apply][kubectl-apply].

```azurecli-interactive
kubectl apply -f virtual-node.yaml
```

Use el comando [kubectl get pods][kubectl-get] con el argumento `-o wide` para generar una lista de los pods y el nodo programado. Observe que el pod `virtual-node-helloworld` se ha programado en el nodo `virtual-node-linux`.

```console
kubectl get pods -o wide
```

```output
NAME                                     READY     STATUS    RESTARTS   AGE       IP           NODE
virtual-node-helloworld-9b55975f-bnmfl   1/1       Running   0          4m        10.241.0.4   virtual-node-aci-linux
```

Se asigna una dirección IP interna al pod de la subred de red virtual de Azure delegada para su uso con los nodos virtuales.

> [!NOTE]
> Si usa imágenes almacenadas en Azure Container Registry, [configure y use un secreto de Kubernetes][acr-aks-secrets]. Una limitación actual de los nodos virtuales es que no se puede usar la autenticación de entidad de servicio de Azure AD integrada. Si no usa un secreto, los pods programados en los nodos virtuales no se pueden iniciar y se notifica el error `HTTP response status code 400 error code "InaccessibleImage"`.

## <a name="test-the-virtual-node-pod"></a>Prueba del pod del nodo virtual

Para probar el pod que se ejecuta en el nodo virtual, vaya a la aplicación de demostración con un cliente web. Como se asigna una dirección IP interna al pod, puede probar rápidamente esta conectividad desde otro pod en el clúster de AKS. Cree un pod de prueba y asóciele una sesión de terminal:

```console
kubectl run -it --rm virtual-node-test --image=debian
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

## <a name="next-steps"></a>Pasos siguientes

En este artículo, se programó un pod en el nodo virtual y se le asignó una dirección IP privada o interna. En su lugar, podría crear una implementación de servicio y enrutar el tráfico a su pod a través de un equilibrador de carga o controlador de entrada. Para más información, consulte [Creación de un controlador de entrada en Azure Kubernetes Service (AKS)][aks-basic-ingress].

Los nodos virtuales son un componente de una solución de escalado en AKS. Para más información sobre soluciones de escalado, consulte los siguientes artículos:

- [Escalador horizontal automático de pods de Kubernetes][aks-hpa]
- [Escalador automático de clústeres de Kubernetes][aks-cluster-autoscaler]
- [Ejemplo de escalador automático para nodos virtuales][virtual-node-autoscale]
- [Más información sobre la biblioteca virtual de código abierto de Kubelet][virtual-kubelet-repo]

<!-- LINKS - external -->
[kubectl]: https://kubernetes.io/docs/user-guide/kubectl/
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[node-selector]:https://kubernetes.io/docs/concepts/configuration/assign-pod-node/
[toleration]: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
[azure-cni]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[aks-github]: https://github.com/azure/aks/issues]
[virtual-node-autoscale]: https://github.com/Azure-Samples/virtual-node-autoscale
[virtual-kubelet-repo]: https://github.com/virtual-kubelet/virtual-kubelet
[acr-aks-secrets]: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/

<!-- LINKS - internal -->
[aks-network]: ./configure-azure-cni.md
[az-aks-get-credentials]: /cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials
[aks-hpa]: tutorial-kubernetes-scale.md
[aks-cluster-autoscaler]: cluster-autoscaler.md
[aks-basic-ingress]: ingress-basic.md
[az-provider-list]: /cli/azure/provider#az-provider-list
[az-provider-register]: /cli/azure/provider?view=azure-cli-latest#az-provider-register
