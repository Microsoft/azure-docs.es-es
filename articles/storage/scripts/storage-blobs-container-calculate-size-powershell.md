---
title: Cálculo del tamaño de un contenedor de blobs con PowerShell
titleSuffix: Azure Storage
description: Calcule el tamaño de un contenedor en Azure Blob Storage sumando el tamaño de cada uno de sus blobs.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.devlang: powershell
ms.topic: sample
ms.date: 12/04/2019
ms.author: tamram
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: de275bcca1644750532809b35ae85d954d3cac6d
ms.sourcegitcommit: 656c0c38cf550327a9ee10cc936029378bc7b5a2
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/28/2020
ms.locfileid: "89076912"
---
# <a name="calculate-the-size-of-a-blob-container-with-powershell"></a>Cálculo del tamaño de un contenedor de blobs con PowerShell

Este script calcula el tamaño de un contenedor en Azure Blob Storage sumando el tamaño de los blobs de dicho contenedor.

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

> [!IMPORTANT]
> Este script de PowerShell proporciona un tamaño estimado del contenedor y no debe usarse para cálculos de facturación. Si quiere un script que calcule el tamaño del contenedor de cara a la facturación, consulte [Calculate the size of a Blob storage container for billing purposes](../scripts/storage-blobs-container-calculate-billing-size-powershell.md) (Cálculo del tamaño de un contenedor de Blob Storage con fines de facturación).

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/storage/calculate-container-size/calculate-container-size.ps1 "Calculate container size")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos, el contenedor y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name bloblisttestrg
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para calcular el tamaño del contenedor de Blob Storage. Cada elemento de la tabla incluye vínculos a la documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [Get-AzStorageAccount](/powershell/module/az.storage/get-azstorageaccount) | Obtiene una cuenta de Storage especificada o todas las cuentas de Storage de un grupo de recursos o la suscripción. |
| [Get-AzStorageBlob](/powershell/module/az.storage/Get-AzStorageBlob) | Enumera los blobs de un contenedor. |

## <a name="next-steps"></a>Pasos siguientes

Si quiere un script que calcule el tamaño del contenedor de cara a la facturación, consulte [Calculate the size of a Blob storage container for billing purposes](../scripts/storage-blobs-container-calculate-billing-size-powershell.md) (Cálculo del tamaño de un contenedor de Blob Storage con fines de facturación).

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Puede encontrar ejemplos de script adicionales de PowerShell de almacenamiento en los [ejemplos de PowerShell para Azure Storage](../blobs/storage-samples-blobs-powershell.md).
