---
title: Uso de controladores de interfaz de almacenamiento de contenedores (CSI) para discos de Azure en Azure Kubernetes Service (AKS)
description: Aprenda a usar los controladores de interfaz de almacenamiento de contenedores (CSI) para discos de Azure en un clúster de Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 08/27/2020
author: palma21
ms.openlocfilehash: edb38b0884629ebddb646df9d12d8b2e8d07b403
ms.sourcegitcommit: 07166a1ff8bd23f5e1c49d4fd12badbca5ebd19c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90089554"
---
# <a name="use-the-azure-disk-container-storage-interface-csi-drivers-in-azure-kubernetes-service-aks-preview"></a>Uso de los controladores de interfaz de almacenamiento de contenedores (CSI) para discos de Azure en Azure Kubernetes Service (AKS) (versión preliminar)
Un controlador de interfaz de almacenamiento de contenedores (CSI) para discos de Azure es un controlador compatible con la [especificación CSI](https://github.com/container-storage-interface/spec/blob/master/spec.md) que usa Azure Kubernetes Service (AKS) para administrar el ciclo de vida de los discos de Azure.

CSI es un estándar para exponer sistemas de almacenamiento de archivos y bloques arbitrarios a cargas de trabajo en contenedores en Kubernetes. Gracias a la adopción y al uso de CSI, AKS puede escribir, implementar e iterar complementos para exponer nuevos sistemas de almacenamiento o mejorar los existentes en Kubernetes sin tener que tocar el código principal de Kubernetes ni esperar a sus ciclos de versión.

Para crear un clúster de AKS con compatibilidad con controladores CSI, vea [Habilitación de controladores de almacenamiento CSI para discos de Azure y Azure Files en AKS](csi-storage-drivers.md).

>[!NOTE]
> Los *controladores en árbol* hacen referencia a los controladores de almacenamiento actuales que forman parte del código principal de Kubernetes frente a los nuevos controladores CSI, que son complementos.

## <a name="use-csi-persistent-volumes-with-azure-disks"></a>Uso de volúmenes persistentes de CSI con discos de Azure

Un [volumen persistente](concepts-storage.md#persistent-volumes) (PV) representa un fragmento de almacenamiento aprovisionado para su uso con pods de Kubernetes. Un PV puede usarse en uno o varios pods y puede aprovisionarse de forma dinámica o estática. En este artículo se muestra cómo crear PV de forma dinámica con discos de Azure para usarlos en un solo pod de un clúster de AKS. En el caso del aprovisionamiento estático, vea [Creación manual y uso de un volumen con discos de Azure](azure-disk-volume.md).

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

Para más información sobre los volúmenes de Kubernetes, consulte [Opciones de almacenamiento para aplicaciones en AKS][concepts-storage].

## <a name="dynamically-create-azure-disk-pvs-by-using-the-built-in-storage-classes"></a>Creación dinámica de PV para discos de Azure mediante las clases de almacenamiento integradas

Una clase de almacenamiento se usa para definir cómo se crea dinámicamente una unidad de almacenamiento con un volumen persistente. Para obtener más información sobre las clases de almacenamiento de Kubernetes, vea [Clases de almacenamiento de Kubernetes][kubernetes-storage-classes]. Cuando se usan controladores de CSI de almacenamiento en AKS, hay dos `StorageClasses` integradas adicionales que usan los controladores de almacenamiento de CSI para discos de Azure. Las clases de almacenamiento de CSI adicionales se crean con el clúster junto con las clases de almacenamiento predeterminadas en árbol.

- `managed-csi`: usa almacenamiento con redundancia local (LRS) SSD estándar de Azure para crear un disco administrado.
- `managed-csi-premium`: usa LRS Premium de Azure para crear un disco administrado.

La directiva de recuperación de ambas clases de almacenamiento garantiza que el disco de Azure subyacente se elimine cuando se elimine el PV correspondiente. Las clases de almacenamiento también configuran los PV de modo que se puedan expandir. Solo tiene que editar la notificación de volumen persistente (PVC) con el nuevo tamaño.

Para aprovechar estas clases de almacenamiento, cree una [PVC](concepts-storage.md#persistent-volume-claims) y el pod correspondiente que haga referencia a ellas y las use. Una PVC se usa para aprovisionar automáticamente el almacenamiento en función de una clase de almacenamiento. Una PVC puede usar una de las clases de almacenamiento creadas previamente o una clase de almacenamiento definida por el usuario para crear un disco administrado de Azure para el tamaño y el SKU deseados. Cuando se crea una definición de pod, se especifica la PVC para solicitar el almacenamiento deseado.

Cree un pod de ejemplo y la PVC respectiva con el comando [kubectl apply][kubectl-apply]:

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/pvc-azuredisk-csi.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/nginx-pod-azuredisk.yaml

persistentvolumeclaim/pvc-azuredisk created
pod/nginx-azuredisk created
```

Una vez que el pod esté en ejecución, cree un nuevo archivo denominado `test.txt`.

```bash
$ kubectl exec nginx-azuredisk -- touch /mnt/azuredisk/test.txt
```

Ahora puede comprobar que el disco se ha montado correctamente; para ello, ejecute el siguiente comando y compruebe que ve el archivo `test.txt` en la salida:

```console
$ kubectl exec nginx-azuredisk -- ls /mnt/azuredisk

lost+found
outfile
test.txt
```

## <a name="create-a-custom-storage-class"></a>Creación de una clase de almacenamiento personalizada

Las clases de almacenamiento predeterminadas se adaptan a los escenarios más comunes, pero no a todos. En algunos casos, puede que quiera tener una clase de almacenamiento propia personalizada con sus propios parámetros. Por ejemplo, hay un escenario en el que es posible que quiera cambiar la clase `volumeBindingMode`.

Las clases de almacenamiento predeterminadas usan una clase `volumeBindingMode: Immediate` que garantiza su ejecución inmediatamente después de crear la PVC. En los casos en los que los grupos de nodos tienen restricciones de topología (por ejemplo, mediante zonas de disponibilidad), los PV se enlazarían o aprovisionarían sin conocimiento de los requisitos de programación del pod (en este caso, estar en una zona determinada).

Para resolver este escenario, puede usar `volumeBindingMode: WaitForFirstConsumer`, que retrasa el enlace y el aprovisionamiento de un PV hasta que se crea un pod que usa la PVC. De esta manera, el PV se ajusta a la zona de disponibilidad (u otra topología) especificada mediante las restricciones de programación del pod y se aprovisiona en ella.

Cree un archivo denominado `sc-azuredisk-csi-waitforfirstconsumer.yaml` y pegue el siguiente manifiesto.
La clase de almacenamiento es la misma que la clase de almacenamiento `managed-csi`, pero con una clase `volumeBindingMode` diferente.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: azuredisk-csi-waitforfirstconsumer
provisioner: disk.csi.azure.com
parameters:
  skuname: StandardSSD_LRS 
allowVolumeExpansion: true
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

Cree la clase de almacenamiento con el comando [kubectl apply][kubectl-apply] y especifique el archivo `sc-azuredisk-csi-waitforfirstconsumer.yaml`:

```console
$ kubectl apply -f sc-azuredisk-csi-waitforfirstconsumer.yaml

storageclass.storage.k8s.io/azuredisk-csi-waitforfirstconsumer created
```

## <a name="volume-snapshots"></a>Instantáneas de volumen

El controlador de CSI para discos de Azure admite la creación de [instantáneas de volúmenes persistentes](https://kubernetes-csi.github.io/docs/snapshot-restore-feature.html). Como parte de esta capacidad, el controlador puede realizar [instantáneas *incrementales*](../virtual-machines/windows/disks-incremental-snapshots.md) o *completas* en función del valor establecido en el parámetro `incremental` (de manera predeterminada, es true).

Para obtener detalles sobre todos los parámetros, vea [parámetros de clase de instantánea de volumen](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/docs/driver-parameters.md#volumesnapshotclass).

### <a name="create-a-volume-snapshot"></a>Creación de una instantánea de volumen

Para obtener un ejemplo de esta capacidad, cree una [clase de instantánea de volumen](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/snapshot/storageclass-azuredisk-snapshot.yaml) con el comando [kubectl apply][kubectl-apply]:

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/snapshot/storageclass-azuredisk-snapshot.yaml

volumesnapshotclass.snapshot.storage.k8s.io/csi-azuredisk-vsc created
```

Ahora vamos a crear una [instantánea de volumen](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/snapshot/azuredisk-volume-snapshot.yaml) de la PVC [creada dinámicamente al principio de este tutorial](#dynamically-create-azure-disk-pvs-by-using-the-built-in-storage-classes), `pvc-azuredisk`.


```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/snapshot/azuredisk-volume-snapshot.yaml

volumesnapshot.snapshot.storage.k8s.io/azuredisk-volume-snapshot created
```

Compruebe que la instantánea se ha creado correctamente:

```bash
$ kubectl describe volumesnapshot azuredisk-volume-snapshot

Name:         azuredisk-volume-snapshot
Namespace:    default
Labels:       <none>
Annotations:  API Version:  snapshot.storage.k8s.io/v1beta1
Kind:         VolumeSnapshot
Metadata:
  Creation Timestamp:  2020-08-27T05:27:58Z
  Finalizers:
    snapshot.storage.kubernetes.io/volumesnapshot-as-source-protection
    snapshot.storage.kubernetes.io/volumesnapshot-bound-protection
  Generation:        1
  Resource Version:  714582
  Self Link:         /apis/snapshot.storage.k8s.io/v1beta1/namespaces/default/volumesnapshots/azuredisk-volume-snapshot
  UID:               dd953ab5-6c24-42d4-ad4a-f33180e0ef87
Spec:
  Source:
    Persistent Volume Claim Name:  pvc-azuredisk
  Volume Snapshot Class Name:      csi-azuredisk-vsc
Status:
  Bound Volume Snapshot Content Name:  snapcontent-dd953ab5-6c24-42d4-ad4a-f33180e0ef87
  Creation Time:                       2020-08-31T05:27:59Z
  Ready To Use:                        true
  Restore Size:                        10Gi
Events:                                <none>
```

### <a name="create-a-new-pvc-based-on-a-volume-snapshot"></a>Creación de una nueva PVC basada en una instantánea de volumen

Puede crear una nueva PVC basada en una instantánea de volumen. Use la instantánea creada en el paso anterior y cree una [nueva PVC](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/snapshot/pvc-azuredisk-snapshot-restored.yaml) y un [nuevo pod](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/snapshot/nginx-pod-restored-snapshot.yaml) para consumirla.

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/snapshot/pvc-azuredisk-snapshot-restored.yaml

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/snapshot/nginx-pod-restored-snapshot.yaml

persistentvolumeclaim/pvc-azuredisk-snapshot-restored created
pod/nginx-restored created
```

Por último, vamos a asegurarnos de que es la misma PVC creada antes; para ello, se comprueba el contenido.

```console
$ kubectl exec nginx-restored -- ls /mnt/azuredisk

lost+found
outfile
test.txt
```

Como era de esperar, todavía se puede ver el archivo `test.txt` creado anteriormente.

## <a name="clone-volumes"></a>Clonación de volúmenes

Un volumen clonado se define como un duplicado de un volumen de Kubernetes existente. Para obtener más información sobre la clonación de volúmenes en Kubernetes, vea la documentación conceptual sobre [clonación de volúmenes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#volume-cloning).

El controlador de CSI para discos de Azure admite la clonación de volúmenes. Para demostrarlo, cree un [volumen clonado](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/cloning/nginx-pod-restored-cloning.yaml) de la `azuredisk-pvc` [creada anteriormente](#dynamically-create-azure-disk-pvs-by-using-the-built-in-storage-classes) y [un nuevo pod para consumirla](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/cloning/nginx-pod-restored-cloning.yaml).


```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/cloning/pvc-azuredisk-cloning.yaml

$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/cloning/nginx-pod-restored-cloning.yaml

persistentvolumeclaim/pvc-azuredisk-cloning created
pod/nginx-restored-cloning created
```

Ahora se puede comprobar el contenido del volumen clonado mediante la ejecución del siguiente comando y la confirmación de que todavía se ve el archivo creado `test.txt`.

```console
$ kubectl exec nginx-restored-cloning -- ls /mnt/azuredisk

lost+found
outfile
test.txt
```

## <a name="resize-a-persistent-volume"></a>Cambio de tamaño de un volumen persistente

También puede solicitar un volumen mayor para una PVC. Edite el objeto PVC y especifique un tamaño mayor. Este cambio desencadena la expansión del volumen subyacente que respalda el PV.

> [!NOTE]
> Nunca se crea un nuevo PV para satisfacer la notificación, sino que se cambia el tamaño de un volumen existente.

En AKS, la clase de almacenamiento `managed-csi` integrada ya permite la expansión, así que use la [PVC creada anteriormente con esta clase de almacenamiento](#dynamically-create-azure-disk-pvs-by-using-the-built-in-storage-classes). La PVC ha solicitado un volumen persistente de 10 GB. Eso se puede confirmar mediante la ejecución de:

```console 
$ kubectl exec -it nginx-azuredisk -- df -h /mnt/azuredisk

Filesystem      Size  Used Avail Use% Mounted on
/dev/sdc        9.8G   42M  9.8G   1% /mnt/azuredisk
```

> [!IMPORTANT]
> Actualmente, el controlador de CSI para discos de Azure solo admite el cambio de tamaño de PVC sin ningún pod asociado (y cuyo volumen no esté montado en un nodo específico).

Por lo tanto, vamos a eliminar el pod creado anteriormente:

```console
$ kubectl delete -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/nginx-pod-azuredisk.yaml

pod "nginx-azuredisk" deleted
```

Vamos a expandir la PVC mediante el aumento del campo `spec.resources.requests.storage`:

```console
$ kubectl patch pvc pvc-azuredisk --type merge --patch '{"spec": {"resources": {"requests": {"storage": "15Gi"}}}}'

persistentvolumeclaim/pvc-azuredisk patched
```

Vamos a confirmar que el volumen ahora es más grande:

```console
$ kubectl get pv

NAME                                       CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                                     STORAGECLASS   REASON   AGE
pvc-391ea1a6-0191-4022-b915-c8dc4216174a   15Gi       RWO            Delete           Bound    default/pvc-azuredisk                     managed-csi             2d2h
(...)
```

> [!NOTE]
> La PVC no refleja el nuevo tamaño hasta que tiene un pod asociado de nuevo.

Vamos a crear un nuevo pod:

```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/nginx-pod-azuredisk.yaml

pod/nginx-azuredisk created
```

Por último, confirme el tamaño de la PVC y dentro del pod:
```console
$ kubectl get pvc pvc-azuredisk
NAME            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-azuredisk   Bound    pvc-391ea1a6-0191-4022-b915-c8dc4216174a   15Gi       RWO            managed-csi    2d2h

$ kubectl exec -it nginx-azuredisk -- df -h /mnt/azuredisk
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdc         15G   46M   15G   1% /mnt/azuredisk
```

<!--- ## Shared disk

[Azure shared disks](../virtual-machines/windows/disks-shared.md) is an Azure managed disks feature that enables attaching an Azure disk to agent nodes simultaneously. Attaching a managed disk to multiple agent nodes allows you, for example, to deploy new or migrate existing clustered applications to Azure.

> [!IMPORTANT] Currently, only raw block device (`volumeMode: Block`) is supported by the Azure disk CSI driver. Applications should manage the coordination and control of writes, reads, locks, caches, mounts, and fencing on the shared disk, which is exposed as a raw block device.

Let's create a file called `shared-disk.yaml` by copying the following command that contains the shared disk storage class and PVC:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-csi-shared
provisioner: disk.csi.azure.com
parameters:
  skuname: Premium_LRS  # Currently shared disk is only available with premium SSD
  maxShares: "2"
  cachingMode: None  # ReadOnly cache is not available for premium SSD with maxShares>1
reclaimPolicy: Delete
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-azuredisk-shared
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 256Gi  # minimum size of shared disk is 256GB (P15)
  volumeMode: Block
  storageClassName: managed-csi-shared
```

Create the storage class with the [kubectl apply][kubectl-apply] command, and specify your `shared-disk.yaml` file:

```console
$ kubectl apply -f shared-disk.yaml

storageclass.storage.k8s.io/managed-csi-shared created
persistentvolumeclaim/pvc-azuredisk-shared created
``` 

Now let's create a file called `deployment-shared.yml` by copying the following command:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: deployment-azuredisk
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
      name: deployment-azuredisk
    spec:
      containers:
        - name: deployment-azuredisk
          image: nginx
          volumeDevices:
            - name: azuredisk
              devicePath: /dev/sdx
      volumes:
        - name: azuredisk
          persistentVolumeClaim:
            claimName: pvc-azuredisk-shared
```

Create the deployment with the [kubectl apply][kubectl-apply] command, and specify your `deployment-shared.yml` file:

```console
$ kubectl apply -f deployment-shared.yml

deployment/deployment-azuredisk created
```

Finally, let's check the block device inside the pod:

```console
# kubectl exec -it deployment-sharedisk-7454978bc6-xh7jp bash
root@deployment-sharedisk-7454978bc6-xh7jp:/# dd if=/dev/zero of=/dev/sdx bs=1024k count=100
100+0 records in
100+0 records out
104857600 bytes (105 MB, 100 MiB) copied, 0.0502999 s, 2.1 GB/s
```
-->

## <a name="windows-containers"></a>Contenedores de Windows

El controlador de CSI para discos de Azure también admite nodos y contenedores de Windows. Si quiere usar contenedores de Windows, siga el [tutorial sobre contenedores de Windows](windows-container-cli.md) para agregar un grupo de nodos de Windows.

Una vez que tenga un grupo de nodos de Windows, puede usar las clases de almacenamiento integradas como `managed-csi`. Puede implementar un [conjunto con estado basado en Windows](https://github.com/kubernetes-sigs/azuredisk-csi-driver/blob/master/deploy/example/windows/statefulset.yaml) de ejemplo que guarde marcas de tiempo en el archivo `data.txt` mediante la implementación del comando siguiente con el comando [kubectl apply][kubectl-apply]:

 ```console
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/azuredisk-csi-driver/master/deploy/example/windows/statefulset.yaml

statefulset.apps/busybox-azuredisk created
```

Ahora puede validar el contenido del volumen mediante la ejecución de:

```console
$ kubectl exec -it busybox-azuredisk-0 -- cat c:\\mnt\\azuredisk\\data.txt # on Linux/MacOS Bash
$ kubectl exec -it busybox-azuredisk-0 -- cat c:\mnt\azuredisk\data.txt # on Windows Powershell/CMD

2020-08-27 08:13:41Z
2020-08-27 08:13:42Z
2020-08-27 08:13:44Z
(...)
```

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre cómo usar controladores de CSI para Azure Files, vea [Uso de Azure Files con controladores de CSI](azure-files-csi.md).
- Para obtener más información sobre los procedimientos recomendados de almacenamiento, vea [Procedimientos recomendados para el almacenamiento y las copias de seguridad en Azure Kubernetes Service (AKS)][operator-best-practices-storage].


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
[premium-storage]: ../virtual-machines/windows/disks-types.md
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
[az-extension-add]: /cli/azure/extension?view=azure-cli-latest#az-extension-add
[az-extension-update]: /cli/azure/extension?view=azure-cli-latest#az-extension-update
[az-feature-register]: /cli/azure/feature?view=azure-cli-latest#az-feature-register
[az-feature-list]: /cli/azure/feature?view=azure-cli-latest#az-feature-list
[az-provider-register]: /cli/azure/provider?view=azure-cli-latest#az-provider-register