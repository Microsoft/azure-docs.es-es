---
title: Asignación de roles de recursos de Azure a invitados en PIM-Azure AD | Microsoft Docs
description: Aprenda a invitar a usuarios invitados externos y a asignar roles de recursos de Azure en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.subservice: pim
ms.date: 11/08/2019
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: 763441d023e4a707c6a1edc09abfb6d8e5525723
ms.sourcegitcommit: 9c3cfbe2bee467d0e6966c2bfdeddbe039cad029
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/24/2020
ms.locfileid: "88782743"
---
# <a name="invite-guest-users-and-assign-azure-resource-roles-in-privileged-identity-management"></a>Invite a usuarios invitados y asigne roles de recursos de Azure en Privileged Identity Management

Los usuarios invitados de Azure Active Directory (Azure AD) forman parte de las funcionalidades de colaboración negocio a negocio (B2B) dentro de Azure AD para que pueda administrar los usuarios y proveedores de invitados externos como invitados en Azure AD. Al combinar la colaboración B2B con Azure AD Privileged Identity Management (PIM), puede ampliar sus requisitos de cumplimiento y gobernanza a los invitados. Por ejemplo, puede usar estas características Privileged Identity Management para las tareas de identidad de Azure con invitados:

- Asignar acceso a recursos específicos de Azure
- Habilitar acceso Just-In-Time
- Especificar la duración de la asignación y la fecha de finalización
- Requerir autenticación multifactor en la asignación activa o activación
- Realizar revisiones de acceso
- Usar alertas y registros de auditoría

En este artículo se describe cómo invitar a un invitado a su organización y administrar su acceso a recursos de Azure con Privileged Identity Management.

## <a name="when-would-you-invite-guests"></a>¿Cuándo invitaría a invitados?

Estos son algunos ejemplos de cuándo se pueden invitar a los invitados a su organización:

- Permitir que un proveedor externo por cuenta propia que solo tiene una cuenta de correo electrónico acceda a los recursos de Azure para la realización de un proyecto
- Permitir que a un asociado externo de una gran organización que usa Servicios de federación de Active Directory (AD FS) locales acceda a la aplicación de gastos
- Permitir que los ingenieros del servicio de soporte técnico que no están en su organización (por ejemplo, el soporte técnico de Microsoft) accedan temporalmente a los recursos de Azure para solucionar problemas

## <a name="how-does-collaboration-using-b2b-guests-work"></a>¿Cómo funciona la colaboración mediante invitados B2B?

Al usar la colaboración B2B, puede invitar a un usuario externo a su organización como invitado. El invitado puede administrarse como un usuario de la organización, pero un invitado debe estar autenticado en su organización principal y no en la organización de Azure AD. Esto significa que, si el invitado ya no tiene acceso a su organización principal, también perderá el acceso a su organización. Por ejemplo, si el invitado abandona la organización, automáticamente pierde acceso a los recursos compartidos con él en Azure AD sin necesidad de hacer nada. Para más información sobre la colaboración B2B, consulte [¿Qué es el acceso de usuarios invitados en Azure Active Directory B2B?](../external-identities/what-is-b2b.md).

![Diagrama que muestra cómo se autentica un usuario invitado en su directorio particular](./media/pim-resource-roles-external-users/b2b-external-user.png)

## <a name="check-guest-collaboration-settings"></a>Comprobación de la configuración de colaboración de invitado

Para asegurarse de que puede invitar a invitados a su organización, debe comprobar la configuración de colaboración de invitado.

