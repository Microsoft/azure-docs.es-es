---
title: Administración de dominios de error en conjuntos de escalado de máquinas virtuales de Azure
description: Obtenga información sobre cómo elegir el número correcto de dominios de error al crear un conjunto de escalado de máquinas virtuales.
author: mimckitt
ms.author: mimckitt
ms.topic: conceptual
ms.service: virtual-machine-scale-sets
ms.subservice: availability
ms.date: 12/18/2018
ms.reviewer: jushiman
ms.custom: mimckitt
ms.openlocfilehash: 32efde2cb2497fb8aab415d09a1063ff07c6b0f0
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87837200"
---
# <a name="choosing-the-right-number-of-fault-domains-for-virtual-machine-scale-set"></a>Elección del número correcto de dominios de error para el conjunto de escalado de máquinas virtuales
Los conjuntos de escalado de máquinas virtuales se crean con cinco dominios de error de forma predeterminada en las regiones de Azure sin zonas. Para las regiones que admiten la implementación con zonas de conjuntos de escalado de máquinas virtuales y con esta opción seleccionada, el valor predeterminado del número de dominios de error es 1 para cada una de las zonas. FD=1 en este caso implica que las instancias de VM que pertenecen al conjunto de escalado se distribuirán entre varios bastidores en función del mejor esfuerzo.

También puede considerar la alineación del número de dominios de error del conjunto de escalado con el número de dominios de error de Managed Disks. Esta alineación puede ayudar a evitar la pérdida de cuórum si todo un dominio de error de Managed Disks deja de funcionar. El recuento de FD puede establecerse como menor o igual al número de dominios de error de Managed Disks disponibles en cada una de las regiones. Consulte este [documento](../virtual-machines/windows/manage-availability.md) para obtener información sobre el número de dominios de error de Managed Disks por región.

## <a name="rest-api"></a>API DE REST
Puede establecer la propiedad `properties.platformFaultDomainCount` en 1, 2 o 3 (el valor predeterminado es 3 si no se especifica). Consulte la documentación de la API REST [aquí](/rest/api/compute/virtualmachinescalesets/createorupdate).

## <a name="azure-cli"></a>Azure CLI
Puede establecer el parámetro `--platform-fault-domain-count` en 1, 2 o 3 (el valor predeterminado es 3 si no se especifica). Consulte la documentación de la CLI de Azure [aquí](/cli/azure/vmss?view=azure-cli-latest#az-vmss-create).

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --admin-username azureuser \
  --platform-fault-domain-count 3\
  --generate-ssh-keys
```

Se tardan unos minutos en crear y configurar todos los recursos de conjunto de escalado y máquinas virtuales.

## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre [características de disponibilidad y redundancia](../virtual-machines/availability.md) para entornos de Azure.