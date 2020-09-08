---
title: Migración de las aplicaciones de Managed Cache Service a Redis - Azure
description: Obtenga información sobre cómo migrar las aplicaciones de Managed Cache Service e In-Role Cache a Azure Cache for Redis
author: yegu-ms
ms.service: cache
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 07/23/2020
ms.author: yegu
ROBOTS: NOINDEX
ms.openlocfilehash: beb6014a9b6d90d1bc9a3c3236877a720a44a0c4
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88211112"
---
# <a name="migrate-from-managed-cache-service-to-azure-cache-for-redis-deprecated"></a>Migración desde Managed Cache Service a Azure Cache for Redis (en desuso)
La migración de aplicaciones que usan Azure Managed Cache Service a Azure Cache for Redis se puede realizar con unos cambios mínimos en la aplicación, según las características de Managed Cache Service usadas por la aplicación de almacenamiento en caché. Si bien las API no son exactamente iguales, son parecidas, y gran parte del código existente que usa Managed Cache Service para tener acceso a una caché se puede reutilizar con cambios mínimos. En este artículo se muestra cómo realizar los cambios de aplicación y configuración necesarios para migrar las aplicaciones de Managed Cache Service que usen Azure Cache for Redis y se indica cómo se pueden usar algunas de las características de Azure Cache for Redis para implementar la funcionalidad de una caché de una memoria caché de Managed Cache Service.

