---
title: Creación de alertas de Azure Advisor para nuevas recomendaciones mediante una plantilla de Resource Manager
description: Creación de alertas de Azure Advisor para nuevas recomendaciones
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 06/29/2020
ms.openlocfilehash: 2becfbbc63beb6451e5e877c5a60553d98650494
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87057825"
---
# <a name="quickstart-create-azure-advisor-alerts-on-new-recommendations-using-an-arm-template"></a>Inicio rápido: Creación de alertas de Azure Advisor para nuevas recomendaciones mediante una plantilla de Resource Manager

En este artículo se muestra cómo configurar una alerta para nuevas recomendaciones de Azure Advisor mediante una plantilla de Azure Resource Manager.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Cada vez que Azure Advisor detecta una nueva recomendación para uno de los recursos, se almacena un evento en el [registro de actividad de Azure](../azure-monitor/platform/platform-logs-overview.md). Puede configurar alertas para estos eventos desde Azure Advisor creando alertas específicas para la recomendación. Puede seleccionar una suscripción y, si lo desea, un grupo de recursos para especificar los recursos sobre los que desea recibir alertas.

También puede determinar los tipos de recomendaciones mediante el uso de estas propiedades:

- Category
- Nivel de impacto
- Tipo de recomendación

También puede configurar la acción que tendrá lugar cuando se desencadene una alerta si:  

- Selecciona un grupo de acciones existente.
- Crea un nuevo grupo de acciones.

Para más información sobre los grupos de acciones, consulte [Creación y administración de grupos de acciones](../azure-monitor/platform/action-groups.md).

> [!NOTE]
> Las alertas de Advisor solo están disponibles para las recomendaciones de alta disponibilidad, rendimiento y costo. No se admiten recomendaciones de seguridad.

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- Para ejecutar los comandos desde la máquina local, instale la CLI de Azure o los módulos de Azure PowerShell. Para más información, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli) e [Instalar Azure Powershell](/powershell/azure/install-az-ps).

## <a name="review-the-template"></a>Revisión de la plantilla

En el siguiente ejemplo se crea un grupo de acciones con un destino de correo electrónico y se habilitan todas las notificaciones de estado de servicio de la suscripción de destino. Guarde esta plantilla como *CreateAdvisorAlert.json*.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "actionGroups_name": {
      "defaultValue": "advisorAlert",
      "type": "string"
    },
    "activityLogAlerts_name": {
      "defaultValue": "AdvisorAlertsTest",
      "type": "string"
    },
    "emailAddress": {
      "defaultValue": "<email address>",
      "type": "string"
    }
  },
  "variables": {
    "alertScope": "[concat('/','subscriptions','/',subscription().subscriptionId)]"
  },
  "resources": [
    {
      "comments": "Action Group",
      "type": "microsoft.insights/actionGroups",
      "name": "[parameters('actionGroups_name')]",
      "apiVersion": "2019-06-01",
      "location": "Global",
      "tags": {},
      "scale": null,
      "properties": {
        "groupShortName": "[parameters('actionGroups_name')]",
        "enabled": true,
        "emailReceivers": [
          {
            "name": "[parameters('actionGroups_name')]",
            "emailAddress": "[parameters('emailAddress')]"
          }
        ],
        "smsReceivers": [],
        "webhookReceivers": []
      },
      "dependsOn": []
    },
    {
      "comments": "Azure Advisor Activity Log Alert",
      "type": "microsoft.insights/activityLogAlerts",
      "name": "[parameters('activityLogAlerts_name')]",
      "apiVersion": "2017-04-01",
      "location": "Global",
      "tags": {},
      "scale": null,
      "properties": {
        "scopes": [
          "[variables('alertScope')]"
        ],
        "condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "Recommendation"
            },
            {
              "field": "properties.recommendationCategory",
              "equals": "Cost"
            },
            {
              "field": "properties.recommendationImpact",
              "equals": "Medium"
            },
            {
              "field": "operationName",
              "equals": "Microsoft.Advisor/recommendations/available/action"
            }
          ]
        },
        "actions": {
          "actionGroups": [
            {
              "actionGroupId": "[resourceId('microsoft.insights/actionGroups', parameters('actionGroups_name'))]",
              "webhookProperties": {}
            }
          ]
        },
        "enabled": true,
        "description": ""
      },
      "dependsOn": [
        "[resourceId('microsoft.insights/actionGroups', parameters('actionGroups_name'))]"
      ]
    }
  ]
}
```

La plantilla define dos recursos:

- [Microsoft.Insights/actionGroups](/azure/templates/microsoft.insights/actiongroups)
- [Microsoft.Insights/activityLogAlerts](/azure/templates/microsoft.insights/activityLogAlerts)

## <a name="deploy-the-template"></a>Implementación de la plantilla

Implemente la plantilla mediante cualquier método estándar de [implementación de una plantilla de Resource Manager](../azure-resource-manager/templates/deploy-portal.md) como en los ejemplos siguientes con la CLI y PowerShell. Reemplace los valores de ejemplo del **grupo de recursos** y **dirección de correo electrónico** por los valores adecuados para su entorno. El nombre del área de trabajo debe ser único entre todas las suscripciones de Azure.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az login
az deployment group create --name CreateAdvisorAlert --resource-group my-resource-group --template-file CreateAdvisorAlert.json --parameters emailAddress='user@contoso.com'
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName my-subscription
New-AzResourceGroupDeployment -Name CreateAdvisorAlert -ResourceGroupName my-resource-group -TemplateFile CreateAdvisorAlert.json -emailAddress user@contoso.com
```

---

## <a name="validate-the-deployment"></a>Validación de la implementación

Para comprobar que se ha creado el área de trabajo, utilice uno de los comandos siguientes. Reemplace los valores de ejemplo del **grupo de recursos** por los valores que usó anteriormente.

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az monitor activity-log alert show --resource-group my-resource-group --name AdvisorAlertsTest
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Get-AzActivityLogAlert -ResourceGroupName my-resource-group -Name AdvisorAlertsTest
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Si planea seguir trabajando en otros inicios rápidos y tutoriales, considere la posibilidad de dejar estos recursos activos. Cuando ya no lo necesite, elimine el grupo de recursos; de este modo, se eliminarán también la regla de alertas y los recursos relacionados. Para eliminar el grupo de recursos mediante la CLI de Azure o Azure PowerShell

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli
az group delete --name my-resource-group
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```powershell
Remove-AzResourceGroup -Name my-resource-group
```

---

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [introducción a las alertas del registro de actividad](../azure-monitor/platform/alerts-overview.md) y aprenda cómo puede recibir alertas.
- Más información sobre los [grupos de acciones](../azure-monitor/platform/action-groups.md).
