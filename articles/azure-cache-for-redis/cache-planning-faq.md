---
title: Preguntas frecuentes sobre Azure Cache for Redis
description: Conozca las respuestas a preguntas comunes que le ayudarán a planear Azure Cache for Redis.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 08/06/2020
ms.openlocfilehash: 13ba529dd3067ae16167f0d9c14c8f72b982f52c
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88010789"
---
# <a name="azure-cache-for-redis-planning-faqs"></a>Preguntas frecuentes sobre Azure Cache for Redis

En este artículo se proporcionan respuestas a preguntas habituales sobre cómo planear Azure Cache for Redis.

## <a name="common-questions-and-answers"></a>Preguntas y respuestas habituales
En esta sección se incluyen las siguientes preguntas frecuentes:

* [Rendimiento de Azure Cache for Redis](#azure-cache-for-redis-performance)
* [¿En qué región debo buscar mi caché?](#in-what-region-should-i-locate-my-cache)
* [¿Dónde residen mis datos en caché?](#where-do-my-cached-data-reside)
* [¿Cómo se factura Azure Cache for Redis?](#how-am-i-billed-for-azure-cache-for-redis)
* [¿Puedo utilizar Azure Cache for Redis con la nube de Azure Government, la nube de Azure China 21Vianet o Microsoft Azure Alemania?](#can-i-use-azure-cache-for-redis-with-azure-government-cloud-azure-china-21vianet-cloud-or-microsoft-azure-germany)

### <a name="azure-cache-for-redis-performance"></a>Rendimiento de Azure Cache for Redis
En la tabla siguiente se muestran los valores máximos del ancho de banda observados durante la comprobación de los diversos tamaños de memorias caché Estándar y Prémium mediante `redis-benchmark.exe` desde una máquina virtual de IaaS en el punto de conexión de Azure Cache for Redis. Para el rendimiento de TLS, se usa redis-benchmark con stunnel a fin de conectarse al punto de conexión de Azure Cache for Redis.

>[!NOTE] 
>Estos valores no se garantizan y no hay ningún SLA para estos números, sino que deben ser los habituales. Debe realizar la prueba de carga de su propia aplicación para determinar el tamaño adecuado de caché para la aplicación.
>Estas cifras pueden cambiar, puesto que se publican resultados más recientes de manera periódica.
>

A partir de esta tabla, podemos extraer las conclusiones siguientes:

* El rendimiento de las memorias caché que tienen el mismo tamaño es mayor en el nivel Premium que en el nivel Estándar. Por ejemplo, en el caso de una memoria caché de 6 GB, el rendimiento de P1 es de 180 000 solicitudes por segundo (RPS) frente a las 100 000 de C3.
* Con la agrupación en clústeres de Redis, el rendimiento aumenta de manera lineal a medida que aumenta el número de particiones (nodos) del clúster. Por ejemplo, si se crea un clúster P4 de 10 particiones, el rendimiento disponible es de 400 000 * 10 = 4 millones de solicitudes por segundo.
* El rendimiento para los tamaños de clave más grandes es mayor en el nivel Premium que en el nivel Estándar.

| Plan de tarifa | Size | Núcleos de CPU | Ancho de banda disponible | Tamaño del valor de 1 kB | Tamaño del valor de 1 kB |
| --- | --- | --- | --- | --- | --- |
| **Tamaños de caché estándar** | | |**Megabits por segundo (Mb/s) o Megabytes por segundo (MB/s)** |**Solicitudes por segundo (RPS) sin SSL** |**Solicitudes por segundo (RPS) SSL** |
| C0 | 250 MB | Compartido | 100 / 12.5  |  15,000 |   7 500 |
| C1 |   1 GB | 1      | 500 / 62.5  |  38 000 |  20 720 |
| C2 | 2,5 GB | 2      | 500 / 62.5  |  41 000 |  37 000 |
| C3 |   6 GB | 4      | 1000 / 125  | 100 000 |  90 000 |
| C4 |  13 GB | 2      | 500 / 62.5  |  60 000 |  55 000 |
| C5 |  26 GB | 4      | 1,000 / 125 | 102 000 |  93 000 |
| C6 |  53 GB | 8      | 2,000 / 250 | 126 000 | 120 000 |
| **Tamaños de caché Premium** | |**Núcleos de CPU por partición** | **Megabits por segundo (Mb/s) o Megabytes por segundo (MB/s)** |**Solicitudes por segundo (RPS) sin SSL por partición** |**Solicitudes por segundo (RPS) SSL por partición** |
| P1 |   6 GB |  2 | 1,500 / 187.5 | 180,000 | 172 000 |
| P2 |  13 GB |  4 | 3,000 / 375   | 350 000 | 341 000 |
| P3 |  26 GB |  4 | 3,000 / 375   | 350 000 | 341 000 |
| P4 |  53 GB |  8 | 6,000 / 750   | 400.000 | 373 000 |
| P5 | 120 GB | 20 | 6,000 / 750   | 400.000 | 373 000 |

Para instrucciones sobre cómo configurar stunnel o descargar las herramientas de Redis, como `redis-benchmark.exe`, consulte [¿Cómo puedo ejecutar los comandos de Redis?](cache-development-faq.md#how-can-i-run-redis-commands).

### <a name="in-what-region-should-i-locate-my-cache"></a>¿En qué región debo buscar mi caché?
Para el mejor rendimiento y la menor latencia, sitúe su instancia de Azure Cache for Redis en la misma región que la aplicación cliente de la caché.

### <a name="where-do-my-cached-data-reside"></a>¿Dónde residen mis datos en caché?
Azure Cache for Redis almacena los datos de la aplicación en la memoria RAM de la máquina virtual (o las máquinas virtuales, según el nivel), que hospeda la caché. Los datos residen estrictamente en la región de Azure que ha seleccionado de forma predeterminada. Hay dos casos en los que los datos pueden dejar una región:
* Al habilitar la persistencia en la caché, Azure Cache for Redis realizará una copia de seguridad de los datos en una cuenta de Azure Storage que posea. Si la cuenta de almacenamiento que proporciona se encuentra en otra región, una copia de los datos terminará allí.
* Si configura la replicación geográfica y la caché secundaria se encuentra en otra región, que sería lo habitual, los datos se replicarán en esa región.

Tendrá que configurar explícitamente Azure Cache for Redis para usar estas características. También tiene control total sobre la región en la que se encuentra la cuenta de almacenamiento o la caché secundaria.

### <a name="how-am-i-billed-for-azure-cache-for-redis"></a>¿Cómo se factura Azure Cache for Redis?
[Aquí](https://azure.microsoft.com/pricing/details/cache/) se encuentran los precios de Azure Cache for Redis. La página de precios muestra los precios como tarifa por hora y mensual. Las memorias caché se facturan por minuto desde el momento en que se crea la memoria caché hasta el momento en que se elimina una memoria caché. No hay ninguna opción de detener o pausar la facturación de una memoria caché.

### <a name="can-i-use-azure-cache-for-redis-with-azure-government-cloud-azure-china-21vianet-cloud-or-microsoft-azure-germany"></a>¿Puedo utilizar Azure Cache for Redis con la nube de Azure Government, la nube de Azure China 21Vianet o Microsoft Azure Alemania?
Sí, Azure Cache for Redis está disponible en la nube de Azure Government, la nube de Azure China 21Vianet y Microsoft Azure Alemania. Las direcciones URL para acceder a Azure Cache for Redis y administrar esta solución son diferentes en estas nubes en comparación con la nube pública de Azure.

| Nube   | Sufijo DNS para Redis            |
|---------|---------------------------------|
| Público  | *.redis.cache.windows.net       |
| US Gov  | *.redis.cache.usgovcloudapi.net |
| Alemania | *.redis.cache.cloudapi.de       |
| China   | *.redis.cache.chinacloudapi.cn  |

Para más información sobre el uso de Azure Cache for Redis con otras nubes, consulte los vínculos siguientes.

- [Bases de datos de Azure Government: Azure Cache for Redis](../azure-government/documentation-government-services-database.md#azure-cache-for-redis)
- [Nube de Azure China 21Vianet: Azure Cache for Redis](https://www.azure.cn/home/features/redis-cache/)
- [Microsoft Azure Alemania](https://azure.microsoft.com/overview/clouds/germany/)

Para más información sobre el uso de Azure Cache for Redis con PowerShell en la nube de Azure Government, la nube de Azure China 21Vianet y Microsoft Azure Alemania, consulte [cómo conectar con otras nubes: PowerShell con Azure Cache for Redis](cache-how-to-manage-redis-cache-powershell.md#how-to-connect-to-other-clouds).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre otras [preguntas frecuentes sobre Azure Cache for Redis](cache-faq.md).
