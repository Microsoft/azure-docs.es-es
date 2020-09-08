---
title: Creación de una VM con SQL Server mediante una plantilla de ARM
description: Aprenda a crear un servidor de SQL Server en una máquina virtual de Azure (VM) mediante una plantilla de Azure Resource Manager (plantilla de ARM).
author: MashaMSFT
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: mathoma
ms.date: 06/29/2020
ms.service: virtual-machines-sql
ms.openlocfilehash: 21909a9c7a1edda5059b9e83cf8dda243862b966
ms.sourcegitcommit: d18a59b2efff67934650f6ad3a2e1fe9f8269f21
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2020
ms.locfileid: "88660214"
---
# <a name="quickstart-create-sql-server-vm-using-an-arm-template"></a>Inicio rápido: Creación de una VM con SQL Server mediante una plantilla de ARM

Use esta plantilla de Azure Resource Manager (plantilla de ARM) para implementar un servidor de SQL Server en una máquina virtual de Azure (VM). 

[!INCLUDE [About Azure Resource Manager](../../../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-sql-vm-new-storage%2fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

La plantilla de Resource Manager de la VM con SQL Server requiere lo siguiente:

- La versión más reciente de la [CLI de Azure](/cli/azure/install-azure-cli) o [PowerShell](/powershell/scripting/install/installing-powershell). 
- Un [grupo de recursos](../../../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups) configurado previamente con una [red virtual](../../../virtual-network/quick-create-portal.md) y una [subred](../../../virtual-network/virtual-network-manage-subnet.md#add-a-subnet) preparadas.
- Suscripción a Azure. Si no tiene una, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.


## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/101-sql-vm-new-storage/).

:::code language="json" source="~/quickstart-templates/101-sql-vm-new-storage/azuredeploy.json":::

En la plantilla se definen cinco recursos de Azure: 

- [Microsoft.Network/publicIpAddresses](/azure/templates/microsoft.network/publicipaddresses): Crea una dirección IP pública. 
- [Microsoft.Network/networkSecurityGroups](/azure/templates/microsoft.network/networksecuritygroups): Crea un grupo de seguridad de red. 
- [Microsoft.Network/networkInterfaces](/azure/templates/microsoft.network/networkinterfaces): Configura la interfaz de red. 
- [Microsoft.Compute/virtualMachines](/azure/templates/microsoft.compute/virtualmachines): Crea una máquina virtual en Azure. 
- [Microsoft.SqlVirtualMachine/SqlVirtualMachines](/azure/templates/microsoft.sqlvirtualmachine/sqlvirtualmachines): registra la máquina virtual con el proveedor de recursos de la máquina virtual con SQL. 

Se pueden encontrar más plantillas de SQL Server en máquinas virtuales de Azure en la [galería de plantillas de inicio rápido](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Sqlvirtualmachine&pageNumber=1&sort=Popular).


## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla. La plantilla crea una máquina virtual con la versión de SQL Server prevista instalada en ella y registrada con el proveedor de recursos de la máquina virtual de SQL. 

   [![Implementación en Azure](../../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-sql-vm-new-storage%2fazuredeploy.json)

2. Seleccione o escriba los siguientes valores.

    * **Suscripción**: Seleccione una suscripción de Azure.
    * **Grupo de recursos**: el grupo de recursos preparado para la VM con SQL Server. 
    * **Región**: Seleccione una región.  Por ejemplo, **Centro de EE. UU**.
    * **Nombre de la máquina virtual**: escriba un nombre para la máquina virtual con SQL Server. 
    * **Tamaño de la máquina virtual**: elija el tamaño adecuado de la máquina virtual en el menú desplegable.
    * **Nombre de la red virtual existente**: escriba el nombre de la red virtual preparada para la VM con SQL Server. 
    * **Grupo de recursos de la red virtual existente**: escriba el grupo de recursos en el que se preparó la red virtual. 
    * **Nombre de subred existente**: el nombre de la subred preparada. 
    * **Oferta de la imagen**: Elija la imagen de SQL Server y de Windows Server que mejor se adapte a sus necesidades empresariales. 
    * **SKU de SQL**: elija la edición de la SKU de SQL Server que mejor se adapte a sus necesidades empresariales. 
    * **Nombre de usuario administrador**: el nombre de usuario del administrador de la máquina virtual. 
    * **Contraseña de administrador**: la contraseña que utiliza el administrador de la cuenta de la máquina virtual. 
    * **Tipo de almacenamiento de la carga de trabajo**:  el tipo de almacenamiento de la carga de trabajo que mejor se adapte a su negocio. 
    * **Número de discos de datos de SQL**:  el número de discos que SQL Server usa para los archivos de datos.  
    * **Ruta de acceso a datos**:  la ruta de acceso a los archivos de datos de SQL Server. 
    * **Número de discos de registro de SQL**:  el número de discos que SQL Server usa para los archivos de registro. 
    * **Ruta de acceso al registro**:  la ruta de acceso a los archivos de registro de SQL Server. 
    * **Ubicación**:  la ubicación de todos los recursos, este valor debe dejarse en su valor predeterminado `[resourceGroup().location]`. 

3. Seleccione **Revisar + crear**. Una vez que se haya implementado la VM con SQL Server correctamente, recibirá una notificación.

Azure Portal se usa para implementar la plantilla. Además de Azure Portal, también puede usar Azure PowerShell, la CLI de Azure y API REST. Para obtener información sobre otros métodos de implementación, consulte [Implementación de plantillas](../../../azure-resource-manager/templates/deploy-powershell.md).

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

Para ver otras formas de implementar una VM con SQL Server, consulte: 
- [Azure Portal](create-sql-vm-portal.md)
- [PowerShell](create-sql-vm-powershell.md)

Para más información, consulte [una introducción sobre SQL Server en máquinas virtuales de Azure](sql-server-on-azure-vm-iaas-what-is-overview.md).
