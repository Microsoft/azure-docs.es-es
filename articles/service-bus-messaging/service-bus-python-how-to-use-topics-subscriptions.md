---
title: 'Inicio rápido: Uso de temas y suscripciones de Azure Service Bus con Python'
description: En este artículo se muestra cómo crear un tema de Azure Service Bus, una suscripción, enviar mensajes a un tema y recibir mensajes de la suscripción.
documentationcenter: python
author: spelluru
ms.devlang: python
ms.topic: quickstart
ms.date: 06/23/2020
ms.author: spelluru
ms.custom: devx-track-python
ms.openlocfilehash: f6d1b25cb502b8cb208ba5b59c91667e03c77778
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88064390"
---
# <a name="quickstart-use-service-bus-topics-and-subscriptions-with-python"></a>Inicio rápido: Uso de temas y suscripciones de Service Bus con Python

[!INCLUDE [service-bus-selector-topics](../../includes/service-bus-selector-topics.md)]

En este artículo se describe cómo usar Python con los temas y las suscripciones de Azure Service Bus. En los ejemplos se usa el paquete del [SDK de Python de Azure][Azure Python package] para: 

- crear temas y suscripciones;
- crear reglas y filtros de suscripción;
- enviar mensajes a temas. 
- Recepción de mensajes de suscripciones
- Eliminación de temas y suscripciones

## <a name="prerequisites"></a>Prerrequisitos
- Suscripción a Azure. Puede activar sus [beneficios de suscriptor de Visual Studio o MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/?WT.mc_id=A85619ABF) o registrarse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF).
- Un espacio de nombres de Service Bus, creado siguiendo los pasos descritos en [Inicio rápido: Uso de Azure Portal para crear suscripciones y temas de Service Bus](service-bus-quickstart-topics-subscriptions-portal.md). Copie el nombre del espacio de nombres, el nombre de la clave de acceso compartido y el valor de clave principal de la pantalla **Directivas de acceso compartido** para usarlos más adelante en este inicio rápido. 
- Python 3.4.x o superior, con el paquete del [SDK de Python de Azure][Azure Python package] instalado. Para obtener más información, consulte la [guía de instalación de Python](/azure/developer/python/azure-sdk-install).

## <a name="create-a-servicebusservice-object"></a>Creación de un objeto ServiceBusService

Un objeto **ServiceBusService** permite trabajar con temas y suscripciones a temas. Para acceder mediante programación a Service Bus, agregue la línea siguiente cerca de la parte superior del archivo de Python:

```python
from azure.servicebus.control_client import ServiceBusService, Message, Topic, Rule, DEFAULT_RULE_NAME
```

Agregue el siguiente código para crear un objeto **ServiceBusService**. Reemplace `<namespace>`, `<sharedaccesskeyname>` y `<sharedaccesskeyvalue>` por el espacio de nombres de Service Bus, el nombre de clave de la firma de acceso compartido (SAS) y el valor de clave principal. Puede encontrar estos valores en **Directivas de acceso compartido**, en el espacio de nombres de Service Bus de [Azure Portal][Azure portal].

```python
bus_service = ServiceBusService(
    service_namespace='<namespace>',
    shared_access_key_name='<sharedaccesskeyname>',
    shared_access_key_value='<sharedaccesskeyvalue>')
```

## <a name="create-a-topic"></a>de un tema

En el código siguiente se usa el método `create_topic` para crear un tema de Service Bus denominado `mytopic` con la configuración predeterminada:

```python
bus_service.create_topic('mytopic')
```

Puede usar las opciones de tema para invalidar la configuración predeterminada de temas, como el período de vida (TTL) del mensaje o el tamaño máximo del tema. En el ejemplo siguiente se crea un tema denominado `mytopic` con un tamaño máximo de tema de 5 GB y un TTL de mensaje predeterminado de 1 minuto:

```python
topic_options = Topic()
topic_options.max_size_in_megabytes = '5120'
topic_options.default_message_time_to_live = 'PT1M'

bus_service.create_topic('mytopic', topic_options)
```

## <a name="create-subscriptions"></a>Creación de suscripciones

También se usa el objeto **ServiceBusService** para crear suscripciones a los temas. Una suscripción puede tener un filtro para restringir el conjunto de mensajes entregado a su cola virtual. Si no especifica un filtro, las nuevas suscripciones usan el filtro predeterminado **MatchAll**, que coloca todos los mensajes publicados en el tema en la cola virtual de la suscripción. En el ejemplo siguiente se crea una suscripción a `mytopic` llamada `AllMessages` que usa el filtro predeterminado **MatchAll**:

```python
bus_service.create_subscription('mytopic', 'AllMessages')
```

### <a name="use-filters-with-subscriptions"></a>Uso de filtros con suscripciones

