---
title: 'Novedades Notas de la versión: Azure Event Grid'
description: Obtenga información acerca de las novedades de Azure Event Grid, como, por ejemplo, las notas de la versión más recientes, los problemas conocidos, las correcciones de errores, las funcionalidades en desuso y los próximos cambios.
ms.topic: overview
ms.date: 07/23/2020
ms.openlocfilehash: 1edfa3e2bc4c8adae113b2215b7fb0483fba4c02
ms.sourcegitcommit: d7bd8f23ff51244636e31240dc7e689f138c31f0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87172709"
---
# <a name="whats-new-in-azure-event-grid"></a>Novedades de Azure Event Grid

>Reciba notificaciones para volver a visitar esta página y obtener actualizaciones; para ello, copie y pegue la dirección URL `https://docs.microsoft.com/api/search/rss?search=%22Release+notes+-+Azure+Event+Grid%22&locale=en-us` en el ![icono del lector de fuentes icono RSS](./media/whats-new/feed-icon-16x16.png) lector de fuentes.

En Azure Event Grid se realizan mejoras de forma continua. Para mantenerse al día de los avances más recientes, este artículo proporciona información acerca de los elementos siguientes:

- Versiones más recientes
- Problemas conocidos
- Corrección de errores
- Funciones obsoletas
- Planes de cambios

## <a name="600-2020-06"></a>6.0.0 (2020-06)
- Agregue compatibilidad a la nueva versión 2020-06-01 de la API de servicio de disponibilidad general.
- Las nuevas características que han pasado a tener disponibilidad general son:
    - [Asignaciones de entradas](input-mappings.md)
    - [Esquema de entrada personalizado](input-mappings.md)
    - [Esquema de eventos en la nube v10](cloud-event-schema.md)
    - [Tema de Service Bus como destino](handler-service-bus.md)
    - [Función de Azure como destino](handler-functions.md)
    - [Procesamiento por lotes de webhooks](./edge/delivery-output-batching.md)
    - [Webhook seguro (soporte técnico de Azure Active Directory)](secure-webhook-delivery.md)
    - [Filtrado de IP](configure-firewall.md)
    - [Soporte técnico del servicio Private Link](configure-private-endpoints.md)
    - [Esquema de entrega de eventos](event-schema.md)

## <a name="532-preview-2020-05"></a>5.3.2-preview (2020-05)
- Esta versión incluye correcciones de errores adicionales para mejorar la calidad.
- Como la versión 5.3.1-preview, esta versión corresponde a la versión de API 2020-04-01-preview, que incluye las siguientes funcionalidades nuevas: 
    - [Compatibilidad con el filtrado de IP al publicar eventos en dominios y temas](configure-firewall.md)
    - [Temas de asociados](partner-topics-overview.md)
    - [Temas del sistema como recursos en seguimiento en Azure Portal](system-topics.md)
    - [Entrega de evento con Managed Service Identity](managed-service-identity.md) 
    - [Soporte técnico del servicio Private Link](configure-private-endpoints.md)

## <a name="531-preview-2020-04"></a>5.3.1-versión preliminar (2020-04)
- Esta versión incluye correcciones de errores diversas para mejorar la calidad.
- Como la versión 5.3.0-preview, esta versión corresponde a la versión de API 2020-04-01-preview, que incluye las siguientes funcionalidades nuevas: 
    - [Compatibilidad con el filtrado de IP al publicar eventos en dominios y temas](configure-firewall.md)
    - [Temas de asociados](partner-topics-overview.md)
    - [Temas del sistema como recursos en seguimiento en Azure Portal](system-topics.md)
    - [Entrega de evento con Managed Service Identity](managed-service-identity.md) 
    - [Soporte técnico del servicio Private Link](configure-private-endpoints.md)

