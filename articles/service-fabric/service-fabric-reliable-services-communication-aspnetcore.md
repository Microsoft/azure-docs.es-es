---
title: Comunicación de servicio con el ASP.NET Core
description: Aprenda a usar ASP.NET Core en aplicaciones de Azure Service Fabric Reliable Services con estado y sin estado.
author: vturecek
ms.topic: conceptual
ms.date: 10/12/2018
ms.author: vturecek
ms.custom: devx-track-csharp
ms.openlocfilehash: 69423e7545178fd74ad44f5cab7b37b6f24b3577
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89022197"
---
# <a name="aspnet-core-in-azure-service-fabric-reliable-services"></a>ASP.NET Core en Reliable Services de Azure Service Fabric

ASP.NET Core es un marco de código abierto y multiplataforma. Este marco está diseñado para crear aplicaciones basadas en la nube, conectadas a internet, como aplicaciones web, aplicaciones de IoT y back-end móviles.

Este artículo es una guía en profundidad sobre cómo hospedar servicios de ASP.NET Core en Reliable Services de Service Fabric mediante el conjunto de paquetes NuGet **Microsoft.ServiceFabric.AspNetCore** .

Para ver un tutorial introductorio sobre ASP.NET Core en Service Fabric e instrucciones para configurar el entorno de desarrollo, consulte [Tutorial: Creación e implementación de una aplicación con un servicio de front-end de ASP.NET Core Web API y un servicio back-end con estado](service-fabric-tutorial-create-dotnet-app.md).

En el resto de este artículo se da por supuesto que ya conoce ASP.NET Core. Si no es así, lea los [fundamentos de ASP.NET Core](/aspnet/core/fundamentals/index).

## <a name="aspnet-core-in-the-service-fabric-environment"></a>ASP.NET Core en el entorno de Service Fabric

Las aplicaciones de ASP.NET Core y Service Fabric pueden ejecutarse en .NET Core y en .NET Framework completo. Puede usar ASP.NET Core de dos maneras diferentes en Service Fabric:
 - **Implementado como archivo ejecutable invitado**. Esta manera se usa principalmente para ejecutar aplicaciones ASP.NET Core existentes en Service Fabric sin realizar ningún cambio en el código.
 - **Ejecutado dentro de una instancia de Reliable Services**. Esta manera permite una mejor integración con el entorno de ejecución de Service Fabric y permite servicios ASP.NET Core con estado.

En el resto de este artículo se explica cómo usar ASP.NET Core dentro de una instancia de Reliable Services mediante los componentes de integración de ASP.NET Core que se incluyen con el SDK de Service Fabric.

## <a name="service-fabric-service-hosting"></a>Hospedaje de servicios de Service Fabric

En Service Fabric, una o varias instancias o réplicas del servicio se ejecutan en un *proceso de host de servicio*: un archivo ejecutable que ejecuta el código del servicio. Aunque, como autor del servicio, usted es el propietario del proceso de host de servicio, Service Fabric lo activa y supervisa automáticamente.

Tradicionalmente, ASP.NET (hasta MVC 5) ha estado estrechamente unido a IIS a través de System.Web.dll. ASP.NET Core proporciona una separación entre el servidor web y la aplicación web. Esta separación permite que las aplicaciones web sean portátiles entre diferentes servidores web. También permite que los servidores web sean *autohospedados*. Esto significa que puede iniciar un servidor web en su propio proceso, en lugar de un proceso que pertenezca a software de servidor web dedicado, como IIS.

Para combinar un servicio de Service Fabric y ASP.NET, ya sea como ejecutable invitado o en una instancia de Reliable Services, debe ser capaz de iniciar ASP.NET dentro de su proceso de host de servicio. El autohospedaje de ASP.NET Core lo permite.

## <a name="hosting-aspnet-core-in-a-reliable-service"></a>Hospedaje de ASP.NET Core en una instancia de Reliable Services
Por lo general, las aplicaciones ASP.NET Core autohospedadas crean un elemento WebHost en el punto de entrada de una aplicación, como el método `static void Main()` en `Program.cs`. En este caso, el ciclo de vida de WebHost está ligado al del proceso.

![Hospedaje de ASP.NET Core en un proceso][0]

Pero el punto de entrada de la aplicación no es el lugar idóneo para crear un elemento WebHost en una instancia de Reliable Service. Eso es porque el punto de entrada de la aplicación solo se usa para registrar un tipo de servicio con el tiempo de ejecución de Service Fabric, por lo que puede crear instancias de ese tipo de servicio. El elemento WebHost debe crearse en una instancia de Reliable Services. Dentro del proceso de host de servicio, las instancias o réplicas de servicio pueden pasar por varios ciclos de vida. 

