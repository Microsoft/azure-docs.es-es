---
title: Rotación de las claves de acceso de las cuentas de almacenamiento con PowerShell
titleSuffix: Azure Storage
description: Cree una cuenta de Azure Storage y luego recupere y rote una de sus claves de acceso de cuenta.
services: storage
author: tamram
ms.service: storage
ms.subservice: blobs
ms.devlang: powershell
ms.topic: sample
ms.date: 12/04/2019
ms.author: tamram
ms.openlocfilehash: d532782847ec09d2a480951ecd77ae7746b000c2
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87089383"
---
# <a name="rotate-storage-account-access-keys-with-powershell"></a>Rotación de las claves de acceso de las cuentas de almacenamiento con PowerShell

Este script crea una cuenta de Azure Storage, muestra la clave de acceso principal de la nueva cuenta de almacenamiento y, por último, renueva (rota) la clave.

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/storage/rotate-storage-account-keys/rotate-storage-account-keys.ps1 "Rotate storage account keys")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación

Ejecute el siguiente comando para quitar el grupo de recursos, la cuenta de almacenamiento y todos los recursos relacionados.

```powershell
Remove-AzResourceGroup -Name rotatekeystestrg
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear la cuenta de almacenamiento y recuperar y rotar una de sus claves de acceso. Cada elemento de la tabla incluye vínculos a la documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [Get-AzLocation](/powershell/module/az.resources/get-azlocation) | Obtiene todas las ubicaciones y los proveedores de recursos admitidos para cada ubicación. |
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crea un grupo de recursos de Azure. |
| [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) | Crea una cuenta de Storage. |
| [Get-AzStorageAccountKey](/powershell/module/az.storage/get-azstorageaccountkey) | Obtiene las claves de acceso de una cuenta de Azure Storage. |
| [New-AzStorageAccountKey](/powershell/module/az.storage/new-azstorageaccountkey) | Vuelve a generar la claves de acceso de una cuenta de Azure Storage. |

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

Puede encontrar ejemplos de script adicionales de PowerShell de almacenamiento en los [ejemplos de PowerShell para Azure Blob Storage](../blobs/storage-samples-blobs-powershell.md).
