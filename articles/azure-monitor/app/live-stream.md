---
title: 'Diagnóstico con Live Metrics Stream: Azure Application Insights'
description: Supervise la aplicación web en tiempo real con métricas personalizadas y diagnostique problemas con una fuente directa de errores, seguimientos y eventos.
ms.topic: conceptual
ms.date: 04/22/2019
ms.reviewer: sdash
ms.openlocfilehash: c12126c23ce1f1e2bd72f88eead5b8f34e4fd83d
ms.sourcegitcommit: a2a7746c858eec0f7e93b50a1758a6278504977e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88142220"
---
# <a name="live-metrics-stream-monitor--diagnose-with-1-second-latency"></a>Live Metrics Stream: supervisión y diagnóstico con una latencia de 1 segundo

Supervise la aplicación web, ya activa y en producción, con Live Metrics Stream (también conocido como QuickPulse) desde [Application Insights](./app-insights-overview.md). Seleccione métricas y contadores de rendimiento y fíltrelos para inspeccionarlos en tiempo real sin que el servicio se vea afectado. Inspeccione seguimientos de la pila procedentes de ejemplos de errores de solicitudes y excepciones. Junto con [Profiler](./profiler.md) y el [depurador de instantáneas](./snapshot-debugger.md), Live Metrics Stream proporciona una herramienta de diagnóstico eficaz y no invasiva para sitios web activos.

Con Live Metrics Stream, puede:

* Inspeccionar el rendimiento y los recuentos de errores mientras se publica una solución para validarla.
* Inspeccionar el efecto de las cargas de prueba y diagnosticar problemas en tiempo real.
* Centrarse en ciertas sesiones de prueba o filtrar problemas conocidos seleccionando las métricas que quiera inspeccionar y filtrándolas.
* Obtener seguimientos de las excepciones cuando se produzcan.
* Experimentar con filtros para localizar los indicadores clave de rendimiento más pertinentes.
* Supervisar cualquier contador de rendimiento de Windows en tiempo real.
* Identifique fácilmente un servidor con problemas y filtre todos los KPI o fuente directa solo en dicho servidor.

![Pestaña Live Metrics](./media/live-stream/live-metric.png)

Actualmente, Live Metrics se admite en aplicaciones de ASP.NET, ASP.NET Core, Azure Functions, Java y Node.js.

## <a name="get-started"></a>Introducción

