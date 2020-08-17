---
title: 'Exploración de los registros de seguimiento de .NET con ILogger: Azure Application Insights'
description: Ejemplos de uso del proveedor ILogger para Azure Application Insights con aplicaciones de ASP.NET Core y de consola.
ms.topic: conceptual
ms.date: 02/19/2019
ms.reviewer: mbullwin
ms.openlocfilehash: 171aaeb624bfedb9aa7408a736c11faca316b392
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87322642"
---
# <a name="applicationinsightsloggerprovider-for-net-core-ilogger-logs"></a>ApplicationInsightsLoggerProvider para los registros de ILogger de .NET Core

ASP.NET Core es compatible con una API de registro que funciona con diferentes tipos de proveedores de registro integrados y de terceros. El registro se realiza mediante una llamada a **Log()** o una variante de este en instancias de *ILogger*. En este artículo se muestra cómo usar *ApplicationInsightsLoggerProvider* para capturar los registros de ILogger en aplicaciones de consola y de ASP.NET Core. También se describe cómo ApplicationInsightsLoggerProvider se integra con otros datos de telemetría de Application Insights.
Para más información, consulte [Registro en ASP.NET Core](/aspnet/core/fundamentals/logging).

## <a name="aspnet-core-applications"></a>Aplicaciones de ASP.NET Core

ApplicationInsightsLoggerProvider se habilita de forma predeterminada en el [SDK Microsoft.ApplicationInsights.AspNet](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) versión 2.7.1 (y posteriores) al activar una supervisión normal de Application Insights mediante uno de los siguientes métodos:

- Mediante una llamada al método de extensión **UseApplicationInsights** en IWebHostBuilder (ahora obsoleto)
- Mediante una llamada al método de extensión **AddApplicationInsightsTelemetry** en IServiceCollection

Los registros de ILogger que captura ApplicationInsightsLoggerProvider están sujetos a la misma configuración que los otros datos de telemetría recopilados. Tienen el mismo conjunto de objetos TelemetryInitializer y TelemetryProcessor, usan el mismo objeto TelemetryChannel, y se ponen en correlación y muestrean de la misma manera que otros datos de telemetría. Si usa la versión 2.7.1 o posteriores, no necesita realizar ninguna acción para capturar registros de ILogger.

