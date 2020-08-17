---
title: Uso de AMQP con Java Message Service API y Azure Service Bus
description: Uso de Java Message Service (JMS) con Azure Service Bus y Advanced Message Queuing Protocol (AMQP) 1.0.
ms.topic: article
ms.date: 06/23/2020
ms.custom: seo-java-july2019, seo-java-august2019, seo-java-september2019, devx-track-java
ms.openlocfilehash: 95558f973041a086dc66d2bebb1ce54ee58c952a
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87320789"
---
# <a name="use-the-java-message-service-jms-with-azure-service-bus-and-amqp-10"></a>Uso de Java Message Service (JMS) con Service Bus y AMQP 1.0
En este artículo se explica cómo utilizar las características de mensajería de Azure Service Bus (colas y temas de publicación y suscripción) desde aplicaciones de Java mediante el popular estándar de API Java Message Service (JMS). Hay un [artículo complementario](service-bus-amqp-dotnet.md) en el que se explica cómo hacer lo mismo mediante la API de Azure Service Bus para .NET. Puede utilizar estas dos guías conjuntamente para obtener información acerca de la mensajería entre diferentes plataformas mediante AMQP 1.0.

Advanced Message Queuing Protocol (AMQP) 1.0 es un protocolo de mensajes a nivel de red, confiable y eficaz que se puede utilizar para crear aplicaciones de mensajería robustas y compatibles con varias plataformas.

La compatibilidad con AMQP 1.0 de Azure Service Bus implica que puede utilizar las funciones de colas y publicación/suscripción de mensajería asincrónica desde una amplia variedad de plataformas mediante un eficaz protocolo binario. Además, puede desarrollar aplicaciones formadas por componentes creados con una mezcla de lenguajes, marcos y sistemas operativos.

