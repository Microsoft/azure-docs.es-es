---
title: 'Configuración de la persistencia de datos: versión Premium de Azure Cache for Redis'
description: Aprenda a configurar y administrar la persistencia de datos en las instancias del nivel Prémium de Azure Cache for Redis.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 08/24/2017
ms.openlocfilehash: aaee1c07f0fc8d5b0bba03550986291aea814fcb
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88004798"
---
# <a name="how-to-configure-data-persistence-for-a-premium-azure-cache-for-redis"></a>Configuración de la persistencia de datos en el nivel Prémium de Azure Cache for Redis
Azure Cache for Redis cuenta con diferentes opciones de caché, lo que proporciona flexibilidad en la elección del tamaño y las características de la memoria caché, incluidas algunas características del nivel Prémium, como la agrupación en clústeres, la persistencia y la compatibilidad con redes virtuales. En este artículo, se describe cómo se configura la persistencia de una instancia de Azure Cache for Redis del nivel Prémium.

## <a name="what-is-data-persistence"></a>¿Qué es la persistencia de datos?
La [persistencia de Redis](https://redis.io/topics/persistence) permite conservar los datos almacenados en Redis. También puede tomar instantáneas y realizar copias de seguridad de los datos que puede cargar en el caso de un error de hardware. Se trata de una inmensa ventaja sobre el nivel Básico o Estándar, donde todos los datos se almacenan en la memoria y puede haber una posible pérdida de datos en caso de error donde los nodos de la memoria caché están inactivos. 

Azure Cache for Redis proporciona la persistencia de Redis utilizando los modelos siguientes:

* **Persistencia de RDB**: cuando se configura la persistencia de RDB (base de datos de Redis), Azure Cache for Redis conserva en el disco una instantánea de la caché en un formato binario de Redis con una frecuencia de copia de seguridad que se puede configurar. Si se produce un evento catastrófico que deshabilita tanto la caché de réplica como la principal, se reconstruye la memoria caché con la instantánea más reciente. Más información sobre las [ventajas](https://redis.io/topics/persistence#rdb-advantages) y [desventajas](https://redis.io/topics/persistence#rdb-disadvantages) de la persistencia de RDB.
* **Persistencia de AOF**: cuando la persistencia de AOF (archivo de solo anexión) está configurada, Azure Cache for Redis almacena cada operación de escritura en un registro que se guarda al menos una vez por segundo en una cuenta de Azure Storage. Si se produce un evento catastrófico que deshabilita tanto la caché de réplica como la principal, se reconstruye la memoria caché con las operaciones de escritura almacenadas. Más información sobre las [ventajas](https://redis.io/topics/persistence#aof-advantages) y [desventajas](https://redis.io/topics/persistence#aof-disadvantages) de la persistencia de AOF.

La persistencia escribe los datos de Redis en una cuenta de Azure Storage que posea y administre. Puede configurarla en la hoja **New Azure Cache for Redis** (Nueva instancia de Azure Cache for Redis) durante la creación de la caché y en el **menú Recurso** de las cachés existentes del nivel Premium.

> [!NOTE]
> 
> Azure Storage cifra automáticamente los datos cuando se conservan. Puede usar sus propias claves para el cifrado. Para más información, consulte [Claves administradas por el cliente con Azure Key Vault](/azure/storage/common/storage-service-encryption).
> 
> 

[!INCLUDE [redis-cache-create](../../includes/redis-cache-premium-create.md)]

Una vez se selecciona un plan de tarifa premium, haga clic en **Persistencia de Redis**.

![Persistencia de Redis][redis-cache-persistence]

Los pasos de la siguiente sección explican cómo configurar la persistencia de Redis en la nueva caché Premium. Una vez configurada la persistencia de Redis, haga clic en **Crear** para crear su nueva caché premium con persistencia de Redis.

## <a name="enable-redis-persistence"></a>Habilitación de la persistencia de Redis

La persistencia de Redis se habilita en la hoja **Persistencia de los datos**, seleccionando la persistencia **RDB** o **AOF**. Para nuevas caché, a esta hoja se obtiene acceso durante el proceso de creación de la caché, como se describe en la sección anterior. Para las memorias caché existentes, a la hoja **Persistencia de los datos** se accede desde el **menú Recursos** de la caché.

![Configuración de Redis][redis-cache-settings]


## <a name="configure-rdb-persistence"></a>Configuración de la persistencia de RDB

Para habilitar la persistencia de RDB, haga clic en **RDB**. Para deshabilitar la persistencia de RDB en una caché Premium previamente habilitada, haga clic en **Deshabilitada**.

![Persistencia de RDB en Redis][redis-cache-rdb-persistence]

Para configurar el intervalo de copia de seguridad, seleccione una **Frecuencia de copia de seguridad** en la lista desplegable. Las opciones disponibles son: **15 minutos**, **30 minutos**, **60 minutos**, **6 horas**, **12 horas** y **24 horas**. Este intervalo empieza la cuenta atrás cuando se completa correctamente la operación de copia de seguridad anterior y se inicia cuando se produce una nueva copia de seguridad.

Haga clic en **Cuenta de almacenamiento** para seleccionar la cuenta de almacenamiento que quiere usar y, en el cuadro desplegable **Clave de almacenamiento**, elija la **clave principal** o la **clave secundaria** que se va a utilizar. Debe elegir una cuenta de almacenamiento en la misma región que la memoria caché y se recomienda una cuenta de **Premium Storage** porque Premium Storage tiene un mayor rendimiento. 

> [!IMPORTANT]
> Si se vuelve a generar la clave de almacenamiento para su cuenta de persistencia, debe volver a configurar la clave que quiera en la lista desplegable **Clave de almacenamiento**.
> 
> 

Haga clic en **Aceptar** para guardar la configuración de persistencia.

La siguiente copia de seguridad (o la primera copia de seguridad para las nuevas cachés) se inicia cuando transcurre el intervalo de frecuencia de copia de seguridad.

## <a name="configure-aof-persistence"></a>Configuración de la persistencia de AOF

Para habilitar la persistencia de AOF, haga clic en **AOF**. Para deshabilitar la persistencia de AOF en una caché Premium habilitada anteriormente, haga clic en **Deshabilitada**.

![Persistencia de AOF en Redis][redis-cache-aof-persistence]

Para configurar la persistencia de AOF, especifique **Primera cuenta de almacenamiento**. Esta cuenta de almacenamiento debe estar en la misma región que la memoria caché y se recomienda una cuenta **Premium Storage** porque el almacenamiento Premium tiene un mayor rendimiento. También puede configurar una cuenta de almacenamiento adicional denominada **Segunda cuenta de almacenamiento**. Si se configura una segunda cuenta de almacenamiento, las operaciones de escritura en la caché de réplica se realizan en esta segunda cuenta de almacenamiento. Para cada cuenta de almacenamiento configurada, seleccione la **Clave principal** o **Clave secundaria** que se usará del menú desplegable **Clave de almacenamiento**. 

> [!IMPORTANT]
> Si se vuelve a generar la clave de almacenamiento para su cuenta de persistencia, debe volver a configurar la clave que quiera en la lista desplegable **Clave de almacenamiento**.
> 
> 

Cuando se habilita la persistencia de AOF, las operaciones de escritura en la memoria caché se guardan en la cuenta de almacenamiento designada (o las cuentas si ha configurado una segunda cuenta de almacenamiento). Si se produce un error catastrófico que daña la caché principal y de réplica, el registro de AOF almacenado se usa para regenerar la memoria caché.

## <a name="persistence-faq"></a>P+F de persistencia
La lista siguiente contiene respuestas a las preguntas frecuentes sobre la persistencia de Azure Cache for Redis.

* [¿Puedo habilitar la persistencia en una memoria caché creada anteriormente?](#can-i-enable-persistence-on-a-previously-created-cache)
* [¿Puedo habilitar la persistencia de AOF y RDB al mismo tiempo?](#can-i-enable-aof-and-rdb-persistence-at-the-same-time)
* [¿Qué modelo de persistencia debería elegir?](#which-persistence-model-should-i-choose)
* [¿Qué sucede si he escalado a otro tamaño y se restaura una copia de seguridad realizada antes de la operación de escalado?](#what-happens-if-i-have-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation)


### <a name="rdb-persistence"></a>Persistencia de RDB
* [¿Puedo cambiar la frecuencia de copia de seguridad de RDB después de crear la memoria caché?](#can-i-change-the-rdb-backup-frequency-after-i-create-the-cache)
* [¿Por qué si tengo una frecuencia de copia de seguridad de RDB de 60 minutos hay más de 60 minutos entre las copias de seguridad?](#why-if-i-have-an-rdb-backup-frequency-of-60-minutes-there-is-more-than-60-minutes-between-backups)
* [¿Qué ocurre con las copias de seguridad de RDB antiguas cuando se realiza una nueva copia de seguridad?](#what-happens-to-the-old-rdb-backups-when-a-new-backup-is-made)

### <a name="aof-persistence"></a>Persistencia de AOF
* [¿Cuándo debo usar una segunda cuenta de almacenamiento?](#when-should-i-use-a-second-storage-account)
* [¿Afecta la persistencia de AOF a la productividad, la latencia o el rendimiento de la memoria caché?](#does-aof-persistence-affect-throughout-latency-or-performance-of-my-cache)
* [¿Cómo puedo quitar la segunda cuenta de almacenamiento?](#how-can-i-remove-the-second-storage-account)
* [¿Qué es una reescritura y cómo afecta a la memoria caché?](#what-is-a-rewrite-and-how-does-it-affect-my-cache)
* [¿Qué debo esperar al escalar una memoria caché con AOF habilitado?](#what-should-i-expect-when-scaling-a-cache-with-aof-enabled)
* [¿Cómo se organizan los datos de AOF en el almacenamiento?](#how-is-my-aof-data-organized-in-storage)


### <a name="can-i-enable-persistence-on-a-previously-created-cache"></a>¿Puedo habilitar la persistencia en una memoria caché creada anteriormente?
Sí, la persistencia de Redis puede configurarse tanto durante la creación de la memoria caché como en las memorias caché premium existente.

### <a name="can-i-enable-aof-and-rdb-persistence-at-the-same-time"></a>¿Puedo habilitar la persistencia de AOF y RDB al mismo tiempo?

No, puede habilitar solo RDB o AOF, pero no ambos a la vez.

### <a name="which-persistence-model-should-i-choose"></a>¿Qué modelo de persistencia debería elegir?

La persistencia de AOF guarda cada operación de escritura en un registro, lo que tiene algún impacto en el rendimiento si se compara con la persistencia de RDB, que guarda las copias de seguridad según el intervalo de copia de seguridad configurado, con un impacto mínimo sobre el rendimiento. Elija la persistencia de AOF si el objetivo principal es minimizar la pérdida de datos y si puede controlar una disminución del rendimiento de la memoria caché. Elija la persistencia de RDB si quiere mantener un rendimiento óptimo en la memoria caché pero todavía quiere un mecanismo para la recuperación de datos.

* Más información sobre las [ventajas](https://redis.io/topics/persistence#rdb-advantages) y [desventajas](https://redis.io/topics/persistence#rdb-disadvantages) de la persistencia de RDB.
* Más información sobre las [ventajas](https://redis.io/topics/persistence#aof-advantages) y [desventajas](https://redis.io/topics/persistence#aof-disadvantages) de la persistencia de AOF.

Para más información sobre el rendimiento al usar la persistencia de AOF, vea [¿Afecta la persistencia de AOF a la productividad, la latencia o el rendimiento de la memoria caché?](#does-aof-persistence-affect-throughout-latency-or-performance-of-my-cache)

### <a name="what-happens-if-i-have-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation"></a>¿Qué sucede si he escalado a otro tamaño y se restaura una copia de seguridad realizada antes de la operación de escalado?

Para la persistencia de RDB y AOF:

* Si ha escalado a un tamaño mayor, no hay ningún impacto.
* Si ha escalado a un tamaño menor y tiene una configuración de [bases de datos](cache-configure.md#databases) personalizada que es mayor que el [límite de bases de datos](cache-configure.md#databases) del nuevo tamaño, los datos de esas bases de datos no se restauran. Para más información, consulte [¿Mi configuración de bases de datos personalizada se ve afectada durante el escalado?](cache-how-to-scale.md#is-my-custom-databases-setting-affected-during-scaling)
* Si ha escalado a un tamaño menor y no hay suficiente espacio en el tamaño más pequeño para contener todos los datos desde la última copia de seguridad, las claves se expulsarán durante el proceso de restauración, normalmente mediante el uso de la directiva de expulsión [allkeys-lru](https://redis.io/topics/lru-cache) .

### <a name="can-i-change-the-rdb-backup-frequency-after-i-create-the-cache"></a>¿Puedo cambiar la frecuencia de copia de seguridad de RDB después de crear la memoria caché?
Sí, puede cambiar la frecuencia de copia de seguridad de la persistencia de RDB en la hoja **Persistencia de los datos**. Para ver las instrucciones, consulte Configurar la persistencia de Redis.

### <a name="why-if-i-have-an-rdb-backup-frequency-of-60-minutes-there-is-more-than-60-minutes-between-backups"></a>¿Por qué si tengo una frecuencia de copia de seguridad de RDB de 60 minutos hay más de 60 minutos entre las copias de seguridad?
El intervalo de frecuencia de copia de seguridad de la persistencia de RDB no se inicia hasta que el proceso de copia de seguridad anterior se ha completado correctamente. Si la frecuencia de copia de seguridad es de 60 minutos y realiza un proceso de copia de seguridad en 15 minutos para completarse correctamente, la siguiente copia de seguridad no se iniciará hasta pasados 75 minutos de la hora de inicio de la copia de seguridad anterior.

### <a name="what-happens-to-the-old-rdb-backups-when-a-new-backup-is-made"></a>¿Qué ocurre con las copias de seguridad de RDB antiguas cuando se realiza una nueva copia de seguridad?
Todas las copias de seguridad de persistencia de RDB excepto la más reciente se eliminan automáticamente. Es posible que esta eliminación no se produzca inmediatamente pero las copias de seguridad anteriores no se guardan de manera indefinida.


### <a name="when-should-i-use-a-second-storage-account"></a>¿Cuándo debo usar una segunda cuenta de almacenamiento?

Debe usar una segunda cuenta de almacenamiento para la persistencia de AOF cuando crea que tiene más operaciones de conjunto de las esperadas en la memoria caché.  La configuración de la cuenta de almacenamiento secundaria ayuda a garantizar que la memoria caché no alcance los límites de ancho de banda de almacenamiento.

### <a name="does-aof-persistence-affect-throughout-latency-or-performance-of-my-cache"></a>¿Afecta la persistencia de AOF a la productividad, la latencia o el rendimiento de la memoria caché?

La persistencia de AOF afecta a la productividad aproximadamente entre un 15 y un 20 % cuando la memoria caché está por debajo de la carga máxima (carga de CPU y servidor ambas por debajo del 90 %). No debería haber problemas de latencia cuando la memoria caché está dentro de estos límites. Pero la memoria caché alcanza estos límites antes con AOF habilitado.

### <a name="how-can-i-remove-the-second-storage-account"></a>¿Cómo puedo quitar la segunda cuenta de almacenamiento?

Puede quitar la cuenta de almacenamiento secundaria de persistencia de AOF si establece la segunda cuenta de almacenamiento de modo que sea la misma que la primera cuenta de almacenamiento. Para obtener instrucciones, vea [Configuración de la persistencia de AOF](#configure-aof-persistence).

### <a name="what-is-a-rewrite-and-how-does-it-affect-my-cache"></a>¿Qué es una reescritura y cómo afecta a la memoria caché?

Cuando el archivo AOF es lo suficientemente grande, en la memoria caché se pone en cola automáticamente una reescritura. La reescritura cambia el tamaño del archivo AOF con el conjunto mínimo de operaciones necesario para crear el conjunto de datos actual. Durante las operaciones de reescritura debe esperar que se alcancen los límites de rendimiento antes, especialmente cuando se trabaja con grandes conjuntos de datos. Las operaciones de reescritura se producen con menos frecuencia a medida que el archivo AOF crece, aunque duran mucho tiempo cuando ocurren.

### <a name="what-should-i-expect-when-scaling-a-cache-with-aof-enabled"></a>¿Qué debo esperar al escalar una memoria caché con AOF habilitado?

Si al escalar el archivo AOF es muy grande, debe esperar que la operación de escalado tarde más de lo esperado, puesto que volverá a cargar el archivo una vez que termine el escalado.

Para más información sobre el escalado, vea [¿Qué sucede si he escalado a otro tamaño y se restaura una copia de seguridad realizada antes de la operación de escalado?](#what-happens-if-i-have-scaled-to-a-different-size-and-a-backup-is-restored-that-was-made-before-the-scaling-operation)

### <a name="how-is-my-aof-data-organized-in-storage"></a>¿Cómo se organizan los datos de AOF en el almacenamiento?

Los datos almacenados en archivos AOF se dividen en varios blobs en páginas por nodo para aumentar el rendimiento al guardar los datos en el almacenamiento. En la siguiente tabla se muestra cuántos blobs en páginas se usan en cada plan de tarifa:

| Nivel Premium | Datos BLOB |
|--------------|-------|
| P1           | 4 por partición    |
| P2           | 8 por partición    |
| P3           | 16 por partición   |
| P4           | 20 por partición   |

Cuando la agrupación en clústeres está habilitada, cada partición de la memoria caché tiene su propio conjunto de blobs en páginas, como se ha indicado en la tabla anterior. Por ejemplo, una caché P2 con tres particiones distribuye su archivo AOF entre 24 blobs en páginas (8 blobs por partición, con 3 particiones).

Después de una reescritura, hay dos conjuntos de archivos AOF en el almacenamiento. Las operaciones de reescritura se producen en segundo plano y se anexan al primer conjunto de archivos, mientras que las operaciones de conjunto que se envían a la memoria caché durante la reescritura se anexan al segundo conjunto. Durante las operaciones de reescritura se almacena de forma temporal una copia de seguridad por si hubiera un error, pero se elimina inmediatamente después de que finalice la reescritura.


## <a name="next-steps"></a>Pasos siguientes
Más información sobre las características de Azure Cache for Redis.

* [Niveles de servicio de Azure Cache for Redis Premium](cache-overview.md#service-tiers)

<!-- IMAGES -->

[redis-cache-premium-pricing-tier]: ./media/cache-how-to-premium-persistence/redis-cache-premium-pricing-tier.png

[redis-cache-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-persistence.png

[redis-cache-rdb-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-rdb-persistence.png

[redis-cache-aof-persistence]: ./media/cache-how-to-premium-persistence/redis-cache-aof-persistence.png

[redis-cache-settings]: ./media/cache-how-to-premium-persistence/redis-cache-settings.png
