---
title: 'Ejemplo de script de la CLI de Azure: creación de un tema personalizado | Microsoft Docs'
description: En este artículo se proporciona un script de la CLI de Azure de ejemplo que muestra cómo crear un tema personalizado de Azure Event Grid.
ms.devlang: azurecli
ms.topic: sample
ms.date: 07/08/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: d61dac0e76fc7b7006fa36f1cb96965ccee3d863
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87494135"
---
# <a name="create-event-grid-custom-topic-with-azure-cli"></a>Creación de un tema personalizado de Event Grid con la CLI de Azure

Este script crea un tema personalizado de Event Grid.

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli[main](../../../cli_scripts/event-grid/create-custom-topic/create-custom-topic.sh "Create custom topic")]

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear el tema personalizado. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az eventgrid topic create](/cli/azure/eventgrid/topic#az-eventgrid-topic-create) | Permite crear un tema personalizado de Event Grid. |


## <a name="next-steps"></a>Pasos siguientes

* Para información sobre la consulta de suscripciones, consulte [Consulta de suscripciones de Event Grid](../query-event-subscriptions.md).
* Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).
