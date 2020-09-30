---
title: Preguntas frecuentes sobre Azure Relay | Microsoft Docs
description: En este artículo se responden algunas de las preguntas más frecuentes relativas al servicio Azure Relay.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 9d967d926c6ab59e027fe4d4cf98e8418a8ff9bc
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89299293"
---
# <a name="azure-relay-faqs"></a>Preguntas frecuentes sobre Azure Relay

En este artículo se responden algunas preguntas frecuentes sobre [Azure Relay](https://azure.microsoft.com/services/service-bus/). Vea [Preguntas frecuentes de soporte técnico de Azure](https://azure.microsoft.com/support/faq/) para obtener información general sobre los precios y el soporte técnico de Azure.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="general-questions"></a>Preguntas generales
### <a name="what-is-azure-relay"></a>¿Qué es Relay de Azure?
El [servicio Azure Relay](relay-what-is-it.md) facilita sus aplicaciones híbridas, al ayudar a exponer a la nube pública, de un modo seguro, servicios que residen en una red empresarial. Puede exponer los servicios sin tener que abrir una conexión de firewall y sin requerir cambios intrusivos en la infraestructura de red corporativa.

### <a name="what-is-a-relay-namespace"></a>¿Qué es el espacio de nombres de Relay?
Un [espacio de nombres](relay-create-namespace-portal.md) es un contenedor de ámbito que puede usar para hacer referencia a recursos de Relay dentro de la aplicación. Debe crear un espacio de nombres para utilizar Relay. Este es uno de los primeros pasos en la introducción.

### <a name="what-happened-to-service-bus-relay-service"></a>¿Qué ha ocurrido con el servicio Service Bus Relay?
El servicio anteriormente llamado Service Bus Relay se conoce ahora como [Azure Relay](service-bus-relay-tutorial.md). Este servicio se puede seguir usando de la manera habitual. La característica Conexiones híbridas es una versión actualizada de un servicio trasplantado desde Azure BizTalk Services. Tanto WCF Relay como Conexiones híbridas seguirán siendo admitidos en el futuro.

## <a name="pricing"></a>Precios
Esta sección responde a algunas preguntas frecuentes sobre la estructura de precios de Relay. También puede ver [Preguntas frecuentes de soporte técnico de Azure](https://azure.microsoft.com/support/faq/) para obtener información general sobre los precios de Azure. Para obtener más información sobre los precios de Relay, consulte [Precios de Service Bus][Pricing overview].

### <a name="how-do-you-charge-for-hybrid-connections-and-wcf-relay"></a>¿Cómo se cobra por Conexiones híbridas y WCF Relay?
Para obtener información completa sobre los precios de retransmisión, consulte la tabla [Conexiones híbridas y retransmisiones de WCF][Pricing overview] en la página de detalles de precios de Service Bus. Además de los precios indicados en dicha página, se le cobrará por las transferencias de datos asociadas para salidas del centro de datos en el que se aprovisiona la aplicación.

### <a name="how-am-i-billed-for-hybrid-connections"></a>¿Cómo se factura Conexiones híbridas?
Presentamos tres escenarios de facturación de ejemplo para Conexiones híbridas:

*   Escenario 1:
    *   Tiene un agente de escucha único, como una instancia del administrador de Conexiones híbridas instalado y en ejecución continua durante todo el mes.
    *   Envía 3 GB de datos a través de la conexión durante el mes. 
    *   El cargo total es de 5 dólares.
*   Escenario 2:
    *   Tiene un agente de escucha único, como una instancia del administrador de Conexiones híbridas instalado y en ejecución continua durante todo el mes.
    *   Envía 10 GB de datos a través de la conexión durante el mes.
    *   El cargo total es de 7,50 dólares. Son 5 dólares por la conexión y los primeros 5 GB más 2,50 dólares por los 5 GB de datos adicionales.
*   Escenario 3:
    *   Tiene dos instancias, A y B, del administrador de Conexiones híbridas instaladas y en ejecución continua durante todo el mes.
    *   Envía 3 GB de datos a través de la conexión A durante el mes.
    *   Envía 6 GB de datos a través de la conexión B durante el mes.
    *   El cargo total es de 10,50 dólares. Son 5 dólares por la conexión A, más 5 dólares por la conexión B, más 0,50 dólares por el sexto GB de la conexión B.

Tenga en cuenta que los precios que se utilizan en los ejemplos son aplicables únicamente durante el período de la versión preliminar de Conexiones híbridas. Los precios están sujetos a cambios en función de la disponibilidad general de Conexiones híbridas.

### <a name="how-are-hours-calculated-for-relay"></a>¿Cómo se calculan las horas de retransmisión?

WCF Relay solo está disponibles en los espacios de nombres de nivel Estándar. En cualquier otro caso, el precio y las [cuotas de conexión](../service-bus-messaging/service-bus-quotas.md) de las retransmisiones permanecen igual. Esto significa que las retransmisiones seguirán cobrándose según el número de mensajes (y no de operaciones) y las horas de retransmisión. Para obtener más información, consulte la tabla [Conexiones híbridas y retransmisiones de WCF](https://azure.microsoft.com/pricing/details/service-bus/) en la página de detalles de precios.

### <a name="what-if-i-have-more-than-one-listener-connected-to-a-specific-relay"></a>¿Qué ocurre si tengo más de un agente de escucha conectado a una retransmisión determinada?
En algunos casos, una sola retransmisión puede tener varios agentes de escucha conectados. Una retransmisión se considera abierta cuando al menos un agente de escucha de retransmisión está conectado a ella. Si agrega agentes de escucha a una retransmisión abierta, se generarán horas de retransmisión adicionales. El número de remitentes de retransmisión (clientes que invocan o envían mensajes a retransmisiones) conectados a una retransmisión no tiene ningún efecto en el cálculo de horas de retransmisión.

### <a name="how-is-the-messages-meter-calculated-for-wcf-relays"></a>¿Cómo se calcula el contador de mensajes para retransmisiones de WCF?
(**Esto solo se aplica a retransmisiones WCF. Los mensajes no son un costo en Conexiones híbridas.** )

En general, los mensajes facturables para retransmisiones se calculan utilizando el mismo método que se usa para las entidades negociadas (colas, temas y suscripciones), que se ha descrito anteriormente. Sin embargo, hay algunas diferencias importantes.

El envío de un mensaje a Azure Relay se trata como un envío de tipo "completo a través" al agente de escucha de retransmisión que recibe el mensaje. No se trata como una operación de envío a Azure Relay seguida de una entrega al agente de escucha de retransmisión. Una invocación de servicio de tipo solicitud-respuesta (de hasta 64 KB) a un agente de escucha de retransmisión producirá dos mensajes facturables: un mensaje facturable por la solicitud y un mensaje facturable por la respuesta (siempre que la respuesta sea también igual o inferior a 64 KB). Esto difiere del uso de una cola para mediar entre un cliente y un servicio. Si utiliza una cola para mediar entre un cliente y un servicio, el propio patrón de solicitud y respuesta requiere el envío de una solicitud a la cola, seguido de una eliminación de la cola y entrega desde la cola al servicio. A esto le sigue el envío de la respuesta a otra cola y la eliminación de la cola y entrega desde esa otra cola al cliente. Con las mismas suposiciones de tamaño (hasta 64 KB), el patrón de cola de mediación produce cuatro mensajes facturables. Se le facturaría dos veces el número de mensajes para implementar el mismo patrón que se logra con el uso de la retransmisión. Por supuesto, existen ventajas en el uso de colas para lograr este patrón, como la duración y la nivelación de la carga. Estas ventajas pueden justificar el gasto adicional.

Las retransmisiones que se abren mediante el enlace WCF **netTCPRelay** tratan los mensajes no como mensajes individuales, sino como un flujo de datos que fluye a través del sistema. Cuando se usa este enlace, solo el remitente y el agente de escucha tienen visibilidad sobre la trama de los mensajes individuales enviados y recibidos. Para las retransmisiones que usan el enlace **netTCPRelay**, todos los datos se tratan como un flujo para el cálculo de los mensajes facturables. En este caso, Service Bus calcula la cantidad total de datos enviados o recibidos a través de cada retransmisión individual en una base de 5 minutos. A continuación, divide la cantidad total de datos entre 64 KB para determinar el número de mensajes facturables para dicha retransmisión durante ese período de tiempo.

## <a name="quotas"></a>Cuotas
| Nombre de cuota | Ámbito |  Notas | Value |
| --- | --- | --- | --- |
| Agentes de escucha simultáneos en una retransmisión |Entidad |Las solicitudes posteriores de conexiones adicionales se rechazarán y el código que realizó la llamada recibe una excepción. |25 |
| Conexiones de retransmisión simultáneas por todos los puntos de conexión de retransmisión en un espacio de nombres de servicio |Espacio de nombres |- |5\.000 |
| Puntos de conexión de retransmisión por espacio de nombres de servicio |Espacio de nombres |- |10 000 |
| Tamaño de mensaje de las retransmisiones [NetOnewayRelayBinding](/dotnet/api/microsoft.servicebus.netonewayrelaybinding) y [NetEventRelayBinding](/dotnet/api/microsoft.servicebus.neteventrelaybinding) |Espacio de nombres |Los mensajes entrantes que superen estas cuotas se rechazarán y el código que realizó la llamada recibe una excepción. |64 KB |
| Tamaño de mensaje de las retransmisiones [HttpRelayTransportBindingElement](/dotnet/api/microsoft.servicebus.httprelaytransportbindingelement) y [NetTcpRelayBinding](/dotnet/api/microsoft.servicebus.nettcprelaybinding) |Espacio de nombres |No hay límite en el tamaño del mensaje. |Sin límite |

### <a name="does-relay-have-any-usage-quotas"></a>¿Tiene Relay cuotas de uso?
De forma predeterminada, para cualquier servicio en la nube, Microsoft establece una cuota de uso mensual agregada que se calcula en todas las suscripciones del cliente. Somos conscientes de que en ocasiones sus necesidades pueden superar estos límites. Puede ponerse en contacto con el servicio de atención al cliente en cualquier momento para que podamos conocer sus necesidades y ajustar estos límites según corresponda. Para Service Bus, las cuotas de uso agregado son las siguientes:

* 5 millardos de mensajes
* 2 millones de horas de retransmisión

Aunque nos reservamos el derecho de deshabilitar una cuenta que supera sus cuotas de uso mensuales, se proporcionará una notificación por correo electrónico y se realizarán varios intentos para ponerse en contacto con el cliente antes de llevar a cabo cualquier acción. Los clientes que superen estas cuotas siguen siendo responsables de cargos por exceso de las mismas.

### <a name="naming-restrictions"></a>Restricciones de nomenclatura
El nombre de un espacio de nombres de Relay debe tener entre 6 y 50 caracteres de longitud.

## <a name="subscription-and-namespace-management"></a>Administración de suscripción y espacio de nombres
### <a name="how-do-i-migrate-a-namespace-to-another-azure-subscription"></a>¿Cómo se migra un espacio de nombres a otra suscripción de Azure?

Para mover un espacio de nombres de una suscripción de Azure a otra, puede utilizar [Azure Portal](https://portal.azure.com) o comandos de PowerShell. Para mover un espacio de nombres a otra suscripción, el espacio de nombres ya debe estar activo. El usuario que ejecuta los comandos debe ser un usuario administrador en las suscripciones de origen y de destino.

#### <a name="azure-portal"></a>Azure Portal

Para usar Azure Portal para migrar espacios de nombres de Azure Relay de una suscripción a otra suscripción, consulte [Mover recursos a un nuevo grupo de recursos o suscripción](../azure-resource-manager/management/move-resource-group-and-subscription.md#use-the-portal). 

#### <a name="powershell"></a>PowerShell

Para usar PowerShell para mover un espacio de nombres de una suscripción de Azure a otra, utilice la siguiente secuencia de comandos. Para ejecutar esta operación, el espacio de nombres ya debe estar activo y el usuario que ejecuta los comandos de PowerShell debe ser un usuario administrador en las suscripciones de origen y destino.

```azurepowershell-interactive
# Create a new resource group in the target subscription.
Select-AzSubscription -SubscriptionId 'ffffffff-ffff-ffff-ffff-ffffffffffff'
New-AzResourceGroup -Name 'targetRG' -Location 'East US'

# Move the namespace from the source subscription to the target subscription.
Select-AzSubscription -SubscriptionId 'aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa'
$res = Find-AzResource -ResourceNameContains mynamespace -ResourceType 'Microsoft.ServiceBus/namespaces'
Move-AzResource -DestinationResourceGroupName 'targetRG' -DestinationSubscriptionId 'ffffffff-ffff-ffff-ffff-ffffffffffff' -ResourceId $res.ResourceId
```

## <a name="troubleshooting"></a>Solución de problemas
### <a name="what-are-some-of-the-exceptions-generated-by-azure-relay-apis-and-suggested-actions-you-can-take"></a>¿Cuáles son algunas de las excepciones generadas por las API de Azure Relay y sus acciones sugeridas?
Para obtener una descripción de excepciones comunes y las acciones sugeridas que puede realizar, consulte [Excepciones de Azure Relay][Relay exceptions].

### <a name="what-is-a-shared-access-signature-and-which-languages-can-i-use-to-generate-a-signature"></a>¿Qué es una firma de acceso compartido y qué lenguajes puedo usar para generar una firma?
Las firmas de acceso compartido (SAS) son un mecanismo de autenticación basado en valores hash seguros SHA-256 o en URI. Para obtener información sobre cómo generar sus propias firmas en Node.js, PHP, Python, Java, C y C#, consulte [Autenticación de Service Bus con firmas de acceso compartido][Shared Access Signatures].

### <a name="is-it-possible-to-allow-only-some-relay-endpoints"></a>¿Se pueden permitir únicamente algunos puntos de conexión de retransmisión?
Sí. El cliente de Relay establece conexiones al servicio de Azure Relay mediante nombres de dominio completos. Los clientes pueden añadir una entrada para `*.servicebus.windows.net` en firewalls compatibles con listas de aprobaciones de DNS.

## <a name="next-steps"></a>Pasos siguientes
* [Creación de un espacio de nombres](relay-create-namespace-portal.md)
* [Introducción a .NET](relay-hybrid-connections-dotnet-get-started.md)
* [Introducción a Node](relay-hybrid-connections-node-get-started.md)

[Pricing overview]: https://azure.microsoft.com/pricing/details/service-bus/
[Relay exceptions]: relay-exceptions.md
[Shared Access Signatures]: ../service-bus-messaging/service-bus-sas.md
