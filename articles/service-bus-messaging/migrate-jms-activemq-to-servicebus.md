---
title: Migración de aplicaciones de Java Message Service (JMS) de Apache ActiveMQ a Azure Service Bus | Microsoft Docs
description: En este artículo se explica cómo migrar aplicaciones JMS existentes que interactúan con Apache ActiveMQ para interactuar con Azure Service Bus.
services: service-bus-messaging
documentationcenter: ''
author: axisc
manager: timlt
editor: spelluru
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/07/2020
ms.author: aschhab
ms.custom: devx-track-java
ms.openlocfilehash: 1b07faa5b2540aafafc27a51192d824d4445ce35
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067161"
---
# <a name="migrate-existing-java-message-service-jms-20-applications-from-apache-activemq-to-azure-service-bus"></a>Migración de aplicaciones de Java Message Service (JMS) 2.0 de Apache ActiveMQ a Azure Service Bus

En este artículo se explica cómo modificar una aplicación de Java Message Service (JMS) 2.0 existente que interactúa con un agente de JMS para que interactúe con Azure Service Bus en su lugar. En concreto, en el artículo se trata la migración de Apache ActiveMQ o Amazon MQ.

Azure Service Bus admite cargas de trabajo de Java 2 Platform, Enterprise Edition y Spring que usan la API de JMS 2.0 en el protocolo Advanced Message Queuing Protocol (AMQP).

## <a name="before-you-start"></a>Antes de comenzar

### <a name="differences-between-azure-service-bus-and-apache-activemq"></a>Diferencias entre Azure Service Bus y Apache ActiveMQ

Tanto Azure Service Bus como Apache ActiveMQ son agentes de mensajes que funcionan como proveedores de JMS para que las aplicaciones del cliente envíen y reciban mensajes. Ambos agentes permiten la semántica punto a punto con colas y la semántica de publicación-suscripción con temas y suscripciones. 

Aun así, hay varias diferencias entre ambos, tal como se muestra en la tabla siguiente:

| Category | ActiveMQ | Azure Service Bus |
| --- | --- | --- |
| Aplicación por niveles | Monolito en clúster | Dos niveles <br> (puerta de enlace + back end) |
| Compatibilidad con protocolos | <ul> <li>AMQP</li> <li> STOMP </li> <li> OpenWire </li> </ul> | AMQP |
| Modo de aprovisionamiento | <ul> <li> Infraestructura como servicio (IaaS), local </li> <li> Amazon MQ (plataforma administrada como servicio) </li> | Plataforma administrada como servicio (PaaS) |
| Tamaño del mensaje | Configurable por el usuario | 1 MB (nivel premium) |
| Alta disponibilidad | Administrado por el cliente | Administrado por plataforma |
| Recuperación ante desastres | Administrado por el cliente | Administrado por plataforma | 

### <a name="current-supported-and-unsupported-features"></a>Características admitidas y no admitidas actualmente

[!INCLUDE [service-bus-jms-features-list](../../includes/service-bus-jms-feature-list.md)]

### <a name="considerations"></a>Consideraciones

La naturaleza de dos niveles de Azure Service Bus ofrece diversas funcionalidades de continuidad del negocio (alta disponibilidad y recuperación ante desastres). Sin embargo, cuando se usan las características de JMS es preciso tener en cuenta varias consideraciones.

#### <a name="service-upgrades"></a>Actualizaciones del servicio

En el caso de las actualizaciones y los reinicios de Service Bus, se eliminan las colas o los temas temporales. Si su aplicación es sensible a la pérdida de datos en colas o temas temporales, no utilice colas ni temas temporales. Use colas, temas y suscripciones duraderas en su lugar.

#### <a name="data-migration"></a>Migración de datos

Como parte de la migración y modificación de las aplicaciones cliente para interactuar con Azure Service Bus, los datos que se conservan en ActiveMQ no se migran a Service Bus. Es posible que necesite una aplicación personalizada para purgar las colas, los temas y las suscripciones de ActiveMQ y, posteriormente, reproducir los mensajes en las colas, los temas y las suscripciones de Service Bus.

