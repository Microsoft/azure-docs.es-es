---
title: 'Supervisión de los cambios en las máquinas virtuales: Azure Event Grid y Logic Apps'
description: Comprobar si hay cambios en las máquinas virtuales mediante el uso de Azure Event Grid y Logic Apps
services: logic-apps, event-grid
ms.service: logic-apps
ms.suite: integration
author: ecfan
ms.author: estfan
ms.reviewer: estfan, LADocs
ms.topic: tutorial
ms.date: 07/20/2020
ms.openlocfilehash: 7af555a634f0e362bdf2d530627a782843105bdf
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87461279"
---
# <a name="tutorial-monitor-virtual-machine-changes-by-using-azure-event-grid-and-logic-apps"></a>Tutorial: Supervisión de los cambios en las máquinas virtuales con Azure Event Grid y Logic Apps

Para supervisar los eventos específicos que se producen en recursos de Azure o en recursos de terceros y responder a ellos, puede automatizar y ejecutar tareas como un flujo de trabajo mediante la creación de una [aplicación lógica](../logic-apps/logic-apps-overview.md) que use el código mínimo. Estos recursos pueden publicar esos eventos en una instancia de [Azure Event Grid](../event-grid/overview.md). A su vez, la cuadrícula de eventos envía esos eventos a los suscriptores que tienen colas, webhooks o [centros de eventos](../event-hubs/event-hubs-about.md) como puntos de conexión. Como suscriptor, su aplicación lógica puede esperar esos eventos desde la cuadrícula de eventos antes de ejecutar flujos de trabajo automatizados para llevar a cabo ciertas tareas.

Por ejemplo, aquí incluimos algunos eventos que los publicadores pueden enviar a los suscriptores a través del servicio Azure Event Grid:

* Crear, leer, actualizar o eliminar un recurso. Por ejemplo, puede supervisar los cambios que podrían incurrir en gastos en su suscripción de Azure y afectar a la facturación.

* Agregar o quitar a una persona de una suscripción de Azure.

* La aplicación realiza una acción concreta.

* Aparece un nuevo mensaje en una cola.

En este tutorial se crea una aplicación lógica que supervisa los cambios realizados en una máquina virtual y envía mensajes de correo electrónico sobre dichos cambios. Cuando crea una aplicación lógica con una suscripción a eventos de un recurso de Azure, los eventos fluyen desde ese recurso a través de una cuadrícula de eventos a la aplicación lógica. El tutorial le guía en la creación de esta aplicación lógica:

