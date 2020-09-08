---
title: Creación dinámica de volúmenes de Azure Disks
titleSuffix: Azure Kubernetes Service
description: Aprenda a crear de forma dinámica un volumen persistente con discos de Azure en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 07/10/2020
ms.openlocfilehash: 06aad076836c0f6fdc59c4ed5d0116231080d15c
ms.sourcegitcommit: 56cbd6d97cb52e61ceb6d3894abe1977713354d9
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88683613"
---
# <a name="dynamically-create-and-use-a-persistent-volume-with-azure-disks-in-azure-kubernetes-service-aks"></a>Creación dinámica y uso de un volumen persistente con discos de Azure en Azure Kubernetes Service (AKS)

Un volumen persistente representa un fragmento de almacenamiento aprovisionado para su uso con pods de Kubernetes. Un volumen persistente puede usarse en uno o varios pods y puede aprovisionarse de forma dinámica o estática. En este artículo se muestra cómo crear dinámicamente volúmenes persistentes con discos de Azure para usarlos en un solo pod de un clúster de Azure Kubernetes Service (AKS).

> [!NOTE]
> Solo se puede montar un disco de Azure con el tipo de *modo de acceso* *ReadWriteOnce*, que lo pone a disposición de un único pod en AKS. Si tiene que compartir un volumen persistente en varios pods, use [Azure Files][azure-files-pvc].

Para más información sobre los volúmenes de Kubernetes, consulte [Opciones de almacenamiento para aplicaciones en AKS][concepts-storage].

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

También es preciso que esté instalada y configurada la versión 2.0.59 de la CLI de Azure u otra versión posterior. Ejecute  `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte  [Install Azure CLI][install-azure-cli] (Instalación de la CLI de Azure).

## <a name="built-in-storage-classes"></a>Clases de almacenamiento integradas

Una clase de almacenamiento se usa para definir cómo se crea dinámicamente una unidad de almacenamiento con un volumen persistente. Para más información sobre las clases de almacenamiento de Kubernetes, consulte las [clases de almacenamiento de Kubernetes][kubernetes-storage-classes].

Cada clúster de AKS incluye cuatro clases de almacenamiento creadas previamente, dos de las cuales están configuradas para funcionar con discos de Azure:

* La clase de almacenamiento *predeterminada* aprovisiona un disco SSD de Azure estándar.
    * Standard Storage está respaldado por SSD estándar y ofrece un almacenamiento rentable, al tiempo que proporciona un rendimiento fiable. 
* La clase de almacenamiento *Premium administrada* aprovisiona un disco de Azure premium.
    * Los discos Premium están respaldados por un disco de latencia reducida y alto rendimiento basado en SSD. Es perfecto para máquinas virtuales que ejecutan cargas de trabajo de producción. Si los nodos de AKS del clúster usan almacenamiento Premium, seleccione la clase *administrada Premium*.
    
Si usa una de las clases de almacenamiento predeterminadas, no puede actualizar el tamaño del volumen una vez creada la clase de almacenamiento. Para poder actualizar el tamaño del volumen después de crear una clase de almacenamiento, agregue la línea `allowVolumeExpansion: true` a una de las clases de almacenamiento predeterminadas o también puede crear su propia clase de almacenamiento personalizada. Tenga en cuenta que no se admite para reducir el tamaño de una PVC (para evitar la pérdida de datos). Puede editar una clase de almacenamiento existente con el comando `kubectl edit sc`. 

Por ejemplo, si desea usar un disco de 4 TiB de tamaño, debe crear una clase de almacenamiento que defina `cachingmode: None`, porque el [almacenamiento en caché de discos no se admite en discos de 4 TiB y más grandes](../virtual-machines/premium-storage-performance.md#disk-caching).

Para más información sobre las clases de almacenamiento y la creación de la suya propia, consulte [Opciones de almacenamiento de aplicaciones en AKS][storage-class-concepts].

Use el comando [kubectl get sc][kubectl-get] para ver las clases de almacenamiento que se crearon previamente. El ejemplo siguiente muestra las clases de almacenamiento creadas previamente disponibles dentro de un clúster de AKS:

```console
$ kubectl get sc

NAME                PROVISIONER                AGE
default (default)   kubernetes.io/azure-disk   1h
managed-premium     kubernetes.io/azure-disk   1h
```

> [!NOTE]
> Las notificaciones de volumen persistente se especifican en GiB, pero los discos administrados de Azure se facturan por SKU para un tamaño específico. Estas SKU varían de 32 GiB para discos S4 o P4 a 32 TiB para discos S80 o P80 (en versión preliminar). El rendimiento de transferencia de datos de red y el rendimiento IOPS de un disco administrado Premium depende de la SKU y del tamaño de instancia de los nodos en el clúster de AKS. Para más información, consulte [Precios y rendimiento de Managed Disks][managed-disk-pricing-performance].

## <a name="create-a-persistent-volume-claim"></a>Creación de una notificación de volumen persistente

Una notificación de volumen persistente (PVC) se usa para aprovisionar automáticamente el almacenamiento en función de una clase de almacenamiento. En ese caso, una PVC puede usar una de las clases de almacenamiento que crearon previamente para crear un disco administrado de Azure estándar o premium.

Cree un archivo denominado `azure-premium.yaml` y cópielo en el siguiente código manifiesto. La notificación solicita un disco llamado `azure-managed-disk` esto es *5GB* de tamaño con acceso *ReadWriteOnce*. La clase *Premium administrada*se especifica como la clase de almacenamiento.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: azure-managed-disk
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: managed-premium
  resources:
    requests:
      storage: 5Gi
```

