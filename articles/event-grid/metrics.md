---
title: Métricas compatibles con Azure Event Grid
description: En este artículo se proporcionan las métricas de Azure Monitor admitidas por el servicio Azure Event Grid.
ms.topic: conceptual
ms.date: 08/13/2020
ms.openlocfilehash: 06c43b1990efc977cae33ced3f66f02e2de0b9c4
ms.sourcegitcommit: 152c522bb5ad64e5c020b466b239cdac040b9377
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88225179"
---
# <a name="metrics-supported-by-azure-event-grid"></a>Métricas compatibles con Azure Event Grid
En este artículo se proporcionan listas de métricas de Event Grid clasificadas por espacios de nombres. 

## <a name="microsofteventgriddomains"></a>Microsoft.EventGrid/domains

|Métrica|¿Se puede exportar con la configuración de diagnóstico?|Nombre de métrica para mostrar|Unidad|Tipo de agregación|Descripción|Dimensions|
|---|---|---|---|---|---|---|
|DeadLetteredCount|Sí|Eventos en la cola de mensajes fallidos|Count|Total|Número total de eventos en la cola de eventos fallidos que coinciden con esta suscripción de eventos|Topic, EventSubscriptionName, DomainEventSubscriptionName, DeadLetterReason|
|DeliveryAttemptFailCount|No|Eventos de error en la entrega|Count|Total|Número total de eventos que no se han podido entregar a esta suscripción de eventos|Topic, EventSubscriptionName, DomainEventSubscriptionName, Error, ErrorType|
|DeliverySuccessCount|Sí|Eventos entregados|Count|Total|Número total de eventos que se han entregado a esta suscripción de eventos|Topic, EventSubscriptionName, DomainEventSubscriptionName|
|DestinationProcessingDurationInMs|No|Duración del procesamiento de destino|Milisegundos|Average|Duración del procesamiento de destino en milisegundos|Topic, EventSubscriptionName, DomainEventSubscriptionName|
|DroppedEventCount|Sí|Eventos quitados|Count|Total|Número total de eventos quitados que coinciden con esta suscripción de eventos|Topic, EventSubscriptionName, DomainEventSubscriptionName, DropReason|
|MatchedEventCount|Sí|Eventos coincidentes|Count|Total|Número total de eventos que coinciden con esta suscripción de eventos|Topic, EventSubscriptionName, DomainEventSubscriptionName|
|PublishFailCount|Sí|Eventos no publicados|Count|Total|Número total de eventos que no se han podido publicar en este tema|Topic, ErrorType, Error|
|PublishSuccessCount|Sí|Eventos publicados|Count|Total|Número total de eventos publicados en este tema|Tema|
|PublishSuccessLatencyInMs|Sí|Latencia de publicación correcta|Milisegundos|Total|Latencia de publicación correcta en milisegundos|Sin dimensiones|


## <a name="microsofteventgrideventsubscriptions"></a>Microsoft.EventGrid/eventSubscriptions

|Métrica|¿Se puede exportar con la configuración de diagnóstico?|Nombre de métrica para mostrar|Unidad|Tipo de agregación|Descripción|Dimensions|
|---|---|---|---|---|---|---|
|DeadLetteredCount|Sí|Eventos en la cola de mensajes fallidos|Count|Total|Número total de eventos en la cola de eventos fallidos que coinciden con esta suscripción de eventos|DeadLetterReason|
|DeliveryAttemptFailCount|No|Eventos de error en la entrega|Count|Total|Número total de eventos que no se han podido entregar a esta suscripción de eventos|Error, ErrorType|
|DeliverySuccessCount|Sí|Eventos entregados|Count|Total|Número total de eventos que se han entregado a esta suscripción de eventos|Sin dimensiones|
|DestinationProcessingDurationInMs|No|Duración del procesamiento de destino|Milisegundos|Average|Duración del procesamiento de destino en milisegundos|Sin dimensiones|
|DroppedEventCount|Sí|Eventos quitados|Count|Total|Número total de eventos quitados que coinciden con esta suscripción de eventos|DropReason|
|MatchedEventCount|Sí|Eventos coincidentes|Count|Total|Número total de eventos que coinciden con esta suscripción de eventos|Sin dimensiones|


## <a name="microsofteventgridextensiontopics"></a>Microsoft.EventGrid/extensionTopics

|Métrica|¿Se puede exportar con la configuración de diagnóstico?|Nombre de métrica para mostrar|Unidad|Tipo de agregación|Descripción|Dimensions|
|---|---|---|---|---|---|---|
|PublishFailCount|Sí|Eventos no publicados|Count|Total|Número total de eventos que no se han podido publicar en este tema|ErrorType, Error|
|PublishSuccessCount|Sí|Eventos publicados|Count|Total|Número total de eventos publicados en este tema|Sin dimensiones|
|PublishSuccessLatencyInMs|Sí|Latencia de publicación correcta|Milisegundos|Total|Latencia de publicación correcta en milisegundos|Sin dimensiones|
|UnmatchedEventCount|Sí|Eventos no coincidentes|Count|Total|Número total de eventos que no coinciden con ninguna de las suscripciones de eventos de este tema|Sin dimensiones|


