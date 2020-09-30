---
title: Comparación del enrutamiento de mensajes y Event Grid para IoT Hub | Microsoft Docs
description: IoT Hub ofrece su propio servicio de enrutamiento de mensajes, pero también se integra con Event Grid para la publicación de eventos. Compare las dos características.
author: kgremban
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 02/20/2019
ms.author: kgremban
ms.custom: fasttrack-edit
ms.openlocfilehash: 05b582e24afddf25b7f50d4c8cd1a029684a2d4f
ms.sourcegitcommit: 70ee014d1706e903b7d1e346ba866f5e08b22761
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/11/2020
ms.locfileid: "90023812"
---
# <a name="compare-message-routing-and-event-grid-for-iot-hub"></a>Comparación del enrutamiento de mensajes y Event Grid para IoT Hub

Azure IoT Hub proporciona la capacidad de transmitir datos desde los dispositivos conectados e integrar esos datos en las aplicaciones empresariales. IOT Hub ofrece dos métodos para integrar eventos de IoT en otros servicios de Azure o en aplicaciones empresariales. En este artículo se tratan las dos características que proporcionan esta funcionalidad, de modo que pueda elegir qué opción es la más adecuada para su escenario.

[!INCLUDE [iot-hub-basic](../../includes/iot-hub-basic-partial.md)]