Una instancia de Reliable Services se representa mediante la clase de servicio derivada de `StatelessService` o `StatefulService`. La pila de comunicación de un servicio se encuentra en una implementación `ICommunicationListener` en la clase de servicio. Los paquetes NuGet `Microsoft.ServiceFabric.AspNetCore.*` contienen implementaciones de `ICommunicationListener` que inician y administran el elemento WebHost de ASP.NET Core para Kestrel o HTTP.sys en una instancia de Reliable Services.

![Diagrama de hospedaje de ASP.NET Core en una instancia de Reliable Services][1]

## <a name="aspnet-core-icommunicationlisteners"></a>ICommunicationListeners en ASP.NET Core
Las implementaciones de `ICommunicationListener` para Kestrel y HTTP.sys en los paquetes NuGet `Microsoft.ServiceFabric.AspNetCore.*` tienen patrones de uso similares. Pero realizan acciones ligeramente diferentes, específicas de cada servidor web. 

Ambos agentes de escucha de comunicación proporcionan un constructor que acepta los argumentos siguientes:
 - **`ServiceContext serviceContext`** : Este es el objeto `ServiceContext` que contiene información acerca del servicio en ejecución.
 - **`string endpointName`** : Es el nombre de una configuración `Endpoint` en ServiceManifest.xml. Es donde más se diferencian los dos agentes de escucha de comunicaciones. HTTP.sys *requiere* una configuración de `Endpoint`, mientras que Kestrel no.
 - **`Func<string, AspNetCoreCommunicationListener, IWebHost> build`** : Esta es una expresión lambda que se implementa para crear y devolver `IWebHost`. Le permite configurar `IWebHost` tal como lo haría normalmente en una aplicación de ASP.NET Core. La expresión lambda proporciona una dirección URL que se genera automáticamente en función de las opciones de integración de Service Fabric que use y la configuración de `Endpoint` que proporcione. A continuación, puede modificar o usar esa dirección URL para iniciar el servidor web.

## <a name="service-fabric-integration-middleware"></a>Middleware de integración de Service Fabric
El paquete NuGet `Microsoft.ServiceFabric.AspNetCore` incluye el método de extensión `UseServiceFabricIntegration` en `IWebHostBuilder` que agrega middleware compatible con Service Fabric. Este middleware configura el `ICommunicationListener` de Kestrel o HTTP.sys para registrar una dirección URL de servicio única en Naming Service de Service Fabric. A continuación, valida las solicitudes del cliente para asegurarse de que los clientes se conecten al servicio adecuado. 

Este paso es necesario para impedir que los clientes se conecten por error al servicio incorrecto. Eso es porque, en un entorno de host compartido como Service Fabric, pueden ejecutarse varias aplicaciones web en la misma máquina física o virtual, pero no usar nombres de host exclusivos. Este escenario se describe con más detalle en la sección siguiente.

### <a name="a-case-of-mistaken-identity"></a>Un caso de identidad equivocada
Las réplicas de servicio, con independencia del protocolo, escuchan en una combinación IP:puerto única. Una vez que una réplica de servicio ha comenzado a escuchar en un punto de conexión IP:puerto, notifica esa dirección de punto de conexión a Naming Service de Service Fabric. Allí, los clientes u otros servicios pueden detectarla. Si los servicios usan puertos de aplicación asignados de forma dinámica, una réplica del servicio podría casualmente usar el mismo punto de conexión IP:puerto que otro servicio que se encontraba antes en la misma máquina física o virtual. Esto puede hacer que un cliente se conecte por error al servicio incorrecto. Este escenario puede ocurrir si se produce la siguiente secuencia de eventos:

 1. Un servicio A escucha en 10.0.0.1:30000 a través de HTTP. 
 2. El cliente resuelve el servicio A y obtiene la dirección 10.0.0.1:30000.
 3. El servicio A se mueve a otro nodo.
 4. El servicio B se coloca en 10.0.0.1 y casualmente usa el mismo puerto 30000.
 5. El cliente intenta conectarse al servicio A con la dirección 10.0.0.1:30000 en caché.
 6. Ahora el cliente está conectado al servicio B sin darse cuenta de que es el servicio incorrecto.

Esto puede causar errores aleatoriamente que son difíciles de diagnosticar.

### <a name="using-unique-service-urls"></a>Uso de direcciones URL de servicio únicas
Para evitar estos errores, los servicios pueden exponer un punto de conexión ante Naming Service con un identificador único y después validar este último durante las solicitudes de cliente. Se trata de una acción cooperativa entre servicios en un entorno de confianza sin inquilinos hostiles. No proporciona autenticación de servicio segura en un entorno con inquilinos hostiles.

