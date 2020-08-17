---
title: Creación de flujos de trabajo automatizados basados en aprobación
description: 'Tutorial: Creación de un flujo de trabajo automatizado basado en aprobación que procesa las suscripciones de listas de distribución de correo mediante Azure Logic Apps'
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: tutorial
ms.custom: mvc
ms.date: 09/20/2019
ms.openlocfilehash: d9d2f29ffc34c203e5f3b3ebf094e73fb9cdfb75
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87132405"
---
# <a name="tutorial-create-automated-approval-based-workflows-by-using-azure-logic-apps"></a>Tutorial: Creación de flujos de trabajo automatizados basados en aprobación mediante Azure Logic Apps

En este tutorial se muestra cómo compilar una [aplicación lógica](../logic-apps/logic-apps-overview.md) que automatiza un flujo de trabajo basado en aprobación. En concreto, esta aplicación lógica procesa las solicitudes de suscripción para obtener una lista de distribución de correo que administra el servicio [MailChimp](https://mailchimp.com/). Esta aplicación lógica supervisa estas solicitudes en una cuenta de correo electrónico, las envía para su aprobación y agrega los miembros aprobados a la lista de distribución de correo.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Crear una aplicación lógica en blanco.
> * Agregue un desencadenador que supervise las solicitudes de suscripción en los correos electrónicos.
> * Agregue una acción que envíe correos electrónicos para aprobar o rechazar estas solicitudes.
> * Agregue una condición que compruebe la respuesta de aprobación.
> * Agregue una acción que permita incorporar los miembros aprobados a la lista de distribución de correo.
> * Agregue una condición que compruebe si estos miembros se han unido correctamente a la lista.
> * Agregue una condición que envíe correos electrónicos que confirmen si estos miembros se han unido correctamente a la lista.

Cuando haya terminado, la aplicación lógica se parecerá a este flujo de trabajo, en un alto nivel:

![Información general de la aplicación lógica de alto nivel finalizada](./media/tutorial-process-mailing-list-subscriptions-workflow/tutorial-high-level-overview.png)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene ninguna suscripción, [suscríbase a una cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

* Una cuenta de MailChimp que contenga una lista denominada "test-members-ML" en la que la aplicación lógica pueda agregar direcciones de correo electrónico para los miembros aprobados. Si no tiene una cuenta, [regístrese para obtener una cuenta gratuita](https://login.mailchimp.com/signup/) y aprenda a [crear una lista de MailChimp](https://us17.admin.mailchimp.com/lists/#).

* Una cuenta de correo electrónico de Office 365 Outlook o Outlook.com, que admita flujos de trabajo de aprobación. Este artículo utiliza Office 365 Outlook. Si utiliza una cuenta de correo electrónico diferente, los pasos generales siguen siendo los mismos pero la interfaz de usuario podría ser ligeramente distinta.

## <a name="create-your-logic-app"></a>Creación de una aplicación lógica

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure.

1. En el menú principal de Azure, seleccione **Crear un recurso** > **Integración** > **Aplicación lógica**.

   ![Creación de un recurso de aplicación lógica](./media/tutorial-process-mailing-list-subscriptions-workflow/create-new-logic-app-resource.png)

1. En **Crear aplicación lógica**, proporcione esta información sobre la aplicación lógica tal como se muestra y se describe a continuación. Seleccione **Crear** cuando haya terminado.

   ![Proporción de información acerca de la aplicación lógica](./media/tutorial-process-mailing-list-subscriptions-workflow/create-logic-app-settings.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **Nombre** | LA-MailingList | El nombre de la aplicación lógica, que solo puede contener letras, números, guiones (`-`), caracteres de subrayado (`_`), paréntesis (`(`, `)`) y puntos (`.`). En este ejemplo se usa "LA-MailingList". |
   | **Suscripción** | <*nombre-de-su-suscripción-a-Azure*> | El nombre de la suscripción de Azure |
   | **Grupos de recursos** | LA-MailingList-RG | El nombre del [grupo de recursos de Azure](../azure-resource-manager/management/overview.md) que se utiliza para organizar recursos relacionados. En este ejemplo se usa "LA-MailingList-RG". |
   | **Ubicación** | Oeste de EE. UU. | La región en la que desea almacenar la información de la aplicación lógica. En este ejemplo se utiliza "Oeste de EE. UU.". |
   | **Log Analytics** | Off | Mantenga el valor **Off** para el registro de diagnóstico. |
   ||||

1. Una vez que Azure implemente la aplicación, en la barra de herramientas de Azure, seleccione **Notificaciones** > **Ir al recurso** para la aplicación lógica implementada.

   ![Vaya al nuevo recurso de la aplicación lógica](./media/tutorial-process-mailing-list-subscriptions-workflow/go-to-logic-app-resource.png)

   O también puede buscar y seleccionar la aplicación lógica escribiendo el nombre en el cuadro de búsqueda.

   El Diseñador de aplicaciones lógicas se abre y muestra una página con un vídeo de introducción y los patrones de aplicaciones lógicas y desencadenadores utilizados frecuentemente. En **Plantillas**, elija **Blank Logic App**.

   ![Selección de Aplicación lógica en blanco](./media/tutorial-process-mailing-list-subscriptions-workflow/select-logic-app-template.png)

A continuación, agregue un [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts) que escuche los mensajes de correo electrónico entrantes que tienen solicitudes de suscripción. Cada aplicación lógica debe comenzar con un desencadenador, que se activa cuando se produce un evento concreto o cuando hay nuevos datos que cumplen una condición determinada. Para más información, consulte [Creación de una nueva aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="add-trigger-to-monitor-emails"></a>Agregar desencadenador para supervisar los correos electrónicos

1. En el Diseñador de aplicación lógica, en el cuadro de búsqueda, escriba `when email arrives` como filtro. En la lista **Desencadenadores**, seleccione el desencadenador **Cuando llega un nuevo correo electrónico** del proveedor de correo.

   En este ejemplo se utiliza el desencadenador de Office 365 Outlook:

   ![Seleccionar el desencadenador "Cuando llega un nuevo correo electrónico" del proveedor de correo](./media/tutorial-process-mailing-list-subscriptions-workflow/add-trigger-new-email.png)

   * Para las cuentas profesionales o educativas de Azure, seleccione Office 365 Outlook.
   * Para las cuentas de Microsoft personales, seleccione Outlook.com.

1. Si se le solicita, inicie sesión en su cuenta de correo electrónico con sus credenciales para que Logic Apps pueda crear una conexión a su cuenta de correo electrónico.

1. En el desencadenador, especifique los criterios para comprobar todo el correo electrónico nuevo.

   1. Especifique la carpeta, el intervalo y la frecuencia de comprobación del correo electrónico.

      ![Especificar la carpeta, el intervalo y la frecuencia de comprobación de los correos electrónicos](./media/tutorial-process-mailing-list-subscriptions-workflow/add-trigger-set-up-email.png)

      | Propiedad | Value | Descripción |
      |----------|-------|-------------|
      | **Carpeta** | `Inbox` | La carpeta de correo electrónico para supervisar |
      | **Intervalo** | `1` | Número de intervalos que se espera entre comprobaciones |
      | **Frecuencia** | `Hour` | Unidad de tiempo que se usa para la periodicidad. |
      ||||

   1. Ahora, agregue otra propiedad al desencadenador para que pueda filtrar por la línea de asunto del correo electrónico. Abra la lista **Agregar nuevo parámetro** y seleccione la propiedad **Filtro de asunto**.

      ![Incorporación de la propiedad "Subject Filter" al desencadenador](./media/tutorial-process-mailing-list-subscriptions-workflow/add-trigger-add-properties.png)

      Para más información acerca de las propiedades de este desencadenador, consulte la [referencia del conector de Office 365 Outlook](/connectors/office365/) o la [referencia del conector de Outlook.com](/connectors/outlook/).

   1. Después de que la propiedad aparezca en el desencadenador, escriba este texto: `subscribe-test-members-ML`

      ![Escritura de texto par ala propiedad "Subject Filter"](./media/tutorial-process-mailing-list-subscriptions-workflow/add-trigger-subject-filter-property.png)

1. Para ocultar por ahora los detalles del desencadenador, haga clic dentro de la barra de título del desencadenador.

   ![Contraer la forma para ocultar detalles](./media/tutorial-process-mailing-list-subscriptions-workflow/collapse-trigger-shape.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

La aplicación lógica estará activa, pero no hace más que comprobar el correo electrónico de entrada. Por lo tanto, agregue una acción que responda cuando se active el desencadenador.

## <a name="send-approval-email"></a>Enviar correo electrónico de aprobación

Ahora que tiene un desencadenador, agregue una [acción](../logic-apps/logic-apps-overview.md#logic-app-concepts) que envíe un correo electrónico de aprobación o rechazo de la solicitud.

1. En el desencadenador, seleccione **Nuevo paso**. 

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `approval` como filtro. En la lista de acciones, seleccione la acción **Enviar correo electrónico de aprobación** del proveedor de correo electrónico. 

   En este ejemplo se utiliza la acción de Office 365 Outlook:

   ![Seleccionar la acción "Enviar correo electrónico de aprobación"](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-send-approval-email.png)

1. Especifique la información acerca de esta acción como se describe a continuación: 

   ![Propiedades de Enviar correo electrónico de aprobación](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-approval-email-settings.png)

   | Propiedad | Value | Descripción |
   |----------|-------|-------------|
   | **To** | <*your-email-address*> | Dirección de correo electrónico del aprobador. Para realizar pruebas, puede usar su propia dirección de correo electrónico. En este ejemplo se usa la dirección de correo electrónico ficticia "sophia.owen@fabrikam.com". |
   | **Subject** | `Approve member request for test-members-ML` | Un asunto de correo electrónico descriptivo |
   | **Opciones de usuario** | `Approve, Reject` | Las opciones de respuesta que puede seleccionar el aprobador. De forma predeterminada, el aprobador puede seleccionar "Aprobar" o "Rechazar" como respuesta. |
   ||||

   Por ahora, ignore la lista de contenido dinámico que aparece al hacer clic en los cuadros específicos de edición. Esta lista le permite seleccionar la salida disponible de las acciones anteriores que puede usar como entradas en el flujo de trabajo.

   Para más información acerca de las propiedades de esta acción, consulte la [referencia del conector de Office 365 Outlook](/connectors/office365/) o la [referencia del conector de Outlook.com](/connectors/outlook/).
 
1. Guarde la aplicación lógica.

A continuación, agregue una condición para comprobar la respuesta seleccionada del aprobador.

## <a name="check-approval-response"></a>Comprobación de la respuesta de aprobación

1. En la acción **Enviar correo electrónico de aprobación**, seleccione **Nuevo paso**.

1. En **Elegir una acción**, seleccione **Integrado**. En el cuadro de búsqueda, escriba `condition` como filtro. En la lista Acciones, seleccione la acción **Condición**.

   ![Búsqueda y selección de la acción "Condition"](./media/tutorial-process-mailing-list-subscriptions-workflow/select-condition-action.png)

1. Cambie el nombre de la condición por una descripción mejor.

   1. En la barra de título de la condición, seleccione el botón de los **puntos suspensivos** ( **...** ) > **Cambiar nombre**.

      ![Cambio de nombre de la descripción de la condición](./media/tutorial-process-mailing-list-subscriptions-workflow/rename-condition-description.png)

   1. Cambie el nombre de la condición con esta descripción: `If request approved`

1. Cree una condición que compruebe si el aprobador ha seleccionado **Aprobar**.

   1. En la condición, haga clic en el cuadro **Elegir un valor**, en el lado izquierdo de la condición.

   1. En la lista de contenido dinámico que aparece, en **Enviar correo electrónico de aprobación**, seleccione la propiedad **SelectedOption**.

      ![Selección de "SelectedOption" en la lista de contenido dinámico](./media/tutorial-process-mailing-list-subscriptions-workflow/build-condition-check-approval-response.png)

   1. En el cuadro de comparación central, seleccione el operador **es igual a**.

   1. En el cuadro **Elegir un valor** del lado derecho de la condición, escriba este texto: `Approve`

      Una vez que haya terminado, la condición debe ser parecida a la de este ejemplo:

      ![Condición finalizada para el ejemplo aprobado](./media/tutorial-process-mailing-list-subscriptions-workflow/build-condition-check-approval-response-2.png)

1. Guarde la aplicación lógica.

A continuación, especifique la acción que realizará la aplicación lógica cuando el revisor apruebe la solicitud. 

## <a name="add-member-to-mailchimp-list"></a>Adición de miembros a la lista MailChimp

Ahora, agregue una acción que incorpore al miembro aprobado a la lista de distribución de correo.

1. En la rama **If true** (Si se cumple) de la condición, seleccione **Agregar una acción**.

1. En **Elegir una acción**, escriba `mailchimp` como filtro y seleccione la acción **Agregar un miembro a una lista**.

   ![Seleccionar la acción "Agregar un miembro a la lista"](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-mailchimp-add-member.png)

1. Si se le solicita que acceda a su cuenta de MailChimp, inicie sesión con sus credenciales correspondientes.

1. Especifique la información acerca de esta acción como se indica aquí:

   ![Proporcionar la información de "Agregar un miembro a una lista"](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-mailchimp-add-member-settings.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Identificador de la lista** | Sí | `test-members-ML` | El nombre de la lista de distribución de correo electrónico de MailChimp. En este ejemplo se usa "test-members-ML". |
   | **Estado** | Sí | `subscribed` | Seleccione el estado de la suscripción del nuevo miembro. En el ejemplo se utiliza "subscribed". <p>Para más información, consulte [Manage subscribers with the MailChimp API](https://developer.mailchimp.com/documentation/mailchimp/guides/manage-subscribers-with-the-mailchimp-api/) (Administración de suscriptores con MailChimp API). |
   | **Dirección de correo electrónico** | Sí | <*new-member-email-address*> | En la lista de contenido dinámico, seleccione **De** en **When a new email arrives** (Cuando llega un nuevo correo) que pasa la dirección de correo electrónico del nuevo miembro. |
   ||||

   Para más información sobre las propiedades de esta acción, consulte la [referencia del conector MailChimp](/connectors/mailchimp/).

1. Guarde la aplicación lógica.

A continuación, agregue una condición para comprobar si el nuevo miembro se ha unido correctamente a la lista de distribución de correo electrónico. De este modo, la aplicación lógica le notificará si esta operación se ha realizado correctamente o si se ha producido un error.

## <a name="check-for-success-or-failure"></a>Comprobación de si la operación se ha realizado correctamente o se ha producido un error

1. En la rama **If true**, en la acción **Agregar un miembro a una lista**, seleccione **Agregar una acción**.

1. En **Elegir una acción**, seleccione **Integrado**. En el cuadro de búsqueda, escriba `condition` como filtro. En la lista de acciones, seleccione **Condición**.

1. Cambie el nombre de la condición por esta descripción: `If add member succeeded`

1. Cree una condición que compruebe si la unión del miembro aprobado a la lista de distribución de correo electrónico se ha realizado correctamente o se ha producido un error:

   1. En la condición, haga clic en el cuadro **Elegir un valor**, en el lado izquierdo de la condición. En la lista de contenido dinámico, en **Agregar un miembro a una lista** seleccione la propiedad **Status** (Estado).

      Por ejemplo, la condición es similar a la de este ejemplo:

      ![En "Agregar un miembro a una lista", seleccionar "Status" (Estado)](./media/tutorial-process-mailing-list-subscriptions-workflow/build-condition-check-added-member.png)

   1. En el cuadro de comparación central, seleccione el operador **es igual a**.

   1. En el cuadro **Elegir un valor** del lado derecho de la condición, escriba este texto: `subscribed`

      Una vez que haya terminado, la condición debe ser parecida a la de este ejemplo:

      ![Condición finalizada para el ejemplo suscrito](./media/tutorial-process-mailing-list-subscriptions-workflow/build-condition-check-added-member-2.png)

A continuación, configure los correos electrónicos que se enviarán si la unión del miembro aprobado a la lista de distribución de correo electrónico se ha realizado correctamente o si se ha producido un error.

## <a name="send-email-if-member-added"></a>Envío de correo electrónico si se ha agregado el miembro

1. En la condición **If add member succeeded** (Si la incorporación del miembro se ha realizado correctamente), en la rama **If true**, seleccione **Agregar una acción**.

   ![En la rama "If true" de la condición, seleccione "Agregar una acción"](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-success.png)

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `outlook send email` como filtro y seleccione la acción **Enviar un correo electrónico**.

   ![Agregar la acción "Enviar un correo electrónico"](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-success-2.png)

1. Cambie el nombre de la acción por esta descripción: `Send email on success`

1. Proporcione información de esta acción como se muestra y se describe a continuación:

   ![Proporcionar información para el correo electrónico de confirmación de unión correcta](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-success-settings.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **To** | Sí | <*your-email-address*> | La dirección de correo electrónico a la que enviar el correo electrónico de confirmación de que la unión se ha realizado correctamente. Para realizar pruebas, puede usar su propia dirección de correo electrónico. |
   | **Subject** | Sí | <*subject-for-success-email*> | El asunto del correo electrónico de confirmación de que la unión se ha realizado correctamente. Para este tutorial, escriba este texto: <p>`Success! Member added to "test-members-ML": ` <p>En la lista de contenido dinámico, en **Agregar un miembro a una lista** seleccione la propiedad **Email Address** (Dirección de correo electrónico). |
   | **Cuerpo** | Sí | <*body-for-success-email*> | El contenido del cuerpo del correo electrónico de confirmación de unión correcta. Para este tutorial, escriba este texto: <p>`New member has joined "test-members-ML":` <p>En la lista de contenido dinámico, seleccione la propiedad **Email Address** (Dirección de correo electrónico). <p>En la fila siguiente, escriba este texto: `Member opt-in status: ` <p> En la lista de contenido dinámico, en **Agregar un miembro a una lista** seleccione la propiedad **Status** (Estado). |
   |||||

1. Guarde la aplicación lógica.

## <a name="send-email-if-member-not-added"></a>Envío de correo electrónico si el miembro no se ha agregado

1. En la condición **If add member succeeded** (Si la incorporación del miembro se ha realizado correctamente), en la rama **If false**, seleccione **Agregar una acción**.

   ![En la rama "If false" de la condición, seleccione "Agregar una acción"](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-failed.png)

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `outlook send email` como filtro y seleccione la acción **Enviar un correo electrónico**.

   ![Agregar acción para "Enviar un correo electrónico"](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-failed-2.png)

1. Cambie el nombre de la acción por esta descripción: `Send email on failure`

1. Especifique la información acerca de esta acción como se indica aquí:

   ![Proporcionar información para el correo electrónico de error](./media/tutorial-process-mailing-list-subscriptions-workflow/add-action-email-failed-settings.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **To** | Sí | <*your-email-address*> | La dirección de correo electrónico a la que enviar el correo electrónico de error. Para realizar pruebas, puede usar su propia dirección de correo electrónico. |
   | **Subject** | Sí | <*subject-for-failure-email*> | El asunto del correo electrónico de error. Para este tutorial, escriba este texto: <p>`Failed, member not added to "test-members-ML": ` <p>En la lista de contenido dinámico, en **Agregar un miembro a una lista** seleccione la propiedad **Email Address** (Dirección de correo electrónico). |
   | **Cuerpo** | Sí | <*body-for-failure-email*> | El contenido del cuerpo del correo electrónico de error. Para este tutorial, escriba este texto: <p>`Member might already exist. Check your MailChimp account.` |
   |||||

1. Guarde la aplicación lógica. 

A continuación, pruebe la aplicación lógica, que ahora es similar a este ejemplo:

![Ejemplo de flujo de trabajo de aplicación lógica finalizado](./media/tutorial-process-mailing-list-subscriptions-workflow/tutorial-high-level-completed.png)

## <a name="run-your-logic-app"></a>Ejecución de la aplicación lógica

1. Envíese a sí mismo una solicitud de correo electrónico para unirse a la lista de distribución de correo electrónico. Espere a que la solicitud aparezca en la bandeja de entrada.

1. Para iniciar manualmente la aplicación lógica, en la barra de herramientas del diseñador, elija **Ejecutar**. 

   Si el correo electrónico tiene un asunto que coincide con el del filtro de asunto del desencadenador, la aplicación lógica le enviará un correo electrónico para que apruebe la solicitud de suscripción.

1. En el correo electrónico de aprobación, seleccione **Aprobar**.

1. Si la dirección de correo electrónico del suscriptor no existe en la lista de distribución correspondiente, la aplicación lógica agregará la dirección de esta persona y enviará un correo electrónico parecido al de este ejemplo:

   ![Correo electrónico de ejemplo: suscripción correcta](./media/tutorial-process-mailing-list-subscriptions-workflow/add-member-mailing-list-success.png)

   Si la aplicación lógica no puede agregar al suscriptor, recibirá un correo electrónico parecido al de este ejemplo:

   ![Correo electrónico de ejemplo: suscripción errónea](./media/tutorial-process-mailing-list-subscriptions-workflow/add-member-mailing-list-failed.png)

   Si no recibe ningún correo electrónico, compruebe la carpeta de correo electrónico no deseado. El filtro de correo electrónico no deseado podría redirigir esta clase de correo. Si tampoco aparece allí y no está seguro de que la aplicación lógica se ejecutara correctamente, consulte el artículo de [solución de problemas en la aplicación lógica](../logic-apps/logic-apps-diagnosing-failures.md).

Enhorabuena, acaba de crear y ejecutar una aplicación lógica que integra la información de Azure, los servicios de Microsoft y otras aplicaciones de SaaS.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no necesite el ejemplo de aplicación lógica, elimine el grupo de recursos que contiene la aplicación lógica y los recursos relacionados. 

1. En el menú principal de Azure, vaya a **Grupos de recursos** y seleccione el grupo de recursos de la aplicación lógica.

1. En el menú del grupo de recursos, seleccione **Información general** > **Eliminar grupo de recursos**. 

   !["Introducción" > "Eliminar grupo de recursos"](./media/tutorial-process-mailing-list-subscriptions-workflow/delete-resource-group.png)

1. Escriba el nombre del grupo de recursos como confirmación y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una aplicación lógica que administra las aprobaciones de las solicitudes de la lista de distribución de correo electrónico. Ahora, aprenda a crear una aplicación lógica que procesa y almacena datos adjuntos de correo electrónico mediante la integración de servicios de Azure, como Azure Storage y Azure Functions.

> [!div class="nextstepaction"]
> [Procesamiento de archivos adjuntos de correo electrónico](../logic-apps/tutorial-process-email-attachments-workflow.md)
