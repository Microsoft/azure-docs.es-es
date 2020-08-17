---
title: Restablecimiento de la contraseña de un usuario en Azure Active Directory | Microsoft Docs
description: Instrucciones acerca de cómo restablecer una contraseña de usuario con Azure Active Directory.
services: active-directory
author: ajburnle
manager: daveba
ms.assetid: fad5624b-2f13-4abc-b3d4-b347903a8f16
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
ms.date: 09/05/2018
ms.author: ajburnle
ms.reviewer: jeffsta
ms.custom: it-pro, seodec18
ms.collection: M365-identity-device-management
ms.openlocfilehash: 69da6aa0253f92f9242a988c6b46de873df0677e
ms.sourcegitcommit: fbb66a827e67440b9d05049decfb434257e56d2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/05/2020
ms.locfileid: "87797210"
---
# <a name="reset-a-users-password-using-azure-active-directory"></a>Restablecimiento de la contraseña de un usuario con Azure Active Directory

Como administrador, puede restablecer la contraseña de un usuario si se olvida la contraseña, si el usuario bloquea su dispositivo o si nunca ha recibido una contraseña.

>[!Note]
>Si el inquilino de Azure AD no es el directorio principal de un usuario, no podrá restablecer su contraseña. Esto significa que si el usuario inicia sesión en su organización mediante una cuenta de otra organización, una cuenta de Microsoft o una cuenta de Google, no podrá restablecer su contraseña.<br><br>Si el usuario tiene una fuente de autoridad como Windows Server Active Directory, solo podrá restablecer la contraseña si ha activado la escritura diferida de contraseñas.<br><br>Si el usuario tiene una fuente de autoridad como Azure AD externo, no podrá restablecer la contraseña. Solo el usuario, o un administrador en Azure AD externo, puede restablecer la contraseña.

>[!Note]
>Si no es administrador y busca instrucciones acerca de cómo restablecer la contraseña profesional o educativa, consulte [Restablecimiento de la contraseña profesional o educativa](../user-help/active-directory-passwords-update-your-own-password.md).

## <a name="to-reset-a-password"></a>Para restablecer una contraseña

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) como administrador de usuarios o administrador de contraseñas. Para más información acerca de los roles disponibles, consulte [Asignación de roles de administrador en Azure Active Directory](../users-groups-roles/directory-assign-admin-roles.md#available-roles)

2. Seleccione **Azure Active Directory**, seleccione **Usuarios** y, a continuación, busque y seleccione los usuarios que necesitan del restablecimiento y haga clic en **Restablecer contraseña**.

    Se muestra la página **Alain Charon - Perfil** con la opción **Restablecer contraseña**.

    ![Página de perfil del usuario, con la opción Restablecer contraseña resaltada](media/active-directory-users-reset-password-azure-portal/user-profile-reset-password-link.png)

3. En la página **Restablecer contraseña**, seleccione **Restablecer contraseña**.

    > [!Note]
    > Al usar Azure Active Directory, una contraseña temporal se genera automáticamente para el usuario. En cambio cuando se usa Active Directory local, se crea la contraseña del usuario.

4. Copie la contraseña y proporciónela al usuario. El usuario deberá cambiar la contraseña durante el siguiente proceso de inicio de sesión.

    >[!Note]
    >La contraseña temporal nunca expira. La próxima vez que el usuario inicie sesión, la contraseña seguirá funcionando, sin importar cuánto tiempo haya transcurrido desde que se generó la contraseña temporal.

## <a name="next-steps"></a>Pasos siguientes

Después de restablecer la contraseña del usuario, puede realizar los siguientes procesos básicos:

- [Adición o eliminación de usuarios](add-users-azure-active-directory.md)

- [Asignación de roles a usuarios](active-directory-users-assign-role-azure-portal.md)

- [Add or change profile information](active-directory-users-profile-azure-portal.md) (Incorporación o modificación de la información del perfil)

- [Creación de un grupo básico e incorporación de miembros](active-directory-groups-create-azure-portal.md)

O bien, puede realizar otros escenarios de usuario complejos, como asignar delegados, usar directivas y compartir cuentas de usuario. Para obtener más información acerca de otras acciones disponibles, consulte la [documentación de administración de usuarios en Azure Active Directory](../users-groups-roles/index.yml).
