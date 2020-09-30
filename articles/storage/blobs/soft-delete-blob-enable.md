---
title: Habilitación y administración de la eliminación temporal para blobs
titleSuffix: Azure Storage
description: Habilitación de la eliminación temporal para blobs a fin de recuperar sus datos con mayor facilidad al modificarse o eliminarse estos de forma errónea.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/15/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurecli, devx-track-csharp
ms.openlocfilehash: 07e306c6c428a0e3a3f64a29a2574cb0378bb81f
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89230236"
---
# <a name="enable-and-manage-soft-delete-for-blobs"></a>Habilitación y administración de la eliminación temporal para blobs

La eliminación temporal de blobs evita que los datos se modifiquen o eliminen de forma accidental o errónea. Cuando la eliminación temporal de blobs está habilitada para una cuenta de almacenamiento, los blobs, las versiones de blobs y las instantáneas de esa cuenta de almacenamiento pueden recuperarse una vez eliminados, dentro de un período de retención que especifique.

Si existe la posibilidad de que una aplicación u otro usuario de la cuenta de almacenamiento modifiquen o borren sus datos accidentalmente, Microsoft aconseja activar la eliminación temporal de blobs. En este artículo se muestra cómo habilitar la eliminación temporal para blobs. Para más información sobre la eliminación temporal de blobs, consulte [Eliminación temporal para blobs](soft-delete-blob-overview.md).

Para información sobre cómo habilitar también la eliminación temporal para contenedores, consulte [Habilitación y administración de la eliminación temporal para contenedores](soft-delete-container-enable.md).

## <a name="enable-blob-soft-delete"></a>Habilitación de la eliminación temporal de blobs

# <a name="portal"></a>[Portal](#tab/azure-portal)

Habilite la eliminación temporal de blobs en la cuenta de almacenamiento mediante Azure Portal:

1. En [Azure Portal](https://portal.azure.com/), vaya a la cuenta de almacenamiento.
1. Busque la opción **Protección de datos** en **Blob service**.
1. Establezca la propiedad **Blob soft delete** (Eliminación temporal de blobs) en *Habilitada*.
1. En **Directivas de retención**, especifique cuánto tiempo se conservan los blobs eliminados temporalmente mediante Azure Storage.
1. Guarde los cambios.

![Captura de pantalla de Azure Portal con el servicio de protección de datos de blobs elegido.](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-configuration.png)

Para ver los blobs eliminados temporalmente, active la casilla **Mostrar blobs eliminados**.

![Captura de pantalla de la página del servicio de protección de datos de blobs con la opción Mostrar blobs eliminados resaltada.](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-view-soft-deleted.png)

Para ver instantáneas eliminadas temporalmente de un blob dado, seleccione el blob y haga clic en **Ver instantáneas**.

![Captura de pantalla de la página del servicio de protección de datos de blobs con la opción Ver instantáneas resaltada.](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-view-soft-deleted-snapshots.png)

Asegúrese de que la casilla **Mostrar instantáneas eliminadas** está activada.

![Captura de pantalla de la página Ver instantáneas con la opción Mostrar blobs eliminados resaltada.](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-view-soft-deleted-snapshots-check.png)

Al hacer clic en un blob o en una instantánea eliminados temporalmente, observe las nuevas propiedades del blob. Indican cuándo se eliminó el objeto y el número de días que quedan hasta que el blob o la instantánea de blob expire de forma permanente. Si el objeto que se ha eliminado temporalmente no es una instantánea, también tendrá la opción de recuperarlo.

![Captura de pantalla de los detalles de un objeto eliminado temporalmente.](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-properties.png)

Recuerde que al recuperar un blob también se recuperan todas las instantáneas asociadas. Para recuperar las instantáneas eliminadas temporalmente de un blob activo, haga clic en el blob y seleccione **Undelete all snapshots** (Recuperar todas las instantáneas).

![Captura de pantalla de los detalles de un blob eliminado temporalmente.](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-undelete-all-snapshots.png)

Después de recuperar las instantáneas de un blob, puede hacer clic en **Promover** para copiar una instantánea en el blob raíz, con lo que se restaura el blob en la instantánea.

![Captura de pantalla de la página Ver instantáneas con la opción Promote (Promover) resaltada.](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-promote-snapshot.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Para habilitar la eliminación temporal, actualice las propiedades del servicio del cliente del blob. En el ejemplo siguiente se habilita la eliminación temporal en un subconjunto de cuentas de una suscripción:

```powershell
Set-AzContext -Subscription "<subscription-name>"
$MatchingAccounts = Get-AzStorageAccount | where-object{$_.StorageAccountName -match "<matching-regex>"}
$MatchingAccounts | Enable-AzStorageDeleteRetentionPolicy -RetentionDays 7
```

Puede comprobar que la eliminación temporal se ha activado mediante el siguiente comando:

```powershell
$MatchingAccounts | $account = Get-AzStorageAccount -ResourceGroupName myresourcegroup -Name storageaccount
   Get-AzStorageServiceProperty -ServiceType Blob -Context $account.Context | Select-Object -ExpandProperty DeleteRetentionPolicy
```

Para recuperar los blobs que se eliminaron accidentalmente, llame a **Undelete Blob** (Recuperar blob) en ellos. Recuerde que al llamar a **Undelete Blob**, tanto en los blobs activos como en los eliminados temporalmente, restaurará todas las instantáneas asociadas eliminadas temporalmente como activas. En el ejemplo siguiente se llama a **Undelete Blob** (Recuperar blob) en todos los blobs activos y eliminados temporalmente de un contenedor:

```powershell
# Create a context by specifying storage account name and key
$ctx = New-AzStorageContext -StorageAccountName $StorageAccountName -StorageAccountKey $StorageAccountKey

# Get the blobs in a given container and show their properties
$Blobs = Get-AzStorageBlob -Container $StorageContainerName -Context $ctx -IncludeDeleted
$Blobs.ICloudBlob.Properties

# Undelete the blobs
$Blobs.ICloudBlob.Undelete()
```
Para buscar la directiva de retención de eliminación temporal actual, use el siguiente comando:

```azurepowershell-interactive
   $account = Get-AzStorageAccount -ResourceGroupName myresourcegroup -Name storageaccount
   Get-AzStorageServiceProperty -ServiceType Blob -Context $account.Context
```

# <a name="cli"></a>[CLI](#tab/azure-CLI)

Para habilitar la eliminación temporal, actualice las propiedades del servicio del cliente del blob:

```azurecli-interactive
az storage blob service-properties delete-policy update --days-retained 7  --account-name mystorageaccount --enable true
```

Para comprobar si la opción eliminación temporal está activada, use el siguiente comando: 

```azurecli-interactive
az storage blob service-properties delete-policy show --account-name mystorageaccount 
```

# <a name="python"></a>[Python](#tab/python)

Para habilitar la eliminación temporal, actualice las propiedades del servicio del cliente del blob:

```python
# Make the requisite imports
from azure.storage.blob import BlockBlobService
from azure.storage.common.models import DeleteRetentionPolicy

# Initialize a block blob service
block_blob_service = BlockBlobService(
    account_name='<enter your storage account name>', account_key='<enter your storage account key>')

# Set the blob client's service property settings to enable soft delete
block_blob_service.set_blob_service_properties(
    delete_retention_policy=DeleteRetentionPolicy(enabled=True, days=7))
```

# <a name="net-v12"></a>[.NET v12](#tab/dotnet)

Para habilitar la eliminación temporal, actualice las propiedades del servicio del cliente del blob:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/DataProtection.cs" id="Snippet_EnableSoftDelete":::

Para recuperar los blobs que se eliminaron accidentalmente, llame a Undelete en ellos. Recuerde que al llamar a **Recuperar**, tanto en los blobs activos como en los eliminados temporalmente, restaurará todas las instantáneas asociadas eliminadas temporalmente como activas. En el ejemplo siguiente se llama a Undelete en todos los blobs activos y eliminados temporalmente de un contenedor:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/DataProtection.cs" id="Snippet_RecoverDeletedBlobs":::

Para recuperar hasta una versión específica del blob, primero llame a Undelete en un blob y, después, copie la instantánea deseada sobre el blob. En el ejemplo siguiente se recupera un blob en bloques a su última instantánea generada:

:::code language="csharp" source="~/azure-storage-snippets/blobs/howto/dotnet/dotnet-v12/DataProtection.cs" id="Snippet_RecoverSpecificBlobVersion":::

# <a name="net-v11"></a>[.NET v11](#tab/dotnet11)

Para habilitar la eliminación temporal, actualice las propiedades del servicio del cliente del blob:

```csharp
// Get the blob client's service property settings
ServiceProperties serviceProperties = blobClient.GetServiceProperties();

// Configure soft delete
serviceProperties.DeleteRetentionPolicy.Enabled = true;
serviceProperties.DeleteRetentionPolicy.RetentionDays = RetentionDays;

// Set the blob client's service property settings
blobClient.SetServiceProperties(serviceProperties);
```

Para recuperar los blobs que se eliminaron accidentalmente, llame a **Undelete Blob** (Recuperar blob) en ellos. Recuerde que al llamar a **Undelete Blob**, tanto en los blobs activos como en los eliminados temporalmente, restaurará todas las instantáneas asociadas eliminadas temporalmente como activas. En el ejemplo siguiente se llama a **Undelete Blob** (Recuperar blob) en todos los blobs activos y eliminados temporalmente de un contenedor:

```csharp
// Recover all blobs in a container
foreach (CloudBlob blob in container.ListBlobs(useFlatBlobListing: true, blobListingDetails: BlobListingDetails.Deleted))
{
       await blob.UndeleteAsync();
}
```

Para recuperar hasta una versión específica del blob, primero llame a la operación **Undelete Blob** (Recuperar blob) y, después, copie la instantánea deseada sobre el blob. En el ejemplo siguiente se recupera un blob en bloques a su última instantánea generada:

```csharp
// Undelete
await blockBlob.UndeleteAsync();

// List all blobs and snapshots in the container prefixed by the blob name
IEnumerable<IListBlobItem> allBlobVersions = container.ListBlobs(
    prefix: blockBlob.Name, useFlatBlobListing: true, blobListingDetails: BlobListingDetails.Snapshots);

// Restore the most recently generated snapshot to the active blob
CloudBlockBlob copySource = allBlobVersions.First(version => ((CloudBlockBlob)version).IsSnapshot &&
    ((CloudBlockBlob)version).Name == blockBlob.Name) as CloudBlockBlob;
blockBlob.StartCopy(copySource);
```  

---

## <a name="next-steps"></a>Pasos siguientes

- [Eliminación temporal para Blob Storage](soft-delete-overview.md)
- [Control de versiones de blobs](versioning-overview.md)
