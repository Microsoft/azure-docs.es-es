---
title: Creación de alertas de métricas para registros en Azure Monitor
description: Tutorial sobre la creación de alertas de métricas casi en tiempo real sobre los datos de análisis de registros populares.
author: harelbr
ms.author: harelbr
ms.topic: conceptual
ms.date: 06/17/2020
ms.subservice: alerts
ms.openlocfilehash: 7085dd601499004a91fc77a9181f0b097d0b543a
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87446185"
---
# <a name="create-metric-alerts-for-logs-in-azure-monitor"></a>Creación de alertas de métricas para registros en Azure Monitor

## <a name="overview"></a>Información general

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Azure Monitor admite el [tipo de alertas de métricas](./alerts-metric-near-real-time.md), que tiene ventajas sobre las [alertas clásicas](./alerts-classic-portal.md). Las métricas están disponibles para una [amplia lista de servicios de Azure](./metrics-supported.md). En este artículo se explica el uso de un subconjunto (es decir) para el recurso `Microsoft.OperationalInsights/workspaces`.

También puede utilizar las alertas de métricas en los registros populares de Log Analytics extraídos como métricas como parte de las métricas de los registros, incluidos recursos en Azure o en la instancia local. A continuación se enumeran las soluciones de Log Analytics admitidas:

- [Contadores de rendimiento](./data-sources-performance-counters.md) para equipos con Windows y Linux
- [Registros de latidos para Agent Health](../insights/solution-agenthealth.md)
- Registros de [administración de actualizaciones](../../automation/update-management/update-mgmt-overview.md)
- [Datos de registros de eventos](./data-sources-windows-events.md)

Hay muchas ventajas al usar **alertas de métricas para registros** a través de [alertas de registro](./alerts-log.md) basadas en consultas en Azure; a continuación se enumeran algunas de ellas:

- Las alertas de métricas ofrecen la funcionalidad de supervisión casi en tiempo real, y las alertas de métricas para registros bifurcan los datos a partir del origen de registro para garantizar lo mismo.
- Las alertas de métricas tienen estado, solo se notifican una vez cuando se desencadena la alerta y una vez cuando se resuelva la alerta; a diferencia de las alertas de registro, que no tienen estado y siguen notificándose en cada intervalo si se cumple la condición de alerta.
- Las alertas de métricas para registros proporcionan varias dimensiones, lo que permite filtrar valores específicos, como equipos, tipo de sistema operativo, etc., de manera más sencilla, sin la necesidad de componer una consulta en el análisis.

> [!NOTE]
> La métrica o dimensión específicas solo se mostrarán si los datos de ellas existen durante el período elegido. Estas métricas están disponibles para los clientes con áreas de trabajo de Azure Log Analytics.