**[Enrutamiento de mensajes de IoT Hub](iot-hub-devguide-messages-d2c.md)** : esta característica de IoT Hub permite a los usuarios enrutar los mensajes del dispositivo a la nube a los puntos de conexión de servicio como contenedores de Azure Storage, Event Hubs, colas de Service Bus y temas de Service Bus. El enrutamiento también proporciona una funcionalidad de consulta para filtrar los datos antes de enrutarlos a los puntos de conexión. Además de los datos de telemetría del dispositivo, también puede enviar [eventos que no son de telemetría](iot-hub-devguide-messages-d2c.md#non-telemetry-events) que se pueden utilizar para desencadenar acciones. 

**Integración de IoT Hub con Event Grid**: Azure Event Grid es un servicio de enrutamiento de eventos totalmente administrado que utiliza un modelo de publicación-suscripción. Hub IoT y Event Grid trabajan juntos para [integrar eventos de IoT Hub en servicios de Azure y no de Azure](iot-hub-event-grid.md), en tiempo casi real. IoT Hub publica [eventos de dispositivo](iot-hub-event-grid.md#event-types) y eventos de telemetría.

## <a name="differences"></a>Diferencias

Aunque tanto el enrutamiento de mensajes como Event Grid habilitan la configuración de alertas, hay algunas diferencias principales entre ambos. Consulte la tabla siguiente para más detalles:

| Característica | Enrutamiento de mensajes de IoT Hub | Integración de IoT Hub con Event Grid |
| ------- | --------------- | ---------- |
| **Eventos y mensajes de dispositivo** | Sí, se puede usar el enrutamiento de mensajes con los datos de telemetría y para notificar los cambios de los dispositivos gemelos, los eventos de ciclo de vida del dispositivo (por ejemplo, cuando los dispositivos se crean, eliminan, conectan y desconectan de IoT Hub) y los eventos de cambio de gemelos digitales (parte de la [versión preliminar pública de IoT Plug and Play](../iot-pnp/overview-iot-plug-and-play.md)). | Sí, Event Grid se puede usar con los datos de telemetría y los eventos de ciclo de vida del dispositivo. Sin embargo, no se puede usar con eventos de cambio de dispositivos gemelos y eventos de cambio de gemelos digitales. |
| **Ordenación** | Sí, se mantiene el orden de los eventos.  | No, no se garantiza el orden de los eventos. | 
| **Filtros** | Filtrado enriquecido de las propiedades de aplicación de mensajes, propiedades del sistema de mensajes, cuerpo del mensaje, etiquetas de dispositivos gemelos y propiedades de dispositivos gemelos. El filtrado no se aplica a los eventos de cambios de gemelos digitales. Para obtener ejemplos, consulte la [sintaxis de consulta de enrutamiento de mensajes](iot-hub-devguide-routing-query-syntax.md). | Filtrado basado en el tipo de evento, el tipo de asunto y los atributos de cada evento. Para ver ejemplos, consulte [Filtrado de eventos en suscripciones de Event Grid](../event-grid/event-filtering.md). Al suscribirse a eventos de telemetría, puede aplicar filtros adicionales sobre los datos para filtrar por las propiedades del mensaje, el cuerpo del mensaje y el dispositivo gemelo en IoT Hub, antes de publicarlos en Event Grid. Consulte [cómo filtrar eventos](../iot-hub/iot-hub-event-grid.md#filter-events). |
| **Extremos** | <ul><li>Event Hubs</li> <li>Azure Blob Storage</li> <li>Cola de Service Bus</li> <li>Temas de Service Bus</li></ul><br>Los SKU de IoT Hub de pago (S1, S2 y S3) están limitados a 10 puntos de conexión personalizados. Se pueden crear 100 rutas por instancia de IoT Hub. | <ul><li>Azure Functions</li> <li>Azure Automation</li> <li>Event Hubs</li> <li>Logic Apps</li> <li>Storage Blob</li> <li>Temas personalizados</li> <li>Queue Storage</li> <li>Microsoft Flow</li> <li>Servicios de terceros mediante webhooks</li></ul><br>Se admiten 500 puntos de conexión por IoT Hub. Para obtener la lista más actualizada de los puntos de conexión, consulte [Controladores de eventos de Event Grid](../event-grid/overview.md#event-handlers). |
| **Costee** | El enrutamiento de mensajes no se cobra aparte. Solo se cobra una entrada de telemetría en IoT Hub. Por ejemplo, si tiene un mensaje enrutado a tres puntos de conexión diferentes, se le facturará por un único mensaje. | No hay cargos desde IoT Hub. Event Grid ofrece las primeras 100 000 operaciones al mes de forma gratuita, y luego 0,60 USD por millón de operaciones. |

## <a name="similarities"></a>Similitudes

El enrutamiento de mensajes de IoT Hub y Event Grid también tienen similitudes, algunas de las cuales se detallan en la siguiente tabla:

| Característica | Enrutamiento de mensajes de IoT Hub | Integración de IoT Hub con Event Grid |
| ------- | --------------- | ---------- |
| **Tamaño de mensaje máximo** | 256 KB, del dispositivo a la nube | 256 KB, del dispositivo a la nube |
| **Confiabilidad** | Alta: entrega cada mensaje al punto de conexión por lo menos una vez en cada ruta. Expira todos los mensajes que no se entregan dentro de una hora. | Alta: entrega cada mensaje al webhook por lo menos una vez en cada suscripción. Expira todos los eventos que no se entregan en 24 horas. | 
| **Escalabilidad** | Alta: está optimizado para admitir millones de dispositivos conectados al mismo tiempo que envían miles de millones de mensajes. | Alta: capaz de enrutar 10 000 000 eventos por segundo por región. |
| **Latency** | Baja: casi en tiempo real. | Baja: casi en tiempo real. |
| **Envío a varios puntos de conexión** | Sí, envíe un único mensaje a varios puntos de conexión. | Sí, envíe un único mensaje a varios puntos de conexión.  
| **Seguridad** | IoT Hub ofrece identidad por dispositivo y control de acceso revocable. Para más información, consulte [Control de acceso a IoT Hub](iot-hub-devguide-security.md). | Event Grid proporciona validación en tres puntos: suscripciones a eventos, publicación de eventos y entrega de eventos de webhook. Para más información, vea [Event Grid security and authentication](../event-grid/security-authentication.md) (Seguridad y autenticación de Event Grid). |

## <a name="how-to-choose"></a>Cómo elegir

El enrutamiento de mensajes IoT Hub y la integración de Hub IoT con Event Grid realizan diferentes acciones para lograr resultados similares. Ambos toman información de la solución de IoT Hub y la transmiten para que otros servicios puedan reaccionar. Entonces, ¿cómo se decide cuál usar? Tenga en cuenta las siguientes preguntas a la hora de tomar una decisión: 

* **¿Qué tipo de datos se envían a los puntos de conexión?**

   Utilice el enrutamiento de mensajes de IoT Hub cuando tenga que enviar datos de telemetría a otros servicios. El enrutamiento de mensajes también habilita la consulta de las propiedades de aplicación de mensajes y del sistema, el cuerpo del mensaje, las etiquetas de dispositivos gemelos y las propiedades de dispositivos gemelos.

   La integración de Hub de IoT con Event Grid funciona con eventos que tienen lugar en el servicio IoT Hub. Estos eventos de IoT Hub incluyen datos de telemetría y eventos de dispositivo creado, eliminado, conectado y desconectado. Al suscribirse a eventos de telemetría, puede aplicar filtros adicionales sobre los datos para filtrar por las propiedades del mensaje, el cuerpo del mensaje y el dispositivo gemelo en IoT Hub, antes de publicarlos en Event Grid. Consulte [cómo filtrar eventos](../iot-hub/iot-hub-event-grid.md#filter-events).

* **¿Qué puntos de conexión deben recibir esta información?**

   El enrutamiento de mensajes de IoT Hub admite un número limitado de puntos de conexión únicos y tipos de puntos de conexión, pero puede crear conectores para redirigir los datos y eventos a puntos de conexión adicionales. Para obtener una lista completa de los puntos de conexión admitidos, consulte la tabla de la sección anterior. 

   La integración de IoT Hub con Event Grid admite 500 puntos de conexión por IoT Hub y una gran variedad de tipos de punto de conexión. Se integra de forma nativa con Azure Functions, Logic Apps, Storage y las colas de Service Bus, y también funciona con webhooks para ampliar el envío de datos fuera del ecosistema de servicios de Azure y a aplicaciones empresariales de terceros.

* **¿Importa si los datos llegan en orden?**

   El enrutamiento de mensajes de IoT Hub mantiene el orden en el que se envían los mensajes, de modo que lleguen de la misma manera.

   Event Grid no garantiza que los puntos de conexión reciban eventos en el mismo orden en que se produjeron. En aquellos casos en los que sea importante el orden absoluto de los mensajes o en los que un consumidor necesite un identificador único de confianza para los mensajes, se recomienda usar el enrutamiento de mensajes. 

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre el [enrutamiento de mensajes de IoT Hub](iot-hub-devguide-messages-d2c.md) y [los puntos de conexión de IoT Hub](iot-hub-devguide-endpoints.md).
* Más información sobre [Azure Event Grid](../event-grid/overview.md).
* Para aprender a crear rutas de mensajes, consulte el tutorial [Procesamiento de mensajes del dispositivo a la nube de IoT Hub mediante rutas](../iot-hub/tutorial-routing.md).
* Pruebe la integración de Event Grid mediante los pasos del artículo [Envío de notificaciones por correo electrónico sobre eventos de Azure IoT Hub mediante Logic Apps](../event-grid/publish-iot-hub-events-to-logic-apps.md).
