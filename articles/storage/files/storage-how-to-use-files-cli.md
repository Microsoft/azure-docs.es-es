---
title: Guía de inicio rápido para administrar recursos compartidos de archivos de Azure mediante la CLI de Azure
description: Use esta guía de inicio rápido para aprender a usar la CLI de Azure para administrar Azure Files.
author: roygara
ms.service: storage
ms.topic: quickstart
ms.date: 10/26/2018
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurecli
ms.openlocfilehash: 0d9d0bad982a12e3b96bdbe4f680f2501b33ec67
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87495782"
---
# <a name="quickstart-create-and-manage-azure-file-shares-using-azure-cli"></a>Guía de inicio rápido: Creación y administración de recursos compartidos de archivos de Azure mediante la CLI de Azure
En esta guía se describen los conceptos básicos sobre cómo trabajar con [recursos compartidos de archivos de Azure](storage-files-introduction.md) mediante la CLI de Azure. Los recursos compartidos de archivos de Azure son iguales a otros recursos compartidos de archivos, pero se almacenan en la nube y están respaldados por la plataforma Azure. Los recursos compartidos de archivos de Azure admiten el protocolo SMB estándar del sector y permiten el uso compartido entre varias máquinas, aplicaciones e instancias. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI de Azure localmente, para los pasos de este artículo debe ejecutar la CLI de Azure versión 2.0.4 o versiones posteriores. Ejecute **az --version** para buscar la versión de la CLI de Azure. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure 2.0](/cli/azure/install-azure-cli). 

