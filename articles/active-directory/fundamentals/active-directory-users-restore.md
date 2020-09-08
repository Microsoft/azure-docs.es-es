---
title: 'Restauración o eliminación permanente de usuarios eliminados recientemente: Azure AD'
description: Visualización de los usuarios que se pueden restaurar, restauración de un usuario eliminado o eliminación permanente de un usuario en Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 04/01/2018
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 718a01abacf6f2c3cbc9523cc644aca02dc5ff49
ms.sourcegitcommit: 5ed504a9ddfbd69d4f2d256ec431e634eb38813e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/02/2020
ms.locfileid: "89321198"
---
# <a name="restore-or-remove-a-recently-deleted-user-using-azure-active-directory"></a>Restauración o eliminación de un usuario recientemente eliminado mediante Azure Active Directory
Después de eliminar a un usuario, la cuenta permanece en estado de suspensión durante 30 días. Durante ese período de 30 días, la cuenta de usuario se puede restaurar, junto con todas sus propiedades. Después de que pase esa ventana de 30 días, el usuario se elimina automáticamente y de forma permanente.

Puede ver a los usuarios que se pueden restaurar, restaurar un usuario eliminado o eliminar permanentemente a un usuario con Azure Active Directory (Azure AD) en Azure Portal.

>[!Important]
>Ni usted ni la asistencia técnica de Microsoft pueden restaurar a un usuario eliminado permanentemente.

## <a name="required-permissions"></a>Permisos necesarios
Debe tener uno de los roles siguientes para restaurar y eliminar permanentemente a los usuarios.

- Administrador global

- Soporte para asociados de nivel 1

- Soporte para asociados de nivel 2

- Administrador de usuarios

## <a name="view-your-restorable-users"></a>Visualización de los usuarios que se pueden restaurar
Puede ver a todos los usuarios que se eliminaron hace menos de 30 días. Estos usuarios se pueden restaurar.

### <a name="to-view-your-restorable-users"></a>Para ver a los usuarios que se pueden restaurar
1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con una cuenta de administrador global para la organización.

2. Seleccione **Azure Active Directory**, **Usuarios** y, a continuación, seleccione **Usuarios eliminados**.

    Revise la lista de usuarios que están disponibles para restaurar.

    ![Página Usuarios - usuarios eliminados, con usuarios que aún se pueden restaurar](media/active-directory-users-restore/users-deleted-users-view-restorable.png)

## <a name="restore-a-recently-deleted-user"></a>Restauración de un usuario recién eliminado

Al eliminarse una cuenta de usuario de la organización, esta está en estado suspendido y se conserva toda la información de la organización relacionada. Cuando se restaura un usuario, también se restaura esta información de la organización.

> [!Note]
> Una vez que se restaura un usuario, también se restauran las licencias que se asignaron al usuario en el momento de la eliminación aunque no haya puestos disponibles para esas licencias. Si a partir de ese momento consume más licencias de las que adquirió, su organización podría incumplir temporalmente todo lo relativo al uso de licencias.

### <a name="to-restore-a-user"></a>Para restaurar a un usuario
1. En la página **Usuarios - usuarios eliminados**, busque y seleccione uno de los usuarios disponibles. Por ejemplo, _Mary Parker_.

2. Seleccione **Restaurar usuario**.

    ![Página Usuarios - usuarios eliminados, con la opción Restaurar usuario resaltada](media/active-directory-users-restore/users-deleted-users-restore-user.png)

## <a name="permanently-delete-a-user"></a>Eliminar un usuario permanentemente
Puede eliminar permanentemente un usuario desde la organización sin esperar a que transcurran los 30 días de la eliminación automática. Ni usted, ni otro administrador, ni la asistencia técnica de Microsoft pueden restaurar a un usuario eliminado permanentemente.

>[!Note]
>Si se elimina permanentemente a un usuario por error, tendrá que crear un nuevo usuario y escribir manualmente toda la información anterior. Para más información acerca de cómo crear un usuario, consulte [cómo agregar o eliminar usuarios](add-users-azure-active-directory.md).

### <a name="to-permanently-delete-a-user"></a>Para eliminar un usuario permanentemente

1. En la página **Usuarios - usuarios eliminados**, busque y seleccione uno de los usuarios disponibles. Por ejemplo, _Rae Huff_.

2. Seleccione **Eliminar permanentemente**.

    ![Página Usuarios - usuarios eliminados, con la opción Restaurar usuario resaltada](media/active-directory-users-restore/users-deleted-users-permanent-delete-user.png)

## <a name="next-steps"></a>Pasos siguientes
Después de restaurar o eliminar a los usuarios, puede realizar los siguientes procesos básicos:

- [Adición o eliminación de usuarios](add-users-azure-active-directory.md)

- [Asignación de roles a usuarios](active-directory-users-assign-role-azure-portal.md)

- [Add or change profile information](active-directory-users-profile-azure-portal.md) (Incorporación o modificación de la información del perfil)

- [Adición de usuarios invitados de otra organización](../external-identities/what-is-b2b.md)

Para obtener más información acerca de otras tareas de administración de usuarios disponibles, consulte la [documentación de administración de usuarios en Azure AD](../users-groups-roles/index.yml).