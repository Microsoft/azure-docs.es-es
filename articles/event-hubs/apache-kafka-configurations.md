---
title: 'Configuraciones recomendadas para clientes de Apache Kafka: Azure Event Hubs'
description: En este artículo se proporcionan configuraciones de Apache Kafka recomendadas para los clientes que interactúan con Azure Event Hubs para Apache Kafka.
ms.topic: reference
ms.date: 07/20/2020
ms.openlocfilehash: f9a03d1d3433461a575b32cd69893408a8b0ef97
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87095108"
---
# <a name="recommended-configurations-for-apache-kafka-clients"></a>Configuraciones recomendadas para clientes de Apache Kafka
Estas son las configuraciones recomendadas para usar Azure Event Hubs desde las aplicaciones cliente de Apache Kafka. 

## <a name="java-client-configuration-properties"></a>Propiedades de configuración del cliente de Java

### <a name="producer-and-consumer-configurations"></a>Configuraciones de productos y consumidor

Propiedad | Valores recomendados | Intervalo permitido | Notas
---|---:|-----:|---
`metadata.max.age.ms` | 180 000 (aproximado) | <240 000 | Se puede reducir para recoger los cambios de metadatos antes.
`connections.max.idle.ms`   | 180000 | <240 000 | Azure cierra la inactividad de TCP entrante > 240 000 ms, lo que puede dar lugar a un envío de conexiones inactivas (que se muestran como lotes expirados debido al tiempo de espera de envío).

### <a name="producer-configurations-only"></a>Solo configuraciones de productor
Las configuraciones de productor se pueden encontrar [aquí](https://kafka.apache.org/documentation/#producerconfigs).

Propiedad | Valores recomendados | Intervalo permitido | Notas
---|---:|---:|---
`max.request.size` | 1000000 | <1 046 528 | El servicio cerrará las conexiones si se envían solicitudes de más de 1 046 528 bytes.  *Este valor **debe** cambiar y causará problemas en escenarios de producción de alto rendimiento.*
`retries` | >0 | | Puede que sea necesario aumentar el valor de delivery.timeout.ms; consulte la documentación.
`request.timeout.ms` | 30 000 .. 60000 | >20 000| EH se establecerá internamente de forma predeterminada en un mínimo de 20 000 ms.  *Aunque se aceptan solicitudes con valores de tiempo de espera menores, no se garantiza el comportamiento del cliente.*
`metadata.max.idle.ms` | 180000 | >5000 | Controla cuánto tiempo el productor almacenará en memoria caché los metadatos de un tema que está inactivo. Si el tiempo transcurrido desde la última vez que se generó un tema supera la duración de inactividad de los metadatos, los metadatos del tema se olvidan y el siguiente acceso a estos forzará la solicitud de captura de metadatos.
`linger.ms` | >0 | | En escenarios de alto rendimiento, el valor de permanencia debe ser igual al valor tolerable más alto para aprovechar el procesamiento por lotes.
`delivery.timeout.ms` | | | Se establece según la fórmula (`request.timeout.ms` + `linger.ms`) * `retries`.
`enable.idempotence` | false | | La idempotencia no se admite actualmente.
`compression.type` | `none` | | La compresión no se admite actualmente.

### <a name="consumer-configurations-only"></a>Solo configuraciones de consumidor
Las configuraciones de consumidor se pueden encontrar [aquí](https://kafka.apache.org/documentation/#consumerconfigs).

Propiedad | Valores recomendados | Intervalo permitido | Notas
---|---:|-----:|---
`heartbeat.interval.ms` | 3000 | | 3000 es el valor predeterminado y no debe cambiarse.
`session.timeout.ms` | 30000 |6000 .. 300000| Comience con 30 000 y auméntelo si ve un reequilibrio frecuente debido a los latidos que faltan.


## <a name="librdkafka-configuration-properties"></a>Propiedades de configuración de librdkafka
El archivo de configuración principal de `librdkafka` ([vínculo](https://github.com/edenhill/librdkafka/blob/master/CONFIGURATION.md)) contiene descripciones extendidas de las propiedades siguientes.

### <a name="producer-and-consumer-configurations"></a>Configuraciones de productos y consumidor

Propiedad | Valores recomendados | Intervalo permitido | Notas
---|---:|-----:|---
`socket.keepalive.enable` | true | | Necesario si se espera que la conexión esté inactiva.  Azure cerrará la inactividad de TCP entrante > 240 000 ms.
`metadata.max.age.ms` | ~ 180 000| <240 000 | Se puede reducir para recoger los cambios de metadatos antes.

### <a name="producer-configurations-only"></a>Solo configuraciones de productor

Propiedad | Valores recomendados | Intervalo permitido | Notas
---|---:|-----:|---
`retries` | >0 | | El valor predeterminado es 2. Se recomienda mantener este valor. 
`request.timeout.ms` | 30 000 .. 60000 | >20 000| EH se establecerá internamente de forma predeterminada en un mínimo de 20 000 ms.  El valor predeterminado de `librdkafka` es 5000, que puede ser problemático. *Aunque se aceptan solicitudes con valores de tiempo de espera menores, no se garantiza el comportamiento del cliente.*
`partitioner` | `consistent_random` | Véase la documentación de librdkafka | `consistent_random` es el valor predeterminado y el mejor.  Las claves vacías y nulas se controlan idealmente en la mayoría de los casos.
`enable.idempotence` | false | | La idempotencia no se admite actualmente.
`compression.codec` | `none` || La compresión no se admite actualmente.

### <a name="consumer-configurations-only"></a>Solo configuraciones de consumidor

Propiedad | Valores recomendados | Intervalo permitido | Notas
---|---:|-----:|---
`heartbeat.interval.ms` | 3000 || 3000 es el valor predeterminado y no debe cambiarse.
`session.timeout.ms` | 30000 |6000 .. 300000| Comience con 30 000 y auméntelo si ve un reequilibrio frecuente debido a los latidos que faltan.


## <a name="further-notes"></a>Notas adicionales

Consulte la siguiente tabla de escenarios de errores comunes relacionados con la configuración.

Síntomas | Problema | Solución
----|---|-----
Errores de confirmación de desplazamiento debido al reequilibrio | El consumidor está esperando demasiado tiempo entre las llamadas a poll() y el servicio está echando al consumidor fuera del grupo. | Tiene varias opciones: <ul><li>Aumentar el tiempo de espera de la sesión.</li><li>Reducir el tamaño de lote de los mensajes para acelerar el procesamiento.</li><li>Mejorar la paralelización del procesamiento para evitar el bloqueo de consumer.poll().</li></ul> La aplicación de alguna combinación de estas tres opciones es probablemente la más acertada.
Excepciones de red con un alto rendimiento de producción | ¿Usa el cliente de Java y el valor predeterminado de max.request.size?  Es posible que las solicitudes sean demasiado grandes. | Consulte las configuraciones de Java anteriores.

## <a name="next-steps"></a>Pasos siguientes
Consulte [Suscripción de Azure y límites de servicio, cuotas y restricciones](..//azure-resource-manager/management/azure-subscription-service-limits.md) para obtener las cuotas y los límites de todos los servicios de Azure. 
