---
title: Creación de temas y colas con particiones de Azure Service Bus | Microsoft Docs
description: Describe cómo realizar la partición de los temas y las colas de Service Bus mediante el uso de varios agentes de mensajes.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 11cc76b0dd0125c7b54438d3f991069b7c44db59
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89007968"
---
# <a name="partitioned-queues-and-topics"></a>Temas y colas con particiones

Azure Service Bus emplea varios agentes de mensajes para procesar mensajes y varios almacenes de mensajería para almacenar mensajes. Un único agente de mensajes controla una cola o tema convencional, que se almacena en un almacén de mensajería. Las *particiones* de Service Bus permiten que las colas y temas, o *entidades de mensajería*, se dividan entre varios agentes de mensajes y almacenes de mensajería. Las particiones significan que el rendimiento general de una entidad particionada ya no está limitado por el rendimiento de un solo agente o almacén de mensajería. Además, una interrupción temporal de un almacén de mensajería no hace que una cola o tema con particiones deje de estar disponible. Las colas y los temas con particiones pueden contener todas las características avanzadas de Service Bus, como la compatibilidad con transacciones y sesiones.

> [!NOTE]
> La creación de particiones está disponible en la creación de la entidad para todas las colas y temas de las SKU básicas o estándar. No está disponible para la SKU de mensajería Premium, pero las entidades con particiones ya existentes en espacios de nombres Premium funcionarán según lo esperado.
 
No es posible cambiar la opción de particionamiento en una cola o tema existentes; solo puede establecer la opción cuando se crea la entidad.

## <a name="how-it-works"></a>Funcionamiento

Cada cola o tema con particiones consta de varias particiones. Cada partición se almacena en un almacén de mensajería diferente y la controla un agente de mensajes diferente. Cuando se envía un mensaje a una cola o un tema con particiones, Service Bus asigna el mensaje a una de las particiones. La selección se realiza de forma aleatoria mediante Service Bus o una clave de partición que el remitente puede especificar.

Cuando un cliente quiere recibir un mensaje de una cola con particiones, o de una suscripción a un tema con particiones, Service Bus consulta en todas las particiones si hay mensajes y luego devuelve al receptor el primer mensaje que se obtiene de cualquiera de los almacenes de mensajería. Service Bus almacena en caché los demás mensajes y los devuelve cuando recibe solicitudes de recepción adicionales. Un cliente de recepción no es consciente de las particiones; el comportamiento de cara al cliente de una cola o tema con particiones (por ejemplo, lectura, finalización, aplazamiento, mensajes fallidos y captura previa) es idéntico al comportamiento de una entidad regular.

La operación de inspección en una entidad sin particiones siempre devuelve el mensaje más antiguo, pero no en una entidad con particiones. En su lugar, devuelve el mensaje más antiguo en una de las particiones cuyo agente de mensajes respondió primero. No hay garantía de que el mensaje devuelto sea el más antiguo de todas las particiones. 

No hay costos adicionales cuando se envía un mensaje a una cola o tema con particiones o cuando se recibe un mensaje de ellos.

## <a name="enable-partitioning"></a>Habilitación de las particiones

Para usar colas o temas con particiones con Azure Service Bus, use la versión 2.2 o posterior de Azure SDK, o bien especifique `api-version=2013-10` o posterior en sus solicitudes HTTP.

### <a name="standard"></a>Estándar

En el nivel de mensajería Estándar, puede crear colas y temas de Service Bus en tamaños de 1, 2, 3, 4 o 5 GB (el valor predeterminado es 1 GB). Con las particiones habilitadas, Service Bus crea 16 copias (16 particiones) de la entidad, cada una del mismo tamaño especificado. Por lo tanto, si crea una cola con un tamaño de 5 GB, con 16 particiones, el tamaño de cola máximo se convierte en (5 \* 16) = 80 GB. Para ver el tamaño máximo de la cola o el tema con particiones, puede examinar su entrada en [Azure Portal][Azure portal], en la hoja **Información general** de esa entidad.

### <a name="premium"></a>Premium

