---
title: Uso de las GPU en Azure Kubernetes Service (AKS)
description: Obtenga información acerca de cómo usar GPU para cargas de trabajo de cálculo de alto rendimiento o de uso intensivo de gráficos en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 08/21/2020
ms.author: jpalma
author: palma21
ms.openlocfilehash: 4dfaa329dd0472b52de2d3306e6a3b61f660e666
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89443065"
---
# <a name="use-gpus-for-compute-intensive-workloads-on-azure-kubernetes-service-aks"></a>Uso de GPU para cargas de trabajo de cálculo intensivo en Azure Kubernetes Service (AKS)

Por lo general, las unidades de procesamiento gráfico (GPU) se usan para cargas de trabajo de cálculo intensivo, como cargas de trabajo de visualización y gráficos. AKS admite la creación de grupos de nodos compatibles con GPU para ejecutar estas cargas de trabajo de cálculo intensivo en Kubernetes. Para obtener más información sobre las máquinas virtuales habilitadas para GPU disponibles, consulte [Tamaños de máquinas virtuales optimizadas para GPU en Azure][gpu-skus]. Para nodos AKS, se recomienda un tamaño mínimo de *Standard_NC6*.

> [!NOTE]
> Las máquinas virtuales habilitadas para GPU contienen hardware especializado que está sujeto a una mayor disponibilidad de precios y región. Para obtener más información, consulte la herramienta de [precios][azure-pricing] y la [disponibilidad de regiones][azure-availability].

Actualmente, el uso de grupos de nodos habilitados para GPU solo está disponible para grupos de nodos de Linux.

## <a name="before-you-begin"></a>Antes de empezar

