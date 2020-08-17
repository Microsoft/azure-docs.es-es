---
title: API de suministro de SaaS v2 en el marketplace comercial de Microsoft
description: Aprenda a crear y administrar una oferta de SaaS en Microsoft AppSource y Azure Marketplace mediante las API de suministro v2.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: reference
ms.date: 06/10/2020
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: f40da30ff0d702078861367dea810cc8ca1ab91b
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87305149"
---
# <a name="saas-fulfillment-apis-version-2-in-microsoft-commercial-marketplace"></a>API de suministro de SaaS v2 en el marketplace comercial de Microsoft

En este artículo se describen las API que permiten a los partners vender sus ofertas de SaaS en Microsoft AppSource y Azure Marketplace. Se requiere un editor a fin de implementar la integración con estas API para publicar una oferta de SaaS comercializable en el Centro de partners.

## <a name="managing-the-saas-subscription-life-cycle"></a>Administrar el ciclo de vida de la suscripción a SaaS

Azure Marketplace administra todo el ciclo de vida de una suscripción de SaaS una vez que la haya comprado el cliente final.  Usa la página de aterrizaje, las API de suministro, las API de operaciones y el webhook como mecanismo para impulsar la activación y el uso reales de la suscripción de SaaS, las actualizaciones y la cancelación de la suscripción.  La factura que recibe el cliente final se basa en el estado de la suscripción de SaaS que mantiene Microsoft. 

### <a name="states-of-a-saas-subscription"></a>Estados de una suscripción a SaaS

Se muestran los estados de una suscripción de SaaS y las acciones aplicables.

![Ciclo de vida de una suscripción de SaaS en Marketplace](./media/saas-subscription-lifecycle-api-v2.png)

#### <a name="purchased-but-not-yet-activated-pendingfulfillmentstart"></a>Comprada pero todavía no activada (*PendingFulfillmentStart*)

Una vez que un cliente final (o CSP) compra una oferta de SaaS en Marketplace, se debe notificar la compra al editor para que se cree y se configure una nueva cuenta de SaaS para el cliente final en el lado del editor.

Para que tenga lugar la creación de una cuenta:

1. El cliente debe hacer clic en el botón **Configurar** que está disponible para una oferta de SaaS después de realizar la compra correctamente en Microsoft AppSource o Azure Portal. También puede hacerlo en el correo electrónico que recibirá poco después de la compra.
2. Después, Microsoft notifica al partner la compra, para lo que abre en la nueva pestaña del explorador la dirección URL de la página de aterrizaje con el parámetro token (el token de identificación de la compra en Marketplace).

Un ejemplo de este tipo de llamada es `https://contoso.com/signup?token=<blob>`, donde la dirección URL de la página de aterrizaje de esta oferta de SaaS en el Centro de partners está configurada como `https://contoso.com/signup`. Este token proporciona al editor un identificador que distingue de forma única la compra de SaaS y el cliente.

>[!NOTE]
>El editor no recibirá ninguna notificación de la compra de SaaS hasta que el cliente inicie el proceso de configuración desde el lado de Microsoft.

La dirección URL de la página de aterrizaje debe estar en funcionamiento de forma ininterrumpida y poder recibir nuevas llamadas de Microsoft en todo momento. Si la página de aterrizaje deja de estar disponible, los clientes no podrán registrarse en el servicio de SaaS y empezar a usarlo.

