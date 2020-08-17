---
title: Rutas de eventos
titleSuffix: Azure Digital Twins
description: Descripción de cómo enrutar eventos en Azure Digital Twins y a otros servicios de Azure.
author: baanders
ms.author: baanders
ms.date: 3/12/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 394752792d143a3712d0bb9c50189936f23062f1
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87800473"
---
# <a name="route-events-within-and-outside-of-azure-digital-twins"></a>Enrutar eventos dentro y fuera de Azure Digital Twins

Azure Digital Twins usa **rutas de eventos** para enviar datos a los consumidores fuera del servicio. 

Durante la versión preliminar, hay dos casos principales para el envío de datos de Azure Digital Twins:
* El envío de datos desde un gemelo del grafo de Azure Digital Twins a otro. Por ejemplo, cuando cambia una propiedad de un gemelo digital, puede que desee notificar y actualizar otro gemelo digital en consecuencia.
* El envío de datos a servicios de datos descendentes, para almacenamiento o procesamiento adicional (también conocido como *salida de datos*). Por ejemplo,
  - Un hospital puede querer enviar datos de eventos de Azure Digital Twins a [Time Series Insights (TSI)](../time-series-insights/time-series-insights-update-overview.md), para registrar los datos de series temporales de eventos relacionados con el lavado de manos para análisis masivo.
  - Una empresa que ya use [Azure Maps](../azure-maps/about-azure-maps.md) podría querer usar Azure Digital Twins para mejorar su solución. Puede habilitar con rapidez un mapa de Azure después de configurar Azure Digital Twins, incorporar las entidades del mapa de Azure a Azure Digital Twins como [gemelos digitales](concepts-twins-graph.md) en el grafo de gemelos, o ejecutar consultas eficaces que aprovechen los datos de Azure Maps y Azure Digital Twins conjuntamente.

Las rutas de eventos se utilizan para ambos escenarios.

## <a name="about-event-routes"></a>Acerca de las rutas de eventos

Una ruta de eventos permite enviar datos de eventos desde gemelos digitales en Azure Digital Twins a puntos de conexión definidos por el usuario en las suscripciones. Actualmente se admiten tres servicios de Azure para puntos de conexión: [Centro de eventos](../event-hubs/event-hubs-about.md), [Event Grid](../event-grid/overview.md) y [Service Bus](../service-bus-messaging/service-bus-messaging-overview.md). Cada uno de estos servicios de Azure puede estar conectado a otros servicios y actuar como intermediario, enviando datos a destinos finales, como TSI o Azure Maps, para cualquier procesamiento que necesite.

En el diagrama siguiente se ilustra el flujo de datos de eventos a través de una solución de IoT más grande con un aspecto de Azure Digital Twins:

:::image type="content" source="media/concepts-route-events/routing-workflow.png" alt-text="Azure Digital Twins que enruta datos a través de puntos de conexión a varios servicios descendentes" border="false":::

Los destinos descendentes típicos para las rutas de eventos son recursos como TSI, Azure Maps, almacenamiento y soluciones de análisis.

### <a name="event-routes-for-internal-digital-twin-events"></a>Rutas de eventos para eventos de gemelos digitales internos

Durante la versión preliminar actual, las rutas de eventos también se usan para controlar los eventos dentro del grafo de gemelos y enviar datos de un gemelo digital a otro. Esto se hace mediante la conexión de las rutas de eventos a través de Event Grid para calcular los recursos, como [Azure Functions](../azure-functions/functions-overview.md). A continuación, estas funciones definen el modo en que los gemelos deben recibir y responder a los eventos. 

Cuando un recurso de proceso quiere modificar el grafo de gemelos en función de un evento recibido a través de la ruta de eventos, es útil que sepa con anterioridad qué gemelo quiere modificar. 

Como alternativa, el mensaje de evento también contiene el id. de gemelo de origen que envió el mensaje, por lo que el recurso de proceso puede usar consultas o atravesar relaciones para encontrar un gemelo de destino para la operación deseada. 

El recurso de proceso también debe establecer los permisos de acceso y seguridad de forma independiente.

Para recorrer el proceso de configuración de una función de Azure para procesar eventos de gemelos digitales, consulte [*Procedimiento: Configuración de una función de Azure para procesar datos*](how-to-create-azure-function.md).

## <a name="create-an-endpoint"></a>Crear un punto de conexión

Para definir una ruta de eventos, los desarrolladores deben definir primero los puntos de conexión. Un **punto de conexión** es un destino fuera de Azure Digital Twins que admite una conexión de ruta. Los destinos admitidos en la versión preliminar actual son:
* Temas personalizados de Event Grid
* Centro de eventos
* Azure Service Bus

Los puntos de conexión se configuran mediante las API de plano de control (compatibles con la [CLI de Azure Digital Twins](how-to-use-cli.md)), o a través de Azure Portal. Una definición de punto de conexión proporciona:
* El nombre del punto de conexión
* El tipo de punto de conexión (Event Grid, centro de eventos o Service Bus)
* La cadena de conexión principal o la cadena de conexión secundaria 
* La ruta de acceso del tema del punto de conexión, como *your-topic.westus2.eventgrid.azure.net*

Las API del punto de conexión que están disponibles en el plano de control son:
* Crear punto de conexión
* Obtener una lista de puntos de conexión
* Obtener punto de conexión por nombre
* Eliminar punto de conexión por nombre

## <a name="create-an-event-route"></a>Crear una ruta de eventos
 
Las rutas de eventos se crean en una aplicación cliente con la siguiente llamada a [SDK de .NET (C#)](how-to-use-apis-sdks.md): 

```csharp
await client.EventRoutes.AddAsync("<name-for-the-new-route>", new EventRoute("<endpoint-name>"));
```

* El elemento `endpoint-name` identifica un punto de conexión, como un centro de eventos, Event Grid o Service Bus. Estos puntos de conexión deben crearse en la suscripción y conectarse a Azure Digital Twins mediante las API de plano de control antes de realizar esta llamada de registro.

El objeto de ruta de eventos que se pasa a `EventRoutes.Add` también toma un [parámetro **filter**](./how-to-manage-routes-apis-cli.md#filter-events), que se puede usar para restringir los tipos de eventos que siguen esta ruta.

Las rutas también se pueden crear con la [CLI de Azure Digital Twins](how-to-use-cli.md).

### <a name="types-of-event-messages"></a>Tipos de mensajes de eventos

Los distintos tipos de eventos de IoT Hub y Azure Digital Twins generan distintos tipos de mensajes de notificación, como se describe a continuación.

[!INCLUDE [digital-twins-notifications.md](../../includes/digital-twins-notifications.md)]

## <a name="next-steps"></a>Pasos siguientes

Vea cómo configurar y administrar una ruta de eventos:
* [*Procedimiento: Administración de puntos de conexión y rutas*](how-to-manage-routes-apis-cli.md)

O bien, consulte cómo usar Azure Functions para enrutar eventos en Azure Digital Twins:
* [*Procedimiento: Configuración de una función de Azure para procesar datos*](how-to-create-azure-function.md)