---
title: Migración de una imagen administrada a Shared Image Gallery
description: Aprenda a usar Azure PowerShell para migrar una imagen administrada a una versión de la imagen de Shared Image Gallery.
author: cynthn
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: c119ebc8d87c00608fc515099711463f47b8ed1a
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87901817"
---
# <a name="migrate-from-a-managed-image-to-a-shared-image-gallery-image"></a>Migración de una imagen administrada a una imagen de Shared Image Gallery

Si tiene una imagen administrada existente que desee migrar a Shared Image Gallery, puede crear una imagen de Shared Image Gallery directamente desde la imagen administrada. Una vez que haya probado la nueva imagen, puede eliminar la imagen administrada de origen. También puede migrar de una imagen administrada a Shared Image Gallery mediante la [CLI de Azure](image-version-managed-image-cli.md).

Las imágenes de una galería de imágenes tienen dos componentes, que se crearán durante este ejemplo:
- Una **definición de imagen**, que contiene información sobre la imagen y los requisitos para usarla. Esto incluye si la imagen es Windows o Linux, si es generalizada o especializada, las notas de la versión y los requisitos de memoria mínima y máxima. Es una definición de un tipo de imagen. 
- Una **versión de imagen** es lo que se usa para crear una máquina virtual cuando se usa Shared Image Gallery. Puede tener varias versiones de una imagen según sea necesario para su entorno. Cuando crea una máquina virtual, la versión de la imagen se usa para crear nuevos discos para dicha máquina virtual. Las versiones de las imágenes pueden usarse varias veces.


## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, debe tener una imagen administrada existente. Si la imagen administrada contiene un disco de datos, el tamaño del disco de datos no puede ser mayor de 1 TB.

Al trabajar en este artículo, reemplace los nombres de grupo de recursos y máquina virtual cuando proceda.

## <a name="get-the-gallery"></a>Obtención de la galería

Puede enumerar todas las galerías y definiciones de imagen por nombre. Los resultados usan el formato `gallery\image definition\image version`.

```azurepowershell-interactive
Get-AzResource -ResourceType Microsoft.Compute/galleries | Format-Table
```

Una vez que encuentre la galería correcta, cree una variable para usarla más adelante. En este ejemplo, se obtiene la galería denominada *myGallery* del grupo de recursos *myResourceGroup*.

```azurepowershell-interactive
$gallery = Get-AzGallery `
   -Name myGallery `
   -ResourceGroupName myResourceGroup
```


## <a name="create-an-image-definition"></a>Creación de la definición de una imagen 

Las definiciones de imagen crean una agrupación lógica de imágenes. Se usan para administrar la información acerca de la imagen. Los nombres de las definiciones de imagen pueden estar formados por letras mayúsculas o minúsculas, números, puntos y guiones. 

Al crear la definición de la imagen, asegúrese de tener toda la información correcta. Dado que las imágenes administradas siempre son generalizadas, debería establecer `-OsState generalized`. 

Para más información sobre los valores que se pueden especificar para una definición de imagen, consulte [Definiciones de imagen](./windows/shared-image-galleries.md#image-definitions).

Cree la definición de imagen mediante [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion). En este ejemplo, la definición de la imagen se denomina *myImageDefinition* y es para un sistema operativo Windows generalizado. Para crear una definición para las imágenes que usan un sistema operativo Linux, utilice `-OsType Linux`. 

```azurepowershell-interactive
$imageDefinition = New-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -Name 'myImageDefinition' `
   -OsState generalized `
   -OsType Windows `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```

## <a name="get-the-managed-image"></a>Obtener la imagen administrada

Puede ver una lista de imágenes disponibles en un grupo de recursos con [Get-AzImage](/powershell/module/az.compute/get-azimage). Una vez que sepa el nombre de la imagen y en qué grupo de recursos está, puede usar `Get-AzImage` nuevamente para obtener el objeto de imagen y almacenarlo en una variable para usarlo más adelante. En este ejemplo se obtiene una imagen denominada *myImage* desde el grupo de recursos "myResourceGroup" y se asigna a la variable *$managedImage*. 

```azurepowershell-interactive
$managedImage = Get-AzImage `
   -ImageName myImage `
   -ResourceGroupName myResourceGroup
```


## <a name="create-an-image-version"></a>Creación de la versión de una imagen

Cree una versión de la imagen a partir de la imagen administrada mediante [New-AzGalleryImageVersion](/powershell/module/az.compute/new-azgalleryimageversion). 

Los caracteres permitidos para la versión de una imagen son números y puntos. Los números deben estar dentro del rango de un entero de 32 bits. Formato: *VersiónPrincipal*.*VersiónSecundaria*.*Revisión*.

En este ejemplo, la versión de imagen es *1.0.0* y se replica en los centros de datos *Centro-oeste de EE. UU.* y *Centro-sur de EE. UU..* Al elegir las regiones de destino de la replicación, recuerde que también debe incluir la región de *origen* como destino de la replicación. 


```azurepowershell-interactive
$region1 = @{Name='South Central US';ReplicaCount=1}
$region2 = @{Name='West Central US';ReplicaCount=2}
$targetRegions = @($region1,$region2)
$job = $imageVersion = New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $imageDefinition.Name `
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $imageDefinition.ResourceGroupName `
   -Location $imageDefinition.Location `
   -TargetRegion $targetRegions  `
   -Source $managedImage.Id.ToString() `
   -PublishingProfileEndOfLifeDate '2020-12-31' `
   -asJob 
```

Puede tardar un rato en replicar la imagen en todas las regiones de destino, por lo que creamos un trabajo para que pueda hacer un seguimiento del progreso. Para ver el progreso, escriba `$job.State`.

```azurepowershell-interactive
$job.State
```


> [!NOTE]
> Deberá esperar a que la versión de la imagen termine de compilarse y replicarse por completo antes de poder usar la misma imagen administrada para crear otra versión de la imagen. 
>
> También puede almacenar la imagen en almacenamiento Premium al agregar `-StorageAccountType Premium_LRS`, o en el [almacenamiento con redundancia de zona](../storage/common/storage-redundancy.md) al agregar `-StorageAccountType Standard_ZRS` durante la creación de la versión de la imagen.
>

## <a name="delete-the-managed-image"></a>Eliminación de la imagen administrada

Una vez que haya comprobado que la nueva versión de la imagen funciona correctamente, puede eliminar la imagen administrada.

```azurepowershell-interactive
Remove-AzImage `
   -ImageName $managedImage.Name `
   -ResourceGroupName $managedImage.ResourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya comprobado que la replicación ha finalizado, puede crear una máquina virtual a partir de la [imagen generalizada](vm-generalized-image-version-powershell.md).

Para obtener información sobre cómo proporcionar información del plan de compra, consulte [Indicación de la información del plan de compra de Azure Marketplace al crear imágenes](marketplace-images.md).
