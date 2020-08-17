---
title: 'Diagnóstico con Live Metrics Stream: Azure Application Insights'
description: Supervise la aplicación web en tiempo real con métricas personalizadas y diagnostique problemas con una fuente directa de errores, seguimientos y eventos.
ms.topic: conceptual
ms.date: 04/22/2019
ms.reviewer: sdash
ms.openlocfilehash: 4b84088c1213801e61a4c669bccb1a983c999310
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87321945"
---
# <a name="live-metrics-stream-monitor--diagnose-with-1-second-latency"></a>Live Metrics Stream: supervisión y diagnóstico con una latencia de 1 segundo

Supervise la aplicación web, ya activa y en producción, con Live Metrics Stream de [Application Insights](./app-insights-overview.md). Seleccione métricas y contadores de rendimiento y fíltrelos para inspeccionarlos en tiempo real sin que el servicio se vea afectado. Inspeccione seguimientos de la pila procedentes de ejemplos de errores de solicitudes y excepciones. Junto con [Profiler](./profiler.md) y el [depurador de instantáneas](./snapshot-debugger.md), Live Metrics Stream proporciona una herramienta de diagnóstico eficaz y no invasiva para sitios web activos.

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

1. [Instale Application Insights](../azure-monitor-app-hub.yml) en la aplicación.
2. Además de los paquetes de Application Insights estándar, se requiere [Microsoft.ApplicationInsights.PerfCounterCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.PerfCounterCollector/) para habilitar Live Metrics Stream.
3. **Actualice a la versión más reciente** del paquete de Application Insights. En Visual Studio, haga clic con el botón derecho en el proyecto y elija **Administrar paquetes NuGet**. Abra la pestaña **Actualizaciones** y seleccione todos los paquetes Microsoft.ApplicationInsights.*.

    Vuelva a implementar la aplicación.