## <a name="530-preview-2020-03"></a>5.3.0-preview (2020-03)
- Incorporamos nuevas características sobre las características ya agregadas en la versión 5.2.0-preview. 
- Como la versión de 5.2.0-preview, esta versión corresponde a la versión de API 2020-04-01-Preview.
- Agrega compatibilidad con las siguientes funcionalidades nuevas: 
    - [Compatibilidad con el filtrado de IP al publicar eventos en dominios y temas](configure-firewall.md)
    - [Temas de asociados](partner-topics-overview.md)
    - [Temas del sistema como recursos en seguimiento en Azure Portal](system-topics.md)
    - [Entrega de evento con Managed Service Identity](managed-service-identity.md) 
    - [Soporte técnico del servicio Private Link](configure-private-endpoints.md)

## <a name="520-preview-2020-01"></a>5.2.0-preview (2020-01)
- Esta versión corresponde a la versión de API 2020-04-01-preview.
- Agrega compatibilidad con la siguiente funcionalidad nueva:
    - [Compatibilidad con el filtrado de IP al publicar eventos en dominios y temas](configure-firewall.md)

## <a name="500-2019-05"></a>5.0.0 (2019-05)
- Esta versión corresponde a la versión de API `2019-06-01`.
- Agrega compatibilidad con las siguientes funcionalidades nuevas:
    * [Dominios](event-domains.md)
    * Paginación y filtro de búsqueda para las operaciones de lista de recursos. Para obtener un ejemplo, consulte [Temas: Lista por suscripción](/rest/api/eventgrid/version2020-04-01-preview/topics/listbysubscription).
    * [Cola de Service Bus como destino](handler-service-bus.md)
    * [Filtrado avanzado](event-filtering.md#advanced-filtering)

## <a name="410-preview-2019-03"></a>4.1.0-preview (2019-03)
- Esta versión corresponde a la versión de API 2019-02-01-preview.
- Agrega compatibilidad con las siguientes funcionalidades nuevas:
    * Paginación y filtro de búsqueda para las operaciones de lista de recursos. Para obtener un ejemplo, consulte [Temas: Lista por suscripción](/rest/api/eventgrid/version2020-04-01-preview/topics/listbysubscription).
    * [Temas de creación y eliminación manual de dominios](how-to-event-domains.md)
    * [Cola de Service Bus como destino](handler-service-bus.md)

## <a name="400-2018-12"></a>4.0.0 (2018-12)
- Esta versión corresponde a la versión de API `2019-01-01` estable.
- Admite la disponibilidad general de las siguientes funcionalidades relacionadas con las suscripciones a eventos:
    * [Destino de la cola de mensajes fallidos](manage-event-delivery.md)
    * [Cola de Azure Storage como destino](handler-storage-queues.md)
    * [Azure Relay: Conexiones híbridas como destino](handler-relay-hybrid-connections.md)
    * [Validación manual del protocolo de enlace](webhook-event-delivery.md)
    * [Compatibilidad con directivas de reintento](delivery-and-retry.md)
- Todavía se puede acceder a las características que aún están en versión preliminar (como los [dominios de Event Grid](event-domains.md) o la [compatibilidad con filtros avanzados](event-filtering.md#advanced-filtering) mediante la versión 3.0.1-preview del SDK.

## <a name="301-preview-2018-10"></a>3.0.1-preview (2018-10)
- Dependencia de la [versión 10.0.3 del paquete NuGet de Newtonsoft](https://www.nuget.org/packages/Newtonsoft.Json/10.0.3).

## <a name="300-preview-2018-10"></a>3.0.0-preview (2018-10)
- Esta versión es un SDK de versión preliminar para las nuevas características introducidas en la versión de API 2018-09-15-preview. Esta versión incluye compatibilidad con:
    - [Dominios y temas de dominio](event-domains.md)
    - Presentación de [fecha de expiración para la suscripción de eventos](concepts.md#event-subscription-expiration)
    - Presentación de [filtrado avanzado](event-filtering.md#advanced-filtering) para suscripciones de eventos
    - La versión estable del SDK que tiene como destino la versión de API `2018-01-01` sigue existiendo como versión 1.3.0

## <a name="next-steps"></a>Pasos siguientes
