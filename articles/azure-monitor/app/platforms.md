---
title: 'Application Insights: lenguajes, plataformas e integraciones| Microsoft Docs'
description: Lenguajes y plataformas e integraciones disponibles para Application Insights
services: application-insights
documentationcenter: ''
author: mrbullwinkle
manager: carmonm
ms.assetid: 974db106-54ff-4318-9f8b-f7b3a869e536
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.topic: conceptual
ms.date: 05/16/2019
ms.reviewer: olegan
ms.author: mbullwin
ms.openlocfilehash: 4a0dbe736b9727491f77b392c199400003abf8be
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65779883"
---
# <a name="developer-analytics-languages-platforms-and-integrations"></a>Análisis del desarrollador: lenguajes y plataformas e integraciones
Estos elementos son implementaciones de [Application Insights](../../azure-monitor/app/app-insights-overview.md) sobre las que tenemos noticias, que incluyen también algunas de terceros.

## <a name="languages---officially-supported-by-application-insights-team"></a>Lenguajes: oficialmente admitidos por el equipo de Application Insights
* [C#|VB (.NET)](../../azure-monitor/app/asp-net.md)
* [Java](../../azure-monitor/app/java-get-started.md)
* [Páginas web de JavaScript](../../azure-monitor/app/javascript.md)
* [Node.JS](../../azure-monitor/app/nodejs.md)

## <a name="languages---community-supported"></a>Lenguajes: admitidos por la comunidad
* [F#](https://safe-stack.github.io/docs/template-azure-ai/)
* [PHP](https://github.com/Microsoft/ApplicationInsights-PHP)
* [Python](https://pypi.python.org/pypi/applicationinsights/0.1.0)
* [Ruby](https://rubygems.org/gems/application_insights)
* [Nada más](#projects)

## <a name="platforms-and-frameworks"></a>Plataformas y marcos
* [ASP.NET](../../azure-monitor/app/asp-net.md)
* [ASP.NET: para aplicaciones que ya están activas](../../azure-monitor/app/monitor-performance-live-website-now.md)
* [ASP.NET Core](../../azure-monitor/app/asp-net-core.md)
* [Android](../../azure-monitor/learn/mobile-center-quickstart.md) (App Center)
* [Android](https://github.com/Microsoft/ApplicationInsights-Android) (App Center)
* [Angular](https://github.com/MarkPieszak/angular-application-insights)
* [Azure App Service](../../azure-monitor/app/azure-web-apps.md)
* [Azure Cloud Services](../../azure-monitor/app/cloudservices.md)&amp;#151;incluidos los roles web y de trabajo
* [Funciones de Azure](https://docs.microsoft.com/azure/azure-functions/functions-monitoring)
* [Docker](../../azure-monitor/app/docker.md)
* [Glimpse](https://azure.microsoft.com/blog/glimpse-application-insights/)
* [iOS](../../azure-monitor/learn/mobile-center-quickstart.md) (App Center)
* [Ionic](https://github.com/SoftwarePioniere/ionic-application-insights)
* [iOS](https://github.com/Microsoft/ApplicationInsights-iOS) (App Center)
* [Java EE](../../azure-monitor/app/java-get-started.md)
* [Node.JS](https://www.npmjs.com/package/applicationinsights)
* [OSX](https://github.com/Microsoft/ApplicationInsights-OSX)
* [SAFE Stack](https://safe-stack.github.io/docs/template-azure-ai/)
* [Spring](https://joe.blog.freemansoft.com/2015/12/enabling-microsoft-application-insight.html)
* [Aplicación Windows universal](../../azure-monitor/learn/mobile-center-quickstart.md) (App Center)
* [WCF](https://github.com/Microsoft/ApplicationInsights-SDK-Labs/blob/master/WCF/readme.md)
* [Aplicaciones, servicios y roles de trabajo del escritorio de Windows](../../azure-monitor/app/windows-desktop.md)
* [Nada más](#projects)

## <a name="logging-frameworks"></a>Marcos de registro
* [ILogger](https://docs.microsoft.com/azure/azure-monitor/app/ilogger)
* [Log4Net, NLog o System.Diagnostics.Trace](../../azure-monitor/app/asp-net-trace-logs.md)
* [Java, Log4J o Logback](../../azure-monitor/app/java-trace-logs.md)
* El [Registro semántico (SLAB)](https://github.com/fidmor89/SLAB_AppInsights) se integra con el [Bloque de aplicación de registro semántico](https://msdn.microsoft.com/library/dn440729.aspx)
* [Pruebas de carga en la nube](https://blogs.msdn.com/b/visualstudioalm/archive/2015/07/30/getting-application-insights-counters-with-cloud-based-load-testing.aspx)
* [Complemento LogStash](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-output-applicationinsights)
* [Logary](https://www.nuget.org/packages/Logary.Targets.AppInsights/)
* [Logrus](https://github.com/jjcollinge/logrus-appinsights)
* [Azure Monitor](https://blogs.technet.microsoft.com/msoms/2016/09/26/application-insights-connector-in-oms/)

## <a name="content-management-systems"></a>Sistemas de administración de contenido
* [Concrete](https://github.com/fidmor89/appInsights-Concrete)
* [Drupal](https://github.com/fidmor89/AppInsights-Drupal)
* [Joomla](https://github.com/fidmor89/AppInsights-Joomla)
* [Orchard](https://azure.microsoft.com/blog/integrating-application-insights-into-a-modular-cms-and-a-multi-tenant-public-saas/preview/)
* [SharePoint](../../azure-monitor/app/sharepoint.md)
* [WordPress](https://wordpress.org/plugins/application-insights/)

## <a name="export-and-data-analysis"></a>Exportación y análisis de datos
* [Alooma](https://www.alooma.com/blog/application-insights-amazon-redshift)
* [Power BI](https://blogs.msdn.com/b/powerbi/archive/2015/11/04/explore-your-application-insights-data-with-power-bi.aspx)
* [Stream Analytics](../../azure-monitor/app/export-power-bi.md )

## <a name="projects"></a> Creación de su propio SDK
Si no hay todavía un SDK para el lenguaje o plataforma, quizás desee crear uno. Examine el código de los SDK existentes que aparecen en el [proyecto de SDK de Application Insights en GitHub](https://github.com/Microsoft/AppInsights-Home).
