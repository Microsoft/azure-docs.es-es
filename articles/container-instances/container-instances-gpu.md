---
title: Implementación de una instancia de contenedor habilitada para GPU
description: Aprenda a implementar instancias de contenedor de Azure para ejecutar aplicaciones de contenedor de uso intensivo de proceso utilizando recursos de GPU.
ms.topic: article
ms.date: 07/22/2020
ms.openlocfilehash: 19240560baa0cebdb6777d7b63d8c91832b12e1a
ms.sourcegitcommit: 5b8fb60a5ded05c5b7281094d18cf8ae15cb1d55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87387103"
---
# <a name="deploy-container-instances-that-use-gpu-resources"></a>Implementación de instancias de contenedor que usan recursos de GPU

Para ejecutar determinadas cargas de trabajo de proceso intensivo en Azure Container Instances, implemente los [grupos de contenedores](container-instances-container-groups.md) con *recursos de GPU*. Las instancias de contenedor en el grupo pueden acceder a una o varias GPU NVIDIA Tesla mientras se ejecutan cargas de contenedor como CUDA y aplicaciones de aprendizaje profundo.

En este artículo se muestra cómo agregar recursos de GPU al implementar un grupo de contenedores mediante el uso de un [archivo YAML](container-instances-multi-container-yaml.md) o una [plantilla de Resource Manager](container-instances-multi-container-group.md). También puede especificar los recursos de GPU al implementar una instancia de contenedor mediante Azure Portal.

