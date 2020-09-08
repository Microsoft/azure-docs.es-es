---
title: Supervisar runbooks de Azure Automation con alertas de métricas
description: En este artículo se describe cómo configurar una alerta de métrica en función del estado de finalización de un runbook.
services: automation
ms.date: 08/10/2020
ms.topic: article
ms.openlocfilehash: 8767687f0b72d3469bef570770ac81fa8300097f
ms.sourcegitcommit: 269da970ef8d6fab1e0a5c1a781e4e550ffd2c55
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/10/2020
ms.locfileid: "88055942"
---
# <a name="monitor-runbooks-with-metric-alerts"></a>Supervisión de runbooks con alertas de métricas

En este artículo, aprenderá a crear una [alerta de métrica](../azure-monitor/platform/alerts-metric-overview.md) basada en el estado de finalización del runbook.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en el [Portal de Azure](https://portal.azure.com)

## <a name="create-alert"></a>Crear una alerta

Las alertas le permiten definir una condición para supervisar y una acción para realizar cuando se cumple esa condición.

1. Inicie el servicio Azure Automation en Azure Portal. Para ello, haga clic en **Todos los servicios** y, luego, busque y seleccione **Cuentas de Automation**.

2. En la lista de cuentas de Automation, seleccione la cuenta para la que quiere crear una alerta. 

3. En **Monitoring** (Supervisión), seleccione **Alerts** (Alertas) y, luego, elija **+ New Alert Rule** (+ Nueva regla de alertas). El ámbito del destino ya está definido y asociado con la cuenta de Automation.

### <a name="configure-alert-criteria"></a>Configuración de los criterios de alerta

1. Haga clic en **Select Condition** (Seleccionar condición). Seleccione **Metrics** (Métricas) en **Signal type** (Tipo de señal) y elija **Total Jobs** (Total de trabajos) en la lista.

2. La página **Configurar lógica de señal** es donde se define la lógica que desencadena la alerta. En el gráfico del historial se presentan dos dimensiones, **Nombre de Runbook** y **Estado**. Las dimensiones son propiedades diferentes de una métrica que puede usarse para filtrar los resultados. En **Nombre de Runbook**, seleccione el runbook sobre el que quiera enviar la alerta o déjelo en blanco enviar alertas de todos los runbooks. En **Estado**, seleccione un estado de la lista desplegable que quiera supervisar. Los valores de nombre y estado del runbook que aparecen en la lista desplegable son solo para los trabajos que se ejecutaron en la última semana.

   Si quiere enviar una alerta sobre un estado o un runbook que no se muestran en la lista desplegable, haga clic en **Add custom value** (Agregar valor personalizado) junto a la dimensión. Esta acción abre un cuadro de diálogo que le permite especificar un valor personalizado que no se haya emitido recientemente para esa dimensión. Si escribe un valor que no existe para una propiedad, no se activará la alerta.

   > [!NOTE]
   > Si no especifica un nombre para la dimensión **Runbook Name** (Nombre del runbook) y existen runbooks que cumplen los criterios de estado, incluidos los runbooks ocultos del sistema, recibirá una alerta.

    Por ejemplo, para generar una alerta cuando un runbook devuelva un estado _Failed_ (Error), además de especificar el nombre del runbook, en la dimensión **Status** (Estado), agregue el valor de dimensión personalizado **Failed** (Error).

    :::image type="content" source="./media/automation-alert-metric/specify-dimension-custom-value.png" alt-text="Especificación del valor de dimensión personalizado" border="false":::

3. En **Lógica de alerta**, defina la condición y el umbral de la alerta. Debajo se muestra una vista previa de la condición definida.

4. En **Evaluated based on** (Se evalúa según), seleccione el intervalo de tiempo de la consulta y la frecuencia con la que quiere que se ejecute esa consulta. Por ejemplo, si elige **Over the last 5 minutes** (En los últimos 5 minutos) en **Period** (Período) y **Every 1 Minute** (Cada minuto) en **Frequency** (Frecuencia), la alerta busca el número de runbooks que cumplen los criterios en los últimos 5 minutos. Esta consulta se ejecuta cada minuto y, una vez que los criterios de alerta que ha definido ya no se encuentran en un periodo de 5 minutos, la alerta se resuelve. Cuando termine, haga clic en **Listo**.

   ![Seleccionar un recurso para la alerta](./media/automation-alert-activity-log/configure-signal-logic.png)

### <a name="define-the-action-to-take"></a>Definición de la acción que se realizará

1. En **Action group** (Grupo de acciones), seleccione **Specify action group** (Especificar grupo de acciones). Un grupo de acciones es un conjunto de acciones que puede usar en varias alertas. Por ejemplo, notificaciones por correo electrónico, runbooks, webhooks y muchas más. Para más información sobre los grupos de acciones y los pasos para crear uno que envíe una notificación por correo electrónico, consulte [Creación y administración de grupos de acciones](../azure-monitor/platform/action-groups.md).

### <a name="define-alert-details"></a>Definición de detalles de la alerta

1. En **Alert rule details** (Detalles de la regla de alertas), asigne a la alerta un nombre descriptivo y proporcione una descripción. Establezca la **Gravedad** para que coincida con la condición de la alerta. Hay cinco niveles de gravedad que van de 0 a 5. Las alertas se tratan de igual manera, independientemente de la gravedad; puede hacer coincidir la gravedad con la lógica de negocios.

1. De forma predeterminada, las reglas se habilitan durante la creación, a menos que seleccione **No** para la opción **Enable alert rule upon creation** (Habilitar regla de alertas tras la creación). En el caso de las alertas creadas en un estado deshabilitado, puede habilitarlas en el futuro cuando esté listo. Seleccione **Create alert rule** (Crear regla de alertas) para guardar los cambios.

## <a name="receive-notification"></a>Recepción de una notificación

Cuando se cumplen los criterios de la alerta, el grupo de acciones ejecuta la acción definida. En el ejemplo de este artículo, se envía un correo electrónico. La imagen siguiente es un ejemplo de un correo electrónico que recibirá después de que se desencadene la alerta:

![Alerta de correo electrónico](./media/automation-alert-activity-log/alert-email.png)

Una vez que la métrica ya no esté fuera del umbral definido, se desactivará la alerta y el grupo de acciones ejecutará la acción definida. Si se selecciona un tipo de acción de correo electrónico, se envía un correo electrónico de resolución que indica que esa acción se ha resuelto.

## <a name="next-steps"></a>Pasos siguientes

* Para más información, consulte [Uso de una alerta para desencadenar un runbook de Azure Automation](automation-create-alert-triggered-runbook.md).