En un espacio de nombres de nivel Premium no se admiten entidades de creación de particiones. Aunque puede crear colas y temas de Service Bus en tamaños de 1, 2, 3, 4, 5, 10, 20, 40 u 80 GB (el valor predeterminado es 1 GB). Para ver el tamaño de la cola o el tema, puede examinar su entrada en [Azure Portal][Azure portal], en la hoja **Información general** de esa entidad.

### <a name="create-a-partitioned-entity"></a>Creación de una entidad particionada

Hay varias maneras de crear una cola o tema con particiones. Al crear la cola o el tema desde la aplicación, puede habilitar las particiones para la cola o el tema estableciendo respectivamente las propiedades [QueueDescription.EnablePartitioning][QueueDescription.EnablePartitioning] o [TopicDescription.EnablePartitioning][TopicDescription.EnablePartitioning] en **true**. Estas propiedades deben establecerse en el momento de crear la cola o el tema y solo están disponibles en la antigua biblioteca [WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/). Como se indicó anteriormente, no es posible cambiar estas propiedades en una cola o tema existente. Por ejemplo:

```csharp
// Create partitioned topic
NamespaceManager ns = NamespaceManager.CreateFromConnectionString(myConnectionString);
TopicDescription td = new TopicDescription(TopicName);
td.EnablePartitioning = true;
ns.CreateTopic(td);
```

También puede crear una cola o un tema con particiones en [Azure Portal][Azure portal]. Al crear una cola o tema en el portal, la opción **Habilitar partición** de la cola o tema, se activa el cuadro de diálogo**Crear** de forma predeterminada. Solo puede deshabilitar esta opción en una entidad de nivel Estándar; en el nivel Premium, la creación de particiones está siempre habilitada, y la casilla no tiene ningún efecto. 

## <a name="use-of-partition-keys"></a>Uso de claves de partición

Cuando un mensaje se coloca en cola en una cola o tema con particiones, Service Bus comprueba la presencia de una clave de partición. Si encuentra una, selecciona la partición basándose en esa clave. Si no encuentra una clave de partición, selecciona la partición basándose en un algoritmo interno.

### <a name="using-a-partition-key"></a>Uso de una clave de partición

En algunos escenarios, como sesiones o transacciones, es necesario que los mensajes se almacenen en una partición específica. Todos estos escenarios requieren el uso de una clave de partición. Todos los mensajes que usan la misma clave de partición se asignan a la misma partición. Si la partición no está disponible temporalmente, Service Bus devuelve un error.

En función del escenario, se usan diferentes propiedades de mensaje como clave de partición:

**SessionId**: si un mensaje tiene establecida la propiedad [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid), Service Bus usa **SessionID** como clave de partición. De esta manera, todos los mensajes que pertenecen a la misma sesión se gestionan por el mismo agente de mensajes. Las sesiones permiten que Service Bus garantice la ordenación de los mensajes, así como la coherencia de los estados de la sesión.

**PartitionKey**: si un mensaje tiene establecida la propiedad [PartitionKey](/dotnet/api/microsoft.azure.servicebus.message.partitionkey), pero no la propiedad [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid), Service Bus usa el valor de la propiedad [PartitionKey](/dotnet/api/microsoft.azure.servicebus.message.partitionkey) como clave de partición. Si el mensaje tiene establecidas las propiedades [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid) y [PartitionKey](/dotnet/api/microsoft.azure.servicebus.message.partitionkey), ambas propiedades deben ser idénticas. Si la propiedad [PartitionKey](/dotnet/api/microsoft.azure.servicebus.message.partitionkey) se establece en un valor distinto del de la propiedad [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid), Service Bus devuelve una excepción de operación no válida. La propiedad [PartitionKey](/dotnet/api/microsoft.azure.servicebus.message.partitionkey) debe usarse si un remitente envía mensajes transaccionales que no tienen en cuenta la sesión. La clave de partición asegura que todos los mensajes que se envían dentro de una transacción se gestionan por el mismo agente de mensajes.

