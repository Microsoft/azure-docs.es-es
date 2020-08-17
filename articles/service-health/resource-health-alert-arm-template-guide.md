---
title: Plantilla para crear alertas de Resource Health
description: Creación de alertas mediante programación que notifiquen cuándo dejan de estar disponibles los recursos de Azure.
ms.topic: conceptual
ms.date: 9/4/2018
ms.openlocfilehash: 399c37f123d06e251514d0cafc3c1c4918ce3edd
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87037940"
---
# <a name="configure-resource-health-alerts-using-resource-manager-templates"></a>Configuración de alertas de estado de los recursos con plantillas de Resource Manager

En este artículo se muestra cómo crear alertas del registro de actividad de Resource Health mediante programación con plantillas de Azure Resource Manager y Azure PowerShell.

Azure Resource Health le mantiene informado sobre el estado actual y pasado de sus recursos de Azure. Además, le notifica casi en tiempo real de los cambios de estado en estos recursos. La creación y la personalización de alertas mediante programación en Resource Health se puede realizar en bloque.

> [!NOTE]
> Las alertas de Resource Health están actualmente en versión preliminar.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para seguir las instrucciones que aparecen en esta página, necesita de antemano algunas cosas:

1. Debe instalar el [módulo de Azure PowerShell](/powershell/azure/install-az-ps)
2. Debe [crear o volver a usar un grupo de acciones](../azure-monitor/platform/action-groups.md) configurado para recibir notificaciones

## <a name="instructions"></a>Instructions
1. Con PowerShell, inicie sesión en su cuenta de Azure y seleccione la suscripción con la que desee interactuar

    ```azurepowershell
    Login-AzAccount
    Select-AzSubscription -Subscription <subscriptionId>
    ```

    > Puede usar `Get-AzSubscription` para enumerar las suscripciones a las que tiene acceso.

2. Busque y guarde el identificador completo del grupo de acciones de Azure Resource Manager

    ```azurepowershell
    (Get-AzActionGroup -ResourceGroupName <resourceGroup> -Name <actionGroup>).Id
    ```

