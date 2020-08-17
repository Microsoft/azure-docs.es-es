---
title: 'Versión preliminar: Incorporación de un grupo de nodos de Spot a un clúster de Azure Kubernetes Service (AKS)'
description: Aprenda a agregar un grupo de nodos de Spot a un clúster de Azure Kubernetes Service (AKS).
services: container-service
ms.service: container-service
ms.topic: article
ms.date: 02/25/2020
ms.openlocfilehash: dbb003c287a18810c2c14c4f2ea401fa55cca427
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87987297"
---
# <a name="preview---add-a-spot-node-pool-to-an-azure-kubernetes-service-aks-cluster"></a>Versión preliminar: Incorporación de un grupo de nodos de Spot a un clúster de Azure Kubernetes Service (AKS)

Un grupo de nodos de Spot está respaldado por un [conjunto de escalado de máquinas virtuales de Spot][vmss-spot]. El uso de máquinas virtuales de Spot para los nodos con el clúster de AKS permite aprovechar la capacidad no utilizada en Azure con un importante ahorro en los costos. La capacidad disponible no utilizada variará en función de muchos factores, incluidos el tamaño del nodo, la región y la hora del día.

Al implementar un grupo de nodos de Spot, Azure asigna los nodos de Spot si hay capacidad disponible. Sin embargo, no hay ningún Acuerdo de Nivel de Servicio para los nodos de Spot. Un conjunto de escalado de Spot que respalda un grupo de nodos de Spot se implementa en un dominio de error único y no ofrece garantías de alta disponibilidad. Siempre que Azure necesite recuperar la capacidad, su infraestructura expulsará los nodos de Spot.

Los nodos de Spot son ideales para cargas de trabajo que controlen las interrupciones, las finalizaciones tempranas o las expulsiones. Por ejemplo, los trabajos de procesamiento por lotes, los entornos de desarrollo y pruebas, y las cargas de trabajo de proceso grandes pueden ser buenos candidatos para la programación en un grupo de nodos de Spot.

En este artículo se agrega un grupo de nodos de Spot secundarios a un clúster de Azure Kubernetes Service (AKS) existente.

Se presupone que tiene un conocimiento básico de los conceptos de Kubernetes y Azure Load Balancer. Para más información, consulte [Conceptos básicos de Kubernetes de Azure Kubernetes Service (AKS)][kubernetes-concepts].

Esta funcionalidad actualmente está en su versión preliminar.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="before-you-begin"></a>Antes de empezar

Al crear un clúster para que use un grupo de nodos de Spot, este también debe usar Virtual Machine Scale Sets para los grupos de nodos y el equilibrador de carga de la SKU *estándar*. También debe agregar un grupo de nodos adicional después de crear el clúster para que pueda usar el grupo de nodos de Spot. La incorporación de un grupo de nodos adicional se trata en un paso posterior, pero primero debe habilitar la característica en versión preliminar.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

### <a name="register-spotpoolpreview-preview-feature"></a>Registro de la característica en versión preliminar spotpoolpreview

Para crear un clúster de AKS que use un grupo de nodos de Spot, debe habilitar la marca de la característica *spotpoolpreview* en la suscripción. Esta característica proporciona el conjunto de mejoras de servicio más recientes al configurar un clúster.

Para registrar la marca de característica *spotpoolpreview*, use el comando [az feature register][az-feature-register] tal como se muestra en el siguiente ejemplo:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "spotpoolpreview"
```

Tarda unos minutos en que el estado muestre *Registrado*. Puede comprobar el estado de registro con el comando [az feature list][az-feature-list]:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/spotpoolpreview')].{Name:name,State:properties.state}"
```

Cuando todo esté listo, actualice el registro del proveedor de recursos *Microsoft.ContainerService* con el comando [az provider register][az-provider-register]:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

### <a name="install-aks-preview-cli-extension"></a>Instalación de la extensión aks-preview de la CLI

Para crear un clúster de AKS que use un grupo de nodos de Spot, necesita la versión 0.4.32 o superior de la extensión de la CLI *aks-preview*. Instale la extensión de la CLI de Azure *aks-preview* con el comando [az extension add][az-extension-add] y, a continuación, busque las actualizaciones disponibles con el comando [az extension update][az-extension-update]:

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview
 
# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

### <a name="limitations"></a>Limitaciones

Al crear y administrar clústeres de AKS con un grupo de nodos de Spot se aplican las limitaciones siguientes:

* Un grupo de nodos de Spot no puede ser el predeterminado del clúster. Un grupo de nodos de Spot solo se puede usar como secundario.
* Los grupos de nodos de Spot no se pueden actualizar, ya que no pueden garantizar el acordonamiento y la purga. Debe reemplazar el grupo de nodos de Spot existente por uno nuevo para las operaciones como la actualización de la versión de Kubernetes. Para reemplazar un grupo de nodos de Spot, cree uno nuevo con una versión diferente de Kubernetes, espere hasta que su estado sea *Listo* y elimine el anterior.
* No se pueden actualizar los grupos de nodos y el plano de control al mismo tiempo. Debe actualizarlos por separado o quitar el grupo de nodos de Spot para actualizar el plano de control y los grupos de nodos restantes al mismo tiempo.
* Los grupos de nodos de Spot deben usar Virtual Machine Scale Sets.
* Una vez creados ScaleSetPriority y SpotMaxPrice no se pueden modificar.
* Al establecer SpotMaxPrice, el valor debe ser -1 o un valor positivo con hasta cinco decimales.
* Un grupo de nodos de Spot tendrá la etiqueta *kubernetes.azure.com/scalesetpriority:spot* y un valor taint *kubernetes.azure.com/scalesetpriority=spot:NoSchedule*, y los pods del sistema, antiafinidad.
* Para programar cargas de trabajo en un grupo de nodos de Spot, debe agregar el valor [toleration correspondiente][spot-toleration].

## <a name="add-a-spot-node-pool-to-an-aks-cluster"></a>Incorporación de un grupo de nodos de Spot a un clúster de AKS

Debe agregar un grupo de nodos de Spot a un clúster existente que tenga habilitados varios grupos de nodos. [Aquí][use-multiple-node-pools] encontrará más información sobre cómo crear un clúster de AKS con varios grupos de nodos.

Cree un grupo de nodos mediante el comando [az aks nodepool add][az-aks-nodepool-add].
```azurecli-interactive
az aks nodepool add \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name spotnodepool \
    --priority Spot \
    --eviction-policy Delete \
    --spot-max-price -1 \
    --enable-cluster-autoscaler \
    --min-count 1 \
    --max-count 3 \
    --no-wait
```

De forma predeterminada, al crear un clúster con varios grupos de nodos, se crea un grupo de nodos con *priority* *Regular* en el clúster de AKS. El comando anterior agrega un grupo de nodos auxiliar a un clúster de AKS existente con *priority* *Spot*. *priority* *Spot* hace que el grupo de nodos sea un grupo de nodos de Spot. El parámetro *eviction-policy* está establecido en *Delete* en el ejemplo anterior, que es el valor predeterminado. Al establecer [eviction-policy][eviction-policy] en *Delete*, los nodos del grupo de escalado subyacente del grupo de nodos se eliminan al ser expulsados. También puede establecer eviction-policy en *Deallocate*. Al establecer eviction-policy en *Deallocate*, los nodos del conjunto de escalado subyacente se establecen en estado stopped-deallocated durante la expulsión. Los nodos en estado stopped-deallocated se consideran también a efectos de la cuota de proceso, lo cual puede generar problemas de escalado o actualización del clúster. Los valores de *priority* y *eviction-policy* solo se pueden establecer durante la creación del grupo de nodos. Estos valores no se pueden actualizar más adelante.

El comando también habilita el [escalador automático de clúster][cluster-autoscaler], recomendado para los grupos de nodos de Spot. En función de las cargas de trabajo que se ejecuten en el clúster, el escalador automático de clúster escala y reduce verticalmente el número de nodos del grupo. En el caso de los grupos de nodos de Spot, el escalador automático de clúster escala verticalmente el número de nodos después de una expulsión si aún se necesitan nodos adicionales. Si cambia el máximo de nodos que puede tener un grupo, también debe ajustar el valor de `maxCount` relativo al escalador automático de clúster. Si no usa un escalador automático de clúster, en la expulsión el grupo de Spot disminuirá a cero y requerirá una operación manual para recibir nodos de Spot adicionales.

