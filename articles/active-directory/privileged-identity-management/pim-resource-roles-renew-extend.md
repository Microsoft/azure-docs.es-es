---
title: Ampliar o renovar las asignaciones de roles de recursos de Azure en PIM - Azure Active Directory | Microsoft Docs
description: Aprenda a ampliar o renovar asignaciones de roles de recursos de Azure en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.subservice: pim
ms.date: 04/02/2018
ms.author: rolyon
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: a064fc67bf94ba6aa443e429fe83179d84cada84
ms.sourcegitcommit: 6ea7f0a6e9add35547c77eef26f34d2504796565
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/14/2019
ms.locfileid: "65602721"
---
# <a name="extend-or-renew-azure-resource-role-assignments-in-pim"></a>Ampliación o renovación de asignaciones de roles de recursos de Azure en PIM

Azure Active Directory (Azure AD) Privileged Identity Management (PIM) presenta nuevos controles para administrar el ciclo de vida de acceso y asignación de recursos de Azure. Los administradores pueden asignar pertenencia mediante las propiedades de fecha y hora de inicio y finalización. Cuando el fin de la asignación se aproxima, PIM envía notificaciones por correo electrónico a los usuarios o grupos afectados. También envía notificaciones por correo electrónico a los administradores del recurso para asegurarse de que se mantenga el acceso adecuado. Las asignaciones pueden renovarse y permanecer visibles en un estado expirado durante un máximo de 30 días, incluso si el acceso no se extiende.

## <a name="who-can-extend-and-renew"></a>¿Quién puede ampliar y renovar?

Las asignaciones de roles solo pueden ampliarlas o renovarlas los administradores del recurso. El miembro afectado puede solicitar que se extiendan los roles que están a punto de expirar, así como que se renueven los que ya han expirado.

## <a name="when-are-notifications-sent"></a>¿Cuándo se envían las notificaciones?

PIM envía notificaciones por correo electrónico tanto a los administradores como a los miembros afectados de roles que expiran en 14 días y un día antes de la expiración. Envía un correo electrónico adicional cuando una asignación ha expirado oficialmente. 

Los administradores reciben notificaciones cuando un miembro de un rol que va a expirar, o que ha expirado, solicita la extensión o renovación. Cuando un administrador específico resuelve la solicitud, se envía a los otros administradores una notificación de la decisión de resolución (aprobada o denegada). Luego, se informa de la decisión al miembro que ha realizado la solicitud. 

## <a name="extend-role-assignments"></a>Extensión de asignaciones de roles

En los pasos siguientes se describe el proceso de la solicitud, resolución o administración de una extensión o renovación de una asignación de roles. 

### <a name="member-extend"></a>Extensión de miembro

Los miembros de una asignación de roles pueden extender las asignaciones de roles que expiran directamente tanto desde las pestañas **Elegible** o **Activo** de la página **Mis roles** de un recurso como desde el nivel superior de la página **Mis roles** del portal de PIM. Los miembros pueden solicitar una ampliación de los roles elegibles (asignados) y activos que expiran en los próximos 14 días.

![Extender roles](media/pim-resource-roles-renew-extend/aadpim-rbac-extend-ui.png)

Cuando la fecha y hora de finalización de la asignación es anterior a 14 días, el botón para **Extender** se convierte en un vínculo activo en la interfaz de usuario. En el ejemplo siguiente, se supone que la fecha actual es el 27 de marzo.

![Extender el botón](media/pim-resource-roles-renew-extend/aadpim-rbac-extend-within-14.png)

Para solicitar una extensión de la asignación de este rol, seleccione **Extender** para abrir el formulario de solicitud.

![Abrir el formulario de solicitud](media/pim-resource-roles-renew-extend/aadpim-rbac-extend-role-assignment-request.png)

Para ver información acerca de la asignación original, expanda **Detalles de asignación**. Especifique un motivo para la solicitud de extensión y, luego, seleccione **Extender**.

>[!Note]
>Se recomienda incluir los detalles de por qué es necesaria la extensión y de cuánto tiempo debe ser (si se tiene acceso a esta información).

![Extender la asignación de roles](media/pim-resource-roles-renew-extend/aadpim-rbac-extend-form-complete.png)

En cuestión de minutos, los administradores de recursos reciben una notificación por correo electrónico en la que se les solicita que revisen la solicitud de extensión. Si ya se ha enviado una solicitud de extensión, aparece una notificación del sistema en la parte superior de Azure Portal en la que se explica el error.

![Notificación que explica el error](media/pim-resource-roles-renew-extend/aadpim-rbac-extend-failed-existing-request.png)

Vaya a la pestaña **Solicitudes pendientes** del panel izquierdo para ver el estado de la solicitud o para cancelarla.

![Solicitudes pendientes](media/pim-resource-roles-renew-extend/aadpim-rbac-extend-cancel-request.png)

### <a name="admin-approve"></a>Aprobación de administrador

Cuando un miembro envía una solicitud para extender la asignación de un rol, los administradores de recursos reciben una notificación por correo electrónico con los detalles de la asignación original y el motivo de la solicitud. La notificación incluye un vínculo directo a la solicitud para que el administrador la apruebe o deniegue. 

