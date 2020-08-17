---
title: Solución de problemas del lado servidor de Redis Cache
description: Obtenga información sobre cómo resolver problemas comunes del lado servidor con Azure Cache for Redis, como la presión de memoria, un uso elevado de la CPU, los comandos de larga duración o las limitaciones de ancho de banda.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 10/18/2019
ms.openlocfilehash: 12d78846f5892e71388de6e6e76b868f9b14d4de
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008923"
---
# <a name="troubleshoot-azure-cache-for-redis-server-side-issues"></a>Solución de problemas del lado servidor de Redis Cache

En esta sección se describen problemas que se producen debido a una condición en una instancia de Redis Cache o en las máquinas virtuales que la hospedan.

- [Presión de memoria en el servidor de Redis](#memory-pressure-on-redis-server)
- [Uso elevado de la CPU o carga del servidor](#high-cpu-usage-or-server-load)
- [Comandos de ejecución prolongada](#long-running-commands)
- [Limitación de ancho de banda del lado servidor](#server-side-bandwidth-limitation)

> [!NOTE]
> Algunos de los pasos de solución de problemas de esta guía incluyen instrucciones para ejecutar comandos de Redis y supervisar diversas métricas de rendimiento. Para más información e instrucciones, consulte los artículos de la sección [Información adicional](#additional-information) .
>

## <a name="memory-pressure-on-redis-server"></a>Presión de memoria en el servidor de Redis

La presión de memoria en el servidor genera todo tipo de problemas de rendimiento que pueden retrasar el procesamiento de solicitudes. Cuando se alcanza la presión de memoria, el sistema puede paginar datos en el disco. Estos _errores de página_ hacen que el sistema se ralentice considerablemente. Hay varias causas posibles de esta presión de memoria:

- La caché se rellena con los datos cerca de su capacidad máxima.
- Redis ve la fragmentación de memoria alta. El almacenamiento de objetos grandes es lo que suele provocar esta fragmentación, ya que Redis está optimizado para objetos pequeños.

Redis expone dos estadísticas a través del comando [INFO](https://redis.io/commands/info) que ayuda a identificar esta incidencia: "used_memory" y "used_memory_rss". También puede [ver estas métricas](cache-how-to-monitor.md#view-metrics-with-azure-monitor) mediante el portal.

Hay varios cambios posibles que puede hacer para ayudar a mantener una utilización de memoria correcta:

- [Configurar una directiva de memoria](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) y establecer tiempos de expiración de las claves. Esta directiva puede no ser suficiente si tiene fragmentación.
- [Configurar un valor rmaxmemory-reserved](cache-configure.md#maxmemory-policy-and-maxmemory-reserved) lo suficientemente grande como para compensar la fragmentación de la memoria.
- Dividir los objetos grandes que están en la memoria caché en objetos relacionados más pequeños.
- [Crear alertas](cache-how-to-monitor.md#alerts) en métricas como la memoria usada para recibir una notificación anticipada sobre los impactos potenciales.
- [Escalar](cache-how-to-scale.md) a un tamaño mayor de caché con más capacidad de memoria.

## <a name="high-cpu-usage-or-server-load"></a>Uso elevado de la CPU o carga del servidor

Una carga o utilización de la CPU elevada en el servidor significa que el servidor no puede procesar solicitudes de manera oportuna. Puede que el servidor tarde en responder y sea capaz de seguir el ritmo con las tasas de solicitud.

[Supervise las métricas](cache-how-to-monitor.md#view-metrics-with-azure-monitor), como la carga de la CPU o el servidor. Busque picos de uso de CPU que correspondan a tiempos de espera agotados.

Hay varios cambios que puede hacer para mitigar la carga elevada en el servidor:

- Investigar lo que provoca picos de la CPU tales como los [comandos de ejecución prolongada](#long-running-commands) que se indican a continuación o los errores de página debido a la presión de memoria alta.
- [Crear alertas](cache-how-to-monitor.md#alerts) en métricas como la carga de la CPU o el servidor para recibir una notificación anticipada sobre los impactos potenciales.
- [Escalar](cache-how-to-scale.md) a un tamaño mayor de caché con más capacidad de CPU.

## <a name="long-running-commands"></a>Comandos de ejecución prolongada

Algunos comandos de Redis son más caros de ejecutar que otros. La [documentación de comandos de Redis](https://redis.io/commands) muestra la complejidad de tiempo de cada comando. Dado que el procesamiento de comandos de Redis es uniproceso, un comando que tarde en ejecutarse bloqueará todos los demás que vayan después. Debe revisar los comandos que emite al servidor de Redis para comprender sus efectos en el rendimiento. Por ejemplo, el comando [KEYS](https://redis.io/commands/keys) a menudo se usa el sin saber que se trata de una operación O(N). Puede evitar el comando KEYS mediante el uso de [SCAN](https://redis.io/commands/scan) para reducir los picos de la CPU.

Mediante el comando [SLOWLOG](https://redis.io/commands/slowlog), puede medir los comandos caros que se ejecutan en el servidor.

## <a name="server-side-bandwidth-limitation"></a>Limitación de ancho de banda del lado servidor

Tamaños de caché diferentes tienen capacidades distintas de ancho de banda de red. Si el servidor supera el ancho de banda disponible, los datos no se enviarán al cliente con la misma rapidez. Las solicitudes de los clientes podrían agotar el tiempo de espera debido a que el servidor no puede insertar datos al cliente lo suficientemente rápido.

Las métricas de "Lectura de caché" y "Escritura de caché" pueden usarse para ver el ancho de banda del lado servidor que se está usando. Puede [ver estas métricas](cache-how-to-monitor.md#view-metrics-with-azure-monitor) en el portal.

Para mitigar las situaciones en las que la utilización de ancho de banda de red está cerca de la capacidad máxima, haga lo siguiente:

- Cambiar el comportamiento de la llamada de cliente para reducir la demanda de la red.
- [Crear alertas](cache-how-to-monitor.md#alerts) en métricas como la lectura de caché o la escritura de caché para recibir una notificación anticipada sobre los impactos potenciales.
- [Escalar](cache-how-to-scale.md) a un tamaño mayor de caché con más capacidad de ancho de banda de red.

## <a name="additional-information"></a>Información adicional

- [Solución de problemas del lado cliente de Redis Cache](cache-troubleshoot-client.md)
- [Elección del nivel correcto](cache-overview.md#choosing-the-right-tier)
- [¿Cómo se pueden realizar bancos de pruebas y probar el rendimiento del caché?](cache-management-faq.md#how-can-i-benchmark-and-test-the-performance-of-my-cache)
- [Supervisión de Azure Cache for Redis](cache-how-to-monitor.md)
- [¿Cómo puedo ejecutar comandos de Redis?](cache-development-faq.md#how-can-i-run-redis-commands)
