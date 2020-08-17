---
title: Habilitación de Update Management de Azure Automation desde una cuenta de Automation
description: En este artículo se indica cómo habilitar Update Management desde una cuenta de Automation.
services: automation
ms.date: 07/28/2020
ms.topic: conceptual
ms.custom: mvc
ms.openlocfilehash: 930861c61843c5963c83d8fa6dc1efdce20853f4
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449802"
---
# <a name="enable-update-management-from-an-automation-account"></a>Habilitación de Update Management desde una cuenta de Automation

En este artículo se describe cómo puede usar su cuenta de Automation para habilitar la característica [Update Management](update-mgmt-overview.md) para las máquinas virtuales del entorno. Para habilitar máquinas virtuales de Azure a escala, debe habilitar una máquina virtual existente mediante Update Management.

> [!NOTE]
> Al habilitar Update Management, solo en determinadas regiones se puede vincular un área de trabajo de Log Analytics y una cuenta de Automation. Para obtener una lista de los pares de asignación que se admiten, consulte [Asignación de región para la cuenta de Automation y el área de trabajo de Log Analytics](../how-to/region-mappings.md).

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción de Azure. Si aún no tiene ninguna, puede [activar las ventajas de la suscripción a MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o suscribirse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Una [cuenta de Automation](../index.yml) para administrar máquinas.
* Una [máquina virtual](../../virtual-machines/windows/quick-create-portal.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="enable-update-management"></a>Habilitación de la Administración de actualizaciones

1. En su cuenta de Automation, seleccione **Update Management** en **Update Management**.

2. Elija el área de trabajo de Log Analytics y la cuenta de Automation, y seleccione **Habilitar** para habilitar Update Management. La instalación tarda hasta 15 minutos en completarse.

    ![Habilitación de la Administración de actualizaciones](media/update-mgmt-enable-automation-account/onboardsolutions2.png)

## <a name="enable-azure-vms"></a>Habilitar máquinas virtuales de Azure

1. En su cuenta de Automation, seleccione **Update Management** en **Update Management**.

2. Seleccione **+ Agregar máquinas virtuales de Azure** y seleccione una o varias máquinas virtuales en la lista. Las máquinas virtuales que no se pueden habilitar aparecen atenuadas y no se pueden seleccionar. Las VM de Azure pueden existir en cualquier región, con independencia de la ubicación de la cuenta de Automation.

3. Seleccione **Habilitar** para agregar las máquinas virtuales seleccionadas a la búsqueda guardada de los grupos de equipos para la característica.

    ![Habilitar máquinas virtuales de Azure](media/update-mgmt-enable-automation-account/enable-azure-vms.png)

## <a name="enable-non-azure-vms"></a>Habilitación de máquinas virtuales que no son de Azure

Las máquinas que no estén en Azure se deben agregar manualmente.

1. En su cuenta de Automation, seleccione **Update Management** en **Update Management**.

2. Seleccione **Agregar máquina que no es de Azure**. Esta acción abre una nueva ventana del explorador con las [instrucciones para instalar y configurar el agente de Log Analytics para Windows](../../azure-monitor/platform/log-analytics-agent.md) y que la máquina pueda empezar a enviar notificaciones a Update Management. Si va a habilitar una máquina que Operations Manager administre, no es necesario un nuevo agente. La información del área de trabajo se agrega a la configuración de los agentes.

## <a name="enable-machines-in-the-workspace"></a>Habilitación de máquinas en el área de trabajo

Las máquinas instaladas manualmente o las máquinas que ya envían notificaciones al área de trabajo deben agregarse a Azure Automation para poder habilitar Update Management.

1. En su cuenta de Automation, seleccione **Update Management** en **Update Management**.

2. Seleccione **Administrar máquinas**. Es posible que, si previamente ha seleccionado la opción **Habilitar en todas las máquinas disponibles y futuras**, el botón **Administrar máquinas** esté atenuado.

    ![Búsquedas guardadas](media/update-mgmt-enable-automation-account/managemachines.png)

3. Para habilitar Update Management en todas las máquinas disponibles, seleccione **Habilitar en todas las máquinas disponibles** en la página Administrar máquinas. Esta operación deshabilita el control que permite agregar máquinas una a una. Esta tarea agrega todos los nombres de las máquinas que informan al área de trabajo en la consulta de la búsqueda guardada de grupos de equipos. Cuando se selecciona, esta acción deshabilita el botón **Administrar máquinas**.

4. Para habilitar la característica en todas las máquinas disponibles y futuras, seleccione **Habilitar en todas las máquinas disponibles y futuras**. Esta opción elimina las búsquedas guardadas y las configuraciones de ámbito del área de trabajo y abre la característica para todas las máquinas de Azure y que no son de Azure que envían notificaciones al área de trabajo. Cuando está seleccionada, esta acción deshabilita permanentemente el botón **Administrar máquinas**, ya que no queda ninguna configuración de ámbito.

5. Si es necesario, puede agregar de nuevo las configuraciones de ámbito si vuelve a agregar las búsquedas guardadas iniciales. Para obtener más información, consulte el tema sobre la [limitación del ámbito de implementación de Update Management](update-mgmt-scope-configuration.md).

6. Para habilitar la característica para una o varias máquinas, seleccione **Habilitar en las máquinas seleccionadas** y seleccione **Agregar** junto a cada máquina. Esta tarea agrega los nombres de máquina seleccionados a la consulta de la búsqueda guardada de grupos de equipos para la característica.

## <a name="next-steps"></a>Pasos siguientes

* Para usar Update Management para máquinas virtuales, consulte [Administración de actualizaciones y revisiones para las máquinas virtuales](update-mgmt-manage-updates-for-vm.md).

* Para solucionar los errores de Update Management generales, consulte [Solución de problemas de Update Management](../troubleshoot/update-management.md).