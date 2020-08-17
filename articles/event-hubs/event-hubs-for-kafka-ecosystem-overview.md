---
title: 'Uso del centro de eventos desde una aplicación de Apache Kafka: Azure Event Hubs | Microsoft Docs'
description: En este artículo se proporciona información sobre la compatibilidad de Apache Kafka con Azure Event Hubs.
ms.topic: article
ms.date: 07/20/2020
ms.openlocfilehash: ab29c9c4270514e95752ab2bbd085ffe1b0a2fb0
ms.sourcegitcommit: 3d56d25d9cf9d3d42600db3e9364a5730e80fa4a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/03/2020
ms.locfileid: "87534879"
---
# <a name="use-azure-event-hubs-from-apache-kafka-applications"></a>Uso de Azure Event Hubs desde aplicaciones de Apache Kafka
Event Hubs proporciona un punto de conexión de Kafka que las aplicaciones basadas en Kafka existentes pueden usar como alternativa a la ejecución de su propio clúster de Kafka. Event Hubs admite el [protocolo de Apache Kafka 1.0 y versiones más recientes](https://kafka.apache.org/documentation/) y funciona con las aplicaciones existentes de Kafka, incluida MirrorMaker.  

> [!VIDEO https://www.youtube.com/embed/UE1WgB96_fc]

## <a name="what-does-event-hubs-for-kafka-provide"></a>¿Qué proporciona Event Hubs para Kafka?

La característica Event Hubs para Kafka proporciona un protocolo principal basado en Azure Event Hubs que es compatible a nivel binario con la versión 1.0 de Kafka y posteriores, tanto para leer desde temas de Kafka como para escribir en ellos. Puede empezar a utilizar el punto de conexión de Kafka desde sus aplicaciones sin tener que realizar ningún cambio en el código, pero será preciso realizar un cambio mínimo en la configuración. Actualice la cadena de conexión en las configuraciones para que apunte al punto de conexión de Kafka que se expone en el evento, en lugar de apuntar al clúster de Kafka. A partir de ahí, puede empezar a hacer streaming de los eventos desde las aplicaciones que usen el protocolo Kafka en Event Hubs. Esta integración también admite marcos como [Kafka Connect](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/connect), que está en versión preliminar actualmente. 

Desde un punto de vista conceptual, Kafka y Event Hubs son prácticamente idénticos: ambos son registros con particiones creados para transmitir datos. En la tabla siguiente, se asignan conceptos entre Kafka y Event Hubs.

### <a name="kafka-and-event-hub-conceptual-mapping"></a>Asignación conceptual de Kafka y Event Hubs

| Concepto de Kafka | Concepto de Event Hubs|
| --- | --- |
| Clúster | Espacio de nombres |
| Tema | Centro de eventos |
| Partition | Partition|
| Grupo de consumidores | Grupo de consumidores |
| Offset | Offset|

### <a name="key-differences-between-kafka-and-event-hubs"></a>Diferencias principales entre Kafka y Event Hubs

Mientras que [Kafka Apache](https://kafka.apache.org/) es un software que se puede ejecutar siempre que elija, Event Hubs es un servicio en la nube similar a Azure Blob Storage. No hay ningún servidor o red que administrar ni ningún agente que configurar. Se crea un espacio de nombres, que es un FQDN en el que están los temas, y después se crean Event Hubs o temas dentro de ese espacio de nombres. Para más información acerca de Event Hubs y de los espacios de nombres, consulte las [características de Event Hubs](event-hubs-features.md#namespace). Como servicio en la nube, Event Hubs usa una única dirección IP virtual estable como punto de conexión, de forma que los clientes no necesitan conocer los agentes o equipos de un clúster. 

La escala en Event Hubs se controla por el número de unidades de rendimiento que adquiere; cada unidad de rendimiento le otorga 1 MB por segundo o 1000 eventos por segundo de entrada. De forma predeterminada, Event Hubs escala verticalmente las unidades de procesamiento cuando se alcanza el límite con la característica [Inflado automático](event-hubs-auto-inflate.md); esta característica también funciona con la característica Event Hubs para Kafka. 

### <a name="security-and-authentication"></a>Seguridad y autenticación
Cada vez que se publican o consumen eventos de Event Hubs para Kafka, el cliente intenta acceder a los recursos de Event Hubs. Quiere asegurarse de que se accede a los recursos mediante una entidad autorizada. Al usar el protocolo Apache Kafka con los clientes, puede establecer la configuración para la autenticación y el cifrado mediante los mecanismos de SASL. El empleo de Event Hubs para Kafka exige cifrado TLS (ya que todos los datos en tránsito con Event Hubs están cifrados con TLS). Para ello, especifique la opción SASL_SSL en el archivo de configuración. 

Azure Event Hubs proporciona varias opciones para autorizar el acceso a los recursos protegidos. 

- OAuth
- Firma de acceso compartido (SAS)

#### <a name="oauth"></a>OAuth
Event Hubs se integra con Azure Active Directory (Azure AD), lo que proporciona un servidor de autorización centralizado compatible con **OAuth 2.0**. Con Azure AD, puede usar el control de acceso basado en rol (RBAC) para aplicar personalización avanzada de permisos a las identidades de cliente. Puede usar esta característica con los clientes de Kafka si especifica **SASL_SSL** para el protocolo y **OAUTHBEARER** para el mecanismo. Para obtener detalles sobre los roles y los niveles de Azure para el control de acceso, consulte [Autorización del acceso con Azure AD](authorize-access-azure-active-directory.md).

```xml
bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
security.protocol=SASL_SSL
sasl.mechanism=OAUTHBEARER
sasl.jaas.config=org.apache.kafka.common.security.oauthbearer.OAuthBearerLoginModule required;
sasl.login.callback.handler.class=CustomAuthenticateCallbackHandler;
```

#### <a name="shared-access-signature-sas"></a>Firma de acceso compartido (SAS)
Event Hubs también proporciona **firmas de acceso compartido (SAS)** para el acceso delegado a recursos de Event Hubs para Kafka. La autorización del acceso mediante el mecanismo basado en tokens de OAuth 2.0 proporciona una mayor seguridad y facilidad de uso que SAS. Los roles integrados también pueden eliminar la necesidad de autorización basada en ACL, que el usuario debe mantener y administrar. Puede usar esta característica con los clientes de Kafka si especifica **SASL_SSL** para el protocolo y **PLAIN** para el mecanismo. 

```xml
bootstrap.servers=NAMESPACENAME.servicebus.windows.net:9093
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";
```

> [!NOTE]
> Al usar la autenticación de SAS con los clientes de Kafka, las conexiones establecidas no se desconectan cuando se vuelve a generar la clave SAS. 

#### <a name="samples"></a>Ejemplos 
Para leer un **tutorial** con instrucciones paso a paso para crear un centro de eventos y acceder a él mediante SAS u OAuth, consulte [Inicio rápido: Streaming de datos con Event Hubs mediante el protocolo de Kafka](event-hubs-quickstart-kafka-enabled-event-hubs.md).

Para obtener más **ejemplos** que muestran cómo usar OAuth con Event Hubs para Kafka, vea [ejemplos en GitHub](https://github.com/Azure/azure-event-hubs-for-kafka/tree/master/tutorials/oauth).

## <a name="other-event-hubs-features-available-for-kafka"></a>Otras características de Event Hubs disponibles para Kafka

La característica Event Hubs para Kafka le permite escribir con un protocolo y leer con otro, para que los productores de Kafka actuales puedan seguir publicando a través de Kafka y usted pueda agregar lectores con Event Hubs, como Azure Stream Analytics o Azure Functions. Además, las características de Event Hubs como [Capture](event-hubs-capture-overview.md) y [Recuperación ante desastres geográfica](event-hubs-geo-dr.md) también funcionan con la característica Event Hubs para Kafka.

## <a name="features-that-are-not-yet-supported"></a>Características que aún no se admiten 

A continuación, se incluye la lista de características de Kafka que aún no se admiten:

*   Transacción
*   Compresión
*   Retención basada en el tamaño
*   Compactación del registro
*   Compatibilidad con HTTP Kafka API
*   Kafka Streams

## <a name="next-steps"></a>Pasos siguientes
En este artículo se proporciona una introducción a Event Hubs para Kafka. Para más información, consulte [Guía del desarrollador de Apache Kafka para Azure Event Hubs](apache-kafka-developer-guide.md).


