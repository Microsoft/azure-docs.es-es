---
title: Creación de un conjunto de equilibrador de carga interno
titleSuffix: Azure Kubernetes Service
description: Aprenda a crear y usar un equilibrador de carga interno para exponer los servicios con Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 03/04/2019
ms.openlocfilehash: ec8fd1f1b32d5bba6dc4dc756e1f95f4a74f9a96
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87285890"
---
# <a name="use-an-internal-load-balancer-with-azure-kubernetes-service-aks"></a>Uso de un equilibrador de carga interno con Azure Kubernetes Service (AKS)

Para restringir el acceso a las aplicaciones en Azure Kubernetes Service (AKS), puede crear y usar un equilibrador de carga interno. Un equilibrador de carga interno permite que un servicio de Kubernetes sea accesible solo para las aplicaciones que se ejecutan en la misma red virtual que el clúster de Kubernetes. En este artículo se muestra cómo crear y usar un equilibrador de carga interno con Azure Kubernetes Service (AKS).

> [!NOTE]
> Azure Load Balancer está disponible en dos SKU: *Básica* y *Estándar*. De forma predeterminada, la SKU Estándar se usa al crear un clúster de AKS.  Al crear un servicio con el tipo LoadBalancer, obtendrá el mismo tipo de LB que al aprovisionar el clúster. Para más información, consulte el apartado de [comparación de las SKU de Azure Load Balancer][azure-lb-comparison].

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

También es preciso que esté instalada y configurada la versión 2.0.59 de la CLI de Azure u otra versión posterior. Ejecute  `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte  [Install Azure CLI][install-azure-cli] (Instalación de la CLI de Azure).

Si usa una subred o un grupo de recursos existentes, la entidad de servicio del clúster de AKS necesita permiso para administrar los recursos de red. Para más información, vea [Uso de redes kubenet con intervalos de direcciones IP propios en Azure Kubernetes Service (AKS)][use-kubenet] o [Configuración de redes de Azure CNI en Azure Kubernetes Service (AKS)][advanced-networking]. Si va a configurar el equilibrador de carga para usar una [dirección IP en una subred diferente][different-subnet], asegúrese de que la entidad de servicio del clúster de AKS también tenga acceso de lectura a esa subred.

En lugar de una entidad de servicio, también puede usar la identidad administrada asignada por el sistema para los permisos. Para más información, consulte [Uso de identidades administradas](use-managed-identity.md). Para más información sobre los permisos, consulte [Delegación del acceso de AKS a otros recursos de Azure][aks-sp].

## <a name="create-an-internal-load-balancer"></a>Creación de un conjunto de equilibrador de carga interno

Para crear un equilibrador de carga interno, cree un manifiesto de servicio denominado `internal-lb.yaml` con el tipo de servicio *LoadBalancer* y la anotación *azure-load-balancer-internal*, tal como se muestra en el siguiente ejemplo:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-app
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: internal-app
```

