---
title: Contadores de rendimiento en Application Insights | Microsoft Docs
description: Supervise los contadores de rendimiento de .NET, tanto del sistema como personalizados, en Application Insights.
ms.topic: conceptual
ms.date: 12/13/2018
ms.custom: devx-track-csharp
ms.openlocfilehash: 4da8aef69b6a83c17fa8a20a80b2c485378e0aef
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88918526"
---
# <a name="system-performance-counters-in-application-insights"></a>Contadores de rendimiento de sistema en Application Insights

Windows proporciona una amplia variedad de [contadores de rendimiento](/windows/desktop/perfctrs/about-performance-counters), como la ocupación de la CPU, memoria, disco y uso de la red. También puede definir sus propios contadores de rendimiento. La colección de contadores de rendimiento es compatible siempre que se aplicación se ejecute en IIS en un host local o en una máquina virtual para la que tenga acceso administrativo. Aunque las aplicaciones que se ejecutan como Azure Web Apps no tienen acceso directo a los contadores de rendimiento, Application Insights recopila un subconjunto de contadores disponibles.

## <a name="view-counters"></a>Visualización de contadores

El panel Métricas muestra el conjunto predeterminado de contadores de rendimiento.

![Contadores de rendimiento notificados en Application Insights](./media/performance-counters/performance-counters.png)

Los contadores predeterminados actuales que se configuran para la recopilación para aplicaciones web de ASP.NET/ASP.NET Core son los siguientes:
- % de Proceso\\Tiempo de procesador
- % de Proceso\\Tiempo de procesador normalizado
- Memoria\\Bytes disponibles
- Solicitudes de ASP.NET/segundo
- Excepciones de .NET CLR iniciadas/segundo
- Tiempo de ejecución de solicitudes de aplicaciones ASP.NET
- Proceso\\Bytes privados
- Proceso\\Bytes de datos de E/S por segundo
- Aplicaciones ASP.NET\\Solicitudes en la cola de aplicaciones
- Procesador(_Total)\\% de tiempo del procesador

## <a name="add-counters"></a>Adición de contadores

Si el contador de rendimiento que quiere no está incluido en la lista de métricas, puede agregarlo.

1. Averigüe qué contadores están disponibles en el servidor mediante este comando de PowerShell en el servidor local:

    `Get-Counter -ListSet *`

    (Consulte [`Get-Counter`](/powershell/module/microsoft.powershell.diagnostics/get-counter?view=powershell-5.1)).
2. Abra ApplicationInsights.config.

   * Si agrega Application Insights a la aplicación durante el desarrollo, edite ApplicationInsights.config en el proyecto y vuelva a implementarlo en los servidores.
3. Edite la directiva del recopilador de rendimiento:

    ```XML

        <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule, Microsoft.AI.PerfCounterCollector">
          <Counters>
            <Add PerformanceCounter="\Objects\Processes"/>
            <Add PerformanceCounter="\Sales(photo)\# Items Sold" ReportAs="Photo sales"/>
          </Counters>
        </Add>
    ```

> [!NOTE]
> Las aplicaciones de ASP.NET Core no tienen `ApplicationInsights.config` y, por lo tanto, el método anterior no es válido para las aplicaciones de ASP.NET Core.

Puede capturar tanto los contadores estándar como los que ha implementado usted mismo. `\Objects\Processes` es un ejemplo de contador estándar que está disponible en todos los sistemas Windows. `\Sales(photo)\# Items Sold` es un ejemplo de contador personalizado que podría implementarse en un servicio web.

El formato es `\Category(instance)\Counter"` o, para las categorías que no tienen instancias, simplemente `\Category\Counter`.

`ReportAs` es necesario para los nombres de contadores que no coinciden `[a-zA-Z()/-_ \.]+`: es decir, que contienen caracteres que no están en los siguientes conjuntos: letras, paréntesis, barra diagonal, guión, subrayado, espacio, punto.

Si especifica una instancia, se recopilará como una dimensión "CounterInstanceName" de la métrica notificada.

### <a name="collecting-performance-counters-in-code-for-aspnet-web-applications-or-netnet-core-console-applications"></a>Recopilación de contadores de rendimiento en el código para aplicaciones web de ASP.NET o aplicaciones de consola de .NET/.NET Core
Para recopilar contadores de rendimiento del sistema y enviarlos a Application Insights, puede adaptar el siguiente fragmento:


```csharp
    var perfCollectorModule = new PerformanceCollectorModule();
    perfCollectorModule.Counters.Add(new PerformanceCounterCollectionRequest(
      @"\Process([replace-with-application-process-name])\Page Faults/sec", "PageFaultsPerfSec"));
    perfCollectorModule.Initialize(TelemetryConfiguration.Active);
```

También puede hacer lo mismo con las métricas personalizadas que haya creado:

```csharp
    var perfCollectorModule = new PerformanceCollectorModule();
    perfCollectorModule.Counters.Add(new PerformanceCounterCollectionRequest(
      @"\Sales(photo)\# Items Sold", "Photo sales"));
    perfCollectorModule.Initialize(TelemetryConfiguration.Active);
```

### <a name="collecting-performance-counters-in-code-for-aspnet-core-web-applications"></a>Recopilación de contadores de rendimiento en el código para aplicaciones web de ASP.NET Core

