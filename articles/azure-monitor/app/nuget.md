---
title: Paquetes NuGet de Application Insights de Azure Monitor
description: Listas de paquetes NuGet de Application Insights de Azure Monitor para ASP.NET, ASP.NET Core, Python
ms.topic: reference
ms.date: 10/16/2018
ms.openlocfilehash: 27a3d89b4a64de159535d346641c21616833e21b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87309960"
---
# <a name="application-insights-nuget-packages"></a>Paquetes NuGet de Application Insights

A continuación se encuentra la lista actual de los paquetes NuGet de versión estable de Application Insights.

## <a name="common-packages-for-aspnet"></a>Paquetes comunes para ASP.NET

| Nombre del paquete | Versión estable | Descripción | Descargar |
|-------------------------------|-----------------------|------------|----|
| Microsoft.ApplicationInsights | 2.12.0 | Proporciona funcionalidad básica para la transmisión de todos los tipos de datos de telemetría de Application Insights y es un paquete dependiente de todos los demás paquetes de Application Insights | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights/) |
|Microsoft.ApplicationInsights.Agent.Intercept | 2.4.0 | Habilita la interceptación de llamadas a método | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Agent.Intercept/) |
| Microsoft.ApplicationInsights.DependencyCollector | 2.12.0 | Recopilador de dependencias de Application Insights para aplicaciones .NET. Es un paquete dependiente para los paquetes específicos de la plataforma de Application Insights y proporciona la recopilación automática de telemetría de dependencias. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector/) |
| Microsoft.ApplicationInsights.PerfCounterCollector | 2.12.0 | El recopilador de contadores de rendimiento de Application Insights permite enviar datos recopilados por los contadores de rendimiento a Application Insights. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector/) |
| Microsoft.ApplicationInsights.Web | 2.12.0 | Application Insights para aplicaciones web .NET | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web/) |
| Microsoft.ApplicationInsights.WindowsServer | 2.12.0 | El paquete NuGet de Windows Server para Application Insights ofrece una recopilación automática de telemetría de Application Insights para aplicaciones .NET. Este paquete se puede usar como un paquete dependiente para los paquetes específicos de la plataforma de Application Insights o como un paquete independiente para las aplicaciones .NET que no están cubiertas por los paquetes específicos de la plataforma (como en el caso de los roles de trabajo. NET). | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer/)  
| Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel | 2.12.0 | Proporciona un canal de telemetría para el SDK de Windows Server para Application Insights que conservará la telemetría en escenarios sin conexión. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel/) |

## <a name="common-packages-for-aspnet-core"></a>Paquetes comunes para ASP.NET Core

| Nombre del paquete | Versión estable | Descripción | Descargar |
|-------------------------------|-----------------------|------------|----|
| Microsoft.ApplicationInsights.AspNetCore | 2.5.0 | Application Insights para aplicaciones web ASP.NET Core | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore/) |
| Microsoft.ApplicationInsights | 2.12.0 | Este paquete proporciona funcionalidad básica para la transmisión de todos los tipos de datos de telemetría de Application Insights y es un paquete dependiente de todos los demás paquetes de Application Insights | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights/) |
| Microsoft.ApplicationInsights.DependencyCollector | 2.12.0 | Recopilador de dependencias de Application Insights para aplicaciones .NET. Es un paquete dependiente para los paquetes específicos de la plataforma de Application Insights y proporciona la recopilación automática de telemetría de dependencias. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector/) |
| Microsoft.ApplicationInsights.PerfCounterCollector | 2.12.0 | El recopilador de contadores de rendimiento de Application Insights permite enviar datos recopilados por los contadores de rendimiento a Application Insights. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector/) |
| Microsoft.ApplicationInsights.WindowsServer | 2.12.0 | El paquete NuGet de Windows Server para Application Insights ofrece una recopilación automática de telemetría de Application Insights para aplicaciones .NET. Este paquete se puede usar como un paquete dependiente para los paquetes específicos de la plataforma de Application Insights o como un paquete independiente para las aplicaciones .NET que no están cubiertas por los paquetes específicos de la plataforma (como en el caso de los roles de trabajo. NET). | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer/)  |
| Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel | 2.12.0 | Proporciona un canal de telemetría para el SDK de Windows Server para Application Insights que conservará la telemetría en escenarios sin conexión. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel/) |

