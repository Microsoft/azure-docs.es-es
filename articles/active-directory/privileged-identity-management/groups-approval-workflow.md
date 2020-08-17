---
title: 'Aprobación de solicitudes de activación para miembros y propietarios del grupo en Privileged Identity Management: Azure AD'
description: Aprenda a aprobar o denegar solicitudes para los grupos a los que se pueden asignar roles en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 07/27/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: b3fbb92c2e3623f5fd9571cd94ae521a41139dd6
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87505580"
---
# <a name="approve-activation-requests-for-privileged-access-group-members-and-owners-preview"></a>Aprobación de solicitudes de activación para miembros y propietarios de un grupo de acceso con privilegios (versión preliminar)

Con Privileged Identity Management (PIM) en Azure Active Directory (Azure AD), puede configurar miembros y propietarios de un grupo de acceso con privilegios para requerir la aprobación de la activación y elegir usuarios o grupos de la organización de Azure AD como aprobadores delegados. Se recomienda seleccionar dos o más aprobadores para cada grupo a fin de reducir la carga de trabajo del administrador de roles con privilegios. Los aprobadores delegados tienen 24 horas para aprobar las solicitudes. Si no se aprueba una solicitud en un plazo de 24 horas, el usuario apto debe volver a enviar una nueva solicitud. El período de tiempo de aprobación de 24 horas no es configurable.

Siga los pasos que se describen en este artículo para aprobar o denegar solicitudes para los roles de recursos de Azure.

## <a name="view-pending-requests"></a>Ver solicitudes en espera

Como aprobador delegado, recibirá una notificación por correo electrónico cuando una solicitud de rol de recursos de Azure está pendiente de su aprobación. Puede ver las solicitudes pendientes en Privileged Identity Management.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Aprobar solicitudes**.

    ![Aprobar solicitudes: página de recursos de Azure que muestra la solicitud de revisión](./media/groups-approval-workflow/groups-select-request.png)

    En la sección **Solicitudes de activación de roles**, verá una lista de solicitudes pendientes de su aprobación.

## <a name="approve-requests"></a>Aprobar solicitudes

1. Busque y seleccione la solicitud que desea aprobar y seleccione **Aprobar**.

    ![Aprobar solicitudes: panel de aprobación o denegación con detalles y cuadro Justificación](./media/groups-approval-workflow/groups-confirm-approval.png)

1. En el cuadro **Justificación**, escriba la justificación empresarial.

1. Seleccione **Confirmar**. La aprobación genera una notificación de Azure.

## <a name="deny-requests"></a>Denegar solicitudes

1. Busque y seleccione la solicitud que desea denegar y seleccione **Denegar**.

    ![Aprobar solicitudes: panel de aprobación o denegación con detalles y cuadro Justificación](./media/groups-approval-workflow/groups-confirm-denial.png)

1. En el cuadro **Justificación**, escriba la justificación empresarial.

1. Seleccione **Confirmar**. La denegación genera una notificación de Azure.

## <a name="workflow-notifications"></a>Notificaciones de flujo de trabajo

A continuación proporcionamos información sobre las notificaciones de flujo de trabajo:

- Los aprobadores reciben una notificación por correo electrónico cuando una solicitud de una asignación de grupo está pendiente de su revisión. Las notificaciones por correo electrónico incluyen un vínculo directo a la solicitud donde el aprobador puede aprobarla o rechazarla.
- Las solicitudes las resuelve el primer aprobador que aprueba o rechaza.
- Cuando un aprobador responde a la solicitud, se notifica a todos los aprobadores de la acción.

>[!Note]
>Un administrador que cree que un usuario aprobado no debe estar activo puede eliminar la asignación de grupo activa en Privileged Identity Management. Aunque los administradores de recursos no reciben notificaciones de solicitudes pendientes a menos que sean aprobadores, pueden ver y cancelar solicitudes pendientes para todos los usuarios viendo solicitudes pendientes en Privileged Identity Management.

## <a name="next-steps"></a>Pasos siguientes

- [Ampliación o renovación de asignaciones de grupo en Privileged Identity Management](pim-resource-roles-renew-extend.md)
- [Notificaciones por correo electrónico en Privileged Identity Management](pim-email-notifications.md)
- [Aprobación o denegación de solicitudes de asignación de grupo en Privileged Identity Management](azure-ad-pim-approval-workflow.md)
