---
title: 'Tutorial: Creación y administración de un conjunto de escalado de máquinas virtuales de Azure: CLI de Azure'
description: Obtenga información sobre cómo usar la CLI de Azure para crear un conjunto de escalado de máquinas virtuales, junto con algunas tareas de administración comunes, como cómo iniciar y detener una instancia, o cambiar la capacidad del conjunto de escalado.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: management
ms.date: 03/27/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: e7267ca90ea11e63c5523dec0a3ee414f7b655b2
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501651"
---
# <a name="tutorial-create-and-manage-a-virtual-machine-scale-set-with-the-azure-cli"></a>Tutorial: Creación y administración de un conjunto de escalado de máquinas virtuales con la CLI de Azure
El conjunto de escalado de máquinas virtuales le permite implementar y administrar un conjunto de máquinas virtuales de escalado automático idénticas. Durante el ciclo de vida de la máquina virtual, es posible que deba ejecutar una o varias tareas de administración. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un conjunto de escalado de máquinas virtuales y conectarse a él
> * Seleccionar y usar imágenes de máquinas virtuales
> * Ver y usar tamaños específicos de instancia de máquina virtual
> * Escalar manualmente un conjunto de escalado
> * Realizar tareas comunes de administración del conjunto de escalado

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure 2.0.29 o versiones posteriores. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli). 


## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Se debe crear un grupo de recursos antes de un conjunto de escalado de máquinas virtuales. Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group). En este ejemplo se crea un grupo de recursos denominado *myResourceGroup* en la región *eastus*. 

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

El nombre del grupo de recursos se especifica al crear o modificar un conjunto de escalado mediante este tutorial.


## <a name="create-a-scale-set"></a>Creación de un conjunto de escalado
Creará un conjunto de escalado de máquinas virtuales con el comando [az vmss create](/cli/azure/vmss). En el ejemplo siguiente se crea un conjunto de escalado denominado *myScaleSet* y se generan claves SSH si estas no existen aún:

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

Se tardan unos minutos en crear y configurar todos los recursos del conjunto de escalado y las instancias de máquina virtual. Para distribuir el tráfico a las instancias individuales de VM, también se crea un equilibrador de carga.


## <a name="view-the-vm-instances-in-a-scale-set"></a>Visualización de las instancias de máquina virtual de un conjunto de escalado
Para ver una lista de instancias de máquina virtual de un conjunto de escalado, use [az vmss list-instances](/cli/azure/vmss) de la manera siguiente:

```azurecli-interactive
az vmss list-instances \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --output table
```

La salida del ejemplo siguiente muestra dos instancias de máquina virtual del conjunto de escalado:

```output
  InstanceId  LatestModelApplied    Location    Name          ProvisioningState    ResourceGroup    VmId
------------  --------------------  ----------  ------------  -------------------  ---------------  ------------------------------------
           1  True                  eastus      myScaleSet_1  Succeeded            MYRESOURCEGROUP  c059be0c-37a2-497a-b111-41272641533c
           3  True                  eastus      myScaleSet_3  Succeeded            MYRESOURCEGROUP  ec19e7a7-a4cd-4b24-9670-438f4876c1f9
```


La primera columna de la salida muestra *InstanceId*. Para ver información adicional sobre una instancia específica de la máquina virtual, agregue el parámetro `--instance-id` a [az vmss get-instance-view](/cli/azure/vmss). En el ejemplo siguiente, se ve información sobre la instancia de máquina virtual *1*:

```azurecli-interactive
az vmss get-instance-view \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --instance-id 1
```


## <a name="list-connection-information"></a>Visualización de información de conexión
Se asigna una dirección IP pública al equilibrador de carga que enruta el tráfico a las instancias de máquina virtual individuales. De manera predeterminada, se agregan reglas de traducción de direcciones de red (NAT) a Azure Load Balancer, que reenvía el tráfico de conexión remota a cada máquina virtual en un puerto dado. Para conectarse a las instancias de máquina virtual de un conjunto de escalado, creará una conexión remota a una dirección IP pública y un número de puerto asignados.

