---
title: Cómo crear alertas para Azure Automation Update Management
description: En este artículo se explica cómo configurar alertas de Azure para notificar el estado de las implementaciones o evaluaciones de las actualizaciones.
services: automation
ms.subservice: update-management
ms.date: 07/28/2020
ms.topic: conceptual
ms.openlocfilehash: 2c39a07ceac4d36bf3ef7394927589b53da7d789
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87449818"
---
# <a name="how-to-create-alerts-for-update-management"></a>Cómo crear alertas para Update Management

Las alertas de Azure le notifican proactivamente los resultados de los trabajos de runbook, los problemas de estado del servicio u otros escenarios relacionados con su cuenta de Automation. Azure Automation no incluye las reglas de alerta preconfiguradas, pero puede crear las suyas propias en función de los datos que recopila. En este artículo se proporcionan instrucciones sobre cómo crear reglas de alertas mediante las métricas incluidas en Update Management.

## <a name="available-metrics"></a>Métricas disponibles

Azure Automation crea dos métricas de plataforma distintas relacionadas con Update Management que se recopilan y reenvían a Azure Monitor. Estas métricas están disponibles para su análisis mediante el [Explorador de métricas](../../azure-monitor/platform/metrics-charts.md) y para enviar alertas con una [regla de alertas de métricas](../../azure-monitor/platform/alerts-metric.md).

Las dos métricas emitidas son:

* Ejecuciones de máquina de implementación de actualizaciones totales
* Ejecuciones de implementación de actualizaciones totales

Cuando se usan en las alertas, ambas métricas admiten dimensiones que contienen información adicional que le permitirá definir el ámbito de la alerta en función de un detalle específico de la implementación de actualizaciones. En la tabla siguiente se muestran los detalles sobre la métrica y las dimensiones disponibles al configurar una alerta.

|Nombre de señal|Dimensions|Descripción
|---|---|---|
|`Total Update Deployment Runs`|- Nombre de implementación de actualizaciones<br>- Estado | Muestra una alerta sobre el estado general de una implementación de actualizaciones.|
|`Total Update Deployment Machine Runs`|- Nombre de implementación de actualizaciones</br>- Estado</br>- Equipo de destino</br>- Identificador de ejecución de implementación de actualizaciones    |Muestra una alerta sobre el estado de una implementación de actualizaciones dirigida a máquinas específicas.|

## <a name="create-alert"></a>Crear una alerta

Realice los pasos siguientes para configurar alertas que le permitan saber el estado de una implementación de actualizaciones. Si no está familiarizado con las alertas de Azure, consulte la [información general sobre alertas de Azure](../../azure-monitor/platform/alerts-overview.md).

1. En la cuenta de Automation, seleccione **Alertas** en la opción de **supervisión** y, a continuación, seleccione **Nueva regla de alerta**.

2. En la página de **creación de reglas de alertas**, su cuenta de Automation aparece seleccionada como recurso. Si quiere cambiar esto, seleccione la opción para **editar recursos**.

3. En la página para seleccionar un recurso, elija **Cuentas de Automation** en la lista desplegable **Filtrar por tipo de recurso**.

4. Seleccione la cuenta de Automation que quiera usar y, a continuación, haga clic en **Listo**.

5. Seleccione **Agregar condición** para elegir la señal adecuada para sus requisitos.

6. En el caso de las dimensiones, seleccione un valor válido de la lista. Si el valor que quiere usar no está en la lista, seleccione el signo **\+** situado junto a la dimensión y escriba el nombre personalizado. A continuación, seleccione el valor que quiere buscar. Si quiere seleccionar todos los valores de una dimensión, haga clic en el botón **Seleccionar \*** . Si no elige ningún valor para una dimensión, Update Management no tendrá en cuenta esa dimensión.

    ![Configuración de la lógica de señal](./media/update-mgmt-manage-updates-for-vm/signal-logic.png)

7. En **Lógica de alerta**, escriba los valores en los campos **Agregación de tiempo** y **Umbral**, y seleccione **Listo**.

8. En el siguiente panel, escriba un nombre y una descripción para la alerta.

9. En el campo **Gravedad**, seleccione **Información (gravedad 2)** para una ejecución correcta, o bien **Información (gravedad 1)** para una ejecución con errores.

    ![Configuración de la lógica de señal](./media/update-mgmt-manage-updates-for-vm/define-alert-details.png)

10. Seleccione **Sí** para habilitar la regla de alerta.

## <a name="configure-action-groups-for-your-alerts"></a>Configuración de grupos de acciones para las alertas

Una vez que haya configurado las alertas, puede configurar un grupo de acciones para usarlas con varias alertas. Por ejemplo, notificaciones por correo electrónico, runbooks, webhooks y mucho más. Para más información sobre los grupos de acciones, consulte [Creación y administración de grupos de acciones](../../azure-monitor/platform/action-groups.md).

1. Seleccione una alerta y, a continuación, **Crear nueva** en **Grupos de acciones**.

2. Escriba un nombre completo y un nombre corto para el grupo de acciones. Update Management usa el nombre corto en lugar del nombre completo para enviar notificaciones mediante el grupo especificado.

3. En **Acciones**, escriba un nombre para especificar la acción (por ejemplo, **Notificación por correo electrónico**).

4. En **Tipo de acción**, seleccione el tipo adecuado (por ejemplo, **Correo electrónico, SMS, Push o Voz**).

5. Seleccione **Editar detalles**.

6. Escriba el tipo de acción en el panel. Por ejemplo, si usa la opción **Correo electrónico, SMS, Push o Voz**, escriba un nombre de acción, seleccione la casilla **Correo electrónico**, escriba una dirección de correo electrónico válida y, por último, haga clic en **Aceptar**.

    ![Configuración de un grupo de acciones de correo electrónico](./media/update-mgmt-manage-updates-for-vm/configure-email-action-group.png)

7. En el panel Agregar grupo de acciones, seleccione **Aceptar**.

8. Para un correo electrónico de alerta, puede personalizar el asunto. Seleccione **Personalizar acciones** en **Crear regla** y, a continuación, seleccione **Asunto del correo electrónico**.

9. Cuando finalice, seleccione **Crear regla de alertas**.

## <a name="next-steps"></a>Pasos siguientes

