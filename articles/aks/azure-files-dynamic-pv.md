---
title: Creación dinámica de recursos compartidos de Azure Files
titleSuffix: Azure Kubernetes Service
description: Aprenda a crear un volumen persistente de forma dinámica con Azure Files para usarlo con varios pods simultáneos en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 07/01/2020
ms.openlocfilehash: 5e9e686d8da420c650709d3bedc103d0043fa679
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87287101"
---
# <a name="dynamically-create-and-use-a-persistent-volume-with-azure-files-in-azure-kubernetes-service-aks"></a>Creación dinámica y uso de un volumen persistente con Azure Files en Azure Kubernetes Service (AKS)

Un volumen persistente representa un fragmento de almacenamiento aprovisionado para su uso con pods de Kubernetes. Un volumen persistente puede usarse en uno o varios pods y puede aprovisionarse de forma dinámica o estática. Si varios pods necesitan acceso simultáneo al mismo volumen de almacenamiento, puede usar Azure Files para conectarse mediante el [protocolo Bloque de mensajes del servidor (SMB)][smb-overview]. Este artículo muestra cómo crear dinámicamente un recurso compartido de Azure Files para ser usado por varios pods en un clúster de Azure Kubernetes Service (AKS).

Para más información sobre los volúmenes de Kubernetes, consulte [Opciones de almacenamiento para aplicaciones en AKS][concepts-storage].

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS. Si necesita un clúster de AKS, consulte el inicio rápido de AKS [mediante la CLI de Azure][aks-quickstart-cli] o [mediante Azure Portal][aks-quickstart-portal].

También es preciso que esté instalada y configurada la versión 2.0.59 de la CLI de Azure u otra versión posterior. Ejecute  `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte  [Install Azure CLI][install-azure-cli] (Instalación de la CLI de Azure).

## <a name="create-a-storage-class"></a>Creación de una clase de almacenamiento

Una clase de almacenamiento se utiliza para definir cómo se crea un recurso compartido de archivos de Azure. En el [grupo de recursos node][node-resource-group], se crea automáticamente una cuenta de almacenamiento para utilizarla con la clase de almacenamiento donde van a guardarse los recursos compartidos de archivos de Azure. Seleccione una de las siguientes [redundancias de Azure Storage][storage-skus] para *skuName*:

* *Standard_LRS*: almacenamiento estándar con redundancia local (LRS)
* *Standard_GRS*: almacenamiento estándar con redundancia geográfica (GRS)
* *Standard_ZRS*: almacenamiento estándar con redundancia de zona (ZRS)
* *Standard_RAGRS*: almacenamiento estándar con redundancia geográfica con acceso de lectura (RA-GRS)
* *Premium_LRS*: almacenamiento con redundancia local premium (LRS)

> [!NOTE]
> Azure Files admite el almacenamiento premium en clústeres de AKS que ejecutan Kubernetes 1.13 o versiones posteriores. El uso compartido de archivos premium mínimo es de 100 GB.

Para más información sobre las clases de almacenamiento de Kubernetes para Azure Files, consulte las [clases de almacenamiento de Kubernetes][kubernetes-storage-classes].

Cree un archivo denominado `azure-file-sc.yaml` y cópielo en el ejemplo siguiente de manifiesto. Para más información sobre *mountOptions*, consulte la sección [Opciones de montaje][mount-options].

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: my-azurefile
provisioner: kubernetes.io/azure-file
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict
parameters:
  skuName: Standard_LRS
```

Cree la clase de almacenamiento con el comando [kubectl apply][kubectl-apply]:

```console
kubectl apply -f azure-file-sc.yaml
```

## <a name="create-a-persistent-volume-claim"></a>Creación de una notificación de volumen persistente

Una notificación de volumen persistente (PVC) usa el objeto de clase de almacenamiento para aprovisionar de forma dinámica un recurso compartido de archivos de Azure. El siguiente código de YAML puede utilizarse para crear una notificación de volumen persistente con un tamaño de *5 GB* y con acceso *ReadWriteMany*. Para más información sobre los modos de acceso, consulte la documentación sobre [volúmenes persistentes de Kubernetes][access-modes].

Ahora cree un archivo denominado `azure-file-pvc.yaml` y cópielo en el siguiente código YAML. Asegúrese de que *storageClassName* coincide con la clase de almacenamiento creada en el último paso:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-azurefile
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: azurefile
  resources:
    requests:
      storage: 5Gi
```

> [!NOTE]
> Si usa la SKU *Premium_LRS* para la clase de almacenamiento, el valor mínimo de *storage* debe ser *100Gi*.

Cree la notificación del volumen persistente con el comando [kubectl apply][kubectl-apply]:

```console
kubectl apply -f azure-file-pvc.yaml
```

Una vez completado, se creará el recurso compartido de archivos. También se crea un secreto de Kubernetes que incluye las credenciales y la información de conexión. Puede usar el comando [kubectl get][kubectl-get] para ver el estado de la PVC:

```console
$ kubectl get pvc my-azurefile

