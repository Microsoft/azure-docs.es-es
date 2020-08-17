---
title: Eliminación de Application Insights en Visual Studio - Azure Monitor
description: Procedimiento para quitar el SDK de Application Insights para ASP.NET y ASP.NET Core en Visual Studio.
ms.topic: conceptual
ms.date: 04/06/2020
ms.openlocfilehash: 08a6d29e9170dde4bf3cc12015d126874006a778
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87049768"
---
# <a name="how-to-remove-application-insights-in-visual-studio"></a>Procedimiento para quitar Application Insights en Visual Studio

En este artículo se mostrará cómo quitar el SDK de Application Insights de ASP.NET y ASP.NET Core en Visual Studio.

Para quitar Application Insights, deberá quitar los paquetes NuGet y las referencias de la API de la aplicación. Puede desinstalar los paquetes NuGet mediante la consola de Administración de paquetes o la solución para administrar NuGet en Visual Studio. En las secciones siguientes se muestran dos maneras de quitar los paquetes NuGet y todo lo que se agregó automáticamente en el proyecto. Asegúrese de confirmar los archivos agregados y las áreas de su propio código en los que se han quitado las llamadas realizadas a la API.

## <a name="uninstall-using-the-package-management-console"></a>Desinstalación mediante la consola de Administración de paquetes

# <a name="net"></a>[.NET](#tab/net)

1. Para abrir la consola de Administración de paquetes, en el menú superior, seleccione Herramientas > Administrador de paquetes NuGet > Consola del Administrador de paquetes.
     
    ![En el menú superior, haga clic en Herramientas > Administrador de paquetes NuGet > Consola del Administrador de paquetes](./media/remove-application-insights/package-manager.png)

    > [!NOTE]
    > Si la recopilación de seguimiento está habilitada, primero debe desinstalar Microsoft.ApplicationInsights.TraceListener. Escriba `Uninstall-package Microsoft.ApplicationInsights.TraceListener` y siga el paso siguiente para quitar Microsoft.ApplicationInsights.Web.

1. Escriba el comando siguiente: `Uninstall-Package Microsoft.ApplicationInsights.Web -RemoveDependencies`

    Después de escribir el comando, el paquete de Application Insights y todas sus dependencias se desinstalarán del proyecto.
    
    ![Escriba el comando en la consola](./media/remove-application-insights/package-management-console.png)

# <a name="net-core"></a>[.NET Core](#tab/netcore)

1. Para abrir la consola de Administración de paquetes, en el menú superior, seleccione Herramientas > Administrador de paquetes NuGet > Consola del Administrador de paquetes.

    ![En el menú superior, haga clic en Herramientas > Administrador de paquetes NuGet > Consola del Administrador de paquetes](./media/remove-application-insights/package-manager.png)

1. Escriba el comando siguiente: ` Uninstall-Package Microsoft.ApplicationInsights.AspNetCore -RemoveDependencies`

    Después de escribir el comando, el paquete de Application Insights y todas sus dependencias se desinstalarán del proyecto.

---

## <a name="uninstall-using-the-visual-studio-nugetui"></a>Desinstalación mediante la interfaz de usuario NuGet de Visual Studio

# <a name="net"></a>[.NET](#tab/net)

1. En el *Explorador de soluciones* de la derecha, haga clic con el botón derecho en **Solución** y seleccione **Administrar paquetes NuGet para la solución**.

    Verá una pantalla que le permite editar todos los paquetes NuGet que forman parte del proyecto.
    
     ![Haga clic con el botón derecho en Solución en el Explorador de soluciones y seleccione Administrar paquetes NuGet para la solución](./media/remove-application-insights/manage-nuget-framework.png)

    > [!NOTE]
    > Si está habilitada la recopilación de seguimiento, primero debe desinstalar Microsoft.ApplicationInsights.TraceListener sin seleccionar la opción para quitar las dependencias y, a continuación, seguir los pasos que se indican a continuación para desinstalar Microsoft.ApplicationInsights.Web con la opción para quitar las dependencias seleccionada.
    
1. Haga clic en el paquete "Microsoft.ApplicationInsights.Web". A la derecha, marque la casilla situada junto a  *Proyecto* para seleccionar todos los proyectos.
    
1. Para quitar todas las dependencias en la desinstalación, seleccione el botón desplegable **Opciones** que se encuentra debajo de la sección donde seleccionó Proyecto.

    En *Desinstalar opciones*, seleccione la casilla situada junto a *Quitar dependencias*.

1. Seleccione **Desinstalar**.
    
    ![Marque Quitar dependencias y Desinstalar](./media/remove-application-insights/uninstall-framework.png)

    Se mostrará un cuadro de diálogo que muestra todas las dependencias que se van a quitar de la aplicación. Seleccione  **Aceptar** para desinstalar.
    
    ![Marque Quitar dependencias y Desinstalar](./media/remove-application-insights/preview-uninstall-framework.png)
    
1.  Después de desinstalar todo, es posible que siga apareciendo "ApplicationInsights.config" y "AiHandleErrorAttribute.cs" en el  *Explorador de soluciones*. Puede eliminar manualmente los dos archivos.

# <a name="net-core"></a>[.NET Core](#tab/netcore)

1. En el *Explorador de soluciones* de la derecha, haga clic con el botón derecho en **Solución** y seleccione **Administrar paquetes NuGet para la solución**.

    Verá una pantalla que le permite editar todos los paquetes NuGet que forman parte del proyecto.

    ![Haga clic con el botón derecho en Solución en el Explorador de soluciones y seleccione Administrar paquetes NuGet para la solución](./media/remove-application-insights/manage-nuget-core.png)

