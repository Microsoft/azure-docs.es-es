---
title: Uso de la inserción de dependencias en Azure Functions con .NET
description: Aprenda a usar la inserción de dependencias para el registro y uso de servicios en las funciones de .NET.
author: ggailey777
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 08/15/2020
ms.author: glenga
ms.reviewer: jehollan
ms.openlocfilehash: 7e45537d0a9fbdd738d6a2142b9259a15498e9c9
ms.sourcegitcommit: 59ea8436d7f23bee75e04a84ee6ec24702fb2e61
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/07/2020
ms.locfileid: "89503812"
---
# <a name="use-dependency-injection-in-net-azure-functions"></a>Uso de la inserción de dependencias en Azure Functions con .NET

Azure Functions admite el modelo de diseño de software de inserción de dependencias (DI), que es una técnica para lograr la [inversión de control (IoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) entre las clases y sus dependencias.

- La inserción de dependencias en Azure Functions se basa en las características de inserción de dependencias de .NET Core. Se recomienda estar familiarizado con la [inserción de dependencias de .NET Core](/aspnet/core/fundamentals/dependency-injection). Hay diferencias en el modo en que se invalidan las dependencias y cómo se leen los valores de configuración con Azure Functions en el plan de consumo.

- La compatibilidad con la inserción de dependencias comienza con Azure Functions 2.x.

## <a name="prerequisites"></a>Requisitos previos

Para poder usar la inserción de dependencias, debe instalar los siguientes paquetes NuGet:

- [Microsoft.Azure.Functions.Extensions](https://www.nuget.org/packages/Microsoft.Azure.Functions.Extensions/)

- Versión 1.0.28 o posterior del paquete [Microsoft.NET.Sdk.Functions](https://www.nuget.org/packages/Microsoft.NET.Sdk.Functions/)

## <a name="register-services"></a>Registro de los servicios

Para registrar los servicios, cree un método para configurar y agregar componentes a una instancia de `IFunctionsHostBuilder`.  El host de Azure Functions crea una instancia de `IFunctionsHostBuilder` y la pasa directamente al método.

Para registrar el método, agregue el atributo de ensamblado `FunctionsStartup` que especifica el nombre del tipo usado durante el inicio.

```csharp
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using Microsoft.Extensions.DependencyInjection;

[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.Services.AddHttpClient();

            builder.Services.AddSingleton<IMyService>((s) => {
                return new MyService();
            });

            builder.Services.AddSingleton<ILoggerProvider, MyLoggerProvider>();
        }
    }
}
```

En este ejemplo se usa el paquete [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http/) para registrar un `HttpClient` en el inicio.

### <a name="caveats"></a>Advertencias

Una serie de pasos de registro se ejecuta antes y después de que el entorno de ejecución procese la clase de inicio. Por lo tanto, tenga en cuenta los elementos siguientes:

- *La clase de inicio está pensada únicamente para la instalación y el registro*. Evite el uso de servicios registrados en el inicio durante el proceso de inicio. Por ejemplo, no intente registrar un mensaje en un registrador que se está registrando durante el inicio. Este punto del proceso de registro es demasiado pronto para que los servicios estén disponibles para su uso. Una vez que se ejecuta el método `Configure`, el entorno de ejecución de Functions continúa registrando las dependencias adicionales, lo que puede afectar el funcionamiento de los servicios.

- *El contenedor de inserción de dependencias solo contiene tipos registrados explícitamente*. Los únicos servicios disponibles como tipos insertables son lo que se configuran en el método `Configure`. Como resultado, los tipos específicos de Functions como `BindingContext` y `ExecutionContext` no están disponibles durante la instalación ni como tipos insertables.

## <a name="use-injected-dependencies"></a>Uso de dependencias insertadas

La inserción de constructores se usa para que las dependencias estén disponibles en una función. El uso de la inserción de constructores requiere que no se usen clases estáticas para los servicios insertados o para las clases de función.

El ejemplo siguiente muestra cómo las dependencias `IMyService` y `HttpClient` se insertan en una función desencadenada por HTTP.

```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;
using System.Net.Http;
using System.Threading.Tasks;

namespace MyNamespace
{
    public class MyHttpTrigger
    {
        private readonly HttpClient _client;
        private readonly IMyService _service;

        public MyHttpTrigger(HttpClient httpClient, MyService service)
        {
            this._client = httpClient;
            this._service = service;
        }

        [FunctionName("MyHttpTrigger")]
        public async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "get", "post", Route = null)] HttpRequest req,
            ILogger log)
        {
            var response = await _client.GetAsync("https://microsoft.com");
            var message = _service.GetMessage();

            return new OkObjectResult("Response from function with injected dependencies.");
        }
    }
}
```

En este ejemplo se usa el paquete [Microsoft.Extensions.Http](https://www.nuget.org/packages/Microsoft.Extensions.Http/) para registrar un `HttpClient` en el inicio.

## <a name="service-lifetimes"></a>Vigencia de los servicios

Las aplicaciones de Azure Functions proporcionan la misma vigencia de servicio que la [inserción de dependencias de ASP.NET](/aspnet/core/fundamentals/dependency-injection#service-lifetimes). En el caso de una aplicación de Functions, las distintas vigencias del servicio se comportan de la manera siguiente:

- **Transitorio**: los servicios transitorios se crean en cada solicitud del servicio.
- **De ámbito**: una vigencia de servicio de ámbito coincide con una vigencia de ejecución de la función. Los servicios de ámbito se crean una vez por cada ejecución. Las solicitudes posteriores para ese servicio durante la ejecución vuelven a usar la instancia de servicio existente.
- **Singleton**: la vigencia singleton del servicio coincide con la vigencia del host y se reutiliza en ejecuciones de la función en esa instancia. Los servicios de vigencia singleton se recomiendan para conexiones y clientes, por ejemplo, para instancias `DocumentClient` o `HttpClient`.

Consulte o descargue un [ejemplo de vigencia de servicio diferente](https://github.com/Azure/azure-functions-dotnet-extensions/tree/main/src/samples/DependencyInjection/Scopes) en GitHub.

## <a name="logging-services"></a>Registro de servicios

Si necesita su propio proveedor de registro, registre un tipo personalizado como instancia de [`ILoggerProvider`](/dotnet/api/microsoft.extensions.logging.iloggerfactory), disponible mediante el paquete NuGet [Microsoft.Extensions.Logging.Abstractions](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Abstractions/).

Azure Functions agrega Application Insights automáticamente.

> [!WARNING]
> - No agregue `AddApplicationInsightsTelemetry()` a la colección de servicios, ya que registra los servicios que entran en conflicto con los servicios proporcionados por el entorno.
> - No registre su propio `TelemetryConfiguration` o `TelemetryClient` si usa la funcionalidad de Application Insights integrada. Si tiene que configurar su propia instancia de `TelemetryClient`, cree una a través de la `TelemetryConfiguration` insertada, tal como se muestra en [Monitor Azure Functions](./functions-monitoring.md#version-2x-and-later-2).

### <a name="iloggert-and-iloggerfactory"></a>ILogger<T> e ILoggerFactory

El host insertará los servicios `ILogger<T>` e `ILoggerFactory` en constructores.  Sin embargo, de forma predeterminada, estos nuevos filtros de registro se filtran de los registros de función.  Tiene que modificar el archivo de `host.json` para participar en otros filtros y categorías.

En el ejemplo siguiente se muestra cómo agregar un elemento `ILogger<HttpTrigger>` con registros que se exponen al host.

```csharp
namespace MyNamespace
{
    public class HttpTrigger
    {
        private readonly ILogger<HttpTrigger> _log;

        public HttpTrigger(ILogger<HttpTrigger> log)
        {
            _log = log;
        }

        [FunctionName("HttpTrigger")]
        public async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req)
        {
            _log.LogInformation("C# HTTP trigger function processed a request.");

            // ...
    }
}
```

En el ejemplo siguiente, el archivo `host.json` se agrega al filtro de seguridad.

```json
{
    "version": "2.0",
    "logging": {
        "applicationInsights": {
            "samplingSettings": {
                "isEnabled": true,
                "excludedTypes": "Request"
            }
        },
        "logLevel": {
            "MyNamespace.HttpTrigger": "Information"
        }
    }
}
```

## <a name="function-app-provided-services"></a>Servicios proporcionados por la aplicación de funciones

El host de la función registra muchos servicios. Los siguientes servicios son seguros para tomar como dependencia en la aplicación:

|Tipo de servicio|Vigencia|Descripción|
|--|--|--|
|`Microsoft.Extensions.Configuration.IConfiguration`|Singleton|Configuración del entorno en tiempo de ejecución|
|`Microsoft.Azure.WebJobs.Host.Executors.IHostIdProvider`|Singleton|Responsable de proporcionar el id. de la instancia de host|

Si hay otros servicios en los que quiere tomar una dependencia, [cree un problema y propóngalo en GitHub](https://github.com/azure/azure-functions-host).

### <a name="overriding-host-services"></a>Invalidación de los servicios de host

La invalidación de los servicios proporcionados por el host no se admite.  Si hay otros servicios que quiere invalidar, [cree un problema y propóngalo en GitHub](https://github.com/azure/azure-functions-host).

## <a name="working-with-options-and-settings"></a>Trabajo con opciones y configuraciones

Los valores definidos en la [configuración de la aplicación](./functions-how-to-use-azure-function-app-settings.md#settings) están disponibles en una instancia de `IConfiguration`, lo que permite leer los valores de la configuración de la aplicación en la clase de inicio.

Puede extraer valores de la instancia de `IConfiguration` a un tipo personalizado. Copiar los valores de configuración de la aplicación en un tipo personalizado facilita la prueba de los servicios, ya que permite que estos valores se inserten. Los valores leídos en la instancia de configuración deben ser pares de clave/valor simples.

Considere la clase siguiente que incluye una propiedad con nombre coherente con una configuración de aplicación:

```csharp
public class MyOptions
{
    public string MyCustomSetting { get; set; }
}
```

Y un archivo `local.settings.json` que puede estructurar la configuración personalizada de la manera siguiente:
```json
{
  "IsEncrypted": false,
  "Values": {
    "MyOptions:MyCustomSetting": "Foobar"
  }
}
```

Desde dentro del método `Startup.Configure`, puede extraer valores de la instancia de `IConfiguration` en el tipo personalizado mediante el código siguiente:

```csharp
builder.Services.AddOptions<MyOptions>()
    .Configure<IConfiguration>((settings, configuration) =>
    {
        configuration.GetSection("MyOptions").Bind(settings);
    });
```

Llamar a `Bind` copia los valores con nombres de propiedad coincidentes de la configuración en la instancia personalizada. La instancia de opciones ahora está disponible en el contenedor de IoC para su inserción en una función.

El objeto de opciones se inserta en la función como una instancia de la interfaz `IOptions` genérica. Use la propiedad `Value` para acceder a los valores encontrados en su configuración.

```csharp
using System;
using Microsoft.Extensions.Options;

public class HttpTrigger
{
    private readonly MyOptions _settings;

    public HttpTrigger(IOptions<MyOptions> options)
    {
        _settings = options.Value;
    }
}
```

Consulte [Patrón de opciones en ASP.NET Core](/aspnet/core/fundamentals/configuration/options) para más detalles sobre cómo trabajar con opciones.

## <a name="customizing-configuration-sources"></a>Personalización de orígenes de configuración

> [!NOTE]
> La personalización de orígenes de configuración está disponible a partir de las versiones de host 2.0.14192.0 y 3.0.14191.0 de Azure Functions.

Para especificar orígenes de configuración adicionales, invalide el método `ConfigureAppConfiguration` en la clase `StartUp` de su aplicación de funciones.

En el ejemplo siguiente se añaden valores de configuración a partir de una base y archivos de configuración de aplicaciones específicos de un entorno opcional.

```csharp
using System.IO;
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

[assembly: FunctionsStartup(typeof(MyNamespace.Startup))]

namespace MyNamespace
{
    public class Startup : FunctionsStartup
    {
        public override void ConfigureAppConfiguration(IFunctionsConfigurationBuilder builder)
        {
            FunctionsHostBuilderContext context = builder.GetContext();

            builder.ConfigurationBuilder
                .AddJsonFile(Path.Combine(context.ApplicationRootPath, "appsettings.json"), optional: true, reloadOnChange: false)
                .AddJsonFile(Path.Combine(context.ApplicationRootPath, $"appsettings.{context.EnvironmentName}.json"), optional: true, reloadOnChange: false)
                .AddEnvironmentVariables();
        }
    }
}
```

Añada proveedores de configuración a la propiedad `ConfigurationBuilder` de `IFunctionsConfigurationBuilder`. Para obtener más información sobre el uso de proveedores de configuración, consulte [Configuración en ASP.NET Core](/aspnet/core/fundamentals/configuration/?view=aspnetcore-3.1#configuration-providers).

`FunctionsHostBuilderContext` se obtiene de `IFunctionsConfigurationBuilder.GetContext()`. Use este contexto para recuperar el nombre del entorno actual y resolver la ubicación de los archivos de configuración almacenados en la carpeta de su aplicación de funciones.

De forma predeterminada, los archivos de configuración como *appsettings.json* no se copian automáticamente en la carpeta de salida de la aplicación de funciones. Modifique el archivo *. csproj* para que coincida con el ejemplo siguiente y asegúrese de que los archivos se copien.

```xml
<None Update="appsettings.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>      
</None>
<None Update="appsettings.Development.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    <CopyToPublishDirectory>Never</CopyToPublishDirectory>
</None>
```

> [!IMPORTANT]
> En el caso de las aplicaciones de funciones que se ejecuten en los planes de consumo o Premium, las modificaciones de los valores de configuración utilizados en los desencadenadores pueden provocar problemas de escalabilidad. Cualquier cambio que se haga en estas propiedades mediante la clase `FunctionsStartup` generará un error de inicio en la aplicación de funciones.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte los siguientes recursos:

- [Cómo supervisar la aplicación de funciones](functions-monitoring.md)
- [Procedimientos recomendados para Functions](functions-best-practices.md)