NAME           STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS      AGE
my-azurefile   Bound     pvc-8436e62e-a0d9-11e5-8521-5a8664dc0477   5Gi        RWX            my-azurefile      5m
```

## <a name="use-the-persistent-volume"></a>Uso del volumen persistente

El siguiente código de YAML crea un pod que utiliza la notificación de volumen persistente *my-azurefile* para montar el recurso compartido de archivos de Azure en la ruta de acceso */mnt/azure*. En el caso de los contenedores de Windows Server, especifique un elemento *mountPath* con la convención de ruta de acceso de Windows, como *"D:"* .

Cree un archivo denominado `azure-pvc-files.yaml` y cópielo en el siguiente código YAML. Asegúrese de que *claimName* coincide con la clase PVC creada en el último paso.

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
        claimName: my-azurefile
```

Cree el pod con el comando [kubectl apply][kubectl-apply].

```console
kubectl apply -f azure-pvc-files.yaml
```

Ahora tiene un pod en ejecución con el recurso compartido de Azure Files montado en el directorio */mnt/azure*. Esta configuración puede verse al examinar el pod mediante `kubectl describe pod mypod`. La siguiente salida de ejemplo condensada muestra el volumen montado en el contenedor:

```
Containers:
  mypod:
    Container ID:   docker://053bc9c0df72232d755aa040bfba8b533fa696b123876108dec400e364d2523e
    Image:          nginx:1.15.5
    Image ID:       docker-pullable://nginx@sha256:d85914d547a6c92faa39ce7058bd7529baacab7e0cd4255442b04577c4d1f424
    State:          Running
      Started:      Fri, 01 Mar 2019 23:56:16 +0000
    Ready:          True
    Mounts:
      /mnt/azure from volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8rv4z (ro)
[...]
Volumes:
  volume:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  my-azurefile
    ReadOnly:   false
[...]
```

## <a name="mount-options"></a>Opciones de montaje

El valor predeterminado de *fileMode* y *dirMode* es *0777* para Kubernetes 1.13.0 y versiones posteriores. Si crea dinámicamente el volumen persistente con una clase de almacenamiento, las opciones de montaje se pueden especificar en el objeto de la clase de almacenamiento. En el ejemplo siguiente se establece *0777*:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: my-azurefile
provisioner: kubernetes.io/azure-file
mountOptions:
  - dir_mode=0777
  - file_mode=0777
  - uid=0
  - gid=0
  - mfsymlinks
  - cache=strict
parameters:
  skuName: Standard_LRS
```

## <a name="next-steps"></a>Pasos siguientes

Para consultar los procedimientos recomendados asociados, consulte [Procedimientos recomendados para el almacenamiento y las copias de seguridad en Azure Kubernetes Service (AKS)][operator-best-practices-storage].

Para obtener información sobre los parámetros de clase de almacenamiento, consulte [Aprovisionamiento dinámico](https://github.com/kubernetes-sigs/azurefile-csi-driver/blob/master/docs/driver-parameters.md#dynamic-provision).

Obtenga más información sobre los volúmenes persistentes de Kubernetes con Azure Files.

> [!div class="nextstepaction"]
> [Complemento de Kubernetes para Azure Files][kubernetes-files]

<!-- LINKS - external -->
[access-modes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubernetes-files]: https://github.com/kubernetes/examples/blob/master/staging/volumes/azure_file/README.md
[kubernetes-secret]: https://kubernetes.io/docs/concepts/configuration/secret/
[kubernetes-security-context]: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/
[kubernetes-storage-classes]: https://kubernetes.io/docs/concepts/storage/storage-classes/#azure-file
[kubernetes-volumes]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/
[pv-static]: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#static
[smb-overview]: /windows/desktop/FileIO/microsoft-smb-protocol-and-cifs-protocol-overview

<!-- LINKS - internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-group-list]: /cli/azure/group#az-group-list
[az-resource-show]: /cli/azure/aks#az-aks-show
[az-storage-account-create]: /cli/azure/storage/account#az-storage-account-create
[az-storage-create]: /cli/azure/storage/account#az-storage-account-create
[az-storage-key-list]: /cli/azure/storage/account/keys#az-storage-account-keys-list
[az-storage-share-create]: /cli/azure/storage/share#az-storage-share-create
[mount-options]: #mount-options
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[az-aks-show]: /cli/azure/aks#az-aks-show
[storage-skus]: ../storage/common/storage-redundancy.md
[kubernetes-rbac]: concepts-identity.md#role-based-access-controls-rbac
[operator-best-practices-storage]: operator-best-practices-storage.md
[concepts-storage]: concepts-storage.md
[node-resource-group]: faq.md#why-are-two-resource-groups-created-with-aks