1. Haga clic en el paquete "Microsoft.ApplicationInsights.AspNetCore". A la derecha, marque la casilla situada junto a *Proyecto* para seleccionar todos los proyectos y, a continuación, seleccione **Desinstalar**.

    ![Marque Quitar dependencias y Desinstalar](./media/remove-application-insights/uninstall-core.png)

---

## <a name="what-is-created-when-you-add-application-insights"></a>Lo que se crea al agregar Application Insights

Cuando agrega Application Insights al proyecto, crea archivos y agrega código a algunos de sus archivos. La desinstalación exclusiva de los paquetes NuGet no siempre descartará los archivos y el código. Para quitar completamente Application Insights, debe comprobar y eliminar manualmente el código o los archivos agregados junto con cualquier llamada API que haya agregado en el proyecto.

# <a name="net"></a>[.NET](#tab/net)

Al agregar Telemetría de Application Insights a un proyecto ASP.NET de Visual Studio, se agregan los siguientes archivos:

- ApplicationInsights.config
- AiHandleErrorAttribute.cs

Se agregan los siguientes fragmentos de código:

- [Nombre del proyecto].csproj

    ```C#
     <ApplicationInsightsResourceId>/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/Default-ApplicationInsights-EastUS/providers/microsoft.insights/components/WebApplication4</ApplicationInsightsResourceId>
    ```

- Packages.config

    ```xml
    <packages>
    ...
    
      <package id="Microsoft.ApplicationInsights" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.Agent.Intercept" version="2.4.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.DependencyCollector" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.PerfCounterCollector" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.Web" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.WindowsServer" version="2.12.0" targetFramework="net472" />
      <package id="Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel" version="2.12.0" targetFramework="net472" />
    
      <package id="Microsoft.AspNet.TelemetryCorrelation" version="1.0.7" targetFramework="net472" />
    
      <package id="System.Buffers" version="4.4.0" targetFramework="net472" />
      <package id="System.Diagnostics.DiagnosticSource" version="4.6.0" targetFramework="net472" />
      <package id="System.Memory" version="4.5.3" targetFramework="net472" />
      <package id="System.Numerics.Vectors" version="4.4.0" targetFramework="net472" />
      <package id="System.Runtime.CompilerServices.Unsafe" version="4.5.2" targetFramework="net472" />
    ...
    </packages>
    ```

- Layout.cshtml

    Si el proyecto tiene un archivo Layout.cshtml, se agrega el código siguiente.
    
    ```html
    <head>
    ...
        <script type = 'text/javascript' >
            var appInsights=window.appInsights||function(config)
            {
                function r(config){ t[config] = function(){ var i = arguments; t.queue.push(function(){ t[config].apply(t, i)})} }
                var t = { config:config},u=document,e=window,o='script',s=u.createElement(o),i,f;for(s.src=config.url||'//az416426.vo.msecnd.net/scripts/a/ai.0.js',u.getElementsByTagName(o)[0].parentNode.appendChild(s),t.cookie=u.cookie,t.queue=[],i=['Event','Exception','Metric','PageView','Trace','Ajax'];i.length;)r('track'+i.pop());return r('setAuthenticatedUserContext'),r('clearAuthenticatedUserContext'),config.disableExceptionTracking||(i='onerror',r('_'+i),f=e[i],e[i]=function(config, r, u, e, o) { var s = f && f(config, r, u, e, o); return s !== !0 && t['_' + i](config, r, u, e, o),s}),t
            }({
                instrumentationKey:'00000000-0000-0000-0000-000000000000'
            });
            
            window.appInsights=appInsights;
            appInsights.trackPageView();
        </script>
    ...
    </head>
    ```

- ConnectedService.json

    ```json
    {
      "ProviderId": "Microsoft.ApplicationInsights.ConnectedService.ConnectedServiceProvider",
      "Version": "16.0.0.0",
      "GettingStartedDocument": {
        "Uri": "https://go.microsoft.com/fwlink/?LinkID=613413"
      }
    }
    ```

- FilterConfig.cs

    ```csharp
            public static void RegisterGlobalFilters(GlobalFilterCollection filters)
            {
                filters.Add(new ErrorHandler.AiHandleErrorAttribute());// This line was added
            }
    ```

# <a name="net-core"></a>[.NET Core](#tab/netcore)

Al agregar Telemetría de Application Insights a un proyecto de plantilla de ASP.NET Core de Visual Studio, se agregan los siguientes archivos:

- [Nombre del proyecto].csproj

    ```csharp
      <PropertyGroup>
        <TargetFramework>netcoreapp3.1</TargetFramework>
        <ApplicationInsightsResourceId>/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/Default-ApplicationInsights-EastUS/providers/microsoft.insights/components/WebApplication4core</ApplicationInsightsResourceId>
      </PropertyGroup>
    
      <ItemGroup>
        <PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.12.0" />
      </ItemGroup>
    
      <ItemGroup>
        <WCFMetadata Include="Connected Services" />
      </ItemGroup>
    ```

- Appsettings.json:

    ```json
    "ApplicationInsights": {
        "InstrumentationKey": "00000000-0000-0000-0000-000000000000"
    ```

- ConnectedService.json
    
    ```json
    {
      "ProviderId": "Microsoft.ApplicationInsights.ConnectedService.ConnectedServiceProvider",
      "Version": "16.0.0.0",
      "GettingStartedDocument": {
        "Uri": "https://go.microsoft.com/fwlink/?LinkID=798432"
      }
    }
    ```
- Startup.cs

    ```csharp
       public void ConfigureServices(IServiceCollection services)
            {
                services.AddRazorPages();
                services.AddApplicationInsightsTelemetry(); // This is added
            }
    ```

---

## <a name="next-steps"></a>Pasos siguientes

- [Azure Monitor](../overview.md)
