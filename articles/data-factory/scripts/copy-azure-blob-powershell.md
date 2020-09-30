---
title: Copia de datos en la nube mediante PowerShell
description: Este script de PowerShell copia datos desde una ubicación en Azure Blob Storage a otra ubicación en la misma instancia de Blob Storage.
services: data-factory
ms.author: jingwang
author: linda33wj
manager: shwang
ms.service: data-factory
ms.workload: data-services
ms.topic: article
ms.custom: seo-lt-2019
ms.date: 03/12/2020
ms.openlocfilehash: cd8c8450d6e747e9789c785d1c594532b94a36cb
ms.sourcegitcommit: bf1340bb706cf31bb002128e272b8322f37d53dd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/03/2020
ms.locfileid: "89443082"
---
# <a name="use-powershell-to-create-a-data-factory-pipeline-to-copy-data-in-the-cloud"></a>Use PowerShell para crear una canalización de factoría de datos para copiar datos en la nube.

Este script de PowerShell de ejemplo crea una canalización en Azure Data Factory que copia los datos de una ubicación a otra en una instancia de Azure Blob Storage.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

## <a name="prerequisites"></a>Prerrequisitos
* **Cuenta de Azure Storage**. El almacenamiento de blobs se puede usar como almacén de datos de **origen** y **receptor**. Si no dispone de una cuenta de Azure Storage, consulte el artículo [Creación de una cuenta de almacenamiento](../../storage/common/storage-account-create.md) para aprender a crearla. 
* Cree un **contenedor de blobs** en Blob Storage, cree una **carpeta** de entrada en el contenedor y cargue algunos archivos en la carpeta. Puede usar herramientas como el [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) para conectarse a Azure Blob Storage, crear un contenedor de blobs, cargar el archivo de entrada y comprobar el archivo de salida.

## <a name="sample-script"></a>Script de ejemplo

> [!IMPORTANT]
> Este script crea archivos JSON que definen entidades de Data Factory (servicio vinculado, conjunto de datos y canalización) en la carpeta c:\ del disco duro.

[!code-powershell[main](../../../powershell_scripts/data-factory/copy-from-azure-blob-to-blob/copy-from-azure-blob-to-blob.ps1 "Copy from Blob Storage -> Blob Storage")]


## <a name="clean-up-deployment"></a>Limpieza de la implementación

Después de ejecutar el script de ejemplo, puede usar el comando siguiente para quitar el grupo de recursos y todos los recursos asociados a él:

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```
Para eliminar la factoría de datos del grupo de recursos, ejecute el siguiente comando:

```powershell
Remove-AzDataFactoryV2 -Name $dataFactoryName -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos:

| Get-Help | Notas |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [Set-AzDataFactoryV2](/powershell/module/az.datafactory/set-Azdatafactoryv2) | Creación de una factoría de datos. |
| [Set-AzDataFactoryV2LinkedService](/powershell/module/az.datafactory/Set-Azdatafactoryv2linkedservice) | Crea un servicio vinculado en la factoría de datos. Un servicio vinculado enlaza un almacén de datos o proceso a una factoría de datos. |
| [Set-AzDataFactoryV2Dataset](/powershell/module/az.datafactory/Set-Azdatafactoryv2dataset) | Crea un conjunto de datos en la factoría de datos. Un conjunto de datos representa la entrada/salida para una actividad en una canalización. |
| [Set-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/Set-Azdatafactoryv2pipeline) | Crea una canalización en la factoría de datos. Una canalización contiene una o varias actividades que realizan una operación determinada. En esta canalización, una actividad de copia realiza una copia de los datos de una ubicación en otra ubicación en una instancia de Azure Blob Storage. |
| [Invoke-AzDataFactoryV2Pipeline](/powershell/module/az.datafactory/Invoke-Azdatafactoryv2pipeline) | Crea una ejecución para la canalización. En otras palabras, ejecuta la canalización. |
| [Get-AzDataFactoryV2ActivityRun](/powershell/module/az.datafactory/get-Azdatafactoryv2activityrun) | Obtiene información detallada sobre la ejecución de la actividad (actividad ejecutar) en la canalización.
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Elimina un grupo de recursos, incluidos todos los recursos anidados. |
|||

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](https://docs.microsoft.com/powershell/).

Encontrará más ejemplos de scripts de PowerShell para Azure Data Factory en el artículo [Ejemplos de Azure PowerShell para Azure Data Factory](../samples-powershell.md).
