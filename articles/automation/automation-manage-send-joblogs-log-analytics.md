---
title: Reenvío de datos de un trabajo de Azure Automation a registros de Azure Monitor
description: En este artículo se indica cómo enviar el estado de un trabajo y los flujos de trabajo de runbook a los registros de Azure Monitor.
services: automation
ms.subservice: process-automation
ms.date: 09/02/2020
ms.topic: conceptual
ms.openlocfilehash: 6dcd2005971927de30ca96173cb2bdb063e46663
ms.sourcegitcommit: 5a3b9f35d47355d026ee39d398c614ca4dae51c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89397444"
---
# <a name="forward-azure-automation-job-data-to-azure-monitor-logs"></a>Reenvío de datos de un trabajo de Azure Automation a registros de Azure Monitor

Azure Automation puede enviar el estado de un trabajo del runbook y de flujos de trabajos al área de trabajo de Log Analytics. Este proceso no implica vincular el área de trabajo y es completamente independiente. Los registros de trabajo y flujos de trabajo están visibles en Azure Portal, o con PowerShell, para los trabajos individuales y esto permite llevar a cabo investigaciones simples. Ahora con los registros de Azure Monitor, puede:

* Obtener información sobre el estado de los trabajos de Automation.
* Desencadenar un correo electrónico o una alerta en función del estado de trabajo del runbook (por ejemplo, error o en suspensión).
* Escribir consultas avanzadas en las transmisiones de trabajos.
* Correlacionar trabajos en cuentas de Automation.
* Usar vistas personalizadas y consultas de búsqueda para visualizar los resultados de runbook, el estado del trabajo de runbook y otras métricas o indicadores clave relacionados.

## <a name="prerequisites"></a>Prerrequisitos

Para empezar a enviar los registros de Automation a registros de Azure Monitor, necesita:

* La versión más reciente de [Azure PowerShell](/powershell/azure/).

* Área de trabajo de Log Analytics con su id. de recurso. Para más información, consulte [Introducción a Azure Monitor](../azure-monitor/overview.md).

* Id. de recurso de su cuenta de Azure Automation.

## <a name="how-to-find-resource-ids"></a>Búsqueda de identificadores de recursos

1. Use el siguiente comando para conocer el identificador de recurso de su cuenta de Azure Automation:

    ```powershell-interactive
    # Find the ResourceId for the Automation account
    Get-AzResource -ResourceType "Microsoft.Automation/automationAccounts"
    ```

2. Copie el valor de **ResourceID**.

3. Utilice el comando siguiente para buscar el identificador de recurso de su área de trabajo de Log Analytics:

    ```powershell-interactive
    # Find the ResourceId for the Log Analytics workspace
    Get-AzResource -ResourceType "Microsoft.OperationalInsights/workspaces"
    ```

4. Copie el valor de **ResourceID**.

Para devolver los resultados de un grupo de recursos específico, incluya el parámetro `-ResourceGroupName`. Para obtener más información, consulte [Get-AzResource](/powershell/module/az.resources/get-azresource).

Si tiene más de una cuenta de Automation o un área de trabajo en la salida de los comandos anteriores, puede encontrar el nombre y otras propiedades relacionadas que forman parte del identificador de recurso completo de la cuenta de Automation realizando lo siguiente:

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En Azure Portal, seleccione la cuenta de Automation en la página **Cuentas de Automation**.
1. En la página de la cuenta de Automation seleccionada, en **Configuración de cuenta**, seleccione **Propiedades**.
1. En la página **Propiedades**, observe los detalles que se muestran a continuación.

    ![Propiedades de la cuenta de Automation](media/automation-manage-send-joblogs-log-analytics/automation-account-properties.png).

## <a name="configure-diagnostic-settings"></a>Configuración de valores de diagnóstico

La configuración de diagnóstico de Automation admite el reenvío de los registros de plataforma y datos de métricas siguientes:

* JobLogs
* JobStreams
* DSCNodeStatus
* Métricas: total de trabajos, ejecuciones de máquinas de implementación de actualizaciones totales, ejecuciones de implementación de actualizaciones totales

Para empezar a enviar los registros de Automation a registros de Azure Monitor, revise [Creación de una configuración de diagnóstico](../azure-monitor/platform/diagnostic-settings.md) para comprender la característica y los métodos disponibles para configurar los valores de diagnóstico para enviar registros de plataforma.

## <a name="azure-monitor-log-records"></a>Registros de Azure Monitor

Los diagnósticos de Azure Automation crean dos tipos de registros en los registros de Azure Monitor, etiquetados como `AzureDiagnostics`. Las tablas de las secciones siguientes son ejemplos de registros que genera Azure Automation y los tipos de datos que aparecen en los resultados de búsqueda de los registros.

### <a name="job-logs"></a>Registros de trabajo

