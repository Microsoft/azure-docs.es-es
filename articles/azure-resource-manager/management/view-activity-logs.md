---
title: Visualización de registros de actividad de Azure para supervisar recursos
description: Use los registros de actividad para revisar las acciones y los errores de los usuarios. Muestra PowerShell de Azure Portal, la CLI de Azure y REST.
ms.topic: conceptual
ms.date: 05/13/2019
ms.openlocfilehash: b02865e4487a187d1296b2971918aa101d711b5b
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87827068"
---
# <a name="view-activity-logs-to-monitor-actions-on-resources"></a>Visualización de registros de actividad para supervisar acciones sobre recursos

Mediante los registros de actividad, puede determinar:

* qué operaciones se realizaron en los recursos en la suscripción
* quién inició la operación
* cuándo tuvo lugar la operación
* el estado de la operación
* los valores de otras propiedades que podrían ayudarle en la investigación de la operación

El registro de actividad contiene todas las operaciones de escritura (PUT, POST, DELETE) para los recursos. No incluye operaciones de lectura (GET). Para obtener una lista de las acciones de recurso, consulte [Operaciones del proveedor de recursos de Azure Resource Manager](../../role-based-access-control/resource-provider-operations.md). Puede usar los registros de actividad para encontrar un error al solucionar problemas o para supervisar cómo un usuario de su organización modificó un recurso.

Los registros de actividad se conservan 90 días. Puede consultar cualquier intervalo de fechas, siempre que no hayan transcurrido más de 90 días desde la fecha inicial.