Después, debe devolverse el *token* a Microsoft desde el editor mediante una llamada a la [API Resolve de SaaS](#resolve-a-purchased-subscription), como valor del parámetro de encabezado `x-ms-marketplace-token header`.  Como resultado de la llamada a la API Resolve, se intercambia el token para los detalles de la compra de SaaS, como el identificador único de la compra, el de la oferta y el plan adquiridos, etc.

En la página de aterrizaje, el cliente debe haber iniciado sesión en la cuenta de SaaS nueva o existente mediante el inicio de sesión único (SSO) de Azure Active Directory (AAD).

El editor debe implementar el inicio de sesión único para proporcionar la experiencia de usuario que Microsoft necesita para este flujo.  Asegúrese de usar la aplicación de Azure AD multiinquilino y de permitir tanto cuentas profesionales y educativas como cuentas personales de Microsoft al configurar el SSO.  Este requisito solo se aplica a la página de aterrizaje y a los usuarios a los que se redirige al servicio de SaaS cuando ya han iniciado sesión con las credenciales de Microsoft. No se aplica a todos los inicios de sesión en el servicio de SaaS.

> [!NOTE]
>Si el inicio de sesión único requiere que un administrador conceda permiso a una aplicación, la descripción de la oferta en el Centro de partners debe indicar que se requiere acceso de nivel de administrador. Esto está pensado para cumplir con las [directivas de certificación de Marketplace comercial](https://docs.microsoft.com/legal/marketplace/certification-policies#10003-authentication-options).

Después de iniciar sesión, el cliente debe completar la configuración de SaaS en el lado del editor. Luego, el editor debe llamar a la [API Activate Subscription](#activate-a-subscription) para enviar una señal a Marketplace que indique que el aprovisionamiento de la cuenta de SaaS se ha completado.
Esto dará comienzo al ciclo de facturación del cliente. Si la llamada a la API Activate Subscription no se realiza correctamente, no se facturará la compra al cliente.


![Llamadas a API para un escenario de aprovisionamiento](./media/saas-update-api-v2-calls-from-saas-service-a.png) 

#### <a name="active-subscribed"></a>Activo (Suscrito)

Es el estado estable de una suscripción de SaaS aprovisionada. Una vez que se ha procesado la llamada a la [API Activate Subscription](#activate-a-subscription) en el lado de Microsoft, la suscripción de SaaS se marca como Suscrita. El servicio de SaaS ya está listo para que lo use el cliente en el lado del editor y se facturará al cliente.

Cuando la suscripción de SaaS ya esté activa y el cliente opte por iniciar la experiencia de SaaS de **Administración** desde Azure Portal o el Centro de administración de M365, se llamará de nuevo a la **dirección URL de la página de aterrizaje** de Microsoft con el parámetro de *token*, igual que en el flujo de activación.  El editor debe distinguir entre las nuevas compras y la administración de las cuentas de SaaS existentes y controlar en consecuencia la llamada a la dirección URL de la página de aterrizaje.

#### <a name="being-updated-subscribed"></a>Actualizando (Suscrito)

Esta acción significa que tanto Microsoft como el editor están procesando una actualización de una suscripción de SaaS activa existente. Pueden iniciar esta actualización:

* El cliente desde Marketplace
* El CSP desde Marketplace
* El cliente desde el sitio de SaaS del editor (no se aplica a las compras realizadas por el CSP)

Hay disponibles dos tipos de actualizaciones para una suscripción de SaaS:

1. Actualización del plan cuando el cliente elige otro plan para la suscripción.
1. Actualización de la cantidad cuando el cliente cambia el número de puestos comprados para la suscripción.

Solo se puede actualizar una suscripción activa. Mientras se actualiza la suscripción, su estado permanece activo en el lado de Microsoft.

##### <a name="update-initiated-from-the-marketplace"></a>Actualización iniciada desde Marketplace

En este flujo, el cliente cambia el plan de suscripción o la cantidad de puestos desde el Centro de administración de M365.  

1. Cuando se introduce una actualización, Microsoft llama a la dirección URL del webhook del editor, configurada en el campo **Webhook de conexión** del Centro de partners, con un valor adecuado para el parámetro *action* y otros pertinentes.  
1. El lado del editor debe realizar los cambios necesarios en el servicio de SaaS y notificar a Microsoft cuando se complete el cambio mediante una llamada a la [API Update Status of Operation](#update-the-status-of-an-operation).
1. Si se envía la revisión con un estado de error, el proceso de actualización no se completará en el lado de Microsoft.  La suscripción de SaaS se quedará con la cantidad de puestos y el plan existentes.

A continuación se muestra la secuencia de llamadas a API para un escenario de actualización iniciada desde Marketplace.

![Llamadas a API para una actualización iniciada desde Marketplace](./media/saas-update-status-api-v2-calls-marketplace-side.png)

##### <a name="update-initiated-from-the-publisher"></a>Actualización iniciada desde el editor

En este flujo, el cliente cambia el plan de suscripción o la cantidad de puestos comprados desde el propio servicio de SaaS. 

1. El código del editor debe llamar a la [API Change Plan](#change-the-plan-on-the-subscription) o a la [API Change Quantity](#change-the-quantity-of-seats-on-the-saas-subscription) antes de efectuar el cambio solicitado en el lado del editor. 

1. Microsoft aplicará el cambio a la suscripción y, luego, enviará una notificación al editor a través del **webhook de conexión** para aplicar el mismo cambio.  

1. Solo entonces el editor debe realizar el cambio necesario en la suscripción de SaaS y notificar a Microsoft cuando se efectúe el cambio mediante una llamada a la [API Update Status of Operation](#update-the-status-of-an-operation).

Esta es la secuencia de llamadas a API para el escenario de actualización iniciada desde el lado del editor.

![Llamadas a API para una actualización iniciada desde el lado del editor](./media/saas-update-status-api-v2-calls-publisher-side.png)

#### <a name="suspended-suspended"></a>Suspendido (*Suspendido*)

Este estado indica que no se ha recibido el pago de un cliente por el servicio de SaaS. Microsoft le notificará este cambio al editor en el estado de suscripción de SaaS. La notificación se realiza a través de una llamada al webhook con el parámetro *action* establecido en *Suspended*.

El editor podría realizar cambios en el servicio de SaaS en el lado del editor. Se recomienda que el editor ponga esta información a disposición del cliente suspendido y que limite o bloquee el acceso del cliente al servicio de SaaS.  Existe la posibilidad de que el pago nunca se reciba.

Microsoft ofrece al cliente un período de gracia de 30 días antes de cancelar automáticamente la suscripción. Cuando una suscripción se encuentra en el estado Suspendido:

* El ISV debe mantener la cuenta de SaaS en un estado recuperable. Se puede restaurar la funcionalidad completa sin que se pierdan datos o valores de configuración.
* Espere recibir una solicitud de restablecimiento para esta suscripción si se recibe el pago durante el período de gracia, o bien una solicitud de desaprovisionamiento al final del período de gracia, a través del mecanismo de webhook.

El estado de la suscripción se cambia a Suspendido en el lado de Microsoft antes de que el editor realice alguna acción. Solo se pueden suspender las suscripciones activas.

#### <a name="reinstated-suspended"></a>Restablecido (*Suspendido*)

La suscripción se restablece.

Esta acción indica que el instrumento de pago del cliente vuelve a ser válido y que se ha realizado un pago para la suscripción de SaaS.  La suscripción se restablece. En este caso: 

1. Microsoft llama al webhook con un parámetro *action* establecido en el valor *Reinstate*.  
1. El editor se asegura de que esta suscripción esté totalmente operativa de nuevo en el lado del editor.
1. El editor llama a la [API Patch Operation](#update-the-status-of-an-operation) con el estado correcto.  
1. Después, el restablecimiento se realizará correctamente y se volverá a facturar al cliente para la suscripción de SaaS. 
1. Si se envía la revisión con un estado de error, el proceso de restablecimiento no se completará en el lado de Microsoft. La suscripción permanecerá suspendida.

Si se envía la revisión con un estado de error, el proceso de restablecimiento no se completará en el lado de Microsoft.  La suscripción permanecerá suspendida.

Solo se puede restablecer una suscripción suspendida.  Mientras se esté restableciendo una suscripción de SaaS, su estado permanecerá suspendido.  Una vez completada esta operación, el estado de la suscripción pasará a ser activo.

#### <a name="renewed-subscribed"></a>Renovado (*Suscrito*)

Al final del período de suscripción (después de un mes o de un año), Microsoft renovará automáticamente la suscripción de SaaS.  El valor predeterminado para la opción de renovación automática es *true* para todas las suscripciones de SaaS. Las suscripciones de SaaS activas seguirán renovándose periódicamente. Microsoft no notifica al editor cuando se renueva una suscripción. Un cliente puede desactivar la renovación automática de una suscripción de SaaS a través del Centro de administración de M365 o de Azure Portal.  En este caso, la suscripción de SaaS se cancelará automáticamente al final del período de facturación actual.  Los clientes también pueden cancelar la suscripción de SaaS en cualquier momento.

Solo se renuevan automáticamente las suscripciones activas.  Las suscripciones permanecen activas durante el proceso de renovación si la renovación automática se realiza correctamente.  Después de la renovación, las fechas de inicio y finalización del período de suscripción se actualizarán con las fechas del nuevo período.

Si se produce un error en una renovación automática debido a un problema con el pago, la suscripción se suspenderá.  Se enviará una notificación al editor.

#### <a name="canceled-unsubscribed"></a>Cancelado (*Suscripción cancelada*) 

Las suscripciones alcanzan este estado como respuesta a una acción explícita del cliente o del CSP para la cancelación de una suscripción desde el sitio del editor, Azure Portal o el Centro de administración de M365.  Una suscripción también puede cancelarse implícitamente, debido al impago de importes pendientes, después de mantenerse en el estado suspendido durante 30 días.

Tras recibir una llamada al webhook de cancelación, el editor debe conservar los datos del cliente para llevar a cabo la recuperación si se solicita durante al menos siete días. Solo entonces se pueden eliminar los datos del cliente.

Una suscripción de SaaS se puede cancelar en cualquier momento de su ciclo de vida. Una vez cancelada, no se puede reactivar una suscripción.

## <a name="api-reference"></a>Referencia de API

En esta sección se documentan las API Subscription y Operations.

Las **API Subscription** deben usarse para controlar el ciclo de vida de la suscripción de SaaS desde el momento de la compra hasta la cancelación.

Las **API Operations** deben usarse para:

* Comprobar y confirmar las llamadas al webhook procesadas
* Obtener una lista de las operaciones pendientes de las aplicaciones que están a la espera de la confirmación del editor

### <a name="enforcing-tls-12-note"></a>Nota sobre la aplicación de TLS 1.2

La versión 1.2 de TLS pronto se aplicará como versión mínima para las comunicaciones HTTPS. Asegúrese de usar esta versión de TLS en el código.  En breves las versiones 1.0 y 1.1 de TLS quedarán en desuso.

### <a name="subscription-apis"></a>API Subscription

#### <a name="resolve-a-purchased-subscription"></a>Resolución de una suscripción comprada

El punto de conexión de resolución permite al editor cambiar el token de identificación de la compra de Marketplace (conocido como *token* en [Comprada pero todavía no activada](#purchased-but-not-yet-activated-pendingfulfillmentstart)) a un identificador de suscripción de SaaS comprado persistente y sus detalles.

Cuando se redirige a un cliente a la dirección URL de la página de aterrizaje del partner, se pasa el token de identificación del cliente como parámetro *token* en esta llamada a la URL. Se espera que el partner use este token y haga una solicitud para resolverlo. La respuesta de la API de resolución contiene el identificador de la suscripción de SaaS y otros detalles para identificar la compra de forma única. El *token* proporcionado con la llamada a la dirección URL de la página de aterrizaje suele ser válido durante 24 horas. Si el *token* que recibe ya ha expirado, le recomendamos que proporcione las instrucciones siguientes al cliente final:

"No pudimos identificar esta compra. Vuelva a abrir esta suscripción de SaaS en Azure Portal o en el Centro de administración de M365 y haga clic de nuevo en el botón 'Configurar cuenta' o 'Administrar cuenta'".

Al llamar a la API Resolve, se devolverán los detalles y el estado de la suscripción para las suscripciones de SaaS en todos los estados admitidos.

##### <a name="posthttpsmarketplaceapimicrosoftcomapisaassubscriptionsresolveapi-versionapiversion"></a>Post`https://marketplaceapi.microsoft.com/api/saas/subscriptions/resolve?api-version=<ApiVersion>`

*Parámetros de consulta*:

|  Parámetro         | Value            |
|  ---------------   |  ---------------  |
|  `ApiVersion`        |  Use 2018-08-31.   |

*Encabezados de la solicitud*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `content-type`      | `application/json` |
|  `x-ms-requestid`    |  Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
|  `x-ms-correlationid` |  Valor de cadena único para la operación en el cliente. Este parámetro pone en correlación todos los eventos de la operación del cliente con los eventos del servidor. Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  `authorization`     |  Token de acceso único que identifica al editor que realiza esta llamada API. El formato es `"Bearer <accessaccess_token>"` cuando el editor recupera el valor del token, tal como se explica en [Obtención de un token basado en la aplicación de Azure AD](./pc-saas-registration.md#get-the-token-with-an-http-post). |
|  `x-ms-marketplace-token`  | Parámetro *token* de identificación de la compra de Marketplace que se va a resolver.  El token se pasa en la llamada a la dirección URL de la página de aterrizaje cuando se redirige al cliente al sitio web del partner de SaaS (por ejemplo, `https://contoso.com/signup?token=<token><authorization_token>`). <br> <br>  *Nota:* El valor de *token* que se está codificando forma parte de la dirección URL de la página de aterrizaje y, por tanto, debe descodificarse antes de usarse como un parámetro en esta llamada API.  <br> <br> Un ejemplo de una cadena codificada en la dirección URL tiene el siguiente aspecto: `contoso.com/signup?token=ab%2Bcd%2Fef`, donde el token es `ab%2Bcd%2Fef`.  El mismo token descodificado será `Ab+cd/ef`. |
| | |

*Códigos de respuesta*:

Código: 200 Devuelve identificadores de suscripción de SaaS únicos basados en el `x-ms-marketplace-token` proporcionado.

Ejemplo de cuerpo de respuesta:

```json
{
    "id": "<guid>",  // purchased SaaS subscription ID 
    "subscriptionName": "Contoso Cloud Solution", // SaaS subscription name 
    "offerId": "offer1", // purchased offer ID
    "planId": "silver", // purchased offer's plan ID
    "quantity": "20", // number of purchased seats, might be empty if the plan is not per seat
    "subscription": { // full SaaS subscription details, see Get Subscription APIs response body for full description
    "id": "<guid>",
    "publisherId": "contoso",
    "offerId": "offer1",
    "name": "Contoso Cloud Solution",
    "saasSubscriptionStatus": " PendingFulfillmentStart ",
    "beneficiary": {
      "emailId": "test@test.com",
      "objectId": "<guid>",
      "tenantId": "<guid>",
"pid": "<ID of the user>"
    },
    "purchaser": {
      "emailId": "test@test.com",
      "objectId": "<guid>",
      "tenantId": "<guid>",
"pid": "<ID of the user>"
    },
    "planId": "silver",
    "term": {
      "termUnit": "P1M",
                   startDate": "2019-05-31", 
   "endDate": "2019-06-29",
    },
    "isTest": true,
    "isFreeTrial": false,
    "allowedCustomerOperations": [
      "Delete",
      "Update",
      "Read"
    ],
    "sandboxType": "None",
    "sessionMode": "None"
  }
}

```

Código: 400 Solicitud incorrecta. Falta el `x-ms-marketplace-token`, o bien tiene un formato incorrecto, no es válido o ha expirado.

Código: 403 Prohibido. El token de autorización no es válido, ha expirado o no se ha proporcionado.  La solicitud intenta acceder a una suscripción de SaaS para una oferta que se publicó con un identificador de aplicación de Azure AD diferente del que se usó para crear el token de autorización.

Este error suele indicar que no se ha realizado correctamente el [registro de SaaS](pc-saas-registration.md).

Código: 500 Error interno del servidor.  Vuelva a intentar la llamada API.  Si el error persiste, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://partner.microsoft.com/support/v2/?stage=1).

#### <a name="activate-a-subscription"></a>Activar una suscripción

Una vez que se ha configurado la cuenta de SaaS para un cliente final, el editor debe llamar a la API Activate Subscription en el lado de Microsoft.  No se facturará al cliente a menos que esta llamada API sea correcta.

##### <a name="posthttpsmarketplaceapimicrosoftcomapisaassubscriptionssubscriptionidactivateapi-versionapiversion"></a>Post`https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/activate?api-version=<ApiVersion>`

*Parámetros de consulta*:

|  Parámetro         | Value             |
|  --------   |  ---------------  |
| `ApiVersion`  |  Use 2018-08-31.   |
| `subscriptionId` | Identificador único de la suscripción de SaaS comprada.  Este identificador se obtiene después de resolver el token de autorización de Marketplace mediante la [API Resolve](#resolve-a-purchased-subscription).
 |

*Encabezados de la solicitud*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
| `content-type`       |  `application/json`  |
| `x-ms-requestid`     |  Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
| `x-ms-correlationid` |  Valor de cadena único para la operación en el cliente.  Esta cadena pone en correlación todos los eventos de la operación en el cliente con los eventos en el servidor.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
| `authorization`      |  Token de acceso único que identifica al editor que realiza esta llamada API. El formato es `"Bearer <access_token>"` cuando el editor recupera el valor del token, tal como se explica en [Obtención de un token basado en la aplicación de Azure AD](./pc-saas-registration.md#get-the-token-with-an-http-post). |

*Ejemplo de solicitud de carga:*

```json
{ // needed for validation of the activation request
    "planId": "gold", // purchased plan, cannot be empty
    "quantity": "" // purchased number of seats, can be empty if plan is not per seat
}
```

*Códigos de respuesta*:

Código: 200 La suscripción se marcó como Suscrita en el lado de Microsoft.

No hay ningún cuerpo de respuesta para esta llamada.

Código: 400 Solicitud incorrecta: error de validación.

* El valor de `planId` no existe en la carga de la solicitud.
* El valor de `planId` de la carga de la solicitud no coincide con el de la compra.
* El valor de `quantity` de la carga de la solicitud no coincide con el de la compra.
* La suscripción de SaaS se encuentra en el estado Suscrito o Suspendido.

Código: 403 Prohibido. El token de autorización no es válido, ha expirado o no se ha proporcionado. La solicitud intenta acceder a una suscripción de SaaS para una oferta que se publicó con un identificador de aplicación de Azure AD diferente del que se usó para crear el token de autorización.

Este error suele indicar que no se ha realizado correctamente el [registro de SaaS](pc-saas-registration.md).

Código: 404 No encontrado. La suscripción de SaaS se encuentra en el estado Suscripción cancelada.

Código: 500 Error interno del servidor.  Vuelva a intentar la llamada API.  Si el error persiste, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://partner.microsoft.com/support/v2/?stage=1).

#### <a name="get-list-of-all-subscriptions"></a>Obtención de una lista de todas las suscripciones

Recupera una lista de todas las suscripciones de SaaS compradas para todas las ofertas que el editor ha publicado en Marketplace.  Se devolverán las suscripciones de SaaS en todos los estados posibles. También se devuelven las suscripciones de SaaS canceladas, ya que esta información no se elimina en el lado de Microsoft.

Esta API devuelve resultados paginados. El tamaño de la página es de 100.

##### <a name="gethttpsmarketplaceapimicrosoftcomapisaassubscriptionsapi-versionapiversion"></a>Get`https://marketplaceapi.microsoft.com/api/saas/subscriptions?api-version=<ApiVersion>`

*Parámetros de consulta*:

|  Parámetro         | Value             |
|  --------   |  ---------------  |
| `ApiVersion`  |  Use 2018-08-31.  |
| `continuationToken`  | Parámetro opcional. Para recuperar la primera página de resultados, déjelo vacío.  Use el valor devuelto en el parámetro `@nextLink` para recuperar la página siguiente. |

*Encabezados de la solicitud*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
| `content-type`       |  `application/json`  |
| `x-ms-requestid`     |  Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
| `x-ms-correlationid` |  Valor de cadena único para la operación en el cliente.  Este parámetro pone en correlación todos los eventos de la operación del cliente con los eventos del servidor.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
| `authorization`      |  Token de acceso único que identifica al editor que realiza esta llamada API.  El formato es `"Bearer <access_token>"` cuando el editor recupera el valor del token, tal como se explica en [Obtención de un token basado en la aplicación de Azure AD](./pc-saas-registration.md#get-the-token-with-an-http-post). |

*Códigos de respuesta*:

Código: 200 Devuelve la lista de todas las suscripciones existentes para todas las ofertas de este editor, en función del token de autorización del editor.

*Ejemplo de cuerpo de respuesta:*

```json
{
  "subscriptions": [
      {
          "id": "<guid>", // purchased SaaS subscription ID
          "name": "Contoso Cloud Solution", // SaaS subscription name
          "publisherId": "contoso", // publisher ID
          "offerId": "offer1", // purchased offer ID
          "planId": "silver", // purchased plan ID
          "quantity": "10", // purchased amount of seats, will be empty if plan is not per seat
          "beneficiary": { // email address, user ID and tenant ID for which SaaS subscription was purchased.
              "emailId": " test@contoso.com",
              "objectId": "<guid>",
              "tenantId": "<guid>",
              "pid": "<ID of the user>"
          },
          "purchaser": { // email address, user ID and tenant ID that purchased the SaaS subscription. These could be different from beneficiary information for reseller (CSP) purchase
              "emailId": " test@contoso.com",
              "objectId": "<guid>",
              "tenantId": "<guid>",
              "pid": "<ID of the user>"
          },
            "term": { // The period for which the subscription was purchased. 
                "startDate": "2019-05-31", //format: YYYY-MM-DD. This is the date when the subscription was activated by the ISV and the billing started. This field is relevant only for Active and Suspended subscriptions.
                "endDate": "2019-06-30", // This is the last day the subscription is valid. Unless stated otherwise, the automatic renew will happen the next day. This field is relevant only for Active and Suspended subscriptions.
                "termUnit": "P1M" // where P1M is monthly and P1Y is yearly. Also reflected in the startDate and endDate values
          },
          "allowedCustomerOperations": [
              "Read", "Update", "Delete" 
          ], // Indicates operations allowed on the SaaS subscription for beneficiary. For CSP-initiated purchases, this will always be "Read" because the customer cannot update or delete subscription in this flow.  Purchaser can perform all operations on the subscription.
          "sessionMode": "None", // not relevant
          "isFreeTrial": true, // true - the customer subscription is currently in free trial, false - the customer subscription is not currently in free trial. (Optional field -– if not returned, the value is false.)
          "isTest": false, // not relevant
          "sandboxType": "None", // not relevant
          "saasSubscriptionStatus": "Subscribed" // Indicates the status of the operation. Can be one of the following: PendingFulfillmentStart, Subscribed, Suspended or Unsubscribed.
      },
// next SaaS subscription details, might be a different offer
{
          "id": "<guid1>", 
          "name": "Contoso Cloud Solution1", 
          "publisherId": "contoso", 
          "offerId": "offer2", 
          "planId": "gold", 
          "quantity": "", 
          "beneficiary": {
              "emailId": " test@contoso.com",
              "objectId": "<guid>",
              "tenantId": "<guid>",
              "pid": "<ID of the user>"
          },
          "purchaser": { 
              "emailId": "purchase@csp.com ",
              "objectId": "<guid>",
              "tenantId": "<guid>",
               "pid": "<ID of the user>"
          },
            "term": { 
                "startDate": "2019-05-31",
                "endDate": "2020-04-30",
                "termUnit": "P1Y"
          },
          "allowedCustomerOperations": [
              "Read" 
          ], 
          "sessionMode": "None",
          "isFreeTrial": false,
          "isTest": false,
          "sandboxType": "None",
          "saasSubscriptionStatus": "Suspended"
      }
  ],
  "@nextLink": "https:// https://marketplaceapi.microsoft.com/api/saas/subscriptions/?continuationToken=%5b%7b%22token%22%3a%22%2bRID%3a%7eYeUDAIahsn22AAAAAAAAAA%3d%3d%23RT%3a1%23TRC%3a2%23ISV%3a1%23FPC%3aAgEAAAAQALEAwP8zQP9%2fFwD%2b%2f2FC%2fwc%3d%22%2c%22range%22%3a%7b%22min%22%3a%22%22%2c%22max%22%3a%2205C1C9CD673398%22%7d%7d%5d&api-version=2018-08-31" // url that contains continuation token to retrieve next page of the SaaS subscriptions list, if empty or absent, this is the last page. ISV can use this url as is to retrieve the next page or extract the value of continuation token from this url.
}
```

Si no se encuentra ninguna suscripción de SaaS comprada para este editor, el cuerpo de respuesta se devuelve vacío.

Código: 403 Prohibido. El token de autorización no está disponible, no es válido o ha expirado.

Este error suele indicar que no se ha realizado correctamente el [registro de SaaS](pc-saas-registration.md). 

Código: 500 Error interno del servidor. Vuelva a intentar la llamada API.  Si el error persiste, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://partner.microsoft.com/support/v2/?stage=1).

#### <a name="get-subscription"></a>Obtener una suscripción

Recupera una determinada suscripción de SaaS comprada para una oferta de SaaS que el editor ha publicado en Marketplace. Use esta llamada para obtener toda la información disponible para una suscripción de SaaS específica por su identificador, en lugar de llamar a la API para obtener una lista de todas las suscripciones.

##### <a name="get-httpsmarketplaceapimicrosoftcomapisaassubscriptionssubscriptionidapi-versionapiversion"></a>Get `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>?api-version=<ApiVersion>`

*Parámetros de consulta*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
| `ApiVersion`        |   Use 2018-08-31. |
| `subscriptionId`     |  Identificador único de la suscripción de SaaS comprada.  Este identificador se obtiene después de resolver el token de autorización de Marketplace mediante la API Resolve. |

*Encabezados de la solicitud*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `content-type`      |  `application/json`  |
|  `x-ms-requestid`    |  Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
|  `x-ms-correlationid` |  Valor de cadena único para la operación en el cliente.  Este parámetro pone en correlación todos los eventos de la operación del cliente con los eventos del servidor.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
|  `authorization`     | Token de acceso único que identifica al editor que realiza esta llamada API. El formato es `"Bearer <access_token>"` cuando el editor recupera el valor del token, tal como se explica en [Obtención de un token basado en la aplicación de Azure AD](./pc-saas-registration.md#get-the-token-with-an-http-post).  |

*Códigos de respuesta*:

Código: 200 Devuelve los detalles de una suscripción de SaaS en función del valor de `subscriptionId` proporcionado.

*Ejemplo de cuerpo de respuesta:*

```json
{
        "id":<guid>, // purchased SaaS subscription ID
        "name":"Contoso Cloud Solution", // SaaS subscription name
         "publisherId": "contoso", // publisher ID
          "offerId": "offer1", // purchased offer ID
          "planId": "silver", // purchased plan ID
          "quantity": "10", // purchased amount of seats, will be empty if plan is not per seat
         "beneficiary": { // email address, user ID and tenant ID for which SaaS subscription is purchased.
              "emailId": "test@contoso.com",
              "objectId": "<guid>",
              "tenantId": "<guid>",
              "pid": "<ID of the user>"
          },
          "purchaser": { // email address ,user ID and tenant ID that purchased the SaaS subscription.  These could be different from beneficiary information for reseller (CSP) scenario
              "emailId": "test@test.com",
              "objectId": "<guid>",
              "tenantId": "<guid>",
              "pid": "<ID of the user>"
          },
        "allowedCustomerOperations": ["Read", "Update", "Delete"], // Indicates operations allowed on the SaaS subscription for beneficiary.  For CSP-initiated purchases, this will always be "Read" because the customer cannot update or delete subscription in this flow.  Purchaser can perform all operations on the subscription.
        "sessionMode": "None", // not relevant
        "isFreeTrial": false, // true - the customer subscription is currently in free trial, false - the customer subscription is not currently in free trial. Optional field – if not returned the value is false.
          "isTest": false, // not relevant
          "sandboxType": "None", // not relevant
          "saasSubscriptionStatus": " Subscribed " // Indicates the status of the operation: PendingFulfillmentStart, Subscribed, Suspended or Unsubscribed.
          "term": { // the period for which the subscription was purchased 
            "startDate": "2019-05-31", //format: YYYY-MM-DD. This is the date when the subscription was activated by the ISV and the billing started. This field is relevant only for Active and Suspended subscriptions.
            "endDate": "2019-06-29", // This is the last day the subscription is valid. Unless stated otherwise, the automatic renew will happen the next day. This field is relevant only for Active and Suspended subscriptions.
            "termUnit": "P1M" //where P1M is monthly and P1Y is yearly. Also reflected in the startDate and endDate values.
        }
}
```

Código: 403 Prohibido. El token de autorización no es válido, ha expirado o no se ha proporcionado. La solicitud intenta acceder a una suscripción de SaaS para una oferta que se publicó con un identificador de aplicación de Azure AD diferente del que se usó para crear el token de autorización.

Este error suele indicar que no se ha realizado correctamente el [registro de SaaS](pc-saas-registration.md). 

Código: 404 No encontrado.  No se encuentra la suscripción de SaaS con el valor de `subscriptionId` especificado.

Código: 500 Error interno del servidor.  Vuelva a intentar la llamada API.  Si el error persiste, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://partner.microsoft.com/support/v2/?stage=1).

#### <a name="list-available-plans"></a>Lista de planes disponibles

Recupera todos los planes de una oferta de SaaS identificados por el valor `subscriptionId` de una compra específica de esta oferta.  Use esta llamada para obtener una lista de todos los planes públicos y privados que el beneficiario de una suscripción de SaaS puede actualizar para la suscripción.  Los planes devueltos estarán disponibles en la misma geografía que el plan comprado.

Esta llamada devuelve una lista de los planes disponibles para ese cliente, además de los que se han comprado.  La lista se puede presentar a un cliente final en el sitio del editor.  Un cliente final puede cambiar el plan de suscripción a uno de los planes de la lista devuelta.  Se producirá un error si se cambia el plan a uno que no aparezca en la lista.

##### <a name="get-httpsmarketplaceapimicrosoftcomapisaassubscriptionssubscriptionidlistavailableplansapi-versionapiversion"></a>Get `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/listAvailablePlans?api-version=<ApiVersion>`

*Parámetros de consulta*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `ApiVersion`        |  Use 2018-08-31.  |
|  `subscriptionId`    |  Identificador único de la suscripción de SaaS comprada.  Este identificador se obtiene después de resolver el token de autorización de Marketplace mediante la API Resolve. |

*Encabezados de la solicitud*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|   `content-type`     |  `application/json` |
|   `x-ms-requestid`   |  Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
|  `x-ms-correlationid`  |  Valor de cadena único para la operación en el cliente.  Este parámetro pone en correlación todos los eventos de la operación del cliente con los eventos del servidor.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
|  `authorization`     |  Token de acceso único que identifica al editor que realiza esta llamada API.  El formato es `"Bearer <access_token>"` cuando el editor recupera el valor del token, tal como se explica en [Obtención de un token basado en la aplicación de Azure AD](./pc-saas-registration.md#get-the-token-with-an-http-post).  |

*Códigos de respuesta*:

Código: 200 Devuelve una lista de todos los planes disponibles para una suscripción de SaaS existente, incluido el que se ha comprado.

Ejemplo de cuerpo de respuesta:

```json
{
    "plans": [{
        "planId": "Platinum001",
        "displayName": "Private platinum plan for Contoso", // display name of the plan as it appears in the marketplace
        "isPrivate": true //true or false
    },
{ 
        "planId": "gold",
        "displayName": "Gold plan for Contoso", 
        "isPrivate": false //true or false
    }
]
}
```

Si no se encuentra `subscriptionId`, el cuerpo de respuesta se devuelve vacío.

Código: 403 Prohibido. El token de autorización no es válido, ha expirado o no se ha proporcionado.  La solicitud podría intentar acceder a una suscripción de SaaS para una oferta que se publicó con un identificador de aplicación de Azure AD diferente del que se usó para crear el token de autorización.

Este error suele indicar que no se ha realizado correctamente el [registro de SaaS](pc-saas-registration.md). 

Código: 500 Error interno del servidor.  Vuelva a intentar la llamada API.  Si el error persiste, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://partner.microsoft.com/support/v2/?stage=1).

#### <a name="change-the-plan-on-the-subscription"></a>Cambiar el plan de la suscripción

Actualice el plan existente que se ha comprado para una suscripción de SaaS a un nuevo plan (público o privado).  El editor debe llamar a esta API cuando se cambia un plan en el lado del editor para una suscripción de SaaS comprada en Marketplace.

Solo se puede llamar a esta API para suscripciones activas.  Un plan se puede cambiar a cualquier otro plan existente (público o privado), pero no a sí mismo.  En el caso de los planes privados, el inquilino del cliente debe definirse como parte del público del plan en el Centro de partners.

##### <a name="patch-httpsmarketplaceapimicrosoftcomapisaassubscriptionssubscriptionidapi-versionapiversion"></a>Patch `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>?api-version=<ApiVersion>`

*Parámetros de consulta*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `ApiVersion`        |  Use 2018-08-31.  |
| `subscriptionId`     | Identificador único de la suscripción de SaaS comprada.  Este identificador se obtiene después de resolver el token de autorización de Marketplace mediante la API Resolve. |

*Encabezados de la solicitud*:
 
|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `content-type`      | `application/json`  |
|  `x-ms-requestid`    | Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID. Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  `x-ms-correlationid`  | Valor de cadena único para la operación en el cliente.  Este parámetro pone en correlación todos los eventos de la operación del cliente con los eventos del servidor.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  `authorization`     |  Token de acceso único que identifica al editor que realiza esta llamada API.  El formato es `"Bearer <access_token>"` cuando el editor recupera el valor del token, tal como se explica en [Obtención de un token basado en la aplicación de Azure AD](./pc-saas-registration.md#get-the-token-with-an-http-post). |

*Ejemplo de solicitud de carga:*

```json
{
    "planId": "gold" // the ID of the new plan to be purchased
}
```

*Códigos de respuesta*:

Código: 202 Se ha aceptado la solicitud para cambiar el plan y se ha administrado de manera asincrónica.  Se espera que el partner sondee la **dirección URL de Operation-Location** para determinar si la solicitud de cambio de plan se ha realizado correctamente o no.  El sondeo debe realizarse cada pocos segundos hasta que se reciba el estado final de Error, Correcto o Conflicto para la operación.  El estado de la operación final debe devolverse rápidamente, pero en algunos casos puede tardar varios minutos.

El partner también recibirá una notificación de webhook cuando la acción esté lista para completarse correctamente en el lado de Marketplace.  Solo entonces debe el editor cambiar el plan en su lado.

*Encabezados de respuesta:*

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `Operation-Location`        |  Dirección URL para obtener el estado de la operación.  Por ejemplo, `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/operations/<operationId>?api-version=2018-08-31`. |

Código: 400 Solicitud incorrecta: errores de validación.

* El nuevo plan no existe o no está disponible para esta suscripción de SaaS específica.
* Se está intentando cambiar al mismo plan.
* El estado de la suscripción de SaaS es Suscripción cancelada.
* La operación de actualización de una suscripción de SaaS no se incluye en `allowedCustomerOperations`.

Código: 403 Prohibido. El token de autorización no es válido, ha expirado o no se ha proporcionado.  La solicitud intenta acceder a una suscripción de SaaS para una oferta que se publicó con un identificador de aplicación de Azure AD diferente del que se usó para crear el token de autorización.

Este error suele indicar que no se ha realizado correctamente el [registro de SaaS](pc-saas-registration.md).

Código: 404 No encontrado.  No se encuentra la suscripción de SaaS con el valor de `subscriptionId`.

Código: 500 Error interno del servidor.  Vuelva a intentar la llamada API.  Si el error persiste, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://partner.microsoft.com/support/v2/?stage=1).

>[!NOTE]
>Se puede cambiar el plan o la cantidad de puestos de cada vez, pero no ambos.

>[!Note]
>Solo se puede llamar a esta API después de obtener la aprobación explícita del cliente final para el cambio.

#### <a name="change-the-quantity-of-seats-on-the-saas-subscription"></a>Cambio de la cantidad de puestos de la suscripción de SaaS

Actualice (es decir, aumente o disminuya) la cantidad de puestos comprados para una suscripción de SaaS.  El editor debe llamar a esta API cuando se cambia el número de puestos en el lado del editor para una suscripción de SaaS creada en Marketplace.

La cantidad de puestos no puede ser superior al número permitido en el plan actual.  En este caso, se debe cambiar el plan antes de cambiar la cantidad.

##### <a name="patchhttpsmarketplaceapimicrosoftcomapisaassubscriptionssubscriptionidapi-versionapiversion"></a>Patch`https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>?api-version=<ApiVersion>`

*Parámetros de consulta*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `ApiVersion`        |  Use 2018-08-31.  |
|  `subscriptionId`     | Identificador único de la suscripción de SaaS comprada.  Este identificador se obtiene después de resolver el token de autorización de Marketplace mediante la API Resolve.  |

*Encabezados de la solicitud*:
 
|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `content-type`      | `application/json`  |
|  `x-ms-requestid`    | Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  `x-ms-correlationid`  | Valor de cadena único para la operación en el cliente.  Este parámetro pone en correlación todos los eventos de la operación del cliente con los eventos del servidor.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  `authorization`     | Token de acceso único que identifica al editor que realiza esta llamada API.  El formato es `"Bearer <access_token>"` cuando el editor recupera el valor del token, tal como se explica en [Obtención de un token basado en la aplicación de Azure AD](./pc-saas-registration.md#get-the-token-with-an-http-post).  |

*Ejemplo de solicitud de carga:*

```json
{
    "quantity": 5 // the new amount of seats to be purchased
}
```

*Códigos de respuesta*:

Código: 202 Se ha aceptado la solicitud para cambiar la cantidad y se ha administrado de manera asincrónica. Se espera que el partner sondee la **dirección URL de Operation-Location** para determinar si la solicitud de cambio de cantidad se ha realizado correctamente o no.  El sondeo debe realizarse cada pocos segundos hasta que se reciba el estado final de Error, Correcto o Conflicto para la operación.  El estado de la operación final debe devolverse rápidamente, pero en algunos casos puede tardar varios minutos.

El partner también recibirá una notificación de webhook cuando la acción esté lista para completarse correctamente en el lado de Marketplace.  Solo entonces debe el editor cambiar la cantidad en su lado.

*Encabezados de respuesta:*

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `Operation-Location`        |  Vínculo a un recurso para obtener el estado de la operación.  Por ejemplo, `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/operations/<operationId>?api-version=2018-08-31`.  |

Código: 400 Solicitud incorrecta: errores de validación.

* La nueva cantidad es superior o inferior al límite del plan actual.
* Falta la nueva cantidad.
* Se está intentando cambiar a la misma cantidad.
* El estado de la suscripción de SaaS es Suscripción cancelada.
* La operación de actualización de una suscripción de SaaS no se incluye en `allowedCustomerOperations`.

Código: 403 Prohibido.  El token de autorización no es válido, ha expirado o no se ha proporcionado.  La solicitud está intentando acceder a una suscripción que no pertenece al editor actual.

Este error suele indicar que no se ha realizado correctamente el [registro de SaaS](pc-saas-registration.md). 

Código: 404 No encontrado.  No se encuentra la suscripción de SaaS con el valor de `subscriptionId`.

Código: 500 Error interno del servidor.  Vuelva a intentar la llamada API.  Si el error persiste, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://partner.microsoft.com/support/v2/?stage=1).

>[!Note]
>Solo se puede cambiar un plan o cantidad a la vez, pero no ambos.

>[!Note]
>Solo se puede llamar a esta API después de obtener la aprobación explícita del cliente final para el cambio.

#### <a name="cancel-a-subscription"></a>Cancelación de una suscripción

Cancele una suscripción de SaaS especificada.  No es necesario que el editor use esta API. Se recomienda redirigir a los clientes a Marketplace para cancelar las suscripciones de SaaS.

Si el editor decide implementar la cancelación de la suscripción de SaaS comprada en Marketplace en el lado del editor, debe llamar a esta API.  Después de la finalización de esta llamada, el estado de la suscripción se convertirá en *Suscripción cancelada* en el lado de Microsoft.

Si se cancela una suscripción dentro de los períodos de gracia siguientes, no se facturará al cliente:

* 24 horas en el caso de una suscripción mensual tras la activación.
* 14 días en el caso de una suscripción anual tras la activación.

Se facturará al cliente si la suscripción se cancela tras los períodos de gracia anteriores.  Una vez que la cancelación se ha realizado correctamente, el cliente perderá inmediatamente el acceso a la suscripción de SaaS en el lado de Microsoft.

##### <a name="deletehttpsmarketplaceapimicrosoftcomapisaassubscriptionssubscriptionidapi-versionapiversion"></a>Delete`https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>?api-version=<ApiVersion>`

*Parámetros de consulta*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `ApiVersion`        |  Use 2018-08-31.  |
|  `subscriptionId`     | Identificador único de la suscripción de SaaS comprada.  Este identificador se obtiene después de resolver el token de autorización de Marketplace mediante la API Resolve.  |

*Encabezados de la solicitud*:
 
|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `content-type`      | `application/json`  |
|  `x-ms-requestid`    | Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  `x-ms-correlationid`  | Valor de cadena único para la operación en el cliente.  Este parámetro pone en correlación todos los eventos de la operación del cliente con los eventos del servidor.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  `authorization`     |  Token de acceso único que identifica al editor que realiza esta llamada API.  El formato es `"Bearer <access_token>"` cuando el editor recupera el valor del token, tal como se explica en [Obtención de un token basado en la aplicación de Azure AD](./pc-saas-registration.md#get-the-token-with-an-http-post). |

*Códigos de respuesta*:

Código: 202 Se ha aceptado la solicitud para cancelar la suscripción y se ha administrado de manera asincrónica.  Se espera que el partner sondee la **dirección URL de Operation-Location** para determinar si la solicitud se ha realizado correctamente o no.  El sondeo debe realizarse cada pocos segundos hasta que se reciba el estado final de Error, Correcto o Conflicto para la operación.  El estado de la operación final debe devolverse rápidamente, pero en algunos casos puede tardar varios minutos.

El partner también recibirá una notificación de webhook cuando la acción se haya completado correctamente en el lado de Marketplace.  Solo entonces debe el editor cancelar la suscripción en su lado.

*Encabezados de respuesta:*

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `Operation-Location`        |  Vínculo a un recurso para obtener el estado de la operación.  Por ejemplo, `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/operations/<operationId>?api-version=2018-08-31`. |

Código: 400 Solicitud incorrecta.  La operación de eliminación no está en la lista `allowedCustomerOperations` para esta suscripción de SaaS.

Código: 403 Prohibido.  El token de autorización no es válido, ha expirado o no está disponible. La solicitud intenta acceder a una suscripción de SaaS para una oferta que se publicó con un identificador de aplicación de Azure AD diferente del que se usó para crear el token de autorización.

Este error suele indicar que no se ha realizado correctamente el [registro de SaaS](pc-saas-registration.md).

Código: 404 No encontrado.  No se encuentra la suscripción de SaaS con el valor de `subscriptionId`.

Código: 500 Error interno del servidor. Vuelva a intentar la llamada API.  Si el error persiste, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://partner.microsoft.com/support/v2/?stage=1).

### <a name="operations-apis"></a>API Operations

#### <a name="list-outstanding-operations"></a>Lista de operaciones pendientes 

Obtiene una lista de las operaciones pendientes para la suscripción de SaaS especificada.  El editor debe confirmar las operaciones devueltas mediante una llamada a la [API Operation Patch](#update-the-status-of-an-operation).

Actualmente, solo se devuelven las **operaciones de restablecimiento** como respuesta para esta llamada API.

##### <a name="get-httpsmarketplaceapimicrosoftcomapisaassubscriptionssubscriptionidoperationsapi-versionapiversion"></a>Get `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/operations?api-version=<ApiVersion>`

*Parámetros de consulta*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|    `ApiVersion`    |  Use 2018-08-31.         |
|    `subscriptionId` | Identificador único de la suscripción de SaaS comprada.  Este identificador se obtiene después de resolver el token de autorización de Marketplace mediante la API Resolve.  |

*Encabezados de la solicitud*:
 
|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `content-type`     |  `application/json` |
|  `x-ms-requestid`    |  Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  `x-ms-correlationid` |  Valor de cadena único para la operación en el cliente.  Este parámetro pone en correlación todos los eventos de la operación del cliente con los eventos del servidor.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  `authorization`     |  El formato es `"Bearer <access_token>"` cuando el editor recupera el valor del token, tal como se explica en [Obtención de un token basado en la aplicación de Azure AD](./pc-saas-registration.md#get-the-token-with-an-http-post).  |

*Códigos de respuesta*:

Código: 200 Devuelve una operación de restablecimiento pendiente en la suscripción de SaaS especificada.

*Ejemplo de carga de respuesta:*

```json
{"operations": [{
    "id": "<guid>",  //Operation ID, should be provided in the operations patch API call
    "activityId": "<guid>", //not relevant
    "subscriptionId": "<guid>", // subscriptionId of the SaaS subscription that is being reinstated
    "offerId": "offer1",  // purchased offer ID
    "publisherId": "contoso",  
    "planId": "silver",  // purchased plan ID
    "quantity": "20", // purchased amount of seats, will be empty is not relevant
    "action": "Reinstate", 
    "timeStamp": "2018-12-01T00:00:00",  // UTC
    "status": "InProgress" // the only status that can be returned in this case
}]
}
```

Devuelve JSON vacío si no hay ninguna operación de restablecimiento pendiente.

Código: 400 Solicitud incorrecta: errores de validación.

Código: 403 Prohibido. El token de autorización no es válido, ha expirado o no se ha proporcionado.  La solicitud intenta acceder a una suscripción de SaaS para una oferta que se publicó con un identificador de aplicación de Azure AD diferente del que se usó para crear el token de autorización.

Este error suele indicar que no se ha realizado correctamente el [registro de SaaS](pc-saas-registration.md). 

Código: 404 No encontrado.  No se encuentra la suscripción de SaaS con el valor de `subscriptionId`.

Código: 500 Error interno del servidor. Vuelva a intentar la llamada API.  Si el error persiste, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://partner.microsoft.com/support/v2/?stage=1).

#### <a name="get-operation-status"></a>Obtener el estado de la operación

Permite al editor hacer un seguimiento del estado de la operación asincrónica especificada:  **Unsubscribe**, **ChangePlan** o **ChangeQuantity**.

El valor de `operationId` para esta llamada API se puede recuperar a partir del valor devuelto por **Operation-Location**, la llamada API para obtener operaciones pendientes o el valor del parámetro `<id>` recibido en una llamada de webhook.

##### <a name="get-httpsmarketplaceapimicrosoftcomapisaassubscriptionssubscriptionidoperationsoperationidapi-versionapiversion"></a>Get `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/operations/<operationId>?api-version=<ApiVersion>`

*Parámetros de consulta*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `ApiVersion`        |  Use 2018-08-31.  |
|  `subscriptionId`    |  Identificador único de la suscripción de SaaS comprada.  Este identificador se obtiene después de resolver el token de autorización de Marketplace mediante la API Resolve. |
|  `operationId`       |  Identificador único de la operación que se va a recuperar. |

*Encabezados de la solicitud*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|  `content-type`      |  `application/json`   |
|  `x-ms-requestid`    |  Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
|  `x-ms-correlationid` |  Valor de cadena único para la operación en el cliente.  Este parámetro pone en correlación todos los eventos de la operación del cliente con los eventos del servidor.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta.  |
|  `authorization`     |  Token de acceso único que identifica al editor que realiza esta llamada API.  El formato es `"Bearer <access_token>"` cuando el editor recupera el valor del token, tal como se explica en [Obtención de un token basado en la aplicación de Azure AD](./pc-saas-registration.md#get-the-token-with-an-http-post).  |

*Códigos de respuesta*:

Código: 200 Obtiene los detalles de la operación de SaaS especificada. 

*Ejemplo de carga de respuesta:*

```json
Response body:
{
    "id  ": "<guid>", //Operation ID, should be provided in the patch operation API call
    "activityId": "<guid>", //not relevant
    "subscriptionId":"<guid>", // subscriptionId of the SaaS subscription for which this operation is relevant
    "offerId": "offer1", // purchased offer ID
    "publisherId": "contoso",  
    "planId": "silver", // purchased plan ID
    "quantity": "20", // purchased amount of seats 
    "action": "ChangePlan", // Can be ChangePlan, ChangeQuantity or Reinstate
    "timeStamp": "2018-12-01T00:00:00", // UTC
    "status": "InProgress", // Possible values: NotStarted, InProgress, Failed, Succeed, Conflict (new quantity / plan is the same as existing)

"errorStatusCode": "",
"errorMessage": ""
}
```

Código: 403 Prohibido. El token de autorización no es válido, ha expirado o no se ha proporcionado.  La solicitud intenta acceder a una suscripción de SaaS para una oferta que se publicó con un identificador de aplicación de Azure AD diferente del que se usó para crear el token de autorización.

Este error suele indicar que no se ha realizado correctamente el [registro de SaaS](pc-saas-registration.md). 

Código: 404 No encontrado.  

* No se encuentra la suscripción con el valor de `subscriptionId`.
* No se encuentra la operación con el valor de `operationId`.

Código: 500 Error interno del servidor.  Vuelva a intentar la llamada API.  Si el error persiste, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://partner.microsoft.com/support/v2/?stage=1).

#### <a name="update-the-status-of-an-operation"></a>Actualizar el estado de una operación

Actualice el estado de una operación pendiente para indicar si se ha realizado correctamente o no en el lado del editor.

El valor de `operationId` para esta llamada API se puede recuperar a partir del valor devuelto por **Operation-Location**, la llamada API para obtener operaciones pendientes o el valor del parámetro `<id>` recibido en una llamada de webhook.

##### <a name="patch-httpsmarketplaceapimicrosoftcomapisaassubscriptionssubscriptionidoperationsoperationidapi-versionapiversion"></a>Patch `https://marketplaceapi.microsoft.com/api/saas/subscriptions/<subscriptionId>/operations/<operationId>?api-version=<ApiVersion>`

*Parámetros de consulta*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|   `ApiVersion`       |  Use 2018-08-31.  |
|   `subscriptionId`   |  Identificador único de la suscripción de SaaS comprada.  Este identificador se obtiene después de resolver el token de autorización de Marketplace mediante la API Resolve.  |
|   `operationId`      |  Identificador único de la operación que se va a completar. |

*Encabezados de la solicitud*:

|  Parámetro         | Value             |
|  ---------------   |  ---------------  |
|   `content-type`   | `application/json`   |
|   `x-ms-requestid`   |  Valor de cadena único para el seguimiento de la solicitud del cliente, preferiblemente un GUID.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
|   `x-ms-correlationid` |  Valor de cadena único para la operación en el cliente.  Este parámetro pone en correlación todos los eventos de la operación del cliente con los eventos del servidor.  Si este valor no se proporciona, se generará uno y se proporcionará en los encabezados de respuesta. |
|  `authorization`     |  Token de acceso único que identifica al editor que realiza esta llamada API.  El formato es `"Bearer <access_token>"` cuando el editor recupera el valor del token, tal como se explica en [Obtención de un token basado en la aplicación de Azure AD](./pc-saas-registration.md#get-the-token-with-an-http-post). |

*Ejemplo de solicitud de carga:*

```json
{ 
    "status": "Success"    // Allowed Values: Success/Failure. Indicates the status of the operation on ISV side.
}
```

*Códigos de respuesta*:

Código: 200 Llamada para informar de la finalización de una operación por parte del partner.  Por ejemplo, esta respuesta podría la finalización del cambio de puestos o planes en el lado del editor.

Código: 403 Prohibido.  El token de autorización no está disponible, no es válido o ha expirado. La solicitud podría estar intentando acceder a una suscripción que no pertenece al editor actual.
Prohibido.  El token de autorización no es válido, ha expirado o no se ha proporcionado.  La solicitud intenta acceder a una suscripción de SaaS para una oferta que se publicó con un identificador de aplicación de Azure AD diferente del que se usó para crear el token de autorización.

Este error suele indicar que no se ha realizado correctamente el [registro de SaaS](pc-saas-registration.md).

Código: 404 No encontrado.

* No se encuentra la suscripción con el valor de `subscriptionId`.
* No se encuentra la operación con el valor de `operationId`.

Código: 409 Conflicto.  Por ejemplo, ya se suministra una actualización más reciente.

Código: 500 Error interno del servidor.  Vuelva a intentar la llamada API.  Si el error persiste, póngase en contacto con el [servicio de soporte técnico de Microsoft](https://partner.microsoft.com/support/v2/?stage=1).

## <a name="implementing-a-webhook-on-the-saas-service"></a>Implementación de un webhook en el servicio de SaaS

Al crear una oferta de SaaS comercializable en el Centro de partners, el partner proporciona la dirección URL del **webhook de conexión** que se va a usar como punto de conexión HTTP.  Microsoft llama a este webhook mediante la llamada HTTP POST para notificar al lado del editor los siguientes eventos que se producen en el lado de Microsoft:

* Cuando la suscripción de SaaS se encuentra en el estado Suscrito:
    * ChangePlan 
    * ChangeQuantity
    * Suspender
    * Cancelar suscripción
* Cuando la suscripción de SaaS se encuentra en el estado Suspendido:
    * Reinstate
    * Cancelar suscripción

El editor debe implementar un webhook en el servicio de SaaS para mantener la coherencia del estado de la suscripción de SaaS con el del lado de Microsoft.  El servicio de SaaS debe llamar a la API Get Operation para validar y autorizar la llamada de webhook y los datos de carga antes de realizar una acción basada en la notificación de webhook.  El editor debe devolver HTTP 200 a Microsoft en cuanto se procese la llamada de webhook.  Este valor confirma que el editor ha recibido correctamente la llamada de webhook.

>[!Note]
>El servicio de dirección URL de webhook debe estar en funcionamiento de forma ininterrumpida y poder recibir nuevas llamadas de Microsoft en todo momento.  Microsoft tiene una directiva de reintentos para la llamada de webhook (500 reintentos durante 8 horas), pero si el editor no acepta la llamada y devuelve una respuesta, la operación de la que informa el webhook producirá un error en el lado de Microsoft.

*Ejemplos de carga de webhook:*

```json
// end customer changed a quantity of purchased seats for a plan on Microsoft side
{
  "id": <guid>, // this is the operation ID to call with get operation API
  "activityId": "<guid>", // do not use
  "subscriptionId": "guid", // The GUID identifier for the SaaS resource which status changes
  "publisherId": "contoso", // A unique string identifier for each publisher
  "offerId": "offer1", // A unique string identifier for each offer
  "planId": "silver", // the most up-to-date plan ID
  "quantity": " 25", // the most up-to-date number of seats, can be empty if not relevant
  "timeStamp": "2019-04-15T20:17:31.7350641Z", // UTC time when the webhook was called
  "action": "ChangeQuantity", // the operation the webhook notifies about
  "status": "Success" // Can be either InProgress or Success  
}
```

```json
// end customer's payment instrument became valid again, after being suspended, and the SaaS subscription is being reinstated
{
  "id": <guid>, 
  "activityId": <guid>, 
  "subscriptionId": "guid", 
  "publisherId": "contoso",
  "offerId": "offer2 ",
  "planId": "gold", 
  "quantity": " 20", 
  "timeStamp": "2019-04-15T20:17:31.7350641Z",
  "action": "Reinstate",
  "status": "In Progress" 
} 
```

## <a name="development-and-testing"></a>Desarrollo y pruebas

Para iniciar el proceso de desarrollo, se recomienda crear respuestas de API ficticias en el lado del editor.  Pueden basarse en las respuestas de ejemplo que se proporcionan en este documento.

Cuando el editor esté listo para las pruebas de un extremo a otro: 

* Publique una oferta de SaaS para un público preliminar limitado y manténgalo en la fase de versión preliminar.
* Esta oferta debe tener un plan con un precio de 0 para que no se produzcan gastos de facturación reales durante las pruebas.  Otra opción consiste en establecer un precio distinto de cero y cancelar todas las compras de prueba durante un plazo de 24 horas. 
* Asegúrese de que todos los flujos se invoquen de extremo a extremo, como si un cliente comprase la oferta. 
* Si el partner quiere probar el flujo completo de compra y facturación, hágalo con una oferta cuyo precio sea superior a 0 USD.  La compra se facturará y se generará una factura.

Se puede desencadenar un flujo de compra desde Azure Portal o sitios de Microsoft AppSource, en función de dónde se publique la oferta.

Las acciones de *cambiar plan*, *cambiar cantidad* y *cancelar suscripción* se prueban desde el lado del editor.  En el lado de Microsoft, se pueden desencadenar la acción de *cancelar suscripción* desde Azure Portal y el Centro de administración (el portal donde se administran las compras de Microsoft AppSource).  Las acciones de *cambiar cantidad y plan* solo se pueden desencadenar desde el Centro de administración.

## <a name="get-support"></a>Obtención de soporte técnico

Consulte [Soporte técnico para el programa Marketplace comercial en el Centro de partners](support.md) para ver las opciones de soporte técnico para editores.


## <a name="next-steps"></a>Pasos siguientes

Consulte las [API del servicio de medición](marketplace-metering-service-apis.md) de Marketplace si quiere descubrir más opciones para las ofertas de SaaS en Marketplace.

Revise y use el [SDK de SaaS](https://github.com/Azure/Microsoft-commercial-marketplace-transactable-SaaS-offer-SDK) basado en las API descritas en este documento.
