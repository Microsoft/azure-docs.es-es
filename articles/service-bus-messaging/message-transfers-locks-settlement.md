---
title: Transferencias de mensajes, bloqueos y liquidación de Azure Service Bus
description: En este artículo se proporciona información general sobre las transferencias de mensajes, los bloqueos y las operaciones de liquidación de Azure Service Bus.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: fd71edd12e478bcd5f14815c105c14482cf7e2bd
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89020038"
---
# <a name="message-transfers-locks-and-settlement"></a>Transferencias, bloqueos y liquidación de mensajes

La funcionalidad fundamental de un agente de mensajes, como Service Bus, es aceptar los mensajes en una cola o tema y disponer de ellos para su recuperación posterior. *Enviar* es el término que se usa normalmente para la transferencia de un mensaje al agente de mensajes. *Recibir* es el término usado habitualmente para la transferencia de un mensaje a un cliente de recuperación.

Cuando un cliente envía un mensaje, normalmente quiere saber si el mensaje se ha transferido correctamente y lo ha aceptado el agente o si se ha producido algún tipo de error. Esta confirmación positiva o negativa establece la comprensión del cliente y del agente sobre el estado de la transferencia del mensaje; por tanto, esto se conoce como *liquidación*.

Del mismo modo, cuando el agente transfiere un mensaje a un cliente, el agente y el cliente quieren comprender si el mensaje se ha procesado correctamente y, por tanto, se puede quitar, o si la entrega o el procedimiento de mensajes ha sido erróneo y, por tanto, es posible que haya que volver a enviar el mensaje.

## <a name="settling-send-operations"></a>Liquidación de las operaciones de envío

Con cualquiera de los clientes compatibles de la API de Service Bus, las operaciones de envío a Service Bus siempre se liquidan explícitamente, lo que significa que la operación de API espera que llegue el resultado de aceptación de Service Bus y después completa la operación de envío.

Si Service Bus rechaza el mensaje, el rechazo contiene un indicador de error y un texto con una propiedad "tracking-id" en el interior. El rechazo también incluye información sobre si se puede volver a intentar la operación con alguna expectativa de éxito. En el cliente, esta información se convierte en una excepción y se envía al autor de llamada de la operación de envío. Si se acepta el mensaje, la operación se completa en modo silencioso.

Cuando se usa el protocolo AMQP, que es el protocolo exclusivo para el cliente de .NET Standard y el cliente de Java y [que es una opción para el cliente de .NET Framework](service-bus-amqp-dotnet.md), las transferencias de mensajes y las liquidaciones se canalizan y se vuelven completamente asincrónicas, y se recomienda usar las variantes de la API del modelo de programación asincrónica.

El remitente puede poner varios mensajes en circulación en una sucesión rápida sin tener que esperar la confirmación de cada mensaje, tal y como se haría con el protocolo SBMP o con HTTP 1.1. Esas operaciones de envío asincrónicas finalizan cuando se aceptan y se almacenan los mensajes correspondientes, y se almacenan en entidades con particiones o cuando se solapa la operación de envío con otras entidades. También podrían producirse las finalizaciones fuera del orden de envío original.

La estrategia para administrar el resultado de las operaciones de envío puede tener un impacto inmediato y significativo en el rendimiento para la aplicación. Los ejemplos de esta sección están escritos en C# y se aplican de forma equivalente a objetos Future en Java.

Si la aplicación genera ráfagas de mensajes, que se muestran aquí con un bucle sencillo, y hubiera que esperar a que finalice cada operación de envío antes de enviar el mensaje siguiente, con API asincrónicas o sincrónicas de formas similares, el envío de 10 mensajes solo se finaliza después de 10 recorridos secuenciales de ida y vuelta completos para la liquidación.

Con una distancia de latencia asumida del recorrido de ida y vuelta de TCP de 70 milisegundos desde un sitio local a Service Bus y concediendo solo 10 ms para que Service Bus acepte y almacene cada mensaje, el siguiente bucle ocupa al menos 8 segundos, sin contar el tiempo de transferencia de carga o los efectos de una posible congestión de la ruta:

```csharp
for (int i = 0; i < 100; i++)
{
  // creating the message omitted for brevity
  await client.SendAsync(…);
}
```

Si la aplicación inicia las 10 operaciones de envío asincrónicas en sucesión inmediata y espera su finalización respectiva por separado, se superpone el recorrido de ida y vuelta de estas 10 operaciones de envío. Los 10 mensajes se transfieren en sucesión inmediata, compartiendo incluso posiblemente los marcos TCP, y la duración total de la transferencia depende en gran medida del tiempo relacionado con la red que se tarda en recibir los mensajes transferidos al agente.

