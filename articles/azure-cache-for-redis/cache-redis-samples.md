---
title: Ejemplos de Azure Cache for Redis
description: Aprenda a usar Azure Cache for Redis con estos ejemplos de código (escenarios de conexión a una caché, lectura y escritura de datos en ella, proveedores de Azure Cache for Redis de ASP.NET).
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.custom: devx-track-dotnet
ms.topic: sample
ms.date: 01/23/2017
ms.openlocfilehash: 553850173f463a05b13768eb3b9e17703bfa2886
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88212295"
---
# <a name="azure-cache-for-redis-samples"></a>Ejemplos de Azure Cache for Redis
Este tema proporciona una lista de ejemplos de Azure Cache for Redis y cubre escenarios como la conexión a una caché, las operaciones de lectura y escritura de datos en una caché y el uso de proveedores de Azure Cache for Redis de ASP.NET. Algunos de los ejemplos son proyectos que se pueden descargar y algunos otros proporcionan instrucciones paso a paso e incluyen fragmentos de código pero no vínculos a un proyecto que se puede descargar.

## <a name="hello-world-samples"></a>Ejemplos Hello world
Los ejemplos de esta sección muestran los conceptos básicos de conexión a una instancia de Azure Cache for Redis, y de lectura y escritura de datos en la memoria caché usando una gran variedad de lenguajes y clientes de Redis.

En el ejemplo [Hello world](https://github.com/rustd/RedisSamples/tree/master/HelloWorld) se muestra cómo realizar diversas operaciones de caché mediante el cliente .NET [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis).

Este ejemplo lo siguiente:

* Uso de distintas opciones de conexión
* Lectura y escritura de objetos hacia y desde la memoria caché mediante operaciones sincrónicas y asincrónicas
* Uso de comandos MGET/MSET de Redis para devolver valores de las claves especificadas
* Realización de operaciones transaccionales de Redis
* Trabajo con listas de Redis y conjuntos ordenados
* Almacenamiento de objetos de .NET con serializadores JsonConvert
* Uso de conjuntos de Redis para implementar el etiquetado
* Trabajar con el Clúster en Redis

Para más información, consulte la documentación de [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) en GitHub y para ver más escenarios de uso, las pruebas unitarias [StackExchange.Redis.Tests](https://github.com/StackExchange/StackExchange.Redis/tree/master/tests).

En el artículo de [uso de Azure Cache for Redis con Python](cache-python-get-started.md) se muestra una introducción a Azure Cache for Redis con Python y el cliente [redis-py](https://github.com/andymccurdy/redis-py).

[Trabajar con objetos .NET en la memoria caché](cache-dotnet-how-to-use-azure-redis-cache.md#work-with-net-objects-in-the-cache) muestra una forma de serializar objetos .NET de forma que pueda realizar operaciones de escritura y lectura con ellos en una instancia de Azure Cache for Redis. 

## <a name="use-azure-cache-for-redis-as-a-scale-out-backplane-for-aspnet-signalr"></a>Uso de Azure Cache for Redis como backplane de escalado horizontal para ASP.NET SignalR
El ejemplo de [Uso de Azure Cache for Redis como backplane de escalado horizontal para ASP.NET SignalR](https://github.com/rustd/RedisSamples/tree/master/RedisAsSignalRBackplane) muestra cómo puede usar Azure Cache for Redis como backplane de SignalR. Para obtener más información acerca de backplane, consulte [Escalado horizontal SignalR con Redis](https://www.asp.net/signalr/overview/performance/scaleout-with-redis).

## <a name="azure-cache-for-redis-customer-query-sample"></a>Ejemplo de consulta de cliente de Azure Cache for Redis
Este ejemplo compara el rendimiento entre el acceso a datos desde una memoria caché y el acceso a datos desde almacenamiento de persistencia. Este ejemplo tiene dos proyectos.

* [Demostración de cómo Azure Cache for Redis puede mejorar el rendimiento con el almacenamiento de datos en caché](https://github.com/rustd/RedisSamples/tree/master/RedisCacheCustomerQuerySample)
* [Inicialización de la base de datos y la memoria caché para la demostración](https://github.com/rustd/RedisSamples/tree/master/SeedCacheForCustomerQuerySample)

## <a name="aspnet-session-state-and-output-caching"></a>Estado de sesión de ASP.NET y almacenamiento en caché de resultados
El ejemplo de [Uso de Azure Cache for Redis para almacenar SessionState y OutputCache de ASP.NET](https://github.com/rustd/RedisSamples/tree/master/SessionState_OutputCaching) muestra cómo usar Azure Cache for Redis para almacenar la sesión y la memoria caché de salida de ASP.NET usando los proveedores SessionState y OutputCache para Redis.

## <a name="manage-azure-cache-for-redis-with-maml"></a>Administración de Azure Cache for Redis con MAML
El ejemplo de [Administración de Azure Cache for Redis mediante bibliotecas de administración de Azure](https://github.com/rustd/RedisSamples/tree/master/ManageCacheUsingMAML) muestra cómo usar bibliotecas de administración de Azure para administrar (crear, actualizar y eliminar) la memoria caché. 

## <a name="custom-monitoring-sample"></a>Ejemplo de personalización de supervisión
El ejemplo de [Acceso a los datos de supervisión de Azure Cache for Redis](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring) muestra cómo puede acceder a datos de supervisión para la instancia de Azure Cache for Redis fuera de Azure Portal.

## <a name="a-twitter-style-clone-written-using-php-and-redis"></a>Un clon de estilo Twitter escrito mediante PHP y Redis
El ejemplo [Retwis](https://github.com/SyntaxC4-MSFT/retwis) es el Hello World de Redis. Es un clon de red social mínima de tipo Twitter escrito mediante Redis y PHP mediante el cliente [Predis](https://github.com/nrk/predis) . El código fuente está diseñado para ser muy sencillo y al mismo tiempo para mostrar diferentes estructuras de datos de Redis.

## <a name="bandwidth-monitor"></a>Supervisión del ancho de banda
El ejemplo [Supervisión del ancho de banda](https://github.com/JonCole/SampleCode/tree/master/BandWidthMonitor) permite supervisar el ancho de banda utilizado en el cliente. Para medir el ancho de banda, ejecute el ejemplo en el equipo cliente de la caché, realice llamadas a la memoria caché y observe el ancho de banda notificado por el ejemplo de supervisión de ancho de banda.
