---
title: Migración a Azure Event Hubs para Apache Kafka
description: En este artículo se muestra cómo los consumidores y los productores que utilizan distintos protocolos (AMQP, Apache Kafka y HTTPS) pueden intercambiar eventos cuando se usa Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: cf585cdc0a43c595ec013ba78a2e3ae191c40b8d
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090661"
---
# <a name="migrate-to-azure-event-hubs-for-apache-kafka-ecosystems"></a>Migración a Azure Event Hubs para ecosistemas de Apache Kafka
Azure Event Hubs expone un punto de conexión de Apache Kafka, que le permite conectarse a Event Hubs mediante el protocolo Kafka. Al realizar cambios mínimos en una aplicación de Kafka existente, puede conectarse a Azure Event Hubs y disfrutar de las ventajas del ecosistema de Azure. Event Hubs para Kafka admite [Apache Kafka versión 1.0](https://kafka.apache.org/10/documentation.html) y posteriores.

## <a name="pre-migration"></a>Antes de la migración 

### <a name="create-an-azure-account"></a>Crear una cuenta de Azure
Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.

### <a name="create-an-event-hubs-namespace"></a>Creación de un espacio de nombres de Event Hubs
Siga las instrucciones paso a paso de [Creación de un centro de eventos](event-hubs-create.md) para crear un espacio de nombres de Event Hubs y un centro de eventos. 

### <a name="connection-string"></a>Cadena de conexión
Siga los pasos del artículo [Obtención de la cadena de conexión del portal](event-hubs-get-connection-string.md#get-connection-string-from-the-portal). Anote la cadena de conexión para usarla más adelante. 

### <a name="fully-qualified-domain-name-fqdn"></a>Nombre de dominio completo (FQDN)
Es posible que también necesite el nombre de dominio completo que señala a su espacio de nombres de Event Hubs. El nombre de dominio completo puede encontrarse en la cadena de conexión de la siguiente manera:

`Endpoint=sb://`**`mynamespace.servicebus.windows.net`**`/;SharedAccessKeyName=XXXXXX;SharedAccessKey=XXXXXX`

Si el espacio de nombres de Event Hubs se implementa en una nube no pública, el nombre de dominio puede ser distinto (por ejemplo, \*.servicebus.chinacloudapi.cn, \*.servicebus.usgovcloudapi.net o \*.servicebus.cloudapi.de).

## <a name="migration"></a>Migración 

### <a name="update-your-kafka-client-configuration"></a>Actualización de la configuración del cliente de Kafka

Para conectarse a un centro de eventos habilitado para Kafka, debe actualizar las configuraciones de cliente de Kafka. Si no encuentra la suya, pruebe a buscar dónde está establecido `bootstrap.servers` en su aplicación.

Inserte las siguientes configuraciones donde corresponda en su aplicación. Asegúrese de actualizar los valores `bootstrap.servers` y `sasl.jaas.config` para dirigir al cliente al punto de conexión de Kafka para Event Hubs con la autenticación correcta. 

```
bootstrap.servers={MYNAMESPACE}.servicebus.windows.net:9093
request.timeout.ms=60000
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{CONNECTION STRING TO YOUR NAMESPACE}";
``` 

Si `sasl.jaas.config` no es una configuración compatible con su marco, busque las configuraciones que se usan para establecer el nombre de usuario y contraseña de SASL y úselas en su lugar. Establezca el nombre de usuario en `$ConnectionString` y la contraseña para la cadena de conexión de Event Hubs.

## <a name="post-migration"></a>Después de la migración
Ejecute la aplicación de Kafka que envía eventos a un centro de eventos. A continuación, compruebe que el centro de eventos recibe los eventos mediante Azure Portal. En la página **Información general** del espacio de nombres de Event Hubs, cambie a la vista **Mensajes** de la sección **Métricas**. Actualice la página para actualizar el gráfico. Puede tardar unos segundos en mostrar que los mensajes se han recibido. 

[![Comprobación de que el centro de eventos ha recibido los mensajes](./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png)](./media/getstarted-dotnet-standard-send-v2/verify-messages-portal.png#lightbox)


## <a name="next-steps"></a>Pasos siguientes
Para obtener más información acerca de Event Hubs y Event Hubs para Kafka, consulte los artículos siguientes:  

- [Guía de solución de problemas de Apache Kafka para Event Hubs](apache-kafka-troubleshooting-guide.md)
- [Preguntas más frecuentes: Event Hubs para Apache Kafka](apache-kafka-frequently-asked-questions.md)
- [Guía del desarrollador de Apache Kafka para Azure Event Hubs](apache-kafka-developer-guide.md)
- [Configuraciones recomendadas](apache-kafka-configurations.md)