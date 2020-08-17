---
title: Detección de mensajes duplicados de Azure Service Bus | Microsoft Docs
description: En este artículo se explica cómo puede detectar duplicados en mensajes de Azure Service Bus. El mensaje duplicado se puede omitir y quitar.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: dbca1b4b4f894d35835e7d37e0b4e742a2d3b917
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87083895"
---
# <a name="duplicate-detection"></a>Detección de duplicados

Si una aplicación deja de funcionar debido a un error grave inmediatamente después de enviar un mensaje y la instancia de la aplicación reiniciada cree falsamente que el mensaje anterior no se ha producido, un posterior envío provoca que el mismo mensaje aparezca en el sistema dos veces.

También puede producirse un error en el nivel de cliente o de red un momento antes y que un mensaje enviado se confirme en la cola, sin que la confirmación se devuelva al cliente correctamente. Este escenario deja al cliente en duda sobre el resultado de la operación de envío.

La detección de duplicados elimina la duda en estas situaciones, ya que permite al remitente reenviar el mismo mensaje, y la cola o el tema descartan las copias duplicadas.

La habilitación de la detección de duplicados ayuda a mantener el seguimiento del valor *MessageId* controlado por la aplicación de todos los mensajes enviados a una cola o un tema durante una ventana de tiempo específica. Si se envía algún mensaje nuevo con *MessageId* que se haya registrado durante el periodo de tiempo, se notifica como aceptado (la operación de envío se realiza correctamente), pero el mensaje recién enviado se ignora y descarta al instante. No se tiene en cuenta ninguna otra parte del mensaje que no sea *MessageId*.

El control de aplicación del identificador es esencial, ya que es lo único que permite que la aplicación enlace el valor *MessageId* a un contexto de proceso empresarial desde el que se pueda reconstruir de manera predecible en caso de error.

En el caso de un proceso empresarial en el que se envían varios mensajes en el transcurso del control de algún contexto de aplicación, el valor *MessageId* puede constar del identificador de contexto de nivel de aplicación, como un número de orden de compra, y el asunto del mensaje, por ejemplo, **12345.2017/payment**.

El valor *MessageId* siempre puede ser algún GUID, pero el anclaje del identificador al proceso empresarial produce una capacidad de repetición predecible, lo que resulta conveniente para aprovechar con efectividad la característica de detección de duplicados.

> [!NOTE]
> Si está habilitada la detección de duplicados y no se establece la clave de partición o el identificador de sesión, se usará el identificador de mensaje como clave de partición. Si tampoco se ha definido el identificador de mensaje, las bibliotecas de .NET y AMQP generarán automáticamente un identificador para el mensaje. Para más información, consulte [Uso de claves de partición](service-bus-partitioning.md#use-of-partition-keys).

## <a name="enable-duplicate-detection"></a>Habilitación de la detección de duplicados

En el portal, la característica se activa durante la creación de la entidad con la casilla **Habilitar detección de duplicados**, que está desactivada de forma predeterminada. La configuración para la creación de nuevos temas es equivalente.

![Captura de pantalla del cuadro de diálogo Crear cola con la opción Habilitar detección de duplicados seleccionada y resaltada en rojo.][1]

> [!IMPORTANT]
> No puede habilitar o deshabilitar la detección de duplicados después de crear la cola. Solo puede hacerlo en el momento de creación de la cola. 

Mediante programación, establezca el indicador con la propiedad [QueueDescription.requiresDuplicateDetection](/dotnet/api/microsoft.servicebus.messaging.queuedescription.requiresduplicatedetection#Microsoft_ServiceBus_Messaging_QueueDescription_RequiresDuplicateDetection) en la API de .NET Framework completa. Con la API de Azure Resource Manager, el valor se establece con la propiedad [queueProperties.requiresDuplicateDetection](/azure/templates/microsoft.servicebus/namespaces/queues#property-values).

El valor predeterminado del historial del tiempo de detección de duplicados es de 30 segundos para colas y temas, con un valor máximo de 7 días. Esta configuración se puede cambiar en la ventana de propiedades de colas y temas de Azure Portal.

![Captura de pantalla de la característica de Service Bus con el valor Propiedades resaltado y la opción del historial de detección de duplicados resaltada en rojo.][2]

Mediante programación, puede configurar el tamaño de la ventana de detección de duplicados durante el cual se conservan los identificadores de mensaje, mediante la propiedad [QueueDescription.DuplicateDetectionHistoryTimeWindow](/dotnet/api/microsoft.servicebus.messaging.queuedescription.duplicatedetectionhistorytimewindow#Microsoft_ServiceBus_Messaging_QueueDescription_DuplicateDetectionHistoryTimeWindow) con la API de .NET Framework completa. Con la API de Azure Resource Manager, el valor se establece con la propiedad [queueProperties.duplicateDetectionHistoryTimeWindow](/azure/templates/microsoft.servicebus/namespaces/queues#property-values).

La habilitación de la detección de duplicados y el tamaño de la ventana afecta directamente al rendimiento de la cola (y del tema), dado que todos los identificadores de mensaje registrados deben coincidir con el identificador de mensaje recién enviado.

Mantener la ventana pequeña significa que menos identificadores de mensaje se deben conservar y coincidir, y el rendimiento resulta afectado en un menor grado. En las entidades de alto rendimiento que requieren la detección de duplicados, debe mantener la ventana lo más pequeña posible.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la mensajería de Service Bus, consulte los siguientes temas:

* [Colas, temas y suscripciones de Service Bus](service-bus-queues-topics-subscriptions.md)
* [Introducción a las colas de Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Uso de temas y suscripciones de Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)

En escenarios en los que el código de cliente no puede volver a enviar un mensaje con el mismo *MessageId* que lo hizo anteriormente, es importante diseñar mensajes que se puedan volver a procesar de forma segura. En esta [entrada de blog sobre idempotencia](https://particular.net/blog/what-does-idempotent-mean) se describen diversas técnicas de cómo hacerlo.

[1]: ./media/duplicate-detection/create-queue.png
[2]: ./media/duplicate-detection/queue-prop.png