> [!TIP]
> Para crear un disco que usa almacenamiento estándar, use `storageClassName: default` en lugar de la clase *Premium administrada*.

Cree la notificación del volumen persistente con el comando [kubectl apply][kubectl-apply] y especifique su archivo *azure-premium.yaml*:

```console
$ kubectl apply -f azure-premium.yaml

persistentvolumeclaim/azure-managed-disk created
```

## <a name="use-the-persistent-volume"></a>Uso del volumen persistente

Una vez que se haya creado la notificación de volumen persistente y tras el aprovisionamiento correcto del disco, se puede crear un pod con acceso al disco. El siguiente manifiesto crea un pod NGINX básico que utiliza la notificación de volumen persistente *azure-managed-disk* para montar el disco de Azure en la ruta de acceso `/mnt/azure`. Para los contenedores de Windows Server, especifique un elemento *mountPath* con la convención de ruta de acceso de Windows, como *"D:"* .

Cree un archivo denominado `azure-pvc-disk.yaml` y cópielo en el siguiente código manifiesto.

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
  - name: mypod
    image: nginx:1.15.5
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
    - mountPath: "/mnt/azure"
      name: volume
  volumes:
    - name: volume
      persistentVolumeClaim:
        claimName: azure-managed-disk
```

Cree el pod con el comando [kubectl apply][kubectl-apply], tal como se muestra en el ejemplo siguiente:

```console
$ kubectl apply -f azure-pvc-disk.yaml

pod/mypod created
```

Ahora tiene un pod en ejecución con el disco de Azure montado en el directorio `/mnt/azure`. Esta configuración puede verse al examinar el pod mediante `kubectl describe pod mypod`, tal y como se muestra en el siguiente ejemplo reducido:

```console
$ kubectl describe pod mypod

[...]
Volumes:
  volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  azure-managed-disk
    ReadOnly:   false
  default-token-smm2n:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-smm2n
    Optional:    false
[...]
Events:
  Type    Reason                 Age   From                               Message
  ----    ------                 ----  ----                               -------
  Normal  Scheduled              2m    default-scheduler                  Successfully assigned mypod to aks-nodepool1-79590246-0
  Normal  SuccessfulMountVolume  2m    kubelet, aks-nodepool1-79590246-0  MountVolume.SetUp succeeded for volume "default-token-smm2n"
  Normal  SuccessfulMountVolume  1m    kubelet, aks-nodepool1-79590246-0  MountVolume.SetUp succeeded for volume "pvc-faf0f176-8b8d-11e8-923b-deb28c58d242"
[...]
```

## <a name="use-ultra-disks"></a>Uso de Ultra Disks
Para utilizar el almacenamiento en disco Ultra, vea [Uso de Ultra Disks en Azure Kubernetes Service (AKS)](use-ultra-disks.md).

## <a name="back-up-a-persistent-volume"></a>Realización de una copia de seguridad de un volumen persistente

Para realizar una copia de seguridad de los datos de su volumen persistente, tome una instantánea del disco administrado para el volumen. A continuación, puede usar esta instantánea para crear un disco restaurado y asociarlo a los pods como un medio para restaurar los datos.

En primer lugar, obtenga el nombre del volumen con el comando `kubectl get pvc`, como por ejemplo para la clase PVC denominada *azure-managed-disk*:

```console
$ kubectl get pvc azure-managed-disk

NAME                 STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
azure-managed-disk   Bound     pvc-faf0f176-8b8d-11e8-923b-deb28c58d242   5Gi        RWO            managed-premium   3m
```

Este nombre de volumen constituye el nombre de disco de Azure subyacente. Realice una consulta para el identificador del disco con [az disk list][az-disk-list] y proporcione el nombre del volumen PVC, tal como se muestra en el ejemplo siguiente:

```azurecli-interactive
$ az disk list --query '[].id | [?contains(@,`pvc-faf0f176-8b8d-11e8-923b-deb28c58d242`)]' -o tsv