**MessageId**: si la cola o el tema tiene la propiedad [RequiresDuplicateDetection](/dotnet/api/microsoft.azure.management.servicebus.models.sbqueue.requiresduplicatedetection) establecida en **true** y no se han establecido las propiedades [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid) o [PartitionKey](/dotnet/api/microsoft.azure.servicebus.message.partitionkey), el valor de la propiedad [MessageId](/dotnet/api/microsoft.azure.servicebus.message.messageid) sirve de clave de partición. (Las bibliotecas de Microsoft .NET y AMQP asignan automáticamente un identificador de mensaje si la aplicación emisora no lo hace). En este caso, todas las copias del mismo mensaje se controlan por el mismo agente de mensajes. Este identificador permite que Service Bus detecte y elimine los mensajes duplicados. Si la propiedad [RequiresDuplicateDetection](/dotnet/api/microsoft.azure.management.servicebus.models.sbqueue.requiresduplicatedetection) no está establecida en **true**, Service Bus no contempla la propiedad [MessageId](/dotnet/api/microsoft.azure.servicebus.message.messageid) como clave de partición.

### <a name="not-using-a-partition-key"></a>Sin uso de una clave de partición

En ausencia de una clave de partición, Service Bus distribuye los mensajes en modo round-robin a todas las particiones de una cola o tema con particiones. Si la partición elegida no está disponible, Service Bus asigna el mensaje a una partición diferente. De esta manera, la operación de envío se realiza correctamente a pesar de la no disponibilidad temporal de un almacén de mensajería. Sin embargo, no obtendrá la ordenación garantizada que proporciona una clave de partición.

Para acceder a un análisis más detallado del equilibrio entre la disponibilidad (sin clave de partición) y la coherencia (con una clave de partición), consulte [este artículo](../event-hubs/event-hubs-availability-and-consistency.md). Esta información se aplica igualmente a entidades de Service Bus con particiones.

Para dar a Service Bus el tiempo suficiente para poner en cola el mensaje en otra partición, el valor de [OperationTimeout](/dotnet/api/microsoft.azure.servicebus.queueclient.operationtimeout) especificado por el cliente que envía el mensaje debe ser superior a quince segundos. Se recomienda establecer la propiedad [OperationTimeout](/dotnet/api/microsoft.azure.servicebus.queueclient.operationtimeout) en el valor predeterminado de 60 segundos.

Una clave de partición “ancla” un mensaje a una partición específica. Si el almacén de mensajería que contiene esta partición no está disponible, Service Bus devuelve un error. En ausencia de una clave de partición, Service Bus puede elegir una partición diferente y la operación se realiza correctamente. Por lo tanto, se recomienda no suministrar una clave de partición a menos que sea necesario.

## <a name="advanced-topics-use-transactions-with-partitioned-entities"></a>Temas avanzados: uso de transacciones con entidades con particiones

Los mensajes que se envían como parte de una transacción deben especificar una clave de partición. La clave puede ser una de las siguientes propiedades: [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid), [PartitionKey](/dotnet/api/microsoft.azure.servicebus.message.partitionkey) o [MessageId](/dotnet/api/microsoft.azure.servicebus.message.messageid). Todos los mensajes que se envían como parte de la misma transacción deben especificar la misma clave de partición. Si intenta enviar un mensaje sin una clave de partición dentro de una transacción, Service Bus devuelve una excepción de operación no válida. Si intenta enviar varios mensajes dentro de la misma transacción que tienen claves de partición diferentes, Service Bus devuelve una excepción de operación no válida. Por ejemplo:

```csharp
CommittableTransaction committableTransaction = new CommittableTransaction();
using (TransactionScope ts = new TransactionScope(committableTransaction))
{
    Message msg = new Message("This is a message");
    msg.PartitionKey = "myPartitionKey";
    messageSender.SendAsync(msg); 
    ts.CompleteAsync();
}
committableTransaction.Commit();
```

Si se establece alguna de las propiedades que sirven de clave de partición, Service Bus ancla el mensaje a una partición. Este comportamiento se produce independientemente de si se usa una transacción o no. Se recomienda que no especifique una clave de partición si no es necesario.

## <a name="using-sessions-with-partitioned-entities"></a>Uso de sesiones con entidades con particiones