> [!Important]
> Programe solo las cargas de trabajo en grupos de nodos de Spot que puedan controlar las interrupciones, como los trabajos de procesamiento por lotes y los entornos de prueba. Se recomienda que configure los valores de [taint y toleration][taints-tolerations] en el grupo de nodos de Spot para asegurarse de que en un grupo de nodos de Spot solo estén programadas las cargas de trabajo que pueden controlar las expulsiones de nodos. Por ejemplo, el comando anterior agrega de forma predeterminada un valor taint de *kubernetes.azure.com/scalesetpriority=spot:NoSchedule* para que solo se programen los pods con el valor toleration correspondiente en este nodo.

## <a name="verify-the-spot-node-pool"></a>Comprobación del grupo de nodos de Spot

Para comprobar que el grupo de nodos se ha agregado como grupo de nodos de Spot:

```azurecli
az aks nodepool show --resource-group myResourceGroup --cluster-name myAKSCluster --name spotnodepool
```

Confirme que *scaleSetPriority* sea *Spot*.

Para programar que un pod se ejecute en un nodo de zona de Spot, agregue un valor toleration que corresponda al valor taint que se aplica al nodo de Spot. En el ejemplo siguiente se muestra una parte de un archivo YAML que define un valor toleration que corresponde a un valor taint *kubernetes.azure.com/scalesetpriority=spot:NoSchedule* que se usa en el paso anterior.

```yaml
spec:
  containers:
  - name: spot-example
  tolerations:
  - key: "kubernetes.azure.com/scalesetpriority"
    operator: "Equal"
    value: "spot"
    effect: "NoSchedule"
   ...
```

Al implementar un pod con este valor toleration, Kubernetes programa el pod correctamente en los nodos donde se ha aplicado el valor taint.

## <a name="max-price-for-a-spot-pool"></a>Precio máximo de un grupo de Spot
[Los precios de las instancias de Spot varían][pricing-spot] en función de la región y la SKU. Para más información, consulte los precios para [Linux][pricing-linux] y [Windows][pricing-windows].

La variabilidad en los precios permite establecer un precio máximo, en dólares estadounidenses (USD), con un máximo de 5 decimales. Por ejemplo, el valor *0.98765* correspondería a un precio máximo de 0,98765 USD por hora. Si establece el precio máximo en *-1*, la instancia no se expulsará según el precio. El precio de la instancia será el actual de Spot o el de una instancia estándar, el menor de los dos, siempre que haya capacidad y cuota disponibles.

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha aprendido a agregar un grupo de nodos de Spot a un clúster de AKS. Para más información sobre cómo controlar pods en grupos de nodos, consulte [Best practices for advanced scheduler features in AKS][operator-best-practices-advanced-scheduler] (Procedimientos recomendados para las características avanzadas del programador en AKS).

<!-- LINKS - External -->
[kubernetes-services]: https://kubernetes.io/docs/concepts/services-networking/service/

<!-- LINKS - Internal -->
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-group-deploy-create]: /cli/azure/group/deployment?view=azure-cli-latest#az-group-deployment-create
[az-aks-nodepool-add]: /cli/azure/aks/nodepool?view=azure-cli-latest#az-aks-nodepool-add
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-template-deploy]: ../azure-resource-manager/templates/deploy-cli.md#deployment-scope
[cluster-autoscaler]: cluster-autoscaler.md
[eviction-policy]: ../virtual-machine-scale-sets/use-spot.md#eviction-policy
[kubernetes-concepts]: concepts-clusters-workloads.md
[operator-best-practices-advanced-scheduler]: operator-best-practices-advanced-scheduler.md
[pricing-linux]: https://azure.microsoft.com/pricing/details/virtual-machine-scale-sets/linux/
[pricing-spot]: ../virtual-machine-scale-sets/use-spot.md#pricing
[pricing-windows]: https://azure.microsoft.com/pricing/details/virtual-machine-scale-sets/windows/
[spot-toleration]: #verify-the-spot-node-pool
[taints-tolerations]: operator-best-practices-advanced-scheduler.md#provide-dedicated-nodes-using-taints-and-tolerations
[use-multiple-node-pools]: use-multiple-node-pools.md
[vmss-spot]: ../virtual-machine-scale-sets/use-spot.md