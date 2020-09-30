---
title: Exploración de registros de seguimiento de .NET en Application Insights
description: Busque registros generados por Seguimiento, NLog o Log4Net.
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 05/08/2019
ms.openlocfilehash: ab3b12bf0401c4060823c6ed1d20dd6385cc397f
ms.sourcegitcommit: bdd5c76457b0f0504f4f679a316b959dcfabf1ef
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90973841"
---
# <a name="explore-netnet-core-and-python-trace-logs-in-application-insights"></a>Exploración de los registros de seguimiento de .NET, .NET Core y Python en Application Insights

Envíe registros de seguimiento de diagnóstico para la aplicación ASP.NET/ASP.NET Core desde ILogger, NLog, log4Net o System.Diagnostics.Trace a [Azure Application Insights][start]. En el caso de las aplicaciones de Python, envíe registros de seguimiento de diagnóstico mediante AzureLogHandler en OpenCensus para Python para Azure Monitor. Después, puede explorarlos y buscar en ellos. Estos registros se combinan con los otros archivos de registro procedentes de su aplicación, de modo que pueda identificar los seguimientos asociados con cada solicitud de usuario y correlacionarlos con otros eventos e informes de excepciones.

> [!NOTE]
> ¿Necesita el módulo de captura de registros? Es un adaptador útil para registradores de terceros, pero si aún no usa NLog, log4Net o System.Diagnostics.Trace, considere la posibilidad de llamar directamente a [**Application Insights TrackTrace()** ](./api-custom-events-metrics.md#tracktrace).
>
>
## <a name="install-logging-on-your-app"></a>Instalación del registro en la aplicación
Instale el marco de registro elegido en el proyecto, lo que debería producir una entrada en app.config o web.config.

```XML
 <configuration>
  <system.diagnostics>
    <trace>
      <listeners>
        <add name="myAppInsightsListener" type="Microsoft.ApplicationInsights.TraceListener.ApplicationInsightsTraceListener, Microsoft.ApplicationInsights.TraceListener" />
      </listeners>
    </trace>
  </system.diagnostics>
</configuration>
```

## <a name="configure-application-insights-to-collect-logs"></a>Configuración de Application Insights para recopilar registros
Si todavía no lo ha hecho, [agregue Application Insights a su proyecto](./asp-net.md). Verá una opción para incluir el compilador de registros.

También puede hacer clic con el botón derecho en el proyecto en el Explorador de soluciones para **configurar Application Insights**. Seleccione la opción **Configure trace collection** (Configurar recolección de seguimientos).

> [!NOTE]
> ¿No aparece ningún menú de Application Insights ni ninguna opción de recopilador de registros? Pruebe la [solución de problemas](#troubleshooting).

## <a name="manual-installation"></a>Instalación manual
Utilice este método si el tipo de proyecto no es compatible con el programa de instalación de Application Insights (por ejemplo, un proyecto de escritorio de Windows).

1. Si planea usar log4Net o NLog, instálelo en su proyecto.
2. En el Explorador de soluciones, haga clic con el botón derecho en el proyecto y seleccione **Administrar paquetes NuGet**.
3. Busque "Application Insights".
4. Seleccione uno de los siguientes paquetes:

   - Para ILogger: [Microsoft.Extensions.Logging.ApplicationInsights](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights/)
[![Banner de iLogger de NuGet](https://img.shields.io/nuget/vpre/Microsoft.Extensions.Logging.ApplicationInsights.svg)](https://www.nuget.org/packages/Microsoft.Extensions.Logging.ApplicationInsights/)
   - Para NLog: [Microsoft.ApplicationInsights.NLogTarget](https://www.nuget.org/packages/Microsoft.ApplicationInsights.NLogTarget/)
[![Banner para NLog de NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.NLogTarget.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.NLogTarget/)
   - Para Log4Net: [Microsoft.ApplicationInsights.Log4NetAppender](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Log4NetAppender/)
[![Banner para Log4Net de NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.Log4NetAppender.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.Log4NetAppender/)
   - Para System.Diagnostics: [Microsoft.ApplicationInsights.TraceListener](https://www.nuget.org/packages/Microsoft.ApplicationInsights.TraceListener/)
[![Banner de System.Diagnostics de NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.TraceListener.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.TraceListener/)
   - [Microsoft.ApplicationInsights.DiagnosticSourceListener](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener/)
[![Banner de la escucha de origen de diagnósticos de NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.DiagnosticSourceListener.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener/)
   - [Microsoft.ApplicationInsights.EtwCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EtwCollector/)
[![Banner del recopilador de Etw de NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.EtwCollector.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EtwCollector/)
   - [Microsoft.ApplicationInsights.DiagnosticSourceListener](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventSourceListener/)
[![Banner de la escucha de origen de eventos de NuGet](https://img.shields.io/nuget/vpre/Microsoft.ApplicationInsights.EventSourceListener.svg)](https://www.nuget.org/packages/Microsoft.ApplicationInsights.EventSourceListener/)

El paquete NuGet instala los ensamblados necesarios y, si es aplicable, modifica el archivo web.config o app.config.

## <a name="ilogger"></a>ILogger

Para ver algunos ejemplos de uso de la implementación de ILogger para Application Insights con aplicaciones de consola y de ASP.NET Core, vea [ApplicationInsightsLoggerProvider para registros ILogger de .NET Core](ilogger.md).

## <a name="insert-diagnostic-log-calls"></a>Insertar llamadas de registro de diagnóstico
Si usa System.Diagnostics.Trace, una llamada típica sería:

```csharp
System.Diagnostics.Trace.TraceWarning("Slow response - database01");
```

Si prefiere log4net o NLog, use lo siguiente:

```csharp
    logger.Warn("Slow response - database01");
```

## <a name="use-eventsource-events"></a>Uso de eventos EventSource
Puede configurar eventos [System.Diagnostics.Tracing.EventSource](/dotnet/api/system.diagnostics.tracing.eventsource?view=netcore-3.1) para enviarse a Application Insights como seguimientos. Primero, instale el paquete de NuGet `Microsoft.ApplicationInsights.EventSourceListener`. Seguidamente, edite la sección `TelemetryModules` del archivo [ApplicationInsights.config](./configuration-with-applicationinsights-config.md).

```xml
    <Add Type="Microsoft.ApplicationInsights.EventSourceListener.EventSourceTelemetryModule, Microsoft.ApplicationInsights.EventSourceListener">
      <Sources>
        <Add Name="MyCompany" Level="Verbose" />
      </Sources>
    </Add>
```

En cada origen, puede establecer los siguientes parámetros:
 * **Name** especifica el nombre del evento EventSource que se recopilará.
 * **Level** especifica el nivel de registro que se recopilará: *Critical*, *Error*, *Informational*, *LogAlways*, *Verbose* o *Warning*.
 * **Keywords** (opcional) especifica el valor del entero de combinaciones de palabras clave que se usará.

## <a name="use-diagnosticsource-events"></a>Uso de eventos DiagnosticSource
Puede configurar eventos [System.Diagnostics.DiagnosticSource](https://github.com/dotnet/corefx/blob/master/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md) para enviarlos a Application Insights como seguimientos. Primero, instale el paquete de NuGet [`Microsoft.ApplicationInsights.DiagnosticSourceListener`](https://www.nuget.org/packages/Microsoft.ApplicationInsights.DiagnosticSourceListener). Después, edite la sección "TelemetryModules" del archivo [ApplicationInsights.config](./configuration-with-applicationinsights-config.md).

```xml
    <Add Type="Microsoft.ApplicationInsights.DiagnosticSourceListener.DiagnosticSourceTelemetryModule, Microsoft.ApplicationInsights.DiagnosticSourceListener">
      <Sources>
        <Add Name="MyDiagnosticSourceName" />
      </Sources>
    </Add>
```

Para cada instancia de DiagnosticSource que quiera seguir, agregue una entrada con el atributo **Name** establecido con el nombre de la instancia DiagnosticSource.

## <a name="use-etw-events"></a>Uso de eventos ETW
Puede configurar eventos de Seguimiento de eventos para Windows (ETW) para enviarse a Application Insights como seguimientos. Primero, instale el paquete de NuGet `Microsoft.ApplicationInsights.EtwCollector`. Después, edite la sección "TelemetryModules" del archivo [ApplicationInsights.config](./configuration-with-applicationinsights-config.md).

> [!NOTE] 
> Solo se pueden recopilar eventos ETW si el proceso que hospeda el SDK se ejecuta bajo una identidad que sea miembro de Usuarios del registro de rendimiento o Administradores.

```xml
    <Add Type="Microsoft.ApplicationInsights.EtwCollector.EtwCollectorTelemetryModule, Microsoft.ApplicationInsights.EtwCollector">
      <Sources>
        <Add ProviderName="MyCompanyEventSourceName" Level="Verbose" />
      </Sources>
    </Add>
```

En cada origen, puede establecer los siguientes parámetros:
 * **ProviderName** es el nombre del proveedor ETW que se recopilará.
 * **ProviderGuid** especifica el GUID del proveedor ETW que se recopilará. Se puede usar en lugar de `ProviderName`.
 * **Level** establece el nivel de registro que se recopilará. Puede ser *Critical*, *Error*, *Informational*, *LogAlways*, *Verbose* o *Warning*.
 * **Keywords** (opcional) establece el valor del entero de combinaciones de palabras clave que se usará.

## <a name="use-the-trace-api-directly"></a>Uso de la API de seguimiento directamente
Puede llamar directamente a la API de seguimiento de Application Insights. Los adaptadores de registro usan esta API.

Por ejemplo:

```csharp
var telemetry = new Microsoft.ApplicationInsights.TelemetryClient();
telemetry.TrackTrace("Slow response - database01");
```

Una ventaja de TrackTrace es que puede colocar datos relativamente largos en el mensaje. Por ejemplo, aquí puede codificar datos POST.

También puede agregar un nivel de gravedad al mensaje. Además, al igual que con otra telemetría, puede agregar valores de propiedad para ayudar a filtrar o buscar distintos conjuntos de seguimientos. Por ejemplo:

  ```csharp
  var telemetry = new Microsoft.ApplicationInsights.TelemetryClient();
  telemetry.TrackTrace("Slow database response",
                 SeverityLevel.Warning,
                 new Dictionary<string,string> { {"database", db.ID} });
  ```

Esto permitiría filtrar fácilmente en [Búsqueda][diagnostic] todos los mensajes de un determinado nivel de gravedad relativos a una determinada base de datos.

## <a name="azureloghandler-for-opencensus-python"></a>AzureLogHandler para OpenCensus para Python
El controlador de registro de Azure Monitor permite exportar registros de Python a Azure Monitor.

Instrumente la aplicación con el [SDK de OpenCensus para Python](./opencensus-python.md) para Azure Monitor.

En este ejemplo se muestra cómo enviar un registro de nivel de advertencia a Azure Monitor.

```python
import logging

from opencensus.ext.azure.log_exporter import AzureLogHandler

logger = logging.getLogger(__name__)
logger.addHandler(AzureLogHandler(connection_string='InstrumentationKey=<your-instrumentation_key-here>'))
logger.warning('Hello, World!')
```

## <a name="explore-your-logs"></a>Explorar los registros
Ejecute la aplicación en modo de depuración o impleméntela para que esté activa.

En el panel de información general de su aplicación, en el [portal de Application Insights][portal], seleccione [Buscar][diagnostic].

Por ejemplo, puede:

* Filtrar por seguimientos de registro o por elementos con determinadas propiedades.
* Inspeccionar un elemento específico en detalle
* Buscar otros datos de registro del sistema que se relacionen con la misma solicitud de usuario (tiene el mismo OperationId).
* Guardar la configuración de una página como favorita.

> [!NOTE]
>Si usa el SDK de Application Insights para ASP.NET versión 2.0.0-beta3 o posterior y la aplicación envía una gran cantidad de datos, la característica de *muestreo adaptativo* puede operar y enviar solamente una parte de los datos de telemetría. [Aprenda más sobre el muestreo.](./sampling.md)
>

## <a name="troubleshooting"></a>Solución de problemas
### <a name="how-do-i-do-this-for-java"></a>¿Cómo se puede hacer para Java?
En la instrumentación sin código de Java (recomendado) donde los registros se recopilan de forma predefinida, use el [agente Java 3.0](./java-in-process-agent.md).

Si usa el SDK de Java, utilice los [adaptadores de registro de Java](./java-trace-logs.md).

### <a name="theres-no-application-insights-option-on-the-project-context-menu"></a>No aparece la opción de Application Insights en el menú contextual del proyecto
* Asegúrese de que Developer Analytics Tools está instalado en el equipo de desarrollo. En Visual Studio, en **Herramientas** > **Extensiones y actualizaciones**, busque **Developer Analytics Tools**. Si no se encuentra en la pestaña **Instalado**, abra la pestaña **En línea** e instálelo.
* Podría tratarse de un tipo de proyecto que no es compatible con Developer Analytics Tools. Use la [instalación manual](#manual-installation).

### <a name="theres-no-log-adapter-option-in-the-configuration-tool"></a>No aparece la opción de adaptador de registro en la herramienta de configuración
* Instale primero la plataforma de registro.
* Si usa System.Diagnostics.Trace, asegúrese de que lo ha [configurado en *web.config*](/dotnet/api/system.diagnostics.eventlogtracelistener?view=dotnet-plat-ext-3.1).
* Asegúrese de que tiene la última versión de Application Insights. En Visual Studio, vaya a **Herramientas** > **Extensiones y actualizaciones** y abra la pestaña **Actualizaciones**. Si **Developer Analytics Tools** está instalado, selecciónelo para actualizarlo.

### <a name="i-get-the-instrumentation-key-cannot-be-empty-error-message"></a><a name="emptykey"></a>Aparece el mensaje de error "La clave de instrumentación no puede estar vacía"
Probablemente haya instalado el paquete NuGet del adaptador de registro sin instalar Application Insights. En el Explorador de soluciones, haga clic con el botón derecho en *ApplicationInsights.config* y seleccione **Actualizar Application Insights**. Se le pedirá que inicie sesión en Azure y que cree un recurso de Application Insights o que vuelva a usar uno existente. Esto debería corregir el problema.

### <a name="i-can-see-traces-but-not-other-events-in-diagnostic-search"></a>Puedo ver seguimientos en la búsqueda de diagnóstico, pero no otros eventos
El paso de todos los eventos y solicitudes por la canalización puede llevar un rato.

### <a name="how-much-data-is-retained"></a><a name="limits"></a>¿Qué cantidad de datos se conserva?
Hay varios factores que afectan a la cantidad de datos que se conservan. Para obtener más información, vea la sección [Límites](./api-custom-events-metrics.md#limits) de la página de métricas de eventos de cliente.

### <a name="i-dont-see-some-log-entries-that-i-expected"></a>No veo algunas entradas del registro que esperaba
Si usa el SDK de Application Insights para ASP.NET versión 2.0.0-beta3 o posterior y la aplicación envía grandes cantidades de datos, la característica de muestreo adaptativo puede operar y enviar solamente una parte de los datos de telemetría. [Aprenda más sobre el muestreo.](./sampling.md)

## <a name="next-steps"></a><a name="add"></a>Pasos siguientes

* [Diagnóstico de errores y excepciones en ASP.NET][exceptions]
* [Más información sobre la búsqueda][diagnostic]
* [Configuración de pruebas de disponibilidad y de capacidad de respuesta][availability]
* [Solución de problemas][qna]

<!--Link references-->

[availability]: ./monitor-web-app-availability.md
[diagnostic]: ./diagnostic-search.md
[exceptions]: asp-net-exceptions.md
[portal]: https://portal.azure.com/
[qna]: ../faq.md
[start]: ./app-insights-overview.md