Para enviar un mensaje transaccional a un tema o cola que tiene en cuenta la sesión, el mensaje debe tener establecida la propiedad [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid). Si también se especifica la propiedad [PartitionKey](/dotnet/api/microsoft.azure.servicebus.message.partitionkey), debe ser idéntica a la propiedad [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid). Si difieren, Service Bus devuelve una excepción de operación no válida.

A diferencia de las colas o temas regulares (sin particiones), no es posible usar una única transacción para enviar varios mensajes a diferentes sesiones. Si se intenta, Service Bus devuelve una excepción de operación no válida. Por ejemplo:

```csharp
CommittableTransaction committableTransaction = new CommittableTransaction();
using (TransactionScope ts = new TransactionScope(committableTransaction))
{
    Message msg = new Message("This is a message");
    msg.SessionId = "mySession";
    messageSender.SendAsync(msg); 
    ts.CompleteAsync();
}
committableTransaction.Commit();
```

## <a name="automatic-message-forwarding-with-partitioned-entities"></a>Reenvío automático de mensajes en entidades con particiones

Service Bus de Azure admite el reenvío automático de mensajes desde entidades con particiones, así como a ellas y entre ellas. Para habilitar el reenvío automático de mensajes, establezca la propiedad [QueueDescription.ForwardTo][QueueDescription.ForwardTo] en la cola o suscripción de origen. Si el mensaje especifica una clave de partición ([SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid), [PartitionKey](/dotnet/api/microsoft.azure.servicebus.message.partitionkey) o [MessageId](/dotnet/api/microsoft.azure.servicebus.message.messageid)), esa clave de partición se usa para la entidad de destino.

## <a name="considerations-and-guidelines"></a>Consideraciones e instrucciones
* **Características de alta coherencia**: si una entidad utiliza características como sesiones, detección de duplicados o control explícito de la clave de creación de particiones, las operaciones de mensajería se enrutarán siempre a particiones específicas. Si cualquiera de estas particiones experimenta un tráfico elevado o el almacén subyacente es incorrecto, se producirá un error en las operaciones y se reducirá la disponibilidad. En general, la coherencia es todavía mucho mayor que en las entidades sin particiones ya que solo un subconjunto del tráfico tiene problemas, en lugar de todo el tráfico. Para obtener más información, consulte este [análisis sobre la disponibilidad y la coherencia](../event-hubs/event-hubs-availability-and-consistency.md).
* **Administración**: las operaciones Create, Update y Delete se deben realizar en todas las particiones de la entidad. Si alguna partición es incorrecta, podría provocar errores en estas operaciones. Para la operación Get, la información, como el número de mensajes, se debe agregar desde todas las particiones. Si alguna partición es incorrecta, se notifica el estado de disponibilidad de la entidad como limitado.
* **Escenarios de bajo volumen de mensajes**: para tales escenarios, especialmente cuando se usa el protocolo HTTP, tendrá que realizar varias operaciones de recepción con el fin de obtener todos los mensajes. Para las solicitudes de recepción, el equipo front-end realiza una operación de recepción en todos las particiones y almacena en caché todas las respuestas recibidas. Una solicitud de recepción posterior en la misma conexión se beneficiaría de este almacenamiento en caché y las latencias de recepción serían menores. Sin embargo, si tiene varias conexiones o utiliza el protocolo HTTP, se establecerá una conexión nueva para cada solicitud. Por lo tanto, no hay ninguna garantía de que llegará al mismo nodo. Si todos los mensajes existentes están bloqueados y almacenados en caché en otro equipo front-end, la operación de recepción devolverá un valor **null**. Finalmente, los mensajes caducan y podrá volver a recibirlos. Se recomienda mantener la conexión HTTP. Cuando se usa la creación de particiones en escenarios de bajo volumen, las operaciones de recepción pueden tardar más de lo esperado. Por lo tanto, se recomienda no usar la creación de particiones en estos escenarios. Elimine las entidades con particiones existentes y vuelva a crearlas con la creación de particiones deshabilitada para mejorar el rendimiento.
* **Examinar o buscar mensajes**: disponible únicamente en la antigua biblioteca [WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/). [PeekBatch](/dotnet/api/microsoft.servicebus.messaging.queueclient.peekbatch) no siempre devuelve el número de mensajes especificado en la propiedad [MessageCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.messagecount). Este comportamiento tiene dos motivos habituales. Uno de ellos es que el tamaño agregado de la colección de mensajes supera el tamaño máximo de 256 KB. El otro es que si la cola o tema tiene la [propiedad EnablePartitioning](/dotnet/api/microsoft.servicebus.messaging.queuedescription.enablepartitioning) establecida en **true**, puede que una partición no tenga suficientes mensajes para completar el número solicitado de mensajes. En general, si una aplicación desea recibir un número específico de mensajes, debe llamar a [PeekBatch](/dotnet/api/microsoft.servicebus.messaging.queueclient.peekbatch) repetidamente hasta que obtenga ese número de mensajes o no habrá más mensajes para inspeccionar. Para obtener más información, incluidos algunos ejemplos de código, consulte la documentación de la API [QueueClient.PeekBatch](/dotnet/api/microsoft.servicebus.messaging.queueclient.peekbatch) o [SubscriptionClient.PeekBatch](/dotnet/api/microsoft.servicebus.messaging.subscriptionclient.peekbatch).

