---
title: 'Referencia de ApplicationInsights.config: Azure | Microsoft Docs'
description: Habilitación o deshabilitación de los módulos de recopilación de datos e incorporación de contadores de rendimiento y otros parámetros.
ms.topic: conceptual
ms.date: 05/22/2019
ms.reviewer: olegan
ms.openlocfilehash: ec446190cd589eb511a7a905faeb5f29f31e7d69
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87310487"
---
# <a name="configuring-the-application-insights-sdk-with-applicationinsightsconfig-or-xml"></a>Configuración del SDK de Application Insights con ApplicationInsights.config o .xml
El SDK de Application Insights para .NET consta de varios paquetes de NuGet. El [paquete principal](https://www.nuget.org/packages/Microsoft.ApplicationInsights) proporciona la API para enviar telemetría a Application Insights. Los [paquetes adicionales](https://www.nuget.org/packages?q=Microsoft.ApplicationInsights) proporcionan *módulos* e *inicializadores* de telemetría para hacer un seguimiento automático de la aplicación y su contexto. Si ajusta el archivo de configuración, puede habilitar o deshabilitar los módulos e inicializadores de telemetría, y establecer los parámetros para algunos de ellos.

El archivo de configuración se denomina `ApplicationInsights.config` o `ApplicationInsights.xml`, dependiendo del tipo de la aplicación. Se agrega automáticamente al proyecto cuando se [instalan la mayoría de las versiones del SDK][start]. De forma predeterminada, cuando se usa la experiencia automatizada de los proyectos de plantilla de Visual Studio que admiten **Agregar > Telemetría de Application Insights**, se crea el archivo ApplicationInsights.config en la carpeta raíz del proyecto y, cuando es compatible, se copia en la carpeta bin. El [Monitor de estado en un servidor IIS][redfield] también lo agrega a una aplicación web. El archivo de configuración se omite si se usa la [extensión para el sitio web de Azure](azure-web-apps.md) o la [extensión para VM de Azure y conjuntos de escalado de máquinas virtuales](azure-vm-vmss-apps.md).

No hay un archivo equivalente para controlar el [SDK en una página web][client].

En este documento se describen las secciones que verá en el archivo de configuración, cómo controlan los componentes del SDK y qué paquetes NuGet cargan esos componentes.

> [!NOTE]
> Las instrucciones de ApplicationInsights.config y .xml no se aplican al SDK de .NET Core. Para configurar aplicaciones de .NET Core, siga [esta](./asp-net-core.md) guía.

## <a name="telemetry-modules-aspnet"></a>Módulos de telemetría (ASP.NET)
Cada módulo de telemetría recopila un tipo específico de datos y usa la API principal para enviar dichos datos. Los módulos los instalan diferentes paquetes NuGet, que también agregan las líneas necesarias al archivo .config.

Hay un nodo en el archivo de configuración para cada módulo. Para deshabilitar un módulo, elimine el nodo o conviértalo en comentario.

### <a name="dependency-tracking"></a>Seguimiento de dependencia
[Dependency tracking](./asp-net-dependencies.md) recopila la telemetría sobre las llamadas que realiza la aplicación a bases de datos y a servicios y bases de datos externos. Para permitir que este módulo funcione en un servidor IIS, deberá [instalar el Monitor de estado][redfield].

También puede escribir su propio código de seguimiento de dependencias con la [API de TrackDependency](./api-custom-events-metrics.md#trackdependency).

* `Microsoft.ApplicationInsights.DependencyCollector.DependencyTrackingTelemetryModule`
* [Microsoft.ApplicationInsights.DependencyCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector) .

Las dependencias se pueden recopilar automáticamente sin modificar el código mediante la asociación basada en agente (sin código). Para utilizarlo en aplicaciones web de Azure, habilite la [extensión Application Insights](azure-web-apps.md). Para usarlo en una VM de Azure o en un conjunto de escalado de máquinas virtuales de Azure, habilite la [extensión de supervisión de aplicaciones para VM y conjuntos de escalado de máquinas virtuales](azure-vm-vmss-apps.md).

### <a name="performance-collector"></a>Recopilador de rendimiento
[Recopila contadores de rendimiento del sistema](./performance-counters.md) como CPU, memoria y carga de la red desde instalaciones de IIS. Puede especificar qué contadores recopilar, incluidos los contadores de rendimiento que ha configurado usted mismo.

* `Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.PerformanceCollectorModule`
* [Microsoft.ApplicationInsights.PerfCounterCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector) .

### <a name="application-insights-diagnostics-telemetry"></a>Telemetría de diagnósticos de Application Insights
`DiagnosticsTelemetryModule` informa de errores en el propio código de instrumentación de Application Insights. Por ejemplo, si el código no puede tener acceso a los contadores de rendimiento o si un `ITelemetryInitializer` inicia una excepción. La telemetría de seguimiento que sigue este módulo aparece en la [Búsqueda de diagnóstico][diagnostic].

```
* `Microsoft.ApplicationInsights.Extensibility.Implementation.Tracing.DiagnosticsTelemetryModule`
* [Microsoft.ApplicationInsights](https://www.nuget.org/packages/Microsoft.ApplicationInsights) NuGet package. If you only install this package, the ApplicationInsights.config file is not automatically created.
```

### <a name="developer-mode"></a>Modo de programador
`DeveloperModeWithDebuggerAttachedTelemetryModule` fuerza al `TelemetryChannel` de Application Insights a enviar los datos inmediatamente, elemento de telemetría a elemento, cuando se adjunta un depurador al proceso de aplicación. Esto reduce la cantidad de tiempo entre el momento en que la aplicación realiza un seguimiento de telemetría y el momento en que esta aparece en el portal de Application Insights. Esto provoca una sobrecarga considerable en la CPU y el ancho de banda.

* `Microsoft.ApplicationInsights.WindowsServer.DeveloperModeWithDebuggerAttachedTelemetryModule`
* [Application Insights Windows Server](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer/)

### <a name="web-request-tracking"></a>Seguimiento de solicitud web
Informa del [tiempo de respuesta y del código del resultado](../../azure-monitor/app/asp-net.md) de solicitudes HTTP.

* `Microsoft.ApplicationInsights.Web.RequestTrackingTelemetryModule`
* [Microsoft.ApplicationInsights.Web](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web)

### <a name="exception-tracking"></a>Seguimiento de excepciones
`ExceptionTrackingTelemetryModule` realiza excepciones no controladas en la aplicación web. Consulte [Errores y excepciones][exceptions].

* `Microsoft.ApplicationInsights.Web.ExceptionTrackingTelemetryModule`
* [Microsoft.ApplicationInsights.Web](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web)
* `Microsoft.ApplicationInsights.WindowsServer.UnobservedExceptionTelemetryModule` - Realiza un seguimiento de excepciones de tareas inadvertidas.
* `Microsoft.ApplicationInsights.WindowsServer.UnhandledExceptionTelemetryModule` - Realiza un seguimiento de excepciones no controladas para roles de trabajo, servicios de Windows y aplicaciones de consola.
* [Application Insights Windows Server](https://www.nuget.org/packages/Microsoft.ApplicationInsights.WindowsServer/) .

### <a name="eventsource-tracking"></a>Seguimiento de EventSource
`EventSourceTelemetryModule` permite configurar eventos EventSource para enviarse a Application Insights como seguimientos. Para obtener información sobre el seguimiento de eventos EventSource, vea [Uso de eventos EventSource](./asp-net-trace-logs.md#use-eventsource-events).

* `Microsoft.ApplicationInsights.EventSourceListener.EventSourceTelemetryModule`
* [Microsoft.ApplicationInsights.EventSourceListener](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventSourceListener) 

### <a name="etw-event-tracking"></a>Seguimiento de eventos ETW
`EtwCollectorTelemetryModule` permite configurar eventos de proveedores ETW para enviarse a Application Insights como seguimientos. Para obtener información sobre el seguimiento de eventos ETW de seguimiento, vea [Uso de eventos ETW](../../azure-monitor/app/asp-net-trace-logs.md#use-etw-events).

* `Microsoft.ApplicationInsights.EtwCollector.EtwCollectorTelemetryModule`
* [Microsoft.ApplicationInsights.EtwCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EtwCollector) 

### <a name="microsoftapplicationinsights"></a>Microsoft.ApplicationInsights
El paquete Microsoft.ApplicationInsights proporciona la [API principal](/dotnet/api/microsoft.applicationinsights?view=azure-dotnet) del SDK. Los otros módulos de telemetría usan esto y también puede [usarlo usted mismo para definir su propia telemetría](./api-custom-events-metrics.md).

* No hay entrada en ApplicationInsights.config.
* [Microsoft.ApplicationInsights](https://www.nuget.org/packages/Microsoft.ApplicationInsights) . Si solamente instala este NuGet, no se genera ningún archivo .config.

## <a name="telemetry-channel"></a>Canal de telemetría
El [canal de telemetría](telemetry-channels.md) administra el almacenamiento en búfer y la transmisión de telemetría al servicio Application Insights.

* `Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.ServerTelemetryChannel` es el canal predeterminado para aplicaciones web. Almacena en búfer los datos en memoria y emplea mecanismos de reintento y almacenamiento de disco local para proporcionar una entrega de telemetría de más confianza.
* `Microsoft.ApplicationInsights.InMemoryChannel` es un canal de telemetría ligero que se utiliza si no se ha configurado ningún otro canal. 

## <a name="telemetry-initializers-aspnet"></a>Inicializadores de telemetría (ASP.NET)
Los inicializadores de telemetría establecen propiedades de contexto que se envían junto con todos los elementos de telemetría.

También puede [escribir sus propios inicializadores](./api-filtering-sampling.md#add-properties) para establecer propiedades de contexto.

Los inicializadores estándar están todos establecidos por los paquetes NuGet web o  WindowsServer:

* `AccountIdTelemetryInitializer` establece la propiedad AccountId.
* `AuthenticatedUserIdTelemetryInitializer` establece la propiedad AuthenticatedUserId establecida por el SDK de JavaScript.
* `AzureRoleEnvironmentTelemetryInitializer` actualiza las propiedades `RoleName` y `RoleInstance` del contexto `Device` para todos los elementos de telemetría con información extraída del entorno de tiempo de ejecución de Azure.
* `BuildInfoConfigComponentVersionTelemetryInitializer` actualiza la propiedad `Version` del contexto `Component` para todos los elementos de telemetría con el valor extraído del archivo `BuildInfo.config` que produce MS Build.
* `ClientIpHeaderTelemetryInitializer` actualiza la propiedad `Ip` del contexto `Location` de todos los elementos de telemetría según el encabezado HTTP `X-Forwarded-For` de la solicitud.
* `DeviceTelemetryInitializer` actualiza las propiedades siguientes del contexto `Device` para todos los elementos de telemetría.
  * `Type` se establece en "PC".
  * `Id` se establece en el nombre de dominio del equipo donde se ejecuta la aplicación web.
  * `OemName` se establece en el valor extraído del campo `Win32_ComputerSystem.Manufacturer` con WMI.
  * `Model` se establece en el valor extraído del campo `Win32_ComputerSystem.Model` con WMI.
  * `NetworkType` se establece en el valor extraído de `NetworkInterface`.
  * `Language` se establece en el nombre de `CurrentCulture`.
* `DomainNameRoleInstanceTelemetryInitializer` actualiza la propiedad `RoleInstance` del contexto `Device` para todos los elementos de telemetría con el nombre de dominio del equipo donde se ejecuta la aplicación web.
* `OperationNameTelemetryInitializer` actualiza la propiedad `Name` de la propiedad `RequestTelemetry` y `Name` propiedad del contexto `Operation` de todos los elementos de telemetría según el método HTTP, así como los nombres del controlador MVC de ASP.NET y la acción que se invoca para procesar la solicitud.
* `OperationIdTelemetryInitializer` o `OperationCorrelationTelemetryInitializer` actualizan la propiedad de contexto `Operation.Id` de todos los elementos de telemetría de los que se realiza un seguimiento mientras se controla una solicitud con el `RequestTelemetry.Id` que se genera.
* `SessionTelemetryInitializer` actualiza la propiedad `Id` del contexto `Session` para todos los elementos de telemetría con valor extraído de la cookie `ai_session` que genera el código de instrumentación JavaScript de Application Insights que se ejecuta en el explorador del usuario.
* `SyntheticTelemetryInitializer` o `SyntheticUserAgentTelemetryInitializer` actualiza las propiedades de los contextos `User`, `Session` y `Operation` de todos los elementos de telemetría de los que se realiza el seguimiento al controlar una solicitud de un origen sintético, como una prueba de disponibilidad o un bot de motor de búsqueda. De forma predeterminada, [Explorador de métricas](../platform/metrics-charts.md) no muestra telemetría sintética.

    Conjunto de `<Filters>` que identifica las propiedades de las solicitudes.
* `UserTelemetryInitializer` actualiza las propiedades `Id` y `AcquisitionDate` del contexto `User` para todos los elementos de telemetría con valores extraídos de la cookie `ai_user` que genera el código de instrumentación JavaScript de Application Insights que se ejecuta en el explorador del usuario.
* `WebTestTelemetryInitializer` establece el identificador de usuario, el identificador de sesión y las propiedades de origen sintético de las solicitudes HTTP que proceden de [pruebas de disponibilidad](./monitor-web-app-availability.md).
  Conjunto de `<Filters>` que identifica las propiedades de las solicitudes.

Para aplicaciones de .NET que se ejecutan en Service Fabric, puede incluir el paquete de NuGet `Microsoft.ApplicationInsights.ServiceFabric`. Este paquete incluye `FabricTelemetryInitializer`, que agrega propiedades de Service Fabric a elementos de telemetría. Para obtener más información, consulte la [página de GitHub](https://github.com/Microsoft/ApplicationInsights-ServiceFabric/blob/master/README.md) sobre las propiedades que agrega este paquete de NuGet.

## <a name="telemetry-processors-aspnet"></a>Procesadores de telemetría (ASP.NET)
Los procesadores de telemetría pueden filtrar y modificar cada elemento de telemetría justo antes de que se envíe desde el SDK al portal.

También puede [escribir sus propios procesadores de telemetría](./api-filtering-sampling.md#filtering).

#### <a name="adaptive-sampling-telemetry-processor-from-200-beta3"></a>Procesador de telemetría de muestreo adaptable (desde 2.0.0-beta3)
Esta opción está habilitada de manera predeterminada. Si la aplicación envía una gran cantidad de datos de telemetría, este procesador quita algunos de ellos.

```xml

    <TelemetryProcessors>
      <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.AdaptiveSamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">
        <MaxTelemetryItemsPerSecond>5</MaxTelemetryItemsPerSecond>
      </Add>
    </TelemetryProcessors>

```

El parámetro proporciona el destino que el algoritmo intenta alcanzar. Cada instancia del SDK funciona de forma independiente, por lo que si el servidor es un clúster de varios equipos, se multiplicará el volumen real de telemetría en consonancia.

[Obtenga más información sobre el muestreo](./sampling.md).

#### <a name="fixed-rate-sampling-telemetry-processor-from-200-beta1"></a>Procesador de telemetría de muestreo de tasa fija (desde 2.0.0-beta1)
También hay un [procesador de telemetría de muestreo](./api-filtering-sampling.md) estándar (desde 2.0.1):

```XML

    <TelemetryProcessors>
     <Add Type="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel.SamplingTelemetryProcessor, Microsoft.AI.ServerTelemetryChannel">

     <!-- Set a percentage close to 100/N where N is an integer. -->
     <!-- E.g. 50 (=100/2), 33.33 (=100/3), 25 (=100/4), 20, 1 (=100/100), 0.1 (=100/1000) -->
     <SamplingPercentage>10</SamplingPercentage>
     </Add>
   </TelemetryProcessors>

```

## <a name="instrumentationkey"></a>InstrumentationKey
Esto determina el recurso de Application Insights en el que aparecen los datos. Normalmente se crea un recurso independiente, con una clave independiente, para cada una de las aplicaciones.

Si desea establecer la clave de forma dinámica (por ejemplo, si desea enviar los resultados de su aplicación a distintos recursos) puede omitir la clave del archivo de configuración y establecerla en el código.

Para establecer la clave de todas las instancias de TelemetryClient, incluidos los módulos de telemetría estándar. Hágalo en un método de inicialización, como global.aspx.cs, en un servicio de ASP.NET:

```csharp
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights;

    protected void Application_Start()
    {
        TelemetryConfiguration configuration = TelemetryConfiguration.CreateDefault();
        configuration.InstrumentationKey = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
        var telemetryClient = new TelemetryClient(configuration);
   
```

Si solo desea enviar un conjunto específico de eventos a un recurso diferente, puede establecer la clave para un TelemetryClient específico:

```csharp

    var tc = new TelemetryClient();
    tc.Context.InstrumentationKey = "----- my key ----";
    tc.TrackEvent("myEvent");
    // ...

```

Para obtener una nueva clave, [cree un nuevo recurso en el portal de Application Insights][new].



## <a name="applicationid-provider"></a>Proveedor ApplicationId

_Disponible a partir de la versión 2.6.0_

El propósito de este proveedor es buscar un identificador de aplicación en función de una clave de instrumentación. El identificador de aplicación se incluye en RequestTelemetry y DependencyTelemetry, y se usa para determinar la correlación en el portal.

Está disponible al establecer `TelemetryConfiguration.ApplicationIdProvider` en el código o en la configuración.

### <a name="interface-iapplicationidprovider"></a>Interfaz: IApplicationIdProvider

```csharp
public interface IApplicationIdProvider
{
    bool TryGetApplicationId(string instrumentationKey, out string applicationId);
}
```


Se proporcionan dos implementaciones en el SDK [Microsoft.ApplicationInsights](https://www.nuget.org/packages/Microsoft.ApplicationInsights): `ApplicationInsightsApplicationIdProvider` y `DictionaryApplicationIdProvider`.

### <a name="applicationinsightsapplicationidprovider"></a>ApplicationInsightsApplicationIdProvider

Es un contenedor alrededor de nuestra API de Profile. Limita las solicitudes y los resultados en la memoria caché.

Este proveedor se agrega al archivo de configuración cuando se instala [Microsoft.ApplicationInsights.DependencyCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DependencyCollector) o [Microsoft.ApplicationInsights.Web](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Web/)

Esta clase tiene una propiedad `ProfileQueryEndpoint` opcional.
De manera predeterminada, se establece en `https://dc.services.visualstudio.com/api/profiles/{0}/appId`.
Si tiene que configurar un proxy para esta configuración, se recomienda un proxy en la dirección base y la inclusión de "/api/profiles/{0}/appId". Tenga en cuenta que "{0}" se sustituye en tiempo de ejecución por cada solicitud con la clave de instrumentación.

#### <a name="example-configuration-via-applicationinsightsconfig"></a>Ejemplo de configuración con ApplicationInsights.config:
```xml
<ApplicationInsights>
    ...
    <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights">
        <ProfileQueryEndpoint>https://dc.services.visualstudio.com/api/profiles/{0}/appId</ProfileQueryEndpoint>
    </ApplicationIdProvider>
    ...
</ApplicationInsights>
```

#### <a name="example-configuration-via-code"></a>Ejemplo de configuración mediante código:
```csharp
TelemetryConfiguration.Active.ApplicationIdProvider = new ApplicationInsightsApplicationIdProvider();
```

### <a name="dictionaryapplicationidprovider"></a>DictionaryApplicationIdProvider

Se trata de un proveedor estático, que se basará en los pares de clave de instrumentación/identificador de la aplicación configurados.

Esta clase tiene una propiedad `Defined`, que es un Diccionario<string,string> de los pares de clave de instrumentación e identificador de aplicación.

Esta clase tiene una propiedad `Next` opcional que puede utilizarse para configurar otro proveedor que se use al solicitar una clave de instrumentación que no exista en la configuración.

#### <a name="example-configuration-via-applicationinsightsconfig"></a>Ejemplo de configuración con ApplicationInsights.config:
```xml
<ApplicationInsights>
    ...
    <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.DictionaryApplicationIdProvider, Microsoft.ApplicationInsights">
        <Defined>
            <Type key="InstrumentationKey_1" value="ApplicationId_1"/>
            <Type key="InstrumentationKey_2" value="ApplicationId_2"/>
        </Defined>
        <Next Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights" />
    </ApplicationIdProvider>
    ...
</ApplicationInsights>
```

#### <a name="example-configuration-via-code"></a>Ejemplo de configuración mediante código:
```csharp
TelemetryConfiguration.Active.ApplicationIdProvider = new DictionaryApplicationIdProvider{
 Defined = new Dictionary<string, string>
    {
        {"InstrumentationKey_1", "ApplicationId_1"},
        {"InstrumentationKey_2", "ApplicationId_2"}
    }
};
```




## <a name="next-steps"></a>Pasos siguientes
[Más información acerca de la API][api].

<!--Link references-->

[api]: ./api-custom-events-metrics.md
[client]: ./javascript.md
[diagnostic]: ./diagnostic-search.md
[exceptions]: ./asp-net-exceptions.md
[netlogs]: ./asp-net-trace-logs.md
[new]: ./create-new-resource.md
[redfield]: ./monitor-performance-live-website-now.md
[start]: ./app-insights-overview.md

