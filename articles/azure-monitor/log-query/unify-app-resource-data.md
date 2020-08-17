---
title: Unificación de varios recursos de Application Insights de Azure Monitor | Microsoft Docs
description: En este artículo se proporcionan detalles sobre cómo usar una función en registros de Azure Monitor para consultar varios recursos de Application Insights y visualizar los datos.
author: bwren
ms.author: bwren
ms.workload: na
ms.tgt_pltfrm: na
ms.topic: conceptual
ms.date: 02/02/2020
ms.openlocfilehash: 40ce2844e33c9a71f87e434a6a3e9f8e0f7e3cc6
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322115"
---
# <a name="unify-multiple-azure-monitor-application-insights-resources"></a>Unificación de varios recursos de Application Insights de Azure Monitor 
En este artículo se describe el proceso para consultar y ver todos los datos de registro de Application Insights en un mismo lugar (incluso si pertenecen a suscripciones a Azure distintas) como reemplazo de Application Insights Connector, que se encuentra en desuso. El número de recursos de Application Insights que se pueden incluir en una sola consulta se limita a 100.

## <a name="recommended-approach-to-query-multiple-application-insights-resources"></a>Enfoque recomendado para consultar varios recursos de Application Insights 
Enumerar varios recursos de Application Insights en una consulta puede ser pesado y difícil de mantener. En su lugar, puede aprovechar la función para separar la lógica de consulta del ámbito de las aplicaciones.  

En este ejemplo se muestra cómo puede supervisar varios recursos de Application Insights y visualizar el número de solicitudes erróneas por nombre de la aplicación.

Cree una función mediante el operador de unión con la lista de aplicaciones y guarde la consulta como función con el alias *applicationsScoping* en el área de trabajo. 

Puede modificar las aplicaciones enumeradas en cualquier momento en el portal; para ello, vaya al explorador de consultas en el área de trabajo y seleccione la función para editarla y guardarla, o use el cmdlet `SavedSearch` de PowerShell. 

>[!NOTE]
>Este método no se puede usar con alertas de registro porque la validación de acceso de los recursos para las regla de alertas (incluidas las áreas de trabajo y las aplicaciones) se realiza en el momento de la creación de la alerta. No se admite la adición de nuevos recursos a la función después de crear la alerta. Si prefiere usar la función para el ámbito de los recursos en las alertas de registro, debe editar la regla de alertas en el portal o usar una plantilla de Resource Manager para actualizar los recursos de ámbito. También puede incluir la lista de recursos en la consulta de alerta de registro.

El comando `withsource= SourceApp` agrega una columna a los resultados, que designa la aplicación que envió el registro. El operador de análisis es opcional en este ejemplo y se usa para extraer el nombre de la aplicación de la propiedad SourceApp. 

```
union withsource=SourceApp 
app('Contoso-app1').requests,  
app('Contoso-app2').requests, 
app('Contoso-app3').requests, 
app('Contoso-app4').requests, 
app('Contoso-app5').requests 
| parse SourceApp with * "('" applicationName "')" *  
```

Ahora está listo para usar la función applicationsScoping en la consulta entre recursos:  

```
applicationsScoping 
| where timestamp > ago(12h)
| where success == 'False'
| parse SourceApp with * '(' applicationName ')' * 
| summarize count() by applicationName, bin(timestamp, 1h) 
| render timechart
```

La consulta usa el esquema de Application Insights, aunque la consulta se ejecuta en el área de trabajo debido a que la función applicationsScoping devuelve la estructura de datos de Application Insights. El alias de la función devuelve la unión de las solicitudes de todas las aplicaciones definidas. A continuación, la consulta filtra las solicitudes erróneas y visualiza las tendencias por aplicación.

![Ejemplo de resultados entre consultas](media/unify-app-resource-data/app-insights-query-results.png)

>[!NOTE]
>La [consulta entre recursos](./cross-workspace-query.md) en las alertas de registro se admite en la nueva [API scheduledQueryRules](/rest/api/monitor/scheduledqueryrules). De forma predeterminada, Azure Monitor usa la [API de alerta heredada de Log Analytics](../platform/api-alerts.md) para crear nuevas reglas de alerta de registro desde Azure Portal, a menos que cambie de [API de alerta de registro heredada](../platform/alerts-log-api-switch.md#process-of-switching-from-legacy-log-alerts-api). Después del cambio, la nueva API se convierte en la predeterminada para las nuevas reglas de alerta en Azure Portal y le permite crear reglas de alertas de registro de consulta entre recursos. Puede crear reglas de alertas de registro de [consulta entre recursos](./cross-workspace-query.md) sin realizar el cambio mediante la [plantilla de ARM de la API scheduledQueryRules](../platform/alerts-log.md#log-alert-with-cross-resource-query-using-azure-resource-template); sin embargo, esta regla de alertas se puede administrar mediante la [API scheduledQueryRules](/rest/api/monitor/scheduledqueryrules) y no desde Azure Portal.

## <a name="application-insights-and-log-analytics-workspace-schema-differences"></a>Diferencias de esquema del área de trabajo de Log Analytics y Application Insights
En la tabla siguiente se muestran las diferencias de esquema entre Log Analytics y Application Insights.  

| Propiedades del área de trabajo de Log Analytics| Propiedades de recursos de Application Insights|
|------------|------------| 
| AnonUserId | user_id|
| ApplicationId | appId|
| ApplicationName | appName|
| ApplicationTypeVersion | application_Version |
| AvailabilityCount | itemCount |
| AvailabilityDuration | duration |
| AvailabilityMessage | message |
| AvailabilityRunLocation | ubicación |
| AvailabilityTestId | id |
| AvailabilityTestName | name |
| AvailabilityTimestamp | timestamp |
| Browser | client_browser |
| City | client_city |
| ClientIP | client_IP |
| Computer | cloud_RoleInstance | 
| Country | client_CountryOrRegion | 
| CustomEventCount | itemCount | 
| CustomEventDimensions | customDimensions |
| CustomEventName | name | 
| DeviceModel | client_Model | 
| DeviceType | client_Type | 
| ExceptionCount | itemCount | 
| ExceptionHandledAt | handledAt |
| ExceptionMessage | message | 
| ExceptionType | type |
| OperationID | operation_id |
| OperationName | operation_Name | 
| SO | client_OS | 
| PageViewCount | itemCount |
| PageViewDuration | duration | 
| PageViewName | name | 
| ParentOperationID | operation_Id | 
| RequestCount | itemCount | 
| RequestDuration | duration | 
| RequestID | id | 
| RequestName | name | 
| RequestSuccess | success | 
| ResponseCode | resultCode | 
| Role | cloud_RoleName |
| RoleInstance | cloud_RoleInstance |
| SessionId | session_Id | 
| SourceSystem | operation_SyntheticSource |
| TelemetryTYpe | type |
| URL | url |
| UserAccountId | user_AccountId |

## <a name="next-steps"></a>Pasos siguientes

Use [Búsqueda de registros](./log-query-overview.md) para ver información detallada de las aplicaciones de Application Insights.

