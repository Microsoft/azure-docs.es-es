---
title: Creación de API web y API REST para Azure Logic Apps
description: Creación de API web y API REST para llamar a las API, los servicios o los sistemas para integraciones de sistemas en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: conceptual
ms.date: 05/26/2017
ms.openlocfilehash: 0fc4fb91653f4a764540df0a7bc0cf0deee30fe6
ms.sourcegitcommit: 2ffa5bae1545c660d6f3b62f31c4efa69c1e957f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88080837"
---
# <a name="create-custom-apis-you-can-call-from-azure-logic-apps"></a>Creación de API personalizadas que se pueden llamar desde Azure Logic Apps

Aunque Azure Logic Apps ofrece [cientos de conectores integrados](../connectors/apis-list.md) que puede usar en flujos de trabajo de aplicación lógica, es posible que desee llamar a las API, los sistemas y los servicios que no estén disponibles como conectores. Puede crear sus propias API que proporcionan acciones y desencadenadores para usarlos en aplicaciones lógicas. Estas son otras razones por las que podría interesarle crear sus propias API que puede llamar desde flujos de trabajo de aplicación lógica:

* Ampliar los flujos de trabajo actuales de integración de datos e integración de sistemas.
* Ayudar a los clientes a utilizar su servicio para administrar tareas profesionales o personales.
* Expandir la cobertura, la detectabilidad y el uso de su servicio.

