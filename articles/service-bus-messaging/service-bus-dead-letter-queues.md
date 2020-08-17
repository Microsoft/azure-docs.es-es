---
title: Colas de mensajes fallidos de Service Bus | Microsoft Docs
description: Aquí se describen las colas de mensajes fallidos de Azure Service Bus. Las colas de Service Bus y las suscripciones a temas proporcionan una subcola secundaria que se denomina cola de mensajes fallidos.
ms.topic: article
ms.date: 06/23/2020
ms.custom: fasttrack-edit
ms.openlocfilehash: 7078a7889947c4121713e9374d1487f408fed871
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86511218"
---
# <a name="overview-of-service-bus-dead-letter-queues"></a>Información general de colas de mensajes fallidos de Service Bus

Las colas de Azure Service Bus y las suscripciones a temas proporcionan una subcola secundaria, llamada *cola de mensajes fallidos* (DLQ). La cola de mensajes fallidos no se necesita crear explícitamente y no se puede eliminar ni administrar independientemente de la entidad principal.

En este artículo se describen las colas de mensajes fallidos de Service Bus. Gran parte de la descripción se muestra en el [ejemplo de colas de mensajes fallidos](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/DeadletterQueue) en GitHub.
 
## <a name="the-dead-letter-queue"></a>Cola de mensajes fallidos

La finalidad de la cola de mensajes fallidos es mantener los mensajes que no se pueden entregar a ningún destinatario o los mensajes que no se pudieron procesar. Después, los mensajes se quitan de la cola de mensajes fallidos y se inspeccionan. Una aplicación podría, con ayuda de un operador, corregir los problemas y volver a enviar el mensaje, registrar el hecho de que se produjo un error y llevar a cabo medidas correctivas. 

Desde el punto de vista de la API y el protocolo, la cola de mensajes fallidos es muy similar a cualquier otra cola, salvo que los mensajes solo se pueden enviar a través de la operación de mensajes fallidos de la entidad principal. Además, no se observa el período de vida, y no puede tratar como fallido un mensaje desde una cola de mensajes fallidos. La cola de mensajes fallidos es totalmente compatible con las operaciones transaccionales y de entrega de bloqueo de información.

No hay limpieza automática de mensajes fallidos. Los mensajes permanecen en la cola de mensajes fallidos hasta que los recupera explícitamente de dicha cola y llama a [Complete()](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync) en el mensaje fallido.

## <a name="dlq-message-count"></a>Recuento de mensajes fallidos
No es posible obtener el número de mensajes de la cola de mensajes fallidos en el nivel de tema. Esto se debe a que los mensajes no se encuentran en el nivel de tema, a menos que Service Bus produzca un error interno. En su lugar, cuando un remitente envía un mensaje a un tema, el mensaje se reenvía a las suscripciones del tema en milisegundos y, por tanto, ya no reside en el nivel de tema. Por lo tanto, puede ver los mensajes en los mensajes fallidos asociados con la suscripción del tema. En el ejemplo siguiente, **Service Bus Explorer** muestra que hay 62 mensajes actualmente en los mensajes fallidos de la suscripción "test1". 

![Recuento de mensajes fallidos](./media/service-bus-dead-letter-queues/dead-letter-queue-message-count.png)

