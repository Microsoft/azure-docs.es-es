---
title: Administración de aplicaciones lógicas en Azure Portal
description: Edite, habilite, deshabilite o elimine aplicaciones lógicas mediante Azure Portal.
services: logic-apps
ms.suite: integration
author: lauradolan
ms.author: ladolan
ms.reviewer: estfan, jonfan, logicappspm
ms.topic: article
ms.custom: mvc
ms.date: 07/20/2020
ms.openlocfilehash: d50f577a7170982be004cc8957114f79675fbc6e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87078615"
---
# <a name="manage-logic-apps-in-the-azure-portal"></a>Administración de aplicaciones lógicas en Azure Portal

Puede administrar aplicaciones lógicas mediante [Azure Portal](https://portal.azure.com) o [Visual Studio](manage-logic-apps-with-visual-studio.md). En este artículo se muestra cómo editar, deshabilitar, habilitar o eliminar aplicaciones lógicas en Azure Portal. Si no está familiarizado con Azure Logic Apps, consulte [¿Qué es Azure Logic Apps?](logic-apps-overview.md)

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Elija una aplicación lógica existente. Para obtener información sobre cómo crear una aplicación lógica mediante Azure Portal, consulte [Inicio rápido: Creación del primer flujo de trabajo mediante Azure Logic Apps: Azure Portal](quickstart-create-first-logic-app-workflow.md).

<a name="find-logic-app"></a>

## <a name="find-your-logic-apps"></a>Búsqueda de las aplicaciones lógicas

Para buscar y abrir la aplicación lógica, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con su cuenta de Azure.

1. En la barra de búsqueda de Azure, escriba `logic apps` y seleccione **Logic Apps**.

   ![Captura de pantalla del menú de Azure Portal, donde se ha buscado y seleccionado Logic Apps en la barra de búsqueda.](./media/manage-logic-apps-with-azure-portal/find-select-logic-apps.png)

1. En la página **Logic Apps**, busque y seleccione la aplicación lógica que desea administrar.

   Una vez que se abra el panel **Información general**, puede filtrar la lista que aparece en la página **Logic Apps** de las siguientes maneras:

   * Buscar aplicaciones lógicas por nombre
   * Filtrar las aplicaciones lógicas por suscripción, grupo de recursos, ubicación y etiquetas
   * Agrupar las aplicaciones lógicas por grupo de recursos, tipo, suscripción y ubicación

## <a name="view-logic-app-properties"></a>Visualización de las propiedades de la aplicación lógica

1. En Azure Portal, [busque y abra la aplicación lógica](#find-logic-app).

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Propiedades**.

1. En el panel **Propiedades**, puede ver y copiar la siguiente información sobre la aplicación lógica:

   * **Nombre**
   * **Identificador del recurso**
   * **Grupos de recursos**
   * **Ubicación**
   * **Tipo** 
   * **Subscription Name**
   * **Subscription ID**
   * **Punto de conexión de acceso**
   * **Direcciones IP salientes en tiempo de ejecución**
   * **Direcciones IP de puntos de conexión de acceso**
   * **Direcciones IP salientes del conector**

## <a name="disable-or-enable-logic-apps"></a>Deshabilitación o habilitación de la aplicaciones lógicas

Puede habilitar o deshabilitar una [sola aplicación lógica](#disable-enable-single-logic-app) o [varias aplicaciones lógicas a la vez](#disable-or-enable-multiple-logic-apps) en el Azure Portal. También puede [habilitar o deshabilitar aplicaciones lógicas en Visual Studio](manage-logic-apps-with-visual-studio.md#disable-or-enable-logic-app).

La deshabilitación de la aplicación lógica afecta a las instancias de flujo de trabajo y se ejecuta de estas maneras:

* Todas las ejecuciones en curso y pendientes continuarán hasta que finalicen. En función del número de ejecuciones, este proceso puede tardar algún tiempo.

* El motor de Logic Apps no creará ni ejecutará nuevas instancias de flujo de trabajo.

* El desencadenador no se activará la próxima vez que se cumplan sus condiciones.

* El estado del desencadenador recuerda el punto en el que se detuvo la aplicación lógica. Por lo tanto, si vuelve a habilitar la aplicación lógica, el desencadenador se activará para todos los elementos no procesados desde la última ejecución.

  Para que la aplicación lógica no se active para los elementos no procesados desde la última ejecución, borre el estado del desencadenador antes de volver a habilitar la aplicación lógica:

  1. En Azure Portal, [busque y abra la aplicación lógica](#find-logic-app).

  1. Edite cualquier parte del desencadenador de la aplicación lógica.

  1. Guarde los cambios. Este paso restablece el estado actual del desencadenador.

  1. [Vuelva a habilitar la aplicación lógica](#disable-enable-single-logic-app).

<a name="disable-enable-single-logic-app"></a>

### <a name="disable-or-enable-single-logic-app"></a>Deshabilitación o habilitación de una sola aplicación lógica

1. En Azure Portal, [busque y abra la aplicación lógica](#find-logic-app).

1. En el menú de la aplicación lógica, seleccione **Información general**. Elija una de estas opciones:

   * En la barra de herramientas, seleccione **Deshabilitar**.

     ![Captura de pantalla de la barra de herramientas de la aplicación lógica, que muestra el botón Deshabilitar seleccionado.](./media/manage-logic-apps-with-azure-portal/disable-single-logic-app.png)

     Si la aplicación lógica ya está deshabilitada, solo verá la opción **Habilitar**.

   * En la barra de herramientas, seleccione **Enable** (Habilitar).

     ![Captura de pantalla de la barra de herramientas de la aplicación lógica, que muestra el botón Habilitar seleccionado.](./media/manage-logic-apps-with-azure-portal/enable-single-logic-app.png)

     Si la aplicación lógica ya está habilitada, solo verá la opción **Deshabilitar**. 

   En Azure Portal se muestra una notificación en la barra de herramientas principal de Azure que confirma si la operación se realizó correctamente o no.

   ![Captura de pantalla de Azure Portal, que muestra la notificación para confirmar el estado de la operación.](./media/manage-logic-apps-with-azure-portal/operation-confirmation-notification.png)

<a name="disable-or-enable-multiple-logic-apps"></a>

### <a name="disable-or-enable-multiple-logic-apps"></a>Deshabilitación o habilitación de varias aplicaciones lógicas

1. En el Azure Portal, [busque las aplicaciones lógicas](#find-logic-app) que desea deshabilitar o habilitar.

1. Para comprobar si una aplicación lógica está habilitada o deshabilitada actualmente, en la página **Logic Apps**, consulte la columna **Estado** de esa aplicación lógica. 

   ![Captura de pantalla de la página de Logic Apps de Azure Portal, que muestra la lista de aplicaciones lógicas ordenadas por la columna de estado.](./media/manage-logic-apps-with-azure-portal/view-logic-app-status.png)

   Si la columna **Estado** no está visible, en la barra de herramientas de **Logic Apps**, seleccione **Probar la versión preliminar**.

   ![Captura de pantalla de la página de Logic Apps de Azure Portal, que muestra el botón Try preview (Probar la versión preliminar) seleccionado.](./media/manage-logic-apps-with-azure-portal/select-try-preview.png)

1. En la columna de casillas, seleccione las aplicaciones lógicas que quiere deshabilitar o habilitar. En la barra de herramientas, seleccione **Deshabilitar** o **Habilitar**.

   ![Captura de pantalla de la página de Logic Apps de Azure Portal, que muestra los botones Habilitar y Deshabilitar de varias aplicaciones lógicas.](./media/manage-logic-apps-with-azure-portal/enable-disable-multiple-logic-apps.png)

1. Cuando aparezca el cuadro de confirmación, seleccione **Sí** para continuar.

   En Azure Portal se muestra una notificación en la barra de herramientas principal de Azure que confirma si la operación se realizó correctamente o no.

## <a name="delete-logic-apps"></a>Eliminación de aplicaciones lógicas

Puede [eliminar una sola aplicación lógica](#delete-single-logic-app) o [eliminar varias aplicaciones lógicas a la vez](#delete-multiple-logic-apps) en el Azure Portal. También puede [eliminar la aplicación lógica en Visual Studio](manage-logic-apps-with-visual-studio.md#delete-your-logic-app).

La eliminación de la aplicación lógica afecta a las instancias de flujo de trabajo de estas maneras:

* Todas las ejecuciones en curso y pendientes continuarán hasta que finalicen. En función del número de ejecuciones, este proceso puede tardar algún tiempo.

* El motor de Logic Apps no creará ni ejecutará nuevas instancias de flujo de trabajo.

> [!NOTE]
> Si elimina una aplicación lógica secundaria y vuelve a crearla, debe guardar de nuevo la aplicación lógica principal. La aplicación secundaria que ha vuelto a crear tendrá metadatos diferentes.
> Si no se reguarda la aplicación lógica principal después de volver a crear su elemento secundario, las llamadas a la aplicación lógica secundaria producirán un error de que no hay autorización. Este comportamiento se aplica a las aplicaciones lógicas de elementos primarios y secundarios, por ejemplo, las que usan artefactos en las cuentas de integración o llaman a funciones de Azure.

<a name="delete-single-logic-app"></a>

### <a name="delete-single-logic-app"></a>Eliminación de una sola aplicación lógica

1. En Azure Portal, [busque y abra la aplicación lógica](#find-logic-app).

1. En el menú de la aplicación lógica, seleccione **Información general**. En la barra de herramientas de la aplicación lógica, seleccione **Eliminar**.

   ![Captura de pantalla de la barra de herramientas de la aplicación lógica, que muestra el botón Eliminar seleccionado.](./media/manage-logic-apps-with-azure-portal/delete-single-logic-app.png)

1. Cuando aparezca el cuadro de confirmación, escriba el nombre de la aplicación lógica y seleccione **Eliminar**.

   ![Captura de pantalla del aviso de Logic Apps para confirmar la eliminación de una sola aplicación lógica.](./media/manage-logic-apps-with-azure-portal/delete-confirmation-single-logic-app.png)

   En Azure Portal se muestra una notificación en la barra de herramientas principal de Azure que confirma si la operación se realizó correctamente o no.

<a name="delete-multiple-logic-apps"></a>

### <a name="delete-multiple-logic-apps"></a>Eliminación de varias aplicaciones lógicas

1. En el Azure Portal, [busque las aplicaciones lógicas que desea eliminar](#find-logic-app).

1. En la columna de casillas, seleccione las aplicaciones lógicas que quiere eliminar. En la barra de herramientas, seleccione **Eliminar**.

   ![Captura de pantalla de la página de Logic Apps, que muestra varias aplicaciones lógicas en una lista seleccionada para su eliminación.](./media/manage-logic-apps-with-azure-portal/delete-multiple-logic-apps.png)

1. Cuando aparezca el cuadro de confirmación, escriba `yes` y seleccione **Eliminar**.

   ![Captura de pantalla del aviso de Logic Apps para confirmar la eliminación de varias aplicaciones lógicas.](./media/manage-logic-apps-with-azure-portal/delete-confirmation-multiple-logic-apps.png)

   En Azure Portal se muestra una notificación en la barra de herramientas principal de Azure que confirma si la operación se realizó correctamente o no.

<a name="manage-logic-app-versions"></a>

## <a name="manage-logic-app-versions"></a>Administración de versiones de aplicaciones lógicas

Puede usar Azure Portal para el control de versiones de las aplicaciones lógicas. Puede encontrar el historial de versiones de una aplicación lógica y restaurar versiones anteriores.

<a name="find-version-history"></a>

### <a name="find-and-view-previous-versions"></a>Búsqueda y visualización de versiones anteriores

1. En Azure Portal, [busque la aplicación lógica que desea administrar](#find-logic-app).

1. En el menú de la aplicación lógica, en **Herramientas de desarrollo**, seleccione **Versiones**.

   ![Captura de pantalla de la aplicación lógica en Azure Portal, que muestra la selección de la página Versiones en las herramientas de desarrollo.](./media/manage-logic-apps-with-azure-portal/logic-apps-menu-versions.png)

1. En la lista seleccione la **versión** de la aplicación lógica que va a administrar. Puede especificar el identificador de la **versión** en la barra de búsqueda para filtrar la lista.

1. En la página **Historial de versiones**, verá los detalles de la versión anterior en modo de solo lectura. Puede seleccionar entre los modos **Diseñador** de Logic Apps y **vista Código**.

   ![Captura de pantalla de la página History version (Historial de versiones) de Logic Apps, que muestra las opciones de vista de código y vista de diseñador.](./media/manage-logic-apps-with-azure-portal/history-version.png)

<a name="promote-previous-versions"></a>

### <a name="promote-previous-versions"></a>Restauración de versiones anteriores

1. En el historial de versiones de la aplicación lógica, [busque y seleccione la versión que desea promocionar](#find-version-history).

1. En la página **Historial de versiones**, seleccione **Promover**.

   ![Captura de pantalla del historial de versiones de la aplicación lógica, que muestra el botón para promocionar una versión anterior.](./media/manage-logic-apps-with-azure-portal/promote-button.png)

1. En la página **Diseñador de Logic Apps** que se abre, edite la versión que va a promocionar según sea necesario. Puede cambiar entre los modos **Diseñador** y **vista de Código**. También puede actualizar **Parámetros**, **Plantillas** y **Conectores**.

   ![Captura de pantalla del Diseñador de aplicaciones lógicas, que muestra el botón para promocionar una versión anterior de una aplicación lógica.](./media/manage-logic-apps-with-azure-portal/promote-page.png)

1. Para guardar las actualizaciones y finalizar la promoción de la versión anterior, seleccione **Guardar**. (Si desea cancelar los cambios, seleccione **Descartar**). 

   Cuando vuelva a [visualizar el historial de versiones de la aplicación lógica](#find-version-history), la versión promocionada aparece al principio de la lista con un nuevo identificador.

## <a name="next-steps"></a>Pasos siguientes

* [Supervisión de aplicaciones lógicas](monitor-logic-apps.md)
