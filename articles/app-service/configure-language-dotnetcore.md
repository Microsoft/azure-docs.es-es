---
title: Configuración de aplicaciones de ASP.NET Core
description: Aprenda a configurar una aplicación de ASP.NET Core en las instancias nativas de Windows, o en un contenedor de Linux pregenerado, en Azure App Service. En este artículo se muestran las tareas de configuración más comunes.
ms.devlang: dotnet
ms.custom: devx-track-csharp
ms.topic: article
ms.date: 06/02/2020
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: 3456adc2b143f1f51115183fe4873938d067d267
ms.sourcegitcommit: 648c8d250106a5fca9076a46581f3105c23d7265
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "88961676"
---
# <a name="configure-an-aspnet-core-app-for-azure-app-service"></a>Configuración de una aplicación de ASP.NET para Azure App Service

> [!NOTE]
> Para ASP.NET en .NET Framework, consulte [Configurar una aplicación de ASP.NET para Azure App Service](configure-language-dotnet-framework.md)

Las aplicaciones de ASP.NET Core deben implementarse en Azure App Service como binarios compilados. La herramienta de publicación de Visual Studio compila la solución y, a continuación, implementa los binarios compilados directamente, mientras que el motor de implementación de App Service implementa el repositorio de código en primer lugar y, a continuación, compila los binarios.

Esta guía incluye conceptos clave e instrucciones para los desarrolladores de ASP.NET Core. Si nunca ha usado Azure App Service, siga primero la [guía de inicio rápido de ASP.NET Core](quickstart-dotnetcore.md) y el [tutorial de ASP.NET Core con SQL Database](tutorial-dotnetcore-sqldb-app.md).

::: zone pivot="platform-windows"  

## <a name="show-supported-net-core-runtime-versions"></a>Mostrar las versiones del entorno de ejecución de .NET Core

En App Service, las instancias de Windows ya tienen instaladas todas las versiones compatibles de .NET Core. Para mostrar las versiones del SDK y .NET Core Runtime disponibles para usted, vaya a `https://<app-name>.scm.azurewebsites.net/DebugConsole` y ejecute el siguiente comando en la consola basada en el explorador:

```azurecli-interactive
dotnet --info
```

::: zone-end

::: zone pivot="platform-linux"

## <a name="show-net-core-version"></a>Mostrar la versión de .NET Core

Para mostrar la versión actual de .NET Core, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query linuxFxVersion
```

Para mostrar todas las versiones compatibles de .NET Core, ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp list-runtimes --linux | grep DOTNETCORE
```

::: zone-end

## <a name="set-net-core-version"></a>Establecer la versión de .NET Core

::: zone pivot="platform-windows"  

Establezca el marco de destino en el archivo de proyecto para el proyecto de ASP.NET Core. Para obtener más información, consulte [Selección de la versión de .NET Core que se va a usar](/dotnet/core/versions/selection) en la documentación de .NET Core.

::: zone-end

::: zone pivot="platform-linux"

