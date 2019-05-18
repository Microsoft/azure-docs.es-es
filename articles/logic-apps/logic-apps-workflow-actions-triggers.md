---
title: Referencia para tipos de desencadenadores y las acciones en el lenguaje de definición de flujo de trabajo - Azure Logic Apps
description: Guía de referencia para tipos de desencadenadores y las acciones en el lenguaje de definición de flujo de trabajo para Azure Logic Apps
services: logic-apps
ms.service: logic-apps
author: ecfan
ms.author: estfan
ms.reviewer: klam, LADocs
ms.suite: integration
ms.topic: reference
ms.date: 05/13/2019
ms.openlocfilehash: aa5d3a0555875571276fdf4046ad0e4dd1e69bbd
ms.sourcegitcommit: 1fbc75b822d7fe8d766329f443506b830e101a5e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65596948"
---
# <a name="reference-for-trigger-and-action-types-in-workflow-definition-language-for-azure-logic-apps"></a>Referencia para tipos de desencadenadores y las acciones en el lenguaje de definición de flujo de trabajo para Azure Logic Apps

Esta referencia describe los tipos generales usados para identificar los desencadenadores y acciones en la definición de flujo de trabajo subyacente de la aplicación lógica, que se describe y se valida el [lenguaje de definición de flujo de trabajo](../logic-apps/logic-apps-workflow-definition-language.md).
Para buscar los desencadenadores de conector específico y las acciones que puede usar en las aplicaciones lógicas, vea la lista bajo el [Introducción a los conectores](https://docs.microsoft.com/connectors/).

<a name="triggers-overview"></a>

## <a name="triggers-overview"></a>Introducción a los desencadenadores

Cada flujo de trabajo incluye un desencadenador, que define las llamadas realizadas por crear una instancia e iniciar el flujo de trabajo. Estas son las categorías generales de desencadenadores:

* Un desencadenador de *sondeo*, que comprueba un punto de conexión de servicio a intervalos regulares

* Un desencadenador de *inserción*, que crea una suscripción a un punto de conexión y proporciona una *dirección URL de devolución de llamada* para que el punto de conexión pueda notificar al desencadenador cuando se produzca el evento especificado o estén disponibles los datos. El desencadenador espera la respuesta del punto de conexión antes de activarse. 

Los desencadenadores tienen estos elementos de nivel superior, aunque algunos son opcionales:  
  
```json
"<trigger-name>": {
   "type": "<trigger-type>",
   "inputs": { "<trigger-inputs>" },
   "recurrence": { 
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>
   },
   "conditions": [ "<array-with-conditions>" ],
   "runtimeConfiguration": { "<runtime-config-options>" },
   "splitOn": "<splitOn-expression>",
   "operationOptions": "<operation-option>"
},
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*trigger-name*> | string | El nombre del desencadenador | 
| <*trigger-type*> | String | El tipo de desencadenador como, por ejemplo, "Http" o "ApiConnection" | 
| <*trigger-inputs*> | Objeto JSON | Las entradas que definen el comportamiento del desencadenador | 
| <*time-unit*> | string | La unidad de tiempo que describe la frecuencia con que se activa el desencadenador: "Second", "Minute", "Hour", "Day", "Week", "Month" | 
| <*number-of-time-units*> | Integer | Un valor que especifica con qué frecuencia se activa el desencadenador según la frecuencia, que es el número de unidades de tiempo que debe esperar hasta que el desencadenador se activa de nuevo <p>Estos son los intervalos mínimo y máximo: <p>- Mes: 1-16 meses </br>- Día: 1-500 días </br>- Hora: 1-12 000 horas </br>- Minuto: 1-72 000 minutos </br>- Segundo: 1-9 999 999 segundos<p>Por ejemplo, si el intervalo es 6 y la frecuencia es "month", la periodicidad es cada 6 meses. | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*array-with-conditions*> | Matriz | Una matriz que contiene una o más [condiciones](#trigger-conditions) que determinan si ejecutar o no el flujo de trabajo. Disponible solo para desencadenadores. | 
| <*runtime-config-options*> | Objeto JSON | Puede cambiar el comportamiento del entorno en tiempo de ejecución del desencadenador estableciendo propiedades `runtimeConfiguration`. Para más información, consulte [Opciones de configuración del entorno en tiempo de ejecución](#runtime-config-options). | 
| <*splitOn-expression*> | string | Para los desencadenadores que devuelven una matriz, puede especificar una expresión que [divide o *desagrupa*](#split-on-debatch) los elementos de matriz en varias instancias de flujo de trabajo para su procesamiento. | 
| <*operation-option*> | string | Puede cambiar el comportamiento predeterminado estableciendo la propiedad `operationOptions`. Para más información, consulte [Opciones de operación](#operation-options). | 
|||| 

## <a name="trigger-types-list"></a>Lista de tipos de desencadenador

Cada tipo de desencadenador tiene una interfaz distinta y entradas que definen el comportamiento del desencadenador. 

### <a name="built-in-triggers"></a>Desencadenadores integrados

| Tipo de desencadenador | DESCRIPCIÓN | 
|--------------|-------------| 
| [**HTTP**](#http-trigger) | Comprueba o *sondea* cualquier punto de conexión. El punto de conexión debe ajustarse a un contrato de desencadenamiento específico, ya sea mediante un patrón asincrónico "202" o devolviendo una matriz. | 
| [**HTTPWebhook**](#http-webhook-trigger) | Crea un punto de conexión invocable para la aplicación lógica pero llama a la dirección URL especificada para registrar o anular el registro. |
| [**Recurrence**](#recurrence-trigger) | Se desencadena en función de una programación definida. Puede establecer una fecha y hora futuras para activar este desencadenador. Según la frecuencia, también puede especificar las horas y días para ejecutar el flujo de trabajo. | 
| [**Request**](#request-trigger)  | Crea un punto de conexión invocable para la aplicación lógica, también conocido como desencadenador "manual". Por ejemplo, consulte [Llamada, desencadenamiento o anidamiento de flujos de trabajo con puntos de conexión HTTP en aplicaciones lógicas](../logic-apps/logic-apps-http-endpoint.md). | 
||| 

### <a name="managed-api-triggers"></a>Desencadenadores de API administrados

| Tipo de desencadenador | DESCRIPCIÓN | 
|--------------|-------------| 
| [**ApiConnection**](#apiconnection-trigger) | Comprueba o *sondea* un punto de conexión mediante [API administradas por Microsoft](../connectors/apis-list.md). | 
| [**ApiConnectionWebhook**](#apiconnectionwebhook-trigger) | Crea un punto de conexión invocable para la aplicación lógica llamando a [API administradas por Microsoft](../connectors/apis-list.md) para suscribirse o cancelar la suscripción. | 
||| 

## <a name="triggers---detailed-reference"></a>Desencadenadores: referencia detallada

<a name="apiconnection-trigger"></a>

### <a name="apiconnection-trigger"></a>Desencadenador APIConnection  

Este desencadenador comprueba o *sondea* un punto de conexión mediante [API administradas por Microsoft](../connectors/apis-list.md) por lo que los parámetros de este desencadenador pueden ser diferentes según el punto de conexión. Muchas de las secciones de esta definición de desencadenador son opcionales. El comportamiento del desencadenador depende de si se incluyen las secciones o no.

```json
"<APIConnection_trigger_name>": {
   "type": "ApiConnection",
   "inputs": {
      "host": {
         "connection": {
            "name": "@parameters('$connections')['<connection-name>']['connectionId']"
         }
      },
      "method": "<method-type>",
      "path": "/<api-operation>",
      "retryPolicy": { "<retry-behavior>" },
      "queries": { "<query-parameters>" }
   },
   "recurrence": { 
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-runs-queue>
      }
   },
   "splitOn": "<splitOn-expression>",
   "operationOptions": "<operation-option>"
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*APIConnection_trigger_name*> | string | El nombre del desencadenador | 
| <*connection-name*> | String | El nombre de la conexión a la API administrada que utiliza el flujo de trabajo | 
| <*method-type*> | String | El método HTTP para comunicarse con la API administrada: "GET", "PUT", "POST", "PATCH", "DELETE" | 
| <*api-operation*> | String | La operación de API a la que llamar | 
| <*time-unit*> | string | La unidad de tiempo que describe la frecuencia con que se activa el desencadenador: "Second", "Minute", "Hour", "Day", "Week", "Month" | 
| <*number-of-time-units*> | Integer | Un valor que especifica con qué frecuencia se activa el desencadenador según la frecuencia, que es el número de unidades de tiempo que debe esperar hasta que el desencadenador se activa de nuevo <p>Estos son los intervalos mínimo y máximo: <p>- Mes: 1-16 meses </br>- Día: 1-500 días </br>- Hora: 1-12 000 horas </br>- Minuto: 1-72 000 minutos </br>- Segundo: 1-9 999 999 segundos<p>Por ejemplo, si el intervalo es 6 y la frecuencia es "month", la periodicidad es cada 6 meses. | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*retry-behavior*> | Objeto JSON | Personaliza el comportamiento de reintento para errores intermitentes, que tienen el código de estado 408, 429 y 5XX, y todas las excepciones de conectividad. Para más información, consulte [Directivas de reintentos](../logic-apps/logic-apps-exception-handling.md#retry-policies). | 
| <*query-parameters*> | Objeto JSON | Cualquier parámetro de consulta que desee incluir con la llamada API. Por ejemplo, el objeto `"queries": { "api-version": "2018-01-01" }` agrega `?api-version=2018-01-01` a la llamada. | 
| <*max-runs*> | Integer | De forma predeterminada, las instancias de flujo de trabajo ejecutan al mismo tiempo, o en paralelo, hasta el [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar este límite con un nuevo valor <*count*>, consulte [Cambio en la simultaneidad de desencadenadores](#change-trigger-concurrency). | 
| <*max-runs-queue*> | Integer | Cuando el flujo de trabajo ya se está ejecutando el número máximo de instancias, que se puede cambiar según la `runtimeConfiguration.concurrency.runs` propiedad, cualquier nuevas ejecuciones se ponen en la cola la [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar el límite predeterminado, consulte [Cambio del límite de ejecuciones en espera](#change-waiting-runs). | 
| <*splitOn-expression*> | String | Para los desencadenadores que devuelven matrices, esta expresión hace referencia a la matriz que se usará para que pueda crear y ejecutar una instancia de flujo de trabajo para cada elemento de la matriz, en lugar de usar un bucle "foreach". <p>Por ejemplo, esta expresión representa un elemento de la matriz devuelto en el contenido del cuerpo del desencadenador: `@triggerbody()?['value']` |
| <*operation-option*> | String | Puede cambiar el comportamiento predeterminado estableciendo la propiedad `operationOptions`. Para más información, consulte [Opciones de operación](#operation-options). |
||||

*Outputs*
 
| Elemento | Type | DESCRIPCIÓN |
|---------|------|-------------|
| encabezados | Objeto JSON | Encabezados de la respuesta |
| cuerpo | Objeto JSON | Cuerpo de la respuesta |
| Código de estado | Integer | El código de estado de la respuesta |
|||| 

*Ejemplo*

Esta definición del desencadenador busca correo electrónico a diario en la bandeja de entrada de una cuenta de Office 365 Outlook: 

```json
"When_a_new_email_arrives": {
   "type": "ApiConnection",
   "inputs": {
      "host": {
         "connection": {
            "name": "@parameters('$connections')['office365']['connectionId']"
         }
      },
      "method": "get",
      "path": "/Mail/OnNewEmail",
      "queries": {
          "fetchOnlyWithAttachment": false,
          "folderPath": "Inbox",
          "importance": "Any",
          "includeAttachments": false
      }
   },
   "recurrence": {
      "frequency": "Day",
      "interval": 1
   }
}
```

<a name="apiconnectionwebhook-trigger"></a>

### <a name="apiconnectionwebhook-trigger"></a>Desencadenador ApiConnectionWebhook

Este desencadenador envía una solicitud de suscripción a un punto de conexión mediante una [API administrada por Microsoft](../connectors/apis-list.md), proporciona una *dirección URL de devolución de llamada* donde el punto de conexión puede enviar una respuesta y espera a que este responda. Para más información, consulte [Suscripciones del punto de conexión](#subscribe-unsubscribe).

```json
"<ApiConnectionWebhook_trigger_name>": {
   "type": "ApiConnectionWebhook",
   "inputs": {
      "body": {
          "NotificationUrl": "@{listCallbackUrl()}"
      },
      "host": {
         "connection": {
            "name": "@parameters('$connections')['<connection-name>']['connectionId']"
         }
      },
      "retryPolicy": { "<retry-behavior>" },
      "queries": "<query-parameters>"
   },
   "runTimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-run-queue>
      }
   },
   "splitOn": "<splitOn-expression>",
   "operationOptions": "<operation-option>"
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*connection-name*> | String | El nombre de la conexión a la API administrada que utiliza el flujo de trabajo | 
| <*body-content*> | Objeto JSON | Cualquier contenido de mensaje para enviar como carga a la API administrada | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*retry-behavior*> | Objeto JSON | Personaliza el comportamiento de reintento para errores intermitentes, que tienen el código de estado 408, 429 y 5XX, y todas las excepciones de conectividad. Para más información, consulte [Directivas de reintentos](../logic-apps/logic-apps-exception-handling.md#retry-policies). | 
| <*query-parameters*> | Objeto JSON | Cualquier parámetro de consulta que desee incluir con la llamada API <p>Por ejemplo, el objeto `"queries": { "api-version": "2018-01-01" }` agrega `?api-version=2018-01-01` a la llamada. | 
| <*max-runs*> | Integer | De forma predeterminada, las instancias de flujo de trabajo ejecutan al mismo tiempo, o en paralelo, hasta el [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar este límite con un nuevo valor <*count*>, consulte [Cambio en la simultaneidad de desencadenadores](#change-trigger-concurrency). | 
| <*max-runs-queue*> | Integer | Cuando el flujo de trabajo ya se está ejecutando el número máximo de instancias, que se puede cambiar según la `runtimeConfiguration.concurrency.runs` propiedad, cualquier nuevas ejecuciones se ponen en la cola la [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar el límite predeterminado, consulte [Cambio del límite de ejecuciones en espera](#change-waiting-runs). | 
| <*splitOn-expression*> | string | Para los desencadenadores que devuelven matrices, esta expresión hace referencia a la matriz que se usará para que pueda crear y ejecutar una instancia de flujo de trabajo para cada elemento de la matriz, en lugar de usar un bucle "foreach". <p>Por ejemplo, esta expresión representa un elemento de la matriz devuelto en el contenido del cuerpo del desencadenador: `@triggerbody()?['value']` |
| <*operation-option*> | string | Puede cambiar el comportamiento predeterminado estableciendo la propiedad `operationOptions`. Para más información, consulte [Opciones de operación](#operation-options). | 
|||| 

*Ejemplo*

Esta definición del desencadenador se suscribe a la API de Office 365 Outlook, proporciona una dirección URL de devolución de llamada al punto de conexión de API y espera a que el punto de conexión responda cuando llega un nuevo correo electrónico.

```json
"When_a_new_email_arrives_(webhook)": {
   "type": "ApiConnectionWebhook",
   "inputs": {
      "body": {
         "NotificationUrl": "@{listCallbackUrl()}" 
      },
      "host": {
         "connection": {
            "name": "@parameters('$connections')['office365']['connectionId']"
         }
      },
      "path": "/MailSubscription/$subscriptions",
      "queries": {
          "folderPath": "Inbox",
          "hasAttachment": "Any",
          "importance": "Any"
      }
   },
   "splitOn": "@triggerBody()?['value']"
}
```

<a name="http-trigger"></a>

### <a name="http-trigger"></a>Desencadenador HTTP

Este desencadenador comprueba o sondea el punto de conexión especificado según la programación de periodicidad especificada. La respuesta del punto de conexión determina si el flujo de trabajo se ejecuta.

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "<method-type>",
      "uri": "<endpoint-URL>",
      "headers": { "<header-content>" },
      "body": "<body-content>",
      "authentication": { "<authentication-method>" },
      "retryPolicy": { "<retry-behavior>" },
      "queries": "<query-parameters>"
   },
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-runs-queue>
      }
   },
   "operationOptions": "<operation-option>"
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*method-type*> | String | El método HTTP que se usará para sondear el punto de conexión especificado: "GET", "PUT", "POST", "PATCH", "DELETE" | 
| <*endpoint-URL*> | String | La dirección URL HTTP o HTTPS que va a sondear el punto de conexión <p>Tamaño máximo de la cadena: 2 KB | 
| <*time-unit*> | string | La unidad de tiempo que describe la frecuencia con que se activa el desencadenador: "Second", "Minute", "Hour", "Day", "Week", "Month" | 
| <*number-of-time-units*> | Integer | Un valor que especifica con qué frecuencia se activa el desencadenador según la frecuencia, que es el número de unidades de tiempo que debe esperar hasta que el desencadenador se activa de nuevo <p>Estos son los intervalos mínimo y máximo: <p>- Mes: 1-16 meses </br>- Día: 1-500 días </br>- Hora: 1-12 000 horas </br>- Minuto: 1-72 000 minutos </br>- Segundo: 1-9 999 999 segundos<p>Por ejemplo, si el intervalo es 6 y la frecuencia es "month", la periodicidad es cada 6 meses. | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*header-content*> | Objeto JSON | Los encabezados que se envían con la solicitud <p>Por ejemplo, para establecer el idioma y el tipo de una solicitud: <p>`"headers": { "Accept-Language": "en-us", "Content-Type": "application/json" }` |
| <*body-content*> | string | El contenido del mensaje que se va a enviar como carga con la solicitud | 
| <*authentication-method*> | Objeto JSON | El método que usa la solicitud para la autenticación. Para más información, consulte [Autenticación saliente de Scheduler](../scheduler/scheduler-outbound-authentication.md). Más allá de Scheduler, se admite la propiedad `authority`. Cuando no se especifica, el valor predeterminado es `https://login.windows.net`, pero puede usar otro valor, como`https://login.windows\-ppe.net`. |
| <*retry-behavior*> | Objeto JSON | Personaliza el comportamiento de reintento para errores intermitentes, que tienen el código de estado 408, 429 y 5XX, y todas las excepciones de conectividad. Para más información, consulte [Directivas de reintentos](../logic-apps/logic-apps-exception-handling.md#retry-policies). |  
 <*query-parameters*> | Objeto JSON | Cualquier parámetro de consulta que desee incluir con la solicitud <p>Por ejemplo, el objeto `"queries": { "api-version": "2018-01-01" }` agrega `?api-version=2018-01-01` a la solicitud. | 
| <*max-runs*> | Integer | De forma predeterminada, las instancias de flujo de trabajo ejecutan al mismo tiempo, o en paralelo, hasta el [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar este límite con un nuevo valor <*count*>, consulte [Cambio en la simultaneidad de desencadenadores](#change-trigger-concurrency). | 
| <*max-runs-queue*> | Integer | Cuando el flujo de trabajo ya se está ejecutando el número máximo de instancias, que se puede cambiar según la `runtimeConfiguration.concurrency.runs` propiedad, cualquier nuevas ejecuciones se ponen en la cola la [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar el límite predeterminado, consulte [Cambio del límite de ejecuciones en espera](#change-waiting-runs). | 
| <*operation-option*> | String | Puede cambiar el comportamiento predeterminado estableciendo la propiedad `operationOptions`. Para más información, consulte [Opciones de operación](#operation-options). | 
|||| 

*Outputs*

| Elemento | Type | DESCRIPCIÓN |
|---------|------|-------------| 
| encabezados | Objeto JSON | Encabezados de la respuesta | 
| cuerpo | Objeto JSON | Cuerpo de la respuesta | 
| Código de estado | Integer | El código de estado de la respuesta | 
|||| 

*Requisitos de las solicitudes entrantes*

Para que funcione bien con la aplicación lógica, el punto de conexión debe cumplir con un patrón de desencadenador específico o un contrato y reconocer estas propiedades:  
  
| Respuesta | Obligatorio | DESCRIPCIÓN | 
|----------|----------|-------------| 
| Código de estado | Sí | El código de estado "200 OK" inicia una ejecución. Cualquier otro código de estado no inicia una ejecución. | 
| Encabezado Retry-after | No | El número de segundos hasta que la aplicación lógica sondea de nuevo el punto de conexión | 
| Encabezado Location | No | La dirección URL para llamar en el siguiente intervalo de sondeo. Si no se especifica, se usa la dirección URL original. | 
|||| 

*Comportamientos de ejemplo para solicitudes distintas*

| Código de estado | Reintentar después | Comportamiento | 
|-------------|-------------|----------|
| 200 | {none} | Ejecutar el flujo de trabajo y luego comprobar de nuevo si hay más datos después de la periodicidad definida. | 
| 200 | 10 segundos | Ejecutar el flujo de trabajo y luego comprobar de nuevo si hay más datos después de 10 segundos. |  
| 202 | 60 segundos | No desencadenar el flujo de trabajo. El próximo intento tiene lugar en un minuto, según cuál sea la periodicidad definida. Si la periodicidad definida es inferior a un minuto, el encabezado retry-after tiene prioridad. Si no, se usa la periodicidad definida. | 
| 400 | {none} | Solicitud incorrecta, no ejecutar el flujo de trabajo. Si no se define ningún `retryPolicy`, se utiliza la directiva predeterminada. Una vez alcanzado el número de reintentos, el desencadenador volverá a comprobar si hay datos después de la periodicidad definida. | 
| 500 | {none}| Error del servidor, no ejecutar el flujo de trabajo. Si no se define ningún `retryPolicy`, se utiliza la directiva predeterminada. Una vez alcanzado el número de reintentos, el desencadenador volverá a comprobar si hay datos después de la periodicidad definida. | 
|||| 

<a name="http-webhook-trigger"></a>

### <a name="httpwebhook-trigger"></a>Desencadenador HTTPWebhook  

Este desencadenador hace que la aplicación lógica sea invocable mediante la creación de un punto de conexión que puede registrar una suscripción llamando a la dirección URL especificada de este. Cuando se crea este desencadenador en el flujo de trabajo, una solicitud saliente realiza la llamada para registrar la suscripción. De este modo, el desencadenador puede iniciar la escucha de eventos. Cuando una operación hace que este desencadenador no sea válido, una solicitud saliente realiza automáticamente la llamada para cancelar la suscripción. Para más información, consulte [Suscripciones del punto de conexión](#subscribe-unsubscribe).

También puede especificar [límites asincrónicos](#asynchronous-limits) en un desencadenador **HTTPWebhook**.
El comportamiento del desencadenador depende de las secciones que se usen o se omitan. 

```json
"HTTP_Webhook": {
   "type": "HttpWebhook",
   "inputs": {
      "subscribe": {
         "method": "<method-type>",
         "uri": "<endpoint-subscribe-URL>",
         "headers": { "<header-content>" },
         "body": "<body-content>",
         "authentication": { "<authentication-method>" },
         "retryPolicy": { "<retry-behavior>" }
         },
      },
      "unsubscribe": {
         "method": "<method-type>",
         "url": "<endpoint-unsubscribe-URL>",
         "headers": { "<header-content>" },
         "body": "<body-content>",
         "authentication": { "<authentication-method>" }
      }
   },
   "runTimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-runs-queue>
      }
   },
   "operationOptions": "<operation-option>"
}
```

Algunos de los valores, como <*method-type*>, están disponibles para objetos `"subscribe"` y `"unsubscribe"`.

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*method-type*> | String | El método HTTP que se usará para la solicitud de suscripción: "GET", "PUT", "POST", "PATCH" o "DELETE" | 
| <*endpoint-subscribe-URL*> | String | Dirección URL del punto de conexión a donde enviar la solicitud de suscripción | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*method-type*> | String | El método HTTP que se usará para la solicitud de cancelación: "GET", "PUT", "POST", "PATCH" o "DELETE" | 
| <*endpoint-unsubscribe-URL*> | string | Dirección URL del punto de conexión a donde enviar la solicitud de cancelación | 
| <*body-content*> | string | Cualquier contenido de mensaje para enviar en la solicitud de suscripción o de cancelación | 
| <*authentication-method*> | Objeto JSON | El método que usa la solicitud para la autenticación. Para más información, consulte [Autenticación saliente de Scheduler](../scheduler/scheduler-outbound-authentication.md). |
| <*retry-behavior*> | Objeto JSON | Personaliza el comportamiento de reintento para errores intermitentes, que tienen el código de estado 408, 429 y 5XX, y todas las excepciones de conectividad. Para más información, consulte [Directivas de reintentos](../logic-apps/logic-apps-exception-handling.md#retry-policies). | 
| <*max-runs*> | Integer | De forma predeterminada, todas las instancias de flujo de trabajo ejecutan al mismo tiempo, o en paralelo, hasta el [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar este límite con un nuevo valor <*count*>, consulte [Cambio en la simultaneidad de desencadenadores](#change-trigger-concurrency). | 
| <*max-runs-queue*> | Integer | Cuando el flujo de trabajo ya se está ejecutando el número máximo de instancias, que se puede cambiar según la `runtimeConfiguration.concurrency.runs` propiedad, cualquier nuevas ejecuciones se ponen en la cola la [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar el límite predeterminado, consulte [Cambio del límite de ejecuciones en espera](#change-waiting-runs). | 
| <*operation-option*> | String | Puede cambiar el comportamiento predeterminado estableciendo la propiedad `operationOptions`. Para más información, consulte [Opciones de operación](#operation-options). | 
|||| 

*Outputs* 

| Elemento | Type | DESCRIPCIÓN |
|---------|------|-------------| 
| encabezados | Objeto JSON | Encabezados de la respuesta | 
| cuerpo | Objeto JSON | Cuerpo de la respuesta | 
| Código de estado | Integer | El código de estado de la respuesta | 
|||| 

*Ejemplo*

Este desencadenador crea una suscripción al punto de conexión especificado, proporciona una dirección URL única de devolución de llamada y espera a artículos sobre tecnología recientemente publicados.

```json
"HTTP_Webhook": {
   "type": "HttpWebhook",
   "inputs": {
      "subscribe": {
         "method": "POST",
         "uri": "https://pubsubhubbub.appspot.com/subscribe",
         "body": {
            "hub.callback": "@{listCallbackUrl()}",
            "hub.mode": "subscribe",
            "hub.topic": "https://pubsubhubbub.appspot.com/articleCategories/technology"
         },
      },
      "unsubscribe": {
         "method": "POST",
         "url": "https://pubsubhubbub.appspot.com/subscribe",
         "body": {
            "hub.callback": "@{workflow().endpoint}@{listCallbackUrl()}",
            "hub.mode": "unsubscribe",
            "hub.topic": "https://pubsubhubbub.appspot.com/articleCategories/technology"
         }
      }
   }
}
```

<a name="recurrence-trigger"></a>

### <a name="recurrence-trigger"></a>Desencadenador de periodicidad  

Este desencadenador se ejecuta según la programación de periodicidad especificada y proporciona una manera sencilla de ejecutar con regularidad un flujo de trabajo. 

```json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
      "startTime": "<start-date-time-with-format-YYYY-MM-DDThh:mm:ss>",
      "timeZone": "<time-zone>",
      "schedule": {
         // Applies only when frequency is Day or Week. Separate values with commas.
         "hours": [ <one-or-more-hour-marks> ], 
         // Applies only when frequency is Day or Week. Separate values with commas.
         "minutes": [ <one-or-more-minute-marks> ], 
         // Applies only when frequency is Week. Separate values with commas.
         "weekDays": [ "Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday" ] 
      }
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-runs-queue>
      }
   },
   "operationOptions": "<operation-option>"
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*time-unit*> | String | La unidad de tiempo que describe la frecuencia con que se activa el desencadenador: "Second", "Minute", "Hour", "Day", "Week", "Month" | 
| <*number-of-time-units*> | Integer | Un valor que especifica con qué frecuencia se activa el desencadenador según la frecuencia, que es el número de unidades de tiempo que debe esperar hasta que el desencadenador se activa de nuevo <p>Estos son los intervalos mínimo y máximo: <p>- Mes: 1-16 meses </br>- Día: 1-500 días </br>- Hora: 1-12 000 horas </br>- Minuto: 1-72 000 minutos </br>- Segundo: 1-9 999 999 segundos<p>Por ejemplo, si el intervalo es 6 y la frecuencia es "month", la periodicidad es cada 6 meses. | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*start-date-time-with-format-YYYY-MM-DDThh:mm:ss*> | String | La fecha y hora de inicio en este formato: <p>AAAA-MM-DDThh:mm:ss si especifica una zona horaria <p>-o bien- <p>AAAA-MM-DDThh:mm:ssZ si no especifica una zona horaria <p>Por ejemplo, si desea la fecha del 18 de septiembre de 2017 a las 2:00 p.m., especifique entonces "2017-09-18T14:00:00" y especifique una zona horaria como "Hora estándar del Pacífico", o bien especifique "2017-09-18T14:00:00Z" sin una zona horaria. <p>**Nota:** Esta hora de inicio debe seguir la [especificación de fecha y hora ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) en [formato de hora y fecha UTC](https://en.wikipedia.org/wiki/Coordinated_Universal_Time), pero sin una [diferencia horaria con UTC](https://en.wikipedia.org/wiki/UTC_offset). Si no se especifica una zona horaria, debe agregar la letra "Z" al final sin espacios. Esta "Z" se refiere al equivalente de [hora náutica](https://en.wikipedia.org/wiki/Nautical_time). <p>Para las programaciones simples, la hora de inicio es la primera aparición, mientras que para programaciones complejas, el desencadenador no se activa antes de la hora de inicio. Para más información sobre las fechas y horas de inicio, consulte [Introducción al desencadenador de periodicidad](../connectors/connectors-native-recurrence.md). | 
| <*time-zone*> | String | Solo se aplica cuando se especifica una hora de inicio porque este desencadenador no acepta [diferencia horaria con UTC](https://en.wikipedia.org/wiki/UTC_offset). Especifique la zona horaria que desea aplicar. | 
| <*one-or-more-hour-marks*> | Entero o matriz de enteros | Si especifica "Day" o "Semana" para `frequency`, puede especificar uno o varios enteros de 0 a 23, separados por comas, como las horas del día en las que desea ejecutar el flujo de trabajo. <p>Por ejemplo, si especifica "10", "12" y "14", obtendrá 10 a. m., 12 p. m. y 2 p. m. como las marcas de hora. | 
| <*one-or-more-minute-marks*> | Entero o matriz de enteros | Si especifica "Day" o "Semana" para `frequency`, puede especificar uno o varios enteros de 0 a 59, separados por comas, como los minutos de la hora en los que desea ejecutar el flujo de trabajo. <p>Por ejemplo, puede especificar "30" como la marca de minuto y, utilizando el ejemplo anterior para las horas del día, obtendrá 10:30 a. m., 12:30 p. m. y las 2:30 p. m. | 
| weekDays | Cadena o matriz de cadenas | Si especifica "Week" para `frequency`, puede especificar uno o varios días, separados por comas, cuando quiera ejecutar el flujo de trabajo: "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday" y "Sunday" | 
| <*max-runs*> | Integer | De forma predeterminada, todas las instancias de flujo de trabajo ejecutan al mismo tiempo, o en paralelo, hasta el [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar este límite con un nuevo valor <*count*>, consulte [Cambio en la simultaneidad de desencadenadores](#change-trigger-concurrency). | 
| <*max-runs-queue*> | Integer | Cuando el flujo de trabajo ya se está ejecutando el número máximo de instancias, que se puede cambiar según la `runtimeConfiguration.concurrency.runs` propiedad, cualquier nuevas ejecuciones se ponen en la cola la [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar el límite predeterminado, consulte [Cambio del límite de ejecuciones en espera](#change-waiting-runs). | 
| <*operation-option*> | String | Puede cambiar el comportamiento predeterminado estableciendo la propiedad `operationOptions`. Para más información, consulte [Opciones de operación](#operation-options). | 
|||| 

*Ejemplo 1*

La periodicidad básica con que se ejecuta diariamente el desencadenador:

```json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "Day",
      "interval": 1
   }
}
```

*Ejemplo 2*

Puede especificar una fecha y hora de inicio para activar el desencadenador. Este desencadenador de periodicidad se inicia en la fecha especificada y luego se activa a diario:

```json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "Day",
      "interval": 1,
      "startTime": "2017-09-18T00:00:00Z"
   }
}
```

*Ejemplo 3*

Este desencadenador de periodicidad se inicia el 9 de septiembre de 2017 a las 2:00 p.m. y se activa semanalmente cada lunes a las 10:30 a.m., 12:30 p.m. y 2:30 p.m., Hora estándar del Pacífico:

``` json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "Week",
      "interval": 1,
      "schedule": {
         "hours": [ 10, 12, 14 ],
         "minutes": [ 30 ],
         "weekDays": [ "Monday" ]
      },
      "startTime": "2017-09-07T14:00:00",
      "timeZone": "Pacific Standard Time"
   }
}
```

Para más información sobre ejemplos de este desencadenador, consulte [Creación y programación de tareas ejecutadas con regularidad con Azure Logic Apps](../connectors/connectors-native-recurrence.md).

<a name="request-trigger"></a>

### <a name="request-trigger"></a>Desencadenador de solicitud

Este desencadenador hace que se pueda llamar a la aplicación lógica mediante la creación de un punto de conexión que puede aceptar solicitudes entrantes. Para este desencadenador, proporcione un esquema JSON que describa y valide la carga o las entradas que el desencadenador recibe de la solicitud entrante. El esquema también facilita la referencia a propiedades del desencadenador desde acciones posteriores del flujo de trabajo. 

Para llamar a este desencadenador, debe usar la `listCallbackUrl`API que se describe en [API REST de Servicio de flujo de trabajo](https://docs.microsoft.com/rest/api/logic/workflows). Para información sobre cómo usar este desencadenador como punto de conexión HTTP, consulte [Llamada, desencadenamiento o anidamiento de flujos de trabajo con puntos de conexión HTTP en aplicaciones lógicas](../logic-apps/logic-apps-http-endpoint.md).

```json
"manual": {
   "type": "Request",
   "kind": "Http",
   "inputs": {
      "method": "<method-type>",
      "relativePath": "<relative-path-for-accepted-parameter>",
      "schema": {
         "type": "object",
         "properties": { 
            "<property-name>": {
               "type": "<property-type>"
            }
         },
         "required": [ "<required-properties>" ]
      }
   },
   "runTimeConfiguration": {
      "concurrency": {
         "runs": <max-runs>,
         "maximumWaitingRuns": <max-run-queue>
      },
   },
   "operationOptions": "<operation-option>"
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*property-name*> | String | El nombre de una propiedad en el esquema JSON que describe la carga | 
| <*property-type*> | String | El tipo de propiedad | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*method-type*> | string | El método que las solicitudes entrantes deben usar para llamar a la aplicación lógica: "GET", "PUT", "POST", "PATCH", "DELETE" |
| <*relative-path-for-accepted-parameter*> | string | La ruta de acceso relativa del parámetro que la dirección URL del punto de conexión puede aceptar | 
| <*required-properties*> | Matriz | Una o más propiedades que requieren valores | 
| <*max-runs*> | Integer | De forma predeterminada, todas las instancias de flujo de trabajo ejecutan al mismo tiempo, o en paralelo, hasta el [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar este límite con un nuevo valor <*count*>, consulte [Cambio en la simultaneidad de desencadenadores](#change-trigger-concurrency). | 
| <*max-runs-queue*> | Integer | Cuando el flujo de trabajo ya se está ejecutando el número máximo de instancias, que se puede cambiar según la `runtimeConfiguration.concurrency.runs` propiedad, cualquier nuevas ejecuciones se ponen en la cola la [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar el límite predeterminado, consulte [Cambio del límite de ejecuciones en espera](#change-waiting-runs). | 
| <*operation-option*> | String | Puede cambiar el comportamiento predeterminado estableciendo la propiedad `operationOptions`. Para más información, consulte [Opciones de operación](#operation-options). | 
|||| 

*Ejemplo*

Este desencadenador especifica que una solicitud entrante debe usar el método HTTP POST para llamar al desencadenador e incluye un esquema que valida la entrada de la solicitud entrante: 

```json
"manual": {
   "type": "Request",
   "kind": "Http",
   "inputs": {
      "method": "POST",
      "schema": {
         "type": "object",
         "properties": {
            "customerName": {
               "type": "String"
            },
            "customerAddress": { 
               "type": "Object",
               "properties": {
                  "streetAddress": {
                     "type": "string"
                  },
                  "city": {
                     "type": "string"
                  }
               }
            }
         }
      }
   }
}
```

<a name="trigger-conditions"></a>

## <a name="trigger-conditions"></a>Condiciones del desencadenador

Para cualquier desencadenador, y solo desencadenadores, puede incluir una matriz con una o varias expresiones de condiciones que determinan si el flujo de trabajo debe ejecutarse o no. Para agregar el `conditions` propiedad a un desencadenador del flujo de trabajo, abra la aplicación lógica en el editor de la vista de código.

Por ejemplo, puede especificar que un desencadenador se active solo cuando un sitio web devuelve un error de servidor interno haciendo referencia al código de estado del desencadenador en la propiedad `conditions`:

```json
"Recurrence": {
   "type": "Recurrence",
   "recurrence": {
      "frequency": "Hour",
      "interval": 1
   },
   "conditions": [ {
      "expression": "@equals(triggers().code, 'InternalServerError')"
   } ]
}
```

De forma predeterminada, un desencadenador solo se activa después de obtener una respuesta "200 OK". Cuando una expresión hace referencia a un código de estado del desencadenador, se reemplaza el comportamiento predeterminado de este. Por tanto, si desea que el desencadenador se active con más de un código de estado, como el "200" y el "201", debe incluir esta expresión como condición: 

`@or(equals(triggers().code, 200),equals(triggers().code, 201))` 

<a name="split-on-debatch"></a>

## <a name="trigger-multiple-runs"></a>Desencadenar varias ejecuciones

Si el desencadenador devuelve una matriz para que la procese la aplicación lógica, un bucle "para cada una" podría tardar demasiado tiempo en procesar cada elemento de la matriz. En lugar de eso, puede usar la propiedad **SplitOn** en el desencadenador para *desagrupar* la matriz. Desagrupación divide los elementos de matriz e inicia una nueva instancia de flujo de trabajo que se ejecuta para cada elemento de matriz. Este enfoque resulta útil, por ejemplo, si desea sondear un punto de conexión que puede devolver varios elementos nuevos entre intervalos de sondeo.
Para conocer el número máximo de elementos de matriz que **SplitOn** puede procesar en una única ejecución de aplicación lógica, consulte [Límites y configuración](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). 

> [!NOTE]
> No puede utilizar **SplitOn** con un patrón de respuesta sincrónica. Cualquier flujo de trabajo que use **SplitOn** e incluya una acción de respuesta se ejecuta de forma asincrónica y envía de inmediato una respuesta `202 ACCEPTED`.

Si el archivo Swagger del desencadenador describe una carga que es una matriz, la propiedad **SplitOn** se agrega automáticamente a su desencadenador. En caso contrario, agregue esta propiedad en la carga de respuesta que tiene la matriz que desea desagrupar. 

*Ejemplo*

Suponga que tiene una API que devuelve esta respuesta: 
  
```json
{
   "Status": "Succeeded",
   "Rows": [ 
      { 
         "id": 938109380,
         "name": "customer-name-one"
      },
      {
         "id": 938109381,
         "name": "customer-name-two"
      }
   ]
}
```
 
La aplicación lógica solo necesita el contenido de la matriz en `Rows`, por lo que puede crear un desencadenador como en este ejemplo:

``` json
"HTTP_Debatch": {
   "type": "Http",
    "inputs": {
        "uri": "https://mydomain.com/myAPI",
        "method": "GET"
    },
   "recurrence": {
      "frequency": "Second",
      "interval": 1
    },
    "splitOn": "@triggerBody()?.Rows"
}
```

> [!NOTE]
> Si usa el comando `SplitOn`, no puede obtener las propiedades que están fuera de la matriz. Así pues, en este ejemplo, no puede obtener la propiedad `status` en la respuesta que se devuelve a través de la API.
> 
> Para evitar un error si la propiedad `Rows` no existe, en este ejemplo se utiliza el operador `?`.

La definición de flujo de trabajo ahora puede usar `@triggerBody().name` para obtener los valores `name` `"customer-name-one"` de la primera ejecución y `"customer-name-two"` de la segunda ejecución. Así, las salidas del desencadenador se parecen a estos ejemplos:

```json
{
   "body": {
      "id": 938109380,
      "name": "customer-name-one"
   }
}
```

```json
{
   "body": {
      "id": 938109381,
      "name": "customer-name-two"
   }
}
```

<a name="actions-overview"></a>

## <a name="actions-overview"></a>Información general sobre acciones

Azure Logic Apps proporciona varios tipos de acción, cada uno con diferentes entradas que definen un comportamiento único de esta. 

Las acciones tienen estos elementos de alto nivel, aunque algunos son opcionales:

```json
"<action-name>": {
   "type": "<action-type>",
   "inputs": { 
      "<input-name>": { "<input-value>" },
      "retryPolicy": "<retry-behavior>" 
   },
   "runAfter": { "<previous-trigger-or-action-status>" },
   "runtimeConfiguration": { "<runtime-config-options>" },
   "operationOptions": "<operation-option>"
},
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------|
| <*action-name*> | string | El nombre de la acción | 
| <*action-type*> | string | El tipo de acción, por ejemplo, "Http" o "ApiConnection"| 
| <*input-name*> | String | El nombre de una entrada que define el comportamiento de la acción | 
| <*input-value*> | Varios | El valor de entrada, que puede ser una cadena, un número entero, un objeto JSON, etc. | 
| <*previous-trigger-or-action-status*> | Objeto JSON | El nombre y el estado resultante del desencadenador o acción que se debe ejecutar inmediatamente antes de que se pueda ejecutar esta acción actual | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------|
| <*retry-behavior*> | Objeto JSON | Personaliza el comportamiento de reintento para errores intermitentes, que tienen el código de estado 408, 429 y 5XX, y todas las excepciones de conectividad. Para obtener más información, consulte Directivas de reintentos. | 
| <*runtime-config-options*> | Objeto JSON | Para algunas acciones, puede cambiar el comportamiento de la acción en el tiempo de ejecución estableciendo propiedades `runtimeConfiguration`. Para más información, consulte [Opciones de configuración del entorno en tiempo de ejecución](#runtime-config-options). | 
| <*operation-option*> | String | Para algunas acciones, puede cambiar el comportamiento predeterminado estableciendo la propiedad `operationOptions`. Para más información, consulte [Opciones de operación](#operation-options). | 
|||| 

## <a name="action-types-list"></a>Lista de tipos de acción

Estos son algunos tipos de acción de uso frecuente: 

* [Tipos de acción integrada](#built-in-actions) como estos ejemplos y muchos otros: 

  * [**HTTP** ](#http-action) para llamar a puntos de conexión a través de HTTP o HTTPS

  * [**Respuesta**](#response-action) para responder a solicitudes

  * [**Ejecutar el código de JavaScript** ](#run-javascript-code) fragmentos de código para la ejecución de JavaScript

  * [**Function**](#function-action) para llamar a Azure Functions

  * Las acciones de operación con datos como [**Combinar**](#join-action), [**Redactar**](#compose-action), [**Tabla**](#table-action), [**Seleccionar**](#select-action) y otras que permiten crear o transformar datos a partir de varias entradas

  * [**Workflow**](#workflow-action) para llamar a otro flujo de trabajo de aplicación lógica

* [Tipos de acción de API administradas](#managed-api-actions) como [**ApiConnection**](#apiconnection-action) y [**ApiConnectionWebHook**](#apiconnectionwebhook-action) que llaman a varios conectores y API administrados por Microsoft como, por ejemplo, Azure Service Bus, Office 365 Outlook, Power BI, Azure Blob Storage, OneDrive, GitHub, etc.

* [Tipos de acción de control de flujo de trabajo](#control-workflow-actions) como, por ejemplo, [**If**](#if-action), [**Foreach**](#foreach-action), [**Switch**](#switch-action), [**Scope**](#scope-action) y [**Until**](#until-action), que contienen otras acciones y le ayudan a organizar la ejecución del flujo de trabajo

<a name="built-in-actions"></a>

### <a name="built-in-actions"></a>Acciones integradas

| Tipo de acción | DESCRIPCIÓN | 
|-------------|-------------| 
| [**Redactar**](#compose-action) | Crea una única salida a partir de las entradas y puede tener varios tipos. | 
| [**Ejecutar el código de JavaScript**](#run-javascript-code) | Ejecutar fragmentos de código de JavaScript que se ajusten a criterios específicos. Para los requisitos de código y obtener más información, consulte [Add y fragmentos de código de ejecución con el código en línea](../logic-apps/logic-apps-add-run-inline-code.md). |
| [**Function**](#function-action) | Llama a una función de Azure. | 
| [**HTTP**](#http-action) | Llama a un punto de conexión HTTP. | 
| [**Combinar**](#join-action) | Crea una cadena con todos los elementos de una matriz y los separa con el carácter delimitador especificado. | 
| [**Análisis del archivo JSON**](#parse-json-action) | Crea tokens fáciles de usar a partir de propiedades del contenido JSON. Posteriormente, puede hacer referencia a esas propiedades mediante la inclusión de los tokens en la aplicación lógica. | 
| [**Consulta**](#query-action) | Crea una matriz a partir de elementos de otra matriz basándose en una condición o un filtro. | 
| [**Respuesta**](#response-action) | Crea una respuesta a una solicitud o una llamada entrante. | 
| [**Seleccionar**](#select-action) | Crea una matriz con objetos JSON mediante la transformación de los elementos de otra matriz según la asignación especificada. | 
| [**Tabla**](#table-action) | Crea una tabla CSV o HTML a partir de una matriz. | 
| [**Terminate**](#terminate-action) | Detiene un flujo de trabajo que se está ejecutando activamente. | 
| [**Esperar**](#wait-action) | Pone en pausa el flujo de trabajo durante un tiempo especificado o hasta la fecha y hora especificadas. | 
| [**Workflow**](#workflow-action) | Anida un flujo de trabajo dentro de otro flujo de trabajo. | 
||| 

<a name="managed-api-actions"></a>

### <a name="managed-api-actions"></a>Acciones de API administradas

| Tipo de acción | DESCRIPCIÓN | 
|-------------|-------------|  
| [**ApiConnection**](#apiconnection-action) | Comprueba un punto de conexión HTTP mediante una [API administrada por Microsoft](../connectors/apis-list.md). | 
| [**ApiConnectionWebhook**](#apiconnectionwebhook-action) | Funciona igual que HTTP Webhook, pero usa una [API administrada por Microsoft](../connectors/apis-list.md). | 
||| 

<a name="control-workflow-actions"></a>

### <a name="control-workflow-actions"></a>Acciones de control del flujo de trabajo

Estas acciones le ayudan a controlar la ejecución del flujo de trabajo e incluyen, a su vez, otras acciones. Desde fuera de una acción de control de flujo de trabajo puede hacer referencia directamente a las acciones de esa acción de control del flujo de trabajo. Por ejemplo, si tiene una acción `Http` dentro de un ámbito, puede hacer referencia a la expresión `@body('Http')` desde cualquier lugar del flujo de trabajo. No obstante, las acciones que existen en una acción de control del flujo de trabajo solo se pueden "ejecutar después" de otras acciones que están en la misma estructura de control del flujo de trabajo.

| Tipo de acción | DESCRIPCIÓN | 
|-------------|-------------| 
| [**ForEach**](#foreach-action) | Ejecuta las mismas acciones en un bucle para todos los elementos de una matriz. | 
| [**If**](#if-action) | Ejecuta acciones en función de si la condición especificada es true o false. | 
| [**Scope**](#scope-action) | Ejecuta acciones según el estado del grupo a partir de un conjunto de acciones. | 
| [**Switch**](#switch-action) | Ejecuta acciones que se organizan en casos cuando los valores de las expresiones, objetos o tokens coinciden con los valores especificados por cada caso. | 
| [**Until**](#until-action) | Ejecuta acciones en un bucle hasta que la condición especificada es true. | 
|||  

## <a name="actions---detailed-reference"></a>Acciones: referencia detallada

<a name="apiconnection-action"></a>

### <a name="apiconnection-action"></a>Acción APIConnection

Esta acción envía una solicitud HTTP a una [API administrada por Microsoft](../connectors/apis-list.md) y requiere información sobre la API y los parámetros más una referencia a una conexión válida. 

``` json
"<action-name>": {
   "type": "ApiConnection",
   "inputs": {
      "host": {
         "connection": {
            "name": "@parameters('$connections')['<api-name>']['connectionId']"
         },
         "<other-action-specific-input-properties>"        
      },
      "method": "<method-type>",
      "path": "/<api-operation>",
      "retryPolicy": "<retry-behavior>",
      "queries": { "<query-parameters>" },
      "<other-action-specific-properties>"
    },
    "runAfter": {}
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*action-name*> | String | El nombre de la acción proporcionada por el conector | 
| <*api-name*> | String | El nombre de la API administrada por Microsoft que se usa para la conexión | 
| <*method-type*> | string | El método HTTP para llamar a la API: "GET", "PUT", "POST", "PATCH" o "DELETE" | 
| <*api-operation*> | string | La operación de API a la que llamar | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*other-action-specific-input-properties*> | Objeto JSON | Cualquier otra propiedad de entrada que sea aplicable a esta acción específica | 
| <*retry-behavior*> | Objeto JSON | Personaliza el comportamiento de reintento para errores intermitentes, que tienen el código de estado 408, 429 y 5XX, y todas las excepciones de conectividad. Para más información, consulte [Directivas de reintentos](../logic-apps/logic-apps-exception-handling.md#retry-policies). | 
| <*query-parameters*> | Objeto JSON | Cualquier parámetro de consulta que desee incluir con la llamada API. <p>Por ejemplo, el objeto `"queries": { "api-version": "2018-01-01" }` agrega `?api-version=2018-01-01` a la llamada. | 
| <*other-action-specific-properties*> | Objeto JSON | Cualquier otra propiedad que sea aplicable a esta acción específica | 
|||| 

*Ejemplo*

Esta definición describe la acción **Enviar un correo electrónico** para el conector de Office 365 Outlook, que es una API administrada por Microsoft: 

```json
"Send_an_email": {
   "type": "ApiConnection",
   "inputs": {
      "body": {
         "Body": "Thank you for your membership!",
         "Subject": "Hello and welcome!",
         "To": "Sophie.Owen@contoso.com"
      },
      "host": {
         "connection": {
            "name": "@parameters('$connections')['office365']['connectionId']"
         }
      },
      "method": "POST",
      "path": "/Mail"
    },
    "runAfter": {}
}
```

<a name="apiconnection-webhook-action"></a>

### <a name="apiconnectionwebhook-action"></a>Acción de APIConnectionWebhook

Esta acción envía una solicitud de suscripción a través de HTTP a un punto de conexión mediante una [API administrada por Microsoft](../connectors/apis-list.md), proporciona una *dirección URL de devolución de llamada* donde el punto de conexión puede enviar una respuesta y espera a que este responda. Para más información, consulte [Suscripciones del punto de conexión](#subscribe-unsubscribe).

```json
"<action-name>": {
   "type": "ApiConnectionWebhook",
   "inputs": {
      "subscribe": {
         "method": "<method-type>",
         "uri": "<api-subscribe-URL>",
         "headers": { "<header-content>" },
         "body": "<body-content>",
         "authentication": { "<authentication-method>" },
         "retryPolicy": "<retry-behavior>",
         "queries": { "<query-parameters>" },
         "<other-action-specific-input-properties>"
      },
      "unsubscribe": {
         "method": "<method-type>",
         "uri": "<api-unsubscribe-URL>",
         "headers": { "<header-content>" },
         "body": "<body-content>",
         "authentication": { "<authentication-method>" },
         "<other-action-specific-properties>"
      },
   },
   "runAfter": {}
}
```

Algunos de los valores, como <*method-type*>, están disponibles para objetos `"subscribe"` y `"unsubscribe"`.

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*action-name*> | String | El nombre de la acción proporcionada por el conector | 
| <*method-type*> | String | El método HTTP que se usará para suscribirse o cancelar la suscripción desde un punto de conexión: "GET", "PUT", "POST", "PATCH" o "DELETE" | 
| <*api-subscribe-URL*> | String | El identificador URI que se utiliza para suscribirse a la API | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*api-unsubscribe-URL*> | string | El identificador URI que se utiliza para cancelar la suscripción desde la API | 
| <*header-content*> | Objeto JSON | Todos los encabezados que se vayan a enviar en la solicitud <p>Por ejemplo, para establecer el idioma y el tipo en una solicitud: <p>`"headers": { "Accept-Language": "en-us", "Content-Type": "application/json" }` |
| <*body-content*> | Objeto JSON | Cualquier contenido de mensaje que se vaya a enviar en la solicitud | 
| <*authentication-method*> | Objeto JSON | El método que usa la solicitud para la autenticación. Para más información, consulte [Autenticación saliente de Scheduler](../scheduler/scheduler-outbound-authentication.md). |
| <*retry-behavior*> | Objeto JSON | Personaliza el comportamiento de reintento para errores intermitentes, que tienen el código de estado 408, 429 y 5XX, y todas las excepciones de conectividad. Para más información, consulte [Directivas de reintentos](../logic-apps/logic-apps-exception-handling.md#retry-policies). | 
| <*query-parameters*> | Objeto JSON | Cualquier parámetro de consulta que desee incluir con la llamada API <p>Por ejemplo, el objeto `"queries": { "api-version": "2018-01-01" }` agrega `?api-version=2018-01-01` a la llamada. | 
| <*other-action-specific-input-properties*> | Objeto JSON | Cualquier otra propiedad de entrada que sea aplicable a esta acción específica | 
| <*other-action-specific-properties*> | Objeto JSON | Cualquier otra propiedad que sea aplicable a esta acción específica | 
|||| 

También puede especificar límites sobre una acción de **ApiConnectionWebhook** de la misma manera que los [límites asincrónicos de HTTP](#asynchronous-limits).

<a name="compose-action"></a>

### <a name="compose-action"></a>Acción Compose

Esta acción crea una única salida a partir de varias entradas, incluidas las expresiones. La salida y las entradas pueden tener cualquier tipo que Azure Logic Apps admita de forma nativa como matrices, objetos JSON, XML y binario.
Posteriormente, puede usar la salida de la acción en otras acciones. 

```json
"Compose": {
   "type": "Compose",
   "inputs": "<inputs-to-compose>",
   "runAfter": {}
},
```

*Obligatorio* 

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*inputs-to-compose*> | Cualquiera | Las entradas para la creación de una única salida | 
|||| 

*Ejemplo 1*

<!-- markdownlint-disable MD038 -->
Esta definición de acción combina `abcdefg ` con un espacio final y con el valor `1234`:
<!-- markdownlint-enable MD038 -->

```json
"Compose": {
   "type": "Compose",
   "inputs": "abcdefg 1234",
   "runAfter": {}
},
```

Esta es la salida que esta acción crea:

`abcdefg 1234`

*Ejemplo 2*

Esta definición de la acción combina una variable de cadena que contiene `abcdefg` y una variable de entero que contiene `1234`:

```json
"Compose": {
   "type": "Compose",
   "inputs": "@{variables('myString')}@{variables('myInteger')}",
   "runAfter": {}
},
```

Esta es la salida que esta acción crea:

`"abcdefg1234"`

<a name="run-javascript-code"></a>

### <a name="execute-javascript-code-action"></a>Ejecutar la acción de código de JavaScript

Esta acción se ejecuta un fragmento de código JavaScript y devuelve los resultados a través de un `Result` símbolo (token) que pueden hacer referencia a las acciones posteriores.

```json
"Execute_JavaScript_Code": {
   "type": "JavaScriptCode",
   "inputs": {
      "code": "<JavaScript-code-snippet>",
      "explicitDependencies": {
         "actions": [ <previous-actions> ],
         "includeTrigger": true
      }
   },
   "runAfter": {}
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN |
|-------|------|-------------|
| <*JavaScript-code-snippet*> | Varía | El código JavaScript que se va a ejecutar. Para los requisitos de código y obtener más información, consulte [Add y fragmentos de código de ejecución con el código en línea](../logic-apps/logic-apps-add-run-inline-code.md). <p>En el `code` , atributo de solo lectura, puede usar el fragmento de código `workflowContext` objeto como entrada. Este objeto tiene subpropiedades que dan al código acceso a los resultados de los desencadenadores y acciones anteriores en el flujo de trabajo. Para obtener más información sobre la `workflowContext` de objetos, consulte [hacen referencia a los resultados de desencadenadores y las acciones en el código](../logic-apps/logic-apps-add-run-inline-code.md#workflowcontext). |
||||

*Necesario en algunos casos*

El `explicitDependencies` atributo especifica que van a incluir explícitamente los resultados desde el desencadenador, las acciones anteriores o ambos como dependencias para el fragmento de código. Para obtener más información acerca de cómo agregar estas dependencias, consulte [agregar parámetros para el código insertado](../logic-apps/logic-apps-add-run-inline-code.md#add-parameters). 

Para el `includeTrigger` atributo, puede especificar `true` o `false` valores.

| Valor | Type | DESCRIPCIÓN |
|-------|------|-------------|
| <*previous-actions*> | Matriz de cadena | Una matriz con los nombres de acción especificado. Use los nombres de acción que aparecen en la definición de flujo de trabajo donde los nombres de acción usan caracteres de subrayado (_), no espacios (""). |
||||

*Ejemplo 1*

Esta acción ejecuta código que obtiene el nombre de la aplicación lógica y devuelve el texto "Hello world de < nombre de la aplicación lógica >" como el resultado. En este ejemplo, el código hace referencia a nombre del flujo de trabajo mediante el acceso a la `workflowContext.workflow.name` propiedad a través de solo lectura `workflowContext` objeto. Para obtener más información sobre el uso de la `workflowContext` de objetos, consulte [hacen referencia a los resultados de desencadenadores y las acciones en el código](../logic-apps/logic-apps-add-run-inline-code.md#workflowcontext).

```json
"Execute_JavaScript_Code": {
   "type": "JavaScriptCode",
   "inputs": {
      "code": "var text = \"Hello world from \" + workflowContext.workflow.name;\r\n\r\nreturn text;"
   },
   "runAfter": {}
}
```

*Ejemplo 2*

Esta acción ejecuta código en una aplicación lógica que se desencadena cuando un nuevo correo electrónico llega en una cuenta de Office 365 Outlook. La aplicación lógica también usa una acción de correo electrónico de aprobación de envío que reenvía el contenido de correo electrónico recibido junto con una solicitud de aprobación. 

El código extrae las direcciones de correo electrónico desde el desencadenador `Body` propiedad y devuelve las direcciones de correo electrónico junto con la `SelectedOption` valor de propiedad de la acción de aprobación. La acción incluya explícitamente la acción de correo electrónico de aprobación de envío como una dependencia en el `explicitDependencies`  >  `actions` atributo.

```json
"Execute_JavaScript_Code": {
   "type": "JavaScriptCode",
   "inputs": {
      "code": "var re = /(([^<>()\\[\\]\\\\.,;:\\s@\"]+(\\.[^<>()\\[\\]\\\\.,;:\\s@\"]+)*)|(\".+\"))@((\\[[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}\\.[0-9]{1,3}])|(([a-zA-Z\\-0-9]+\\.)+[a-zA-Z]{2,}))/g;\r\n\r\nvar email = workflowContext.trigger.outputs.body.Body;\r\n\r\nvar reply = workflowContext.actions.Send_approval_email_.outputs.body.SelectedOption;\r\n\r\nreturn email.match(re) + \" - \" + reply;\r\n;",
      "explicitDependencies": {
         "actions": [
            "Send_approval_email_"
         ]
      }
   },
   "runAfter": {}
}
```



<a name="function-action"></a>

### <a name="function-action"></a>Acción de la función

Esta acción llama a una [función de Azure](../azure-functions/functions-create-first-azure-function.md) creada previamente.

```json
"<Azure-function-name>": {
   "type": "Function",
   "inputs": {
     "function": {
        "id": "<Azure-function-ID>"
      },
      "method": "<method-type>",
      "headers": { "<header-content>" },
      "body": { "<body-content>" },
      "queries": { "<query-parameters>" } 
   },
   "runAfter": {}
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------|  
| <*Azure-function-ID*> | string | El identificador de recurso de la función de Azure que quiere llamar. Este es el formato de este valor:<p>"/subscriptions/<*Azure-subscription-ID*>/resourceGroups/<*Azure-resource-group*>/providers/Microsoft.Web/sites/<*Azure-function-app-name*>/functions/<*Azure-function-name*>" | 
| <*method-type*> | String | El método HTTP que se usará para llamar a la función: "GET", "PUT", "POST", "PATCH" o "DELETE" <p>Si no se especifica, "POST" es el método predeterminado. | 
||||

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------|  
| <*header-content*> | Objeto JSON | Todos los encabezados que se vayan a enviar con la llamada <p>Por ejemplo, para establecer el idioma y el tipo en una solicitud: <p>`"headers": { "Accept-Language": "en-us", "Content-Type": "application/json" }` |
| <*body-content*> | Objeto JSON | Cualquier contenido de mensaje que se vaya a enviar en la solicitud | 
| <*query-parameters*> | Objeto JSON | Cualquier parámetro de consulta que desee incluir con la llamada API <p>Por ejemplo, el objeto `"queries": { "api-version": "2018-01-01" }` agrega `?api-version=2018-01-01` a la llamada. | 
| <*other-action-specific-input-properties*> | Objeto JSON | Cualquier otra propiedad de entrada que sea aplicable a esta acción específica | 
| <*other-action-specific-properties*> | Objeto JSON | Cualquier otra propiedad que sea aplicable a esta acción específica | 
||||

Cuando se guarda la aplicación lógica, el motor de Logic Apps realiza algunas de estas comprobaciones en la función a la que se hace referencia:

* El flujo de trabajo debe tener acceso a la función.

* El flujo de trabajo solo puede usar un desencadenador HTTP estándar o webhook JSON genérico. 

  El motor de Logic Apps obtiene y almacena en caché la dirección URL del desencadenador, que se utiliza en tiempo de ejecución. Sin embargo, si una operación invalida la dirección URL almacenada en caché, se produce un error en tiempo de ejecución en la acción **Function**. Para corregir este problema, vuelva a guardar la aplicación lógica de forma que esta obtenga y almacene en caché la dirección URL del desencadenador de nuevo.

* La función no puede tener ninguna ruta definida.

* Solo se permiten los niveles de autorización "function" y "anonymous". 

*Ejemplo*

Esta definición de acción llama a la función "GetProductID" creada anteriormente:

```json
"GetProductID": {
   "type": "Function",
   "inputs": {
     "function": {
        "id": "/subscriptions/<XXXXXXXXXXXXXXXXXXXX>/resourceGroups/myLogicAppResourceGroup/providers/Microsoft.Web/sites/InventoryChecker/functions/GetProductID"
      },
      "method": "POST",
      "headers": { 
          "x-ms-date": "@utcnow()"
       },
      "body": { 
          "Product_ID": "@variables('ProductID')"
      }
   },
   "runAfter": {}
}
```

<a name="http-action"></a>

### <a name="http-action"></a>Acción HTTP

Esta acción envía una solicitud al punto de conexión especificado y comprueba la respuesta para determinar si se debe ejecutar el flujo de trabajo. 

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "<method-type>",
      "uri": "<HTTP-or-HTTPS-endpoint-URL>"
   },
   "runAfter": {}
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*method-type*> | String | El método que se usará para enviar la solicitud: "GET", "PUT", "POST", "PATCH" o "DELETE" | 
| <*HTTP-or-HTTPS-endpoint-URL*> | String | El punto de conexión HTTP o HTTPS al que se llama. Tamaño máximo de la cadena: 2 KB | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*header-content*> | Objeto JSON | Cualquier encabezado que se vaya a enviar con la solicitud <p>Por ejemplo, para establecer el idioma y el tipo: <p>`"headers": { "Accept-Language": "en-us", "Content-Type": "application/json" }` |
| <*body-content*> | Objeto JSON | Cualquier contenido de mensaje que se vaya a enviar en la solicitud | 
| <*retry-behavior*> | Objeto JSON | Personaliza el comportamiento de reintento para errores intermitentes, que tienen el código de estado 408, 429 y 5XX, y todas las excepciones de conectividad. Para más información, consulte [Directivas de reintentos](../logic-apps/logic-apps-exception-handling.md#retry-policies). | 
| <*query-parameters*> | Objeto JSON | Cualquier parámetro de consulta que desee incluir con la solicitud <p>Por ejemplo, el objeto `"queries": { "api-version": "2018-01-01" }` agrega `?api-version=2018-01-01` a la llamada. | 
| <*other-action-specific-input-properties*> | Objeto JSON | Cualquier otra propiedad de entrada que sea aplicable a esta acción específica | 
| <*other-action-specific-properties*> | Objeto JSON | Cualquier otra propiedad que sea aplicable a esta acción específica | 
|||| 

*Ejemplo*

Esta definición de acción obtiene las últimas noticias mediante el envío de una solicitud al punto de conexión especificado:

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "https://mynews.example.com/latest"
   }
}
```

<a name="join-action"></a>

### <a name="join-action"></a>Acción Combinar

Esta acción crea una cadena con todos los elementos de una matriz y los separa con el carácter delimitador especificado. 

```json
"Join": {
   "type": "Join",
   "inputs": {
      "from": <array>,
      "joinWith": "<delimiter>"
   },
   "runAfter": {}
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*array*> | Matriz | La matriz o expresión que proporciona los elementos de origen. Si especifica una expresión, incluya esa expresión entre comillas dobles. | 
| <*delimitador*> | Cadena de un único carácter | El carácter que separa cada elemento de la cadena | 
|||| 

*Ejemplo*

Supongamos que ha creado anteriormente la variable "myIntegerArray" que contiene esta matriz de enteros: 

`[1,2,3,4]` 

Esta definición de acción obtiene los valores de la variable mediante la función `variables()` en una expresión y crea esta cadena con esos valores separados por una coma: `"1,2,3,4"`

```json
"Join": {
   "type": "Join",
   "inputs": {
      "from": "@variables('myIntegerArray')",
      "joinWith": ","
   },
   "runAfter": {}
}
```

<a name="parse-json-action"></a>

### <a name="parse-json-action"></a>Acción Análisis del archivo JSON

Esta acción crea campos o *tokens* fáciles de usar a partir de las propiedades del contenido JSON. Posteriormente, puede acceder a esas propiedades de la aplicación lógica mediante los tokens. Por ejemplo, si desea utilizar una salida JSON de servicios como Azure Service Bus y Azure Cosmos DB, puede incluir esta acción en la aplicación lógica para que pueda hacer referencia más fácilmente a los datos de esa salida. 

```json
"Parse_JSON": {
   "type": "ParseJson",
   "inputs": {
      "content": "<JSON-source>",
         "schema": { "<JSON-schema>" }
      },
      "runAfter": {}
},
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*JSON-source*> | Objeto JSON | El contenido JSON que desea analizar | 
| <*JSON-schema*> | Objeto JSON | El esquema JSON que describe el contenido JSON subyacente, que la acción utiliza para analizar el contenido JSON de origen. <p>**Sugerencia**: En el diseñador de Logic Apps, puede proporcionar el esquema, o bien una carga de ejemplo para que la acción pueda generar el esquema. | 
|||| 

*Ejemplo*

Definición de esta acción crea estos tokens que puede usar en el flujo de trabajo, pero solo en las acciones que ejecute los siguientes el **analizar JSON** acción: 

`FirstName`, `LastName` y `Email`

```json
"Parse_JSON": {
   "type": "ParseJson",
   "inputs": {
      "content": {
         "Member": {
            "Email": "Sophie.Owen@contoso.com",
            "FirstName": "Sophie",
            "LastName": "Owen"
         }
      },
      "schema": {
         "type": "object",
         "properties": {
            "Member": {
               "type": "object",
               "properties": {
                  "Email": {
                     "type": "string"
                  },
                  "FirstName": {
                     "type": "string"
                  },
                  "LastName": {
                     "type": "string"
                  }
               }
            }
         }
      }
   },
   "runAfter": { }
},
```

En este ejemplo, la propiedad "content" especifica el contenido JSON de la acción que se va a analizar. También puede proporcionar este contenido JSON como la carga de ejemplo para generar el esquema.

```json
"content": {
   "Member": { 
      "FirstName": "Sophie",
      "LastName": "Owen",
      "Email": "Sophie.Owen@contoso.com"
   }
},
```

La propiedad "schema" especifica el esquema JSON que se usa para describir el contenido JSON:

```json
"schema": {
   "type": "object",
   "properties": {
      "Member": {
         "type": "object",
         "properties": {
            "FirstName": {
               "type": "string"
            },
            "LastName": {
               "type": "string"
            },
            "Email": {
               "type": "string"
            }
         }
      }
   }
}
```

<a name="query-action"></a>

### <a name="query-action"></a>Acción de consulta

Esta acción crea una matriz a partir de elementos de otra matriz basándose en una condición o un filtro especificados.

```json
"Filter_array": {
   "type": "Query",
   "inputs": {
      "from": <array>,
      "where": "<condition-or-filter>"
   },
   "runAfter": {}
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*array*> | Matriz | La matriz o expresión que proporciona los elementos de origen. Si especifica una expresión, incluya esa expresión entre comillas dobles. |
| <*condition-or-filter*> | string | La condición usada para filtrar elementos en la matriz de origen <p>**Nota**: Si ningún valor satisface la condición, la acción crea una matriz vacía. |
|||| 

*Ejemplo*

Esta definición de acción crea una matriz que contiene valores superiores al valor especificado, que es dos:

```json
"Filter_array": {
   "type": "Query",
   "inputs": {
      "from": [ 1, 3, 0, 5, 4, 2 ],
      "where": "@greater(item(), 2)"
   }
}
```

<a name="response-action"></a>

### <a name="response-action"></a>Acción de respuesta  

Esta acción crea la carga de la respuesta a una solicitud HTTP. 

```json
"Response" {
    "type": "Response",
    "kind": "http",
    "inputs": {
        "statusCode": 200,
        "headers": { <response-headers> },
        "body": { <response-body> }
    },
    "runAfter": {}
},
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*response-status-code*> | Integer | El código de estado HTTP que se envía a la solicitud entrante. El código predeterminado es "200 OK", pero el código puede ser cualquier código de estado válido que comience por 2xx, 4xx o 5xx, pero no por 3xxx. | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*response-headers*> | Objeto JSON | Uno o más encabezados que se incluyen con la respuesta | 
| <*response-body*> | Varios | El cuerpo de respuesta, que puede ser una cadena, un objeto JSON o incluso contenido binario de una acción anterior | 
|||| 

*Ejemplo*

Esta definición de acción crea una respuesta a una solicitud HTTP con el código de estado, el cuerpo del mensaje y los encabezados de mensaje especificados:

```json
"Response": {
   "type": "Response",
   "inputs": {
      "statusCode": 200,
      "body": {
         "ProductID": 0,
         "Description": "Organic Apples"
      },
      "headers": {
         "x-ms-date": "@utcnow()",
         "content-type": "application/json"
      }
   },
   "runAfter": {}
}
```

*Restricciones*

A diferencia de otras acciones, la acción **Respuesta** tiene restricciones especiales: 

* El flujo de trabajo solo puede usar la acción **Respuesta** si empieza por un desencadenador de solicitud HTTP, lo cual significa que el flujo de trabajo se debe desencadenar mediante una solicitud HTTP.

* El flujo de trabajo puede usar la acción **Respuesta** en cualquier lugar *excepto* dentro de los bucles **Foreach** y los bucles **Until**, incluidos los bucles secuenciales y las ramas paralelas. 

* La solicitud HTTP original obtiene la respuesta del flujo de trabajo solo cuando todas las acciones requeridas por la acción **Respuesta** han finalizado dentro del [límite de tiempo de espera de la solicitud HTTP](../logic-apps/logic-apps-limits-and-config.md#request-limits).

  No obstante, si el flujo de trabajo llama a otra aplicación lógica como un flujo de trabajo anidado, el flujo de trabajo primario espera hasta que el anidado termina, independientemente del tiempo que transcurra hasta que eso suceda.

* Cuando el flujo de trabajo usa la acción **Response** y un patrón de respuesta sincrónico, el flujo de trabajo no puede usar el comando **splitOn** en la definición del desencadenador ya que este comando crea múltiples ejecuciones. Comprueba si este es el caso cuando se usa el método PUT, y en caso de que así sea, devuelve una respuesta de "solicitud incorrecta".

  En caso contrario, si el flujo de trabajo usa el comando **splitOn** y una acción **Respuesta**, el flujo de trabajo se ejecuta de forma asincrónica y devuelve inmediatamente una respuesta "202-Aceptado".

* Si la ejecución del flujo de trabajo alcanza la acción **Respuesta**, pero la solicitud entrante ya ha recibido una respuesta, la acción **Respuesta** se marcará como "Errónea" debido al conflicto. Y, como resultado, la ejecución de la aplicación lógica también se marcará con el estado "Erróneo".

<a name="select-action"></a>

### <a name="select-action"></a>Acción Select

Esta acción crea una matriz con objetos JSON mediante la transformación de los elementos de otra matriz según la asignación especificada. La matriz de salida y la matriz de origen siempre tienen el mismo número de elementos. Aunque no puede cambiar el número de objetos de la matriz de salida, puede agregar o quitar propiedades y sus valores para esos objetos. La propiedad `select` especifica al menos un par clave-valor que define la asignación para transformar los elementos de la matriz de origen. Un par clave-valor representa una propiedad y su valor en todos los objetos de la matriz de salida. 

```json
"Select": {
   "type": "Select",
   "inputs": {
      "from": <array>,
      "select": { 
          "<key-name>": "<expression>",
          "<key-name>": "<expression>"        
      }
   },
   "runAfter": {}
},
```

*Obligatorio* 

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*array*> | Matriz | La matriz o expresión que proporciona los elementos de origen. Asegúrese de que incluye una expresión entre comillas dobles. <p>**Nota**: Si la matriz de origen está vacía, la acción creará una matriz vacía. | 
| <*key-name*> | String | El nombre de la propiedad asignado al resultado de <*expression*> <p>Para agregar una nueva propiedad en todos los objetos de la matriz de salida, proporcione un <*nombre de clave*> para esa propiedad y una <*expresión*> para el valor de propiedad. <p>Para quitar una propiedad de todos los objetos de la matriz, omita el <*nombre de clave*> para esa propiedad. | 
| <*expresión*> | String | La expresión que transforma el elemento de la matriz de origen y asigna el resultado al <*nombre de clave*> | 
|||| 

La acción **Seleccionar** crea una matriz como salida, por lo que cualquier acción que desee usar esta salida debe aceptar una matriz, o debe convertir la matriz en un tipo que acepte la acción del consumidor. Por ejemplo, para convertir la matriz de salida en una cadena, puede pasar esa matriz a la acción **Redactar** y, a continuación, hacer referencia a la salida de la acción **Redactar** en las demás acciones.

*Ejemplo*

Esta definición de acción crea una matriz de objetos JSON a partir de una matriz de enteros. La acción itera la matriz de origen, obtiene cada valor entero mediante la expresión `@item()` y asigna cada valor a la propiedad "`number`" de cada objeto JSON: 

```json
"Select": {
   "type": "Select",
   "inputs": {
      "from": [ 1, 2, 3 ],
      "select": { 
         "number": "@item()" 
      }
   },
   "runAfter": {}
},
```

Esta es la matriz que esta acción crea:

`[ { "number": 1 }, { "number": 2 }, { "number": 3 } ]`

Para usar esta salida de matriz en otras acciones, pásela a una acción **Redactar**:

```json
"Compose": {
   "type": "Compose",
   "inputs": "@body('Select')",
   "runAfter": {
      "Select": [ "Succeeded" ]
   }
},
```

Después, puede usar la salida de la acción **Redactar** en otras acciones como, por ejemplo, la acción **Office 365 Outlook: enviar un correo electrónico**:

```json
"Send_an_email": {
   "type": "ApiConnection",
   "inputs": {
      "body": {
         "Body": "@{outputs('Compose')}",
         "Subject": "Output array from Select and Compose actions",
         "To": "<your-email@domain>"
      },
      "host": {
         "connection": {
            "name": "@parameters('$connections')['office365']['connectionId']"
         }
      },
      "method": "post",
      "path": "/Mail"
   },
   "runAfter": {
      "Compose": [ "Succeeded" ]
   }
},
```

<a name="table-action"></a>

### <a name="table-action"></a>Acción Table

Esta acción crea una tabla CSV o HTML a partir de una matriz. Para las matrices con objetos JSON, esta acción crea automáticamente los encabezados de columna a partir de los nombres de propiedad de los objetos. Para las matrices con otros tipos de datos, debe especificar los encabezados de columna y los valores. Por ejemplo, esta matriz incluye las propiedades "ID" y "Product_Name" que puede usar esta acción para los nombres de encabezados de columna:

`[ {"ID": 0, "Product_Name": "Apples"}, {"ID": 1, "Product_Name": "Oranges"} ]` 

```json
"Create_<CSV | HTML>_table": {
   "type": "Table",
   "inputs": {
      "format": "<CSV | HTML>",
      "from": <array>,
      "columns": [ 
         {
            "header": "<column-name>",
            "value": "<column-value>"
         },
         {
            "header": "<column-name>",
            "value": "<column-value>"
         } 
      ]
   },
   "runAfter": {}
}
```

*Obligatorio* 

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <CSV *or* HTML>| string | El formato de la tabla que desea crear | 
| <*array*> | Matriz | La matriz o expresión que proporciona los elementos de origen de la tabla. <p>**Nota**: Si la matriz de origen está vacía, la acción creará una tabla vacía. | 
|||| 

*Opcional*

Para especificar o personalizar los encabezados y los valores de columna, use la matriz `columns`. Cuando los pares `header-value` tienen el mismo nombre de encabezado, sus valores se mostrarán en la misma columna bajo ese nombre de encabezado. En caso contrario, cada encabezado único define una columna única.

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*column-name*> | String | El nombre del encabezado de una columna | 
| <*column-value*> | Cualquiera | El valor de esa columna | 
|||| 

*Ejemplo 1*

Supongamos que ha creado anteriormente la variable "myItemArray" que contiene actualmente esta matriz: 

`[ {"ID": 0, "Product_Name": "Apples"}, {"ID": 1, "Product_Name": "Oranges"} ]`

Esta definición de acción crea una tabla CSV a partir de la variable "myItemArray". La expresión utilizada por la propiedad `from` obtiene la matriz a partir de "myItemArray" mediante la función `variables()`: 

```json
"Create_CSV_table": {
   "type": "Table",
   "inputs": {
      "format": "CSV",
      "from": "@variables('myItemArray')"
   },
   "runAfter": {}
}
```

Esta es la tabla CSV que esta acción crea: 

```
ID,Product_Name 
0,Apples 
1,Oranges 
```

*Ejemplo 2*

Esta definición de acción crea una tabla HTML a partir de la variable "myItemArray". La expresión utilizada por la propiedad `from` obtiene la matriz a partir de "myItemArray" mediante la función `variables()`: 

```json
"Create_HTML_table": {
   "type": "Table",
   "inputs": {
      "format": "HTML",
      "from": "@variables('myItemArray')"
   },
   "runAfter": {}
}
```

Esta es la tabla HTML que esta acción crea: 

<table><thead><tr><th>ID</th><th>Product_Name</th></tr></thead><tbody><tr><td>0</td><td>Apples (Manzanas)</td></tr><tr><td>1</td><td>Oranges</td></tr></tbody></table>

*Ejemplo 3*

Esta definición de acción crea una tabla HTML a partir de la variable "myItemArray". Sin embargo, este ejemplo reemplaza los nombres de los encabezados de columna predeterminados por "Stock_ID" y "Description", y agrega la palabra "Organic" a los valores de la columna "Description".

```json
"Create_HTML_table": {
   "type": "Table",
   "inputs": {
      "format": "HTML",
      "from": "@variables('myItemArray')",
      "columns": [ 
         {
            "header": "Stock_ID",
            "value": "@item().ID"
         },
         {
            "header": "Description",
            "value": "@concat('Organic ', item().Product_Name)"
         }
      ]
    },
   "runAfter": {}
},
```

Esta es la tabla HTML que esta acción crea: 

<table><thead><tr><th>Stock_ID</th><th>DESCRIPCIÓN</th></tr></thead><tbody><tr><td>0</td><td>Organic Apples (Manzanas orgánicas)</td></tr><tr><td>1</td><td>Organic Oranges (Naranjas orgánicas)</td></tr></tbody></table>

<a name="terminate-action"></a>

### <a name="terminate-action"></a>Acción Terminate

Esta acción detiene la ejecución de una instancia de flujo de trabajo, cancela cualquier acción en curso, omite las acciones restantes y devuelve el estado especificado. Por ejemplo, puede usar la acción **Terminate** si la aplicación lógica debe salir completamente después de un estado de error. Esta acción no afecta a las acciones ya finalizadas y no puede aparecer dentro de bucles **Foreach** y **Until**, incluidos los bucles secuenciales. 

```json
"Terminate": {
   "type": "Terminate",
   "inputs": {
       "runStatus": "<status>",
       "runError": {
            "code": "<error-code-or-name>",
            "message": "<error-message>"
       }
   },
   "runAfter": {}
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*status*> | string | El estado que se devolverá para la ejecución: "Erróneo", "Cancelado" o "Correcto" |
|||| 

*Opcional*

Las propiedades del objeto "runStatus" se aplican solo cuando se establece la propiedad "runStatus" en estado "Erróneo".

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*error-code-or-name*> | string | El código o nombre del error |
| <*error-message*> | String | El mensaje o el texto que describe el error y las acciones que el usuario de la aplicación puede realizar | 
|||| 

*Ejemplo*

Esta definición de acción detiene la ejecución de un flujo de trabajo, establece el estado de la ejecución en "Erróneo" y devuelve el estado, un código de error y un mensaje de error:

```json
"Terminate": {
    "type": "Terminate",
    "inputs": {
        "runStatus": "Failed",
        "runError": {
            "code": "Unexpected response",
            "message": "The service received an unexpected response. Please try again."
        }
   },
   "runAfter": {}
}
```

<a name="wait-action"></a>

### <a name="wait-action"></a>Acción Wait  

Esta acción detiene la ejecución del flujo de trabajo durante el intervalo especificado o hasta la hora especificada, pero no ambas opciones. 

*Intervalo especificado*

```json
"Delay": {
   "type": "Wait",
   "inputs": {
      "interval": {
         "count": <number-of-units>,
         "unit": "<interval>"
      }
   },
   "runAfter": {}
},
```

*Hora especificada*

```json
"Delay_until": {
   "type": "Wait",
   "inputs": {
      "until": {
         "timestamp": "<date-time-stamp>"
      }
   },
   "runAfter": {}
},
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*number-of-units*> | Integer | Para la acción **Delay** es el número de unidades que debe esperar | 
| <*intervalo*> | String | Para la acción de **retraso**, el intervalo que se esperará: "Second", "Minute", "Hour", "Day", "Week", "Month" | 
| <*date-time-stamp*> | String | Para la acción **Delay Until**, la fecha y hora en la que se reanudará la ejecución. Este valor debe usar el [formato UTC de fecha y hora](https://en.wikipedia.org/wiki/Coordinated_Universal_Time). | 
|||| 

*Ejemplo 1*

Esta definición de acción detiene el flujo de trabajo durante 15 minutos:

```json
"Delay": {
   "type": "Wait",
   "inputs": {
      "interval": {
         "count": 15,
         "unit": "Minute"
      }
   },
   "runAfter": {}
},
```

*Ejemplo 2*

Esta definición de acción detiene el flujo de trabajo hasta la hora especificada:

```json
"Delay_until": {
   "type": "Wait",
   "inputs": {
      "until": {
         "timestamp": "2017-10-01T00:00:00Z"
      }
   },
   "runAfter": {}
},
```

<a name="workflow-action"></a>

### <a name="workflow-action"></a>Acción Workflow

Esta acción llama a otra aplicación lógica creada anteriormente, lo que significa que puede incluir y volver a usar otros flujos de trabajo de la aplicación lógica. También puede usar las salidas de la aplicación lógica secundaria o *anidada* en acciones que sigan a la aplicación lógica anidada siempre que la aplicación lógica secundaria devuelva una respuesta.

El motor de Logic Apps comprueba el acceso al desencadenador que desea llamar, así que asegúrese de que puede acceder a ese desencadenador. Además, la aplicación lógica anidada debe cumplir estos criterios:

* Un desencadenador hace que la aplicación lógica anidada sea invocable como, por ejemplo, un desencadenador [Request](#request-trigger) o [HTTP](#http-trigger)

* La misma suscripción de Azure que su aplicación lógica primaria

* Para usar las salidas de la aplicación lógica anidada en la aplicación lógica primaria, la aplicación lógica anidada debe tener una acción [Respuesta](#response-action) 

```json
"<nested-logic-app-name>": {
   "type": "Workflow",
   "inputs": {
      "body": { "<body-content" },
      "headers": { "<header-content>" },
      "host": {
         "triggerName": "<trigger-name>",
         "workflow": {
            "id": "/subscriptions/<Azure-subscription-ID>/resourceGroups/<Azure-resource-group>/providers/Microsoft.Logic/<nested-logic-app-name>"
         }
      }
   },
   "runAfter": {}
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*nested-logic-app-name*> | String | El nombre de la aplicación lógica a la que desea llamar | 
| <*trigger-name*> | String | El nombre del desencadenador de la aplicación lógica anidada a la que desea llamar | 
| <*Azure-subscription-ID*> | string | El identificador de suscripción de Azure para la aplicación lógica anidada |
| <*Azure-resource-group*> | String | El nombre del grupo de recursos de Azure para la aplicación lógica anidada |
| <*nested-logic-app-name*> | string | El nombre de la aplicación lógica a la que desea llamar |
||||

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------|  
| <*header-content*> | Objeto JSON | Todos los encabezados que se vayan a enviar con la llamada | 
| <*body-content*> | Objeto JSON | Cualquier contenido de mensaje que se vaya a enviar con la llamada | 
||||

*Outputs*

Las salidas de esta acción varían en función de la acción Respuesta de la aplicación lógica anidada. Si la aplicación lógica anidada no incluye una acción Respuesta, las salidas estarán vacías.

*Ejemplo*

Una vez completada la acción "Start_search" correctamente, esta definición de acción del flujo de trabajo llama a otra aplicación lógica denominada "Get_product_information", que pasa las entradas especificadas: 

```json
"actions": {
   "Start_search": { <action-definition> },
   "Get_product_information": {
      "type": "Workflow",
      "inputs": {
         "body": {
            "ProductID": "24601",
         },
         "host": {
            "id": "/subscriptions/XXXXXXXXXXXXXXXXXXXXXXXXXX/resourceGroups/InventoryManager-RG/providers/Microsoft.Logic/Get_product_information",
            "triggerName": "Find_product"
         },
         "headers": {
            "content-type": "application/json"
         }
      },
      "runAfter": { 
         "Start_search": [ "Succeeded" ]
      }
   }
},
```

## <a name="control-workflow-action-details"></a>Detalles de la acción de control del flujo de trabajo

<a name="foreach-action"></a>

### <a name="foreach-action"></a>Acción Foreach

Esta acción de bucle recorre en iteración una matriz y realiza acciones en cada elemento de la matriz. De forma predeterminada, el bucle "for each" se ejecuta en paralelo hasta alcanzar un número máximo de bucles. Para conocer este valor máximo, consulte [Límites y configuración](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Aprenda [a crear bucles "for each"](../logic-apps/logic-apps-control-flow-loops.md#foreach-loop).

```json
"For_each": {
   "type": "Foreach",
   "actions": { 
      "<action-1>": { "<action-definition-1>" },
      "<action-2>": { "<action-definition-2>" }
   },
   "foreach": "<for-each-expression>",
   "runAfter": {},
   "runtimeConfiguration": {
      "concurrency": {
         "repetitions": <count>
      }
    },
    "operationOptions": "<operation-option>"
}
```

*Obligatorio* 

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*action-1...n*> | string | Los nombres de las acciones que se ejecutan en cada elemento de la matriz | 
| <*action-definition-1...n*> | Objeto JSON | Las definiciones de las acciones que se ejecutan | 
| <*for-each-expression*> | String | La expresión que hace referencia a cada elemento de la matriz especificada | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*recuento*> | Integer | De forma predeterminada, las iteraciones de bucles "for each" se ejecutan al mismo tiempo o en paralelo hasta el [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar este límite con un nuevo valor <*count*>, consulte [Cambio de la simultaneidad del bucle "for each"](#change-for-each-concurrency). | 
| <*operation-option*> | String | Para ejecutar un bucle "for each" secuencialmente, en lugar de en paralelo, establezca <*operation-option*> en `Sequential` o <*count*> en `1`, pero no ambas opciones a la vez. Para más información, consulte [Ejecución secuencial de bucles "for each"](#sequential-for-each). | 
|||| 

*Ejemplo*

Este bucle "for each" envía un correo electrónico para cada elemento de la matriz que contiene los datos adjuntos de un correo electrónico entrante. El bucle envía un correo electrónico, con los datos adjuntos incluidos, a una persona que los revisa.

```json
"For_each": {
   "type": "Foreach",
   "actions": {
      "Send_an_email": {
         "type": "ApiConnection",
         "inputs": {
            "body": {
               "Body": "@base64ToString(items('For_each')?['Content'])",
               "Subject": "Review attachment",
               "To": "Sophie.Owen@contoso.com"
                },
            "host": {
               "connection": {
                  "id": "@parameters('$connections')['office365']['connectionId']"
               }
            },
            "method": "post",
            "path": "/Mail"
         },
         "runAfter": {}
      }
   },
   "foreach": "@triggerBody()?['Attachments']",
   "runAfter": {}
}
```

Para especificar solo una matriz que se pase como salida desde el desencadenador, esta expresión obtiene la matriz <*array-name*> del cuerpo del desencadenador. Para evitar un error si la matriz no existe, la expresión utiliza el operador `?`:

`@triggerBody()?['<array-name>']` 

<a name="if-action"></a>

### <a name="if-action"></a>Acción If

Esta acción, que es una *instrucción condicional*, evalúa una expresión que representa una condición y ejecuta una rama diferente en función de si la condición es true o false. Si la condición es true, la condición se marca con el estado "Correcto". Aprenda [a crear instrucciones condicionales](../logic-apps/logic-apps-control-flow-conditional-statement.md).

``` json
"Condition": {
   "type": "If",
   "expression": { "<condition>" },
   "actions": {
      "<action-1>": { "<action-definition>" }
   },
   "else": {
      "actions": {
        "<action-2>": { "<action-definition" }
      }
   },
   "runAfter": {}
}
```

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*condition*> | Objeto JSON | La condición que se va a evaluar, que puede ser una expresión | 
| <*action-1*> | Objeto JSON | La acción que se ejecutará si <*condition*> se evalúa como true | 
| <*action-definition*> | Objeto JSON | La definición de la acción | 
| <*action-2*> | Objeto JSON | La acción que se ejecutará si <*condition*> se evalúa como false | 
|||| 

Las acciones de los objetos `actions` o `else` obtienen estos estados:

* "Succeeded" cuando se ejecutan y lo hacen correctamente
* "Failed" cuando se ejecutan pero no lo hacen correctamente
* "Skipped" cuando la rama correspondiente no se ejecuta

*Ejemplo*

Esta condición especifica que si la variable de entero tiene un valor mayor que cero, el flujo de trabajo comprobará un sitio web. Si la variable es cero o menos, el flujo de trabajo comprobará un sitio web diferente.

```json
"Condition": {
   "type": "If",
   "expression": {
      "and": [ {
         "greater": [ "@variables('myIntegerVariable')", 0 ] 
      } ]
   },
   "actions": { 
      "HTTP - Check this website": {
         "type": "Http",
         "inputs": {
         "method": "GET",
            "uri": "http://this-url"
         },
         "runAfter": {}
      }
   },
   "else": {
      "actions": {
         "HTTP - Check this other website": {
            "type": "Http",
            "inputs": {
               "method": "GET",
               "uri": "http://this-other-url"
            },
            "runAfter": {}
         }
      }
   },
   "runAfter": {}
}
```  

#### <a name="how-conditions-use-expressions"></a>Uso de expresiones en las condiciones

A continuación se muestran algunos ejemplos de cómo puede usar expresiones en condiciones:
  
| JSON | Resultado | 
|------|--------| 
| "expression": "@parameters('<*hasSpecialAction*>')" | En el caso de las expresiones booleanas, la condición se pasa para cualquier valor que se evalúe como true. <p>Para convertir otros tipos a booleanas, use las funciones `empty()` o `equals()`. | 
| "expression": "@greater(actions('<*action*>').output.value, parameters('<*threshold*>'))" | En el caso de las funciones de comparación, la acción se ejecuta solo si la salida de <*action*> es superior al valor de <*threshold*> valor. | 
| "expression": "@or(greater(actions('<*action*>').output.value, parameters('<*threshold*>')), less(actions('<*same-action*>').output.value, 100))" | En el caso de las funciones lógicas y la creación de expresiones booleanas anidadas, la acción se ejecuta si la salida de <*action*> es superior al valor de <*threshold*> o inferior a 100. | 
| "expression": "@equals(length(actions('<*action*>').outputs.errors), 0))" | Puede usar funciones de matriz para comprobar si la matriz tiene elementos. La acción se ejecuta cuando la matriz `errors` está vacía. | 
||| 

<a name="scope-action"></a>

### <a name="scope-action"></a>Acción Scope

Esta acción agrupa lógicamente las acciones en *ámbitos*, que obtienen su propio estado después de que las acciones del ámbito terminen de ejecutarse. A continuación, puede usar el estado del ámbito para determinar si se ejecutan otras acciones. Aprenda [a crear ámbitos](../logic-apps/logic-apps-control-flow-run-steps-group-scopes.md).

```json
"Scope": {
   "type": "Scope",
   "actions": {
      "<inner-action-1>": {
         "type": "<action-type>",
         "inputs": { "<action-inputs>" },
         "runAfter": {}
      },
      "<inner-action-2>": {
         "type": "<action-type>",
         "inputs": { "<action-inputs>" },
         "runAfter": {}
      }
   }
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------|  
| <*inner-action-1...n*> | Objeto JSON | Una o varias acciones que se ejecutan dentro del ámbito |
| <*action-inputs*> | Objeto JSON | Las entradas de cada acción |
|||| 

<a name="switch-action"></a>

### <a name="switch-action"></a>Acción Switch

Esta acción, también conocida como *instrucción switch*, organiza otras acciones en *casos* y asigna un valor a cada caso, excepto para el caso predeterminado, si existe alguno. Cuando se ejecuta el flujo de trabajo, la acción **Switch** compara el valor de una expresión, un objeto o un token con los valores especificados para cada caso. Si la acción **Switch** busca un caso coincidente, el flujo de trabajo ejecuta solo las acciones para ese caso. Cada vez que se ejecuta la acción **Switch** existe solo un caso coincidente o ninguno. Si no existe ninguna coincidencia, la acción **Switch** ejecutará las acciones predeterminadas. Aprenda [a crear instrucciones switch](../logic-apps/logic-apps-control-flow-switch-statement.md).

``` json
"Switch": {
   "type": "Switch",
   "expression": "<expression-object-or-token>",
   "cases": {
      "Case": {
         "actions": {
           "<action-name>": { "<action-definition>" }
         },
         "case": "<matching-value>"
      },
      "Case_2": {
         "actions": {
           "<action-name>": { "<action-definition>" }
         },
         "case": "<matching-value>"
      }
   },
   "default": {
      "actions": {
         "<default-action-name>": { "<default-action-definition>" }
      }
   },
   "runAfter": {}
}
```

*Obligatorio*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*expression-object-or-token*> | Varía | La expresión, objeto JSON o token que se va a evaluar | 
| <*action-name*> | string | El nombre de la acción que se va a ejecutar para el caso coincidente | 
| <*action-definition*> | Objeto JSON | La definición de la acción que se va a ejecutar para el caso coincidente | 
| <*matching-value*> | Varía | El valor que se compara con el resultado evaluado | 
|||| 

*Opcional*

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*default-action-name*> | String | El nombre de la acción predeterminada que se va a ejecutar si no hay ningún caso coincidente | 
| <*default-action-definition*> | Objeto JSON | La definición de la acción que se va a ejecutar si no hay ningún caso coincidente | 
|||| 

*Ejemplo*

Esta definición de acción evalúa si la persona que responde al correo electrónico de solicitud de aprobación seleccionó la opción "Aprobar" o la opción "Rechazar". Según esta elección, la acción **Switch** ejecuta las acciones para el caso coincidente, que consiste en enviar otro correo electrónico al respondedor pero con palabras diferentes en cada caso. 

``` json
"Switch": {
   "type": "Switch",
   "expression": "@body('Send_approval_email')?['SelectedOption']",
   "cases": {
      "Case": {
         "actions": {
            "Send_an_email": { 
               "type": "ApiConnection",
               "inputs": {
                  "Body": "Thank you for your approval.",
                  "Subject": "Response received", 
                  "To": "Sophie.Owen@contoso.com"
               },
               "host": {
                  "connection": {
                     "name": "@parameters('$connections')['office365']['connectionId']"
                  }
               },
               "method": "post",
               "path": "/Mail"
            },
            "runAfter": {}
         },
         "case": "Approve"
      },
      "Case_2": {
         "actions": {
            "Send_an_email_2": { 
               "type": "ApiConnection",
               "inputs": {
                  "Body": "Thank you for your response.",
                  "Subject": "Response received", 
                  "To": "Sophie.Owen@contoso.com"
               },
               "host": {
                  "connection": {
                     "name": "@parameters('$connections')['office365']['connectionId']"
                  }
               },
               "method": "post",
               "path": "/Mail"
            },
            "runAfter": {}     
         },
         "case": "Reject"
      }
   },
   "default": {
      "actions": { 
         "Send_an_email_3": { 
            "type": "ApiConnection",
            "inputs": {
               "Body": "Please respond with either 'Approve' or 'Reject'.",
               "Subject": "Please respond", 
               "To": "Sophie.Owen@contoso.com"
            },
            "host": {
               "connection": {
                  "name": "@parameters('$connections')['office365']['connectionId']"
               }
            },
            "method": "post",
            "path": "/Mail"
         },
         "runAfter": {} 
      }
   },
   "runAfter": {
      "Send_approval_email": [ 
         "Succeeded"
      ]
   }
}
```

<a name="until-action"></a>

### <a name="until-action"></a>Acción Until

Esta acción de bucle contiene acciones que se ejecutan hasta que la condición especificada es true. El bucle comprueba la condición como último paso después de que todas las demás acciones se han ejecutado. Puede incluir más de una acción en el objeto `"actions"` y la acción debe definir al menos un límite. Aprenda a [crear bucles "until"](../logic-apps/logic-apps-control-flow-loops.md#until-loop). 

```json
 "Until": {
   "type": "Until",
   "actions": {
      "<action-name>": {
         "type": "<action-type>",
         "inputs": { "<action-inputs>" },
         "runAfter": {}
      },
      "<action-name>": {
         "type": "<action-type>",
         "inputs": { "<action-inputs>" },
         "runAfter": {}
      }
   },
   "expression": "<condition>",
   "limit": {
      "count": <loop-count>,
      "timeout": "<loop-timeout>"
   },
   "runAfter": {}
}
```

| Valor | Type | DESCRIPCIÓN | 
|-------|------|-------------| 
| <*action-name*> | String | El nombre de la acción que desea ejecutar dentro del bucle | 
| <*action-type*> | String | El tipo de acción que desea ejecutar | 
| <*action-inputs*> | Varios | Las entradas para que la acción se ejecute | 
| <*condition*> | String | La condición o expresión que va a evaluar una vez finalizada la ejecución de todas las acciones del bucle | 
| <*loop-count*> | Integer | El límite en el mayor número de bucles que puede ejecutar la acción. El valor predeterminado de `count` es 60. | 
| <*loop-timeout*> | string | El límite en el tiempo más largo que puede ejecutar el bucle. El valor predeterminado de `timeout` es `PT1H`, que es el [formato ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) obligatorio. |
|||| 

*Ejemplo*

Esta definición de acción del bucle envía una solicitud HTTP a la dirección URL especificada hasta que se cumple una de estas condiciones: 

* La solicitud obtiene una respuesta con el código de estado "200 OK".
* El bucle se ha ejecutado 60 veces.
* El bucle se ha ejecutado durante una hora.

```json
 "Run_until_loop_succeeds_or_expires": {
    "type": "Until",
    "actions": {
        "Http": {
            "type": "Http",
            "inputs": {
                "method": "GET",
                "uri": "http://myurl"
            },
            "runAfter": {}
        }
    },
    "expression": "@equals(outputs('Http')['statusCode', 200])",
    "limit": {
        "count": 60,
        "timeout": "PT1H"
    },
    "runAfter": {}
}
```

<a name="subscribe-unsubscribe"></a>

## <a name="webhooks-and-subscriptions"></a>Webhooks y suscripciones

Los desencadenadores y acciones basados en webhooks no comprueban regularmente los puntos de conexión, sino que esperan a que se produzcan eventos o datos específicos en esos puntos de conexión. Estos desencadenadores y acciones se *suscriben* a los puntos de conexión proporcionando una *dirección URL de devolución de llamada* en la que el punto de conexión puede enviar respuestas.

La llamada `subscribe` tiene lugar cuando el flujo de trabajo cambia de algún modo; por ejemplo, cada vez que se renuevan las credenciales o cambian los parámetros de entrada de un desencadenador o acción. Esta llamada usa los mismos parámetros que las acciones HTTP estándar. 

La llamada `unsubscribe` se produce automáticamente cuando una operación invalida el desencadenador o la acción, por ejemplo:

* Eliminar o deshabilitar el desencadenador. 
* Eliminar o deshabilitar el flujo de trabajo. 
* Eliminar o deshabilitar la suscripción. 

Para admitir estas llamadas, la expresión `@listCallbackUrl()` devuelve una "dirección URL de devolución de llamada" única para el desencadenador o acción. Esta dirección URL representa un identificador único de los puntos de conexión que usan la API de REST del servicio. Los parámetros de esta función son los mismos que los del desencadenador o acción de webhook.

<a name="asynchronous-limits"></a>

## <a name="change-asynchronous-duration"></a>Cambio de la duración asincrónica

Para los desencadenadores y las acciones, puede limitar la duración del modelo asincrónico a un intervalo de tiempo específico agregando la propiedad `limit.timeout`. De este modo, si no ha finalizado la acción una vez transcurrido el intervalo, el estado de la acción se marcará como `Cancelled` con el código `ActionTimedOut`. La propiedad `timeout` usa el [formato ISO 8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations). 

``` json
"<trigger-or-action-name>": {
   "type": "Workflow | Webhook | Http | ApiConnectionWebhook | ApiConnection",
   "inputs": {},
   "limit": {
      "timeout": "PT10S"
   },
   "runAfter": {}
}
```

<a name="runtime-config-options"></a>

## <a name="runtime-configuration-settings"></a>Valores de configuración del runtime

Puede cambiar el comportamiento predeterminado del runtime para los desencadenadores y acciones con estas propiedades `runtimeConfiguration` en la definición de desencadenador o de acción.

| Propiedad | Escriba | DESCRIPCIÓN | Desencadenador o acción | 
|----------|------|-------------|-------------------| 
| `runtimeConfiguration.concurrency.runs` | Integer | Cambiar el [ *límite predeterminado* ](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits) en el número de instancias de flujo de trabajo que se pueden ejecutar al mismo tiempo, o en paralelo. Este valor puede ayudar a limitar el número de solicitudes que reciben los sistemas de back-end. <p>Establecer la propiedad `runs` en `1` funciona del mismo modo que establecer la propiedad `operationOptions` en `SingleInstance`. Puede establecer una propiedad u otra, pero no ambas. <p>Para cambiar el límite predeterminado, consulte [Cambio en la simultaneidad de desencadenadores](#change-trigger-concurrency) o [Desencadenamiento secuencial de instancias](#sequential-trigger). | Todos los desencadenadores | 
| `runtimeConfiguration.concurrency.maximumWaitingRuns` | Integer | Cambiar el [ *límite predeterminado* ](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits) en el número de instancias de flujo de trabajo que puede esperar que se ejecutará cuando el flujo de trabajo ya se está ejecutando el número máximo de instancias simultáneo. Puede cambiar el límite de simultaneidad en la propiedad `concurrency.runs`. <p>Para cambiar el límite predeterminado, consulte [Cambio del límite de ejecuciones en espera](#change-waiting-runs). | Todos los desencadenadores | 
| `runtimeConfiguration.concurrency.repetitions` | Integer | Cambie el [*límite predeterminado*](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits) en el número de iteraciones de bucles "for each" que se pueden ejecutar al mismo tiempo o en paralelo. <p>Establecer la propiedad `repetitions` en `1` funciona del mismo modo que establecer la propiedad `operationOptions` en `SingleInstance`. Puede establecer una propiedad u otra, pero no ambas. <p>Para cambiar el límite predeterminado, consulte [Cambio de la simultaneidad de los bucles "for each"](#change-for-each-concurrency) o [Ejecución secuencial de bucles "for each"](#sequential-for-each). | Acción: <p>[Foreach](#foreach-action) | 
| `runtimeConfiguration.paginationPolicy.minimumItemCount` | Integer | Para acciones específicas que tienen la paginación activada, este valor especifica el *mínimo* número de resultados que se va a recuperar. <p>Para activar la paginación, vea [obtención masiva de datos, los elementos o los resultados mediante el uso de paginación](../logic-apps/logic-apps-exceed-default-page-size-with-pagination.md) | Acción: Variados |
| `runtimeConfiguration.staticResult` | Objeto JSON | Para las acciones que tienen el [resultado estático](../logic-apps/test-logic-apps-mock-data-static-results.md) activada, el `staticResult` objeto tiene estos atributos: <p>- `name`, que hace referencia el nombre de definición de la acción actual resultado estático, que aparece dentro de la `staticResults` atributo en de la aplicación lógica `definition` atributo. Para obtener más información, consulte [resultados estáticos: referencia de esquema de lenguaje de definición de flujo de trabajo](../logic-apps/logic-apps-workflow-definition-language.md#static-results). <p> - `staticResultOptions`, que especifica si los resultados estáticos son `Enabled` o no para la acción actual. <p>Para activar los resultados estáticos, vea [probar las aplicaciones lógicas con datos simulados mediante la configuración de resultados estáticos](../logic-apps/test-logic-apps-mock-data-static-results.md) | Acción: Variados |
||||| 

<a name="operation-options"></a>

## <a name="operation-options"></a>Opciones de operación

Puede cambiar el comportamiento predeterminado de los desencadenadores y acciones con la propiedad `operationOptions` de la definición de desencadenador o de acción.

| Opción de operación | Type | DESCRIPCIÓN | Desencadenador o acción | 
|------------------|------|-------------|-------------------| 
| `DisableAsyncPattern` | String | Ejecuta acciones basadas en HTTP sincrónicamente en lugar de hacerlo de forma asincrónica. <p><p>Para establecer esta opción, consulte [Ejecutar acciones sincrónicamente](#asynchronous-patterns). | Acciones: <p>[ApiConnection](#apiconnection-action), <br>[HTTP](#http-action), <br>[Respuesta](#response-action) | 
| `OptimizedForHighThroughput` | String | Cambia el [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#throughput-limits) del número de ejecuciones de acciones por cada 5 minutos al [límite máximo](../logic-apps/logic-apps-limits-and-config.md#throughput-limits). <p><p>Para establecer esta opción, consulte [Ejecutar en modo de alto rendimiento](#run-high-throughput-mode). | Todas las acciones | 
| `Sequential` | string | Ejecuta iteraciones de bucles "for each" una a una, en lugar de todas al mismo tiempo en paralelo. <p>Esta opción funciona de la misma manera que establecer la propiedad `runtimeConfiguration.concurrency.repetitions` en `1`. Puede establecer una propiedad u otra, pero no ambas. <p><p>Para más información, consulte [Ejecución secuencial de bucles "for each"](#sequential-for-each).| Acción: <p>[Foreach](#foreach-action) | 
| `SingleInstance` | string | Ejecuta secuencialmente el desencadenador de cada instancia de aplicación lógica y espera a que termine la ejecución anteriormente activa antes de desencadenar la siguiente instancia de aplicación lógica. <p><p>Esta opción funciona de la misma manera que establecer la propiedad `runtimeConfiguration.concurrency.runs` en `1`. Puede establecer una propiedad u otra, pero no ambas. <p>Para establecer esta opción, consulte [Desencadenamiento secuencial de instancias](#sequential-trigger). | Todos los desencadenadores | 
||||

<a name="change-trigger-concurrency"></a>

### <a name="change-trigger-concurrency"></a>Cambio en la simultaneidad de desencadenadores

De forma predeterminada, las instancias de la aplicación lógica se ejecutan al mismo tiempo (simultáneamente) o en paralelo hasta el [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Por lo tanto, cada instancia del desencadenador se activa antes de la instancia de flujo de trabajo anterior finaliza la ejecución. Este límite ayuda a controlar el número de solicitudes que reciben los sistemas de back-end. 

Para cambiar el límite predeterminado, puede usar el editor de la vista Código o el diseñador de Logic Apps ya que al cambiar el valor de simultaneidad a través del diseñador se agrega o actualiza la propiedad `runtimeConfiguration.concurrency.runs` en la definición del desencadenador subyacente y viceversa. Esta propiedad controla el número máximo de instancias de flujo de trabajo que se pueden ejecutar en paralelo. 

> [!NOTE] 
> Si configura el desencadenador para que se ejecute secuencialmente mediante el diseñador o mediante el editor de la vista Código, no establezca la propiedad `operationOptions` del desencadenador en `SingleInstance` en el editor de la vista Código. De lo contrario, obtendrá un error de validación. Para más información, consulte [Desencadenamiento secuencial de instancias](#sequential-trigger).

#### <a name="edit-in-code-view"></a>Edición en la vista Código 

En la definición subyacente del desencadenador, agregue o actualice la propiedad `runtimeConfiguration.concurrency.runs` a un valor entre `1` y `50`, ambos inclusive.

Este es un ejemplo que limita las ejecuciones simultáneas a 10 instancias:

```json
"<trigger-name>": {
   "type": "<trigger-name>",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": 10
      }
   }
}
```

#### <a name="edit-in-logic-apps-designer"></a>Edición en el diseñador de Logic Apps

1. En la esquina superior derecha del desencadenador, elija el botón de los puntos suspensivos (...) y, a continuación, elija **Configuración**.

2. En **Control de simultaneidad**, establezca **Límite** en **Activado**. 

3. Arrastre el control deslizante **Grado de paralelismo** al valor que desee. Para ejecutar la aplicación lógica de manera secuencial, arrastre el valor del control deslizante a **1**.

<a name="change-for-each-concurrency"></a>

### <a name="change-for-each-concurrency"></a>Cambio de la simultaneidad de los bucles "for each"

De forma predeterminada, las iteraciones de bucles "for each" se ejecutan al mismo tiempo o en paralelo hasta el [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Para cambiar el límite predeterminado, puede usar el editor de la vista Código o el diseñador de Logic Apps ya que al cambiar el valor de simultaneidad a través del diseñador se agrega o actualiza la propiedad `runtimeConfiguration.concurrency.repetitions` en la definición de la acción "for each" subyacente y viceversa. Esta propiedad controla el número máximo de iteraciones que se pueden ejecutar en paralelo.

> [!NOTE] 
> Si configura la acción "for each" para que se ejecute secuencialmente mediante el diseñador o mediante el editor de la vista Código, no establezca la propiedad `operationOptions` de la acción en `Sequential` en el editor de la vista Código. De lo contrario, obtendrá un error de validación. Para más información, consulte [Ejecución secuencial de bucles "for each"](#sequential-for-each).

#### <a name="edit-in-code-view"></a>Edición en la vista Código 

En la definición subyacente de la acción "for each", agregue o actualice la propiedad `runtimeConfiguration.concurrency.repetitions` a un valor entre `1` y `50`, ambos inclusive. 

Este es un ejemplo que limita las ejecuciones simultáneas a 10 iteraciones:

```json
"For_each" {
   "type": "Foreach",
   "actions": { "<actions-to-run>" },
   "foreach": "<for-each-expression>",
   "runAfter": {},
   "runtimeConfiguration": {
      "concurrency": {
         "repetitions": 10
      }
   }
}
```

#### <a name="edit-in-logic-apps-designer"></a>Edición en el diseñador de Logic Apps

1. En la acción **For each**, en la esquina superior derecha, elija el botón de puntos suspensivos (...) y luego elija **Configuración**.

2. En **Control de simultaneidad**, establezca **Control de simultaneidad** en **Activado**. 

3. Arrastre el control deslizante **Grado de paralelismo** al valor que desee. Para ejecutar la aplicación lógica de manera secuencial, arrastre el valor del control deslizante a **1**.

<a name="change-waiting-runs"></a>

### <a name="change-waiting-runs-limit"></a>Cambio del límite de ejecuciones en espera

De forma predeterminada, instancias de flujo de trabajo de aplicación lógica todos ejecutan al mismo tiempo, al mismo tiempo, o en paralelo hasta el [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits). Cada instancia del desencadenador se activa antes de que finaliza la ejecución de la instancia de flujo de trabajo activa anteriormente. Aunque puede [cambiar este límite predeterminado](#change-trigger-concurrency), cuando el número de instancias de flujo de trabajo alcanza el límite de simultaneidad nuevo, deben esperar otras nuevas instancias a ejecutar. 

El número de ejecuciones que se pueden poner en espera también tiene un [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#looping-debatching-limits) que puede modificar. No obstante, una vez que la aplicación lógica alcanza el límite de ejecuciones en espera, el motor de Logic Apps ya no acepta nuevas ejecuciones. Los desencadenadores de solicitudes y de webhooks devuelven errores 429 y los desencadenadores recurrentes empiezan a omitir los intentos de sondeo.

Para cambiar el límite predeterminado de las ejecuciones en espera, en la definición del desencadenador subyacente, agregue la propiedad `runtimeConfiguration.concurency.maximumWaitingRuns` con un valor entre `0` y `100`. 

```json
"<trigger-name>": {
   "type": "<trigger-name>",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
   },
   "runtimeConfiguration": {
      "concurrency": {
         "maximumWaitingRuns": 50
      }
   }
}
```

<a name="sequential-trigger"></a>

### <a name="trigger-instances-sequentially"></a>Desencadenamiento secuencial de instancias

Para ejecutar lógica de cada instancia de flujo de trabajo de aplicación solo después de que finalice la instancia anterior está ejecutando, establezca el desencadenador que se ejecutan secuencialmente. Puede usar el editor de la vista Código o el diseñador de Logic Apps ya que al cambiar el valor de simultaneidad a través del diseñador también se agrega o actualiza la propiedad `runtimeConfiguration.concurrency.runs` en la definición del desencadenador subyacente y viceversa. 

> [!NOTE] 
> Si configura un desencadenador para que se ejecute secuencialmente mediante el diseñador o mediante el editor de la vista Código, no establezca la propiedad `operationOptions` del desencadenador en `Sequential` en el editor de la vista Código. De lo contrario, obtendrá un error de validación. 

#### <a name="edit-in-code-view"></a>Edición en la vista Código

En la definición del desencadenador, establezca cualquiera de estas propiedades, pero no ambas. 

Establezca la propiedad `runtimeConfiguration.concurrency.runs` en `1`:

```json
"<trigger-name>": {
   "type": "<trigger-name>",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
   },
   "runtimeConfiguration": {
      "concurrency": {
         "runs": 1
      }
   }
}
```

*o*

Establezca la propiedad `operationOptions` en `SingleInstance`:

```json
"<trigger-name>": {
   "type": "<trigger-name>",
   "recurrence": {
      "frequency": "<time-unit>",
      "interval": <number-of-time-units>,
   },
   "operationOptions": "SingleInstance"
}
```

#### <a name="edit-in-logic-apps-designer"></a>Edición en el diseñador de Logic Apps

1. En la esquina superior derecha del desencadenador, elija el botón de los puntos suspensivos (...) y, a continuación, elija **Configuración**.

2. En **Control de simultaneidad**, establezca **Límite** en **Activado**. 

3. Arrastre el control deslizante **Grado de paralelismo** al número `1`. 

<a name="sequential-for-each"></a>

### <a name="run-for-each-loops-sequentially"></a>Ejecución secuencial de bucles "for each"

Para que una iteración de bucle "for each" solo se ejecute una vez que haya finalizado la ejecución de la iteración anterior, establezca la acción "for each" para que se ejecute de forma secuencial. Puede usar el editor de la vista Código o el diseñador de Logic Apps ya que al cambiar la simultaneidad de la acción a través del diseñador también se agrega o actualiza la propiedad `runtimeConfiguration.concurrency.repetitions` en la definición de acción subyacente y viceversa. 

> [!NOTE] 
> Si configura una acción "for each" para que se ejecute secuencialmente mediante el diseñador o mediante el editor de la vista Código, no establezca la propiedad `operationOptions` de la acción en `Sequential` en el editor de la vista Código. De lo contrario, obtendrá un error de validación. 

#### <a name="edit-in-code-view"></a>Edición en la vista Código

En la definición de la acción, establezca cualquiera de estas propiedades, pero no ambas. 

Establezca la propiedad `runtimeConfiguration.concurrency.repetitions` en `1`:

```json
"For_each" {
   "type": "Foreach",
   "actions": { "<actions-to-run>" },
   "foreach": "<for-each-expression>",
   "runAfter": {},
   "runtimeConfiguration": {
      "concurrency": {
         "repetitions": 1
      }
   }
}
```

*o*

Establezca la propiedad `operationOptions` en `Sequential`:

```json
"For_each" {
   "type": "Foreach",
   "actions": { "<actions-to-run>" },
   "foreach": "<for-each-expression>",
   "runAfter": {},
   "operationOptions": "Sequential"
}
```

#### <a name="edit-in-logic-apps-designer"></a>Edición en el diseñador de Logic Apps

1. En la esquina superior derecha de la acción **For each**, elija el botón de puntos suspensivos (...) y, a continuación, elija **Configuración**.

2. En **Control de simultaneidad**, establezca **Control de simultaneidad** en **Activado**. 

3. Arrastre el control deslizante **Grado de paralelismo** al número `1`. 

<a name="asynchronous-patterns"></a>

### <a name="run-actions-synchronously"></a>Ejecutar acciones sincrónicamente

De forma predeterminada, todas las acciones basadas en HTTP siguen el patrón estándar de operación asincrónica. Este patrón especifica que cuando una acción basada en HTTP envía una solicitud al punto de conexión especificado, el servidor remoto devuelve una respuesta "202-Aceptado". Esta respuesta significa que el servidor aceptó la solicitud para su procesamiento. El motor de Logic Apps sigue comprobando la dirección URL especificada por el encabezado de ubicación de la respuesta hasta que se detiene el procesamiento, es decir, cuando se recibe cualquier respuesta que no sea la 202.

No obstante, las solicitudes tienen un límite de tiempo de espera, por lo que para las acciones de larga ejecución puede deshabilitar el comportamiento asincrónico agregando y estableciendo la propiedad `operationOptions` en `DisableAsyncPattern` en las entradas de la acción.
  
```json
"<some-long-running-action>": {
   "type": "Http",
   "inputs": { "<action-inputs>" },
   "operationOptions": "DisableAsyncPattern",
   "runAfter": {}
}
```

<a name="run-high-throughput-mode"></a>

### <a name="run-in-high-throughput-mode"></a>Ejecutar en modo de alto rendimiento

Para una única ejecución de aplicación lógica, el número de acciones que se ejecutan cada 5 minutos tiene un [límite predeterminado](../logic-apps/logic-apps-limits-and-config.md#throughput-limits). Para aumentar este límite al [máximo](../logic-apps/logic-apps-limits-and-config.md#throughput-limits) posible, establezca la propiedad `operationOptions` en `OptimizedForHighThroughput`. Esta opción pone a la aplicación lógica en modo de "alto rendimiento". 

> [!NOTE]
> El modo de alto rendimiento se encuentra en versión preliminar. También puede distribuir una carga de trabajo entre varias aplicaciones lógicas, según sea necesario.

```json
"<action-name>": {
   "type": "<action-type>",
   "inputs": { "<action-inputs>" },
   "operationOptions": "OptimizedForHighThroughput",
   "runAfter": {}
}
```

<a name="connector-authentication"></a>

## <a name="authenticate-http-triggers-and-actions"></a>Autenticación de acciones y desencadenadores HTTP

Los puntos de conexión HTTP admiten diferentes tipos de autenticación. Puede configurar la autenticación para estas acciones y desencadenadores HTTP:

* [HTTP](../connectors/connectors-native-http.md)
* [HTTP + Swagger](../connectors/connectors-native-http-swagger.md)
* [Webhook HTTP](../connectors/connectors-native-webhook.md)

Estos son los tipos de autenticación que se pueden configurar:

* [Autenticación básica](#basic-authentication)
* [Autenticación de certificados de clientes](#client-certificate-authentication)
* [Autenticación de OAuth de Azure Active Directory (Azure AD)](#azure-active-directory-oauth-authentication)

> [!IMPORTANT]
> Asegúrese de proteger la información confidencial que gestiona su definición de flujo de trabajo de aplicación lógica. Use parámetros protegidos y codifique los datos según sea necesario. Para obtener más información acerca de cómo usar y proteger los parámetros, consulte [Protección de las aplicaciones lógicas](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters).

<a name="basic-authentication"></a>

### <a name="basic-authentication"></a>Autenticación básica

Para utilizar la [autenticación básica](../active-directory-b2c/active-directory-b2c-custom-rest-api-netfw-secure-basic.md) mediante Azure Active Directory, la definición del desencadenador o acción puede incluir un objeto JSON `authentication`, que tiene las propiedades especificadas en la siguiente tabla. Para acceder al valor de parámetro durante el tiempo de ejecución, puede usar la expresión `@parameters('parameterName')`, proporcionada por el [lenguaje de definición de flujo de trabajo](https://aka.ms/logicappsdocs). 

| Propiedad | Obligatorio | Value | DESCRIPCIÓN | 
|----------|----------|-------|-------------| 
| **type** | Sí | "Básica" | Tipo de autenticación que se debe usar, que en este caso es "Básica" | 
| **username** | Sí | "@parameters('userNameParam')" | Nombre de usuario para autenticar el acceso al extremo del servicio de destino. |
| **password** | Sí | "@parameters('passwordParam')" | Contraseña para autenticar el acceso al extremo del servicio de destino. |
||||| 

En este ejemplo de definición de la acción HTTP, se usa la sección `authentication` para especificar la autenticación `Basic`. Para obtener más información acerca de cómo usar y proteger los parámetros, consulte [Protección de las aplicaciones lógicas](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters).

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "https://www.microsoft.com",
      "authentication": {
         "type": "Basic",
         "username": "@parameters('userNameParam')",
         "password": "@parameters('passwordParam')"
      }
  },
  "runAfter": {}
}
```

> [!IMPORTANT]
> Asegúrese de proteger la información confidencial que gestiona su definición de flujo de trabajo de aplicación lógica. Use parámetros protegidos y codifique los datos según sea necesario. Para obtener más información acerca de cómo proteger los parámetros, consulte [Protección de las aplicaciones lógicas](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters).

<a name="client-certificate-authentication"></a>

### <a name="client-certificate-authentication"></a>Autenticación de certificados de cliente

Para utilizar la [autenticación basada en certificados](../active-directory/authentication/active-directory-certificate-based-authentication-get-started.md) mediante Azure Active Directory, la definición del desencadenador o acción puede incluir un objeto JSON `authentication`, que tiene las propiedades especificadas en la siguiente tabla. Para acceder al valor de parámetro durante el tiempo de ejecución, puede usar la expresión `@parameters('parameterName')`, proporcionada por el [lenguaje de definición de flujo de trabajo](https://aka.ms/logicappsdocs). Para conocer los límites en el número de certificados de cliente que puede usar, consulte [Limits and configuration for Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md) (Límites y configuración de Azure Logic Apps).

| Propiedad | Obligatorio | Value | DESCRIPCIÓN |
|----------|----------|-------|-------------|
| **type** | Sí | "ClientCertificate" | Tipo de autenticación que se usará para los certificados de cliente de Capa de sockets seguros (SSL). Aunque se admiten los certificados autofirmados, no se admiten los certificados autofirmados para SSL. |
| **pfx** | Sí | "@parameters('pfxParam') | El contenido codificado en base 64 del archivo de intercambio de información personal (PFX) |
| **password** | Sí | "@parameters('passwordParam')" | La contraseña para acceder al archivo PFX |
||||| 

En este ejemplo de definición de la acción HTTP, se usa la sección `authentication` para especificar la autenticación `ClientCertificate`. Para obtener más información acerca de cómo usar y proteger los parámetros, consulte [Protección de las aplicaciones lógicas](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters).

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "https://www.microsoft.com",
      "authentication": {
         "type": "ClientCertificate",
         "pfx": "@parameters('pfxParam')",
         "password": "@parameters('passwordParam')"
      }
   },
   "runAfter": {}
}
```

> [!IMPORTANT]
> Asegúrese de proteger la información confidencial que gestiona su definición de flujo de trabajo de aplicación lógica. Use parámetros protegidos y codifique los datos según sea necesario. Para obtener más información acerca de cómo proteger los parámetros, consulte [Protección de las aplicaciones lógicas](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters).

<a name="azure-active-directory-oauth-authentication"></a>

### <a name="azure-active-directory-ad-oauth-authentication"></a>Autenticación de OAuth de Azure Active Directory (AD)

Para utilizar la [autenticación de OAuth de Azure AD](../active-directory/develop/authentication-scenarios.md), la definición del desencadenador o acción puede incluir un objeto JSON `authentication`, que tiene las propiedades especificadas en la siguiente tabla. Para acceder al valor de parámetro durante el tiempo de ejecución, puede usar la expresión `@parameters('parameterName')`, proporcionada por el [lenguaje de definición de flujo de trabajo](https://aka.ms/logicappsdocs).

| Propiedad | Obligatorio | Value | DESCRIPCIÓN |
|----------|----------|-------|-------------|
| **type** | Sí | `ActiveDirectoryOAuth` | El tipo de autenticación para usar, que es "ActiveDirectoryOAuth" para OAuth de Azure AD |
| **authority** | No | <*URL-for-authority-token-issuer*> | La dirección URL de la autoridad que proporciona el token de autenticación |
| **tenant** | Sí | <*tenant-ID*> | El identificador del inquilino de Azure AD |
| **audience** | Sí | <*resource-to-authorize*> | Recurso que quiere usar para la autorización; por ejemplo, `https://management.core.windows.net/` |
| **clientId** | Sí | <*client-ID*> | El identificador de cliente para la aplicación que solicita autorización |
| **credentialType** | Sí | "Certificado" o "secreto" | El tipo de credencial que el cliente usa para solicitar autorización. Esta propiedad y el valor no aparecen en la definición subyacente, pero determina los parámetros necesarios para el tipo de credencial. |
| **pfx** | Sí, solo para el tipo de credencial de "Certificado" | "@parameters('pfxParam') | El contenido codificado en base 64 del archivo de intercambio de información personal (PFX) |
| **password** | Sí, solo para el tipo de credencial de "Certificado" | "@parameters('passwordParam')" | La contraseña para acceder al archivo PFX |
| **secret** | Sí, solo para el tipo de credencial de "Secreto" | "@parameters('secretParam')" | Secreto de cliente para solicitar la autorización |
|||||

En este ejemplo de definición de la acción HTTP, se usa la sección `authentication` para especificar la autenticación `ActiveDirectoryOAuth` y la credencial de tipo "Secreto". Para obtener más información acerca de cómo usar y proteger los parámetros, consulte [Protección de las aplicaciones lógicas](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters).

```json
"HTTP": {
   "type": "Http",
   "inputs": {
      "method": "GET",
      "uri": "https://www.microsoft.com",
      "authentication": {
         "type": "ActiveDirectoryOAuth",
         "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db47",
         "audience": "https://management.core.windows.net/",
         "clientId": "34750e0b-72d1-4e4f-bbbe-664f6d04d411",
         "secret": "@parameters('secretParam')"
     }
   },
   "runAfter": {}
}
```

> [!IMPORTANT]
> Asegúrese de proteger la información confidencial que gestiona su definición de flujo de trabajo de aplicación lógica. Use parámetros protegidos y codifique los datos según sea necesario. Para obtener más información acerca de cómo proteger los parámetros, consulte [Protección de las aplicaciones lógicas](../logic-apps/logic-apps-securing-a-logic-app.md#secure-action-parameters).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre el [lenguaje de definición de flujo de trabajo](../logic-apps/logic-apps-workflow-definition-language.md).