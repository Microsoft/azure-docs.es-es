---
title: archivo de inclusión
description: archivo de inclusión
services: service-bus-messaging
author: axisc
ms.service: service-bus-messaging
ms.topic: include
ms.date: 6/9/2020
ms.author: aschhab
ms.custom: include file
ms.openlocfilehash: 9030080d0b8c8e032cb2992a62275efcdb04aabc
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87798144"
---
En la tabla siguiente se enumeran las características de Java Message Service (JMS) que Azure Service Bus admite actualmente. También se muestran las características que no son compatibles.


| Característica | API |Estado |
|---|---|---|
| Colas   | <ul> <li> JMSContext.createQueue( String queueName) </li> </ul>| **Compatible** |
| Temas   | <ul> <li> JMSContext.createTopic( String topicName) </li> </ul>| **Compatible** |
| Colas temporales |<ul> <li> JMSContext.createTemporaryQueue() </li> </ul>| **Compatible** |
| Temas temporales |<ul> <li> JMSContext.createTemporaryTopic() </li> </ul>| **Compatible** |
| Productor de mensajes /<br/> JMSProducer |<ul> <li> JMSContext.createProducer() </li> </ul>| **Compatible** |
| Exploradores de colas |<ul> <li> JMSContext.createBrowser(Queue queue) </li> <li> JMSContext.createBrowser(Queue queue, String messageSelector) </li> </ul> | **Compatible** |
| Consumidor de mensajes/ <br/> JMSConsumer | <ul> <li> JMSContext.createConsumer( Destination destination) </li> <li> JMSContext.createConsumer( Destination destination, String messageSelector) </li> <li> JMSContext.createConsumer( Destination destination, String messageSelector, boolean noLocal)</li> </ul>  <br/> Actualmente, noLocal no se admite. | **Compatible** |
| Suscripciones duraderas compartidas | <ul> <li> JMSContext.createSharedDurableConsumer(Topic topic, String name) </li> <li> JMSContext.createSharedDurableConsumer(Topic topic, String name, String messageSelector) </li> </ul>| **Compatible**|
| Suscripciones duraderas no compartidas | <ul> <li> JMSContext.createDurableConsumer(Topic topic, String name) </li> <li> createDurableConsumer(Topic topic, String name, String messageSelector, boolean noLocal) </li> </ul> <br/> Actualmente no se admite noLocal y debe establecerse en false. | **Compatible** |
| Suscripciones no duraderas compartidas |<ul> <li> JMSContext.createSharedConsumer(Topic topic, String sharedSubscriptionName) </li> <li> JMSContext.createSharedConsumer(Topic topic, String sharedSubscriptionName, String messageSelector) </li> </ul> | **Compatible** |
| Suscripciones no duraderas no compartidas |<ul> <li> JMSContext.createConsumer(Destination destination) </li> <li> JMSContext.createConsumer( Destination destination, String messageSelector) </li> <li> JMSContext.createConsumer( Destination destination, String messageSelector, boolean noLocal) </li> </ul> <br/> Actualmente no se admite noLocal y debe establecerse en false. | **Compatible** |
| Selectores de mensajes | dependen del consumidor creado | **Compatible** |
| Retraso de la entrega (mensajes programados) | <ul> <li> JMSProducer.setDeliveryDelay( long deliveryDelay) </li> </ul>|**Compatible**|
| Mensaje creado |<ul> <li> JMSContext.createMessage() </li> <li> JMSContext.createBytesMessage() </li> <li> JMSContext.createMapMessage() </li> <li> JMSContext.createObjectMessage( Serializable object) </li> <li> JMSContext.createStreamMessage() </li> <li> JMSContext.createTextMessage() </li> <li> JMSContext.createTextMessage( String text) </li> </ul>| **Compatible** |
| Distributed transactions || No compatibles |