De forma predeterminada, solo se envían a Application Insights los registros de *Advertencia* o superiores de ILogger (de todas las [categorías](/aspnet/core/fundamentals/logging/?view=aspnetcore-3.1#log-category)). Sin embargo, puede [aplicar filtros para modificar este comportamiento](#control-logging-level). Se necesitan pasos adicionales para capturar registros de ILogger desde **Program.cs** o **Startup.cs** (consulte [Captura de registros de ILogger de Startup.cs y Program.cs en aplicaciones de ASP.NET Core](#capture-ilogger-logs-from-startupcs-and-programcs-in-aspnet-core-apps)).

Si usa una versión anterior del SDK de Microsoft.ApplicationInsights.AspNet o desea usar solo ApplicationInsightsLoggerProvider sin ninguna otra supervisión de Application Insights, use el procedimiento siguiente:

1. Instale el paquete de NuGet:

   ```xml
       <ItemGroup>
         <PackageReference Include="Microsoft.Extensions.Logging.ApplicationInsights" Version="2.9.1" />  
       </ItemGroup>
   ```

1. Modifique **Program.cs** tal como se muestra aquí:

   ```csharp
   using Microsoft.AspNetCore;
   using Microsoft.AspNetCore.Hosting;
   using Microsoft.Extensions.Logging;

   public class Program
   {
       public static void Main(string[] args)
       {
           CreateWebHostBuilder(args).Build().Run();
       }

       public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
         WebHost.CreateDefaultBuilder(args)
           .UseStartup<Startup>()
         .ConfigureLogging(
               builder =>
               {
                   // Providing an instrumentation key here is required if you're using
                   // standalone package Microsoft.Extensions.Logging.ApplicationInsights
                   // or if you want to capture logs from early in the application startup
                   // pipeline from Startup.cs or Program.cs itself.
                   builder.AddApplicationInsights("ikey");

                   // Optional: Apply filters to control what logs are sent to Application Insights.
                   // The following configures LogLevel Information or above to be sent to
                   // Application Insights for all categories.
                   builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                                    ("", LogLevel.Information);
               }
           );
   }
   ```

El código del paso 2 configura `ApplicationInsightsLoggerProvider`. El código siguiente muestra un ejemplo de la clase Controller, que usa `ILogger` para enviar registros. Application Insights captura los registros.

```csharp
public class ValuesController : ControllerBase
{
    private readonly ILogger _logger;

    public ValuesController(ILogger<ValuesController> logger)
    {
        _logger = logger;
    }

    // GET api/values
    [HttpGet]
    public ActionResult<IEnumerable<string>> Get()
    {
        // All the following logs will be picked up by Application Insights.
        // and all of them will have ("MyKey", "MyValue") in Properties.
        using (_logger.BeginScope(new Dictionary<string, object> { { "MyKey", "MyValue" } }))
            {
                _logger.LogWarning("An example of a Warning trace..");
                _logger.LogError("An example of an Error level message");
            }
        return new string[] { "value1", "value2" };
    }
}
```

### <a name="capture-ilogger-logs-from-startupcs-and-programcs-in-aspnet-core-apps"></a>Captura de registros de ILogger de Startup.cs y Program.cs en aplicaciones de ASP.NET Core

> [!NOTE]
> En ASP.NET Core 3.0 y versiones posteriores, ya no es posible insertar `ILogger` en Startup.cs y Program.cs. Para obtener más información, consulte https://github.com/aspnet/Announcements/issues/353.

El nuevo proveedor ApplicationInsightsLoggerProvider puede capturar registros antes en la canalización de inicio de una aplicación. Aunque ApplicationInsightsLoggerProvider está habilitado automáticamente en Application Insights (a partir de la versión 2.7.1), no tiene una clave de instrumentación configurada hasta más adelante en la canalización. Por lo tanto, solo se capturarán registros de **Controller** y otras clases. Para capturar todos los registros, ya desde **Program.cs** y **Startup.cs**, debe habilitar explícitamente una clave de instrumentación para ApplicationInsightsLoggerProvider. Además, la configuración de *TelemetryConfiguration* no está completa cuando se registra desde los propios **Program.cs** o **Startup.cs**. Por lo tanto, esos registros tendrán una configuración mínima que utiliza InMemoryChannel, sin muestreo y sin inicializadores ni procesadores de telemetría estándar.

En los siguientes ejemplos se muestra esta funcionalidad con **Program.cs** y **Startup.cs**.

#### <a name="example-programcs"></a>Ejemplo de Program.cs

```csharp
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Logging;

public class Program
{
    public static void Main(string[] args)
    {
        var host = CreateWebHostBuilder(args).Build();
        var logger = host.Services.GetRequiredService<ILogger<Program>>();
        // This will be picked up by AI
        logger.LogInformation("From Program. Running the host now..");
        host.Run();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureLogging(
        builder =>
            {
            // Providing an instrumentation key here is required if you're using
            // standalone package Microsoft.Extensions.Logging.ApplicationInsights
            // or if you want to capture logs from early in the application startup 
            // pipeline from Startup.cs or Program.cs itself.
            builder.AddApplicationInsights("ikey");

            // Adding the filter below to ensure logs of all severity from Program.cs
            // is sent to ApplicationInsights.
            builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                             (typeof(Program).FullName, LogLevel.Trace);

            // Adding the filter below to ensure logs of all severity from Startup.cs
            // is sent to ApplicationInsights.
            builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                             (typeof(Startup).FullName, LogLevel.Trace);
            }
        );
}
```

#### <a name="example-startupcs"></a>Ejemplo de Startup.cs

```csharp
public class Startup
{
    private readonly ILogger _logger;

    public Startup(IConfiguration configuration, ILogger<Startup> logger)
    {
        Configuration = configuration;
        _logger = logger;
    }

    public IConfiguration Configuration { get; }

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddApplicationInsightsTelemetry();

        // The following will be picked up by Application Insights.
        _logger.LogInformation("Logging from ConfigureServices.");
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            // The following will be picked up by Application Insights.
            _logger.LogInformation("Configuring for Development environment");
            app.UseDeveloperExceptionPage();
        }
        else
        {
            // The following will be picked up by Application Insights.
            _logger.LogInformation("Configuring for Production environment");
        }

        app.UseMvc();
    }
}
```

## <a name="migrate-from-the-old-applicationinsightsloggerprovider"></a>Migración desde el antiguo ApplicationInsightsLoggerProvider

Las versiones del SDK de Microsoft.ApplicationInsights.AspNet anteriores a 2.7.1 eran compatibles con un proveedor de registro que ahora está obsoleto. Este proveedor se habilitaba a través del método de extensión **AddApplicationInsights()** de ILoggerFactory. Le recomendamos que migre al nuevo proveedor, lo que implica dos pasos:

1. Quitar la llamada a *ILoggerFactory.AddApplicationInsights()* del método **Startup.Configure()** para evitar un registro doble.
2. Volver a aplicar las reglas de filtrado en el código, porque el nuevo proveedor no las respetará. Las sobrecargas de *ILoggerFactory.AddApplicationInsights()* incluían un mínimo de funciones de LogLevel o filtro. Con el nuevo proveedor, el filtrado forma parte de la propia plataforma de registro. Ya no lo realiza el proveedor de Application Insights. Por lo tanto, deben quitarse los filtros que se proporcionan a través de las sobrecargas de *ILoggerFactory.AddApplicationInsights()* . Las reglas de filtrado deben proporcionarse siguiendo las instrucciones de [Control del nivel de registro](#control-logging-level). Si usa *appsettings.json* para filtrar el registro, seguirá funcionando con el nuevo proveedor, ya que ambos usan el mismo alias de proveedor, *ApplicationInsights*.

Todavía puede usar el proveedor anterior (solo se quitará en un cambio de versión principal a 3.*xx*). Sin embargo, le recomendamos que migre al nuevo proveedor, por los siguientes motivos:

- El proveedor anterior no es compatible con [ámbitos de registro](/aspnet/core/fundamentals/logging/?view=aspnetcore-2.2#log-scopes). En el nuevo proveedor, las propiedades del ámbito se agregan automáticamente como propiedades personalizadas a los datos de telemetría recopilados.
- Ahora se pueden capturar registros mucho antes en la canalización de inicio de la aplicación. Ya se pueden capturar registros de las clases **Program** y **Startup**.
- Con el nuevo proveedor, el filtrado se realiza en el propio nivel de la plataforma. Puede filtrar los registros del proveedor de Application Insights de la misma manera que en otros proveedores, incluidos los proveedores integrados, como la consola, depuración, etcétera. También puede aplicar los mismos filtros a varios proveedores.
- En ASP.NET Core (versión 2.0 y posteriores), el método recomendado para [habilitar proveedores de registro](https://github.com/aspnet/Announcements/issues/255) consiste en usar métodos de extensión en ILoggingBuilder en el propio archivo **Program.cs**.

> [!Note]
> El nuevo proveedor está disponible para aplicaciones dirigidas a NETSTANDARD2.0 o posterior. Desde [SDK Microsoft.ApplicationInsights.AspNet](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) versión 2.14.0, también hay un nuevo proveedor disponible para las aplicaciones que tienen como destino .NET Framework NET461 o posterior. Si la aplicación se dirige a versiones anteriores de .NET Core, como .NET Core 1.1, o a .NET Framework anteriores a NET46, siga usando el proveedor anterior.

## <a name="console-application"></a>Aplicación de consola

> [!NOTE]
> Hay un nuevo SDK de Application Insights llamado [Microsoft.ApplicationInsights.WorkerService](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WorkerService) que se puede usar para habilitar Application Insights (ILogger y otra información de telemetría de Application Insights) en cualquier aplicación de consola. Se recomienda usar este paquete y las instrucciones asociadas que se indican [aquí](./worker-service.md).

En el siguiente código se muestra una aplicación de ejemplo de la consola configurada para enviar seguimientos de ILogger a Application Insights.

Paquetes instalados:

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="2.1.0" />  
  <PackageReference Include="Microsoft.Extensions.Logging.ApplicationInsights" Version="2.9.1" />
  <PackageReference Include="Microsoft.Extensions.Logging.Console" Version="2.1.0" />
</ItemGroup>
```

```csharp
class Program
{
    static void Main(string[] args)
    {
        // Create the DI container.
        IServiceCollection services = new ServiceCollection();

        // Channel is explicitly configured to do flush on it later.
        var channel = new InMemoryChannel();
        services.Configure<TelemetryConfiguration>(
            (config) =>
            {
                config.TelemetryChannel = channel;
            }
        );

        // Add the logging pipelines to use. We are using Application Insights only here.
        services.AddLogging(builder =>
        {
            // Optional: Apply filters to configure LogLevel Trace or above is sent to
            // Application Insights for all categories.
            builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                             ("", LogLevel.Trace);
            builder.AddApplicationInsights("--YourAIKeyHere--");
        });

        // Build ServiceProvider.
        IServiceProvider serviceProvider = services.BuildServiceProvider();

        ILogger<Program> logger = serviceProvider.GetRequiredService<ILogger<Program>>();

        // Begin a new scope. This is optional.
        using (logger.BeginScope(new Dictionary<string, object> { { "Method", nameof(Main) } }))
        {
            logger.LogInformation("Logger is working"); // this will be captured by Application Insights.
        }

        // Explicitly call Flush() followed by sleep is required in Console Apps.
        // This is to ensure that even if application terminates, telemetry is sent to the back-end.
        channel.Flush();
        Thread.Sleep(1000);
    }
}
```

Este ejemplo usa el paquete independiente `Microsoft.Extensions.Logging.ApplicationInsights`. De forma predeterminada, esta configuración utiliza la opción "mínima" de TelemetryConfiguration para enviar datos a Application Insights. Mínima significa que el canal que se utiliza es InMemoryChannel. No hay ningún muestreo ni TelemetryInitializers estándar. Este comportamiento se puede invalidar para una aplicación de consola, tal como se muestra en el siguiente ejemplo.

Instale este paquete adicional:

```xml
<PackageReference Include="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel" Version="2.9.1" />
```

En la sección siguiente se muestra cómo invalidar el valor predeterminado de TelemetryConfiguration con el método **services.Configure\<TelemetryConfiguration>()** . En este ejemplo se configura `ServerTelemetryChannel` y el muestreo. Se agrega un elemento ITelemetryInitializer personalizado a TelemetryConfiguration.

```csharp
    // Create the DI container.
    IServiceCollection services = new ServiceCollection();
    var serverChannel = new ServerTelemetryChannel();
    services.Configure<TelemetryConfiguration>(
        (config) =>
        {
            config.TelemetryChannel = serverChannel;
            config.TelemetryInitializers.Add(new MyTelemetryInitalizer());
            config.DefaultTelemetrySink.TelemetryProcessorChainBuilder.UseSampling(5);
            serverChannel.Initialize(config);
        }
    );

    // Add the logging pipelines to use. We are adding Application Insights only.
    services.AddLogging(loggingBuilder =>
    {
        loggingBuilder.AddApplicationInsights();
    });

    ........
    ........

    // Explicitly calling Flush() followed by sleep is required in Console Apps.
    // This is to ensure that even if the application terminates, telemetry is sent to the back end.
    serverChannel.Flush();
    Thread.Sleep(1000);
```

## <a name="control-logging-level"></a>Control del nivel de registro

La infraestructura *ILogger* de ASP.NET Core incluye un mecanismo integrado para aplicar el [filtrado del registro](/aspnet/core/fundamentals/logging/?view=aspnetcore-2.2#log-filtering). Esto permite controlar los registros que se envían a cada proveedor registrado, incluido el proveedor de Application Insights. El filtrado se puede realizar en la configuración (normalmente mediante un archivo *appsettings.json*) o en el código. Esta función se proporciona en la propia plataforma. No es específica del proveedor de Application Insights.

En los ejemplos siguientes se aplican reglas de filtro a ApplicationInsightsLoggerProvider.

### <a name="create-filter-rules-in-configuration-with-appsettingsjson"></a>Creación de reglas de filtro en la configuración con appsettings.json

Para ApplicationInsightsLoggerProvider, el alias de proveedor es `ApplicationInsights`. En la siguiente sección de *appsettings.json* se indica a los proveedores de registro que generalmente se registren en el nivel *Advertencia* y superiores. A continuación, invalida el `ApplicationInsightsLoggerProvider` para registrar las categorías que comienzan por "Microsoft" en el nivel *Error* y superiores.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Warning"
    },
    "ApplicationInsights": {
      "LogLevel": {
        "Microsoft": "Error"
      }
    }
  }
}
```

### <a name="create-filter-rules-in-code"></a>Creación de reglas de filtro en el código

En el siguiente fragmento de código se configuran los registros para que el nivel de *Warning* (Advertencia) y superiores de todas las categorías, y el nivel de *Error* y superiores de las categorías que comienzan con "Microsoft"se envíen a `ApplicationInsightsLoggerProvider`. Esta configuración es la misma que en la sección anterior en *appsettings.json*.

```csharp
    WebHost.CreateDefaultBuilder(args)
    .UseStartup<Startup>()
    .ConfigureLogging(logging =>
      logging.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                        ("", LogLevel.Warning)
             .AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                        ("Microsoft", LogLevel.Error);
```

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="what-are-the-old-and-new-versions-of-applicationinsightsloggerprovider"></a>¿Cuáles son las versiones antigua y nueva de ApplicationInsightsLoggerProvider?

El [SDK de Microsoft.ApplicationInsights.AspNet](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) incluía un elemento ApplicationInsightsLoggerProvider integrado (Microsoft.ApplicationInsights.AspNetCore.Logging.ApplicationInsightsLoggerProvider), que se habilitaba a través de métodos de extensión de  **ILoggerFactory**. Este proveedor está marcado como obsoleto desde la versión 2.7.1. Se quitará por completo en el siguiente cambio de versión principal. El paquete [Microsoft.ApplicationInsights.AspNetCore 2.6.1](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) en sí no está obsoleto. Se necesita para habilitar la supervisión de las solicitudes, las dependencias, etcétera.

La alternativa propuesta es el nuevo paquete independiente [Microsoft.Extensions.Logging.ApplicationInsights](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights), que contiene un proveedor ApplicationInsightsLoggerProvider (Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider) y métodos de extensión en ILoggerBuilder para habilitarlo.

El [SDK de Microsoft.ApplicationInsights.AspNet](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) versión 2.7.1 depende del nuevo paquete y habilita la captura de ILogger automáticamente.

### <a name="why-are-some-ilogger-logs-shown-twice-in-application-insights"></a>¿Por qué algunos registros de ILogger se muestran dos veces en Application Insights?

La duplicación puede ocurrir si tiene la versión anterior (ya obsoleta) de ApplicationInsightsLoggerProvider habilitada mediante una llamada a `AddApplicationInsights` en `ILoggerFactory`. Compruebe si el método **Configure** incluye la siguiente información y quítela:

```csharp
 public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
 {
     loggerFactory.AddApplicationInsights(app.ApplicationServices, LogLevel.Warning);
     // ..other code.
 }
```

Si se produce un registro doble cuando se depura desde Visual Studio, establezca `EnableDebugLogger` en *false* en el código que habilita Application Insights, como se indica a continuación. Esta duplicación y la corrección solo son pertinentes cuando se depura la aplicación.

```csharp
 public void ConfigureServices(IServiceCollection services)
 {
     ApplicationInsightsServiceOptions options = new ApplicationInsightsServiceOptions();
     options.EnableDebugLogger = false;
     services.AddApplicationInsightsTelemetry(options);
     // ..other code.
 }
```

### <a name="i-updated-to-microsoftapplicationinsightsaspnet-sdk-version-271-and-logs-from-ilogger-are-captured-automatically-how-do-i-turn-off-this-feature-completely"></a>Actualicé al [SDK de Microsoft.ApplicationInsights.AspNet](https://www.nuget.org/packages/Microsoft.ApplicationInsights.AspNetCore) versión 2.7.1 y los registros de ILogger se capturan automáticamente. ¿Cómo desactivo por completo esta característica?

Consulte en la sección [Control del nivel de registro](#control-logging-level) cómo filtrar los registros en general. Para desactivar ApplicationInsightsLoggerProvider, utilice `LogLevel.None`:

**En el código:**

```csharp
    builder.AddFilter<Microsoft.Extensions.Logging.ApplicationInsights.ApplicationInsightsLoggerProvider>
                      ("", LogLevel.None);
```

**En la configuración:**

```json
{
  "Logging": {
    "ApplicationInsights": {
      "LogLevel": {
        "Default": "None"
      }
}
```

### <a name="why-do-some-ilogger-logs-not-have-the-same-properties-as-others"></a>¿Por qué algunos registros de ILogger no tiene las mismas propiedades que otros?

Application Insights captura y envía los registros de ILogger con el mismo valor de TelemetryConfiguration que se usa para los demás datos de telemetría. Sin embargo, hay una excepción. De forma predeterminada, la configuración de TelemetryConfiguration no está completa cuando se registra desde **Program.cs** o **Startup.cs**. Los registros con estas procedencias no tendrán la configuración predeterminada, por lo que no se ejecutarán todos los objetos TelemetryInitializer y TelemetryProcessor.

### <a name="im-using-the-standalone-package-microsoftextensionsloggingapplicationinsights-and-i-want-to-log-some-additional-custom-telemetry-manually-how-should-i-do-that"></a>Estoy usando el paquete independiente Microsoft.Extensions.Logging.ApplicationInsights y quiero registrar manualmente algunos datos de telemetría personalizados adicionales. ¿Cómo debo hacerlo?

Cuando se usa el paquete independiente, `TelemetryClient` no se inserta en el contenedor de DI, por lo que necesita crear una nueva instancia de `TelemetryClient` y utilizar la misma configuración que utilice el proveedor de registro, como se muestra en el código siguiente. Esto garantiza que se usa la misma configuración para todos los datos de telemetría personalizados y los datos de telemetría de ILogger.

```csharp
public class MyController : ApiController
{
   // This telemetryclient can be used to track additional telemetry using TrackXXX() api.
   private readonly TelemetryClient _telemetryClient;
   private readonly ILogger _logger;

   public MyController(IOptions<TelemetryConfiguration> options, ILogger<MyController> logger)
   {
        _telemetryClient = new TelemetryClient(options.Value);
        _logger = logger;
   }  
}
```

> [!NOTE]
> Si utiliza el paquete Microsoft.ApplicationInsights.AspNetCore para habilitar Application Insights, modifique este código para obtener `TelemetryClient` directamente en el constructor. Para ver un ejemplo, consulte [esta pregunta frecuente](./asp-net-core.md#frequently-asked-questions).


### <a name="what-application-insights-telemetry-type-is-produced-from-ilogger-logs-or-where-can-i-see-ilogger-logs-in-application-insights"></a>¿Qué tipo de datos de telemetría de Application Insights se generan a partir de los registros de ILogger? O bien, ¿dónde puedo ver los registros de ILogger en Application Insights?

ApplicationInsightsLoggerProvider captura los registros de ILogger y crea un elemento TraceTelemetry a partir de ellos. Si se pasa un objeto Exception al método **Log()** de ILogger, se crea un elemento *ExceptionTelemetry* en lugar de TraceTelemetry. Estos elementos de telemetría pueden encontrarse en los mismos lugares que otros elementos TraceTelemetry o ExceptionTelemetry para Application Insights, incluido el portal, análisis o el depurador local de Visual Studio.

Si prefiere enviar siempre un elemento TraceTelemetry, use este fragmento de código: ```builder.AddApplicationInsights((opt) => opt.TrackExceptionsAsExceptionTelemetry = false);```

### <a name="i-dont-have-the-sdk-installed-and-i-use-the-azure-web-apps-extension-to-enable-application-insights-for-my-aspnet-core-applications-how-do-i-use-the-new-provider"></a>No tengo instalado el SDK y uso la extensión Web Apps de Azure para habilitar Application Insights para mis aplicaciones de ASP.NET Core. ¿Cómo uso el nuevo proveedor? 

La extensión de Application Insights en Azure Web Apps usa el nuevo proveedor. Puede modificar las reglas de filtrado en el archivo *appsettings.json* para la aplicación.

### <a name="im-using-the-standalone-package-microsoftextensionsloggingapplicationinsights-and-enabling-application-insights-provider-by-calling-builderaddapplicationinsightsikey-is-there-an-option-to-get-an-instrumentation-key-from-configuration"></a>Estoy usando el paquete independiente Microsoft.Extensions.Logging.ApplicationInsights y habilito el proveedor de Application Insights mediante una llamada a **builder.AddApplicationInsights("ikey")** . ¿Existe alguna opción para obtener una clave de instrumentación a partir de la configuración?


Modifique Program.cs y appsettings.json como se indica a continuación:

   ```csharp
   public class Program
   {
       public static void Main(string[] args)
       {
           CreateWebHostBuilder(args).Build().Run();
       }

       public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
           WebHost.CreateDefaultBuilder(args)
               .UseStartup<Startup>()
               .ConfigureLogging((hostingContext, logging) =>
               {
                   // hostingContext.HostingEnvironment can be used to determine environments as well.
                   var appInsightKey = hostingContext.Configuration["myikeyfromconfig"];
                   logging.AddApplicationInsights(appInsightKey);
               });
   }
   ```

   Sección relevante de `appsettings.json`:

   ```json
   {
     "myikeyfromconfig": "putrealikeyhere"
   }
   ```

Este código solo es necesario cuando se usa un proveedor de registro independiente. En la supervisión normal de Application Insights, la clave de instrumentación se carga automáticamente desde la ruta de acceso de configuración *ApplicationInsights: Instrumentationkey*. Appsettings.json debe tener este aspecto:

   ```json
   {
     "ApplicationInsights":
       {
           "Instrumentationkey":"putrealikeyhere"
       }
   }
   ```

## <a name="next-steps"></a>Pasos siguientes

Más información sobre:

* [Registro en ASP.NET Core](/aspnet/core/fundamentals/logging)
* [Registros de seguimiento de .NET en Application Insights](./asp-net-trace-logs.md)

