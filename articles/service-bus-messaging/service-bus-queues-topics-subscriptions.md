---
title: 'Mensajería de Azure Service Bus: colas, temas y suscripciones'
description: En este artículo se proporciona información general sobre las entidades de mensajería (colas, temas y suscripciones) de Azure Service Bus.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 3ee03fe5219736a1b1ca66c652fe6ac410cb40cb
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87799623"
---
# <a name="service-bus-queues-topics-and-subscriptions"></a>Colas, temas y suscripciones de Service Bus

Microsoft Azure Service Bus admite un conjunto de tecnologías de middleware basadas en la nube y orientadas a mensajes, incluidas una cola de mensajes de confianza y una mensajería de publicación/suscripción duradera. Estas funcionalidades de mensajería "asíncrona" pueden considerarse como características de mensajería desacopladas que admiten la publicación-suscripción, el desacoplamiento temporal y los escenarios de equilibrio de carga mediante la carga de trabajo de mensajería de Service Bus. La comunicación desacoplada ofrece muchas ventajas; por ejemplo, los clientes y servidores pueden conectarse según sea necesario y realizar sus operaciones de forma asincrónica.

Las entidades de mensajería que forman el núcleo de las funcionalidades de mensajería de Service Bus son las colas, los temas y suscripciones y las reglas o acciones.

## <a name="queues"></a>Colas

Las colas ofrecen una entrega de mensajes según el modelo *primero en entrar, primero en salir (FIFO [PEPS])* a uno o más destinatarios de la competencia. Es decir, los receptores normalmente reciben y procesan los mensajes en el orden en el que se agregaron a la cola y solo un destinatario de mensaje recibe y procesa cada uno de los mensajes. La principal ventaja del uso de colas es conseguir un "desacoplamiento temporal" de los componentes de la aplicación. En otras palabras, los productores (remitentes) y los consumidores (receptores) no tienen que enviar y recibir mensajes al mismo tiempo, ya que los mensajes se almacenan de forma duradera en la cola. El productor no tiene que esperar una respuesta del destinatario para continuar el proceso y el envío de más mensajes.

Una ventaja relacionada es la "nivelación de la carga", lo que permite a los productores y consumidores enviar y recibir mensajes con distintas velocidades. En muchas aplicaciones, la carga del sistema varía con el tiempo, mientras que el tiempo de procesamiento requerido por cada unidad de trabajo suele ser constante. La intermediación de productores y consumidores de mensajes con una cola implica que la aplicación consumidora solo necesita ser aprovisionada para administrar una carga promedio en lugar de una carga pico. La profundidad de la cola aumenta y se contrae a medida que varíe la carga entrante, lo que permite ahorrar dinero directamente en función de la cantidad de infraestructura requerida para dar servicio a la carga de la aplicación. A medida que aumenta la carga, se pueden agregar más procesos de trabajo para que puedan leerse desde la cola. Cada mensaje se procesa únicamente por uno de los procesos de trabajo. Es más, este equilibrio de carga basado en la extracción permite el uso óptimo de los equipos de trabajo aunque estos equipos difieran en términos de capacidad de procesamiento, ya que extraerán mensajes a su frecuencia máxima propia. Este patrón con frecuencia se denomina "patrón de consumo de competidor".

El uso de colas para intermediar entre los consumidores y productores de mensajes proporciona un acoplamiento no estricto inherente entre los componentes. Dado que los productores y consumidores no están relacionados entre sí, un consumidor puede actualizarse sin tener ningún efecto en el productor.

### <a name="create-queues"></a>Creación de colas

Puede crear colas mediante [Azure Portal](service-bus-quickstart-portal.md), [PowerShell](service-bus-quickstart-powershell.md), la [CLI](service-bus-quickstart-cli.md) o [plantillas de Resource Manager](service-bus-resource-manager-namespace-queue.md). Luego envía y recibe mensajes mediante un objeto [QueueClient](/dotnet/api/microsoft.azure.servicebus.queueclient).

Para información rápida sobre cómo crear una cola y luego enviar y recibir mensajes desde y hacia la cola, consulte los [inicios rápidos](service-bus-quickstart-portal.md) de cada método. Para un tutorial más detallado sobre cómo usar las colas, consulte [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md).

