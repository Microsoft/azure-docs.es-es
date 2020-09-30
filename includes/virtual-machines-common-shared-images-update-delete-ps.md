---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-machines
author: cynthn
ms.service: virtual-machines
ms.topic: include
ms.date: 04/25/2019
ms.author: cynthn
ms.custom: include file
ms.openlocfilehash: 6e0612a017650f0c6e4c9f63d9a5fd097b0b92c4
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89304089"
---
## <a name="update-resources"></a>Actualización de recursos

Existen algunas limitaciones en lo que se puede actualizar. Se pueden actualizar los siguientes elementos: 

Galería de imágenes compartidas:
- Descripción

Definición de la imagen:
- vCPU recomendadas:
- Memoria recomendada
- Descripción
- Fecha final del ciclo de vida

Versión de la imagen:
- Recuento de réplicas regionales
- Regiones de destino
- Exclusión de la versión más reciente
- Fecha final del ciclo de vida

Si planea agregar regiones de réplica, no elimine la imagen administrada de origen. La imagen administrada de origen es necesaria para replicar la versión de la imagen a regiones adicionales. 

Para actualizar la descripción de una galería, use [Update-AzGallery](https://docs.microsoft.com/powershell/module/az.compute/update-azgallery).

```azurepowershell-interactive
Update-AzGallery `
   -Name $gallery.Name ` 
   -ResourceGroupName $resourceGroup.Name
```

En este ejemplo se muestra cómo usar [Update-AzGalleryImageDefinition](https://docs.microsoft.com/powershell/module/az.compute/update-azgalleryimagedefinition) para actualizar la fecha de vencimiento de nuestra definición de imagen.

```azurepowershell-interactive
Update-AzGalleryImageDefinition `
   -GalleryName $gallery.Name `
   -Name $galleryImage.Name `
   -ResourceGroupName $resourceGroup.Name `
   -EndOfLifeDate 01/01/2030
```

En este ejemplo se muestra cómo usar [Update-AzGalleryImageVersion](https://docs.microsoft.com/powershell/module/az.compute/update-azgalleryimageversion) para impedir que esta versión de imagen se use como la imagen *más reciente*.

```azurepowershell-interactive
Update-AzGalleryImageVersion `
   -GalleryImageDefinitionName $galleryImage.Name `
   -GalleryName $gallery.Name `
   -Name $galleryVersion.Name `
   -ResourceGroupName $resourceGroup.Name `
   -PublishingProfileExcludeFromLatest
```

En este ejemplo se muestra cómo usar [Update-AzGalleryImageVersion](https://docs.microsoft.com/powershell/module/az.compute/update-azgalleryimageversion) para que esta versión de imagen se considere como imagen *más reciente*.

```azurepowershell-interactive
Update-AzGalleryImageVersion `
   -GalleryImageDefinitionName $galleryImage.Name `
   -GalleryName $gallery.Name `
   -Name $galleryVersion.Name `
   -ResourceGroupName $resourceGroup.Name `
   -PublishingProfileExcludeFromLatest:$false
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Al eliminar los recursos, debe comenzar por el último elemento de los recursos anidados: la versión de imagen. Cuando se hayan eliminado las versiones, puede eliminar la definición de imagen. No se puede eliminar la galería hasta que se hayan eliminado todos los recursos por debajo de ella.

```azurepowershell-interactive
$resourceGroup = "myResourceGroup"
$gallery = "myGallery"
$imageDefinition = "myImageDefinition"
$imageVersion = "myImageVersion"

Remove-AzGalleryImageVersion `
   -GalleryImageDefinitionName $imageDefinition `
   -GalleryName $gallery `
   -Name $imageVersion `
   -ResourceGroupName $resourceGroup

Remove-AzGalleryImageDefinition `
   -ResourceGroupName $resourceGroup `
   -GalleryName $gallery `
   -GalleryImageDefinitionName $imageDefinition

Remove-AzGallery `
   -Name $gallery `
   -ResourceGroupName $resourceGroup

Remove-AzResourceGroup -Name $resourceGroup
```

