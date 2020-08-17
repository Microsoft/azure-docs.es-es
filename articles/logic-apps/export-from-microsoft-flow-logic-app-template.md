---
title: Exportación de flujos desde Power Automate en Azure Logic Apps
description: Migración de flujos desde Power Automate a Azure Logic Apps mediante su exportación como plantillas de Azure Resource Manager
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: conceptual
ms.date: 06/03/2020
ms.openlocfilehash: b2a45218118282f4f1cadb29c9022fc05b30f907
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87078732"
---
# <a name="export-flows-from-power-automate-and-deploy-to-azure-logic-apps"></a>Exportación de flujos desde Power Automate y su implementación en Azure Logic Apps

Para ampliar y expandir las funcionalidades de un flujo, puede migrarlo desde [Power Automate](https://flow.microsoft.com) a [Azure Logic Apps](../logic-apps/logic-apps-overview.md). Puede exportar el flujo en forma de plantilla de Azure Resource Manager para una aplicación lógica, implementar esa plantilla en un grupo de recursos de Azure y, después, abrir la aplicación en el Diseñador de aplicación lógica.

> [!NOTE]
> No todos los conectores de Power Automate están disponibles en Azure Logic Apps. Solo se pueden migrar flujos de Power Automate que tengan conectores equivalentes en Azure Logic Apps. Por ejemplo, el desencadenador Button, el conector Approvals y el conector Notification son específicos de Power Automate. En la actualidad, los flujos basados en OpenAPI de Power Automate no se admiten para la exportación e implementación como plantillas de aplicación lógica.
>
> * Para saber qué conectores de Power Automate carecen de equivalente en Logic Apps, vea [Conectores de Power Automate](/connectors/connector-reference/connector-reference-powerautomate-connectors).
>
> * Para saber qué conectores de Logic Apps carecen de equivalente en Power Automate, vea [Conectores de Logic Apps](/connectors/connector-reference/connector-reference-powerautomate-connectors).

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* El flujo que desea exportar desde Power Automate

## <a name="export-your-flow"></a>Exportación del flujo

1. Inicie sesión en [Power Automate](https://flow.microsoft.com) y seleccione **Mis flujos**. Busque y seleccione el flujo. En la barra de herramientas, seleccione el botón de puntos suspensivos ( **...** ) > **Exportar** > **Plantilla de Logic Apps (.json)** .

   ![Exportación del flujo desde Power Automate](./media/export-from-microsoft-flow-logic-app-template/export-flow.png)

1. Guarde el archivo .json de la plantilla en la ubicación que quiera.

Para más información, consulte [Crecer hasta Azure Logic Apps](https://flow.microsoft.com/blog/grow-up-to-logic-apps/).

## <a name="deploy-template-by-using-the-azure-portal"></a>Implementación de una plantilla mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

1. En la página principal de Azure, en el cuadro de búsqueda, escriba `custom template`. En los resultados, seleccione **Implementar una plantilla personalizada** > **Crear**.

   ![Búsqueda y selección de "Implementación de plantillas"](./media/export-from-microsoft-flow-logic-app-template/select-template-deployment.png)

1. En **Implementación personalizada**, seleccione **Cree su propia plantilla en el editor**.

   ![Seleccionar "Cree su propia plantilla en el editor"](./media/export-from-microsoft-flow-logic-app-template/build-template-in-editor.png)

1. En la barra de herramientas **Editar plantilla**, seleccione **Cargar archivo**.

   ![Seleccionar "Cargar archivo"](./media/export-from-microsoft-flow-logic-app-template/load-file.png)

1. Vaya a la ubicación donde ha guardado el archivo de plantilla JSON que ha exportado de Power Automate. Seleccione el archivo de plantilla > **Abrir**.

1. Después de que el editor muestre el archivo JSON, los parámetros y los recursos de la plantilla, seleccione**Guardar**.

   ![Guardar plantilla](./media/export-from-microsoft-flow-logic-app-template/save-template.png)

1. Ahora proporcione más información de la aplicación lógica.

   1. Seleccione o especifique los valores de los parámetros de entrada de la plantilla.

      | Propiedad | Descripción |
      |----------|-------------|
      | **Suscripción** | Suscripción de Azure que se va a usar para la facturación |
      | **Grupos de recursos** | Grupo de recursos de Azure que se va a usar para la aplicación lógica. Puede usar un grupo existente o crear uno nuevo. |
      | **Ubicación** | Región de Azure que se va a usar si crea un nuevo grupo de recursos |
      | **Nombre de la aplicación lógica** | Nombre que se va a usar para el recurso de aplicación lógica |
      | **Ubicación de la aplicación lógica** | Región de Azure en la que quiere crear el recurso de aplicación lógica, en caso de que sea diferente del grupo de recursos de Azure |
      | <*connection-name*> | Uno o varios nombres de cualquier conexión creada previamente que la aplicación lógica pueda volver a usar <p><p>**Nota**: Si esta es la primera aplicación lógica que crea, todas las conexiones se crean como nuevas, por lo que puede aceptar los nombres predeterminados. Si no es así, puede especificar los nombres de las conexiones creadas anteriormente, que puede usar en varias aplicaciones lógicas. |
      |||

      Por ejemplo:

      ![Especificar los parámetros de entrada de la plantilla](./media/export-from-microsoft-flow-logic-app-template/template-input-parameters.png)

   1. Cuando haya terminado, revise los **Términos y condiciones** relacionados con la creación de los recursos de Azure necesarios y la facturación de la suscripción de Azure en consecuencia.

   1. Cuando esté listo, seleccione **Acepto los términos y condiciones indicados anteriormente** > **Comprar**.

      Azure implementa la plantilla como una aplicación lógica en el grupo de recursos que especifique.

1. Todas las aplicaciones lógicas que migre desde Power Automate se implementan en estado deshabilitado. Antes de habilitar la aplicación lógica, autorice las nuevas conexiones mediante estos pasos:

   1. En Azure Portal, abra la aplicación lógica que ha creado. En el menú de la aplicación lógica, seleccione **Diseñador de aplicación lógica**.

      Todas las conexiones que requieren autorización muestran un icono de advertencia:

      ![Icono de advertencia](./media/export-from-microsoft-flow-logic-app-template/authorize-connections.png)

   1. En todos los pasos que requieran una conexión autorizada, expanda el paso y seleccione **Agregar nueva**.

      ![Agregar nueva conexión](./media/export-from-microsoft-flow-logic-app-template/add-new-connection.png)

   1. Inicie sesión en cada servicio o proporcione las credenciales necesarias para autorizar la conexión.

   1. Después de actualizar las conexiones, en la barra de herramientas del diseñador, seleccione **Guardar**.

1. Cuando esté listo para activar la aplicación lógica, en el menú de la aplicación lógica, seleccione **Información general** y, después, seleccione **Habilitar**.

   ![Habilitar aplicación lógica](./media/export-from-microsoft-flow-logic-app-template/enable-logic-app.png)

1. Para evitar que se ejecuten flujos de trabajo duplicados, asegúrese de desactivar o eliminar el flujo original.

## <a name="deploy-template-by-using-visual-studio"></a>Implementación de una plantilla mediante Visual Studio

Si ha configurado Visual Studio con los [requisitos previos](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#prerequisites) para la creación de aplicaciones lógicas, puede implementar la plantilla exportada en Azure Logic Apps mediante Visual Studio.

1. En Visual Studio, busque y abra el archivo .json de la plantilla de aplicación lógica que ha exportado desde Power Automate.

1. En Visual Studio, cree un proyecto de **Grupo de recursos de Azure** que use la plantilla **Aplicación lógica** mediante los pasos de [Inicio rápido: Creación y automatización de tareas, procesos y flujos de trabajo con Azure Logic Apps en Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

   En este ejemplo se crea una solución de Visual Studio de nombre "ImportedLogicApp".

   ![Creación de un proyecto de grupo de recursos de Azure](./media/export-from-microsoft-flow-logic-app-template/create-azure-resource-group-project.png)

1. Una vez creada la solución, en el Explorador de soluciones, abra el archivo **LogicApp.json**, si aún no está abierto.

1. Copie el contenido de la plantilla exportada y sobrescriba el contenido del archivo **LogicApp.json**.

1. Antes de implementar la aplicación lógica, autorice las nuevas conexiones, para lo que debe seguir estos pasos:

   1. Abra el menú contextual de **LogicApp.json** y, después, seleccione **Abrir con el Diseñador de aplicación lógica**.

      ![Abrir plantilla con el Diseñador de aplicación lógica](./media/export-from-microsoft-flow-logic-app-template/open-logic-app-designer.png)

   1. Si se solicita, seleccione la suscripción de Azure y el grupo de recursos que desea usar para implementar la aplicación lógica.

      ![Seleccionar una suscripción de Azure y un grupo de recursos](./media/export-from-microsoft-flow-logic-app-template/select-azure-subscription-resource-group-deployment.png)

      Después de que la aplicación lógica aparezca en el diseñador, todas las conexiones que requieran autorización mostrarán iconos de advertencia:

      ![Conexiones con iconos de advertencia](./media/export-from-microsoft-flow-logic-app-template/authorize-connections-vs.png)

   1. En todos los pasos que requieran una conexión autorizada, expanda el paso y seleccione **Agregar nueva**.

      ![Agregar nueva conexión](./media/export-from-microsoft-flow-logic-app-template/add-new-connection-vs.png)

   1. Inicie sesión en cada servicio o proporcione las credenciales necesarias para autorizar la conexión.

   1. Guarde la solución antes de implementar la aplicación lógica.

1. En el Explorador de soluciones, abra el menú contextual del proyecto y seleccione **Implementar** > **Nuevo**. Si se le pide, inicie sesión en su cuenta de Azure.

1. Cuando se le solicite, confirme la suscripción de Azure, el grupo de recursos de Azure y cualquier otro valor que desee usar para la implementación, como el [archivo de parámetros](../azure-resource-manager/templates/parameter-files.md) que se usará para pasar los valores de los parámetros de la plantilla y, después, seleccione **Implementar**.

   ![Confirmar valores de implementación](./media/export-from-microsoft-flow-logic-app-template/confirm-azure-subscription-resource-group-deployment.png)

1. Si el cuadro **Editar parámetros** aparece, especifique el nombre del recurso de la aplicación lógica en Azure y seleccione **Guardar**.  

   ![Editar parámetros de implementación](./media/export-from-microsoft-flow-logic-app-template/edit-parameters-deployment.png)

   Cuando se inicia la implementación, aparece el estado de la implementación de la aplicación en la ventana **Salida** de Visual Studio. Si no aparece el estado, abra la lista **Mostrar salida de** y seleccione el grupo de recursos de Azure. Por ejemplo:

   ![Ventana de salida](./media/export-from-microsoft-flow-logic-app-template/output-window.png)

   Si las conexiones de la aplicación lógica necesitan que el usuario escriba datos, se abrirá una ventana de PowerShell en segundo plano y le solicitará las contraseñas o claves secretas necesarias. Después de escribir esta información, la implementación continúa.

   ![Autenticar conexiones](./media/export-from-microsoft-flow-logic-app-template/logic-apps-powershell-window.png)

   Una vez finalizada la implementación, la aplicación lógica se publica, pero no está activada en Azure Portal.

1. Cuando esté listo para activar la aplicación lógica en Azure Portal, busque la aplicación lógica y ábrala en el Diseñador de aplicación lógica. En el menú de la aplicación lógica, seleccione **Información general** y, después, seleccione **Habilitar**.

1. Para evitar que se ejecuten flujos de trabajo duplicados, asegúrese de desactivar o eliminar el flujo original.

Para más información acerca de estos pasos de implementación, consulte [Inicio rápido: Creación y automatización de tareas, procesos y flujos de trabajo con Azure Logic Apps en Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md#deploy-to-Azure)

## <a name="next-steps"></a>Pasos siguientes

* Más información sobre [conectores para Azure Logic Apps](../connectors/apis-list.md)
* Obtenga más información sobre [Azure Logic Apps](../logic-apps/logic-apps-overview.md).