En un entorno de confianza, el middleware que se agrega mediante el método `UseServiceFabricIntegration` anexa automáticamente un identificador único a la dirección que se expone ante Naming Service. Luego, valida ese identificador en cada solicitud. Si el identificador no coincide, el middleware devuelve inmediatamente una respuesta HTTP 410 Ya no existe.

Los servicios que utilizan un puerto asignado de forma dinámica deberían usar este middleware.

Los servicios que usan un puerto fijo único no tienen este problema en un entorno cooperativo. Un puerto fijo único se utiliza normalmente para servicios orientados hacia el exterior que necesitan usar un puerto conocido al que las aplicaciones cliente puedan conectarse. Por ejemplo, la mayoría de las aplicaciones web accesibles desde Internet usarán el puerto 80 o 443 para las conexiones del explorador web. En este caso, el identificador único no se debe habilitar.

En el siguiente diagrama se muestra el flujo de la solicitud con el middleware habilitado:

![Integración de ASP.NET Core de Service Fabric][2]

Tanto las implementaciones `ICommunicationListener` de Kestrel y HTTP.sys usan este mecanismo exactamente de la misma manera. Aunque HTTP.sys puede diferenciar internamente las solicitudes basadas en rutas de acceso de dirección URL únicas mediante la característica subyacente para compartir puertos **HTTP.sys**, esa funcionalidad *no* se usa en la implementación `ICommunicationListener` de HTTP.sys. Esto se debe a que se producirían códigos de estado de error HTTP 503 y HTTP 404 en el escenario descrito antes. A su vez, esto hace difícil que los clientes determinen la intención del error, ya que HTTP 503 y HTTP 404 se usan habitualmente para indicar otros errores. 

Por lo tanto, las implementaciones `ICommunicationListener` de Kestrel y HTTP.sys se estandarizan en middleware proporcionado por el método de extensión `UseServiceFabricIntegration`. Por lo tanto, los clientes solo tienen que realizar una acción de volver a resolver el punto de conexión del servicio en las respuestas HTTP 410.

## <a name="httpsys-in-reliable-services"></a>HTTP.sys en Reliable Services
HTTP.sys puede usarse en una instancia de Reliable Services si se importa el paquete NuGet **Microsoft.ServiceFabric.AspNetCore.HttpSys**. Este paquete contiene `HttpSysCommunicationListener`, una implementación de `ICommunicationListener`. `HttpSysCommunicationListener` le permite crear un elemento WebHost de ASP.NET Core dentro de una instancia de Reliable Services con HTTP.sys como servidor web.

HTTP.sys se basa en la [API de servidor HTTP de Windows](/windows/win32/http/http-api-start-page). Esta API usa el controlador de kernel **HTTP.sys** para procesar las solicitudes HTTP y enrutarlas a procesos que ejecutan aplicaciones web. Esto permite que varios procesos en la misma máquina física o virtual hospeden aplicaciones web en el mismo puerto, eliminando la ambigüedad con un nombre de host o una ruta de acceso de dirección URL únicos. Estas características son útiles en Service Fabric para hospedar varios sitios web en el mismo clúster.

>[!NOTE]
>La implementación de HTTP.sys solo funciona en la plataforma Windows.

En el siguiente diagrama se muestra cómo HTTP.sys usa el controlador de kernel **HTTP.sys** en Windows para compartir puertos:

![Diagrama de HTTP.sys][3]

### <a name="httpsys-in-a-stateless-service"></a>HTTP.sys en un servicio sin estado
Para usar `HttpSys` en un servicio sin estado, invalide el método `CreateServiceInstanceListeners` y devuelva una instancia de `HttpSysCommunicationListener`:

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    return new ServiceInstanceListener[]
    {
        new ServiceInstanceListener(serviceContext =>
            new HttpSysCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
                new WebHostBuilder()
                    .UseHttpSys()
                    .ConfigureServices(
                        services => services
                            .AddSingleton<StatelessServiceContext>(serviceContext))
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
                    .UseStartup<Startup>()
                    .UseUrls(url)
                    .Build()))
    };
}
```

### <a name="httpsys-in-a-stateful-service"></a>HTTP.sys en un servicio con estado

Actualmente, `HttpSysCommunicationListener` no está diseñado para usarse en servicios con estado a causa de las complicaciones con la característica subyacente para compartir puertos **HTTP.sys**. Para más información, consulte la sección siguiente sobre la asignación dinámica de puertos con HTTP.sys. Para los servicios con estado, Kestrel es el servidor web sugerido.

### <a name="endpoint-configuration"></a>Configuración del punto de conexión

Se necesita una configuración de `Endpoint` para los servidores web que usen la API de servidor HTTP de Windows, lo que incluye HTTP.sys. Los servidores web que usan la API de servidor HTTP de Windows deben reservar en primer lugar su dirección URL con HTTP.sys (normalmente con la herramienta [netsh](/windows/win32/http/netsh-commands-for-http)). 

Esta acción requiere privilegios elevados, que sus servicios carecen de forma predeterminada. Las opciones "http" o "https" para la propiedad `Protocol` de la configuración de `Endpoint` en ServiceManifest.xml se usan específicamente para indicar al entorno de ejecución de Service Fabric que registre una dirección URL con HTTP.sys en su nombre. Para ello, usa el prefijo de dirección URL con [*carácter comodín seguro*](/windows/win32/http/urlprefix-strings).

Por ejemplo, si se quiere reservar `http://+:80` para un servicio, use la siguiente configuración en ServiceManifest.xml:

```xml
<ServiceManifest ... >
    ...
    <Resources>
        <Endpoints>
            <Endpoint Name="ServiceEndpoint" Protocol="http" Port="80" />
        </Endpoints>
    </Resources>

</ServiceManifest>
```

Y se debe pasar el nombre del punto de conexión al constructor `HttpSysCommunicationListener`:

```csharp
 new HttpSysCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
 {
     return new WebHostBuilder()
         .UseHttpSys()
         .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
         .UseUrls(url)
         .Build();
 })
```

#### <a name="use-httpsys-with-a-static-port"></a>Uso de HTTP.sys con un puerto estático
Para usar un puerto estático con HTTP.sys, proporcione el número de puerto en la configuración de `Endpoint`:

```xml
  <Resources>
    <Endpoints>
      <Endpoint Protocol="http" Name="ServiceEndpoint" Port="80" />
    </Endpoints>
  </Resources>
```

#### <a name="use-httpsys-with-a-dynamic-port"></a>Uso de HTTP.sys con un puerto dinámico
Para usar un puerto asignado de forma dinámica con HTTP.sys, omita la propiedad `Port` en la configuración de `Endpoint`:

```xml
  <Resources>
    <Endpoints>
      <Endpoint Protocol="http" Name="ServiceEndpoint" />
    </Endpoints>
  </Resources>
```

Un puerto dinámico asignado por una configuración de `Endpoint` proporciona solo un puerto *por cada proceso de host*. El modelo de hospedaje actual de Service Fabric permite que el mismo proceso hospede varias réplicas o instancias de servicio. Esto significa que cada una compartirá el mismo puerto cuando se asignan a través de la configuración de `Endpoint`. Varias instancias de **HTTP.sys** pueden compartir un puerto al usar la característica subyacente para compartir puertos **HTTP.sys**. Pero no es compatible con `HttpSysCommunicationListener` debido a las complicaciones que presenta para las solicitudes de cliente. Para el uso de puertos dinámicos, Kestrel es el servidor web sugerido.

## <a name="kestrel-in-reliable-services"></a>Kestrel en Reliable Services
Puede usar Kestrel en Reliable Services si importa el paquete NuGet **Microsoft.ServiceFabric.AspNetCore.Kestrel**. Este paquete contiene `KestrelCommunicationListener`, una implementación de `ICommunicationListener`. `KestrelCommunicationListener` le permite crear un elemento WebHost de ASP.NET Core dentro de una instancia de Reliable Services con Kestrel como servidor web.

Kestrel es un servidor web multiplataforma para ASP.NET Core. A diferencia de HTTP.sys, Kestrel no usa un administrador centralizado de puntos de conexión. También a diferencia de HTTP.sys, Kestrel no admite que varios procesos compartan puertos. Cada instancia de Kestrel debe usar un puerto único. Para más información sobre Kestrel, consulte los [detalles de implementación](/aspnet/core/fundamentals/servers/kestrel?view=aspnetcore-2.2).

![Diagrama de Kestrel][4]

### <a name="kestrel-in-a-stateless-service"></a>Kestrel en un servicio sin estado
Para usar `Kestrel` en un servicio sin estado, invalide el método `CreateServiceInstanceListeners` y devuelva una instancia de `KestrelCommunicationListener`:

```csharp
protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
{
    return new ServiceInstanceListener[]
    {
        new ServiceInstanceListener(serviceContext =>
            new KestrelCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
                new WebHostBuilder()
                    .UseKestrel()
                    .ConfigureServices(
                        services => services
                            .AddSingleton<StatelessServiceContext>(serviceContext))
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
                    .UseStartup<Startup>()
                    .UseUrls(url)
                    .Build();
            ))
    };
}
```

### <a name="kestrel-in-a-stateful-service"></a>Kestrel en un servicio con estado
Para usar `Kestrel` en un servicio con estado, invalide el método `CreateServiceReplicaListeners` y devuelva una instancia de `KestrelCommunicationListener`:

```csharp
protected override IEnumerable<ServiceReplicaListener> CreateServiceReplicaListeners()
{
    return new ServiceReplicaListener[]
    {
        new ServiceReplicaListener(serviceContext =>
            new KestrelCommunicationListener(serviceContext, (url, listener) =>
                new WebHostBuilder()
                    .UseKestrel()
                    .ConfigureServices(
                         services => services
                             .AddSingleton<StatefulServiceContext>(serviceContext)
                             .AddSingleton<IReliableStateManager>(this.StateManager))
                    .UseContentRoot(Directory.GetCurrentDirectory())
                    .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.UseUniqueServiceUrl)
                    .UseStartup<Startup>()
                    .UseUrls(url)
                    .Build();
            ))
    };
}
```

En este ejemplo, se proporciona una instancia singleton de `IReliableStateManager` en el contenedor de inyección de dependencias de WebHost. Esto no es estrictamente necesario, pero sí permite usar `IReliableStateManager` y Reliable Collections en los métodos de acción de controlador MVC.

*No* se proporciona un nombre de configuración `Endpoint` a `KestrelCommunicationListener` en un servicio con estado. Esto se explica con más detalle en la sección siguiente.

### <a name="configure-kestrel-to-use-https"></a>Configurar Kestrel para que use HTTPS
Al habilitar HTTPS con Kestrel en el servicio, deberá establecer varias opciones de escucha. Actualice `ServiceInstanceListener` para usar un punto de conexión *EndpointHttps* y escuche en un puerto específico (por ejemplo, el puerto 443). Al configurar el host web para usar el servidor web de Kestrel, debe configurar Kestrel para escuchar las direcciones IPv6 en todas las interfaces de red: 

```csharp
new ServiceInstanceListener(
serviceContext =>
    new KestrelCommunicationListener(
        serviceContext,
        "EndpointHttps",
        (url, listener) =>
        {
            ServiceEventSource.Current.ServiceMessage(serviceContext, $"Starting Kestrel on {url}");

            return new WebHostBuilder()
                .UseKestrel(opt =>
                {
                    int port = serviceContext.CodePackageActivationContext.GetEndpoint("EndpointHttps").Port;
                    opt.Listen(IPAddress.IPv6Any, port, listenOptions =>
                    {
                        listenOptions.UseHttps(GetCertificateFromStore());
                        listenOptions.NoDelay = true;
                    });
                })
                .ConfigureAppConfiguration((builderContext, config) =>
                {
                    config.AddJsonFile("appsettings.json", optional: false, reloadOnChange: true);
                })

                .ConfigureServices(
                    services => services
                        .AddSingleton<HttpClient>(new HttpClient())
                        .AddSingleton<FabricClient>(new FabricClient())
                        .AddSingleton<StatelessServiceContext>(serviceContext))
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseStartup<Startup>()
                .UseServiceFabricIntegration(listener, ServiceFabricIntegrationOptions.None)
                .UseUrls(url)
                .Build();
        }))
```

