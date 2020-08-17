---
title: Actualización de un clúster de Azure Kubernetes Service (AKS)
description: Obtenga información sobre cómo actualizar un clúster de Azure Kubernetes Service (AKS) para obtener las últimas características y actualizaciones de seguridad.
services: container-service
ms.topic: article
ms.date: 05/28/2020
ms.openlocfilehash: da46c44dc9cc16dfa44aacb15b35b652c0c912a9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87050621"
---
# <a name="upgrade-an-azure-kubernetes-service-aks-cluster"></a>Actualización de un clúster de Azure Kubernetes Service (AKS)

Como parte del ciclo de vida de un clúster de AKS, a menudo es preciso actualizar Kubernetes a su versión más reciente. Es importante que aplique las versiones de seguridad de Kubernetes más recientes o que realice la actualización necesaria para disfrutar de las últimas características. En este artículo se muestra cómo actualizar los componentes principales o un único grupo de nodos predeterminado en un clúster de AKS.

Para los clústeres de AKS que usan varios grupos de nodos o nodos de Windows Server, consulte [Actualización de un grupo de nodos en AKS][nodepool-upgrade].

## <a name="before-you-begin"></a>Antes de empezar

Para este artículo es preciso usar la versión 2.0.65 de la CLI de Azure, o cualquier versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli-install].

> [!WARNING]
> Una actualización del clúster de AKS desencadena un acordonamiento y purga de los nodos. Si tiene una cuota de proceso baja disponible, se puede producir un error en la actualización. Consulte el [aumento de cuotas](../azure-portal/supportability/resource-manager-core-quotas-request.md) para más información.

## <a name="check-for-available-aks-cluster-upgrades"></a>Compruebe las actualizaciones disponibles del clúster de AKS

Para comprobar qué versiones de Kubernetes están disponibles para su clúster, use el comando [az aks get-upgrades][az-aks-get-upgrades]. En el ejemplo siguiente se comprueba si hay actualizaciones disponibles para el clúster denominado *myAKSCluster* en el grupo de recursos denominado *myResourceGroup*:

```azurecli-interactive
az aks get-upgrades --resource-group myResourceGroup --name myAKSCluster --output table
```

> [!NOTE]
> Cuando se actualiza un clúster de AKS compatible, no pueden omitirse las versiones secundarias de Kubernetes. Por ejemplo, se permiten las actualizaciones entre *1.12.x* -> *1.13.x* o *1.13.x* -> *1.14.x*, pero no entre *1.12.x* -> *1.14.x*.
>
> Para actualizar de *1.12.x* -> *1.14.x*, la primera actualización sería de *1.12.x* -> *1.13.x* y, después, de *1.13.x* -> *1.14.x*.
>
> Solo se pueden omitir varias versiones cuando se actualiza de una versión que no es compatible a otra que sí lo es. Por ejemplo, si se actualiza de la versión *1.10.x*, que no es compatible, a la versión *1.15.x*, que sí lo es.

La siguiente salida de ejemplo muestra que el clúster puede actualizarse a las versiones *1.13.9* y *1.13.10*:

```console
Name     ResourceGroup     MasterVersion    NodePoolVersion    Upgrades
-------  ----------------  ---------------  -----------------  ---------------
default  myResourceGroup   1.12.8           1.12.8             1.13.9, 1.13.10
```
Si no hay ninguna actualización disponible, obtendrá:
```console
ERROR: Table output unavailable. Use the --query option to specify an appropriate query. Use --debug for more info.
```

## <a name="customize-node-surge-upgrade-preview"></a>Personalización de la actualización de sobrecargas de nodo (versión preliminar)

> [!Important]
> Los sobrecargas de nodo requieren la cuota de suscripción para el recuento máximo de sobrecargas solicitado para cada operación de actualización. Por ejemplo, un clúster con 5 grupos de nodos, cada uno con un recuento de 4 nodos, tiene un total de 20 nodos. Si cada grupo de nodos tiene un valor de sobrecarga máxima del 50 %, se requiere un proceso adicional y una cuota de IP de 10 nodos (2 nodos x 5 grupos) para completar la actualización.
>
> Si usa Azure CNI, compruebe que haya direcciones IP disponibles en la subred, además de [cumplir los requisitos de direcciones IP de Azure CNI](configure-azure-cni.md).

De forma predeterminada, AKS configura las actualizaciones para la sobrecarga con un nodo adicional. Un valor predeterminado de uno para la configuración de sobrecarga máxima permite a AKS minimizar la interrupción de la carga de trabajo mediante la creación de un nodo adicional antes de acordonar o purgar las aplicaciones existentes para reemplazar un nodo de una versión anterior. El valor de sobrecarga máxima se puede personalizar por grupo de nodos para permitir un equilibrio entre la velocidad de actualización y la interrupción de la actualización. Al aumentar el valor de sobrecarga máxima, el proceso de actualización se completa más rápido, pero si se establece un valor alto para la sobrecarga máxima, pueden producirse interrupciones durante el proceso de actualización. 

Por ejemplo, un valor de sobrecarga máxima del 100 % proporciona el proceso de actualización más rápido posible (se duplica el número de nodos), pero también hace que todos los nodos del grupo de nodos se purguen simultáneamente. Es posible que desee usar un valor más alto, como este, para los entornos de prueba. En el caso de los grupos de nodos de producción, se recomienda un valor de sobrecarga máxima del 33 %.