Para un ejemplo funcional, consulte el [ejemplo BasicSendReceiveUsingQueueClient](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted/Microsoft.Azure.ServiceBus/BasicSendReceiveUsingQueueClient) en GitHub.

### <a name="receive-modes"></a>Modos de recepción

Puede especificar dos nodos distintos en los que Service Bus recibe los mensajes: *ReceiveAndDelete* o *PeekLock*. Al usar el modo [ReceiveAndDelete](/dotnet/api/microsoft.azure.servicebus.receivemode), la operación de recepción consta de una sola fase; es decir, cuando Service Bus recibe la solicitud del consumidor, marca el mensaje como consumido y lo devuelve a la aplicación del consumidor. El modo **ReceiveAndDelete** es el modelo más sencillo y funciona mejor para los escenarios en los que la aplicación puede tolerar no procesar un mensaje en caso de error. Para entender este escenario, pongamos una situación en la que un consumidor emite la solicitud de recepción que se bloquea antes de procesarla. Como Service Bus marca el mensaje como consumido, cuando la aplicación se reinicie y empiece a consumir mensajes de nuevo, habrá perdido el mensaje que se consumió antes del bloqueo.

En el modo [PeekLock](/dotnet/api/microsoft.azure.servicebus.receivemode), la operación de recepción se convierte en una operación de dos fases que hace posible admitir aplicaciones que no toleran la pérdida de mensajes. Cuando Service Bus recibe una solicitud, busca el siguiente mensaje que se va a consumir, lo bloquea para impedir que otros consumidores lo reciban y, a continuación, lo devuelve a la aplicación. Una vez que la aplicación termina de procesar el mensaje (o lo almacena de forma fiable para su futuro procesamiento), completa la segunda fase del proceso de recepción creando la llamada [CompleteAsync](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync) en el mensaje recibido. Cuando Service Bus ve la llamada **CompleteAsync**, marca el mensaje como consumido.

Si por cualquier motivo la aplicación no puede procesar el mensaje, realice la llamada al método [AbandonAsync](/dotnet/api/microsoft.azure.servicebus.queueclient.abandonasync) del mensaje recibido (en lugar de [CompleteAsync](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync)). Este método permite que Service Bus desbloquee el mensaje y esté disponible para que el mismo consumidor u otro vuelvan a recibirlo. En segundo lugar, hay un tiempo de espera asociado con el bloqueo y, si la aplicación no puede procesar el mensaje antes de que el tiempo de espera del bloqueo expire (por ejemplo, si la aplicación se bloquea), Service Bus desbloquea el mensaje y hace que esté disponible para recibirse de nuevo (básicamente realizando una operación [AbandonAsync](/dotnet/api/microsoft.azure.servicebus.queueclient.abandonasync) de forma predeterminada).

En caso de que la aplicación se bloquee después de procesar el mensaje, pero antes de que se emita la solicitud **CompleteAsync**, se volverá a enviar el mensaje a la aplicación cuando esta se reinicie. Este proceso se conoce como procesamiento *Al menos una vez*; es decir, cada mensaje se procesa como mínimo una vez. Sin embargo, en determinadas situaciones, es posible que se vuelva a entregar el mismo mensaje. Si el escenario no puede tolerar el procesamiento duplicado, se requiere una lógica adicional en la aplicación para detectar duplicados que se puedan conseguir de acuerdo con la propiedad [MessageId](/dotnet/api/microsoft.azure.servicebus.message.messageid) del mensaje, que permanece constante en los intentos de entrega. Esta característica se conoce como procesamiento *Exactamente una vez*.

## <a name="topics-and-subscriptions"></a>Temas y suscripciones

A diferencia de las colas, en las que un solo consumidor procesa cada mensaje, los *temas* y las *suscripciones* proporcionan una forma de comunicación de uno a varios mediante un patrón de *publicación o suscripción*. Cada mensaje publicado se pone a disposición de todas las suscripciones registradas en el tema, lo que es útil para escalar a un gran número de destinatarios. Los mensajes se envían a un tema y se entregan a una o varias suscripciones asociadas, según las reglas de filtro que se pueden establecer por suscripción. Las suscripciones pueden usar filtros adicionales para restringir los mensajes que desean recibir. Los mensajes se envían a un tema de la misma manera que se envían a una cola, pero no se reciben del tema directamente. En su lugar, se reciben de las suscripciones. Una suscripción al tema se parece a una cola virtual en que recibe copias de los mensajes que se envían al tema. Los mensajes se reciben de una suscripción exactamente de la misma forma en que se reciben de una cola.