## <a name="microsofteventgridsystemtopics"></a>Microsoft.EventGrid/systemTopics

|Métrica|¿Se puede exportar con la configuración de diagnóstico?|Nombre de métrica para mostrar|Unidad|Tipo de agregación|Descripción|Dimensions|
|---|---|---|---|---|---|---|
|DeadLetteredCount|Sí|Eventos en la cola de mensajes fallidos|Count|Total|Número total de eventos en la cola de eventos fallidos que coinciden con esta suscripción de eventos|DeadLetterReason, EventSubscriptionName|
|DeliveryAttemptFailCount|No|Eventos de error en la entrega|Count|Total|Número total de eventos que no se han podido entregar a esta suscripción de eventos|Error, ErrorType, EventSubscriptionName|
|DeliverySuccessCount|Sí|Eventos entregados|Count|Total|Número total de eventos que se han entregado a esta suscripción de eventos|EventSubscriptionName|
|DestinationProcessingDurationInMs|No|Duración del procesamiento de destino|Milisegundos|Average|Duración del procesamiento de destino en milisegundos|EventSubscriptionName|
|DroppedEventCount|Sí|Eventos quitados|Count|Total|Número total de eventos quitados que coinciden con esta suscripción de eventos|DropReason, EventSubscriptionName|
|MatchedEventCount|Sí|Eventos coincidentes|Count|Total|Número total de eventos que coinciden con esta suscripción de eventos|EventSubscriptionName|
|PublishFailCount|Sí|Eventos no publicados|Count|Total|Número total de eventos que no se han podido publicar en este tema|ErrorType, Error|
|PublishSuccessCount|Sí|Eventos publicados|Count|Total|Número total de eventos publicados en este tema|Sin dimensiones|
|PublishSuccessLatencyInMs|Sí|Latencia de publicación correcta|Milisegundos|Total|Latencia de publicación correcta en milisegundos|Sin dimensiones|
|UnmatchedEventCount|Sí|Eventos no coincidentes|Count|Total|Número total de eventos que no coinciden con ninguna de las suscripciones de eventos de este tema|Sin dimensiones|


## <a name="microsofteventgridtopics"></a>Microsoft.EventGrid/topics

|Métrica|¿Se puede exportar con la configuración de diagnóstico?|Nombre de métrica para mostrar|Unidad|Tipo de agregación|Descripción|Dimensions|
|---|---|---|---|---|---|---|
|DeadLetteredCount|Sí|Eventos en la cola de mensajes fallidos|Count|Total|Número total de eventos en la cola de eventos fallidos que coinciden con esta suscripción de eventos|DeadLetterReason, EventSubscriptionName|
|DeliveryAttemptFailCount|No|Eventos de error en la entrega|Count|Total|Número total de eventos que no se han podido entregar a esta suscripción de eventos|Error, ErrorType, EventSubscriptionName|
|DeliverySuccessCount|Sí|Eventos entregados|Count|Total|Número total de eventos que se han entregado a esta suscripción de eventos|EventSubscriptionName|
|DestinationProcessingDurationInMs|No|Duración del procesamiento de destino|Milisegundos|Average|Duración del procesamiento de destino en milisegundos|EventSubscriptionName|
|DroppedEventCount|Sí|Eventos quitados|Count|Total|Número total de eventos quitados que coinciden con esta suscripción de eventos|DropReason, EventSubscriptionName|
|MatchedEventCount|Sí|Eventos coincidentes|Count|Total|Número total de eventos que coinciden con esta suscripción de eventos|EventSubscriptionName|
|PublishFailCount|Sí|Eventos no publicados|Count|Total|Número total de eventos que no se han podido publicar en este tema|ErrorType, Error|
|PublishSuccessCount|Sí|Eventos publicados|Count|Total|Número total de eventos publicados en este tema|Sin dimensiones|
|PublishSuccessLatencyInMs|Sí|Latencia de publicación correcta|Milisegundos|Total|Latencia de publicación correcta en milisegundos|Sin dimensiones|
|UnmatchedEventCount|Sí|Eventos no coincidentes|Count|Total|Número total de eventos que no coinciden con ninguna de las suscripciones de eventos de este tema|Sin dimensiones|

## <a name="next-steps"></a>Pasos siguientes
Consulte el artículo siguiente: [Registros de diagnóstico](diagnostic-logs.md)
