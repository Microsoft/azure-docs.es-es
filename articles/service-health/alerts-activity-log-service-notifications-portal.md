---
title: Recepción de alertas del registro de actividad en las notificaciones del servicio de Azure con Azure Portal
description: Reciba notificaciones por SMS, correo electrónico o webhook cuando se produzcan eventos en el servicio de Azure.
ms.topic: conceptual
ms.date: 06/27/2019
ms.openlocfilehash: b90940c4532370e7742f736708625ddec283aab1
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87499305"
---
# <a name="create-activity-log-alerts-on-service-notifications-using-the-azure-portal"></a>Creación de alertas del registro de actividad en notificaciones del servicio mediante Azure Portal
## <a name="overview"></a>Información general

En este artículo se explica cómo usar Azure Portal para configurar las alertas del registro de actividad para las notificaciones de mantenimiento de un servicio mediante Azure Portal.  

Las notificaciones de mantenimiento del servicio se almacenan en el [registro de actividad de Azure](../azure-monitor/platform/platform-logs-overview.md). Debido al gran volumen de información almacenada en el registro de actividad, existe una interfaz de usuario independiente que facilita la visualización y la configuración de alertas en las notificaciones de mantenimiento del servicio. 

Puede recibir una alerta cuando Azure envía notificaciones de estado del servicio a la suscripción de Azure. Puede configurar la alerta en función de:

- La clase de notificación de estado del servicio (problemas de servicio, mantenimiento planificado, avisos de estado y avisos de seguridad).
- La suscripción afectada.
- Los servicios afectados.
- Las regiones afectadas.

> [!NOTE]
> Las notificaciones de estado del servicio no envían alertas para los eventos de estado de los recursos.

También puede configurar a quién se debe enviar la alerta:

- Seleccione un grupo de acciones existente.
- Cree un nuevo grupo de acciones (que puede usarse para futuras alertas).

Para más información sobre los grupos de acciones, consulte [Creación y administración de grupos de acciones](../azure-monitor/platform/action-groups.md).

Para obtener información sobre cómo configurar las alertas de notificación de mantenimiento del servicio mediante plantillas de Azure Resource Manager, consulte [Plantillas de Resource Manager](../azure-monitor/platform/alerts-activity-log.md).

### <a name="watch-a-video-on-setting-up-your-first-azure-service-health-alert"></a>Ver un vídeo sobre cómo configurar su primera alerta de Azure Service Health

>[!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2OaXt]

## <a name="create-service-health-alert-using-azure-portal"></a>Creación de alertas de Service Health mediante Azure Portal
1. En el [portal](https://portal.azure.com), seleccione **Estado del servicio**.

    ![El servicio "Estado del servicio"](media/alerts-activity-log-service-notifications/home-servicehealth.png)

1. En la sección **Alertas**, seleccione **Alertas de estado**.

    ![La pestaña "Alertas de estado"](media/alerts-activity-log-service-notifications/alerts-blades-sh.png)

1. Seleccione **Agregar alerta de Service Health** y rellene los campos.

    ![El comando "Crear alerta de estado de servicio"](media/alerts-activity-log-service-notifications/service-health-alert.png)

1. Seleccione la **suscripción**, los **servicios** y las **regiones** para los cuales quiere recibir alertas.

    [![Cuadro de diálogo "Agregar alerta de registro de actividad"](./media/alerts-activity-log-service-notifications/activity-log-alert-new-ux.png)](./media/alerts-activity-log-service-notifications/activity-log-alert-new-ux.png#lightbox)

> [!NOTE]
>Esta suscripción se usa para guardar la alerta del registro de actividad. El recurso de la alerta se implementa en esta suscripción y supervisa los eventos en el registro de actividad para dicho recurso.

5. Elija los **Tipos de evento** para los que quiere recibir alertas: *Problemas de servicio*, *Mantenimiento planeado*, *Avisos de estado* y *Avisos de seguridad*.

6. Haga clic en **Seleccionar grupo de acciones** para elegir un grupo de acciones existente o para crear un nuevo grupo de acciones. Para obtener más información sobre los grupos de acciones, consulte [Creación y administración de grupos de acciones en Azure Portal](../azure-monitor/platform/action-groups.md).


7. Para definir los detalles de la alerta, escriba un **nombre de regla de alertas** y una **descripción**.

8. Seleccione el **grupo de recursos** donde quiere guardar la alerta.



En unos minutos, la alerta está activa y comienza a desencadenarse en función de las condiciones especificadas durante la creación.

Obtenga información acerca de cómo [configurar notificaciones de webhook para los sistemas de administración de problemas existentes](service-health-alert-webhook-guide.md). Para obtener información sobre el esquema de webhook para las alertas del registro de actividad, consulte [Webhooks para alertas del registro de actividad de Azure](../azure-monitor/platform/activity-log-alerts-webhook.md).


## <a name="next-steps"></a>Pasos siguientes
- Obtenga información sobre los [procedimientos recomendados para la configuración de alertas de Azure Service Health](https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUa).
- Obtenga información sobre cómo [configurar notificaciones de inserción móviles de Azure Service Health](https://www.microsoft.com/en-us/videoplayer/embed/RE2OtUw).
- Obtenga información acerca de cómo [configurar notificaciones de webhook para los sistemas de administración de problemas existentes](service-health-alert-webhook-guide.md).
- Más información acerca de las [Notificaciones del estado del servicio](service-notifications.md).
- Más información sobre la [Limitación del número de notificaciones](../azure-monitor/platform/alerts-rate-limiting.md).
- Revise el [Esquema de webhook de alertas del registro de actividad](../azure-monitor/platform/activity-log-alerts-webhook.md).
- Consulte la [introducción a las alertas del registro de actividad](../azure-monitor/platform/alerts-overview.md) y aprenda cómo puede recibir alertas.
- Más información sobre los [grupos de acciones](../azure-monitor/platform/action-groups.md).
