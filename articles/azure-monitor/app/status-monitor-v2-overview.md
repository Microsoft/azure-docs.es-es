---
title: Información general sobre Azure Monitor de estado v2 | Microsoft Docs
description: Información general del Monitor de estado v2. Supervisar el rendimiento del sitio Web sin volver a implementar el sitio Web. Funciona con las aplicaciones web de ASP.NET hospedadas en local, en las máquinas virtuales o en Azure.
services: application-insights
documentationcenter: .net
author: MS-TimothyMothra
manager: alexklim
ms.assetid: 769a5ea4-a8c6-4c18-b46c-657e864e24de
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: tilee
ms.openlocfilehash: 976545f588895421e7c2be7e18b844d7994bc1ff
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65779987"
---
# <a name="status-monitor-v2"></a>Monitor de estado v2

V2 de Monitor de estado es un módulo de PowerShell que se publican en el [PowerShellGallery](https://www.powershellgallery.com/packages/Az.ApplicationMonitor) y es el reemplazo de [Monitor de estado](https://docs.microsoft.com/azure/azure-monitor/app/monitor-performance-live-website-now). Este módulo proporciona instrumentación de menos de código de aplicaciones web de .NET hospedado con IIS.
Datos de telemetría se enviarán a Azure portal donde puede [monitor](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview) la aplicación.

> [!IMPORTANT]
> V2 de Monitor de estado está actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para obtener más información, consulte [términos de uso complementarios para vistas previas de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)

## <a name="powershell-gallery"></a>Galería de PowerShell

https://www.powershellgallery.com/packages/Az.ApplicationMonitor


## <a name="instructions"></a>Instrucciones
- Revise nuestra [instrucciones de introducción](status-monitor-v2-get-started.md) para empezar a trabajar ahora con los ejemplos de código conciso.
- Revise nuestra [instrucciones detalladas](status-monitor-v2-detailed-instructions.md) para profundizar en cómo empezar a trabajar.

## <a name="powershell-api-reference"></a>Referencia de API de PowerShell
- [Disable-ApplicationInsightsMonitoring](status-monitor-v2-api-disable-monitoring.md)
- [Disable-InstrumentationEngine](status-monitor-v2-api-disable-instrumentation-engine.md)
- [Enable-ApplicationInsightsMonitoring](status-monitor-v2-api-enable-monitoring.md)
- [Enable-InstrumentationEngine](status-monitor-v2-api-enable-instrumentation-engine.md)
- [Get-ApplicationInsightsMonitoringConfig](status-monitor-v2-api-get-config.md)
- [Get-ApplicationInsightsMonitoringStatus](status-monitor-v2-api-get-status.md)
- [Set-ApplicationInsightsMonitoringConfig](status-monitor-v2-api-set-config.md)

## <a name="troubleshooting"></a>solución de problemas
- [Solución de problemas](status-monitor-v2-troubleshoot.md)
- [Problemas conocidos](status-monitor-v2-troubleshoot.md#known-issues)


## <a name="faq"></a>Preguntas más frecuentes

- ¿V2 del Monitor de estado compatible con las instalaciones del proxy?

  **Sí**. Tiene varias opciones para descargar el Monitor de estado v2. Si el equipo tiene acceso a internet, puede incorporar a la Galería de PowerShell mediante `-Proxy` parámetros. Puede descargar este módulo también manualmente e instalarlo en su equipo o usar el módulo directamente. Cada una de estas opciones se describe en nuestra [instrucciones detalladas](status-monitor-v2-detailed-instructions.md).
  
- ¿Cómo comprobar la habilitación fue correcta?

   No hay un cmdlet para comprobar que la habilitación se realizó correctamente. Se recomienda usar [Live Metrics](https://docs.microsoft.com/azure/azure-monitor/app/live-stream) a observar rápidamente si la aplicación enviaba telemetría.

   También puede usar [Analytics](../log-query/get-started-portal.md) para enumerar todos los roles en la nube actualmente enviando telemetría.
   ```Kusto
   union * | summarize count() by cloud_RoleName, cloud_RoleInstance
   ```

## <a name="next-steps"></a>Pasos siguientes

Vea la telemetría:

* [Explore las métricas](../../azure-monitor/app/metrics-explorer.md) para supervisar el rendimiento y uso.
* [Busque eventos y registros](../../azure-monitor/app/diagnostic-search.md) para diagnosticar problemas
* [Análisis](../../azure-monitor/app/analytics.md) para más consultas avanzadas
* [Creación de paneles](../../azure-monitor/app/app-insights-dashboards.md)

Agregue más telemetría:

* [Cree pruebas web](monitor-web-app-availability.md) para asegurarse de que el sitio permanece activo.
* [Agregue telemetría de cliente web](../../azure-monitor/app/javascript.md) para ver las excepciones de código de la página web y para que le permitan insertar llamadas de seguimiento.
* [Agregue el SDK de Application Insights al código](../../azure-monitor/app/asp-net.md) para que pueda insertar seguimiento y registrar las llamadas