Puede recuperar información de los registros de actividad mediante el portal, PowerShell, la CLI de Azure, la API de REST de Insights o la [biblioteca .NET de Insights](https://www.nuget.org/packages/Microsoft.Azure.Insights/).

## <a name="azure-portal"></a>Portal de Azure

Para ver los registros de actividad mediante el portal, siga estos pasos:

1. En el menú de Azure Portal, seleccione **Monitor** o busque y seleccione **Monitor** desde cualquier página.

    ![Seleccionar Monitor](./media/view-activity-logs/select-monitor-from-menu.png)

1. Seleccione **Registro de actividad**.

    ![Seleccionar el registro de actividades](./media/view-activity-logs/select-activity-log.png)

1. Verá un resumen de operaciones recientes. Se aplica un conjunto predeterminado de filtros a las operaciones. Tenga en cuenta que en la información del resumen se incluye quién ha iniciado la acción y cuándo ha tenido lugar.

    ![Vista de resumen de las operaciones recientes](./media/view-activity-logs/audit-summary.png)

1. Para ejecutar de forma rápida un conjunto predefinido de filtros, seleccione **Conclusiones rápidas**.

    ![Selección de Conclusiones rápidas](./media/view-activity-logs/select-quick-insights.png)

1. Seleccione una de las opciones. Por ejemplo, seleccione **Implementaciones con errores** para ver los errores de las implementaciones.

    ![Selección de Implementaciones con errores](./media/view-activity-logs/select-failed-deployments.png)

1. Tenga en cuenta que los filtros se han cambiado para centrarse en los errores de implementación de las últimas 24 horas. Solo se muestran las operaciones que coinciden con los filtros.

    ![Ver filtros](./media/view-activity-logs/view-filters.png)

1. Para centrarse en operaciones específicas, cambie los filtros o aplique otros nuevos. Por ejemplo, la siguiente imagen muestra que se estableció un nuevo valor para **Timespan** y **Resource type** en cuentas de almacenamiento.

    ![Establecer opciones de filtro](./media/view-activity-logs/set-filter.png)

1. Si necesita volver a ejecutar la consulta más tarde, seleccione **Anclar los filtros actuales**.

    ![Anclar filtros](./media/view-activity-logs/pin-filters.png)

1. Asigne un nombre al filtro.

    ![Asignar un nombre a los filtros](./media/view-activity-logs/name-filters.png)

1. El filtro está disponible en el panel. En el menú de Azure Portal, seleccione **Panel**.

    ![Mostrar filtro en el panel](./media/view-activity-logs/activity-log-on-dashboard.png)

1. Desde el portal, puede ver los cambios realizados en un recurso. Vaya hasta la vista predeterminada en el Monitor y seleccione una operación que implique el cambio de un recurso.

    ![Selección de la operación](./media/view-activity-logs/select-operation.png)

1. Seleccione **Historial de cambios (versión preliminar)** y elija una de las operaciones disponibles.

    ![Selección del Historial de cambios](./media/view-activity-logs/select-change-history.png)

1. Se mostrarán los cambios en el recurso.

    ![Mostrar cambios](./media/view-activity-logs/show-changes.png)

Para obtener más información sobre el historial de cambios, vea [Obtención de los cambios del recurso](../../governance/resource-graph/how-to/get-resource-changes.md).

## <a name="powershell"></a>PowerShell

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Para recuperar las entradas de registro, ejecute el comando **Get-AzLog**. Ofrezca parámetros adicionales para filtrar la lista de entradas. Si no especifica una hora de inicio y de finalización, se devuelven las entradas de los últimos siete días.

```azurepowershell-interactive
Get-AzLog -ResourceGroup ExampleGroup
```

En el ejemplo siguiente se muestra cómo usar el registro de actividad para investigar operaciones llevadas a cabo durante un tiempo especificado. Las fechas inicial y final se especifican en un formato de fecha.

```azurepowershell-interactive
Get-AzLog -ResourceGroup ExampleGroup -StartTime 2019-05-05T06:00 -EndTime 2019-05-09T06:00
```

También puede usar funciones de fecha para especificar el intervalo de fechas, como los últimos 14 días.

```azurepowershell-interactive
Get-AzLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-14)
```

Puede buscar las acciones realizadas por un usuario determinado.

```azurepowershell-interactive
Get-AzLog -ResourceGroup ExampleGroup -StartTime (Get-Date).AddDays(-14) -Caller someone@contoso.com
```

Puede filtrar por las operaciones con errores.

```azurepowershell-interactive
Get-AzLog -ResourceGroup ExampleGroup -Status Failed
```

Para centrarse en un error, examine el mensaje de estado para esa entidad.

```azurepowershell-interactive
(Get-AzLog -ResourceGroup ExampleGroup -Status Failed).Properties.Content.statusMessage | ConvertFrom-Json
```

Puede seleccionar valores específicos para limitar los datos que se devuelven.

```azurepowershell-interactive
Get-AzLog -ResourceGroupName ExampleGroup | Format-table EventTimeStamp, Caller, @{n='Operation'; e={$_.OperationName.value}}, @{n='Status'; e={$_.Status.value}}, @{n='SubStatus'; e={$_.SubStatus.LocalizedValue}}
```

En función de la hora de inicio que especifique, los comandos anteriores pueden devolver una lista larga de operaciones para el grupo de recursos. Puede filtrar los resultados para lo que busca ofreciendo criterios de búsqueda. Por ejemplo, puede filtrar por el tipo de operación.

```azurepowershell-interactive
Get-AzLog -ResourceGroup ExampleGroup | Where-Object {$_.OperationName.value -eq "Microsoft.Resources/deployments/write"}
```

Puede usar Resource Graph para ver el historial de cambios de un recurso. Para más información, vea [Obtención de los cambios del recurso](../../governance/resource-graph/how-to/get-resource-changes.md).

## <a name="azure-cli"></a>Azure CLI

Para recuperar las entradas del registro, ejecute el comando [az monitor activity-log list](/cli/azure/monitor/activity-log#az-monitor-activity-log-list) con un desplazamiento para indicar el intervalo de tiempo.

```azurecli-interactive
az monitor activity-log list --resource-group ExampleGroup --offset 7d
```

En el ejemplo siguiente se muestra cómo usar el registro de actividad para investigar operaciones llevadas a cabo durante un tiempo especificado. Las fechas inicial y final se especifican en un formato de fecha.

```azurecli-interactive
az monitor activity-log list -g ExampleGroup --start-time 2019-05-01 --end-time 2019-05-15
```

Puede buscar las acciones realizadas por un usuario determinado, incluso para un grupo de recursos que ya no existe.

```azurecli-interactive
az monitor activity-log list -g ExampleGroup --caller someone@contoso.com --offset 5d
```

Puede filtrar por las operaciones con errores.

```azurecli-interactive
az monitor activity-log list -g ExampleGroup --status Failed --offset 1d
```

Para centrarse en un error, examine el mensaje de estado para esa entidad.

```azurecli-interactive
az monitor activity-log list -g ExampleGroup --status Failed --offset 1d --query [].properties.statusMessage
```

Puede seleccionar valores específicos para limitar los datos que se devuelven.

```azurecli-interactive
az monitor activity-log list -g ExampleGroup --offset 1d --query '[].{Operation: operationName.value, Status: status.value, SubStatus: subStatus.localizedValue}'
```

En función de la hora de inicio que especifique, los comandos anteriores pueden devolver una lista larga de operaciones para el grupo de recursos. Puede filtrar los resultados para lo que busca ofreciendo criterios de búsqueda. Por ejemplo, puede filtrar por el tipo de operación.

```azurecli-interactive
az monitor activity-log list -g ExampleGroup --offset 1d --query "[?operationName.value=='Microsoft.Storage/storageAccounts/write']"
```

Puede usar Resource Graph para ver el historial de cambios de un recurso. Para más información, vea [Obtención de los cambios del recurso](../../governance/resource-graph/how-to/get-resource-changes.md).

## <a name="rest-api"></a>API DE REST

Las operaciones REST para trabajar con el registro de actividad forman parte de la [API de REST de Insights](/rest/api/monitor/). Para recuperar eventos de registro de actividad, consulte [Lista de los eventos de administración de una suscripción](/rest/api/monitor/activitylogs).

## <a name="next-steps"></a>Pasos siguientes

* Los registros de actividad de Azure se pueden usar con Power BI para obtener mayor información sobre las acciones en su suscripción. Consulte [View and analyze Azure Audit Logs in Power BI and more](https://azure.microsoft.com/blog/analyze-azure-audit-logs-in-powerbi-more/)(Consulta y análisis de registros de auditoría de Azure en Power BI y más).
* Para obtener información sobre cómo establecer directivas de seguridad, consulte [Control de acceso basado en rol de Azure (Azure RBAC)](../../role-based-access-control/role-assignments-portal.md).
* Para ver más detalles sobre los cambios en las aplicaciones desde el nivel de infraestructura hasta la implementación de aplicaciones, consulte el artículo sobre el [Uso de Application Change Analysis en Azure Monitor](../../azure-monitor/app/change-analysis.md).
* Para aprender sobre los comandos para ver operaciones de implementación, consulte [Visualización de operaciones de implementación](../templates/deployment-history.md).
* Para aprender a evitar eliminaciones en un recurso para todos los usuarios, consulte [Bloqueo de recursos con Azure Resource Manager](lock-resources.md).
* Para ver la lista de operaciones disponibles para cada proveedor de Microsoft Azure Resource Manager, consulte [Operaciones del proveedor de recursos de Azure Resource Manager](../../role-based-access-control/resource-provider-operations.md)
