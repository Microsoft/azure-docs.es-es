---
title: 'Incorporación o eliminación de usuarios: Azure Active Directory | Microsoft Docs'
description: Instrucciones sobre cómo agregar usuarios nuevos o eliminar usuarios existentes con Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: fundamentals
ms.topic: how-to
ms.date: 11/12/2019
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: b6d8e6ed412a0e71e8c85827cfc8fc05cd744e3d
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87797107"
---
# <a name="add-or-delete-users-using-azure-active-directory"></a>Incorporación o eliminación de usuarios mediante Azure Active Directory

Agregue usuarios nuevos o elimine usuarios existentes desde la organización de Azure Active Directory (Azure AD). Para agregar o eliminar usuarios debe ser administrador de usuarios o administrador de empresa.

## <a name="add-a-new-user"></a>Agregar un nuevo usuario

Puede crear un nuevo usuario con el portal de Azure Active Directory.

Para agregar un usuario, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de usuarios para la organización.

1. Busque y seleccione *Azure Active Directory* en cualquier página.

1. Seleccione **Usuarios** y, a continuación, seleccione **Nuevo usuario**.

    ![Agregar un usuario a través de Usuarios - Todos los usuarios en Azure AD](media/add-users-azure-active-directory/add-user-in-users-all-users.png)

1. En la página **Usuario**, escriba información para este usuario:

   - **Nombre**. Necesario. Nombre y apellidos del nuevo usuario. Por ejemplo, *Mary Parker*.

   - **Nombre de usuario**. Necesario. Nombre de usuario del nuevo usuario. Por ejemplo, `mary@contoso.com`.

     La parte del dominio del nombre de usuario debe usar el nombre de dominio predeterminado inicial, *\<yourdomainname>.onmicrosoft.com* o un nombre de dominio personalizado, como de *contoso.com*. Para más información sobre cómo crear un nombre de dominio personalizado, consulte [Incorporación del nombre de dominio personalizado mediante el portal de Azure Active Directory](add-custom-domain.md).

   - **Grupos**. Si quiere, puede agregar el usuario a uno o varios de los grupos existentes. También puede agregar el usuario a grupos en un momento posterior. Para más información sobre cómo agregar usuarios a grupos, consulte [Creación de un grupo básico e incorporación de miembros con Azure Active Directory](active-directory-groups-create-azure-portal.md).

   - **Rol del directorio**. Si necesita permisos administrativos de Azure AD para el usuario, puede agregarlos a un rol de Azure AD. Puede asignar el rol de administrador global al usuario, o uno o varios de los otros roles de administrador limitados de Azure AD. Para obtener más información sobre la asignación de roles, consulte [Asignación de roles a usuarios](active-directory-users-assign-role-azure-portal.md).

   - **Información del trabajo**. Puede agregar más información sobre el usuario aquí o hacerlo más adelante. Para obtener más información sobre cómo agregar información de usuario, vea [How to add or change user profile information](active-directory-users-profile-azure-portal.md) (Incorporación o modificación de la información del perfil de usuario).

1. Copie la contraseña generada automáticamente proporcionada en el cuadro de texto **Contraseña**. Deberá proporcionar esta contraseña al usuario para iniciar sesión por primera vez.

1. Seleccione **Crear**.

El usuario se crea y se agrega a la organización de Azure AD.

## <a name="add-a-new-guest-user"></a>Incorporación de un usuario invitado nuevo

También puede invitar a un usuario invitado nuevo a colaborar con su organización si selecciona **Invitar usuario** en la página **Nuevo usuario**. Si la configuración de colaboración externa de la organización se configura de modo tal que se le permite invitar a otros usuarios, el usuario recibirá una invitación por correo electrónico que debe aceptar para empezar a colaborar. Para más información sobre cómo invitar a usuarios de colaboración B2B, consulte [Invitación a usuarios B2B a Azure Active Directory](../b2b/add-users-administrator.md).

## <a name="add-a-consumer-user"></a>Incorporación de un usuario consumidor

Puede haber escenarios en los que desee crear manualmente cuentas de consumidor en el directorio de Azure Active Directory B2C (Azure AD B2C). Para más información sobre cómo crear cuentas de consumidor, consulte [Creación y eliminación de usuarios consumidores en Azure AD B2C](../../active-directory-b2c/manage-users-portal.md).

## <a name="add-a-new-user-within-a-hybrid-environment"></a>Agregar un nuevo usuario en un entorno híbrido

Si tiene un entorno con Azure Active Directory (nube) y Windows Server Active Directory (local), puede agregar nuevos usuarios mediante la sincronización de los datos de la cuenta de usuario existentes. Para obtener más información sobre los entornos híbridos, consulte [Integración de los directorios locales con Azure Active Directory](../hybrid/whatis-hybrid-identity.md).

## <a name="delete-a-user"></a>Eliminar un usuario

Puede eliminar un usuario existente mediante el portal de Azure Active Directory.

Siga estos pasos para eliminar un usuario:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con una cuenta de administrador de usuarios para la organización.

1. Busque y seleccione *Azure Active Directory* en cualquier página.

1. Busque y seleccione el usuario que quiere eliminar del inquilino de Azure AD. Por ejemplo, _Mary Parker_.

1. Seleccione **Eliminar usuario**.

    ![Página Users - All users (Usuarios: Todos los usuarios) con Eliminar usuario resaltado](media/add-users-azure-active-directory/delete-user-all-users-blade.png)

El usuario se elimina y ya no aparece en la página **Users - All users** (Usuarios: Todos los usuarios). El usuario se puede ver en la página **Usuarios eliminados** durante los próximos 30 días y puede restaurarse durante ese tiempo. Para más información sobre cómo restaurar un usuario, consulte [Restauración o eliminación de un usuario recientemente eliminado mediante Azure Active Directory](active-directory-users-restore.md).

Cuando se elimina un usuario, las licencias consumidas por el usuario se ponen a disposición de otros usuarios.

>[!Note]
>Debe usar Windows Server Active Directory para actualizar la identidad, la información de contacto o la información del trabajo para los usuarios cuyo origen de autoridad es Windows Server Active Directory. Después de completar la actualización, debe esperar a que se complete el próximo ciclo de sincronización antes de poder ver los cambios.

## <a name="next-steps"></a>Pasos siguientes

Después de agregar a los usuarios, puede realizar los procesos básicos siguientes:

- [Add or change profile information](active-directory-users-profile-azure-portal.md) (Incorporación o modificación de la información del perfil)

- [Asignación de roles a usuarios](active-directory-users-assign-role-azure-portal.md)

- [Creación de un grupo básico e incorporación de miembros](active-directory-groups-create-azure-portal.md)

- [Trabajo con usuarios y grupos dinámicos](../users-groups-roles/groups-create-rule.md)

O bien puede realizar otras tareas de administración de usuarios, como [agregar usuarios invitados de otro directorio](../b2b/what-is-b2b.md) o [restaurar un usuario eliminado](active-directory-users-restore.md). Para obtener más información acerca de otras acciones disponibles, consulte la [documentación de administración de usuarios en Azure Active Directory](../users-groups-roles/index.yml).
