---
title: Creación de alertas de Resource Health mediante Azure Portal
description: Cree una alerta mediante Azure Portal que notifique cuándo dejan de estar disponibles los recursos de Azure.
ms.topic: conceptual
ms.date: 6/23/2020
ms.openlocfilehash: 3a0f4704880d040106a2e4112d621b6d8ffbede6
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87091579"
---
# <a name="configure-resource-health-alerts-using-azure-portal"></a>Configuración de alertas del estado de los recursos mediante Azure Portal

En este artículo se explica cómo configurar las alertas del registro de actividad para las notificaciones de mantenimiento de un recurso mediante Azure Portal.

Azure Resource Health le mantiene informado sobre el estado actual y pasado de sus recursos de Azure. Además, le notifica casi en tiempo real de los cambios de estado en estos recursos. La creación y la personalización de alertas mediante programación en Resource Health se puede realizar en bloque.

> [!NOTE]
> Las alertas de Resource Health están actualmente en versión preliminar.

Las notificaciones de mantenimiento del recurso se almacenan en el [registro de actividad de Azure](../azure-monitor/platform/platform-logs-overview.md). Debido al volumen posiblemente grande de la información almacenada en el registro de actividad, hay una interfaz de usuario independiente que facilita la visualización y la configuración de alertas en las notificaciones de mantenimiento del servicio.
Puede recibir una alerta cuando el recurso de Azure envía notificaciones de mantenimiento del recurso a la suscripción de Azure. Puede configurar la alerta en función de:

* La suscripción afectada.
* Los tipos de recursos afectados.
* Los grupos de recursos afectados.
* Los recursos afectados.
* Los estados de evento de los recursos afectados.
* Los estados afectados de los recursos.
* Los tipos de recursos de los recursos afectados.

También puede configurar a quién se debe enviar la alerta:

* Seleccione un grupo de acciones existente.
* Cree un nuevo grupo de acciones (que puede usarse para futuras alertas).

Para más información sobre los grupos de acciones, consulte [Creación y administración de grupos de acciones](../azure-monitor/platform/action-groups.md).

Para obtener información sobre cómo configurar las alertas de notificación de mantenimiento del recurso mediante plantillas de Azure Resource Manager, consulte [Plantillas de Resource Manager](./resource-health-alert-arm-template-guide.md).
Alerta de Resource Health mediante Azure Portal

## <a name="resource-health-alert-using-azure-portal"></a>Alerta de Resource Health mediante Azure Portal

1. En Azure [Portal](https://portal.azure.com/), seleccione **Mantenimiento del servicio**.

    ![Selección de Mantenimiento del servicio](./media/resource-health-alert-monitor-guide/service-health-selection.png)
2. En la sección **Estado de los recursos**, seleccione **Service Health**.
3. Seleccione **Add resource health alert** (Agregar alerta de mantenimiento del recurso) y rellene los campos.
4. En Destino de la alerta, seleccione estos datos para los que desea recibir alertas: **Suscripción**, **Tipos de recursos**, **Grupos de recursos** y **Recurso**.

    ![Selección del destino](./media/resource-health-alert-monitor-guide/alert-target.png)

5. En la condición de alerta, seleccione:
    1. El **Estado del evento** para el que quiere recibir alertas. Nivel de gravedad del evento: Activo, Resuelto,En curso, Actualizado
    2. El **Estado del recurso** para el que quiere recibir alertas. El estado del recurso del evento: Disponible, No disponible, Desconocido y Degradado
    3. El **Tipo de motivo** para el que quiere recibir alertas. La causa del evento: Plataforma iniciada, Iniciada por el usuario ![Selección de mantenimiento y selección de condiciones de alerta](./media/resource-health-alert-monitor-guide/alert-condition.png)
6. En Definir detalles de la alerta, proporcione los detalles siguientes:
    1. **Nombre de la regla de alertas**: El nombre de la nueva regla de alertas.
    2. **Descripción**: La descripción de la nueva regla de alertas.
    3. **Guardar la alerta en el grupo de recursos**: Seleccione el grupo de recursos donde quiere guardar esta nueva regla.
7. En el **grupo de acciones**, en el menú desplegable, especifique el grupo de acciones que quiere asignar a esta nueva regla de alertas. O bien, [cree un nuevo grupo de acción](../azure-monitor/platform/action-groups.md) y asígneselo a la nueva regla. Para crear un nuevo grupo, seleccione **+ Nuevo grupo**.
8. Para habilitar las reglas después de crearlas, seleccione **Sí** en la opción **Habilitar regla tras la creación**.
9. Seleccione **Crear regla de alertas**.

Se crea la nueva regla de alertas del registro de actividad y aparece un mensaje de confirmación en la esquina superior derecha de la ventana.
Puede habilitar, deshabilitar, editar o eliminar una regla. Obtenga más información sobre [cómo administrar las reglas del registro de actividad](../azure-monitor/platform/alerts-activity-log.md#view-and-manage-in-the-azure-portal).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre Resource Health:

* [Información general sobre Azure Resource Health](Resource-health-overview.md)
* [Tipos de recursos y comprobaciones de mantenimiento disponibles a través de Azure Resource Health](resource-health-checks-resource-types.md)

Creación de alertas de Service Health:

* [Configuración de alertas de Service Health](./alerts-activity-log-service-notifications-portal.md) 
* [Esquema de eventos del registro de actividad de Azure](../azure-monitor/platform/activity-log-schema.md)
* [Configuración de alertas de estado de los recursos con plantillas de Resource Manager](./resource-health-alert-arm-template-guide.md)
