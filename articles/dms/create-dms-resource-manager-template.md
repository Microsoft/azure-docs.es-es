---
title: Creación de una instancia de Database Migration Service (plantilla de Azure Resource Manager)
description: Aprenda a crear una instancia de Database Migration Service mediante una plantilla de Azure Resource Manager (plantilla de ARM).
author: MashaMSFT
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: mathoma
ms.date: 06/29/2020
ms.service: dms
ms.openlocfilehash: 24a0d16a6ff052df4b7a9bcdd078542987b4fd50
ms.sourcegitcommit: d18a59b2efff67934650f6ad3a2e1fe9f8269f21
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88661187"
---
# <a name="quickstart-create-instance-of-azure-database-migration-service-using-arm-template"></a>Inicio rápido: Creación de una instancia de Azure Database Migration Service con una plantilla de ARM

Use esta plantilla de Azure Resource Manager (plantilla de ARM) para implementar una instancia de Azure Database Migration Service. 

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-azure-database-migration-simple-deploy%2fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

La plantilla de Resource Manager de Azure Database Migration Service requiere: 

- La versión más reciente de la [CLI de Azure](/cli/azure/install-azure-cli) o [PowerShell](/powershell/scripting/install/installing-powershell). 
- Suscripción a Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-azure-database-migration-simple-deploy/).

:::code language="json" source="~/quickstart-templates/101-azure-database-migration-simple-deploy/azuredeploy.json":::

En la plantilla se definen tres recursos de Azure: 

- [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks): Crea la red virtual. 
- [Microsoft.Network/virtualNetworks/subnets](/azure/templates/microsoft.network/virtualnetworks/subnets): Crea la subred. 
- [Microsoft.DataMigration/services](/azure/templates/microsoft.datamigration/services): implementa una instancia de Azure Database Migration Service. 

Encontrará más plantillas de Azure Database Migration Service en la [galería de plantillas de inicio rápido](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Datamigration&pageNumber=1&sort=Popular).


## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla. La plantilla crea una instancia de Azure Database Migration Service. 

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-azure-database-migration-simple-deploy%2fazuredeploy.json)

2. Seleccione o escriba los siguientes valores.

    * **Suscripción**: Seleccione una suscripción de Azure.
    * **Grupo de recursos**: Seleccione un grupo de recursos existente en la lista desplegable o seleccione la opción **Crear nuevo** para crear un grupo de recursos. 
    * **Región**: la ubicación donde se implementarán los recursos.
    * **Nombre del servicio**: nombre del nuevo servicio de migración.
    * **Ubicación**: la ubicación predeterminada del grupo de recursos. Deje el valor predeterminado, `[resourceGroup().location]`.
    * **Nombre de la red virtual**: nombre de la nueva red virtual.
    * **Nombre de subred**: nombre de la nueva subred asociada a la red virtual.



3. Seleccione **Revisar + crear**. Una vez que la instancia de Azure Database Migration Service se haya implementado correctamente, se recibe una notificación. 


Azure Portal se usa para implementar la plantilla. Además de Azure Portal, también puede usar Azure PowerShell, la CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Puede usar la CLI de Azure para comprobar los recursos implementados. 


```azurecli-interactive
echo "Enter the resource group where your SQL Server VM exists:" &&
read resourcegroupName &&
az resource list --resource-group $resourcegroupName 
```


## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando no lo necesite, elimine el grupo de recursos mediante la CLI de Azure o Azure PowerShell:

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="next-steps"></a>Pasos siguientes

Para obtener un tutorial paso a paso que le guíe en el proceso de creación de una plantilla, consulte:

> [!div class="nextstepaction"]
> [ Tutorial: Creación e implementación de su primera plantilla de Resource Manager](/azure/azure-resource-manager/templates/template-tutorial-create-first-template)

Para conocer otras formas de implementar Azure Database Migration Service, consulte: 
- [Azure Portal](quickstart-create-data-migration-service-portal.md)

Para más información consulte la [introducción a Azure Database Migration Service](dms-overview.md)