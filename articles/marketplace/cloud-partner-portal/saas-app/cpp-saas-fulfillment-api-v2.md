---
title: SaaS realización API V2 | Azure Marketplace
description: Explica cómo crear una oferta de SaaS en los AppSource y Azure Marketplace mediante la realización asociado API V2.
services: Azure, Marketplace, Cloud Partner Portal,
author: v-miclar
ms.service: marketplace
ms.topic: conceptual
ms.date: 03/28/2019
ms.author: pabutler
ms.openlocfilehash: 551f3be2ca23bc18224d28faeea6a6df80eba1db
ms.sourcegitcommit: be9fcaace62709cea55beb49a5bebf4f9701f7c6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/17/2019
ms.locfileid: "65823532"
---
# <a name="saas-fulfillment-apis-version-2"></a>Versión de API de SaaS realización 2 

Este artículo se describe la API que permite a los proveedores de software independientes (ISV) para vender sus aplicaciones SaaS en Azure Marketplace y AppSource. Esta API es un requisito para enumerar SaaS se ofrece en AppSource y en Azure Marketplace.

## <a name="managing-the-saas-subscription-lifecycle"></a>Administrar el ciclo de vida de suscripción de SaaS

Microsoft SaaS Service administra todo el ciclo de vida de una compra de suscripción de SaaS y usa la API de realización de un mecanismo para controlar el pedido real, los cambios en los planes y la eliminación de la suscripción con el ISV. La factura se basa en el estado de la suscripción de SaaS que mantiene Microsoft. El siguiente diagrama muestra los Estados y las operaciones que controlan los cambios entre Estados.

![Estados de ciclo de vida de suscripción de SaaS](./media/saas-subscription-lifecycle-api-v2.png)


### <a name="states-of-a-saas-subscription"></a>Estados de una suscripción de SaaS

En la tabla siguiente se enumera los Estados de aprovisionamiento de una suscripción de SaaS, incluido un diagrama de secuencia y la descripción para cada una (si procede). 

#### <a name="provisioning"></a>Aprovisionamiento

Cuando un cliente inicia una compra, el ISV recibe esta información en un código de autenticación en una página web interactiva del cliente mediante un parámetro de dirección URL. Por ejemplo: `https://contoso.com/signup?token=..`, donde es el proveedor de dirección URL de página de aterrizaje en el centro de partners `https://contoso.com/signup`. El código de autenticación se puede validar e intercambian los detalles de lo que debe aprovisionarse mediante una llamada a la API de resolver.  Cuando el servicio de SaaS finalice el aprovisionamiento, envía una llamada de activar para indicar que el procesamiento está completo y puede facturará al cliente.  El diagrama siguiente muestra la secuencia de llamadas de API para un escenario de aprovisionamiento.  

![Llamadas de API para el aprovisionamiento de un servicio SaaS.](./media/saas-post-provisioning-api-v2-calls.png)

#### <a name="provisioned"></a>Aprovisionado

Este estado es el estado estable de un servicio de aprovisionamiento.

#### <a name="provisioning-for-update"></a>Aprovisionamiento de actualización
(desde marketplace) 

Este estado representa una actualización a un servicio está pendiente. Esta actualización se puede iniciar el cliente en marketplace, o en el servicio de SaaS (solo para directamente a las transacciones del cliente.) El siguiente diagrama muestra las acciones cuando se inicia una actualización desde el marketplace.

![Llamadas de API cuando se inicia la actualización desde marketplace.](./media/saas-update-api-v2-calls-from-marketplace-a.png)

#### <a name="provisioning-for-update"></a>Aprovisionamiento de actualización  
(desde el servicio de SaaS) 

