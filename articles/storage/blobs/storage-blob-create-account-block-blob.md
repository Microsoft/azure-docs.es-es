---
title: 'Crear una cuenta de almacenamiento de blob en bloques: Azure Storage| Microsoft Docs'
description: Aquí se muestra cómo crear una cuenta de almacenamiento de BlockBlobStorage con las características de rendimiento Premium.
author: tamram
services: storage
ms.service: storage
ms.topic: how-to
ms.date: 05/10/2020
ms.author: tamram
ms.subservice: blobs
ms.custom: devx-track-azurecli, devx-track-azurepowershell
ms.openlocfilehash: 0e312cf49ee54020d0feb8d8e2f4c7a9bb77215a
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89074549"
---
# <a name="create-a-blockblobstorage-account"></a>Creación de una cuenta BlockBlobStorage

El tipo de cuenta BlockBlobStorage le permite crear blobs en bloques con características de rendimiento Premium. Este tipo de cuenta de almacenamiento está optimizado para cargas de trabajo con altas tasas de transacciones o que requieren tiempos de acceso muy rápidos. En este artículo se muestra cómo crear una cuenta BlockBlobStorage mediante Azure Portal, la CLI de Azure o Azure PowerShell.

> [!NOTE]
> La característica de espacio de nombres jerárquico en una cuenta de almacenamiento de blobs en bloques está en versión preliminar pública y está disponible en las regiones Este de EE. UU., Este de EE. UU. 2, Centro de EE. UU., Centro y Sur de EE. UU., Oeste de EE. UU. 2, Sur de Reino Unido, Centro de Canadá y Este de Australia. Para revisar las limitaciones, consulte [Características de Blob Storage disponibles en Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md) y [Problemas conocidos](data-lake-storage-known-issues.md). Para inscribirse en la versión preliminar, visite [este formulario](https://aka.ms/adlspremiumonboard).

Para obtener más información sobre las cuentas de BlockBlobStorage, consulte [Introducción a las cuentas de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-account-overview).

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

# <a name="portal"></a>[Portal](#tab/azure-portal)

Ninguno.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Este artículo de procedimientos requiere la versión 1.2.0 del módulo Az de Azure PowerShell o cualquier versión posterior. Ejecute `Get-Module -ListAvailable Az` para buscar la versión actual. Si necesita instalarla o actualizarla, consulte el artículo sobre [cómo instalar el módulo de Azure PowerShell](/powershell/azure/install-Az-ps).

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Puede iniciar sesión en Azure y ejecutar los comandos de la CLI de Azure de dos maneras:

- Puede ejecutar los comandos de la CLI desde Azure Portal, en Azure Cloud Shell.
- Puede instalar la CLI y ejecutar sus comandos en local.

### <a name="use-azure-cloud-shell"></a>Uso de Azure Cloud Shell

Azure Cloud Shell es un shell de Bash gratuito que puede ejecutarse directamente en Azure Portal. La CLI de Azure está preinstalada y configurada para su uso con la cuenta. Haga clic en el botón **Cloud Shell** del menú situado en la sección superior derecha de Azure Portal:

[![Cloud Shell](../common/media/storage-quickstart-create-account/cloud-shell-menu.png)](https://portal.azure.com)

El botón inicia un shell interactivo que se puede usar para ejecutar los pasos de este artículo de procedimientos:

[![Captura de pantalla en la que aparece la ventana de Cloud Shell del portal](../common/media/storage-quickstart-create-account/cloud-shell.png)](https://portal.azure.com)

### <a name="install-the-cli-locally"></a>Instalación local de la CLI

También puede instalar y usar la CLI de Azure localmente. Para este artículo de procedimientos es preciso usar la versión 2.0.46 de la CLI de Azure o cualquier versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli). 

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

```azurecli
az login
```

---

## <a name="create-a-blockblobstorage-account"></a>Creación de una cuenta BlockBlobStorage

## <a name="portal"></a>[Portal](#tab/azure-portal)
Para crear una cuenta de BlockBlobStorage en Azure Portal, siga estos pasos:

1. En Azure Portal, seleccione **Todos los servicios** > la categoría **Almacenamiento** > **Cuentas de almacenamiento**.

2. En **Cuentas de almacenamiento**, seleccione **Agregar**.

3. Seleccione la suscripción en la que se va a crear la cuenta de almacenamiento en el campo **Suscripción**.

4. En el campo del **Grupo de recursos**, seleccione un grupo de recursos existente o **Crear nuevo** y escriba un nombre para el grupo de recursos.

5. En el campo **Nombre de la cuenta de almacenamiento**, escriba un nombre para la cuenta. Tenga en cuenta estas directrices:

   - El nombre debe ser único en Azure.
   - El nombre debe tener entre tres y 24 caracteres.
   - El nombre solo puede contener números y letras minúsculas.

6. En el campo **Ubicación**, seleccione una ubicación para la cuenta de almacenamiento o utilice la ubicación predeterminada.

7. Defina la configuración de los siguientes valores:

   |Campo     |Value  |
   |---------|---------|
   |**Rendimiento**    |  Seleccione **Premium**.   |
   |**Tipo de cuenta**    | Seleccione **BlockBlobStorage**.      |
   |**Replicación**    |  Deje el valor predeterminado de **Almacenamiento con redundancia local (LRS)** .      |

   ![Se muestra la interfaz de usuario del portal para crear una cuenta de almacenamiento de blob en bloques](media/storage-blob-create-account-block-blob/create-block-blob-storage-account.png)

8. Elija la pestaña **Opciones avanzadas**.

9. Si desea optimizar la cuenta de almacenamiento para el análisis de datos, establezca **Espacio de nombres jerárquico** como **Habilitado**. De lo contrario, deje esta opción establecida en su valor predeterminado.

   Para obtener más información, consulte [Introducción a Azure Data Lake Storage Gen2](data-lake-storage-introduction.md).

   > [!NOTE]
   > La característica de espacio de nombres jerárquico en una cuenta de almacenamiento de blobs en bloques está en versión preliminar pública y está disponible en las regiones Este de EE. UU., Este de EE. UU. 2, Centro de EE. UU., Centro y Sur de EE. UU., Oeste de EE. UU. 2, Sur de Reino Unido, Centro de Canadá y Este de Australia. Para revisar las limitaciones, consulte [Características de Blob Storage disponibles en Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md) y [Problemas conocidos](data-lake-storage-known-issues.md). Para inscribirse en la versión preliminar, visite [este formulario](https://aka.ms/adlspremiumonboard).

8. Seleccione **Revisar y crear** para revisar la configuración de la cuenta de almacenamiento.

9. Seleccione **Crear**.

## <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

1. Abra una sesión de Windows PowerShell con privilegios elevados (ejecución como administrador).

2. Ejecute el siguiente comando para asegurarse de que la versión más reciente del módulo de PowerShell `Az` esté instalada.

   ```powershell
   Install-Module -Name Az -AllowClobber
   ```

3. Abra una nueva consola de PowerShell e inicie sesión con su cuenta de Azure.

   ```powershell
   Connect-AzAccount -SubscriptionId <SubscriptionID>
   ```

4. Si es necesario, cree un grupo de recursos. Reemplace los valores por expresiones de código delimitadas y ejecute el siguiente comando.

   ```powershell
   $resourcegroup = "new_resource_group_name"
   $location = "region_name"
   New-AzResourceGroup -Name $resourceGroup -Location $location
   ```

5. Cree la cuenta BlockBlobStorage. Reemplace los valores por expresiones de código delimitadas y ejecute el siguiente comando.

   ```powershell
   $resourcegroup = "resource_group_name"
   $storageaccount = "new_storage_account_name"
   $location = "region_name"

   New-AzStorageAccount -ResourceGroupName $resourcegroup -Name $storageaccount -Location $location -Kind "BlockBlobStorage" -SkuName "Premium_LRS"
   ```
   Si desea optimizar la cuenta de almacenamiento para el análisis de datos, agregue `-EnableHierarchicalNamespace $True` al comando. Para obtener más información, consulte [Introducción a Azure Data Lake Storage Gen2](data-lake-storage-introduction.md).

   > [!NOTE]
   > La característica de espacio de nombres jerárquico en una cuenta de almacenamiento de blobs en bloques está en versión preliminar pública y está disponible en las regiones Este de EE. UU., Este de EE. UU. 2, Centro de EE. UU., Centro y Sur de EE. UU., Oeste de EE. UU. 2, Sur de Reino Unido, Centro de Canadá y Este de Australia. Para revisar las limitaciones, consulte [Características de Blob Storage disponibles en Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md) y [Problemas conocidos](data-lake-storage-known-issues.md). Para inscribirse en la versión preliminar, visite [este formulario](https://aka.ms/adlspremiumonboard).

## <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para crear una cuenta de blob en bloques con la CLI de Azure, debe instalar primero la CLI de Azure en su versión 2.0.46 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

1. Inicie sesión en la suscripción de Azure.

   ```azurecli
   az login
   ```

2. Si es necesario, cree un grupo de recursos. Reemplace los valores entre corchetes (incluidos los corchetes) y ejecute el siguiente comando.

   ```azurecli
   az group create \
    --name "<new_resource_group_name>" \
    --location "<location>"
   ```

3. Cree la cuenta BlockBlobStorage. Reemplace los valores entre corchetes (incluidos los corchetes) y ejecute el siguiente comando.

   ```azurecli
   az storage account create \
    --location "<location>" \
    --name "<new_storage_account_name>" \
    --resource-group "<resource_group_name>" \
    --kind "BlockBlobStorage" \
    --sku "Premium_LRS"
   ```

   Si desea optimizar la cuenta de almacenamiento para el análisis de datos, agregue `--hierarchical-namespace true` al comando. Para obtener más información, consulte [Introducción a Azure Data Lake Storage Gen2](data-lake-storage-introduction.md).

   > [!NOTE]
   > La característica de espacio de nombres jerárquico en una cuenta de almacenamiento de blobs en bloques está en versión preliminar pública y está disponible en las regiones Este de EE. UU., Este de EE. UU. 2, Centro de EE. UU., Centro y Sur de EE. UU., Oeste de EE. UU. 2, Sur de Reino Unido, Centro de Canadá y Este de Australia. Para revisar las limitaciones, consulte [Características de Blob Storage disponibles en Azure Data Lake Storage Gen2](data-lake-storage-supported-blob-storage-features.md) y [Problemas conocidos](data-lake-storage-known-issues.md). Para inscribirse en la versión preliminar, visite [este formulario](https://aka.ms/adlspremiumonboard).
   
---

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre las cuentas de almacenamiento, vea [Introducción a las cuentas de Azure Storage](https://docs.microsoft.com/azure/storage/common/storage-account-overview).

- Para más información sobre los grupos de recursos, consulte [Información general de Azure Resource Manager](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-overview).
