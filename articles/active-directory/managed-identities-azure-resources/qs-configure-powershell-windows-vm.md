---
title: 'Configuración de identidades administradas en una máquina virtual de Azure mediante PowerShell: Azure AD'
description: Instrucciones detalladas para configurar identidades administradas de recursos de Azure en una máquina virtual de Azure mediante PowerShell.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 08/19/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: cbbe4dbb7c7db0ca17a9ddddf5016bdc52b9e93d
ms.sourcegitcommit: bcda98171d6e81795e723e525f81e6235f044e52
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89269359"
---
# <a name="configure-managed-identities-for-azure-resources-on-an-azure-vm-using-powershell"></a>Configuración de identidades administradas de recursos de Azure en una VM de Azure mediante PowerShell

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory. Puede usar esta identidad para autenticar a cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código.

En este artículo, con PowerShell, aprenderá a usar las siguientes identidades administradas para operaciones de recursos de Azure en una máquina virtual de Azure.

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Requisitos previos

- Si no está familiarizado con las identidades administradas de los recursos de Azure, consulte la [sección de introducción](overview.md). **No olvide revisar la [diferencia entre una identidad administrada asignada por el sistema y una identidad administrada asignada por el usuario](overview.md#managed-identity-types)** .
- Si aún no tiene una cuenta de Azure, [regístrese para una cuenta gratuita](https://azure.microsoft.com/free/) antes de continuar.
- Instale [la versión más reciente de Azure PowerShell](/powershell/azure/install-az-ps) si aún no lo ha hecho.

## <a name="system-assigned-managed-identity"></a>Identidad administrada asignada por el sistema

En esta sección, aprenderá a habilitar y deshabilitar la identidad administrada asignada por el sistema con Azure PowerShell.

### <a name="enable-system-assigned-managed-identity-during-creation-of-an-azure-vm"></a>Habilitación de una identidad administrada asignada por el sistema durante la creación de una VM de Azure

Para crear una máquina virtual de Azure que tenga habilitada la identidad administrada asignada por el sistema, la cuenta debe tener la asignación de roles [Colaborador de la máquina Virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Consulte una de las siguientes guías de inicio rápido sobre máquinas virtuales de Azure y finalice solo las secciones necesarias ("Iniciar sesión en Azure", "Creación de un grupo de recursos", "Creación de un grupo de red", "Creación de la máquina virtual").

    Cuando llegue a la sección "Creación de la máquina virtual", realice una pequeña modificación en la sintaxis del cmdlet [New-AzVMConfig](/powershell/module/az.compute/new-azvm). Asegúrese de agregar un parámetro `-IdentityType SystemAssigned` para aprovisionar la máquina virtual con la identidad asignada por el sistema habilitada, por ejemplo:

    ```powershell
    $vmConfig = New-AzVMConfig -VMName myVM -IdentityType SystemAssigned ...
    ```

   - [Creación de una máquina virtual Windows con PowerShell](../../virtual-machines/windows/quick-create-powershell.md)
   - [Creación de una máquina virtual Linux con PowerShell](../../virtual-machines/linux/quick-create-powershell.md)



### <a name="enable-system-assigned-managed-identity-on-an-existing-azure-vm"></a>Habilitación de una identidad administrada asignada por el sistema en una VM de Azure existente

Para habilitar una identidad administrada asignada por el sistema en una máquina virtual que en un principio se aprovisionó sin dicha identidad, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Inicie sesión en Azure con `Connect-AzAccount`. Use una cuenta asociada a la suscripción de Azure que contenga la máquina virtual.

   ```powershell
   Connect-AzAccount
   ```

2. En primer lugar, recupere las propiedades de la máquina virtual mediante el cmdlet `Get-AzVM`. Después, para habilitar una identidad administrada asignada por el sistema, use el modificador `-IdentityType` en el cmdlet [Update-AzVM](/powershell/module/az.compute/update-azvm):

   ```powershell
   $vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
   Update-AzVM -ResourceGroupName myResourceGroup -VM $vm -IdentityType SystemAssigned
   ```



### <a name="add-vm-system-assigned-identity-to-a-group"></a>Adición de una identidad asignada por el sistema de una VM a un grupo

Después de haber habilitado la identidad asignada por el sistema en una VM, puede agregarla a un grupo.  Con el siguiente procedimiento se agrega la identidad asignada por el sistema de la VM a un grupo.

1. Inicie sesión en Azure con `Connect-AzAccount`. Use una cuenta asociada a la suscripción de Azure que contenga la máquina virtual.

   ```powershell
   Connect-AzAccount
   ```

2. Recupere y anote el `ObjectID` (según lo especificado en el campo `Id` de valores devueltos) de la entidad de servicio de la VM:

   ```powershell
   Get-AzADServicePrincipal -displayname "myVM"
   ```

3. Recupere y anote el `ObjectID` (según lo especificado en el campo `Id` de valores devueltos) del grupo:

   ```powershell
   Get-AzADGroup -searchstring "myGroup"
   ```

4. Agregue la entidad de servicio de la VM al grupo:

   ```powershell
   Add-AzureADGroupMember -ObjectId "<objectID of group>" -RefObjectId "<object id of VM service principal>"
   ```

## <a name="disable-system-assigned-managed-identity-from-an-azure-vm"></a>Deshabilitación de una identidad administrada asignada por el sistema de una VM de Azure

Para deshabilitar una identidad administrada asignada por el sistema en una máquina virtual, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).  No se requiere ninguna otra asignación de roles de directorio de Azure AD.

Si tiene una máquina virtual que ya no necesita la identidad administrada asignada por el sistema, pero aún necesita identidades administradas asignadas por el usuario, use el siguiente cmdlet:

1. Inicie sesión en Azure con `Connect-AzAccount`. Use una cuenta asociada a la suscripción de Azure que contenga la máquina virtual.

   ```powershell
   Connect-AzAccount
   ```

2. Recupere las propiedades de la máquina virtual mediante el cmdlet `Get-AzVM` y establezca el parámetro `-IdentityType` en `UserAssigned`:

   ```powershell
   $vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
   Update-AzVm -ResourceGroupName myResourceGroup -VM $vm -IdentityType "UserAssigned"
   ```

Si tiene una máquina virtual que ya no necesita la identidad administrada asignada por el sistema y no tiene identidades administradas asignadas por el usuario, use los siguientes comandos:

```powershell
$vm = Get-AzVM -ResourceGroupName myResourceGroup -Name myVM
Update-AzVm -ResourceGroupName myResourceGroup -VM $vm -IdentityType None
```



## <a name="user-assigned-managed-identity"></a>Identidad administrada asignada por el usuario

En esta sección, aprenderá a agregar y quitar una identidad administrada asignada por el usuario de una máquina virtual con Azure PowerShell.

### <a name="assign-a-user-assigned-managed-identity-to-a-vm-during-creation"></a>Asignación de una identidad administrada asignada por el usuario a una máquina virtual durante la creación

Para asignar una identidad asignada por un usuario a una máquina virtual, la cuenta debe tener las asignaciones de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) y [Operador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-operator). No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Consulte una de las siguientes guías de inicio rápido sobre máquinas virtuales de Azure y finalice solo las secciones necesarias ("Iniciar sesión en Azure", "Creación de un grupo de recursos", "Creación de un grupo de red", "Creación de la máquina virtual").

    Cuando llegue a la sección "Creación de la máquina virtual", realice una pequeña modificación en la sintaxis del cmdlet [`New-AzVMConfig`](/powershell/module/az.compute/new-azvm). Agregue los parámetros `-IdentityType UserAssigned` y `-IdentityID` para aprovisionar la máquina virtual con una identidad asignada por el usuario.  Reemplace `<VM NAME>`, `<SUBSCRIPTION ID>`, `<RESROURCE GROUP>` y `<USER ASSIGNED IDENTITY NAME>` con sus propios valores.  Por ejemplo:

    ```powershell
    $vmConfig = New-AzVMConfig -VMName <VM NAME> -IdentityType UserAssigned -IdentityID "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESROURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>..."
    ```

    - [Creación de una máquina virtual Windows con PowerShell](../../virtual-machines/windows/quick-create-powershell.md)
    - [Creación de una máquina virtual Linux con PowerShell](../../virtual-machines/linux/quick-create-powershell.md)



### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-vm"></a>Asignación de una identidad administrada asignada por el usuario a una VM de Azure existente

Para asignar una identidad asignada por un usuario a una máquina virtual, la cuenta debe tener las asignaciones de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) y [Operador de identidades administradas](../../role-based-access-control/built-in-roles.md#managed-identity-operator). No se requiere ninguna otra asignación de roles de directorio de Azure AD.

1. Inicie sesión en Azure con `Connect-AzAccount`. Use una cuenta asociada a la suscripción de Azure que contenga la máquina virtual.

   ```powershell
   Connect-AzAccount
   ```

2. Cree una identidad administrada asignada por el usuario mediante el cmdlet [New-AzUserAssignedIdentity](/powershell/module/az.managedserviceidentity/new-azuserassignedidentity).  Anote `Id` en la salida, porque lo necesitará en el paso siguiente.

   > [!IMPORTANT]
   > La creación de identidades administradas asignadas por el usuario solo admite caracteres alfanuméricos, subrayados y guiones (0-9, a-z, A-Z, \_ o -). Además, el nombre debe limitarse a una longitud de entre 3 y 128 caracteres para que la asignación a VM/VMSS funcione correctamente. Para obtener más información, consulte [Preguntas más frecuentes y problemas conocidos](known-issues.md).

   ```powershell
   New-AzUserAssignedIdentity -ResourceGroupName <RESOURCEGROUP> -Name <USER ASSIGNED IDENTITY NAME>
   ```
3. Recupere las propiedades de la máquina virtual mediante el cmdlet `Get-AzVM`. A continuación, para asignar una identidad administrada asignada por el usuario a la máquina virtual de Azure, use los modificadores `-IdentityType` y `-IdentityID` en el cmdlet [Update-AzVM](/powershell/module/az.compute/update-azvm).  El valor del parámetro `-IdentityId` es el `Id` que ha anotado en el paso anterior.  Reemplace `<VM NAME>`, `<SUBSCRIPTION ID>`, `<RESROURCE GROUP>` y `<USER ASSIGNED IDENTITY NAME>` con sus propios valores.

   > [!WARNING]
   > Para conservar las identidades administradas previamente asignadas al usuario, pero asignadas ahora a la máquina virtual, consulte la propiedad `Identity` del objeto de la máquina virtual (por ejemplo, `$vm.Identity`).  Si se devuelven identidades administradas asignadas al usuario, inclúyalas en el siguiente comando junto con la nueva identidad administrada asignada por el usuario que desea asignar a la máquina virtual.

   ```powershell
   $vm = Get-AzVM -ResourceGroupName <RESOURCE GROUP> -Name <VM NAME>
   Update-AzVM -ResourceGroupName <RESOURCE GROUP> -VM $vm -IdentityType UserAssigned -IdentityID "/subscriptions/<SUBSCRIPTION ID>/resourcegroups/<RESROURCE GROUP>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<USER ASSIGNED IDENTITY NAME>"
   ```



### <a name="remove-a-user-assigned-managed-identity-from-an-azure-vm"></a>Eliminación de una identidad administrada asignada por el usuario de una VM de Azure

Para quitar una identidad asignada por un usuario de una máquina virtual, la cuenta necesita la asignación de roles [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor).

Si la máquina virtual tiene varias identidades administradas asignadas por el usuario, puede quitar todas, menos la última, mediante los comandos siguientes. Asegúrese de reemplazar los valores de los parámetros `<RESOURCE GROUP>` y `<VM NAME>` con sus propios valores. `<USER ASSIGNED IDENTITY NAME>` es la propiedad de nombre de la identidad administrada asignada por el usuario, que debe permanecer en la máquina virtual. Esta información puede encontrarse consultando la propiedad `Identity` del objeto de la máquina virtual.  Por ejemplo, `$vm.Identity`:

```powershell
$vm = Get-AzVm -ResourceGroupName myResourceGroup -Name myVm
Update-AzVm -ResourceGroupName myResourceGroup -VirtualMachine $vm -IdentityType UserAssigned -IdentityID <USER ASSIGNED IDENTITY NAME>
```
Si la máquina virtual no tiene una identidad administrada asignada por el sistema y desea quitar de ella todas las identidades administradas asignadas por el usuario, utilice el siguiente comando:

```powershell
$vm = Get-AzVm -ResourceGroupName myResourceGroup -Name myVm
Update-AzVm -ResourceGroupName myResourceGroup -VM $vm -IdentityType None
```
Si la máquina virtual tiene identidades administradas asignadas tanto por el sistema como por el usuario, puede quitar todas las identidades administradas asignadas por el usuario si cambia para usar solo las identidades administradas asignadas por el sistema.

```powershell
$vm = Get-AzVm -ResourceGroupName myResourceGroup -Name myVm
Update-AzVm -ResourceGroupName myResourceGroup -VirtualMachine $vm -IdentityType "SystemAssigned"
```

## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre las identidades administradas de recursos de Azure](overview.md)
- Para ver guías de inicio rápido completas acerca de la creación de máquinas virtuales de Azure, consulte:

  - [Creación de una máquina virtual Windows con PowerShell](../../virtual-machines/windows/quick-create-powershell.md)
  - [Creación de una máquina virtual Linux con PowerShell](../../virtual-machines/linux/quick-create-powershell.md)