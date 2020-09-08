---
title: 'Azure Resource Manager: Creación de una instancia administrada de Azure SQL'
description: Aprenda a crear una instancia de Azure SQL Managed Instance mediante una plantilla de Azure Resource Manager.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: subject-armqs
ms.devlang: ''
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: carlrab
ms.date: 06/22/2020
ms.openlocfilehash: d405cfda3c72ce9c724a7d96925d3346428a3089
ms.sourcegitcommit: d18a59b2efff67934650f6ad3a2e1fe9f8269f21
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88660621"
---
# <a name="quickstart-create-an-azure-sql-managed-instance-using-an-arm-template"></a>Inicio rápido: Creación de una instancia de Azure SQL Managed Instance mediante una plantilla de Resource Manager

Este inicio rápido se centra en el proceso de implementación de una plantilla de Azure Resource Manager para crear una instancia de Azure SQL Managed Instance y una red virtual. [Azure SQL Managed Instance](sql-managed-instance-paas-overview.md) es una base de datos en la nube inteligente, totalmente administrada y escalable, con casi un 100 % de paridad de características con el motor de base de datos de SQL Server.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-sqlmi-new-vnet%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/).

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-sqlmi-new-vnet/).

:::code language="json" source="~/quickstart-templates/101-sqlmi-new-vnet/azuredeploy.json":::

Estos recursos se definen en la plantilla:

- [**Microsoft.Network/networkSecurityGroups**](/azure/templates/microsoft.Network/networkSecurityGroups)
- [**Microsoft.Network/routeTables**](/azure/templates/microsoft.Network/routeTables)
- [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.Network/virtualNetworks)
- [**Microsoft.Sql/managedinstances**](/azure/templates/microsoft.sql/managedinstances)

Se pueden encontrar más plantillas de ejemplo en [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Sql&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Implementación de la plantilla

Seleccione **Pruébelo** en el bloque de código de PowerShell siguiente para abrir Azure Cloud Shell.

> [!IMPORTANT]
> La implementación de una instancia administrada es una operación de ejecución prolongada. La implementación de la primera instancia en la subred suele tardar mucho más que la implementación en una subred en la que ya existen instancias administradas. Para conocer los tiempos de aprovisionamiento medios, consulte el apartado sobre las [operaciones de administración de Instancia administrada de SQL](sql-managed-instance-paas-overview.md#management-operations).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"
$templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-sqlmi-new-vnet/azuredeploy.json"

$resourceGroupName = "${projectName}rg"

New-AzResourceGroup -Name $resourceGroupName -Location $location
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

Read-Host -Prompt "Press [ENTER] to continue ..."
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
read -p "Enter a project name that is used for generating resource names:" projectName &&
read -p "Enter the location (i.e. centralus):" location &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-sqlmi-new-vnet/azuredeploy.json" &&
resourceGroupName="${projectName}rg" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
echo "Press [ENTER] to continue ..." &&
read
```

---

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Visite [Azure Portal](https://portal.azure.com/#blade/HubsExtension/BrowseResourceGroups) y compruebe que la instancia administrada se encuentra en el grupo de recursos seleccionado. Dado que la creación de una instancia administrada puede tardar un tiempo, es posible que tenga que comprobar el vínculo **Implementaciones** de la página **Información general** del grupo de recursos.

- Para ver un inicio rápido que muestre cómo conectarse a SQL Managed Instance desde una máquina virtual de Azure, consulte el documento sobre la [configuración de la conexión a una máquina virtual de Azure](connect-vm-instance-configure.md).
- Para ver un inicio rápido que muestre cómo conectarse a SQL Managed Instance desde un equipo cliente local mediante una conexión de punto a sitio, consulte el artículo acerca de la [configuración de una conexión de punto a sitio](point-to-site-p2s-configure.md).

## <a name="clean-up-resources"></a>Limpieza de recursos

Conserve la instancia administrada si desea ir a [Pasos siguientes](#next-steps), pero elimine la instancia administrada y los recursos relacionados después de completar los tutoriales adicionales que desee. Después de eliminar una instancia administrada, consulte [Eliminación de una subred después de eliminar una instancia administrada](virtual-cluster-delete.md).


Para eliminar el grupo de recursos:

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName
```

---

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Configuración de una VM de Azure para la conexión a Azure SQL Managed Instance](connect-vm-instance-configure.md)
