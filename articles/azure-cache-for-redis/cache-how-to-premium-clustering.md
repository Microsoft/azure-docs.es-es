---
title: 'Configuración de la agrupación en clústeres de Redis: Azure Cache for Redis Premium'
description: Obtener información sobre cómo crear y administrar la agrupación en clústeres de para sus instancias de Azure Cache for Redis de nivel Prémium
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 06/13/2018
ms.openlocfilehash: d37aa275a07586738bf7416cee6611bdc8284df3
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88004773"
---
# <a name="how-to-configure-redis-clustering-for-a-premium-azure-cache-for-redis"></a>Configuración de la agrupación en clústeres de Redis para una instancia de Azure Cache for Redis de nivel Prémium
Azure Cache for Redis cuenta con diferentes opciones de caché, lo que proporciona flexibilidad en la elección del tamaño y las características de la memoria caché, incluidas algunas características del nivel Prémium, como la agrupación en clústeres, la persistencia y la compatibilidad con las redes virtuales. En este artículo se describe cómo configurar la agrupación en clústeres en una instancia de Azure Cache for Redis de nivel Prémium.

## <a name="what-is-redis-cluster"></a>¿Qué es Clúster Redis?
Azure Cache for Redis ofrece clúster de Redis como [implementado en Redis](https://redis.io/topics/cluster-tutorial). Con el Clúster de Redis, obtendrá las siguientes ventajas: 

* La capacidad de dividir automáticamente el conjunto de datos entre varios nodos. 
* La capacidad de continuar las operaciones cuando un subconjunto de los nodos está teniendo errores o no se puede comunicar con el resto del clúster. 
* Mayor rendimiento: el rendimiento aumenta de manera lineal a medida que aumenta el número de particiones. 
* Más tamaño de memoria: aumenta de manera lineal a medida que aumenta el número de particiones.  

La agrupación en clústeres no aumenta el número de conexiones disponibles para una caché en clúster. Para obtener más información acerca del tamaño, la transferencia y el ancho de banda de las memorias caché de nivel Premium, consulte [Elección del nivel correcto](cache-overview.md#choosing-the-right-tier).

En Azure, el clúster de Redis se ofrece como un modelo de principal/réplica donde cada partición tiene un par de principal/réplica con la replicación donde la replicación se administra mediante el servicio Azure Cache for Redis. 

## <a name="clustering"></a>Agrupación en clústeres
La agrupación en clústeres se habilita en la hoja **New Azure Cache for Redis** (Nuevo Azure Cache for Redis) durante la creación de la memoria caché. 

[!INCLUDE [redis-cache-create](../../includes/redis-cache-premium-create.md)]

La agrupación en clústeres se configura en la hoja **Clúster de Redis** .

![Agrupación en clústeres][redis-cache-clustering]

Puede tener hasta 10 particiones en el clúster. Haga clic en **Habilitado** y deslice el control deslizante o escriba un número comprendido entre 1 y 10 para **Número de particiones** y haga clic en **Aceptar**.

Cada partición es un par de caché principal/réplica administrado por Azure y el tamaño total de la memoria caché se calcula multiplicando el número de particiones por el tamaño de la memoria caché seleccionado en el plan de tarifa. 

![Agrupación en clústeres][redis-cache-clustering-selected]

Cuando se haya creado la memoria caché, conéctese a ella y úsela simplemente como una memoria caché no agrupada y Redis distribuirá los datos a través de las particiones de memoria caché. Si el diagnóstico está [habilitado](cache-how-to-monitor.md#enable-cache-diagnostics), las métricas se capturan por separado para cada partición y pueden [verse](cache-how-to-monitor.md) en la hoja Azure Cache for Redis. 

> [!NOTE]
> 
> Hay algunas diferencias menores necesarias en la aplicación cliente cuando se configura la agrupación en clústeres. Para más información, consulte [Configuración de Caché en Redis de Azure](#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)
> 
> 

Para obtener el código de ejemplo sobre el trabajo con clústeres con el cliente StackExchange.Redis, consulte la parte [clustering.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/Clustering.cs) del ejemplo [Hello World](https://github.com/rustd/RedisSamples/tree/master/HelloWorld).

<a name="cluster-size"></a>

## <a name="change-the-cluster-size-on-a-running-premium-cache"></a>Cambio del tamaño del clúster en una caché premium en ejecución
Para cambiar el tamaño del clúster de una caché premium en ejecución con la agrupación en clústeres habilitada, haga clic en **Tamaño del clúster** desde el **menú Recursos**.

![Tamaño del Clúster en Redis][redis-cache-redis-cluster-size]

Para modificar el tamaño del clúster, utilice el control deslizante o especifique un número comprendido entre 1 y 10 en el cuadro de texto **Número de particiones**. Después, haga clic en **Aceptar** para guardar.

Aumentar el tamaño del clúster aumenta el rendimiento máximo y el tamaño de caché. Aumentar el tamaño del clúster no aumenta las conexiones máximas disponibles para los clientes.

> [!NOTE]
> El escalado de un clúster ejecuta el comando [MIGRATE](https://redis.io/commands/migrate), que es un comando caro. Por lo tanto, para un impacto mínimo, considere ejecutar esta operación durante horas de poca actividad. Durante el proceso de migración, verá un pico de carga del servidor. El escalado de un clúster es un proceso de ejecución duradero y la cantidad de tiempo transcurrido depende del número de claves y el tamaño de los valores asociados a estas claves.
> 
> 

## <a name="clustering-faq"></a>P+F de agrupación en clústeres
La lista siguiente contiene respuestas a las preguntas frecuentes sobre la agrupación en clústeres de Azure Cache for Redis.

* [¿Es necesario realizar algún cambio en mi aplicación cliente para usar la agrupación en clústeres?](#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)
* [¿Cómo se distribuyen las claves en un clúster?](#how-are-keys-distributed-in-a-cluster)
* [¿Cuál es el mayor tamaño de caché que puedo crear?](#what-is-the-largest-cache-size-i-can-create)
* [¿Todos los clientes de Redis admiten la agrupación en clústeres?](#do-all-redis-clients-support-clustering)
* [¿Cómo me conecto a mi memoria caché cuando la agrupación en clústeres esté habilitada?](#how-do-i-connect-to-my-cache-when-clustering-is-enabled)
* [¿Puedo conectarme directamente a las particiones individuales de mi memoria caché?](#can-i-directly-connect-to-the-individual-shards-of-my-cache)
* [¿Puedo configurar la agrupación en clústeres para una memoria caché creada anteriormente?](#can-i-configure-clustering-for-a-previously-created-cache)
* [¿Puedo configurar la agrupación en clústeres para una caché básica o estándar?](#can-i-configure-clustering-for-a-basic-or-standard-cache)
* [¿Puedo usar la agrupación en clústeres con los proveedores de estado de sesión y de almacenamiento en caché de salida de ASP.NET de Redis?.](#can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers)
* [Estoy recibiendo excepciones MOVE al usar StackExchange.Redis y agrupaciones en clústeres, ¿qué debo hacer?](#i-am-getting-move-exceptions-when-using-stackexchangeredis-and-clustering-what-should-i-do)

### <a name="do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering"></a>¿Es necesario realizar algún cambio en mi aplicación cliente para usar la agrupación en clústeres?
* Cuando la agrupación en clústeres está habilitada, solo está disponible la base de datos 0. Si la aplicación cliente usa varias bases de datos e intenta leer o escribir en una base de datos distinta de 0, se produce la siguiente excepción. `Unhandled Exception: StackExchange.Redis.RedisConnectionException: ProtocolFailure on GET --->` `StackExchange.Redis.RedisCommandException: Multiple databases are not supported on this server; cannot switch to database: 6`
  
  Para obtener más información, consulte [Redis Cluster Specification - Implemented subset](https://redis.io/topics/cluster-spec#implemented-subset)(Especificación de clúster en Redis: subconjunto implementado).
* Si usa [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/), debe usar la versión 1.0.481 o posterior. Se conecta a la memoria caché con los mismos [puntos de conexión, puertos y claves](cache-configure.md#properties) que usa al conectarse a una memoria caché que no tenga la agrupación en clústeres habilitada. La única diferencia es que se deben realizar todas las lecturas y escrituras en la base de datos 0.
  
  Otros clientes pueden tener requisitos diferentes. Vea [¿Todos los clientes de Redis admiten la agrupación en clústeres?](#do-all-redis-clients-support-clustering)
* Si la aplicación usa varias operaciones de claves por lotes en un solo comando, todas las claves deben estar ubicadas en la misma partición. Para ubicar claves en la misma partición, vea [¿Cómo se distribuyen las claves en un clúster?](#how-are-keys-distributed-in-a-cluster)
* Si está usando el proveedor de estado de sesión de ASP.NET de Redis, debe usar 2.0.1 o posterior. Consulte [¿Puedo usar la agrupación en clústeres con los proveedores de estado de sesión y de almacenamiento en caché de salida de ASP.NET de Redis?](#can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers)

### <a name="how-are-keys-distributed-in-a-cluster"></a>¿Cómo se distribuyen las claves en un clúster?
Según la documentación del [modelo de distribución de claves](https://redis.io/topics/cluster-spec#keys-distribution-model) de Redis, el espacio de claves se divide en 16 384 espacios. Cada clave tiene un hash y se asigna a una de estas ranuras, que se distribuyen por todos los nodos del clúster. Puede configurar qué parte de la clave está con hash para asegurarse de que varias claves se encuentran en la misma partición con etiquetas hash.

* Claves con una etiqueta hash: si cualquier parte de la clave está encerrada entre `{` y `}`, se aplica un algoritmo hash únicamente a esa parte de la clave para determinar la ranura hash de una clave. Por ejemplo, las siguientes tres claves se encontrarían en la misma partición: `{key}1`, `{key}2` y `{key}3` ya que solo la parte `key` del nombre está con hash. Para obtener una lista completa de especificaciones de etiquetas hash de claves, consulte [Etiquetas hash de claves](https://redis.io/topics/cluster-spec#keys-hash-tags).
* Claves sin etiquetas hash: se usa todo el nombre de la clave para aplicar el hash. Esto produce una distribución estadísticamente uniforme entre las particiones de la memoria caché.

Para optimizar el rendimiento, se recomienda distribuir las claves de manera uniforme. Si usa claves con etiquetas hash, es responsabilidad de la aplicación asegurarse de que las claves se distribuyan de manera uniforme.

Para obtener más información, consulte [Modelo de distribución de claves](https://redis.io/topics/cluster-spec#keys-distribution-model), [Particionamiento de datos de clúster Redis](https://redis.io/topics/cluster-tutorial#redis-cluster-data-sharding) y [Etiquetas hash de claves](https://redis.io/topics/cluster-spec#keys-hash-tags).

Para obtener el código de ejemplo sobre el trabajo con agrupación en clústeres y la ubicación de claves en la misma partición con el cliente StackExchange.Redis, consulte la parte [clustering.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/Clustering.cs) del ejemplo [Hello World](https://github.com/rustd/RedisSamples/tree/master/HelloWorld).

### <a name="what-is-the-largest-cache-size-i-can-create"></a>¿Cuál es el mayor tamaño de caché que puedo crear?
El tamaño máximo de caché premium es de 120 GB. Puede crear hasta 10 particiones con un tamaño máximo de 1,2 TB GB. Si necesita un tamaño mayor, puede [solicitar más](mailto:wapteams@microsoft.com?subject=Redis%20Cache%20quota%20increase). Para más información, consulte [Precios de Azure Cache for Redis](https://azure.microsoft.com/pricing/details/cache/).

### <a name="do-all-redis-clients-support-clustering"></a>¿Todos los clientes de Redis admiten la agrupación en clústeres?
No todos los clientes admiten la agrupación en clústeres de Redis. Consulte la documentación de la biblioteca que está usando para comprobar si utiliza una biblioteca y una versión que admiten la agrupación en clústeres. StackExchange.Redis es una biblioteca que admite la agrupación en clústeres, en sus versiones más recientes. Para obtener más información sobre otros clientes, consulte la sección [Jugar con el clúster](https://redis.io/topics/cluster-tutorial#playing-with-the-cluster) del [Tutorial de clúster de Redis](https://redis.io/topics/cluster-tutorial). 

El protocolo de agrupación en clústeres de Redis requiere que cada cliente se conecte a cada partición directamente en modo de agrupación en clústeres y también define nuevas respuestas de error, como 'MOVED' y 'CROSSSLOTS'. Si se intenta usar un cliente que no admite la agrupación en clústeres con una caché en modo de clúster, se pueden producir muchas [excepciones de redireccionamiento MOVED](https://redis.io/topics/cluster-spec#moved-redirection), o simplemente se interrumpe la aplicación si se realizan solicitudes de varias claves entre espacios.

> [!NOTE]
> Si está usando StackExchange.Redis como su cliente, asegúrese de que está usando la versión más reciente de [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/) 1.0.481 o posterior para que la agrupación en clústeres funcione correctamente. Si tiene problemas con las excepciones move, consulte la explicación sobre [excepciones move](#move-exceptions) para obtener más información.
>

### <a name="how-do-i-connect-to-my-cache-when-clustering-is-enabled"></a>¿Cómo me conecto a mi memoria caché cuando la agrupación en clústeres esté habilitada?
Puede conectarse a su memoria caché con los mismos [puntos de conexión](cache-configure.md#properties), [puertos](cache-configure.md#properties) y [claves](cache-configure.md#access-keys) que usa al conectarse a una memoria caché que no tenga la agrupación en clústeres habilitada. Redis administra la agrupación en clústeres en el back-end para que no tenga que administrarla desde el cliente.

### <a name="can-i-directly-connect-to-the-individual-shards-of-my-cache"></a>¿Puedo conectarme directamente a las particiones individuales de mi memoria caché?
El protocolo de agrupación en clústeres requiere que el cliente realice las conexiones de la partición correcta. Así, el cliente debe hacer esto correctamente en su lugar. Dicho esto, cada partición consta de un par de caché principal/réplica que se conoce colectivamente como una instancia de caché. Puede conectarse a estas instancias de caché mediante la utilidad redis-cli en la rama [inestable](https://redis.io/download) del repositorio de Redis en GitHub. Esta versión implementa compatibilidad básica cuando se inicia con el conmutador `-c` . Para más información, consulte la sección [Jugar con el clúster](https://redis.io/topics/cluster-tutorial#playing-with-the-cluster) en [https://redis.io](https://redis.io), en el [tutorial de clústeres de Redis](https://redis.io/topics/cluster-tutorial).

Cuando no sea TLS, use los siguientes comandos.

```bash
Redis-cli.exe –h <<cachename>> -p 13000 (to connect to instance 0)
Redis-cli.exe –h <<cachename>> -p 13001 (to connect to instance 1)
Redis-cli.exe –h <<cachename>> -p 13002 (to connect to instance 2)
...
Redis-cli.exe –h <<cachename>> -p 1300N (to connect to instance N)
```

Para TLS, reemplace `1300N` por `1500N`.

### <a name="can-i-configure-clustering-for-a-previously-created-cache"></a>¿Puedo configurar la agrupación en clústeres para una memoria caché creada anteriormente?
Sí. En primer lugar, asegúrese de que la memoria caché es Premium y realice un escalado si no lo es. A continuación, debería poder ver las opciones de configuración del clúster, incluida una opción para habilitarlo. Puede cambiar el tamaño del clúster una vez creada la memoria caché o después de habilitar la agrupación en clústeres por primera vez.

   >[!IMPORTANT]
   >La habilitación de la agrupación en clústeres no se puede deshacer. Hay que tener en cuenta que una caché que tiene habilitada la agrupación en clústeres y solo una partición se comporta *de modo diferente* que una caché del mismo tamaño *sin* agrupación en clústeres.

### <a name="can-i-configure-clustering-for-a-basic-or-standard-cache"></a>¿Puedo configurar la agrupación en clústeres para una caché básica o estándar?
La agrupación en clústeres solo está disponible para las memorias cachés premium.

### <a name="can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers"></a>¿Puedo usar la agrupación en clústeres con los proveedores de estado de sesión y de almacenamiento en caché de salida de ASP.NET de Redis?.
* **Proveedor de caché de salida de Redis** : no se requieren cambios.
* **Proveedor de estado de sesión de Redis**: para usar la agrupación en clústeres, debe usar [RedisSessionStateProvider](https://www.nuget.org/packages/Microsoft.Web.RedisSessionStateProvider) 2.0.1 o superior o se iniciará una excepción. Este es un cambio importante. Para más información, consulte [Detalles sobre cambios importantes de la versión 2.0.0](https://github.com/Azure/aspnet-redis-providers/wiki/v2.0.0-Breaking-Change-Details).

<a name="move-exceptions"></a>

### <a name="i-am-getting-move-exceptions-when-using-stackexchangeredis-and-clustering-what-should-i-do"></a>Estoy recibiendo excepciones MOVE al usar StackExchange.Redis y agrupaciones en clústeres, ¿qué debo hacer?
Si utiliza StackExchange.Redis y recibe `MOVE` excepciones al emplear agrupaciones en clústeres, asegúrese de que está utilizando la versión [1.1.603 de StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/) o posterior. Para obtener instrucciones sobre cómo configurar las aplicaciones .NET para usar StackExchange.Redis, consulte [Configuración de los clientes de caché](cache-dotnet-how-to-use-azure-redis-cache.md#configure-the-cache-clients).

## <a name="next-steps"></a>Pasos siguientes
Más información sobre las características de Azure Cache for Redis.

* [Niveles de servicio de Azure Cache for Redis Premium](cache-overview.md#service-tiers)

<!-- IMAGES -->

[redis-cache-clustering]: ./media/cache-how-to-premium-clustering/redis-cache-clustering.png

[redis-cache-clustering-selected]: ./media/cache-how-to-premium-clustering/redis-cache-clustering-selected.png

[redis-cache-redis-cluster-size]: ./media/cache-how-to-premium-clustering/redis-cache-redis-cluster-size.png
