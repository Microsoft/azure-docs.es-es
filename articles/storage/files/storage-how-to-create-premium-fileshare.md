---
title: Creación de un recurso compartido de archivos de Azure premium
description: Aprenda a crear un recurso compartido de archivos de Azure Premium mediante Azure Portal, el módulo de Azure PowerShell o la CLI de Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 08/26/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 75ba8e1e2037ba8ef249b548dfb38e5fd1618cb2
ms.sourcegitcommit: 6e1124fc25c3ddb3053b482b0ed33900f46464b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2020
ms.locfileid: "90564192"
---
# <a name="how-to-create-an-azure-premium-file-share"></a>Creación de un recurso compartido de archivos de Azure Premium

Los recursos compartidos de archivos Premium se ofrecen en medios de almacenamiento de discos de estado sólido (SSD) y son útiles para cargas de trabajo de E/S intensiva, incluido el hospedaje de bases de datos y de informática de alto rendimiento (HPC). Los recursos compartidos de archivos Premium se hospedan en un tipo de cuenta de almacenamiento de propósito especial denominada cuenta FileStorage. Los recursos compartidos de archivos Premium están diseñados para aplicaciones de alto rendimiento y escala empresarial, que proporcionan recursos compartidos coherentes de baja latencia, IOPS alta y alto rendimiento.

En este artículo se muestra cómo crear este nuevo tipo de cuenta con [Azure Portal](https://portal.azure.com/), el módulo de Azure PowerShell y la CLI de Azure.

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Si planea usar la CLI de Azure, [instale la versión más reciente](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).
- Si planea usar el módulo de Azure PowerShell, [instale la versión más reciente](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-4.6.0).

## <a name="create-a-filestorage-storage-account"></a>Creación de una cuenta de almacenamiento FileStorage

Cada cuenta de almacenamiento debe pertenecer a un grupo de recursos de Azure. Un grupo de recursos es un contenedor lógico para agrupar servicios de Azure. Al crear una cuenta de almacenamiento, puede elegir entre crear un grupo de recursos o usar uno existente. Los recursos compartidos de archivos Premium requieren una cuenta de FileStorage.

# <a name="portal"></a>[Portal](#tab/azure-portal)

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com/).

Ahora ya está listo para crear la cuenta de almacenamiento.

1. En Azure Portal, seleccione **Cuentas de almacenamiento** en el menú de la izquierda.

    ![Página principal de Azure Portal con la selección de la cuenta de almacenamiento](media/storage-how-to-create-premium-fileshare/azure-portal-storage-accounts.png)

1. En la ventana **Cuentas de almacenamiento** que aparece, elija **Agregar**.
1. Seleccione la suscripción en la que se va a crear la cuenta de almacenamiento.
1. En el campo **Grupo de recursos**, haga clic en **Crear nuevo**. Escriba un nombre para el nuevo grupo de recursos, como se muestra en la siguiente imagen.

1. Después, escriba un nombre para la cuenta de almacenamiento. El nombre que elija debe ser único en Azure. El nombre debe tener también una longitud de entre 3 y 24 caracteres y solo puede contener números y letras minúsculas.
1. Seleccione una ubicación para la cuenta de almacenamiento o utilice la ubicación predeterminada.
1. En **Rendimiento**, seleccione **Premium**.

    Debe seleccionar **Premium** para que **FileStorage** sea una opción disponible en la lista desplegable **Tipo de cuenta**.

1. Seleccione **Tipo de cuenta** y elija **FileStorage**.
1. Mantenga la opción **Replicación** establecida en su valor predeterminado de **Almacenamiento con redundancia local (LRS)** .

    ![Procedimientos para crear una cuenta de almacenamiento para un recurso compartido de archivos Premium](media/storage-how-to-create-premium-fileshare/create-filestorage-account.png)

1. Seleccione **Revisar y crear** para revisar la configuración de la cuenta de almacenamiento y crear la cuenta.
1. Seleccione **Crear**.

Una vez que se ha creado el recurso de la cuenta de almacenamiento, vaya hasta él.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

### <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Use el comando `Connect-AzAccount` y siga las instrucciones en pantalla para realizar la autenticación.

```powershell
Connect-AzAccount
```

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos con PowerShell, use el comando [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup):

```powershell
# put resource group in a variable so you can use the same group name going forward,
# without hardcoding it repeatedly
$resourceGroup = "storage-how-to-resource-group"
$location = "westus2"
New-AzResourceGroup -Name $resourceGroup -Location $location
```

### <a name="create-a-filestorage-storage-account"></a>Creación de una cuenta de almacenamiento FileStorage

Para crear una cuenta de almacenamiento de FileStorage desde PowerShell, use el comando [New-AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount):

```powershell
$storageAcct = New-AzStorageAccount -ResourceGroupName $resourceGroup -Name "fileshowto" -SkuName "Premium_LRS" -Location "westus2" -Kind "FileStorage"
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para iniciar Azure Cloud Shell, inicie sesión en [Azure Portal](https://portal.azure.com).

Si quiere iniciar sesión en la instalación local de la CLI, asegúrese de que tiene la versión más reciente y, luego, inicie sesión:

```azurecli
az login
```

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos con la CLI de Azure, use el comando [az group create](/cli/azure/group).

```azurecli-interactive
az group create `
    --name files-howto-resource-group `
    --location westus2
