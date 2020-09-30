---
title: Temas del sistema en Azure Event Grid
description: Describe los temas del sistema en Azure Event Grid.
ms.topic: conceptual
ms.date: 08/27/2020
ms.openlocfilehash: f5ca472ab5141207222987d476284813c2aacf56
ms.sourcegitcommit: 419cf179f9597936378ed5098ef77437dbf16295
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/27/2020
ms.locfileid: "89019103"
---
# <a name="system-topics-in-azure-event-grid"></a>Temas del sistema en Azure Event Grid
Un tema del sistema en Event Grid representa uno o varios eventos publicados por los servicios de Azure, como Azure Storage y Azure Event Hubs. Por ejemplo, un tema del sistema puede representar **todos los eventos de blobs** o solo los eventos de **blob creado** y **blobs eliminado** publicados para una **cuenta de almacenamiento específica**. En este ejemplo, cuando se carga un blob en la cuenta de almacenamiento, el servicio de Azure Storage publica un evento de **blob creado** en el tema del sistema en Event Grid, que luego reenvía el evento a los [suscriptores](event-handlers.md) del tema que reciben y procesan el evento. 

> [!NOTE] 
> Solo los servicios de Azure pueden publicar eventos en los temas del sistema. Por lo tanto, no se obtiene un punto de conexión ni claves de acceso que se pueden usar para publicar eventos como en el caso de los temas o dominios personalizados.

## <a name="azure-services-that-support-system-topics"></a>Servicios de Azure que admiten temas del sistema
Esta es la lista actual de los servicios de Azure que admiten la creación de temas del sistema.

- [Azure App Configuration](event-schema-app-configuration.md)
- [Azure App Service](event-schema-app-service.md)
- [Azure Blob Storage](event-schema-blob-storage.md)
- [Azure Container Registry](event-schema-container-registry.md)
- [Azure Event Hubs](event-schema-event-hubs.md)
- [Azure IoT Hub](event-schema-iot-hub.md)
- [Azure Key Vault](event-schema-key-vault.md)
- [Azure Machine Learning](event-schema-machine-learning.md)
- [Azure Maps](event-schema-azure-maps.md)
- [Azure Media Services](event-schema-media-services.md)
- [Grupos de recursos de Azure](event-schema-resource-groups.md)
- [Azure Service Bus](event-schema-service-bus.md)
- [Azure SignalR](event-schema-azure-signalr.md)
- [Suscripciones de Azure](event-schema-subscriptions.md)

## <a name="system-topics-as-azure-resources"></a>Temas del sistema como recursos de Azure
Anteriormente, un tema del sistema estaba implícito y no se exponía para simplificar. Los temas del sistema ahora están visibles como recursos de Azure y proporcionan las siguientes funcionalidades:

- [Ver los temas del sistema en Azure Portal](create-view-manage-system-topics.md#view-all-system-topics)
- Exportar plantillas de Resource Manager para los temas del sistema y las suscripciones de eventos en Azure Portal
- [Configurar registros de diagnóstico para los temas del sistema](enable-diagnostic-logs-topic.md#enable-diagnostic-logs-for-a-system-topic)
- Configurar alertas sobre los errores de publicación y entrega 

## <a name="lifecycle-of-system-topics"></a>Ciclo de vida de los temas del sistema
Puede crear un tema del sistema de dos maneras: 

- Cree una [suscripción de eventos en un recurso de Azure como recurso de extensión](/rest/api/eventgrid/version2020-06-01/eventsubscriptions/createorupdate), que crea automáticamente un tema del sistema con un nombre en el formato: `<Azure resource name>-<GUID>`. El tema del sistema que se creó de esta manera se elimina automáticamente cuando se elimine la última suscripción de eventos para el tema. 
- Cree un tema del sistema para un recurso de Azure y, después, cree una suscripción de eventos para ese tema del sistema. Cuando use este método, puede especificar un nombre para el tema del sistema. El tema del sistema no se elimina automáticamente cuando se elimina la última suscripción de eventos. Debe eliminarlo manualmente. 

    Cuando se usa Azure Portal, siempre se aplica este método. Cuando se crea una suscripción de eventos mediante la página [**Eventos** de un recurso de Azure](blob-event-quickstart-portal.md#subscribe-to-the-blob-storage), primero se crea el tema del sistema y, a continuación, se crea la suscripción para el tema. Puede crear explícitamente un tema del sistema primero mediante la [página **Temas del sistema de Event Grid**](create-view-manage-system-topics.md#create-a-system-topic) y, a continuación, puede crear una suscripción para ese tema. 

Cuando se usa la [CLI](create-view-manage-system-topics-cli.md), [REST](/rest/api/eventgrid/version2020-06-01/eventsubscriptions/createorupdate) o una [plantilla de Azure Resource Manager](create-view-manage-system-topics-arm.md), puede elegir cualquiera de los métodos anteriores. Se recomienda crear primero un tema del sistema y, a continuación, crear una suscripción en el tema, ya que esta es la forma más reciente para crear temas del sistema.

Se producirá un error en la creación del tema del sistema si ha configurado las directivas de Azure de forma que el servicio Event Grid no pueda crear al tema. Por ejemplo, puede tener una directiva que permita la creación de solo determinados tipos de recursos (por ejemplo: Azure Storage, Azure Event Hubs, etc.) en la suscripción. 

## <a name="location-and-resource-group-for-a-system-topic"></a>Ubicación y grupo de recursos para un tema del sistema
En el caso de los orígenes del evento de Azure que se encuentran en una región o ubicación específica, el tema del sistema se crea en la misma ubicación que el origen del evento de Azure. Por ejemplo, si crea una suscripción de evento para un almacenamiento de blobs de Azure en la región Este de EE. UU., el tema del sistema se crea en Este de EE. UU. En el caso de los orígenes del evento globales de Azure, como las suscripciones y grupos de recursos de Azure o Azure Maps, Event Grid crea el tema del sistema en una ubicación **global**. 

En general, el tema del sistema se crea en el mismo grupo de recursos en el que se encuentra el origen del evento de Azure. En el caso de las suscripciones de evento creadas en el ámbito de la suscripción de Azure, el tema del sistema se crea en el grupo de recursos **Default-EventGrid** en la región **Oeste de EE. UU. 2**. Si el grupo de recursos no existe, Azure Event Grid lo crea antes de crear el tema del sistema. 

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes: 

- [Creación, visualización y administración de temas del sistema mediante Azure Portal](create-view-manage-system-topics.md)
- [Creación, visualización y administración de temas del sistema de Event Grid mediante la CLI de Azure](create-view-manage-system-topics-cli.md)
- [Creación de temas del sistema de Event Grid mediante plantillas de Azure Resource Manager](create-view-manage-system-topics-arm.md)