Si se realizan las mismas suposiciones que para el bucle anterior, el tiempo de ejecución superpuesto total para el bucle siguiente puede permanecer menos de un segundo:

```csharp
var tasks = new List<Task>();
for (int i = 0; i < 100; i++)
{
  tasks.Add(client.SendAsync(…));
}
await Task.WhenAll(tasks);
```

Es importante tener en cuenta que todos los modelos de programación asincrónicos usan algún tipo de cola de trabajo oculta y basado en memoria que contiene las operaciones pendientes. Cuando se devuelve [SendAsync](/dotnet/api/microsoft.azure.servicebus.queueclient.sendasync#Microsoft_Azure_ServiceBus_QueueClient_SendAsync_Microsoft_Azure_ServiceBus_Message_) (C#) o **Send** (Java), la tarea de envío se pone en esa cola de trabajo pero el gesto de protocolo solo se inicia cuando la tarea vaya a ejecutarse. Para el código que suele insertar ráfagas de mensajes y en donde la confiabilidad es un problema, debe tener cuidado que no lanzar demasiados mensajes a la vez, porque todos los mensajes enviados consumen memoria hasta que realmente se hayan puesto en circulación.

Lo semáforos, tal como se muestra en el siguiente fragmento de código en C#, son objetos de sincronización que permiten dicha limitación de nivel de aplicación cuando sea necesario. Este uso de un semáforo permite al menos que 10 mensajes se envíen a la vez. Antes del envío se toma uno de los 10 bloqueos de semáforo disponibles y se libera cuando se complete el envío. El paso 11 a través del bucle espera hasta que no se envíe al menos uno de los envíos anteriores se haya completado y, después, da acceso al bloqueo:

```csharp
var semaphore = new SemaphoreSlim(10);

var tasks = new List<Task>();
for (int i = 0; i < 100; i++)
{
  await semaphore.WaitAsync();

  tasks.Add(client.SendAsync(…).ContinueWith((t)=>semaphore.Release()));
}
await Task.WhenAll(tasks);
```

Las aplicaciones **nunca** deben iniciar una operación de envío asincrónica de forma "enviar y olvidarse" sin recuperar el resultado de la operación. Si lo hace, puede cargar la cola de tareas internas y invisibles hasta el agotamiento de la memoria y evitar que la aplicación detecte errores de envío:

```csharp
for (int i = 0; i < 100; i++)
{

  client.SendAsync(message); // DON’T DO THIS
}
```

Con un cliente AMQP de bajo nivel, Service Bus también acepta transferencias "previamente liquidadas". Una transferencia previamente liquidada es una operación del tipo "enviar y olvidarse" para la cual el resultado, en cualquier caso, no se vuelve a notificar al cliente y el mensaje se considera liquidado cuando se envía. La falta de comentarios al cliente también significa que no hay ningún dato procesable disponible para el diagnóstico, lo que significa que este modo no cumple los requisitos para obtener ayuda a través del soporte técnico de Azure.

## <a name="settling-receive-operations"></a>Liquidación de las operaciones de recepción

Para las operaciones de recepción, los clientes de la API de Service Bus habilitan dos modos explícitos diferentes: *Receive-and-Delete* y *Peek-Lock*.

### <a name="receiveanddelete"></a>ReceiveAndDelete

El modo [Receive-and-Delete](/dotnet/api/microsoft.servicebus.messaging.receivemode) indica al agente que considere todos los mensajes que envía al cliente receptor como liquidados cuando se envían. Esto significa que el mensaje se considera consumido tan pronto como el agente lo pone en circulación. Si se produce un error en la transferencia del mensaje, el mensaje se pierde.

La ventaja de este modo es que el receptor no necesita realizar ninguna otra acción en el mensaje y además no se ralentizará al esperar el resultado de la liquidación. Si los datos contenidos en los mensajes individuales tienen un valor bajo o solo son significativos durante muy poco tiempo, este modo es una opción razonable.

### <a name="peeklock"></a>PeekLock

El modo [Bloque de inspección](/dotnet/api/microsoft.servicebus.messaging.receivemode) indica al agente que el cliente receptor desea liquidar explícitamente los mensajes recibidos. El mensaje ahora está disponible para que el receptor lo procese, mientras se mantiene en un bloqueo exclusivo en el servicio para que los demás receptores competidores no puedan verlo. La duración del bloqueo se define inicialmente en el nivel de cola o suscripción y la puede ampliar el cliente que posee el bloqueo, a través de la operación [RenewLock](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.renewlockasync#Microsoft_Azure_ServiceBus_Core_MessageReceiver_RenewLockAsync_System_String_).

Cuando se bloquea un mensaje, los otros clientes que reciben la misma cola o suscripción pueden encargarse de los bloqueos y recuperar los siguientes mensajes disponibles que no se encuentran en un bloqueo activo. Cuando se libera explícitamente el bloqueo en un mensaje o cuando expira el bloqueo, el mensaje vuelve a emerger en o cerca de la parte delantera del orden de recuperación para poder entregarlo de nuevo.

Cuando los receptores liberan repetidamente el mensaje o permiten el bloqueo transcurra durante un número definido de veces ([maxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount#Microsoft_ServiceBus_Messaging_QueueDescription_MaxDeliveryCount)), el mensaje se quita de la cola o suscripción automáticamente y se coloca en cola de mensajes fallidos asociada.

El cliente receptor inicia la liquidación de un mensaje recibido con una confirmación positiva cuando llama al método [Complete](/dotnet/api/microsoft.servicebus.messaging.queueclient.complete#Microsoft_ServiceBus_Messaging_QueueClient_Complete_System_Guid_) en el nivel de API. Esto indica al agente que el mensaje se ha procesado correctamente y este se quita de la cola o suscripción. El agente responde al intento de liquidación del receptor con una respuesta que indica si se pudo realizar la liquidación.

Cuando el cliente receptor no puede procesar un mensaje pero quiere que el mensaje se vuelva a entregar, puede pedir explícitamente que el mensaje se libere y se desbloquee al instante mediante una llamada al método [Abandon](/dotnet/api/microsoft.servicebus.messaging.queueclient.abandon) o bien puede no hacer nada y dejar que el bloqueo transcurra.

Si un cliente receptor no puede procesar un mensaje y sabe que volver a enviar el mensaje e reintentar la operación no va a ser la solución, puede rechazar el mensaje, que lo traslada a la cola de mensajes fallidos mediante una llamada al método [DeadLetter](/dotnet/api/microsoft.servicebus.messaging.queueclient.deadletter). Este método también permite establecer una propiedad personalizada con un código de motivo que se puede recuperar con el mensaje procedente de la cola de mensajes fallidos.

Un caso especial de liquidación es el aplazamiento, que se explica en otro artículo.

Las operaciones **Complete** o **Deadletter**, así como la operación **RenewLock** pueden producir un error debido a problemas de red, si el bloqueo mantenido ha expirado o si hay otras condiciones del servicio que impiden la liquidación. En uno de los últimos casos, el servicio envía una confirmación negativa que se muestra como una excepción en los clientes de API. Si el motivo es una conexión de red rota, se quita el bloqueo, ya que Service Bus no admite la recuperación de los vínculos AMQP existentes en una conexión diferente.

Si el método **Complete** produce un error, que surge normalmente al final del control de mensajes y, en algunos casos, después de algunos minutos de trabajo de procesamiento, la aplicación receptora puede decidir si se conserva el estado del trabajo y pasa por alto el mismo mensaje cuando se envía una segunda vez, o si elimina el resultado del trabajo y lo vuelve a intentar cuando el mensaje se reenvía.

El mecanismo típico para identificar entregas de mensajes duplicados es la comprobación del identificador de mensaje, lo que puede y debe establecerse por el remitente a un valor único, que posiblemente se alinea con un identificador de proceso de origen. Un programador de trabajos probablemente establecería el identificador del mensaje para el identificador del trabajo que está intentando volver a asignar a un trabajo con el trabajo determinado y el trabajo puede ignorar la segunda aparición de la asignación de trabajo si ese trabajo ya se ha realizado.

> [!IMPORTANT]
> Es importante tener en cuenta que el bloqueo que PeekLock adquiere en el mensaje es volátil y puede perderse en las siguientes condiciones:
>   * Actualización del servicio
>   * Actualización del sistema operativo
>   * Cambiar las propiedades de la entidad (cola, tema, suscripción) mientras se mantiene el bloqueo.
>
> Cuando se pierde el bloqueo, Azure Service Bus generará una excepción LockLostException que se mostrará en el código de la aplicación cliente. En este caso, la lógica de reintento predeterminada del cliente debe iniciarse automáticamente y vuelva a intentar la operación.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la mensajería de Service Bus, consulte los siguientes temas:

* [Colas, temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md)
* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Uso de temas y suscripciones de Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)
