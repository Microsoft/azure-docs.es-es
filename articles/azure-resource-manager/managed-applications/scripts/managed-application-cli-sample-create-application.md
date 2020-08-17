---
title: 'Ejemplo de script de la CLI de Azure: implementación de una aplicación administrada'
description: Proporciona un ejemplo de script de la CLI de Azure que implementa una definición de aplicación administrada de Azure a la suscripción.
author: tfitzmac
ms.devlang: azurecli
ms.topic: sample
ms.date: 10/25/2017
ms.author: tomfitz
ms.custom: devx-track-azurecli
ms.openlocfilehash: a9912f5134c3596740ef53f23531d57fe9467682
ms.sourcegitcommit: 11e2521679415f05d3d2c4c49858940677c57900
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/31/2020
ms.locfileid: "87497875"
---
# <a name="deploy-a-managed-application-for-service-catalog-with-azure-cli"></a>Implementación de una aplicación administrada para el catálogo de servicios con la CLI de Azure

Este script implementa una definición de aplicación administrada desde el catálogo de servicios. 


[!INCLUDE [sample-cli-install](../../../../includes/sample-cli-install.md)]

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

## <a name="sample-script"></a>Script de ejemplo

[!code-azurecli[main](../../../../cli_scripts/managed-applications/create-application/create-application.sh "Create application")]


## <a name="script-explanation"></a>Explicación del script

Este script usa el siguiente comando para implementar la aplicación administrada. Cada comando de la tabla crea un vínculo a documentación específica del comando.

| Get-Help | Notas |
|---|---|
| [az managedapp create](/cli/azure/managedapp#az-managedapp-create) | Cree una aplicación administrada. Proporcione el identificador de definición y los parámetros para la plantilla. |


## <a name="next-steps"></a>Pasos siguientes

* Para una introducción a las aplicaciones administradas, consulte la [introducción a las aplicaciones administradas de Azure](../overview.md).
* Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).
