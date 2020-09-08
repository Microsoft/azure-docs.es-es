---
title: Seguimiento de dependencia en Azure Application Insights | Microsoft Docs
description: Supervise las llamadas de dependencia de su aplicación web o local de Microsoft Azure con Application Insights.
ms.topic: conceptual
ms.date: 08/26/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 3d98fe91994c992d11fc58e3fec42d1796c0c966
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88936544"
---
# <a name="dependency-tracking-in-azure-application-insights"></a>Seguimiento de dependencias en Azure Application Insights 

Una *dependencia* es un componente al que la aplicación llama. Suele ser un servicio al que se llama mediante HTTP, una base de datos o un sistema de archivos. [Application Insights](./app-insights-overview.md) mide la duración de las llamadas de dependencia, independientemente de si devuelven un error o no, junto con información adicional como el nombre de la dependencia y más. Puede investigar llamadas de dependencia específicas y relacionarlas a solicitudes y excepciones.

## <a name="automatically-tracked-dependencies"></a>Dependencias con seguimiento automático

Los SDK de Application Insights para .NET y .NET Core se suministran con `DependencyTrackingTelemetryModule`, un módulo de telemetría que recopila dependencias automáticamente. Esta recolección de dependencias se habilita automáticamente para las aplicaciones de [ASP.NET](./asp-net.md) y [ASP.NET Core](./asp-net-core.md) cuando se configuran según la documentación oficial vinculada. `DependencyTrackingTelemetryModule` se distribuye como [este](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector/) paquete NuGet y se incluye automáticamente cuando usa cualquiera de los paquetes NuGet `Microsoft.ApplicationInsights.Web` o `Microsoft.ApplicationInsights.AspNetCore`.

 `DependencyTrackingTelemetryModule` actualmente realiza un seguimiento automático de las siguientes dependencias:

