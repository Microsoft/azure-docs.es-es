---
title: 'Habilitación de los valores de colaboración externa B2B: Azure AD'
description: Obtenga información sobre cómo habilitar la colaboración externa B2B de Active Directory y administrar quién puede invitar a usuarios invitados. Uso del rol de invitador de usuarios invitados para delegar invitaciones.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 08/20/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: ae8bb66141e4cc4e67f1502b208cf519d37c0374
ms.sourcegitcommit: e0785ea4f2926f944ff4d65a96cee05b6dcdb792
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/21/2020
ms.locfileid: "88706014"
---
# <a name="enable-b2b-external-collaboration-and-manage-who-can-invite-guests"></a>Habilitación de la colaboración externa B2B y administración de quién puede invitar a otros usuarios

En este artículo se describe cómo habilitar la colaboración B2B de Azure Active Directory (Azure AD), designar quién puede tener invitados y determinar los permisos que tienen los usuarios invitados en Azure AD. 

De manera predeterminada, todos los usuarios e invitados del directorio pueden invitar a otros usuarios, incluso si no tienen asignado un rol de administrador. La configuración de colaboración externa le permite activar o desactivar las invitaciones de invitados para los distintos tipos de usuarios de la organización. También puede delegar las invitaciones a usuarios individuales mediante la asignación de roles que les permitan invitar a otros usuarios.

Azure AD le permite restringir qué usuarios invitados externos se pueden ver en el directorio de Azure AD. De forma predeterminada, los usuarios invitados tienen un nivel de permiso limitado que les impide enumerar usuarios, grupos u otros recursos de directorio, pero sí ver la pertenencia de grupos no ocultos. Una nueva opción en versión preliminar le permite restringir aún más el acceso de invitado, de modo que los invitados solo pueden ver su propia información de perfil. 

## <a name="configure-b2b-external-collaboration-settings"></a>Configuración de los valores de colaboración externa B2B

Gracias a la colaboración B2B de Azure AD, el administrador de inquilinos puede establecer las siguientes directivas de invitación:

- Desactivar invitaciones
- Solo los administradores y usuarios del rol Invitador de personas pueden invitar
- Los administradores, el rol Invitador de personas y los miembros pueden invitar
- Todos los usuarios, incluidos los invitados, pueden invitar

De manera predeterminada, todos los usuarios, incluidos los invitados, pueden invitar a usuarios.

### <a name="to-configure-external-collaboration-settings"></a>Para configurar los valores de colaboración externa B2B:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de inquilinos.
2. Seleccione **Azure Active Directory**.
3. Seleccione **External Identities** > **Configuración de colaboración externa**.

4. En **Restricciones de acceso de usuarios invitados (versión preliminar)** , elija el nivel de acceso que desea que tengan los usuarios invitados:

   > [!IMPORTANT]
   > Durante un corto tiempo, estos nuevos controles del portal de los permisos de usuario invitado serán visibles solo con la dirección URL [https://aka.ms/AADRestrictedGuestAccess](https://aka.ms/AADRestrictedGuestAccess). Para más información, consulte [Restricción de los permisos de acceso de usuario (versión preliminar)](https://aka.ms/exid-users-restrict-guest-permissions).

   - **Guest users have the same access as members (most inclusive)** (Los usuarios invitados tienen el mismo acceso que los miembros [principalmente inclusivo]): esta opción permite a los invitados tener el mismo acceso a los recursos de Azure AD y a los datos del directorio que los usuarios miembros.

   - **Guest users have limited access to properties and memberships of directory objects** (Los usuarios invitados tienen acceso limitado a las propiedades y pertenencias de los objetos de directorio): (predeterminada) Esta opción impide que los usuarios realicen determinadas tareas de directorio, como enumerar usuarios, grupos u otros recursos de directorio. Los invitados pueden ver la pertenencia de todos los grupos no ocultos.

   - **Guest user access is restricted to properties and memberships of their own directory objects (most restrictive)** (El acceso de los usuarios invitados está restringido a las propiedades y pertenencias de sus propios objetos de directorio [opción más restrictiva]): con esta configuración, los invitados solo pueden tener acceso a sus propios perfiles. No se permite a los invitados ver perfiles, grupos o pertenencias a grupos de otros usuarios.
  
    ![Configuración de restricciones de acceso de usuarios invitados](./media/delegate-invitations/guest-user-access.png)

5. En **Configuración de la invitación de usuarios**, elija la configuración adecuada:

   - **Los administradores y los usuarios del rol de invitador de usuarios invitados pueden invitar**: Para permitir que los administradores y usuarios del rol "Invitador de usuarios invitados" inviten a otros usuarios, establezca esta directiva en **Sí**.

   - **Los miembros pueden invitar**: Para permitir que los miembros que no son administradores del directorio inviten a otros usuarios, establezca esta directiva en **Sí**.

   - **Los invitados pueden invitar**: Para permitir que los invitados puedan invitar a otros usuarios, establezca esta directiva en **Sí**.

   - **Habilitar el código de acceso de un solo uso de correo electrónico para invitados (versión preliminar)** : para más información sobre la característica de código de acceso de un solo uso, consulte [Autenticación con código de acceso de un solo uso de correo electrónico (versión preliminar)](one-time-passcode.md).

   - **Enable guest self-service sign up via user flows (Preview)** (Habilitación del autoservicio de registro de invitados mediante flujos de usuario [versión preliminar]): para más información sobre esta opción, consulte [Incorporación de un flujo de usuario de registro de autoservicio a una aplicación (versión preliminar)](self-service-sign-up-user-flow.md).

   > [!NOTE]
   > Si **Los miembros pueden invitar** está establecido en **No** y **Los administradores y los usuarios del rol de invitador de personas pueden invitar** está establecido en **Sí**, los usuarios del rol **Invitador de usuarios invitados** todavía podrán invitar a usuarios invitados.

    ![Configuración de la invitación de usuarios](./media/delegate-invitations/guest-invite-settings.png)

6. En **Restricciones de colaboración**, elija si desea permitir o denegar las invitaciones a los dominios que especifique. Para obtener más información, consulte [Allow or block invitations to B2B users from specific organizations](allow-deny-list.md) (Permitir o bloquear invitaciones a usuarios de B2B procedentes de determinadas organizaciones).

## <a name="assign-the-guest-inviter-role-to-a-user"></a>Asignación del rol de invitador de usuarios invitados a un usuario

Con el rol de invitador de usuarios invitados, puede conceder a usuarios individuales la capacidad de invitar a otros usuarios sin asignarles un rol de administrador global u otro rol de administrador. Asigne el rol de invitador de usuarios invitados a diferentes individuos. A continuación, asegúrese de establecer la opción **Los administradores y los usuarios del rol de invitador de usuarios invitados pueden invitar** en **Sí**.

Este es un ejemplo que muestra cómo usar PowerShell para agregar un usuario al rol Invitador de personas:

```
Add-MsolRoleMember -RoleObjectId 95e79109-95c0-4d8e-aee3-d01accf2d47b -RoleMemberEmailAddress <RoleMemberEmailAddress>
```

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos sobre la colaboración de B2B de Azure AD:

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
- [Incorporación de usuarios de colaboración B2B sin invitación](add-user-without-invite.md)
- [Incorporación de usuarios de colaboración B2B a un rol](add-guest-to-role.md)

