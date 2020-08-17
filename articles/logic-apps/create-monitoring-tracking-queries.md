---
title: Visualización y creación de consultas para aplicaciones lógicas en registros de Azure Monitor
description: Visualización y creación de consultas en registros de Azure Monitor para Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: divswa, logicappspm
ms.topic: article
ms.date: 01/30/2020
ms.openlocfilehash: cb1af437fc663fcb95c768ec295862209a26064e
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87090406"
---
# <a name="view-and-create-queries-for-monitoring-and-tracking-in-azure-monitor-logs-for-azure-logic-apps"></a>Visualización y creación de consultas para supervisar y realizar un seguimiento de los registros de Azure Monitor para Azure Logic Apps

Puede ver las consultas subyacentes que producen los resultados en los [registros de Azure Monitor](../azure-monitor/log-query/log-query-overview.md) y crear consultas que filtren los resultados en función de sus criterios específicos. Por ejemplo, puede encontrar mensajes según un número de control de intercambio específico. Las consultas usan el [lenguaje de consulta Kusto](https://aka.ms/LogAnalyticsLanguageReference), que puede modificar si quiere ver otros resultados. Para más información, consulte las [consultas de registro de Azure Monitor](../azure-monitor/log-query/query-language.md).

## <a name="prerequisites"></a>Prerrequisitos

* Un área de trabajo de Log Analytics. Si no tiene un área de trabajo de Log Analytics, aprenda a [crear un área de trabajo de Log Analytics](../azure-monitor/learn/quick-create-workspace.md).

* Una aplicación lógica que se configura con el registro de Azure Monitor y que envía esa información a un área de trabajo de Log Analytics. Aprenda a [configurar registros de Azure Monitor para la aplicación lógica](../logic-apps/monitor-logic-apps.md).

* Si usa una cuenta de integración, asegúrese de que ha configurado la cuenta con el registro de Azure Monitor para enviar esa información a un área de trabajo de Log Analytics. Más información sobre cómo [configurar el registro de Azure Monitor para la cuenta de integración](../logic-apps/monitor-b2b-messages-log-analytics.md).

## <a name="view-queries-behind-results"></a>Visualización de consultas detrás de los resultados

Para ver o editar la consulta que genera los resultados en el resumen del área de trabajo, siga estos pasos:

1. En la parte inferior de cualquier página de resultados, seleccione **Ver todo**.

   ![Ver todos los resultados](./media/create-monitoring-tracking-queries/logic-app-see-all.png)

   Se abre la página de registros y muestra la consulta que se encuentra detrás de la página de resultados anterior.

   ![Página Registros: vista de consulta](./media/create-monitoring-tracking-queries/view-query-behind-results.png)

1. En la página **Registros**, puede seleccionar estas opciones:

   * Para ver los resultados de la consulta en forma de tabla, vaya al editor de consultas y seleccione **Tabla**.

   * Si quiere modificar la consulta, actualice la cadena de consulta y seleccione **Ejecutar** para ver los resultados en la tabla.

## <a name="create-your-own-query"></a>Creación de su propia consulta

Para buscar o filtrar los resultados en función de propiedades o valores específicos, puede crear su propia consulta a partir de una consulta vacía o usar una consulta existente. Para más información, consulte [Introducción a las consultas de registro en Azure Monitor](../azure-monitor/log-query/get-started-queries.md).

1. En [Azure Portal](https://portal.azure.com), busque y seleccione el área de trabajo de Log Analytics.

1. En el menú del área de trabajo, en **General**, seleccione **Registros**.

1. Comience con una consulta vacía o con cualquier consulta existente disponible.

   * Para comprobar si alguna de las consultas existentes está disponible, en la barra de herramientas de consulta, seleccione **Consultas de ejemplo** > **Historia**, que muestra consultas de ejecuciones de consulta anteriores o seleccione **Explorador de consultas**, que muestra consultas precompiladas.

     Por ejemplo, la solución Logic Apps B2B proporciona estas consultas precompiladas:

     ![Comienzo con las consultas precompiladas de la solución "Logic Apps B2B"](./media/create-monitoring-tracking-queries/b2b-prebuilt-queries.png)

   * Para comenzar con una consulta vacía, en el editor de consultas, escriba el [lenguaje de consulta Kusto](../azure-monitor/log-query/query-language.md) de la consulta.

     ![Comienzo con una consulta vacía](./media/create-monitoring-tracking-queries/create-query-from-blank.png)

## <a name="next-steps"></a>Pasos siguientes

* [Esquemas de seguimiento de AS2](../logic-apps/logic-apps-track-integration-account-as2-tracking-schemas.md)
* [Esquemas de seguimiento de X12](../logic-apps/logic-apps-track-integration-account-x12-tracking-schema.md)
* [Esquemas de seguimiento personalizados](../logic-apps/logic-apps-track-integration-account-custom-tracking-schema.md)