Básicamente, los conectores son API web que usan REST para las interfaces conectables, [formato de metadatos de Swagger](https://swagger.io/specification/) para documentación y JSON como formato de intercambio de datos. Dado que los conectores son API REST que se comunican a través de puntos de conexión HTTP, puede utilizar cualquier lenguaje para crear conectores, como, por ejemplo, .NET, Java, Python o Node.js. También puede hospedar las API en [Azure App Service](../app-service/overview.md), una oferta de plataforma como servicio (PaaS) que proporciona una de las formas más eficaces, sencillas y escalables para hospedar API. 

Para que las API personalizadas funcionen con las aplicaciones lógicas, su API puede proporcionar [*acciones*](./logic-apps-overview.md#logic-app-concepts) que realicen tareas específicas en flujos de trabajo de aplicaciones lógicas. La API también puede actuar como un [*desencadenador*](./logic-apps-overview.md#logic-app-concepts) que inicia un flujo de trabajo de aplicación lógica cuando un evento o datos nuevos cumplen una condición especificada. En este tema se describen los patrones comunes que puede seguir para crear acciones y desencadenadores en la API, en función del comportamiento que desee que ofrezca su API.

Puede hospedar las API en [Azure App Service](../app-service/overview.md), una oferta de plataforma como servicio (PaaS) que proporciona hospedaje API fácil y altamente escalable.

> [!TIP] 
> Aunque puede implementar las API como aplicaciones web, considere la posibilidad de implementarlas como aplicaciones de API, lo que puede facilitar la tarea de crear, hospedar y utilizar API en la nube y en el entorno local. No tiene que cambiar ningún código en las API; basta con implementar el código en una aplicación de API. Por ejemplo, aprenda a crear aplicaciones API creadas con estos lenguajes: 
> 
> * [ASP.NET](../app-service/quickstart-dotnetcore.md). 
> * [Java](../app-service/quickstart-java.md)
> * [Node.js](../app-service/quickstart-nodejs.md)
> * [PHP](../app-service/quickstart-php.md)
> * [Python](../app-service/quickstart-python.md)
> * [Ruby](../app-service/quickstart-ruby.md)
>
> Para ver ejemplos de aplicaciones de API creadas para aplicaciones lógicas, visite el [repositorio de GitHub de Azure Logic Apps](https://github.com/logicappsio) o el [blog](https://aka.ms/logicappsblog).

## <a name="how-do-custom-apis-differ-from-custom-connectors"></a>¿Cómo se diferencia las API personalizadas de los conectores personalizados?

Las API personalizadas y los [conectores personalizados](../logic-apps/custom-connector-overview.md) son API web que usan REST para las interfaces conectables, [formato de metadatos de Swagger](https://swagger.io/specification/) para documentación y JSON como formato de intercambio de datos. Y dado que estas API y conectores son API REST que se comunican a través de puntos de conexión HTTP, puede utilizar cualquier lenguaje para crear conectores y API personalizadas, como .NET, Java, Python o Node.js.

Las API personalizadas le permiten llamar a las API que no son conectores y proporcionar puntos de conexión que puede llamar con HTTP + Swagger, Azure API Management o App Services. Los conectores personalizados funcionan como las API personalizadas, pero también tienen estos atributos:

* Se registran como recursos de conector de Logic Apps en Azure.
* Aparecen con iconos junto con conectores administrados por Microsoft en el Diseñador de Logic Apps.
* Están disponibles solo para los autores y usuarios de aplicación lógica de los conectores que tienen el mismo inquilino de Azure Active Directory y la suscripción de Azure en la región donde se implementan las aplicaciones lógicas.

También puede designar conectores registrados para la certificación de Microsoft. Este proceso comprueba que los conectores registrados cumplen los criterios para uso público y hace que los conectores estén disponibles para los usuarios en Power Automate y Microsoft Power Apps.

Para obtener más información sobre los conectores personalizados, vea 

* [Introducción a los conectores personalizados](../logic-apps/custom-connector-overview.md)
* [Create custom connectors from Web APIs](/connectors/custom-connectors/create-web-api-connector) (Creación de conectores personalizados desde API web)
* [Register custom connectors in Azure Logic Apps](/connectors/custom-connectors/) (Registro de conectores personalizados en Azure Logic Apps)

## <a name="helpful-tools"></a>Herramientas útiles

Una API personalizada funciona mejor con aplicaciones lógicas cuando la API también cuenta con un [documento Swagger](https://swagger.io/specification/) que describe las operaciones y los parámetros de la API.
Muchas bibliotecas, como [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle), pueden generar el archivo Swagger automáticamente. Para anotar el archivo Swagger para nombres para mostrar, tipos de propiedades, etc., también puede usar [TRex](https://github.com/nihaue/TRex) para que el archivo Swagger funcione bien con aplicaciones lógicas.

<a name="actions"></a>

## <a name="action-patterns"></a>Patrones de acción

Para que las aplicaciones lógicas realicen tareas, la API personalizada debe proporcionar [*acciones*](./logic-apps-overview.md#logic-app-concepts). Cada operación de la API se asigna a una acción. Una acción básica es un controlador que acepta solicitudes HTTP y devuelve respuestas HTTP. Por ejemplo, una aplicación lógica envía una solicitud HTTP a su aplicación web o aplicación de API. A continuación, la aplicación devuelve una respuesta HTTP, junto con el contenido que puede procesar la aplicación lógica.

Para una acción estándar, puede escribir un método de solicitud HTTP en la API y describir ese método en un archivo Swagger. A continuación, puede llamar directamente a su API con una [acción HTTP](../connectors/connectors-native-http.md) o una acción [HTTP + Swagger](../connectors/connectors-native-http-swagger.md). De forma predeterminada, se deben devolver las respuestas en el [límite de tiempo de expiración de la solicitud](./logic-apps-limits-and-config.md). 

![Patrón de acción estándar](./media/logic-apps-create-api-app/standard-action.png)

<a name="pattern-overview"></a> Para que una aplicación lógica espere mientras su API finaliza tareas más prolongadas, la API puede seguir el [patrón asincrónico de sondeo](#async-pattern) o el [patrón asincrónico de webhook](#webhook-actions) descritos en este tema. Para ayudarle a visualizar los distintos comportamientos de estos patrones, imagínese el proceso de pedir a una pastelería una tarta personalizada. El patrón de sondeo imita el comportamiento de llamar a la pastelería cada 20 minutos para comprobar si la tarta está lista. El patrón de webhook imita el comportamiento de la pastelería cuando le solicita su número de teléfono para llamarle cuando la tarta esté lista.

Para ver ejemplos, visite el [repositorio de GitHub de Logic Apps](https://github.com/logicappsio). Además, obtenga más información sobre la [medición de uso para acciones](logic-apps-pricing.md).

<a name="async-pattern"></a>

### <a name="perform-long-running-tasks-with-the-polling-action-pattern"></a>Realización de tareas prolongadas con el patrón de acción de sondeo

Para que la API realice tareas que pueden prolongarse por encima del [límite de tiempo de expiración de solicitud](./logic-apps-limits-and-config.md), puede usar el patrón asincrónico de sondeo. Este patrón hace que la API realice trabajo en un subproceso independiente, pero mantiene una conexión activa con el motor de Logic Apps. De este modo, la aplicación lógica no agota el tiempo de espera ni continúa con el siguiente paso del flujo de trabajo antes de que la API finalice el trabajo.

El siguiente es el patrón general:

1. Asegúrese de que el motor sepa que la API aceptó la solicitud y empezó a trabajar.
2. Cuando el motor realice las solicitudes posteriores de estado de trabajo, indíquele que la API ha finalizado la tarea cuando eso ocurra.
3. Devuelva al motor datos pertinentes para que el flujo de trabajo de aplicación lógica pueda continuar.

<a name="bakery-polling-action"></a> Ahora aplique la analogía sobre la pastelería anterior al patrón de sondeo y suponga que llama a una pastelería y pide una tarta personalizada para entregar a domicilio. El proceso para preparar la tarta lleva tiempo y no desea esperar al teléfono mientras la pastelería la elabora. La pastelería confirma su pedido y le pide que llame cada 20 minutos para comprobar el estado de la tarta. Después de 20 minutos, llama a la pastelería y le indican que la tarta aún no está preparada, por lo que debe volver a llamar pasados otros 20 minutos. Este proceso de ida y vuelta continúa hasta que llama a la pastelería y le informan de que su pedido está listo y se le hace entrega de la tarta. 

Ahora vamos a volver a asignar este mismo patrón de sondeo. La pastelería representa la API personalizada, mientras que usted, el cliente que desea la tarta, representa el motor de Logic Apps. Cuando el motor llama a la API con una solicitud, la API confirma la solicitud y responde con el intervalo de tiempo en el que el motor puede comprobar el estado del trabajo. El motor continúa comprobando el estado del trabajo hasta que la API responde que el trabajo está terminado y devuelve datos a la aplicación lógica que, a continuación, prosigue con el flujo de trabajo. 

![Patrón de acción de sondeo](./media/logic-apps-create-api-app/custom-api-async-action-pattern.png)

Estos son los pasos específicos que debe seguir la API, descritos desde la perspectiva de la API:

1. Cuando la API recibe una solicitud HTTP para empezar a trabajar, devuelve inmediatamente una respuesta HTTP `202 ACCEPTED` con el encabezado `location` que se describe más adelante en este paso. Esta respuesta permite que el motor de Logic Apps sepa que la API ha recibido la solicitud, ha aceptado la carga útil de la solicitud (entrada de datos) y ahora la está procesando. 
   
   La respuesta `202 ACCEPTED` debe incluir estos encabezados:
   
   * *Obligatorio*: Un encabezado `location` que especifica la ruta de acceso absoluta a una dirección URL en la que el motor de Logic Apps puede comprobar el estado de trabajo de la API.

   * *Opcional*: Un encabezado `retry-after` que especifica el número de segundos que el motor debe esperar antes de comprobar la dirección URL `location` para el estado del trabajo. 

     De forma predeterminada, el motor realiza la comprobación cada 20 segundos. Para especificar un intervalo diferente, incluya el encabezado `retry-after` y el número de segundos hasta el siguiente sondeo.

2. Una vez transcurrido el tiempo especificado, el motor de Logic Apps sondea la dirección URL `location` para comprobar el estado de trabajo. La API debe realizar estas comprobaciones y devolver estas respuestas:
   
   * Si el trabajo se ha completado, devuelve una respuesta HTTP `200 OK`, junto con la carga útil de la respuesta (entrada para el paso siguiente).

   * Si el trabajo todavía se está procesando, devuelve otra respuesta `202 ACCEPTED`, pero con los mismos encabezados que la respuesta original.

Cuando la API sigue este patrón, no tiene que hacer nada en la definición del flujo de trabajo de aplicación lógica para continuar la comprobación del estado de trabajo. Cuando el motor obtiene una respuesta HTTP `202 ACCEPTED` y un encabezado `location` válido, el motor respeta el modelo asincrónico y comprueba el encabezado `location` hasta que la API devuelve una respuesta no 202.

> [!TIP]
> Para ver un ejemplo de modelo asincrónico, revise este [ejemplo de respuesta del controlador asincrónico en GitHub](https://github.com/logicappsio/LogicAppsAsyncResponseSample).

<a name="webhook-actions"></a>

### <a name="perform-long-running-tasks-with-the-webhook-action-pattern"></a>Realización de tareas prolongadas con el patrón de acción de webhook

Como alternativa, puede usar el patrón de webhook para tareas prolongadas y el procesamiento asincrónico. Este patrón hace que la aplicación lógica haga una pausa y espere una "devolución de llamada" de la API para finalizar el procesamiento antes de continuar el flujo de trabajo. Esta devolución de llamada es un método HTTP POST que envía un mensaje a una dirección URL cuando se produce un evento. 

<a name="bakery-webhook-action"></a> Ahora aplique la analogía anterior sobre la pastelería al patrón de webhook y suponga que llama a una pastelería y pide una tarta personalizada para entregar a domicilio. El proceso para preparar la tarta lleva tiempo y no desea esperar al teléfono mientras la pastelería la elabora. La pastelería confirma su pedido, pero esta vez les proporciona su número de teléfono para que le llamen cuando la tarta esté terminada. Esta vez, la pastelería le informa cuando el pedido está listo y le hace entrega de la tarta.

Al volver a asignar este patrón de webhook, la pastelería representa la API personalizada, mientras que usted, el cliente que desea la tarta, representa el motor de Logic Apps. El motor llama a la API con una solicitud e incluye una dirección URL de "devolución de llamada".
Cuando el trabajo está listo, la API utiliza la dirección URL para notificar al motor y devuelve datos a la aplicación lógica que, a continuación, prosigue con el flujo de trabajo. 

Para este patrón, configure dos puntos de conexión en el controlador: `subscribe` y `unsubscribe`

*  Punto de conexión `subscribe`: Cuando la ejecución alcanza la acción de la API en el flujo de trabajo, el motor de Logic Apps llama al punto de conexión `subscribe`. Este paso hará que la aplicación lógica cree una dirección URL de devolución de llamada que la API almacenará y, a continuación, que espere la devolución de llamada de la API cuando se complete el trabajo. Después, la API realiza la devolución de llamada con un método HTTP POST a la dirección URL y transfiere todo el contenido y los encabezados devueltos como entrada a la aplicación lógica.

* Punto de conexión `unsubscribe`: Si se cancela la ejecución de la aplicación lógica, el motor de Logic Apps llama al punto de conexión `unsubscribe`. A continuación, la API puede anular el registro de la dirección URL de devolución de llamada y detener todos los procesos según sea necesario.

![Patrón de acción de webhook](./media/logic-apps-create-api-app/custom-api-webhook-action-pattern.png)

Actualmente, el Diseñador de aplicaciones lógicas no permite detectar puntos de conexión de webhook a través de Swagger. Por lo tanto, para este patrón, tiene que agregar una [acción de **webhook**](../connectors/connectors-native-webhook.md) y especificar la dirección URL, los encabezados y el cuerpo de la solicitud. Consulte también [Acciones y desencadenadores de flujo de trabajo](logic-apps-workflow-actions-triggers.md#apiconnection-webhook-action). Para ver un ejemplo de patrón de webhook, consulte este [ejemplo de desencadenador de webhook en GitHub](https://github.com/logicappsio/LogicAppTriggersExample/blob/master/LogicAppTriggers/Controllers/WebhookTriggerController.cs).

Estas son algunas sugerencias y notas:

* Para pasar la dirección URL de devolución de llamada, puede usar la función de flujo de trabajo `@listCallbackUrl()` en cualquiera de los campos anteriores según sea necesario.

* Si es el propietario de la aplicación lógica y del servicio suscrito, no tiene que llamar al punto de conexión `unsubscribe` después de llamar a la dirección URL de devolución de llamada. Si no, el tiempo de ejecución de Logic Apps ha de llamar al punto de conexión `unsubscribe` para indicar que no se esperan más llamadas y permitir la limpieza de recursos en el lado servidor.

<a name="triggers"></a>

## <a name="trigger-patterns"></a>Patrones de desencadenador

La API personalizada puede actuar como un [*desencadenador*](./logic-apps-overview.md#logic-app-concepts) que inicia una aplicación lógica cuando un evento o datos nuevos cumplen una condición especificada. Este desencadenador puede realizar comprobaciones periódicamente o esperar y escuchar nuevos datos o eventos en su punto de conexión de servicio. Si un evento o datos nuevos cumplen la condición especificada, el desencadenador se activa e inicia la aplicación lógica, que está escuchando a ese desencadenador. Para iniciar aplicaciones lógicas de esta manera, la API puede seguir el patrón de [*desencadenador de sondeo*](#polling-triggers) o [*desencadenador de webhook*](#webhook-triggers). Estos patrones son similares a sus [acciones de sondeo](#async-pattern) y [acciones de webhook](#webhook-actions) homólogas. Obtenga también más información sobre la [medición de uso para desencadenadores](logic-apps-pricing.md).

<a name="polling-triggers"></a>

### <a name="check-for-new-data-or-events-regularly-with-the-polling-trigger-pattern"></a>Comprobación periódica de nuevos datos o eventos con el patrón de desencadenador de sondeo

Un *desencadenador de sondeo* actúa de manera muy similar a la [acción de sondeo](#async-pattern) descrita anteriormente en este tema. El motor de Logic Apps llama periódicamente y comprueba el punto de conexión del desencadenador para buscar nuevos datos o eventos. Si el motor encuentra nuevos datos o un evento que cumplan la condición especificada, el desencadenador se activa. A continuación, el motor crea una instancia de aplicación lógica que procesa los datos como entrada. 

![Patrón de desencadenador de sondeo](./media/logic-apps-create-api-app/custom-api-polling-trigger-pattern.png)

> [!NOTE]
> Cada solicitud de sondeo cuenta como una ejecución de acción, incluso cuando no se crea ninguna instancia de aplicación lógica. Para evitar el procesamiento de los mismos datos varias veces, el desencadenador debe limpiar los datos ya leídos y transmitidos a la aplicación lógica.

Estos son los pasos específicos que debe seguir el desencadenador de sondeo, descritos desde la perspectiva de la API:

| ¿Se han encontrado nuevos datos o eventos?  | Respuesta de la API | 
| ------------------------- | ------------ |
| Encontrado | Devuelve un estado de HTTP `200 OK`con la carga útil de la respuesta (entrada para el paso siguiente). <br/>Esta respuesta crea una instancia de aplicación lógica e inicia el flujo de trabajo. | 
| No encontrado | Devuelve un estado de HTTP `202 ACCEPTED` con un encabezado `location` y un encabezado `retry-after`. <br/>Para los desencadenadores, el encabezado `location` también debe contener un parámetro de consulta `triggerState`, que normalmente es una "marca de tiempo". La API puede usar este identificador para llevar un seguimiento de la última vez que se desencadenó la aplicación lógica. | 
||| 

Por ejemplo, para comprobar periódicamente el servicio para buscar nuevos archivos, podría crear un desencadenador de sondeo que tenga estos comportamientos:

| ¿La solicitud incluye `triggerState`? | Respuesta de la API | 
| -------------------------------- | -------------| 
| No | Devuelve un estado de HTTP `202 ACCEPTED` más un encabezado `location` con `triggerState` establecido en la hora actual y el intervalo `retry-after` en 15 segundos. | 
| Sí | Compruebe si en el servicio se han agregado archivos después de la fecha `DateTime` para `triggerState`. | 
||| 

| Número de archivos encontrados | Respuesta de la API | 
| --------------------- | -------------| 
| Un solo archivo | Devuelve un estado de HTTP `200 OK` y la carga útil del contenido, actualiza `triggerState` a la fecha `DateTime` para el archivo devuelto y establece un intervalo `retry-after` de 15 segundos. | 
| Varios archivos | Devuelve un archivo cada vez y un estado de HTTP `200 OK`, actualiza `triggerState` y establece el intervalo `retry-after` en 0 segundos. </br>Estos pasos permiten que el motor sepa que hay más datos disponibles y que debe solicitar inmediatamente los datos de la dirección URL del encabezado `location`. | 
| Ningún archivo | Devuelve un estado de HTTP `202 ACCEPTED`, no cambia `triggerState` y establece el intervalo `retry-after` en 15 segundos. | 
||| 

> [!TIP]
> Para ver un ejemplo de patrón de desencadenador de sondeo, consulte este [ejemplo de controlador de desencadenador de sondeo en GitHub](https://github.com/logicappsio/LogicAppTriggersExample/blob/master/LogicAppTriggers/Controllers/PollTriggerController.cs).

<a name="webhook-triggers"></a>

### <a name="wait-and-listen-for-new-data-or-events-with-the-webhook-trigger-pattern"></a>Espera y escucha de nuevos datos o eventos con el patrón de desencadenador de webhook

Un desencadenador de webhook es un *desencadenador de push* que espera y escucha nuevos datos o eventos en su punto de conexión de servicio. Si un evento o datos nuevos cumplen la condición especificada, el desencadenador se activa y crea una instancia de aplicación lógica que, a continuación, procesa los datos como entrada.
Los desencadenadores de webhook actúan de manera muy similar a las [acciones de webhook](#webhook-actions) anteriormente descritas en este tema y se configuran con los puntos de conexión `subscribe` y `unsubscribe`. 

* Punto de conexión `subscribe`: Al agregar y guardar un desencadenador de webhook en la aplicación lógica, el motor de Logic Apps llama al punto de conexión `subscribe`. Este paso hará que la aplicación lógica cree una dirección URL de devolución de llamada que la API almacena. Cuando un evento o datos nuevos cumplen la condición especificada, la API devuelve la llamada a la dirección URL con un método HTTP POST. La carga útil del contenido y los encabezados se transmiten como entrada a la aplicación lógica.

* Punto de conexión `unsubscribe`: Si se elimina la aplicación lógica completa o el desencadenador de webhook, el motor de Logic Apps llama al punto de conexión `unsubscribe`. A continuación, la API puede anular el registro de la dirección URL de devolución de llamada y detener todos los procesos según sea necesario.

![Patrón de desencadenador de webhook](./media/logic-apps-create-api-app/custom-api-webhook-trigger-pattern.png)

Actualmente, el Diseñador de aplicaciones lógicas no permite detectar puntos de conexión de webhook a través de Swagger. Por lo tanto, para este patrón, tiene que agregar un [desencadenador de **webhook**](../connectors/connectors-native-webhook.md) y especificar la dirección URL, los encabezados y el cuerpo de la solicitud. Consulte también el [desencadenador HTTPWebhook](logic-apps-workflow-actions-triggers.md#httpwebhook-trigger). Para ver un ejemplo de patrón de webhook, consulte este [ejemplo de controlador de desencadenador de webhook en GitHub](https://github.com/logicappsio/LogicAppTriggersExample/blob/master/LogicAppTriggers/Controllers/WebhookTriggerController.cs).

Estas son algunas sugerencias y notas:

* Para pasar la dirección URL de devolución de llamada, puede usar la función de flujo de trabajo `@listCallbackUrl()` en cualquiera de los campos anteriores según sea necesario.

* Para evitar el procesamiento de los mismos datos varias veces, el desencadenador debe limpiar los datos ya leídos y transmitidos a la aplicación lógica.

* Si es el propietario de la aplicación lógica y del servicio suscrito, no tiene que llamar al punto de conexión `unsubscribe` después de llamar a la dirección URL de devolución de llamada. Si no, el tiempo de ejecución de Logic Apps ha de llamar al punto de conexión `unsubscribe` para indicar que no se esperan más llamadas y permitir la limpieza de recursos en el lado servidor.

## <a name="improve-security-for-calls-to-your-apis-from-logic-apps"></a>Mejora de la seguridad de las llamadas a las API desde aplicaciones lógicas

Después de crear las API personalizadas, configure la autenticación para las API de forma que pueda llamarlas de forma segura desde las aplicaciones lógicas. Obtenga información sobre [cómo mejorar la seguridad de las llamadas a API personalizadas desde aplicaciones lógicas](../logic-apps/logic-apps-custom-api-authentication.md).

## <a name="deploy-and-call-your-apis"></a>Implementación de sus API y llamada a las mismas

Después de configurar la autenticación, configure la implementación de las API. Obtenga información sobre [cómo implementar API personalizadas y llamar a las mismas desde aplicaciones lógicas](../logic-apps/logic-apps-custom-api-host-deploy-call.md).

## <a name="publish-custom-apis-to-azure"></a>Publicación de API personalizadas en Azure

Para que las API personalizadas estén disponibles para otros usuarios de Logic Apps en Azure, debe agregar seguridad y registrarlas como conectores de aplicación lógica. Para más información consulte [Custom connectors overview](../logic-apps/custom-connector-overview.md) (Información general sobre conectores personalizados) 

Para que las API personalizadas estén disponibles para todos los usuarios de Logic Apps, Power Automate y Microsoft Power Apps, debe agregar seguridad, registrar las API como conectores de aplicación lógica y designar los conectores para el [programa Microsoft Azure Certified](https://azure.microsoft.com/marketplace/programs/certified/logic-apps/). 

## <a name="get-support"></a>Obtención de soporte técnico

* Para obtener ayuda específica sobre las API personalizadas, póngase en contacto con [customapishelp@microsoft.com](mailto:customapishelp@microsoft.com).

* Si tiene alguna duda, visite la [página de preguntas y respuestas de Microsoft sobre Azure Logic Apps](/answers/topics/azure-logic-apps.html).

* Para ayudar a mejorar Logic Apps, vote o envíe ideas en el [sitio de comentarios de usuario de Logic Apps](https://aka.ms/logicapps-wish). 

## <a name="next-steps"></a>Pasos siguientes

* [Control de errores y excepciones](../logic-apps/logic-apps-exception-handling.md)
* [Llamada, desencadenamiento o anidamiento de aplicaciones lógicas con puntos de conexión HTTP](../logic-apps/logic-apps-http-endpoint.md)
* [Medición de uso para acciones y desencadenadores](../logic-apps/logic-apps-pricing.md)
