---
title: Muestreo de telemetría en Azure Application Insights | Microsoft Docs
description: Cómo mantener el volumen de telemetría bajo control.
ms.topic: conceptual
ms.date: 01/17/2020
ms.reviewer: vitalyg
ms.custom: fasttrack-edit
ms.openlocfilehash: bb6793bc1e3d5bb55426c1f344520ae19a22a9f9
ms.sourcegitcommit: 023d10b4127f50f301995d44f2b4499cbcffb8fc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/18/2020
ms.locfileid: "88549572"
---
# <a name="sampling-in-application-insights"></a>Muestreo en Application Insights.

El muestreo es una característica de [Azure Application Insights](./app-insights-overview.md). Se trata de la manera recomendada de reducir el tráfico de telemetría, los costos de datos y los costos de almacenamiento conservando, al mismo tiempo, un análisis estadísticamente correcto de los datos de las aplicaciones. El muestreo también le ayuda a evitar que Application Insights limite los datos de telemetría. El filtro de muestreo selecciona los elementos relacionados para que pueda desplazarse entre los elementos cuando realiza investigaciones de diagnóstico.

Cuando los recuentos de métrica se presentan en el portal, se vuelven a normalizar para tener en cuenta el muestreo. De este modo, se minimizan los efectos en las estadísticas.

## <a name="brief-summary"></a>Resumen breve

* Hay tres tipos diferentes de muestreo: muestreo adaptable, muestreo de frecuencia fija y muestreo de ingesta.
* El muestreo adaptable está habilitado de manera predeterminada en todas las versiones más recientes de los kits de desarrollo de software (SDK) de Application Insights para ASP.NET y ASP.NET Core. También lo usa [Azure Functions](../../azure-functions/functions-overview.md).
* El muestreo de frecuencia fija está disponible en las versiones recientes de los SDK de Application Insights para ASP.NET, ASP.NET Core, Java (tanto el agente como el SDK) y Python.
* El muestreo de ingesta funciona en el punto de conexión de servicio de Application Insights. Solo se aplica cuando no hay ningún otro muestreo en vigor. Si el SDK muestrea los datos de telemetría, el muestreo de ingesta está deshabilitado.
* Para aplicaciones web, si registra eventos personalizados y necesita asegurarse de que un conjunto de eventos se conserva o descarta en conjunto, los eventos deben tener el mismo valor de `OperationId`.
* Si escribe consultas de Analytics, debería [tener en cuenta el muestreo](../log-query/aggregations.md). En concreto, en lugar de simplemente contar registros, debería usar `summarize sum(itemCount)`.
* Algunos tipos de telemetría, incluidas las métricas de rendimiento y las métricas personalizadas, siempre se conservan independientemente de si el muestreo está habilitado o no.

En la tabla siguiente se resumen los tipos de muestreo disponibles para cada SDK y tipo de aplicación:

