---
title: Creación y administración de grupos de acciones en Azure Portal
description: Obtenga información acerca de cómo crear y administrar grupos de acciones en Azure Portal.
author: dkamstra
services: azure-monitor
ms.service: azure-monitor
ms.topic: conceptual
ms.date: 5/10/2019
ms.author: dukek
ms.subservice: alerts
ms.openlocfilehash: fba5119feb1ff7a0170a573371e479caa5fc33eb
ms.sourcegitcommit: f6c85922b9e70bb83879e52c2aec6307c99a0cac
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2019
ms.locfileid: "65544467"
---
# <a name="create-and-manage-action-groups-in-the-azure-portal"></a>Creación y administración de grupos de acciones en Azure Portal
## <a name="overview"></a>Información general ##
Un grupo de acciones es una colección de las preferencias de notificación que el propietario de una suscripción de Azure define. Las alertas de Azure Monitor y Service Health usan grupos de acciones para notificar a los usuarios que se ha desencadenado una alerta. Varias alertas pueden usar el mismo grupo de acciones o distintos grupos de acciones en función de los requisitos del usuario. Puede configurar un máximo de 2000 grupos de acciones en una suscripción.

Configurar una acción para notificar a una persona por correo electrónico o SMS, que reciben una confirmación que indica que se han agregado al grupo de acción.

En este artículo se muestra cómo crear y administrar grupos de acciones en el portal de Azure.

Cada acción se compone de las siguientes propiedades:

* **Nombre**: un identificador único dentro del grupo de acciones.  
* **Tipo de acción**: Realizar la acción. El envío de llamadas de voz, mensajes de texto o correo electrónico o el desencadenamiento de varios tipos de acciones automatizadas son algunos ejemplos. Consulte los tipos más adelante en este artículo.
* **Detalles**: Los detalles correspondientes que varían según el *tipo de acción*.

Para más información sobre el uso de plantillas de Azure Resource Manager para configurar grupos de acciones, consulte [Plantillas de Resource Manager para grupos de acciones](../../azure-monitor/platform/action-groups-create-resource-manager-template.md).

