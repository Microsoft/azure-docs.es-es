---
title: Quitar máquinas virtuales de Azure Automation Update Management
description: En este artículo se explica cómo quitar máquinas administradas con Update Management.
services: automation
ms.topic: conceptual
ms.date: 07/28/2020
ms.custom: mvc
ms.openlocfilehash: d7f7e4aa8b2c192688020b4449c8750f94af29f6
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449768"
---
# <a name="remove-vms-from-update-management"></a>Eliminación de una máquina virtual desde Update Management

Cuando haya terminado de administrar las actualizaciones en las máquinas virtuales de su entorno, puede dejar de administrar las máquinas virtuales con la característica [Update Management](update-mgmt-overview.md).

## <a name="sign-into-the-azure-portal"></a>Inicie sesión en Azure Portal.

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="to-remove-your-vms"></a>Para quitar las máquinas virtuales

1. En su cuenta de Automation, seleccione **Update Management** en **Update Management**.

2. Use el siguiente comando para identificar el UUID de un equipo que desea quitar de la administración.

    ```azurecli
    az vm show -g MyResourceGroup -n MyVm -d
    ```

3. En el área de trabajo de Log Analytics, debajo de **General**, acceda a las búsquedas guardadas para la configuración de ámbito `MicrosoftDefaultScopeConfig-Updates`.

4. Para la búsqueda guardada `MicrosoftDefaultComputerGroup`, haga clic en la elipse a la derecha y seleccione **Editar**.

5. Quite el UUID de la máquina virtual.

6. Repita los pasos para las demás máquinas virtuales que se van a quitar.

7. Guarde la búsqueda guardada cuando haya terminado de modificarla.

>[!NOTE]
>Las máquinas todavía se muestran después de anular la inscripción, ya que se informa de todas las máquinas que se han evaluado en las últimas 24 horas. Después de desconectar la máquina, debe esperar 24 horas antes de que ya no aparezcan en la lista.

## <a name="next-steps"></a>Pasos siguientes

Para volver a habilitar la administración de la máquina virtual, consulte [Examinen de Azure Portal para habilitar Update Management](update-mgmt-enable-portal.md) o [Habilitación de Update Management desde una máquina virtual de Azure](update-mgmt-enable-vm.md).