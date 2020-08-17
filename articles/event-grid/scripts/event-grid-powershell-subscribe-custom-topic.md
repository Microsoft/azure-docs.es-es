---
title: 'Ejemplo de script de Azure PowerShell: suscripción a temas personalizados | Microsoft Docs'
description: En este artículo se proporciona un script de Azure PowerShell de ejemplo que muestra cómo suscribirse a los eventos de Azure Event Grid de un tema personalizado.
ms.devlang: powershell
ms.topic: sample
ms.date: 07/08/2020
ms.openlocfilehash: f37a90b84e5e55298efc0b3cd53812db95e4d86d
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87460735"
---
# <a name="subscribe-to-events-for-a-custom-topic-with-powershell"></a>Suscripción a eventos de un tema personalizado con PowerShell

Este script crea una suscripción de Event Grid a los eventos de un tema personalizado.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

El script de ejemplo de la versión preliminar requiere el módulo de Event Grid. Para instalarlo, ejecute `Install-Module -Name AzureRM.EventGrid -AllowPrerelease -Force -Repository PSGallery`.

## <a name="sample-script---stable"></a>Script de ejemplo: estable

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-powershell[main](../../../powershell_scripts/event-grid/subscribe-to-custom-topic/subscribe-to-custom-topic.ps1 "Subscribe to custom topic")]

## <a name="sample-script---preview-module"></a>Script de ejemplo: módulo de la versión preliminar

[!INCLUDE [requires-azurerm](../../../includes/requires-azurerm.md)]

[!code-powershell[main](../../../powershell_scripts/event-grid/subscribe-to-custom-topic-preview/subscribe-to-custom-topic-preview.ps1 "Subscribe to custom topic")]

## <a name="script-explanation"></a>Explicación del script

Este script usa el siguiente comando para crear la suscripción de eventos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzEventGridSubscription](/powershell/module/az.eventgrid/new-azeventgridsubscription) | Cree una suscripción de Event Grid. |

## <a name="next-steps"></a>Pasos siguientes

* Para una introducción a las aplicaciones administradas, consulte la [introducción a las aplicaciones administradas de Azure](../overview.md).
* Para más información sobre PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/get-started-azureps).