|Dependencias |Detalles|
|---------------|-------|
|HTTP/HTTPS | Llamadas HTTP/HTTPS locales o remotas |
|Llamadas WCF| Solo realiza un seguimiento automáticamente si se usan enlaces basados en HTTP.|
|SQL | Llamadas realizadas con `SqlClient`. Consulte [esta sección](#advanced-sql-tracking-to-get-full-sql-query) para capturar consultas SQL.  |
|[Azure Storage (blobs, tablas, colas)](https://www.nuget.org/packages/WindowsAzure.Storage/) | Llamadas realizadas con el cliente de Azure Storage. |
|[SDK de cliente de EventHub](https://www.nuget.org/packages/Microsoft.Azure.EventHubs) | Versión 1.1.0 y posteriores. |
|[SDK de cliente de Service Bus](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus)| Versión 3.0.0 y posteriores. |
|Azure Cosmos DB | Se realiza un seguimiento automático solo si se usa HTTP/HTTPS. Application Insights no capturará el modo TCP. |

Si falta una dependencia o está usando un SDK diferente, asegúrese de que se encuentra en la lista de [dependencias recopiladas automáticamente](./auto-collect-dependencies.md). Si la dependencia no se recopila automáticamente, todavía puede realizar un seguimiento manual con una [llamada a TrackDependency ](./api-custom-events-metrics.md#trackdependency).

## <a name="setup-automatic-dependency-tracking-in-console-apps"></a>Configuración de seguimiento automático de dependencias en aplicaciones de consola

Para hacer un seguimiento automático de las dependencias de aplicaciones de consola de .NET, instale el paquete de NuGet `Microsoft.ApplicationInsights.DependencyCollector` e inicialice `DependencyTrackingTelemetryModule` como se indica a continuación:

```csharp
    DependencyTrackingTelemetryModule depModule = new DependencyTrackingTelemetryModule();
    depModule.Initialize(TelemetryConfiguration.Active);
```

En el caso de las aplicaciones de consola de .NET Core TelemetryConfiguration.Active está obsoleto. Consulte las instrucciones de la [documentación del servicio de trabajo](./worker-service.md) y la [documentación de supervisión de ASP.NET Core](./asp-net-core.md)

### <a name="how-automatic-dependency-monitoring-works"></a>Funcionamiento de la supervisión automática de dependencias

Las dependencias se recopilan automáticamente mediante una de las siguientes técnicas:

* Se usa instrumentación del código de byte alrededor de determinados métodos. (InstrumentationEngine de StatusMonitor o de la extensión de aplicación web de Azure)
* Devoluciones de llamada de EventSource
* Devoluciones de llamada de DiagnosticSource (en los últimos SDK de .NET/.NET Core)

## <a name="manually-tracking-dependencies"></a>Seguimiento manual las dependencias

Los siguientes son algunos ejemplos de las dependencias que no se recopilan automáticamente y, por lo tanto, requieren seguimiento manual.

* Azure Cosmos DB se rastrea automáticamente solo si se usa [HTTP/HTTPS](../../cosmos-db/performance-tips.md#networking). Application Insights no capturará el modo TCP.
* Redis

Para las dependencias que el SDK no recopila automáticamente, puede realizar un seguimiento manual mediante la [API de TrackDependency](api-custom-events-metrics.md#trackdependency) que utilizan los módulos de recolección automática estándar.

Por ejemplo, si compila el código con un ensamblado que no escribió usted mismo, podría cronometrar todas las llamadas al ensamblado para averiguar cómo contribuye a los tiempos de respuesta. Para que estos datos se muestren en los gráficos de dependencia en Application Insights, envíelos mediante `TrackDependency`.

```csharp

    var startTime = DateTime.UtcNow;
    var timer = System.Diagnostics.Stopwatch.StartNew();
    try
    {
        // making dependency call
        success = dependency.Call();
    }
    finally
    {
        timer.Stop();
        telemetryClient.TrackDependency("myDependencyType", "myDependencyCall", "myDependencyData",  startTime, timer.Elapsed, success);
    }
```

Como alternativa, `TelemetryClient` proporciona los métodos de extensión `StartOperation` y `StopOperation` que pueden usarse para realizar un seguimiento manual de las dependencias, tal como se muestra [aquí](custom-operations-tracking.md#outgoing-dependencies-tracking).

Si desea desactivar el módulo de seguimiento de dependencia estándar, quite la referencia a DependencyTrackingTelemetryModule en [ApplicationInsights.config](../../azure-monitor/app/configuration-with-applicationinsights-config.md) para las aplicaciones de ASP.NET. Para las aplicaciones de ASP.NET Core, siga las instrucciones [descritas aquí](asp-net-core.md#configuring-or-removing-default-telemetrymodules).

## <a name="tracking-ajax-calls-from-web-pages"></a>Seguimiento de llamadas AJAX desde páginas web

Para las páginas web, el SDK de JavaScript de Application Insights recopila automáticamente las llamadas AJAX como dependencias.

## <a name="advanced-sql-tracking-to-get-full-sql-query"></a>Seguimiento de SQL avanzado para obtener la consulta SQL completa

Para las llamadas SQL, el nombre del servidor y la base de datos siempre se recopilan y almacenan como el nombre de la `DependencyTelemetry` recopilada. Hay un campo adicional denominado "data", que puede contener el texto completo de la consulta SQL.

En el caso de aplicaciones ASP.NET Core, ahora es necesario participar en la recopilación de texto SQL mediante:
```csharp
services.ConfigureTelemetryModule<DependencyTrackingTelemetryModule>((module, o) => { module. EnableSqlCommandTextInstrumentation = true; });
```

En el caso de las aplicaciones de ASP.NET, se recopila texto de la consulta SQL completa con la ayuda de la instrumentación de código de bytes, que requiere el uso del motor de instrumentación o puede realizarse mediante el paquete NuGet [Microsoft.Data.SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient), en lugar de la biblioteca System.Data.SqlClient. A continuación, se describen los pasos específicos de la plataforma para habilitar la recopilación completa de consultas SQL:

| Plataforma | Pasos necesarios para obtener la consulta SQL completa |
| --- | --- |
| Aplicación web de Azure |En el panel de control de la aplicación web [abra la hoja de Application Insights](../../azure-monitor/app/azure-web-apps.md) y habilite los comandos SQL de .NET |
| Servidor IIS (máquina virtual de Azure, del entorno local, entre otras). | Use el paquete de NuGet [Microsoft.Data.SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient) o use el módulo Monitor de estado de PowerShell para [instalar el motor de instrumentación](../../azure-monitor/app/status-monitor-v2-api-reference.md) y reiniciar IIS. |
| Servicio en la nube de Azure | Agregue una [tarea de inicio para instalar StatusMonitor](../../azure-monitor/app/cloudservices.md#set-up-status-monitor-to-collect-full-sql-queries-optional). <br> La aplicación se debe incorporar al SDK de ApplicationInsights en tiempo de compilación mediante la instalación de paquetes de NuGet para las aplicaciones de [ASP.NET](./asp-net.md) o [ASP.NET Core](./asp-net-core.md). |
| IIS Express | Use el paquete NuGet [Microsoft.Data.SqlClient](https://www.nuget.org/packages/Microsoft.Data.SqlClient).

Además de los pasos específicos de la plataforma anteriores, también **debe optar explícitamente por habilitar la colección de comandos SQL** al modificar el archivo applicationInsights.config con lo siguiente:

```xml
<Add Type="Microsoft.ApplicationInsights.DependencyCollector.DependencyTrackingTelemetryModule, Microsoft.AI.DependencyCollector">
<EnableSqlCommandTextInstrumentation>true</EnableSqlCommandTextInstrumentation>
</Add>
```

En los casos anteriores, la forma correcta de validar que el motor de instrumentación está instalado correctamente es comprobar que la versión de SDK de la `DependencyTelemetry` recopilada sea "rddp". Un valor "rdddsd" o "rddf" indica que las dependencias se recopilan a través de las devoluciones de llamada de DiagnosticSource o EventSource y, por lo tanto, no se puede capturar la consulta SQL completa.

## <a name="where-to-find-dependency-data"></a>Dónde encontrar los datos de dependencia

* [Asignación de aplicación](app-map.md) visualiza las dependencias entre la aplicación y los componentes colindantes.
* [Diagnósticos de transacción](transaction-diagnostics.md) muestra los datos de servidor unificados correlacionados.
* La [pestaña Exploradores](javascript.md) muestra las llamadas AJAX de los exploradores de los usuarios.
* Haga clic en las solicitudes lentas o con errores para comprobar sus llamadas de dependencia.
* [Análisis](#logs-analytics) puede utilizarse para consultar los datos de dependencia.

## <a name="diagnose-slow-requests"></a><a name="diagnosis"></a> Diagnóstico de solicitudes lentas

Cada evento de solicitud está asociado a las llamadas de dependencia, las excepciones y otros eventos de los que se realiza un seguimiento mientras la aplicación procesa la solicitud. Por lo tanto, si algunas solicitudes no tienen un buen rendimiento, puede averiguar si se debe a respuestas lentas de una dependencia.

### <a name="tracing-from-requests-to-dependencies"></a>Seguimiento de solicitudes en dependencias

Abra la pestaña **Rendimiento** y vaya a la pestaña **Dependencias** en la parte superior junto a las operaciones.

Haga clic en un **nombre de dependencia** de la sección general. Después de seleccionar una dependencia, a la derecha se mostrará un gráfico con la distribución de duraciones de esa dependencia.

![En la pestaña Rendimiento, haga clic en la pestaña Dependencia en la parte superior y, a continuación, en un nombre de dependencia en el gráfico](./media/asp-net-dependencies/2-perf-dependencies.png)

Haga clic en el botón azul **Ejemplos** en la parte inferior derecha y, a continuación, en un ejemplo para ver los detalles de transacción completos.

![Haga clic en un ejemplo para ver los detalles de transacción completos](./media/asp-net-dependencies/3-end-to-end.png)

### <a name="profile-your-live-site"></a>Generación de un perfil del sitio activo

¿Quiere saber en qué se invierte el tiempo? El [generador de perfiles de Application Insights](../../azure-monitor/app/profiler.md) realiza un seguimiento de las llamadas HTTP a los sitios activos y muestra qué funciones del código tardan más tiempo en ejecutarse.

## <a name="failed-requests"></a>Error en las solicitudes

Las solicitudes con error también podrían estar asociadas a llamadas a dependencias con errores.

Podemos ir a la pestaña **Errores** a la izquierda y, a continuación, hacer clic en la pestaña **Dependencias** en la parte superior.

![Haga clic en el gráfico de las solicitudes con error.](./media/asp-net-dependencies/4-fail.png)

Aquí podrá ver el recuento de dependencias con errores. Para obtener más detalles sobre una repetición con error, pruebe a hacer clic en el nombre de una dependencia en la tabla inferior. Puede hacer clic en el botón azul **Dependencias** situado en la parte inferior derecha para obtener los detalles de transacción completos.

## <a name="logs-analytics"></a>Registros (Analytics)

Puede realizar un seguimiento de las dependencias en el [lenguaje de consulta de Kusto](/azure/kusto/query/). Estos son algunos ejemplos.

* Búsqueda de llamadas de dependencia con errores:

``` Kusto

    dependencies | where success != "True" | take 10
```

* Búsqueda de llamadas AJAX:

``` Kusto

    dependencies | where client_Type == "Browser" | take 10
```

* Búsqueda de llamadas de dependencia asociadas a solicitudes:

``` Kusto

    dependencies
    | where timestamp > ago(1d) and  client_Type != "Browser"
    | join (requests | where timestamp > ago(1d))
      on operation_Id  
```


* Búsqueda de llamadas AJAX asociadas a vistas de página:

``` Kusto 

    dependencies
    | where timestamp > ago(1d) and  client_Type == "Browser"
    | join (browserTimings | where timestamp > ago(1d))
      on operation_Id
```

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="how-does-automatic-dependency-collector-report-failed-calls-to-dependencies"></a>*¿Qué método usa el recopilador de dependencias para informar sobre las llamadas con error a las dependencias?*

* Las llamadas de dependencia con error tendrán el campo "success" establecido en False. `DependencyTrackingTelemetryModule` no notifica sobre `ExceptionTelemetry`. El modelo de datos completo para las dependencias se describe [aquí](data-model-dependency-telemetry.md).

### <a name="how-do-i-calculate-ingestion-latency-for-my-dependency-telemetry"></a>*¿Cómo calculo la latencia de ingesta para mi telemetría de dependencia?*

```kusto
dependencies
| extend E2EIngestionLatency = ingestion_time() - timestamp 
| extend TimeIngested = ingestion_time()
```

### <a name="how-do-i-determine-the-time-the-dependency-call-was-initiated"></a>*Cómo determino la hora en la que se inició la llamada de dependencia?*

En la vista de consulta de Log Analytics `timestamp` representa el momento en que se inició la llamada a TrackDependency(), que se produjo inmediatamente después de recibir la respuesta de la llamada de dependencia. Para calcular la hora de inicio de la llamada de dependencia, debería tomar `timestamp` y restarle el valor de `duration` registrado para la llamada de dependencia.

## <a name="open-source-sdk"></a>SDK de código abierto
Como todos los SDK de Application Insights, el módulo de recolección de dependencias también es de código abierto. Puede leer y contribuir al código o notificar sobre problemas en [el repositorio oficial de GitHub](https://github.com/Microsoft/ApplicationInsights-dotnet-server).

## <a name="next-steps"></a>Pasos siguientes

* [Excepciones](./asp-net-exceptions.md)
* [Datos de página y usuario](./javascript.md)
* [Disponibilidad](./monitor-web-app-availability.md)

