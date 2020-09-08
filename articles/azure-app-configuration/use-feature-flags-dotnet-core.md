---
title: Tutorial para usar las marcas de características en una aplicación de .NET Core | Microsoft Docs
description: En este tutorial, aprenderá a implementar las marcas de características en aplicaciones de .NET Core.
services: azure-app-configuration
documentationcenter: ''
author: lisaguthrie
manager: maiye
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 08/12/2020
ms.author: lcozzens
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: 3f8a43a1ff28206a4bcc5fd059f69492c83eb34d
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88224720"
---
# <a name="tutorial-use-feature-flags-in-an-aspnet-core-app"></a>Tutorial: Uso de marcas de características en una aplicación de ASP.NET Core

Las bibliotecas de administración de características de .NET Core proporcionan soporte idiomático para implementar las marcas de características en una aplicación de .NET o ASP.NET Core. Estas bibliotecas le permiten agregar mediante declaración marcas de características al código para que no sea necesario escribir todas las instrucciones `if` correspondientes manualmente.

Las bibliotecas de administración de características también administran los ciclos de vida de las marcas de características en segundo plano. Por ejemplo, actualizan y almacenan en caché los estados de marca y garantizan que un estado de marca sea inmutable durante una llamada de solicitud. Además, la biblioteca de ASP.NET Core ofrece integraciones listas para usar, como acciones de controlador de MVC, vistas, rutas y middleware.

