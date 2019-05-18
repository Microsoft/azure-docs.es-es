---
title: Incorporación de usuarios de colaboración B2B en Azure Portal | Microsoft Docs
description: Muestra cómo un administrador puede agregar usuarios invitados a su directorio desde una organización asociada con la colaboración B2B de Azure Active Directory (Azure AD).
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
ms.openlocfilehash: 66b3e68ff2199c6a8bf4da9e02caaf93ee69342b
ms.sourcegitcommit: 36c50860e75d86f0d0e2be9e3213ffa9a06f4150
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/16/2019
ms.locfileid: "65812824"
---
# <a name="add-azure-active-directory-b2b-collaboration-users-in-the-azure-portal"></a>Incorporación de usuarios de colaboración B2B de Azure Active Directory en Azure Portal

Como un usuario que se le asignan cualquiera de los roles de directorio administrador limitado, puede usar el portal de Azure para invitar a usuarios de colaboración B2B. Puede invitar a usuarios invitados al directorio, a un grupo o a una aplicación. Después de invitar a un usuario mediante cualquiera de estos métodos, la cuenta del usuario invitado se agrega a Azure Active Directory (Azure AD), con *invitado* como tipo de usuario. El usuario invitado deberá canjear después su invitación para acceder a los recursos.

Después de agregar un usuario invitado al directorio, puede enviarle un vínculo directo a una aplicación compartida o bien, el propio usuario invitado puede hacer clic en la dirección URL de canje del correo electrónico de invitación. Para más información sobre el proceso de canje, consulte [Canje de invitación de colaboración B2B](redemption-experience.md).