1. Siga las instrucciones específicas del lenguaje para habilitar Live Metrics.
   * [ASP.NET](./asp-net.md): Live Metrics está habilitada de forma predeterminada.
   * [ASP.NET Core](./asp-net-core.md): Live Metrics está habilitada de forma predeterminada.
   * [.NET, Consola de .NET Core o Worker](./worker-service.md): Live Metrics está habilitada de forma predeterminada.
   * [Aplicaciones .NET: se habilita mediante código](#enable-livemetrics-using-code-for-any-net-application).
   * [Node.js](./nodejs.md#live-metrics)

2. En [Azure Portal](https://portal.azure.com), abra el recurso de Application Insights de la aplicación y, a continuación, abra Live Stream.

3. [Proteja el canal de control](#secure-the-control-channel) si puede que vaya a usar información confidencial como nombres de clientes en los filtros.

### <a name="enable-livemetrics-using-code-for-any-net-application"></a>Habilitación de LiveMetrics mediante código para cualquier aplicación .NET

Aunque LiveMetrics se habilita de forma predeterminada cuando se incorpora siguiendo las instrucciones recomendadas para las aplicaciones .NET, a continuación se muestra cómo configurar Live Metrics de forma manual.

1. Instale el paquete NuGet [Microsoft.ApplicationInsights.PerfCounterCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector).
2. El siguiente código de aplicación de consola de ejemplo muestra la configuración de Live Metrics.

```csharp
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
using System;
using System.Threading.Tasks;

namespace LiveMetricsDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            // Create a TelemetryConfiguration instance.
            TelemetryConfiguration config = TelemetryConfiguration.CreateDefault();
            config.InstrumentationKey = "INSTRUMENTATION-KEY-HERE";
            QuickPulseTelemetryProcessor quickPulseProcessor = null;
            config.DefaultTelemetrySink.TelemetryProcessorChainBuilder
                .Use((next) =>
                {
                    quickPulseProcessor = new QuickPulseTelemetryProcessor(next);
                    return quickPulseProcessor;
                })
                .Build();

            var quickPulseModule = new QuickPulseTelemetryModule();

            // Secure the control channel.
            // This is optional, but recommended.
            quickPulseModule.AuthenticationApiKey = "YOUR-API-KEY-HERE";
            quickPulseModule.Initialize(config);
            quickPulseModule.RegisterTelemetryProcessor(quickPulseProcessor);

            // Create a TelemetryClient instance. It is important
            // to use the same TelemetryConfiguration here as the one
            // used to setup Live Metrics.
            TelemetryClient client = new TelemetryClient(config);

            // This sample runs indefinitely. Replace with actual application logic.
            while (true)
            {
                // Send dependency and request telemetry.
                // These will be shown in Live Metrics stream.
                // CPU/Memory Performance counter is also shown
                // automatically without any additional steps.
                client.TrackDependency("My dependency", "target", "http://sample",
                    DateTimeOffset.Now, TimeSpan.FromMilliseconds(300), true);
                client.TrackRequest("My Request", DateTimeOffset.Now,
                    TimeSpan.FromMilliseconds(230), "200", true);
                Task.Delay(1000).Wait();
            }
        }
    }
}
```

Aunque el ejemplo anterior es para una aplicación de consola, se puede usar el mismo código en cualquier aplicación .NET. Si se habilita cualquier otro módulo TelemetryModules que recopile automáticamente la telemetría, es importante asegurarse de que la misma configuración que se use para la inicialización de los módulos también se emplee para el módulo de Live Metrics.

## <a name="how-does-live-metrics-stream-differ-from-metrics-explorer-and-analytics"></a>¿En qué se diferencia Live Metrics Stream de Explorador de métricas y Analytics?

| |Live Stream | Explorador de métricas y Analytics |
|---|---|---|
|**Latency**|Los datos se muestran en un segundo.|La agregación se realiza en minutos.|
|**Sin retención**|Los datos se conservan solo mientras se encuentren en el gráfico y luego se descartan.|[Los datos se conservan durante 90 días.](./data-retention-privacy.md#how-long-is-the-data-kept)|
|**A petición**|Los datos solo se transmiten mientras el panel de Live Metrics está abierto. |Se envían datos siempre que el SDK esté instalado y habilitado.|
|**Gratis**|No se efectúa ningún cargo por los datos de Live Stream.|Están sujetos a aplicación de [precios](./pricing.md).
|**Muestreo**|Se transmiten todas las métricas y los contadores seleccionados. Se muestrean los errores y seguimientos de la pila. |Se pueden [muestrear](./api-filtering-sampling.md) eventos.|
|**Canal de control**|Las señales de control de filtro se envían al SDK. Se recomienda proteger este canal.|La comunicación es unidireccional, hacia el portal|

## <a name="select-and-filter-your-metrics"></a>Selección y filtrado de métricas

[Disponible con ASP.NET, ASP.NET Core y Azure Functions (versión 2).]

Puede supervisar KPI personalizados en vivo aplicando filtros arbitrarios en cualquier telemetría de Application Insights desde el portal. Haga clic en el control de filtro que se muestra cuando coloca el mouse sobre cualquiera de los gráficos. En el siguiente gráfico se traza un KPI de recuento de solicitudes personalizado con filtros en atributos de URL y duración. Valide los filtros con la sección de versión preliminar de secuencias que muestra una fuente activa de telemetría que coincide con los criterios especificados en cualquier momento.

![Tasa de solicitudes del filtro](./media/live-stream/filter-request.png)

Puede supervisar un valor que no sea el de recuento. Las opciones dependen del tipo de secuencia, que podría ser cualquier telemetría de Application Insights: solicitudes, dependencias, excepciones, seguimientos, eventos o métricas. Puede ser su propia [medida personalizada](./api-custom-events-metrics.md#properties):

![Generador de consultas sobre tasa de solicitudes con métricas personalizadas](./media/live-stream/query-builder-request.png)

Además de telemetría de Application Insights, también puede supervisar cualquier contador de rendimiento de Windows eligiendo entre las opciones de secuencias y proporcionando el nombre del contador de rendimiento.

Las métricas activas se agregan en dos puntos: localmente en cada servidor y, a continuación, en todos los servidores. Puede cambiar el valor predeterminado en cualquiera de ellos seleccionando otras opciones en las correspondientes listas desplegables.

## <a name="sample-telemetry-custom-live-diagnostic-events"></a>Telemetría de ejemplo: eventos personalizados de diagnóstico en vivo
De forma predeterminada, la fuente directa de eventos muestra ejemplos de solicitudes con error y llamadas de dependencia, excepciones, eventos y seguimientos. Haga clic en el icono de filtro para ver los criterios aplicados en cualquier momento.

![Botón de filtro](./media/live-stream/filter.png)

Como con las métricas, puede especificar cualquier criterio arbitrario en cualquiera de los tipos de telemetría de Application Insights. En este ejemplo, seleccionaremos los eventos y errores de solicitud específicos.

![Generador de consultas](./media/live-stream/query-builder.png)

> [!NOTE]
> Actualmente, para los criterios basados en mensajes de excepción, use el mensaje de excepción más externo. En el ejemplo anterior, para filtrar la excepción benigna con el mensaje de excepción interna (lo que sigue al delimitador "<--"): "El cliente está desconectado". Use un mensaje que no contenga criterios de error al leer el contenido de la solicitud.

Consulte los detalles de un elemento en la fuente directa haciendo clic en él. Puede pausar la fuente haciendo clic en **Pausar** o simplemente desplazándose hacia abajo o haciendo clic en un elemento. La fuente directa se reanudará una vez que se desplace hacia la parte superior, o bien al hacer clic en el contador de elementos recopilados mientras estaba en pausa.

![Errores de muestreo de Live](./media/live-stream/sample-telemetry.png)

## <a name="filter-by-server-instance"></a>Filtrado por instancia de servidor

Si quiere supervisar una instancia de rol de servidor en particular, puede filtrar por servidor. Para filtrar, seleccione el nombre del servidor en *Servidores*.

![Errores de muestreo de Live](./media/live-stream/filter-by-server.png)

## <a name="secure-the-control-channel"></a>Protección del canal de control

> [!NOTE]
> Actualmente, solo se puede configurar un canal autenticado mediante la supervisión basada en código y no se pueden autenticar los servidores con la conexión sin código.

Los criterios de filtros personalizados que especifique en el portal de Live Metrics se envían hacia el componente de Live Metrics del SDK de Application Insights. Los filtros podrían contener información confidencial, como identificadores de clientes. Puede hacer que el canal seguro con una clave secreta de API, además de la clave de instrumentación.

### <a name="create-an-api-key"></a>Creación de una clave de API

![Clave de API > Crear una clave de API](./media/live-stream/api-key.png)
![Pestaña Crear una clave de API Selección de "Autenticar canal de control del SDK" y luego "Generar clave"](./media/live-stream/create-api-key.png)

### <a name="add-api-key-to-configuration"></a>Incorporación de la clave de API a la configuración

### <a name="aspnet"></a>ASP.NET

En el archivo applicationinsights.config, agregue AuthenticationApiKey a QuickPulseTelemetryModule:

```XML
<Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse.QuickPulseTelemetryModule, Microsoft.AI.PerfCounterCollector">
      <AuthenticationApiKey>YOUR-API-KEY-HERE</AuthenticationApiKey>
</Add>
```

### <a name="aspnet-core"></a>ASP.NET Core

Para las aplicaciones de [ASP.NET Core](./asp-net-core.md), siga las instrucciones descritas aquí.

Modifique `ConfigureServices` del archivo Startup.cs como sigue:

Agregue el siguiente espacio de nombres.

```csharp
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
```

A continuación, modifique el método `ConfigureServices` como se indica a continuación.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // existing code which include services.AddApplicationInsightsTelemetry() to enable Application Insights.
    services.ConfigureTelemetryModule<QuickPulseTelemetryModule> ((module, o) => module.AuthenticationApiKey = "YOUR-API-KEY-HERE");
}
```

Puede encontrar más información sobre la configuración de las aplicaciones de ASP.NET Core en nuestras instrucciones para [configurar los módulos de telemetría en ASP.NET Core](./asp-net-core.md#configuring-or-removing-default-telemetrymodules).

### <a name="workerservice"></a>WorkerService

Para las aplicaciones de [WorkerService](./worker-service.md), siga las instrucciones que se indican a continuación.

Agregue el siguiente espacio de nombres.

```csharp
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
```

A continuación, agregue la siguiente línea antes de la llamada a `services.AddApplicationInsightsTelemetryWorkerService`.

```csharp
    services.ConfigureTelemetryModule<QuickPulseTelemetryModule> ((module, o) => module.AuthenticationApiKey = "YOUR-API-KEY-HERE");
```

Puede encontrar más información sobre la configuración de aplicaciones de WorkerService en nuestras instrucciones sobre la [configuración de los módulos de telemetría en WorkerServices](./worker-service.md#configuring-or-removing-default-telemetrymodules).

### <a name="azure-function-apps"></a>Azure Function Apps

En el caso de Azure Function Apps (versión 2), la protección del canal con una clave de API puede realizarse con una variable de entorno.

Cree una clave de API desde su recurso de Application Insights y vaya a **Opciones > Configuración** en la aplicación de función. Seleccione **Nueva configuración de la aplicación** y escriba un nombre de `APPINSIGHTS_QUICKPULSEAUTHAPIKEY` y un valor que corresponda a la clave de API.

Sin embargo, si reconoce y confía en todos los servidores conectados, puede probar los filtros personalizados sin el canal autenticado. Esta opción está disponible durante seis meses. Esta invalidación se requiere una vez cada nueva sesión, o cuando se pone en línea un nuevo servidor.

![Opciones de autenticación de Live Metrics](./media/live-stream/live-stream-auth.png)

>[!NOTE]
>Se recomienda que configure el canal autenticado antes de escribir información potencialmente confidencial, como identificadores de clientes en los criterios de filtro.
>

## <a name="supported-features-table"></a>Tabla de características compatibles

| Idioma                         | Métricas básicas       | Métricas de rendimiento | Filtrado personalizado    | Telemetría de ejemplo    | División de CPU por proceso |
|----------------------------------|:--------------------|:--------------------|:--------------------|:--------------------|:---------------------|
| .NET Framework                   | Se admite (V2.7.2+) | Se admite (V2.7.2+) | Se admite (V2.7.2+) | Se admite (V2.7.2+) | Se admite (V2.7.2+)  |
| .NET Core (destino=.NET Framework)| Se admite (V2.4.1+) | Se admite (V2.4.1+) | Se admite (V2.4.1+) | Se admite (V2.4.1+) | Se admite (V2.4.1+)  |
| .NET Core (destino=.NET Core)     | Se admite (V2.4.1+) | Admitido*          | Se admite (V2.4.1+) | Se admite (V2.4.1+) | **No admitido**    |
| Azure Functions v2               | Compatible           | Compatible           | Compatible           | Compatible           | **No admitido**    |
| Java                             | Se admite (V2.0.0+) | Se admite (V2.0.0+) | **No admitido**   | **No admitido**   | **No admitido**    |
| Node.js                          | Se admite (V1.3.0+) | Se admite (V1.3.0+) | **No admitido**   | Se admite (V1.3.0+) | **No admitido**    |

Las métricas básicas incluyen la tasa de solicitudes, dependencias y excepciones. Las métricas de rendimiento (contadores de rendimiento) incluyen la memoria y la CPU. La telemetría de ejemplo muestra un flujo de información detallada para las solicitudes y dependencias con errores, excepciones, eventos y seguimientos.

 \* La compatibilidad con PerfCounters varía ligeramente en las diversas versiones de .NET Core que no tienen .NET Framework como destino:

- Se admiten las métricas de PerfCounters cuando se ejecutan en Azure App Service para Windows. (AspNetCore SDK Version 2.4.1 o superior)
- Se admiten PerfCounters cuando la aplicación se ejecuta en CUALQUIER máquina de Windows, ya sea una máquina virtual, un servicio en la nube, local, etc. (AspNetCore SDK versión 2.7.1 o superior), pero para las aplicaciones que tienen como destino .NET Core 2.0 o superior.
- Los PerfCounters se admiten cuando la aplicación se ejecuta en cualquier lugar (Linux, Windows, App Service para Linux, contenedores, etc.) en las versiones más recientes (es decir, AspNetCore SDK versión 2.8.0 o posterior), pero solo para las aplicaciones que tengan como destino .NET Core 2.0 o posterior.

## <a name="troubleshooting"></a>Solución de problemas

Live Metrics Stream usa direcciones IP diferentes a las de otros datos de telemetría de Application Insights. Asegúrese de que [esas direcciones IP](./ip-addresses.md) están abiertos en el firewall. Compruebe también que los [puertos de salida de Live Metrics Stream](./ip-addresses.md#outgoing-ports) estén abiertos en el firewall de los servidores.

## <a name="next-steps"></a>Pasos siguientes

* [Supervisión del uso con Application Insights](./usage-overview.md)
* [Uso de la Búsqueda de diagnóstico](./diagnostic-search.md)
* [Generador de perfiles](./profiler.md)
* [Depurador de instantáneas](./snapshot-debugger.md)
