---
title: 'Asignación de etiquetas de confidencialidad a grupos: Azure AD | Microsoft Docs'
description: Creación de reglas de pertenencia para rellenar automáticamente grupos y creación de una referencia de regla.
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: how-to
ms.date: 08/13/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: 46d692b81d24b6c5088ffc42644ed1dd7f45b2d2
ms.sourcegitcommit: c5021f2095e25750eb34fd0b866adf5d81d56c3a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/25/2020
ms.locfileid: "88795315"
---
# <a name="assign-sensitivity-labels-to-microsoft-365-groups-in-azure-active-directory"></a>Asignación de etiquetas de confidencialidad a grupos de Microsoft 365 en Azure Active Directory

Azure Active Directory (Azure AD) permite aplicar etiquetas de confidencialidad publicadas por el [Centro de cumplimiento de Microsoft 365](https://sip.protection.office.com/homepage) a los grupos de Microsoft 365. Las etiquetas de confidencialidad se aplican a grupos de servicios como Outlook, Microsoft Teams y SharePoint. Esta característica está actualmente en disponibilidad general. Para más información sobre el soporte técnico de aplicaciones de Office 365, consulte [Compatibilidad de Office 365 con las etiquetas de confidencialidad](/microsoft-365/compliance/sensitivity-labels-teams-groups-sites#support-for-the-sensitivity-labels).

> [!IMPORTANT]
> Para configurar esta característica, debe haber al menos una licencia activa de Azure Active Directory Premium P1 en la organización de Azure AD.

## <a name="enable-sensitivity-label-support-in-powershell"></a>Habilitación de la compatibilidad con la etiqueta de confidencialidad en PowerShell

Para aplicar etiquetas publicadas a grupos, primero debe habilitar la característica. Estos pasos permiten habilitar la característica en Azure AD.

1. Abra una ventana de Windows PowerShell en el equipo. La puede abrir sin privilegios elevados.
1. Ejecute los comandos siguientes para prepararse para ejecutar los cmdlets.

    ```PowerShell
    Import-Module AzureADPreview
    Connect-AzureAD
    ```

    En la página **Iniciar sesión en tu cuenta**, escriba la cuenta y la contraseña de administrador para conectarse al servicio y seleccione **Iniciar sesión**.
1. Capture la configuración de grupo actual para la organización de Azure AD.

    ```PowerShell
    $Setting = Get-AzureADDirectorySetting -Id (Get-AzureADDirectorySetting | where -Property DisplayName -Value "Group.Unified" -EQ).id
    ```

    > [!NOTE]
    > Si no se ha creado ninguna configuración de grupo para esta organización de Azure AD, primero debe crearla. Siga los pasos de [Cmdlets de Azure Active Directory para configurar las opciones de grupo](./groups-settings-cmdlets.md) y cree una configuración de grupo para esta organización de Azure AD.

1. A continuación, muestre la configuración actual del grupo.

    ```PowerShell
    $Setting.Values
    ```

1. Después, habilite la característica:

    ```PowerShell
    $Setting["EnableMIPLabels"] = "True"
    ```

1. Finalmente, guarde los cambios y aplique la configuración:

    ```PowerShell
    Set-AzureADDirectorySetting -Id $Setting.Id -DirectorySetting $Setting
    ```

Eso es todo. Ha habilitado la característica y puede aplicar las etiquetas publicadas a los grupos.

## <a name="assign-a-label-to-a-new-group-in-azure-portal"></a>Asignación de una etiqueta a un grupo nuevo en Azure Portal

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com).
1. Seleccione **Grupos** y, a continuación, seleccione **Nuevo grupo**.
1. En la página **Nuevo grupo**, seleccione **Office 365** y, a continuación, rellene la información necesaria para el nuevo grupo y seleccione una etiqueta de confidencialidad de la lista.

   ![Asignación de una etiqueta de confidencialidad en la página Nuevos grupos](./media/groups-assign-sensitivity-labels/new-group-page.png)

1. Guarde los cambios y seleccione **Crear**.

Se creará el grupo, y se aplicará automáticamente la configuración del sitio web y del grupo asociada a la etiqueta seleccionada.

## <a name="assign-a-label-to-an-existing-group-in-azure-portal"></a>Asignación de una etiqueta a un grupo actual en Azure Portal

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com) con una cuenta de administradores de grupos o como propietario del grupo.
1. Seleccione **Grupos**.
1. En la página **Todos los grupos**, seleccione el grupo que desea etiquetar.
1. En la página del grupo seleccionado, seleccione **Propiedades** y una etiqueta de confidencialidad en la lista.

   ![Asignación de una etiqueta de confidencialidad en la página de información general de un grupo](./media/groups-assign-sensitivity-labels/assign-to-existing.png)

1. Haga clic en **Guardar** para guardar los cambios.

