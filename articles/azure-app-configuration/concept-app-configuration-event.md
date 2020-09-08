---
title: Reacción ante los eventos de pares clave-valor de Azure App Configuration
description: Use Azure Event Grid para suscribirse a eventos de App Configuration, que permiten a las aplicaciones reaccionar a los cambios en los pares clave-valor sin necesidad de código complicado.
services: azure-app-configuration,event-grid
author: jimmyca
ms.author: jimmyca
ms.date: 02/20/2020
ms.topic: article
ms.service: azure-app-configuration
ms.openlocfilehash: ae3417f991c0d810d8946cdaf358218ebbe4f6a5
ms.sourcegitcommit: 02ca0f340a44b7e18acca1351c8e81f3cca4a370
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/19/2020
ms.locfileid: "88590037"
---
# <a name="reacting-to-azure-app-configuration-events"></a>Reacción ante los eventos de Azure App Configuration

Los eventos de Azure App Configuration permiten a las aplicaciones reaccionar ante los cambios en los pares clave-valor. Esto se consigue sin necesidad de código complejo ni de servicios de sondeo costosos e ineficientes. En su lugar, se insertan eventos mediante [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) a los suscriptores como [Azure Functions](https://azure.microsoft.com/services/functions/), [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/) o incluso su propio cliente de escucha http. De forma crítica, solo paga por lo que utiliza.

Los eventos de Azure App Configuration se envían a Azure Event Grid, que proporciona servicios de entrega confiables para sus aplicaciones mediante directivas de reintento enriquecidas y la entrega de mensajes fallidos. Para más información, consulte [Entrega y reintento de entrega de mensajes de Event Grid](https://docs.microsoft.com/azure/event-grid/delivery-and-retry).

Entre los escenarios comunes de eventos de App Configuration se incluyen la actualización de la configuración de la aplicación, el desencadenamiento de implementaciones o cualquier flujo de trabajo de configuración. Cuando se realizan pocos cambios en el escenario, pero se requiere una respuesta inmediata, la arquitectura basada en eventos puede ser especialmente eficaz.

En el artículo de [Enrutamiento de eventos de Azure App Configuration a un punto de conexión web personalizado: CLI](./howto-app-configuration-event.md) puede encontrar un ejemplo rápido. 

![Modelo de Event Grid](./media/event-grid-functional-model.png)

## <a name="available-azure-app-configuration-events"></a>Eventos de Azure App Configuration disponibles
Event Grid usa las [suscripciones a eventos](../event-grid/concepts.md#event-subscriptions) para enrutar los mensajes de eventos a los suscriptores. Las suscripciones a eventos de Azure App Configuration pueden incluir dos tipos de eventos:  

> |Nombre del evento|Descripción|
> |----------|-----------|
> |`Microsoft.AppConfiguration.KeyValueModified`|Se genera cuando se crea o se sustituye un par clave-valor.|
> |`Microsoft.AppConfiguration.KeyValueDeleted`|Se genera cuando se elimina un par clave-valor.|

## <a name="event-schema"></a>Esquema del evento
Los eventos de Azure App Configuration contienen toda la información necesaria para responder a cualquier cambio que se produzca en los datos. Puede identificar un evento de App Configuration porque la propiedad eventType comienza por "Microsoft.AppConfiguration". Puede encontrar información adicional acerca del uso de las propiedades de los eventos de Event Grid en [Esquema de eventos de Event Grid](../event-grid/event-schema.md).  

> |Propiedad|Tipo|Descripción|
> |-------------------|------------------------|-----------------------------------------------------------------------|
> |topic|string|Identificador completo de Azure Resource Manager de la instancia de App Configuration que emite el evento.|
> |subject|string|Dirección URL del par clave-valor asunto del evento.|
> |eventTime|string|Fecha y hora en que se generó el evento, en formato ISO 8601.|
> |eventType|string|"Microsoft.AppConfiguration.KeyValueModified" o "Microsoft.AppConfiguration.KeyValueDeleted".|
> |Identificador|string|Identificador único de este evento.|
> |dataVersion|string|Versión del esquema del objeto de datos.|
> |metadataVersion|string|Versión del esquema de propiedades de nivel superior.|
> |datos|object|Recopilación de datos de evento específicos de Azure App Configuration.|
> |data.key|string|Clave del valor clave que se ha modificado o eliminado.|
> |data.label|string|Etiqueta, si procede, del valor clave que se ha modificado o eliminado.|
> |data.etag|string|Para `KeyValueModified`, la etag del nuevo valor clave. Para `KeyValueDeleted`, la etag del valor clave que se ha eliminado.|

Este es un ejemplo de evento KeyValueModified:
```json
[{
  "id": "84e17ea4-66db-4b54-8050-df8f7763f87b",
  "topic": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testrg/providers/microsoft.appconfiguration/configurationstores/contoso",
  "subject": "https://contoso.azconfig.io/kv/Foo?label=FizzBuzz",
  "data": {
    "key": "Foo",
    "label": "FizzBuzz",
    "etag": "FnUExLaj2moIi4tJX9AXn9sakm0"
  },
  "eventType": "Microsoft.AppConfiguration.KeyValueModified",
  "eventTime": "2019-05-31T20:05:03Z",
  "dataVersion": "1",
  "metadataVersion": "1"
}]

```

Para más información, consulte el [esquema de los eventos de Azure App Configuration](../event-grid/event-schema-app-configuration.md).

## <a name="practices-for-consuming-events"></a>Prácticas para consumir eventos
Las aplicaciones que administran los eventos de App Configuration deben seguir estos procedimientos recomendados:
> [!div class="checklist"]
> * Se pueden configurar varias suscripciones para enrutar los eventos al mismo controlador de eventos, por lo que no hay que asumir que los eventos proceden de un origen determinado. En su lugar, consulte el tema del mensaje para asegurarse de cuál es la instancia de App Configuration que envía el evento.
> * Compruebe el eventType y no asuma que todos los eventos que reciba van a ser de los tipos que espera.
> * Utilice los campos de la etiqueta de entidad para saber si la información sobre los objetos está aún actualizada.  
> * Utilice los campos del secuenciador para conocer el orden de los eventos en cualquier objeto determinado.
> * Utilice el campo del asunto para acceder al par clave-valor que se ha modificado.


## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre Event Grid y pruebe los eventos de Azure App Configuration:

- [Una introducción a Azure Event Grid](../event-grid/overview.md)
- [Enrutamiento de eventos de Azure App Configuration a un punto de conexión web personalizado](./howto-app-configuration-event.md)