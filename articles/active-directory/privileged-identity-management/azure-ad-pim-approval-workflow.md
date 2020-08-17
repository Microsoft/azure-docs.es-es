---
title: 'Aprobación o rechazo de solicitudes de roles de Azure AD en PIM: Azure Active Directory | Microsoft Docs'
description: Aprenda a aprobar o rechazar solicitudes para los roles de Azure AD en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: pim
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/07/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 24471ff5a085be905f9316f1af3c95c3a7912bf1
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421542"
---
# <a name="approve-or-deny-requests-for-azure-ad-roles-in-privileged-identity-management"></a>Aprobación o rechazo de solicitudes para los roles de Azure AD en Privileged Identity Management

Con Azure Active Directory (Azure AD) Privileged Identity Management (PIM), puede configurar roles para requerir la aprobación para la activación y elegir uno o varios usuarios o grupos como aprobadores delegados. Los aprobadores delegados tienen 24 horas para aprobar las solicitudes. Si no se aprueba una solicitud en un plazo de 24 horas, el usuario apto debe volver a enviar una nueva solicitud. El período de tiempo de aprobación de 24 horas no es configurable.

## <a name="determine-your-version-of-pim"></a>Determinación de la versión de PIM

Desde noviembre de 2019, la parte de roles de Azure AD de Privileged Identity Management se está actualizando a una nueva versión que coincide con las experiencias de los roles de Azure. Esta actualización introduce características adicionales y [cambios en la API existente](azure-ad-roles-features.md#api-changes). Mientras se implementa la nueva versión, los procedimientos que seguirá en este artículo dependerán de la versión de Privileged Identity Management que tenga actualmente. Siga los pasos de esta sección para determinar la versión de Privileged Identity Management que tiene. Cuando averigüe la versión de Privileged Identity Management, puede seleccionar los procedimientos de este artículo que coincidan con esa versión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un usuario que tenga el rol [Administrador de roles con privilegios](../users-groups-roles/directory-assign-admin-roles.md#privileged-role-administrator).
1. Abra **Azure AD Privileged Identity Management**. Si tiene un banner en la parte superior de la página de introducción, siga las instrucciones de la pestaña **Nueva versión** de este artículo. De lo contrario, siga las instrucciones de la pestaña **Versión anterior**.

    [![Seleccionar Azure AD > Privileged Identity Management.](media/pim-how-to-add-role-to-user/pim-new-version.png)](media/pim-how-to-add-role-to-user/pim-new-version.png#lightbox)

Siga los pasos que se describen en este artículo para aprobar o denegar solicitudes para los roles de Azure AD.

# <a name="new-version"></a>[Nueva versión](#tab/new)

## <a name="view-pending-requests"></a>Ver solicitudes en espera

Como aprobador delegado, recibirá una notificación por correo electrónico cuando una solicitud de rol de Azure AD está pendiente de su aprobación. Puede ver estas solicitudes pendientes en Privileged Identity Management.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Aprobar solicitudes**.

    ![Aprobar solicitudes: página de recursos que muestra la solicitud de revisión de roles de Azure AD](./media/azure-ad-pim-approval-workflow/resources-approve-pane.png)

    En la sección **Solicitudes de activación de roles**, verá una lista de solicitudes pendientes de su aprobación.

## <a name="approve-requests"></a>Aprobar solicitudes

1. Busque y seleccione la solicitud que desea aprobar. Aparece una página aprobar o denegar.

    ![Aprobar solicitudes: panel de aprobación o denegación con detalles y cuadro Justificación](./media/azure-ad-pim-approval-workflow/resources-approve-pane.png)

1. En el cuadro **Justificación**, escriba la justificación empresarial.

1. Seleccione **Aprobar**. Recibirá una notificación de Azure de su aprobación.

    ![Notificación de aprobación que muestra que se ha aprobado la solicitud](./media/pim-resource-roles-approval-workflow/resources-approve-pane.png))

## <a name="deny-requests"></a>Denegar solicitudes

1. Busque y seleccione la solicitud que desea denegar. Aparece una página aprobar o denegar.

    ![Aprobar solicitudes: panel de aprobación o denegación con detalles y cuadro Justificación](./media/pim-resource-roles-approval-workflow/resources-approve-pane.png)

1. En el cuadro **Justificación**, escriba la justificación empresarial.

1. Seleccione **Denegar**. Aparecerá una notificación con su denegación.

## <a name="workflow-notifications"></a>Notificaciones de flujo de trabajo

A continuación proporcionamos información sobre las notificaciones de flujo de trabajo:

- Los aprobadores reciben una notificación por correo cuando una solicitud de un rol está pendiente de su revisión. Las notificaciones por correo electrónico incluyen un vínculo directo a la solicitud donde el aprobador puede aprobarla o rechazarla.
- Las solicitudes las resuelve el primer aprobador que aprueba o rechaza.
- Cuando un aprobador responde a la solicitud, se notifica a todos los aprobadores de la acción.
- Los administradores globales y los administradores de roles con privilegios reciben una notificación cuando un usuario aprobado se activa en su rol.

>[!NOTE]
>Un administrador global o un administrador de roles con privilegios que cree que un usuario aprobado no debe estar activo puede quitar la asignación de roles activa en Privileged Identity Management. Aunque los administradores no reciben notificaciones de solicitudes pendientes a menos que sean aprobadores, pueden ver y cancelar todas las solicitudes pendientes de todos los usuarios, para lo cual deben dirigirse a las solicitudes pendientes en Privileged Identity Management.

# <a name="previous-version"></a>[Versión anterior](#tab/previous)

## <a name="view-pending-requests"></a>Ver solicitudes en espera

Como aprobador delegado, recibirá una notificación por correo electrónico cuando una solicitud de rol de Azure AD está pendiente de su aprobación. Puede ver estas solicitudes pendientes en Privileged Identity Management.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Abra **Azure AD Privileged Identity Management**.

1. Haga clic en **Roles de Azure AD**.

1. Haga clic en **Aprobar solicitudes**.

    ![Roles de Azure AD: Aprobar solicitudes](./media/azure-ad-pim-approval-workflow/approve-requests.png)

    Verá una lista de solicitudes pendientes de su aprobación.

## <a name="approve-requests"></a>Aprobar solicitudes

1. Seleccione las solicitudes que quiera aprobar y luego haga clic en **Aprobar** para abrir el panel Aprobar las solicitudes seleccionadas.

    ![Lista de Aprobar solicitudes con la opción Aprobar resaltada](./media/azure-ad-pim-approval-workflow/pim-approve-requests-list.png)

1. En el cuadro **Razón de la aprobación**, escriba una razón.

    ![Panel de Aprobar las solicitudes seleccionadas con una razón de la aprobación](./media/azure-ad-pim-approval-workflow/pim-approve-selected-requests.png)

1. Haga clic en **Approve** (Aprobar).

    El símbolo de estado se actualizará con la aprobación.

    ![Panel de Aprobar las solicitudes seleccionadas tras hacer clic en el botón Aprobar](./media/azure-ad-pim-approval-workflow/pim-approve-status.png)

## <a name="deny-requests"></a>Denegar solicitudes

1. Seleccione las solicitudes que quiera denegar y luego haga clic en **Denegar** para abrir el panel Denegar las solicitudes seleccionadas.

    ![Lista de Aprobar solicitudes con la opción Denegar resaltada](./media/azure-ad-pim-approval-workflow/pim-deny-requests-list.png)

1. En el cuadro **Razón de la denegación**, escriba una razón.

    ![Panel de Aprobar las solicitudes seleccionadas con una razón de la denegación](./media/azure-ad-pim-approval-workflow/pim-deny-selected-requests.png)

1. Seleccione **Denegar**.

    El símbolo de estado se actualizará con la denegación.

---

## <a name="next-steps"></a>Pasos siguientes

- [Notificaciones por correo electrónico en Privileged Identity Management](pim-email-notifications.md)
- [Aprobación o rechazo de solicitudes para los roles de Azure en Privileged Identity Management](pim-resource-roles-approval-workflow.md)
