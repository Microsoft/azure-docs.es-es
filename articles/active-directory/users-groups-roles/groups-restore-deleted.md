---
title: Restauración de un grupo de Microsoft 365 eliminado en Azure AD | Microsoft Docs
description: Restauración de un grupo eliminado, visualización de grupos restaurables y eliminación permanente de un grupo en Azure Active Directory
services: active-directory
author: curtand
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.subservice: users-groups-roles
ms.topic: quickstart
ms.date: 08/13/2020
ms.author: curtand
ms.reviewer: krbain
ms.custom: it-pro, seo-update-azuread-jan
ms.collection: M365-identity-device-management
ms.openlocfilehash: 133371007702ce6443455d7381722fdbbc79eabc
ms.sourcegitcommit: 4913da04fd0f3cf7710ec08d0c1867b62c2effe7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/14/2020
ms.locfileid: "88213660"
---
# <a name="restore-a-deleted-microsoft-365-group-in-azure-active-directory"></a>Restauración de un grupo de Microsoft 365 eliminado en Azure Active Directory

Cuando se elimina un grupo de Microsoft 365 en Azure Active Directory (Azure AD), el grupo eliminado queda retenido y deja de estar visible durante 30 días a contar desde la fecha de eliminación. Este comportamiento es tal para que el grupo y su contenido se puedan restaurar en caso necesario. Esta funcionalidad está restringida exclusivamente a los grupos de Microsoft 365 en Azure AD. No está disponible para los grupos de seguridad ni los grupos de distribución. Tenga en cuenta que el período de 30 días de restauración del grupo no es personalizable.

> [!NOTE]
> No utilice `Remove-MsolGroup` porque purga el grupo de forma permanente. Utilice siempre `Remove-AzureADMSGroup` para eliminar un grupo de Microsoft 365.

Los permisos necesarios para restaurar un grupo pueden ser cualquiera de los siguientes:

Role | Permisos
--------- | ---------
Administrador global, Administrador de grupo, Soporte para asociados de nivel 2 y Administrador de Intune | Puede restaurar cualquier grupo de Microsoft 365 eliminado
Administrador de usuarios y soporte técnico de nivel 1 para asociados | Puede restaurar cualquier grupo de Microsoft 365 eliminado, excepto los que tienen asignado el rol Administrador de la compañía
Usuario | Puede restaurar cualquier grupo de Microsoft 365 eliminado de su propiedad

## <a name="view-and-manage-the-deleted-microsoft-365-groups-that-are-available-to-restore"></a>Ver y administrar los grupos de Microsoft 365 eliminados disponibles para la restauración

1. Inicie sesión en el [Centro de administración de Azure AD](https://aad.portal.azure.com) con una cuenta de administrador de usuarios.

2. Seleccione **Grupos** y, a continuación, seleccione **Grupos eliminados** para ver los grupos eliminados que están disponibles para restaurar.

    ![ver los grupos que están disponibles para restaurar](media/groups-lifecycle/deleted-groups3.png)

3. En la hoja **Grupos eliminados**, puede:

   - Seleccionar **Restaurar grupo** para restaurar el grupo eliminado y su contenido.
   - Quite de forma permanente el grupo eliminado mediante la selección de **Eliminar permanentemente**. Para quitar de forma definitiva un grupo, debe ser un administrador.

## <a name="view-the-deleted-microsoft-365-groups-that-are-available-to-restore-using-powershell"></a>Ver los grupos de Microsoft 365 eliminados disponibles para restaurar con PowerShell

Los cmdlets siguientes se pueden usar para ver los grupos eliminados y comprobar que todavía no se purgaron los que le interesan. Estos cmdlets son parte del [módulo de Azure AD PowerShell](https://www.powershellgallery.com/packages/AzureAD/). Puede encontrar más información sobre este módulo en el artículo sobre la [versión 2 de Azure Active Directory PowerShell](/powershell/azure/active-directory/install-adv2?view=azureadps-2.0).

1.  Ejecute el cmdlet siguiente para mostrar todos los grupos de Microsoft 365 eliminados de la organización de Azure AD que aún se pueden restaurar.
   

    ```powershell
    Get-AzureADMSDeletedGroup
    ```

2.  Como alternativa, si conoce el id. de objeto de un grupo específico (y puede obtenerlo del cmdlet del paso 1), ejecute el cmdlet siguiente para comprobar que el grupo eliminado específico todavía no se purgó de forma permanente.

    ```
    Get-AzureADMSDeletedGroup –Id <objectId>
    ```

## <a name="how-to-restore-your-deleted-microsoft-365-group-using-powershell"></a>Procedimiento para restaurar grupos de Microsoft 365 eliminados con PowerShell

Una vez que haya comprobado que el grupo todavía se puede restaurar, siga uno de los pasos siguientes para restaurar el grupo eliminado. Si el grupo contiene documentos, sitios de SP u otros objetos persistentes, el proceso completo de restauración de un grupo y su contenido podría demorar hasta 24 horas.

1. Ejecute el cmdlet siguiente para restaurar el grupo y su contenido.
 

   ```
    Restore-AzureADMSDeletedDirectoryObject –Id <objectId>
    ``` 

2. Como alternativa, puede ejecutar el cmdlet siguiente para quitar el grupo eliminado de forma permanente.
    

    ```
    Remove-AzureADMSDeletedDirectoryObject –Id <objectId>
    ```

## <a name="how-do-you-know-this-worked"></a>¿Cómo se puede saber si funcionó?

Para comprobar que restauró correctamente un grupo de Microsoft 365, ejecute el cmdlet `Get-AzureADGroup –ObjectId <objectId>` para mostrar información sobre el grupo. Una vez que la solicitud de restauración se complete:

- El grupo aparece en la barra de navegación izquierda de Exchange
- El plan del grupo aparecerá en Planner
- Los sitios de SharePoint y su contenido estarán disponibles
- Se podrá tener acceso al grupo desde cualquiera de los puntos de conexión de Exchange y otras cargas de trabajo de Microsoft 365 que admitan grupos de Microsoft 365.

## <a name="next-steps"></a>Pasos siguientes

En estos artículos se proporciona información adicional sobre los grupos de Azure Active Directory.

* [Consulta de los grupos existentes](../fundamentals/active-directory-groups-view-azure-portal.md)
* [Administración de la configuración de un grupo](../fundamentals/active-directory-groups-settings-azure-portal.md)
* [Administrar miembros de un grupo](../fundamentals/active-directory-groups-members-azure-portal.md)
* [Administrar la pertenencia a grupos](../fundamentals/active-directory-groups-membership-azure-portal.md)
* [Administrar reglas dinámicas de los usuarios de un grupo](groups-dynamic-membership.md)
