---
title: ¿Qué es Azure Cache for Redis?
description: Información sobre Azure Cache for Redis para permitir cache-aside, el almacenamiento en caché de contenido, el almacenamiento en caché de la sesión de usuario, el trabajo y la cola de mensajes, así como las transacciones distribuidas.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: overview
ms.date: 05/12/2020
ms.openlocfilehash: 3751560125ea8ac6cc00ed63521bff30b751e688
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88009603"
---
# <a name="azure-cache-for-redis"></a>Azure Cache for Redis
Azure Cache for Redis proporciona un almacén de datos en memoria basado en el software de código abierto [Redis](https://redis.io/). Redis mejora el rendimiento y la escalabilidad de una aplicación que use en gran medida los almacenes de datos de back-end. Es capaz de procesar grandes volúmenes de solicitudes de aplicación al mantener los datos a los que se accede con frecuencia en la memoria del servidor en la que se puede escribir y leer rápidamente. Redis incorpora una solución crítica de almacenamiento de datos de baja latencia y alto rendimiento en las aplicaciones modernas.

Azure Cache for Redis ofrece Redis como servicio administrado. Proporciona instancias de servidor Redis seguras y dedicadas, así como una compatibilidad completa con la API de Redis. Microsoft ofrece el servicio, que se hospeda en Azure, y cualquier aplicación de dentro o fuera de Azure puede acceder a él.

Azure Cache for Redis se puede usar como una memoria caché de datos distribuidos o de contenido, un almacén de sesión, un agente de mensajes, entre otras opciones. Se puede implementar como independiente o junto con otro servicio de base de datos de Azure, como Azure SQL o Cosmos DB.

## <a name="key-scenarios"></a>Escenarios principales
Azure Cache for Redis mejora el rendimiento de las aplicaciones, ya que admite patrones de arquitectura de aplicaciones comunes. Estos son algunos de los más comunes:

| Patrón      | Descripción                                        |
| ------------ | -------------------------------------------------- |
| [Caché de datos](cache-web-app-cache-aside-leaderboard.md) | Las bases de datos suelen ser demasiado grandes para cargarlas directamente en una caché. Es habitual usar el patrón [cache-aside](https://docs.microsoft.com/azure/architecture/patterns/cache-aside) para cargar datos en la memoria caché solo cuando es necesario. Cuando el sistema realiza cambios en los datos, también puede actualizar la caché, que se distribuye luego a otros clientes. Además, el sistema puede establecer una fecha de expiración en los datos o usar una directiva de expulsión para desencadenar las actualizaciones de los datos en la memoria caché.|
| [Caché de contenido](cache-aspnet-output-cache-provider.md) | Muchas páginas web se generan a partir de plantillas que usan contenido estático como encabezados, pies de página y banners. Estos elementos estáticos no deberían cambiar a menudo. El uso de una caché en memoria proporciona acceso rápido a contenido estático en comparación con los almacenes de datos de back-end. Este patrón reduce el tiempo de procesamiento y la carga del servidor, lo que permite que los servidores web tengan mayor capacidad de respuesta. Puede permitirle reducir el número de servidores necesarios para administrar las cargas. Azure Cache for Redis proporciona el proveedor de caché de salida de Redis, que admite este patrón con ASP.NET.|
| [Almacén de sesión](cache-aspnet-session-state-provider.md) | Este patrón se utiliza normalmente con carros de la compra y otros datos de historial del usuario que una aplicación web puede desear asociar con las cookies del usuario. El almacenamiento de demasiados datos en una cookie puede tener un impacto negativo en el rendimiento, ya que aumenta su tamaño y no hay que olvidar que se pasa y se valida con cada solicitud. Una solución habitual usa la cookie como clave cuando se consultan datos en una base de datos. El uso de una memoria caché en memoria, como Azure Redis Cache, para asociar información a un usuario es mucho más rápido que su interacción con una base de datos relacional completa. |
| Almacenamiento en cola de trabajos y mensajes | Las aplicaciones agregan a menudo tareas a una cola cuando las operaciones asociadas a la solicitud tardan tiempo en ejecutarse. Las operaciones con ejecuciones más largas se ponen en cola para procesarse en secuencia, a menudo por parte de otro servidor.  Este método de aplazar trabajo se denomina puesta en cola de tareas. Azure Cache for Redis proporciona una cola distribuida que habilita este patrón en la aplicación.|
| Distributed transactions | A veces, las aplicaciones requieren una serie de comandos en un almacén de datos de back-end para ejecutarse como una única operación atómica. El resultado de todos los comandos debe ser satisfactorio, o todos deben revertirse al estado inicial. Azure Cache for Redis admite la ejecución de un lote de comandos como [transacción](https://redis.io/topics/transactions) única. |

## <a name="service-tiers"></a>Niveles de servicio
Azure Redis Cache está disponible en los niveles siguientes:

| Nivel | Descripción |
|---|---|
Básico | Caché de un solo nodo. Este nivel admite varios tamaños de memoria (250 MB-53 GB) y es ideal para el desarrollo o la realización de pruebas, así como para cargas de trabajo no críticas. El nivel Básico no tiene ningún Acuerdo de Nivel de Servicio (SLA). |
| Estándar | Una memoria caché replicada en una configuración de dos nodos, principal y de réplica, administrada por Azure y con un [Acuerdo de Nivel de Servicio (SLA)](https://azure.microsoft.com/support/legal/sla/cache/v1_0/) de alta disponibilidad. |
| Premium | El nivel Premium es el nivel específico para la empresa. Las memorias caché de nivel Premium admiten más características y tienen un rendimiento mayor con latencias más bajas. En el nivel Premium, las memorias caché se implementan en un hardware más potente, lo que proporciona un mejor rendimiento en comparación con los niveles Estándar o Básico. Esta ventaja significa que el rendimiento de una caché del mismo tamaño será mayor en el nivel Premium, en comparación con el nivel Estándar. |

### <a name="feature-comparison"></a>Comparación de características
En [Precios de Azure Cache for Redis](https://azure.microsoft.com/pricing/details/cache/) encontrará una comparación detallada de cada nivel. La tabla siguiente le ayuda a describir algunas de las características que admite cada nivel:

| Descripción de la característica | Premium | Estándar | Básico |
| ------------------- | :-----: | :------: | :---: |
| [Acuerdo de Nivel de Servicio (SLA)](https://azure.microsoft.com/support/legal/sla/cache/v1_0/) |✔|✔|-|
| [Persistencia de datos de Redis](cache-how-to-premium-persistence.md) |✔|-|-|
| [Clúster en Redis](cache-how-to-premium-clustering.md) |✔|-|-|
| [Seguridad a través de las reglas de firewall](cache-configure.md#firewall) |✔|✔|✔|
| Cifrado en tránsito |✔|✔|✔|
| [Aislamiento y seguridad mejorados con VNet](cache-how-to-premium-vnet.md) |✔|-|-|
| [Import/Export](cache-how-to-import-export-data.md) |✔|-|-|
| [Actualizaciones programadas](cache-administration.md#schedule-updates) |✔|✔|✔|
| [Replicación geográfica](cache-how-to-geo-replication.md) |✔|-|-|
| [Reboot](cache-administration.md#reboot) |✔|✔|✔|

### <a name="choosing-the-right-tier"></a>Elección del nivel correcto
Debe tener en cuenta lo siguiente al elegir un nivel de Azure Cache for Redis.

* **Memoria**: los niveles Básico y Estándar ofrecen de 250 MB a 53 GB. El nivel premium ofrece hasta 1,2 TB (como clúster) o 120 GB (sin clúster). Para más información, consulte [Precios de Azure Cache for Redis](https://azure.microsoft.com/pricing/details/cache/).
* **Rendimiento de la red**: si tiene una carga de trabajo que requiere un rendimiento alto, el nivel Prémium ofrece más ancho de banda en comparación con los niveles Estándar o Básico. También dentro de cada nivel, las cachés de mayor tamaño tienen más ancho de banda debido a la máquina virtual subyacente que hospeda la memoria caché. Para más información, consulte [Rendimiento de Azure Cache for Redis](cache-planning-faq.md#azure-cache-for-redis-performance).
* **Rendimiento**: el nivel Prémium ofrece el máximo rendimiento disponible. Si el servidor o el cliente de caché alcanzan los límites del ancho de banda, recibirá los tiempos de espera del cliente. Para más información, vea la tabla siguiente.
* **Alta disponibilidad**: Azure Cache for Redis garantiza que la memoria caché de los niveles Estándar y Prémium estará disponible de acuerdo con nuestro [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/cache/v1_0/). El Acuerdo de Nivel de Servicio solo cubre la conectividad para los puntos de conexión de la memoria caché. El SLA no cubre la protección frente a la pérdida de datos. Se recomienda usar la característica de persistencia de datos de Redis en el nivel Premium para aumentar la resistencia contra la pérdida de datos.
* **Persistencia de los datos de Redis**: El nivel Premium permite conservar los datos de la memoria caché en una cuenta de Azure Storage. En una caché Básico/Estándar todos los datos se almacenan solo en la memoria. Los problemas con la infraestructura subyacente podrían provocar una pérdida de datos. Se recomienda usar la característica de persistencia de datos de Redis en el nivel Premium para aumentar la resistencia contra la pérdida de datos. Azure Cache for Redis ofrece las opciones RDB y AOF (versión preliminar) en la persistencia de Redis. Para más información, vea [How to configure persistence for a Premium Azure Cache for Redis](cache-how-to-premium-persistence.md) (Configuración de la persistencia para una instancia de Azure Cache for Redis Prémium).
* **Clúster en Redis**: para crear memorias caché de más de 120 GB o particionar los datos entre varios nodos de Redis, puede usar la agrupación en clústeres Redis, disponible en el nivel Premium. Cada nodo consta de un par de caché principal/réplica para alta disponibilidad. Para más información, consulte [How to configure clustering for a Premium Azure Cache for Redis](cache-how-to-premium-clustering.md) (Configuración de la agrupación en clústeres para una instancia de Azure Cache for Redis Prémium).
* **Seguridad y aislamiento de red mejorados**: La implementación de Azure Virtual Network aporta más seguridad y aislamiento de su instancia de Azure Cache for Redis, así como subredes, directivas de control de acceso y otras características para restringir aún más el acceso. Para más información, consulte [How to configure Virtual Network support for a Premium Azure Cache for Redis](cache-how-to-premium-vnet.md) (Configuración de la compatibilidad con Virtual Network para una instancia de Azure Cache for Redis Prémium).
* **Configuración de Redis**: tanto en los niveles Estándar como Prémium, puede configurar Redis para las notificaciones de Keyspace.
* **Número máximo de conexiones de cliente**: el nivel Prémium ofrece el número máximo de clientes que se pueden conectar a Redis, con un número mayor de conexiones para memorias caché de mayor tamaño. La agrupación en clústeres no aumenta el número de conexiones disponibles para una caché en clúster. Para más información, consulte [Precios de Azure Cache for Redis](https://azure.microsoft.com/pricing/details/cache/).
* **Núcleo específico para el servidor de Redis**: en el nivel Prémium todos los tamaños de caché tienen un núcleo específico para Redis. En los niveles Básico/Estándar, el tamaño C1 y superiores tienen un núcleo dedicado para el servidor Redis.
* **Procesamiento de un solo subproceso**: Redis, por diseño, solo usa un subproceso para el procesamiento de comandos. Azure Cache for Redis también emplea núcleos adicionales para el procesamiento de E/S. El hecho de tener más núcleos mejora el rendimiento del procesamiento, aunque pueda no producir un escalado lineal. Además, los tamaños de máquina virtual mayores suelen tener límites de ancho de banda también mayores. De este modo, contribuye a evitar la saturación de la red, lo que provocará tiempos de espera en la aplicación.
* **Mejoras en el rendimiento**: las memorias caché en el nivel Prémium se implementan en el hardware que tiene procesadores más rápidos y ofrecen un mejor rendimiento en comparación con el nivel Básico o Estándar. Las cachés de nivel Premium tienen un mayor rendimiento y latencias más bajas. Para más información, consulte [Rendimiento de Azure Cache for Redis](cache-planning-faq.md#azure-cache-for-redis-performance).

Una vez creada, la memoria caché se puede escalar a un nivel superior. Sin embargo, no se admite su reducción vertical. Para instrucciones detalladas acerca del escalado, consulte [How to Scale Azure Cache for Redis](cache-how-to-scale.md) (Escalado de Azure Redis Cache) y [How to automate a scaling operation](cache-how-to-scale.md#how-to-automate-a-scaling-operation) (Automatización de una operación de escalado).

## <a name="next-steps"></a>Pasos siguientes
* [Inicio rápido de una aplicación web de ASP.NET](cache-web-app-howto.md) Cree una aplicación web de ASP.NET simple que use Azure Redis Cache.
* [Inicio rápido de .NET](cache-dotnet-how-to-use-azure-redis-cache.md) Cree una aplicación .NET que use Azure Redis Cache.
* [Inicio rápido de .NET Core](cache-dotnet-core-quickstart.md) Cree una aplicación .NET Core que use Azure Redis Cache.
* [Inicio rápido de Node.js](cache-nodejs-get-started.md) Cree una aplicación Node.js simple que use Azure Redis Cache.
* [Inicio rápido de Java](cache-java-get-started.md) Cree una aplicación Java simple que use Azure Redis Cache.
* [Inicio rápido de Python](cache-python-get-started.md) Cree una aplicación Python que use Azure Redis Cache.