```

### <a name="create-a-filestorage-storage-account"></a>Creación de una cuenta de almacenamiento FileStorage

Para crear una cuenta de almacenamiento FileStorage desde la CLI de Azure, use el comando [az storage account create](/cli/azure/storage/account).

```azurecli-interactive
az storage account create `
    --name fileshowto `
    --resource-group files-howto-resource-group `
    --location westus `
    --sku Premium_LRS `
    --kind FileStorage
```

### <a name="get-the-storage-account-key"></a>Obtención de la clave de la cuenta de almacenamiento

Las claves de cuenta de almacenamiento controlan el acceso a los recursos de una cuenta de almacenamiento; en este artículo se usa la clave para crear un recurso compartido de archivos Premium. Las claves se crean automáticamente al crear una cuenta de almacenamiento. Puede obtener las claves de cuenta de almacenamiento con el comando [az storage account keys list](/cli/azure/storage/account/keys):

```azurecli-interactive
STORAGEKEY=$(az storage account keys list \
    --resource-group "myResourceGroup" \
    --account-name $STORAGEACCT \
    --query "[0].value" | tr -d '"')
```
---

## <a name="create-a-premium-file-share"></a>Creación de un recurso compartido de archivos premium

Ahora que ha creado una cuenta de FileStorage, puede crear un recurso compartido de archivos Premium con esa cuenta de almacenamiento.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En el menú de la izquierda de la cuenta de almacenamiento, desplácese a la sección **Servicio de archivo** y, después, seleccione **Archivos**.
1. Seleccione **Recurso compartido de archivos** para crear un recurso compartido de archivos Premium.
1. Escriba un nombre y una cuota deseada para el recurso compartido de archivos y luego seleccione **Crear**.

> [!NOTE]
> Los tamaños de recurso compartido aprovisionados se especifican mediante la cuota del recurso compartido y los recursos compartidos de archivos se facturan en función del tamaño aprovisionado. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/storage/files/)para obtener más información.

   ![Creación de un recurso compartido de archivos premium](media/storage-how-to-create-premium-fileshare/create-premium-file-share.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para crear un recurso compartido de archivos Premium con el módulo de Azure PowerShell, use el cmdlet [New-AzStorageShare](/powershell/module/az.storage/New-AzStorageShare).

> [!NOTE]
> Los tamaños de recurso compartido aprovisionados se especifican mediante la cuota del recurso compartido y los recursos compartidos de archivos se facturan en función del tamaño aprovisionado. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/storage/files/)para obtener más información.

```powershell
New-AzStorageShare `
   -Name myshare `
   -EnabledProtocol SMB `
   -Context $storageAcct.Context
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para crear un recurso compartido de archivos Premium con la CLI de Azure, utilice el comando [az storage share create](/cli/azure/storage/share).

> [!NOTE]
> Los tamaños de recurso compartido aprovisionados se especifican mediante la cuota del recurso compartido y los recursos compartidos de archivos se facturan en función del tamaño aprovisionado. Consulte la [página de precios](https://azure.microsoft.com/pricing/details/storage/files/)para obtener más información.

```azurecli-interactive
az storage share create \
    --account-name $STORAGEACCT \
    --account-key $STORAGEKEY \
    --enabled-protocol SMB \
    --name "myshare" 
```
---

## <a name="clean-up-resources"></a>Limpieza de recursos

# <a name="portal"></a>[Portal](#tab/azure-portal)

Si quiere limpiar los recursos creados en este artículo, elimine el grupo de recursos. Al eliminar el grupo de recursos también se elimina la cuenta de almacenamiento asociada y cualquier otro recurso que esté asociado a ese grupo.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Si quiere limpiar los recursos creados en este artículo, elimine el grupo de recursos. Al eliminar el grupo de recursos también se elimina la cuenta de almacenamiento asociada y cualquier otro recurso que esté asociado a ese grupo.

Para quitar el grupo de recursos y sus recursos asociados, incluida la nueva cuenta de almacenamiento, use el comando [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup): 

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Si quiere limpiar los recursos creados en este artículo, elimine el grupo de recursos. Al eliminar el grupo de recursos también se elimina la cuenta de almacenamiento asociada y cualquier otro recurso que esté asociado a ese grupo.

Para quitar el grupo de recursos y sus recursos asociados, incluida la nueva cuenta de almacenamiento, use el comando [az group delete](/cli/azure/group).

```azurecli-interactive
az group delete --name myResourceGroup
```
---

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha creado un recurso compartido de archivos Premium. Para obtener información sobre el rendimiento que ofrece esta cuenta, continúe a la sección sobre niveles de rendimiento de la guía de plan.

> [!div class="nextstepaction"]
> [Niveles recursos compartidos de archivos](storage-files-planning.md#storage-tiers)
