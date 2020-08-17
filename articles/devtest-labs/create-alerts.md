---
title: Creación de alertas del registro de actividad para laboratorios en Azure DevTest Labs
description: En este artículo se indican los pasos para crear alertas del registro de actividad para laboratorios en Azure DevTest Labs.
ms.topic: how-to
ms.date: 07/10/2020
ms.openlocfilehash: 6c859e70ab8a101c533e16f9cb118e26727baa52
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095342"
---
# <a name="create-activity-log-alerts-for-labs-in-azure-devtest-labs"></a>Creación de alertas del registro de actividad para laboratorios en Azure DevTest Labs
En este artículo se explica cómo crear alertas del registro de actividad para laboratorios en Azure DevTest Labs (por ejemplo, cuando se crea o elimina una VM).

## <a name="create-alerts"></a>Creación de alertas
En este ejemplo, creará una alerta para todas las operaciones administrativas de un laboratorio con una acción que envía un correo electrónico a los propietarios de la suscripción. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En la barra de búsqueda de Azure Portal, escriba **Supervisión** y, a continuación, seleccione **Supervisión** en la lista de resultados. 

    :::image type="content" source="./media/activity-logs/search-monitor.png" alt-text="Búsqueda de Supervisión":::        
1. Seleccione **Alertas** en el menú de la izquierda, y, a continuación, seleccione **Nueva regla de alertas** en la barra de herramientas. 

    :::image type="content" source="./media/activity-logs/alerts-page.png" alt-text="Página Alertas":::    
1. En la página **Crear regla de alerta**, haga clic en **Seleccionar recurso**. 

    :::image type="content" source="./media/activity-logs/select-resource-link.png" alt-text="Selección del recurso para la alerta":::        
1. Seleccione **DevTest Labs** para **Filtrar por tipo de recurso**, seleccione el laboratorio en la lista y, a continuación, seleccione **Listo**.

    :::image type="content" source="./media/activity-logs/select-lab-resource.png" alt-text="Selección del laboratorio como recurso":::
1. Nuevamente en la página **Crear regla de alerta**, haga clic en **Seleccionar condición**. 

    :::image type="content" source="./media/activity-logs/select-condition-link.png" alt-text="Vínculo Seleccionar condición":::    
1. En la página **Configurar lógica de señal**, seleccione una señal admitida por DevTest Labs. 

    :::image type="content" source="./media/activity-logs/select-signal.png" alt-text="Selección de una señal":::
1. Filtre por **nivel de evento** (Detallado, Informativo, Advertencia, Error, Crítico, Todo), **estado** (Con error, Iniciado, Correcto) y **quién inició** el evento. 
1. Seleccione **Listo** para completar la configuración de la condición. 

    :::image type="content" source="./media/activity-logs/configure-signal-logic-done.png" alt-text="Configurar lógica de señal: listo":::
1. Ha especificado el ámbito (laboratorio) y la condición de la alerta. Ahora, debe especificar un grupo de acciones que se ejecutarán cuando se cumpla la condición. Nuevamente en la página **Crear regla de alertas**, elija **Seleccionar grupo de acciones**. 

    :::image type="content" source="./media/activity-logs/select-action-group-link.png" alt-text="Vínculo Seleccionar grupo de acciones":::
1. Seleccione el vínculo **Crear grupo de acciones** en la barra de herramientas. 

    :::image type="content" source="./media/activity-logs/create-action-group-link.png" alt-text="Vínculo Crear grupo de acciones":::
1. En la página **Agregar grupo de acciones**, siga estos pasos:
    1. Escriba un **nombre** para el grupo de acciones.
    1. Escriba un **nombre corto** para el grupo de acciones. 
    1. Seleccione el **grupo de recursos** donde desea que se cree la alerta. 
    1. Escriba un **nombre para la acción**. 
    1. Seleccione el **tipo de acción** (en este ejemplo, **Rol de Azure Resource Manager de correo electrónico**). 

        :::image type="content" source="./media/activity-logs/add-action-group.png" alt-text="Página Agregar grupo de acciones":::
    1. En la página **Rol de Azure Resource Manager de correo electrónico**, seleccione el rol. En este ejemplo, es **Propietario**. Después, seleccione **Aceptar**. 

        :::image type="content" source="./media/activity-logs/select-role.png" alt-text="Selección del rol":::            
    1. Seleccione **Aceptar** en la página **Agregar grupo de acciones**. 
1. Ahora, en la página **Crear regla de alerta**, escriba un nombre para la regla de alerta y, a continuación, seleccione **Aceptar**. 

    :::image type="content" source="./media/activity-logs/create-alert-rule-done.png" alt-text="Crear regla de alerta: listo":::

## <a name="view-alerts"></a>Visualización de alertas 
1. Verá alertas en las **Alertas** para todas las operaciones administrativas (en este ejemplo). Las alertas pueden tardar unos minutos en aparecer. 

    :::image type="content" source="./media/activity-logs/alerts.png" alt-text="Alertas":::
1. Si selecciona el número en una columna (por ejemplo: **Total de alertas**), verá las alertas que se generaron. 

    :::image type="content" source="./media/activity-logs/all-alerts.png" alt-text="Todas las alertas":::
1. Si selecciona una alerta, verá detalles sobre ella. 

    :::image type="content" source="./media/activity-logs/alert-details.png" alt-text="Detalles de alerta":::
1. En este ejemplo, también recibirá un correo electrónico con contenido, tal como se muestra en el ejemplo siguiente: 

    :::image type="content" source="./media/activity-logs/alert-email.png" alt-text="Correo electrónico de alerta":::

## <a name="next-steps"></a>Pasos siguientes
- Para más información sobre la creación de grupos de acciones con distintos tipos de acción, consulte [Creación y administración de grupos de acciones en Azure Portal](../azure-monitor/platform/action-groups.md).
- Para obtener más información sobre los registros de actividad, consulte [Registro de actividad de Azure](../azure-monitor/platform/activity-log.md).
- Para obtener información sobre cómo configurar alertas en los registros de actividad, consulte [Alertas de registro de actividad](../azure-monitor/platform/activity-log-alerts.md).

