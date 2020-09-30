---
title: 'Asignación de roles de Azure AD a usuarios: Azure Active Directory | Microsoft Docs'
description: Instrucciones acerca de cómo asignar roles de administrador y de no administrador a usuarios con Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 08/31/2020
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: fb7ab83bc9939d2f0b4b0ff0860ea97a0b07f12f
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321249"
---
# <a name="assign-administrator-and-non-administrator-roles-to-users-with-azure-active-directory"></a>Asignación de roles de administrador y de no administrador a usuarios con Azure Active Directory

En Azure Active Directory (Azure AD), si uno de los usuarios necesita permiso para administrar recursos de Azure AD, debe asignarlo a un rol que proporcione los permisos que necesita. Para obtener información sobre qué roles administran los recursos de Azure y qué roles administran los recursos de Azure AD, consulte [Roles de administrador de la suscripción clásica, roles de Azure y roles de Azure AD](../../role-based-access-control/rbac-and-directory-admin-roles.md).

Para más información acerca de los roles de Azure AD disponibles, consulte [Asignación de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md). Para agregar usuarios, consulte [Incorporación de nuevos usuarios en Azure Active Directory](add-users-azure-active-directory.md).

## <a name="assign-roles"></a>Asignación de roles

Comúnmente, se asignan roles de Azure AD a los usuarios desde la página **Roles asignados** de un usuario. Puede igualmente configurar que se eleve la idoneidad del usuario cuando sea necesario a un rol mediante Privileged Identity Management (PIM). Para obtener más información sobre cómo usar PIM, consulte [Privileged Identity Management](../privileged-identity-management/index.yml).

> [!Note]
> Si tiene un plan de licencias de Azure AD Premium P2 y ya usa PIM, todas las tareas de administración de roles se realizan en la [experiencia de Privileged Identity Management](../users-groups-roles/directory-manage-roles-portal.md). Esta característica está limitada actualmente a la asignación de un rol a la vez. Actualmente no se pueden seleccionar varios roles y asignarlos a un usuario a la vez.
>
> ![Roles de Azure AD administrados en PIM para los usuarios que ya usan PIM y tienen una licencia Premium P2](./media/active-directory-users-assign-role-azure-portal/pim-manages-roles-for-p2.png)

## <a name="assign-a-role-to-a-user"></a>Asignar de un rol a un usuario

1. Vaya a [Azure Portal](https://portal.azure.com/) e inicie sesión con una cuenta de administrador global para el directorio.

2. Busque y seleccione **Azure Active Directory**.

      ![Búsqueda de Azure Active Directory en Azure Portal](media/active-directory-users-assign-role-azure-portal/search-azure-active-directory.png)

3. Seleccione **Usuarios**.

4. Busque y seleccione el usuario que obtiene la asignación de roles. Por ejemplo, _Alain Charon_.

      ![Página Todos los usuarios, selección del usuario](media/active-directory-users-assign-role-azure-portal/directory-role-select-user.png)

5. En la página **Alain Charon - Perfil**, seleccione **Roles asignados**.

    Aparece la página **Alain Charon - Roles administrativos**.

6. Seleccione **Agregar asignaciones**, seleccione el rol que asignará a Alain (por ejemplo, _Administrador de aplicaciones_) y, a continuación, elija **Seleccionar**.

    ![Página Roles asignados, se muestra el rol seleccionado](media/active-directory-users-assign-role-azure-portal/directory-role-select-role.png)

    El rol Administrador de aplicaciones se asigna a Alain Charon y se muestra en la página **Alain Charon - Roles administrativos**.

## <a name="remove-a-role-assignment"></a>Eliminación de una asignación de rol

Si necesita quitar la asignación de roles de un usuario, también puede hacerlo desde la página **Alain Charon - Roles administrativos**.

### <a name="to-remove-a-role-assignment-from-a-user"></a>Para quitar una asignación de rol a un usuario

1. Seleccione **Azure Active Directory**, seleccione **Usuarios** y, a continuación, busque y seleccione los usuarios a los que quitará una asignación de roles. Por ejemplo, _Alain Charon_.

2. Seleccione **Roles asignados**, seleccione **Administrador de aplicaciones** y, luego, **Quitar asignación**.

    ![Página Roles asignados, se muestra el rol seleccionado y la opción para quitarlo](media/active-directory-users-assign-role-azure-portal/directory-role-remove-role.png)

    El rol Administrador de aplicaciones se quita de Alain Charon y ya no se muestra en la página **Alain Charon - Roles administrativos**.

## <a name="next-steps"></a>Pasos siguientes

- [Adición o eliminación de usuarios](add-users-azure-active-directory.md)

- [Add or change profile information](active-directory-users-profile-azure-portal.md) (Incorporación o modificación de la información del perfil)

- [Adición de usuarios invitados de otro directorio](../external-identities/what-is-b2b.md)

Consulte la [documentación de administración de usuarios en Azure Active Directory](../users-groups-roles/index.yml) para ver otras tareas de administración de usuarios en detalle.