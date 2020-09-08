---
title: 'Asignación de roles para entidades de servicio de Windows Virtual Desktop (clásico): Azure'
description: Creación de entidades de servicio y asignación de roles con PowerShell en la versión preliminar de Windows Virtual Desktop (clásico).
author: Heidilohr
ms.topic: tutorial
ms.date: 05/27/2020
ms.author: helohr
ms.custom: devx-track-azurepowershell
manager: lizross
ms.openlocfilehash: 91bb14a174d5bd5c16b38513825579097e1d6f7f
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89078535"
---
# <a name="tutorial-create-service-principals-and-role-assignments-with-powershell-in-windows-virtual-desktop-classic"></a>Tutorial: Creación de entidades de servicio y asignaciones de roles con PowerShell en Windows Virtual Desktop (clásico)

>[!IMPORTANT]
>Este contenido se aplica a Windows Virtual Desktop (clásico), que no admite objetos de Windows Virtual Desktop para Azure Resource Manager.

Las entidades de servicio son identidades que puede crear en Azure Active Directory para asignar roles y permisos para un propósito específico. En Windows Virtual Desktop puede crear una entidad de servicio para:

- Automatizar tareas de administración específicas de Windows Virtual Desktop.
- Usarla como credenciales en lugar de los usuarios necesarios para la autenticación multifactor al ejecutar cualquier plantilla de Azure Resource Manager para Windows Virtual Desktop.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Cree una entidad de servicio en Azure Active Directory.
> * Crear una asignación de roles en Windows Virtual Desktop.
> * Iniciar sesión en Windows Virtual Desktop con la entidad de servicio.

## <a name="prerequisites"></a>Prerrequisitos

Para poder crear entidades de servicio y asignaciones de roles, necesita hacer tres cosas:

1. Instalar el módulo de Azure AD. Instalar el módulo, ejecutar PowerShell como administrador y ejecutar el siguiente cmdlet:

    ```powershell
    Install-Module AzureAD
    ```

2. [Descargue e importe el módulo de PowerShell para Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/).

3. Seguir todas las instrucciones de este artículo en la misma sesión de PowerShell. Es posible que el proceso no funcione si interrumpe la sesión de PowerShell al cerrar la ventana y volver a abrirla más adelante.

## <a name="create-a-service-principal-in-azure-active-directory"></a>Creación de una entidad de servicio en Azure Active Directory

Una vez cumplidos los requisitos previos en la sesión de PowerShell, ejecute los siguientes cmdlets de PowerShell para crear una entidad de servicio multiinquilino en Azure.

```powershell
Import-Module AzureAD
$aadContext = Connect-AzureAD
$svcPrincipal = New-AzureADApplication -AvailableToOtherTenants $true -DisplayName "Windows Virtual Desktop Svc Principal"
$svcPrincipalCreds = New-AzureADApplicationPasswordCredential -ObjectId $svcPrincipal.ObjectId
```
## <a name="view-your-credentials-in-powershell"></a>Visualización de las credenciales en PowerShell

Antes de crear la asignación de roles para la entidad de servicio, consulte sus credenciales y escríbalas para futura referencia. La contraseña es especialmente importante, porque no se podrá recuperar una vez que se cierre esta sesión de PowerShell.

Estas son las tres credenciales que debe anotar y los cmdlets que necesita para ejecutar para obtenerlas:

- Contraseña:

    ```powershell
    $svcPrincipalCreds.Value
    ```

- Identificador de inquilino:

    ```powershell
    $aadContext.TenantId.Guid
    ```

- Identificador de aplicación:

    ```powershell
    $svcPrincipal.AppId
    ```

## <a name="create-a-role-assignment-in-windows-virtual-desktop"></a>Creación de una asignación de roles en Windows Virtual Desktop

A continuación debe crear una asignación de roles para que la entidad de servicio pueda iniciar sesión en Windows Virtual Desktop. Asegúrese de que iniciar sesión con una cuenta que tenga permisos para crear la asignación de roles.

En primer lugar y, si aún no lo ha hecho, [descargue e importe el módulo de PowerShell para Windows Virtual Desktop](/powershell/windows-virtual-desktop/overview/) que se usará en la sesión de PowerShell.

Ejecute los siguientes cmdlets de PowerShell para conectarse a Windows Virtual Desktop y mostrar los inquilinos.

```powershell
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com"
Get-RdsTenant
```

Cuando encuentre el nombre del inquilino para el que desea crear una asignación de roles, úselo en el siguiente cmdlet:

```powershell
$myTenantName = "<Windows Virtual Desktop Tenant Name>"
New-RdsRoleAssignment -RoleDefinitionName "RDS Owner" -ApplicationId $svcPrincipal.AppId -TenantName $myTenantName
```

## <a name="sign-in-with-the-service-principal"></a>Inicio de sesión con la entidad de servicio

Después de crear una asignación de roles para la entidad de servicio, asegúrese de que esta puede iniciar sesión en Windows Virtual Desktop mediante la ejecución del siguiente cmdlet:

```powershell
$creds = New-Object System.Management.Automation.PSCredential($svcPrincipal.AppId, (ConvertTo-SecureString $svcPrincipalCreds.Value -AsPlainText -Force))
Add-RdsAccount -DeploymentUrl "https://rdbroker.wvd.microsoft.com" -Credential $creds -ServicePrincipal -AadTenantId $aadContext.TenantId.Guid
```

Una vez iniciada la sesión, pruebe algunos cmdlets de PowerShell en Windows Virtual Desktop con la entidad de servicio para asegurarse de que todo funciona.

## <a name="next-steps"></a>Pasos siguientes

Una vez que ha creado la entidad de servicio y le ha asignado un rol en el inquilino de Windows Virtual Desktop, puede usarla para crear un grupo host. Para más información acerca de los grupos host, continúe con el tutorial de creación de un grupo host en Windows Virtual Desktop.

 > [!div class="nextstepaction"]
 > [Creación de un grupo host con Azure Marketplace](create-host-pools-azure-marketplace-2019.md)
