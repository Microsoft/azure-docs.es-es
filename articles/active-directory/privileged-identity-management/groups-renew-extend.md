---
title: 'Renovación del propietario del grupo expirado de las asignaciones de miembro en Privileged Identity Management: Azure AD | Microsoft Docs'
description: Aprenda a ampliar o renovar las asignaciones de grupos con roles asignables en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.subservice: pim
ms.date: 06/30/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0cfbb0be22dee4550050d6af10314f3a3bb1f583
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87505620"
---
# <a name="extend-or-renew-privileged-access-group-assignments-preview-in-privileged-identity-management"></a>Ampliación o renovación de las asignaciones de grupos de acceso con privilegios (versión preliminar) en Privileged Identity Management

Azure Active Directory (Azure AD) Privileged Identity Management (PIM) proporciona controles para administrar el ciclo de vida de acceso y asignación de los grupos de acceso con privilegios. Los administradores pueden asignar roles mediante las propiedades de fecha y hora de inicio y finalización. Cuando el fin de la asignación se aproxima, Privileged Identity Management envía notificaciones por correo electrónico a los usuarios o grupos afectados. También envía notificaciones por correo electrónico a los administradores del recurso para asegurarse de que se mantenga el acceso adecuado. Las asignaciones pueden renovarse y permanecer visibles en un estado expirado durante un máximo de 30 días, incluso si el acceso no se extiende.

## <a name="who-can-extend-and-renew"></a>Quién puede ampliar y renovar

Solo los administradores del recurso pueden ampliar o renovar las asignaciones del grupo de acceso con privilegios. El usuario o grupo afectado puede solicitar que se amplíen las asignaciones que están a punto de expirar, así como que se renueven las que ya han expirado.

## <a name="when-notifications-are-sent"></a>Cuándo se envían notificaciones

Privileged Identity Management envía notificaciones por correo electrónico a los administradores y a los usuarios afectados de las asignaciones de grupos de acceso con privilegios que expiran:

- En un plazo de 14 días antes de la expiración.
- Un día antes de la expiración.
- Cuando expira una asignación.

Los administradores reciben notificaciones cuando un usuario o grupo asignado a un rol que va a expirar, o que ha expirado, solicita la ampliación o renovación. Cuando un administrador resuelve la solicitud, se notifica la aprobación o denegación a todos los administradores y al usuario que lo solicita.

## <a name="extend-group-assignments"></a>Extensión de las asignaciones de grupo

En los pasos siguientes se describe el proceso de la solicitud, resolución o administración de una extensión o renovación de una asignación de grupos.

### <a name="self-extend-expiring-assignments"></a>Ampliación automática de asignaciones que van a expirar

Los usuarios asignados a un grupo de acceso con privilegios pueden extender las asignaciones de grupos que van a expirar directamente desde la pestaña **Válido** o **Activo** en la página de **asignaciones** del grupo. Los usuarios o grupos pueden solicitar la ampliación de las asignaciones válidas y activas que expiren en los próximos 14 días.

![Página "Mis roles" en la que se enumeran las asignaciones con una columna de acción.](media/groups-renew-extend/self-extend-group-assignment.png)

Cuando la fecha y hora de finalización de la asignación es de 14 días, el comando **Extender** está disponible. Para solicitar una extensión de la asignación de grupo, seleccione **Extender** para abrir el formulario de solicitud.

![Extensión del panel asignación de grupos con un cuadro de motivos y detalles](media/groups-renew-extend/extend-request-details-group-assignment.png)

>[!NOTE]
>Se recomienda incluir los detalles de por qué es necesaria la extensión y de cuánto tiempo debe ser (si se tiene acceso a esta información).

En cuestión de minutos, los administradores reciben una notificación por correo electrónico en la que se les solicita que revisen la solicitud de extensión. Si ya se ha enviado una solicitud para la ampliación, aparecerá una notificación de Azure en el portal.

Para ver el estado o cancelar la solicitud, abra la página de **solicitudes pendientes** de la asignación de grupo.

![Asignaciones de grupos de acceso con privilegios: página solicitudes pendientes que muestran el vínculo para cancelar](media/groups-renew-extend/group-assignment-extend-cancel-request.png)

### <a name="admin-approved-extension"></a>Ampliación aprobada por el administrador

Cuando un usuario o grupo envía una solicitud para extender la asignación de un grupo, los administradores reciben una notificación por correo electrónico con los detalles de la asignación original y el motivo de la solicitud. La notificación incluye un vínculo directo a la solicitud para que el administrador la apruebe o deniegue.

