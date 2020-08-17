---
title: Esquema de eventos del registro de actividad de Azure
description: Describe el esquema de eventos para cada categoría en el registro de actividad de Azure.
author: bwren
services: azure-monitor
ms.topic: reference
ms.date: 06/09/2020
ms.author: bwren
ms.subservice: logs
ms.openlocfilehash: 656161849ce8d48fb15cfac4024ec5b77adb5fee
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87829516"
---
# <a name="azure-activity-log-event-schema"></a>Esquema de eventos del registro de actividad de Azure
El [registro de actividad de Azure](platform-logs-overview.md) proporciona información sobre los eventos que se han producido en Azure en el nivel de la suscripción. En este artículo se describen las categorías de registro de actividad y el esquema de cada una. 

El esquema variará en función de cómo acceda al registro:
 
- Los esquemas que se describen en este artículo corresponden al acceso al registro de actividad desde la [API REST](/rest/api/monitor/activitylogs). Este es también el esquema que se usa al seleccionar la opción **JSON** al ver un evento en Azure Portal.
- Consulte la sección final titulada [Esquema de la cuenta de almacenamiento y Event Hubs](#schema-from-storage-account-and-event-hubs) para el esquema correspondiente cuando se usa una [configuración de diagnóstico](diagnostic-settings.md) para enviar el registro de actividad a Azure Storage o Azure Event Hubs.
- Consulte [Referencia de datos de Azure Monitor](/azure/azure-monitor/reference/) parar conocer el esquema correspondiente cuando se usa una [configuración de diagnóstico](diagnostic-settings.md) para enviar el registro de actividad a un área de trabajo de Log Analytics.


## <a name="categories"></a>Categorías
Cada evento del registro de actividad tiene una categoría determinada que se describe en la tabla siguiente. Consulte las secciones siguientes para más información sobre cada categoría y su esquema al acceder al registro de actividad desde el portal, PowerShell, la CLI y la API REST. El esquema es diferente cuando el [registro de actividad se transmite a Azure Storage o a Event Hubs](./resource-logs.md#send-to-azure-event-hubs). En la última sección del artículo, encontrará la correspondencia de las propiedades en el [esquema de registros de recursos](./resource-logs-schema.md).

| Category | Descripción |
|:---|:---|
| [Administrativas](#administrative-category) | Contiene el registro de todas las operaciones de creación, actualización, eliminación y acción realizadas mediante Resource Manager. Algunos ejemplos de eventos administrativos incluyen la _creación de una máquina virtual_ y la _eliminación de un grupo de seguridad de red_.<br><br>Cada acción realizada por un usuario o aplicación mediante Resource Manager se modela como una operación en un tipo de recurso determinado. Si el tipo de operación es _Write_, _Delete_ o _Action_, los registros de inicio y corrección o error de esa operación se registran en la categoría Administrativo. Los eventos de la categoría Administrativo también incluyen los cambios realizados en el control de acceso basado en rol de una suscripción. |
| [Service Health](#service-health-category) | Contiene el registro de los incidentes de estado del servicio que se han producido en Azure. Ejemplo de un evento de Service Health: _SQL Azure está experimentando un tiempo de inactividad en la región Este de EE. UU._ . <br><br>Los eventos de Service Health pueden encuadrarse dentro de seis variedades: _Acción requerida_, _Recuperación asistida_, _Incidente_, _Mantenimiento_, _Información_ o _Seguridad_. Estos eventos solo se crean si tiene un recurso en la suscripción que se puede ver afectado por el evento.
| [Estado de los recursos](#resource-health-category) | Contiene el registro de los eventos de estado de los recursos que se han producido en los recursos de Azure. Ejemplo de un evento de Resource Health: _Cambio del estado de mantenimiento de una máquina virtual a No disponible_.<br><br>Los eventos de Resource Health pueden representar uno de los cuatro estados de mantenimiento siguientes: _Disponible_, _No disponible_, _Degradado_ y _Desconocido_. Además, los eventos de Resource Health se pueden clasificar como _iniciados por la plataforma_ o _por el usuario_. |
| [Alerta](#alert-category) | Contiene el registro de activaciones de alertas de Azure. Ejemplo de un evento de alerta: _% de CPU en myVM ha estado por encima de 80 durante los últimos 5 minutos_.|
| [Autoscale](#autoscale-category) | Contiene el registro de los eventos relacionados con el funcionamiento del motor de escalado automático en función de cualquier configuración de escalado automático que haya definido en la suscripción. Ejemplo de un evento de escalado automático: _Error durante la acción de escalado vertical_. |
| [Recomendación](#recommendation-category) | Contiene eventos de recomendación de Azure Advisor. |
| [Seguridad](#security-category) | Contiene el registro de todas las alertas generadas por Azure Security Center. Ejemplo de un evento de seguridad: _Se ha ejecutado un archivo de extensión doble_. |
| [Directiva](#policy-category) | Contiene registros de todas las operaciones de acción de efecto realizadas por Azure Policy. Ejemplos de eventos de directiva: _Auditar_ y _Denegar_. Cada acción llevada a cabo por Azure Policy se modela como una operación en un recurso. |

## <a name="administrative-category"></a>Categoría administrativa
Esta categoría contiene el registro de todas las operaciones de creación, actualización, eliminación y acción realizadas a través de Resource Manager. Los ejemplos de los tipos de eventos que aparecen en esta categoría incluyen "crear máquina virtual" y "eliminar grupo de seguridad de red". Cada acción realizada por un usuario o una aplicación mediante Resource Manager se modela como una operación en un tipo de recurso determinado. Si el tipo de operación es Write, Delete o Action, los registros de inicio y corrección o error de esa operación se registran en la categoría Administrativo. La categoría Administrativo también incluye los cambios realizados en el control de acceso basado en roles de una suscripción.

### <a name="sample-event"></a>Evento de ejemplo
```json
{
    "authorization": {
        "action": "Microsoft.Network/networkSecurityGroups/write",
        "scope": "/subscriptions/<subscription ID>/resourcegroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNSG"
    },
    "caller": "rob@contoso.com",
    "channels": "Operation",
    "claims": {
        "aud": "https://management.core.windows.net/",
        "iss": "https://sts.windows.net/1114444b-7467-4144-a616-e3a5d63e147b/",
        "iat": "1234567890",
        "nbf": "1234567890",
        "exp": "1234567890",
        "_claim_names": "{\"groups\":\"src1\"}",
        "_claim_sources": "{\"src1\":{\"endpoint\":\"https://graph.microsoft.com/1114444b-7467-4144-a616-e3a5d63e147b/users/f409edeb-4d29-44b5-9763-ee9348ad91bb/getMemberObjects\"}}",
        "http://schemas.microsoft.com/claims/authnclassreference": "1",
        "aio": "A3GgTJdwK4vy7Fa7l6DgJC2mI0GX44tML385OpU1Q+z+jaPnFMwB",
        "http://schemas.microsoft.com/claims/authnmethodsreferences": "rsa,mfa",
        "appid": "355249ed-15d9-460d-8481-84026b065942",
        "appidacr": "2",
        "http://schemas.microsoft.com/2012/01/devicecontext/claims/identifier": "10845a4d-ffa4-4b61-a3b4-e57b9b31cdb5",
        "e_exp": "262800",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "Robertson",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "Rob",
        "ipaddr": "111.111.1.111",
        "name": "Rob Robertson",
        "http://schemas.microsoft.com/identity/claims/objectidentifier": "f409edeb-4d29-44b5-9763-ee9348ad91bb",
        "onprem_sid": "S-1-5-21-4837261184-168309720-1886587427-18514304",
        "puid": "18247BBD84827C6D",
        "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "b-24Jf94A3FH2sHWVIFqO3-RSJEiv24Jnif3gj7s",
        "http://schemas.microsoft.com/identity/claims/tenantid": "1114444b-7467-4144-a616-e3a5d63e147b",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": "rob@contoso.com",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "rob@contoso.com",
        "uti": "IdP3SUJGtkGlt7dDQVRPAA",
        "ver": "1.0"
    },
    "correlationId": "b5768deb-836b-41cc-803e-3f4de2f9e40b",
    "eventDataId": "d0d36f97-b29c-4cd9-9d3d-ea2b92af3e9d",
    "eventName": {
        "value": "EndRequest",
        "localizedValue": "End request"
    },
    "category": {
        "value": "Administrative",
        "localizedValue": "Administrative"
    },
    "eventTimestamp": "2018-01-29T20:42:31.3810679Z",
    "id": "/subscriptions/<subscription ID>/resourcegroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNSG/events/d0d36f97-b29c-4cd9-9d3d-ea2b92af3e9d/ticks/636528553513810679",
    "level": "Informational",
    "operationId": "04e575f8-48d0-4c43-a8b3-78c4eb01d287",
    "operationName": {
        "value": "Microsoft.Network/networkSecurityGroups/write",
        "localizedValue": "Microsoft.Network/networkSecurityGroups/write"
    },
    "resourceGroupName": "myResourceGroup",
    "resourceProviderName": {
        "value": "Microsoft.Network",
        "localizedValue": "Microsoft.Network"
    },
    "resourceType": {
        "value": "Microsoft.Network/networkSecurityGroups",
        "localizedValue": "Microsoft.Network/networkSecurityGroups"
    },
    "resourceId": "/subscriptions/<subscription ID>/resourcegroups/myResourceGroup/providers/Microsoft.Network/networkSecurityGroups/myNSG",
    "status": {
        "value": "Succeeded",
        "localizedValue": "Succeeded"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2018-01-29T20:42:50.0724829Z",
    "subscriptionId": "<subscription ID>",
    "properties": {
        "statusCode": "Created",
        "serviceRequestId": "a4c11dbd-697e-47c5-9663-12362307157d",
        "responseBody": "",
        "requestbody": ""
    },
    "relatedEvents": []
}

```

### <a name="property-descriptions"></a>Descripciones de propiedades
| Nombre del elemento | Descripción |
| --- | --- |
| authorization |Blob de propiedades RBAC del evento. Normalmente incluye las propiedades "action", "role" y "scope". |
| caller |Dirección de correo electrónico del usuario que ha realizado la operación, la notificación de UPN o la notificación de SPN basada en la disponibilidad. |
| canales nueva |Uno de los valores siguientes: “Admin”, “Operation” |
| claims |Token de JWT que se usa en Active Directory para autenticar el usuario o la aplicación para llevar a cabo esta operación en Resource Manager. |
| correlationId |Normalmente, un GUID en formato de cadena. Los eventos que comparten correlationId pertenecen a la misma acción general. |
| description |Descripción de texto estático de un evento. |
| eventDataId |Identificador único de un evento. |
| eventName | Nombre descriptivo del evento administrativo. |
| category | Siempre "Administrativo" |
| httpRequest |Blob que describe la solicitud HTTP. Normalmente incluye "clientRequestId", "clientIpAddress" y "method" (método HTTP). Por ejemplo, PUT). |
| level |Nivel del evento. Uno de los valores siguientes: “Critical”, “Error”, “Warning” e “Informational” |
| resourceGroupName |Nombre del grupo de recursos del recurso afectado. |
| resourceProviderName |Nombre del proveedor de recursos del recurso afectado. |
| resourceType | Tipo de recurso que se vio afectado por un evento administrativo. |
| resourceId |Identificador de recurso del recurso afectado. |
| operationId |GUID compartido entre los eventos correspondientes a una sola operación. |
| operationName |Nombre de la operación. |
| properties |Conjunto de pares `<Key, Value>` (es decir, diccionario) que describen los detalles del evento. |
| status |Cadena que describe el estado de la operación. Entre los valores habituales, se incluyen: Started, In Progress, Succeeded, Failed, Active, Resolved. |
| subStatus |Normalmente el código de estado HTTP de la llamada REST correspondiente, pero también puede incluir otras cadenas que describen un subestado, como estos valores habituales: Correcto (código de estado HTTP: 200), Creado (código de estado HTTP: 201), Aceptado (código de estado HTTP: 202), Sin contenido (código de estado HTTP: 204), Solicitud incorrecta (código de estado HTTP: 400), No encontrado (código de estado HTTP): 404), Conflicto (código de estado HTTP: 409), Error interno del servidor (código de estado HTTP: 500), Servicio no disponible (código de estado HTTP: Tiempo de espera agotado para la puerta de enlace (código de estado HTTP: 503) 504). |
| eventTimestamp |Marca de tiempo de cuándo el servicio de Azure generó el evento que procesó la solicitud correspondiente al evento. |
| submissionTimestamp |Marca de tiempo de cuándo el evento empezó a estar disponible para las consultas. |
| subscriptionId |Identificador de suscripción de Azure |

## <a name="service-health-category"></a>Categoría de estado del servicio
Esta categoría contiene el registro de los incidentes de estado del servicio que se han producido en Azure. Un ejemplo del tipo de evento que aparece en esta categoría es "SQL Azure en el este de EE. UU. está experimentando un tiempo de inactividad". Los eventos de estado del servicio pueden encuadrarse dentro de cinco variedades: Acción requerida, Recuperación asistida, Incidente, Mantenimiento, Información o Seguridad, y solo aparecen si tiene un recurso en la suscripción que se podría ver afectado por el evento.

### <a name="sample-event"></a>Evento de ejemplo
```json
{
  "channels": "Admin",
  "correlationId": "c550176b-8f52-4380-bdc5-36c1b59d3a44",
  "description": "Active: Network Infrastructure - UK South",
  "eventDataId": "c5bc4514-6642-2be3-453e-c6a67841b073",
  "eventName": {
      "value": null
  },
  "category": {
      "value": "ServiceHealth",
      "localizedValue": "Service Health"
  },
  "eventTimestamp": "2017-07-20T23:30:14.8022297Z",
  "id": "/subscriptions/<subscription ID>/events/c5bc4514-6642-2be3-453e-c6a67841b073/ticks/636361902148022297",
  "level": "Warning",
  "operationName": {
      "value": "Microsoft.ServiceHealth/incident/action",
      "localizedValue": "Microsoft.ServiceHealth/incident/action"
  },
  "resourceProviderName": {
      "value": null
  },
  "resourceType": {
      "value": null,
      "localizedValue": ""
  },
  "resourceId": "/subscriptions/<subscription ID>",
  "status": {
      "value": "Active",
      "localizedValue": "Active"
  },
  "subStatus": {
      "value": null
  },
  "submissionTimestamp": "2017-07-20T23:30:34.7431946Z",
  "subscriptionId": "<subscription ID>",
  "properties": {
    "title": "Network Infrastructure - UK South",
    "service": "Service Fabric",
    "region": "UK South",
    "communication": "Starting at approximately 21:41 UTC on 20 Jul 2017, a subset of customers in UK South may experience degraded performance, connectivity drops or timeouts when accessing their Azure resources hosted in this region. Engineers are investigating underlying Network Infrastructure issues in this region. Impacted services may include, but are not limited to App Services, Automation, Service Bus, Log Analytics, Key Vault, SQL Database, Service Fabric, Event Hubs, Stream Analytics, Azure Data Movement, API Management, and Azure Cognitive Search. Multiple engineering teams are engaged in multiple workflows to mitigate the impact. The next update will be provided in 60 minutes, or as events warrant.",
    "incidentType": "Incident",
    "trackingId": "NA0F-BJG",
    "impactStartTime": "2017-07-20T21:41:00.0000000Z",
    "impactedServices": "[{\"ImpactedRegions\":[{\"RegionName\":\"UK South\"}],\"ServiceName\":\"Service Fabric\"}]",
    "defaultLanguageTitle": "Network Infrastructure - UK South",
    "defaultLanguageContent": "Starting at approximately 21:41 UTC on 20 Jul 2017, a subset of customers in UK South may experience degraded performance, connectivity drops or timeouts when accessing their Azure resources hosted in this region. Engineers are investigating underlying Network Infrastructure issues in this region. Impacted services may include, but are not limited to App Services, Automation, Service Bus, Log Analytics, Key Vault, SQL Database, Service Fabric, Event Hubs, Stream Analytics, Azure Data Movement, API Management, and Azure Cognitive Search. Multiple engineering teams are engaged in multiple workflows to mitigate the impact. The next update will be provided in 60 minutes, or as events warrant.",
    "stage": "Active",
    "communicationId": "636361902146035247",
    "version": "0.1.1"
  }
}
```
Consulte el artículo sobre las [notificaciones de estado de servicio](../../service-health/service-notifications.md) para la documentación sobre los valores de las propiedades.

## <a name="resource-health-category"></a>Categoría de estado de los recursos
Esta categoría contiene el registro de los eventos de estado del servicio que se han producido en los recursos de Azure. Un ejemplo del tipo de evento que aparece en esta categoría es "El estado de mantenimiento de la máquina virtual se cambió a No disponible". Los eventos de mantenimiento de recursos pueden representar uno de los cuatro estados de mantenimiento siguientes: Disponible, No disponible, Degradado y Desconocido. Además, los eventos de mantenimiento de recursos se pueden clasificar como iniciados por la plataforma o por el usuario.

### <a name="sample-event"></a>Evento de ejemplo

```json
{
    "channels": "Admin, Operation",
    "correlationId": "28f1bfae-56d3-7urb-bff4-194d261248e9",
    "description": "",
    "eventDataId": "a80024e1-883d-37ur-8b01-7591a1befccb",
    "eventName": {
        "value": "",
        "localizedValue": ""
    },
    "category": {
        "value": "ResourceHealth",
        "localizedValue": "Resource Health"
    },
    "eventTimestamp": "2018-09-04T15:33:43.65Z",
    "id": "/subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Compute/virtualMachines/<resource name>/events/a80024e1-883d-42a5-8b01-7591a1befccb/ticks/636716720236500000",
    "level": "Critical",
    "operationId": "",
    "operationName": {
        "value": "Microsoft.Resourcehealth/healthevent/Activated/action",
        "localizedValue": "Health Event Activated"
    },
    "resourceGroupName": "<resource group>",
    "resourceProviderName": {
        "value": "Microsoft.Resourcehealth/healthevent/action",
        "localizedValue": "Microsoft.Resourcehealth/healthevent/action"
    },
    "resourceType": {
        "value": "Microsoft.Compute/virtualMachines",
        "localizedValue": "Microsoft.Compute/virtualMachines"
    },
    "resourceId": "/subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.Compute/virtualMachines/<resource name>",
    "status": {
        "value": "Active",
        "localizedValue": "Active"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2018-09-04T15:36:24.2240867Z",
    "subscriptionId": "<subscription ID>",
    "properties": {
        "stage": "Active",
        "title": "Virtual Machine health status changed to unavailable",
        "details": "Virtual machine has experienced an unexpected event",
        "healthStatus": "Unavailable",
        "healthEventType": "Downtime",
        "healthEventCause": "PlatformInitiated",
        "healthEventCategory": "Unplanned"
    },
    "relatedEvents": []
}
```

### <a name="property-descriptions"></a>Descripciones de propiedades
| Nombre del elemento | Descripción |
| --- | --- |
| canales nueva | Siempre es "Admin, Operation". |
| correlationId | GUID en formato de cadena. |
| description |Descripción de texto estático del evento de alerta. |
| eventDataId |Identificador único del evento de alerta. |
| category | Siempre "Resource Health". |
| eventTimestamp |Marca de tiempo de cuándo el servicio de Azure generó el evento que procesó la solicitud correspondiente al evento. |
| level |Nivel del evento. Uno de los valores siguientes: “Critical”, “Error”, “Warning”, “Informational” y “Verbose” |
| operationId |GUID compartido entre los eventos correspondientes a una sola operación. |
| operationName |Nombre de la operación. |
| resourceGroupName |Nombre del grupo de recursos que contiene el recurso. |
| resourceProviderName |Siempre "Microsoft.Resourcehealth/healthevent/action". |
| resourceType | Tipo de recurso que se vio afectado por un evento de Resource Health. |
| resourceId | Id. del recurso afectado. |
| status |Cadena que describe el estado del evento de estado. Los valores pueden ser: Active, Resolved, InProgress, Updated. |
| subStatus | Normalmente es null para las alertas. |
| submissionTimestamp |Marca de tiempo de cuándo el evento empezó a estar disponible para las consultas. |
| subscriptionId |Identificador de suscripción de Azure |
| properties |Conjunto de pares `<Key, Value>` (es decir, diccionario) que describen los detalles del evento.|
| properties.title | Cadena fácil de usar que describe el estado del recurso. |
| properties.details | Cadena fácil de usar que aporta detalles adicionales sobre el evento. |
| properties.currentHealthStatus | Estado de mantenimiento actual del recurso. Uno de los valores siguientes: "Available", "Unavailable", "Degraded", and "Unknown". |
| properties.previousHealthStatus | Estado de mantenimiento anterior del recurso. Uno de los valores siguientes: "Available", "Unavailable", "Degraded", and "Unknown". |
| properties.type | Descripción del tipo de evento de estado del recurso. |
| properties.cause | Descripción de la causa del evento de estado del recurso. Ya sea "UserInitiated" o "PlatformInitiated". |


## <a name="alert-category"></a>Categoría de alertas
Esta categoría contiene el registro de todas las activaciones de alertas de Azure clásicas. Un ejemplo del tipo de evento que aparece en esta categoría es "el % de CPU en myVM ha estado por encima de 80 durante los últimos 5 minutos". Varios sistemas de Azure tienen un concepto de alerta: puede definir una regla de algún tipo y recibir una notificación cuando las condiciones coincidan con esa regla. Cada vez que un tipo de alerta de Azure compatible "se activa" o se cumplen las condiciones para generar una notificación, también se inserta un registro de la activación en esta categoría del Registro de actividad.

### <a name="sample-event"></a>Evento de ejemplo

```json
{
  "caller": "Microsoft.Insights/alertRules",
  "channels": "Admin, Operation",
  "claims": {
    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/spn": "Microsoft.Insights/alertRules"
  },
  "correlationId": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/microsoft.insights/alertrules/myalert/incidents/L3N1YnNjcmlwdGlvbnMvZGY2MDJjOWMtN2FhMC00MDdkLWE2ZmItZWIyMGM4YmQxMTkyL3Jlc291cmNlR3JvdXBzL0NzbUV2ZW50RE9HRk9PRC1XZXN0VVMvcHJvdmlkZXJzL21pY3Jvc29mdC5pbnNpZ2h0cy9hbGVydHJ1bGVzL215YWxlcnQwNjM2MzYyMjU4NTM1MjIxOTIw",
  "description": "'Disk read LessThan 100000 ([Count]) in the last 5 minutes' has been resolved for CloudService: myResourceGroup/Production/Event.BackgroundJobsWorker.razzle (myResourceGroup)",
  "eventDataId": "149d4baf-53dc-4cf4-9e29-17de37405cd9",
  "eventName": {
    "value": "Alert",
    "localizedValue": "Alert"
  },
  "category": {
    "value": "Alert",
    "localizedValue": "Alert"
  },
  "id": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ClassicCompute/domainNames/myResourceGroup/slots/Production/roles/Event.BackgroundJobsWorker.razzle/events/149d4baf-53dc-4cf4-9e29-17de37405cd9/ticks/636362258535221920",
  "level": "Informational",
  "resourceGroupName": "myResourceGroup",
  "resourceProviderName": {
    "value": "Microsoft.ClassicCompute",
    "localizedValue": "Microsoft.ClassicCompute"
  },
  "resourceId": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ClassicCompute/domainNames/myResourceGroup/slots/Production/roles/Event.BackgroundJobsWorker.razzle",
  "resourceType": {
    "value": "Microsoft.ClassicCompute/domainNames/slots/roles",
    "localizedValue": "Microsoft.ClassicCompute/domainNames/slots/roles"
  },
  "operationId": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/microsoft.insights/alertrules/myalert/incidents/L3N1YnNjcmlwdGlvbnMvZGY2MDJjOWMtN2FhMC00MDdkLWE2ZmItZWIyMGM4YmQxMTkyL3Jlc291cmNlR3JvdXBzL0NzbUV2ZW50RE9HRk9PRC1XZXN0VVMvcHJvdmlkZXJzL21pY3Jvc29mdC5pbnNpZ2h0cy9hbGVydHJ1bGVzL215YWxlcnQwNjM2MzYyMjU4NTM1MjIxOTIw",
  "operationName": {
    "value": "Microsoft.Insights/AlertRules/Resolved/Action",
    "localizedValue": "Microsoft.Insights/AlertRules/Resolved/Action"
  },
  "properties": {
    "RuleUri": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/microsoft.insights/alertrules/myalert",
    "RuleName": "myalert",
    "RuleDescription": "",
    "Threshold": "100000",
    "WindowSizeInMinutes": "5",
    "Aggregation": "Average",
    "Operator": "LessThan",
    "MetricName": "Disk read",
    "MetricUnit": "Count"
  },
  "status": {
    "value": "Resolved",
    "localizedValue": "Resolved"
  },
  "subStatus": {
    "value": null
  },
  "eventTimestamp": "2017-07-21T09:24:13.522192Z",
  "submissionTimestamp": "2017-07-21T09:24:15.6578651Z",
  "subscriptionId": "<subscription ID>"
}
```

### <a name="property-descriptions"></a>Descripciones de propiedades
| Nombre del elemento | Descripción |
| --- | --- |
| caller | Siempre es Microsoft.Insights/alertRules. |
| canales nueva | Siempre es "Admin, Operation". |
| claims | Blob JSON con el SPN (nombre de entidad de seguridad de servicio), o tipo de recurso, del motor de alertas. |
| correlationId | GUID en formato de cadena. |
| description |Descripción de texto estático del evento de alerta. |
| eventDataId |Identificador único del evento de alerta. |
| category | Siempre "Alerta" |
| level |Nivel del evento. Uno de los valores siguientes: “Critical”, “Error”, “Warning” e “Informational” |
| resourceGroupName |Nombre del grupo de recursos del recurso afectado si se trata de una alerta de métrica. En otros tipos de alertas, es el nombre del grupo de recursos que contiene la propia alerta. |
| resourceProviderName |Nombre del proveedor de recursos del recurso afectado si se trata de una alerta de métrica. En otros tipos de alertas, es el nombre del proveedor de recursos de la propia alerta. |
| resourceId | Nombre del identificador del recurso afectado si se trata de una alerta de métrica. En otros tipos de alertas, es el identificador del propio recurso de alerta. |
| operationId |GUID compartido entre los eventos correspondientes a una sola operación. |
| operationName |Nombre de la operación. |
| properties |Conjunto de pares `<Key, Value>` (es decir, diccionario) que describen los detalles del evento. |
| status |Cadena que describe el estado de la operación. Entre los valores habituales, se incluyen: Started, In Progress, Succeeded, Failed, Active, Resolved. |
| subStatus | Normalmente es null para las alertas. |
| eventTimestamp |Marca de tiempo de cuándo el servicio de Azure generó el evento que procesó la solicitud correspondiente al evento. |
| submissionTimestamp |Marca de tiempo de cuándo el evento empezó a estar disponible para las consultas. |
| subscriptionId |Identificador de suscripción de Azure |

### <a name="properties-field-per-alert-type"></a>Campo Propiedades por tipo de alerta
El campo Propiedades contendrá valores diferentes dependiendo del origen del evento de alerta. Dos proveedores de eventos de alerta comunes son las alertas de registro de actividad y las alertas de métrica.

#### <a name="properties-for-activity-log-alerts"></a>Propiedades de las alertas del registro de actividad
| Nombre del elemento | Descripción |
| --- | --- |
| properties.subscriptionId | Identificador de suscripción del evento del registro de actividad que hizo que se activara esta regla de alertas del registro de actividad. |
| properties.eventDataId | Identificador de datos del evento del registro de actividad que hizo que se activara esta regla de alertas del registro de actividad. |
| properties.resourceGroup | Grupo de recursos del evento del registro de actividad que hizo que se activara esta regla de alertas del registro de actividad. |
| properties.resourceId | Identificador de recurso del evento del registro de actividad que hizo que se activara esta regla de alertas del registro de actividad. |
| properties.eventTimestamp | Marca de tiempo del evento del registro de actividad que hizo que se activara esta regla de alertas del registro de actividad. |
| properties.operationName | Nombre de la operación del evento del registro de actividad que hizo que se activara esta regla de alertas del registro de actividad. |
| properties.status | Estado del evento del registro de actividad que hizo que se activara esta regla de alertas del registro de actividad.|

#### <a name="properties-for-metric-alerts"></a>Propiedades de las alertas de métrica
| Nombre del elemento | Descripción |
| --- | --- |
| properties.RuleUri | Identificador de recurso de la propia regla de alerta de métrica. |
| properties.RuleName | Nombre de la regla de alertas de métrica. |
| properties.RuleDescription | Descripción de la regla de alertas de métrica (tal y como se define en la regla de alertas). |
| properties.Threshold | Valor de umbral que se usa en la evaluación de la regla de alertas de métrica. |
| properties.WindowSizeInMinutes | Tamaño de la ventana que se usa en la evaluación de la regla de alertas de métrica. |
| properties.Aggregation | Tipo de agregación definido en la regla de alertas de métrica. |
| properties.Operator | Operador condicional usado en la evaluación de la regla de alertas de métrica. |
| properties.MetricName | Nombre de la métrica usada en la evaluación de la regla de alertas de métrica. |
| properties.MetricUnit | Unidad de la métrica usada en la evaluación de la regla de alertas de métrica. |

## <a name="autoscale-category"></a>Categoría de escalado automático
Esta categoría contiene el registro de los eventos relacionados con el funcionamiento del motor de escalado automático en función de cualquier configuración de escalado automático que haya definido en la suscripción. Un ejemplo del tipo de evento que aparece en esta categoría es "Error de acción de escalado automático". Con el escalado automático puede escalar horizontalmente o reducir horizontalmente de forma automática el número de instancias de un tipo de recurso compatible en función de la hora del día o cargar datos (métricas) mediante una configuración de escalado automático. Cuando se cumplen las condiciones para escalar o reducir verticalmente, se registran los eventos de inicio y corrección o error en esta categoría.

### <a name="sample-event"></a>Evento de ejemplo
```json
{
  "caller": "Microsoft.Insights/autoscaleSettings",
  "channels": "Admin, Operation",
  "claims": {
    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/spn": "Microsoft.Insights/autoscaleSettings"
  },
  "correlationId": "fc6a7ff5-ff68-4bb7-81b4-3629212d03d0",
  "description": "The autoscale engine attempting to scale resource '/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ClassicCompute/domainNames/myResourceGroup/slots/Production/roles/myResource' from 3 instances count to 2 instances count.",
  "eventDataId": "a5b92075-1de9-42f1-b52e-6f3e4945a7c7",
  "eventName": {
    "value": "AutoscaleAction",
    "localizedValue": "AutoscaleAction"
  },
  "category": {
    "value": "Autoscale",
    "localizedValue": "Autoscale"
  },
  "id": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/microsoft.insights/autoscalesettings/myResourceGroup-Production-myResource-myResourceGroup/events/a5b92075-1de9-42f1-b52e-6f3e4945a7c7/ticks/636361956518681572",
  "level": "Informational",
  "resourceGroupName": "myResourceGroup",
  "resourceProviderName": {
    "value": "microsoft.insights",
    "localizedValue": "microsoft.insights"
  },
  "resourceId": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/microsoft.insights/autoscalesettings/myResourceGroup-Production-myResource-myResourceGroup",
  "resourceType": {
    "value": "microsoft.insights/autoscalesettings",
    "localizedValue": "microsoft.insights/autoscalesettings"
  },
  "operationId": "fc6a7ff5-ff68-4bb7-81b4-3629212d03d0",
  "operationName": {
    "value": "Microsoft.Insights/AutoscaleSettings/Scaledown/Action",
    "localizedValue": "Microsoft.Insights/AutoscaleSettings/Scaledown/Action"
  },
  "properties": {
    "Description": "The autoscale engine attempting to scale resource '/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ClassicCompute/domainNames/myResourceGroup/slots/Production/roles/myResource' from 3 instances count to 2 instances count.",
    "ResourceName": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.ClassicCompute/domainNames/myResourceGroup/slots/Production/roles/myResource",
    "OldInstancesCount": "3",
    "NewInstancesCount": "2",
    "LastScaleActionTime": "Fri, 21 Jul 2017 01:00:51 GMT"
  },
  "status": {
    "value": "Succeeded",
    "localizedValue": "Succeeded"
  },
  "subStatus": {
    "value": null
  },
  "eventTimestamp": "2017-07-21T01:00:51.8681572Z",
  "submissionTimestamp": "2017-07-21T01:00:52.3008754Z",
  "subscriptionId": "<subscription ID>"
}

```

### <a name="property-descriptions"></a>Descripciones de propiedades
| Nombre del elemento | Descripción |
| --- | --- |
| caller | Siempre es Microsoft.Insights/autoscaleSettings. |
| canales nueva | Siempre es "Admin, Operation". |
| claims | Blob JSON con el SPN (nombre de entidad de seguridad de servicio), o tipo de recurso, del motor de escalado automático. |
| correlationId | GUID en formato de cadena. |
| description |Descripción de texto estático del evento de escalado automático. |
| eventDataId |Identificador único del evento de escalado automático. |
| level |Nivel del evento. Uno de los valores siguientes: “Critical”, “Error”, “Warning” e “Informational” |
| resourceGroupName |Nombre del grupo de recursos para la configuración del escalado automático. |
| resourceProviderName |Nombre del proveedor de recursos para la configuración del escalado automático. |
| resourceId |Identificador de recurso de la configuración del escalado automático. |
| operationId |GUID compartido entre los eventos correspondientes a una sola operación. |
| operationName |Nombre de la operación. |
| properties |Conjunto de pares `<Key, Value>` (es decir, diccionario) que describen los detalles del evento. |
| properties.Description | Descripción detallada de lo que hacía el motor de escalado automático. |
| properties.ResourceName | Identificador del recurso afectado (recurso en el que se estaba llevando a cabo la acción de escalado) |
| properties.OldInstancesCount | Número de instancias antes de que tuviera efecto la acción de escalado automático. |
| properties.NewInstancesCount | Número de instancias después de que tuviera efecto la acción de escalado automático. |
| properties.LastScaleActionTime | Marca de tiempo de cuándo se produjo la acción de escalado automático. |
| status |Cadena que describe el estado de la operación. Entre los valores habituales, se incluyen: Started, In Progress, Succeeded, Failed, Active, Resolved. |
| subStatus | Normalmente es null para el escalado automático. |
| eventTimestamp |Marca de tiempo de cuándo el servicio de Azure generó el evento que procesó la solicitud correspondiente al evento. |
| submissionTimestamp |Marca de tiempo de cuándo el evento empezó a estar disponible para las consultas. |
| subscriptionId |Identificador de suscripción de Azure |

## <a name="security-category"></a>Categoría de seguridad
Esta categoría contiene el registro de todas las alertas generado por Azure Security Center. Un ejemplo del tipo de evento que vería en esta categoría es "Se ejecutó un archivo de extensión doble sospechoso".

### <a name="sample-event"></a>Evento de ejemplo
```json
{
    "channels": "Operation",
    "correlationId": "965d6c6a-a790-4a7e-8e9a-41771b3fbc38",
    "description": "Suspicious double extension file executed. Machine logs indicate an execution of a process with a suspicious double extension.\r\nThis extension may trick users into thinking files are safe to be opened and might indicate the presence of malware on the system.",
    "eventDataId": "965d6c6a-a790-4a7e-8e9a-41771b3fbc38",
    "eventName": {
        "value": "Suspicious double extension file executed",
        "localizedValue": "Suspicious double extension file executed"
    },
    "category": {
        "value": "Security",
        "localizedValue": "Security"
    },
    "eventTimestamp": "2017-10-18T06:02:18.6179339Z",
    "id": "/subscriptions/<subscription ID>/providers/Microsoft.Security/locations/centralus/alerts/965d6c6a-a790-4a7e-8e9a-41771b3fbc38/events/965d6c6a-a790-4a7e-8e9a-41771b3fbc38/ticks/636439033386179339",
    "level": "Informational",
    "operationId": "965d6c6a-a790-4a7e-8e9a-41771b3fbc38",
    "operationName": {
        "value": "Microsoft.Security/locations/alerts/activate/action",
        "localizedValue": "Microsoft.Security/locations/alerts/activate/action"
    },
    "resourceGroupName": "myResourceGroup",
    "resourceProviderName": {
        "value": "Microsoft.Security",
        "localizedValue": "Microsoft.Security"
    },
    "resourceType": {
        "value": "Microsoft.Security/locations/alerts",
        "localizedValue": "Microsoft.Security/locations/alerts"
    },
    "resourceId": "/subscriptions/<subscription ID>/providers/Microsoft.Security/locations/centralus/alerts/2518939942613820660_a48f8653-3fc6-4166-9f19-914f030a13d3",
    "status": {
        "value": "Active",
        "localizedValue": "Active"
    },
    "subStatus": {
        "value": null
    },
    "submissionTimestamp": "2017-10-18T06:02:52.2176969Z",
    "subscriptionId": "<subscription ID>",
    "properties": {
        "accountLogonId": "0x2r4",
        "commandLine": "c:\\mydirectory\\doubleetension.pdf.exe",
        "domainName": "hpc",
        "parentProcess": "unknown",
        "parentProcess id": "0",
        "processId": "6988",
        "processName": "c:\\mydirectory\\doubleetension.pdf.exe",
        "userName": "myUser",
        "UserSID": "S-3-2-12",
        "ActionTaken": "Detected",
        "Severity": "High"
    },
    "relatedEvents": []
}

```

### <a name="property-descriptions"></a>Descripciones de propiedades
| Nombre del elemento | Descripción |
| --- | --- |
| canales nueva | Siempre "Operation" |
| correlationId | GUID en formato de cadena. |
| description |Descripción de texto estático del evento de seguridad. |
| eventDataId |Identificador único del evento de seguridad. |
| eventName |Nombre descriptivo del evento de seguridad. |
| category | Siempre "Seguridad" |
| ID |Identificador único del recurso del evento de seguridad. |
| level |Nivel del evento. Uno de los valores siguientes: “Critical”, “Error”, “Warning” o “Informational” |
| resourceGroupName |Nombre del grupo de recursos del recurso. |
| resourceProviderName |Nombre del proveedor de recursos de Azure Security Center. Siempre "Microsoft.Security". |
| resourceType |Tipo de recurso que generó el evento de seguridad, por ejemplo, "Microsoft.Security/locations/alerts" |
| resourceId |Identificador de recurso de la alerta de seguridad. |
| operationId |GUID compartido entre los eventos correspondientes a una sola operación. |
| operationName |Nombre de la operación. |
| properties |Conjunto de pares `<Key, Value>` (es decir, diccionario) que describen los detalles del evento. Estas propiedades variarán según el tipo de alerta de seguridad. Vea [esta página](../../security-center/security-center-alerts-overview.md) para obtener una descripción de los tipos de alertas que proceden de Security Center. |
| properties.Severity |Nivel de gravedad. Los valores posibles son "High," "Medium" o "Low". |
| status |Cadena que describe el estado de la operación. Entre los valores habituales, se incluyen: Started, In Progress, Succeeded, Failed, Active, Resolved. |
| subStatus | Normalmente es null para los eventos de seguridad. |
| eventTimestamp |Marca de tiempo de cuándo el servicio de Azure generó el evento que procesó la solicitud correspondiente al evento. |
| submissionTimestamp |Marca de tiempo de cuándo el evento empezó a estar disponible para las consultas. |
| subscriptionId |Identificador de suscripción de Azure |

## <a name="recommendation-category"></a>Categoría de recomendaciones
Esta categoría contiene el registro de cualquier recomendación nueva que se genere para los servicios. Un ejemplo de recomendación sería "Use conjuntos de disponibilidad para obtener una tolerancia a errores mejorada". Hay cuatro tipos de eventos de recomendación que se pueden generar: Alta disponibilidad, Rendimiento, Seguridad y Optimización de costos. 

### <a name="sample-event"></a>Evento de ejemplo
```json
{
    "channels": "Operation",
    "correlationId": "92481dfd-c5bf-4752-b0d6-0ecddaa64776",
    "description": "The action was successful.",
    "eventDataId": "06cb0e44-111b-47c7-a4f2-aa3ee320c9c5",
    "eventName": {
        "value": "",
        "localizedValue": ""
    },
    "category": {
        "value": "Recommendation",
        "localizedValue": "Recommendation"
    },
    "eventTimestamp": "2018-06-07T21:30:42.976919Z",
    "id": "/SUBSCRIPTIONS/<Subscription ID>/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.COMPUTE/VIRTUALMACHINES/MYVM/events/06cb0e44-111b-47c7-a4f2-aa3ee320c9c5/ticks/636640038429769190",
    "level": "Informational",
    "operationId": "",
    "operationName": {
        "value": "Microsoft.Advisor/generateRecommendations/action",
        "localizedValue": "Microsoft.Advisor/generateRecommendations/action"
    },
    "resourceGroupName": "MYRESOURCEGROUP",
    "resourceProviderName": {
        "value": "MICROSOFT.COMPUTE",
        "localizedValue": "MICROSOFT.COMPUTE"
    },
    "resourceType": {
        "value": "MICROSOFT.COMPUTE/virtualmachines",
        "localizedValue": "MICROSOFT.COMPUTE/virtualmachines"
    },
    "resourceId": "/SUBSCRIPTIONS/<Subscription ID>/RESOURCEGROUPS/MYRESOURCEGROUP/PROVIDERS/MICROSOFT.COMPUTE/VIRTUALMACHINES/MYVM",
    "status": {
        "value": "Active",
        "localizedValue": "Active"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2018-06-07T21:30:42.976919Z",
    "subscriptionId": "<Subscription ID>",
    "properties": {
        "recommendationSchemaVersion": "1.0",
        "recommendationCategory": "Security",
        "recommendationImpact": "High",
        "recommendationRisk": "None"
    },
    "relatedEvents": []
}

```
### <a name="property-descriptions"></a>Descripciones de propiedades
| Nombre del elemento | Descripción |
| --- | --- |
| canales nueva | Siempre "Operation" |
| correlationId | GUID en formato de cadena. |
| description |Descripción de texto estático del evento de recomendación |
| eventDataId | Identificador único del evento de recomendación. |
| category | Siempre "Recomendación" |
| ID |Identificador único del recurso del evento de recomendación. |
| level |Nivel del evento. Uno de los valores siguientes: “Critical”, “Error”, “Warning” o “Informational” |
| operationName |Nombre de la operación.  Siempre "Microsoft.Advisor/generateRecommendations/action"|
| resourceGroupName |Nombre del grupo de recursos del recurso. |
| resourceProviderName |Nombre del proveedor de recursos del recurso al que se aplica esta recomendación, como "MICROSOFT.COMPUTE" |
| resourceType |Nombre del tipo de recurso del recurso al que se aplica esta recomendación, como "MICROSOFT.COMPUTE/virtualmachines" |
| resourceId |Identificador de recurso del recurso al que se aplica la recomendación |
| status | Siempre "Activo" |
| submissionTimestamp |Marca de tiempo de cuándo el evento empezó a estar disponible para las consultas. |
| subscriptionId |Identificador de suscripción de Azure |
| properties |Conjunto de pares `<Key, Value>` (es decir, diccionario) que describe los detalles de la recomendación.|
| properties.recommendationSchemaVersion| Versión de esquema de las propiedades de recomendación publicadas en la entrada Registro de actividad |
| properties.recommendationCategory | Categoría de la recomendación. Los valores posibles son "High Availability", "Performance", "Security" y "Cost" |
| properties.recommendationImpact| Impacto de la recomendación. Los valores posibles son "High", "Medium", "Low" |
| properties.recommendationRisk| Riesgo de la recomendación. Los valores posibles son "Error", "Warning", "None" |

## <a name="policy-category"></a>Categoría de directivas

Esta categoría contiene registros de todas las operaciones de acción de efecto realizadas por [Azure Policy](../../governance/policy/overview.md). Algunos ejemplos de los tipos de eventos que aparecen en esta categoría son _Auditoría_ y _Denegación_. Cada acción llevada a cabo por Azure Policy se modela como una operación en un recurso.

### <a name="sample-policy-event"></a>Evento de Azure Policy de ejemplo

```json
{
    "authorization": {
        "action": "Microsoft.Resources/checkPolicyCompliance/read",
        "scope": "/subscriptions/<subscriptionID>"
    },
    "caller": "33a68b9d-63ce-484c-a97e-94aef4c89648",
    "channels": "Operation",
    "claims": {
        "aud": "https://management.azure.com/",
        "iss": "https://sts.windows.net/1114444b-7467-4144-a616-e3a5d63e147b/",
        "iat": "1234567890",
        "nbf": "1234567890",
        "exp": "1234567890",
        "aio": "A3GgTJdwK4vy7Fa7l6DgJC2mI0GX44tML385OpU1Q+z+jaPnFMwB",
        "appid": "1d78a85d-813d-46f0-b496-dd72f50a3ec0",
        "appidacr": "2",
        "http://schemas.microsoft.com/identity/claims/identityprovider": "https://sts.windows.net/1114444b-7467-4144-a616-e3a5d63e147b/",
        "http://schemas.microsoft.com/identity/claims/objectidentifier": "f409edeb-4d29-44b5-9763-ee9348ad91bb",
        "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "b-24Jf94A3FH2sHWVIFqO3-RSJEiv24Jnif3gj7s",
        "http://schemas.microsoft.com/identity/claims/tenantid": "1114444b-7467-4144-a616-e3a5d63e147b",
        "uti": "IdP3SUJGtkGlt7dDQVRPAA",
        "ver": "1.0"
    },
    "correlationId": "b5768deb-836b-41cc-803e-3f4de2f9e40b",
    "description": "",
    "eventDataId": "d0d36f97-b29c-4cd9-9d3d-ea2b92af3e9d",
    "eventName": {
        "value": "EndRequest",
        "localizedValue": "End request"
    },
    "category": {
        "value": "Policy",
        "localizedValue": "Policy"
    },
    "eventTimestamp": "2019-01-15T13:19:56.1227642Z",
    "id": "/subscriptions/<subscriptionID>/resourceGroups/myResourceGroup/providers/Microsoft.Sql/servers/contososqlpolicy/events/13bbf75f-36d5-4e66-b693-725267ff21ce/ticks/636831551961227642",
    "level": "Warning",
    "operationId": "04e575f8-48d0-4c43-a8b3-78c4eb01d287",
    "operationName": {
        "value": "Microsoft.Authorization/policies/audit/action",
        "localizedValue": "Microsoft.Authorization/policies/audit/action"
    },
    "resourceGroupName": "myResourceGroup",
    "resourceProviderName": {
        "value": "Microsoft.Sql",
        "localizedValue": "Microsoft SQL"
    },
    "resourceType": {
        "value": "Microsoft.Resources/checkPolicyCompliance",
        "localizedValue": "Microsoft.Resources/checkPolicyCompliance"
    },
    "resourceId": "/subscriptions/<subscriptionID>/resourceGroups/myResourceGroup/providers/Microsoft.Sql/servers/contososqlpolicy",
    "status": {
        "value": "Succeeded",
        "localizedValue": "Succeeded"
    },
    "subStatus": {
        "value": "",
        "localizedValue": ""
    },
    "submissionTimestamp": "2019-01-15T13:20:17.1077672Z",
    "subscriptionId": "<subscriptionID>",
    "properties": {
        "isComplianceCheck": "True",
        "resourceLocation": "westus2",
        "ancestors": "72f988bf-86f1-41af-91ab-2d7cd011db47",
        "policies": "[{\"policyDefinitionId\":\"/subscriptions/<subscriptionID>/providers/Microsoft.
            Authorization/policyDefinitions/5775cdd5-d3d3-47bf-bc55-bb8b61746506/\",\"policyDefiniti
            onName\":\"5775cdd5-d3d3-47bf-bc55-bb8b61746506\",\"policyDefinitionEffect\":\"Deny\",\"
            policyAssignmentId\":\"/subscriptions/<subscriptionID>/providers/Microsoft.Authorization
            /policyAssignments/991a69402a6c484cb0f9b673/\",\"policyAssignmentName\":\"991a69402a6c48
            4cb0f9b673\",\"policyAssignmentScope\":\"/subscriptions/<subscriptionID>\",\"policyAssig
            nmentParameters\":{}}]"
    },
    "relatedEvents": []
}
```

### <a name="policy-event-property-descriptions"></a>Descripciones de las propiedades del evento de Azure Policy

| Nombre del elemento | Descripción |
| --- | --- |
| authorization | Matriz de propiedades RBAC del evento. Para los nuevos recursos, se trata de la acción y el ámbito de la solicitud que activó la evaluación. Para los recursos existentes, la acción es "Microsoft.Resources/checkPolicyCompliance/read". |
| caller | Para los nuevos recursos, la identidad que ha iniciado una implementación. Para los recursos existentes, el GUID de Microsoft Azure Policy Insights RP. |
| canales nueva | Los eventos de Azure Policy utilizan solo el canal "Operation". |
| claims | Token de JWT que se usa en Active Directory para autenticar el usuario o la aplicación para llevar a cabo esta operación en Resource Manager. |
| correlationId | Normalmente, un GUID en formato de cadena. Los eventos que comparten correlationId pertenecen a la misma acción general. |
| description | Este campo está en blanco para los eventos de Azure Policy. |
| eventDataId | Identificador único de un evento. |
| eventName | "BeginRequest" o "EndRequest". "BeginRequest" se usa para las evaluaciones auditIfNotExists y deployIfNotExists diferidas y cuando un efecto deployIfNotExists inicia una implementación de plantilla. Todas las demás operaciones devuelven "EndRequest". |
| category | Declara que el evento de registro de actividad pertenece a "Azure Policy". |
| eventTimestamp | Marca de tiempo de cuándo el servicio de Azure generó el evento que procesó la solicitud correspondiente al evento. |
| ID | Identificador único del evento en el recurso específico. |
| level | Nivel del evento. Auditoría usa "Warning" y la denegación usa "Error". Un error auditIfNotExists o deployIfNotExists puede generar un mensaje "Warning" o "Error" según la gravedad. Todos los demás eventos de Azure Policy utilizan "Informational". |
| operationId | GUID compartido entre los eventos correspondientes a una sola operación. |
| operationName | Nombre de la operación que se correlaciona directamente con el efecto de Azure Policy. |
| resourceGroupName | Nombre del grupo de recursos del recurso evaluado. |
| resourceProviderName | Nombre del proveedor de recursos del recurso evaluado. |
| resourceType | Para los nuevos recursos, es el tipo que se va a evaluar. Para los recursos existentes, devuelve "Microsoft.Resources/checkPolicyCompliance". |
| resourceId | Identificador de recurso del recurso evaluado. |
| status | Cadena que describe el estado del resultado de evaluación de Azure Policy. La mayoría de las evaluaciones de Azure Policy devuelven "Succeeded", pero un efecto de denegación devuelve "Failed". Los errores de auditIfNotExists o deployIfNotExists también devuelven "Failed". |
| subStatus | Este campo está en blanco para los eventos de Policy. |
| submissionTimestamp | Marca de tiempo de cuándo el evento empezó a estar disponible para las consultas. |
| subscriptionId | Identificador de suscripción de Azure |
| properties.isComplianceCheck | Devuelve "False" cuando se implementa un nuevo recurso o se actualizan las propiedades de Resource Manager de un recurso existente. Todos los demás [desencadenadores de evaluación](../../governance/policy/how-to/get-compliance-data.md#evaluation-triggers) dan "True" como resultado. |
| properties.resourceLocation | Región de Azure del recurso que se va a evaluar. |
| properties.ancestors | Lista separada por comas que contiene los grupos de administración primarios ordenados del primario directo al principal más lejano. |
| properties.policies | Incluye detalles sobre la definición, asignación, efecto y parámetros de directiva que ocasionaron esta evaluación de Azure Policy. |
| relatedEvents | Este campo está en blanco para los eventos de Azure Policy. |


## <a name="schema-from-storage-account-and-event-hubs"></a>Esquema de una cuenta de almacenamiento y un centro de eventos
Cuando el registro de actividad de Azure se transmite a una cuenta de almacenamiento o a un centro de eventos, los datos siguen el [esquema de registros de recursos](./resource-logs-schema.md). En la tabla siguiente, se muestra la correspondencia de las propiedades entre los esquemas anteriores y el esquema de los registros de recursos.

> [!IMPORTANT]
> El formato de los datos de registro de actividad escritos en una cuenta de almacenamiento cambiaron a Líneas JSON el 1 de noviembre de 2018. Consulte [Preparación para el cambio de formato a los registros de recursos de Azure Monitor archivados en una cuenta de almacenamiento](./resource-logs-blob-format.md) para más información sobre este cambio de formato.


| Propiedad del esquema de registros de recursos | Propiedad del esquema de API REST de registro de actividad | Notas |
| --- | --- | --- |
| time | eventTimestamp |  |
| resourceId | resourceId | subscriptionId, resourceType, resourceGroupName se deducen todos de resourceId. |
| operationName | operationName.value |  |
| category | Parte del nombre de la operación | Desglose del tipo de operación: "Write"/"Delete"/"Action" |
| resultType | status.value | |
| resultSignature | substatus.value | |
| resultDescription | description |  |
| durationMs | N/D | Siempre 0 |
| callerIpAddress | httpRequest.clientIpAddress |  |
| correlationId | correlationId |  |
| identity | notificaciones y propiedades de autorización |  |
| Nivel | Nivel |  |
| ubicación | N/D | Ubicación de donde se procesó el evento. *Esto no es la ubicación del recurso, sino el lugar donde se procesó el evento. Esta propiedad se quitará en una futura actualización.* |
| Propiedades | properties.eventProperties |  |
| properties.eventCategory | category | Si properties.eventCategory no está presente, la categoría será "Administrative" |
| properties.eventName | eventName |  |
| properties.operationId | operationId |  |
| properties.eventProperties | properties |  |

A continuación, se muestra un ejemplo de un evento que utiliza este esquema.

``` JSON
{
    "records": [
        {
            "time": "2019-01-21T22:14:26.9792776Z",
            "resourceId": "/subscriptions/s1/resourceGroups/MSSupportGroup/providers/microsoft.support/supporttickets/115012112305841",
            "operationName": "microsoft.support/supporttickets/write",
            "category": "Write",
            "resultType": "Success",
            "resultSignature": "Succeeded.Created",
            "durationMs": 2826,
            "callerIpAddress": "111.111.111.11",
            "correlationId": "c776f9f4-36e5-4e0e-809b-c9b3c3fb62a8",
            "identity": {
                "authorization": {
                    "scope": "/subscriptions/s1/resourceGroups/MSSupportGroup/providers/microsoft.support/supporttickets/115012112305841",
                    "action": "microsoft.support/supporttickets/write",
                    "evidence": {
                        "role": "Subscription Admin"
                    }
                },
                "claims": {
                    "aud": "https://management.core.windows.net/",
                    "iss": "https://sts.windows.net/72f988bf-86f1-41af-91ab-2d7cd011db47/",
                    "iat": "1421876371",
                    "nbf": "1421876371",
                    "exp": "1421880271",
                    "ver": "1.0",
                    "http://schemas.microsoft.com/identity/claims/tenantid": "00000000-0000-0000-0000-000000000000",
                    "http://schemas.microsoft.com/claims/authnmethodsreferences": "pwd",
                    "http://schemas.microsoft.com/identity/claims/objectidentifier": "2468adf0-8211-44e3-95xq-85137af64708",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn": "admin@contoso.com",
                    "puid": "20030000801A118C",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "9vckmEGF7zDKk1YzIY8k0t1_EAPaXoeHyPRn6f413zM",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname": "John",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname": "Smith",
                    "name": "John Smith",
                    "groups": "cacfe77c-e058-4712-83qw-f9b08849fd60,7f71d11d-4c41-4b23-99d2-d32ce7aa621c,31522864-0578-4ea0-9gdc-e66cc564d18c",
                    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name": " admin@contoso.com",
                    "appid": "c44b4083-3bq0-49c1-b47d-974e53cbdf3c",
                    "appidacr": "2",
                    "http://schemas.microsoft.com/identity/claims/scope": "user_impersonation",
                    "http://schemas.microsoft.com/claims/authnclassreference": "1"
                }
            },
            "level": "Information",
            "location": "global",
            "properties": {
                "statusCode": "Created",
                "serviceRequestId": "50d5cddb-8ca0-47ad-9b80-6cde2207f97c"
            }
        }
    ]
}
```





## <a name="next-steps"></a>Pasos siguientes
* [Más información sobre el registro de actividad](platform-logs-overview.md)
* [Creación de una configuración de diagnóstico para enviar el registro de actividad a un área de trabajo de Log Analytics, Azure Storage o Event Hubs](diagnostic-settings.md)