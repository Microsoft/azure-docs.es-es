---
title: Preguntas más frecuentes sobre Azure Event Hubs | Microsoft Docs
description: En este artículo se proporciona una lista de las preguntas más frecuentes (P+F) acerca de Azure Event Hubs y sus respuestas.
services: event-hubs
documentationcenter: na
author: ShubhaVijayasarathy
manager: timlt
ms.service: event-hubs
ms.topic: article
ms.custom: seodec18
ms.date: 05/15/2019
ms.author: shvija
ms.openlocfilehash: acc756ac04e5127d07760746bd0178f0f6cb1d6f
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65789256"
---
# <a name="event-hubs-frequently-asked-questions"></a>Preguntas frecuentes sobre Event Hubs

## <a name="general"></a>General

### <a name="what-is-an-event-hubs-namespace"></a>¿Qué es un espacio de nombres de Event Hubs?
Un espacio de nombres es un contenedor de temas de Event Hubs y Kafka. Proporciona un [FQDN](https://en.wikipedia.org/wiki/Fully_qualified_domain_name) único. Un espacio de nombres actúa como un contenedor de aplicaciones que puede alojar varios temas de Event Hubs y Kafka. 

### <a name="when-do-i-create-a-new-namespace-vs-use-an-existing-namespace"></a>¿Cuando crea un nuevo espacio de nombres frente al uso un espacio de nombres existente?
Las asignaciones de capacidad ([unidades de procesamiento (tu)](#throughput-units)) se facturan según el nivel de espacio de nombres. Un espacio de nombres también se asocia con una región.

Es posible que desee crear un nuevo espacio de nombres en lugar de usar una existente en cada uno de los siguientes escenarios: 

- Necesita un centro de eventos asociados con una nueva región.
- Necesita un centro de eventos asociados con una suscripción diferente.
- Necesita un centro de eventos con una asignación de capacidad diferentes (es decir, la capacidad es necesario para el espacio de nombres con el centro de eventos se ha agregado podría superar el umbral de MA 40 y no quiere ir para el clúster dedicado)  

### <a name="what-is-the-difference-between-event-hubs-basic-and-standard-tiers"></a>¿Cuál es la diferencia entre los niveles Basic y Standard de Event Hubs?

El nivel Estándar de Azure Event Hubs proporciona más características de las disponibles en el nivel Básico. Estas son las características disponibles en el nivel Estándar:

* Retención de eventos más prolongada
* Más conexiones asíncronas, con un cargo por uso por encima del límite de más del número incluido
* Más de un único [grupo de consumidores](event-hubs-features.md#consumer-groups)
* [Capture](event-hubs-capture-overview.md)
* [Integración de Kafka](event-hubs-for-kafka-ecosystem-overview.md)

Para más información sobre los planes de tarifa, incluido Event Hubs dedicado, vea los [detalles de los precios de Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/).

### <a name="where-is-azure-event-hubs-available"></a>¿Dónde está disponible Azure Event Hubs?

Azure Event Hubs está disponible en todas las regiones de Azure admitidas. Para obtener una lista, visite la página [Regiones de Azure](https://azure.microsoft.com/regions/).  

### <a name="can-i-use-a-single-amqp-connection-to-send-and-receive-from-multiple-event-hubs"></a>¿Se puede usar una sola conexión AMQP para enviar y recibir en varios centros de eventos?

Sí, siempre y cuando todos los centros de eventos se encuentren en el mismo espacio de nombres.

### <a name="what-is-the-maximum-retention-period-for-events"></a>¿Cuál es el período de retención máximo para eventos?

El nivel Event Hubs estándar admite actualmente un período de retención máximo de siete días. Event Hubs no está concebido como un almacén de datos permanente. Los períodos de retención superiores a 24 horas están pensados para escenarios en los que es útil volver a reproducir un flujo de eventos en los mismos sistemas; por ejemplo, para entrenar o comprobar un nuevo modelo de aprendizaje automático con datos existentes. Si necesita conservar los mensajes más de siete días, habilite [Event Hubs Capture](event-hubs-capture-overview.md) en el centro de eventos para extraer los datos de este a la cuenta de almacenamiento o la cuenta del servicio Azure Data Lake que prefiera. Al habilita Capture se le cobrará un cargo en función de las unidades de procesamiento compradas.

### <a name="how-do-i-monitor-my-event-hubs"></a>¿Cómo puedo supervisar mi instancia de Event Hubs?
Event Hubs emite métricas exhaustivas que proporcionan el estado de los recursos a [Azure Monitor](../azure-monitor/overview.md). También permite evaluar el estado general del servicio Event Hubs, no solo en el nivel de espacio de nombres, sino también en el nivel de entidad. Obtenga información sobre la supervisión que se ofrece para [Azure Event Hubs](event-hubs-metrics-azure-monitor.md).

### <a name="what-ports-do-i-need-to-open-on-the-firewall"></a>¿Qué puertos es necesario abrir en el firewall? 
Puede usar los siguientes protocolos con Azure Service Bus para enviar y recibir mensajes:

- Advanced Message Queuing Protocol (AMQP)
- HTTP
- Apache Kafka

Consulte la siguiente tabla para los puertos de salida que se debe abrir para usar estos protocolos para comunicarse con Azure Event Hubs. 

| Protocol | Puertos | Detalles | 
| -------- | ----- | ------- | 
| AMQP | 5671 y 5672 | Consulte [Guía del protocolo AMQP](../service-bus-messaging/service-bus-amqp-protocol-guide.md) | 
| HTTP, HTTPS | 80, 443 |  |
| Kafka | 9092 | Consulte [Use centros de eventos de aplicaciones de Kafka](event-hubs-for-kafka-ecosystem-overview.md)

### <a name="what-ip-addresses-do-i-need-to-whitelist"></a>¿Qué direcciones IP es necesario en la lista blanca?
Para buscar las direcciones IP correctas a la lista blanca para las conexiones, siga estos pasos:

1. Ejecute el siguiente comando desde un símbolo del sistema: 

    ```
    nslookup <YourNamespaceName>.servicebus.windows.net
    ```
2. Anote la dirección IP devuelta en `Non-authoritative answer`. Esta dirección IP es estática. El único punto en el tiempo que se vayan a cambiar es si restaurar el espacio de nombres en un clúster distinto.

Si usa la redundancia de zona para el espacio de nombres, deberá realizar algunos pasos adicionales: 

1. En primer lugar, ejecute nslookup en el espacio de nombres.

    ```
    nslookup <yournamespace>.servicebus.windows.net
    ```
2. Anote el nombre de la **respuesta no autoritativa** sección, que se encuentra en uno de los siguientes formatos: 

    ```
    <name>-s1.servicebus.windows.net
    <name>-s2.servicebus.windows.net
    <name>-s3.servicebus.windows.net
    ```
3. Ejecute nslookup para cada uno con sufijos s1, s2 y s3 para obtener las direcciones IP de las tres instancias que se ejecutan en tres zonas de disponibilidad 

## <a name="apache-kafka-integration"></a>Integración de Apache Kafka

### <a name="how-do-i-integrate-my-existing-kafka-application-with-event-hubs"></a>¿Cómo se puede integrar una aplicación existente de Kafka con Event Hubs?
Event Hubs proporciona un punto de conexión de Kafka que las aplicaciones basadas en Apache Kafka existentes pueden usar. Basta con un cambio de configuración para poder disfrutar de la experiencia de PaaS Kafka. Proporciona una alternativa a ejecutar su propio clúster de Kafka. Event Hubs admite Apache Kafka 1.0 y versiones del cliente más recientes y funciona con marcos, herramientas y aplicaciones de Kafka existentes. Para obtener más información, consulte [Event Hubs for Kafka repo](https://github.com/Azure/azure-event-hubs-for-kafka) (Event Hubs para el repositorio de Kafka).

### <a name="what-configuration-changes-need-to-be-done-for-my-existing-application-to-talk-to-event-hubs"></a>¿Qué cambios de configuración deben realizarse para que mi aplicación existente se comunique con Event Hubs?
Para conectarse a un centro de eventos habilitado para Kafka, debe actualizar las configuraciones de cliente de Kafka. Se hace mediante la creación de un espacio de nombres de Event Hubs y la obtención de la [cadena de conexión](event-hubs-get-connection-string.md). Cambie bootstrap.servers para que señale al FQDN de Event Hubs y el puerto, a 9093. Actualice sasl.jaas.config para dirigir el cliente de Kafka al punto de conexión de Event Hubs habilitado para Kafka (que es la cadena de conexión que ha obtenido), con la autenticación correcta tal y como se muestra a continuación:

bootstrap.servers={YOUR.EVENTHUBS.FQDN}:9093 request.timeout.ms=60000 security.protocol=SASL_SSL sasl.mechanism=PLAIN sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="{YOUR.EVENTHUBS.CONNECTION.STRING}";

Ejemplo:

bootstrap.servers=dummynamespace.servicebus.windows.net:9093 request.timeout.ms=60000 security.protocol=SASL_SSL sasl.mechanism=PLAIN sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="$ConnectionString" password="Endpoint=sb://dummynamespace.servicebus.windows.net/;SharedAccessKeyName=DummyAccessKeyName;SharedAccessKey=5dOntTRytoC24opYThisAsit3is2B+OGY1US/fuL3ly=";

Nota: Si sasl.jaas.config no es una configuración compatible con su marco, busque las configuraciones que se usan para establecer el nombre de usuario y contraseña SASL y úselas en su lugar. Establezca el nombre de usuario para $ConnectionString y la contraseña para la cadena de conexión de Event Hubs.

### <a name="what-is-the-messageevent-size-for-kafka-enabled-event-hubs"></a>¿Cuál es el tamaño de mensaje o evento para Event Hubs habilitado para Kafka?
El tamaño máximo de mensaje permitido para Event Hubs habilitado para Kafka es 1 MB.

## <a name="throughput-units"></a>Unidades de procesamiento

### <a name="what-are-event-hubs-throughput-units"></a>¿Qué son las unidades de procesamiento de Event Hubs?
El rendimiento de Event Hubs define la cantidad de datos en megabytes o el número (en miles) de eventos de 1 KB que entran y salen a través de Event Hubs. Este rendimiento se mide en unidades de procesamiento (TU). Compre TU antes de empezar a usar el servicio Event Hubs. Las TU de Event Hubs se pueden seleccionar explícitamente mediante el portal o las plantillas de Resource Manager para Event Hubs. 


### <a name="do-throughput-units-apply-to-all-event-hubs-in-a-namespace"></a>¿Se aplican las unidades de procesamiento a todos los centros de eventos de un espacio de nombres?
Sí, las unidades de procesamiento (TU) se aplican a todos los centros de eventos de un espacio de nombres de Event Hubs. Significa que las TU se compran en el nivel de espacio de nombres y se comparten entre los centros de eventos de ese espacio de nombres. Cada TU da al espacio de nombres derecho a las siguientes funcionalidades:

- Hasta 1 MB por segundo de eventos de entrada (eventos enviados a un centro de eventos), pero no más de 1000 eventos de entrada, operaciones de administración o llamadas a la API de control por segundo.
- Hasta 2 MB por segundo de eventos de salida (eventos consumidos de un centro de eventos), pero no más de 4096 eventos de salida.
- Hasta 84 GB de almacenamiento de eventos (suficiente para el período de retención predeterminado de 24 horas).

### <a name="how-are-throughput-units-billed"></a>¿Cómo se facturan las unidades de procesamiento?
Las unidades de procesamiento (TU) se facturan por hora. La facturación se basa en el número máximo de unidades que se seleccionaron durante la hora en cuestión. 

### <a name="how-can-i-optimize-the-usage-on-my-throughput-units"></a>¿Cómo puedo optimizar el uso de las unidades de procesamiento?
Puede empezar con solo una unidad de procesamiento (TU) y activar el [inflado automático](event-hubs-auto-inflate.md). La función de inflado automático le permite aumentar las TU a medida que incrementa la carga y el tráfico. También puede establecer un límite superior en el número de TU.

### <a name="how-does-auto-inflate-feature-of-event-hubs-work"></a>¿Cómo funciona la característica de inflado automático de Event Hubs?
La función de inflado automático le permite escalar verticalmente las unidades de procesamiento (TU). Significa que, para empezar, puede comprar un número bajo de TU y el inflado automático escalará verticalmente las TU a medida que aumente la entrada. Le ofrece una opción rentable y el control total del número de TU que se van a administrar. Esta característica permite **solo escalar verticalmente** y puede controlar por completo la reducción vertical del número de TU mediante su actualización. 

Es recomendable empezar con un número bajo de unidades de procesamiento (TU); por ejemplo, 2 TU. Si prevé que el tráfico puede crecer hasta 15 TU, active la función de inflado automático en el espacio de nombres y establezca el límite máximo en 15 TU. Ahora podrá aumentar automáticamente las TU a medida que crezca el tráfico.

### <a name="is-there-a-cost-associated-when-i-turn-on-the-auto-inflate-feature"></a>¿Hay algún costo asociado a la activación de la característica de inflado automático?
No hay **ningún costo** asociado con esta característica. 

### <a name="how-are-throughput-limits-enforced"></a>¿Cómo se aplican los límites de procesamiento?
Si el procesamiento total de entrada o la tasa total de eventos de entrada en todos los centros de eventos de un espacio de nombres superan el número total de unidades de procesamiento permitidas, los remitentes quedan sujetos a limitaciones y reciben errores que indican que se superó la cuota de entrada.

Si el procesamiento total de salida o la tasa total de eventos de salida en todos los centros de eventos de un espacio de nombres superan el número total de unidades de procesamiento permitidas, los receptores quedan sujetos a limitaciones y reciben errores que indican que se superó la cuota de salida. Las cuotas de entrada y de salida se aplican por separado, por lo que ningún remitente puede provocar que se ralentice el consumo de eventos ni ningún receptor puede impedir que los eventos se envíen a un centro de eventos.

### <a name="is-there-a-limit-on-the-number-of-throughput-units-tus-that-can-be-reservedselected"></a>¿Hay algún límite en el número de unidades de procesamiento (TU) que se pueden reservar o seleccionar?
En una oferta multiempresa, las unidades de procesamiento pueden aumentar hasta 40 TU (se pueden seleccionar hasta 20 TU en el portal y generar una incidencia de soporte técnico para elevarlas a 40 TU en el mismo espacio de nombres). Más allá de 40 TU, Event Hubs ofrece el modelo basado en recursos o capacidad denominado **clústeres de Event Hubs dedicado**. Los clústeres dedicados se venden en unidades de capacidad (CU).

## <a name="dedicated-clusters"></a>Clústeres dedicados

### <a name="what-are-event-hubs-dedicated-clusters"></a>¿Qué son los clústeres de Event Hubs dedicado?
Los clústeres de Event Hubs dedicado ofrecen implementaciones de un único inquilino para los clientes con los requisitos más exigentes. Esta oferta construye un clúster basado en la capacidad que no está limitado por las unidades de procesamiento. Esto significa que podría utilizar el clúster para la ingesta y transmisión de los datos tal y como indica el uso de CPU y memoria del clúster. Para obtener más información, consulte [Event Hubs Dedicated clusters](event-hubs-dedicated-overview.md) (Clústeres de Event Hubs dedicado).

### <a name="how-much-does-a-single-capacity-unit-let-me-achieve"></a>¿Qué me permite realizar una única unidad de capacidad?
Para un clúster dedicado, la cantidad que se puede ingerir y transmitir depende de varios factores, como los productores, los consumidores, la velocidad a la que se ingiere y procesa, etc. 

En la tabla siguiente se muestran los resultados del banco de pruebas que hemos obtenido durante nuestras pruebas:

| Forma de la carga | Receptores | Ancho de banda de entrada| Mensajes de entrada | Ancho de banda de salida | Mensajes de salida | TU totales | TU por CU |
| ------------- | --------- | ---------------- | ------------------ | ----------------- | ------------------- | --------- | ---------- |
| Lotes de 100 x 1 KB | 2 | 400 MB/s | 400 000 mensajes/s | 800 MB/s | 800 000 mensajes/s | 400 TU | 100 TU | 
| Lotes de 10 x 10 KB | 2 | 666 MB/s | 66 600 mensajes/s | 1,33 GB/s | 133 000 mensajes/s | 666 TU | 166 TU |
| Lotes de 6 x 32 KB | 1 | 1,05 GB/s | 34 000 mensajes/s | 1,05 GB/s | 34 000 mensajes/s | 1000 TU | 250 TU |

En las pruebas, se usaron los siguientes criterios:

- Se usó un clúster de Event Hubs dedicado con cuatro unidades de capacidad (CU). 
- El centro de eventos usado para la ingesta tenía 200 particiones. 
- Dos aplicaciones receptoras recibieron los datos ingeridos de todas las particiones.

Los resultados ofrecen una idea de lo que puede lograrse con un clúster de Event Hubs dedicado. Además, un clúster dedicado incluye Event Hubs Capture habilitado para los escenarios de retención a largo plazo y de microlotes.

### <a name="how-do-i-create-an-event-hubs-dedicated-cluster"></a>¿Cómo puedo crear un clúster de Event Hubs dedicado?
Un clúster de Event Hubs dedicado se crea mediante el envío de una [solicitud de soporte técnico de aumento de cuota](https://portal.azure.com/#create/Microsoft.Support) o poniéndose en contacto con el [equipo de Event Hubs](mailto:askeventhubs@microsoft.com). Normalmente, se necesitan unas dos semanas para que el clúster esté implementado y entregado, y pueda usarlo. Este proceso es temporal hasta que un autoservicio completo esté disponible a través de Azure Portal o de plantillas de Azure Resource Manager, que tardan unas dos horas en implementar el clúster.

## <a name="best-practices"></a>Procedimientos recomendados

### <a name="how-many-partitions-do-i-need"></a>¿Cuántas particiones necesito?

El número de particiones de un centro de eventos no se puede modificar después de la configuración. Por lo tanto, es importante considerar cuántas particiones necesita antes de comenzar. 

Event Hubs está diseñado para permitir un lector de partición única por grupo de consumidores. En la mayoría de los casos de uso, el valor predeterminado de cuatro particiones es suficiente. Si desea escalar el procesamiento de eventos, tal vez desee agregar particiones adicionales. No hay ningún límite de procesamiento específico en una partición; pero el número total de unidades de procesamiento limita el procesamiento agregado en el espacio de nombres. A medida que aumenta el número de unidades de procesamiento en el espacio de nombres, puede que necesite particiones adicionales para permitir que los lectores simultáneos logren su propio procesamiento máximo.

Pero si tiene un modelo en el que su aplicación tiene afinidad con una partición determinada, aumentar el número de particiones puede que no suponga ventaja alguna. Para más información, vea [Disponibilidad y coherencia](event-hubs-availability-and-consistency.md).

## <a name="pricing"></a>Precios

### <a name="where-can-i-find-more-pricing-information"></a>¿Dónde puedo encontrar más información sobre precios?

Para obtener una completa información sobre los precios de Event Hubs, consulte los [detalles de precios de Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/).

### <a name="is-there-a-charge-for-retaining-event-hubs-events-for-more-than-24-hours"></a>¿Hay un cargo por retener eventos de Event Hubs durante más de 24 horas?

El nivel Event Hubs estándar permite períodos de retención de mensajes superiores a 24 horas, hasta un máximo de siete días. Si el tamaño del número total de eventos almacenados supera la asignación de almacenamiento para el número de unidades de procesamiento seleccionadas (84 GB por unidad de procesamiento), el tamaño que supere la asignación se cargará con la tarifa publicada de almacenamiento de blobs de Azure. La asignación de almacenamiento en cada unidad de procesamiento cubre todos los costos de almacenamiento de los períodos de retención de 24 horas (valor predeterminado), incluso aunque la unidad de procesamiento se consuma hasta la asignación de entrada máxima.

### <a name="how-is-the-event-hubs-storage-size-calculated-and-charged"></a>¿Cómo se calcula y se cobra el tamaño de almacenamiento de Event Hubs?

El tamaño total de todos los eventos almacenados, incluida la sobrecarga interna de encabezados de eventos o las estructuras de almacenamiento en disco de todos los centros de eventos, se mide a lo largo del día. Al final del día, se calcula el tamaño máximo de almacenamiento. La asignación de almacenamiento diario se calcula basándose en el número mínimo de unidades de procesamiento seleccionadas durante el día (cada unidad de procesamiento ofrece una asignación de 84 GB). Si el tamaño total supera la asignación de almacenamiento diaria calculada, el exceso de almacenamiento se factura con las tarifas de Azure Blob Storage (con la tarifa de **almacenamiento con redundancia local** ).

### <a name="how-are-event-hubs-ingress-events-calculated"></a>¿Cómo se calculan los eventos de entrada de Event Hubs?

Cada evento enviado a un centro de eventos cuenta como mensaje facturable. Un *evento de entrada* se define como una unidad de datos que es menor o igual que 64 KB. Cualquier evento que tenga un tamaño menor o igual que 64 KB se considera un evento facturable. Si el evento es mayor de 64 KB, el número de eventos facturables se calcula según el tamaño del evento, en múltiplos de 64 KB. Por ejemplo, un evento de 8 KB enviado al centro de eventos se factura como un evento, pero un mensaje de 96 KB enviado al centro de eventos se factura como dos eventos.

Los eventos consumidos en un centro de eventos, así como las operaciones de administración y las llamadas de control (como los puntos de comprobación), no se cuentan como eventos de entrada facturables, pero se acumulan hasta llegar a la asignación de unidades de procesamiento.

### <a name="do-brokered-connection-charges-apply-to-event-hubs"></a>¿Los cargos por conexión desacoplada se aplican a Event Hubs?

Los cargos de conexión se aplican solo cuando se usa el protocolo AMQP. No hay ningún cargo de conexión por el envío de eventos mediante HTTP, independientemente del número de sistemas o dispositivos emisores. Si tiene previsto usar AMQP (por ejemplo, para conseguir un flujo de eventos más eficiente o para habilitar la comunicación bidireccional en escenarios de comando y control de IoT), vea la página de [información de precios de Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/) para obtener información sobre el número de conexiones incluidas en cada nivel de servicio.

### <a name="how-is-event-hubs-capture-billed"></a>¿Cómo se factura Event Hubs Capture?

Capture se habilita cuando algún centro de eventos del espacio de nombres tiene la opción Capture habilitada. Event Hubs Capture se factura por horas y por unidad de rendimiento comprada. A medida que el número de unidades de rendimiento aumenta o disminuye, la facturación de Event Hubs Capture refleja estos cambios en incrementos de horas completas. Para más información sobre Event Hubs Capture, vea los [detalles de los precios de Event Hubs](https://azure.microsoft.com/pricing/details/event-hubs/).

### <a name="do-i-get-billed-for-the-storage-account-i-select-for-event-hubs-capture"></a>¿Se me cobrará la cuenta de almacenamiento que seleccione para Event Hubs Capture?

Capture usa la cuenta de almacenamiento que se especifique al habilitarlo en un servicio centro de eventos. Dado que esta es su cuenta de almacenamiento, los cambios efectuados en esta configuración se facturan en su suscripción a Azure.

## <a name="quotas"></a>Cuotas

### <a name="are-there-any-quotas-associated-with-event-hubs"></a>¿Hay alguna cuota asociada a los centros Event Hubs?

Para obtener una lista de todas las cuotas de los centros Event Hubs, consulte [Cuotas](event-hubs-quotas.md).

## <a name="troubleshooting"></a>solución de problemas

### <a name="why-am-i-not-able-to-create-a-namespace-after-deleting-it-from-another-subscription"></a>¿Por qué no puedo crear un espacio de nombres después de eliminarlo de otra suscripción? 
Cuando elimine un espacio de nombres de una suscripción, espere 4 horas antes de volver a crearlo con el mismo nombre en otra suscripción. En caso contrario, es posible que reciba el mensaje de error siguiente: `Namespace already exists`. 

### <a name="what-are-some-of-the-exceptions-generated-by-event-hubs-and-their-suggested-actions"></a>¿Cuáles son algunas de las excepciones generadas por los centros Event Hubs y sus acciones sugeridas?

Para obtener una lista de posibles excepciones de Event Hubs, consulte [Información general sobre excepciones](event-hubs-messaging-exceptions.md).

### <a name="diagnostic-logs"></a>Registros de diagnóstico

Event Hubs admite dos tipos de [registros de diagnóstico](event-hubs-diagnostic-logs.md): los registros de errores de Capture y los registros operativos. Ambos, se representan en JSON y se pueden activar a través de Azure Portal.

### <a name="support-and-sla"></a>SLA y soporte técnico

El soporte técnico para Event Hubs está disponible a través de los [foros de la comunidad](https://social.msdn.microsoft.com/forums/azure/home?forum=servbus). Se ofrece de forma gratuita soporte técnico para la administración de suscripciones y la facturación.

Para más información sobre nuestro SLA, consulte la página [Acuerdos de nivel de servicio](https://azure.microsoft.com/support/legal/sla/) .

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de Event Hubs, visite los vínculos siguientes:

* [Información general de Event Hubs](event-hubs-what-is-event-hubs.md)
* [Creación de un Centro de eventos](event-hubs-create.md)
* [Inflado automático de Event Hubs](event-hubs-auto-inflate.md)
