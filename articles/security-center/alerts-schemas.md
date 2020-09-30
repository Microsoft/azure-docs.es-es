---
title: Esquemas para las alertas de Azure Security Center
description: En este artículo se describen los distintos esquemas que usa Azure Security Center para las alertas de seguridad.
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.service: security-center
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/19/2020
ms.author: memildin
ms.openlocfilehash: 894fc4066ad408f0749e7a982011d82b205e6fec
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90901340"
---
# <a name="security-alerts-schemas"></a>Esquemas de las alertas de seguridad

Si su suscripción tiene habilitado Azure Defender, recibirá alertas de seguridad cuando Security Center detecte amenazas para sus recursos.

Estas alertas se pueden ver en las páginas de **Protección contra amenazas** de Azure Security Center o a través de herramientas externas como:

- [Azure Sentinel](https://docs.microsoft.com/azure/sentinel/): SIEM nativo de la nube de Microsoft. El conector de Sentinel obtiene las alertas de Azure Security Center y las envía al [área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace) para Azure Sentinel.
- SIEM de terceros: use las herramientas de [exportación continua](continuous-export.md) de Security Center para enviar datos a [Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/). Luego, integre los datos de Event Hubs con un SIEM de terceros.
- [La API REST](https://docs.microsoft.com/rest/api/securitycenter/): si usa la API REST para acceder a las alertas, consulte la [documentación de Alerts API en línea](https://docs.microsoft.com/rest/api/securitycenter/alerts).

Si utiliza cualquier método de programación para consumir las alertas, necesitará el esquema correcto para encontrar los campos pertinentes. De igual modo, si va a realizar una exportación a un centro de eventos o a intentar desencadenar Automatización de flujo de trabajo con conectores HTTP genéricos, utilice los esquemas para analizar correctamente los objetos JSON.

>[!IMPORTANT]
> El esquema es ligeramente diferente en cada uno de estos escenarios, así que asegúrese de que selecciona la pestaña pertinente a continuación.


## <a name="the-schemas"></a>Los esquemas 


### <a name="workflow-automation-and-continuous-export-to-event-hub"></a>[Automatización de flujo de trabajo y exportación continua a un centro de eventos](#tab/schema-continuousexport)

### <a name="sample-json-for-alerts-sent-to-logic-apps-event-hub-and-third-party-siems"></a>JSON de ejemplo para alertas enviadas a Logic Apps, centro de eventos y SIEM de terceros

A continuación encontrará el esquema de los eventos de alerta pasados a:

- Instancias de Azure Logic App que se configuraron en la automatización del flujo de trabajo de Security Center
- Azure Event Hub que usa la característica de exportación continua de Security Center

Para más información sobre la característica de automatización del flujo de trabajo, consulte [Automatización de respuestas a alertas y recomendaciones](workflow-automation.md).
Para más información sobre la exportación continua, consulte [Exportación de alertas y recomendaciones](continuous-export.md).

[!INCLUDE [Workflow schema](../../includes/security-center-alerts-schema-workflow-automation.md)]




### <a name="azure-sentinel-and-log-analytics-workspaces"></a>[Áreas de trabajo de Log Analytics y Azure Sentinel](#tab/schema-sentinel)

El conector de Sentinel obtiene las alertas de Azure Security Center y las envía al área de trabajo de Log Analytics para Azure Sentinel. 

Para crear un caso o incidente de Sentinel mediante alertas de Security Center, necesitará el esquema para las alertas que se muestran a continuación. 

Para más información acerca de Azure Sentinel, consulte [la documentación](https://docs.microsoft.com/azure/sentinel/).

[!INCLUDE [Sentinel and workspace schema](../../includes/security-center-alerts-schema-log-analytics-workspace.md)]




### <a name="azure-activity-log"></a>[Registro de actividad de Azure](#tab/schema-activitylog)

Las auditorías de Azure Security Center generaban alertas de seguridad como eventos en el registro de actividad de Azure.

Para ver los eventos de las alertas de seguridad en el registro de actividad, busque el evento Activar alerta como se muestra:

[![Búsqueda del registro de actividad para el evento activar alerta](media/alerts-schemas/sample-activity-log-alert.png)](media/alerts-schemas/sample-activity-log-alert.png#lightbox)


### <a name="sample-json-for-alerts-sent-to-azure-activity-log"></a>JSON de ejemplo para alertas enviadas al registro de actividad de Azure

```json
{
    "channels": "Operation",
    "correlationId": "2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff",
    "description": "PREVIEW - Role binding to the cluster-admin role detected. Kubernetes audit log analysis detected a new binding to the cluster-admin role which gives administrator privileges.\r\nUnnecessary administrator privileges might cause privilege escalation in the cluster.",
    "eventDataId": "2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff",
    "eventName": {
        "value": "PREVIEW - Role binding to the cluster-admin role detected",
        "localizedValue": "PREVIEW - Role binding to the cluster-admin role detected"
    },
    "category": {
        "value": "Security",
        "localizedValue": "Security"
    },
    "eventTimestamp": "2019-12-25T18:52:36.801035Z",
    "id": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RESOURCE_GROUP_NAME/providers/Microsoft.Security/locations/centralus/alerts/2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff/events/2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff/ticks/637128967568010350",
    "level": "Informational",
    "operationId": "2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff",
    "operationName": {
        "value": "Microsoft.Security/locations/alerts/activate/action",
        "localizedValue": "Activate Alert"
    },
    "resourceGroupName": "RESOURCE_GROUP_NAME",
    "resourceProviderName": {
        "value": "Microsoft.Security",
        "localizedValue": "Microsoft.Security"
    },
    "resourceType": {
        "value": "Microsoft.Security/locations/alerts",
        "localizedValue": "Microsoft.Security/locations/alerts"
    },
    "resourceId": "/subscriptions/SUBSCRIPTION_ID/resourceGroups/RESOURCE_GROUP_NAME/providers/Microsoft.Security/locations/centralus/alerts/2518250008431989649_e7313e05-edf4-466d-adfd-35974921aeff",
    "status": {
        "value": "Active",
        "localizedValue": "Active"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2019-12-25T19:14:03.5507487Z",
    "subscriptionId": "SUBSCRIPTION_ID",
    "properties": {
        "clusterRoleBindingName": "cluster-admin-binding",
        "subjectName": "for-binding-test",
        "subjectKind": "ServiceAccount",
        "username": "masterclient",
        "actionTaken": "Detected",
        "resourceType": "Kubernetes Service",
        "severity": "Low",
        "intent": "[\"Persistence\"]",
        "compromisedEntity": "ASC-IGNITE-DEMO",
        "remediationSteps": "[\"Review the user in the alert details. If cluster-admin is unnecessary for this user, consider granting lower privileges to the user.\"]",
        "attackedResourceType": "Kubernetes Service"
    },
    "relatedEvents": []
}
```

### <a name="the-data-model-of-the-schema"></a>El modelo de datos del esquema

|Campo|Descripción|
|----|----|
|**channels**|Constante, "Operation"|
|**correlationId**|Identificador de alerta de Azure Security Center|
|**description**|Descripción de la alerta|
|**eventDataId**|Consulte correlationId|
|**eventName**|Los subcampos value y localizedValue contienen el nombre para mostrar de la alerta|
|**category**|Los subcampos Value y localizedValue son constantes: "Security"|
|**eventTimestamp**|La marca de tiempo UTC de cuando se generó la alerta|
|**id**|Identificador completo de la alerta|
|**level**|Constante, "Informational"|
|**operationId**|Consulte correlationId|
|**operationName**|El campo value es constante: "Microsoft.Security/locations/alerts/activate/action" y el valor localizado será "Activar alerta" (puede que esté posiblemente localizado en función de la configuración regional del usuario)|
|**resourceGroupName**|Incluirá el nombre del grupo de recursos|
|**resourceProviderName**|Los subcampos value y localizedValue son constantes: "Microsoft.Security"|
|**resourceType**|Los subcampos value y localizedValue son constantes: "Microsoft.Security/locations/alerts"|
|**resourceId**|El identificador de recursos de Azure completo|
|**status**|Los subcampos value y localizedValue son constantes: "Active"|
|**subStatus**|Los subcampos value y localizedValue están vacíos|
|**submissionTimestamp**|Marca de tiempo UTC del envío de eventos al registro de actividad|
|**subscriptionId**|Identificador de suscripción del recurso en peligro|
|**properties**|Una bolsa JSON de propiedades adicionales que pertenecen a la alerta. Pueden cambiar de una alerta a la otra; sin embargo, los siguientes campos aparecerán en todas ellas:<br>- severity: la gravedad del ataque<br>- compromisedEntity: el nombre del recurso en peligro<br>- remediationSteps: matriz de pasos de corrección que se deben realizar<br>- intent: La intención de la cadena de eliminación de la alerta. Las posibles intenciones se documentan en la [tabla de intenciones](alerts-reference.md#intentions)|
|**relatedEvents**|Constante: matriz vacía|
|||





### <a name="ms-graph-api"></a>[MS Graph API](#tab/schema-graphapi)

Microsoft Graph es la puerta de enlace a los datos y la inteligencia en Microsoft 365. Proporciona un modelo de programación unificado que puede usar para acceder a la ingente cantidad de datos en Microsoft 365, Windows 10 y Enterprise Mobility + Security. Use la riqueza de datos de Microsoft Graph para crear aplicaciones para organizaciones y consumidores que interactúen con millones de usuarios.

El esquema y una representación JSON de las alertas de seguridad enviadas a MS Graph, están disponibles en [la documentación de Microsoft Graph](https://docs.microsoft.com/graph/api/resources/alert?view=graph-rest-1.0).

---


## <a name="next-steps"></a>Pasos siguientes

En este artículo se han descrito los esquemas que usan las herramientas de protección contra amenazas de Azure Security Center al enviar información de alertas de seguridad.

Para más información sobre las distintas formas de acceder a las alertas de seguridad desde fuera de Security Center, consulte las siguientes páginas:

- [Azure Sentinel](https://docs.microsoft.com/azure/sentinel/): SIEM nativo de la nube de Microsoft
- [Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/): servicio de ingesta de datos en tiempo real y totalmente administrado de Microsoft
- [Característica de exportación continua](continuous-export.md) de Security Center
- [Áreas de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/learn/quick-create-workspace): Azure Monitor almacena los datos de registro en un área de trabajo de Log Analytics, un contenedor que incluye información de configuración y datos
