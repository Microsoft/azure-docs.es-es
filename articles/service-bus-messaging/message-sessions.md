---
title: Sesiones de mensajes de Azure Service Bus | Microsoft Docs
description: En este artículo se explica cómo usar sesiones de para habilitar la administración ordenada y conjunta de secuencias sin enlace de mensajes relacionados.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 05efc550e119186a2925c13d3fcfed11bec17251
ms.sourcegitcommit: 3543d3b4f6c6f496d22ea5f97d8cd2700ac9a481
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/20/2020
ms.locfileid: "86511303"
---
# <a name="message-sessions"></a>Sesiones de mensajes
Las sesiones de Microsoft Azure Service Bus permiten la administración ordenada y conjunta de secuencias sin enlace de mensajes relacionados. Se pueden usar sesiones en patrones **FIFO (primero en entrar, primero en salir)** y de **solicitud-respuesta**. En este artículo se muestra cómo usar sesiones para implementar estos patrones al utilizar Service Bus. 

> [!NOTE]
> El nivel Básico de Service Bus no admite sesiones. Los niveles Estándar y Premium admiten sesiones. Para conocer las diferencias entre estos niveles, consulte [Precios de Service Bus](https://azure.microsoft.com/pricing/details/service-bus/).

## <a name="first-in-first-out-fifo-pattern"></a>Patrón FIFO (primero en entrar, primero en salir)
Para realizar una garantía FIFO en Service Bus, use sesiones. Service Bus no prescribe con respecto a la naturaleza de la relación entre los mensajes, y tampoco define un modelo en particular para determinar dónde comienza o termina una secuencia de mensajes.

Cualquier remitente puede crear una sesión al enviar mensajes en un tema o una cola si se establece la propiedad [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid#Microsoft_Azure_ServiceBus_Message_SessionId) en algún identificador definido por la aplicación que sea único para la sesión. En el nivel de protocolo AMQP 1.0, este valor se asigna a la propiedad *group-id*.

En las colas o suscripciones basadas en sesiones, las sesiones existen cuando hay al menos un mensaje con el valor de [SessionId](/dotnet/api/microsoft.azure.servicebus.message.sessionid#Microsoft_Azure_ServiceBus_Message_SessionId) de la sesión. Una vez que una sesión existe, no hay ningún tiempo o API definidos para cuando la sesión expira o desaparece. En teoría, se puede recibir un mensaje para una sesión hoy, el mensaje siguiente en un momento del año y, si el valor de **SessionId** coincide, la sesión es la misma desde la perspectiva de Service Bus.

Por lo general, sin embargo, una aplicación tiene una noción clara de dónde comienza o termina un conjunto de mensajes relacionados. Service Bus no establece ninguna regla específica.

Un ejemplo de cómo delinear una secuencia para transferir un archivo es establecer la propiedad **Label** del primer mensaje en **start**, de los mensajes intermedios en **content** y del último mensaje en **end**. La posición relativa de los mensajes de contenido se puede calcular como la diferencia de *SequenceNumber* del mensaje actual a partir del valor **SequenceNumber** del mensaje de *inicio*.

La característica de sesión de Service Bus permite una operación de recepción específica, en forma de [MessageSession](/dotnet/api/microsoft.servicebus.messaging.messagesession) en las API de Java y C#. Para habilitar la característica, se establece la propiedad [requiresSession](/azure/templates/microsoft.servicebus/namespaces/queues#property-values) de la cola o la suscripción mediante Azure Resource Manager o la marca del portal. Esto es necesario antes de intentar usar las operaciones de API relacionadas.

En el portal, establezca la marca con la casilla siguiente:

![Captura de pantalla del cuadro de diálogo Crear cola con la opción Habilitar sesiones seleccionada y resaltada en rojo.][2]

> [!NOTE]
> Cuando las sesiones están habilitadas en una cola o una suscripción, las aplicaciones cliente ***ya no*** pueden enviar ni recibir mensajes normales. Todos los mensajes se deben enviar como parte de una sesión (estableciendo el identificador de sesión) y recibir mediante la recepción de la sesión.

Las API de las sesiones existen en los clientes de colas y suscripciones. Hay un modelo imperativo que controla cuándo se reciben mensajes y sesiones, y un modelo basado en controlador, similar a *OnMessage*, que oculta la complejidad de la administración del bucle de recepción.

### <a name="session-features"></a>Características de las sesiones

Las sesiones proporcionan la demultiplexación simultánea de flujos de mensajes intercalados, al tiempo que se conserva y garantiza la entrega ordenada.

![Un diagrama que muestra la forma en que la característica Sesiones mantiene la entrega ordenada.][1]

El cliente que acepta la sesión crea un receptor [MessageSession](/dotnet/api/microsoft.servicebus.messaging.messagesession). El cliente llama a [QueueClient.AcceptMessageSession](/dotnet/api/microsoft.servicebus.messaging.queueclient.acceptmessagesession#Microsoft_ServiceBus_Messaging_QueueClient_AcceptMessageSession) o [QueueClient.AcceptMessageSessionAsync](/dotnet/api/microsoft.servicebus.messaging.queueclient.acceptmessagesessionasync#Microsoft_ServiceBus_Messaging_QueueClient_AcceptMessageSessionAsync) en C#. En el modelo de devolución de llamada reactivo, se registra un controlador de sesión.

Cuando el objeto [MessageSession](/dotnet/api/microsoft.servicebus.messaging.messagesession) se acepta y mientras está contenido en el cliente, ese cliente mantiene un bloqueo exclusivo sobre todos los mensajes con ese valor de [SessionId](/dotnet/api/microsoft.servicebus.messaging.messagesession.sessionid#Microsoft_ServiceBus_Messaging_MessageSession_SessionId) de la sesión que existe en la cola o la suscripción y también sobre todos los mensajes con ese valor de **SessionId** que sigan llegando mientras tiene lugar la sesión.

El bloqueo se libera cuando se llama a **Close** o **CloseAsync**, o cuando el bloqueo expira en caso de que la aplicación sea incapaz de llevar a cabo la operación de cierre. El bloqueo de sesión debe tratarse como un bloqueo exclusivo en un archivo, lo que significa que la aplicación debería cerrar la sesión en cuanto ya no lo necesite o no espere ningún otro mensaje.

Cuando varios receptores simultáneos se extraen de la cola, los mensajes que pertenecen a una sesión determinada se envían al receptor específico que actualmente mantiene el bloqueo en esa sesión. Con esa operación, una secuencia de mensajes intercalados en una cola o suscripción se demultiplexa limpiamente en receptores diferentes, y esos receptores también pueden encontrarse en distintas máquinas cliente, puesto que la administración del bloqueo tiene lugar en el servidor, dentro de Service Bus.

La ilustración anterior muestra tres receptores de sesiones simultáneas. Una sesión con `SessionId` = 4 no tiene ningún cliente activo, propietario, lo que significa que no se entregan mensajes de esta sesión concreta. Una sesión actúa de muchas maneras, como de subcola.

El bloqueo de sesión mantenido por el receptor de sesión sirve de protección para los bloqueos de mensaje usados en el modo de usado en el modo de liquidación *bloqueo de inspección*. No puede haber más de un receptor que tenga un bloqueo en una sesión. Un receptor puede tener muchos mensajes en curso y los recibirá en orden. Abandonar un mensaje hace que el mismo mensaje se sirva de nuevo en la siguiente operación de recepción.

### <a name="message-session-state"></a>Estado de la sesión de mensaje

Cuando se procesan flujos de trabajo en sistemas de nube de gran escala y alta disponibilidad, el controlador del flujo de trabajo asociado con una sesión determinada debe ser capaz de recuperarse de errores inesperados y también de reanudar el trabajo finalizado parcialmente en un proceso o una máquina diferentes de aquellos donde comienza el trabajo.

El servicio de estado de sesión permite una anotación definida por la aplicación de una sesión de mensajes en el agente, de modo que el estado de procesamiento registrado con respecto a esa sesión deja de estar disponible al instante cuando un nuevo procesador adquiere la sesión.

Desde el punto de vista de Service Bus, el estado de la sesión de mensajes es un objeto binario opaco que puede contener datos del tamaño de un mensaje, que es de 256 KB para Service Bus Standard y 1 MB para Service Bus Premium. El estado de procesamiento relativo a una sesión puede estar contenido en el estado de sesión, o el estado de sesión puede apuntar a alguna ubicación de almacenamiento o registro de base de datos que contiene dicha información.

Las API para administrar el estado de sesión, [SetState](/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate#Microsoft_ServiceBus_Messaging_MessageSession_SetState_System_IO_Stream_) y [GetState](/dotnet/api/microsoft.servicebus.messaging.messagesession.getstate#Microsoft_ServiceBus_Messaging_MessageSession_GetState), se encuentran en el objeto [MessageSession](/dotnet/api/microsoft.servicebus.messaging.messagesession) en las API de C# y Java. Una sesión que no tenía anteriormente un estado de sesión definido devuelve una referencia **null** para **GetState**. Para borrar el estado de sesión establecido previamente se usa [SetState(null)](/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate#Microsoft_ServiceBus_Messaging_MessageSession_SetState_System_IO_Stream_).

El estado de sesión permanecerá siempre y cuando no se borre (se devuelva **null**), incluso si se consumen todos los mensajes de una sesión.

Todas las sesiones existentes de una cola o suscripción se pueden enumerar con el método **SessionBrowser** de la API de Java y con [GetMessageSessions](/dotnet/api/microsoft.servicebus.messaging.queueclient.getmessagesessions#Microsoft_ServiceBus_Messaging_QueueClient_GetMessageSessions) en [QueueClient](/dotnet/api/microsoft.servicebus.messaging.queueclient) y [SubscriptionClient](/dotnet/api/microsoft.servicebus.messaging.subscriptionclient) en el cliente de .NET Framework.

El estado de sesión mantenido en una cola o en que una suscripción se tiene en cuenta en la cuota de almacenamiento de esa entidad. Cuando la aplicación se termina con una sesión, se recomienda por lo tanto que dicha aplicación limpie su estado retenido para evitar costos de administración externos.

### <a name="impact-of-delivery-count"></a>Efecto del recuento de entregas

La definición de recuento de entregas por mensaje en el contexto de las sesiones varía ligeramente de la definición cuando no hay sesiones. En esta tabla se resume cuándo se incrementa el recuento de entregas.

| Escenario | Se incrementa el recuento de entregas del mensaje |
|----------|---------------------------------------------|
| Se acepta la sesión, pero el bloqueo de la sesión expira (debido al tiempo de espera) | Sí |
| Se acepta la sesión, los mensajes de la sesión no se completan (aunque estén bloqueados) y la sesión se cierra. | No |
| Se acepta la sesión, se completan los mensajes y, luego, la sesión se cierra explícitamente. | N/A (es el flujo estándar. Aquí se quitan los mensajes de la sesión) |

## <a name="request-response-pattern"></a>Patrón de solicitud-respuesta
El [patrón de solicitud-respuesta](https://www.enterpriseintegrationpatterns.com/patterns/messaging/RequestReply.html) es un modelo de integración bien establecido que permite a la aplicación remitente enviar una solicitud y proporciona una manera de que el receptor envíe correctamente una respuesta a la aplicación remitente. Normalmente, este patrón necesita una cola o un tema de corta duración a los que la aplicación envíe respuestas. En este escenario, las sesiones proporcionan una solución alternativa sencilla con semántica comparable. 

Varias aplicaciones pueden enviar sus solicitudes a una única cola de solicitudes, con un parámetro de encabezado específico establecido para identificar de forma única a la aplicación remitente. La aplicación receptora puede procesar las solicitudes que entran en la cola y enviar respuestas en una cola habilitada para sesiones, de forma que el identificador de sesión se establezca en el identificador único que el remitente había enviado en el mensaje de solicitud. La aplicación que envió la solicitud puede recibir luego mensajes en un identificador de sesión específico y procesar correctamente las respuestas.

> [!NOTE]
> La aplicación que envía las solicitudes iniciales debe conocer el identificador de sesión y usar `SessionClient.AcceptMessageSession(SessionID)` para bloquear la sesión en la que se espera la respuesta. Se recomienda usar un GUID que identifique de forma única la instancia de la aplicación como un identificador de sesión. No debe haber ningún controlador de sesión ni `AcceptMessageSession(timeout)` en la cola para garantizar que las respuestas estén disponibles para que las bloqueen y procesen destinatarios específicos.

## <a name="next-steps"></a>Pasos siguientes

- Consulte los [ejemplos de Microsoft.Azure.ServiceBus](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.Azure.ServiceBus/Sessions) o los [ejemplos de Microsoft.ServiceBus.Messaging](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/Sessions) para obtener un ejemplo que use el cliente de .NET Framework para controlar los mensajes que tienen en cuenta la sesión. 

Para más información sobre la mensajería de Service Bus, consulte los siguientes temas:

* [Colas, temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md)
* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Uso de temas y suscripciones de Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)

[1]: ./media/message-sessions/sessions.png
[2]: ./media/message-sessions/queue-sessions.png
