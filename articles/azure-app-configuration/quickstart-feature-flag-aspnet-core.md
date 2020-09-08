---
title: Inicio rápido para agregar marcas de características a ASP.NET Core
description: Adición de marcas de características a aplicaciones de ASP.NET Core y su administración en Azure App Configuration
author: lisaguthrie
ms.service: azure-app-configuration
ms.custom: devx-track-csharp
ms.topic: quickstart
ms.date: 01/14/2020
ms.author: lcozzens
ms.openlocfilehash: 12b66dc173a8d3f93f97fb369ce03533299a65d7
ms.sourcegitcommit: 3bf69c5a5be48c2c7a979373895b4fae3f746757
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88235271"
---
# <a name="quickstart-add-feature-flags-to-an-aspnet-core-app"></a>Inicio rápido: Adición de marcas de características a una aplicación web de ASP.NET Core

En este inicio rápido, va a crear una implementación de un extremo a otro de administración de características en una aplicación de ASP.NET Core mediante Azure App Configuration. Podrá usar el servicio App Configuration para almacenar en una ubicación central todas las marcas de características y controlar sus estados. 

Las bibliotecas de administración de características de .NET Core amplían la plataforma con una compatibilidad completa con las marcas de características. Estas bibliotecas se compilan sobre el sistema de configuración de .NET Core. Se integran sin problemas con App Configuration a través de su proveedor de configuración de .NET Core.

## <a name="prerequisites"></a>Prerrequisitos

- Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)
- [SDK de .NET Core](https://dotnet.microsoft.com/download).

## <a name="create-an-app-configuration-store"></a>Creación de un almacén de App Configuration

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

6. Seleccione **Administrador de características** >  **+Agregar** para agregar una marca de características denominada `Beta`.

    > [!div class="mx-imgBorder"]
    > ![Habilitación de la marca de característica denominada Beta](media/add-beta-feature-flag.png)

    Deje `label` sin definir por ahora. Seleccione **Aplicar** para guardar la nueva marca de características.

## <a name="create-an-aspnet-core-web-app"></a>Cree una aplicación web ASP.NET Core

Utilice la [interfaz de la línea de comandos (CLI) de .NET Core](https://docs.microsoft.com/dotnet/core/tools/) para crear un proyecto de aplicación web de ASP.NET Core MVC. La ventaja de usar la CLI de .NET Core frente a Visual Studio es que está disponible en las plataformas Windows, macOS y Linux.

1. Cree una carpeta nueva para su proyecto. En este inicio rápido, lo llamaremos *TestFeatureFlags*.

1. En la nueva carpeta, ejecute el siguiente comando para crear un nuevo proyecto de aplicación web de ASP.NET Core MVC:

   ```    
   dotnet new mvc --no-https
   ```

## <a name="add-secret-manager"></a>Incorporación de Secret Manager

Para usar Secret Manager, agregue un elemento `UserSecretsId` al archivo *.csproj*.

1. Abra el archivo *.csproj*.

1.  Agregue un elemento `UserSecretsId` como se muestra aquí. Puede usar el mismo identificador único global, o bien puede reemplazar este valor por el suyo propio.

    > [!IMPORTANT]
    > `CreateHostBuilder` reemplaza a `CreateWebHostBuilder` en .NET Core 3.0.  Seleccione la sintaxis correcta en función de su entorno.

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```xml
    <Project Sdk="Microsoft.NET.Sdk.Web">

        <PropertyGroup>
            <TargetFramework>netcoreapp2.1</TargetFramework>
            <UserSecretsId>79a3edd0-2092-40a2-a04d-dcb46d5ca9ed</UserSecretsId>
        </PropertyGroup>

        <ItemGroup>
            <PackageReference Include="Microsoft.AspNetCore.App" />
            <PackageReference Include="Microsoft.AspNetCore.Razor.Design" Version="2.1.2" PrivateAssets="All" />
        </ItemGroup>

    </Project>
    ```

    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)

    ```xml
    <Project Sdk="Microsoft.NET.Sdk.Web">

        <PropertyGroup>
            <TargetFramework>netcoreapp3.1</TargetFramework>
            <UserSecretsId>79a3edd0-2092-40a2-a04d-dcb46d5ca9ed</UserSecretsId>
        </PropertyGroup>

    </Project>
    ```
    ---

1. Guarde el archivo *.csproj*.

La herramienta Secret Manager almacena información confidencial para el trabajo de desarrollo fuera de su árbol de proyecto. Este enfoque ayuda a evitar el uso compartido accidental de secretos de la aplicación en el código fuente.

> [!TIP]
> Para más información sobre Secret Manager, consulte [Almacenamiento seguro de secretos de aplicación en el desarrollo en ASP.NET Core](https://docs.microsoft.com/aspnet/core/security/app-secrets).

## <a name="connect-to-an-app-configuration-store"></a>Conexión a un almacén de App Configuration

1. Agregue una referencia a los paquetes NuGet `Microsoft.Azure.AppConfiguration.AspNetCore` y `Microsoft.FeatureManagement.AspNetCore`, para lo que debe ejecutar los siguientes comandos:

    ```dotnetcli
    dotnet add package Microsoft.Azure.AppConfiguration.AspNetCore
    dotnet add package Microsoft.FeatureManagement.AspNetCore
    ```

1. Ejecute el siguiente comando para restaurar los paquetes para el proyecto:

    ```dotnetcli
    dotnet restore
    ```

1. Agregue un secreto llamado **ConnectionStrings:AppConfig** a Secret Manager.

    Este secreto contiene la cadena de conexión necesaria para acceder a su almacén de App Configuration. Sustituya el valor `<your_connection_string>` en el comando siguiente por la cadena de conexión de su almacén de App Configuration. Puede encontrar la cadena de conexión de la clave de solo lectura principal en **Claves de acceso** en Azure Portal.

    Este comando debe ejecutarse en el mismo directorio que el archivo *.csproj*.

    ```dotnetcli
    dotnet user-secrets set ConnectionStrings:AppConfig <your_connection_string>
    ```

    Usará Secret Manager solo para probar la aplicación web localmente. Cuando se implementa la aplicación en [Azure App Service](https://azure.microsoft.com/services/app-service), por ejemplo, usará una configuración de la aplicación llamada **Connection Strings** (Cadenas de conexión) en App Service, en lugar de usar Secret Manager para almacenar la cadena de conexión.

    Puede acceder a este secreto con la API de App Configuration. Un signo de dos puntos (:) funciona en el nombre de configuración con la API de App Configuration en todas las plataformas compatibles. Consulte [Configuración en ASP.NET Core](https://docs.microsoft.com/aspnet/core/fundamentals/configuration).

1. En *Program.cs*, actualice el método `CreateWebHostBuilder` para usar App Configuration mediante la llamada al método `config.AddAzureAppConfiguration()`.

    > [!IMPORTANT]
    > `CreateHostBuilder` reemplaza a `CreateWebHostBuilder` en .NET Core 3.0.  Seleccione la sintaxis correcta en función de su entorno.

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)

    ```csharp
    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();
                config.AddAzureAppConfiguration(options => {
                    options.Connect(settings["ConnectionStrings:AppConfig"])
                        .UseFeatureFlags();
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
            config.AddAzureAppConfiguration(options => {
                options.Connect(settings["ConnectionStrings:AppConfig"])
                    .UseFeatureFlags();
            });
        })
        .UseStartup<Startup>());
    ```
    ---

1. Abra *Startup.cs* y agregue referencias al administrador de características de .NET Core.

    ```csharp
    using Microsoft.FeatureManagement;
    ```

1. Actualice el método `ConfigureServices` para agregar compatibilidad con la marca de características mediante una llamada al método `services.AddFeatureManagement()`. También tiene la opción de incluir cualquier filtro para usar con las marcas de características mediante la llamada a `services.AddFeatureFilter<FilterType>()`:

    #### <a name="net-core-2x"></a>[.NET Core 2.x](#tab/core2x)
    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);        
        services.AddFeatureManagement();
    }
    ```
    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)
    ```csharp    
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllersWithViews();
        services.AddFeatureManagement();
    }

    ---

1. Update the `Configure` method to add a middleware to allow the feature flag values to be refreshed at a recurring interval while the ASP.NET Core web app continues to receive requests.

    #### [.NET Core 2.x](#tab/core2x)
    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
            }

            app.UseStaticFiles();
            app.UseCookiePolicy();
            app.UseAzureAppConfiguration();
            app.UseMvc(routes =>
            {
                routes.MapRoute(
                    name: "default",
                    template: "{controller=Home}/{action=Index}/{id?}");
            });
    }
    ```
    #### <a name="net-core-3x"></a>[.NET Core 3.x](#tab/core3x)
    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
            }
            app.UseStaticFiles();
            app.UseRouting();
            app.UseAuthorization();
            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller=Home}/{action=Index}/{id?}");
            });
            app.UseAzureAppConfiguration();
    }
    ```
    ---

