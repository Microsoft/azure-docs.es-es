---
title: Habilitación y creación de recursos compartidos de archivos grandes - Azure Files
description: En este artículo aprenderá a habilitar y crear recursos compartidos de archivos grandes.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 05/29/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurecli
ms.openlocfilehash: 538cbfea2480573c190a1dd0ee0480e21aecefe2
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87502093"
---
# <a name="enable-and-create-large-file-shares"></a>Habilitación y creación de recursos compartidos de archivos grandes

Cuando se habilitan recursos compartidos de archivos de gran tamaño en la cuenta de almacenamiento, los recursos compartidos de archivos se pueden escalar verticalmente hasta 100 TiB. Esta escalabilidad se puede habilitar para los recursos compartidos de archivos existentes en las cuentas de almacenamiento existentes.

## <a name="prerequisites"></a>Requisitos previos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
- Si planea usar la CLI de Azure, [instale la versión más reciente](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest).
- Si planea usar Azure PowerShell, [instale la versión más reciente](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-3.0.0).

## <a name="restrictions"></a>Restricciones

Por ahora, solo puede usar el almacenamiento con redundancia local (LRS) o el almacenamiento con redundancia de zona (ZRS) en cuentas habilitadas para recursos compartidos de archivos de gran tamaño. No puede usar el almacenamiento con redundancia de zona geográfica (GZRS), el almacenamiento con redundancia geográfica (GRS), el almacenamiento con redundancia geográfica con acceso de lectura (RA-GRS) ni el almacenamiento con redundancia de zona geográfica con acceso de lectura (RA-GZRS).

La habilitación de recursos compartidos de archivos grandes en una cuenta es un proceso irreversible. Después de habilitarlos, no podrá convertir la cuenta a GZRS, GRS, RA-GRS o RA-GZRS.

## <a name="create-a-new-storage-account"></a>Creación de una cuenta de almacenamiento nueva

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. En Azure Portal, seleccione **Todos los servicios**. 
1. En la lista de recursos, escriba **Cuentas de almacenamiento**. La lista se filtra en función de lo que escriba. Seleccione **Cuentas de almacenamiento**.
1. En la ventana **Cuentas de almacenamiento** que aparece, seleccione **Agregar**.
1. Seleccione la suscripción que usará para crear la cuenta de almacenamiento.
1. En el campo **Grupo de recursos**, haga clic en **Crear nuevo**. Escriba un nombre para el nuevo grupo de recursos.

    ![Captura de pantalla que muestra cómo crear un grupo de recursos en el portal](media/storage-files-how-to-create-large-file-share/create-large-file-share.png)

1. Después, escriba un nombre para la cuenta de almacenamiento. El nombre debe ser único en Azure. El nombre también debe tener entre 3 y 24 caracteres de longitud y solo puede incluir números y letras minúsculas.
1. Seleccione la ubicación para la cuenta de almacenamiento.
1. Establezca la replicación en **Almacenamiento con redundancia local** o **Almacenamiento con redundancia de zona**.
1. Deje estos campos con sus valores predeterminados:

   |Campo  |Value  |
   |---------|---------|
   |Modelo de implementación     |Resource Manager         |
   |Rendimiento     |Estándar         |
   |Tipo de cuenta     |StorageV2 (uso general v2)         |
   |Nivel de acceso     |Acceso frecuente         |

1. Seleccione **Avanzado** y, luego, seleccione el botón de opción **Habilitado** a la derecha de **Large file shares** (Recursos compartidos de archivos grandes).
1. Seleccione **Revisar y crear** para revisar la configuración de la cuenta de almacenamiento y crear la cuenta.

    ![Captura de pantalla con la opción "habilitado" en una nueva cuenta de almacenamiento de Azure Portal](media/storage-files-how-to-create-large-file-share/large-file-shares-advanced-enable.png)

1. Seleccione **Crear**.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

En primer lugar, [instale la versión más reciente de la CLI de Azure](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) para que pueda habilitar recursos compartidos de archivos grandes.

