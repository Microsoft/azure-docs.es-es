---
title: 'Administración de recursos: Azure PowerShell'
description: Use Azure PowerShell y de Azure Resource Manager para administrar los recursos. Muestra cómo implementar y eliminar los recursos.
author: mumian
ms.topic: conceptual
ms.date: 02/11/2019
ms.author: jgao
ms.openlocfilehash: 10f8c2a17c161d7e5622636b08b524932defe451
ms.sourcegitcommit: 2ff0d073607bc746ffc638a84bb026d1705e543e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/06/2020
ms.locfileid: "87826881"
---
# <a name="manage-azure-resources-by-using-azure-powershell"></a>Administración de recursos de Azure con Azure PowerShell

Obtenga información sobre cómo utilizar Azure PowerShell con [Azure Resource Manager](overview.md) para administrar los recursos de Azure. Para la administración de grupos de recursos, consulte [Administración de grupos de recursos de Azure con Azure PowerShell](manage-resource-groups-powershell.md).

Otros artículos sobre la administración de recursos:

- [Administración de recursos de Azure con Azure Portal](manage-resources-portal.md)
- [Administración de grupos de recursos de Azure mediante la CLI de Azure](manage-resources-cli.md)

## <a name="deploy-resources-to-an-existing-resource-group"></a>Implementación de recursos en un grupo de recursos existente

Puede implementar los recursos de Azure mediante Azure PowerShell directamente o implementar una plantilla de Resource Manager para crear recursos de Azure.

### <a name="deploy-a-resource"></a>Implementación de un recurso

El script siguiente crea una cuenta de almacenamiento.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

# Create the storage account.
$storageAccount = New-AzStorageAccount -ResourceGroupName $resourceGroupName `
  -Name $storageAccountName `
  -Location $location `
  -SkuName "Standard_LRS"

# Retrieve the context.
$ctx = $storageAccount.Context
```

### <a name="deploy-a-template"></a>Implementación de una plantilla

El script siguiente implementa una plantilla de inicio rápido para crear una cuenta de almacenamiento. Para más información, consulte [Inicio rápido: Creación de plantillas de Azure Resource Manager mediante Visual Studio Code](../templates/quickstart-create-templates-use-visual-studio-code.md?tabs=PowerShell).

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -Location $location
```

Para más información, consulte [Implementación de recursos con plantillas de Resource Manager y Azure PowerShell](../templates/deploy-powershell.md).

## <a name="deploy-a-resource-group-and-resources"></a>Implementación de un grupo de recursos y de recursos

Puede crear un grupo de recursos e implementar recursos en el grupo. Para más información, consulte [Creación de un grupo de recursos e implementación de recursos](../templates/deploy-to-subscription.md#resource-groups).

## <a name="deploy-resources-to-multiple-subscriptions-or-resource-groups"></a>Implementación de recursos en varias suscripciones o grupos de recursos

Por lo general, todos los recursos de la plantilla se implementan en un único grupo de recursos. Sin embargo, existen escenarios en los que desea implementar un conjunto de recursos juntos pero colocarlos en distintos grupos de recursos o suscripciones. Para obtener más información, consulte [Implementación de recursos en Azure en varias suscripciones o grupos de recursos](../templates/cross-scope-deployment.md).

## <a name="delete-resources"></a>Eliminar recursos

El script siguiente muestra cómo eliminar una cuenta de almacenamiento.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

Remove-AzStorageAccount -ResourceGroupName $resourceGroupName -AccountName $storageAccountName
```

Para obtener más información sobre cómo ordena Azure Resource Manager la eliminación de recursos, consulte [Eliminación del grupo de recursos en Azure Resource Manager](delete-resource-group.md).

## <a name="move-resources"></a>Traslado de recursos

El script siguiente muestra cómo quitar una cuenta de almacenamiento de un grupo de recursos a otro.

```azurepowershell-interactive
$srcResourceGroupName = Read-Host -Prompt "Enter the source Resource Group name"
$destResourceGroupName = Read-Host -Prompt "Enter the destination Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

$storageAccount = Get-AzResource -ResourceGroupName $srcResourceGroupName -ResourceName $storageAccountName
Move-AzResource -DestinationResourceGroupName $destResourceGroupName -ResourceId $storageAccount.ResourceId
```

Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](move-resource-group-and-subscription.md).

## <a name="lock-resources"></a>Bloqueo de recursos

Los bloqueos impiden que otros usuarios de la organización eliminen o modifiquen por error recursos esenciales, como una suscripción de Azure, un grupo de recursos o un recurso. 

El script siguiente bloquea una cuenta de almacenamiento para que no se pueda eliminar.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

New-AzResourceLock -LockName LockStorage -LockLevel CanNotDelete -ResourceGroupName $resourceGroupName -ResourceName $storageAccountName -ResourceType Microsoft.Storage/storageAccounts 
```

El script siguiente obtiene todos los bloqueos de una cuenta de almacenamiento:

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

Get-AzResourceLock -ResourceGroupName $resourceGroupName -ResourceName $storageAccountName -ResourceType Microsoft.Storage/storageAccounts
```

El script siguiente elimina un bloqueo de una cuenta de almacenamiento:

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"

$lockId = (Get-AzResourceLock -ResourceGroupName $resourceGroupName -ResourceName $storageAccountName -ResourceType Microsoft.Storage/storageAccounts).LockId
Remove-AzResourceLock -LockId $lockId
```

Para obtener más información, consulte [Bloqueo de recursos con el Administrador de recursos de Azure](lock-resources.md).

## <a name="tag-resources"></a>Etiquetado de recursos

El etiquetado ayuda a organizar lógicamente los grupos de recursos y los recursos. Para obtener información, vea [Uso de etiquetas para organizar los recursos de Azure](tag-resources.md#powershell).

## <a name="manage-access-to-resources"></a>Administración del acceso a los recursos

El [control de acceso basado en rol (RBAC) de Azure](../../role-based-access-control/overview.md) es la forma en la que se administra el acceso a los recursos de Azure. Para obtener más información, consulte [Administración del acceso mediante RBAC y Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre Azure Resource Manager, consulte [Información general de Azure Resource Manager](overview.md).
- Para obtener información sobre la sintaxis de las plantillas de Resource Manager, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](../templates/template-syntax.md).
- Para obtener información sobre cómo desarrollar plantillas, consulte los [tutoriales paso a paso](../index.yml).
- Para ver los esquemas de plantilla de Azure Resource Manager, vea la [referencia de plantilla](/azure/templates/).
