---
title: Entrega de eventos de WebHook
description: En este artículo se describe la entrega de eventos de webhook y la validación de puntos de conexión al usar webhooks.
ms.topic: conceptual
ms.date: 07/07/2020
ms.openlocfilehash: e9a52d0cb3e4e880d91e1b748d97ef3041298930
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461245"
---
# <a name="webhook-event-delivery"></a>Entrega de eventos de webhook
Los webhooks son una de las muchas maneras de recibir eventos de Azure Event Grid. Cuando un nuevo evento está preparado, el servicio EventGrid publica una solicitud HTTP en el punto de conexión configurado con el evento en el cuerpo de la solicitud.

Al igual que muchos otros servicios que admiten webhooks, EventGrid requiere que demuestre la propiedad de su punto de conexión de Webhook antes de que comience a entregar eventos a ese punto de conexión. Este requisito evita que un usuario malintencionado sature el punto de conexión con eventos. Cuando utiliza cualquiera de los tres servicios de Azure enumerados a continuación, la infraestructura de Azure controla automáticamente esta validación:

- Azure Logic Apps con el [conector de Event Grid](/connectors/azureeventgrid/)
- Azure Automation a través de [webhook](../event-grid/ensure-tags-exists-on-new-virtual-machines.md)
- Azure Functions con el [desencadenador de Event Grid](../azure-functions/functions-bindings-event-grid.md)

## <a name="endpoint-validation-with-event-grid-events"></a>Validación de puntos de conexión con eventos de Event Grid
Si utiliza cualquier otro tipo de punto de conexión, como una función de Azure basada en un desencadenador HTTP, el código del punto de conexión debe participar en un protocolo de enlace de validación con EventGrid. Event Grid admite dos formas de validar la suscripción.

1. **Enlace sincrónico**: En el momento de crear la suscripción a eventos, Event Grid envía un evento de validación de suscripción en su punto de conexión. El esquema de este evento es similar a cualquier otro evento de Event Grid. La parte de datos de este evento incluye una propiedad `validationCode`. La aplicación comprueba que la solicitud de validación es para una suscripción a un evento esperado, y devuelve el código de validación en la respuesta de forma sincrónica. Este mecanismo del protocolo de enlace se admite en todas las versiones de EventGrid.

