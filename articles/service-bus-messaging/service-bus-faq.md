---
title: Preguntas más frecuentes (FAQ) sobre Azure Service Bus | Microsoft Docs
description: En este artículo se responden algunas de las preguntas más frecuentes (P+F) relativas a Azure Service Bus.
ms.topic: article
ms.date: 07/15/2020
ms.openlocfilehash: e098b05dba25a51d5d6ef7c50a1b73730828357a
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88080820"
---
# <a name="azure-service-bus---frequently-asked-questions-faq"></a>Preguntas más frecuentes (FAQ) sobre Azure Service Bus

En este artículo se tratan algunas preguntas frecuentes sobre Microsoft Azure Service Bus. También puede consultar las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/) para obtener información general sobre los precios y el soporte técnico de Azure.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="general-questions-about-azure-service-bus"></a>Preguntas generales sobre Azure Service Bus
### <a name="what-is-azure-service-bus"></a>Qué es Azure Service Bus
[Azure Service Bus](service-bus-messaging-overview.md) es una plataforma en la nube de mensajería asincrónica que le permite enviar datos entre sistemas desacoplados. Microsoft ofrece esta característica como un servicio, lo que significa que el usuario no tendrá que hospedar su propio hardware para poder utilizarlo.

### <a name="what-is-a-service-bus-namespace"></a>¿Qué es un espacio de nombres de Service Bus?
Un [espacio de nombres](service-bus-create-namespace-portal.md) proporciona un contenedor con un ámbito para el desvío de recursos de Service Bus en la aplicación. Es necesario crear un espacio de nombres para usar Service Bus y es uno de los primeros pasos para comenzar.

### <a name="what-is-an-azure-service-bus-queue"></a>¿Qué es una cola de Azure Service Bus?
Una [cola de Service Bus](service-bus-queues-topics-subscriptions.md) es una entidad en la que se almacenan los mensajes. Las colas son útiles cuando tiene varias aplicaciones o varias partes de una aplicación distribuida que necesitan comunicarse entre sí. La cola es similar a un centro de distribución en el sentido de que se recibe múltiples productos (mensajes) que luego se envían desde esa ubicación.

### <a name="what-are-azure-service-bus-topics-and-subscriptions"></a>¿Qué son los temas y las suscripciones de Azure Service Bus?
Un tema se puede visualizar como una cola y cuando utiliza varias suscripciones, se convierte en un modelo de mensajería más enriquecido; básicamente en una herramienta de comunicación de uno a varios. Este modelo de publicación o suscripción (o *pub/sub*) permite que cuando una aplicación envía un mensaje a un tema con varias suscripciones, el mensaje lo reciban varias aplicaciones.

### <a name="what-is-a-partitioned-entity"></a>¿Qué es una entidad con particiones?
Un único agente de mensajes controla una cola o tema convencional, que se almacena en un almacén de mensajería. Las [colas o temas con particiones](service-bus-partitioning.md), que solo se admiten en los niveles de mensajería Básico y Estándar, las administran varios agentes de mensajes y se almacenan en varios almacenes de mensajería. Esta característica significa que el rendimiento general de una cola o tema particionado ya no está limitado por el rendimiento de un solo agente o almacén de mensajería. Además, una interrupción temporal de un almacén de mensajería no hace que una cola o tema con particiones deje de estar disponible.

La ordenación no está garantizada al utilizar entidades con particiones. En caso de que una partición no esté disponible, puede enviar y recibir mensajes de las otras particiones.

 No se admiten las entidades con particiones en el nivel [SKU Premium](service-bus-premium-messaging.md). 

### <a name="what-ports-do-i-need-to-open-on-the-firewall"></a>¿Qué puertos es necesario abrir en el firewall? 
Puede usar los siguientes protocolos con Azure Service Bus para enviar y recibir mensajes:

- Advanced Message Queuing Protocol (AMQP)
- Protocolo de mensajería de Service Bus (SBMP)
- HTTP

Consulte en la siguiente tabla los puertos de salida que se deben abrir para usar estos protocolos para comunicarse con Azure Event Hubs. 

| Protocolo | Puertos | Detalles | 
| -------- | ----- | ------- | 
| AMQP | 5671 y 5672 | Consulte la [guía del protocolo AMQP](service-bus-amqp-protocol-guide.md) | 
| SBMP | 9350 a 9354 | Consulte [Modo de conectividad](/dotnet/api/microsoft.servicebus.connectivitymode?view=azure-dotnet) |
| HTTP, HTTPS | 80, 443 | 

### <a name="what-ip-addresses-do-i-need-to-add-to-allow-list"></a>¿Qué direcciones IP debo agregar a la lista de permitidas?
Para buscar las direcciones IP correctas para agregar a la lista de permitidas para las conexiones, siga estos pasos:

1. Ejecute el siguiente comando desde el símbolo del sistema: 

    ```
    nslookup <YourNamespaceName>.servicebus.windows.net
    ```
2. Anote la dirección IP devuelta en `Non-authoritative answer`. Esta dirección IP es estática. El único momento en que cambiaría es si restaurara el espacio de nombres en un clúster distinto.

