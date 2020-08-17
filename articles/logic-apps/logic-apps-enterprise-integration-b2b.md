---
title: Intercambio de mensajes para escenarios de integración empresarial B2B
description: Reciba y envíe mensajes B2B entre entidades con Azure Logic Apps mediante Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 02/10/2020
ms.openlocfilehash: 2fd96777a5fe74d44aeed5f6de82e54b2663a024
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090389"
---
# <a name="receive-and-confirm--b2b-as2-messages-by-using-azure-logic-apps-and-enterprise-integration-pack"></a>Recepción y confirmación de mensajes B2B AS2 con Azure Logic Apps y Enterprise Integration Pack

Si tiene una cuenta de integración que define entidades y contratos, puede crear un flujo de trabajo de negocio a negocio (B2B) automatizado que intercambie mensajes entre los socios comerciales mediante [Azure Logic Apps](../logic-apps/logic-apps-overview.md) con [Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md). Azure Logic Apps funciona con conectores que admiten los protocolos AS2, X12, EDIFACT y RosettaNet estándar del sector. También puede combinar estos conectores con otros [conectores disponibles en Logic Apps](../connectors/apis-list.md), por ejemplo, Salesforce y Office 365 Outlook.

En este artículo se muestra cómo crear una aplicación lógica que recibe una solicitud HTTP mediante el uso de un desencadenador de solicitud, descodifica el contenido del mensaje mediante las acciones AS2 y X12 y, a continuación, devuelve una respuesta mediante la acción de respuesta.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si aún no tiene suscripción, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* Una aplicación lógica en blanco para que pueda crear el flujo de trabajo de B2B mediante el desencadenador [Solicitud](../connectors/connectors-native-reqres.md) seguido de estas acciones:

  * [Descodificación con AS2](../logic-apps/logic-apps-enterprise-integration-as2.md)

  * [Condición](../logic-apps/logic-apps-control-flow-conditional-statement.md), que envía una [Respuesta](../connectors/connectors-native-reqres.md) en función de si la acción de descodificación de AS2 se realiza correctamente o no.

  * [Descodificar mensaje X12](../logic-apps/logic-apps-enterprise-integration-x12.md) 

  Si no está familiarizado con las aplicaciones lógicas, consulte [¿Qué es Azure Logic Apps?](../logic-apps/logic-apps-overview.md) e [Inicio rápido: Creación de la primera aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

* Una [cuenta de integración](./logic-apps-enterprise-integration-create-integration-account.md) asociada a la suscripción de Azure y vinculada a la aplicación lógica. Tanto la cuenta de integración como la aplicación de lógica deben existir en la misma ubicación o región de Azure.

* Al menos dos [entidades](../logic-apps/logic-apps-enterprise-integration-partners.md) que ya haya definido en su cuenta de integración, junto con los[contratos AS2 y X12](logic-apps-enterprise-integration-agreements.md) para esas entidades.

## <a name="add-request-trigger"></a>Agregar un desencadenador de solicitud

En este ejemplo se usa el diseñador de aplicaciones lógicas en Azure Portal, pero puede seguir pasos similares para el diseñador de aplicaciones lógicas en Visual Studio.

1. En [Azure Portal](https://portal.azure.com), abra la aplicación lógica en blanco en el Diseñador de aplicación lógica.

1. En el cuadro de búsqueda, escriba `when a http request` y seleccione **Cuando se recibe una solicitud HTTP** para usar la opción como desencadenador.

   ![Selección del desencadenador Solicitud para iniciar el flujo de trabajo de la aplicación lógica](./media/logic-apps-enterprise-integration-b2b/select-http-request-trigger.png)

1. Deje vacío el cuadro**Esquema JSON del cuerpo de la solicitud**, ya que el mensaje X12 es un archivo plano.

   ![Deje "Esquema JSON del cuerpo de la solicitud" vacío](./media/logic-apps-enterprise-integration-b2b/receive-trigger-message-body-json-schema.png)

1. Cuando esté listo, seleccione **Guardar** en la barra de herramientas del diseñador.

   Este paso genera la **dirección URL de HTTP POST** que se usará para enviar la solicitud que desencadena la aplicación lógica. Para copiar esta URL, seleccione el icono de copia que se encuentra junto a la URL.

   ![Dirección URL generada para el desencadenador Solicitud para recibir llamadas](./media/logic-apps-enterprise-integration-b2b/generated-url-request-trigger.png)

## <a name="add-as2-decode-action"></a>Incorporación de la acción de descodificación AS2

Ahora, agregue las acciones B2B que desee usar. En este ejemplo, se usan las acciones AS2 y X12.

1. En el desencadenador, seleccione **Nuevo paso**. Para ocultar los detalles del desencadenador, haga clic en la barra de título del desencadenador.

   ![Adición de otro paso al flujo de trabajo de la aplicación lógica](./media/logic-apps-enterprise-integration-b2b/add-new-action-under-trigger.png)

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `as2 decode` y seleccione **Descodificación AS2 (v2)** .

   ![Busque y seleccione "Descodificación AS2 (v2)"](./media/logic-apps-enterprise-integration-b2b/add-as2-decode-action.png)

1. Para la propiedad **Mensaje que se descodificará**, escriba la entrada que desea que descodifique la acción AS2, que es el contenido `body` que recibe el desencadenador de la solicitud HTTP. Hay varias maneras de especificar este contenido como entrada, ya sea desde la lista de contenido dinámico o como expresión:

   * Para seleccionar de una lista que muestre las salidas de desencadenador disponibles, haga clic dentro del cuadro **Mensaje que se descodificará**. Cuando aparezca la lista de contenido dinámico, en **Cuando se recibe una solicitud HTTP**, seleccione el valor de propiedad **Cuerpo**, por ejemplo:

     ![Seleccione el valor "Cuerpo" del desencadenador](./media/logic-apps-enterprise-integration-b2b/select-body-content-from-trigger.png)

   * Para especificar una expresión que haga referencia a la salida `body` del desencadenador, haga clic dentro del cuadro **Mensaje que se descodificará**. Después de que aparezca la lista de contenido dinámico, seleccione **Expresión**. En el editor de expresiones, escriba esta expresión y seleccione **Aceptar**:

     `triggerOutputs()['body']`

     O bien, en el cuadro **Mensaje que se descodificará**, escriba directamente esta expresión:

     `@triggerBody()`

     La expresión se resuelve en el token **Cuerpo**.

     ![Resultado de cuerpo resuelto desde el desencadenador](./media/logic-apps-enterprise-integration-b2b/resolved-trigger-outputs-body-expression.png)

1. Para la propiedad **Encabezados de mensaje**, escriba los encabezados necesarios para la acción AS2, que se describen en el contenido `headers` que recibe el desencadenador de la solicitud HTTP.

   Para especificar una expresión que haga referencia a la salida `headers` del desencadenador, haga clic dentro del cuadro **Encabezados de mensaje**. Después de que aparezca la lista de contenido dinámico, seleccione **Expresión**. En el editor de expresiones, escriba esta expresión y seleccione **Aceptar**:

   `triggerOutputs()['Headers']`

   Para que esta expresión se resuelva como este token, cambie entre la vista del diseñador y la vista de código, por ejemplo:

   ![Resultado de los encabezados resueltos desde el desencadenador](./media/logic-apps-enterprise-integration-b2b/resolved-trigger-outputs-headers-expression.png)

## <a name="add-response-action-for-message-receipt-notification"></a>Adición de la acción Respuesta para la notificación de recepción de mensajes

Para notificar a la entidad que el mensaje se ha recibido, puede volver a enviar una respuesta que contenga una notificación de disposición del mensaje (MDN) AS2 en una acción **Respuesta**. Al agregar esta acción inmediatamente después de la acción **Descodificación AS2**, en caso de que se produzca un error en la acción, la aplicación lógica no continúa el procesamiento.

1. En la acción **Descodificación AS2**, seleccione **Nuevo paso**.

1. En **Elegir una acción**, en el cuadro de búsqueda, seleccione **Integrado**. En el cuadro de búsqueda, escriba `condition`. En la lista **Acciones**, seleccione **Condición**.

   ![Agregar la acción "Condición"](./media/logic-apps-enterprise-integration-b2b/add-condition-action.png)

   Ahora aparece la forma de la condición, incluidas las rutas de acceso de si la condición se cumple o no.

   ![Forma de la condición con rutas de acceso de decisión](./media/logic-apps-enterprise-integration-b2b/added-condition-action.png)

1. Ahora especifique la condición que se va a evaluar. En el cuadro **Elegir un valor**, escriba esta expresión:

   `@body('AS2_Decode')?['AS2Message']?['MdnExpected']`

   En el cuadro central, asegúrese de que la operación de comparación esté establecida en `is equal to`. En el cuadro de la derecha, escriba el valor `Expected`. Para que esta expresión se resuelva como este token, cambie entre la vista del diseñador y la vista de código.

   ![Forma de la condición con rutas de acceso de decisión](./media/logic-apps-enterprise-integration-b2b/expression-for-evaluating-condition.png)

1. Ahora, especifique las respuestas para devolver si la acción **Descodificación AS2** se realiza correctamente o no.

   1. En el caso de que la acción **Descodificación AS2** sea correcta, en la forma **If true**, seleccione **Agregar una acción**. En **Elegir una acción**, en el cuadro de búsqueda, escriba `response` y seleccione **Respuesta**.

      ![Buscar y seleccionar la acción "Respuesta"](./media/logic-apps-enterprise-integration-b2b/select-http-response-action.png)

   1. Para acceder a la notificación de disposición del mensaje de AS2 desde la salida de la acción **Descodificación AS2**, especifique estas expresiones:

      * En la propiedad **Encabezados** de la acción **Respuesta**, escriba esta expresión:

        `@body('AS2_Decode')?['OutgoingMdn']?['OutboundHeaders']`

      * En la propiedad **Cuerpo** de la acción **Respuesta**, escriba esta expresión:

        `@body('AS2_Decode')?['OutgoingMdn']?['Content']`

   1. Para que esta expresión se resuelva como token, cambie entre la vista del diseñador y la vista de código:

      ![Expresión resuelta para acceder a la MDN de AS2](./media/logic-apps-enterprise-integration-b2b/response-action-success-resolved-expression.png)

   1. En el caso de que la acción **Descodificación AS2** genere un error, en la forma **If false**, seleccione **Agregar una acción**. En **Elegir una acción**, en el cuadro de búsqueda, escriba `response` y seleccione **Respuesta**. Configure la acción **Respuesta** para devolver el estado y el error que desea.

1. Guarde la aplicación lógica.

## <a name="add-decode-x12-message-action"></a>Agregue la acción Descodificar mensaje X12.

1. Ahora agregue la acción **Descodificar mensaje X12**. En la acción **Respuesta**, seleccione **Agregar una acción**.

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `x12 decode` y seleccione **Descodificar mensaje X12**.

   ![Busque y seleccione la acción "Descodificar mensaje X12"](./media/logic-apps-enterprise-integration-b2b/add-x12-decode-action.png)

1. Si la acción X12 solicita información de conexión, proporcione el nombre de la conexión, seleccione la cuenta de integración que desee usar y, a continuación, seleccione **Crear**.

   ![Crear la conexión de X12 a la cuenta de integración](./media/logic-apps-enterprise-integration-b2b/create-x12-integration-account-connection.png)

1. Ahora, especifique la entrada para la acción X12. En este ejemplo se usa la salida de la acción AS2, que es el contenido del mensaje, pero tenga en cuenta que este contenido está en formato de objeto JSON y está codificado en base64. Por lo tanto, tiene que convertir este contenido en una cadena.

   En el cuadro **Mensaje de archivo plano X12 que se va a descodificar**, escriba esta expresión para convertir la salida AS2:

   `@base64ToString(body('AS2_Decode')?['AS2Message']?['Content'])`

    Para que esta expresión se resuelva como este token, cambie entre la vista del diseñador y la vista de código.

    ![Convertir el contenido codificado en base64 en una cadena](./media/logic-apps-enterprise-integration-b2b/x12-decode-message-content.png)

1. Guarde la aplicación lógica.

   Si necesita pasos adicionales para esta aplicación lógica, por ejemplo, para descodificar el contenido del mensaje y generar ese contenido en formato de objeto JSON, continúe con la compilación de la aplicación lógica.

Con esto, ya ha terminado de configurar la aplicación lógica B2B. En una aplicación real, puede almacenar los datos X12 descodificados en un almacén de datos o una aplicación de línea de negocio (LOB). Por ejemplo, consulte estos artículos:

* [Conexión a sistemas SAP desde Azure Logic Apps](../logic-apps/logic-apps-using-sap-connector.md)
* [Supervisión, creación y administración de archivos SFTP mediante SSH y Azure Logic Apps](../connectors/connectors-sftp-ssh.md)

Puede agregar más acciones o [escribir API personalizadas](../logic-apps/logic-apps-create-api-app.md) que se conecten a sus propias aplicaciones LOB, así como emplear estas API en su aplicación lógica.

## <a name="next-steps"></a>Pasos siguientes

* [Recepción de llamadas HTTPS entrantes y respuesta a ellas](../connectors/connectors-native-reqres.md)
* [Intercambio de mensajes AS2 para la integración empresarial B2B](../logic-apps/logic-apps-enterprise-integration-as2.md)
* [Intercambio de mensajes X12 para la integración empresarial B2B](../logic-apps/logic-apps-enterprise-integration-x12.md)
* Obtenga más información acerca de [Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md)
