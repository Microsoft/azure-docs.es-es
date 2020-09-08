---
title: 'Disponibilidad y coherencia: Azure Event Hubs | Microsoft Docs'
description: Cómo proporcionar el máximo nivel de disponibilidad y coherencia con Azure Event Hubs mediante el uso de particiones.
ms.topic: article
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: 774332b8f2d5c336f1a22d717516ae35a62b341f
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89000641"
---
# <a name="availability-and-consistency-in-event-hubs"></a>Disponibilidad y coherencia en Event Hubs

## <a name="overview"></a>Información general
Azure Event Hubs usa un [modelo de creación de particiones](event-hubs-scalability.md#partitions) para mejorar la disponibilidad y paralelización dentro de un solo centro de eventos. Por ejemplo, si un centro de eventos tiene cuatro particiones y una de ellas se mueve de un servidor a otro en una operación de equilibrio de carga, se puede enviar y recibir desde las otras tres. Además, tener un mayor número de particiones permite que más lectores simultáneos procesen los datos, lo que mejora el rendimiento agregado. Conocer las implicaciones de la creación de particiones y la ordenación de un sistema distribuido es un aspecto fundamental del diseño de soluciones.

Para ayudar a explicar el equilibrio entre ordenación y disponibilidad, consulte el [teorema CAP](https://en.wikipedia.org/wiki/CAP_theorem), que también se conoce como teorema de Brewer. Dicho teorema trata la elección entre coherencia, disponibilidad y tolerancia a la partición. Indica que para los sistemas con particiones por red siempre hay correlación entre coherencia y disponibilidad.

El teorema de Brewer define la coherencia y la disponibilidad de la forma siguiente:
* Tolerancia a la partición: la capacidad de un sistema de procesamiento de datos de continuar procesando datos aunque se produzcan errores en una partición.
* Disponibilidad: un nodo sin error devuelve una respuesta razonable en un plazo prudente (sin errores ni tiempos de espera).
* Coherencia: se garantiza que una lectura devuelva la última escritura de un cliente determinado.

## <a name="partition-tolerance"></a>Tolerancia a la partición
Event Hubs se basa en un modelo de datos con particiones. Se puede configurar el número de particiones en el centro de eventos durante la instalación, pero no se puede cambiar este valor más adelante. Puesto que se deben utilizar particiones con Event Hubs, debe tomar una decisión con respecto a la disponibilidad y la coherencia de la aplicación.

## <a name="availability"></a>Disponibilidad
La manera más sencilla de empezar a trabajar con Event Hubs es usar el comportamiento predeterminado. 

#### <a name="azuremessagingeventhubs-500-or-later"></a>[Azure.Messaging.EventHubs (5.0.0 o posterior)](#tab/latest)
Si crea un objeto **[EventHubProducerClient](/dotnet/api/azure.messaging.eventhubs.producer.eventhubproducerclient?view=azure-dotnet)** y usa el método **[SendAsync](/dotnet/api/azure.messaging.eventhubs.producer.eventhubproducerclient.sendasync?view=azure-dotnet)** , los eventos se distribuyen automáticamente entre las particiones del centro de eventos. Este comportamiento permite disfrutar del máximo tiempo de actividad.

#### <a name="microsoftazureeventhubs-410-or-earlier"></a>[Microsoft.Azure.EventHubs (4.1.0 o anterior)](#tab/old)
Si crea un nuevo objeto **[EventHubClient](/dotnet/api/microsoft.azure.eventhubs.eventhubclient)** y usa el método **[Send](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.sendasync?view=azure-dotnet#Microsoft_Azure_EventHubs_EventHubClient_SendAsync_Microsoft_Azure_EventHubs_EventData_)** , los eventos se distribuyen automáticamente entre las particiones del centro de eventos. Este comportamiento permite disfrutar del máximo tiempo de actividad.

---

Para los casos de uso que requieren el máximo tiempo de actividad, se prefiere este modelo.

## <a name="consistency"></a>Coherencia
En algunos escenarios, el orden de los eventos puede ser importante. Por ejemplo, puede que prefiera el sistema back-end para procesar un comando de actualización antes que un comando de eliminación. En este caso, puede establecer la clave de partición en un evento, o usar un objeto `PartitionSender` (si usa la biblioteca antigua Microsoft.Azure.Messaging) para enviar eventos solo a una determinada partición. De esta forma, se garantiza que, cuando se lean eventos de la partición, la lectura siga un orden. Si usa la biblioteca **Azure.Messaging.EventHubs** y quiere obtener más información, vea [Migración de código de PartitionSender a EventHubProducerClient para publicar eventos en una partición](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md#migrating-code-from-partitionsender-to-eventhubproducerclient-for-publishing-events-to-a-partition).

#### <a name="azuremessagingeventhubs-500-or-later"></a>[Azure.Messaging.EventHubs (5.0.0 o posterior)](#tab/latest)

```csharp
var connectionString = "<< CONNECTION STRING FOR THE EVENT HUBS NAMESPACE >>";
var eventHubName = "<< NAME OF THE EVENT HUB >>";

await using (var producerClient = new EventHubProducerClient(connectionString, eventHubName))
{
    var batchOptions = new CreateBatchOptions() { PartitionId = "my-partition-id" };
    using EventDataBatch eventBatch = await producerClient.CreateBatchAsync(batchOptions);
    eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes("First")));
    eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes("Second")));
    
    await producerClient.SendAsync(eventBatch);
}
```

#### <a name="microsoftazureeventhubs-410-or-earlier"></a>[Microsoft.Azure.EventHubs (4.1.0 o anterior)](#tab/old)

```csharp
var connectionString = "<< CONNECTION STRING FOR THE EVENT HUBS NAMESPACE >>";
var eventHubName = "<< NAME OF THE EVENT HUB >>";

var connectionStringBuilder = new EventHubsConnectionStringBuilder(connectionString){ EntityPath = eventHubName }; 
var eventHubClient = EventHubClient.CreateFromConnectionString(connectionStringBuilder.ToString());
PartitionSender partitionSender = eventHubClient.CreatePartitionSender("my-partition-id");
try
{
    EventDataBatch eventBatch = partitionSender.CreateBatch();
    eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes("First")));
    eventBatch.TryAdd(new EventData(Encoding.UTF8.GetBytes("Second")));

    await partitionSender.SendAsync(eventBatch);
}
finally
{
    await partitionSender.CloseAsync();
    await eventHubClient.CloseAsync();
}
```

---

Con esta configuración, tenga en cuenta que si la partición concreta a la que se realiza el envío no se encuentra disponible, recibirá una respuesta de error. Como punto de comparación, si no tiene una afinidad para una sola partición, el servicio Event Hubs envía el evento a la siguiente partición disponible.

Una posible solución para garantizar el orden, mientras también se maximiza el tiempo de actividad, sería agregar eventos como parte de la aplicación de procesamiento de eventos. La manera más fácil de lograr esto es marcar el evento con una propiedad de número de secuencia personalizada. El código siguiente muestra un ejemplo:

#### <a name="azuremessagingeventhubs-500-or-later"></a>[Azure.Messaging.EventHubs (5.0.0 o posterior)](#tab/latest)

```csharp
// create a producer client that you can use to send events to an event hub
await using (var producerClient = new EventHubProducerClient(connectionString, eventHubName))
{
    // get the latest sequence number from your application
    var sequenceNumber = GetNextSequenceNumber();

    // create a batch of events 
    using EventDataBatch eventBatch = await producerClient.CreateBatchAsync();

    // create a new EventData object by encoding a string as a byte array
    var data = new EventData(Encoding.UTF8.GetBytes("This is my message..."));

    // set a custom sequence number property
    data.Properties.Add("SequenceNumber", sequenceNumber);

    // add events to the batch. An event is a represented by a collection of bytes and metadata. 
    eventBatch.TryAdd(data);

    // use the producer client to send the batch of events to the event hub
    await producerClient.SendAsync(eventBatch);
}
```

#### <a name="microsoftazureeventhubs-410-or-earlier"></a>[Microsoft.Azure.EventHubs (4.1.0 o anterior)](#tab/old)
```csharp
// Create an Event Hubs client
var client = new EventHubClient(connectionString, eventHubName);

//Create a producer to produce events
EventHubProducer producer = client.CreateProducer();

// Get the latest sequence number from your application 
var sequenceNumber = GetNextSequenceNumber();

// Create a new EventData object by encoding a string as a byte array
var data = new EventData(Encoding.UTF8.GetBytes("This is my message..."));

// Set a custom sequence number property
data.Properties.Add("SequenceNumber", sequenceNumber);

// Send single message async
await producer.SendAsync(data);
```
---

Este ejemplo envía el evento a una de las particiones disponibles en el centro de eventos y establece el número de secuencia correspondiente a partir de la aplicación. Esta solución requiere que la aplicación de procesamiento conserve el estado, pero proporciona a los remitentes un punto de conexión con más probabilidades de estar disponible.

## <a name="next-steps"></a>Pasos siguientes
Para más información acerca de Event Hubs, visite los vínculos siguientes:

* [Información general sobre el servicio Event Hubs](./event-hubs-about.md)
* [Creación de un centro de eventos](event-hubs-create.md)