| SDK de Application Insights | Se admite el muestreo adaptable | Se admite el muestreo de frecuencia fija | Se admite el muestreo de ingesta |
|-|-|-|-|
| ASP.NET | [Sí (activado de forma predeterminada)](#configuring-adaptive-sampling-for-aspnet-applications) | [Sí](#configuring-fixed-rate-sampling-for-aspnet-applications) | Solo si no hay otro muestreo en vigor |
| ASP.NET Core | [Sí (activado de forma predeterminada)](#configuring-adaptive-sampling-for-aspnet-core-applications) | [Sí](#configuring-fixed-rate-sampling-for-aspnet-core-applications) | Solo si no hay otro muestreo en vigor |
| Azure Functions | [Sí (activado de forma predeterminada)](#configuring-adaptive-sampling-for-azure-functions) | No | Solo si no hay otro muestreo en vigor |
| Java | No | [Sí](#configuring-fixed-rate-sampling-for-java-applications) | Solo si no hay otro muestreo en vigor |
| Node.JS | No | [Sí](./nodejs.md#sampling) | Solo si no hay otro muestreo en vigor
| Python | No | [Sí](#configuring-fixed-rate-sampling-for-opencensus-python-applications) | Solo si no hay otro muestreo en vigor |
| Todos los demás | No | No | [Sí](#ingestion-sampling) |

> [!NOTE]
> La información de la mayor parte de esta página se aplica a las versiones actuales de los SDK de Application Insights. Para obtener información sobre las versiones anteriores de los SDK, [consulte la sección siguiente](#older-sdk-versions).

## <a name="types-of-sampling"></a>Tipos de muestreo

Hay tres métodos de muestreo diferentes:

* El **muestreo adaptable** ajusta automáticamente el volumen de telemetría que se envía desde el SDK en la aplicación de ASP.NET/ASP.NET Core y desde Azure Functions. Este es el muestreo predeterminado cuando se usa el SDK de ASP.NET o ASP.NET Core. Actualmente, el muestreo adaptable solo está disponible para la telemetría del lado servidor de ASP.NET y para Azure Functions.

* El **muestreo de frecuencia fija** reduce el volumen de telemetría que se envía desde el servidor ASP.NET, ASP.NET Core o Java y desde los exploradores de los usuarios. El usuario establece la frecuencia. El cliente y el servidor sincronizarán su muestreo por lo que, en Búsqueda, puede desplazarse entre las solicitudes y las vistas de página relacionadas.

* El **muestreo de ingesta** se produce en el punto de conexión de servicio de Application Insights. Lo que hace es descartar algunos de los datos de telemetría que llegan desde su aplicación según la frecuencia de muestreo establecida. Aunque no reduce el tráfico de telemetría enviado desde su aplicación, le ayuda a mantenerse dentro de su cuota mensual. La principal ventaja del muestreo de ingesta es que puede establecer la frecuencia de muestreo sin volver a implementar la aplicación. El muestreo de ingesta funciona uniformemente para todos los servidores y clientes, pero no se aplica cuando están en funcionamiento otros tipos de muestreo.

> [!IMPORTANT]
> Si los métodos de muestreo adaptable o de frecuencia fija están en funcionamiento, el muestreo de ingesta se deshabilita.

## <a name="adaptive-sampling"></a>muestreo adaptable

Influye en el volumen de telemetría enviado desde su aplicación de servidor web hasta el punto de conexión del servicio de Application Insights.

> [!TIP]
> El muestreo adaptable está habilitado de forma predeterminada cuando se usa el SDK de ASP.NET o el SDK de ASP.NET Core y también está habilitado de forma predeterminada para Azure Functions.

El volumen se ajusta automáticamente para mantenerse dentro de una frecuencia máxima especificada de tráfico y se controla mediante el valor `MaxTelemetryItemsPerSecond`. Si la aplicación genera una cantidad baja de telemetría, como al depurar o debido a un uso bajo, el procesador de muestreo no podrá descargar los elementos mientras el volumen se encuentre por debajo de `MaxTelemetryItemsPerSecond`. A medida que aumenta el volumen de telemetría, la frecuencia de muestreo se ajusta para lograr el volumen objetivo. El ajuste se actualiza a intervalos regulares y se basa en una media móvil de la velocidad de transmisión de salida.

Para lograr el volumen objetivo, se descartan algunos de los datos de telemetría generados. Pero, al igual que otros tipos de muestreo, el algoritmo conserva los elementos de telemetría relacionados. Por ejemplo, cuando se inspeccione la telemetría en Búsqueda, podrá buscar la solicitud relacionada con una excepción determinada.

Los recuentos de métrica, como la tasa de solicitudes y la tasa de excepciones, se ajustan para compensar la frecuencia de muestreo, de modo que exhiban valores aproximadamente correctos en el Explorador de métricas.

### <a name="configuring-adaptive-sampling-for-aspnet-applications"></a>Configuración del muestreo adaptable para aplicaciones de ASP.NET

> [!NOTE]
> Esta sección se aplica a las aplicaciones de ASP.NET, no a las aplicaciones de ASP.NET Core. [Obtenga información sobre cómo configurar el muestreo adaptable para aplicaciones de ASP.NET Core más adelante en este documento.](#configuring-adaptive-sampling-for-aspnet-core-applications)

En [`ApplicationInsights.config`](./configuration-with-applicationinsights-config.md), puede ajustar varios parámetros en el nodo `AdaptiveSamplingTelemetryProcessor`. Las cifras que se muestran son los valores predeterminados:

* `<MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>`
  
    La velocidad de destino de las [operaciones lógicas ](./correlation.md#data-model-for-telemetry-correlation) que el algoritmo adaptativo pretende recopilar **en cada host de servidor**. Si la aplicación web se ejecuta en varios hosts, reduzca este valor para que se mantenga dentro de la velocidad objetivo de tráfico en el portal de Application Insights.

* `<EvaluationInterval>00:00:15</EvaluationInterval>` 
  
    Intervalo en el que se vuelve a evaluar la velocidad actual de telemetría. La evaluación se realiza como una media móvil. Se recomienda acortar este intervalo si la telemetría experimenta ráfagas repentinas.

* `<SamplingPercentageDecreaseTimeout>00:02:00</SamplingPercentageDecreaseTimeout>`
  
    Cuando se cambia el valor del porcentaje de muestreo, ¿qué tiempo mínimo se tarda en permitir de nuevo que se reduzca el porcentaje de muestreo para capturar menos datos?

* `<SamplingPercentageIncreaseTimeout>00:15:00</SamplingPercentageIncreaseTimeout>`
  
    Cuando se cambia el valor del porcentaje de muestreo, ¿qué tiempo mínimo se tarda en permitir de nuevo que se aumente el porcentaje de muestreo para capturar más datos?

* `<MinSamplingPercentage>0.1</MinSamplingPercentage>`
  
    A medida que el porcentaje de muestreo varía, ¿cuál es el valor mínimo que se permite establecer?

* `<MaxSamplingPercentage>100.0</MaxSamplingPercentage>`
  
    A medida que el porcentaje de muestreo varía, ¿cuál es el valor máximo que se permite establecer?

* `<MovingAverageRatio>0.25</MovingAverageRatio>` 
  
    En el cálculo de la media móvil, especifica el peso que se debería asignar al valor más reciente. Use un valor igual o menor que 1. Los valores menores hacen que el algoritmo reaccione con menor agilidad a los cambios repentinos.

* `<InitialSamplingPercentage>100</InitialSamplingPercentage>`
  
    Cantidad de datos de telemetría que se muestra cuando se acaba de iniciar la aplicación. No reduzca este valor durante la depuración.

* `<ExcludedTypes>Trace;Exception</ExcludedTypes>`
  
    Una lista delimitada por puntos y coma de tipos que no desea que estén sujetos a muestreo. Los tipos reconocidos son: `Dependency`, `Event`, `Exception`, `PageView`, `Request` y `Trace`. Se transmiten todos los datos de telemetría de los tipos especificados; los tipos no especificados se muestrean.

* `<IncludedTypes>Request;Dependency</IncludedTypes>`
  
    Una lista delimitada por puntos y coma de tipos que desea que estén sujetos a muestreo. Los tipos reconocidos son: `Dependency`, `Event`, `Exception`, `PageView`, `Request` y `Trace`. Los tipos especificados se muestrean, todos los datos de telemetría del resto de tipos siempre se transmitirán.

**Para desactivar** el muestreo adaptable, elimine los nodos `AdaptiveSamplingTelemetryProcessor` de `ApplicationInsights.config`.

#### <a name="alternative-configure-adaptive-sampling-in-code"></a>Alternativa: Configuración del muestreo adaptable en el código

En lugar de establecer el parámetro de muestreo en el archivo `.config`, puede establecer estos valores mediante programación.

1. Elimine todos los nodos `AdaptiveSamplingTelemetryProcessor` del archivo `.config`.
2. Use el siguiente fragmento de código para configurar el muestreo adaptable:

    ```csharp
    using Microsoft.ApplicationInsights;
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.WindowsServer.Channel.Implementation;
    using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;
    
    // ...

    var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
    // For older versions of the Application Insights SDK, use the following line instead:
    // var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;

    // Enable AdaptiveSampling so as to keep overall telemetry volume to 5 items per second.
    builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5);

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));

    builder.Build();
    ```

    ([Más información sobre los procesadores de telemetría](./api-filtering-sampling.md#filtering)).

También puede ajustar la frecuencia de muestreo para cada tipo de telemetría individualmente o incluso puede excluir determinados tipos del muestreo:

```csharp
// The following configures adaptive sampling with 5 items per second, and also excludes Dependency telemetry from being subjected to sampling.
builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5, excludedTypes: "Dependency");
```

### <a name="configuring-adaptive-sampling-for-aspnet-core-applications"></a>Configuración del muestreo adaptable para aplicaciones de ASP.NET Core

No hay ningún objeto `ApplicationInsights.config` para aplicaciones de ASP.NET Core, por lo que todas las configuraciones se realizan mediante código.
El muestreo adaptable está habilitado de forma predeterminada para todas las aplicaciones de ASP.NET Core. Puede deshabilitar o personalizar el comportamiento de muestreo.

#### <a name="turning-off-adaptive-sampling"></a>Desactivación del muestreo adaptable

La característica de muestreo predeterminada se puede deshabilitar al agregar el servicio de Application Insights, en el método `ConfigureServices`, con `ApplicationInsightsServiceOptions` dentro del archivo `Startup.cs`:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ...

    var aiOptions = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
    aiOptions.EnableAdaptiveSampling = false;
    services.AddApplicationInsightsTelemetry(aiOptions);

    //...
}
```

El código anterior deshabilitará el muestreo adaptable. Siga los pasos a continuación para agregar el muestreo con más opciones de personalización.

#### <a name="configure-sampling-settings"></a>Configuración de las opciones de muestreo

Use los métodos de extensión de `TelemetryProcessorChainBuilder` como se muestra a continuación para personalizar el comportamiento de muestreo.

> [!IMPORTANT]
> Si usa este método para configurar el muestreo, asegúrese de establecer la propiedad `aiOptions.EnableAdaptiveSampling` en `false` al llamar a `AddApplicationInsightsTelemetry()`.

```csharp
using Microsoft.ApplicationInsights.Extensibility

public void Configure(IApplicationBuilder app, IHostingEnvironment env, TelemetryConfiguration configuration)
{
    var builder = configuration.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
    // For older versions of the Application Insights SDK, use the following line instead:
    // var builder = configuration.TelemetryProcessorChainBuilder;

    // Using adaptive sampling
    builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5);

    // Alternately, the following configures adaptive sampling with 5 items per second, and also excludes DependencyTelemetry from being subject to sampling.
    // builder.UseAdaptiveSampling(maxTelemetryItemsPerSecond:5, excludedTypes: "Dependency");

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));
    
    builder.Build();

    // ...
}
```

### <a name="configuring-adaptive-sampling-for-azure-functions"></a>Configuración del muestreo adaptable para Azure Functions

Siga las instrucciones de [esta página](../../azure-functions/functions-monitoring.md#configure-sampling) para configurar el muestreo adaptable para las aplicaciones que se ejecutan en Azure Functions.

## <a name="fixed-rate-sampling"></a>Muestreo de frecuencia fija

El muestreo de frecuencia fija reduce el tráfico enviado desde el servidor web y los exploradores web. A diferencia del muestreo adaptable, reduce la telemetría a una tasa fija que usted decide. El muestreo de frecuencia fija está disponible para las aplicaciones de ASP.NET, ASP.NET Core, Java y Python.

Al igual que otras técnicas de muestreo, esto también conserva elementos relacionados. También sincroniza el muestreo del cliente y del servidor para que los elementos relacionados se conserven; por ejemplo, cuando mira una vista de página en Búsqueda, puede encontrar sus solicitudes de servidor relacionadas. 

En el Explorador de métricas, las tasas, como los recuentos de solicitudes y de excepciones, se multiplican por un factor para compensar la frecuencia de muestreo, de forma que sean aproximadamente correctas.

### <a name="configuring-fixed-rate-sampling-for-aspnet-applications"></a>Configuración del muestreo de frecuencia fija para aplicaciones de ASP.NET

1. **Deshabilite el muestreo adaptable**: En [`ApplicationInsights.config`](./configuration-with-applicationinsights-config.md), elimine o marque como comentario el nodo `AdaptiveSamplingTelemetryProcessor`.

    ```xml
    <TelemetryProcessors>
        <!-- Disabled adaptive sampling:
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
        </Add>
        -->
    ```

2. **Habilite el módulo de muestreo de frecuencia fija.** Agregue este fragmento de código a [`ApplicationInsights.config`](./configuration-with-applicationinsights-config.md):
   
    ```XML
    <TelemetryProcessors>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.SamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <!-- Set a percentage close to 100/N where N is an integer. -->
            <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
            <SamplingPercentage>10</SamplingPercentage>
        </Add>
    </TelemetryProcessors>
    ```

      Como alternativa, en lugar de establecer el parámetro de muestreo en el archivo `ApplicationInsights.config`, puede establecer estos valores mediante programación:

    ```csharp
    using Microsoft.ApplicationInsights.Extensibility;
    using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;

    // ...

    var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
    // For older versions of the Application Insights SDK, use the following line instead:
    // var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;

    builder.UseSampling(10.0); // percentage

    // If you have other telemetry processors:
    builder.Use((next) => new AnotherProcessor(next));

    builder.Build();
    ```

    ([Más información sobre los procesadores de telemetría](./api-filtering-sampling.md#filtering)).

### <a name="configuring-fixed-rate-sampling-for-aspnet-core-applications"></a>Configuración del muestreo de frecuencia fija para aplicaciones de ASP.NET Core

1. **Deshabilite el muestreo adaptable**:  Se pueden realizar cambios en el método `ConfigureServices` mediante `ApplicationInsightsServiceOptions`:

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        // ...

        var aiOptions = new Microsoft.ApplicationInsights.AspNetCore.Extensions.ApplicationInsightsServiceOptions();
        aiOptions.EnableAdaptiveSampling = false;
        services.AddApplicationInsightsTelemetry(aiOptions);

        //...
    }
    ```

2. **Habilite el módulo de muestreo de frecuencia fija.** Se pueden realizar cambios en el método `Configure`, tal como se muestra en el siguiente fragmento de código:

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        var configuration = app.ApplicationServices.GetService<TelemetryConfiguration>();

        var builder = configuration.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
        // For older versions of the Application Insights SDK, use the following line instead:
        // var builder = TelemetryConfiguration.Active.TelemetryProcessorChainBuilder;

        // Using fixed rate sampling   
        double fixedSamplingPercentage = 10;
        builder.UseSampling(fixedSamplingPercentage);

        builder.Build();

        // ...
    }
    ```

### <a name="configuring-fixed-rate-sampling-for-java-applications"></a>Configuración del muestreo de frecuencia fija para aplicaciones de Java

De manera predeterminada, no hay ningún muestreo habilitado en el agente ni en el SDK de Java. Actualmente, solo admite el muestreo de frecuencia fija. No se admite el muestreo adaptable en Java.

#### <a name="configuring-java-agent"></a>Configuración del agente de Java

1. Descargue [applicationinsights-agent-3.0.0-PREVIEW.5.jar](https://github.com/microsoft/ApplicationInsights-Java/releases/download/3.0.0-PREVIEW.5/applicationinsights-agent-3.0.0-PREVIEW.5.jar).

1. Para habilitar el muestreo, agregue lo siguiente al archivo `ApplicationInsights.json`:

```json
{
  "instrumentationSettings": {
    "preview": {
      "sampling": {
        "fixedRate": {
          "percentage": 10 //this is just an example that shows you how to enable only only 10% of transaction 
        }
      }
    }
  }
}
```

#### <a name="configuring-java-sdk"></a>Configuración del SDK de Java

1. Descargue y configure la aplicación web con la versión más reciente del [SDK de Application Insights para Java](./java-get-started.md).

2. **Habilite el módulo de muestreo de frecuencia fija**; para ello, agregue el siguiente fragmento de código al archivo `ApplicationInsights.xml`:

    ```XML
    <TelemetryProcessors>
        <BuiltInProcessors>
            <Processor type="FixedRateSamplingTelemetryProcessor">
                <!-- Set a percentage close to 100/N where N is an integer. -->
                <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
                <Add name="SamplingPercentage" value="50" />
            </Processor>
        </BuiltInProcessors>
    </TelemetryProcessors>
    ```

3. Puede incluir o excluir determinados tipos de telemetría del muestreo con las siguientes etiquetas dentro del elemento `FixedRateSamplingTelemetryProcessor` de la etiqueta `Processor`:
   
    ```XML
    <ExcludedTypes>
        <ExcludedType>Request</ExcludedType>
    </ExcludedTypes>

    <IncludedTypes>
        <IncludedType>Exception</IncludedType>
    </IncludedTypes>
    ```

Los tipos de telemetría que se pueden incluir o excluir del muestreo son: `Dependency`, `Event`, `Exception`, `PageView`, `Request` y `Trace`.

> [!NOTE]
> Para el porcentaje de muestreo, elija un porcentaje que esté cerca de 100/N, donde N es un número entero.  Actualmente el muestreo no es compatible con otros valores.

### <a name="configuring-fixed-rate-sampling-for-opencensus-python-applications"></a>Configuración del muestreo de frecuencia fija para aplicaciones de Python de OpenCensus

Instrumente la aplicación con los [exportadores de Azure Monitor de OpenCensus](./opencensus-python.md) más recientes.

> [!NOTE]
> El muestreo de frecuencia fija no está disponible para el exportador de métricas. Esto significa que las métricas personalizadas son los únicos tipos de telemetría en los que NO se puede configurar el muestreo. El exportador de métricas enviará toda la telemetría de la que realice el seguimiento.

#### <a name="fixed-rate-sampling-for-tracing"></a>Muestreo de frecuencia fija para seguimiento ####
Puede especificar un elemento `sampler` como parte de la configuración de `Tracer`. Si no se proporciona ningún elemento sampler explícito, se usará de forma predeterminada `ProbabilitySampler`. `ProbabilitySampler` usaría una frecuencia de 1/10 000 de forma predeterminada, lo que significa que se enviará a Application Insights una de cada 10 000 solicitudes. Si desea especificar una frecuencia de muestreo, consulte a continuación.

Para especificar la frecuencia de muestreo, asegúrese de que `Tracer` especifica un elemento sampler con una frecuencia de muestreo entre 0,0 y 1,0, ambos inclusive. Una velocidad de muestreo de 1,0 representa el 100 %, lo que significa que todas las solicitudes se enviarán como datos de telemetría a Application Insights.

```python
tracer = Tracer(
    exporter=AzureExporter(
        instrumentation_key='00000000-0000-0000-0000-000000000000',
    ),
    sampler=ProbabilitySampler(1.0),
)
```

#### <a name="fixed-rate-sampling-for-logs"></a>Muestreo de frecuencia fija para registros ####
Puede configurar el muestreo de frecuencia fija para `AzureLogHandler` mediante la modificación del argumento opcional `logging_sampling_rate`. Si no se proporciona ningún argumento, se usará una frecuencia de muestreo de 1.0. Una velocidad de muestreo de 1,0 representa el 100 %, lo que significa que todas las solicitudes se enviarán como datos de telemetría a Application Insights.

```python
handler = AzureLogHandler(
    instrumentation_key='00000000-0000-0000-0000-000000000000',
    logging_sampling_rate=0.5,
)
```

### <a name="configuring-fixed-rate-sampling-for-web-pages-with-javascript"></a>Configuración del muestreo de frecuencia fija para páginas web con JavaScript

Las páginas web basadas en JavaScript se pueden configurar para usar Application Insights. Los datos de telemetría se envían desde la aplicación cliente que se ejecuta en el explorador del usuario y las páginas se pueden hospedar en cualquier servidor.

Cuando [configure las páginas web basadas en JavaScript para Application Insights](javascript.md), modifique el fragmento de código de JavaScript que obtiene del portal de Application Insights.

> [!TIP]
> En aplicaciones de ASP.NET con JavaScript incluido, el fragmento de código suele ir en `_Layout.cshtml`.

Inserte una línea como `samplingPercentage: 10,` antes de la clave de instrumentación:

```xml
<script>
    var appInsights = // ... 
    ({ 
      // Value must be 100/N where N is an integer.
      // Valid examples: 50, 25, 20, 10, 5, 1, 0.1, ...
      samplingPercentage: 10, 

      instrumentationKey: ...
    }); 

    window.appInsights = appInsights; 
    appInsights.trackPageView(); 
</script>
```

Para el porcentaje de muestreo, elija un porcentaje que esté cerca de 100/N, donde N es un número entero. Actualmente el muestreo no es compatible con otros valores.

#### <a name="coordinating-server-side-and-client-side-sampling"></a>Coordinación del muestreo en el lado servidor y en el cliente

El SDK de JavaScript del cliente participa en el muestreo de frecuencia fija junto con el SDK del lado servidor. Las páginas instrumentadas solo enviarán telemetría del cliente desde el mismo usuario para el que el SDK del servidor decidió incluir en el muestreo. Esta lógica está diseñada para mantener la integridad de la sesión de usuario entre las aplicaciones en el lado cliente y el lado servidor. Como resultado, a partir de cualquier elemento específico de telemetría en Application Insights, puede encontrar todos los demás elementos de telemetría para ese usuario o esa sesión y, en Buscar, puede desplazarse entre las vistas de página y las solicitudes relacionadas.

Si la telemetría del lado cliente y la del lado servidor no muestran muestras coordinadas:

* Compruebe que habilitó el muestreo tanto en el cliente como en el servidor.
* Compruebe que establece el mismo porcentaje de muestreo en el cliente y el servidor.
* Asegúrese de que la versión del SDK es 2.0 o superior.

## <a name="ingestion-sampling"></a>muestreo de ingesta

El muestreo de ingesta funciona en el punto en el que la telemetría del servidor web, los exploradores y los dispositivos alcanzan el punto de conexión de servicio de Application Insights. Aunque no reduce el tráfico de telemetría enviado desde su aplicación, sí reduce la cantidad procesada y retenida (y cobrada) por Application Insights.

Utilice este tipo de muestreo si con frecuencia su aplicación sobrepasa su cuota mensual y no tiene la opción de usar ninguno de los tipos de muestreo basados en el SDK. 

Establecer la frecuencia de muestreo en la página Uso y costos estimados:

![En la hoja Información general de la aplicación, haga clic en Configuración, Cuota, Muestreos y, a continuación, seleccione una frecuencia de muestreo y haga clic en Actualizar.](./media/sampling/data-sampling.png)

Al igual que otros tipos de muestreo, el algoritmo conserva elementos de telemetría relacionados. Por ejemplo, cuando se inspeccione la telemetría en Búsqueda, podrá buscar la solicitud relacionada con una excepción determinada. Los recuentos de métrica, como la tasa de solicitudes y la tasa de excepciones se mantienen correctamente.

Los puntos de datos que se descartan por muestreo no están disponibles en ninguna característica de Application Insights como [exportación continua](./export-telemetry.md).

El muestreo de ingesta no funciona mientras el muestreo adaptable o el de frecuencia fija están en funcionamiento. El muestreo adaptable está habilitado de forma predeterminada cuando se usa el SDK de ASP.NET o el SDK de ASP.NET Core o cuando Application Insights está habilitado en [Azure App Service ](azure-web-apps.md) o mediante el Monitor de estado. Cuando el punto de conexión de servicio de Application Insights recibe la telemetría, examina la telemetría y, si se indica que la frecuencia de muestreo es inferior al 100% (lo que indica que se muestreará la telemetría), se omite la frecuencia del muestreo de ingesta que establezca.

> [!WARNING]
> El valor que se muestra en el icono del portal indica el valor que ha establecido para el muestreo de ingesta. Si un muestreo del SDK está en funcionamiento (muestreo adaptable o de frecuencia fija), no representa la frecuencia real de muestreo.

## <a name="when-to-use-sampling"></a>¿Cuándo usar el muestreo?

En general, para la mayoría de las aplicaciones pequeñas y medianas no es necesario realizar muestreos. La información de diagnóstico más útil y las estadísticas más precisas se obtienen mediante la recopilación de datos en todas las actividades del usuario. 

Las principales ventajas del muestreo son:

* El servicio de Application Insights descarta ("limita") puntos de datos cuando la aplicación envía un número muy elevado de datos de telemetría en un intervalo de tiempo corto. El muestreo reduce la probabilidad de que se produzca una limitación en la aplicación.
* Para mantenerse dentro de la [cuota](pricing.md) de puntos de datos de su plan de tarifa. 
* Para reducir el tráfico de red de la recopilación de telemetría. 

### <a name="which-type-of-sampling-should-i-use"></a>¿Qué tipo de muestreo debo utilizar?

**Use el muestreo de ingesta si:**

* Sobrepasa con frecuencia su cuota mensual de telemetría.
* Recibe muchos datos de telemetría de los exploradores web de los usuarios.
* Usa una versión del SDK que no admite el muestreo; por ejemplo, las versiones de ASP.NET anteriores a la 2.

**Use el muestreo de frecuencia fija si:**

* Quiere un muestreo sincronizado entre cliente y servidor para que, cuando investigue eventos en [Buscar](./diagnostic-search.md), pueda desplazarse entre los eventos relacionados en el cliente y el servidor, como vistas de página y solicitudes HTTP.
* Está seguro del porcentaje de muestreo adecuado para la aplicación. Debe ser lo bastante alto como para obtener métricas precisas, pero inferior a la frecuencia que supera la cuota de precios y los valores de limitación.

**Use el muestreo adaptable si:**

Si no se cumplen las condiciones para usar las otras formas de muestreo, le recomendamos el muestreo adaptable. Esta opción está habilitada de manera predeterminada en el SDK de ASP.NET y ASP.NET Core. No reducirá el tráfico hasta que se alcance un determinado índice mínimo; por lo tanto, los sitios con poco uso probablemente no se muestrearán.

## <a name="knowing-whether-sampling-is-in-operation"></a>¿Cómo se puede saber si el muestreo está en funcionamiento?

Para conocer la frecuencia de muestreo real independientemente de dónde se ha aplicado, use una [consulta de Analytics](../log-query/log-query-overview.md) como esta:

```kusto
union requests,dependencies,pageViews,browserTimings,exceptions,traces
| where timestamp > ago(1d)
| summarize RetainedPercentage = 100/avg(itemCount) by bin(timestamp, 1h), itemType
```

Si ve que `RetainedPercentage` para cualquier tipo es inferior a 100, se muestrea ese tipo de telemetría.

> [!IMPORTANT]
> Application Insights no realiza el muestreo de los contadores de sesión, métricas (incluidas las métricas personalizadas) ni rendimiento de los tipos de telemetría en ninguna de las técnicas de muestreo. Estos tipos siempre se excluyen del muestreo, ya que la reducción en la precisión puede ser altamente indeseable para estos tipos de telemetría.

## <a name="how-sampling-works"></a>Cómo funciona el muestreo

El algoritmo de muestreo decide qué elementos de telemetría se van a eliminar y cuáles conservar. Esto es cierto si el muestreo lo realiza el SDK o el servicio de Application Insights. La decisión del muestreo se basa en varias reglas que tienen como objetivo conservar intactos todos los puntos de datos interrelacionados, manteniendo una experiencia de diagnóstico en Application Insights que sea práctica y confiable, incluso con un conjunto reducido de datos. Por ejemplo, si la aplicación tiene una solicitud con error incluida en una muestra, se conservarán los elementos de telemetría adicionales (como la excepción y los seguimientos registrados para esta solicitud). El muestreo los mantiene o los elimina en conjunto. Como resultado, al examinar los detalles de solicitud en Application Insights, siempre puede ver la solicitud junto con sus elementos de telemetría asociados.

La decisión de muestreo se basa en el identificador de operación de la solicitud, lo que significa que todos los elementos de telemetría que pertenecen a una operación determinada se conservan o se descartan. Para los elementos de telemetría que no tienen establecido un identificador de operación (por ejemplo, los elementos de telemetría notificados desde subprocesos asincrónicos sin contexto HTTP), el muestreo simplemente captura un porcentaje de elementos de telemetría de cada tipo.

Al presentarle la telemetría, el servicio de Application Insights ajusta las métricas con el mismo porcentaje de muestreo que se usó en el momento de la recopilación, para compensar por los puntos de datos que faltan. Por lo tanto, al examinar la telemetría en Application Insights, los usuarios ven aproximaciones estadísticamente correctas que están muy próximas a los números reales.

La precisión de la aproximación depende en gran medida del porcentaje de muestreo configurado. También, la precisión aumenta en las aplicaciones que administran un gran volumen de solicitudes básicamente similares de muchos usuarios. Por otro lado, para las aplicaciones que no funcionan con una carga significativa, el muestreo no es necesario ya que estas aplicaciones normalmente pueden enviar toda su telemetría manteniéndose dentro de la cuota, sin causar pérdida de datos por causa de la limitación. 

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

*¿Cuál es el comportamiento de muestreo predeterminado en los SDK de ASP.NET y ASP.NET Core?*

* Si usa una de las versiones más recientes del SDK anterior, el muestreo adaptable está habilitado de manera predeterminada con cinco elementos de telemetría por segundo.
  Hay dos nodos `AdaptiveSamplingTelemetryProcessor` agregados de forma predeterminada y uno incluye el tipo `Event` en el muestreo, mientras que el otro excluye el tipo `Event` del muestreo. Esta configuración significa que el SDK intentará limitar los elementos de telemetría a cinco elementos de telemetría de tipo `Event` y cinco elementos de telemetría de todos los demás tipos combinados, lo que garantiza que el muestreo de `Events` se realiza por separado de otros tipos de telemetría. Los eventos se suelen usar para la telemetría empresarial y es más probable que no deban verse afectados por los volúmenes de telemetría de diagnóstico.
  
  A continuación se muestra el archivo `ApplicationInsights.config` predeterminado generado. En ASP.NET Core, el mismo comportamiento predeterminado se habilita en el código. Use los [ejemplos de la sección anterior de esta página](#configuring-adaptive-sampling-for-aspnet-core-applications) para cambiar este comportamiento predeterminado.

    ```xml
    <TelemetryProcessors>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
            <ExcludedTypes>Event</ExcludedTypes>
        </Add>
        <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
            <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
            <IncludedTypes>Event</IncludedTypes>
        </Add>
    </TelemetryProcessors>
    ```

*¿La telemetría se puede muestrear más de una vez?*

* No. Los elementos SamplingTelemetryProcessors omiten elementos de las consideraciones de muestreo si el elemento ya se ha muestreado. Lo mismo sucede para el muestreo de ingesta, que no aplicará el muestreo a los elementos ya muestreados en el propio SDK.

*¿Por qué realizar un muestreo no consiste simplemente en "recopilar X por ciento de cada tipo de telemetría"?*

* Aunque este método de muestreo proporcionaría un nivel de precisión alto en las aproximaciones de métricas, destruiría la capacidad de poner en correlación datos de diagnóstico por usuario, sesión y solicitud, lo cual es crítico para el diagnóstico. Por lo tanto, el muestreo funciona mejor con directivas como "recopilar todos los elementos de telemetría para el X por ciento de los usuarios de la aplicación" o "recopilar toda la telemetría para el X por ciento de las solicitudes de la aplicación". Para los elementos de telemetría no asociados con las solicitudes (por ejemplo, el procesamiento asincrónico en segundo plano), la acción de reserva es "recopilar X por ciento de todos los elementos para cada tipo de telemetría". 

*¿Se puede cambiar el porcentaje de muestreo con el tiempo?*

* Sí, el muestreo adaptable cambia gradualmente el porcentaje de muestreo, en función del volumen de datos de telemetría observado en ese momento.

*Si uso el muestreo de frecuencia fija, ¿cómo sé cuál es el porcentaje de muestreo que funcionará mejor para mi aplicación?*

* Una manera de comenzar es con muestreo adaptable, averigüe qué velocidad elige (consulte la pregunta anterior) y luego cambie a muestreo de tasa fija con esa velocidad. 
  
    Si no, lo tendrá que adivinar. Analice su uso actual de telemetría en Application Insights, observe las limitaciones que se produzcan y estime el volumen de telemetría recopilado. Estas tres entradas, junto con el plan de tarifa seleccionado, le sugerirán cuánto debería reducir el volumen de telemetría recopilado. Sin embargo, un aumento del número de usuarios o algún otro incremento en el volumen de datos de telemetría podrían provocar que la estimación no fuese válida.

*¿Qué sucede si configuro un porcentaje de muestreo demasiado bajo?*

* Un porcentaje de muestreo excesivamente bajo provoca un muestreo excesivo y reduce la precisión de las aproximaciones cuando Application Insights intenta compensar la visualización de los datos por la reducción del volumen de datos. Además, la experiencia de diagnóstico puede verse afectada negativamente, ya que algunas de las solicitudes con poca frecuencia errores o lentas pueden quedar fuera del muestreo.

*¿Qué sucede si configuro un porcentaje de muestreo demasiado alto?*

* Configurar un porcentaje de muestreo demasiado alto (no suficientemente drástico) da como resultado una reducción insuficiente del volumen de telemetría recopilada. Se pueden seguir produciendo pérdidas de datos de telemetría relacionadas con la limitación, y el costo de usar Application Insights puede ser mayor de lo planeado, debido a los cargos debidos al uso por encima del límite.

*¿En qué plataformas puedo usar muestreo?*

* El muestreo de ingesta puede producirse automáticamente cuando cualquier dato de telemetría se sitúa por encima de un determinado volumen, si el SDK no realiza muestreo. Esta configuración funcionaría, por ejemplo, si usa una versión antigua del SDK de ASP.NET o del SDK de Java.
* Si usa los SDK de ASP.NET o ASP.NET Core actuales (hospedado en Azure o en su propio servidor), obtendrá de forma predeterminada el muestreo adaptable, pero puede cambiar al de frecuencia fija como se ha descrito anteriormente. Con el muestreo de frecuencia fija, el SDK del explorador se sincroniza automáticamente con los eventos relacionados con el muestreo. 
* Si usa el agente de Java actual, puede configurar `ApplicationInsights.json` (para el SDK de Java, configure `ApplicationInsights.xml`) para activar el muestreo de frecuencia fija. De manera predeterminada, el muestreo está desactivado. Con el muestreo de frecuencia fija, el SDK del explorador y el servidor se sincronizan automáticamente para muestrear los eventos relacionados.

*Hay ciertos eventos excepcionales que siempre quiero ver. ¿Cómo se consigue que el módulo de muestreo los reconozca?*

* La mejor forma de lograrlo es escribir un objeto [TelemetryInitializer](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) personalizado, que establece `SamplingPercentage` en 100 en el elemento de telemetría que desee retener, tal como se muestra a continuación. Cuando se garantiza que los inicializadores se ejecutan antes que los procesadores de telemetría (incluido el muestreo), se tiene la seguridad de que todas las técnicas de muestreo omitirán este elemento de cualquier consideración de muestreo. Se dispone de inicializadores de telemetría personalizados en el SDK de ASP.NET, el SDK de ASP.NET Core, el SDK de JavaScript y el SDK de Java. Por ejemplo, puede configurar un inicializador de telemetría mediante el SDK de ASP.NET:

    ```csharp
    public class MyTelemetryInitializer : ITelemetryInitializer
    {
        public void Initialize(ITelemetry telemetry)
        {
            if(somecondition)
            {
                ((ISupportSampling)telemetry).SamplingPercentage = 100;
            }
        }
    }
    ```

## <a name="older-sdk-versions"></a>Versiones anteriores de SDK

El muestreo adaptable está disponible para el SDK de Application Insights para ASP.NET v2.0.0-beta3 y versiones posteriores y el SDK de Microsoft.ApplicationInsights.AspNetCore v2.2.0-beta1 y versiones posteriores y está habilitado de manera predeterminada.

El muestreo de frecuencia fija es una característica del SDK de ASP.NET de la versión 2.0.0 y posteriores y del SDK de Java de la versión 2.0.1 y posteriores.

Antes de la versión v2.5.0-beta2 del SDK de ASP.NET y de la versión v2.2.0-beta3 del SDK de ASP.NET Core, la decisión de muestreo se basaba en el hash del identificador de usuario de las aplicaciones que definen al "usuario" (es decir, las aplicaciones web más habituales). Para los tipos de aplicaciones que no definen a los usuarios (como los servicios web), la decisión de muestreo se basaba en el identificador de operación de la solicitud. Las versiones recientes de los SDK de ASP.NET y ASP.NET Core usan el identificador de operación para la decisión de muestreo.

## <a name="next-steps"></a>Pasos siguientes

* [filtro](./api-filtering-sampling.md) puede proporcionar un control más estricto de los SDK que envía.
* Lea el artículo de Developer Network [Optimize Telemetry with Application Insights](/archive/msdn-magazine/2017/may/devops-optimize-telemetry-with-application-insights) (Optimización de la telemetría con Application Insights).

