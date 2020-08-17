---
title: Solución de problemas del lado cliente de Redis Cache
description: Obtenga información sobre cómo resolver problemas comunes del lado cliente con Azure Cache for Redis, como la presión de memoria del cliente de Redis, las ráfagas de tráfico, un uso elevado de la CPU, el ancho de banda limitado, las solicitudes grandes o el tamaño de respuesta grande.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: troubleshooting
ms.date: 10/18/2019
ms.openlocfilehash: 122c96c95aea794fbba9cab8a9a5b867f9f34b48
ms.sourcegitcommit: 98854e3bd1ab04ce42816cae1892ed0caeedf461
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "88008974"
---
# <a name="troubleshoot-azure-cache-for-redis-client-side-issues"></a>Solución de problemas del lado cliente de Redis Cache

En esta sección se describen problemas que se producen debido a una condición en la aplicación cliente.

- [Presión de memoria en el cliente de Redis](#memory-pressure-on-redis-client)
- [Ráfaga de tráfico](#traffic-burst)
- [Uso elevado de la CPU del cliente](#high-client-cpu-usage)
- [Limitación de ancho de banda del lado cliente](#client-side-bandwidth-limitation)
- [Tamaño de solicitud o respuesta grande](#large-request-or-response-size)

## <a name="memory-pressure-on-redis-client"></a>Presión de memoria en el cliente de Redis

La presión de memoria en la máquina del cliente genera todo tipo de problemas de rendimiento que pueden retrasar el procesamiento de respuestas de la caché. Cuando se alcanza la presión de memoria, el sistema puede paginar datos en el disco. Estos _errores de página_ hacen que el sistema se ralentice considerablemente.

Para detectar presión de memoria en el cliente:

- Supervise la utilización de memoria en la máquina para asegurarse de que no exceda la memoria disponible.
- Supervise el contador de rendimiento `Page Faults/Sec` del cliente. Durante el funcionamiento normal, la mayoría de los sistemas tienen algunos errores de página. Los picos de errores de página que corresponden con los tiempos de expiración de solicitudes pueden indicar la presión de memoria.

La presión de memoria alta en el cliente se puede mitigar varias maneras:

- Examine con más detalle los patrones de utilización de memoria para reducir el consumo de memoria en el cliente.
- Actualice la máquina virtual del cliente a un tamaño mayor con más memoria.

## <a name="traffic-burst"></a>Ráfaga de tráfico

Las ráfagas de tráfico se combinan con una mala configuración de `ThreadPool` pueden provocar retrasos en el procesamiento de datos ya enviados por el servidor Redis, pero que aún no se han consumido en el lado cliente.

Supervise cómo las estadísticas de `ThreadPool` cambian a lo largo del tiempo con [un ejemplo `ThreadPoolLogger`](https://github.com/JonCole/SampleCode/blob/master/ThreadPoolMonitor/ThreadPoolLogger.cs). Puede usar mensajes de `TimeoutException` de StackExchange.Redis, tal como aparece debajo, para investigar más a fondo:

```output
    System.TimeoutException: Timeout performing EVAL, inst: 8, mgr: Inactive, queue: 0, qu: 0, qs: 0, qc: 0, wr: 0, wq: 0, in: 64221, ar: 0,
    IOCP: (Busy=6,Free=999,Min=2,Max=1000), WORKER: (Busy=7,Free=8184,Min=2,Max=8191)
```

En la excepción anterior, hay varias incidencias que son interesantes:

- Observe que en la sección `IOCP` y la sección `WORKER` tiene un valor de `Busy` que es mayor que el valor de `Min`. Esta diferencia significa que es necesario ajustar la configuración de `ThreadPool`.
- También puede ver `in: 64221`. Este valor indica que se han recibido 64 211 bytes en la capa de sockets del kernel del cliente, pero que la aplicación aún no los ha leído. Esta diferencia normalmente significa que la aplicación (por ejemplo, StackExchange.Redis) no está leyendo los datos de la red con la rapidez con la que el servidor se los envía.

Puede [configurar las opciones de `ThreadPool`](cache-management-faq.md#important-details-about-threadpool-growth) para asegurarse de que su grupo de subprocesos se escala verticalmente a gran velocidad en escenarios de ráfaga.

## <a name="high-client-cpu-usage"></a>Uso elevado de la CPU del cliente

Un uso elevado de la CPU del cliente indica que el sistema no puede seguir el ritmo del trabajo que se le ha solicitado hacer. Aunque la caché haya enviado la respuesta rápidamente, es posible que el cliente no pueda procesar la respuesta de manera oportuna.

Supervise la utilización de la CPU de todo el sistema del cliente mediante las métricas disponibles en Azure Portal o a través de los contadores de rendimiento en la máquina. Tenga cuidado de no supervisar la CPU de un *proceso*, porque un solo proceso puede tener un uso de CPU bajo pero la CPU de todo el sistema puede ser elevada. Busque picos de uso de CPU que correspondan a tiempos de espera agotados. Una utilización de la CPU alta también puede provocar valores de `in: XXX` en mensajes de error de `TimeoutException`, como se describe en la sección [Ráfagas de tráfico](#traffic-burst).

> [!NOTE]
> StackExchange.Redis 1.1.603 y versiones posteriores incluyen la métrica `local-cpu` en mensajes de error de `TimeoutException`. Asegúrese de usar la versión más reciente del [paquete StackExchange.Redis NuGet](https://www.nuget.org/packages/StackExchange.Redis/). Hay errores que se solucionan constantemente en el código que lo hacen más solido frente a tiempos de espera agotados, por lo que es importante tener la versión más reciente.
>

Para mitigar la utilización de la CPU elevada de un cliente:

- Investigue lo que está provocando los picos en la CPU.
- Actualice la máquina virtual del cliente a un tamaño mayor con más capacidad de CPU.

## <a name="client-side-bandwidth-limitation"></a>Limitación de ancho de banda del lado cliente

En función de la arquitectura de los equipos cliente, estos pueden tener limitaciones en el ancho de banda de red disponible. Si el cliente supera el ancho de banda disponible por una sobrecarga de la capacidad de red, los datos no se procesarán en el lado cliente con la rapidez con la que el servidor se los envía. Esta situación puede provocar tiempos de espera.

Supervise cómo cambia la utilización del ancho de banda a lo largo del tiempo mediante [un ejemplo `BandwidthLogger`](https://github.com/JonCole/SampleCode/blob/master/BandWidthMonitor/BandwidthLogger.cs). Este código puede no ejecutarse correctamente en algunos entornos con permisos restringidos (como sitios web de Azure).

Para mitigarlo, reduzca el consumo de ancho de banda de red o aumente el tamaño de la máquina virtual del cliente a una con más capacidad de red.

## <a name="large-request-or-response-size"></a>Tamaño de solicitud o respuesta grande

Una solicitud/respuesta grande puede provocar tiempos de espera agotados. Por ejemplo, suponga que el valor del tiempo de expiración configurado en el cliente es de 1 segundo. La aplicación solicita dos claves (por ejemplo, "A" y "B") al mismo tiempo (con la utilización de la misma conexión de red física). La mayoría de clientes admiten la "canalización" de las solicitudes, en la que ambas solicitudes, "A" y "B", se envían una tras otra sin tener que esperar las respuestas. El servidor vuelve a enviar las respuestas en el mismo orden. Si la respuesta "A" es grande, puede consumir la mayor parte del tiempo de expiración para las solicitudes posteriores.

En el ejemplo siguiente, las solicitudes "A" y "B" se envían rápidamente al servidor. El servidor empieza a enviar las respuestas "A" y "B" rápidamente. Debido a los tiempos de transferencia de datos, la respuesta "B" debe esperar a que se agote el tiempo de espera de la respuesta "A", aunque el servidor haya respondido con rapidez.

```console
|-------- 1 Second Timeout (A)----------|
|-Request A-|
     |-------- 1 Second Timeout (B) ----------|
     |-Request B-|
            |- Read Response A --------|
                                       |- Read Response B-| (**TIMEOUT**)
```

Esta solicitud/respuesta es difícil de medir. Puede instrumentar el código del cliente para realizar un seguimiento de las respuestas y solicitudes grandes.

Las resoluciones para los tamaños grandes de respuesta varían, pero incluyen lo siguiente:

1. Optimización de la aplicación para un gran número de valores pequeños, en lugar de unos pocos valores grandes.
    - La mejor solución es dividir los datos en valores menores relacionados.
    - Consulte la publicación [What is the ideal value size range for redis? Is 100KB too large?](https://groups.google.com/forum/#!searchin/redis-db/size/redis-db/n7aa2A4DZDs/3OeEPHSQBAAJ) (¿Cuál es el intervalo de tamaño de valor ideal para Redis? ¿100 KB es demasiado grande?) para obtener detalles de por qué se recomiendan valores más pequeños.
1. Aumento del tamaño de la máquina virtual para obtener mayores capacidades de ancho de banda.
    - Un aumento de ancho de banda en la máquina virtual del cliente o servidor puede reducir los tiempos de transferencia de datos para respuestas más grandes.
    - Compare la utilización actual de la red en ambas máquinas con los límites del tamaño actual de la máquina virtual. Es posible que obtener más ancho de banda solo en el servidor o en el cliente no sea suficiente.
1. Aumente el número de objetos de conexión que usa la aplicación.
    - Use un enfoque round robin para realizar solicitudes a través de objetos de conexión distintos.

## <a name="additional-information"></a>Información adicional

- [Solución de problemas del lado servidor de Redis Cache](cache-troubleshoot-server.md)
- [¿Cómo se pueden realizar bancos de pruebas y probar el rendimiento del caché?](cache-management-faq.md#how-can-i-benchmark-and-test-the-performance-of-my-cache)
