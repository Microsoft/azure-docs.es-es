---
title: 'Inicio rápido: Uso de colas de Azure Service Bus con Python'
description: En este artículo se muestra cómo usar Python para crear, enviar y recibir mensajes de las colas de Azure Service Bus.
author: spelluru
documentationcenter: python
ms.devlang: python
ms.topic: quickstart
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: seo-python-october2019, devx-track-python
ms.openlocfilehash: 55ed71c6947c70ac797656f2f18cf71dd8aaae9d
ms.sourcegitcommit: 7fe8df79526a0067be4651ce6fa96fa9d4f21355
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87852487"
---
# <a name="quickstart-use-azure-service-bus-queues-with-python"></a>Inicio rápido: Uso de colas de Azure Service Bus con Python

[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]

En este artículo se muestra cómo usar Python para crear, enviar y recibir mensajes de las colas de Azure Service Bus. 

Para obtener más información sobre las bibliotecas de Azure Service Bus de Python, consulte [Bibliotecas de Service Bus para Python](/python/api/overview/azure/servicebus?view=azure-python).

## <a name="prerequisites"></a>Prerrequisitos
- Suscripción a Azure. Puede activar sus [beneficios de suscriptor de Visual Studio o MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Un espacio de nombres de Service Bus, creado siguiendo los pasos descritos en [Inicio rápido: Uso de Azure Portal para crear suscripciones y temas de Service Bus](service-bus-quickstart-topics-subscriptions-portal.md). Copie la cadena de conexión principal de la pantalla **Directivas de acceso compartido** para usarla más adelante en este artículo. 
- Python 3.4.x o superior, con el paquete de [Azure Service Bus de Python][Python Azure Service Bus package] instalado. Para obtener más información, consulte la [guía de instalación de Python](/azure/developer/python/azure-sdk-install). 

## <a name="create-a-queue"></a>Creación de una cola

Un objeto **ServiceBusClient** permite trabajar con colas. Para acceder mediante programación a Service Bus, agregue la línea siguiente cerca de la parte superior del archivo de Python:

```python
from azure.servicebus import ServiceBusClient
```

El siguiente código crea un objeto **ServiceBusClient**. Reemplace `<connectionstring>` por el valor de la cadena de conexión principal de Service Bus. Puede encontrar este valor en **Directivas de acceso compartido**, en el espacio de nombres de Service Bus de [Azure Portal][Azure portal].

```python
sb_client = ServiceBusClient.from_connection_string('<connectionstring>')
```

En el código siguiente se usa el método `create_queue` de **ServiceBusClient** para crear una cola denominada `taskqueue` con la configuración predeterminada:

```python
sb_client.create_queue("taskqueue")
```

Puede usar las opciones para invalidar la configuración predeterminada de colas, como el período de vida (TTL) del mensaje o el tamaño máximo del tema. En el código siguiente se crea una cola llamada `taskqueue` con un tamaño de cola máximo de 5 GB y un valor de TTL de 1 minuto:

```python
sb_client.create_queue("taskqueue", max_size_in_megabytes=5120,
                       default_message_time_to_live=datetime.timedelta(minutes=1))
```

## <a name="send-messages-to-a-queue"></a>mensajes a una cola

Para enviar un mensaje a una cola de Service Bus, la aplicación debe llamar al método `send` del objeto **ServiceBusService**. En el ejemplo de código siguiente se crea un cliente de cola y envía un mensaje de prueba a la cola `taskqueue`. Reemplace `<connectionstring>` por el valor de la cadena de conexión principal de Service Bus. 

```python
from azure.servicebus import QueueClient, Message

# Create the QueueClient
queue_client = QueueClient.from_connection_string("<connectionstring>", "taskqueue")

# Send a test message to the queue
msg = Message(b'Test Message')
queue_client.send(msg)
```

### <a name="message-size-limits-and-quotas"></a>Límites y cuotas de tamaño de los mensajes

El tamaño máximo de mensaje que admiten las colas de Service Bus es de 256 KB en el [nivel Estándar](service-bus-premium-messaging.md) y de 1 MB en el [nivel Premium](service-bus-premium-messaging.md). El encabezado, que incluye propiedades de la aplicación estándar y personalizadas, puede tener un tamaño máximo de 64 KB. No hay límite para el número de mensajes que contiene una cola, pero hay un tope para el tamaño total de los mensajes de la cola. El tamaño de la cola se define en el momento de la creación, con un límite de 5 GB. 

Para obtener más información sobre las cuotas, consulte [Cuotas de Service Bus][Service Bus quotas].

## <a name="receive-messages-from-a-queue"></a>mensajes de una cola

El cliente de cola recibe mensajes de una cola mediante el método `get_receiver` del objeto **ServiceBusClient**. En el ejemplo de código siguiente se crea un cliente de cola y recibe un mensaje de la cola `taskqueue`. Reemplace `<connectionstring>` por el valor de la cadena de conexión principal de Service Bus. 

```python
from azure.servicebus import QueueClient, Message

# Create the QueueClient
queue_client = QueueClient.from_connection_string("<connectionstring>", "taskqueue")

# Receive the message from the queue
with queue_client.get_receiver() as queue_receiver:
    messages = queue_receiver.fetch_next(timeout=3)
    for message in messages:
        print(message)
        message.complete()
```

### <a name="use-the-peek_lock-parameter"></a>Uso del parámetro peek_lock

El parámetro opcional `peek_lock` de `get_receiver` determina si Service Bus elimina los mensajes de la cola a medida que se leen. El modo predeterminado para la recepción de mensajes es *PeekLock*, o bien `peek_lock` establecido en **True** que lee (inspecciona) y bloquea los mensajes sin eliminarlos de la cola. Cada mensaje se debe completar explícitamente para quitarlo de la cola.

Para eliminar mensajes de la cola a medida que se leen, puede establecer el parámetro `peek_lock` de `get_receiver` en **False**. La eliminación de mensajes como parte de la operación de recepción es el modelo más sencillo y funciona bien si la aplicación puede tolerar los mensajes que faltan si se produce un error. Para entender este comportamiento, pongamos una situación en la que un consumidor emite la solicitud de recepción que se bloquea antes de procesarla. Si se eliminó el mensaje al recibirlo, cuando se reinicie la aplicación y empiece a consumir mensajes de nuevo, se omitirá el mensaje recibido antes del bloqueo.

Si la aplicación no puede tolerar mensajes perdidos, la recepción se convierte en una operación de dos fases. PeekLock busca el siguiente mensaje que se va a consumir, lo bloquea para impedir que otros consumidores lo reciban y, a continuación, lo devuelve a la aplicación. Después de procesar o almacenar el mensaje, la aplicación completa la segunda fase del proceso de recepción llamando al método `complete` del objeto **Message**.  El método `complete` marca el mensaje como consumido y lo elimina de la cola.

## <a name="handle-application-crashes-and-unreadable-messages"></a>errores de la aplicación y mensajes que no se pueden leer

Service Bus proporciona una funcionalidad que le ayuda a superar sin problemas los errores de la aplicación o las dificultades para procesar un mensaje. Si, por cualquier motivo, una aplicación de recepción no puede procesar el mensaje, entonces puede llamar al método `unlock` del objeto **Message**. Service Bus desbloquea el mensaje dentro de la cola y hace que esté disponible para recibirse de nuevo, ya sea por la misma aplicación que lo consume u otra.

También hay un tiempo de expiración para los mensajes bloqueados dentro de la cola. Si la aplicación no puede procesar un mensaje antes de que finalice el tiempo de expiración de bloqueo (por ejemplo, si se produce un error en la aplicación), entonces Service Bus desbloquea el mensaje automáticamente y hace que esté disponible para que pueda volver a recibirse.

En caso de que la aplicación sufra un error después de procesar el mensaje y antes de llamar al método `complete`, entonces el mensaje se volverá a entregar a la aplicación cuando esta se reinicie. Este comportamiento se suele denominar *Al menos un procesamiento*. Cada mensaje se procesa al menos una vez, aunque en determinadas situaciones podría volver a entregarse el mismo mensaje. Si el escenario no puede tolerar el procesamiento duplicado, puede utilizar la propiedad **MessageId** del mensaje, que permanece constante en todos los intentos de entrega, para administrar la entrega de mensajes duplicados. 

> [!TIP]
> Puede administrar los recursos de Service Bus con el [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/). El Explorador de Service Bus permite conectarse a un espacio de nombres de Service Bus y administrar fácilmente las entidades de mensajería. La herramienta dispone de características avanzadas, como la funcionalidad de importación y exportación o la capacidad de probar temas, colas, suscripciones, servicios de retransmisión, centros de notificaciones y centros de eventos.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya conoce los aspectos básicos de las colas de Service Bus, consulte [Colas, temas y suscripciones][Queues, topics, and subscriptions], donde encontrará más información.

[Azure portal]: https://portal.azure.com
[Python Azure Service Bus package]: https://pypi.python.org/pypi/azure-servicebus  
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[Service Bus quotas]: service-bus-quotas.md