Para obtener un ejemplo completo usado en un tutorial, consulte [Configuración de Kestrel para que use HTTPS](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md#configure-kestrel-to-use-https).


### <a name="endpoint-configuration"></a>Configuración del punto de conexión
No se requiere una configuración de `Endpoint` para usar Kestrel. 

Kestrel es un servidor web independiente simple. A diferencia de HTTP.sys (o HttpListener), no es necesaria una configuración de `Endpoint` en ServiceManifest.xml, ya que no requiere el registro de la dirección URL antes del inicio. 

#### <a name="use-kestrel-with-a-static-port"></a>Uso de Kestrel con un puerto estático
Puede configurar un puerto estático en la configuración de `Endpoint` de ServiceManifest.xml para usarlo con Kestrel. Aunque esto no es estrictamente necesario, ofrece dos ventajas potenciales:
 - Si el puerto no se encuentra en el intervalo de puertos de la aplicación, Service Fabric lo abre a través del firewall del sistema operativo.
 - La dirección URL proporcionada a través de `KestrelCommunicationListener` usará este puerto.

```xml
  <Resources>
    <Endpoints>
      <Endpoint Protocol="http" Name="ServiceEndpoint" Port="80" />
    </Endpoints>
  </Resources>
```

Si se configura una instancia de `Endpoint`, se debe pasar su nombre al constructor `KestrelCommunicationListener`: 

```csharp
new KestrelCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) => ...
```

Si ServiceManifest.xml no usa una configuración de `Endpoint`, omita el nombre en el constructor `KestrelCommunicationListener`. En este caso, usará un puerto dinámico. Para más información sobre esto, consulte la siguiente sección.

#### <a name="use-kestrel-with-a-dynamic-port"></a>Uso de Kestrel con un puerto dinámico
Kestrel no puede usar la asignación automática de puertos desde la configuración de `Endpoint` en ServiceManifest.xml. Esto se debe a que la asignación automática de puertos de una configuración de `Endpoint` asigna un puerto único por cada *proceso de host*, y un único proceso de host puede contener varias instancias de Kestrel. Esto no funciona con Kestrel porque no admite compartir puertos. Por lo tanto, cada instancia de Kestrel debe abrirse en un puerto único.

Para usar la asignación dinámica de puertos con Kestrel, omita la configuración de `Endpoint` en ServiceManifest.xml por completo y no pase un nombre de punto de conexión al constructor `KestrelCommunicationListener`, del modo siguiente:

```csharp
new KestrelCommunicationListener(serviceContext, (url, listener) => ...
```

En esta configuración, `KestrelCommunicationListener` seleccionará automáticamente un puerto no utilizado en el intervalo de puertos de la aplicación.

Para HTTPS, debe tener el punto de conexión configurado con el protocolo HTTPS sin un puerto especificado en ServiceManifest.xml y pasar el nombre del punto de conexión al constructor KestrelCommunicationListener.


## <a name="service-fabric-configuration-provider"></a>Proveedor de configuración de Service Fabric
La configuración de aplicaciones en ASP.NET Core se basa en pares de clave-valor establecidos por el proveedor de configuración. Lea [Configuración en ASP.NET Core](/aspnet/core/fundamentals/configuration/) para comprender más sobre la compatibilidad general de configuración de ASP.NET Core.

En esta sección se describe cómo el proveedor de configuración de Service Fabric se integra en la configuración de ASP.NET Core mediante la importación del paquete NuGet `Microsoft.ServiceFabric.AspNetCore.Configuration`.

### <a name="addservicefabricconfiguration-startup-extensions"></a>Extensiones de inicio AddServiceFabricConfiguration
Después de importar el paquete NuGet `Microsoft.ServiceFabric.AspNetCore.Configuration`, deberá registrar el origen de configuración de Service Fabric con la API de configuración de ASP.NET Core. Para ello, compruebe las extensiones **AddServiceFabricConfiguration** en el espacio de nombres `Microsoft.ServiceFabric.AspNetCore.Configuration` en `IConfigurationBuilder`.

```csharp
using Microsoft.ServiceFabric.AspNetCore.Configuration;

public Startup(IHostingEnvironment env)
{
    var builder = new ConfigurationBuilder()
        .SetBasePath(env.ContentRootPath)
        .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
        .AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true)
        .AddServiceFabricConfiguration() // Add Service Fabric configuration settings.
        .AddEnvironmentVariables();
    Configuration = builder.Build();
}

public IConfigurationRoot Configuration { get; }
```

Ahora el servicio de ASP.NET Core puede acceder a los valores de configuración de Service Fabric, al igual que cualquier otra configuración de la aplicación. Por ejemplo, puede usar el patrón de opciones para cargar la configuración en objetos fuertemente tipados.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<MyOptions>(Configuration);  // Strongly typed configuration object.
    services.AddMvc();
}
```
### <a name="default-key-mapping"></a>Asignación de claves predeterminada
De forma predeterminada, el proveedor de configuración de Service Fabric incluye el nombre del paquete, el nombre de la sección y el nombre de la propiedad. Juntos, forman la clave de configuración de ASP.NET Core, como sigue:
```csharp
$"{this.PackageName}{ConfigurationPath.KeyDelimiter}{section.Name}{ConfigurationPath.KeyDelimiter}{property.Name}"
```

Por ejemplo, si tiene un paquete de configuración denominado `MyConfigPackage` con el siguiente contenido, el valor de configuración estará disponible en ASP.NET Core `IConfiguration` a través de *MyConfigPackage:MyConfigSection:MyParameter*.
```xml
<?xml version="1.0" encoding="utf-8" ?>
<Settings xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" xmlns="http://schemas.microsoft.com/2011/01/fabric">  
  <Section Name="MyConfigSection">
    <Parameter Name="MyParameter" Value="Value1" />
  </Section>  