#### <a name="authentication-and-authorization"></a>Autenticación y autorización

El control de acceso basado en rol (RBAC) respaldado por Azure Active Directory es el mecanismo de autenticación preferido para Service Bus. No obstante, como la autenticación basada en notificaciones o RBAC no es compatible actualmente con Apache QPID JMS, es preciso usar claves de SAS para la autenticación.

## <a name="pre-migration"></a>Antes de la migración

### <a name="version-check"></a>Comprobación de versión

Puede usar los siguientes componentes y versiones mientras escribe las aplicaciones de JMS: 

| Componente | Versión |
|---|---|
| API de Java Message Service (JMS) | 1.1 o superior |
| Protocolo AMQP | 1.0 |

### <a name="ensure-that-amqp-ports-are-open"></a>Asegúrese de que los puertos AMQP estén abiertos.

Service Bus admite la comunicación a través del protocolo AMQP. Para ello, habilite la comunicación a través de los puertos 5671 (AMQP) y 443 (TCP). Según dónde se hospeden las aplicaciones cliente, es posible que necesite abrir una incidencia de soporte técnico para permitir la comunicación a través de estos puertos.

> [!IMPORTANT]
> Service Bus solo admite el protocolo AMQP 1.0.

### <a name="set-up-enterprise-configurations"></a>Realización de configuraciones empresariales

Service Bus habilita diversas características de seguridad y alta disponibilidad de la empresa. Para más información, consulte: 

  * [Puntos de conexión de servicio de red virtual](service-bus-service-endpoints.md)
  * [Firewall](service-bus-ip-filtering.md)
  * [Cifrado del lado del servicio con clave administrada por el cliente (BYOK)](configure-customer-managed-key.md)
  * [Puntos de conexión privados](private-link-service.md)
  * [Autenticación y autorización](service-bus-authentication-and-authorization.md)

### <a name="monitoring-alerts-and-tracing"></a>Supervisión, alertas y seguimiento

Para todos los espacios de nombres de Service Bus, publique las métricas en Azure Monitor. Estas métricas se pueden usar no solo para la generación de alertas, sino también para el escalado dinámico de los recursos asignados al espacio de nombres.

Para más información sobre las diferentes métricas y cómo configurar alertas en ellas, consulte [Métricas de Service Bus en Azure Monitor](service-bus-metrics-azure-monitor.md). También puede obtener más información sobre el [seguimiento del lado del cliente para las operaciones de datos](service-bus-end-to-end-tracing.md) y el [registro operativo o de diagnóstico para las operaciones de administración](service-bus-diagnostic-logs.md).

### <a name="metrics---new-relic"></a>Métricas: New Relic

