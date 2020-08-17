---
title: Habilite Snapshot Debugger para aplicaciones .NET en Azure Service Fabric, Servicio en la nube y máquinas virtuales | Microsoft Docs
description: Habilite Snapshot Debugger para aplicaciones .NET en Azure Service Fabric, servicio en la nube y máquinas virtuales
ms.topic: conceptual
author: brahmnes
ms.author: bfung
ms.date: 03/07/2019
ms.reviewer: mbullwin
ms.openlocfilehash: c1cc9893a309dcdf7ac575494d164052bb0c617c
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87325685"
---
# <a name="enable-snapshot-debugger-for-net-apps-in-azure-service-fabric-cloud-service-and-virtual-machines"></a>Habilite Snapshot Debugger para aplicaciones .NET en Azure Service Fabric, servicio en la nube y máquinas virtuales

Si la aplicación ASP.NET o ASP.NET Core se ejecuta en Azure App Service, se recomienda encarecidamente [habilitar Snapshot Debugger en la página del portal de Application Insights](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json). Sin embargo, si la aplicación requiere una configuración personalizada de Snapshot Debugger o una versión preliminar de .NET Core, debe seguir esta instrucción ***además*** de las instrucciones de [habilitación mediante la página del portal de Application Insights](snapshot-debugger-appservice.md?toc=/azure/azure-monitor/toc.json).

Si la aplicación se ejecuta en Azure Service Fabric, el servicio en la nube, máquinas virtuales o máquinas locales, se deben aplicar las instrucciones siguientes. 
    
## <a name="configure-snapshot-collection-for-aspnet-applications"></a>Configurar la recopilación de instantáneas para aplicaciones ASP.NET

1. [Habilite Application Insights en su aplicación web](./asp-net.md), si aún no lo ha hecho.

2. Incluya el paquete NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) en la aplicación.

3. Si es necesario, personalice la configuración de Snapshot Debugger agregada a [ApplicationInsights.config](./configuration-with-applicationinsights-config.md). La configuración predeterminada de Snapshot Debugger está principalmente vacía y todas las configuraciones son opcionales. Este es un ejemplo que muestra una configuración equivalente a la configuración predeterminada:

    ```xml
    <TelemetryProcessors>
        <Add Type="Microsoft.ApplicationInsights.SnapshotCollector.SnapshotCollectorTelemetryProcessor, Microsoft.ApplicationInsights.SnapshotCollector">
        <!-- The default is true, but you can disable Snapshot Debugging by setting it to false -->
        <IsEnabled>true</IsEnabled>
        <!-- Snapshot Debugging is usually disabled in developer mode, but you can enable it by setting this to true. -->
        <!-- DeveloperMode is a property on the active TelemetryChannel. -->
        <IsEnabledInDeveloperMode>false</IsEnabledInDeveloperMode>
        <!-- How many times we need to see an exception before we ask for snapshots. -->
        <ThresholdForSnapshotting>1</ThresholdForSnapshotting>
        <!-- The maximum number of examples we create for a single problem. -->
        <MaximumSnapshotsRequired>3</MaximumSnapshotsRequired>
        <!-- The maximum number of problems that we can be tracking at any time. -->
        <MaximumCollectionPlanSize>50</MaximumCollectionPlanSize>
        <!-- How often we reconnect to the stamp. The default value is 15 minutes.-->
        <ReconnectInterval>00:15:00</ReconnectInterval>
        <!-- How often to reset problem counters. -->
        <ProblemCounterResetInterval>1.00:00:00</ProblemCounterResetInterval>
        <!-- The maximum number of snapshots allowed in ten minutes.The default value is 1. -->
        <SnapshotsPerTenMinutesLimit>3</SnapshotsPerTenMinutesLimit>
        <!-- The maximum number of snapshots allowed per day. -->
        <SnapshotsPerDayLimit>30</SnapshotsPerDayLimit>
        <!-- Whether or not to collect snapshot in low IO priority thread. The default value is true. -->
        <SnapshotInLowPriorityThread>true</SnapshotInLowPriorityThread>
        <!-- Agree to send anonymous data to Microsoft to make this product better. -->
        <ProvideAnonymousTelemetry>true</ProvideAnonymousTelemetry>
        <!-- The limit on the number of failed requests to request snapshots before the telemetry processor is disabled. -->
        <FailedRequestLimit>3</FailedRequestLimit>
        </Add>
    </TelemetryProcessors>
    ```

4. Las instantáneas solo se recopilan en las excepciones de las que se informa a Application Insights. En algunos casos (por ejemplo, en versiones anteriores de la plataforma. NET), es posible que tenga que [configurar la recopilación de excepciones](./asp-net-exceptions.md#exceptions) para ver las excepciones con las instantáneas en el portal.


## <a name="configure-snapshot-collection-for-applications-using-aspnet-core-20-or-above"></a>Configuración de la recopilación de instantáneas para aplicaciones que usen ASP.NET Core 2.0 y versiones posteriores

1. [Habilite Application Insights en su aplicación web ASP.NET Core](./asp-net-core.md), si aún no lo ha hecho.

    > [!NOTE]
    > Procure que la aplicación haga referencia a la versión 2.1.1 (o una más reciente) del paquete Microsoft.ApplicationInsights.AspNetCore.

2. Incluya el paquete NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) en la aplicación.