![Captura de pantalla del diseñador de Logic Apps que muestra el flujo de trabajo para supervisar la máquina virtual con Event Grid.](./media/monitor-virtual-machine-changes-event-grid-logic-app/monitor-virtual-machine-event-grid-logic-app-overview.png)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una aplicación lógica que supervisa eventos procedentes de una cuadrícula de eventos.
> * Incorporar una condición que comprueba específicamente cambios en la máquina virtual.
> * Enviar un mensaje de correo electrónico cuando la máquina virtual cambia.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Una cuenta de correo electrónico de cualquier proveedor de correo electrónico que sea compatible con Azure Logic Apps, como Office 365 Outlook, Outlook.com o Gmail, para el envío de notificaciones. En el caso de otros proveedores, [consulte la lista de conectores que se muestra aquí](/connectors/).

  Este tutorial utiliza una cuenta de Office 365 Outlook. Si utiliza una cuenta de correo electrónico diferente, los pasos generales siguen siendo los mismos pero la interfaz de usuario podría ser ligeramente distinta.

  > [!IMPORTANT]
  > Si quiere usar el conector de Gmail, solo las cuentas empresariales de G-Suite pueden usar este conector sin restricciones en las aplicaciones lógicas. Si tiene una cuenta de consumidor de Gmail, puede usar este conector solo con servicios específicos aprobados por Google, o puede [crear una aplicación cliente de Google para usarla en la autenticación con el conector de Gmail](/connectors/gmail/#authentication-and-bring-your-own-application). Para más información, consulte [Directivas de privacidad y seguridad de datos de los conectores de Google en Azure Logic Apps](../connectors/connectors-google-data-security-privacy-policy.md).

* Una [máquina virtual](https://azure.microsoft.com/services/virtual-machines) sola en su grupo de recursos de Azure. Si aún no lo ha hecho, cree una máquina virtual siguiendo un [tutorial para crear una máquina virtual](../virtual-machines/windows/quick-create-portal.md). Para hacer que la máquina virtual publique eventos, [no es necesario hacer nada más](../event-grid/overview.md).

## <a name="create-blank-logic-app"></a>Creación de una aplicación lógica en blanco

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con sus credenciales de su cuenta de Azure.

1. En el menú principal de Azure, seleccione **Crear un recurso** > **Integración** > **Aplicación lógica**.

   ![Captura de pantalla de Azure Portal que muestra el botón para crear un recurso de aplicación lógica.](./media/monitor-virtual-machine-changes-event-grid-logic-app/azure-portal-create-logic-app.png)

1. En la **aplicación lógica**, proporcione información acerca del recurso de aplicación lógica. Seleccione **Crear** cuando haya terminado.

   ![Captura de pantalla del menú de creación de aplicaciones lógicas que muestra detalles como el nombre, la suscripción, el grupo de recursos y la ubicación.](./media/monitor-virtual-machine-changes-event-grid-logic-app/create-logic-app-for-event-grid.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **Nombre** | Sí | <*nombre-de-la-aplicación-lógica*> | Proporcione un nombre único para la aplicación lógica. |
   | **Suscripción** | Sí | <*Azure-subscription-name*> | Seleccione la misma suscripción de Azure para todos los servicios de este tutorial. |
   | **Grupos de recursos** | Sí | <*Azure-resource-group*> | Nombre del grupo de recursos de Azure para la aplicación lógica, que puede seleccionar para todos los servicios de este tutorial. |
   | **Ubicación** | Sí | <*Azure-region*> | Seleccione la misma región para todos los servicios de este tutorial. |
   |||

1. Una vez que Azure ha implementado la aplicación lógica, el Diseñador de aplicaciones lógicas muestra una página con un vídeo de introducción y desencadenadores utilizados frecuentemente. Desplácese más allá del vídeo y los desencadenadores.

1. En **Plantillas**, elija **Blank Logic App**.

   ![Captura de pantalla de las plantillas de Logic Apps que muestra la selección para crear una aplicación lógica en blanco.](./media/monitor-virtual-machine-changes-event-grid-logic-app/choose-logic-app-template.png)

   El Diseñador de aplicaciones lógicas ahora muestra los [*desencadenadores*](../logic-apps/logic-apps-overview.md#logic-app-concepts) que puede usar para iniciar la aplicación lógica. Cada aplicación lógica debe comenzar con un desencadenador, que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador se activa, Azure Logic Apps crea una instancia de flujo de trabajo que ejecuta la aplicación lógica.

## <a name="add-an-event-grid-trigger"></a>Incorporación de un desencadenador de Event Grid

Ahora, agregue el desencadenador de Event Grid que usa para supervisar el grupo de recursos de la máquina virtual.

1. En el cuadro de búsqueda del diseñador, escriba `event grid` como filtro. En la lista de desencadenadores, seleccione el desencadenador **Cuando se produce un evento de recursos**.

   ![Captura de pantalla del diseñador de Logic Apps que muestra la selección del desencadenador Event Grid en un evento de recurso.](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-event-grid-trigger.png)

1. Cuando se le solicite, inicie sesión en Azure Event Grid con las credenciales de la cuenta de Azure. En la lista **Inquilino**, que muestra el inquilino de Azure Active Directory asociado con la suscripción de Azure, compruebe que aparece al inquilino correcto, por ejemplo:

   ![Captura de pantalla del diseñador de Logic Apps que muestra el mensaje de inicio de sesión de Azure para conectarse a Event Grid.](./media/monitor-virtual-machine-changes-event-grid-logic-app/sign-in-event-grid.png)

   > [!NOTE]
   > Si inició sesión con una cuenta personal de Microsoft, como @outlook.com o @hotmail.com, el desencadenador de Event Grid podría no aparecer correctamente. Como alternativa, seleccione [Conectar con entidad de servicio](../active-directory/develop/howto-create-service-principal-portal.md) o autentíquese como miembro de Azure Active Directory asociado con la suscripción de Azure, por ejemplo, *nombreDeUsuario*@emailoutlook.onmicrosoft.com.

1. Ahora suscriba la aplicación lógica a los eventos del publicador. Proporcione los detalles de la suscripción a eventos como se describe en la tabla siguiente, por ejemplo:

   ![Captura de pantalla del diseñador de Logic Apps que muestra el editor de detalles del desencadenador cuando se produce un evento de recurso.](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-event-grid-trigger-details.png)

   | Propiedad | Obligatorio | Value | Descripción |
   | -------- | -------- | ----- | ----------- |
   | **Suscripción** | Sí | <*event-publisher-Azure-subscription-name*> | Seleccione el nombre de la suscripción de Azure asociada con el *publicador de eventos*. Para este tutorial, seleccione el nombre de la suscripción de Azure de la máquina virtual. |
   | **Tipo de recurso** | Sí | <*event-publisher-Azure-resource-type*> | Seleccione el tipo de recurso de Azure para el publicador de eventos. Para más información sobre los tipos de recursos de Azure, consulte [Tipos y proveedores de recursos de Azure](../azure-resource-manager/management/resource-providers-and-types.md). Para este tutorial, seleccione el valor `Microsoft.Resources.ResourceGroups` para supervisar los grupos de recursos de Azure. |
   | **Nombre de recurso** |  Sí | <*event-publisher-Azure-resource-name*> | Seleccione el nombre de recurso de Azure para el publicador de eventos. Esta lista varía según el tipo de recurso que ha seleccionado. Para este tutorial, seleccione el nombre del grupo de recursos de Azure que incluye la máquina virtual. |
   | **Elemento de tipo de evento** |  No | <*event-types*> | Seleccione uno o varios tipos de evento específicos para filtrar y enviar a Event Grid. Por ejemplo, si lo desea puede agregar estos tipos de eventos para detectar cuándo se cambian o eliminan los recursos: <p><p>- `Microsoft.Resources.ResourceActionSuccess` <br>- `Microsoft.Resources.ResourceDeleteSuccess` <br>- `Microsoft.Resources.ResourceWriteSuccess` <p>Para más información, consulte los temas siguientes: <p><p>- [Esquema de eventos de Azure Event Grid para grupos de recursos](../event-grid/event-schema-resource-groups.md) <br>- [Descripción del filtrado de eventos](../event-grid/event-filtering.md) <br>- [Filtrado de eventos para Event Grid](../event-grid/how-to-filter-events.md) |
   | Para agregar propiedades opcionales, seleccione **Agregar nuevo parámetro** y seleccione las propiedades que desee. | No | {ver descripciones} | * **Filtro de prefijo**: En este tutorial, deje esta propiedad vacía. El comportamiento predeterminado coincide con todos los valores. Sin embargo, puede especificar una cadena de prefijo como un filtro, por ejemplo, una ruta de acceso y un parámetro para un recurso concreto. <p>* **Filtro de sufijo**: En este tutorial, deje esta propiedad vacía. El comportamiento predeterminado coincide con todos los valores. Sin embargo, puede especificar una cadena de sufijo como un filtro, por ejemplo, una extensión de nombre de archivo, si desea solo determinados tipos de archivos. <p>* **Nombre de la suscripción**: Para este tutorial puede proporcionar un nombre único para la suscripción a eventos. |
   |||

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar). Para contraer y ocultar los detalles de una acción en la aplicación lógica, seleccione la barra de título de la acción.

   ![Captura de pantalla del diseñador de Logic Apps que muestra el botón Guardar para guardar los cambios de flujo de trabajo.](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-event-grid-save.png)

   Cuando se guarda la aplicación lógica con un desencadenador de Event Grid, Azure crea automáticamente una suscripción a eventos para la aplicación lógica en el recurso seleccionado. Así, cuando el recurso publica un evento en la cuadrícula de eventos, esa cuadrícula de eventos inserta automáticamente el evento en la aplicación lógica. Este evento desencadena la aplicación lógica, que crea y ejecuta una instancia del flujo de trabajo que se define en los pasos siguientes.

La aplicación lógica ahora está activa y escucha los eventos de Event Grid, pero no hace nada hasta que agregue acciones al flujo de trabajo.

## <a name="add-a-condition"></a>Agregar una condición

Si desea ejecutar la aplicación lógica solo cuando se produzca un evento u operación específicos, agregue una condición que compruebe la operación `Microsoft.Compute/virtualMachines/write`. Cuando esta condición se cumpla, la aplicación lógica le enviará un mensaje de correo electrónico con detalles acerca de la máquina virtual actualizada.

1. En el Diseñador de aplicación lógica, bajo el desencadenador de Event Grid, seleccione **Nuevo paso**.

   ![Captura de pantalla del diseñador de Logic Apps que muestra el botón para agregar un nuevo paso al flujo de trabajo.](./media/monitor-virtual-machine-changes-event-grid-logic-app/choose-new-step-condition.png)

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `condition` como filtro. En la lista Acciones, seleccione la acción **Condición**.

   ![Captura de pantalla del diseñador de Logic Apps que muestra el botón para agregar una acción Condición.](./media/monitor-virtual-machine-changes-event-grid-logic-app/select-condition.png)

   El Diseñador de aplicaciones lógicas agrega una condición vacía al flujo de trabajo, incluidas las rutas de acceso de la acción que se deben seguir en función de si la condición se cumple o no.

   ![Captura de pantalla del diseñador de Logic Apps que muestra una condición vacía que se agregó al flujo de trabajo.](./media/monitor-virtual-machine-changes-event-grid-logic-app/empty-condition.png)

1. Cambie el nombre del título de la condición a `If a virtual machine in your resource group has changed`. En la barra de título de la condición, seleccione el botón de los puntos suspensivos ( **...** ) y **Cambiar nombre**.

   ![Captura de pantalla del diseñador de Logic Apps que muestra el menú contextual del editor de condiciones con la opción Cambiar nombre seleccionada.](./media/monitor-virtual-machine-changes-event-grid-logic-app/rename-condition.png)

1. Cree una condición que comprueba el evento `body` de un objeto `data` donde la propiedad `operationName` es igual a la operación `Microsoft.Compute/virtualMachines/write`. Más información sobre el [Esquema de eventos de la cuadrícula de eventos](../event-grid/event-schema.md).

   1. En la primera fila, debajo de **And**, haga clic en el cuadro izquierdo. En la lista de contenido dinámico que aparece, seleccione **Expresión**.

      ![Captura de pantalla del diseñador de Logic Apps que muestra una condición con el editor de expresiones seleccionado.](./media/monitor-virtual-machine-changes-event-grid-logic-app/condition-choose-expression.png)

   1. En el editor de expresiones, escriba esta expresión, que devuelve el nombre de la operación del desencadenador, y seleccione **Aceptar**:

      `triggerBody()?['data']['operationName']`

      Por ejemplo:

      ![Captura de pantalla del diseñador de Logic Apps que muestra el editor de condiciones con la expresión para extraer el nombre de la operación.](./media/monitor-virtual-machine-changes-event-grid-logic-app/condition-add-data-operation-name.png)

   1. En el cuadro central, mantenga el operador **es igual que**.

   1. En el cuadro de la derecha, escriba este valor, que es la operación específica que se va a supervisar:

      `Microsoft.Compute/virtualMachines/write`

   Ahora, la condición finalizada se parecerá a este ejemplo:

   ![Captura de pantalla del diseñador de Logic Apps que muestra una condición que compara la operación.](./media/monitor-virtual-machine-changes-event-grid-logic-app/complete-condition.png)

   Si cambia de la vista de diseño a la vista de código y vuelve a la vista de diseño, la expresión que especificó en la condición se resuelve en el token **data.operationName**:

   ![Captura de pantalla del diseñador de Logic Apps que muestra una condición con tokens resueltos.](./media/monitor-virtual-machine-changes-event-grid-logic-app/resolved-condition.png)

1. Guarde la aplicación lógica.

## <a name="send-email-notifications"></a>Envío de notificaciones por correo electrónico

Ahora, agregue una [*acción*](../logic-apps/logic-apps-overview.md#logic-app-concepts) para poder recibir un mensaje de correo electrónico cuando se cumpla la condición especificada.

1. En el cuadro **If true** de la condición, seleccione **Agregar una acción**.

   ![Captura de pantalla del editor de condiciones del diseñador de Logic Apps que muestra el botón Agregar una acción cuando la condición es verdadera.](./media/monitor-virtual-machine-changes-event-grid-logic-app/condition-true-add-action.png)

1. En **Elegir una acción**, en el cuadro de búsqueda, escriba `send an email` como filtro. En función de su proveedor de correo electrónico, busque y seleccione el conector correspondiente. A continuación, seleccione la acción "enviar correo electrónico" para el conector. Por ejemplo:

   * Para una cuenta profesional o educativa de Azure, seleccione el conector de Office 365 Outlook.

   * Para las cuentas de Microsoft personales, seleccione el conector de Outlook.com.

   * Para las cuentas de Gmail, seleccione el conector de Gmail.

   Este tutorial continúa con el conector de Office 365 Outlook. Si utiliza un proveedor diferente, los pasos siguen siendo los mismos, pero la interfaz de usuario podría ser ligeramente diferente.

   ![Captura de pantalla del diseñador de Logic Apps que muestra los resultados de buscar la acción Enviar correo electrónico en el conector de Office 365 Outlook.](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-send-email.png)

1. Si ya no tiene una conexión para el proveedor de correo electrónico, inicie sesión en su cuenta de correo electrónico cuando se le pida que se autentique.

1. Cambie el nombre de la acción del correo electrónico de envío por este título: `Send email when virtual machine updated`

1. Proporcione información sobre el correo electrónico como se especifica en la tabla siguiente:

   ![Captura de pantalla del diseñador de Logic Apps que muestra cómo el contenido dinámico se agrega a la línea de asunto del correo electrónico cuando una condición es verdadera.](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-empty-email-action.png)

   > [!TIP]
   > Para seleccionar salidas de los pasos anteriores del flujo de trabajo, haga clic en un cuadro de edición para que aparezca la lista de contenido dinámico o seleccione **Agregar contenido dinámico**. Para más resultados, seleccione **Ver más** para cada sección de la lista. Para cerrar la lista de contenido dinámico, seleccione **Agregar contenido dinámico** de nuevo.

   | Propiedad | Obligatorio | Value | Descripción |
   | -------- | -------- | ----- | ----------- |
   | **To** | Sí | <*destinatario\@dominio*> | Escriba la dirección de correo electrónico del destinatario. Para realizar pruebas, puede usar su propia dirección de correo electrónico. |
   | **Subject** | Sí | `Resource updated:` **Asunto** | Escriba el contenido del asunto del correo electrónico. Para este tutorial, escriba el texto especificado y seleccione el campo **Asunto** del evento. En este caso, el asunto del correo electrónico incluye el nombre del recurso actualizado (máquina virtual). |
   | **Cuerpo** | Sí | `Resource:` **Tema** <p>`Event type:` **Tipo de evento**<p>`Event ID:` **ID**<p>`Time:` **Hora del evento** | Escriba el contenido del cuerpo del correo electrónico. Para este tutorial, escriba el texto especificado y seleccione los campos **Tema**, **Tipo de evento**, **ID** y **Hora del evento** del evento para que el correo electrónico incluya el recurso que desencadenó el evento, el tipo de evento, la marca de tiempo del evento y el identificador del evento de la actualización. Para este tutorial, el recurso es el grupo de recursos de Azure seleccionado en el desencadenador. <p>Para agregar líneas en blanco en el contenido, presione MAYÚS + ENTRAR. |
   ||||

   > [!NOTE]
   > Si selecciona un campo que representa una matriz, el diseñador agrega automáticamente un bucle **For each** alrededor de la acción que haga referencia a la matriz. De este modo, la aplicación lógica realiza la acción en cada elemento de la matriz.

   Ahora, la acción de correo electrónico podría parecerse a este ejemplo:

   ![Captura de pantalla del diseñador de Logic Apps que muestra los resultados seleccionados que se enviarán en un correo electrónico cuando se actualice la máquina virtual.](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-send-email-details.png)

   Y la aplicación lógica terminada podría parecerse a este ejemplo:

   ![Captura de pantalla del diseñador de Logic Apps que muestra la aplicación lógica que se creó con los detalles del desencadenador y las acciones.](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-completed.png)

1. Guarde la aplicación lógica. Para contraer y ocultar los detalles de cada acción en la aplicación lógica, seleccione la barra de título de la acción.

   La aplicación lógica ahora está activa, pero espera a que haya cambios en la máquina virtual antes de hacer algo. Para probar la aplicación lógica ahora, pase a la sección siguiente.

## <a name="test-your-logic-app-workflow"></a>Comprobación del flujo de trabajo de la aplicación lógica

1. Para comprobar que la aplicación lógica está recibiendo los eventos especificados, actualice la máquina virtual.

   Por ejemplo, puede cambiar el tamaño de la máquina virtual en Azure Portal o [cambiar el tamaño de la máquina virtual con Azure PowerShell](../virtual-machines/windows/resize-vm.md).

   Transcurridos unos instantes, debería recibir un correo electrónico. Por ejemplo:

   ![Captura de pantalla de un ejemplo de correo electrónico de Outlook que muestra detalles sobre la actualización de la máquina virtual.](./media/monitor-virtual-machine-changes-event-grid-logic-app/email.png)

1. Para revisar el historial de ejecución y desencadenamiento de la aplicación lógica, en el menú de la aplicación lógica, elija **Información general**. Para ver más detalles acerca de una ejecución, seleccione la fila correspondiente a esa ejecución.

   ![Captura de pantalla de la página de información general de una aplicación lógica que muestra una ejecución correcta seleccionada.](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-run-history.png)

1. Para ver las entradas y salidas de cada paso, expanda el paso que desea revisar. Esta información puede ayudarle a diagnosticar y depurar los problemas de la aplicación lógica.

   ![Captura de pantalla del historial de ejecución de la aplicación lógica que muestra los detalles de cada ejecución.](./media/monitor-virtual-machine-changes-event-grid-logic-app/logic-app-run-history-details.png)

Enhorabuena, ha creado y ejecutado una aplicación lógica que supervisa los eventos del recurso mediante una cuadrícula de eventos y le envía por correo electrónico un mensaje cuando se producen esos eventos. También ha aprendido cómo puede crear fácilmente flujos de trabajo que automatizan los procesos y a integrar sistemas y servicios en la nube.

Puede supervisar otros cambios de configuración con las cuadrículas de eventos y las aplicaciones lógicas, por ejemplo:

* Una máquina virtual obtiene derechos de control de acceso basado en roles (RBAC).
* Se realizan cambios en un grupo de seguridad de red (NSG) en una interfaz de red (NIC).
* Los discos para una máquina virtual se agregan o se quitan.
* Se asigna una dirección IP pública a una máquina virtual de NIC.

## <a name="clean-up-resources"></a>Limpieza de recursos

Este tutorial utiliza recursos y realiza acciones que generan gastos en su suscripción de Azure. Cuando haya terminado con el tutorial y las pruebas, asegúrese de deshabilitar o eliminar todos los recursos si no desea que generen gastos.

* Para detener la ejecución de la aplicación lógica sin eliminar el trabajo, deshabilítela. En el menú de la aplicación lógica, seleccione **Introducción**. En la barra de herramientas, seleccione **Deshabilitar**.

  ![Captura de pantalla de información general de la aplicación lógica que muestra el botón Deshabilitar seleccionado para deshabilitar la aplicación lógica.](./media/monitor-virtual-machine-changes-event-grid-logic-app/turn-off-disable-logic-app.png)

  > [!TIP]
  > Si no ve el menú de la aplicación lógica, intente volver al panel de Azure y vuelva a abrir la aplicación lógica.

* Para eliminar la aplicación lógica de forma definitiva, en el menú de la aplicación lógica, seleccione **Información general**. En la barra de herramientas, seleccione **Eliminar**. Confirme que quiere eliminar la aplicación lógica y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

* [Creación y enrutamiento de eventos personalizados con Event Grid](../event-grid/custom-event-quickstart.md)
