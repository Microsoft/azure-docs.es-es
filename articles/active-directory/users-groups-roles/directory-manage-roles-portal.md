---
title: 'Visualización y asignación de permisos de roles de administrador: Azure AD | Microsoft Docs'
description: Ahora puede ver y administrar los miembros de un rol de administrador de Azure AD en el portal. Para aquellos que administran con frecuencia las asignaciones de roles.
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: how-to
ms.date: 06/15/2020
ms.author: curtand
ms.reviewer: vincesm
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: fea05ada52ba4281acb3451769446d7f0bad790d
ms.sourcegitcommit: cee72954f4467096b01ba287d30074751bcb7ff4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/30/2020
ms.locfileid: "87448701"
---
# <a name="view-and-assign-administrator-roles-in-azure-active-directory"></a>Visualización y asignación de roles de administrador en Azure Active Directory

Ahora puede ver y administrar todos los miembros de los roles de administrador en el portal de Azure Active Directory. Si administra las asignaciones de roles con frecuencia, probablemente preferirá esta experiencia. Y si alguna vez se ha preguntado "¿qué es lo que realmente hacen estos roles?", puede ver una lista detallada de los permisos para cada uno de los roles de administrador de Azure AD.

## <a name="view-all-roles"></a>Ver todos los roles

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra **Azure Active Directory**.

1. Seleccione **Roles y administradores** para ver la lista de todos los roles disponibles.

1. Seleccione los puntos suspensivos a la derecha de cada fila para ver los permisos del rol. Seleccione un rol para ver los usuarios asignados a él. Si lo que ve no concuerda con la siguiente imagen, lea la nota de [Visualización de asignaciones de roles con privilegios](#view-assignments-for-privileged-roles) para comprobar si se encuentra en Privileged Identity Management (PIM).

    ![lista de roles en el portal de Azure AD](./media/directory-manage-roles-portal/view-roles-in-azure-active-directory.png)

## <a name="view-my-roles"></a>Ver mis roles

También es fácil ver sus propios permisos. Seleccione **su rol** en la página **Roles y administradores** para ver los roles que tiene asignados actualmente.

## <a name="view-assignments-for-privileged-roles"></a>Visualización de asignaciones de roles con privilegios

Puede hacer clic en **Administrar en PIM** para obtener funcionalidades de administración adicionales. Los administradores de roles con privilegios pueden cambiar las asignaciones "Permanentes" (siempre activas en el rol) a "Elegibles" (en el rol solo cuando se elevan). Si no tiene Privileged Identity Management, todavía puede seleccionar **Administrar en PIM** para registrarse en una versión de prueba. Privileged Identity Management requiere un [Plan de licencias de Azure AD Premium P2](../privileged-identity-management/subscription-requirements.md).

![lista de miembros de un rol de administrador](./media/directory-manage-roles-portal/member-list.png)

Si es un administrador global o un administrador de roles con privilegios, puede agregar o quitar miembros, filtrar la lista o seleccionar un miembro fácilmente para ver sus roles asignados activos.

> [!Note]
> Si tiene una licencia Azure AD Premium P2 y ya usa Privileged Identity Management, todas las tareas de administración de roles se realizan ahí y no en Azure AD.
>
> ![Roles de Azure AD administrados en PIM para los usuarios que ya usan PIM y tienen una licencia Premium P2](./media/directory-manage-roles-portal/pim-manages-roles-for-p2.png)

## <a name="view-a-users-role-permissions"></a>Ver permisos de rol de un usuario

Cuando esté viendo los miembros de un rol, haga clic en **Descripción** para ver una lista completa de los permisos otorgados por la asignación de roles. En la página se incluyen vínculos a documentación relevante que le guiarán a través de la administración de los roles de directorio.

![lista de permisos para un rol de administrador](./media/directory-manage-roles-portal/role-description.png)

## <a name="download-role-assignments"></a>Descarga de asignaciones de rol

Para descargar todas las asignaciones de un rol específico, en la página **Roles y administradores**, seleccione un rol y luego **Descargar asignaciones de roles**. Se descargará un archivo CSV en el que se enumeran las asignaciones en todos los ámbitos para ese rol.

![Descarga de todas las asignaciones de un rol](./media/directory-manage-roles-portal/download-role-assignments.png)

## <a name="assign-a-role"></a>Asignar un rol

1. Inicie sesión en [Azure Portal](https://portal.azure.com) con los permisos de administrador global o administrador de roles con privilegios y seleccione **Azure Active Directory**.

1. Seleccione **Roles y administradores** para ver la lista de todos los roles disponibles.

1. Seleccione un rol para ver sus asignaciones.

    ![lista de permisos para un rol de administrador](./media/directory-manage-roles-portal/member-list.png)

1. Seleccione **Agregar asignaciones** y elija los roles que quiere asignar. Puede hacer clic en **Administrar en PIM** para obtener funcionalidades de administración adicionales. Si lo que ve no concuerda con la siguiente imagen, lea la nota de [Visualización de asignaciones de roles con privilegios](#view-assignments-for-privileged-roles) para comprobar si se encuentra en PIM.

    ![lista de permisos para un rol de administrador](./media/directory-manage-roles-portal/directory-role-select-role.png)

## <a name="next-steps"></a>Pasos siguientes

* No dude en compartir con nosotros en el [foro de roles administrativos de Azure AD](https://feedback.azure.com/forums/169401-azure-active-directory?category_id=166032).
* Para obtener más información sobre los roles y la asignación de roles de administrador, consulte el artículo [Assign administrator roles](directory-assign-admin-roles.md) (Asignación de roles de administrador).
* Para conocer los permisos predeterminados de usuario, vea una [comparación de los permisos predeterminados de usuario miembro e invitado](../fundamentals/users-default-permissions.md).