Ejecute el siguiente comando en [Cloud Shell](https://shell.azure.com) para establecer la versión 3.1 de .NET Core:

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <resource-group-name> --linux-fx-version "DOTNETCORE|3.1"
```

::: zone-end

::: zone pivot="platform-linux"

## <a name="customize-build-automation"></a>Personalización de la automatización de compilaciones

Si implementa la aplicación utilizando paquetes Git o zip con la automatización de compilaciones activada, la automatización de compilaciones de App Service se ejecutará en este orden:

1. Se ejecuta un script personalizado, si se especifica en `PRE_BUILD_SCRIPT_PATH`.
1. Se ejecuta `dotnet restore` para restaurar las dependencias de NuGet.
1. Se ejecuta `dotnet publish` para compilar un archivo binario para producción.
1. Se ejecuta un script personalizado, si se especifica en `POST_BUILD_SCRIPT_PATH`.

`PRE_BUILD_COMMAND` y `POST_BUILD_COMMAND` son variables de entorno que, de forma predeterminada, están vacías. Para ejecutar comandos anteriores a la compilación, defina `PRE_BUILD_COMMAND`. Para ejecutar comandos posteriores a la compilación, defina `POST_BUILD_COMMAND`.

En el ejemplo siguiente se especifican las dos variables para una serie de comandos, separados por comas.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings PRE_BUILD_COMMAND="echo foo, scripts/prebuild.sh"
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings POST_BUILD_COMMAND="echo foo, scripts/postbuild.sh"
```

Si desea conocer otras variables de entorno para personalizar la automatización de compilaciones, consulte este artículo sobre la [configuración de Oryx](https://github.com/microsoft/Oryx/blob/master/doc/configuration.md).

Para más información acerca de cómo se ejecuta App Service y se compilan aplicaciones de ASP.NET Core en Linux, consulte el siguiente artículo de la [documentación de Oryx sobre la detección y compilación de aplicaciones de .NET Core](https://github.com/microsoft/Oryx/blob/master/doc/runtimes/dotnetcore.md).

::: zone-end

## <a name="access-environment-variables"></a>Acceso a variables de entorno

En App Service, puede [establecer la configuración de la aplicación](configure-common.md#configure-app-settings) fuera del código de la aplicación. Después, puede tener acceso a ella en cualquier clase mediante el patrón de inserción de dependencias de ASP.NET Core estándar:

```csharp
using Microsoft.Extensions.Configuration;

namespace SomeNamespace 
{
    public class SomeClass
    {
        private IConfiguration _configuration;
    
        public SomeClass(IConfiguration configuration)
        {
            _configuration = configuration;
        }
    
        public SomeMethod()
        {
            // retrieve nested App Service app setting
            var myHierarchicalConfig = _configuration["My:Hierarchical:Config:Data"];
            // retrieve App Service connection string
            var myConnString = _configuration.GetConnectionString("MyDbConnection");
        }
    }
}
```

Si configura una opción de aplicación con el mismo nombre en App Service y en *appsettings.json*, por ejemplo, el valor de App Service tiene prioridad sobre el valor de *appsettings.json*. El valor de *appsettings.json* local permite depurar la aplicación localmente, pero el valor de App Service permite la ejecución de la aplicación en el producto con la configuración de producción. Las cadenas de conexión funcionan de la misma manera. De este modo, puede conservar los secretos de aplicación fuera de su repositorio de código y tener acceso a los valores adecuados sin cambiar el código.

> [!NOTE]
> Tenga en cuenta que el acceso a los [datos de configuración jerárquica](/aspnet/core/fundamentals/configuration/#hierarchical-configuration-data) en *appsettings.json* se tiene mediante el delimitador `:` que es estándar para .NET Core. Para invalidar una opción de configuración jerárquica específica en App Service, establezca el nombre de configuración de la aplicación con el mismo formato delimitado en la clave. Puede ejecutar el siguiente ejemplo en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings My:Hierarchical:Config:Data="some value"
```

## <a name="deploy-multi-project-solutions"></a>Implementar soluciones de varios proyectos

Cuando una solución de Visual Studio incluye varios proyectos, el proceso de publicación de Visual Studio ya incluye la selección del proyecto que se va a implementar. Cuando se implementa en el motor de implementación de App Service, como con Git o con implementación de archivo ZIP, con la automatización de compilación activada, el motor de implementación de App Service elige el primer sitio web o el proyecto de aplicación web que encuentra como aplicación de App Service. Para especificar el proyecto que App Service debe usar, especifique la configuración de la aplicación `PROJECT`. Por ejemplo, ejecute lo siguiente en [Cloud Shell](https://shell.azure.com):

```azurecli-interactive
az webapp config appsettings set --resource-group <resource-group-name> --name <app-name> --settings PROJECT="<project-name>/<project-name>.csproj"
```

## <a name="access-diagnostic-logs"></a>Acceso a los registros de diagnóstico

ASP.NET Core proporciona un [proveedor de registro integrado para App Service](/aspnet/core/fundamentals/logging/#azure-app-service). En *Program.cs* del proyecto, agregue el proveedor a la aplicación a través del método de extensión de `ConfigureLogging`, como se muestra en el ejemplo siguiente:

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureLogging(logging =>
        {
            logging.AddAzureWebAppDiagnostics();
        })
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

Después, podrá configurar y generar registros con el [patrón de .NET Core estándar](/aspnet/core/fundamentals/logging).

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

Para obtener más información sobre cómo solucionar problemas de aplicaciones de ASP.NET Core en App Service, consulte [Solución de problemas de ASP.NET Core en Azure App Service e IIS.](/aspnet/core/test/troubleshoot-azure-iis)

## <a name="get-detailed-exceptions-page"></a>Obtener la página de excepciones detalladas

Cuando la aplicación de ASP.NET Core genera una excepción en el depurador de Visual Studio, el explorador muestra una página de excepciones detalladas, pero, en App Service, esa página se sustituye por un error genérico **HTTP 500** o **Error al procesar la solicitud**. de la directiva). Para mostrar la página de excepciones detalladas en App Service, agregue la configuración de aplicación `ASPNETCORE_ENVIRONMENT` a la aplicación con el siguiente comando en <a target="_blank" href="https://shell.azure.com" >Cloud Shell</a>.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings ASPNETCORE_ENVIRONMENT="Development"
```

## <a name="detect-https-session"></a>Detección de sesión de HTTPS

En App Service, la [terminación de SSL](https://wikipedia.org/wiki/TLS_termination_proxy) se produce en los equilibradores de carga de red, por lo que todas las solicitudes HTTPS llegan a su aplicación en forma de solicitudes HTTP sin cifrar. Si necesita que la lógica de aplicación sepa si las solicitudes del usuario están cifradas o no, configure el middleware de encabezados reenviados en *Startup.cs*:

- Configure el middleware con [ForwardedHeadersOptions](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersoptions) para reenviar los encabezados `X-Forwarded-For` y `X-Forwarded-Proto` en `Startup.ConfigureServices`.
- Agregue intervalos de direcciones IP privadas a las redes conocidas, de modo que el middleware pueda confiar en el equilibrador de carga de App Service.
- Invoque el método [UseForwardedHeaders](/dotnet/api/microsoft.aspnetcore.builder.forwardedheadersextensions.useforwardedheaders) en `Startup.Configure` antes de invocar otro middleware.

Al colocar juntos los tres elementos, el código es similar al ejemplo siguiente:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();

    services.Configure<ForwardedHeadersOptions>(options =>
    {
        options.ForwardedHeaders =
            ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto;
        options.KnownNetworks.Add(new IPNetwork(IPAddress.Parse("::ffff:10.0.0.0"), 104));
        options.KnownNetworks.Add(new IPNetwork(IPAddress.Parse("::ffff:192.168.0.0"), 112));
        options.KnownNetworks.Add(new IPNetwork(IPAddress.Parse("::ffff:172.16.0.0"), 108));
    });
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseForwardedHeaders();

    ...

    app.UseMvc();
}
```

Para obtener más información, consulte [Configuración de ASP.NET Core para trabajar con servidores proxy y equilibradores de carga](/aspnet/core/host-and-deploy/proxy-load-balancer).

::: zone pivot="platform-linux"

## <a name="open-ssh-session-in-browser"></a>Abrir sesión SSH en el explorador

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-builtin-no-h.md)]

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

::: zone-end

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Aplicación de ASP.NET Core con SQL Database](tutorial-dotnetcore-sqldb-app.md)

::: zone pivot="platform-linux"

> [!div class="nextstepaction"]
> [P+F sobre App Service en Linux](faq-app-service-linux.md)

::: zone-end