---
title: Uso de la API de Java Message Service 2.0 con Azure Service Bus Premium
description: Aprenda a usar Java Message Service (JMS) con Azure Service Bus.
ms.topic: article
ms.date: 07/17/2020
ms.custom: seo-java-july2019, seo-java-august2019, seo-java-september2019
ms.openlocfilehash: 8363011187a4c2ef77681ece4bb8b1de73ec7a63
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87801267"
---
# <a name="use-java-message-service-20-api-with-azure-service-bus-premium-preview"></a>Uso de la API de Java Message Service 2.0 con Azure Service Bus Premium (versión preliminar)

En este artículo se explica cómo usar la popular API de **Java Message Service (JMS) 2.0** para interactuar con Azure Service Bus en el protocolo Advanced Message Queuing Protocol (AMQP 1.0).

> [!NOTE]
> La compatibilidad con la API de Java Message Service (JMS) 2.0 solo está disponible en el **nivel Premium de Azure Service Bus** y actualmente se encuentra en **versión preliminar**.
>

## <a name="get-started-with-service-bus"></a>Introducción a Service Bus

En esta guía se da por hecho que ya dispone de un espacio de nombres de Service Bus. Si no es así, puede [crear el espacio de nombres y la cola](service-bus-create-namespace-portal.md) con [Azure Portal](https://portal.azure.com). 

Para obtener más información sobre cómo crear espacios de nombres y colas de Service Bus, vea [Introducción a las colas de Service Bus a través de Azure Portal](service-bus-quickstart-portal.md).

## <a name="what-jms-features-are-supported"></a>¿Qué características de JMS se admiten?

[!INCLUDE [service-bus-jms-features-list](../../includes/service-bus-jms-feature-list.md)]

## <a name="downloading-the-java-message-service-jms-client-library"></a>Descarga de la biblioteca cliente de Java Message Service (JMS)

Para usar todas las características disponibles en el nivel Premium de Azure Service Bus, se debe agregar la siguiente biblioteca a la ruta de acceso de compilación del proyecto.

[Azure-servicebus-jms](https://search.maven.org/artifact/com.microsoft.azure/azure-servicebus-jms)

> [!NOTE]
> Para agregar [Azure-servicebus-jms](https://search.maven.org/artifact/com.microsoft.azure/azure-servicebus-jms) a la ruta de acceso de compilación, use la herramienta de administración de dependencias preferida para su proyecto, como [Maven](https://maven.apache.org/) o [Gradle](https://gradle.org/).
>

## <a name="coding-java-applications"></a>Codificación de las aplicaciones Java

Una vez que se han importado las dependencias, las aplicaciones de Java se pueden escribir en un proveedor de JMS de manera independiente.

### <a name="connecting-to-azure-service-bus-using-jms"></a>Conexión a Azure Service Bus mediante JMS

Para conectarse a Azure Service Bus mediante clientes de JMS, necesita una **cadena de conexión**, que está disponible en [Azure Portal](https://portal.azure.com), en la opción **Cadena de conexión principal** de las directivas de acceso compartido.

1. Creación de una instancia de `ServiceBusJmsConnectionFactorySettings`

    ```java
    ServiceBusJmsConnectionFactorySettings connFactorySettings = new ServiceBusJmsConnectionFactorySettings();
    connFactorySettings.setConnectionIdleTimeoutMS(20000);
    ```
2. Cree una instancia de `ServiceBusJmsConnectionFactory` con valor de `ServiceBusConnectionString` adecuado.

    ```java
    String ServiceBusConnectionString = "<SERVICE_BUS_CONNECTION_STRING_WITH_MANAGE_PERMISSIONS>";
    ConnectionFactory factory = new ServiceBusJmsConnectionFactory(ServiceBusConnectionString, connFactorySettings);
    ```

3. Use `ConnectionFactory` para crear `Connection` y, a continuación, `Session` 

    ```java
    Connection connection = factory.createConnection();
    Session session = connection.createSession();
    ```
    o `JMSContext` (para clientes de JMS 2.0).

    ```java
    JMSContext jmsContext = factory.createContext();
    ```

### <a name="write-the-jms-application"></a>Escritura de la aplicación JMS

Una vez creadas las instancias de `Session` o `JMSContext`, la aplicación puede usar las conocidas API de JMS para realizar operaciones de administración y de datos.

Consulte la lista de [características de JMS compatibles](how-to-use-java-message-service-20.md#what-jms-features-are-supported) para ver qué API se admiten como parte de esta versión preliminar.

A continuación, se muestran algunos fragmentos de código de ejemplo para empezar a trabajar con JMS:

#### <a name="sending-messages-to-a-queue-and-topic"></a>Envío de mensajes a una cola y un tema

```java
// Create the queue and topic
Queue queue = jmsContext.createQueue("basicQueue");
Topic topic = jmsContext.createTopic("basicTopic");
// Create the message
Message msg = jmsContext.createMessage();

// Create the JMS message producer
JMSProducer producer = jmsContext.createProducer();

// send the message to the queue
producer.send(queue, msg);
// send the message to the topic
producer.send(topic, msg);
```

#### <a name="receiving-messages-from-a-queue"></a>Recepción de mensajes desde una cola

```java
// Create the queue
Queue queue = jmsContext.createQueue("basicQueue");

// Create the message consumer
JMSConsumer consumer = jmsContext.createConsumer(queue);

// Receive the message
Message msg = (Message) consumer.receive();
```

#### <a name="receiving-messages-from-a-shared-durable-subscription-on-a-topic"></a>Recepción de mensajes desde una suscripción duradera compartida en un tema

```java
// Create the topic
Topic topic = jmsContext.createTopic("basicTopic");

// Create a shared durable subscriber on the topic
JMSConsumer sharedDurableConsumer = jmsContext.createSharedDurableConsumer(topic, "sharedDurableConsumer");

// Receive the message
Message msg = (Message) sharedDurableConsumer.receive();
```

## <a name="summary"></a>Resumen

En esta guía se muestra cómo las aplicaciones cliente de Java que usan Java Message Service (JMS) a través de AMQP 1.0 pueden interactuar con Azure Service Bus.

También puede utilizar AMQP 1.0 de Service Bus desde otros lenguajes, como .NET, C, Python y PHP. Los componentes creados utilizando estos lenguajes pueden intercambiar mensajes con seguridad y fidelidad gracias a la compatibilidad de AMQP 1.0 en Service Bus.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Service Bus y las entidades de Java Message Service (JMS), consulte los vínculos siguientes: 
* [Colas, temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md)
* [Service Bus: entidades de Java Message Service](service-bus-queues-topics-subscriptions.md#java-message-service-jms-20-entities-preview)
* [Compatibilidad de AMQP 1.0 en Azure Service Bus](service-bus-amqp-overview.md)
* [Guía para desarrolladores sobre AMQP 1.0 de Service Bus](service-bus-amqp-dotnet.md)
* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [API de Java Message Service (documento de Oracle externo)](https://docs.oracle.com/javaee/7/api/javax/jms/package-summary.html)
* [Migración de ActiveMQ a Service Bus](migrate-jms-activemq-to-servicebus.md)