En este artículo se supone que ya tiene un clúster de AKS con nodos que admiten GPU. El clúster de AKS debe ejecutar Kubernetes 1.10 o una versión posterior. Si necesita un clúster de AKS que cumpla estos requisitos, consulte la primera sección de este artículo para [crear un clúster de AKS](#create-an-aks-cluster).

También es preciso tener instalada y configurada la versión 2.0.64 de la CLI de Azure u otra versión posterior. Ejecute  `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte  [Install Azure CLI][install-azure-cli] (Instalación de la CLI de Azure).

## <a name="create-an-aks-cluster"></a>Creación de un clúster de AKS

Si necesita un clúster de AKS que cumpla los requisitos mínimos (nodo habilitado para GPU y Kubernetes versión 1.10 o posterior), siga estos pasos. Si ya tiene un clúster de AKS que cumpla estos requisitos, [vaya a la sección siguiente](#confirm-that-gpus-are-schedulable).

Primero, cree un grupo de recursos para el clúster con el comando [az group create][az-group-create]. En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la región *eastus*:

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Ahora cree un clúster de AKS con el comando [az aks create][az-aks-create]. En el ejemplo siguiente se crea un clúster con un solo nodo de tamaño `Standard_NC6`:

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --node-vm-size Standard_NC6 \
    --node-count 1
```

Obtenga las credenciales para el clúster de AKS mediante el comando [az aks get-credentials][az-aks-get-credentials]:

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myAKSCluster
```

## <a name="install-nvidia-device-plugin"></a>Instalación del complemento de dispositivo NVIDIA

Para poder usar las GPU en los nodos, debe implementar un elemento DaemonSet para el complemento de dispositivo NVIDIA. Este objeto DaemonSet ejecuta un pod en cada nodo para proporcionar los controladores requeridos para las GPU.

En primer lugar, cree un espacio de nombres mediante el comando [kubectl create namespace][kubectl-create], como *gpu-resources*:

```console
kubectl create namespace gpu-resources
```

Cree un archivo denominado *nvidia-device-plugin-ds.yaml* y pegue el siguiente manifiesto YAML. Este manifiesto se proporciona como parte del [complemento de dispositivo NVIDIA para el proyecto de Kubernetes][nvidia-github].

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: nvidia-device-plugin-daemonset
  namespace: gpu-resources
spec:
  selector:
    matchLabels:
      name: nvidia-device-plugin-ds
  updateStrategy:
    type: RollingUpdate
  template:
    metadata:
      # Mark this pod as a critical add-on; when enabled, the critical add-on scheduler
      # reserves resources for critical add-on pods so that they can be rescheduled after
      # a failure.  This annotation works in tandem with the toleration below.
      annotations:
        scheduler.alpha.kubernetes.io/critical-pod: ""
      labels:
        name: nvidia-device-plugin-ds
    spec:
      tolerations:
      # Allow this pod to be rescheduled while the node is in "critical add-ons only" mode.
      # This, along with the annotation above marks this pod as a critical add-on.
      - key: CriticalAddonsOnly
        operator: Exists
      - key: nvidia.com/gpu
        operator: Exists
        effect: NoSchedule
      containers:
      - image: nvidia/k8s-device-plugin:1.11
        name: nvidia-device-plugin-ctr
        securityContext:
          allowPrivilegeEscalation: false
          capabilities:
            drop: ["ALL"]
        volumeMounts:
          - name: device-plugin
            mountPath: /var/lib/kubelet/device-plugins
      volumes:
        - name: device-plugin
          hostPath:
            path: /var/lib/kubelet/device-plugins
```

Ahora, use el comando [kubectl apply][kubectl-apply] para crear el elemento DaemonSet y confirme que el complemento de dispositivo NVIDIA se crea correctamente, tal como se muestra en la salida de ejemplo siguiente:

```console
$ kubectl apply -f nvidia-device-plugin-ds.yaml

daemonset "nvidia-device-plugin" created
```

## <a name="use-the-aks-specialized-gpu-image-preview"></a>Uso de la imagen de GPU especializada de AKS (versión preliminar)

Como alternativa a estos pasos, AKS proporciona una imagen de AKS totalmente configurada que ya contiene el [complemento de dispositivo NVIDIA para Kubernetes][nvidia-github].

> [!WARNING]
> No debe instalar manualmente el conjunto de demonio del complemento de dispositivo NVIDIA para clústeres mediante la nueva imagen de GPU especializada de AKS.


Registre la característica `GPUDedicatedVHDPreview`:

```azurecli
az feature register --name GPUDedicatedVHDPreview --namespace Microsoft.ContainerService
```

Pueden pasar unos minutos hasta que el estado aparezca como **Registrado**. Puede comprobar el estado del registro con el comando [az feature list](/cli/azure/feature?view=azure-cli-latest#az-feature-list):

```azurecli
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/GPUDedicatedVHDPreview')].{Name:name,State:properties.state}"
```

Cuando el estado se muestre como Registrado, actualice el registro del proveedor de recursos `Microsoft.ContainerService` mediante el comando [az provider register](/cli/azure/provider?view=azure-cli-latest#az-provider-register):

```azurecli
az provider register --namespace Microsoft.ContainerService
```

Para instalar la extensión aks-preview de la CLI, use los siguientes comandos de la CLI de Azure:

```azurecli
az extension add --name aks-preview
```

Para actualizar la extensión aks-preview de la CLI, use los siguientes comandos de la CLI de Azure:

```azurecli
az extension update --name aks-preview
```

### <a name="use-the-aks-specialized-gpu-image-on-new-clusters-preview"></a>Uso de la imagen de GPU especializada de AKS en clústeres nuevos (versión preliminar)    

Configure el clúster para usar la imagen de GPU especializada de AKS cuando se cree el clúster. Use la marca `--aks-custom-headers` para los nodos del agente GPU en el nuevo clúster para usar la imagen de GPU especializada de AKS.

```azurecli
az aks create --name myAKSCluster --resource-group myResourceGroup --node-vm-size Standard_NC6 --node-count 1 --aks-custom-headers UseGPUDedicatedVHD=true
```

Para crear un clúster con imágenes normales de AKS, puede omitir la etiqueta `--aks-custom-headers` personalizada. También puede optar por añadir más grupos de nodos de GPU especializados, tal y como se indica a continuación.


### <a name="use-the-aks-specialized-gpu-image-on-existing-clusters-preview"></a>Uso de la imagen de GPU especializada de AKS en clústeres existentes (versión preliminar)

Configure un grupo de nodos nuevo para usar la imagen de GPU especializada de AKS. Use la marca de `--aks-custom-headers` para los nodos del agente de GPU en el nuevo grupo de nodos para usar la imagen de GPU especializada de AKS.

```azurecli
az aks nodepool add --name gpu --cluster-name myAKSCluster --resource-group myResourceGroup --node-vm-size Standard_NC6 --node-count 1 --aks-custom-headers UseGPUDedicatedVHD=true
```

Para crear un grupo de nodos mediante imágenes AKS normales, puede omitir la etiqueta `--aks-custom-headers` personalizada. 

> [!NOTE]
> Si la SKU de GPU requiere máquinas virtuales de generación 2, para crearlas puede:
> ```azurecli
> az aks nodepool add --name gpu --cluster-name myAKSCluster --resource-group myResourceGroup --node-vm-size Standard_NC6s_v2 --node-count 1 --aks-custom-headers UseGPUDedicatedVHD=true,usegen2vm=true
> ```

## <a name="confirm-that-gpus-are-schedulable"></a>Confirmación de que las GPU son programables

Con el clúster de AKS creado, confirme que las GPU son programables en Kubernetes. Primero, enumere los nodos del clúster con el comando [kubectl get nodes][kubectl-get]:

```console
$ kubectl get nodes

NAME                       STATUS   ROLES   AGE   VERSION
aks-nodepool1-28993262-0   Ready    agent   13m   v1.12.7
```

Ahora, use el comando [kubectl describe node][kubectl-describe] para confirmar que las GPU son programables. En la sección *Capacidad*, la GPU debe aparecer como `nvidia.com/gpu:  1`.

En el siguiente ejemplo reducido se muestra que una GPU está disponible en el nodo denominado *aks-nodepool1-18821093-0*:

```console
$ kubectl describe node aks-nodepool1-28993262-0

Name:               aks-nodepool1-28993262-0
Roles:              agent
Labels:             accelerator=nvidia

[...]

Capacity:
 attachable-volumes-azure-disk:  24
 cpu:                            6
 ephemeral-storage:              101584140Ki
 hugepages-1Gi:                  0
 hugepages-2Mi:                  0
 memory:                         57713784Ki
 nvidia.com/gpu:                 1
 pods:                           110
Allocatable:
 attachable-volumes-azure-disk:  24
 cpu:                            5916m
 ephemeral-storage:              93619943269
 hugepages-1Gi:                  0
 hugepages-2Mi:                  0
 memory:                         51702904Ki
 nvidia.com/gpu:                 1
 pods:                           110
System Info:
 Machine ID:                 b0cd6fb49ffe4900b56ac8df2eaa0376
 System UUID:                486A1C08-C459-6F43-AD6B-E9CD0F8AEC17
 Boot ID:                    f134525f-385d-4b4e-89b8-989f3abb490b
 Kernel Version:             4.15.0-1040-azure
 OS Image:                   Ubuntu 16.04.6 LTS
 Operating System:           linux
 Architecture:               amd64
 Container Runtime Version:  docker://1.13.1
 Kubelet Version:            v1.12.7
 Kube-Proxy Version:         v1.12.7
PodCIDR:                     10.244.0.0/24
ProviderID:                  azure:///subscriptions/<guid>/resourceGroups/MC_myResourceGroup_myAKSCluster_eastus/providers/Microsoft.Compute/virtualMachines/aks-nodepool1-28993262-0
Non-terminated Pods:         (9 in total)
  Namespace                  Name                                     CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                  ----                                     ------------  ----------  ---------------  -------------  ---
  kube-system                nvidia-device-plugin-daemonset-bbjlq     0 (0%)        0 (0%)      0 (0%)           0 (0%)         2m39s

[...]
```

## <a name="run-a-gpu-enabled-workload"></a>Ejecución de una carga de trabajo habilitada para GPU

Para ver la GPU en acción, programe una carga de trabajo compatible con la GPU con la solicitud de recursos adecuada. En este ejemplo se va a ejecutar un trabajo [Tensorflow](https://www.tensorflow.org/) en el [conjunto de datos MNIST](http://yann.lecun.com/exdb/mnist/).

Cree un archivo denominado *samples-tf-mnist-demo.yaml* y pegue el siguiente manifiesto YAML. El siguiente manifiesto de trabajo incluye un límite de recursos de `nvidia.com/gpu: 1`:

> [!NOTE]
> Si recibe un error de coincidencia de versión al llamar a los controladores, como, por ejemplo, que la versión del controlador CUDA no es suficiente para la versión en tiempo de ejecución de CUDA, revise el gráfico de compatibilidad de la matriz de controladores de NVIDIA: [https://docs.nvidia.com/deploy/cuda-compatibility/index.html](https://docs.nvidia.com/deploy/cuda-compatibility/index.html).

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  labels:
    app: samples-tf-mnist-demo
  name: samples-tf-mnist-demo
spec:
  template:
    metadata:
      labels:
        app: samples-tf-mnist-demo
    spec:
      containers:
      - name: samples-tf-mnist-demo
        image: microsoft/samples-tf-mnist-demo:gpu
        args: ["--max_steps", "500"]
        imagePullPolicy: IfNotPresent
        resources:
          limits:
           nvidia.com/gpu: 1
      restartPolicy: OnFailure
```

Use el comando [kubectl apply][kubectl-apply] para ejecutar el trabajo. Este comando analiza el archivo de manifiesto y crea los objetos de Kubernetes definidos:

```console
kubectl apply -f samples-tf-mnist-demo.yaml
```

## <a name="view-the-status-and-output-of-the-gpu-enabled-workload"></a>Ver el estado y la salida de la carga de trabajo habilitada para GPU

Supervise el progreso del trabajo mediante el comando [kubectl get jobs][kubectl-get] con el argumento `--watch`. Puede tardar unos minutos en extraer primero la imagen y procesar el conjunto de datos. Cuando la columna *COMPLETIONS* (FINALIZACIONES) muestra *1/1*, significa que el trabajo ha finalizado correctamente. Salga del comando `kubetctl --watch` con *Ctrl-C*:

```console
$ kubectl get jobs samples-tf-mnist-demo --watch

NAME                    COMPLETIONS   DURATION   AGE

samples-tf-mnist-demo   0/1           3m29s      3m29s
samples-tf-mnist-demo   1/1   3m10s   3m36s
```

Para ver la salida de la carga de trabajo habilitada para GPU, obtenga primero el nombre del pod con el comando [kubectl get pods][kubectl-get]:

```console
$ kubectl get pods --selector app=samples-tf-mnist-demo

NAME                          READY   STATUS      RESTARTS   AGE
samples-tf-mnist-demo-mtd44   0/1     Completed   0          4m39s
```

Ahora, use el comando [kubectl logs][kubectl-logs] para ver los registros del pod. En el siguiente ejemplo, los registros de pod confirman que se ha descubierto el dispositivo GPU adecuado, `Tesla K80`. Proporcione el nombre para su pod:

```console
$ kubectl logs samples-tf-mnist-demo-smnr6

2019-05-16 16:08:31.258328: I tensorflow/core/platform/cpu_feature_guard.cc:137] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
2019-05-16 16:08:31.396846: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1030] Found device 0 with properties: 
name: Tesla K80 major: 3 minor: 7 memoryClockRate(GHz): 0.8235
pciBusID: 2fd7:00:00.0
totalMemory: 11.17GiB freeMemory: 11.10GiB
2019-05-16 16:08:31.396886: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1120] Creating TensorFlow device (/device:GPU:0) -> (device: 0, name: Tesla K80, pci bus id: 2fd7:00:00.0, compute capability: 3.7)
2019-05-16 16:08:36.076962: I tensorflow/stream_executor/dso_loader.cc:139] successfully opened CUDA library libcupti.so.8.0 locally
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Extracting /tmp/tensorflow/input_data/train-images-idx3-ubyte.gz
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Extracting /tmp/tensorflow/input_data/train-labels-idx1-ubyte.gz
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Extracting /tmp/tensorflow/input_data/t10k-images-idx3-ubyte.gz
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting /tmp/tensorflow/input_data/t10k-labels-idx1-ubyte.gz
Accuracy at step 0: 0.1081
Accuracy at step 10: 0.7457
Accuracy at step 20: 0.8233
Accuracy at step 30: 0.8644
Accuracy at step 40: 0.8848
Accuracy at step 50: 0.8889
Accuracy at step 60: 0.8898
Accuracy at step 70: 0.8979
Accuracy at step 80: 0.9087
Accuracy at step 90: 0.9099
Adding run metadata for 99
Accuracy at step 100: 0.9125
Accuracy at step 110: 0.9184
Accuracy at step 120: 0.922
Accuracy at step 130: 0.9161
Accuracy at step 140: 0.9219
Accuracy at step 150: 0.9151
Accuracy at step 160: 0.9199
Accuracy at step 170: 0.9305
Accuracy at step 180: 0.9251
Accuracy at step 190: 0.9258
Adding run metadata for 199
Accuracy at step 200: 0.9315
Accuracy at step 210: 0.9361
Accuracy at step 220: 0.9357
Accuracy at step 230: 0.9392
Accuracy at step 240: 0.9387
Accuracy at step 250: 0.9401
Accuracy at step 260: 0.9398
Accuracy at step 270: 0.9407
Accuracy at step 280: 0.9434
Accuracy at step 290: 0.9447
Adding run metadata for 299
Accuracy at step 300: 0.9463
Accuracy at step 310: 0.943
Accuracy at step 320: 0.9439
Accuracy at step 330: 0.943
Accuracy at step 340: 0.9457
Accuracy at step 350: 0.9497
Accuracy at step 360: 0.9481
Accuracy at step 370: 0.9466
Accuracy at step 380: 0.9514
Accuracy at step 390: 0.948
Adding run metadata for 399
Accuracy at step 400: 0.9469
Accuracy at step 410: 0.9489
Accuracy at step 420: 0.9529
Accuracy at step 430: 0.9507
Accuracy at step 440: 0.9504
Accuracy at step 450: 0.951
Accuracy at step 460: 0.9512
Accuracy at step 470: 0.9539
Accuracy at step 480: 0.9533
Accuracy at step 490: 0.9494
Adding run metadata for 499
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Para eliminar los objetos de Kubernetes asociados que creó en este artículo, use el comando [kubectl delete job][kubectl delete] como se muestra a continuación:

```console
kubectl delete jobs samples-tf-mnist-demo
```

## <a name="next-steps"></a>Pasos siguientes

Para ejecutar trabajos de Apache Spark, consulte [Ejecución de trabajos de Apache Spark en AKS][aks-spark].

Para obtener más información acerca de la ejecución de cargas de trabajo de aprendizaje automático (ML) en Kubernetes, vea [Kubeflow Labs][kubeflow-labs].

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubeflow-labs]: https://github.com/Azure/kubeflow-labs
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[kubectl delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[azure-pricing]: https://azure.microsoft.com/pricing/
[azure-availability]: https://azure.microsoft.com/global-infrastructure/services/
[nvidia-github]: https://github.com/NVIDIA/k8s-device-plugin

<!-- LINKS - internal -->
[az-group-create]: /cli/azure/group#az-group-create
[az-aks-create]: /cli/azure/aks#az-aks-create
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[aks-spark]: spark-job.md
[gpu-skus]: ../virtual-machines/sizes-gpu.md
[install-azure-cli]: /cli/azure/install-azure-cli
