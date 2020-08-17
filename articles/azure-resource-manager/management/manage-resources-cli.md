---
title: 'Administración de recursos: CLI de Azure'
description: Uso de la CLI de Azure y de Azure Resource Manager para administrar los recursos. Muestra cómo implementar y eliminar los recursos.
author: mumian
ms.topic: conceptual
ms.date: 02/11/2019
ms.author: jgao
ms.openlocfilehash: edd424d7d64e484f3b42084d1aa0744ec522bf92
ms.sourcegitcommit: f353fe5acd9698aa31631f38dd32790d889b4dbb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/29/2020
ms.locfileid: "87366824"
---
# <a name="manage-azure-resources-by-using-azure-cli"></a>Administración de recursos de Azure mediante la CLI de Azure

Obtenga información sobre cómo utilizar la CLI de Azure con [Azure Resource Manager](overview.md) para administrar los recursos de Azure. Para la administración de grupos de recursos, consulte [Administrar recursos de Azure mediante la CLI de Azure](manage-resource-groups-cli.md).

Otros artículos sobre la administración de recursos:

- [Administración de recursos de Azure con Azure Portal](manage-resources-portal.md)
- [Administración de recursos de Azure con Azure PowerShell](manage-resources-powershell.md)

## <a name="deploy-resources-to-an-existing-resource-group"></a>Implementación de recursos en un grupo de recursos existente

Puede implementar los recursos de Azure mediante la CLI de Azure directamente o implementar una plantilla de Resource Manager para crear recursos de Azure.

### <a name="deploy-a-resource"></a>Implementación de un recurso

El script siguiente crea una cuenta de almacenamiento.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az storage account create --resource-group $resourceGroupName --name $storageAccountName --location $location --sku Standard_LRS --kind StorageV2 &&
az storage account show --resource-group $resourceGroupName --name $storageAccountName 
```

### <a name="deploy-a-template"></a>Implementación de una plantilla

El script siguiente implementa una plantilla de inicio rápido para crear una cuenta de almacenamiento. Para más información, consulte [Inicio rápido: Creación de plantillas de Azure Resource Manager mediante Visual Studio Code](../templates/quickstart-create-templates-use-visual-studio-code.md?tabs=PowerShell).

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
az group deployment create --resource-group $resourceGroupName --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
```

Para más información, consulte [Implementación de recursos con plantillas de Resource Manager y la CLI de Azure](../templates/deploy-cli.md).

## <a name="deploy-a-resource-group-and-resources"></a>Implementación de un grupo de recursos y de recursos

Puede crear un grupo de recursos e implementar recursos en el grupo. Para más información, consulte [Creación de un grupo de recursos e implementación de recursos](../templates/deploy-to-subscription.md#resource-groups).

## <a name="deploy-resources-to-multiple-subscriptions-or-resource-groups"></a>Implementación de recursos en varias suscripciones o grupos de recursos

Por lo general, todos los recursos de la plantilla se implementan en un único grupo de recursos. Sin embargo, existen escenarios en los que desea implementar un conjunto de recursos juntos pero colocarlos en distintos grupos de recursos o suscripciones. Para obtener más información, consulte [Implementación de recursos en Azure en varias suscripciones o grupos de recursos](../templates/cross-scope-deployment.md).

## <a name="delete-resources"></a>Eliminar recursos

El script siguiente muestra cómo eliminar una cuenta de almacenamiento.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az storage account delete --resource-group $resourceGroupName --name $storageAccountName 
```

Para obtener más información sobre cómo ordena Azure Resource Manager la eliminación de recursos, consulte [Eliminación del grupo de recursos en Azure Resource Manager](delete-resource-group.md).

## <a name="move-resources"></a>Traslado de recursos

El script siguiente muestra cómo quitar una cuenta de almacenamiento de un grupo de recursos a otro.

```azurecli-interactive
echo "Enter the source Resource Group name:" &&
read srcResourceGroupName &&
echo "Enter the destination Resource Group name:" &&
read destResourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
storageAccount=$(az resource show --resource-group $srcResourceGroupName --name $storageAccountName --resource-type Microsoft.Storage/storageAccounts --query id --output tsv) &&
az resource move --destination-group $destResourceGroupName --ids $storageAccount
```

Para obtener más información, consulte [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](move-resource-group-and-subscription.md).

## <a name="lock-resources"></a>Bloqueo de recursos

Los bloqueos impiden que otros usuarios de la organización eliminen o modifiquen por error recursos esenciales, como una suscripción de Azure, un grupo de recursos o un recurso. 

El script siguiente bloquea una cuenta de almacenamiento para que no se pueda eliminar.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az lock create --name LockSite --lock-type CanNotDelete --resource-group $resourceGroupName --resource-name $storageAccountName --resource-type Microsoft.Storage/storageAccounts 
```

El script siguiente obtiene todos los bloqueos de una cuenta de almacenamiento:

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az lock list --resource-group $resourceGroupName --resource-name $storageAccountName --resource-type Microsoft.Storage/storageAccounts --parent ""
```

El script siguiente elimina un bloqueo de una cuenta de almacenamiento:

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
lockId=$(az lock show --name LockSite --resource-group $resourceGroupName --resource-type Microsoft.Storage/storageAccounts --resource-name $storageAccountName --output tsv --query id)&&
az lock delete --ids $lockId
```

Para obtener más información, consulte [Bloqueo de recursos con el Administrador de recursos de Azure](lock-resources.md).

## <a name="tag-resources"></a>Etiquetado de recursos

El etiquetado ayuda a organizar lógicamente los grupos de recursos y los recursos. Para obtener información, vea [Uso de etiquetas para organizar los recursos de Azure](tag-resources.md#azure-cli).

## <a name="manage-access-to-resources"></a>Administración del acceso a los recursos

El [control de acceso basado en rol (RBAC)](../../role-based-access-control/overview.md) es la forma en que se administra el acceso a los recursos en Azure. Para obtener más información, consulte [Administración del acceso a los recursos de Azure mediante RBAC y la CLI de Azure](../../role-based-access-control/role-assignments-cli.md).

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información sobre Azure Resource Manager, consulte [Información general de Azure Resource Manager](overview.md).
- Para obtener información sobre la sintaxis de las plantillas de Resource Manager, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](../templates/template-syntax.md).
- Para obtener información sobre cómo desarrollar plantillas, consulte los [tutoriales paso a paso](../index.yml).
- Para ver los esquemas de plantilla de Azure Resource Manager, vea la [referencia de plantilla](/azure/templates/).
