---
title: Preguntas frecuentes sobre el desarrollo para Azure Cache for Redis
description: Conozca las respuestas a preguntas comunes que le ayudarán a desarrollar para Azure Cache for Redis.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 08/06/2020
ms.openlocfilehash: ef85b6f9e4595e7b4ff367da415fad777de68679
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88211304"
---
# <a name="azure-cache-for-redis-development-faqs"></a>Preguntas frecuentes sobre el desarrollo para Azure Cache for Redis

En este artículo se proporcionan respuestas a preguntas habituales sobre cómo desarrollar para Azure Cache for Redis.

## <a name="common-questions-and-answers"></a>Preguntas y respuestas habituales
En esta sección se incluyen las siguientes preguntas frecuentes:

* [¿Cómo se empieza con Azure Cache for Redis?](#how-can-i-get-started-with-azure-cache-for-redis)
* [¿Qué hacen las opciones de configuración de StackExchange.Redis?](#what-do-the-stackexchangeredis-configuration-options-do)
* [¿Qué clientes de Azure Cache for Redis puedo usar?](#what-azure-cache-for-redis-clients-can-i-use)
* [¿Hay un emulador local para Azure Cache for Redis?](#is-there-a-local-emulator-for-azure-cache-for-redis)
* [¿Cómo puedo ejecutar comandos de Redis?](#how-can-i-run-redis-commands)
* [¿Por qué Azure Cache for Redis no tiene una referencia de biblioteca de clases MSDN?](#why-doesnt-azure-cache-for-redis-have-an-msdn-class-library-reference)
* [¿Puedo usar Azure Cache for Redis como caché de la sesión PHP?](#can-i-use-azure-cache-for-redis-as-a-php-session-cache)
* [¿Cuáles son las bases de datos de Redis?](#what-are-redis-databases)

### <a name="how-can-i-get-started-with-azure-cache-for-redis"></a>¿Cómo se empieza con Azure Cache for Redis?
Hay varias maneras de empezar a utilizar Azure Cache for Redis.

* Puede consultar uno de los tutoriales disponibles para [.NET](cache-dotnet-how-to-use-azure-redis-cache.md), [ASP.NET](cache-web-app-howto.md), [Java](cache-java-get-started.md), [Node.js](cache-nodejs-get-started.md) y [Python](cache-python-get-started.md).
* Puede ver el vídeo [How to Build High Performance Apps Using Microsoft Azure Cache for Redis](https://azure.microsoft.com/documentation/videos/how-to-build-high-performance-apps-using-microsoft-azure-cache/)(Procedimiento para compilar aplicaciones de alto rendimiento con Azure Cache for Redis).
* Puede consultar la documentación de los clientes que coincidan con el lenguaje de desarrollo de su proyecto para ver cómo usar Redis. Hay muchos clientes de Redis que pueden utilizarse con Azure Cache for Redis. Para ver una lista de clientes de Redis, consulte [https://redis.io/clients](https://redis.io/clients).

Si no tiene una cuenta de Azure, siga estos pasos:

* [Abrir una cuenta de Azure gratis](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=redis_cache_hero). Obtenga créditos que puede usar para probar los servicios de Azure de pago. Incluso después de que se agoten los créditos, puede mantener la cuenta y usar los servicios y características gratuitos de Azure.
* [Activar los beneficios de suscripción a Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=redis_cache_hero). Su suscripción a MSDN le proporciona créditos todos los meses que puede usar para servicios de Azure de pago.

### <a name="what-do-the-stackexchangeredis-configuration-options-do"></a>¿Qué hacen las opciones de configuración de StackExchange.Redis?
StackExchange.Redis tiene muchas opciones. En esta sección se describen algunas de las configuraciones comunes. Para obtener más información acerca de las opciones de StackExchange.Redis, consulte [Configuración de StackExchange.Redis](https://stackexchange.github.io/StackExchange.Redis/Configuration).

| ConfigurationOptions | Descripción | Recomendación |
| --- | --- | --- |
| AbortOnConnectFail |Cuando se establece en true, la conexión no se volverá a conectar después de un error de red. |Establézcalo en false y deje que StackExchange.Redis se vuelva a conectar automáticamente. |
| ConnectRetry |El número de veces que se repiten los intentos de conexión durante la conexión inicial. |Consulte la siguiente imagen para obtener instrucciones. |
| ConnectTimeout |Tiempo de espera en milisegundos para operaciones de conexión. |Consulte la siguiente imagen para obtener instrucciones. |

Habitualmente, los valores predeterminados del cliente son suficientes. Ajuste las opciones en función de su carga de trabajo.

* **Reintentos**
  * Para ConnectRetry y ConnectTimeout, la regla general es fracasar y responder rápido a los errores y volver a intentarlo. Esta regla se basa en la carga de trabajo y en cuánto tiempo como media tarda el cliente en enviar un comando de Redis y en recibir una respuesta.
  * Permita que StackExchange.Redis se vuelva a conectar automáticamente en lugar de comprobar el estado de conexión y volver a conectarse. **Evite el uso de la propiedad ConnectionMultiplexer.IsConnected**.
  * Efecto bola de nieve: a veces puede encontrarse con un problema cuando lo está intentando y este error aumenta y nunca se recupera. En este caso, debe considerar el uso de un algoritmo de reintento de retroceso exponencial, tal y como se describe en el artículo [Orientación general sobre reintentos](../best-practices-retry-general.md) publicado por el grupo de Microsoft Patterns & Practices.
  
* **Valores de tiempo de expiración**
  * Tenga en cuenta la carga de trabajo y establezca los valores según corresponda. Si almacena valores grandes, establezca el tiempo de expiración en un valor superior.
  * Establezca `AbortOnConnectFail` en False y deje que StackExchange.Redis se vuelva a conectar automáticamente.
  * Utilice una única instancia de ConnectionMultiplexer para la aplicación. Puede usar LazyConnection para crear una instancia única que se devuelva por una propiedad de conexión, tal como se muestra en [Conexión a la caché mediante la clase ConnectionMultiplexer](cache-dotnet-how-to-use-azure-redis-cache.md#connect-to-the-cache).
  * Establezca la propiedad `ConnectionMultiplexer.ClientName` a un nombre único de la instancia de aplicación con fines de diagnóstico.
  * Use varias instancias `ConnectionMultiplexer` para cargas de trabajo personalizadas.
      * Puede seguir este modelo si tiene una carga variable en la aplicación. Por ejemplo:
      * Puede tener un multiplexor para tratar con claves grandes.
      * Puede tener un multiplexor para tratar con claves pequeñas.
      * Puede establecer distintos valores para los tiempos de expiración de la conexión así como lógica de reintento para cada ConnectionMultiplexer que use.
      * Establezca la propiedad `ClientName` en cada multiplexor para ayudar con el diagnóstico.
      * Esta guía puede conducir a una latencia más simplificada por `ConnectionMultiplexer`.

### <a name="what-azure-cache-for-redis-clients-can-i-use"></a>¿Qué clientes de Azure Cache for Redis puedo usar?
Una de las grandes virtudes de Redis es que hay muchos clientes que admiten muchos lenguajes de desarrollo diferentes. Para obtener una lista de clientes, consulte [Redis clients](https://redis.io/clients)(Clientes de Redis). Para ver tutoriales que abarcan diversos lenguajes y clientes, consulte [Cómo usar Azure Cache for Redis](cache-dotnet-how-to-use-azure-redis-cache.md) y los artículos relacionados de la tabla de contenido.

[!INCLUDE [redis-cache-create](../../includes/redis-cache-access-keys.md)]

### <a name="is-there-a-local-emulator-for-azure-cache-for-redis"></a>¿Hay un emulador local para Azure Cache for Redis?
No hay ningún emulador local para Azure Cache for Redis, pero puede ejecutar la versión MSOpenTech de redis-server.exe desde las [herramientas de la línea de comandos de Redis](https://github.com/MSOpenTech/redis/releases/) en su máquina local y conectarse a ella para obtener una experiencia similar a un emulador de memoria caché local, tal y como se muestra en el ejemplo siguiente:

```csharp
private static Lazy<ConnectionMultiplexer>
    lazyConnection = new Lazy<ConnectionMultiplexer> (() =>
    {
        // Connect to a locally running instance of Redis to simulate
        // a local cache emulator experience.
        return ConnectionMultiplexer.Connect("127.0.0.1:6379");
    });

public static ConnectionMultiplexer Connection
{
    get
    {
        return lazyConnection.Value;
    }
}
```

Si lo desea, también puede configurar un archivo [redis.conf](https://redis.io/topics/config) para ajustarse con más precisión a la [configuración de caché predeterminada](cache-configure.md#default-redis-server-configuration) del servicio en línea Azure Cache for Redis.

### <a name="how-can-i-run-redis-commands"></a>¿Cómo puedo ejecutar comandos de Redis?
Puede usar cualquiera de los comandos enumerados en el artículo de [Comandos de Redis](https://redis.io/commands#), salvo los comandos indicados en [Redis commands not supported in Azure Cache for Redis](cache-configure.md#redis-commands-not-supported-in-azure-cache-for-redis) (Comandos de Redis no admitidos en Azure Cache for Redis). Para ejecutar comandos de Redis, tiene varias opciones.

* Si tiene una caché Estándar o Premium, puede ejecutar comandos de Redis mediante la [Consola de Redis](cache-configure.md#redis-console). La consola de Redis ofrece una manera segura de ejecutar comandos de Redis en Azure Portal.
* También puede usar las herramientas de línea de comandos de Redis. Para ello, realice los siguientes pasos:
* Descargue las [herramientas de línea de comandos de Redis](https://github.com/MSOpenTech/redis/releases/).
* Conexión a la memoria caché mediante `redis-cli.exe`. Pase el punto de conexión de caché con el modificador -h y la clave con - a, tal como se muestra en el ejemplo siguiente:
* `redis-cli -h <Azure Cache for Redis name>.redis.cache.windows.net -a <key>`

> [!NOTE]
> Las herramientas de línea de comandos de Redis no funcionan con el puerto TLS, pero puede usar una utilidad como `stunnel` para conectar de forma segura las herramientas al puerto TLS si sigue las instrucciones del artículo [Uso de la herramienta de línea de comandos de Redis con Azure Cache for Redis](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-redis-cli-tool).
>
>

### <a name="why-doesnt-azure-cache-for-redis-have-an-msdn-class-library-reference"></a>¿Por qué Azure Cache for Redis no tiene una referencia de biblioteca de clases MSDN?
Microsoft Azure Cache for Redis se basa en Redis, el conocido almacén de datos en memoria de código abierto. Se puede acceder mediante una amplia variedad de [clientes de Redis](https://redis.io/clients) para muchos lenguajes de programación. Cada cliente tiene su propia API que realiza llamadas a la instancia de Azure Cache for Redis mediante los [comandos de Redis](https://redis.io/commands).

Dado que cada cliente es diferente, no hay no una referencia de clase centralizada en MSDN; cada cliente mantiene su propia documentación de referencia. Además de la documentación de referencia, hay varios tutoriales que muestran cómo empezar a trabajar con Azure Cache for Redis con distintos idiomas y clientes de la caché. Para acceder a estos tutoriales, consulte [Cómo usar Azure Cache for Redis](cache-dotnet-how-to-use-azure-redis-cache.md) y los artículos relacionados de la tabla de contenido.

### <a name="can-i-use-azure-cache-for-redis-as-a-php-session-cache"></a>¿Puedo usar Azure Cache for Redis como caché de la sesión PHP?
Sí, para usar Azure Cache for Redis como una memoria caché de sesión PHP, especifique la cadena de conexión a la instancia de Azure Cache for Redis en `session.save_path`.

> [!IMPORTANT]
> Al usar Azure Cache for Redis como una caché de sesión PHP, debe codificar como dirección URL la clave de seguridad que se usa para conectarse a la memoria caché, tal como se muestra en el ejemplo siguiente:
>
> `session.save_path = "tcp://mycache.redis.cache.windows.net:6379?auth=<url encoded primary or secondary key here>";`
>
> Si la clave no está codificada como dirección URL, puede recibir una excepción con un mensaje similar al siguiente: `Failed to parse session.save_path`
>

Para más información sobre el uso de Azure Cache for Redis como caché de la sesión PHP con el cliente de PhpRedis, consulte [PHP Session handler](https://github.com/phpredis/phpredis#php-session-handler) (Controlador de sesión de PHP).

### <a name="what-are-redis-databases"></a>¿Cuáles son las bases de datos de Redis?

Las bases de datos de Redis son simplemente una separación lógica de los datos en la misma instancia de Redis. La memoria caché se comparte entre todas las bases de datos y el consumo real de una base de datos depende de los pares clave-valor almacenados en esa base de datos. Por ejemplo, una caché de C6 tiene 53 GB de memoria y una de P5 tiene 120 GB. Puede elegir colocar los 53 GB o los 120 GB en una base de datos o dividirlos entre varias bases de datos. 

> [!NOTE]
> Cuando se utiliza una instancia Prémium de Azure Cache for Redis con clústeres habilitados, solo está disponible la base de datos 0. Se trata de una limitación de Redis intrínseca y no es específica de Azure Cache for Redis. Para obtener más información, consulte [¿Es necesario realizar algún cambio en mi aplicación cliente para usar la agrupación en clústeres?](cache-how-to-premium-clustering.md#do-i-need-to-make-any-changes-to-my-client-application-to-use-clustering)
> 
> 

## <a name="next-steps"></a>Pasos siguientes

Más información sobre otras [preguntas frecuentes sobre Azure Cache for Redis](cache-faq.md).
