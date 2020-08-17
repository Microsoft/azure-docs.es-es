---
title: Actualización de aplicaciones lógicas y runbooks para la migración de alertas
description: Obtenga información sobre cómo modificar los webhooks, las aplicaciones lógicas y los runbooks para preparar la migración voluntaria.
author: yanivlavi
ms.author: yalavi
ms.topic: conceptual
ms.date: 03/19/2018
ms.subservice: alerts
ms.openlocfilehash: 09db7684c84bbde038c67f9ccfb3f27f6b61bee6
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86539557"
---
# <a name="prepare-your-logic-apps-and-runbooks-for-migration-of-classic-alert-rules"></a>Preparar las aplicaciones lógicas y los runbooks para la migración de las reglas de alertas clásicas

Tal como se [anunció anteriormente](monitoring-classic-retirement.md), las alertas clásicas de Azure Monitor se retirarán en septiembre de 2019 (la fecha original era julio de 2019). Hay disponible una herramienta de migración en Azure Portal para los clientes que usan reglas de alertas clásicas y que desean desencadenar la migración ellos mismos.

> [!NOTE]
> Debido a un retraso en la implementación de la herramienta de migración, la fecha de retirada para la migración de alertas clásicas se ha ampliado hasta el 31 de agosto de 2019, desde la fecha anunciada originalmente del 30 de junio de 2019.

Si opta por migrar voluntariamente las reglas de alertas clásicas a las nuevas reglas de alertas, tenga en cuenta que existen algunas diferencias entre los dos sistemas. En este artículo se explican esas diferencias y cómo puede prepararse para el cambio.

## <a name="api-changes"></a>Cambios de API

Las API que crean y administran las reglas de alertas clásicas (`microsoft.insights/alertrules`) son diferentes de las API que crean y administran las nuevas alertas de métricas (`microsoft.insights/metricalerts`). Si crea y administra mediante programación reglas de alertas clásicas hoy, actualice los scripts de implementación para trabajar con las nuevas API.

La tabla siguiente es una referencia a las interfaces de programación para las alertas clásicas y nuevas:

| Tipo de script de implementación | Alertas clásicas | Nuevas alertas de métricas |
| ---------------------- | -------------- | ----------------- |
|API DE REST     | [microsoft.insights/alertrules](/rest/api/monitor/alertrules)         | [microsoft.insights/metricalerts](/rest/api/monitor/metricalerts)       |
|Azure CLI     | [az monitor alert](/cli/azure/monitor/alert?view=azure-cli-latest)        | [az monitor metrics alert](/cli/azure/monitor/metrics/alert?view=azure-cli-latest)        |
|PowerShell      | [Referencia](/powershell/module/az.monitor/add-azmetricalertrule)       |  [Referencia](/powershell/module/az.monitor/add-azmetricalertrulev2)    |
| Plantilla del Administrador de recursos de Azure | [Para alertas clásicas](./alerts-enable-template.md)|[Para nuevas alertas de métricas](./alerts-metric-create-templates.md)|

## <a name="notification-payload-changes"></a>Cambios de carga de notificación

