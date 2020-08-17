---
title: 'Asignación de roles de recursos de Azure en Privileged Identity Management: Azure Active Directory | Microsoft Docs'
description: Aprenda a asignar roles de recursos de Azure en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: mtillman
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 07/01/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 32b678058e2d24d277eaf56a1aadec3678e25a5b
ms.sourcegitcommit: 0e8a4671aa3f5a9a54231fea48bcfb432a1e528c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/24/2020
ms.locfileid: "87128172"
---
# <a name="assign-azure-resource-roles-in-privileged-identity-management"></a>Asignación de roles de recursos de Azure en Privileged Identity Management

Azure Active Directory (Azure AD) Privileged Identity Management (PIM) puede administrar los roles de recurso integrados de Azure, así como los roles personalizados, entre ellos los siguientes:

- Propietario
- Administrador de acceso de usuario
- Colaborador
- Administrador de seguridad
- Administrador de seguridad

> [!NOTE]
> Los usuarios o los miembros de un grupo asignados a los roles de suscripción Propietario o Administrador de acceso de usuario y los administradores globales de Azure AD que permiten la administración de suscripciones en Azure AD tienen permisos de administrador de recursos de forma predeterminada. Estos administradores pueden asignar roles, configurar opciones de rol y revisar el acceso con Privileged Identity Management a los recursos de Azure. Un usuario no puede administrar Privileged Identity Management para recursos sin permisos de administrador de recursos. Consulte la lista de [roles integrados de Azure](../../role-based-access-control/built-in-roles.md).

## <a name="assign-a-role"></a>Asignar un rol

Siga estos pasos para hacer que un usuario sea elegible para un rol de directorio de Azure AD.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un usuario que sea miembro del rol [Administrador de roles con privilegios](../users-groups-roles/directory-assign-admin-roles.md#privileged-role-administrator).

    Para información sobre cómo conceder acceso a otro administrador para gestionar Privileged Identity Management, consulte [Concesión de acceso a otros administradores para administrar Privileged Identity Management](pim-how-to-give-access-to-pim.md).

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Azure resources** (Recursos de Azure).

1. Use el filtro de recursos para buscar los recursos administrados que está buscando.

    ![Lista de recursos de Azure para administrar](./media/pim-resource-roles-assign-roles/resources-list.png)

1. Seleccione el recurso que desea administrar para abrir la página de información general de recursos.

1. En **Administrar**, seleccione **Roles** para ver la lista de roles de los recursos de Azure.

    ![Roles de recursos de Azure](./media/pim-resource-roles-assign-roles/resources-roles.png)

1. Seleccione **Agregar asignaciones** para abrir el panel **Agregar asignaciones**.

1. Elija **Seleccionar un rol** para abrir la página **Seleccionar un rol**.

    ![Panel Nueva asignación](./media/pim-resource-roles-assign-roles/resources-select-role.png)

1. Seleccione el rol que quiera asignar y luego haga clic en **Seleccionar**.

    Se abre el panel **Seleccionar un miembro o grupo**.

1. Seleccione el miembro o grupo que quiera asignar al rol y luego haga clic en **Seleccionar**.

    ![Panel Seleccionar un miembro o grupo](./media/pim-resource-roles-assign-roles/resources-select-member-or-group.png)

1. En la pestaña **Configuración**, en la lista **Tipo de asignación**, seleccione **Apto** o **Activo**.

    ![Panel Configuración de pertenencia](./media/pim-resource-roles-assign-roles/resources-membership-settings-type.png)

    Privileged Identity Management para recursos de Azure ofrece dos tipos de asignación distintos:

    - Las asignaciones tipo **Apto** requieren que el miembro del rol realice una acción para usar el rol. Entre las acciones se puede incluir realizar una comprobación de autenticación multifactor (MFA), proporcionar una justificación de negocios o solicitar la aprobación de los aprobadores designados.

    - Las asignaciones tipo **Activo** no requieren que el miembro realice ninguna acción para usar el rol. Los miembros asignados como activos tienen siempre los privilegios asignados al rol.

1. Para especificar una duración de asignación específica, cambie las fechas y horas de inicio y finalización.

1. Cuando termine, seleccione **Asignar**.

1. Una vez creada la nueva asignación de roles, se muestra una notificación de estado.

    ![Nueva asignación: notificación](./media/pim-resource-roles-assign-roles/resources-new-assignment-notification.png)

## <a name="update-or-remove-an-existing-role-assignment"></a>Actualizar o quitar una asignación de roles existente

Siga estos pasos para actualizar o quiotar una asignación de roles existente.

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Azure resources** (Recursos de Azure).

1. Seleccione el recurso que desea administrar para abrir su página de información general.

1. En **Administrar**, seleccione **Roles** para ver la lista de roles de los recursos de Azure.

    ![Roles de recursos de Azure: seleccionar rol](./media/pim-resource-roles-assign-roles/resources-update-select-role.png)

1. Seleccione el rol que quiera actualizar o quitar.

1. Busque la asignación de roles en las pestañas **Roles elegibles** o **Roles activos**.

    ![Actualizar o quitar la asignación de roles](./media/pim-resource-roles-assign-roles/resources-update-remove.png)

1. Seleccione **Actualizar** o **Quitar** para actualizar o quitar la asignación de roles.

    Para obtener información sobre cómo ampliar una asignación de roles, consulte [Ampliación o renovación de roles de recursos de Azure en Privileged Identity Management](pim-resource-roles-renew-extend.md).

## <a name="next-steps"></a>Pasos siguientes

- [Ampliación o renovación de roles de recursos de Azure en Privileged Identity Management](pim-resource-roles-renew-extend.md)
- [Configuración de las opciones de rol de recursos de Azure en Privileged Identity Management](pim-resource-roles-configure-role-settings.md)
- [Asignación de roles de Azure AD en Privileged Identity Management](pim-how-to-add-role-to-user.md)
