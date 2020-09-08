---
title: 'Escalado con varias instancias: Azure SignalR Service'
description: En muchos escenarios de escalado, el cliente necesita a menudo aprovisionar varias instancias y configurarlas para usarlas juntas, a fin de crear una implementación a gran escala. Por ejemplo, el particionamiento requiere compatibilidad con varias instancias.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 03/27/2019
ms.author: zhshang
ms.openlocfilehash: ecf4a35fc239a70e87550a97e71d7abd3d00ecfa
ms.sourcegitcommit: 62e1884457b64fd798da8ada59dbf623ef27fe97
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/26/2020
ms.locfileid: "88921994"
---
# <a name="how-to-scale-signalr-service-with-multiple-instances"></a>¿Cómo escalar SignalR Service con varias instancias?
El SDK más reciente de SignalR Service admite varios puntos de conexión para las instancias de SignalR Service. Puede usar esta característica para escalar las conexiones simultáneas, o usarla para la mensajería entre regiones.

## <a name="for-aspnet-core"></a>Para ASP.NET Core

### <a name="how-to-add-multiple-endpoints-from-config"></a>¿Cómo se agrega varios puntos de conexión desde la configuración?

Configuración con la clave `Azure:SignalR:ConnectionString` o `Azure:SignalR:ConnectionString:` para la cadena de conexión de SignalR Service.

Si la clave empieza por `Azure:SignalR:ConnectionString:`, debe estar en formato `Azure:SignalR:ConnectionString:{Name}:{EndpointType}`, donde `Name` y `EndpointType` son propiedades del objeto `ServiceEndpoint` y son accesibles desde el código.

Puede agregar varias cadenas de conexión de instancias con los siguientes comandos `dotnet`:

```batch
dotnet user-secrets set Azure:SignalR:ConnectionString:east-region-a <ConnectionString1>
dotnet user-secrets set Azure:SignalR:ConnectionString:east-region-b:primary <ConnectionString2>
dotnet user-secrets set Azure:SignalR:ConnectionString:backup:secondary <ConnectionString3>
```

### <a name="how-to-add-multiple-endpoints-from-code"></a>¿Cómo se agregan varios puntos de conexión desde el código?

Una clase `ServicEndpoint` se introdujo para describir las propiedades de un punto de conexión de Azure SignalR Service.
Puede configurar varios puntos de conexión de instancia al usar el SDK de Azure SignalR Service mediante:
```cs
services.AddSignalR()
        .AddAzureSignalR(options => 
        {
            options.Endpoints = new ServiceEndpoint[]
            {
                // Note: this is just a demonstration of how to set options.Endpoints
                // Having ConnectionStrings explicitly set inside the code is not encouraged
                // You can fetch it from a safe place such as Azure KeyVault
                new ServiceEndpoint("<ConnectionString0>"),
                new ServiceEndpoint("<ConnectionString1>", type: EndpointType.Primary, name: "east-region-a"),
                new ServiceEndpoint("<ConnectionString2>", type: EndpointType.Primary, name: "east-region-b"),
                new ServiceEndpoint("<ConnectionString3>", type: EndpointType.Secondary, name: "backup"),
            };
        });
```

### <a name="how-to-customize-endpoint-router"></a>¿Cómo se personaliza el enrutador de punto de conexión personalizado?

