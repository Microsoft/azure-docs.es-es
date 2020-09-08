---
title: Reacción ante eventos de Azure Media Services | Microsoft Docs
description: En este artículo se describe cómo usar Azure Event Grid para suscribirse a eventos de Media Services.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: conceptual
ms.date: 08/31/2020
ms.author: inhenkel
ms.openlocfilehash: 4ef6a920e9334c6e98b18d1db1abf39136c6f4e2
ms.sourcegitcommit: 58d3b3314df4ba3cabd4d4a6016b22fa5264f05a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89289161"
---
# <a name="handling-event-grid-events"></a>Control de eventos de Event Grid

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Los eventos de Media Services permiten que las aplicaciones reaccionen a distintos eventos (por ejemplo, el evento de cambio del estado del trabajo) a través de modernas arquitecturas sin servidor. Esto se consigue sin necesidad de código complejo ni de servicios de sondeo costosos e ineficientes. En su lugar, se insertan eventos a través de [Azure Event Grid](https://azure.microsoft.com/services/event-grid/) a los controladores de eventos como [Azure Functions](https://azure.microsoft.com/services/functions/), [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/) o incluso a su propio webhook y solo se paga por lo que se usa. Para información sobre los precios, consulte [Precios de Event Grid](https://azure.microsoft.com/pricing/details/event-grid/).

La disponibilidad de los eventos de Media Services está asociada a la [disponibilidad](../../event-grid/overview.md) de Event Grid. Estarán disponibles en otras regiones cuando lo esté Event Grid.  

## <a name="media-services-events-and-schemas"></a>Esquemas y eventos de Media Services

Event Grid usa las [suscripciones a eventos](../../event-grid/concepts.md#event-subscriptions) para enrutar los mensajes de eventos a los suscriptores. Los eventos de Media Services contienen toda la información necesaria para responder a cualquier cambio que se produzca en los datos. Puede identificar un evento de Media Services porque la propiedad eventType comienza por "Microsoft.Media".

Para más información, consulte el artículo sobre los [esquemas de eventos de Media Services](media-services-event-schemas.md).

## <a name="practices-for-consuming-events"></a>Prácticas para consumir eventos

Las aplicaciones que controlan los eventos de Media Services deben seguir algunos procedimientos recomendados:

* Dado que se pueden configurar varias suscripciones para enrutar eventos al mismo controlador de eventos, es importante no asumir que los eventos provienen de un origen determinado, sino comprobar el tema del mensaje para asegurarse de que proviene de la cuenta de almacenamiento esperable.
* De igual forma, compruebe que eventType es uno de los que está preparado para procesar y no asuma que todos los eventos que reciba van a ser los tipos que espera.
* Ignore los campos que no comprenda.  Esta práctica le ayudará a mantenerse resistente a las nuevas características que puedan agregarse en el futuro.
* Use las coincidencias de prefijo y sufijo "subject" para limitar los eventos a un evento determinado.

> [!NOTE]
> Los eventos están sujetos al [Contrato de nivel de servicio (SLA)](https://azure.microsoft.com/support/legal/sla/event-grid/v1_0/) para Event Grid. Si quiere obtener notificaciones de eventos mediante las API, vea los ejemplos sobre cómo consumir eventos, con [SDK de .NET](https://github.com/Azure-Samples/media-services-v3-dotnet) o [SDK de Java](https://github.com/Azure-Samples/media-services-v3-java).

## <a name="next-steps"></a>Pasos siguientes

* [Supervisión de eventos: Portal](monitor-events-portal-how-to.md)
* [Supervisión de eventos: CLI](job-state-events-cli-how-to.md)
