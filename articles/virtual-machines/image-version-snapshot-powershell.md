---
title: 'PowerShell: creación de una imagen a partir de una instantánea o un disco administrado en Shared Image Gallery'
description: Obtenga información sobre cómo crear una imagen a partir de una instantánea o disco administrado en Shared Image Gallery con PowerShell.
author: cynthn
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: imaging
ms.workload: infrastructure
ms.date: 06/30/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 2ebff0d86c27bcdbc11d23e18116b33b4ea838a6
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89300262"
---
# <a name="create-an-image-from-a-managed-disk-or-snapshot-in-a-shared-image-gallery-using-powershell"></a>Creación de una imagen a partir de un disco administrado o una instantánea en Shared Image Gallery mediante PowerShell

Si tiene una instantánea o disco administrado existente que quiera migrar a Shared Image Gallery, puede crear una imagen de Shared Image Gallery directamente a partir del disco administrado o la instantánea. Una vez que haya probado la nueva imagen, puede eliminar el disco administrado o instantánea de origen. También puede crear una imagen a partir de un disco administrado o una instantánea en Shared Image Gallery mediante la [CLI de Azure](image-version-snapshot-cli.md).

Las imágenes de una galería de imágenes tienen dos componentes, que se crearán durante este ejemplo:
- Una **definición de imagen**, que contiene información sobre la imagen y los requisitos para usarla. Esto incluye si la imagen es Windows o Linux, si es generalizada o especializada, las notas de la versión y los requisitos de memoria mínima y máxima. Es una definición de un tipo de imagen. 
- Una **versión de imagen** es lo que se usa para crear una máquina virtual cuando se usa Shared Image Gallery. Puede tener varias versiones de una imagen según sea necesario para su entorno. Cuando crea una máquina virtual, la versión de la imagen se usa para crear nuevos discos para dicha máquina virtual. Las versiones de las imágenes pueden usarse varias veces.


## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, debe tener una instantánea o disco administrado. 

Si desea incluir un disco de datos, el tamaño del disco de datos no puede ser superior a 1 TB.

Al trabajar en este artículo, reemplace los nombres de los recursos cuando proceda.


## <a name="get-the-snapshot-or-managed-disk"></a>Obtención de la instantánea o del disco administrado

Puede ver una lista de instantáneas disponibles en un grupo de recursos con [Get-AzSnapshot](/powershell/module/az.compute/get-azsnapshot). 

```azurepowershell-interactive
get-azsnapshot | Format-Table -Property Name,ResourceGroupName
```

Una vez que sepa el nombre de la instantánea y cuál es el grupo de recursos al que pertenece, puede usar `Get-AzSnapshot` nuevamente para obtener el objeto de instantánea y almacenarlo en una variable para su uso posterior. En este ejemplo se obtiene una instantánea denominada *mySnapshot* del grupo de recursos "myResourceGroup" y se asigna a la variable *$source*. 

```azurepowershell-interactive
$source = Get-AzSnapshot `
   -SnapshotName mySnapshot `
   -ResourceGroupName myResourceGroup
```

También puede usar un disco administrado en lugar de una instantánea. Para obtener un disco administrado, use [Get-AzDisk](/powershell/module/az.compute/get-azdisk). 

```azurepowershell-interactive
Get-AzDisk | Format-Table -Property Name,ResourceGroupName
```

A continuación, obtenga el disco administrado y asígnelo a la variable `$source`.

```azurepowershell-interactive
$source = Get-AzDisk `
   -Name myDisk
   -ResourceGroupName myResourceGroup