2. **Enlace asincrónico**: En algunos casos, no se puede devolver ValidationCode en la respuesta de forma sincrónica. Por ejemplo, si usa un servicio de terceros (como [`Zapier`](https://zapier.com) o [IFTTT](https://ifttt.com/)), no puede responder con el código de validación mediante programación.

   A partir de la versión 2018-05-01-preview, EventGrid admite un protocolo de enlace de validación manual. Si va a crear una suscripción de eventos mediante el SDK o la herramienta que usa la versión de API 2018-05-01-preview, EventGrid envía una propiedad `validationUrl` en los datos del evento de validación de suscripción. Para completar el protocolo de enlace, busque esa dirección URL en los datos del evento y envíele una solicitud GET. Puede usar un cliente de REST o el explorador web.

   La dirección URL proporcionada es válida durante **5 minutos**. Durante ese tiempo, el estado de aprovisionamiento de la suscripción al eventos es `AwaitingManualAction`. Si no ha completado la validación manual en 5 minutos, el estado de aprovisionamiento se establece en `Failed`. Tendrá que crear la suscripción de eventos de nuevo antes de intentar la validación manual.

   Este mecanismo de autenticación también requiere el punto de conexión de webhook para devolver un código de estado HTTP de 200 para que sepa que se aceptó el objeto POST para el evento de validación antes de se pueda colocar en el modo de validación manual. Es decir, si el punto de conexión devuelve 200, pero no devuelve una respuesta de validación de forma sincrónica, el modo se cambia al modo de validación manual. Si hay una operación GET en la dirección URL de validación dentro de un plazo de cinco minutos, se considera el protocolo de enlace de validación se realizó correctamente.

> [!NOTE]
> Para la validación no se admite el uso de certificados autofirmados. En su lugar, use un certificado firmado de una entidad de certificación (CA) comercial.

### <a name="validation-details"></a>Detalles de la validación

- En el momento de crear o actualizar la suscripción a eventos, Event Grid publica un evento de validación de suscripción en el punto de conexión de destino.
- El evento contiene un valor de encabezado "aeg-event-type: SubscriptionValidation".
- El cuerpo del evento tiene el mismo esquema que otros eventos de Event Grid.
- La propiedad eventType del evento es `Microsoft.EventGrid.SubscriptionValidationEvent`.
- La propiedad de datos del evento incluye una propiedad `validationCode` con una cadena generada aleatoriamente. Por ejemplo, "validationCode: acb13…".
- Los datos del evento también incluyen una propiedad `validationUrl` con una dirección URL para validar manualmente la suscripción.
- La matriz contiene solo el evento de validación. Otros eventos se envían en una solicitud aparte después de devolver el código de validación.
- Los SDK de EventGrid DataPlane tienen clases que corresponden a los datos de evento de validación de suscripción y a la respuesta de validación de suscripción.

En el siguiente ejemplo se muestra un evento SubscriptionValidationEvent de ejemplo:

```json
[
  {
    "id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66",
    "topic": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "subject": "",
    "data": {
      "validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6",
      "validationUrl": "https://rp-eastus2.eventgrid.azure.net:553/eventsubscriptions/estest/validate?id=512d38b6-c7b8-40c8-89fe-f46f9e9622b6&t=2018-04-26T20:30:54.4538837Z&apiVersion=2018-05-01-preview&token=1A1A1A1A"
    },
    "eventType": "Microsoft.EventGrid.SubscriptionValidationEvent",
    "eventTime": "2018-01-25T22:12:19.4556811Z",
    "metadataVersion": "1",
    "dataVersion": "1"
  }
]
```

Para comprobar la propiedad del punto de conexión, devuelva el código de validación en la propiedad validationResponse, como se muestra en el siguiente ejemplo:

```json
{
  "validationResponse": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"
}
```

Debe devolver un código de estado de respuesta HTTP 200 OK. HTTP 202 Aceptado no se reconoce como una respuesta válida de validación de suscripción a Event Grid. La solicitud http debe completarse en 30 segundos. Si la operación no finaliza en 30 segundos, se cancelará y puede que se vuelva a intentar pasados cinco segundos. Si se producen errores en todos los intentos, se tratará como un error de protocolo de enlace de validación.

O bien, puede enviar una solicitud GET a la dirección URL de validación para validar la suscripción manualmente. La suscripción a eventos permanece en estado pendiente hasta que se valida. La dirección URL de validación usa el puerto 553. Si las reglas de firewall bloquean el puerto 553, puede que sea necesario actualizarlas para aplicar el protocolo de enlace manual de forma satisfactoria.

Para ver un ejemplo del tratamiento del protocolo de enlace de validación de suscripción, consulte un [ejemplo de C#](https://github.com/Azure-Samples/event-grid-dotnet-publish-consume-events/blob/master/EventGridConsumer/EventGridConsumer/Function1.cs).

## <a name="endpoint-validation-with-cloudevents-v10"></a>Validación de puntos de conexión con CloudEvents v1.0
Si ya está familiarizado con Event Grid, es posible que conozca el protocolo de enlace de validación de punto de conexión de Event Grid para evitar el uso inapropiado. CloudEvents v1.0 implementa su propia [semántica de protección contra abusos](webhook-event-delivery.md) mediante el método HTTP OPTIONS. Puede leer más sobre este tema [aquí](https://github.com/cloudevents/spec/blob/v1.0/http-webhook.md#4-abuse-protection). Al usar el esquema de CloudEvents para la salida, Event Grid lo hace con la protección contra abusos de CloudEvents v1.0 en lugar del mecanismo de eventos de validación de Event Grid.

## <a name="next-steps"></a>Pasos siguientes
Consulte el siguiente artículo para saber cómo solucionar problemas con las validaciones de suscripciones a eventos: 

[Solución de problemas de validación de suscripciones a eventos](troubleshoot-subscription-validation.md)
