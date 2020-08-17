---
title: Crear una cuenta de almacenamiento
titleSuffix: Azure Storage
description: Obtenga información para crear una nueva cuenta de almacenamiento con Azure Portal, Azure PowerShell o la CLI de Azure. Una cuenta de Azure Storage proporciona un espacio de nombres único en Microsoft Azure para almacenar y tener acceso a los datos.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 02/07/2020
ms.author: tamram
ms.subservice: common
ms.custom: devx-track-azurecli
ms.openlocfilehash: 636efe27ec82fa870267d2cf69d5cdf990fd509a
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87501362"
---
# <a name="create-an-azure-storage-account"></a>Creación de una cuenta de Azure Storage

Una cuenta de Azure Storage contiene todos los objetos de datos de Azure Storage: blobs, archivos, colas, tablas y discos. La cuenta de almacenamiento proporciona un espacio de nombres único para los datos de Azure Storage que es accesible desde cualquier lugar del mundo a través de HTTP o HTTPS. Los datos de la cuenta de Azure Storage son duraderos y altamente disponibles, seguros y escalables a gran escala.

En este artículo de procedimientos se aprende a crear una cuenta de almacenamiento con [Azure Portal](https://portal.azure.com/), [Azure PowerShell](https://docs.microsoft.com/powershell/azure/), la [CLI de Azure](https://docs.microsoft.com/cli/azure?view=azure-cli-latest) o una [plantilla de Azure Resource Manager](../../azure-resource-manager/management/overview.md).  

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Ninguno.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para crear una cuenta de almacenamiento de Azure con PowerShell, asegúrese de que ha instalado la versión 0.7 o posterior del módulo Az de Azure PowerShell. Para obtener más información, consulte [Presentación del módulo Az de Azure PowerShell](/powershell/azure/new-azureps-module-az).

Para saber cuál es su versión actual, ejecute el comando siguiente:

```powershell
Get-InstalledModule -Name "Az"
```

Para instalar o actualizar Azure PowerShell, consulte [Instalación del módulo Azure PowerShell](/powershell/azure/install-Az-ps).

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Puede iniciar sesión en Azure y ejecutar los comandos de la CLI de Azure de dos maneras:

- Puede ejecutar los comandos de la CLI desde Azure Portal, en Azure Cloud Shell.
- Puede instalar la CLI y ejecutar sus comandos en local.

### <a name="use-azure-cloud-shell"></a>Uso de Azure Cloud Shell

Azure Cloud Shell es un shell de Bash gratuito que puede ejecutarse directamente en Azure Portal. La CLI de Azure está preinstalada y configurada para su uso con la cuenta. Haga clic en el botón **Cloud Shell** del menú situado en la sección superior derecha de Azure Portal:

[![Cloud Shell](./media/storage-quickstart-create-account/cloud-shell-menu.png)](https://portal.azure.com)

El botón inicia un shell interactivo que se puede usar para ejecutar los pasos de este artículo de procedimientos:

[![Captura de pantalla en la que aparece la ventana de Cloud Shell del portal](./media/storage-quickstart-create-account/cloud-shell.png)](https://portal.azure.com)

### <a name="install-the-cli-locally"></a>Instalación local de la CLI

También puede instalar y usar la CLI de Azure localmente. Para este artículo de procedimientos es preciso usar la versión 2.0.4 de la CLI de Azure o cualquier versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 

# <a name="template"></a>[Plantilla](#tab/template)

Ninguno.

---

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

# <a name="portal"></a>[Portal](#tab/azure-portal)

Inicie sesión en [Azure Portal](https://portal.azure.com).

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Inicie sesión en la suscripción de Azure con el comando `Connect-AzAccount` y siga las instrucciones que aparecen en pantalla para autenticarse.

```powershell
Connect-AzAccount
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para iniciar Azure Cloud Shell, inicie sesión en [Azure Portal](https://portal.azure.com).

Para iniciar sesión en la instalación local de la CLI, ejecute el comando [az login](/cli/azure/reference-index#az-login):

```azurecli-interactive
az login
```

# <a name="template"></a>[Plantilla](#tab/template)

N/D

---

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Ahora ya está listo para crear una cuenta de almacenamiento.

Cada cuenta de almacenamiento debe pertenecer a un grupo de recursos de Azure. Un grupo de recursos es un contenedor lógico para agrupar servicios de Azure. Al crear una cuenta de almacenamiento, puede elegir entre crear un grupo de recursos o usar uno existente. En este artículo se muestra cómo crear un nuevo grupo de recursos.

Una cuenta de almacenamiento de **uso general v2** proporciona acceso a todos los servicios de Azure Storage: Blob, File, Queue, Table y Disk. Los pasos indicados aquí crean una cuenta de almacenamiento de uso general v2, pero los pasos para crear cualquier otro tipo de cuenta de almacenamiento son similares.

# <a name="portal"></a>[Portal](#tab/azure-portal)

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

En primer lugar, cree un grupo de recursos con PowerShell mediante el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

```powershell
# put resource group in a variable so you can use the same group name going forward,
# without hard-coding it repeatedly
$resourceGroup = "storage-resource-group"
$location = "westus"
New-AzResourceGroup -Name $resourceGroup -Location $location
```

Si no está seguro de qué región especificar para el parámetro `-Location`, puede recuperar una lista de regiones admitidas para la suscripción con el comando [Get-AzLocation](/powershell/module/az.resources/get-azlocation):

```powershell
Get-AzLocation | select Location
```

Luego, para crear una cuenta de almacenamiento de uso general v2 con almacenamiento con redundancia local con acceso de lectura (RA-GRS), use el comando [New-AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount). Recuerde que el nombre de la cuenta de almacenamiento debe ser único en Azure, así que reemplace el valor de marcador de posición entre corchetes por su propio valor único:

```powershell
New-AzStorageAccount -ResourceGroupName $resourceGroup `
  -Name <account-name> `
  -Location $location `
  -SkuName Standard_RAGRS `
  -Kind StorageV2
```

> [!IMPORTANT]
> Si planea usar [Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage/), incluya `-EnableHierarchicalNamespace $True` en esta lista de parámetros.

Para crear una cuenta de almacenamiento de uso general v2 con otra opción de replicación, sustituya el valor deseado de la tabla siguiente por el parámetro **SkuName**.

|Opción Replicación  |Parámetro SkuName  |
|---------|---------|
|Almacenamiento con redundancia local (LRS)     |Standard_LRS         |
|Almacenamiento con redundancia de zona (ZRS)     |Standard_ZRS         |
|Almacenamiento con redundancia geográfica (GRS)     |Standard_GRS         |
|Almacenamiento con redundancia geográfica con acceso de lectura (GRS)     |Standard_RAGRS         |
|Almacenamiento con redundancia de zona geográfica (GZRS)    |Standard_GZRS         |
|Almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS)    |Standard_RAGZRS         |

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Primero, cree un grupo de recursos con la CLI de Azure mediante el comando [az group create](/cli/azure/group#az_group_create).

```azurecli-interactive
az group create \
    --name storage-resource-group \
    --location westus
```

Si no está seguro de qué región especificar para el parámetro `--location`, puede recuperar una lista de regiones admitidas para la suscripción con el comando [az account list-locations](/cli/azure/account#az_account_list).

```azurecli-interactive
az account list-locations \
    --query "[].{Region:name}" \
    --out table
```

Luego, para crear una cuenta de almacenamiento de uso general v2 con almacenamiento con redundancia local con acceso de lectura (RA-GRS), use el comando [az storage account create](/cli/azure/storage/account#az_storage_account_create). Recuerde que el nombre de la cuenta de almacenamiento debe ser único en Azure, así que reemplace el valor de marcador de posición entre corchetes por su propio valor único:

```azurecli-interactive
az storage account create \
    --name <account-name> \
    --resource-group storage-resource-group \
    --location westus \
    --sku Standard_RAGRS \
    --kind StorageV2
```

> [!IMPORTANT]
> Si planea usar [Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage/), incluya `--enable-hierarchical-namespace true` en esta lista de parámetros. 

Para crear una cuenta de almacenamiento de uso general v2 con otra opción de replicación, sustituya el valor deseado de la tabla siguiente por el parámetro **sku**.

|Opción Replicación  |Parámetro sku  |
|---------|---------|
|Almacenamiento con redundancia local (LRS)     |Standard_LRS         |
|Almacenamiento con redundancia de zona (ZRS)     |Standard_ZRS         |
|Almacenamiento con redundancia geográfica (GRS)     |Standard_GRS         |
|Almacenamiento con redundancia geográfica con acceso de lectura (GRS)     |Standard_RAGRS         |
|Almacenamiento con redundancia de zona geográfica (GZRS)    |Standard_GZRS         |
|Almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS)    |Standard_RAGZRS         |

# <a name="template"></a>[Plantilla](#tab/template)

Puede usar Azure PowerShell o la CLI de Azure para implementar una plantilla de Resource Manager y crear así una cuenta de almacenamiento. La plantilla empleada en este artículo de procedimientos proviene de [Plantillas de inicio rápido de Azure Resource Manager](https://azure.microsoft.com/resources/templates/101-storage-account-create/). Para ejecutar los scripts, seleccione **Pruébelo** para abrir Azure Cloud Shell. Para pegar el script, haga clic con el botón derecho en el shell y, a continuación, seleccione **Pegar**.

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
$location = Read-Host -Prompt "Enter the location (i.e. centralus)"

New-AzResourceGroup -Name $resourceGroupName -Location "$location"
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
```

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the location (i.e. centralus):" &&
read location &&
az group create --name $resourceGroupName --location "$location" &&
az group deployment create --resource-group $resourceGroupName --template-file "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json"
```

> [!NOTE]
> Esta plantilla sirve solo como ejemplo. Hay muchas opciones de una cuenta de almacenamiento que no se configuran como parte de esta plantilla. Por ejemplo, si desea utilizar [Azure Data Lake Storage](https://azure.microsoft.com/services/storage/data-lake-storage/), debe modificar esta plantilla estableciendo la propiedad `isHnsEnabledad` del objeto `StorageAccountPropertiesCreateParameters` en `true`. 

Para obtener información sobre cómo modificar esta plantilla o crear otras nuevas, consulte:

- [Documentación de Azure Resource Manager](/azure/azure-resource-manager/).
- [Referencia de plantilla de cuenta de almacenamiento](/azure/templates/microsoft.storage/allversions).
- [Ejemplos de plantillas de cuenta de almacenamiento adicionales](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Storage).

---

Para más información acerca de las opciones de replicación, consulte [Opciones de replicación de Azure Storage](storage-redundancy.md).

## <a name="delete-a-storage-account"></a>Eliminar una cuenta de almacenamiento

Si se elimina la cuenta de almacenamiento, se elimina toda la cuenta, incluidos todos los datos de la cuenta. Esta acción no se puede deshacer.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Navegue hasta cuenta de almacenamiento en [Azure Portal](https://portal.azure.com).
1. Haga clic en **Eliminar**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para eliminar la cuenta de almacenamiento, utilice el comando [Remove-AzStorageAccount](/powershell/module/az.storage/remove-azstorageaccount):

```powershell
Remove-AzStorageAccount -Name <storage-account> -ResourceGroupName <resource-group>
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para eliminar la cuenta de almacenamiento, utilice el comando [az storage account delete](/cli/azure/storage/account#az-storage-account-delete):

```azurecli-interactive
az storage account delete --name <storage-account> --resource-group <resource-group>
```

# <a name="template"></a>[Plantilla](#tab/template)

Para eliminar la cuenta de almacenamiento, use Azure PowerShell o la CLI de Azure.

```azurepowershell-interactive
$storageResourceGroupName = Read-Host -Prompt "Enter the resource group name"
$storageAccountName = Read-Host -Prompt "Enter the storage account name"
Remove-AzStorageAccount -Name $storageAccountName -ResourceGroupName $storageResourceGroupName
```

```azurecli-interactive
echo "Enter the resource group name:" &&
read resourceGroupName &&
echo "Enter the storage account name:" &&
read storageAccountName &&
az storage account delete --name storageAccountName --resource-group resourceGroupName
```

---

Como alternativa, puede eliminar el grupo de recursos, que elimina la cuenta de almacenamiento y otros recursos de ese grupo de recursos. Para obtener más información sobre cómo eliminar un grupo de recursos, consulte [Eliminación del grupo de recursos y los recursos](../../azure-resource-manager/management/delete-resource-group.md).

> [!WARNING]
> No es posible restaurar una cuenta de almacenamiento eliminada ni recuperar el contenido que contenía antes de la eliminación. Asegúrese de hacer una copia de seguridad de cualquier contenido que desee guardar antes de eliminar la cuenta. Esto también es verdad para los recursos de la cuenta: cuando se elimina un blob, tabla, cola o archivo, este se eliminará definitivamente.
>
> Si intenta eliminar una cuenta de almacenamiento asociada a una máquina virtual de Azure, puede obtener un error sobre que la cuenta de almacenamiento aún está usándose. Para obtener ayuda para solucionar este error, consulte [Solución de errores al eliminar las cuentas de almacenamiento](../common/storage-resource-manager-cannot-delete-storage-account-container-vhd.md).

## <a name="next-steps"></a>Pasos siguientes

En este artículo de procedimientos se ha creado una cuenta de almacenamiento estándar de uso general v2. Para aprender a cargar y descargar blobs en la cuenta de almacenamiento, vaya a alguna de las guías de inicio rápido de Blob Storage.

# <a name="portal"></a>[Portal](#tab/azure-portal)

> [!div class="nextstepaction"]
> [Uso de blobs con Azure Portal](../blobs/storage-quickstart-blobs-portal.md)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

> [!div class="nextstepaction"]
> [Uso de blobs con PowerShell](../blobs/storage-quickstart-blobs-powershell.md)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

> [!div class="nextstepaction"]
> [Uso de blobs con la CLI de Azure](../blobs/storage-quickstart-blobs-cli.md)

# <a name="template"></a>[Plantilla](#tab/template)

> [!div class="nextstepaction"]
> [Uso de blobs con Azure Portal](../blobs/storage-quickstart-blobs-portal.md)

---
