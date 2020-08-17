---
title: Creación de la configuración de diagnóstico en Azure con una plantilla de Resource Manager
description: Cree una configuración de diagnóstico con una plantilla de Resource Manager para reenviar los registros de plataforma de Azure a los registros de Azure Monitor, a Azure Storage o a Azure Event Hubs.
author: bwren
services: azure-monitor
ms.topic: conceptual
ms.date: 12/13/2019
ms.author: bwren
ms.subservice: ''
ms.openlocfilehash: 73f2b3ea90cc94fa3411552c7b812fe53eb4dbbb
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87008105"
---
# <a name="create-diagnostic-setting-in-azure-using-a-resource-manager-template"></a>Creación de la configuración de diagnóstico en Azure con una plantilla de Resource Manager
[Configuración de diagnóstico](diagnostic-settings.md) en Azure Monitor especifica dónde se envían los [Registros de plataforma](platform-logs-overview.md) recopilados por los recursos de Azure y la plataforma de Azure de los que dependen. En este artículo se proporcionan detalles y ejemplos sobre el uso de una [plantilla de Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md) a fin de crear y definir la configuración de diagnóstico para recopilar los registros de plataforma en distintos destinos.

> [!NOTE]
> Dado que no puede [crear una configuración de diagnóstico](diagnostic-settings.md) para el registro de actividad de Azure mediante PowerShell o la CLI, como la configuración de diagnóstico de otros recursos de Azure, cree una plantilla de Resource Manager para el registro de actividad mediante la información de este artículo e implemente la plantilla mediante PowerShell o la CLI.

## <a name="deployment-methods"></a>Métodos de implementación
Puede implementar las plantillas de Resource Manager mediante cualquier método válido, como PowerShell y la CLI. La configuración de diagnóstico del registro de actividad se debe implementar en una suscripción mediante `az deployment create` para la CLI o `New-AzDeployment` para PowerShell. La configuración de diagnóstico de los registros de recursos se debe implementar en un grupo de recursos mediante `az group deployment create` para la CLI o `New-AzResourceGroupDeployment` para PowerShell.

Consulte [Implementación de recursos con las plantillas de Resource Manager y Azure PowerShell](../../azure-resource-manager/templates/deploy-powershell.md) e [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../../azure-resource-manager/templates/deploy-cli.md) para ver información más detallada. 





## <a name="resource-logs"></a>Registros del recurso
En el caso de los registros de recursos, agregue un recurso de tipo `<resource namespace>/providers/diagnosticSettings` a la plantilla. La sección de propiedades tiene el formato que se describe en [Configuración de diagnóstico: creación o actualización](/rest/api/monitor/diagnosticsettings/createorupdate). Proporcione un valor de `category` en la sección `logs` a cada una de las categorías válidas para el recurso que desea recopilar. Agregue la propiedad `metrics` para recopilar métricas de recurso en los mismos destinos si el [recurso admite métricas](metrics-supported.md).

A continuación, se encuentra una plantilla que recopila una categoría de registro de recursos para un recurso determinado en un área de trabajo de Log Analytics, una cuenta de almacenamiento y un centro de eventos.

```json
"resources": [
  {
    "type": "/<resource namespace>/providers/diagnosticSettings",
    "name": "[concat(parameters('resourceName'),'/microsoft.insights/', parameters('settingName'))]",
    "dependsOn": [
      "[<resource Id for which resource logs will be enabled>]"
    ],
    "apiVersion": "2017-05-01-preview",
    "properties": {
      "name": "[parameters('settingName')]",
      "storageAccountId": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]",
      "eventHubAuthorizationRuleId": "[parameters('eventHubAuthorizationRuleId')]",
      "eventHubName": "[parameters('eventHubName')]",
      "workspaceId": "[parameters('workspaceId')]",
      "logs": [
        {
          "category": "<category name>",
          "enabled": true
        }
      ],
      "metrics": [
        {
          "category": "AllMetrics",
          "enabled": true
        }
      ]
    }
  }
]
```



