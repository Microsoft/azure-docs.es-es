---
title: 'PowerShell: Creación de una imagen personalizada a partir de un archivo VHD en Azure Lab Services'
description: Este script de PowerShell crea una imagen personalizada a partir de un archivo VHD en Azure Lab Services.
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/11/2020
ms.openlocfilehash: d22e6e1d226e826bf114a0fdb378879b828d9b4a
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88136191"
---
# <a name="use-powershell-to-create-a-custom-image-from-a-vhd-file-in-azure-lab-services"></a>Uso de PowerShell para crear una imagen personalizada a partir de un archivo VHD en Azure Lab Services

Este script de PowerShell de ejemplo crea una imagen personalizada a partir de un archivo VHD en Azure Lab Services.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh-az.md)]

## <a name="prerequisites"></a>Prerequisites
* **Un laboratorio**. Este script requiere que disponga de un laboratorio existente. 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/devtest-lab/create-custom-image-from-vhd/create-custom-image-from-vhd.ps1 "Add external user to a lab")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos: 

| Get-Help | Notas |
|---|---|
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Obtiene recursos. |
| [Get-AzStorageAccountKey](/powershell/module/az.storage/get-azstorageaccountkey) | Obtiene las claves de acceso de una cuenta de Azure Storage. |
| [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) | Agrega una implementación de Azure a un grupo de recursos. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/).

Encontrará más ejemplos de script de PowerShell de Azure Lab Services en los [ejemplos de PowerShell de Azure Lab Services](../samples-powershell.md).
