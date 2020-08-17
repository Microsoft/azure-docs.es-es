---
title: Procedimiento para etiquetar una máquina virtual Linux en Azure
description: Infórmese sobre el etiquetado de una máquina virtual Linux de Azure creada con el modelo de implementación de Resource Manager.
services: virtual-machines-linux
documentationcenter: ''
author: mmccrory
manager: gwallace
tags: azure-resource-manager
ms.assetid: ca0e17e5-d78e-42e6-9dad-c1e8f1c58027
ms.service: virtual-machines-linux
ms.topic: how-to
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 02/28/2017
ms.author: memccror
ms.custom: devx-track-azurecli
ms.openlocfilehash: a830f6eb604bb1173a7a552ad5f81c936f02d5fd
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87500528"
---
# <a name="how-to-tag-a-linux-virtual-machine-in-azure"></a>Etiquetado de una máquina virtual Linux en Azure
En este artículo se describen diferentes maneras de etiquetar una máquina virtual en Azure el modelo de implementación de Resource Manager. Las etiquetas son pares clave-valor definidos por el usuario que se pueden colocar directamente en un recurso o un grupo de recursos. Actualmente, Azure admite un máximo de 50 etiquetas por recurso y grupo de recursos. Las etiquetas se pueden colocar en un recurso en el momento de su creación, o bien se pueden agregar a un recurso existente. Tenga en cuenta que las etiquetas solo son compatibles con los recursos creados mediante el modelo de implementación de Resource Manager.

[!INCLUDE [virtual-machines-common-tag](../../../includes/virtual-machines-common-tag.md)]

## <a name="tagging-with-azure-cli"></a>Etiquetado con la CLI de Azure

Para empezar, necesita tener instalada la última versión de la [CLI de Azure](/cli/azure/install-azure-cli) e iniciar sesión en una cuenta de Azure con [az login](/cli/azure/reference-index#az-login).

Con este comando es posible ver todas las propiedades de una máquina virtual dada, incluidas las etiquetas:

```azurecli
az vm show --resource-group MyResourceGroup --name MyTestVM
```

Para agregar una nueva etiqueta de máquina virtual a través de la CLI de Azure, puede usar el comando `azure vm update` junto con el parámetro de etiqueta **--set**:

```azurecli
az vm update \
    --resource-group MyResourceGroup \
    --name MyTestVM \
    --set tags.myNewTagName1=myNewTagValue1 tags.myNewTagName2=myNewTagValue2
```

Para quitar las etiquetas, puede usar el parámetro **--remove** en el comando `azure vm update`.

```azurecli
az vm update --resource-group MyResourceGroup --name MyTestVM --remove tags.myNewTagName1
```

Ahora que hemos aplicado etiquetas a nuestros recursos a través de PowerShell, la CLI de Azure y el portal, analicemos los datos de uso para ver las etiquetas del portal de facturación.

[!INCLUDE [virtual-machines-common-tag-usage](../../../includes/virtual-machines-common-tag-usage.md)]

## <a name="next-steps"></a>Pasos siguientes
* Para más información sobre el etiquetado de los recursos de Azure, consulte [Información general de Azure Resource Manager][Azure Resource Manager Overview] y [Uso de etiquetas para organizar los recursos de Azure][Using Tags to organize your Azure Resources].
* Para ver cómo las etiquetas pueden ayudarle a administrar el uso de los recursos de Azure, consulte [Comprender la factura de Microsoft Azure][Understanding your Azure Bill] y [Obtención de información sobre el consumo de recursos de Microsoft Azure][Gain insights into your Microsoft Azure resource consumption].

[Azure CLI environment]: ../../azure-resource-manager/management/manage-resources-cli.md
[Azure Resource Manager Overview]: ../../azure-resource-manager/management/overview.md
[Using Tags to organize your Azure Resources]: ../../azure-resource-manager/management/tag-resources.md
[Understanding your Azure Bill]:../../cost-management-billing/understand/review-individual-bill.md
[Gain insights into your Microsoft Azure resource consumption]:../../cost-management-billing/manage/usage-rate-card-overview.md
