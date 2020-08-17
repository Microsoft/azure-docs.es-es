---
title: 'Tutorial: Creación y uso de discos para conjuntos de escalado con la CLI de Azure'
description: Aprenda a usar la CLI de Azure para crear y usar discos administrados con conjuntos de escalado de máquinas virtuales, incluido cómo agregar, preparar, enumerar y desconectar discos.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 03/27/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: a4be498b25aee7c5a50b2f35fe06be3763eb4732
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87825861"
---
# <a name="tutorial-create-and-use-disks-with-virtual-machine-scale-set-with-the-azure-cli"></a>Tutorial: Creación y uso de discos con conjuntos de escalado de máquinas virtuales con la CLI de Azure
Los conjuntos de escalado de máquinas virtuales usan discos para almacenar el sistema operativo, las aplicaciones y los datos de las máquinas virtuales. Al crear y administrar un conjunto de escalado, es importante elegir un tamaño de disco y la configuración adecuada para la carga de trabajo esperada. Este tutorial explica cómo crear y administrar discos de máquina virtual. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Discos del SO y temporales.
> * Discos de datos.
> * Discos Estándar y Premium.
> * Rendimiento de disco.
> * Conexión y preparación de los discos de datos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure 2.0.29 o versiones posteriores. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).


## <a name="default-azure-disks"></a>Discos de Azure predeterminados
Cuando se crea o se escala un conjunto de escalado, se conectan automáticamente dos discos a cada instancia de máquina virtual.

**Disco del sistema operativo**: hospeda el sistema operativo de las instancias de máquina virtual. Pueden tener un tamaño de hasta 2 terabytes. El disco del sistema operativo lleva de forma predeterminada la etiqueta */dev/sda* . La configuración de almacenamiento en caché del disco del sistema operativo está optimizada para el rendimiento del sistema operativo. Debido a esta configuración, el disco del sistema operativo **no debe** hospedar aplicaciones o datos. Para aplicaciones y datos, use discos de datos, que se detallan más adelante en este artículo.

**Disco temporal**: los discos temporales usan una unidad de estado sólido que se encuentra en el mismo host de Azure que la instancia de máquina virtual. Son discos de gran rendimiento y se pueden usar para operaciones tales como el procesamiento temporal de los datos. Sin embargo, si la instancia de máquina virtual se mueve a un nuevo host, los datos almacenados en un disco temporal se eliminarán. El tamaño del disco temporal se determina por el tamaño de la instancia de máquina virtual. Los discos temporales llevan la etiqueta */dev/sdb* y tienen un punto de montaje de */mnt*.

### <a name="temporary-disk-sizes"></a>Tamaños de disco temporal
| Tipo | Tamaños comunes | Tamaño máximo de disco temporal (GiB) |
|----|----|----|
| [Uso general](../virtual-machines/sizes-general.md) | Series A, B y D | 1600 |
| [Proceso optimizado](../virtual-machines/sizes-compute.md) | Serie F | 576 |
| [Memoria optimizada](../virtual-machines/sizes-memory.md) | Series D, E, G y M | 6144 |
| [Almacenamiento optimizado](../virtual-machines/sizes-storage.md) | Serie L | 5630 |
| [GPU](../virtual-machines/sizes-gpu.md) | Serie N | 1440 |
| [Alto rendimiento](../virtual-machines/sizes-hpc.md) | Series A y H | 2000 |


## <a name="azure-data-disks"></a>Discos de datos de Azure
Se pueden agregar discos de datos adicionales si necesita instalar aplicaciones y almacenar datos. Los discos de datos deben usarse en cualquier situación donde desee un almacenamiento de datos duradero y con capacidad de respuesta. Cada disco de datos tiene una capacidad máxima de 4 TB. El tamaño de la instancia de máquina virtual determina cuántos discos de datos se pueden conectar. Para cada vCPU de máquina virtual, se pueden conectar dos discos de datos hasta un máximo de 64 discos por máquina virtual.

## <a name="vm-disk-types"></a>Tipos de disco de máquina virtual
Azure proporciona dos tipos de disco.

### <a name="standard-disk"></a>Disco estándar
Standard Storage está respaldado por unidades de disco duro y ofrece un almacenamiento rentable con un buen rendimiento. Los discos estándar son ideales para cargas de trabajo de desarrollo y prueba rentables.

### <a name="premium-disk"></a>Disco Premium
Los discos Premium están respaldados por un disco de latencia reducida y alto rendimiento basado en SSD. Estos discos se recomiendan para las máquinas virtuales que ejecutan cargas de trabajo de producción. Premium Storage es compatible con las máquinas virtuales de las series DS, DSv2, GS y FS. Al seleccionar el tamaño de un disco, el valor se redondea al alza al siguiente tipo. Por ejemplo, si el tamaño del disco es inferior a 128 GB, el tipo de disco es P10. Si el tamaño de disco está entre 129 y 512 GB, el tamaño es un P20. Cualquier tamaño por encima de 512 GB equivale a un tipo P30.