## <a name="metrics-and-dimensions-supported-for-logs"></a>Métricas y dimensiones compatibles para registros

 Las alertas de métricas admiten alertas de métricas que utilizan dimensiones. Puede usar dimensiones para filtrar las métricas al nivel adecuado. Se incluye la lista completa de métricas admitidas para los registros de [áreas de trabajo de Log Analytics](./metrics-supported.md#microsoftoperationalinsightsworkspaces); a través de las soluciones compatibles.

> [!NOTE]
> Para ver una métrica admitida extraída de un área de trabajo de Log Analytics a través de [Azure Monitor: Métricas](./metrics-charts.md), debe crearse una alerta de métrica para el registro en la métrica específica. Las dimensiones elegidas en la alerta de métrica para registros solo aparecerán para la exploración a través de Azure Monitor: Métricas.

## <a name="creating-metric-alert-for-log-analytics"></a>Creación de alertas de métricas para Log Analytics

Los datos de métricas de registros populares se canalizan antes de procesarse en Log Analytics, en Azure Monitor: Métrica. Esto permite a los usuarios aprovechar las funcionalidades de la plataforma de métricas, así como la alerta de métrica: incluso tener alertas con una frecuencia de apenas 1 minuto.
A continuación aparecen los medios para crear una alerta de métrica para registros.

## <a name="prerequisites-for-metric-alert-for-logs"></a>Requisitos previos para alertas de métricas para registros

Antes de que funcione la métrica para registros recopilada en los datos de Log Analytics, debe configurarse lo siguiente y estar disponible:

1. **Área de trabajo de Log Analytics activa**: debe haber un área de trabajo de Log Analytics activa y válida. Para obtener más información, consulte [Creación de un área de trabajo de Log Analytics en Azure Portal](../learn/quick-create-workspace.md).
2. **Agente configurado para el área de trabajo de Log Analytics**: el agente debe configurarse para que las máquinas virtuales de Azure o las máquinas virtuales locales envíen datos al área de trabajo de Log Analytics usada en el paso anterior. Para obtener más información, consulte [Log Analytics - Agent Overview](./agents-overview.md) (Log Analytics: Introducción al agente).
3. **Soluciones de Log Analytics admitidas instaladas**: la solución de Log Analytics debe estar configurada y enviar datos al área de trabajo de Log Analytics; las soluciones admitidas son [Contadores de rendimiento para Windows y Linux](./data-sources-performance-counters.md), [Registros de latidos para Agent Health](../insights/solution-agenthealth.md), [Administración de actualizaciones](../../automation/update-management/update-mgmt-overview.md) y [Datos de eventos](./data-sources-windows-events.md).
4. **Soluciones de Log Analytics configuradas para enviar registros**: la solución de Log Analytics debe tener los datos o registros que corresponden a las [métricas admitidas para las áreas de trabajo de Log Analytics](./metrics-supported.md#microsoftoperationalinsightsworkspaces) habilitadas. Por ejemplo, para el contador *% de memoria disponible*, debe configurarse primero en la solución [Contadores de rendimiento](./data-sources-performance-counters.md).

## <a name="configuring-metric-alert-for-logs"></a>Configuración de alertas de métricas para registros

 Las alertas de métricas se pueden crear y administrar mediante Azure Portal, plantillas de Resource Manager, la API REST, PowerShell y la CLI de Azure. Dado que las alertas de métricas para registros es una variante de alertas de métricas: una vez que se cumplan los requisitos previos, se puede crear una alerta de métrica para registros para el área de trabajo de Log Analytics especificada. Todas las características y funcionalidades de [ alertas de métricas](./alerts-metric-near-real-time.md) se aplicarán a las alertas de métricas para registros, incluido el esquema de carga, los límites de cuota aplicables y el precio facturado.

Para obtener instrucciones paso a paso y ejemplos, consulte [creación y administración de alertas de métricas](https://aka.ms/createmetricalert). En concreto, para las alertas de métricas de registros, siga las instrucciones para administrar las alertas de métricas y asegúrese de lo siguiente:

- El destino para la alerta de métrica es un *área de trabajo de Log Analytics* válida.
- La señal elegida para la alerta de métrica para el *área de trabajo de Log Analytics* seleccionada es del tipo **Métrica**.
- Filtrar por condiciones o recurso específicos mediante filtros de dimensión; las métricas para registros son multidimensionales.
- Al configurar la *lógica de señal*, se puede crear una sola alerta para abarcar varios valores de dimensión (por ejemplo, equipo).
- Si **no** usa Azure Portal para crear la alerta de métrica para el *área de trabajo de Log Analytics* seleccionada, el usuario debe crear primero manualmente una regla explícita para convertir los datos de registro en una métrica mediante [Azure Monitor: Scheduled Query Rules](/rest/api/monitor/scheduledqueryrules) (Azure Monitor: Reglas de consulta programadas).

> [!NOTE]
> Al crear la alerta de métrica para el área de trabajo de Log Analytics mediante Azure Portal, la regla correspondiente para convertir datos de registro en la métrica a través de [Azure Monitor: Scheduled Query Rules](/rest/api/monitor/scheduledqueryrules) se crea automáticamente en segundo plano,  *sin necesidad de ninguna intervención o acción del usuario*. Para la creación de alertas de métricas para registros a través de medios distintos de Azure Portal, consulte la sección [Plantilla de recursos para las alertas de métricas de registros](#resource-template-for-metric-alerts-for-logs) por medios de ejemplo para crear una regla de conversión de registro a métrica basada en ScheduledQueryRule antes de la creación de la alerta de métrica o, de lo contrario, no habrá datos para la alerta de métrica en los registros creados.

## <a name="resource-template-for-metric-alerts-for-logs"></a>Plantilla de recursos para las alertas de métricas de registros

Como se indicó anteriormente, el proceso de creación de alertas de métricas para registros tiene dos puntas:

1. Crear una regla para extraer las métricas de los registros admitidos mediante la API scheduledQueryRule
2. Crear una alerta de métrica para la métrica extraída del registro (en el paso 1) y el área de trabajo de Log Analytics como recurso de destino

### <a name="metric-alerts-for-logs-with-static-threshold"></a>Alertas de métricas para los registros de umbral estático

Para lograr el mismo, se puede utilizar el ejemplo de la plantilla de Azure Resource Manager a continuación, cuando la creación de la alerta de métrica de umbral estático dependa de la creación correcta de la regla para extraer las métricas de los registros a través de scheduledQueryRule.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "convertRuleName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the rule to convert log to metric"
            }
        },
        "convertRuleDescription": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Description for log converted to metric"
            }
        },
        "convertRuleRegion": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the region used by workspace"
            }
        },
        "convertRuleStatus": {
            "type": "string",
            "defaultValue": "true",
            "metadata": {
                "description": "Specifies whether the log conversion rule is enabled"
            }
        },
        "convertRuleMetric": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the metric once extraction done from logs."
            }
        },
        "alertName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the alert"
            }
        },
        "alertDescription": {
            "type": "string",
            "defaultValue": "This is a metric alert",
            "metadata": {
                "description": "Description of alert"
            }
        },
        "alertSeverity": {
            "type": "int",
            "defaultValue": 3,
            "allowedValues": [
                0,
                1,
                2,
                3,
                4
            ],
            "metadata": {
                "description": "Severity of alert {0,1,2,3,4}"
            }
        },
        "isEnabled": {
            "type": "bool",
            "defaultValue": true,
            "metadata": {
                "description": "Specifies whether the alert is enabled"
            }
        },
        "resourceId": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Full Resource ID of the resource emitting the metric that will be used for the comparison. For example: /subscriptions/00000000-0000-0000-0000-0000-00000000/resourceGroups/ResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
            }
        },
        "metricName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the metric used in the comparison to activate the alert."
            }
        },
        "operator": {
            "type": "string",
            "defaultValue": "GreaterThan",
            "allowedValues": [
                "Equals",
                "NotEquals",
                "GreaterThan",
                "GreaterThanOrEqual",
                "LessThan",
                "LessThanOrEqual"
            ],
            "metadata": {
                "description": "Operator comparing the current value with the threshold value."
            }
        },
        "threshold": {
            "type": "string",
            "defaultValue": "0",
            "metadata": {
                "description": "The threshold value at which the alert is activated."
            }
        },
        "timeAggregation": {
            "type": "string",
            "defaultValue": "Average",
            "allowedValues": [
                "Average",
                "Minimum",
                "Maximum",
                "Total"
            ],
            "metadata": {
                "description": "How the data that is collected should be combined over time."
            }
        },
        "windowSize": {
            "type": "string",
            "defaultValue": "PT5M",
            "metadata": {
                "description": "Period of time used to monitor alert activity based on the threshold. Must be between five minutes and one day. ISO 8601 duration format."
            }
        },
        "evaluationFrequency": {
            "type": "string",
            "defaultValue": "PT1M",
            "metadata": {
                "description": "how often the metric alert is evaluated represented in ISO 8601 duration format"
            }
        },
        "actionGroupId": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The ID of the action group that is triggered when the alert is activated or deactivated"
            }
        }
    },
    "variables": {
        "convertRuleTag": "hidden-link:/subscriptions/1234-56789-1234-567a/resourceGroups/resourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName",
        "convertRuleSourceWorkspace": {
            "SourceId": "/subscriptions/1234-56789-1234-567a/resourceGroups/resourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
        }
    },
    "resources": [
        {
            "name": "[parameters('convertRuleName')]",
            "type": "Microsoft.Insights/scheduledQueryRules",
            "apiVersion": "2018-04-16",
            "location": "[parameters('convertRuleRegion')]",
            "tags": {
                "[variables('convertRuleTag')]": "Resource"
            },
            "properties": {
                "description": "[parameters('convertRuleDescription')]",
                "enabled": "[parameters('convertRuleStatus')]",
                "source": {
                    "dataSourceId": "[variables('convertRuleSourceWorkspace').SourceId]"
                },
                "action": {
                    "odata.type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.Microsoft.AppInsights.Nexus.DataContracts.Resources.ScheduledQueryRules.LogToMetricAction",
                    "criteria": [{
                            "metricName": "[parameters('convertRuleMetric')]",
                            "dimensions": []
                        }
                    ]
                }
            }
        },
        {
            "name": "[parameters('alertName')]",
            "type": "Microsoft.Insights/metricAlerts",
            "location": "global",
            "apiVersion": "2018-03-01",
            "tags": {},
            "dependsOn":["[resourceId('Microsoft.Insights/scheduledQueryRules',parameters('convertRuleName'))]"],
            "properties": {
                "description": "[parameters('alertDescription')]",
                "severity": "[parameters('alertSeverity')]",
                "enabled": "[parameters('isEnabled')]",
                "scopes": ["[parameters('resourceId')]"],
                "evaluationFrequency":"[parameters('evaluationFrequency')]",
                "windowSize": "[parameters('windowSize')]",
                "criteria": {
                    "odata.type": "Microsoft.Azure.Monitor.SingleResourceMultipleMetricCriteria",
                    "allOf": [
                        {
                            "name" : "1st criterion",
                            "metricName": "[parameters('metricName')]",
                            "dimensions":[],
                            "operator": "[parameters('operator')]",
                            "threshold" : "[parameters('threshold')]",
                            "timeAggregation": "[parameters('timeAggregation')]"
                        }
                    ]
                },
                "actions": [
                    {
                        "actionGroupId": "[parameters('actionGroupId')]"
                    }
                ]
            }
        }
    ]
}
```

Supongamos que el código JSON anterior se guarda como metricfromLogsAlertStatic.json, a continuación, puede ir acompañado de un archivo JSON de parámetros para la creación basada en la plantilla de recursos. A continuación se incluye un ejemplo de archivo de JSON de parámetros:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "convertRuleName": {
            "value": "TestLogtoMetricRule" 
        },
        "convertRuleDescription": {
            "value": "Test rule to extract metrics from logs via template"
        },
        "convertRuleRegion": {
            "value": "West Central US"
        },
        "convertRuleStatus": {
            "value": "true"
        },
        "convertRuleMetric": {
            "value": "Average_% Idle Time"
        },
        "alertName": {
            "value": "TestMetricAlertonLog"
        },
        "alertDescription": {
            "value": "New multi-dimensional metric alert created via template"
        },
        "alertSeverity": {
            "value":3
        },
        "isEnabled": {
            "value": true
        },
        "resourceId": {
            "value": "/subscriptions/1234-56789-1234-567a/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
        },
        "metricName":{
            "value": "Average_% Idle Time"
        },
        "operator": {
            "value": "GreaterThan"
        },
        "threshold":{
            "value": "1"
        },
        "timeAggregation":{
            "value": "Average"
        },
        "actionGroupId": {
            "value": "/subscriptions/1234-56789-1234-567a/resourceGroups/myRG/providers/microsoft.insights/actionGroups/actionGroupName"
        }
    }
}
```

