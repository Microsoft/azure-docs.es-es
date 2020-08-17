---
title: 'Mensajería de Azure Service Bus: entidades de Java Message Service (versión preliminar)'
description: En este artículo se proporciona información general sobre las entidades de mensajería de Azure Service Bus accesibles a través de la API de Java Message Service.
ms.topic: article
ms.date: 07/20/2020
ms.openlocfilehash: 1a7fe3d6355146ccf0fce50266a6f3b8da5231b3
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87801330"
---
# <a name="java-message-service-jms-20-entities-preview"></a>Entidades de Java Message Service (JMS) 2.0 (versión preliminar)

Las aplicaciones cliente que se conectan a Azure Service Bus Premium y usan la [biblioteca JMS de Azure Service Bus](https://search.maven.org/artifact/com.microsoft.azure/azure-servicebus-jms) pueden aprovechar las siguientes entidades.

## <a name="queues"></a>Colas

Las colas de JMS se pueden comparar semánticamente con las [colas de Service Bus](service-bus-queues-topics-subscriptions.md#queues) tradicionales.

Para crear una cola, use los métodos siguientes de la clase `JMSContext`:

```java
Queue createQueue(String queueName)
```

## <a name="topics"></a>Temas

Los temas de JMS se pueden comparar semánticamente con los [temas de Service Bus](service-bus-queues-topics-subscriptions.md#topics-and-subscriptions) tradicionales.

Para crear un tema, use los métodos siguientes de la clase `JMSContext`:

```java
Topic createTopic(String topicName)
```

## <a name="temporary-queues"></a>Colas temporales

Cuando una aplicación cliente requiere una entidad temporal que existe durante la vigencia de la aplicación, puede usar colas temporales. Estas se utilizan en el patrón de [solicitud-respuesta](https://www.enterpriseintegrationpatterns.com/patterns/messaging/RequestReply.html).

Para crear una cola temporal, use los métodos siguientes de la clase `JMSContext`:

```java
TemporaryQueue createTemporaryQueue()
```

## <a name="temporary-topics"></a>Temas temporales

Al igual que las colas temporales, los temas temporales existen para habilitar la publicación o suscripción a través de una entidad temporal que existe durante la vigencia de la aplicación.

Para crear un tema temporal, use los métodos siguientes de la clase `JMSContext`:

```java
TemporaryTopic createTemporaryTopic()
```

## <a name="java-message-service-jms-subscriptions"></a>Suscripciones de Java Message Service (JMS)

Aunque estas son semánticamente similares a las [suscripciones](service-bus-queues-topics-subscriptions.md#topics-and-subscriptions) (es decir, existen en un tema y habilitan la semántica de publicación o suscripción), la especificación de Java Message Service presenta los conceptos de los atributos **Compartido**, **No compartido**, **Duradero** y **No duradero** para una suscripción determinada.

> [!NOTE]
> Las suscripciones siguientes están disponibles en el nivel Premium de Azure Service Bus para la versión preliminar de las aplicaciones cliente que se conectan a Azure Service Bus mediante la [biblioteca JMS de Azure Service Bus](https://search.maven.org/artifact/com.microsoft.azure/azure-servicebus-jms).
>
> En el caso de la versión preliminar pública, estas suscripciones no se pueden crear mediante Azure Portal.
>

### <a name="shared-durable-subscriptions"></a>Suscripciones duraderas compartidas

Una suscripción duradera compartida se usa cuando una aplicación va a recibir y procesar todos los mensajes publicados en un tema, independientemente de si la aplicación se utiliza de forma activa en la suscripción en todo momento.

Puesto que se trata de una suscripción compartida, cualquier aplicación autenticada para recibir de Service Bus puede recibir de la suscripción.

Para crear una suscripción duradera compartida, use los métodos siguientes de la clase `JMSContext`:

```java
JMSConsumer createSharedDurableConsumer(Topic topic, String name)

JMSConsumer createSharedDurableConsumer(Topic topic, String name, String messageSelector)
```

La suscripción duradera compartida seguirá existiendo a menos que se elimine mediante el método `unsubscribe` de la clase `JMSContext`.

```java
void unsubscribe(String name)
```

### <a name="unshared-durable-subscriptions"></a>Suscripciones duraderas no compartidas

Al igual que una suscripción duradera compartida, una suscripción duradera no compartida se usa cuando una aplicación va a recibir y procesar todos los mensajes publicados en un tema, independientemente de si la aplicación se utiliza de forma activa en la suscripción en todo momento.

Sin embargo, puesto que se trata de una suscripción no compartida, solo la aplicación que creó la suscripción puede recibir de ella.

Para crear una suscripción duradera no compartida, use los métodos siguientes de la clase `JMSContext`: 

```java
JMSConsumer createDurableConsumer(Topic topic, String name)

JMSConsumer createDurableConsumer(Topic topic, String name, String messageSelector, boolean noLocal)
```

> [!NOTE]
> La característica `noLocal` no se admite actualmente y se omite.
>

La suscripción duradera no compartida seguirá existiendo a menos que se elimine mediante el método `unsubscribe` de la clase `JMSContext`.

```java
void unsubscribe(String name)
```

### <a name="shared-non-durable-subscriptions"></a>Suscripciones no duraderas compartidas

Una suscripción no duradera compartida se usa cuando es necesario que varias aplicaciones cliente reciban y procesen mensajes de una suscripción única, solo hasta que se utilicen de forma activa en ella o reciban de ella.

Dado que la suscripción no es duradera, no se conserva. Esta suscripción no recibe los mensajes cuando no hay consumidores activos en la misma.

Para crear una suscripción no duradera compartida, cree `JmsConsumer` como se muestra en los siguientes métodos de la clase `JMSContext`:

```java
JMSConsumer createSharedConsumer(Topic topic, String sharedSubscriptionName)

JMSConsumer createSharedConsumer(Topic topic, String sharedSubscriptionName, String messageSelector)
```

La suscripción no duradera compartida seguirá existiendo hasta que haya consumidores activos que reciban de la misma.

### <a name="unshared-non-durable-subscriptions"></a>Suscripciones no duraderas no compartidas

Una suscripción no duradera no compartida se usa cuando es necesario que la aplicación cliente reciba y procese un mensaje de una suscripción, solo hasta que se utilice de forma activa en ella. Solo puede existir un consumidor en esta suscripción, es decir, el cliente que creó la suscripción.

Dado que la suscripción no es duradera, no se conserva. Esta suscripción no recibe los mensajes cuando no hay ningún consumidor activo en la misma.

Para crear una suscripción no duradera no compartida, cree `JMSConsumer` como se muestra en los siguientes métodos de la clase `JMSContext`:

```java
JMSConsumer createConsumer(Destination destination)

JMSConsumer createConsumer(Destination destination, String messageSelector)

JMSConsumer createConsumer(Destination destination, String messageSelector, boolean noLocal)
```

> [!NOTE]
> La característica `noLocal` no se admite actualmente y se omite.
>

La suscripción no duradera no compartida seguirá existiendo hasta que haya un consumidor activo que reciba de la misma.

### <a name="message-selectors"></a>Selectores de mensajes

Al igual que existen **filtros y acciones** para las suscripciones normales a Service Bus, existen **selectores de mensajes** para las suscripciones de JMS.

Los selectores de mensajes se pueden configurar en cada una de las suscripciones de JMS y existen como una condición de filtro en las propiedades del encabezado del mensaje. Solo se entregarán aquellos mensajes cuyas propiedades de encabezado coincidan con la expresión de selector de mensajes. Un valor de null o una cadena vacía indica que no hay ningún selector de mensajes para el consumidor o la suscripción de JMS.

## <a name="next-steps"></a>Pasos siguientes

Para más información y ejemplos de cómo usar la mensajería de Service Bus, consulte estos temas avanzados:

* [Introducción a la mensajería de Service Bus](service-bus-messaging-overview.md)
* [Uso de la API de Java Message Service 2.0 con Azure Service Bus Premium (versión preliminar)](how-to-use-java-message-service-20.md)