```

Puede usar los mismos cmdlets para obtener los discos de datos que desee incluir en la imagen. Asígnelos a variables y, a continuación, use esas variables más adelante cuando cree la versión de la imagen.


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

Al crear la definición de la imagen, asegúrese de tener toda la información correcta. En este ejemplo, se da por hecho que la instantánea o el disco administrado proceden de una VM que está en uso y que no se ha generalizado. Si se tomó el disco administrado o la instantánea de un sistema operativo generalizado (después de ejecutar Sysprep para Windows o [waagent](https://github.com/Azure/WALinuxAgent) `-deprovision` o `-deprovision+user` para Linux), cambie `-OsState` a `generalized`. 

Para más información sobre los valores que se pueden especificar para una definición de imagen, consulte [Definiciones de imagen](./windows/shared-image-galleries.md#image-definitions).

Cree la definición de imagen mediante [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion). En este ejemplo, la definición de la imagen se denomina *myImageDefinition* y es para un sistema operativo Windows especializado. Para crear una definición para las imágenes que usan un sistema operativo Linux, utilice `-OsType Linux`. 

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

### <a name="purchase-plan-information"></a>Información sobre el plan de compra

En algunos casos, debe pasar la información del plan de compra al crear una máquina virtual a partir de una imagen basada en una imagen de Azure Marketplace. En estos casos, se recomienda incluir la información del plan de compra en la definición de la imagen. En este caso, consulte [Indicación de la información del plan de compra de Azure Marketplace al crear imágenes](marketplace-images.md).


## <a name="create-an-image-version"></a>Creación de la versión de una imagen

Cree una versión de la imagen a partir de la instantánea mediante [New-AzGalleryImageVersion](/powershell/module/az.compute/new-azgalleryimageversion). 

Los caracteres permitidos para la versión de una imagen son números y puntos. Los números deben estar dentro del rango de un entero de 32 bits. Formato: *VersiónPrincipal*.*VersiónSecundaria*.*Revisión*.

Si quiere que la imagen contenga un disco de datos, además del disco del sistema operativo, agregue el parámetro `-DataDiskImage` y defínalo con el identificador de la instantánea de disco de datos o disco administrado.

En este ejemplo, la versión de imagen es *1.0.0* y se replica en los centros de datos *Centro-oeste de EE. UU.* y *Centro-sur de EE. UU..* Al elegir las regiones de destino de la replicación, recuerde que también debe incluir la región de *origen* como destino de la replicación.


```azurepowershell-interactive
$region1 = @{Name='South Central US';ReplicaCount=1}
$region2 = @{Name='West Central US';ReplicaCount=2}
$targetRegions = @($region1,$region2)
$job = $imageVersion = New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $imageDefinition.Name `
   -GalleryImageVersionName '1.0.0' `
   -GalleryName $gallery.Name `
   -ResourceGroupName $gallery.ResourceGroupName `
   -Location $gallery.Location `
   -TargetRegion $targetRegions  `
   -OSDiskImage @{Source = @{Id=$source.Id}; HostCaching = "ReadOnly" } `
   -PublishingProfileEndOfLifeDate '2025-01-01' `
   -asJob 
```

Puede tardar un rato en replicar la imagen en todas las regiones de destino, por lo que creamos un trabajo para que pueda hacer un seguimiento del progreso. Para ver el progreso del trabajo, escriba `$job.State`.

```azurepowershell-interactive
$job.State
```

> [!NOTE]
> Deberá esperar a que la versión de la imagen termine de compilarse y replicarse por completo antes de poder usar la misma instantánea para crear otra versión de la imagen. 
>
> También puede almacenar la versión de la imagen en [almacenamiento con redundancia de zona](../storage/common/storage-redundancy.md) si agrega `-StorageAccountType Standard_ZRS` al crear la versión de la imagen.
>

## <a name="delete-the-source"></a>Eliminación del origen

Una vez que haya comprobado que la nueva versión de la imagen funciona correctamente, puede eliminar el origen de la imagen con [Remove-AzSnapshot](/powershell/module/Az.Compute/Remove-AzSnapshot) o [Remove-AzDisk](/powershell/module/az.compute/remove-azdisk).


## <a name="next-steps"></a>Pasos siguientes

Una vez que haya comprobado que la replicación ha finalizado, puede crear una máquina virtual a partir de la [imagen especializada](vm-specialized-image-version-powershell.md).

Para obtener información sobre cómo proporcionar información del plan de compra, consulte [Indicación de la información del plan de compra de Azure Marketplace al crear imágenes](marketplace-images.md).
