---
title: Tutorial sobre el uso de la configuración dinámica de Azure App Configuration en una aplicación de ASP.NET Core | Microsoft Docs
description: En este tutorial aprenderá a actualizar dinámicamente los datos de configuración de aplicaciones de ASP.NET Core
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: balans
editor: ''
ms.assetid: ''
ms.service: azure-app-configuration
ms.workload: tbd
ms.devlang: csharp
ms.topic: tutorial
ms.date: 02/24/2019
ms.author: yegu
ms.custom: mvc
ms.openlocfilehash: cae29fe045d1bdc17f414ff016642635b74320df
ms.sourcegitcommit: 6f043a4da4454d5cb673377bb6c4ddd0ed30672d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65408832"
---
# <a name="tutorial-use-dynamic-configuration-in-an-aspnet-core-app"></a>Tutorial: Uso de la configuración dinámica en una aplicación de ASP.NET Core

ASP.NET Core dispone de un sistema de configuración conectable que puede leer datos de configuración de diversos orígenes. Es capaz de controlar los cambios sobre la marcha sin hacer que una aplicación se reinicie. ASP.NET Core admite el enlace de valores de configuración a clases .NET fuertemente tipadas. Estas clases las inserta en el código mediante los diversos patrones `IOptions<T>`. Uno de estos patrones, en concreto `IOptionsSnapshot<T>`, recarga automáticamente la configuración de la aplicación cuando cambian los datos subyacentes.

Puede insertar `IOptionsSnapshot<T>` en controladores de la aplicación para acceder a la configuración más reciente almacenada en Azure App Configuration. Además, puede configurar la biblioteca cliente de ASP.NET Core para App Configuration para que supervise continuamente y recupere cualquier cambio en un almacén de App Configuration. El usuario define el intervalo periódico de sondeo.

Este tutorial le muestra cómo puede implementar las actualizaciones de configuración dinámica en el código. Se basa en la aplicación web que se introdujo en los inicios rápidos. Antes de continuar, finalice primero el tutorial [Creación de una aplicación ASP.NET Core con Azure App Configuration](./quickstart-aspnet-core-app.md).

Para realizar los pasos de este tutorial, puede usar cualquier editor de código. [Visual Studio Code](https://code.visualstudio.com/) es una excelente opción que está disponible en las plataformas Windows, macOS y Linux.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar la aplicación para actualizar su configuración en respuesta a los cambios en un almacén de App Configuration
> * Insertar la configuración más reciente en los controladores de la aplicación

## <a name="prerequisites"></a>Requisitos previos

Para realizar este tutorial, instale el [SDK de .NET Core](https://dotnet.microsoft.com/download).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="reload-data-from-app-configuration"></a>Recarga de datos de App Configuration

1. Abra *Program.cs* y actualice el método `CreateWebHostBuilder` mediante la incorporación del método `config.AddAzureAppConfiguration()`.

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();
                config.AddAzureAppConfiguration(options =>
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                           .Watch("TestApp:Settings:BackgroundColor")
                           .Watch("TestApp:Settings:FontColor")
                           .Watch("TestApp:Settings:Message"));
            })
            .UseStartup<Startup>();
    ```

    El segundo parámetro del método `.Watch` es el intervalo de sondeo en el que la biblioteca cliente de ASP.NET consulta un almacén de App Configuration. La biblioteca cliente comprueba la configuración específica para ver si se ha producido algún cambio.

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

3. Abra *Startup.cs* y actualice el método `ConfigureServices` para enlazar los datos de configuración a la clase `Settings`.

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.Configure<Settings>(Configuration.GetSection("TestApp:Settings"));

        services.Configure<CookiePolicyOptions>(options =>
        {
            options.CheckConsentNeeded = context => true;
            options.MinimumSameSitePolicy = SameSiteMode.None;
        });

        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
    }
    ```

## <a name="use-the-latest-configuration-data"></a>Uso de los datos de configuración más recientes

1. Abra *HomeController.cs* en el directorio de controladores y agregue una referencia al paquete `Microsoft.Extensions.Options`.

    ```csharp
    using Microsoft.Extensions.Options;
    ```

2. Actualice la clase `HomeController` para recibir `Settings` mediante la inserción de dependencias y asegúrese de usar sus valores.

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
            font-size: @ViewData["FontSize"];
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

        dotnet build

2. Una vez que la compilación se haya realizado correctamente, ejecute el siguiente comando para ejecutar la aplicación web localmente:

        dotnet run

3. Inicie una ventana del explorador y vaya a `http://localhost:5000`, que es la dirección URL predeterminada de la aplicación web hospedada localmente.

    ![Inicio de la aplicación del artículo de inicio rápido en un entorno local](./media/quickstarts/aspnet-core-app-launch-local-before.png)

4. Inicie sesión en el [Azure Portal](https://aka.ms/azconfig/portal). Seleccione **Todos los recursos**y seleccione la instancia de almacén de App Configuration que creó en el inicio rápido.

5. Seleccione **Explorador de configuración** y actualice los valores de las claves siguientes:

    | Clave | Valor |
    |---|---|
    | TestAppSettings:BackgroundColor | green |
    | TestAppSettings:FontColor | lightGray |
    | TestAppSettings:Message | Datos de Azure App Configuration: ahora con actualizaciones directas |

6. Actualice la página del explorador para ver los nuevos valores de configuración.

    ![Inicio rápido de actualización de la aplicación en el entorno local](./media/quickstarts/aspnet-core-app-launch-local-after.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha agregado una identidad de servicio administrada de Azure para optimizar el acceso a App Configuration y mejorar la administración de credenciales de su aplicación. Para más información sobre App Configuration, continúe con los ejemplos de la CLI de Azure.

> [!div class="nextstepaction"]
> [Ejemplos de CLI](./cli-samples.md)