</Settings>
```
### <a name="service-fabric-configuration-options"></a>Opciones de configuración de Service Fabric
El proveedor de configuración de Service Fabric también admite `ServiceFabricConfigurationOptions` para cambiar el comportamiento predeterminado de asignación de claves.

#### <a name="encrypted-settings"></a>Configuración cifrada
Service Fabric admite la configuración cifrada, igual que el proveedor de configuración de Service Fabric. De manera predeterminada, la configuración cifrada no se descifra en `IConfiguration` de ASP.NET Core. En su lugar, los valores cifrados se almacenan allí. Pero si quiere descifrar el valor para almacenarlos en IConfiguration de ASP.NET Core, puede establecer la marca *DecryptValue* en false en la extensión `AddServiceFabricConfiguration`, como se indica a continuación:

```csharp
public Startup()
{
    ICodePackageActivationContext activationContext = FabricRuntime.GetActivationContext();
    var builder = new ConfigurationBuilder()        
        .AddServiceFabricConfiguration(activationContext, (options) => options.DecryptValue = false); // set flag to decrypt the value
    Configuration = builder.Build();
}
```
#### <a name="multiple-configuration-packages"></a>Varios paquetes de configuración
Service Fabric admite varios paquetes de configuración. De manera predeterminada, el nombre del paquete se incluye en la clave de configuración. Pero puede establecer la marca `IncludePackageName` en false, como se indica a continuación:
```csharp
public Startup()
{
    ICodePackageActivationContext activationContext = FabricRuntime.GetActivationContext();
    var builder = new ConfigurationBuilder()        
        // exclude package name from key.
        .AddServiceFabricConfiguration(activationContext, (options) => options.IncludePackageName = false); 
    Configuration = builder.Build();
}
```
#### <a name="custom-key-mapping-value-extraction-and-data-population"></a>Asignación de claves personalizada, extracción de valores y rellenado de datos
El proveedor de configuración de Service Fabric también admite escenarios más avanzados para personalizar la asignación de claves con `ExtractKeyFunc` y extraer los valores con `ExtractValueFunc` de manera personalizada. Incluso puede cambiar todo el proceso de rellenado de datos de la configuración de Service Fabric a la configuración de ASP.NET Core con `ConfigAction`.

Los ejemplos siguientes muestran cómo usar `ConfigAction` para personalizar el rellenado de datos:
```csharp
public Startup()
{
    ICodePackageActivationContext activationContext = FabricRuntime.GetActivationContext();
    
    this.valueCount = 0;
    this.sectionCount = 0;
    var builder = new ConfigurationBuilder();
    builder.AddServiceFabricConfiguration(activationContext, (options) =>
        {
            options.ConfigAction = (package, configData) =>
            {
                ILogger logger = new ConsoleLogger("Test", null, false);
                logger.LogInformation($"Config Update for package {package.Path} started");

                foreach (var section in package.Settings.Sections)
                {
                    this.sectionCount++;

                    foreach (var param in section.Parameters)
                    {
                        configData[options.ExtractKeyFunc(section, param)] = options.ExtractValueFunc(section, param);
                        this.valueCount++;
                    }
                }

                logger.LogInformation($"Config Update for package {package.Path} finished");
            };
        });
  Configuration = builder.Build();
}
```

### <a name="configuration-updates"></a>Actualizaciones de configuración
El proveedor de configuración de Service Fabric también admite las actualizaciones de configuración. Puede usar ASP.NET Core `IOptionsMonitor` para recibir notificaciones de cambios y, luego, usar `IOptionsSnapshot` para recargar los datos de configuración. Para más información, consulte [Opciones de ASP.NET Core](/aspnet/core/fundamentals/configuration/options).

Estas opciones se admiten de manera predeterminada. No se necesita ninguna codificación adicional para habilitar las actualizaciones de configuración.

## <a name="scenarios-and-configurations"></a>Escenarios y configuraciones
En esta sección se describe la combinación recomendada de servidor web, configuración de puertos, opciones de integración de Service Fabric y diversas configuraciones para solucionar los problemas de los escenarios siguientes:
 - Servicios sin estado de ASP.NET Core expuestos al exterior
 - Servicios sin estado de ASP.NET Core solo internos
 - Servicios con estado de ASP.NET Core solo internos

Un **servicio expuesto al exterior** es aquel que expone un punto de conexión al que se llama desde fuera del clúster, por lo general mediante un equilibrador de carga.

Un servicio **solo interno** es aquel a cuyo punto de conexión solo se llama desde dentro del clúster.

> [!NOTE]
> En general, los puntos de conexión de servicio con estado no deberían exponerse a Internet. Los clústeres detrás de los equilibradores de carga que no son conscientes de resolución de servicio de Service Fabric, como Azure Load Balancer, no podrán exponer los servicios con estado. Eso es porque el equilibrador de carga no podrá ubicar ni enrutar el tráfico a la réplica del servicio con estado adecuada. 

### <a name="externally-exposed-aspnet-core-stateless-services"></a>Servicios sin estado de ASP.NET Core expuestos al exterior
Kestrel es el servidor web sugerido para los servicios de front-end que exponen puntos de conexión HTTP externos orientados a Internet en HTTP. En Windows, HTTP.sys puede proporcionar la funcionalidad para compartir puertos, lo que le permite hospedar varios servicios web en el mismo conjunto de nodos con el mismo puerto. En este escenario, los servicios web se diferencian por nombre de host o ruta de acceso, sin tener que depender de un proxy de front-end ni puerta de enlace para brindar enrutamiento HTTP.
 
Cuando se expone a Internet, un servicio sin estado debe usar un punto de conexión conocido y estable que sea accesible a través de un equilibrador de carga. Usted proporcionará esta dirección URL a los usuarios de su aplicación. Se recomienda la siguiente configuración:

| Tipo | Recomendación | Notas |
| ---- | -------------- | ----- |
| Servidor Web | Kestrel | Kestrel es el servidor web preferido, porque se admite en Windows y Linux. |
| Configuración de puerto | estática | Se debe configurar un puerto estático conocido en la configuración `Endpoints` de ServiceManifest.xml, como 80 para HTTP o 443 para HTTPS. |
| ServiceFabricIntegrationOptions | None | Use la opción `ServiceFabricIntegrationOptions.None` al configurar el middleware de integración de Service Fabric para que el servicio no intente validar solicitudes entrantes para un identificador único. Los usuarios externos de la aplicación no conocerán la información de identificación única que usa el middleware. |
| Recuento de instancias | -1 | En los casos de uso típicos, la configuración de recuento de instancias se debe establecer en *-1*. Esto se hace para que haya disponible una instancia en todos los nodos que reciben tráfico de un equilibrador de carga. |

Si varios servicios expuestos al exterior comparten el mismo conjunto de nodos, puede usar HTTP.sys con una ruta de acceso de dirección URL única, pero estable. Para lograr esto, puede modificar la dirección URL proporcionada al configurar IWebHost. Tenga en cuenta que esto solo se aplica a HTTP.sys.

 ```csharp
 new HttpSysCommunicationListener(serviceContext, "ServiceEndpoint", (url, listener) =>
 {
     url += "/MyUniqueServicePath";
 
     return new WebHostBuilder()
         .UseHttpSys()
         ...
         .UseUrls(url)
         .Build();
 })
 ```

### <a name="internal-only-stateless-aspnet-core-service"></a>Servicio sin estado de ASP.NET Core solo interno
Los servicios sin estado a los que solo se llama desde dentro del clúster deben utilizar direcciones URL únicas y puertos asignados de forma dinámica para garantizar la cooperación entre varios servicios. Se recomienda la siguiente configuración:

| Tipo | Recomendación | Notas |
| ---- | -------------- | ----- |
| Servidor Web | Kestrel | Aunque puede usar HTTP.sys para servicios sin estado internos, Kestrel es el mejor servidor para permitir que varias instancias de servicio compartan un host.  |
| Configuración de puerto | asignado de forma dinámica | Varias réplicas de un servicio con estado pueden compartir un proceso de host o un sistema operativo host y, por tanto, necesitarán puertos únicos. |
| ServiceFabricIntegrationOptions | UseUniqueServiceUrl | Con la asignación dinámica de puertos, esta configuración evita el problema de identidad equivocada descrito antes. |
| InstanceCount | cualquiera | La configuración de recuento de instancias se puede establecer en cualquier valor necesario para hacer funcionar el servicio. |

### <a name="internal-only-stateful-aspnet-core-service"></a>Servicio con estado de ASP.NET Core solo interno
Los servicios con estado a los que solo se llama desde dentro del clúster deben utilizar puertos asignados de forma dinámica para garantizar la cooperación entre varios servicios. Se recomienda la siguiente configuración:

| Tipo | Recomendación | Notas |
| ---- | -------------- | ----- |
| Servidor Web | Kestrel | `HttpSysCommunicationListener` no está diseñado para usarse en servicios con estado en los que las réplicas comparten un proceso de host. |
| Configuración de puerto | asignado de forma dinámica | Varias réplicas de un servicio con estado pueden compartir un proceso de host o un sistema operativo host y, por tanto, necesitarán puertos únicos. |
| ServiceFabricIntegrationOptions | UseUniqueServiceUrl | Con la asignación dinámica de puertos, esta configuración evita el problema de identidad equivocada descrito antes. |

## <a name="next-steps"></a>Pasos siguientes
[Depurar la aplicación de Service Fabric con Visual Studio](service-fabric-debugging-your-application.md)


<!--Image references-->
[0]:./media/service-fabric-reliable-services-communication-aspnetcore/webhost-standalone.png
[1]:./media/service-fabric-reliable-services-communication-aspnetcore/webhost-servicefabric.png
[2]:./media/service-fabric-reliable-services-communication-aspnetcore/integration.png
[3]:./media/service-fabric-reliable-services-communication-aspnetcore/httpsys.png
[4]:./media/service-fabric-reliable-services-communication-aspnetcore/kestrel.png
