---
title: Configuración de la replicación de objetos (versión preliminar)
titleSuffix: Azure Storage
description: Obtenga información sobre cómo configurar la replicación de objetos para copiar de forma asincrónica blobs en bloques de un contenedor de una cuenta de almacenamiento a otra.
services: storage
author: tamram
ms.service: storage
ms.topic: how-to
ms.date: 07/16/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: c28e869bff1d0e921a1e5a952dbfcb21ee97d16b
ms.sourcegitcommit: d68c72e120bdd610bb6304dad503d3ea89a1f0f7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/01/2020
ms.locfileid: "89228331"
---
# <a name="configure-object-replication-for-block-blobs-preview"></a>Configuración de la replicación de objetos para blobs en bloques (versión preliminar)

La replicación de objetos (versión preliminar) copia asincrónicamente los blobs en bloques entre una cuenta de almacenamiento de origen y una de destino. Para obtener más información sobre la replicación de objetos, vea [Replicación de objetos (versión preliminar)](object-replication-overview.md).

Al configurar la replicación de objetos, crea una política de replicación que especifica las cuentas de almacenamiento de origen y destino. Una directiva de replicación incluye una o más reglas que especifican un contenedor de origen y uno de destino e indican qué blobs en bloques del contenedor de origen se replicarán.

En este artículo se describe cómo configurar la replicación de objetos para la cuenta de almacenamiento mediante Azure Portal, PowerShell o la CLI de Azure. También puede usar una de las bibliotecas cliente del proveedor de recursos de Azure Storage para configurar la replicación de objetos.

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

## <a name="create-a-replication-policy-and-rules"></a>Creación de una directiva de replicación y reglas

Antes de configurar la replicación de objetos, cree las cuentas de almacenamiento de origen y destino si aún no existen. Ambas cuentas de almacenamiento deben ser de uso general v2. Para obtener más información, consulte [Creación de una cuenta de Azure Storage](../common/storage-account-create.md).

Una cuenta de almacenamiento puede servir como cuenta de origen para un máximo dos cuentas de destino. Y una cuenta de destino puede tener como máximo dos cuentas de origen. Las cuentas de origen y destino pueden estar en diferentes regiones. Puede configurar directivas de replicación independientes para replicar los datos en cada una de las cuentas de destino.

Antes comenzar, asegúrese de haberse registrado para obtener las versiones preliminares de las características siguientes:

- [Replicación de objetos (versión preliminar)](object-replication-overview.md)
- [Control de versiones de blobs](versioning-overview.md)
- [Compatibilidad con la fuente de cambios en Azure Blob Storage (versión preliminar)](storage-blob-change-feed.md)

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Antes de configurar la replicación de objetos en Azure Portal, cree los contenedores de origen y destino en sus cuentas de almacenamiento respectivas, si aún no existen. Así mismo, habilite el control de versiones de blobs y la fuente de cambios en la cuenta de origen, y el control de versiones de blobs en la cuenta de destino.

Para crear una directiva de replicación en Azure Portal, siga estos pasos:

1. Navegue hasta la cuenta de almacenamiento de origen en Azure Portal.
1. En **Blob service**, seleccione **Replicación de objetos**.
1. Seleccione **Set up replication rules** (Configurar reglas de replicación).
1. Seleccione la suscripción de destino y la cuenta de almacenamiento.
1. En la sección **Container pairs** (Pares de contenedores), seleccione un contenedor de origen de la cuenta de origen y uno de destino de la cuenta de destino. Puede crear hasta 10 pares de contenedores por directiva de replicación.

    En la siguiente imagen se muestra un conjunto de reglas de replicación.

    :::image type="content" source="media/object-replication-configure/configure-replication-policy.png" alt-text="Captura de pantalla que muestra las reglas de replicación en Azure Portal":::

1. Si quiere, especifique uno o varios filtros para copiar solo los blobs que coinciden con un patrón de prefijo. Por ejemplo, si especifica un prefijo `b`, solo se replicarán los blobs cuyo nombre empiece por esa letra. Puede especificar un directorio virtual como parte del prefijo. La cadena de prefijo no admite caracteres comodín.

    En la imagen siguiente se muestran filtros que restringen los blobs que se copian como parte de una regla de replicación.

    :::image type="content" source="media/object-replication-configure/configure-replication-copy-prefix.png" alt-text="Captura de pantalla que muestra filtros para una regla de replicación":::