De forma predeterminada, los comandos de la CLI de Azure devuelven formato JSON (notación de objetos JavaScript). JSON es el método estándar para enviar y recibir mensajes desde las API REST. Para facilitar el trabajo con las respuestas JSON, algunos de los ejemplos de este artículo usan el parámetro *query* en los comandos de la CLI de Azure. Este parámetro utiliza el [lenguaje de consulta JMESPath](http://jmespath.org/) para el análisis de datos JSON. Para aprender más acerca de cómo usar los resultados de los comandos de la CLI de Azure mediante el lenguaje de consulta JMESPath, siga las indicaciones del [tutorial de JMESPath](http://jmespath.org/tutorial.html).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos
Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Si aún no tiene un grupo de recursos de Azure, puede utilizar el comando [az group create](/cli/azure/group) para crear uno nuevo. 

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *Oeste de EE. UU. 2*:

```azurecli-interactive 
export resourceGroupName="myResourceGroup"
region="westus2"

az group create \
    --name $resourceGroupName \
    --location $region \
    --output none
```

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento
Una cuenta de almacenamiento es un grupo compartido de almacenamiento en el que puede implementar recursos compartidos de archivos de Azure u otros recursos de almacenamiento como blobs o colas. Una cuenta de almacenamiento puede contener un número ilimitado de recursos compartidos de archivos. Un recurso compartido puede almacenar un número ilimitado de archivos, hasta los límites de capacidad de la cuenta de almacenamiento.

En el ejemplo siguiente se crea una cuenta de almacenamiento mediante el comando [az storage account create](/cli/azure/storage/account). Los nombres de la cuenta de almacenamiento deben ser únicos, por lo que debe usar `$RANDOM` para anexar un número al nombre para que sea único.

```azurecli-interactive 
export storageAccountName="mystorageacct$RANDOM"

az storage account create \
    --resource-group $resourceGroupName \
    --name $storageAccountName \
    --location $region \
    --kind StorageV2 \
    --sku Standard_LRS \
    --enable-large-file-share \
    --output none
```

> [!Note]  
> Los recursos compartidos de archivos mayores de 5 TiB (hasta un máximo de 100 TiB por recurso compartido) solo están disponibles en cuentas de almacenamiento con redundancia local (LRS) y con redundancia de zona (ZRS). Para crear una cuenta de almacenamiento con redundancia geográfica (GRS) o con redundancia de zona geográfica (GZRS), quite el parámetro `--enable-large-file-share`.

### <a name="get-the-storage-account-key"></a>Obtención de la clave de la cuenta de almacenamiento
Las claves de cuenta de almacenamiento controlan el acceso a los recursos de una cuenta de almacenamiento. Las claves se crean automáticamente al crear una cuenta de almacenamiento. Puede obtener las claves de cuenta de almacenamiento con el comando [az storage account keys list](/cli/azure/storage/account/keys): 

```azurecli-interactive 
export storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroupName \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"')
```

## <a name="create-an-azure-file-share"></a>Creación de un recurso compartido de archivos de Azure
Ahora puede crear su primer recurso compartido de archivos de Azure. Puede crear recursos compartidos de archivos con el comando [az storage share create](/cli/azure/storage/share). Este ejemplo crea un recurso compartido de archivos de Azure llamado *myshare*: 

```azurecli-interactive
shareName="myshare"

az storage share create \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $shareName \
    --quota 1024 \
    --output none
```

Los nombres de los recursos compartidos solo pueden contener letras minúsculas, números y guiones sencillos (pero no pueden empezar con un guion). Para obtener detalles completos sobre cómo asignar un nombre a los recursos compartidos y los archivos, consulte [Asignación de nombres y referencia a recursos compartidos, directorios, archivos y metadatos](https://docs.microsoft.com/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata).

## <a name="use-your-azure-file-share"></a>Uso de un recurso compartido de archivos de Azure
Azure Files proporciona dos métodos para trabajar con archivos y carpetas dentro de un recurso compartido de archivos de Azure: el [protocolo de bloque de mensajes del servidor (SMB)](https://msdn.microsoft.com/library/windows/desktop/aa365233.aspx) estándar del sector y el [REST de archivo](https://docs.microsoft.com/rest/api/storageservices/file-service-rest-api). 

Para montar un recurso compartido de archivos con SMB, consulte el siguiente documento según su sistema operativo:
- [Linux](storage-how-to-use-files-linux.md)
- [macOS](storage-how-to-use-files-mac.md)
- [Windows](storage-how-to-use-files-windows.md)

### <a name="using-an-azure-file-share-with-the-file-rest-protocol"></a>Uso de un recurso compartido de archivos de Azure con el protocolo File REST 
Se puede trabajar directamente con el protocolo REST de archivo (escribir a mano las llamadas HTTP de REST), pero la manera más habitual de usar este protocolo es con la CLI de Azure, el [módulo de Azure PowerShell](storage-how-to-use-files-powershell.md) o un SDK de Azure Storage; todos ellos proporcionan un buen contenedor para el protocolo REST de archivo en el lenguaje de programación o script de su elección.  

Es esperable que en la mayoría de casos de uso de Azure Files se trabajará con el recurso compartido de archivos de Azure a través del protocolo SMB, dado que permite usar las aplicaciones y herramientas existentes que se desean usar; sin embargo, existen varias razones por las que es beneficioso usar la API REST de archivo en lugar de SMB, como por ejemplo:

- Va a examinar el recurso compartido de archivos desde Cloud Shell de Bash de Azure (que no puede montar recursos compartidos de archivos a través de SMB).
- Se quieren aprovechar las ventajas de los recursos sin servidor, como [Azure Functions](../../azure-functions/functions-overview.md). 
- Va a crear un servicio de valor agregado que interactuará con muchos recursos compartidos de archivos de Azure, como la realización de análisis antivirus o de copias de seguridad.

En los ejemplos siguientes se muestra cómo usar la CLI de Azure para manipular el recurso compartido de archivos de Azure con el protocolo REST de archivo. 

### <a name="create-a-directory"></a>Creación de un directorio
Para crear un nuevo directorio llamado *myDirectory* en la raíz del recurso compartido de archivos de Azure, use el comando [`az storage directory create`](/cli/azure/storage/directory):

```azurecli-interactive
az storage directory create \
   --account-name $storageAccountName \
   --account-key $storageAccountKey \
   --share-name $shareName \
   --name "myDirectory" \
   --output none
```

### <a name="upload-a-file"></a>Cargar un archivo
Para demostrar cómo cargar un archivo mediante el comando [`az storage file upload`](/cli/azure/storage/file), primero cree un archivo para cargarlo en la unidad temporal de Cloud Shell. En el ejemplo siguiente, se crea y se carga el archivo:

```azurecli-interactive
cd ~/clouddrive/
date > SampleUpload.txt

az storage file upload \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --source "SampleUpload.txt" \
    --path "myDirectory/SampleUpload.txt"
```

Si está ejecutando la CLI de Azure localmente, sustituya `~/clouddrive` por una ruta de acceso que exista en la máquina.

Después de cargar el archivo, puede usar el comando [`az storage file list`](/cli/azure/storage/file) para asegurarse de que el archivo se cargó en el recurso compartido de archivos de Azure:

```azurecli-interactive
az storage file list \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --path "myDirectory" \
    --output table
```

### <a name="download-a-file"></a>Descarga de un archivo
Puede usar el comando [`az storage file download`](/cli/azure/storage/file) para descargar una copia del archivo que cargó en la unidad temporal de Cloud Shell:

```azurecli-interactive
# Delete an existing file by the same name as SampleDownload.txt, if it exists, because you've run this example before
rm -f SampleDownload.txt

az storage file download \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --path "myDirectory/SampleUpload.txt" \
    --dest "SampleDownload.txt" \
    --output none
```

### <a name="copy-files"></a>Copiar archivos
Una tarea habitual consiste en copiar los archivos de un recurso compartido de archivos a otro. Para demostrar esta funcionalidad, cree un nuevo recurso compartido. Copie el archivo que ha cargado en este nuevo recurso compartido mediante el comando [az storage file copy](/cli/azure/storage/file/copy): 

```azurecli-interactive
otherShareName="myshare2"

az storage share create \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $otherShareName \
    --quota 1024 \
    --output none

az storage directory create \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $otherShareName \
    --name "myDirectory2" \
    --output none

az storage file copy start \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --source-share $shareName \
    --source-path "myDirectory/SampleUpload.txt" \
    --destination-share $otherShareName \
    --destination-path "myDirectory2/SampleCopy.txt"
```

Ahora, si enumera los archivos del nuevo recurso compartido, debería ver el archivo copiado:

```azurecli-interactive
az storage file list \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $otherShareName \
    --path "myDirectory2" \
    --output table
```

Aunque el comando `az storage file copy start` es práctico para los movimientos de archivos entre recursos compartidos de archivos de Azure, en el caso de las migraciones y los movimientos de datos más grandes, se recomienda `rsync` en macOS y Linux y `robocopy` en Windows. `rsync` y `robocopy` usan SMB para realizar los movimientos de datos en lugar de la API de FileREST.

## <a name="create-and-manage-share-snapshots"></a>Creación y administración de instantáneas de recurso compartido
Otra tarea útil que puede hacer con un recurso compartido de archivos de Azure consiste en crear instantáneas del recurso compartido. Una instantánea conserva una copia de un momento dado de un recurso compartido de archivos de Azure. Las instantáneas del recurso compartido son similares a algunas tecnologías de sistemas operativos con las que es posible que ya esté familiarizado:

- Instantáneas del [Administrador de volúmenes lógicos (LVM)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)#Basic_functionality) para sistemas Linux.
- Instantáneas de [Apple File System (APFS)](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/APFS_Guide/Features/Features.html) para macOS.
- [Servicio de instantáneas de volumen (VSS)](https://docs.microsoft.com/windows/desktop/VSS/volume-shadow-copy-service-portal) para sistemas de archivos Windows como NTFS y ReFS.
 
Puede crear una instantánea del recurso compartido con el comando [`az storage share snapshot`](/cli/azure/storage/share):

```azurecli-interactive
snapshot=$(az storage share snapshot \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $shareName \
    --query "snapshot" | tr -d '"')
```

### <a name="browse-share-snapshot-contents"></a>Examen del contenido de la instantánea del recurso compartido
Para examinar el contenido de una instantánea del recurso compartido, pase la marca de tiempo de la instantánea del recurso compartido que ha capturado en la variable `$snapshot` al comando `az storage file list`:

```azurecli-interactive
az storage file list \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --snapshot $snapshot \
    --output table
```

### <a name="list-share-snapshots"></a>Enumerar instantáneas del recurso compartido
Puede ver la lista de instantáneas que ha realizado para el recurso compartido con el siguiente comando:

```azurecli-interactive
az storage share list \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --include-snapshot \
    --query "[? name== '$shareName' && snapshot!=null].snapshot" \
    --output tsv
```

### <a name="restore-from-a-share-snapshot"></a>Restauración desde una instantánea de recurso compartido
Puede restaurar un archivo con el comando `az storage file copy start` que ha utilizado antes. En primer lugar, elimine el archivo SampleUpload.txt que cargó, de forma que pueda restaurarlo desde la instantánea:

```azurecli-interactive
# Delete SampleUpload.txt
az storage file delete \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --path "myDirectory/SampleUpload.txt" \
    --output none

# Build the source URI for a snapshot restore
URI=$(az storage account show \
    --resource-group $resourceGroupName \
    --name $storageAccountName \
    --query "primaryEndpoints.file" | tr -d '"')

URI=$URI$shareName"/myDirectory/SampleUpload.txt?sharesnapshot="$snapshot

# Restore SampleUpload.txt from the share snapshot
az storage file copy start \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --source-uri $URI \
    --destination-share $shareName \
    --destination-path "myDirectory/SampleUpload.txt"
```

### <a name="delete-a-share-snapshot"></a>Eliminación de una instantánea de recurso compartido
Para eliminar una instantánea del recurso compartido, utilice el comando [`az storage share delete`](/cli/azure/storage/share). Use la variable que contiene la referencia `$SNAPSHOT` al parámetro `--snapshot`:

```azurecli-interactive
az storage share delete \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $shareName \
    --snapshot $snapshot \
    --output none
```

## <a name="clean-up-resources"></a>Limpieza de recursos
Cuando haya acabado, puede usar el comando [`az group delete`](/cli/azure/group) para eliminar el grupo de recursos y todos los recursos relacionados: 

```azurecli-interactive 
az group delete --name $resourceGroupName
```

Como alternativa, puede eliminar los recursos individualmente.
- Para eliminar los recursos compartidos de archivos de Azure que ha creado en este artículo:

    ```azurecli-interactive
    az storage share list \
            --account-name $storageAccountName \
            --account-key $storageAccountKey \
            --query "[].name" \
            --output tsv | \
        xargs -L1 bash -ec '\
            az storage share delete \
                --account-name "$storageAccountName" \
                --account-key "$storageAccountKey" \
                --name $0 \
                --delete-snapshots include \
                --output none'
    ```

- Para eliminar la propia cuenta de almacenamiento. (Esto elimina implícitamente los recursos compartidos de archivos de Azure que ha creado y cualquier otro recurso de almacenamiento que pueda haber creado, por ejemplo, un contenedor de Azure Blob Storage).

    ```azurecli-interactive
    az storage account delete \
        --resource-group $resourceGroupName \
        --name $storageAccountName \
        --yes
    ```

## <a name="next-steps"></a>Pasos siguientes
> [!div class="nextstepaction"]
> [¿Qué es Azure Files?](storage-files-introduction.md)
