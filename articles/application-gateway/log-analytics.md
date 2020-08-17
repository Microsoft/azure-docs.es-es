---
title: Examen de los registros de WAF con Azure Log Analytics
titleSuffix: Azure Application Gateway
description: En este artículo se muestra cómo usar Azure Log Analytics para examinar registros del firewall de aplicaciones web de Application Gateway
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: troubleshooting
ms.date: 11/14/2019
ms.author: victorh
ms.openlocfilehash: 88e360c5630c24dd997f72bda3fc4a480264763d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87086921"
---
# <a name="use-log-analytics-to-examine-application-gateway-web-application-firewall-waf-logs"></a>Uso de Log Analytics para examinar registros del firewall de aplicaciones web (WAF) de Application Gateway

Una vez que el WAF de Application Gateway está operativo, puede habilitar registros para inspeccionar lo que sucede con cada solicitud. Los registros del firewall proporcionan información sobre lo que el WAF evalúa, empareja y bloquea. Con Log Analytics, puede examinar los datos de los registros del firewall para ofrecer aún más información. Para obtener más información sobre la creación de un área de trabajo de Log Analytics, vea [Creación de un área de trabajo de Log Analytics en Azure Portal](../azure-monitor/learn/quick-create-workspace.md). Para obtener más información sobre las consultas de registro, vea [Análisis de datos de registro en Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

## <a name="import-waf-logs"></a>Importar registros de WAF

Para importar registros del firewall en Log Analytics, vea [Mantenimiento del back-end, Registros de diagnóstico y Métricas de Application Gateway](application-gateway-diagnostics.md#diagnostic-logging). Cuando tenga los registros del firewall en el área de trabajo de Log Analytics, puede ver datos, escribir consultas, crear visualizaciones y agregarlas al panel del portal.

## <a name="explore-data-with-examples"></a>Explorar datos con ejemplos

Para ver los datos sin procesar del registro del firewall, puede ejecutar la consulta siguiente:

```
AzureDiagnostics 
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
```

Su aspecto es similar al de la siguiente consulta:

![Consulta de Log Analytics](media/log-analytics/log-query.png)

Puede explorar en profundidad los datos y trazar gráficos o crear visualizaciones desde aquí. Vea las siguientes consultas como punto de partida:

### <a name="matchedblocked-requests-by-ip"></a>Solicitudes emparejadas o bloqueadas por IP

```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize count() by clientIp_s, bin(TimeGenerated, 1m)
| render timechart
```

### <a name="matchedblocked-requests-by-uri"></a>Solicitudes emparejadas o bloqueadas por URI

```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize count() by requestUri_s, bin(TimeGenerated, 1m)
| render timechart
```

### <a name="top-matched-rules"></a>Principales reglas emparejadas

```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize count() by ruleId_s, bin(TimeGenerated, 1m)
| where count_ > 10
| render timechart
```

### <a name="top-five-matched-rule-groups"></a>Principales cinco grupos de reglas emparejadas

```
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayFirewallLog"
| summarize Count=count() by details_file_s, action_s
| top 5 by Count desc
| render piechart
```

## <a name="add-to-your-dashboard"></a>Agregar al panel

Una vez que se crea una consulta, se puede agregar al panel.  Seleccione **Anclar al panel** en la parte superior derecha del área de trabajo de Log Analytics. Con las cuatro consultas anteriores ancladas a un panel de ejemplo, estos son los datos que se pueden ver de un vistazo:

![Panel](media/log-analytics/dashboard.png)

## <a name="next-steps"></a>Pasos siguientes

[Mantenimiento del back-end, Registros de diagnóstico y Métricas de Application Gateway](application-gateway-diagnostics.md)
