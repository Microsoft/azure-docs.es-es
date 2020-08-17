---
title: 'Configuración de roles de Azure AD en PIM: Azure AD | Microsoft Docs'
description: Aprenda a configurar roles de Azure AD en Azure AD Privileged Identity Management (PIM).
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
editor: ''
ms.service: active-directory
ms.topic: how-to
ms.workload: identity
ms.subservice: pim
ms.date: 02/28/2020
ms.author: curtand
ms.custom: pim
ms.collection: M365-identity-device-management
ms.openlocfilehash: e3a42cdfbf251e24ed2b59adca1a73b4dfe427a7
ms.sourcegitcommit: e71da24cc108efc2c194007f976f74dd596ab013
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87420012"
---
# <a name="configure-azure-ad-role-settings-in-privileged-identity-management"></a>Configuración de roles de Azure AD en Privileged Identity Management

Un administrador de roles con privilegios puede personalizar Privileged Identity Management (PIM) en su organización de Azure Active Directory (Azure AD), incluido el cambio de la experiencia de un usuario que va a activar una asignación de rol válida.

## <a name="determine-your-version-of-pim"></a>Determinación de la versión de PIM

Desde noviembre de 2019, la parte de roles de Azure AD de Privileged Identity Management se está actualizando a una nueva versión que coincide con las experiencias de los roles de recursos de Azure. Esta actualización introduce características adicionales y [cambios en la API existente](azure-ad-roles-features.md#api-changes). Mientras se implementa la nueva versión, los procedimientos que seguirá en este artículo dependerán de la versión de Privileged Identity Management que tenga actualmente. Siga los pasos de esta sección para determinar la versión de Privileged Identity Management que tiene. Cuando averigüe la versión de Privileged Identity Management, puede seleccionar los procedimientos de este artículo que coincidan con esa versión.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un usuario que tenga el rol [Administrador de roles con privilegios](../users-groups-roles/directory-assign-admin-roles.md#privileged-role-administrator).
1. Abra **Azure AD Privileged Identity Management**. Si tiene un banner en la parte superior de la página de introducción, siga las instrucciones de la pestaña **Nueva versión** de este artículo. De lo contrario, siga las instrucciones de la pestaña **Versión anterior**.

  [![Seleccionar Azure AD > Privileged Identity Management.](media/pim-how-to-add-role-to-user/pim-new-version.png)](media/pim-how-to-add-role-to-user/pim-new-version.png#lightbox)

Siga los pasos que se describen en este artículo para aprobar o denegar solicitudes para los roles de Azure AD.

# <a name="new-version"></a>[Nueva versión](#tab/new)

## <a name="open-role-settings"></a>Apertura de la configuración de roles

Siga estos pasos para abrir la configuración de un rol de Azure AD.

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) con un usuario en el rol [Administrador de roles con privilegios](../users-groups-roles/directory-assign-admin-roles.md#privileged-role-administrator).
gt
1. Abra **Azure AD Privileged Identity Management** &gt; **Roles de Azure AD** &gt; **Configuración de roles**.

    ![Página de configuración de roles que muestra los roles de Azure AD](./media/pim-how-to-change-default-settings/role-settings.png)

1. Seleccione el rol cuya configuración desea configurar.

    ![Página de detalles de configuración de roles que muestra varias opciones de asignación y activación](./media/pim-how-to-change-default-settings/role-settings-page.png)

1. Seleccione **Editar** para abrir la página Configuración de roles.

    ![Página de edición de configuración de roles con opciones para actualizar la configuración de asignación y activación](./media/pim-how-to-change-default-settings/role-settings-edit.png)

    En el panel Configuración de roles de cada rol, hay varias opciones que puede configurar.

## <a name="assignment-duration"></a>Duración de la asignación

Puede elegir entre dos opciones de duración de asignación para cada tipo de asignación (Apto y Activo) cuando se configuran las opciones para un rol. Estas opciones se convierten en la duración máxima predeterminada cuando se asigna un usuario al rol en Privileged Identity Management.

Puede elegir uno de estas opciones de duración de asignación tipo **Apto**:

| | |
| --- | --- |
| **Permitir asignación elegible permanente** | Los administradores globales y los administradores de roles con privilegios pueden asignar una asignación elegible permanente. |
| **Hacer que las asignaciones elegibles expiren después de** | Los administradores globales y los administradores de roles con privilegios pueden requerir que todas las asignaciones elegibles tengan una fecha de inicio y finalización especificada. |

Además, puede elegir una de estas opciones de duración de asignación tipo **Activo**:

| | |
| --- | --- |
| **Permitir asignaciones activas permanentes** | Los administradores globales y los administradores de roles con privilegios pueden asignar una asignación activa permanente. |
| **Hacer que las asignaciones activas expiren después de** | Los administradores globales y los administradores de roles con privilegios pueden requerir que todas las asignaciones activas tengan una fecha de inicio y finalización especificada. |

> [!NOTE]
> Los administradores globales y los administradores de roles con privilegios pueden renovar todas las asignaciones que tienen una fecha de finalización específica. Además, los usuarios pueden iniciar solicitudes de autoservicio para [ampliar o renovar las asignaciones de roles](pim-resource-roles-renew-extend.md).

## <a name="require-multi-factor-authentication"></a>Requerir autenticación multifactor

Privileged Identity Management proporciona la aplicación opcional de Azure Multi-Factor Authentication para dos escenarios distintos.

### <a name="require-multi-factor-authentication-on-active-assignment"></a>Requerir autenticación multifactor para las asignaciones activas

En algunos casos, es posible que quiera asignar un usuario a un rol durante un tiempo breve (por ejemplo, un día). En este caso, no es necesario que los usuarios asignados soliciten la activación. En este escenario, Privileged Identity Management no puede exigir la autenticación multifactor cuando el usuario usa su asignación de roles, porque ya está activa en el rol desde el momento en que se asignan.

Para asegurarse de que el administrador que realiza la asignación es quien dicen ser, puede exigir la autenticación multifactor en la asignación activa; para ello, active la casilla **Requerir autenticación multifactor en la asignación activa**.

### <a name="require-multi-factor-authentication-on-activation"></a>Requerir Multi-Factor Authentication durante la activación

Puede requerir que los usuarios que son aptos para un rol demuestren quién están usando Azure Multi-Factor Authentication antes de que se puedan activar. La autenticación multifactor garantiza que el usuario sea quien dice ser con certeza razonable. Aplicar esta opción protege los recursos críticos en situaciones en las que es posible que la cuenta de usuario se haya puesto en peligro.

Para requerir la autenticación multifactor antes de la activación, active la casilla **Requerir autenticación multifactor en activación** en la pestaña Asignación de **Edición de la configuración de roles**.

Para obtener más información, consulte [Autenticación multifactor y Privileged Identity Management](pim-how-to-require-mfa.md).

## <a name="activation-maximum-duration"></a>Duración máxima de la activación

Use control deslizante **Duración máxima de la activación (horas)** para establecer el tiempo máximo, en horas, que un rol permanece activo antes de expirar. Este valor puede oscilar entre una y 24 horas.

## <a name="require-justification"></a>Requerir justificación

Puede requerir que los usuarios escriban una justificación empresarial cuando se activen. Para requerir justificación, active la casilla **Requerir justificación para las asignaciones activas** o la casilla **Requerir justificación en las activaciones**.

## <a name="require-approval-to-activate"></a>Solicitud de aprobación para activar

Si se establecen varios aprobadores, la aprobación se completa en cuanto uno de ellos aprueba o deniega. No se puede exigir la aprobación de al menos dos usuarios. Para solicitar aprobación a fin de activar un rol, siga estos pasos.

1. Active la casilla **Se requiere aprobación para activar**.

1. Seleccione **Seleccionar aprobadores**.

    ![Seleccionar un panel de usuarios o grupos para seleccionar aprobadores](./media/pim-resource-roles-configure-role-settings/resources-role-settings-select-approvers.png)

1. Seleccione al menos un usuario y haga clic en **Seleccionar**. Debe seleccionar al menos un aprobador. No hay aprobadores predeterminados.

    Su elección aparecerá en la lista de aprobadores seleccionados.

1. Una vez que haya especificado todos los valores de rol, seleccione **Actualizar** para guardar los cambios.

# <a name="previous-version"></a>[Versión anterior](#tab/previous)

## <a name="open-role-settings"></a>Apertura de la configuración de roles

Siga estos pasos para abrir la configuración de un rol de Azure AD.

1. Abra **Azure AD Privileged Identity Management**.

1. Seleccione **Roles de Azure AD**.

1. Seleccione **Configuración**.

    ![Roles de AD Azure: configuración](./media/pim-how-to-change-default-settings/pim-directory-roles-settings.png)

1. Seleccione **Roles**.

1. Seleccione el rol cuya configuración desea configurar.

    ![Roles de AD Azure: roles de configuración](./media/pim-how-to-change-default-settings/pim-directory-roles-settings-role.png)

    En la página de configuración de cada rol, hay varios valores que puede configurar. Dichos valores solo afectan a los usuarios que son asignaciones **elegibles**, no asignaciones **permanentes**.

## <a name="activations"></a>Activaciones

Use control deslizante **Activaciones** para establecer el tiempo máximo, en horas, que un rol permanece activo antes de expirar. Este valor puede oscilar entre 1 y 72 horas.

## <a name="notifications"></a>Notificaciones

Use el conmutador **Notificaciones** para especificar si los administradores recibirán notificaciones por correo electrónico cuando se activan los roles. Esta notificación puede ser útil para detectar activaciones no autorizadas o ilícitas.

Cuando se establece en **Habilitar**, las notificaciones se envían al:

- Administrador de roles con privilegios
- Administrador de seguridad
- Administrador global

Para más información, consulte [Notificaciones por correo electrónico en Privileged Identity Management](pim-email-notifications.md).

## <a name="incidentrequest-ticket"></a>Vale de solicitud o incidencia

Use el modificador **Vale de solicitud o incidente** para requerir que los administradores válidos incluyan un número de vale al activar su rol. Esta práctica puede hacer que las auditorías de acceso de los roles sean más eficaces.

## <a name="multi-factor-authentication"></a>Multi-Factor Authentication

Use el modificador **Autenticación multifactor** para especificar si se requiere que los usuarios comprueben su identidad con MFA antes de poder activar sus roles. Solo tienen que comprobar su identidad una vez por sesión, no cada vez que activan un rol. Al habilitar MFA, es conveniente tener en cuenta dos sugerencias:

- Los usuarios que tienen cuentas de Microsoft para sus direcciones de correo electrónico (normalmente @outlook.com, pero no siempre) no se pueden registrar en Azure Multi-Factor Authentication. Si quiere asignar roles a los usuarios con cuentas de Microsoft, debe establecerlos en administradores permanentes o deshabilitar la autenticación multifactor para ese rol.
- No se puede deshabilitar Azure Multi-Factor Authentication para roles con privilegios elevados en Azure AD y Office 365. Esta característica de seguridad ayuda a proteger los roles siguientes:  
  
  - Administrador de Azure Information Protection
  - Administrador de facturación
  - Administrador de aplicaciones en la nube
  - Administrador de cumplimiento
  - Administrador de acceso condicional
  - Administrador de Dynamics 365
  - Aprobador del acceso a la Caja de seguridad del cliente
  - Escritores de directorios
  - Administrador de Exchange
  - Administrador global
  - Administrador de Intune
  - Administrador de Power BI
  - Administrador de roles con privilegios
  - Administrador de seguridad
  - Administrador de SharePoint
  - Administrador de Skype Empresarial
  - Administrador de usuarios

Para obtener más información, consulte [Autenticación multifactor y Privileged Identity Management](pim-how-to-require-mfa.md).

## <a name="require-approval"></a>Requerir aprobación

Si desea delegar la aprobación necesaria para activar un rol, siga estos pasos.

1. Establezca el modificador **Requerir aprobación** en **Habilitado**. El panel se expande con opciones para seleccionar aprobadores.

    ![Roles de Azure AD - Configuración - Requerir aprobación](./media/pim-how-to-change-default-settings/pim-directory-roles-settings-require-approval.png)

    Si no especifica ningún aprobador, el administrador de roles con privilegios se convierte en el aprobador predeterminado y, a continuación, se le solicita que apruebe todas las solicitudes de activación para este rol.

1. Para agregar aprobadores, haga clic en **Seleccionar aprobadores**.

    ![Roles de Azure AD - Configuración - Requerir aprobación](./media/pim-how-to-change-default-settings/pim-directory-roles-settings-require-approval-select-approvers.png)

1. Seleccione uno o varios aprobadores además del administrador de roles con privilegios y, a continuación, haga clic en **Seleccionar**. Se recomienda agregar al menos dos aprobadores. Incluso si se agrega a sí mismo como aprobador, no se puede aprobar a sí mismo una activación de rol. Su elección aparecerá en la lista de aprobadores seleccionados.

1. Una vez que haya especificado todos los valores del rol, seleccione **Guardar** para guardar los cambios.

---

## <a name="next-steps"></a>Pasos siguientes

- [Asignación de roles de Azure AD en Privileged Identity Management](pim-how-to-add-role-to-user.md)
- [Configuración de alertas de seguridad para roles de Azure AD en Privileged Identity Management](pim-how-to-configure-security-alerts.md)