> [!IMPORTANT]
> Esta funcionalidad actualmente está en su versión preliminar y se [aplican algunas limitaciones](#preview-limitations). Las versiones preliminares están a su disposición con la condición de que acepte los [términos de uso adicionales][terms-of-use]. Es posible que algunos de los aspectos de esta característica cambien antes de ofrecer disponibilidad general.

## <a name="preview-limitations"></a>Limitaciones de vista previa

En la versión preliminar, las siguientes limitaciones se aplican al uso de recursos de GPU en grupos de contenedores. 

[!INCLUDE [container-instances-gpu-regions](../../includes/container-instances-gpu-regions.md)]

Con el tiempo se admitirán más regiones.

**Tipos de sistemas operativos compatibles**: solo Linux.

**Limitaciones adicionales**: no se pueden usar los recursos de GPU al implementar un grupo de contenedores en una [red virtual](container-instances-vnet.md).

## <a name="about-gpu-resources"></a>Acerca de los recursos de GPU

### <a name="count-and-sku"></a>Recuento y SKU

Para usar GPU en una instancia de contenedor, especifique un *recurso de GPU* con la siguiente información:

* **Recuento**: el número de GPU: **1**, **2** o **4**.
* **SKU**: SKU de GPU: **K80**, **P100** o **V100**. Cada SKU asigna la GPU NVIDIA Tesla a una de las siguientes familias de máquinas virtuales habilitadas para GPU de Azure:

  | SKU | Familia de VM |
  | --- | --- |
  | K80 | [NC](../virtual-machines/nc-series.md) |
  | P100 | [NCv2](../virtual-machines/ncv2-series.md) |
  | V100 | [NCv3](../virtual-machines/ncv3-series.md) |

[!INCLUDE [container-instances-gpu-limits](../../includes/container-instances-gpu-limits.md)]

Al implementar recursos de GPU, establezca los recursos de CPU y memoria apropiados para la carga de trabajo, hasta los valores máximos mostrados en la siguiente anterior. Estos valores actualmente son mayores que los recursos de CPU y memoria disponibles en los grupos de contenedores sin recursos de GPU.  

> [!IMPORTANT]
> Los [límites de suscripción](container-instances-quotas.md) (cuotas) predeterminados para los recursos de GPU difieren en la SKU. Los límites de CPU predeterminados para las SKU P100 y V100 se establecen inicialmente en 0. Para solicitar un aumento en una región disponible, envíe una [solicitud de soporte técnico de Azure][azure-support].

### <a name="things-to-know"></a>Cosas que debe saber

* **Tiempo de implementación**: la creación de un grupo de contenedores que contienen recursos de GPU tarda hasta **8-10 minutos**. Esto se debe al tiempo adicional para aprovisionar y configurar una VM de GPU en Azure. 

* **Precios**: igual que en los grupos de contenedores sin recursos de GPU, Azure factura por los recursos consumidos a lo largo de la *duración* de un grupo de contenedores con recursos GPU. La duración se calcula desde el momento en que se extrae la imagen del primer contenedor hasta que el grupo de contenedores finaliza. No incluye el tiempo de implementación del grupo de contenedores.

  Consulte los [detalles de los precios](https://azure.microsoft.com/pricing/details/container-instances/).

* **Controladores de CUDA**: las instancias de contenedor con los recursos de GPU se aprovisionan previamente con controladores de NVIDIA CUDA y tiempos de ejecución de contenedor, por lo que puede usar imágenes de contenedor desarrolladas para cargas de trabajo de CUDA.

  Solo se admite CUDA 9.0 en esta fase. Por ejemplo, puede usar las imágenes base siguientes para su archivo de Docker:
  * [nvidia/cuda:9.0-base-ubuntu16.04](https://hub.docker.com/r/nvidia/cuda/)
  * [tensorflow/tensorflow: 1.12.0-gpu-py3](https://hub.docker.com/r/tensorflow/tensorflow)
    
## <a name="yaml-example"></a>Ejemplo de YAML

Una forma de agregar los recursos de GPU es implementar un grupo de contenedores mediante una [archivo YAML](container-instances-multi-container-yaml.md). Copie el siguiente archivo YAML en un nuevo archivo denominado *gpu-deploy-aci.yaml* y, después, guárdelo. Este archivo YAML crea un grupo de contenedores denominado *gpucontainergroup* en el que se especifica una instancia de contenedor con una GPU K80. La instancia ejecuta una aplicación de adición de vector CUDA de ejemplo. Las solicitudes de recursos son suficientes para ejecutar la carga de trabajo.

```YAML
additional_properties: {}
apiVersion: '2019-12-01'
name: gpucontainergroup
properties:
  containers:
  - name: gpucontainer
    properties:
      image: k8s-gcrio.azureedge.net/cuda-vector-add:v0.1
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
          gpu:
            count: 1
            sku: K80
  osType: Linux
  restartPolicy: OnFailure
```

Implemente el grupo de contenedores con el comando [az container create][az-container-create] y especifique el nombre del archivo YAML para el parámetro `--file`. Deberá proporcionar el nombre de un grupo de recursos y una ubicación para el grupo de contenedores como *eastus* que admita los recursos de GPU.  

```azurecli
az container create --resource-group myResourceGroup --file gpu-deploy-aci.yaml --location eastus
```

La implementación tarda varios minutos en completarse. A continuación, el contenedor se inicia y ejecuta una operación de adición de vector CUDA. Ejecute el comando [az container logs][az-container-logs] para ver la salida del registro:

```azurecli
az container logs --resource-group myResourceGroup --name gpucontainergroup --container-name gpucontainer
```

Salida:

```Console
[Vector addition of 50000 elements]
Copy input data from the host memory to the CUDA device
CUDA kernel launch with 196 blocks of 256 threads
Copy output data from the CUDA device to the host memory
Test PASSED
Done
```

## <a name="resource-manager-template-example"></a>Ejemplo de plantilla de Resource Manager

Otra manera de implementar un grupo de contenedores con recursos de GPU es usar una [plantilla de Resource Manager](container-instances-multi-container-group.md). Para empezar, cree un archivo llamado `gpudeploy.json` y copie el siguiente código JSON en él. Este ejemplo implementa una instancia de contenedor con una GPU V100 que ejecuta un trabajo de aprendizaje [TensorFlow](https://www.tensorflow.org/) en relación con el conjunto de datos MNIST. Las solicitudes de recursos son suficientes para ejecutar la carga de trabajo.

```JSON
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "containerGroupName": {
        "type": "string",
        "defaultValue": "gpucontainergrouprm",
        "metadata": {
          "description": "Container Group name."
        }
      }
    },
    "variables": {
      "containername": "gpucontainer",
      "containerimage": "microsoft/samples-tf-mnist-demo:gpu"
    },
    "resources": [
      {
        "name": "[parameters('containerGroupName')]",
        "type": "Microsoft.ContainerInstance/containerGroups",
        "apiVersion": "2019-12-01",
        "location": "[resourceGroup().location]",
        "properties": {
            "containers": [
            {
              "name": "[variables('containername')]",
              "properties": {
                "image": "[variables('containerimage')]",
                "resources": {
                  "requests": {
                    "cpu": 4.0,
                    "memoryInGb": 12.0,
                    "gpu": {
                        "count": 1,
                        "sku": "V100"
                  }
                }
              }
            }
          }
        ],
        "osType": "Linux",
        "restartPolicy": "OnFailure"
        }
      }
    ]
}
```

Implemente la plantilla con el comando [az deployment group create][az-deployment-group-create]. Deberá proporcionar el nombre de un grupo de recursos que se creó en una región como *eastus* que admite recursos de GPU.

```azurecli-interactive
az deployment group create --resource-group myResourceGroup --template-file gpudeploy.json
```

La implementación tarda varios minutos en completarse. A continuación, el contenedor se inicia y ejecuta el trabajo de TensorFlow. Ejecute el comando [az container logs][az-container-logs] para ver la salida del registro:

```azurecli
az container logs --resource-group myResourceGroup --name gpucontainergrouprm --container-name gpucontainer
```

Salida:

```Console
2018-10-25 18:31:10.155010: I tensorflow/core/platform/cpu_feature_guard.cc:137] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
2018-10-25 18:31:10.305937: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1030] Found device 0 with properties:
name: Tesla K80 major: 3 minor: 7 memoryClockRate(GHz): 0.8235
pciBusID: ccb6:00:00.0
totalMemory: 11.92GiB freeMemory: 11.85GiB
2018-10-25 18:31:10.305981: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1120] Creating TensorFlow device (/device:GPU:0) -> (device: 0, name: Tesla K80, pci bus id: ccb6:00:00.0, compute capability: 3.7)
2018-10-25 18:31:14.941723: I tensorflow/stream_executor/dso_loader.cc:139] successfully opened CUDA library libcupti.so.8.0 locally
Successfully downloaded train-images-idx3-ubyte.gz 9912422 bytes.
Extracting /tmp/tensorflow/input_data/train-images-idx3-ubyte.gz
Successfully downloaded train-labels-idx1-ubyte.gz 28881 bytes.
Extracting /tmp/tensorflow/input_data/train-labels-idx1-ubyte.gz
Successfully downloaded t10k-images-idx3-ubyte.gz 1648877 bytes.
Extracting /tmp/tensorflow/input_data/t10k-images-idx3-ubyte.gz
Successfully downloaded t10k-labels-idx1-ubyte.gz 4542 bytes.
Extracting /tmp/tensorflow/input_data/t10k-labels-idx1-ubyte.gz
Accuracy at step 0: 0.097
Accuracy at step 10: 0.6993
Accuracy at step 20: 0.8208
Accuracy at step 30: 0.8594
...
Accuracy at step 990: 0.969
Adding run metadata for 999
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Como el uso de recursos de GPU puede ser caro, asegúrese de que los contenedores no se ejecutan inesperadamente durante largos períodos. Supervise los contenedores en Azure Portal o consulte el estado de un grupo de contenedores con el comando [az container show][az-container-show]. Por ejemplo:

```azurecli
az container show --resource-group myResourceGroup --name gpucontainergroup --output table
```

Cuando termine de trabajar con las instancias de contenedor que creó, elimínelas con los siguientes comandos:

```azurecli
az container delete --resource-group myResourceGroup --name gpucontainergroup -y
az container delete --resource-group myResourceGroup --name gpucontainergrouprm -y
```

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre cómo implementar un grupo de contenedores con un [archivo YAML](container-instances-multi-container-yaml.md) o una [plantilla de Resource Manager](container-instances-multi-container-group.md).
* Obtenga más información sobre los [tamaños de máquinas virtuales optimizadas para GPU](../virtual-machines/sizes-gpu.md) en Azure.


<!-- IMAGES -->
[aci-vnet-01]: ./media/container-instances-vnet/aci-vnet-01.png

<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[azure-support]: https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest

<!-- LINKS - Internal -->
[az-container-create]: /cli/azure/container#az-container-create
[az-container-show]: /cli/azure/container#az-container-show
[az-container-logs]: /cli/azure/container#az-container-logs
[az-container-show]: /cli/azure/container#az-container-show
[az-deployment-group-create]: /cli/azure/deployment/group#az-deployment-group-create