Use el método `create_rule` del objeto **ServiceBusService** para filtrar los mensajes que aparecen en una suscripción. Puede especificar reglas al crear la suscripción o agregar reglas a suscripciones.

El tipo de filtro más flexible es **SqlFilter**, que utiliza un subconjunto de SQL-92. Los filtros de SQL operan en las propiedades de los mensajes que se publican en el tema. Para obtener más información sobre las expresiones que se pueden usar con un filtro de SQL, consulte la sintaxis [SqlFilter.SqlExpression][SqlFilter.SqlExpression].

Dado que el filtro predeterminado **MatchAll** se aplica automáticamente a todas las suscripciones nuevas, debe quitarlo de las suscripciones que desea filtrar; de lo contrario, **MatchAll** invalidará cualquier otro filtro que especifique. Puede eliminar la regla predeterminada utilizando el método `delete_rule` del objeto **ServiceBusService**.

En el ejemplo siguiente se crea una suscripción a `mytopic` denominada `HighMessages`, con una regla **SqlFilter** llamada `HighMessageFilter`. La regla `HighMessageFilter` selecciona solo los mensajes con una propiedad `messageposition` personalizada mayor que 3:

```python
bus_service.create_subscription('mytopic', 'HighMessages')

rule = Rule()
rule.filter_type = 'SqlFilter'
rule.filter_expression = 'messageposition > 3'

bus_service.create_rule('mytopic', 'HighMessages', 'HighMessageFilter', rule)
bus_service.delete_rule('mytopic', 'HighMessages', DEFAULT_RULE_NAME)
```

En el ejemplo siguiente se crea una suscripción a `mytopic` denominada `LowMessages`, con una regla **SqlFilter** llamada `LowMessageFilter`. La regla `LowMessageFilter` solo selecciona los mensajes con una propiedad `messageposition` menor o igual que 3:

```python
bus_service.create_subscription('mytopic', 'LowMessages')

rule = Rule()
rule.filter_type = 'SqlFilter'
rule.filter_expression = 'messageposition <= 3'

bus_service.create_rule('mytopic', 'LowMessages', 'LowMessageFilter', rule)
bus_service.delete_rule('mytopic', 'LowMessages', DEFAULT_RULE_NAME)
```

Con las reglas `AllMessages`, `HighMessages` y `LowMessages` aplicadas, los mensajes enviados a `mytopic` siempre se entregan a los destinatarios de la suscripción `AllMessages`. Los mensajes también se entregan de forma selectiva a la suscripción `HighMessages` o `LowMessages`, según el valor de la propiedad `messageposition` del mensaje. 

## <a name="send-messages-to-a-topic"></a>Envío de mensajes a un tema

Las aplicaciones usan el método `send_topic_message` del objeto **ServiceBusService** para enviar mensajes a un tema de Service Bus.

En el ejemplo siguiente se envían cinco mensajes de prueba al tema `mytopic`. El valor de la propiedad `messageposition` personalizada depende de la iteración del bucle y determina qué suscripciones reciben los mensajes. 

```python
for i in range(5):
    msg = Message('Msg {0}'.format(i).encode('utf-8'),
                  custom_properties={'messageposition': i})
    bus_service.send_topic_message('mytopic', msg)
```

### <a name="message-size-limits-and-quotas"></a>Límites y cuotas de tamaño de los mensajes

El tamaño máximo de mensaje que admiten los temas de Service Bus es de 256 KB en el [nivel Estándar](service-bus-premium-messaging.md) y de 1 MB en el [nivel Premium](service-bus-premium-messaging.md). El encabezado, que incluye propiedades de la aplicación estándar y personalizadas, puede tener un tamaño máximo de 64 KB. No hay límite para el número de mensajes que contiene un tema, pero hay un tope para el tamaño total de los mensajes de un tema. El tamaño de los temas se define en el momento de la creación (el límite máximo es de 5 GB). 

Para obtener más información sobre las cuotas, consulte [Cuotas de Service Bus][Service Bus quotas].

## <a name="receive-messages-from-a-subscription"></a>Recepción de mensajes de una suscripción

Las aplicaciones usan el método `receive_subscription_message` del objeto **ServiceBusService** para recibir mensajes de una suscripción. En el ejemplo siguiente se reciben los mensajes de la suscripción `LowMessages` y se eliminan a medida que se leen:

```python
msg = bus_service.receive_subscription_message('mytopic', 'LowMessages', peek_lock=False)
print(msg.body)
```

El parámetro opcional `peek_lock` de `receive_subscription_message` determina si Service Bus elimina los mensajes de la suscripción a medida que se leen. El modo predeterminado para la recepción de mensajes es *PeekLock*, o bien `peek_lock` establecido en **True** que lee (inspecciona) y bloquea los mensajes sin eliminarlos de la suscripción. Cada mensaje se debe completar explícitamente para quitarlo de la suscripción.

