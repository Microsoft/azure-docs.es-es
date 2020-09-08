---
title: Preguntas frecuentes sobre la supervisión y solución de problemas de Azure Cache for Redis
description: Conozca las respuestas a preguntas comunes que le ayudarán a supervisar y solucionar problemas de Azure Cache for Redis.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 08/06/2020
ms.openlocfilehash: 2149d069ce9cb0b636e461d19963f413b162bc9d
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88010797"
---
# <a name="azure-cache-for-redis-monitoring-and-troubleshooting-faqs"></a>Preguntas frecuentes sobre la supervisión y solución de problemas de Azure Cache for Redis
En este artículo se proporcionan respuestas a preguntas habituales sobre cómo administrar y solucionar los problemas de Azure Cache for Redis.

## <a name="common-questions-and-answers"></a>Preguntas y respuestas habituales
En esta sección se incluyen las siguientes preguntas frecuentes:

* [¿Cómo se puede supervisar el estado y el rendimiento de la memoria caché?](#how-do-i-monitor-the-health-and-performance-of-my-cache)
* [¿Por qué estoy viendo los tiempos de expiración?](#why-am-i-seeing-timeouts)
* [¿Por qué se desconectó el cliente desde la memoria caché?](#why-was-my-client-disconnected-from-the-cache)

### <a name="how-do-i-monitor-the-health-and-performance-of-my-cache"></a>¿Cómo se puede supervisar el estado y el rendimiento de la memoria caché?
Se pueden supervisar las instancias de Microsoft Azure Cache for Redis en [Azure Portal](https://portal.azure.com). Puede ver las métricas, anclar los gráficos de métricas al panel de inicio, personalizar el intervalo de fecha y hora de los gráficos de supervisión, agregar y quitar métricas de los gráficos y establecer alertas cuando se cumplen determinadas condiciones. Para más información, consulte [How to monitor Azure Cache for Redis](cache-how-to-monitor.md) (Supervisión de Azure Cache for Redis).

El **menú de recursos** de Azure Cache for Redis también contiene varias herramientas para supervisar las cachés y solucionar sus problemas.

* En **Diagnosticar y solucionar problemas** se proporciona información sobre los problemas comunes y las estrategias para resolverlos.
* **estado de los recursos** supervisa el recurso e indica si se ejecuta del modo previsto. Para obtener más información sobre el servicio Estado de los recursos de Azure, consulte [Información general sobre Estado de los recursos de Azure](../resource-health/resource-health-overview.md).
* **Nueva solicitud de soporte** para abrir una solicitud de soporte técnico para su memoria caché.

Estas herramientas permiten supervisar el estado de las instancias de Azure Cache for Redis y le ayudan a administrar sus aplicaciones de almacenamiento en caché. Para más información, vea la sección "Support & troubleshooting settings" (Configuración de soporte técnico y solución de problemas) de [How to configure Azure Cache for Redis](cache-configure.md) (Configuración de Azure Cache for Redis).

### <a name="why-am-i-seeing-timeouts"></a>¿Por qué estoy viendo los tiempos de expiración?
Los tiempos de expiración se producen en el cliente que usa para comunicarse con Redis. Cuando se envía un comando al servidor de Redis, el comando se pone en cola y el servidor de Redis finalmente recoge el comando y lo ejecuta. Sin embargo el cliente puede agotar el tiempo de espera durante este proceso y, si lo hace, se produce una excepción en el lado de la llamada. Para más información sobre la solución de problemas de tiempo de espera, consulte [Solución de problemas del lado cliente](cache-troubleshoot-client.md) y [Excepciones de tiempo de espera de StackExchange.Redis](cache-troubleshoot-timeouts.md#stackexchangeredis-timeout-exceptions).

### <a name="why-was-my-client-disconnected-from-the-cache"></a>¿Por qué se desconectó el cliente desde la memoria caché?
A continuación se indican algunas razones habituales por las que se desconecta la memoria caché.

* Causas de cliente
  * Se volvió a implementar la aplicación cliente.
  * La aplicación cliente realiza una operación de escalado.
    * En el caso de Cloud Services o Web Apps, puede deberse al escalado automático.
  * Cambia el nivel de red del cliente.
  * Se produjeron errores transitorios en el cliente o en los nodos de red entre el cliente y el servidor.
  * Se han alcanzado los límites de umbral de ancho de banda.
  * Las operaciones de la CPU tardaron demasiado tiempo en completarse.
* Causas de servidor
  * En la oferta de caché estándar, el servicio Azure Cache for Redis inició una conmutación por error desde el nodo principal al nodo de réplica.
  * Azure revisó la instancia donde se implementó la memoria caché
    * Esto puede servir para actualizaciones del servidor de Redis o para el mantenimiento general de máquinas virtuales.


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la supervisión y la solución de problemas de las instancias de Azure Cache for Redis, consulte [Supervisión de Azure Cache for Redis](cache-how-to-monitor.md) y las diferentes guías de solución de problemas.

Más información sobre otras [preguntas frecuentes sobre Azure Cache for Redis](cache-faq.md).
