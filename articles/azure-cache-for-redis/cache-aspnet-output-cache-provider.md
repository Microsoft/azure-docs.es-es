---
title: Proveedor de caché de resultados de ASP.NET para Azure Cache for Redis
description: Obtenga información acerca de cómo almacenar en caché los resultados de página de ASP.NET con Azure Cache for Redis. El proveedor de la caché de salida de Redis es un mecanismo de almacenamiento fuera de proceso para los datos de la caché de salida.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.custom: devx-track-csharp
ms.topic: conceptual
ms.date: 04/22/2018
ms.openlocfilehash: cb986f1e0172c4a44381b2f9cf043025cb5abe8a
ms.sourcegitcommit: e0785ea4f2926f944ff4d65a96cee05b6dcdb792
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88705034"
---
# <a name="aspnet-output-cache-provider-for-azure-cache-for-redis"></a>Proveedor de caché de resultados de ASP.NET para Azure Cache for Redis

El proveedor de la caché de salida de Redis es un mecanismo de almacenamiento fuera de proceso para los datos de la caché de salida. Estos datos resultan necesarios específicamente para respuestas HTTP completas (caché de resultados de la página). El proveedor se conecta al nuevo punto de extensibilidad del proveedor de caché de salida que se introdujo en ASP.NET 4. En el caso de las aplicaciones ASP.NET Core, lea [Almacenamiento en caché de respuestas en ASP.NET Core](https://docs.microsoft.com/aspnet/core/performance/caching/response). 

Para usar el proveedor de la caché de salida de Redis, configure primero la caché y luego configure la aplicación de ASP.NET mediante el paquete NuGet del proveedor de la caché de salida de Redis. En este tema se proporcionan instrucciones sobre cómo configurar la aplicación para usar el proveedor de la caché de salida de Redis. Para más información acerca de cómo crear y configurar una instancia de Azure Cache for Redis, consulte [Creación de una caché](cache-dotnet-how-to-use-azure-redis-cache.md#create-a-cache).

## <a name="store-aspnet-page-output-in-the-cache"></a>Almacenamiento de los resultados de página de ASP.NET en la caché

Para configurar una aplicación cliente en Visual Studio usando el paquete NuGet de estado de sesión de Azure Cache for Redis, en el menú **Herramientas**, haga clic en **Administrador de paquetes NuGet** y en **Consola del Administrador de paquetes**.

Ejecute el siguiente comando desde la ventana `Package Manager Console`.

```
Install-Package Microsoft.Web.RedisOutputCacheProvider
```

El paquete NuGet del proveedor de la caché de salida de Redis tiene una dependencia del paquete StackExchange.Redis.StrongName. Si el paquete StackExchange.Redis.StrongName no existe en el proyecto, se instalará. Para obtener más información sobre el paquete de NuGet del proveedor de caché de salida de Redis, vea la página [RedisOutputCacheProvider](https://www.nuget.org/packages/Microsoft.Web.RedisOutputCacheProvider/) de NuGet.

>[!NOTE]
>Tenga en cuenta que, además del paquete StackExchange.Redis.StrongName con nombre seguro, también está la versión con nombre no seguro de StackExchange.Redis. Si su proyecto está utilizando la versión de StackExchange.Redis con nombre no seguro, deberá desinstalarla, ya que, de lo contrario, se producirán conflictos de nomenclatura en el proyecto. Para obtener más información sobre estos paquetes, consulte [Configuración de los clientes de la caché de .NET](cache-dotnet-how-to-use-azure-redis-cache.md#configure-the-cache-clients).

El paquete NuGet descarga y agrega las referencias de ensamblado necesarias y agrega la siguiente sección al archivo web.config. Esta sección contiene la configuración necesaria para que la aplicación ASP.NET use el proveedor de memoria caché de salida de Redis.

```xml
<caching>
  <outputCache defaultProvider="MyRedisOutputCache">
    <providers>
      <add name="MyRedisOutputCache" type="Microsoft.Web.Redis.RedisOutputCacheProvider"
           host=""
           accessKey=""
           ssl="true" />
    </providers>
  </outputCache>
</caching>
```

Configure los atributos con los valores de la hoja de la caché en el Portal de Microsoft Azure y configure los demás valores según prefiera. Para obtener instrucciones acerca de cómo acceder a las propiedades de la caché, consulte [Configuración de Azure Cache for Redis](cache-configure.md#configure-azure-cache-for-redis-settings).

| Atributo | Tipo | Valor predeterminado | Descripción |
| --------- | ---- | ------- | ----------- |
| *host* | string | "localhost" | Nombre de host o dirección del IP del servidor de Redis |
| *port* | entero positivo | 6379 (non-TLS/SSL)<br/>6380 (TLS/SSL) | Puerto del servidor de Redis |
| *accessKey* | string | "" | Contraseña del servidor de Redis con la autorización de Redis habilitada. El valor es una cadena vacía de forma predeterminada, lo que significa que el proveedor de estado de sesión no utilizará ninguna contraseña al conectarse al servidor de Redis. **Si el servidor de Redis está en una red de acceso público como Azure Redis Cache, asegúrese de habilitar la autorización de Redis mejorar la seguridad y proporcione una contraseña segura.** |
| *ssl* | boolean | **false** | Si desea conectarse al servidor de Redis a través de TLS. Este valor es **false** de forma predeterminada, dado que Redis no admite TLS inicialmente. **Si usa Azure Redis Cache que admita SSL inicialmente, no olvide establecerlo en true para mejorar la seguridad.**<br/><br/>El puerto no TLS está deshabilitado de forma predeterminada para las memorias caché nuevas. Especifique **true** en este valor para usar el puerto que no es TLS. Para más información sobre cómo habilitar el puerto no TLS, vea la sección [Puertos de acceso](cache-configure.md#access-ports) del tema de [Configuración de caché](cache-configure.md). |
| *databaseIdNumber* | entero positivo | 0 | *Este atributo se puede especificar únicamente mediante web.config o AppSettings.*<br/><br/>Especifique qué base de datos de Redis usar. |
| *connectionTimeoutInMilliseconds* | entero positivo | Suministrado por StackExchange.Redis | Se usa para establecer *ConnectTimeout* al crear StackExchange.Redis.ConnectionMultiplexer. |
| *operationTimeoutInMilliseconds* | entero positivo | Suministrado por StackExchange.Redis | Se usa para establecer *SyncTimeout* al crear StackExchange.Redis.ConnectionMultiplexer. |
| *connectionString* (cadena de conexión StackExchange.Redis válida) | string | *n/a* | Una referencia de parámetro a AppSettings o web.config, o una cadena de conexión de StackExchange.Redis válida. Este atributo puede proporcionar valores para *host*, *port*, *accessKey*, *ssl* y otros atributos de StackExchange.Redis. Para más detalles de *connectionString*, consulte [Valor de connectionString](#setting-connectionstring) en la sección [Notas sobre los atributos](#attribute-notes). |
| *settingsClassName*<br/>*settingsMethodName* | string<br/>string | *n/a* | *Estos atributos se pueden especificar únicamente mediante web.config o AppSettings.*<br/><br/>Use estos atributos para proporcionar una cadena de conexión. *settingsClassName* debe ser un nombre de clase completo del ensamblado que contenga el método especificado por *settingsMethodName*.<br/><br/>El método especificado por *settingsMethodName* debe ser público, estático y void (que no toma ningún parámetro), con un tipo **cadena** de valor devuelto. Este método devuelve la cadena de conexión real. |
| *loggingClassName*<br/>*loggingMethodName* | string<br/>string | *n/a* | *Estos atributos se pueden especificar únicamente mediante web.config o AppSettings.*<br/><br/>Use estos atributos para depurar la aplicación proporcionando los registros desde la caché de estado de sesión y salida, junto con los registros de StackExchange.Redis. *loggingClassName* debe ser un nombre de clase completo del ensamblado que contenga el método especificado por *loggingMethodName*.<br/><br/>El método especificado por *loggingMethodName* debe ser público, estático y void (que no toma ningún parámetro), con un tipo **System.IO.TextWriter** de valor devuelto. |
| *applicationName* | string | El nombre del módulo del proceso actual o "/" | *SessionStateProvider only*<br/>*Este atributo se puede especificar únicamente mediante web.config o AppSettings.*<br/><br/>El prefijo del nombre de aplicación que usar en la memoria caché de Redis. El cliente podría usar la misma caché de Redis para distintos fines. Para garantizar que las claves de sesión no entren en conflicto, se puede tener como prefijo el nombre de la aplicación. |
| *throwOnError* | boolean | true | *SessionStateProvider only*<br/>*Este atributo se puede especificar únicamente mediante web.config o AppSettings.*<br/><br/>Si se produce una excepción cuando se produce un error.<br/><br/>Para más información acerca de *throwOnError*, consulte [Notas sobre *throwOnError*](#notes-on-throwonerror) en la sección [Notas sobre los atributos](#attribute-notes). |>*Microsoft.Web.Redis.RedisSessionStateProvider.LastException*. |
| *retryTimeoutInMilliseconds* | entero positivo | 5000 | *SessionStateProvider only*<br/>*Este atributo se puede especificar únicamente mediante web.config o AppSettings.*<br/><br/>Tiempo para volver a intentarlo cuando falla una operación. Si este valor es menor que *operationTimeoutInMilliseconds*, el proveedor no volverá a intentarlo.<br/><br/>Para más información acerca de *retryTimeoutInMilliseconds*, consulte [Notas sobre *retryTimeoutInMilliseconds*](#notes-on-retrytimeoutinmilliseconds) en la sección [Notas sobre los atributos](#attribute-notes). |
| *redisSerializerType* | string | *n/a* | Especifica el nombre de tipo calificado de ensamblado de una clase que implementa Microsoft.Web.Redis. ISerializer y que contiene la lógica personalizada para serializar y deserializar los valores. Para más información, consulte [Acerca de *redisSerializerType*](#about-redisserializertype) en la sección [Notas sobre los atributos](#attribute-notes). |

## <a name="attribute-notes"></a>Notas sobre los atributos

### <a name="setting-connectionstring"></a>Valor de *connectionString*

El valor de *connectionString* se utiliza como clave para capturar la cadena de conexión real de AppSettings (si este tipo de cadena existe en AppSettings). Si no se encuentra en AppSettings, el valor de *connectionString* se utilizará como clave para recuperar la cadena de conexión real desde la sección **ConnectionString** de web.config (si existe esa sección). Si la cadena de conexión no existe en AppSettings o la sección **ConnectionString** de web.config, se usará el valor literal de *connectionString* como cadena de conexión al crear StackExchange.Redis.ConnectionMultiplexer.

Los siguientes ejemplos muestran cómo se utiliza *connectionString*.

#### <a name="example-1"></a>Ejemplo 1

```xml
<connectionStrings>
    <add name="MyRedisConnectionString" connectionString="mycache.redis.cache.windows.net:6380,password=actual access key,ssl=True,abortConnect=False" />
</connectionStrings>
```

En `web.config`, utilice la clave anterior como valor del parámetro en lugar del valor real.

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
    <providers>
        <add type = "Microsoft.Web.Redis.RedisSessionStateProvide"
             name = "MySessionStateStore"
             connectionString = "MyRedisConnectionString"/>
    </providers>
</sessionState>
```

#### <a name="example-2"></a>Ejemplo 2

```xml
<appSettings>
    <add key="MyRedisConnectionString" value="mycache.redis.cache.windows.net:6380,password=actual access key,ssl=True,abortConnect=False" />
</appSettings>
```

En `web.config`, utilice la clave anterior como valor del parámetro en lugar del valor real.

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
    <providers>
        <add type = "Microsoft.Web.Redis.RedisSessionStateProvide"
             name = "MySessionStateStore"
             connectionString = "MyRedisConnectionString"/>
    </providers>
</sessionState>
```

#### <a name="example-3"></a>Ejemplo 3

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
    <providers>
        <add type = "Microsoft.Web.Redis.RedisSessionStateProvide"
             name = "MySessionStateStore"
             connectionString = "mycache.redis.cache.windows.net:6380,password=actual access key,ssl=True,abortConnect=False"/>
    </providers>
</sessionState>
```

### <a name="notes-on-throwonerror"></a>Notas sobre *throwOnError*

Actualmente, si se produce un error durante una operación de sesión, el proveedor de estado de sesión produce una excepción. Esto hace que la aplicación se apague.

Este comportamiento se modificó de manera que sea compatible con las expectativas de los usuarios existentes del proveedor de estado de sesión de ASP.NET al tiempo que proporciona también la capacidad de actuar sobre las excepciones, si así se desea. El comportamiento predeterminado todavía produce una excepción cuando se produce un error, de manera coherente con otros proveedores de estado de sesión de ASP.NET; el código existente debería funcionar igual que antes.

Si establece *throwOnError* en **false**, en lugar de producir una excepción cuando se produce un error, se produce en modo silencioso. Para ver si se produjo un error y, si es así, detectar la excepción, compruebe la propiedad estática *Microsoft.Web.Redis.RedisSessionStateProvider.LastException*.

### <a name="notes-on-retrytimeoutinmilliseconds"></a>Notas sobre *retryTimeoutInMilliseconds*

Esto proporciona lógica de reintento para simplificar la situación de que alguna operación de sesión deba reintentar en caso de error debido a cosas como problemas de red, mientras que también permite controlar el tiempo de espera de reintento o dejar reintentar completamente.

Si establece *retryTimeoutInMilliseconds* en un número, por ejemplo, 2000, cuando se produzca un error en una operación de sesión, se vuelve a intentar durante 2000 milisegundos antes de tratarlo como error. Por lo tanto, para que el proveedor de estado de sesión aplique esta lógica de reintento, solo tiene que configurar el tiempo de espera. Cuando se produzca un problema de red, el primer reintento se realizará a los 20 milisegundos, tiempo suficiente en la mayoría de los casos. Después de eso, volverá a intentarlo cada segundo hasta que se agote el tiempo. Justo después del tiempo de espera, volverá a intentarlo para asegurarse de que no se corta el tiempo de espera por un segundo (como máximo).

Si cree que no necesita volver a intentar (por ejemplo, cuando se está ejecutando el servidor de Redis en la misma máquina que la aplicación) o si desea controlar la lógica de reintento personalmente, establezca *retryTimeoutInMilliseconds* en 0.

### <a name="about-redisserializertype"></a>Acerca de *redisSerializerType*

De forma predeterminada, la serialización para almacenar los valores en Redis se realiza en un formato binario proporcionado por la clase **BinaryFormatter**. Use *redisSerializerType* para especificar el nombre del tipo calificado de ensamblado de una clase que implemente **Microsoft.Web.Redis.ISerializer** y tenga la lógica personalizada para serializar y deserializar los valores. Por ejemplo, esta es una clase de serializador de Json con JSON.NET:

```cs
namespace MyCompany.Redis
{
    public class JsonSerializer : ISerializer
    {
        private static JsonSerializerSettings _settings = new JsonSerializerSettings() { TypeNameHandling = TypeNameHandling.All };

        public byte[] Serialize(object data)
        {
            return Encoding.UTF8.GetBytes(JsonConvert.SerializeObject(data, _settings));
        }

        public object Deserialize(byte[] data)
        {
            if (data == null)
            {
                return null;
            }
            return JsonConvert.DeserializeObject(Encoding.UTF8.GetString(data), _settings);
        }
    }
}
```

Supongamos que esta clase se define en un ensamblado con nombre **MyCompanyDll**, puede establecer el parámetro *redisSerializerType* para usarlo:

```xml
<sessionState mode="Custom" customProvider="MySessionStateStore">
    <providers>
        <add type = "Microsoft.Web.Redis.RedisSessionStateProvider"
             name = "MySessionStateStore"
             redisSerializerType = "MyCompany.Redis.JsonSerializer,MyCompanyDll"
             ... />
    </providers>
</sessionState>
```

## <a name="output-cache-directive"></a>Directiva de caché de salida

Incorpore una directiva OutputCache a cada página cuyos resultados desea almacenar en caché.

```xml
<%@ OutputCache Duration="60" VaryByParam="*" %>
```

En el ejemplo anterior, los datos de la página almacenados en la memoria caché permanecerán ahí durante 60 segundos y se almacenará en la memoria caché una versión diferente de la página para cada combinación de parámetros. Para obtener más información sobre la directiva OutputCache, consulte [@OutputCache](https://go.microsoft.com/fwlink/?linkid=320837).

Después de realizar estos pasos, la aplicación está configurada para usar el proveedor de la caché de salida de Redis.

## <a name="third-party-output-cache-providers"></a>Proveedores de caché de resultados de terceros

* [NCache](https://www.alachisoft.com/blogs/how-to-use-a-distributed-cache-for-asp-net-output-cache/)
* [Apache Ignite](https://apacheignite-net.readme.io/docs/aspnet-output-caching)


## <a name="next-steps"></a>Pasos siguientes

Consulte el [proveedor de estado de sesión de ASP.NET para Azure Cache for Redis](cache-aspnet-session-state-provider.md).
