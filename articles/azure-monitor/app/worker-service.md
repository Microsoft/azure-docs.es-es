---
title: Application Insights para las aplicaciones de Worker Service (aplicaciones no HTTP) | Microsoft Docs
description: Supervisión de aplicaciones de .NET Core/.NET Framework que no son HTTP con Azure Monitor Application Insights.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 05/11/2020
ms.openlocfilehash: 2ca5fc2d8f5e9e399fd7dfd3238d0ec16056d537
ms.sourcegitcommit: 3c66bfd9c36cd204c299ed43b67de0ec08a7b968
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/10/2020
ms.locfileid: "90007220"
---
# <a name="application-insights-for-worker-service-applications-non-http-applications"></a>Application Insights para las aplicaciones de servicio de trabajo (aplicaciones sin HTTP)

Application Insights va a lanzar un nuevo SDK, denominado `Microsoft.ApplicationInsights.WorkerService`, más adecuado para cargas de trabajo sin HTTP, como mensajería, tareas en segundo plano, aplicaciones de consola, etc. Estos tipos de aplicaciones no tienen la noción de una solicitud HTTP entrante como las aplicaciones web ASP.NET/ASP.NET Core tradicionales y, por tanto, no se admiten los paquetes de Application Insights para las aplicaciones [ASP.NET](asp-net.md) o [ASP.NET Core](asp-net-core.md).

El nuevo SDK no recopila telemetría por sí mismo. En su lugar, ofrece en otros recopiladores automáticos de Application Insights conocidos, como [DependencyCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector/), [PerfCounterCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector/) o [ApplicationInsightsLoggingProvider](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights). Este SDK expone métodos de extensión en `IServiceCollection` para habilitar y configurar la recopilación de telemetría.

## <a name="supported-scenarios"></a>Escenarios admitidos

