---
title: 'Azure PowerShell: suscripción a una cuenta de Blob Storage'
description: En este artículo se proporciona un script de Azure PowerShell de ejemplo que muestra cómo suscribirse a los eventos de Azure Event Grid de una cuenta de Blob Storage.
ms.devlang: powershell
ms.topic: sample
ms.date: 07/08/2020
ms.openlocfilehash: 15daa9843eb721ebf9caa2ae49e75fadbd0a4be8
ms.sourcegitcommit: f988fc0f13266cea6e86ce618f2b511ce69bbb96
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87460803"
---
# <a name="subscribe-to-events-for-a-blob-storage-account-with-powershell"></a>Suscripción a eventos de una cuenta de Blob Storage con PowerShell

Este script crea una suscripción de Event Grid a los eventos de una cuenta de Blob Storage.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script---stable"></a>Script de ejemplo: estable

[!code-powershell[main](../../../powershell_scripts/event-grid/subscribe-to-blob-storage/subscribe-to-blob-storage.ps1 "Subscribe to Blob storage")]

## <a name="script-explanation"></a>Explicación del script

Este script usa el siguiente comando para crear la suscripción de eventos. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [New-AzEventGridSubscription](/powershell/module/az.eventgrid/new-azeventgridsubscription) | Cree una suscripción de Event Grid. |

## <a name="next-steps"></a>Pasos siguientes

* Para una introducción a las aplicaciones administradas, consulte la [introducción a las aplicaciones administradas de Azure](../overview.md).
* Para más información sobre PowerShell, consulte la [documentación de Azure PowerShell](/powershell/azure/get-started-azureps).