También puede obtener el recuento de mensajes fallidos mediante el comando de la CLI de Azure: [`az servicebus topic subscription show`](/cli/azure/servicebus/topic/subscription?view=azure-cli-latest#az-servicebus-topic-subscription-show). 

## <a name="moving-messages-to-the-dlq"></a>Movimiento de mensajes a la cola de mensajes fallidos

Hay varias actividades en Service Bus que provocan que los mensajes se inserten en la cola de mensajes fallidos desde dentro del propio motor de mensajería. Una aplicación también puede mover mensajes explícitamente a la cola de mensajes fallidos. 

Como el agente mueve el mensaje, se agregan dos propiedades a dicho mensaje dado que el agente llama a su versión interna del método [DeadLetter](/dotnet/api/microsoft.azure.servicebus.queueclient.deadletterasync) en el mensaje: `DeadLetterReason` y `DeadLetterErrorDescription`.

Las aplicaciones pueden definir sus propios códigos para la propiedad `DeadLetterReason`, pero el sistema establece los valores siguientes.

| DeadLetterReason | DeadLetterErrorDescription |
| --- | --- |
|HeaderSizeExceeded |Se superó la cuota de tamaño de esta transmisión. |
|TTLExpiredException |El mensaje expiró y se consideró fallido. Consulte la sección [Se supera el valor de TimeToLive](#exceeding-timetolive) para obtener más información. |
|El identificador de sesión es null. |La entidad habilitada por sesión no permite un mensaje cuyo identificador de sesión es null. |
|MaxTransferHopCountExceeded | Número máximo de saltos permitidos al reenviar contenido entre colas. El valor se establece en 4. |
| MaxDeliveryCountExceededExceptionMessage | El mensaje no se pudo usar después de que transcurriera el número máximo de intentos de entrega. Para obtener más información, consulte la sección [Se supera el valor de MaxDeliveryCount](#exceeding-maxdeliverycount). |

## <a name="exceeding-maxdeliverycount"></a>Superación de MaxDeliveryCount

Las colas y las suscripciones tienen las propiedades [QueueDescription.MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) y [SubscriptionDescription.MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription.maxdeliverycount), respectivamente; el valor predeterminado es 10. Cuando un mensaje se entrega bajo un bloqueo ([ReceiveMode.PeekLock](/dotnet/api/microsoft.azure.servicebus.receivemode)), pero se abandona explícitamente o el bloqueo expira, la propiedad [BrokeredMessage.DeliveryCount](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) del mensaje se incrementa. Cuando [DeliveryCount](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) supera a [MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount), el mensaje se mueve a la cola de mensajes fallidos, y se especifica el código de motivo `MaxDeliveryCountExceeded`.

Este comportamiento no se puede deshabilitar, pero puede establecer [MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) en un número grande.

## <a name="exceeding-timetolive"></a>Superación de TimeToLive

Cuando las propiedades [QueueDescription.EnableDeadLetteringOnMessageExpiration](/dotnet/api/microsoft.servicebus.messaging.queuedescription) o [SubscriptionDescription.EnableDeadLetteringOnMessageExpiration](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription) están establecidas en **true** (el valor predeterminado es **false**), todos los mensajes que expiran se mueven a la cola de mensajes fallidos y se especifica el código de motivo `TTLExpiredException`.

Los mensajes que hayan expirado solo se purgan y se mueven a la cola de mensajes con problemas de entrega cuando hay, al menos, un receptor activo que extrae contenido de la cola o suscripción principal, pero los [mensajes diferidos](./message-deferral.md) no se purgan y de mueven a la cola de mensajes con problemas de entrega tras su expiración. Estos comportamientos son los predeterminados.

## <a name="errors-while-processing-subscription-rules"></a>Errores al procesar reglas de suscripción

Cuando la propiedad [SubscriptionDescription.EnableDeadLetteringOnFilterEvaluationExceptions](/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription) está habilitada en una suscripción, todos los errores que se producen mientras se ejecuta la regla de filtro SQL de una suscripción se capturan en la cola de mensajes fallidos junto con el mensaje infractor.

## <a name="application-level-dead-lettering"></a>Mensajes fallidos de nivel de aplicación

Además de las características de mensajes fallidos proporcionadas por el sistema, las aplicaciones pueden usar la cola de mensajes fallidos para rechazar explícitamente mensajes inaceptables. Esto puede incluir mensajes que no se pueden procesar correctamente debido a cualquier tipo de problema del sistema, mensajes que mantienen cargas útiles con un formato incorrecto o mensajes que no superan la autenticación cuando se utiliza algún esquema de seguridad de nivel de mensaje.

## <a name="dead-lettering-in-forwardto-or-sendvia-scenarios"></a>Colas de mensajes fallidos en escenarios ForwardTo o SendVia

Los mensajes se enviarán a la cola de mensajes fallidos de transferencia en las siguientes condiciones:

- Un mensaje pasa a través de más de cuatro colas o temas que están [encadenados](service-bus-auto-forwarding.md).
- La cola de destino o el tema se han deshabilitado o eliminado.
- El tema o la cola de destino superan el tamaño máximo de entidad.

Para recuperar estos mensajes fallidos, puede crear un receptor con el método de utilidad [FormatTransferDeadletterPath](/dotnet/api/microsoft.azure.servicebus.entitynamehelper.formattransferdeadletterpath).

## <a name="example"></a>Ejemplo

El siguiente fragmento de código crea un receptor de mensajes. En el bucle de recepción de la cola principal, el código recupera el mensaje con [Receive(TimeSpan.Zero)](/dotnet/api/microsoft.servicebus.messaging.messagereceiver), que solicita al agente que devuelva al instante cualquier mensaje disponible, o que vuelva sin ningún resultado. Si el código recibe un mensaje, inmediatamente lo abandona, lo que incrementa `DeliveryCount`. Una vez que el sistema mueve el mensaje a la cola de mensajes fallidos, la cola principal se vacía y se sale del bucle, dado que [ReceiveAsync](/dotnet/api/microsoft.servicebus.messaging.messagereceiver) devuelve **null**.

```csharp
var receiver = await receiverFactory.CreateMessageReceiverAsync(queueName, ReceiveMode.PeekLock);
while(true)
{
    var msg = await receiver.ReceiveAsync(TimeSpan.Zero);
    if (msg != null)
    {
        Console.WriteLine("Picked up message; DeliveryCount {0}", msg.DeliveryCount);
        await msg.AbandonAsync();
    }
    else
    {
        break;
    }
}
```

## <a name="path-to-the-dead-letter-queue"></a>Ruta de acceso para la cola de mensajes fallidos
Puede tener acceso a la cola de mensajes fallidos mediante la sintaxis siguiente:

```
<queue path>/$deadletterqueue
<topic path>/Subscriptions/<subscription path>/$deadletterqueue
```

Si usa el SDK de .NET, puede obtener la ruta de acceso para la cola de mensajes fallidos mediante el método SubscriptionClient.FormatDeadLetterPath(). Este método toma el nombre del tema o suscripción y le agrega el sufijo **/$DeadLetterQueue**.


## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos para más información sobre las colas de Service Bus:

* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Comparación de colas de Azure y colas de Service Bus](service-bus-azure-and-service-bus-queues-compared-contrasted.md)

