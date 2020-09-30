---
title: 'CLI: Creación de una imagen a partir de una instantánea o un disco administrado en Shared Image Gallery'
description: Obtenga información sobre cómo crear una imagen a partir de una instantánea o disco administrado en Shared Image Gallery con la CLI de Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 06/30/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: e694630d8bcd7879d9405152c4141fb6e5bad4e2
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89297100"
---
# <a name="create-an-image-from-a-managed-disk-or-snapshot-in-a-shared-image-gallery-using-the-azure-cli"></a>Creación de una imagen a partir de un disco administrado o una instantánea en Shared Image Gallery mediante la CLI de Azure

Si tiene una instantánea o disco administrado existente que quiera migrar a Shared Image Gallery, puede crear una imagen de Shared Image Gallery directamente a partir del disco administrado o la instantánea. Una vez que haya probado la nueva imagen, puede eliminar el disco administrado o instantánea de origen. También puede crear una imagen a partir de un disco administrado o una instantánea en Shared Image Gallery mediante [Azure PowerShell](image-version-snapshot-powershell.md).

Las imágenes de una galería de imágenes tienen dos componentes, que se crearán durante este ejemplo:
- Una **definición de imagen**, que contiene información sobre la imagen y los requisitos para usarla. Esto incluye si la imagen es Windows o Linux, si es generalizada o especializada, las notas de la versión y los requisitos de memoria mínima y máxima. Es una definición de un tipo de imagen. 
- Una **versión de imagen** es lo que se usa para crear una máquina virtual cuando se usa Shared Image Gallery. Puede tener varias versiones de una imagen según sea necesario para su entorno. Cuando crea una máquina virtual, la versión de la imagen se usa para crear nuevos discos para dicha máquina virtual. Las versiones de las imágenes pueden usarse varias veces.


## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, debe tener una instantánea o disco administrado. 

Si quiere incluir un disco de datos, el tamaño del disco de datos no puede ser superior a 1 TB.

Al trabajar en este artículo, reemplace los nombres de los recursos cuando proceda.

## <a name="find-the-snapshot-or-managed-disk"></a>Búsqueda de la instantánea o del disco administrado 

Puede ver una lista de instantáneas disponibles en un grupo de recursos con [az snapshot list](/cli/azure/snapshot#az-snapshot-list). 

```azurecli-interactive
az snapshot list --query "[].[name, id]" -o tsv
```

También puede usar un disco administrado en lugar de una instantánea. Para obtener un disco administrado, utilice [az disk list](/cli/azure/disk#az-disk-list). 

```azurecli-interactive
az disk list --query "[].[name, id]" -o tsv
```

Cuando tenga el identificador de la instantánea o disco administrado, asígnelo a una variable llamada `$source` que se usará más adelante.

Puede usar el mismo proceso para obtener los discos de datos que quiera incluir en la imagen. Asígnelos a variables y, a continuación, use esas variables más adelante cuando cree la versión de la imagen.


## <a name="find-the-gallery"></a>Búsqueda de la galería

Necesitará información sobre la galería de imágenes para crear la definición de la imagen.

Muestre la información sobre las galerías de imágenes disponibles mediante [az sig list](/cli/azure/sig#az-sig-list). Tome nota del nombre de la galería y el grupo de recursos en el que se encuentra, ya que se usará más adelante.

```azurecli-interactive 
az sig list -o table
```


## <a name="create-an-image-definition"></a>Creación de la definición de una imagen

Las definiciones de imagen crean una agrupación lógica de imágenes. Se usan para administrar la información acerca de la imagen. Los nombres de las definiciones de imagen pueden estar formados por letras mayúsculas o minúsculas, números, puntos y guiones. 

Al crear la definición de la imagen, asegúrese de tener toda la información correcta. En este ejemplo, se da por hecho que la instantánea o el disco administrado proceden de una máquina virtual que está en uso y que no se ha generalizado. Si se tomó el disco administrado o la instantánea de un sistema operativo generalizado (después de ejecutar Sysprep para Windows o [waagent](https://github.com/Azure/WALinuxAgent) `-deprovision` o `-deprovision+user` para Linux), cambie `-OsState` a `generalized`. 

Para más información sobre los valores que se pueden especificar para una definición de imagen, consulte [Definiciones de imagen](./linux/shared-image-galleries.md#image-definitions).

Cree una definición de imagen en la galería mediante [az sig image-definition create](/cli/azure/sig/image-definition#az-sig-image-definition-create).

En este ejemplo, la definición de la imagen se denomina *myImageDefinition* y es para una imagen del sistema operativo Linux [especializada](./linux/shared-image-galleries.md#generalized-and-specialized-images). Para crear una definición para las imágenes que usan un sistema operativo Windows, utilice `--os-type Windows`. 

En este ejemplo, la galería se denomina *myGallery*, se encuentra en el grupo de recursos *myGalleryRG* y el nombre de la definición de imagen será *mImageDefinition*.

```azurecli-interactive 
resourceGroup=myGalleryRG
gallery=myGallery
imageDef=myImageDefinition
az sig image-definition create \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --gallery-image-definition $imageDef \
   --publisher myPublisher \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --os-state specialized
```


## <a name="create-the-image-version"></a>Creación de la versión de la imagen

Cree una versión de la imagen con [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create). 

Los caracteres permitidos para la versión de una imagen son números y puntos. Los números deben estar dentro del rango de un entero de 32 bits. Formato: *VersiónPrincipal*.*VersiónSecundaria*.*Revisión*.

En este ejemplo, la versión de la imagen es *1.0.0* y vamos a crear una réplica en la región *Centro-sur de EE. UU.* y otra en la región *Este de EE. UU. 2* mediante el almacenamiento con redundancia de zona. Al elegir las regiones de destino de la replicación, recuerde que también debe incluir la región de *origen* del disco administrado o la instantánea como destino de la replicación.

Use el identificador del disco administrado o la instantánea en el parámetro `--os-snapshot`.


```azurecli-interactive 
az sig image-version create \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --gallery-image-definition $imageDef \
   --gallery-image-version 1.0.0 \
   --target-regions "southcentralus=1" "eastus2=1=standard_zrs" \
   --replica-count 2 \
   --os-snapshot $source
```

Si quiere incluir discos de datos en la imagen, debe incluir el parámetro `--data-snapshot-luns` establecido en el número LUN y el parámetro `--data-snapshots` establecido en el id. del disco de datos o la instantánea.

> [!NOTE]
> Deberá esperar a que la versión de la imagen termine de compilarse y replicarse por completo antes de poder usar la misma imagen administrada para crear otra versión de la imagen.
>
> También puede almacenar todas las réplicas de la versión de la imagen en [almacenamiento con redundancia de zona](../storage/common/storage-redundancy.md) si agrega `--storage-account-type standard_zrs` al crear la versión de la imagen.
>

## <a name="next-steps"></a>Pasos siguientes

Creación de una VM a partir de una [versión de imagen especializada](vm-specialized-image-version-cli.md).

Para obtener información sobre cómo proporcionar información del plan de compra, consulte [Indicación de la información del plan de compra de Azure Marketplace al crear imágenes](marketplace-images.md).
