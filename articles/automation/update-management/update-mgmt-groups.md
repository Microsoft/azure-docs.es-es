---
title: Uso de grupos dinámicos con Update Management de Azure Automation
description: En este artículo se explica cómo usar los grupos dinámicos con Update Management de Azure Automation.
services: automation
ms.subservice: update-management
ms.date: 07/28/2020
ms.topic: conceptual
ms.openlocfilehash: 20e6d26808964c8e697c694bd796af2851e7ca48
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449784"
---
# <a name="use-dynamic-groups-with-update-management"></a>Uso de grupos dinámicos con Update Management

Update Management permite tener como destino un grupo dinámico de VM de Azure (o de VM que no sean de Azure) para implementaciones de actualizaciones. El uso de un grupo dinámico evita que tenga que editar la implementación para actualizar las máquinas.

> [!NOTE]
> Los grupos dinámicos no funcionan con máquinas virtuales clásicas.

Puede definir grupos dinámicos para máquinas, sean o no de Azure, desde **Update Management** en Azure Portal. Consulte [Administrar las actualizaciones de las VM](update-mgmt-manage-updates-for-vm.md).

Un grupo dinámico se define mediante una consulta que Azure Automation evalúa en el momento de la implementación. Aunque una consulta de grupo dinámico recupere un gran número de máquinas, Azure Automation solo puede procesar un máximo de 1000 máquinas a la vez. Vea [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md#update-management).

> [!NOTE]
> Si espera actualizar más de 1000 máquinas, es recomendable que divida las actualizaciones entre varias programaciones de actualización. 

## <a name="define-dynamic-groups-for-azure-machines"></a>Definición de grupos dinámicos para máquinas de Azure

Al definir una consulta de grupo dinámico para las máquinas de Azure, puede usar los siguientes elementos para rellenar el grupo dinámico:

* Subscription
* Grupos de recursos
* Ubicaciones
* Etiquetas

![Selección de grupos](./media/update-mgmt-groups/select-groups.png)

Para obtener una vista previa de los resultados de un grupo dinámico, haga clic en **Vista previa**. La versión preliminar muestra la pertenencia al grupo en el momento actual. En el ejemplo, buscamos máquinas que tengan la etiqueta `Role` para el grupo **BackendServer**. Si se ha agregado esta etiqueta a varias máquinas, se agregan a las implementaciones futuras en ese grupo.

![vista previa de grupos](./media/update-mgmt-groups/preview-groups.png)

## <a name="define-dynamic-groups-for-non-azure-machines"></a>Definición de grupos dinámicos para máquinas que no son de Azure

Un grupo dinámico para máquinas que no son de Azure usa búsquedas guardadas, que también se denominan grupos de equipos. Para aprender a crear una búsqueda guardada, consulte [Creación de un grupo de equipos](../../azure-monitor/platform/computer-groups.md#creating-a-computer-group). Una vez creada la búsqueda guardada, puede seleccionarla en la lista de búsquedas guardadas en **Update Management** en Azure Portal. Haga clic en **Vista previa** para obtener una vista previa de los equipos de la búsqueda guardada.

![Selección de grupos](./media/update-mgmt-groups/select-groups-2.png)

## <a name="next-steps"></a>Pasos siguientes

Puede [consultar los registros de Azure Monitor](update-mgmt-query-logs.md) para analizar las evaluaciones de las actualizaciones, las implementaciones y otras tareas de administración relacionadas. Incluye consultas predefinidas para ayudarle a empezar.
