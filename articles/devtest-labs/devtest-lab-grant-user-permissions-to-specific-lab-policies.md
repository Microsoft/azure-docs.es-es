---
title: Concesión de permisos de usuario a directivas específicas de laboratorio | Microsoft Docs
description: Obtenga información acerca de cómo conceder permisos de usuario para las directivas específicas de laboratorio en DevTest Labs según las necesidades de cada usuario
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 8e910a5d4499d104e4b09076ec7862ae96272ef4
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87835687"
---
# <a name="grant-user-permissions-to-specific-lab-policies"></a>Concesión de permisos de usuario a directivas específicas de laboratorio
## <a name="overview"></a>Información general
Este artículo muestra cómo usar PowerShell para conceder permisos de usuario a una directiva concreta de laboratorio. De este modo, se pueden aplicar permisos en función de las necesidades de cada usuario. Por ejemplo, quizá prefiera conceder a un usuario determinado la posibilidad de cambiar la configuración de directiva de máquina virtual, pero no las directivas de costos.

## <a name="policies-as-resources"></a>Directivas como recursos
Tal y como se describe en el artículo [Control de acceso basado en rol de Azure (RBAC)](../role-based-access-control/role-assignments-portal.md), RBAC permite la administración pormenorizada del acceso a los recursos de Azure. Gracias a RBAC puede dividir las tareas entre el equipo de DevOps, y conceder a los usuarios únicamente el nivel de acceso que necesitan para realizar su trabajo.

En DevTest Labs, una directiva es un tipo de recurso que permite la acción de RBAC **Microsoft.DevTestLab/labs/policySets/policies/** . Cada directiva de laboratorio es un recurso en el tipo de recurso Directiva, y se puede asignar como ámbito a un rol de Azure.

Por ejemplo, para conceder permisos de lectura y escritura a los usuarios para la directiva **Tamaños permitidos de máquina virtual**, debe crear un rol personalizado que funcione con la acción **Microsoft.DevTestLab/labs/policySets/policies/** y, después, asignar los usuarios adecuados a este rol personalizado en el ámbito de **Microsoft.DevTestLab/labs/policySets/policies/AllowedVmSizesInLab**.

Para obtener más información sobre los roles personalizados de RBAC, consulte [Control de acceso de los roles personalizados](../role-based-access-control/custom-roles.md).

## <a name="creating-a-lab-custom-role-using-powershell"></a>Creación de un rol personalizado de laboratorio con PowerShell
Para empezar, deberá [instalar Azure PowerShell](/powershell/azure/install-az-ps). 

Una vez haya configurado los cmdlets de Azure PowerShell, podrá realizar las siguientes tareas:

* Enumerar todas las operaciones y acciones para un proveedor de recursos
* Enumerar las acciones de un rol determinado
* Crear un rol personalizado

El siguiente script de PowerShell muestra ejemplos de cómo realizar estas tareas:

```azurepowershell
# List all the operations/actions for a resource provider.
Get-AzProviderOperation -OperationSearchString "Microsoft.DevTestLab/*"

# List actions in a particular role.
(Get-AzRoleDefinition "DevTest Labs User").Actions

# Create custom role.
$policyRoleDef = (Get-AzRoleDefinition "DevTest Labs User")
$policyRoleDef.Id = $null
$policyRoleDef.Name = "Policy Contributor"
$policyRoleDef.IsCustom = $true
$policyRoleDef.AssignableScopes.Clear()
$policyRoleDef.AssignableScopes.Add("/subscriptions/<SubscriptionID> ")
$policyRoleDef.Actions.Add("Microsoft.DevTestLab/labs/policySets/policies/*")
$policyRoleDef = (New-AzRoleDefinition -Role $policyRoleDef)
```

## <a name="assigning-permissions-to-a-user-for-a-specific-policy-using-custom-roles"></a>Asignación de permisos a un usuario para una directiva determinada utilizando roles personalizados
Una vez haya definido los roles personalizados, puede asignárselos a los usuarios. A fin de asignar un rol personalizado a un usuario, primero debe obtener el **ObjectId** que representa a ese usuario. Para ello, use el cmdlet **Get-AzADUser**.

En el ejemplo siguiente, el valor **ObjectId** del usuario *SomeUser* es 05DEFF7B-0AC3-4ABF-B74D-6A72CD5BF3F3.

```azurepowershell
PS C:\>Get-AzADUser -SearchString "SomeUser"

DisplayName                    Type                           ObjectId
-----------                    ----                           --------
someuser@hotmail.com                                          05DEFF7B-0AC3-4ABF-B74D-6A72CD5BF3F3
```

Una vez que tenga el **ObjectId** para el usuario y un nombre de rol personalizado, puede asignar ese rol al usuario con el cmdlet **New-AzRoleAssignment**:

```azurepowershell
PS C:\>New-AzRoleAssignment -ObjectId 05DEFF7B-0AC3-4ABF-B74D-6A72CD5BF3F3 -RoleDefinitionName "Policy Contributor" -Scope /subscriptions/<SubscriptionID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.DevTestLab/labs/<LabName>/policySets/default/policies/AllowedVmSizesInLab
```

En el ejemplo anterior, se utiliza la directiva **AllowedVmSizesInLab** . Puede utilizar cualquiera de las directivas siguientes:

* MaxVmsAllowedPerUser
* MaxVmsAllowedPerLab
* AllowedVmSizesInLab
* LabVmsShutdown

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## <a name="next-steps"></a>Pasos siguientes
Una vez que haya concedido permisos de usuario a las directivas específicas del laboratorio, estos son algunos de los siguientes pasos que debe considerar:

* [Acceso seguro a un laboratorio](devtest-lab-add-devtest-user.md)
* [Definición de directivas de laboratorio](devtest-lab-set-lab-policy.md)
* [Creación de una plantilla de laboratorio](devtest-lab-create-template.md)
* [Creación de artefactos personalizados para máquinas virtuales](devtest-lab-artifact-author.md)
* [Adición de una máquina virtual a un laboratorio](devtest-lab-add-vm.md)