1. Inicie sesión en el [portal de Azure](https://portal.azure.com/).

1. Seleccione **Azure Active Directory** > **Configuración de usuario**.

1. Seleccione **Administrar la configuración de colaboración externa**.

    ![Página de configuración de colaboración externa en la que se muestra la configuración de restricción de la colaboración, la invitación y los permisos](./media/pim-resource-roles-external-users/external-collaboration-settings.png)

1. Asegúrese de que el conmutador **Los administradores y los usuarios del rol de invitador de personas pueden invitar** se establece en **Sí**.

## <a name="invite-a-guest-and-assign-a-role"></a>Invitar a un invitado y asignar un rol

Con Privileged Identity Management, puede invitar a un invitado y hacer que sea válido para un rol de recurso de Azure.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un usuario que sea miembro del rol [Administrador de roles con privilegios](../users-groups-roles/directory-assign-admin-roles.md#privileged-role-administrator) o [Administrador de usuarios](../users-groups-roles/directory-assign-admin-roles.md#user-administrator).

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Azure resources** (Recursos de Azure).

1. Use **Filtro de recursos** para filtrar la lista de recursos administrados.

1. Seleccione el recurso que quiere administrar, como un recurso, un grupo de recursos, una suscripción o un grupo de administración.

    Debe establecer el ámbito a solo lo que necesita el invitado.

1. En Administrar, seleccione **Roles** para ver la lista de roles de los recursos de Azure.

    ![Lista de roles de recursos de Azure en la que se muestra el número de usuarios que están activos y se pueden elegir](./media/pim-resource-roles-external-users/resources-roles.png)

1. Seleccione el rol mínimo que necesitará el usuario.

    ![Página del rol seleccionado en la que se muestran los miembros actuales de ese rol](./media/pim-resource-roles-external-users/selected-role.png)

1. En la página rol, seleccione **Agregar miembro** para abrir el Panel nueva asignación.

1. Haga clic en **Seleccionar un miembro o grupo**.

    ![Panel Nueva asignación: Seleccionar un miembro o grupo en el que se muestran los usuarios y grupos, con una opción de invitar](./media/pim-resource-roles-external-users/select-member-group.png)

1. Para invitar a un invitado, haga clic en **Invitar**.

    ![Página Invitar a un invitado con cuadros para escribir una dirección de correo electrónico y especificar un mensaje personal](./media/pim-resource-roles-external-users/invite-guest.png)

1. Después de haber seleccionado un invitado, haga clic en **Invitar**.

    Se debería agregar al invitado como un miembro seleccionado.

1. En el panel **Seleccionar un miembro o grupo**, haga clic en **Seleccionar**.

1. En el panel **Configuración de pertenencia**, seleccione el tipo de asignación y la duración.

    ![Página Nueva asignación: Configuración de pertenencia con opciones para especificar el tipo de asignación, la fecha de inicio y la fecha de finalización](./media/pim-resource-roles-external-users/membership-settings.png)

1. Para completar la asignación, seleccione **Listo** y, después, **Agregar**.

    La asignación de roles de invitado aparecerá en la lista de roles.

    ![Página de roles en la que aparece el invitado como apto](./media/pim-resource-roles-external-users/role-assignment.png)

## <a name="activate-role-as-a-guest"></a>Activación del rol como invitado

Si es un usuario externo, debe aceptar la invitación como invitado en la organización de Azure AD y, posiblemente, activar la asignación de roles.

1. Abra el correo electrónico con la invitación. El correo electrónico se parecerá al siguiente:

    ![Invitación por correo electrónico con el nombre del directorio, el mensaje personal y un vínculo Comenzar](./media/pim-resource-roles-external-users/email-invite.png)

1. Seleccione el vínculo **Comenzar** en el correo.

1. Después de revisar los permisos, haga clic en **Aceptar**.

    ![Página de revisión de permisos en un explorador con una lista de permisos que la organización quiere que revisen los usuarios](./media/pim-resource-roles-external-users/invite-accept.png)

1. Puede que se le pida que acepte las condiciones de uso y que especifique si quiere mantener la sesión iniciada. En Azure Portal, si es *apto* para un rol, aún no tendrá acceso a los recursos.

1. Para activar la asignación de roles, abra el correo con el vínculo activar rol. El correo electrónico se parecerá al siguiente:

    ![Correo que indica que es apto para un rol con un Vínculo activar rol](./media/pim-resource-roles-external-users/email-role-assignment.png)

1. Seleccione **Activar rol** para abrir los roles aptos en Privileged Identity Management.

    ![Página mis roles de Privileged Identity Management enumera los roles válidos](./media/pim-resource-roles-external-users/my-roles-eligible.png)

1. En Acción, seleccione el vínculo **Activar**.

    Según la configuración de rol, deberá especificar la información para activar el rol.

1. Cuando haya especificado la configuración del rol, haga clic en **Activar** para activar el rol.

    ![Página Activar en la que aparece el ámbito y las opciones para especificar la hora de inicio, la duración y el motivo](./media/pim-resource-roles-external-users/activate-role.png)

    A menos que el administrador deba aprobar la solicitud, debería tener acceso a los recursos especificados.

## <a name="view-activity-for-a-guest"></a>Visualización de la actividad de un invitado

Puede ver los registros de auditoría para realizar un seguimiento de lo que hacen los invitados.

1. Como administrador, abra Privileged Identity Management y seleccione el recurso que se ha compartido.

1. Seleccione **Auditoría de recursos** para ver la actividad de ese recurso. A continuación se muestra un ejemplo de la actividad para un grupo de recursos.

    ![Recursos de Azure: página Auditoría de recursos en la que aparece la hora, el solicitante y la acción](./media/pim-resource-roles-external-users/audit-resource.png)

1. Para ver la actividad del invitado, seleccione **Azure Active Directory** > **Usuarios** > *nombre del invitado*.

1. Seleccione **Registros de auditoría** para ver los registros de auditoría de la organización. Si es necesario, puede especificar filtros.

    ![Registros de auditoría del directorio en los que se muestran la fecha, el destino, quien lo ha iniciado y la actividad](./media/pim-resource-roles-external-users/audit-directory.png)

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de roles de administrador de Azure AD en Privileged Identity Management](pim-how-to-add-role-to-user.md)
- [¿Qué es el acceso de usuarios invitados en Azure AD colaboración B2B?](../external-identities/what-is-b2b.md)