1. Agregue un archivo *MyFeatureFlags.cs*:

    ```csharp
    namespace TestFeatureFlags
    {
        public enum MyFeatureFlags
        {
            Beta
        }
    }
    ```

1. Agregue *BetaController.cs* al directorio *Controllers*:

    ```csharp
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.FeatureManagement;
    using Microsoft.FeatureManagement.Mvc;

    namespace TestFeatureFlags.Controllers
    {
        public class BetaController: Controller
        {
            private readonly IFeatureManager _featureManager;

            public BetaController(IFeatureManagerSnapshot featureManager)
            {
                _featureManager = featureManager;
            }

            [FeatureGate(MyFeatureFlags.Beta)]
            public IActionResult Index()
            {
                return View();
            }
        }
    }
    ```

1. Abra *_ViewImports.cshtml* en el directorio *Views* y agregue el asistente de etiquetas del administrador de características:

    ```html
    @addTagHelper *, Microsoft.FeatureManagement.AspNetCore
    ```

1. Abra *_Layout.cshtml* en el directorio *Views*\\*Shared* y reemplace el código de barras `<nav>` situado en `<body>` > `<header>` por el código siguiente:

    ```html
    <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
        <div class="container">
            <a class="navbar-brand" asp-area="" asp-controller="Home" asp-action="Index">TestFeatureFlags</a>
            <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse" aria-controls="navbarSupportedContent"
            aria-expanded="false" aria-label="Toggle navigation">
            <span class="navbar-toggler-icon"></span>
            </button>
            <div class="navbar-collapse collapse d-sm-inline-flex flex-sm-row-reverse">
                <ul class="navbar-nav flex-grow-1">
                    <li class="nav-item">
                        <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Index">Home</a>
                    </li>
                    <feature name="Beta">
                    <li class="nav-item">
                        <a class="nav-link text-dark" asp-area="" asp-controller="Beta" asp-action="Index">Beta</a>
                    </li>
                    </feature>
                    <li class="nav-item">
                        <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Privacy">Privacy</a>
                    </li>
                </ul>
            </div>
        </div>
    </nav>
    ```

1. Cree un directorio *Beta* en *Views* y agréguele *Index.cshtml*:

    ```html
    @{
        ViewData["Title"] = "Beta Home Page";
    }

    <h1>
        This is the beta website.
    </h1>
    ```

## <a name="build-and-run-the-app-locally"></a>Compilación y ejecución de la aplicación en un entorno local

1. Para compilar la aplicación mediante la CLI de .NET Core, ejecute el siguiente comando en el shell de comandos:

    ```
    dotnet build
    ```

1. Una vez que la compilación se haya realizado correctamente, ejecute el siguiente comando para ejecutar la aplicación web localmente:

    ```
    dotnet run
    ```

1. Inicie una ventana del explorador y vaya a `https://localhost:5000`, que es la dirección URL predeterminada de la aplicación web hospedada localmente.
    Si trabajando en Azure Cloud Shell, seleccione el botón *Vista previa web* y después *Configurar*.  Cuando se le pida, seleccione el puerto 5000.

    ![Busque el botón Vista previa web](./media/quickstarts/cloud-shell-web-preview.png)

    El explorador debe mostrar una página similar a la imagen siguiente.
    ![Inicio rápido sobre el inicio local de la aplicación](./media/quickstarts/aspnet-core-feature-flag-local-before.png)

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Seleccione **Todos los recursos** y seleccione la instancia de almacén de App Configuration que creó en el inicio rápido.

1. Seleccione **Feature Manager** (Administrador de características) y cambie el estado de la clave **Beta** a **On** (Activado).

1. Vuelva al símbolo del sistema y cancele el proceso de ejecución de `dotnet`; para ello, presione `Ctrl-C`.  Reinicie la aplicación mediante `dotnet run`.

1. Actualice la página del explorador para ver los nuevos valores de configuración.

    ![Inicio de la aplicación del artículo de inicio rápido en un entorno local](./media/quickstarts/aspnet-core-feature-flag-local-after.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un almacén de App Configuration y lo ha usado para administrar características en una aplicación web de ASP.NET Core mediante las [bibliotecas de administración de características](https://go.microsoft.com/fwlink/?linkid=2074664).

- Más información sobre la [administración de características](./concept-feature-management.md).
- [Administración de marcas de características](./manage-feature-flags.md).
- [Uso de marcas de características en una aplicación de ASP.NET Core](./use-feature-flags-dotnet-core.md).
- [Uso de la configuración dinámica en una aplicación de ASP.NET Core](./enable-dynamic-configuration-aspnet-core.md)