Para mostrar las direcciones y los puertos para conectarse a las instancias de máquina virtual de un conjunto de escalado, use [az vmss list-instance-connection-info](/cli/azure/vmss).

```azurecli-interactive
az vmss list-instance-connection-info \
  --resource-group myResourceGroup \
  --name myScaleSet
```

La salida del ejemplo siguiente muestra el nombre de instancia, la dirección IP pública del equilibrador de carga y el número de puerto al que las reglas NAT reenvían el tráfico:

```output
{
  "instance 1": "13.92.224.66:50001",
  "instance 3": "13.92.224.66:50003"
}
```

Conéctese mediante SSH a la primera instancia de máquina virtual. Especifique la dirección IP pública y el número de puerto con el parámetro `-p`, tal como se muestra en el comando anterior:

```console
ssh azureuser@13.92.224.66 -p 50001
```

Una vez que inicia sesión en la instancia de máquina virtual, puede realizar algunos cambios de configuración manual según sea necesario. Por ahora, cierre la sesión SSH de la manera habitual:

```console
exit
```


## <a name="understand-vm-instance-images"></a>Imágenes de instancia de máquina virtual
Cuando creó un conjunto de escalado al inicio de este tutorial, se especificó un parámetro `--image` de *UbuntuLTS* para las instancias de máquina virtual. Azure Marketplace incluye muchas imágenes que pueden usarse para crear instancias de máquina virtual. Para ver una lista de las imágenes usadas con más frecuencia, use el comando [az vm image list](/cli/azure/vm/image).

```azurecli-interactive
az vm image list --output table
```

En la siguiente salida de ejemplo se muestran las imágenes de máquina virtual más comunes en Azure. *UrnAlias* se puede usar para especificar una de estas imágenes comunes cuando se crea un conjunto de escalado.

```output
Offer          Publisher               Sku                 Urn                                                             UrnAlias             Version
-------------  ----------------------  ------------------  --------------------------------------------------------------  -------------------  ---------
CentOS         OpenLogic               7.3                 OpenLogic:CentOS:7.3:latest                                     CentOS               latest
CoreOS         CoreOS                  Stable              CoreOS:CoreOS:Stable:latest                                     CoreOS               latest
Debian         credativ                8                   credativ:Debian:8:latest                                        Debian               latest
openSUSE-Leap  SUSE                    42.2                SUSE:openSUSE-Leap:42.2:latest                                  openSUSE-Leap        latest
RHEL           RedHat                  7.3                 RedHat:RHEL:7.3:latest                                          RHEL                 latest
SLES           SUSE                    12-SP2              SUSE:SLES:12-SP2:latest                                         SLES                 latest
UbuntuServer   Canonical               16.04-LTS           Canonical:UbuntuServer:16.04-LTS:latest                         UbuntuLTS            latest
WindowsServer  MicrosoftWindowsServer  2016-Datacenter     MicrosoftWindowsServer:WindowsServer:2016-Datacenter:latest     Win2016Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2012-R2-Datacenter  MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest  Win2012R2Datacenter  latest
WindowsServer  MicrosoftWindowsServer  2012-Datacenter     MicrosoftWindowsServer:WindowsServer:2012-Datacenter:latest     Win2012Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2008-R2-SP1         MicrosoftWindowsServer:WindowsServer:2008-R2-SP1:latest         Win2008R2SP1         latest
```

Para ver una lista completa, agregue el argumento `--all`. También puede filtrar la lista de imágenes por `--publisher` u `–-offer`. En el ejemplo siguiente, la lista se ha filtrado por todas las imágenes con una oferta que coincide con *CentOS*:

```azurecli-interactive
az vm image list --offer CentOS --all --output table
```

La siguiente salida reducida muestra algunas de las imágenes de CentOS 7.3 disponibles:

```output
Offer    Publisher   Sku   Urn                                 Version
-------  ----------  ----  ----------------------------------  -------------
CentOS   OpenLogic   7.3   OpenLogic:CentOS:7.3:7.3.20161221   7.3.20161221
CentOS   OpenLogic   7.3   OpenLogic:CentOS:7.3:7.3.20170421   7.3.20170421
CentOS   OpenLogic   7.3   OpenLogic:CentOS:7.3:7.3.20170517   7.3.20170517
CentOS   OpenLogic   7.3   OpenLogic:CentOS:7.3:7.3.20170612   7.3.20170612
CentOS   OpenLogic   7.3   OpenLogic:CentOS:7.3:7.3.20170707   7.3.20170707
CentOS   OpenLogic   7.3   OpenLogic:CentOS:7.3:7.3.20170925   7.3.20170925
```

Para implementar un conjunto de escalado que usa una imagen específica, utilice el valor de la columna *Urn*. Al especificar la imagen, se puede reemplazar el número de versión de la imagen por *latest*, que selecciona la versión más reciente de la distribución. En el ejemplo siguiente, se emplea el argumento `--image` para especificar la versión más reciente de una imagen de CentOS 7.3.

> [!IMPORTANT]
> Se recomienda usar la versión *más reciente* de la imagen. Especifique "más reciente" para usar la versión más reciente de una imagen disponible en el momento de la implementación. Tenga en cuenta que, aunque use "más reciente", la imagen de la máquina virtual no se actualizará automáticamente después de la implementación, incluso si hay disponible una nueva versión.

Como se tarda unos minutos en crear y configurar todos los recursos del conjunto de escalado y las instancias de máquina virtual, no tiene que implementar el siguiente conjunto de escalado:

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSetCentOS \
  --image OpenLogic:CentOS:7.3:latest \
  --admin-user azureuser \
  --generate-ssh-keys
```


## <a name="understand-vm-instance-sizes"></a>Tamaños de instancia de máquina virtual
El tamaño de instancia de la máquina virtual, o *SKU*, determina la cantidad de recursos de proceso, como memoria, CPU y GPU, que están disponibles para la instancia de máquina virtual. Las instancias de máquina virtual de un conjunto de escalado deben tener el tamaño adecuado para la carga de trabajo esperada.

### <a name="vm-instance-sizes"></a>Tamaños de instancia de máquina virtual
En la tabla siguiente se clasifican los tamaños de máquina virtual comunes en casos de uso.

| Tipo                     | Tamaños comunes           |    Descripción       |
|--------------------------|-------------------|------------------------------------------------------------------------------------------------------------------------------------|
| [Uso general](../virtual-machines/sizes-general.md)         |Dsv3, Dv3, DSv2, Dv2, DS, D, Av2, A0-7| Uso equilibrado de CPU y memoria. Ideal para desarrollo/pruebas, así como soluciones de datos y aplicaciones de tamaño pequeño a mediano.  |
| [Proceso optimizado](../virtual-machines/sizes-compute.md)   | Fs, F             | Uso elevado de la CPU respecto a la memoria. Adecuado para aplicaciones, dispositivos de red y procesos por lotes con tráfico mediano.        |
| [Memoria optimizada](../virtual-machines/sizes-memory.md)    | Esv3, Ev3, M, GS, G, DSv2, DS, Dv2, D   | Uso elevado de memoria respecto al núcleo. Excelente para bases de datos relacionales, memorias caché de capacidad de mediana a grande y análisis en memoria.                 |
| [Almacenamiento optimizado](../virtual-machines/sizes-storage.md)      | LS                | Alto rendimiento de disco y E/S. Perfecto para bases de datos SQL y NoSQL y macrodatos.                                                         |
| [GPU](../virtual-machines/sizes-gpu.md)          | NV, NC            | Máquinas virtuales especializadas para actividades intensas de representación de gráficos y edición de vídeo.       |
| [Alto rendimiento](../virtual-machines/sizes-hpc.md) | H, A8-11          | Nuestras máquinas virtuales con CPU más eficaces e interfaces de red de alto rendimiento (RDMA) opcionales. 

### <a name="find-available-vm-instance-sizes"></a>Búsqueda de tamaños de instancia de máquina virtual disponibles
Para ver una lista de tamaños de instancia de máquina virtual disponibles en una región determinada, use el comando [az vm list-sizes](/cli/azure/vm).

```azurecli-interactive
az vm list-sizes --location eastus --output table
```

La salida es similar al siguiente ejemplo reducido, que muestra los recursos asignados a cada tamaño de máquina virtual:

```output
  MaxDataDiskCount    MemoryInMb  Name                      NumberOfCores    OsDiskSizeInMb    ResourceDiskSizeInMb
