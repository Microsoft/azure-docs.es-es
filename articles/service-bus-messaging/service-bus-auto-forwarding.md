---
title: Entidades de mensajería de Azure Service Bus con reenvío automático
description: En este artículo se describe cómo encadenar una cola o suscripción de Azure Service Bus a otra cola u otro tema.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: af1c8a8e043ae964c4917a58ea67275e8379817f
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89021721"
---
# <a name="chaining-service-bus-entities-with-autoforwarding"></a>Encadenamiento de entidades de Service Bus con reenvío automático

La característica de *reenvío automático* de Service Bus permite encadenar una cola o suscripción a otra cola u otro tema que forman parte del mismo espacio de nombres. Cuando el reenvío automático está habilitado, Service Bus elimina automáticamente los mensajes que se colocan en la primera cola o suscripción (origen) y los coloca en la segunda cola o en el segundo tema (destino). Todavía se puede enviar un mensaje a la entidad de destino directamente.

## <a name="using-autoforwarding"></a>Uso de reenvío automático

Para habilitar el reenvío automático, establezca las propiedades [QueueDescription.ForwardTo][QueueDescription.ForwardTo] o [SubscriptionDescription.ForwardTo][SubscriptionDescription.ForwardTo] de los objetos [QueueDescription][QueueDescription] o [SubscriptionDescription][SubscriptionDescription] para el origen, como en este ejemplo:

```csharp
SubscriptionDescription srcSubscription = new SubscriptionDescription (srcTopic, srcSubscriptionName);
srcSubscription.ForwardTo = destTopic;
namespaceManager.CreateSubscription(srcSubscription));
```

La entidad de destino debe existir en el momento en que se creó la entidad de origen. Si la entidad de destino no existe, Service Bus devuelve una excepción cuando se le pide que cree la entidad de origen.

El reenvío automático se puede usar para escalar horizontalmente un tema individual. Service Bus limita el [número de suscripciones de un tema dado](service-bus-quotas.md) a 2000. Para alojar suscripciones adicionales, cree temas de segundo nivel. Aunque no tenga la limitación de Service Bus sobre el número de suscripciones, el hecho de agregar un segundo nivel de temas puede mejorar el rendimiento general del tema.

![Escenario de reenvío automático][0]

También puede usarlo para desacoplar los remitentes de los destinatarios. Por ejemplo, suponga que un sistema ERP consta de tres módulos: procesamiento de pedidos, administración de inventario y administración de relaciones con clientes. Cada uno de estos módulos genera mensajes que se ponen en cola en el tema correspondiente. Alice y Bob son representantes de ventas que están interesados en todos los mensajes relacionados con sus clientes. Para recibir dichos mensajes, Alice y Bob crean una cola personal y una suscripción en cada uno de los temas de ERP que reenvían automáticamente todos los mensajes a su cola.

![Escenario de reenvío automático][1]

Si Alice se va de vacaciones, se llena su cola personal, en lugar del tema de ERP. En este escenario, como un representante de ventas no ha recibido ningún mensaje, ninguno de los temas de ERP alcanza la cuota.

> [!NOTE]
> Cuando el reenvío automático está configurado, el valor de AutoDeleteOnIdle en **el origen y el destino** se establece automáticamente en el valor máximo del tipo de datos.
> 
>   - En el lado de origen, el reenvío automático actúa como una operación de recepción. Por lo tanto, el origen que tiene configurado el reenvío automático nunca está realmente "inactivo".
>   - En el lado de destino, esto se hace para garantizar que siempre haya un destino al que reenviar el mensaje.

## <a name="autoforwarding-considerations"></a>Consideraciones del reenvío automático

Si la entidad de destino acumula muchos mensajes y supera la cuota (o la entidad de destino está deshabilitada), la entidad de origen agrega los mensajes a su [cola de mensajes fallidos](service-bus-dead-letter-queues.md) hasta que haya espacio en el destino (o hasta que se vuelva a habilitar la entidad). Esos mensajes siguen estando en la cola de mensajes fallidos, por lo que debe recibirlos y procesarlos explícitamente desde dicha cola.

Al encadenar temas individuales para obtener un tema compuesto con muchas suscripciones, se recomienda tener un número moderado de suscripciones en el tema de primer nivel y muchas suscripciones en los temas de segundo nivel. Por ejemplo, un tema de primer nivel con 20 suscripciones, cada una de ellas encadenada a un tema de segundo nivel con 200 suscripciones, ofrece un mayor rendimiento que un tema de primer nivel con 200 suscripciones, cada una de ellas encadenada a un tema de segundo nivel con 20 suscripciones.

Service Bus factura una operación por cada mensaje reenviado. Por ejemplo, el envío de un mensaje a un tema con 20 suscripciones, cada una de ellas configurada para reenviar automáticamente mensajes a otra cola, o a otro tema, se factura como 21 operaciones si todas las suscripciones del primer nivel reciben una copia del mensaje.

Para crear una suscripción encadenada a otra cola o a otro tema, el creador debe tener permisos de **administración** tanto en la entidad de origen como en la de destino. Para enviar mensajes al tema de origen, solo se requieren permisos de **envío** en el tema de origen.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre el reenvío automático, vea estos temas de referencia:

* [ForwardTo][QueueDescription.ForwardTo]
* [QueueDescription][QueueDescription]
* [SubscriptionDescription][SubscriptionDescription]

Para más información sobre las mejoras de rendimiento de Service Bus, vea: 

* [Procedimientos recomendados para mejorar el rendimiento mediante la mensajería de Service Bus](service-bus-performance-improvements.md)
* [Entidades de mensajería con particiones][Partitioned messaging entities]

[QueueDescription.ForwardTo]: /dotnet/api/microsoft.servicebus.messaging.queuedescription.forwardto#Microsoft_ServiceBus_Messaging_QueueDescription_ForwardTo
[SubscriptionDescription.ForwardTo]: /dotnet/api/microsoft.servicebus.messaging.subscriptiondescription.forwardto#Microsoft_ServiceBus_Messaging_SubscriptionDescription_ForwardTo
[QueueDescription]: /dotnet/api/microsoft.servicebus.messaging.queuedescription
[SubscriptionDescription]: /dotnet/api/microsoft.servicebus.messaging.queuedescription
[0]: ./media/service-bus-auto-forwarding/IC628631.gif
[1]: ./media/service-bus-auto-forwarding/IC628632.gif
[Partitioned messaging entities]: service-bus-partitioning.md