### <a name="premium-disk-performance"></a>Rendimiento del disco Premium
|Tipo de disco de Premium Storage | P4 | P6 | P10 | P20 | P30 | P40 | P50 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Tamaño del disco (redondeo hacia arriba) | 32 GB | 64 GB | 128 GB | 512 GB | 1024 GB (1 TB) | 2048 GB (2 TB) | 4095 GB (4 TB) |
| Máximo de IOPS por disco | 120 | 240 | 500 | 2,300 | 5\.000 | 7500 | 7500 |
Rendimiento de disco. | 25 MB/s | 50 MB/s | 100 MB/s | 150 MB/s | 200 MB/s | 250 MB/s | 250 MB/s |

Aunque la tabla anterior identifica las IOPS máximas por disco, se puede obtener un mayor nivel de rendimiento dividiendo varios discos de datos. Por ejemplo, una máquina virtual Standard_GS5 puede conseguir 80 000 IOPS como máximo. Para más información sobre el número máximo de IOPS por máquina virtual, consulte los [tamaños de máquinas virtuales Linux](../virtual-machines/sizes.md).


## <a name="create-and-attach-disks"></a>Creación y conexión de discos
Puede crear y conectar discos durante la creación de un conjunto de escalado o a un conjunto de escalado existente.

### <a name="attach-disks-at-scale-set-creation"></a>Conexión de discos durante la creación del conjunto de escalado
En primer lugar, cree un grupo de recursos con el comando [az group create](/cli/azure/group). En este ejemplo se crea un grupo de recursos denominado *myResourceGroup* en la región *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Cree un conjunto de escalado de máquinas virtuales con el comando [az vmss create](/cli/azure/vmss). En el ejemplo siguiente se crea un conjunto de escalado llamado *myScaleSet* y se generan claves SSH si estas no existen aún. Se crean dos discos con el parámetro `--data-disk-sizes-gb`. Es el primer disco tiene un tamaño de *64* GB y, el segundo disco, de *128* GB:

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy automatic \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 64 128
```

Se tardan unos minutos en crear y configurar todos los recursos del conjunto de escalado y las instancias de máquina virtual.

### <a name="attach-a-disk-to-existing-scale-set"></a>Conexión de un disco a un conjunto de escalado existente
También puede conectar discos a un conjunto de escalado existente. Use el conjunto de escalado creado en el paso anterior para agregar otro disco con [az vmss disk attach](/cli/azure/vmss/disk). En el ejemplo siguiente, se conecta un disco adicional de *128* GB:

```azurecli-interactive
az vmss disk attach \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --size-gb 128
```


## <a name="prepare-the-data-disks"></a>Preparación de los discos de datos
Los discos que se crean y conectan a las instancias de máquina virtual del conjunto de escalado son discos sin formato. Para poder usarlos con sus aplicaciones y datos, los discos deben prepararse. Para preparar los discos, cree una partición, cree un sistema de archivos y móntelos.

Para automatizar el proceso en varias instancias de máquina virtual en un conjunto de escalado, puede usar la extensión de script personalizado de Azure. Esta extensión puede ejecutar scripts localmente en cada instancia de máquina virtual, por ejemplo, para preparar los discos de datos conectados. Para obtener más información, consulte [Información general de la extensión de script personalizado](../virtual-machines/extensions/custom-script-linux.md).

En el ejemplo siguiente, se ejecuta un script desde un repositorio de ejemplo de GitHub en cada instancia de máquina virtual con [az vmss extension set](/cli/azure/vmss/extension), que prepara todos los discos de datos conectados sin formato:

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/prepare_vm_disks.sh"],"commandToExecute":"./prepare_vm_disks.sh"}'
```

Para confirmar que los discos se han preparado correctamente, conéctese mediante SSH a una de las instancias de máquina virtual. Para mostrar la información de conexión del conjunto de escalado, use [az vmss list-instance-connection-info](/cli/azure/vmss):

```azurecli-interactive
az vmss list-instance-connection-info \
  --resource-group myResourceGroup \
  --name myScaleSet
```

Use su propia dirección IP pública y número de puerto para conectarse a la primera instancia de máquina virtual, tal y como se muestra en el ejemplo siguiente:

```console
ssh azureuser@52.226.67.166 -p 50001
```

Examine las particiones en la instancia de máquina virtual, tal y como se indica a continuación:

```bash
sudo fdisk -l
```

La salida del ejemplo siguiente muestra que los tres discos están conectados con la instancia de máquina virtual: */dev/sdc*, */dev/sdd*, and */dev/sde*. Cada uno de esos discos tiene una única partición que usa todo el espacio disponible:

```bash
Disk /dev/sdc: 64 GiB, 68719476736 bytes, 134217728 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa47874cb

Device     Boot Start       End   Sectors Size Id Type
/dev/sdc1        2048 134217727 134215680  64G 83 Linux

Disk /dev/sdd: 128 GiB, 137438953472 bytes, 268435456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd5c95ca0

Device     Boot Start       End   Sectors  Size Id Type
/dev/sdd1        2048 268435455 268433408  128G 83 Linux

Disk /dev/sde: 128 GiB, 137438953472 bytes, 268435456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x9312fdf5

Device     Boot Start       End   Sectors  Size Id Type
/dev/sde1        2048 268435455 268433408  128G 83 Linux
```

Examine los sistemas de archivos y los puntos de montaje en la instancia de máquina virtual, de la siguiente manera:

```bash
sudo df -h
```

La siguiente salida de ejemplo muestra que los tres discos tienen sus sistemas de archivos montados correctamente en */datadisks*:

```output
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        30G  1.3G   28G   5% /
/dev/sdb1        50G   52M   47G   1% /mnt
/dev/sdc1        63G   52M   60G   1% /datadisks/disk1
/dev/sdd1       126G   60M  120G   1% /datadisks/disk2
/dev/sde1       126G   60M  120G   1% /datadisks/disk3
```

Los discos en cada instancia de máquina virtual del conjunto de escalado se preparan automáticamente de la misma manera. A medida que conjunto de escalado escale verticalmente, se conectan los discos de datos necesarios a las nuevas instancias de máquina virtual. La extensión de script personalizado también se ejecuta para preparar automáticamente los discos.

Cierre la conexión SSH a la instancia de máquina virtual:

```bash
exit
```


## <a name="list-attached-disks"></a>Enumeración de los discos conectados
Para ver información acerca de los discos conectados a un conjunto de escalado, use [az vmss show](/cli/azure/vmss) y consulte *virtualMachineProfile.storageProfile.dataDisks*:

```azurecli-interactive
az vmss show \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --query virtualMachineProfile.storageProfile.dataDisks
```

Se muestra información sobre el tamaño del disco, la capa de almacenamiento y el LUN (número de unidad lógica). La salida del ejemplo siguiente muestra los tres discos de datos conectados con el conjunto de escalado:

```json
[
  {
    "additionalProperties": {},
    "caching": "None",
    "createOption": "Empty",
    "diskSizeGb": 64,
    "lun": 0,
    "managedDisk": {
      "additionalProperties": {},
      "storageAccountType": "Standard_LRS"
    },
    "name": null
  },
  {
    "additionalProperties": {},
    "caching": "None",
    "createOption": "Empty",
    "diskSizeGb": 128,
    "lun": 1,
    "managedDisk": {
      "additionalProperties": {},
      "storageAccountType": "Standard_LRS"
    },
    "name": null
  },
  {
    "additionalProperties": {},
    "caching": "None",
    "createOption": "Empty",
    "diskSizeGb": 128,
    "lun": 2,
    "managedDisk": {
      "additionalProperties": {},
      "storageAccountType": "Standard_LRS"
    },
    "name": null
  }
]
```


## <a name="detach-a-disk"></a>Desconexión de un disco
Cuando ya no se necesita un disco determinado, se puede desconectar del conjunto de escalado. El disco se quita de todas las instancias de máquina virtual del conjunto de escalado. Para desconectar un disco de un conjunto de escalado, use [az vmss disk detach](/cli/azure/vmss/disk) y especifique el LUN del disco. Los LUN se muestran en la salida de [az vmss show](/cli/azure/vmss), en la sección anterior. En el ejemplo siguiente, se desconecta el LUN *2* del conjunto de escalado:

```azurecli-interactive
az vmss disk detach \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --lun 2
```


## <a name="clean-up-resources"></a>Limpieza de recursos
Para quitar el conjunto de escalado y los discos, elimine el grupo de recursos y todos sus recursos con [az group delete](/cli/azure/group). El parámetro `--no-wait` devuelve el control a la petición de confirmación sin esperar a que finalice la operación. El parámetro `--yes` confirma que desea eliminar los recursos sin pedir confirmación adicional.

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>Pasos siguientes
En este tutorial, aprendió a crear y utilizar discos con conjuntos de escalado con la CLI de Azure:

> [!div class="checklist"]
> * Discos del SO y temporales.
> * Discos de datos.
> * Discos Estándar y Premium.
> * Rendimiento de disco.
> * Conexión y preparación de los discos de datos

Vaya al siguiente tutorial para aprender a usar una imagen personalizada para las instancias de máquina virtual de su conjunto de escalado.

> [!div class="nextstepaction"]
> [Uso de una imagen personalizada para las instancias de máquina virtual de un conjunto de escalado](tutorial-use-custom-image-cli.md)