## <a name="get-started-with-service-bus"></a>Introducción a Service Bus
En esta guía se da por hecho que ya dispone de un espacio de nombres de Service Bus con una cola denominada `basicqueue`. Si no es así, puede [crear el espacio de nombres y la cola](service-bus-create-namespace-portal.md) con [Azure Portal](https://portal.azure.com). Para obtener más información sobre cómo crear espacios de nombres y colas de Service Bus, vea [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md).

> [!NOTE]
> Los temas y colas con particiones también admiten AMQP. Para más información, consulte [Temas y colas con particiones](service-bus-partitioning.md) y [Compatibilidad de AMQP 1.0 con los temas y las colas con particiones de Service Bus](service-bus-partitioned-queues-and-topics-amqp-overview.md).
> 
> 

## <a name="downloading-the-amqp-10-jms-client-library"></a>Descarga de la biblioteca de cliente AMQP 1.0 JMS
Para obtener más información sobre dónde descargar la versión más reciente de la biblioteca cliente de Apache Qpid JMS AMQP 1.0, visite [https://qpid.apache.org/download.html](https://qpid.apache.org/download.html).

Debe agregar los cuatro archivos JAR siguientes del archivo de distribución de Apache Qpid JMS AMQP 1.0 al CLASSPATH de Java cuando vaya a crear y ejecutar aplicaciones JMS con Service Bus:

* geronimo-jms\_1.1\_spec-1.0.jar
* qpid-jms-client-[version].jar

> [!NOTE]
> Los nombres y las versiones de los archivos JAR de JMS pueden haber cambiado. Para más información, consulte [Qpid JMS - AMQP 1.0](https://qpid.apache.org/maven.html#qpid-jms-amqp-10).

## <a name="coding-java-applications"></a>Codificación de las aplicaciones Java
### <a name="java-naming-and-directory-interface-jndi"></a>Interfaz de denominación y directorio Java (JNDI)
JMS usa la interfaz de denominación y directorio Java (JNDI) para crear una separación entre los nombres lógicos y los físicos. Se resuelven dos tipos de objeto JMS mediante JNDI: ConnectionFactory y Destination. JNDI usa un modelo de proveedor al que se pueden acoplar diferentes servicios de directorio para controlar labores de resolución de nombres. La biblioteca Apache Qpid JMS AMQP 1.0 se presenta con un proveedor JNDI basado en archivo de propiedades sencillo que se configura mediante un archivo de propiedades cuyo formato es el siguiente:

```TEXT
# servicebus.properties - sample JNDI configuration

# Register a ConnectionFactory in JNDI using the form:
# connectionfactory.[jndi_name] = [ConnectionURL]
connectionfactory.SBCF = amqps://[SASPolicyName]:[SASPolicyKey]@[namespace].servicebus.windows.net

# Register some queues in JNDI using the form
# queue.[jndi_name] = [physical_name]
# topic.[jndi_name] = [physical_name]
queue.QUEUE = queue1
```

#### <a name="setup-jndi-context-and-configure-the-connectionfactory"></a>Configuración del contexto de JNDI y de ConnectionFactory

El elemento **ConnectionString** al que se hace referencia es el único disponible en "Directivas de acceso compartido" de [Azure Portal](https://portal.azure.com) en **Cadena de conexión primaria**
```java
// The connection string builder is the only part of the azure-servicebus SDK library
// we use in this JMS sample and for the purpose of robustly parsing the Service Bus 
// connection string. 
ConnectionStringBuilder csb = new ConnectionStringBuilder(connectionString);
        
// set up JNDI context
Hashtable<String, String> hashtable = new Hashtable<>();
hashtable.put("connectionfactory.SBCF", "amqps://" + csb.getEndpoint().getHost() + "?amqp.idleTimeout=120000&amqp.traceFrames=true");
hashtable.put("queue.QUEUE", "BasicQueue");
hashtable.put(Context.INITIAL_CONTEXT_FACTORY, "org.apache.qpid.jms.jndi.JmsInitialContextFactory");
Context context = new InitialContext(hashtable);

ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");

// Look up queue
Destination queue = (Destination) context.lookup("QUEUE");
```

#### <a name="configure-producer-and-consumer-destination-queues"></a>Configuración de las colas de destino de productor y consumidor
La entrada usada para definir un destino en el proveedor JNDI de archivo de propiedades Qpid tiene el formato siguiente:

Para crear la cola de destino del productor: 
```java
String queueName = "queueName";
Destination queue = (Destination) queueName;

ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");
Connection connection - cf.createConnection(csb.getSasKeyName(), csb.getSasKey());

Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);

// Create Producer
MessageProducer producer = session.createProducer(queue);
```

Para crear una cola de destino para el consumidor: 
```java
String queueName = "queueName";
Destination queue = (Destination) queueName;

ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");
Connection connection - cf.createConnection(csb.getSasKeyName(), csb.getSasKey());

Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);

// Create Consumer
MessageConsumer consumer = session.createConsumer(queue);
```

### <a name="write-the-jms-application"></a>Escritura de la aplicación JMS
No existen API especiales ni opciones obligatorias al usar JMS con Service Bus. Sin embargo, hay algunas restricciones que se tratarán más adelante. Como con cualquier otra aplicación JMS, el primer requisito necesario es configurar el entorno JNDI para poder resolver **ConnectionFactory** y los destinos.

#### <a name="configure-the-jndi-initialcontext"></a>Configuración de JNDI InitialContext
El entorno JNDI se configura pasando una tabla hash con información de configuración al constructor de la clase javax.naming.InitialContext. Los dos elementos necesarios de la tabla hash son el nombre de la clase de Initial Context Factory y la URL del proveedor. El código siguiente indica cómo configurar el entorno JNDI para usar el proveedor JNDI basado en archivo de propiedades Qpid con un archivo de propiedades llamado **servicebus.properties**.

```java
// set up JNDI context
Hashtable<String, String> hashtable = new Hashtable<>();
hashtable.put("connectionfactory.SBCF", "amqps://" + csb.getEndpoint().getHost() + \
"?amqp.idleTimeout=120000&amqp.traceFrames=true");
hashtable.put("queue.QUEUE", "BasicQueue");
hashtable.put(Context.INITIAL_CONTEXT_FACTORY, "org.apache.qpid.jms.jndi.JmsInitialContextFactory");
Context context = new InitialContext(hashtable);
``` 

### <a name="a-simple-jms-application-using-a-service-bus-queue"></a>Aplicación JMS sencilla que usa una cola de Service Bus
El programa de ejemplo siguiente envía JMS TextMessages a una cola de Service Bus con el nombre lógico JNDI de QUEUE y recibe los mensajes de vuelta.

Puede acceder a todo el código fuente y la información de configuración desde la [guía de inicio rápido de colas de JMS de ejemplo de Azure Service Bus](https://github.com/Azure/azure-service-bus/tree/master/samples/Java/qpid-jms-client/JmsQueueQuickstart).

```java
// Copyright (c) Microsoft. All rights reserved.
// Licensed under the MIT license. See LICENSE file in the project root for full license information.

package com.microsoft.azure.servicebus.samples.jmsqueuequickstart;

import com.microsoft.azure.servicebus.primitives.ConnectionStringBuilder;
import org.apache.commons.cli.*;
import org.apache.log4j.*;

import javax.jms.*;
import javax.naming.Context;
import javax.naming.InitialContext;
import java.util.Hashtable;
import java.util.concurrent.atomic.AtomicInteger;
import java.util.function.Function;

/**
 * This sample demonstrates how to send messages from a JMS Queue producer into
 * an Azure Service Bus Queue, and receive them with a JMS message consumer.
 * JMS Queue. 
 */
public class JmsQueueQuickstart {

    // Number of messages to send
    private static int totalSend = 10;
    //Tracking counter for how many messages have been received; used as termination condition
    private static AtomicInteger totalReceived = new AtomicInteger(0);
    // log4j logger 
    private static Logger logger = Logger.getRootLogger();

    public void run(String connectionString) throws Exception {

        // The connection string builder is the only part of the azure-servicebus SDK library
        // we use in this JMS sample and for the purpose of robustly parsing the Service Bus 
        // connection string. 
        ConnectionStringBuilder csb = new ConnectionStringBuilder(connectionString);
        
        // set up JNDI context
        Hashtable<String, String> hashtable = new Hashtable<>();
        hashtable.put("connectionfactory.SBCF", "amqps://" + csb.getEndpoint().getHost() + "?amqp.idleTimeout=120000&amqp.traceFrames=true");
        hashtable.put("queue.QUEUE", "BasicQueue");
        hashtable.put(Context.INITIAL_CONTEXT_FACTORY, "org.apache.qpid.jms.jndi.JmsInitialContextFactory");
        Context context = new InitialContext(hashtable);
        ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");
        
        // Look up queue
        Destination queue = (Destination) context.lookup("QUEUE");

        // we create a scope here so we can use the same set of local variables cleanly 
        // again to show the receive side separately with minimal clutter
        {
            // Create Connection
            Connection connection = cf.createConnection(csb.getSasKeyName(), csb.getSasKey());
            // Create Session, no transaction, client ack
            Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);

            // Create producer
            MessageProducer producer = session.createProducer(queue);

            // Send messages
            for (int i = 0; i < totalSend; i++) {
                BytesMessage message = session.createBytesMessage();
                message.writeBytes(String.valueOf(i).getBytes());
                producer.send(message);
                System.out.printf("Sent message %d.\n", i + 1);
            }

            producer.close();
            session.close();
            connection.stop();
            connection.close();
        }

        {
            // Create Connection
            Connection connection = cf.createConnection(csb.getSasKeyName(), csb.getSasKey());
            connection.start();
            // Create Session, no transaction, client ack
            Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
            // Create consumer
            MessageConsumer consumer = session.createConsumer(queue);
            // create a listener callback to receive the messages
            consumer.setMessageListener(message -> {
                try {
                    // receives message is passed to callback
                    System.out.printf("Received message %d with sq#: %s\n",
                            totalReceived.incrementAndGet(), // increments the tracking counter
                            message.getJMSMessageID());
                    message.acknowledge();
                } catch (Exception e) {
                    logger.error(e);
                }
            });

            // wait on the main thread until all sent messages have been received
            while (totalReceived.get() < totalSend) {
                Thread.sleep(1000);
            }
            consumer.close();
            session.close();
            connection.stop();
            connection.close();
        }

        System.out.printf("Received all messages, exiting the sample.\n");
        System.out.printf("Closing queue client.\n");
    }

    public static void main(String[] args) {

        System.exit(runApp(args, (connectionString) -> {
            JmsQueueQuickstart app = new JmsQueueQuickstart();
            try {
                app.run(connectionString);
                return 0;
            } catch (Exception e) {
                System.out.printf("%s", e.toString());
                return 1;
            }
        }));
    }

    static final String SB_SAMPLES_CONNECTIONSTRING = "SB_SAMPLES_CONNECTIONSTRING";

    public static int runApp(String[] args, Function<String, Integer> run) {
        try {

            String connectionString = null;

            // parse connection string from command line
            Options options = new Options();
            options.addOption(new Option("c", true, "Connection string"));
            CommandLineParser clp = new DefaultParser();
            CommandLine cl = clp.parse(options, args);
            if (cl.getOptionValue("c") != null) {
                connectionString = cl.getOptionValue("c");
            }

            // get overrides from the environment
            String env = System.getenv(SB_SAMPLES_CONNECTIONSTRING);
            if (env != null) {
                connectionString = env;
            }

            if (connectionString == null) {
                HelpFormatter formatter = new HelpFormatter();
                formatter.printHelp("run jar with", "", options, "", true);
                return 2;
            }
            return run.apply(connectionString);
        } catch (Exception e) {
            System.out.printf("%s", e.toString());
            return 3;
        }
    }
}
```

### <a name="run-the-application"></a>Ejecución de la aplicación
Pase la **cadena de conexión** desde las directivas de acceso compartido para ejecutar la aplicación.
Esta es la salida del formulario mediante la ejecución de la aplicación:

```Output
> mvn clean package
>java -jar ./target/jmsqueuequickstart-1.0.0-jar-with-dependencies.jar -c "<CONNECTION_STRING>"

Sent message 1.
Sent message 2.
Sent message 3.
Sent message 4.
Sent message 5.
Sent message 6.
Sent message 7.
Sent message 8.
Sent message 9.
Sent message 10.
Received message 1 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-1
Received message 2 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-2
Received message 3 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-3
Received message 4 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-4
Received message 5 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-5
Received message 6 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-6
Received message 7 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-7
Received message 8 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-8
Received message 9 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-9
Received message 10 with sq#: ID:7f6a7659-bcdf-4af6-afc1-4011e2ddcb3c:1:1:1-10
Received all messages, exiting the sample.
Closing queue client.

```

## <a name="amqp-disposition-and-service-bus-operation-mapping"></a>Asignación de la operación de Service Bus y eliminación de AMQP
Así es como se convierte una disposición de AMQP en una operación de Service Bus:

```Output
ACCEPTED = 1; -> Complete()
REJECTED = 2; -> DeadLetter()
RELEASED = 3; (just unlock the message in service bus, will then get redelivered)
MODIFIED_FAILED = 4; -> Abandon() which increases delivery count
MODIFIED_FAILED_UNDELIVERABLE = 5; -> Defer()
```

## <a name="jms-topics-vs-service-bus-topics"></a>Temas de JMS frente a Temas de Service Bus
El uso de temas y suscripciones de Azure Service Bus a través de la API de Java Message Service (JMS) proporciona funcionalidades de envío y recepción básicas. Es una opción práctica al migrar aplicaciones desde otros agentes de mensajes con API compatibles con JMS, aunque los temas de Service Bus sean distintos de los de JMS y necesiten algunos ajustes. 

Los temas de Azure Service Bus enrutan mensajes en suscripciones con nombre, compartidas y duraderas que se administran a través de la interfaz de administración de recursos de Azure, las herramientas de línea de comandos de Azure o a través de Azure Portal. Cada suscripción permite hasta 2000 reglas de selección, que pueden tener una condición de filtro cada una y, en el caso de los filtros SQL, también una acción de transformación de metadatos. Cada coincidencia de condición de filtro selecciona el mensaje de entrada que se va a copiar en la suscripción.  

Recibir mensajes de suscripciones es igual que recibir mensajes de colas. Cada suscripción tiene una cola de mensajes fallidos asociada, así como la capacidad de reenviar mensajes automáticamente a otra cola o temas. 

Los temas de JMS permiten a los clientes crear de forma dinámica suscriptores duraderos y no duraderos que permiten opcionalmente filtrar mensajes con selectores de mensajes. Estas entidades no compartidas no son compatibles con Service Bus. Pero la sintaxis de regla de filtro SQL para Service Bus es muy similar a la sintaxis del selector de mensajes compatible con JMS. 

El lado del publicador de los temas de JMS es compatible con Service Bus, como se muestra en este ejemplo, pero los suscriptores dinámicos no lo son. Las siguientes API de JMS relacionadas con la topología no son compatibles con Service Bus. 

## <a name="unsupported-features-and-restrictions"></a>Características no admitidas y restricciones
Existen las restricciones siguientes al usar JMS sobre AMQP 1.0 con Service Bus, a saber:

* Solo se permite un elemento **MessageProducer** o **MessageConsumer** por cada **sesión**. Si tiene que crear varios elementos **MessageProducers** o **MessageConsumers** en una aplicación, cree una **sesión** dedicada para cada uno de ellos.
* Actualmente no se admiten las suscripciones a tema volátiles.
* Por el momento no se admiten **MessageSelectors**.
* No se admiten las transacciones distribuidas (pero sí las sesiones de transacción).

Además, Azure Service Bus divide el plano de control del plano de datos y, por tanto, no admite varias funciones de topología dinámica de JMS:

| Método no admitido          | Reemplazar por                                                                             |
|-----------------------------|------------------------------------------------------------------------------------------|
| createDurableSubscriber     | Crea una suscripción a un tema que lleva el selector de mensajes                                 |
| createDurableConsumer       | Crea una suscripción a un tema que lleva el selector de mensajes                                 |
| createSharedConsumer        | Los temas de Service Bus siempre se pueden compartir, consulte anteriormente.                                       |
| createSharedDurableConsumer | Los temas de Service Bus siempre se pueden compartir, consulte anteriormente.                                       |
| createTemporaryTopic        | Crea un tema mediante la API de administración, herramientas o el portal con *AutoDeleteOnIdle* establecido en un período de expiración. |
| createTopic                 | Crea un tema mediante la API de administración, herramientas o el portal.                                           |
| unsubscribe                 | Elimina el tema mediante la API de administración, herramientas o el portal.                                             |
| createBrowser               | No se admite. Uso de la funcionalidad Peek() de la API de Service Bus                         |
| createQueue                 | Crea una cola mediante la API de administración, herramientas o el portal.                                           | 
| createTemporaryQueue        | Crea una cola mediante la API de administración, herramientas o el portal con *AutoDeleteOnIdle* establecido en un período de expiración. |
| receiveNoWait               | Usa el método receive() proporcionado por el SDK de Service Bus y especifica un tiempo de expiración muy bajo o cero. |

## <a name="summary"></a>Resumen
En esta guía de instrucciones se indica cómo usar las características de mensajería asíncrona de Service Bus (colas y publicación/suscripción a temas) desde Java utilizando las populares JMS API y AMQP 1.0.

También puede utilizar AMQP 1.0 de Service Bus desde otros lenguajes, como .NET, C, Python y PHP. Los componentes creados utilizando estos lenguajes pueden intercambiar mensajes con seguridad y fidelidad gracias a la compatibilidad de AMQP 1.0 en Service Bus.

## <a name="next-steps"></a>Pasos siguientes
* [Compatibilidad de AMQP 1.0 en Azure Service Bus](service-bus-amqp-overview.md)
* [Uso de AMQP 1.0 con la API .NET de Service Bus](service-bus-dotnet-advanced-message-queuing.md)
* [Guía para desarrolladores sobre AMQP 1.0 de Service Bus](service-bus-amqp-dotnet.md)
* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Centro para desarrolladores de Java](https://azure.microsoft.com/develop/java/)