## <a name="remove-a-label-from-an-existing-group-in-azure-portal"></a>Eliminación de una etiqueta de un grupo actual en Azure Portal

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com) con una cuenta de administrador global o de administradores de grupos, o como propietario del grupo.
1. Seleccione **Grupos**.
1. En la página **Todos los grupos**, seleccione el grupo del que desee quitar la etiqueta.
1. En la página **Grupo**, seleccione **Propiedades**.
1. Seleccione **Quitar**.
1. Seleccione **Guardar** para aplicar los cambios.

## <a name="using-classic-azure-ad-classifications"></a>Uso de las clasificaciones de Azure AD clásicas

Después de habilitar esta característica, las clasificaciones "clásicas" para los grupos solo aparecerán en los grupos y sitios web existentes, y solo se deben usar para los nuevos grupos si se crean grupos en aplicaciones que no admiten etiquetas de confidencialidad. El administrador puede convertirlas en etiquetas de confidencialidad posteriormente si es necesario. Las clasificaciones clásicas son las antiguas que se configuran definiendo los valores de la configuración `ClassificationList` en Azure AD PowerShell. Si esta característica está habilitada, las clasificaciones no se aplicarán a los grupos.

## <a name="troubleshooting-issues"></a>Solución de problemas

### <a name="sensitivity-labels-are-not-available-for-assignment-on-a-group"></a>Las etiquetas de confidencialidad no están disponibles para la asignación en un grupo

La opción de etiqueta de confidencialidad solo se muestra para los grupos cuando se cumplen todas las condiciones siguientes:

1. Las etiquetas se publican en el Centro de cumplimiento de Microsoft 365 de esta organización de Azure AD.
1. La característica está habilitada, EnableMIPLabels está establecido en True en PowerShell.
1. El grupo es un grupo de Microsoft 365.
1. La organización tiene una licencia activa de Azure Active Directory Premium P1.
1. El usuario que inició la sesión actual tiene privilegios suficientes para asignar etiquetas. El usuario debe ser un administrador global, un administrador de grupo o el propietario del grupo.

Asegúrese de que se cumplen todas las condiciones para asignar etiquetas a un grupo.

### <a name="the-label-i-want-to-assign-is-not-in-the-list"></a>La etiqueta que quiero asignar no está en la lista

Si la etiqueta que está buscando no está en la lista, podría deberse a uno de los siguientes motivos:

- La etiqueta podría no estar publicada en el Centro de cumplimiento de Microsoft 365. Esto también se puede aplicar a las etiquetas que ya no se publican. Para más información, póngase en contacto con el administrador.
- No obstante, es posible que la etiqueta publicada no esté disponible para el usuario que haya iniciado la sesión. Para más información sobre cómo acceder a la etiqueta, consulte al administrador.

### <a name="how-to-change-the-label-on-a-group"></a>Cómo se puede cambiar la etiqueta en un grupo

Las etiquetas se pueden intercambiar en cualquier momento siguiendo los mismos pasos que para asignar una etiqueta a un grupo existente, como se indica a continuación:

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com) con una cuenta de administrador global o de administrador de grupo, o como propietario del grupo.
1. Seleccione **Grupos**.
1. En la página **Todos los grupos**, seleccione el grupo que desea etiquetar.
1. En la página del grupo seleccionado, seleccione **Propiedades** y una nueva etiqueta de confidencialidad en la lista.
1. Seleccione **Guardar**.

### <a name="group-setting-changes-to-published-labels-are-not-updated-on-the-groups"></a>Los cambios de configuración de grupo en las etiquetas publicadas no se actualizan en los grupos

Se recomienda que no cambie la configuración de un grupo después de aplicar la etiqueta a los grupos. Cuando realiza cambios en la configuración de grupo asociada a las etiquetas publicadas en el [Centro de cumplimiento de Microsoft 365](https://sip.protection.office.com/homepage), esos cambios de directiva no se aplican automáticamente a los grupos afectados.

Si debe realizar un cambio, use un [script de Azure AD PowerShell](https://github.com/microsoftgraph/powershell-aad-samples/blob/master/ReassignSensitivityLabelToO365Groups.ps1) para aplicar manualmente las actualizaciones a los grupos afectados. Este método garantiza que todos los grupos existentes apliquen la nueva configuración.

## <a name="next-steps"></a>Pasos siguientes

- [Uso de etiquetas de confidencialidad con Microsoft Teams, grupos de Microsoft 365 y sitios de SharePoint](/microsoft-365/compliance/sensitivity-labels-teams-groups-sites)
- [Actualización de grupos después de cambiar la directiva de etiquetas manualmente con el script de Azure AD PowerShell](https://github.com/microsoftgraph/powershell-aad-samples/blob/master/ReassignSensitivityLabelToO365Groups.ps1)
- [Edición de la configuración de un grupo](../fundamentals/active-directory-groups-settings-azure-portal.md)
- [Administrar grupos mediante los comandos de PowerShell](./groups-settings-v2-cmdlets.md)