## <a name="create-an-action-group-by-using-the-azure-portal"></a>Creación de un grupo de acciones con Azure Portal ##
1. En el [portal](https://portal.azure.com), seleccione **Monitor**. El **Monitor** panel consolida toda la supervisión la configuración y datos en una vista.

    ![Servicio "Monitor"](./media/action-groups/home-monitor.png)
1. Seleccione **Alertas** y seleccione **Administrar los grupos de acciones**.

    ![Botón Administrar los grupos de acciones](./media/action-groups/manage-action-groups.png)
1. Seleccione **Agregar grupo de acciones** y rellene los campos.

    ![Comando "Agregar grupo de acciones"](./media/action-groups/add-action-group.png)
1. Escriba un nombre en el cuadro de texto **Nombre del grupo de acciones** y especifique un nombre en el cuadro de texto **Nombre corto**. El nombre corto se utiliza en lugar del nombre completo del grupo de acciones cuando se envían notificaciones mediante este grupo.

      ![Cuadro de diálogo "Agregar grupo de acciones"](./media/action-groups/action-group-define.png)

1. El cuadro de texto **Suscripción** se rellena automáticamente con la suscripción actual. La suscripción es aquella en la que se guarda el grupo de acciones.

1. Seleccione el **Grupo de recursos** en el que se guarda el grupo de acciones.

1. Definir una lista de acciones. Proporcionar lo siguiente para cada acción:

     a. **Nombre**: escriba un identificador único para esta acción.

    b. **Tipo de acción**: seleccione correo electrónico, SMS, notificación push, voz, aplicación lógica, webhook, ITSM o runbook de Automation.

    c. **Detalles**: según el tipo de acción, proporcione un número de teléfono, una dirección de correo electrónico o un identificador URI de webhook, una aplicación de Azure, una conexión de ITSM o un runbook de Automation. Para la acción de ITSM, especifique además **Elemento de trabajo** y otros campos que requiera la herramienta ITSM.

1. Seleccione **Aceptar** para crear el grupo de acciones.

## <a name="manage-your-action-groups"></a>Administración de los grupos de acciones ##
Después de crear un grupo de acciones, está visible en el **grupos de acciones** sección de la **Monitor** panel. Seleccione el grupo de acciones que desea administrar para:

* Agregar, editar o quitar acciones.
* Eliminar el grupo de acciones.

## <a name="action-specific-information"></a>Información específica de la acción
> [!NOTE]
> Consulte [límites de servicio de suscripción para la supervisión](https://docs.microsoft.com/azure/azure-subscription-service-limits#monitor-limits) para los límites numéricos en cada uno de los siguientes elementos.  

**Aplicación de Azure inserción** -puede tener un número limitado de acciones de aplicación de Azure en un grupo de acciones.

**Correo electrónico**: se enviarán mensajes de correo electrónico desde las siguientes direcciones de correo electrónico. Asegúrese de que el filtrado de correo electrónico esté configurado correctamente.
- azure-noreply@microsoft.com
- azureemail-noreply@microsoft.com
- alerts-noreply@mail.windowsazure.com

Puede tener un número limitado de acciones de correo electrónico en un grupo de acciones. Consulte el artículo de [información sobre las limitaciones](./../../azure-monitor/platform/alerts-rate-limiting.md).

**ITSM** -puede tener un número limitado de acciones de ITSM en un grupo de acciones. La acción de ITSM requiere una conexión de ITSM. Aprenda cómo crear una [conexión de ITSM](../../azure-monitor/platform/itsmc-overview.md).

**Aplicación lógica** -puede tener un número limitado de acciones de aplicación lógica en un grupo de acciones.

**Aplicación de función** -claves de la función para las aplicaciones de función configuradas como acciones se leen a través de la API de funciones, que actualmente requiere que las aplicaciones de función de v2 para configurar el valor "AzureWebJobsSecretStorageType" de la aplicación a "archivos". Para obtener más información, consulte [cambia a la administración de claves en Functions V2]( https://aka.ms/funcsecrets).

**Runbook** -puede tener un número limitado de acciones de Runbook en un grupo de acciones. Hacer referencia a la [límites de servicio de suscripción de Azure](../../azure-subscription-service-limits.md) para conocer los límites en las cargas de Runbook.

**SMS** -puede tener un número limitado de acciones de SMS en un grupo de acciones. Consulte también el [información sobre las limitaciones](./../../azure-monitor/platform/alerts-rate-limiting.md) y [comportamiento de las alertas por SMS](../../azure-monitor/platform/alerts-sms-behavior.md) para obtener información adicional. 

**Voz** -puede tener un número limitado de acciones de voz en un grupo de acciones. Consulte la [información sobre las limitaciones](./../../azure-monitor/platform/alerts-rate-limiting.md) artículo.

**Webhook** -puede tener un número limitado de acciones de Webhook en un grupo de acciones. Los Webhooks se reintentan utilizando las reglas siguientes. Se vuelve a intentar la llamada al webhook un máximo de 2 veces cuando los siguientes códigos de estado HTTP se devuelvan: 408, 429, 503, 504 o el punto de conexión HTTP no responda. El primer reintento se produce transcurridos 10 segundos. El segundo reintento se produce transcurridos 100 segundos. Después de dos errores, no hay ningún grupo de acción llamará el punto de conexión durante 30 minutos. 

Rangos de direcciones IP de origen
 - 13.72.19.232
 - 13.106.57.181
 - 13.106.54.3
 - 13.106.54.19
 - 13.106.38.142
 - 13.106.38.148
 - 13.106.57.196
 - 52.244.68.117
 - 52.244.65.137
 - 51.4.138.199
 - 51.5.148.86
 - 51.5.149.19

Para recibir actualizaciones sobre los cambios a estas direcciones IP, se recomienda que configurar una alerta de estado del servicio, que se supervisa para notificaciones informativas sobre el servicio de grupos de acciones.

## <a name="next-steps"></a>Pasos siguientes ##
* Más información sobre el [comportamiento de las alertas por SMS](../../azure-monitor/platform/alerts-sms-behavior.md).  
* [Comprender el esquema de webhook de alertas del registro de actividad](../../azure-monitor/platform/activity-log-alerts-webhook.md).  
* Obtenga más información sobre [Conector de ITSM](../../azure-monitor/platform/itsmc-overview.md)
* Más información sobre la [limitación de velocidad](../../azure-monitor/platform/alerts-rate-limiting.md) en las alertas.
* Consulte la [introducción a las alertas del registro de actividad](../../azure-monitor/platform/alerts-overview.md) y aprenda cómo puede recibir alertas.  
* Aprenda a [configurar alertas siempre que se publique una notificación de mantenimiento de un servicio](../../azure-monitor/platform/alerts-activity-log-service-notifications.md).
