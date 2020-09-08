---
title: 'Tutorial: Administración de máquinas virtuales con PowerShell'
description: En este tutorial, aprenderá a usar Azure PowerShell para administrar máquinas virtuales de Azure mediante la aplicación de RBAC, directivas, bloqueos y etiquetas
author: tfitzmac
ms.service: virtual-machines-windows
ms.workload: infrastructure
ms.topic: tutorial
ms.date: 12/05/2018
ms.author: tomfitz
ms.custom: mvc
ms.openlocfilehash: 723eaeb6eb8946473b31b447e817a0a3b696f1cc
ms.sourcegitcommit: 4f1c7df04a03856a756856a75e033d90757bb635
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/07/2020
ms.locfileid: "87926576"
---
# <a name="tutorial-learn-about-windows-virtual-machine-management-with-azure-powershell"></a>Tutorial: Información acerca de la administración de máquinas virtuales Windows con Azure PowerShell

[!INCLUDE [Resource Manager governance introduction](../../../includes/resource-manager-governance-intro.md)]

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

Azure Cloud Shell es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta. 

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede ir a [https://shell.azure.com/powershell](https://shell.azure.com/powershell) para iniciar Cloud Shell en una pestaña independiente del explorador. Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, luego, presione Entrar para ejecutarlos.

## <a name="understand-scope"></a>Descripción del ámbito

[!INCLUDE [Resource Manager governance scope](../../../includes/resource-manager-governance-scope.md)]

En este tutorial, se aplica toda la configuración de administración a un grupo de recursos para poder quitar fácilmente esas opciones al finalizar.

Vamos a crear el grupo de recursos.

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroup -Location EastUS
```

Actualmente, el grupo de recursos está vacío.

## <a name="role-based-access-control"></a>Control de acceso basado en rol

Desea asegurarse de que los usuarios de una organización tengan el nivel adecuado de acceso a estos recursos. No desea conceder acceso ilimitado a los usuarios, pero también es necesario garantizar que puedan realizar su trabajo. El [control de acceso basado en rol de Azure (Azure RBAC)](../../role-based-access-control/overview.md) le permite administrar los usuarios con permiso para completar acciones específicas en un ámbito.

Para crear y quitar las asignaciones de rol, los usuarios deben tener acceso a `Microsoft.Authorization/roleAssignments/*`. Esta acción se concede mediante los roles Propietario o Administrador de acceso de usuario.

Para administrar las soluciones de máquina virtual, hay tres roles específicos a los recursos, que normalmente proporcionan el acceso necesario:

* [Colaborador de la máquina virtual](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor)
* [Colaborador de la red](../../role-based-access-control/built-in-roles.md#network-contributor)
* [Colaborador de la cuenta de almacenamiento](../../role-based-access-control/built-in-roles.md#storage-account-contributor)

En lugar de asignar roles a usuarios individuales, a menudo resulta más fácil usar un grupo de Azure Active Directory con usuarios que tienen que realizar acciones similares. A continuación, asigne a ese grupo el rol apropiado. Para este artículo puede usar un grupo existente para administrar la máquina virtual o el portal para [crear un grupo de Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md).

Después de crear un grupo o de buscar uno existente, use el comando [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) para asignar el grupo de Azure Active Directory al rol Colaborador de la máquina virtual para el grupo de recursos.  

```azurepowershell-interactive
$adgroup = Get-AzADGroup -DisplayName <your-group-name>

New-AzRoleAssignment -ObjectId $adgroup.id `
  -ResourceGroupName myResourceGroup `
  -RoleDefinitionName "Virtual Machine Contributor"
```

Si recibe un error que indica **Principal \<guid> does not exist in the directory** (La entidad de seguridad no existe en el directorio), el nuevo grupo no se ha propagado en Azure Active Directory. Intente ejecutar el comando de nuevo.

Por lo general, repetirá el proceso para *Colaborador de la red* y *Colaborador de la cuenta de almacenamiento* con el objetivo de asegurarse de que los usuarios se asignan para administrar los recursos implementados. En este artículo, puede omitir esos pasos.

## <a name="azure-policy"></a>Azure Policy

[Azure Policy](../../governance/policy/overview.md) ayuda a asegurarse de que todos los recursos de la suscripción satisfacen los estándares corporativos. La suscripción ya tiene varias definiciones de directiva. Para ver las definiciones de directivas disponibles, use el comando [Get-AzPolicyDefinition](/powershell/module/az.resources/get-azpolicydefinition):

```azurepowershell-interactive
(Get-AzPolicyDefinition).Properties | Format-Table displayName, policyType
```

Ve las definiciones de directiva existentes. El tipo de directiva es **BuiltIn** o **Custom**. Examine las definiciones para buscar las que describan una condición que quiera asignar. En este artículo, puede asignar directivas que:

* Limite las ubicaciones para todos los recursos.
* Limite las SKU para las máquinas virtuales.
* Realice auditorías de las máquinas virtuales que no usen discos administrados.

En el siguiente ejemplo, se recuperan tres definiciones de directivas en función del nombre para mostrar. Puede usar el comando [New-AzPolicyAssignment](/powershell/module/az.resources/new-azpolicyassignment) para asignar esas definiciones al grupo de recursos. En algunas directivas, puede proporcionar los valores de los parámetros para especificar los valores permitidos.

```azurepowershell-interactive
# Values to use for parameters
$locations ="eastus", "eastus2"
$skus = "Standard_DS1_v2", "Standard_E2s_v2"

# Get the resource group
$rg = Get-AzResourceGroup -Name myResourceGroup

# Get policy definitions for allowed locations, allowed SKUs, and auditing VMs that don't use managed disks
$locationDefinition = Get-AzPolicyDefinition | where-object {$_.properties.displayname -eq "Allowed locations"}
$skuDefinition = Get-AzPolicyDefinition | where-object {$_.properties.displayname -eq "Allowed virtual machine size SKUs"}
$auditDefinition = Get-AzPolicyDefinition | where-object {$_.properties.displayname -eq "Audit VMs that do not use managed disks"}

# Assign policy for allowed locations
New-AzPolicyAssignment -Name "Set permitted locations" `
  -Scope $rg.ResourceId `
  -PolicyDefinition $locationDefinition `
  -listOfAllowedLocations $locations

# Assign policy for allowed SKUs
New-AzPolicyAssignment -Name "Set permitted VM SKUs" `
  -Scope $rg.ResourceId `
  -PolicyDefinition $skuDefinition `
  -listOfAllowedSKUs $skus

# Assign policy for auditing unmanaged disks
New-AzPolicyAssignment -Name "Audit unmanaged disks" `
  -Scope $rg.ResourceId `
  -PolicyDefinition $auditDefinition
```

## <a name="deploy-the-virtual-machine"></a>Implementación de la máquina virtual

Ha asignado roles y directivas, por lo que está listo para implementar la solución. El tamaño predeterminado es Standard_DS1_v2, que es una de las SKU permitidas. Cuando se realiza este paso, se le solicitan las credenciales. Los valores que especifique se configuran como el nombre de usuario y la contraseña de la máquina virtual.

```azurepowershell-interactive
New-AzVm -ResourceGroupName "myResourceGroup" `
     -Name "myVM" `
     -Location "East US" `
     -VirtualNetworkName "myVnet" `
     -SubnetName "mySubnet" `
     -SecurityGroupName "myNetworkSecurityGroup" `
     -PublicIpAddressName "myPublicIpAddress" `
     -OpenPorts 80,3389
```

Una vez finalizada la implementación, puede aplicar más opciones de configuración de administración a la solución.

## <a name="lock-resources"></a>Bloqueo de recursos

Los [bloqueos de recursos](../../azure-resource-manager/management/lock-resources.md) impiden que los usuarios de una organización eliminen o modifiquen de forma accidental recursos críticos. A diferencia del control de acceso basado en rol, los bloqueos de recursos aplican una restricción a todos los usuarios y roles. Puede establecer el bloqueo de nivel en *CanNotDelete* o *ReadOnly*.

Para bloquear la máquina virtual y el grupo de seguridad de red, use el comando [New-AzResourceLock](/powershell/module/az.resources/new-azresourcelock):

```azurepowershell-interactive
# Add CanNotDelete lock to the VM
New-AzResourceLock -LockLevel CanNotDelete `
  -LockName LockVM `
  -ResourceName myVM `
  -ResourceType Microsoft.Compute/virtualMachines `
  -ResourceGroupName myResourceGroup

# Add CanNotDelete lock to the network security group
New-AzResourceLock -LockLevel CanNotDelete `
  -LockName LockNSG `
  -ResourceName myNetworkSecurityGroup `
  -ResourceType Microsoft.Network/networkSecurityGroups `
  -ResourceGroupName myResourceGroup
```

Para probar los bloqueos, intente ejecutar el siguiente comando:

```azurepowershell-interactive 
Remove-AzResourceGroup -Name myResourceGroup
```

Verá un error en el que se indica que la operación de eliminación no se puede realizar debido a un bloqueo. Solo se puede eliminar el grupo de recursos si quita los bloqueos específicamente. Ese paso se muestra en [Limpieza de recursos](#clean-up-resources).

## <a name="tag-resources"></a>Etiquetado de recursos

Se aplican [etiquetas](../../azure-resource-manager/management/tag-resources.md) a los recursos de Azure para organizarlos de forma lógica por categorías. Cada etiqueta consta de un nombre y un valor. Por ejemplo, puede aplicar el nombre "Environment" y el valor "Production" a todos los recursos en producción.

[!INCLUDE [Resource Manager governance tags Powershell](../../../includes/resource-manager-governance-tags-powershell.md)]

Para aplicar etiquetas a una máquina virtual, use el comando [Set-AzResource](/powershell/module/az.resources/set-azresource):

```azurepowershell-interactive
# Get the virtual machine
$r = Get-AzResource -ResourceName myVM `
  -ResourceGroupName myResourceGroup `
  -ResourceType Microsoft.Compute/virtualMachines

# Apply tags to the virtual machine
Set-AzResource -Tag @{ Dept="IT"; Environment="Test"; Project="Documentation" } -ResourceId $r.ResourceId -Force
```

### <a name="find-resources-by-tag"></a>Búsqueda de recursos por etiqueta

Para buscar recursos con un valor y nombre de etiqueta, use el comando [Get-AzResource](/powershell/module/az.resources/get-azresource):

```azurepowershell-interactive
(Get-AzResource -Tag @{ Environment="Test"}).Name
```

Puede utilizar los valores devueltos para las tareas de administración, como detener todas las máquinas virtuales con un valor de etiqueta.

```azurepowershell-interactive
Get-AzResource -Tag @{ Environment="Test"} | Where-Object {$_.ResourceType -eq "Microsoft.Compute/virtualMachines"} | Stop-AzVM
```

### <a name="view-costs-by-tag-values"></a>Visualización de los costos por valores de etiqueta

[!INCLUDE [Resource Manager governance tags billing](../../../includes/resource-manager-governance-tags-billing.md)]

## <a name="clean-up-resources"></a>Limpieza de recursos

El grupo de seguridad de red bloqueado no se puede eliminar hasta que se quite el bloqueo. Para quitar el bloqueo, utilice el comando [Remove-AzResourceLock](/powershell/module/az.resources/remove-azresourcelock):

```azurepowershell-interactive
Remove-AzResourceLock -LockName LockVM `
  -ResourceName myVM `
  -ResourceType Microsoft.Compute/virtualMachines `
  -ResourceGroupName myResourceGroup
Remove-AzResourceLock -LockName LockNSG `
  -ResourceName myNetworkSecurityGroup `
  -ResourceType Microsoft.Network/networkSecurityGroups `
  -ResourceGroupName myResourceGroup
```

Cuando ya no se necesiten, puede usar el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) para quitar el grupo de recursos, la máquina virtual y todos los recursos relacionados.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="manage-costs"></a>Administrar costos

[!INCLUDE [cost-management-horizontal](../../../includes/cost-management-horizontal.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha creado una imagen de máquina virtual personalizada. Ha aprendido a:

> [!div class="checklist"]
> * Asignar un rol a los usuarios
> * Aplicar directivas que hagan cumplir los estándares
> * Proteger los recursos críticos con bloqueos
> * Etiquetar recursos para la facturación y la administración

Pase al siguiente tutorial para aprender a identificar los cambios y administrar las actualizaciones de paquetes en una máquina virtual Linux.

> [!div class="nextstepaction"]
> [Administración de máquinas virtuales](tutorial-config-management.md)
