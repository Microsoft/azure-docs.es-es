---
title: Tutorial para usar las marcas de características en una aplicación de .NET Core | Microsoft Docs
description: En este tutorial, obtendrá información sobre cómo implementar las marcas de características en aplicaciones de .NET Core
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: maiye
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 04/19/2019
ms.author: yegu
ms.custom: mvc
ms.openlocfilehash: 28ba4397ca5a5fd3c281555238fc7eec8a82943d
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65413666"
---
# <a name="tutorial-use-feature-flags-in-a-net-core-app"></a>Tutorial: Uso de marcas de características en una aplicación de .NET Core

Las bibliotecas de administración de características de .NET Core proporcionan soporte idiomático para implementar las marcas de características en una aplicación de .NET o ASP.NET Core. Le permiten agregar marcas de características al código de una forma más declarativa para que no sea necesario escribir todas las instrucciones `if` correspondientes manualmente. Administran los ciclos de vida de las marcas de características (por ejemplo, actualizan y almacenan en caché los estados de marca y garantizan que un estado de marca sea inmutable durante una llamada de solicitud) en segundo plano. Además, la biblioteca de ASP.NET Core ofrece integraciones listas para usar, como acciones de controlador de MVC, vistas, rutas y middleware.

El inicio rápido [Add feature flags to an ASP.NET Core app](./quickstart-feature-flag-aspnet-core.md) (Agregar marcas de características a una aplicación ASP.NET Core) muestra varias maneras de agregar las marcas de características en una aplicación ASP.NET Core. Este tutorial explica esto con más detalles. Consulte la [documentación de administración de características de ASP.NET Core](https://go.microsoft.com/fwlink/?linkid=2091410) para obtener una referencia completa.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregar marcas de características en las partes clave de la aplicación para controlar la disponibilidad de características.
> * Integrar con App Configuration cuando se usa para administrar las marcas de características.

## <a name="setup"></a>Configuración

El administrador de características de .NET Core `IFeatureManager` obtiene las marcas de características del sistema de configuración nativo del marco. Como resultado, puede definir marcas de características de la aplicación con cualquier origen de configuración que admita .NET Core, incluido el archivo local *appsettings.json* o las variables de entorno. El administrador de características se basa en la inserción de dependencias de .NET Core. Puede registrar los servicios de administración de características mediante las convenciones estándar.

```csharp
using Microsoft.FeatureManagement;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement();
    }
}
```

El administrador de características recupera las marcas de características de la sección "FeatureManagement" de los datos de configuración de .NET Core de forma predeterminada. En el ejemplo siguiente se le indica que lea de una sección diferente que se llama "MyFeatureFlags" en su lugar.

```csharp
using Microsoft.FeatureManagement;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement(options =>
        {
                options.UseConfiguration(Configuration.GetSection("MyFeatureFlags"));
        });
    }
}
```

Si usa el filtro en sus marcas de características, deberá incluir una biblioteca adicional y registrarla. El ejemplo siguiente muestra cómo usar un filtro de características integrado que se llama **"PercentageFilter"**.

```csharp
using Microsoft.FeatureManagement;
using Microsoft.FeatureManagement.FeatureFilters;

public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddFeatureManagement()
                .AddFeatureFilter<PercentageFilter>();
    }
}
```

Para que funcione de forma eficaz, debe mantener las marcas de características fuera de la aplicación y administrarlas por separado. De este modo, podrá modificar los estados de las marcas en cualquier momento y hacer que estos cambios surtan efecto en la aplicación de inmediato. App Configuration proporciona un lugar centralizado para organizar y controlar todas las marcas de características a través de una IU de portal dedicada y ofrece las marcas a la aplicación directamente a través de sus bibliotecas de cliente de .NET Core. La manera más fácil de conectar la aplicación de ASP.NET Core con App Configuration es a través del proveedor de configuración `Microsoft.Extensions.Configuration.AzureAppConfiguration`. Puede usar este paquete de NuGet en el código agregando lo siguiente al archivo *Program.cs*:

```csharp
using Microsoft.Extensions.Configuration.AzureAppConfiguration;

public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
           .ConfigureAppConfiguration((hostingContext, config) => {
               var settings = config.Build();
               config.AddAzureAppConfiguration(options => {
                   options.Connect(settings["ConnectionStrings:AppConfig"])
                          .UseFeatureFlags();
                });
           })
           .UseStartup<Startup>();
```

Los valores de las marcas de características se esperan que cambien con el tiempo. De forma predeterminada, el administrador de características actualizará los valores de las marcas de características cada 30 segundos. Puede usar un intervalo de sondeo distinto en la llamada `options.UseFeatureFlags()` anterior.

```csharp
config.AddAzureAppConfiguration(options => {
    options.Connect(settings["ConnectionStrings:AppConfig"])
           .UseFeatureFlags(featureFlagOptions => {
                featureFlagOptions.PollInterval = TimeSpan.FromSeconds(5);
           });
});
```

## <a name="feature-flag-declaration"></a>Declaración de la marca de características

Cada marca de características tiene dos partes: un nombre y una lista de uno o varios filtros que se utilizan para evaluar si el estado de la característica está *activo* (es decir, cuando su valor es `True`). Un filtro define un caso de uso para el que debe activarse una característica. Si una marca de características tiene varios filtros, la lista de filtros se recorre en orden hasta que uno de los filtros determina que se debe habilitar la característica. En este momento, se considera la marca de característica está *activa* y se omiten los resultados del filtro restantes. Si ningún filtro indica que se debe habilitar la característica, la marca de características está *desactivada*.

El administrador de la característica admite *appsettings.json* como origen de configuración para las marcas de características. El ejemplo siguiente muestra cómo establecer las marcas de características en un archivo json.

```JSON
"FeatureManagement": {
    "FeatureX": true, // Feature flag set to on
    "FeatureY": false, // Feature flag set to off
    "FeatureC": {
        "EnabledFor": [
            {
                "Name": "Percentage",
                "Parameters": {
                    "Value": 50
                }
            }
        ]
    }
}
```

Por convención, la sección `FeatureManagement` de este documento json se usa para la configuración de la marca de características. El ejemplo anterior muestra tres marcas de características con los filtros definidos en la propiedad *EnabledFor*:

* **FeatureA** está en estado *on* (activo).
* **FeatureA** está en estado *off* (desactivado).
* **FeatureC** especifica un filtro denominado *Percentage* (Porcentaje) con una propiedad *Parameters* (Parámetros). *Percentage* es un ejemplo de un filtro configurable y especifica una probabilidad del 50 % para que **FeatureC** esté en estado *on*.

## <a name="referencing"></a>Referencia

Aunque no es necesario, las marcas de características deben definirse como variables `enum` de modo que se pueda hacer referencia a ellas con facilidad en el código.

```csharp
public enum MyFeatureFlags
{
    FeatureA,
    FeatureB,
    FeatureC
}
```

## <a name="feature-flag-check"></a>Comprobación de la marca de características

El patrón básico de administración de características consiste en comprobar primero si una marca de características está configurada en estado *on* y, a continuación, realizar las acciones asociadas si ese es el caso.

```csharp
IFeatureManager featureManager;
...
if (featureManager.IsEnabled(nameof(MyFeatureFlags.FeatureA)))
{
    // Run the following code
}
```

## <a name="dependency-injection"></a>Inserción de dependencia

En ASP.NET Core MVC, el administrador de características `IFeatureManager` puede obtenerse a través de la inserción de dependencias.

```csharp
public class HomeController : Controller
{
    private readonly IFeatureManager _featureManager;

    public HomeController(IFeatureManager featureManager)
    {
        _featureManager = featureManager;
    }
}
```

## <a name="controller-action"></a>Acción del controlador

En los controladores MVC, se puede usar un atributo `Feature` para controlar si se habilita una clase de controlador completo o una acción específica. El siguiente controlador `HomeController` requiere que *FeatureA* esté en estado *on* para ejecutar cualquier acción que contiene.

```csharp
[Feature(MyFeatureFlags.FeatureA)]
public class HomeController : Controller
{
    ...
}
```

La siguiente acción `Index` requiere que *FeatureA* esté en estado *on* para ejecutarse.

```csharp
[Feature(MyFeatureFlags.FeatureA)]
public IActionResult Index()
{
    return View();
}
```

Cuando una acción o controlador MVC está bloqueado porque la marca de característica de control está en estado *off*, se llama a un valor `IDisabledFeatureHandler` registrado. El valor predeterminado `IDisabledFeatureHandler` devuelve un código de estado 404 al cliente sin cuerpo de respuesta.

## <a name="view"></a>Ver

En las vistas de MVC, se puede usar una etiqueta `<feature>` para representar el contenido en función de si una marca de características está habilitada o no.

```html
<feature name="FeatureA">
    <p>This can only be seen if 'FeatureA' is enabled.</p>
</feature>
```

## <a name="mvc-filter"></a>Filtro MVC

Los filtros MVC se pueden configurar de manera que se activen en función del estado de una marca de características. Lo siguiente agrega un filtro MVC denominado `SomeMvcFilter`. Este filtro se desencadena en la canalización de MVC solo si está habilitado *FeatureA*.

```csharp
using Microsoft.FeatureManagement.FeatureFilters;

IConfiguration Configuration { get; set;}

public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc(options => {
        options.Filters.AddForFeature<SomeMvcFilter>(nameof(MyFeatureFlags.FeatureA));
    });
}
```

## <a name="route"></a>Enrutar

Las rutas se pueden exponer de forma dinámica según las marcas de características. Lo siguiente agrega una ruta, que establece `Beta` como controlador predeterminado, solo cuando *FeatureA* está habilitado.

```csharp
app.UseMvc(routes => {
    routes.MapRouteForFeature(nameof(MyFeatureFlags.FeatureA), "betaDefault", "{controller=Beta}/{action=Index}/{id?}");
});
```

## <a name="middleware"></a>Software intermedio

Las marcas de características pueden utilizarse para agregar middleware y ramas de aplicación condicionalmente. Lo siguiente inserta un componente de middleware en la canalización de la solicitud solo cuando *FeatureA* está habilitado.

```csharp
app.UseMiddlewareForFeature<ThirdPartyMiddleware>(nameof(MyFeatureFlags.FeatureA));
```

Esto desarrolla la capacidad más genérica de bifurcar toda la aplicación en función de una marca de características.

```csharp
app.UseForFeature(featureName, appBuilder => {
    appBuilder.UseMiddleware<T>();
});
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a implementar marcas de características en la aplicación ASP.NET Core mediante el uso de la bibliotecas `Microsoft.FeatureManagement`. Consulte los siguientes recursos para obtener más información sobre la compatibilidad de administración de características en ASP.NET Core y App Configuration.

* [ASP.NET Core feature flag sample code]() (Código de ejemplo de marca de características de ASP.NET Core)
* [Microsoft.FeatureManagement documentation]() (Documentación de Microsoft.FeatureManagement)
* [Administración de marcas de características](./manage-feature-flags.md)