/subscriptions/<guid>/resourceGroups/MC_MYRESOURCEGROUP_MYAKSCLUSTER_EASTUS/providers/MicrosoftCompute/disks/kubernetes-dynamic-pvc-faf0f176-8b8d-11e8-923b-deb28c58d242
```

Use el identificador del disco para crear un disco de instantánea con [az snapshot create][az-snapshot-create]. El ejemplo siguiente crea una instantánea denominada *pvcSnapshot* en el mismo grupo de recursos que el clúster de AKS (*MC_myResourceGroup_myAKSCluster_eastus*). Puede tener algún problema con los permisos si crea instantáneas y restaura discos en grupos de recursos a los que el clúster de AKS no tenga acceso.

```azurecli-interactive
$ az snapshot create \
    --resource-group MC_myResourceGroup_myAKSCluster_eastus \
    --name pvcSnapshot \
    --source /subscriptions/<guid>/resourceGroups/MC_myResourceGroup_myAKSCluster_eastus/providers/MicrosoftCompute/disks/kubernetes-dynamic-pvc-faf0f176-8b8d-11e8-923b-deb28c58d242
```

Según la cantidad de datos en el disco, la instantánea puede tardar unos minutos en crearse.

## <a name="restore-and-use-a-snapshot"></a>Restauración y uso de una instantánea

Para restaurar el disco y usarlo con un pod de Kubernetes, utilice la instantánea como un origen al crear un disco con [az disk create][az-disk-create]. Esta operación conserva el recurso original si luego necesita tener acceso a la instantánea de datos original. En el ejemplo siguiente se crea un disco llamado *pvcRestored* desde la instantánea denominada *pvcSnapshot*:

```azurecli-interactive
az disk create --resource-group MC_myResourceGroup_myAKSCluster_eastus --name pvcRestored --source pvcSnapshot
```

Para usar el disco restaurado con un pod, especifique el identificador del disco en el manifiesto. Obtenga el identificador del disco con el comando [az disk show][az-disk-show]. En el ejemplo siguiente se obtiene el identificador del disco para *pvcRestored* creado en el paso anterior:

```azurecli-interactive
az disk show --resource-group MC_myResourceGroup_myAKSCluster_eastus --name pvcRestored --query id -o tsv
```

Cree un manifiesto de pod llamado `azure-restored.yaml` y especifique el URI de disco obtenido en el paso anterior. En el ejemplo siguiente se crea un servidor web NGINX básico, con el disco restaurado montado como un volumen en */mnt/azure*:

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypodrestored
spec:
  containers:
  - name: mypodrestored
    image: nginx:1.15.5
    resources:
      requests:
        cpu: 100m
        memory: 128Mi
      limits:
        cpu: 250m
        memory: 256Mi
    volumeMounts:
    - mountPath: "/mnt/azure"
      name: volume
  volumes:
    - name: volume
      azureDisk:
        kind: Managed
        diskName: pvcRestored
        diskURI: /subscriptions/<guid>/resourceGroups/MC_myResourceGroupAKS_myAKSCluster_eastus/providers/Microsoft.Compute/disks/pvcRestored
```

Cree el pod con el comando [kubectl apply][kubectl-apply], tal como se muestra en el ejemplo siguiente:

```console
$ kubectl apply -f azure-restored.yaml

pod/mypodrestored created
```

Puede usar `kubectl describe pod mypodrestored` para ver los detalles del pod, como el siguiente ejemplo reducido que muestra la información de volumen:

```console
$ kubectl describe pod mypodrestored

[...]
Volumes:
  volume:
    Type:         AzureDisk (an Azure Data Disk mount on the host and bind mount to the pod)
    DiskName:     pvcRestored
    DiskURI:      /subscriptions/19da35d3-9a1a-4f3b-9b9c-3c56ef409565/resourceGroups/MC_myResourceGroupAKS_myAKSCluster_eastus/providers/Microsoft.Compute/disks/pvcRestored
    Kind:         Managed
    FSType:       ext4
    CachingMode:  ReadWrite
    ReadOnly:     false
[...]
```

## <a name="next-steps"></a>Pasos siguientes

Para consultar los procedimientos recomendados asociados, consulte [Procedimientos recomendados para el almacenamiento y las copias de seguridad en Azure Kubernetes Service (AKS)][operator-best-practices-storage].

Obtenga más información sobre los volúmenes persistentes de Kubernetes con discos de Azure.

> [!div class="nextstepaction"]
> [Complemento de Kubernetes para discos de Azure][azure-disk-volume]

<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
[managed-disk-pricing-performance]: https://azure.microsoft.com/pricing/details/managed-disks/

<!-- LINKS - internal -->
[azure-disk-volume]: azure-disk-volume.md
[azure-files-pvc]: azure-files-dynamic-pv.md
[premium-storage]: ../virtual-machines/disks-types.md
[az-disk-list]: /cli/azure/disk#az-disk-list
[az-snapshot-create]: /cli/azure/snapshot#az-snapshot-create
[az-disk-create]: /cli/azure/disk#az-disk-create
[az-disk-show]: /cli/azure/disk#az-disk-show
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
[storage-class-concepts]: concepts-storage.md#storage-classes
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