------------------  ------------  ----------------------  ---------------  ----------------  ----------------------
                 4          3584  Standard_DS1_v2                       1           1047552                    7168
                 8          7168  Standard_DS2_v2                       2           1047552                   14336
[...]
                 1           768  Standard_A0                           1           1047552                   20480
                 2          1792  Standard_A1                           1           1047552                   71680
[...]
                 4          2048  Standard_F1                           1           1047552                   16384
                 8          4096  Standard_F2                           2           1047552                   32768
[...]
                24         57344  Standard_NV6                          6           1047552                   38912
                48        114688  Standard_NV12                        12           1047552                  696320
```

### <a name="create-a-scale-set-with-a-specific-vm-instance-size"></a>Creación de un conjunto de escalado con un tamaño específico de instancia de máquina virtual
Cuando creó un conjunto de escalado al comienzo del tutorial, se proporcionó una SKU de máquina virtual predeterminada de *Standard_D1_v2* para las instancias de máquina virtual. Puede especificar un tamaño de instancia de máquina virtual diferente en función de la salida de [az vm list-sizes](/cli/azure/vm). En el ejemplo siguiente se crearía un conjunto de escalado con el parámetro `--vm-sku` para especificar un tamaño de instancia de máquina virtual de *Standard_F1*. Como se tarda unos minutos en crear y configurar todos los recursos del conjunto de escalado y las instancias de máquina virtual, no tiene que implementar el siguiente conjunto de escalado:

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSetF1Sku \
  --image UbuntuLTS \
  --vm-sku Standard_F1 \
  --admin-user azureuser \
  --generate-ssh-keys
```


## <a name="change-the-capacity-of-a-scale-set"></a>Cambio de la capacidad de un conjunto de escalado
Cuando creó un conjunto de escalado al comienzo del tutorial, se implementaron de forma predeterminada dos instancias de máquina virtual. Puede especificar el parámetro `--instance-count` con [az vmss create](/cli/azure/vmss) para cambiar el número de instancias creadas con un conjunto de escalado. Para aumentar o disminuir el número de instancias de máquina virtual del conjunto de escalado existente, puede cambiar manualmente la capacidad. El conjunto de escalado crea o quita el número necesario de instancias de máquina virtual y, luego, configura el equilibrador de carga para distribuir el tráfico.

Para aumentar o reducir manualmente el número de instancias de máquina virtual del conjunto de escalado, use [az vmss scale](/cli/azure/vmss). En el ejemplo siguiente se establece el número de instancias de máquina virtual del conjunto de escalado en *3*:

```azurecli-interactive
az vmss scale \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --new-capacity 3
```

La capacidad del conjunto de escalado tarda unos minutos en actualizarse. Para ver el número de instancias que tiene ahora en un conjunto de escalado, use [az vmss show](/cli/azure/vmss) y realice consultas a *sku.capacity*:

```azurecli-interactive
az vmss show \
    --resource-group myResourceGroup \
    --name myScaleSet \
    --query [sku.capacity] \
    --output table
```


