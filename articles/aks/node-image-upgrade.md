---
title: Actualización de imágenes de nodos de Azure Kubernetes Service (AKS)
description: Obtenga información sobre cómo actualizar las imágenes de nodos y grupos de nodos de un clúster de AKS.
author: laurenhughes
ms.author: lahugh
ms.service: container-service
ms.topic: conceptual
ms.date: 07/13/2020
ms.openlocfilehash: 040f4378e01c3696b9a74bfcc27230503828f19a
ms.sourcegitcommit: 97a0d868b9d36072ec5e872b3c77fa33b9ce7194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/04/2020
ms.locfileid: "87562794"
---
# <a name="preview---azure-kubernetes-service-aks-node-image-upgrades"></a>Versión preliminar: actualizaciones de imágenes de nodo de Azure Kubernetes Service (AKS)

AKS admite la actualización de las imágenes de un nodo para que esté actualizado con las actualizaciones más recientes del sistema operativo y en tiempo de ejecución. AKS proporciona una nueva imagen por semana con las actualizaciones más recientes, por lo que es beneficioso actualizar las imágenes del nodo con regularidad por las características más recientes, incluidas las revisiones de Linux o Windows. En este artículo se muestra cómo actualizar las imágenes de nodo de un clúster de AKS y cómo actualizar las imágenes de grupo de nodos sin actualizar la versión de Kubernetes.

Si está interesado en obtener información sobre las imágenes más recientes proporcionadas por AKS, consulte las [notas de la versión de AKS](https://github.com/Azure/AKS/releases) para obtener más detalles.

Para obtener información sobre la actualización de la versión del clúster de Kubernetes, vea [Actualización de un clúster de AKS][upgrade-cluster].

## <a name="register-the-node-image-upgrade-preview-feature"></a>Registro de la característica de versión preliminar de actualización de imágenes de nodo

Para usar la característica de actualización de imágenes de nodo durante el período de versión preliminar, debe registrar la característica.

```azurecli
# Register the preview feature
az feature register --namespace "Microsoft.ContainerService" --name "NodeImageUpgradePreview"
```

El registro tardará varios minutos en completarse. Use el siguiente comando para comprobar que la característica está registrada:

```azurecli
# Verify the feature is registered:
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/NodeImageUpgradePreview')].{Name:name,State:properties.state}"
```

Durante la versión preliminar, necesita la extensión *aks-preview* de la CLI para usar la actualización de imágenes de nodo. Use el comando [az extension add][az-extension-add] y, a continuación, busque las actualizaciones disponibles mediante el comando [az extension update][az-extension-update]:

```azurecli
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

Cuando el estado se muestre como Registrado, actualice el registro del proveedor de recursos `Microsoft.ContainerService` mediante el comando [az provider register](/cli/azure/provider?view=azure-cli-latest#az-provider-register):

```azurecli
az provider register --namespace Microsoft.ContainerService
```  

## <a name="upgrade-all-nodes-in-all-node-pools"></a>Actualización de todos los nodos de todos los grupos de nodos

La actualización de imágenes de nodo se realiza con `az aks upgrade`. Para actualizar la imagen de nodo, utilice el siguiente comando:

```azurecli
az aks upgrade \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-image-only
```

Durante la actualización, compruebe el estado de las imágenes de nodo con el siguiente comando `kubectl`, para obtener las etiquetas y filtrar la información de la imagen de nodo actual:

```azurecli
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'
```

Una vez completada la actualización, use `az aks show` para obtener los detalles actualizados del grupo de nodos. La imagen del nodo actual se muestra en la propiedad `nodeImageVersion`.

```azurecli
az aks show \
    --resource-group myResourceGroup \
    --name myAKSCluster
```

## <a name="upgrade-a-specific-node-pool"></a>Actualización de un grupo de nodos específico

La actualización de la imagen en un grupo de nodos es similar a la actualización de la imagen en un clúster.

Para actualizar la imagen del sistema operativo del grupo de nodos sin realizar una actualización del clúster de Kubernetes, use la opción `--node-image-only` en el ejemplo siguiente:

```azurecli
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --node-image-only
```

Durante la actualización, compruebe el estado de las imágenes de nodo con el siguiente comando `kubectl`, para obtener las etiquetas y filtrar la información de la imagen de nodo actual:

```azurecli
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'
```

Una vez completada la actualización, use `az aks nodepool show` para obtener los detalles actualizados del grupo de nodos. La imagen del nodo actual se muestra en la propiedad `nodeImageVersion`.

```azurecli
az aks nodepool show \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool
```

## <a name="upgrade-node-images-with-node-surge"></a>Actualización de imágenes de nodo con sobrecarga de nodos

Para acelerar el proceso de actualización de la imagen de nodo, puede actualizar las imágenes de nodo mediante un valor de sobrecarga de nodo personalizable. De forma predeterminada, AKS usa un nodo adicional para configurar las actualizaciones.

Si desea aumentar la velocidad de las actualizaciones, use el valor `--max-surge` para configurar el número de nodos que se usarán para las actualizaciones, de forma que se completen más rápido. Para obtener más información sobre las ventajas e inconvenientes de varias configuraciones de `--max-surge`, consulte [Personalización de la actualización de sobrecargas de nodo][max-surge].

El siguiente comando establece el valor máximo de sobrecarga para realizar una actualización de imagen de nodo:

```azurecli
az aks nodepool upgrade \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool \
    --max-surge 33% \
    --node-image-only \
    --no-wait
```

Durante la actualización, compruebe el estado de las imágenes de nodo con el siguiente comando `kubectl`, para obtener las etiquetas y filtrar la información de la imagen de nodo actual:

```azurecli
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.metadata.labels.kubernetes\.azure\.com\/node-image-version}{"\n"}{end}'
```

Use `az aks nodepool show` para obtener los detalles actualizados del grupo de nodos. La imagen del nodo actual se muestra en la propiedad `nodeImageVersion`.

```azurecli
az aks nodepool show \
    --resource-group myResourceGroup \
    --cluster-name myAKSCluster \
    --name mynodepool
```

## <a name="next-steps"></a>Pasos siguientes

- Consulte las [notas de la versión de AKS](https://github.com/Azure/AKS/releases) para más información sobre las imágenes de nodo más recientes.
- Obtenga información sobre cómo actualizar la versión de Kubernetes con [Actualización de un clúster de AKS][upgrade-cluster].
- [Aplicación de actualizaciones de kernel y seguridad a los nodos de Linux en Azure Kubernetes Service (AKS)][security-update]
- Obtenga más información sobre varios grupos de nodos y cómo actualizar los grupos de nodos con [Creación y administración de varios grupos de nodos][use-multiple-node-pools].

<!-- LINKS - internal -->
[upgrade-cluster]: upgrade-cluster.md
[security-update]: node-updates-kured.md
[use-multiple-node-pools]: use-multiple-node-pools.md
[max-surge]: upgrade-cluster.md#customize-node-surge-upgrade-preview
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
