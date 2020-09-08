---
title: Creación de una imagen a partir de una máquina virtual (versión preliminar)
description: Obtenga información sobre cómo usar Azure PowerShell para crear una imagen en Shared Image Gallery a partir de una máquina virtual en Azure.
author: cynthn
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 757b297d3d74365928cda0934485c0018f28ffee
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88225655"
---
# <a name="preview-create-an-image-from-a-vm"></a>Vista previa: Creación de una imagen a partir de una máquina virtual

Si tiene una máquina virtual existente que le gustaría usar para crear varias máquinas virtuales idénticas, puede usarla para crear una imagen en una instancia de Shared Image Gallery mediante Azure PowerShell. También puede crear una imagen a partir de una máquina virtual usando la [CLI de Azure](image-version-vm-cli.md).

Puede capturar una imagen a partir de máquinas virtuales [especializadas y generalizadas](./windows/shared-image-galleries.md#generalized-and-specialized-images) mediante Azure PowerShell. 

Las imágenes de una galería de imágenes tienen dos componentes, que se crearán durante este ejemplo:
- Una **definición de imagen**, que contiene información sobre la imagen y los requisitos para usarla. Esto incluye si la imagen es Windows o Linux, si es generalizada o especializada, las notas de la versión y los requisitos de memoria mínima y máxima. Es una definición de un tipo de imagen. 
- Una **versión de imagen** es lo que se usa para crear una máquina virtual cuando se usa Shared Image Gallery. Puede tener varias versiones de una imagen según sea necesario para su entorno. Cuando crea una máquina virtual, la versión de la imagen se usa para crear nuevos discos para dicha máquina virtual. Las versiones de las imágenes pueden usarse varias veces.


## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, debe tener una instancia existente de Shared Image Gallery, así como una máquina virtual existente en Azure para usarla como origen. 

Si la máquina virtual tiene conectado un disco de datos, el tamaño del mismo no puede ser mayor que 1 TB.

Al trabajar en este artículo, reemplace los nombres de los recursos y de la máquina virtual cuando proceda.


## <a name="get-the-gallery"></a>Obtención de la galería

Puede enumerar todas las galerías y definiciones de imagen por nombre. Los resultados usan el formato `gallery\image definition\image version`.

```azurepowershell-interactive
Get-AzResource -ResourceType Microsoft.Compute/galleries | Format-Table
```

Una vez que encuentre la adecuada y las definiciones de imagen adecuadas, cree variables para usarlas más adelante. En este ejemplo, se obtiene la galería denominada *myGallery* del grupo de recursos *myResourceGroup*.

```azurepowershell-interactive
$gallery = Get-AzGallery `
   -Name myGallery `
   -ResourceGroupName myResourceGroup
```

## <a name="get-the-vm"></a>Obtención de la máquina virtual

Puede ver una lista de máquinas virtuales disponibles en un grupo de recursos con [Get-AzVM](/powershell/module/az.compute/get-azvm). Una vez que sepa el nombre de la máquina virtual y cuál es el grupo de recursos al que pertenece, puede usar `Get-AzVM` nuevamente para obtener el objeto de máquina virtual y almacenarlo en una variable para su uso posterior. En este ejemplo se obtiene una máquina virtual denominada *sourceVM* del grupo de recursos "myResourceGroup" y se asigna a la variable *$sourceVm*. 

```azurepowershell-interactive
$sourceVm = Get-AzVM `
   -Name sourceVM `
   -ResourceGroupName myResourceGroup
```

Se recomienda detener o desasignar la máquina virtual antes de crear una imagen mediante [Stop-AzVM](/powershell/module/az.compute/stop-azvm).

```azurepowershell-interactive
Stop-AzVM `
   -ResourceGroupName $sourceVm.ResourceGroupName `
   -Name $sourceVm.Name `
   -Force
```

## <a name="create-an-image-definition"></a>Creación de la definición de una imagen 

Las definiciones de imagen crean una agrupación lógica de imágenes. Se usan para administrar la información acerca de la imagen. Los nombres de las definiciones de imagen pueden estar formados por letras mayúsculas o minúsculas, números, puntos y guiones. 

Al crear la definición de la imagen, asegúrese de tener toda la información correcta. Si ha hecho generalizada la máquina virtual (con Sysprep para Windows o waagent-deprovision para Linux), debe crear una definición de imagen mediante `-OsState generalized`. Si no ha generalizado la máquina virtual, cree una definición de imagen mediante `-OsState specialized`.

Para más información sobre los valores que se pueden especificar para una definición de imagen, consulte [Definiciones de imagen](./windows/shared-image-galleries.md#image-definitions).

Cree la definición de imagen mediante [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion). 

En este ejemplo, la definición de la imagen se denomina *myImageDefinition* y es para una máquina virtual especializada que ejecuta Windows. Para crear una definición para las imágenes que usan Linux, use `-OsType Linux`. 

```azurepowershell-interactive
$imageDefinition = New-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -Name 'myImageDefinition' `
   -OsState specialized `
   -OsType Windows `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```


## <a name="create-an-image-version"></a>Creación de la versión de una imagen

Cree una versión de la imagen mediante [New-AzGalleryImageVersion](/powershell/module/az.compute/new-azgalleryimageversion). 

Los caracteres permitidos para la versión de una imagen son números y puntos. Los números deben estar dentro del rango de un entero de 32 bits. Formato: *VersiónPrincipal*.*VersiónSecundaria*.*Revisión*.

En este ejemplo, la versión de imagen es *1.0.0* y se replica en los centros de datos *Centro-oeste de EE. UU.* y *Centro-sur de EE. UU..* Al elegir las regiones de destino de la replicación, recuerde que también debe incluir la región de *origen* como destino de la replicación.

Para crear una versión de la imagen a partir de la máquina virtual, use `$vm.Id.ToString()` como `-Source`.

```azurepowershell-interactive
   $region1 = @{Name='South Central US';ReplicaCount=1}
   $region2 = @{Name='East US';ReplicaCount=2}
   $targetRegions = @($region1,$region2)

$job = $imageVersion = New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $imageDefinition.Name`
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -TargetRegion $targetRegions  `
   -Source $sourceVm.Id.ToString() `
   -PublishingProfileEndOfLifeDate '2020-12-01' `  
   -asJob 
```

Puede tardar un rato en replicar la imagen en todas las regiones de destino, por lo que creamos un trabajo para que pueda hacer un seguimiento del progreso. Para ver el progreso del trabajo, escriba `$job.State`.

```azurepowershell-interactive
$job.State
```

> [!NOTE]
> Deberá esperar a que la versión de la imagen termine de compilarse y replicarse por completo antes de poder usar la misma imagen administrada para crear otra versión de la imagen.
>
> También puede almacenar la imagen en almacenamiento Premium agregando `-StorageAccountType Premium_LRS`, o en el [almacenamiento con redundancia de zona](../storage/common/storage-redundancy.md) agregando `-StorageAccountType Standard_ZRS` al crear la versión de la imagen.
>

## <a name="next-steps"></a>Pasos siguientes

Una vez que haya comprobado que la nueva versión de la imagen funciona correctamente, puede crear una máquina virtual. Cree una máquina virtual a partir de una [versión de imagen especializada](vm-specialized-image-version-powershell.md) o [generalizada](vm-generalized-image-version-powershell.md).

Para saber cómo proporcionar información del plan de compra, consulte [Indicación de la información del plan de compra de Azure Marketplace al crear imágenes](marketplace-images.md).
