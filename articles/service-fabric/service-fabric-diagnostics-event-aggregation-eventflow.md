---
title: Agregación de eventos de Azure Service Fabric con EventFlow
description: Obtenga información sobre la agregación y la recopilación de eventos con EventFlow para la supervisión y el diagnóstico de clústeres de Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 2/25/2019
ms.author: srrengar
ms.custom: devx-track-csharp
ms.openlocfilehash: a80eea5a50aa7b1e441049eeb2cae381994cd3ec
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89018661"
---
# <a name="event-aggregation-and-collection-using-eventflow"></a>Recopilación y agregación de eventos con EventFlow

[Microsoft Diagnostics EventFlow](https://github.com/Azure/diagnostics-eventflow) puede enrutar los eventos de un nodo a uno o varios destinos de supervisión. Dado que se incluye como un paquete de NuGet en el proyecto de servicio, el código y la configuración de EventFlow van con el servicio, por lo que se elimina el problema de la configuración por nodo que se mencionó antes sobre Diagnósticos de Azure. EventFlow se ejecuta dentro del proceso de servicio y se conecta directamente con las salidas configuradas. Gracias a esta conexión directa, EventFlow funciona en implementaciones de servicios de Azure, de contenedor o locales. Tenga cuidado si ejecuta EventFlow en escenarios de alta densidad, como un contenedor, ya que cada canalización EventFlow genera una conexión externa. Por tanto, si hospeda varios procesos, obtendrá varias conexiones salientes. Esto no supone un problema para las aplicaciones de Service Fabric, ya que todas las réplicas de `ServiceType` se ejecutan en el mismo proceso, lo que limita el número de conexiones salientes. EventFlow también proporciona filtrado de eventos, de manera que solo se envían los eventos que coinciden con el filtro especificado.

## <a name="set-up-eventflow"></a>Configuración de EventFlow

Los archivos binarios de EventFlow están disponibles como un conjunto de paquetes de NuGet. Para agregar EventFlow a un proyecto de servicio de Service Fabric, en el Explorador de soluciones, haga clic con el botón derecho en el proyecto y elija "Administrar paquetes de NuGet". Cambie a la pestaña "Examinar" y busque "`Diagnostics.EventFlow`":

![Paquetes de NuGet para EventFlow en la interfaz de usuario del administrador de paquetes de NuGet de Visual Studio](./media/service-fabric-diagnostics-event-aggregation-eventflow/eventflow-nuget.png)

Aparecerá una lista de los diferentes paquetes, marcados con las etiquetas "Entradas" y "Salidas". EventFlow es compatible con varios analizadores y proveedores de registro. El servicio que hospede EventFlow debe incluir los paquetes adecuados según el origen y destino de los registros de aplicaciones. Además del paquete principal de Service Fabric, también necesita tener configurado al menos un paquete de entrada y de salida. Por ejemplo, puede agregar los siguientes paquetes para enviar eventos de EventSource a Application Insights:

* `Microsoft.Diagnostics.EventFlow.Inputs.EventSource` (para capturar datos de la clase EventSource del servicio y de las clases EventSource estándar como *Microsoft-ServiceFabric-Services* y *Microsoft-ServiceFabric-Actors*)
* `Microsoft.Diagnostics.EventFlow.Outputs.ApplicationInsights` (vamos a enviar los registros a un recurso de Azure Application Insights)
* `Microsoft.Diagnostics.EventFlow.ServiceFabric` (permite inicializar la canalización de EventFlow desde la configuración del servicio de Service Fabric, y notifica los problemas con el envío de datos de diagnóstico como informes de mantenimiento de Service Fabric)

>[!NOTE]
>El paquete `Microsoft.Diagnostics.EventFlow.Inputs.EventSource` necesita que proyecto de servicio sea para .NET Framework 4.6 o versiones posteriores. Asegúrese de establecer la plataforma de destino adecuada en las propiedades del proyecto antes de instalar este paquete.

Una vez instalados todos los paquetes, el siguiente paso es configurar y habilitar EventFlow en el servicio.

## <a name="configure-and-enable-log-collection"></a>Configuración y habilitación de la colección de registros
La canalización de EventFlow, responsable del envío de los registros, se crea a partir de una especificación almacenada en un archivo de configuración. El paquete `Microsoft.Diagnostics.EventFlow.ServiceFabric` instala un archivo de configuración inicial de EventFlow en la carpeta de la solución `PackageRoot\Config`, llamada `eventFlowConfig.json`. Este archivo de configuración debe modificarse para capturar los datos de la clase `EventSource` del servicio predeterminado y cualquier otra entrada que desee configurar y enviar los datos al servicio Application Insights.

>[!NOTE]
>Si el archivo de proyecto tiene un formato de VisualStudio 2017, el archivo `eventFlowConfig.json` no se añadirá automáticamente. Para corregir esto, cree el archivo en la carpeta `Config` y establezca la acción de compilación en `Copy if newer`. 

A continuación, se presenta un ejemplo de *eventFlowConfig.json* basado en los paquetes NuGet mencionados anteriormente:
```json
{
  "inputs": [
    {
      "type": "EventSource",
      "sources": [
        { "providerName": "Microsoft-ServiceFabric-Services" },
        { "providerName": "Microsoft-ServiceFabric-Actors" },
        // (replace the following value with your service's ServiceEventSource name)
        { "providerName": "your-service-EventSource-name" }
      ]
    }
  ],
  "filters": [
    {
      "type": "drop",
      "include": "Level == Verbose"
    }
  ],
  "outputs": [
    {
      "type": "ApplicationInsights",
      // (replace the following value with your AI resource's instrumentation key)
      "instrumentationKey": "00000000-0000-0000-0000-000000000000"
    }
  ],
  "schemaVersion": "2016-08-11"
}
```

El nombre del origen ServiceEventSource del servicio es el valor de la propiedad Name de `EventSourceAttribute` aplicado a la clase ServiceEventSource. Se especifica todo en el archivo `ServiceEventSource.cs`, que forma parte del código del servicio. Por ejemplo, en el siguiente fragmento de código, el nombre del origen ServiceEventSource es *MyCompany-Application1-Stateless1*:

```csharp
[EventSource(Name = "MyCompany-Application1-Stateless1")]
internal sealed class ServiceEventSource : EventSource
{
    // (rest of ServiceEventSource implementation)
}
```

Tenga en cuenta que el archivo `eventFlowConfig.json` forma parte del paquete de configuración del servicio. Los cambios en este archivo pueden incluirse en actualizaciones completas o de solo configuración del servicio; están sujetos a las comprobaciones del estado de actualización de Service Fabric y se revierten automáticamente en caso de errores durante la actualización. Para más información, consulte el [tutorial sobre la actualización de aplicaciones de Service Fabric](service-fabric-application-upgrade.md).

La sección de *filtros* de la configuración permite personalizar aún más la información que se va a enviar a las salidas a través de la canalización de EventFlow, de tal forma que se puede quitar o incluir información concreta o cambiar la estructura de los datos del evento. Para más información sobre filtrado, vea los [filtros de EventFlow](https://github.com/Azure/diagnostics-eventflow#filters).

El último paso consiste en crear una instancia de la canalización de EventFlow en el código de inicio del servicio, ubicado en el archivo `Program.cs`:

```csharp
using System;
using System.Diagnostics;
using System.Threading;
using Microsoft.ServiceFabric;
using Microsoft.ServiceFabric.Services.Runtime;

// **** EventFlow namespace
using Microsoft.Diagnostics.EventFlow.ServiceFabric;

namespace Stateless1
{
    internal static class Program
    {
        /// <summary>
        /// This is the entry point of the service host process.
        /// </summary>
        private static void Main()
        {
            try
            {
                // **** Instantiate log collection via EventFlow
                using (var diagnosticsPipeline = ServiceFabricDiagnosticPipelineFactory.CreatePipeline("MyApplication-MyService-DiagnosticsPipeline"))
                {

                    ServiceRuntime.RegisterServiceAsync("Stateless1Type",
                    context => new Stateless1(context)).GetAwaiter().GetResult();

                    ServiceEventSource.Current.ServiceTypeRegistered(Process.GetCurrentProcess().Id, typeof(Stateless1).Name);

                    Thread.Sleep(Timeout.Infinite);
                }
            }
            catch (Exception e)
            {
                ServiceEventSource.Current.ServiceHostInitializationFailed(e.ToString());
                throw;
            }
        }
    }
}
```

El nombre que se pasa como parámetro del método `CreatePipeline` de `ServiceFabricDiagnosticsPipelineFactory` es el nombre de la *entidad de estado de mantenimiento* que representa la canalización de recopilación de registros de EventFlow. Este nombre se usa si EventFlow encuentra un error y lo notifica a través del subsistema de estado de mantenimiento de Service Fabric.

### <a name="use-service-fabric-settings-and-application-parameters-in-eventflowconfig"></a>Uso de los parámetros de la aplicación y de la configuración de Service Fabric en eventFlowConfig

EventFlow permite usar los parámetros de la aplicación y las opciones de Service Fabric para su propia configuración. Puede hacer referencia a los parámetros de configuración de Service Fabric mediante la utilización de esta sintaxis especial para los valores:

```json
servicefabric:/<section-name>/<setting-name>
```

`<section-name>` es el nombre de la sección de configuración de Service Fabric, y `<setting-name>` es el conjunto de configuración que proporciona el valor que se usará para configurar una opción de EventFlow. Para leer más información sobre cómo realizar esto, vaya a [Support for Service Fabric settings and application parameters](https://github.com/Azure/diagnostics-eventflow#support-for-service-fabric-settings-and-application-parameters) (Compatibilidad de los parámetros de la aplicación y la configuración de Service Fabric).

## <a name="verification"></a>Comprobación

Inicie el servicio y observe la ventana de resultados de depuración de Visual Studio. Una vez iniciado el servicio, deberá comenzar a ver la evidencia de que el servicio está enviando registros a la salida configurada. Vaya a la plataforma de visualización y análisis de eventos y confirme que los registros han empezado a mostrarse, una operación que puede tardar algunos minutos.

## <a name="next-steps"></a>Pasos siguientes

* [Análisis y visualización de eventos con Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md)
* [Análisis y visualización de eventos con los registros de Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md)
* [Documentación de EventFlow](https://github.com/Azure/diagnostics-eventflow)
