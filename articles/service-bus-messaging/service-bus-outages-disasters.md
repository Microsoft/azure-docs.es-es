---
title: Aislamiento de las aplicaciones de Azure Service Bus ante interrupciones y desastres
description: En este artículo se proporcionan técnicas para proteger las aplicaciones frente a posibles interrupciones de Azure Service Bus.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 4f3ff89e3ec59ad4445ab0b7ee7eeb45d18fa3b8
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88065631"
---
# <a name="best-practices-for-insulating-applications-against-service-bus-outages-and-disasters"></a>Procedimientos recomendados para aislar aplicaciones ante desastres e interrupciones de Service Bus

Las aplicaciones esenciales deben funcionar de manera continua, incluso si se producen interrupciones imprevistas o desastres. En este artículo se describen técnicas que puede usar para proteger las aplicaciones de Service Bus ante un posible desastre o una interrupción del servicio.

Se define la interrupción como la falta temporal de disponibilidad de Azure Service Bus. La interrupción puede afectar a algunos componentes de Service Bus, como a un almacén de mensajería, o incluso a todo el centro de datos. Una vez corregido el problema, Service Bus vuelva a estar disponible. Normalmente, una interrupción no provoca la pérdida de mensajes ni otros datos. Un ejemplo de error de componente es la falta de disponibilidad de un determinado almacén de mensajería. Un ejemplo de interrupción de todo el centro de datos es un error de alimentación del centro de datos o un conmutador de red defectuoso en él. Una interrupción puede durar desde unos pocos minutos hasta varios días.

Un desastre se define como la pérdida permanente de una unidad de escalado de Service Bus o un centro de datos. El centro de datos no volverá necesariamente a estar disponible. Normalmente, un desastre provoca la pérdida de algunos o todos los mensajes u otros datos. Algunos ejemplos de desastres son los incendios, las inundaciones o los terremotos.

## <a name="protecting-against-outages-and-disasters---service-bus-premium"></a>Protección contra interrupciones y desastres: Service Bus Premium
Los conceptos de alta disponibilidad y recuperación ante desastres se han integrado en el nivel Premium de Azure Service Bus, ambos en la misma región (mediante Availability Zones) o en regiones diferentes (mediante la recuperación ante desastres geográfica).

### <a name="geo-disaster-recovery"></a>Recuperación ante desastres geográfica

Service Bus Premium admite la recuperación ante desastres geográfica en el nivel del espacio de nombres. Para obtener más información, consulte [Recuperación ante desastres con localización geográfica de Azure Service Bus](service-bus-geo-dr.md). La característica de recuperación ante desastres, disponible solo para la [SKU premium](service-bus-premium-messaging.md), implementa la recuperación ante desastres de metadatos y depende de espacios de nombres de recuperación ante desastres principales y secundarios.

### <a name="availability-zones"></a>Zonas de disponibilidad

La SKU de Service Bus Premium es compatible con [Availability Zones](../availability-zones/az-overview.md), lo que proporciona ubicaciones con aislamiento de errores dentro de la misma región de Azure. Service Bus administra tres copias del almacén de mensajería (una principal y dos secundarias). Service Bus mantiene las tres copias sincronizadas para operaciones de datos y administración. Si se produce un error en la copia principal, una de las copias secundarias se promueve a principal sin tiempo de inactividad perceptible. Si las aplicaciones experimentan desconexiones transitorias de Service Bus, la lógica de reintento en el SDK volverá a establecer la conexión con Service Bus automáticamente. 

> [!NOTE]
> La compatibilidad de Availability Zones con Azure Service Bus Premium solo está disponible en aquellas [regiones de Azure](../availability-zones/az-region.md) en las que hay zonas de disponibilidad.

Solo puede habilitar Availability Zones en los espacios de nombres nuevos mediante Azure Portal. Service Bus no admite la migración de espacios de nombres existentes. No se puede deshabilitar la redundancia de zona después de habilitarla en el espacio de nombres.