3. Cree y guarde una plantilla de Resource Manager para las alertas de Resource Health como `resourcehealthalert.json` ([consulte los detalles a continuación](#resource-manager-template-options-for-resource-health-alerts))

4. Cree una nueva implementación de Azure Resource Manager con esta plantilla

    ```azurepowershell
    New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName <resourceGroup> -TemplateFile <path\to\resourcehealthalert.json>
    ```

5. Se le pedirá que escriba el nombre de la alerta y el identificador de recurso del grupo de acciones que copió anteriormente:

    ```azurepowershell
    Supply values for the following parameters:
    (Type !? for Help.)
    activityLogAlertName: <Alert Name>
    actionGroupResourceId: /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/microsoft.insights/actionGroups/<actionGroup>
    ```

6. Si todo fue bien, recibirá una confirmación en PowerShell

    ```output
    DeploymentName          : ExampleDeployment
    ResourceGroupName       : <resourceGroup>
    ProvisioningState       : Succeeded
    Timestamp               : 11/8/2017 2:32:00 AM
    Mode                    : Incremental
    TemplateLink            :
    Parameters              :
                            Name                     Type       Value
                            ===============          =========  ==========
                            activityLogAlertName     String     <Alert Name>
                            activityLogAlertEnabled  Bool       True
                            actionGroupResourceId    String     /...

    Outputs                 :
    DeploymentDebugLogLevel :
    ```

Tenga en cuenta que si planea automatizar completamente este proceso, basta con editar la plantilla de Resource Manager para que no solicite los valores del paso 5.

## <a name="resource-manager-template-options-for-resource-health-alerts"></a>Opciones de la plantilla de Resource Manager para las alertas de Resource Health

Para crear alertas de Resource Health puede usar esta plantilla base como punto de partida. Esta plantilla funcionará como se escriba y le permitirá recibir alertas de todos los eventos de estado de recurso que se activen a partir de ese momento en los recursos de una suscripción.

> En la parte inferior de este artículo hemos incluido también una plantilla de alerta más compleja que debe aumentar la relación señal/ruido de las alertas de Resource Health en comparación con esta plantilla.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "activityLogAlertName": {
      "type": "string",
      "metadata": {
        "description": "Unique name (within the Resource Group) for the Activity log alert."
      }
    },
    "actionGroupResourceId": {
      "type": "string",
      "metadata": {
        "description": "Resource Id for the Action group."
      }
    }
  },
  "resources": [   
    {
      "type": "Microsoft.Insights/activityLogAlerts",
      "apiVersion": "2017-04-01",
      "name": "[parameters('activityLogAlertName')]",      
      "location": "Global",
      "properties": {
        "enabled": true,
        "scopes": [
            "[subscription().id]"
        ],        
        "condition": {
          "allOf": [
            {
              "field": "category",
              "equals": "ResourceHealth"
            },
            {
              "field": "status",
              "equals": "Active"
            }
          ]
        },
        "actions": {
          "actionGroups":
          [
            {
              "actionGroupId": "[parameters('actionGroupResourceId')]"
            }
          ]
        }
      }
    }
  ]
}
```

Sin embargo, una alerta amplia como esta no suele ser recomendable. Aprenda a reducir el ámbito para que la alerta se centre en los eventos que nos importan a continuación.

### <a name="adjusting-the-alert-scope"></a>Ajuste del ámbito de alerta

Las alertas de Resource Health se pueden configurar para supervisar eventos en tres ámbitos distintos:

 * A nivel de suscripción
 * A nivel de grupo de recursos
 * A nivel de recurso

La plantilla de alerta se configura a nivel de suscripción, pero si desea configurarla para que solo se notifiquen determinados recursos o recursos de un determinado grupo, basta con modificar la sección `scopes` en la anterior plantilla.

Para un ámbito a nivel de grupo de recursos, la sección de ámbito tiene este aspecto:
```json
"scopes": [
    "/subscriptions/<subscription id>/resourcegroups/<resource group>"
],
```

Para un ámbito a nivel de recurso, la sección de ámbito tiene este aspecto:

```json
"scopes": [
    "/subscriptions/<subscription id>/resourcegroups/<resource group>/providers/<resource>"
],
```

Por ejemplo: `"/subscriptions/d37urb3e-ed41-4670-9c19-02a1d2808ff9/resourcegroups/myRG/providers/microsoft.compute/virtualmachines/myVm"`

> Puede ir a Azure Portal y observar la dirección URL del recurso de Azure para obtener esta cadena.

### <a name="adjusting-the-resource-types-which-alert-you"></a>Ajuste de los tipos de recursos con alertas

Las alertas a nivel de suscripción o de grupo de recursos tienen distintos tipos de recursos. Si desea limitar las alertas a las que procedan de un determinado subconjunto de tipos de recursos, puede definirlo en la sección `condition` de la plantilla de este modo:

```json
"condition": {
    "allOf": [
        ...,
        {
            "anyOf": [
                {
                    "field": "resourceType",
                    "equals": "MICROSOFT.COMPUTE/VIRTUALMACHINES",
                    "containsAny": null
                },
                {
                    "field": "resourceType",
                    "equals": "MICROSOFT.STORAGE/STORAGEACCOUNTS",
                    "containsAny": null
                },
                ...
            ]
        }
    ]
},
```

Aquí usamos el contenedor `anyOf` para permitir que la alerta de estado de recurso coincida con cualquiera de las condiciones especificadas para las alertas se centren en determinados tipos de recursos.

### <a name="adjusting-the-resource-health-events-that-alert-you"></a>Ajuste de los eventos de Resource Health con alerta
Cuando se produce un evento de estado en los recursos, pueden pasar por una serie de fases que representan el estado del evento: `Active`, `In Progress`, `Updated` y `Resolved`.

Quizá solo desee recibir notificaciones en caso de que el estado del recurso sea incorrecto, para lo que querrá configurar la alerta para notificar solo cuando `status` sea `Active`. Sin embargo si desea recibir una notificación también en las otras fases, puede agregar esos detalles de este modo:

```json
"condition": {
    "allOf": [
        ...,
        {
            "anyOf": [
                {
                    "field": "status",
                    "equals": "Active"
                },
                {
                    "field": "status",
                    "equals": "In Progress"
                },
                {
                    "field": "status",
                    "equals": "Resolved"
                },
                {
                    "field": "status",
                    "equals": "Updated"
                }
            ]
        }
    ]
}
```

Si desea recibir una notificación para las cuatro fases de los eventos de estado, elimine esta condición y la alerta le notificará independientemente de la propiedad `status`.

> [!NOTE]
> Cada sección "anyOf" debe contener solo un valor de tipo de campo.

### <a name="adjusting-the-resource-health-alerts-to-avoid-unknown-events"></a>Ajuste de las alertas de Resource Health para evitar eventos desconocidos

Azure Resource Health puede notificar el estado más reciente de los recursos gracias a la constante supervisión mediante ejecutores de pruebas. Los estados de mantenimiento notificados pertinentes son: "Disponible", "No disponible" y "Degradado". Sin embargo, en situaciones en las que el ejecutor y el recurso de Azure no se pueden comunicar, se notifica un estado "Desconocido" para el recurso y que se considera un evento de estado "Activo".

No obstante, cuando un recurso se notifica como "Desconocido", es probable que su estado no haya cambiado desde el último informe preciso. Si desea eliminar las alertas de eventos desconocidos, puede especificar esa lógica en la plantilla:

```json
"condition": {
    "allOf": [
        ...,
        {
            "anyOf": [
                {
                    "field": "properties.currentHealthStatus",
                    "equals": "Available",
                    "containsAny": null
                },
                {
                    "field": "properties.currentHealthStatus",
                    "equals": "Unavailable",
                    "containsAny": null
                },
                {
                    "field": "properties.currentHealthStatus",
                    "equals": "Degraded",
                    "containsAny": null
                }
            ]
        },
        {
            "anyOf": [
                {
                    "field": "properties.previousHealthStatus",
                    "equals": "Available",
                    "containsAny": null
                },
                {
                    "field": "properties.previousHealthStatus",
                    "equals": "Unavailable",
                    "containsAny": null
                },
                {
                    "field": "properties.previousHealthStatus",
                    "equals": "Degraded",
                    "containsAny": null
                }
            ]
        },
    ]
},
```

En este ejemplo, solo se notifican los eventos en los que el estado actual y anterior no son desconocidos. Este cambio puede resultar útil si las alertas se envían directamente al correo electrónico o teléfono móvil. 

Tenga en cuenta que es posible que las propiedades currentHealthStatus y previousHealthStatus sean null en algunos eventos. Por ejemplo, cuando se produce un evento Updated, es probable que el estado de mantenimiento del recurso no haya cambiado desde el último informe y solo está disponible esa información adicional del evento (por ejemplo, la causa). Por lo tanto, mediante la cláusula anterior puede provocar que algunas alertas no se desencadenen, ya que los valores de properties.currentHealthStatus y properties.previousHealthStatus se establecerán en null.

### <a name="adjusting-the-alert-to-avoid-user-initiated-events"></a>Ajuste de la alerta para evitar eventos iniciados por el usuario

Los eventos de Resource Health se pueden desencadenar mediante eventos iniciados por la plataforma o por el usuario. Puede que tenga sentido solo enviar una notificación cuando el evento lo genere la plataforma Azure.

Es fácil de configurar la alerta para filtrar solo estos tipos de eventos:

```json
"condition": {
    "allOf": [
        ...,
        {
            "field": "properties.cause",
            "equals": "PlatformInitiated",
            "containsAny": null
        }
    ]
}
```
Tenga en cuenta que es posible que el campo de la causa sea null en algunos eventos. Es decir, una transición de estado tiene lugar (por ejemplo, pasa de disponible a no disponible) y el evento se registra inmediatamente a fin de evitar que la notificación se retrase. Por lo tanto, mediante la cláusula anterior puede provocar que una alerta no se desencadene, ya que el valor de la propiedad properties.clause se establecerá en null.

## <a name="complete-resource-health-alert-template"></a>Completar la plantilla de alerta de Resource Health

Con los distintos ajustes que se describen en la sección anterior, a continuación se muestra una plantilla de muestra configurada para maximizar la relación señal/ruido. Tenga en cuenta las observaciones que se han indicado anteriormente donde los valores de las propiedades currentHealthStatus, previousHealthStatus y cause pueden ser null en algunos eventos.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "activityLogAlertName": {
            "type": "string",
            "metadata": {
                "description": "Unique name (within the Resource Group) for the Activity log alert."
            }
        },
        "actionGroupResourceId": {
            "type": "string",
            "metadata": {
                "description": "Resource Id for the Action group."
            }
        }
    },
    "resources": [
        {
            "type": "Microsoft.Insights/activityLogAlerts",
            "apiVersion": "2017-04-01",
            "name": "[parameters('activityLogAlertName')]",
            "location": "Global",
            "properties": {
                "enabled": true,
                "scopes": [
                    "[subscription().id]"
                ],
                "condition": {
                    "allOf": [
                        {
                            "field": "category",
                            "equals": "ResourceHealth",
                            "containsAny": null
                        },
                        {
                            "anyOf": [
                                {
                                    "field": "properties.currentHealthStatus",
                                    "equals": "Available",
                                    "containsAny": null
                                },
                                {
                                    "field": "properties.currentHealthStatus",
                                    "equals": "Unavailable",
                                    "containsAny": null
                                },
                                {
                                    "field": "properties.currentHealthStatus",
                                    "equals": "Degraded",
                                    "containsAny": null
                                }
                            ]
                        },
                        {
                            "anyOf": [
                                {
                                    "field": "properties.previousHealthStatus",
                                    "equals": "Available",
                                    "containsAny": null
                                },
                                {
                                    "field": "properties.previousHealthStatus",
                                    "equals": "Unavailable",
                                    "containsAny": null
                                },
                                {
                                    "field": "properties.previousHealthStatus",
                                    "equals": "Degraded",
                                    "containsAny": null
                                }
                            ]
                        },
                        {
                            "anyOf": [
                                {
                                    "field": "properties.cause",
                                    "equals": "PlatformInitiated",
                                    "containsAny": null
                                }
                            ]
                        },
                        {
                            "anyOf": [
                                {
                                    "field": "status",
                                    "equals": "Active",
                                    "containsAny": null
                                },
                                {
                                    "field": "status",
                                    "equals": "Resolved",
                                    "containsAny": null
                                },
                                {
                                    "field": "status",
                                    "equals": "In Progress",
                                    "containsAny": null
                                },
                                {
                                    "field": "status",
                                    "equals": "Updated",
                                    "containsAny": null
                                }
                            ]
                        }
                    ]
                },
                "actions": {
                    "actionGroups": [
                        {
                            "actionGroupId": "[parameters('actionGroupResourceId')]"
                        }
                    ]
                }
            }
        }
    ]
}
```

Sin embargo, usted conoce mejor las configuraciones que necesita; use las herramientas que le hemos mostrado en esta documentación para la personalización.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre Resource Health:
-  [Información general sobre Azure Resource Health](Resource-health-overview.md)
-  [Tipos de recursos y comprobaciones de mantenimiento disponibles a través de Azure Resource Health](resource-health-checks-resource-types.md)


Creación de alertas de Service Health:
-  [Configuración de alertas de Service Health](./alerts-activity-log-service-notifications-portal.md) 
-  [Esquema de eventos del registro de actividad de Azure](../azure-monitor/platform/activity-log-schema.md)
