---
title: 'Obtención de grupos de recursos administrados y cambio de tamaño de las máquinas virtuales: Azure PowerShell'
description: Proporciona un script de ejemplo de Azure PowerShell que obtiene un grupo de recursos administrado para una aplicación administrada de Azure. El script cambia el tamaño de las máquinas virtuales.
author: tfitzmac
ms.devlang: powershell
ms.topic: sample
ms.date: 10/27/2017
ms.author: tomfitz
ms.openlocfilehash: 602aaeb67ca081ebac71ca1d6d24a2de3c020603
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "86055994"
---
# <a name="get-resources-in-a-managed-resource-group-and-resize-vms-with-powershell"></a>Obtención de recursos de un grupo de recursos administrados y cambio de tamaño de las máquinas virtuales con PowerShell

Este script recupera los recursos de un grupo de recursos administrados y cambia el tamaño de las máquinas virtuales de ese grupo de recursos.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../../powershell_scripts/managed-applications/get-application/get-application.ps1 "Get application")]


## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para implementar la aplicación administrada. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [Get-AzManagedApplication](/powershell/module/az.resources/get-azmanagedapplication) | Enumera las aplicaciones administradas. Proporcione el nombre del grupo de recursos para enfocar los resultados. |
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Enumera los recursos. Proporcione un grupo de recursos y el tipo de recurso para enfocar el resultado. |
| [Update-AzVM](/powershell/module/az.compute/update-azvm) | Actualiza el tamaño de una máquina virtual. |


## <a name="next-steps"></a>Pasos siguientes

* Para una introducción a las aplicaciones administradas, consulte la [introducción a las aplicaciones administradas de Azure](../overview.md).
* Para más información sobre PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/get-started-azureps).
