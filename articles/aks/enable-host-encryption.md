---
title: Habilitación del cifrado basado en host en Azure Kubernetes Service (AKS)
description: Aprenda a configurar un cifrado basado en host en un clúster de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 07/10/2020
ms.openlocfilehash: 4b5deeec0b76520952345e9b03135fa094a1f78e
ms.sourcegitcommit: 25bb515efe62bfb8a8377293b56c3163f46122bf
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87986872"
---
# <a name="host-based-encryption-on-azure-kubernetes-service-aks-preview"></a>Cifrado basado en host en Azure Kubernetes Service (AKS) (versión preliminar)

Con el cifrado basado en host, los datos almacenados en el host de máquina virtual de las máquinas virtuales de los nodos de agente de AKS se cifran en reposo y se transmiten cifrados al servido Storage. Esto significa que los discos temporales se cifran en reposo con claves administradas por la plataforma. La memoria caché de los discos de datos y del sistema operativo se cifra en reposo con claves administradas por la plataforma o por el cliente, según el tipo de cifrado establecido en esos discos. De forma predeterminada, cuando se usa AKS, el sistema operativo y los discos de datos se cifran en reposo con claves administradas por la plataforma, lo que significa que las memorias caché de estos discos también se cifran de forma predeterminada en reposo con claves administradas por la plataforma.  Puede especificar sus propias claves administradas siguiendo [Traiga sus propias claves (BYOK) con discos de Azure en Azure Kubernetes Service (AKS)](azure-disk-customer-managed-keys.md). La memoria caché de estos discos también se cifrará con la clave que especifique en este paso.


## <a name="before-you-begin"></a>Antes de empezar

Esta característica solo se puede establecer durante la creación del clúster o en el momento de creación del grupo de nodos.

> [!NOTE]
> El cifrado basado en host está disponible en [regiones de Azure][supported-regions] que admiten el cifrado del lado servidor de discos administrados de Azure y solo con [tamaños de máquinas virtuales compatibles][supported-sizes] específicos.

### <a name="prerequisites"></a>Requisitos previos

- Asegúrese de que tiene instalada la `aks-preview` extensión de la CLI v 0.4.55 o superior.
- Asegúrese de que tiene habilitada la marca de características `EncryptionAtHost` bajo `Microsoft.Compute`.
- Asegúrese de que tiene habilitada la marca de características `EnableEncryptionAtHostPreview` bajo `Microsoft.ContainerService`.

### <a name="register-encryptionathost--preview-features"></a>Registro de las características en versión preliminar de `EncryptionAtHost`

Para crear un clúster de AKS que usa el cifrado basado en host, debe habilitar las marcas de características `EnableEncryptionAtHostPreview` y `EncryptionAtHost` en su suscripción.

Registro de `EncryptionAtHost` la marca de característica con el comando de [característica de registro az][az-feature-register], tal como se muestra en el siguiente ejemplo:

```azurecli-interactive
az feature register --namespace "Microsoft.Compute" --name "EncryptionAtHost"

az feature register --namespace "Microsoft.ContainerService"  --name "EnableEncryptionAtHostPreview"
```

Tarda unos minutos en que el estado muestre *Registrado*. Puede comprobar el estado de registro con el comando [az feature list][az-feature-list]:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.Compute/EncryptionAtHost')].{Name:name,State:properties.state}"

az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnableEncryptionAtHostPreview')].{Name:name,State:properties.state}"
```

Cuando todo esté listo, actualice el registro de los proveedores de recursos `Microsoft.ContainerService` y `Microsoft.Compute` con el comando [az provider register][az-provider-register]:

```azurecli-interactive
az provider register --namespace Microsoft.Compute

az provider register --namespace Microsoft.ContainerService
```

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

### <a name="install-aks-preview-cli-extension"></a>Instalación de la extensión aks-preview de la CLI

Para crear un clúster de AKS que realice el cifrado basado en host, requiere la extensión de la CLI *aks-preview* más reciente. Instale la extensión de la CLI de Azure *aks-preview* con el comando [az extension add][az-extension-add] o busque las actualizaciones disponibles con el comando [az extension update][az-extension-update]:

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

### <a name="limitations"></a>Limitaciones

- Solo se puede habilitar en grupos de nodos nuevos o en clústeres nuevos.
- Solo se puede habilitar en [regiones de Azure][supported-regions] que admiten el cifrado del lado servidor de discos administrados de Azure y solo con [tamaños de máquinas virtuales compatibles][supported-sizes] específicos.
- Requiere un clúster de AKS y un grupo de nodos basado en Virtual Machine Scale Sets (VMSS) como *tipo de conjunto de máquinas virtuales*.

## <a name="use-host-based-encryption-on-new-clusters-preview"></a>Uso del cifrado basado en host en clústeres nuevos (versión preliminar)

Configure los nodos de agente de clúster para usar el cifrado basado en host cuando se cree el clúster. Use la marca `--aks-custom-headers` para establecer el encabezado `EnableEncryptionAtHost`.

```azurecli-interactive
az aks create --name myAKSCluster --resource-group myResourceGroup -s Standard_DS2_v2 -l westus2 --aks-custom-headers EnableEncryptionAtHost=true
```

Si desea crear clústeres sin el cifrado basado en host, puede omitir el parámetro `--aks-custom-headers` personalizado para hacerlo.

## <a name="use-host-based-encryption-on-existing-clusters-preview"></a>Uso del cifrado basado en host en clústeres existentes (versión preliminar)

Puede habilitar el cifrado basado en host en clústeres existentes agregando un nuevo grupo de nodos al clúster. Configure un grupo de nodos nuevo para usar el cifrado basado en host mediante la marca `--aks-custom-headers`.

```azurecli
az aks nodepool add --name hostencrypt --cluster-name myAKSCluster --resource-group myResourceGroup -s Standard_DS2_v2 -l westus2 --aks-custom-headers EnableEncryptionAtHost=true
```

Si desea crear grupos de nodos nuevos sin la característica de cifrado basada en host, puede hacerlo omitiendo el parámetro `--aks-custom-headers` personalizado.

## <a name="next-steps"></a>Pasos siguientes

Revise las [prácticas recomendadas para la seguridad del clúster de AKS][best-practices-security]. Lea más sobre el [cifrado basado en host](../virtual-machines/linux/disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data).


<!-- LINKS - external -->

<!-- LINKS - internal -->
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[best-practices-security]: ./operator-best-practices-cluster-security.md
[supported-regions]: ../virtual-machines/linux/disk-encryption.md#supported-regions
[supported-sizes]: ../virtual-machines/linux/disk-encryption.md#supported-vm-sizes
[azure-cli-install]: /cli/azure/install-azure-cli
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