![1][]


## <a name="protecting-against-outages-and-disasters---service-bus-standard"></a>Protección contra interrupciones y desastres: Service Bus Standard
Para lograr resistencia frente a interrupciones del centro de datos al usar el plan de tarifa de mensajería estándar, Service Bus admite dos enfoques: replicación *activa* y *pasiva*. Para cada enfoque, si una cola o un tema determinados deben permanecer accesibles en el caso de una interrupción del centro de datos, puede crear la entidad en ambos espacios de nombres. Ambas entidades pueden tener el mismo nombre. Por ejemplo, se puede acceder a una cola principal en **contosoPrimary.servicebus.windows.net/myQueue**, mientras que su equivalente secundaria resulta accesible en **contosoSecondary.servicebus.windows.net/myQueue**.

>[!NOTE]
> La configuración de **replicación activa** y **pasiva** son soluciones de uso general, no características específicas de Service Bus. La lógica de replicación (el envío a 2 espacios de nombres distintos) reside en las aplicaciones del remitente y el receptor debe tener una lógica personalizada para la detección de duplicados.

Si la aplicación no requiere comunicación permanente entre remitente y receptor, esta puede implementar una cola del lado cliente duradera para evitar la pérdida de mensajes y proteger al remitente ante los errores transitorios de Service Bus.

### <a name="active-replication"></a>Replicación activa
La replicación activa usa entidades en ambos espacios de nombres para cada operación. Cualquier cliente que envíe un mensaje envía dos copias de él. La primera copia se envía a la entidad principal (por ejemplo, **contosoPrimary.servicebus.windows.net/sales**), y la segunda copia del mensaje se envía a la entidad secundaria (por ejemplo, **contosoSecondary.servicebus.windows.net/sales**).

Un cliente recibe mensajes de ambas colas. El receptor procesa la primera copia de un mensaje y se suprime la segunda copia. Para suprimir mensajes duplicados, el remitente debe etiquetar cada mensaje con un identificador único. Ambas copias del mensaje deben estar etiquetadas con el mismo identificador. Puede usar las propiedades [BrokeredMessage.MessageId][BrokeredMessage.MessageId] o [BrokeredMessage.Label][BrokeredMessage.Label], o bien una propiedad personalizada, para etiquetar el mensaje. El receptor debe mantener una lista de los mensajes que ya haya recibido.

En el ejemplo de [replicación geográfica con el plan de tarifa estándar de Service Bus][Geo-replication with Service Bus Standard Tier] se muestra la replicación activa de entidades de mensajería.

> [!NOTE]
> La replicación activa dobla el número de operaciones; por lo tanto, este enfoque puede suponer un costo mayor.
> 
> 

### <a name="passive-replication"></a>Replicación pasiva
En el caso de que no se hayan producido errores, la replicación pasiva solo usa una de las dos entidades de mensajería. Un cliente envía el mensaje a la entidad activa. Si la operación de la entidad activa genera un código de error que indica que es posible que el centro de datos que hospeda la entidad activa no esté disponible, el cliente envía una copia del mensaje a la entidad de reserva. En ese momento, la entidad activa y la de reserva intercambian roles: el cliente remitente considera la anterior entidad activa como la nueva entidad de reserva y la anterior entidad de reserva pasa a ser la nuevo entidad activa. Si ambas operaciones de envío generan un error, no se modifican los roles de las dos entidades y se devuelve un error.

Un cliente recibe mensajes de ambas colas. Dado que es probable que el receptor reciba dos copias del mismo mensaje, este debe suprimir los mensajes duplicados. Puede suprimir duplicados de la misma manera descrita para la replicación activa.

En general, la replicación pasiva es más económica que la activa porque en la mayoría de los casos se realiza una única operación. La latencia, el rendimiento y el costo económico son idénticos para el escenario sin replicación.

Cuando se usa la replicación pasiva, en los siguientes escenarios se pueden perder mensajes o recibirse dos veces:

* **Demora o pérdida de mensajes**: se supone que el remitente envió correctamente un mensaje m1 a la cola principal y después la cola deja de estar disponible antes de que el receptor reciba m1. El remitente envía otro mensaje m2 a la cola secundaria. Si la cola principal no está disponible temporalmente, el receptor recibe m1 una vez que la cola esté disponible de nuevo. En caso de desastre, puede que el receptor no reciba nunca m1.
* **Recepción duplicada**: supongamos que el remitente envía un mensaje m a la cola principal. Service Bus procesa m correctamente , pero no envía una respuesta. Una vez que la operación de envío agota el tiempo de espera, el remitente envía una copia idéntica de m a la cola secundaria. Si el receptor puede recibir la primera copia de m antes de que la cola principal deje de estar disponible, el receptor recibe ambas copias de m aproximadamente al mismo tiempo. Si el receptor no recibe la primera copia de m antes de que la cola principal deje de estar disponible, el receptor recibe inicialmente solo la segunda copia de m pero, a continuación, recibe una segunda copia de m cuando la cola principal vuelve a estar disponible.

En el ejemplo de [replicación geográfica con el plan de tarifa estándar de Service Bus][Geo-replication with Service Bus Standard Tier] se muestra la replicación pasiva de entidades de mensajería.

## <a name="protecting-relay-endpoints-against-datacenter-outages-or-disasters"></a>Protección de los extremos de retransmisión contra desastres o interrupciones del centro de datos
La replicación geográfica de los puntos de conexión de [Azure Relay](../azure-relay/relay-what-is-it.md) permite que un servicio que expone un punto de conexión de retransmisión sea accesible en caso de interrupciones de Service Bus. Para lograr la replicación geográfica, el servicio debe crear dos extremos de retransmisión en diferentes espacios de nombres. Los espacios de nombres deben residir en distintos centros de datos y ambos extremos deben tener nombres diferentes. Por ejemplo, se puede acceder a un punto de conexión principal en **contosoPrimary.servicebus.windows.net/myPrimaryService**, mientras que su equivalente secundario resulta accesible en **contosoSecondary.servicebus.windows.net/mySecondaryService**.

A continuación, el servicio escucha en ambos extremos y un cliente puede invocar el servicio a través de cualquiera de ellos. Una aplicación cliente selecciona de forma aleatoria una de las retransmisiones como extremo principal y envía su solicitud al extremo activo. Si la operación da como resultado un código de error, este error indica que el extremo de retransmisión no está disponible. La aplicación abre un canal al extremo de reserva y vuelve a emitir la solicitud. En ese momento, el extremo activo y el de reserva intercambian roles: la aplicación cliente considera el anterior extremo activo como el nuevo extremo de reserva y el anterior extremo de reserva pasa a ser el nuevo extremo activo. Si ambas operaciones de envío generan un error, no se modifican los roles de las dos entidades y se devuelve un error.

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información acerca de la recuperación ante desastres, consulte estos artículos:

* [Recuperación ante desastres con localización geográfica de Azure Service Bus](service-bus-geo-dr.md)
* [Continuidad del negocio con Azure SQL Database][Azure SQL Database Business Continuity]
* [Diseño de aplicaciones resistentes de Azure][Azure resiliency technical guidance]

[Service Bus Authentication]: service-bus-authentication-and-authorization.md
[Partitioned messaging entities]: service-bus-partitioning.md
[Asynchronous messaging patterns and high availability]: service-bus-async-messaging.md#failure-of-service-bus-within-an-azure-datacenter
[BrokeredMessage.MessageId]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[BrokeredMessage.Label]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
[Geo-replication with Service Bus Standard Tier]: https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/GeoReplication
[Azure SQL Database Business Continuity]:../azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview.md
[Azure resiliency technical guidance]: /azure/architecture/resiliency

[1]: ./media/service-bus-outages-disasters/az.png