Por medio de la comparación, la funcionalidad de envío de mensajes de una cola los asigna directamente a un tema y su funcionalidad de recepción de mensajes a una suscripción. Entre otras cosas, esta característica significa que las suscripciones admiten los mismos patrones que se han descrito en esta misma sección con respecto a las colas: consumidor en competencia, desacoplamiento temporal, nivelación de carga y equilibrio de la carga.

### <a name="create-topics-and-subscriptions"></a>Creación de temas y suscripciones

La creación de un tema es similar a la creación de una cola, como se describe en la sección anterior. Después puede enviar mensajes mediante la clase [TopicClient](/dotnet/api/microsoft.azure.servicebus.topicclient). Para recibir mensajes, cree una o varias suscripciones al tema. Al igual que las colas, los mensajes se reciben de una suscripción mediante un objeto [SubscriptionClient](/dotnet/api/microsoft.azure.servicebus.subscriptionclient), en lugar de un objeto [QueueClient](/dotnet/api/microsoft.azure.servicebus.queueclient). Cree el cliente de la suscripción y pase el nombre del tema, el nombre de la suscripción y (opcionalmente) el modo de recepción como parámetros.

Si desea ver un ejemplo funcional completo, consulte el [ejemplo BasicSendReceiveUsingTopicSubscriptionClient](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted/Microsoft.Azure.ServiceBus/BasicSendReceiveUsingTopicSubscriptionClient) en GitHub.

### <a name="rules-and-actions"></a>Reglas y acciones

En muchos escenarios, los mensajes que tienen características específicas deben procesarse de maneras diferentes. Para permitir este procesamiento, puede configurar suscripciones para buscar los mensajes que tengan las propiedades deseadas y, después, realizar determinadas modificaciones en dichas propiedades. Mientras que las suscripciones del Service Bus ven todos los mensajes enviados al tema, solo se puede copiar un subconjunto de dichos mensajes en la cola de suscripción virtual. Este filtrado se consigue mediante los filtros de suscripción. Dichas modificaciones se denominan *acciones de filtrado*. Al crear una suscripción, se puede incluir una expresión de filtro que opere en las propiedades del mensaje, tanto en las propiedades del sistema (por ejemplo, **Label**) como en las propiedades de la aplicación personalizada (por ejemplo, **StoreName**). La expresión de filtro SQL es opcional en este caso; sin una expresión de filtro SQL, todas las acciones de filtro definidas en una suscripción se realizarán en todos los mensajes de dicha suscripción.

Si desea ver un ejemplo funcional completo, consulte el [ejemplo TopicSubscriptionWithRuleOperationsSample](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted/Microsoft.Azure.ServiceBus/TopicSubscriptionWithRuleOperationsSample) en GitHub.

Para más información sobre los valores de filtro posibles, vea la documentación de las clases [SqlFilter](/dotnet/api/microsoft.azure.servicebus.sqlfilter) y [SqlRuleAction](/dotnet/api/microsoft.azure.servicebus.sqlruleaction).

## <a name="java-message-service-jms-20-entities-preview"></a>Entidades de Java Message Service (JMS) 2.0 (versión preliminar)

Se puede acceder a las entidades siguientes a través de la API de Java Message Service (JMS) 2.0.

  * Colas temporales
  * Temas temporales
  * Suscripciones duraderas compartidas
  * Suscripciones duraderas no compartidas
  * Suscripciones no duraderas compartidas
  * Suscripciones no duraderas no compartidas

Obtenga más información sobre las [entidades de JMS 2.0](java-message-service-20-entities.md) y sobre cómo [usarlas](how-to-use-java-message-service-20.md).

## <a name="next-steps"></a>Pasos siguientes

Para más información y ejemplos de cómo usar la mensajería de Service Bus, consulte estos temas avanzados:

* [Introducción a la mensajería de Service Bus](service-bus-messaging-overview.md)
* [Inicio rápido: Envío y recepción de mensajes mediante Azure Portal y .NET](service-bus-quickstart-portal.md)
* [Tutorial: Actualización del inventario mediante Azure Portal y temas y suscripciones](service-bus-tutorial-topics-subscriptions-portal.md)