>[!NOTE]
>Managed Cache Service e In-Role Cache se [retiraron](https://azure.microsoft.com/blog/azure-managed-cache-and-in-role-cache-services-to-be-retired-on-11-30-2016/) el 30 de noviembre de 2016. Si desea migrar alguna implementación de In-Role Cache a Azure Cache for Redis, puede seguir los pasos que se proporcionan en este artículo.

## <a name="migration-steps"></a>Pasos de migración
A continuación se describen los pasos necesarios para migrar una aplicación de Managed Cache Service para usar Azure Cache for Redis.

* Asignación de características de Managed Cache Service a Azure Cache for Redis
* Elegir una oferta de caché
* Creación de una caché
* Configuración de los clientes de caché
  * Eliminación de la configuración de Managed Cache Service
  * Configuración de un cliente de caché mediante el paquete NuGet StackExchange.Redis
* Migración del código de Managed Cache Service
  * Conexión a la caché mediante la clase ConnectionMultiplexer
  * Acceder a tipos de datos primitivos en la caché
  * Trabajar con objetos .NET en la memoria caché
* Migrar el estado de sesión de ASP.NET y el almacenamiento en caché de resultados a Azure Cache for Redis 

## <a name="map-managed-cache-service-features-to-azure-cache-for-redis"></a>Asignación de características de Managed Cache Service a Azure Cache for Redis
Azure Managed Cache Service y Azure Cache for Redis guardan ciertas semejanzas pero implementan algunas de sus características de formas diferentes. En esta sección se describen algunas de las diferencias y se proporcionan instrucciones para la implementación de las características de Managed Cache Service en Azure Cache for Redis.

| Características de Managed Cache Service | Compatibilidad con Managed Cache Service | Compatibilidad con Azure Cache for Redis |
| --- | --- | --- |
| Cachés con nombre |Se configura una caché predeterminada y, en las ofertas de caché Estándar y Premium, se pueden configurar, si se desea, hasta nueve cachés con nombre adicionales. |Las instancias de Azure Cache for Redis tienen un número configurable de bases de datos (valor predeterminado: 16) que pueden utilizarse para implementar una funcionalidad similar a las memorias caché con nombre. Para más información, consulte [What are Redis databases?](cache-development-faq.md#what-are-redis-databases) (¿Qué son las bases de datos de Redis?) y [Configuración predeterminada del servidor Redis](cache-configure.md#default-redis-server-configuration). |
| Alta disponibilidad |Proporciona alta disponibilidad para los elementos de la caché en las ofertas de caché Estándar y Premium. Si los elementos se pierden debido a un error, aún se puede acceder a las copias de seguridad de los elementos en la caché. Las escrituras en la caché de réplica se realizan de manera sincrónica. |Existe alta disponibilidad en las ofertas de caché Estándar y Premio, que tienen una configuración de réplica-principal de dos nodos (cada partición de una caché Premium tiene un par de réplica-principal). Las escrituras en la réplica se realizan de forma asincrónica. Para más información, consulte [Precios de Azure Cache for Redis](https://azure.microsoft.com/pricing/details/cache/). |
| Notificaciones |Permite a los clientes recibir notificaciones asincrónicas cuando se producen diversas operaciones de caché en una caché con nombre. |Las aplicaciones cliente pueden usar pub/sub de Redis o [notificaciones de Keyspace](cache-configure.md#keyspace-notifications-advanced-settings) para lograr una funcionalidad similar a las notificaciones. |
| Caché local |Almacena una copia de los objetos en caché de forma local en el cliente para un acceso a un más rápido. |Las aplicaciones cliente deberán implementar esta funcionalidad mediante un diccionario o una estructura de datos parecida. |
| Directiva de expulsión |Ninguna o LRU. La directiva predeterminada es LRU. |Azure Cache for Redis admite las siguientes directivas de expulsión: volatile-lru, allkeys-lru, volatile-random, allkeys-random, volatile-ttl, noeviction. La directiva predeterminada es volatile-lru. Para obtener más información, consulte [Configuración de servidor predeterminada en Redis](cache-configure.md#default-redis-server-configuration). |
| Directiva de caducidad |La directiva de caducidad predeterminada es Absoluta y el intervalo de caducidad predeterminado es de diez minutos. También hay directivas variable y Nunca. |De forma predeterminada, los elementos de la caché no caducan, pero se puede configurar una caducidad por escritura mediante el uso de sobrecargas de conjunto de caché. |
| Regiones y etiquetado |Las regiones son subgrupos de elementos en caché. Las regiones también admiten la anotación de elementos en caché con cadenas descriptivas adicionales llamadas etiquetas. Además, ofrecen la posibilidad de realizar operaciones de búsqueda en elementos etiquetados en dicha región. Todos los elementos de una región se encuentran en un único nodo del clúster de caché. |Una instancia de Azure Cache for Redis consta de un único nodo (a menos que el clúster Redis esté habilitado), por lo que no se aplica el concepto de regiones de Managed Cache Service. Redis admite búsqueda y operaciones con caracteres comodín al recuperar claves, de modo que se pueden insertar etiquetas descriptivas en los nombres de clave y usarlas más adelante para recuperar los elementos. Para ver un ejemplo de la implementación de una solución de etiquetado con Redis, consulte [Implementing cache tagging with Redis](https://stackify.com/implementing-cache-tagging-redis/)(Implementación del etiquetado de caché con Redis). |
| Serialización |La caché administrada admite NetDataContractSerializer, BinaryFormatter y el uso de serializadores personalizados. El valor predeterminado es NetDataContractSerializer. |Es responsabilidad de la aplicación cliente serializar los objetos .NET antes de colocarlos en la caché y dejar en manos del desarrollador de la aplicación cliente la elección del serializador. Para obtener más información y ejemplos de código, consulte [Objetos .NET en la caché](cache-dotnet-how-to-use-azure-redis-cache.md#work-with-net-objects-in-the-cache). |
| Emulador de caché |El servicio de caché administrado ofrece un emulador de la memoria caché local. |Azure Cache for Redis no tiene un emulador, pero puede [ejecutar Redis localmente](cache-development-faq.md#is-there-a-local-emulator-for-azure-cache-for-redis) para proporcionar una experiencia de emulador. |

## <a name="choose-a-cache-offering"></a>Elegir una oferta de caché
Microsoft Azure Cache for Redis está disponible en los niveles siguientes:

* **Básico** – Nodo único. Varios tamaños de hasta 53 GB.
* **Estándar**: principal/réplica de dos nodos. Varios tamaños de hasta 53 GB. Contrato de nivel de servicio del 99,9 %.
* **Premium** : principal/réplica de dos nodos con hasta 10 particiones. Varios tamaños, de 6 GB a 1,2 GB. Todas las características del nivel Estándar y algunas otras características son compatibles con los [clústeres de Redis](cache-how-to-premium-clustering.md), la [persistencia de Redis](cache-how-to-premium-persistence.md) y [Azure Virtual Network](cache-how-to-premium-vnet.md). Contrato de nivel de servicio del 99,9 %.

Estos niveles difieren en las características y el precio. Las características se tratan más adelante en esta guía; por otro lado, para obtener más información acerca de los precios consulte [Detalles de precios de caché](https://azure.microsoft.com/pricing/details/cache/).

Un punto de partida para la migración es seleccionar el tamaño que coincida con el tamaño anterior de la caché de Managed Cache Service, y luego escalar o reducir verticalmente en función de los requisitos de la aplicación. Para más información sobre cómo elegir la oferta apropiada de Azure Cache for Redis, consulte [Elección del nivel de servicios correcto](cache-overview.md#choosing-the-right-tier).

## <a name="create-a-cache"></a>Creación de una caché
[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

## <a name="configure-the-cache-clients"></a>Configuración de los clientes de caché
Después de crear y configurar la caché, el siguiente paso consiste en eliminar la configuración de Managed Cache Service y agregar la configuración de Azure Cache for Redis, junto con referencias para que los clientes de caché puedan acceder a ella.

* Eliminación de la configuración de Managed Cache Service
* Configuración de un cliente de caché mediante el paquete NuGet StackExchange.Redis

### <a name="remove-the-managed-cache-service-configuration"></a>Eliminación de la configuración de Managed Cache Service
Para que las aplicaciones cliente se puedan configurar para Azure Cache for Redis, se debe eliminar la configuración y las referencias de ensamblado existentes de Managed Cache Service, desinstalando el paquete NuGet de Managed Cache Service.

Para desinstalar el paquete de NuGet de Managed Cache Service, haga clic con el botón derecho en el proyecto de cliente en el **Explorador de soluciones** y elija **Administrar paquetes de NuGet**. Seleccione el nodo **Paquetes instalados** y escriba **WindowsAzure.Caching** en el cuadro de búsqueda de paquetes instalados. Seleccione **Windows** **Azure Cache** (o **Windows** **Azure Caching** en función de la versión del paquete NuGet), haga clic en **Desinstalar** y, después, haga clic en **Cerrar**.

![Desinstalar paquetes NuGet de Azure Managed Cache Service](./media/cache-migrate-to-redis/IC757666.jpg)

Al desinstalar el paquete NuGet de Managed Cache Service se eliminan los ensamblados de Managed Cache Service y sus entradas en app.config o web.config de la aplicación cliente. Como no es posible eliminar algunos valores de configuración personalizados al desinstalar el paquete NuGet, abra web.config o app.config, y asegúrese de que los siguientes elementos se han eliminado.

Asegúrese de que la entrada `dataCacheClients` se ha eliminado del elemento `configSections`. No quite el elemento `configSections` entero; solo la entrada `dataCacheClients`, si existe.

```xml
<configSections>
  <!-- Existing sections omitted for clarity. -->
  <section name="dataCacheClients"type="Microsoft.ApplicationServer.Caching.DataCacheClientsSection, Microsoft.ApplicationServer.Caching.Core" allowLocation="true" allowDefinition="Everywhere"/>
</configSections>
```

Asegúrese de que la sección `dataCacheClients` esté eliminada. La sección `dataCacheClients` se parecerá al ejemplo siguiente.

```xml
<dataCacheClients>
  <dataCacheClientname="default">
    <!--To use the in-role flavor of Azure Cache, set identifier to be the cache cluster role name -->
    <!--To use the Azure Managed Cache Service, set identifier to be the endpoint of the cache cluster -->
    <autoDiscoverisEnabled="true"identifier="[Cache role name or Service Endpoint]"/>

    <!--<localCache isEnabled="true" sync="TimeoutBased" objectCount="100000" ttlValue="300" />-->
    <!--Use this section to specify security settings for connecting to your cache. This section is not required if your cache is hosted on a role that is a part of your cloud service. -->
    <!--<securityProperties mode="Message" sslEnabled="true">
      <messageSecurity authorizationInfo="[Authentication Key]" />
    </securityProperties>-->
  </dataCacheClient>
</dataCacheClients>
```

Después de quitar la configuración de Managed Cache Service, puede configurar el cliente de caché como se describe en la siguiente sección.

### <a name="configure-a-cache-client-using-the-stackexchangeredis-nuget-package"></a>Configuración de un cliente de caché mediante el paquete NuGet StackExchange.Redis
[!INCLUDE [redis-cache-configure](../../includes/redis-cache-configure-stackexchange-redis-nuget.md)]

## <a name="migrate-managed-cache-service-code"></a>Migración del código de Managed Cache Service
La API del cliente de caché StackExchange.Azure Cache for Redis es parecida a Managed Cache Service. En esta sección se proporciona un resumen de las diferencias.

### <a name="connect-to-the-cache-using-the-connectionmultiplexer-class"></a>Conexión a la caché mediante la clase ConnectionMultiplexer
En Managed Cache Service, las conexiones a la caché se controlan mediante las clases `DataCacheFactory` y `DataCache`. En Azure Cache for Redis, estas conexiones se administran con la clase `ConnectionMultiplexer`.

Agregue la siguiente instrucción using en la parte superior de cualquier archivo desde el que desea obtener acceso a la caché.

```csharp
using StackExchange.Redis
```

Si no se resuelve este espacio de nombres, asegúrese de que ha agregado el paquete NuGet StackExchange.Redis, como se describe en [Inicio rápido: Uso de Azure Cache for Redis con una aplicación .NET](cache-dotnet-how-to-use-azure-redis-cache.md).

> [!NOTE]
> Tenga en cuenta que el cliente StackExchange.Redis requiere .NET Framework 4 o superior.
> 
> 

Para conectarse a una instancia de Azure Cache for Redis, llame a método estático `ConnectionMultiplexer.Connect` y pase el punto de conexión y la clave. Un enfoque para compartir una instancia de `ConnectionMultiplexer` en su aplicación es tener una propiedad estática que devuelva una instancia conectada, como en el ejemplo siguiente. Este enfoque proporciona una manera segura para subprocesos de inicializar una sola instancia de `ConnectionMultiplexer` conectada. En este ejemplo, `abortConnect` se establece en false, lo que significa que la llamada se realizará correctamente incluso si no se establece una conexión a la caché. Una de las características principales de `ConnectionMultiplexer` es que restaura automáticamente la conectividad a la caché una vez que el problema de red u otras causas se resuelven.

```csharp
private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
{
    return ConnectionMultiplexer.Connect("contoso5.redis.cache.windows.net,abortConnect=false,ssl=true,password=...");
});

public static ConnectionMultiplexer Connection
{
    get
    {
        return lazyConnection.Value;
    }
}
```

El punto de conexión, las claves y los puertos de la caché se pueden obtener de la hoja **Azure Cache for Redis** de su instancia de caché. Para más información, consulte [Propiedades de Azure Cache for Redis](cache-configure.md#properties).

Una vez establecida la conexión, devuelva una referencia a la base de datos de Azure Cache for Redis llamando al método `ConnectionMultiplexer.GetDatabase`. El objeto devuelto desde el método `GetDatabase` es un objeto de paso a través ligero y no necesita almacenarse.

```csharp
IDatabase cache = Connection.GetDatabase();

// Perform cache operations using the cache object...
// Simple put of integral data types into the cache
cache.StringSet("key1", "value");
cache.StringSet("key2", 25);

// Simple get of data types from the cache
string key1 = cache.StringGet("key1");
int key2 = (int)cache.StringGet("key2");
```

El cliente StackExchange.Redis usa los tipos `RedisKey` y `RedisValue` para obtener acceso y almacenar elementos en la caché. Estos tipos se asignan a tipos de lenguajes más primitivos, como la cadena, y no suelen usarse directamente. Las cadenas de Redis son el tipo más básico de valor de Redis y pueden contener muchos tipos de datos, como secuencias binarias serializadas. Aunque es posible que no use el tipo directamente, usará los métodos que contienen `String` en el nombre. Para los tipos de datos más primitivos, los elementos se almacenan y recuperan de la caché mediante los métodos `StringSet` y `StringGet`, a menos que vaya a almacenar colecciones u otros tipos de datos de Redis en la caché. 

`StringSet` y `StringGet` se parecen a los métodos `Put` y `Get` de Managed Cache Service, siendo una de las diferencias principales que antes de establecer y obtener un objeto .NET en la caché se debe serializar primero. 

Cuando llame a `StringGet`, si el objeto existe, se devuelve y, si no existe, se devuelve null. En este caso, puede recuperar el valor desde el origen de datos que quiera y almacenarlo en la memoria caché para su uso posterior. Este patrón se conoce como patrón cache-aside.

Para especificar la expiración de un elemento en la memoria caché, use el parámetro `TimeSpan` de `StringSet`.

```csharp
cache.StringSet("key1", "value1", TimeSpan.FromMinutes(90));
```

Azure Cache for Redis puede trabajar con objetos .NET, así como con tipos de datos primitivos; sin embargo, antes de poder almacenar en caché un objeto .NET, se debe serializar. Esta serialización es responsabilidad del desarrollador de la aplicación, que tiene total flexibilidad a la hora de elegir el serializador. Para obtener más información y ejemplos de código, consulte [Objetos .NET en la caché](cache-dotnet-how-to-use-azure-redis-cache.md#work-with-net-objects-in-the-cache).

## <a name="migrate-aspnet-session-state-and-output-caching-to-azure-cache-for-redis"></a>Migrar el estado de sesión de ASP.NET y el almacenamiento en caché de resultados a Azure Cache for Redis
Azure Cache for Redis cuenta con proveedores de estado de sesión ASP.NET y almacenamiento en caché de los resultados de las páginas. Para migrar la aplicación que usa versiones de estos proveedores de Managed Cache Service, elimine primero las secciones existentes de web.config, y luego configure las versiones de Azure Cache for Redis de los proveedores. Para instrucciones sobre cómo usar los proveedores de ASP.NET de Azure Cache for Redis, consulte [Proveedor de estado de sesión de ASP.NET para Azure Cache for Redis](cache-aspnet-session-state-provider.md) y [Proveedor de caché de resultados de ASP.NET para Azure Cache for Redis](cache-aspnet-output-cache-provider.md).

## <a name="next-steps"></a>Pasos siguientes
Explore la [documentación de Azure Cache for Redis](https://azure.microsoft.com/documentation/services/cache/) donde encontrará tutoriales, ejemplos, vídeos y mucho más.

