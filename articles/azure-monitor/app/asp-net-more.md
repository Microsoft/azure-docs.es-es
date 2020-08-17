---
title: Más información sobre Azure Application Insights | Microsoft Docs
description: Aquí se presentan algunas características que puede explorar una vez que sepa utilizar Application Insights.
ms.topic: conceptual
ms.date: 02/03/2017
ms.openlocfilehash: c0be377636159dbb2bdb628af6a0bc9077942397
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87321350"
---
# <a name="more-telemetry-from-application-insights"></a>Más telemetría desde Application Insights
Una vez que haya [agregado Application Insights a su código de ASP.NET](./asp-net.md), puede realizar varias acciones para obtener más datos de telemetría. 

| Acción | Lo que obtiene|
|---|---|
|(Servidores IIS) [Instale Monitor de estado](https://go.microsoft.com/fwlink/?LinkId=506648) en cada máquina del servidor.<br/>(Aplicaciones web de Azure) En el panel de control de Azure de la aplicación web, abra la hoja Application Insights.| [**Contadores de rendimiento**](./performance-counters.md).<br/>[**Excepciones**](asp-net-exceptions.md): seguimiento de la pila detallado.<br/>[**Dependencias**](./asp-net-dependencies.md).|
|[Agregue el fragmento de código de JavaScript en las páginas web](./javascript.md).|[Rendimiento de página](./usage-overview.md), excepciones de explorador y rendimiento de AJAX. Telemetría del lado cliente personalizada.|
|[Cree pruebas web de disponibilidad](./monitor-web-app-availability.md).|Obtención de alertas si el sitio deja de estar disponible.|
|[Asegúrese de que se genera buildinfo.config](/visualstudio/debugger/diagnose-problems-after-deployment?view=vs-2015) mediante MSBuild.|[Generación de anotaciones en gráficos de métricas](./annotations.md)
|[Cree eventos y métricas personalizados](./api-custom-events-metrics.md).|Recuento de métricas y eventos empresariales, seguimiento de uso detallado y mucho más.|
|[Genere un perfil del sitio activo.](https://aka.ms/AIProfilerPreview)|Intervalos de función detallados de la aplicación web activa.|