El inicio rápido [Add feature flags to an ASP.NET Core app](./quickstart-feature-flag-aspnet-core.md) (Agregar marcas de características a una aplicación ASP.NET Core) muestra varias maneras de agregar las marcas de características en una aplicación ASP.NET Core. Este tutorial explica estos métodos en profundidad. Consulte la [documentación de administración de características de ASP.NET Core](https://go.microsoft.com/fwlink/?linkid=2091410) para obtener una referencia completa.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Agregar marcas de características en las partes clave de la aplicación para controlar la disponibilidad de características.
> * Integrar con App Configuration cuando se usa para administrar las marcas de características.

## <a name="set-up-feature-management"></a>Configuración de la administración de características

Agregue una referencia a los paquetes NuGet `Microsoft.FeatureManagement.AspNetCore` y `Microsoft.FeatureManagement` para que utilicen el administrador de características de .NET Core.
    
El administrador de características de .NET Core `IFeatureManager` obtiene las marcas de características del sistema de configuración nativo del marco. Como resultado, puede definir marcas de características de la aplicación con cualquier origen de configuración que admita .NET Core, incluido el archivo local *appsettings.json* o las variables de entorno. `IFeatureManager` se basa en la inserción de dependencias de .NET Core. Puede registrar los servicios de administración de características mediante las convenciones estándar:

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

De forma predeterminada, el administrador de características recupera las marcas de características de la sección `"FeatureManagement"` de los datos de configuración de .NET Core. En el ejemplo siguiente se le indica al administrador de características que lea de una sección diferente que se llama `"MyFeatureFlags"` en su lugar:

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

Si usa el filtros en sus marcas de características, deberá incluir una biblioteca adicional y registrarla. El ejemplo siguiente muestra cómo usar un filtro de características integrado que se llama `PercentageFilter`:

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

Es recomendable que mantenga las marcas de características fuera de la aplicación y las administre por separado. De este modo, podrá modificar los estados de las marcas en cualquier momento y hacer que estos cambios surtan efecto en la aplicación de inmediato. App Configuration proporciona un lugar centralizado para organizar y controlar todas las marcas de características mediante la interfaz de usuario de un portal dedicado. App Configuration proporciona también las marcas a la aplicación directamente a través de sus bibliotecas cliente de .NET Core.

La manera más fácil de conectar la aplicación de ASP.NET Core con App Configuration es a través del proveedor de configuración `Microsoft.Azure.AppConfiguration.AspNetCore`. Siga estos pasos para usar este paquete NuGet.

1. Abra el archivo *Program.cs* y agregue el código siguiente.

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

2. Abra *Startup.cs* y actualice el método `Configure` para agregar un middleware que permita actualizar los valores de marca de la característica en un intervalo periódico mientras la aplicación web de ASP.NET Core sigue recibiendo solicitudes.

   ```csharp
   public void Configure(IApplicationBuilder app, IHostingEnvironment env)
   {
       app.UseAzureAppConfiguration();
       app.UseMvc();
   }
   ```

Los valores de las marcas de características se esperan que cambien con el tiempo. De forma predeterminada, los valores de la marca de característica se almacenan en caché durante un período de 30 segundos, por lo que una operación de actualización desencadenada cuando el middleware recibe una solicitud no actualizará el valor hasta que expire el valor almacenado en caché. El código siguiente muestra cómo cambiar la hora de expiración de caché o el intervalo de sondeo a 5 minutos en la llamada `options.UseFeatureFlags()`.

```csharp
config.AddAzureAppConfiguration(options => {
    options.Connect(settings["ConnectionStrings:AppConfig"])
           .UseFeatureFlags(featureFlagOptions => {
                featureFlagOptions.CacheExpirationTime = TimeSpan.FromMinutes(5);
           });
});
```

## <a name="feature-flag-declaration"></a>Declaración de la marca de características

Cada marca de características tiene dos partes: un nombre y una lista de uno o varios filtros que se utilizan para evaluar si el estado de la característica está *activo* (es decir, cuando su valor es `True`). Un filtro define un caso de uso en el que debe activarse una característica.

Cuando una marca de características tiene varios filtros, la lista de filtros se recorre en orden hasta que uno de los filtros determina que se debe habilitar la característica. En ese momento, la marca de característica está *activa* y se omiten los resultados del filtro restantes. Si ningún filtro indica que se debe habilitar la característica, la marca de características está *desactivada*.

El administrador de la característica admite *appsettings.json* como origen de configuración para las marcas de características. El ejemplo siguiente muestra cómo establecer las marcas de características en un archivo JSON:

```JSON
"FeatureManagement": {
    "FeatureA": true, // Feature flag set to on
    "FeatureB": false, // Feature flag set to off
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

Por convención, la sección `FeatureManagement` de este documento JSON se usa para la configuración de la marca de características. El ejemplo anterior muestra tres marcas de características con los filtros definidos en la propiedad `EnabledFor`:

* `FeatureA` está *activada*.
* `FeatureB` está *desactivada*.
* `FeatureC` especifica un filtro denominado `Percentage` con una propiedad `Parameters`. `Percentage` es un filtro configurable. En este ejemplo, `Percentage` especifica una probabilidad del 50 por ciento de que la marca `FeatureC` esté *activada*.

## <a name="feature-flag-references"></a>Referencias de marcas de características

Para que pueda hacer referencia fácilmente a las marcas de características en el código, deberá definirlas como variables `enum`:

```csharp
public enum MyFeatureFlags
{
    FeatureA,
    FeatureB,
    FeatureC
}
```

## <a name="feature-flag-checks"></a>Comprobaciones de las marcas de características

El patrón básico de administración de características consiste en comprobar primero si una marca de características está *activada*. Si es así, el administrador de características ejecutará las acciones que contiene la característica. Por ejemplo:

```csharp
IFeatureManager featureManager;
...
if (await featureManager.IsEnabledAsync(nameof(MyFeatureFlags.FeatureA)))
{
    // Run the following code
}
```

## <a name="dependency-injection"></a>Inserción de dependencia

En ASP.NET Core MVC, puede acceder al administrador de características `IFeatureManager` mediante la inserción de dependencias:

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

## <a name="controller-actions"></a>Acciones de controlador

En los controladores MVC, puede usar un atributo `FeatureGate` para controlar si se habilita una clase de controlador completa o una acción específica. El siguiente controlador `HomeController` requiere que `FeatureA` esté *activada* para ejecutar cualquier acción que contenga la clase del contenedor:

```csharp
using Microsoft.FeatureManagement.Mvc;

[FeatureGate(MyFeatureFlags.FeatureA)]
public class HomeController : Controller
{
    ...
}
```

La siguiente acción `Index` requiere que `FeatureA` esté *activada* para ejecutarse:

```csharp
using Microsoft.FeatureManagement.Mvc;

[FeatureGate(MyFeatureFlags.FeatureA)]
public IActionResult Index()
{
    return View();
}
```

Cuando una acción o controlador MVC está bloqueado porque la marca de característica de control está *desactivada*, se llama a una interfaz `IDisabledFeaturesHandler` registrada. La interfaz predeterminada `IDisabledFeaturesHandler` devuelve un código de estado 404 al cliente sin cuerpo de respuesta.

## <a name="mvc-views"></a>Vistas de MVC

En las vistas de MVC, se puede usar una etiqueta `<feature>` para representar el contenido en función de si una marca de características está habilitada:

```html
<feature name="FeatureA">
    <p>This can only be seen if 'FeatureA' is enabled.</p>
</feature>
```

Para mostrar contenido alternativo cuando no se cumplen los requisitos, puede usar el atributo `negate`.

```html
<feature name="FeatureA" negate="true">
    <p>This will be shown if 'FeatureA' is disabled.</p>
</feature>
```

También se puede usar la etiqueta `<feature>` de la característica para mostrar el contenido si están habilitadas todas o alguna de las características de una lista.

```html
<feature name="FeatureA, FeatureB" requirement="All">
    <p>This can only be seen if 'FeatureA' and 'FeatureB' are enabled.</p>
</feature>
<feature name="FeatureA, FeatureB" requirement="Any">
    <p>This can be seen if 'FeatureA', 'FeatureB', or both are enabled.</p>
</feature>
```

## <a name="mvc-filters"></a>Filtros MVC

Los filtros MVC se pueden configurar de manera que se activen en función del estado de una marca de características. El código siguiente agrega un filtro MVC denominado `SomeMvcFilter`. Este filtro se desencadena en la canalización de MVC solo si está habilitada `FeatureA`. Esta funcionalidad está limitada a `IAsyncActionFilter`. 

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

## <a name="middleware"></a>Software intermedio

También puede usar las marcas de características para agregar condicionalmente ramas de aplicación y middleware. El código siguiente inserta un componente de middleware en la canalización de la solicitud solo cuando `FeatureA` está habilitada:

```csharp
app.UseMiddlewareForFeature<ThirdPartyMiddleware>(nameof(MyFeatureFlags.FeatureA));
```

Este código desarrolla una funcionalidad más genérica de crear una rama de toda la aplicación en función de una marca de características:

```csharp
app.UseForFeature(featureName, appBuilder => {
    appBuilder.UseMiddleware<T>();
});
```

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a implementar marcas de características en la aplicación ASP.NET Core mediante el uso de bibliotecas `Microsoft.FeatureManagement`. Consulte los siguientes recursos para más información sobre la compatibilidad de la administración de características en ASP.NET Core y App Configuration:

* [ASP.NET Core feature flag sample code](/azure/azure-app-configuration/quickstart-feature-flag-aspnet-core) (Código de ejemplo de marca de características de ASP.NET Core)
* [Microsoft.FeatureManagement documentation](https://docs.microsoft.com/dotnet/api/microsoft.featuremanagement) (Documentación de Microsoft.FeatureManagement)
* [Administración de marcas de características](./manage-feature-flags.md)