Para eliminar mensajes de la suscripción a medida que se leen, puede establecer el parámetro `peek_lock` en **False**, como en el ejemplo anterior. La eliminación de mensajes como parte de la operación de recepción es el modelo más sencillo y funciona bien si la aplicación puede tolerar los mensajes que faltan si se produce un error. Para entender este comportamiento, pongamos una situación en la que la aplicación emite la solicitud de recepción que se bloquea antes de procesarla. Si se eliminó el mensaje al recibirlo, cuando se reinicie la aplicación y empiece a consumir mensajes de nuevo, se omitirá el mensaje recibido antes del bloqueo.

Si la aplicación no puede tolerar mensajes perdidos, la recepción se convierte en una operación de dos fases. PeekLock busca el siguiente mensaje que se va a consumir, lo bloquea para impedir que otros consumidores lo reciban y, a continuación, lo devuelve a la aplicación. Después de procesar o almacenar el mensaje, la aplicación completa la segunda fase del proceso de recepción llamando al método `complete` del objeto **Message**.  El método `complete` marca el mensaje como consumido y lo elimina de la suscripción.

En el ejemplo siguiente se muestra un escenario de bloqueo de inspección:

```python
msg = bus_service.receive_subscription_message('mytopic', 'LowMessages', peek_lock=True)
if msg.body is not None:
    print(msg.body)
    msg.complete()
```

## <a name="handle-application-crashes-and-unreadable-messages"></a>errores de la aplicación y mensajes que no se pueden leer

Service Bus proporciona una funcionalidad que le ayuda a superar sin problemas los errores de la aplicación o las dificultades para procesar un mensaje. Si, por cualquier motivo, una aplicación de recepción no puede procesar el mensaje, entonces puede llamar al método `unlock` del objeto **Message**. Service Bus desbloquea el mensaje dentro de la suscripción y hace que esté disponible para recibirse de nuevo, ya sea por la misma aplicación que lo consume u otra.

También hay un tiempo de expiración para los mensajes bloqueados dentro de la suscripción. Si la aplicación no puede procesar un mensaje antes de que finalice el tiempo de expiración de bloqueo (por ejemplo, si se produce un error en la aplicación), entonces Service Bus desbloquea el mensaje automáticamente y hace que esté disponible para que pueda volver a recibirse.

En caso de que la aplicación sufra un error después de procesar el mensaje y antes de llamar al método `complete`, entonces el mensaje se volverá a entregar a la aplicación cuando esta se reinicie. Este comportamiento se suele denominar *Al menos un procesamiento*. Cada mensaje se procesa al menos una vez, aunque en determinadas situaciones podría volver a entregarse el mismo mensaje. Si el escenario no puede tolerar el procesamiento duplicado, puede utilizar la propiedad **MessageId** del mensaje, que permanece constante en todos los intentos de entrega, para administrar la entrega de mensajes duplicados. 

## <a name="delete-topics-and-subscriptions"></a>Eliminación de temas y suscripciones

Para eliminar temas y suscripciones, use [Azure Portal][Azure portal] o el método `delete_topic`. El siguiente código elimina el tema denominado `mytopic`:

```python
bus_service.delete_topic('mytopic')
```

Al eliminar un tema, se eliminan todas las suscripciones al tema. También puede eliminar las suscripciones de forma independiente. El código siguiente elimina la suscripción llamada `HighMessages` del tema `mytopic`:

```python
bus_service.delete_subscription('mytopic', 'HighMessages')
```

De forma predeterminada, los temas y las suscripciones son persistentes y existen hasta que se eliminan. Para eliminar automáticamente las suscripciones después de que transcurra un período determinado, puede establecer el parámetro [auto_delete_on_idle](/python/api/azure-mgmt-servicebus/azure.mgmt.servicebus.models.sbsubscription?view=azure-python) de la suscripción. 

> [!TIP]
> Puede administrar los recursos de Service Bus con el [Explorador de Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/). El Explorador de Service Bus permite conectarse a un espacio de nombres de Service Bus y administrar fácilmente las entidades de mensajería. La herramienta dispone de características avanzadas, como la funcionalidad de importación y exportación o la capacidad de probar temas, colas, suscripciones, servicios de retransmisión, centros de notificaciones y centros de eventos. 

## <a name="next-steps"></a>Pasos siguientes

Ahora que conoce los fundamentos de los temas de Service Bus, siga estos vínculos para obtener más información:

* [Colas, temas y suscripciones][Queues, topics, and subscriptions]
* Referencia de [SqlFilter.SqlExpression][SqlFilter.SqlExpression]

[Azure portal]: https://portal.azure.com
[Azure Python package]: https://pypi.python.org/pypi/azure
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[SqlFilter.SqlExpression]: service-bus-messaging-sql-filter.md
[Service Bus quotas]: service-bus-quotas.md