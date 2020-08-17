---
title: 'Configuración de reglas de detección inteligente: Azure Application Insights'
description: Automatización de la administración y configuración de reglas de detección inteligente de Azure Application Insights con plantillas de Azure Resource Manager
ms.topic: conceptual
author: harelbr
ms.author: harelbr
ms.date: 06/26/2019
ms.reviewer: mbullwin
ms.openlocfilehash: aa8529abf3d7eea7d413c59ce62c93c7eb6c76d1
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87309348"
---
# <a name="manage-application-insights-smart-detection-rules-using-azure-resource-manager-templates"></a>Administración de reglas de detección inteligente de Application Insights con plantillas de Azure Resource Manager

Las reglas de detección inteligente de Application Insights se pueden administrar y configurar con [plantillas de Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md).
Este método puede usarse al implementar nuevos recursos de Application Insights con la automatización de Azure Resource Manager o para modificar la configuración de los recursos existentes.

## <a name="smart-detection-rule-configuration"></a>Configuración de una regla de detección inteligente

Puede configurar los siguientes valores en una regla de detección inteligente:
- Si la regla se habilita (el valor predeterminado es **true**).
- Si no se deben enviar correos electrónicos a los usuarios asociados con los roles [Lector de supervisión](../../role-based-access-control/built-in-roles.md#monitoring-reader) y [Colaborador de supervisión](../../role-based-access-control/built-in-roles.md#monitoring-contributor) cuando se encuentra una detección (el valor predeterminado es **true**).
- Los destinatarios de correo electrónico adicionales que deben recibir una notificación cuando se encuentra una detección.
    -  La configuración de correo electrónico no está disponible para las reglas de detección inteligente marcadas como _versión preliminar_.

Para permitir la configuración de los valores de regla a través de Azure Resource Manager, la configuración de reglas de detección inteligente ahora está disponible como recurso interno en el recurso de Application Insights denominado **ProactiveDetectionConfigs**.
Para obtener la máxima flexibilidad, cada regla de detección inteligente puede configurarse con valores de notificación únicos.

## <a name="examples"></a>Ejemplos

A continuación se muestran algunos ejemplos que muestran cómo configurar los valores de reglas de detección inteligente mediante plantillas de Azure Resource Manager.
Todos los ejemplos hacen referencia a un recurso de Application Insights denominado _"myApplication"_ y a la "regla de detección inteligente de duración de la dependencia prolongada" que internamente se denomina _"longdependencyduration"_ .
Asegúrese de reemplazar el nombre de recurso de Application Insights y de especificar el nombre interno de la regla de detección inteligente pertinente. En la tabla siguiente, busque una lista de los nombres internos de Azure Resource Manager correspondientes a cada regla de detección inteligente.

### <a name="disable-a-smart-detection-rule"></a>Deshabilitación de una regla de detección inteligente

```json
{
      "apiVersion": "2018-05-01-preview",
      "name": "myApplication",
      "type": "Microsoft.Insights/components",
      "location": "[resourceGroup().location]",
      "properties": {
        "ApplicationId": "myApplication"
      },
      "resources": [
        {
          "apiVersion": "2018-05-01-preview",
          "name": "longdependencyduration",
          "type": "ProactiveDetectionConfigs",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[resourceId('Microsoft.Insights/components', 'myApplication')]"
          ],
          "properties": {
            "name": "longdependencyduration",
            "sendEmailsToSubscriptionOwners": true,
            "customEmails": [],
            "enabled": false
          }
        }
      ]
    }
```

### <a name="disable-sending-email-notifications-for-a-smart-detection-rule"></a>Deshabilitación del envío de notificaciones por correo electrónico para una regla de detección inteligente

```json
{
      "apiVersion": "2018-05-01-preview",
      "name": "myApplication",
      "type": "Microsoft.Insights/components",
      "location": "[resourceGroup().location]",
      "properties": {
        "ApplicationId": "myApplication"
      },
      "resources": [
        {
          "apiVersion": "2018-05-01-preview",
          "name": "longdependencyduration",
          "type": "ProactiveDetectionConfigs",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[resourceId('Microsoft.Insights/components', 'myApplication')]"
          ],
          "properties": {
            "name": "longdependencyduration",
            "sendEmailsToSubscriptionOwners": false,
            "customEmails": [],
            "enabled": true
          }
        }
      ]
    }
```

### <a name="add-additional-email-recipients-for-a-smart-detection-rule"></a>Adición de otros destinatarios de correo electrónico para una regla de detección inteligente

```json
{
      "apiVersion": "2018-05-01-preview",
      "name": "myApplication",
      "type": "Microsoft.Insights/components",
      "location": "[resourceGroup().location]",
      "properties": {
        "ApplicationId": "myApplication"
      },
      "resources": [
        {
          "apiVersion": "2018-05-01-preview",
          "name": "longdependencyduration",
          "type": "ProactiveDetectionConfigs",
          "location": "[resourceGroup().location]",
          "dependsOn": [
            "[resourceId('Microsoft.Insights/components', 'myApplication')]"
          ],
          "properties": {
            "name": "longdependencyduration",
            "sendEmailsToSubscriptionOwners": true,
            "customEmails": ['alice@contoso.com', 'bob@contoso.com'],
            "enabled": true
          }
        }
      ]
    }

```


## <a name="smart-detection-rule-names"></a>Nombres de regla de detección inteligente

Seguidamente se muestra una tabla de nombres de reglas de detección inteligente tal y como aparecen en el portal, así como sus nombres internos, que deben utilizarse en la plantilla de Azure Resource Manager.

> [!NOTE]
> Las reglas de detección inteligente marcadas como de _versión preliminar_ no admiten notificaciones por correo electrónico. Por lo tanto, solo se puede establecer la propiedad _habilitada_ para estas reglas. 

| Nombre de regla de Azure Portal | Nombre interno
|:---|:---|
| Carga lenta de página | slowpageloadtime |
| Lentitud en el tiempo de respuesta del servidor | slowserverresponsetime |
| Duración de la dependencia prolongada | longdependencyduration |
| Degradación del tiempo de respuesta del servidor | degradationinserverresponsetime |
| Reducción de la duración de la dependencia | degradationindependencyduration |
| Degradación en la relación de gravedad de seguimiento (versión preliminar) | extension_traceseveritydetector |
| Aumento anormal del volumen de excepciones (versión preliminar) | extension_exceptionchangeextension |
| Detección de una posible fuga de memoria (versión preliminar) | extension_memoryleakextension |
| Detección de un posible problema de seguridad (versión preliminar) | extension_securityextensionspackage |
| Aumento anómalo del volumen de datos diario (versión preliminar) | extension_billingdatavolumedailyspikeextension |

### <a name="failure-anomalies-alert-rule"></a>Regla de alertas Anomalías de errores

Esta plantilla de Azure Resource Manager muestra la configuración de una regla de alertas de Anomalías en los errores con una gravedad de 2. Esta versión nueva de la regla de alertas de Anomalías en los errores es parte de la nueva plataforma de alertas de Azure y reemplaza la versión clásica que se retira como parte del [proceso de retirada de alertas clásicas](https://azure.microsoft.com/updates/classic-alerting-monitoring-retirement/).

> [!NOTE]
> Las anomalías de errores son un servicio global, por lo que la ubicación de la regla se crea en la ubicación global.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "resources": [
        {
            "type": "microsoft.alertsmanagement/smartdetectoralertrules",
            "apiVersion": "2019-03-01",
            "name": "Failure Anomalies - my-app",
            "location": "global", 
            "properties": {
                  "description": "Failure Anomalies notifies you of an unusual rise in the rate of failed HTTP requests or dependency calls.",
                  "state": "Enabled",
                  "severity": "2",
                  "frequency": "PT1M",
                  "detector": {
                  "id": "FailureAnomaliesDetector"
                  },
                  "scope": ["/subscriptions/00000000-1111-2222-3333-444444444444/resourceGroups/MyResourceGroup/providers/microsoft.insights/components/my-app"],
                  "actionGroups": {
                        "groupIds": ["/subscriptions/00000000-1111-2222-3333-444444444444/resourcegroups/MyResourceGroup/providers/microsoft.insights/actiongroups/MyActionGroup"]
                  }
            }
        }
    ]
}
```

> [!NOTE]
> Esta plantilla de Azure Resource Manager es única para la regla de alertas de Anomalías en los errores y es distinta de las otras reglas clásicas de detección inteligente que se describen en este artículo. Si desea administrar Anomalías en los errores manualmente, esto se hace en Alertas de Azure Monitor, mientras que todas las demás reglas de detección inteligente se administran en el panel Detección inteligente de la interfaz de usuario.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la detección automática:

- [Anomalías de error](./proactive-failure-diagnostics.md)
- [Fugas de memoria](./proactive-potential-memory-leak.md)
- [Anomalías de rendimiento](./proactive-performance-diagnostics.md)

