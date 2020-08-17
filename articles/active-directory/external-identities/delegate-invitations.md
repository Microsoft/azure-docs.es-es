---
title: 'Habilitación de los valores de colaboración externa B2B: Azure AD'
description: Obtenga información sobre cómo habilitar la colaboración externa B2B de Active Directory y administrar quién puede invitar a usuarios invitados. Uso del rol de invitador de usuarios invitados para delegar invitaciones.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 05/11/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: c6a2c1a9b908503ee5afc2687ebef473ffed626a
ms.sourcegitcommit: 4e5560887b8f10539d7564eedaff4316adb27e2c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87907728"
---
# <a name="enable-b2b-external-collaboration-and-manage-who-can-invite-guests"></a>Habilitación de la colaboración externa B2B y administración de quién puede invitar a otros usuarios

En este artículo se describe cómo habilitar la colaboración B2B de Azure Active Directory (Azure AD) y determinar quién puede invitar a otros usuarios. De manera predeterminada, todos los usuarios e invitados del directorio pueden invitar a otros usuarios, incluso si no tienen asignado un rol de administrador. La configuración de colaboración externa le permite activar o desactivar las invitaciones de invitados para los distintos tipos de usuarios de la organización. También puede delegar las invitaciones a usuarios individuales mediante la asignación de roles que les permitan invitar a otros usuarios.

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
6. En la página **Configuración de colaboración externa**, elija las directivas que quiere habilitar.

   ![Configuración de colaboración externa](./media/delegate-invitations/control-who-to-invite.png)

  - **Los permisos de los usuarios invitados están limitados**: Esta directiva determina los permisos para los invitados de su directorio. Seleccione **Sí** para impedir que los invitados realicen determinadas tareas de directorio, como la enumeración de usuarios, grupos u otros recursos de directorio. Seleccione **No** para proporcionar a los invitados el mismo acceso a datos del directorio que el de los usuarios normales del directorio.
   - **Los administradores y los usuarios del rol de invitador de usuarios invitados pueden invitar**: Para permitir que los administradores y usuarios del rol "Invitador de usuarios invitados" inviten a otros usuarios, establezca esta directiva en **Sí**.
   - **Los miembros pueden invitar**: Para permitir que los miembros que no son administradores del directorio inviten a otros usuarios, establezca esta directiva en **Sí**.
   - **Los invitados pueden invitar**: Para permitir que los invitados puedan invitar a otros usuarios, establezca esta directiva en **Sí**.
   - **Habilitar el código de acceso de un solo uso de correo electrónico para invitados (versión preliminar)** : Para obtener más información sobre la característica de código de acceso de un solo uso, consulte [Email one-time passcode authentication (preview)](one-time-passcode.md) [Autenticación con código de acceso de un solo uso de correo electrónico (versión preliminar)].
   - **Restricciones de colaboración**: Para obtener más información sobre cómo permitir o bloquear invitaciones en dominios concretos, consulte [Allow or block invitations to B2B users from specific organizations](allow-deny-list.md) (Permitir o bloquear invitaciones a usuarios B2B procedentes de determinadas organizaciones).
   
   > [!NOTE]
   > Si **Los miembros pueden invitar** está establecido en **No** y **Los administradores y los usuarios del rol de invitador de personas pueden invitar** está establecido en **Sí**, los usuarios del rol **Invitador de usuarios invitados** todavía podrán invitar a usuarios invitados.

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

