---
title: 'Azure Event Hubs: SDK cliente | Microsoft Docs'
description: En este artículo se proporciona información sobre los SDK cliente de Azure Event Hubs.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 2b75a66197b152425198b94a68ac77e392cb4c0e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87087567"
---
# <a name="azure-event-hubs---client-sdks"></a>Azure Event Hubs: SDK cliente
En este artículo se proporciona la siguiente información sobre los SDK compatibles con Azure Event Hubs: 

- Ubicación de paquete que se puede usar en las aplicaciones 
- Ubicación de GitHub donde se puede encontrar código fuente, ejemplos, archivo Léame, registro de cambios, problemas detectados y además notificar nuevos problemas 
- Vínculos a tutoriales de inicio rápido 

## <a name="client-sdks"></a>SDK de cliente
En la siguiente sección se describen todos los clientes del runtime de Azure Event Hubs disponibles en este momento. Aunque algunas de estas bibliotecas también incluyen funcionalidad de administración limitada, hay bibliotecas específicas dedicadas a las operaciones de administración. El principal objetivo de estas bibliotecas es **enviar y recibir mensajes** en un centro de eventos.

| Idioma | Paquete | Referencia | 
| -------- | ------- | --------------- | 
| . NET Standard (**más reciente** y admite .NET Core y .NET Framework) | [Azure.Messaging.EventHubs](https://www.nuget.org/packages/Azure.Messaging.EventHubs/) |<ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs)</li><li>[Tutorial](get-started-dotnet-standard-send-v2.md)</li></ul> |
|       | [Azure.Messaging.EventHubs.Processor](https://www.nuget.org/packages/Azure.Messaging.EventHubs.Processor/) | <ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor)</li><li>[Tutorial](get-started-dotnet-standard-send-v2.md)</li></ul> |
| . NET Standard (**heredado** y admite .NET Core y .NET Framework) | [Microsoft.Azure.EventHubs](https://www.nuget.org/packages/Microsoft.Azure.EventHubs/) | <ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.EventHubs)</li><li>[Tutorial](event-hubs-dotnet-standard-getstarted-send.md)</li></ul> | 
|       | [Microsoft.Azure.EventHubs.Processor](https://www.nuget.org/packages/Microsoft.Azure.EventHubs.Processor) | <ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.EventHubs.Processor)</li><li>[Tutorial](event-hubs-dotnet-standard-getstarted-send.md)</li></ul> |
| . NET Framework (**anterior**) | [WindowsAzure.Messaging](https://www.nuget.org/packages/WindowsAzure.ServiceBus/) |<ul><li>[Tutorial](event-hubs-dotnet-framework-getstarted-send.md)</li></ul> |
| Java | [azure-messaging-eventhubs](https://search.maven.org/search?q=a:azure-messaging-eventhubs) | <ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/azure-messaging-eventhubs)</li><li>[Tutorial](get-started-java-send-v2.md)</li></ul> |
|      | [azure-eventhubs](https://search.maven.org/search?q=a:azure-eventhubs) **(heredado)** | <ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/eventhubs/microsoft-azure-eventhubs)</li><li>[Tutorial](event-hubs-java-get-started-send.md)</li></ul> |
| Python |  [azure-eventhub](https://pypi.org/project/azure-eventhub/) | <ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/eventhub/azure-eventhub)</li><li>[Tutorial](get-started-python-send-v2.md)</li></ul> |
|        | [azure-eventhub-checkpointstoreblob-aio](https://pypi.org/project/azure-eventhub-checkpointstoreblob-aio/) | <ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/eventhub/azure-eventhub-checkpointstoreblob-aio)</li><li>[Tutorial](get-started-python-send-v2.md)</li></ul> |
| JavaScript | [azure/event-hubs](https://www.npmjs.com/package/@azure/event-hubs) | <ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/event-hubs)</li><li>[Tutorial](get-started-node-send-v2.md)</li></ul> |
|            | [azure/eventhubs-checkpointstore-blob](https://www.npmjs.com/package/@azure/eventhubs-checkpointstore-blob) | <ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/eventhub/eventhubs-checkpointstore-blob)</li><li>[Tutorial](get-started-node-send-v2.md)</li></ul> |
| Go | [azure-event-hubs-go](https://github.com/Azure/azure-event-hubs-go) | <ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-event-hubs-go)</li><li>[Tutorial](event-hubs-go-get-started-send.md)</li></ul> |
| C | [azure-event-hubs-c](https://github.com/Azure/azure-event-hubs-c) | <ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-event-hubs-c)</li><li>[Tutorial](event-hubs-c-getstarted-send.md)</li></ul> |

## <a name="management-sdks"></a>SDK de administración
En la tabla siguiente se listan todas las bibliotecas específicas de administración disponibles actualmente. Ninguna de estas bibliotecas contiene operaciones de runtime; además, su único propósito es **administrar entidades de Event Hubs**.

| Idioma | Paquete | Referencia | 
| -------- | ------- | --------------- | 
| .NET Standard | [Microsoft.Azure.Management.EventHub](https://www.nuget.org/packages/Microsoft.Azure.Management.EventHub) |<ul><li>[Ubicación de GitHub](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.Management.EventHub)</li><li>[Tutorial](get-started-dotnet-standard-send-v2.md)</li></ul> |


## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de Event Hubs, visite los vínculos siguientes:

* [Información general de Event Hubs](./event-hubs-about.md)
* [Creación de un Centro de eventos](event-hubs-create.md)
* [Preguntas más frecuentes sobre Event Hubs](event-hubs-faq.md)