Puede correlacionar qué métricas de ActiveMQ se asignan a las métricas de Azure Service Bus. Consulte lo siguiente en el sitio web de New Relic:

  * [Métricas de New Relic de ActiveMQ/Amazon MQ](https://docs.newrelic.com/docs/integrations/amazon-integrations/aws-integrations-list/aws-mq-integration)
  * [Métricas de New Relic de Azure Service Bus](https://docs.newrelic.com/docs/integrations/microsoft-azure-integrations/azure-integrations-list/azure-service-bus-monitoring-integration)

> [!NOTE]
> Actualmente, New Relic no tiene una integración directa perfecta con ActiveMQ, pero tiene métricas disponibles para Amazon MQ. Como Amazon MQ se deriva de ActiveMQ, la tabla siguiente asigna las métricas de New Relic de Amazon MQ a Azure Service Bus.
>

|Agrupación de métricas| Métrica de Amazon MQ/ActiveMQ | Métricas de Azure Service Bus |
|------------|---------------------------|--------------------------|
|Agente|`CpuUtilization`|`CPUXNS`|
|Agente|`MemoryUsage`|`WSXNS`|
|Agente|`CurrentConnectionsCount`|`activeConnections`|
|Agente|`EstablishedConnectionsCount`|`activeConnections` + `connectionsClosed`|
|Agente|`InactiveDurableTopicSubscribersCount`|Uso de métricas de suscripción|
|Agente|`TotalMessageCount`|Uso del nivel de cola, tema o suscripción `activeMessages`|
|Cola o tema|`EnqueueCount`|`incomingMessages`|
|Cola o tema|`DequeueCount`|`outgoingMessages`|
|Cola|`QueueSize`|`sizeBytes`|



## <a name="migration"></a>Migración

Para migrar una aplicación de JMS 2.0 existente para interactuar con Service Bus, siga los pasos que se indican en las siguientes secciones.

### <a name="export-the-topology-from-activemq-and-create-the-entities-in-service-bus-optional"></a>Exportación de la topología de ActiveMQ y creación de las entidades en Service Bus (opcional)

Para asegurarse de que las aplicaciones cliente pueden conectarse sin problemas con Service Bus, migre la topología (que incluye colas, temas y suscripciones) desde Apache ActiveMQ a Service Bus.

> [!NOTE]
> En el caso de las aplicaciones de JMS, cree las colas, los temas y las suscripciones como una operación en tiempo de ejecución. La mayoría de los proveedores de JMS (agentes de mensajes) brindan la posibilidad de crearlos en tiempo de ejecución. Ese es el motivo por el que este paso de exportación se considera opcional. Para asegurarse de que la aplicación tiene los permisos necesarios para crear la topología en tiempo de ejecución, use la cadena de conexión con permisos de SAS `Manage`.

Para ello, siga estos pasos:

1. Use las [herramientas de la línea de comandos de ActiveMQ](https://activemq.apache.org/activemq-command-line-tools-reference) para exportar la topología.
1. Vuelva a crear la misma topología mediante una [plantilla de Azure Resource Manager](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).
1. Ejecute la plantilla de Azure Resource Manager.


### <a name="import-the-maven-dependency-for-service-bus-jms-implementation"></a>Importe la dependencia de Maven para la implementación de Service Bus JMS.

Para garantizar una conectividad sin problemas con Service Bus, agregue el paquete `azure-servicebus-jms` como una dependencia al archivo `pom.xml` de Maven, como se indica a continuación:

```xml
<dependencies>
...
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>azure-servicebus-jms</artifactId>
    </dependency>
...
</dependencies>
```

### <a name="application-server-configuration-changes"></a>Cambios en la configuración del servidor de aplicaciones

Esta parte está personalizada para el servidor de aplicaciones que hospeda las aplicaciones cliente que se conectan a ActiveMQ.

#### <a name="tomcat"></a>Tomcat

Aquí es donde se comienza con la configuración específica de ActiveMQ, como se muestra en el archivo `/META-INF/context.xml`.

```XML
<Context antiJARLocking="true">
    <Resource
        name="jms/ConnectionFactory"
        auth="Container"
        type="org.apache.activemq.ActiveMQConnectionFactory"
        description="JMS Connection Factory"
        factory="org.apache.activemq.jndi.JNDIReferenceFactory"
        brokerURL="tcp://localhost:61616"
        brokerName="LocalActiveMQBroker"
        useEmbeddedBroker="false"/>

    <Resource name="jms/topic/MyTopic"
        auth="Container"
        type="org.apache.activemq.command.ActiveMQTopic"
        factory="org.apache.activemq.jndi.JNDIReferenceFactory"
        physicalName="MY.TEST.FOO"/>
    <Resource name="jms/queue/MyQueue"
        auth="Container"
        type="org.apache.activemq.command.ActiveMQQueue"
        factory="org.apache.activemq.jndi.JNDIReferenceFactory"
        physicalName="MY.TEST.FOO.QUEUE"/>
</Context>
```

Adáptela para que apunte a Service Bus, como se indica a continuación:

```xml
<Context antiJARLocking="true">
    <Resource
        name="jms/ConnectionFactory"
        auth="Container"
        type="com.microsoft.azure.servicebus.jms.ServiceBusJmsConnectionFactory"
        description="JMS Connection Factory"
        factory="org.apache.qpid.jms.jndi.JNDIReferenceFactory"
        connectionString="<INSERT YOUR SERVICE BUS CONNECTION STRING HERE>"/>

    <Resource name="jms/topic/MyTopic"
        auth="Container"
        type="org.apache.qpid.jms.JmsTopic"
        factory="org.apache.qpid.jms.jndi.JNDIReferenceFactory"
        physicalName="MY.TEST.FOO"/>
    <Resource name="jms/queue/MyQueue"
        auth="Container"
        type="org.apache.qpid.jms.JmsQueue"
        factory="org.apache.qpid.jms.jndi.JNDIReferenceFactory"
        physicalName="MY.TEST.FOO.QUEUE"/>
</Context>
```

#### <a name="spring-applications"></a>Aplicaciones de spring

##### <a name="update-the-applicationproperties-file"></a>Actualización del archivo `application.properties`

Si va a usar una aplicación de Spring Boot para conectarse a ActiveMQ, quite las propiedades específicas de ActiveMQ del archivo de `application.properties`.

```properties
spring.activemq.broker-url=<ACTIVEMQ BROKER URL>
spring.activemq.user=<ACTIVEMQ USERNAME>
spring.activemq.password=<ACTIVEMQ PASSWORD>
```

Luego, agregue las propiedades específicas de Service Bus al archivo `application.properties`.

```properties
azure.servicebus.connection-string=Endpoint=myEndpoint;SharedAccessKeyName=mySharedAccessKeyName;SharedAccessKey=mySharedAccessKey
```

##### <a name="replace-activemqconnectionfactory-with-servicebusjmsconnectionfactory"></a>Reemplazar `ActiveMQConnectionFactory` con `ServiceBusJmsConnectionFactory`

El siguiente paso es reemplazar la instancia de `ActiveMQConnectionFactory` por la de `ServiceBusJmsConnectionFactory`.

> [!NOTE] 
> Los cambios de código reales son específicos de la aplicación y de la forma en que administran las dependencias, pero en el ejemplo siguiente se proporciona una guía para saber lo que no se debe cambiar.
>

Anteriormente, es posible que haya creado una instancia de un objeto de `ActiveMQConnectionFactory` como se indica a continuación:

```java

String BROKER_URL = "<URL of the hosted ActiveMQ broker>";
ConnectionFactory factory = new ActiveMQConnectionFactory(BROKER_URL);

Connection connection = factory.createConnection();
connection.start();

```

Ahora, va a cambiarlo y va a crear una instancia de un objeto de `ServiceBusJmsConnectionFactory` como se indica a continuación:

```java

ServiceBusJmsConnectionFactorySettings settings = new ServiceBusJmsConnectionFactorySettings();
String SERVICE_BUS_CONNECTION_STRING = "<Service Bus Connection string>";

ConnectionFactory factory = new ServiceBusJmsConnectionFactory(SERVICE_BUS_CONNECTION_STRING, settings);

Connection connection = factory.createConnection();
connection.start();

```

## <a name="post-migration"></a>Después de la migración

Ahora que ha modificado la aplicación para empezar a enviar y recibir mensajes desde Service Bus, debe comprobar que funciona como cabría esperar. Cuando haya finalizado, puede continuar para refinar y modernizar aún más la pila de aplicaciones.

## <a name="next-steps"></a>Pasos siguientes

Use [Spring Boot Starter para JMS en Azure Service Bus](/azure/developer/java/spring-framework/configure-spring-boot-starter-java-app-with-azure-service-bus) para lograr una integración sin problemas con Service Bus.

Para más información sobre la mensajería y JMS de Service Bus, consulte:

* [JMS de Service Bus](service-bus-java-how-to-use-jms-api-amqp.md)
* [Colas, temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md)
* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)