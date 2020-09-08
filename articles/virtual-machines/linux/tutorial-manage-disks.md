---
title: 'Tutorial: Administración de discos de Azure con la CLI de Azure'
description: En este tutorial, aprenderá a usar la CLI de Azure para crear y administrar discos de Azure para máquinas virtuales
author: cynthn
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 08/20/2020
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.subservice: disks
ms.openlocfilehash: 948a4ae8c329d69e404ef8d0f609748b955b0ecc
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078856"
---
# <a name="tutorial---manage-azure-disks-with-the-azure-cli"></a>Tutorial: Administración de discos de Azure con la CLI de Azure

Las máquinas virtuales (VM) de Azure usan discos para almacenar el sistema operativo, las aplicaciones y los datos. Cuando se crea una máquina virtual es importante elegir un tamaño de disco y la configuración adecuada para la carga de trabajo esperada. En este tutorial se muestra cómo implementar y administrar los discos de una máquina virtual. Aprenderá sobre los siguientes temas:

> [!div class="checklist"]
> * Discos del SO y temporales.
> * Discos de datos.
> * Discos Estándar y Premium.
> * Rendimiento de disco.
> * Conectar y preparar los discos de datos
> * Instantáneas de disco.


## <a name="default-azure-disks"></a>Discos de Azure predeterminados

Cuando se crea una máquina virtual de Azure, se conectan dos discos automáticamente a la máquina virtual.

**Disco del sistema operativo**: el tamaño de los discos del sistema operativo puede llegar a los 2 TB y hospedan el sistema operativo de las máquinas virtuales. El disco del sistema operativo lleva de forma predeterminada la etiqueta */dev/sda* . La configuración de almacenamiento en caché del disco del sistema operativo está optimizada para el rendimiento del sistema operativo. Debido a esta configuración, el disco del sistema operativo **no se debe** usar para aplicaciones o datos. Para aplicaciones y datos, use discos de datos, que se explican más adelante en este tutorial.

**Disco temporal**: los discos temporales usan una unidad de estado sólido que se encuentra en el mismo host de Azure que la máquina virtual. Los discos temporales son muy eficiente y se pueden usar para operaciones tales como el procesamiento temporal de los datos. Sin embargo, si la máquina virtual se mueve a un nuevo host, los datos almacenados en un disco temporal se eliminarán. El tamaño del disco temporal se determina por el tamaño de la máquina virtual. Los discos temporales llevan la etiqueta */dev/sdb* y tienen un punto de montaje de */mnt*.

## <a name="azure-data-disks"></a>Discos de datos de Azure

Para instalar aplicaciones y almacenar datos, se pueden agregar más discos de datos. Los discos de datos deben usarse en cualquier situación donde desee un almacenamiento de datos duradero y con capacidad de respuesta. El tamaño de la máquina virtual determina cuántos discos de datos se pueden conectar a una máquina virtual.

## <a name="vm-disk-types"></a>Tipos de disco de máquina virtual

Azure proporciona dos tipos de discos.

**Discos estándar**: respaldados por unidades de disco duro, ofrecen un almacenamiento rentable y buen rendimiento. Los discos estándar son ideales para cargas de trabajo de desarrollo y prueba rentables.

**Discos Premium**: respaldados por un disco de latencia reducida y alto rendimiento basado en SSD. Es perfecto para máquinas virtuales que ejecutan cargas de trabajo de producción. Los tamaños de máquina virtual con una letra **S** en el [nombre de tamaño](../vm-naming-conventions.md), normalmente admiten Premium Storage. Por ejemplo, las máquinas virtuales de las series DS, DSv2, GS y Fs admiten Premium Storage. Al seleccionar el tamaño de un disco, el valor se redondea al alza al siguiente tipo. Por ejemplo, si el tamaño del disco es superior a 64 GB, pero inferior a 128 GB, el tipo de disco es P10. 

<br>


[!INCLUDE [disk-storage-premium-ssd-sizes](../../../includes/disk-storage-premium-ssd-sizes.md)]

Cuando se aprovisiona un disco de Premium Storage, a diferencia de Standard Storage, se garantizan la capacidad, las E/S por segundo y el rendimiento del mismo. Por ejemplo, si crea un disco P50, Azure aprovisiona una capacidad de almacenamiento de 4095 GB, 7500 E/S por segundo y un rendimiento de 250 MB/s para él. La aplicación puede usar toda la capacidad y el rendimiento o parte de ellos. Los discos SSD Premium están diseñados para proporcionar bajas latencias inferiores a 10 milisegundos y un IOPS y rendimiento que se describen en la tabla anterior como del 99,9 % del tiempo.

