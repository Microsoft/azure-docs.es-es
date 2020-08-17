---
title: 'Asignación de roles de Azure AD en PIM: Azure Active Directory | Microsoft Docs'
description: Aprenda a asignar roles de Azure AD en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 07/01/2020
ms.author: curtand
ms.collection: M365-identity-device-management
ms.openlocfilehash: a3370ea52c5bd189f929c0f81a0aa9b59cc77b97
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87421389"
---
# <a name="assign-azure-ad-roles-in-privileged-identity-management"></a>Asignación de roles de Azure AD en Privileged Identity Management

Con Azure Active Directory (Azure AD), un administrador global puede realizar asignaciones de roles de administrador de Azure AD **permanentes**. Estas asignaciones de roles se pueden crear mediante [Azure Portal](../users-groups-roles/directory-assign-admin-roles.md) o mediante [comandos de PowerShell](/powershell/module/azuread#directory_roles).

El servicio Azure AD Privileged Identity Management (PIM) permite también a los administradores de roles con privilegios realizar asignaciones de roles de administrador permanentes. Además, los administradores de rol con privilegios pueden hacer que los usuarios sean **aptos** para roles de administrador de Azure AD. Un administrador apto puede activar el rol cuando lo necesite y, cuando termina, sus permisos caducan.

## <a name="determine-your-version-of-pim"></a>Determinar la versión de PIM

Desde noviembre de 2019, la parte de roles de Azure AD de Privileged Identity Management se está actualizando a una nueva versión que coincide con las experiencias de los roles de recursos de Azure. Esta actualización introduce características adicionales y [cambios en la API existente](azure-ad-roles-features.md#api-changes). Mientras se implementa la nueva versión, los procedimientos que seguirá en este artículo dependerán de la versión de Privileged Identity Management que tenga actualmente. Siga los pasos de esta sección para determinar la versión de Privileged Identity Management que tiene. Cuando averigüe la versión de Privileged Identity Management, puede seleccionar los procedimientos de este artículo que coincidan con esa versión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un usuario que tenga el rol [Administrador de roles con privilegios](../users-groups-roles/directory-assign-admin-roles.md#privileged-role-administrator).
1. Abra **Azure AD Privileged Identity Management**. Si tiene un banner en la parte superior de la página de introducción, siga las instrucciones de la pestaña **Nueva versión** de este artículo. De lo contrario, siga las instrucciones de la pestaña **Versión anterior**.

  [![Selección de Azure AD > Privileged Identity Management](media/pim-how-to-add-role-to-user/pim-new-version.png)](media/pim-how-to-add-role-to-user/pim-new-version.png#lightbox)

# <a name="new-version"></a>[Nueva versión](#tab/new)

## <a name="assign-a-role"></a>Asignar un rol

Siga estos pasos para hacer que un usuario sea elegible para un rol de administrador de Azure AD.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un usuario que sea miembro del rol [Administrador de roles con privilegios](../users-groups-roles/directory-assign-admin-roles.md#privileged-role-administrator).

    Para información sobre cómo conceder acceso a otro administrador para gestionar Privileged Identity Management, consulte [Concesión de acceso a otros administradores para administrar Privileged Identity Management](pim-how-to-give-access-to-pim.md).

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Roles de Azure AD**.

1. Seleccione **Roles** para ver la lista de roles de los permisos de Azure AD.

    ![Roles de Azure AD](./media/pim-how-to-add-role-to-user/roles-list.png)

1. Seleccione **Agregar asignaciones** para abrir la página **Agregar asignaciones**.

1. Elija **Seleccionar un rol** para abrir la página **Seleccionar un rol**.

    ![Panel Nueva asignación](./media/pim-how-to-add-role-to-user/select-role.png)

1. Seleccione el rol que desea asignar, seleccione un miembro al que desee asignar el rol y luego, seleccione **Siguiente**.

1. En la lista **Tipo de asignación** en el panel **Configuración de pertenencia**, seleccione **Apto** o **Activo**.

    - Las asignaciones tipo **Apto** requieren que el miembro del rol realice una acción para usar el rol. Entre las acciones se puede incluir realizar una comprobación de autenticación multifactor (MFA), proporcionar una justificación de negocios o solicitar la aprobación de los aprobadores designados.

    - Las asignaciones tipo **Activo** no requieren que el miembro realice ninguna acción para usar el rol. Los miembros asignados como activos tienen siempre los privilegios asignados al rol.

1. Para especificar una duración de asignación específica, agregue un cuadro de fecha y hora de inicio y de finalización. Cuando termine, seleccione **Asignar** para crear la nueva asignación de roles.

    ![Configuración de pertenencias: fecha y hora](./media/pim-how-to-add-role-to-user/start-and-end-dates.png)

1. Una vez asignado el rol, se muestra una notificación de estado de la asignación.

    ![Nueva asignación: notificación](./media/pim-how-to-add-role-to-user/assignment-notification.png)

## <a name="update-or-remove-an-existing-role-assignment"></a>Actualizar o quitar una asignación de roles existente

Siga estos pasos para actualizar o quiotar una asignación de roles existente.

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Roles de Azure AD**.

1. Seleccione **Roles** para ver la lista de roles de Azure AD.

1. Seleccione el rol que quiera actualizar o quitar.

1. Busque la asignación de roles en las pestañas **Roles elegibles** o **Roles activos**.

    ![Actualizar o quitar la asignación de roles](./media/pim-how-to-add-role-to-user/remove-update-assignments.png)

1. Seleccione **Actualizar** o **Quitar** para actualizar o quitar la asignación de roles.

# <a name="previous-version"></a>[Versión anterior](#tab/previous)

## <a name="make-a-user-eligible-for-a-role"></a>Hacer que un usuario sea apto para un rol

Siga estos pasos para hacer que un usuario sea elegible para un rol de administrador de Azure AD.

1. Seleccione **Roles** o **Miembros**.

    ![Roles de Azure AD](./media/pim-how-to-add-role-to-user/pim-directory-roles.png)

1. Seleccione **Agregar miembro** para abrir **Agregar miembros administrados**.

1. Seleccione **Seleccionar un rol**, seleccione un rol que desee administrar y luego **Seleccionar**.

    ![Seleccione un rol.](./media/pim-how-to-add-role-to-user/pim-select-a-role.png)

1. Seleccione **Seleccionar miembros**, seleccione los usuarios que desea asignar al rol y, a continuación, **Seleccionar**.

    ![Seleccione un rol.](./media/pim-how-to-add-role-to-user/pim-select-members.png)

1. En **Agregar miembros administrados**, seleccione **Aceptar** para agregar el usuario al rol.

1. En la lista de roles, seleccione el que acaba de asignar para ver la lista de miembros.

     Cuando el rol esté asignado, el usuario que ha seleccionado aparecerá en la lista de miembros como **Elegible** para el rol.

    ![Usuario apto para un rol](./media/pim-how-to-add-role-to-user/pim-directory-role-eligible.png)

1. Ahora que el usuario es apto para el rol, hágale saber que puede activarlo de acuerdo con las instrucciones que se describen en [Activación de mis roles de Azure AD en Privileged Identity Management](pim-how-to-activate-role.md).

    Se pide a los administradores válidos que se registren en Azure Multi-Factor Authentication durante la activación. Si un usuario no puede registrarse en MFA o usa una cuenta de Microsoft (como @outlook.com), deberá establecerlo como permanente en todos sus roles.

## <a name="make-a-role-assignment-permanent"></a>Hacer que una asignación de roles sea permanente

De forma predeterminada, los nuevos usuarios solo son *aptos* para un rol de administrador de Azure AD. Siga estos pasos si desea hacer que una asignación de roles sea permanente.

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Roles de Azure AD**.

1. Seleccione **Miembros**.

    ![Lista de miembros](./media/pim-how-to-add-role-to-user/pim-directory-role-list-members.png)

1. Seleccione en un rol **Apto** que desee convertir en permanente.

1. Seleccione **Más** y, después, seleccione **Establecer como permanente**.

    ![Hacer que una asignación de roles sea permanente](./media/pim-how-to-add-role-to-user/pim-make-perm.png)

    El rol aparece ahora como **permanente**.

    ![Lista de miembros con el cambio permanente](./media/pim-how-to-add-role-to-user/pim-directory-role-list-members-permanent.png)

## <a name="remove-a-user-from-a-role"></a>Eliminación de un usuario de un rol

Puede quitar a los usuarios de las asignaciones de roles, pero asegúrese de que siempre haya al menos un usuario que sea un administrador global permanente. Si no está seguro de si los usuarios necesitan aún sus asignaciones de roles, puede [iniciar una revisión de acceso del rol](pim-how-to-start-security-review.md).

Siga estos pasos para quitar a un usuario específico de un rol de administrador de Azure AD.

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Roles de Azure AD**.

1. Seleccione **Miembros**.

    ![Lista de miembros](./media/pim-how-to-add-role-to-user/pim-directory-role-list-members.png)

1. Seleccione una asignación de roles que desee quitar.

1. Seleccione **Más** y, después, elija **Quitar**.

    ![Quitar un rol](./media/pim-how-to-add-role-to-user/pim-remove-role.png)

1. En el mensaje que le pide confirmación, seleccione **Sí**.

    ![Quitar un rol](./media/pim-how-to-add-role-to-user/pim-remove-role-confirm.png)

    La asignación de rol se quita.

## <a name="authorization-error-when-assigning-roles"></a>Error de autorización al asignar roles

Si recientemente habilitó Privileged Identity Management para una suscripción y obtiene un error de autorización cuando intenta que un usuario pueda optar a un rol de administrador de Azure AD, es posible que la entidad de servicio MS-PIM aún no tenga los permisos adecuados. La entidad de servicio MS-PIM debe tener el rol [Administrador de acceso de usuario](../../role-based-access-control/built-in-roles.md#user-access-administrator) para asignar roles a otros usuarios. En lugar de esperar hasta que se asigne a MS-PIM el rol de administrador de acceso de usuario, puede asignarlo manualmente.

Siga estos pasos para asignar el rol de administrador de acceso de usuario a la entidad de servicio de MS-PIM para una suscripción.

1. Inicie sesión en Azure Portal como administrador global.

1. Elija **Todos los servicios** y, después, **Suscripciones**.

1. Elija su suscripción.

1. Haga clic en **Control de acceso (IAM)** .

1. Elija **Role assignments** (Asignación de roles) para ver la lista actual de las asignaciones de roles en el ámbito de la suscripción.

   ![Hoja Control de acceso (IAM) para una suscripción](./media/pim-how-to-add-role-to-user/ms-pim-access-control.png)

1. Compruebe si la entidad de servicio **MS PIM** tiene asignado el rol **Administrador de acceso de usuario**.

1. En caso contrario, elija **Agregar asignación de roles** para abrir el panel **Agregar asignación de roles**.

1. En la lista desplegable **Rol**, seleccione el rol **Administrador de acceso de usuario**.

1. En la lista **Seleccionar**, busque y seleccione la entidad de servicio **MS PIM**.

   ![Panel Agregar asignación de roles: adición de permisos para la entidad de servicio MS-PIM](./media/pim-how-to-add-role-to-user/ms-pim-add-permissions.png)

1. Elija **Guardar** para asignar el rol.

   Después de unos momentos, se asignará a la entidad de servicio MS-PIM el rol Administrador de acceso de usuario en el ámbito de la suscripción.

   ![Página de control de acceso que muestra la asignación de roles de administrador de acceso de usuario para la entidad de servicio MS-PIM](./media/pim-how-to-add-role-to-user/ms-pim-user-access-administrator.png)

 ---

## <a name="next-steps"></a>Pasos siguientes

- [Configuración del rol de administrador de Azure AD en Privileged Identity Management](pim-how-to-change-default-settings.md)
- [Asignación de roles de recursos de Azure en Privileged Identity Management](pim-resource-roles-assign-roles.md)