De forma predeterminada, el SDK usa [DefaultEndpointRouter](https://github.com/Azure/azure-signalr/blob/dev/src/Microsoft.Azure.SignalR/EndpointRouters/DefaultEndpointRouter.cs) para seleccionar los puntos de conexión.

#### <a name="default-behavior"></a>Comportamiento predeterminado 
1. Enrutamiento de solicitudes de cliente

    Cuando el cliente negocia (acción `/negotiate`) con el servidor de aplicaciones. De forma predeterminada, el SDK **selecciona aleatoriamente** un punto de conexión del conjunto de puntos de conexión de servicio disponibles.

2. Selección del enrutamiento de mensajes

    Al *enviar mensajes a una **conexión*** específica, si la conexión de destino se enruta al servidor actual, el mensaje va directamente a ese punto de conexión conectado. En caso contrario, los mensajes se difunden a cada punto de conexión de Azure SignalR.

#### <a name="customize-routing-algorithm"></a>Personalización del algoritmo de enrutamiento
Puede crear su propio enrutador cuando tenga conocimientos especiales para identificar los puntos de conexión a los que deberían ir los mensajes.

A continuación se define como ejemplo un enrutador personalizado cuando los grupos comienzan por `east-` ir siempre al punto de conexión denominado `east`:

```cs
private class CustomRouter : EndpointRouterDecorator
{
    public override IEnumerable<ServiceEndpoint> GetEndpointsForGroup(string groupName, IEnumerable<ServiceEndpoint> endpoints)
    {
        // Override the group broadcast behavior, if the group name starts with "east-", only send messages to endpoints inside east
        if (groupName.StartsWith("east-"))
        {
            return endpoints.Where(e => e.Name.StartsWith("east-"));
        }

        return base.GetEndpointsForGroup(groupName, endpoints);
    }
}
```

Este otro ejemplo, que invalida el comportamiento de negociación predeterminado para seleccionar los puntos de conexión, depende de dónde se encuentre el servidor de aplicaciones.

```cs
private class CustomRouter : EndpointRouterDecorator
{
    public override ServiceEndpoint GetNegotiateEndpoint(HttpContext context, IEnumerable<ServiceEndpoint> endpoints)
    {
        // Override the negotiate behavior to get the endpoint from query string
        var endpointName = context.Request.Query["endpoint"];
        if (endpointName.Count == 0)
        {
            context.Response.StatusCode = 400;
            var response = Encoding.UTF8.GetBytes("Invalid request");
            context.Response.Body.Write(response, 0, response.Length);
            return null;
        }

        return endpoints.FirstOrDefault(s => s.Name == endpointName && s.Online) // Get the endpoint with name matching the incoming request
               ?? base.GetNegotiateEndpoint(context, endpoints); // Or fallback to the default behavior to randomly select one from primary endpoints, or fallback to secondary when no primary ones are online
    }
}
```

No olvide registrar el enrutador en el contenedor de DI mediante:

```cs
services.AddSingleton(typeof(IEndpointRouter), typeof(CustomRouter));
services.AddSignalR()
        .AddAzureSignalR(
            options => 
            {
                options.Endpoints = new ServiceEndpoint[]
                {
                    new ServiceEndpoint(name: "east", connectionString: "<connectionString1>"),
                    new ServiceEndpoint(name: "west", connectionString: "<connectionString2>"),
                    new ServiceEndpoint("<connectionString3>")
                };
            });
```

## <a name="for-aspnet"></a>Para ASP.NET

### <a name="how-to-add-multiple-endpoints-from-config"></a>¿Cómo se agrega varios puntos de conexión desde la configuración?

Configuración con la clave `Azure:SignalR:ConnectionString` o `Azure:SignalR:ConnectionString:` para la cadena de conexión de SignalR Service.

Si la clave empieza por `Azure:SignalR:ConnectionString:`, debe estar en formato `Azure:SignalR:ConnectionString:{Name}:{EndpointType}`, donde `Name` y `EndpointType` son propiedades del objeto `ServiceEndpoint` y son accesibles desde el código.

Puede agregar varias cadenas de conexión de instancia a `web.config`:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <connectionStrings>
    <add name="Azure:SignalR:ConnectionString" connectionString="<ConnectionString1>"/>
    <add name="Azure:SignalR:ConnectionString:en-us" connectionString="<ConnectionString2>"/>
    <add name="Azure:SignalR:ConnectionString:zh-cn:secondary" connectionString="<ConnectionString3>"/>
    <add name="Azure:SignalR:ConnectionString:Backup:secondary" connectionString="<ConnectionString4>"/>
  </connectionStrings>
  ...
</configuration>
```

### <a name="how-to-add-multiple-endpoints-from-code"></a>¿Cómo se agregan varios puntos de conexión desde el código?

Una clase `ServicEndpoint` se introdujo para describir las propiedades de un punto de conexión de Azure SignalR Service.
Puede configurar varios puntos de conexión de instancia al usar el SDK de Azure SignalR Service mediante:

```cs
app.MapAzureSignalR(
    this.GetType().FullName, 
    options => {
            options.Endpoints = new ServiceEndpoint[]
            {
                // Note: this is just a demonstration of how to set options.Endpoints
                // Having ConnectionStrings explicitly set inside the code is not encouraged
                // You can fetch it from a safe place such as Azure KeyVault
                new ServiceEndpoint("<ConnectionString1>"),
                new ServiceEndpoint("<ConnectionString2>"),
                new ServiceEndpoint("<ConnectionString3>"),
            }
        });
```

### <a name="how-to-customize-router"></a>¿Cómo se personaliza el enrutador?

La única diferencia entre ASP.NET SignalR y ASP.NET Core SignalR es el tipo de contexto http para `GetNegotiateEndpoint`. En ASP.NET SignalR, es de tipo [IOwinContext](https://github.com/Azure/azure-signalr/blob/dev/src/Microsoft.Azure.SignalR.AspNet/EndpointRouters/DefaultEndpointRouter.cs#L19).

A continuación se muestra el ejemplo de negociación personalizada para ASP.NET SignalR:

```cs
private class CustomRouter : EndpointRouterDecorator
{
    public override ServiceEndpoint GetNegotiateEndpoint(IOwinContext context, IEnumerable<ServiceEndpoint> endpoints)
    {
        // Override the negotiate behavior to get the endpoint from query string
        var endpointName = context.Request.Query["endpoint"];
        if (string.IsNullOrEmpty(endpointName))
        {
            context.Response.StatusCode = 400;
            context.Response.Write("Invalid request.");
            return null;
        }

        return endpoints.FirstOrDefault(s => s.Name == endpointName && s.Online) // Get the endpoint with name matching the incoming request
               ?? base.GetNegotiateEndpoint(context, endpoints); // Or fallback to the default behavior to randomly select one from primary endpoints, or fallback to secondary when no primary ones are online
    }
}
```

No olvide registrar el enrutador en el contenedor de DI mediante:

```cs
var hub = new HubConfiguration();
var router = new CustomRouter();
hub.Resolver.Register(typeof(IEndpointRouter), () => router);
app.MapAzureSignalR(GetType().FullName, hub, options => {
    options.Endpoints = new ServiceEndpoint[]
                {
                    new ServiceEndpoint(name: "east", connectionString: "<connectionString1>"),
                    new ServiceEndpoint(name: "west", connectionString: "<connectionString2>"),
                    new ServiceEndpoint("<connectionString3>")
                };
});
```

## <a name="configuration-in-cross-region-scenarios"></a>Configuración en escenarios entre regiones

El objeto `ServiceEndpoint` tiene una propiedad `EndpointType` con valor `primary` o `secondary`.

Los puntos de conexión `primary` son los preferidos para recibir el tráfico de cliente y se considera que tienen conexiones de red más confiables; los puntos de conexión `secondary` se considera que tienen menos conexiones de red confiables y solo se usan para llevar el tráfico del servidor al cliente, por ejemplo, difundir mensajes, pero no del cliente al servidor.

En los casos entre regiones, la red puede ser inestable. Para un servidor de aplicaciones ubicado en *Este de EE. UU.*, el punto de conexión de SignalR Service ubicado en esta misma región puede configurarse como `primary` y los puntos de conexión de otras regiones marcarse como `secondary`. En esta configuración, los puntos de conexión de servicio de otras regiones pueden **recibir** mensajes de este servidor de aplicaciones de *Este de EE. UU.* , pero no habrá ningún cliente **entre regiones** enrutado a este servidor de aplicaciones. La arquitectura se muestra en el diagrama siguiente:

![Infraestructura entre regiones geográficas](./media/signalr-howto-scale-multi-instances/cross_geo_infra.png)

Cuando un cliente intenta una acción de negociación (`/negotiate`) con el servidor de aplicaciones, con el enrutador predeterminado, el SDK **selecciona aleatoriamente** un punto de conexión del conjunto de puntos de conexión `primary` disponibles. Cuando el punto de conexión principal está disponible, el SDK **selecciona aleatoriamente** de todos los puntos de conexión `secondary` disponibles. El punto de conexión se marca como **disponible** cuando la conexión entre el servidor y el punto de conexión de servicio está activa.

En el escenario entre regiones, cuando un cliente intenta la acción de negociación (`/negotiate`) con el servidor de aplicaciones hospedado en *Este de EE. UU.* , siempre devuelve de forma predeterminada el punto de conexión `primary` ubicado en la misma región. Cuando no todos los puntos de conexión de *Este de EE. UU.* están disponibles, el cliente se redirige a los puntos de conexión de otras regiones. En la siguiente sección de conmutación por error se describe detalladamente el escenario.

![Negociación normal](./media/signalr-howto-scale-multi-instances/normal_negotiate.png)

## <a name="fail-over"></a>Conmutación por error

Cuando no todos los puntos de conexión `primary` están disponibles, la acción de negociación (`/negotiate`) del cliente selecciona de entre los puntos de conexión `secondary` disponibles. Este mecanismo de conmutación por error requiere que cada punto de conexión sirva como punto de conexión `primary` para al menos un servidor de aplicaciones.

![Conmutación por error](./media/signalr-howto-scale-multi-instances/failover_negotiate.png)

## <a name="next-steps"></a>Pasos siguientes

En esta guía, ha aprendido a configurar varias instancias de la misma aplicación para escenarios de escalado, particionamiento y entre regiones.

La compatibilidad con varios puntos de conexión también puede usarse en escenarios de recuperación ante desastres y alta disponibilidad.

> [!div class="nextstepaction"]
> [Configuración de SignalR Service para la recuperación ante desastres y la alta disponibilidad](./signalr-concept-disaster-recovery.md)