> [!IMPORTANT]
> Debe seguir los pasos descritos en [Procedimiento: Agregar información de privacidad con Azure Active Directory](https://aka.ms/adprivacystatement) para agregar la dirección URL de la declaración de privacidad de su organización. Como parte del proceso de canje de invitación por primera vez, el usuario invitado debe indicar su consentimiento con los términos de privacidad para poder continuar. 

## <a name="before-you-begin"></a>Antes de empezar

Asegúrese de que la configuración de colaboración externa de su organización se configura de forma que le permite invitar a otros usuarios. De forma predeterminada, todos los usuarios y administradores pueden invitar a otros usuarios. Pero las directivas de colaboración externa de su organización se pueden configurar para evitar que determinados tipos de los usuarios o administradores inviten a otros usuarios. Para obtener información sobre cómo ver y establecer estas directivas, consulte [permiten la colaboración de B2B externo y administrar quién puede invitar a otros usuarios](delegate-invitations.md).

## <a name="add-guest-users-to-the-directory"></a>Adición de usuarios invitados al directorio

Para agregar usuarios de colaboración B2B al directorio, siga estos pasos:

1. Inicie sesión en el [portal Azure](https://portal.azure.com) como un usuario que se haya asignado un rol de directorio administrador limitado o el rol de Invitador invitado.
2. En el panel de navegación, seleccione **Azure Active Directory**.
3. En **Administrar**, seleccione **Usuarios**.
4. Seleccione **Nuevo usuario invitado**.

   ![Muestra dónde se encuentra el nuevo usuario invitado en la interfaz de usuario.](./media/add-users-administrator/NewGuestUser-Directory.png) 
 
   > [!NOTE]
   > La opción **Nuevo usuario invitado** también está disponible en la página **Relaciones de organización**. En **Azure Active Directory**, en **Administrar**, seleccione **Relaciones de organización**.

5. En **Nombre de usuario**, escriba la dirección de correo electrónico del usuario externo. Opcionalmente, puede incluir un mensaje de bienvenida. Por ejemplo:

   ![Muestra dónde se encuentra el nuevo usuario invitado en la interfaz de usuario.](./media/add-users-administrator/InviteGuest.png) 

    > [!NOTE]
    > No se admiten direcciones de correo electrónico de grupos; escriba la dirección de un usuario individual. Además, algunos proveedores de correo electrónico permiten a los usuarios agregar un signo más (+) y texto adicional a sus direcciones de correo electrónico, ya que ello sirve de ayuda a algunas funciones como el filtrado de la bandeja de entrada. Sin embargo, Azure AD no admite actualmente estos símbolos más (+) en las direcciones de correo electrónico. Para evitar problemas de entrega, omita el signo más y los caracteres siguientes hasta el símbolo @.

6. Seleccione **Invitar** para enviar automáticamente la invitación al usuario invitado. 
 
Después de enviar la invitación, la cuenta de usuario se agrega automáticamente al directorio como invitado.


![Muestra el usuario de B2B con el tipo de usuario de invitado.](./media/add-users-administrator/GuestUserType.png)  

## <a name="add-guest-users-to-a-group"></a>Adición de usuarios invitados a un grupo
Si tiene que agregar manualmente usuarios de colaboración B2B a un grupo, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de Azure AD.
2. En el panel de navegación, seleccione **Azure Active Directory**.
3. En **Administrar**, seleccione **Grupos**.
4. Seleccione un grupo (o haga clic en **Nuevo grupo** para crear uno). Es una buena idea incluir en la descripción del grupo que el grupo contiene los usuarios invitados B2B.
5. Seleccione **Miembros**. 
6. Realice una de las operaciones siguientes:
   - Si el usuario invitado ya existe en el directorio, busque el usuario B2B. Seleccione el usuario y haga clic en **Seleccionar** para agregar el usuario al grupo.
   - Si el usuario invitado no existe ya en el directorio, invítelo al grupo; para ello, escriba su dirección de correo electrónico en el cuadro de búsqueda, escriba un mensaje personal opcional y, a continuación, haga clic en **Seleccionar**. La invitación se dirige automáticamente al usuario invitado.
     
     ![Adición del botón Invitar para agregar miembros invitados](./media/add-users-administrator/GroupInvite.png)
   
También puede utilizar grupos dinámicos con colaboración B2B de Azure AD. Para más información, consulte [Grupos dinámicos y colaboración B2B de Azure Active Directory](use-dynamic-groups.md).

## <a name="add-guest-users-to-an-application"></a>Adición de usuarios invitados a una aplicación

Para agregar usuarios de colaboración B2B a una aplicación, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de Azure AD.
2. En el panel de navegación, seleccione **Azure Active Directory**.
3. En **Administrar**, seleccione **Aplicaciones empresariales** > **Todas las aplicaciones**.
4. Seleccione la aplicación a la que desea agregar usuarios invitados.
5. En el panel de la aplicación, seleccione **Usuarios totales** para abrir el panel **Usuarios y grupos**.

    ![Botón Usuarios totales para abrir Usuarios y grupos](./media/add-users-administrator/AppUsersAndGroups.png)

6. Seleccione **Agregar usuario**.
7. En **Agregar asignación**, seleccione **Usuarios y grupos**.
8. Realice una de las operaciones siguientes:
   - Si el usuario invitado ya existe en el directorio, busque el usuario B2B. Seleccione el usuario, haga clic en **Seleccionar** y luego haga clic en **Asignar** para agregar el usuario a la aplicación.
   - Si el usuario invitado no existe en el directorio, en **Seleccione un miembro o invite a un usuario externo**, escriba la dirección de correo electrónico del usuario. En el cuadro de mensaje, escriba un mensaje personal opcional. Debajo del cuadro de mensaje, haga clic en **Invitar**.
           
       ![Adición del botón Invitar para agregar miembros invitados](./media/add-users-administrator/AppInviteUsers.png)
   
      Haga clic en **Seleccionar** y luego haga clic en **Asignar** para agregar al usuario a la aplicación. Una invitación se dirige automáticamente al usuario invitado.

9. El usuario invitado aparece en la lista **Usuarios y grupos** de la aplicación con el rol asignado de **Acceso predeterminado**. Si quiere cambiar el rol, haga lo siguiente:
   - Seleccione el usuario invitado y, a continuación, seleccione **Editar**. 
   - En **Editar asignación**, haga clic en **Seleccionar rol** y seleccione el rol que quiere asignar al usuario seleccionado.
   - Haga clic en **Seleccionar**.
   - Haga clic en **Asignar**.
 
## <a name="resend-invitations-to-guest-users"></a>Reenvío de invitaciones a usuarios invitados

Si un usuario invitado todavía no ha canjeado su invitación, puede volver a enviarle el correo electrónico de invitación.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador de Azure AD.
2. En el panel de navegación, seleccione **Azure Active Directory**.
3. En **Administrar**, seleccione **Usuarios**.
5. Seleccione la cuenta del usuario.
6. En **Administrar**, seleccione **Perfil**.
7. Si el usuario todavía no ha aceptado la invitación, la opción **Volver a enviar la invitación** está disponible. Seleccione este botón para volver a enviarla.

   ![Opción Volver a enviar la invitación en el perfil de usuario](./media/add-users-administrator/Resend-Invitation.png)

> [!NOTE]
> Si vuelve a enviar una invitación que originalmente dirigía al usuario a una aplicación específica, debe entender que el vínculo de la nueva invitación lleva por el contrario al usuario al panel de acceso de nivel superior.

## <a name="next-steps"></a>Pasos siguientes

- Para aprender cómo los administradores que no son de Azure AD pueden agregar usuarios invitados B2B, consulte [¿Cómo agregan los trabajadores de la información usuarios de colaboración B2B a Azure Active Directory?](add-users-information-worker.md)
- Para más información sobre el correo electrónico de invitación, consulte [Los elementos del correo electrónico de invitación de colaboración B2B](invitation-email-elements.md).