Además de usar el vínculo del correo electrónico, los administradores pueden aprobar o rechazar las solicitudes en el portal de administración de PIM y seleccionar **Aprobar solicitudes** en el panel izquierdo.

![Captura de pantalla del error](media/pim-resource-roles-renew-extend/aadpim-rbac-extend-admin-approve-grid.png)

Cuando un administrador selecciona **Aprobar** o **Denegar**, se muestran los detalles de la solicitud, junto con un campo para especificar una justificación para los registros de auditoría.

![Aprobar solicitud de asignación de roles](media/pim-resource-roles-renew-extend/aadpim-rbac-extend-admin-approve-blade.png)

Al aprobar una solicitud para extender la asignación de roles, los administradores de recursos pueden elegir una nueva fecha de inicio y finalización y un tipo de asignación. Cambiar el tipo de asignación puede ser necesario si el administrador quiere proporcionar acceso limitado para completar una tarea específica (por ejemplo, un día). En este ejemplo, el administrador puede cambiar la asignación de **Elegible** a **Activa**. Esto quiere decir que puede brindar acceso al solicitante sin requerirle que la activación.

### <a name="admin-extend"></a>Ampliación de administradores

Si un miembro del rol olvida solicitar la extensión de la pertenencia al rol, o no puede solicitarla, un administrador puede extender una asignación en nombre del miembro. Las extensiones administrativas de la pertenencia a un rol no requieren aprobación, pero se envían notificaciones a todos los administradores restantes después de haber extendido el rol.

Para extender una pertenencia a un rol, navegue a la vista de miembro o de rol de recurso en el PIM. Busque al miembro que requiere una extensión. Luego, seleccione **Extender** en la columna de acción.

![Extender la pertenencia a un rol](media/pim-resource-roles-renew-extend/aadpim-rbac-extend-admin-extend.png)

## <a name="renew-role-assignments"></a>Renovación de asignaciones de roles

Aunque conceptualmente el proceso para solicitar una extensión es similar, el proceso para renovar una asignación de rol que ha expirado es diferente. Mediante los siguientes pasos, tanto los miembros como los administradores pueden renovar el acceso a los roles que hayan expirado cuando sea necesario.

### <a name="member-renew"></a>Renovación de miembro

Los miembros que ya no tengan acceso a recursos pueden tener acceso durante un período de hasta 30 días al historial de asignaciones expiradas. Para ello, vaya a **Mis roles** en el panel izquierdo y luego seleccione la pestaña **Roles expirados** en la sección de roles de recursos de Azure.

![Pestaña roles expirados](media/pim-resource-roles-renew-extend/aadpim-rbac-renew-from-myroles.png)

El valor predeterminado de la lista de roles que se muestra es **Roles elegibles**. Utilice el menú desplegable para alternar entre los roles asignados Elegibles y Activos.

Para solicitar la renovación de cualquiera de las asignaciones de roles de la lista, seleccione la acción **Renovar**. A continuación, indique un motivo para la solicitud. Resulta útil indicar una duración, además de cualquier contexto adicional que ayude al administrador de recursos a la hora de decidir si la aprueba o la deniega.

![Renovar asignaciones de roles](media/pim-resource-roles-renew-extend/aadpim-rbac-renew-request-form.png)

Después de haber enviado la solicitud, se notifica a los administradores de recursos de que hay una solicitud pendiente para renovar la asignación de un rol.

### <a name="admin-approves"></a>Aprobaciones de los administradores

Los administradores de recursos pueden acceder a la solicitud de renovación desde el vínculo de la notificación por correo electrónico, o bien pueden acceder a PIM en Azure Portal y seleccionar **Aprobar solicitudes** en el panel izquierdo.

![Aprobar solicitudes](media/pim-resource-roles-renew-extend/aadpim-rbac-extend-admin-approve-grid.png)

Cuando un administrador selecciona **Aprobar** o **Denegar**, se muestran los detalles de la solicitud, junto con un campo para especificar una justificación para los registros de auditoría.

![Aprobar asignaciones de roles](media/pim-resource-roles-renew-extend/aadpim-rbac-extend-admin-approve-blade.png)

Al aprobar una solicitud para renovar la asignación de roles, los administradores de recursos deben especificar una nueva fecha de inicio y finalización y un tipo de asignación. 

### <a name="admin-renew"></a>Renovación de administradores

Los administradores de recursos pueden renovar las asignaciones de roles expirados en la pestaña **Miembros** del menú de navegación izquierdo de un recurso. También pueden renovar las asignaciones de roles expirados en la pestaña de roles **Expirados** de un rol de recursos.

En la pantalla **Miembros**, seleccione **Roles expirados** para ver una lista de todas las asignaciones de roles que han expirado.

![Roles expirados](media/pim-resource-roles-renew-extend/aadpim-rbac-renew-from-member-blade.png)

## <a name="next-steps"></a>Pasos siguientes

- [Aprobación o rechazo de solicitudes de roles de recursos de Azure en PIM](pim-resource-roles-approval-workflow.md)
- [Configuración de roles de recurso de Azure en PIM](pim-resource-roles-configure-role-settings.md)
