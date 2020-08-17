---
title: Habilitación de Update Management de Azure Automation desde una máquina virtual de Azure
description: En este artículo se indica cómo habilitar Update Management desde una máquina virtual de Azure.
services: automation
ms.date: 07/28/2020
ms.topic: conceptual
ms.custom: mvc
ms.openlocfilehash: 27832190125840e367edbfb2db8e4134f98b192d
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449786"
---
# <a name="enable-update-management-from-an-azure-vm"></a>Habilitación de Update Management desde una maquina virtual de Azure

En este artículo se describe cómo puede usar una máquina virtual de Azure para habilitar la característica [Update Management](update-mgmt-overview.md) en otras máquinas. Para habilitar máquinas virtuales de Azure a escala, debe habilitar una máquina virtual existente mediante Update Management.

> [!NOTE]
> Al habilitar Update Management, solo en determinadas regiones se puede vincular un área de trabajo de Log Analytics y una cuenta de Automation. Para obtener una lista de los pares de asignación que se admiten, consulte [Asignación de región para la cuenta de Automation y el área de trabajo de Log Analytics](../how-to/region-mappings.md).

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción de Azure. Si aún no tiene ninguna, puede [activar las ventajas de la suscripción a MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o suscribirse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Una [cuenta de Automation](../index.yml) para administrar máquinas.
* Una [máquina virtual](../../virtual-machines/windows/quick-create-portal.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="enable-the-feature-for-deployment"></a>Habilitación de la característica de implementación

1. En [Azure Portal](https://portal.azure.com), seleccione **Máquinas virtuales** o busque y seleccione **Máquinas virtuales** en la página principal.

2. Seleccione la máquina virtual para la que desea habilitar Update Management. Las máquina virtuales pueden existir en cualquier región, independientemente de la ubicación de la cuenta de Automation. Los

3. En la página de la máquina virtual, en **Operaciones**, seleccione **Update Management**.

4. Debe tener el permiso `Microsoft.OperationalInsights/workspaces/read` para determinar si la máquina virtual está habilitada para un área de trabajo. Para obtener información sobre los permisos adicionales requeridos, vea los [permisos necesarios para habilitar máquinas](../automation-role-based-access-control.md#feature-setup-permissions). Para obtener información sobre cómo habilitar varias máquinas a la vez, vea [Habilitación de Update Management desde una cuenta de Automation](update-mgmt-enable-automation-account.md).

5. Para habilitar Update Management, elija el área de trabajo de Log Analytics y la cuenta de Automation y haga clic en **Habilitar**. Una vez que haya habilitado Update Management, puede que tarde unos 15 minutos en poder ver la evaluación de la actualización desde la máquina virtual.

    ![Habilitación de la Administración de actualizaciones](media/update-mgmt-enable-vm/manageupdates-update-enable.png)

## <a name="next-steps"></a>Pasos siguientes

* Para usar Update Management para máquinas virtuales, consulte [Administración de actualizaciones y revisiones para las máquinas virtuales de Azure](update-mgmt-manage-updates-for-vm.md).

* Para solucionar los errores de Update Management generales, consulte [Solución de problemas de Update Management](../troubleshoot/update-management.md).
