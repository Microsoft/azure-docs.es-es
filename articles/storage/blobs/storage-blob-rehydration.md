---
title: Rehidratación de los datos de blob desde el nivel de archivo
description: Rehidrate los blobs de Archive Storage para poder acceder a los datos.
services: storage
author: mhopkins-msft
ms.author: mhopkins
ms.date: 04/08/2020
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.reviewer: hux
ms.openlocfilehash: 9d1d663dce8791b70b9fd8679730d5681d66013a
ms.sourcegitcommit: dccb85aed33d9251048024faf7ef23c94d695145
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/28/2020
ms.locfileid: "87282473"
---
# <a name="rehydrate-blob-data-from-the-archive-tier"></a>Rehidratación de los datos de blob desde el nivel de archivo

Mientras un blob se encuentra en el nivel de acceso de archivo, se considera sin conexión y no se puede leer ni modificar. Los metadatos de blob quedan en línea y se mantienen disponibles, lo que permite enumerar el blob y sus propiedades. La lectura y modificación de los datos de blob solo están disponibles con niveles en línea, como acceso frecuente o esporádico. Hay dos opciones para recuperar los datos almacenados en el nivel de acceso de archivo y obtener acceso a ellos.

1. [Rehidratación de un blob archivado a un nivel en línea](#rehydrate-an-archived-blob-to-an-online-tier): rehidrate un blob de archivo a acceso frecuente o esporádico cambiando su nivel mediante la operación [Establecer el nivel del blob](https://docs.microsoft.com/rest/api/storageservices/set-blob-tier).
2. [Copia de un blob archivado en un nivel en línea](#copy-an-archived-blob-to-an-online-tier): cree una nueva copia de blob de archivo mediante la operación [Copiar blob](https://docs.microsoft.com/rest/api/storageservices/copy-blob). Especifique otro nombre de blob y un nivel de destino de acceso frecuente o esporádico.

 Para obtener más información sobre los niveles de acceso, consulte [Azure Blob Storage: niveles de acceso frecuente, esporádico y de archivo](storage-blob-storage-tiers.md).

## <a name="rehydrate-an-archived-blob-to-an-online-tier"></a>Rehidratación de un blob archivado en un nivel en línea

[!INCLUDE [storage-blob-rehydration](../../../includes/storage-blob-rehydrate-include.md)]

## <a name="copy-an-archived-blob-to-an-online-tier"></a>Copia de un blob archivado en un nivel en línea

Si no quiere rehidratar un archivo blob, puede elegir realizar una operación [Copiar blob](https://docs.microsoft.com/rest/api/storageservices/copy-blob). Su blob original permanecerá sin modificaciones en el archivo, mientras se crea un nuevo blob en el nivel frecuente o esporádico en línea para que pueda trabajar. En la operación Copiar blob, también puede establecer la propiedad opcional *​​x-ms-rehydrate-priority* en Estándar o Alta para especificar la prioridad con la que desea que se cree su copia de blob.

La copia de un BLOB desde el archivo puede tardar horas en completarse, en función de la prioridad de rehidratación seleccionada. En segundo plano, la operación **Copiar blob** lee el blob de origen de archivo para crear un nuevo blob en línea en el nivel de destino seleccionado. El nuevo blob puede ser visible cuando enumera los blobs, pero los datos no estarán disponibles hasta que la lectura del blob de archivo de origen esté completa y los datos se escriban en el nuevo blob de destino en línea. El nuevo BLOB es una copia independiente y cualquier modificación o eliminación en él no afecta al BLOB de archivo de origen.

> [!IMPORTANT]
> No elimine el blob de origen hasta que la copia se complete correctamente en el destino. Si se elimina el blob de origen, es posible que el blob de destino no finalice la copia y quedará vacío. Puede comprobar el elemento*x-ms-copy-status* para determinar el estado de la operación de copia.

Los blobs de archivo solo se pueden copiar en los niveles de destino en línea dentro de la misma cuenta de almacenamiento. No se admite la copia de un blob de archivo en otro blob de archivo. En la tabla siguiente se indican las funcionalidades de CopyBlob.

|                                           | **Origen de nivel de acceso frecuente**   | **Origen de nivel de acceso esporádico** | **Origen de nivel de archivo**    |
| ----------------------------------------- | --------------------- | -------------------- | ------------------- |
| **Destino de nivel de acceso frecuente**                  | Compatible             | Compatible            | Se admite dentro de la misma cuenta; rehidratación pendiente               |
| **Destino de nivel de acceso esporádico**                 | Compatible             | Compatible            | Se admite dentro de la misma cuenta; rehidratación pendiente               |
| **Destino de nivel de archivo**              | Compatible             | Compatible            | No compatible         |

## <a name="pricing-and-billing"></a>Precios y facturación

La rehidratación de blobs del archivo en los niveles de acceso frecuente o esporádico se cobran como operaciones de lectura y recuperación de datos. El uso de la prioridad alta tiene mayores costos de operación y recuperación de datos que los de la prioridad estándar. La rehidratación de alta prioridad aparece como una partida independiente en la factura. Si una solicitud de prioridad alta para devolver un blob de archivo de varios gigabytes tarda más de 5 horas, no se le cobrará la tarifa de recuperación de prioridad alta. Sin embargo, se siguen aplicando las tarifas de recuperación estándar, ya que la rehidratación se prioriza en otras solicitudes.

La copia de blobs del archivo en los niveles de acceso frecuente o esporádico se cobran como operaciones de lectura y recuperación de datos. Una operación de escritura se cobra por la creación de la nueva copia de BLOB. Las tarifas de eliminación anticipada no se aplican al realizar la copia en un blob en línea porque el blob de origen permanece sin modificar en el nivel de archivo. Si se selecciona, se aplican cargos por recuperación de alta prioridad.

Los blobs de nivel de archivo deben estar almacenados durante un mínimo de 180 días. La eliminación o rehidratación de blobs archivados antes de 180 días incurrirá en tarifas de eliminación anticipada.

> [!NOTE]
> Para obtener más información sobre los precios de los blobs en bloques, consulte la página [Precios de Azure Storage](https://azure.microsoft.com/pricing/details/storage/blobs/). Para obtener más información sobre los cargos por la transferencia de datos salientes, consulte la página [Detalles de precios de ancho de banda](https://azure.microsoft.com/pricing/details/data-transfers/).

## <a name="quickstart-scenarios"></a>Escenarios de inicio rápido

### <a name="rehydrate-an-archive-blob-to-an-online-tier"></a>Rehidratación de un blob de archivo en un nivel en línea
# <a name="portal"></a>[Portal](#tab/azure-portal)
1. Inicie sesión en [Azure Portal](https://portal.azure.com).

1. En Azure Portal, busque y seleccione **Todos los recursos**.

1. Seleccione su cuenta de almacenamiento.

1. Seleccione el contenedor y, luego, seleccione el blob.

1. En las **propiedades del blob**, seleccione **Cambiar nivel**.

1. Seleccione el nivel de acceso **Frecuente** o **Esporádico**. 

1. Seleccione una prioridad de rehidratación **Estándar** o **Alta**.

1. En la parte inferior, seleccione **Guardar**.

![Cambiar el nivel de la cuenta de almacenamiento](media/storage-tiers/blob-access-tier.png)
![Comprobar el estado de rehidratación](media/storage-tiers/rehydrate-status.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
El siguiente script de PowerShell se puede usar para cambiar el nivel de blob de un blob de archivo. La variable `$rgName` se debe inicializar con el nombre del grupo de recursos. La variable `$accountName` se debe inicializar con el nombre de la cuenta de almacenamiento. La variable `$containerName` se debe inicializar con el nombre del contenedor. La variable `$blobName` se debe inicializar con el nombre del blob. 
```powershell
#Initialize the following with your resource group, storage account, container, and blob names
$rgName = ""
$accountName = ""
$containerName = ""
$blobName = ""

#Select the storage account and get the context
$storageAccount =Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

#Select the blob from a container
$blob = Get-AzStorageBlob -Container $containerName -Blob $blobName -Context $ctx

#Change the blob’s access tier to Hot using Standard priority rehydrate
$blob.ICloudBlob.SetStandardBlobTier("Hot", "Standard")
```
---

### <a name="copy-an-archive-blob-to-a-new-blob-with-an-online-tier"></a>Copia de un blob de archivo en un blob nuevo con un nivel en línea
El siguiente script de PowerShell se puede usar para copiar un blob de archivo en un nuevo blob dentro de la misma cuenta de almacenamiento. La variable `$rgName` se debe inicializar con el nombre del grupo de recursos. La variable `$accountName` se debe inicializar con el nombre de la cuenta de almacenamiento. Las variables `$srcContainerName` y `$destContainerName` se deben inicializar con los nombres de los contenedores. Las variables `$srcBlobName` y `$destBlobName` se deben inicializar con los nombres de los blobs. 
```powershell
#Initialize the following with your resource group, storage account, container, and blob names
$rgName = ""
$accountName = ""
$srcContainerName = ""
$destContainerName = ""
$srcBlobName = ""
$destBlobName = ""

#Select the storage account and get the context
$storageAccount =Get-AzStorageAccount -ResourceGroupName $rgName -Name $accountName
$ctx = $storageAccount.Context

#Copy source blob to a new destination blob with access tier hot using standard rehydrate priority
Start-AzStorageBlobCopy -SrcContainer $srcContainerName -SrcBlob $srcBlobName -DestContainer $destContainerName -DestBlob $destBlobName -StandardBlobTier Hot -RehydratePriority Standard -Context $ctx
```

## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre los niveles de Blob Storage](storage-blob-storage-tiers.md)
* [Comprobación de los precios de los niveles de acceso frecuente, esporádico y de archivo en cuentas de Blob Storage y de GPv2 por región](https://azure.microsoft.com/pricing/details/storage/)
* [Administración del ciclo de vida de Azure Blob Storage](storage-lifecycle-management-concepts.md)
* [Detalles de precios de Transferencias de datos](https://azure.microsoft.com/pricing/details/data-transfers/)