AKS acepta valores enteros y un valor de porcentaje para la sobrecarga máxima. Un entero como "5" indica cinco nodos adicionales para la sobrecarga. Un valor de "50 %" indica un valor de sobrecarga de la mitad del número de nodos actual del grupo. Los valores de porcentaje de sobrecarga máxima pueden ser de 1 % como mínimo y 100 % como máximo. Un valor de porcentaje se redondea al número de nodos más próximo. Si el valor de sobrecarga máxima es inferior al número de nodos actual en el momento de la actualización, el número de nodos actual se usa para el valor de sobrecarga máxima.

Durante una actualización, el valor de sobrecarga máxima puede ser 1 como mínimo y un valor igual al número de nodos del grupo de nodos como máximo. Puede establecer valores mayores, pero el número máximo de nodos que se usan para la sobrecarga máxima no será mayor que el número de nodos del grupo en el momento de la actualización.

### <a name="set-up-the-preview-feature-for-customizing-node-surge-upgrade"></a>Configuración de la característica de vista previa para la personalización de la actualización de la sobrecarga de nodos

```azurecli-interactive
# register the preview feature
az feature register --namespace "Microsoft.ContainerService" --name "MaxSurgePreview"
```

Tardará varios minutos en registrarse. Use el siguiente comando para comprobar que la característica está registrada:

```azurecli-interactive
# Verify the feature is registered:
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/MaxSurgePreview')].{Name:name,State:properties.state}"
```

Durante la versión preliminar, necesita la extensión *aks-preview* de la CLI para usar la sobrecarga máxima. Use el comando [az extension add][az-extension-add] y, a continuación, busque las actualizaciones disponibles mediante el comando [az extension update][az-extension-update]:

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

> [!Important]
> La configuración de sobrecarga máxima en un grupo de nodos es permanente.  Las actualizaciones posteriores de Kubernetes o las actualizaciones de la versión de nodo usarán esta configuración. Puede cambiar el valor de sobrecarga máxima para los grupos de nodos en cualquier momento. En el caso de los grupos de nodos de producción, se recomienda un valor de sobrecarga máxima del 33 %.

Use los siguientes comandos para establecer los valores de sobrecarga máxima para los grupos de nodos nuevos o existentes.

```azurecli-interactive
# Set max surge for a new node pool
az aks nodepool add -n mynodepool -g MyResourceGroup --cluster-name MyManagedCluster --max-surge 33%
```

```azurecli-interactive
# Update max surge for an existing node pool 
az aks nodepool update -n mynodepool -g MyResourceGroup --cluster-name MyManagedCluster --max-surge 5
```

## <a name="upgrade-an-aks-cluster"></a>Actualización de un clúster de AKS

Con una lista de las versiones disponibles para el clúster de AKS, use el comando [az aks upgrade][az-aks-upgrade] para realizar la actualización. En el proceso de actualización, AKS agrega un nuevo nodo al clúster que ejecuta la versión de Kubernetes especificada y después [acordona y purga][kubernetes-drain] uno de los nodos antiguos para minimizar las interrupciones de las aplicaciones en ejecución. Cuando se confirma que el nuevo nodo ejecuta pods de aplicación, el anterior se elimina. Este proceso se repite hasta que se hayan actualizado todos los nodos del clúster.

```azurecli-interactive
az aks upgrade \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --kubernetes-version KUBERNETES_VERSION
```

Dicha actualización tarda varios minutos. El tiempo exacto dependerá del número de nodos que tenga.

> [!NOTE]
> Hay un tiempo total permitido para que se complete la actualización de un clúster. Este tiempo se calcula tomando el producto de `10 minutes * total number of nodes in the cluster`. Por ejemplo, en un clúster de 20 nodos, las operaciones de actualización deben realizarse correctamente en 200 minutos o AKS producirá un error en la operación para evitar un estado de clúster irrecuperable. Para la recuperación en caso de error de actualización, vuelva a intentar la operación una vez alcanzado el tiempo de espera.

Para confirmar que la actualización se ha realizado correctamente, use el comando [az aks show][az-aks-show]:

```azurecli-interactive
az aks show --resource-group myResourceGroup --name myAKSCluster --output table
```

En la salida de ejemplo siguiente se muestra que el clúster ahora ejecuta la versión *1.13.10*:

```json
Name          Location    ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
------------  ----------  ---------------  -------------------  -------------------  ---------------------------------------------------------------
myAKSCluster  eastus      myResourceGroup  1.13.10               Succeeded            myaksclust-myresourcegroup-19da35-90efab95.hcp.eastus.azmk8s.io
```

## <a name="next-steps"></a>Pasos siguientes

En este artículo se ha mostrado cómo actualizar un clúster de AKS existente. Para más información acerca de la implementación y administración de clústeres de AKS, consulte el conjunto de tutoriales.

> [!div class="nextstepaction"]
> [Tutoriales de AKS][aks-tutorial-prepare-app]

<!-- LINKS - external -->
[kubernetes-drain]: https://kubernetes.io/docs/tasks/administer-cluster/safely-drain-node/

<!-- LINKS - internal -->
[aks-tutorial-prepare-app]: ./tutorial-kubernetes-prepare-app.md
[azure-cli-install]: /cli/azure/install-azure-cli
[az-aks-get-upgrades]: /cli/azure/aks#az-aks-get-upgrades
[az-aks-upgrade]: /cli/azure/aks#az-aks-upgrade
[az-aks-show]: /cli/azure/aks#az-aks-show
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
