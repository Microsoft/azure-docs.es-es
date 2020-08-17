---
title: 'Azure Service Bus: aplazamiento de mensajes'
description: En este artículo se explica cómo diferir la entrega de mensajes de Azure Service Bus. El mensaje permanece en la cola o suscripción, pero se mantiene separado.
ms.topic: article
ms.date: 06/23/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: 7c9ec55de24c97df3530d80deef55ed87be84077
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86511286"
---
# <a name="message-deferral"></a>Aplazamiento de mensajes

Cuando un cliente de cola o suscripción recibe un mensaje que se desea procesar, pero cuyo procesamiento no es posible en ese momento debido a circunstancias especiales dentro de la aplicación, tiene la opción de "aplazar" la recuperación del mensaje para un momento posterior. El mensaje permanece en la cola o suscripción, pero se mantiene separado.

El aplazamiento es una característica creada específicamente para escenarios de procesamiento de flujo de trabajo. Los marcos de flujo de trabajo pueden requerir que se procesen ciertas operaciones en un orden concreto y puede que tenga que posponer el procesamiento de algunos mensajes recibidos hasta que se haya completado el trabajo anterior prescrito notificado por otros mensajes.

Un ejemplo sencillo es una secuencia de procesamiento de pedidos en la que una notificación de pago de un proveedor de pago externo aparece en un sistema antes de haberse propagado el pedido de compra correspondiente desde el escaparate al sistema de suministro. En ese caso, el sistema de suministro puede aplazar el procesamiento de la notificación de pago hasta que haya un pedido con el que se vaya a asociar. En escenarios de encuentro, donde los mensajes de orígenes diferentes conducen a un desvío de flujo de trabajo, el orden de ejecución en tiempo real puede, de hecho, ser correcto pero es posible que los mensajes reflejen que los resultados llegan sin orden.

En última instancia, el aplazamiento contribuye a volver a ordenar los mensajes de su orden de llegada a un orden en el que se pueden procesar, lo que permite dejar los mensajes de forma segura en el almacén de mensajes cuyo procesamiento tiene que posponerse.

> [!NOTE]
> Los mensajes diferidos no se moverán de forma automática a la cola de mensajes fallidos [tras expirar](./service-bus-dead-letter-queues.md#exceeding-timetolive). Este comportamiento es así por diseño.

## <a name="message-deferral-apis"></a>API de aplazamiento de mensajes

La API es [BrokeredMessage.Defer](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.defer?view=azureservicebus-4.1.1#Microsoft_ServiceBus_Messaging_BrokeredMessage_Defer) o [BrokeredMessage.DeferAsync](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.deferasync?view=azureservicebus-4.1.1#Microsoft_ServiceBus_Messaging_BrokeredMessage_DeferAsync) en el cliente de .NET Framework, [MessageReceiver.DeferAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.deferasync) en el cliente de .NET Standard y [IMessageReceiver.defer](/java/api/com.microsoft.azure.servicebus.imessagereceiver.defer?view=azure-java-stable) o [IMessageReceiver.deferAsync](/java/api/com.microsoft.azure.servicebus.imessagereceiver.deferasync?view=azure-java-stable) en el cliente de Java. 

Los mensajes aplazados permanecen en la cola principal junto con todos los demás mensajes activos (a diferencia de los mensajes fallidos que están activos en una subcola), pero que ya no se reciben mediante las funciones Receive/ReceiveAsync normales. Los mensajes aplazados se pueden detectar a través de la [exploración de mensajes](message-browsing.md) si una aplicación pierde su pista.

Para recuperar un mensaje aplazado, su propietario es responsable de recordar la propiedad [SequenceNumber](/dotnet/api/microsoft.azure.servicebus.message.systempropertiescollection.sequencenumber#Microsoft_Azure_ServiceBus_Message_SystemPropertiesCollection_SequenceNumber) cuando se aplaza. El receptor que conoce el número de secuencia de un mensaje aplazado puede recibir después el mensaje explícitamente con `Receive(sequenceNumber)`.

Si no se puede procesar un mensaje porque un recurso concreto para controlar ese mensaje no está disponible temporalmente, pero el procesamiento de mensajes no debe suspenderse sumariamente, una forma de apartar ese mensaje durante unos minutos es recordar la propiedad **SequenceNumber** de un [mensaje programado](message-sequencing.md) que se vaya a publicar en unos minutos y volver a recuperar el mensaje aplazado cuando llega el mensaje programado. Tenga en cuenta que, si un controlador de mensajes depende de una base de datos para todas las operaciones y esa base de datos no está disponible temporalmente, esta no debe usar el aplazamiento, sino que, en su lugar, debe suspender la recepción de mensajes por completo hasta que la base de datos vuelva a estar disponible.


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la mensajería de Service Bus, consulte los siguientes temas:

* [Colas, temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md)
* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Uso de temas y suscripciones de Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)