1. De forma predeterminada, el ámbito de copia se establece para copiar solo los nuevos objetos. Para copiar todos los objetos del contenedor o copiar objetos a partir de una fecha y hora personalizadas, seleccione el vínculo **Cambiar** y configure el ámbito de copia para el par de contenedores.

    En la imagen siguiente se muestra un ámbito de copia personalizado.

    :::image type="content" source="media/object-replication-configure/configure-replication-copy-scope.png" alt-text="Captura de pantalla que muestra el ámbito de copia personalizado para la replicación de objetos":::

1. Seleccione **Guardar y aplicar** para crear la directiva de replicación y empezar a replicar los datos.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para crear una directiva de replicación con PowerShell, primero instale la versión [2.0.1-preview](https://www.powershellgallery.com/packages/Az.Storage/2.0.1-preview) o posterior del módulo Az.Storage de PowerShell. Para instalar el módulo de la versión preliminar, siga estos pasos:

1. Desinstale cualquier instalación anterior de Azure PowerShell desde Windows usando el ajuste **Aplicaciones y características** en **Configuración**.

1. Asegúrese de tener instalada la versión más reciente de PowerShellGet. Abra una ventana de Windows PowerShell y ejecute el siguiente comando para instalar la versión más reciente:

    ```powershell
    Install-Module PowerShellGet –Repository PSGallery –Force
    ```

    Cierre y vuelva a abrir la ventana de PowerShell después de instalar PowerShellGet.

1. Instale la versión más reciente de Azure PowerShell:

    ```powershell
    Install-Module Az –Repository PSGallery –AllowClobber
    ```

1. Instale el módulo de versión preliminar Az.Storage:

    ```powershell
    Install-Module Az.Storage -Repository PSGallery -RequiredVersion 2.0.1-preview -AllowPrerelease -AllowClobber -Force
    ```

Para más información sobre cómo instalar Azure PowerShell, consulte [Instalación de Azure PowerShell con PowerShellGet](/powershell/azure/install-az-ps).

En el ejemplo siguiente se muestra cómo crear una directiva de replicación en las cuentas de origen y destino. No olvide reemplazar los valores entre corchetes angulares por sus propios valores:

```powershell
# Sign in to your Azure account.
Connect-AzAccount

# Set variables.
$rgName = "<resource-group>"
$srcAccountName = "<source-storage-account>"
$destAccountName = "<destination-storage-account>"
$srcContainerName1 = "source-container1"
$destContainerName1 = "dest-container1"
$srcContainerName2 = "source-container2"
$destContainerName2 = "dest-container2"

# Enable blob versioning and change feed on the source account.
Update-AzStorageBlobServiceProperty -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName `
    -EnableChangeFeed $true `
    -IsVersioningEnabled $true

# Enable blob versioning on the destination account.
Update-AzStorageBlobServiceProperty -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName `
    -IsVersioningEnabled $true

# List the service properties for both accounts.
Get-AzStorageBlobServiceProperty -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName
Get-AzStorageBlobServiceProperty -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName

# Create containers in the source and destination accounts.
Get-AzStorageAccount -ResourceGroupName $rgname -StorageAccountName $srcAccountName |
    New-AzStorageContainer $srcContainerName1
Get-AzStorageAccount -ResourceGroupName $rgname -StorageAccountName $destAccountName |
    New-AzStorageContainer $destContainerName1
Get-AzStorageAccount -ResourceGroupName $rgname -StorageAccountName $srcAccountName |
    New-AzStorageContainer $srcContainerName2
Get-AzStorageAccount -ResourceGroupName $rgname -StorageAccountName $destAccountName |
    New-AzStorageContainer $destContainerName2

# Define replication rules for each container.
$rule1 = New-AzStorageObjectReplicationPolicyRule -SourceContainer $srcContainerName1 `
    -DestinationContainer $destContainerName1 `
    -PrefixMatch b
$rule2 = New-AzStorageObjectReplicationPolicyRule -SourceContainer $srcContainerName2 `
    -DestinationContainer $destContainerName2  `
    -MinCreationTime 2020-05-10T00:00:00Z

# Create the replication policy on the destination account.
$destPolicy = Set-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName `
    -PolicyId default `
    -SourceAccount $srcAccountName `
    -Rule $rule1,$rule2

# Create the same policy on the source account.
Set-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName `
    -InputObject $destPolicy
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para crear una directiva de replicación con la CLI de Azure, instale primero la extensión de versión preliminar para Azure Storage:

```azurecli
az extension add -n storage-or-preview
```

A continuación, inicie sesión con sus credenciales de Azure:

```azurecli
az login
```

Habilite el control de versiones de blobs en las cuentas de almacenamiento de origen y destino, y la fuente de cambios en la cuenta de destino. No olvide reemplazar los valores entre corchetes angulares por sus propios valores:

```azurecli
az storage blob service-properties update \
    --resource-group <resource-group> \
    --account-name <source-storage-account> \
    --enable-versioning

az storage blob service-properties update \
    --resource-group <resource-group> \
    --account-name <source-storage-account> \
    --enable-change-feed

az storage blob service-properties update \
    --resource-group <resource-group> \
    --account-name <dest-storage-account> \
    --enable-versioning
```

Cree los contenedores de origen y destino en sus cuentas de almacenamiento respectivas.

```azurecli
az storage container create \
    --account-name <source-storage-account> \
    --name source-container3 \
    --auth-mode login
az storage container create \
    --account-name <source-storage-account> \
    --name source-container4 \
    --auth-mode login

az storage container create \
    --account-name <dest-storage-account> \
    --name source-container3 \
    --auth-mode login
az storage container create \
    --account-name <dest-storage-account> \
    --name source-container4 \
    --auth-mode login
```

Cree una nueva directiva de replicación y reglas asociadas en la cuenta de destino.

```azurecli
az storage account or-policy create \
    --account-name <dest-storage-account> \
    --resource-group <resource-group> \
    --source-account <source-storage-account> \
    --destination-account <dest-storage-account> \
    --source-container source-container3 \
    --destination-container dest-container3 \
    --min-creation-time '2020-05-10T00:00:00Z' \
    --prefix-match a

az storage account or-policy rule add \
    --account-name <dest-storage-account> \
    --destination-container dest-container4 \
    --policy-id <policy-id> \
    --resource-group <resource-group> \
    --source-container source-container4 \
    --prefix-match b
```

Cree la directiva en la cuenta de origen con el identificador de la directiva.

```azurecli
az storage account or-policy show \
    --resource-group <resource-group> \
    --name <dest-storage-account> \
    --policy-id <policy-id> |
    --az storage account or-policy create --resource-group <resource-group> \
    --name <source-storage-account> \
    --policy "@-"
```

---

## <a name="remove-a-replication-policy"></a>Eliminación de una directiva de replicación

Para quitar una directiva de replicación y sus reglas asociadas, use Azure Portal, PowerShell o la CLI.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para quitar una directiva de replicación en Azure Portal, siga estos pasos:

1. Navegue hasta la cuenta de almacenamiento de origen en Azure Portal.
1. En **Configuración**, seleccione **Replicación de objetos**.
1. Haga clic en el botón **Más** situado junto al nombre de la directiva.
1. Seleccione **Eliminar reglas**.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para quitar una directiva de replicación, elimine la directiva tanto de la cuenta de origen como de la de destino. Al eliminar la directiva, también se eliminan las reglas asociadas.

```powershell
# Remove the policy from the destination account.
Remove-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $destAccountName `
    -PolicyId $destPolicy.PolicyId

# Remove the policy from the source account.
Remove-AzStorageObjectReplicationPolicy -ResourceGroupName $rgname `
    -StorageAccountName $srcAccountName `
    -PolicyId $destPolicy.PolicyId
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para quitar una directiva de replicación, elimine la directiva de la cuenta de origen y destino. Al eliminar la directiva, también se eliminan las reglas asociadas.

```azurecli
az storage account or-policy delete \
    --policy-id $policyid \
    --account-name <source-storage-account> \
    --resource-group <resource-group>

az storage account or-policy delete \
    --policy-id $policyid \
    --account-name <dest-storage-account> \
    --resource-group <resource-group>
```

---

## <a name="next-steps"></a>Pasos siguientes

- [Información general sobre la replicación de objetos (versión preliminar)](object-replication-overview.md)
