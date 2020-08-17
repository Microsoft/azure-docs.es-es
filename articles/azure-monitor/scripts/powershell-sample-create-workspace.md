---
title: Creación de un área de trabajo de Log Analytics con Azure PowerShell
description: 'Ejemplo de script de Azure PowerShell: Creación de un área de trabajo de Log Analytics'
ms.subservice: logs
ms.topic: sample
author: bwren
ms.author: bwren
ms.date: 09/07/2017
ms.openlocfilehash: f6bfb3a244874f6160d34c174b6d10b9a03ca437
ms.sourcegitcommit: 3d79f737ff34708b48dd2ae45100e2516af9ed78
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/23/2020
ms.locfileid: "87024136"
---
# <a name="create-a-log-analytics-workspace-with-powershell"></a>Creación de un área de trabajo de Log Analytics con PowerShell

Este script le permite trabajar rápidamente con un área de trabajo de Azure Log Analytics, que es necesaria si desea empezar a recopilar, analizar y realizar acciones en los datos.  

[!INCLUDE [sample-powershell-install](../../../includes/sample-powershell-install-no-ssh.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../powershell_scripts/log-analytics/log-analytics-create-new-resource/log-analytics-create-new-resource.ps1 "Create new Log Analytics workspace")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear una nueva área de trabajo de Log Analytics en su suscripción. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [Get-AzOperationalInsightsWorkspace](/powershell/module/az.operationalinsights/get-azoperationalinsightsworkspace) | Le permite obtener información sobre un área de trabajo existente. |
| [New-AzOperationalInsightsWorkspace](/powershell/module/az.operationalinsights/new-azoperationalinsightsworkspace) | Le permite crear un área de trabajo en la ubicación y el grupo de recursos especificados. |


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el módulo de Azure PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/).