El formato de carga de notificación es ligeramente diferente entre las [reglas de alertas clásicas](alerts-webhooks.md) y las [nuevas alertas de métricas](alerts-metric-near-real-time.md#payload-schema). Si tiene alguna acción de webhook, aplicación lógica o runbook que se desencadene mediante reglas de alertas clásicas, debe actualizar esos puntos de conexión de notificación para que acepten el formato de carga de las nuevas alertas de métricas.

Utilice la tabla siguiente para asignar los campos de carga de webhook del formato clásico al nuevo formato:

| Tipo de punto de conexión de notificaciones | Alertas clásicas | Nuevas alertas de métricas |
| -------------------------- | -------------- | ----------------- |
|¿Se activó o resolvió la alerta?    | **status**       | **data.status** |
|Información contextual sobre la alerta     | **contextoo**        | **data.context**        |
|Marca de tiempo en que se activó o resolvió la alerta     | **context.timestamp**       | **data.context.timestamp**        |
| Id. de regla de alerta | **context.id** | **data.context.id** |
| Nombre de regla de alerta | **context.name** | **data.context.name** |
| Descripción de la regla de alerta | **context.description** | **data.context.description** |
| Condición de regla de alerta | **context.condition** | **data.context.condition** |
| Nombre de métrica | **context.condition.metricName** | **data.context.condition.allOf[0].metricName** |
| Agregación de tiempo (cómo se agrega la métrica en la ventana de evaluación)| **context.condition.timeAggregation** | **context.condition.timeAggregation** |
| Período de evaluación | **context.condition.windowSize** | **data.context.condition.windowSize** |
| Operador (cómo se compara el valor de métrica agregado con el umbral) | **context.condition.operator** | **data.context.condition.operator** |
| Umbral | **context.condition.threshold** | **data.context.condition.allOf[0].threshold** |
| Valor de métrica | **context.condition.metricValue** | **data.context.condition.allOf[0].metricValue** |
| Id. de suscripción | **context.subscriptionId** | **data.context.subscriptionId** |
| Grupo de recursos del recurso afectado | **context.resourceGroup** | **data.context.resourceGroup** |
| Nombre del recurso afectado | **context.resourceName** | **data.context.resourceName** |
| Tipo del recurso afectado | **context.resourceType** | **data.context.resourceType** |
| Id. del recurso afectado | **context.resourceId** | **data.context.resourceId** |
| Vínculo directo a la página de resumen de recursos del portal | **context.portalLink** | **data.context.portalLink** |
| Campos de carga personalizada que se deben pasar a la aplicación lógica o webhook | **properties** | **data.properties** |

Las cargas son similares, como puede ver. En la siguiente sección se ofrece:

- Detalles acerca de cómo modificar las aplicaciones lógicas para trabajar con el nuevo formato.
- Un ejemplo de runbook que analiza la carga de notificación para las alertas nuevas.

## <a name="modify-a-logic-app-to-receive-a-metric-alert-notification"></a>Modificar una aplicación lógica para recibir una notificación de alerta de métrica

Si usa aplicaciones lógicas con alertas clásicas, debe modificar el código de aplicación de lógica para analizar la nueva carga de alertas de métricas. Siga estos pasos:

1. Cree una nueva aplicación lógica.

1. Use la plantilla "Azure Monitor - Metrics Alert Handler" (Azure Monitor: controlador de alertas de métricas). Esta plantilla tiene un desencadenador de **solicitud HTTP** con el esquema apropiado definido.

    ![logic-app-template](media/alerts-migration/logic-app-template.png "Plantilla de alerta de métrica")

1. Agregue una acción para hospedar la lógica de procesamiento.

## <a name="use-an-automation-runbook-that-receives-a-metric-alert-notification"></a>Usar un runbook de automatización que reciba una notificación de alerta de métrica

En el ejemplo siguiente se proporciona código de PowerShell para usar en el runbook. Este código puede analizar las cargas para reglas de alertas de métricas clásicas y nuevas reglas de alertas de métricas.

```PowerShell
## Example PowerShell code to use in a runbook to handle parsing of both classic and new metric alerts.

[OutputType("PSAzureOperationResponse")]

param
(
    [Parameter (Mandatory=$false)]
    [object] $WebhookData
)

$ErrorActionPreference = "stop"

if ($WebhookData)
{
    # Get the data object from WebhookData.
    $WebhookBody = (ConvertFrom-Json -InputObject $WebhookData.RequestBody)

    # Determine whether the alert triggering the runbook is a classic metric alert or a new metric alert (depends on the payload schema).
    $schemaId = $WebhookBody.schemaId
    Write-Verbose "schemaId: $schemaId" -Verbose
    if ($schemaId -eq "AzureMonitorMetricAlert") {

        # This is the new metric alert schema.
        $AlertContext = [object] ($WebhookBody.data).context
        $status = ($WebhookBody.data).status

        # Parse fields related to alert rule condition.
        $metricName = $AlertContext.condition.allOf[0].metricName
        $metricValue = $AlertContext.condition.allOf[0].metricValue
        $threshold = $AlertContext.condition.allOf[0].threshold
        $timeAggregation = $AlertContext.condition.allOf[0].timeAggregation
    }
    elseif ($schemaId -eq $null) {
        # This is the classic metric alert schema.
        $AlertContext = [object] $WebhookBody.context
        $status = $WebhookBody.status

        # Parse fields related to alert rule condition.
        $metricName = $AlertContext.condition.metricName
        $metricValue = $AlertContext.condition.metricValue
        $threshold = $AlertContext.condition.threshold
        $timeAggregation = $AlertContext.condition.timeAggregation
    }
    else {
        # The schema is neither a classic metric alert nor a new metric alert.
        Write-Error "The alert data schema - $schemaId - is not supported."
    }

    # Parse fields related to resource affected.
    $ResourceName = $AlertContext.resourceName
    $ResourceType = $AlertContext.resourceType
    $ResourceGroupName = $AlertContext.resourceGroupName
    $ResourceId = $AlertContext.resourceId
    $SubId = $AlertContext.subscriptionId

    ## Your logic to handle the alert here.
}
else {
    # Error
    Write-Error "This runbook is meant to be started from an Azure alert webhook only."
}

```

Para obtener un ejemplo completo de un runbook que detenga una máquina virtual cuando se desencadene una alerta, consulte la [documentación de Azure Automation](../../automation/automation-create-alert-triggered-runbook.md).

## <a name="partner-integration-via-webhooks"></a>Integración de asociados a través de webhooks

La mayoría de [nuestros asociados que se integran con las alertas clásicas](./partners.md) ya son compatibles con las nuevas alertas de métricas a través de sus integraciones. Las integraciones conocidas que ya funcionan con las nuevas alertas de métricas son:

- [PagerDuty](https://www.pagerduty.com/docs/guides/azure-integration-guide/)
- [OpsGenie](https://docs.opsgenie.com/docs/microsoft-azure-integration)
- [Signl4](https://www.signl4.com/blog/mobile-alert-notifications-azure-monitor/)

Si usa una integración de asociados que no aparece aquí, confirme con el proveedor de integración que funciona la integración con las nuevas alertas de métricas.

## <a name="next-steps"></a>Pasos siguientes

- [Uso de la herramienta de migración](alerts-using-migration-tool.md)
- [Descripción del funcionamiento de la herramienta de migración](alerts-understand-migration.md)
