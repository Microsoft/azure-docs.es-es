---
title: Habilitar la configuración de colaboración externo de B2B - Azure Active Directory | Microsoft Docs
description: Obtenga información sobre cómo habilitar la colaboración de B2B de Active Directory externo y administrar quién puede invitar a usuarios invitados. Usar el rol de Invitador invitado para delegar las invitaciones.
services: active-directory
ms.service: active-directory
ms.subservice: B2B
ms.topic: conceptual
ms.date: 04/11/2019
ms.author: mimart
author: msmimart
manager: celestedg
ms.reviewer: mal
ms.collection: M365-identity-device-management
ms.openlocfilehash: 11dda7fc3760f468c094fb4cf4484a27895f83b9
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65812670"
---
# <a name="enable-b2b-external-collaboration-and-manage-who-can-invite-guests"></a>Permitir la colaboración de B2B externo y administrar quién puede invitar a otros usuarios

En este artículo se describe cómo habilitar la colaboración B2B de Azure Active Directory (Azure AD) y determinar quién puede invitar a otros usuarios. De forma predeterminada, todos los usuarios y los invitados del directorio pueden invitar a otros usuarios incluso si no le asigna un rol de administrador. Configuración de colaboración externa le permite convertir las invitaciones de invitados activado o desactivado para los distintos tipos de usuarios de su organización. También puede delegar las invitaciones a usuarios individuales mediante la asignación de roles que permitirle invitar a otros usuarios.

## <a name="configure-b2b-external-collaboration-settings"></a>Configurar opciones de colaboración externo de B2B

Gracias a la colaboración B2B de Azure AD, el administrador de inquilinos puede establecer las siguientes directivas de invitación:

- Desactivar invitaciones
- Solo los administradores y usuarios del rol Invitador de personas pueden invitar
- Los administradores, el rol Invitador de personas y los miembros pueden invitar
- Todos los usuarios, incluidos los invitados, pueden invitar

De forma predeterminada, todos los usuarios, incluidos a los invitados, pueden invitar a usuarios invitados.

### <a name="to-configure-external-collaboration-settings"></a>Para configurar las opciones de colaboración externa:

1. Inicie sesión en el [portal Azure](https://portal.azure.com) como administrador de inquilinos.
2. Seleccione **Azure Active Directory** > **Usuarios** > **Configuración de usuario**.
3. En **Usuarios externos**, seleccione **Administrar la configuración de colaboración externa**.
   > [!NOTE]
   > La opción **Configuración de colaboración externa** también está disponible en la página **Relaciones de organización**. En Azure Active Directory, en **Administrar**, vaya a **Relaciones de organización** > **Configuración**.
4. En el **configuración de colaboración externa** página, elija las directivas que desea habilitar.

   ![Configuración de colaboración externa](./media/delegate-invitations/control-who-to-invite.png)

  - **Permisos de los usuarios invitados están limitados**: Esta directiva determina los permisos para los invitados en su directorio. Seleccione **Sí** a los invitados de bloque de determinadas tareas de directorio, como enumerar los usuarios, grupos u otros recursos de directorio. Seleccione **No** para proporcionar a los invitados el mismo acceso a datos de directorio como usuarios normales en el directorio.
   - **Los administradores y los usuarios del rol de invitador invitado pueden invitar**: Para permitir que los administradores y usuarios en el rol "Invitador" invitar a otros usuarios, establecer esta directiva en **Sí**.
   - **Los miembros pueden invitar**: Para permitir que los miembros que no es administrador del directorio de invitar a otros usuarios, establecer esta directiva en **Sí**.
   - **Los invitados pueden invitar**: Para permitir que los invitados puedan invitar a otras personas, establecer esta directiva en **Sí**.
   - **Habilitar el código de acceso de correo electrónico de un solo uso para los invitados (versión preliminar)**: Para obtener más información acerca de la característica de código de acceso de un solo uso, consulte [autenticación de correo electrónico única del código de acceso (versión preliminar)](one-time-passcode.md).
   - **Restricciones de colaboración**: Para obtener más información sobre cómo permitir o bloquear invitaciones a dominios concretos, vea [permitir o bloquear invitaciones a usuarios B2B procedentes de determinadas organizaciones](allow-deny-list.md).

## <a name="assign-the-guest-inviter-role-to-a-user"></a>Asignar el rol de Invitador invitado a un usuario

Con el rol de Invitador invitado, puede dar la capacidad de invitar a otros usuarios sin asignarles un administrador global u otro rol de administrador de usuarios individuales. Asigne el rol de invitador invitado a usuarios individuales. Asegúrese de establecer **los administradores y los usuarios del rol de invitador invitado pueden invitar** a **Sí**.

Este es un ejemplo que muestra cómo usar PowerShell para agregar un usuario al rol Invitador de personas:

```
Add-MsolRoleMember -RoleObjectId 95e79109-95c0-4d8e-aee3-d01accf2d47b -RoleMemberEmailAddress <RoleMemberEmailAddress>
```

## <a name="next-steps"></a>Pasos siguientes

Consulte los siguientes artículos sobre la colaboración de B2B de Azure AD:

- [¿Qué es la colaboración B2B de Azure AD?](what-is-b2b.md)
- [Incorporación de usuarios de colaboración B2B sin invitación](add-user-without-invite.md)
- [Incorporación de usuarios de colaboración B2B a un rol](add-guest-to-role.md)