Para crear una cuenta de almacenamiento que tenga habilitados recursos compartidos de archivos grandes, use el siguiente comando. Reemplace `<yourStorageAccountName>`, `<yourResourceGroup>` y `<yourDesiredRegion>` por su información.

```azurecli-interactive
## This command creates a large file share–enabled account. It will not support GZRS, GRS, RA-GRS, or RA-GZRS.
az storage account create --name <yourStorageAccountName> -g <yourResourceGroup> -l <yourDesiredRegion> --sku Standard_LRS --kind StorageV2 --enable-large-file-share
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

En primer lugar, [instale la versión más reciente de PowerShell](https://docs.microsoft.com/powershell/azure/install-az-ps?view=azps-3.0.0) para poder habilitar recursos compartidos de archivos grandes.

Para crear una cuenta de almacenamiento que tenga habilitados recursos compartidos de archivos grandes, use el siguiente comando. Reemplace `<yourStorageAccountName>`, `<yourResourceGroup>` y `<yourDesiredRegion>` por su información.

```powershell
## This command creates a large file share–enabled account. It will not support GZRS, GRS, RA-GRS, or RA-GZRS.
New-AzStorageAccount -ResourceGroupName <yourResourceGroup> -Name <yourStorageAccountName> -Location <yourDesiredRegion> -SkuName Standard_LRS -EnableLargeFileShare;
```
---

## <a name="enable-large-files-shares-on-an-existing-account"></a>Habilitación de recursos compartidos de archivos grandes en una cuenta existente

También puede habilitar recursos compartidos de archivos grandes en las cuentas existentes. En este caso, no podrá convertirlos a GZRS, GRS, RA-GRS o RA-GZRS. La habilitación de recursos compartidos de archivos grandes es irreversible en esta cuenta de almacenamiento.

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. Abra [Azure Portal](https://portal.azure.com) y vaya a la cuenta de almacenamiento en la que quiere habilitar recursos compartidos de archivos grandes.
1. Abra la cuenta de almacenamiento y seleccione **Configuración**.
1. Seleccione **Habilitado** en **Large file shares** (Recursos compartidos de archivos grandes) y, a continuación, seleccione **Guardar**.
1. Seleccione **Información general** y elija **Actualizar**.

![Selección del botón de opción Habilitado en una cuenta de almacenamiento existente en Azure Portal](media/storage-files-how-to-create-large-file-share/enable-large-file-shares-on-existing.png)

Ahora ha habilitado los recursos compartidos de archivos grandes en la cuenta de almacenamiento. Luego, debe [actualizar la cuota del recurso compartido existente](#expand-existing-file-shares) para aprovechar las ventajas de una mayor capacidad y escalabilidad.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para habilitar recursos compartidos de archivos grandes en una cuenta existente, use el siguiente comando. Reemplace `<yourStorageAccountName>` y `<yourResourceGroup>` por su propia información.

```azurecli-interactive
az storage account update --name <yourStorageAccountName> -g <yourResourceGroup> --enable-large-file-share
```

Ahora ha habilitado los recursos compartidos de archivos grandes en la cuenta de almacenamiento. Luego, debe [actualizar la cuota del recurso compartido existente](#expand-existing-file-shares) para aprovechar las ventajas de una mayor capacidad y escalabilidad.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para habilitar recursos compartidos de archivos grandes en una cuenta existente, use el siguiente comando. Reemplace `<yourStorageAccountName>` y `<yourResourceGroup>` por su propia información.

```powershell
Set-AzStorageAccount -ResourceGroupName <yourResourceGroup> -Name <yourStorageAccountName> -EnableLargeFileShare
```

Ahora ha habilitado los recursos compartidos de archivos grandes en la cuenta de almacenamiento. Luego, debe [actualizar la cuota del recurso compartido existente](#expand-existing-file-shares) para aprovechar las ventajas de una mayor capacidad y escalabilidad.

---

## <a name="create-a-large-file-share"></a>Creación de un recurso compartido de archivos grandes

Después de habilitar recursos compartidos de archivos grandes en la cuenta de almacenamiento, puede crear recursos compartidos de archivos en esa cuenta con cuotas más altas. 

# <a name="portal"></a>[Portal](#tab/azure-portal)

La creación de un recurso compartido de archivos grandes es casi idéntica a la creación de un recurso compartido de archivos estándar. La principal diferencia es que puede establecer una cuota de hasta 100 TiB.

1. En la cuenta de almacenamiento, seleccione **Recurso compartido de archivos**.
1. Seleccione **+ Recurso compartido de archivos**.
1. Escriba el nombre del recurso compartido de archivos. También puede establecer el tamaño de cuota que prefiera, hasta 100 TiB. Seleccione **Crear**. 

![La interfaz de usuario de Azure Portal con los cuadros de nombre y cuota](media/storage-files-how-to-create-large-file-share/large-file-shares-create-share.png)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para crear un recurso compartido de archivos grande, use el siguiente comando. Reemplace `<yourStorageAccountName>`, `<yourStorageAccountKey>` y `<yourFileShareName>` por su información.

```azurecli-interactive
az storage share create --account-name <yourStorageAccountName> --account-key <yourStorageAccountKey> --name <yourFileShareName>
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para crear un recurso compartido de archivos grande, use el siguiente comando. Reemplace `<YourStorageAccountName>`, `<YourStorageAccountKey>` y `<YourStorageAccountFileShareName>` por su información.

