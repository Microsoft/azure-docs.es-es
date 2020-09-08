---
title: 'Tutorial: Uso de la configuración dinámica de App Configuration en ASP.NET Core'
titleSuffix: Azure App Configuration
description: En este tutorial aprenderá a actualizar dinámicamente los datos de configuración de aplicaciones de ASP.NET Core
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
ms.date: 02/24/2019
ms.author: lcozzens
ms.custom: devx-track-csharp, mvc
ms.openlocfilehash: b0435daee7f5bbd1435d5e69fc32f27c21e5e4a3
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89300296"
---
# <a name="tutorial-use-dynamic-configuration-in-an-aspnet-core-app"></a>Tutorial: Uso de la configuración dinámica en una aplicación de ASP.NET Core

ASP.NET Core dispone de un sistema de configuración conectable que puede leer datos de configuración de diversos orígenes. Es capaz de controlar los cambios de forma dinámica sin hacer que una aplicación se reinicie. ASP.NET Core admite el enlace de valores de configuración a clases .NET fuertemente tipadas. Estas clases las inserta en el código mediante los diversos patrones `IOptions<T>`. Uno de estos patrones, en concreto `IOptionsSnapshot<T>`, recarga automáticamente la configuración de la aplicación cuando cambian los datos subyacentes. Puede insertar `IOptionsSnapshot<T>` en controladores de la aplicación para acceder a la configuración más reciente almacenada en Azure App Configuration.

También puede configurar la biblioteca cliente de ASP.NET Core de App Configuration para que actualice dinámicamente un conjunto de ajustes de configuración mediante un middleware. Los valores de la configuración se actualizan con el almacén de configuración cada vez que la aplicación web recibe solicitudes.

App Configuration almacena en caché cada configuración para evitar demasiadas llamadas al almacén de configuración. La operación de actualización espera hasta que el valor de una opción almacenada en la caché expira y, después, actualiza ese valor, incluso aunque este haya cambiado ya en el almacén de configuración. El tiempo de expiración predeterminado de la memoria caché es de 30 segundos. Puede reemplazar este tiempo de expiración si es necesario.

Este tutorial le muestra cómo puede implementar las actualizaciones de configuración dinámica en el código. Se basa en la aplicación web que se introdujo en los inicios rápidos. Antes de continuar, finalice primero el tutorial [Creación de una aplicación ASP.NET Core con Azure App Configuration](./quickstart-aspnet-core-app.md).