### <a name="example"></a>Ejemplo
A continuación, se ofrece un ejemplo que crea una configuración de diagnóstico para una configuración de escalabilidad automática que permite la transmisión de registros de recursos a un centro de eventos, una cuenta de almacenamiento y un área de trabajo de Log Analytics.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "autoscaleSettingName": {
            "type": "string",
            "metadata": {
                "description": "The name of the autoscale setting"
            }
        },
        "settingName": {
            "type": "string",
            "metadata": {
                "description": "The name of the diagnostic setting"
            }
        },
        "workspaceId": {
            "type": "string",
            "metadata": {
                "description": "ResourceIDl of the Log Analytics workspace in which resource logs should be saved."
            }
        },
        "storageAccountId": {
            "type": "string",
            "metadata": {
              "description": "ResourceID of the Storage Account in which resource logs should be saved."
            }
        },
        "eventHubAuthorizationRuleId": {
            "type": "string",
            "metadata": {
              "description": "Resource ID of the event hub authorization rule for the Event Hubs namespace in which the event hub should be created or streamed to."
            }
        },
        "eventHubName": {
            "type": "string",
            "metadata": {
                "description": "Optional. Name of the event hub within the namespace to which logs are streamed. Without this, an event hub is created for each log category."
            }
        }
    },
    "variables": {},
    "resources": [
    {
      "type": "microsoft.insights/autoscalesettings/providers/diagnosticSettings",
      "apiVersion": "2017-05-01-preview",
      "name": "[concat(parameters('autoscaleSettingName'),'/microsoft.insights/', parameters('settingName'))]",
      "dependsOn": [
        "[resourceId('Microsoft.Insights/autoscalesettings', parameters('autoscaleSettingName'))]"
      ],
      "properties": {
        "workspaceId": "[parameters('workspaceId')]",
        "storageAccountId": "[parameters('storageAccountId')]",
        "eventHubAuthorizationRuleId": "[parameters('eventHubAuthorizationRuleId')]",
        "eventHubName": "[parameters('eventHubName')]",
        "logs": [
          {
            "category": "AutoscaleScaleActions",
            "enabled": true
          },
          {
            "category": "AutoscaleEvaluations",
            "enabled": true
          }
        ]
      }
    }
  ]
}
```

## <a name="activity-log"></a>Registro de actividades
Para el registro de actividad de Azure, agregue un recurso de tipo `Microsoft.Insights/diagnosticSettings`. Las categorías disponibles se enumeran en [Categorías del registro de actividad](./activity-log.md#view-the-activity-log). A continuación, se muestra una plantilla que recopila todas las categorías de registro de actividades en un área de trabajo de Log Analytics, una cuenta de almacenamiento y un centro de eventos.


```json
{
    "$schema": "https://schema.management.azure.com/schemas/2018-05-01/subscriptionDeploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "settingName": {
            "type": "string",
            "metadata": {
                "description": "The name of the diagnostic setting"
            }
        },
        "workspaceId": {
            "type": "string",
            "metadata": {
                "description": "ResourceID of the Log Analytics workspace in which resource logs should be saved."
            }
        },
        "storageAccountId": {
            "type": "string",
            "metadata": {
              "description": "ResourceID of the Storage Account in which resource logs should be saved."
            }
        },
        "eventHubAuthorizationRuleId": {
            "type": "string",
            "metadata": {
              "description": "Resource ID of the event hub authorization rule for the Event Hubs namespace in which the event hub should be created or streamed to."
            }
        },
        "eventHubName": {
            "type": "string",
            "metadata": {
                "description": "Optional. Name of the event hub within the namespace to which logs are streamed. Without this, an event hub is created for each log category."
            }
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Insights/diagnosticSettings",
            "apiVersion": "2017-05-01-preview",
            "name": "[parameters('settingName')]",
            "location": "global",
            "properties": {
                "workspaceId": "[parameters('workspaceId')]",
                "storageAccountId": "[parameters('storageAccountId')]",
                "eventHubAuthorizationRuleId": "[parameters('eventHubAuthorizationRuleId')]",
                "eventHubName": "[parameters('eventHubName')]",
                "logs": [
                    {
                        "category": "Administrative",
                        "enabled": true
                    },
                    {
                        "category": "Security",
                        "enabled": true
                    },
                    {
                        "category": "ServiceHealth",
                        "enabled": true
                    },
                    {
                        "category": "Alert",
                        "enabled": true
                    },
                    {
                        "category": "Recommendation",
                        "enabled": true
                    },
                    {
                        "category": "Policy",
                        "enabled": true
                    },
                    {
                        "category": "Autoscale",
                        "enabled": true
                    },
                    {
                        "category": "ResourceHealth",
                        "enabled": true
                    }
                ]
            }
        }
    ]
}
```


## <a name="next-steps"></a>Pasos siguientes
* Más información sobre los [registros de plataforma en Azure](platform-logs-overview.md).
* Información sobre la [configuración de diagnóstico](diagnostic-settings.md).