Modifique el método `ConfigureServices` en su clase `Startup.cs` como aparece a continuación.

```csharp
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector;

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetry();

        // The following configures PerformanceCollectorModule.
  services.ConfigureTelemetryModule<PerformanceCollectorModule>((module, o) =>
            {
                // the application process name could be "dotnet" for ASP.NET Core self-hosted applications.
                module.Counters.Add(new PerformanceCounterCollectionRequest(
    @"\Process([replace-with-application-process-name])\Page Faults/sec", "DotnetPageFaultsPerfSec"));
            });
    }
```

## <a name="performance-counters-in-analytics"></a>Contadores de rendimiento en Analytics
Puede buscar y mostrar informes de contador de rendimiento en [Analytics](../log-query/log-query-overview.md).

El esquema **performanceCounters** expone `category`, el nombre de `counter` y el nombre de `instance` de cada contador de rendimiento.  En la telemetría de cada aplicación, solo se ven los contadores de dicha aplicación. Por ejemplo, para ver qué contadores están disponibles: 

![Contadores de rendimiento en Application Insights Analytics](./media/performance-counters/analytics-performance-counters.png)

(Aquí "Instance" hace referencia a la instancia de contador de rendimiento, no al rol ni a la instancia de máquina de servidor. El nombre de la instancia del contador de rendimiento normalmente segmenta contadores, como el tiempo de procesador por el nombre del proceso o la aplicación).

Para obtener un gráfico de la memoria disponible en un período reciente: 

![Gráfico de tiempo de la memoria in Application Insights Analytics](./media/performance-counters/analytics-available-memory.png)

Al igual que otros datos de telemetría, **performanceCounters** también tiene una columna `cloud_RoleInstance` que indica la identidad de la instancia del servidor host en el que se ejecuta la aplicación. Por ejemplo, para comparar el rendimiento de una aplicación en distintas máquinas: 

![Rendimiento segmentado por instancia de rol en Application Insights Analytics](./media/performance-counters/analytics-metrics-role-instance.png)

## <a name="aspnet-and-application-insights-counts"></a>Recuentos de ASP.NET y Application Insights

*¿En qué se diferencian la tasa de excepciones y las métricas de excepciones?*

* *tasa de excepciones* es un contador de rendimiento del sistema. El CLR cuenta todas las excepciones controladas y no controladas que se producen, y divide el total de un intervalo de muestreo entre la duración del intervalo. El SDK de Application Insights recopila este resultado y lo envía al portal.

* *Excepciones* es un recuento de los informes de TrackException recibidos a través del portal en el intervalo de muestreo del gráfico. Solo incluye las excepciones controladas para las que ha escrito llamadas a TrackException en el código y no incluye todas las [excepciones no controladas](./asp-net-exceptions.md). 

## <a name="performance-counters-for-applications-running-in-azure-web-apps"></a>Contadores de rendimiento para aplicaciones que se ejecutan en Azure Web Apps

Las aplicaciones de ASP.NET y ASP.NET Core implementadas en Azure Web Apps se ejecutan en un entorno de espacio aislado especial. Este entorno no permite el acceso directo a los contadores de rendimiento del sistema. Sin embargo, se expone un subconjunto limitado de contadores como variables de entorno como se describe [aquí](https://github.com/projectkudu/kudu/wiki/Perf-Counters-exposed-as-environment-variables). El SDK de Application Insights para ASP.NET y ASP.NET Core recopila contadores de rendimiento desde Azure Web Apps desde estas variables de entorno especiales. Solo un subconjunto de contadores está disponibles en este entorno, y puede encontrar la lista completa [aquí](https://github.com/microsoft/ApplicationInsights-dotnet-server/blob/develop/WEB/Src/PerformanceCollector/Perf.Shared/Implementation/WebAppPerformanceCollector/CounterFactory.cs).

## <a name="performance-counters-in-aspnet-core-applications"></a>Contadores de rendimiento en aplicaciones de ASP.NET Core

La compatibilidad con los contadores de rendimiento en ASP.Net Core es limitada:

* Las versiones 2.4.1 y posteriores del [SDK](https://nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) recopilan contadores de rendimiento si la aplicación se ejecuta en Azure Web Apps (Windows).
* Las versiones 2.7.1 y posteriores del SDK recopilan contadores de rendimiento si la aplicación se ejecuta en Windows y tiene como destino `NETSTANDARD2.0` o una versión posterior.
* Para las aplicaciones que tienen como destino .NET Framework, todas las versiones del SDK admiten contadores de rendimiento.
* Las versiones 2.8.0 y posteriores del SDK admiten el contador de CPU/memoria de Linux. No se admite ningún otro contador en Linux. La manera recomendada de obtener contadores del sistema en Linux (y otros entornos que no son Windows) es mediante [EventCounters](eventcounters.md).

## <a name="alerts"></a>Alertas
Al igual que otras métricas, puede [establecer una alerta](../platform/alerts-log.md) para advertirle si un contador de rendimiento queda fuera de un límite especificado. Abra el panel de alertas y haga clic en Agregar alerta.

## <a name="next-steps"></a><a name="next"></a>Pasos siguientes

* [Seguimiento de dependencias](./asp-net-dependencies.md)
* [Seguimiento de excepciones](./asp-net-exceptions.md)

