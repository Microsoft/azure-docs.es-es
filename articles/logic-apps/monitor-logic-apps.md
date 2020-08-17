---
title: Supervisión del estado, visualización del historial y configuración de alertas
description: Solución de problemas con las aplicaciones lógicas mediante la comprobación del estado de ejecución, la revisión del historial de los desencadenadores y la habilitación de alertas en Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: divswa, logicappspm
ms.topic: article
ms.date: 05/04/2020
ms.openlocfilehash: 66796a819c0ca7e114d82210a988fc7e13003941
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87078197"
---
# <a name="monitor-run-status-review-trigger-history-and-set-up-alerts-for-azure-logic-apps"></a>Supervisión del estado de ejecución, revisión del historial de los desencadenadores y configuración de alertas en Azure Logic Apps

Después de [crear y ejecutar una aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md), puede comprobar su estado de ejecución, el [historial de ejecuciones](#review-runs-history), el [historial de los desencadenadores](#review-trigger-history) y el rendimiento. Para recibir notificaciones sobre errores u otros posibles problemas, configure [alertas](#add-azure-alerts). Por ejemplo, puede crear una alerta que detecte "cuando se produzcan errores en más de cinco ejecuciones en una hora".

Para la supervisión de eventos en tiempo real y una depuración más rica, configure el registro de diagnóstico de la aplicación lógica mediante los [registros de Azure Monitor](../azure-monitor/overview.md). Este servicio de Azure le ayuda a supervisar los entornos locales y en la nube para que pueda mantener más fácilmente su disponibilidad y rendimiento. De este modo, puede buscar y ver eventos, como los de desencadenador, de ejecución y de acción. Al almacenarla en [registros de Azure Monitor](../azure-monitor/platform/data-platform-logs.md), puede crear [consultas de registro](../azure-monitor/log-query/log-query-overview.md) que le ayuden a buscar y analizar esta información. También puede usar estos datos de diagnóstico con otros servicios de Azure, como Azure Storage y Azure Event Hubs. Para más información, consulte [Supervisión de aplicaciones lógicas con Azure Monitor](../logic-apps/monitor-logic-apps-log-analytics.md).

> [!NOTE]
> Si las aplicaciones lógicas se ejecutan en un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) que se creó para usar un [punto de conexión de acceso interno](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#endpoint-access), puede ver las entradas y salidas del historial de ejecuciones de la aplicación lógica y acceder a ellas *solo desde dentro de la red virtual*. Asegúrese de tener conectividad de red entre los puntos de conexión privados y el equipo desde el que desea acceder al historial de ejecuciones. Por ejemplo, el equipo cliente puede existir dentro de la red virtual del ISE o en una red virtual que esté conectada a la red virtual del ISE, por ejemplo, a través de emparejamiento o de una red privada virtual. Para obtener más información, consulte [Acceso al punto de conexión del ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md#endpoint-access). 

<a name="review-runs-history"></a>

## <a name="review-runs-history"></a>Revisión del historial de ejecuciones

Cada vez que el desencadenador se activa por un elemento o un evento, el motor de Logic Apps crea y ejecuta una instancia distinta del flujo de trabajo para cada uno de ellos. De forma predeterminada, las instancias del flujo de trabajo se ejecutan en paralelo para que ningún flujo de trabajo tenga que esperar para iniciar una ejecución. Puede revisar lo que ha ocurrido durante la ejecución, incluido el estado de cada paso del flujo de trabajo, además de las entradas y salidas de cada paso.

1. En [Azure Portal](https://portal.azure.com), busque y abra la aplicación lógica en el Diseñador de aplicación lógica.

   Para buscar la aplicación lógica, en el cuadro de búsqueda principal de Azure, escriba `logic apps` y seleccione **Logic Apps**.

   ![Búsqueda y selección del servicio "Logic Apps"](./media/monitor-logic-apps/find-your-logic-app.png)

   En Azure Portal se muestran todas las aplicaciones lógicas asociadas a las suscripciones de Azure. Puede filtrar esta lista por nombre, suscripción, grupo de recursos, ubicación, etc.

   ![Visualización de las aplicaciones lógicas asociadas a las suscripciones](./media/monitor-logic-apps/logic-apps-list-in-subscription.png)

1. Seleccione la aplicación lógica y, a continuación, seleccione **Introducción**.

   En el panel de información general, en **Runs history** (Historial de ejecuciones), aparecen todas las ejecuciones pasadas, actuales y en espera de la aplicación lógica. Si en la lista aparece una gran cantidad de ejecuciones y no se encuentra la entrada deseada, intente filtrar la lista. Si no encuentra los datos previstos, elija **Actualizar** en la barra de herramientas.

   ![Introducción, historial de ejecuciones y otros datos de la aplicación lógica](./media/monitor-logic-apps/overview-pane-logic-app-details-run-history.png)

   Estos son los estados posibles de una ejecución de aplicación lógica:

   | Status | Descripción |
   |--------|-------------|
   | **Cancelado** | El flujo de trabajo se estaba ejecutando pero recibió una solicitud de cancelación. |
   | **Erróneo** | Error en al menos una acción y ninguna de las acciones posteriores en el flujo de trabajo se ha configurado para controlar el error. |
   | **Ejecución** | El flujo de trabajo se ejecuta actualmente. <p>Este estado también puede ocurrir en flujos de trabajo con limitaciones o a causa del plan de precios actual. Para más información, consulte [los límites de acción en la página de precios](https://azure.microsoft.com/pricing/details/logic-apps/). Si se configura un [registro de diagnóstico](../logic-apps/monitor-logic-apps.md), se puede obtener información sobre los eventos de limitación que se produzcan. |
   | **Correcto** | Todas las acciones correctas. <p>**Nota**: Aunque se hayan producido errores en una acción específica, una acción posterior del flujo de trabajo controló ese error. |
   | **En espera** | El flujo de trabajo no se ha iniciado o está en pausa, por ejemplo, debido a un flujo de trabajo anterior aún en ejecución. |
   |||

1. Para revisar los pasos y otros datos de una ejecución concreta, selecciónela en **Runs history** (Historial de ejecuciones).

   ![Selección de una ejecución concreta para revisar](./media/monitor-logic-apps/select-specific-logic-app-run.png)

   En el panel **Ejecución de aplicación lógica** se muestra cada paso de la ejecución seleccionada, el estado de ejecución de estos y el tiempo que se tarda en ejecutar cada uno de ellos, por ejemplo:

   ![Acciones de la ejecución concreta](./media/monitor-logic-apps/logic-app-run-pane.png)

   Para ver esta información en forma de lista, en la barra de herramientas de **Ejecución de aplicación lógica**, seleccione **Detalles de ejecución**.

   ![Selección de "Detalles de ejecución" en la barra de herramientas](./media/monitor-logic-apps/select-run-details-on-toolbar.png)

   En la vista Detalles de ejecución se muestra cada paso, su estado y otros datos.

   ![Revisión de los detalles de cada paso de la ejecución](./media/monitor-logic-apps/review-logic-app-run-details.png)

   Por ejemplo, puede obtener la propiedad **Correlation ID** de la ejecución, que podría necesitar al usar la [API REST de Logic Apps](/rest/api/logic).

1. Para más información sobre un paso específico, seleccione cualquiera de las dos opciones:

   * En el panel **Ejecución de aplicación lógica**, seleccione el paso para que la forma se expanda. Podrá ver información como las entradas, las salidas y los errores que se produjeran en ese paso, por ejemplo:

     ![Visualización de un paso con error en el panel Ejecución de aplicación lógica](./media/monitor-logic-apps/specific-step-inputs-outputs-errors.png)

   * Seleccione el paso que desee en el panel **Logic app run details** (Detalles de ejecución de la aplicación lógica).

     ![Visualización de un paso con error en el panel de detalles de la ejecución](./media/monitor-logic-apps/select-failed-step-in-failed-run.png)

     Podrá ver información como las entradas y las salidas de ese paso, por ejemplo:

   > [!NOTE]
   > Todos los eventos y los detalles de runtime se cifran en el servicio Logic Apps. Solo se descifran cuando un usuario solicita ver esos datos. Puede [ocultar las entradas y salidas en el historial de ejecución](../logic-apps/logic-apps-securing-a-logic-app.md#obfuscate) o controlar el acceso de los usuarios a esta información mediante el [control de acceso basado en roles de Azure (RBAC de Azure)](../role-based-access-control/overview.md).

<a name="review-trigger-history"></a>

## <a name="review-trigger-history"></a>Revisión del historial de desencadenadores

Cada ejecución de aplicación lógica se inicia con un desencadenador. En este historial de desencadenadores se muestran todos los intentos de desencadenamiento realizados por la aplicación lógica e información sobre las entradas y salidas de cada intento.

1. En [Azure Portal](https://portal.azure.com), busque y abra la aplicación lógica en el Diseñador de aplicación lógica.

   Para buscar la aplicación lógica, en el cuadro de búsqueda principal de Azure, escriba `logic apps` y seleccione **Logic Apps**.

   ![Búsqueda y selección del servicio "Logic Apps"](./media/monitor-logic-apps/find-your-logic-app.png)

   En Azure Portal se muestran todas las aplicaciones lógicas asociadas a las suscripciones de Azure. Puede filtrar esta lista por nombre, suscripción, grupo de recursos, ubicación, etc.

   ![Visualización de las aplicaciones lógicas asociadas a las suscripciones](./media/monitor-logic-apps/logic-apps-list-in-subscription.png)

1. Seleccione la aplicación lógica y, a continuación, seleccione **Introducción**.

1. En el menú de la aplicación lógica, seleccione **Introducción**. En **Evaluación** en la sección **Resumen**, seleccione **Ver el historial de desencadenadores**.

   ![Visualización del historial de desencadenadores de la aplicación lógica](./media/monitor-logic-apps/overview-pane-logic-app-details-trigger-history.png)

   En el panel del historial de desencadenadores se muestran todos los intentos de desencadenamiento que la aplicación lógica ha realizado. Cada vez que el desencadenador se activa por un elemento o un evento, el motor de Logic Apps crea una instancia de aplicación lógica independiente que ejecuta el flujo de trabajo. De forma predeterminada, cada instancia se ejecuta en paralelo para que ningún flujo de trabajo tenga que esperar antes de iniciar una ejecución. Por lo tanto, si la aplicación lógica se desencadena en varios elementos al mismo tiempo, se mostrará una entrada de desencadenador con la misma fecha y hora para cada elemento.

   ![Varios intentos de desencadenamiento para elementos diferentes](./media/monitor-logic-apps/logic-app-trigger-history.png)

   Estos son los estados posibles de un intento de desencadenador:

   | Status | Descripción |
   |--------|-------------|
   | **Erróneo** | Se produjo un error. Para revisar los mensajes de error generado para un desencadenador con error, seleccione el intento de desencadenador y elija **Salidas**. Por ejemplo, podría encontrar entradas que no son válidas. |
   | **Omitido** | El desencadenador comprueba el punto de conexión, pero no ha encontrado datos. |
   | **Correcto** | El desencadenador comprueba el punto de conexión y encuentra datos disponibles. Por lo general, un estado "activado" también aparece junto a este estado. Si no es así, la definición del desencadenador puede tener una condición o un comando `SplitOn` que no se ha cumplido. <p>Este estado puede aplicarse a un desencadenador manual, uno de periodicidad o uno de sondeo. Un desencadenador se puede ejecutar correctamente, pero la misma ejecución podría fallar cuando las acciones generan errores no controlados. |
   |||

   > [!TIP]
   > Puede volver a comprobar el desencadenador sin esperar a la siguiente repetición. En la barra de herramientas de información general, seleccione **Ejecutar desencadenador** y el desencadenador para forzar una comprobación. O bien, seleccione **Ejecutar** en la barra de herramientas del Diseñador de aplicaciones lógicas.

1. Para ver más información sobre un intento de desencadenamiento específico, seleccione el evento de desencadenador en el panel de desencadenamiento. Si la lista muestra muchos intentos y no encuentra la entrada que desea, pruebe a filtrar la lista. Si no encuentra los datos previstos, elija **Actualizar** en la barra de herramientas.

   ![Visualización de un intento de desencadenamiento específico](./media/monitor-logic-apps/select-trigger-event-for-review.png)

   Ahora puede revisar la información sobre el evento de desencadenador seleccionado, por ejemplo:

   ![Visualización de la información de un desencadenador específico](./media/monitor-logic-apps/view-specific-trigger-details.png)

<a name="add-azure-alerts"></a>

## <a name="set-up-monitoring-alerts"></a>Configuración de alertas de supervisión

Para obtener alertas de métricas o umbrales superados concretos de la aplicación lógica, configure las [alertas en Azure Monitor](../azure-monitor/platform/alerts-overview.md). Más información sobre [métricas de Azure](../azure-monitor/platform/data-platform.md). Para configurar alertas sin [Azure Monitor](../azure-monitor/log-query/log-query-overview.md), siga estos pasos.

1. En el menú de la aplicación lógica, en **Supervisión**, seleccione **Alertas** > **Nueva regla de alertas**.

   ![Adición de una alerta de la aplicación lógica](./media/monitor-logic-apps/add-new-alert-rule.png)

1. En **Recurso** en el panel **Crear regla**, seleccione la aplicación lógica si aún no está seleccionada. En **Condición**, seleccione **Agregar** para poder definir la condición que desencadena la alerta.

   ![Incorporación de una condición para la regla](./media/monitor-logic-apps/add-condition-for-rule.png)

1. En el panel **Configurar lógica de señal**, busque y seleccione la señal para la que desea obtener una alerta. Puede usar el cuadro de búsqueda u ordenar las señales alfabéticamente. Para esto último, seleccione el encabezado de columna **Nombre de señal**.

   Por ejemplo, si desea enviar una alerta cuando se produzca un error en un desencadenador, siga estos pasos:

   1. En la columna **Nombre de señal**, busque y seleccione **Desencadenadores con error**.

      ![Selección de la señal para crear una alerta](./media/monitor-logic-apps/find-and-select-signal.png)

   1. En **Lógica de alerta** en el panel de información que se abre para la señal seleccionada, configure la condición, por ejemplo:

   1. Para **Operador**, seleccione **Es mayor o igual que**.

   1. Para **Tipo de agregación**, seleccione **Recuento**.

   1. Para **Valor del umbral**, escriba `1`.

   1. En **Vista previa de condición**, confirme que la condición parece correcta.

   1. En **Se evaluó basándose en**, configure el intervalo y la frecuencia de ejecución de la regla de alerta. En **Granularidad de agregación (período)** , seleccione el período de agrupación de los datos. Para **Frecuencia de evaluación**, seleccione la frecuencia con la que desea que se compruebe la condición.

   1. Cuando lo tenga todo preparado, seleccione **Listo**.

   Esta es la condición finalizada:

   ![Configuración de la condición para la alerta](./media/monitor-logic-apps/set-up-condition-for-alert.png)

   En la página **Crear regla** ahora se muestra la condición que ha creado y el costo de ejecución de esa alerta.

   ![Nueva alerta en la página "Crear regla"](./media/monitor-logic-apps/finished-alert-condition-cost.png)

1. Especifique un nombre, una descripción (opcional) y el nivel de gravedad de la alerta. Deje activada la opción **Habilitar regla tras la creación** o desactívela hasta que esté listo para habilitar la regla.

1. Seleccione **Crear regla de alertas** cuando haya terminado.

> [!TIP]
> Para ejecutar una aplicación lógica desde una alerta, puede incluir el [desencadenador de solicitud](../connectors/connectors-native-reqres.md) en el flujo de trabajo, lo que permite realizar tareas como estos ejemplos:
> 
> * [Publicar en Slack](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-slack-with-logic-app)
> * [Enviar un texto](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-text-message-with-logic-app)
> * [Agregar un mensaje a una cola](https://github.com/Azure/azure-quickstart-templates/tree/master/201-alert-to-queue-with-logic-app)

## <a name="next-steps"></a>Pasos siguientes

* [Supervisión de aplicaciones lógicas con Azure Monitor](../logic-apps/monitor-logic-apps-log-analytics.md)
