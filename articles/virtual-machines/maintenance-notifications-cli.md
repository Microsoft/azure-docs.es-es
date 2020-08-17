---
title: Obtención de notificaciones de mantenimiento con la CLI
description: Use la CLI de Azure para ver las notificaciones de mantenimiento de máquinas virtuales que se ejecutan en Azure e iniciar el mantenimiento de autoservicio.
author: shants123
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 11/19/2019
ms.author: shants
ms.openlocfilehash: 633708219adaba2fb4c4889754b2112fbf3c4180
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87069356"
---
# <a name="handling-planned-maintenance-notifications-using-the-azure-cli"></a>Control de las notificaciones de mantenimiento planeado con la CLI de Azure

**Este artículo es válido para máquinas virtuales que se ejecutan tanto en Linux como en Windows.**

La CLI se puede usar para ver cuándo está programado el [mantenimiento](maintenance-notifications.md) de las máquinas virtuales. La información de mantenimiento planeado está disponible en [azure vm get-instance-view](/cli/azure/vm?view=azure-cli-latest#az-vm-get-instance-view).
 
La información de mantenimiento se devuelve solo si hay mantenimiento planeado. 

```azurecli-interactive
az vm get-instance-view -n myVM -g myResourceGroup --query instanceView.maintenanceRedeployStatus
```

## <a name="start-maintenance"></a>Iniciar mantenimiento

La siguiente llamada iniciará el mantenimiento en una máquina virtual si `IsCustomerInitiatedMaintenanceAllowed` está establecido en true.

```azurecli-interactive
az vm perform-maintenance -g myResourceGroup -n myVM 
```

## <a name="classic-deployments"></a>Implementaciones clásicas

[!INCLUDE [classic-vm-deprecation](../../includes/classic-vm-deprecation.md)]

Si todavía tiene máquinas virtuales heredadas que se implementaron según el modelo de implementación clásica, puede usar la CLI de Azure clásica para consultar las máquinas virtuales e iniciar el mantenimiento.

Asegúrese de que se encuentra en el modo correcto para trabajar con máquinas virtuales clásicas. Para ello, escriba lo siguiente:

```
azure config mode asm
```

Para obtener el estado de mantenimiento de una máquina virtual denominada *myVM*, escriba lo siguiente:

```
azure vm show myVM 
``` 

Para iniciar el mantenimiento en una máquina virtual clásica denominada *myVM* en el servicio *myService* y la implementación *myDeployment*, escriba lo siguiente:

```
azure compute virtual-machine initiate-maintenance --service-name myService --name myDeployment --virtual-machine-name myVM
```

## <a name="next-steps"></a>Pasos siguientes

El mantenimiento planeado también se puede controlar con [Azure PowerShell](maintenance-notifications-powershell.md) o con [Portal](maintenance-notifications-portal.md).
