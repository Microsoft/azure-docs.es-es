---
title: Configuración de alertas de estado de los recursos de Azure con plantillas de Resource Manager | Microsoft Docs
description: Creación de alertas mediante programación que notifiquen cuándo dejan de estar disponibles los recursos de Azure.
author: stephbaron
ms.author: stbaron
ms.topic: conceptual
ms.service: service-health
ms.date: 9/4/2018
ms.openlocfilehash: 3d9a5ebb2e25cfbabf8cfdbd94c2d1d04ae1bbee
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65788470"
---
# <a name="configure-resource-health-alerts-using-resource-manager-templates"></a>Configuración de alertas de estado de los recursos con plantillas de Resource Manager

En este artículo se muestra cómo crear alertas del registro de actividad de Resource Health mediante programación con plantillas de Azure Resource Manager y Azure PowerShell.

Azure Resource Health le mantiene informado sobre el estado actual y pasado de sus recursos de Azure. Además, le notifica casi en tiempo real de los cambios de estado en estos recursos. La creación y la personalización de alertas mediante programación en Resource Health se puede realizar en bloque.

> [!NOTE]
> Alertas de estado de recursos están actualmente en versión preliminar.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

Para seguir las instrucciones que aparecen en esta página, necesita de antemano algunas cosas:

1. Deberá instalar el [módulo Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-Az-ps)
2. Debe [crear o volver a usar un grupo de acciones](../azure-monitor/platform/action-groups.md) configurado para recibir notificaciones

## <a name="instructions"></a>Instrucciones
1. Con PowerShell, inicie sesión en su cuenta de Azure y seleccione la suscripción con la que desee interactuar

        Login-AzAccount
        Select-AzSubscription -Subscription <subscriptionId>

    > Puede usar `Get-AzSubscription` para enumerar las suscripciones a las que tiene acceso.

2. Busque y guarde el identificador completo del grupo de acciones de Azure Resource Manager

        (Get-AzActionGroup -ResourceGroupName <resourceGroup> -Name <actionGroup>).Id

3. Cree y guarde una plantilla de Resource Manager para las alertas de Resource Health como `resourcehealthalert.json` ([consulte los detalles a continuación](#resource-manager-template-options-for-resource-health-alerts))

4. Cree una nueva implementación de Azure Resource Manager con esta plantilla

        New-AzResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName <resourceGroup> -TemplateFile <path\to\resourcehealthalert.json>

5. Se le pedirá que escriba el nombre de la alerta y el identificador de recurso del grupo de acciones que copió anteriormente:

        Supply values for the following parameters:
        (Type !? for Help.)
        activityLogAlertName: <Alert Name>
        actionGroupResourceId: /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/microsoft.insights/actionGroups/<actionGroup>

6. Si todo fue bien, recibirá una confirmación en PowerShell

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

Tenga en cuenta que si planea automatizar completamente este proceso, basta con editar la plantilla de Resource Manager para que no solicite los valores del paso 5.

## <a name="resource-manager-template-options-for-resource-health-alerts"></a>Opciones de plantilla del Administrador de recursos para las alertas de estado de los recursos

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
                    "equals": "Microsoft.Compute/virtualMachines",
                    "containsAny": null
                },
                {
                    "field": "resourceType",
                    "equals": "Microsoft.Storage/storageAccounts",
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
Cuando se produce un evento de estado en los recursos, pueden pasar por una serie de fases que representan el estado del evento: `Active`, `InProgress`, `Updated` y `Resolved`.

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
                    "equals": "InProgress"
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

Tenga en cuenta que es posible que las propiedades currentHealthStatus y previousHealthStatus para ser null, en algunos eventos. Por ejemplo, cuando un evento Updated se produce es probable que el estado de mantenimiento del recurso no ha cambiado desde el último informe, solo está disponible esa información adicional del evento (por ejemplo, producir). Por lo tanto, mediante la cláusula anterior puede generar algunas alertas no se desencadene, ya los valores properties.currentHealthStatus y properties.previousHealthStatus se establecerá en null.

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
Tenga en cuenta que es posible que el campo causa sea null, en algunos eventos. Es decir, una transición de estado tiene lugar (por ejemplo, disponible como no disponible) y se registra el evento inmediatamente evitar que la notificación se retrasa. Por lo tanto, mediante la cláusula anterior puede generar una alerta no se desencadene, ya se establecerá el valor de propiedad properties.clause en null.

## <a name="complete-resource-health-alert-template"></a>Plantilla de alerta de estado de los recursos completa

Aquí se usa los ajustes diferentes que se describe en la sección anterior, es una plantilla de ejemplo que está configurada para maximizar la señal en proporción de ruido. Tenga en cuenta las observaciones que se ha indicado anteriormente donde los currentHealthStatus, previousHealthStatus y valores de propiedad de la causa pueden ser nulos en algunos eventos.

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
                                    "equals": "InProgress",
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
-  [Configuración de alertas de Service Health](../azure-monitor/platform/alerts-activity-log-service-notifications.md) 