| Propiedad | Descripción |
| --- | --- |
| TimeGenerated |Fecha y hora en que se ejecuta el trabajo de Runbook. |
| RunbookName_s |El nombre del Runbook. |
| Caller_s |El autor de la llamada que inició la operación. Los valores posibles son una dirección de correo electrónico o el sistema para los trabajos programados. |
| Tenant_g | El GUID que identifica al inquilino del autor de la llamada. |
| JobId_g |El GUID que identifica el trabajo de runbook. |
| ResultType |El estado del trabajo de Runbook. Los valores posibles son:<br>- New<br>- Created<br>Started<br>Stopped<br>Suspended<br>Con error<br>Completado |
| Category | Clasificación del tipo de datos. Para Automation, el valor será JobLogs. |
| OperationName | El tipo de operación realizada en Azure. En Automation, el valor es Job. |
| Resource | El nombre de la cuenta de Automation |
| SourceSystem | El sistema que utilizan los registros de Azure Monitor para recopilar los datos. En los diagnósticos de Azure, el valor siempre es "Azure". |
| ResultDescription |El estado de resultado del trabajo de runbook. Los valores posibles son:<br>- Se inicia el trabajo<br>- Error del trabajo<br>- Trabajo completado |
| CorrelationId |El GUID de correlación del trabajo de runbook. |
| ResourceId |El identificador de recurso de la cuenta de Azure Automation del runbook. |
| SubscriptionId | El GUID de la suscripción de Azure para la cuenta de Automation. |
| ResourceGroup | nombre del grupo de recursos de la cuenta de Automation. |
| ResourceProvider | El proveedor de recursos. El valor es MICROSOFT.AUTOMATION. |
| ResourceType | El tipo de recurso. El valor es AUTOMATIONACCOUNTS. |

### <a name="job-streams"></a>Transmisiones de trabajo
| Propiedad | Descripción |
| --- | --- |
| TimeGenerated |Fecha y hora en que se ejecuta el trabajo de Runbook. |
| RunbookName_s |El nombre del Runbook. |
| Caller_s |El autor de la llamada que inició la operación. Los valores posibles son una dirección de correo electrónico o el sistema para los trabajos programados. |
| StreamType_s |El tipo de flujo de trabajo. Los valores posibles son:<br>progreso<br>- Salida<br>Warning (Advertencia)<br>Error<br>DEBUG<br>- Detallado |
| Tenant_g | El GUID que identifica al inquilino del autor de la llamada. |
| JobId_g |El GUID que identifica el trabajo de runbook. |
| ResultType |El estado del trabajo de Runbook. Los valores posibles son:<br>- In Progress |
| Category | Clasificación del tipo de datos. Para Automation, el valor es JobStreams. |
| OperationName | El tipo de operación realizada en Azure. En Automation, el valor es Job. |
| Resource | El nombre de la cuenta de Automation. |
| SourceSystem | El sistema que utilizan los registros de Azure Monitor para recopilar los datos. En los diagnósticos de Azure, el valor siempre es "Azure". |
| ResultDescription |Descripción que incluye el flujo de salida del runbook. |
| CorrelationId |El GUID de correlación del trabajo de runbook. |
| ResourceId |El identificador de recurso de la cuenta de Azure Automation del runbook. |
| SubscriptionId | El GUID de la suscripción de Azure para la cuenta de Automation. |
| ResourceGroup | nombre del grupo de recursos de la cuenta de Automation. |
| ResourceProvider | El proveedor de recursos. El valor es MICROSOFT.AUTOMATION. |
| ResourceType | El tipo de recurso. El valor es AUTOMATIONACCOUNTS. |

## <a name="view-automation-logs-in-azure-monitor-logs"></a>Visualización de registros de Automation en registros de Azure Monitor

Ahora que ha iniciado el envío de los registros y los flujos de trabajo de Automation a los registros de Azure Monitor, veamos lo que puede hacer con ellos en los registros de Azure Monitor.

Para ver los registros, ejecute la consulta siguiente: `AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION"`

### <a name="send-an-email-when-a-runbook-job-fails-or-suspends"></a>Envío de un correo electrónico cuando se produce un error en un trabajo del runbook o se suspende

En los pasos siguientes se muestra cómo configurar alertas en Azure Monitor para recibir una notificación cuando se produzca algún problema con un trabajo de runbook.

Para crear una regla de alerta, lo primero es crear una búsqueda de los registros del trabajo del runbook que deben invocar la alerta. Haga clic en el botón **Alerta** para crear y configurar la regla de alerta.

1. En la página de información general del área de trabajo de Log Analytics, haga clic en **Ver registros**.

2. Cree una consulta de búsqueda de registros para la alerta; para ello, escriba la siguiente búsqueda en el campo de la consulta: `AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobLogs" and (ResultType == "Failed" or ResultType == "Suspended")`<br><br>También puede agrupar por el nombre del runbook, para lo que debe usar: `AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobLogs" and (ResultType == "Failed" or ResultType == "Suspended") | summarize AggregatedValue = count() by RunbookName_s`

   Si configura registros de más de una cuenta de Automation o suscripción a su área de trabajo, puede agrupar las alertas por suscripción o cuenta de Automation. El nombre de la cuenta de Automation puede encontrarse en el campo `Resource` de la búsqueda de `JobLogs`.