Suponiendo que el archivo de parámetros anterior se guarda como metricfromLogsAlertStatic.parameters.json, se puede crear la alerta de métrica para registros con la [plantilla de recursos para la creación en Azure Portal](../../azure-resource-manager/templates/deploy-portal.md).

Como alternativa, también se puede usar el siguiente comando de Azure Powershell:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "myRG" -TemplateFile metricfromLogsAlertStatic.json TemplateParameterFile metricfromLogsAlertStatic.parameters.json
```

O bien, implemente la plantilla de recursos con la CLI de Azure:

```azurecli
az group deployment create --resource-group myRG --template-file metricfromLogsAlertStatic.json --parameters @metricfromLogsAlertStatic.parameters.json
```

### <a name="metric-alerts-for-logs-with-dynamic-thresholds"></a>Alertas de métricas para registros con umbrales dinámicos

Para lograr el mismo, se puede utilizar el ejemplo de la plantilla de Azure Resource Manager a continuación, cuando la creación de la alerta de métrica de umbral dinámico dependa de la creación correcta de la regla para extraer las métricas de los registros a través de scheduledQueryRule.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "convertRuleName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the rule to convert log to metric"
            }
        },
        "convertRuleDescription": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Description for log converted to metric"
            }
        },
        "convertRuleRegion": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the region used by workspace"
            }
        },
        "convertRuleStatus": {
            "type": "string",
            "defaultValue": "true",
            "metadata": {
                "description": "Specifies whether the log conversion rule is enabled"
            }
        },
        "convertRuleMetric": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the metric once extraction done from logs."
            }
        },
        "alertName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the alert"
            }
        },
        "alertDescription": {
            "type": "string",
            "defaultValue": "This is a metric alert",
            "metadata": {
                "description": "Description of alert"
            }
        },
        "alertSeverity": {
            "type": "int",
            "defaultValue": 3,
            "allowedValues": [
                0,
                1,
                2,
                3,
                4
            ],
            "metadata": {
                "description": "Severity of alert {0,1,2,3,4}"
            }
        },
        "isEnabled": {
            "type": "bool",
            "defaultValue": true,
            "metadata": {
                "description": "Specifies whether the alert is enabled"
            }
        },
        "resourceId": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Full Resource ID of the resource emitting the metric that will be used for the comparison. For example: /subscriptions/00000000-0000-0000-0000-0000-00000000/resourceGroups/ResourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
            }
        },
        "metricName": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "Name of the metric used in the comparison to activate the alert."
            }
        },
        "operator": {
            "type": "string",
            "defaultValue": "GreaterOrLessThan",
            "allowedValues": [
                "GreaterThan",
                "LessThan",
                "GreaterOrLessThan"
            ],
            "metadata": {
                "description": "Operator comparing the current value with the threshold value."
            }
        },
        "alertSensitivity": {
            "type": "string",
            "defaultValue": "Medium",
            "allowedValues": [
                "High",
                "Medium",
                "Low"
            ],
            "metadata": {
                "description": "Tunes how 'noisy' the Dynamic Thresholds alerts will be: 'High' will result in more alerts while 'Low' will result in fewer alerts."
            }
        },
        "numberOfEvaluationPeriods": {
            "type": "string",
            "defaultValue": "4",
            "metadata": {
                "description": "The number of periods to check in the alert evaluation."
            }
        },
        "minFailingPeriodsToAlert": {
            "type": "string",
            "defaultValue": "3",
            "metadata": {
                "description": "The number of unhealthy periods to alert on (must be lower or equal to numberOfEvaluationPeriods)."
            }
        },
        "timeAggregation": {
            "type": "string",
            "defaultValue": "Average",
            "allowedValues": [
                "Average",
                "Minimum",
                "Maximum",
                "Total"
            ],
            "metadata": {
                "description": "How the data that is collected should be combined over time."
            }
        },
        "windowSize": {
            "type": "string",
            "defaultValue": "PT5M",
            "metadata": {
                "description": "Period of time used to monitor alert activity based on the threshold. Must be between five minutes and one day. ISO 8601 duration format."
            }
        },
        "evaluationFrequency": {
            "type": "string",
            "defaultValue": "PT1M",
            "metadata": {
                "description": "how often the metric alert is evaluated represented in ISO 8601 duration format"
            }
        },
        "actionGroupId": {
            "type": "string",
            "defaultValue": "",
            "metadata": {
                "description": "The ID of the action group that is triggered when the alert is activated or deactivated"
            }
        }
    },
    "variables": {
        "convertRuleTag": "hidden-link:/subscriptions/1234-56789-1234-567a/resourceGroups/resourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName",
        "convertRuleSourceWorkspace": {
            "SourceId": "/subscriptions/1234-56789-1234-567a/resourceGroups/resourceGroupName/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
        }
    },
    "resources": [
        {
            "name": "[parameters('convertRuleName')]",
            "type": "Microsoft.Insights/scheduledQueryRules",
            "apiVersion": "2018-04-16",
            "location": "[parameters('convertRuleRegion')]",
            "tags": {
                "[variables('convertRuleTag')]": "Resource"
            },
            "properties": {
                "description": "[parameters('convertRuleDescription')]",
                "enabled": "[parameters('convertRuleStatus')]",
                "source": {
                    "dataSourceId": "[variables('convertRuleSourceWorkspace').SourceId]"
                },
                "action": {
                    "odata.type": "Microsoft.WindowsAzure.Management.Monitoring.Alerts.Models.Microsoft.AppInsights.Nexus.DataContracts.Resources.ScheduledQueryRules.LogToMetricAction",
                    "criteria": [{
                            "metricName": "[parameters('convertRuleMetric')]",
                            "dimensions": []
                        }
                    ]
                }
            }
        },
        {
            "name": "[parameters('alertName')]",
            "type": "Microsoft.Insights/metricAlerts",
            "location": "global",
            "apiVersion": "2018-03-01",
            "tags": {},
            "dependsOn":["[resourceId('Microsoft.Insights/scheduledQueryRules',parameters('convertRuleName'))]"],
            "properties": {
                "description": "[parameters('alertDescription')]",
                "severity": "[parameters('alertSeverity')]",
                "enabled": "[parameters('isEnabled')]",
                "scopes": ["[parameters('resourceId')]"],
                "evaluationFrequency":"[parameters('evaluationFrequency')]",
                "windowSize": "[parameters('windowSize')]",
                "criteria": {
                    "odata.type": "Microsoft.Azure.Monitor.MultipleResourceMultipleMetricCriteria",
                    "allOf": [
                        {
                            "criterionType": "DynamicThresholdCriterion",
                            "name" : "1st criterion",
                            "metricName": "[parameters('metricName')]",
                            "dimensions":[],
                            "operator": "[parameters('operator')]",
                            "alertSensitivity": "[parameters('alertSensitivity')]",
                            "failingPeriods": {
                                "numberOfEvaluationPeriods": "[parameters('numberOfEvaluationPeriods')]",
                                "minFailingPeriodsToAlert": "[parameters('minFailingPeriodsToAlert')]"
                            },
                            "timeAggregation": "[parameters('timeAggregation')]"
                        }
                    ]
                },
                "actions": [
                    {
                        "actionGroupId": "[parameters('actionGroupId')]"
                    }
                ]
            }
        }
    ]
}
```