3. Modifique la clase `Startup` de la aplicación para agregar y configurar el procesador de telemetría del recopilador de instantáneas.
    1. Si usa la versión 1.3.5 o posterior del paquete NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector), agregue las siguientes instrucciones mediante declaraciones en `Startup.cs`.

       ```csharp
            using Microsoft.ApplicationInsights.SnapshotCollector;
       ```

       Agregue lo siguiente al final del método ConfigureServices en la clase `Startup` de `Startup.cs`.

       ```csharp
            services.AddSnapshotCollector((configuration) => Configuration.Bind(nameof(SnapshotCollectorConfiguration), configuration));
       ```
    2. Si usa la versión 1.3.4 o anterior del paquete Nuget [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector), agregue lo siguiente mediante declaraciones en `Startup.cs`.

       ```csharp
       using Microsoft.ApplicationInsights.SnapshotCollector;
       using Microsoft.Extensions.Options;
       using Microsoft.ApplicationInsights.AspNetCore;
       using Microsoft.ApplicationInsights.Extensibility;
       ```
    
       Agregue la siguiente clase `SnapshotCollectorTelemetryProcessorFactory` a la clase `Startup`.
    
       ```csharp
       class Startup
       {
           private class SnapshotCollectorTelemetryProcessorFactory : ITelemetryProcessorFactory
           {
               private readonly IServiceProvider _serviceProvider;
    
               public SnapshotCollectorTelemetryProcessorFactory(IServiceProvider serviceProvider) =>
                   _serviceProvider = serviceProvider;
    
               public ITelemetryProcessor Create(ITelemetryProcessor next)
               {
                   var snapshotConfigurationOptions = _serviceProvider.GetService<IOptions<SnapshotCollectorConfiguration>>();
                   return new SnapshotCollectorTelemetryProcessor(next, configuration: snapshotConfigurationOptions.Value);
               }
           }
           ...
        ```
        Agregue los servicios `SnapshotCollectorConfiguration` y `SnapshotCollectorTelemetryProcessorFactory` a la canalización de inicio:
    
        ```csharp
           // This method gets called by the runtime. Use this method to add services to the container.
           public void ConfigureServices(IServiceCollection services)
           {
               // Configure SnapshotCollector from application settings
               services.Configure<SnapshotCollectorConfiguration>(Configuration.GetSection(nameof(SnapshotCollectorConfiguration)));
    
               // Add SnapshotCollector telemetry processor.
               services.AddSingleton<ITelemetryProcessorFactory>(sp => new SnapshotCollectorTelemetryProcessorFactory(sp));
    
               // TODO: Add other services your application needs here.
           }
       }
       ```

4. Si es necesario, personalice la configuración de Snapshot Debugger; para ello, agregue una sección SnapshotCollectorConfiguration a appsettings.json. Todos los ajustes de la configuración de Snapshot Debugger son opcionales. Este es un ejemplo que muestra una configuración equivalente a la configuración predeterminada:

   ```json
   {
     "SnapshotCollectorConfiguration": {
       "IsEnabledInDeveloperMode": false,
       "ThresholdForSnapshotting": 1,
       "MaximumSnapshotsRequired": 3,
       "MaximumCollectionPlanSize": 50,
       "ReconnectInterval": "00:15:00",
       "ProblemCounterResetInterval":"1.00:00:00",
       "SnapshotsPerTenMinutesLimit": 1,
       "SnapshotsPerDayLimit": 30,
       "SnapshotInLowPriorityThread": true,
       "ProvideAnonymousTelemetry": true,
       "FailedRequestLimit": 3
     }
   }
   ```

## <a name="configure-snapshot-collection-for-other-net-applications"></a>Configurar la recopilación de instantáneas para otras aplicaciones .NET

1. Si la aplicación aún no tiene Application Insights, debe empezar por [habilitar Application Insights y establecer la clave de instrumentación](./windows-desktop.md).

2. Agregue el paquete NuGet [Microsoft.ApplicationInsights.SnapshotCollector](https://www.nuget.org/packages/Microsoft.ApplicationInsights.SnapshotCollector) en la aplicación.

3. Las instantáneas solo se recopilan en las excepciones de las que se informa a Application Insights. Es posible que tenga que modificar el código para informar de las excepciones. El código de control de excepciones depende de la estructura de la aplicación, pero a continuación se muestra un ejemplo:
    ```csharp
   TelemetryClient _telemetryClient = new TelemetryClient();

   void ExampleRequest()
   {
        try
        {
            // TODO: Handle the request.
        }
        catch (Exception ex)
        {
            // Report the exception to Application Insights.
            _telemetryClient.TrackException(ex);

            // TODO: Rethrow the exception if desired.
        }
   }
    ```

## <a name="next-steps"></a>Pasos siguientes

- Genere tráfico para la aplicación que pueda desencadenar una excepción. A continuación, espere de 10 a 15 minutos para que se empiecen a enviar instantáneas a la instancia de Application Insights.
- Vea las [instantáneas](snapshot-debugger.md?toc=/azure/azure-monitor/toc.json#view-snapshots-in-the-portal) en Azure Portal.
- Por ayuda para solucionar problemas de Snapshot Debugger, consulte la sección [Solución de problemas de Snapshot Debugger](snapshot-debugger-troubleshoot.md?toc=/azure/azure-monitor/toc.json).

