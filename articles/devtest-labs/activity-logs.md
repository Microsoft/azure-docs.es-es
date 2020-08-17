---
title: Registros de actividad en Azure DevTest Labs | Microsoft Docs
description: En este artículo se indican los pasos para ver los registros de actividad de Azure DevTest Labs.
ms.topic: how-to
ms.date: 07/10/2020
ms.openlocfilehash: 24b5eb25f1c8c9044ad6e77545381a43a11714f9
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095145"
---
# <a name="view-activity-logs-for-labs-in-azure-devtest-labs"></a>Visualización de registros de actividad para laboratorios en Azure DevTest Labs 
Después de crear uno o varios laboratorios, es probable que desee supervisar cómo y cuándo se accede a los laboratorios, cómo y cuándo se modifican y administran, y quién lo hace. Azure DevTest Labs usa Azure Monitor, en concreto **registros de actividad**, para proporcionar información sobre estas operaciones en los laboratorios. 

En este artículo se explica cómo ver los registros de actividad de un laboratorio en Azure DevTest Labs.

## <a name="view-activity-log-for-a-lab"></a>Visualización de un registro de actividad de un laboratorio

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Seleccione **Todos los servicios** y, luego, **DevTest Labs** en la sección **DevOps**. Si selecciona * (asterisco) junto a **DevTest Labs** en la sección **DevOps**. Esta acción agrega **DevTest Labs** al menú de navegación izquierdo para facilitarle el acceso la próxima vez. A continuación, puede seleccionar **DevTest Labs** en el menú de navegación izquierdo.

    ![Todos los servicios. Selección de DevTest Labs](./media/devtest-lab-create-lab/all-services-select.png)
1. En la lista de laboratorios, seleccione el suyo.
1. En la página principal del laboratorio, seleccione **Configuración y directivas** en el menú de la izquierda. 

    :::image type="content" source="./media/activity-logs/configuration-policies-link.png" alt-text="Seleccione Configuración y directivas en el menú de la izquierda":::
1. En la página **Configuración y directivas**, seleccione **Registro de actividad** en el menú debajo de **Administrar**. Debería ver las entradas de las operaciones realizadas en el laboratorio. 

    :::image type="content" source="./media/activity-logs/activity-log.png" alt-text="Registro de actividad":::    
1. Seleccione un evento para ver detalles sobre él. En la página **Resumen**, verá información, como el nombre de la operación, la marca de tiempo y quién realizó la operación. 
    
    :::image type="content" source="./media/activity-logs/stop-vm-event.png" alt-text="Evento de detención de VM: Resumen":::        
1. Cambie a la pestaña **JSON** para ver más detalles. En el ejemplo siguiente, puede ver el nombre de la VM y la operación realizada en la VM (detenida).

    :::image type="content" source="./media/activity-logs/stop-vm-event-json.png" alt-text="Evento de detención de VM: JSON":::           
1. Cambie a la pestaña **Historial de cambios (vista previa)** para ver el historial de cambios. En el ejemplo siguiente, verá el cambio realizado en la VM. 

    :::image type="content" source="./media/activity-logs/change-history.png" alt-text="Evento de detención de VM: Historial de cambios":::             
1. Seleccione el cambio en la lista del historial de cambios para ver más detalles sobre el cambio. 

    :::image type="content" source="./media/activity-logs/change-details.png" alt-text="Evento de detención de VM: Detalles del cambio":::             

Para obtener más información sobre los registros de actividad, consulte [Registro de actividad de Azure](../azure-monitor/platform/activity-log.md).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre cómo configurar **alertas** en los registros de actividad, consulte [Creación de alertas](create-alerts.md).
- Para obtener más información sobre los registros de actividad, consulte [Registro de actividad de Azure](../azure-monitor/platform/activity-log.md).

