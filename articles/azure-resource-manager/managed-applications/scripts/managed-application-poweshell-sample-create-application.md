---
title: 'Ejemplo de script de Azure PowerShell: implementación de una aplicación administrada'
description: Proporciona un ejemplo de script de Azure PowerShell que implementa una definición de aplicación administrada a la suscripción.
author: tfitzmac
ms.devlang: powershell
ms.topic: sample
ms.date: 10/27/2017
ms.author: tomfitz
ms.openlocfilehash: a2687e9c943df8454ff42a17f44866dcdb7f4730
ms.sourcegitcommit: 62717591c3ab871365a783b7221851758f4ec9a4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/22/2020
ms.locfileid: "86055892"
---
# <a name="deploy-a-managed-application-for-a-service-catalog-with-powershell"></a>Implementación de una aplicación administrada para el catálogo de servicios con PowerShell

Este script implementa una definición de aplicación administrada desde el catálogo de servicios.


[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../../powershell_scripts/managed-applications/create-application/create-application.ps1 "Create application")]


## <a name="script-explanation"></a>Explicación del script

Este script usa el siguiente comando para implementar la aplicación administrada. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzManagedApplication](/powershell/module/az.resources/new-azmanagedapplication) | Cree una aplicación administrada. Proporcione el identificador de definición y los parámetros para la plantilla. |


## <a name="next-steps"></a>Pasos siguientes

* Para una introducción a las aplicaciones administradas, consulte la [introducción a las aplicaciones administradas de Azure](../overview.md).
* Para más información sobre PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/get-started-azureps).
