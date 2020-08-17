---
title: Eliminación de la característica Update Management de Azure Automation
description: En este artículo se indica cómo dejar de usar Update Management y desvincular una cuenta de Automation del área de trabajo de Log Analytics.
services: automation
ms.date: 07/28/2020
ms.topic: conceptual
ms.custom: mvc
ms.openlocfilehash: 4b4946da9f63299c7ba2b383d6c153673595a1ab
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449760"
---
# <a name="remove-update-management-from-automation-account"></a>Eliminación de Update Management de una cuenta de Automation

Después de habilitar la administración de actualizaciones en las máquinas virtuales mediante Update Management de Azure Automation, puede decidir dejar de usarla y quitar la configuración de la cuenta y del área de trabajo de Log Analytics vinculada.  En este artículo se explica cómo quitar por completo Update Management de las máquinas virtuales administradas, la cuenta de Automation y el área de trabajo de Log Analytics.

## <a name="sign-into-the-azure-portal"></a>Inicie sesión en Azure Portal.

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="remove-management-of-vms"></a>Eliminación de la administración de máquinas virtuales

Antes de quitar Update Management, primero debe detener la administración de las máquinas virtuales. Consulte [Eliminación de máquinas virtuales desde Update Management](update-mgmt-remove-vms.md) para anular la inscripción de la característica.

## <a name="remove-updatemanagement-solution"></a>Eliminación de la solución Update Management

Si desea desvincular la cuenta de Automation del área de trabajo, debe seguir estos pasos para quitar completamente Update Management. Va a quitar la solución **Actualizaciones** del área de trabajo.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. En Azure Portal, seleccione **Todos los servicios**. En la lista de recursos, escriba **Log Analytics**. Cuando comience a escribir, la lista filtrará las sugerencias en función de la entrada. Seleccione **Log Analytics**.

3. En la lista de áreas de trabajo de Log Analytics, seleccione la que eligió al habilitar Update Management.

4. A la izquierda, seleccione **Soluciones**.  

5. En la lista de soluciones, seleccione **Updates(nombre de área de trabajo)** . En la página **Introducción** de la solución, seleccione **Eliminar**. Cuando se le solicite confirmación, haga clic en **Sí**.

## <a name="unlink-workspace-from-automation-account"></a>Desvinculación de un área de trabajo de una cuenta de Automation

1. En Azure Portal, seleccione **Cuentas de Automation**.

2. Abra la cuenta de Automation y seleccione **Área de trabajo vinculada** en **Recursos relacionados** a la izquierda.

3. En la página **Desvincular área de trabajo**, seleccione **Desvincular área de trabajo** y responda a los mensajes.

   ![Página Desvincular área de trabajo](media/update-mgmt-remove-feature/automation-unlink-workspace-blade.png)

Mientras intenta desvincular el área de trabajo de Log Analytics, puede realizar el seguimiento del progreso en la opción **Notificaciones** del menú.

También puede desvincular el área de trabajo de Log Analytics de la cuenta de Automation desde el área de trabajo:

1. En Azure Portal, seleccione **Log Analytics**.

2. En el área de trabajo, seleccione **Cuenta de Automation** en **Recursos relacionados**.

3. En la página Cuenta de Automation, seleccione **Desvincular cuenta**.

Mientras intenta desvincular la cuenta de Automation, puede seguir el progreso en **Notificaciones** desde el menú.

## <a name="cleanup-automation-account"></a>Limpieza de la cuenta de Automation

Si Update Management se configuró para admitir versiones anteriores de la supervisión de SQL de Azure, es posible que el programa de instalación de la característica haya creado recursos de Automation que se deberían eliminar. En el caso de Update Management, es posible que desee eliminar los siguientes elementos que ya no son necesarios:

   * Programaciones de actualizaciones: cada una tiene un nombre que coincide con la implementación de actualizaciones que ha creado.
   * Grupos de Hybrid Worker que se han creado para Update Management: cada uno de ellos tiene un nombre similar a *machine1.contoso.com_9ceb8108-26c9-4051-b6b3-227600d715c8*.

## <a name="next-steps"></a>Pasos siguientes

Para volver a habilitar esta característica, consulte [Habilitación de Update Management desde una cuenta de Automation](update-mgmt-enable-automation-account.md), [Habilitación de Update Management examinando Azure Portal](update-mgmt-enable-portal.md), [Habilitación de Update Management desde un runbook](update-mgmt-enable-runbook.md) o [Habilitación de Update Management desde una máquina virtual de Azure](update-mgmt-enable-vm.md).
