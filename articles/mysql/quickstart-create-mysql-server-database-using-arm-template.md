---
title: 'Inicio rápido: Creación de una instancia de Azure DB for MySQL: plantilla de Resource Manager'
description: En este inicio rápido, aprenderá a crear un servidor de Azure Database for MySQL con la integración de red virtual mediante una plantilla de Azure Resource Manager.
author: mgblythe
ms.service: mysql
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: mblythe
ms.date: 05/19/2020
ms.openlocfilehash: 97a16bd81fd79646fb3693ec2dc4b8a40b9a6a3b
ms.sourcegitcommit: 53acd9895a4a395efa6d7cd41d7f78e392b9cfbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/22/2020
ms.locfileid: "90896159"
---
# <a name="quickstart-use-an-arm-template-to-create-an-azure-database-for-mysql-server"></a>Inicio rápido: Uso de una plantilla de Resource Manager para crear un servidor de Azure Database for MySQL

Azure Database for MySQL es un servicio administrado que se usa para ejecutar, administrar y escalar bases de datos MySQL de alta disponibilidad en la nube. En este inicio rápido se usa una plantilla de Azure Resource Manager para crear un servidor de Azure Database for MySQL con integración de red virtual. El servidor se puede crear en Azure Portal, la CLI de Azure o Azure PowerShell.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Implementación en Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-managed-mysql-with-vnet%2fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

# <a name="portal"></a>[Portal](#tab/azure-portal)

Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/).

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

* Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/).
* Si desea ejecutar el código localmente, [Azure PowerShell](/powershell/azure/).

# <a name="cli"></a>[CLI](#tab/CLI)

* Una cuenta de Azure con una suscripción activa. [cree una de forma gratuita](https://azure.microsoft.com/free/).
* Si desea ejecutar el código localmente, la [CLI de Azure](/cli/azure/).

---

## <a name="review-the-template"></a>Revisión de la plantilla

Cree un servidor de Azure Database for MySQL con un conjunto definido de recursos de almacenamiento y proceso. Para más información, consulte el artículo sobre los [planes de tarifa de Azure Database for MySQL](concepts-pricing-tiers.md). Cree el servidor dentro en un [grupo de recursos de Azure](../azure-resource-manager/management/overview.md).

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-managed-mysql-with-vnet/).

:::code language="json" source="~/quickstart-templates/101-managed-mysql-with-vnet/azuredeploy.json":::

La plantilla define cinco recursos de Azure:

* [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
* [**Microsoft.Network/virtualNetworks/subnets**](/azure/templates/microsoft.network/virtualnetworks/subnets)
* [**Microsoft.DBforMySQL/servers**](/azure/templates/microsoft.dbformysql/servers)
* [**Microsoft.DBforMySQL/servers/virtualNetworkRules**](/azure/templates/microsoft.dbformysql/servers/virtualnetworkrules)
* [**Microsoft.DBforMySQL/servers/firewallRules**](/azure/templates/microsoft.dbformysql/servers/firewallrules)

Encontrará más ejemplos de plantillas de Azure Database for MySQL en la [galería de plantillas de inicio rápido](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Dbformysql&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Implementación de la plantilla

# <a name="portal"></a>[Portal](#tab/azure-portal)

Seleccione el siguiente vínculo para implementar la plantilla del servidor de Azure Database for MySQL Server en Azure Portal:

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Implementación en Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-managed-mysql-with-vnet%2fazuredeploy.json)

En la página **Implementación de Azure Database for MySQL con red virtual**:

1. En **Grupo de recursos**, seleccione **Crear nuevo** y, después, especifique un nombre válido para el nuevo grupo de recursos y seleccione **Aceptar**.

2. Si ha creado un grupo de recursos, seleccione una **ubicación** para el grupo de recursos y el nuevo servidor.

3. Escriba un **nombre de servidor**, **Inicio de sesión del administrador** y **Contraseña de inicio de sesión del administrador**.

    :::image type="content" source="./media/quickstart-create-mysql-server-database-using-arm-template/deploy-azure-database-for-mysql-with-vnet.png" alt-text="Implementación de Azure Database for MySQL con la ventana de red virtual, plantilla de inicio rápido de Azure, Azure Portal":::

4. Cambie los valores predeterminados si lo desea:

    * **Subscription** (Suscripción): la suscripción de Azure que desea usar para el servidor.
    * **Sku Capacity** (Capacidad de SKU): la capacidad del núcleo virtual, que puede ser *2* (el valor predeterminado), *4*, *8*, *16*, *32* o *64*.
    * **Sku Name** (Nombre de SKU): el prefijo del nivel de SKU, la familia del SKU y la capacidad del SKU, unidos por guiones bajos, como *B_Gen5_1*, *GP_Gen5_2* (el valor predeterminado) o *MO_Gen5_32*.
    * **Sku Size MB** (Tamaño de SKU, en MB): el tamaño de almacenamiento, en megabytes, del servidor de Azure Database for MySQL (valor predeterminado, *5120*).
    * **Nivel de SKU**: el nivel de implementación, como *Basic*, *GeneralPurpose* (el valor predeterminado) o *MemoryOptimized*.
    * **Sku Family** (Familia de SKU): *Gen4* o *Gen5* (el valor predeterminado), que indica la generación de hardware de la implementación de servidores.
    * **Mysql Version** (Versión de MySQL): la versión del servidor de MySQL que se va a implementar, como *5.6* o *5.7* (el valor predeterminado).
    * **Backup Retention Days** (Días de retención de copia de seguridad): el período deseado para la conservación de copias de seguridad con redundancia geográfica, en días (el valor predeterminado es *7*).
    * **Geo Redundant Backup** (Copia de seguridad con redundancia geográfica): *Habilitado* o *Deshabilitado* (el valor predeterminado), en función de los requisitos de la recuperación ante desastres geográfica (Geo-DR).
    * **Virtual Network Name** (Nombre de red virtual): el nombre de la red virtual (el valor predeterminado es *azure_mysql_vnet*).
    * **Subnet Name** (Nombre de subred): el nombre de la subred (el valor predeterminado es *azure_mysql_subnet*).
    * **Virtual Network Rule Name** (Nombre de regla de red virtual): el nombre de la regla de red virtual que permite la subred (el valor predeterminado es *AllowSubnet*).
    * **Vnet Address Prefix** (Prefijo de dirección de red virtual): el prefijo de dirección de la red virtual (valor predeterminado, *10.0.0.0/16*).
    * **Subnet Prefix** (Prefijo de subred): el prefijo de dirección de la subred (valor predeterminado, *10.0.0.0/16*).

5. Consulte los términos y condiciones y seleccione **Acepto los términos y condiciones indicados anteriormente**.

6. Seleccione **Comprar**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Use el siguiente código interactivo para crear un nuevo servidor de Azure Database for MySQL mediante la plantilla. El código le pide el nombre del nuevo servidor, el nombre y la ubicación de un nuevo grupo de recursos, y un nombre y una contraseña de cuenta de administrador.

Para ejecutar el código en Azure Cloud Shell, seleccione **Probar** en la esquina superior de cualquier bloque de código.

```azurepowershell-interactive
$serverName = Read-Host -Prompt "Enter a name for the new Azure Database for MySQL server"
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group where the server will exist"
$location = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"
$adminUser = Read-Host -Prompt "Enter the Azure Database for MySQL server's administrator account name"
$adminPassword = Read-Host -Prompt "Enter the administrator password" -AsSecureString

New-AzResourceGroup -Name $resourceGroupName -Location $location # Use this command when you need to create a new resource group for your deployment
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-managed-mysql-with-vnet/azuredeploy.json `
    -serverName $serverName `
    -administratorLogin $adminUser `
    -administratorLoginPassword $adminPassword

Read-Host -Prompt "Press [ENTER] to continue ..."
```

# <a name="cli"></a>[CLI](#tab/CLI)

Use el siguiente código interactivo para crear un servidor de Azure Database for MySQL mediante la plantilla. El código le pide el nombre del nuevo servidor, el nombre y la ubicación de un nuevo grupo de recursos, y un nombre y una contraseña de cuenta de administrador.

Para ejecutar el código en Azure Cloud Shell, seleccione **Probar** en la esquina superior de cualquier bloque de código.

```azurecli-interactive
echo "Enter a name for the new Azure Database for MySQL server:" &&
read serverName &&
echo "Enter a name for the new resource group where the server will exist:" &&
read resourceGroupName &&
echo "Enter an Azure region (for example, centralus) for the resource group:" &&
read location &&
echo "Enter the Azure Database for MySQL server's administrator account name:" &&
read adminUser &&
echo "Enter the administrator password:" &&
read adminPassword &&
params='serverName='$serverName' administratorLogin='$adminUser' administratorLoginPassword='$adminPassword &&
az group create --name $resourceGroupName --location $location &&
az deployment group create --resource-group $resourceGroupName --parameters $params --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-managed-mysql-with-vnet/azuredeploy.json &&
echo "Press [ENTER] to continue ..."
```

---

## <a name="review-deployed-resources"></a>Revisión de los recursos implementados

# <a name="portal"></a>[Portal](#tab/azure-portal)

Para ver información general del nuevo servidor de Azure Database for MySQL, siga estos pasos:

1. En [Azure Portal](https://portal.azure.com), busque y seleccione los servidores de **Azure Database for MySQL**.

2. En la lista base de datos, seleccione el nuevo servidor. Aparece la página de **información general**  del nuevo servidor de Azure Database for MySQL.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Ejecute el siguiente código interactivo para ver detalles sobre el servidor de Azure Database for MySQL. Tendrá que escribir el nombre del nuevo servidor.

```azurepowershell-interactive
$serverName = Read-Host -Prompt "Enter the name of your Azure Database for MySQL server"
Get-AzResource -ResourceType "Microsoft.DBforMySQL/servers" -Name $serverName | ft
Write-Host "Press [ENTER] to continue..."
```

# <a name="cli"></a>[CLI](#tab/CLI)

Ejecute el siguiente código interactivo para ver detalles sobre su servidor de Azure Database for MySQL. Tendrá que escribir el nombre y el grupo de recursos del nuevo servidor.

```azurecli-interactive
echo "Enter your Azure Database for MySQL server name:" &&
read serverName &&
echo "Enter the resource group where the Azure Database for MySQL server exists:" &&
read resourcegroupName &&
az resource show --resource-group $resourcegroupName --name $serverName --resource-type "Microsoft.DbForMySQL/servers"
```

---

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, elimine el grupo de recursos, con lo que se eliminan los recursos que contiene.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com), busque la opción **Grupos de recursos** y selecciónela.

2. En la lista de los grupos de recursos, elija el nombre del grupo de recursos.

3. En la página de **información general** del grupo de recursos, seleccione **Eliminar grupo de recursos**.

4. En el cuadro de diálogo de confirmación, escriba el nombre del grupo de recursos y seleccione **Eliminar**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

# <a name="cli"></a>[CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

---

## <a name="next-steps"></a>Pasos siguientes

Para ver un tutorial paso a paso que le guíe en el proceso de creación de una plantilla de Resource Manager, consulte:

> [!div class="nextstepaction"]
> [ Tutorial: Creación e implementación de su primera plantilla de Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md)