3. En [Azure Portal](https://portal.azure.com), abra el recurso de Application Insights de la aplicación y, a continuación, abra Live Stream.

4. [Proteja el canal de control](#secure-the-control-channel) si puede que vaya a usar información confidencial como nombres de clientes en los filtros.

### <a name="no-data-check-your-server-firewall"></a>¿No hay datos? Comprobación del firewall del servidor

Compruebe que [los puertos de salida de Live Metrics Stream](./ip-addresses.md#outgoing-ports) estén abiertos en el firewall de los servidores.

## <a name="how-does-live-metrics-stream-differ-from-metrics-explorer-and-analytics"></a>¿En qué se diferencia Live Metrics Stream de Explorador de métricas y Analytics?

| |Live Stream | Explorador de métricas y Analytics |
|---|---|---|
|**Latency**|Los datos se muestran en un segundo.|La agregación se realiza en minutos.|
|**Sin retención**|Los datos se conservan solo mientras se encuentren en el gráfico y luego se descartan.|[Los datos se conservan durante 90 días.](./data-retention-privacy.md#how-long-is-the-data-kept)|
|**A petición**|Los datos solo se transmiten mientras el panel de Live Metrics está abierto. |Se envían datos siempre que el SDK esté instalado y habilitado.|
|**Gratis**|No se efectúa ningún cargo por los datos de Live Stream.|Están sujetos a aplicación de [precios](./pricing.md).
|**Muestreo**|Se transmiten todas las métricas y los contadores seleccionados. Se muestrean los errores y seguimientos de la pila. No se aplican elementos TelemetryProcessor.|Se pueden [muestrear](./api-filtering-sampling.md) eventos.|
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
> Actualmente, solo se puede configurar un canal autenticado mediante la supervisión de base de código y no se pueden autenticar servidores mediante la asociación sin código.

Los criterios de filtros personalizados que especifique se envían hacia el componente de Live Metrics del SDK de Application Insights. Los filtros podrían contener información confidencial, como identificadores de clientes. Puede hacer que el canal seguro con una clave secreta de API, además de la clave de instrumentación.
### <a name="create-an-api-key"></a>Creación de una clave de API

![Clave de API > Crear una clave de API](./media/live-stream/api-key.png)
![Pestaña Crear una clave de API Selección de "Autenticar canal de control del SDK" y luego "Generar clave"](./media/live-stream/create-api-key.png)

### <a name="add-api-key-to-configuration"></a>Incorporación de la clave de API a la configuración

### <a name="classic-aspnet"></a>ASP.NET clásico

En el archivo applicationinsights.config, agregue AuthenticationApiKey a QuickPulseTelemetryModule:
``` XML

<Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse.QuickPulseTelemetryModule, Microsoft.AI.PerfCounterCollector">
      <AuthenticationApiKey>YOUR-API-KEY-HERE</AuthenticationApiKey>
</Add>

```
O bien, en el código, establezca esto en QuickPulseTelemetryModule:

```csharp
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
using Microsoft.ApplicationInsights.Extensibility;

             TelemetryConfiguration configuration = new TelemetryConfiguration();
            configuration.InstrumentationKey = "YOUR-IKEY-HERE";

            QuickPulseTelemetryProcessor processor = null;

            configuration.TelemetryProcessorChainBuilder
                .Use((next) =>
                {
                    processor = new QuickPulseTelemetryProcessor(next);
                    return processor;
                })
                        .Build();

            var QuickPulse = new QuickPulseTelemetryModule()
            {

                AuthenticationApiKey = "YOUR-API-KEY"
            };
            QuickPulse.Initialize(configuration);
            QuickPulse.RegisterTelemetryProcessor(processor);
            foreach (var telemetryProcessor in configuration.TelemetryProcessors)
                {
                if (telemetryProcessor is ITelemetryModule telemetryModule)
                    {
                    telemetryModule.Initialize(configuration);
                    }
                }

```

### <a name="azure-function-apps"></a>Azure Function Apps

En el caso de Azure Function Apps (versión 2), la protección del canal con una clave de API puede realizarse con una variable de entorno.

Cree una clave de API desde su recurso de Application Insights y vaya a **Configuración de la aplicación** en Function App. Seleccione **agregar nueva configuración** y escriba un nombre de `APPINSIGHTS_QUICKPULSEAUTHAPIKEY` y un valor que corresponda a la clave de API.

### <a name="aspnet-core-requires-application-insights-aspnet-core-sdk-230-or-greater"></a>ASP.NET Core (requiere el SDK de Application Insights ASP.NET Core 2.3.0 o superior)

Modifique el archivo startup.cs como sigue:

Primero agregue

```csharp
using Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse;
```

Después, en el método ConfigureServices, agregue:

```csharp
services.ConfigureTelemetryModule<QuickPulseTelemetryModule> ((module, o) => module.AuthenticationApiKey = "YOUR-API-KEY-HERE");
```

Sin embargo, si reconoce y confía en todos los servidores conectados, puede probar los filtros personalizados sin el canal autenticado. Esta opción está disponible durante seis meses. Esta invalidación se requiere una vez cada nueva sesión, o cuando se pone en línea un nuevo servidor.

![Opciones de autenticación de Live Metrics](./media/live-stream/live-stream-auth.png)

>[!NOTE]
>Se recomienda que configure el canal autenticado antes de escribir información potencialmente confidencial, como identificadores de clientes en los criterios de filtro.
>

## <a name="supported-features-table"></a>Tabla de características compatibles

| Idioma                         | Métricas básicas       | Métricas de rendimiento | Filtrado personalizado    | Telemetría de ejemplo    | División de CPU por proceso |
|----------------------------------|:--------------------|:--------------------|:--------------------|:--------------------|:---------------------|
| .NET                             | Se admite (V2.7.2+) | Se admite (V2.7.2+) | Se admite (V2.7.2+) | Se admite (V2.7.2+) | Se admite (V2.7.2+)  |
| .NET Core (destino=.NET Framework)| Se admite (V2.4.1+) | Se admite (V2.4.1+) | Se admite (V2.4.1+) | Se admite (V2.4.1+) | Se admite (V2.4.1+)  |
| .NET Core (destino=.NET Core)     | Se admite (V2.4.1+) | Admitido*          | Se admite (V2.4.1+) | Se admite (V2.4.1+) | **No admitido**    |
| Azure Functions v2               | Compatible           | Compatible           | Compatible           | Compatible           | **No admitido**    |
| Java                             | Se admite (V2.0.0+) | Se admite (V2.0.0+) | **No admitido**   | **No admitido**   | **No admitido**    |
| Node.js                          | Se admite (V1.3.0+) | Se admite (V1.3.0+) | **No admitido**   | Se admite (V1.3.0+) | **No admitido**    |

Las métricas básicas incluyen la tasa de solicitudes, dependencias y excepciones. Las métricas de rendimiento (contadores de rendimiento) incluyen la memoria y la CPU. La telemetría de ejemplo muestra un flujo de información detallada para las solicitudes y dependencias con errores, excepciones, eventos y seguimientos.

 \* La compatibilidad con PerfCounters varía ligeramente en las diversas versiones de .NET Core que no tienen .NET Framework como destino:

- Se admiten las métricas de PerfCounters cuando se ejecutan en Azure App Service para Windows. (AspNetCore SDK Version 2.4.1 o superior)
- Se admiten PerfCounters cuando la aplicación se ejecuta en CUALQUIER máquina de Windows, ya sea una máquina virtual, un servicio en la nube, local, etc. (AspNetCore SDK versión 2.7.1 o superior), pero para las aplicaciones que tienen como destino .NET Core 2.0 o superior.
- Se admiten PerfCounters cuando la aplicación se ejecuta en CUALQUIER LUGAR (Linux, Windows, App Service para Linux, contenedores, etc.) en la versión beta más reciente (es decir, AspNetCore SDK versión 2.8.0-beta1 o superior), pero para aplicaciones que tienen como destino .NET Core 2.0 o superior.

De manera predeterminada, Live Metrics está deshabilitado en el SDK de Node.js. Para habilitar Live Metrics, agregue `setSendLiveMetrics(true)` a los [métodos de configuración](https://github.com/Microsoft/ApplicationInsights-node.js#configuration) al inicializar el SDK.

## <a name="troubleshooting"></a>Solución de problemas

¿No hay datos? Si la aplicación está en una red protegida: Live Metrics Stream usa direcciones IP diferentes a las de otros datos de telemetría de Application Insights. Asegúrese de que [esas direcciones IP](./ip-addresses.md) están abiertos en el firewall.

## <a name="next-steps"></a>Pasos siguientes
* [Supervisión del uso con Application Insights](./usage-overview.md)
* [Uso de la Búsqueda de diagnóstico](./diagnostic-search.md)
* [Generador de perfiles](./profiler.md)
* [Depurador de instantáneas](./snapshot-debugger.md)