Implemente el equilibrador de carga interno mediante [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl apply -f internal-lb.yaml
```

Se crea una instancia de Azure Load Balancer en el grupo de recursos del nodo, y se conecta a la misma red virtual que el clúster de AKS.

Cuando visualiza los detalles del servicio, la dirección IP del equilibrador de carga interno se muestra en la columna *EXTERNAL-IP*. En este contexto, *externo* es en relación con la interfaz externa del equilibrador de carga, no es que reciba una dirección IP pública externa. La dirección IP puede tardar uno o dos minutos en cambiar de *\<pending\>* a una dirección IP interna real, como se muestra en el ejemplo siguiente:

```
$ kubectl get service internal-app

NAME           TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
internal-app   LoadBalancer   10.0.248.59   10.240.0.7    80:30555/TCP   2m
```

## <a name="specify-an-ip-address"></a>Especificación de una dirección IP

Si quiere usar una dirección IP específica con el equilibrador de carga interno, agregue la propiedad *loadBalancerIP* al manifiesto YAML del equilibrador de carga. En este escenario, la dirección IP especificada debe encontrarse en la misma subred que el clúster de AKS y no se debe haber asignado ya a un recurso. Por ejemplo, no debe usar una dirección IP en el intervalo designado para la subred de Kubernetes.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-app
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
spec:
  type: LoadBalancer
  loadBalancerIP: 10.240.0.25
  ports:
  - port: 80
  selector:
    app: internal-app
```

Al implementar y ver los detalles del servicio, la dirección IP de la columna *EXTERNAL-IP* refleja la dirección IP especificada:

```
$ kubectl get service internal-app

NAME           TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
internal-app   LoadBalancer   10.0.184.168   10.240.0.25   80:30225/TCP   4m
```

Para más información sobre cómo configurar el equilibrador de carga en una subred diferente, vea [Especificación de una subred diferente][different-subnet].

## <a name="use-private-networks"></a>Uso de redes privadas

Al crear su clúster de AKS, puede especificar la configuración de red avanzada. Este enfoque le permite implementar el clúster en una red y subredes virtuales de Azure existentes. Un escenario es implementar el clúster de AKS en una red privada conectada a su entorno local y ejecutar servicios solo accesibles internamente. Para más información, consulte cómo configurar sus propias subredes de red virtual con [Kubenet][use-kubenet] o [Azure CNI][advanced-networking].

No se necesitan cambios en los pasos anteriores para implementar un equilibrador de carga interno en un clúster de AKS que utilice una red privada. El equilibrador de carga se crea en el mismo grupo de recursos que el clúster de AKS pero está conectado a la red virtual privada y subred, como se muestra en el siguiente ejemplo:

```
$ kubectl get service internal-app

NAME           TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
internal-app   LoadBalancer   10.1.15.188   10.0.0.35     80:31669/TCP   1m
```

> [!NOTE]
> Es posible que tenga que conceder a la entidad de servicio del clúster de AKS la función *Colaborador de red* al grupo de recursos en el que están implementados los recursos de la red virtual de Azure. Consulte la entidad de servicio con [az aks show][az-aks-show], como `az aks show --resource-group myResourceGroup --name myAKSCluster --query "servicePrincipalProfile.clientId"`. Para crear una asignación de roles, use el comando [az role assignment create][az-role-assignment-create].

## <a name="specify-a-different-subnet"></a>Especificación de una subred diferente

Para especificar una subred para el equilibrador de carga, agregue la anotación *azure-load-balancer-internal-subnet* al servicio. La subred especificada debe estar en la misma red virtual que el clúster de AKS. Cuando se implementa, la dirección *EXTERNAL-IP* del equilibrador de carga forma parte de la subred especificada.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-app
  annotations:
    service.beta.kubernetes.io/azure-load-balancer-internal: "true"
    service.beta.kubernetes.io/azure-load-balancer-internal-subnet: "apps-subnet"
spec:
  type: LoadBalancer
  ports:
  - port: 80
  selector:
    app: internal-app
```

## <a name="delete-the-load-balancer"></a>Eliminación del equilibrador de carga

Cuando se han eliminado todos los servicios que usan el equilibrador de carga interno, también se elimina el propio equilibrador de carga.

También puede eliminar directamente un servicio del mismo modo que cualquier recurso de Kubernetes, como `kubectl delete service internal-app`, acción que también eliminará el equilibrador de carga de Azure subyacente.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre los servicios de Kubernetes en la [documentación de servicios de Kubernetes][kubernetes-services].

<!-- LINKS - External -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubernetes-services]: https://kubernetes.io/docs/concepts/services-networking/service/
[aks-engine]: https://github.com/Azure/aks-engine

<!-- LINKS - Internal -->
[advanced-networking]: configure-azure-cni.md
[az-aks-show]: /cli/azure/aks#az-aks-show
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[azure-lb-comparison]: ../load-balancer/skus.md
[use-kubenet]: configure-kubenet.md
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[aks-sp]: kubernetes-service-principal.md#delegate-access-to-other-azure-resources
[different-subnet]: #specify-a-different-subnet