El [SDK de Application Insights para el servicio de trabajo](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) es más adecuado para aplicaciones sin HTTP, independientemente de dónde y cómo se ejecuten. Si la aplicación se está ejecutando y tiene conectividad de red a Azure, se pueden recopilar datos de telemetría. La supervisión de Application Insights se admite siempre y cuando .NET Core sea compatible. Este paquete se puede usar en el [servicio de trabajo de .NET Core 3.0](https://devblogs.microsoft.com/aspnet/dotnet-core-workers-in-azure-container-instances) presentado recientemente, en [tareas en segundo plano en Asp.Net Core 2.1/2.2](/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-2.2&preserve-view=true), en aplicaciones de consola (.NET Core/.NET Framework), etc.

## <a name="prerequisites"></a>Prerrequisitos

Una clave de instrumentación de Application Insights válida. Esta clave es necesaria para enviar los datos de telemetría a Application Insights. Si necesita crear un nuevo recurso de Application Insights para obtener un clave de instrumentación, consulte [Creación de recursos en Application Insights](./create-new-resource.md).

## <a name="using-application-insights-sdk-for-worker-services"></a>Uso del SDK de Application Insights para servicios de trabajo

1. Instale el paquete [Microsoft.ApplicationInsights.WorkerService](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) en la aplicación.
   En el siguiente fragmento de código se muestran los cambios que se agregarán al archivo `.csproj` del proyecto.

```xml
    <ItemGroup>
        <PackageReference Include="Microsoft.ApplicationInsights.WorkerService" Version="2.13.1" />
    </ItemGroup>
```

1. Llame al método de extensión `AddApplicationInsightsTelemetryWorkerService(string instrumentationKey)` en `IServiceCollection` y proporcione la clave de instrumentación. Se debe llamar a este método al inicio de la aplicación. La ubicación exacta depende del tipo de aplicación.

1. Recupere una instancia de `ILogger` o de `TelemetryClient` del contenedor de inserción de dependencias (DI) mediante una llamada a `serviceProvider.GetRequiredService<TelemetryClient>();` o la inserción de constructores. Este paso desencadenará la configuración de los módulos de `TelemetryConfiguration` y de recopilación automática.

Las instrucciones concretas para cada tipo de aplicación se indican en las secciones siguientes.

## <a name="net-core-30-worker-service-application"></a>Aplicación de servicio de trabajo .NET Core 3.0

El ejemplo completo se comparte [aquí](https://github.com/microsoft/ApplicationInsights-Home/tree/master/Samples/WorkerServiceSDK/WorkerServiceSampleWithApplicationInsights)

1. Descargue e instale [.NET Core 3.0](https://dotnet.microsoft.com/download/dotnet-core/3.0).
2. Cree un proyecto de servicio de trabajo mediante la nueva plantilla de proyecto de Visual Studio o la línea de comandos `dotnet new worker`.
3. Instale el paquete [Microsoft.ApplicationInsights.WorkerService](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) en la aplicación.

4. Agregue `services.AddApplicationInsightsTelemetryWorkerService();` al método `CreateHostBuilder()` en su clase `Program.cs`, como en este ejemplo:

```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
                services.AddHostedService<Worker>();
                services.AddApplicationInsightsTelemetryWorkerService();
            });
```

5. Modifique `Worker.cs` como se muestra en el ejemplo siguiente.

```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.DataContracts;

    public class Worker : BackgroundService
    {
        private readonly ILogger<Worker> _logger;
        private TelemetryClient _telemetryClient;
        private static HttpClient _httpClient = new HttpClient();

        public Worker(ILogger<Worker> logger, TelemetryClient tc)
        {
            _logger = logger;
            _telemetryClient = tc;
        }

        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);

                using (_telemetryClient.StartOperation<RequestTelemetry>("operation"))
                {
                    _logger.LogWarning("A sample warning message. By default, logs with severity Warning or higher is captured by Application Insights");
                    _logger.LogInformation("Calling bing.com");
                    var res = await _httpClient.GetAsync("https://bing.com");
                    _logger.LogInformation("Calling bing completed with status:" + res.StatusCode);
                    _telemetryClient.TrackEvent("Bing call event completed");
                }

                await Task.Delay(1000, stoppingToken);
            }
        }
    }
```

6. Comprobar la clave de instrumentación.

    Aunque puede proporcionar la clave de instrumentación como un argumento a `AddApplicationInsightsTelemetryWorkerService`, se recomienda que especifique la clave de instrumentación en la configuración. En el siguiente ejemplo de código se muestra cómo especificar una clave de instrumentación en `appsettings.json`. Asegúrese de que `appsettings.json` se copia en la carpeta raíz de la aplicación durante la publicación.

```json
    {
        "ApplicationInsights":
            {
            "InstrumentationKey": "putinstrumentationkeyhere"
            },
        "Logging":
        {
            "LogLevel":
            {
                "Default": "Warning"
            }
        }
    }
```

Como alternativa, puede especificar la clave de instrumentación en cualquiera de las siguientes variables de entorno.
`APPINSIGHTS_INSTRUMENTATIONKEY` o `ApplicationInsights:InstrumentationKey`

Por ejemplo: `SET ApplicationInsights:InstrumentationKey=putinstrumentationkeyhere`
o `SET APPINSIGHTS_INSTRUMENTATIONKEY=putinstrumentationkeyhere`

Por lo general, `APPINSIGHTS_INSTRUMENTATIONKEY` especifica la clave de instrumentación para las aplicaciones implementadas en Web Apps como trabajos web.

> [!NOTE]
> Una clave de instrumentación especificada en el código prevalece frente a la variable del entorno `APPINSIGHTS_INSTRUMENTATIONKEY`, que prevalece sobre otras opciones.

## <a name="aspnet-core-background-tasks-with-hosted-services"></a>Tareas en segundo plano de ASP.NET Core con servicios hospedados

En [este](/aspnet/core/fundamentals/host/hosted-services?tabs=visual-studio&view=aspnetcore-2.2&preserve-view=true) documento se describe cómo crear tareas en segundo plano en la aplicación ASP.NET Core 2.1/2.2.

El ejemplo completo se comparte [aquí](https://github.com/microsoft/ApplicationInsights-Home/tree/master/Samples/WorkerServiceSDK/BackgroundTasksWithHostedService)

1. Instale el paquete Microsoft.ApplicationInsights.WorkerService (https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) ) en la aplicación.
2. Agregue `services.AddApplicationInsightsTelemetryWorkerService();` al método `ConfigureServices()`, como en este ejemplo:

```csharp
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .ConfigureAppConfiguration((hostContext, config) =>
            {
                config.AddJsonFile("appsettings.json", optional: true);
            })
            .ConfigureServices((hostContext, services) =>
            {
                services.AddLogging();
                services.AddHostedService<TimedHostedService>();

                // instrumentation key is read automatically from appsettings.json
                services.AddApplicationInsightsTelemetryWorkerService();
            })
            .UseConsoleLifetime()
            .Build();

        using (host)
        {
            // Start the host
            await host.StartAsync();

            // Wait for the host to shutdown
            await host.WaitForShutdownAsync();
        }
    }
```

A continuación se encuentra el código para `TimedHostedService` donde reside la lógica de la tarea en segundo plano.

```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.DataContracts;

    public class TimedHostedService : IHostedService, IDisposable
    {
        private readonly ILogger _logger;
        private Timer _timer;
        private TelemetryClient _telemetryClient;
        private static HttpClient httpClient = new HttpClient();

        public TimedHostedService(ILogger<TimedHostedService> logger, TelemetryClient tc)
        {
            _logger = logger;
            this._telemetryClient = tc;
        }

        public Task StartAsync(CancellationToken cancellationToken)
        {
            _logger.LogInformation("Timed Background Service is starting.");

            _timer = new Timer(DoWork, null, TimeSpan.Zero,
                TimeSpan.FromSeconds(1));

            return Task.CompletedTask;
        }

        private void DoWork(object state)
        {
            _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);

            using (_telemetryClient.StartOperation<RequestTelemetry>("operation"))
            {
                _logger.LogWarning("A sample warning message. By default, logs with severity Warning or higher is captured by Application Insights");
                _logger.LogInformation("Calling bing.com");
                var res = httpClient.GetAsync("https://bing.com").GetAwaiter().GetResult();
                _logger.LogInformation("Calling bing completed with status:" + res.StatusCode);
                _telemetryClient.TrackEvent("Bing call event completed");
            }
        }
    }
```

3. Comprobar la clave de instrumentación.
   Use el mismo `appsettings.json` del ejemplo anterior del servicio de trabajo de .NET Core 3.0.

## <a name="net-corenet-framework-console-application"></a>Aplicación de consola .NET Core/.NET Framework

Como se mencionó al principio de este artículo, el nuevo paquete se puede usar para habilitar la telemetría de Application Insights, incluso desde una aplicación de consola normal. Este paquete tiene como destino [`NetStandard2.0`](/dotnet/standard/net-standard) y, por lo tanto, se puede usar para aplicaciones de consola con .NET Core 2.0 o una versión posterior, y .NET Framework 4.7.2 o una versión posterior.

El ejemplo completo se comparte [aquí](https://github.com/microsoft/ApplicationInsights-Home/tree/master/Samples/WorkerServiceSDK/ConsoleAppWithApplicationInsights)

1. Instale el paquete Microsoft.ApplicationInsights.WorkerService (https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) ) en la aplicación.

2. Modifique Program.cs tal como se muestra en el ejemplo siguiente.

```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.DataContracts;
    using Microsoft.Extensions.DependencyInjection;
    using Microsoft.Extensions.Logging;
    using System;
    using System.Net.Http;
    using System.Threading.Tasks;

    namespace WorkerSDKOnConsole
    {
        class Program
        {
            static async Task Main(string[] args)
            {
                // Create the DI container.
                IServiceCollection services = new ServiceCollection();

                // Being a regular console app, there is no appsettings.json or configuration providers enabled by default.
                // Hence instrumentation key and any changes to default logging level must be specified here.
                services.AddLogging(loggingBuilder => loggingBuilder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>("Category", LogLevel.Information));
                services.AddApplicationInsightsTelemetryWorkerService("instrumentationkeyhere");

                // Build ServiceProvider.
                IServiceProvider serviceProvider = services.BuildServiceProvider();

                // Obtain logger instance from DI.
                ILogger<Program> logger = serviceProvider.GetRequiredService<ILogger<Program>>();

                // Obtain TelemetryClient instance from DI, for additional manual tracking or to flush.
                var telemetryClient = serviceProvider.GetRequiredService<TelemetryClient>();

                var httpClient = new HttpClient();

                while (true) // This app runs indefinitely. replace with actual application termination logic.
                {
                    logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);

                    // Replace with a name which makes sense for this operation.
                    using (telemetryClient.StartOperation<RequestTelemetry>("operation"))
                    {
                        logger.LogWarning("A sample warning message. By default, logs with severity Warning or higher is captured by Application Insights");
                        logger.LogInformation("Calling bing.com");                    
                        var res = await httpClient.GetAsync("https://bing.com");
                        logger.LogInformation("Calling bing completed with status:" + res.StatusCode);
                        telemetryClient.TrackEvent("Bing call event completed");
                    }

                    await Task.Delay(1000);
                }

                // Explicitly call Flush() followed by sleep is required in Console Apps.
                // This is to ensure that even if application terminates, telemetry is sent to the back-end.
                telemetryClient.Flush();
                Task.Delay(5000).Wait();
            }
        }
    }
```

Esta aplicación de consola también usa el mismo `TelemetryConfiguration` predeterminado y se puede personalizar de la misma manera que los ejemplos de la sección anterior.

## <a name="run-your-application"></a>Ejecución de la aplicación

Ejecute la aplicación. Todos los trabajos de ejemplo anteriores realizan una llamada HTTP por segundo a bing.com y emiten pocos registros mediante ILogger. Estas líneas se incluyen dentro de la llamada `StartOperation` de `TelemetryClient`, que se usa para crear una operación (en este ejemplo, `RequestTelemetry`, denominada "operation"). Application Insights recopilará estos registros de ILogger (de nivel advertencia o superior, de forma predeterminada) y las dependencias, y se correlacionarán con `RequestTelemetry` con relación de elementos primarios y secundarios. La correlación también funciona a través de límites proceso/red. Por ejemplo, si la llamada se realizó a otro componente supervisado, se correlacionará también con este elemento primario.

Esta operación personalizada de `RequestTelemetry` puede considerarse como el equivalente de una solicitud web de entrada en una aplicación web típica. Aunque no es necesario usar una operación, se adapta mejor al [modelo de datos de correlación de Application Insights](./correlation.md), donde `RequestTelemetry` actúa como operación primaria y la telemetría generada en la iteración del trabajo se trata como que lógicamente pertenece a la misma operación. Este enfoque también garantiza que toda la telemetría generada (automática y manual) tendrá el mismo `operation_id`. Dado que el muestreo se basa en `operation_id`, el algoritmo de muestreo mantiene o quita toda la telemetría de una sola iteración.

A continuación se enumera toda la telemetría que Application Insights recopila automáticamente.

### <a name="live-metrics"></a>Live Metrics

[Live Metrics](./live-stream.md) se puede usar para comprobar rápidamente si la supervisión de Application Insights está configurada correctamente. Aunque la telemetría puede tardar unos minutos en aparecer en el portal y en el análisis, Live Metrics mostraría el uso de la CPU del proceso en ejecución casi en tiempo real. También puede mostrar otros tipos de telemetría, como las solicitudes, las dependencias, los seguimientos, etc.

### <a name="ilogger-logs"></a>Registros de ILogger

Los registros emitidos a través de `ILogger` de gravedad `Warning` o mayor se capturan automáticamente. Siga [la documentación de ILogger](ilogger.md#control-logging-level) para personalizar los niveles de registro que captura Application Insights.

### <a name="dependencies"></a>Dependencias

La recopilación de dependencias está habilitada de manera predeterminada. En [este](asp-net-dependencies.md#automatically-tracked-dependencies) artículo se explican las dependencias que se recopilan automáticamente; también contiene pasos para realizar el seguimiento manual.

### <a name="eventcounter"></a>EventCounter

`EventCounterCollectionModule` está habilitado de forma predeterminada y recopilará un conjunto predeterminado de contadores de las aplicaciones de .NET Core 3.0. En el tutorial de [EventCounter](eventcounters.md) se muestra el conjunto de contadores recopilado de manera predeterminada. También incluye instrucciones sobre la personalización de la lista.

### <a name="manually-tracking-additional-telemetry"></a>Seguimiento manual de la telemetría adicional

Aunque el SDK recopila automáticamente la telemetría como se explicó anteriormente, en la mayoría de los casos el usuario deberá enviar telemetría adicional al servicio Application Insights. El mecanismo recomendado para hacer un seguimiento de esta telemetría adicional consiste en obtener una instancia de `TelemetryClient` a partir de la inserción de dependencias y llamar después a uno de los métodos `TrackXXX()` de la [API](api-custom-events-metrics.md) admitidos. Otro caso de uso típico es el [seguimiento personalizado de las operaciones](custom-operations-tracking.md). Este enfoque se muestra en los ejemplos de trabajo anteriores.

## <a name="configure-the-application-insights-sdk"></a>Configuración del SDK de Application Insights

El `TelemetryConfiguration` predeterminado que usa el SDK del servicio de trabajo es similar a la configuración automática que se usa en una aplicación ASP.NET o ASP.NET Core, menos los enriquecedores de telemetría que sirven para enriquecer la telemetría de `HttpContext`.

Puede personalizar el SDK de Application Insights para el servicio de trabajo para cambiar la configuración predeterminada. Es posible que los usuarios de los SDK de Application Insights para ASP.NET estén familiarizados con el cambio de configuración mediante el uso de la [inserción de dependencias](/aspnet/core/fundamentals/dependency-injection) incorporada de ASP.NET Core. El SDK de del servicio de trabajo también se basa en principios similares. Realice casi todos los cambios de configuración en la sección `ConfigureServices()` mediante una llamada a los métodos adecuados en `IServiceCollection`, tal como se detalla a continuación.

> [!NOTE]
> Al usar este SDK, no se admite el cambio de la configuración mediante la modificación de `TelemetryConfiguration.Active` y los cambios no se reflejan.

### <a name="using-applicationinsightsserviceoptions"></a>Uso de ApplicationInsightsServiceOptions

Puede modificar algunos ajustes de configuración comunes pasando de `ApplicationInsightsServiceOptions` a `AddApplicationInsightsTelemetryWorkerService`, como se muestra en este ejemplo:

```csharp
    using Microsoft.ApplicationInsights.WorkerService;

    public void ConfigureServices(IServiceCollection services)
    {
        Microsoft.ApplicationInsights.WorkerService.ApplicationInsightsServiceOptions aiOptions
                    = new Microsoft.ApplicationInsights.WorkerService.ApplicationInsightsServiceOptions();
        // Disables adaptive sampling.
        aiOptions.EnableAdaptiveSampling = false;

        // Disables QuickPulse (Live Metrics stream).
        aiOptions.EnableQuickPulseMetricStream = false;
        services.AddApplicationInsightsTelemetryWorkerService(aiOptions);
    }
```

Tenga en cuenta que `ApplicationInsightsServiceOptions` en este SDK está en el espacio de nombres `Microsoft.ApplicationInsights.WorkerService` en lugar de en `Microsoft.ApplicationInsights.AspNetCore.Extensions` en el SDK de ASP.NET Core.

Configuraciones que se suelen usar en `ApplicationInsightsServiceOptions`

|Configuración | Descripción | Valor predeterminado
|---------------|-------|-------
|EnableQuickPulseMetricStream | Habilitar o deshabilitar la característica LiveMetrics | true
|EnableAdaptiveSampling | Habilitar o deshabilitar el muestreo adaptable | true
|EnableHeartbeat | Habilitar o deshabilitar la característica de latidos, que periódicamente (cada 15 minutos de forma predeterminada) envía una métrica personalizada denominada "HeartBeatState" con información sobre el entorno de ejecución, por ejemplo, la versión de .NET, información del entorno de Azure, si procede, etc. | true
|AddAutoCollectedMetricExtractor | Habilitar o deshabilitar el extractor de AutoCollectedMetrics, que es un elemento TelemetryProcessor que envía métricas previamente agregadas sobre las solicitudes o dependencias antes de que tenga lugar el muestreo. | true
|EnableDiagnosticsTelemetryModule | Habilitar o deshabilitar `DiagnosticsTelemetryModule`. Si se deshabilita, se omitirá la siguiente configuración; `EnableHeartbeat`, `EnableAzureInstanceMetadataTelemetryModule`, `EnableAppServicesHeartbeatTelemetryModule` | true

Consulte los [valores configurables en `ApplicationInsightsServiceOptions`](https://github.com/microsoft/ApplicationInsights-dotnet/blob/develop/NETCORE/src/Shared/Extensions/ApplicationInsightsServiceOptions.cs) para obtener la lista más actualizada.

### <a name="sampling"></a>muestreo

El SDK de Application Insights para el servicio de trabajo admite el muestreo adaptable y el de tipo fijo. El muestreo adaptable está habilitado de forma predeterminada. La configuración del muestreo para el servicio de trabajo se realiza de la misma manera que para las [aplicaciones ASP.NET Core](./sampling.md#configuring-adaptive-sampling-for-aspnet-core-applications).

### <a name="adding-telemetryinitializers"></a>Adición de TelemetryInitializers

Use [inicializadores de telemetría](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) cuando quiera definir propiedades para que se envíen con toda la telemetría.

Agregue cualquier `TelemetryInitializer` nuevo al contenedor `DependencyInjection` y el SDK lo agregará automáticamente a `TelemetryConfiguration`.

```csharp
    using Microsoft.ApplicationInsights.Extensibility;

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSingleton<ITelemetryInitializer, MyCustomTelemetryInitializer>();
        services.AddApplicationInsightsTelemetryWorkerService();
    }
```

### <a name="removing-telemetryinitializers"></a>Eliminación de Telemetryinitializer

Los inicializadores de telemetría están presentes de forma predeterminada. Para quitar todos los inicializadores de telemetría o solo algunos específicos, use el siguiente código de ejemplo *después* de llamar a `AddApplicationInsightsTelemetryWorkerService()`.

```csharp
   public void ConfigureServices(IServiceCollection services)
   {
        services.AddApplicationInsightsTelemetryWorkerService();
        // Remove a specific built-in telemetry initializer
        var tiToRemove = services.FirstOrDefault<ServiceDescriptor>
                            (t => t.ImplementationType == typeof(AspNetCoreEnvironmentTelemetryInitializer));
        if (tiToRemove != null)
        {
            services.Remove(tiToRemove);
        }

        // Remove all initializers
        // This requires importing namespace by using Microsoft.Extensions.DependencyInjection.Extensions;
        services.RemoveAll(typeof(ITelemetryInitializer));
   }
```

### <a name="adding-telemetry-processors"></a>Adición de procesadores de telemetría

Puede agregar procesadores de telemetría personalizados a `TelemetryConfiguration` mediante el método de extensión `AddApplicationInsightsTelemetryProcessor` en `IServiceCollection`. Puede usar los procesadores de telemetría en [escenarios de filtrado avanzado](./api-filtering-sampling.md#itelemetryprocessor-and-itelemetryinitializer) para permitir un control más directo sobre lo que se incluirá o excluirá de la telemetría que envía al servicio Application Insights. Use el ejemplo siguiente.

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetryWorkerService();
        services.AddApplicationInsightsTelemetryProcessor<MyFirstCustomTelemetryProcessor>();
        // If you have more processors:
        services.AddApplicationInsightsTelemetryProcessor<MySecondCustomTelemetryProcessor>();
    }
```

### <a name="configuring-or-removing-default-telemetrymodules"></a>Configuración o eliminación de objetos TelemetryModules predeterminados

Application Insights usa módulos de telemetría para recopilar automáticamente información de telemetría sobre cargas de trabajo específicas sin necesidad de seguimiento manual.

Los siguientes módulos de recopilación automática están habilitados de forma predeterminada. Estos módulos son responsables de recopilar automáticamente datos de telemetría. Puede deshabilitarlos o configurarlos para modificar su comportamiento predeterminado.

* `DependencyTrackingTelemetryModule`
* `PerformanceCollectorModule`
* `QuickPulseTelemetryModule`
* `AppServicesHeartbeatTelemetryModule` - (Actualmente hay un problema relacionado con este módulo de telemetría. Para obtener una solución temporal, consulte el [problema de GitHub 1689](https://github.com/microsoft/ApplicationInsights-dotnet/issues/1689
)).
* `AzureInstanceMetadataTelemetryModule`

Para configurar cualquier objeto `TelemetryModule` predeterminado, use el método de la extensión `ConfigureTelemetryModule<T>` en `IServiceCollection`, como se muestra en el ejemplo siguiente.

```csharp
    using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
    using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector;

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetryWorkerService();

            // The following configures QuickPulseTelemetryModule.
            // Similarly, any other default modules can be configured.
            services.ConfigureTelemetryModule<QuickPulseTelemetryModule>((module, o) =>
            {
                module.AuthenticationApiKey = "keyhere";
            });

            // The following removes PerformanceCollectorModule to disable perf-counter collection.
            // Similarly, any other default modules can be removed.
            var performanceCounterService = services.FirstOrDefault<ServiceDescriptor>
                                        (t => t.ImplementationType == typeof(PerformanceCollectorModule));
            if (performanceCounterService != null)
            {
                services.Remove(performanceCounterService);
            }
    }
```

### <a name="configuring-telemetry-channel"></a>Configuración del canal de telemetría

El canal predeterminado es `ServerTelemetryChannel`. Puede invalidarlo tal como se muestra en el ejemplo siguiente.

```csharp
using Microsoft.ApplicationInsights.Channel;

    public void ConfigureServices(IServiceCollection services)
    {
        // Use the following to replace the default channel with InMemoryChannel.
        // This can also be applied to ServerTelemetryChannel.
        services.AddSingleton(typeof(ITelemetryChannel), new InMemoryChannel() {MaxTelemetryBufferCapacity = 19898 });

        services.AddApplicationInsightsTelemetryWorkerService();
    }
```

### <a name="disable-telemetry-dynamically"></a>Deshabilitar la telemetría dinámicamente

Si desea deshabilitar la telemetría condicional y dinámicamente, puede resolver la instancia `TelemetryConfiguration` con el contenedor de inyección de dependencias de ASP.NET Core en cualquier parte del código y establecer `DisableTelemetry` como marca.

```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetryWorkerService();
    }

    public void Configure(IApplicationBuilder app, IHostingEnvironment env, TelemetryConfiguration configuration)
    {
        configuration.DisableTelemetry = true;
        ...
    }
```

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="how-can-i-track-telemetry-thats-not-automatically-collected"></a>¿Cómo puedo realizar un seguimiento de la telemetría que se recopila automáticamente?

Obtenga una instancia de `TelemetryClient` mediante el uso de la inserción del constructor y llame al método `TrackXXX()` necesario que incluye. No se recomienda crear instancias de `TelemetryClient`. Una instancia singleton de `TelemetryClient` ya está registrada en el contenedor `DependencyInjection`, que comparte `TelemetryConfiguration` con el resto de los datos de telemetría. Se recomienda crear una nueva instancia `TelemetryClient` solo si se necesita una configuración independiente del resto de los datos de telemetría.

### <a name="can-i-use-visual-studio-ide-to-onboard-application-insights-to-a-worker-service-project"></a>¿Puedo usar el IDE de Visual Studio para incorporar Application Insights a un proyecto de servicio de trabajo?

La incorporación del IDE de Visual Studio se admite actualmente solo para aplicaciones ASP.NET/ASP.NET Core. Este documento se actualizará cuando Visual Studio ofrezca compatibilidad con las aplicaciones de servicio de trabajo de incorporación.

### <a name="can-i-enable-application-insights-monitoring-by-using-tools-like-status-monitor"></a>¿Puedo habilitar la supervisión de Application Insights con herramientas como Monitor de estado?

No. [Monitor de estado](./monitor-performance-live-website-now.md) y [Monitor de estado v2](./status-monitor-v2-overview.md) actualmente solo son compatibles con ASP.NET 4.x.

### <a name="if-i-run-my-application-in-linux-are-all-features-supported"></a>Si ejecuto mi aplicación en Linux, ¿se admiten todas las características?

Sí. La compatibilidad de características para el SDK es la misma en todas las plataformas, con las siguientes excepciones:

* Los contadores de rendimiento solo se admiten en Windows, con la excepción de CPU o memoria de procesos que se muestran en Live Metrics.
* Aunque el objeto `ServerTelemetryChannel` está habilitado de forma predeterminada, si la aplicación se ejecuta en Linux o MacOS, el canal no crea automáticamente una carpeta de almacenamiento local para mantener los datos de telemetría temporalmente si hay problemas de red. Debido a esta limitación, la telemetría se pierde cuando hay problemas temporales de red o del servidor. Para solucionar este problema, configure una carpeta local para el canal:

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;

    public void ConfigureServices(IServiceCollection services)
    {
        // The following will configure the channel to use the given folder to temporarily
        // store telemetry items during network or Application Insights server issues.
        // User should ensure that the given folder already exists
        // and that the application has read/write permissions.
        services.AddSingleton(typeof(ITelemetryChannel),
                                new ServerTelemetryChannel () {StorageFolder = "/tmp/myfolder"});
        services.AddApplicationInsightsTelemetryWorkerService();
    }
```

## <a name="sample-applications"></a>Aplicaciones de ejemplo

[Aplicación de consola .NET Core](https://github.com/microsoft/ApplicationInsights-Home/tree/master/Samples/WorkerServiceSDK/ConsoleAppWithApplicationInsights): use este ejemplo si usa una aplicación de consola escrita en .NET Core (versión 2.0 o posterior) o .NET Framework (versión 4.7.2 o posterior).

[Tareas en segundo plano de ASP.NET Core con HostedServices](https://github.com/microsoft/ApplicationInsights-Home/tree/master/Samples/WorkerServiceSDK/BackgroundTasksWithHostedService): use este ejemplo si está en ASP.NET Core 2.1/2.2 y cree tareas en segundo plano según la orientación oficial que se muestra [aquí](/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-2.2&preserve-view=true).

[Servicio de trabajo .NET Core 3.0](https://github.com/microsoft/ApplicationInsights-Home/tree/master/Samples/WorkerServiceSDK/WorkerServiceSampleWithApplicationInsights): use este ejemplo si tiene una aplicación de servicio de trabajo .NET Core 3.0 según las instrucciones oficiales que se muestran [aquí](/aspnet/core/fundamentals/host/hosted-services?tabs=visual-studio&view=aspnetcore-3.0&preserve-view=true#worker-service-template).

## <a name="open-source-sdk"></a>SDK de código abierto

[Lectura y contribución al código](https://github.com/Microsoft/ApplicationInsights-aspnetcore#recent-updates).

## <a name="next-steps"></a>Pasos siguientes

* [Use la API](./api-custom-events-metrics.md) para enviar sus propios eventos y métricas para obtener una vista detallada del rendimiento y del uso de la aplicación.
* [Realice el seguimiento de las dependencias adicionales cuyo seguimiento no se realiza automáticamente](./auto-collect-dependencies.md).
* [Enriquezca o filtre la telemetría recopilada automáticamente](./api-filtering-sampling.md).
* [Inserción de dependencias en ASP.NET Core](/aspnet/core/fundamentals/dependency-injection).