Supongamos que el código JSON anterior se guarda como metricfromLogsAlertDynamic.json, a continuación, puede ir acompañado de un archivo JSON de parámetros para la creación basada en la plantilla de recursos. A continuación se incluye un ejemplo de archivo de JSON de parámetros:

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "convertRuleName": {
            "value": "TestLogtoMetricRule"
        },
        "convertRuleDescription": {
            "value": "Test rule to extract metrics from logs via template"
        },
        "convertRuleRegion": {
            "value": "West Central US"
        },
        "convertRuleStatus": {
            "value": "true"
        },
        "convertRuleMetric": {
            "value": "Average_% Idle Time"
        },
        "alertName": {
            "value": "TestMetricAlertonLog"
        },
        "alertDescription": {
            "value": "New multi-dimensional metric alert created via template"
        },
        "alertSeverity": {
            "value":3
        },
        "isEnabled": {
            "value": true
        },
        "resourceId": {
            "value": "/subscriptions/1234-56789-1234-567a/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/workspaceName"
        },
        "metricName":{
            "value": "Average_% Idle Time"
        },
        "operator": {
            "value": "GreaterOrLessThan"
          },
          "alertSensitivity": {
              "value": "Medium"
          },
          "numberOfEvaluationPeriods": {
              "value": "4"
          },
          "minFailingPeriodsToAlert": {
              "value": "3"
          },
        "timeAggregation":{
            "value": "Average"
        },
        "actionGroupId": {
            "value": "/subscriptions/1234-56789-1234-567a/resourceGroups/myRG/providers/microsoft.insights/actionGroups/actionGroupName"
        }
    }
}
```

Suponiendo que el archivo de parámetros anterior se guarda como metricfromLogsAlertDynamic.parameters.json, se puede crear la alerta de métrica para registros con la [plantilla de recursos para la creación en Azure Portal](../../azure-resource-manager/templates/deploy-portal.md).

Como alternativa, también se puede usar el siguiente comando de Azure Powershell:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "myRG" -TemplateFile metricfromLogsAlertDynamic.json TemplateParameterFile metricfromLogsAlertDynamic.parameters.json
```

O bien, implemente la plantilla de recursos con la CLI de Azure:

```azurecli
az group deployment create --resource-group myRG --template-file metricfromLogsAlertDynamic.json --parameters @metricfromLogsAlertDynamic.parameters.json
```

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre las [alertas de métricas](alerts-metric.md).
- Más información sobre las [alertas de registro en Azure](./alerts-unified-log.md).
- Más información sobre las [alertas en Azure](alerts-overview.md).

