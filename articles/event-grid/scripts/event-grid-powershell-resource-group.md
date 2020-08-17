---
title: 'Ejemplo de script de Azure PowerShell: suscripción a grupos de recursos | Microsoft Docs'
description: En este artículo se proporciona un script de Azure PowerShell de ejemplo que muestra cómo suscribirse a los eventos de Azure Event Grid de un grupo de recursos.
ms.devlang: powershell
ms.topic: sample
ms.date: 07/08/2020
ms.openlocfilehash: c8e84fe12cf239e7dbb0e6a15088efda793ddec9
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87460752"
---
# <a name="subscribe-to-events-for-a-resource-group-with-powershell"></a>Suscripción a eventos de un grupo de recursos con PowerShell

Este script crea una suscripción de Event Grid a los eventos de un grupo de recursos.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

El script de ejemplo de la versión preliminar requiere el módulo de Event Grid. Para instalarlo, ejecute `Install-Module -Name AzureRM.EventGrid -AllowPrerelease -Force -Repository PSGallery`.

## <a name="sample-script---stable"></a>Script de ejemplo: estable

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../powershell_scripts/event-grid/subscribe-to-resource-group/subscribe-to-resource-group.ps1 "Subscribe to resource group")]

## <a name="sample-script---preview-module"></a>Script de ejemplo: módulo de la versión preliminar

[!INCLUDE [requires-azurerm](../../../includes/requires-azurerm.md)]

[!code-powershell[main](../../../powershell_scripts/event-grid/subscribe-to-resource-group-preview/subscribe-to-resource-group-preview.ps1 "Subscribe to resource group")]

## <a name="script-explanation"></a>Explicación del script

Este script usa el siguiente comando para crear la suscripción de eventos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzEventGridSubscription](/powershell/module/az.eventgrid/new-azeventgridsubscription) | Cree una suscripción de Event Grid. |

## <a name="next-steps"></a>Pasos siguientes

* Para una introducción a las aplicaciones administradas, consulte la [introducción a las aplicaciones administradas de Azure](../overview.md).
* Para más información sobre PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/get-started-azureps).