## <a name="common-management-tasks"></a>Tareas comunes de administración
Ahora puede crear un conjunto de escalado, mostrar información de conexión y conectarse a instancias de máquina virtual. Ha aprendido cómo podría usar otra imagen de sistema operativo para sus instancias de máquina virtual, cómo seleccionar otro tamaño de máquina virtual o cómo escalar manualmente el número de instancias. Como parte de la administración diaria, puede que deba detener, iniciar o reiniciar las instancias de máquina virtual del conjunto de escalado.

### <a name="stop-and-deallocate-vm-instances-in-a-scale-set"></a>Detención y desasignación de instancias de máquina virtual de un conjunto de escalado
Para detener una o más instancias de máquina virtual de un conjunto de escalado, use [az vmss stop](/cli/azure/vmss). El parámetro `--instance-ids` le permite especificar que se detengan una o varias máquinas virtuales. Si no especifica un identificador de instancia, se detienen todas las instancias de máquina virtual del conjunto de escalado. En el ejemplo siguiente se detiene la instancia *1*:

```azurecli-interactive
az vmss stop --resource-group myResourceGroup --name myScaleSet --instance-ids 1
```

Las instancias de máquina virtual detenidas permanecen asignadas y siguen acumulando cargos por procesos. Si lo que desea es desasignar las instancias de máquina virtual y solo acumular cargos por procesos, use [az vmss deallocate](/cli/azure/vmss). En el ejemplo siguiente se detiene y desasigna la instancia *1*:

```azurecli-interactive
az vmss deallocate --resource-group myResourceGroup --name myScaleSet --instance-ids 1
```

### <a name="start-vm-instances-in-a-scale-set"></a>Inicio de las instancias de máquina virtual de un conjunto de escalado
Para iniciar una o varias instancias de máquina virtual de un conjunto de escalado, use [az vmss start](/cli/azure/vmss). El parámetro `--instance-ids` le permite especificar que se inicien una o varias instancias de máquina virtual. Si no especifica un identificador de instancia, se inician todas las máquinas virtuales del conjunto de escalado. En el ejemplo siguiente se inicia la instancia *1*:

```azurecli-interactive
az vmss start --resource-group myResourceGroup --name myScaleSet --instance-ids 1
```

### <a name="restart-vm-instances-in-a-scale-set"></a>Reinicio de las instancias de máquina virtual de un conjunto de escalado
Para reiniciar una o varias instancias de máquina virtual de un conjunto de escalado, use [az vmss restart](/cli/azure/vmss). El parámetro `--instance-ids` le permite especificar que se reinicien una o varias instancias de máquina virtual. Si no especifica un identificador de instancia, se reinician todas las instancias de máquina virtual del conjunto de escalado. En el ejemplo siguiente, se reinicia la instancia *1*:

```azurecli-interactive
az vmss restart --resource-group myResourceGroup --name myScaleSet --instance-ids 1
```


## <a name="clean-up-resources"></a>Limpieza de recursos
Cuando se elimina un grupo de recursos, todos los recursos que contiene, como las instancias de máquina virtual, la red virtual y los discos, también se eliminan. El parámetro `--no-wait` devuelve el control a la petición de confirmación sin esperar a que finalice la operación. El parámetro `--yes` confirma que desea eliminar los recursos sin pedir confirmación adicional.

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, ha aprendido cómo realizar algunas tareas de creación y administración de un conjunto de escalado básico con la CLI de Azure:

> [!div class="checklist"]
> * Crear un conjunto de escalado de máquinas virtuales y conectarse a él
> * Seleccionar y usar imágenes de máquinas virtuales
> * Ver y usar tamaños de una máquina virtual específicos
> * Escalar manualmente un conjunto de escalado
> * Realizar tareas comunes de administración del conjunto de escalado

Vaya al siguiente tutorial para aprender sobre los discos de conjuntos de escalado.

> [!div class="nextstepaction"]
> [Uso de discos de datos con conjuntos de escalado](tutorial-use-disks-cli.md)