3. Para abrir la pantalla **Crear regla**, haga clic en **Nueva regla de alertas** en la parte superior de la página. Para más información sobre las opciones para configurar la alerta, consulte [Alertas de registro en Azure](../azure-monitor/platform/alerts-unified-log.md).

### <a name="find-all-jobs-that-have-completed-with-errors"></a>Búsqueda de todos los trabajos que se han completado con errores

Además de las alertas al producirse errores, puede saber cuando un trabajo de runbook tiene un error de no terminación. En estos casos, PowerShell genera un flujo de errores, aunque los errores de no terminación no causan la suspensión ni producen un error en el trabajo.

1. En el área de trabajo de Log Analytics, haga clic en **Registros**.

2. En el campo de consulta, escriba `AzureDiagnostics | where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobStreams" and StreamType_s == "Error" | summarize AggregatedValue = count() by JobId_g`.

3. Haga clic en el botón **Buscar**.

### <a name="view-job-streams-for-a-job"></a>Visualización de las transmisiones de un trabajo

Cuando se depura un trabajo, también se pueden examinar sus transmisiones. La siguiente consulta muestra todos los flujos de un solo trabajo con GUID `2ebd22ea-e05e-4eb9-9d76-d73cbd4356e0`:

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobStreams" and JobId_g == "2ebd22ea-e05e-4eb9-9d76-d73cbd4356e0"
| sort by TimeGenerated asc
| project ResultDescription
```

### <a name="view-historical-job-status"></a>Visualización del estado del historial de trabajos

Por último, puede que desee visualizar el historial de trabajos a lo largo del tiempo. Esta consulta se puede usar para buscar el estado de los trabajos con el paso del tiempo.

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.AUTOMATION" and Category == "JobLogs" and ResultType != "started"
| summarize AggregatedValue = count() by ResultType, bin(TimeGenerated, 1h)
```

![Gráfico del historial de trabajos de Log Analytics](media/automation-manage-send-joblogs-log-analytics/historical-job-status-chart.png)

### <a name="filter-job-status-output-converted-into-a-json-object"></a>Filtrado de la salida de estado del trabajo convertida en un objeto JSON

Recientemente hemos cambiado el comportamiento de la escritura de los datos de registros de Automation en la tabla `AzureDiagnostics` del servicio Log Analytics, donde ya no se desglosan las propiedades JSON en campos independientes. Si configuró el runbook para dar formato a los objetos del flujo de salida en formato JSON como columnas independientes, es necesario volver a configurar las consultas para analizar ese campo en un objeto JSON con el fin de acceder a esas propiedades. Esto se logra mediante [parsejson](../azure-monitor/log-query/json-data-structures.md#parsejson) para acceder a un elemento JSON específico en una ruta de acceso conocida.

Por ejemplo, un runbook da formato a la propiedad *ResultDescription* en el flujo de salida en formato JSON con varios campos. Para buscar el estado de los trabajos que se encuentran en estado de error, tal y como se especifica en un campo denominado **Estado**, use esta consulta de ejemplo para buscar el objeto *ResultDescription* con el estado **Error**:

```kusto
AzureDiagnostics
| where Category == 'JobStreams'
| extend jsonResourceDescription = parse_json(ResultDescription)
| where jsonResourceDescription.Status == 'Failed'
```

![Formato JSON del flujo de trabajo histórico de Log Analytics](media/automation-manage-send-joblogs-log-analytics/job-status-format-json.png)

## <a name="next-steps"></a>Pasos siguientes

* Para aprender a crear consultas de búsqueda y a revisar los registros de trabajos de Automation con registros de Azure Monitor, consulte [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/log-query/log-query-overview.md).
* Para comprender la creación y recuperación de mensajes de error y de salida de runbooks, consulte [Salidas de runbook y mensajes en Azure Automation](automation-runbook-output-and-messages.md).
* Para más información sobre la ejecución de runbooks, la supervisión de trabajos de runbook y otros detalles técnicos, consulte [Ejecución de un runbook en Azure Automation](automation-runbook-execution.md).
* Para obtener más información sobre los registros de Azure Monitor y los orígenes de recopilación de datos, consulte [Introducción a la recopilación de datos de almacenamiento en los registros de Azure Monitor](../azure-monitor/platform/resource-logs.md#send-to-log-analytics-workspace).
* Si necesita ayuda con la solución de problemas de análisis de registros, consulte el artículo sobre [por qué el análisis de registros ya no recopila datos](../azure-monitor/platform/manage-cost-storage.md#troubleshooting-why-log-analytics-is-no-longer-collecting-data).