Si usa la redundancia de zona para el espacio de nombres, deberá realizar algunos pasos adicionales: 

1. En primer lugar, ejecute nslookup en el espacio de nombres.

    ```
    nslookup <yournamespace>.servicebus.windows.net
    ```
2. Anote el nombre de la sección **respuesta no autoritativa**, que se encuentra en uno de los siguientes formatos: 

    ```
    <name>-s1.cloudapp.net
    <name>-s2.cloudapp.net
    <name>-s3.cloudapp.net
    ```
3. Ejecute nslookup para cada uno con los sufijos s1, s2 y s3 para obtener las direcciones IP de las tres instancias que se ejecutan en tres zonas de disponibilidad. 

### <a name="where-can-i-find-the-ip-address-of-the-client-sendingreceiving-messages-tofrom-a-namespace"></a>¿Dónde puedo encontrar la dirección IP del cliente que envía o recibe mensajes hacia o desde un espacio de nombres? 
No registramos las direcciones IP de clientes que envían o reciben mensajes hacia o desde su espacio de nombres. Vuelva a generar las claves para que todos los clientes existentes no puedan autenticarse y revise la configuración de control de acceso basado en roles ([RBAC](authenticate-application.md#azure-built-in-roles-for-azure-service-bus)) para asegurarse de que solo los usuarios o las aplicaciones permitidos tienen acceso al espacio de nombres. 

Si usa un espacio de nombres **Premium**, use el [filtrado de IP](service-bus-ip-filtering.md), [puntos de conexión de servicio de red virtual](service-bus-service-endpoints.md) y [puntos de conexión privados](private-link-service.md) para limitar el acceso al espacio de nombres. 

## <a name="best-practices"></a>Procedimientos recomendados
### <a name="what-are-some-azure-service-bus-best-practices"></a>¿Cuáles son algunos de los procedimientos recomendados de Azure Service Bus?
Consulte los [procedimientos recomendados para mejorar el rendimiento mediante Service Bus][Best practices for performance improvements using Service Bus]: en este artículo se describe cómo optimizar el rendimiento al intercambiar mensajes.

### <a name="what-should-i-know-before-creating-entities"></a>¿Qué debo saber antes de crear entidades?
Las siguientes propiedades de una cola y un tema son inmutables. Tenga en cuenta esta limitación al aprovisionar las entidades, ya que estas no se pueden modificar sin crear una entidad de sustitución.

* Creación de particiones
* Sesiones
* Detección de duplicados
* Entidad exprés

## <a name="pricing"></a>Precios
En esta sección se responde a algunas preguntas frecuentes sobre la estructura de precios de Service Bus.

En el artículo [Precios y facturación de Service Bus](https://azure.microsoft.com/pricing/details/service-bus/) se explican los medidores de facturación de Service Bus. Para obtener información específica sobre las opciones de precios de Service Bus, vea [Precios de Service Bus](https://azure.microsoft.com/pricing/details/service-bus/).

También puede consultar las [Preguntas más frecuentes de soporte técnico de Microsoft Azure](https://azure.microsoft.com/support/faq/) para obtener información general sobre los precios de Azure. 

### <a name="how-do-you-charge-for-service-bus"></a>¿Cómo se cobra Service Bus?
Para obtener más información sobre los precios de Service Bus, consulte [Precios de Service Bus][Pricing overview]. Además de los precios indicados, se le cobrará por las transferencias de datos asociadas para salidas del centro de datos en el que se aprovisiona la aplicación.

### <a name="what-usage-of-service-bus-is-subject-to-data-transfer-what-isnt"></a>¿Qué uso de Service Bus está sujeto a la transferencia de datos? ¿Cuál no lo está?
Cualquier transferencia de datos dentro de una determinada región de Azure se proporciona sin cargo alguno, así como las transferencias de datos entrantes. La transferencia de datos fuera de una región está sujeta a cargos por concepto de salida; consulte [esta página](https://azure.microsoft.com/pricing/details/bandwidth/).

### <a name="does-service-bus-charge-for-storage"></a>¿Service Bus cobra por almacenamiento?
No. Service Bus no cobra por almacenamiento. Sin embargo, hay una cuota que limita la cantidad máxima de datos que pueden persistir por cola o tema. Consulte la siguiente pregunta.

### <a name="i-have-a-service-bus-standard-namespace-why-do-i-see-charges-under-resource-group-system"></a>Tengo un espacio de nombres estándar de Service Bus. ¿Por qué veo cargos en el grupo de recursos "$system"?
Azure Service Bus ha actualizado recientemente los componentes de facturación. Debido a este cambio, si tiene un espacio de nombres estándar de Service Bus, puede ver los elementos de línea del recurso "/subscriptions/<azure_subscription_id>/resourceGroups/$system/providers/Microsoft.ServiceBus/namespaces/$system" del grupo de recursos "$system".

Estos cargos representan el cargo base por suscripción de Azure que ha aprovisionado un espacio de nombres de Service Bus estándar. 

Es importante tener en cuenta que no se trata de cargos nuevos, es decir, que ya existían en el modelo de facturación anterior. El único cambio es que ahora aparecen en "$system". Esto se hace debido a las restricciones en el nuevo sistema de facturación que agrupa los cargos de nivel de suscripción, no asociados a un recurso específico, en el identificador de recurso "$system".

## <a name="quotas"></a>Cuotas

Para obtener una lista de las cuotas y los límites de Service Bus, consulte la [información general sobre cuotas de Service Bus][Quotas overview].

### <a name="how-to-handle-messages-of-size--1-mb"></a>Cómo administrar los mensajes con un tamaño superior a 1 MB
Los servicios de mensajería de Service Bus (colas y temas o suscripciones) permiten que la aplicación envíe mensajes de un tamaño de hasta 256 KB (nivel estándar) o 1 MB (nivel prémium). Si trabaja con mensajes de tamaño superior a 1 MB, use el patrón de comprobación de notificaciones que se describe en [esta entrada de blog](https://www.serverless360.com/blog/deal-with-large-service-bus-messages-using-claim-check-pattern).

## <a name="troubleshooting"></a>Solución de problemas
### <a name="why-am-i-not-able-to-create-a-namespace-after-deleting-it-from-another-subscription"></a>¿Por qué no puedo crear un espacio de nombres después de eliminarlo de otra suscripción? 
Cuando elimine un espacio de nombres de una suscripción, espere 4 horas antes de volver a crearlo con el mismo nombre en otra suscripción. En caso contrario, es posible que reciba el mensaje de error siguiente: `Namespace already exists`. 

### <a name="what-are-some-of-the-exceptions-generated-by-azure-service-bus-apis-and-their-suggested-actions"></a>¿Cuáles son algunas de las excepciones generadas por las API de Azure Service Bus y sus acciones sugeridas?
Para obtener una lista de posibles excepciones de Service Bus, consulte [Información general sobre excepciones][Exceptions overview].

### <a name="what-is-a-shared-access-signature-and-which-languages-support-generating-a-signature"></a>¿Qué es una firma de acceso compartido y qué lenguajes admiten la generación de una firma?
Firmas de acceso compartido son un mecanismo de autenticación basado en URI y valores hash seguros SHA-256. Para obtener más información acerca de cómo generar sus propias firmas en Node.js, PHP, Java, Python y C#, consulte el artículo sobre las [firmas de acceso compartido][Shared Access Signatures].

## <a name="subscription-and-namespace-management"></a>Administración de suscripción y espacio de nombres
### <a name="how-do-i-migrate-a-namespace-to-another-azure-subscription"></a>¿Cómo se migra un espacio de nombres a otra suscripción de Azure?

Puede mover un espacio de nombres de una suscripción de Azure a otra, a través de [Azure Portal](https://portal.azure.com) o con comandos de PowerShell. Para ejecutar la operación, el espacio de nombres tiene que estar ya activo. El usuario que ejecuta los comandos debe ser administrador en las suscripciones de origen y de destino.

#### <a name="portal"></a>Portal

Para usar Azure Portal para migrar espacios de nombres de Service Bus a otra suscripción, siga las instrucciones indicadas [aquí](../azure-resource-manager/management/move-resource-group-and-subscription.md#use-the-portal). 

#### <a name="powershell"></a>PowerShell

La siguiente secuencia de comandos de PowerShell mueve un espacio de nombres de una suscripción de Azure a otra. Para ejecutar esta operación, el espacio de nombres ya debe estar activo y el usuario que ejecuta los comandos de PowerShell debe ser administrador en las suscripciones de origen y destino.

```powershell
# Create a new resource group in target subscription
Select-AzSubscription -SubscriptionId 'ffffffff-ffff-ffff-ffff-ffffffffffff'
New-AzResourceGroup -Name 'targetRG' -Location 'East US'

# Move namespace from source subscription to target subscription
Select-AzSubscription -SubscriptionId 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa'
$res = Find-AzResource -ResourceNameContains mynamespace -ResourceType 'Microsoft.ServiceBus/namespaces'
Move-AzResource -DestinationResourceGroupName 'targetRG' -DestinationSubscriptionId 'ffffffff-ffff-ffff-ffff-ffffffffffff' -ResourceId $res.ResourceId
```

## <a name="next-steps"></a>Pasos siguientes
Para más información sobre Service Bus, vea los artículos siguientes:

* [Introducción a Azure Service Bus Premium (entrada de blog)](https://azure.microsoft.com/blog/introducing-azure-service-bus-premium-messaging/)
* [Introducción a Azure Service Bus Premium (Channel9)](https://channel9.msdn.com/Blogs/Subscribe/Introducing-Azure-Service-Bus-Premium-Messaging)
* [Información general de Service Bus](service-bus-messaging-overview.md)
* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)

[Best practices for performance improvements using Service Bus]: service-bus-performance-improvements.md
[Best practices for insulating applications against Service Bus outages and disasters]: service-bus-outages-disasters.md
[Pricing overview]: https://azure.microsoft.com/pricing/details/service-bus/
[Quotas overview]: service-bus-quotas.md
[Exceptions overview]: service-bus-messaging-exceptions.md
[Shared Access Signatures]: service-bus-sas.md
