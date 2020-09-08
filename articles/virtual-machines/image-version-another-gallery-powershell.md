---
title: Copia de una imagen de otra galería mediante PowerShell
description: Copie una imagen de otra galería mediante Azure PowerShell.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: 2f26716a4d04b574bf393f502758a725948419da
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299956"
---
# <a name="copy-an-image-from-another-gallery-using-powershell"></a>Copia de una imagen de otra galería mediante PowerShell

Si tiene varias galerías en la organización, puede crear imágenes a partir de imágenes que estén almacenadas en otras galerías. Por ejemplo, puede tener una galería de desarrollo y pruebas para crear y probar imágenes nuevas. Cuando estén listas para usarse en producción, puede copiarlas a una galería de producción con este ejemplo. También pueda usar la [CLI de Azure](image-version-another-gallery-cli.md) para crear una imagen a partir de una imagen en otra galería.


## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, debe tener una galería de origen, una definición de imagen y una versión de imagen existentes. Además, debe tener una galería de destino. 

La versión de la imagen de origen se debe replicar en la región en la que se encuentra la galería de destino. 

Crearemos una definición de imagen y una versión de la imagen nuevas en la galería de destino.


Al trabajar en este artículo, reemplace los nombres de los recursos y de la máquina virtual cuando proceda.


## <a name="get-the-source-image"></a>Obtención de la imagen de origen 

Necesitará información de la definición de imagen de origen para crear una copia de ella en la galería de destino.

Use el cmdlet [Get-AzResource](/powershell/module/az.resources/get-azresource) para mostrar la información acerca de las galerías, definiciones de imagen y versiones de imagen existentes.

Los resultados usan el formato `gallery\image definition\image version`.

```azurepowershell-interactive
Get-AzResource `
   -ResourceType Microsoft.Compute/galleries/images/versions | `
   Format-Table -Property Name,ResourceGroupName
```

Una vez que tenga toda la información que necesita, puede obtener el id. de la versión de la imagen de origen mediante [Get-AzGalleryImageVersion](/powershell/module/az.compute/get-azgalleryimageversion). En este ejemplo, vamos a obtener la versión de imagen `1.0.0` de la definición `myImageDefinition`, en la galería de origen `myGallery`, en el grupo de recursos `myResourceGroup`.

```azurepowershell-interactive
$sourceImgVer = Get-AzGalleryImageVersion `
   -GalleryImageDefinitionName myImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name 1.0.0
```


## <a name="create-the-image-definition"></a>Creación de una definición de imagen 

Debe crear una definición de imagen nueva que coincida con la definición de imagen del origen. Puede ver toda la información necesaria para volver a crear la definición de imagen mediante [Get-AzGalleryImageDefinition](/powershell/module/az.compute/get-azgalleryimagedefinition).

```azurepowershell-interactive
Get-AzGalleryImageDefinition `
   -GalleryName myGallery `
   -ResourceGroupName myResourceGroup `
   -Name myImageDefinition
```


El resultado tendrá un aspecto similar al siguiente:

```output
{
  "description": null,
  "disallowed": null,
  "endOfLifeDate": null,
  "eula": null,
  "hyperVgeneration": "V1",
  "id": "/subscriptions/1111abcd-1a23-4b45-67g7-1234567de123/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition",
  "identifier": {
    "offer": "myOffer",
    "publisher": "myPublisher",
    "sku": "mySKU"
  },
  "location": "eastus",
  "name": "myImageDefinition",
  "osState": "Specialized",
  "osType": "Windows",
  "privacyStatementUri": null,
  "provisioningState": "Succeeded",
  "purchasePlan": null,
  "recommended": null,
  "releaseNoteUri": null,
  "resourceGroup": "myGalleryRG",
  "tags": null,
  "type": "Microsoft.Compute/galleries/images"
}
```

Cree una nueva definición de imagen, en la galería de destino, con el cmdlet [New-AzGalleryImageDefinition](/powershell/module/az.compute/new-azgalleryimageversion) y la información de la salida anterior.


En este ejemplo, la definición de la imagen se denomina *myDestinationImgDef* en la galería denominada *myDestinationGallery*.


```azurepowershell-interactive
$destinationImgDef  = New-AzGalleryImageDefinition `
   -GalleryName myDestinationGallery `
   -ResourceGroupName myDestinationRG `
   -Location WestUS `
   -Name 'myDestinationImgDef' `
   -OsState specialized `
   -OsType Windows `
   -HyperVGeneration v1 `
   -Publisher 'myPublisher' `
   -Offer 'myOffer' `
   -Sku 'mySKU'
```


## <a name="create-the-image-version"></a>Creación de la versión de la imagen

Cree una versión de la imagen mediante [New-AzGalleryImageVersion](/powershell/module/az.compute/new-azgalleryimageversion). Tendrá que pasar el id. de la imagen de origen en el parámetro `--managed-image` para crear la versión de imagen en la galería de destino. 

Los caracteres permitidos para la versión de una imagen son números y puntos. Los números deben estar dentro del rango de un entero de 32 bits. Formato: *VersiónPrincipal*.*VersiónSecundaria*.*Revisión*.

En este ejemplo, la galería de destino se denomina *myDestinationGallery* y se encuentra en el grupo de recursos *myDestinationRG* de la ubicación *Oeste de EE. UU.* . La versión de la imagen es *1.0.0* y vamos a crear una réplica en la región *Centro-sur de EE. UU.* y dos réplicas en la región *Oeste de EE. UU.* . 


```azurepowershell-interactive
$region1 = @{Name='South Central US';ReplicaCount=1}
$region2 = @{Name='West US';ReplicaCount=2}
$targetRegions = @($region1,$region2)

$job = $imageVersion = New-AzGalleryImageVersion `
   -GalleryImageDefinitionName $destinationImgDef.Name`
   -GalleryImageVersionName '1.0.0' `
   -GalleryName myDestinationGallery `
   -ResourceGroupName myDestinationRG `
   -Location WestUS `
   -TargetRegion $targetRegions  `
   -Source $sourceImgVer.Id.ToString() `
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
> También puede almacenar la imagen en almacenamiento Premium al agregar `-StorageAccountType Premium_LRS`, o en el [almacenamiento con redundancia de zona](../storage/common/storage-redundancy.md) al agregar `-StorageAccountType Standard_ZRS` durante la creación de la versión de la imagen.
>


## <a name="next-steps"></a>Pasos siguientes

Cree una máquina virtual a partir de una versión de imagen [generalizada](vm-generalized-image-version-powershell.md) o [especializada](vm-specialized-image-version-powershell.md).

[Azure Image Builder (versión preliminar)](./linux/image-builder-overview.md) puede ayudar a automatizar la creación de versiones de la imagen, incluso se puede usar para actualizar y [crear una nueva versión de la imagen a partir de una versión de imagen existente](./linux/image-builder-gallery-update-image-version.md). 

Para saber cómo proporcionar información del plan de compra, consulte [Indicación de la información del plan de compra de Azure Marketplace al crear imágenes](marketplace-images.md).
