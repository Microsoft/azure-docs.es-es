---
title: Creación de una imagen a partir de una máquina virtual
description: Aprenda a crear una imagen en Shared Image Gallery desde una máquina virtual de Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/01/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: devx-track-azurecli
ms.openlocfilehash: fa4a649115f8e89c27f435888b682b7de36e9894
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87533927"
---
# <a name="create-an-image-version-from-a-vm-in-azure-using-the-azure-cli"></a>Creación de la versión de una imagen a partir de una máquina virtual de Azure mediante la CLI de Azure

Si tiene una máquina virtual que le gustaría usar para crear varias máquinas virtuales idénticas, utilícela para crear una imagen en una instancia de Shared Image Gallery mediante la CLI de Azure. También puede crear una máquina virtual usando [Azure PowerShell](image-version-vm-powershell.md).

La **versión de una imagen** es lo que se usa para crear una máquina virtual cuando se utiliza una instancia de Shared Image Gallery. Puede tener varias versiones de una imagen según sea necesario para su entorno. Cuando se usa una versión de la imagen para crear una máquina virtual, se utiliza para crear discos para la nueva máquina virtual. Las versiones de las imágenes pueden usarse varias veces.


## <a name="before-you-begin"></a>Antes de empezar

Para llevar a cabo los pasos de este artículo, debe tener una instancia de Shared Image Gallery. 

También debe tener una máquina virtual en Azure, en la misma región que la galería. 

Si la máquina virtual tiene asociado un disco de datos, el tamaño del disco de datos no puede ser mayor de 1 TB.

Al trabajar en este artículo, reemplace los nombres de los recursos cuando proceda.

## <a name="get-information-about-the-vm"></a>Obtención de información acerca de la máquina virtual

Puede ver una lista de las máquinas virtuales que están disponibles mediante [az vm list](/cli/azure/vm#az-vm-list). 

```azurecli-interactive
az vm list --output table
```

Una vez que conozca el nombre de la máquina virtual y el grupo de recursos en el que se encuentra, obtenga el identificador de la máquina virtual con [az vm get-instance-view](/cli/azure/vm#az-vm-get-instance-view). 

```azurecli-interactive
az vm get-instance-view -g MyResourceGroup -n MyVm --query id
```


## <a name="create-an-image-definition"></a>Creación de la definición de una imagen

Las definiciones de imagen crean una agrupación lógica de imágenes. Estas se usan para administrar la información sobre las versiones de la imagen que se crean dentro de ellas. 

Los nombres de las definiciones de imagen pueden estar formados por letras mayúsculas o minúsculas, números, puntos y guiones. 

Asegúrese de que la definición de la imagen sea del tipo correcto. Si ha generalizado la máquina virtual (con Sysprep para Windows o waagent-deprovision para Linux), debe crear una definición de imagen generalizada mediante `--os-state generalized`. Si quiere usar la máquina virtual sin quitar las cuentas de usuario existentes, cree una definición de imagen especializada mediante `--os-state specialized`.

Para más información sobre los valores que se pueden especificar para una definición de imagen, consulte [Definiciones de imagen](./linux/shared-image-galleries.md#image-definitions).

Cree una definición de imagen en la galería mediante [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create).

En este ejemplo, la definición de la imagen se denomina *myImageDefinition* y es para una imagen del sistema operativo Linux [especializada](./linux/shared-image-galleries.md#generalized-and-specialized-images). Para crear una definición de las imágenes que emplean un sistema operativo Windows, use `--os-type Windows`. 

```azurecli-interactive 
az sig image-definition create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --publisher myPublisher \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --os-state specialized
```


## <a name="create-the-image-version"></a>Creación de la versión de la imagen

Cree una versión de la imagen a partir de la máquina virtual con [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create).  

Los caracteres permitidos para la versión de una imagen son números y puntos. Los números deben estar dentro del rango de un entero de 32 bits. Formato: *VersiónPrincipal*.*VersiónSecundaria*.*Revisión*.

En este ejemplo, es la versión de la imagen es la *1.0.0* y vamos a crear dos réplicas en la región *Centro-oeste de EE. UU.* , una réplica en la región *Centro-sur de EE. UU.* y una réplica en la región *Este de EE. UU. 2* región mediante almacenamiento con redundancia de zona. Las regiones de replicación deben incluir la región donde se encuentra la máquina virtual de origen.

Reemplace el valor de `--managed-image` en este ejemplo por el identificador de la máquina virtual del paso anterior.

```azurecli-interactive 
az sig image-version create \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --target-regions "westcentralus" "southcentralus=1" "eastus=1=standard_zrs" \
   --replica-count 2 \
   --managed-image "/subscriptions/<Subscription ID>/resourceGroups/MyResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"
```

> [!NOTE]
> Deberá esperar a que la versión de la imagen termine de compilarse y replicarse por completo antes de poder usar la misma imagen administrada para crear otra versión de la imagen.
>
> También puede almacenar la imagen en almacenamiento Premium agregando `--storage-account-type  premium_lrs`, o en el [almacenamiento con redundancia de zona](../storage/common/storage-redundancy.md) agregando `--storage-account-type  standard_zrs` al crear la versión de la imagen.
>

## <a name="next-steps"></a>Pasos siguientes

Ahora puede crear una máquina virtual a partir de la [imagen generalizada](vm-generalized-image-version-cli.md) mediante la CLI de Azure.

Para saber cómo proporcionar información del plan de compra, consulte [Indicación de la información del plan de compra de Azure Marketplace al crear imágenes](marketplace-images.md).
