---
title: 'API del servicio de medición: marketplace comercial de Microsoft'
description: La API de eventos de uso permite emitir eventos de uso para las ofertas de SaaS en Microsoft AppSource y Azure Marketplace.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
ms.date: 05/26/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: ac48973653e89d43521979a5606a8a3a3c2e1346
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87319990"
---
# <a name="marketplace-metered-billing-apis"></a>API de facturación según uso de Marketplace

Las API de facturación según uso deben usarse cuando el publicador crea dimensiones de medición personalizadas para que una oferta se publique en el Centro de partners. La integración con las API de facturación según uso es necesaria para cualquier oferta comprada que tenga uno o más planes con dimensiones personalizadas para emitir eventos de uso.

Para obtener más información sobre cómo crear dimensiones de medición personalizadas para SaaS, consulte [Facturación según uso de SaaS](saas-metered-billing.md).

Para obtener más información sobre cómo crear dimensiones de medición personalizadas para una oferta de aplicación de Azure con un plan de aplicación administrada, consulte la [sección de configuración técnica sobre cómo crear una nueva oferta de Azure Apps](create-new-azure-apps-offer.md#technical-configuration)).

## <a name="enforcing-tls-12-note"></a>Nota sobre la aplicación de TLS 1.2

La versión 1.2 de TLS se aplica como versión mínima para las comunicaciones HTTPS. Asegúrese de usar esta versión de TLS en el código. Las versiones 1.0 y 1.1 están en desuso y se rechazarán los intentos de conexión.

## <a name="metered-billing-single-usage-event"></a>Evento de uso único de facturación según uso

El publicador debe llamar a la API de eventos de uso para emitir eventos de uso en un recurso activo (suscrito) para el plan adquirido por el cliente específico. El evento de uso se emite por separado para cada dimensión personalizada del plan definido por el publicador al publicar la oferta.

Solo se puede emitir un evento de uso durante cada hora de un día natural. Por ejemplo, hoy a las 8:15 puede emitir un evento de uso. Si se acepta este evento, se aceptará el siguiente evento de uso desde hoy a las 9:00. Si envía un evento adicional hoy entre las 8:15 y las 8:59:59, se rechazará como duplicado. Debe acumular todas las unidades consumidas en una hora y, a continuación, proceder a su emisión en un solo evento.

Solo se puede emitir un evento de uso durante cada hora de un día natural por recurso. Si se consume más de una unidad en una hora, acumule todas las unidades consumidas en la hora y, a continuación, proceda a su emisión en un solo evento. Los eventos de uso solo se pueden emitir durante las últimas 24 horas. Si emite un evento de uso en cualquier momento entre las 8:00 y las 8:59:59 (y se acepta) y envía un evento adicional durante el mismo día entre las 8:00 y las 8:59:59, se rechazará como duplicado.

**POST**: `https://marketplaceapi.microsoft.com/api/usageEvent?api-version=<ApiVersion>`

*Parámetros de consulta*:

| Parámetro | Recomendación          |
| ---------- | ---------------------- |
| `ApiVersion` | Use 2018-08-31. |
| | |

*Encabezados de la solicitud*:

| Content-type       | Use `application/json`  |
| ------------------ | ---------------------------- |
| `x-ms-requestid`     | Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta. |
| `x-ms-correlationid` | Valor de cadena único para la operación en el cliente. Este parámetro pone en correlación todos los eventos de la operación del cliente con los eventos del servidor. Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
| `authorization`   | Token de acceso único que identifica al ISV que realiza esta llamada API. El formato es `"Bearer <access_token>"` cuando el publicador recupera el valor del token, tal como se explica para <br> <ul> <li> SaaS en [Obtención del token con una solicitud HTTP POST](./pc-saas-registration.md#get-the-token-with-an-http-post). </li> <li> Aplicación administrada en [Estrategias de autenticación](./marketplace-metering-service-authentication.md). </li> </ul> |
| | |

*Ejemplo de cuerpo de la solicitud:*

```json
{
  "resourceId": <guid>, // unique identifier of the resource against which usage is emitted. 
  "quantity": 5.0, // how many units were consumed for the date and hour specified in effectiveStartTime, must be greater than 0, can be integer or float value
  "dimension": "dim1", // custom dimension identifier
  "effectiveStartTime": "2018-12-01T08:30:14", // time in UTC when the usage event occurred, from now and until 24 hours back
  "planId": "plan1", // id of the plan purchased for the offer
}
```

>[!NOTE]
>`resourceId` tiene un significado diferente para la aplicación SaaS y para la aplicación administrada que emite el medidor personalizado. 

En el caso de los planes de las aplicaciones administradas de Aplicación de Azure, `resourceId` es el elemento `resourceUsageId` que se encuentra en el elemento `billingDetails` del objeto de metadatos de la aplicación administrada. Se puede encontrar un script de ejemplo de captura en [uso del token de identidades administradas por Azure](./marketplace-metering-service-authentication.md#using-the-azure-managed-identities-token). 

En el caso de las ofertas de SaaS, `resourceId` es el id. de suscripción de SaaS. Para obtener más información sobre las suscripciones de SaaS, consulte [lista de suscripciones](./pc-saas-fulfillment-api-v2.md#get-list-of-all-subscriptions).

### <a name="responses"></a>Respuestas

Código: 200<br>
Aceptar. La emisión de uso se aceptó y registró en el lado de Microsoft para su posterior procesamiento y facturación.

Ejemplo de carga de respuesta: 

```json
{
  "usageEventId": <guid>, // unique identifier associated with the usage event in Microsoft records
  "status": "Accepted" // this is the only value in case of single usage event
  "messageTime": "2020-01-12T13:19:35.3458658Z", // time in UTC this event was accepted
  "resourceId": <guid>, // unique identifier of the resource against which usage is emitted. For SaaS it's the subscriptionId.
  "quantity": 5.0, // amount of emitted units as recorded by Microsoft
  "dimension": "dim1", // custom dimension identifier
  "effectiveStartTime": "2018-12-01T08:30:14", // time in UTC when the usage event occurred, as sent by the ISV
  "planId": "plan1", // id of the plan purchased for the offer
}
```

Código: 400 <br>
Solicitud incorrecta.

* Datos de la solicitud no válidos proporcionados o que faltan.
* `effectiveStartTime` es más de 24 horas en el pasado. El evento ha expirado.
* La suscripción de SaaS no se encuentra en el estado Suscrito.

Ejemplo de carga de respuesta: 

```json
{
  "message": "One or more errors have occurred.",
  "target": "usageEventRequest",
  "details": [
    {
      "message": "The resourceId is required.",
      "target": "ResourceId",
      "code": "BadArgument"
    }
  ],
  "code": "BadArgument"
}
```

Código: 403<br>

Prohibido. El token de autorización no se ha proporcionado, no es válido o ha expirado,  o bien la solicitud intenta acceder a una suscripción para una oferta que se publicó con un identificador de aplicación de Azure AD diferente del que se usó para crear el token de autorización.

Código: 409<br>
Conflicto. Ya se ha notificado correctamente un evento de uso para el id. de recurso especificado, la fecha y la hora de entrada en vigor del uso.

Ejemplo de carga de respuesta: 

```json
{
  "additionalInfo": {
    "acceptedMessage": {
      "usageEventId": "<guid>", //unique identifier associated with the usage event in Microsoft records
      "status": "Duplicate",
      "messageTime": "2020-01-12T13:19:35.3458658Z",
      "resourceId": "<guid>", //unique identifier of the resource against which usage is emitted.
      "quantity": 1.0,
      "dimension": "dim1",
      "effectiveStartTime": "2020-01-12T11:03:28.14Z",
      "planId": "plan1"
    }
  },
  "message": "This usage event already exist.",
  "code": "Conflict"
}
```

## <a name="metered-billing-batch-usage-event"></a>Evento de uso por lotes de facturación según uso

La API de eventos de uso por lotes permite emitir a la vez eventos de uso para más de un recurso específico que haya comprado. También permite emitir varios eventos de uso para el mismo recurso, siempre que tengan lugar durante horas del calendario diferentes. El número máximo de eventos en un solo lote es 25.

**POST:** `https://marketplaceapi.microsoft.com/api/batchUsageEvent?api-version=<ApiVersion>`

*Parámetros de consulta*:

| Parámetro  | Recomendación     |
| ---------- | -------------------- |
| `ApiVersion` | Use 2018-08-31. |

*Encabezados de la solicitud*:

| Content-type       | Use `application/json`       |
| ------------------ | ------ |
| `x-ms-requestid`     | Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta. |
| `x-ms-correlationid` | Valor de cadena único para la operación en el cliente. Este parámetro pone en correlación todos los eventos de la operación del cliente con los eventos del servidor. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta. |
| `authorization`      | Token de acceso único que identifica al ISV que realiza esta llamada API. El formato es `Bearer <access_token>` cuando el publicador recupera el valor del token, tal como se explica para <br> <ul> <li> SaaS en [Obtención del token con una solicitud HTTP POST](./pc-saas-registration.md#get-the-token-with-an-http-post). </li> <li> Aplicación administrada en [Estrategias de autenticación](./marketplace-metering-service-authentication.md). </li> </ul> |
| | |


*Ejemplo de cuerpo de la solicitud:*

```json
{
  "request": [ // list of usage events for the same or different resources of the publisher
    { // first event
      "resourceId": "<guid1>", // Unique identifier of the resource against which usage is emitted. 
      "quantity": 5.0, // how many units were consumed for the date and hour specified in effectiveStartTime, must be greater than 0, can be integer or float value
      "dimension": "dim1", //Custom dimension identifier
      "effectiveStartTime": "2018-12-01T08:30:14",//Time in UTC when the usage event occurred, from now and until 24 hours back
      "planId": "plan1", // id of the plan purchased for the offer
    },
    { // next event
      "resourceId": "<guid2>", 
      "quantity": 39.0, 
      "dimension": "email", 
      "effectiveStartTime": "2018-11-01T23:33:10
      "planId": "gold", // id of the plan purchased for the offer
    }
  ]
}
```

>[!NOTE]
>`resourceId` tiene un significado diferente para la aplicación SaaS y para la aplicación administrada que emite el medidor personalizado. 

En el caso de los planes de las aplicaciones administradas de Aplicación de Azure, `resourceId` es el elemento `resourceUsageId` que se encuentra en el elemento `billingDetails` del objeto de metadatos de la aplicación administrada. Se puede encontrar un script de ejemplo de captura en [uso del token de identidades administradas por Azure](./marketplace-metering-service-authentication.md#using-the-azure-managed-identities-token). 

En el caso de las ofertas de SaaS, `resourceId` es el id. de suscripción de SaaS. Para obtener más información sobre las suscripciones de SaaS, consulte [lista de suscripciones](./pc-saas-fulfillment-api-v2.md#get-list-of-all-subscriptions).

### <a name="responses"></a>Respuestas

Código: 200<br>
Aceptar. La emisión de uso por lotes se aceptó y registró en el lado de Microsoft para su posterior procesamiento y facturación. La lista de respuestas se devuelve con el estado de cada evento individual del lote. Debe recorrer en iteración la carga de respuesta para comprender las respuestas de cada evento de uso individual que se envía como parte del evento por lotes.

Ejemplo de carga de respuesta: 

```json
{
  "count": 2, // number of records in the response
  "result": [
    { // first response
      "usageEventId": "<guid>", // unique identifier associated with the usage event in Microsoft records
      "status": "Accepted" // see list of possible statuses below,
      "messageTime": "2020-01-12T13:19:35.3458658Z", // Time in UTC this event was accepted by Microsoft,
      "resourceId": "<guid1>", // unique identifier of the resource against which usage is emitted.
      "quantity": 5.0, // amount of emitted units as recorded by Microsoft 
      "dimension": "dim1", // custom dimension identifier
      "effectiveStartTime": "2018-12-01T08:30:14",// time in UTC when the usage event occurred, as sent by the ISV
      "planId": "plan1", // id of the plan purchased for the offer
    },
    { // second response
      "status": "Duplicate",
      "messageTime": "0001-01-01T00:00:00",
      "error": {
        "additionalInfo": {
          "acceptedMessage": {
            "usageEventId": "<guid>",
            "status": "Duplicate",
            "messageTime": "2020-01-12T13:19:35.3458658Z",
            "resourceId": "<guid2>",
            "quantity": 1.0,
            "dimension": "email",
            "effectiveStartTime": "2020-01-12T11:03:28.14Z",
            "planId": "gold"
          }
        },
        "message": "This usage event already exist.",
        "code": "Conflict"
      },
      "resourceId": "<guid2>",
      "quantity": 1.0,
      "dimension": "email",
      "effectiveStartTime": "2020-01-12T11:03:28.14Z",
      "planId": "gold"
    }
  ]
}
```

Descripción del código de estado al que se hace referencia en respuesta de la API `BatchUsageEvent`:

| status code  | Descripción |
| ---------- | -------------------- |
| `Accepted` | Accepted. |
| `Expired` | Uso expirado. |
| `Duplicate` | Se ha proporcionado un uso duplicado. |
| `Error` | Código de error. |
| `ResourceNotFound` | El recurso de uso proporcionado no es válido. |
| `ResourceNotAuthorized` | No está autorizado a proporcionar el uso de este recurso. |
| `InvalidDimension` | La dimensión para la que se ha pasado el uso no es válida para esta oferta o plan. |
| `InvalidQuantity` | La cantidad pasada es menor o igual que 0. |
| `BadArgument` | Falta la entrada o tiene un formato incorrecto. |

Código: 400<br>
Solicitud incorrecta. El lote contenía más de 25 eventos de uso.

Código: 403<br>
Prohibido. El token de autorización no se ha proporcionado, no es válido o ha expirado,  o bien la solicitud intenta acceder a una suscripción para una oferta que se publicó con un identificador de aplicación de Azure AD diferente del que se usó para crear el token de autorización.

## <a name="development-and-testing-best-practices"></a>Procedimientos recomendados de desarrollo y pruebas

Para probar la emisión del medidor personalizada, implemente la integración con la API de medición y cree un plan para la oferta de SaaS publicada con dimensiones personalizadas definidas en él con un precio de cero por unidad. Además, publique esta oferta como versión preliminar. De este modo, solo los usuarios limitados podrán acceder a la integración y probarla.

También puede usar el plan privado para que una oferta activa existente limite el acceso a este plan durante las pruebas a una audiencia limitada.

## <a name="get-support"></a>Obtención de soporte técnico

Siga las instrucciones de [Soporte técnico para el programa de marketplace comercial en el Centro de partners](./support.md) para conocer las opciones de soporte técnico para publicadores y abrir una incidencia de soporte técnico con Microsoft.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre las API del servicio de medición, consulte las [preguntas más frecuentes de las API del servicio de medición de Marketplace](./marketplace-metering-service-apis-faq.md).