## <a name="latest-added-features"></a>Características agregadas más recientes

* La opción Agregar o quitar regla ya es compatible con las entidades con particiones. A diferencia de las entidades sin particiones, estas operaciones no se admiten en las transacciones. 
* Ahora se admite AMQP para enviar mensajes a una entidad con particiones o recibirlos de ella.
* Ahora se admite AMQP para las siguientes operaciones: [Batch Send](/dotnet/api/microsoft.servicebus.messaging.queueclient.sendbatch), [Batch Receive](/dotnet/api/microsoft.servicebus.messaging.queueclient.receivebatch), [Receive by Sequence Number](/dotnet/api/microsoft.servicebus.messaging.queueclient.receive), [Peek](/dotnet/api/microsoft.servicebus.messaging.queueclient.peek), [Renew Lock](/dotnet/api/microsoft.servicebus.messaging.queueclient.renewmessagelock), [Schedule Message](/dotnet/api/microsoft.azure.servicebus.queueclient.schedulemessageasync), [Cancel Scheduled Message](/dotnet/api/microsoft.azure.servicebus.queueclient.cancelscheduledmessageasync), [Add Rule](/dotnet/api/microsoft.azure.servicebus.ruledescription), [Remove Rule](/dotnet/api/microsoft.azure.servicebus.ruledescription), [Session Renew Lock](/dotnet/api/microsoft.servicebus.messaging.messagesession.renewlock), [Set Session State](/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate), [Get Session State](/dotnet/api/microsoft.servicebus.messaging.messagesession.getstate) y [Enumerate Sessions](/dotnet/api/microsoft.servicebus.messaging.queueclient.getmessagesessions).

## <a name="partitioned-entities-limitations"></a>Limitaciones de las entidades con particiones

En su implementación actual, Service Bus impone las siguientes limitaciones en colas y temas con particiones:

* Los temas y las colas con particiones no se admiten en el nivel de mensajería Premium. Se admiten las sesiones en el nivel premier mediante el uso de SessionId. 
* Las colas y los temas con particiones no admiten el envío de mensajes que pertenecen a sesiones diferentes en una sola transacción.
* Actualmente Service Bus permite hasta 100 colas o temas particionados por espacio de nombres. Cada cola o tema con particiones se tiene en cuenta para la cuota de 10.000 entidades por espacio de nombres (no se aplica al nivel Premium).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre los conceptos básicos de la especificación de mensajería de AMQP 1.0 en la [guía del protocolo AMQP 1.0](service-bus-amqp-protocol-guide.md).

[Azure portal]: https://portal.azure.com
[QueueDescription.EnablePartitioning]: /dotnet/api/microsoft.servicebus.messaging.queuedescription.enablepartitioning
[TopicDescription.EnablePartitioning]: /dotnet/api/microsoft.servicebus.messaging.topicdescription.enablepartitioning
[QueueDescription.ForwardTo]: /dotnet/api/microsoft.servicebus.messaging.queuedescription.forwardto
[AMQP 1.0 support for Service Bus partitioned queues and topics]: ./service-bus-amqp-protocol-guide.md