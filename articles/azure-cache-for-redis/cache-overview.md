---
title: ¿Qué es Azure Redis Cache? | Microsoft Docs
description: Aprenda qué es Azure Redis Cache y cómo se utiliza habitualmente.
services: cache
documentationcenter: ''
author: yegu-ms
manager: jhubbard
editor: ''
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache
ms.devlang: na
ms.topic: overview
ms.date: 03/26/2018
ms.author: yegu
ms.custom: mvc
ms.openlocfilehash: fd790d27c958bf982f95b98426c6ab4d94c5f17f
ms.sourcegitcommit: 300cd05584101affac1060c2863200f1ebda76b7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2019
ms.locfileid: "65412726"
---
# <a name="azure-cache-for-redis-description"></a>Descripción de Azure Redis Cache

Azure Cache for Redis se basa en el popular software [Redis](https://redis.io/). Normalmente se usa como una memoria caché para mejorar el rendimiento y la escalabilidad de aquellos sistemas que dependen mucho de almacenes de datos back-end. La mejora del rendimiento se logra mediante la copia de los datos a los que se accede con mayor frecuencia en un almacenamiento rápido ubicado cerca de la aplicación. Con [Azure Redis Cache](https://redis.io/), este almacenamiento rápido se coloca en la memoria con Azure Redis Cache en lugar de que una base de datos lo cargue desde el disco.

Azure Redis Cache también puede usarse como almacén de estructura de datos en memoria, base de datos no relacional distribuida y agente de mensajes. El rendimiento de las aplicaciones mejora porque se aprovecha de la baja latencia y alto rendimiento del motor Redis.

Azure Redis Cache le proporciona acceso a una caché en Redis segura y dedicada. Azure Redis Cache lo administra Microsoft, está hospedado en Azure y cualquier aplicación de dentro o fuera de Azure puede acceder a él.

## <a name="using-azure-cache-for-redis"></a>Uso de Azure Redis Cache

Hay muchos patrones comunes en los que se usa Azure Redis Cache para admitir la arquitectura de las aplicaciones o para mejorar el rendimiento de estas. Estos son algunos de los más comunes:

| Patrón      | Descripción                                        |
| ------------ | -------------------------------------------------- |
| [Cache-Aside](cache-web-app-cache-aside-leaderboard.md) | Dado que una base de datos puede ser grande, no se recomienda cargarla toda ella en la memoria caché. Es habitual usar el patrón [cache-aside](https://docs.microsoft.com/azure/architecture/patterns/cache-aside) para cargar elementos de datos en la memoria caché solo cuando sea necesario. Cuando el sistema realiza cambios en los datos de back-end, también puede actualizar la caché, que se distribuye con otros clientes. Además, el sistema puede establecer una fecha de expiración en los elementos de datos o usar una directiva de expulsión para que las actualizaciones de los datos se vuelvan a cargar en la memoria caché.|
| [Almacenamiento en caché de contenido](cache-aspnet-output-cache-provider.md) | La mayoría de las páginas web se generan a partir de plantillas con encabezados, pies de página, barras de herramientas, menús, etc. Realmente no cambian con frecuencia y no se deben generar dinámicamente. El uso de una memoria caché en memoria, como Azure Redis Cache, proporcionará a los servidores web acceso rápido a este tipo de contenido estático, en comparación con almacenes de datos de back-end. Este patrón reduce tanto el tiempo de procesamiento como la carga del servidor necesarias para generar contenido dinámicamente, lo que mejora la capacidad de respuesta de los servidores web y le puede permiten reducir el número de servidores necesarios para controlar cargas. Azure Redis Cache proporciona el proveedor de caché de salida de Redis, que ayuda a admitir este patrón con ASP.NET.|
| [Almacenamiento en caché de sesión de usuario](cache-aspnet-session-state-provider.md) | Este patrón se utiliza normalmente con carros de la compra y otra información de tipo de historial de usuario que una aplicación web puede desear asociar con las cookies del usuario. El almacenamiento de demasiados datos en una cookie puede tener un impacto negativo en el rendimiento, ya que aumenta su tamaño y no hay que olvidar que se pasa y se valida con cada solicitud. Una solución habitual es usar la cookie como clave cuando se consultan datos en una base de datos del back-end. El uso de una memoria caché en memoria, como Azure Redis Cache, para asociar información a un usuario es mucho más rápido que su interacción con una base de datos relacional completa. |
| Almacenamiento en cola de trabajos y mensajes | Cuando las aplicaciones reciben solicitudes, a menudo las operaciones asociadas con la solicitud tardan más tiempo en ejecutarse. Es una práctica común para aplazar las operaciones que más tardan en ejecutarse mediante su inclusión en una cola, que se procesa posteriormente, y posiblemente por otro servidor. Este método de aplazar trabajo se denomina puesta en cola de tareas. Hay muchos componentes de software diseñados para admitir colas de tareas. Azure Redis Cache también se usa para este fin, así como cola distribuida.|
| Transacciones distribuidas | Es un requisito común que las aplicaciones puedan ejecutar una serie de comandos en un almacén de datos de back-end en una única operación (atómica). El resultado de todos los comandos debe ser satisfactorio, o todos deben revertirse al estado inicial. Azure Redis Cache admite la ejecución de un lote de comandos como una única operación en forma de [transacciones](https://redis.io/topics/transactions). |

## <a name="azure-cache-for-redis-offerings"></a>Ofertas de Azure Redis Cache

Azure Redis Cache está disponible en los niveles siguientes:

| Nivel | Descripción |
|---|---|
Básico | Caché de un solo nodo. Este nivel admite varios tamaños de memoria (250 MB - 53 GB). Es un nivel ideal para el desarrollo o la realización de pruebas, así como para cargas de trabajo no críticas. El nivel Básico no tiene ningún Acuerdo de Nivel de Servicio (SLA) |
| Estándar | Caché replicada en una configuración de dos nodos, principal y secundario, administrada por Microsoft y con un Acuerdo de Nivel de Servicio de alta disponibilidad (99,9 %). |
| Premium | El nivel Premium es el nivel específico para la empresa. Las memorias caché de nivel Premium admiten más características y tienen un rendimiento mayor con latencias más bajas. En el nivel Premium, las memorias caché se implementan en un hardware más potente, lo que proporciona un mejor rendimiento en comparación con los niveles Estándar o Básico. Esta ventaja significa que el rendimiento de una caché del mismo tamaño será mayor en el nivel Premium, en comparación con el nivel Estándar. |

> [!TIP]
> Para más información acerca del tamaño, la transferencia y el ancho de banda de las memorias caché de nivel Premium, consulte el artículo [Azure Redis Cache FAQ](cache-faq.md#what-azure-cache-for-redis-offering-and-size-should-i-use) (Preguntas más frecuentes de Azure Redis Cache).
>

Una vez creada, la memoria caché se puede escalar a un nivel superior. Sin embargo, no se admite su reducción vertical. Para instrucciones detalladas acerca del escalado, consulte [How to Scale Azure Cache for Redis](cache-how-to-scale.md) (Escalado de Azure Redis Cache) y [How to automate a scaling operation](cache-how-to-scale.md#how-to-automate-a-scaling-operation) (Automatización de una operación de escalado).

### <a name="feature-comparison"></a>Comparación de características

En la página [Precios de Azure Cache for Redis](https://azure.microsoft.com/pricing/details/cache/) encontrará una comparación detallada de cada nivel. La tabla siguiente le ayuda a describir algunas de las características que admite cada nivel:

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

## <a name="next-steps"></a>Pasos siguientes

* [Inicio rápido de una aplicación web de ASP.NET](cache-web-app-howto.md) Cree una aplicación web de ASP.NET simple que use Azure Redis Cache.
* [Inicio rápido de .NET](cache-dotnet-how-to-use-azure-redis-cache.md) Cree una aplicación .NET que use Azure Redis Cache.
* [Inicio rápido de .NET Core](cache-dotnet-core-quickstart.md) Cree una aplicación .NET Core que use Azure Redis Cache.
* [Inicio rápido de Node.js](cache-nodejs-get-started.md) Cree una aplicación Node.js simple que use Azure Redis Cache.
* [Inicio rápido de Java](cache-java-get-started.md) Cree una aplicación Java simple que use Azure Redis Cache.
* [Inicio rápido de Python](cache-python-get-started.md) Cree una aplicación Python que use Azure Redis Cache.
