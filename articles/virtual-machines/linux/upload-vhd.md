---
title: Carga o copia de una máquina virtual Linux personalizada con la CLI de Azure
description: Cargar o copiar una máquina virtual personalizada con el modelo de implementación de Resource Manager y la CLI de Azure
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.assetid: a8c7818f-eb65-409e-aa91-ce5ae975c564
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: how-to
ms.date: 10/10/2019
ms.author: cynthn
ms.openlocfilehash: df2b58e0067932edd9dfa21ee1a6fbb2a5c1fdf7
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87289765"
---
# <a name="create-a-linux-vm-from-a-custom-disk-with-the-azure-cli"></a>Creación de una VM Linux desde un disco personalizado con la CLI de Azure

<!-- rename to create-vm-specialized -->

En este artículo se muestra cómo cargar un disco duro virtual (VHD) personalizado y cómo copiar un VHD existente en Azure. Después, el VHD recién creado se usa para crear nuevas máquinas virtuales (VM) Linux. Puede instalar y configurar una distribución de Linux para sus requisitos y, después, usar ese VHD para crear una nueva máquina virtual de Azure.

Para crear varias VM desde el disco personalizado, cree primero una imagen de la VM o el VHD. Para más información, vea [Creación de una imagen personalizada de una máquina virtual de Azure con la CLI](tutorial-custom-images.md).

Tiene dos opciones para crear un disco personalizado:
* Carga de un disco duro virtual
* Copia de una máquina virtual de Azure existente


## <a name="requirements"></a>Requisitos
Para completar los pasos siguientes, necesita:

- Una máquina virtual Linux que se ha preparado para su uso en Azure. En la sección [Preparación de la VM](#prepare-the-vm) de este artículo se explica cómo encontrar información de distribución sobre la instalación del agente Linux de Azure (waagent), que es necesario para conectarse a una VM con SSH.
- El archivo VHD de una [distribución de Linux aprobada por Azure](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) existente (o vea [Información para las distribuciones no aprobadas](create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)) en un disco virtual en el formato VHD. Existen varias herramientas para crear una máquina virtual y archivos VHD:
  - Instale y configure [QEMU](https://en.wikibooks.org/wiki/QEMU/Installing_QEMU) o [KVM](https://www.linux-kvm.org/page/RunningKVM), teniendo cuidado de usar VHD como formato de imagen. Si es necesario, puede [convertir una imagen](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats) con `qemu-img convert`.
  - También puede usar Hyper-V [en Windows 10](/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) o [en Windows Server 2012/2012 R2](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

> [!NOTE]
> el reciente formato VHDX no se admite en Azure. Al crear una máquina virtual, especifique un VHD como formato. Si es necesario, puede convertir discos VHDX en VHD mediante [qemu-img convert](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats) o el cmdlet de PowerShell [Convert-VHD](/powershell/module/hyper-v/convert-vhd?view=win10-ps). Azure no permite cargar VHD dinámicos, por lo que tendrá que convertir estos discos a VHD estáticos antes de cargarlos. Puede usar herramientas, como [Azure VHD Utilities for GO](https://github.com/Microsoft/azure-vhd-utils-for-go), para convertir discos dinámicos durante el proceso de carga en Azure.
> 
> 


- Asegúrese de que tiene la versión más reciente de la [CLI de Azure](/cli/azure/install-az-cli2) instalada y de que ha iniciado sesión en una cuenta de Azure con [az login](/cli/azure/reference-index#az-login).

En los ejemplos siguientes, reemplace los nombres de los parámetros del ejemplo por los suyos propios, como `myResourceGroup`, `mystorageaccount` y `mydisks`.

<a id="prepimage"> </a>

## <a name="prepare-the-vm"></a>Preparación de la VM

Azure admite varias distribuciones Linux (consulte [Distribuciones aprobadas](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)). En los artículos siguientes se describe el proceso de preparación de las distintas distribuciones de Linux admitidas en Azure:

* [Distribuciones basadas en CentOS](create-upload-centos.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Debian Linux](debian-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Oracle Linux](oracle-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Red Hat Enterprise Linux](redhat-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [SLES y openSUSE](suse-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Ubuntu](create-upload-ubuntu.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Otras: distribuciones no aprobadas](create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

Vea también las [notas de instalación de Linux](create-upload-generic.md#general-linux-installation-notes) para obtener más sugerencias generales sobre la preparación de imágenes de Linux para Azure.

> [!NOTE]
> El [Acuerdo de Nivel de Servicio de la plataforma Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/) se aplica a las VM que ejecutan Linux solo cuando una de las distribuciones aprobadas se use con los detalles de la configuración según se indica en la sección "Versiones admitidas" en [Linux en distribuciones aprobadas por Azure](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
> 
> 

## <a name="option-1-upload-a-vhd"></a>Opción 1: Carga de un disco duro virtual

Ahora puede cargar un disco duro virtual directamente en un disco administrado. Para obtener instrucciones al respecto, consulte [Carga de un disco duro virtual en Azure mediante la CLI de Azure](disks-upload-vhd-to-managed-disk-cli.md).

## <a name="option-2-copy-an-existing-vm"></a>Opción 2: Copiar una máquina virtual existente

También se puede crear una VM personalizada en Azure y, luego, copiar el disco del sistema operativo y asociarlo a una nueva VM para crear otra copia. Esto es adecuado para realizar pruebas, pero si quiere usar una VM existente de Azure como modelo para varias VM nuevas, en realidad debe crear una *imagen*. Para más información sobre cómo crear una imagen a partir de una VM existente de Azure, vea [Crear una imagen personalizada de una máquina virtual de Azure mediante la CLI](tutorial-custom-images.md).

Si quiere copiar una máquina virtual existente en otra región, quizá quiera usar azcopy para [crear una copia de un disco en otra región](disks-upload-vhd-to-managed-disk-cli.md#copy-a-managed-disk). 

Si no es así, debe tomar una instantánea de la máquina virtual y, a continuación, crear un nuevo disco duro virtual del sistema operativo a partir de la instantánea.

### <a name="create-a-snapshot"></a>Crear una instantánea

En este ejemplo se crea una instantánea de una máquina virtual denominada *myVM* en el grupo de recursos *myResourceGroup* y se crea una instantánea denominada *osDiskSnapshot*.

```azurecli
osDiskId=$(az vm show -g myResourceGroup -n myVM --query "storageProfile.osDisk.managedDisk.id" -o tsv)
az snapshot create \
    -g myResourceGroup \
    --source "$osDiskId" \
    --name osDiskSnapshot
```
###  <a name="create-the-managed-disk"></a>Creación del disco administrado

Cree un nuevo disco administrado a partir de la instantánea.

Obtenga el identificador de la instantánea. En este ejemplo, la instantánea se denomina *osDiskSnapshot* y se encuentra en el grupo de recursos *myResourceGroup*.

```azurecli
snapshotId=$(az snapshot show --name osDiskSnapshot --resource-group myResourceGroup --query [id] -o tsv)
```

Cree el disco administrado. En este ejemplo se va a crear un disco administrado denominado *myManagedDisk* a partir de la instantánea, donde el disco tiene un almacenamiento estándar y un tamaño de 128 GB.

```azurecli
az disk create \
    --resource-group myResourceGroup \
    --name myManagedDisk \
    --sku Standard_LRS \
    --size-gb 128 \
    --source $snapshotId
```

## <a name="create-the-vm"></a>Creación de la máquina virtual

Cree la VM con [az vm create](/cli/azure/vm#az-vm-create) y asocie (--attach-os-disk) el disco administrado como disco del sistema operativo. En el ejemplo siguiente se crea una VM denominada *myNewVM* mediante el disco administrado creado a partir del VHD cargado:

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --location eastus \
    --name myNewVM \
    --os-type linux \
    --attach-os-disk myManagedDisk
```

Debe ser capaz de iniciar sesión con SSH en la VM con las credenciales de la VM de origen. 

## <a name="next-steps"></a>Pasos siguientes
Después de haber preparado y cargado el disco virtual personalizado, puede leer más sobre el [uso de Resource Manager y las plantillas](../../azure-resource-manager/management/overview.md). También es posible que quiera [agregar un disco de datos](add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) a las nuevas máquinas virtuales. Si tiene aplicaciones que se ejecutan en las máquinas virtuales a las que necesite tener acceso, asegúrese de [abrir puertos y puntos de conexión](nsg-quickstart.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