Para realizar los pasos de este tutorial, puede usar cualquier editor de código. [Visual Studio Code](https://code.visualstudio.com/) es una excelente opción que está disponible en las plataformas Windows, macOS y Linux.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar la aplicación para actualizar su configuración en respuesta a los cambios en un almacén de App Configuration.
> * Insertar la configuración más reciente en los controladores de la aplicación

## <a name="prerequisites"></a>Prerrequisitos

Para realizar este tutorial, instale el [SDK de .NET Core](https://dotnet.microsoft.com/download).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Antes de continuar, finalice primero el tutorial [Creación de una aplicación ASP.NET Core con Azure App Configuration](./quickstart-aspnet-core-app.md).

## <a name="add-a-sentinel-key"></a>Adición de una clave de Sentinel

Una clave de *Sentinel* es una clave especial que se usa para indicar cuándo ha cambiado la configuración. La aplicación supervisa la clave de Sentinel en busca de cambios. Cuando se detecta un cambio, se actualizan todos los valores de configuración. Este enfoque reduce el número total de solicitudes que realiza la aplicación a App Configuration, en comparación con la supervisión de todas las claves en busca de cambios.

1. En Azure Portal, seleccione **Explorador de configuración > Crear > Clave-valor**.
1. En **Clave**, escriba *TestApp:Settings:Sentinel*. En **Valor**, escriba 1. Deje **Etiqueta** y **Tipo de contenido** en blanco.
1. Seleccione **Aplicar**.

> [!NOTE]
> Si no está utilizando una clave de Sentinel, deberá registrar manualmente cada clave que quiera ver.

## <a name="reload-data-from-app-configuration"></a>Recarga de datos de App Configuration

1. Para agregar una referencia al paquete NuGet `Microsoft.Azure.AppConfiguration.AspNetCore`, ejecute el comando siguiente:

    ```dotnetcli
    dotnet add package Microsoft.Azure.AppConfiguration.AspNetCore
    ```

1. Abra *Program.cs* y actualice el método `CreateWebHostBuilder` para agregar el método `config.AddAzureAppConfiguration()`.

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();

                config.AddAzureAppConfiguration(options =>
                {
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                           .ConfigureRefresh(refresh =>
                                {
                                    refresh.Register("TestApp:Settings:Sentinel", refreshAll: true)
                                           .SetCacheExpiration(new TimeSpan(0, 5, 0));
                                });
                });
            })
            .UseStartup<Startup>();
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
                webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
                {
                    var settings = config.Build();
                    config.AddAzureAppConfiguration(options =>
                    {
                        options.Connect(settings["ConnectionStrings:AppConfig"])
                               .ConfigureRefresh(refresh =>
                                    {
                                        refresh.Register("TestApp:Settings:Sentinel", refreshAll: true)
                                               .SetCacheExpiration(new TimeSpan(0, 5, 0));
                                    });
                    });
                })
            .UseStartup<Startup>());
    ```
    ---

    El método `ConfigureRefresh` se utiliza para especificar la configuración usada para actualizar los datos de configuración con el almacén de App Configuration cuando se desencadena una operación de actualización. El parámetro `refreshAll` del método `Register` indica que se deben actualizar todos los valores de configuración si cambia la clave de Sentinel.

    Además, el método `SetCacheExpiration` reemplaza el tiempo de expiración predeterminado de la memoria caché (30 segundos) y especifica un tiempo de 5 minutos en su lugar. Esto reduce el número de solicitudes que se realiza a App Configuration.

    > [!NOTE]
    > Puede que desee reducir el tiempo de expiración de la memoria caché con fines de prueba.

    Para desencadenar realmente una operación de actualización, es necesario configurar un middleware de actualización para que la aplicación actualice los datos de configuración cuando se produzca cualquier cambio. Puede ver cómo hacerlo en un paso posterior.

2. Agregue un archivo *Settings.cs* que defina e implemente una nueva clase `Settings`.

    ```csharp
    namespace TestAppConfig
    {
        public class Settings
        {
            public string BackgroundColor { get; set; }
            public long FontSize { get; set; }
            public string FontColor { get; set; }
            public string Message { get; set; }
        }
    }
    ```

3. Abra *Startup.cs* y use `IServiceCollection.Configure<T>` en el método `ConfigureServices` para enlazar los datos de configuración a la clase `Settings`.

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<Settings>(Configuration.GetSection("TestApp:Settings"));
        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
    }
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<Settings>(Configuration.GetSection("TestApp:Settings"));
        services.AddControllersWithViews();
    }
    ```
    ---
    > [!Tip]
    > Para más información sobre el patrón de opciones al leer valores de configuración, consulte  [Patrón de opciones en ASP.NET Core](https://docs.microsoft.com/aspnet/core/fundamentals/configuration/options?view=aspnetcore-3.1).

4. Actualice el método `Configure` para agregar un middleware `UseAzureAppConfiguration` que permita actualizar los valores de la configuración mientras que la aplicación web de ASP.NET Core sigue recibiendo solicitudes.


    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        app.UseAzureAppConfiguration();

        services.Configure<CookiePolicyOptions>(options =>
        {
            options.CheckConsentNeeded = context => true;
            options.MinimumSameSitePolicy = SameSiteMode.None;
        });

        app.UseMvc();
    }
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
                // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
                app.UseHsts();
            }

            // Add the following line:
            app.UseAzureAppConfiguration();

            app.UseHttpsRedirection();
            
            app.UseStaticFiles();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller=Home}/{action=Index}/{id?}");
            });
    }
    ```
    ---
    
    El middleware utiliza la configuración de actualización especificada en el método `AddAzureAppConfiguration` de `Program.cs` para desencadenar una actualización para cada solicitud recibida por la aplicación web de ASP.NET Core. Para cada solicitud, se desencadena una operación de actualización y la biblioteca cliente comprueba si el valor en caché de los ajustes de configuración registrados ha expirado. Si ha expirado, se actualiza.

## <a name="use-the-latest-configuration-data"></a>Uso de los datos de configuración más recientes

1. Abra *HomeController.cs* en el directorio de controladores y agregue una referencia al paquete `Microsoft.Extensions.Options`.

    ```csharp
    using Microsoft.Extensions.Options;
    ```

2. Actualice la clase `HomeController` para recibir `Settings` mediante la inserción de dependencias y asegúrese de usar sus valores.

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public class HomeController : Controller
    {
        private readonly Settings _settings;
        public HomeController(IOptionsSnapshot<Settings> settings)
        {
            _settings = settings.Value;
        }

        public IActionResult Index()
        {
            ViewData["BackgroundColor"] = _settings.BackgroundColor;
            ViewData["FontSize"] = _settings.FontSize;
            ViewData["FontColor"] = _settings.FontColor;
            ViewData["Message"] = _settings.Message;

            return View();
        }
    }
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```csharp
    public class HomeController : Controller
    {
        private readonly Settings _settings;
        private readonly ILogger<HomeController> _logger;

        public HomeController(ILogger<HomeController> logger, IOptionsSnapshot<Settings> settings)
        {
            _logger = logger;
            _settings = settings.Value;
        }

        public IActionResult Index()
        {
            ViewData["BackgroundColor"] = _settings.BackgroundColor;
            ViewData["FontSize"] = _settings.FontSize;
            ViewData["FontColor"] = _settings.FontColor;
            ViewData["Message"] = _settings.Message;

            return View();
        }

        // ...
    }
    ```
    ---



3. Abra el archivo *Index.cshtml* en el directorio Views > Home, y sustituya su contenido por el siguiente script:

    ```html
    <!DOCTYPE html>
    <html lang="en">
    <style>
        body {
            background-color: @ViewData["BackgroundColor"]
        }
        h1 {
            color: @ViewData["FontColor"];
            font-size: @ViewData["FontSize"]px;
        }
    </style>
    <head>
        <title>Index View</title>
    </head>
    <body>
        <h1>@ViewData["Message"]</h1>
    </body>
    </html>
    ```

## <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación en un entorno local

1. Para compilar la aplicación mediante la CLI de .NET Core, ejecute el siguiente comando en el shell de comandos:

    ```console
        dotnet build
    ```

1. Una vez que la compilación se haya realizado correctamente, ejecute el siguiente comando para ejecutar la aplicación web localmente:

    ```console
        dotnet run
    ```

1. Abra una ventana del explorador y vaya a la dirección URL que aparece en la salida `dotnet run`.

    ![Inicio local de la aplicación de inicio rápido](./media/quickstarts/aspnet-core-app-launch-local-before.png)

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Todos los recursos** y seleccione la instancia de almacén de App Configuration que creó en el inicio rápido.

1. Seleccione **Explorador de configuración** y actualice los valores de las claves siguientes:

    | Clave | Value |
    |---|---|
    | TestApp:Settings:BackgroundColor | green |
    | TestApp:Settings:FontColor | lightGray |
    | TestApp:Settings:Message | Datos de Azure App Configuration: ahora con actualizaciones directas |
    | TestApp:Settings:Sentinel | 2 |

1. Actualice la página del explorador para ver los nuevos valores de configuración. Puede que tenga que actualizar más de una vez para que se reflejen los cambios.

    ![Inicio local de la aplicación de inicio rápido actualizada](./media/quickstarts/aspnet-core-app-launch-local-after.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha habilitado la aplicación web de ASP.NET Core para actualizar dinámicamente la configuración a partir de App Configuration. Para aprender a usar una identidad administrada de Azure para simplificar el acceso a App Configuration, vaya al siguiente tutorial.

> [!div class="nextstepaction"]
> [Integración de identidades administradas](./howto-integrate-azure-managed-service-identity.md)