Además de usar el vínculo del correo electrónico, los administradores pueden aprobar o rechazar las solicitudes en el portal de administración de Privileged Identity Management; para ello, deben seleccionar **Aprobar solicitudes** en el panel izquierdo.

![Asignaciones de grupos de acceso con privilegios: página para aprobar solicitudes que enumera las solicitudes y vínculos que se deben aprobar o denegar.](media/groups-renew-extend/group-assignment-extend-admin-approve.png)

Cuando un administrador selecciona **Aprobar** o **Denegar**, se muestran los detalles de la solicitud, junto con un campo para especificar una justificación comercial para los registros de auditoría.

![Solicitud de aprobación de la asignación de grupos con el motivo del solicitante, el tipo de asignación, la hora de inicio, la hora de finalización y el motivo.](media/groups-renew-extend/group-assignment-extend-admin-approve-reason.png)

Al aprobar una solicitud para extender la asignación de grupos, los administradores de recursos pueden elegir una nueva fecha de inicio y finalización y un tipo de asignación. Cambiar el tipo de asignación puede ser necesario si el administrador quiere proporcionar acceso limitado para completar una tarea específica (por ejemplo, un día). En este ejemplo, el administrador puede cambiar la asignación de **Elegible** a **Activa**. Esto quiere decir que puede brindar acceso al solicitante sin requerirle que la activación.

### <a name="admin-initiated-extension"></a>Ampliación iniciada por el administrador

Si un usuario asignado a un grupo no solicita una extensión para la asignación de grupos, un administrador puede extender una asignación en nombre del usuario. Las extensiones administrativas de la asignación de grupos no requieren aprobación, pero se envían notificaciones a todos los administradores restantes después de haber extendido la asignación.

Para extender una asignación de grupos, vaya a la vista de la asignación en Privileged Identity Management. Busque la asignación que requiere una ampliación. Luego, seleccione **Extender** en la columna de acción.

![Página de asignaciones en la que se enumeran las asignaciones de grupo válidas con vínculos para extenderlas.](media/groups-renew-extend/group-assignment-extend-admin-approve.png)

## <a name="renew-group-assignments"></a>Renovación de las asignaciones de grupo

Aunque conceptualmente es similar al proceso para solicitar una extensión, el proceso para renovar una asignación de grupo expirada es diferente. Mediante los siguientes pasos, tanto las asignaciones como los administradores pueden renovar cuando sea necesario el acceso a las asignaciones que hayan expirado.

### <a name="self-renew"></a>Renovación automática

Los usuarios que ya no tengan acceso a los recursos pueden tener acceso durante un período de hasta 30 días al historial de asignaciones expiradas. Para ello, vaya a **Mis roles** en el panel izquierdo y luego seleccione la pestaña **Asignaciones expiradas**.

![Página "Mis roles": pestaña de asignaciones expiradas.](media/groups-renew-extend/groups-renew-from-my-roles.png)

La lista de asignaciones que se muestra de forma predeterminada se denomina **Asignaciones válidas**. Utilice el menú desplegable para alternar entre las asignaciones válidas y activas.

Para solicitar la renovación de cualquiera de las asignaciones de grupos de la lista, seleccione la acción **Renovar**. A continuación, indique un motivo para la solicitud. Resulta útil indicar una duración, además de cualquier contexto adicional o una justificación comercial que ayude al administrador de recursos a la hora de decidir si la aprueba o la deniega.

![Panel para renovar asignación de grupos con el cuadro del motivo.](media/groups-renew-extend/groups-renew-request-form.png)

Después de haber enviado la solicitud, se notifica a los administradores de recursos de que hay una solicitud pendiente para renovar la asignación de un grupo.

### <a name="admin-approves"></a>Aprobaciones de los administradores

Los administradores de recursos pueden acceder a la solicitud de renovación desde el vínculo de la notificación por correo electrónico o bien pueden acceder a Privileged Identity Management en Azure Portal y seleccionar **Aprobar solicitudes** en el panel izquierdo.

Cuando un administrador selecciona **Aprobar** o **Denegar**, se muestran los detalles de la solicitud, junto con un campo para especificar una justificación comercial para los registros de auditoría.

Al aprobar una solicitud para renovar la asignación de grupos, los administradores de recursos deben especificar una nueva fecha de inicio y finalización y un tipo de asignación.

## <a name="next-steps"></a>Pasos siguientes

- [Aprobar o denegar solicitudes de asignación de grupos de acceso con privilegios en Privileged Identity Management](groups-approval-workflow.md)
- [Configuración del grupo de acceso con privilegios en Privileged Identity Management](groups-role-settings.md)
