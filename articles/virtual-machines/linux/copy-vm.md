---
title: Copia de una máquina virtual Linux mediante la CLI de Azure
description: Aprenda a crear una copia de la máquina virtual Linux de Azure mediante la CLI de Azure y Managed Disks.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 10/17/2018
ms.author: cynthn
ms.custom: legacy
ms.openlocfilehash: ec8fd91dc768b44b027c96efead6924782eb0f75
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87368615"
---
# <a name="create-a-copy-of-a-linux-vm-by-using-azure-cli-and-managed-disks"></a>Creación de una copia de una máquina virtual Linux mediante la CLI de Azure y Managed Disks

En este artículo se muestra cómo crear una copia de su máquina virtual (VM) de Azure con Linux mediante la CLI de Azure. Para copiar, crear, almacenar y compartir imágenes de VM a escala, consulte [Shared Image Gallery](../shared-images-cli.md).

También puede [cargar y crear una máquina virtual a partir de un VHD](upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

## <a name="prerequisites"></a>Prerrequisitos

-   Instale la [CLI de Azure](/cli/azure/install-az-cli2).

-   Iniciar sesión en una cuenta de Azure con [az login](/cli/azure/reference-index#az-login).

-   Tener una máquina virtual de Azure como el origen de la copia.

## <a name="stop-the-source-vm"></a>Detención de la máquina virtual de origen

Desasigne la máquina virtual de origen mediante [az vm deallocate](/cli/azure/vm#az-vm-deallocate).
En el ejemplo siguiente se desasigna la máquina virtual denominada "*myVM*" en el grupo de recursos *myResourceGroup*:

```azurecli
az vm deallocate \
    --resource-group myResourceGroup \
    --name myVM
```

## <a name="copy-the-source-vm"></a>Copia de la máquina virtual de origen

Para copiar una VM, se crea una copia del disco duro virtual subyacente. Mediante este proceso, se crea un disco duro virtual (VHD) especializado como disco administrado que contiene la misma configuración que la máquina virtual de origen.

Para más información sobre Azure Managed Disks, consulte [Azure Managed Disks overview](../windows/managed-disks-overview.md) (Introducción a Azure Managed Disks). 

1.  Enumere cada máquina virtual y el nombre de su disco del SO con [az vm list](/cli/azure/vm#az-vm-list). En el ejemplo siguiente se enumeran todas las VM del grupo de recursos denominado *myResourceGroup*:
    
    ```azurecli
    az vm list -g myResourceGroup \
         --query '[].{Name:name,DiskName:storageProfile.osDisk.name}' \
         --output table
    ```

    La salida es similar a la del ejemplo siguiente:

    ```azurecli
    Name    DiskName
    ------  --------
    myVM    myDisk
    ```

1.  Copie el disco mediante la creación de un disco administrado con [az disk create](/cli/azure/disk#az-disk-create). En el ejemplo siguiente se crea un disco llamado "*myCopiedDisk*" a partir del disco administrado *myDisk*:

    ```azurecli
    az disk create --resource-group myResourceGroup \
         --name myCopiedDisk --source myDisk
    ``` 

1.  Compruebe ahora los discos administrados del grupo de recursos mediante [az disk list](/cli/azure/disk#az-disk-list). En el ejemplo siguiente se enumeran todos los discos administrados del grupo de recursos *myResourceGroup*:

    ```azurecli
    az disk list --resource-group myResourceGroup --output table
    ```


## <a name="set-up-a-virtual-network"></a>Configuración de una red virtual

Con los siguientes pasos opcionales se crea una nueva red virtual, una subred, una dirección IP pública y una tarjeta de interfaz de red virtual (NIC).

Si va a copiar una máquina virtual para solucionar problemas o para realizar implementaciones adicionales, puede que no desee usar una máquina virtual de una red virtual existente.

Si desea crear una infraestructura de red virtual para las máquinas virtuales copiadas, siga estos pasos. Si no desea crear una red virtual, vaya a [Creación de una máquina virtual](#create-a-vm).

1.  Cree la red virtual mediante [az network vnet create](/cli/azure/network/vnet#az-network-vnet-create). En el ejemplo siguiente se crea una red virtual denominada *myVnet* y una subred *mySubnet*:

    ```azurecli
    az network vnet create --resource-group myResourceGroup \
        --location eastus --name myVnet \
        --address-prefix 192.168.0.0/16 \
        --subnet-name mySubnet \
        --subnet-prefix 192.168.1.0/24
    ```

1.  Cree una IP pública mediante [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create). En el ejemplo siguiente se crea una IP pública denominada "*myPublicIP*" con el nombre DNS *mypublicdns*. (El nombre DNS debe ser único, así que proporcione un nombre único).

    ```azurecli
    az network public-ip create --resource-group myResourceGroup \
        --location eastus --name myPublicIP --dns-name mypublicdns \
        --allocation-method static --idle-timeout 4
    ```

1.  Cree la NIC mediante [az network nic create](/cli/azure/network/nic#az-network-nic-create).
    En el ejemplo siguiente se crea una NIC *myNic* conectada a la subred *mySubnet*:

    ```azurecli
    az network nic create --resource-group myResourceGroup \
        --location eastus --name myNic \
        --vnet-name myVnet --subnet mySubnet \
        --public-ip-address myPublicIP
    ```

## <a name="create-a-vm"></a>Crear una VM

Cree una máquina virtual mediante [az vm create](/cli/azure/vm#az-vm-create).

Especifique el disco administrado copiado que usará como disco del SO (`--attach-os-disk`), como se indica a continuación:

```azurecli
az vm create --resource-group myResourceGroup \
    --name myCopiedVM --nics myNic \
    --size Standard_DS1_v2 --os-type Linux \
    --attach-os-disk myCopiedDisk
```

## <a name="next-steps"></a>Pasos siguientes

Aprenda a usar una [galería de imágenes compartidas](../shared-images-cli.md) para administrar imágenes de máquinas virtuales.
