---
title: 'Inicio rápido: Creación de un blob con PowerShell'
titleSuffix: Azure Storage
description: En esta guía de inicio rápido, utilizará Azure PowerShell en el almacenamiento de objetos (Blob). Después, puede usar PowerShell para cargar un blob en Azure Storage, descargar un blob o enumerar los blobs de un contenedor.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.date: 03/31/2020
ms.author: tamram
ms.openlocfilehash: 6ea03b1ff2e2a1e9b7d5256a0f1417c5ac113e0e
ms.sourcegitcommit: d8b8768d62672e9c287a04f2578383d0eb857950
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/11/2020
ms.locfileid: "88067331"
---
# <a name="quickstart-upload-download-and-list-blobs-with-powershell"></a>Inicio rápido: Carga, descarga y enumeración de blobs mediante PowerShell

Use el módulo de Azure PowerShell para crear y administrar los recursos de Azure. La creación o administración de los recursos de Azure puede realizarse desde la línea de comandos de PowerShell o en scripts. En esta guía se detalla el uso de PowerShell para transferir archivos entre el disco local y Azure Blob Storage.

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="prerequisites"></a>Prerrequisitos

Para acceder a Azure Storage, necesitará una suscripción de Azure. Si todavía no tiene una suscripción, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Para realizar los pasos de este inicio rápido, se requiere la versión 0.7 del módulo Az de Azure PowerShell, o cualquier versión posterior. Ejecute `Get-InstalledModule -Name Az -AllVersions | select Name,Version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte el artículo sobre [cómo instalar el módulo de Azure PowerShell](/powershell/azure/install-az-ps).

[!INCLUDE [storage-quickstart-tutorial-intro-include-powershell](../../../includes/storage-quickstart-tutorial-intro-include-powershell.md)]

## <a name="create-a-container"></a>Crear un contenedor

Los blobs siempre se cargan en un contenedor. Puede organizar los grupos de blobs de una forma similar a como organiza los archivos en carpetas en el equipo.

Establezca el nombre del contenedor y, después, cree el contenedor mediante [New-AzStorageContainer](/powershell/module/az.storage/new-azstoragecontainer). Establezca los permisos para `blob` de modo que se permita el acceso público de los archivos. El nombre del contenedor en este ejemplo es *quickstartblobs*.

```powershell
$containerName = "quickstartblobs"
New-AzStorageContainer -Name $containerName -Context $ctx -Permission blob
```

## <a name="upload-blobs-to-the-container"></a>Carga de blobs al contenedor

Blob Storage admite blobs en bloques, blobs en anexos y blobs en páginas. Los archivos VHD utilizados para respaldar VM IaaS son blobs en páginas. Use los blobs en anexos para el registro, por ejemplo, cuando desea escribir en un archivo y luego sigue agregando más información. La mayoría de los archivos almacenados en Blob Storage son blobs en bloques. 

Para cargar un archivo en un blob en bloques, obtenga una referencia de contenedor y luego obtenga una referencia al blob en bloques en ese contenedor. Cuando tenga la referencia del blob, puede cargar datos en él mediante [Set-AzStorageBlobContent](/powershell/module/az.storage/set-azstorageblobcontent). De este modo, se creará el blob si no existe, o se sobrescribirá si ya existe.

Los ejemplos siguientes cargan *Image001.jpg* e *Image002.png* desde la carpeta *D:\\_TestImages* del disco local al contenedor que creó.

```powershell
# upload a file to the default account (inferred) access tier
Set-AzStorageBlobContent -File "D:\_TestImages\Image000.jpg" `
  -Container $containerName `
  -Blob "Image001.jpg" `
  -Context $ctx 

# upload a file to the Hot access tier
Set-AzStorageBlobContent -File "D:\_TestImages\Image001.jpg" `
  -Container $containerName `
  -Blob "Image001.jpg" `
  -Context $ctx 
  -StandardBlobTier Hot

