---
title: Copia de una versión de una imagen desde otra galería mediante la CLI
description: Copie una versión de una imagen desde otra galería con la CLI de Azure.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.reviewer: akjosh
ms.openlocfilehash: e8aabcd1c68272a78b3c1fe88913c5a62496f681
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88225842"
---
# <a name="copy-an-image-from-another-gallery-using-the-azure-cli"></a>Copia de una imagen de otra galería mediante la CLI de Azure

Si tiene varias galerías en su organización, también puede crear versiones de imágenes a partir de otras versiones existentes que estén almacenadas en otras galerías. Por ejemplo, puede tener una galería de desarrollo y pruebas para crear y probar imágenes nuevas. Cuando estén listas para usarlas en producción, puede copiarlas a una galería de producción con este ejemplo. También puede usar [Azure PowerShell](image-version-another-gallery-powershell.md) para crear una imagen a partir de una imagen de otra galería.



## <a name="before-you-begin"></a>Antes de empezar

Para completar este artículo, debe tener una galería de origen, una definición de imagen y una versión de imagen existentes. Además, debe tener una galería de destino. 

La versión de la imagen de origen se debe replicar en la región en que se encuentra la galería de destino. 

Al trabajar en este artículo, reemplace los nombres de los recursos cuando proceda.



## <a name="get-information-from-the-source-gallery"></a>Obtención de información de la galería de origen

Necesitará información de la definición de imagen de origen para crear una copia de ella en la nueva galería.

Para encontrar datos acerca de la galería de origen, muestre la información de las galerías de imágenes disponibles mediante [az sig list](/cli/azure/sig#az-sig-list).

```azurecli-interactive 
az sig list -o table
```

Enumere las definiciones de imágenes en una galería mediante [az sig image-definition list](/cli/azure/sig/image-definition#az-sig-image-definition-list). En este ejemplo, vamos a buscar definiciones de imágenes en la galería llamada *myGallery* del grupo de recursos *myGalleryRG*.

```azurecli-interactive 
az sig image-definition list \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   -o table
```

Enumere las versiones de una imagen que hay en una galería y use [az sig image-version list](/cli/azure/sig/image-version#az-sig-image-version-list) para buscar la versión de la imagen que desea copiar en la nueva galería. En este ejemplo, vamos a buscar todas las versiones de una imagen que forman parte de la definición de la imagen *myImageDefinition*.

```azurecli-interactive
az sig image-version list \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   -o table
```

Una vez que tenga toda la información que necesita, puede obtener el identificador de la versión de la imagen de origen mediante [az sig image-version show](/cli/azure/sig/image-version#az-sig-image-version-show).

```azurecli-interactive
az sig image-version show \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --query "id" -o tsv
```


## <a name="create-the-image-definition"></a>Creación de una definición de imagen 

Debe crear una definición de imagen que coincida con la de la versión de la imagen de origen. Puede ver toda la información necesaria para volver a crear la definición de imagen en la nueva galería mediante [az sig image-definition show](/cli/azure/sig/image-definition#az-sig-image-definition-show).

```azurecli-interactive
az sig image-definition show \
   --resource-group myGalleryRG \
   --gallery-name myGallery \
   --gallery-image-definition myImageDefinition
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
  "osType": "Linux",
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

Cree una definición de imagen en la galería nueva con la información de la salida anterior.


```azurecli-interactive 
az sig image-definition create \
   --resource-group myNewGalleryRG \
   --gallery-name myNewGallery \
   --gallery-image-definition myImageDefinition \
   --publisher myPublisher \
   --offer myOffer \
   --sku mySKU \
   --os-type Linux \
   --hyper-v-generation V1 \
   --os-state specialized 
```


## <a name="create-the-image-version"></a>Creación de la versión de la imagen

Cree versiones mediante [az image gallery create-image-version](/cli/azure/sig/image-version#az-sig-image-version-create). Deberá pasar el identificador de la imagen administrada que se usará como base para crear la versión de la imagen. Puede usar [az image list](/cli/azure/image?view#az-image-list) para obtener información sobre las imágenes que se encuentran en un grupo de recursos. 

Los caracteres permitidos para la versión de una imagen son números y puntos. Los números deben estar dentro del rango de un entero de 32 bits. Formato: *VersiónPrincipal*.*VersiónSecundaria*.*Revisión*.

En este ejemplo, la versión de la imagen es *1.0.0* y vamos a crear una réplica en la región *Centro-sur de EE. UU.* y otra en la región *Este de EE. UU.* mediante el almacenamiento con redundancia de zona.


```azurecli-interactive 
az sig image-version create \
   --resource-group myNewGalleryRG \
   --gallery-name myNewGallery \
   --gallery-image-definition myImageDefinition \
   --gallery-image-version 1.0.0 \
   --target-regions "southcentralus=1" "eastus=1=standard_zrs" \
   --replica-count 2 \
   --managed-image "/subscriptions/<Subscription ID>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition/versions/1.0.0"
```

> [!NOTE]
> Deberá esperar a que la versión de la imagen termine de compilarse y replicarse por completo antes de poder usar la misma imagen administrada para crear otra versión de la imagen.
>
> También puede almacenar la imagen en almacenamiento Premium agregando `--storage-account-type  premium_lrs`, o en el [almacenamiento con redundancia de zona](../storage/common/storage-redundancy.md) agregando `--storage-account-type  standard_zrs` al crear la versión de la imagen.
>

## <a name="next-steps"></a>Pasos siguientes

Cree una máquina virtual a partir de una versión de imagen [generalizada](vm-generalized-image-version-cli.md) o [especializada](vm-specialized-image-version-cli.md).

Pruebe también [Azure Image Builder (versión preliminar)](./linux/image-builder-overview.md), que puede ayudarle a automatizar la creación de versiones de la imagen; incluso se puede usar para actualizar y [crear una nueva versión de la imagen a partir de una versión de imagen existente](./linux/image-builder-gallery-update-image-version.md). 

Para obtener información sobre cómo proporcionar información del plan de compra, consulte [Indicación de la información del plan de compra de Azure Marketplace al crear imágenes](marketplace-images.md).
