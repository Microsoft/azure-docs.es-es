---
title: Espera y respuesta a eventos
description: Automatización de flujos de trabajo que se desencadenan, pausan y reanudan en función de eventos en un punto de conexión de servicio mediante el uso de Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: conceptual
ms.date: 08/27/2020
tags: connectors
ms.openlocfilehash: 7c6f3c4e3e4a2a29fe6a02c03043e3dfb81a2010
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89227906"
---
# <a name="create-and-run-automated-event-based-workflows-by-using-http-webhooks-in-azure-logic-apps"></a>Creación y ejecución de flujos de trabajo basados en eventos automatizados mediante webhooks HTTP en Azure Logic Apps

Con [Azure Logic Apps](../logic-apps/logic-apps-overview.md) y el conector de webhook de HTTP integrado, puede crear tareas automatizadas y flujos de trabajo que se suscriban a un punto de conexión de servicio, esperen eventos específicos y se ejecuten en función de esos eventos, en lugar de comprobar periódicamente o *sondear* ese punto de conexión.

Estos son algunos ejemplos de flujos de trabajo basados en webhook:

* Espere a que llegue un elemento de un [Centro de eventos de Azure](https://github.com/logicappsio/EventHubAPI) antes de desencadenar una ejecución de la aplicación lógica.
* Espere a una aprobación antes de continuar con un flujo de trabajo.

En este artículo se muestra cómo usar el desencadenador de webhook y la acción de webhook para que la aplicación lógica pueda recibir y responder a los eventos de un punto de conexión de servicio.

## <a name="how-do-webhooks-work"></a>¿Cómo funcionan los webhooks?

Un desencadenador de webhook de HTTP se basa en eventos, que no dependen de la comprobación o sondeo regular de nuevos elementos. Cuando se guarda una aplicación lógica que empieza con un desencadenador de webhook, o cuando se cambia la aplicación lógica de deshabilitada a habilitada, el desencadenador de webhook *se suscribe* a un punto de conexión de servicio específico mediante el registro de una *dirección URL de devolución de llamada* con ese punto de conexión. El desencadenador, luego, espera a que ese punto de conexión de servicio llame a la dirección URL, que empieza a ejecutar la aplicación lógica. Al igual que el [desencadenador de solicitud](connectors-native-reqres.md), la aplicación lógica se activa inmediatamente cuando se produce el evento especificado. El desencadenador de webhook *cancela la suscripción* del punto de conexión de servicio si quita el desencadenador y guarda la aplicación lógica, o al cambiar la aplicación lógica de habilitada a deshabilitada.

Una acción de webhook también está basada en eventos y *se suscribe* a un punto de conexión de servicio específico mediante el registro de una *dirección URL de devolución de llamada* con ese punto de conexión de servicio. La acción de webhook pausa el flujo de trabajo de la aplicación lógica y espera hasta que el punto de conexión de servicio llama a la dirección URL antes de que la aplicación lógica reanude la ejecución. La acción de webhook *cancela la suscripción* del punto de conexión de servicio en estos casos:

* Cuando finaliza correctamente la acción de webhook
* Si se cancela la ejecución de la aplicación lógica mientras espera una respuesta
* Antes de que la aplicación lógica agote el tiempo de espera

Por ejemplo, la acción [**Enviar correo de aprobación**](connectors-create-api-office365-outlook.md) del conector de Office 365 Outlook es un ejemplo de acción de webhook que sigue este patrón. Puede ampliar este patrón a cualquier servicio mediante el uso de la acción de webhook.

Para más información, consulte los temas siguientes:

* [Webhooks y suscripciones](../logic-apps/logic-apps-workflow-actions-triggers.md#webhooks-and-subscriptions)
* [Creación de API personalizadas compatibles con webhook](../logic-apps/logic-apps-create-api-app.md)

Para información sobre el cifrado, la seguridad y la autorización de llamadas entrantes para la aplicación lógica, como la [Seguridad de la capa de transporte (TLS)](https://en.wikipedia.org/wiki/Transport_Layer_Security), conocida anteriormente como Capa de sockets seguros (SSL) o la [Autenticación abierta de Azure Active Directory Azure (Azure AD OAuth)](../active-directory/develop/index.yml), consulte [Proteger el acceso y los datos: Acceso de llamadas entrantes para desencadenadores basados en solicitud](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests).

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* La dirección URL para una API o un punto de conexión ya implementado que admita el patrón de suscripción y cancelación de suscripción de webhook para [desencadenadores de webhook en aplicaciones lógicas](../logic-apps/logic-apps-create-api-app.md#webhook-triggers) o [acciones de webhook en aplicaciones lógicas](../logic-apps/logic-apps-create-api-app.md#webhook-actions), según corresponda.

* Conocimientos básicos acerca de [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md). Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md)

* La aplicación lógica en la que quiera esperar los eventos específicos en el punto de conexión de destino. Para comenzar con un desencadenador de webhook HTTP, [cree una aplicación lógica en blanco](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para usar la acción de webhook HTTP, inicie la aplicación lógica con el desencadenador que quiera. En este ejemplo se usa el desencadenador HTTP como primer paso.

## <a name="add-an-http-webhook-trigger"></a>Adición de un desencadenador de webhook HTTP

Este desencadenador integrado llama al punto de conexión de suscripción en el servicio de destino y registra una dirección URL de devolución de llamada en el servicio de destino. A continuación, la aplicación lógica espera a que el servicio de destino envíe una solicitud `HTTP POST` a la dirección URL de devolución de llamada. Cuando se produce este evento, el desencadenador se activa y pasa los datos de la solicitud junto al flujo de trabajo.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Abra la aplicación lógica en blanco en el diseñador de aplicación lógica.

1. En el cuadro de búsqueda del diseñador, escriba `http webhook` como filtro. En la lista de **desencadenadores**, seleccione el desencadenador **webhook HTTP**.

   ![Selección de un desencadenador de webhook HTTP](./media/connectors-native-webhook/select-http-webhook-trigger.png)

   En este ejemplo se cambia el nombre del desencadenador a `HTTP Webhook trigger` para que el paso tenga un nombre más descriptivo. Además, más adelante, en el ejemplo se agrega una acción de webhook HTTP, y ambos nombres deben ser únicos.

1. Proporcione los valores para los [parámetros del desencadenador de webhook HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-webhook-trigger) que quiera usar para las llamadas de suscripción y cancelación de suscripción.

   En este ejemplo, el desencadenador incluye los métodos, los identificadores URI y los cuerpos de mensaje que se van a usar al realizar las operaciones de suscripción y cancelación de suscripción.

   ![Introducción de parámetros del desencadenador de webhook HTTP](./media/connectors-native-webhook/http-webhook-trigger-parameters.png)

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Suscripción: método** | Sí | Método que se va a usar al suscribirse al punto de conexión de destino. |
   | **Suscripción: URI** | Sí | Dirección URL que se va a usar al suscribirse al punto de conexión de destino. |
   | **Suscripción: cuerpo** | No | Cuerpo del mensaje que se va a incluir en la solicitud de suscripción. En este ejemplo se incluye la dirección URL de devolución de llamada que identifica de forma única al suscriptor, que es la aplicación lógica, mediante el uso de la expresión `@listCallbackUrl()` para recuperar la dirección URL de devolución de llamada de la aplicación lógica. |
   | **Cancelación de suscripción: método** | No | Método que se va a usar al cancelar la suscripción del punto de conexión de destino. |
   | **Cancelación de suscripción: URI** | No | Dirección URL que se va a usar al cancelar la suscripción del punto de conexión de destino. |
   | **Cancelación de suscripción: cuerpo** | No | Cuerpo del mensaje opcional que se va a incluir en la solicitud de cancelación de suscripción. <p><p>**Nota**: Esta propiedad no admite el uso de la función `listCallbackUrl()`. Sin embargo, el desencadenador incluye y envía automáticamente los encabezados, `x-ms-client-tracking-id` y `x-ms-workflow-operation-name`, que el servicio de destino puede usar para identificar de forma única al suscriptor. |
   ||||

1. Para agregar otras propiedades del desencadenador, abra la lista **Agregar nuevo parámetro**.

   ![Adición de más propiedades del desencadenador](./media/connectors-native-webhook/http-webhook-trigger-add-properties.png)

   Por ejemplo, si necesita usar la autenticación, puede agregar las propiedades **Suscripción: autenticación** y **Cancelación de suscripción: autenticación**. Para obtener más información sobre los tipos de autenticación disponibles para webhook de HTTP, consulte [Incorporación de la autenticación en las llamadas salientes](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound).

1. Continúe creando el flujo de trabajo de la aplicación lógica con acciones que se ejecuten cuando se activa el desencadenador.

1. Cuando haya finalizado, recuerde guardar la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

   Al guardar la aplicación lógica, se llama al punto de conexión de suscripción del servicio de destino y se registra la dirección URL de devolución de llamada. A continuación, la aplicación lógica espera a que el servicio de destino envíe una solicitud `HTTP POST` a la dirección URL de devolución de llamada. Cuando se produce este evento, el desencadenador se activa y pasa los datos de la solicitud junto al flujo de trabajo. Si esta operación se completa correctamente, el desencadenador anula la suscripción del punto de conexión y la aplicación lógica continúa con el flujo de trabajo restante.

## <a name="add-an-http-webhook-action"></a>Adición de una acción de webhook HTTP

Esta acción integrada llama al punto de conexión de suscripción en el servicio de destino y registra una dirección URL de devolución de llamada en el servicio de destino. A continuación, la aplicación lógica se pone en pausa y espera a que el servicio de destino envíe una solicitud `HTTP POST` a la dirección URL de devolución de llamada. Cuando se produce este evento, la acción pasa los datos de la solicitud junto al flujo de trabajo. Si la operación se completa correctamente, la acción cancela la suscripción del punto de conexión y la aplicación lógica reanuda la ejecución del flujo de trabajo restante.

1. Inicie sesión en [Azure Portal](https://portal.azure.com). Abra la aplicación lógica en el Diseñador de aplicaciones lógicas.

   En este ejemplo se usa el desencadenador de webhook HTTP como primer paso.

1. En el paso en el que quiera agregar la acción de webhook HTTP, seleccione **Nuevo paso**.

   Para agregar una acción entre un paso y otro, mueva el puntero sobre la flecha entre ellos. Seleccione el signo más ( **+** ) que aparece y, luego, seleccione **Agregar una acción**.

1. En el cuadro de búsqueda del diseñador, escriba `http webhook` como filtro. En la lista **Acciones**, seleccione la acción **webhook HTTP**.

   ![Selección de una acción de webhook HTTP](./media/connectors-native-webhook/select-http-webhook-action.png)

   En este ejemplo se cambia el nombre de la acción a "acción de webhook HTTP" para que el paso tenga un nombre más descriptivo.

1. Proporcione los valores para los parámetros de la acción de webhook HTTP, que son parecidos a los [parámetros del desencadenador de webhook HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-webhook-trigger), que quiera usar para las llamadas de suscripción y cancelación de suscripción.

   En este ejemplo, la acción incluye los métodos, los identificadores URI y los cuerpos de mensaje que se van a usar al realizar las operaciones de suscripción y cancelación de suscripción.

   ![Introducción de los parámetros de acción de webhook HTTP](./media/connectors-native-webhook/http-webhook-action-parameters.png)

   | Propiedad | Obligatorio | Descripción |
   |----------|----------|-------------|
   | **Suscripción: método** | Sí | Método que se va a usar al suscribirse al punto de conexión de destino. |
   | **Suscripción: URI** | Sí | Dirección URL que se va a usar al suscribirse al punto de conexión de destino. |
   | **Suscripción: cuerpo** | No | Cuerpo del mensaje que se va a incluir en la solicitud de suscripción. En este ejemplo se incluye la dirección URL de devolución de llamada que identifica de forma única al suscriptor, que es la aplicación lógica, mediante el uso de la expresión `@listCallbackUrl()` para recuperar la dirección URL de devolución de llamada de la aplicación lógica. |
   | **Cancelación de suscripción: método** | No | Método que se va a usar al cancelar la suscripción del punto de conexión de destino. |
   | **Cancelación de suscripción: URI** | No | Dirección URL que se va a usar al cancelar la suscripción del punto de conexión de destino. |
   | **Cancelación de suscripción: cuerpo** | No | Cuerpo del mensaje opcional que se va a incluir en la solicitud de cancelación de suscripción. <p><p>**Nota**: Esta propiedad no admite el uso de la función `listCallbackUrl()`. Sin embargo, la acción incluye y envía automáticamente los encabezados, `x-ms-client-tracking-id` y `x-ms-workflow-operation-name`, que el servicio de destino puede usar para identificar de forma única al suscriptor. |
   ||||

1. Para agregar otras propiedades de la acción, abra la lista **Agregar nuevo parámetro**.

   ![Adición de más propiedades de la acción](./media/connectors-native-webhook/http-webhook-action-add-properties.png)

   Por ejemplo, si necesita usar la autenticación, puede agregar las propiedades **Suscripción: autenticación** y **Cancelación de suscripción: autenticación**. Para obtener más información sobre los tipos de autenticación disponibles para webhook de HTTP, consulte [Incorporación de la autenticación en las llamadas salientes](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound).

1. Cuando haya finalizado, recuerde guardar la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

   Ahora, cuando se ejecuta esta acción, la aplicación lógica llama al punto de conexión de suscripción del servicio de destino y registra la dirección URL de devolución de llamada. A continuación, la aplicación lógica pone en pausa el flujo de trabajo y espera a que el servicio de destino envíe una solicitud `HTTP POST` a la dirección URL de devolución de llamada. Cuando se produce este evento, la acción pasa los datos de la solicitud junto al flujo de trabajo. Si la operación se completa correctamente, la acción cancela la suscripción del punto de conexión y la aplicación lógica reanuda la ejecución del flujo de trabajo restante.

## <a name="trigger-and-action-outputs"></a>Salidas de los desencadenadores y las acciones

Aquí tiene más información acerca de las salidas de un desencadenador o acción de webhook HTTP, que devuelve esta información:

| Nombre de propiedad | Tipo | Descripción |
|---------------|------|-------------|
| headers | object | Encabezados de la solicitud |
| body | object | Objeto JSON | Objeto con el contenido del cuerpo de la solicitud |
| status code | int | Código de estado de la solicitud |
|||

| status code | Descripción |
|-------------|-------------|
| 200 | Aceptar |
| 202 | Accepted |
| 400 | Solicitud incorrecta |
| 401 | No autorizado |
| 403 | Prohibido |
| 404 | No encontrado |
| 500 | Error interno del servidor Error desconocido. |
|||

## <a name="connector-reference"></a>Referencia de conectores

Para obtener más información acerca de los parámetros del desencadenador y la acción, que son parecidos entre sí, consulte [Parámetros del webhook HTTP](../logic-apps/logic-apps-workflow-actions-triggers.md#http-webhook-trigger).

## <a name="next-steps"></a>Pasos siguientes

* [Proteger el acceso y datos: Acceso de llamadas entrantes para desencadenadores basados en solicitud](../logic-apps/logic-apps-securing-a-logic-app.md#secure-inbound-requests)
* [Conectores de Logic Apps](../connectors/apis-list.md)