Aunque la tabla anterior identifica las IOPS máximas por disco, se puede obtener un mayor nivel de rendimiento dividiendo varios discos de datos. Por ejemplo, 64 discos de datos pueden conectarse a la máquina virtual Standard_GS5. Si cada uno de estos discos tiene un tamaño P30, se puede lograr un máximo de 80 000 IOPS. Para más información sobre el número máximo de IOPS por máquina virtual, vea los [tamaños y topos de máquinas virtuales](../sizes.md).

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta.

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/bash) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="create-and-attach-disks"></a>Creación y conexión de discos

Los discos de datos se pueden crear y conectar en el momento de creación de la máquina virtual o a una máquina virtual existente.

### <a name="attach-disk-at-vm-creation"></a>Conexión del disco en el momento de creación de la máquina virtual

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group#az-group-create).

```azurecli-interactive
az group create --name myResourceGroupDisk --location eastus
```

Cree una máquina virtual mediante el comando [az vm create](/cli/azure/vm#az-vm-create). En el ejemplo siguiente, se crea una máquina virtual denominada *myVM*, se agrega una cuenta de usuario denominada *azureuser* y se generan claves SSH, si no existen. El argumento `--datadisk-sizes-gb` se usa para especificar que se debe crear y conectar un disco adicional a la máquina virtual. Para crear y conectar más de un disco, use una lista delimitada por espacios de valores de tamaño de disco. En el ejemplo siguiente, se crea una máquina virtual con dos discos de datos, ambos de 128 GB. Dado que los tamaños de disco son de 128 GB, estos discos se configuran ambos como P10s, que proporcionan el número máximo de 500 IOPS por disco.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroupDisk \
  --name myVM \
  --image UbuntuLTS \
  --size Standard_DS2_v2 \
  --generate-ssh-keys \
  --data-disk-sizes-gb 128 128
```

### <a name="attach-disk-to-existing-vm"></a>Conexión del disco a máquina virtual existente

Para crear y conectar un nuevo disco a una máquina virtual existente, use el comando [az vm disk attach](/cli/azure/vm/disk#az-vm-disk-attach). En el ejemplo siguiente se crea un disco Premium con un tamaño de 128 gigabytes y se conecta a la máquina virtual que creó en el último paso.

```azurecli-interactive
az vm disk attach \
    --resource-group myResourceGroupDisk \
    --vm-name myVM \
    --name myDataDisk \
    --size-gb 128 \
    --sku Premium_LRS \
    --new
```

## <a name="prepare-data-disks"></a>Preparación de los discos de datos

Después de que se ha conectado un disco a la máquina virtual, es necesario configurar el sistema operativo para usar el disco. En el ejemplo siguiente se muestra cómo configurar manualmente un disco. Este proceso también se puede automatizar mediante cloud-init, que se trata en un [tutorial posterior](./tutorial-automate-vm-deployment.md).


Cree una conexión SSH con la máquina virtual. Reemplace la dirección IP de ejemplo por la dirección IP pública de la máquina virtual:

```console
ssh 10.101.10.10
```

Cree particiones del disco con `parted`.

```bash
sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%
```

Escriba un sistema de archivos en la partición con el comando `mkfs`. Use `partprobe` para hacer que el sistema operativo tenga en cuenta el cambio.

```bash
sudo mkfs.xfs /dev/sdc1
sudo partprobe /dev/sdc1
```

Monte el disco nuevo para que sea accesible en el sistema operativo.

```bash
sudo mkdir /datadrive && sudo mount /dev/sdc1 /datadrive
```

Ahora se puede acceder al disco mediante el punto de montaje `/datadrive`, lo que se puede comprobar con el comando `df -h`.

```bash
df -h | grep -i "sd"
```

El resultado muestra la nueva unidad montada en `/datadrive`.

```bash
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        29G  2.0G   27G   7% /
/dev/sda15      105M  3.6M  101M   4% /boot/efi
/dev/sdb1        14G   41M   13G   1% /mnt
/dev/sdc1        50G   52M   47G   1% /datadrive
```

Para asegurarse de que la unidad se vuelve a montar tras un reinicio, se debe agregar al archivo */etc/fstab*. Para ello, obtenga el UUID del disco con la utilidad `blkid`.

```bash
sudo -i blkid
```

El resultado muestra el UUID de la unidad, en este caso `/dev/sdc1`.

```bash
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="xfs"
```

> [!NOTE]
> La edición incorrecta del archivo **/etc/fstab** puede tener como resultado un sistema que no se pueda arrancar. Si no está seguro, consulte la documentación de distribución para obtener información sobre cómo editar correctamente ese archivo. También se recomienda realizar una copia de seguridad del archivo /etc/fstab antes de editarlo.

Abra el archivo `/etc/fstab` en un editor de texto tal y como se indica a continuación:

```bash
sudo nano /etc/fstab
```

Agregue una línea similar a la siguiente al archivo */etc/fstab* y reemplace el valor de UUID por el suyo propio.

```bash
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive  xfs    defaults,nofail   1  2
```

Cuando haya terminado de editar el archivo, use `Ctrl+O` para escribir el archivo y `Ctrl+X` para salir del editor.

Ahora que se ha configurado el disco, cierre la sesión de SSH.

```bash
exit
```

## <a name="take-a-disk-snapshot"></a>Tomar una instantánea de disco

Cuando se toma una instantánea de un disco, Azure crea una copia de solo lectura y de un momento dado del disco. Las instantáneas de máquina virtual de Azure resultan útiles para guardar rápidamente el estado de una máquina virtual antes de realizar cambios en la configuración. Si se produce un error, se puede restaurar la máquina virtual mediante una instantánea. Cuando una máquina virtual tiene más de un disco, se toma una instantánea de cada uno con independencia de los demás. Para realizar copias de seguridad coherentes con la aplicación, considere la posibilidad de detener la máquina virtual antes de tomar instantáneas de disco. Como alternativa, use el [servicio Azure Backup](../../backup/index.yml), que permite realizar copias de seguridad automatizadas mientras se ejecuta la máquina virtual.

### <a name="create-snapshot"></a>Create snapshot

Antes de crear una instantánea, necesita el identificador o el nombre del disco. Utilice [az vm show](/cli/azure/vm#az-vm-show) para mostrar el identificador del disco. En este ejemplo, el identificador del disco se almacena en una variable para que se pueda usar en un paso posterior.

```azurecli-interactive
osdiskid=$(az vm show \
   -g myResourceGroupDisk \
   -n myVM \
   --query "storageProfile.osDisk.managedDisk.id" \
   -o tsv)
```

Ahora que tiene el identificador, utilice [az snapshot create](/cli/azure/snapshot#az-snapshot-create) para crear una instantánea del disco.

```azurecli-interactive
az snapshot create \
    --resource-group myResourceGroupDisk \
    --source "$osdiskid" \
    --name osDisk-backup
```

### <a name="create-disk-from-snapshot"></a>Creación del disco a partir de la instantánea

Esta instantánea se puede convertir en un disco mediante [az disk create](/cli/azure/disk#az-disk-create), que se puede usar para volver a crear la máquina virtual.

```azurecli-interactive
az disk create \
   --resource-group myResourceGroupDisk \
   --name mySnapshotDisk \
   --source osDisk-backup
```

### <a name="restore-virtual-machine-from-snapshot"></a>Restauración de la máquina virtual a partir de la instantánea

Para demostrar la recuperación de la máquina virtual, elimine la máquina virtual existente con [az vm delete](/cli/azure/vm#az-vm-delete).

```azurecli-interactive
az vm delete \
--resource-group myResourceGroupDisk \
--name myVM
```

Cree una nueva máquina virtual en el disco de instantáneas.

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupDisk \
    --name myVM \
    --attach-os-disk mySnapshotDisk \
    --os-type linux
```

### <a name="reattach-data-disk"></a>Reconexión de un disco de datos

Todos los discos de datos se deben volver a conectar a la máquina virtual.

Busque el nombre del disco de datos con el comando [az disk list](/cli/azure/disk#az-disk-list). En este ejemplo se coloca el nombre del disco en una variable denominada `datadisk`, que se usa en el paso siguiente.

```azurecli-interactive
datadisk=$(az disk list \
   -g myResourceGroupDisk \
   --query "[?contains(name,'myVM')].[id]" \
   -o tsv)
```

Use el comando [az vm disk attach](/cli/azure/vm/disk#az-vm-disk-attach) para adjuntar el disco.

```azurecli-interactive
az vm disk attach \
   –g myResourceGroupDisk \
   --vm-name myVM \
   --name $datadisk
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido sobre temas relacionados con los discos de máquina virtual; por ejemplo:

> [!div class="checklist"]
> * Discos del SO y temporales.
> * Discos de datos.
> * Discos Estándar y Premium.
> * Rendimiento de disco.
> * Conectar y preparar los discos de datos
> * Instantáneas de disco.

Siga con el siguiente tutorial para aprender sobre la automatización de la configuración de la máquina virtual.

> [!div class="nextstepaction"]
> [Automatización de la configuración de máquinas virtuales](./tutorial-automate-vm-deployment.md)
