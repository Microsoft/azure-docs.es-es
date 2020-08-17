---
title: Creación de un conjunto de escalado a partir de una versión de imagen especializada mediante la CLI de Azure
description: Cree un conjunto de escalado con una versión de imagen especializada en Shared Image Gallery mediante la CLI de Azure.
author: cynthn
ms.service: virtual-machine-scale-sets
ms.subservice: imaging
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 05/01/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.custom: devx-track-azurecli
ms.openlocfilehash: e58cef5ae1fc727235539a91e80e424e6ed3be34
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87496379"
---
# <a name="create-a-scale-set-using-a-specialized-image-version-with-the-azure-cli"></a>Creación de un conjunto de escalado mediante una versión de imagen especializada con la CLI de Azure

Cree un conjunto de escalado a partir de una [versión de imagen especializada](../virtual-machines/linux/shared-image-galleries.md#generalized-and-specialized-images) que esté almacenada en una instancia de Shared Image Gallery. Si quiere crear un conjunto de escalado con una versión de imagen generalizada, consulte [Creación de u conjunto de escalado a partir de una imagen generalizada](instance-generalized-image-version-cli.md).

Si decide instalar y usar la CLI de forma local, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.4.0, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

Reemplace los nombres de los recursos según sea necesario en este ejemplo. 

Enumere las definiciones de imagen en una galería mediante [az sig image-definition list](/cli/azure/sig/image-definition#az-sig-image-definition-list) para ver el nombre y el id. de las definiciones.

```azurecli-interactive 
resourceGroup=myGalleryRG
gallery=myGallery
az sig image-definition list \
   --resource-group $resourceGroup \
   --gallery-name $gallery \
   --query "[].[name, id]" \
   --output tsv
```

Cree un conjunto de escalado mediante [`az vmss create`](/cli/azure/vmss#az-vmss-create) con el parámetro `--specialized` para indicar que se trata de una imagen especializada.

Use el id. de definición de imagen para el parámetro `--image`; así, se crearán las instancias del conjunto de escalado a partir de la versión más reciente de la imagen que está disponible. También puede crear las instancias del conjunto de escalado a partir de una versión específica si proporciona el identificador de la versión de la imagen en el parámetro `--image`. Tenga en cuenta que el uso de una versión de imagen específica significa que la automatización podría producir un error si dicha versión no está disponible porque se eliminó o se quitó de la región. Se recomienda usar el identificador de definición de la imagen para crear la nueva máquina virtual, a menos que se requiera una versión de imagen específica.

En este ejemplo, se crearán instancias a partir de la versión más reciente de la imagen *myImageDefinition*.

```azurecli
az group create --name myResourceGroup --location eastus
az vmss create \
   --resource-group myResourceGroup \
   --name myScaleSet \
   --image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition" \
   --Specialized
```


## <a name="next-steps"></a>Pasos siguientes
[Azure Image Builder (versión preliminar)](../virtual-machines/linux/image-builder-overview.md) puede ayudar a automatizar la creación de versiones de la imagen, incluso se puede usar para actualizar y [crear una nueva versión de la imagen a partir de una versión de imagen existente](../virtual-machines/linux/image-builder-gallery-update-image-version.md). 

Puede crear también recursos de galería de imágenes compartidas con plantillas. Hay varias plantillas de Inicio rápido de Azure disponibles: 

- [Creación de una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/101-sig-create/)
- [Creación de una definición de imagen en una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/101-sig-image-definition-create/)
- [Creación de una versión de imagen en una galería de imágenes compartidas](https://azure.microsoft.com/resources/templates/101-sig-image-version-create/)
