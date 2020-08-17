---
title: Supervisión del uso y el rendimiento en las aplicaciones de escritorio de Windows
description: Analice el uso y el rendimiento de la aplicación de escritorio de Windows con Application Insights.
ms.topic: conceptual
ms.date: 06/11/2020
ms.openlocfilehash: 17613fc6cea24643c2b88182e7e56a1d216b2da8
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87323424"
---
# <a name="monitoring-usage-and-performance-in-classic-windows-desktop-apps"></a>Supervisión del uso y el rendimiento en las aplicaciones de escritorio de Windows clásicas

Las aplicaciones hospedadas en el entorno local, en Azure y en otras nubes pueden aprovechar las ventajas de Application Insights. La única limitación es la necesidad de [permitir la comunicación](./ip-addresses.md) al servicio de Application Insights. Para supervisar las aplicaciones de la Plataforma universal de Windows (UWP), se recomienda [Visual Studio App Center](../learn/mobile-center-quickstart.md).

## <a name="to-send-telemetry-to-application-insights-from-a-classic-windows-application"></a>Para enviar datos de telemetría a Application Insights desde una aplicación Windows clásica
1. En [Azure Portal](https://portal.azure.com), [cree un recurso de Application Insights](./create-new-resource.md). 
2. Realice una copia de la clave de instrumentación.
3. En Visual Studio, edite los paquetes NuGet de su proyecto de aplicación y agregue Microsoft.ApplicationInsights.WindowsServer. (O elija Microsoft.ApplicationInsights si únicamente le interesa la API de base, sin los módulos de recopilación de datos de telemetría estándar).
4. Establezca la clave de instrumentación en el código:
   
    `TelemetryConfiguration.Active.InstrumentationKey = "` *su clave* `";`
   
    o en ApplicationInsights.config (si tiene instalado uno de los paquetes de telemetría estándar):
   
    `<InstrumentationKey>`*su clave*`</InstrumentationKey>` 
   
    Si usa ApplicationInsights.config, asegúrese de que sus propiedades en el Explorador de soluciones se establecen en **Acción de compilación = Contenido, Copiar en el directorio de salida = Copiar**.
5. [Use la API](./api-custom-events-metrics.md) para enviar telemetría.
6. Ejecute la aplicación y vea la telemetría en el recurso que creó en Azure Portal.

## <a name="example-code"></a><a name="telemetry"></a>Ejemplo de código

```csharp
using Microsoft.ApplicationInsights;

    public partial class Form1 : Form
    {
        private TelemetryClient tc = new TelemetryClient();
        ...
        private void Form1_Load(object sender, EventArgs e)
        {
            // Alternative to setting ikey in config file:
            tc.InstrumentationKey = "key copied from portal";

            // Set session data:
            tc.Context.Session.Id = Guid.NewGuid().ToString();
            tc.Context.Device.OperatingSystem = Environment.OSVersion.ToString();

            // Log a page view:
            tc.TrackPageView("Form1");
            ...
        }

        protected override void OnClosing(System.ComponentModel.CancelEventArgs e)
        {
            e.Cancel = true;

            if (tc != null)
            {
                tc.Flush(); // only for desktop apps

                // Allow time for flushing:
                System.Threading.Thread.Sleep(1000);
            }
            base.OnClosing(e);
        }

```

## <a name="override-storage-of-computer-name"></a>Invalidación del almacenamiento del nombre de equipo

De forma predeterminada, este SDK recopilará y almacenará el nombre de equipo del sistema que emite la telemetría.

El [plan de tarifa Enterprise (por nodo) heredado](./pricing.md#legacy-enterprise-per-node-pricing-tier) de Application Insights usa el nombre de equipo para la facturación interna. De forma predeterminada, si usa un inicializador de telemetría para invalidar `telemetry.Context.Cloud.RoleInstance`, se enviará una propiedad `ai.internal.nodeName` independiente con el valor del nombre de equipo. Este valor no se almacenará con los datos de telemetría de Application Insights, sino que se usa internamente en la ingesta para permitir la compatibilidad con versiones anteriores con el modelo de facturación basado en nodos heredado.

Si está en el [plan de tarifa Enterprise (por nodo) heredado](./pricing.md#legacy-enterprise-per-node-pricing-tier) y simplemente necesita invalidar el almacenamiento del nombre de equipo, use un inicializador de telemetría:

**Escriba un elemento TelemetryInitializer personalizado como el siguiente.**

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;

namespace CustomInitializer.Telemetry
{
    public class MyTelemetryInitializer : ITelemetryInitializer
    {
        public void Initialize(ITelemetry telemetry)
        {
            if (string.IsNullOrEmpty(telemetry.Context.Cloud.RoleInstance))
            {
                // Set custom role name here. Providing an empty string will result
                // in the computer name still be sent via this property.
                  telemetry.Context.Cloud.RoleInstance = "Custom RoleInstance";
            }
        }
    }
}
```

Cree una instancia del inicializador en el método `Main()` de `Program.cs` que aparece a continuación al establecer la clave de instrumentación:

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;

   static void Main()
        {
            TelemetryConfiguration.Active.InstrumentationKey = "{Instrumentation-key-here}";
            TelemetryConfiguration.Active.TelemetryInitializers.Add(new MyTelemetryInitializer());
            //...
        }
```

## <a name="override-transmission-of-computer-name"></a>Invalidación de la transmisión del nombre de equipo

Si no está en el [plan de tarifa Enterprise (por nodo) heredado](./pricing.md#legacy-enterprise-per-node-pricing-tier) y desea evitar completamente que los datos de telemetría contengan el nombre de equipo, debe usar un procesador de telemetría.

### <a name="telemetry-processor"></a>Procesador de telemetría

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.Extensibility;


namespace WindowsFormsApp2
{
    public class CustomTelemetryProcessor : ITelemetryProcessor
    {
        private readonly ITelemetryProcessor _next;

        public CustomTelemetryProcessor(ITelemetryProcessor next)
        {
            _next = next;
        }

        public void Process(ITelemetry item)
        {
            if (item != null)
            {
                item.Context.Cloud.RoleInstance = string.Empty;
            }

            _next.Process(item);
        }
    }
}
```

Cree una instancia del procesador de telemetría en el método `Main()` de `Program.cs` que aparece después de establecer la clave de instrumentación:

```csharp
using Microsoft.ApplicationInsights.Extensibility;

namespace WindowsFormsApp2
{
    static class Program
    {
        static void Main()
        {
            TelemetryConfiguration.Active.InstrumentationKey = "{Instrumentation-key-here}";
            var builder = TelemetryConfiguration.Active.DefaultTelemetrySink.TelemetryProcessorChainBuilder;
            builder.Use((next) => new CustomTelemetryProcessor(next));
            builder.Build();
            //...
        }
    }
}

```

> [!NOTE]
> Aunque técnicamente puede usar un procesador de telemetría tal y como se describió anteriormente aunque se encuentre en el plan de tarifa [empresa heredada (por nodo)](./pricing.md#legacy-enterprise-per-node-pricing-tier), esto dará lugar a una posible facturación excesiva debido a la incapacidad de distinguir correctamente los nodos en el caso de los precios por nodo.

## <a name="next-steps"></a>Pasos siguientes
* [Creación de un panel](./overview-dashboard.md)
* [Búsqueda de diagnóstico](./diagnostic-search.md)
* [Exploración de métricas](../platform/metrics-charts.md)
* [Escribir consultas de Analytics](../log-query/log-query-overview.md)