El siguiente diagrama muestra las acciones cuando se inicia una actualización al servicio SaaS. (La llamada de webhook se sustituye por una actualización de la suscripción iniciada por el SaaS Service. 

![Llamadas de API cuando se inicia la actualización al servicio SaaS.](./media/saas-update-api-v2-calls-from-saas-service-a.png) 

#### <a name="suspended"></a>Suspendido

Este estado indica que no se ha recibido el pago del cliente. Mediante la directiva, se proporcionará al cliente un período de gracia antes unfulfilling la suscripción. Cuando una suscripción está en este estado: 

- Como ISV, se puede degradar o bloquear el acceso del usuario al servicio. 
- La suscripción debe estar en un estado recuperable que puede restaurar toda la funcionalidad sin ninguna pérdida de datos o la configuración. 
- Puede esperar obtener una solicitud de restablecimiento para esta suscripción a través de la API de realización, o una solicitud de desaprovisionamiento al final del período de gracia. 

#### <a name="unsubscribed"></a>Suscripción cancelada 

Las suscripciones alcanza este estado en respuesta a una solicitud de cliente explícita o como respuesta a un impago de cuotas. La expectativa de que el ISV es que los datos del cliente se conservan para la recuperación en la solicitud durante un período mínimo de X días y, a continuación, se eliminan. 


## <a name="api-reference"></a>Referencia de la API

Esta sección documentan el SaaS *API de suscripción* y *API Operations*.  El valor de la `api-version` las API de parámetro para la versión 2 están `2018-08-31`.  


### <a name="parameter-and-entity-definitions"></a>Definiciones de parámetro y entidad

En la tabla siguiente se enumera las definiciones de las entidades utilizadas por las API de entrega y parámetros comunes.

|     Parámetro/entidad     |     Definición                         |
|     ----------------     |     ----------                         |
| `subscriptionId`         | Identificador GUID para un recurso de SaaS  |
| `name`                   | Nombre descriptivo proporcionado para este recurso por el cliente |
| `publisherId`            | Identificador de cadena único para cada publicador, por ejemplo, "contoso" |
| `offerId`                | Identificador de cadena único para cada oferta, por ejemplo, "offer1"  |
| `planId`                 | Identificador de cadena único para cada plan/sku, por ejemplo "silver" |
| `operationId`            | Identificador GUID para una operación determinada  |
|  `action`                | La acción realizada en un recurso, ya sea `subscribe`, `unsubscribe`, `suspend`, `reinstate`, o `changePlan`, `changeQuantity`, `transfer`  |
|   |   |

Identificadores únicos globales ([GUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)) son números (hexadecimal de 32) de 128 bits que normalmente se generan automáticamente. 

#### <a name="resolve-a-subscription"></a>Resolver una suscripción 

Permite que el punto de conexión de resolver el publicador resolver un símbolo (token) de marketplace a un identificador de recurso persistente. El identificador de recurso es el identificador único para la suscripción de SAAS.  Cuando se redirige a un usuario al sitio web del ISV, la dirección URL contiene un token en los parámetros de consulta. Se espera que el ISV use este token y haga una solicitud para resolverlo. La respuesta contiene el id. de suscripción de SAAS, el nombre, el id. de oferta y el plan exclusivos para el recurso. Este token es válido durante una hora únicamente. 

**Exponer:<br>`https://marketplaceapi.microsoft.com/api/saas/subscriptions/resolve?api-version=<ApiVersion>`**

*Parámetros de consulta:*

|                    |                   |
|  ---------------   |  ---------------  |
|  ApiVersion        |  Versión de la operación que se usará para esta solicitud  |

*Encabezados de solicitud:*
 
|                    |                   |
|  ---------------   |  ---------------  |
|  Content-Type      | `application/json` |
|  x-ms-requestid    |  Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta. |
|  x-ms-correlationid |  Valor de cadena único para la operación en el cliente. Este parámetro se establece una correlación entre todos los eventos de la operación de cliente con los eventos en el servidor. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  authorization     |  [Obtener un token de portador JSON web token (JWT)](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/saas-app/cpp-saas-registration#get-a-token-based-on-the-azure-ad-app) |
|  x-ms-marketplace-token  |  Parámetro de token de consulta en la dirección URL cuando se redirige al usuario al sitio Web del ISV de SaaS de Azure (para p. ej.: `https://contoso.com/signup?token=..`). *Nota:* La dirección URL descodifica el valor del token desde el explorador antes de usarlo.  |

*Códigos de respuesta:*

Código: 200<br>
Resuelve el token opaco para una suscripción de SaaS.<br>

```json
Response body:
{
    "subscriptionId": "<guid>",  
    "subscriptionName": "Contoso Cloud Solution",
    "offerId": "offer1",
    "planId": "silver",
    "quantity": "20" 
}
```

Código: 404<br>
No encontrado

Código: 400<br>
Solicitud incorrecta. x-ms-marketplace-token es falta, con formato incorrecto o expirado.

Código: 403<br>
No dispone de los permisos necesarios. No se proporcionó el token de autenticación, no es válido, o la solicitud está intentando obtener acceso a una adquisición que no pertenece al publicador actual.

Código: 500<br>
Internal Server Error

```json
{
    "error": {
      "code": "UnexpectedError",
      "message": "An unexpected error has occurred."
    }
}
```

### <a name="subscription-api"></a>API de suscripción

La API de suscripción admite las siguientes operaciones de HTTPS: **Obtener**, **Post**, **Patch**, y **eliminar**.


#### <a name="list-subscriptions"></a>Mostrar suscripciones

Enumera todas las suscripciones de SaaS para un publicador.

**Obtener:<br>`https://marketplaceapi.microsoft.com/api/saas/subscriptions?api-version=<ApiVersion>`**

*Parámetros de consulta:*

|             |                   |
|  --------   |  ---------------  |
| ApiVersion  |  Versión de la operación que se usará para esta solicitud.  |

*Encabezados de solicitud:*

|                    |                   |
|  ---------------   |  ---------------  |
| Content-Type       |  `application/json`  |
| x-ms-requestid     |  Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta. |
| x-ms-correlationid |  Valor de cadena único para la operación en el cliente. Este parámetro se establece una correlación entre todos los eventos de la operación de cliente con los eventos en el servidor. Si no se proporciona este valor, uno genera y se proporcionan en los encabezados de respuesta.  |
| authorization      |  [Obtener el token de portador JSON web token (JWT).](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/saas-app/cpp-saas-registration#get-a-token-based-on-the-azure-ad-app)  |

*Códigos de respuesta:*

Código: 200 <br/>
Según el token de autenticación, obtener el publicador y las suscripciones correspondientes para las ofertas de todos los del publicador.<br> Carga de respuesta:<br>

```json
{
  [
      {
          "id": "<guid>",
          "name": "Contoso Cloud Solution",
          "publisherId": "contoso",
          "offerId": "offer1",
          "planId": "silver",
          "quantity": "10",
          "beneficiary": { // Tenant for which SaaS subscription is purchased.
              "tenantId": "<guid>"
          },
          "purchaser": { // Tenant that purchased the SaaS subscription. These could be different for reseller scenario
              "tenantId": "<guid>"
          },
          "allowedCustomerOperations": [
              "Read" // Possible Values: Read, Update, Delete.
          ], // Indicates operations allowed on the SaaS subscription. For CSP initiated purchases, this will always be Read.
          "sessionMode": "None", // Possible Values: None, DryRun (Dry Run indicates all transactions run as Test-Mode in the commerce stack)
          "saasSubscriptionStatus": "Subscribed" // Indicates the status of the operation. [Provisioning, Subscribed, Suspended, Unsubscribed]
      }
  ],
  "continuationToken": ""
}
```

El token de continuación solo estarán presente si hay más "páginas" de los planes para recuperar. 

Código: 403 <br>
No dispone de los permisos necesarios. No se proporcionó el token de autenticación, no es válido, o la solicitud está intentando obtener acceso a una adquisición que no pertenece al publicador actual. 

Código: Error de servidor interno 500

```json
{
    "error": {
      "code": "UnexpectedError",
      "message": "An unexpected error has occurred."
    }
}
```

#### <a name="get-subscription"></a>Obtener id. de suscripción

Obtiene la suscripción especificada de SaaS. Use esta llamada para obtener información de licencia y la información del plan.

**Obtener:<br> `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId?api-version=<ApiVersion>`**

*Parámetros de consulta:*

|                    |                   |
|  ---------------   |  ---------------  |
| subscriptionId     |   Identificador único de la suscripción de SaaS que se obtiene después de resolver el token a través de API resolver   |
|  ApiVersion        |   Versión de la operación que se usará para esta solicitud   |

*Encabezados de solicitud:*

|                    |                   |
|  ---------------   |  ---------------  |
|  Content-Type      |  `application/json`  |
|  x-ms-requestid    |  Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta. |
|  x-ms-correlationid |  Valor de cadena único para la operación en el cliente. Este parámetro se establece una correlación entre todos los eventos de la operación de cliente con los eventos en el servidor. Si no se proporciona este valor, uno genera y se proporcionan en los encabezados de respuesta.  |
|  authorization     |  [Obtener el token de portador JSON web token (JWT).](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/saas-app/cpp-saas-registration#get-a-token-based-on-the-azure-ad-app)  |

*Códigos de respuesta:*

Código: 200<br>
Obtiene la suscripción de SaaS de identificador<br> Carga de respuesta:<br>

```json
Response Body:
{ 
        "id":"",
        "name":"Contoso Cloud Solution",
        "publisherId": "contoso",
        "offerId": "offer1",
        "planId": "silver",
        "quantity": "10",
          "beneficiary": { // Tenant for which SaaS subscription is purchased.
              "tenantId": "<guid>"
          },
          "purchaser": { // Tenant that purchased the SaaS subscription. These could be different for reseller scenario
              "tenantId": "<guid>"
          },
        "allowedCustomerOperations": ["Read"], // Indicates operations allowed on the SaaS subscription. For CSP initiated purchases, this will always be Read.
        "sessionMode": "None", // Dry Run indicates all transactions run as Test-Mode in the commerce stack
        "status": "Subscribed", // Indicates the status of the operation.
}
```

Código: 404<br>
No encontrado<br> 

Código: 403<br>
No dispone de los permisos necesarios. No se proporcionó el token de autenticación, no es válido, o la solicitud está intentando obtener acceso a una adquisición que no pertenece al publicador actual.

Código: 500<br>
Internal Server Error<br>

```json
{
    "error": {
      "code": "UnexpectedError",
      "message": "An unexpected error has occurred."
    }  
```

#### <a name="list-available-plans"></a>Lista de planes disponibles

Use esta llamada para averiguar si no hay ninguna oferta pública y privada para el publicador actual.

**Obtener:<br> `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/listAvailablePlans?api-version=<ApiVersion>`**

*Parámetros de consulta:*

|                    |                   |
|  ---------------   |  ---------------  |
|  ApiVersion        |   Versión de la operación que se usará para esta solicitud  |

*Encabezados de solicitud:*

|                    |                   |
|  ---------------   |  ---------------  |
|   Content-Type     |  `application/json` |
|   x-ms-requestid   |   Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta. |
|  x-ms-correlationid  | Valor de cadena único para la operación en el cliente. Este parámetro se establece una correlación entre todos los eventos de la operación de cliente con los eventos en el servidor. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta. |
|  authorization     |  [Obtener el token de portador JSON web token (JWT).](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/saas-app/cpp-saas-registration#get-a-token-based-on-the-azure-ad-app) |

*Códigos de respuesta:*

Código: 200<br>
Obtener una lista de los planes disponibles para un cliente.<br>

Cuerpo de respuesta:

```json
{
    "plans": [{
        "planId": "Platinum001",
        "displayName": "Private platinum plan for Contoso",
        "isPrivate": true
    }]
}
```

Código: 404<br>
No encontrado<br> 

Código: 403<br>
No dispone de los permisos necesarios. No se proporcionó el token de autenticación, no es válido, o la solicitud está intentando obtener acceso a una adquisición que no pertenece al publicador actual. <br> 

Código: 500<br>
Internal Server Error<br>

```json
{ 
    "error": { 
      "code": "UnexpectedError", 
      "message": "An unexpected error has occurred." 
    } 
```

#### <a name="activate-a-subscription"></a>Activar una suscripción

**Exponer:<br> `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/activate?api-version=<ApiVersion>`**

*Parámetros de consulta:*

|                    |                   |
|  ---------------   |  ---------------  |
|  ApiVersion        |  Versión de la operación que se usará para esta solicitud  |
| subscriptionId     | Identificador único de la suscripción de SaaS que se obtiene después de resolver el token mediante la API de resolver  |

*Encabezados de solicitud:*
 
|                    |                   |
|  ---------------   |  ---------------  |
|  Content-Type      | `application/json`  |
|  x-ms-requestid    | Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  x-ms-correlationid  | Valor de cadena único para la operación en el cliente. Esta cadena correlaciona todos los eventos de la operación de cliente con los eventos en el servidor. Si no se proporciona este valor, uno genera y se proporcionan en los encabezados de respuesta.  |
|  authorization     |  [Obtener el token de portador JSON web token (JWT).](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/saas-app/cpp-saas-registration#get-a-token-based-on-the-azure-ad-app) |

*Solicitud:*

```json
{
    "planId": "gold",
    "quantity": ""
}
```

*Códigos de respuesta:*

Código: 200<br>
Activa la suscripción.<br>

Código: 404<br>
No encontrado

Código: 400<br>
Errores de validación de solicitud incorrecta

Código: 403<br>
No dispone de los permisos necesarios. No se proporcionó el token de autenticación, no es válido, o la solicitud está intentando obtener acceso a una adquisición que no pertenece al publicador actual.

Código: 500<br>
Internal Server Error

```json
{
    "error": {
      "code": "UnexpectedError",
      "message": "An unexpected error has occurred."
    }
}
```

#### <a name="change-the-plan-on-the-subscription"></a>Cambiar el plan en la suscripción

Actualizar el plan en la suscripción.

**Revisión:<br> `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>?api-version=<ApiVersion>`**

*Parámetros de consulta:*

|                    |                   |
|  ---------------   |  ---------------  |
|  ApiVersion        |  Versión de la operación que se usará para esta solicitud.  |
| subscriptionId     | Identificador único de la suscripción de SaaS que se obtiene después de resolver el token mediante la API de resolver.  |

*Encabezados de solicitud:*

|                    |                   |
|  ---------------   |  ---------------  |
|  Content-Type      | `application/json` |
|  x-ms-requestid    |   Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  x-ms-correlationid  |  Valor de cadena único para la operación en el cliente. Este parámetro se establece una correlación entre todos los eventos de la operación de cliente con los eventos en el servidor. Si no se proporciona este valor, uno genera y se proporcionan en los encabezados de respuesta.    |
| authorization      |  [Obtener el token de portador JSON web token (JWT).](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/saas-app/cpp-saas-registration#get-a-token-based-on-the-azure-ad-app)  |

*Carga de solicitud:*

```json
Request Body:
{
    "planId": "gold"
}
```

*Encabezados de solicitud:*

|                    |                   |
|  ---------------   |  ---------------  |
| Operation-Location | Vincular a un recurso para obtener el estado de la operación.   |

*Códigos de respuesta:*

Código: 202<br>
Se ha aceptado la solicitud para cambiar el plan. Se espera el ISV para sondear la operación de ubicación para determinar un éxito/error. <br>

Código: 404<br>
No encontrado

Código: 400<br>
Errores de validación de solicitud incorrecta.

>[!Note]
>Solo un plan o cantidad puede ser revisada al mismo tiempo, no ambos. Edita en una suscripción con **actualización** no se encuentra en `allowedCustomerOperations`.

Código: 403<br>
No dispone de los permisos necesarios. No se proporcionó el token de autenticación, no es válido, o la solicitud está intentando obtener acceso a una adquisición que no pertenece al publicador actual.

Código: 500<br>
Internal Server Error

```json
{
    "error": {
      "code": "UnexpectedError",
      "message": "An unexpected error has occurred."
    }
}
```

#### <a name="change-the-quantity-on-the-subscription"></a>Cambiar la cantidad en la suscripción

Actualizar la cantidad en la suscripción.

**Revisión:<br> `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>?api-version=<ApiVersion>`**

*Parámetros de consulta:*

|                    |                   |
|  ---------------   |  ---------------  |
|  ApiVersion        |  Versión de la operación que se usará para esta solicitud.  |
| subscriptionId     | Identificador único de la suscripción de SaaS que se obtiene después de resolver el token mediante la API de resolver.  |

*Encabezados de solicitud:*

|                    |                   |
|  ---------------   |  ---------------  |
|  Content-Type      | `application/json` |
|  x-ms-requestid    |   Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  x-ms-correlationid  |  Valor de cadena único para la operación en el cliente. Este parámetro se establece una correlación entre todos los eventos de la operación de cliente con los eventos en el servidor. Si no se proporciona este valor, uno genera y se proporcionan en los encabezados de respuesta.    |
| authorization      |  [Obtener el token de portador JSON web token (JWT).](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/saas-app/cpp-saas-registration#get-a-token-based-on-the-azure-ad-app)  |

*Carga de solicitud:*

```json
Request Body:
{
    "quantity": 5
}
```

*Encabezados de solicitud:*

|                    |                   |
|  ---------------   |  ---------------  |
| Operation-Location | Vincular a un recurso para obtener el estado de la operación.   |

*Códigos de respuesta:*

Código: 202<br>
Accepted. Se ha aceptado la solicitud para cambiar la cantidad. Se espera el ISV para sondear la operación de ubicación para determinar un éxito/error. <br>

Código: 404<br>
No encontrado

Código: 400<br>
Errores de validación de solicitud incorrecta.

>[!Note]
>Solo un plan o cantidad puede ser revisada al mismo tiempo, no ambos. Edita en una suscripción con **actualización** no se encuentra en `allowedCustomerOperations`.

Código: 403<br>
No dispone de los permisos necesarios. No se proporcionó el token de autenticación, no es válido, o la solicitud está intentando obtener acceso a una adquisición que no pertenece al publicador actual.

Código: 500<br>
Internal Server Error

```json
{
    "error": {
      "code": "UnexpectedError",
      "message": "An unexpected error has occurred."
    }
}
```

#### <a name="delete-a-subscription"></a>Eliminar una suscripción

Cancelar la suscripción y eliminar la suscripción especificada.

**Eliminar:<br> `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId> ?api-version=<ApiVersion>`**

*Parámetros de consulta:*

|                    |                   |
|  ---------------   |  ---------------  |
|  ApiVersion        |  Versión de la operación que se usará para esta solicitud.  |
| subscriptionId     | Identificador único de la suscripción de SaaS que se obtiene después de resolver el token mediante la API de resolver.  |

*Encabezados de solicitud:*
 
|                    |                   |
|  ---------------   |  ---------------  |
|   Content-Type     |  `application/json` |
|  x-ms-requestid    |   Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, uno genera y se proporcionan en los encabezados de respuesta.   |
|  x-ms-correlationid  |  Valor de cadena único para la operación en el cliente. Este parámetro se establece una correlación entre todos los eventos de la operación de cliente con los eventos en el servidor. Si no se proporciona este valor, uno genera y se proporcionan en los encabezados de respuesta.   |
|  authorization     |  [Obtener el token de portador JSON web token (JWT).](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/saas-app/cpp-saas-registration#get-a-token-based-on-the-azure-ad-app)  |

*Códigos de respuesta:*

Código: 202<br>
Llamada de ISV que se inicia para indicar su suscripción en una suscripción de SaaS.<br>

Código: 404<br>
No encontrado

Código: 400<br>
Eliminar en una suscripción con **eliminar** no en `allowedCustomerOperations`.

Código: 403<br>
No dispone de los permisos necesarios. No se proporcionó el token de autenticación, no es válido, o la solicitud está intentando obtener acceso a una adquisición que no pertenece al publicador actual.

Código: 500<br>
Internal Server Error

```json
{
    "error": {
      "code": "UnexpectedError",
      "message": "An unexpected error has occurred."
    }
}
```


### <a name="operations-api"></a>Operaciones de API

La API de operaciones es compatible con las siguientes operaciones de revisión y Get.

#### <a name="list-outstanding-operations"></a>Lista de operaciones pendiente 

Enumera las operaciones pendientes para el publicador actual. 

**Obtener:<br> `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/operations?api-version=<ApiVersion>`**

*Parámetros de consulta:*

|             |        |
|  ---------------   |  ---------------  |
|    ApiVersion                |   Versión de la operación que se usará para esta solicitud.                |
| subscriptionId     | Identificador único de la suscripción de SaaS que se obtiene después de resolver el token mediante la API de resolver.  |

*Encabezados de solicitud:*
 
|                    |                   |
|  ---------------   |  ---------------  |
|   Content-Type     |  `application/json` |
|  x-ms-requestid    |  Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  x-ms-correlationid |  Valor de cadena único para la operación en el cliente. Este parámetro se establece una correlación entre todos los eventos de la operación de cliente con los eventos en el servidor. Si no se proporciona este valor, uno genera y se proporcionan en los encabezados de respuesta.  |
|  authorization     |  [Obtener el token de portador JSON web token (JWT).](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/saas-app/cpp-saas-registration#get-a-token-based-on-the-azure-ad-app)  |

*Códigos de respuesta:*

Código: 200<br> Obtiene la lista de operaciones pendientes en una suscripción.<br>
Carga de respuesta:

```json
[{
    "id": "<guid>",  
    "activityId": "<guid>",
    "subscriptionId": "<guid>",
    "offerId": "offer1",
    "publisherId": "contoso",  
    "planId": "silver",
    "quantity": "20",
    "action": "Convert",
    "timeStamp": "2018-12-01T00:00:00",  
    "status": "NotStarted"  
}]
```

Código: 404<br>
No encontrado

Código: 400<br>
Errores de validación de solicitud incorrecta

Código: 403<br>
No dispone de los permisos necesarios. No se proporcionó el token de autenticación, no es válido, o la solicitud está intentando obtener acceso a una adquisición que no pertenece al publicador actual.

Código: 500<br>
Internal Server Error

```json
{
    "error": {
      "code": "UnexpectedError",
      "message": "An unexpected error has occurred."
    }
}

```

#### <a name="get-operation-status"></a>Obtener estado de la operación

Permite que el publicador un seguimiento del estado de la operación asincrónica desencadenadas especificado (suscribirse o cancelar la suscripción, cambiar plan / cambiar cantidad).

**Obtener:<br> `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/operations/<operationId>?api-version=<ApiVersion>`**

*Parámetros de consulta:*

|                    |                   |
|  ---------------   |  ---------------  |
|  ApiVersion        |  Versión de la operación que se usará para esta solicitud.  |

*Encabezados de solicitud:*

|                    |                   |
|  ---------------   |  ---------------  |
|  Content-Type      |  `application/json`   |
|  x-ms-requestid    |   Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  x-ms-correlationid |  Valor de cadena único para la operación en el cliente. Este parámetro se establece una correlación entre todos los eventos de la operación de cliente con los eventos en el servidor. Si no se proporciona este valor, uno genera y se proporcionan en los encabezados de respuesta.  |
|  authorization     |[Obtener el token de portador JSON web token (JWT).](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/saas-app/cpp-saas-registration#get-a-token-based-on-the-azure-ad-app)  |

*Códigos de respuesta:* Código: 200<br> Obtiene el objeto SaaS operación pendiente<br>
Carga de respuesta:

```json
Response body:
{
    "id  ": "<guid>",
    "activityId": "<guid>",
    "subscriptionId":"<guid>",
    "offerId": "offer1",
    "publisherId": "contoso",  
    "planId": "silver",
    "quantity": "20",
    "action": "Convert",
    "timeStamp": "2018-12-01T00:00:00",
    "status": "NotStarted"
}

```

Código: 404<br>
No encontrado

Código: 400<br>
Errores de validación de solicitud incorrecta

Código: 403<br>
No dispone de los permisos necesarios. No se proporcionó el token de autenticación, no es válido, o la solicitud está intentando obtener acceso a una adquisición que no pertenece al publicador actual.
 
Código: 500<br> Internal Server Error

```json
{
    "error": {
      "code": "UnexpectedError",
      "message": "An unexpected error has occurred."
    }
}

```
#### <a name="update-the-status-of-an-operation"></a>El estado de una operación de actualización

Actualizar el estado de una operación para indicar el éxito o error con los valores proporcionados.

**Revisión:<br> `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/operations/<operationId>?api-version=<ApiVersion>`**

*Parámetros de consulta:*

|                    |                   |
|  ---------------   |  ---------------  |
|   ApiVersion       |  Versión de la operación que se usará para esta solicitud.  |
| subscriptionId     | Identificador único de la suscripción de SaaS que se obtiene después de resolver el token mediante la API de resolver.  |
|  operationId       | La operación que se ha completado. |

*Encabezados de solicitud:*

|                    |                   |
|  ---------------   |  ---------------  |
|   Content-Type     | `application/json`   |
|   x-ms-requestid   |   Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si no se proporciona este valor, se generará uno y se proporcionará en los encabezados de respuesta. |
|  x-ms-correlationid |  Valor de cadena único para la operación en el cliente. Este parámetro se establece una correlación entre todos los eventos de la operación de cliente con los eventos en el servidor. Si no se proporciona este valor, uno genera y se proporcionan en los encabezados de respuesta. |
|  authorization     |  [Obtener el token de portador JSON web token (JWT).](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal/saas-app/cpp-saas-registration#get-a-token-based-on-the-azure-ad-app)  |

*Carga de solicitud:*

```json
{
    "planId": "offer1",
    "quantity": "44",
    "status": "Success"    // Allowed Values: Success/Failure. Indicates the status of the operation.
}

```

*Códigos de respuesta:*

Código: 200<br> Llamada para informar del progreso de una operación en el lado de ISV. Por ejemplo, esta respuesta podría indicar el cambio de asientos/planes.

Código: 404<br>
No encontrado

Código: 400<br>
Errores de validación de solicitud incorrecta

Código: 403<br>
No dispone de los permisos necesarios. No se proporcionó el token de autenticación, no es válido, o la solicitud está intentando obtener acceso a una adquisición que no pertenece al publicador actual.

Código: 409<br>
Conflicto. Por ejemplo, ya se cumple una transacción más reciente

Código: 500<br> Internal Server Error

```json
{
    "error": {
      "code": "UnexpectedError",
      "message": "An unexpected error has occurred."
    }
}

```

## <a name="webhook-on-the-saas-service"></a>Webhook en el servicio de SaaS

El publicador debe implementar un webhook en este servicio de SaaS para informar a los usuarios de los cambios en su servicio. La API debe estar autenticado y será llamada por el servicio de SaaS de Microsoft. Se espera el servicio de SaaS para llamar a las operaciones de API para validar y autorización antes de llevar a cabo una acción en la notificación de webhook.

```json
{
    "operationId": "<guid>",
    "activityId": "<guid>",
    "subscriptionId":"<guid>",
    "offerId": "offer1",
    "publisherId": "contoso",
    "planId": "silver",
    "quantity": "20"  ,
    "action": "Subscribe",
    "timeStamp": "2018-12-01T00:00:00"
}
```

Donde la acción puede ser uno de los siguientes: 
- `Subscribe`  (Cuando se ha activado el recurso)
- `Unsubscribe` (Cuando se ha eliminado el recurso)
- `ChangePlan` (Si ha completado la operación de cambio de plan)
- `ChangeQuantity` (Si ha completado la operación de cambio de cantidad)
- `Suspend` (Cuando se ha suspendido el recurso)
- `Reinstate` (Cuando se ha recuperado recurso tras suspensión)


## <a name="mock-api"></a>API simulada

Puede usar nuestras API ficticio que le ayudarán a empezar a trabajar con el desarrollo, especialmente la creación de prototipos, y los proyectos de prueba. 

Punto de conexión de host: `https://marketplaceapi.microsoft.com/api` <br/>
Versión de API: `2018-09-15` <br/>
No se requiere autenticación <br/>
Uri de ejemplo: `https://marketplaceapi.microsoft.com/api/saas/subscriptions?api-version=2018-09-15` <br/>

Las rutas de acceso del punto de conexión de API son los mismos a través de simulacro y API Real, pero las versiones de API son diferentes. La versión es 2018-09-15 de simulacro y 2018-08-31 para la versión de producción. 

Cualquiera de las llamadas de API en este artículo se pueden realizar al punto de conexión de host ficticio. Puede esperar obtener datos simulados como respuesta. En general, puede esperar obtener datos simulados como respuesta. Las llamadas a los métodos de suscripción de actualización en la API simulada siempre devuelven 500. 

## <a name="next-steps"></a>Pasos siguientes

Los programadores pueden recuperar mediante programación y manipulación de las cargas de trabajo, ofertas y el Editor de perfiles utilizando la [API de REST de Cloud Partner Portal](https://docs.microsoft.com/azure/marketplace/cloud-partner-portal-orig/cloud-partner-portal-api-overview).
