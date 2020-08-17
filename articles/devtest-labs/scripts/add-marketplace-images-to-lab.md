---
title: 'PowerShell: Incorporación de una imagen de Marketplace a un laboratorio de Azure DevTest Labs'
description: Este script de PowerShell agrega una imagen de Marketplace a un laboratorio de Azure DevTest Labs.
ms.devlang: azurecli
ms.topic: sample
ms.date: 08/11/2020
ms.openlocfilehash: 543f20af270769dd16e4a1ecf6ee93e9259cdfd1
ms.sourcegitcommit: 1aef4235aec3fd326ded18df7fdb750883809ae8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/12/2020
ms.locfileid: "88136242"
---
# <a name="use-powershell-to-add-a-marketplace-image-to-a-lab-in-azure-devtest-labs"></a>Usar PowerShell para agregar una imagen de Marketplace a un laboratorio de Azure DevTest Labs

Este script de PowerShell de ejemplo agrega una imagen de Marketplace a un laboratorio de Azure DevTest Labs. 

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

## <a name="prerequisites"></a>Prerequisites
* **Un laboratorio**. Este script requiere que disponga de un laboratorio existente. 

## <a name="sample-script"></a>Script de ejemplo

[!code-powershell[main](../../../powershell_scripts/devtest-lab/add-marketplace-images-to-lab/add-marketplace-images-to-lab.ps1 "Add marketplace images to a lab")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos: 

| Get-Help | Notas |
|---|---|
| Find-AzResource | Busca recursos en función de los parámetros especificados. |
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Obtiene recursos. |
| [Set-AzResource](/powershell/module/az.resources/set-azresource) | Modifica un recurso. |
| [New-AzResource](/powershell/module/az.resources/new-azresource) | Crea un recurso. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/).

Encontrará más ejemplos de script de PowerShell de Azure Lab Services en los [ejemplos de PowerShell de Azure Lab Services](../samples-powershell.md).