```powershell
##Config
$storageAccountName = "<YourStorageAccountName>"
$storageAccountKey = "<YourStorageAccountKey>"
$shareName="<YourStorageAccountFileShareName>"
$ctx = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
New-AzStorageShare -Name $shareName -Context $ctx
```
---

## <a name="expand-existing-file-shares"></a>Expansión de recursos compartidos de archivos existentes

Después de habilitar recursos compartidos de archivos grandes en la cuenta de almacenamiento, puede expandir también los recursos compartidos de archivos existentes en esa cuenta a una cuota más alta. 

# <a name="portal"></a>[Portal](#tab/azure-portal)

1. En la cuenta de almacenamiento, seleccione **Recurso compartido de archivos**.
1. Haga clic con el botón derecho en el recurso compartido de archivos y seleccione **Cuota**.
1. Escriba el nuevo tamaño que quiera y, luego, seleccione **Aceptar**.

![La interfaz de usuario de Azure Portal con la cuota de los recursos compartidos de archivos existentes](media/storage-files-how-to-create-large-file-share/update-large-file-share-quota.png)

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para establecer la cuota en el tamaño máximo, use el siguiente comando. Reemplace `<yourStorageAccountName>`, `<yourStorageAccountKey>` y `<yourFileShareName>` por su información.

```azurecli-interactive
az storage share update --account-name <yourStorageAccountName> --account-key <yourStorageAccountKey> --name <yourFileShareName> --quota 102400
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Para establecer la cuota en el tamaño máximo, use el siguiente comando. Reemplace `<YourStorageAccountName>`, `<YourStorageAccountKey>` y `<YourStorageAccountFileShareName>` por su información.

```powershell
##Config
$storageAccountName = "<YourStorageAccountName>"
$storageAccountKey = "<YourStorageAccountKey>"
$shareName="<YourStorageAccountFileShareName>"
$ctx = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
# update quota
Set-AzStorageShareQuota -ShareName $shareName -Context $ctx -Quota 102400
```
---

## <a name="next-steps"></a>Pasos siguientes

* [Conexión y montaje de un recurso compartido de archivos en Windows](storage-how-to-use-files-windows.md)
* [Conexión y montaje de un recurso compartido de archivos en Linux](storage-how-to-use-files-linux.md)
* [Conexión y montaje de un recurso compartido de archivos en macOS](storage-how-to-use-files-mac.md)
