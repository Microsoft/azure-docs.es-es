---
title: Solución de problemas con Azure Event Hubs para Apache Kafka
description: En este artículo se muestra cómo solucionar problemas con Azure Event Hubs para Apache Kafka
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 034541aa6ea683c0e294ca8790b02f0dc60b5440
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090576"
---
# <a name="apache-kafka-troubleshooting-guide-for-event-hubs"></a>Guía de solución de problemas de Apache Kafka para Event Hubs
En este artículo se proporcionan sugerencias para la solución de problemas que pueden aparecer al usar Event Hubs para Apache Kafka. 

## <a name="server-busy-exception"></a>Excepción de servidor ocupado
Puede recibir una excepción de servidor ocupado que se deba a una limitación de Kafka. Con los clientes de AMQP, Event Hubs devuelve inmediatamente una excepción de **servidor ocupado** tras la limitación del servicio. Esto equivale a un mensaje de "inténtelo de nuevo más tarde". En Kafka, los mensajes se retrasan antes de que se completen. La duración del retraso se devuelve en milisegundos como `throttle_time_ms` en la respuesta de generación/captura. En la mayoría de los casos, estas solicitudes retrasadas no se registran como excepciones de servidor ocupado en los paneles de Event Hubs. En su lugar, se debe usar el valor `throttle_time_ms` de la respuesta como un indicador de que el rendimiento ha superado la cuota aprovisionada.

Si el tráfico es excesivo, el servicio tiene el siguiente comportamiento:

- Si el retraso de la solicitud de generación supera el tiempo de expiración de la solicitud, Event Hubs devuelve un código de error de **Infracción de la directiva**.
- Si el retraso de la solicitud de captura supera el tiempo de expiración de la solicitud, Event Hubs registra la solicitud como limitada y responde con un conjunto vacío de registros y sin código de error.

[Los clústeres dedicados](event-hubs-dedicated-overview.md) no tienen mecanismos de limitación. Puede consumir todos los recursos del clúster.

## <a name="no-records-received"></a>No se recibieron registros
Puede que vea que los consumidores no están obteniendo ningún registro y se están reequilibrando constantemente. En este escenario, los consumidores no obtienen ningún registro y se reequilibran constantemente. Cuando se produce no hay ninguna excepción o error, pero los registros de Kafka mostrarán que los consumidores están atascados intentando volver a unirse al grupo y asignar las particiones. Las causas principales pueden ser varias:

- Asegúrese de que `request.timeout.ms` y `session.timeout.ms` tengan al menos los valores recomendados de 60000 y 30000 respectivamente. Si estos valores son demasiado bajos podrían provocar tiempos de expiración de consumidor, que, a su vez, provocarían reequilibrios (lo que provocaría más tiempos de expiración, que provocarían más reequilibrios, y así sucesivamente) 
- Si la configuración coincide con los valores recomendados y aun así sigue viendo un reequilibrio constante, no dude en abrir una incidencia (no olvide incluir toda la configuración en la incidencia para que podamos ayudarle a depurar).

## <a name="compressionmessage-format-version-issue"></a>Problema de compresión y versión de formato de mensaje
Kafka admite la compresión, pero Event Hubs para Kafka actualmente no lo hace. Los errores que mencionan una versión de formato de mensaje (por ejemplo, `The message format version on the broker does not support the request.`) se producen cuando un cliente intenta enviar mensajes Kafka comprimidos a nuestros agentes.

Si los datos comprimidos son necesarios, una solución alternativa consiste en comprimir los datos antes de enviarlos a los agentes y descomprimirlos después de recibirlos. El cuerpo del mensaje es simplemente una matriz de bytes para el servicio, por lo que la compresión/descompresión en el lado del cliente no causará ningún problema.

## <a name="unknownserverexception"></a>UnknownServerException
Puede recibir un excepción UnknownServerException de las bibliotecas de cliente de Kafka que sea similar al ejemplo siguiente: 

```
org.apache.kafka.common.errors.UnknownServerException: The server experienced an unexpected error when processing the request
```

Registre una incidencia con el servicio de soporte técnico de Microsoft.  El registro en el nivel de depuración y las marcas de tiempo de excepción en UTC son útiles para depurar el problema. 

## <a name="other-issues"></a>Otros problemas
Si ve problemas al usar Kafka en Event Hubs, compruebe los siguientes elementos.

- **Firewall que bloquea el tráfico**: asegúrese de que el firewall no está bloqueando el puerto **9093**.
- **TopicAuthorizationException**: las causas más comunes de esta excepción son:
    - Un error de escritura en la cadena de conexión del archivo de configuración, o
    - Un intento de uso de Event Hubs para Kafka en un espacio de nombres de nivel básico. La característica Event Hubs para Kafka [solo es compatible con los espacios de nombres de niveles Estándar y Dedicado](https://azure.microsoft.com/pricing/details/event-hubs/).
- **La versión de Kafka no coincide**: Event Hubs para ecosistemas de Kafka admite las versiones 1.0 y posteriores de Kafka. Algunas aplicaciones que usan la versión 0.10 de Kafka y versiones posteriores pueden funcionar ocasionalmente gracias a la compatibilidad con versiones anteriores del protocolo de Kafka, pero se recomienda seriamente no usar versiones antiguas de la API. Las versiones 0.9 y anteriores de Kafka no admiten los protocolos SASL necesarios y no se pueden conectar a Event Hubs.
- **Codificaciones extrañas en encabezados de AMQP al consumir con Kafka**: al enviar eventos a un centro de eventos a través de AMQP, los encabezados de carga de AMQP se serializan en la codificación de AMQP. Los consumidores de Kafka no deserializan los encabezados de AMQP. Para leer los valores de encabezado, descodifique manualmente los encabezados AMQP. Como alternativa, puede evitar el uso de encabezados AMQP si sabe que va a consumir a través del protocolo Kafka. Para más información, consulte [este problema de GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/issues/56).
- **Autenticación SASL**: la obtención de un marco para cooperar con el protocolo de autenticación SASL que requiere Event Hubs puede resultar más difícil de lo que parece. Vea si puede solucionar problemas relacionados con la configuración mediante los recursos del marco en la autenticación SASL. 

## <a name="limits"></a>límites
Apache Kafka frente a Event Hubs Kafka. En su mayor parte, Event Hubs Kafka tienen los mismos valores predeterminados, propiedades, códigos de error y comportamiento general que Apache Kafka. Las instancias en las que difieren explícitamente (o donde Event Hubs impone un límite que Kafka no tiene) se enumeran a continuación:

- La longitud máxima de la propiedad `group.id` es 256 caracteres
- El tamaño máximo de `offset.metadata.max.bytes` es 1024 bytes
- Las confirmaciones de desplazamiento se limitan a 4 llamadas por segundo por partición con un tamaño de registro interno máximo de 1 MB


## <a name="next-steps"></a>Pasos siguientes
Para obtener más información acerca de Event Hubs y Event Hubs para Kafka, consulte los artículos siguientes:  

- [Guía del desarrollador de Apache Kafka para Event Hubs](apache-kafka-developer-guide.md)
- [Guía de migración de Kafka para Event Hubs](apache-kafka-migration-guide.md)
- [Preguntas más frecuentes: Event Hubs para Apache Kafka](apache-kafka-frequently-asked-questions.md)
- [Configuraciones recomendadas](apache-kafka-configurations.md)