## <a name="common-packages-for-python-using-opencensus"></a>Paquetes comunes para Python mediante OpenCensus
| Nombre del paquete | Versión estable | Descripción | Descargar |
|-------------------------------|-----------------------|------------|----|
| opencensus-ext-azure | 1.0.0 | Application Insights para las aplicaciones de Python en Azure Monitor a través de OpenCensus. | [Descargar paquete](https://pypi.org/project/opencensus-ext-azure/) |
| opencensus-ext-django | 0.7.2 | Este paquete proporciona la integración con la biblioteca [django](https://pypi.org/project/django/) de Python. | [Descargar paquete](https://pypi.org/project/opencensus-ext-django/) |
| opencensus-ext-flask | 0.7.3 | Este paquete proporciona la integración con la biblioteca [flask](https://pypi.org/project/flask/) de Python. | [Descargar paquete](https://pypi.org/project/opencensus-ext-flask/) |
| opencensus-ext-httplib | 0.7.2 | Este paquete proporciona la integración con la biblioteca [http.client](https://docs.python.org/3/library/http.client.html) de Python para Python3 y [httplib](https://docs.python.org/2/library/httplib.html) para Python2. | [Descargar paquete](https://pypi.org/project/opencensus-ext-httplib/) |
| opencensus-ext-logging | 0.1.0 | Este paquete enriquece las entradas de registro con los datos de seguimiento. | [Descargar paquete](https://pypi.org/project/opencensus-ext-logging/) |
| opencensus-ext-mysql | 0.1.2 | Este paquete proporciona la integración con la biblioteca [mysql-connector](https://pypi.org/project/mysql-connector/) de Python. | [Descargar paquete](https://pypi.org/project/opencensus-ext-mysql/) |
| opencensus-ext-postgresql | 0.1.2 | Este paquete proporciona la integración con la biblioteca [psycopg2](https://pypi.org/project/psycopg2/) de Python. | [Descargar paquete](https://pypi.org/project/opencensus-ext-postgresql/) |
| opencensus-ext-pymongo | 0.7.1 | Este paquete proporciona la integración con la biblioteca [pymongo](https://pypi.org/project/pymongo/) de Python. | [Descargar paquete](https://pypi.org/project/opencensus-ext-pymongo/) |
| opencensus-ext-pymysql | 0.1.2 | Este paquete proporciona la integración con la biblioteca [PyMySQL](https://pypi.org/project/PyMySQL/) de Python. | [Descargar paquete](https://pypi.org/project/opencensus-ext-pymysql/) |
| opencensus-ext-pyramid | 0.7.1 | Este paquete proporciona la integración con la biblioteca [pyramid](https://pypi.org/project/pyramid/) de Python. | [Descargar paquete](https://pypi.org/project/opencensus-ext-pyramid/) |
| opencensus-ext-requests | 0.7.2 | Este paquete proporciona la integración con la biblioteca [requests](https://pypi.org/project/requests/) de Python. | [Descargar paquete](https://pypi.org/project/opencensus-ext-requests/) |
| opencensus-ext-sqlalchemy | 0.1.2 | Este paquete proporciona la integración con la biblioteca [SQLAlchemy](https://pypi.org/project/SQLAlchemy/) de Python. | [Descargar paquete](https://pypi.org/project/opencensus-ext-sqlalchemy/) |

## <a name="listenerscollectorsappenders"></a>Agentes de escucha/recopiladores/appenders

| Nombre del paquete | Versión estable | Descripción | Descargar |
|-------------------------------|-----------------------|------------|----|
| Microsoft.ApplicationInsights.DiagnosticSourceListener | 2.7.2 |  Permite el reenvío de eventos de DiagnosticSource a Application Insights. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener/) |
| Microsoft.ApplicationInsights.EventSourceListener | 2.7.2 | EventSourceListener de Application Insights permite enviar datos de eventos de EventSource a Application Insights. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventSourceListener/) |
| Microsoft.ApplicationInsights.EtwCollector | 2.7.2 | EtwCollector de Application Insights permite enviar datos de Seguimiento de eventos para Windows (ETW) a Application Insights. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EtwCollector/) |
| Microsoft.ApplicationInsights.TraceListener | 2.7.2 | Un TraceListener personalizado que le permite enviar mensajes de registro de seguimiento a Application Insights. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.TraceListener/) |
| Microsoft.ApplicationInsights.Log4NetAppender | 2.7.2 | Un appender personalizado que le permite enviar mensajes de registro de Log4Net a Application Insights. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Log4NetAppender/)
| Microsoft.ApplicationInsights.NLogTarget | 2.7.2 |  Un destino personalizado que le permite enviar mensajes de registro de NLog a Application Insights. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.NLogTarget/)
| Microsoft.ApplicationInsights.SnapshotCollector | 1.3.1 | Supervisa las excepciones en la aplicación y recopila automáticamente las instantáneas para realizar análisis sin conexión. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector/)

## <a name="service-fabric"></a>Service Fabric

| Nombre del paquete | Versión estable | Descripción | Descargar |
|-------------------------------|-----------------------|------------|----|
| Microsoft.ApplicationInsights.ServiceFabric | 2.2.0 | Este paquete proporciona decoración automática de telemetría con el contexto de Service Fabric en el que se ejecuta la aplicación. No use este NuGet para aplicaciones nativas de Service Fabric. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.ServiceFabric/) |
| Microsoft.ApplicationInsights.ServiceFabric.Native | 2.2.0 | Módulo de Application Insights para aplicaciones de Service Fabric. Use este NuGet para aplicaciones nativas de Service Fabric. Para aplicaciones que se ejecutan en contenedores, use el paquete Microsoft.ApplicationInsights.ServiceFabric. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.ServiceFabric.Native/) |  

## <a name="status-monitor"></a>Monitor de estado

| Nombre del paquete | Versión estable | Descripción | Descargar |
|-------------------------------|-----------------------|------------|----|
| Microsoft.ApplicationInsights.Agent_x64 | 2.2.1 |  Habilita la recopilación de datos en tiempo de ejecución para aplicaciones x64 | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Agent_x64/) |
| Microsoft.ApplicationInsights.Agent_x86 | 2.2.1 |  Habilita la recopilación de datos en tiempo de ejecución para aplicaciones x86. | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Agent_x86/) |

Estos paquetes forman parte de la funcionalidad básica de la supervisión en tiempo de ejecución en [Monitor de estado](./monitor-performance-live-website-now.md). No necesita descargar estos paquetes directamente; use solo el instalador del Monitor de estado. Si desea obtener más información sobre cómo funcionan estos paquetes en segundo plato, esta [entrada de blog](https://apmtips.com/posts/2016-11-18-how-application-insights-status-monitor-not-monitors-dependencies/) de uno de nuestros desarrolladores es un buen punto de partida.

## <a name="additional-packages"></a>Paquetes adicionales

| Nombre del paquete | Versión estable | Descripción | Descargar |
|-------------------------------|-----------------------|------------|----|
| Microsoft.ApplicationInsights.AzureWebSites | 2.6.5 | Esta extensión permite la supervisión de Application Insights en Azure App Service. Versión del SDK 2.6.1. Instrucciones: Agregue la configuración de la aplicación "APPINSIGHTS_INSTRUMENTATIONKEY" con su ikey y reinicie la aplicación web para que surta efecto.| [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AzureWebSites/) |
| Microsoft.ApplicationInsights.Injector | 2.6.7 | Este paquete contiene los archivos necesarios para la inserción de Application Insights sin código | [Descargar paquete](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Injector/) |

## <a name="next-steps"></a>Pasos siguientes

- Supervisar [ASP.NET Core](./asp-net-core.md).
- Generar perfiles de [aplicaciones web con Linux en Azure](./profiler-aspnetcore-linux.md) para ASP.NET Core.
- Depurar [instantáneas](./snapshot-debugger.md) de ASP.NET.