# upload another file to the Cool access tier
Set-AzStorageBlobContent -File "D:\_TestImages\Image002.png" `
  -Container $containerName `
  -Blob "Image002.png" `
  -Context $ctx
  -StandardBlobTier Cool

# upload a file to a folder to the Archive access tier
Set-AzStorageBlobContent -File "D:\_TestImages\foldername\Image003.jpg" `
  -Container $containerName `
  -Blob "Foldername/Image003.jpg" `
  -Context $ctx 
  -StandardBlobTier Archive
```

Cargue tantos archivos como desee antes de continuar.

## <a name="list-the-blobs-in-a-container"></a>Enumerar los blobs de un contenedor

Obtenga una lista de blobs del contenedor mediante [Get-AzStorageBlob](/powershell/module/az.storage/get-azstorageblob). Este ejemplo muestra solo los nombres de los blobs cargados.

```powershell
Get-AzStorageBlob -Container $ContainerName -Context $ctx | select Name
```

## <a name="download-blobs"></a>Descargar blobs

Descargue los blobs en el disco local. Para cada blob que quiera descargar, establezca el nombre y llame a [Get-AzStorageBlobContent](/powershell/module/az.storage/get-azstorageblobcontent) para descargar el blob.

Este ejemplo descarga los blobs a *D:\\_TestImages\Downloads* en el disco local. 

```powershell
# download first blob
Get-AzStorageBlobContent -Blob "Image001.jpg" `
  -Container $containerName `
  -Destination "D:\_TestImages\Downloads\" `
  -Context $ctx 

# download another blob
Get-AzStorageBlobContent -Blob "Image002.png" `
  -Container $containerName `
  -Destination "D:\_TestImages\Downloads\" `
  -Context $ctx
```

## <a name="data-transfer-with-azcopy"></a>Transferencia de datos con AzCopy

La utilidad de línea de comandos AzCopy ofrece transferencia de datos de alto rendimiento mediante scripts para Azure Storage. Puede utilizar AzCopy para transferir datos a y desde Blob Storage y Azure Files. Para más información sobre AzCopy v10, la versión más reciente de AzCopy, consulte [Introducción a AzCopy](../common/storage-use-azcopy-v10.md). Para más información acerca del uso de AzCopy v10 con el almacenamiento de blobs, consulte [Transferencia de datos con AzCopy y Blob Storage](../common/storage-use-azcopy-blobs.md).

En el ejemplo siguiente se usa AzCopy para cargar un archivo local en un blob. No olvide reemplazar los valores de ejemplo por sus propios valores:

```powershell
azcopy login
azcopy copy 'C:\myDirectory\myTextFile.txt' 'https://mystorageaccount.blob.core.windows.net/mycontainer/myTextFile.txt'
```

## <a name="clean-up-resources"></a>Limpieza de recursos

Quite todos los recursos que ha creado. La manera más fácil de quitar los recursos consiste en eliminar el grupo de recursos. Al quitar el grupo de recursos también se eliminan todos los recursos contenidos en él. En el ejemplo siguiente, al quitar el grupo de recursos, se quita la cuenta de almacenamiento y el propio grupo de recursos.

```powershell
Remove-AzResourceGroup -Name $resourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En esta guía de inicio rápido, ha trasferido archivos entre un sistema de archivos local y Azure Blob Storage. Para más información sobre cómo trabajar con Blob Storage mediante PowerShell, explore los ejemplos de Azure PowerShell para Blob Storage.

> [!div class="nextstepaction"]
> [Ejemplos de Azure PowerShell para Azure Blob Storage](storage-samples-blobs-powershell.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)

### <a name="microsoft-azure-powershell-storage-cmdlets-reference"></a>Referencia de cmdlets de almacenamiento de Microsoft Azure PowerShell

* [Cmdlets de PowerShell de almacenamiento](/powershell/module/az.storage)

### <a name="microsoft-azure-storage-explorer"></a>Explorador de Microsoft Azure Storage

* El [Explorador de Microsoft Azure Storage](../../vs-azure-tools-storage-manage-with-storage-explorer.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json) es una aplicación independiente y gratuita de Microsoft que permite trabajar visualmente con los datos de Azure Storage en Windows, macOS y Linux.
